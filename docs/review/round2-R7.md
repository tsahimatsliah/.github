# Round 2 — R7 (owner of `12-ai-content-voice-pipelines.md`, `13-delivery-roadmap-qa.md`, `15-use-cases-journeys-edge-cases.md`)

Scope: every item in `fable-issues-F1…F9.md` and spec 15 §3 whose target is 12, 13 or 15, plus the round-2 blockers sent by the coordinator and by R1/R2/R3/R4/R6 (00 rev 3, error-code namespace, 06 count, Maestro path, music LUFS). Dispositions: **applied** / **already resolved** (round 1 did it; verified in the current text) / **rejected** (+why) / **forwarded** (owner named). Each owned file got a "### Round 2 changelog". The spec-15 pass was done last, against the owners' files as they stood at 23:27 (after R1–R6 wrote their reports). Nothing committed.

## 1. Issues targeting 12

| Source · id | Topic | Disposition | Where / note |
|---|---|---|---|
| F1 #17 | Dual-route Opus+Sonnet handwriting conditional (00 E8) | already resolved | §1.1 `photo.extract.hw`, §3.5.1 (`photo.dualRoute` off; +0.10 word coverage, 0 silent errors, ≤$0.12/page, ADR), §10 P2 |
| F1 #18 | Gate G4b + `plan:"paid"` for Recraft | already resolved | §7.1 G4b row, `LICENSES.json` CI check |
| F1 #21 | OPD ids on parent items | already resolved; **applied** update to rev 3 | §11: range OPD-01…36, OPD-35 adopted (was "מוצע"), OPD-08 wording of rev 3 |
| F1 #26 | Consolidated recording plan (OPD-12/13/15) | already resolved | §11 table + §6.10–6.11 |
| F2 #4 | 4-voice minimal-pairs bank | already resolved | §6.10.1 HVPT bank (4 TTS voices + 16 human pairs) |
| F2 #25 / R2 → 12 | Pip CH-11, bottles, MX-E05/06/07, hero wording lint | already resolved; **applied** asset-id contract | §6.10.1: 01 §7.4 asset ids (`en/bottles/bottle-<nn>/…`, `en/mx-e05/…`, …) are the contract for `LICENSES.json` + CI check; V23 hero-wording lint already there |
| F3 #10 / F5 → 04 #2 / 05 #3 / **R3 decision** | Error-code namespace | **applied** | §3.7 + §12.2: canonical targets are `EM.*` (math) / `EH.*` (Hebrew); `E.*` = English skill ids only; 05 keeps `ERR.*`/`SIG.*`. Draft Hebrew names (`E.SPL.T_TET`, `E.SPL.K_Q`, `E.SPL.K_CH`, `E.SPL.S_SIN`, `E.SPL.B_V`) replaced by 04's canonical ids (`EH.SPL.TET_TAV`, `EH.SPL.KUF_KAF`, `EH.SPL.KAF_HET`, `EH.SPL.SAMEKH_SIN`, `EH.SPL.VET_VAV`, `EH.SPL.HE_ALEF_END`, `EH.FIN.WRITE_BASE/MID`, `EH.SPL.KM_*`, `EH.NK.SAME_SOUND`, `EH.NK.VOWEL_SWAP`); `MATH_TABLE_NEIGHBOR→EM.MUL_NEIGHBOR`, `MATH_ADD_FOR_MULT→EM.MUL_ADD`. The P0-E1-S6 ⟂VERIFY became a CI check "every alias target exists in the catalog" |
| F3 #11 | `numerals.json` single source, `numPhrase`, vav rule, MATH-AC-25, ⟂VERIFY חֲלָקֵי | already resolved | §1.5.3, §12.1 |
| F3 #12 | K2 stories: 1:1 lexicon mapping, V08 fail | already resolved | §2.5 |
| F3 #13 | Notation confirm line in the review screen | already resolved | §3.5.6 |
| F3 #14 | Counted nouns copied from NOUNS | already resolved | §2.4 |
| F3 #15 | ~300 Hebrew clips | already resolved | §6.10.2 |
| F4 #31 | P3 chips-only (00 C7a) | already resolved | §1.3.2, §4.2, §4.4, §4.6, §10 (`CHILD_CHIPS`, no `TextInput` under `app/(child)`) |
| F4 #32 / #33 | 435 cells / 144 answers | already resolved | §4.5 |
| F4 #34 | 45 English files, 05 §6 canonical | already resolved | §6.10.1 |
| F4 #35 | Mr. Mix-Up voice + 4 HVPT talkers | already resolved | §6.2 |
| F4 #36 | `ladder_topics` / `calm_suppress` blocklists | already resolved | §12.3 |
| F4 #37 | Errata pipeline `fact.reported` | already resolved | §2.7.1 (≤7 days) |
| **F5 → 12 #1** | "דגלי P3 (ספירות)" in the weekly-summary LLM input | **applied (larger than asked)** | §5 rewritten: the **weekly report has no LLM at all** (10 §11.7: templates T-W-*, deterministic); Haiku 5.5 only for the **monthly** strengths narrative from `StrengthsNarrativeInput` (08 §8.4; `{CHILD}`, no name/gender/flags/check-in/EF). §0.1, §0.5, §1.1 (`parent.weekly` → `parent.monthly_strengths`), §4.7 (distress chip → same-day alert via 10 §10, never through a report or an LLM), §5.1–5.5, §9, §10 P4 (network test: 0 Anthropic calls in the Friday job; `GET /reports/weekly` schema without flags) |
| F6 → 12 #1 | Nightly cron 02:10 | already resolved | §2.1 |
| F6 → 12 #2 | `vo.generic.<intent>` validator | already resolved | V22 |
| F6 → 12 #3 | Vendor-down/cap mechanism referenced from 11 | already resolved | §8 |
| F7 #2/#5, F9 GAP-14e, R1 OPD-08 rev 3, R6 | AI caps unification | **applied** | §1.9 cites OPD-08 rev 3 (console ≥ soft cap, default $50; $3/$40 soft) and 11's field `families.prefs.consoleCapCents`; consistency rule `MONTHLY_USD_CAP_EFFECTIVE = min(MONTHLY_USD_CAP, consoleLimit − $5)` if the parent picks another console value. 11 §8.4 now has 300/4000 — aligned |
| F7 #12 / R6-2 / 00 E9 rev 3 | Music loudness −23 vs 09 −26 | **applied** | §6.9 music → −26 LUFS (09 owns the mix table); VO −18 already aligned |
| F8 → 12 #1 | Audio format rules + pack size budget in validators | already resolved | V21 |
| F9 GAP-16 | Photo-game surfacing wording | already resolved | §3.1 #13 |
| F9 GAP-47 / F7 #13 | `kid_flag` approval type | **applied** | §1.8 names A7 (now in 10 §6.10) and the card behaviour; 3 flags/week → family quarantined |
| F9 GAP-52 | `photo.rerender_variants` | already resolved | §3.1, §3.13 |
| F9 §3.9 | Clone-failure parent copy | already resolved | §3.9/§3.15 |
| F9 GAP-67 / R6 §7.13 | Parent voice clips storage/distribution | **applied** | §6.11 aligned to 11 §7.13: `POST /voice-clips` (≤10 s / ≤200 KB), R2 `olam-private/voice/<familyId>/<guardianId>/<clipId>.m4a`, family pack `family.voice` (`packs.source='family_voice'`), re-record replaces, delete propagates, export as files, never to an AI vendor |
| R1 (00 rev 3) | B7 missed-window rule; OPD-35/36; LUFS; caps | **applied** | §5.1 late-send rule (GAP-23b); §11 OPD-35 adopted + range OPD-01…36; see rows above |
| R6 (11 photo pipeline) | 11 §7.9/§8.3 now = 12 §3.1/§3.3/§3.16 | already consistent | no change (12 was the source) |

