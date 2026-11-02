# Research log — technical architecture (spec 11)

Date: 2026-11-02. Author: tech-architecture agent. Feeds `docs/spec/11-technical-architecture.md`.

## Network conditions in this session (important for the build loop)

The cloud container's egress policy **denied** (HTTP 403 at CONNECT) these hosts:
`docs.expo.dev`, `expo.dev`, `api.expo.dev`, `u.expo.dev`, `exp.host`, `developers.cloudflare.com`,
`api.cloudflare.com`, `rive.app`, `hono.dev`, `appleid.apple.com`, `api.ngrok.com`,
`connect.ngrok-agent.com`, `tunnel.us.ngrok.com`, `playwright.azureedge.net`, `cdn.playwright.dev`.
Reachable: `registry.npmjs.org`, `api.github.com`, `raw.githubusercontent.com`, `api.anthropic.com`,
`platform.claude.com`.

Consequences:
- From this container, the agent **cannot** run `expo start --tunnel` (ngrok), `eas update`, `eas build`,
  `wrangler deploy` or `wrangler d1 ...` against the real account. These either run from **GitHub
  Actions** (open internet), or the parent adds the hosts to the cloud environment's *Allowed domains*.
  Required allowlist: see spec 11 §15.3.
- Playwright can still run locally: `/usr/bin/google-chrome` exists, so use `channel: 'chrome'`
  instead of downloading browsers.
- The WebSearch budget ran out mid-session. Facts not checked against a primary page are marked
  *verify* in the spec.

## Method

1. The npm registry (reachable) is the primary source for **exact versions**. The `expo@58.0.22`
   tarball's `bundledNativeModules.json` is the authoritative SDK 58 version map, which is what
   `npx expo install` uses.
2. Package tarballs were inspected for API facts (expo-localization plugin, expo-sqlite web, expo-audio,
   Skia web loader, Rive RN package contents, R3F peer deps).
3. Web search extracts were used for Apple and Expo behaviour that couldn't be fetched directly.
4. The bundled `claude-api` skill (cached 2026-10-06) was used for Anthropic SDK and API shapes.

## Findings

### Expo SDK 58 version map (from `expo@58.0.22/bundledNativeModules.json`, published 2026-10-31)
- react 19.3.0, react-native 0.88.1, react-native-web ~0.21.4, expo-router ~58.0.29,
  react-native-reanimated 4.7.1, react-native-worklets 0.13.0, react-native-gesture-handler ~3.3.0,
  @shopify/react-native-skia 2.14.0, react-native-screens ~4.30.1, safe-area-context ~5.10.1,
  react-native-svg 15.16.0, lottie-react-native ~7.5.0, @shopify/flash-list 2.3.2,
  expo-audio ~58.0.11, expo-haptics ~58.0.7, expo-sqlite ~58.0.15, expo-localization ~58.0.5,
  expo-camera ~58.0.14, expo-image ~58.0.18, expo-image-picker ~58.0.10, expo-image-manipulator ~58.0.16,
  expo-local-authentication ~58.0.2, expo-secure-store ~58.0.2, expo-crypto ~58.0.7,
  expo-file-system ~58.0.13, expo-notifications ~58.1.7, expo-updates ~58.0.20, expo-dev-client ~58.0.18,
  expo-gl ~58.0.4, expo-sensors ~58.0.4, expo-apple-authentication ~58.0.2, expo-font ~58.0.11,
  expo-asset ~58.0.19, expo-splash-screen ~58.0.12, expo-screen-orientation ~58.0.3,
  expo-keep-awake ~58.0.3, expo-network ~58.0.4, @sentry/react-native ~8.28.0, jest-expo ~58.0.13,
  eslint-config-expo ~58.0.4.
- The Expo default template for SDK 58 (`expo-template-default@58.0.31`) pins **typescript ~6.0.3**
  and @types/react ~19.3.0. TypeScript 7.0.2 (native compiler) is out, but Expo's template stays on 6.0.
  Decision: TS 6.0.3.
