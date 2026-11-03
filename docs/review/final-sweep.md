# Final consistency sweep — summary of changes (2026-11-02)

Single editor for the whole corpus after rounds 1–2. Inputs: `REVIEW-BRIEF.md`, `ROUND2-BRIEF.md`, `round2-R1…R7.md` (every "forwarded"/"new issue" row), the current specs 00–15. Outputs: in-place edits to 13 specs (each with a `### Final sweep changelog`), the new `docs/spec/16-verify-register-and-index.md`, this file. Nothing committed.

## 1. Leftover forwarded items (Step 1)

| Item (source) | Target | Disposition |
|---|---|---|
| R3 #2: MX-H12…H15 cards + AC-MX-10 use `E.PA.*`, `E.VIS.<PAIR>`, `E.SPL.PHON`, `E.MO.*` | 01 §7.3, §17 | **applied** → `EH.*` (5 lines) |
| R3 #6 / R4-5: ⟂VERIFY on the 03/04 prefix direction | 05 §3.1, changelog | **applied** — closed; note rewritten to the final state (`EM.*`/`EH.*`, aliases file) |
| R4 non-blocking: Mr. Mix-Up cast row ⟂VERIFY; `palette.english` note | 05 §4, §20.2, changelog | **applied** — closed (01 §4.1 CH-04 row and §4.5 exist; 01 withdrew `#19B7B0`) |
| R3 #3 / R6-4 / R2 → 08: U-STEP-5 `E.NO_CARRY`; `errorTags` example; §3.3א ⟂VERIFY (04 half); math `priorFrom`; `questionId` ownership of `lang.*`/`motor.name_writing`/`motor.tracing`; `nikud_lens` kind; `safe_room` wire-only; `levelMode` from `EpisodePlan.mode` | 08 §1.1, §1.2, §1.4, §1.10, §3.3א, §12.10ב | **applied** — all; `nikud_lens` was already in §1.9 (round 2); §3.3א ⟂VERIFY closed for both 04 and 06 (both published their maps) |
| R6-3: `familygame://` in 10 §11.9 | 10 | **already resolved** (round 2); final sweep aligned 10's allowed-path list to 11 §4.1's closed list and fixed 15 UC-059 E8 |
| R7-4 / R2 → 10: FAM-03 4th-child block + "show the opening again" | 10 FAM-03 (+ 11 §7.3 `409 child_limit`) | **applied** |
| R2 → 10: W-ECO-06 "תן מדבקה" K3-only | 10 | **already resolved** (round 2, GAP-28) |
| R7-5: PA-SET shows `consoleCapCents` | 10 SET-07 | **applied** (read-only line, default $50) |
| R7-1: 11 §7.1 step 2(a) / §7.3 `POST /ai/reports/summary` weekly Haiku | 11 | **applied** — monthly strengths only; weekly report template-only (0 LLM calls); route renamed `/ai/reports/monthly-strengths` (alias kept); §9.8 reference updated |
| R7-2: Resend retry schedule mirror | 11 §7.1 step 2(c) | **applied** (3× in 2 h: +10/+30/+80, idempotent on `sent_at`, never across the Shabbat window) |
| R6-5: `rf_raised` is a field of the encrypted flags doc | 07 §2.7 | **applied** |
| R5 #1 / R6-7: `DayKey` = family-/travel-city day | 02 §2.1, §10.1, UC-16 | **already resolved** (round 2) — verified |
| R2 → 05/08: K1 English placement timing (08 E3 Tue vs 05 week-3 bottle) | 05 §14.1, §15.1; 01 §14.3 | **applied** — 08's E3 (week 1, Tue) probes `english.listening` + `english.letter_sounds`; 05's week-3 "המכתב של Sunny" is the story wrapper for the lazy probes (oral_vocab, decoding, minimal pairs, questionnaire); 08 §3.2 unchanged |
| R6-6: Maestro path | 13, 15 | **already resolved** (round 2); only history rows keep `tools/e2e/maestro` |
| R7-3: 09 SCR-13 K3 gate | 09 | **already resolved** (round 2) |
| R1-1…R1-10, R2 → 09/11/12/07, R4-1…R4-3, R5 #2–#5, R6-1/R6-2 | various | **already resolved** by the owners in round 2 — verified by grep against the current text |
| 06 §12.6, 07 §0.2, §2.6, §7.3 "⟂VERIFY round 2: 11/02/13 do not yet …" | 06, 07 | **applied** — closed (11 §7.6/§7.7/§5.8, 02 §2.2/§2.7, 13 §5.6 now carry the items) |
| 15 §3.1 GAP-49b / GAP-53 "open on the 11 / 10 side" | 15 | **applied** — closed |

