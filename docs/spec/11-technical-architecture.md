# 11 — Technical Architecture (implementation-ready)

> Status: v1, 2026-11-02. Binding inputs: `00-foundation-decisions.md` sections **E (incl. E1a/E1b), G, H**,
> plus the A–D guardrails it inherits. Research log: `docs/research-log/tech.md`.
> Code name **Olam** (עולם), a placeholder until the family names the game. Every identifier below that
> contains `olam` is a placeholder too.
> Where this spec needs something 00 doesn't say, it decides and logs the reasoning. Where it **must**
> deviate (free-signing constraints), see §18 "Proposed change to 00".

---

## 0. Decisions at a glance

| Area | Decision | Why |
|---|---|---|
| Install path (E1a) | **Phase 1 (now): expo-dev-client debug build plus a "family" release build, installed over USB/Wi-Fi from the parent's Mac** via Claude Remote Control, with **free personal-team signing**. Builds re-sign automatically every ≤6 days (launchd + `tools/mac/resign.sh`). **Phase 0 concept prototypes: Expo web export viewed in iPad Safari** (Claude artifact or Cloudflare Pages preview). **Phase 2 (once the paid program is bought): EAS Build internal/ad hoc plus optional TestFlight.** | App Store Expo Go stops at SDK 54 (E1a update). There's no paid program yet. The parent has a Mac. |
| OTA | **EAS Update works in every phase** (it needs only a free Expo account, not Apple). Release builds embed channel `family`; `runtimeVersion.policy = "fingerprint"`. A free-signed app still dies after 7 days whatever OTA does, which is why re-signing is automated. | Most changes are JS/assets. |
| RTL | `forcesRTL: true` in native builds, **plus** the app never reads `I18nManager.isRTL`. A root `direction:'rtl'` and our own `Dir` context drive layout. Every `Text` gets an explicit `textAlign` and `writingDirection`. English islands are `<Dir value="ltr">`. | Works identically on device and web, and is immune to RN RTL regressions. |
| State | **SQLite is the source of truth** (expo-sqlite + drizzle). An append-only event log feeds deterministic projections (pure TS in `packages/engine`). **Zustand** holds ephemeral session/UI state. TanStack Query is used only for server-only parent views. `expo-sqlite/kv-store` stands in for MMKV. | Local-first (E6), testable, and with no Nitro dependency in the critical path. |
| 3D (E1b) | three.js 0.186.1 + `@react-three/fiber@9.8.2` (`/native`) + expo-gl + expo-asset, with custom kinematics + `three-mesh-bvh`. The Hebrew HUD is 2D RN only. Fallbacks: WebView-hosted three.js, then 2.5D Skia. Filament is native-only plan B. A **10-day spike with 4 gates** (§4.9). | The same scene code runs on web, which keeps Playwright useful. |
| Characters | `@rive-app/react-native@0.5.5` (Nitro) on device and `@rive-app/react-canvas@4.37.1` on web, behind one `<Character>` interface (§4.10). | E2. |
| Backend | One Cloudflare Worker `olam-api` (Hono 4.13.14) with D1 `olam-db`, R2 `olam-private` + `olam-content`, **two Cron triggers** (nightly `10 0 * * *` = 02:10/03:10 Israel; and `*/15 * * * *` for the alert queue, the Friday weekly-report `send_at` check, batch polling and expiry nudges — per 10 §15.3), the Durable Object `CoopRoom`, and the Rate Limiting binding. The parent dashboard is a Cloudflare Pages project `olam-parent` that proxies `/api/*` to the Worker through a service binding (same origin, cookie auth). | E5, H6, 10 §10–§11 (weekly report on Friday 12:00–14:00, ≥ 3 h before candle lighting). |
| Fonts | Static TTFs only (no variable fonts, 09 §15): Rubik 500/700/800/900 (display), Noto Sans Hebrew 400/500/600/700 (reading, nikud), Andika 400/700 (English). Gate candidates (Fredoka, Frank Ruhl Libre) ship only in the `development` variant until GATE-UX-2 decides. | 09 §3.6 owns the choice; B5 nikud. |
| iPad multitasking | `UIRequiresFullScreen: true`. Split View, Slide Over and resizable Stage Manager windows are **off**: a kids' game must not shrink to a 1/3-width layout nobody tested, and a child must not drag another app in. The desktop dashboard covers parents' multitasking. ⟂VERIFY on iPadOS 27 that a full-screen-required app also stays unresizable under Stage Manager. | §4.18 E-13. |
| Resilience | Every failure the walk in §4.18 names has one rule and one acceptance test: first launch offline, migration failure, corrupted DB, storage full, OTA apply timing (never mid-episode), 7-day free-signing expiry, pairing brute force, lost device, concurrent parent edits (field-level LWW), Worker/AI vendor/$-cap outages, missing R2 asset, audio interruptions, Low Power Mode, thermal throttling, orientation changes in 3D, Dynamic Type extremes with nikud, VoiceOver in Hebrew. | Playbooks (14) + REVIEW-BRIEF. |
| Auth (H + free signing) | **Pluggable identity providers:** `email_code` (primary until the paid program exists), `apple` (Sign in with Apple; turned on after the paid program), and a one-time `bootstrap` owner code. Devices hold opaque device tokens. Guardians hold short JWT access tokens plus rotating refresh tokens. The parent gate is local Face ID / PIN. | SIWA, Push and App Attest need the paid program (§18). |
| AI | All AI calls go through the Worker under a per-day $ cap: Opus 5.5 Batch for content, Sonnet 5.5 for judging and vision extraction, Haiku 5.5 for companion turns and summaries. Structured outputs everywhere. | E8. |
| Co-op | Same device first (split-zone iPad). Multi-device runs over a `CoopRoom` Durable Object WebSocket (hibernation API) with a server-authoritative shared state at 10 Hz. | Cheap, works across any network, and needs no native module. |
| CI | GitHub Actions on Linux: typecheck, lint, unit, worker tests, web export, Playwright matrix, then artifacts. On `main`: wrangler deploy, `eas update --channel preview`, and a manual promote to `family`. | The container can't reach Expo/Cloudflare (research log), so CI is the deployer. |

---

## 1. System overview

```
                         ┌──────────────────────── Parent's Mac (Claude Remote Control) ───────────────────────┐
                         │ Xcode + free personal team · `tools/mac/resign.sh` (launchd, daily check)            │
                         │ builds: olam-dev (expo-dev-client) · olam (Release, channel "family")               │
                         └───────────────┬────────────────────────────────────────────────────────────────────┘
                                         │ USB first time, then Wi-Fi (devicectl)
 ┌──────────── iPad (shared-kids) / iPhones ─────────────┐
 │ Expo SDK 58 app (RN 0.88, Hermes, New Arch)           │       EAS Update (u.expo.dev)  ◄── CI `eas update`
 │  expo-router · Zustand · SQLite(drizzle) event log    │◄────── JS/asset bundles, channel family/preview
 │  engine (pure TS) · R3F world · Skia games · Rive     │
 │  AudioService · Haptics · ContentPackLoader · Sync    │
 └───────────────┬────────────────────────────────────────┘
                 │ HTTPS  Bearer dev_… / JWT     WSS (co-op)
                 ▼
 ┌──────────────────────────── Cloudflare (Workers Paid $5) ──────────────────────────────┐
 │ Worker olam-api (Hono)                                                                 │
 │   /v1/auth/* /v1/family/* /v1/devices/* /v1/pairings/* /v1/sync /v1/content/*          │
 │   /v1/ai/* /v1/tts/* /v1/photo-jobs/* /v1/voice-clips/* /v1/fs/* /v1/reports/* /v1/admin/* /v1/coop/* │
 │ D1 olam-db (Time Travel 30d) · R2 olam-content (packs, audio) · R2 olam-private        │
 │ (photos 30d, backups) · DO CoopRoom · Rate Limiting · Cron (02:10 nightly; every 15 min:  │
 │   alert queue, Friday report send_at check, batch polling, expiry nudges)                │
 │ Pages olam-parent (parent web dashboard, /api/* → service binding → olam-api)          │
 └──────┬──────────────────────┬─────────────────────────┬──────────────────────────────┘
        ▼                      ▼                         ▼
   Anthropic API         TTS vendor (bake-off       Resend (email codes, weekly report)
   (Batch/Messages)      winner; offline baking)
```

**The agent's loop.** The agent edits code in the Linux container and runs typecheck, unit tests, a web
export and Playwright (local Chrome). It pushes a branch, and CI repeats the checks and posts
screenshots. Merging to `main` deploys the API and publishes an OTA update to `preview`. The parent
checks on their own iPhone, then a promote workflow moves it to `family`. Native changes go through
the Mac build path (§3).

---

## 2. Repository, packages and tooling

### 2.1 Monorepo layout (pnpm workspaces; default new private repo `family-learning-game`, E11)

```
family-learning-game/
├─ apps/
│  ├─ mobile/                      Expo app (iOS primary; web export for tests/prototypes)
│  │  ├─ app/                      expo-router routes (§4.1) — thin; only composition + params
│  │  ├─ src/
│  │  │  ├─ boot/                  startup sequence, migrations, config, splash gating
│  │  │  ├─ db/                    drizzle schema (local), migrations/, repositories/
│  │  │  ├─ state/                 zustand stores (session, ui, audio prefs, coop)
│  │  │  ├─ services/              audio/, haptics/, sync/, content/, assets/, ai/, auth/, flags/,
│  │  │  │                         motion/, notifications/, telemetry/, devicecmd/
│  │  │  ├─ world/                 3D world: pure three.js + game logic (no RN imports) (§4.9)
│  │  │  ├─ platform/              platform shims: gl canvas (native/web/webview), rive (.native/.web/
│  │  │  │                         .placeholder), secure storage, file hashing
│  │  │  ├─ games/<gameId>/        2D mini-games: logic.ts (pure), Scene.tsx (Skia), config.ts
│  │  │  ├─ features/              screens' feature modules (picker, hub, collection, shop, parent/*)
│  │  │  ├─ i18n/                  runtime: message lookup, gender resolution, voice key mapping
│  │  │  └─ test/                  test ids, deterministic clock/RNG hooks (`window.__olam`)
│  │  ├─ assets/                   bundled: fonts/, rive/, audio/core/, models/ (glb), images/
│  │  ├─ e2e/                      Playwright specs, fixtures, baselines (__screenshots__/)
│  │  ├─ app.config.ts  eas.json  metro.config.js  babel.config.js  serve.json
│  ├─ parent-web/                  Parent dashboard (Vite + React + react-router), deployed to Pages
│  │  └─ functions/api/[[path]].ts Pages Function → service binding to olam-api
├─ workers/
│  └─ api/                         Cloudflare Worker (Hono): src/routes/*, src/db/schema.ts (D1),
│                                  src/do/CoopRoom.ts, src/cron/*, src/ai/*, migrations/*.sql,
│                                  wrangler.jsonc, test/ (vitest-pool-workers)
├─ packages/
│  ├─ engine/                      adaptive engine (Elo, Leitner/SRS, placement, success bands,
│  │                               projections/reducers for events → state). Pure TS, no I/O.
│  ├─ content/                     pack JSON schemas (zod), validators (nikud, numerals, gender),
│  │                               item templates, error-catalog distractors, pack builder
│  ├─ shared/                      wire types: API request/response zod schemas, event types,
│  │                               ids (uuidv7), HLC clock, error codes, RBAC permission table
│  ├─ i18n/                        message catalogs he/en (ICU), numerals lookup (B4), lints
│  ├─ ui/                          design system: tokens, RTL primitives (Dir, HText, Row),
│  │                               buttons, cards, gates; works on native + web
│  └─ api-client/                  typed fetch client used by mobile + parent-web
├─ tools/
│  ├─ content-gen/                 batch submit/collect, QA judge, nikud pass, pack publish
│  ├─ tts-bake/                    bake-off + bake pipeline → R2 (E9)
│  ├─ assets/                      image pipeline (E10), glb pipeline (gltf-transform), budget linter
│  ├─ mac/                         resign.sh, launchd plist, devices.json.example, doctor.sh
│  ├─ ops/                         bootstrap-owner, backup-export, restore, purge, cost-report
│  └─ screens/                     contact-sheet generator from Playwright output (PR review)
├─ docs/ (spec/, decisions/ ADRs, research-log/)
├─ .github/workflows/              ci.yml, deploy-api.yml, ota.yml, promote.yml, parent-web.yml,
│                                  native-eas.yml (phase 2), nightly-visual.yml
├─ CLAUDE.md                       agent conventions (RTL rules, gender rules, no 3rd-party analytics…)
├─ package.json  pnpm-workspace.yaml  .npmrc  tsconfig.base.json  eslint.config.mjs
├─ .prettierrc  lefthook.yml  renovate.json  .nvmrc (22)  .editorconfig
```

Each folder has one job:
- `app/` holds no business logic. A route file under 40 lines is the norm.
- `packages/engine` and `packages/content` are pure and run under Node, Workers and Hermes. They are
  the main unit-test surface.
- `packages/shared` is the single source of wire contracts. The Worker validates with the same zod
  schemas the client sends.
- `src/world` never imports `react-native`. `src/platform` is the only place with platform branches
  (`.native.tsx` / `.web.tsx`).

### 2.2 Root files

```yaml
# pnpm-workspace.yaml
packages: ["apps/*", "workers/*", "packages/*", "tools/*"]
```
```ini
# .npmrc — RN autolinking and Metro are most reliable with a hoisted layout
node-linker=hoisted
auto-install-peers=false
strict-peer-dependencies=false
save-exact=true
```
```jsonc
// package.json (root)
{
  "name": "family-learning-game", "private": true,
  "packageManager": "pnpm@10.34.9",
  "engines": { "node": ">=22.18 <25" },
  "scripts": {
    "typecheck": "pnpm -r --parallel typecheck",
    "lint": "eslint . --max-warnings=0",
    "format": "prettier --write .",
    "test": "pnpm -r --parallel test",
    "web:export": "pnpm --filter mobile web:export",
    "e2e": "pnpm --filter mobile e2e",
    "api:dev": "pnpm --filter api dev",
    "lint:i18n": "pnpm --filter @olam/i18n lint",
    "lint:assets": "pnpm --filter @olam/tools-assets budget"
  },
  "devDependencies": {
    "typescript": "6.0.3", "eslint": "10.13.0", "typescript-eslint": "8.72.0",
    "eslint-config-expo": "58.0.4", "prettier": "3.9.10", "lefthook": "2.2.2", "tsx": "4.23.15"
  }
}
```

### 2.3 Pinned package list

Rules:
- Everything uses an exact version (`save-exact`).
- Expo-managed packages are installed with `npx expo install` and must equal the SDK 58 map below.
- Others use the **newest release that is at least 7 days old**, enforced by Renovate
  `minimumReleaseAge: "7 days"` with one grouped PR per week.
- Expo SDK upgrades are manual: run `npx expo install expo@^59 --fix` and `npx expo-doctor`, then
  re-run the full visual and device checklist.

**apps/mobile — Expo SDK 58 set (from `expo@58.0.22` bundledNativeModules)**

| Package | Version | Notes |
|---|---|---|
| expo | 58.0.22 | |
| react / react-dom | 19.3.0 | |
| react-native | 0.88.1 | Hermes, New Architecture (mandatory since SDK 55) |
| react-native-web | 0.21.4 | |
| expo-router | 58.0.29 | typed routes on |
| expo-dev-client | 58.0.18 | dev variant only |
| expo-updates | 58.0.20 | |
| react-native-reanimated / react-native-worklets | 4.7.1 / 0.13.0 | babel plugin is auto-added by babel-preset-expo |
| react-native-gesture-handler | 3.3.0 | |
| @shopify/react-native-skia | 2.14.0 | `setup-skia-web` copies canvaskit.wasm |
| react-native-screens / react-native-safe-area-context | 4.30.1 / 5.10.1 | |
| react-native-svg | 15.16.0 | |
| lottie-react-native | 7.5.0 | one-shots only |
| @shopify/flash-list | 2.3.2 | |
| expo-audio / expo-haptics | 58.0.11 / 58.0.7 | |
| expo-sqlite | 58.0.15 | also `expo-sqlite/kv-store` |
| expo-file-system / expo-asset / expo-font | 58.0.13 / 58.0.19 / 58.0.11 | |
| expo-image / expo-image-picker / expo-image-manipulator / expo-camera | 58.0.18 / 58.0.10 / 58.0.16 / 58.0.14 | camera is also the QR scanner |
| expo-localization | 58.0.5 | plugin `{supportsRTL:true, forcesRTL:true, supportedLocales:["he","en"]}` |
| expo-local-authentication / expo-secure-store / expo-crypto | 58.0.2 / 58.0.2 / 58.0.7 | |
| expo-notifications | 58.1.7 | **local only** in phase 1 (no push entitlement on a free team) |
| expo-gl / expo-sensors | 58.0.4 / 58.0.4 | 3D and motion (E3: tilt, shake, "device in hand" check before jump moments) |
| expo-device / expo-battery | SDK map | `modelId → ptPerMm` (A7, 09 §4.1; unknown model → 6.42 pt/mm + telemetry) / Low Power Mode (§4.7) |
| *(deferred)* `olam-gamecontroller` Expo Module | — | **Not in the vertical-slice allow-list.** iOS GameController (DualSense/MFi): stick = move, X = jump, haptics map → controller rumble. Allowed after C4, never required by any mechanic (00 E3, 09 PC-2). ADR when picked up. |
| *(tiny)* `olam-thermal` Expo Module | — | `ProcessInfo.thermalState` + `os_proc_available_memory()` (§4.7). ~40 lines Swift; written at C4 perf pass |
| *(local)* `modules/vision-privacy` Expo Module | — | Swift, ~80 lines: `VNDetectFaceRectanglesRequest` (rects grown 20 % → Skia blur σ=25 + pixelate) and `VNRecognizeTextRequest` for the "non-family names" list when Hebrew is supported — the **on-device face/name blur that runs before any photo upload** (12 §3.3, §8.3, §9.4). Required by SCR-14; in the dev-build native allow-list from the sprint that lands it |
| *(optional)* `react-native-document-scanner-plugin` | latest ≥ 7 days old at adoption | VisionKit document camera + perspective fix (12 §3.3). Until adopted: `expo-camera` + the Skia crop screen (CMP-44); ADR when picked up |
| expo-splash-screen / expo-screen-orientation / expo-keep-awake / expo-network / expo-constants / expo-application / expo-device | SDK map | |
| expo-apple-authentication | 58.0.2 | installed but enabled by flag in phase 2 only |
| react-native-webview | 14.0.1 | 3D fallback 1 and the parent's in-app web views |

**apps/mobile — third-party**

| Package | Version | Notes |
|---|---|---|
| @rive-app/react-native | 0.5.5 | Nitro; peer nitro `>=0.35.10 <0.37`. Bump to 0.5.10+ after an on-device check. |
| react-native-nitro-modules | 0.36.5 | RN ≥0.87 needs ≥0.36.2 |
| @rive-app/react-canvas | 4.37.1 | web split only |
| three / @types/three | 0.186.1 / 0.186.0 | |
| @react-three/fiber | 9.8.2 | use `@react-three/fiber/native` on device. **Not** `@react-three/native` (pre-alpha). |
| @react-three/drei | 10.7.9 | native-safe subset only (no `<Text>`, no Draco/KTX2 loaders on native) |
| three-mesh-bvh | 0.9.16 | raycasts for kinematics and tap-to-move |
| drizzle-orm | 0.45.4 | `drizzle-orm/expo-sqlite` |
| zustand | 5.0.16 | |
| @tanstack/react-query | 5.104.5 | parent server views only |
| intl-messageformat | 12.1.4 | ICU; add `@formatjs/intl-pluralrules` only if the Hermes `he` plural check fails |
| zod | 4.6.7 | |
| uuidv7 | 1.2.1 | event and entity ids |
| @noble/hashes | 2.4.0 | sha256 of files, scrypt for PINs |
| @noble/curves | 2.4.0 | Ed25519 verification of pack manifests |
| react-native-audio-api | 0.13.6 | **optional**, adopted only if the expo-audio latency gate fails (§4.5) |
| @sentry/react-native | 8.28.0 | **optional**, dev build only, behind flag (§13) |
| react-native-vision-camera | 5.2.3 | **later phase** (camera pose, G6) |

**Dev/test (mobile):** jest 30.5.2, jest-expo 58.0.13, @testing-library/react-native 14.1.0,
**reassure 1.6.1** (package name `reassure`, not `@callstack/reassure`; render-count/duration gate per
UI PR), @playwright/test 1.64.1, serve 14.2.6, drizzle-kit 0.31.11, fast-check 4.10.2,
**@stryker-mutator/core + @stryker-mutator/vitest-runner 10.1.1**, **pixelmatch** (Maestro
double-screenshot layout-shift diff, 14 PQ-T-06), eas-cli 25.3.0 (invoked via `npx eas-cli@25.3.0`).
The Maestro CLI is installed on the Mac by `tools/mac/doctor.sh`. Note: `react-native-worklets`
0.13.1 was published 2026-11-01 (< 7 days old), so 0.13.0 is the correct pin today; Renovate bumps it
after the 7-day window — do not hand-edit.

**workers/api:** hono 4.13.14, @hono/zod-validator 0.9.1, zod 4.6.7, drizzle-orm 0.45.4
(`drizzle-orm/d1`), jose 6.2.13, @anthropic-ai/sdk 1.3.0, aws4fetch 1.0.20 (R2 presigned URLs),
uuidv7 1.2.1, @noble/curves 2.4.0 (manifest signing). Dev: wrangler 4.152.0, @cloudflare/workers-types 5.20261025.1, vitest 4.1.12,
@cloudflare/vitest-pool-workers 0.23.0, drizzle-kit 0.31.11.

**apps/parent-web:** react 19.3.0, react-dom 19.3.0, react-router 7.x (latest ≥7 days at scaffold),
@tanstack/react-query 5.104.5, vite 7.x (latest ≥7 days at scaffold), @olam/ui (web build via
react-native-web), @olam/api-client.

**packages/engine, content, shared, i18n:** zero runtime dependencies besides zod and uuidv7. Tests
use vitest 4.1.12 + fast-check 4.10.2.

**tools:** @gltf-transform/cli 4.5.1, sharp 0.35.5, @anthropic-ai/sdk 1.3.0, tsx 4.23.15. Headless
Blender is installed in CI via apt; spike gate #4 decides whether it's used.

### 2.4 TypeScript, lint and format

```jsonc
// tsconfig.base.json
{
  "compilerOptions": {
    "target": "ES2022", "lib": ["ES2023"], "module": "ESNext", "moduleResolution": "Bundler",
    "strict": true, "noUncheckedIndexedAccess": true, "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true, "noFallthroughCasesInSwitch": true, "verbatimModuleSyntax": true,
    "isolatedModules": true, "skipLibCheck": true, "resolveJsonModule": true,
    "forceConsistentCasingInFileNames": true, "useUnknownInCatchVariables": true
  }
}
```
- `apps/mobile/tsconfig.json` extends `expo/tsconfig.base` and then the base above, adding `paths`
  `{"@/*": ["./src/*"]}`.
- `workers/api` adds `"types": ["@cloudflare/workers-types/2023-07-01"]` and runs
  `wrangler types` to generate `worker-configuration.d.ts`.
- **ESLint 10 flat config:** `eslint-config-expo/flat` for mobile, `typescript-eslint`
  strict-type-checked for packages and workers, and custom local rules in `tools/eslint-rules`:
  - `olam/no-i18nmanager`: bans `I18nManager.isRTL` and `I18nManager.forceRTL` outside `src/boot`.
  - `olam/no-raw-text`: no `<Text>` with a string literal in `app/` or `src/features`. Use `t()` and
    `<HText>`.
  - `olam/no-physical-direction`: bans `marginLeft/Right`, `paddingLeft/Right`, `left/right`,
    `textAlign:'left'|'right'` and `flexDirection:'row-reverse'` in style objects. Use `Start/End`
    and the `Row` primitive.
  - `olam/no-live-tts`: no imports of `expo-speech` in child-facing code.
  - `olam/no-third-party-analytics`: a deny list of SDK names.
  - `olam/perf-bans`, from `performance_reliability_playbook.md` §11D (binding; the standalone
    playbook spec is `14-performance-quality-playbook.md`, which owns the full rule text — this spec
    only wires the rules into code, lint and CI):
    - `InteractionManager` and `Touchable*` (removed in RN 0.88)
    - `I18nManager.forceRTL` (RTL is set natively, never toggled)
    - reading `.value` of shared values in render
    - `new THREE.*` inside `useFrame`
    - `createAudioPlayer` inside event handlers
    - `<Image>`/`expo-image` without width/height or aspectRatio
    - `BlurView` or a blurred shadow on an animated layer (09 §3.9)
  - `olam/no-raw-design-values` (09 §3): no literal color, duration, radius or size in component
    code under `packages/ui` and `src/features`; tokens only.
  - `olam/no-raw-pressable` (09 UX-AC-10): no raw `Pressable`/`TouchableOpacity` in child screens.
  - `olam/engine-purity` (quality playbook §3): `Math.random`, `Date.now`, `new Date()`, `fetch`,
    `setTimeout` and any `expo-*`/`react-native` import are banned in `packages/engine` and
    `packages/content` (`no-restricted-globals` + `no-restricted-imports`).
  - `olam/no-skipped-tests` (quality playbook §4): `.only`/`.skip`/`xit` fail lint; deleting a test
    file needs the PR label `tests-removed` with a reason (checked by `tools/ci/test-delta.mjs`).
  - From 14 §3: `olam/no-layout-prop-animation` (no `width/height/margin/padding/top/left` in
    `withTiming/withSpring`), `olam/no-blur-on-animated`, `olam/no-sync-io` (no `*Sync(` from
    expo-sqlite/expo-file-system outside `src/boot`), `olam/no-nondeterminism` (= `engine-purity`
    above, extended to `src/games/*/logic.ts` and `src/world/sim`), `olam/no-unsafe-cast` (no `as`
    on a `JSON.parse`/network/DB result — zod only), plus `eslint-plugin-no-only-tests`.
  - `@typescript-eslint/switch-exhaustiveness-check`, `no-floating-promises`,
    `react-hooks/exhaustive-deps` as **errors**.
  - `react-hooks` rules plus the React Compiler lint; zero compiler bail-outs allowed in
    `src/games/**` and `src/world/**` (checked with `react-compiler-healthcheck` in CI).
- **Prettier:** 2-space indent, single quotes, 100 cols, trailing commas `all`.
- **Lefthook:** pre-commit runs prettier on staged files plus `eslint --cache` on staged files.
  Pre-push runs `pnpm typecheck`.
- **React Compiler:** on (`experiments.reactCompiler: true`). It does automatic memoization, so avoid
  hand-written `useMemo` unless profiling shows a need.

---

## 3. Install, signing, distribution and OTA (E1a)

### 3.1 Decision tree

```
Does the family have a paid Apple Developer Program ($99/yr)?
├─ NO  (current state, 2026-11-02)
│   ├─ Concept stage (no native features needed yet)?
│   │     → PATH 0: Expo **web export** → Claude artifact / Pages preview URL → open in iPad Safari.
│   │       (No install. No haptics, no Rive native, no Face ID; Skia/R3F/Reanimated render on web.)
│   └─ Need real device features (haptics, audio session, Rive, Face ID, camera, perf)?
│         Has a Mac? ── YES (current) → PATH 1: dev build + release build over cable/Wi-Fi from the Mac,
│                                       free personal team, auto re-sign ≤ every 6 days.
│                       NO            → only PATH 0, or buy the paid program → PATH 2.
└─ YES
    ├─ Has a Mac? YES → PATH 1 with paid team (1-year profiles; no weekly re-sign) — or —
    └─ any        → PATH 2: EAS Build (cloud, no Mac) internal distribution (ad hoc, install by link)
                    + optional TestFlight for parents; push, SIWA, App Attest become available.
Expo Go: NOT a path for SDK 58 (App Store Expo Go ≤ SDK 54; `eas go`/TestFlight needs paid program).
         Re-verify at build time; if a compatible Expo Go appears, it is still only a toy — Rive/Nitro
         and our config plugins (forcesRTL, permissions strings) don't run there.
```

**Capabilities by path** (this drives feature flags; see `capabilities.ts` in §4.12):

| Capability | Path 0 web | Path 1 free team | Path 1/2 paid |
|---|---|---|---|
| Skia, Reanimated, R3F/expo-gl | ✅ (web impls) | ✅ | ✅ |
| Rive Nitro | ❌ (react-canvas) | ✅ | ✅ |
| Haptics | ❌ | ✅ (iPhone only, no iPad Taptic) | ✅ |
| expo-audio session control | partial | ✅ | ✅ |
| SQLite | ✅ (alpha, needs COOP/COEP) | ✅ | ✅ |
| Face ID (expo-local-authentication) | ❌ (PIN only) | ✅ (needs `NSFaceIDUsageDescription` only) | ✅ |
| Keychain (expo-secure-store) | ❌ (memory/IndexedDB) | ✅ | ✅ |
| Camera / QR / photo | ✅ (getUserMedia) | ✅ | ✅ |
| Local notifications | ❌ | ✅ | ✅ |
| Remote push (APNs / Expo Push) | ❌ | ❌ (no `aps-environment` entitlement) | ✅ |
| Sign in with Apple (native) | ❌ | ❌ | ✅ |
| App Attest | ❌ | ❌ | ✅ |
| EAS Update OTA | n/a | ✅ | ✅ |
| Lifetime of install | n/a | **7 days** per signing | 1 year (ad hoc) / 90 days (TestFlight) |

### 3.2 PATH 0 — web prototypes in iPad Safari

- `pnpm --filter mobile web:export` runs `expo export -p web`, writing a static site to
  `apps/mobile/dist`. Routes are rendered with `web.output: "static"` and `expo-router`.
- Delivery has two options:
  - **(a)** Publish as a Claude artifact. The agent passes the exported folder through the Artifact
    tool's `files`, entry `index.html`.
  - **(b)** `wrangler pages deploy apps/mobile/dist --project-name olam-proto --branch <branch>` from
    CI, giving the preview URL `https://<branch>.olam-proto.pages.dev`.
- SQLite on web needs these headers. With Pages, they go in an `_headers` file:
  `Cross-Origin-Opener-Policy: same-origin` and `Cross-Origin-Embedder-Policy: credentialless`.
  If a host can't send headers, prototypes use the in-memory repository adapter
  (`EXPO_PUBLIC_DB=memory`).
- The parent opens the URL in iPad Safari and may use *Share → Add to Home Screen*. This is for
  concept and taste approval only. Not for kids' daily use.

### 3.3 PATH 1 — Mac + free personal team (the primary path now)

**One-time Mac setup (parent, about 45 min; the agent then drives everything over Remote Control):**
1. Install **Xcode** (App Store, latest stable for iOS 27 SDK). Open it once, accept the license, and
   let it install components. Run `xcode-select -p` to check.
2. Xcode → Settings → **Accounts** → "+" → Apple ID. This creates a *Personal Team* (free). The agent
   reads the Team ID with
   `security find-certificate -c "Apple Development" -p | openssl x509 -noout -subject`, or from
   Xcode, and stores it in `tools/mac/devices.json` → `teamId`.
3. Install Homebrew. Then the agent runs `tools/mac/doctor.sh`, which installs or checks: Node 22
   (fnm), corepack pnpm 10.34.9, CocoaPods (if `expo prebuild` still emits a Podfile on SDK 58;
   *verify*), watchman (optional), and `ios-deploy` not needed (we use `devicectl`).
4. Claude desktop app → Settings → Claude Code → **Remote Control** → add the folder
   `~/Code/family-learning-game`. The cloud session then calls `request_computer_folder`, and the parent
   approves each request in the app.
5. Clone the repo into that folder (the agent does it once attached).

**One-time per device (iPad/iPhone; parent, about 5 min each):**
1. Connect by USB-C/Lightning to the Mac and unlock the device. Tap **Trust This Computer** and enter
   the passcode.
2. On iOS 16+: Settings → Privacy & Security → **Developer Mode** → On → Restart → confirm. The toggle
   appears only after the device has been seen by Xcode or devicectl. The agent triggers this by
   running `xcrun devicectl list devices`.
3. In Xcode → Window → Devices and Simulators, select the device and tick **Connect via network**, so
   later installs work over Wi-Fi on the same LAN.
4. After the first install, the app won't open until the developer is trusted: Settings → General →
   **VPN & Device Management** → *Developer App* "Apple Development: <apple-id>" → Trust. This is
   needed once per team per device.
5. The agent records the device in `tools/mac/devices.json`:
   `{ "name":"iPad-Family", "udid":"…", "role":"kids", "installs":["olam"] }`. The parent iPhone gets
   `"installs":["olam","olam-dev"]`.

**Build variants (app.config.ts, `APP_VARIANT`):**

| Variant | Bundle ID | Contents | Installed on | Free-team app slots used |
|---|---|---|---|---|
| `development` | `com.<familyslug>.olam.dev` | expo-dev-client, Debug, dev menu, Metro or EAS Update branches | parent iPhone (+ optionally iPad for debugging) | 1 |
| `family` | `com.<familyslug>.olam` | Release, expo-updates channel `family` (or `preview` on the parent iPhone), no dev menu | all kid devices + parent phones | 1 |

- Keep to **these two bundle IDs forever**. Free teams can only register a few App IDs per week and
  allow about 3 apps per device. `<familyslug>` must be globally unique, e.g. the family surname plus
  birth year. The parent confirms it once.
- **Never change the signing Apple ID/team** on an installed device without first running a full sync
  (§6.5). A different team can force delete-and-reinstall, which wipes local SQLite. The server copy
  makes this recoverable (§6.6 "restore device"), but it is still a migration step on the checklist.

**Commands the agent runs on the Mac (via Remote Control):**
```bash
# first time per variant (generates ios/ with our config plugins, then builds + installs)
APP_VARIANT=family npx expo prebuild -p ios --clean
APP_VARIANT=family npx expo run:ios --configuration Release --device <UDID>   # signs with personal team
APP_VARIANT=development npx expo run:ios --device <UDID>                        # dev client build

# fast install of an already-built app to other devices (USB or network)
xcrun devicectl list devices
xcrun devicectl device install app --device <UDID> ios/build/Build/Products/Release-iphoneos/Olam.app
xcrun devicectl device info details --device <UDID>   # transportType, developerModeStatus
```
Signing settings are written by a config plugin (`plugins/withSigning.ts`):
- `DEVELOPMENT_TEAM=$OLAM_TEAM_ID`, `CODE_SIGN_STYLE=Automatic`.
- Entitlements are **stripped** of `aps-environment`, `com.apple.developer.applesignin` and
  `com.apple.developer.devicecheck.appattest-environment` when `OLAM_SIGNING=free`.
- xcodebuild runs with `-allowProvisioningUpdates -allowProvisioningDeviceRegistration`.

**Weekly re-sign routine (automated, ≤ 6-day cadence):**
- `tools/mac/resign.sh` is installed as a LaunchAgent (`~/Library/LaunchAgents/il.olam.resign.plist`)
  that runs **daily at 18:40** plus at login. Each run:
  1. Reads `~/.olam/resign-state.json` → `{udid: {variant, installedAt, buildSha}}`.
  2. For each device with `now - installedAt ≥ 5 days`, or never installed, it rebuilds only if the
     cached `.app` is older than 5 days. A rebuild is needed to get a fresh 7-day profile:
     `git -C repo fetch && git checkout <latest release tag "family-*">`, then
     `pnpm i --frozen-lockfile`, then `npx expo run:ios --configuration Release --no-install
     --no-bundler`, or `xcodebuild … -allowProvisioningUpdates archive/build`.
  3. Installs over Wi-Fi with `devicectl` (falling back to USB if attached). Then verifies with
     `xcrun devicectl device info apps --device <UDID> | grep com.<slug>.olam`.
  4. On success it updates the state. On failure (device off or asleep, not on LAN, locked; *verify*
     whether locked devices accept installs) it retries the next day. **At ≥ 6 days without success**,
     it posts `POST /v1/ops/notify` (guardian email: "צריך לחבר את האייפד למק לחידוש האפליקציה") and
     shows a macOS notification.
- The app also knows its own expiry. `extra.buildSignedAt` is injected at build time. The parent area
  and the device list show "תוקף ההתקנה: עוד N ימים". At ≤ 1 day, the parent gate screen shows a
  gentle banner. Kids never see expiry.
- When an app has expired on a device, nothing is lost. Re-install, or the next routine run, restores
  it, and local data stays because the bundle ID and team are the same. ⟂VERIFY (01 DD-37, 02 UC-21,
  F2 #22) in the **first re-sign cycle, on one device**: a `devicectl device install app` over the same
  bundle id keeps the SQLite container (the routine's smoke check: `device_self.paired_at` unchanged,
  `event` row count ≥ before, `resign-state.json` records `containerPreserved: true`). Until that is
  verified, `resign.sh` runs with `--pre-install-sync` (default on): before installing it queues a
  `resync` device command through `POST /v1/ops/notify {kind:'pre_install_sync', deviceId}`, waits up
  to 10 min for `devices.last_sync_at` to advance (the app flushes on its next foreground), then
  installs and logs the outcome; the parent area's device card shows "סנכרון: N אירועים ממתינים"
  (10 FAM-04) so the parent can press "סנכרון עכשיו" first. If the container is ever lost, the
  server copy restores everything except `local_only` data (§6.6).
- **Manual trigger (13 §3.2 compatibility):** the parent can also say "חתימה שבועית" in chat, or run
  `tools/mac/resign.sh --all` themselves. The script is the same; the LaunchAgent is only the
  automatic caller. (13 calls the script `tools/device/resign.sh`; the canonical path is
  `tools/mac/resign.sh`, see `docs/review/fable-issues-F6.md`.)

**7-day expiry walk (what actually happens, E-06 in §4.18):**

| Moment | Device | Kid sees | Data |
|---|---|---|---|
| Day 5 | routine rebuilds + reinstalls over Wi-Fi; app restarts cleanly on next launch | nothing (install happens while the app is closed; if it is open, iOS quits it and the next launch resumes at the picker) | untouched; an unsynced event queue is flushed on next launch |
| Day 6, install failed | `POST /v1/ops/notify {kind:'resign_overdue'}` → email to O/C; macOS notification; parent-gate banner "תוקף ההתקנה: עוד יום" | nothing | untouched |
| Day 7+, expired | iOS refuses to launch: system alert "Unable to Verify App" / "לא ניתן לאמת את האפליקציה". This is iOS UI; we cannot restyle it | the system alert. The parent checklist says what it means ("צריך לחבר למק"); a child who taps it just sees the home screen again | **SQLite, Keychain and packs stay on disk** as long as the app is not deleted. Up to 7 days of unsynced events wait locally (sync runs every 5 min while online, so normally ≤ 5 min of play is unsynced) |
| Reinstall (same bundle ID + team) | app launches, `app.start` event, sync flush, `app.resigned` telemetry | picker | preserved; verified by the routine's smoke check (`device_self.paired_at` unchanged, event count ≥ before) |
| Reinstall after delete, or team change | fresh install → pair as a new device (§5.3 S3) → restore from family cloud (§6.6) | setup screen, then picker | server copy restores everything except `local_only` events (mood check-ins, C8) and the per-device bedtime overrides, which the parent re-enters; the parent is told this in the restore screen |

- Rule: **the app is never deleted to "fix" an expiry**; the checklist and the parent-gate banner both
  say so (13 §3.2 agrees).

**OTA in PATH 1.** Release builds embed the following. JS/asset changes then arrive without the Mac;
only native changes or the 7-day re-sign need it.
```ts
updates: { url: `https://u.expo.dev/${EAS_PROJECT_ID}`, checkAutomatically: "ON_LOAD",
           fallbackToCacheTimeout: 0, requestHeaders: { "expo-channel-name": channel } },
runtimeVersion: { policy: "fingerprint" }
```
- `family` is the default channel. A build made with `OLAM_CHANNEL=preview` (the parent iPhone) gets
  preview updates first.
- The dev client loads either Metro (`npx expo start --dev-client`, run on the **Mac** via Remote
  Control on the LAN; the container can't host a tunnel) or a published EAS Update branch via its
  extensions panel or `qr.expo.dev`.
- **OTA apply timing (binding, E-05 in §4.18).** `checkAutomatically: 'ON_LOAD'` +
  `fallbackToCacheTimeout: 0` means the embedded/cached bundle starts immediately and a newer update
  downloads in the background. It is **applied only at these moments**, by `src/services/updates/
  UpdateGate.ts`:
  1. The next cold start (iOS normal case).
  2. On the **picker** (`/picker` or `/(setup)`), when no child session is open, the device has been
     idle ≥ 60 s, and no co-op room is active → silent `Updates.reloadAsync()` (the picker re-renders
     in < 2 s; a kid who taps during the reload simply lands on the picker again). This covers the
     shared iPad that is never cold-started for weeks.
  2b. **Episode boundary / profile switch** (GAP-26): when `session.ended` has been written and the
     app is about to render the picker (wind-down done, "החלפת שחקן", inactivity return), and an
     update is already downloaded → `reloadAsync()` **before** the picker mounts (the child sees the
     picker ~1 s later; the resume doc was just saved, so nothing is lost). Not after `rest.tsx`
     transitions (a bedtime lock must stay visible).
  2c. **Foreground after a long background** (GAP-26): `AppState` → `active` after ≥ 30 min in the
     background, no child session open (`useSession().active === false`) and an update downloaded →
     `reloadAsync()` before any route renders. A backgrounded mid-episode app (E-16, ≤ 10 min) is
     *not* reloaded; it resumes to "לגעת כדי להמשיך".
  3. From the parent area, "עדכון זמין — להפעיל מחדש?" (manual).
  - `UpdateGate.ts` is the **only** caller of `Updates.reloadAsync()` (lint, 14 PQ-X-20); every
    moment above logs `app.update{where:'cold'|'picker'|'boundary'|'foreground'|'parent'}`.
  - **Never** while a `(child)` route, a mini-game, a chest reveal, the wind-down, a movement activity
    or a co-op room is mounted. `UpdateGate` checks `useSession().active === false` and
    `useCoop().roomId === null` before every reload; a test mounts a child route and asserts that
    `reloadAsync` is not called even when an update is ready.
  - On `(child)` mount the gate also calls `Updates.checkForUpdateAsync()` only if the last check is
    > 6 h old; the download itself is deferred until the episode ends (no bandwidth competition with
    pack downloads during play).
  - Every `app.start` event carries `updateId`; an S0 bug names the update group to roll back (§11.3).

### 3.4 PATH 2 — paid program (deferred; ready-to-flip)

- `eas.json` profiles `development`, `preview` and `production` exist from day 1 (§11.1). They're
  unused until `OLAM_SIGNING=paid`.
- Steps when the parent buys the program:
  1. The parent enrolls ($99/yr).
  2. The parent creates an App Store Connect API key (Issuer ID, Key ID, .p8) and stores it in EAS
     (`eas credentials`) or as GitHub secrets `ASC_KEY_ID`, `ASC_ISSUER_ID`, `ASC_KEY_P8`.
  3. Run `eas device:create`, which gives a URL/QR for each device to open in Safari and install the
     profile. That registers the UDIDs.
  4. Run `eas build -p ios --profile preview` (internal distribution, ad hoc) and install by link.
  5. Turn on flags `auth.apple`, `push.remote` and `attest.appAttest`.
  6. Run the **team-migration checklist**: sync every device, check the server copy, install the
     paid-team build (the bundle ID can stay the same), and if iOS refuses the upgrade, delete and
     reinstall, then run "restore from family cloud" (§6.6).
- Ad hoc profiles last 1 year, so the re-sign routine becomes a yearly reminder. TestFlight builds
  expire after 90 days; `native-eas.yml` rebuilds monthly if TestFlight is used.

---

## 4. App architecture (apps/mobile)

### 4.1 Navigation map (expo-router, typed routes)

```
app/
├─ _layout.tsx                 Providers: ErrorBoundary › DbProvider › ConfigProvider › DirProvider(rtl)
│                              › AudioProvider › QueryClientProvider › GestureHandlerRootView; splash gate
├─ index.tsx                   Boot router: no device token → /(setup)/welcome; device mode
│                              single-child → /(child)/[childId]/hub; else → /picker
├─ (setup)/                    First-run for a new device (no child UI here)
│  ├─ welcome.tsx              "הגדרת מכשיר חדש": [אני הורה – הקמת משפחה] [צירוף מכשיר למשפחה]
│  ├─ sign-in.tsx              email code (phase 1) | Sign in with Apple (phase 2) | bootstrap code
│  ├─ family.tsx               create family (owner only; bootstrap path)
│  ├─ children/new.tsx         add child: name, nickname, birthdate, gender (f/m), grade/gan, avatar
│  ├─ parent-pin.tsx           set 6-digit PIN + enable Face ID
│  ├─ pair.tsx                 new device: scan QR / type 6-digit code
│  ├─ device-setup.tsx         device name, mode, allowed children (guardian-confirmed)
│  └─ done.tsx                 → placement expedition per child
├─ picker.tsx                  SCR-01 Profile picker (avatars, optional picture-PIN, K3 → parent-present gate)
├─ safe-room.tsx               SCR-16 החדר המוגן — reachable from the picker and pause WITHOUT a gate (B8);
│                              works offline and in the dark; no child session required
├─ rest.tsx                    SCR-20 מנוחה (Shabbat/holiday/bedtime/daily cap/memorial day) — rendered by
│                              `RestGate` whenever the calendar or device_settings say so; only exits: safe-room, parent gate
├─ (child)/[childId]/          ChildSessionProvider (opens a play session, caps, wind-down timer, FTUE state)
│  ├─ _layout.tsx              Stack; HUD layer; pause sheet (SCR-18) is a modal in this layout, not a route;
│  │                           `gestureEnabled:false`; long-press → picker; parent gate to leave the app
│  ├─ hub.tsx                  SCR-03 3D/2.5D hub (rescued friends live here); SCR-02 FTUE = hub with
│  │                           `child_resume.ftueStep < done` (same route, scripted overlay)
│  ├─ map.tsx                  SCR-04 map / path (zones, nodes, review nodes, secret nodes)
│  ├─ world/[zoneId].tsx       SCR-05 explorable zone (R3F Canvas + 2D HUD)
│  ├─ level/[levelId].tsx      SCR-05 gimmick level (G3); K3 diorama is a level with `controls:'diorama'`
│  ├─ game/[gameId].tsx        SCR-06 2D learning mini-game (Skia) launched from world triggers / map nodes
│  ├─ english/[islandId].tsx   SCR-06h English island — wraps children in <Dir value="ltr">, EN jingle on enter
│  ├─ results.tsx              SCR-07 results (CHOREO-06) → chest → receipt; params: episodeId
│  ├─ wind-down.tsx            SCR-07 episode end ritual (WindDownCard) → picker
│  ├─ collection.tsx           SCR-12 display room with silhouettes (G2); K3 → StickerBook variant
│  ├─ shop.tsx                 SCR-08 cosmetic shop (K1 full, K2 simple; route guarded off for K3 per D5)
│  ├─ avatar.tsx               SCR-09 avatar / hero customizer
│  ├─ base.tsx                 SCR-10 "הבסיס שלי" (BuildGrid)
│  ├─ pet.tsx                  SCR-11 pet (K1/K2 only)
│  ├─ chest/[chestId].tsx      chest reveal (contents fixed at earn time, D2)
│  ├─ move/[activityId].tsx    SCR-19 movement activity (SafetyCard → MovementCoach → cooldown)
│  ├─ playroom.tsx             SCR-17 K3 lap-mode playroom (calmMeadow; the only K3 "hub")
│  ├─ car.tsx                  SCR-21 "מצב רכב" (P2, flag `carMode`; 01 §15.1 audio-only option): playlist +
│  │                           one "עצור"; no session/econ events, 0 cap minutes; parent-gate toggle (09 §8.2)
│  └─ photo-game/[packId].tsx  play a parent-approved photo-to-game pack (opens as a `secret` map node)
├─ (coop)/
│  ├─ lobby.tsx                SCR-13 same-device or multi-device; choose children/roles
│  └─ room/[roomId].tsx        SCR-13 campfire / co-op session (evening defence)
├─ (parent)/                   _layout: ParentGate (Face ID/PIN) + guardian session check; RBAC guard
│  ├─ index.tsx                dashboard (per child cards; "מה לעשות הערב")
│  ├─ child/[childId]/…        report, skills, settings (caps, quiet hours, subjects), profile
│  ├─ approvals/…              content packs / photo extractions awaiting approval
│  ├─ photo/capture.tsx        photo-to-game capture → crop → upload → review
│  ├─ family/…                 guardians, invites, roles
│  ├─ devices/index.tsx        devices list; devices/[deviceId].tsx (rename, mode, children,
│  │                           bedtime lock, unpair/wipe, install expiry)
│  ├─ rewards.tsx              real-world rewards (D6)
│  ├─ ai-log.tsx               "what the AI said" log
│  ├─ data.tsx                 export / delete
│  └─ debug.tsx                (owner + dev builds) perf overlay toggles, sync state, flags
├─ +not-found.tsx
└─ +html.tsx (web)             <html dir="rtl" lang="he">, COOP/COEP meta n/a (headers via server)
```

Navigation rules:
- Kids can never reach `(parent)` or `(setup)` without the gate.
- Leaving the app from the child layout requires a parent gate in `shared-kids` mode. iOS Guided Access
  is recommended for K3 (parent checklist).
- Deep links use the scheme **`olam://`** (one scheme everywhere; 10 §11.9's `familygame://` is
  superseded). Universal links need associated domains, which means the paid program, so they're
  not used. Closed route list (`src/services/deeplinks.ts`, zod-parsed; anything else → `+not-found`):
  - `olam://pair?p=<pairingId>&n=<nonce>&api=<base>` — QR pairing (S3), setup screens only.
  - `olam://invite?c=<code>` — co-admin/viewer invite (S2) → `(setup)/sign-in`.
  - `olam://report/{id}` — weekly/monthly report (10 §11.9 email link) → parent gate → `(parent)/…`.
  - `olam://approvals[/{id}]` — approvals inbox → parent gate.
  - `olam://device/{id}` — device detail (expiry/health emails) → parent gate.
  - **Dev/preview builds only** (stripped from the `family` variant by `APP_VARIANT`; 14 PQ-T-06,
    PQ-DEV-04/06): `olam://dev/fixture?id=FX-..` loads a fixture family/child from
    `apps/mobile/e2e-maestro/fixtures/` and resets the local DB; `olam://dev/setNow?ts=` injects the
    clock (`window.__olam.setNow` equivalent); `olam://dev/moveTo?x=&z=`, `olam://dev/route?id=`,
    `olam://dev/cycle?scene=&n=` drive the world/perf loops. In the `family` build these links are
    ignored and logged as `app.error{kind:'deeplink_rejected'}`.
  - A child-facing route is never a deep-link target: every link lands on setup or behind the parent
    gate, so a link in an email cannot open a session.
- `RestGate` (in `(child)/_layout` and `picker`) applies 00 B7's family mode from
  `family.prefs.shabbatMode` (`soft` default / `full` / `off`) during the pause window
  `[candle − 30 min, havdalah + 30 min]` computed by `@hebcal/core` for the family city
  (`packages/content/calendar/5787.json` is the bundled offline copy):
  - `soft`: play continues; the engine runs with `ctx.frozen=true` (no coins/gems, no goal/cap
    counters, no events open/close, camp in Shabbat scenery, levels as free practice); no notifications.
  - `full`: redirect to `/rest` (app sleeps for children; the parent gate opens it for 30 min).
  - `off`: nothing.
  - Bedtime (`device_settings.bedtime`) and a reached daily cap always redirect to `/rest` (that
    child only: the picker shows the sleeping avatar, SCR-01). Quiet days (B7a list `QD-*`) set
    `rest.mode='memorial'`-style restrictions without sleeping: celebrations capped at T1, no events,
    no boss, calm music, no chest auto-open.

**Screen ↔ route ↔ owner (every SCR in 09 §8.2 must appear here; test `routes.spec.ts` snapshots
this table from `app/` so a new screen without a route fails CI):**

| 09 screen | Route | Feature module (`src/features/…`) |
|---|---|---|
| SCR-01 picker | `/picker` | `picker/` |
| SCR-02 FTUE | `/(child)/[id]/hub` + FTUE overlay | `hub/ftue/` |
| SCR-03 hub | `/(child)/[id]/hub` | `hub/` (+ `src/world`) |
| SCR-04 map | `/(child)/[id]/map` | `map/` |
| SCR-05 world level | `/(child)/[id]/world/[zoneId]`, `/level/[levelId]` | `world/` (+ `src/world`) |
| SCR-06 episode player + mechanics 06a–06g | `/(child)/[id]/game/[gameId]` | `episode/` + `src/games/<gameId>` |
| SCR-06h English island | `/(child)/[id]/english/[islandId]` | `english/` |
| SCR-07 results + wind-down | `/(child)/[id]/results`, `/wind-down` | `episode/results/`, `episode/winddown/` |
| SCR-08 shop | `/(child)/[id]/shop` | `shop/` |
| SCR-09 avatar | `/(child)/[id]/avatar` | `avatar/` |
| SCR-10 base | `/(child)/[id]/base` | `base/` |
| SCR-11 pet | `/(child)/[id]/pet` | `pet/` |
| SCR-12 collection / sticker book | `/(child)/[id]/collection` | `collection/` |
| SCR-13 co-op lobby + campfire | `/(coop)/lobby`, `/(coop)/room/[roomId]` | `coop/` |
| SCR-14 photo-to-game (parent) | `/(parent)/photo/capture` | `parent/photo/` |
| SCR-15 parent gate | `/(parent)/_layout` ParentGate | `parent/gate/` |
| SCR-16 safe room | `/safe-room` | `saferoom/` |
| SCR-17 K3 playroom | `/(child)/[id]/playroom` | `playroom/` |
| SCR-18 pause | modal sheet inside `(child)/_layout` | `episode/pause/` |
| SCR-19 movement | `/(child)/[id]/move/[activityId]` | `movement/` |
| SCR-20 rest | `/rest` | `rest/` |
| SCR-21 car mode (P2, flag `carMode`) | `/(child)/[id]/car` | `car/` |
| chest reveal | `/(child)/[id]/chest/[chestId]` | `economy/chest/` |
| parent dashboard screens (10 §6) | `/(parent)/…` | `parent/<area>/` mirrors 10's PA-* ids |

**Component ownership (09 §7 `CMP-xx` → package):**

| Components | Owner | Why |
|---|---|---|
| CMP-01..08, 10..17, 19..20, 22..32, 46 (buttons, tiles, drag/drop, trace canvas, nikud text, bubbles, progress, counters, chest, shop card, jar, carousel, swatch, island frame, profile card, turn ring, parent gate, dialogs, toasts, hint, coach strip, loader, settings slider) | `packages/ui` (`components/`, `choreo/`, `audio/`, `haptics/`, `rtl/`, `tokens/`) | platform-neutral, used by mobile **and** parent-web; Storybook-web + Playwright per state × profile |
| CMP-09 CharacterRig, CMP-18 AvatarPreview | `apps/mobile/src/platform/rive` + contract in `packages/ui/src/character.ts` | needs the Rive native/web split (§4.10) |
| CMP-21 BuildGrid, CMP-47 StickerBook, CMP-48 PetView | `apps/mobile/src/features/{base,collection,pet}` | screen-specific, Skia-heavy |
| CMP-33 WorldAnchor, 34 MoveTarget, 35 OffscreenIndicator, 36 HUD, 37 VirtualJoystick, 38 Action/Jump, 39 PowerUpBanner | `apps/mobile/src/features/world/hud` (2D) + `src/world` (3D side of 33/34) | bound to the R3F canvas and Reanimated shared values |
| CMP-40 DisplayShelf/CollectionSlot | `apps/mobile/src/features/collection` | 3D/2.5D room |
| CMP-41 MovementCoach, 42 SafetyCard | `apps/mobile/src/features/movement` | sensors (§4.17) |
| CMP-43 CoopRoleCard/ReadyUp | `apps/mobile/src/features/coop` | split-zone gestures (§10.1) |
| CMP-44 CaptureFrame/CropHandles/ConfirmItemRow | `apps/mobile/src/features/parent/photo` | camera + manipulator |
| CMP-45 WindDownCard | `packages/ui` | reused by rest screen |
| tokens (`color.*`, `type.*`, `touch.*`, `spring.*`, `sfx.*`, `hap.*`) | `packages/ui/tokens/*.ts` → `tokens.json` built in CI for Figma/Rive/Blender | 09 §3 single source |

A component that has no row here cannot be merged (`tools/ci/component-owner.mjs` checks every
`CMP-` id mentioned in 09 §7 against this table).

### 4.2 State management and data access

Three layers of state:
1. **Persistent domain state** lives in SQLite through repositories (`src/db/repositories/*.ts`),
   built on drizzle. Reads use `useLiveQuery` (drizzle `expo-sqlite` driver with
   `enableChangeListener: true`) so screens update on writes.
2. **Ephemeral session/UI state** lives in Zustand stores:
   - `useSession` (active child, play session id, episode timer)
   - `useUi` (modals, toasts)
   - `useAudioPrefs`
   - `useWorld` (3D world runtime state: hero position, collected ids; *mirrored to events on
     milestones only*)
   - `useCoop`

   Stores never persist themselves. Anything that must survive a restart is an event or a doc.
3. **Server-only views** (cross-device reports, AI log, devices list from the server) use TanStack
   Query with `persistQueryClient` disabled. Offline, they show the last sync snapshot from SQLite
   instead.

**Write path (the only one allowed):**
```ts
// src/db/repositories/events.ts
await db.transaction(async (tx) => {
  const ev = makeEvent(type, payload, ctx);          // uuidv7 id, seq++, envelope (§6.4)
  await tx.insert(events).values(ev);
  await applyProjections(tx, ev);                    // packages/engine reducers → projection tables
});
syncScheduler.poke();                                // debounced background sync (§6.5)
```
- Projections are deterministic functions `(state, event) → state` in `packages/engine/projections`.
- A full rebuild (`rebuildProjections(childId)`) replays all events in canonical order:
  `(ts, deviceId, seq)`. It runs after a pull brings older or foreign events (§6.5), and in tests.

### 4.3 Engine integration

`packages/engine` exports pure modules:
- `skills` (skill graph)
- `elo` (item/child ratings)
- `srs` (Leitner box; FSRS later)
- `placement`
- `selector` (`nextItems(childState, pack, ctx) → ItemPlan[]`, honouring the C2 success bands and the
  C9 exploration share)
- `economy` (earn rules D1–D5; per-age gates)
- `episodes` (C3 caps, wind-down)
- `calendar` (B7 Shabbat/holiday pause, using a bundled Hebcal-derived table)

The app calls them only through `src/services/engineFacade.ts`, which loads the child's projection
rows, calls the engine and returns plans. **The engine never touches I/O or time.** `now` and `rng`
are injected, which keeps Playwright and tests deterministic (`window.__olam.setNow()`).

### 4.4 Content pack loader (hash-verified)

The pack manifest is versioned, immutable and content-addressed:
```json
{
  "packId": "he.letters.bet-gimel", "version": "1.3.0", "schema": 3,
  "minAppRuntime": "fp:7c1e…", "minEngine": 2, "locale": "he-IL", "bands": ["G"],   // T|G|S (00 A6), never K1/K2/K3
  "skills": ["he.letter.bet", "he.letter.gimel"],
  "approvedBy": "guardian:01J…", "approvedAt": "2026-11-02T18:00:00Z",
  "files": [
    {"path": "items.json", "sha256": "…", "bytes": 18234},
    {"path": "audio/vo/he.bet.intro.f.hila.9a1c.m4a", "sha256": "…", "bytes": 40211}
  ],
  "manifestSig": "ed25519:…"
}
```
Load flow:
1. `GET /v1/content/catalog?since=<etag>` returns the list of `{packId, version, manifestUrl}` for this
   family, approved packs only.
2. For each new version, fetch the manifest, then verify `manifestSig` against the **Ed25519 public
   key bundled in the app**. The key pair is kept in Worker secret `PACK_SIGNING_KEY`; @noble/curves
   handles verification.
3. Download files with `expo-file-system` (`File` API; resumable when large) into
   `documentDirectory/packs/<packId>/<version>/`. URLs are R2 presigned GETs, valid 15 min.
4. Compute sha256 of each file with @noble/hashes, streaming in 1 MB chunks off the UI frames. Use
   `requestIdleCallback`; `InteractionManager` was removed in RN 0.88 / SDK 58. Mismatch → delete and retry ×3 → mark the pack `failed`. The app keeps the
   previous good version.
5. Atomic switch: `content_pack.active_version = new` in one SQLite transaction. Old versions are
   deleted after 7 days, or immediately when storage is low.
6. `minAppRuntime`/`minEngine` guard: a pack that needs a newer engine stays `pending_app_update`.
   **Parent-facing (GAP-27):** a `pending_app_update` pack appears in the device health line (10
   FAM-04: "חבילה ממתינה לעדכון האפליקציה") and raises `AL-11{variant:'app_update_needed'}` at most
   once per 7 days per device; on a free-signed install the same line names the re-sign routine.
7. **`revoked` (12 §2.7.1, S0):** the catalog carries `status` per `{packId, version}`; a pack marked
   `revoked` is removed from selection on the **next catalog pull** (every sync, ≤ 5 min while online,
   also mid-episode: the current item finishes, the next item comes from another pack, and the map
   node disappears at the next picker). Its files are deleted at the next picker, `content_pack.status
   = 'revoked'`, and an `app.update`-class telemetry row `pack.revoked{packId, version}` is written.
   The E-22 "no downloads mid-episode" rule does not delay a revocation (hiding costs no bandwidth).
   Test (§16 #20): a revoked pack is unselectable within 5 min on a connected device.

**Bundled core content** (letters, numbers, core VO, hub assets) ships inside the binary under
`assets/`, so a fresh install works fully offline.

### 4.5 Audio service

`src/services/audio/AudioService.ts`, the single owner of audio:
- **Session** (called **once at startup** in `boot/`, never per scene): `setAudioModeAsync({ playsInSilentMode: true, interruptionMode: 'doNotMix',
  shouldPlayInBackground: false })`. Learning narration must be heard even when the iPhone's ring
  switch is on silent. `playsInSilentMode` is a parent toggle per device (09 §6.7); `doNotMix` is
  fixed (the game has its own music; a parent's podcast would collide with VO).
- **Interruptions (E-16 in §4.18):** phone call, FaceTime, Siri, alarm, another app taking the
  session (`AVAudioSessionInterruption` surfaces through expo-audio's status / `AppState`
  `inactive`):
  1. On interruption begin: pause all buses, pause the game loop (same path as `AppState` inactive),
     persist resume state. The VO queue item that was cut is **replayed from its start** on resume
     (an instruction half-heard is worse than none).
  2. On interruption end: do **not** auto-resume gameplay. Show the "לגעת כדי להמשיך" overlay;
     music fades back in over 600 ms after the tap; the cut VO replays.
  3. Route change (Bluetooth headphones connect/disconnect, AirPlay): keep playing; re-run the
     latency self-check silently (`AudioService.measureLatency()` on the next 3 SFX) and switch
     the haptics/visual lead per 09 §6.8 if measured latency > 120 ms.
  4. Microphone use (K1 read-aloud only) requests `allowsRecording` for that activity and releases
     it after; never at startup (it changes the speaker route on iPhone).
- **Buses:**
  - `music`: one looping player, crossfade 600 ms.
  - `vo`: a queue with no overlap; it ducks music to 25% over 150 ms.
  - `sfx`: a pool.
  - `ui`: a pool.

  Volumes are per-bus and per-child (`audio_prefs`).
- **Low-latency SFX:** at scene mount, `preload(source)` the hot set (≤ 12 sounds, expo-audio's
  first-class preload API; `clearAllPreloadedSources()` on scene exit — 14 PQ-P-06) and create
  `createAudioPlayer(src)` × 2 instances each (round-robin). Playing means `seekTo(0)` + `play()`.
  Players are created at scene mount, never inside event handlers (lint rule), and destroyed on exit
  with **`player.remove()`** (there is no `release()` in expo-audio 58.0.11). The latency target is **p95 tap→sound ≤ 50 ms** (hard release gate ≤ 80 ms, measured with 240 fps video per the device protocol), measured by the device test screen
  `/(parent)/debug` → "audio latency" (records `performance.now()` deltas with the native playback
  callback).
  - **Gate:** if p95 > 50 ms on the oldest device, implement `AudioService` on `react-native-audio-api`
    (Web-Audio-style buffers, dev build), keeping the same interface.
- **VO resolution:** `vo(key, {child})` resolves `key` + child gender variant + voice id to the asset
  path through the pack's `audioIndex`. A missing variant is a **build error** in the content linter
  (never fall back to another gender at runtime). Numbers come from the numerals lookup clips (B4).
- **Spatial-ish 3D audio:** every 6 frames compute pan and volume from hero↔source distance and angle
  (`player.volume`, stereo pan via pre-rendered L/R variants if `pan` isn't supported; *verify the pan
  API*).
- No siren-like or explosion SFX (B8). The SFX asset list is reviewed against a banned-sounds list in
  `tools/assets`.

### 4.6 Haptics service

`src/services/haptics.ts` wraps `expo-haptics` with semantic events:
- `tap` → selection
- `collect` → impactLight
- `success` → notificationSuccess
- `chestOpen` → a pattern of impactMedium ×3 at 80 ms
- `boundary` → impactSoft
- `error` → never used for kids' wrong answers (soft visual or audio only)

It's a no-op on iPad (no Taptic Engine) and on web. Per-child toggle. Rate-limited to 1 per 40 ms.

### 4.7 Asset loader and caching

- **Bundled:** core Rive files, core audio and hub GLBs come via `expo-asset`. **Fonts are embedded at
  build time by the `expo-font` config plugin (static TTFs, 09 §15 / 14 PQ-R-41) — no runtime
  `useFonts`**; the splash stays until DB migration (incl. the §6.3 ladder), the config cache and the
  picker assets are ready: target 3 s, hard timeout 4 s (`app.boot_timeout` telemetry, then the
  picker renders with whatever is ready).
- **Downloaded:** pack files go to `documentDirectory` (persistent; not `cacheDirectory`, which iOS may
  purge).
- **Images:** `expo-image` with `cachePolicy:'disk'` for remote, plus `recyclingKey` in lists.
- **Rive:** `useRiveFile` with a module-level cache keyed by asset sha. Files are released on scene
  unmount, and at most 3 are kept alive.
- **3D:** `useGLTF.preload()` for the next zone during wind-down of the current one. Geometries,
  materials and textures are disposed on zone exit, and `renderer.info.memory` is checked in debug.
- **Memory budget** (oldest device, Instruments footprint; "target / gate" tiers from 14 PQ-B-10…13 —
  a target miss is a ticket, a gate miss blocks the release):
  - home steady: target 200 MB / gate 250 MB
  - 2D gameplay steady: target 200 MB / gate 300 MB
  - 3D gameplay steady: target 300 MB / gate 400 MB
  - per-scene peak: target 500 MB / gate 650 MB
  - growth after 20 scene enter/exit cycles ≤ 20 MB (gate)

  A memory-warning handler (`AppState` `memoryWarning`) drops image, Rive and GLB caches. `AppState`
  `inactive`/`background` pauses the render loop, persists resume state and releases music.
- **Storage full (E-04):** before any pack download the loader checks
  `FileSystem.getFreeDiskStorageAsync()` and requires `free ≥ packBytes × 2 + 200 MB`; otherwise the
  pack stays `pending_storage`, the parent area shows "אין מקום באייפד (נותרו N MB)", and the kid
  keeps playing the packs already present. Old pack versions and the `expo-image` disk cache are
  purged first (oldest first) before declaring `pending_storage`. SQLite writes that fail with
  `SQLITE_FULL` are retried once after the purge; if they still fail the episode continues
  **in memory** (events buffered in `useSession().pendingEvents`), the child is not interrupted, and
  the buffer is flushed on the next successful write; the parent gets `AL-11` ("אין מקום"). Telemetry
  `app.storage_full` is emitted once per day.
  - **Storage low (GAP-40):** at `free < 500 MB` the device reports `storageFreeMb` in the `/sync`
    `device` block and the Worker raises `AL-11{variant:'storage_low'}` (≤ 1 per 7 days per device);
    new packs still download while `free ≥ packBytes × 2 + 200 MB`. The downloaded audio/TTS cache
    (`documentDirectory/audio-cache/`, per-item VO fetched outside packs) is an **LRU capped at 400 MB**
    (evict oldest-played first; bundled core VO is never evicted). A map node whose pack is
    `pending_storage` shows the 09 §9.4 line "האזור עוד נטען" — never an error.
- **Low Power Mode (E-17):** `expo-battery` `useLowPowerMode()` → `perfTier = 'low'`: ambient Rive
  loops at 30 fps, particles ≤ 3, 3D DPR capped at 1.5, `frameloop="demand"` after 1 s idle instead of
  3 s, music stems reduced to `bed` + `melody`. Gameplay still targets 60 fps; the screen refresh is
  60 Hz anyway under Low Power Mode. Nothing is said to the child.
- **Thermal throttling (E-18):** a 40-line Expo Module `olam-thermal` exposes
  `ProcessInfo.thermalState` and `os_proc_available_memory()`. On `serious`/`critical`: the same
  degradations as Low Power Mode plus shadows off and a 30 fps cap for the 3D world (`setFrameCap(30)`
  in `src/world/sim`), and the next zone load is deferred until `nominal`/`fair`. `app.perf` records
  `thermalState` histograms per session; a device that hits `serious` in ≥ 20% of 3D sessions fails
  gate #2 for that device class.
- **Textures:** WebP/PNG ≤ 1024² (props) and ≤ 2048² (hero/atlas only). **No KTX2/Basis on native**:
  it's reported broken with Expo + three.js, and its transcoder needs WASM. AVIF only after a decode-cost
  check on the oldest iPad.

### 4.8 Offline behavior

| Function | Offline |
|---|---|
| Play, progress, economy, collection | ✅ fully (local events) |
| Profile switching, picture-PIN, parent gate (PIN/Face ID) | ✅ (PIN hash synced to device) |
| Settings edits by parent on device | ✅ (doc LWW, synced later) |
| New content packs, TTS for new items | ❌ (queued; existing packs keep working) |
| Photo-to-game, companion AI turns, parent AI summary | ❌ (UI says "צריך אינטרנט"; companion falls back to scripted hints) |
| Co-op multi-device | ❌ (same-device co-op ✅) |
| Pairing a new device, inviting a guardian | ❌ |
| Remote device commands (bedtime lock) | Applied on next contact; **local bedtime schedule is enforced offline** |

Sync triggers:
- app foreground
- every 5 min while foreground and online (`expo-network` state)
- after an episode ends (`session.ended` written → push first, then pull)
- **app background** (`AppState` → `background`): push-only flush of unpushed events and dirty docs,
  ≤ 2 s budget, no pull (02 UC-15 / 01 UC-15 rely on the last sync being recent)
- a manual "סנכרון עכשיו" in the parent area

**`lastServerTime` for the engine and the economy (F2 #21):** every successful `/sync` stores
`sync_state.last_server_time = serverTime` and `last_server_time_local = localNow`. The engine facade
passes `ctx.lastServerTime` (and `ctx.now` corrected per §6.4 when |skew| > 2 min) so that: the
Sunday savings bonus is paid only when `lastServerTime ≥ weekStart` or the device has been unsynced
≥ 7 days (02 UC-15); the Shabbat window prefers server-derived time when the last sync is < 24 h old
(01 UC-15); otherwise the device clock is used. `lastServerTime` is also shown in the device card.

**Timezone and travel (01 DD-31, 02 UC-16, GAP-39):** `DayKey` is the **device** local day
(`ts + tzOffsetMin`, 02 §2.1; implemented once in `packages/shared/dayKey.ts`, §6.4). Shabbat/holiday
and quiet-day windows always use the **family city** (`family.prefs.city`, OPD-04) until a guardian
sets `family_prefs.travelCity` ("עיר נסיעה", 10 SET-01; field-level LWW, cleared by the parent when
back). When `deviceTz ≠ family.tz` for > 24 h the app writes `app.tz_mismatch{deviceTz, familyTz}`
(telemetry, once per day) and the parent area shows the travel banner (10). The daily cap and the
family-goal week follow `DayKey` of the device that created the event (02 UC-16); no event is ever
re-dated.

**First launch offline (E-01).** A freshly installed (or freshly re-signed) app with no network:
- If `device_self` exists (re-sign/reinstall over an existing install): boot normally, everything in
  the table above works; the sync scheduler waits for connectivity.
- If `device_self` is empty (new device, or restore after delete): the setup screen shows
  "צריך אינטרנט כדי לצרף את המכשיר למשפחה" with a retry button; pairing needs the Worker (there is no
  offline family). The owner's **bootstrap** path also needs the network. Nothing else is offered —
  there is no "guest" child, because events must carry a real `childId`/`familyId`.
- If pairing succeeded but the **initial pull** was cut off: the device is in state
  `bootstrapping` (`sync_state.pull_cursor = 0`, `device_self` present). The picker shows only the
  allowed children from `familySnapshot` (returned by `/pairings/claim`, so the picker always has
  names/avatars), play is allowed immediately with the **bundled core content**, and the pull resumes
  on the next trigger. Projections are rebuilt once the first full pull completes (§6.5).

**Long offline periods.** There is no limit: 30 days offline still plays. Caps and calendar use the
device clock; `rest.tsx` uses the bundled Hebcal table (B7) so Shabbat still pauses offline. On
reconnect, if |skew| > 2 min the telemetry row is logged and HLC physical time takes
`max(local, server)` (§6.5).

### 4.9 3D world layer (E1b)

**Stack:**
- three 0.186.1
- `@react-three/fiber@9.8.2` using the **`@react-three/fiber/native`** entry on iOS (Canvas on
  expo-gl) and the default entry on web (WebGL2)
- drei 10.7.9, native-safe subset: `useGLTF` (no Draco/meshopt/KTX2 on native), `useAnimations`,
  `Instances`
- three-mesh-bvh 0.9.16

**Module boundaries (enforced by ESLint import rules):**
```
src/world/            NO react-native imports
  sim/                fixed-timestep simulation (60 Hz step, render interpolates): hero controller,
                      kinematics (ground raycast via BVH, push-out vs AABB/circle colliders),
                      triggers (distance checks), scripted tweens (jumps = analytic parabola)
  scene/              R3F components: Zone, Hero, Collectible(instanced), Friend, Sky, BlobShadow
  materials/          toon gradient maps, inverted-hull outline
  levels/             level definitions (JSON): spawn, triggers → learning actions, gimmick config
  store.ts            zustand world store (hero pos, collected set, active trigger)
  api.ts              world ⇄ app contract: onTrigger(triggerId) → app opens learning action;
                      world.resolve(triggerId, outcome) → plays rescue/collect juice
src/platform/gl/
  Canvas.native.tsx   import { Canvas } from '@react-three/fiber/native'
  Canvas.web.tsx      import { Canvas } from '@react-three/fiber'
  CanvasWebView.tsx   fallback 1: same world bundle in react-native-webview (postMessage bridge)
src/features/world/
  WorldScreen.tsx     Canvas + 2D HUD overlay (Reanimated); projects 3D anchors to screen for
                      Hebrew labels/speech bubbles (never 3D text)
```

**Rules:**
- No React state updates inside `useFrame`. Mutate refs; the HUD reads Reanimated shared values
  updated at most 10×/s.
- Tap-to-move raycasts **only** the walkable ground mesh (BVH), not R3F's event system across meshes.
  Forgiving radius snaps to the nearest walkable point or interactable (≥ 64 pt equivalent for K3).
- `frameloop="demand"` when idle more than 3 s (toddler staring). Canvas `frameloop="never"` and
  unmounted while a 2D mini-game is open. Pause on `AppState` background.
- The world is **not mirrored** for RTL. Only the HUD is RTL. Joystick left and action right by
  default (thumb ergonomics), swappable in settings.
- **Controls per age:**
  - K3: tap-to-move + auto-interact; no jumps or fail states.
  - K2: tap-to-move + big jump button; auto-run mini-levels with tap-to-jump.
  - K1: optional floating joystick (15% deadzone) + action button + optional swipe-orbit with
    auto-recenter.
  - Tilt is a bonus mini-game only.
- **Determinism hooks (web/dev):** `window.__olam.world = { moveTo(x,z), teleport(id), step(n),
  pause(), state() }`, plus `window.__perf` (fps p50/p95, `renderer.info.render.calls`, triangles,
  JS ms per frame). 09 §13.2 uses the same names (`window.__olam.ready`, `window.__olam.world.moveTo`).
- **Orientation (E-13):** world/level/game/coop routes lock **landscape** on iPhone via
  `expo-screen-orientation` (`lockAsync(LANDSCAPE)` on mount, `unlockAsync` on unmount); hub, map,
  shop, avatar, collection allow both on iPad (09 §8.1). On an iPad rotation during 3D: the sim keeps
  stepping, `useThree().size` triggers `camera.aspect`/`updateProjectionMatrix`, HUD anchors
  re-project on the next frame, and the HUD does a 200 ms cross-fade (CHOREO-10). The GL context is
  **not** recreated (expo-gl keeps it); a test rotates the web viewport 5× and asserts no context
  loss, no duplicate `Canvas`, and that `renderer.info.memory.textures` is unchanged.
- **GL context loss** (`webglcontextlost` on web; expo-gl `onContextLost` ⟂VERIFY event name on
  native): pause the sim, show the per-route error boundary's "נטען מחדש" with the mascot, re-mount
  the `Canvas` once; on a second loss in 60 s fall back to the 2.5D Skia zone renderer for this
  session and emit `app.error{kind:'gl_lost'}`.
- **Split View / Stage Manager:** disabled by `UIRequiresFullScreen: true` (§0). The 3D canvas
  therefore always gets the full screen; no mid-size layout exists in the Playwright matrix.

**Performance budgets (oldest family device; A12-class assumed until the parent lists devices):**

| Metric | Budget |
|---|---|
| fps | ≥ 55 p95 over a 60 s scripted walk (≥ 45 floor under 10-min soak) |
| draw calls | ≤ 80 (A12), ≤ 120 newer |
| visible triangles | ≤ 100k (A12) – 150k |
| JS per frame | ≤ 4 ms target, ≤ 6 ms gate |
| textures | ≤ 1024², total GPU textures ≤ 64 MB; toon/Lambert only; blob shadows; DPR `[1, 2]` |
| hero | 3–6k tris, ≤ 40 bones, one skinned mesh |
| level assets | ≤ 50 MB each; cold zone load ≤ 3 s |
| memory | **target** 300 MB steady (3D) / **gate** 400 MB; **peak target** 500 MB / **gate** 650 MB; ≤ 20 MB growth over 20 zone enter/exit cycles (14 PQ-B-10…13 tiers; 13 §10.3's "≤ 300 MB" is the target) |

**glTF asset pipeline (`tools/assets/glb`):**
1. Sources: Kenney/Quaternius CC0 (spike) → custom hero later (Meshy/Tripo or designer; Rive stays
   for 2D characters).
2. Optional headless Blender normalize (`blender -b -P tools/assets/glb/normalize.py`): apply
   transforms, scale to meters, merge static meshes by material, generate a `*_collision` simplified
   walkable mesh.
3. `gltf-transform` 4.5.1: `dedup`, `prune`, `weld`, `resample`, `simplify --ratio`, `resize
   --width 1024`, `quantize`. This emits two variants:
   - **native:** no extensions that need runtime WASM decoders (no Draco/meshopt/KTX2).
     KHR_mesh_quantization is allowed if three loads it natively; *verify in spike*.
   - **web:** Draco/meshopt/KTX2 allowed.
4. **Budget linter** (`pnpm lint:assets`) fails CI on any GLB over its tri/material/texture/bone
   budget from `assets/models/budgets.json`.
5. Provenance: `assets/LICENSES.json` entry per file (source URL, license, author, date, AI vendor and
   prompt if generated). Required by E10.

**Spike (≈10 agent-days, from `tech_3d_world.md`) — go/no-go gates:**

| Gate | When | GO criteria | If NO-GO |
|---|---|---|---|
| #1 Platform | day 1–2 | R3F v9 native + expo-gl renders a GLB on SDK 58 / RN 0.88 **dev build** without red screen or EXC_BAD_ACCESS; survives 10 bg/fg and 5 unmount cycles. Web screenshot non-blank and matches baseline. | Fallback 1: three.js in WebView; re-run #1 |
| #2 Performance | day 3–5 | Budgets above on the oldest device (Instruments via Remote Control on the Mac) | DPR 1, no shadows, instancing → WebView compare → Filament (native-only; Playwright covers HUD only) or Skia 2.5D |
| #3 Kid usability | day 6–8 | K3 reaches 5 collectibles unassisted ≤ 3 min; K2 finishes the auto-run level ≤ 3 tries; ≤ 10% toddler "nothing happens" taps; Playwright route suite runs headless in < 5 min | Simplify controls; K3 goes 2.5D Skia diorama |
| #4 Assets | day 9–10 | Pipeline + linter pass in the container; one AI prop + one AI-rigged variant animate on device and web | Stay on Quaternius/Kenney rigs |

The spike's outcome is recorded as ADR `docs/decisions/0003-world-renderer.md`, and §4.9 is updated.

### 4.10 Characters (Rive) and placeholders

`src/platform/rive/`:
- `Character.native.tsx`: `useRiveFile({source})` + `<RiveView file stateMachine autoPlay>`, with
  view-model data binding via the typed bindings from `npx rive-gen-types`.
- `Character.web.tsx`: `@rive-app/react-canvas` `useRive` with the same props contract.
- `Character.placeholder.tsx`: a Skia/Reanimated sprite (idle bob, cheer, think). It's used when the
  Rive native module is missing, which is detected with a guarded `require` +
  `NitroModules.hasHybridObject`, and during the concept stage before rigs are approved.

The contract lives in `packages/ui/src/character.ts`:
`{ characterId, mood: 'idle'|'cheer'|'think'|'oops'|'wave', lookAt?, say?: VoKey, onEvent }`.
Data-binding property names are shared with the designer in `assets/rive/contract.json`.

### 4.11 Error handling

- **Error boundaries:**
  - Root boundary → "משהו השתבש. ננסה שוב?" (neutral, with mascot) → reload.
  - Per-route boundaries for `(child)` and `(parent)`.
  - Mini-game boundary → returns the child to the hub with **no loss**: the attempt is discarded, no
    penalty.
  - **Per-canvas boundary** (Skia / GL / Rive surfaces, 14 PQ-P-10): on a throw inside a canvas the
    boundary (1) releases that scene's audio players (`remove()`) and Rive instances, (2) retries the
    canvas **once in place** from `child_resume`, (3) on a second failure swaps to the 2D fallback
    (Skia 2.5D for GL, the sprite placeholder for Rive, a static image for a Skia game) for the rest
    of the session and emits `app.error{kind:'canvas', surface}`, (4) a third failure → hub, no loss.
    Native crashes (GL/Rive/Skia) are not catchable here; they surface as `app.start{prevExitClean:
    false}` (§13).
- **Classes and handling:**

  | Class | Example | Handling |
  |---|---|---|
  | Expected domain | pack pending update | inline UI state |
  | Network | timeout, 5xx, DNS | retry with exponential backoff (0.5 s → 30 s, jitter, cap 15 min for sync), queue. Every call: 10 s timeout + `AbortController`. The child never sees a network error (09 §9.4: silence); the parent area shows `ParentToast` |
  | Worker down (E-09) | 502/503/522/530 from Cloudflare, or `/v1/health` failing | same as Network; the app is local-first so play continues. Parent area tabs that are server-only show the last snapshot with "מעודכן לפני N דק׳". After 24 h without a successful sync `AL-11` is raised by the Worker when it is back (device `last_seen_at`) and locally as a parent-gate banner |
  | AI vendor down (E-09) | Anthropic 529 overloaded / 5xx / timeout 20 s | companion: scripted hint from the pack immediately (no spinner > 1 s), `ai_log.flagged='vendor_down'`; photo extract: job `queued` with retry at 1, 5, 15 min then `failed` + `AL-08b` text "ננסה שוב מאוחר יותר"; batch: next 15-min poll retries `batches.create` up to 3× per night, then the night is skipped and logged |
  | $ cap hit (E-09) | 503 `cap_reached` (12 §1.9 `costGuard` state machine, §8.4) | **80–100 % of the daily cap:** new batch jobs are deferred to the next night, everything else runs, PA-HOME shows "AI: 80% מהתקציב היומי". **≥ 100 %:** companion (P3) → scripted hints for the rest of the day (child unaware); photo-to-game (P2) → "המשחק יוכן מחר בבוקר (06:00)" + "להכין עכשיו" (Face ID, `OVERRIDE_USD` $5, ≤ 3/month) and the job goes to the `deferred` queue; batch skipped until 00:00 Israel; one PA-INBOX notice, email after 3 consecutive days (`AL-11{variant:'ai_cap'}`). **Monthly ≥ 100 %:** batch off until the 1st, P2 only with override, P3 scripted. **Kill switch** `killSwitches.ai`: everything scripted/code-only, "לא זמין כרגע" for the parent. `PATCH /admin/limits` changes the caps within 12 §1.9's ranges (owner) |
| Auth | 401 | refresh once, then re-pair prompt (parent only) |
  | Integrity | hash mismatch, **R2 object missing (404) for a catalogued file** (E-10) | discard + refetch ×3 with backoff; a 404 marks the pack version `failed{reason:'asset_missing'}`, keeps the previous version active, and raises `ops/notify {kind:'asset_missing', packId, path}` so the agent republishes the pack. A **VO file missing at play time** (should be impossible after verification) plays the bundled generic line for that message key (`vo.generic.<intent>`) and shows the text; `app.error{kind:'vo_missing'}` plus the learn-class `asset.missing{assetId, kind:'vo'\|'image'\|'level_chunk'\|'rive'\|'glb'}` (01 DD-38) |

- **`VoIntent` (closed enum, `packages/shared/vo.ts`; 12 V22 and the player agree on it):**
  `'instruction' | 'replay' | 'praise' | 'retry' | 'hint' | 'worked_example' | 'reveal' | 'greeting' |
  'transition' | 'goodnight' | 'safety' | 'calm' | 'coop' | 'english_enter' | 'english_exit' |
  'name_call'`. Every `voice: true` message key declares its `intent`; the bundled core pack must
  contain `vo.generic.<intent>` for every member × `{m, f, pl}` (content validator `V22`, build
  fails otherwise); `name_call` falls back to the companion's two-note signature, never to a name
  spoken by TTS. Adding a member is a `packages/shared` PR that also adds the three generic lines.
  | Local DB (E-02/E-03) | migration failure, `SQLITE_CORRUPT`, `quick_check` ≠ ok | §6.3 recovery ladder: retry → restore from server copy; the child sees the picker a few seconds later, never a stack trace |
  | Storage | `SQLITE_FULL`, disk < 200 MB | §4.7 storage-full rule |
  | Programming | bug | boundary + `app.error` event |

- **API errors** use one shape:
  `{ "error": { "code": "pairing_expired", "message": "…", "requestId": "…", "retryable": false } }`.
  The codes enum lives in `packages/shared/errors.ts`; client UI copy is in i18n.
- **Error events:** `app.error` (stack truncated to 4 KB, route, release, runtimeVersion) are stored
  locally and synced as **telemetry-class** events, kept 30 days (§5.9).

### 4.12 Feature flags, remote config, capabilities

- `GET /v1/config` (ETag, cached in SQLite `remote_config`) returns:
  ```json
  { "version": 12, "flags": { "coop.multiDevice": false, "ai.companion": true, "photo.enabled": true,
    "world.renderer": "r3f", "auth.apple": false, "push.remote": false },
    "killSwitches": { "ai": false, "photo": false, "coop": false },
    "models": { "content": "claude-opus-5-5", "judge": "claude-sonnet-5-5", "vision": "claude-sonnet-5-5",
                "companion": "claude-haiku-5-5", "summary": "claude-haiku-5-5" },
    "minRuntime": "fp:…", "perChild": { "<childId>": { "flags": {} } } }
  ```
- Bundled defaults live in `src/services/flags/defaults.ts`. Evaluation order: defaults → server
  global → per-family → per-child → per-device. Flags never gate safety rules; those are code.
- `capabilities.ts` is computed at boot: `{ path: 'web'|'free'|'paid', rive, haptics, faceId, push,
  siwa, appAttest, gl }`. Features consult it (e.g. hide "Sign in with Apple" on free signing).

### 4.13 i18n with grammatical gender (B3, B4)

**Catalog format** (`packages/i18n/he/*.json5`). Each message is ICU MessageFormat plus metadata:
```json5
{
  "hub.greeting": {
    audience: "child",             // child | parent | both
    voice: true,                   // has pre-rendered VO; key per variant
    msg: "{gender, select, f {שלום {name}! מוכנה להרפתקה?} m {שלום {name}! מוכן להרפתקה?} pl {שלום לכולם! מוכנים להרפתקה?} other {שלום {name}! יוצאים להרפתקה?}}",
    nikud: "{gender, select, f {שָׁלוֹם {name}! מוּכָנָה לְהַרְפַּתְקָה?} m {…} pl {…} other {…}}",
    note: "neutral 'other' used when gender unknown (K3 pending)"
  },
  "game.tap_star": { audience: "child", voice: true, neutral: true, msg: "לגעת בכוכב", nikud: "לָגַעַת בַּכּוֹכָב" }
}
```
**Runtime:**
- `t(key, {child | addressee: 'f'|'m'|'pl'|'n', ...vars})`. It picks `nikud` vs plain from the child's
  nikud-track state (B5), then formats with `intl-messageformat` (cached compiled formatters).
- `addressee` comes from the profile's `grammatical_gender`. **Never** from an LLM or TTS.
- Family/co-op screens use `pl`.
- Parent-facing strings use `pl` or neutral forms, or the guardian's own gender when it's stored and
  the message is addressed to one guardian.

**Lints** (`pnpm lint:i18n`, CI-blocking):
1. Every `audience: child|both` message has `neutral: true`, or a `gender` select with `f`, `m`, `pl`
   and `other`.
2. `nikud` exists for every child message and has the same placeholders.
3. Messages are free of banned praise words (C4 list from the UX spec).
4. Every `voice: true` message has an audio file for each variant × voice in the active voice set.
5. No digits in `voice: true` messages. Numbers use `{n, numeral, fem|masc|construct}`, a custom
   formatter backed by the hand-checked numerals table (B4).
6. Slash forms ("מוכן/ה") are banned in child messages.

**Plural rules:** Hebrew CLDR categories `one/two/other`. Boot runs a self-test of
`new Intl.PluralRules('he').select(2) === 'two'` and loads the `@formatjs/intl-pluralrules`
polyfill if that fails.

### 4.14 RTL implementation rules (binding for all UI code)

- **R-1** The app never branches on `I18nManager.isRTL`. `<DirProvider value="rtl">` at the root sets
  `style={{direction: 'rtl'}}` on the root View. Native builds also set `forcesRTL: true` (Info.plist
  `ExpoLocalization_forcesRTL`) so system UI matches. Web sets `<html dir="rtl" lang="he">`.
- **R-2** Use logical props only: `marginStart/End`, `paddingStart/End`, `start/end`,
  `borderStartWidth`. Banned by lint: `Left/Right` variants.
- **R-3** Every text goes through `<HText>`, which always sets `textAlign: dir==='rtl'?'right':'left'`
  and `writingDirection` explicitly. iOS doesn't infer alignment reliably (RN issue history).
- **R-4** `Row` primitive = `flexDirection:'row'` under the inherited `direction`, which flips
  automatically. Never `row-reverse` to fake RTL.
- **R-5** English islands: `<Dir value="ltr">` subtree, with the English character voice, the
  distinct color accent and jingle (B2). Mixed inline text (Hebrew sentence with an English word)
  uses Unicode isolates `⁨…⁩` inserted by the `bidi()` helper. Never manual LRM hacks.
- **R-6** Directional icons (back, next, progress arrows) come from `packages/ui/icons` with
  `mirrorInRtl` metadata. "Back" points right in RTL. Progress bars fill from right to left. Number
  lines are **LTR in math** (numbers increase left→right, as in Israeli math books); this exception is
  documented in the math UX spec and marked `<Dir value="ltr">`.
- **R-7** Stock components that ignore direction (native sliders, native headers) are not used.
  Headers are custom (`headerShown:false`), sliders are custom Skia/Gesture.
- **R-8** Gesture semantics: swipe "forward" = swipe **left** in RTL (content moves right→left). The
  helper `useForwardSwipe()` encapsulates this.
- **R-9** 3D world, Skia canvases and game coordinates are direction-agnostic. Any text in them is a
  2D overlay (R-3). Skia paragraph text is not used for Hebrew in kid screens until nikud rendering is
  verified on device (B5).
- **R-10** Font: **Noto Sans Hebrew** for reading text and **Rubik** for display (09 §3.6, both
  with verified nikud GPOS; Open Sans, Heebo, Varela Round and Karantina are banned for pointed text),
  tested on device on day 1 with the nikud torture string in `packages/i18n/test/nikud-sample.txt`.
  GATE-UX-2 may swap the reading font (Fredoka / Frank Ruhl Libre candidates); the token name stays.
- **R-11** Dynamic Type (E-14): child screens set `allowFontScaling={false}` at the `HText` level
  and scale by the per-profile slider (100–150 %, 09 §3.6) instead; parent screens honour system
  Dynamic Type up to **AX3 (≈ 235 %)** with no clipping (10 AC-35), larger sizes scroll. Nikud
  text at any scale keeps `lineHeight = round(size × ratio)` recomputed from the scaled size (never a
  fixed pt), `paddingVertical ≥ 0.12em`, `overflow: visible` on the nearest ancestor, and `numberOfLines`
  only with `adjustsFontSizeToFit` **off** (shrinking pointed text detaches marks on iOS ⟂VERIFY on
  iOS 27). Playwright shoots every nikud screen at 100/135/150 % (09 §13.2) and the parent screens at
  AX3; a mark crossing the glyph box is a failed `UX-AC-03b` (bounding-box check per word token).
- **R-12** VoiceOver in Hebrew (E-15): every `HText` child-facing word gets `accessibilityLabel` =
  the **unpointed** string (VoiceOver's Hebrew voice reads combining marks letter-by-letter on isolated
  glyphs); a single letter tile gets the letter *name* ("בֵּית") as label; numbers use the B4 lookup
  (gendered) in the label; English islands set `accessibilityLanguage="en-US"` on the `<Dir
  value="ltr">` root so VoiceOver switches voice (B2); directional hints ("החלק שמאלה") are never in
  labels (RTL-neutral verbs only). Focus order follows DOM/tree order, which is RTL under
  `direction:'rtl'`; a Playwright `axe` + `accessibility tree` snapshot per screen asserts the order
  starts at the start-edge counter (CMP-36). VoiceOver + Reanimated: the HUD exposes
  `accessibilityLiveRegion="polite"` only for the friends counter, never for per-frame values.
- **R-13 Known RN RTL bugs → regression tests** (each has a Playwright or Maestro case tagged
  `@rtl-bug`):
  1. `ScrollView`/`FlatList` horizontal initial offset and `contentOffset` mirrored (iOS) → use
     `inverted`-free custom carousels with explicit `scrollToOffset` from the start edge; test:
     carousel shows item 1 at the right edge on first and second visit.
  2. `TextInput` caret/placeholder alignment under `direction:'rtl'` → always set `textAlign:'right'`
     + `writingDirection:'rtl'` on inputs (parent area only; kids have no inputs except K1 words).
  3. `Image`/`expo-image` `resizeMode` ignores direction (fine) but `transform:[{scaleX:-1}]` must
     not be applied to illustrations (lint `mirrorInRtl` manifest, 09 UX-AC-14).
  4. `Modal` and native sheets flip entry edge on iOS 17+ with `forcesRTL` → custom sheets only (R-7).
  5. `position:'absolute'` with `left/right` → lint-banned (R-2); `start/end` only.
  6. Negative `marginStart` on a `Row` inside `Reanimated.View` with `transform` → test both
     `translateX` signs via `dirX()` (09 §3.11) in each `CHOREO`.
  7. Progress bars filled by `width` + `alignSelf:'flex-start'` render from the left under RTL →
     use `alignSelf:'flex-start'` inside a `Row` (flips) and never `left:0`.
  8. `FlashList` `horizontal` + RTL: ⟂VERIFY on FlashList 2.3.2; until verified, horizontal lists
     are custom `ScrollView`s.
  9. react-native-screens native header is LTR unless `semanticContentAttribute` is set → headers are
     custom (`headerShown:false`, R-7).
  10. Web: `dir="rtl"` on `<html>` plus `unicode-bidi: isolate` on every `HText` (set by the
      `bidi()` helper) so mixed Hebrew/Latin strings never reorder punctuation.

### 4.15 Performance techniques

- Gestures and frame loops run on the UI thread with **Reanimated 4 worklets**: `useFrameCallback`
  for 2D mini-games, and `Gesture.Pan().onUpdate` worklets writing shared values. No `runOnJS` per
  frame. `scheduleOnRN` (worklets 0.13) is used only for discrete events.
- **Skia:** one `<Canvas>` per mini-game, with picture recording for static layers, `useClock` for
  particles, and atlases for sprites (`drawAtlas`) when more than 50 sprites are on screen.
- **React:** React Compiler on; lists use FlashList 2. Stores use selectors (`useStore(s => s.x)`) to
  avoid wide re-renders. Expensive engine calls run between episodes or in
  `requestIdleCallback`/`startTransition`. **`InteractionManager` and `Touchable*` were removed in RN
  0.88 / SDK 58** (lint-banned; use `Pressable`).
- **The 25 performance rules** from `performance_reliability_playbook.md` §11A are binding, are
  copied into `CLAUDE.md`, and are owned as a spec by `14-performance-quality-playbook.md`. This
  spec maps each to its enforcement point (lint = §2.4, CI = §11.5, device = §12.5, review = PR
  checklist):
  | # | Rule (short) | Enforced by |
  |---|---|---|
  | 1 | per-frame state in shared values/refs, never React/Zustand | lint `.value` in render; review |
  | 2 | JS thread idle during play; discrete events only | Time Profiler (§12.5 #5); `app.perf.jsMsP95` |
  | 3 | gestures as UI-thread worklets; `scheduleOnRN` only on end/hit | review; lint `runOnJS` in `onUpdate` |
  | 4 | animate transform/opacity only | lint `olam/perf-bans` (layout props in `withTiming`) |
  | 5 | time-based motion (`dt`), 60/120 Hz; `CADisableMinimumFrameDurationOnPhone` | app.config; sim fixed step (§4.9) |
  | 6 | Reanimated `IOS_SYNCHRONOUSLY_UPDATE_UI_PROPS` (opt-in; `USE_COMMIT_HOOK_ONLY_FOR_REACT_COMMITS` is already the default in 4.7.1) | `package.json` → `"reanimated": {"staticFeatureFlags": {"IOS_SYNCHRONOUSLY_UPDATE_UI_PROPS": true}}`; A/B on device is ADR-0006 (14 PQ-R-06); ⟂VERIFY no conflict with Skia 2.14 |
  | 7 | React Compiler on, zero bail-outs in `src/games/**`, `src/world/**`; atomic selectors | CI `react-compiler-healthcheck` |
  | 8 | no `InteractionManager`; `requestIdleCallback`/`startTransition` | lint |
  | 9 | one Skia Canvas per scene; atlases ≤ 2048²; paints/shaders/paths created once | review; asset linter |
  | 10 | Rive Nitro, one file per character, idle `frameRate` capped, ≤ 3 instances | `RiveRegistry` asserts in dev |
  | 11 | R3F `frameloop="demand"` when idle; dpr ≤ 2; ≤ 80/120 draw calls; instancing; no allocation in `useFrame`; dispose on unmount; one GL context | lint `new THREE.*`; `window.__perf` in Playwright (trend); gate #2 |
  | 12 | never stack live Skia + Rive + GL in one frame unless profiled | `SurfaceBudget` dev assert (≤ 2 Skia, ≤ 3 Rive, 1 GL) |
  | 13 | audio session once; SFX preloaded/pooled; released on exit | §4.5; lint `createAudioPlayer` in handlers |
  | 14 | haptics fire with the visual, throttled, iPad no-op | §4.6 |
  | 15 | fonts via config plugin; splash held until fonts + home essentials, **4 s timeout** | §4.7 (3 s target, 4 s hard) |
  | 16 | explicit image size/aspectRatio; sprites `transition={0}` | lint |
  | 17 | RTL set natively; world never mirrored; LTR isolation for numbers/math | §4.14 |
  | 18 | nikud: lineHeight per the 09 §3.6 tokens (1.6 K2 read styles, 1.55 K1 pointed, 1.45 K1 unpointed — never a single ratio), vertical padding, `FixedSlot` containers, tested at max scale; in-game labels capped | R-11; Playwright at 150 %/AX3 |
  | 19 | fixed logical game resolution scaled into the safe area; orientation locked per route | §4.9 orientation |
  | 20 | error boundary at root and per scene; scenes rebuild from persisted state | §4.11 |
  | 21 | pause everything on `inactive`; persist; resume to "tap to continue" | §4.5 interruptions; §4.18 |
  | 22 | memory-warning handler clears caches; per-scene memory budget | §4.7 |
  | 23 | SQLite WAL, async only, batched transactions at checkpoints | repositories; lint `*Sync(` from expo-sqlite outside `boot/` |
  | 24 | offline-first; timeout + `AbortController` on every call; never block UI on network | `apiClient` wrapper is the only `fetch` (lint) |
  | 25 | perf decisions only on Release builds on the oldest iPad | §12.5 |
- **Startup:** cold start to interactive picker ≤ 2.5 s on the oldest iPad. Only the core fonts and
  the picker Rive load before first paint; the hub GLB is preloaded while the picker is shown.
- **Tap-to-feedback:** visual response in the same frame (worklet), haptic ≤ 20 ms, sound p95 ≤ 50 ms.
- **Debug overlay** (`/(parent)/debug`): fps, JS frame ms, UI frame ms, draw calls, memory. In dev
  builds it can be pinned on screen.

### 4.16 Multi-profile on a shared iPad

- Device mode `shared-kids` shows the picker of **allowed children for this device** (H1).
- Selecting a child opens a `play_session` event (`session.started`). All events carry `childId` and
  `sessionId`.
- Per-child **resume state** (last zone, last hero position, pending chest, **`ftueCompletedAt`**) is a
  doc `child_resume/<childId>`, saved on every background/switch **and every ≤ 20 s checkpoint during
  play**. The FTUE overlay (SCR-02) runs only while `ftueCompletedAt` is `null`; it is set when the
  first rescue lands (01 §14.2), so a second or restored device never replays the 60-s opening
  (GAP-01; the doc syncs like any other). Each save also writes the previous good copy as `kind='child_resume_bak'` in the same
  transaction (14 PQ-P-11). Fallback order on resume: `child_resume` (zod-valid) → `child_resume_bak`
  → hub spawn with the episode's earned items intact (`app.resume_fallback` telemetry). A resume doc
  is never blank-reset.
- **Switching** has three entry points:
  - Long-press (1.5 s) on the avatar badge → "החלפת שחקן" → back to the picker. This saves resume and
    closes the session (`session.ended{reason:'parent_exit'}` — 08 §1.10 maps the old `'switch'`
    to `parent_exit`; the whole `SessionEndReason` enum is in §5.8).
  - Inactivity of 5 min also returns to the picker (`reason:'inactivity'`; GAP-06). The idle gap is
    **not charged to the cap**: caps count `focusedMs` (08 `Session.focusedMs`, from `focus.span`),
    not wall time, so a child who walks away loses no minutes and the level resumes from the
    ≤ 20 s checkpoint next time.
  - An episode cap reached → the wind-down screen, then the picker (`reason:'daily_cap'`).
- **Picture-PIN** (H3): optional per child (default on for K1). A 3-picture sequence from a 3×3 grid,
  stored as `scrypt(sequence, salt)` in the child doc. Verified locally; after 3 misses it says "אפשר
  לבקש עזרה מאמא או אבא" and **the grid is hidden for 60 s** (no visible countdown, no sound; the
  4th tap just re-shows the message), telemetry `picturepin.miss{childId}` (cls telemetry, feeds 10
  M-14 "sibling access attempts"). **Reset:** the parent clears it in the child's settings
  (`PUT /children/:id/picture-pin {clear:true}`) → at the child's next launch a one-time "בחרי/בחר 3
  תמונות" step runs (GAP-07). No lockout beyond the 60 s, no punishment.
- **K3 lap mode / launch policy (GAP-11b, 01 DD-18):** `child_settings.launchMode ∈ {'lap',
  'coplayFirst', 'solo'}` (defaults by tier: K3 `lap`, K2 `coplayFirst`, K1 `solo`; parent-editable in
  10 §6.13). `lap`: every launch needs a parent-present confirmation (the parent gate, or the parent's
  "hold for 2 s" button on a device paired to a guardian session) and a lap segment with no parent
  prompt answered within 60 s switches to the playroom (01 UC-07). `coplayFirst`: the first segment
  of the day needs the parent's "אני כאן" hold, the rest is solo. `solo`: picker tap only. Episode
  cap 3–5 min for K3 (C3).
- **Caps** are per child per day, enforced on the device from local events plus a server-merged total
  (cross-device usage is merged at sync; offline overrun is tolerated and never punished).
  - The cap and the "day" are **02 `DayKey`** (device local day, monotonic per device, §6.4; 00
    OPD-26 names the cap values). `packages/shared/dayKey.ts` has one constant `DAY_START_HOUR`
    (default 0); if 02/08 ever adopt a 04:00 boundary (15 GAP-44) it changes there only.
  - **Device-local override (RBAC L, GAP-22):** a parent at the local gate without a guardian session
    writes `device_settings.childOverrides[childId].{dailyCapMin, quietHours}` (field-level LWW,
    this device only). Effective cap = `min(child_settings.dailyCapMin, override)`; effective quiet
    hours = the union. The child's screen in 10 shows "הוגדר במכשיר {device}" next to such a value.
  - **"+10 דקות" (01 DD-09, GAP-02):** `POST /children/:id/cap-bonus` (guardian, parent gate; or the
    same action offline from the gate, queued in `outbox_request`) writes the admin event
    `cap.bonus_granted{childId, minutes:10, guardianId}`; the server (and the local fold) reject a
    second grant for the same `DayKey` (422 `bonus_used`); audit row; it extends minutes only — never
    an episode, never a reward (02).

### 4.17 Motion input (G6)

- **Tier 1 (spike-ready):** `expo-sensors` `DeviceMotion` at 60 Hz (`setUpdateInterval(16)`).
  - Tilt-to-steer runs through a worklet low-pass filter (α = 0.15) with calibration on start ("hold
    the iPad how you like, then tap").
  - Shake detection: |a| > 2.2 g for > 60 ms.
  - Requires `NSMotionUsageDescription` (Hebrew string).
  - Only for **iPhone-in-hand** or seated iPad tilt. Never "jump while holding the iPad".
- **Tier 2 (later, dev build only):** front camera pose. Either `react-native-vision-camera` v5 with a
  custom Expo Module running Apple Vision `DetectHumanBodyPoseRequest`/`DetectHumanHandPoseRequest`
  (iOS 18+) on frames at ≤ 720p and 15–30 fps, or MediaPipe via v4 plugins (rejected: Google SDK plus
  an upgrade trap).
  - Only discrete events reach JS (`jump`, `lean_left`, `arms_up`, `fingers=3`).
  - **No frames or landmarks are stored or sent.** A visible camera-on indicator is shown, and parents
    enable it per child behind the gate.
- **Clap detection (later):** mic RMS onset detection on-device, with no recording.
- **Safety:** every movement activity has a seated alternative (same learning action via tap). The
  parent sets the space checklist once. Each activity has a max of 90 s of continuous movement.
- **Web/testing:** DeviceMotion is simulated via `window.__olam.motion.feed(samples)` in Playwright.

### 4.18 Edge-case walk (binding rules + acceptance tests)

Each row is one rule, one owner section, and one test id (`E-xx` appears in the test name). The
child-facing outcome always follows 09 §9.4 ("silence or a mascot sneeze"; details only in the
parent area).

| Id | Situation | Rule | Kid sees | Parent sees | Test |
|---|---|---|---|---|---|
| E-01 | First launch offline | §4.8: existing device boots and plays; new device needs network to pair; `bootstrapping` state plays bundled content with the `familySnapshot` picker | picker or "צריך אינטרנט" (setup only) | — | Maestro `offline-first-launch`; Playwright route-intercept |
| E-02 | DB migration fails at boot | §6.3 ladder: retry once → backup file → fresh DB → restore from server (cursor 0) → if also offline: play with bundled content in `bootstrapping`; `local_only` lost, parent told | splash ≤ 4 s, then picker | banner "המכשיר שוחזר מהענן; צ'ק-אינים מקומיים לא שוחזרו" + `app.error{kind:'migration_failed'}` | jest fixture DBs v-1..v-3 + a poisoned migration |
| E-03 | Corrupted local DB | boot runs `PRAGMA quick_check(1)` (≤ 50 ms at our size); on failure or `SQLITE_CORRUPT` at any time → same ladder as E-02 after an attempt to salvage unpushed events (`SELECT` from the corrupt file in a try; on success they are pushed first) | as E-02 | as E-02 | unit: corrupt header fixture |
| E-04 | Storage full | §4.7: no new packs (`pending_storage`), purge caches, in-memory event buffer on `SQLITE_FULL` | plays what is installed | "אין מקום באייפד" + AL-11 | unit: injected `SQLITE_FULL` |
| E-05 | OTA update ready mid-session | §3.3 `UpdateGate`: applied only at cold start, idle picker (≥ 60 s), or parent action; never in `(child)`/co-op | nothing | "עדכון זמין" in parent area | unit: gate with mounted child route |
| E-06 | 7-day free-signing expiry | §3.3 walk; app never deleted; data stays; re-sign restores | iOS "Unable to Verify App" (system) | email day 6, banner ≤ 1 day, devices list | routine smoke; acceptance #2 |
| E-07 | Pairing code brute force | §9.2: 6 digits, 10 min TTL, 5 attempts per pairing then the pairing is **burned** (new one required, audit `pairing.burned`), 5/min/IP, ≤ 3 open pairings per family, QR carries a 128-bit nonce | — | "הקוד בוטל אחרי 5 ניסיונות" | worker test: 6th attempt → 410 + burned |
| E-08 | Lost/stolen device, wipe | §5.3 S5: unpair revokes the token now; `wipe` runs on next contact (push events → delete DB, packs, Keychain → ack); `wipe_pending` auto-revokes after 7 days; PIN hashes on the device are scrypt-hardened (N=2^15) and grant only the local parent gate; guardian refresh tokens are Face ID/passcode-bound in Keychain | setup screen after revoke ("המכשיר נותק מהמשפחה") | device status `unpaired`/`wipe_pending` with timestamps | worker test S5; Maestro `wipe` |
| E-09 | Worker down / AI vendor down / $ cap hit | §4.11 table | plays; companion silently scripted | last-snapshot banners, AL-11, cost alerts | worker tests with fetch mocks: 529, 503 |
| E-10 | R2 asset missing | §4.11 Integrity row: version `failed{asset_missing}`, previous stays, agent notified; missing VO at play → generic line + text | generic VO line | approvals tab shows pack `failed` | worker test 404 on presign target; unit `vo_missing` |
| E-11 | Two parents edit settings at once | §6.5 **field-level** LWW for `child_settings`, `family_prefs`, `device_settings`, `reward` docs: each field carries its own HLC; concurrent edits of different fields both survive; same field → higher HLC wins and the loser's screen shows "עודכן על ידי {name} לפני רגע" with the new value (TanStack refetch on `docs` pull). Identity rows are server-authoritative with `If-Match: hlc` → 409 `conflict` + reload | — | toast + merged values | fast-check: two writers, any interleaving → converged map |
| E-12 | Audio interrupted (call, Siri, alarm, BT route) | §4.5 interruptions | "לגעת כדי להמשיך"; VO replays | — | device checklist #14; unit on interruption events |
| E-13 | Orientation change during 3D; Split View / Stage Manager | §4.9: resize without context loss; landscape lock on iPhone game routes; multitasking disabled (`UIRequiresFullScreen`) | 200 ms HUD cross-fade | — | Playwright viewport rotate ×5; device checklist |
| E-14 | Dynamic Type extremes with nikud | R-11 | child text unaffected by system size (profile slider only) | AX3 parent screens, no clipping | Playwright 150 % / AX3 + `UX-AC-03b` |
| E-15 | VoiceOver in Hebrew | R-12 | labels in unpointed Hebrew, English islands in en-US | fully accessible parent area | axe + a11y-tree snapshot |
| E-16 | App backgrounded mid-episode (home button, Control Center, lock) | playbook rule 21: pause loops on `inactive`, persist `child_resume` + flush events; on `active` show "לגעת כדי להמשיך"; if away > 10 min the episode ends cleanly (`session.ended{reason:'background'}`), the chest/receipt earned so far is kept (no loss) and the next launch opens the picker | tap-to-continue, or picker | — | Maestro background/foreground; unit timer |
| E-17 | Low Power Mode | §4.7 | nothing | — | device protocol pass 2 |
| E-18 | Thermal throttling | §4.7 | nothing (30 fps cap) | ops tab: thermal histogram | 30-min soak |
| E-19 | Clock set wrong / far-future device | §6.4 server clamp +5 min; §6.5 skew telemetry; daily caps use `DayKey` from the device clock but the server recomputes `daily_usage` from `received_at` when skew > 2 h and the device adopts it on the next pull | may get an extra episode once | skew noted in device detail | worker test: ts = now + 1 y |
| E-20 | Same child on two devices at once | allowed (C5: no punishment). Each device opens its own `session`; caps merge at sync (overrun tolerated); economy double-spend → `econ.overdraft`, balance shown as `max(0, coins)` | both play | usage per device | fast-check convergence |
| E-21 | Child deleted while a device is offline with that child's events | server accepts the events (soft-deleted child keeps 30 days), projections skip deleted children; device drops the child from the picker on the next pull | child vanishes from picker after sync | audit row | worker test |
| E-22 | Pack approved while the child is mid-episode | catalog pull after the episode only (`ContentPackLoader` holds downloads while `useSession().active`); the atomic switch happens at the picker | nothing mid-episode | approvals → "יורד למכשירים" | unit: loader paused during session |
| E-23 | Guardian removed / role downgraded while signed in on a device | next API call → 403 `forbidden` → app clears the guardian session on that device (device token stays), parent area falls back to local mode L | — | "ההרשאה שונתה" | worker + client test |
| E-24 | Shabbat window / quiet day starts mid-episode | `RestGate` lets the current episode finish (≤ 20 min); from the window start the engine runs `frozen` (no coins/counters, B7 `soft`) and celebrations drop to the `calm` register; in `full` mode the next episode is blocked by `rest.tsx`; a quiet day (B7a) applies T1 caps immediately | wind-down as usual; Shabbat scenery or rest screen | — | unit with fake clock around candle lighting, all three modes |
| E-25 | Pack revoked (S0 content) while installed, even mid-episode | §4.4 #7: unselectable on the next catalog pull (≤ 5 min online), current item finishes, files deleted at the picker, map node gone | nothing (the next item is from another pack) | approvals tab: pack `revoked` + reason | worker test: catalog returns `revoked` → device projection has no selectable item within one sync |
| E-26 | Family deletion in its 7-day grace (owner pressed DELETE /family) | §7.3: every device call answers 410 `family_deleting`; the app switches to **offline-only** (play continues locally, sync paused, parent gate banner "המשפחה נמחקת ב-{date}; ביטול באתר"); after the purge the next call is 401 → setup screen and local data is deleted on the parent's confirmation | plays as usual | banner + cancel link (web) | worker test with fake clock: 410 during grace, 401 after; client test: no push during grace |
| E-27 | Pairing claimed on a device that already holds another family's data | §5.3 S3: the app compares the local `device_self.family_id` with the claim result; on mismatch it refuses to store the new token until the parent confirms "למחוק את הנתונים של {old family} במכשיר הזה" by typing the old family name; unsynced events of the old family are pushed first if the old token still works | setup screen only | confirmation dialog | client test: mismatch → no token written without confirm |

---

## 5. Identity, users, devices and admin (section H)

### 5.1 Domain model

```
Family (tenant, exactly 1 in practice; schema supports N)
 ├─ Guardian (user)        role: owner | coadmin | viewer      identities: email, apple(sub)
 │    └─ Session (refresh-token family per guardian×client)
 ├─ Invite (pending guardian: role, email?, code, expires)
 ├─ Child (profile, no credentials)   grammatical_gender f|m|n(pending) · birthdate · grade · avatar
 ├─ Device                 type ipad|iphone|web · mode shared_kids|single_child|parent_only
 │    ├─ DeviceChild (allowed children per device)
 │    ├─ DeviceCommand (bedtime_lock|unlock|unpair|wipe|refresh_content|resync|child_remove)  queued → acked
 │    │     resync = "push everything unpushed now, then pull from cursor 0 and rebuild projections"
 │    │     child_remove{childId} = drop the child from this device's picker, delete its local projections
 │    │       and resume docs (events stay until pushed; GAP-51), ack; sent on child soft-delete and on
 │    │       removal from device_children
 │    └─ device token (hashed)  · install expiry (free signing) · last_seen · app/runtime version
 ├─ Pairing (one-time code/QR to attach a new Device; 10 min TTL)
 ├─ AuditLog (append-only admin actions)
 └─ Events (learning/telemetry log, per child/device/session)  · Docs (LWW records)
```

**Principals and how they authenticate:**

| Principal | Credential | Where stored | Lifetime |
|---|---|---|---|
| Device | `dev_<base64url 32B>` opaque token, SHA-256 hashed in D1 | iOS Keychain (`expo-secure-store`, `WHEN_UNLOCKED_THIS_DEVICE_ONLY`) | until unpaired. Rotated on each re-pair, and every 180 days via `/v1/devices/self/rotate` |
| Guardian (app) | access JWT (HS256, 15 min) + refresh `grt_<32B>` (rotating, 60 days, reuse detection) | refresh in Keychain with `requireAuthentication: true` (Face ID/passcode to read) | 60 days sliding |
| Guardian (web) | same, refresh in `__Host-olam_rt` HttpOnly Secure SameSite=Strict cookie | browser | 30 days sliding |
| Child | none. Selected on a device that is allowed; optional picture-PIN is local UX, not auth | — | — |

Authorization for any API call follows these rules:
- Device token required for device-scoped routes (sync, content, ai for child).
- Guardian JWT required for admin routes. On a family device, admin calls carry **both**
  (`Authorization: Bearer <jwt>` + `X-Olam-Device: <device token>`), so the audit log records the
  device.
- The web dashboard is guardian-only.

### 5.2 Identity providers (pluggable; `/v1/auth/{provider}/…`)

| Provider | Phase | Flow | Notes |
|---|---|---|---|
| `bootstrap` | 1 (day 0) | the agent runs `pnpm ops bootstrap-owner --email <owner>` (wrangler D1 via CI) → prints a one-time **setup code** `XXXX-XXXX-XXXX` (**24 h TTL, single use**, scrypt-hashed in `bootstrap_codes`) → owner enters it in the app → owner guardian + family created. **Co-admin without a sending domain (00 H2, GAP-36):** `pnpm ops bootstrap-guardian --role coadmin --email <email>` (same table, `role` column, `family_id` set) prints a code the co-admin enters in `(setup)/sign-in` → "יש לי קוד הקמה" → guardian created with that role, same session rules as S2 | lets the family start before any email domain exists. **Revocation (00 E7 / F1 #27):** the code row is marked `used_at` on redeem; any unused code for that email is revoked the moment an `email_code` sign-in for the same email succeeds, and the nightly cron deletes expired rows. 5 redeem attempts per IP per hour (RL_AUTH); a wrong code answers 404 `code_invalid` without revealing whether the email exists |
| `email_code` | 1 (primary) | `POST /v1/auth/email/start {email}` → 6-digit code + web link by email (Resend) → `POST /v1/auth/email/verify {email, code}` → session | needs a verified sending domain (~$10/yr). 10-min TTL, 5 attempts per code, 5 starts/hour/email, 20/hour/IP. Always answers 202 (no account enumeration). Only emails of existing guardians/invites receive codes. |
| `apple` | 2 (paid program) | native `expo-apple-authentication` → `identityToken` + raw nonce → `POST /v1/auth/apple {identityToken, nonce, fullName?}` → verify with `jose` (`createRemoteJWKSet('https://appleid.apple.com/auth/keys')`, `iss=https://appleid.apple.com`, `aud=<bundle id>` or the web Services ID, `nonce=sha256(raw)`) → link by `sub` | web uses Sign in with Apple JS + Services ID (paid). Linking an Apple id to an existing guardian requires being signed in (no auto-merge by email: private-relay addresses differ). |
| `passkey` | later (web only) | WebAuthn on the dashboard origin | optional |

Account linking: a guardian may hold several identities (`guardian_identities` rows). The `email` and
`apple` providers resolve to the same guardian only through an explicit "link" action while signed in.

### 5.3 Onboarding sequences (text sequence diagrams)

**S1 — Owner creates the family (phase 1, bootstrap path)**
```
Agent(CI)        Worker/D1                 Owner app (iPad, fresh install)
  │ ops bootstrap-owner(email)
  │──INSERT bootstrap_codes(hash, email, exp 24h)──►│
  │◄─ prints code ABCD-EFGH-JKLM (shown to parent in chat, once)
                                         welcome → "אני הורה – הקמת משפחה" → "יש לי קוד הקמה"
                 ◄── POST /v1/auth/bootstrap/redeem {code, guardianName, email} ──│
   verify hash, unused, not expired; create family, guardian(role=owner), identity(email, unverified→
   verified later by first email code), session; mark code used; audit(family.created)
                 ── 201 {accessToken, refreshToken, family, guardian} ──────────►│
                                         family.tsx: name, Shabbat/holiday prefs, timezone (Asia/Jerusalem)
                 ◄── PATCH /v1/family ────────────────────────────────────────────│
                                         children/new.tsx ×N
                 ◄── POST /v1/children {name, nickname, birthdate, gender, grade, avatar} ──│
                                         parent-pin.tsx: PIN (6 digits) + Face ID enable
                 ◄── PUT /v1/guardians/me/pin {pinHash (scrypt, client-side salt)} ──│
                                         "this device" pairing (implicit for owner's first device):
                 ◄── POST /v1/devices/self {name, type, mode, allowedChildIds} ──│
                 ── 201 {deviceId, deviceToken} ──────────────────────────────────►│
                                         done → placement expedition per child
```

**S2 — Owner invites a co-admin**
```
Owner app/web                     Worker                         Co-admin (own iPhone)
 POST /v1/invites {role:'coadmin', email?} ─►  create invite(code 8 chars, exp 7d), audit(invite.created)
 ◄─ {inviteId, code, link: https://parent.<domain>/join?c=…, qr}
 share via WhatsApp/AirDrop (outside app) ───────────────────────────► opens link (web) or app "הצטרפות"
                                    ◄─ POST /v1/invites/accept {code, provider:'email', email} ─┤
                                    send email code ───────────────────────────────────────► inbox
                                    ◄─ POST /v1/auth/email/verify {email, code, inviteCode} ─┤
       create guardian(role from invite), identity, session; invite.used; audit(guardian.joined)
                                    ── {accessToken, refreshToken} ───────────────────────► sets PIN, Face ID
 notification (email) "X הצטרף/ה כמנהל/ת משנה"
```

**S3 — Pair a new device (QR or 6-digit code)**
```
Guardian device (signed in)                 Worker/D1                      New device (fresh install)
 devices → "הוספת מכשיר": name, type, mode, allowed children
 POST /v1/pairings {name, mode, allowedChildIds} ─►  pairing(code6 hashed, nonce, exp 10m, attempts 0)
 ◄─ {pairingId, code:"482 913", qr:"olam://pair?p=<pairingId>&n=<nonce>&api=<base>"}
 shows QR + code (big digits, Hebrew instructions)                         welcome → "צירוף מכשיר למשפחה"
                                                                            scan QR (expo-camera) or type code
                                        ◄─ POST /v1/pairings/claim {pairingId?+nonce | code, deviceInfo} ─┤
   rate-limit (5/10min/IP; 5 attempts per pairing, then the pairing is BURNED: cancelled_at set,
   audit(pairing.burned), guardian screen says "הקוד בוטל אחרי 5 ניסיונות — צרו קוד חדש"; ≤ 3 open
   pairings per family); verify; create device + device_children;
   issue device token; pairing.claimedBy; audit(device.paired by guardian)
                                        ── {deviceId, deviceToken, familySnapshot} ────────────────►
 guardian screen live-updates (poll 2s) "המכשיר צורף ✓"                    stores token in Keychain,
                                                                            initial pull sync → picker
```
**Device that already belongs to a family (E-27, GAP-57):** before storing the new token the app
compares `device_self.family_id` with `familySnapshot.family.id`. Same family → the device is simply
re-paired (old token revoked server-side). Different family → the claim is kept in memory only and
the parent must type the old family's name to confirm the wipe; the app first tries to push the old
family's unpushed events with the old token (best effort, ≤ 10 s), then deletes SQLite, packs and
Keychain entries of the old family and stores the new token. Cancelling leaves the old family intact
and the new pairing unclaimed (it expires after its 10 min).

On shared kid devices no guardian session is stored by default. The parent area uses the PIN or Face
ID gate locally. If a guardian wants admin actions from that device, they sign in there once
("התחברות הורה במכשיר זה"), and the refresh token is stored with `requireAuthentication: true`.

**S4 — Parent gate on any device**
```
child taps hidden-corner "adults" icon (hold 2s, 09 CMP-25) → ParentGate
  ├─ capabilities.faceId && device_settings.gate.biometrics !== 'off'
  │     → LocalAuthentication.authenticateAsync({promptMessage:"כניסת הורים",
  │         disableDeviceFallback: mode === 'shared_kids' || mode === 'single_child' ? true : false })
  │       ·  shared/kid devices: NO fallback to the iPad passcode (kids know it; 10 §18.2) — a Face ID
  │          miss goes straight to the PIN pad
  │       ·  parent_only devices: fallback allowed (the phone passcode is the parent's)
  │       ·  ON-06 "ילדים רשומים ב-Face ID / מכירים את הקוד" → device_settings.gate.biometrics='off' → PIN only
  │     success → guardian picker ("מי נכנס? אמא / אבא", 2 big cards; skipped when only 1 guardian) so the
  │               audit log records who entered (10 SEC-02) →
  │               if that guardian's session is present on this device: unlock Keychain refresh → access JWT → (parent) routes
  │               else: local-only parent mode L (device-local settings + docs LWW via the device token with
  │                     actor = 'device-local:<guardianId>'; restricted set; see RBAC)
  └─ PIN pad (6 digits, scrypt verify against synced guardian pin hashes, which also identifies the guardian;
       5 fails → 60 s cooldown, doubling each round (60→120→240…), reset after a success; the cooldown is
       stored in kv-store so a restart doesn't skip it; the calm "בקשו מאבא או מאמא" screen, 09 CMP-25)
No arithmetic gates (E7). `sfx.parent.gate` plays on success; nothing on failure (no "error" sound for a child who tapped by mistake).
```

**S5 — Remote bedtime lock / unpair / wipe**
```
Guardian (web/app)            Worker                                    Target device
 POST /v1/devices/:id/commands {type:'bedtime_lock', until:'07:00'} ─► device_commands(queued); audit
                                                                         ── delivered in next /v1/sync
                                                                            response `commands[]` (or
                                                                            immediate if foreground: the
                                                                            app polls /v1/devices/self/commands
                                                                            every 60s while foreground)
                                       ◄─ POST /v1/devices/self/commands/:cid/ack {status:'applied'} ─┤
 UI shows "ננעל ✓ (לפני 2 דק׳)"
unpair: **push, then revoke** (GAP-57b): the server sets `devices.status='unpair_pending'`; the token stays
        valid only for /sync push, /commands and /ack; on its next contact the device flushes every
        unpushed event and dirty doc, acks `unpair`, then the token is revoked → the device shows
        "המכשיר נותק מהמשפחה" and returns to setup, keeping local data until wipe/reinstall. If the
        device does not come online within 7 days the token is revoked anyway (unsynced events lost;
        the devices list shows "נותק; N אירועים לא סונכרנו" from the last known `seq` gap).
        `{force:true}` (lost/stolen device, 10 FAM-04 "המכשיר אבד") revokes immediately and queues `wipe`.
wipe:   command delivered on next contact (token stays valid only for /commands + /ack); device pushes
        any unsynced events first, then deletes SQLite + packs + Keychain, acks, then token revoked.
        If device never comes online: token already marked `wipe_pending` → revoked after 7 days.
```

### 5.4 RBAC permission matrix

Roles: **O** = owner, **C** = coadmin, **V** = viewer, **D** = device token (by mode), **L** = local
parent gate without a guardian session (device-local actions only).

| Permission | O | C | V | D (shared/single) | L |
|---|---|---|---|---|---|
| `family.read` (name, children list, celebrations) | ✅ | ✅ | ✅ | ✅ (allowed children only) | ✅ |
| `family.update` (name, tz, Shabbat/holiday prefs) | ✅ | ✅ | ❌ | ❌ | ❌ |
| `family.delete` / `data.export` | ✅ | ❌ / ✅ | ❌ | ❌ | ❌ |
| `guardian.invite` (coadmin/viewer) | ✅ | viewer only | ❌ | ❌ | ❌ |
| `guardian.remove` / `guardian.changeRole` | ✅ | ❌ | ❌ | ❌ | ❌ |
| `child.create/update/delete` | ✅ | ✅ (no delete) | ❌ | ❌ | ❌ |
| `child.settings` (caps, quiet hours, subjects weights, nikud track, picture-PIN) | ✅ | ✅ | ❌ | ❌ | caps/quiet hours on this device only |
| `child.reports.read` (skills, accuracy) | ✅ | ✅ | ❌ (celebrations only; scores opt-in by owner) | ❌ | ✅ (this device's children) |
| `content.approve` (packs, photo extractions) | ✅ | ✅ | ❌ | ❌ | ❌ |
| `rewards.manage` (D6) | ✅ | ✅ | ❌ | ❌ | ❌ |
| `ai.log.read` | ✅ | ✅ | ❌ | ❌ | ❌ |
| `photo.capture` (start photo-to-game) | ✅ | ✅ | ❌ | ❌ | ❌ |
| `device.pair` | ✅ | ✅ | ❌ | ❌ | ❌ |
| `device.update` (rename, mode, allowed children) | ✅ | ✅ | ❌ | ❌ | ❌ |
| `device.command.lock/unlock` | ✅ | ✅ | ❌ | ❌ | ✅ (this device) |
| `device.command.unpair/wipe` | ✅ | ✅ | ❌ | self-unpair only | ❌ |
| `sync.push/pull`, `content.read`, `tts.read`, `ai.companion` | — | — | — | ✅ | — |
| `admin.ops` (backups, flags, cost caps) | ✅ | ❌ | ❌ | ❌ | ❌ |

The table lives in `packages/shared/rbac.ts` as data (`PERMS[role][perm]`), used by:
- Hono middleware `requirePerm('child.settings')`
- client UI (`can('device.command.wipe')`)
- a test that snapshots the table (any change needs a reviewed diff)

### 5.5 Device modes and allowed children

| Mode | Launch behavior | Parent area | Typical device |
|---|---|---|---|
| `shared_kids` | picker of `device_children`; K3 needs parent-present | gate → local or signed-in guardian | family iPad |
| `single_child` | straight into that child's hub; switching hidden (gate) | gate | K1's own iPhone/iPad |
| `parent_only` | parent dashboard; can launch a child in "parent-present" mode | always signed-in guardian | parents' iPhones |

Bedtime has two layers (GAP-04): a **per-child bedtime** (`child_settings.bedtime {start, end, days}`;
defaults per 10 §6.13 / 00 OPD-26: K3 19:30, K2 20:00, K1 20:30) and an optional **per-device
schedule** (doc `device_settings/<deviceId>`: `bedtime {start:'19:30', end:'07:00', days}`). Both are
enforced locally and offline; the **effective lock is the earliest** of the two. A new session for a
child starts only if `bedtime − now ≥ plannedEpisodeMin + 2 min`; otherwise the picker shows the
sleeping avatar and writes `launch.blocked{reason:'bedtime'}` (§5.8). In bedtime, the app shows a
calm "לילה טוב" screen (SCR-20) with no game access, and the parent gate unlocks it for 30 min
(device-wide; at the end of the window a 2-min warning, then the current beat lands and the picker
returns). A remote `bedtime_lock` command applies the lock immediately until a given time.

### 5.6 Audit log

- **What is logged:** every admin mutation, plus auth events:
  - sign-in, sign-in failed burst, refresh-reuse detected
  - pairing created/claimed/expired
  - device commands
  - guardian invited/joined/removed/role changed
  - child created/updated/deleted
  - settings changed (diff)
  - content approved/rejected
  - rewards changed
  - data export/delete
  - AI cost cap hit
  - flags changed
- **Row shape:** `{id, family_id, at, actor_type: guardian|device|system|agent, actor_id, device_id,
  action, target_type, target_id, diff_json (redacted: PIN hashes and tokens never logged),
  ip_hash (sha256(ip+daily salt)), ua, request_id}`.
- Append-only. There is no update or delete route. Visible to O/C in the dashboard ("יומן פעולות",
  filterable).
- Retention: 24 months. Cron purges older rows.

### 5.7 Analytics event envelope (H5)

Every event (learning and telemetry) carries:
```ts
type EventEnvelope = {
  id: string;            // uuidv7 (time-ordered)
  familyId: string;      // ulid-like uuidv7
  childId: string | null;// null for device/guardian-level events
  deviceId: string;
  sessionId: string | null;   // play session (child) or app session (device)
  appVersion: string;    // e.g. "1.4.0+fp:7c1e…/ota:2026-11-02T10:11Z"
  seq: number;           // device-local monotonic
  ts: number;            // device epoch ms
  tzOffsetMin: number;   // e.g. 120 / 180
  type: EventType;       // dotted, versioned in payload schema registry
  v: number;             // payload schema version
  cls: 'learn' | 'econ' | 'telemetry' | 'admin' | 'local_only';
  payload: unknown;      // validated by zod per type
};
```
- `local_only` events are emotional check-ins (`behavior.signal{kind:'emotion_checkin'}`, C8/C8a)
  and raw per-tap telemetry. They **never leave the device**: the sync pusher filters by class, and a
  test asserts it. Per 00 C8a the check-in **label is not persisted beyond the episode**: the event
  row is deleted at `session.ended`; what survives is the 14-day aggregate counter table
  `fs_local_counters` (07 §2.7; `{childId, dayKey, ruleId, count}`), which is **not an event table, is
  excluded from the event log, from `/sync` push and from export**, and is read only by the on-device
  `evaluateFlags` (08 §9.3). The only thing that may leave the device is the derived boolean inside
  the encrypted `flags/<childId>` doc (§9.8) when consent `T4b` / `consent.checkinNudge` is on — 07's
  `rf_raised{ruleId, week}` is a field of that encrypted doc, not a plain event.
- The dashboard filters by child (primary) and device (secondary: usage minutes per device, last sync,
  install expiry, app version).

### 5.8 Event types (initial registry, `packages/shared/events.ts`)

| Type | cls | Payload (abridged) |
|---|---|---|
| `session.started` / `session.ended` | learn | started: `{mode: 08 Session.mode ('episode'\|'mini'\|'placement'\|'boss'\|'benchmark'\|'reentry'\|'playroom'\|'coop') \| 'safe_room', launchMode:'lap'\|'coplayFirst'\|'solo', coPlay:boolean, plannedMinutes}`; ended (`v:2`): `{reason: SessionEndReason, plannedMinutes, blocksCompleted, focusedMs}`. **`SessionEndReason` is 08 §1.10's enum, imported from `packages/shared/session.ts`:** `'completed' \| 'daily_cap' \| 'child_exit' \| 'parent_exit' \| 'frustration_break' \| 'disengaged' \| 'bedtime_lock' \| 'inactivity' \| 'app_killed'` (old wire values map: `switch`/`parent` → `parent_exit`, `cap` → `daily_cap`, `background` (E-16 > 10 min) → `app_killed`, `done` → `completed`; v1 rows are upcast by the reducer). `mode:'safe_room'` (SCR-16, GAP-19b) is wire-only: `toEngineEvent` drops it, it adds **0 minutes to any cap**, and 10 PA-LOG files it under "רוגע" |
| `episode.completed` | learn | `{episodeId, coop:boolean, participants[], durationMs, celebrationMs}` (02 `episode_completed`; 09 UX-AC-08 budget uses `celebrationMs/durationMs`) |
| `placement.step` / `placement.finished` | learn | `{probeId, skill, outcome, rtMs}` / `{subject, strand, theta, se, reason:'completed'\|'interrupted'\|'resumed_later'\|'lazy', quality}` (08 §3, §10.5.1; 10 W-ASM) |
| `benchmark.result` | learn | `{window:'BOY'\|'MOY'\|'EOY', strand, theta, se, wordsCorrect45s?, wcpm?, errors?, pointed?}` (08 §8; 10 M-11) |
| `item.presented` / `item.answered` (**v2**) | learn | v1 fields `{packId, packVer, itemId, skill, template, answer, correct, firstTry, hintUsed, workedExample, isReview, stretch, embodied, rtMs, attempt}` (02 §2.1 `item_answered` + 08 §4 RT filters) **plus v2** (08 §1.4 `Attempt`, imported from `packages/engine`): `outcome:'correct'\|'wrong'\|'too_tricky'\|'timeout'\|'abandoned'`, `chosenErrorTag?`, `touchesBeforeAnswer`, `idleMaxMs`, `appBackgrounded:boolean`, `context:{mode, role, assisted:'parent'\|'sibling'\|'none', wrapperTopic?}`, `steps?:{total, correct}` (O.STEP partial credit), `representation?:'concrete'\|'pictorial'\|'abstract'` (O.REP), `answerableAt`, `firstResponseAt`. The reducer reads v1 rows through 08's safe defaults (`outcome` from `correct`, `touchesBeforeAnswer=1`, `idleMaxMs=0`, `assisted:'none'`) so old logs replay unchanged |
| `hint.shown` | learn | `{itemId, level, source:'scripted'\|'ai'}` |
| `behavior.signal` | learn (**`local_only` when `kind='emotion_checkin'`**) | payload = 08 §1.9 `BehaviorSignal` (`kind` ∈ `error, hint_request, hint_available_unused, challenge_choice, topic_choice, revisit, i_wonder, optional_extra, reaction, dwell_after_complete, early_quit, idle, rage_tap, rapid_guess_burst, break_request, calm_tool, recovery, novelty_offer, not_into_this, want_more, emotion_checkin, parent_correction` + `context`); replaces the earlier `checkin.mood`. The envelope `cls` is derived from `kind` by `makeEvent` (a test asserts `emotion_checkin` can never be `learn`) |
| `skill.level_changed` / `skill.level_reverted` | learn | `{skill, from:LevelId, to:LevelId, reason:'evidence'\|'delayed_check'\|'parent'\|'rollover', evidence:{n, acc}}` / `{skill, from:LevelId, to:LevelId, cause:'session_reattribute'\|'reset_progress', sessionId?}` (emitted by the engine reducer, deterministic; 08 §1.1 LevelId `new…master`, §10.5.3; 02's `skill_state_changed` uses the `packages/shared/levels.ts` mapping and ignores `level_reverted` for the wallet — coins/gems/badges are never taken back, C5) |
| `skill.delayed_check` | learn | `{skill, gapDays, passed}` (02 `delayed_check_passed`) |
| `review.rung_passed` / `receipt.shown` / `episode.closed` / `season.tier_reached` / `brain_token.deposited` / `tool.state_changed` / `campfire.lantern_lit` | learn | 02 §2.1 economy inputs: `{skillId, rungDays:1\|3\|7\|14\|30\|60}` / `{episodeId}` / `{episodeId, reason:'abandoned'\|'profileSwitch'\|'cap'\|'parentStop', levelsCompleted}` / `{seasonId, tier}` / `{itemId}` / `{toolId, to:'seen'\|'tried'\|'used'\|'chosen'\|'transferred'}` / `{source:'circuit'\|'reading'}` (the lantern is light only, never logs/coins). `review.rung_passed` is emitted by the engine reducer on every rung pass |
| `screen.enter` / `screen.exit` | learn | `{screenType, ms}` with `screenType` = the `focus.span` enum + `'picker'\|'results'\|'winddown'\|'rest'\|'safe_room'\|'parent'`; **one `exit` per visit** (ms = dwell), never per transition; 10 M-02 |
| `app.background` / `app.foreground` | telemetry | `{route, sessionOpen:boolean}` / `{route, awayMs}` (10 M-02, E-16) |
| `creation.saved` | learn | `{kind:'build'\|'drawing'\|'story', id}` (10 M-13; the creation itself is a doc/asset, the event is the fact) |
| `launch.blocked` | learn (device-level, `childId` optional) | `{reason:'quiet_hours'\|'bedtime'\|'shabbat_full'\|'cap'\|'lap_needs_parent'}` (10 M-14, AL-06; written once per child per minute at most) |
| `parent.tonight_done` | admin | `{cardId, childId}` ("עשינו ✔" on the evening card, 10 §4.1) |
| `parent.input` | admin | payload = 08 §1.9א `ParentInput` union (`questionnaire`, `session_reattribute{sessionId, to}`, `reset_profile_model`, `reset_progress{subjects}`, `skill_action`, `flag_action`, `class_topic`, `grade_rollover`, `interest_override`, `disposition_correction`, `external_evidence`); written only by guardian-scoped routes (§7.3, §7.10a), audit-logged, synced to all devices and folded in canonical order; `session_reattribute` / `reset_progress` trigger the engine's checkpoint replay (08 §10.5.4). The §7.10a admin events `econ.refund`, `sticker.granted`, `mission.confirmed` stay separate types |
| `cap.bonus_granted` | admin | `{childId, minutes:10, guardianId}` (§4.16; ≤ 1 per child per `DayKey`) |
| `asset.missing` | learn | `{assetId, kind:'vo'\|'image'\|'level_chunk'\|'rive'\|'glb'}` (01 DD-38; the player kept going) |
| `picturepin.miss` / `app.tz_mismatch` / `pack.revoked` | telemetry | `{childId}` / `{deviceTz, familyTz}` / `{packId, version}` (§4.16, §4.8, §4.4) |
| `fact.reported` | admin | `{packId, factCardId, note}` (06 §12.6 errata; `POST /content/fact-report`) |
| `world.trigger` / `world.rescue` / `world.collect` / `world.secret` / `world.boss` | learn | `{zoneId, levelId, triggerId, friendId?, pieceId?, secretId?, bossId?, first?}` (02 `friend_rescued`, `puzzle_piece_found`, `secret_level_cleared`, `boss_defeated`, `region_boss_defeated`) |
| `focus.span` | learn | `{start, end, focusedSec, screenType:'episode'\|'world'\|'hub'\|'shop'\|'base'\|'collection'\|'coop'\|'move'}` — **computed on the device** by `src/services/telemetry/focus.ts` from the raw tap stream (which stays in memory and is never stored); one event per screen visit ≥ 10 s; 10 §4 M-01 reads it. Raw taps never leave the device (10 §18.2) |
| `mission.confirmed` / `mentor.completed` / `holiday.completed` | learn | `{missionId, confirmedBy}` / `{mentorId, menteeId}` / `{eventId}` (02 §2.1) |
| `econ.spend` / `chest.opened` / `inventory.equip` / `wishlist.set` / `econ.overdraft` | econ | `{itemId, price, currency}` / `{chestId}` / `{slot, itemId}` / `{itemId\|null}` / `{coins}` — **user actions and anomalies only**. Earnings, chest contents and grants are **not stored as events**: they are a deterministic fold of `learn` events by `packages/engine/economy` (02 §2.1), so two devices can never double-pay. Chest contents are fixed by `chestId = hash(sourceEventId)` (D2) |
| `coop.joined` / `coop.contribution` | learn | `{roomId, role, contribution}` |
| `motion.activity` | learn | `{activity, counts:{jumps:n}, seated:boolean}` (aggregates only) |
| `app.error` / `app.perf` / `app.start` / `app.update` / `app.storage_full` / `app.resigned` | telemetry | `{route, message, stack≤4KB, kind?}` / `{fpsP95, jsMsP95, coldStartMs, memPeakMb, thermal:{nominal,fair,serious,critical}, scene}` / `{runtimeVersion, updateId, signedAt}` / `{fromUpdateId, toUpdateId, where:'cold'\|'picker'\|'boundary'\|'foreground'\|'parent'}` / `{freeMb}` / `{signedAt}` |
| ~~`checkin.mood`~~ | — | **removed**: it is `behavior.signal{kind:'emotion_checkin'}` (`local_only`, 08 §1.9, 00 C8a); only the derived boolean inside the encrypted `flags/<childId>` doc may sync, and only with consent `T4b` (§5.7, §9.8) |

**Naming contract:** wire `type` is dotted and lower-case; `packages/shared/events.ts` exports
`toEngineEvent(envelope): EngineEvent` which maps to the snake_case `t` names 02 §2.1 and 08 use
(`item.answered` ↔ `item_answered`, `world.rescue` ↔ `friend_rescued`, `review.rung_passed` ↔
`review_rung_passed`, `behavior.signal` ↔ `BehaviorSignal`, `parent.input` ↔ `ParentInput`, …). One
mapping, one test that every 02 `LearnEvent.t`, every 08 `EngineEvent` kind and every event named in
10 §4.1 has exactly one wire type with a zod schema (§16 #18). Every row above is `v:1` unless marked
(`item.answered` v2, `session.ended` v2); reducers keep the upcasters.

### 5.9 Retention policy

| Data | Where | Retention | Mechanism |
|---|---|---|---|
| `learn`/`econ` events | D1 `events` | 24 months raw, then rolled into `child_skill_month` aggregates; raw archived to R2 `backups/events/<yyyy-mm>.ndjson.gz` (kept until family delete) | Cron monthly |
| `telemetry` events | D1 `events` | 30 days | Cron nightly |
| `local_only` | device only | **not persisted beyond the episode** (row deleted at `session.ended`, 00 C8a); only the 14-day aggregate counters in `fs_local_counters` survive, rows older than 14 days purged nightly on the device; raw tap telemetry stays in memory only | device purge job (part of `UpdateGate`/picker idle work) |
| AI request/response log | D1 `ai_log` | 90 days (parent can pin) | Cron nightly |
| Photos (raw) | R2 `olam-private/photos/` | deleted right after parent approval/rejection; **hard max 30 days** (R2 lifecycle rule) | worker + lifecycle |
| Extracted item JSON (approved) | packs | until parent deletes pack | — |
| Audit log | D1 `audit_log` | 24 months | Cron |
| Sessions/refresh tokens | D1 | expired + 7 days | Cron |
| Backups | R2 `backups/d1/` | 35 daily + 12 monthly | Cron |
| Worker logs (Workers Logs) | Cloudflare | platform default (7 days, *verify*) | — |

### 5.10 Parent web dashboard (H6)

- **App:** `apps/parent-web` (Vite + React 19 + react-router + TanStack Query). It shares
  `@olam/api-client`, `@olam/i18n` (parent catalog) and `@olam/ui` tokens. Hebrew RTL (`<html
  dir="rtl">`), responsive desktop-first.
- **Hosting:** Cloudflare Pages project `olam-parent`. `functions/api/[[path]].ts` forwards
  `/api/*` to the Worker via a **service binding** `API` (same origin, so cookie auth works without
  CORS):
  ```ts
  export const onRequest: PagesFunction<{ API: Fetcher }> = ({ request, env }) => {
    const url = new URL(request.url); url.pathname = url.pathname.replace(/^\/api/, '');
    return env.API.fetch(new Request(url, request));
  };
  ```
- **Auth:**
  - Email code (phase 1) or Sign in with Apple JS (phase 2).
  - The refresh token lives in the `__Host-olam_rt` cookie (`HttpOnly; Secure; SameSite=Strict;
    Path=/`). The access token is kept in memory only.
  - CSRF: SameSite=Strict + required header `X-Olam-Client: web` on mutations.
  - CSP: `default-src 'self'; img-src 'self' blob: data:; connect-src 'self'; frame-ancestors 'none'`.
- **Pages:**
  - כניסה (sign-in)
  - סקירה (per-child cards, weekly summary, "מה לעשות הערב")
  - ילד/ה › התקדמות / מיומנויות / הגדרות
  - אישורי תוכן (pack and photo-extraction approvals with side-by-side preview)
  - מכשירים (list: mode, children, last seen, app version, install-expiry; actions: rename, mode,
    children, bedtime lock, unpair, wipe; "add device" shows QR + code)
  - משפחה (guardians, invites, roles)
  - יומן AI
  - יומן פעולות (audit)
  - נתונים (export/delete)
- **Viewer role** sees only סקירה, with celebrations and no scores unless the owner enables them.
- **One login, not two (00 H6, 10 SEC-09):** Cloudflare Access in front of `olam-parent` is an
  **optional extra layer, not in the MVP plan**. If the owner ever turns it on, every guardian
  (including viewers) must be allowlisted there as well; the mobile app is unaffected because it
  calls the Worker directly, never through Pages.
- **Deployed by** CI `parent-web.yml` with `wrangler pages deploy apps/parent-web/dist --project-name
  olam-parent`. Without a custom domain it's `https://olam-parent.pages.dev`; with one,
  `https://parent.<domain>`.

---

## 6. Data: schemas, migrations, events, sync

Conventions:
- IDs are uuidv7 text.
- Times are integer epoch ms (`*_at`).
- JSON is stored as TEXT validated by zod at the boundaries.
- Booleans are INTEGER 0/1.
- All tables are `STRICT`. Both D1 and expo-sqlite are SQLite 3.4x.
- Drizzle schema files are the source; drizzle-kit generates the SQL migration files below.

### 6.1 Local database (expo-sqlite, file `olam.db`, WAL)

```sql
PRAGMA journal_mode = WAL; PRAGMA foreign_keys = ON;

-- identity snapshot (pulled from server; read-only locally except via docs)
CREATE TABLE device_self (
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL, name TEXT NOT NULL,
  type TEXT NOT NULL CHECK (type IN ('ipad','ipad_mini','iphone','web')),
  mode TEXT NOT NULL CHECK (mode IN ('shared_kids','single_child','parent_only')),
  paired_at INTEGER NOT NULL, signed_at INTEGER,            -- build signing time (free-team expiry)
  next_seq INTEGER NOT NULL DEFAULT 1,
  last_day_key TEXT,                                        -- 02 §2.1: DayKey monotonic per device (§6.4)
  clock_skew_ms INTEGER NOT NULL DEFAULT 0,                 -- device − server at the last sync (§6.4; 08 §10.5.5)
  last_server_time INTEGER, last_server_time_local INTEGER  -- §4.8 lastServerTime for the engine/economy
) STRICT;

CREATE TABLE family (
  id TEXT PRIMARY KEY, name TEXT NOT NULL, tz TEXT NOT NULL DEFAULT 'Asia/Jerusalem',
  prefs_json TEXT NOT NULL DEFAULT '{}', hlc TEXT NOT NULL
) STRICT;

CREATE TABLE guardian (
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL, display_name TEXT NOT NULL,
  role TEXT NOT NULL CHECK (role IN ('owner','coadmin','viewer')),
  gender TEXT CHECK (gender IN ('f','m')), pin_hash TEXT, pin_salt TEXT, hlc TEXT NOT NULL
) STRICT;

CREATE TABLE child (
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL, name TEXT NOT NULL, nickname TEXT,
  birthdate TEXT NOT NULL,                                  -- YYYY-MM-DD
  grammatical_gender TEXT NOT NULL CHECK (grammatical_gender IN ('f','m','n')),
  grade TEXT NOT NULL,                                      -- 08 §1.1 Grade: 'toddler'|'gan_trom'|'gan_chova'|'g1'…
  tier TEXT NOT NULL CHECK (tier IN ('K1','K2','K3')),      -- band T|G|S derived (A6)
  dominant_hand TEXT CHECK (dominant_hand IN ('left','right','unset')) DEFAULT 'unset',
  avatar_json TEXT NOT NULL DEFAULT '{}', picture_pin_hash TEXT, picture_pin_salt TEXT,
  model_epoch INTEGER NOT NULL DEFAULT 0, subject_epoch_json TEXT NOT NULL DEFAULT '{}',  -- 08 §10.5.6: raised by parent.input{reset_progress}; events are never deleted
  allowed_on_device INTEGER NOT NULL DEFAULT 1, deleted INTEGER NOT NULL DEFAULT 0, hlc TEXT NOT NULL
) STRICT;

-- generic LWW documents (settings, resume state, device settings…)
CREATE TABLE doc (
  kind TEXT NOT NULL,                 -- 'child_settings'|'child_resume'|'child_resume_bak'|'device_settings'|'family_prefs'|'reward'
                                      -- |'skill_prefs'|'interests'|'class_topics'|'family_calendar'|'guardian_prefs'
                                      -- |'neighborhood_layout' (02 §7.7, whole-doc LWW)|'flags' (ciphertext, §9.8)|'family_pubkey'
  id TEXT NOT NULL,                   -- e.g. childId / deviceId / rewardId
  body_json TEXT NOT NULL, hlc TEXT NOT NULL, deleted INTEGER NOT NULL DEFAULT 0,
  dirty INTEGER NOT NULL DEFAULT 0,   -- 1 = needs push
  PRIMARY KEY (kind, id)
) STRICT;

-- append-only event log
CREATE TABLE event (
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL, child_id TEXT, device_id TEXT NOT NULL,
  session_id TEXT, app_version TEXT NOT NULL, seq INTEGER NOT NULL, ts INTEGER NOT NULL,
  tz_offset_min INTEGER NOT NULL, type TEXT NOT NULL, v INTEGER NOT NULL,
  cls TEXT NOT NULL CHECK (cls IN ('learn','econ','telemetry','admin','local_only')),
  payload_json TEXT NOT NULL,
  origin TEXT NOT NULL CHECK (origin IN ('local','remote')),
  pushed_at INTEGER,                  -- null = not yet acknowledged by server (local origin only)
  server_seq INTEGER                  -- set when pulled/acked
) STRICT;
CREATE INDEX event_child_ts ON event(child_id, ts, device_id, seq);
CREATE INDEX event_unpushed ON event(pushed_at) WHERE pushed_at IS NULL AND origin = 'local' AND cls != 'local_only';
CREATE UNIQUE INDEX event_device_seq ON event(device_id, seq);

-- projections (derived; rebuildable from events)
CREATE TABLE app_meta (                   -- schema/version bookkeeping for the recovery ladder (§6.3)
  key TEXT PRIMARY KEY, value TEXT NOT NULL  -- 'schema_version', 'last_quick_check_at', 'restored_from_cloud_at', 'engine_version'
) STRICT;
CREATE TABLE skill_state (
  child_id TEXT NOT NULL, skill TEXT NOT NULL, elo REAL NOT NULL, n INTEGER NOT NULL,
  correct INTEGER NOT NULL, box INTEGER NOT NULL, due_at INTEGER, last_at INTEGER,
  level TEXT NOT NULL DEFAULT 'new' CHECK (level IN ('new','beginner','advancing','proficient','master')), -- 08 §1.1 LevelId
  mastered_at INTEGER, PRIMARY KEY (child_id, skill)
) STRICT;
CREATE TABLE item_state (
  child_id TEXT NOT NULL, item_id TEXT NOT NULL, elo REAL NOT NULL, seen INTEGER NOT NULL,
  last_correct INTEGER, last_at INTEGER, PRIMARY KEY (child_id, item_id)
) STRICT;
CREATE TABLE wallet (
  child_id TEXT PRIMARY KEY, coins INTEGER NOT NULL DEFAULT 0, gems INTEGER NOT NULL DEFAULT 0,
  xp INTEGER NOT NULL DEFAULT 0, level INTEGER NOT NULL DEFAULT 1
) STRICT;
CREATE TABLE inventory (
  child_id TEXT NOT NULL, item_id TEXT NOT NULL, acquired_at INTEGER NOT NULL, equipped_slot TEXT,
  PRIMARY KEY (child_id, item_id)
) STRICT;
CREATE TABLE chest (
  id TEXT PRIMARY KEY, child_id TEXT NOT NULL, rarity TEXT NOT NULL, contents_json TEXT NOT NULL,
  earned_at INTEGER NOT NULL, opened_at INTEGER
) STRICT;
CREATE TABLE collection (                 -- rescued friends, pieces, stickers, badges
  child_id TEXT NOT NULL, kind TEXT NOT NULL, item_id TEXT NOT NULL, at INTEGER NOT NULL,
  PRIMARY KEY (child_id, kind, item_id)
) STRICT;
CREATE TABLE daily_usage (
  child_id TEXT NOT NULL, day TEXT NOT NULL, minutes REAL NOT NULL, episodes INTEGER NOT NULL,
  PRIMARY KEY (child_id, day)
) STRICT;
CREATE TABLE projection_checkpoint (
  child_id TEXT PRIMARY KEY, last_event_key TEXT NOT NULL, engine_version INTEGER NOT NULL
) STRICT;
CREATE TABLE engine_checkpoint (          -- 08 §1.11: EngineState snapshots, 30 per child (≈ a month)
  child_id TEXT NOT NULL, seq INTEGER NOT NULL,           -- seq = running number per child
  last_event_key TEXT NOT NULL,                           -- "(ts, deviceId, eventId)" of the last folded event
  engine_version INTEGER NOT NULL, model_epoch INTEGER NOT NULL,
  state_blob BLOB NOT NULL,                               -- zod-validated EngineState JSON, gzip
  created_at INTEGER NOT NULL, PRIMARY KEY (child_id, seq)
) STRICT;
-- written after every session end and every 200 events; the oldest beyond 30 is deleted in the same
-- transaction. A late event (key < last_event_key), parent.input{session_reattribute|reset_progress}
-- or an engine_version bump loads the newest checkpoint older than the affected key and re-folds
-- (08 §10.5.4; S-4 ≤ 1 ms/event). Never a source of truth: `rebuildProjections` from events wins.

-- 07 §2.7 / 00 C8a: 14-day aggregate counters for the on-device flag rules. NOT an event table:
-- excluded from push, export and the event log; read only by evaluateFlags (08 §9.3)
CREATE TABLE fs_local_counters (
  child_id TEXT NOT NULL, day_key TEXT NOT NULL, rule_id TEXT NOT NULL, count INTEGER NOT NULL DEFAULT 0,
  PRIMARY KEY (child_id, day_key, rule_id)
) STRICT;   -- rows older than 14 days are purged nightly on the device

-- 12 §3.16: offline capture queue (parent devices only; JPEG files encrypted with a SecureStore key)
CREATE TABLE capture_queue (
  client_job_id TEXT PRIMARY KEY,         -- ulid; idempotency key for POST /photo-jobs
  child_ids_json TEXT NOT NULL, source_kind TEXT NOT NULL, mode TEXT NOT NULL,
  pages_json TEXT NOT NULL,               -- [{localUri, bytes, sha256}] after the §9.4 privacy pass
  created_at INTEGER NOT NULL, expires_at INTEGER NOT NULL,   -- created + 7 d
  attempts INTEGER NOT NULL DEFAULT 0, last_error TEXT
) STRICT;   -- ≤ 20 pages / 60 MB in total; FIFO upload on connectivity; cleared on unpair/family delete

-- content & assets
CREATE TABLE content_pack (
  pack_id TEXT PRIMARY KEY, active_version TEXT, pending_version TEXT,
  status TEXT NOT NULL CHECK (status IN ('active','downloading','failed','pending_app_update','retired')),
  manifest_json TEXT, updated_at INTEGER NOT NULL
) STRICT;
CREATE TABLE content_file (
  pack_id TEXT NOT NULL, version TEXT NOT NULL, path TEXT NOT NULL, sha256 TEXT NOT NULL,
  bytes INTEGER NOT NULL, local_uri TEXT, verified INTEGER NOT NULL DEFAULT 0,
  PRIMARY KEY (pack_id, version, path)
) STRICT;

-- sync & config
CREATE TABLE sync_state (
  id INTEGER PRIMARY KEY CHECK (id = 1), pull_cursor INTEGER NOT NULL DEFAULT 0,
  last_push_at INTEGER, last_pull_at INTEGER, last_error TEXT, config_version INTEGER
) STRICT;
CREATE TABLE remote_config (id INTEGER PRIMARY KEY CHECK (id = 1), etag TEXT, body_json TEXT NOT NULL) STRICT;
CREATE TABLE device_command (
  id TEXT PRIMARY KEY, type TEXT NOT NULL, args_json TEXT NOT NULL, received_at INTEGER NOT NULL,
  applied_at INTEGER, acked_at INTEGER
) STRICT;
CREATE TABLE outbox_request (                -- non-sync mutations made offline (e.g. parent settings via API)
  id TEXT PRIMARY KEY, method TEXT NOT NULL, path TEXT NOT NULL, body_json TEXT NOT NULL,
  idempotency_key TEXT NOT NULL UNIQUE, created_at INTEGER NOT NULL, attempts INTEGER NOT NULL DEFAULT 0
) STRICT;
```
Also: `expo-sqlite/kv-store` holds per-device conveniences only (last route, debug toggles). Secrets
go in `expo-secure-store`.

### 6.2 D1 database `olam-db`

```sql
-- 0001_identity.sql ---------------------------------------------------------
CREATE TABLE families (
  id TEXT PRIMARY KEY, name TEXT NOT NULL, tz TEXT NOT NULL DEFAULT 'Asia/Jerusalem',
  prefs_json TEXT NOT NULL DEFAULT '{}', prefs_hlc TEXT NOT NULL,
  -- prefs_json fields (field-level LWW doc `family_prefs`, §6.5): shabbatMode 'soft'|'full'|'off', city,
  -- travelCity?, quietHours, calmMode:boolean (00 B8b, default false), consent:{checkinNudge:boolean (T4b,
  -- default false), …}, hasChildUnder36m:boolean (00 C13; SERVER-DERIVED nightly + on every birthdate
  -- edit, read-only for devices), smallPartsOverride:{until, by} (00 C13 audited override)
  ai_daily_cap_cents INTEGER NOT NULL DEFAULT 300,      -- 12 §1.9 / 00 OPD-08: $3/day, parent range 100–1000
  ai_monthly_cap_cents INTEGER NOT NULL DEFAULT 4000,   -- $40/month, parent range 1000–10000
  ai_override_count_month INTEGER NOT NULL DEFAULT 0, ai_override_month TEXT,   -- OVERRIDE_USD $5, ≤ 3/month
  created_at INTEGER NOT NULL, deleted_at INTEGER, purge_at INTEGER   -- purge_at = deleted_at + 7 d (grace)
) STRICT;

CREATE TABLE guardians (
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL REFERENCES families(id),
  display_name TEXT NOT NULL, role TEXT NOT NULL CHECK (role IN ('owner','coadmin','viewer')),
  gender TEXT CHECK (gender IN ('f','m')), email TEXT, email_verified_at INTEGER,
  pin_hash TEXT, pin_salt TEXT, hlc TEXT NOT NULL,
  created_at INTEGER NOT NULL, removed_at INTEGER
) STRICT;
CREATE UNIQUE INDEX guardians_email ON guardians(lower(email)) WHERE removed_at IS NULL;

CREATE TABLE guardian_identities (
  provider TEXT NOT NULL CHECK (provider IN ('email','apple','passkey')),
  subject TEXT NOT NULL,                 -- lowercased email | apple sub | credential id
  guardian_id TEXT NOT NULL REFERENCES guardians(id),
  created_at INTEGER NOT NULL, last_used_at INTEGER,
  PRIMARY KEY (provider, subject)
) STRICT;

CREATE TABLE bootstrap_codes (
  code_hash TEXT PRIMARY KEY, email TEXT NOT NULL, expires_at INTEGER NOT NULL,
  used_at INTEGER, family_id TEXT
) STRICT;

CREATE TABLE email_codes (
  id TEXT PRIMARY KEY, email TEXT NOT NULL, code_hash TEXT NOT NULL, purpose TEXT NOT NULL
    CHECK (purpose IN ('signin','invite','verify')),
  invite_id TEXT, expires_at INTEGER NOT NULL, attempts INTEGER NOT NULL DEFAULT 0,
  used_at INTEGER, created_at INTEGER NOT NULL
) STRICT;
CREATE INDEX email_codes_email ON email_codes(email, created_at);

CREATE TABLE sessions (                  -- guardian refresh-token families
  id TEXT PRIMARY KEY, guardian_id TEXT NOT NULL REFERENCES guardians(id),
  family_id TEXT NOT NULL, client TEXT NOT NULL CHECK (client IN ('app','web')),
  device_id TEXT, refresh_hash TEXT NOT NULL, prev_refresh_hash TEXT,
  created_at INTEGER NOT NULL, rotated_at INTEGER NOT NULL, expires_at INTEGER NOT NULL,
  revoked_at INTEGER, revoke_reason TEXT
) STRICT;
CREATE UNIQUE INDEX sessions_refresh ON sessions(refresh_hash);

CREATE TABLE invites (
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL, role TEXT NOT NULL CHECK (role IN ('coadmin','viewer')),
  email TEXT, code_hash TEXT NOT NULL UNIQUE, created_by TEXT NOT NULL,
  expires_at INTEGER NOT NULL, used_at INTEGER, used_by TEXT, revoked_at INTEGER
) STRICT;

CREATE TABLE children (
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL REFERENCES families(id),
  name TEXT NOT NULL, nickname TEXT, birthdate TEXT NOT NULL,
  grammatical_gender TEXT NOT NULL CHECK (grammatical_gender IN ('f','m','n')),
  grade TEXT NOT NULL,                                   -- 08 §1.1 Grade enum
  tier TEXT NOT NULL CHECK (tier IN ('K1','K2','K3')),   -- profile tier (A2); band T|G|S is DERIVED by packages/shared/bands.ts (A6), never stored
  dominant_hand TEXT CHECK (dominant_hand IN ('left','right','unset')) DEFAULT 'unset',  -- H2; joystick side default (09 §3.11)
  avatar_json TEXT NOT NULL DEFAULT '{}', picture_pin_hash TEXT, picture_pin_salt TEXT,
  model_epoch INTEGER NOT NULL DEFAULT 0, subject_epoch_json TEXT NOT NULL DEFAULT '{}',   -- 08 §10.5.6 (server projection of parent.input{reset_progress})
  hlc TEXT NOT NULL, created_at INTEGER NOT NULL, deleted_at INTEGER
) STRICT;

CREATE TABLE devices (
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL REFERENCES families(id),
  name TEXT NOT NULL, type TEXT NOT NULL CHECK (type IN ('ipad','ipad_mini','iphone','web')),  -- ipad_mini = iPhone-class layout (A3)
  mode TEXT NOT NULL CHECK (mode IN ('shared_kids','single_child','parent_only')),
  token_hash TEXT NOT NULL UNIQUE, token_rotated_at INTEGER NOT NULL,
  paired_by TEXT NOT NULL, paired_at INTEGER NOT NULL,
  model TEXT, os_version TEXT, app_version TEXT, runtime_version TEXT, update_id TEXT,
  signed_at INTEGER, signing TEXT CHECK (signing IN ('free','paid','web')),
  last_seen_at INTEGER, last_sync_at INTEGER, pull_cursor INTEGER NOT NULL DEFAULT 0,
  clock_skew_ms INTEGER NOT NULL DEFAULT 0, storage_free_mb INTEGER,   -- from the /sync device block (§6.4, §4.7)
  status TEXT NOT NULL DEFAULT 'active' CHECK (status IN ('active','unpair_pending','wipe_pending','unpaired')),
  unpaired_at INTEGER
) STRICT;

CREATE TABLE device_children (
  device_id TEXT NOT NULL REFERENCES devices(id), child_id TEXT NOT NULL REFERENCES children(id),
  PRIMARY KEY (device_id, child_id)
) STRICT;

CREATE TABLE pairings (
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL, created_by TEXT NOT NULL,
  code_hash TEXT NOT NULL, nonce_hash TEXT NOT NULL,
  device_name TEXT NOT NULL, device_mode TEXT NOT NULL, allowed_child_ids_json TEXT NOT NULL,
  expires_at INTEGER NOT NULL, attempts INTEGER NOT NULL DEFAULT 0,
  claimed_at INTEGER, claimed_device_id TEXT, cancelled_at INTEGER
) STRICT;
CREATE INDEX pairings_code ON pairings(code_hash) WHERE claimed_at IS NULL;

CREATE TABLE device_commands (
  id TEXT PRIMARY KEY, device_id TEXT NOT NULL REFERENCES devices(id), family_id TEXT NOT NULL,
  type TEXT NOT NULL CHECK (type IN ('bedtime_lock','unlock','unpair','wipe','refresh_content','resync','child_remove')),
  args_json TEXT NOT NULL DEFAULT '{}', created_by TEXT NOT NULL, created_at INTEGER NOT NULL,
  delivered_at INTEGER, acked_at INTEGER, ack_status TEXT, expires_at INTEGER
) STRICT;
CREATE INDEX device_commands_pending ON device_commands(device_id) WHERE acked_at IS NULL;

CREATE TABLE audit_log (
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL, at INTEGER NOT NULL,
  actor_type TEXT NOT NULL CHECK (actor_type IN ('guardian','device','system','agent')),
  actor_id TEXT, device_id TEXT, action TEXT NOT NULL, target_type TEXT, target_id TEXT,
  diff_json TEXT, ip_hash TEXT, ua TEXT, request_id TEXT
) STRICT;
CREATE INDEX audit_family_at ON audit_log(family_id, at DESC);

-- 0002_sync.sql -------------------------------------------------------------
CREATE TABLE events (
  server_seq INTEGER PRIMARY KEY AUTOINCREMENT,      -- global pull cursor
  id TEXT NOT NULL UNIQUE,                           -- client uuidv7 (idempotency)
  family_id TEXT NOT NULL, child_id TEXT, device_id TEXT NOT NULL, session_id TEXT,
  app_version TEXT NOT NULL, seq INTEGER NOT NULL, ts INTEGER NOT NULL, tz_offset_min INTEGER NOT NULL,
  type TEXT NOT NULL, v INTEGER NOT NULL,
  cls TEXT NOT NULL CHECK (cls IN ('learn','econ','telemetry','admin')),   -- never local_only
  payload_json TEXT NOT NULL, received_at INTEGER NOT NULL
) STRICT;
CREATE INDEX events_family_seq ON events(family_id, server_seq);
CREATE INDEX events_child_ts ON events(child_id, ts);
CREATE INDEX events_cls_received ON events(cls, received_at);

CREATE TABLE docs (
  family_id TEXT NOT NULL, kind TEXT NOT NULL, id TEXT NOT NULL,
  body_json TEXT NOT NULL, hlc TEXT NOT NULL, deleted INTEGER NOT NULL DEFAULT 0,
  server_seq INTEGER NOT NULL,                     -- from doc_seq counter, for pull
  updated_by_device TEXT, updated_by_guardian TEXT, updated_at INTEGER NOT NULL,
  PRIMARY KEY (family_id, kind, id)
) STRICT;
CREATE INDEX docs_family_seq ON docs(family_id, server_seq);
CREATE TABLE counters (name TEXT PRIMARY KEY, value INTEGER NOT NULL) STRICT;  -- 'doc_seq'

CREATE TABLE idempotency (
  key TEXT PRIMARY KEY, family_id TEXT NOT NULL, route TEXT NOT NULL,
  status INTEGER NOT NULL, response_json TEXT NOT NULL, created_at INTEGER NOT NULL
) STRICT;

-- 0003_content_ai.sql ------------------------------------------------------
CREATE TABLE packs (
  pack_id TEXT NOT NULL, version TEXT NOT NULL, family_id TEXT NOT NULL,
  status TEXT NOT NULL CHECK (status IN ('draft','qa_failed','awaiting_approval','approved','rejected','retired','revoked')),  -- revoked: 12 §2.7.1 S0, §4.4 #7
  source TEXT NOT NULL CHECK (source IN ('curriculum','photo','parent_request','seasonal','family_voice')),   -- family_voice: §7.13
  manifest_key TEXT NOT NULL,                        -- R2 key in olam-content
  child_ids_json TEXT NOT NULL DEFAULT '[]', qa_report_json TEXT,
  created_at INTEGER NOT NULL, approved_by TEXT, approved_at INTEGER,
  PRIMARY KEY (pack_id, version)
) STRICT;
CREATE INDEX packs_family_status ON packs(family_id, status);

CREATE TABLE ai_jobs (
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL, kind TEXT NOT NULL
    CHECK (kind IN ('generate_pack','photo_extract','photo_build','photo_rerender_variants','parent_summary','qa_judge','nikud','fs_t2','class_topics_map')),
  status TEXT NOT NULL CHECK (status IN ('queued','deferred','submitted','running','succeeded','failed','cancelled')),
  run_after INTEGER,                      -- deferred (12 §1.9 cost guard): next 06:00 Israel, FIFO by created_at
  batch_id TEXT, input_ref TEXT, output_ref TEXT, error TEXT,
  created_by TEXT, created_at INTEGER NOT NULL, updated_at INTEGER NOT NULL
) STRICT;
CREATE INDEX ai_jobs_deferred ON ai_jobs(run_after) WHERE status = 'deferred';

CREATE TABLE ai_cost_ledger (             -- 12 §1.9: every cost-guard state transition (also an audit row)
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL, at INTEGER NOT NULL,
  scope TEXT NOT NULL CHECK (scope IN ('daily','monthly','kill')),
  from_state TEXT NOT NULL, to_state TEXT NOT NULL,   -- 'ok'|'p80'|'p100'|'override'|'off'
  spent_cents INTEGER NOT NULL, cap_cents INTEGER NOT NULL, trigger_job_id TEXT
) STRICT;

CREATE TABLE ai_log (
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL, child_id TEXT, job_id TEXT,
  route TEXT NOT NULL, model TEXT NOT NULL, input_tokens INTEGER, output_tokens INTEGER,
  cache_read_tokens INTEGER, cost_microusd INTEGER NOT NULL,
  request_json TEXT NOT NULL,    -- redacted: no images (R2 key only), names replaced by placeholders
  response_json TEXT NOT NULL, flagged INTEGER NOT NULL DEFAULT 0, pinned INTEGER NOT NULL DEFAULT 0,
  created_at INTEGER NOT NULL
) STRICT;
CREATE INDEX ai_log_family_at ON ai_log(family_id, created_at DESC);

CREATE TABLE ai_spend_daily (
  family_id TEXT NOT NULL, day TEXT NOT NULL, cost_microusd INTEGER NOT NULL DEFAULT 0,
  requests INTEGER NOT NULL DEFAULT 0, PRIMARY KEY (family_id, day)
) STRICT;

CREATE TABLE tts_cache (
  key TEXT PRIMARY KEY,           -- sha256(vendor|model|voice|settings|normalized_text|nikud)
  r2_key TEXT NOT NULL, vendor TEXT NOT NULL, voice TEXT NOT NULL, text TEXT NOT NULL,
  approved INTEGER NOT NULL DEFAULT 0, duration_ms INTEGER, created_at INTEGER NOT NULL
) STRICT;

CREATE TABLE photo_jobs (                 -- 12 §3.4/§3.15/§3.16 (one job = ≤ 4 pages of one game)
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL, client_job_id TEXT NOT NULL,   -- ulid from the device; idempotency
  uploaded_by TEXT NOT NULL, child_ids_json TEXT NOT NULL DEFAULT '[]',
  source_kind TEXT NOT NULL, mode TEXT NOT NULL CHECK (mode IN ('study_list','homework_practice','exam_page','story','number_page')),
  pages_json TEXT NOT NULL,               -- [{page, r2_key, bytes, phash}]
  status TEXT NOT NULL CHECK (status IN ('uploaded','extracting','needs_confirm','building','ready','failed','deferred')),
  extraction_ref TEXT, pack_ref TEXT, error TEXT,
  created_at INTEGER NOT NULL, confirm_by INTEGER NOT NULL,   -- created + 24 h: no confirm → job + photos deleted (12 §3.15 #16)
  delete_after INTEGER NOT NULL, deleted_at INTEGER           -- photos: right after confirm/reject, hard max 30 d
) STRICT;
CREATE UNIQUE INDEX photo_jobs_client ON photo_jobs(family_id, client_job_id);
CREATE INDEX photo_jobs_phash ON photo_jobs(family_id, created_at);   -- pHash duplicate check over the last 30 d (12 §3.15 #9)

CREATE TABLE voice_clips (                -- §7.13 parent voice clips (00 E9 parent script, 01 §14.1)
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL, guardian_id TEXT NOT NULL,
  kind TEXT NOT NULL CHECK (kind IN ('child_name','praise','goodnight','holiday','custom')), child_id TEXT,
  r2_key TEXT NOT NULL, duration_ms INTEGER NOT NULL, lufs REAL, approved INTEGER NOT NULL DEFAULT 0,
  created_at INTEGER NOT NULL, replaced_by TEXT
) STRICT;

-- 0004_reports.sql ---------------------------------------------------------
CREATE TABLE child_skill_month (
  family_id TEXT NOT NULL, child_id TEXT NOT NULL, month TEXT NOT NULL, skill TEXT NOT NULL,
  attempts INTEGER NOT NULL, correct INTEGER NOT NULL, minutes REAL NOT NULL, mastered INTEGER NOT NULL,
  PRIMARY KEY (child_id, month, skill)
) STRICT;
CREATE TABLE reports (
  id TEXT PRIMARY KEY,                       -- weekly: `${familyId}:${isoWeek}` (idempotent send, 10 §11.1)
  family_id TEXT NOT NULL, kind TEXT NOT NULL CHECK (kind IN ('weekly','monthly')),
  period_from TEXT NOT NULL, period_to TEXT NOT NULL,   -- DayKey; weekly period is rolling (previous snapshot → now)
  send_at INTEGER NOT NULL,                  -- computed per family: clamp(candleLighting − 3h, 12:00, 14:00) Friday / erev chag (10 §11.1)
  body_json TEXT NOT NULL, summary_he TEXT, sent_at INTEGER, created_at INTEGER NOT NULL
) STRICT;
CREATE INDEX reports_due ON reports(send_at) WHERE sent_at IS NULL;
CREATE TABLE coop_rooms (
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL, created_by_device TEXT NOT NULL,
  created_at INTEGER NOT NULL, ended_at INTEGER, summary_json TEXT
) STRICT;

-- 0005_parent_area.sql (tables requested by 10 §15.1; owned here, consumed by 10) ---------------
CREATE TABLE alerts (
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL, child_id TEXT, rule TEXT NOT NULL,   -- 'AL-01'…'AL-14'
  key TEXT NOT NULL,                          -- dedupe key (rule + skill/device/…)
  payload_json TEXT NOT NULL, created_at INTEGER NOT NULL,
  status TEXT NOT NULL CHECK (status IN ('queued','sent','report','expired')),
  feedback TEXT CHECK (feedback IN ('up','down')), feedback_at INTEGER
) STRICT;
CREATE INDEX alerts_family_status ON alerts(family_id, status, created_at);
CREATE UNIQUE INDEX alerts_dedupe ON alerts(family_id, child_id, rule, key);

CREATE TABLE deliveries (
  id TEXT PRIMARY KEY, ref_type TEXT NOT NULL CHECK (ref_type IN ('alert','report')), ref_id TEXT NOT NULL,
  guardian_id TEXT NOT NULL, channel TEXT NOT NULL CHECK (channel IN ('push','email','telegram','inbox')),
  scheduled_for INTEGER NOT NULL, sent_at INTEGER, error TEXT, attempts INTEGER NOT NULL DEFAULT 0
) STRICT;
CREATE INDEX deliveries_due ON deliveries(scheduled_for) WHERE sent_at IS NULL;

CREATE TABLE report_feedback (report_id TEXT NOT NULL, guardian_id TEXT NOT NULL, vote TEXT NOT NULL, at INTEGER NOT NULL, PRIMARY KEY (report_id, guardian_id)) STRICT;
CREATE TABLE observation_feedback (observation_id TEXT NOT NULL, guardian_id TEXT NOT NULL, vote TEXT NOT NULL, correction TEXT, at INTEGER NOT NULL, PRIMARY KEY (observation_id, guardian_id)) STRICT;
CREATE TABLE external_evidence (
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL, child_id TEXT NOT NULL,
  kind TEXT NOT NULL CHECK (kind IN ('paperProbe','teacherNote','transfer')), subject TEXT,
  value_json TEXT NOT NULL, created_by TEXT NOT NULL, ts INTEGER NOT NULL
) STRICT;
CREATE TABLE approvals (                      -- unified queue over packs + photos + stories + SEL + creations + share cards (10 API-18)
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL,
  kind TEXT NOT NULL CHECK (kind IN ('photo','pack','story','sel','creation','share','viewerVoice')),
  child_ids_json TEXT NOT NULL DEFAULT '[]', payload_ref TEXT NOT NULL,   -- packs.pack_id:version | photos.id | R2 key
  status TEXT NOT NULL CHECK (status IN ('pending','approved','approved_some','rejected','expired')),
  created_at INTEGER NOT NULL, decided_by TEXT, decided_at INTEGER, reasons_json TEXT
) STRICT;
CREATE INDEX approvals_family_status ON approvals(family_id, status, created_at);
CREATE TABLE share_cards (
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL, child_id TEXT NOT NULL, type TEXT NOT NULL, r2_key TEXT NOT NULL,
  consent TEXT NOT NULL CHECK (consent IN ('child','parent')), status TEXT NOT NULL, created_at INTEGER NOT NULL
) STRICT;
CREATE TABLE viewer_feed (viewer_id TEXT NOT NULL, card_id TEXT NOT NULL, reaction TEXT, at INTEGER, PRIMARY KEY (viewer_id, card_id)) STRICT;
CREATE TABLE push_tokens (                    -- phase 2 only (no APNs on a free team); kept empty in phase 1
  guardian_id TEXT NOT NULL, device_id TEXT NOT NULL, token TEXT NOT NULL, updated_at INTEGER NOT NULL,
  PRIMARY KEY (guardian_id, device_id)
) STRICT;
CREATE TABLE telegram_links (guardian_id TEXT PRIMARY KEY, chat_id TEXT NOT NULL, linked_at INTEGER NOT NULL) STRICT;   -- P2, flag `channels.telegram`
CREATE TABLE child_skill_week (               -- weekly rollup next to child_skill_month (10 M-* metrics)
  family_id TEXT NOT NULL, child_id TEXT NOT NULL, iso_week TEXT NOT NULL, skill_id TEXT NOT NULL,
  n_first INTEGER NOT NULL, c_first INTEGER NOT NULL, hints INTEGER NOT NULL,
  pae_num REAL NOT NULL, pae_den REAL NOT NULL, focused_sec INTEGER NOT NULL, level_end TEXT NOT NULL,
  PRIMARY KEY (child_id, iso_week, skill_id)
) STRICT;
```
Tables that 10 lists as "belonging to 08/02" (`SkillState`, `ReviewSchedule`, `InterestMap`,
`DispositionMap`, `FlagState`, `ClassTopics`, wallet, chests, purchases, family goal, `real_rewards`)
are **not D1 tables**: they are projections (device, rebuilt from events) and server-side projections
rebuilt nightly into `child_skill_month`/`child_skill_week` plus LWW docs (`class_topics/<childId>`,
`reward/<rewardId>`, `interests/<childId>`). `FlagState` is computed by `evaluateFlags` nightly and
stored as doc `flags/<childId>` (never in the weekly payload).

### 6.3 Migration strategy

- **D1:** drizzle-kit generates `workers/api/migrations/NNNN_name.sql`. CI applies them with
  `wrangler d1 migrations apply olam-db --remote` **before** `wrangler deploy`.
  - Migrations are **expand-only** in the same release: add columns, tables and indexes. Destructive
    changes (drop or rename) happen two releases later, after every device reports a
    `runtime_version` that no longer uses the old field.
  - Before each remote migration, CI records a Time Travel bookmark:
    `wrangler d1 time-travel info olam-db`, saved to the job summary, so a rollback is
    `wrangler d1 time-travel restore olam-db --bookmark=<id>`.
  - Local tests run the migrations against Miniflare D1 in `vitest-pool-workers`.
- **Local SQLite:** drizzle-kit `driver: 'expo'` bundles migrations into `src/db/migrations/`.
  `useMigrations(db, migrations)` runs in `boot/` before the splash hides.
  - Migrations must be **backward compatible with pulled data** and idempotent.
  - Every app migration must have a test that migrates a fixture DB from each of the last 3 schema
    versions.
  - **Recovery ladder (E-02/E-03), `src/boot/dbRecovery.ts`, total budget 4 s before the splash
    must hide (playbook rule 15):**
    1. Open `olam.db`; run `PRAGMA quick_check(1)` if `app_meta.last_quick_check_at` is > 24 h old or
       the previous run did not reach `boot_ok` (a `kv-store` flag set at the end of boot and cleared
       at the start).
    2. Run migrations inside one transaction per migration. On any throw: roll back, retry **once**
       (covers a transient lock).
    3. On a second failure, or `quick_check ≠ ok`, or `SQLITE_CORRUPT`/`SQLITE_NOTADB`: rename the
       file to `olam.db.broken-<ts>` (plus `-wal`/`-shm`), **try to salvage** unpushed events
       (`SELECT * FROM event WHERE pushed_at IS NULL` in a separate best-effort connection, ≤ 500 ms),
       create a fresh DB at the current schema, write `app_meta.restored_from_cloud_at = null`.
    4. If `device_self` could be read from the broken file or from `expo-secure-store` (the device
       token and `deviceId`/`familyId` are **also** kept in the Keychain for this purpose), mark
       `sync_state.pull_cursor = 0` and boot into `bootstrapping` (§4.8): the picker works from the
       next pull; projections rebuild after it. Salvaged events are pushed first.
    5. If neither exists → setup screen ("המכשיר צריך צירוף מחדש"), which the parent does via S3.
    6. The parent gate shows "המכשיר שוחזר מהענן ב-<date>; צ'ק-אינים מקומיים ופקודת שינה מקומית
       לא שוחזרו" until dismissed; `app.error{kind:'db_recovered', step}` is emitted. Broken files
       are deleted after 14 days or when storage is low; the agent may ask the parent to export one
       (debug screen → share sheet) for diagnosis.
    - Acceptance: a poisoned migration fixture and a corrupt-header fixture both reach the picker in
      ≤ 6 s with the server copy present; with no network they reach `bootstrapping` and still play
      bundled content.
  - Projections can be dropped and rebuilt from events (the engine version is stored in
    `projection_checkpoint`; a bump triggers a rebuild per child, at about 5k events/child/year, well
    under 1 s).
- **Content/event payload versions:** each event `type` has `v`. Reducers handle all versions (an
  upcast function per version). Removing a version requires that no un-upcast data remains (none, since
  rebuild uses upcasters).

### 6.4 Event log format and clocks

- The envelope is in §5.7. Canonical order for projections is **`(ts, deviceId, eventId)`** (00
  E11a; `eventId` is uuidv7, so ties on `ts` within one device still order by creation). It is
  deterministic on every device; two devices replaying the same log produce the identical child
  model θ (CI replay test, E11a).
- `seq` is device-local monotonic (from `device_self.next_seq` in the same transaction). It is used
  for **gap detection** on push (the server rejects a batch whose `seq` jumps, asking for a
  resend from the last acked `seq`), not for ordering.
- **HLC** for docs: `"<physical ms 13 digits>:<counter 4 digits>:<deviceId>"`, compared
  lexicographically, implemented in `packages/shared/hlc.ts`. The server clamps the physical part to
  `serverNow + 5 min` to defeat a far-future device clock.
- **`DayKey` (02 §2.1, F2 #20):** `packages/shared/dayKey.ts` exports `dayKeyOf(ts, tzOffsetMin)`
  (`YYYY-MM-DD` of the device-local day) and `nextDayKey(ev, deviceSelf)` =
  `max(dayKeyOf(ev), device_self.last_day_key)`, written into the event in the same transaction that
  bumps `device_self.last_day_key`. A clock moved backwards therefore never opens a new day and never
  re-pays a daily grant; the server stores the event's `dayKey` as sent (it is part of the payload for
  every learn/econ event) and never recomputes it (08 §10.5.5). Property test: for any sequence of
  `ts` the sequence of `dayKey` is non-decreasing per device.
- **Clock skew (08 §10.5.5, 10 FAM-04 / AC-41, F5 #7):** each `/sync` response carries `serverTime`;
  the device stores `clock_skew_ms = localNow − serverTime` on `device_self` and reports it in the
  next request's `device` block, so `devices.clock_skew_ms` is visible in `GET /devices`
  (`clockSkewMs`, `clockInaccurate: |skew| > 2 min`) and on the device card. When |skew| > 2 min the
  engine facade passes `ctx.now = lastServerTime + (localNow − lastServerTimeLocal)` (08 §10.5.5 (ה))
  so "today" decisions match the Worker's reports; event `ts` stays the raw device time (facts are
  never rewritten) and E-19's server-side `daily_usage` recompute applies when |skew| > 2 h.

### 6.5 Sync protocol

**Endpoint:** `POST /v1/sync` (device token). A single round trip does push and pull; the client loops
while `hasMore`.

Request:
```json
{
  "deviceId": "0193…", "cursor": { "events": 18234, "docs": 911 },
  "push": {
    "events": [ { "id": "0193…", "childId": "…", "sessionId": "…", "appVersion": "1.4.0+fp:7c1e",
                  "seq": 812, "ts": 1762106400000, "tzOffsetMin": 120, "type": "item.answered",
                  "v": 2, "cls": "learn", "payload": { "...": "..." } } ],
    "docs": [ { "kind": "child_settings", "id": "<childId>", "hlc": "1762106400123:0001:<dev>",
                "deleted": false, "body": { "dailyCapMin": 25 } } ],
    "commandAcks": [ { "id": "cmd_…", "status": "applied" } ]
  },
  "device": { "appVersion": "1.4.0", "runtimeVersion": "fp:7c1e…", "updateId": "…",
              "signedAt": 1761900000000, "os": "iPadOS 27.0.1", "model": "iPad13,18",
              "localNow": 1762106411000, "clockSkewMs": -1000, "storageFreeMb": 9120,
              "pendingAppUpdatePacks": 0 },
  "limit": 500
}
```
Response:
```json
{
  "accepted": { "events": ["0193…"], "docs": [{"kind":"child_settings","id":"…","hlc":"…"}] },
  "rejected": [ { "id": "0193…", "reason": "schema_invalid", "detail": "payload.rtMs" } ],
  "pull": {
    "events": [ /* envelopes from other devices of this family, cls != telemetry */ ],
    "docs": [ /* docs with server_seq > cursor.docs (incl. identity snapshot docs) */ ],
    "identity": { "family": {…}, "guardians": [...], "children": [...], "device": {…},
                  "allowedChildIds": [...] }   // included when changed since cursor
  },
  "cursor": { "events": 18734, "docs": 930 }, "hasMore": false,
  "commands": [ { "id": "cmd_…", "type": "bedtime_lock", "args": { "until": "07:00" } } ],
  "configVersion": 12, "serverTime": 1762106412000
}
```

**Rules:**
- **Idempotency:** events are `INSERT … ON CONFLICT(id) DO NOTHING`. A re-sent event counts as
  accepted. Docs are idempotent by `(kind, id, hlc)`. Non-sync mutations carry an
  `Idempotency-Key: <uuidv7>` header; the server stores the response for 24 h in `idempotency`.
- **Validation:** each event payload is validated with the zod schema for `(type, v)`. Invalid events
  are rejected individually (not the whole batch), kept locally with `last_error`, and surfaced in
  debug.
  - `child_id` must belong to the family **and** be allowed on that device. Exception: the
    `parent_only` mode, which accepts any child of the family.
  - `cls='local_only'` is rejected (defense in depth).
- **Conflicts:**
  - **Events never conflict.** They're facts, and projections are recomputed in canonical order.
    When a pull delivers events whose `ts` is older than the projection checkpoint for that child,
    the device runs `rebuildProjections(childId)`.
  - **Docs:** last-writer-wins by HLC. Two granularities:
    - **Whole-doc LWW** for `child_resume` (one writer: the device the child plays on),
      `neighborhood_layout/<childId>` (02 §7.7 / GAP-09: the losing layout's displaced items return
      to the inventory with a mailbox note, computed by the economy fold, nothing is lost) and the
      encrypted `flags/<childId>` (§9.8).
    - **Field-level LWW (E-11)** for `child_settings`, `family_prefs`, `device_settings`, `reward`,
      `class_topics`, `interests`: `body` is `{ [field]: { v, hlc } }`; the merge takes, per field,
      the entry with the higher HLC, so two parents changing different settings at the same time
      both win, and the same setting goes to the later writer. The client sends only changed fields;
      the server merges and returns the merged body plus `updated_by_guardian`, and the loser's
      screen shows "עודכן על ידי {name} לפני רגע" (10 PA-CSET). Nested objects are one field
      (e.g. `bedtime`), never deep-merged.
    - A tombstone (`deleted:true`) wins only if its HLC is higher than every field.
  - **Identity rows** (family, guardians, children, devices) are **server-authoritative**. Clients
    change them only via admin REST routes (§7), never through docs.
  - **Economy double-spend** (same child offline on two devices): purchases are never revoked, since
    C5 means no loss. A balance can go negative in the projection; the UI shows `max(0, coins)` and
    later earnings fill it silently. This is logged as `econ.overdraft` telemetry.
- **Ordering and batching:** the client pushes the oldest unpushed events first, ≤ 500 per call, and
  ≤ 1 MB body. The server responds with pull pages of ≤ 500 events.
- **Scheduling:** see §4.8, with exponential backoff on failure (cap 15 min). A full sync runs on
  "סנכרון עכשיו".
- **Clock skew:** `serverTime` is compared with the device clock. If |skew| > 2 min, telemetry is
  recorded and HLC uses `max(local, server)` physical time.

### 6.6 Restore and device replacement

- **Restore device** (after a team change, reinstall or new iPad): pair as a new device, then the
  first pull with `cursor 0` downloads all family events (paged) and docs, rebuilds projections and
  resumes. Expected size is about 15k events/year, around 6 MB of JSON. This is fine.
- **Backups:**
  - D1 Time Travel gives 30-day point-in-time restore.
  - Nightly Cron (02:10 Asia/Jerusalem) exports each table to `olam-private/backups/d1/<date>/<table>.ndjson.gz`.
    This uses paged `SELECT`. It is not `wrangler d1 export`, which is CLI-only.
  - A monthly copy is kept 12 months. `tools/ops/restore.ts` reimports into a fresh D1 for drills.
    A quarterly restore drill is in the ops checklist.

---

## 7. API specification (Worker `olam-api`, Hono)

**Base:**
- `https://api.<domain>/v1`, or `https://olam-api.<account>.workers.dev/v1` without a domain.
- The web dashboard reaches it as `/api/v1` via the Pages proxy.
- JSON only, UTF-8; request bodies are zod-validated (`@hono/zod-validator`, schemas from
  `@olam/shared`).
- Every response carries an `X-Request-Id`.

**Auth headers:**
- `Authorization: Bearer dev_…`: device token, for device scope.
- `Authorization: Bearer <JWT>`: guardian scope. On family devices, add `X-Olam-Device: dev_…`.
- `Idempotency-Key`: required on POST/PUT/PATCH/DELETE except `/sync` and `/auth/*`.

**Error shape:** see §4.11. Status codes:

| Code | Meaning |
|---|---|
| 400 | validation |
| 401 | `auth_required` / `token_expired` |
| 403 | `forbidden` (RBAC) |
| 404 | not found |
| 409 | `conflict` / `idempotency_mismatch` |
| 410 | `pairing_expired` |
| 413 | too large |
| 422 | domain rule |
| 429 | `rate_limited` (with `Retry-After`) |
| 503 | `ai_disabled` / `cap_reached` |

### 7.1 Worker skeleton

```ts
// workers/api/src/index.ts
import { Hono } from 'hono';
import { requestId } from 'hono/request-id';
import { secureHeaders } from 'hono/secure-headers';
import type { Env } from './env';
const app = new Hono<{ Bindings: Env; Variables: Ctx }>().basePath('/v1');
app.use('*', requestId(), secureHeaders(), errorHandler(), rateLimit());   // RL binding per route class
app.route('/auth', auth); app.route('/family', family); app.route('/guardians', guardians);
app.route('/invites', invites); app.route('/children', children); app.route('/devices', devices);
app.route('/pairings', pairings); app.route('/sync', sync); app.route('/config', config);
app.route('/content', content); app.route('/ai', ai); app.route('/tts', tts); app.route('/photo-jobs', photoJobs);
app.route('/voice-clips', voiceClips); app.route('/fs', fs); app.route('/notifications', notifications);
app.route('/reports', reports); app.route('/coop', coop); app.route('/audit', audit);
app.route('/admin', admin); app.route('/ops', ops);
export default {
  fetch: app.fetch,
  scheduled: (ev, env, ctx) => ctx.waitUntil(runCron(ev.cron, env)),
} satisfies ExportedHandler<Env>;
export { CoopRoom } from './do/CoopRoom';
```

```jsonc
// workers/api/wrangler.jsonc
{
  "name": "olam-api", "main": "src/index.ts", "compatibility_date": "2026-10-20",
  "compatibility_flags": ["nodejs_compat"],
  "observability": { "enabled": true, "head_sampling_rate": 1 },
  "d1_databases": [{ "binding": "DB", "database_name": "olam-db", "database_id": "<id>", "migrations_dir": "migrations" }],
  "r2_buckets": [
    { "binding": "CONTENT", "bucket_name": "olam-content" },
    { "binding": "PRIVATE", "bucket_name": "olam-private" }
  ],
  "durable_objects": { "bindings": [{ "name": "COOP", "class_name": "CoopRoom" }] },
  "migrations": [{ "tag": "v1", "new_sqlite_classes": ["CoopRoom"] }],
  "ratelimits": [
    { "name": "RL_AUTH", "namespace_id": "1001", "simple": { "limit": 10, "period": 60 } },
    { "name": "RL_PAIR", "namespace_id": "1002", "simple": { "limit": 5,  "period": 60 } },
    { "name": "RL_API",  "namespace_id": "1003", "simple": { "limit": 120, "period": 60 } },
    { "name": "RL_AI",   "namespace_id": "1004", "simple": { "limit": 20, "period": 60 } }
  ],
  "triggers": { "crons": ["10 0 * * *", "*/15 * * * *"] },
  "vars": { "APP_ENV": "production", "SIGNING_MODE": "free", "EMAIL_FROM": "olam@<domain>" }
  // secrets (wrangler secret put): ANTHROPIC_API_KEY, SESSION_SIGNING_KEY, PACK_SIGNING_KEY,
  // RESEND_API_KEY, TTS_API_KEY, R2_ACCESS_KEY_ID, R2_SECRET_ACCESS_KEY, IP_HASH_SALT, OPS_TOKEN
}
```
Cron times are UTC (`src/cron/index.ts` dispatches on `ev.cron`):
- `10 0 * * *` = 02:10/03:10 Israel: nightly backup, purge (§5.9), rollups into `child_skill_week`/
  `child_skill_month`, `evaluateFlags` (08 §9.3) → docs `flags/<childId>`, alert candidates
  AL-02..06 (10 §10.1), monthly report preparation on the 1st, and content batch **submit** (§8.1).
- `*/15 * * * *` — one 15-minute job with four idempotent steps, in this order:
  1. **Alert queue** (10 §10.2): for each guardian, bundle due `deliveries` (quiet hours 21:00–07:30,
     Shabbat/holiday window from `@hebcal/core` with the family city, Friday rule Q-3, caps CAP-1/2,
     expiry EXP-1 = 5 days → `status:'report'`). Phase 1 channels: `inbox` (always) and `email`
     (Resend); `push`/`telegram` are flag-gated for phase 2.
  2. **Weekly report** (10 §11.1, 12 §5.1): the row is created at 09:00 Israel on the send day with
     `send_at = clamp(candleLighting − 3h, 12:00, 14:00)` Israel time on **Friday**; if Friday (or
     Thursday evening) is a holiday, the first erev-chag of that week; with Shabbat mode off, Friday
     12:00. Sub-steps on the send day (each idempotent, keyed by `reports.id = family:isoWeek`):
     (a) **06:05** (first run after) — submit the monthly-strengths/weekly **Haiku Batch**
     (`ai_jobs kind:'parent_summary'`, input = `StrengthsNarrativeInput` progress facts only, 10 §11.7;
     never `flags`); (b) **11:00** — if the batch has not returned, one realtime Haiku call
     (≈ $0.002); if that fails too, the template-only report; (c) at `send_at` — render + send when
     `sent_at IS NULL`. The payload contains **no `flags[]`** (§7.10). GAP-23b: a missed window
     (Worker down) sends at the next run the same day if ≥ 3 h before candle lighting, else Sunday
     07:30 with "(באיחור)" in the subject.
  3. **Batch polling** (§8.1): `batches.retrieve` for `ai_jobs.status='submitted'`; collect when
     ended (12 §5 also says every 15 min). Skipped when there are no submitted jobs. **Deferred jobs**
     (12 §1.9): at the first run after **06:00 Israel**, `ai_jobs.status='deferred' AND run_after ≤ now`
     are re-queued FIFO (before the nightly batch of P1 is planned at 02:10 the *following* night).
  4. **Expiry and health nudges** (once per day, first run after 06:00 Israel): devices with
     `signed_at + 6d < now` and `signing='free'` → `AL-11{variant:'resign_overdue'}` inbox + email
     "צריך לחבר את האייפד למק"; devices with `pendingAppUpdatePacks > 0` → `AL-11{variant:
     'app_update_needed'}` (≤ 1 per 7 days per device); devices reporting `storageFreeMb < 500` →
     `AL-11{variant:'storage_low'}` (≤ 1 per 7 days); families with `hasChildUnder36m` to recompute
     (also nightly) and `smallPartsOverride.until` passed → override cleared + audit.
- This **replaces** the earlier `41 4 * * 0` (Sunday report) and `0 */6 * * *` triggers, per 10 §18.2.
  Workers allow up to 5 cron triggers per Worker (⟂VERIFY current limit); we use 2.
- Every cron step is wrapped: a throw in one step is logged (`cron.error`, `ops/notify` after 3
  consecutive failures of the same step) and the next step still runs.

The `ratelimits` binding and its exact config syntax should be checked against the current Workers
docs at scaffold time (*verify*).

**Environments (timing per 00 E5):**
- local `wrangler dev` (Miniflare) — from day 1; the **vertical slice (C4) runs fully local**
  (SQLite, bundled packs, JSON export via the debug screen), no Cloudflare account needed.
- `preview` = **staging** (wrangler env `preview`, a separate D1 `olam-db-preview` and buckets
  `*-preview`, used by CI e2e smoke and the agent's experiments) — deployed at **C3 with a
  placeholder family and no child data** (names `{K1}/{K2}/{K3}`).
- `production` (the family) — from MVP sprint 3 (~February 2027). Until then multi-device co-op is
  disabled (flag `coop.multiDevice=false`) and sync is off; the app runs local-only.

### 7.2 Auth and sessions

| Method & path | Auth | Request | Response |
|---|---|---|---|
| POST `/auth/bootstrap/redeem` | none (RL_AUTH) | `{code, guardianName, email, gender?}` | 201 `{accessToken, refreshToken, expiresIn:900, guardian, family}` |
| POST `/auth/email/start` | none (RL_AUTH) | `{email, purpose:'signin'\|'invite', inviteCode?}` | 202 `{}` (always) |
| POST `/auth/email/verify` | none (RL_AUTH) | `{email, code, inviteCode?, client:'app'\|'web', deviceToken?}` | 200 `{accessToken, refreshToken (app) \| Set-Cookie (web), guardian}` |
| POST `/auth/apple` *(phase 2)* | none | `{identityToken, rawNonce, fullName?, client}` | 200 same as above; 404 `identity_unknown` if not linked/invited |
| POST `/auth/link/apple` *(phase 2)* | guardian | `{identityToken, rawNonce}` | 204 |
| POST `/auth/refresh` | refresh token (body for app, cookie for web) | `{refreshToken?}` | 200 `{accessToken, refreshToken (rotated)}`; reuse → 401 + session family revoked + audit |
| POST `/auth/logout` | guardian | `{}` | 204 (revoke session) |
| GET `/auth/sessions` / DELETE `/auth/sessions/:id` | guardian | — | list / revoke own sessions (owner: all family) |

The access JWT is signed with jose, HS256 using `SESSION_SIGNING_KEY`:
`{iss:'olam', sub:guardianId, fam:familyId, role, sid, did?, iat, exp:+900}`.

### 7.3 Family, guardians, invites, children

| Method & path | Perm | Request → Response |
|---|---|---|
| GET `/family` | family.read | → `{family, guardians[], children[], devices[] (summary)}` |
| PATCH `/family` | family.update | `{name?, tz?, prefs?{shabbatMode:'soft'\|'full'\|'off', city, travelCity?, holidays, quietHours, calmMode:boolean (00 B8b; parent gate + audit), consent:{checkinNudge:boolean (T4b, OPD-27)}, smallPartsOverride?:{on:true} (00 C13: step-up + audit; server sets `until` = next nightly recompute)}}` → `{family}`. `hasChildUnder36m` is **read-only** (server-derived nightly and on every child birthdate edit; 400 if sent). Field-level LWW doc `family_prefs` (§6.5) |
| DELETE `/family` | family.delete (owner) | `{confirm:"<family name>"}` → 202, `families.deleted_at = now`, `purge_at = now + 7 d`. **During the grace (E-26, GAP-56):** every device/guardian call except `POST /family/undelete` answers **410 `family_deleting{purgeAt}`**; devices switch to offline-only with the parent-gate banner; the owner can cancel on the web dashboard (`POST /family/undelete`, audit). After `purge_at` the nightly cron deletes D1 rows, R2 prefixes (`photos/`, `voice/`, backups of this family), DO rooms; later calls → 401 → setup screen |
| GET `/guardians/me` · PATCH `/guardians/me` | self | `{displayName?, gender?}` |
| PUT `/guardians/me/pin` | self | `{pinHash, pinSalt}` (scrypt N=2^15,r=8,p=1 computed on client) → 204 |
| PATCH `/guardians/:id` | guardian.changeRole | `{role}` → `{guardian}` |
| DELETE `/guardians/:id` | guardian.remove | → 204. Revokes that guardian's sessions and refresh tokens, deletes their PIN hash from the synced set and their `guardian_prefs` doc. **Devices belong to the family, not to the guardian** (UC-007): devices the removed co-admin paired stay paired and keep playing; only the signed-in guardian session on them is cleared (E-23) |
| POST `/invites` | guardian.invite | `{role:'coadmin'\|'viewer', email?}` → 201 `{inviteId, code, link, qrPayload, expiresAt}` |
| GET `/invites` · DELETE `/invites/:id` | guardian.invite | list / revoke |
| POST `/invites/accept` | none (RL_AUTH) | `{code}` → 200 `{familyName, role, next:'email_verify'}` (the email step completes) |
| GET `/children` | family.read | → `{children[]}` (device scope: allowed only) |
| POST `/children` | child.create | `{name, nickname?, birthdate, grammaticalGender:'f'\|'m'\|'n', grade, dominantHand?:'left'\|'right', avatar}` → 201 `{child}` (server derives `tier` from birthdate+grade; `band` T/G/S is computed by `packages/shared/bands.ts` on read, A6) |
| PATCH `/children/:id` | child.update | partial → `{child}` |
| PUT `/children/:id/picture-pin` | child.settings | `{hash, salt}` or `{clear:true}` → 204 (`clear` triggers the one-time "בחרי 3 תמונות" step, §4.16) |
| POST `/children/:id/cap-bonus` | child.settings (parent gate) | `{}` → 201 `{eventId}`; writes `cap.bonus_granted` (admin event); 422 `bonus_used` on a second grant for the same `DayKey`; audit (01 DD-09, §4.16) |
| DELETE `/children/:id` | child.delete (owner) | → 202 (soft delete; events purge after 30 d; queues `child_remove{childId}` to every device that lists the child, §5.1) |

### 7.4 Devices and pairing

| Method & path | Perm / auth | Request → Response |
|---|---|---|
| POST `/devices/self` | guardian (first device of owner during onboarding) | `{name, type, mode, allowedChildIds, info}` → 201 `{deviceId, deviceToken}` |
| POST `/pairings` | device.pair | `{deviceName, mode, allowedChildIds}` → 201 `{pairingId, code:"482913", qrPayload, expiresAt}` |
| GET `/pairings/:id` | device.pair | → `{status:'open'\|'claimed'\|'expired', deviceId?}` (poll 2 s) |
| DELETE `/pairings/:id` | device.pair | cancel |
| POST `/pairings/claim` | none (RL_PAIR, 5/10 min/IP) | `{pairingId?, nonce?, code?, info:{model, os, appVersion, runtimeVersion, signedAt, signing}}` → 201 `{deviceId, deviceToken, familySnapshot}`; 410 expired; 422 `too_many_attempts` |
| GET `/devices` | family.read (guardian) | → `{devices:[{id,name,type,mode,children[],lastSeenAt,lastSyncAt,appVersion,runtimeVersion,installExpiresAt,status,clockSkewMs,clockInaccurate,storageFreeMb,pendingAppUpdatePacks,unsyncedEstimate}]}` (`unsyncedEstimate` = last reported `next_seq − 1` minus the last accepted `seq`; feeds "סנכרון: N אירועים ממתינים", 10 FAM-04 / 01 DD-37) |
| PATCH `/devices/:id` | device.update | `{name?, mode?, allowedChildIds?}` → `{device}` (removing a child queues `child_remove`) |
| POST `/devices/:id/commands` | device.command.* | `{type:'bedtime_lock'\|'unlock'\|'unpair'\|'wipe'\|'refresh_content'\|'resync'\|'child_remove', args?:{until?, childId?, force?:boolean}}` → 201 `{commandId}`. `unpair` = push-then-revoke unless `force:true` (S5) |
| GET `/devices/:id/commands` | guardian | history with ack status |
| GET `/devices/self/commands` | device | → `{commands[]}` (pending) |
| POST `/devices/self/commands/:cid/ack` | device | `{status:'applied'\|'failed', detail?}` → 204 |
| POST `/devices/self/rotate` | device | → `{deviceToken}` (old token valid 5 min) |
| DELETE `/devices/self` | device | self-unpair → 204 |

### 7.5 Sync and config

| Method & path | Auth | Notes |
|---|---|---|
| POST `/sync` | device | §6.5. Body ≤ 1 MB, ≤ 500 events. Updates `devices.last_seen_at/last_sync_at/app_version/…` |
| GET `/config` | device or guardian | ETag/If-None-Match → 304; body §4.12 |
| PATCH `/admin/config` | admin.ops | `{flags?, killSwitches?, models?, perChild?}` → new version; audit |

### 7.6 Content packs

| Method & path | Auth | Request → Response |
|---|---|---|
| GET `/content/catalog` | device | `?since=<etag>` → `{packs:[{packId, version, manifestUrl, childIds, required:boolean, status:'approved'\|'revoked'}]}`; only `approved` packs for the device's allowed children, **plus every version marked `revoked` in the last 30 days** so a device that already holds it hides it (§4.4 #7) |
| POST `/content/fact-report` | content.approve (or the parent gate from a fact card, 06 §12.6) | `{packId, factCardId, note?}` → 201; writes the admin event `fact.reported`, opens an `approvals` row of kind `pack` with `reasons_json:['fact_report']`, and the nightly batch regenerates the card from the corrected brief (12 errata pipeline, ≤ 7 days to `version+1`) |
| GET `/content/packs/:packId/:version/manifest` | device | → manifest JSON (signed) |
| GET `/content/files?pack=&ver=&paths=` | device | → `{urls:{path: presignedUrl (15 min)}}` |
| GET `/content/review` | content.approve | packs `awaiting_approval` with QA report + preview items |
| POST `/content/packs/:packId/:version/approve` | content.approve | `{childIds, note?}` → 200; signs manifest, status → approved; audit |
| POST `/content/packs/:packId/:version/reject` | content.approve | `{reason}` → 200 |
| POST `/content/requests` | content.approve | parent request: `{childId, topic, skills?, count?, interestTags?}` → 202 `{jobId}` (goes to the nightly batch unless `urgent`) |

Presigned R2 URLs are made with `aws4fetch` against the R2 S3 endpoint
`https://<account>.r2.cloudflarestorage.com` (the `R2_ACCESS_KEY_ID`/`SECRET` secrets, scoped to the
buckets):
```ts
const r2 = new AwsClient({ accessKeyId: env.R2_ACCESS_KEY_ID, secretAccessKey: env.R2_SECRET_ACCESS_KEY, service: 's3', region: 'auto' });
const url = new URL(`https://${env.CF_ACCOUNT_ID}.r2.cloudflarestorage.com/olam-content/${key}`);
url.searchParams.set('X-Amz-Expires', '900');
const signed = await r2.sign(new Request(url, { method: 'GET' }), { aws: { signQuery: true } });
return signed.url;
```

### 7.7 AI proxy endpoints (server-side only; keys never on device)

Every AI route runs these steps:
1. `killSwitches.ai` check
2. RBAC
3. RL_AI
4. **cost pre-check** (`costGuard.assertAllowed`, 12 §1.9): the route's **upper-bound estimate** from
   the `ROUTES` table (e.g. `photo.extract` $0.08/page, `companion.turn` $0.003) is added to
   `ai_spend_daily` / the month's total; the state machine in §8.4 decides (run / defer / scripted /
   503 `cap_reached{scope:'daily'|'monthly'|'kill'}`)
5. child-safety system prompt + structured output
6. post-validation in code
7. `ai_log` row
8. spend update

| Method & path | Auth | Model | Request → Response |
|---|---|---|---|
| POST `/ai/packs/generate` | content.approve (or Cron) | Opus 5.5 **Batch** + Sonnet 5.5 judge (batch) | `{childId, skills[], template?, count, interestTags?, source}` → 202 `{jobId}`; the job ends as a pack in `awaiting_approval` |
| GET `/ai/jobs/:id` | guardian | — | `{status, progress?, packRef?, error?}` |
| *(internal step of `POST /photo-jobs`, §7.9)* photo extract | job runner | Sonnet 5.5 (vision, realtime, `fallbacks:"default"`; Opus 5.5 in parallel only behind `photo.dualRoute`, 12 §3.5.1) | per page: `{extraction (schema §8.3), proposedPack}` → `photo_jobs.status = 'needs_confirm'` |
| *(internal step of `POST /photo-jobs/:id/confirm`)* photo build | job runner | templates in code; Haiku 5.5 for per-child variants/hints (12 §3.1 #10) | `{items (edited), childIds}` → pack `ready` (approved by the confirm); pages deleted |
| POST `/ai/companion/turn` | device | Haiku 5.5 realtime | `{childId, sessionId, itemId, context:{template, skill, attemptHistory[≤5], childAnswer?}, turnIndex}` → 200 `{hint:{textKey?, text, nikudText, level:1..3}, revealsAnswer:false, safety:{ok:true}}` |
| POST `/ai/reports/summary` | Cron / guardian | Haiku 5.5 (Friday batch 06:05 → realtime fallback 11:00, §7.1) | `{familyId, week}` → `{reportId}` |
| POST `/fs/t2/request` | device (K1 only, 07 §7.3) | Haiku 5.5 **batch** (nightly) + Sonnet 5.5 judge | `{childId, activityId, chips: string[]}` — **chips only** (00 C7a): every chip must belong to the closed `chipSets[activityId]` list in `packages/content/fs/chipSets.json`; any other string → 400 `chip_invalid` + audit (07 AC-32). → 202 `{jobId}` (`ai_jobs kind:'fs_t2'`); the answer lands in the parent approval queue, never live |
| GET `/ai/log` | ai.log.read | — | `?childId&from&to&cursor` → `{entries[]}` |
| POST `/ai/log/:id/pin` · POST `/ai/log/:id/flag` | ai.log.read | — | retention pin / flag for review |

**Companion turn guardrails (C7):**
- The companion is available only inside an exercise, after the child has tried ("try first"), and
  capped at **3 turns per item** and **20 per child per day**.
- The model gets **no PII**: `{name}` is substituted after generation, and gender is passed as a
  grammatical-form flag only.
- The output schema is
  `{hintText, hintNikud, level, mentionsAnswer:boolean, offTopic:boolean}`.
- Code then rejects a reply if:
  - `mentionsAnswer` is true, or the answer string or number appears in the text (normalized for
    nikud and final letters);
  - its length is over 120 chars;
  - it contains a banned praise word.
- On rejection, the app shows a scripted hint from the pack.
- The UI labels it "רמז ממכונת העזרה 🤖" (machine, distinct from the mascot).

### 7.8 TTS cache

| Method & path | Auth | Notes |
|---|---|---|
| POST `/tts/render` | admin.ops / Cron only | `{lines:[{key, text, nikud, voice, gender}]}` → 202 job. Renders via the bake-off-winning vendor, ASR round-trip QA (§8.2), stores `r2://olam-content/audio/<sha>.m4a` |
| GET `/tts/lookup?keys=` | device | → `{key: {sha, url}}` for approved audio only |
| POST `/tts/:key/approve` · `/reject` | content.approve | parent listening pass |

There is **no live TTS for children** (E9). The device only ever downloads approved, cached files.

### 7.9 Photo jobs (12 §3 is the owner of the pipeline; this is the wire contract)

| Method & path | Auth | Request → Response |
|---|---|---|
| POST `/photo-jobs` | photo.capture | multipart: `meta` JSON `{clientJobId (ulid), childIds[], sourceKind, mode, handwritten?, pages:[{sha256, bytes}]}` + ≤ 4 JPEG parts (≤ 3 MB each, long edge 2,576 px, q0.88, already face/name-blurred and EXIF-stripped on device, §9.4) → 201 `{jobId, status:'uploaded', confirmBy}`; **idempotent by `(familyId, clientJobId)`** (a retry after a crash returns the same job). 409 `duplicate_page{jobId, date}` when a page's pHash is within distance ≤ 6 of a job from the last 30 days (12 §3.15 #9; the client offers "open the existing game"). 413 over 4 pages (the client splits, 12 §3.15 #8) |
| GET `/photo-jobs/:id` | guardian | → `{status:'uploaded'\|'extracting'\|'needs_confirm'\|'building'\|'ready'\|'failed'\|'deferred', extraction?, proposedPack?, error?, confirmBy, costEstimateCents}` (poll 2 s while `extracting`/`building`; the parent also gets AL-08 when `needs_confirm`) |
| POST `/photo-jobs/:id/confirm` | photo.capture | `{items (edited), childIds, mode, notationUpdate?:{mulSign, divSign, carryRow, borrowMark} (03 §1.5, only on ✓)}` → 202 → `building` → `ready` `{packId, version}` (approved by this action; photos deleted now). Downgrading `exam_page` to another mode needs a fresh Face ID (`X-Olam-StepUp` token, 12 §3.10) |
| DELETE `/photo-jobs/:id` | photo.capture | immediate delete (R2 pages + row) |
| *(job runner)* `photo.rerender_variants` | Cron / on `PATCH /children/:id {grammaticalGender}` | renders the missing `m/f/pl` TTS lines of every photo pack of that child and bumps the pack version; devices download silently (10 AC-25 scope; GAP-52) |

Rules (12 §3.15/§3.16): a job with no confirm within **24 h** is deleted with its pages (one
PA-INBOX reminder after 2 h); `deferred` is set by the cost guard (§8.4) and re-queued at 06:00;
vendor errors keep the job `uploaded` and retry every 5 min for 1 h in the background, then
`failed{reason:'vendor'}` + "לא הצלחנו לקרוא את הדף הזה"; a refusal is never retried. The device
side keeps an offline **capture queue** (`capture_queue`, §6.1: ≤ 20 pages / 60 MB, 7-day expiry,
FIFO upload when online or at app open; cleared on unpair/family delete). The server re-checks every
page by parsing the JPEG and rejecting any `APP1`/`APP13` segment (double EXIF strip). The earlier
`/photos` presigned-PUT routes and `/ai/photo/*` are **removed**; `/ai/photo/extract` is now the
internal step the job runner performs.

### 7.10 Reports

| Method & path | Auth | Notes |
|---|---|---|
| GET `/reports/weekly?week=` | child.reports.read (viewer: celebrations only) | `{week, periodFrom, periodTo, sentAt, children:[{childId, minutes, sessions, highlights[], skills:{mastered[], practicing[], suggested[]}, eveningIdeas[]}], summaryHe}` — **no `flags[]`** (10 §18.2, 08 §9.1): flags are served only by `GET /children/:id/flags` (PA-FLAG) and inside the monthly view |
| GET `/reports/monthly?month=` · POST `/reports/:id/feedback` | child.reports.read | 10 API-25 |
| GET `/reports/children/:id/skills` | child.reports.read | per-skill state from server projections (rebuilt nightly from events) |
| GET `/reports/children/:id/overview?period=&device=` · GET `/reports/children/:id/skills/:skillId` · GET `/reports/children/:id/sessions?from&to&device=&subject=&mode=` · GET `/reports/sessions/:id` · GET `/reports/children/:id/economy?week=` | child.reports.read | 10 API-10, 11, 13, 20 (web + report rendering; the app computes the same from local SQLite via `packages/engine/metrics`, 10 AC-34) |
| GET `/reports/devices/usage?from&to&child=` | family.read (O/C) | minutes per device × child, last sync (10 API-08) |
| POST `/reports/weekly/:id/email` | O/C | re-send email |

### 7.10a Parent-area endpoints requested by 10 §15.2 (all guardian-scoped, audit-logged, zod from `@olam/shared`)

| 10 id | Method & path | Perm | Storage |
|---|---|---|---|
| API-12 | POST `/children/:id/skills/:skillId/actions` `{action:'classTaught'\|'prefer'\|'lock'\|'unlock'\|'skipChallenge'\|'notRelevant'\|'interfaceIssue', reason?}` | child.settings | admin event `parent.input{kind:'skill_action'}` (08 §1.9א) + doc `skill_prefs/<childId>` (field-level LWW) for the UI |
| API-13 | POST `/children/:id/sessions/:sid/reattribute` `{to: childId \| 'unknown' \| 'revert'}` (10 W-LOG-05 "לא {name} שיחק/ה"; AL-15 ✓/✗ card) | child.settings | admin event `parent.input{kind:'session_reattribute', sessionId, to}` (`'revert'` = 08's `to:'confirmed_self'`), synced to every device, audit-logged; each device re-folds from the checkpoint before the session (08 §10.5.3/§10.5.4) and may emit `skill.level_reverted`; wallet/collection untouched (C5) |
| API-14 | GET/PATCH `/children/:id/interests[/:topicId]`, POST `…/interests/seeds` | child.settings | doc `interests/<childId>` |
| API-15 | GET `/children/:id/observations`, POST `/observations/:id/feedback` | child.reports.read | nightly projection + `observation_feedback` |
| API-16 | GET `/children/:id/assessments`, POST `/children/:id/evidence` | child.reports.read / child.settings | projection + `external_evidence` |
| API-17 | GET `/children/:id/flags`, POST `/flags/:id/action {talkedToTeacher\|notRelevant\|snooze\|print}` | child.reports.read (**403 for device tokens and viewers**, 10 AC-18) | doc `flags/<childId>` |
| API-18 | GET `/approvals?status=`, GET `/approvals/:id`, POST `/approvals/:id/decision {approve\|reject\|approveSome, itemIds?, edits?[], reasons?[]}` | content.approve | `approvals` (unifies §7.6 pack approve/reject and §7.9 photos; those routes stay as thin aliases) |
| API-19 | GET/POST/PATCH `/rewards`, POST `/children/:id/appreciation` | rewards.manage | doc `reward/<id>`; appreciation = event `mission.confirmed`-class admin event (**no coins/gems from the parent area**, 10 AC-37) |
| API-20 | POST `/econ/refund/:purchaseId`, POST `/children/:id/sticker`, POST `/missions/:id/confirm` | child.settings | admin events `econ.refund`, `sticker.granted`, `mission.confirmed` (cls `admin`, pulled by devices and folded by the economy) |
| API-22 | GET/PUT `/children/:id/settings`, GET/PUT `/family/calendar`, POST `/family/calendar/import` | child.settings / family.update | docs `child_settings/<id>`, `family_calendar` |
| API-23 | GET/POST/PATCH/DELETE `/children/:id/class-topics`, POST `/class-topics/map` (Sonnet 5.5, structured output, parent confirms nodes) | child.settings | doc `class_topics/<childId>`; AI route rules §7.7 |
| API-24 | GET `/alerts?child=`, POST `/alerts/:id/feedback`, POST `/alerts/mute` | guardian | `alerts` |
| API-26 | POST `/notifications/push-token` *(phase 2)*, PUT `/notifications/prefs`, POST `/telegram/link` *(P2)*, POST `/telegram/webhook` *(secret, P2)* | guardian / secret | `push_tokens`, doc `guardian_prefs/<id>`, `telegram_links` |
| API-27 | POST `/share-cards`, GET `/viewer/feed`, POST `/viewer/feed/:id/react` | O/C; viewer | `share_cards`, `viewer_feed`; PNG without EXIF, K1 consent flag (10 AC-28) |
| API-28 | GET `/system/health` | O/C | aggregates §13 ops data (sync lag, backup, AI spend, expiry) |

### 7.11 Co-op

| Method & path | Auth | Notes |
|---|---|---|
| POST `/coop/rooms` | device | `{childIds, activity}` → `{roomId, joinCode (4 digits, 10 min), wsUrl}` |
| POST `/coop/rooms/join` | device (same family only) | `{joinCode}` → `{roomId, wsUrl, wsTicket (JWT 60 s)}` |
| GET `/coop/rooms/:id/ws` | `?ticket=` | WebSocket upgrade → Durable Object `CoopRoom` (§10) |

### 7.12 Audit, admin, ops

| Method & path | Auth | Notes |
|---|---|---|
| GET `/audit?from&to&action&actor&cursor` | O/C | audit log page |
| GET `/admin/costs?month=` | admin.ops | AI spend by route/model/day, plus D1/R2 usage snapshot |
| PATCH `/admin/limits` | admin.ops | `{aiDailyCapCents?: 100–1000, aiMonthlyCapCents?: 1000–10000}` (12 §1.9 ranges; 10 PA-SET); a monthly cap above the console limit the parent recorded in setup (`families.prefs.consoleCapCents`, OPD-08) is refused with 422 `above_console_cap` |
| POST `/admin/export` | data.export | `{includeFlags?: boolean}` → 202; builds a ZIP in R2: NDJSON per table (events, docs, audit, approvals…), **`engine-state.v1/<childId>.json`** per child (08 §10.5.10: `EngineState.children[childId]` without `ItemCalibration`, plus the `levels.ts` mapping), `FlagState` only when `includeFlags` is ticked (decrypted client-side first — the server holds ciphertext, so the web dashboard uploads the plaintext part into the ZIP via `POST /admin/export/:id/flags`), **never `emotion_checkin`** or `fs_local_counters` (device-only), no photos, an audio file list; the 24 h link is emailed **and shown in-app/web** (10 GAP-49: no domain in phase 1) |
| POST `/admin/backup/run` | admin.ops | run the backup now |
| POST `/ops/notify` | `OPS_TOKEN` (Mac resign script, CI) | `{kind:'resign_overdue'\|'ci_failed'\|…, deviceId?, text}` → emails O/C, rate-limited 1/kind/12 h |
| POST `/ops/bootstrap` | `OPS_TOKEN` (CI only) | `{email, role:'owner'\|'coadmin', familyId?}` → `{setupCode}` (used by `tools/ops/bootstrap-owner` and `bootstrap-guardian --role coadmin`, §5.2; 00 H2) |

### 7.13 Parent voice clips (00 E9 parent script; 01 §14.1, §19; 09 §6.6; GAP-67)

| Method & path | Auth | Request → Response |
|---|---|---|
| POST `/voice-clips` | guardian (self) | multipart `m4a` (AAC-LC mono, **≤ 10 s, ≤ 200 KB**) + `{kind:'child_name'\|'praise'\|'goodnight'\|'holiday'\|'custom', childId?, scriptLineId}` → 201 `{clipId}`; stored at R2 `olam-private/voice/<familyId>/<guardianId>/<clipId>.m4a`; the runner normalises to −18 LUFS (E9) and writes `lufs`; **re-recording the same `scriptLineId` replaces** (new row, old row `replaced_by`, old object deleted) |
| GET `/voice-clips` · DELETE `/voice-clips/:id` | guardian | list own / delete own (owner: all) |

Distribution: approved clips are assembled into the family pack **`family.voice`** (`packs.source =
'family_voice'`, one pack per family, version bumped on every change), signed and served through the
normal catalog (§4.4), so devices cache it under `documentDirectory/packs/` like any pack and the
player resolves `vo(key)` to a guardian clip first (child's name, "כל הכבוד", goodnight, holiday
greetings — placed only in the wind-down and special events, 09 §6.6). Clips never go to any AI
vendor; they are included in the export ZIP as files and deleted with the family.
| GET `/health` | none (RL_API) | `{ok, version, d1:'ok'\|'error', r2:'ok'\|'error', cron:{lastNightly, last15m}}`; used by CI smoke and the 6-hourly uptime check (§13); returns 503 when D1 is unreachable so the client treats it as "Worker down" (E-09) |

---

## 8. AI, TTS and photo pipelines

### 8.1 Content generation (nightly, Batch)

1. **Plan** (Cron 02:10): for each child, the engine computes `needs` (skills due or weak, interest
   tags, 20–30% exploration share) and turns them into `ai_jobs(kind:'generate_pack')`.
2. **Submit:** `client.messages.batches.create({requests})`, with one request per (child × skill ×
   template).
   - `model: 'claude-opus-5-5'`, `max_tokens: 16000`
   - `output_config: { effort: 'high', format: { type: 'json_schema', schema: ItemBatchSchema } }`
   - `system`: frozen style guide, curriculum excerpt, gender and nikud rules, and the child-safety
     prompt, with `cache_control: {type:'ephemeral'}` on the stable prefix
   - `custom_id = jobId:childId:skill:template`
   - No `temperature` (rejected since 2026-10-21) and no prefill. The `fallbacks` param isn't allowed
     on Batch.
3. **Collect** (every 15 min, step 3 of the 15-minute cron, §7.1; 12 §5 agrees): `batches.retrieve(id)` until `processing_status === 'ended'`, then
   `for await (const r of await client.messages.batches.results(id))`. Results are keyed by
   `custom_id`.
4. **Validate in code** (`packages/content/validate`):
   - zod schema
   - math answers recomputed, and the LLM's answer field dropped
   - distractors replaced or augmented from the code error catalogs
   - numerals agreement (lookup table)
   - gender variants present for every child-directed string
   - banned praise words
   - age-band lexicon/length limits
   - de-duplication against existing items
5. **Nikud pass:** run Dicta Nakdan or Phonikud (tools service, if a hosted API is permitted by its
   terms; otherwise offline in `tools/content-gen` on CI) and diff it against the LLM nikud.
   Disagreements go to `qa_report.nikudConflicts`.
6. **Judge:** a second batch with Sonnet 5.5 and a rubric (age fit, Hebrew grammar and nikud, math
   correctness, safety, C4 praise rules) returns `{scores, issues[]}`. Below threshold means
   regenerate once, then `qa_failed`.
7. **TTS bake** for new lines (§8.2), then a pack in `awaiting_approval`. The parent approves (bulk
   approve is allowed for curriculum packs once spot-checks pass; photo packs always need explicit
   approval). Then the manifest is signed and the pack published.

### 8.2 TTS baking (E9)

- The text is normalized as follows:
  - Phonikud nikud + stress marks for ambiguous words.
  - Numbers become words from the table.
  - Gender variants are expanded.
- Then it's rendered by the bake-off winner. The vendor client lives in `tools/tts-bake/vendors/*`,
  and the vendor ID is part of the cache key.
- **QA:** ivrit.ai Whisper round-trip (in CI, with CPU whisper.cpp) computes CER against the script.
  - CER > 8% → re-roll up to 2 takes.
  - Still failing → `needs_listen`.
- The parent's listening queue is in the dashboard.
- Loudness is normalized to **−18 LUFS integrated ±1, true peak ≤ −1 dBTP** (ffmpeg `loudnorm`; short
  clips < 3 s get the per-voice calibrated gain instead, 12 §6.9 — 00 E9 / 09 §6.7 own the number)
  and encoded as AAC-LC 64–96 kbps mono m4a (14 PQ-R-36); `content:validate` fails any clip outside
  ±1 LU.
- Stored content-addressed in R2.

### 8.3 Photo-to-game (E8, parent confirm mandatory)

```
Parent device (gate)                         Worker                              Anthropic
capture (expo-camera, or the document-scanner plugin when adopted) ≤ 4 pages → crop to the exercise
  (CMP-44; "exclude" rectangles) → ON-DEVICE PRIVACY PASS (12 §3.3, mandatory, also offline):
  1 modules/vision-privacy: face rects (+20 %) → Skia blur σ=25 + pixelate; non-family names
    (OPD-35 list) via VNRecognizeTextRequest when Hebrew is supported
  2 bake EXIF orientation into pixels, THEN re-encode with expo-image-manipulator (drops EXIF/GPS)
  3 resize long edge 2,576 px, JPEG q0.88, never grayscale (Claude's high-res tier)
  4 blur/brightness check (Laplacian < 60 or mean < 40 → "לצלם שוב?", non-blocking)
→ capture_queue row (clientJobId) → POST /photo-jobs (multipart) when online
                                              server: MIME check, APP1/APP13 segment rejection (double
                                              EXIF strip), pHash vs 30 d, R2 photos/<familyId>/<jobId>/<page>.jpg
                                              (private, lifecycle 30 d) → status extracting
                                              messages.create({model:'claude-sonnet-5-5',
                                                output_config:{format:{type:'json_schema', schema: ExtractionSchema}},
                                                fallbacks:'default', system: extraction+safety}) ─► 
                                              ◄─ extraction (items, pii_spans[], confidence, looks_like_exam)
                                              code: recompute math, drop low-confidence, V17/V18/V02,
                                              names → [NAME], notation detection → confirm line, templates
                                              → status needs_confirm (+ AL-08 to the parent)
◄── poll GET /photo-jobs/:id → review screen (SCR-14): photo ⟷ items; edit; mode; choose child(ren)
POST /photo-jobs/:id/confirm ────────────────► status building: pack (templates in code), Haiku
                                              variants/hints, validators, TTS from cache or bake queue,
                                              delete pages now → ready → catalog. Playable at once with
                                              on-screen text + bundled generic VO; item VO swaps in when baked.
```
Latency target: shutter → review screen ≤ 15 s p50. ExtractionSchema follows the
`photo_to_game_pipeline.md` schema, within Claude's structured-output limits (no numeric bounds, so
counts are enforced in code). Everything the parent sees in the review screen (`exam_page` rules,
"[שם הוסר]" chips, "לא הצלחנו לשכפל פריט N", cost estimate for > 4 pages, queue status, 24 h
auto-delete notice) is specified in 12 §3.5.6/§3.15 and 10 SCR-14.

### 8.4 Cost controls

The **values** are 12 §1.9's (00 OPD-08, J1); this section only wires them.

1. **Worker soft caps:** `families.ai_daily_cap_cents = 300` ($3.00 per family-day, resets 00:00
   Asia/Jerusalem; parent range $1–$10 in 10 PA-SET) and `ai_monthly_cap_cents = 4000` ($40, range
   $10–$100), plus per-route quotas (companion 20/child/day, photo 10/day).
2. **`costGuard` state machine** (per family, evaluated at every AI call with the route's
   upper-bound estimate, §7.7 step 4; every transition → `ai_cost_ledger` + audit):

   | State (`spent_today / daily cap`) | P1 batch | P2 photo-to-game | P3 companion | P4 summaries | Parent sees |
   |---|---|---|---|---|---|
   | `ok` < 80 % | runs | runs | runs | runs | — |
   | `p80` 80–100 % | **new jobs → `deferred`** (submitted ones continue) | runs | runs | runs | PA-HOME line "AI: 80% מהתקציב היומי" |
   | `p100` ≥ 100 % | deferred | review screen works; "צור משחק" → "המשחק יוכן מחר בבוקר (06:00)" + **"להכין עכשיו"** (Face ID step-up, `OVERRIDE_USD` $5, ≤ 3 per month, counter on `families`) | **scripted lines only** (child sees no change) | runs (negligible, not counted) | one PA-INBOX notice; email if 3 consecutive days (`AL-11{variant:'ai_cap'}`) |
   | monthly ≥ 100 % | off until the 1st | only with override | scripted | runs | notice + "להעלות תקרה?" |
   | `killSwitches.ai` | off | "לא זמין כרגע" | scripted | code-only | the owner switched it (S0 or suspected leak) |

   Deferred jobs run at the first 15-min tick after **06:00 Israel**, FIFO (§7.1 step 3).
3. **Anthropic workspace** "olam-family" with a console hard spend limit **≥ the monthly soft cap —
   default $50/month** (OPD-08: the safety net sits *above* the soft cap, never below it). The parent
   records the chosen limit at setup (`families.prefs.consoleCapCents`); `PATCH /admin/limits` refuses
   a monthly soft cap above it.
4. A kill switch in config (`killSwitches.ai`, §4.12) the owner can flip from the dashboard.
5. Alerts at 80 % (quiet PA-HOME line) and 100 % (PA-INBOX; email after 3 consecutive days).
6. A Batch-first policy for anything not interactive.
7. Integration test (12 §10): a simulated day with 40 photos crosses 80 %/100 % at exactly the
   expected item, the override is counted, the reset happens at midnight, the kill switch stops
   every route with 503.

**Cost estimate:** `(input_tokens × in_price + output × out_price) × (batch ? 0.5 : 1)`, with cache
reads at the model's cache price. It's computed from `usage` on each response and stored in
`ai_log.cost_microusd`.

---

## 9. Security and privacy

### 9.1 Secrets

| Secret | Where | Who sets it |
|---|---|---|
| `ANTHROPIC_API_KEY` (workspace "olam-family") | Worker secret | parent pastes in Cloudflare dashboard → Workers → olam-api → Settings → Variables (never in chat) |
| `SESSION_SIGNING_KEY`, `PACK_SIGNING_KEY` (Ed25519), `IP_HASH_SALT`, `OPS_TOKEN` | Worker secrets | agent generates and sets via CI (`wrangler secret put` from a one-off workflow with masked inputs) |
| `RESEND_API_KEY`, `TTS_API_KEY`, `R2_ACCESS_KEY_ID/SECRET` | Worker secrets | parent creates the vendor keys; agent sets them via the workflow or the parent pastes them in the dashboard |
| `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID`, `EXPO_TOKEN` | GitHub Actions secrets | parent |
| `OLAM_TEAM_ID`, `OPS_TOKEN` (copy) | Mac `~/.olam/env` (chmod 600) | agent via Remote Control |
| Device tokens, guardian refresh tokens | iOS Keychain (`expo-secure-store`) | runtime |

- No secret is ever in the app bundle, the repo, `EXPO_PUBLIC_*` vars or logs.
- `gitleaks` runs in CI (`ci.yml` step).
- Rotation: `SESSION_SIGNING_KEY` supports `kid` with two active keys (rotate yearly). `OPS_TOKEN` is
  rotated yearly.

### 9.2 Transport and API hardening

- TLS only. HSTS is set via `secureHeaders`.
- CORS: none on the API (the dashboard is same-origin through Pages), except `localhost` in dev.
- Body limit 1 MB (sync), 64 KB (other JSON routes), **12 MB multipart for `POST /photo-jobs`** (≤ 4 ×
  3 MB, §7.9) and 250 KB for `POST /voice-clips` (§7.13); the Worker streams multipart parts to R2.
- Rate limits (Workers Rate Limiting binding, keyed by `ip` for unauthenticated routes and by
  `deviceId`/`guardianId` otherwise): auth 10/min, pairing claim 5/min, general 120/min, AI 20/min.
  D1-backed counters handle per-code attempts: pairing 5, email code 5.
- **Brute-force math (E-07):** the 6-digit pairing code has 10⁶ space, a 10-min TTL, **5 attempts per
  pairing after which the pairing is burned** (410 `pairing_expired`, `cancelled_at` set, audit
  `pairing.burned`, the guardian's screen offers "קוד חדש"), 5/min/IP (RL_PAIR) and 5/10 min/IP in
  D1, and ≤ 3 open pairings per family at once. The success probability per pairing is ≤ 5×10⁻⁶ and
  an attacker cannot keep guessing the same code. An unauthenticated claim carries no family, so the
  per-IP limit plus the per-pairing burn rule is the whole defence; a worker test sends 6 wrong codes
  and asserts 410 + `cancelled_at`, then 20 claims from one IP and asserts 429 with `Retry-After`. QR payloads
  also carry a 128-bit nonce, so a scanned QR is not guessable at all. Email codes: 5 attempts per
  code, 5 starts/hour/email, 20/hour/IP (§5.2); PIN: 5 fails → doubling cooldown (S4).
- All D1 queries use bound parameters (drizzle). No string-built SQL.
- Every row access is **tenant-scoped**: middleware injects `familyId` from the principal, and
  repository functions require it. A test suite tries cross-family access for every route
  (`test/tenancy.spec.ts`).
- **App Attest (phase 2, paid):** `@expo/app-integrity` attests the key at pairing. Sensitive routes
  (`/sync`, `/ai/*`) then require an assertion header. Server verification (CBOR + X.509 chain to
  Apple's App Attest root + ECDSA via WebCrypto) is implemented behind flag `attest.appAttest` and
  required only once every device has attested.

### 9.3 $ caps

See §8.4: Worker daily cap, per-route quotas, Anthropic workspace limit, kill switch and alert.
Cloudflare: Workers Paid usage notifications at $10/month (parent sets them in the billing
dashboard).

### 9.4 Photos

- **On device (12 §3.3 is the owner; mandatory before any upload, also offline):**
  - The user crops to the exercise and may mark 1–3 "exclude" rectangles.
  - `modules/vision-privacy` (§2.3) blurs faces (`VNDetectFaceRectanglesRequest`, rects +20 %, Skia
    blur σ=25 + pixelate) and, where Hebrew OCR is supported, the "non-family names" list (OPD-35);
    otherwise names are handled server-side (12 V18). A page with > 25 % blurred area prompts a
    retake.
  - EXIF orientation is baked into the pixels **first**, then `expo-image-manipulator` re-encodes to
    JPEG, long edge **2,576 px, quality 0.88**, never grayscale; the re-encode writes no EXIF/GPS and
    the server rejects any `APP1`/`APP13` segment (double strip).
  - Queued pages (`capture_queue`) are stored encrypted with a SecureStore key.
  - Kids can never start a photo flow; it's parent-gated. A test uploads a fixture page with a
    synthetic face and asserts the stored object has the blurred region and no EXIF (§16 #19).
- **Storage:** private bucket `olam-private`, key `photos/<familyId>/<jobId>/<page>.jpg`, no public
  access. Access is only via 10-min presigned URLs to guardians (review screen).
- **Retention:** deleted right after confirm or reject. An R2 **lifecycle rule** deletes the
  `photos/` prefix at 30 days as a backstop, and the nightly Cron also purges `photos.delete_after`.
- **AI:** images go to Anthropic only via the API (no training on API data per Anthropic commercial
  terms; *re-verify at build*). The `ai_log` stores the R2 key, not the image, and the key is dead
  after deletion.
- Names detected in extraction are replaced before content creation. Classmates' names never reach
  packs.

### 9.5 Logs and PII

- Worker logs (`console.log` → Workers Logs) carry **no PII**: request id, route, status, latency,
  familyId (an opaque UUID), and error code. A helper `log.info()` drops known PII fields, and a lint
  bans `console.log(req.body)`.
- `ai_log` has prompts with names replaced by `{child}`. The parent can see it and it's purged at
  90 days.
- IPs are stored only as `sha256(ip + IP_HASH_SALT + yyyy-mm-dd)` in the audit log.
- Third-party analytics SDKs are not allowed (lint + review). Sentry is optional and off by default
  (§13).

### 9.6 Backups and DR

- D1 Time Travel gives 30-day PITR. A bookmark is recorded before each migration.
- Nightly NDJSON export to R2 `olam-private/backups/d1/…` (35 daily + 12 monthly).
- R2 content packs are reproducible from git + R2 audio. Audio is itself the costly artifact, so
  `olam-content/audio/` is mirrored monthly to `olam-private/backups/audio/` (dedup by sha).
- Quarterly drill: restore into `olam-db-preview` and run the integrity check script.
- Optional: a monthly encrypted export emailed to the owner (ZIP, password = owner PIN-derived;
  later).

### 9.7 Threat model summary (family scale)

| Threat | Mitigation |
|---|---|
| A sibling plays as another | picture-PIN (UX, not security) |
| A kid reaches parent settings | Face ID / PIN gate; no arithmetic gate; 2 s hidden hold |
| Offline PIN brute force from a stolen device (PIN hashes are synced so the gate works offline) | PIN is a convenience gate only; scrypt N=2^15 slows it; admin power needs the guardian session, whose refresh token is Keychain-bound to Face ID/passcode; remote unpair revokes the device |
| A stolen or lost iPad | remote unpair/wipe; tokens in Keychain (device passcode); guardian refresh needs Face ID |
| Leaked device token | per-device revoke; scope limited to the family's sync/content/companion; rate limits |
| AI cost runaway (bug loop) | hard daily cap + workspace cap + kill switch |
| Prompt injection via photo text | extraction-only schema; no tools; output validated; parent confirm before any child sees it |
| An AI reply giving the answer or something unsafe | schema flags + code checks + scripted fallback; logged for the parent |
| Lost data from free-signing expiry or a team change | server sync + restore flow; never change team without sync |

### 9.8 Sensitive derived data: encrypted to guardian devices only (00 E6a, C8a)

Red-flag states (`flags/<childId>`), context-dependent dispositions, EF calibration and the derived
`EMO_CHECKIN` boolean are computed **on the device** (08 `evaluateFlags` runs in the app from local
counters; the nightly server run only re-derives what it can from non-sensitive events) and are
stored on the server **only as ciphertext**:
- **Keys:** at S1 the owner's device generates a family **X25519 keypair** (`@noble/curves`,
  randomness from `expo-crypto getRandomValues`). The private key lives in each guardian's Keychain
  (`requireAuthentication: true`). The public key is a plain doc `family_pubkey` pulled by every
  device.
- **Transfer to the co-admin:** at S2, the owner's device shows a **QR with the private key** (or a
  24-word phrase as fallback) that the co-admin scans in "התחברות הורה במכשיר זה". The server never
  sees it. A new guardian device without the key sees "לפתוח גישה לאזורי תשומת לב — סרקו קוד מהמכשיר
  של {owner}" and all other parent features work.
- **Writing:** a kids' device encrypts the doc body with `crypto_box_seal`-style ECIES
  (X25519 + XChaCha20-Poly1305 via `@noble/ciphers`), `kind='flags'`, `enc=1`; the Worker stores and
  syncs it as an opaque doc (field-level LWW does not apply; whole-doc LWW with the device HLC).
- **Reading:** guardian devices and the web dashboard decrypt in the client (web: the private key is
  entered once per browser via QR scan with the phone camera or pasted phrase, kept in
  `sessionStorage` only).
- **Never to AI vendors:** `/ai/reports/summary` receives the progress facts only (10 §11), never
  `flags`, dispositions or check-in counters; a worker test asserts the request body schema has no
  such keys. Lost key (both guardian devices wiped): the flags are recomputed from the kids' devices'
  local 14-day counters after a new keypair is generated; older history is gone by design.
- ⟂VERIFY `@noble/ciphers` XChaCha20-Poly1305 performance on Hermes (≤ 5 ms per 4 KB doc) at C3.

### 9.9 Data at rest on a lost or stolen device (GAP-41, UC-083)

What is on disk, in which protection class, and what an attacker with the unlocked-once device gets:

| Data | Where | Protection | Note |
|---|---|---|---|
| `olam.db` (+`-wal`/`-shm`): children's names, nicknames, birthdates, avatars, grade; the learning event log; projections; settings docs | `documentDirectory` | iOS default class **`NSFileProtectionCompleteUntilFirstUserAuthentication`** (encrypted at rest with the device passcode; readable after the first unlock since boot) ⟂VERIFY by reading `FileManager.attributesOfItem(...)[.protectionKey]` in `/(parent)/debug` on day 1 of the first native build; raising the class to `Complete` would break background sync and is not planned | the device passcode is the control (parent checklist: a passcode on every family device); no flags or check-in labels are here (C8a) |
| Packs, audio cache, `family.voice` clips | `documentDirectory/packs`, `audio-cache` | same | content, not personal data (voice clips are the exception: deleted on `wipe`) |
| Capture-queue JPEGs | `documentDirectory/capture` | same **plus** AES-GCM with a key in `expo-secure-store` (12 §3.16) | pages can hold other children's names before the server pass |
| Device token, guardian refresh token, `deviceId`/`familyId` copy, capture key, family private key | Keychain (`expo-secure-store`) | `WHEN_UNLOCKED_THIS_DEVICE_ONLY`; guardian refresh + family private key with `requireAuthentication: true` (Face ID/passcode per read) | no admin power without Face ID; the device token only reaches this family's sync/content/companion routes |
| Parent PIN hashes (synced so the gate works offline) | `olam.db` | as the DB | scrypt N=2^15; a brute-forced PIN opens only the local parent gate (§9.7), never the guardian session |
| `kv-store` conveniences, `boot_marker` | `Library` | same as the DB | nothing personal |
| Nothing from `flags/*` in plaintext, no emotional labels, no raw taps | — | — | by design (§9.8, C8a) |

On loss: the parent runs **unpair `{force:true}` + wipe** from any other device or the web (S5);
the token is dead immediately, the wipe executes on the next contact and local data is deleted,
and until then the data stays behind the passcode. The parent checklist (13 §3.3) says: passcode on
every device, no shared "1234", and "המכשיר אבד" is the first button on the device card.

---

## 10. Co-op and multiplayer

### 10.1 Same device first (default)

- The iPad is split into **two touch zones** (landscape halves, or top/bottom "across the table"
  mode with the far half's HUD rotated 180°). Each zone has its own `GestureDetector`, and touches are
  routed by `absoluteX`.
- Each child has a role with different learning actions (C6): K1 "builder" solves multiplication to
  place blocks, K2 "gatherer" counts or sorts items, K3 "lap helper" taps glowing items with
  parent-present.
- The shared state (campfire level, shared resources) lives in one Zustand `useCoop` store.
  Contributions emit `coop.contribution` events attributed to each child.
- There are no scores side by side, only a shared meter and per-child private celebration (C6).

### 10.2 Multi-device (phase v1; flag `coop.multiDevice`)

**Choice: a Cloudflare Durable Object `CoopRoom` per session, using the WebSocket Hibernation API.**
- Zero native modules: RN's built-in `WebSocket` works in all builds and on web.
- It works across any network, including a parent away from home.
- Authoritative server state, with negligible cost: hibernated sockets don't accrue duration, and
  messages count as requests.
- Rejected: LAN P2P (Bonjour/UDP/TCP needs native modules such as `react-native-tcp-socket` and
  `react-native-zeroconf`, plus the local network permission and more complex host migration), and
  SharePlay/Game Center (needs Game Center friends and the paid program).
- **Latency:** Israel to the nearest Cloudflare PoP is typically 10–30 ms RTT. The design tolerates
  150 ms.

**Room lifecycle:**
1. Device A calls `POST /coop/rooms` and gets a `joinCode`.
2. Device B calls `POST /coop/rooms/join` (same-family check) and gets a `wsTicket`.
3. Both connect `wss://…/v1/coop/rooms/:id/ws?ticket=…`. The Worker validates the ticket and forwards
   to `env.COOP.idFromName(roomId)`. The DO calls `this.ctx.acceptWebSocket(ws, [deviceId])` and
   handles `webSocketMessage/webSocketClose`.
4. The room ends when all leave, after **15 min idle** (00 E5; "idle" = no `action`/`pose` from
   any member), or after 30 min max (episode cap). The DO writes `coop_rooms.summary_json`.

**Authority model:**
- Each client simulates its own avatar locally (no input lag) and sends `pose` at 10 Hz.
- The DO is authoritative for **shared** state: campfire HP and level, shared resource counts, event
  timers and "night" wave progress. It broadcasts `state` diffs at 10 Hz (only when changed) with a
  monotonic `tick`.
- Learning answers are evaluated **on the client** with `packages/engine`, and the client reports
  `action{kind, correct, power}`. The DO applies the game effect. It's a family game, so there's no
  anti-cheat beyond schema validation and rate limits.
- The DO is persisted in its SQLite storage every 5 s, so a hibernation or eviction restores the
  state.

**Message protocol** (JSON, zod-validated both sides, `v:1`):
```ts
// client → server
type C2S =
 | { t: 'hello'; v: 1; deviceId: string; childIds: string[]; appVersion: string; lastTick?: number }
 | { t: 'pose'; tick: number; childId: string; p: [number, number, number]; r: number; anim: string }
 | { t: 'action'; id: string; childId: string; kind: 'add_wood'|'light_torch'|'build'|'rescue'|'gather';
     correct: boolean; power: number; itemId?: string }
 | { t: 'emote'; childId: string; emote: 'wave'|'cheer'|'heart' }
 | { t: 'ping'; ts: number }
 | { t: 'leave' };
// server → client
type S2C =
 | { t: 'welcome'; roomId: string; tick: number; you: string; members: Member[]; state: SharedState }
 | { t: 'state'; tick: number; patch: Partial<SharedState> }           // shallow diff
 | { t: 'poses'; tick: number; poses: Record<string /*childId*/, Pose> } // 10 Hz, others only
 | { t: 'ack'; id: string; applied: boolean; reason?: 'duplicate'|'rate'|'invalid' }
 | { t: 'event'; kind: 'night_start'|'night_end'|'friend_rescued'|'campfire_level'; data: unknown }
 | { t: 'member'; change: 'join'|'leave'; member: Member }
 | { t: 'pong'; ts: number; serverTs: number }
 | { t: 'end'; reason: 'all_left'|'idle'|'cap'; summary: CoopSummary };
```
- **Idempotency:** `action.id` is uuidv7, and the DO keeps the last 500 ids.
- **Reconnect / rejoin (01 DD-34, 00 E5):** `hello{lastTick}` gets a full `welcome` back. A device
  that rejoins **within the 15-min idle window** resumes its child's role and position (the DO keeps
  `members[]` with `left_at`); meanwhile the scripted friend plays the role (≤ 1 s swap). After the
  window the room has closed and the device gets `end`.
- **Single-sourced outcomes:** `boss_defeated{participants[], eventId}` (and `friend_rescued`,
  `campfire_level`) are emitted **once by the DO**, with one uuidv7 `eventId`, to every member and
  to late rejoiners in `welcome`; each device writes the learn event with that same id, so the
  economy fold pays each child exactly once from one event (02 UC-17). When the room ends the DO
  writes `coop_rooms.summary_json` and the Worker emits a server-authored event
  `coop.room_ended{roomId, summary}` (`deviceId:'server'`, cls `learn`) that every family device
  pulls. **Campfire level-ups come only from that summary** (GAP-21): the fold applies level changes
  from `coop.room_ended`/DO events, never from client-side `coop.contribution` alone.
- **Rate limit:** ≤ 20 msgs/s per socket.
- **Offline fallback:** if the socket drops for more than 10 s, the client continues in local
  "solo co-op" mode; its `coop.contribution` events are merged afterwards as **light/logs toward the
  next meter**, not as a level-up (GAP-21). There is no loss for the child (their own receipt pays
  as usual, 02).

---

## 11. Build, release, CI

### 11.1 `app.config.ts` (abridged)

```ts
import type { ExpoConfig } from 'expo/config';
const VARIANT = (process.env.APP_VARIANT ?? 'family') as 'development' | 'family';
const SIGNING = (process.env.OLAM_SIGNING ?? 'free') as 'free' | 'paid';
const CHANNEL = process.env.OLAM_CHANNEL ?? 'family';
const SLUG = process.env.OLAM_FAMILY_SLUG ?? 'familyslug';
export default (): ExpoConfig => ({
  name: VARIANT === 'development' ? 'עולם (dev)' : 'עולם',
  slug: 'olam', scheme: 'olam', version: '0.1.0', orientation: 'default',
  runtimeVersion: { policy: 'fingerprint' },
  updates: { url: `https://u.expo.dev/${process.env.EAS_PROJECT_ID}`, checkAutomatically: 'ON_LOAD',
             fallbackToCacheTimeout: 0, requestHeaders: { 'expo-channel-name': CHANNEL } },
  ios: {
    bundleIdentifier: `com.${SLUG}.olam${VARIANT === 'development' ? '.dev' : ''}`,
    supportsTablet: true, appleTeamId: process.env.OLAM_TEAM_ID,
    infoPlist: {
      NSCameraUsageDescription: 'המצלמה משמשת לסריקת קוד צירוף ולצילום דפי עבודה (רק בהורים).',
      NSFaceIDUsageDescription: 'זיהוי פנים פותח את אזור ההורים.',
      NSMotionUsageDescription: 'חיישני התנועה משמשים למשחקי הטיה ותנועה.',
      NSMicrophoneUsageDescription: 'המיקרופון משמש לתרגול קריאה בקול (רק כשהורה מפעיל).',
      CADisableMinimumFrameDurationOnPhone: true, ITSAppUsesNonExemptEncryption: false,
      UIRequiresFullScreen: true,   // 00 A3, §0: no Split View / Slide Over / resizable Stage Manager on iPad
    },
    usesAppleSignIn: SIGNING === 'paid',
  },
  web: { output: 'static', bundler: 'metro' },
  plugins: [
    'expo-router',
    ['expo-localization', { supportsRTL: true, forcesRTL: true, supportedLocales: ['he', 'en'] }],
    ['expo-font', { fonts: [   // static weights only (09 §3.6/§15, 14 PQ-P-07); fontFamily = 'Rubik-800' etc.
      './assets/fonts/Rubik-500.ttf', './assets/fonts/Rubik-700.ttf', './assets/fonts/Rubik-800.ttf', './assets/fonts/Rubik-900.ttf',
      './assets/fonts/NotoSansHebrew-400.ttf', './assets/fonts/NotoSansHebrew-500.ttf', './assets/fonts/NotoSansHebrew-600.ttf', './assets/fonts/NotoSansHebrew-700.ttf',
      './assets/fonts/Andika-400.ttf', './assets/fonts/Andika-700.ttf',
      ...(VARIANT === 'development' ? ['./assets/fonts/Fredoka-500.ttf', './assets/fonts/Fredoka-600.ttf', './assets/fonts/FrankRuhlLibre-400.ttf', './assets/fonts/FrankRuhlLibre-500.ttf'] : []), // GATE-UX-2 candidates
    ] }],
    'expo-sqlite', 'expo-secure-store', 'expo-audio',
    ['expo-camera', { recordAudioAndroid: false }],
    ['expo-local-authentication', { faceIDPermission: 'זיהוי פנים פותח את אזור ההורים.' }],
    ['expo-notifications', { enableBackgroundRemoteNotifications: false }],
    ['expo-splash-screen', { backgroundColor: '#FFF6E5', image: './assets/splash.png', imageWidth: 200 }],
    ['expo-build-properties', { ios: { deploymentTarget: '17.0' } }],
    ...(VARIANT === 'development' ? ['expo-dev-client'] : []),
    './plugins/withSigning',            // strips paid-only entitlements when SIGNING==='free'
    './plugins/withBuildStamp',         // injects extra.buildSignedAt, git sha
  ],
  experiments: { typedRoutes: true, reactCompiler: true },
  extra: { eas: { projectId: process.env.EAS_PROJECT_ID }, signing: SIGNING, variant: VARIANT,
           apiBase: process.env.EXPO_PUBLIC_API_BASE },
});
```
The deployment target is iOS 17.0 (Rive needs 15.1+). This is to be confirmed against the family's
oldest device: if an iPad is stuck on iPadOS 16, set 16.4.

### 11.2 `eas.json` (paths 1 & 2)

```json
{
  "cli": { "version": ">= 25.3.0", "appVersionSource": "local" },
  "build": {
    "development": { "developmentClient": true, "distribution": "internal", "channel": "dev",
                     "env": { "APP_VARIANT": "development", "OLAM_SIGNING": "paid" } },
    "preview":     { "distribution": "internal", "channel": "preview",
                     "env": { "APP_VARIANT": "family", "OLAM_SIGNING": "paid", "OLAM_CHANNEL": "preview" } },
    "production":  { "distribution": "internal", "channel": "family", "autoIncrement": true,
                     "env": { "APP_VARIANT": "family", "OLAM_SIGNING": "paid" } }
  },
  "submit": { "production": { "ios": { "ascAppId": "<set in phase 2>" } } }
}
```
In path 1 these profiles are used only for `eas update` channels. Builds happen on the Mac with
`expo run:ios`, and the same env vars come from `tools/mac/env`.

### 11.3 runtimeVersion and OTA policy

- `runtimeVersion.policy = "fingerprint"`. Any native change (dependency, config plugin, app.config
  native field) changes the fingerprint, so old binaries can never receive incompatible JS.
- Channels → branches:
  - `dev` → `dev`, for the dev client.
  - `preview` → `main`, the parent iPhone canary.
  - `family` → `family`, the kids.
- Promotion: `eas update:republish --branch main --destination-branch family` (`promote.yml`, manual
  approval by the parent clicking "Run workflow"). It may also be automatic 24 h after preview
  without an S0/S1 issue.
- **Rollback:** `eas update:roll-back-to-embedded --branch family`, or republish the previous group.
  An S0 issue is rolled back immediately.
- **Native change flow:** `ota.yml` compares `npx expo-updates fingerprint:generate` (or
  `eas fingerprint:compare`) with the last native build's fingerprint stored in
  `tools/mac/last-native.json`.
  - If it differs, CI **does not** publish to `family`. It labels the PR `needs-native-build`, and
    the agent runs the Mac build (path 1) or `native-eas.yml` (path 2).
  - JS updates published for the new fingerprint wait until devices get the new binary.
- Updates are applied only at cold start, never mid-episode (§3.3).

### 11.4 Branch strategy

- Trunk-based: `main` is protected (PR required, CI green). Branches are `feat/*`, `fix/*`, `content/*`
  and `spike/*`. Squash merge.
- PRs stay under about 400 changed lines (excluding generated, baselines and content). Each PR carries
  a fingerprint label `ota-safe` or `needs-native-build`.
- Native release tags `family-YYYYMMDD-N` mark the commit used by the Mac resign routine.
- Content packs are data in R2. Their generators and validators are code in `packages/content` and go
  through PRs.

### 11.5 GitHub Actions workflows (outlines)

```yaml
# .github/workflows/ci.yml
name: ci
on: { pull_request: {}, push: { branches: [main] } }
concurrency: { group: ci-${{ github.ref }}, cancel-in-progress: true }
jobs:
  static:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: pnpm }
      - run: pnpm install --frozen-lockfile
      - run: pnpm typecheck
      - run: pnpm lint
      - run: pnpm --filter mobile exec expo install --check
      - run: pnpm --filter mobile exec expo-doctor
      - run: node tools/ci/assert-native-versions.mjs   # hermes/reanimated/worklets/skia/rive/nitro allow-list
      - run: pnpm lint:i18n && pnpm lint:assets
      - uses: gitleaks/gitleaks-action@v2
  unit:
    needs: static
    runs-on: ubuntu-24.04
    steps:
      - (checkout + setup as above)
      - run: pnpm --filter "./packages/*" test -- --coverage   # vitest + fast-check (seed printed)
      - run: pnpm --filter mobile test -- --ci                  # jest-expo + RNTL
      - run: pnpm --filter api test                             # vitest-pool-workers (D1 + migrations + tenancy)
      - run: pnpm content:validate                              # zod, nikud, gender variants, banned words, math recompute, audio LUFS
      - run: node tools/ci/routes-and-owners.mjs                # §4.1 tables: every 09 SCR has a route, every CMP an owner
      - run: node tools/ci/test-delta.mjs                       # no deleted/skipped tests without label `tests-removed`
      - run: pnpm --filter mobile exec react-compiler-healthcheck src/games src/world   # zero bail-outs (rule 7)
  mutation:                                                     # quality playbook §4: Stryker incremental on PR, full nightly
    needs: unit
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }
      - (setup)
      - run: pnpm --filter "@olam/engine" exec stryker run --incremental   # threshold: break < 80 % on engine/economy/migrations
  bundle:
    needs: static
    runs-on: ubuntu-24.04
    steps:
      - (checkout + setup)
      - run: pnpm --filter mobile exec expo export -p ios --dump-sourcemap --output-dir dist-ios
      - run: node tools/ci/bundle-budget.mjs dist-ios           # Hermes bundle ≤ 8 MB; >100 KB growth flagged
  web-visual:
    needs: static
    runs-on: ubuntu-24.04
    container: mcr.microsoft.com/playwright:v1.64.1-noble     # pinned image = stable baselines
    steps:
      - (checkout + setup)
      - run: pnpm --filter mobile exec setup-skia-web public
      - run: pnpm --filter mobile web:export                    # EXPO_PUBLIC_TEST=1 → deterministic mode
      - run: pnpm --filter mobile e2e -- --shard=${{ matrix.shard }}/4
      - uses: actions/upload-artifact@v4
        if: always()
        with: { name: playwright-${{ matrix.shard }}, path: apps/mobile/e2e/report }
    strategy: { matrix: { shard: [1, 2, 3, 4] } }
  contact-sheet:
    needs: web-visual
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/download-artifact@v4
      - run: node tools/screens/contact-sheet.mjs                 # grid PNG per screen × profile × device
      - uses: actions/upload-artifact@v4
        with: { name: contact-sheet, path: out/contact-sheet }
      - run: gh pr comment ${{ github.event.number }} --body-file out/summary.md   # PRs only
  fingerprint:
    runs-on: ubuntu-24.04
    steps:
      - (checkout + setup)
      - run: node tools/ci/fingerprint-label.mjs   # compares to tools/mac/last-native.json; sets PR label
```
```yaml
# .github/workflows/deploy-api.yml
on: { push: { branches: [main], paths: ['workers/**', 'packages/shared/**', 'packages/engine/**'] }, workflow_dispatch: {} }
jobs:
  deploy:
    runs-on: ubuntu-24.04
    environment: production
    env: { CLOUDFLARE_API_TOKEN: '${{ secrets.CLOUDFLARE_API_TOKEN }}', CLOUDFLARE_ACCOUNT_ID: '${{ secrets.CLOUDFLARE_ACCOUNT_ID }}' }
    steps:
      - (checkout + setup)
      - run: pnpm --filter api test
      - run: pnpm --filter api exec wrangler d1 time-travel info olam-db >> $GITHUB_STEP_SUMMARY
      - run: pnpm --filter api exec wrangler d1 migrations apply olam-db --remote
      - run: pnpm --filter api exec wrangler deploy
      - run: node tools/ci/smoke-api.mjs https://api.<domain>/v1/health
```
```yaml
# .github/workflows/ota.yml — JS/asset updates
on: { push: { branches: [main], paths: ['apps/mobile/**', 'packages/**'] } }
jobs:
  update:
    runs-on: ubuntu-24.04
    steps:
      - (checkout + setup)
      - uses: expo/expo-github-action@v8
        with: { eas-version: 25.3.0, token: '${{ secrets.EXPO_TOKEN }}' }
      - run: node tools/ci/fingerprint-label.mjs --fail-if-native-changed=false --out fp.json
      - run: cd apps/mobile && APP_VARIANT=family eas update --branch main --message "${{ github.event.head_commit.message }}" --non-interactive
      - run: node tools/ci/notify-native-needed.mjs fp.json   # opens/updates issue if fingerprint changed
```
```yaml
# .github/workflows/promote.yml — manual
on: { workflow_dispatch: { inputs: { group: { description: 'update group id (blank = latest on main)', required: false } } } }
jobs:
  promote:
    runs-on: ubuntu-24.04
    environment: family-release        # required reviewer = parent
    steps:
      - uses: expo/expo-github-action@v8
        with: { eas-version: 25.3.0, token: '${{ secrets.EXPO_TOKEN }}' }
      - run: cd apps/mobile && eas update:republish --branch main --destination-branch family ${{ inputs.group && format('--group {0}', inputs.group) || '' }} --non-interactive
```
Also:
- `parent-web.yml`: build `apps/parent-web`, then `wrangler pages deploy dist --project-name
  olam-parent`. On PRs it uses `--branch pr-<n>` for a preview.
- `proto-web.yml`: PATH 0 prototype deploys to `olam-proto`.
- `nightly-visual.yml`: the full Playwright matrix (all devices × profiles × screens) + Renovate
  dashboard + `expo-doctor`.
- `native-eas.yml` (phase 2): `eas build -p ios --profile preview --non-interactive`, then
  `eas build:list` → comment.
- `secrets.yml` (manual, masked inputs): `wrangler secret put` for agent-generated secrets.
- `reassure` job in `ci.yml` (UI PRs; needs `fetch-depth: 0` for the baseline) and `pr-hygiene` job
  (`tools/ci/test-delta.mjs`, `baseline-origin.mjs` — a changed Playwright baseline must come from the
  `baselines.yml` run, PR label `baselines-updated` with a reason; `bundle-budget.mjs` also fails on
  `console.*` in the Hermes bundle outside `__DEV__`).
- `baselines.yml` (`workflow_dispatch`): `playwright test --update-snapshots` inside the pinned image,
  commits to a `baselines/<pr>` branch for review.
- `rollback.yml` (`workflow_dispatch`, agent-runnable for S0): `eas update:republish --branch family
  --group <previous>` or `eas update:roll-back-to-embedded --branch family`; posts the group ids and
  pings `ops/notify`.
- `tools/ci/calendar-guard.mjs` runs first in `ota.yml` and `promote.yml`: **no OTA publish to
  `family` on Friday after 12:00 Israel, inside the Shabbat/holiday window or on a B7a quiet day**
  unless the run carries input `s0-hotfix=true` (13 §10.2).

### 11.6 Release checklist (native release)

1. CI green and the contact sheet reviewed.
2. `perf/results/<version>.md` from the device protocol (§12.5), all within budget.
3. Bump the version, tag `family-YYYYMMDD-N`, and update `tools/mac/last-native.json`.
4. Mac: run `tools/mac/resign.sh --all --force`, which installs on all devices.
5. Smoke on each device: launch, picker, one episode, sync.
6. Then OTA promote as usual.

---

## 12. Testing strategy

### 12.1 Pyramid and tools

| Layer | Weight | Tool | Runs | What |
|---|---|---|---|---|
| Pure logic | ~70% | vitest 4.1.12 + fast-check 4.10.2 | CI every PR | engine (Elo/SRS/selector/success bands), economy (no-loss invariants), projections, HLC, sync merge, numerals, gender resolution, content validators, RBAC table |
| Content | — | `pnpm content:validate` | CI on content PRs + nightly on R2 drafts | zod, nikud presence/placeholders, gender variants, banned words, math recompute, audio exists + LUFS ±1, reading-level limits |
| Components | ~15% | jest-expo 58.0.13 + RNTL 14.1.0 (+ Reanimated test setup) | CI | HText/Dir/Row RTL props, ParentGate states, Picker, PIN pad, pairing screens, error boundaries |
| Worker/API | — | vitest-pool-workers 0.23.0 (Miniflare D1/R2/DO) + Hono `testClient` | CI | every route: auth, RBAC matrix (generated cases), tenancy isolation, idempotency, pairing brute-force limits, sync conflicts, cap enforcement, DO protocol |
| Migrations | — | fixture DBs (local v-1..v-3; D1 `applyD1Migrations`) | CI on migration PRs | upgrade + rebuild projections equals golden |
| Visual/RTL/gender | ~10% | Playwright 1.64.1 in a **pinned Docker image** on the web export | CI (PR subset) + nightly (full) | §12.2 |
| E2E golden paths | <5% | **Maestro** YAML (`apps/mobile/e2e-maestro/*.yaml`, the 10 flows of 14 PQ-T-06) on the **Mac iOS Simulator** via Remote Control, against the `development` variant; weekly also on one real iPad (manual trigger; real-device Maestro is unofficial) | before each native build and before promote; weekly | onboarding (bootstrap) → add child → pair (simulated code) → picker → episode → chest → parent gate → settings → sync. Depends on: `testID` on every interactive element (09 §7 common props, §12 #17; lint `olam/require-testid` on `packages/ui` exports), the dev-only deep links `olam://dev/fixture?id=FX-..` (loads a fixture family and resets the DB) and `olam://dev/setNow?ts=` (injected clock) from §4.1, and `EXPO_PUBLIC_TEST=1` audio/Rive stubs. 11 §12.4 rows 1, 10, 12 and 13 map to flows `03`, `08`, `09`, `10` (GAP-48) |
| Real device | — | checklist §12.4 + device perf protocol §12.5 | each native build; monthly | haptics, audio latency, Rive, nikud, Face ID, camera QR, 3D fps, thermal |
| Render-perf regression | — | **Reassure 1.6.1** (`reassure --baseline` on main, compare on PR) | every UI PR | gate: no added render per interaction, ≤ +10 % duration (14 PQ-T-05) |
| Mutation | — | Stryker 10.1.1 vitest-runner, incremental on PR, full `--force` nightly | PR + nightly | **blocking:** mutation score ≥ 80 % on `packages/engine` (economy, projections, migrations) |
| Native layout shift | — | Maestro double screenshot (t+0 / t+1 s) + `pixelmatch` on the Mac simulator | before promote | gate: 0 differing pixels on static screens (14 PQ-T-06) |

**Key properties tested with fast-check:**
1. **Convergence:** for any interleaving and partition of events across N simulated devices, after
   full sync every device's projections are equal.
2. **No-loss economy:** no event sequence reduces owned inventory, collection or rescued friends.
   Balance displayed ≥ 0.
3. Chest contents are fixed at earn time: opening never changes contents.
4. Success-band selector: over simulated learners at ability θ, observed correctness stays within the
   band ±5% after warm-up.
5. HLC LWW is monotonic and commutative (whole-doc and field-level, §6.5).
6. `local_only` events are never present in any push payload.
7. **Never stuck** (quality playbook §2): the session state machine (`picker → hub → level → game →
   results → wind-down → picker`, plus `pause`, `background`, `rest`, `error`) is a typed reducer
   with a discriminated union; `fc.commands` fires random events (including `background`,
   `memoryWarning`, `interruption`, `update_ready`, `rest_window_start`) and asserts a path back to
   `picker` always exists and no transition changes coins outside `results`.
8. **Save/migration round-trip:** arbitrary old local DB fixtures (v-1..v-3) → migrate → validates
   against the latest zod schema; projections rebuilt equal golden.
9. **Replay determinism (E11a):** the same event log on two simulated devices yields identical θ,
   wallet, collection (bit-for-bit JSON).
10. **Field-level LWW convergence (E-11):** two writers, random field subsets and interleavings →
    identical merged doc on both sides.

**Edge-case suite:** every `E-xx` row in §4.18 has a test named `E-xx-*` (unit, worker, Playwright
or Maestro as the row says). `tools/ci/edge-matrix.mjs` greps the spec table and fails CI when a
row has no test.

**Replay:**
- Every bug report from the parent's "דיווח על בעיה" attaches the last 500 events, `updateId` and a
  screenshot, uploaded via `/ops/notify` with an R2 key.
- `tools/replay` reproduces the state in vitest. Each fix lands with a regression test from the
  replay.

### 12.2 Playwright visual matrix

**Setup:**
- `expo export -p web` with `EXPO_PUBLIC_TEST=1`. Deterministic mode freezes time
  (`2026-11-08T16:00:00+02:00`), seeds RNG, disables animations (Reanimated `ReduceMotion.Always`),
  pauses Rive at frame 0, and steps the R3F world to a fixed tick.
- `apps/mobile/dist` is served by `serve` with `serve.json` setting COOP/COEP headers (SQLite web).
- In the container, use `channel: 'chrome'` (local `/usr/bin/google-chrome`). In CI, use the pinned
  image `mcr.microsoft.com/playwright:v1.64.1-noble`. Baselines are **generated only in CI** with
  `--update-snapshots` via a `workflow_dispatch` job, then committed after review.
- Readiness gate per screen: `await page.waitForFunction(() => window.__olam?.ready === true)`. This
  is set when fonts are loaded, Skia `LoadSkiaWeb` has resolved (`data-skia-ready`), the world has
  rendered its first frame, and there is no pending image.
- `toHaveScreenshot({ maxDiffPixelRatio: 0.002, animations: 'disabled' })`. Plus layout-shift
  observer CLS == 0, no console errors, and `axe` checks on parent screens.

**Matrix:**

| Dimension | Values |
|---|---|
| Viewport (device descriptors; **09 §13.2 is the source**) | `ipad-11-l`/`-p` 1180×820 / 820×1180 @2 (primary), `ipad-mini-l` 1133×744 @2 (ptPerMm 6.42), `ipad-pro-13-l` 1376×1032 @2, `iphone-16-p`/`-l` 393×852 / 852×393 @3, `iphone-se-p` 375×667 @2 (small-screen edge) |
| Profile | K1 (f, grade 3, nikud partial), K2 (m, gan), K3-n, K3-f, K3-m (lap mode), family/co-op (pl) |
| Guardian addressee | guardian-f, guardian-m, plural (parent screens) |
| Direction | RTL (all), English island LTR screens, mixed bidi strings |
| Theme/size | default; child text scale 100/135/150 % (profile slider); parent screens at AX3; `prefers-reduced-motion`; world day/night |
| Network | online; offline banner states (route-intercepted); Worker-down (503 intercept) |

**Screens:**
- picker
- hub (3D first frame)
- world HUD
- each mini-game first state + success state
- English island
- collection
- shop (K1/K2)
- chest reveal (fixed frame)
- wind-down
- parent gate
- parent dashboard
- child report
- devices list + device detail
- pairing (QR + code)
- invite
- approvals
- AI log
- settings
- onboarding steps
- error boundary
- bedtime screen
- install-expiry banner

The PR subset is iPad landscape + iPhone portrait × K1, K2, K3-f × changed screens (by route
mapping). Nightly runs everything (about 1,200 shots). The **contact sheet** groups each screen
across genders side by side for the native-speaker parent review (gender errors are the #1 Hebrew UX
risk).

**Gameplay assertions:**
- `window.__olam.world.moveTo()` drives the hero along a route, then the test asserts the collectible
  count and screenshots milestones.
- `window.__olam.motion.feed()` drives tilt mini-games.
- `window.__perf` is collected as a trend artifact. It's non-blocking on web.

### 12.3 Agent's local loop (in the container)

```bash
pnpm typecheck && pnpm lint && pnpm test
pnpm web:export && pnpm --filter mobile exec serve dist -c ../serve.json -l 8081 &
pnpm --filter mobile exec playwright test --project=ipad-landscape --grep @changed
# Read screenshot PNGs back with the Read tool to self-review; then push → CI repeats in pinned Docker.
```
Local screenshots aren't committed as baselines, because the font rasterization differs from the
Docker image.

### 12.4 Real-device checklist (parent, about 15 min per native build)

| # | Check | How | Pass |
|---|---|---|---|
| 1 | Install & launch | open app on each device | launches; no "Untrusted Developer" (else trust in Settings → VPN & Device Management) |
| 2 | Nikud rendering | parent area → debug → "ניקוד" sample | marks placed correctly under/over letters, no clipping, at default and largest text |
| 3 | RTL | picker, hub HUD, settings | back arrow on right, lists right-aligned, English island left-aligned |
| 4 | Gender | switch K1/K2 | girl/boy forms correct on hub greeting & first game prompt (text + voice) |
| 5 | Audio | debug → "השהיית צליל" test: tap 10× | sounds feel instant (≤ 50 ms target; overlay shows p95) |
| 6 | Silent switch (iPhone) | flip to silent, play | narration still audible |
| 7 | Haptics (iPhone) | collect a star | light tap felt |
| 8 | Rive mascot | hub | animates, reacts to correct answer |
| 9 | 3D | hub/world, debug overlay on | fps ≥ 55, no stutter during 60 s walk |
| 10 | Face ID / PIN | open parent area | Face ID prompt; PIN fallback works |
| 11 | Camera | pair another device via QR | scan succeeds < 3 s |
| 12 | Offline | airplane mode → play an episode → back online | progress kept; syncs; other device shows it |
| 13 | Bedtime lock | from web dashboard lock the iPad | iPad shows night screen within 60 s (foreground) |
| 14 | Interruptions | Control Center, background mid-level, Bluetooth headphones on/off | resumes to "לגעת כדי להמשיך"; audio routes correctly |
| 15 | Expiry | parent area → devices | "תוקף ההתקנה" shows days left (free signing) |

### 12.5 Device performance protocol (agent via Remote Control, per native release)

This follows `performance_reliability_playbook.md` §11E:
- Release build, oldest device, rebooted, 50% brightness.
- Cold start ×5 (`xcrun xctrace record --template 'App Launch'`).
- Animation Hitches on Maestro-driven level and world runs (60 s each).
- Time Profiler on the worst scene.
- Allocations/VM Tracker with 20× enter/exit (≤ 20 MB growth).
- Metal System Trace on Skia, Rive and 3D scenes.
- A 30-min soak.
- Audio latency from 240 fps video, filmed by the parent (optional).
- Results go to `perf/results/<version>.md`. Any red item blocks the release.
- **Hermes V1 + worklets memory smoke** after every Expo/Reanimated/worklets bump: 10 min in the
  world and 2D game loop, memory must stay flat. This guards against the SDK 56/57 regression class.
  The native allow-list (`tools/ci/native-allowlist.json`, asserted by `assert-native-versions.mjs`)
  pins the **Hermes V1 floor at `260318099.0.5`** (the version RN 0.88.1 bundles in
  `sdks/.hermesv1version`; 14 PQ-R-02), reanimated 4.7.1 / worklets 0.13.0, Skia 2.14.0, Rive 0.5.5,
  nitro-modules 0.36.5.
- The full device protocol with step IDs **`PQ-DEV-01…13`** (incl. the interruption matrix, Split View
  N/A because of `UIRequiresFullScreen`, Dynamic Type at AX3, and the crash-log pull from Mac Console)
  lives in **14 §8**; this section is its summary.

---

## 13. Observability

- **Self-hosted events:** `app.error`, `app.perf` (cold start, fps p95, JS ms p95, memory peak per
  scene, `nav` route-transition ms) and `app.start` (`runtimeVersion`, `updateId`, signing, expiry,
  **`prevExitClean`**) are telemetry-class events, synced and kept 30 days. Also `app.js_stall`,
  `app.ui_stall`, `app.memory_warning`, `app.degrade{reason:'lowPower'|'thermal'|'memory', tier}`,
  `app.boot_timeout`, `app.resume_fallback` (14 PQ-R-49/PQ-B-27).
- **Crash-free without Sentry:** boot writes `boot_marker=running` to `kv-store` and sets it to
  `clean` on `AppState` background/inactive and on a normal exit path; the next `app.start` reports
  `prevExitClean = (marker === 'clean')`. Crash-free sessions = `1 − count(prevExitClean=false) /
  count(app.start)`; gate ≥ 99.8 % over 7 days per device class (Ops tab).
- **Dashboard → Ops tab (owner):**
  - errors by release
  - fps p95 by device
  - sync lag per device
  - install expiry per device
  - AI spend per day vs cap
  - batch jobs status
  - D1/R2 size
- **Worker:** Workers Logs (`observability.enabled`), structured JSON with no PII. `wrangler tail` is
  available to the agent from CI or the Mac when debugging. Cron failures go to `ops/notify` email.
- **Weekly ops line** in the Friday parent email (owner only): "הכול תקין" or a short list (failed
  syncs, expiring installs, cap hits).
- **Sentry (optional, off by default):** `@sentry/react-native` 8.28.0 is enabled only if the owner
  opts in. When on, it uses `sendDefaultPii:false`, a `beforeSend` scrubber that strips names, child
  ids and breadcrumbs with text, and is dev/preview channel only. The Developer plan is free (5k
  errors/mo). Apple crash logs via Xcode Organizer aren't available for free-signed builds, so the
  self-hosted `app.error` plus Mac Console logs are the baseline.
- **Uptime:** a GitHub Actions cron calls `/v1/health` every 6 h and emails on failure. That's enough
  for a family.
- **Hang watchdog (playbook §8):** in `development`/`preview` builds a JS heartbeat (`setInterval`
  500 ms) records gaps > 250 ms as `app.perf{jsStallMs}`; a UI-thread heartbeat via `useFrameCallback`
  writes a shared-value timestamp and a gap > 1 s is logged as `app.error{kind:'ui_hang'}`. Never in
  the `family` channel's hot path (cost), only sampled 1 session in 20.
- **Thermal and Low Power histograms** per session (§4.7) feed the Ops tab "חום/סוללה" card.

---

## 14. Cost model (monthly, USD; family of 5, ~6 devices)

| Item | Phase 1 (free signing) | Phase 2 (paid program) | Notes |
|---|---|---|---|
| Apple Developer Program | $0 | $8.25 ($99/yr) | phase 2 only |
| Cloudflare Workers Paid (Workers, D1, DO, Cron, Rate Limiting, Pages) | $5.00 | $5.00 | D1 25B reads / 50M writes included; Pages free |
| R2 (≤ 10 GB: packs, audio, backups) | $0 | $0 | $0.015/GB-mo beyond; no egress fee |
| Expo (EAS Update ≤ 1,000 MAU; Build free tier) | $0 | $0 (Starter $19 only if the build queue is too slow) | |
| GitHub (private repo, Actions Linux minutes) | $0 | $0 | ~1,500 min/mo est. within the free 2,000 |
| Domain (for email codes + dashboard URL) | ~$0.85 | ~$0.85 | ~$10/yr at Cloudflare Registrar; optional if bootstrap-only |
| Resend (email codes + weekly report) | $0 | $0 | free tier 3k/mo, 100/day |
| Claude: content gen (Opus 5.5 Batch $2/$10) + judge (Sonnet 5.5 Batch $1/$5) | $3–12 | $3–12 | ~400 items/mo |
| Claude: companion (Haiku 5.5) + summaries + photo extraction (Sonnet 5.5) | $1–4 | $1–4 | soft caps $3/day, $40/month (12 §1.9, §8.4); console limit $50 |
| TTS (bake-off winner; pre-rendered, cached forever) | $0–16 | $0–16 | falls to ~$0–3 at steady state |
| Images/3D/audio assets (production months only) | $0–40 | $0–40 | E10 pipeline; steady state ~$0–5 |
| **Total steady state** | **~$10–25** | **~$18–33** | production months +$20–50 |

---

## 15. Setup checklist

### 15.1 What the parent must do (exact steps; batched, about 2 hours total, never blocking)

1. **GitHub:**
   - Create a private repo `family-learning-game`, or approve the agent creating it.
   - Settings → Environments → create `production` and `family-release`, with yourself as the
     required reviewer on `family-release`.
2. **Cloudflare:**
   - Sign up at dash.cloudflare.com, then Workers & Pages → Plans → **Workers Paid ($5)**.
   - My Profile → API Tokens → Create Token → "Edit Cloudflare Workers" template, plus add **D1:Edit**,
     **R2 Storage:Edit**, **Cloudflare Pages:Edit** and **Account Settings:Read** (and **Zone DNS:Edit**
     if you buy a domain).
   - Copy the token and the Account ID (Workers overview sidebar) into GitHub → Settings → Secrets →
     Actions: `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID`.
   - R2 → Manage API tokens → create an "Object Read & Write" token for buckets `olam-content` and
     `olam-private` (the agent creates the buckets first). Paste the values into Workers → olam-api →
     Settings → Variables as secrets `R2_ACCESS_KEY_ID` and `R2_SECRET_ACCESS_KEY`.
   - Billing → notifications → usage alert at $10.
3. **Expo:**
   - Sign up at expo.dev (free).
   - Account settings → Access tokens → create a "CI" token → GitHub secret `EXPO_TOKEN`.
   - Tell the agent your Expo username. The agent runs `eas init`, which creates the project and its
     `EAS_PROJECT_ID`.
4. **Anthropic:**
   - console.anthropic.com → Workspaces → create "olam-family" → Limits → monthly spend limit
     **$50** (OPD-08: must be ≥ the $40 soft monthly cap of 12 §1.9; tell the agent the value you
     set — it is stored as `consoleCapCents` and bounds `PATCH /admin/limits`).
   - Create an API key in that workspace.
   - Paste it **directly** into Cloudflare → olam-api → Settings → Variables → Secret
     `ANTHROPIC_API_KEY`. Never paste it into chat.
5. **Domain + email** (recommended for email-code login):
   - Cloudflare → Domain Registration → buy e.g. `<family>-olam.com` (~$10/yr).
   - resend.com → sign up → Domains → add the domain. The agent adds DNS records via the API token
     with DNS:Edit, or you click "Auto configure" with Cloudflare.
   - Create an API key and paste it into Cloudflare secrets as `RESEND_API_KEY`.
6. **Mac** (§3.3): Xcode, the Apple ID in Xcode Accounts, Homebrew, and Claude desktop → Remote
   Control folder `~/Code/family-learning-game`. Approve the agent's connection request when asked.
7. **Each iPad/iPhone** (§3.3):
   - Cable to the Mac, then Trust, then Developer Mode on, then restart.
   - Xcode "Connect via network".
   - After the first install, Settings → General → VPN & Device Management → trust the developer.
   - For K3 on the shared iPad, Settings → Accessibility → **Guided Access** on, with a passcode.
8. **Owner sign-in:** the agent gives a one-time setup code in chat. Enter it in the app (welcome →
   "אני הורה" → "יש לי קוד הקמה").
9. **Personal inputs** (F2, batched; placeholders until then): children's names and nicknames,
   birthdates, K3 grammatical gender, school/gan, devices list with models (for perf budgets), and the
   `familyslug` for bundle IDs.
10. **TTS vendor** account per the bake-off (E9). ElevenLabs only with written approval (E8a).
11. **Cloud environment network allowlist**: the single list is **00 E13** (Cloud environment menu →
    Edit → Network access → Allowed domains). Optional: CI deploys regardless. Today the container
    cannot reach Expo/Cloudflare hosts (research log), so `expo start --tunnel`, `eas update`,
    `eas build`, `wrangler deploy/d1` run **only from GitHub Actions or the parent's Mac** — 13 §3.1's
    "tunnel from the container" line is not available until E13 is applied (see
    `docs/review/fable-issues-F6.md`). A blocked host is logged in `docs/research-log/` and listed
    under "allowlist requests" in the weekly status; never bypassed.

### 15.2 What the agent does alone

- Scaffold the monorepo, packages, lint/test/CI, `CLAUDE.md` rules, Renovate.
- Write all app, Worker, dashboard and tool code. Write the drizzle schemas and migrations.
- From CI: create D1/R2/Pages/Rate-Limit resources (`wrangler d1 create`, `wrangler r2 bucket create`,
  plus the lifecycle rule `wrangler r2 bucket lifecycle add olam-private photos-30d --prefix photos/
  --expire-days 30`, *verify syntax*). Deploy the Worker and Pages, and set generated secrets.
- `eas init`, publish OTA updates, promote with parent approval.
- On the Mac (via Remote Control, with parent approval per session): doctor, prebuild, build, install,
  the re-sign routine install (`launchctl load`), Maestro simulator runs, Instruments perf protocol.
- Content generation, validation, TTS baking and pack publishing (approval gates stay with the
  parent).
- Bootstrap the owner (setup code), seed placeholder children if names aren't provided yet.
- Weekly: Renovate PR review, `expo-doctor`, cost report, backup drill (quarterly).

### 15.3 What only the parent can do

Payments and enrollments; creating vendor accounts and keys; Apple ID sign-in in Xcode; physical
cable/Trust/Developer Mode/trust-developer steps; approving Remote Control connections; taste and
approval gates (E10, content approval, voice); naming the kids; approving `family` promotions.

---

## 16. Acceptance criteria (this architecture is "done" when…)

1. **Repo:** `pnpm i && pnpm typecheck && pnpm lint && pnpm test` pass from a clean clone on Node 22.
   CI is green on `main`, with all workflows in §11.5 present.
2. **Install:**
   - A Release build installs on a family iPad via the Mac with free signing.
   - The launchd re-sign routine reinstalls automatically before day 7 (shown by `resign-state.json`
     and in-app expiry ≥ 2 days after 8 days of elapsed time).
   - The OTA `family` channel delivers a JS change without the Mac.
3. **RTL/gender:**
   - The full Playwright matrix passes. Every child screen exists for f/m (and K3 n), and contact
     sheets have been reviewed by the parent.
   - The i18n lint has zero violations.
   - The nikud sample is approved on device.
4. **Offline-first:** airplane-mode play on device A and play on device B (same child) both survive.
   After reconnect, both devices show identical skill states, wallet and collection. The fast-check
   convergence suite passes with ≥ 1,000 runs.
5. **Identity:**
   - Bootstrap → owner → children → PIN → pair a 2nd device by QR and by code → co-admin invite via
     email code all work end-to-end on device and web dashboard.
   - The RBAC matrix tests are generated for every route × role (100% route coverage).
   - Cross-family access tests fail closed.
   - Bedtime lock reaches a foreground device in ≤ 60 s. Unpair flushes unsynced events on the
     device's next contact, then revokes (≤ 7 days, or immediately with `force:true`). Wipe deletes
     local data and acks.
6. **Audit and retention:** every admin mutation produces an audit row. Cron purges per §5.9, verified
   by a time-travel test (Miniflare fake time).
7. **Content:**
   - Packs download, verify sha256 and the Ed25519 signature, and switch atomically.
   - A tampered file is rejected and the previous version stays active.
   - Unapproved packs are never served to devices (test).
8. **AI:**
   - Every AI route enforces the §8.4 cost-guard state machine ($3/day, $40/month; 503 `cap_reached`
     at the cap; deferred/scripted/override behaviour per pipeline; the 12 §10 40-photo simulation
     passes), logs to `ai_log` with cost, and uses structured outputs.
   - Companion replies that leak the answer are blocked by code (unit tests with adversarial
     fixtures).
   - Photos are deleted after confirm and never older than 30 days (lifecycle + cron test).
9. **Performance** on the oldest family device, Release build:
   - cold start ≤ 2.5 s (target 2.0 s)
   - 2D games 60 fps (120 fps on ProMotion) with hitch ratio < 5 ms/s
   - 3D world ≥ 55 fps p95 (gate #2)
   - tap→sound p95 ≤ 50 ms (≤ 80 ms hard gate)
   - memory gates: ≤ 300 MB steady (2D) / ≤ 400 MB (3D) / ≤ 650 MB peak, ≤ 20 MB growth over 20
     cycles (targets 200 / 300 / 500 MB per §4.7; a target miss opens a ticket, a gate miss blocks)
   - Hermes bundle ≤ 8 MB
10. **Co-op:** same-device two-zone co-op works with no cross-talk of touches. Multi-device
    (flagged): two devices join a room by code, shared campfire state converges, and a reconnect
    resumes within 3 s.
11. **Security:**
    - gitleaks clean; no secrets in the bundle (CI scans `dist-ios` for key patterns).
    - Rate limits are verified by tests.
    - The device token and guardian refresh token are only in Keychain (code review + test of the
      storage adapter).
12. **Backups:** a nightly export exists in R2, and a restore drill into preview passes the integrity
    script.
13. **3D spike:** ADR 0003 records gate outcomes. If a gate fails, the fallback is chosen per §4.9.
14. **Resilience (§4.18):** every `E-01…E-24` row has a passing test (`tools/ci/edge-matrix.mjs`
    green); on device: airplane-mode first launch after re-sign reaches the picker; a poisoned
    migration fixture reaches the picker in ≤ 6 s; `UpdateGate` never reloads with a child route
    mounted; a 6th pairing attempt burns the pairing; two parents editing different settings within
    1 s both persist; a call during an episode resumes to "לגעת כדי להמשיך" with the VO replayed.
15. **Crons (10 §15.3, 00 E5):** the Friday report is sent between 12:00 and 14:00 Israel and ≥ 3 h
    before candle lighting for every Friday of תשפ"ז (test with `@hebcal/core`, Tel Aviv and
    Jerusalem, fake clock); erev-chag Fridays shift per 10 §11.1; nothing is sent twice
    (`reports.id` idempotency); the alert queue never sends inside the Shabbat window or quiet hours
    (10 AC-11); `GET /reports/weekly` has no `flags` key (schema test).
16. **Routes and owners:** `routes-and-owners.mjs` green — every 09 SCR has a route, every CMP an
    owner package.
17. **Sensitive data:** no `flags`/dispositions/check-in keys in any AI request body (worker test);
    `flags/*` docs on the server are ciphertext (test reads D1 directly); `fs_local_counters` and
    `behavior.signal{emotion_checkin}` rows never appear in a push payload or an export (test).
18. **Event registry completeness:** a `packages/shared` test asserts that every `LearnEvent.t` of 02
    §2.1, every engine event kind of 08 §1.4/§1.9/§1.9א and every event named in 10 §4.1 has exactly
    one wire type with a zod schema, that `toEngineEvent` round-trips `item.answered` v1 and v2, and
    that `session.ended.reason` only accepts 08's `SessionEndReason`.
19. **Photo privacy:** uploading a fixture page with a synthetic face and an EXIF/GPS header yields
    an R2 object with the region blurred and no `APP1`/`APP13` segment; the capture queue survives a
    kill mid-upload without creating a second job (`clientJobId`).
20. **Revocation and updates:** a pack marked `revoked` is unselectable on a connected device within
    5 min; an update ready during an episode is applied only at the next boundary/picker/foreground
    moment of §3.3 (unit test on `UpdateGate` with every moment).
21. **Clock and travel:** with the device clock 3 h off, `GET /devices` shows `clockInaccurate`,
    the engine's `ctx.now` is the corrected time and `DayKey` stays monotonic; with
    `deviceTz = America/New_York` the Shabbat window follows the family city until `travelCity` is
    set (fake clock + `@hebcal/core`).

---

## 17. Risks and open spikes

| Risk | Likelihood | Impact | Mitigation / owner |
|---|---|---|---|
| R3F v9 + expo-gl broken on RN 0.88 New Arch | M | H | Gate #1 on day 1. WebView fallback. Pin three/R3F. |
| Free-signing friction (7-day expiry, devices asleep, Wi-Fi install fails) | H | M | daily routine + email nudge + in-app expiry; upgrade to paid when it's annoying (ADR) |
| Team change wipes local data | L | H | sync-before-migrate checklist; restore flow (§6.6) |
| expo-sqlite web (alpha) flakiness in Playwright | M | M | memory adapter fallback for visual tests (`EXPO_PUBLIC_DB=memory`), real SQLite only in unit/device |
| expo-audio latency > 50 ms | M | M | pooled players; `react-native-audio-api` swap behind the interface |
| Hermes V1 + worklets memory regressions on SDK bumps | M | H | allow-list versions; memory smoke per bump; don't upgrade SDKs mid-season |
| RN RTL regressions (ScrollView/FlatList invisible under forceRTL, iOS text alignment) | M | M | R-1/R-3 rules (no reliance on I18nManager); visual + device checks per release |
| Rive Nitro + Xcode 27 dynamic frameworks crash on iOS ≤ 17 | L | M | static linkage (default); deployment target ≥ 17 |
| SIWA unavailable until paid | certain | L | email code + bootstrap; flag-flip later |
| Email deliverability (codes in spam) | M | L | verified domain (SPF/DKIM via Resend); bootstrap/backup codes; device pairing doesn't need email |
| Hebrew `Intl.PluralRules` gaps on Hermes | L | L | boot self-test + polyfill |
| Container network blocks (Expo/Cloudflare) | certain now | M | CI as the deployer; parent may allowlist (§15.1 #11) |

---

## 18. Proposed change to 00

> These keep 00's intent. Each is required by the free-signing reality (E1a parent answer) or by
> evidence gathered here.

1. **E1:** change "Built in the cloud via EAS Build … No Mac required" to: "Phase 1: built locally
   on the parent's Mac (expo run:ios) with free personal-team signing. Phase 2 (after the paid
   program): EAS Build. OTA via EAS Update in both phases (channels `preview` → `family`)." E1 and
   E1a currently contradict each other.
2. **E7 / H2 / H6 — Sign in with Apple:** it is **not available on a free personal team**: no
   `com.apple.developer.applesignin` entitlement, and web SIWA needs a Services ID, which also needs
   the paid program. *Verify* against Apple's membership comparison. Proposed wording: "Guardians
   authenticate via **email one-time code** (primary in phase 1) or a one-time **bootstrap code**.
   Sign in with Apple is added when the paid program is active. All providers map to the same
   Guardian identity model." The same applies to **App Attest** (later, paid) and **remote push**:
   phase 1 has no APNs entitlement on a free team, so parent notifications go by email (Resend);
   `expo-notifications` is local-only.
3. **E3:** change "`expo-sqlite` and/or MMKV" to "**expo-sqlite + `expo-sqlite/kv-store`**; no MMKV".
   MMKV v4 is a Nitro module and adds native surface with no need.
4. **E2/E1b:** add "Hebrew text never inside GL/Skia canvases on kid screens until nikud is verified
   (2D RN overlay)" (already implied by E1b), and "**no KTX2/Draco/meshopt on native**; WebP/PNG
   textures ≤ 1024² (hero ≤ 2048²)".
5. **H6:** "web dashboard on Cloudflare Pages **proxying `/api/*` to the Worker via a service
   binding** (same-origin cookies)". This is a clarification, not a change.
6. **E11:** add packages `packages/shared` (wire contracts, RBAC, events), `packages/i18n` and
   `packages/api-client`, plus `apps/parent-web`. This is additive.
7. **E4:** add "**Maestro** golden-path flows on the Mac's iOS Simulator via Remote Control" and
   "Playwright baselines generated only in the pinned Playwright Docker image".

**Status after 00 revision 2 (Fable review):** items 1–7 were accepted into 00 (E1, E7/H2/H6, E3,
E1b, H6, E11, E4). **Status after 00 revision 3 (round 2):** items 8–11 below were also accepted
(E5 now says "two Cron triggers: nightly + one 15-min job with four ordered steps" and "CoopRoom
15 min idle / 30 min hard cap"; A3 says `UIRequiresFullScreen: true`; E6a references the §9.8 X25519
mechanism). They stay here as the record only.

8. **E5 crons:** 00 lists `*/15 6-12 * * *` for the weekly-report scheduler and "every 15 min" for the
   alert queue. This spec implements both inside **one** `*/15 * * * *` trigger (window filter in
   code, §7.1) — functionally identical, one fewer trigger, and the batch poll + expiry nudge share
   it. Proposed wording: "a 15-minute Cron trigger that runs the alert queue, the Friday `send_at`
   check, batch polling and expiry nudges".
9. **E5 `CoopRoom`:** "auto-close after 15 min idle" is adopted; add "30 min hard cap (C3 episode
   length)".
10. **A3/H1:** `UIRequiresFullScreen: true` (no Split View / Stage Manager resizing on iPad) — a
    safety and test-matrix decision; propose adding to A3.
11. **E6a:** the key-distribution mechanism (family X25519 keypair, private key only in guardian
    Keychains, QR transfer at S2) in §9.8 — propose referencing it from E6a.

---

## 19. Sources (primary where reachable; see `docs/research-log/tech.md` for method)

- npm registry metadata and tarballs (versions, bundledNativeModules, package APIs):
  https://registry.npmjs.org/expo/58.0.22 · https://registry.npmjs.org/@rive-app/react-native ·
  https://registry.npmjs.org/@react-three/fiber · https://registry.npmjs.org/expo-sqlite ·
  https://registry.npmjs.org/expo-audio · https://registry.npmjs.org/expo-localization
- Expo SDK 58 docs (via search extracts): Skia https://docs.expo.dev/versions/v58.0.0/sdk/skia.md ·
  Reanimated https://docs.expo.dev/versions/v58.0.0/sdk/reanimated/ · third-party overview
  https://docs.expo.dev/versions/v58.0.0/sdk/third-party-overview.md · SQLite
  https://docs.expo.dev/versions/latest/sdk/sqlite.md · QR codes https://docs.expo.dev/more/qr-codes ·
  localization https://docs.expo.dev/guides/localization/ · SDK 58 upgrade
  https://expo.dev/blog/how-to-upgrade-to-sdk-58 · Expo Go version mismatch
  https://docs.expo.dev/troubleshooting/expo-go-version-mismatch/
- Apple device setup and free signing (third-party/forums): https://flaviocopes.com/install-iphone-apps/ ·
  https://docs.hex-rays.com/core/debugger/tutorials/ios_debugging_coredevice ·
  https://developer.apple.com/forums/thread/843534 · https://www.rubydoc.info/gems/unlimit ·
  https://takazudo-zudo-tauri.pages.dev/pj/zudo-tauri/docs/mobile/ios-signing-free-team ·
  https://anexplorer.io/devmode/apple/iphone
- Cloudflare: Durable Objects pricing https://developers.cloudflare.com/durable-objects/platform/pricing ·
  WebSockets best practices https://developers.cloudflare.com/durable-objects/best-practices/websockets/index.md ·
  Workers/D1/R2 pricing and limits as cited in `backend_hosting_services.md`
- Anthropic: claude-api skill reference (cached 2026-10-06): models/pricing, Batches, structured
  outputs, fallbacks; https://platform.claude.com/docs/en/build-with-claude/structured-outputs
- Internal research notes: `app_stack_ai_built_workflow.md`, `backend_hosting_services.md`,
  `ios_technical_architecture.md`, `ai_models_assets.md`, `voice_tech_he_en.md`,
  `hebrew_first_language_voice.md`, `tech_3d_world.md`, `learning_through_movement.md`,
  `performance_reliability_playbook.md`, `quality_bug_prevention_playbook.md`,
  `personalization_tech.md`, `coop_economy_ios.md`, `photo_to_game_pipeline.md`.

---

## Fable review changelog

Reviewer F6, 2026-11-02. Inputs: 00 revision 2 (E3, E5, E6a, E7, E11a, E12, E13, A6, A7, B7/B7a,
C15, H1/H2), `docs/review/fable-issues-F1.md` #8/#14/#15/#19/#27/#28, 10 §15–§18, 05 §20.2, 09, 02
§2.1, 08 §1, 13 §3, the two playbooks. No web research was needed; nothing new was added to the
research log.

- **Crons (10 §18.2, 00 E5):** replaced `41 4 * * 0` (Sunday report) and `0 */6 * * *` with a single
  `*/15 * * * *` job (alert queue → Friday `send_at` report check → batch polling → expiry nudges) next
  to the nightly `10 0 * * *`; `send_at = clamp(candle − 3h, 12:00, 14:00)` Friday / erev chag, idempotent
  `reports.id = family:isoWeek`. §0, §1, §7.1, §8.1, §18 #8.
- **Weekly payload:** removed `flags[]` from `GET /reports/weekly`; flags only via `/children/:id/flags`
  and the monthly view (§7.10). Added 10's new endpoints as §7.10a and 10's tables as migration
  `0005_parent_area.sql` (§6.2) with a note on what stays a projection/doc rather than a table.
- **Events (§5.8):** added `focus.span` (computed on device, raw taps never stored), `episode.completed`,
  `skill.level_changed` (08 LevelId), `skill.delayed_check`, `world.secret/boss`, mission/mentor/holiday
  events, `app.update/storage_full/resigned`; made earnings/chest contents a pure fold of learn events
  (02 §2.1) with only user actions stored as `econ.*`; one wire↔engine name mapping in `packages/shared`.
- **Parent gate (S4):** `disableDeviceFallback:true` on `shared_kids`/`single_child`, `false` on
  `parent_only`; ON-06 "kids know the passcode" → PIN only; guardian picker after success for the audit
  log; cooldown persisted. Pairing: burn after 5 attempts, ≤ 3 open pairings per family (§5.3, §9.2).
- **00 rev 2 alignment:** canonical order `(ts, deviceId, eventId)` (E11a); `expo-device`,
  `expo-battery`, deferred game-controller and `olam-thermal` modules (E3, F1 #19/#28); bootstrap code
  TTL/revocation (E7, F1 #27); allowlist → 00 E13; staging at C3 / prod sprint 3 / VS local (E5);
  `CoopRoom` 15-min idle (E5); `dominant_hand`, `tier`+derived band, `ipad_mini` device type (H1/H2/A3/A6);
  B7 modes `soft/full/off` + B7a quiet days in `RestGate`; §9.8 encryption of sensitive derived data to
  guardian devices (E6a/C8a) with key transfer at S2.
- **Playbooks (E12, spec 14):** every one of the 25 performance rules now has an enforcement point
  (§4.15 table); new lints `no-raw-design-values`, `no-raw-pressable`, `engine-purity`,
  `no-skipped-tests`, exhaustiveness/floating-promises as errors, React-Compiler healthcheck; CI gained
  `content:validate`, routes/owners check, test-delta check and a Stryker incremental job (§2.4, §11.5);
  tests gained never-stuck, migration round-trip, replay determinism and field-LWW properties (§12.1);
  hang watchdog + thermal histograms (§13).
- **Coherence with 09:** every SCR-01…20 now has a route (added `map`, `results`, `base`, `pet`,
  `move/[activityId]`, `playroom`, `safe-room`, `rest`; pause is a modal) and every CMP-01…48 an owner
  package (§4.1 tables, CI-checked); fonts fixed to 09 B5a static files (§0, R-10); viewports and
  determinism hook names unified with 09 §13.2 (`window.__olam.*`); audio session `doNotMix` + parent
  toggle for silent mode; `UIRequiresFullScreen: true` (§0).
- **Edge-case walk (§4.18, 24 rows) + the rules behind it:** first launch offline and `bootstrapping`
  state (§4.8); DB recovery ladder for migration failure / corruption with event salvage and Keychain
  copy of the device identity (§6.3); storage-full, Low Power Mode and thermal degradation (§4.7);
  `UpdateGate` apply timing — never mid-episode (§3.3, §11.3); 7-day expiry walk (§3.3); Worker/AI
  vendor/$-cap/R2-missing handling (§4.11); field-level LWW for concurrent parent edits (§6.5); audio
  interruptions and route changes (§4.5); orientation/GL-context-loss/multitasking (§4.9); Dynamic Type
  extremes with nikud (R-11), VoiceOver in Hebrew (R-12), known RN RTL bugs → regression tests (R-13).
- **Acceptance criteria** 14–17 added (§16). `⟂VERIFY` markers: Stage Manager behaviour with
  `UIRequiresFullScreen`, expo-gl context-loss event name, Workers cron-trigger limit, Reanimated build
  flags vs Skia, `adjustsFontSizeToFit` with nikud on iOS 27, FlashList horizontal RTL, `@noble/ciphers`
  speed on Hermes.
- Cross-file issues for 13 (script path, channel name, tunnel line, Playwright viewports), 02/08 (event
  name mapping), 10 (PIN lockout numbers already aligned; `reports` table shape), 12 (nightly time) are
  in `docs/review/fable-issues-F6.md`.

### Round 2 changelog

Reviewer R6, 2026-11-02. Inputs: 00 revision 3 (A3, B8b, C8a, C13, E5, E9, H2, OPD-08, OPD-26),
`fable-issues-F2/F4/F5/F7/F8/F9.md` and 15 §3 GAPs addressed to 11, `round2-R1.md` §2, current 08
§1.4/§1.9–1.11/§10.5, 12 §1.9/§3.3/§3.15/§3.16/§6.9, 10 §4.1/§15.2, 02 §2.1/§6.3, 01 §12/DD-09/DD-31/
DD-34/DD-37/DD-38, 07 §2.7/§7.3, 06 §12.6. Disposition table: `docs/review/round2-R6.md`.

- **Events (§5.8, F5 #1–#4/#6, F2 #19/#24, F9 GAP-02/24, 02 §2.1):** `session.started/ended` v2 with
  08's `SessionEndReason` enum (old values mapped), `plannedMinutes/blocksCompleted/focusedMs`,
  wire-only `mode:'safe_room'` (0 cap minutes); `item.answered` v2 = 08 `Attempt` fields with v1
  upcast; new `behavior.signal` (replaces `checkin.mood`), `skill.level_reverted`, `benchmark.result`,
  `placement.finished`, `screen.enter/exit`, `app.background/foreground`, `creation.saved`,
  `launch.blocked`, `parent.tonight_done`, `parent.input` (08 `ParentInput` union), `cap.bonus_granted`,
  `asset.missing`, the seven 02 economy inputs (`review.rung_passed`…`campfire.lantern_lit`),
  `fact.reported`, `picturepin.miss`, `app.tz_mismatch`, `pack.revoked`; registry completeness test
  (§16 #18).
- **C8a (F5 #4, F4 #29):** `local_only` = not persisted beyond the episode; `fs_local_counters` table
  (device-only, excluded from log/sync/export); `rf_raised` lives inside the encrypted `flags` doc
  (§5.7, §5.9, §6.1, §16 #17).
- **Engine integration (F5 #7/#8, F2 #20/#21):** `engine_checkpoint` table (30/child, every 200
  events + session end), `model_epoch/subject_epoch` on child rows, `packages/shared/dayKey.ts` with
  the per-device monotonic `last_day_key`, `clock_skew_ms` on `device_self`/`devices` + `clockInaccurate`
  in `GET /devices`, corrected `ctx.now`, `lastServerTime` for 02 UC-15/01 UC-15, push-only flush on
  app background (§4.8, §6.1, §6.4, §6.5, §7.4).
- **AI caps and cost guard (F7 #5, F9 GAP-14e, OPD-08, R1-5):** $3/day (`ai_daily_cap_cents=300`),
  $40/month, console ≥ soft cap default $50 stored as `consoleCapCents`; the 12 §1.9 state machine
  per pipeline (`p80`/`p100`/monthly/kill), `deferred` jobs at 06:00, `OVERRIDE_USD` counter,
  `ai_cost_ledger`, upper-bound pre-check (§4.11, §6.2, §7.1, §7.7, §8.4, §14, §15.1, §16 #8).
- **Photo pipeline (F7 #6/#7, F9 GAP-15/15b/52, R1):** on-device privacy pass with
  `modules/vision-privacy`, orientation bake → EXIF strip → 2,576 px q0.88, double strip server-side;
  `POST /photo-jobs` multipart with `clientJobId` idempotency, states `uploaded → extracting →
  needs_confirm → building → ready | failed | deferred`, pHash 30-day duplicate check, 24 h
  auto-delete, `photo.rerender_variants` job, `capture_queue` (20 pages / 60 MB / 7 d); old
  `/photos` + `/ai/photo/*` removed (§2.3, §6.1, §6.2, §7.7, §7.9, §8.3, §9.2, §9.4, §16 #19).
- **Content (F7 #8/#9, F4 #30, 06 §12.6):** `revoked` pack status + loader rule (≤ 5 min hide, E-25),
  `pending_app_update` parent surface (GAP-27), closed `VoIntent` enum in `packages/shared`,
  `POST /content/fact-report`, `POST /fs/t2/request` chips-only (§4.4, §4.11, §7.6, §7.7).
- **Identity/devices (F9 GAP-04/07/11b/22/36/51/56/57/57b, R1-6/7, 00 B8b/C13/H2):** `family_prefs`
  fields `calmMode`, `consent.checkinNudge`, `travelCity`, server-derived `hasChildUnder36m` +
  audited `smallPartsOverride`; `bootstrap-guardian --role coadmin`; `child_remove` command and a
  defined `resync`; unpair = push-then-revoke (`force` for lost devices); family-delete grace (410
  `family_deleting`, E-26); pairing on a device with another family's data (E-27); per-child +
  per-device bedtime with "earliest wins" and the start rule; `launchMode`; device-local cap overrides;
  `+10 min` bonus route/event; picture-PIN miss/reset flow; guardian-removal semantics (§4.16, §5.1–5.5,
  §7.3, §7.4).
- **OTA and install (F9 GAP-26, F2 #22):** `UpdateGate` moments 2b (episode boundary / profile switch)
  and 2c (foreground after ≥ 30 min), the only `reloadAsync` caller; ⟂VERIFY container preservation on
  re-sign with the `--pre-install-sync` fallback and the unsynced counter (§3.3, §7.4).
- **Deep links (F9 GAP-68/48):** closed `olam://` route list incl. `report/{id}`, `approvals`,
  `invite`, `device/{id}` and dev-only `dev/fixture|setNow|moveTo|route|cycle`; Maestro layer wired to
  them with `testID` lint (§4.1, §12.1).
- **Storage/time/co-op/security:** audio cache LRU 400 MB + `storage_low` AL-11 (GAP-40); timezone/
  travel rules (GAP-39, 01 DD-31); DO single-sourced `boss_defeated` with id, 15-min rejoin, level-ups
  only from `coop.room_ended` (F2 #23, GAP-21); §9.9 data-at-rest table (GAP-41); parent voice clips
  §7.13 + `family.voice` pack (GAP-67); SCR-21 car-mode route stub (P2, GAP-19); VO loudness −18 LUFS
  ±1 (E9, R1-4); `UIRequiresFullScreen: true` in `app.config.ts` (A3, R1-1); §18 #8–11 marked accepted.
- **Rejected / deferred:** GAP-44 04:00 cap boundary — 00 OPD-26 binds "cap day = `DayKey`" (02's
  device-local day); `DAY_START_HOUR` constant kept at 0. F9's 4-job/48 h capture-queue numbers →
  12 §3.16's 20 pages / 60 MB / 7 days (12 owns the pipeline).
