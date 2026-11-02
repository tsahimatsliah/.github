# Fable review — issues for other files, from F7 (owner of 12 AI/voice pipelines, 13 delivery/QA)

Format: file · section · problem · proposed fix. Nothing here blocks: 12/13 already reference the proposed values.

## For F1 — `00-foundation-decisions.md`

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 1 | OPD table | Two parent inputs are not in OPD-01…34: (a) a list of "non-family" names (classmates, teachers, gan/school name) used by 12 V18 (photo blur) and V19 (pre-send PII guard); (b) the monthly token/compute quota of the agent account, which decides whether 3 parallel streams are affordable (13 §7.1 #3, §16.2 #46). | Add **OPD-35** (non-family names; default: empty list + generic PII detection; by C5) and **OPD-36** (monthly token quota; default: assume ≥100M; week 1). 13 §16.2/§18.4 already carry them as "מוצע". |
| 2 | OPD-08 | Says Anthropic monthly spend limit "~$30". 12 §1.9 sets the soft caps $3/day and $40/month and proposes the console hard limit at **$50** (a safety net above the soft cap, not below it). 11 §8.4 still says $1/day, $30/month (F9 GAP-14e). | Make OPD-08 say "console spend limit ≥ soft monthly cap, default $50"; 11 adopts `ai_daily_cap_cents = 300`. |
| 3 | E9 | VO loudness: 09 §6.7 and 14 `PQ-R-36` use **−18 LUFS**; 12 used −16 and has now been aligned to −18. 00 E9 does not state a number. | Optional: add "VO −18 LUFS ±1, peak ≤ −1 dBTP (09 owns the mix table)" to E9 so no spec drifts again. |
| 4 | I3 | 13 §7.1 now estimates the VS at 30–38 agent-days with "this is the bar" for K1+K2 by ~17.1.2027 rather than 3.1 (K3 polished by end of January). I3's "done 3.1.2027" is the alpha-plus date. | Reword C4 as "alpha 6.12.2026; VS complete ~17.1.2027". |

## For F6 — `11-technical-architecture.md`

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 5 | §8.4, §6.2 | AI caps $1/day, $30/month vs 12 §1.9 ($3/$40, override $5 ×3/month, 80%/100% state machine, `deferred_jobs` queue, upper-bound estimate before each call). 12 owns the values (00 J1). | Set `ai_daily_cap_cents = 300`, monthly 4000; implement the state machine hooks from 12 §1.9 (deferred jobs at 06:00, override counter, kill switch). |
| 6 | §8.3 | Photo resize 1600 px / "face check later" vs 12 §3.3 (2576 px long edge, JPEG q0.88, on-device face blur module `modules/vision-privacy`, EXIF orientation baked before strip, double EXIF strip server-side). | Adopt 12 §3.3; add the native module to the dev-build allow-list. |
| 7 | §7.9 / §6.2 | 12 adds: capture queue (`capture_queue` table on the parent device, `clientJobId` idempotency, 20 pages/60 MB, 7-day expiry), job states `uploaded → extracting → needs_confirm → building → ready | failed | deferred`, pHash duplicate check (30 days), `photo.rerender_variants` job, job auto-delete 24 h after no confirm. | Add the columns/states and the idempotency key to `photo_jobs`; add the two jobs to the job runner. |
| 8 | §7.6 / §4.4 | 12 §2.7.1 S0 handling marks a pack `revoked` in the manifest; devices must hide a revoked pack on next sync (≤5 min online) even if already downloaded. | Add `revoked` to the pack manifest schema and the loader rule. |
| 9 | §4.11 | 12 V22 requires one `vo.generic.<intent>` line per intent in the bundled core pack; please publish the closed `intent` enum in `packages/shared` so the validator and the player agree. | Export `VoIntent` enum. |
| 10 | §7.1 cron | 12 §5.1 submits the weekly Haiku batch at 06:05 Friday and falls back to a realtime Haiku call at 11:00 if the batch is late; the `*/15 6-12` job should trigger both. | Add the two steps to the Friday job. |

## For the owner of `09-ux-ui-design-system.md`

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 11 | §14 gates | 12 §7.1 adds gate **G4b** (rigged 3D hero model, if ADR-0005 picks 3D; 00 E10) and 13 §16.7 consolidates GATE-UX-1..10 with 12 G1–G7 and the 01/02/10 gates. | Reference G4b under GATE-UX-6 and link to 13 §16.7 as the single schedule. |
| 12 | §6.7 | 12 now uses −18 LUFS for VO per 09's mix table; 12 §6.9 keeps −23 LUFS music / ducking −8 dB (09 says −26 / −8). | Confirm −26 vs −23 for the summed music stems; 12 will adopt 09's number. |

## For the owner of `10-parent-area-dashboard.md`

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 13 | PA-APPR | 12 §1.8 `kid_flag` reason and §2.7.1 "3 flags in a week on one item family → whole family quarantined" need an approval type (F9 GAP-47 "A7") and a card "הילד סימן" without pressure. | Add A7 and the card. |
| 14 | PA-SET | 12 §1.9 lets the parent change `DAILY_USD_CAP` ($1–$10) and `MONTHLY_USD_CAP` ($10–$100) and shows "AI: 80% מהתקציב היומי" on PA-HOME; PA-INBOX gets the 100% notice (email after 3 consecutive days). | Add the two settings and the inbox/notice copy. |
| 15 | SCR-14 / review screen | 12 §3.5.6 adds mandatory items: mode `exam_page` (needs Face ID to downgrade), notation confirm line (03 §1.5), "[שם הוסר]" chips, "לא הצלחנו לשכפל פריט N — יוצג רק להורה", cost estimate for >4 pages, offline queue status, 24 h auto-delete notice. | Add to the capture/confirm flow states. |

## For the owner of `01-game-design.md`

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 16 | §7.1 / §13.2 | 12 §6.10.1 now produces the content 01 asked for (Pip CH-11 sheet, ~15 bottles, MX-E05/06/07 clips/pages/trees, 4-voice minimal pairs). 01 should list the asset IDs it expects so `LICENSES.json` and the manifests match. | Publish the asset ID list (`en/bottle_<n>`, `en/mx-e05/<word>`, …). |

## For the owner of `14-performance-quality-playbook.md`

| # | Section | Problem | Proposed fix |
|---|---|---|---|
| 17 | §2 PQ-B-24 | 12 V21 enforces "2D pack ≤ 8 MB on disk" in `content:validate` and splits larger packs by `skillId`; 14 should state that the split is the content pipeline's job, not the asset linter's. | One sentence. |
| 18 | §7 PQ-REL-01 #6 | Calendar guard must also block on 00 B7a quiet days (Yom HaShoah, Yom HaZikaron, Tisha B'Av, Yom Kippur), not only Shabbat/holidays. | Add B7a to `calendar-guard.mjs` inputs. |

## Observations (no action)

- 06 "132 skills" and 07 "72 skills" could not be verified by grep in this session (the counts are not stated in those files' summaries); 13 §5.6 marks them ⟂VERIFY for P0-E1-S6 (`13a-id-map.md`).
- 12 §13 "Proposed change to 00" items 1–6 were all adjudicated in 00 revision 2 (accept / accept-modified); the section is left as the record.
