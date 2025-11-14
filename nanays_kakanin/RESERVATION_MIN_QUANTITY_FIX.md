# Reservation Minimum Quantity Fix

## Issue
The reservation modal was hardcoded to require a minimum of 10 pieces for all products, even when the product's `min_order_quantity` was set to 1. This caused confusion where:
- Product card showed "Min: 1 pcs"
- Reservation modal enforced "Minimum: 10 pieces"
- Users couldn't order just 1 piece even though the UI indicated they could

## Solution
Made the reservation modal dynamic to respect each product's individual `min_order_quantity` setting.

---

## Changes Made

### 1. Frontend - Shop User Template (`shop_user.html`)

#### A. Updated Button Click Handler (Line 427)
**Before:**
```javascript
onclick="openReservationModal({{ kakanin.id }}, '{{ kakanin.name }}', {{ kakanin.price }})"
```

**After:**
```javascript
onclick="openReservationModal({{ kakanin.id }}, '{{ kakanin.name }}', {{ kakanin.price }}, {{ kakanin.min_order_quantity|default:1 }})"
```

**Effect:** Now passes the product's minimum quantity to the modal function.

---

#### B. Updated Modal HTML (Lines 985-989)
**Before:**
```html
<input type="number" name="quantity" value="10" min="10" required>
<p class="text-sm text-gray-500 mt-1">Minimum: 10 pieces</p>
```

**After:**
```html
<input type="number" name="quantity" id="reservationQuantity" value="1" min="1" required>
<p class="text-sm text-gray-500 mt-1" id="reservationMinText">Minimum: 1 piece</p>
```

**Effect:** Added IDs for dynamic updates and changed default to 1.

---

#### C. Updated JavaScript Function (Lines 1089-1113)
**Before:**
```javascript
function openReservationModal(productId, productName, price) {
  document.getElementById('modalProductId').value = productId;
  document.getElementById('modalProductName').textContent = productName;
  document.getElementById('reservationForm').action = `/reservation/cart/add/${productId}/`;
  document.getElementById('reservationModal').classList.remove('hidden');
  
  // Set minimum date...
}
```

**After:**
```javascript
function openReservationModal(productId, productName, price, minQuantity) {
  minQuantity = minQuantity || 1; // Default to 1 if not provided
  
  document.getElementById('modalProductId').value = productId;
  document.getElementById('modalProductName').textContent = productName;
  document.getElementById('reservationForm').action = `/reservation/cart/add/${productId}/`;
  
  // Set minimum quantity
  const quantityInput = document.getElementById('reservationQuantity');
  quantityInput.value = minQuantity;
  quantityInput.min = minQuantity;
  
  // Update minimum text
  const minText = document.getElementById('reservationMinText');
  const pieceText = minQuantity === 1 ? 'piece' : 'pieces';
  minText.textContent = `Minimum: ${minQuantity} ${pieceText}`;
  
  document.getElementById('reservationModal').classList.remove('hidden');
  
  // Set minimum date...
}
```

**Effect:** 
- Accepts `minQuantity` parameter
- Dynamically sets the input's value and min attribute
- Updates the help text to show correct minimum with proper singular/plural

---

### 2. Backend - Reservation Views (`reservation_views.py`)

#### Added Minimum Quantity Validation (Lines 48-53)
**Added:**
```python
# Validate minimum order quantity
min_quantity = product.min_order_quantity if product.min_order_quantity else 1
if quantity < min_quantity:
    piece_text = "piece" if min_quantity == 1 else "pieces"
    messages.error(request, f'Minimum order quantity for {product.name} is {min_quantity} {piece_text}.')
    return redirect('reservation_shop')
```

**Effect:** Backend now validates that the submitted quantity meets the product's minimum requirement.

---

## How It Works Now

### Example 1: Product with min_order_quantity = 1
1. Product card shows: "Min: 1 pcs"
2. User clicks reservation button
3. Modal opens with:
   - Quantity field: value="1", min="1"
   - Help text: "Minimum: 1 piece"
4. User can order 1 or more pieces
5. Backend validates quantity ≥ 1

### Example 2: Product with min_order_quantity = 10
1. Product card shows: "Min: 10 pcs"
2. User clicks reservation button
3. Modal opens with:
   - Quantity field: value="10", min="10"
   - Help text: "Minimum: 10 pieces"
4. User must order 10 or more pieces
5. Backend validates quantity ≥ 10

### Example 3: Product with no min_order_quantity set
1. System defaults to 1
2. Modal shows minimum of 1 piece
3. Backend validates quantity ≥ 1

---

## Benefits

1. **Consistency:** UI and backend now match the product's actual minimum quantity
2. **Flexibility:** Each product can have its own minimum (1, 5, 10, etc.)
3. **User Experience:** Clear, accurate information prevents confusion
4. **Data Integrity:** Backend validation ensures rules are enforced
5. **Proper Grammar:** Singular/plural text ("1 piece" vs "10 pieces")

---

## Testing Checklist

- [x] Product with min_order_quantity = 1 shows correct minimum in modal
- [x] Product with min_order_quantity = 10 shows correct minimum in modal
- [x] Product with no min_order_quantity defaults to 1
- [x] Modal prevents entering quantity below minimum (HTML5 validation)
- [x] Backend rejects quantities below minimum with error message
- [x] Singular/plural text displays correctly ("1 piece" vs "2 pieces")
- [x] Existing functionality (date validation, cart addition) still works

---

## Related Files Modified

1. `kakanin/templates/kakanin/shop_user.html` - Frontend modal and JavaScript
2. `kakanin/reservation_views.py` - Backend validation

---

## Notes

- The fix respects the product's `min_order_quantity` field from the database
- If `min_order_quantity` is not set or is 0, it defaults to 1
- The HTML5 `min` attribute provides client-side validation
- Backend validation provides server-side security
- Error messages are user-friendly and grammatically correct
