# 5x5 Iron Simple Tracker (`5x5-iron-simple-tracker`)

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F5K229QP0X)

A fast, offline-first **$5 \times 5$ Progressive Overload & Custom Strength Workout Tracker** with **Google Health Connect** integration and a **Dual-Path Release Architecture**:
1. **Path 1 — Google Play Store Release (`.aab` / `.apk`):** Uses the native **Google Play In-App Billing Tip Jar** (`PlayBillingBridge.kt` powered by Google Pay) and automatically strips all Ko-fi links/QR codes during the Android build to remain 100% compliant with Google Play Payments policy.
2. **Path 2 — Standalone Web / GitHub Pages PWA:** Keeps the **Ko-fi Support Button & Scannable QR Code** (`kofi-qr.png`) for web and direct Pixel WebAPK users.

---

## Project Structure

```text
5x5-iron-simple-tracker/
├── index.html                                 # Main app (auto-switches between PWA Ko-fi & Play Store Tip Jar)
├── manifest.json                              # PWA Web App Manifest
├── sw.js                                      # Offline Service Worker (caches app shell + kofi-qr.png for PWA)
├── kofi-qr.png                                # Scannable Ko-fi QR code (used in PWA, excluded from Play Store build)
├── package.json                               # Capacitor & Gradle build scripts
├── capacitor.config.json                      # Capacitor Android configuration (com.iron5x5.tracker)
├── scripts/
│   └── copy-web.js                            # Prepares Play Store compliant www/ (injects PLAY_STORE flag & strips Ko-fi)
├── android/                                   # Native Android project
│   ├── build.gradle
│   ├── settings.gradle
│   ├── keystore.properties.example            # Template for local release signing
│   └── app/
│       ├── build.gradle                       # Health Connect SDK + Google Play Billing SDK + Release signing
│       ├── proguard-rules.pro                 # Keeps JavascriptInterface bridges for R8 minification
│       └── src/main/
│           ├── AndroidManifest.xml            # Health Connect permissions + com.android.vending.BILLING
│           └── java/com/iron5x5/tracker/
│               ├── MainActivity.kt            # Attaches AndroidHealthConnect & AndroidBilling bridges
│               ├── HealthConnectBridge.kt     # Writes ExerciseSessionRecord & ActiveCaloriesBurnedRecord
│               └── PlayBillingBridge.kt       # Native Google Play Billing Tip Jar (tip_coffee, tip_shake, tip_preworkout)
└── .github/workflows/
    └── android-release.yml                    # Automated cloud build for Signed APK & Play Store AAB
```

---

## Step-by-Step Guide 1: How to Update Your GitHub Project (and Keep the PWA Live)

### Option A: Using Git in Terminal / VS Code (Recommended)
Run these commands in your project folder (`c:\Users\p_ang\VIbeCoding\5x5\V1`):

```bash
git status
git add .
git commit -m "Add custom workouts, Health Connect, Play Store Tip Jar, and PWA Ko-fi support"
git push origin main
```

### Option B: Uploading via the GitHub Web Interface
1. Open your repository on GitHub (`5x5-iron-simple-tracker`).
2. Click **Add file -> Upload files**.
3. Drag and drop **all files and folders** from `c:\Users\p_ang\VIbeCoding\5x5\V1`, making sure to include:
   - `index.html`, `manifest.json`, `sw.js`, `kofi-qr.png`, `README.md`
   - `package.json`, `capacitor.config.json`, `.gitignore`
   - `scripts/` folder
   - `android/` folder
   - `.github/` folder *(Note: On Windows Explorer, check **View -> Show -> Hidden items** so `.github` and `.gitignore` are visible)*
4. Click **Commit changes** to `main`.

### What Happens Automatically When You Push to GitHub:
1. **Your PWA Updates (Path 2):** GitHub Pages serves the root `index.html`, `sw.js`, and `kofi-qr.png` directly. Users visiting your GitHub Pages URL (or using the installed Pixel WebAPK) get all the new custom workout features **with the Ko-fi button and QR code intact**.
2. **Your Play Store Bundle Builds (Path 1):** GitHub Actions runs `.github/workflows/android-release.yml`, which executes `npm run sync:web` (`scripts/copy-web.js`). This creates a sanitized `www/` folder with `window.APP_DISTRIBUTION_CHANNEL = "PLAY_STORE"` and zero Ko-fi references, then compiles the signed `.apk` and `.aab` under the **Actions** tab.

---

## Step-by-Step Guide 2: How to Publish to the Google Play Store

