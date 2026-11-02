# Tech: 3D / 2.5D Explorable World Layer (Expo SDK 58 / RN 0.88, iPad + iPhone, built by an AI agent in a Linux container)

> Research note, written 2026-11-02. Network limits: from this container, docs.expo.dev, expo.dev, r3f.docs.pmnd.rs, margelo.github.io, buttondown.com and newly.app could not be fetched directly (DNS blocked). Facts about those pages come from search-result summaries, not full-page reads. github.com pages could be fetched. Claims that rest only on the researcher's background knowledge are labeled "inference" and kept out of the Cited Findings.

---

## Q1. Which 3D stack options exist, what is their 2026 status, which work in Expo Go vs need a dev build, and which also run on web (for Expo web export + Playwright)?

### Takeaway
Use three.js + React Three Fiber **v9** (`@react-three/fiber/native` + `expo-gl` + `expo-asset`). It is the only option that gives one codebase that renders on iOS **and** in Chromium through Expo web, which is what the Playwright testing loop needs. Be aware of two 2026 facts. (1) R3F v10 moved native support into a separate `@react-three/native` package, which is "pre-alpha, DO NOT USE YET", so **pin R3F v9**. (2) The App Store build of Expo Go stops at SDK 54, so on an iPhone or iPad an SDK 58 project needs a TestFlight Expo Go (`eas go`, which requires an Apple Developer account) or a dev build from day one. react-native-filament, react-native-wgpu, Babylon React Native, Unity and Godot all need dev builds, and none of them gives the same web-screenshot path (react-native-wgpu partly does).

