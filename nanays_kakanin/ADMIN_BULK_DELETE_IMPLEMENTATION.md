# Admin Bulk Delete Implementation Guide

## Overview
Add checkboxes to admin orders and reservations pages to allow bulk selection and deletion.

## ✅ What's Already Done

### 1. **Delete Button Added** ✅
- Added "Delete Selected" button next to Filter and Reset buttons
- Shows count of selected items
- Disabled when nothing selected
- Red color for destructive action

### 2. **JavaScript Functions Added** ✅
- `toggleAllOrders()` - Select/deselect all checkboxes
- `updateOrderSelection()` - Update UI based on selection
- `deleteSelectedOrders()` - Handle bulk delete with confirmation

### 3. **First Table Updated** ✅
- "Pending Payment Verification" table now has:
  - Select All checkbox in header
  - Individual checkboxes for each row

## 🔧 What Needs to Be Done

### Add Checkboxes to Remaining Tables

You need to add checkboxes to **5 more tables** in `admin_orders.html`:

1. **Pending Confirmation** (line ~240)
2. **Confirmed - Being Prepared** (line ~295)
3. **Ready for Pickup** (line ~346)
4. **Out for Delivery** (line ~392)
5. **Completed** (line ~453)

### For Each Table:

#### Step 1: Add Select All Checkbox in Header
```html
<thead class="bg-gray-50">
  <tr>
    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
      <input type="checkbox" class="w-4 h-4 text-green-600 rounded focus:ring-green-500 select-all-orders" onchange="toggleAllOrders(this)">
    </th>
    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Order ID</th>
    <!-- ... other headers ... -->
  </tr>
</thead>
```

#### Step 2: Add Individual Checkbox in Each Row
```html
<tr class="hover:bg-blue-50">
  <td class="px-6 py-4">
    <input type="checkbox" class="order-checkbox w-4 h-4 text-green-600 rounded focus:ring-green-500" value="{{ order.id }}" onchange="updateOrderSelection()">
  </td>
  <td class="px-6 py-4"><strong class="text-gray-900">#{{ order.id }}</strong></td>
  <!-- ... other cells ... -->
</tr>
```

## Backend Implementation

### Create Bulk Delete View in `views.py`:

```python
@login_required
@user_passes_test(lambda u: u.is_superuser)
def admin_bulk_delete_orders(request):
    """Bulk delete orders"""
    if request.method == 'POST':
        order_ids = request.POST.getlist('order_ids')
        
        if not order_ids:
            messages.error(request, 'No orders selected.')
            return redirect('admin_orders')
        
        try:
            # Delete the orders
            deleted_count = Order.objects.filter(id__in=order_ids).delete()[0]
            messages.success(request, f'Successfully deleted {deleted_count} order(s).')
        except Exception as e:
            messages.error(request, f'Error deleting orders: {str(e)}')
        
        return redirect('admin_orders')
    
    return redirect('admin_orders')
```

### Add URL Pattern in `urls.py`:

```python
path('admin/orders/bulk-delete/', views.admin_bulk_delete_orders, name='admin_bulk_delete_orders'),
```

## Reservations Page

### Apply Same Changes to `admin_reservations.html`:

1. **Add Delete Button** next to Filter/Reset
2. **Add Checkboxes** to all reservation tables
3. **Add JavaScript Functions** (similar to orders)
4. **Create Backend View** for bulk delete reservations

### Reservation JavaScript Functions:

