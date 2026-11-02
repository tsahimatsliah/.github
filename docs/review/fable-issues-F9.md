# Fable review — issues found by F9 (use cases, journeys & edge cases) in files F9 does not own

> Source: `docs/spec/15-use-cases-journeys-edge-cases.md` §3 (gap list) and §2 (journeys). Each item: **file · section · problem · proposed fix · journey that exposed it**. GAP IDs match §3 of spec 15 (non-sequential by design). Owning reviewers decide; the proposed fix is a default the developer can implement if no decision arrives.

---

## 01-game-design.md

| GAP | Section | Problem | Proposed fix | UC |
|---|---|---|---|---|
| GAP-01 | §14.2 | FTUE per child has no "already ran" state; a second/restored device replays the 60-s opening. | Add `ftueCompletedAt` to the `child_resume` doc (11 §4.16); FTUE only when null. | UC-012, UC-081 |
| GAP-02 | §15.1, DD-09 | Overtime ≤1:30 vs. 08 §6.10 60-s grace; "+10 min bonus" has no event/API/audit. | Beat finishes ≤90 s then lands; 08's 60 s applies inside `closing` only. Add `cap.bonus_granted` event (gate-only, audited, ≤1/child/day). | UC-040 |
| GAP-03 | §15.1 | K3 cap 2 episodes / 10 min conflicts with 10 §6.13 (15 min). | Keep 10 min default, parent range 5–15; ask 10 to align. | UC-011 |
| GAP-04 | §15.1 | Bedtime defaults (K2 19:30) differ from 10 §6.13 (K2 20:00). | Adopt 10's defaults (K3 19:30 · K2 20:00 · K1 20:30). | UC-040 |
| GAP-08 | §16.2 | "Taps = 1 + effort marks" is a proposal; 02 §6.5 tier-based taps is canonical; keep only the "effort badge" idea if 02 adopts. | Reference 02 §6.1/6.5 explicitly; drop the tap formula or mark it `⟲ADOPTED-IF-02`. | UC-017 |
| GAP-12 | §14.2–14.3 | "No explicit placement test" contradicts 08 §3 (explicit story-wrapped expeditions) and 10 ON-12. | State that expeditions exist (08 structure, 08 §3.6 UI rules give the "no test feel"); allocate expedition slots in the week-1 table (K2 Sun–Tue, K1 Sun–Wed). | UC-009 |
| GAP-13 | §6.6 level schema | No `mode` field to tell economy/collection that a level is a placement/boss/event level. | Add `mode: 'placement'|'normal'|'review'|'boss'|'event'` to the level schema. | UC-009 |
| GAP-14b | §9.3 | Weekly boss "K1: 2 gems" vs. 02 §6.2 silver chest + 1 gem. | 02 wins (review brief item 1): silver + 1 gem. | UC-029 |
| GAP-14c | §15.4 | Candle-lighting offset Tel Aviv 20 min vs. 10 SET-01 (18 min, Haifa 30, Jerusalem 40). | Use `@hebcal/core` city defaults + manual override. | UC-045 |
| GAP-14d | §15.4 | Shabbat-soft starting mid-episode: are the episode's receipts paid? | Episode started before lighting is paid in full; freeze applies to sessions started after lighting. | UC-045 |
| GAP-19 | §15.1 | "Audio-only car mode" exists only here (no screen, launch path, cap accounting in 09/11). | P2: parent-gate toggle "מצב רכב" → playlist; no econ events; 0 cap minutes; one "עצור" button. Hand to 09/11. | UC-039 |
| GAP-25 | §15.1 | "Back to camp" from pause: does it count as `childQuit` for AL-04? | Yes: `session.ended{reason:'childQuit', secondsSinceLastError}`. | UC-016 |
| GAP-29 | DD-20 | iPhone hub landscape contradicts 09 §8.1 (hub portrait-first). | 09 wins; update DD-20. | UC-013 |
| GAP-31 | §5.3 (K2 table) | Check-in "every other day" contradicts 07 §2.4 (every episode, ≤20 s, skippable). | 07 wins; update the K2 table. | UC-035 |
| GAP-33 | §12.5 | "Patience heart" / VIP surprise lacks a detection rule and event. | `turn.yielded` when `session.ended{reason:'switch'}` at <80 % of planned length and another child starts ≤120 s later on the same device; ≤1/child/day. | UC-034 |
| GAP-43 | §15.1 vs 09 SCR-07 | K1 after goodnight may open collection room/music hut; 09 sends everyone to the picker. | K1 tile after wind-down opens a "camp asleep" view (collection + music only, no session/econ); K2/K3 via gate. Align with 09. | UC-039 |
| GAP-46 | §12.1 CO-4 | K1-authored letter to K2 needs moderation if free text. | Structured authoring only (chips/numbers/pictures); `coop_letter` doc. | UC-033 |
| GAP-50 | §5.1, §15.1 | Printing (season certificate, offline mission) has no library/flow. | `expo-print` → PDF → share sheet/AirPrint; all printables A4 RTL (coordinate with 11 §2.3). | UC-039, UC-043 |
| GAP-53 | §10.1 | Hub has 3 neighborhoods; 11 schema supports N children. | v1: FAM-03 blocks a 4th child; v2 item. | UC-003 |
| GAP-63 | §9.3, DD-06 | Finale with an absent child; boss day on a Yom Tov/quiet day. | `bossHP` counts only children with ≥1 episode this week; finale completes with any present subset; if Thursday is Yom Tov/QD the window moves to the previous weekday evening. | UC-029 |
| GAP-67 | §14.1, §19 | Parent voice clips: storage, sync, format unspecified (also 12 §6.11, 11). | m4a ≤10 s; R2 private `voice/<familyId>/<guardianId>/<clipId>.m4a` + device cache; shipped as a family pack via the catalog; re-record replaces. | UC-091 |

