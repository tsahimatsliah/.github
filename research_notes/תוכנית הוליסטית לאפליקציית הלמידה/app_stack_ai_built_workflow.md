# App stack + AI-built workflow for a Hebrew-first, animated kids' learning game (iPhone + iPad, family-only, agent on Linux) — Nov 2026

Research note: in this session WebFetch could not reach expo.dev or docs.github.com (DNS blocked by the sandbox network allowlist), so the findings below come from search-result extracts of those primary pages plus secondary sources. Figures marked "verify" should be checked on the live page before money is spent.

## Q1. Option A — Native Swift (SwiftUI + SpriteKit): can a Linux-based agent build and ship it?

### Takeaway
Possible, but the agent would be working blind. Every compile, UI check and signed build has to go through a macOS CI runner (Xcode Cloud's 25 free hours, or GitHub's macOS runners at about $0.062/min). There's no way to preview inside the container, so each UI change costs a full CI round trip of 10 minutes or more. That gives the best polish ceiling and the worst iteration loop for this setup. Not recommended as the primary path.

### Cited Findings
- Every Apple Developer Program membership includes 25 Xcode Cloud compute hours/month (since January 2024). Unused hours don't roll over. 100 hours/month costs US$49.99/month. It needs Xcode 15+ and a program membership. — [Apple Developer News](https://developer.apple.com/news?id=ik9z4ll6); [Apple: Get started with Xcode Cloud](https://developer.apple.com/xcode-cloud/get-started)
- GitHub-hosted macOS runners (3–4 core, M1/Intel) cost $0.062/min after a January 2026 price cut (from $0.080). Linux is $0.006/min, so macOS is roughly 10x Linux. The Free plan includes 2,000 minutes/month, but the search didn't show what multiplier macOS minutes burn against that pool (historically 10x, so roughly 200 macOS minutes; verify). — [GitHub Docs: actions runner pricing](https://docs.github.com/en/enterprise-server@3.17/billing/reference/actions-runner-pricing); [DEV: GitHub Actions changed pricing in 2026](https://dev.to/shubhamoriginx/github-actions-changed-pricing-in-2026-heres-exactly-what-youre-paying-now-51db); [cicdcalculator](https://cicdcalculator.com/github-actions-pricing-changes-2026)
- GitHub's planned $0.002/min self-hosted runner fee was postponed indefinitely in December 2025 (third-party claim). — [cicdcalculator](https://cicdcalculator.com/github-actions-pricing-changes-2026)
- Codemagic gives individual (non-team) accounts 500 free macOS M2 minutes/month, then $0.095/min. Linux and Windows builds have no free minutes. — [Codemagic docs: pricing](https://docs.codemagic.io/billing/pricing/)

### Inferences
- Rough loop: the agent edits Swift → pushes → a macOS CI job builds and runs snapshot tests (e.g. swift-snapshot-testing on an iOS simulator) → PNGs upload as artifacts → the agent downloads them with `gh run download` and inspects them. That works, but each loop takes about 8–20 minutes, and 25 Xcode Cloud hours covers roughly 75–150 builds a month. That's fine for releases and painful for polishing animation.
- I couldn't confirm in this session that SwiftUI and SpriteKit are unavailable in Swift on Linux. The general understanding is that they're Apple-only frameworks, so the agent couldn't even type-check UI code locally. That makes AI-written SwiftUI riskier, because compile errors only show up in CI.
- You'd still need fastlane (match + pilot) or Xcode Cloud's TestFlight post-action to ship. That's more moving parts than EAS for a non-iOS-engineer parent.

### Gaps
- I couldn't fetch the official GitHub page on the macOS included-minute multiplier for private repos (docs.github.com was blocked).
- I found no source benchmarking simulator snapshot-test runtimes on GitHub or Xcode Cloud in 2026.

## Q2. Option B — Expo / React Native: state in Nov 2026, build/OTA/distribution, RTL, animation stack, testability from Linux

### Takeaway
This is the best fit. The current Expo SDK 58 (React Native 0.88, iOS 27 targeted) builds iOS apps in the cloud with no Mac (EAS Build: 15 free iOS builds/month). It pushes JS/asset changes over the air (EAS Update: 1,000 MAU free; the family needs about 5). The same code exports to web, so the agent can render and screenshot it in Chromium/Playwright inside the container. RTL works but has real, version-specific bugs, so pin versions and test RTL on a real device every release.

### Cited Findings
**SDK and versions**
- SDK 55 (React Native 0.83, React 19.2) removed the Legacy Architecture, so the New Architecture is mandatory and `newArchEnabled` is gone. Package versions now track the SDK major (e.g. expo-camera ^55.0.0). Hermes v1 is opt-in in 55. — [Expo changelog SDK 55](https://expo.dev/changelog/sdk-55)
- SDK 56 (May 21, 2026): React Native 0.85. Expo UI SwiftUI/Jetpack Compose APIs are stable. The heaviest iOS modules ship as prebuilt XCFrameworks, which speeds up iOS builds. There's a known Hermes V1 memory regression affecting `react-native-worklets`/`react-native-reanimated`. — [Expo changelog SDK 56](https://expo.dev/changelog/sdk-56.md)
- SDK 57 (June 30, 2026): React Native 0.86, a small non-breaking release, with reanimated/worklets/gesture-handler bumped. The initial 57.0.9 had a Hermes V1 regression that could sharply increase memory use with reanimated/worklets, fixed in a later update. — [Expo changelog SDK 57](https://expo.dev/changelog/sdk-57); [releases.sh](https://releases.sh/release/rel_aH7NaXbDXMlY2_I-Ak0D0)
- SDK 58 (current, autumn 2026): React Native 0.86 → 0.88. Targets iOS 27. Prebuild supports the required UIKit scene-based lifecycle. Strict TypeScript API is the default. New Expo Go supports only SDK 58. Upgrade with `npx expo install expo@^58.0.0 --fix` then `npx expo-doctor@latest`. — [Expo blog: how to upgrade to SDK 58](https://expo.dev/blog/how-to-upgrade-to-sdk-58); [Expo changelog SDK 58 beta](https://expo.dev/changelog/sdk-58-beta); [buildmvpfast (third-party)](https://www.buildmvpfast.com/blog/expo-sdk-58-ios-27-react-native-088-breaking-changes-2026)

**Cloud build, OTA, cost**
- EAS Free: up to 15 iOS + 15 Android builds/month in a low-priority queue with a 45-minute timeout. Starter is $19/month with $45 of build credit and a high-priority queue. Older sources say "30 total, 15 iOS". — [newly.app guide (Oct 2026)](https://newly.app/guides/build-an-ios-app-without-a-mac); [vibecoding.app EAS review 2026](https://vibecoding.app/blog/expo-eas-review); [Expo changelog 2023 free-plan limits](https://expo.dev/changelog/2023/08-01-eas-free-plan-limits)
- EAS Update Free: 1,000 monthly active users, a hard ceiling with no overage on Free. An MAU is a unique user who downloads at least one update in a period. — [Expo docs: plans](https://docs.expo.dev/billing/plans/); [Expo billing FAQ](https://docs.expo.dev/billing/faq)
- EAS Submit pushes to TestFlight. Expo's TestFlight doc notes a limit of 30 devices per internal tester. — [Expo docs: submit to TestFlight](https://docs.expo.dev/submit/testflight.md)

**RTL**
- The expo-localization config plugin supports `supportsRTL` and `forcesRTL` (written to Info.plist / Android resources). `forcesRTL: true` forces RTL regardless of device language, and the docs suggest it for RTL-only apps. It requires a native rebuild. Dynamic overrides must happen in app code and don't work in Expo Go. — [Expo docs: localization guide](https://docs.expo.dev/guides/localization/); [Expo SDK 58 localization reference](https://docs.expo.dev/versions/v58.0.0/sdk/localization/)
- Known RTL bugs in the New Architecture era:
  - RN 0.84 on iOS: ScrollView/FlatList content goes invisible on a second visit to a screen while `forceRTL(true)` is active (0.83.2 was fine). — [facebook/react-native#55821](https://github.com/facebook/react-native/issues/55821)
  - `<Text>` without an explicit `textAlign` aligns right in RTL on Android but not on iOS. A merged PR fixes this regression. — [react-native PR #57634](https://github.com/react/react-native/pull/57634)
  - A `forceRTL` change needs a full restart. — [RN #45661](https://gh.hhj520.top/react/react-native/issues/45661)
  - react-native-slider follows device locale instead of forceRTL on iOS New Architecture. — [react-native-slider PR #803](https://download.plaud.ai/callstack/react-native-slider/pull/803/{{%20revealButtonHref%20}})
  - react-native-screens native elements don't follow I18nManager. — [react-native-screens#1884](https://github.com/software-mansion/react-native-screens/issues/1884)

**Animation and graphics stack**
- The new Rive runtime `@rive-app/react-native` (built on Nitro Modules, replacing legacy `rive-react-native`) is generally available. It requires RN 0.78+, Expo SDK 53+, iOS 15.1+ and `react-native-nitro-modules` 0.25.2+, and uses a `useRiveFile` + `RiveView` API. — [Rive docs: React Native](https://rive.app/docs/runtimes/react-native); [Rive migration guide](https://rive.app/docs/runtimes/react-native/migration-guide)
- Margelo (who wrote it) reports up to 94x faster multi-view load, 4.7x lower memory and 3.2x faster property writes than the legacy SDK. On their device, four views of one file went from about 500 ms to about 58 ms. These are vendor-adjacent numbers. — [Margelo blog, Jul 9 2026](https://blog.margelo.com/rewriting-rive-react-native-with-nitro-modules)
- react-native-skia runs on web through CanvasKit (WASM). It needs the `setup-skia-web` script (re-run on each upgrade) or a version-matched CDN load, and `LoadSkiaWeb()` before render. Gate rendering until CanvasKit loads (this matters for screenshot timing). — [Shopify react-native-skia: web](https://shopify.github.io/react-native-skia/docs/getting-started/web); [Expo docs: Skia](https://docs.expo.dev/versions/latest/sdk/skia)
- Reanimated runs on web, but entirely in JS there, so it's less efficient. The Babel plugin is auto-configured by `babel-preset-expo`. — [Reanimated docs: web support](https://docs.swmansion.com/react-native-reanimated/docs/3.x/guides/web-support)
- Playwright's `toHaveScreenshot()` does baseline-image visual regression. I found no Expo/Skia-specific Playwright guide. — [DEV: visual testing with Playwright](https://dev.to/dwwhalen/visual-testing-with-playwright-3fhh)
- Expo web builds can be deployed with EAS Hosting in two commands, which gives a shareable preview URL. — [Expo blog: EAS Hosting](https://expo.dev/blog/deploying-an-expo-app-to-web-with-eas-hosting)

### Inferences
- Concrete package set (SDK 58; let `npx expo install` pick versions):
  - App shell: `expo`, `expo-router`, `expo-dev-client`, `expo-updates`, `expo-localization` (plugin with `supportsRTL: true, forcesRTL: true`), `expo-font`, `expo-image`, `expo-asset`, `expo-splash-screen`, `expo-screen-orientation`, `expo-keep-awake`
  - Motion and gesture: `react-native-reanimated` 4.x + `react-native-worklets`, `react-native-gesture-handler`
  - Graphics: `@shopify/react-native-skia` (drawing, letter tracing, particles, game canvas), `@rive-app/react-native` + `react-native-nitro-modules` (character animation and state machines), `lottie-react-native` only if a specific asset needs it
  - Device APIs: `expo-haptics`, `expo-audio`, `expo-speech` (Hebrew TTS uses iOS system voices; quality is device-dependent, so pre-recorded parent or kid voice clips are better for core content), `expo-camera` (photo-to-game; vision-camera only if you need frame processors)
  - Storage: `expo-sqlite` or MMKV for progress
  - Testing: Jest + `@testing-library/react-native` for logic, Playwright for web visuals
- Skip react-native-game-engine (old, unmaintained pattern). Build mini-games as a Skia `<Canvas>` driven by a Reanimated `useFrameCallback` loop. That gives UI-thread 120 Hz on ProMotion devices (inference: native-thread rendering isn't bound by the JS thread).
- For web previews, Rive may need a platform split (`RiveCharacter.web.tsx` using `@rive-app/react-canvas`). I didn't confirm whether the Nitro runtime supports web. Plan for the split so Playwright screenshots stay faithful.
- RTL strategy: set `forcesRTL: true` globally (Hebrew is the default). Wrap English-learning screens in `<View style={{direction:'ltr'}}>`. Always set `textAlign` and `writingDirection` explicitly on `Text` to dodge the iOS/Android divergence. On web, set `<html dir="rtl" lang="he">`. Avoid stock sliders and native headers that ignore I18nManager.
- Pin Expo SDK and patch versions, and watch the Hermes V1 + reanimated memory regressions seen in SDK 56/57. If memory spikes, stay on the default engine setting until it's fixed.
- 15 free iOS builds/month is plenty, because most iterations ship as OTA updates (`eas update --channel family`). A native rebuild is only needed when native deps or config change.

### Gaps
- I couldn't fetch the live expo.dev/pricing page to confirm 15 vs 30 free builds and the current Starter credit.
- Not confirmed: whether `@rive-app/react-native` (Nitro) renders on react-native-web.
- Not confirmed: whether EAS Workflows can run Maestro iOS-simulator E2E on the Free plan.
- No 2026 benchmark found comparing RN, Flutter and Swift frame pacing for animation-heavy apps.

## Q3. Option C — Flutter (and Option D — Godot/Unity)

### Takeaway
Flutter is a strong runner-up. RTL comes built in through `Directionality`/localization (not re-verified here), Impeller renders on iOS, Flame + Rive is mature for 2D, Codemagic gives 500 free M2 minutes, and a web build allows Chromium previews. It loses to Expo here on two counts: there's no first-party OTA update service comparable to EAS Update, and the parent's web-UX skills carry over less. Godot and Unity need a Mac (or macOS CI) for iOS export, and their web exports aren't faithful for Hebrew text and UI. Reject them for this app.

### Cited Findings
- Flame is a 2D engine on top of Flutter, rendering through Flutter's Impeller on iOS. One vendor source dates version 1.35.0 to April 2026. It's 2D-strong and 3D-early. — [DEV: Flutter game dev 2026](https://dev.to/synfinity-dynamics-pvt-ltd/flutter-game-development-in-2026-can-you-build-real-games-with-flutter-peh); [dianapps guide](https://dianapps.com/blog/?p=14475)
- `flame_rive` (published by flame-engine.org, MIT) integrates Rive into Flame. The Rive Flutter runtime was actively maintained (updated May 2026). — [pub.dev flame_rive](https://pub.dev/packages/flame_rive); [fluttergems: game development](https://fluttergems.dev/game-development/)
- Codemagic: 500 free macOS M2 minutes/month for personal accounts, then $0.095/min. — [Codemagic docs](https://docs.codemagic.io/billing/pricing/)
- Kids-category apps face extra App Store restrictions (ads, data collection, external links, third-party SDKs). This doesn't apply to a non-public TestFlight/ad hoc app. — [kidspark tech stack post](https://portfolio-thuan.pages.dev/blog/kidspark-tech-stack-flutter-react-native-part4)

### Inferences
- Flutter's web build uses CanvasKit/Skwasm, so Playwright screenshots work but text isn't DOM-selectable. The agent would test through screenshots and Flutter integration tests rather than DOM selectors, which is a weaker autonomous loop than Expo web, where the DOM can be queried.
- Shorebird offers Flutter code push, but it's another vendor and paid tier (not researched here).

### Gaps
- No 2026 source found on Flutter Hebrew RTL or nikud quality. My general understanding (unverified this session) is that Flutter's RTL support is first-class.

## Q4. Option E — PWA on iPad/iPhone home screen

### Takeaway
Fine for throwaway concept sketches. As the product, or even as the main prototype, it's a dead end: there are no haptics, iOS 26 home-screen PWAs have reported audio failures, and there's no true fullscreen. Since the Expo web export is already a PWA-capable preview, Option B gives you this for free.

### Cited Findings
- iOS Safari, and every iOS browser (all WebKit), doesn't support the Vibration API, so web haptics aren't possible. — [web-haptics browser support](https://www.mintlify.com/lochie/web-haptics/advanced/browser-support); [Progressier: Vibration API](https://progressier.com/pwa-capabilities/vibration-api)
- iOS 26 home-screen PWAs: developers report audio works on first launch, then fails on later launches until a reboot. The reports say this started with the iOS 26 betas and isn't fixed (anecdotal). — [MacRumors forum](https://forums.macrumors.com/threads/ios-26-audio-issues-in-pwa-web-apps.2466839)
- iOS PWAs: no `beforeinstallprompt`, no true Fullscreen API, a status bar that can't be hidden, and partial push (16.4+, outside the EU). — [MagicBell PWA iOS guide](https://www.magicbell.com/blog/pwa-ios-limitations-safari-support-complete-guide)

### Inferences
- For a kids' game where touch feedback, sound and haptics are core to the experience, these gaps rule PWA out as the shipping target.

### Gaps
- I didn't verify Apple Pencil pressure/tilt (PointerEvent) support in iOS 26/27 Safari.

## Q5. Cross-cutting: Hebrew nikud, distribution to family devices, and the recommended pipeline

### Takeaway
Nikud quality depends on the font and the shaping engine (CoreText on iOS), not on the framework. Pick a font with proper mark positioning (GPOS) and test a nikud sample on a real device. For distribution, use **EAS internal distribution (ad hoc)** for the kids' devices, since they can't be App Store Connect users. TestFlight internal testing for the parents is optional. **EAS Update** delivers day-to-day changes over the air.

### Cited Findings
- Nikud rendering bugs historically come from fonts and shaping engines: Mozilla bugs on Microsoft Hebrew fonts, a HarfBuzz regression that rendered marks as spacing characters, and an iOS 11.3 WebKit vowel bug fixed in iOS 12. — [WebKit bug 189269](https://bugs.webkit.org/show_bug.cgi?id=189269); [Mozilla bug 60546](https://bugzilla.mozilla.org/show_bug.cgi?id=60546); [HarfBuzz commit](https://ara-mdk.googlesource.com/platform/external/harfbuzz/+/ca4c6948f69f807967f91d0e995716dd0f421dc5)
- TestFlight: up to 100 internal testers, who must be App Store Connect users. Builds expire 90 days after upload. — [App Store Connect Help: add internal testers](https://developer-rno.apple.com/help/app-store-connect/test-a-beta-version/add-internal-testers); [TestFlight overview](https://developer-rno.apple.com/help/app-store-connect/test-a-beta-version/testflight-overview)
- Ad hoc distribution is capped at 100 registered device UDIDs. A third-party blog claims TestFlight internal testers don't count against that cap. — [Kodeco: iOS distribution, TestFlight chapter](https://www.kodeco.com/books/ios-app-distribution-best-practices/v1.0/chapters/6-testflight); [Drizz blog](https://www.drizz.dev/post/testflight-access)

### Inferences
**Decision.** Primary path: Expo SDK 58 + EAS (Build, Update, Submit), with the Expo web export + Playwright as the agent's eyes. Prototype path: the same repo's web export hosted on EAS Hosting or GitHub Pages and opened in iPad Safari. That's a zero-install prototype, with no separate PWA codebase.

**Repo structure** (single Expo app, no monorepo needed):
- `app/` — expo-router routes: `(home)`, `games/[id]`, `english/` (LTR-wrapped), `parent/`
- `src/games/<game>/` — the Skia canvas, Reanimated loop, pure-TS game logic (unit-testable)
- `src/ui/` — design tokens and components (RTL-aware)
- `src/i18n/` — Hebrew strings, with nikud stored in the strings
- `assets/rive/*.riv`, `assets/audio/*.m4a`, `assets/fonts/` (a Hebrew font with good nikud GPOS, e.g. Noto Sans/Serif Hebrew or a licensed kids' font)
- `tests/unit/` (Jest)
- `e2e-web/` (Playwright specs + screenshot baselines)
- `eas.json` profiles: `development` (dev client, internal), `preview` (internal, channel `family`), `production` (store/TestFlight)
- `.github/workflows/`: `ci.yml` (Linux: typecheck, lint, Jest, `expo export -p web`, Playwright screenshots uploaded as artifacts), `ota.yml` (on merge to main: `eas update --channel family`), `native.yml` (manual or on native-dep change: `eas build -p ios --profile preview`)

**Agent loop inside the Linux container:**
1. Edit code.
2. Run `tsc` and `jest`.
3. Run `npx expo export -p web` (or `expo start --web`) and drive it with Playwright in Chromium at iPhone and iPad viewports with `dir=rtl`. Take screenshots and read them back as images to self-review.
4. Commit and open a PR. CI repeats the checks and posts screenshots, so the parent reviews visuals in the PR.
5. Merge, which publishes an OTA update. Devices pick it up on next launch.

Native-only behavior (haptics, Rive Nitro, camera, real 120 Hz feel, nikud on CoreText) gets checked by the parent on a device after a `preview` build.

**What the parent sets up, once:**
1. Join the Apple Developer Program ($99/yr, individual).
2. Create an Expo account (Free plan).
3. Create an App Store Connect API key (Issuer ID, Key ID, .p8) and give it to EAS so the agent can build and submit non-interactively. Store `EXPO_TOKEN` and the ASC key as GitHub secrets.
4. Register each family iPhone/iPad UDID through EAS's device registration flow (`eas device:create` generates a link/QR to open on each device). Turn on Developer Mode on iOS 16+ devices.
5. Install preview builds from the EAS build page link/QR. Optionally add the two parents as TestFlight internal testers.
6. Choose and license the Hebrew font, the Rive files (Rive editor account) and voice recordings.

**Expected cost:** $99/yr Apple. Expo Free, since about 5 MAU is far under the 1,000 cap and a few native builds a month fit inside 15. GitHub Actions Linux minutes only (negligible or free). Optionally $19/month EAS Starter if the free queue gets too slow. No Mac needed.

**Main risks:**
- RTL regressions per RN version. Mitigate with pinned versions and an RTL screenshot suite.
- Hermes V1 + reanimated memory regressions.
- Ad hoc provisioning profiles must be regenerated when devices are added and yearly (inference based on the standard Apple certificate and profile lifecycle).
- Web previews diverge from native for Rive and haptics.

### Gaps
- Not confirmed: Apple's current rule on whether the 100-device ad hoc limit applies per device family.
- Not confirmed: the exact `eas device:create` flow and Developer Mode requirements on iOS 27 (expo.dev docs couldn't be fetched).
- No 2026 side-by-side test found of nikud quality in RN Text vs Flutter Text vs SwiftUI Text. All use platform or engine shaping, so test a sample string on a real device early.
- Hebrew quality of `expo-speech` (iOS system TTS voices) wasn't evaluated.
