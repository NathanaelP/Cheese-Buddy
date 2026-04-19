# Cheese Buddy — Project Plan

A PWA-based cheese expiration tracker for deli use. Inspired by Bread Buddy V2 but built around manufacturer expiration dates rather than thaw logic. Includes push notifications via Firebase when a product expires.

---

## Tech Stack

- **Frontend:** Single-file HTML/CSS/JS (PWA)
- **Storage:** Firebase Firestore (replaces localStorage for cross-device sync)
- **Notifications:** Firebase Cloud Messaging (FCM)
- **Backend Logic:** Firebase Cloud Functions (scheduled daily cron)
- **Hosting:** GitHub Pages or Firebase Hosting

---

## Data Design

Two separate stores in localStorage (Phase 1), migrated to Firestore in Phase 2.

### `cheeseProducts`
The master product name list. Powers the searchable dropdown.

| Field | Type | Notes |
|---|---|---|
| `id` | Number | Unique identifier |
| `name` | String | Product name (e.g. "Boar's Head Gouda") |

- Starts with a default list of known Publix deli cheeses
- Grows permanently as new names are added by the user
- Names are **never deleted** — removing a tracking card does not remove the product name
- No dates stored here

### `cheeseCards`
The active expiration tracking cards. One card per package being tracked.

| Field | Type | Notes |
|---|---|---|
| `id` | Number | Unique identifier (uses `Date.now()`) |
| `name` | String | Product name (matched from cheeseProducts or new) |
| `date` | String | Manufacturer expiration date (YYYY-MM-DD) |

- Multiple cards with the same name are fully supported
- Example: three boxes of Boar's Head Swiss with three different dates = three separate cards
- Deleting a card removes it from tracking but does NOT affect `cheeseProducts`
- This is the data that drives the color-coded status display

---

## Workflow

1. User taps the name field → searchable dropdown appears showing all products in `cheeseProducts`
2. User types to filter the list, or types a brand new name not yet in the list
3. User selects or confirms the name, enters the expiration date, taps Add
4. A new card is created in `cheeseCards`
5. If the name was new (not already in `cheeseProducts`), it is automatically added there too
6. The card appears in the list immediately with its color-coded status

---

## Color-Coded Status System

| Status | Condition | Card Color |
|---|---|---|
| Green | 4+ days until expiration | Green background, green left border |
| Yellow | 1–3 days until expiration | Yellow/orange background, orange left border |
| Red | Expired (0 or fewer days) | Red background, red left border |

---

## UI Features

### Filter Buttons
Three single-tap filter buttons above the item list:
- **All** — shows everything, sorted soonest expiry first
- **Expiring Soon** — shows only yellow cards
- **Expired** — shows only red cards

Tapping a filter hides all cards not in that category. Cards with no date set are shown only in the All view.

### Searchable Dropdown
- Appears when user taps the product name input
- Filters in real time as the user types
- Shows all names from `cheeseProducts`
- If the typed name does not match any existing product, it is treated as a new product and added to `cheeseProducts` on save
- Selecting from the dropdown fills the name field and moves focus to the date field

### Add Item Form
- Product name (text input with searchable dropdown)
- Expiration date (date picker)
- "Add to Tracker" button

### Item Cards
- Product name (bold)
- Expiration date (formatted, e.g. "Aug 25, 2026")
- Days remaining badge (e.g. "12 days left", "Expires tomorrow", "Expired 2d ago")
- Cards with no date show a "No date set" badge in neutral gray
- Delete button (trash icon) per card — removes card only, not product name

### Search Bar
- Filters visible cards in real time as user types
- ✕ button to clear search appears when search field has text

### JSON Export / Import
- Export button saves full `cheeseCards` list as a `.json` file
- Import button accepts a `.json` file and restores the cards list
- `cheeseProducts` is exported and imported separately to preserve the master name list

---

## Default Product List

Pre-loaded into `cheeseProducts` on first launch. Based on known Publix deli inventory:

**Publix Brand**
- Yellow American, White American, Swiss, Provolone, Muenster, Havarti, Gouda, Aged Cheddar

**Boar's Head**
- Baby Swiss, Lacey Swiss, Mild Swiss
- Provolone, Picante Provolone
- Yellow American, White American
- Smoked Beechwood Wisconsin Cheddar
- Muenster
- Cream Havarti, Havarti Jalapeño
- Smoked Gouda, Chipotle Gouda
- Caramelized Onion Monterey Jack

**Specialty / Imported**
- Manchego, Gruyère, Brie, Blue Cheese

*User can add any additional cheeses carried at their specific store via the add form.*

---

## Push Notification System

### Goal
A "set it and forget it" alert that fires once daily. No user action required after initial setup.

### Trigger
Notifications fire **only when a product has expired** (expiration date is today or earlier). No notifications for items that are merely expiring soon.

### Flow
1. User opens Cheese Buddy and grants notification permission
2. App registers device with Firebase Cloud Messaging (FCM)
3. FCM token is saved to Firestore under the user's device record
4. A Firebase Cloud Function runs once per day (scheduled cron, e.g. 6:00 AM)
5. Function queries Firestore for any cards where `date <= today`
6. If any are found, it sends a push notification via FCM listing the expired items
7. Notification appears on the device lock screen regardless of whether the app is open

### Notification Content (example)
> **Cheese Buddy Alert**
> The following items have expired: Boar's Head Gouda, Swiss Lorraine. Open the app to review.

### Platform Notes
- **Android:** Push works as long as the app has been opened at least once and notification permission granted
- **iOS:** App must be installed as a PWA (Add to Home Screen) for push notifications to work

### Cost
Firebase free tier (Spark plan) covers this use case comfortably:
- Firestore reads/writes are well within free limits for a small item list
- Cloud Functions free tier includes 2 million invocations/month (one daily call uses nearly none of this)
- FCM is free with no message limits

---

## PWA Requirements

- `manifest.json` — app name, icons, theme color
- `service-worker.js` — offline caching
- Installable on Android and iOS home screens

---

## Build Phases

### Phase 1 — Frontend (standalone)
- Single HTML/CSS/JS file
- localStorage for both `cheeseProducts` and `cheeseCards`
- Full UI: searchable dropdown, add form, delete cards, filter buttons, color-coded cards, JSON export/import

### Phase 2 — Firebase Integration
- Set up Firebase project (free Spark plan)
- Migrate `cheeseCards` from localStorage to Firestore
- Migrate `cheeseProducts` from localStorage to Firestore
- Add FCM registration and permission request on app open

### Phase 3 — Cloud Function
- Write and deploy scheduled Cloud Function
- Query expired cards daily
- Send FCM push notification if any expired cards found

### Phase 4 — Polish
- PWA manifest and service worker
- iOS home screen compatibility
- Test notifications on Android and iOS

---

## Reference

Based on: [Bread Buddy V2](https://github.com/NathanaelP/Bread-Buddy-V2)
Live reference: https://nathanaelp.github.io/Bread-Buddy-V2/
