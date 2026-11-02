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
| Backend | One Cloudflare Worker `olam-api` (Hono 4.13.14) with D1 `olam-db`, R2 `olam-private` + `olam-content`, Cron, the Durable Object `CoopRoom`, and the Rate Limiting binding. The parent dashboard is a Cloudflare Pages project `olam-parent` that proxies `/api/*` to the Worker through a service binding (same origin, cookie auth). | E5, H6. |
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
 │   /v1/ai/* /v1/tts/* /v1/photos/* /v1/reports/* /v1/admin/* /v1/coop/*                 │
 │ D1 olam-db (Time Travel 30d) · R2 olam-content (packs, audio) · R2 olam-private        │
 │ (photos 30d, backups) · DO CoopRoom · Rate Limiting · Cron (02:10 nightly, Sun 07:41)  │
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
| expo-gl / expo-sensors | 58.0.4 / 58.0.4 | 3D and motion |
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
@playwright/test 1.64.1, serve 14.2.6, drizzle-kit 0.31.11, fast-check 4.10.2, eas-cli 25.3.0
(invoked via `npx eas-cli@25.3.0`).

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
  - `olam/perf-bans`, from `performance_reliability_playbook.md` §11D:
    - `InteractionManager` and `Touchable*` (removed in RN 0.88)
    - `I18nManager.forceRTL` (RTL is set natively, never toggled)
    - reading `.value` of shared values in render
    - `new THREE.*` inside `useFrame`
    - `createAudioPlayer` inside event handlers
    - `<Image>`/`expo-image` without width/height or aspectRatio
  - `react-hooks` rules plus the React Compiler lint.
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
  it, and local data stays because the bundle ID and team are the same.

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
- Updates apply on the **next cold start**. The app never reloads mid-episode. `Updates.reloadAsync()`
  is called only from the parent area ("עדכון זמין — להפעיל מחדש?").

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
├─ picker.tsx                  Profile picker (avatars, optional picture-PIN, K3 → parent-present gate)
├─ (child)/[childId]/          ChildSessionProvider (opens a play session, caps, wind-down timer)
│  ├─ _layout.tsx              Stack; HUD layer; pause → picker on long-press (parent gate to exit app)
│  ├─ hub.tsx                  3D/2.5D hub (rescued friends live here)
│  ├─ world/[zoneId].tsx       explorable zone (R3F Canvas + 2D HUD)
│  ├─ level/[levelId].tsx      gimmick level (G3)
│  ├─ game/[gameId].tsx        2D learning mini-game (Skia) launched from world triggers
│  ├─ english/[islandId].tsx   English island — wraps children in <Dir value="ltr">, EN jingle on enter
│  ├─ collection.tsx           display room with silhouettes (G2)
│  ├─ shop.tsx                 cosmetic shop (K1 full, K2 simple; hidden for K3 per D5)
│  ├─ avatar.tsx               avatar / hero customizer
│  ├─ chest/[chestId].tsx      chest reveal (contents fixed at earn time, D2)
│  ├─ photo-game/[packId].tsx  play a parent-approved photo-to-game pack
│  └─ wind-down.tsx            episode end ritual
├─ (coop)/
│  ├─ lobby.tsx                same-device or multi-device; choose children/roles
│  └─ room/[roomId].tsx        campfire / co-op session
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
- Deep links use scheme `olam://`, for `olam://pair?code=…` only. Universal links need associated
  domains, which means the paid program, so they're not used.

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
  "minAppRuntime": "fp:7c1e…", "minEngine": 2, "locale": "he-IL", "ageBands": ["K2"],
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

**Bundled core content** (letters, numbers, core VO, hub assets) ships inside the binary under
`assets/`, so a fresh install works fully offline.

### 4.5 Audio service

`src/services/audio/AudioService.ts`, the single owner of audio:
- **Session** (called **once at startup** in `boot/`, never per scene): `setAudioModeAsync({ playsInSilentMode: true, interruptionMode: 'doNotMix',
  shouldPlayInBackground: false })`. Learning narration must be heard even when the iPhone's ring
  switch is on silent.
- **Buses:**
  - `music`: one looping player, crossfade 600 ms.
  - `vo`: a queue with no overlap; it ducks music to 25% over 150 ms.
  - `sfx`: a pool.
  - `ui`: a pool.

  Volumes are per-bus and per-child (`audio_prefs`).
- **Low-latency SFX:** at scene mount, preload the hot set (≤ 12 sounds), creating
  `createAudioPlayer(src)` × 2 instances each (round-robin). Playing means `seekTo(0)` + `play()`.
  Players are created at scene mount, never inside event handlers (lint rule), and released on exit. The latency target is **p95 tap→sound ≤ 50 ms** (hard release gate ≤ 80 ms, measured with 240 fps video per the device protocol), measured by the device test screen
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

- **Bundled:** fonts, core Rive files, core audio and hub GLBs come via `expo-asset`. Fonts are loaded
  before the splash screen hides (`useFonts`); the splash stays until fonts, DB migration and config
  are ready, max 3 s.
- **Downloaded:** pack files go to `documentDirectory` (persistent; not `cacheDirectory`, which iOS may
  purge).
- **Images:** `expo-image` with `cachePolicy:'disk'` for remote, plus `recyclingKey` in lists.
- **Rive:** `useRiveFile` with a module-level cache keyed by asset sha. Files are released on scene
  unmount, and at most 3 are kept alive.
- **3D:** `useGLTF.preload()` for the next zone during wind-down of the current one. Geometries,
  materials and textures are disposed on zone exit, and `renderer.info.memory` is checked in debug.
- **Memory budget** (oldest device, Instruments footprint):
  - home steady ≤ 250 MB
  - gameplay steady ≤ 400 MB
  - **per-scene peak ≤ 650 MB**
  - growth after 20 scene enter/exit cycles ≤ 20 MB

  A memory-warning handler (`AppState` `memoryWarning`) drops image, Rive and GLB caches. `AppState`
  `inactive`/`background` pauses the render loop, persists resume state and releases music.
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
- after an episode ends
- a manual "סנכרון עכשיו" in the parent area

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
  JS ms per frame).

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
| memory | ≤ 400 MB steady, ≤ 650 MB peak, ≤ 20 MB growth over 20 zone enter/exit cycles |

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
- **Classes and handling:**

  | Class | Example | Handling |
  |---|---|---|
  | Expected domain | pack pending update | inline UI state |
  | Network | timeout, 5xx | retry with exponential backoff (0.5 s → 30 s, jitter), queue |
  | Auth | 401 | refresh once, then re-pair prompt (parent only) |
  | Integrity | hash mismatch | discard + refetch |
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
- **R-10** Font: a Hebrew font with correct nikud GPOS (candidates Noto Sans Hebrew / Assistant /
  licensed kids' font), tested on device on day 1 with the nikud torture string in
  `packages/i18n/test/nikud-sample.txt`.

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
- **The 25 performance rules** from `performance_reliability_playbook.md` §11A are binding and are
  copied into `CLAUDE.md`. The key ones:
  - Per-frame state lives in shared values or refs, never React or Zustand state. No JS work per frame
    during gameplay.
  - Animate transform and opacity only.
  - Motion is time-based (`dt`), correct at 60 and 120 Hz. Set `CADisableMinimumFrameDurationOnPhone`
    in Info.plist for 120 Hz.
  - Never stack live Skia + Rive + GL surfaces in one frame unless profiled. At most 2 Skia canvases
    and 3 animating Rive instances (`.riv` ≤ 200 KB each).
  - Every image has explicit size or aspectRatio. Game sprites use `transition={0}`.
  - Nikud text uses lineHeight ≥ 1.5×, and in-game labels are capped by `maxFontSizeMultiplier`.
  - Fixed logical game resolution scaled into the safe area.
  - SQLite uses WAL and **async APIs only**, with batched transactions at checkpoints.
  - Every network call has a timeout plus `AbortController`.
  - Perf decisions are made only on Release builds on the oldest iPad.
- **Startup:** cold start to interactive picker ≤ 2.5 s on the oldest iPad. Only the core fonts and
  the picker Rive load before first paint; the hub GLB is preloaded while the picker is shown.
- **Tap-to-feedback:** visual response in the same frame (worklet), haptic ≤ 20 ms, sound p95 ≤ 50 ms.
- **Debug overlay** (`/(parent)/debug`): fps, JS frame ms, UI frame ms, draw calls, memory. In dev
  builds it can be pinned on screen.

### 4.16 Multi-profile on a shared iPad

- Device mode `shared-kids` shows the picker of **allowed children for this device** (H1).
- Selecting a child opens a `play_session` event (`session.started`). All events carry `childId` and
  `sessionId`.
- Per-child **resume state** (last zone, last hero position, pending chest) is a doc
  `child_resume/<childId>`, saved on every background/switch.
- **Switching** has three entry points:
  - Long-press (1.5 s) on the avatar badge → "החלפת שחקן" → back to the picker. This saves resume and
    closes the session (`session.ended{reason:'switch'}`).
  - Inactivity of 5 min also returns to the picker.
  - An episode cap reached → the wind-down screen, then the picker.
- **Picture-PIN** (H3): optional per child (default on for K1). A 3-picture sequence from a 3×3 grid,
  stored as `scrypt(sequence, salt)` in the child doc. Verified locally; after 3 misses it says "אפשר
  לבקש עזרה מאמא או אבא" (no lockout timer, no punishment).
- **K3 lap mode:** launching K3 needs a parent-present confirmation: the parent gate or a parent
  "hold for 2 s" button on a device paired to a guardian session. Episode cap 3–5 min (C3).
- **Caps** are per child per day, enforced on the device from local events plus a server-merged total
  (cross-device usage is merged at sync; offline overrun is tolerated and never punished).

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
 │    ├─ DeviceCommand (bedtime_lock|unlock|unpair|wipe|refresh|resync)  queued → acked
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
| `bootstrap` | 1 (day 0) | the agent runs `pnpm ops bootstrap-owner --email <owner>` (wrangler D1 via CI) → prints a one-time **setup code** `XXXX-XXXX-XXXX` (24 h) → owner enters it in the app → owner guardian + family created | lets the family start before any email domain exists; code single-use, hashed |
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
   rate-limit (5/10min/IP; 5 attempts per pairing); verify; create device + device_children;
   issue device token; pairing.claimedBy; audit(device.paired by guardian)
                                        ── {deviceId, deviceToken, familySnapshot} ────────────────►
 guardian screen live-updates (poll 2s) "המכשיר צורף ✓"                    stores token in Keychain,
                                                                            initial pull sync → picker
```
On shared kid devices no guardian session is stored by default. The parent area uses the PIN or Face
ID gate locally. If a guardian wants admin actions from that device, they sign in there once
("התחברות הורה במכשיר זה"), and the refresh token is stored with `requireAuthentication: true`.

**S4 — Parent gate on any device**
```
child taps hidden-corner gear (hold 2s) → ParentGate
  ├─ capabilities.faceId → LocalAuthentication.authenticateAsync({promptMessage:"כניסת הורים", disableDeviceFallback:false})
  │     success → if guardian session present: unlock Keychain refresh → access JWT → (parent) routes
  │                else: local-only parent mode (settings that are device-local + docs LWW sync via device token
  │                      with guardian attribution = "device-local parent", restricted set; see RBAC)
  └─ PIN pad (6 digits, scrypt verify against synced guardian pin hashes; 5 fails → 60s cooldown, doubling)
No arithmetic gates (E7).
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
unpair: server revokes device token immediately (401 on next call → device shows "המכשיר נותק מהמשפחה"
        and returns to setup, keeping local data until wipe/reinstall).
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

Bedtime is a **per-device schedule** (doc `device_settings/<deviceId>`: `bedtime {start:'19:30',
end:'07:00', days}`). It's enforced locally and offline. In bedtime, the app shows a calm "לילה טוב"
screen with no game access, and the parent gate unlocks it for 30 min. A remote `bedtime_lock` command
applies the lock immediately until a given time.

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
- `local_only` events are emotional check-ins (C8) and raw per-tap telemetry. They **never leave the
  device**: the sync pusher filters by class, and a test asserts it.
- The dashboard filters by child (primary) and device (secondary: usage minutes per device, last sync,
  install expiry, app version).

### 5.8 Event types (initial registry, `packages/shared/events.ts`)

| Type | cls | Payload (abridged) |
|---|---|---|
| `session.started` / `session.ended` | learn | `{mode:'solo'\|'coop'\|'lap', reason?}` |
| `placement.step` | learn | `{probeId, skill, outcome, rtMs}` |
| `item.presented` / `item.answered` | learn | `{packId, packVer, itemId, skill, template, answer, correct, rtMs, hintsUsed, attempt}` |
| `hint.shown` | learn | `{itemId, level}` |
| `skill.mastered` | learn | `{skill, evidence:{n, acc}}` (emitted by engine, deterministic) |
| `world.trigger` / `world.rescue` / `world.collect` | learn | `{zoneId, triggerId, friendId?, pieceId?}` |
| `econ.earn` / `econ.spend` / `chest.earned` / `chest.opened` | econ | `{currency, amount, reasonEventId}` / `{itemId, price}` / `{chestId, contents[] fixed}` |
| `inventory.equip` | econ | `{slot, itemId}` |
| `coop.joined` / `coop.contribution` | learn | `{roomId, role, contribution}` |
| `motion.activity` | learn | `{activity, counts:{jumps:n}}` (aggregates only) |
| `app.error` / `app.perf` / `app.start` | telemetry | `{route, message, stack≤4KB}` / `{fpsP95, jsMsP95, coldStartMs}` / `{runtimeVersion, updateId}` |
| `checkin.mood` | local_only | `{mood}` (never synced) |

### 5.9 Retention policy

| Data | Where | Retention | Mechanism |
|---|---|---|---|
| `learn`/`econ` events | D1 `events` | 24 months raw, then rolled into `child_skill_month` aggregates; raw archived to R2 `backups/events/<yyyy-mm>.ndjson.gz` (kept until family delete) | Cron monthly |
| `telemetry` events | D1 `events` | 30 days | Cron nightly |
| `local_only` | device only | until device wipe; capped at last 90 days locally | device purge job |
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
  type TEXT NOT NULL CHECK (type IN ('ipad','iphone','web')),
  mode TEXT NOT NULL CHECK (mode IN ('shared_kids','single_child','parent_only')),
  paired_at INTEGER NOT NULL, signed_at INTEGER,            -- build signing time (free-team expiry)
  next_seq INTEGER NOT NULL DEFAULT 1
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
  grade TEXT NOT NULL,                                      -- 'toddler'|'gan_chova'|'grade_3'…
  age_band TEXT NOT NULL CHECK (age_band IN ('K1','K2','K3')),
  avatar_json TEXT NOT NULL DEFAULT '{}', picture_pin_hash TEXT, picture_pin_salt TEXT,
  allowed_on_device INTEGER NOT NULL DEFAULT 1, deleted INTEGER NOT NULL DEFAULT 0, hlc TEXT NOT NULL
) STRICT;

-- generic LWW documents (settings, resume state, device settings…)
CREATE TABLE doc (
  kind TEXT NOT NULL,                 -- 'child_settings'|'child_resume'|'device_settings'|'family_prefs'|'reward'
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
CREATE TABLE skill_state (
  child_id TEXT NOT NULL, skill TEXT NOT NULL, elo REAL NOT NULL, n INTEGER NOT NULL,
  correct INTEGER NOT NULL, box INTEGER NOT NULL, due_at INTEGER, last_at INTEGER,
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
  ai_daily_cap_cents INTEGER NOT NULL DEFAULT 100, created_at INTEGER NOT NULL, deleted_at INTEGER
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
  grade TEXT NOT NULL, age_band TEXT NOT NULL CHECK (age_band IN ('K1','K2','K3')),
  avatar_json TEXT NOT NULL DEFAULT '{}', picture_pin_hash TEXT, picture_pin_salt TEXT,
  hlc TEXT NOT NULL, created_at INTEGER NOT NULL, deleted_at INTEGER
) STRICT;

CREATE TABLE devices (
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL REFERENCES families(id),
  name TEXT NOT NULL, type TEXT NOT NULL CHECK (type IN ('ipad','iphone','web')),
  mode TEXT NOT NULL CHECK (mode IN ('shared_kids','single_child','parent_only')),
  token_hash TEXT NOT NULL UNIQUE, token_rotated_at INTEGER NOT NULL,
  paired_by TEXT NOT NULL, paired_at INTEGER NOT NULL,
  model TEXT, os_version TEXT, app_version TEXT, runtime_version TEXT, update_id TEXT,
  signed_at INTEGER, signing TEXT CHECK (signing IN ('free','paid','web')),
  last_seen_at INTEGER, last_sync_at INTEGER, pull_cursor INTEGER NOT NULL DEFAULT 0,
  status TEXT NOT NULL DEFAULT 'active' CHECK (status IN ('active','wipe_pending','unpaired')),
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
  type TEXT NOT NULL CHECK (type IN ('bedtime_lock','unlock','unpair','wipe','refresh_content','resync')),
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
  status TEXT NOT NULL CHECK (status IN ('draft','qa_failed','awaiting_approval','approved','rejected','retired')),
  source TEXT NOT NULL CHECK (source IN ('curriculum','photo','parent_request','seasonal')),
  manifest_key TEXT NOT NULL,                        -- R2 key in olam-content
  child_ids_json TEXT NOT NULL DEFAULT '[]', qa_report_json TEXT,
  created_at INTEGER NOT NULL, approved_by TEXT, approved_at INTEGER,
  PRIMARY KEY (pack_id, version)
) STRICT;
CREATE INDEX packs_family_status ON packs(family_id, status);

CREATE TABLE ai_jobs (
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL, kind TEXT NOT NULL
    CHECK (kind IN ('generate_pack','photo_extract','parent_summary','qa_judge','nikud')),
  status TEXT NOT NULL CHECK (status IN ('queued','submitted','running','succeeded','failed','cancelled')),
  batch_id TEXT, input_ref TEXT, output_ref TEXT, error TEXT,
  created_by TEXT, created_at INTEGER NOT NULL, updated_at INTEGER NOT NULL
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

CREATE TABLE photos (
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL, child_id TEXT, uploaded_by TEXT NOT NULL,
  r2_key TEXT NOT NULL, bytes INTEGER, status TEXT NOT NULL
    CHECK (status IN ('pending_upload','uploaded','extracting','extracted','approved','rejected','deleted')),
  job_id TEXT, created_at INTEGER NOT NULL, delete_after INTEGER NOT NULL, deleted_at INTEGER
) STRICT;

-- 0004_reports.sql ---------------------------------------------------------
CREATE TABLE child_skill_month (
  family_id TEXT NOT NULL, child_id TEXT NOT NULL, month TEXT NOT NULL, skill TEXT NOT NULL,
  attempts INTEGER NOT NULL, correct INTEGER NOT NULL, minutes REAL NOT NULL, mastered INTEGER NOT NULL,
  PRIMARY KEY (child_id, month, skill)
) STRICT;
CREATE TABLE reports (
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL, week TEXT NOT NULL,   -- ISO week starting Sunday
  body_json TEXT NOT NULL, summary_he TEXT, emailed_at INTEGER, created_at INTEGER NOT NULL
) STRICT;
CREATE TABLE coop_rooms (
  id TEXT PRIMARY KEY, family_id TEXT NOT NULL, created_by_device TEXT NOT NULL,
  created_at INTEGER NOT NULL, ended_at INTEGER, summary_json TEXT
) STRICT;
```

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
  - Projections can be dropped and rebuilt from events (the engine version is stored in
    `projection_checkpoint`; a bump triggers a rebuild per child, at about 5k events/child/year, well
    under 1 s).
- **Content/event payload versions:** each event `type` has `v`. Reducers handle all versions (an
  upcast function per version). Removing a version requires that no un-upcast data remains (none, since
  rebuild uses upcasters).

### 6.4 Event log format and clocks

- The envelope is in §5.7. Canonical order for projections is `(ts, device_id, seq)`, which is
  deterministic on every device.
- `seq` is device-local monotonic (from `device_self.next_seq` in the same transaction).
- **HLC** for docs: `"<physical ms 13 digits>:<counter 4 digits>:<deviceId>"`, compared
  lexicographically, implemented in `packages/shared/hlc.ts`. The server clamps the physical part to
  `serverNow + 5 min` to defeat a far-future device clock.

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
              "signedAt": 1761900000000, "os": "iPadOS 27.0.1", "model": "iPad13,18" },
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
  - **Docs:** last-writer-wins by HLC per document. Server and client both keep max(hlc). A
    tombstone (`deleted:true`) wins only if its HLC is higher.
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
app.route('/content', content); app.route('/ai', ai); app.route('/tts', tts); app.route('/photos', photos);
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
  "triggers": { "crons": ["10 0 * * *", "41 4 * * 0", "0 */6 * * *"] },
  "vars": { "APP_ENV": "production", "SIGNING_MODE": "free", "EMAIL_FROM": "olam@<domain>" }
  // secrets (wrangler secret put): ANTHROPIC_API_KEY, SESSION_SIGNING_KEY, PACK_SIGNING_KEY,
  // RESEND_API_KEY, TTS_API_KEY, R2_ACCESS_KEY_ID, R2_SECRET_ACCESS_KEY, IP_HASH_SALT, OPS_TOKEN
}
```
Cron times are UTC:
- `10 0 * * *` = 02:10/03:10 Israel: nightly backup, purge, content batch submit and collect.
- `41 4 * * 0` = Sunday about 06:41/07:41: weekly parent report.
- `0 */6 * * *`: batch polling, plus expiry nudges for free-signing installs.

The `ratelimits` binding and its exact config syntax should be checked against the current Workers
docs at scaffold time (*verify*).

**Environments:**
- `production` (the family)
- `preview` (wrangler env `preview`, a separate D1 `olam-db-preview` and buckets `*-preview`, used by
  CI e2e smoke and the agent's experiments)
- local `wrangler dev` (Miniflare)

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
| PATCH `/family` | family.update | `{name?, tz?, prefs?{shabbatPause, holidays, quietHours}}` → `{family}` |
| DELETE `/family` | family.delete (owner) | `{confirm:"<family name>"}` → 202, schedules full purge (7-day grace) |
| GET `/guardians/me` · PATCH `/guardians/me` | self | `{displayName?, gender?}` |
| PUT `/guardians/me/pin` | self | `{pinHash, pinSalt}` (scrypt N=2^15,r=8,p=1 computed on client) → 204 |
| PATCH `/guardians/:id` | guardian.changeRole | `{role}` → `{guardian}` |
| DELETE `/guardians/:id` | guardian.remove | → 204 (revokes sessions) |
| POST `/invites` | guardian.invite | `{role:'coadmin'\|'viewer', email?}` → 201 `{inviteId, code, link, qrPayload, expiresAt}` |
| GET `/invites` · DELETE `/invites/:id` | guardian.invite | list / revoke |
| POST `/invites/accept` | none (RL_AUTH) | `{code}` → 200 `{familyName, role, next:'email_verify'}` (the email step completes) |
| GET `/children` | family.read | → `{children[]}` (device scope: allowed only) |
| POST `/children` | child.create | `{name, nickname?, birthdate, grammaticalGender:'f'\|'m'\|'n', grade, avatar}` → 201 `{child}` (server derives `ageBand`) |
| PATCH `/children/:id` | child.update | partial → `{child}` |
| PUT `/children/:id/picture-pin` | child.settings | `{hash, salt}` or `{clear:true}` → 204 |
| DELETE `/children/:id` | child.delete (owner) | → 202 (soft delete; events purge after 30 d) |

### 7.4 Devices and pairing

| Method & path | Perm / auth | Request → Response |
|---|---|---|
| POST `/devices/self` | guardian (first device of owner during onboarding) | `{name, type, mode, allowedChildIds, info}` → 201 `{deviceId, deviceToken}` |
| POST `/pairings` | device.pair | `{deviceName, mode, allowedChildIds}` → 201 `{pairingId, code:"482913", qrPayload, expiresAt}` |
| GET `/pairings/:id` | device.pair | → `{status:'open'\|'claimed'\|'expired', deviceId?}` (poll 2 s) |
| DELETE `/pairings/:id` | device.pair | cancel |
| POST `/pairings/claim` | none (RL_PAIR, 5/10 min/IP) | `{pairingId?, nonce?, code?, info:{model, os, appVersion, runtimeVersion, signedAt, signing}}` → 201 `{deviceId, deviceToken, familySnapshot}`; 410 expired; 422 `too_many_attempts` |
| GET `/devices` | family.read (guardian) | → `{devices:[{id,name,type,mode,children[],lastSeenAt,lastSyncAt,appVersion,runtimeVersion,installExpiresAt,status}]}` |
| PATCH `/devices/:id` | device.update | `{name?, mode?, allowedChildIds?}` → `{device}` |
| POST `/devices/:id/commands` | device.command.* | `{type:'bedtime_lock'\|'unlock'\|'unpair'\|'wipe'\|'refresh_content'\|'resync', args?}` → 201 `{commandId}` |
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
| GET `/content/catalog` | device | `?since=<etag>` → `{packs:[{packId, version, manifestUrl, childIds, required:boolean}]}`; only `approved` packs for the device's allowed children |
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
4. **cost pre-check**: if `ai_spend_daily.cost` plus the estimate is over `families.ai_daily_cap_cents`,
   return 503 `cap_reached`
5. child-safety system prompt + structured output
6. post-validation in code
7. `ai_log` row
8. spend update

| Method & path | Auth | Model | Request → Response |
|---|---|---|---|
| POST `/ai/packs/generate` | content.approve (or Cron) | Opus 5.5 **Batch** + Sonnet 5.5 judge (batch) | `{childId, skills[], template?, count, interestTags?, source}` → 202 `{jobId}`; the job ends as a pack in `awaiting_approval` |
| GET `/ai/jobs/:id` | guardian | — | `{status, progress?, packRef?, error?}` |
| POST `/ai/photo/extract` | photo.capture | Sonnet 5.5 (vision, realtime, `fallbacks:"default"`) | `{photoId, childId, hint?:'math'\|'spelling'\|…}` → 202 `{jobId}`. The result is `{extraction (schema §8.3), proposedPack}` awaiting parent confirm |
| POST `/ai/photo/:jobId/confirm` | photo.capture | — (templates in code) + Opus 5.5 realtime for distractors/hints only if needed | `{items (edited), childIds}` → 201 `{packId, version}` (approved by this action); photo deleted |
| POST `/ai/companion/turn` | device | Haiku 5.5 realtime | `{childId, sessionId, itemId, context:{template, skill, attemptHistory[≤5], childAnswer?}, turnIndex}` → 200 `{hint:{textKey?, text, nikudText, level:1..3}, revealsAnswer:false, safety:{ok:true}}` |
| POST `/ai/reports/summary` | Cron / guardian | Haiku 5.5 (batch nightly) | `{familyId, week}` → `{reportId}` |
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

### 7.9 Photos (signed upload)

| Method & path | Auth | Request → Response |
|---|---|---|
| POST `/photos` | photo.capture | `{childId?, bytes, contentType:'image/jpeg'}` → 201 `{photoId, uploadUrl (PUT, 10 min, presigned, Content-Type and Content-Length bound), deleteAfter}` |
| POST `/photos/:id/uploaded` | photo.capture | → 200; verifies the object exists and size ≤ 5 MB |
| DELETE `/photos/:id` | photo.capture | immediate delete (R2 + row) |

The client must strip EXIF and downscale before upload (§9.4). The server re-checks by parsing the
JPEG and rejecting any APP1/EXIF segment.

### 7.10 Reports

| Method & path | Auth | Notes |
|---|---|---|
| GET `/reports/weekly?week=` | child.reports.read (viewer: celebrations only) | `{week, children:[{childId, minutes, sessions, highlights[], skills:{mastered[], practicing[], suggested[]}, eveningIdeas[], flags[] (non-diagnostic)}], summaryHe}` |
| GET `/reports/children/:id/skills` | child.reports.read | per-skill state from server projections (rebuilt nightly from events) |
| GET `/reports/devices/usage?from&to` | family.read (O/C) | minutes per device × child, last sync |
| POST `/reports/weekly/:id/email` | O/C | re-send email |

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
| PATCH `/admin/limits` | admin.ops | `{aiDailyCapCents}` (max 1000 without a second confirm) |
| POST `/admin/export` | data.export | → 202; builds a ZIP (NDJSON per table + photos none + audio list) in R2 and emails a 24 h link |
| POST `/admin/backup/run` | admin.ops | run the backup now |
| POST `/ops/notify` | `OPS_TOKEN` (Mac resign script, CI) | `{kind:'resign_overdue'\|'ci_failed'\|…, deviceId?, text}` → emails O/C, rate-limited 1/kind/12 h |
| POST `/ops/bootstrap` | `OPS_TOKEN` (CI only) | `{email}` → `{setupCode}` (used by `tools/ops/bootstrap-owner`) |

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
3. **Collect** (6-hourly): `batches.retrieve(id)` until `processing_status === 'ended'`, then
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
- Loudness is normalized to −16 LUFS (ffmpeg loudnorm) and encoded as AAC 64 kbps mono m4a.
- Stored content-addressed in R2.

### 8.3 Photo-to-game (E8, parent confirm mandatory)

```
Parent device (gate)                         Worker                              Anthropic
capture (expo-camera / image-picker) → crop to exercise area (user) → expo-image-manipulator:
  resize long edge 1600px, JPEG q0.8 (re-encode drops EXIF/GPS) → on-device face check (later)
