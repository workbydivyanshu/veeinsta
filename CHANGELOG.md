# VeeInsta changelog

## v12.61-veeinsta1 — 2026-09-21

Base: AGInsta v12.61 / Instagram 439.0.0.37.89 (same IG base as v12.55 — this release is a mod-layer bump: 12.55 → 12.61).
Package: `com.veeinsta.android`, signed with the same key (updates install over it, no uninstall).

### Privacy strip (re-verified on the new base)
- Same as v12.55: Google Analytics dispatch pair + AdMob ping loopbacked to device; Meta/Instagram hosts, translate endpoints, mod proxy, login/session/auth path, OS push registry untouched by design.
- No VPN bypass exists in this build (verified same as v12.55: no forced Network, stock network-security-config).

### Nag removal
- Same as v12.55: FollowDeveloper lifecycle registration + WelcomeMessage first-launch dialog neutered; update-check and verification-submit left functional.

### Rebrand
- Launcher label + settings show **VeeInsta**; About → Developer is **shaolin** (github.com/workbydivyanshu) in default + all locales (27 locale files on this base, was 28).
- Gold Apps dashboard, helper/updater hooks, donate rows, author links removed.
- Crash-report recipient is shaolin; reports stay on-device + review-gated (unchanged property).
- Analytics ships default-ON-disabled for fresh installs and migrates existing installs to ON (still user-toggleable).

### Delta vs v12.55 (new in this base)
- GoldSecrets URLs are now int-array-obfuscated (were plain const-strings) → whole-method body replacement with blank strings instead of const swaps. New `updateFeedBackup()` in the same family blanked too.
- Gold-apps teardown narrowed to a single-point `startGoldApps()` NOP: v12.61's `startFragment(String)` also dispatches legitimate fragments, so the broader removal from v12.55 would have broken them.
- Build fix new to this base: the `res/xml` rename broke the aapt link via stale `public.xml` entries → the two entries updated to veeinsta names (same resource IDs).
- Badging label is now **VeeInsta** (v12.55 still badged the upstream name).

### Verification (all evidenced)
- Clean apktool build, first try after the public.xml fix, zero further fixes. Same-key signature verified by fingerprint.
- Dex recount vs source decode accounted for string-pool dedup: 127.0.0.1 ×3 (GA/AdMob loopback), github ×1 pool entry, shaolin entries, sole analytics remnant is the deliberate host-only literal.
- On-phone: `adb install -r` → Success over the prior build (update path, no uninstall); package `com.veeinsta.android`, version 439.0.0.37.89 confirmed on device; stock Instagram and AGInsta side-by-side installs untouched.

### Residual risks (honest, same as v12.55)
- `libgoldinsta.so` builds verify/update/OTA/translate requests natively with encrypted strings — static analysis cannot prove what fires at runtime.
- Confidence raiser: run a PCAPdroid/RethinkDNS capture during real use; allowlist Meta + Google-translate + ftapi.pythonanywhere.com and flag anything else.

## v12.55-veeinsta1 — 2026-09-15

Base: AGInsta v12.55 / Instagram 439.0.0.37.89 (same IG base as v12.50 — this release is a mod-layer bump: 12.50 → 12.55).
Package: `com.veeinsta.android`, signed with the same key as v12.50 (updates install over it, no uninstall).

### Privacy strip (re-verified on the new base)
- Google Analytics dispatch pair + AdMob ping loopbacked to device (verifier-safe const swaps); GA manifest components were already disabled upstream.
- Untouched by design: all Meta/Instagram hosts, translate.google.com + translate.googleapis.com (translate feature), ftapi.pythonanywhere.com (mod proxy), login/session/auth path, OS push registry.
- No VPN bypass exists in this build (no bindProcessToNetwork, no forced Network in HTTP builders, stock network-security-config, airplane patch is a user-toggled ghost-mode feature).

### Nag removal
- FollowDeveloper lifecycle registration + WelcomeMessage first-launch dialog neutered (early returns, callers intact).
- Update-check (user-tapped) and verification-submit dialog left functional.

### Rebrand
- Launcher label + settings show **VeeInsta**; About → Developer is **shaolin** (github.com/workbydivyanshu) in default + all locales.
- Gold Apps dashboard, helper/updater hooks, donate rows, author links removed.
- Crash-report recipient is shaolin; reports stay on-device + review-gated (unchanged property).
- Analytics ships default-ON-disabled for fresh installs and migrates existing installs to ON (still user-toggleable).

### Verification (all evidenced)
- Clean apktool build, first try, zero fixes. Same-key signature verified by fingerprint.
- Zero `assem.mahgoob` hits in built APK; `instagold`/`aginsta` recount identical between source decode and built-APK re-decode (manifest 43/43, res 1294/1294, smali consts 444/444).
- On-phone: installed over prior build, cold launch, pid alive, VeeInsta activity focused, zero FATAL, login screen reached (never logged in — owner's credentials).

### Residual risks (honest)
- `libgoldinsta.so` builds verify/update/OTA/translate requests natively with encrypted strings — static analysis cannot prove what fires at runtime. JNI shows native *can* call verification/update endpoints, not that it does unprompted.
- Confidence raiser: run a PCAPdroid/RethinkDNS capture during real use; allowlist Meta + Google-translate + ftapi.pythonanywhere.com and flag anything else.

## v12.50-veeinsta1 — 2026-09-12

Initial VeeInsta release. Same pipeline (rename, crash fix, privacy strip, VPN+nag, rebrand, analytics default-ON, same-key sign). Base: AGInsta v12.50 / IG 439.0.0.37.89.