## 2. Issues targeting 13

| Source · id | Topic | Disposition | Where / note |
|---|---|---|---|
| F1 #7 | magic link → email one-time code; Cloudflare Access optional | already resolved | §3.1, RK-05, §19 #1 |
| F1 #8 | Allowlist cite OPD-33 | already resolved | §16.8 |
| F1 #20 | `docs/parent/questions.md` from the OPD table; Q→OPD map | already resolved; **applied** rev 3 | §15.5, §18.4 (now "רוויזיה 3", OPD-35/36 rows no longer "ממתין ל-00") |
| F1 #21 / #25 | OPD ids / bedtimes | already resolved | §16.2 |
| F1 #29 | C3 onboarding vs staging with a placeholder family | already resolved | §4.3 |
| F2 #22 | ⟂VERIFY re-sign preserves SQLite; `preInstallSync` | already resolved | §3.3 #14 (= 00 E1a rev 3) |
| F2 #26 | Tier A/B/C mechanics order | already resolved | §5.3 |
| F3 #27 / **R3 #8** | MATH-AC-20…25 / HEB-AC-19…30 in the test matrix; **new** MATH-AC-26, HEB-AC-24/31 | already resolved + **applied** | §10.1 acceptance row now `MATH-AC-20…26`, `HEB-AC-19…31` |
| F3 #28 | OPD ids in 03/04 | n/a | nothing for 13 |
| F4 #38 / #39 | VS scope (bottles, MX-E01; 06 12 packs + 3 K.TOT by MVP); 1,000-week scheduler CI job | already resolved | §5.2, §5.6, §10.1 |
| **F5 → 13 #1** | Event registry in the C3 checklist; weekly email blocked on OPD-05 | **applied** | §4.3 C3 bullet + P0-E7-S7 (`packages/shared/events.ts`, 10 §4.1's missing events, `item.answered` v2, `SessionEndReason`, CI "every 10 §4.2 metric from fixtures" = 10 AC-38); §5.3 sprint 5 `10/PARENT-WEEKLY` blocked on OPD-05 (inbox + web until the domain; template-only, no Anthropic dependency) |
| **F5 → 13 #2** | S-5/S-6/S-7 and 10 AC-38…53 in the CI matrix | **applied** | §10.1 acceptance row (S-6 reports, non-blocking until calibrated) |
| F6 → 13 #1–#9 | resign.sh path, launchd, tunnel, `preview` channel, viewports, `__olam`, memory, OTA guard, E-01…E-24, parent checklist | already resolved | §3.1–3.3, §8.1, §10.1–10.3, §17 |
| F8 → 13 #1–#9 | preview channel, vitest/jest-expo, `__olam`, 20 cycles, memory target/gate, viewports + spec-14 section, DoD, severity map, weekly signing | already resolved | §9, §10.1–10.3, §17 |
| F9 GAP-48 / **R6-6** | Maestro layer | already resolved (partial) → **applied** | §10.1 Maestro row: `apps/mobile/e2e-maestro/*.yaml`, Simulator per native build + one real iPad weekly, `testID`, dev links `olam://dev/fixture` / `olam://dev/setNow` (11 §4.1), mapping of 11 §12.4 rows |
| F9 §10.3 rows / §3.2 ⟂VERIFY | expiry banner, island cap, picture-PIN reset, family-delete grace; data survives re-sign | already resolved | §10.3, §3.3 #14 |
| F7 #4 (own, to 00) / **R1-8** | C4 date | **applied** (00 I3 rev 3 adopted it) | §2.2, §2.3, §4.0, §4.4, §5.2 heading + exit criteria, §16.2 "עד VS": alpha 6.12 (greybox-plus), VS complete ~17.1.2027 (K1+K2), K3 by end of January |
| **R1-9** | OPD-35/36 "מוצע" | **applied** | §7.1 #3, §16.2 #45/#46, §18.4, OPD range |
| R1 OPD-08 / OPD-26 | console cap wording; C4 question 4 cap defaults | **applied** | §16.2 #15; §4.4 Q4 default = 10 §6.13 / OPD-26 (K1 40 · K2 25 · K3 15; cap day = `DayKey`) |
| self-found | §2.2/§4.0/§4.x/§5.x effort numbers still the pre-§7.1 estimate | **applied** | all "מאמץ" lines and both tables cite §7.1 (C1 4–5, C2 11–13, C3 9–11, C4 30–38, C5 8–10, C6 55–70, C7 50–65, v2 60–80) |
| **R4-3** | 06 skill count | **applied** | §5.6, §7.1: 134 (⟂VERIFY dropped); 07 = 72 verified |

