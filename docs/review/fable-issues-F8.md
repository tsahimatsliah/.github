# Fable review — issues found by F8 (owner of spec 14) in files F8 does not own

Format: file · section · problem · proposed fix. Spec 14 (`docs/spec/14-performance-quality-playbook.md`) already applies
the "stricter value wins until resolved" rule for every conflict below; these notes ask the owners to align.

## For F6 — `docs/spec/11-technical-architecture.md`

1. **§4.7 vs §11.1 — fonts.** §4.7 says "Fonts are loaded before the splash screen hides (`useFonts`)", while §11.1's
   `app.config.ts` embeds fonts with the `expo-font` config plugin, and 09 §15 + 14 `PQ-R-41` require build-time
   embedding with **static** TTFs and no runtime `useFonts`. Also §11.1 lists `NotoSansHebrew-VF.ttf` and
   `Fredoka-VF.ttf` (variable fonts) whereas 09 §3.6 fixes Rubik (display), Noto Sans Hebrew (read), Andika (English)
   as static weights and says "not `fontWeight` on a variable family"; Fredoka is only a gate candidate.
   → Fix: §4.7 "Fonts are embedded by the `expo-font` plugin (no runtime load); the splash waits for DB migration,
   config cache and the picker assets, max 3 s". §11.1 plugin list = the static files in 14 `PQ-P-07`.
2. **§2.3 pinned list — missing `expo-battery`** (14 `PQ-R-57`/`PQ-P-15` use `useLowPowerMode`; verified in
   `expo-battery@58.0.2`). Also missing dev deps **`reassure` 1.6.1** (package name is `reassure`, not
   `@callstack/reassure`), **`@stryker-mutator/core` + `vitest-runner` 10.1.1**, `pixelmatch` (for the Maestro
   double-screenshot diff), and the Maestro CLI in `tools/mac/doctor.sh`. → Add rows.
3. **§2.3 — `react-native-worklets` 0.13.0 vs registry 0.13.1** (published 2026-11-01, i.e. < 7 days old, so 0.13.0 is
   correct under the 7-day rule today). Note it in the table so the next Renovate run isn't "fixed" by hand.
4. **§4.5 audio — `player.release()`** does not exist in `expo-audio@58.0.11`; the method is **`remove()`**. §4.5 also
   doesn't mention the first-class `preload(source)` / `clearAllPreloadedSources()` API, which 14 `PQ-P-06` uses.
   → Replace "released (`player.release()`)" with `remove()`; add `preload()` to the scene-mount step.
5. **§4.15 / §12.5 — Hermes allow-list floor.** The text relies on the playbook's "≥ 250829098.0.17". RN 0.88.1 bundles
   Hermes V1 **`260318099.0.5`** (`sdks/.hermesv1version`, `hermes-compiler` dep). → State the exact floor in
   `tools/ci/native-allowlist.json` and in §12.5.
6. **§4.15 — Reanimated flags.** `USE_COMMIT_HOOK_ONLY_FOR_REACT_COMMITS` is already the default (`true`) in 4.7.1;
   only `IOS_SYNCHRONOUSLY_UPDATE_UI_PROPS` is opt-in, configured via `package.json` → `"reanimated": {"staticFeatureFlags": {…}}`.
   → Mention the config key and that the flag A/B is an ADR (14 `PQ-R-06`).
7. **§4.15 nikud line-height "≥ 1.5×"** conflicts with 09 §3.6 tokens (1.6 for K2 read styles, 1.55 K1 pointed, 1.45 K1
   unpointed). → Point to the 09 tokens instead of a single ratio.
8. **§4.7 / §4.9 / §16 memory budgets (home 250 / gameplay 400 / peak 650)** vs 09 §11 (≤300 MB 3D, ≤200 MB 2D), 13 §10.3
   (≤300 MB) and the spike gate (≤300 MB). 14 `PQ-B-10…13` resolves with target/gate tiers (2D 200/300, 3D 300/400,
   peak 500/650). → Adopt the same wording ("target … gate …") so the three specs stop disagreeing.
9. **§12.2 viewports** (iPad Pro 11 1194×834, iPad mini 1133×744, iPhone 16 393×852) differ from 09 §13.2 (`ipad-11-l`
   1180×820, `iphone-se-p`, `ipad-pro-13-l` …). 09 owns UX; 14 uses 09's IDs. → Replace the §12.2 viewport row with
   "per 09 §13.2".
10. **§12.1 test table — add rows** for Reassure (every UI PR, gate: no added render / +10% duration) and make Stryker
    "nightly, blocking on `packages/engine` ≥ 80%" rather than "optional"; add the Maestro native layout-shift check
    (14 `PQ-T-06`).
11. **§11.5 `ci.yml`** — add jobs `reassure`, `pr-hygiene`, and workflows `nightly.yml` (already named), `baselines.yml`
    (`workflow_dispatch --update-snapshots` in the pinned image) and `rollback.yml` (`eas update:republish --group` /
    `roll-back-to-embedded`, agent-runnable for S0). Also add `tools/ci/calendar-guard.mjs` to `ota.yml`/`promote.yml`
    (no OTA Friday afternoon/Shabbat/holiday except `s0-hotfix`, 13 §10.2).
12. **§6.1 `doc` table** — 14 `PQ-P-11` writes a `kind='child_resume_bak'` previous-good copy in the same transaction. No
    schema change (kind is free text) but §4.16 "resume state" should mention the backup copy and the fallback order.
13. **§4.11 error handling** — add the per-**canvas** boundary (Skia/GL/Rive → 2D fallback) and the "one in-place retry,
    then hub" rule, and that the boundary releases the scene's audio players (14 `PQ-P-10`).
