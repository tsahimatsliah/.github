# Round 2 — R3 (owner of `03-curriculum-math.md`, `04-curriculum-hebrew.md`)

Date 2026-11-02. Rules followed: edited only 03/04; language and structure kept; parent > 00 > owning spec (08 for engine/levels/flags, 01 for mechanics/co-op, 02 for economy, 12 for pipelines) > evidence. Both files carry a "### Round 2 changelog" under their "## Fable review changelog".

---

## DECISION — error-code namespace (final; binding for 03/04, coordinated with 05/12)

| Namespace | Meaning | Owner | Status |
|---|---|---|---|
| **`EM.*`** | **Math error codes** (was `E.*` in 03) | 03 §4.2, catalog file `packages/content/math/errors.json` | **Applied in 03** (566 occurrences renamed, mechanical `E.X → EM.X`, no other name changes) |
| **`EH.*`** | **Hebrew error codes** (was `E.*` in 04) | 04 §2.3, catalog file `packages/content/hebrew/errors.json` | **Applied in 04** (256 occurrences renamed, mechanical `E.X → EH.X`) |
| **`E.*`** | **English *skill* IDs only** (`E.PA.*`, `E.PH.*`, `E.VO.*`…) | 05 §3 | Unchanged; 05 line 120 / changelog already states this and points a ⟂VERIFY at this file → **confirmed, ⟂VERIFY can be closed** |
| `ERR.*`, `SIG.*` | English error / signal codes | 05 §3.5 | Accepted as-is (no collision). `EE.*` is *not* required; 05 keeps `ERR.*` |
| `MATH_*`, `HE_*`, `NIK_*` | 12's photo-pipeline codes | 12 §3.7/§12.2 | Remain **aliases** in `packages/content/errors/aliases.json`; `classifyError()` accepts an alias and always returns the canonical `EM.*`/`EH.*`. A new code is born only in 03/04 |

Why `EH.`/`EM.` and not F5's `HE.*`/`ERR.H.*`: parallel to the skill prefixes `M.`/`H.`/`E.`, two characters, and `HE.*` would be confusable with 12's `HE_*` aliases. Engine impact: none beyond strings (`Skill.errorTags`, `Attempt.chosenErrorTag`, `Attempt.steps.firstErrorTag` now carry `EM.`/`EH.`; `SkillId` strings can never collide with them). Lint rule recorded in 03 `MATH-AC-26` and 04 `HEB-AC-24`: every error code in a math pack matches `^EM\.`, in a Hebrew pack `^EH\.`; any `E.`-prefixed string in `errorCodes`/`errorTags`/`distractors[].errorTag` fails.

**Canonical Hebrew targets for 12's aliases** (12 §12.2 currently carries draft names `E.SPL.T_TET`, `E.SPL.K_Q`, `E.SPL.K_CH`, `E.SPL.S_SIN`, `E.SPL.B_V` — these never existed in 04 and must be replaced):

| 12 alias | Canonical (04 §2.3ט) |
|---|---|
| `HE_SOUNDALIKE_T` | `EH.SPL.TET_TAV` |
| `HE_SOUNDALIKE_K` | `EH.SPL.KUF_KAF` (כ↔ק) · `EH.SPL.KAF_HET` (כ↔ח) |
| `HE_SOUNDALIKE_S` | `EH.SPL.SAMEKH_SIN` |
| `HE_SOUNDALIKE_V` | `EH.SPL.VET_VAV` |
| `HE_GUTTURAL` | `EH.SPL.ALEF_AYIN` · `EH.SPL.HE_ALEF_END` |
| `HE_FINAL_FORM` | `EH.FIN.WRITE_BASE` / `EH.FIN.WRITE_MID` |
| `HE_MATER_OMIT` / `HE_MATER_EXTRA` | `EH.SPL.KM_MISSING` / `EH.SPL.KM_EXTRA` (and `EH.SPL.KM_DOUBLE`) |
| `HE_VISUAL` | `EH.VIS.<PAIR>` (pair-specific; served only when both letters ≥70% learned, 04 P12) |
| `NIK_KAMATZ_PATACH_OK` | `EH.NK.SAME_SOUND` |
| `NIK_A_I` / `NIK_E_A` | `EH.NK.VOWEL_SWAP` |

Math aliases are unchanged except the prefix: `MATH_OP_SWAP→EM.OP_SWAP`, `MATH_OFF_BY_ONE→EM.NEAR`, `MATH_DIGIT_REVERSAL→EM.REVERSE_DIGITS`, `MATH_PLACE_VALUE→EM.PV_CONCAT` (22 without carry = `EM.NO_CARRY`), `MATH_CARRY_OMIT→EM.NO_CARRY`, `MATH_BORROW_OMIT→EM.SUB_SMALL_FROM_LARGE` (variant `EM.BORROW_NO_DEC`), `MATH_TABLE_NEIGHBOR→EM.MUL_NEIGHBOR`, `MATH_ADD_FOR_MULT→EM.MUL_ADD`, `MATH_COUNT_ALL_AGAIN→EM.COUNT_ALL`, `MATH_HALF_DOUBLE→EM.HALF_DOUBLE`.

