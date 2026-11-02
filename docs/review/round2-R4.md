# Round 2 — R4 (owner of 05 English, 06 extra subjects, 07 future skills)

Date 2026-11-02. Inputs: `fable-issues-F1…F9.md`, spec 15 §3 GAPs, 00 **revision 3** (B8b, C7a, C13, D1a, E9), the current 01/02/08/09/10/11/12/13, `round2-R1.md`, `round2-R5.md`. `round2-R3.md` had not been filed when this closed; the one item that depends on it (error-code prefix) is marked ⟂VERIFY in 05 §3.1.

Dispositions: **applied** · **already resolved** (by the round-1 owner or by the other spec) · **rejected** (with rationale) · **forwarded** (to the owner named).

## 1. Issues targeting 05

| Source | Item | Disposition | Where / note |
|---|---|---|---|
| F1 | #11 English share 25%/15% → 08 | already resolved (withdrawn) | 08 §6.8 adopted the 05/06/07 table (K1 20 / K2 15); 00 Appendix B records the withdrawal. 05 §2, §2.1, §20.2 now say "adopted", with 08's three caveats (Hebrew ≥ English; `english`=5 before the island; K2 TPR double-counted) and the S-8 simulation = AC18. |
| F1 | #12 mechanics + type tokens → 01/09 | applied | 09 §3.6 has `type.en.letter` 64/56 and `type.en.k1.decode` 36/32 → 05 §10.6 says "adopted". 01 §7.3 has MX-E01 `pairs`, MX-E05/06/07, CH-11 → 05 §4 table rows renamed, §20.2 statuses "התקבל". |
| F1 | #21 OPD IDs in §19 | already resolved | §19 carries OPD-20/01/15/34/04. |
| F1 | #26 recording sessions consolidated in 12 §11 | applied | 05 §12.3 new bullet points to 12 §6.10–6.12/§11 + OPD-12/13/15 and limits 05 to session A's content. |
| F1 | #30 safety tags on printables/routines | already resolved | AC22 (PR-2 `choking-hazard-T` + `adult-required`, FR-3/5/6). |
| F2 | #3 statuses → accepted with IDs; T09 `echo_pairs` = `MX-E01:pairs`; Pip = CH-11; bottles = `english_bottle` S0 slot | applied | §4 table, §4 Pip paragraph, §15.1 bottles paragraph, §20.2 (five rows). |
| F2 | #4 4-voice minimal-pairs bank → 12 | already resolved | 12 §6.10.1 `en-minimal-pairs.json`; 05 §12.2 now cites it. |
| F4 (own, answered by F5) | #1 weights | applied | see F1 #11. |
| F5 → 05 | #1 reference 08 §6.8, drop 25% | applied | §2 row, §2.1. |
| F5 → 05 | #2 placement IDs, spine order | applied | new **§14.5**: spine per strand (`english.oral_vocab`, `.listening`, `.letter_sounds`, `.decoding`, `.speaking`) with `spineOrder`/`placementEligible`; `placement` field in §3.2; questionnaire map G13/S13/S14; "`yes` is a pseudo-observation (w 0.7), not `known`" (08 §3.5/§3.7) → §17 (5) reworded. |
| F5 → 05 | #3 `E.*` namespace collision with 04 error codes | applied (partial, ⟂VERIFY) | §3.1 namespace paragraph + AC1 lint: `E.*` reserved for English skill IDs; English errors stay `ERR.*`/`SIG.*`; 03/04 error prefixes expected `EM.*`/`EH.*` per R3 (⟂VERIFY `round2-R3.md`). |
| F5 answers to F4 | #2 `masteryProfile` floor; 6/5 rejected → 7/6; `kind` | applied | §3.3 M-REC = `{7, 6, 0.80}`; F-FAM = `kind:'performance'`, X-EXP = `kind:'exposure'` (capped at `advancing`); §3.2 schema. |
| F5 answers to F4 | #4 S13 `yes` ≠ `known` | applied | §14.5, §17 (5), §20.2. |
| F5 answers to F4 | #23–27 (10: W-CH-11, SET-08) | applied | §17 cites W-CH-11 and SET-08. |
| F4 obs. 3 / 00 E9 rev 3 | 45 files; VO −18 LUFS | applied | §12.2 (12 §6.10.1 aligned, CI check), §12.3 and AC5 → **−18 LUFS ±1 / ≤ −1 dBTP** (was −16; R1-3). |
| 00 B8b rev 3 | `family.calmMode` name bound | applied | §17 cites 00-B8b + 10 SET-08; ⟂VERIFY on the name removed (R1-10). |
| F9 / 15 | GAP-30 Hebrew cap warning inside the island | applied | §10.4 new bullet: entry guard (island length + 60 s), "Bye-bye" + exit jingle before the Hebrew wind-down, SR-10 the only exception; AC26; EC-24. |
| F9 / 15 | §13 PP-3 mic permission denied (UC-026) | applied | PP-3a: "Let's say it together!" without recording, F-FAM from parent/child tap, no re-prompt inside an episode, status line in W-CH-11; AC27; EC-25. |
| F4 #13 (to 01) | Mr. Mix-Up cast row | forwarded (non-blocking) | 12 §6.2 assigned the voice; 01 §4.1 still has no cast row. 05 §4/§20.2 keep a narrow ⟂VERIFY. |
| F4 #18/#20 | island colour 01 vs 09 | forwarded (non-blocking) | 09 `region.englishIsle` purple-teal is binding (J1); 01 §2.2 still `#19B7B0`. 05 uses tokens only. |
| 13 §5.2 | island opening sprint 5 vs 05's ~20.12 | applied | §15.1: bottles continue until `R-08.unlocked`; no hard end date. |

