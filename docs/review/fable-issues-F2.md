# Fable review — cross-file issues from F2 (owner of `01-game-design.md`, `02-economy-progression.md`)

Each row: file · section · problem · proposed fix. 01/02 have already been edited to the position described; the owning reviewer applies the fix in their file. IDs: `MX-*` are 01 §7.1/§7.3; `UC-nn` are 01 §21 / 02 §18.

## A. Mechanic IDs and mappings other specs must adopt

| # | File / section | Problem | Proposed fix |
|---|---|---|---|
| 1 | 03 §5.4 | New mechanics were "proposed to 01" without IDs. 01 §7.3 now defines them. | Map: `MECH.BALANCE`→`MX-M13`, `CLOCK_CAMP`→`MX-M14`, `MAKOLET`→`MX-M15`, `MIRROR`→`MX-M16`, `NUMBER_TALK`→`MX-M17`, `DUEL`→`MX-M18`, `TANGRAM`→`MX-M19`, `SCOUT_CHART`→`MX-M20`; variants: `TAP_COUNT` lasso = `MX-M03:lasso`, `TENFRAME_CART`/`HIDE_HUT` = `MX-M04:cart`/`MX-M04:hut`, `TABLE_MAP` = `MX-M07:table-map` (progress screen), `PATH_RACE` = `MX-M12:board-game`. Use these IDs in `skills.json` `MECH.*` references (or keep `MECH.*` and add the `MX` column in §5.4). |
| 2 | 03 `MECH.MAKOLET` | Play shekels could be read as a currency. 01 MX-M15 + 02 §2.7 make them a level prop outside the wallet (00 D1 allows learning *about* money). | State in 03 that shekels never enter the economy (cite 02 §2.7, AC-ECO-5). |
| 3 | 05 §20.2 | Requests to 01 are all **accepted**: MX-E01 `pairs` mode, MX-E05 `lighthouse-lights`, MX-E06 `ship-log`, MX-E07 `dock-talk`, Pip = **CH-11** (silent crab), "bottles from the sea" = S0 daily-surprise slot, 2–3/week, 3–5 min (01 §13.2, §5.3 `english_bottle`). | Update 05 §20.2 statuses to "accepted" and reference the IDs; T09 `echo_pairs` = `MX-E01:pairs`. |
| 4 | 05 §12 / 12 | MX-E01 `pairs` needs the 4-adult-speaker pool + human recording per word (05 §12). 01 AC-MX-8 asserts rotation. | 12: add the 4-voice minimal-pairs bank (~70 words × 4) to the English production plan with OPD-15. |
| 5 | 07 §2.2–2.4, §12 | 01 adopted: `TOOLBELT` = `MX-T01` (unified card, states seen/tried/used/chosen/transferred), `BRAIN_TOKEN` = `MX-T06` (a gate across every hint ladder; 07 AC-07 implemented once in `packages/engine/hints/brainToken.ts`), `RIT-OPEN/MID/CLOSE` = 01 episode segments (`arrival_plan`, `ritual_mid`, `goodnight_tell`; open+close ≤10%). | 07: reference the MX IDs and the segment names; confirm `RIT-MID` frequency (K2 3/5 days, K1 daily) matches 01 §5.3. |
| 6 | 07 §0.5 | Per F1 #22 and 00 G4: the machine family keeps gears/screen-face/"AI • מכונה"; 01 §3.1 now bans those traits and the word "robot/bot" for the hero and renames `antenna`→`topper`. | 07: one line citing G4; make sure no 07 card calls the hero a bot. |
| 7 | 06 §4.5 | 01 adopted the hooks: `blicket` skins on `MX-X02`, `build-sim` = `MX-X05`, weather station = `weather_tap` segment + `MX-M20` chart renderer, camp lantern = `campfire_lantern_lit` event (light only, no logs/coins), garden never wilts. | 06: cite the IDs; the lantern is **not** a campfire-log source (02 §2.1). |
| 8 | 08 §5 / `packages/shared` | 02 §4.1 now defines `retained` = `master` + 30-day rung passed and expects an event `review_rung_passed{skillId, rungDays}` from 08; also `skill_state_changed` is derived through `packages/shared/levels.ts` (00 C15 mapping). | 08: emit `review_rung_passed` on every rung pass (at least 30 and 60) and export `LevelId` + `toEconomyState()` from `packages/shared/levels.ts`. |
| 9 | 08 §4.4 / 02 §2.6 | Rapid-guess rule: 02 pays 0 for `rtMs < 800` wrong answers (K1, non-fluency). 08 owns RT; make sure the engine exposes `rtMs` on `item_answered` for K1 only and `null` for gan (`grade < g1`). | 08: `rtMs: number | null`, null below g1. |

## B. Parent area / UX

