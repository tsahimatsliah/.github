# Round 2 — R1 (owner of `docs/spec/00-foundation-decisions.md`)

Scope: every item addressed to 00 in `fable-issues-F1…F9.md`, 11 §18 #8–11, 14 §14, 15 §3 GAPs, plus decision-level contradictions still present across specs. 00 is now **revision 3**; the adjudication is also recorded in 00 Appendix B and the "### Round 2 changelog" at the end of 00.

Dispositions: **applied** (00 edited) · **already resolved** (rev 2 already covers it) · **rejected** (with why) · **forwarded** (not a 00 matter; owner named).

## 1. Disposition table

| Source | Item | Target in 00 | Disposition | Notes |
|---|---|---|---|---|
| F1 (own file) #1–#30 | round-1 cross-file issues from 00 | other specs | n/a | Addressed to other owners; not re-processed here. Only #11 is touched (see below). |
| F1 #11 | English share 25% (K1) / 15% (K2) recommended to 08 | — | **withdrawn** | 08 §6.8 adopted the 05/06/07 budget table (K1 20, K2 15); 00 never fixed weights (J1). Recorded in Appendix B so nobody re-applies the 25%. |
| F2 #27 | E5 Durable Objects for CO-3 still "proposed" | E5 | **already resolved** | Appendix A (01 §20 #1) accepted it in rev 2; E5 names `CoopRoom` and the fallback. |
| F2 #28 | D4: `fireLevel` unbounded, 10 flame steps per colour ring | D4 | **applied** | D4 now: never resets, `max: null`, monotonic, levels 1–10 = flame sizes, +1 ring per 10 levels (`flameStepsPerRing: 10`). Matches 01 §10.4 and 02 §7.6. |
| F2 #29 | G2/D4 region-unlock rule | G2 | **applied** | "(calendar chapter OR fire threshold, earliest) AND readiness; calendar guarantees, fire only opens early" (01 AC-REG-1). |
| F2 #23 | DO emits `boss_defeated` once; rejoin within idle window | E5 | **applied** (bonus) | Folded into the E5 `CoopRoom` sentence while adding the 30-min cap; detail stays with 11 §10.2. |
| F2 #22 | ⟂VERIFY re-sign preserves SQLite container | E1a | **applied** | Decision-level because the install path is 00's. E1a: must survive; ⟂VERIFY in the first re-sign cycle; fallback `preInstallSync` in `tools/mac/resign.sh`; parent rule "never delete the app". |
| F3 (all) | no 00 items | — | n/a | All addressed to 08/12/01/09/10/13. |
| F4 obs. 1 | Family garden one-liner in D1a | D1a | **applied** | Cosmetic layer beside the campfire, fed by parent-confirmed kindness acts, not a currency, not the campfire pot, never wilts. |
| F4 obs. 2 | Is a parent override of the small-parts rule acceptable? (06 P-06-8) | C13, OPD-27 | **applied (accept-modified)** | Allowed with conditions: default off, parent gate + step-up, explanation first, audit-logged, auto-expires when `hasChildUnder36m` turns false. OPD-27 lists it. |
| F4 obs. 3 | E9 "~44 phonemes" vs 05's 45 files | E9 | **applied** | E9 now: 45 phoneme files (43 phonemes incl. schwa + `ks`/`kw`), ×2 styles, 26 letter names, 45 keywords, ~60 nonsense words, 16 minimal pairs, songs. 05 §6 owns the list. |
| F4 obs. 4 (+ F4 #25/#28, F5 to 05/06/07 #23–27) | `family.calmMode` working name, ⟂VERIFY in 05/06/07/10 | **B8b** (new), J1 | **applied** | One switch `family.calmMode` in `family_prefs`, LWW, parent gate (10 SET-08), default off, never explained on a child screen; fixed effect list (no events/boss, T1 celebrations, calm music, `calm-suppress` nodes hidden, ladders paused, `EMO_CHECKIN` counters frozen, no noisy TPR, safe room only after OPD-19); quiet days reuse it. The four ⟂VERIFY markers on the name are resolved. 11 must add the field (forwarded). |
| F4 #25 / 06 §16.1 | `family.hasChildUnder36m` | C13 | **applied** | Derived nightly and on birthdate edit; `largePartsVariantId` fallback; 11 owns the field (J1 sentence added). |
| F4 #31 | 12 P3 "chips + short text ≤60 chars" vs C7a | C7a | **already resolved** | C7a (rev 2) already says no free text from any child, not even K1; 12 is the one that must change (F4 already forwarded to 12). Reinforced by the GAP-46 clause. |
| F5 (to 11 #5, to 02 #4, to 09 #2/#3, to 12 #1) | items citing 00 E5/H5/E7/B7/C15/B9/E6a | — | **already resolved** | They cite 00 as the binding source; nothing to change in 00. |
| F5 → 10 #2 / F6 → 10 #2 | 10 to say "runs inside the 15-minute job" | E5 | **forwarded** (10) | E5 wording now matches 11 §7.1, so 10 can simply cite E5. |
| F6 #1 / 11 §18 #8 | One `*/15 * * * *` trigger with four ordered steps | E5 | **applied** | "two Cron Triggers: nightly `10 0 * * *` + one 15-min job: alert queue → Friday `send_at` → batch polling → expiry nudges; window filters in code". |
| F6 #2 / 11 §18 #9 | `CoopRoom` 30-min hard cap | E5 | **applied** | "auto-close after 15 min idle or at a 30-min hard cap (C3)". |
| F6 #3 / 11 §18 #10 | `UIRequiresFullScreen: true` on iPad | A3 (+ E3) | **applied** | Safety + test-matrix rationale accepted. 14's `PQ-R-47` letterbox kept as defensive fallback only. ⟂VERIFY Stage Manager on iPadOS 27 kept. Two files still carry `false` (see §2). |
| F6 #4 / 11 §18 #11 | E6a references the key mechanism | E6a | **applied** | Family X25519 keypair at S1, private key only in guardian Keychains, QR/24-word transfer at S2, ciphertext-only on the server, browser key in `sessionStorage`, lost-key behaviour. 11 §9.8 owns the detail. |
| F6 #5 | B9(c) — nothing to decide | B9 | **already resolved** | 09 §3.11 implements the token and the ⟂VERIFY. |
| F7 #1 | OPD-35 (non-family names) and OPD-36 (monthly token quota) | OPD table | **applied** | Added with the defaults 13 §16.2 #45/#46 and 12 §11 #10 proposed; 13 can drop "מוצע/ממתין ל-00". |
| F7 #2 / 15 GAP-14e | OPD-08 "~$30" vs 12 §1.9 ($3/$40 soft, $50 console) | OPD-08 | **applied** | "console hard spend limit ≥ soft monthly cap, default $50; soft caps $3/day, $40/month (12 §1.9, parent-editable); 11 adopts `ai_daily_cap_cents = 300`". 11 §8.4 still says $1/$30 → forwarded (F6 already has it as F7 #5 / GAP-14e). |
| F7 #3 | E9 VO loudness −18 LUFS | E9 | **applied** | "VO −18 LUFS integrated ±1, true peak ≤ −1 dBTP; 09 §6.7 owns the mix table; no spec may state another VO number". 05 (−16 in §12.3 and AC5) and 11 §8 (−16) still differ → forwarded. |
| F7 #4 | I3 C4 wording | I3 | **applied** | "alpha 6.12.2026 (greybox-plus); VS complete ~17.1.2027 for K1+K2, K3 polished by end of January; 13 §7.1 re-estimate 30–38 agent-days". 13 §4 table row C4 and §5.2 heading still say 3.1.2027 → forwarded. |
| F8 → 00 (14 §14 #1) | E4 additions (Reassure, Maestro, Stryker, pinned baselines, layout shift = 0) | E4 | **applied** | Plus the vitest/jest-expo split from F8 (13 #2) and the viewport-ID rule (09 §13.2). |
| F8 → 00 (14 §14 #2) | E3 `expo-battery`, `expo-screen-orientation` | E3 | **applied** | Both verified in SDK 58 by 14. |
| F8 → 00 (14 §14 #3) | E1b draw calls 80 Low / 120 newer; 14 §2 budgets binding | E1b | **applied** | 11 §4 already uses 80/120. |
| F8 → 00 (14 §14 #4) | New E12 "spec 14 binding; perf only on Release/oldest device" | E12 | **applied (accept-modified)** | No new ID: E12 extended. The research playbooks stay binding (brief #14 is a parent request); spec 14 is their operational form and its `PQ-*` rules supersede the 25 rules where they differ; `pnpm verify` is the done-gate. |
| F9 GAP-46 | Sibling letter CO-4 moderation (cites 00 C7) | C7a | **applied** | "includes sibling letters and co-op messages: chips, numbers, pack pictures only; `coop_letter` doc". |
| F9 GAP-23b | Cron missed the Friday window | B7 | **applied** | Late-send rule: next 15-min run same day if ≥3 h before candle lighting, else Sunday 07:30 "(באיחור)"; never inside the pause window; also the Yom-Tov-Friday rule and `off`-mode rule from 11 §7.1. 10 owns the copy. |
| F9 GAP-44 | Cap day = 04:00–04:00 | C3 | **rejected (modified)** | One day definition everywhere: the existing `DayKey` (calendar day, family tz) used by C11/02/08; week = Sunday 00:00 → Saturday 23:59. Children are asleep at either boundary; a second day definition would split engine and cap logic. C3 now says so; 08/11 need no new boundary. |
| F9 GAP-03 / GAP-04 | K3 cap 10 vs 15 min; bedtimes | OPD-26 | **rejected (keep 10's values)** | 01 §15.1 and 10 §6.13 now both say K3 15 min; C3's 3–5-min episodes allow ≤3/day; OPD-26 states "15 min, parent range 5–15". Bedtimes already match (19:30/20:00/20:30). |
| F9 GAP-35 | "App in Viewer mode" impossible on free signing | H4 | **applied** | Phase-1 Viewer = web dashboard only; join by email one-time code; app-side Viewer with phase-2 distribution (OPD-25). |
| F9 GAP-36 | Co-admin cannot join without a sending domain | H2, OPD-05 | **applied** | Co-admin also uses a one-time bootstrap code (`pnpm ops bootstrap-guardian --role coadmin`, single-use, 24 h TTL, revoked after the first email code). 11 §5.2 must add the command → forwarded. |
| F9 GAP-49 | Export link only by email; no domain | OPD-05 | **applied** | OPD-05 default now says export links are shown in-app too. |
| F9 GAP-14c | Candle-lighting offsets | B7 | **already resolved** | B7 already says `@hebcal/core` city defaults; 01/10 align per F1 #1. |
| F9 GAP-62 | "Immediate push" impossible on free signing | E3/H6 | **already resolved** | E3 (local-only notifications) and H6 (push only in phase 2). |
| F9 (13 §3.2) | "Data preserved as long as the app is not deleted — to verify" | E1a | **applied** | Same change as F2 #22. |
| F9 other GAPs | all remaining GAPs | — | n/a | Addressed to 01/02/05/07/08/09/10/11/12/13; none names 00 as owner. |

## 2. New issues for other owners (blockers only — each is a contradiction with a decision 00 now binds)

| # | File / section | Problem | Fix |
|---|---|---|---|
| R1-1 | 11 §11.1 `app.config.ts` (`UIRequiresFullScreen: false`) vs 11 §0 / §4.9 / E-13 (`true`) | Internal contradiction in 11; 00 A3 now binds `true`. | Set `UIRequiresFullScreen: true` in the plist snippet; keep the ⟂VERIFY on Stage Manager. |
| R1-2 | 14 §2 (`PQ-R-47` ⟂VERIFY "`UIRequiresFullScreen: false` + resize events"), §13 ⟂VERIFY #6, `PQ-DEV-12` | Assume `false`; A3 binds `true`. | Reword `PQ-R-47` as the defensive fallback (letterbox if a resize ever arrives), drop ⟂VERIFY #6 or point it at 11 §0's Stage Manager check. |
| R1-3 | 05 §12.3 (line ~1687) and AC5 (−16 LUFS) | E9 binds VO −18 LUFS ±1 (09 §6.7, 12 §6.9, 14 `PQ-R-36`). | Change to −18 LUFS ±1 (short clips: calibrated gain per 12 §6.9). |
| R1-4 | 11 §8 TTS pipeline (line ~2514, "−16 LUFS") | Same. | −18 LUFS ±1. |
| R1-5 | 11 §8.4 / §6.2 (`ai_daily_cap_cents` default 100, workspace limit $30) | OPD-08 now binds 12 §1.9's values. | `ai_daily_cap_cents = 300`, monthly 4000, console $50 (already listed as F7 #5 / GAP-14e). |
| R1-6 | 11 schema (`family_prefs`) | Fields `family.calmMode` (B8b), `family.hasChildUnder36m` (C13, derived nightly + on birthdate edit), `consent.checkinNudge` (C8a) are not defined in 11. | Add to `family_prefs` / consent docs; `calmMode` LWW; `hasChildUnder36m` server-derived, read-only on devices. |
| R1-7 | 11 §5.2 / §15 ops | H2 now relies on `pnpm ops bootstrap-guardian --role coadmin` (single-use, 24 h TTL, revoked after first email code). | Add the command and the TTL/revocation rule (F1 #27 asked for the owner's code; extend to co-admin). |
| R1-8 | 13 §4 table row C4 and §5.2 heading ("סיום ב-3.1.2027") vs 13 §7.1 (~17.1.2027) | I3 now says alpha 6.12 / VS complete ~17.1.2027. | Align the table and heading; keep 3.1 only as the end of the calendar window if wanted, labelled. |
| R1-9 | 13 §16.2 #45/#46, §18.4 ("מוצע OPD-35/36 … ממתין ל-00") | OPD-35/36 now exist. | Drop "מוצע/ממתין"; cite the IDs. |
| R1-10 | 10 SET-08 (⟂VERIFY on the field name) and 05 §17 / 06 §16.4 / 07 §10 (same marker) | Name is now bound by B8b. | Remove the ⟂VERIFY markers; cite 00 B8b. |

## 3. Notes

- No other spec had a "### Round 2 changelog" at the time of this pass; the checks above are against the current files.
- Also fixed in 00 two pre-existing Markdown table breaks (unescaped `|` inside code spans in A6 and C15).
- Nothing committed.
