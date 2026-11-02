# Round 2 — reviewer R2 (owner of `01-game-design.md`, `02-economy-progression.md`)

Scope: every item in `docs/review/fable-issues-F1…F9.md` and spec 15 §3 whose target is 01 or 02. Dispositions: **applied** / **already resolved** (round 1 had done it; verified in the current text) / **rejected** (+ why) / **forwarded** (needs the owner of another file). Section numbers are the current ones (01 gained §7.4; 02 sections unchanged). Both files got a "### Round 2 changelog".

## 1. Issues targeting 01-game-design.md

| Source · id | Topic | Disposition | Where / note |
|---|---|---|---|
| F1 #1 | §15.4 NOAA → `@hebcal/core`, B7 window | already resolved | §15.4 first bullet, AC-SHAB-3 |
| F1 #2 | §11.2 numbers → pointer to 02; campfire no reset; pet by mastery | already resolved | §11.2 pointer + 9 requirements; §10.4; §4.12. Numbers that remain (§9.2/§9.3/§13.3) are mirrors that cite 02 triggers and match 02's config |
| F1 #21 | §19 OPD ids | already resolved | §19 |
| F1 #23 | §4 hero never "robot" | already resolved | §2.1, §3.1, AC-HERO-1 |
| F2 D27 (to 00) | Durable Objects for CO-3 | applied (00 accepted in E5) | §20 #1 marked ACCEPTED; UC-17, DD-34 (+30-min room cap per 11) |
| F2 D28 (to 00) | `fireLevel` unbounded | already resolved | 00 D4 "never resets, adds a ring"; 01 §10.4 + 02 config `max: null`. 02's stale "(1–10)" fixed (see 02 table) |
| F2 D29 (to 00) | Region unlock = (calendar OR fire) AND readiness | no 00 change needed | 01 owns regions (00 J1); §10.4 rule + AC-REG-1 stand |
| F3 #16 | MX-H12…H15 | already resolved | §7.1, §7.3 |
| F3 #17 | MX-M13…M20 | already resolved | §7.1 |
| F3 #18 | Touch zones + `assisted` | already resolved | §12.2, AC-CO-7 |
| F3 #19 | `hint:lens` | already resolved | AC-MX-11 |
| F4 #12 | Cast roles for 06/07 | applied | paragraph after §4.1 table (ניצי/דובדובה/ברגית/גלגלון/פרופסור למה; hats = props; `characterId` = `CH-*`) |
| F4 #13 | Mr. Mix-Up identity | applied | = CH-04 in island costume, **separate en-US male voice**, Hebrew voice never on the island (§2.4 week 6, §4.1 row, §4.5) |
| F4 #14 | Bottles phase in §2.4 + week-6 date | applied | §2.4 "בקבוקים מהים" paragraph; island opens week 6 from launch (week of 13.12, gate ~20.12) = 05 §15.1 |
| F4 #15 | 06/07 mechanic names without MX ids | applied | mapping table after §7.3 (modes/variants; `shelf-3`, `route-planner`, `passport`, `season-track`, `rule-builder` declared FS screens owned by 07) |
| F4 #16 / F9 GAP-31 | Check-in cadence | already resolved (07 now follows 01: G every other day); made explicit | §5.3 K2/K1/K3 rows + MX-S01: S every episode, G every other day (first episode of the day), T parent-led; ≤20 s, skippable. Spec 15's "07 wins: every episode" is stale — 07 §2.4 itself says G = every other day |
| F4 #17 | Family garden next to the campfire | applied | §10.4 (cosmetic, fed by `kindness_confirmed` only, no counter) |
| F4 #18 | `palette.english` vs 09 | applied | §2.3: 09's `region.englishIsle` + `subject.english = grape` own the tokens; `#19B7B0` withdrawn; Sunny stays turquoise + yellow (§2.2 row) |
| F6 #1 | §16.3 praise → 09 P19–P25 | applied | §16.3 intro + per-row 09 ids; new lines go to 09 first |
| F6 #2 | Route names in ACs | applied | AC-WORLD-1 cites `map`, `world/[zoneId]`, `level/[levelId]`, `results` and 09 SCR ids |
| F7 #16 | English asset-ID list | applied | new §7.4 (bottles, mx-e05/06/07, pairs ×5 voices, hrec, songs, Pip, Mix-Up, Wordlings) |
| F9 GAP-01 | FTUE once | applied | §14.2 `child_resume.ftueCompletedAt` rule, `ftue_completed` event, AC-FTUE-1 |
| F9 GAP-02 | Overtime vs 08 grace; +10 min event | applied | §15.1: 85% → `closing`; cap in a beat ≤1:30; 08's 60 s only inside `closing`; `cap_bonus_granted` (cls admin) in DD-09/§15.1/Appendix B; AC-STOP-2/5 |
| F9 GAP-03 | K3 cap 10 vs 15 | **rejected** | 10 §6.13 owns the defaults (01 says so explicitly, round 1); 15 min = 2 lap episodes (≤5 min each, 00 C3) + the 5-min solo playroom counted inside it (§15.1). A 10-min cap would make the playroom eat a lap episode. Parent range stays 10's |
| F9 GAP-04 | Bedtime defaults; pre-bedtime semantics | already resolved (defaults) + applied | §15.1: 10's 19:30/20:00/20:30; "no new episode 60 min before" kept (10 R5 owns); effective lock = earliest of child/device bedtime; `bedtime_lock` end reason |
| F9 GAP-08 | Chest taps = effort marks | already resolved | §16.2 withdrawn in round 1; 02 §6.1/§6.5 canonical |
| F9 GAP-12 | "No explicit placement test" vs 08 §3 | applied | §14.2 rewritten: 08's expeditions exist as story-wrapped `mode:'placement'` levels with 08 §3.6 UI rules; §14.3 week-1 table allocates E1–E3 (K2 Sun–Tue) / E1–E4 (K1 Sun–Wed); week-1 episode template; UC-02; AC-PLC-1 |
| F9 GAP-13 | Level `mode` field | applied | §6.6 `mode?: 'normal'|'placement'|'benchmark'|'review'|'boss'|'event'`; consumed by 02 §2.7 |
| F9 GAP-14b | Weekly boss reward | already resolved | §9.3: K1 silver + 1 gem (02 `bossDefeated`) |
| F9 GAP-14c | Candle offsets | already resolved | §15.4: hebcal city defaults + override |
| F9 GAP-14d | Shabbat starting mid-episode | applied | §15.4: window applies by `session.started`; 02 §10.3 `appliesBySessionStart` |
| F9 GAP-19 | Audio-only car mode | applied as **P2** | §15.1: toggle, playlist, dimmed "עצור" screen, no events, 0 cap minutes; 09 SCR-21 / 11 own screen + launch (forwarded) |
| F9 GAP-25 | "Back to camp" end reason | applied | §15.1: `session.ended{reason:'child_exit', secondsSinceLastError}` — 08 §1.10's canonical enum (its `child_exit` = 10's "childQuit"); pause does not end the session |
| F9 GAP-29 | iPhone hub orientation | applied | DD-20: 09 §8.1 wins — hub/map/shop portrait-first, game scenes landscape locked |
| F9 GAP-33 | Patience heart detection | applied | §12.5 `turn_yielded` rule (`parent_exit` < 80% planned + other child within 120 s, ≤1/child/day, no grant); AC-CO-8; Appendix B |
| F9 GAP-39 | Travel tz | applied | DD-31: `family_prefs.travelCity` (hebcal needs a city; 11's `travelMode` should read as this), `tz_mismatch` telemetry |
| F9 GAP-43 | K1 after goodnight | applied | §15.1 `camp_asleep` view state (collection + music only, no session/econ, 0 cap, auto-close 10 min); AC-STOP-5; 09 SCR-07 needs the state (forwarded) |
| F9 GAP-46 | CO-4 letter moderation | applied | §12.1: chips/numbers/pack pictures only, `coop_letter` doc; AC-CO-8 |
| F9 GAP-50 | Printing | applied | §15.1: `expo-print` → A4 RTL PDF → share sheet/AirPrint; printables static; 11 to add the module (forwarded) |
| F9 GAP-53 | 4th child | applied | §10.1: v1 = 3 children; FAM-03 blocks a 4th (10 to add the message) |
| F9 GAP-63 | Boss finale with absent child / QD boss day | applied | §9.3 "who counts" + QD/Yom Tov window move; DD-06; AC-BOSS-3 |
| F9 GAP-67 | Parent voice clips | applied | §4.10 + §14.1 point to 12 §6.11 (m4a ≤10 s, `family/{familyId}/voice/{speaker}/…`, `family-voice.json`, family pack, re-record replaces) |
| F9 GAP-30 (05/01) | Cap warning inside the island | applied | §15.1 + AC-MX-3: Hebrew warning deferred; Sunny's bye-bye + exit jingle first |
| F9 GAP-11b (01 DD-18/11) | K3 launch mode | applied | §15.3 + DD-18: `child_settings.launchMode` lap / coplayFirst (gate 1×/day, solo ≤5 min) / solo; change on parent confirmation only |
| F9 GAP-21 (02/11; 01 mirror) | Campfire level-ups vs DO | applied | §10.4: only the four events raise the level; CO-3 source = `CoopRoom`'s single `boss_defeated`; UC-17 |

## 2. Issues targeting 02-economy-progression.md

| Source · id | Topic | Disposition | Where / note |
|---|---|---|---|
| F1 #3 | §15 Shabbat default `soft` | already resolved | §15 #1 |
| F1 #4 | §4 C15 mapping | already resolved + extended | §4.1 table; now also imports `toEconomyState` (F5 #1) |
| F1 #24 | Savings bonus = only time source, `capPerWeek` in zod | already resolved | Appendix A validation rules |
| F4 #7 | `neverEarns` for D1a events | applied | §2.7 bullet; config `neverEarns` (+`checkinDone`, `kindnessConfirmed`, `braveryStepDone`, `journalEntry`, `wonderAsked`); zod rule; AC |
| F4 #8 | `braveryLadderComplete` gem; `SD.PASS.2` showcase chest | applied | §2.2 rows, §6.2 K1/K2 tables, config triggers: K1 silver chest {coins 0, gems 1, no item, never announced}; K2 badge + garden only; `makerProjectComplete` = offlineMission grant + unique showcase frame, shares the daily mission-chest cap |
| F4 #9 | `stretchAttempt` K1/S only | applied | §2.2 note; config `stretchAttemptEnabledFor: ["K1"]`; zod; AC |
| F4 #10 | `offlineMission` conditional on `mission.economy` | applied | §2.2 row; config `requiresMissionEconomy` |
| F4 #11 | Family garden ≠ logs pot | applied | §7.6 bullet; config `familyGardenInLogsPot: false` |
| F5 #1 | Plant icons = C15 five names; badges overlay; import `toEconomyState` | applied | §4.1 (import), §4.2 (זרע/נבט/שתיל/עץ/עץ פורח from `LEVEL_CHILD_HE`; bronze/silver/gold as overlay; downgrade keeps the badge) |
| F5 #2 | `skill_state_changed` → `LevelId`; `level_reverted` never claws back | applied | §2.1 union renamed `skill_level_changed{from,to: LevelId}` + `skill_level_reverted`; `item_answered.level`; §4.1 rule; invariant 14; AC; UC-09 text updated |
| F5 #3 | `profile.paused` → `effTarget = 0` | applied | §10.1 formula + bullet; config `pausedChildTarget: 0`; AC |
| F5 #4 | Shabbat `soft` zeroes grants regardless of UI; test | applied | §10.3 bullet; invariant 13; §14.1 "שבת" tests; `appliesBySessionStart` |
| F6 #1 | Wire names per 11 §5.8 | applied | §2.1 bullet (`toEngineEvent` in `packages/shared/events.ts`) |
| F6 #2 | `chestId = hash(sourceEventId)`; idempotent by source id; no stored earn events | applied | §2.1 bullet; §6.3 rules 1 and 7 (`chest_earned` is a derived fold output, only `chest_opened` persisted); config `chestIdDerivation`; 01 §16.2/AC-ECO-4 wording aligned |
| F9 GAP-09 | Neighborhood layout offline merge | applied | §7.7: `neighborhood_layout/<childId>` LWW per tile, displaced item → inventory + mailbox note; config `neighborhoodLayoutMerge`; AC |
| F9 GAP-10 | Savings bonus on negative min balance | applied | §7.5: base `max(0, minBalance)`; overdraft week pays 0; config `basis: minBalanceDuringWeekClamped0`; invariant 15; AC |
| F9 GAP-13 | Placement/benchmark never earns, keyed on level mode | applied | §2.1 bullet + §2.7; `levelModeNeverEarns`; `benchmarkLevel` in `neverEarns`; AC |
| F9 GAP-21 | Campfire level-ups only from DO summary | applied | §7.6 first bullet; config `levelUpSources`; UC-17 |
| F9 GAP-28 | "תן מדבקה" scope | applied | §2.4: K3 only; K2/K1 → appreciation letter (10 W-RWD-04); config `stickerButtonFor` |
| F9 GAP-37 | Mission chest cap + surfacing | applied | §2.2, §6.2: ≤1 chest/child/day (`offlineMissionChestsPerDayMax`), XP ≤3/day, surfaced next episode as mailbox letter + shelf chest; invariant 15; AC |
| F9 GAP-38 | Parent refund of equipped/placed item | applied | §7.3: unequip to previous, remove from grid, coins back to the spend jar, neutral mailbox note, no "taken away" animation; config `parentRefund`; AC |
| F9 GAP-42 | "Shop closed in quiet hours" | applied | §7.3: child "app sleeps" windows + bedtime only; the sign exists only in K1's `camp_asleep` view |
| F9 GAP-44 | Cap day boundary | applied (modified) | §10.1: `DayKey` rolls at **04:00** (`dayRolloverHour`), **in the event's `tzOffsetMin`** (device tz, round-1 rule kept for offline determinism; equals 00 C11's Asia/Jerusalem in the home case); **the week is derived from `DayKey`** (not Sunday 00:00 as spec 15 proposed) so there is one boundary rule for days and weeks; `packages/shared/dayKey.ts` is the single implementation (11) |
| F2 #28 leftover | §7.6 "רמת המדורה (1–10)" | applied | text now "בלי תקרה" |

