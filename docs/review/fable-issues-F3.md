# Fable review — issues for other files, from F3 (owner of 03 math, 04 Hebrew)

Each row: file · section · problem · proposed fix. The owning reviewer applies it; 03/04 already reference the proposed names so nothing blocks.

## 08 — assessment engine

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 1 | §1.2 `Skill` | No `band`/`subBand` fields although 00 A6 tags skills by band; 03 §6.0א and 04 §2.7 derive them. | Add `band: 'T'\|'G'\|'S'`, `subBand?: string`, and `extraGates?: string[]` (00 C11 allows entry gates). Export `bandOf(child)` from `packages/shared`. |
| 2 | §3.2 placement strands | 04 defines 4 new Hebrew strands (`hebrew.oral`, `hebrew.fluency` (WCPM, not Elo), `hebrew.writing`, `hebrew.ran`) and 03 defines 5 math strands. Spine lists live in 03 §6.0א / 04 §2.7. | Treat them as "continuation strands": probed only if time remains (subject cap 25/14 items); otherwise prior from the named sibling strand. |
| 3 | §3.3 questionnaire IDs | Placeholder IDs (`hebrew.letter_sounds`, `math.add_sub100`, …) map to several `M.*`/`H.*` nodes each (03 §6.0א, 04 §2.7). | Keep the placeholders as `aliases` in the content index; questionnaire pseudo-observations fan out to every mapped node with the stated weights (0.7, or 0.35 where marked). |
| 4 | §4.1 scoring | 03 `O.STEP` needs `x = correctSteps/steps` (partial credit). 08's table already admits non-binary x; please state it explicitly. | Add the row "multi-step procedure item (K1): x = correctSteps/steps, w = 1". |
| 5 | §4.4 / new | Sibling-profile swap is undetected (K2 has no picture-PIN by default, H3). 03 §14.4 / 04 §13.4 define `possible_profile_swap` (accuracy 0.35/0.98 vs P̂, RT ×0.4/×2.5, 10-min adjacency) with parent confirm → `w = 0` for the session. | Implement in the composer: suspend block scoring from item 11 (`disengaged_block`), parent card at episode end, exclude on ✓, restore on ✗, exclude after 48 h without answer; offer picture-PIN after 2 confirmed events in 30 days. |
| 6 | §9.2 flags | 03 proposes `MATH_K1_FACT_PACE`, `MATH_K1_READ_VS_LISTEN`; 04 proposes `HEB_K1_FADE_STALL` (3 fade regressions in 8 weeks or >10 pp gap after 4 weeks of hold) and `HEB_K1_HANDWRITING` (illegible ≥30% in 3 consecutive sheets, or letters/min ≥40% below personal best for 6 weeks, or mirror reversals ≥20% in grade 3). All `watch`-only until approved. | Add the four rules with `provisional: true`; `HEB_K1_HANDWRITING` referral path = teacher + OT via pediatrician (C8). |
| 7 | §6.2 / §10.4 | 03 §7.1 and 04 §13.4 define a "maintenance mode" for school breaks ≥5 school days (K1 15/25/50/10, K2 20/30/40/10 incl. fun 20%; `monthAlign` of the next unit frozen; ≤6 reviews per episode after a long gap) and a `Plan.breakMode` toggle. | Add `breakMode` to `Plan` and the composition override; derive breaks from `IsraeliSchoolCalendar.vacations`. |
| 8 | §1.7 `ChildProfile` | 03/04 need `handedness: 'right'\|'left'\|'unknown'` (OPD-01 "dominant hand") for keypad/toolbar placement and printables. | Add the field (UI-only; never enters the engine). |
| 9 | §5.4 | 04 fade algorithm needs a per-word review card (`reviewMode:'item'` for unpointed-word knowledge) and reads `R = 0.9^(elapsed/ladder[rung])` from it. | Confirm per-item cards may be keyed by `lexId` (`H.WR.UNP.FAM#lx_…`). |