## 3. Spec 15 (journeys / gaps)

- New **§3.1 Round 2 resolution status**: all 64 GAPs + the 6 non-GAP rows, each with status and the owning section that now carries the rule, checked against the owners' current text (`round2-R1…R6.md` + greps). Totals: 58 resolved (6 with modified values/names: GAP-04, 15b, 24, 39, 48, 53-partial), **3 rejected** (GAP-03 — 00 OPD-26 rev 3 keeps K3 15 min; GAP-31 — 07 §2.4 is canonical and 01 already matched, round 1 misread it; GAP-44 — 00 C3 rev 3: one `DayKey`, midnight in the family city), 1 partially open (GAP-53 copy in 10).
- §3 rows whose proposal was rejected/changed carry a "**Round 2:**" note with the binding reading (GAP-03/04/14e/15b/24/25/31/39/44/48/68).
- Journey sentences that stated now-rejected/changed rules were corrected (UC-011, 016, 034, 035, 036 chips-only, 040, 045, 054, 057, 059 template-only, 069, 072, 077, 078, 079, 082, 083, 090, 091); `GAP-nn` references elsewhere in journeys are kept as the audit trail (note in §0.2).
- §0.2 `[MAE]`: Maestro path/links per 11. §4 matrix: new **14** column (performance playbook) + updated cells. §5: parent items 1, 3, 4 struck as decided by 00 rev 3.