- Expo Go for SDK 58 includes Skia and Reanimated (Expo docs SDK 58 Skia/Reanimated pages, via search
  extract): https://docs.expo.dev/versions/v58.0.0/sdk/skia.md ,
  https://docs.expo.dev/versions/v58.0.0/sdk/reanimated/ ,
  https://docs.expo.dev/versions/v58.0.0/sdk/third-party-overview.md

### expo-localization plugin (tarball `plugin/src/withExpoLocalization.ts`)
- Options are `supportsRTL` (default true), `forcesRTL` (default false), `allowDynamicLocaleChangesAndroid`
  and `supportedLocales` (array, or `{ios, android}`).
- On iOS, `forcesRTL` writes `ExpoLocalization_forcesRTL=true` to Info.plist. It is a **native config
  plugin**, so it does nothing in Expo Go. Hence spec rule R-1: the app never relies on
  `I18nManager.isRTL`; the root sets `direction: 'rtl'` itself.

### expo-sqlite
- Web support uses **wa-sqlite (WASM) in a Worker with SharedArrayBuffer** (`web/worker.ts`,
  `web/WorkerChannel.ts`). The page must be served with `Cross-Origin-Opener-Policy: same-origin` and
  `Cross-Origin-Embedder-Policy: credentialless` (or `require-corp`). Metro must treat `.wasm` as an
  asset. Web support is labelled **alpha** in the docs: https://docs.expo.dev/versions/latest/sdk/sqlite.md
- The package ships `expo-sqlite/kv-store` (sync key-value API, also available as a localStorage
  polyfill). Decision: use it **instead of MMKV**, because MMKV v4 is a Nitro module and not in Expo Go.

### expo-audio (tarball `build/ExpoAudio.d.ts`, `Audio.types.d.ts`)
- API: `createAudioPlayer`, `useAudioPlayer`, `preload(source, {preferredForwardBufferDuration})`,
  `clearPreloadedSource`, `setAudioModeAsync({playsInSilentMode, interruptionMode:
  'mixWithOthers'|'duckOthers'|'doNotMix'|'doNotMixPersistent', shouldPlayInBackground,
  keepAudioSessionActive})`, `useAudioPlaylist`, `useAudioStream`.
- iOS is backed by AVPlayer/AVPlayerItem (the preload docs reference `preferredForwardBufferDuration`).
  Sub-50 ms SFX latency is **not guaranteed**: use a pool of preloaded players and measure on device.
  Fallback: `react-native-audio-api` 0.13.6 (Software Mansion Web-Audio-style engine; dev build only).

### Skia web
- The `setup-skia-web` bin copies `canvaskit.wasm` into `public/`. API: `LoadSkiaWeb(opts)` and
  `<WithSkiaWeb getComponent=... fallback=...>`. Playwright must wait until CanvasKit has loaded
  (spec 11 §12 uses a `data-skia-ready` attribute).

### Rive React Native (tarball `@rive-app/react-native@0.5.10`, README)
- "Rive React Native 2.0" is built on **Nitro**. It requires RN ≥ 0.78, Expo SDK ≥ 53, iOS 15.1+,
  Xcode 16.4+. Peer: `react-native-nitro-modules >=0.35.10 <0.37`. The README says RN ≥ 0.87 needs
  Nitro ≥ 0.36.2. **Pin nitro 0.36.5** (0.37.x falls outside the peer range).
- `@rive-app/canvas` is an *optional* peer, used by the `rive-gen-types` CLI that generates typed
  view-model bindings. **There are no `.web.*` implementation files in the package**, so web needs the
  separate `@rive-app/react-canvas` (4.37.1) via a `.web.tsx` platform split.
- Known issue: Xcode 27 + `use_frameworks! :linkage => :dynamic` crashes on iOS ≤ 17. Static linkage
  (the default) is fine. Keep static linkage.
- Not in Expo Go (custom native code). Use a placeholder implementation there.