14. **§13 observability** — define crash-free without Sentry: `app.start{prevExitClean}` from a `boot_marker` in
    `kv-store` (14 `PQ-R-49`, `PQ-B-27`). Add `app.js_stall`, `app.ui_stall`, `app.memory_warning`, `app.degrade`,
    `app.boot_timeout`, `app.resume_fallback`, `app.perf.nav` to the telemetry event list.
15. **§11.1 `UIRequiresFullScreen: false`** — fine, but §4.15 should state the Split View/Stage Manager policy
    (letterbox a fixed logical resolution, 14 `PQ-R-47`) and the per-scene `expo-screen-orientation` lock.
16. **§4.9 spike gate #2 "memory ≤ 300 MB"** — keep as the *target*; the *gate* is 400 MB steady / 650 peak (14 `PQ-B-12/13`).
17. **§12.5** — reference the full 14 §8 protocol (IDs `PQ-DEV-01…13`) instead of the research note, including the
    interruption matrix and crash-log pull.
18. **§2.4 lint list** — add the new rules from 14 §3: `olam/no-layout-prop-animation`, `olam/no-blur-on-animated`,
    `olam/no-sync-io`, `olam/no-nondeterminism`, `olam/no-unsafe-cast`, plus `eslint-plugin-no-only-tests` and the CI
    scripts `test-delta.mjs`, `baseline-origin.mjs`, `bundle-budget.mjs` console check.

## For the owner of `docs/spec/09-ux-ui-design-system.md`

1. **§13.2 determinism hooks** use `window.__app.ready` and `window.__game.moveTo`; 11 §4.9/§12.2 use
   `window.__olam.ready` / `window.__olam.world.moveTo`. 14 treats `window.__olam` as canonical. → Rename in 09.
2. **§11 memory "≤300MB (3D), ≤200MB (2D)"** → state these as *targets*; gates are 400/300 (see 11 item 8).
3. **§11 Rive "≤300KB per file"** vs 11 "≤200 KB" → 14 uses 200 KB target / 300 KB max; say so.
4. **§3.11 implementation line `dirX(x) = I18nManager.isRTL ? -x : x`** contradicts 11 R-1 ("never reads
   `I18nManager.isRTL`") and the `olam/no-i18nmanager` lint. → `dirX` reads the `Dir` context (`useDir()`).
5. **§13.2 run tiers** — add the nikud torture-string shots at text scale 100/135/150 and the layout-shift (CLS = 0)
   measurement as `UX-AC-17`/`UX-AC-18`, matching 14 `PQ-T-04`.
6. **§3.6 typography** — add the rule "pointed text slots are fixed-size (`FixedSlot`), never `overflow:hidden` on an
   ancestor" explicitly as an acceptance item (it is in the prose; 14 `PQ-R-43` makes it a gate).

## For the owner of `docs/spec/13-delivery-roadmap-qa.md`

1. **§10.2 #4 and §17 CLAUDE.md** name the canary channel `family-preview`; 11 §11.3 names it `preview` (branch `main`)
   and the kids' channel `family`. → Align to 11.
2. **§10.1 test table** says "Jest, fast-check" for unit/property; 11 uses vitest for `packages/*` and jest-expo only
   for `apps/mobile`. → "vitest + fast-check (packages); jest-expo + RNTL + Reassure (mobile)".
3. **§10.1 / §10.3** use `window.__game` → `window.__olam` (see 09 item 1).
4. **§10.3 "no leak after 5 enter/exit cycles"** vs 11/14 "20 cycles, ≤ 20 MB growth" → use 20.
5. **§10.3 "memory ≤300MB"** → target; gate per 14 `PQ-B-12/13`.
6. **§17 CLAUDE.md "Commands"** lists Playwright viewports "iPad 1024×1366 p/l + iPhone 393×852"; 09 §13.2 uses
   1180×820 etc. → "per 09 §13.2". Also add a "## Performance & quality (spec 14)" section pointing to `PQ-AGT-01…12`,
   and `pnpm verify` as the done-gate (14 `PQ-AGT-06`).
7. **§9 DoD "ביצועים"** — add "Reassure delta attached for UI PRs; layout shift = 0 on changed screens (Playwright);
   perf-affecting PRs queued for the device protocol".
8. **§10.1 severity** — add the symptom mapping from 14 §9 (e.g. clipped nikud = S2, stuck machine = S1, lost
   resume = S0) so triage is mechanical.
9. **§3.2 weekly signing** says "Thursday evening" and a parent-triggered run; 11 §3.3 automates a daily launchd check
   at 18:40 with a ≤ 6-day cadence. Both can coexist (the daily job is the mechanism; Thursday is the parent's
   fallback), but say so.

## For F1 — `docs/spec/00-foundation-decisions.md`

See 14 §14 (additive): E4 (Reassure/Maestro/Stryker/pinned-Docker baselines/layout-shift gate), E3 (`expo-battery`,
`expo-screen-orientation`), E1b (draw calls 80 Low / 120 newer; budgets table in 14), new E12 (spec 14 binding; perf
judgements only on Release builds on the oldest device).

## For the owner of `docs/spec/12-ai-content-voice-pipelines.md`

1. Content validators (§1.6) should include the audio format rules from 14 `PQ-R-36` (short SFX as PCM WAV/CAF ≤ 150 KB,
   else AAC-LC; VO AAC 64–96 kbps mono; music AAC 128 kbps stereo) and the per-pack size budget (≤ 8 MB on disk for 2D
   packs), so `content:validate` and `lint:assets` agree.