### Cited Findings
**R3F native (expo-gl)**
- R3F's native entry point is `@react-three/fiber/native`. It relies on `expo-gl` (WebGL2 bindings) and `expo-asset` (so three.js loaders work with Metro). Install with `npx expo install expo-gl` plus `three @react-three/fiber`, and import drei from its `/native` path. Fiber v8 pairs with React 18 and v9 with React 19. — [R3F Installation docs](https://r3f.docs.pmnd.rs/getting-started/installation) (via search summary)
- The docs warn that iOS simulators often have incomplete or unreliable OpenGL ES support, which can cause EXC_BAD_ACCESS crashes, so test on a physical device. — [R3F Installation docs](https://r3f.docs.pmnd.rs/getting-started/installation) (via search summary)
- **R3F v10 change:** `@react-three/fiber/native` becomes `@react-three/native`, versioned on its own, and core has no Expo dependencies. The v10 guide says `@react-three/native` "is not on npm yet" and that "React Native apps stay on v9 until it is published." — [R3F v10 Migration Guide](https://r3f.docs.pmnd.rs/next/migration/v10); [native migration page](https://r3f.docs.pmnd.rs/next/development/native-migration) (via search summary)
- The `pmndrs/native` repo README says "DO NOT USE YET", "Unstable, incomplete, and probably broken", and "raw, pre alpha and unusable". It lists requirements of Expo SDK ≥43, RN ≥0.78 and React ≥19. The author says native work is being "pushed toward WebGPU" (an `apps/webgpu-example` uses react-native-webgpu), and a `GLContextProvider` API for WebGPU or custom GL is planned but not finished. R3F v10 is "in early alpha". — [pmndrs/native GitHub](https://github.com/pmndrs/native)
- Reliability history: GitHub issue #3517 ("Warning: React Native is no longer supported"), opened Apr 2025, reported blank screens in Expo Go with Expo 52 / R3F 8. A community sample reportedly worked on Expo 53.0.11 / RN 0.79.3 / React 19.0.0 on iOS and Android devices. The maintainers closed the issue on Jun 29, 2025 via PR #3539 ("fix crash on rerendering GLView with new arch"). Workarounds reported in the thread: three.js loading more than once (deduplicate it), pass position/rotation as tuples instead of `Vector3`/`Euler`, and load textures manually with `useTexture` when GLTF-embedded textures fail. — [pmndrs/react-three-fiber #3517](https://github.com/pmndrs/react-three-fiber/issues/3517)
- An older post reported a version mismatch: Expo SDK 53 shipped expo-gl@15 while R3F depended on an older expo-gl. — [Medium: Current state of RN tech stack: Expo & R3F](https://medium.com/@codedomen/current-state-of-react-native-tech-stack-expo-react-three-fiber-0678d1016c5b) (via search summary; may be resolved)

**Expo Go availability for SDK 55+ (critical for "Expo Go first")**
- Expo's troubleshooting docs say the Apple App Store version of Expo Go stops at SDK 54, and SDK 55 and later are not available there. For SDK 55+ on a physical iPhone, open sign.expo.dev, pick the SDK version, and install that Expo Go build. Periodic re-signing is required. — [Expo docs: Expo Go version mismatch](https://docs.expo.dev/troubleshooting/expo-go-version-mismatch/) (via search summary)
- The `eas go` command builds Expo Go on EAS and deploys it to your TestFlight internal team. It requires Apple Developer Program membership. Expo's May 2026 changelog said the SDK 55 build was still awaiting App Store approval as of May 4, and that the SDK 56 TestFlight External Beta group was "at capacity". — [Expo changelog: Expo Go and the App Store in May 2026](https://expo.dev/changelog/expo-go-and-app-store-may-2026) (via search summary)
- Expo Go is a fixed prebuilt app. Native libraries that are not bundled in it require a development build (debug build + expo-dev-client), which can include any native library. — [Expo blog: Expo Go vs Development Builds](https://expo.dev/blog/expo-go-vs-development-builds); [Expo dev builds FAQ](https://docs.expo.dev/develop/development-builds/faq/)

**react-native-filament (Margelo)**
- A real-time, physically based 3D engine for RN that uses Google Filament for rendering and Bullet3 for physics. It renders with Metal on iOS (OpenGL/Vulkan elsewhere) on separate threads and supports both old and new architecture. Out of the box it loads only `.glb` ("only glb supported yet"). MIT licensed, about 1.4k stars, "provided as is", with paid support through Margelo. — [margelo/react-native-filament GitHub](https://github.com/margelo/react-native-filament)
- It depends on `react-native-worklets-core`, which needs a Babel plugin (`processNestedWorklets: true`, or that option on the Reanimated plugin if you use Reanimated), followed by `pod install`. — [Filament Getting Started](https://margelo.github.io/react-native-filament/docs/guides) (via search summary)
- A small third-party plugin, `expo-plugin-worklet-cleanup`, targets iOS crashes on force-quit in apps that use worklets and lists react-native-filament among the covered libraries. I found no official Filament Expo config plugin. — [pkgstats: expo-plugin-worklet-cleanup](https://www.pkgstats.com/pkg:expo-plugin-worklet-cleanup) (via search summary)

**react-native-wgpu (WebGPU, William Candillon / Shopify)**
- "WebGPU for React Native, powered by Dawn". MIT licensed, about 1.3k stars, 27 open issues and 28 open PRs at fetch time. — [wcandillon/react-native-webgpu GitHub](https://github.com/wcandillon/react-native-webgpu)
- Requires RN ≥0.81 and the New Architecture. Latest version listed: v0.5.11. There is an Expo template (`npx create-expo-app@latest -e with-webgpu`) that works with R3F on iOS, Android and Web. Three.js needs a Metro tweak to resolve its WebGPU build, and R3F needs a `package.json` patch (patch-package) so it resolves to the WebGPU entry point. Running in the iOS simulator requires turning off "Metal Validation". — [pkgstats: react-native-wgpu](https://www.pkgstats.com/pkg:react-native-wgpu); [fork README](https://github.com/wonkyDD/react-native-webgpu) (via search summary)
- A 2026 practitioner review ("Sensei Notes #06 - Can R3F run on mobile?") found the basics work, but events are a pain point and extra dependencies can break depending on the three/R3F version. It recommends a WebView for most apps and native WebGPU only where performance matters and rough edges are acceptable. — [Sensei Notes #06](https://buttondown.com/wawasensei/archive/sensei-notes-06-can-r3f-run-on-mobile-plus) (via search summary; page not fetchable)

**WebView + three.js**
- Several sources say Safari 26 (iOS 26) ships WebGPU on by default. — [AppDeveloperMagazine: WebGPU in iOS 26](https://appdevelopermagazine.com/webgpu-in-ios-26/); [vr.org: WebGPU baseline 2026](https://vr.org/articles/webgpu-baseline-2026-three-js-webxr-default) (via search summary)
- In a 2024 Apple Developer Forums thread, the WebGPU feature flags in Safari on iOS 18.2 did not give **WKWebView** GPU access, because those flags affect only Safari. I found no confirmation that a WKWebView in a shipped iOS 26 app gets WebGPU by default. — [Apple Dev Forums 770862](https://developer.apple.com/forums/thread/770862)
- An old, unresolved Cordova issue reports WebGL in iOS WebViews becoming much slower from iOS 15 onward. — [apache/cordova-ios #1246](https://github.com/apache/cordova-ios/issues/1246) (via search summary)

**Babylon React Native**
- I found no current statement of official Expo support. Forum users report failed builds when adding Babylon Native to Expo apps. One answer says Expo works with it "only for asset management", and Babylon Native is described as "public preview in source form only" (older README). An Oct 2025 forum thread asks which package combination works on RN 0.82. — [Babylon forum: Cannot initialize Babylon Native with Expo](https://forum.babylonjs.com/t/cannot-initialize-babylon-native-with-expo/57707); [Babylon forum: Babylon React Native with expo?](https://forum.babylonjs.com/t/babylon-react-native-with-expo/47964); [Babylon Native README (jsDelivr)](https://cdn.jsdelivr.net/npm/@babylonjs/react-native@2.0.5/shared/BabylonNative/Repo/README.md) (via search summary)
- Background: recent Expo SDKs make the New Architecture mandatory (secondary sources). — [DEV: Expo SDK 56 and React 19.2](https://dev.to/davekurian/react-native-ecosystem-advances-with-expo-sdk-56-and-react-192-updates-in-2026-3df5) (via search summary)

### Inferences
- **Compatibility matrix (inference; the Expo Go and web columns combine the sources above with background knowledge):**

| Option | Expo Go (SDK 58) | Dev build | Web (Expo web + Playwright/Chromium) | Notes |
|---|---|---|---|---|
| R3F v9 + expo-gl + three.js | Yes, but only with a TestFlight/sign.expo.dev Expo Go on iOS (expo-gl is a first-party Expo module bundled in Expo Go; not re-verified for SDK 58) | Yes | Yes (R3F web Canvas is plain three.js WebGL) | **Recommended primary** |
| react-native-wgpu + three WebGPU + R3F | No (custom native) | Yes | Yes (browser WebGPU; headless Chromium may need flags or fall back) | Promising next step; event and dependency rough edges |
| react-native-filament | No | Yes (Nitro/worklets, pods) | No | Best native perf and PBR, but breaks the "test on web" loop |
| WebView + three.js bundle | Yes (react-native-webview is in Expo Go) | Yes | Yes (the same HTML runs in Chromium directly) | Easiest dual-target; WASM available; touch/bridge latency; GPU-in-WKWebView behavior to verify |
| Babylon React Native | No | Fragile | Babylon.js on web, but a separate code path | Not recommended |
| Unity (react-native-unity) / Godot | No | Needs a Mac + Xcode + Unity/Godot export pipeline; the AI agent in a Linux container cannot build or iterate on it | Separate WebGL export only | Reject for this project |
| Skia 2.5D (isometric/parallax) | Yes (`@shopify/react-native-skia` is in Expo Go) | Yes | Yes (CanvasKit WASM on web) | **Fallback**; best for the 2.5-year-old |

- R3F v9 is the only stack where an agent in a Linux container can (a) write scene code, (b) run `expo export --platform web`, and (c) screenshot or measure it in Chromium with Playwright, with the same React components then running on device. That is why it wins despite the maintenance risk.
- Keep the scene layer renderer-agnostic: plain three.js scene graph + R3F components, no expo-gl-specific calls outside one `<WorldCanvas>` wrapper. Migrating to `@react-three/native` (when published) or to react-native-wgpu is then a wrapper swap.
- The Expo Go situation changes the plan. Because App Store Expo Go stops at SDK 54, "Expo Go first" on the parent's iPad/iPhone means either `eas go` (TestFlight, needs a paid Apple Developer account) or the sign.expo.dev flow. If the family already needs a paid Apple account for dev builds, going straight to an EAS-cloud dev build (`eas build --profile development`, no Mac needed for the build itself) may be simpler. The parent's Mac is then needed only for local Xcode debugging or profiling (Instruments).

### Gaps
- Could not confirm from a primary page that R3F v9 + expo-gl works on Expo SDK 58 / RN 0.88 specifically. The newest confirmed working combination found is Expo 53 / RN 0.79 / React 19 (issue #3517). This must be verified in spike day 1.
- Could not read Expo's expo-gl docs (blocked) to confirm current limitations (antialias/MSAA support, worklet/UI-thread rendering, Expo Go inclusion for SDK 58).
- Could not confirm whether `@react-three/native` has been published to npm as of Nov 2026.
- No measured fps comparison between expo-gl, react-native-wgpu, Filament and WebView on the same iPad scene was found.

---

## Q2. Physics: Rapier on native? cannon-es? custom kinematics?

### Takeaway
Do **not** depend on `@react-three/rapier` on native: Rapier is WASM, Hermes has no WebAssembly, and the polyfill route is reported broken or slow. For a kids' explorer, write simple custom kinematics: a character controller with capsule-vs-ground raycasts against a low-poly collision mesh, sphere or circle trigger volumes for collectibles, and scripted tweens for "physics-y" juice. Add cannon-es (pure JS) only if a mini-level truly needs rigid bodies.

### Cited Findings
- `@react-three/rapier` wraps the Rapier WASM engine and lazily initializes it inside Suspense. v1 pairs with React 18 / Fiber v8, and v2 with React 19 / Fiber v9. — [pmndrs/react-three-rapier GitHub](https://github.com/pmndrs/react-three-rapier); [npm @react-three/rapier](https://www.npmjs.com/package/@react-three/rapier) (via search summary)
- From GitHub issue #383: "React Native for iOS and Android doesn't support WASM out of the box". Users tried polywasm (WASM translated to JS), which a commenter expected to be "very slow". One report (Aug 2024; RN 0.72, Expo 49, rapier 1.4) hit `TypeError: Cannot read property 'prototype' of undefined, js engine: hermes`. A maintainer-level commenter "wouldn't expect this to work or ever be performant". A WebView (which supports WASM) was suggested instead. — [react-three-rapier #383](https://github.qingwei-s.workers.dev/pmndrs/react-three-rapier/issues/383) (mirror, via search summary)
- react-native-filament bundles native Bullet3 physics, an option only if Filament is the renderer. — [margelo/react-native-filament](https://github.com/margelo/react-native-filament)

### Inferences
- Recommended physics approach (inference): (1) Ground snapping: each frame, cast a downward ray from the hero against a merged, simplified "walkable" mesh, using `three-mesh-bvh` for fast raycasts (pure JS, works on Hermes and web). (2) Walls: a few axis-aligned boxes or circle obstacles with simple push-out resolution. (3) Collectibles: distance checks (`hero.position.distanceTo(item) < r`). (4) Jumps and bounces: an analytic parabola or a spring tween. This is deterministic, so Playwright can assert positions, and it costs far less CPU than a full solver.
- cannon-es is pure JavaScript and should run on Hermes (not verified on RN 0.88). Use it only for a contained mini-level (for example, a block-stacking toy), with a small body count (<30) stepped at 30 Hz.
- If a mini-level needs heavy physics, it could run inside the WebView fallback, where Rapier WASM works.

### Gaps
- No source confirming cannon-es performance on Hermes in 2026.
- Did not verify whether the Hermes version in RN 0.88 has gained WebAssembly support (it is not known to have it).

---

## Q3. Performance budgets on iPad (A12+) and iPhone: draw calls, polys, textures, instancing, LOD, 60fps, battery/thermal

### Takeaway
Target ≤100 draw calls, ≤100–150k visible triangles, 1–2 texture atlases at ≤1024² (2048² max for terrain), cap the pixel ratio at about 2, turn off MSAA and post-processing, and use InstancedMesh for every repeated prop or collectible. Lock to 60 fps on iPad, and offer a 30 fps "battery saver" mode for long sessions. Validate on the oldest family device with `renderer.info` and Xcode Instruments.

### Cited Findings
- Draw-call targets: one guide says keep total draw calls under 500 per frame, ideally under 100, and measure with `renderer.info.render.calls`. — [MetaDesign Solutions: three.js performance](https://metadesignsolutions.com/blog/threejs-performance-optimization-best-practices)
- Another guide puts comfortable 60 FPS on mid-range hardware at roughly 100–150 draw calls, says mid-range mobile struggles past a few hundred, and recommends `BatchedMesh` (r159+, improved r170) for same-material/different-geometry objects and `mergeGeometries` for static parts. — [hontran.dev: three.js performance optimization](https://www.hontran.dev/blog/three-js-performance-optimization)
- InstancedMesh benchmark: about 19,365 meshes went to 2 draw calls, with render CPU time falling from 28.5 ms to 0.5 ms at p95 (measured on an M1 Pro with software WebGL, so directional only). — [skillselion: threejs-perf](https://skillselion.com/skills/opusgamelabs/game-creator/threejs-perf)
- Mobile tips: frustum culling, LOD for distant objects, disabling antialiasing and limiting pixel ratio on mobile, and disposing GPU resources on unmount. — [r3f-performance skill](https://skills.sh/bbeierle12/skill-mcp-claude/r3f-performance); half-resolution post-processing as a major mobile win — [Digital Strategy Force](https://digitalstrategyforce.com/journal/how-do-you-optimize-threejs-performance-for-mobile-devices/) (its "0.1 ms per draw call" figure is unsourced; do not rely on it)

### Inferences
- **Proposed budget for the vertical slice (inference, to be validated on device):**
  - Frame: 16.6 ms at 60 fps. JS/React work per frame ≤4 ms (no React re-renders in the frame loop; mutate refs inside `useFrame`).
  - Draw calls ≤80 (iPad A12 / iPhone XS class), ≤120 on newer devices.
  - Triangles on screen ≤100k (A12) to 150k. Hero about 3–6k tris. Collectible about 300–800 tris, instanced. Props 200–2k tris each.
  - Textures: palette/gradient atlas, one 512² or 1024² per biome. Prefer vertex colors or a tiny palette texture (Kenney/Quaternius-style flat shading). Total GPU texture memory ≤64 MB.
  - Materials: `MeshToonMaterial` or `MeshLambertMaterial`, not `MeshStandardMaterial`/PBR. One directional light + hemisphere/ambient. Bake shadows into vertex colors or use a blob shadow decal under the hero instead of real-time shadow maps, or at most one 1024² shadow map on the hero only.
  - Pixel ratio: `dpr={[1, 2]}`. On iPad, consider 1.5 if fill-bound.
  - Bones: hero skeleton ≤30–40 bones, one skinned mesh.
  - Memory: keep each level ≤50 MB of assets. Dispose on level exit.
  - Thermal/battery: add a `frameloop="demand"` idle mode (stop rendering when nothing moves, which matters for toddlers who stare at the screen), a 30 fps cap option, and pause on app background (AppState).
- **Compression caveat (inference):** Draco and Meshopt decoders and KTX2/Basis texture transcoders in three.js normally rely on WASM or Web Workers, which are not available on Hermes/expo-gl. Use Draco/Meshopt/KTX2 only for the web build. Ship native GLBs with `gltf-transform` optimizations that need no runtime decoder (dedup, weld, `simplify`, quantize where three.js supports it natively without an extension decoder, resize textures to power-of-two, PNG/JPEG). Measure GLB load time on device.
- Bundle assets locally via expo-asset (family app, offline use). Do not download at runtime.

### Gaps
- No device-measured numbers for expo-gl on A12 iPads (triangles/sec, draw-call ceiling). The spike must produce them.
- Could not confirm whether expo-gl currently supports MSAA, or which WebGL2 extensions it exposes (needed for KHR_mesh_quantization, instancing).
- Did not verify whether `meshoptimizer`'s JS (non-WASM) decoder fallback works in Hermes.

---

## Q4. Controls and camera for kids aged 2.5 / 5.5 / 8.5

### Takeaway
Use tap-to-move (tap the ground and the robot walks there along a path) as the universal default, with an auto-follow camera the child never has to manage. Add an optional floating virtual joystick for the 8.5-year-old only, and a one-touch "auto-run" mode for mini-levels in the style of Super Mario Run. Avoid tilt as a primary control.

### Cited Findings
- Super Mario Run: the character runs forward automatically and the only control is jump. A short tap gives a short hop, holding gives a longer jump. Miyamoto: "We wanted to ask how we can make the experience even simpler for somebody who had never played a Mario game before." He called the touchscreen "quite possibly the simplest controller they've ever worked with". — [iMore: Miyamoto on Super Mario Run design](https://imore.com/game-design-design-philosophy-shigeru-miyamoto); [GSMArena review](https://www.gsmarena.com/super_mario_run_review-blog-22234.php); [Pocket-lint preview](https://www.pocket-lint.com/games/news/nintendo/138796-super-mario-run-preview-playing-the-game-with-miyamoto-himself)
- Reviewers noted that despite the one-direction control "it rarely feels limiting", but that levels require a lot of repetition and retries to 100% them. — [GSMArena review](https://m.gsmarena.com/super_mario_run_review-blog-22234.php); [Nintendo World Report review](https://www.nintendoworldreport.com/review/43802)

### Inferences
- **Per-age control profile (inference, based on general kids' UX practice; verify with play-testing):**
  - **2.5 y:** Tap anywhere and the robot walks there. Tapping an object makes the robot go to it and interact (auto-collect). No fail states and no jumping gaps. Large hit areas (about ≥64 pt), with forgiving raycasts that snap to the nearest walkable point or interactable within a radius. Camera fully automatic. The world is small, like a "diorama", so the child cannot get lost. Toddlers drag and multi-touch by accident: ignore second touches, debounce, and treat a drag as a "move toward finger" follow.
  - **5.5 y:** Tap-to-move plus a big "jump" button or tap-on-robot to jump. Mini-levels use auto-run with tap-to-jump (Super Mario Run pattern). Hints via a glowing trail toward the next collectible after N seconds of idleness.
  - **8.5 y:** Optional floating joystick (appears where the left thumb lands; deadzone about 15%; relative input), jump/action button on the right, optional camera swipe-to-orbit with auto-recentering.
- **Camera:** A third-person follow camera with spring damping, fixed pitch (about 35–50°), automatic yaw toward the movement direction, collision push-in (or simply no tall occluders near paths, plus fading of occluding props). No pinch-zoom for young kids, or zoom clamped to 2 levels.
- **Tilt (expo-sensors DeviceMotion):** Use only as a bonus mini-game (for example, roll a ball). It is unreliable for toddlers, who hold the iPad flat on laps or tables, and it creates orientation, calibration and motion-sickness issues.
- **Accessibility:** One-hand play. Every control reachable in the iPad landscape lower thirds. Audio-first prompts (Hebrew voice-over) because the 2.5- and 5.5-year-olds can't read. Color-blind-safe collectible cues (shape + sparkle + sound, not only color). A "calm mode" with reduced camera motion. Honor the OS Reduce Motion setting via `AccessibilityInfo.isReduceMotionEnabled`.
- **RTL note:** The world itself is not mirrored. Only the 2D HUD uses RTL layout. Place the joystick on the left and actions on the right regardless of RTL (thumb ergonomics), or let a settings toggle swap them.

### Gaps
- Found no primary sources on Toca Boca, Sago Mini or LEGO mobile control design specifics (search returned nothing citable). Those claims should not be cited.
- No research found on joystick usability at ages 5–8. The age thresholds above are design judgment.

---

## Q5. Asset pipeline: Blender → glTF/GLB, CC0 kits, AI generation, rigging/animation, toon/outline shading, Hebrew text

### Takeaway
Build the world from CC0 kits (Kenney + Quaternius, both in GLB/glTF with a consistent flat low-poly style), make the robot hero from a Quaternius CC0 animated robot or a Meshy/Tripo-generated mesh auto-rigged and cleaned in Blender, and run every asset through a scripted `gltf-transform` step in CI (headless Blender also runs on Linux). Use toon materials and inverted-hull outlines for the "polished" look. Never render Hebrew in 3D; use the 2D overlay instead.

### Cited Findings
- **Kenney:** 270+ packs and tens of thousands of assets under CC0 (no attribution required), in GLB/GLTF, OBJ and FBX, including characters, environments, vehicles, monsters and an animation library. — [Cinevva: Best Free 3D Model Sites for Games (2026)](https://app.cinevva.com/guides/free-3d-model-sites) (third-party; confirm on kenney.nl)
- **Quaternius:** about 80 themed packs and 1,100+ models under CC0, in GLTF/GLB and FBX, as free per-pack ZIPs, described as game-ready with clean topology. — [Cinevva guide](https://app.cinevva.com/guides/free-3d-model-sites); [licenseorg: Quaternius](https://licenseorg.com/guide/3d-assets/quaternius)
- An animated low-poly robot with 14 animations (idle, jump, dance, punch, run, ...) is published under CC0 on OpenGameArt, and Quaternius has robot/mech-tagged CC0 packs. Patreon support is requested, but that is not a license condition. — [OpenGameArt: Animated Lowpoly Robot](https://opengameart.org/content/animated-lowpoly-robot)
- **Meshy:** Meshy-6 (Jan 2026) added a "Low Poly Mode" for real-time engines. The documented pipeline is Generate → Remesh → AI Texturing (PBR) → Rigging (characters) → Export (FBX/GLB). — [Meshy docs: Game Assets](https://docs.meshy.ai/en/webapp/guides/use-cases/game-assets); [okmiku: Meshy-6 vs Tripo v3.1](https://okmiku.com/blog/meshy-6-vs-tripo-v31)
- **Tripo:** a "Smart Mesh" low-poly option, auto-rigging in beta, export to GLB/FBX/OBJ/STL. Vendor pricing comparison: Tripo Pro $19.90/mo for 3,000 credits vs Meshy Pro about $20/mo for 1,000 credits. — [Tripo blog: best Meshy alternative](https://www.tripo3d.ai/blog/best-meshy-ai-alternative-for-game-ready-3d-assets) (vendor source)
- A competitor-run benchmark gives Tripo the edge on quad-dominant low-poly topology and Meshy the edge on stylized texturing. Hacker News commenters were skeptical ("too much slop, broken topology, made-up dimensions"). No source confirmed that rigs from either tool import cleanly. — [Yugma: Meshy vs Tripo benchmark](https://yugmastudio.web.app/blog/meshy-vs-tripo-game-asset-benchmark/) (competitor source; treat as unverified)
- react-native-filament accepts only `.glb`, which is relevant if Filament is ever adopted. — [margelo/react-native-filament](https://github.com/margelo/react-native-filament)

### Inferences
- **Pipeline (inference):**
  1. Source: Kenney (e.g., nature, platformer and space kits), Quaternius (robots, animals, nature), and Poly Pizza (check each model's license: many are CC-BY, not CC0, so keep an attribution file).
  2. Hero robot: start from a CC0 Quaternius robot (already rigged and animated) for the spike. Later, generate a custom design in Meshy/Tripo (low-poly mode), retopologize or decimate in Blender to about 5k tris, auto-rig (Meshy/Tripo rig, or Mixamo; Mixamo works on humanoids, so robot proportions must be humanoid-ish), then add 6–8 clips: idle, walk, run, jump, happy/celebrate, wave, "found it!", sad-but-cute.
  3. Headless Blender (`blender -b -P export.py`) runs in a Linux container. Script normalization: scale to meters, apply transforms, merge static meshes by material, generate a simplified collision mesh, and export GLB.
  4. `gltf-transform` CLI (Node, runs in CI): dedup, prune, weld, resample animations, simplify, texture resize to ≤1024, and an optional palette-texture conversion. Emit two variants: native (no decoder-dependent extensions) and web (Draco/Meshopt + KTX2 allowed).
  5. A budget linter script fails CI if any GLB exceeds its triangle, material, texture-size or bone budget.
- **Look:** `MeshToonMaterial` with a 3–4 step gradient map, plus an inverted-hull outline (a duplicate back-face mesh scaled along normals, which is cheap and works on WebGL1/2 and expo-gl) rather than screen-space post-process outlines (expensive on mobile, and post-processing may be unsupported on expo-gl). Use a soft gradient sky dome and bright saturated palettes. "Polish" comes mostly from animation juice: squash and stretch on collect, particle sparkles (instanced quads), camera bounce, and haptics + sound on every success.
- **Hebrew text:** Do not use 3D text (troika/drei `<Text>` requires font shaping, and Hebrew RTL/nikud shaping is error-prone). Render labels, speech bubbles and counters as 2D RN views positioned by projecting 3D world positions to screen coordinates.
- **Licensing hygiene:** Keep a `LICENSES/assets.json` manifest listing each asset's source URL and license. CC0 needs no attribution but the record is still useful. AI-generated assets fall under each vendor's terms of service (check commercial and ownership terms even for a family-only app).

### Gaps
- Could not verify on kenney.nl / quaternius.com directly (third-party summaries only).
- No independent evaluation found of Meshy/Tripo auto-rig quality on non-humanoid robots.
- Poly Pizza licensing (mixed CC0/CC-BY) was not verified in this session.

---

## Q6. Integration with 2D UI (Reanimated/Skia overlays), audio spatialization, haptics

### Takeaway
Stack the layers: `<Canvas>` (R3F, full-screen) at the bottom, an RN/Reanimated HUD on top (RTL, Hebrew, big buttons), and optional Skia effects in a separate overlay canvas. Bridge game state through a tiny store (zustand, which is also pmndrs) that the frame loop mutates without React re-renders. Use stereo panning by distance instead of true 3D spatial audio, and expo-haptics on collect and jump.

### Cited Findings
- react-native-filament runs rendering on separate threads and relies on worklets (`react-native-worklets-core`), and its Babel config must coexist with Reanimated's plugin (`processNestedWorklets`). — [margelo/react-native-filament](https://github.com/margelo/react-native-filament); [Filament docs](https://margelo.github.io/react-native-filament/docs/guides) (via search summary)
- The react-native-wgpu ecosystem includes a Software Mansion Labs project running three.js / TypeGPU on Reanimated worklets. — [relatedrepos: react-native-webgpu-worklets](https://relatedrepos.com/gh/software-mansion-labs/react-native-webgpu-worklets) (via search summary)

### Inferences
- With R3F + expo-gl, three.js runs on the JS thread, so heavy React work in the HUD (for example, re-rendering long lists) causes frame drops. Keep the HUD light and drive HUD animations with Reanimated shared values on the UI thread.
- Pointer events: R3F native maps RN touches to raycast events. For tap-to-move, raycast against the ground mesh only (one object, BVH-accelerated) instead of relying on R3F's event system across all meshes. This also sidesteps the event pain points reported for the WebGPU route.
- Audio: use `expo-audio`, which is in Expo Go. Compute pan and volume from hero-to-source distance and angle each N frames, which gives "good enough" spatialization. A true HRTF/WebAudio-style graph would need a native module such as react-native-audio-api (dev build) or the WebView route. Hebrew voice-over clips should be pre-recorded or TTS-generated offline and bundled.
- Haptics: `expo-haptics` (in Expo Go). Light impact on step-onto-collectible, success notification on level complete, selection on UI taps. iPads have no Taptic Engine, so haptics only work on iPhone. Design feedback to be audio + visual first.
- Transitions between the 3D world and 2D learning mini-games: unmount or pause the Canvas (`frameloop="never"`) when a 2D activity is open, which saves battery and GPU memory on older iPads.

### Gaps
- No source verified for react-native-audio-api spatial support on Expo SDK 58.
- Not verified whether expo-gl GLView can render under a transparent RN overlay without compositing cost on iOS (believed fine, untested).

---

## Q7. Recommendation: primary stack, fallback, vertical-slice spike plan with go/no-go criteria

### Takeaway
**Primary:** three.js (r17x/r18x) + `@react-three/fiber@9` (pinned) + `@react-three/drei` (native-safe subset) + `expo-gl` + `expo-asset`, custom kinematics with `three-mesh-bvh`, toon low-poly CC0 assets, and a 2D RN/Reanimated Hebrew HUD. **Fallback 1:** the same three.js scene code loaded in `react-native-webview` (identical web build, WASM and Rapier available). **Fallback 2:** a 2.5D Skia isometric/parallax world, which is also the best experience for the 2.5-year-old. **Watch list:** react-native-wgpu (WebGPU) and `@react-three/native`. **Reject:** Unity/Godot (needs a Mac and native toolchains, which the AI agent cannot drive from Linux), Babylon RN (fragile with Expo), and Filament as primary (no web path for Playwright, dev build only, though it is a strong native-only plan B if perf fails).

### Cited Findings
- R3F v10 native support is unpublished and pre-alpha, so RN apps must stay on v9. — [R3F v10 Migration Guide](https://r3f.docs.pmnd.rs/next/migration/v10); [pmndrs/native](https://github.com/pmndrs/native)
- The App Store Expo Go stops at SDK 54. SDK 55+ uses `eas go` (TestFlight, Apple Developer membership) or sign.expo.dev. — [Expo docs: version mismatch](https://docs.expo.dev/troubleshooting/expo-go-version-mismatch/); [Expo changelog May 2026](https://expo.dev/changelog/expo-go-and-app-store-may-2026)
- Rapier does not work on Hermes. — [react-three-rapier #383](https://github.qingwei-s.workers.dev/pmndrs/react-three-rapier/issues/383)
- A practitioner recommends a WebView for most R3F mobile apps and native WebGPU only when performance matters and rough edges are acceptable. — [Sensei Notes #06](https://buttondown.com/wawasensei/archive/sensei-notes-06-can-r3f-run-on-mobile-plus) (via search summary)
- Known R3F native pitfalls to pre-empt: duplicate three.js instances, `Vector3`/`Euler` props, GLTF texture loading, and GLView rerender crashes on the new architecture (fixed in PR #3539). — [R3F #3517](https://github.com/pmndrs/react-three-fiber/issues/3517)

### Inferences
**Vertical-slice tech spike (about 1–2 agent-weeks). All criteria measurable:**

*Spike scope:* One "island" level (Kenney nature kit, merged), a Quaternius CC0 robot hero with idle/walk/jump/celebrate, 20 instanced collectibles (stars/gears), tap-to-move + auto-camera, one auto-run mini-level (tap to jump over 5 obstacles), a Hebrew RTL HUD (counter "⭐ 3/20", a back button, a Hebrew voice prompt), collect sound + haptic, and a level-complete celebration.

*Day 1–2: Platform smoke test (go/no-go #1)*
- `npx create-expo-app` on SDK 58, install `three`, `@react-three/fiber@9`, `@react-three/drei`, `expo-gl`, `expo-asset`. A spinning GLB robot renders:
  - on web (`npx expo export -p web` + Playwright screenshot in Chromium, with pixel-diff baseline committed). **GO if the screenshot is non-blank and matches.**
  - on device via a TestFlight Expo Go (`eas go`) or an EAS dev build. **GO if it renders with no red screen, no EXC_BAD_ACCESS, and survives 10 background/foreground cycles plus 5 navigation unmount/remount cycles.**
- **NO-GO:** expo-gl/R3F v9 fails on SDK 58 and can't be fixed in about 1 day. Switch to Fallback 1 (WebView) and re-run.

*Day 3–5: Performance (go/no-go #2), on the oldest family device (target A12 iPad 8th gen / iPad mini 5, or whatever is oldest)*
- Full island scene: **≥55 fps p95 over a 60 s scripted walk on the oldest iPad**, ≤80 draw calls (`gl.info.render.calls` logged to the HUD in debug), ≤150k tris, JS frame time ≤6 ms, cold level load ≤3 s, memory ≤300 MB (Xcode Instruments on the parent's Mac).
- 10-minute soak: no thermal throttling below 45 fps, battery drain ≤ roughly 1%/min (rough check via iOS battery %).
- **NO-GO:** under 45 fps after DPR=1, shadow removal and instancing. Try Fallback 1 (WebView, compare fps). If still failing, either evaluate react-native-filament (dev build, accept losing web screenshots for the 3D layer and keep Playwright for the HUD only) or drop to Fallback 2 (Skia 2.5D).

*Day 6–8: Kid-usability (go/no-go #3)*
- Tap-to-move: the 2.5-year-old reaches 5 collectibles unassisted within 3 minutes. The 5.5-year-old completes the auto-run mini-level within 3 tries. The 8.5-year-old prefers either joystick or tap-to-move (record which).
- Misfire rate: ≤10% of toddler taps produce "nothing happens" (log taps that hit no walkable or interactable).
- Agent-testability: Playwright scripts can drive a deterministic route (inject `window.__game.moveTo(x,z)`), assert the collectible count and screenshot milestones. **GO if the CI suite runs headless in the Linux container in under 5 min.**

*Day 9–10: Asset pipeline (go/no-go #4)*
- Headless Blender + `gltf-transform` run in the container, and the budget linter passes. One AI-generated (Meshy/Tripo) prop and one AI-rigged character variant import with animations playing correctly on device and web. **NO-GO for AI characters** if rigs break, in which case stay on Quaternius/Kenney rigs.

**Architecture rules from day 1 (inference):**
- Separate `world/` (pure three.js + game logic; zustand store; no RN imports) from `platform/` (expo-gl Canvas wrapper, web Canvas, WebView host). Then swapping to `@react-three/native`, react-native-wgpu or a WebView only touches `platform/`.
- Deterministic simulation (a fixed-timestep game loop separated from rendering) enables Playwright assertions and replay.
- A debug overlay shows fps, draw calls, triangles and memory, toggleable in dev builds and exposed as `window.__perf` on web for Playwright to collect.
- Hebrew/RTL lives only in the RN HUD layer (`I18nManager`/`direction: 'rtl'` styles), never inside three.js.

### Gaps
- The Fallback 1 (WebView) perf delta vs expo-gl on iPad is unknown; the spike must measure it.
- The family's exact devices (oldest iPad/iPhone chip) are unknown. Budgets assume A12 as the floor.
- Whether the family has, or will pay for, an Apple Developer Program membership matters. Both `eas go` TestFlight and on-device dev builds effectively need it (a free personal team via the parent's Mac/Xcode is possible but expires every 7 days, which is background knowledge, not verified this session).
- Several official doc sites (docs.expo.dev, r3f.docs.pmnd.rs, margelo.github.io) were unreachable from this container. Allowing those domains would let the details above be re-verified against primary pages.
