# Reservation Cart - Color Theme Update

## Change Summary
Changed the reservation cart color scheme from purple/violet to green to match the overall application theme.

---

## Changes Made to `unified_cart.html`

### 1. Select All Checkbox (Line 410)
**Before:**
```html
<input type="checkbox" id="selectAllReservations" class="w-5 h-5 text-purple-600 rounded focus:ring-purple-500" onchange="toggleAllReservations(this)">
```

**After:**
```html
<input type="checkbox" id="selectAllReservations" class="w-5 h-5 text-green-600 rounded focus:ring-green-500" onchange="toggleAllReservations(this)">
```

**Changes:**
- `text-purple-600` → `text-green-600`
- `focus:ring-purple-500` → `focus:ring-green-500`

---

### 2. Individual Item Checkboxes (Line 417)
**Before:**
```html
<input type="checkbox" class="reservation-checkbox w-5 h-5 text-purple-600 rounded focus:ring-purple-500 flex-shrink-0" data-item-id="{{ item.id }}" data-price="{{ item.get_subtotal }}" onchange="updateReservationSelection()">
```

**After:**
```html
<input type="checkbox" class="reservation-checkbox w-5 h-5 text-green-600 rounded focus:ring-green-500 flex-shrink-0" data-item-id="{{ item.id }}" data-price="{{ item.get_subtotal }}" onchange="updateReservationSelection()">
```

**Changes:**
- `text-purple-600` → `text-green-600`
- `focus:ring-purple-500` → `focus:ring-green-500`

---

### 3. Item Container Border (Line 420)
**Before:**
```html
<div class="bg-white rounded-xl shadow-md p-3 md:p-6 border-l-4 border-purple-600 reservation-item flex-1" data-item-id="{{ item.id }}" data-price="{{ item.get_subtotal }}">
```

**After:**
```html
<div class="bg-white rounded-xl shadow-md p-3 md:p-6 border-l-4 border-green-600 reservation-item flex-1" data-item-id="{{ item.id }}" data-price="{{ item.get_subtotal }}">
```

**Changes:**
- `border-purple-600` → `border-green-600`

---

## Visual Changes

### Before:
- Purple checkboxes (when checked)
- Purple focus ring on checkboxes
- Purple left border on cart items

### After:
- Green checkboxes (when checked) ✅
- Green focus ring on checkboxes ✅
- Green left border on cart items ✅

---

## Consistency

This change ensures the reservation cart matches the application's primary color scheme:
- **Primary Green**: `#16a34a` (green-600)
- **Secondary Green**: `#15803d` (green-700)

The green color is already used throughout the application:
- Navigation bars
- Buttons (Add to Cart, Update, etc.)
- Price displays
- Icons and badges
- Order cart items

---

## Files Modified

1. `kakanin/templates/kakanin/unified_cart.html`
   - Line 410: Select All checkbox
   - Line 417: Individual item checkboxes
   - Line 420: Item container border

---

## Testing Checklist

✅ Select All checkbox displays green when checked  
✅ Individual item checkboxes display green when checked  
✅ Checkbox focus rings are green  
✅ Cart item left borders are green  
✅ Color scheme is consistent with rest of application  
✅ No purple/violet colors remain in reservation cart

---

## Notes

- The change affects only the reservation cart section
- Order cart items already use green colors
- All interactive elements now have consistent green theming
- Tailwind CSS classes used: `text-green-600`, `focus:ring-green-500`, `border-green-600`