### Step 1: Generate Your Permanent Upload Keystore (One-Time Setup)
Google Play requires your `.aab` bundle to be signed with your own upload key.
1. If you have Java installed (or inside GitHub Codespaces / Cloud Shell), run:
   ```bash
   keytool -genkeypair -v -keystore release-keystore.jks -alias iron5x5 -keyalg RSA -keysize 2048 -validity 10000
   ```
2. Convert `release-keystore.jks` to a Base64 string:
   - **PowerShell (Windows):**
     ```powershell
     [Convert]::ToBase64String([IO.File]::ReadAllBytes("release-keystore.jks")) | Set-Clipboard
     ```
3. In your GitHub repository, go to **Settings -> Secrets and variables -> Actions -> New repository secret** and add these 4 secrets:
   - `ANDROID_KEYSTORE_BASE64`: *(paste the Base64 string from your clipboard)*
   - `ANDROID_STORE_PASSWORD`: *(your keystore password)*
   - `ANDROID_KEY_ALIAS`: `iron5x5`
   - `ANDROID_KEY_PASSWORD`: *(your key password)*
4. Trigger a run in the **Actions** tab (`Build Signed Android APK & Play Store AAB`) and download the **`5x5-iron-tracker-playstore-aab`** zip artifact, which contains `app-release.aab`.

---

### Step 2: Create the App in Google Play Console
1. Go to the [Google Play Console](https://play.google.com/console) and sign in with your Google Play Developer account.
2. Click **Create app**:
   - **App name:** `5x5 Iron Simple Tracker`
   - **Default language:** English
   - **App or game:** App
   - **Free or paid:** Free
3. Accept the declarations and click **Create app**.

---

### Step 3: Configure the Google Play Tip Jar Products (In-App Products)
Because `PlayBillingBridge.kt` queries 3 consumable tip IDs, create them in Play Console so users can tip via Google Pay:
1. In the left sidebar of Google Play Console, go to **Monetize with Play -> Products -> In-app products**.
2. *(Note: If Play Console asks you to upload a `.aab` with the `BILLING` permission first, upload `app-release.aab` to **Testing -> Internal testing** first, then return here.)*
3. Click **Create product** for each of the 3 tip tiers:
   - **Product 1:**
     - **Product ID:** `tip_coffee`
     - **Name:** `Gym Coffee Tip`
     - **Description:** `Fuel a late-night coding session`
     - **Price:** `$1.99` (or your local currency equivalent)
   - **Product 2:**
     - **Product ID:** `tip_shake`
     - **Name:** `Post-Workout Shake`
     - **Description:** `Support ongoing features & updates`
     - **Price:** `$4.99`
   - **Product 3:**
     - **Product ID:** `tip_preworkout`
     - **Name:** `Heavy PR / Pre-Workout Tub`
     - **Description:** `Huge support for indie development`
     - **Price:** `$9.99`
4. Click **Save** and **Activate** on all 3 products. (`PlayBillingBridge.kt` automatically marks them as *consumed* after purchase so supporters can tip more than once.)

---

### Step 4: Complete the Health Connect & App Content Declarations
In the left sidebar, go to **Policy and programs -> App content** and complete the required forms:
1. **Privacy Policy:** Provide a URL to a simple page (you can host a `PRIVACY.md` or `privacy.html` on your GitHub Pages repo!) stating that `5x5 Iron Simple Tracker` stores workout logs locally on the user's device, writes workouts/calories to Android Health Connect only when authorized by the user, and does not sell or transmit personal health data to external servers.
2. **Health apps declaration:**
   - Select **Health & Fitness** / **Fitness and exercise tracking**.
   - Declare the Health Connect permissions requested in `AndroidManifest.xml`:
     - `READ_EXERCISE` / `WRITE_EXERCISE` (Used to record completed 5x5 strength sessions).
     - `READ_ACTIVE_CALORIES_BURNED` / `WRITE_ACTIVE_CALORIES_BURNED` / `TOTAL_CALORIES_BURNED` (Used to record estimated calories burned during strength workouts).
3. **Data safety:**
   - Does your app collect or share any of the required user data types? Select **No** (since all data remains on-device / on-device Health Connect).
4. **Ads:** Select **No, my app does not contain ads**.

---

### Step 5: Upload `app-release.aab` & Roll Out
1. Go to **Test and release -> Testing -> Internal testing** (or **Closed testing** / **Production**).
2. Click **Create new release**.
3. Under **App bundles**, upload `app-release.aab` (extracted from `5x5-iron-tracker-playstore-aab`).
4. Enter release notes (e.g., *"v1.1.0: Added custom workout routines, custom exercise library, Google Health Connect sync, and Google Play Tip Jar"*).
5. Click **Next -> Save and publish**.

---

## Support & Donate (Web / PWA Version)

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F5K229QP0X)

<img src="./kofi-qr.png" alt="Ko-fi QR Code" width="220" />