---

## Disposition of every issue targeting 03/04

Sources read: `fable-issues-F1…F9.md` and spec 15 §3 (no GAP targets 03/04; the only 03/04 mentions are in the §4 traceability matrix and are consistent). F6, F8 have no 03/04 items. F7 #15 targets 10 and merely cites 03 §1.5 (no change needed in 03).

| Source | Item | Target | Disposition | Where / what |
|---|---|---|---|---|
| F1 | #8 allowlist → cite OPD-33 | 04 §17 #6 | **already resolved** (round 1) | 04 §17 #6 carries OPD-33 |
| F1 | #21 local parent lists carry `OPD-nn` | 03 §16, 04 §17 | **already resolved** | both lists carry OPD ids; "00 defaults win" sentence present |
| F1 | #30 C13 safety tags on missions/printables | 03 §12.3, 04 §14.1 | **already resolved** | 03 §12.3 table (`HOME.*`, `PRN.*`), 04 §14.1 + HEB-AC-28 |
| F2 | #1 MX ids for 03 mechanics | 03 §5.4 | **applied** | §5.4 rewritten: `MX-M13…M20`, variants `MX-M03:lasso`, `MX-M04:cart/hut`, `MX-M07:table-map`, `MX-M12:board-game`; alias file `packages/content/mechanics.aliases.json`; build tier per 01 §7.1; `MATH-AC-03` extended |
| F2 | #2 play shekels never enter the economy | 03 §5.1/§5.4 | **applied** | MAKOLET row cites 02 §2.7 `neverEarns`, 02 AC-ECO-5, 01 AC-MX-7 |
| F2 | #9a MX-H12…H15 + variants, alias table, §14.4 verbatim in MX-H14 | 04 §7.3א, §14.4, HEB-AC-23 | **applied** | all "הצעה ל-01" rows replaced; variants `MX-H05:syllable-clap`, `MX-H07:summary-chest`/`story-campfire`, `MX-H08:readers-theater`/`echo-cave`/`phrase-bridge`, `MX-H11:prefix-peeler`; tiers (H12 in S0); §14.4 link note; AC-23 now checks alias file + shared snapshot |
| F2 | #9b consume `hint:lens{wordId, lexId}` | 04 §6.4 rule 4, §6.6 | **applied** | count from the event only (= 08 §1.9 `BehaviorSignal{kind:'nikud_lens'}`, wire `behavior.signal`); `lexId` feeds the per-word review card; new HEB-AC-31 |
| F2 | #9c cite 01 §12.2 for touch zones/`assisted` | 03 §5.3, §13.7; 04 §13.4, §14.5 | **applied** | rules quoted from 01 §12.2 (binding) instead of restated; `Attempt.context.assisted` (08 §1.4), w ×0.3, `itemRecovered` (02); `sibling` only on parent mark |
| F2 | #9d `CoopZoneLayout` in 09 | 09 (not mine) | **forwarded** | 09 still has no `CoopZoneLayout`; 03 §13.7 cites it as "requested from 09 §4.2, F2 #9d" |
| F4 | #40 keep 7 IDs stable for 06 `crossPrereqs` | 03 §1.1 | **applied** | all 7 exist; frozen-ID list in §1.1 + `MATH-AC-26` |
| F4 | #41 scissors printables `adult-required` | 04 §14.1 | **already resolved** | `adult-required`, `choking-hazard-T` on `P.K2.CARDS`, `P.K2.BOOK`, `P.K1.THEATER` |
| F5→03 | #1 reference 08 §3.3א/§3.7/§5.1/§9.2; drop "temporary IDs" | 03 §17.2, §6.0א, §15 | **applied** | §17.2 is now a status table (all 8 requests accepted, with 08 section per row); "מזהים זמניים" → `questionId` of 08 §3.3א everywhere (§6.0א, MATH-AC-19); question numbers T6/T7/G7–G11/S7–S12 added |
| F5→03 | #2 `O.REP` on P-only skills | 03 §3.2, §6.0א, §6.x, MATH-AC-12 | **applied** | rule: `O.REP` only on a CPA path with both P and A. Removed from `M.NUM.SUBIT.C5`, `M.NUM.SUBIT.C10` (CPA P), `M.OPS.BOND.5` (C→P), `M.GEO.SHAPE.ID2D` (P). `M.MEAS.PERIM_AREA` keeps the gate and gains an A stage (`T.PERIM.AREA{grid:false}`, numeric dims). Parser rule in §6.0א `extraGates` row; pack validation (≥2 pictorial + ≥2 abstract reviewed templates) in MATH-AC-12 |
| F5→03 | #3 `band` derivation `T*→T, G*→G, S*→S` | 03 §6.0א | **already resolved** (round 1) | row `band`/`subBand` present; MATH-AC-24 |
| F5→03 | #4 `MATH_K1_READ_VS_LISTEN` is a marker for `HEB_K1_LISTEN_READ_GAP` | 03 §14.3 | **applied** | both rules "registered in 08 §9.2, `maxStatus:'watch'`"; READ_VS_LISTEN has no own message; week counts as met at θ-gap 0.7–1.0 with ≥15-pt read/listen delta; audio prompt on by default in `watch` |
| F5→F3 | #5 align profile-swap wording to `identity_suspect` / `identity_pending` | 03 §14.4, §13.7, MATH-AC-23; 04 §13.4, HEB-AC-27 | **applied** | 03 §14.4 rewritten to 08 §10.5.3 (score from item 10, `excluded:'identity_pending'` not `disengaged_block`, `session_reattribute`, 48 h → `unknown`, reversible 30 d, picture-PIN offer after 2 ✓/30 d, threshold 5 with picture-PIN, co-op `assisted:'sibling'` taps never trigger); 04 row + AC updated |
| F5→F3 | #7 break composition must sum (08 uses K2 20/30/30/0/20) | 03 §7.1; 04 §13.4 | **applied** | 03 §7.1 and 04 §13.4 now carry K1 15/25/50/0/10, K2 20/30/30/0/20 and `Plan.breakMode ∈ {'auto','on','off'}` (08 §1.10/§6.2א); 03's old `{"maintain","normal"}` enum dropped |
| F5→F3 | #1, #2, #3, #4, #6, #8, #9 (band/subBand, continuation strands, alias fan-out, O.STEP row, four flag rules, handedness, `H.WR.UNP.FAM#lx_…`) | 03/04 | **already resolved / applied as wording** | continuation + fan-out wording updated in 03 §6.0א and 04 §2.7 (`StrandMeta.priorFrom`, lazy placement, `w = 0.7 × weight`); 04 §15 rows for `HEB_K1_FADE_STALL`/`HEB_K1_HANDWRITING` now say "registered in 08 §9.2 (`watch`, `provisional`)" |
| F5→04 | #1 strand names must match 08 §1.1; publish questionnaire map T1–T5, T8, G1–G6, G12, S1–S6 | 04 §2.7, HEB-AC-20 | **applied** | names confirmed identical (`primary`/`continuation`, `priorFrom`, `hebrew.fluency` = `wcpm`); map published in `questionnaire-map.json` format with weights; added T3 `lang.receptive`, T4 `lang.receptive_body`, T5 `world.animals` (06-owned; 0.5 reinforcement here), S4 (not a skill); G12 split 1.0/0.5 |
| F5→04 | #2 `E.*` namespace collision with 05 | 04 §2.1, §2.3ט, HEB-AC-24 | **applied** | decision above; rename done |
| F5→05 | #3 (same collision) | 05 (not mine) | **coordinated** | 05 already carries the matching note with ⟂VERIFY → this file confirms |
| Coordinator | "04 must publish strands/spine/questionnaire map consumed by 08" | 04 §2.7 | **applied** | see F5→04 #1 |
| Coordinator | "K2 break composition must sum" | 03 §7.1, 04 §13.4 | **applied** | see F5→F3 #7 |
| F7 | #15 (10 SCR-14 notation confirm line, cites 03 §1.5) | 10 | **n/a** | 03 §1.5 unchanged; consistent with 12 §3.5.6 |
| F9 / 15 §3 | — | — | **none** | no GAP targets 03/04 |

