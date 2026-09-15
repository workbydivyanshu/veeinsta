# VeeInsta Survival RE Report — how the AGInsta/InstaGold mod attaches, so VeeInsta can be rebuilt from stock IG

- Workdir: `/home/divyu/GitHub/veeinsta`
- Sources (read-only): `ag-src` (AGInsta v12.50 decode), `scratch-tmp/ag-v12.55` (AGInsta v12.55 decode), `~/.claude/jarvis/veeinsta-re-map.md` (2026-09-12 map — built on, not redone)
- Both mods sit on stock IG base **439.0.0.37.89** (`versionName` in both `apktool.yml`; apktool 3.0.3). Both decodes in this tree already carry committed VeeInsta edits (rename to `com.veeinsta.android`, nag/privacy/rebrand strips, stage-2 crash hunk) — findings below describe the **upstream mod layer**; known VeeInsta deltas are flagged as such.
- No secrets in this report. No edits were made except this file.

## 1. MOD LAYER ANATOMY (v12.55 census, `com.GoldInsta` + `com.dcc`)

Mod-only smali (excluding vendored `internal/` guava/media3): **1298 files (v12.50) → 1563 files (v12.55)**. Dex spread changes between versions (v12.50 mod lives almost entirely in `smali/`; v12.55 mod spans `smali/` + `smali_classes3/` + `smali_classes4/`) — a re-applier must collect `com/GoldInsta` + `com/dcc` from **all** dex dirs, not just `smali/`.

### 1.1 Loader shim: `com.dcc`

- `smali/com/dcc/DccApplication.smali` (25 lines, both versions): `extends Application`, `static clinit` does `System.loadLibrary("goldinsta")`, one `public static native initDcc()`. This is the **entire native bootstrap**. Only v12.50↔v12.55 diff in this file is `const-string` vs `const-string/jumbo` (assembler noise, no semantic change).

### 1.2 `patches/` submodules (`.../instagram/patches/`, file counts v12.50 → v12.55)

