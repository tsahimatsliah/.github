# Fable review — cross-file issues from F4 (owner of 05, 06, 07)

Each row: file · section · problem · proposed fix. Decisions I made inside 05/06/07 that other files must mirror are marked **(mirror)**; open conflicts are marked **(decide)**. 00 revision 2 IDs are used.

## 08 — assessment engine (owner of `EngineConfig`, levels, flags; 00-J1)

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 1 | §6.8 `subjectWeights` **(decide)** | 05, 06 and 07 together asked for far more than 100% of K1 screen time (05: 35–40 min ≈ 40%; 06: 30–35%; 07: 2 FS episodes/week + 10% rituals) against 08's "other 15%". F1's issues file (#11) recommends 25%/15% for English alone, which leaves 06+07 nothing. 05 §2.1 / 06 §2.5 / 07 §12.1 now carry one identical budget table derived from 02 §12.1 (K1 5×18 ≈ 90 min, K2 6×10 ≈ 60 min) and 08 daily caps. | Set `EngineConfig.subjectWeights`: **K1 `{math:30, hebrew:28, english:20, enrichment:12, fs:10}`**, **K2 `{hebrew:30, math:30, english:15, enrichment:15, fs:10}`**, where `enrichment = science|world|logic|art|music|motor` and `fs = ai_literacy|sel`. Parent-editable (H4). K2 TPR songs may count for `music`/`motor` too (F1 #11). If 08 prefers F1's 25%, cut `enrichment`/`fs` accordingly and tell 06/07; the sum must stay 100. Add the 1,000-week scheduler simulation used by 05 AC18 / 06 AC-06-22 / 07 AC-35. |
| 2 | §1.1 `Skill` **(mirror)** | 05 M-REC (K2 receptive vocab: window 6, clean 5, p 0.80) and 06 M-PERF/M-EXPO (performance/exposure nodes) have no representation; C11 allows entry gates but 06's node kinds are not "weaker rules", they are different units. | Add optional `masteryProfile: {window, proficientClean, proficientP}` (05 falls back to M-STD until then) and `kind: 'performance' | 'exposure'` (06 §2.2 describes the interim mapping). |
| 3 | §9.2 flags **(mirror)** | 07 §10 now registers two SEL flags in 08's engine and drops its own duplicates: `EMO_CHECKIN` (= 10's proposed rule, with 07's threshold: ≥5 of last 7 check-ins in storm/rain on ≥5 distinct days within 14 days, **or** ≥4 fear/worry words on ≥3 days), consent `T4b` / `consent.checkinNudge` (boolean-only sync, default off, OPD-27); `SEL_LADDER_PAUSE` (bravery-ladder auto-pause). 07 RF-03 was deleted in favour of 08 `EMO_DISTRESS`. `NOTE_*` items are parent-card notes, not flags. Counters pause while `family.calmMode` is on. | Add the two rules to `RULES` with `weeklyMarkers` reading `fs_local_counters` (device-only, never synced raw, C8a); keep the 28-day notice gap and the standard disclaimer. |
| 4 | §2.4 / §3 placement IDs **(mirror)** | `english.oral_vocab`, `english.listening_vocab`, `english.letter_sounds`, `english.decoding_cvc` have no `E.*` mapping. | Map: `oral_vocab`/`listening_vocab` → `E.VO.*.REC`; `letter_sounds` → `E.PH.*.GPC`; `decoding_cvc` → `E.PH.*.BLEND`. Questionnaire S13 `yes` → `known` with rung-1 confirmation (not skip). |
| 5 | §1.1 `SubjectId` | 07 maps AI+TK → `ai_literacy`, SE+SD → `sel`; 06 maps C → `logic`, F → `world`, H.MOVE/A.MOVE → `motor`. | Confirm; optionally add `coding`/`finance` later (06 §2.2 note). |
| 6 | §11 `explorationShare` | 06/07 cite 08's values (K1 0.25, K2 0.30, K3 0.50) and 00-C9 (topics/wrappers only). | No change; just keep these as the single source. |

## 02 — economy (owner of every number; 00-D7)

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 7 | §2.7 `neverEarns` **(mirror, 00-D1a)** | 07 events that must never pay coins/gems are not listed: emotional check-in (`checkin_done`), `kindness_confirmed`, bravery-ladder steps, reflection entries (`journal_entry`), wonder questions (`wonder.asked`; 07 dropped its "1 coin per question"). | Add them to `neverEarns` and to the zod invariant. |
| 8 | §2.2 / §6 | 07 needs two milestone events: `braveryLadderComplete` → one gem, never announced (00-D1a allows a gem at the **end** of a ladder); `seasonComplete`/showcase chest for `SD.PASS.2` (07 §2.6). | Add rows (amounts are 02's). |
| 9 | §2.2 `stretchAttempt` | 07 `SD.STRETCH.1` is band G; 02 pays `stretchAttempt` only in the K1 table. 07 now says G gets a badge only, S gets `stretchAttempt`. | Confirm `stretchAttempt` is K1/S-only in config (`enabledFor`). |
| 10 | §2.2 `offlineMission` | 06 §16.1 and 07 cite 02's values (wood chest, K1 15 / K2 3 coins, 60 XP). 07 kindness/bravery offline missions use `economy:'kindness_confirmed'` instead. | Make `offlineMission` grants conditional on `mission.economy === 'offlineMission'`. |
| 11 | §7.6 campfire | 00-D1a names a "family garden"; 07 §1 #8 defines it as a cosmetic layer beside the campfire fed only by parent-confirmed kindness/team acts (no trees, no coins). | Confirm it is **not** part of the "עצים למדורה" pot; 01 places it visually. |

## 01 — game design (owner of cast, mechanics, calendar)

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 12 | §4.1 cast **(mirror)** | 07 had an unlisted companion "שועלי" with "hats". 07 now casts `COMPANION` = ניצי (CH-01) for AI/TK/SD, דובדובה (CH-07) for SE and the check-in, ברגית (CH-06) for SD.MAKER, גלגלון (CH-03) = `ROBOT`/`BOTS`; hats are costume props on those characters. 06 uses פרופסור למה (CH-08) for S, ברגית for S.TECH. | Note the FS/extra-subject roles on the cast cards; no new character needed. |
| 13 | §4.3 / episode 6 **(decide)** | "Mr. Mix-Up" (English-island boss) is not in the cast table; 05 §4 assumes he is הבלגניסט (CH-04) in an island costume with a **separate en-US voice** (one voice = one language, B2). | Confirm, or add a cast row; either way CH-04's Hebrew voice never speaks on the island. |
| 14 | §2.4 / §13.2 | 05 §15.1 proposes a "bottles from the sea" pre-island phase from launch (8.11) until episode 6 (≈ week 6 ≈ 13–20.12.2026, inside S1); 01 S1 row says "R-08 English island (week 6)" — confirm "week 6 from launch". | Add the phase to §2.4 and confirm the date. |
| 15 | §8 mechanics catalogue | 05 requests `MX-E01` mode `pairs` (minimal pairs), `MX-E05` lighthouse-tricky, `MX-E06` ship-log, `MX-E07` dock-chat, and Pip the crab (silent). 07 §7.1 lists new mechanics without `MX-*` IDs: `blicket`, `fair-test-builder`, `breath-mic`, bravery `ladder`, `shelf-3`, `route-planner`, `passport`, `season-track`, `rule-builder`. 06 uses `build-sim`, `code-strip/stack`, `map-place`. | Adopt or map each; until then they are implemented as modes/FS screens outside the catalogue. |
| 16 | §7 check-in cadence **(mirror)** | 01 says K2 feelings check-in "every other day"; 07 §2.4 now follows that (S every episode, G every other day, T optional). | Keep in sync. |
| 17 | §10.4 campfire | Place 07's family garden (issue 11) next to the campfire. | Visual only. |
| 18 | §2.2 `palette.english` **(decide, with 09)** | 01 `#19B7B0` turquoise+yellow vs 09 `region.englishIsle` purple-teal + `grape`. 05 references tokens only. | 09 owns tokens (J1); 01 updates the value; Sunny stays turquoise+yellow. |

## 09 — UX / design system

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 19 | §5 type tokens | 05 needs `type.en.k1.decode` ≥36 pt (single word in decoding items) and `type.en.letter` ≥64 pt (single letter). | Add tokens (F1 #12 agrees). |
| 20 | §3 `region.englishIsle` | Conflict with 01 `palette.english` (issue 18). | Decide one value; 05 AC8 screenshots will use whichever token wins. |
| 21 | `CMP-22` | 05 §10.4 relies on: exit button at the Hebrew UI's end position, pause button (דובדובה) visible inside the island, no `rtl` component inside. | Confirm in the component spec; 05 AC19 tests it. |
| 22 | B9(a) **(mirror)** | 06 code strips: G flows RTL, S vertical; arrows absolute. 05 AC19 / 06 AC-06-28 forbid the code studio in the same episode as an English block. | No token needed; note it under B9. |

## 10 — parent area

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 23 | §6.9 `EMO_CHECKIN` **(decide)** | 10's draft threshold "sad/afraid 3 of last 5 days" is more sensitive than 07's (≥5/7 check-ins on ≥5 days in 14 days, or ≥4 fear words on ≥3 days). 07 (SEL owner) proposes its threshold as canonical. | Point §6.9 to 07 §10 for the rule text; keep T4b/`consent.checkinNudge` and default-off sync (OPD-27). |
| 24 | Parent-card stream | 07 `NOTE_BREAKS`, `NOTE_BELL`, `NOTE_REENTRY` are "notes" (info), not flags; they must not go through the 28-day flag gate and must not appear in PA-FLAG. | Render them in the card stream (PA-STR or the inbox). |
| 25 | Settings | 05/06/07 use `family.calmMode` (security-situation / "safe room" calm mode, 00-B8) and the derived `family.hasChildUnder36m` (00-C13; with a manual override P-06-8). Neither exists in 10 yet; `safeRoom` is only a screen. | Add the toggle (parent gate), name the field, and expose the small-parts override behind an explanation. |
| 26 | Fact errata | 06 §12.6 needs a "report an error" button on every fact card / quiz item in the parent view, feeding `fact.reported`. | Add to the content-approval screens. |
| 27 | §… English card | 05 §17 lists the per-child English card (input minutes vs target, current UNIT, "ask tonight", minimal-pair status, no sibling comparison). | Confirm it exists in PA-CHILD. |

## 11 — technical architecture

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 28 | Settings schema | Fields `family.calmMode`, `family.hasChildUnder36m` (derived nightly from birthdates), `consent.checkinNudge`. | Add; `calmMode` is LWW like other settings. |
| 29 | Sync exclusions | `fs_local_counters` (07 §2.7) must be excluded from the event log and sync; only `rf_raised {ruleId, week}` syncs, encrypted, with consent. | Add to the sync allow/deny list (C8a, E6a). |
| 30 | API | `POST /fs/t2/request` (07 §7.3): body `{childId, activityId, chips[]}`; server validates every chip against the closed `chipSets` list and rejects anything else with 400 (00-C7a, 07 AC-32). `POST /content/fact-report` for 06 §12.6. | Add endpoints and the chip schema. |

## 12 — AI / content / voice pipelines

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 31 | §4.5 / §3.x P3 **(decide; 00-C7a binds)** | P3 K1 `llm_turn` accepts "chips + short text ≤60 chars". 00-C7a (rev 2) says no free text from any child reaches the server, not even K1; 07 AC-05/AC-32 enforce it for FS. | Make P3 chips-only (the sanitised `CHILD_TEXT` line in the prompt becomes a chip label); if a free-text path is kept it needs a 00 change, not a spec note. |
| 32 | §4.5 "שיקוי פרומפט" **(mirror)** | 12 has `5×5×4×3 = 300 + 2–3 "change one"`; 07 `AI.PROMPT.3` now fixes **4 mandatory slots full product = 300**, + 60 "detail" variants + 75 "no-where" images = **435 cells**, per-cell parent approval, 13 single-component neighbours per cell. | Update the table row and the G5 gate count. |
| 33 | §4.5 "שאלה טובה יותר" **(mirror)** | 12 has `6×3×3 = 54`; 07 `AI.PROMPT.4` now fixes **3 upgrade cards → 8 variants + 2 hats × 2 = 12 per topic, 12 topics = 144** pre-generated answers, with a human-verified fact key per topic. | Update the row; T2 path (v1) is chips-only via `/fs/t2/request`. |
| 34 | §6.10 English recording list **(mirror)** | 12 lists 44 sounds with keywords dog/hot/kite/thin; 05 §6 (owner) lists **45 files** (43 phonemes incl. schwa + `ks`, `kw` clusters) with keywords pot/cat/thumb, 3 takes, plus 26 letter names, ~60 nonsense words, 8 segmented words, 16 human minimal pairs (05 adopted 12's pair list), and the song vocals (≈2.5 h total). | Mirror 05 §6/§12.2–12.4 and OPD-15; present one consolidated recording plan (F1 #26). |
| 35 | Voice bible | 05 §4 needs a second en-US male voice for Mr. Mix-Up (the "friend" voice in 05 §12.2) and 4 adult HVPT talkers (2 f, 2 m) from the bake-off vendor. | Add to `voice-bible.yaml`. |
| 36 | §12.3 blocklists | 06 `calm-suppress` node list and 07's bravery-ladder topic blocklist (security, invasive medicine, strangers, internet, deep water, heights, dangerous animals) should be linted centrally. | Add a `ladder_topics` blocklist. |
| 37 | Errata pipeline | 06 §12.6: `fact.reported` → regenerate the fact card in the next Batch from the corrected brief → Sonnet judge → parent approval → pack `version+1`; ≤7 days. | Add to the nightly pipeline states (`quarantined` already exists). |

## 13 — delivery / QA

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 38 | §4 checkpoints | 07 §11.1 now dates MVP/v1/v2 to C5/C6/C7; 05's island opening (~20.12) falls inside the vertical-slice window (30.11–3.1) and needs `MX-E01` + the bottles phase in the VS scope; 06 AC-06-06 (12 packs + 3 K.TOT) by MVP. | Confirm scope per checkpoint. |
| 39 | QA matrix | New simulation-based ACs (05 AC18, 06 AC-06-22, 07 AC-35) share one 1,000-week scheduler simulation; add it to CI. | Add a job. |

## 03 / 04 — math and Hebrew

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 40 | 03 §3 IDs | 06 `crossPrereqs` now cite `M.NUM.COUNT.ROTE10`, `M.MEAS.MONEY.CHANGE`, `M.OPS.ADD.2D2D`, `M.MEAS.LEN.CM`, `M.MEAS.LEN.NONSTD`, `M.DATA.PICTO.G`, `M.DATA.BAR` (06 AC-06-26 checks they exist). | Keep these IDs stable. |
| 41 | 04 printables | 00-C13 applies to 04's cut-and-paste printables (scissors = `adult-required`); 05 adopted the same for PR-2. | Tag them. |

## 00 (F1) — observations, no decision needed

- 00-D1a mentions "the family garden"; it is now defined in 07 §1 #8 (cosmetic, no currency). If 00 wants a one-liner: "family garden = cosmetic layer at the campfire fed by parent-confirmed kindness acts; never a currency".
- 00-C13 says "while a child under 3 lives at home … no small parts for any child"; 06 adds a parent override (P-06-8, default off) after an explanation. Say if an override is unacceptable.
- 00-E9 says "~44 phonemes"; 05's bank is 45 files (43 phonemes incl. schwa + 2 clusters). Cosmetic.
- `family.calmMode` is the working name used by 05/06/07 for the B8 calm/"safe room" switch; 10/11 own the final field name (⟂VERIFY in each file).
