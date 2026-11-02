# Fable review — cross-file issues from F5 (owner of `08-assessment-engine-child-model.md` and `10-parent-area-dashboard.md`)

Each row: file · section · problem · proposed fix. Decisions that 08/10 already made are marked **(decided in 08/10)**; the other file only needs to reference them.

## To 11 — technical architecture

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 1 | §5.8 event registry | 10 §4.2 metrics cannot be computed from the current registry. Missing events (defined in 10 §4.1 and 08): `focus.span {start,end,focusedSec,screenType}`; `screen.enter`/`screen.exit {screenType, ms}`; `app.background`/`app.foreground`; `behavior.signal` (= 08 §1.9 `BehaviorSignal`, cls `learn`, except `emotion_checkin` = `local_only`); `skill.level_changed {skill, from: LevelId, to: LevelId, reason}` and `skill.level_reverted` (08 §10.5.3); `benchmark.result {window, strand, theta, se, wordsCorrect45s?, wcpm?, errors?, pointed?}`; `placement.finished {subject, strand, theta, se, reason, quality}`; `creation.saved {kind, id}`; `launch.blocked {reason: quiet_hours\|bedtime\|shabbat_full\|cap}` (device-level, childId optional); `parent.tonight_done {cardId, childId}` (cls `admin`). | Add all to `packages/shared/events.ts` with zod schemas; cite 08 §1.4/§1.9/§1.10 as payload owners. |
| 2 | §5.8 `item.answered` | v1 payload lacks `outcome`, `firstTryCorrect`, `chosenErrorTag`, `touchesBeforeAnswer`, `idleMaxMs`, `appBackgrounded`, `context.{role,mode,assisted,wrapperTopic,afterWorkedExample}`, `steps`, `representation`, `answerableAt`, `firstResponseAt`, `dayKey`. 08 §4 and 10 M-04/M-07/M-08 need them. | Define `item.answered` **v2** = 08 §1.4 `Attempt` verbatim (import the type from `packages/engine`), keep v1 readable for replay. |
| 3 | §5.8 `session.ended` / §4.16 | `reason?` is free text and §4.16 uses `'switch'`; 10 used `childQuit`/`natural`; 08 used its own list. **Decided in 08 §1.10:** `SessionEndReason = completed \| daily_cap \| child_exit \| parent_exit \| frustration_break \| disengaged \| bedtime_lock \| inactivity \| app_killed` (`'switch'` → `parent_exit`). Also add `plannedMinutes`, `blocksCompleted`, `focusedMs` to the payload. | Adopt the enum from `packages/shared` (08 is the owner). |
| 4 | §5.8 `checkin.mood` | Same thing as 08 `behavior.signal{kind:'emotion_checkin'}`; two names for one event. Per 00 C8a the label is **not persisted beyond the episode**; only a 14-day local counter `{dayKey, checkins, sadOrScared}` survives. §5.9 "local_only … capped at last 90 days" contradicts C8a. | Rename to the 08 name (or alias), and change §5.9 `local_only` retention to "not persisted; 14-day aggregate counters only". |
| 5 | §7.1 cron, §7.10 weekly payload, §5.3 S4 | Already bound by 00 E5/H5/E7 (10 §18.2): weekly report `*/15 6-12 * * *` with Friday `send_at`, alert queue `*/15 * * * *`, remove `flags[]` from `GET /reports/weekly`, `disableDeviceFallback:true` on `shared_kids`. | Apply. |
| 6 | §7 endpoints | 10 §15.2 lists endpoints marked "חדש" (API-10..API-28). New in this review: `POST /children/:id/sessions/:sid/reattribute {to}` (API-13) writing `ParentInput.session_reattribute` as an `admin` event synced to all devices. `ParentInput` (08 §1.9א) needs an event type `parent.input` (cls `admin`) with the 08 union as payload. | Add `parent.input` to the registry; add the route; audit-log it. |
| 7 | §6.5 clock skew | 11 corrects HLC only. 08 §10.5.5 needs `serverTime` exposed to the engine (`ctx.now` = corrected time when skew > 2 min) and a per-device "clock inaccurate" flag for 10 FAM-04 / AC-41. | Store `clockSkewMs` on `device_self` and expose via `/sync` meta + `GET /devices`. |
| 8 | §6.1 local DB | 08 §1.11 checkpoints: 30 per child, every 200 events + session end; `modelEpoch`/`subjectEpoch` on the child doc (08 §10.5.6, never deletes events). | Add `engine_checkpoint(child_id, seq, last_event_key, state_blob)` table and `model_epoch` fields. |
| 9 | §5.10 web dashboard | 00 H6 makes Cloudflare Access optional. 10 SEC-09 says: optional layer, Viewer must be allowlisted, mobile app unaffected (does not go through Pages). | One sentence in §5.10 so nobody adds a second login in MVP. |
| 10 | §5.9 export | 08 §10.5.10: export adds `engine-state.v1` JSON per child; `FlagState` only when the parent ticks "include"; `emotion_checkin` never. | Extend `POST /admin/export` output spec. |

