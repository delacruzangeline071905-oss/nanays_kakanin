# Notification System - Complete Guide

## Overview
The notification system separates admin and user notifications to ensure each party only sees relevant notifications in their navbar.

---

## System Architecture

### 1. Notification Model
Located in `kakanin/models.py`:

```python
class Notification(models.Model):
    type = models.CharField(max_length=32, choices=TYPE_CHOICES)
    message = models.TextField()
    user = models.ForeignKey(User, null=True, blank=True)  # NULL = Admin notification
    order = models.ForeignKey('Order', null=True, blank=True)
    reservation = models.ForeignKey('Reservation', null=True, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    read = models.BooleanField(default=False)
```

**Key Field:**
- `user=None` → Admin notification (shown in admin navbar)
- `user=<User>` → User notification (shown in that user's navbar)

---

### 2. Automatic Notification Creation (Signals)
Located in `kakanin/signals.py`:

#### How It Works:
Signals automatically create notifications when Order or Reservation status changes:

**New Order/Reservation:**
- **Trigger:** When created
- **Notifies:** Admin only (`user=None`)
- **Message:** "New order/reservation from [username]"

**Status Changes:**
- **Trigger:** When status field changes
- **Notifies:** User only (`user=<User>`)
- **Message:** Based on new status (confirmed, rejected, ready, etc.)

#### Supported Status Changes:

**Orders:**
- `confirmed` → "Your order has been confirmed!"
- `ready_for_pickup` → "Your order is ready for pickup."
- `out_for_delivery` → "Your order is out for delivery."
- `completed` → "Your order has been completed. Thank you!"
- `cancelled` → "Your order has been cancelled."
- `rejected` → "Your order payment was rejected."

**Reservations:**
- `confirmed` → "Your reservation has been confirmed! Please proceed to payment."
- `rejected` → "Your reservation was rejected."
- `completed` → "Your reservation has been completed. Thank you!"
- `cancelled` → "Your reservation has been cancelled."

---

### 3. Context Processor
Located in `kakanin/context_processors.py`:

Automatically adds notification data to all templates:

#### For Admin Users (`is_staff=True`):
```python
admin_unread_notifications_count  # Count of unread admin notifications
admin_recent_notifications        # Last 5 unread admin notifications
```

#### For Regular Users:
```python
unread_notifications_count  # Count of unread user notifications
recent_notifications        # Last 5 unread user notifications
```

**Filtering Logic:**
- Admin: `Notification.objects.filter(user__isnull=True)`
- User: `Notification.objects.filter(user=request.user)`

---

### 4. Navbar Display

#### Admin Navbar
Shows only admin notifications (`user=None`):
```django
{% if admin_unread_notifications_count > 0 %}
  <span class="badge">{{ admin_unread_notifications_count }}</span>
{% endif %}

{% for notification in admin_recent_notifications %}
  {{ notification.message }}
{% endfor %}
```

#### User Navbar
Shows only user's own notifications:
```django
{% if unread_notifications_count > 0 %}
  <span class="badge">{{ unread_notifications_count }}</span>
{% endif %}

{% for notification in recent_notifications %}
  {{ notification.message }}
{% endfor %}
```

---

## Usage Examples

### Example 1: User Places an Order

**What Happens:**
1. User submits order form
2. Order is created in database
3. Signal `post_save` fires
4. Signal creates admin notification:
   ```python
   Notification.objects.create(
       type='order_submitted',
       message='Order #123: John Doe placed a pickup order.',
       user=None,  # Admin notification
       order=order
   )
   ```
5. Admin sees notification in navbar
6. User does NOT see this notification

---

### Example 2: Admin Confirms Order

**What Happens:**
1. Admin changes order status to "confirmed"
2. Order is saved
3. Signal `post_save` fires
4. Signal detects status change
5. Signal creates user notification:
   ```python
   Notification.objects.create(
       type='order_confirmed',
       message='Order #123: Your order has been confirmed!',
       user=order.user,  # User notification
       order=order
   )
   ```
6. User sees notification in navbar
7. Admin does NOT see this notification

---

### Example 3: User Places Reservation

**What Happens:**
1. User submits reservation form
2. Reservation is created
3. Signal creates admin notification:
   ```python
   Notification.objects.create(
       type='reservation_submitted',
       message='Reservation #45: Jane Smith submitted a reservation for Suman on 2025-11-01.',
       user=None,  # Admin notification
       reservation=reservation
   )
   ```
4. Admin sees notification
5. User does NOT see this notification

---

### Example 4: Admin Rejects Reservation

**What Happens:**
1. Admin changes reservation status to "rejected"
2. Reservation is saved
3. Signal creates user notification:
   ```python
   Notification.objects.create(
       type='reservation_rejected',
       message='Reservation #45: Your reservation was rejected.',
       user=reservation.user,  # User notification
       reservation=reservation
   )
   ```
4. User sees notification
5. Admin does NOT see this notification

---

## Utility Functions

Located in `kakanin/notification_utils.py`:

### Create Admin Notification
```python
from kakanin.notification_utils import create_admin_notification

create_admin_notification(
    notification_type='order_submitted',
    message='New order from customer',
    order=order
)
```

### Create User Notification
```python
from kakanin.notification_utils import create_user_notification

create_user_notification(
    user=user,
    notification_type='order_confirmed',
    message='Your order has been confirmed',
    order=order
)
```

### Mark as Read
```python
from kakanin.notification_utils import mark_notification_as_read

mark_notification_as_read(notification_id, user)
```

### Mark All as Read
```python
from kakanin.notification_utils import mark_all_notifications_as_read

mark_all_notifications_as_read(user)
```

---

## Views

### User Notifications
**URL:** `/notifications/`
**View:** `user_notifications`
**Template:** `notifications.html`
**Shows:** Only notifications for logged-in user

### Admin Notifications
**URL:** `/admin/notifications/`
**View:** `admin_notifications`
**Template:** `admin_notifications.html`
**Shows:** Only admin notifications (`user=None`)

### Mark Notification as Read
**User:** `/notifications/mark-read/<id>/`
**Admin:** `/admin/notifications/mark-read/<id>/`

---

## Important Rules

### ✅ DO:
1. **Use signals** for automatic notification creation
2. **Set `user=None`** for admin notifications
3. **Set `user=<User>`** for user notifications
4. **Filter by `user__isnull=True`** for admin notifications
5. **Filter by `user=request.user`** for user notifications

### ❌ DON'T:
1. **Don't create duplicate notifications** in views (signals handle it)
2. **Don't mix admin and user notifications** in the same query
3. **Don't show admin notifications to users** or vice versa
4. **Don't forget to mark notifications as read** when viewed
5. **Don't create notifications without linking** to order/reservation

---

## Testing Checklist

### User Places Order:
- [ ] Admin sees notification in navbar
- [ ] Admin notification count increases
- [ ] User does NOT see this notification
- [ ] Notification links to order detail

### Admin Confirms Order:
- [ ] User sees notification in navbar
- [ ] User notification count increases
- [ ] Admin does NOT see this notification
- [ ] Notification links to order detail

### User Places Reservation:
- [ ] Admin sees notification in navbar
- [ ] Admin notification count increases
- [ ] User does NOT see this notification
- [ ] Notification links to reservation detail

### Admin Rejects Reservation:
- [ ] User sees notification in navbar
- [ ] User notification count increases
- [ ] Admin does NOT see this notification
- [ ] Notification links to reservation detail

### Mark as Read:
- [ ] Clicking notification marks it as read
- [ ] Count decreases after marking as read
- [ ] Read notifications don't show in navbar dropdown
- [ ] "Mark all as read" button works

---

## Troubleshooting

### Problem: Notifications appear for both admin and user
**Solution:** Check the `user` field:
- Admin notifications must have `user=None`
- User notifications must have `user=<specific_user>`

### Problem: Duplicate notifications
**Solution:** 
- Remove manual `Notification.objects.create()` from views
- Let signals handle notification creation automatically

### Problem: Notification count doesn't update
**Solution:**
- Check context processor is registered in settings
- Ensure template uses correct variable names
- Verify notifications are being marked as read

### Problem: Wrong user sees notification
**Solution:**
- Check filtering in context processor
- Verify `user` field is set correctly
- Ensure navbar template uses correct variables

---

## Database Cleanup

To prevent database bloat, periodically delete old read notifications:

```python
from kakanin.notification_utils import delete_old_notifications

# Delete read notifications older than 30 days
deleted_count = delete_old_notifications(days=30)
```

Consider adding this to a scheduled task (cron job or Celery beat).

---

## Files Modified/Created

1. **Created:** `kakanin/signals.py` - Automatic notification creation
2. **Created:** `kakanin/notification_utils.py` - Utility functions
3. **Modified:** `kakanin/apps.py` - Register signals
4. **Existing:** `kakanin/models.py` - Notification model (already correct)
5. **Existing:** `kakanin/context_processors.py` - Already correctly separates notifications
6. **Existing:** `kakanin/views.py` - Notification views (already correct)

---

## Summary

The notification system now:
✅ Automatically creates notifications via signals
✅ Separates admin and user notifications
✅ Shows correct count in navbar for each user type
✅ Prevents duplicate notifications
✅ Links notifications to related orders/reservations
✅ Supports marking as read
✅ Updates navbar count in real-time

**Key Principle:** 
- `user=None` = Admin notification
- `user=<User>` = User notification
- Never mix them!