## 12 — AI, content & voice pipelines

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 10 | §3.7, §12.2 error codes | 12 uses `MATH_*`, `HE_*`, `NIK_*`; 03 §4.2 and 04 §2.3 are the owning catalogs (`E.*`). Mapping tables are in 03 §4.2א and 04 §2.3 ט. Mismatches: `MATH_PLACE_VALUE` conflates `E.PV_CONCAT` (95) and `E.NO_CARRY` (22); `MATH_BORROW_OMIT` = `E.SUB_SMALL_FROM_LARGE` (not "borrow without decrement", which is `E.BORROW_NO_DEC`); `HE_SOUNDALIKE_K` merges כ/ק and כ/ח which need different remediation; `HE_VISUAL` must be pair-specific (P12: only pairs both ≥70% learned). `MATH_HALF_DOUBLE` had no 03 equivalent → added as `E.HALF_DOUBLE`. | Switch to `E.*` or ship `errors.json` alias tables; `classifyError` must accept both names (03 `MATH-AC-22`, 04 #24). |
| 11 | §1.5.3, §12.1 numerals | 03 §10.4 and 12 §12.1 are two statements of the same table. 03 now declares `packages/content/hebrew/numerals.json` (12's file, 00 B4) the single source and wraps `numeral()` in `numPhrase()`. 03's rules add: vav-conjunction pointing (וּ before shva/בומ"ף, וַ before chataf, otherwise וְ; never וָ in numbers), hundreds (מֵאָה feminine → שְׁלוֹשׁ מֵאוֹת), thousands (שְׁלוֹשֶׁת אֲלָפִים…, "אֶלֶף" singular from 11,000), plural noun always after ≥2, ordinals, fractions, clock phrasing. | Extend `numerals.json` with tens/hundreds/thousands composition and the vav rule; verify construct forms (OPD-31); add the shared CI test (03 `MATH-AC-25`). Mark "חֲלָקֵי" vs "חֶלְקֵי" ⟂VERIFY before recording the ":" clip. |
| 12 | §2.5 K2 stories | 12's prompt lets the model return Hebrew word strings from `ALLOWED_WORDS`; 04 §5.5 requires lexicon IDs. 04 now accepts both, provided each token maps 1:1 to a lexicon `unpointedMale` entry (after stripping allowed prefixes) and **the pointed text is copied from the lexicon, never produced by Nakdan/Phonikud for K2**. | State the 1:1 mapping step in §2.5 processing and in V08; a token with no exact lexicon match = V08 fail → revise. |
| 13 | §3.5.6 photo review | Notation detected from a worksheet (`·` vs `×`, `:` vs `÷`, borrow marks) must not switch the child's display silently (03 §1.5). | Add a confirm line to the review screen: "בכיתה כותבים · במקום ×. לעדכן?" → writes `notation.*` only on ✓. |
| 14 | §2.4 skin prompt | Prompt rule 3 forbids numbers but 03 §10.3 also forbids counted nouns outside slots and requires `{{N:k}}` noun slots so `numPhrase` can agree gender; current `{{a}} מגדלים` form is fine only if the noun is taken verbatim from NOUNS. | Add "nouns counted by a slot must be copied exactly from NOUNS (no inflection by the model)"; validator rejects inflected forms. |
| 15 | §11 recording plan | 04 §17 #2 now cites OPD-13 and defers the consolidated plan to 12 §11 (per F1 #26). | Nothing further; keep ~300 Hebrew clips (23 consonants, 5 vowels, ~120 CVs ×2 speeds, closed syllables) per 04 §4.4. |

## 01 — game design

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 16 | §7.1 catalog | 04 §7.3א maps Hebrew mechanics to `MX-H01…H11`; five have no equivalent: **`MX-H12` rune-splitter** (CV → consonant + vowel; the core K2 predictor `H.PA.PH.SPLIT` has no mechanic in 01), **`MX-H13` twin-detective** (visual letter discrimination with a magnifier), **`MX-H14` writing-studio** (tiles + parent mediation script; glow-trace is explicitly not writing, P8), **`MX-H15` word-machine** (oral morphology: singular→plural, m→f, agreement), plus variants: syllable-clap on `MX-H05`, readers' theater / summary chest on `MX-H07/H08`, prefix-peeler on `MX-H11`. | Add the four mechanics (or formally declare them variants) so the "one gimmick per level" rule has a gimmick for PA split and letter discrimination. 00 G3 allows. |
| 17 | §7.1 catalog (math) | 03 §5.4 still lists `DUEL`, `TANGRAM`, `CLOCK_CAMP`, `MAKOLET`, `BALANCE`, `MIRROR`, `SCOUT_CHART`, `NUMBER_TALK` as "proposals to 01" (unchanged from the first draft). | Decide: add as `MX-M13…M20` or map to variants; the `NUMBER_TALK` strategy-chip UI is also 07's `MX-T01` tool-belt — reuse it. |
| 18 | §9.5 campfire defense | 03 §5.3 co-op and 04 tutor-mode need per-child `assisted` tagging and separated touch zones so K3 taps on the shared iPad never count as K2's answers (03 §13.7). | State the touch-zone rule and the `assisted:'sibling'` flag in the co-op section. |
| 19 | MX-H06 fog-goggles | "Goggles lend the nikud for 2 seconds without lowering the track" matches 04 §6.6; but 04 also counts ≥8 lens taps per 100 words as a difficulty signal (§6.4 rule 4). | Emit a `hint:lens` event per tap so the engine can count. |

## 09 — UX/UI

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 20 | §3.6 typography | 04 §7.4 now defers to 09 but asks two things: (a) ⟂VERIFY line height 1.55/1.6 for **two consecutive pointed lines** (measure "חֲנֻכָּה / שֶׁלָּהּ", Noto 500, 28 pt; chataf/qamats of the upper line must not touch the lower line's letters; if they do, 1.7 for multi-line pointed text); (b) a way to render nikud marks at ~120% in `type.read.k2.*` (glyph-baker or a font-feature), since `fontVariant` is forbidden. | Add the measurement to the font gate and decide (b). |
| 21 | §4 / components | `profile.handedness` (03 §1.5, 04 §11.1): numeric keypad, column pad and "done" button on the left for left-handed children; trace-warmup toolbar on the right. | Add a `handedness` prop to `CMP-*` input components and a snapshot per hand. |
| 22 | §9.2 praise | 03 §13.5 praise keys `praise.math.{strategy}` and 12 §2.6's five process lines must land in 09's approved list. | Reserve the key namespace; lint `UX-AC-11` runs on them. |

## 10 — parent area

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 23 | reports | 03 §13.6 and 04 §14.1 define a monthly paper probe with a per-item ✓/✗ grid entered by the parent (not a total score), a `paperGap` metric, and a "teacher said…" free-text field per unit. | Add the grid screen (`PA-PROBE`), show `paperGap` next to app accuracy in the monthly report, and the free-text field. |
| 24 | settings | New per-child settings: `notation.mulSign/divSign/carryRow/borrowMark` (03 §1.5), `handedness`, `plan.breakMode` (03 §7.1), `classTopics[].order:'school'` (03 §7.5), fade `fadeHold` release (04 §6.4). | Add to the child settings screen with 00 defaults. |
| 25 | profile swap | `possible_profile_swap` parent card (03 §14.4): "נראה ש{שם} שיחק/ה בפרופיל של {שם}? ✓/✗", plus the picture-PIN offer. | Add the card and the offer. |

## 06 / 07 — bands

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 26 | 07 §0.2, 06 | 03 §1.2 uses sub-bands `T+`, `G+`, `S-R`, `S+` mapped onto 07's `T/G/S`; 04 §2.1א the same. If 07/06 ever need finer granularity, reuse these `subBand` names rather than inventing new ones. | No action unless 07 adds sub-bands. |

## 13 — delivery / QA

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 27 | QA plan | New acceptance criteria to schedule: 03 `MATH-AC-20…25`, 04 `HEB-AC-19…30` (simulation tests for edge cases, paper probe, profile swap, fade regression, band/level mappings, shared numerals test). | Add to the test matrix of the sprint that lands `packages/engine` + `packages/content`. |
| 28 | parent questions | 03 §16 and 04 §17 now carry OPD IDs (OPD-01/02/04/13/20/21/31/33). | Nothing further. |
