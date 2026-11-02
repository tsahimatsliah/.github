# Research log — Fable reviewer F8 (spec 14, performance & quality playbook) — 2026-11-02

## Method

Doc sites named in the research notes (expo.dev, docs.swmansion.com, shopify.github.io, r3f.docs.pmnd.rs, unpkg.com)
are unreachable from the container (proxy 403 / DNS). The **npm registry is reachable**, so facts were taken from the
published tarballs of the versions pinned in 11 §2.3, downloaded with `npm pack` into an isolated scratchpad folder and
read with `grep`/`sed` only (never executed). Nothing from those tarballs was copied into the repo.

Packages inspected: `react-native@0.88.1`, `react-native-worklets@0.13.0`, `react-native-reanimated@4.7.1`,
`expo-audio@58.0.11`, `expo-sqlite@58.0.15`, `expo-image@58.0.18`, `expo-font@58.0.11`, `expo-splash-screen@58.0.6`,
`expo-battery@58.0.2`, `@shopify/react-native-skia@2.14.0`, `@react-three/fiber@9.8.2`, `reassure@1.6.1`; `npm view`
for `@rive-app/react-native@0.5.5`, `@stryker-mutator/*`, `react-native-worklets`, `react-native-reanimated`.

## Findings (primary evidence = file in tarball)

1. **Hermes V1 in RN 0.88.1 = `260318099.0.5`** — `sdks/.hermesv1version`, `sdks/hermes-engine/version.properties`,
   `package.json` dep `hermes-compiler: 260318099.0.5`. This is above the community-reported fix line
   `250829098.0.17` for the SDK 56/57 worklets memory regression, so the allow-list floor is the bundled version
   itself (14 `PQ-CI-03`).
2. **`InteractionManager` removed** — `index.js` lines ~431–440 define a dev-only getter that throws
   `invariant(false, 'InteractionManager has been removed from react-native core…')`. `Libraries/Interaction/` no longer
   contains it. `requestIdleCallback` is installed by `Libraries/Core/setUpTimers.js`.
3. **`AppState` `memoryWarning`** still exists in 0.88.1 (`Libraries/AppState/AppState.js`).
4. **worklets 0.13.0 exports** (`lib/typescript/index.d.ts`): `scheduleOnRN`, `scheduleOnUI`, `runOnJS`, `runOnUI`,
   `runOnUIAsync`, `runOnUISync`, `executeOnUIRuntimeSync`, `createWorkletRuntime`, `runOnRuntime`. Registry latest is
   0.13.1 (2026-11-01, < 7 days → 0.13.0 stays pinned under the Renovate rule).
5. **Reanimated 4.7.1 static flags** (`src/featureFlags/staticFeatureFlags.ts`): defaults
   `ANDROID_SYNCHRONOUSLY_UPDATE_UI_PROPS:false, IOS_SYNCHRONOUSLY_UPDATE_UI_PROPS:false, IOS_CSS_CORE_ANIMATION:false,
   USE_COMMIT_HOOK_ONLY_FOR_REACT_COMMITS:true, ENABLE_SHARED_ELEMENT_TRANSITIONS:false,
   FORCE_REACT_RENDER_FOR_SETTLED_ANIMATIONS:true, USE_ANIMATION_BACKEND:false, EXPERIMENTAL_CSS_ANIMATIONS_FOR_SVG_COMPONENTS:true`.
   Config: `scripts/reanimated_utils.rb` reads `package.json` → `reanimated.staticFeatureFlags` and rejects
   `USE_ANIMATION_BACKEND` + `FORCE_REACT_RENDER_FOR_SETTLED_ANIMATIONS`, and `USE_LEGACY_LAYOUT_ANIMATIONS_PROXY` +
   `ENABLE_SHARED_ELEMENT_TRANSITIONS`. The research note's claim that `USE_COMMIT_HOOK_ONLY_FOR_REACT_COMMITS` must be
   enabled is outdated (already default).
