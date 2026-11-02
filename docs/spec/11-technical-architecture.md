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