| Submodule | 12.50 | 12.55 | Hooks (stock surface) |
|---|---|---|---|
| accounts | 3 | 3 | `UnlimitedAccounts`, `CloneAccountCompat` — multi-account limit removal |
| actionbar | 12 | 33 | `ActionBarPatch`, `HomeActionBarLayout`, `HomeHeaderPolicy`, `MainFeedActionBar` — feed header buttons |
| activity | 62 | 62 | `ActivityHook`, `ActivityHistory` + `ActivityHistoryActivity` (manifest activity) — activity-status tracking UI |
| airplane | 12 | 12 | `AirplaneModeController` — ghost mode: stops MQTT realtime via reflection (`stopMqttClient`/`startMqttClient`), restarts activity. Feature, NOT a network bypass (stage-4 verdict) |
| comment | 9 | 9 | `copyTextButton/`, `debugButton/`, `saveMediaButton/`, `translateButton/` — per-comment action buttons |
| devFlags | 9 | 12 | `HookFlags`, `ExperimentalFeatures`, `DeveloperOptions`, `RecommendedFlags` — internal flags UI |
| dm | 208 | 210 | `DirectChatButtonsHook`, `DirectChatCustomizer`, `DirectMessageTranslator` (Google + ftapi proxy), `DeletedMessages*` + `DeletedMessagesActivity`/`DeletedThreadsActivity`/`DeletedMediaPreviewActivity` (undeleter), `HiddenChats*` (applock-adjacent), `MarkChatAsRead`, `SavedMessagesHook`, `GoToFirstMessageButton`, `VideoToVoiceFeature` |
| download | 93 | 117 | `MediaDownloader`, `DownloadUtils`, `DownloadMapping`, `DownloadHistoryStore` + `DownloadHistoryActivity`, `GoldDirectMediaPreviewActivity`, NEW `DownloadMetadata` + `MetadataMuxer` (native metadata mux) — reels/posts/stories saver |
| feed | 16 | 17 | `FeedActionsLayout`, `PostDownloadComposeCallback`, `MoreOptionsOnPostPatch`, `LimitFeedToFollowingProfiles` |
| filter | 1 | 1 | `FilterStory` (+`filter/story/`) |
| hide/navigation | 1 (`hide/navigation/HideNavigationButtonsPatch`) | 0 / NEW `navigation/` 7 | Hide-tabs patch **replaced** by full `NavigationBarPatch` + tab-editor UI (`NavigationBarAdapter`, `NavigationBarPreference`, `NavigationStartupPreference`, access-policy guards) |
| liquidglass | 74 | 75 | `LiquidGlassManager/Layout/Drawable/Shader/TouchOverlay/Preview*` — glassmorphism nav/theme engine |
| media | 9 | 9 | `LongPressZoom`, `MediaDecoder`, `MediaOptionsPopup`, `PostOptions` |
| overflowMenuButton | 15 | 20 | `FeedButton`, `ReelButton`, `ReelOverflowButton`, `ReelsOptionsButton`, `reels/buttons/` — inject download buttons into post/reel menus |
| splitter | 16 | 16 | `VideoSplitter` — video split for status/stories |
| story | 23 | 83 | `StorySeenBridge/History/Button/View/BindingRetry/RequestScope/State/Key`, `StoryMentionIndicatorView`, `ViewStoryMentionsPatch`, `FilterStory`, `LongPressZoom`-adjacent — **seen-state subsystem is the big v12.55 addition**; `EphemeralMediaPatch` (view-once bypass) also lives at this level |
| text | 12 | 12 | `TranslationClient` (googleapis + `GOOGLE_V2_ENDPOINT = "https://ftapi.pythonanywhere.com/translate"`), `TextActions`, `GoldTextView` |
| updates | 31 | 94 | `UpdateManager`, `GoldOtaUpdater`, `GoldUpdateDialog`, `GoldUpdateInfo`, NEW `GoldUpdateFileProvider` (manifest provider) — **self-update pipeline is the other big v12.55 addition** |
| userprofile | 54 | 55 | `ProfilePictureViewer`, `FavoriteProfiles` + `FavoriteProfilesActivity`, `UserProfileButton/ActionBar`, `FriendshipStatusIndicator`, `ProfileMoreOption` |
| verification | 10 | 12 | `VerificationManager`, badge cache |
| (top-level) | — | — | `Block`, `Links` (author-link choke point), `FollowDeveloper` (dev-follow nag), `WelcomeMessage`, `Branding`, `LauncherIconGuard`, `RestartHelper`, `FragmentHook`, `ActivityHook` |

### 1.3 `settings/` + storage

