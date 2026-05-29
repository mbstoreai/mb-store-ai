# MB Store AI — Google Play Store Packaging

The web app is wrapped with **Capacitor** so it can ship as a native Android app
without rebuilding any business logic. All web features (POS, inventory, invoices,
offline mode, PWA install) continue to work unchanged.

## What's already in this repo

| Asset | Path |
| --- | --- |
| Capacitor config | `capacitor.config.ts` |
| Icon/splash sources (used by `@capacitor/assets`) | `resources/` |
| Pre-rendered Android launcher icons | `android-assets/icons/` |
| Pre-rendered splash variants | `android-assets/splash/` |
| Play Store listing assets | `android-assets/play-store/` |
| Brand logo (PWA + web) | `public/brand/logo.png`, `public/icon-*.png` |

App identity:
- **appId**: `com.mbstoreai.app`
- **appName**: `MB Store AI`
- **webDir**: `dist/client` (TanStack Start client build output)

## Local build steps (machine with Android Studio + JDK 17)

These three steps MUST be run locally — the sandbox has no Android SDK.

```bash
# 1. Build the web app
npm install
npm run build

# 2. Add the Android platform (creates the android/ Gradle project)
npx cap add android

# 3. Generate icons + splash for every density
npx capacitor-assets generate --android

# 4. Sync web build into the native shell
npx cap sync android

# 5. Open in Android Studio (build APK / signed AAB from there)
npx cap open android
```

## Producing a signed AAB for Play Store

1. In Android Studio: **Build → Generate Signed Bundle / APK → Android App Bundle**.
2. Create a keystore the first time:
   ```bash
   keytool -genkey -v -keystore mbstoreai-release.jks \
     -keyalg RSA -keysize 2048 -validity 10000 -alias mbstoreai
   ```
   Store the keystore + passwords in a password manager. Losing it means you
   can no longer publish updates under the same listing.
3. Choose **release** build variant. Output goes to
   `android/app/release/app-release.aab`.

For a quick test APK: **Build → Build Bundle(s) / APK(s) → Build APK(s)**.

## Play Store listing assets

Located in `android-assets/play-store/`:

- `icon-512.png` — 512×512 hi-res icon (required)
- `feature-graphic-1024x500.png` — feature graphic (required)

Screenshots you still need to capture (use a real device or AVD at the listed
sizes; minimum 2, maximum 8 per form factor):

- **Phone**: 1080×1920 or 1080×2400 (portrait)
- **7" tablet**: 1200×1920
- **10" tablet**: 1920×1200
- Suggested screens to capture: POS, Inventory, Dashboard, Invoice detail,
  Reports, Offline indicator.

## Permissions

Only what the existing web app already uses:
- `INTERNET` (added by Capacitor by default)
- `CAMERA` (for barcode scanner — declared automatically by Capacitor when
  `getUserMedia` is called; if Play Console flags it, add to
  `android/app/src/main/AndroidManifest.xml`).

No background services, no location, no contacts → fewer Play Store review
questions.

## Session / back-button behavior

- Session persists via Supabase's `localStorage` adapter (works inside the
  WebView).
- Android hardware back button is handled by Capacitor's `App` plugin via
  TanStack Router history — no extra code needed for the common case.

## Updating the app

Web-only changes (no native plugin added/removed):
```bash
npm run build && npx cap sync android
```
Then rebuild the AAB from Android Studio and upload a new release.
