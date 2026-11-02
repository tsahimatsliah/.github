# Fable review — cross-file issues from F6 (owner of 09 and 11)

Format: file · section · problem · proposed fix. Decisions that 11/09 now bind are marked **bound**; the owning
reviewer aligns their file. All F1 (#8, #14, #15, #19, #27, #28) and F8 items addressed to 11/09 have been applied
in place; see the changelogs at the end of 11 and 09.

## For the owner of `13-delivery-roadmap-qa.md`

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 1 | §3.2, §16.2, §17 | Re-sign script is named `tools/device/resign.sh`; 11 §3.3 and 00 E1a/E11 name it `tools/mac/resign.sh`. | Rename to `tools/mac/resign.sh`. |
| 2 | §3.2 | Weekly signing is described as a fixed day the parent triggers (Thursday). 11 §3.3 automates a daily launchd check at 18:40 with a ≤ 6-day cadence; the parent-triggered "חתימה שבועית" is the fallback. Both coexist. | Say: "the daily launchd job is the mechanism; the Thursday reminder is the parent's fallback when the Mac was asleep." Keep OPD-07/Q-07 for the day. |
| 3 | §3.1 "Metro ו-OTA בלי כבל" | "אפשר גם להריץ `npx expo start --tunnel` מהקונטיינר" — the container cannot reach `exp.host`/ngrok (research log tech.md; 00 E13 not applied yet). | "Metro runs on the Mac (LAN) via Remote Control; a tunnel from the container only after 00 E13 is applied." |
| 4 | §10.2 #4, §17 | Canary channel named `family-preview`; 11 §11.3 / 00 E1: `preview` (branch `main`) → `family`. | Align to `preview`. |
| 5 | P0-E1-S3, §17 | Playwright viewports "iPad 1024×1366 p/l + iPhone 393×852"; 09 §13.2 (source) uses `ipad-11-l` 1180×820, `ipad-mini-l` 1133×744, `ipad-pro-13-l`, `iphone-16-p/l`, `iphone-se-p`. | "per 09 §13.2". |
| 6 | §10.1 / §10.3 | `window.__game` → `window.__olam` (11 §4.9, 09 §13.2, 14). Memory "≤300 MB" is the *target*; gates are 400 (3D) / 300 (2D) / 650 peak (11 §4.7, 14 PQ-B-10…13). "5 enter/exit cycles" → 20 cycles, ≤ 20 MB growth. | Update numbers and names. |
| 7 | §3.1 row B | Says the `dev` build "receives JS from Metro on the Mac"; also note that OTA for `family` is gated by `tools/ci/calendar-guard.mjs` (no OTA Friday afternoon / Shabbat / quiet day unless `s0-hotfix`), now in 11 §11.5. | Reference 11 §11.5. |
| 8 | §10 QA | 11 §4.18 defines 24 edge cases `E-01…E-24` with test ids and `tools/ci/edge-matrix.mjs`; 13's QA plan should list them as a required suite before C4 (E-01/02/03/05/06/16 at least) and before MVP (all). | Add a row to the QA matrix. |
| 9 | §3.3 #6 / §19 | Parent checklist should include the iPad rule "never delete the app to fix an expiry" and the iOS "Unable to Verify App" alert explanation (11 §3.3 expiry walk). | Add to the parent setup checklist. |

## For the owner of `10-parent-area-dashboard.md`

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 1 | §15.1 `reports` | 10 proposes `reports(id, family_id, kind, period_from, period_to, json, created_at)`; 11 §6.2 (bound) adds `send_at`, `sent_at`, `summary_he`, uses `id = family:isoWeek` and `body_json`. | Reference 11 §6.2; use `sent_at` (not a separate `deliveries` row) as the idempotency marker for the weekly email. |
| 2 | §15.3 crons | `*/15 6-12 * * *` and `*/15 * * * *` are implemented as **one** `*/15 * * * *` trigger with a window filter in code (11 §7.1; functionally identical, proposed to 00 as E5 wording). | Say "runs inside the 15-minute job". |
| 3 | §9 SEC-02 / AC-17 | Now bound in 11 S4: `disableDeviceFallback:true` on `shared_kids`/`single_child`, `false` on `parent_only`; guardian picker after Face ID; cooldown persisted across restarts. 10 already matches; just cite 11 S4 instead of "proposes". | Wording. |
| 4 | §4 M-01 | `focus.span` is bound in 11 §5.8 with `screenType ∈ {episode, world, hub, shop, base, collection, coop, move}` and "one event per visit ≥ 10 s". | Reference the enum. |
| 5 | §15.2 API-18 | `approvals` unifies packs/photos; 11 keeps `/content/packs/:id/:ver/approve|reject` and `/photos/*` as thin aliases that write the same `approvals` row (11 §7.10a). | Note the aliases. |
| 6 | §6.13 PA-CSET | Concurrent edits by two parents use **field-level LWW** with a toast "עודכן על ידי {name} לפני רגע" (11 §6.5 E-11). | Add the toast to PA-CSET's states. |
| 7 | §10.1 AL-11 | Add triggers now emitted by 11: storage full (`app.storage_full`), DB restored from cloud (`app.error{kind:'db_recovered'}`), pack `failed{asset_missing}`, re-sign overdue (`resign_overdue`). | Extend AL-11's trigger list. |

## For the owners of `02-economy-progression.md` and `08-assessment-engine-child-model.md`

| # | File / section | Problem | Proposed fix |
|---|---|---|---|
| 1 | 02 §2.1 `LearnEvent.t` | Engine event names are snake_case (`item_answered`, `friend_rescued`…); the wire registry in 11 §5.8 is dotted (`item.answered`, `world.rescue`…). 11 binds one mapping in `packages/shared/events.ts` (`toEngineEvent`). | 02: add a sentence "wire names per 11 §5.8; mapping in `packages/shared`". |
| 2 | 02 §2.1 | Earnings/chest contents are a pure fold of learn events (02's own rule); 11 therefore stores **no** `econ.earn`/`chest.earned` events — only user actions (`econ.spend`, `chest.opened`, `inventory.equip`, `wishlist.set`) and `econ.overdraft`. `chestId = hash(sourceEventId)`. | 02: confirm `chestId` derivation and that `applyEvent` is idempotent by source event id. |
| 3 | 08 §1.1 / §4 | 11 §5.8 `item.answered` payload = 02's `item_answered` fields + `rtMs`, `attempt`, `hintUsed`, `workedExample`, `isReview`, `stretch`, `embodied`. 08 should state which of these its RT filters and HSHS read, and that `EngineEvent` is produced by `toEngineEvent`. | 08: list consumed fields. |
| 4 | 08 §9 | `evaluateFlags` runs **on device** from local 14-day counters; the output doc `flags/<childId>` is stored encrypted to guardian devices (11 §9.8, 00 E6a/C8a); the nightly server run only re-derives non-sensitive parts. | 08: say the rule engine runs on device and emits a doc, not a D1 row. |
| 5 | 08 §1.1 | 11 stores `tier` ('K1'…'K3') on children and derives `Band` via `packages/shared/bands.ts` (00 A6); `skill_state.level` column uses 08's `LevelId`. | 08: export `LevelId` and `bandOf()` from `packages/shared` (same as F1 #4/#9). |

## For the owner of `12-ai-content-voice-pipelines.md`

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 1 | §5 diagram | Nightly cron "01:30 IL"; 11 §7.1 is `10 0 * * *` = 02:10/03:10 Israel. Batch polling every 15 min matches 11. | Use 02:10. |
| 2 | §1.6 validators | 11 §4.11 defines `vo.generic.<intent>` bundled fallback lines for a missing VO file at play time; the content linter must guarantee one generic line per intent exists in the bundled core pack. | Add to the validator list. |
| 3 | §8 | Companion vendor-down behaviour (529/timeout 20 s → scripted hint, `ai_log.flagged='vendor_down'`) and `$` cap behaviour are bound in 11 §4.11/§8.4. | Reference instead of restating. |

## For the owner of `01-game-design.md`

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 1 | §16.3 | Praise lines merged into 09 §9.2 as P19–P25 and banned words into 09 §9.3 (09 is the owner per 00 C4). | Point to 09 §9.2 ids; new lines go to 09 first. |
| 2 | §17 AC-WORLD/LV | 11 §4.1 route names: `world/[zoneId]`, `level/[levelId]`, `map`, `results`; 09 SCR ids. | Use these ids in acceptance criteria. |

## For F1 — `00-foundation-decisions.md` (proposals, non-blocking; also in 11 §18 #8–11)

1. **E5 crons:** one `*/15 * * * *` trigger (alert queue + Friday `send_at` + batch poll + expiry nudge) instead of two schedules — functionally identical.
2. **E5 CoopRoom:** add the 30-min hard cap next to "15 min idle".
3. **A3:** `UIRequiresFullScreen: true` on iPad (no Split View / Stage Manager resizing) — safety and test-matrix decision.
4. **E6a:** reference the key mechanism in 11 §9.8 (family X25519 keypair, private key only in guardian Keychains, QR transfer at S2).
5. **B9(c):** 09 §3.11 implements `chart.timeDirection` and the ⟂VERIFY; nothing else to decide.