## 02-economy-progression.md

| GAP | Section | Problem | Proposed fix | UC |
|---|---|---|---|---|
| GAP-09 | §7.7 | Neighborhood layout edited offline on two devices (same tile) — no merge rule. | `neighborhood_layout/<childId>` doc LWW; displaced item returns to inventory with a mailbox note. | UC-019 |
| GAP-10 | §7.5 | Savings bonus on the week's minimum balance can go negative after an overdraft merge (11 §6.5). | Bonus base = `max(0, minBalance)`; overdraft weeks pay no bonus. | UC-018 |
| GAP-13 | §2.7 | Placement/benchmark "never earns" has no flag from the level/plan to key on. | Consume `level.mode` / `plan.mode` (`placement`, `benchmark`) — see 01/08. | UC-009 |
| GAP-37 | §6.2 | Wood chest per confirmed real-world mission: no daily cap, no child-side surfacing. | ≤1 mission chest/child/day; surfaced as mailbox letter + shelf chest next episode. | UC-039 |
| GAP-38 | §7.3 | Parent refund of an equipped/placed item: UI consequence undefined. | Revert to previous outfit / remove from grid; coins back; neutral mailbox note; no "taken away" animation. | UC-018, UC-064 |
| GAP-42 | §7 (with 10 W-ECO-06) | "Shop closed in quiet hours": which quiet hours? | Child "app sleeps" windows and bedtime only, never parent Push quiet hours. | UC-018 |
| GAP-21 | §7.6 (with 11 §10.2) | Offline co-op contributions vs. DO-authoritative campfire level. | Level-ups only from the DO summary; offline contributions add logs/light to the next meter. | UC-031 |

## 05-curriculum-english.md

| GAP | Section | Problem | Proposed fix | UC |
|---|---|---|---|---|
| GAP-30 | §10.4 | Hebrew cap/bedtime warning (ניצי) would fire inside an English island (violates 01 AC-MX-3 / OPOL). | Defer the warning to island exit; if the cap ends inside, Sunny's "Bye-bye" + exit jingle, then the Hebrew wind-down. | UC-025 |
| — | §13 PP-3 | Mic permission denied → behavior of "record and compare" not defined. | Fall back to "Let's say it together" without recording; no data recorded. | UC-026 |

## 07-curriculum-future-skills.md

| GAP | Section | Problem | Proposed fix | UC |
|---|---|---|---|---|
| GAP-32 | §10 RF-01/RF-02 | Overlaps with 10's proposed `EMO_CHECKIN` (3 of 5 days) and 08 `EMO_DISTRESS`. | Single rule in 08 §9.2 `EMO_CHECKIN` using RF-01 thresholds (≥5/7 in 14 d, ≥5 days), consent T4b; 07 references it. | UC-035, UC-062 |
| GAP-31 | §2.4 | (For awareness) 01 §5.3 says check-in every other day; 07 should state it is canonical (every episode, skippable). | Add "canonical; 01 follows" note. | UC-035 |

## 08-assessment-engine-child-model.md