## To 02 — economy

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 1 | §4.1–4.2 | 00 C15 makes 08's five names canonical **also for the child view** (זרע/נבט/שתיל/עץ/עץ פורח). 02's plant mapping (ניצן סגור/נבט/שתיל/פרח ארד/פרי כסוף/פרי זהב) is a second scale. The 7 states are "internal economy triggers" (C15) and are derived via `packages/shared/levels.ts` (08 §1.1א): `locked`/`ready` = pre-states, `introduced` = `new && firstSeenAt`, `practicing` = `beginner\|advancing`, `mastered` = `master && rung<30d`, `retained` = `master && rung≥30d`. | Keep the plant as the **level** (5 growth stages from C15) and render the bronze/silver/gold **badge** as an overlay on `proficient`/`master`/`retained`; import `toEconomyState` instead of computing states. |
| 2 | §2.1 `skill_state_changed` | Should carry 08 `LevelId` (`from`, `to`), not economy states; 02 derives its trigger from the mapping. Add `skill.level_reverted` handling: after a parent `session_reattribute` (08 §10.5.3) a level can go back — **coins, gems, badges are never taken back** (C5). 02 should assert `badgeNeverDowngrades` and ignore `level_reverted` for wallet purposes. | Update the `LearnEvent` union and the simulation. |
| 3 | §10.1 | `profile.paused` children (08 §10.5.8, 10 FAM-03) must have `effTarget = 0` like "parent removed K3 from the goal". | Add `paused` to the formula inputs. |
| 4 | §10.3 Shabbat | 00 B7 `soft` = no coins/gems during the window; 08 §6.10 still updates θ. Make sure `applyEvent` zeroes grants in the window regardless of mode text. | Cite B7; add a test for `soft`. |

## To 03 — math

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 1 | §17.2 | All five requests accepted in 08: `Skill.extraGates` (`O.REP`, `O.STEP`), partial-credit `x = correctSteps/steps` (incl. self-correction = ×0.5 and BKT shadow rule), the 5 strands (`math.facts` in E1 with ≤6 fact items; `math.fractions` 3 probes in E3; geometry/measurement/patterns_data as `continuation` strands with lazy placement, 08 §3.7), the 2 flags at `maxStatus:'watch'`, and the `M.*` questionnaire map (08 §3.3א). | Reference 08 §3.3א/§3.7/§5.1/§9.2 instead of restating; drop "temporary IDs" wording. |
| 2 | §6.0א / §3.2 `O.REP` | 08 requires, for any skill with `O.REP`, ≥2 reviewed templates with `representation:'pictorial'` and ≥2 with `'abstract'` (content validator, 08 §1.2). Some G-band skills tagged `CPA: P` only (e.g. `M.NUM.SUBIT.C5`, `M.GEO.SHAPE.ID2D`) carry `MP: +O.REP` but may have no abstract template. | Either add an abstract template or drop `O.REP` on P-only skills; add a Jest check (`MATH-AC-12` extension). |
| 3 | §6.0א `band` | 00 A6 / 08 §1.2 now require `Skill.band: 'T'\|'G'\|'S'` as a mandatory field (derived from the band column T/T+/G/G+/S-R/S/S+). | Add to the parser derivation table (`T*→T`, `G*→G`, `S*→S`). |
| 4 | §14.3 `MATH_K1_READ_VS_LISTEN` | 08 makes it a marker feeding `HEB_K1_LISTEN_READ_GAP` (counts a week as "met" when θ-gap ≥ 0.7 and the read-vs-listen delta ≥ 15 pts), not a standalone note. | Reference 08 §9.2 wording. |

## To 04 — Hebrew

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 1 | Skill IDs / strands | 04 does not publish `StrandId`s nor a placement spine. 08 §1.1 reserves `hebrew.pa`, `hebrew.letters`, `hebrew.listening`, `hebrew.decoding`, `hebrew.comprehension`, `hebrew.spelling_morph`, `hebrew.writing` (⟂VERIFY) and 08 §3.3א needs a `questionId → H.*` map for T1–T5, T8, G1–G6, G12, S1–S6. | Add a derivation table like 03 §6.0א (strand, anchor, band, spine, questionnaire map). |
| 2 | Error-code namespace | 04 uses `E.VIS.*`, `E.PA.*` for **error codes**, while 05 uses `E.*` (`E.PH.SAT.GPC`) for **skill IDs**. `Skill.errorTags` and `chosenErrorTag` share one string space in the engine. | Prefix Hebrew error codes (`HE.*` or `ERR.H.*`) or namespace by subject in the content schema. |