## 3. Forwarded to other owners (only what blocks a consistent build)

| To | Item |
|---|---|
| 11 (F6) | Add to `packages/shared/events.ts`: `cap.bonus_granted{childId, minutes, grantedBy, dayKey}` (cls admin), `turn.yielded{childId, sessionId, nextChildId, plannedShare}` (learn), `ftue.completed`, `tz_mismatch` (telemetry); adopt 08's `SessionEndReason` enum in `session.ended` (11 §5.8 still has `'cap'|'switch'|'background'|…`); wire names `skill.level_changed{from,to: LevelId}` / `skill.level_reverted`. Doc kinds: `child_resume.ftueCompletedAt`, `neighborhood_layout/<childId>` (LWW per tile), `coop_letter`. Fields: `child_settings.launchMode ∈ {lap, coplayFirst, solo}`, `family_prefs.travelCity` (hebcal city; GAP-39's `travelMode` → this). `packages/shared/dayKey.ts`: rollover 04:00 in the event's `tzOffsetMin`; week derived from DayKey. Module list: `expo-print`. `CoopRoom` 30-min hard cap is cited by 01 DD-34 as 11's. |
| 09 (F6) | SCR-07: add the K1 `camp_asleep` state (collection room + music hut only; K2/K3 → gate). **§5.3 chest choreography still says "auto after 1.5 s without tap" (line ~596) — 02 §6.5 wins: animated hand at 4 s, auto-open only via the accessibility setting (DD-39).** Praise sets: add the "persistence" and "neutral chain escalation" rows from 01 §16.3 (P19–P25 cover the other seven). SCR-21 "מצב רכב" is P2 (01 §15.1). |
| 10 (F5) | FAM-03: block a 4th child with "בגרסה הזו יש מקום לשלושה ילדים"; "להראות את הפתיחה שוב" (clears `ftueCompletedAt`); W-ECO-06 "תן מדבקה" = K3 only, "החנות סגורה" = child windows/bedtime only; `launchMode` change on grade rollover = parent confirmation; PA-CSET "+10 דקות" emits `cap.bonus_granted`. |
| 08 (F5) | `item_answered` → 02 now reads `level: LevelId`, `rtMs | null`, `provenance`, `levelMode`, `assisted` (all already in 08 §1.4/`toEngineEvent` per F5; please confirm `levelMode` is populated from `EpisodePlan.mode`/`Session.mode`). `skill_level_reverted` payload `{skillId, from, to, sessionId}`. 08 §6.10 wording "60 s grace" is now explicitly "inside `closing` only" in 01 §15.1 — no change needed unless 08 disagrees. |
| 07 (F4) | 02 named the events `bravery_step_done`, `braveryLadderComplete` (K1 silver chest with 1 gem only; K2 none), `makerProjectComplete` (SD.PASS.2 showcase), `kindness_confirmed`, `checkin_done`, `journal_entry`, `wonder_asked`; `stretchAttempt` K1/S only. FS screens named `FS-SHELF3/ROUTE/PASSPORT/SEASON/RULES` in 01 §7.3's mapping — rename there if 07 prefers other ids. |
| 05 (F4) | 08 E3 probes `english.listening`/`english.letter_sounds` on K1's Tue expedition while 05 §15.1 puts K1's English placement in week 3 via a bottle (`bottle-06`). 01 §14.3 says "per 08/05 timing" — 05/08 to pick one (default: 08's E3 runs; the week-3 bottle is the story wrapper for the lazy probes). |
| 12 (F7) | Asset ids in 01 §7.4 are now the contract for `packages/content/en/` and `LICENSES.json`; parent voice path/manifest cited from 12 §6.11 unchanged. |
| 15 (F9) | Stale readings to update in §3: GAP-03 (15 min stands), GAP-31 (07 follows 01: G every other day), GAP-44 (week derived from DayKey, device tz + 04:00). |

## 4. Consistency re-check against current referenced specs (done before finalizing)

- 00 rev 2: E5 (Durable Objects) ✔, D1a ✔, C15 ✔, B7/B7a ✔, D4/D7 ✔, G4 ✔, A3 (orientation) ✔, C3 (K3 3–5 min ×2) ✔.
- 08: `LevelId`/`LEVEL_CHILD_HE` (§1.1א) ✔, `SessionEndReason` (§1.10) ✔, expeditions (§3.1–3.2, §3.6, §10.5.12) ✔, `closing` at 85%/80% + 60 s grace (§6.10) ✔, `profile.paused` (§10.5.8) ✔, `skill.level_reverted` (§10.5.3) ✔.
- 09: `region.englishIsle` + `grape` (§3) ✔, P19–P25 (§9.2) ✔, orientation table (§8.1) ✔, SCR-07 (needs the new state, forwarded) ✔, type tokens exist ✔.
- 10: §6.13 caps/bedtimes ✔, FAM-03 paused ✔, W-ECO-06 (forwarded) ✔.
- 11: routes (§4.1) ✔, `chestId = hash(sourceEventId)` and "no earn events" (§5.8) ✔, `CoopRoom` ✔; new events/docs/fields forwarded.
- 12: Mr. Mix-Up second en-US voice (§6.4 voice bible) ✔, §6.11 parent voice path ✔, §6.10.1 English content list ✔.
- 05: Mr. Mix-Up = CH-04 (§4), island ~20.12 (§15.1), bottles ✔. 07: §2.4 cadence, §1 #8 garden, §2.6 economy table ✔.

`⟂VERIFY` still open in both files: DD-37 / 02 UC-21 (SQLite container survives a re-signed install).
