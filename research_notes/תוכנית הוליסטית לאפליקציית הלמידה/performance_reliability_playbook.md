# Performance & Reliability Playbook — Expo SDK 58 / RN 0.88 kids' game (iPad + iPhone, Hebrew RTL)

> Research date: 2026-11-02. Method: web search, extended mode (summarised results). **Limitation:** WebFetch could not reach `expo.dev`, `docs.expo.dev` or `docs.swmansion.com` from this sandbox. They are outside the session's allowed network domains (DNS ENOTFOUND). Facts from those sites come from search-result summaries, not full-page reads. A writer who needs exact flag names or defaults should check them against the installed package versions. Items marked **[prior knowledge]** are well-established practice from my training data that I did not re-verify in this session. They sit under *Inferences*, never under *Cited Findings*.

---

## 1. RN New Architecture / Hermes V1 / startup / re-render control

### Takeaway
Use SDK 58 (RN 0.88). Make sure the Hermes V1 build is past the SDK 56/57 worklets memory regression. Expect `InteractionManager` to be gone in 0.88. Start-up comes from Hermes bytecode, a lean root, inline requires and lazy routes. Most of the re-render control comes from the React Compiler plus narrow Zustand selectors.

### Cited Findings
- SDK 58 ships React Native 0.88 and targets iOS 27. It jumps from 0.86, so it includes everything from 0.87. — [Expo SDK 58 changelog](https://expo.dev/changelog/sdk-58)
- The SDK 58 release date is around Oct 15 2026. An upgrade issue from today reports the latest is 58.0.22. — [GitHub discussion](https://github.com/software-mansion/react-native-enriched-html/discussions/823); [payload-expo-tauri issue #91](https://github.com/alex-soldatkin/payload-expo-tauri/issues/91)
- **Breaking in RN 0.88 / SDK 58:** `InteractionManager` removed, the `Touchable` export removed, `NativeMethods` types removed, the Modal `animated` prop removed, and several StatusBar props removed. Expo Router native tabs moved from `expo-router/unstable-native-tabs` to `expo-router/native-tabs`. The upgrade command is `npx expo install expo@^58.0.0 --fix`. — [Expo blog: How to upgrade to SDK 58](https://expo.dev/blog/how-to-upgrade-to-sdk-58)
- On web, Expo Router's safe-area insets now come from CSS rather than `react-native-safe-area-context`. This matters for Playwright layout tests. — [Medium summary of SDK 58](https://medium.com/@onix_react/whats-new-in-expo-sdk-58-803fd0e57e34) (secondary source)
- The default fingerprint preset is now `balanced`. — [Expo SDK 58 changelog](https://expo.dev/changelog/sdk-58)
- **Hermes V1 memory regression:** The SDK 56 changelog lists it as a known regression. Apps that import `react-native-worklets` or `react-native-reanimated` can see large memory increases, introduced with RN 0.85. Expo recommends SDK 57 for Hermes v1 apps on SDK 55/56. — [Expo SDK 56 changelog](https://expo.dev/changelog/sdk-56)
- In issue #46519 the minimal repro's memory went from 149 MB to 266 MB. Even importing worklets triggers it. Disabling Hermes v1 via expo-build-properties reportedly did not help. — [expo/expo #46519](https://github.com/expo/expo/issues/46519)
- The initial SDK 57 release inherited the regression and later patch versions fixed it. — [Expo SDK 57 changelog](https://expo.dev/changelog/sdk-57)
- Community PRs report the fix in `expo@^57.0.9` (RN 0.86.3) with Hermes build `250829098.0.17`. Builds `≤250829098.0.15` are affected, and 57.0.8 / RN 0.86.0 still showed the issue. — [gldc/hermes-mobile-app #37](https://github.com/gldc/hermes-mobile-app/pull/37); [Wadboy70/meditatio-v2 #6](https://github.com/Wadboy70/meditatio-v2/pull/6); [Dayova PR #545](https://github.com/Dayova/dayova-mvp/pull/545); [dacown87/rezepti #58](https://github.com/dacown87/rezepti/pull/58) (community sources; the exact versions are not in an official release note)
- A separate RN issue reports 25–30% more RAM on 0.85.x vs 0.83.x with the new Hermes (Android, same app). — [react-native #57059](https://github.com/react/react-native/issues/57059)
- In SDK 55, Hermes v1 was opt-in and needed a source build. — [Expo SDK 55 changelog](https://expo.dev/changelog/sdk-55)
- Expo enabled the React Compiler in the default template from SDK 54, when it was at RC stage. SDK 54 also turned on experimental import support for better tree shaking. — [Expo SDK 54 changelog](https://expo.dev/changelog/sdk-54); [SDK 54 beta](https://expo.dev/changelog/sdk-54-beta)
- The New Architecture is the only option from SDK 55 (RN 0.83). — [Expo SDK 54 changelog](https://expo.dev/changelog/sdk-54)
- Hermes precompiles JS to bytecode at build time (AOT). Callstack credits Hermes with up to 40% better startup and much lower memory, for about 2 MB of binary size. — [Callstack: Why you don't have to minify JS in RN](https://www.callstack.com/blog/why-you-dont-have-to-minify-javascript-code-in-react-native-apps); [Callstack: Optimize startup with Hermes](https://www.callstack.com/blog/optimize-android-app-startup-time-with-hermes)
- Inline requires are a Metro option that defers module evaluation until first use. Some frameworks disable this, so some modules may need exclusion. — [React Native docs: Optimizing JavaScript loading](https://reactnative.dev/docs/optimizing-javascript-loading)
- FlashList v2 is a New-Architecture-only, JS-only ground-up rewrite. It needs no size estimates, reduces blank area by up to 50% vs v1, and keeps the visible position by default. — [Shopify Engineering: FlashList v2](https://shopify.engineering/flashlist-v2); [v2 migration guide](https://shopify.github.io/flash-list/docs/v2-migration/)
- Memoize FlashList row components, or let the React Compiler do it. — [reactnativerelay FlashList v2 guide](https://reactnativerelay.com/article/react-native-flashlist-v2-expo-high-performance-lists-migration) (secondary)

### Inferences
- **Rule: pin the Hermes build.** In CI, assert that the resolved `hermes-compiler`/Hermes V1 version is ≥ the 250829098.0.17-line fix. Also run a memory smoke test (idle on home screen after 60 s, memory under budget) on every SDK bump. The regression went in silently with an RN bump.
- **Rule: replace `InteractionManager.runAfterInteractions`.** Use `requestIdleCallback`, `setTimeout(…,0)` after `onLayout`/first frame, or React `startTransition`. Run `grep -r InteractionManager` in CI, because third-party libs may still call it. **[prior knowledge]** `requestIdleCallback` exists in RN's New Architecture event loop.
- **[prior knowledge] React Compiler:** In SDK 54+ templates it is enabled via `babel-preset-expo` (`experiments.reactCompiler: true` in app.json). Use the ESLint React Compiler rules (in `eslint-plugin-react-hooks` v6+) to find components it bails out on. The compiler does not protect against: inline object props passed into Reanimated `useAnimatedStyle` deps, context fan-out, or store subscriptions that are too broad.
- **[prior knowledge] Zustand:** Subscribe with atomic selectors (`useStore(s => s.score)`) or `useShallow` for tuples. Never put per-frame game state (positions, timers) in React state or Zustand. It belongs in shared values (UI thread) or in plain refs inside the game loop. React state is only for things that change the *UI tree*: screen, level-complete modal, star count at the end.
- **[prior knowledge] Startup:** Keep `app/_layout.tsx` minimal. Load fonts via the `expo-font` config plugin (embedded at build time, so nothing loads at runtime). Lazy-load heavy modules (three/R3F, Skia scenes, Rive files) only in the routes that use them. Expo Router does lazy route evaluation in production by default with `--async-routes`/bundle splitting on web; on native, inline requires do the equivalent. Hold the splash with `SplashScreen.preventAutoHideAsync()` only until fonts plus the home-screen essentials are ready, never until the whole app has loaded.
- **Rule:** no `console.log` in hot paths in release builds (use `babel-plugin-transform-remove-console` or Metro `drop_console`), because logging crosses into native and is slow. **[prior knowledge]**

### Gaps
- I could not fetch the SDK 58 changelog's Hermes/React Compiler sections. I could not confirm whether Hermes V1 is the default in SDK 58 (likely, since SDK 56 lists V1 regressions as general), or whether the React Compiler has reached 1.0 and is on by default in RN 0.88 templates.
- I found no official TTI target for RN apps. The "<2 s splash→home" target is a product goal, not a published benchmark.

---

## 2. Reanimated 4 + worklets + Gesture Handler (UI-thread motion, 120 Hz)

### Takeaway
Run every per-frame motion and gesture-driven value on the UI thread with shared values. Animate only `transform`/`opacity` where you can. Turn on the Reanimated static flags for synchronous UI-prop updates and commit-hook optimisation. Make sure `CADisableMinimumFrameDurationOnPhone` is set so iPhone ProMotion can reach 120 Hz.

### Cited Findings
- **Many animated components scrolling causes FPS drops.** The fix is RN ≥0.80, Reanimated ≥4.2.0 and the `USE_COMMIT_HOOK_ONLY_FOR_REACT_COMMITS` static flag. Also consider `enableCppPropsIteratorSetter`. — [Reanimated docs: Performance](https://docs.swmansion.com/react-native-reanimated/docs/guides/performance/) (via search summary)
- **Many simultaneous animations:** enable `ANDROID_SYNCHRONOUSLY_UPDATE_UI_PROPS` (from 4.0.0) and `IOS_SYNCHRONOUSLY_UPDATE_UI_PROPS` (from 4.2.0). They give a fast path for non-layout styles such as opacity and transform. — [Reanimated docs: Performance](https://docs.swmansion.com/react-native-reanimated/docs/guides/performance/); [Feature flags](https://docs.swmansion.com/react-native-reanimated/docs/guides/feature-flags/)
- **120 fps on iOS** needs `CADisableMinimumFrameDurationOnPhone` in Info.plist. It is on by default in the RN template from 0.82. — [Reanimated docs: Performance](https://docs.swmansion.com/react-native-reanimated/docs/guides/performance/)
- **Flag conflicts (unreleased docs):** one scrolling-related flag is only safe with RN's `preventShadowTreeCommitExhaustion` (RN ≥0.81). `ENABLE_SHARED_ELEMENT_TRANSITIONS` and `USE_ANIMATION_BACKEND` each conflict with that flag. — [Reanimated feature flags (unreleased)](https://docs.swmansion.com/react-native-reanimated/docs/unreleased/guides/feature-flags/)
- Reanimated 4.2.0 introduced `ENABLE_SHARED_ELEMENT_TRANSITIONS`, `FORCE_REACT_RENDER_FOR_SETTLED_ANIMATIONS` and `USE_COMMIT_HOOK_ONLY_FOR_REACT_COMMITS`. — [SWM: Introducing Reanimated 4.2.0](https://swmansion.com/blog/introducing-reanimated-4-2-0-71eea21ca861/)
- Reanimated 4.4.0 added an `IOS_CSS_CORE_ANIMATION` flag and removed `EXPERIMENTAL_CSS_ANIMATIONS_FOR_SVG_COMPONENTS`. — [newreleases: Reanimated 4.4.0](https://newreleases.io/project/github/software-mansion/react-native-reanimated/release/4.4.0)
- SDK 57 updated the animation and gesture libraries; the main performance note was the Hermes fix. — [Expo SDK 57 changelog](https://expo.dev/changelog/sdk-57)

### Inferences
- **Rules [prior knowledge]:**
  1. Never read `sv.value` on the JS thread during gameplay; each read is a synchronous cross-runtime read. Use `useAnimatedReaction` to react on the UI thread, and call back to JS (`scheduleOnRN` in worklets 0.5+, formerly `runOnJS`) **only on discrete events** (hit, level end), never per frame.
  2. Animate `transform` and `opacity`, not `width`/`height`/`top`/`left`/`margin`. Layout props force Yoga layout and shadow-tree commits, and they skip the synchronous fast path above.
  3. Use the Reanimated 4 **CSS animations/transitions API** for declarative, fire-and-forget UI flourishes (button pulse, star pop). They run without per-frame JS. Use `useFrameCallback` for game loops that need delta time.
  4. **Layout animations (`entering`/`exiting`/`layout`)**: avoid them on large lists or on items that mount and unmount together, and never put them on the same view as a heavy Skia, GL or Rive surface. Exiting animations keep views alive and have caused flicker and memory issues with lists and navigation. Prefer manual opacity/translate on a fixed-size container.
  5. Gesture Handler: define gestures with `Gesture.*()` builders and keep callbacks as worklets (the default when Reanimated is installed). Avoid `.runOnJS(true)` on Pan/Pinch. Wrap the root in `GestureHandlerRootView`. Use `Gesture.Tap().maxDuration()` and `hitSlop` for small fingers: kids mis-tap, so hit areas should be at least 64 pt on iPad.
  6. Deltas on the UI thread should be frame-rate independent, using the `timeSincePreviousFrame` given by `useFrameCallback`. A game tuned at 60 Hz must not run double speed at 120 Hz.
  7. **Low Power Mode on iOS caps ProMotion displays at 60 Hz [prior knowledge].** The 8.3 ms budget is the best case; design every animation to look right at 60 Hz.
  8. Release builds only for perf judgments. Dev mode, Remote JS debugging and Fast Refresh distort worklet timing.
- **Starting flag set to evaluate (A/B on device):** `IOS_SYNCHRONOUSLY_UPDATE_UI_PROPS=true`, `USE_COMMIT_HOOK_ONLY_FOR_REACT_COMMITS=true`, plus RN `preventShadowTreeCommitExhaustion` when you use the flag that needs it. Leave shared element transitions off unless you need them, because of the conflicts.

### Gaps
- I could not read the full Reanimated 4.x performance page or check which flags became defaults in Reanimated 4.4/4.5, the versions likely bundled with SDK 58.
- I found no published per-device count of how many concurrently animated views Reanimated 4 can drive at 120 Hz.

---

## 3. @shopify/react-native-skia (2D game layer)

### Takeaway
Use one Skia `Canvas` per scene, sprites packed into atlas textures drawn with `Atlas` and driven by worklet-updated RSXform buffers, cached static layers (offscreen surface or Picture), and memoised paths and paints. Treat Picture vs Atlas speed as something to measure on the target iPad.

### Cited Findings
- `useTexture` creates a texture directly on the UI thread without copies. `useRectBuffer`/`useRSXformBuffer` animate sprites and transforms efficiently. Atlas transforms can be animated from worklets at near-zero cost. — [Skia docs: Atlas](https://shopify.github.io/react-native-skia/docs/shapes/atlas)
- Atlas batching is per texture. In one benchmark, 1,000 sprites on one atlas took about 26.6 µs to queue; spreading them over 8 atlases cost about 35% more. — [grzegorzotto.dev: Skia Atlas 2,000 sprites](https://grzegorzotto.dev/blog/skia-atlas-2000-sprites-react-native) (single-author benchmark)
- Conflicting evidence: one report says Picture beat Atlas at 1,000 circles on a Galaxy A34. Another found the Picture/offscreen approach fixed memory but used 100% CPU. — [samuelscheit.com: Skia list renderer](https://samuelscheit.com/blog/2024/react-native-skia-list); [grzegorzotto.dev](https://grzegorzotto.dev/blog/skia-atlas-2000-sprites-react-native)
- Layers create offscreen buffers and should be used sparingly. Render complex static content once offscreen and reuse it (`makeOffscreenSurface`/`drawOffscreen` in newer docs; older docs use `Skia.Surface.MakeOffscreen`). — [Skia performance guide (mintlify mirror)](https://www.mintlify.com/shopify/react-native-skia/advanced/performance); [Offscreen canvas](https://mintlify.wiki/shopify/react-native-skia/canvas/offscreen)
- Creating a new offscreen canvas/image every frame is a bottleneck. — [samuelscheit.com](https://samuelscheit.com/blog/2024/react-native-skia-list)
- Define `Skia.Path` objects outside components or memoise them, and keep object references stable. — [Skia performance guide (mintlify mirror)](https://www.mintlify.com/shopify/react-native-skia/advanced/performance)

### Inferences
- **Rules [prior knowledge + above]:**
  - Pass Reanimated shared values/derived values straight into Skia props. Skia reads them on the UI thread with no React re-render. Never put `useState` per frame into Skia components.
  - Create paints, shaders (`Skia.RuntimeEffect.Make`), fonts (`useFont`/`matchFont`) and images once (module scope or `useMemo`). Compile runtime shaders at scene load, not on first use mid-animation, to avoid first-frame stalls.
  - Pack sprites into ≤2048×2048 sheets, one sheet per scene where possible. A 2048² RGBA texture is 16 MB of GPU memory (2048×2048×4 bytes), about 21 MB with mipmaps.
  - **Crispness:** Skia Canvas renders at device pixel density automatically. Blur comes from scaling up low-res bitmaps. Ship sprites at @2x for iPad/iPhone and @3x only where iPhone close-ups need it, prefer vectors (paths/SVG) for UI shapes, and avoid `transform: scale` > 1 on raster content.
  - Use only one or two Canvases mounted at once. Each Canvas is a GPU surface; unmount offscreen scenes.
- Skia on web runs CanvasKit WASM (several MB), so web perf numbers for Skia scenes do not predict iOS.

### Gaps
- I could not check the current (2026) Skia API names for offscreen rendering or the official Atlas sprite limits.
- I found no iPad-specific Skia benchmarks.

---

## 4. Rive (@rive-app/react-native, Nitro runtime)

### Takeaway
Use the Nitro runtime. Load each `.riv` once and share it across views, dispose explicitly, cap `frameRate` on looping/idle animations, and keep few active instances on screen.

### Cited Findings
- The Margelo Nitro rewrite reports up to 94× faster multi-view loads, 4.7× lower memory and 3.2× faster property writes vs the legacy SDK (vendor benchmark on an iPhone 13 mini). — [Margelo: Rewriting Rive RN with Nitro](https://margelo.com/blog/rewriting-rive-react-native-with-nitro-modules)
- Legacy loads a separate copy of the file per view; Nitro shares one copy. Four graphics from one file took about 58 ms (Nitro with preload and sharing) vs about 500 ms (legacy). Load and dispose are explicit. — [Margelo blog](https://margelo.com/blog/rewriting-rive-react-native-with-nitro-modules)
- `RiveView` renders at the display refresh rate by default. The `frameRate` prop caps drawing (not animation speed) to save CPU and battery. — [Rive docs: React Native](https://rive.app/docs/runtimes/react-native)
- Requires Nitro Modules ≥0.35.10 (≥0.36.2 on RN 0.87+). The legacy runtime is still supported but migration is recommended. — [Rive docs: React Native](https://rive.app/docs/runtimes/react-native)
- In an older Android comparison, Rive used about half Lottie's RAM (about 12 vs 23 MB Java, about 25 vs 49 MB native). — [Callstack: Lottie vs Rive](https://www.callstack.com/blog/lottie-vs-rive-optimizing-mobile-app-animation)

### Inferences
- **Rules [prior knowledge]:** Preload `.riv` files for the next screen during the current screen's idle time. Use one Rive file per character (mascot) with multiple artboards and state machines rather than many files. Drive state-machine inputs and data binding on events, not per frame from JS. Set `frameRate={30}` for ambient idle loops and full rate only for the focal interactive character. Pause or unmount Rive views that are off screen or behind modals. Keep `.riv` files small by using vector over embedded raster, and if you must embed images, use referenced/out-of-band assets loaded once. Target ≤3 simultaneously animating Rive instances on an older iPad until device profiling says otherwise.
- Rive draws through its own renderer (Metal on iOS). Stacking Rive, Skia and GL surfaces in one screen multiplies GPU surfaces and compositing cost. Avoid placing all three live in the same frame.

### Gaps
- I found no official Rive guidance on state-machine evaluation cost, a max instance count, or a file-size budget. Instance and size budgets here are judgment calls to check with Instruments.

---

## 5. three.js via @react-three/fiber@9 native + expo-gl

### Takeaway
Treat 3D as the most expensive, least-proven layer. Budget about 100 draw calls on mobile, instance repeated meshes, render on demand (`frameloop="demand"`) when the scene is static, dispose everything, and render at a capped pixel ratio. KTX2/ASTC compressed textures are not reliably working in expo-gl, so plan for PNG/JPEG/WebP-decoded textures with tight resolution budgets.

### Cited Findings
- R3F guidance: keep meshes to a few hundred, never more than about 1000, and instance repeated objects. — [R3F docs: Scaling performance](https://r3f.docs.pmnd.rs/advanced/scaling-performance)
- Mobile rule of thumb: about 100 draw calls per frame. — [utsubo: 100 three.js tips (2026)](https://www.utsubo.com/blog/threejs-best-practices-100-tips) (community)
- `frameloop="demand"` stops continuous rendering; call `invalidate()` after changes. — [R3F docs: Scaling performance](https://docs.pmnd.rs/react-three-fiber/advanced/scaling-performance)
- `useFrame` subscriptions are removed automatically on unmount. Manually created geometries, materials and textures stay in GPU memory until `.dispose()`. `useLoader` caches, so swapped textures need `useLoader.clear()`. — [R3F hooks docs](https://r3f.docs.pmnd.rs/api/hooks); [utsubo tips](https://www.utsubo.com/blog/threejs-best-practices-100-tips)
- KTX2 textures in an Expo + three.js GLB fail to render while PNGs work. Supporting `THREE.CompressedTexture` (ASTC/ETC2) is the likely requirement. No fix was confirmed. — [three.js forum: KTX2 not loading in Expo](https://discourse.threejs.org/t/ktx2-texture-not-loading-in-expo-three-js-react-native/87148)
- The expo-three texture loader is deprecated (load with three's own modules). three is a peer dependency from expo-three 5. — [expo-three npm](https://npmjs.com/package/expo-three)
- A context created with `createContextAsync` must be destroyed with `destroyContextAsync`. — [Expo GLView docs](https://docs.expo.dev/versions/latest/sdk/gl-view/)
- There is a historical expo-gl crash that occurred only in iOS Release builds (RN 0.66). This is a reminder to test release builds. — [expo/expo #16057](https://github.com/expo/expo/issues/16057)

### Inferences
- **Rules [prior knowledge]:**
  - **Frame loop:** use `frameloop="demand"` for menus and 3D dioramas. Switch to `"always"` only during active 3D gameplay and back afterwards. Pause the loop on `AppState` background and when the screen loses focus (`useIsFocused`).
  - **GC churn:** never allocate `new Vector3()`/`Matrix4`/`Color` inside `useFrame`. Preallocate at module scope and reuse; pool projectiles and particles. Hermes GC pauses show up as periodic hitches.
  - **Pixel ratio:** set `dpr={[1, 2]}`, capped at 2. iPhone 3× rendering costs about 2.25× the fill rate of 2× for a small visual gain. Combine with antialias off plus FXAA, or MSAA only if fill rate allows.
  - **Draw calls:** merge static geometry, use `InstancedMesh`, share materials, keep shadow maps off or baked, and use ≤1 dynamic light plus baked lighting or matcaps.
  - **Textures:** power-of-two, ≤1024² for props and ≤2048² for hero assets. GPU memory per 2048² RGBA texture is about 16–21 MB, so ten hero textures already use about 200 MB. Downscale at build time.
  - **GLB:** use meshopt compression (decoded in JS/WASM; check Hermes compatibility), or Draco only if a JS decoder works under Hermes without WASM. Validate in the dev build early.
  - **Unmount:** traverse the scene and dispose geometry, material and texture; call `renderer.dispose()`; destroy the GL context. Test with 20 mount/unmount cycles while watching Instruments Allocations and the GPU memory in the Xcode memory gauge.
  - **Context loss:** iOS can tear down GL contexts when the app is backgrounded under memory pressure. Listen for context loss where available, and design scenes to be re-creatable from state (rebuild from a level descriptor, not from mutable GPU objects).
- Apple deprecated OpenGL ES in 2018; expo-gl still sits on it on iOS. This is a long-term risk. The Margelo "Filament"/"react-native-wgpu" (WebGPU) paths are alternatives worth tracking. **[prior knowledge; not verified this session]**

### Gaps
- I found no source confirming KTX2/ASTC support or `EXT_texture_compression_astc`/`WEBGL_compressed_texture_astc` exposure in expo-gl on iOS in 2026. I also found no R3F-native-specific benchmarks or expo-gl context-loss docs.

---

## 6. Audio (expo-audio) and haptics

### Takeaway
Configure the audio session once at startup, before the first `play()`. Preload every SFX player before a level starts (one or a small pool per sound) and reuse players with `seekTo(0)` + `play()`. Release players on level exit. Handle interruptions and the silent switch deliberately.

### Cited Findings
- `interruptionMode` defaults to `'mixWithOthers'`. Allowed values are `'mixWithOthers' | 'doNotMix' | 'doNotMixPersistent' | 'duckOthers'`. Lock-screen controls need `doNotMix`/`doNotMixPersistent`. — [Expo docs: expo-audio](https://docs.expo.dev/versions/latest/sdk/audio/)
- One report (expo-audio 57.0.5) says `mixWithOthers` reaches the iOS session only once `setAudioModeAsync` is called; on Android `interruptionMode` is null until then and `play()` requests transient focus. The proposed fix is to call `setAudioModeAsync` once at startup before the first play. In that game, music killed the player's podcast. — [rush_hour_ios_android #189](https://github.com/ekomlenovic/rush_hour_ios_android/issues/189) (single-developer diagnosis)
- `playsInSilentMode: false` combined with `interruptionMode: 'mixWithOthers'` threw "Impossible audio mode" on iOS in an older issue. — [expo/expo #34902](https://github.com/expo/expo/issues/34902)
- The default iOS category is ambient, so the silent switch mutes it; `playsInSilentMode` overrides that. — [shipnative: Expo Audio](https://www.shipnative.dev/blog/expo-audio) (secondary)
- Ambient audio stopped about 3 minutes after screen lock without the right background configuration. — [rorklab: expo-audio screen-lock stop](https://rorklab.net/en/articles/rork-dev/expo-audio-screen-lock-stop-three-layer-diagnosis) (secondary)
- General SFX practice in game engines: preload one pool per sound, reuse players, and expect platform caps on concurrent low-latency effects. — [Flame AudioPool](https://docs.flame-engine.org/1.3.0/flame_audio/audio_pool.html); [Qt QSoundEffect](https://doc.qt.io/qtforpython-6.10/PySide6/QtMultimedia/QSoundEffect.html)
- Old Expo forum reports show that loading about 14 small sounds at once on Android was flaky, and rapid replays lagged (older expo-av APIs). — [Expo forums: laggy audio](https://forums.expo.dev/t/laggy-audio-on-multiple-plays/3169)

### Inferences
- **Audio rules [prior knowledge + above]:**
  - Startup: `setAudioModeAsync({ playsInSilentMode: true, interruptionMode: 'mixWithOthers' /* or 'duckOthers' for voice prompts */, shouldPlayInBackground: false })`. A kids' learning app should play voice prompts even with the silent switch on (parents often have it on), but this is a product decision, so expose a mute toggle.
  - Create players with `createAudioPlayer(source)` (imperative, not tied to a component) in a `SoundBank` module during level loading. Keep 2–3 players for sounds that overlap (taps, correct chime) and round-robin through them. `useAudioPlayer` is fine for screen-scoped narration.
  - Play SFX via `player.seekTo(0); player.play()`. Do not create players at tap time, because that adds decode and allocation latency on the JS thread.
  - Release (`player.release()`/`remove()`) all level players on level exit to cap memory.
  - Formats: short SFX as AAC/M4A or CAF/WAV (PCM decodes fastest, so use it for the tiniest, most latency-critical SFX); narration as AAC 64–96 kbps mono.
  - Trigger SFX from the JS thread on the event (via `scheduleOnRN` from the gesture worklet). One JS hop costs about 1 frame. Acceptable perceived audio latency for taps is roughly ≤50–80 ms. Measure on the device by filming at 240 fps.
  - Interruptions (calls, Siri, alarms): pause the game loop on interruption and on `AppState` `inactive`. On resume, do not auto-resume voice mid-sentence; restart the prompt.
- **Haptics [prior knowledge]:** `expo-haptics` calls are async native calls. Fire them at the same moment as the visual feedback, from the same event handler. Throttle to ≤1 per ~100 ms, because rapid repeated haptics feel mushy and cost battery. Use `selectionAsync` and `impactAsync(Light)` for kids, and check that haptics no-op cleanly on iPads, most of which have no Taptic Engine.

### Gaps
- I could not read the current expo-audio API page in full. I found no measured expo-audio latency numbers and no documented max concurrent players on iOS.

---

## 7. Layout stability (no layout shift), fonts, RTL, nikud, images

### Takeaway
Never render a frame you will re-lay-out. Embed fonts at build time and hold the splash until fonts and the first screen's data are ready. Give every image and media surface fixed dimensions. Set RTL at the native level before JS starts, and size Hebrew text with nikud using generous line heights tested on iOS.

### Cited Findings
- `lineHeight` in RN is absolute pixels; glyph parts beyond it clip. One blog suggests ≥1.4× font size. — [Medium: Your styles are lying to you](https://medium.com/@tharunbalaji110/your-styles-are-lying-to-you-and-react-native-is-not-sorry-6663749c4174) (blog)
- iOS `TextInput` with `lineHeight` clips content (iOS only). — [RN #28012](https://github.com/react/react-native/issues/28012); [RN #41240](https://github.com/facebook/react-native/issues/41240)
- An RN PR proposed recentering glyphs when an explicit lineHeight is smaller than the font's glyph bounds (behind a flag); it is marked closed. — [RN PR #57408](https://github.com/react/react-native/pull/57408)
- An Android 15 text-clipping fix uses `useBoundsForWidth`. — [djazairdev discussion](https://github.com/orgs/djazairdev/discussions/4)
- The Expo Router SDK 58 web safe-area now uses CSS insets. — [Medium SDK 58](https://medium.com/@onix_react/whats-new-in-expo-sdk-58-803fd0e57e34)

### Inferences
- **Rules [prior knowledge]:**
  - **Fonts:** use the `expo-font` config plugin to embed Hebrew fonts (for example Heebo, Assistant, Rubik, Noto Sans Hebrew, or a nikud-capable font such as "Noto Serif Hebrew"/"Frank Ruhl Libre"). Check that the font actually has nikud glyph positioning (GPOS mark anchors). Fonts without it render misplaced dots. No runtime `useFonts` means no fallback-to-system flash and no reflow.
  - **Splash:** `SplashScreen.preventAutoHideAsync()` at module scope; hide in the root layout's `onLayout` once fonts plus the critical home assets (mascot `.riv`, home background) are ready. Use a timeout fallback (for example 4 s) so an asset failure never hangs the splash.
  - **Nikud:** set `lineHeight` ≈ 1.5–1.6× fontSize for nikud text (marks sit below the baseline: kamatz, patach, segol, hirik; dagesh is inside the letter; holam is above). Add `paddingVertical` ≈ 0.1× fontSize on the text container. Test strings that combine high marks (holam, shin/sin dot) with low marks (kubutz, segol) at every Dynamic Type size. Add Playwright and iOS snapshot tests with a nikud "torture string".
  - **Dynamic Type:** for game UI, cap `maxFontSizeMultiplier` (for example 1.3) on in-game labels so layouts do not overflow fixed game boards. Let reading screens scale fully. A child's iPad may have accessibility text sizes set by parents.
  - **RTL:** with `expo-localization` set `supportsRTL`/`forcesRTL` in app config (native, applied before JS) rather than `I18nManager.forceRTL()` at runtime, which needs a reload and causes a mirrored-then-flipped first frame. Use `start`/`end` (not `left`/`right`) in styles. Do **not** mirror game-world coordinates: physics, Skia and GL canvases use absolute x. Mirror only UI chrome, and decide per scene whether the progress direction (path, number line) goes right-to-left. Icons with direction (arrows, play) need explicit flipping; logos and clocks do not. Numbers and math expressions stay LTR inside RTL text (wrap in a Unicode LRI/PDI or a separate `Text` with `writingDirection: 'ltr'`).
  - **Images:** `expo-image` with explicit `width`/`height` or `aspectRatio`; `placeholder` (blurhash/thumbhash) for remote images; `cachePolicy="memory-disk"`; `Image.prefetch()` for the next level; `transition={0}` for game sprites, because a fade looks like lag. Bundle level-1 assets locally.
  - **Containers:** fixed-size game boards computed once from `useWindowDimensions` + safe-area insets. Use `onLayout` only to *read*, never to trigger a second render pass that moves content. Skeletons must match the final layout exactly. Lock orientation per app (landscape for iPad games) in app.json so no rotate-reflow happens mid-game; if both orientations are allowed, re-layout behind a short fade.
  - **iPad multitasking (Split View / Stage Manager):** window size can change at runtime. Either opt out (`UIRequiresFullScreen`, which iPadOS increasingly ignores) or handle resizes by scaling a fixed logical game resolution (letterboxing), not by reflowing the board.

### Gaps
- I found no source specifically on Hebrew nikud clipping in React Native, and no RN-specific guidance on GPOS support in iOS CoreText for Hebrew webfonts. The line-height figures above are inference and must be validated with screenshots.

---

## 8. Reliability: crashes, hangs, memory, lifecycle, storage, offline

### Takeaway
On 3 GB iPads the memory ceiling (jetsam) is the dominant crash risk. Budget memory per scene, release on unmount and on memory warnings, keep heavy SQLite work async with WAL, wrap each scene in an error boundary that can rebuild it, and pause everything on background or interruption.

### Cited Findings
- `os_proc_available_memory()` returns the bytes left before the app's limit (0 if already over). Limits can change during the app's life and don't map directly to physical RAM. — [Apple docs: os_proc_available_memory](https://developer.apple.com/tutorials/data/documentation/os/os_proc_available_memory.md)
- Under memory pressure iOS force-quits apps that don't free memory fast enough, and logs a jetsam event. Jetsam decisions use resident size. — [antongubarenko: Memory efficiency in iOS](https://antongubarenko.substack.com/p/memory-efficiency-in-ios-reducing); [zenn: memory entitlements](https://zenn.dev/mtfum/articles/ios_memory_entitlements?locale=en)
- iPadOS per-app limits: about 5 GB on 8/16 GB iPad Pro; about 3 GB (about half of RAM) on a 6 GB iPad Pro (2021 report). — [9to5Mac](https://9to5mac.com/2021/05/28/ipad-pro-ram-limits/)
- Reported available memory: iPhone 13 (iOS 15) about 2.2–2.3 GB; 4 GB iPad Pro 11 about 2.8 GB. — [Apple dev forums #702400](https://developer.apple.com/forums/thread/702400)
- The `increased-memory-limit` entitlement raises the per-app limit only on some models and makes background termination more likely. — [zenn](https://zenn.dev/mtfum/articles/ios_memory_entitlements?locale=en); [Apple forums #770868](https://developer.apple.com/forums/thread/770868)
- expo-sqlite: enable `PRAGMA journal_mode = WAL` on new databases. The docs warn that the sync APIs can block the JS thread. `SQLiteProvider` with `useSuspense` works inside a `React.Suspense` boundary placed above it. — [Expo docs: SQLite](https://docs.expo.dev/versions/latest/sdk/sqlite/); [expo docs source](https://github.com/expo/expo/blob/main/docs/pages/versions/unversioned/sdk/sqlite.mdx)
- An issue argues that the async SQLite methods can still run on a thread that hurts performance (Android `Dispatcher.IO` request; unresolved in the results). — [expo/expo #34531](https://github.com/expo/expo/issues/34531)
- A Suspense placement different from the docs caused an infinite loop. — [zenn article](https://zenn.dev/ud/articles/ac0436a55a5835)
- Bulk inserts of 1000+ rows should use async to avoid dropped frames (blog opinion). — [zomeru.dev](https://zomeru.dev/blogs/optimizing-react-native-expo-new-architecture-jsi-sqlite)

### Inferences
- **Memory (no direct source for 3 GB iPads):** Extrapolating from the data above (about half of RAM on 6 GB devices; about 2.8 GB available on 4 GB), a 3 GB iPad (iPad 8th/9th gen, iPad Air 3, iPad mini 5) is likely to jetsam somewhere around 1.2–1.5 GB footprint. **Budget: ≤400 MB steady state, ≤650 MB peak per scene**, leaving headroom for iOS, background audio and the Hermes V1 regression class.
- **Memory warnings [prior knowledge]:** RN `AppState.addEventListener('memoryWarning')` exists on iOS. On a warning: clear `expo-image` memory cache (`Image.clearMemoryCache()`), `useLoader.clear()`, release non-current-level audio players, and drop prefetched next-level assets. Optionally add a tiny Expo module exposing `os_proc_available_memory()` to pick asset quality tiers at launch.
- **Error boundaries:** use one at the root (friendly "let's try again" screen with a mascot, no stack trace) and one per game scene that remounts the scene from persisted state. Expo Router supports a per-route `ErrorBoundary` export. Native crashes (GL, Rive, Skia) are not caught by JS boundaries; they need crash reporting (Sentry `@sentry/react-native` with the Expo plugin, or native crash logs via TestFlight/Xcode Organizer).
- **Hang watchdog [prior knowledge]:** in dev/beta, schedule a JS-thread heartbeat with `setInterval(…, 500)` and record gaps > 250 ms (JS stall). A UI-thread heartbeat via `useFrameCallback` writes a shared value timestamp. Sentry's App Hang detection (iOS) and Xcode Organizer's Hang reports and MetricKit `MXHangDiagnostic` cover release builds. Never run synchronous JSI calls (sync SQLite, sync file reads, large `JSON.parse`) during gameplay.
- **Lifecycle:** on `AppState` → `inactive`/`background`: pause game loops (Reanimated `cancelAnimation` / pause `useFrameCallback`, R3F `frameloop="never"`, Rive pause), pause audio, flush progress to SQLite (it is async, so begin it on `inactive`, which iOS gives you first), and stop haptics. On `active`: rebuild GL context if lost, and resume to a "tap to continue" overlay rather than live gameplay.
- **Low Power Mode / thermal:** `expo-battery` exposes `isLowPowerModeEnabledAsync`/`useLowPowerMode` **[prior knowledge]**. When on, drop to 30 fps for ambient animations (Rive `frameRate`), disable particles and post-processing, and cap GL at dpr 1.5. Thermal state (`ProcessInfo.thermalState`) needs a small native module. On `serious`/`critical`, apply the same degradations.
- **Offline-first:** the app must be fully playable offline. Bundle levels 1–N in the binary, download later packs with `fetch` + `AbortController` timeouts (≤10 s), and retry with backoff. Never block the UI on network. All persistent progress goes in expo-sqlite (WAL, single connection, async API, writes batched in transactions at level end). Avoid AsyncStorage for anything large or frequently written: it is a single serialized key-value file, and large values slow startup reads.

### Gaps
- I found no official per-device jetsam limit for 3 GB iPads, no current documentation on the iOS `memoryWarning` AppState event under the New Architecture, and no resolution of the async SQLite threading issue on iOS.

---

## 9. Asset loading strategy

### Takeaway
Ship the first session's assets in the binary, preload per level behind a short, fixed-layout transition, prefetch the next level during idle time, and keep formats compact (WebP/AVIF for raster, `.riv` for vector characters, AAC for voice, compressed GLB). KTX2 is not yet a dependable option in expo-gl.

### Cited Findings
- KTX2 in expo-gl/three is unreliable (see §5). — [three.js forum](https://discourse.threejs.org/t/ktx2-texture-not-loading-in-expo-three-js-react-native/87148)
- Rive Nitro supports explicit preload and file sharing (about 58 ms vs about 500 ms for 4 views). — [Margelo](https://margelo.com/blog/rewriting-rive-react-native-with-nitro-modules)
- Skia `useTexture` and Atlas allow one GPU texture per sprite sheet. — [Skia Atlas docs](https://shopify.github.io/react-native-skia/docs/shapes/atlas)

### Inferences
- **Pipeline [prior knowledge]:**
  - Splash → home in under 2 s: home needs fonts (embedded), one mascot `.riv` (≤150 KB), a background WebP sized to the device (≤300 KB), and a SQLite open plus profile read (async, WAL).
  - Level loading: a `LevelLoader` resolves a manifest (images, atlases, `.riv`, audio, GLB). It downloads via `expo-asset` `Asset.loadAsync`, prefetches images (`expo-image` `Image.prefetch`), creates audio players and decodes Skia textures, all while a fixed-size "getting ready" animation plays. It enters the level only when everything has resolved, with a hard timeout and a degraded fallback.
  - Prefetch level N+1 during the end-of-level celebration, and evict level N−1.
  - Formats: WebP (wide support, expo-image native) for photos and backgrounds, PNG only for pixel-art or atlases needing exact alpha, AVIF where expo-image supports it (check iOS decode cost on older devices), SVG/Skia paths for UI. Sprite atlases built at build time with TexturePacker-like tooling (for example `free-tex-packer-cli`, which runs on Linux), ≤2048². Audio as AAC mono. 3D as GLB with meshopt and textures as WebP/PNG ≤1024² until KTX2 is proven.
  - Size budgets per level: ≤8 MB on disk and ≤150 MB decoded in memory (textures and audio buffers).
  - Use `expo-asset`'s config plugin to embed bundled assets at build time so the first launch does no extraction.

### Gaps
- I did not verify AVIF decoding support and performance in expo-image on iOS 16–27, or expo-asset embed behaviour in SDK 58.

---

## 10. Measuring: tooling from Linux vs on device

### Takeaway
From Linux the agent can check correctness, layout, RTL, bundle size, render counts (Reassure), web CLS and long tasks, and Android perf if an emulator is available. Real frame-pacing, memory, GPU, thermal and audio-latency judgments need release builds on the iPad/iPhone, profiled with Xcode Instruments on the parent's Mac.

### Cited Findings
- The React Native DevTools Performance panel has been stable since RN 0.83. It records JS execution, React Performance tracks, network and custom User Timings in one timeline. — [RN 0.83 blog](https://reactnative.dev/blog/2025/12/10/react-native-0.83); [RN DevTools docs](https://reactnative.dev/docs/react-native-devtools); [SWM: New Performance panel](https://swmansion.com/blog/react-native-debugging-new-performance-panel-in-react-native-0-83-21ca90871f6d/)
- Use DevTools for everyday profiling and native profilers when problems go beyond JS; compare JS vs UI thread for frame drops. — [reactnativerelay debugging guide 2026](https://reactnativerelay.com/article/complete-guide-debugging-react-native-apps-2026-devtools-performance-panel-radon-ide-production-monitoring) (secondary)
- Flashlight runs perf benchmarks (FPS, CPU, memory, JS thread) on **real Android devices via ADB** during scripted runs. — [rapidnative 2026 playbook](https://www.rapidnative.com/blogs/react-native-performance-optimization-2026-playbook) (secondary; Android-only)

### Inferences
- **Linux-side toolkit [prior knowledge]:**
  - **Reassure** (Callstack) runs in Jest with React Native Testing Library. It measures render count and duration of components (HUD, board, answer tiles) against a baseline branch. It runs headless on Linux CI, so it is the best automated guard against re-render regressions.
  - **Expo Atlas** (`EXPO_ATLAS=1 npx expo export`) / `source-map-explorer` for bundle composition. `npx expo export --platform ios` produces the Hermes bytecode bundle, so CI can track its size.
  - **Playwright on Expo web**: visual regression snapshots in RTL, nikud clipping checks, Layout Shift via `PerformanceObserver({type:'layout-shift'})` (CLS should be ≈0), Long Tasks (>50 ms), and a `requestAnimationFrame` frame-time histogram during scripted animations. Chrome tracing via CDP. Web results check logic and layout only. Reanimated, Skia (CanvasKit), Rive and GL take different code paths on web, so web FPS is **not** evidence of iOS smoothness.
  - **Android emulator on Linux** (if KVM is available): Maestro flows plus Flashlight give a rough JS-thread health score. Emulator GPU numbers are unreliable.
  - **Maestro**: YAML E2E flows run on the Android emulator on Linux and on the iOS simulator or devices on the Mac. One flow set covers both. Detox is heavier; Maestro suits an AI agent better.
  - **Static checks**: ESLint React Compiler rules, a custom lint banning `.value` reads outside worklets, banning `new THREE.Vector3` in `useFrame`, banning `InteractionManager`, and banning `I18nManager.forceRTL` at runtime. Also `npx expo-doctor` and `tsc --noEmit`.
- **Device-side toolkit (parent's Mac) [prior knowledge]:** Xcode Instruments on a **Release** (or `--configuration Release` dev-client-less) build:
  - *Animation Hitches* (hitch-time ratio; Apple's guidance is <5 ms/s "good", >10 ms/s "bad" — **[prior knowledge, verify]**)
  - *Time Profiler* (main thread vs `com.facebook.react.JavaScript`/Hermes thread vs UI-runtime worklet thread)
  - *Allocations* + *VM Tracker* (footprint per scene, growth across 20 mount/unmount cycles)
  - *Leaks*
  - *Metal System Trace* (GPU frame time for Skia/Rive/GL)
  - *Energy Log* / *Power Profiler*
  - *Core Animation FPS* (older template)
  - `xcrun xctrace record --template 'Animation Hitches' --device <udid> --attach <pid> --time-limit 60s --output run.trace` can be scripted over SSH or Remote Control.
  - The Xcode Organizer and MetricKit give field hang, launch-time and memory metrics from TestFlight users.
  - In-app dev overlay: an FPS meter (UI thread via `useFrameCallback` delta; JS thread via an rAF loop) and a JS stall counter, shown only in dev and beta builds behind a long-press gesture.
- **Sentry (optional):** `@sentry/react-native` gives app-start, slow/frozen frames, app-hang and crash reports. For a kids' app, check COPPA/Children's Privacy and App Store "Kids Category" rules on third-party SDKs before shipping it. A self-hosted or no-PII config, or MetricKit only, may be needed.

### Gaps
- No search result covered Reassure's 2026 version or Flashlight's iOS status (it was Android-only historically). I also could not verify Apple's current numeric hitch thresholds.

---

## 11. Synthesis: playbook rules, budgets, release checklist, CI checks, device protocol

### Takeaway
Six habits prevent most jank in RN games: per-frame work on the UI thread, a JS thread idle during play, stable layouts, preloaded and released assets, capped GPU cost (dpr, draw calls, textures), and memory budgets enforced per scene. Automate whatever Linux can check, and use a fixed, scripted device protocol on the Mac/iPad for the rest. *(This whole section is inference built on §§1–10.)*

### Cited Findings
- 120 Hz needs `CADisableMinimumFrameDurationOnPhone` — [Reanimated performance](https://docs.swmansion.com/react-native-reanimated/docs/guides/performance/)
- Mobile draw-call budget about 100 — [utsubo](https://www.utsubo.com/blog/threejs-best-practices-100-tips)
- Hermes V1 + worklets memory regression fixed in SDK 57 patches — [Expo SDK 56](https://expo.dev/changelog/sdk-56); [expo #46519](https://github.com/expo/expo/issues/46519)
- `InteractionManager` removed in SDK 58 / RN 0.88 — [Expo blog](https://expo.dev/blog/how-to-upgrade-to-sdk-58)

### Inferences

**A. Playbook — the 25 rules**
1. Per-frame state lives in shared values or refs, never in React/Zustand state.
2. The JS thread does no work per frame during gameplay; JS only handles discrete events.
3. Gestures run as UI-thread worklets; `scheduleOnRN` only on gesture end or hit.
4. Animate transform and opacity only; no layout-prop animation in gameplay.
5. Time-based motion (`dt`), correct at 60 and 120 Hz.
6. Enable `IOS_SYNCHRONOUSLY_UPDATE_UI_PROPS` + `USE_COMMIT_HOOK_ONLY_FOR_REACT_COMMITS` (check conflicts).
7. React Compiler on; zero compiler bail-outs in `src/game/**`; atomic Zustand selectors.
8. No `InteractionManager`; use `requestIdleCallback`/`startTransition` for deferred work.
9. One Skia Canvas per scene; atlases ≤2048²; paints, shaders and paths created once.
10. Rive: Nitro runtime, one shared file per character, `frameRate` capped on idle loops, ≤3 active instances.
11. R3F: `frameloop="demand"` outside active 3D play; dpr ≤2; ≤100 draw calls; instancing; no allocation in `useFrame`; dispose on unmount; one GL context at a time.
12. Never stack live Skia + Rive + GL surfaces in the same frame unless profiled.
13. Audio session set once at startup; SFX preloaded and pooled; players released on exit.
14. Haptics fire with the visual and are throttled; iPads with no haptics no-op cleanly.
15. Fonts embedded via the config plugin; splash held until fonts and home essentials load, with a 4 s timeout.
16. Every image and media surface has explicit size or aspectRatio; game sprites use `transition={0}`.
17. RTL set natively in app config; game-world coordinates are never mirrored; LTR isolation for numbers and math.
18. Nikud text: lineHeight ≥1.5×, vertical padding, tested at max Dynamic Type; in-game labels capped by `maxFontSizeMultiplier`.
19. Fixed logical game resolution scaled into the safe area; orientation locked.
20. Error boundary at the root and per scene; scenes can be rebuilt from persisted state.
21. Pause everything on `inactive`; persist on `inactive`; resume to a "tap to continue" screen.
22. Memory-warning handler clears caches; per-scene memory budget enforced.
23. SQLite: WAL, async API only, batched transactions at checkpoints; no AsyncStorage for big data.
24. Offline-first; every network call has a timeout plus AbortController; never block UI on network.
25. Perf decisions only on Release builds on the oldest supported iPad.

**B. Performance budgets (target device classes: "Low" = 3 GB iPad e.g. iPad 9th gen A13; "Mid" = iPad 10th gen / iPhone 13; "High" = ProMotion iPhone/iPad Pro)**

| Metric | Budget | Notes |
|---|---|---|
| Frame time, 120 Hz devices | ≤8.3 ms UI/GPU; p95 ≤8.3, p99 ≤16.7 | Low Power Mode → 60 Hz |
| Frame time, 60 Hz devices | ≤16.7 ms; p99 ≤25 ms | Low-class iPads are 60 Hz |
| JS thread work per frame during gameplay | ≤4 ms average; no JS task >50 ms during play | Hermes GC pauses count |
| Animation hitch ratio (Instruments) | <5 ms/s in gameplay, 0 hitches >100 ms | verify Apple thresholds |
| Cold start → splash hidden (home interactive) | ≤2.0 s Low, ≤1.2 s High | Release build, measured 5× median |
| Level load (tap → playable) | ≤1.5 s bundled, ≤3 s downloaded (with animation) | |
| Memory footprint steady (home) | ≤250 MB | Instruments footprint |
| Memory peak per scene (Low) | ≤650 MB; growth after 20 enter/exit cycles ≤20 MB | jetsam is likely ~1.2–1.5 GB on 3 GB (extrapolated) |
| Hermes bytecode bundle (iOS) | ≤8 MB; per-PR growth >100 KB flagged | Expo Atlas |
| App download size (IPA, thinned) | ≤150 MB initial | more via level packs |
| Per-level assets | ≤8 MB on disk, ≤150 MB decoded | |
| Single texture | ≤2048² (hero), ≤1024² (props) | 2048² RGBA ≈16 MB GPU |
| GL draw calls | ≤100/frame (Low), ≤50 preferred | |
| Triangles (3D scene) | ≤100k Low | [prior knowledge rule of thumb] |
| Rive active instances | ≤3 animating; file ≤200 KB each | |
| Skia Canvases mounted | ≤2 | |
| SFX latency (tap → audible) | ≤80 ms | 240 fps video measurement |
| Re-renders per correct answer | ≤N components (Reassure baseline) | no full-tree renders |
| CLS on web export | 0.00 after first paint | Playwright layout-shift observer |
| Crash-free sessions | ≥99.8%; hang rate ~0 | TestFlight/Organizer |

**C. Per-release test checklist**
- [ ] `npx expo-doctor` clean; Hermes version ≥ fixed build; no `InteractionManager` usage (including in node_modules, as a report).
- [ ] Reassure: no render-count or duration regressions over threshold.
- [ ] Bundle size within budget (Expo Atlas diff attached).
- [ ] Asset audit: all images have dims, textures ≤ budget, audio formats right, `.riv` sizes.
- [ ] Playwright web: RTL snapshots, nikud torture strings at 3 font scales, CLS = 0, no console errors.
- [ ] Maestro smoke flow (onboarding → level 1 → complete → home) passes on the Android emulator (Linux) and the iOS simulator (Mac).
- [ ] Device protocol (below) run on the Low iPad + one ProMotion iPhone; results logged in `perf/results/<version>.md`.
- [ ] Interruption tests: incoming call / Siri / alarm, Control Center pull-down, backgrounding mid-level, Low Power Mode on, airplane mode, silent switch on, Bluetooth headphones connect/disconnect mid-SFX.
- [ ] 30-minute soak: memory flat, no thermal "serious", battery drain recorded.
- [ ] Crash/hang reports from previous TestFlight reviewed (Organizer / MetricKit / Sentry).

**D. CI checks the agent can automate on Linux (GitHub Actions)**
1. `tsc --noEmit`, ESLint (react-compiler rules plus custom bans: `.value` in render, `InteractionManager`, `I18nManager.forceRTL`, `new THREE.*` inside `useFrame`, `createAudioPlayer` inside event handlers, images without size).
2. `npx expo-doctor`; a script asserting the resolved versions of hermes, reanimated, worklets, Skia, Rive and nitro-modules against an allow-list.
3. Jest + Reassure (`reassure --baseline` on main, compare on PR; fail on >10% duration or any added render).
4. `npx expo export --platform ios --dump-sourcemap` → bundle size check, plus an Expo Atlas artifact upload.
5. Asset linter: walk `assets/`, fail if any PNG/WebP is > the max dims or bytes, any texture is not a power of two, `.riv` > 200 KB, any audio is not AAC/M4A/WAV, or a level pack is > 8 MB.
6. Playwright on `npx expo export --platform web` served statically: visual diffs (RTL), layout-shift observer, long tasks, rAF frame histogram as a smoke test only.
7. Optional: Android emulator job with Maestro + Flashlight score (trend only, non-blocking).
8. Hermes regression smoke via EAS Build plus a device farm, if budget allows. Otherwise this stays a manual device step.

**E. Device test protocol (parent's Mac + iPad/iPhone via Remote Control)**
1. **Build:** `eas build --profile perf --local` or `npx expo run:ios --configuration Release --device` on the Mac. The perf profile is Release with the dev overlay enabled via an env flag and source maps uploaded.
2. **Prep device:** reboot, wait 2 min, battery >50%, unplugged for battery tests (plugged for profiling), Low Power Mode off (then a second pass with it on), brightness fixed at 50%, Do Not Disturb on.
3. **Cold start ×5:** force-quit, launch with `xcrun xctrace record --template 'App Launch'`; record the median time to splash hide (log a `performance.mark` + native log line).
4. **Gameplay hitch run:** run the Maestro flow for level 1 and the 3D level while recording `Animation Hitches` (60 s each); export the hitch ratio and the longest hitch.
5. **Time Profiler** on the worst level: check that the JS/Hermes thread is mostly idle during play and that the main thread has no long Yoga layout or commit spikes.
6. **Memory:** Allocations + VM Tracker; enter/exit each scene 20× via a Maestro loop; footprint must return to baseline ±20 MB. Note the peak per scene. Trigger Simulate Memory Warning (simulator) and confirm caches drop.
7. **GPU:** Metal System Trace on the Skia, Rive and 3D scenes; GPU frame time ≤ 6 ms at 120 Hz.
8. **Audio latency:** film tap + sound at 240 fps on a second phone; measure the gap across 10 taps.
9. **Soak:** 30 min of autoplay (Maestro loop) with the Energy Log; record thermal state, battery % drop and memory trend.
10. **Interruption matrix** (from §C), plus orientation, Split View / Stage Manager resize and Dynamic Type at the largest size.
11. **Record** results in `perf/results/<version>.md` against the budgets table. Any red item blocks release.

### Gaps
- All budget numbers are informed targets, not published standards. Calibrate them against the first measured baseline on the parent's actual oldest iPad.
- I could not confirm which old 3 GB iPads iPadOS 27 / SDK 58's minimum iOS version still supports; check this before naming the "Low" device.