## 2. Issues targeting 06

| Source | Item | Disposition | Where / note |
|---|---|---|---|
| F1 | #5 C11 → C13 for safety tags | already resolved | §0, §16.1, §20 (round 1). |
| F1 | #6 time-axis direction → cite B9 | already resolved | §20 #3 note, AC-06-28 use `chart.timeDirection`. |
| F1 | #21 OPD IDs | already resolved | §19 P-06-1…8. |
| F1 | #24 D1 savings bonus only time source | already resolved | AC-06-12, §20 #1. |
| F1 | #30 safety enum in mission schema | already resolved | §2.1 `SafetyTag`, §16.1. |
| F2 | #7 hook IDs (blicket skins on MX-X02, build-sim = MX-X05, `weather_tap` + MX-M20, `campfire_lantern_lit` light-only, garden never wilts) | applied | §4.5 #1–#6 rewritten with the IDs; lantern explicitly not a log/coin/level source (02 §2.1, D4); base garden distinguished from 07's family garden (D1a). |
| F3 | #26 sub-band names | applied | §0: reuse 08 `subBand` names if ever needed. |
| F4 (own) → F5 answers | #2 `kind` adopted | applied | §2.2: M-PERF = `kind:'performance'` + `masteryProfile {3,3,0.80}` → `proficient` after **3** instances (08's "last 3 clean" floor; was 2); M-EXPO = `kind:'exposure'` capped at `advancing`; interim `concept` mapping removed. |
| F5 → 06 | #1 no `coding`/`finance` SubjectId; strands `logic.coding`/`world.finance` | applied | §2.1 `engineStrand`, §2.2 derivation rule, AC-06-30; "הערה ל-08" removed. |
| F5 → 06 | #2 questionnaire map T5/T6/T9/G4/G14/`lang.*` | applied + forwarded | new **§2.7** map with weights (T5 → `S.INQ.01`/`S.LIFE.01`; T6 → `A.COLOR.01`/`L.SPAT.01`; T9 → `L.SPAT.02`; G14 → `A.SHAPE.01` secondary); G4 (`motor.name_writing`) and T1–T4 (`lang.*`) are **04's** (`hebrew.oral`, name-banner/trace-warmup) → forwarded to R3/R5. AC-06-31. |
| F5 → 06 | #3 `profile.paused` / `grade_rollover` re-evaluate `bandOf` | applied | §0 (bandOf derived from grade, 08 §1.1), §16.1 mission filter sentence, EC-21. |
| F5 answers to F4 | #1 weights + S-8 | applied | §2.5 "adopted", caveats, S-8 = AC-06-22. |
| F5 answers to F4 | #25–26 (10 SET-08, fact-report button) | applied | §16.4 cites B8b/SET-08; §12.6 cites 10 §6.10 + `POST /content/fact-report` (12 implements; 11 missing → forwarded). |
| F7 obs. | "132 skills" not verifiable | applied | Generated count = **134** nodes in §4–§11 (S 39, G 26, K 5, L 15, C 15, A 14, F 11, H 9) stated in §0 with AC-06-29 (`pnpm content:count`); 13 §5.6 "132" → forwarded. |
| F9 / 15 | GAP-37 mission chest daily cap (owner 02) | applied on 06's side | §16.1 cites the ≤1/child/day cap as 02's. |
| 00 B8b / C13 rev 3 | `family.calmMode`, `hasChildUnder36m` override | applied / already resolved | §16.4 cites B8b (⟂VERIFY removed); C13 override already in §16.1/P-06-8/AC-06-24. |
| 15 UC-003 E1 | which drives `child.band`: grade or birthdate | applied | §0: `bandOf` is grade-derived (08 §1.1); ages descriptive; birthdate only for touch targets and `hasChildUnder36m`. |

