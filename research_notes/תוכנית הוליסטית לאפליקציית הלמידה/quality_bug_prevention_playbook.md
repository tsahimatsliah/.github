# Quality & Bug-Prevention Playbook (Expo SDK 58 / RN 0.88 kids' game, built by an AI agent)

> Research method note: in this cloud session, WebFetch was blocked for expo.dev, docs.sentry.io, playwright.dev and docs.maestro.dev (DNS not in allowed domains). All findings below come from search-result snippets (official doc pages appeared in results, but full pages could not be opened). Version-specific claims for Expo SDK 58 / RN 0.88 could NOT be verified; treat them as "check against installed version". Items under "Inferences" are engineering recommendations synthesized by the researcher, not sourced facts.

## 1. Testing pyramid for an RN game (unit / component / visual / E2E / contract / Workers / migrations)

### Takeaway
Put most of the test weight on pure TypeScript logic (engine, economy, content validators) with Vitest/Jest plus fast-check property and model-based tests. Add a thin RNTL component layer. Use Playwright screenshots of the Expo web build as the main visual/RTL regression net on Linux, a handful of Maestro YAML flows on the parent's Mac simulator for E2E, and `@cloudflare/vitest-pool-workers` (real workerd + real D1 + migrations) for the backend.

### Cited Findings
**Unit / property-based (fast-check)**
- fast-check supports model-based testing: "we compare our system to a highly simplified version of it: the model". Commands implement `ICommand` with `check(model)`, `run`, `toString`. `fc.commands([...])` builds scenarios, is "like an enhanced version of the array", and unlike `array` "can efficiently shrink failing scenarios". Runners include `fc.modelRun` (sync), with a setup returning `{ model, real }` inside `fc.assert(fc.property(...))`. — [fast-check model-based testing docs](https://fast-check.dev/docs/advanced/model-based-testing/)
- To replay a failing model-based run you pass `{ seed, path }` to `fc.assert`, and `fc.commands` must also get `{ replayPath }` (from an older doc version, so verify). — [fast-check Tips.md on GitHub](https://github.com/dubzzz/fast-check/blob/68a4c78741d23f068febe8b2816474726133882c/documentation/1-Guides/Tips.md)

**Component tests (jest-expo + RNTL + Reanimated)**
- Reanimated's current docs: set the Jest resolver to `react-native-reanimated/jest/resolver`, call `setUpTests()` in the setup file (default config `{ fps: 60 }`), and first follow the separate React Native Worklets Jest guide. — [Reanimated Testing docs (unreleased)](https://docs.swmansion.com/react-native-reanimated/docs/unreleased/guides/testing/); [Reanimated Testing with Jest](https://docs.swmansion.com/react-native-reanimated/docs/guides/testing/)
- Older tutorials mock with `jest.mock('react-native-reanimated', () => require('react-native-reanimated/mock'))`, which disables animations and returns synchronous values. jest-expo is the Jest preset for Expo projects. — [reactnativerelay 2026 testing guide (FA)](https://reactnativerelay.com/fa/article/rahnama-jame-test-react-native-2026); [RNTL PR #1024](https://github.com/callstack/react-native-testing-library/pull/1024/files)

**Visual regression (Playwright on Expo web)**
- Baselines must come from the same OS and browser version. Run Playwright in Docker (`mcr.microsoft.com/playwright`) and update baselines with `npx playwright test --update-snapshots` from CI, not a laptop, then review and commit the PNGs. — [Argos: Playwright VRT in CI](https://argos-ci.com/blog/playwright-visual-regression-testing-ci)
- Argos says `animations: "disabled"` is the default for `toHaveScreenshot`: CSS animations are frozen and the caret is hidden. Other guides set it explicitly. — [Argos](https://argos-ci.com/blog/playwright-visual-regression-testing-ci); [oneuptime Playwright visual testing (2026)](https://oneuptime.com/blog/post/2026-01-27-playwright-visual-testing/view)
- `mask` replaces elements with a colored box (magenta by default, changeable via `maskColor`), and the comparison ignores those regions. `maxDiffPixelRatio` is the max fraction of differing pixels (0.01 = 1%). It is commonly set globally, with per-assertion overrides for pages that vary more. — [testdino Playwright visual testing](https://testdino.com/blog/playwright-visual-testing); [bug0 KB](https://bug0.com/knowledge-base/playwright-visual-regression-testing); [helpmetest](https://helpmetest.com/blog/playwright-visual-regression-testing/)

**E2E: Maestro vs Detox (2026)**
- Detox is grey-box: it syncs with the RN JS runtime to know when the app is idle, so it needs no sleeps. Maestro is black-box and drives the app through accessibility APIs. Maestro tests are YAML with no compile step. — [pkgpulse Detox vs Maestro vs Appium 2026](https://www.pkgpulse.com/guides/detox-vs-maestro-vs-appium-react-native-e2e-testing-2026); [oneuptime Maestro RN (Jan 2026)](https://oneuptime.com/blog/post/2026-01-15-react-native-maestro-testing/view)
- Setup estimate: about 30 minutes for Maestro versus 4 to 8 hours for Detox. For Expo, Maestro is "plug-and-play", while Detox needs prebuild plus extra eas.json config and is tied to specific RN versions. Detox v20 is described as stable but less actively developed than Maestro. — [reactnativerelay CZ](https://reactnativerelay.com/cs/article/detox-vs-maestro-e2e-testovani-react-native-2026); [reactnativerelay BG](https://reactnativerelay.com/bg/article/detox-vs-maestro-react-native-e2e-testirane-2026); [pkgpulse](https://www.pkgpulse.com/guides/detox-vs-maestro-vs-appium-react-native-e2e-testing-2026)
- Anecdote: one team cut a 150-test run from 42 to 12 minutes moving from Detox 20.1 to Maestro 1.30. Single-team claim. A Slovak article reports Detox faster on some flows, using unsourced benchmarks, so the speed claims conflict. — [DEV Community](https://dev.to/johalputt/we-cut-70-of-mobile-test-time-in-2026-by-ditching-detox-for-maestro-130-48kn); contradicted by [reactnativerelay SK](https://reactnativerelay.com/sk/article/maestro-vs-detox-2026-react-native-testing)
- Flakiness sources differ: Detox flakes usually come from animation synchronization, Maestro flakes more often from assertion timing. — [reactnativerelay BG](https://reactnativerelay.com/bg/article/detox-vs-maestro-react-native-e2e-testirane-2026)
- Maestro officially runs on any Xcode-managed iOS Simulator and needs Xcode Command Line Tools. — [Maestro docs: iOS supported platform](https://docs.maestro.dev/get-started/supported-platform/ios)
- As of Dec 2025, real iOS devices were not officially supported. A community patch, `maestro-ios-device` (macOS, Maestro 2.x, Xcode, USB, iOS 15+, trusted dev cert, "use at your own risk"), builds an XCTest runner on the device. DeviceLab's commercial maestro-runner claims real-device support. — [devicelab maestro-ios-device](https://devicelab.dev/open-source/maestro-ios-device); [DEV: Maestro on real iOS devices](https://dev.to/omnarayan/maestro-on-real-ios-devices-working-guide-5dfk); [BrowserStack guide](https://www.browserstack.com/guide/maestro-real-ios-device-testing)

**API contract + Workers tests**
- `@cloudflare/vitest-pool-workers` runs tests inside the same workerd runtime as production, so D1 bindings are real rather than mocked. `readD1Migrations(migrationsPath)` in the Vitest config returns migrations ordered by number, typically passed to a test-only binding (`TEST_MIGRATIONS`). `applyD1Migrations(db, migrations, tableName?)` applies un-applied ones in a setup or test file and records state in `d1_migrations`. — [Cloudflare Vitest integration config](https://developers.cloudflare.com/workers/testing/vitest-integration/configuration/); [Cloudflare test APIs](https://developers.cloudflare.com:2053/workers/testing/vitest-integration/test-apis); [Cloudflare recipes](https://developers.cloudflare.com/workers/testing/vitest-integration/recipes/index.md); [helpmetest D1 guide](https://helpmetest.com/blog/cloudflare-d1-testing-guide/)
- Hono RPC: export `AppType = typeof app` from chained routes, since breaking the chain loses inference. The client uses `hc<AppType>()`, with `InferRequestType` and `InferResponseType` helpers. `testClient` from `hono/testing` mirrors the RPC client in tests without opening ports. — [Yusuke Wada: Hono RPC](https://blog-yusuke.pages.dev/hono-rpc); [Codex CLI for Hono (Jun 2026)](https://codex.danielvaughan.com/2026/06/03/codex-cli-hono-edge-apis-zod-openapi-rpc-cloudflare-workers/); [hono-dev skill](https://skills.sh/outfitter-dev/agents/hono-dev)

### Inferences
- **Suggested pyramid weight:** about 70% pure-logic tests (Vitest, with fast-check for the economy, scoring, adaptive difficulty and save migrations), about 15% RNTL component tests, about 10% Playwright web screenshots, and under 5% Maestro flows (5 to 10 golden paths).
- **Maestro over Detox** for this project: Expo-friendly, YAML is easy for an agent to write, and it can run on the parent's Mac simulator through Remote Control. Real-iPad automation stays manual or exploratory.
- **Visual matrix:** profiles × gender (m/f) × viewport (iPhone SE, iPhone Pro Max, iPad 11" portrait and landscape) × `dir=rtl`. Freeze time with Playwright's clock API and a fixed RNG seed. Mask 3D canvases, Rive and Skia animated regions, or render them at a fixed frame through a `?testFrame=` debug hook. Use `maxDiffPixelRatio` of about 0.01 globally and looser for GL views.
- R3F/three.js, Skia and Rive on web differ from native renderers, so web screenshots validate layout, RTL and copy, not native GPU output. Native visuals need simulator screenshots via `maestro takeScreenshot` on the Mac.
- **Contract:** one shared `packages/contracts` with zod schemas, imported by the Hono `zValidator` server-side and by the client to parse responses. Test with `testClient` running inside vitest-pool-workers so D1 is real.
- **Migration tests:** apply every migration to an empty D1, then run a "previous schema + fixture data → apply new migration → assert invariants" test for each migration.

### Gaps
- Exact Playwright defaults (e.g. whether `animations: 'disabled'` is the default in the 2026 version) are unverified because playwright.dev was blocked.
- I found no source on testing R3F or `react-three-fiber/native` in Jest. Its renderer `@react-three/test-renderer` exists, but no 2026 doc was retrieved.
- I found no source confirming Maestro official real-iOS-device support in 2026.

## 2. Static safety (TS strict, ESLint, zod, state machines, flags, invariants)

### Takeaway
Few sources were retrieved specifically for this question. The recommendations rest on the type-flow facts from Hono and zod, plus standard practice.

### Cited Findings
- Hono's type inference flows only through chained routes and validators. Assigning routes to variables between them drops the types. — [Yusuke Wada: Hono RPC](https://blog-yusuke.pages.dev/hono-rpc); [hono-dev skill](https://skills.sh/outfitter-dev/agents/hono-dev)
- fast-check's `fc.commands` plus `check(model)` lets you generate random action sequences against a state model, which is how you find unreachable or stuck states. — [fast-check model-based testing](https://fast-check.dev/docs/advanced/model-based-testing/)

### Inferences
- **TypeScript:** `strict`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `noImplicitOverride`. `tsc --noEmit` is a CI gate for every package (app, worker, contracts, content).
- **ESLint:** `react-hooks` (rules-of-hooks, exhaustive-deps as error), `@typescript-eslint/switch-exhaustiveness-check`, `no-floating-promises`, and a ban on `left`/`right`/`marginLeft`/`paddingRight` style keys in favor of `start`/`end` (custom `no-restricted-syntax` rule; see §8 RTL). Also ban raw string literals in JSX so every string goes through the gendered i18n function.
- **zod at every boundary:** API responses, persisted saves, content packs, deep links, Remote Control/dev-menu inputs. Never `as` a parsed `JSON.parse` result.
- **Session and game flows** (lesson → activity → reward → back to map; app background/resume) as explicit state machines, either XState v5 or a typed reducer with a discriminated-union state. Requirements: every state has an exit (a "home" or timeout event is always handled), plus an fast-check model test that fires random events and asserts "never stuck" (a reachable path back to home always exists) and "no illegal currency changes".
- **Dev-only invariants:** `invariant(cond, msg)` that throws in `__DEV__`, logs to the backend in prod, and never crashes a kid's session.
- **Feature flags:** a static JSON served from the Worker, per child profile, with kill switches for 3D scenes and new mini-games.

### Gaps
- I did not retrieve current XState v5 docs or ESLint plugin versions for RN 0.88. Verify the plugin names and config format (flat config) against the installed versions.

## 3. Determinism for games (fixed timestep, seeded RNG, replay, save integrity)

### Takeaway
No game-determinism sources were retrieved within the tool budget. The cited findings below are adjacent (fast-check seeds and replay), and the rest is engineering practice.

### Cited Findings
- fast-check failures are reproducible through `{ seed, path }` (plus `replayPath` for commands), which supports a "bug → seed → regression test" workflow. — [fast-check Tips](https://github.com/dubzzz/fast-check/blob/68a4c78741d23f068febe8b2816474726133882c/documentation/1-Guides/Tips.md)
- Reanimated test setup advances on a configurable fps (default `{ fps: 60 }`), so animation-driven logic can be stepped deterministically in Jest. — [Reanimated testing](https://docs.swmansion.com/react-native-reanimated/docs/guides/testing/)
- The deprecated-packages table lists `AsyncStorage` → `expo-sqlite/localStorage/install` (community skill, secondary). — [upgrading-expo skill](https://skills.cat/skills/rakaadi/agent-kit/upgrading-expo)

### Inferences
- **Pure engine core:** `step(state, input, dt)`, with all randomness from an injected seeded PRNG (e.g. mulberry32 or sfc32) and time from an injected clock. No `Math.random`/`Date.now` in the engine, enforced by an ESLint `no-restricted-globals` rule in `packages/engine`.
- **Fixed timestep:** a 1/60 s accumulator for gameplay logic. Rendering (R3F `useFrame`, Reanimated) interpolates and never mutates game state.
- **Event log:** record `{seed, appVersion, contentVersion, events[{t, type, payload}]}` per session in a ring buffer, and attach it to error reports (no PII). A `replay(log)` dev tool and a Vitest helper turn any bug report into a failing test.
- **Saves:** a versioned zod schema (`v: number`) with a migration chain `migrate[v→v+1]`, tested by fast-check (`arbitrary old save → migrate → validates against latest`). Write atomically: write `save.tmp`, fsync/rename (expo-file-system) or one SQLite transaction. Keep the last N good saves, and on parse failure fall back to the previous good one, never a blank profile. Sync to D1 with last-writer-wins plus a monotonic `rev`.

### Gaps
- I found no 2026 source specific to RN game-loop determinism or Reanimated/R3F fixed-step patterns. Engineering blogs such as Gaffer On Games "Fix Your Timestep" were not retrieved.

## 4. AI-agent coding safeguards

### Takeaway
Use small PRs gated by CI, with test-first for logic. Mutation testing (Stryker, incremental on PRs) catches weak tests the agent writes. Pin versions and run expo-doctor and `eas fingerprint:compare` to catch hallucinated or mismatched native APIs and OTA-unsafe changes.

### Cited Findings
- StrykerJS incremental mode (since 6.2, `--incremental` or `incremental: true`) mutates only changed code but still produces a full report. Limitations: it ignores changes in files other than mutated and test files, and detects mutant changes but not test changes. Use `--force` to rerun. — [Stryker incremental docs](https://stryker-mutator.io/docs/stryker-js/incremental/)
- Vitest setup: `@stryker-mutator/core` + `@stryker-mutator/vitest-runner`, `testRunner: "vitest"`, `coverageAnalysis: "perTest"`. Suggested CI: incremental on PRs, full `--force` nightly, which needs `fetch-depth: 0`. These are community sources. — [oneuptime Stryker (Jan 2026)](https://oneuptime.com/blog/post/2026-01-25-mutation-testing-with-stryker/view); [helpmetest Stryker guide](https://helpmetest.com/blog/stryker-js-guide/)
- Expo upgrade sequence: install the latest `expo`, run `npx expo install --fix`, run `expo-doctor` (diagnoses dependency mismatches and config issues), then clear caches and reinstall. — [upgrading-expo skill](https://skills.cat/skills/rakaadi/agent-kit/upgrading-expo); [agentskillsfinder upgrading-expo](https://agentskillsfinder.com/skills/upgrading-expo)
- `eas fingerprint:compare --build-id <ID>` checks whether a change touches the native surface (deps, config, native code), which tells you whether it is OTA-safe. — [Expo blog: 5 OTA best practices (Feb 2026)](https://expo.dev/blog/5-ota-update-best-practices-every-mobile-team-should-know)

### Inferences
- **CLAUDE.md rules:**
  - Read the docs for the installed version (`node_modules/<pkg>/package.json` plus its README or types) before using an API.
  - Never add a dependency without `npx expo install`.
  - One concern per PR (aim for under 400 changed lines).
  - Logic changes start with a failing test.
  - Run `pnpm verify` (typecheck + lint + unit + content validate + web screenshots) before declaring done.
  - Attach screenshots for any UI change and look at them (Read the PNG) before claiming success.
  - Never update snapshots or baselines without stating why in the PR.
- **Anti-cheating guards:** CI fails if a test file is deleted or skipped (`.skip`/`.only` lint rule). Snapshot or baseline updates need a label. Stryker mutation-score threshold (e.g. ≥80% break) on `packages/engine`, `economy` and `save-migrations` only.
- **Hallucination guard:** `tsc` against real type definitions catches most invented APIs. `expo-doctor` and `expo install --check` catch version drift. Pin exact versions and commit the lockfile.
- **Dependency hygiene:** Renovate grouped by "expo SDK" (bumped only via `expo install --fix`), with automerge for devDeps patch updates only when green. Native-dep bumps never automerge because they need a new dev build.
- **Self-review checklist** in the PR template: RTL checked, both genders, the Hebrew string with nikud renders, offline behavior, error path, backgrounding, accessibility labels, no new `any`.

### Gaps
- No 2026 primary source was found on Claude Code-specific best practices. Anthropic docs were not searched within the budget.

## 5. Content QA (packs, nikud, gender variants, audio, math, reading level)

### Takeaway
No direct sources were found. Treat content as code: a zod-schema'd pack compiled by a CLI validator in CI with property-style checks.

### Cited Findings
- (Adjacent) fast-check arbitraries can generate inputs to recompute and verify answers. — [fast-check docs](https://fast-check.dev/docs/advanced/model-based-testing/)

### Inferences
- **`content:validate` CLI** (Vitest suite over every pack) checks the following:
  1. zod schema plus unique IDs and valid cross-references.
  2. Every user-facing string has `{m, f}` (and optionally `neutral`) variants, or is explicitly marked gender-invariant.
  3. Nikud presence on early-reader strings: regex for Hebrew points U+05B0–U+05C7 (U+05B0–U+05BD plus U+05C1, U+05C2, U+05C7). Set a min ratio of pointed letters per word for kid-reading levels and flag mixed Latin/Hebrew bidi risks.
  4. Every audio reference exists, has a correct duration, and has loudness within target (e.g. ffmpeg `loudnorm`/ebur128 at about −16 LUFS ±2), plus no clipping.
  5. Math answers are recomputed by the engine from the question spec, and distractors are never equal to the answer.
  6. Reading level: word count and syllable/letter heuristics per level, plus an allow-list vocabulary per age.
  7. Image assets exist, with size budgets.
- **Snapshot of all rendered strings** per gender × profile, as a text snapshot (`strings.m.snap`, `strings.f.snap`), so diffs in PRs are human-readable for the parent.

### Gaps
- No sources were found on automated Hebrew nikud validation or Hebrew reading-level metrics. This needs a domain-specific search.

## 6. Crash & error monitoring (kids/family privacy)

### Takeaway
Sentry's RN/Expo SDK offers `beforeSend`, `beforeSendTransaction` and `beforeBreadcrumb` hooks plus server-side scrubbing. For a family-only app, a minimal self-hosted error sink on Workers/D1 is a viable, more private alternative or complement.

### Cited Findings
- Sentry RN/Expo `beforeSend` runs before an error event is sent and can strip fields (their example deletes `event.user.email`). `beforeSendTransaction` does the same for transactions, and transaction names can contain PII such as IDs in URLs. Breadcrumbs, HTTP spans and query strings can also carry PII, filtered via `beforeBreadcrumb`. Server-side scrubbing exists in the Sentry UI, or a local Relay keeps data from reaching Sentry. — [Sentry RN Expo: sensitive data](https://docs.sentry.io/platforms/react-native/guides/expo/data-management/sensitive-data/); [Sentry RN advanced scrubbing](https://docs.sentry.io/platforms/react-native/data-management/sensitive-data/advanced-datascrubbing)
- User-context behavior is governed by `send_default_pii` (native SDK docs). The RN default could not be confirmed. — [Sentry Native: sensitive data](https://docs.sentry.io/platforms/native/data-management/sensitive-data/)

### Inferences
- Keep `sendDefaultPii: false`. Set the user as an opaque profile hash only, never a child's name. Disable session replay, or keep it with full masking. Strip breadcrumbs of text input values.
- **Alternative:** a Workers `/errors` endpoint, zod-validated and rate-limited, storing `{ts, appVersion, updateId, runtimeVersion, platform, screen, error, stack, eventLogTail}` in D1. Upload source maps per EAS Update so stacks are symbolicated. A daily digest goes to the parent.
- **Error boundaries:** per-screen boundaries with a kid-friendly recovery screen ("Oops, the bunny tripped. Let's go back to the map") that never loses progress. On recovery, restore the last good session checkpoint from the state machine plus save. Add a separate boundary around each 3D/Skia/Rive canvas so a GL failure falls back to a 2D view.

### Gaps
- Sentry's mobile privacy FAQ and COPPA guidance were not retrieved (fetch blocked). For a family-only, non-store app, COPPA applicability is probably limited, but this was not verified.

## 7. Release gates, OTA rollout, rollback, free-signing expiry

### Takeaway
Use EAS Update percentage rollouts or a one-device channel, plus `eas update:rollback` and `roll-back-to-embedded` for rollback, gated by fingerprint checks. With a free Apple ID, profiles expire about every 7 days, so monitor and rebuild weekly, or buy a paid account.

### Cited Findings
- `eas update --rollout-percentage N` starts a rollout. `eas update:edit` changes the percentage. `eas update:revert-update-rollout` reverts and republishes the control update. After 100%, use `eas update:rollback`, which rolls back either to a previous update or to the embedded update. Only one rollout can be active per branch/platform/runtime. `--force-end-active-rollout` works with `roll-back-to-embedded`. Publishing after a rollback reaches all clients. — [Expo docs: Rollouts](https://docs.expo.dev/eas-update/rollouts/); [Expo docs: Rollbacks](https://docs.expo.dev/eas-update/rollbacks)
- Sources conflict on runtime version policy. Expo docs say the `appVersion` policy only protects you if you bump it when native code changes, while `fingerprint` "make[s] incompatible updates extremely unlikely" at the cost of more builds. Expo's Feb 2026 blog recommends `appVersion` plus `eas fingerprint:compare` as a detection tool, and community posts favor `fingerprint`. — [Expo runtime versions](https://docs.expo.dev/eas-update/runtime-versions/); [Expo blog 5 OTA practices](https://expo.dev/blog/5-ota-update-best-practices-every-mobile-team-should-know); [dev.to: OTA is a loaded gun](https://dev.to/mason_roy/expo-ota-updates-are-a-loaded-gun-heres-the-safety-1c1k)
- A free (Personal Team) provisioning profile expires after 7 days, versus a year for paid accounts. Apps stop launching after expiry, and rebuilding from Xcode reinstalls them without data loss unless you uninstall. Limits cited are about 3 apps per device and up to 10 App IDs per 7 days, though the figures vary by source. — [Apple Dev Forums: profile expiring in 7 days](https://developer.apple.com/forums/thread/69248); [Apple Dev Forums 724896](https://developer.apple.com/forums/thread/724896); [mybyways: free account limits](https://mybyways.com/blog/new-limitations-imposed-on-free-apple-developer-account)

### Inferences
- **Preflight before a dev build or OTA:**
  1. `pnpm verify` is green.
  2. `expo-doctor` is clean.
  3. `eas fingerprint:compare` shows no native change; if there is one, you need a new build, not an OTA.
  4. Content validation passes.
  5. Web screenshot diff is approved.
  6. The Maestro smoke suite passes on the Mac simulator.
  7. Release notes are written for the parent.
- **Staged rollout for a family:** a `canary` channel is installed on one device (the parent's iPad) and soaks for 24 hours. Then promote via `eas update:republish` to the `kids` channel, or use `--rollout-percentage` if there are several devices.
- **Rollback:** use `eas update:rollback` to the previous update, or `roll-back-to-embedded` if the update corrupts startup. Keep `expo-updates` error recovery on. The app should report `updateId` with every error so a bad update is identified fast.
- **Signing expiry monitoring:** store the build date in the app and on the Worker. A scheduled task or Worker cron warns the parent on day 5 or 6. The app shows a gentle parent-only banner when `buildAge > 5d`. The $99/year paid account removes the weekly churn and enables TestFlight/ad-hoc builds. That is strongly worth it if feasible.

### Gaps
- The Expo production OTA playbook blog could not be fetched. Expo-updates crash auto-rollback behavior (`checkAutomatically`, error recovery) is not confirmed for SDK 58.

## 8. Common RN/Expo pitfalls in 2025-2026 and early detection

### Takeaway
The main risk areas are New Architecture-only builds breaking legacy native libs, SDK upgrade drift (removed or split modules), and RTL. RTL `forceRTL` needs a reload, and native elements don't flip automatically.

### Cited Findings
- The New Architecture is on by default, so `newArchEnabled` is redundant. Expo Go has supported only the New Architecture since SDK 53. Check each native dependency for New Architecture support before upgrading. — [upgrading-expo skill](https://skills.cat/skills/rakaadi/agent-kit/upgrading-expo); [rapidnative Expo upgrade planner](https://www.rapidnative.com/tools/expo-sdk-upgrade-planner)
- Deprecated modules: `expo-av` → `expo-audio`/`expo-video`; `AsyncStorage` → `expo-sqlite/localStorage/install`. — [openskillindex upgrading-expo](https://openskillindex.com/skills/eddiebelaval-squire-upgrading-expo)
- RTL: `forceRTL` only sets a native flag that applies after reload (no way around it on Android). The forceRTL delay is undocumented and not meant for production, and `isRTL` on iOS is "often misread". — [RN PR #51648](https://github.com/react/react-native/pull/51648); [RN PR #51661](https://github.com/facebook/react-native/pull/51661); [intlayer RN i18n](https://intlayer.org/en/blog/i18n-technologies/frameworks/react-native)
- RN auto-flips `flexDirection: row` in RTL. Most bugs come from hard-coded left/right values, so use logical start/end. — [vp0 RTL flexbox fix](https://vp0.com/blogs/react-native-rtl-flexbox-layout-fix-ai/)
- `forceRTL` changes only React views. Native elements such as the react-native-screens nav bar need `semanticContentAttribute`. — [react-native-screens #1884](https://github.com/software-mansion/react-native-screens/issues/1884)
- Expo RTL setup with i18next + I18nManager: Android works after one reload, and iOS below RN 0.79 needed a patch. — [GeekyAnts Expo RTL guide (Apr 2025)](https://geekyants.com/blog/implementing-rtl-right-to-left-in-react-native-expo---a-step-by-step-guide)

### Inferences
- Since the app is Hebrew-only, set RTL at native config level so it is RTL from first launch. Do not toggle at runtime. The expo-localization config plugin has `supportsRTL`/`forcesRTL`, but verify the plugin keys.
- **Lint ban** on left/right styles. Add a Playwright RTL screenshot suite plus one Maestro screenshot flow in the simulator, since web RTL ≠ native RTL.
- Directional icons (arrows, "next") need explicit mirroring. R3F, Skia and Rive canvases do not mirror, so handle coordinates in game space.
- **Early detection:** a weekly CI job that runs `npx expo install --check` and `expo-doctor`, plus a canary branch on the next SDK beta. Renovate surfaces upgrades. Keep a pinned list of New Architecture-verified native libs (Reanimated/Worklets, Skia, Rive, gesture-handler, screens) in CLAUDE.md.

### Gaps
- No primary source was found for Expo SDK 58 / RN 0.88 specific breaking changes. The Expo changelog was blocked. Rive RN and R3F-native New Architecture status in late 2026 is unverified.

## 9. Deliverables: playbook rules, test matrix, CI pipeline, DoD, triage, parent-device needs

### Takeaway
A Linux-only GitHub Actions pipeline can cover everything except native rendering, native RTL, performance, audio, haptics and real-device gestures. Those need the parent's Mac simulator (Maestro) and the iPad, reached through Remote Control.

### Cited Findings
- Visual baselines from Docker in CI. — [Argos](https://argos-ci.com/blog/playwright-visual-regression-testing-ci)
- Workers tests in real workerd with D1 migrations. — [Cloudflare Vitest integration](https://developers.cloudflare.com/workers/testing/vitest-integration/configuration/)
- Stryker incremental on PR, full nightly. — [Stryker incremental](https://stryker-mutator.io/docs/stryker-js/incremental/); [oneuptime](https://oneuptime.com/blog/post/2026-01-25-mutation-testing-with-stryker/view)
- Maestro on iOS needs macOS + Xcode simulators. — [Maestro iOS docs](https://docs.maestro.dev/get-started/supported-platform/ios)

### Inferences
**Test matrix (what / tool / where / when)**

| Layer | Tool | Runs on | Trigger |
|---|---|---|---|
| Engine, economy, adaptive logic, save migrations | Vitest + fast-check (property + `fc.commands`) | Linux CI | every PR |
| Mutation | Stryker vitest-runner, incremental | Linux CI | PR (incremental), nightly full |
| Content packs | `content:validate` (zod, nikud, gender, audio LUFS, math recompute) | Linux CI | every PR touching content |
| Components | jest-expo + RNTL + Reanimated setUpTests | Linux CI | every PR |
| Visual/RTL/gender | Playwright (Docker image) on `expo export -p web` | Linux CI | every UI PR |
| API contract + Workers + D1 | vitest-pool-workers + Hono testClient + shared zod | Linux CI | every backend/contract PR |
| D1 migrations | applyD1Migrations + fixture upgrade tests | Linux CI | every migration PR |
| E2E golden paths (5–10 flows) | Maestro YAML on iOS Simulator | Parent's Mac (Remote Control) | before each dev build or OTA |
| Real device: perf/FPS, 3D, audio, haptics, touch, native RTL, Rive/Skia | Manual checklist + screenshot | Parent's iPad | before promoting canary → kids |

**CI pipeline outline (GitHub Actions, ubuntu-latest)**
1. `setup`: pnpm with frozen lockfile and cache.
2. `static`: `tsc -b`, eslint (zero warnings), `expo install --check`, `expo-doctor`.
3. `unit`: Vitest with coverage and fast-check (fixed `seed` printed on failure).
4. `content`: validate plus string snapshots per gender.
5. `workers`: vitest-pool-workers (D1 + migrations).
6. `web-visual`: Expo web export, then Playwright in `mcr.microsoft.com/playwright:<pinned>`, uploading the diff artifacts.
7. `mutation`: incremental with `fetch-depth: 0`. Nightly full.
8. `fingerprint`: `eas fingerprint:compare`, labeling the PR "needs-native-build" vs "ota-safe".
9. On main: `eas update --channel canary`. Manual approval promotes to `kids`.

Renovate and the weekly `expo-doctor` run are scheduled jobs.

**Definition of Done**
- Types, lint and tests are green.
- New logic has tests written first. Bug fixes include a regression test, from the replay log or seed.
- Both genders and all affected profiles are screenshot-reviewed.
- RTL is verified.
- Offline, background and resume paths are handled.
- An error boundary covers new screens.
- Content validates.
- Fingerprint classification is noted.
- The PR is under ~400 lines with a self-review checklist ticked.
- No new deps without `expo install` and a New Architecture check.
- If UI changed, a simulator or iPad screenshot was obtained before the canary is promoted.

**Bug triage process**
- **Sources:** the parent's report (via a "report a problem" parent-gate button that attaches the event log, updateId and a screenshot), the Workers error sink, and CI.
- **Severity:**
  - S0: crash or lost progress or a stuck kid. Roll back the OTA immediately, then hotfix.
  - S1: wrong answer, economy bug or broken RTL. Fix within the next release.
  - S2: visual polish.
  - S3: idea.
- **Fix loop:** reproduce from the replay log in Vitest first, then fix, then add a property or regression test, then canary.
- Track the work in GitHub Issues with labels `sev:*`, `area:*` and `needs-device`.

**What needs the parent's device**
- Native builds and signing (Xcode, weekly free-profile refresh).
- Maestro iOS runs.
- Real-GPU performance (R3F/Skia/Rive frame rate, thermal).
- Audio playback and loudness on real speakers.
- Haptics, multitouch and Apple Pencil.
- Native RTL (nav bars, system sheets, keyboard).
- Hebrew font and nikud rendering on iOS.
- Safe areas and iPad split view.
- OTA install, rollback, and offline/first-launch checks.
- Kid usability sessions.

### Gaps
- The CI steps are synthesized, not taken from an official reference pipeline.
- I didn't verify the GitHub Actions specifics for running `eas fingerprint:compare` without a Mac or with EAS credentials in CI.