| GAP | Section | Problem | Proposed fix | UC |
|---|---|---|---|---|
| GAP-02 | §6.10 | 60-s grace vs. 01 "finish the beat ≤1:30". | Grace applies inside `closing`; beat completion rule from 01 applies before closing. | UC-040 |
| GAP-06 | §6.10 | Idle 5 min → picker (11 §4.16): session end reason, cap accounting, checkpoint not defined. | `session.ended{reason:'idle'}`; idle minutes excluded from caps; level resumes from checkpoint. | UC-010 |
| GAP-13 | §3 | Expedition levels must not pay; define `plan.mode='placement'` consumed by economy. | Add `mode` to `EpisodePlan` (§1.10). | UC-009 |
| GAP-24 | §1.10 Session | `endReason` values used across specs are inconsistent. | Adopt the shared enum `natural|cap|childQuit|switch|idle|bedtime|parentStop|crash|coopEnd|shabbat` (11 `events.ts`). | UC-039 |
| GAP-32 | §9.2 | Emotional rules split across 07/10/08. | Own one `EMO_CHECKIN` rule (see 07 row). | UC-062 |
| GAP-44 | §6.10 | Daily cap "day" boundary undefined. | Day = 04:00–04:00 family timezone (caps, surprises); week stays Sunday 00:00. | UC-040 |
| GAP-66 | §6.3 | Settings changed while a child is mid-episode. | Caps/bedtime immediate; content settings from the next episode. | UC-066 |
| GAP-72 | §6.9 | Test date < 3 days away. | One review block next session; no AL-13; copy "המבחן קרוב, חזרה קצרה היום". | UC-053 |

## 09-ux-ui-design-system.md

| GAP | Section | Problem | Proposed fix | UC |
|---|---|---|---|---|
| GAP-08 | §5.3 chest choreography | "Auto after 1.5 s without tap" contradicts 02 §6.5 (hand at 4 s; auto-open only via a11y). | Align with 02. | UC-017 |
| GAP-16 | §8.3 F4 | Photo-game surfacing wording differs from 12 §3.1. | Map node (09) labelled per 12; K2 gets it as the next episode's review slot. | UC-054 |
| GAP-19 | — | No screen for 01's audio-only car mode. | Add SCR-21 "מצב רכב" (P2) or reject in 01. | UC-039 |
| GAP-19b | SCR-16 | Safe-room time vs. caps/PA-LOG category. | Never counts toward caps; `session.started{mode:'safeRoom'}` → category "רוגע". | UC-041 |
| GAP-43 | SCR-07 | WindDownCard "back = picker" conflicts with 01 K1 view-only camp. | Add the K1 "camp asleep" view state. | UC-039 |
| GAP-48 | §12 | `testID` on all interactive elements is needed for Maestro (see 13). | Add to the polish checklist. | all `[MAE]` |
| GAP-60 | §10 | Guided Access exit passcode may be known to kids. | Parent guide: separate Guided Access passcode. | UC-011 |
| — | §8.4 SCR-13 | K3 joining a co-op lobby: parent-present gate not mentioned. | Lobby requests the gate when K3 is selected. | UC-030 |

## 10-parent-area-dashboard.md