## 3. Issues targeting 07

| Source | Item | Disposition | Where / note |
|---|---|---|---|
| F1 | #21 OPD IDs | already resolved | §16 P-01…P-08. |
| F1 | #22 / F2 #6 G4 line in §0.5, no "bot" for the hero | already resolved | §0.5 last bullets; AC-33 bans the hero's `characterId`. Re-checked: no 07 card calls the hero a robot. |
| F1 | #30 safety tags | already resolved | §2.7 `OfflineMission.safety`, AC-39. |
| F2 | #5 `TOOLBELT` = MX-T01, `BRAIN_TOKEN` = MX-T06, RIT segments, RIT-MID cadence | applied | §2.2 (already MX-T01), §2.3 (MX-T06 gate, `brainToken.ts`), §2.4 (`arrival_plan`/`ritual_mid`/`goodnight_tell`, open+close ≤10%), §7.1. Cadence K2 3/5 days, K1 daily = 01 §5.3 (confirmed). |
| F3 | #26 sub-band names | applied | §0.2. |
| F4 (own) #15 → 01 | unmapped FS mechanics | applied | 01 mapped `blicket`/`fair-test-builder` to MX-X02 modes and `breath-mic` to MX-S02; the rest (`ladder`, `shelf-3`, `route-planner`, `passport`, `season-track`, `branching-dilemma`) are final FS screens outside the `MX-*` catalogue (01's F4 #15 answer). §7.1 rewritten; `mechanics.aliases.json`. |
| F4 (own) #16 / F9 GAP-31 | check-in cadence 01 vs 07 | **rejected** (F9's "07 wins: every episode") | 07 §2.4 and 01 §5.3/§7 already agree (S every episode, G every other day, T optional) since round 1; `EMO_CHECKIN` counts "5 of the last 7 check-ins on ≥5 distinct days in 14 d", so it does not need a daily cadence (R5 confirms, 08 §9.2 note). §2.4 now states it is the canonical wording and 01 mirrors it. |
| F4 #7–#11 (to 02) | `neverEarns` additions, `braveryLadderComplete`, `stretchAttempt enabledFor`, `offlineMission` conditional, family garden | forwarded (still open in 02) | 02 §2.7/config has none of them. 07 §2.6 now states the safe behaviour (07's engine emits no economy events for these activities; AC-13) so the outcome is identical until 02 adds the rows. |
| F5 → 07 | #1 check-in emits `behavior.signal{kind:'emotion_checkin'}` `local_only`, label never stored | applied | §2.4, §2.7 storage bullet. |
| F5 → 07 | #2 `sel.*`/`ai_literacy.*` strands, no displayed levels for `sel` | applied | §2.7 `FSSkill.engineStrand`. |
| F5 answers to F4 | #3 `EMO_CHECKIN` 07 thresholds, `SEL_LADDER_PAUSE` semantics, `NOTE_*` → AL-17 | applied | §10 principles rewritten: 14-day cadence without `watch`, ladder `watch` immediate, no referral, AL-17 for notes. |
| F5 answers to F4 | #1 weights | applied | §12.1 "adopted", `fs` 12 before the island (U-WGT-2), S-8 = AC-35. |
| F7 obs. | "72 skills" ⟂VERIFY | applied (verified) | Generated count = 72 (AI 21, TK 19, SE 19, SD 13) = card headings = graph tables; stated in §0.2 with AC-40. |
| F9 / 15 | GAP-32 one emotional rule | already resolved | §10 (round 1) + round-2 wording per 08 §9.2. |
| F9 / 15 | GAP-46 sibling letter moderation (owner 01 CO-4; 00 C7a rev 3) | applied on 07's side | §2.7 "הודעות בין אחים": chips/numbers/pack pictures only, `coop_letter` doc, parent thumb before K2/K3 see a K1 message, never through an LLM; `ACT-SE.KIND.3b` points to it; AC-41, EC-21. |
| 00 B8b / C13 rev 3 | `family.calmMode` effect list; `hasChildUnder36m` override | applied | §10 calm bullet cites B8b (⟂VERIFY removed), EC-22; AC-39 cites the derived field + audited override. |
| F4 #30 (to 11) | `POST /fs/t2/request`, `chipSets` schema | forwarded (still missing in 11) | §7.3 ⟂VERIFY. |

