# Notification System Migration Guide

## Overview
This guide explains how to migrate from manual notification creation to the new signal-based system.

---

## What Changed?

### Before (Manual Creation):
```python
# In views.py - checkout_cart function
order = Order.objects.create(...)

# Manually create notification
Notification.objects.create(
    type='order_submitted',
    message=f'Order #{order.id}: ...',
    user=None,  # Admin
    order=order
)
```

### After (Automatic via Signals):
```python
# In views.py - checkout_cart function
order = Order.objects.create(...)

# That's it! Signal automatically creates notification
# No manual Notification.objects.create() needed
```

---

## Migration Steps

### Step 1: Identify Manual Notification Creation

Search for `Notification.objects.create` in your codebase:
- `views.py` - Order and cart views
- `reservation_views.py` - Reservation views
- Any other files creating notifications

### Step 2: Determine if Removal is Safe

**Safe to Remove:**
- Notifications created when Order/Reservation is created (signals handle this)
- Notifications created when Order/Reservation status changes (signals handle this)

**Keep (Not Handled by Signals):**
- User-initiated cancellations (user cancels their own order)
- User confirms receipt of order
- Special case notifications not tied to status changes

### Step 3: Remove Duplicate Notifications

#### Example 1: Order Creation (REMOVE)

**Before:**
```python
order = Order.objects.create(
    user=request.user,
    status='pending_confirmation',
    ...
)

# REMOVE THIS - Signal handles it
Notification.objects.create(
    type='order_submitted',
    message=f'Order #{order.id}: ...',
    user=None,
    order=order
)
```

**After:**
```python
order = Order.objects.create(
    user=request.user,
    status='pending_confirmation',
    ...
)
# Signal automatically creates admin notification
```

#### Example 2: Status Change (REMOVE)

**Before:**
```python
order.status = 'confirmed'
order.save()

# REMOVE THIS - Signal handles it
Notification.objects.create(
    type='order_confirmed',
    message=f'Order #{order.id}: Your order has been confirmed!',
    user=order.user,
    order=order
)
```

**After:**
```python
order.status = 'confirmed'
order.save()
# Signal automatically creates user notification
```

#### Example 3: User Cancellation (KEEP)

**Before:**
```python
order.status = 'cancelled'
order.save()

# KEEP THIS - User-initiated cancellation needs admin notification
Notification.objects.create(
    type='order_cancelled',
    message=f'Order #{order.id}: {user.username} cancelled their order.',
    user=None,  # Notify admin
    order=order
)
```

**After:**
```python
order.status = 'cancelled'
order.save()

# Signal creates user notification automatically
# But we also want to notify admin about user-initiated cancellation
create_admin_notification(
    notification_type='order_cancelled',
    message=f'Order #{order.id}: {user.username} cancelled their order.',
    order=order
)
```

---

## Specific Files to Update

### 1. `views.py` - checkout_cart function

**Lines to Remove:** ~1557-1577

**Current Code:**
```python
order = Order.objects.create(...)

# REMOVE: Signal handles this
if is_delivery:
    Notification.objects.create(
        type='payment_pending',
        message=f'Order #{order.id}: Your downpayment has been submitted...',
        user=request.user,
        order=order
    )
else:
    Notification.objects.create(
        type='order_submitted',
        message=f'Order #{order.id}: Your pickup request has been received!',
        user=request.user,
        order=order
    )
    
    # REMOVE: Signal handles this
    Notification.objects.create(
        type='order_submitted',
        message=f'Order #{order.id}: {customer_name}\'s pickup request...',
        user=None,
        order=order
    )
```

**New Code:**
```python
order = Order.objects.create(...)
# Signal automatically creates admin notification
```

---

### 2. `views.py` - confirm_order_receipt function

**Lines to Keep:** ~1660-1664

**Current Code:**
```python
# KEEP: User-initiated action, notify admin
Notification.objects.create(
    type='order_completed',
    message=f'Order #{order.id}: {request.user.username} confirmed receipt...',
    user=None,
    order=order
)
```

**Reason:** This is a user-initiated action that admins need to know about, not a status change.

---

### 3. `views.py` - cancel_reservation function

**Lines to Keep:** ~1702-1706

**Current Code:**
```python
# KEEP: User-initiated cancellation, notify admin
Notification.objects.create(
    type='reservation_cancelled',
    message=f'Reservation #{reservation.id}: {user} cancelled their reservation...',
    user=None,
    reservation=reservation
)
```

**Reason:** User-initiated cancellation needs admin notification.

---

### 4. `views.py` - cancel_order function

**Lines to Keep:** ~1736-1740

**Current Code:**
```python
# KEEP: User-initiated cancellation, notify admin
Notification.objects.create(
    type='order_cancelled',
    message=f'Order #{order.id}: {user} cancelled their order.',
    user=None,
    order=order
)
```

**Reason:** User-initiated cancellation needs admin notification.

---

### 5. `views.py` - admin_update_order_status function

**Lines to Remove:** ~1848-1852, 1868-1872, 1884-1887, 1898-1901, 1912-1915

**Current Code:**
```python
# REMOVE ALL: Signal handles status change notifications
order.status = 'confirmed'
order.save()

Notification.objects.create(
    type='order_confirmed',
    message=f'Order #{order.id}: Your order has been confirmed!',
    user=order.user,
    order=order
)
```

