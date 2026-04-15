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

## Data Model

Each item has two fields:

| Field | Type | Notes |
|---|---|---|
| `name` | String | Product name (e.g. "Boar's Head Gouda") |
| `expirationDate` | Date | Manufacturer expiration date from packaging |

No quantity, no vendor, no other fields needed.

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

Tapping a filter hides all cards not in that category.

### Add Item Form
- Product name (text input)
- Expiration date (date picker, defaults to today)
- "Add" button

### Item Cards
- Product name (bold)
- Expiration date (formatted, e.g. "Aug 25, 2026")
- Days remaining badge (e.g. "12 days left", "Expires tomorrow", "Expired 2d ago")
- Delete button (trash icon) per card

### JSON Export / Import
- Export button saves full item list as a `.json` file
- Import button accepts a `.json` file and restores the list
- Useful for backup and transferring between devices

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
5. Function queries Firestore for any items where `expirationDate <= today`
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
- LocalStorage for data (temporary, before Firestore)
- Full UI: add, delete, filter buttons, color-coded cards, JSON export/import

### Phase 2 — Firebase Integration
- Set up Firebase project (free Spark plan)
- Migrate storage from localStorage to Firestore
- Add FCM registration and permission request on app open

### Phase 3 — Cloud Function
- Write and deploy scheduled Cloud Function
- Query expired items daily
- Send FCM push notification if any expired items found

### Phase 4 — Polish
- PWA manifest and service worker
- iOS home screen compatibility
- Test notifications on Android and iOS

---

## Reference

Based on: [Bread Buddy V2](https://github.com/NathanaelP/Bread-Buddy-V2)
Live reference: https://nathanaelp.github.io/Bread-Buddy-V2/