| GAP | Section | Problem | Proposed fix | UC |
|---|---|---|---|---|
| GAP-03 | §6.13 | K3 cap 15 min vs. 01 10 min. | 10 min default (00 C3), range 5–15. | UC-011 |
| GAP-04 | §6.13 | "No new session 60 min before bedtime" lacks exact semantics; per-child bedtime vs. 11 per-device schedule. | Rule: start only if `bedtime − now ≥ plannedEpisodeMin + 2`; effective lock = earliest of child and device bedtime. | UC-040 |
| GAP-05 | FAM-04 | After the 30-min gate unlock, where does the device go? | Picker; window device-wide; at its end 2-min warning + land. | UC-068 |
| GAP-07 | §6.13 picture-PIN | Reset flow and attempt counting undefined. | See 11 row; add "sibling access attempts" to M-14. | UC-010 |
| GAP-22 | RB-0 | L-mode cap edits "on this device only" vs. per-child caps across devices. | `device_settings.childOverrides[childId].dailyCapMin`; effective = min. | UC-050 |
| GAP-23b | §11.1 | Cron missed the 12:00–14:00 window. | Send at next run same day if ≥3 h before lighting, else Sunday 07:30 "(באיחור)". | UC-079 |
| GAP-27 | FAM-04 | No copy for `pending_app_update` packs / expired signing. | Health line + AL-11 variant `app_update_needed` (≤1/7 days). | UC-077 |
| GAP-28 | W-ECO-06 | "תן מדבקה" without limit — for which children? | K3 only; K2/K1 use the appreciation letter. | UC-064 |
| GAP-32 | §6.9 | Proposed `EMO_CHECKIN` thresholds differ from 07 RF-01. | Defer to one rule in 08 with 07 thresholds. | UC-062 |
| GAP-35 | §7.1 Viewer | "App in Viewer mode" impossible on free signing. | Phase 1 viewers web-only. | UC-008 |
| GAP-40 | §10.1 AL-11 | No `storage_low` alert. | Add variant (<500 MB free). | UC-082 |
| GAP-42 | W-ECO-06 | "Shop closed in quiet hours" ambiguous. | Child windows + bedtime only. | UC-018 |
| GAP-47 | §6.10 | `kid_flag` quarantine items (12 §1.8) not in the approval types. | Add A7 "סימון של ילד". | UC-038 |
| GAP-49 | PRIV-03 | Export link only by email; no domain in phase 1. | Return the link in-app too. | UC-071 |
| GAP-49b | §11.2 | Email retry policy on Resend failure. | 3× backoff over 2 h, then inbox-only + SET-07 note. | UC-079 |
| GAP-52 | AC-25 | "Gender change needs no download" is false for P2 packs (12 §3.13 renders selected variants only). | Scope AC-25 to core content; P2 variants re-rendered and downloaded silently. | UC-066 |
| GAP-56 | PRIV-04 | Family delete grace: device behavior. | Sync 410 `family_deleting`; offline-only play + banner; after purge 401 → setup. | UC-072 |
| GAP-62 | AL-08a | "Immediate Push" impossible on free signing. | In-app poll + local notification when backgrounded; Push on path 2. | UC-054 |
| GAP-68 | §11.9 | Deep-link scheme `familygame://` vs 11 `olam://`. | Use `olam://report/{id}`. | UC-059 |
| GAP-72 | §6.13.1 | Test date < 3 days. | See 08 row. | UC-053 |
| — | §6.13 | Audio-only car mode (01) absent from PA-CSET. | Add toggle if 01 keeps it (P2). | UC-039 |

## 11-technical-architecture.md