**New Code:**
```python
order.status = 'confirmed'
order.save()
# Signal automatically creates user notification
```

---

### 6. `reservation_views.py` - submit_reservation function

**Lines to Remove:** ~166-179

**Current Code:**
```python
reservation = Reservation.objects.create(...)

# REMOVE: Signal handles this
Notification.objects.create(
    type='reservation_submitted',
    message=f'Reservation #{reservation.id}: Your reservation...',
    user=request.user,
    reservation=reservation
)

# REMOVE: Signal handles this
Notification.objects.create(
    type='reservation_submitted',
    message=f'Reservation #{reservation.id}: {customer_name}\'s reservation...',
    user=None,
    reservation=reservation
)
```

**New Code:**
```python
reservation = Reservation.objects.create(...)
# Signal automatically creates admin notification
```

---

### 7. `reservation_views.py` - reservation_payment function

**Lines to Remove:** ~229-242

**Current Code:**
```python
reservation.status = 'confirmed'
reservation.save()

# REMOVE: Signal handles this
Notification.objects.create(
    type='reservation_payment_submitted',
    message=f'Reservation #{reservation.id}: Payment submitted...',
    user=request.user,
    reservation=reservation
)

# REMOVE: Signal handles this
Notification.objects.create(
    type='reservation_payment_submitted',
    message=f'Reservation #{reservation.id}: {customer_name} has submitted payment...',
    user=None,
    reservation=reservation
)
```

**New Code:**
```python
reservation.status = 'confirmed'
reservation.save()
# Signal automatically creates user notification
```

---

### 8. `reservation_views.py` - confirm_reservation function

**Lines to Remove:** ~560-564

**Current Code:**
```python
reservation.status = 'confirmed'
reservation.save()

# REMOVE: Signal handles this
Notification.objects.create(
    type='reservation_confirmed',
    message=f'Reservation #{reservation.id}: Your reservation... has been confirmed!',
    user=reservation.user,
    reservation=reservation
)
```

**New Code:**
```python
reservation.status = 'confirmed'
reservation.save()
# Signal automatically creates user notification
```

---

### 9. `reservation_views.py` - reject_reservation function

**Lines to Remove:** ~595-599

**Current Code:**
```python
reservation.status = 'rejected'
reservation.save()

# REMOVE: Signal handles this
Notification.objects.create(
    type='reservation_rejected',
    message=f'Reservation #{reservation.id}: Your reservation was rejected...',
    user=reservation.user,
    reservation=reservation
)
```

**New Code:**
```python
reservation.status = 'rejected'
reservation.save()
# Signal automatically creates user notification
```

---

### 10. `reservation_views.py` - complete_reservation function

**Lines to Remove:** ~623-627

**Current Code:**
```python
reservation.status = 'completed'
reservation.save()

# REMOVE: Signal handles this
Notification.objects.create(
    type='reservation_completed',
    message=f'Reservation #{reservation.id}: Your reservation has been completed...',
    user=reservation.user,
    reservation=reservation
)
```

**New Code:**
```python
reservation.status = 'completed'
reservation.save()
# Signal automatically creates user notification
```

---

## Testing After Migration

### Test 1: New Order
1. User places order
2. Check admin navbar - should see notification
3. Check user navbar - should NOT see this notification
4. Verify no duplicate notifications in database

### Test 2: Admin Confirms Order
1. Admin changes order status to "confirmed"
2. Check user navbar - should see notification
3. Check admin navbar - should NOT see this notification
4. Verify no duplicate notifications in database

### Test 3: User Cancels Order
1. User cancels their order
2. Check admin navbar - should see notification (user-initiated)
3. Check user navbar - should see notification (status change)
4. Verify both notifications exist (this is correct)

### Test 4: New Reservation
1. User submits reservation
2. Check admin navbar - should see notification
3. Check user navbar - should NOT see this notification
4. Verify no duplicate notifications in database

### Test 5: Admin Rejects Reservation
1. Admin changes reservation status to "rejected"
2. Check user navbar - should see notification
3. Check admin navbar - should NOT see this notification
4. Verify no duplicate notifications in database

---

## Rollback Plan

If issues occur, you can temporarily disable signals:

```python
# In signals.py, comment out the @receiver decorators:

# @receiver(post_save, sender=Order)  # DISABLED
def create_order_notifications(sender, instance, created, **kwargs):
    pass  # Temporarily disabled
```

Then restore manual notification creation in views.

---

## Benefits of Signal-Based System

1. **Consistency:** All notifications follow same pattern
2. **Maintainability:** One place to update notification logic
3. **No Duplicates:** Automatic creation prevents manual errors
4. **Separation:** Clear distinction between admin and user notifications
5. **Automatic:** No need to remember to create notifications in views

---

## Summary

**Remove manual notifications for:**
- Order creation
- Reservation creation
- Status changes (confirmed, rejected, completed, etc.)

**Keep manual notifications for:**
- User-initiated cancellations (notify admin)
- User confirms receipt (notify admin)
- Special cases not tied to status changes

**After migration:**
- Signals handle 90% of notifications automatically
- Views only create notifications for special user-initiated actions
- No more duplicate notifications
- Clear separation between admin and user notifications