- `.../instagram/settings/`: `SettingsActivity` (manifest activity) + `SettingsShortcutActivity` (v12.55 only), `SettingsTaskService` (manifest service), `SettingsStatus`, `SettingsRestart`, `GoldAppPref`/`Pref`, appearance (`ThemeChooserActivity`, `GoldAppearanceManager`, `GoldThemeHooks`, `GoldFontEmojiHooks`, `GoldStoryFontBridge/Registry`, `MaterialYouTheme`, `LauncherIconGuard`), applock (`AppLockManager`, `HiddenChatsManager`), language (`GoldLanguageManager`), `preference/fragments/` (incl. `BackupPrefActivity`, `RestorePrefActivity`), `preference/widgets/`, `GoldSettingsSearchIndex`, `GoldPreferenceSearchController`.
- `settings/storage/` + `shared/settings/`: `BaseSharedPref`, `SharedPref`, `HookFlags`, `Boolean/StringSetting`-style typed prefs. `core/SharedPref.ensurePreferencesReady()` is called at app attach.
- `shared/`: `Utils` (+`setContext`), `ResourceUtils`, `Logger`, `StringRef`, `ShareLinkSanitizer`, `MarkChatAsReadScope`, `requests/`, `ui/`.
- `constants/` + `instagram/constants/` + `instagram/entity/` (`MediaData`, `UserData`, `MessageInfo`, …) + `instagram/db/` (`GoldMessageDb`, `GoldMessageDb` stores) + `instagram/utils/` (`Pref` — the **#1 stock-called bridge, 52 call sites**, `InstaUtils`, `DownloadUtils`-adjacent) + `instagram/theme/` + `instagram/security/` (`GoldSecrets` — see §3) + `instagram/legacy/` + `core/` (`GoldUtils`, `ObjectBrowser`, `CustomCrashHandler`) + `diagnostics/` (`CrashReportManager/Store/Activity` — on-device only, recipient is a username string) + `downloader/` (`FolderPickerActivity` — manifest activity — `MediaDownloader`, `MetadataMuxer`, `Player`).

### 1.4 Resources / assets

- `res/xml/gold_*.xml`: **17 files, identical count in both versions** (`gold_main_settings`, `gold_settings_about/app_lock/apps/backup/chat/developer/download/experimental/gestures`, …) — the whole settings UI is data-driven from these.
- `assets/gold/`: 4 `.arsc` theme overlays (`amoled`, `amoled_material_you`, `material_you_dark/light`) — both versions.
- `res/xml/`: locale config is `instagold_locale_config.xml` upstream; this tree's v12.50 manifest points at `@xml/veeinsta_locale_config` with file `ag-src/res/xml/veeinsta_locale_config.xml` — **that rename is a VeeInsta edit, not upstream**. v12.55 decode still ships `instagold_locale_config.xml`.

## 2. ATTACH POINTS — how the mod injects

No Xposed, no dynamic hooking framework. It is **static smali surgery + additive components**:

### 2.1 Application hook (2 hunks, both in stock `com.instagram.app.InstagramAppShell`)

1. `scratch-tmp/ag-v12.55/smali/com/instagram/app/InstagramAppShell.smali:23-35` — stock `<clinit>` opens with `invoke-static {}, Lcom/dcc/DccApplication;->initDcc()V` (line 26). Native init runs before anything else. (Same hunk in v12.50; only the `LX/06Bz` obfuscated field name will differ per base.)
2. `.../InstagramAppShell.smali:677-701` (`attachBaseContext`, after `attachBaseContext_begin` tracing) — ordered mod init block, the mod's real entry point:
   - `677: GoldInsta/shared/Utils.setContext`
   - `679: instagram/theme/MaterialYouTheme.initialize`
   - `681: patches/activity/ActivityHistory.rememberContext`
   - `683: instagram/diagnostics/CrashReportManager.init`
   - `685: core/SharedPref.ensurePreferencesReady`
   - `687: settings/appearance/GoldAppearanceManager.onApplicationCreated`
   - `689: settings/applock/AppLockManager.init`
   - `691: patches/FollowDeveloper.init` (nag init — still present in decode; neutralized downstream by VeeInsta strip)
   - `693: patches/liquidglass/LiquidGlassManager.init`
   - `695: settings/SettingsStatus.load`
   - `697: patches/devFlags/HookFlags.load`
   - `699: constants/Constants.load`
   - `701: patches/verification/VerificationManager.hasVerified`
   - Manifest `android:name` stays stock (`com.instagram.app.InstagramAppShell`) — no Application subclass swap, no extra content provider for init.

### 2.2 Inline stock edits: **216 files (v12.55; 203 in v12.50)** outside `com/GoldInsta` reference `com/GoldInsta`

All under obfuscated `smali*/X/*.smali` (base IG is R8-obfuscated, so stock method names are **not recoverable** — only the mod-side bridge is). Hook style is uniform: stock code calls **static bridge methods** on named GoldInsta classes. Top bridges by stock call-site count (v12.55, `smali/X` only):
`instagram/utils/Pref` 52, `settings/appearance/GoldThemeHooks` 39, `settings/appearance/LauncherIconGuard` 38, `settings/appearance/GoldFontEmojiHooks` 15, `patches/Links` 9, `patches/actionbar/ActionBarPatch` 9, `instagram/theme/MaterialYouTheme` 7, `settings/applock/HiddenChatsManager` 7, `settings/appearance/GoldStoryFontBridge` 7, `patches/story/StorySeenBridge` 6, `patches/dm/SavedMessagesHook` 6, `patches/dm/DirectChatButtonsHook` 6, `patches/navigation/NavigationBarPatch` 5, `patches/feed/PostDownloadComposeCallback` 5, … down to `AirplaneModeController`, `TranslateButton`, `VerificationManager` (full list recoverable with the one-liner in §5 step 5).
Rebase rule: these 216 hunks are **base-version-coupled** (obfuscated names shift every IG release). They cannot be replayed by script onto a different base — only onto the same 439.0.0.37.89 base, by binary-copy of the edited stock files.

### 2.3 Manifest deltas vs stock (no stock manifest on hand to diff — categories + exact mod entries)

Package in this tree is `com.veeinsta.android` (**VeeInsta rename; stock is `com.instagram.android`** — stale `com.instagram.android` refs still remain in the manifest, evidence of the rename method). Mod-additive entries, identical v12.50→v12.55 except where noted:
- **Activities (12 + 2 new in v12.55)**: `GoldInsta.downloader.FolderPickerActivity`, `instagram.diagnostics.CrashReportActivity`, `patches/activity/ActivityHistoryActivity`, `patches/dm/deleted/DeletedMediaPreviewActivity`, `patches/dm/deleted/DeletedMessagesActivity`, `patches/dm/deleted/DeletedThreadsActivity`, `patches/download/DownloadHistoryActivity`, `patches/download/GoldDirectMediaPreviewActivity`, `patches/userprofile/FavoriteProfilesActivity`, `settings/appearance/ThemeChooserActivity`, `settings/preference/fragments/BackupPrefActivity`, `settings/preference/fragments/RestorePrefActivity`, `settings/SettingsActivity`, **v12.55+: `settings/SettingsShortcutActivity`**, plus 39 `instagram.appicon.GoldIcon2…40` launcher-icon aliases (for `LauncherIconGuard`). Zero standalone mod activities was wrong in the old map — there are 14; what is true is the mod **injects into stock screens** rather than replacing them.
- **Providers (2, both mod)**: `androidx.core.content.FileProvider` @ `..._instagold.deletedmedia`, and **v12.55+** `patches/updates/GoldUpdateFileProvider` @ `..._instagold.updates` (OTA APK install). All other providers are stock (authorities mechanically follow the package rename).
- **Services (1)**: `instagram.settings.SettingsTaskService` (`exported=false`, `stopWithTask=false`).
- **Receivers**: none mod-owned.
- **Permissions**: full list is stock IG's (~80, location/camera/BLE/billing/badge/fb-katana etc. — §2.3 background check printed them); nothing is recognizably mod-added. A stock-manifest diff is still owed (see §6).

## 3. NATIVE LAYER — `libgoldinsta.so` (arm64-v8a only; no other ABI ships it)

- Loader: `com/dcc/DccApplication.smali:6-14` (`loadLibrary("goldinsta")` in `clinit`).
- Size/JNI collapse: **3,169,680 B + 910 `Java_com_GoldInsta_*` exports (v12.50) → 859,904 B + 125 exports (v12.55)**. Cause identified: the DM database moved native→Java — v12.50 `instagram/db/GoldMessageDb.smali` declares **33 native methods**; v12.55 same file declares **0** (pure-Java `GoldMessageDb.smali` still present at `smali_classes3/.../instagram/db/`). Remaining native surface covers: downloader SAF/metadata (`MediaDownloader.createSafTarget/getOrCreateDirectory`, `MetadataMuxer.buildMetadataEntries/collectDownloadMetadata/extractAudioTrack`), deleted-message stores, `ActivityHistory` CRUD, translate (`executeGoogleRequest/executeGoogleV2Request`), applock (`clearHiddenChatPin/clearMainLock/isValidPin`), OTA (`GoldUpdateInfo.handleResult/markPrompt`), misc (`openDeveloperProfile`, `hideAndRefresh`, `NavigationBarPatch.Config encodeConfig/encodeHiddenKeys`, `constantTimeEquals`, `localized/hours`, `fetchFeed`).
- **Must stay binary**: `instagram/security/GoldSecrets.smali` — ALL 13 endpoint accessors are `native` stubs (`updateFeed/updatePage/goldAppUrl/website/telegram/reportEmail/emojiBase/fontBase/pikoMappingsBase/verificationListUrls/verificationRequestBase` + `decode/decodeSource`). Stage-5 verified zero plaintext author hosts anywhere in smali/res/assets — every author URL is an obfuscated `int[]` decoded by native `decode()`. So: update channel, author links, font/emoji packs, verification lists **cannot be reimplemented from these artifacts** — they must either keep calling the author's `.so`, or be re-pointed/rebuilt by VeeInsta (as stage-5 did with early-return choke-point patches).
- **Reimplementable in principle**: DB layer (already done upstream in v12.55 — pure Java), download/SAF helpers, pin checks, translate HTTP (endpoints are plaintext: googleapis + ftapi). The `.so` also exports `Java_com_dcc_DccApplication_initDcc` — entry behavior is opaque; keep the binary.
- Note: `com.dcc` smells like a generic mod-build pipeline ("DCC" + numeric-suffixed JNI mangling `__...__Ljava_lang_String_2`), i.e. the author's toolchain, not IG.

## 4. DECODE DIFF v12.50 → v12.55 (same base 439.0.0.37.89 — this is mod velocity, not a base bump)

- Scale: mod smali 1298 → 1563 (+20%); stock hook sites 203 → 216 (+13); JNI 910 → 125 (−86%); `.so` 3.17 MB → 860 KB (−73%); dex spread `smali/` → `smali/`+`smali_classes3/4` (+`internal/` guava/media3 vendoring inflates raw totals to 4373 — exclude it).
- New subsystems: story seen-state (story/ 23→83), OTA self-update (updates/ 31→94 + provider + `SettingsShortcutActivity`), `DownloadMetadata`/`MetadataMuxer`, `NavigationBarPatch` tab editor **replacing** `hide/navigation/HideNavigationButtonsPatch` (only true removal), actionbar 12→33 (`HomeActionBarLayout/HomeHeaderPolicy`), devFlags 9→12, download 93→117, overflowMenuButton 15→20, feed 16→17, verification 10→12.
- Unchanged: airplane, comment, text, splitter, media, accounts, activity (counts identical); `FollowDeveloper`/`Links`/`WelcomeMessage`/`EphemeralMediaPatch`/`Block`/`Branding` still present; 17 `gold_*.xml` prefs; 4 `assets/gold` arsc; manifest shape (+2 entries).
- Noise to ignore: `-IA` (`R8$$SyntheticClass`) files in v12.50 and `a0..z0` single-letter helpers in v12.55 are toolchain exhaust, not features.
- Calibration: on a constant base the author still moved ~20% of the mod in one minor version and re-cut the native lib. A rebase to a **new IG base** means re-doing all ~216 obfuscated stock hunks by hand against shifted names — that is the dominant cost, dwarfing the merge.

## 5. STOCK-IG REBUILD PLAN (stock 439.0.0.37.89 → VeeInsta)

Doable **only** on the matching base; a newer base needs the author (or full RE of new obfuscated names). Needs author's binaries where marked [BIN], derivable from these decodes where marked [TREE].

1. [TREE] Acquire stock IG APK at exactly 439.0.0.37.89 and `apktool d` it (3.0.3). Keep a pristine copy — it is your only stock reference (this tree has none).
2. [TREE] Package rename: `com.instagram.android` → `com.veeinsta.android` across manifest (authorities follow mechanically) + `apktool.yml`; fix stragglers (this tree still has some) and rename `res/xml/instagold_locale_config.xml` → `veeinsta_locale_config.xml` to match the manifest attr.
3. [TREE] Merge additive mod tree: copy `smali*/com/GoldInsta/**` (minus `internal/` unless you want the vendored guava/media3 — decide once; v12.55 needs it on classpath) + `smali/com/dcc/DccApplication.smali` into the corresponding dex dirs of the stock decode. Multidex placement must mirror the donor (classes3/4 paths matter for the verifier).
4. [TREE] Application hook: re-apply the 2 hunks — `initDcc()` call at the top of `InstagramAppShell.<clinit>` and the 13-call init block in `attachBaseContext` (order matters: `Utils.setContext` first, verification last). Then the 216 stock X/ hunks: **do not hand-replay** — copy the exact edited stock files from the v12.55 donor decode (same base ⇒ same obfuscated names ⇒ byte-reuse is safe). Regenerate the bridge list to verify coverage: `grep -rl com/GoldInsta` over non-mod smali should return ~216.
5. [TREE] Manifest merge: add the 14 activities + 39 icon aliases + 2 providers + 1 service (§2.3). No mod permissions/receivers needed.
6. [TREE] Native: copy `lib/arm64-v8a/libgoldinsta.so` [BIN — the v12.55 860 KB build; no source]. Confirm no other ABI needs it (stock IG ships all ABIs; mod only augments arm64 — same as donor, leave others stock).
7. [TREE] Crash-fix hunk (stage-2, sibling-derived): the `DccApplication` constructor-area fix referenced in `veeinsta-v1255-stage3-fix.md` — re-derive by diffing donor `DccApplication.smali` + `InstagramAppShell` attach region against your fresh merge; do not skip (clean merge bootloops without it — reason it was called out as untouchable).
8. [TREE] Re-apply VeeInsta strips in order: (a) privacy strip per `scratch-tmp/ag-v12.55-stage3-report.md` + inventory (conservative localhost/empty-constant pattern, verifier-safe); (b) nag/link purge per stage-5 (two choke points: `security/GoldSecrets` early-returns + `patches/Links`/`FollowDeveloper`/`WelcomeMessage` neutralization — endpoints are native-obfuscated so this is patch-around, not removal); (c) rebrand strings + launcher label per stage-5/CHANGELOG.
9. [TREE] Build/sign/install-verify: `apktool b` (expect EXIT=0 first try per stage-6), zipalign, same-key sign (key per `KEYSTORE-NOTES.md` — local-only, never committed), `adb install -r`, smoke: login, feed, download, translate, ghost toggles, crash-log writes local-only.
10. [BIN-gated] Anything touching `GoldSecrets` endpoints, OTA feed, font/emoji packs, or `initDcc` internals cannot be regenerated — only neutralized or kept. A future base bump additionally needs the author's new `.so` + new hook map; budget that as a fresh RE cycle, not a merge.

## 6. HONEST GAPS — what these artifacts cannot recover

- **No stock manifest/dex**: no pristine 439.0.0.37.89 APK in-tree, so manifest "deltas" above are mod-entry enumerations, not true diffs; permission-level additions (if any) unprovable. Source APKs live outside the repo (`~/Downloads/AGInsta_v12.50_0.apk`, `scratch-tmp/AGInsta_v12.55.apk` — mod builds, not stock).
- **Native internals**: `initDcc()` behavior, `GoldSecrets.decode()` arrays→URLs, and all 125 remaining JNI bodies are opaque machine code. `strings` shows no plaintext author hosts (stage-5) — endpoints are recoverable only at runtime (intercepted traffic / debugger), never statically.
- **Server side**: ftapi.pythonanywhere.com proxy logic, OTA feed content/signing, verification-list contents, font/emoji pack hosts — third-party infrastructure; if it goes dark, dependent features (V2 translate, self-update, remote fonts) degrade to their local fallbacks or die.
- **Keys/signing**: release key is local-only (`KEYSTORE-NOTES.md`); nothing here re-signs without it. Debug builds can't update over releases.
- **Obfuscation coupling**: all ~216 stock hunks reference R8-obfuscated `X/` names valid only for 439.0.0.37.89. Any base upgrade invalidates §2.2 wholesale — the single most expensive unknown.
- **`com.dcc` provenance**: author-toolchain shim; if a future mod drops/renames it, the attach recipe (§2.1) must be re-derived from the new donor — treat this report as version-pinned, not timeless.
