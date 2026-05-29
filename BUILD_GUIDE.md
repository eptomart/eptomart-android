# Eptomart Android App — Play Store Build Guide

## What this is
A **Trusted Web Activity (TWA)** — Google's official method to publish a PWA website
as a real Android app on the Play Store. The app opens eptomart.com in a full-screen
Chrome browser with no address bar. Users get native features: home screen icon,
splash screen, push notifications, deep links.

---

## Prerequisites
- Java JDK 11+ (check: `java -version`)
- Android Studio (installs Gradle + SDK automatically)  
  OR just Android command-line tools

---

## Step 1 — Open project in Android Studio

1. Open Android Studio
2. File → Open → select the `eptomart-android` folder
3. Let Gradle sync finish (downloads dependencies)

---

## Step 2 — Add the app icon (launcher icon)

The app needs a 512×512 PNG icon for the Play Store and launcher icons for the device.

**Easiest way:**
1. In Android Studio → right-click `app/src/main/res` → New → Image Asset
2. Select the Eptomart logo PNG from `eptomart-frontend/public/icons/icon-512x512.png`
3. Set background color to `#0B1729`
4. This auto-generates all mipmap sizes

**Manual way:** Copy your icon PNG files to:
- `app/src/main/res/mipmap-hdpi/ic_launcher.png`       (72×72)
- `app/src/main/res/mipmap-xhdpi/ic_launcher.png`      (96×96)
- `app/src/main/res/mipmap-xxhdpi/ic_launcher.png`     (144×144)
- `app/src/main/res/mipmap-xxxhdpi/ic_launcher.png`    (192×192)

---

## Step 3 — Add the splash screen image

Create `app/src/main/res/drawable/splash.xml`:
```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@color/splashBackground"/>
    <item android:gravity="center">
        <!-- Use your logo PNG as the splash center image -->
        <bitmap android:src="@mipmap/ic_launcher" android:gravity="center"/>
    </item>
</layer-list>
```

---

## Step 4 — Generate the signing keystore (ONE TIME ONLY — save it forever!)

Run this command in your terminal (replace values in CAPS):

```bash
keytool -genkey -v \
  -keystore eptomart-release.keystore \
  -alias eptomart \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -dname "CN=Eptomart, OU=Mobile, O=Eptomart, L=Chennai, S=Tamil Nadu, C=IN"
```

It will ask for a keystore password and key password — **save these securely forever**.
If you lose the keystore you can never update the app on Play Store.

### Get the SHA-256 fingerprint (needed for assetlinks.json):
```bash
keytool -list -v -keystore eptomart-release.keystore -alias eptomart
```
Look for the line: `SHA256: XX:XX:XX:...`

**Important:** The SHA-256 fingerprint in the backend's assetlinks.json route
(server.js line ~149) must match the SHA-256 of this keystore exactly.
Update it if you generated a new keystore.

---

## Step 5 — Build the release AAB

**Using Android Studio:**
1. Build → Generate Signed Bundle / APK
2. Choose "Android App Bundle" (.aab)
3. Point to your keystore file, enter passwords
4. Build type: Release
5. The .aab file is saved to `app/release/app-release.aab`

**Using command line:**
```bash
# Set env vars (or edit app/build.gradle directly)
export KEYSTORE_PATH=./eptomart-release.keystore
export KEYSTORE_PASSWORD=yourpassword
export KEY_ALIAS=eptomart
export KEY_PASSWORD=yourkeypassword

# Build
./gradlew bundleRelease

# Output: app/build/outputs/bundle/release/app-release.aab
```

---

## Step 6 — Upload to Play Console

1. Go to https://play.google.com/console
2. Create New App → App name: **Eptomart**
3. Production → Create new release
4. Upload `app-release.aab`
5. Fill in release notes (see PLAY_STORE_LISTING.md)
6. Submit for review

---

## Step 7 — Verify Digital Asset Links (Critical!)

After uploading to Play Store, Google must verify that eptomart.com claims this app.
The backend already serves `/.well-known/assetlinks.json` — make sure it's live.

Test it:
```
https://eptomart.com/.well-known/assetlinks.json
```
Should return JSON with `in.eptomart.app` and the correct SHA-256.

Also verify at: https://developers.google.com/digital-asset-links/tools/generator

---

## Common Issues

| Problem | Fix |
|---------|-----|
| App opens Chrome with address bar | assetlinks.json SHA256 doesn't match keystore — regenerate or update |
| Build fails: SDK not found | Install Android SDK 34 in Android Studio → SDK Manager |
| `KEYSTORE_PASSWORD` empty | Set env vars or hardcode in `signingConfigs` (don't commit to git!) |
| Play Store rejects AAB | Make sure `targetSdk 34` (required for new apps in 2024) |

---

## File Structure
```
eptomart-android/
├── app/
│   ├── build.gradle              ← package name, SDK versions, signing config
│   └── src/main/
│       ├── AndroidManifest.xml   ← TWA URL, deep links, permissions
│       └── res/
│           ├── values/colors.xml ← #0B1729 navy theme
│           ├── values/strings.xml
│           ├── values/themes.xml
│           └── xml/file_paths.xml
├── build.gradle
├── settings.gradle
├── gradle.properties
└── BUILD_GUIDE.md               ← this file
```