Rejected: none. Modified from the proposal: F5's prefix suggestion (`HE.*` / `ERR.H.*`) → `EH.*`/`EM.*` (rationale above).

---

## NEW issues for other owners (only what blocks consistency)

| # | Owner / file | Problem | Fix |
|---|---|---|---|
| 1 | **R7 — 12 §3.7, §12.2 (lines ~1049, 1053, 1054), changelog ~2172** | Alias targets still written as `E.*`; Hebrew targets use draft names that do not exist in 04 (`E.SPL.T_TET`, `E.SPL.K_Q`, `E.SPL.K_CH`, `E.SPL.S_SIN`, `E.SPL.B_V`). Blocks the P0-E1-S6 ⟂VERIFY against `errors.he.json`. | Replace with `EM.*`/`EH.*` and the canonical table above; keep `packages/content/errors/aliases.json` as the single alias file; `classifyError` returns `EM.*`/`EH.*`. |
| 2 | **R2 — 01 §7.3 cards MX-H12 (~1415), MX-H13 (~1426), MX-H14 (~1431), MX-H15 (~1444), AC-MX-10 (~2234)** | Use `E.PA.CVWHOLE`, `E.PA.ADDVOWEL`, `E.VIS.<PAIR>`, `E.SPL.PHON`, `E.MO.*`. | Prefix → `EH.`. No other change. |
| 3 | **R5 — 08 §12.10ב U-STEP-5 (~1880), §1.2 `errorTags` comment, §3.3א last row** | `firstErrorTag='E.NO_CARRY'` → `EM.NO_CARRY`; the `errorTags` example (`'place_value_reversal' → 'math.pv.tens'`) should read `'EM.REVERSE_DIGITS' → 'M.PV.TENS100'`; the row "T1–T5, T8–T9, G1–G6, G12, G14, S1–S6 … ⟂VERIFY: 04 ו-06 טרם פרסמו" — 04 has now published (04 §2.7; T5 → 06). Also `StrandMeta.priorFrom` for math (03 §6.0א): `math.facts←math.operations`, `math.fractions←math.number`, none for geometry/measurement/patterns_data. | Cosmetic renames + close the 04 half of the ⟂VERIFY. |
| 4 | **R6 — 09 §4.2** | F2 #9d `CoopZoneLayout` (CO-1 rotated halves; CO-2 ≥1/3 width per child, 12 mm dead strip; gesture router drops cross-zone taps) is still absent from 09. 01 AC-CO-7, 03 §13.7 and 04 §14.5 depend on it. | Add the layout contract + snapshot per device class. |
| 5 | **R6 — 11 §5.8** | `behavior.signal{kind:'nikud_lens', wordId, lexId}` must be in the wire registry (F5 → 11 #1 already asks for `behavior.signal`); 04 §6.4 now counts lens taps only from this event. `item.answered` v2 `chosenErrorTag` / `steps.firstErrorTag` carry `EM.`/`EH.` strings (regex `^(EM|EH|ERR|SIG)\.` if zod validates them). | Add/confirm; no schema shape change. |
| 6 | **R4 — 05 line 120, changelog** | ⟂VERIFY on the 03/04 prefix direction. | Resolved exactly as anticipated (`EM.*`, `EH.*`; `ERR.*`/`SIG.*` kept for English); remove the ⟂VERIFY. |
| 7 | **R5 — 08 §3.3א, R4 — 06 §2 (R4-4)** | Ownership of questionnaire rows: `lang.vocab_expressive` (T1), `lang.combinations` (T2), `lang.receptive` (T3), `lang.receptive_body` (T4), `motor.name_writing` (G4), `motor.tracing` (G14) are mapped by **04 §2.7** (`H.OL.*`, `H.HW.*`, `H.SP.NAME`); `motor.fine` (T9) and `world.animals` (T5) stay with 06. | 08: point these `questionId`s to 04 in `questionnaire-map.json`; 06: do not duplicate them. |
| 8 | **R7 — 13 QA matrix / `13a-id-map.md`** | New acceptance items `MATH-AC-26` (namespace lint + frozen IDs) and `HEB-AC-31` (lens-event simulation); F3 #27's list grows by two. | Add to the test matrix of the sprint that lands `packages/content`. |

---

## Verification performed

- Rename: 566 `E.→EM.` in 03, 255 + 1 manual `E.→EH.` in 04 (regex `(?<![A-Za-z0-9_-])E\.(?=[A-Z0-9<*])`, then one hyphen-preceded case fixed by hand). Residual `E.` strings in 03/04 are only (a) references to 05 skill IDs and (b) historical mentions inside the namespace notes, 03 §4.2א, 04 §2.3ט and the changelogs. Lower-case `e.` (code/prose) untouched.
- Markdown tables in both files: 0 column-count anomalies after the edits.
- Numbers and names were checked against the **current** files of the owning specs: 08 (§1.1 strands/`Band`, §1.2 `extraGates` validation, §1.4 `Attempt.steps`/`context.assisted`, §1.9 `nikud_lens`, §1.10 `breakMode`, §3.3א, §3.7, §4.1, §6.2א, §9.2, §10.5.3), 01 (§7.1 tiers, §7.3 cards MX-M13…M20 / MX-H12…H15, §12.2, AC-MX-7/10/11, AC-CO-7), 02 (§2.7 `neverEarns`, AC-ECO-5), 00 (A6, C11, C12, C13), 05 (line 120 namespace note), 12 (§12.2 alias paragraph).
- Not changed: the 233 math skills (only 5 meta lines), the ~160 Hebrew nodes, lexicon, letter/nikud sequence, fade algorithm (except the count source), fluency, comprehension, morphology, writing, toddler programme, economy numbers, Hebrew numeral rules.
- Not committed (coordinator commits).
