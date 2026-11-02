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
| react-native-audio-api | 0.13.6 | **optional**, adopted only if the expo-audio latency gate fails (§4.5) |
| @sentry/react-native | 8.28.0 | **optional**, dev build only, behind flag (§13) |
| react-native-vision-camera | 5.2.3 | **later phase** (camera pose, G6) |

**Dev/test (mobile):** jest 30.5.2, jest-expo 58.0.13, @testing-library/react-native 14.1.0,
@playwright/test 1.64.1, serve 14.2.6, drizzle-kit 0.31.11, fast-check 4.10.2, eas-cli 25.3.0
(invoked via `npx eas-cli@25.3.0`).

**workers/api:** hono 4.13.14, @hono/zod-validator 0.9.1, zod 4.6.7, drizzle-orm 0.45.4
(`drizzle-orm/d1`), jose 6.2.13, @anthropic-ai/sdk 1.3.0, aws4fetch 1.0.20 (R2 presigned URLs),
uuidv7 1.2.1. Dev: wrangler 4.152.0, @cloudflare/workers-types 5.20261025.1, vitest 4.1.12,
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
4. Compute sha256 of each file with @noble/hashes, streaming in 1 MB chunks off the UI frames (use
   `InteractionManager`). Mismatch → delete and retry ×3 → mark the pack `failed`. The app keeps the
   previous good version.
5. Atomic switch: `content_pack.active_version = new` in one SQLite transaction. Old versions are
   deleted after 7 days, or immediately when storage is low.
6. `minAppRuntime`/`minEngine` guard: a pack that needs a newer engine stays `pending_app_update`.

**Bundled core content** (letters, numbers, core VO, hub assets) ships inside the binary under
`assets/`, so a fresh install works fully offline.

### 4.5 Audio service

`src/services/audio/AudioService.ts`, the single owner of audio:
- **Session:** `setAudioModeAsync({ playsInSilentMode: true, interruptionMode: 'doNotMix',
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
  The latency target is **p95 tap→sound ≤ 50 ms**, measured by the device test screen
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
- **Memory budget:** ≤ 300 MB resident on the oldest device. Warnings come from the debug overlay;
  `AppState` background → pause the render loop and release music.

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
| memory | ≤ 300 MB resident |

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
  avoid wide re-renders. Expensive engine calls run in `InteractionManager.runAfterInteractions` or
  between episodes.
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
