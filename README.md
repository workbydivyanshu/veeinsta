# VeeInsta

Privacy-hardened Instagram mod. Base: AGInsta v12.55 / IG 439.0.0.37.89. See CHANGELOG.md for version history.

## Features

- **Downloader** — save reels, posts, stories, and media to your device.
- **Ghost kit** — hide story views, typing indicators, and read receipts; browse quietly.
- **View-once bypass** — view view-once photos/videos without forced expiry, screenshot freely on-device.
- **Dev flags** — internal developer options / experimental flags exposed for power users.
- **Analytics OFF by default** — in-app analytics and telemetry toggles ship disabled.

## Install

1. Download the APK from **Releases** (`veeinsta.apk`).
2. On Android, allow installs from unknown apps when prompted (Settings → Apps → Special access → Install unknown apps).
3. Open the APK and install.
4. Log in with your Instagram account.

Requirements: Android device (arm64 recommended), working network connection.

## Update policy

- All releases are signed with the **same key**, so updates install over the old version with no uninstall needed.
- No backup needed for normal same-key updates — your login and app data carry over.
- If Android ever complains about a signature mismatch, uninstall and reinstall clean (you will have to log in again).

## Privacy notes

- Crash logs are **on-device only** — VeeInsta does not exfiltrate crash reports to third-party servers.
- No third-party endpoints added by this mod — network traffic is between the app and Meta/Instagram infrastructure.
- **Meta telemetry is inherent** — logging in and using Instagram means Meta still receives standard app traffic. This mod disables optional analytics by default but cannot make Instagram itself fully private.

## Credits

- Upstream: **InstaGold** by assem.mahgoob.
- Maintained by **shaolin** — https://github.com/workbydivyanshu

## Disclaimer

Unofficial third-party mod. Not affiliated with, endorsed, or sponsored by Meta or Instagram. Use at your own risk — modded clients can violate Instagram's Terms of Use and may result in account restrictions. No warranty provided.