## 4. New issues for other owners (few; non-blocking unless marked)

| # | Owner / file | Problem | Fix |
|---|---|---|---|
| R7-1 (**blocker-ish**, consistency with 00 E6a / 10 §11.7) | R6 — 11 §7.1 step 2(a) and §7.3 `POST /ai/reports/summary` | Still describe a **weekly** Haiku Batch ("monthly-strengths/weekly Haiku Batch", "Friday batch 06:05"). 10 §11.7 (owner of reports) makes the weekly report template-only; Haiku is used only for the monthly strengths narrative (attached to the first Friday report after month end). 12 §5 and 13 now say so. | Reword step 2(a): "on the first send day after month end — submit the monthly-strengths Haiku Batch (`ai_jobs kind:'parent_summary'`, one request per consenting child, input `StrengthsNarrativeInput`)"; the weekly report never waits on AI. Rename the route to `/ai/reports/monthly-strengths` or document that it is monthly. My round-1 note F7 #10 ("weekly Haiku batch") is withdrawn. |
| R7-2 | R6 — 11 §7.10 / email sender | 10 §11.2 defines the Resend retry schedule (3× over 2 h: +10/+30/+80 min, then inbox-only + SET-07 "לשלוח שוב"). 11 does not state it (GAP-49b). | Add the retry schedule to the `reports.email` step (idempotent on `sent_at`). |
| R7-3 | R6 — 09 SCR-13 | K3 selected in the co-op lobby must request the parent-present gate (15 UC-030 E6; 01 §12 requires parent-present for K3). | One line in SCR-13. |
| R7-4 | R5 — 10 FAM-03 | 01 §10.1 (R2) blocks a 4th child in v1; 10 lacks the message (R2 proposed "בגרסה הזו יש מקום לשלושה ילדים"). GAP-53. | Add the blocked state + copy. |
| R7-5 | R5 — 10 §6.10 / PA-SET | 12 §1.9 now exposes `families.prefs.consoleCapCents` next to the two soft caps in PA-SET (11 `PATCH /admin/limits` refuses a soft monthly cap above it). | Show the console value (read-only) under the AI-cap settings. |

## 5. Notes

- 12 §3.7/§12.2 aliases were updated after R3's file appeared (23:21); 15's pass ran after R6's (23:27). Later edits by other owners are not reflected.
- Not changed on purpose: 12 §13 "Proposed change to 00" stays as the historical record (all six items were adjudicated in 00 rev 2/3). 13 §19 likewise.
- Nothing committed; no files outside 12/13/15 and this report were edited.