```javascript
function toggleAllReservations(checkbox) {
  const reservationCheckboxes = document.querySelectorAll('.reservation-checkbox');
  reservationCheckboxes.forEach(cb => {
    cb.checked = checkbox.checked;
  });
  updateReservationSelection();
}

function updateReservationSelection() {
  const reservationCheckboxes = document.querySelectorAll('.reservation-checkbox');
  const selectAllCheckboxes = document.querySelectorAll('.select-all-reservations');
  const deleteBtn = document.getElementById('deleteReservationsBtn');
  const selectedCount = Array.from(reservationCheckboxes).filter(cb => cb.checked).length;
  
  // Update count
  document.getElementById('selectedReservationCount').textContent = selectedCount;
  
  // Enable/disable delete button
  if (selectedCount > 0) {
    deleteBtn.disabled = false;
    deleteBtn.classList.remove('opacity-50', 'cursor-not-allowed');
  } else {
    deleteBtn.disabled = true;
    deleteBtn.classList.add('opacity-50', 'cursor-not-allowed');
  }
  
  // Update select all checkboxes
  selectAllCheckboxes.forEach(cb => {
    if (selectedCount === 0) {
      cb.checked = false;
      cb.indeterminate = false;
    } else if (selectedCount === reservationCheckboxes.length) {
      cb.checked = true;
      cb.indeterminate = false;
    } else {
      cb.checked = false;
      cb.indeterminate = true;
    }
  });
}

function deleteSelectedReservations() {
  const selectedReservations = Array.from(document.querySelectorAll('.reservation-checkbox:checked')).map(cb => cb.value);
  
  if (selectedReservations.length === 0) {
    alert('Please select at least one reservation to delete.');
    return;
  }
  
  if (confirm(`Are you sure you want to delete ${selectedReservations.length} reservation(s)? This action cannot be undone.`)) {
    // Create form and submit
    const form = document.createElement('form');
    form.method = 'POST';
    form.action = '{% url "admin_bulk_delete_reservations" %}';
    
    // Add CSRF token
    const csrfToken = document.querySelector('[name=csrfmiddlewaretoken]');
    if (csrfToken) {
      const csrfInput = document.createElement('input');
      csrfInput.type = 'hidden';
      csrfInput.name = 'csrfmiddlewaretoken';
      csrfInput.value = csrfToken.value;
      form.appendChild(csrfInput);
    }
    
    // Add selected reservation IDs
    selectedReservations.forEach(reservationId => {
      const input = document.createElement('input');
      input.type = 'hidden';
      input.name = 'reservation_ids';
      input.value = reservationId;
      form.appendChild(input);
    });
    
    document.body.appendChild(form);
    form.submit();
  }
}
```

### Reservation Backend View:

```python
@login_required
@user_passes_test(lambda u: u.is_superuser)
def admin_bulk_delete_reservations(request):
    """Bulk delete reservations"""
    if request.method == 'POST':
        reservation_ids = request.POST.getlist('reservation_ids')
        
        if not reservation_ids:
            messages.error(request, 'No reservations selected.')
            return redirect('admin_reservations')
        
        try:
            # Delete the reservations
            deleted_count = Reservation.objects.filter(id__in=reservation_ids).delete()[0]
            messages.success(request, f'Successfully deleted {deleted_count} reservation(s).')
        except Exception as e:
            messages.error(request, f'Error deleting reservations: {str(e)}')
        
        return redirect('admin_reservations')
    
    return redirect('admin_reservations')
```

## Features

### ✅ Select All Checkbox
- Click to select/deselect all items in all tables
- Shows indeterminate state when some (but not all) items selected

### ✅ Individual Checkboxes
- Each row has its own checkbox
- Can select specific items

### ✅ Smart Selection Logic
- Delete button disabled when nothing selected
- Shows count of selected items
- Confirmation dialog before deletion

### ✅ Delete Action
- Red button for destructive action
- Confirmation prompt
- Bulk delete via POST request
- Success/error messages

## Visual Design

### Delete Button:
```
[Filter] [Reset] [Delete Selected (3)]
  ↑        ↑            ↑
Green    Gray         Red
```

### Table with Checkboxes:
```
┌──┬────────┬──────────┬───────┬──────┬─────────┐
│☑│Order ID│ Customer │ Total │ Date │ Actions │
├──┼────────┼──────────┼───────┼──────┼─────────┤
│☑│  #1    │ John Doe │ ₱30  │ ...  │ [View]  │
│☐│  #2    │ Jane     │ ₱50  │ ...  │ [View]  │
│☑│  #3    │ Bob      │ ₱40  │ ...  │ [View]  │
└──┴────────┴──────────┴───────┴──────┴─────────┘
```

## Testing

1. **Select Individual Items**
   - Check a few checkboxes
   - Verify count updates
   - Verify delete button enables

2. **Select All**
   - Click header checkbox
   - Verify all items selected
   - Verify count shows total

3. **Delete**
   - Select items
   - Click Delete button
   - Confirm dialog
   - Verify items deleted
   - Verify success message

4. **Indeterminate State**
   - Select some (not all) items
   - Verify header checkbox shows dash (-)
   - Click header checkbox
   - Verify all items selected

## Notes

- Checkboxes use green color to match theme
- Delete button is red for destructive action
- Confirmation dialog prevents accidental deletion
- Works across all order/reservation statuses
- Responsive on mobile and desktop
