# Mobile App Permission "Configuration Weakness" Playbook

Domain guidance for the finding class in these reports: **Configuration Weakness → Mobile
Application Security → "Company Mobile Application Permissions Disclosed"**, where each row
is a declared permission (iOS usage-description key or Android manifest permission). Load
this when triaging those findings.

## What the finding means

The scanner flags that the published app **declares** permissions that (a) are visible in
the app metadata/binary and (b) may be broader than needed. Declaring a sensitive
permission the app doesn't actually use widens the attack surface and hurts user trust /
store review. Severity is usually INFO per item, but the **set** deserves a real review.

- **iOS** — `NS…UsageDescription` keys in `Info.plist` (e.g. `NSCameraUsageDescription`,
  `NSContactsUsageDescription`, `NSPhotoLibraryUsageDescription`,
  `NSMicrophoneUsageDescription`, `NSLocationWhenInUseUsageDescription`). A key present =
  the app can request that permission and must show a purpose string.
- **Android** — `<uses-permission>` in `AndroidManifest.xml` (e.g.
  `android.permission.CAMERA`, `READ_CONTACTS`, `RECORD_AUDIO`, `ACCESS_FINE_LOCATION`,
  `WRITE_EXTERNAL_STORAGE`, `POST_NOTIFICATIONS`, `ACCESS_COARSE_LOCATION`).

## Triage each permission — is it justified?

For each flagged permission, decide with the mobile team:

1. **Is it actually used** by a live feature? (camera for scan/upload, location for SPBU
   finder, notifications for alerts, etc.)
   - **Yes, used** → *Accept*, and ensure the **purpose string is accurate & specific**
     (iOS) / the permission is **scoped** (Android: prefer coarse over fine location,
     scoped storage / media permissions over broad `WRITE_EXTERNAL_STORAGE`, etc.).
   - **No, unused / leftover** (from an old feature or a dependency/SDK) → *Fix*: remove
     the declaration.
   - **Unsure** → *Needs investigation*: check the code and third-party SDKs that may pull
     the permission in transitively.

2. **Can it be narrowed?** Even if used, prefer the least-privilege variant:
   - Location: `ACCESS_COARSE_LOCATION` instead of `FINE` when precise isn't needed;
     `WhenInUse` instead of `Always` on iOS.
   - Storage/media: scoped storage / `READ_MEDIA_*` (Android 13+) instead of broad external
     storage.
   - Remove `WRITE_EXTERNAL_STORAGE` where the app targets modern Android and uses scoped
     storage.

## Evaluate directly in the codebase (when you have repo access)

When the iOS and Android repos are available, **verify each permission against the actual
code** — don't guess. For every flagged permission: find where it's declared, then search
for the API that requires it. No usage found → strong candidate for removal.

**iOS** (in the iOS repo):
- Declarations: `Info.plist` (the `NS…UsageDescription` keys) and `*.entitlements`.
- Map key → framework/API and grep the source (Swift/ObjC):
  - `NSCameraUsageDescription` → `AVCaptureDevice`, `UIImagePickerController` (camera),
    `AVFoundation` capture.
  - `NSContactsUsageDescription` → `CNContactStore`, `Contacts` / `ContactsUI`.
  - `NSPhotoLibraryUsageDescription` → `PHPhotoLibrary`, `PhotosUI`, image-picker
    (photo library source).
  - `NSMicrophoneUsageDescription` → `AVAudioSession` record, `AVAudioRecorder`.
  - `NSLocationWhenInUseUsageDescription` → `CLLocationManager`, `CoreLocation`.
- If the framework/API isn't referenced anywhere → the key is likely unused.

**Android** (in the Android repo):
- Declarations: app `AndroidManifest.xml`, and the **merged manifest**
  (`app/build/outputs/logs/manifest-merger-*.txt` or the merged manifest in build outputs)
  to catch permissions added by libraries.
- Map permission → API and grep the source (Kotlin/Java):
  - `CAMERA` → `CameraX`, `Camera2`, `MediaStore.ACTION_IMAGE_CAPTURE`.
  - `READ_CONTACTS` → `ContactsContract`, contacts `ContentResolver` queries.
  - `RECORD_AUDIO` → `MediaRecorder`, `AudioRecord`.
  - `ACCESS_FINE/COARSE_LOCATION` → `FusedLocationProviderClient`, `LocationManager`.
  - `WRITE_EXTERNAL_STORAGE` → legacy file writes; usually removable on modern targetSdk
    with scoped storage / `MediaStore`.
  - `POST_NOTIFICATIONS` → `NotificationManagerCompat` / notifications (Android 13+).
- **Transitive/SDK permissions**: check `build.gradle` dependencies — maps, Firebase,
  analytics, and ad SDKs often inject permissions via their manifests. If a permission is
  only from an SDK and unwanted, remove it with a manifest override
  (`<uses-permission android:name="…" tools:node="remove"/>`) or drop/replace the SDK.

**Conclusion per permission**: cite the evidence (declared at <file:line>; used at
<file:line> / no usage found; injected by <SDK>). That evidence is what makes the
Fix/Accept decision defensible.

## Remediation

- **Remove** unused permissions: delete the `<uses-permission>` (Android) / the
  `NS…UsageDescription` key (iOS), verify the feature set still works, and re-scan.
- **Right-size** used permissions to the least-privilege variant.
- **Justify & document** the ones that stay: accurate iOS purpose strings; a short
  permission-justification note per permission (which feature needs it) for the review
  record and store submissions.
- Watch **transitive permissions** pulled by SDKs — remove or replace the SDK, or use
  manifest `remove` rules where appropriate.
- **Owner**: mobile app team (iOS/Android). This is a mobile-app change, not dashboard/web.

## Verification
Rebuild the app, confirm the permission list in the built artifact (Info.plist / merged
AndroidManifest), confirm affected features still work, and re-run the scan to see the
finding clear.

## Quality bar
- Each flagged permission classified: used (accept + right-size), unused (remove), or
  investigate.
- Least-privilege variants applied where possible; iOS purpose strings accurate.
- Transitive/SDK-pulled permissions considered.
- Remaining permissions documented with justification; owner = mobile team; re-scan to verify.
