# VeeInsta changelog

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