6. **expo-audio 58.0.11** (`build/ExpoAudio.d.ts`, `Audio.types.d.ts`, `AudioModule.types.d.ts`): `setAudioModeAsync(Partial<AudioMode>)`
   with `playsInSilentMode`, `interruptionMode: 'mixWithOthers'|'doNotMix'|'doNotMixPersistent'|'duckOthers'`,
   `allowsRecording`, `shouldPlayInBackground`, `shouldRouteThroughEarpiece`; `createAudioPlayer(source, {updateInterval})`;
   `AudioPlayer` has `play()`, `pause()`, `seekTo(seconds, tolBefore?, tolAfter?)`, `replace()`, `remove()`, `volume`,
   `loop`, `setPlaybackRate()` — **no `release()`**; module-level `preload(source, {…buffer options})`,
   `clearPreloadedSource`, `clearAllPreloadedSources`, `getPreloadedSources`, `setIsAudioActiveAsync`.
7. **expo-sqlite 58.0.15**: `withTransactionAsync`, `withExclusiveTransactionAsync(txn => …)`, `execAsync`, `runAsync`,
   `getAllAsync`, `prepareAsync`; subpath exports `./kv-store` and `./localStorage/install`.
8. **expo-image 58.0.18**: `Image.prefetch(urls, cachePolicy | options)`, `Image.clearMemoryCache()`, `clearDiskCache()`,
   `getCachePathAsync()`, `loadAsync()`; props `transition` (duration default 0), `placeholder`, `placeholderContentFit`.
9. **expo-font 58.0.11 plugin**: `{ fonts?: string[]; android?: { fonts }; ios?: { fonts } }`.
10. **expo-splash-screen 58.0.6**: `preventAutoHideAsync()`, `hideAsync()`, `hide()`, `setOptions({ duration, … })`.
11. **expo-battery 58.0.2**: `isLowPowerModeEnabledAsync()`, `useLowPowerMode()`, `getPowerStateAsync()`, `useBatteryLevel()`.
12. **Skia 2.14.0** exports `Atlas`, `useTexture`, `useRSXformBuffer`, `useRectBuffer`, `useImage`, `createPicture`,
    `usePictureAsTexture`, `makeOffscreenSurface`, `drawOffscreen`, `drawAsImage`, `drawAsImageFromPicture`, `useCanvasRef`.
13. **R3F 9.8.2**: `native/` entry and `src/native/Canvas.d.ts` exist; store exposes `frameloop`, `invalidate(frames?)`,
    `setFrameloop`, `setDpr`, `flat`. Peers: `react >=19 <19.4`, `three >=0.156`, `expo-gl >=11`, `expo-asset >=8.4`,
    `expo-file-system >=11`, `react-native >=0.78`.
14. **Rive 0.5.5** peers: `react-native-nitro-modules >=0.35.10 <0.37`, `@rive-app/canvas ^2.39.0` (web split confirmed as
    a separate package).
15. **Reassure**: the npm package is `reassure` (1.6.1, 2026-10-21), binaries under `@callstack/reassure-cli`; API
    `measureRenders`, `measureFunction`, `measureAsyncFunction`; CLI `reassure --baseline` then `reassure`; default
    `testMatch` `**/__perf__/**`, `*.perf.test.*`. `@callstack/reassure` does not exist (404).
16. **Stryker** 10.1.1 for `@stryker-mutator/core`, `vitest-runner`, `jest-runner` (2026-10-23).
17. **Maestro**: `maestro-cli` on npm (1.1.10) is **not** the official Maestro; official install is the shell script.
    Version left ⟂VERIFY on the Mac.

## Not verifiable here (listed in spec 14 §13)

Apple hitch thresholds, `xctrace`/`devicectl` exact flags, expo-gl MSAA/extensions, iPadOS Split View resize events,
Low Power Mode refresh cap, expo-file-system class API names in SDK 58, `transformOrigin` in RN 0.88 styles,
`SkRSXform.set` argument order, expo-router focus hook, ESLint 10 support of `eslint-plugin-no-only-tests`, Expo Atlas
invocation, Reanimated/worklets Jest setup for 0.13, Reassure defaults, Maestro version, the family's oldest device,
the thermal-state module, `performance.rnStartupTiming`.
