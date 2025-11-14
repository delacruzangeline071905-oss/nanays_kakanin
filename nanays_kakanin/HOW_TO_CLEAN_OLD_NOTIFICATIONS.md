# How to Clean Old Notifications

## Problem
Old notifications created before the signal system are still showing to users. These need to be cleaned up.

---

## Solution: Clean Up Old Notifications

You have **3 options** to clean up old notifications:

---

### Option 1: Django Management Command (Recommended)

#### View Statistics:
```bash
python manage.py clean_notifications
```

This shows:
- Total notifications
- Admin vs User notifications
- Read vs Unread

#### Delete All Notifications:
```bash
python manage.py clean_notifications --all
```

#### Delete Only Read Notifications:
```bash
python manage.py clean_notifications --read
```

#### Delete Old Notifications (e.g., older than 30 days):
```bash
python manage.py clean_notifications --old 30
```

---

### Option 2: Python Script

Run the cleanup script:
```bash
python clean_old_notifications.py
```

This will:
1. Show count of existing notifications
2. Ask for confirmation
3. Delete all notifications if confirmed

---

### Option 3: Django Shell (Manual)

```bash
python manage.py shell
```

Then run:
```python
from kakanin.models import Notification

# View counts
print(f"Total: {Notification.objects.count()}")
print(f"Admin: {Notification.objects.filter(user__isnull=True).count()}")
print(f"User: {Notification.objects.filter(user__isnull=False).count()}")

# Delete all notifications
Notification.objects.all().delete()

# Or delete only read notifications
Notification.objects.filter(read=True).delete()

# Or delete specific user's notifications
from django.contrib.auth.models import User
user = User.objects.get(username='username')
Notification.objects.filter(user=user).delete()
```

---

## After Cleanup

Once old notifications are deleted:

1. **New orders** will create admin notifications (via signal)
2. **Status changes** will create user notifications (via signal)
3. **No duplicates** will be created
4. **Counts will be accurate** in navbar

---

## Recommended Approach

**For Production:**
```bash
# 1. View statistics first
python manage.py clean_notifications

# 2. Delete only read notifications (safer)
python manage.py clean_notifications --read

# 3. Test that new notifications work
# Place a test order and check admin navbar
```

**For Development:**
```bash
# Delete all and start fresh
python manage.py clean_notifications --all
```

---

## Preventing Future Issues

### Automatic Cleanup (Optional)

Add to your cron job or scheduled tasks:
```bash
# Delete read notifications older than 30 days
python manage.py clean_notifications --old 30
```

Or use Django's periodic tasks:
```python
# In your tasks.py or celery beat
from kakanin.notification_utils import delete_old_notifications

# Run daily
delete_old_notifications(days=30)
```

---

## Verification

After cleanup, verify:

1. **Check navbar counts:**
   - Admin navbar should show 0 (if no new orders)
   - User navbar should show 0 (if no status changes)

2. **Place test order:**
   - Admin should see 1 notification
   - User should NOT see notification

3. **Confirm order:**
   - User should see 1 notification
   - Admin should NOT see notification

4. **Check database:**
   ```python
   from kakanin.models import Notification
   print(Notification.objects.count())  # Should be 2 (1 admin, 1 user)
   ```

---

## Troubleshooting

### Issue: Command not found
```bash
# Make sure you're in the project directory
cd c:\Users\Admin\Desktop\no_orreser - Copy\nanays_kakanin

# Try with python path
python manage.py clean_notifications
```

### Issue: Permission denied
```bash
# Run as administrator (Windows)
# Or use sudo (Linux/Mac)
```

### Issue: Database locked
```bash
# Stop the development server first
# Then run the cleanup command
```

---

## Quick Reference

| Command | What it does |
|---------|-------------|
| `python manage.py clean_notifications` | Show statistics |
| `python manage.py clean_notifications --all` | Delete all |
| `python manage.py clean_notifications --read` | Delete read only |
| `python manage.py clean_notifications --old 30` | Delete older than 30 days |
| `python clean_old_notifications.py` | Run cleanup script |

---

## Summary

**To fix the issue in your screenshot:**

1. Run: `python manage.py clean_notifications --all`
2. Confirm: `yes`
3. Refresh the page
4. Old notifications should be gone
5. New notifications will be created by signals

**That's it!** 🎉