| GAP | Section | Problem | Proposed fix | UC |
|---|---|---|---|---|
| GAP-01 | §4.16 | Add `ftueCompletedAt` to `child_resume`. | As stated. | UC-012 |
| GAP-02 | §5.8, §7 | No event/route for the parent "+10 min" bonus. | `cap.bonus_granted` event (cls `admin`), audited. | UC-040 |
| GAP-04 | §5.5 | Per-device bedtime only; 10 has per-child bedtime. | Support both docs; effective lock = earliest. | UC-068 |
| GAP-06 | §4.16 | Idle → picker accounting. | `session.ended{reason:'idle'}`; idle excluded from caps. | UC-010 |
| GAP-07 | §4.16 | Picture-PIN after 3 misses: 4th tap/reset flow undefined. | Grid hidden 60 s (no countdown); `picturepin.miss` telemetry; parent reset → "בחרי 3 תמונות" once. | UC-010 |
| GAP-09 | §6.1 docs | Add `neighborhood_layout` doc kind. | As stated. | UC-019 |
| GAP-11b | §4.16 | K3 launch rule is binary; DD-18 needs "co-play-first". | `child_settings.launchMode ∈ {lap, coplayFirst, solo}`. | UC-049 |
| GAP-14e | §8.4, §6.2 | AI daily cap $1/day & $30/month vs. 12 §1.9 ($3 / $40). | Adopt 12's numbers; `ai_daily_cap_cents` default 300. | UC-078 |
| GAP-15 | §8.3, §9.4 | Photo resize 1600 px and "face check later" vs. 12 §3.3 (2576 px, on-device blur module). | Adopt 12. | UC-054 |
| GAP-15b | §4.8 | Offline photo capture: queue or block? | Capture/crop/blur offline; upload queued (≤4 jobs, 48-h expiry). | UC-057 |
| GAP-21 | §10.2 | Solo-co-op merge vs. DO summary authority. | Level-ups only from DO summary. | UC-031 |
| GAP-22 | §5.4 (L) | L-mode cap edits need a storage location. | `device_settings.childOverrides`. | UC-050 |
| GAP-24 | §5.8 | `session.ended.reason` untyped. | Enum in `packages/shared/events.ts`. | UC-039 |
| GAP-26 | §3.3, §11.3 | Updates apply only at cold start; shared iPads rarely cold-start. | Reload boundary: update downloaded + backgrounded ≥30 min + no active session → `reloadAsync()` on foreground before the picker. | UC-077 |
| GAP-27 | §4.4 | `pending_app_update` has no parent-facing surface. | Expose in FAM-04 health + AL-11 variant. | UC-077 |
| GAP-36 | §5.2 | Co-admin cannot join without a sending domain (email code). | `pnpm ops bootstrap-guardian --role coadmin` one-time code. | UC-007 |
| GAP-39 | §6.4 | Timezone/travel: device tz ≠ family city. | Family-city calendar; `family_prefs.travelMode`; `tz_mismatch` telemetry. | UC-090 |
| GAP-40 | §4.4, §4.7 | No storage-low handling for new packs; audio cache unbounded. | TTS cache LRU ≤400 MB; "האזור עוד נטען" gate copy; AL-11 `storage_low`. | UC-082 |
| GAP-41 | §9.7 | Data at rest on a lost offline device. | `NSFileProtectionCompleteUntilFirstUserAuthentication` for `olam.db`/packs; document PII at rest. | UC-083 |
| GAP-44 | §4.16 | Cap day boundary. | 04:00–04:00 family tz. | UC-040 |
| GAP-51 | §5.1 DeviceCommand | Child soft-delete needs a device command; `resync` undefined. | Add `child_remove {childId}`; define `resync`. | UC-072 |
| GAP-52 | — | Gender change requires P2 variant re-render job. | Worker job + silent download. | UC-066 |
| GAP-56 | §7.3 | Family delete grace behavior for devices. | 410 `family_deleting` → offline-only. | UC-072 |
| GAP-57 | §5.3 S3 | Claiming a pairing on a device holding another family's data. | Compare local `familyId`; require wipe confirm on mismatch. | UC-006 |
| GAP-57b | §5.3 S5 | `unpair` revokes immediately → unsynced events lost. | `unpair` = push then revoke; `{force:true}` for lost devices. | UC-069 |
| GAP-67 | — | Parent voice clip storage/distribution. | See 01 row. | UC-091 |
| GAP-68 | §4.1 | Deep links limited to `olam://pair`; 10 needs report links. | Allow `report/{id}`, `approvals`, dev-only `test/fixture`. | UC-059 |
| GAP-48 | §12.1 | No on-device E2E layer. | Maestro via the Mac (see 13 row); dev-only fixture deep link + injected clock. | all `[MAE]` |
| — | §7.3 guardian remove | Semantics for devices a removed co-admin paired are not stated. | Devices belong to the family; they stay; the guardian's sessions are revoked. | UC-007 |

## 12-ai-content-voice-pipelines.md

| GAP | Section | Problem | Proposed fix | UC |
|---|---|---|---|---|
| GAP-16 | §3.1 step 13 | Child-side surfacing wording differs from 09 F4. | Reference 09's map node; keep the label. | UC-054 |
| GAP-47 | §1.8 | `kid_flag` needs a PA-APPR type (10). | Add A7 in 10; keep reason code here. | UC-038 |
| GAP-52 | §3.13 | TTS rendered only for selected children's variants → gender change needs a re-render job. | Add `photo.rerender_variants` job. | UC-066 |
| — | §3.9 | Clone generator failure for a homework item has no parent copy. | "לא הצלחנו לשכפל פריט N — יוצג רק להורה". | UC-055 |

## 13-delivery-roadmap-qa.md

| GAP | Section | Problem | Proposed fix | UC |
|---|---|---|---|---|
| GAP-48 | §10.1 | QA layers have no automated on-device E2E (Maestro); only Playwright web + manual checklist. | Add a "device E2E" layer: Maestro flows (`tools/e2e/maestro/*.yaml`) run by the agent over Remote Control on the simulator and one real iPad per native build and weekly; depends on `testID`s (09) and a dev-only fixture deep link (11). Map 11 §12.4 rows 1, 10, 12, 13 to Maestro flows. | all `[MAE]` |
| — | §10.3 | Checklist lacks: expiry banner at ≤1 day; English-island cap handling; picture-PIN reset; family-delete grace. | Add rows from spec 15 UC-076, UC-025, UC-010, UC-072. | — |
| — | §3.2 | "Data preserved as long as the app is not deleted (to verify)" — spec 15 relies on it (UC-076/E2). | Keep the `⟂VERIFY` and test it in the first re-sign cycle. | UC-076 |