### 3D candidates (npm metadata)
- `@react-three/fiber@9.8.2` peers: expo ≥43, expo-gl ≥11, expo-asset, expo-file-system, react
  ≥19 <19.4, react-native ≥0.78, three ≥0.156. Its native entry (`@react-three/fiber/native`) runs on
  **expo-gl, which ships in Expo Go**, and the same scene code runs on web/WebGL, which suits
  Playwright testing.
- `three@0.186.1` (≥ 7 days old), `@react-three/drei@10.7.9`.
- `@react-three/rapier@2.2.0` wraps Rapier **WASM**. Hermes has no WebAssembly (general knowledge,
  *verify in spike*), so Rapier is not expected to run on native. Physics fallback: `cannon-es@0.20.0`
  (pure JS, unmaintained since 2022) or a custom kinematic controller (recommended).
- `react-native-filament@1.11.0` depends on `react-native-worklets-core` (a different worklets runtime
  from Reanimated 4's `react-native-worklets`, so a conflict risk). It has no web build and isn't in
  Expo Go.
- `react-native-wgpu@0.5.17` (WebGPU) is experimental; noted only.
- Texture/mesh compression: KTX2/Basis, Draco and meshopt decoders all use WASM, so the same Hermes
  constraint applies. Use quantized glTF (KHR_mesh_quantization, no decoder needed) plus PNG/JPEG
  textures. *Verify in spike.*

### Apple free personal team (search extracts, forums and third-party; not Apple's membership page)
- Free provisioning profiles last **7 days**, after which the app won't launch until it is re-installed.
  Sources: https://flaviocopes.com/install-iphone-apps/ ,
  https://takazudo-zudo-tauri.pages.dev/pj/zudo-tauri/docs/mobile/ios-signing-free-team
- Limits: about **3 free-team apps per device**
  (https://anexplorer.io/devmode/apple/iphone), App IDs that expire quickly, and a device-registration
  cap with no self-service reset (https://developer.apple.com/forums/thread/843534 ). Third-party tooling
  notes that Push Notifications and App Extensions capabilities are unavailable on personal teams
  (https://www.rubydoc.info/gems/unlimit ).
- Sign in with Apple on a personal team: **no source found in this session**. Apple's membership
  comparison historically lists SIWA, Push, App Attest/DeviceCheck and iCloud as paid-program
  capabilities (general knowledge, *verify*). The spec assumes SIWA is **unavailable** until the paid
  program and designs email-code login plus bootstrap codes as the primary path.
- Device setup: Developer Mode lives at Settings → Privacy & Security → Developer Mode. Accept the
  Trust prompt over cable, then trust the developer under Settings → General → VPN & Device Management.
  `xcrun devicectl list devices`, `xcrun devicectl device install app --device <id> <path.app>`, and
  `devicectl device info details` reports `transportType` (localNetwork or wired). Source:
  https://flaviocopes.com/install-iphone-apps.md , https://docs.hex-rays.com/core/debugger/tutorials/ios_debugging_coredevice
- Implication (inference): **don't change the signing team** later without first syncing data. A
  different team prefix can force a delete and reinstall, which wipes on-device SQLite.

### Expo Go + EAS Update
- `qr.expo.dev` produces QR codes for EAS Updates (by project, channel, branch, group or update) that
  open in a development build or Expo Go: https://docs.expo.dev/more/qr-codes . The EAS preview GitHub
  Action has a `qr-target` option (dev-build or expo-go).
- Inference (*verify*): for Expo Go, the update's runtimeVersion must match Expo Go's SDK runtime, i.e.
  `runtimeVersion: {policy: 'sdkVersion'}` during the Expo Go phase.

### Cloudflare
- Durable Objects WebSocket **Hibernation**: no duration (GB-s) charges accrue while hibernated, and
  incoming WS messages count as requests (1M included on Paid, then about $0.15/M per a mirror).
  SQLite-backed DO storage uses D1-like row pricing:
  https://developers.cloudflare.com/durable-objects/platform/pricing ,
  https://developers.cloudflare.com/durable-objects/best-practices/websockets/index.md
- D1/R2/Workers prices and limits come from `backend_hosting_services.md`: D1 Time Travel 30 days on
  Paid; R2 $0.015/GB-month, 10 GB free, no egress fees.

### Anthropic (claude-api skill, cached 2026-10-06)
- IDs and prices: `claude-opus-5-5` $4/$20, `claude-sonnet-5-5` $2/$10, `claude-haiku-5-5` $0.10/$0.50
  (≤100K-token prompts). Batch is 50% off.
- Structured outputs: `output_config: { format: { type: 'json_schema', schema } }`, or
  `client.messages.parse()` with `zodOutputFormat()` from `@anthropic-ai/sdk/helpers/zod`. Schema limits:
  no numeric min/max, no string length limits, minItems only 0/1, `additionalProperties: false`
  required, no recursion.
- Opus 5.5: thinking can't be disabled; effort defaults to `medium` (set it explicitly). Forced
  `tool_choice` returns 400, so use structured outputs. No assistant prefill. The server-side
  `fallbacks: "default"` (beta `server-side-fallback-2026-07-01`) is recommended on Opus 5.5 / Sonnet 5.5
  realtime calls and **rejected on the Batches API**.
- Batches: `client.messages.batches.create({requests:[{custom_id, params}]})`, then
  `retrieve(id).processing_status === 'ended'`, then `for await (r of await batches.results(id))`.
  Results come back in any order (key by `custom_id`), within 24 h max, and are kept for 29 days.
- The SDK is fetch-based, so it runs on Workers. Set `timeout` in ms; stream long outputs.
- From 2026-10-21, `temperature`/`top_p`/`top_k` return 400 on every model. Don't send them.

### Versions pinned (newest release ≥ 7 days old as of 2026-11-02, via npm `time`)
hono 4.13.14 · wrangler 4.152.0 · @anthropic-ai/sdk 1.3.0 · jose 6.2.13 · zod 4.6.7 ·
@hono/zod-validator 0.9.1 · drizzle-orm 0.45.4 · drizzle-kit 0.31.11 · zustand 5.0.16 ·
@tanstack/react-query 5.104.5 · intl-messageformat 12.1.4 · @playwright/test 1.64.1 · vitest 4.1.12
(5.x is days old) · @cloudflare/vitest-pool-workers 0.23.0 · @cloudflare/workers-types 5.20261025.1 ·
jest 30.5.2 · @testing-library/react-native 14.1.0 · eslint 10.13.0 · typescript-eslint 8.72.0 ·
prettier 3.9.10 · pnpm 10.34.9 · eas-cli 25.3.0 · @rive-app/react-native 0.5.5 (0.5.10 is from today;
bump after on-device check) · react-native-nitro-modules 0.36.5 · @rive-app/react-canvas 4.37.1 ·
three 0.186.1 · @react-three/fiber 9.8.2 · @react-three/drei 10.7.9 · @types/three 0.186.0 ·
fast-check 4.10.2 · @noble/hashes 2.4.0 · uuidv7 1.2.1 · aws4fetch 1.0.20 · @gltf-transform/cli 4.5.1 ·
serve 14.2.6 · lefthook 2.2.2 · react-native-audio-api 0.13.6 (optional) ·
react-native-vision-camera 5.2.3 (later phase).

## Open items to verify (on first chance)
1. SIWA/App Attest/Push on a free personal team (expected: unavailable).
2. Face ID via expo-local-authentication inside Expo Go (expected: may be unavailable; passcode fallback).
3. Expo Go loading EAS Updates with the `sdkVersion` runtime policy.
4. R3F native on expo-gl at 60 fps on the family's oldest iPad; Hermes and WASM (Rapier/Draco/KTX2).
5. expo-audio tap-to-sound latency (p95 target ≤ 50 ms).
6. Hermes `Intl.PluralRules` for `he` (intl-messageformat plural/select). Add a polyfill if missing.
7. Exact free-team limits (apps per device, App IDs per 7 days).
8. Whether `xcrun devicectl device install` over Wi-Fi works while the device is locked.