POST /photos → PUT uploadUrl → POST /photos/:id/uploaded
POST /ai/photo/extract ─────────────────────► fetch R2 object → base64 image block
                                              messages.create({model:'claude-sonnet-5-5',
                                                output_config:{format:{type:'json_schema', schema: ExtractionSchema}},
                                                betas/fallbacks:'default', system: extraction+safety}) ─► 
                                              ◄─ extraction (items, pii_detected[], confidence)
                                              code: recompute math, drop low-confidence, replace
                                              detected names with placeholders, choose templates
◄── poll GET /ai/jobs/:id → review screen: photo ⟷ items side by side; edit; choose child(ren)
POST /ai/photo/:jobId/confirm ───────────────► build pack (templates in code), optional
                                              distractors/hints call, TTS from cache or bake queue,
                                              delete photo now. Playable at once with on-screen text
                                              + bundled generic VO; item VO swaps in when baked.
```
Latency target: shutter → review screen ≤ 15 s p50. ExtractionSchema follows the
`photo_to_game_pipeline.md` schema, within Claude's structured-output limits (no numeric bounds, so
counts are enforced in code).

### 8.4 Cost controls

1. **Worker hard cap:** default $1.00 per family-day (`ai_daily_cap_cents`), plus per-route quotas
   (companion 20/child/day, photo 10/day).
2. **Anthropic workspace** "olam-family" with a monthly spend limit of $30, set by the parent.
3. A kill switch in config.
4. An alert email when 80% of the daily cap is reached.
5. A Batch-first policy for anything not interactive.

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
- Body limit 1 MB (sync), 64 KB (other routes). Photos go directly to R2 via presigned PUT.
- Rate limits (Workers Rate Limiting binding, keyed by `ip` for unauthenticated routes and by
  `deviceId`/`guardianId` otherwise): auth 10/min, pairing claim 5/min, general 120/min, AI 20/min.
  D1-backed counters handle per-code attempts: pairing 5, email code 5.
- **Brute-force math:** the 6-digit pairing code has 10⁶ space, a 10-min TTL, 5 attempts per pairing
  and 5/min/IP. The success probability per pairing is ≤ 5×10⁻⁶. QR payloads also carry a 128-bit
  nonce, so a scanned QR is not guessable at all.
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

- **On device:**
  - The user crops to the exercise.
  - `expo-image-manipulator` re-encodes to JPEG with a 1600 px long edge. The re-encode writes no
    EXIF; the server verifies this.
  - Later: on-device face detection blurs faces before upload.
  - Kids can never start a photo flow; it's parent-gated.
- **Storage:** private bucket `olam-private`, key `photos/<familyId>/<photoId>.jpg`, no public access.
  Access is only via 10-min presigned URLs to guardians.
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
4. The room ends when all leave, after 2 min idle, or after 30 min max (episode cap). The DO writes
   `coop_rooms.summary_json`.

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
- **Reconnect:** `hello{lastTick}` gets a full `welcome` back.
- **Rate limit:** ≤ 20 msgs/s per socket.
- **Offline fallback:** if the socket drops for more than 10 s, the client continues in local
  "solo co-op" mode and merges contributions as events afterwards. There is no loss.

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
      UIRequiresFullScreen: false,
    },
    usesAppleSignIn: SIGNING === 'paid',
  },
  web: { output: 'static', bundler: 'metro' },
  plugins: [
    'expo-router',
    ['expo-localization', { supportsRTL: true, forcesRTL: true, supportedLocales: ['he', 'en'] }],
    ['expo-font', { fonts: ['./assets/fonts/NotoSansHebrew-VF.ttf', './assets/fonts/Fredoka-VF.ttf'] }],
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
| E2E golden paths | <5% | **Maestro** YAML on the **Mac iOS Simulator** via Remote Control | before each native build and before promote | onboarding (bootstrap) → add child → pair (simulated code) → picker → episode → chest → parent gate → settings → sync |
| Real device | — | checklist §12.4 + device perf protocol §12.5 | each native build; monthly | haptics, audio latency, Rive, nikud, Face ID, camera QR, 3D fps, thermal |
| Mutation (optional) | — | Stryker incremental on `packages/engine` | nightly | test strength of core logic |

**Key properties tested with fast-check:**
1. **Convergence:** for any interleaving and partition of events across N simulated devices, after
   full sync every device's projections are equal.
2. **No-loss economy:** no event sequence reduces owned inventory, collection or rescued friends.
   Balance displayed ≥ 0.
3. Chest contents are fixed at earn time: opening never changes contents.
4. Success-band selector: over simulated learners at ability θ, observed correctness stays within the
   band ±5% after warm-up.
5. HLC LWW is monotonic and commutative.
6. `local_only` events are never present in any push payload.

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
| Viewport (device descriptors) | iPad Pro 11 landscape (1194×834), iPad Pro 11 portrait, iPad mini landscape (1133×744), iPhone 16 portrait (393×852), iPhone 16 landscape (game scenes only) |
| Profile | K1 (f, grade 3, nikud partial), K2 (m, gan), K3-f, K3-m (lap mode), family/co-op (pl) |
| Guardian addressee | guardian-f, guardian-m, plural (parent screens) |
| Direction | RTL (all), English island LTR screens, mixed bidi strings |
| Theme/size | default; largest Dynamic Type (parent screens); `prefers-reduced-motion` |
| Network | online; offline banner states (route-intercepted) |

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

---

## 13. Observability

- **Self-hosted events:** `app.error`, `app.perf` (cold start, fps p95, JS ms p95, memory peak per
  scene) and `app.start` (`runtimeVersion`, `updateId`, signing, expiry) are telemetry-class events,
  synced and kept 30 days.
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
- **Weekly ops line** in the Sunday parent email (owner only): "הכול תקין" or a short list (failed
  syncs, expiring installs, cap hits).
- **Sentry (optional, off by default):** `@sentry/react-native` 8.28.0 is enabled only if the owner
  opts in. When on, it uses `sendDefaultPii:false`, a `beforeSend` scrubber that strips names, child
  ids and breadcrumbs with text, and is dev/preview channel only. The Developer plan is free (5k
  errors/mo). Apple crash logs via Xcode Organizer aren't available for free-signed builds, so the
  self-hosted `app.error` plus Mac Console logs are the baseline.
- **Uptime:** a GitHub Actions cron calls `/v1/health` every 6 h and emails on failure. That's enough
  for a family.

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
| Claude: companion (Haiku 5.5) + summaries + photo extraction (Sonnet 5.5) | $1–4 | $1–4 | capped $1/day hard |
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
   - console.anthropic.com → Workspaces → create "olam-family" → Limits → monthly spend limit $30.
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
11. **Cloud environment network allowlist** (so the agent can verify docs and run CLIs from the
    container):
    - Cloud environment menu → Edit → Network access → add these to Allowed domains:
      - `docs.expo.dev`, `expo.dev`, `api.expo.dev`, `u.expo.dev`, `qr.expo.dev`
      - `developers.cloudflare.com`, `api.cloudflare.com`, `*.workers.dev`, `*.pages.dev`
      - `appleid.apple.com`, `rive.app`, `hono.dev`
      - `playwright.azureedge.net`, `cdn.playwright.dev`
      - the TTS vendor API host
    - Optional. CI works without it.

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
   - Bedtime lock reaches a foreground device in ≤ 60 s. Unpair revokes on next request. Wipe deletes
     local data and acks.
6. **Audit and retention:** every admin mutation produces an audit row. Cron purges per §5.9, verified
   by a time-travel test (Miniflare fake time).
7. **Content:**
   - Packs download, verify sha256 and the Ed25519 signature, and switch atomically.
   - A tampered file is rejected and the previous version stays active.
   - Unapproved packs are never served to devices (test).
8. **AI:**
   - Every AI route enforces the daily $ cap (503 at cap), logs to `ai_log` with cost, and uses
     structured outputs.
   - Companion replies that leak the answer are blocked by code (unit tests with adversarial
     fixtures).
   - Photos are deleted after confirm and never older than 30 days (lifecycle + cron test).
9. **Performance** on the oldest family device, Release build:
   - cold start ≤ 2.5 s (target 2.0 s)
   - 2D games 60 fps (120 fps on ProMotion) with hitch ratio < 5 ms/s
   - 3D world ≥ 55 fps p95 (gate #2)
   - tap→sound p95 ≤ 50 ms (≤ 80 ms hard gate)
   - memory ≤ 400 MB steady / ≤ 650 MB peak, ≤ 20 MB growth over 20 cycles
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
