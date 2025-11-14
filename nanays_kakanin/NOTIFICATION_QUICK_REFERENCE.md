# Notification System - Quick Reference

## 🎯 Core Principle

```
user=None     → Admin notification (shown in admin navbar)
user=<User>   → User notification (shown in user's navbar)
```

---

## 📋 Automatic Notifications (via Signals)

### When Order/Reservation is Created:
✅ **Admin** gets notified automatically  
❌ **User** does NOT get notified

### When Status Changes:
✅ **User** gets notified automatically  
❌ **Admin** does NOT get notified

---

## 🔧 Manual Notification Creation

### Only create manually for:
- User-initiated cancellations (notify admin)
- User confirms receipt (notify admin)
- Special cases not tied to status changes

### Use utility functions:

```python
from kakanin.notification_utils import create_admin_notification, create_user_notification

# Notify admin
create_admin_notification(
    notification_type='order_cancelled',
    message='User cancelled their order',
    order=order
)

# Notify user
create_user_notification(
    user=user,
    notification_type='custom_notification',
    message='Your custom message',
    order=order
)
```

---

## 📊 Context Variables

### Admin Navbar:
```django
{{ admin_unread_notifications_count }}
{{ admin_recent_notifications }}
```

### User Navbar:
```django
{{ unread_notifications_count }}
{{ recent_notifications }}
```

---

## 🔍 Database Queries

### Get Admin Notifications:
```python
Notification.objects.filter(user__isnull=True)
```

### Get User Notifications:
```python
Notification.objects.filter(user=request.user)
```

---

## ✅ Checklist for New Features

When adding new order/reservation features:

- [ ] Does it create an Order/Reservation? → Signal handles notification
- [ ] Does it change status? → Signal handles notification
- [ ] Is it user-initiated action admin needs to know? → Create manual admin notification
- [ ] Is it special case? → Create manual notification with utility function

---

## 🚫 Common Mistakes to Avoid

❌ Creating notification manually when creating Order/Reservation  
❌ Creating notification manually when changing status  
❌ Setting `user=request.user` for admin notifications  
❌ Setting `user=None` for user notifications  
❌ Mixing admin and user notifications in same query  
❌ Forgetting to link notification to order/reservation  

---

## 🧪 Quick Test

```python
# Test admin notification
admin_notifs = Notification.objects.filter(user__isnull=True)
print(f"Admin has {admin_notifs.count()} notifications")

# Test user notification
user_notifs = Notification.objects.filter(user=some_user)
print(f"User has {user_notifs.count()} notifications")

# They should NEVER overlap!
```

---

## 📞 Troubleshooting

**Problem:** Notifications appear for both admin and user  
**Fix:** Check `user` field is set correctly

**Problem:** Duplicate notifications  
**Fix:** Remove manual creation, let signals handle it

**Problem:** Count doesn't update  
**Fix:** Mark notifications as read after viewing

---

## 🎓 Learn More

See full documentation:
- `NOTIFICATION_SYSTEM_GUIDE.md` - Complete guide
- `NOTIFICATION_MIGRATION_GUIDE.md` - Migration from manual to signals
- `kakanin/signals.py` - Signal implementation
- `kakanin/notification_utils.py` - Utility functions
