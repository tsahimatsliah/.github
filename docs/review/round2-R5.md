# Round 2 — R5 (owner of `08-assessment-engine-child-model.md`, `10-parent-area-dashboard.md`)

Inputs: `fable-issues-F1…F9.md`, spec 15 §3 GAPs, 00 **revision 3** (re-read mid-task: B7 late-send, B8b `family.calmMode`, C3 single `DayKey`, H2 co-admin bootstrap, H4 Viewer web-only, OPD-08 AI caps, OPD-26 K3 15 min / range 5–15). Both files carry a "### Round 2 changelog". Nothing committed.

Dispositions: **applied** (edited now) · **already resolved** (round 1 did it; verified against the current file) · **rejected** (with rationale; current text consistent with 00) · **forwarded** (not my file).

## A. Issues targeting 08

| Source | Item | Disposition | Where / note |
|---|---|---|---|
| F1 | #4 LevelId mapping / `packages/shared/levels.ts` | already resolved | 08 §1.1א |
| F1 | #9 `Band`, `bandOf` | already resolved | 08 §1.1, §1.2 |
| F1 | #10 RT rule by grade, not age | already resolved | 08 §4.4 `grade < 'g1'` |
| F1 | #11 English 25%/15% | already resolved (superseded by F4 #1 → 20/15) | 08 §6.8 |
| F1 | #13 03 §17.2 requests (extraGates, O.STEP, strands, flags, IDs) | already resolved | 08 §1.2, §3.3א, §4.1, §5.1, §9.2 |
| F1 | #16 `EMO_CHECKIN` under T4b | already resolved | 08 §9.2 |
| F2 | #8 `review_rung_passed`, `toEconomyState` | **applied** (and a bug fixed) | 08 §1.1א, §1.6 `maxPassedDays`, §5.4 `onReviewResult` emits per rung pass. Bug: `retained` was derived from `ladder[rung] ≥ 30`, which is true for K2 right after the 14-day review (ladder ends at 30). Now `retained` = 30-day review actually passed. Tests U-REV-1..2, AC #20 |
| F2 | #9 `rtMs: number \| null` below g1 | applied (modified) | 08 §1.4: `rtMs` stays on the wire (K2 needs it for guess exclusion and §3.8); `toEngineEvent` adds `rtMsScored = null` when `rtPolicy='ignore'`; 02 §2.6 reads `rtMsScored` |
| F2 | #9b consume `hint:lens` | applied | 08 §1.9 `BehaviorSignal{kind:'nikud_lens'}` (not a hint for §4.1); 04 §6.4 rule 4 reads it |
| F2 | #9c `assisted` on `Attempt`, w ×0.3 | already resolved | 08 §1.4, §4.1 |
| F3 | #1–#9 | already resolved | per F5 answer table; verified §1.2, §1.1 strands, §3.3א, §4.1, §10.5.3, §9.2, §6.2א, §1.7, §1.6 |
| F4 | #1–#6 | already resolved | §6.8 weights + S-8, `masteryProfile`/`kind`, `EMO_CHECKIN`/`SEL_LADDER_PAUSE`, §3.3א E.* map, closed `SubjectId`, `explorationShare` |
| F6 | #3 consumed fields of `item.answered` | already resolved | 08 §1.4 table |
| F6 | #4 `evaluateFlags` on device → encrypted doc | already resolved | 08 §9.3 |
| F6 | #5 export `LevelId`, `bandOf` | already resolved | 08 §1.1 |
| F9 | GAP-02 60-s grace vs 01 "finish the beat ≤1:30"; +10 min event | **applied** | 08 §6.10: beat ≤90 s (01) then `closing` ≤60 s (08), `capOvertimeMs`; `ParentInput.cap_bonus` (§1.9א) = wire `cap.bonus_granted`, 1/child/day, idempotent, never adds an episode; `cfg.cap` in §11; U-CAP-1/2 |
| F9 | GAP-06 idle → session reason / cap accounting | applied | 08 §6.1 (`focusedMinutesToday` = M-01 focused minutes, so idle/background/safe room never consume cap), §6.10 (`inactivity` at 5 min, `backgrounded` >10 min, resume from checkpoint); U-CAP-3 |
| F9 | GAP-13 `plan.mode='placement'` for the economy | applied | 08 §1.10 `EpisodePlan.mode` = `Session.mode`; `composeEpisode` sets it; 02 §2.7 reads `placement`/`benchmark`; U-PLN-1 |
| F9 | GAP-24 `endReason` enum | applied (08 is the owner; F9's list mapped) | 08 §1.10: added `backgrounded` (11 E-16) and `coop_ended`; alias table for natural/childQuit/switch/idle/bedtime/parentStop/crash/coopEnd/background/done/cap/parent; no `shabbat` (11 E-24 ends the episode `completed`); ESLint in AC #19 |
| F9 | GAP-32 one emotional rule | already resolved + verified | 08 §9.2 note added: rule only in 08, wording in 07 §10, 10 references; check-in cadence (07 §2.4: S every episode, G every other day) fits the "last 7 in 14 days" window |
| F9 | GAP-44 cap day boundary 04:00–04:00 | **rejected** → explicit rule applied | 08 §1.1, §6.10: one `DayKey` (00 C3 rev 3: calendar day in the family city, midnight, monotonic per device); 02 §2.1 / 11 §6.4 already bind `DayKey`; a second boundary would split cap-day from economy-day (breaks 10 AC-40). Bedtime lock makes the night gap moot. U-CAP-4. 08 §1.1's old "per Asia/Jerusalem" comment fixed |
| F9 | GAP-66 settings changed mid-episode | applied (reconciled with 01 §15.1 / F2 #10) | 08 §6.10: content settings from the next episode; caps/windows re-checked at every block boundary (= 01's segment boundary, ≤90 s); remote `bedtime_lock` immediate with 2-min warning; U-CAP-5 |
| F9 | GAP-72 test date < 3 days | applied | 08 §6.9: one `review` block (`near_test`), no spread, no AL-13; `cfg.classTopics`; U-TST-1 |
| task | placement model consistency with 01 (GAP-12) | applied | 08 §3.1 note: structure = 08 (00 H2 "placement expeditions"), child experience = 01's "no test feel" (§3.6); 01 asked to reword |
| F5 (own) | #7 clock skew `ctx.now` | already resolved | 08 §10.5.5(ה) |

## B. Issues targeting 10

| Source | Item | Disposition | Where / note |
|---|---|---|---|
| F1 | #3 Shabbat default `soft` + OPD-04 | already resolved | SET-01, ON-03, Q-02 |
| F1 | #6 chart direction → 00 B9 | already resolved | RTL-1 |
| F1 | #14/#15 (10 §18.2 → 11) | already resolved | §18.2 marked binding; 11 applied |
| F1 | #21 OPD IDs on parent questions | already resolved | §17 |
| F1 | #25 bedtimes / sign-off cite OPD-07/26 | applied | §6.13 cap + bedtime rows cite OPD-26; §11.1 unchanged (B7/OPD-07 already) |
| F2 | #10 "+10 min" control; cap edits at next segment boundary | applied | §6.13 "בונוס +10 דקות" row (gate, also mode L, 1/child/day, `cap.bonus_granted`, API-29, audit, never shown to child); §6.13 "שינוי באמצע פרק" row; RB-0; AC-54 |
| F2 | #11 guide mode cite 01 §12.6; counts only | applied | §6.13 row |
| F2 | #12 mode ids next to Hebrew labels, default `soft` | already resolved | SET-01 |
| F2 | #13 travel banner + "עיר נסיעה" | applied | SET-01: `family_prefs.travel {city, until}`, banner in W-HOME-08/FAM-04 on `deviceTz ≠ family.tz` >24 h, `tz_mismatch` |
| F2 | #14 unsynced-events counter from day 5 | applied | FAM-04 card: "סנכרון: N אירועים ממתינים" + "סנכרון עכשיו", emphasised from day 5 with the expiry banner |
| F2 | #15 M-14 K3 solo metric from `k3.parentPromptTimeout` | applied | §4.1 new event row, M-14 definition |
| F3 | #23 PA-PROBE grid, `paperGap`, teacher note | already resolved | W-ASM-06, M-16, API-16 |
| F3 | #24 new child settings | already resolved | §6.13 |
| F3 | #25 profile-swap card + PIN offer | already resolved | AL-15, W-LOG-05 |
| F4 | #23 `EMO_CHECKIN` → 07 wording | already resolved | §6.9 |
| F4 | #24 `NOTE_*` not flags | already resolved | AL-17 |
| F4 | #25 `family.calmMode`, `hasChildUnder36m` | already resolved; ⟂VERIFY dropped now | SET-08 cites 00 B8b and its fixed effect list |
| F4 | #26 "report an error" on fact cards | already resolved | §6.10 |
| F4 | #27 English card | already resolved | W-CH-11 |
| F6 | #1 `reports` table per 11 §6.2, `sent_at` | already resolved | §15.1 |
| F6 | #2 one 15-min trigger | already resolved | §15.3 |
| F6 | #3 SEC-02 cite 11 S4 | already resolved | SEC-02, AC-17 |
| F6 | #4 `focus.span` enum | already resolved | §4.1 M-01 |
| F6 | #5 approvals aliases | already resolved | API-18 |
| F6 | #6 field-level LWW toast | already resolved | §6.13 |
| F6 | #7 AL-11 triggers | already resolved (+ extended now) | AL-11 |
| F7 | #13 A7 `kid_flag` approval type | applied | §6.10 A7 row (manual, locked in SET-05, item + replacement, 3 flags/week on one `jobId` → game hidden), AC-56 |
| F7 | #14 AI cap settings + notices | applied | SET-07 ($3 / $40 editable in $1–10 / $10–100, override counter, kill switch), W-HOME-08 80% line, PA-INBOX 100% card, email after 3 days; AL-11; API-28 `PUT /admin/ai-caps`; AC-57 |
| F7 | #15 confirm-screen states | applied | §6.10 "מסך A1 — מצבים ומחרוזות חובה" (a)–(h): `exam_page` + Face ID downgrade, notation line, "[שם הוסר]", clone failure copy, >4 pages cost estimate, offline queue (≤4 jobs, 48 h), 24 h auto-delete, blur on inactive |
| F9 | GAP-03 K3 cap 10 vs 15 | **rejected** (00 OPD-26 rev 3: 10's values, K3 15, range 5–15; 01 already mirrors 15) | §6.13 cap row now states ranges K3 5–15 / K2 10–40 / K1 15–60 and that "2 episodes/day" binds first |
| F9 | GAP-04 bedtime semantics per child vs per device | applied | §6.13 bedtime row (a)–(e): no new episode at `bedtime − preBedtimeMin` (default 60, 15–60), `mini` when `< plannedEpisodeMin + 2`, effective lock = earliest(child, device); FAM-04; AC-55 |
| F9 | GAP-05 after the 30-min gate unlock | applied | FAM-04: picker, device-wide window, 2-min warning + land, `bedtime_lock`, audit |
| F9 | GAP-07 picture-PIN reset / attempt counting | applied | FAM-03 reset flow; `picturepin.miss` in §4.1; M-14 "ניסיונות גישה של אח" |
| F9 | GAP-22 L-mode cap edits | applied | RB-0 `device_settings.childOverrides[childId]`, effective = min, audit `device-local`; §6.13; AC-62 |
| F9 | GAP-23b cron missed window | applied (00 B7 rev 3 wording) | §11.1 |
| F9 | GAP-27 `pending_app_update` copy | applied | FAM-04 line + AL-11 `app_update_needed` (1/7 d) |
| F9 | GAP-28 sticker: which children | applied | W-ECO-06: K3 only; K2/K1 → appreciation letter |
| F9 | GAP-32 EMO_CHECKIN thresholds | already resolved | §6.9 |
| F9 | GAP-35 Viewer app mode impossible in phase 1 | applied (00 H4 rev 3) | RB-4, §7.1, Q-07, AC-63 |
| F9 | GAP-36 co-admin bootstrap without a domain | applied (00 H2 rev 3) | §7.1 co-admin step 3, FAM-02, ON-02, Q-05 |
| F9 | GAP-40 `storage_low` alert | applied | AL-11 (<500 MB, 1/7 d), FAM-04 storage line, AC-60 |
| F9 | GAP-42 "shop closed in quiet hours" | applied | W-ECO-06: child "app sleeps" windows + bedtime only |
| F9 | GAP-47 A7 | applied | see F7 #13 |
| F9 | GAP-49 export link in-app | applied | PRIV-03 |
| F9 | GAP-49b email retry | applied | §11.2: 3× over 2 h (+10/+30/+80 min), then inbox-only + SET-07 "לשלוח שוב" |
| F9 | GAP-52 gender change re-render | applied | FAM-03 copy; AC-25 scoped to core content; P2 `photo.rerender_variants`, `n` variant until then |
| F9 | GAP-56 family-delete grace | applied | PRIV-04: 410 `family_deleting`, offline-only, banner with "ביטול", 401 → setup; AC-58 |
| F9 | GAP-62 AL-08a immediate push impossible | applied | AL-08a phase 1 = poll (API-30) + one local notification on the parent device only (CH-1 rule); AC-59 |
| F9 | GAP-68 deep-link scheme | applied | §11.9 `olam://report/{id}` (+`approvals`, dev `test/fixture`) |
| F9 | GAP-72 test date < 3 days | applied | §6.13.1, AL-13 suppression, AC-61 |
| F9 | car mode toggle absent from PA-CSET | applied (P2 row) | §6.13 "מצב רכב", 0 cap minutes, no econ; AC-64 |
| F9 | GAP-19b safe-room time category (addressed to 09) | applied on 10's side | W-LOG-04 "רוגע" category, never counts to caps |
| F9 | GAP-25 "back to camp" = child_exit | already resolved + noted | §4.1 session.ended row, AL-04 |
| F9 | GAP-38 refund consequences (addressed to 02) | applied on 10's side | W-ECO-06 copy; forwarded to 02 |
| F9 | GAP-51 `child_remove` (addressed to 11) | applied on 10's side | PRIV-04 |
| F9 | GAP-39 travel (addressed to 11) | applied on 10's side | SET-01; field name forwarded to 11 |

## C. New issues for other owners (blockers only)

| # | Owner | Problem | Proposed fix |
|---|---|---|---|
| 1 | **01 (DD-31), 02 (§2.1, UC-16), 11 (§5.7 `tzOffsetMin`)** | 00 C3 rev 3 defines the single `DayKey` as the calendar day **in the family's city timezone**; 01 DD-31, 02 §2.1/UC-16 and 11 §5.7 define it from the **device** offset. 08 §1.1 now follows 00. | `tzOffsetMin` of each event = offset of `family.tz` (or `family_prefs.travel.city` while set, 10 SET-01) at event time, computed on the device; keep "monotonic per device". One sentence in each file. |
| 2 | **02 (§4.1)** | `retained` must key on `review_rung_passed{rungDays ≥ 30}` / `ReviewCard.maxPassedDays`, not on the current rung (08 §1.1א; K2's ladder ends at 30, so "rung = 30" is true right after the 14-day review). Also read `rtMsScored` (null below g1) for the rapid-guess rule (08 §1.4). | Update §4.1 wording and the zod invariant; `paysOncePerSkill` per `(skillId, rungDays)`. |
| 3 | **11 (§5.8, §7, §5.1, §4.1, §5.2)** | Needed by 10 round 2 (all listed in 10 §18.2): events `cap.bonus_granted`, `k3.parentPromptTimeout`, `picturepin.miss`, `parent.input`; `SessionEndReason` from 08 (adds `backgrounded`, `coop_ended`; drop `'switch'`/`'background'` literals); `device_settings.childOverrides[childId]`; `family_prefs.travel {city, until}`; `olam://report/{id}`, `olam://approvals`; `410 family_deleting` + `child_remove`; `PUT /admin/ai-caps`; `GET /photos/jobs/:id` with `estimatedReadyAt`; AL-11 `storage_low`/`app_update_needed`. | Add to the registry, schemas and routes; most already appear in F9's list for 11. |
| 4 | **01 (§14.2, §15.1)** | (a) "There is no explicit placement test" contradicts 00 H2 and 08 §3; (b) "60 minutes before bedtime" is now the parent-editable `preBedtimeMin` (10 §6.13, 15–60). | (a) "no *feeling* of a test; the expeditions of 08 §3 are story-wrapped levels with `mode:'placement'`" and allocate the week-1 slots; (b) cite 10 §6.13. |
| 5 | **09 (SCR-01/SCR-20)** | 10 §6.13 now defines the pre-bedtime picker tile state ("הגיבור כבר מתכונן לשינה") and the "camp asleep" view after the 30-min unlock; 09 has no such tile state. | Add the tile state to SCR-01 and reference 10 §6.13 (d). |

## D. Notes

- 08 §12.10ג gained U-CAP-1..6, U-REV-1..2, U-TST-1, U-PLN-1; 08 §13 gained acceptance criteria 19–20. 10 §16 gained AC-54..AC-64.
- Rejected proposals (GAP-03 K3 10 min; GAP-44 04:00 boundary) are documented in place with the 00 reference so the developer does not re-open them.