| # | File / section | Problem | Proposed fix |
|---|---|---|---|
| 10 | 10 §6.13 | 01 aligned caps to 10 (K3 15 min, bedtime K2 20:00). 01 DD-09 defines a parent **"+10 min" bonus** (behind the gate, 1/child/day, logged, extends minutes only) and §15.1 defines **mid-session cap-change** behaviour (applied at the next segment boundary). | 10: add the "+10 דקות" control to PA-CSET or the device lock screen; document that cap edits apply at the next segment boundary (no immediate cut). |
| 11 | 10 §6.13 "מצב מדריך" | 01 §12.6: parent toggle (default off) → per-session child opt-in; cap 1/day ≤10 min; parent summary = counts of good prompts only. | 10: cite 01 §12.6; W-CO summary shows "שאלה 'איך ידעת' N פעמים", never K2's accuracy. |
| 12 | 10 SET-01 / 01 §15.4 | Mode IDs must match: 10's (מנוחה / נעילה לילדים / כבוי) = 01/00 `soft` / `full` / `off`, default `soft`. | 10: show the config ids next to the Hebrew labels; default `soft` (OPD-04). |
| 13 | 10 (new) | 01 DD-31 travel: banner when `deviceTz ≠ family.tz` for >24 h with a "עיר נסיעה" setting for Shabbat times. | 10: add the banner + setting (SET-01). |
| 14 | 10 (new) | 01 DD-37 / 02 UC-21: from day 5 of a free-signing install the parent area shows "סנכרון: N אירועים ממתינים" and the expiry banner (11 §4). | 10: add the unsynced-events counter to the device card. |
| 15 | 10 M-14 | 01 UC-07: a K3 lap segment with no parent tap for 60 s switches to the playroom; the "solo minutes in learning segments" metric should count exactly those switches. | 10: define M-14's K3 solo metric from `k3.parentPromptTimeout` events. |
| 16 | 09 §10 / components | 01 UC-19..22 rely on: chest shapes round/shield/star/crown; evening flame icons (snowflake/leaf/star/question mark); a `dragOrTapTap` input mode on every drag; chest auto-open setting; captions default on for K1; counting voice always with numerals. | 09: add the flame icon set and `dragOrTapTap` to the component contract; put "פתיחה אוטומטית של תיבות" under accessibility settings. |
| 17 | 09 (receipt) | 01 §16.2: effort marks (`firstTry`, `hintFree`, `braveChoice`, `mentor`, `movedBody`) are **receipt lines**, not taps; need phrasing in the praise sets (process language only). | 09: add 5 receipt-line templates ×{m,f,pl} ×K1/K2. |
| 18 | 09 §4 | New mechanics need target sizes/gestures: MX-M14 rotating hands (drag along an arc, snap 5 min), MX-M19 rotate-by-tap (no two-finger rotate), MX-M16 Skia draw with 8 mm tolerance. | 09: add the three gesture specs. |

## C. Tech

| # | File / section | Problem | Proposed fix |
|---|---|---|---|
| 19 | 11 §5.8 / §6.4 | New economy events (02 §2.1): `review_rung_passed`, `chest_opened`, `receipt_shown`, `episode_closed`, `season_tier_reached`, `brain_token_deposited`, `tool_state_changed`, `campfire_lantern_lit`. | 11: add zod schemas (cls `learn`/`econ`), `v:1`. |
| 20 | 11 §6.4 | 02 §2.1 requires **`DayKey` monotonic per device** (`max(DayKey(ev), lastDayKeyOnDevice)`) and `DayKey` from `ts + tzOffsetMin`. | 11: implement in `packages/shared/dayKey.ts`; store `last_day_key` in `device_self`. |
| 21 | 11 §4.8 / §6.5 | 02 UC-15: the Sunday savings bonus is paid only when `lastServerTime ≥ weekStart` or the device has been unsynced ≥7 days; 01 UC-15: Shabbat window prefers `serverTime` when the last sync is <24 h old. | 11: expose `lastServerTime` to the engine; sync at episode end and on app background (already implied, make explicit). |
| 22 | 11 §4 / 13 | **`⟂VERIFY`** (01 DD-37, 02 UC-21): does a re-signed build installed over the same bundle id with `devicectl` preserve the SQLite data container? If not, `tools/mac/resign.sh` must trigger "סנכרון עכשיו" (parent area) before install. | 11/13: add the test to the re-sign routine checklist; add a `preInstallSync` step if needed. |
| 23 | 11 §6.5 Durable Objects | 01 DD-34 / UC-17: the CO-3 room's DO emits `boss_defeated{participants}` **once** with one id to all devices; a device that rejoins within 15 min resumes its role. | 11: specify the DO's event emission and the rejoin window (01 §20 #1 is still a proposal to 00 E5). |
| 24 | 11 (telemetry) | 01 DD-38 `asset.missing`; 01 Appendix B unchanged otherwise. | 11: add `asset.missing{assetId, kind}` (cls telemetry). |
| 25 | 12 | New content needs: Pip (CH-11) character sheet, bottles audio (Sunny song + 2–3 words per bottle, ~15 bottles for S0), MX-E05 tricky-word clips, MX-E06 decodable pages, MX-E07 dialogue trees; 4-voice minimal-pairs bank; hero prompts must say "soft toy lantern", never robot (AC-HERO-1 also lints `tools/`). | 12: add to the production plan and prompt library. |
| 26 | 13 | 01 §7.1 build order: Tier A (MVP 11, unchanged), Tier B (20 by S1), Tier C (rest by S3); chunk budget 12 for A+B, 6 for C. | 13: reflect Tier B in the S0/S1 checkpoints; keep the VS scope unchanged. |

## D. For 00 (F1)

| # | File / section | Problem | Proposed fix |
|---|---|---|---|
| 27 | 00 E5 | 01 §20 #1 (Durable Objects for CO-3) is still "proposed". 01 UC-17 and 02 UC-17 depend on it; fallback is same-device co-op only. | Decide; if rejected, 01 §12.1 CO-3 is dropped from scope (01 already states the fallback). |
| 28 | 00 D4 | 01 §10.4 made `fireLevel` **unbounded** (rings every 10) because "1–10, never resets" would hit 10 in ~5 weeks at ~2 level-ups/week. 02 config `max: null`. | Confirm wording "campfire level is unbounded; the visual has 10 flame steps per colour ring". |
| 29 | 00 G2 / new | 01 §10.4 region-unlock rule: (calendar OR fire level, earliest) AND readiness — the calendar guarantees access; fire only opens early. | Record as the binding reading of G2 + D4 (one sentence). |