## To 05 — English

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 1 | §17 row "08" | Accepted in 08 §6.8: K1 25% / K2 15% with two conditions — Hebrew ≥ English always (B1; K2's increase comes from math), and English ≤ 5% until the island opens (05 §15.1, ~20.12); K2 TPR songs are double-counted toward music/motor and fill the `fun` block. Expected in-app English for K1 ≈ 14 min/week; the rest of the 35–40 min is off-screen per 05. | Reference 08 §6.8; make sure 05's weekly plan assumes ~14 min in-app for K1. |
| 2 | §17 row "08" IDs | Adopted: `english.oral_vocab = E.VO.*.REC`, `english.letter_sounds = E.PH.*.GPC`, `english.decoding = E.PH.*.BLEND` (08 renamed `decoding_cvc` → `decoding`, `listening_vocab` → `listening`). Questionnaire G13 → `E.VO.COL.REC, E.VO.NUM.REC, E.VO.ANI.REC`; S13 → `E.PH.SAT.GPC`; S14 → `E.PH.SAT.BLEND`. | Confirm the spine order for each strand (placementEligible + spineOrder) in 05's skill JSON. |
| 3 | §3 ID prefix | `E.*` collides with 04's error codes (see 04 #2). | Coordinate with 04. |

## To 06 — extra subjects

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 1 | §2 "הערה ל-08" | **Decided in 08 §1.1:** no `coding`/`finance` `SubjectId`; use strands `logic.coding` and `world.finance`; the dashboard groups by strand. | Set `strand` accordingly in 06's derivation; remove the note. |
| 2 | §2 mapping | 08 §3.3א needs the questionnaire map for T5 (`world.animals`), T6 shapes/colors (shared with 03), T9 `motor.fine`, G4 `motor.name_writing`, G14 `motor.tracing`, and `lang.*` (T1–T4) if 06/07 own early language. | Add a map table. |
| 3 | missions | `profile.paused` and `grade_rollover` (08 §10.5.7–8) change `bandOf(child)`; missions filtered by band must re-evaluate. | One line in the mission scheduler. |

## To 07 — future skills / SEL

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 1 | SEL check-in | 08 §9.2 `EMO_CHECKIN` and 00 C8a: the check-in emits `behavior.signal{kind:'emotion_checkin'}` (`local_only`), the label is not persisted, only a 14-day counter. Make sure 07's check-in UI does not store the label in any doc. | Reference 08 §1.9/§9.2. |
| 2 | `sel.*` strands | 08 §1.1 reserves `sel.*` and `ai_literacy.*` with no displayed levels for `sel` (C8). | Confirm IDs. |

## To 09 — UX

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 1 | Parent typography | 10 §14.4 A11Y-1 needs `type.parent.*` tokens scaled with `UIFontMetrics` up to AX5 and the AX1+ layout rules (grids → lists). | Add the tokens and a `textStyle` → AX mapping. |
| 2 | Child level glyphs | 00 C15: child view = זרע/נבט/שתיל/עץ/עץ פורח (08 §1.1א exports `LEVEL_CHILD_HE`); 02's bronze/silver/gold badges overlay `proficient`+. | One icon set, imported from `packages/shared/levels.ts`. |
| 3 | B9 ⟂VERIFY | 10 RTL-1 now defers to 00 B9(c); 09 owns the verification (two grade-3 homeland textbooks + Hebrew iOS Health charts). | Record the result in 09's research log and flip the token once if needed. |

## To 12 — AI pipelines

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 1 | §? line ~1322 weekly-summary input | Lists "דגלי P3 (ספירות)" as an input to the LLM weekly summary. Contradicts 08 §8.4 (flags never in narrative input), 10 §11.7 and 00 E6a/H5. | Remove flags from any LLM input; the weekly report is template-only (10 §11.7), Haiku is used only for the monthly strengths narrative with `StrengthsNarrativeInput`. |

## To 13 — delivery

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 1 | Stories `10/PARENT-MINI`, `10/PARENT-WEEKLY` | P0 now depends on the 11 events listed in 10 §4.1 and on `item.answered` v2; the weekly email needs the domain (OPD-05). | Add the event registry work to the C3 tech-foundation checklist; mark the weekly email as blocked on OPD-05. |
| 2 | QA | New simulations S-5 (reattribute), S-6 (anomaly), S-7 (rollover) in 08 §12.11 and AC-38..53 in 10 §16. | Add to the CI matrix / acceptance list. |
