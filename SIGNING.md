# SIGNING.md — Code signing & recovery

## What the project uses

- **Team ID:** `53T98NLS49`
- **Signing:** Automatic for every target (`CODE_SIGN_STYLE: Automatic` in `project.yml`).
  Xcode creates and renews certificates and provisioning profiles by itself.

| Target | Bundle ID | Capabilities |
|---|---|---|
| Forge (app) | `com.bilalhammad.forge.native` | HealthKit (+ Background Delivery), Sign in with Apple, App Groups |
| ForgeWidgets (Live Activity) | `com.bilalhammad.forge.native.widgets` | App Groups |
| ForgeTests / ForgeUITests | `…native.unittests` / `…native.uitests` | none (Xcode handles these) |

- **App Group:** `group.com.bilalhammad.forge.native` (timer pause/stop signal between the
  Live Activity and the app).
- **Not used:** Push Notifications, iCloud/CloudKit, Background Modes, Keychain Sharing,
  HealthKit Clinical Health Records (removed 2026-09-23, never used by code).
- **No `.p8` key is needed** by this project.

In-app purchase product IDs (must match `ProductIdentifiers.swift` / `Configuration/Forge.storekit`):
- `com.bilalhammad.forge.native.premium.monthly` — auto-renewable, 1 month, group "Forge Premium"
- `com.bilalhammad.forge.native.premium.yearly` — auto-renewable, 1 year, group "Forge Premium"
- `com.bilalhammad.forge.native.pack.islamic` — non-consumable

## If certificates get deleted/revoked again

This is exactly what was done on 2026-09-23 (App IDs and App Group were still on
developer.apple.com; only certificates were gone):

1. **Delete the stale local profiles** (they were signed with the dead certificates):
   ```
   rm ~/Library/Developer/Xcode/UserData/Provisioning\ Profiles/*.mobileprovision
   ```
2. **Restart Xcode.** In Xcode → Settings → Accounts, make sure your Apple ID is signed in
   and team `53T98NLS49` is listed.
3. Connect the iPhone, open `Forge.xcodeproj` → **Forge** target → **Signing & Capabilities**.
   Leave "Automatically manage signing" on. Click **Try Again** if Xcode asks — it creates a
   new Apple Development certificate and profile.
4. Repeat step 3 for the **ForgeWidgets** target.
5. Build and run on the device. If it runs, signing is fixed.

If Xcode says the certificate's private key is missing: open Keychain Access, delete the old
revoked "Apple Development" certificate for this team, then click **Try Again**.

### If App IDs or the App Group were also deleted

On developer.apple.com → Certificates, Identifiers & Profiles → Identifiers, recreate:
- App Group `group.com.bilalhammad.forge.native`
- App ID `com.bilalhammad.forge.native` with HealthKit, Sign in with Apple, App Groups (assign the group)
- App ID `com.bilalhammad.forge.native.widgets` with App Groups (assign the group)

Then follow the steps above. (Xcode's automatic signing can usually register these itself, but
doing it by hand avoids capability errors.)

## Distribution (TestFlight / App Store)

The distribution certificate and App Store profile are created automatically the first time
you use **Product → Archive → Distribute App** — nothing to set up in advance.

## Note for anyone editing the project

`project.yml` (XcodeGen) is the source of truth. Capabilities live in
`Forge/Forge.entitlements` and `ForgeWidgets/ForgeWidgets.entitlements` — change them there,
not only in Xcode's Signing & Capabilities tab, or a regenerate will wipe the change.