## 2. Cross-reference integrity (Step 2) — violations found and fixed

Script: `scratchpad/sweep/integrity.py` (throwaway). Final run: **0 violations** in every automated check over 00–16; full counts in 16 §3.

- (a) skill ids: 01 `M.FRAC.COMP` → `M.FRAC.COMPARE.*`; 01 `M.DATA.PICTO` → `M.DATA.PICTO.G`; 04 §2.3ט no longer spells the five never-existing draft names as `E.`-prefixed tokens. (Definition rules learned on the way: 05 generates `E.PH.<UNIT>.*`/`E.TW.<UNIT>.READ`/`E.VO.<TOPIC>.REC`/`E.SG.<SONG>.*`; 04 rows use `/` alternatives and `B1–B5` ranges; 06 bolds/italicises pack ids; 07 activity ids are `ACT-`/`OFF-` prefixed and referenced bare.)
- (b) `MX-*`: 0. (c) `OPD-nn`: 0. (e) AC uniqueness: 0 real duplicates (summary tables, `AC-31..39` ranges and line-start continuations are not definitions).
- (d) decisions: 01 `00 D14` (does not exist) → `00 A4` + `E10`/OPD-10; 01 "00 C7, F9 GAP-46" → "00 C7a; 15 GAP-46"; 01 `00 B9a/B9b/B9d` → `00 B9 (a)/(b)/(d)`.
- (f) error codes: the 01 renames above; 08 `EM.NO_CARRY` and `'EM.REVERSE_DIGITS' → 'M.PV.TENS100'`; 12 alias targets all resolve.
- (g) numbers: 12 §8 music loops `−23` → `−26 LUFS`; 15 UC-078 "$30/$40 (same GAP)" → console ≥ soft cap default $50; 15 UC-043 "havdalah +40" → +30; 10 §11.9 deep-link paths → 11 §4.1's list. AI caps, VO LUFS, K3 cap, bedtimes, celebration budget, memory tiers, touch targets, Friday window, Shabbat window, OTA channels, Maestro path and versions were consistent everywhere else.
- (h) banned items: 09 §2.5 listed "Nano Banana" as a usable tool → removed (00 E8a); 02 §14 and 09 §6 cited Expo Go as a fallback runtime → web (11 §3.1); 13 §3.1 offered `eas go` for phase 2 → removed; 12 §1.3/§6.4/§7.2 gated pages behind Sign in with Apple → guardian session (email code in phase 1); **04 §11.3 + §12 templates sent a K1 composition / free-text answer to Sonnet/Haiku → on-device scoring in code, LLM only on a parent-initiated photo upload (00 C7a)**. `useFonts`, `player.release`, `InteractionManager`, `I18nManager.isRTL`, loot/random chests and trait praise: 0 (only bans, "instead of" columns, linguistic examples).

## 3. Register and index (Step 3)

`docs/spec/16-verify-register-and-index.md`: master index of 00–16 with one-paragraph summaries and anchors; register of all remaining ⟂VERIFY claims (99 markers / 97 lines in spec bodies; 19 are legend/log/pointer lines; the claims are 14 web-research rows, 18 on-device rows, 7 parent rows, 20 at-build rows — none blocking the vertical slice); integrity results; **Spec freeze v1.0 (2026-11-02)** with nine rules (ADR in `docs/decisions/`, update 00's index in the same change, owner-only edits, stable ids, ⟂VERIFY lifecycle, changelogs, OPD for parent questions, integrity gate, freeze scope).

Markers removed in place this sweep (resolved): 05 §3.1, 05 §4/§20.2 (×3), 06 §12.6, 07 §0.2, §2.6, §7.3, 08 §3.3א, and 16 journey markers in 15 (UC-003, 007, 010, 015, 023, 026, 029, 030, 039, 041, 045, 049, 055, 061, 066, 090).

## 4. Files touched

Edited: 01, 02, 04, 05, 06, 07, 08, 09, 10, 11, 12, 13, 15 (each with `### Final sweep changelog`). Created: `docs/spec/16-verify-register-and-index.md`, this file. Not edited: 00 (revision 3 stands; the two stale `00 D14`/`00 F9` references were in 01), 03, 14.

Not committed (coordinator commits).
