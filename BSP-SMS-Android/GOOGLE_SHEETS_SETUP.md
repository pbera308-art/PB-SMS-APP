# Google Sheet + Google Apps Script Setup (Free / Serverless)

## 1. Create or Upgrade the Google Sheet
1. Open [Google Sheets](https://sheets.google.com) and create a new spreadsheet named **`BSP-SMS Users`** (or open your existing **`BSP-SMS Users`** sheet).
2. Rename the bottom sheet tab from `Sheet1` to **`Users`** (or let `Code.gs` auto-create it on first request).
3. Required base columns in **Row 1 (`A1:I1`)** — and `Code.gs` automatically extends **`J1:Q1`** non-destructively without touching existing user rows:
   - `A1`: `user_id`
   - `B1`: `phone_number`
   - `C1`: `password` (stored as salted SHA-256 `sha256s:<salt>:<hash>`; never returned to Admin or clients)
   - `D1`: `status` (`inactive` or `active`)
   - `E1`: `expiry_date` (`YYYY-MM-DD`)
   - `F1`: `active_device_id`
   - `G1`: `last_login`
   - `H1`: `last_activity`
   - `I1`: `created_at`
   - `J1`: `full_name`
   - `K1`: `username`
   - `L1`: `role` (`general`)
   - `M1`: `app_logged_in` (`true` / `false`)
   - `N1`: `session_token`
   - `O1`: `portal_session_active` (`true` / `false`)
   - `P1`: `portal_reset_version` (integer counter incremented on Force Portal Logout / Reset Portal Session)
   - `Q1`: `portal_last_updated`

## 2. Deploy or Update Google Apps Script
1. Inside the Google Sheet, click **Extensions → Apps Script**.
2. Replace the contents of `Code.gs` with the full contents of `google-apps-script/Code.gs`.
3. Click **Save** (`Ctrl+S`).
4. For a new deployment: click **Deploy → New deployment** → **Web app** (**Execute as:** `Me`, **Who has access:** `Anyone`) → **Deploy** and copy the `/exec` URL.
   For an existing deployment: click **Deploy → Manage deployments → Edit (pencil) → Version: New version → Deploy** so the same `/exec` URL runs the updated script.

## 3. Connect the URL to the Android App
The Android project is pre-configured with your deployed Web App `/exec` URL (`DEFAULT_APPS_SCRIPT_URL` in `AuthRepository.kt` and `APPS_SCRIPT_URL` in `BSP-SMS-Android/app/build.gradle.kts`), so **Register** and **General Login** work immediately on new installs. You can still override it anytime in **either** of two ways:
- **Option A (Built into APK):** Update `APPS_SCRIPT_URL` in `BSP-SMS-Android/app/build.gradle.kts` (and `DEFAULT_APPS_SCRIPT_URL` in `AuthRepository.kt`), then rebuild the APK via GitHub Actions.
- **Option B (No rebuild needed):** Open **Admin Login** (`ID: 9679843774`, `Pass: 9933115114`), open the left drawer → **Admin User Monitor**, tap the link icon to view or update the **Apps Script Web App URL**, and tap **Save URL & Load Users**.

## 4. Admin Controls & Per-User Portal Isolation
- **Admin Dashboard & User Management:** Shows **Total Users**, **Active**, **Inactive**, **Logged In** (from backend `app_logged_in`), and **Portal Active** counts, with Search and Filters (`All`, `Active`, `Inactive`, `Logged In`, `Expired`).
- **Individual Account Controls:** Tap **Activate 30d / Reactivate 30d**, **Deactivate** (immediately revokes the user's app session token on backend without affecting other users), **Edit** (Name, Username, Phone), **Password** (securely reset app password without revealing existing passwords), **Reset Device**, or **Delete**.
- **Individual Portal Session Controls:** View each user's live Banglarshiksha portal status (`Portal: Active Session` / `Signed Out`), tap **Force Portal Out** or **Reset Portal** to increment `portal_reset_version` so that user's isolated WebView profile is wiped and returned to the fresh Banglarshiksha Sign In screen on next check or open.
- **Directly in Google Sheets:** Changing `status` (`D`) to `active` runs the Apps Script `onEdit` trigger to assign a 30-day `expiry_date` if empty/expired and clear `active_device_id`. Changing `status` (`D`) to `inactive` automatically clears `app_logged_in` and `session_token`.