## 4. New / still-open issues for other owners (blockers only)

| # | Owner | File / section | Problem | Proposed fix |
|---|---|---|---|---|
| R4-1 | 11 (R6) | 11 schema / §5 / §7 | None of these exist in 11 yet although 00 rev 3 binds the names and 05/06/07/10 use them: `family_prefs.calmMode` (B8b), `family.hasChildUnder36m` (C13, nightly derivation + audited override expiry), `consent.checkinNudge` (C8a/T4b), `fs_local_counters` excluded from the event log and sync (07 §2.7, AC-10), `rf_raised {ruleId, week}` encrypted sync, `POST /fs/t2/request {childId, activityId, chips[]}` validated against `chipSets` (400 otherwise; 07 AC-32), `POST /content/fact-report` + `fact.reported` event (06 §12.6, 10 §6.10, 12 §2.7), `coop_letter` doc schema (C7a, 07 §2.7, 01 CO-4), `campfire_lantern_lit` event (06 §4.5). | Add fields, endpoints, event schemas and the sync deny-list entries. |
| R4-2 | 02 (R2) | 02 §2.2 / §2.7 / config | 07's economy mapping is bound by 00 D1a but 02 still lacks: `checkin_done`, `kindness_confirmed`, `journal_entry`, `wonder.asked`, bravery-ladder steps in `neverEarns`; milestone `braveryLadderComplete` (one gem, never announced); `stretchAttempt.enabledFor = ['K1']` (G gets a badge only); `offlineMission` grant conditional on `mission.economy === 'offlineMission'`; mission chest ≤1/child/day (15 GAP-37). | Add rows + zod invariants; 07 §2.6 documents the interim (07 emits no economy events for these). |
| R4-3 | 13 (R7) | 13 §5.6, §7.1 | 06's skill count is **134**, not 132 (generated from §4–§11; S 39, G 26, K 5, L 15, C 15, A 14, F 11, H 9). 07's 72 is verified. | Replace 132 → 134 and drop the ⟂VERIFY; keep `pnpm content:count` as the P0-E1-S6 check. |
| R4-4 | 04 (R3) / 08 (R5) | 08 §3.3א questionnaire map | 06 does not own early language or writing: T1–T4 (`lang.*`) belong to 04 `hebrew.oral`; G4 `motor.name_writing` and G14 `motor.tracing` belong to 04 (name-banner MX-H01, trace-warmup MX-H09). 06 §2.7 maps only T5/T6/T9 (+ G14 secondary). | 04 publishes those rows in its questionnaire map; 08 keeps `lang.*` as 04 strands (or aliases to `hebrew.oral`). |
| R4-5 | 03/04 (R3) | error-code prefix | 05 §3.1 / AC1 now reserve `E.*` for English skill IDs and expect 03/04 error codes to move to `EM.*`/`EH.*` with `errors/aliases.json` (12 §3.7 `classifyError` accepts both). | Confirm the prefixes in `round2-R3.md`; if R3 chooses other prefixes, only the one ⟂VERIFY sentence in 05 §3.1 changes. |

Non-blocking, recorded in the specs: 01 §4.1 has no cast row for Mr. Mix-Up (12 §6.2 already assigns the voice); 01 §2.2 `palette.english` still `#19B7B0` while 09 `region.englishIsle` (binding) is purple-teal.

## 5. Consistency re-checks done against current specs

- 08 §6.8 / §11 config weights, U-WGT-2, S-8, `masteryProfile` floor (U-MPR-1/2), `kind` semantics, §3.3א English rows, `bandOf` grade rule, `EMO_CHECKIN`/`SEL_LADDER_PAUSE` rows → mirrored verbatim in 05/06/07.
- 01 §5.3 segments (`english_bottle`, `ritual_mid` cadence, `weather_tap`), §7.1 tiers, §7.3 cards (MX-E01 pairs, E05–E07, T06, X02 skins, X05), §4.11a CH-11, §13.2 R-08 week 6 → cited.
- 09 §3 `region.englishIsle`, §3.6 `type.en.*`, CMP-22 → cited. 10 W-CH-11, SET-08, AL-17, §6.9/§6.10 → cited. 12 §6.2 voices, §6.10.1 bank (45 IDs, file names, HVPT, Pip/bottles), §2.7 errata → cited. 13 §5.2 VS scope (bottles + MX-E01), sprint 5 island → cited.
- Counts generated by grep over the files (07: `###` cards = graph-table rows = 72; 06: unique first-column IDs in §4–§11 = 134).
