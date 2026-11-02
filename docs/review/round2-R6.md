# Round 2 — R6 (owner of 09 UX/UI, 11 technical architecture, 14 performance & quality)

Date 2026-11-02. Inputs: `ROUND2-BRIEF.md`, 00 **revision 3**, every `fable-issues-F1…F9.md`, 15 §3 GAPs, the
coordinator forwards from R1 §2, R2 §3, R3, R4 §4, R5 §C and 10 §18.2. Every item targeting 09/11/14 is listed below.
"applied" = edited in the owned file this round; "already resolved" = present before this round (usually round 1);
"rejected" carries the rationale; "forwarded" = needs another owner (also listed in §2). Each file has a
"### Round 2 changelog" under its "## Fable review changelog". Nothing committed.

## 1. Dispositions

### 1.1 Items for 11 — technical architecture

| Source | Item | Disposition |
|---|---|---|
| F1 #8 | allowlist → 00 E13 | already resolved (§15.1 #11) |
| F1 #14 | crons / `flags[]` / `focus.span` / tables | already resolved (§7.1, §7.10, §5.8, §6.2) |
| F1 #15 | `disableDeviceFallback:true` on shared devices | already resolved (S4) |
| F1 #19 | game-controller deferred module | already resolved (§2.3) |
| F1 #27 | bootstrap code TTL/revocation | already resolved (§5.2) |
| F1 #28 | `expo-device` | already resolved (§2.3) |
| F2 #19 | 02 economy events (`review_rung_passed`, `chest_opened`, `receipt_shown`, `episode_closed`, `season_tier_reached`, `brain_token_deposited`, `tool_state_changed`, `campfire_lantern_lit`) | applied (§5.8 row "review.rung_passed / …"; `chest.opened` existed) |
| F2 #20 | `DayKey` monotonic per device, `packages/shared/dayKey.ts`, `last_day_key` | applied (§6.1, §6.4) — **with 00 C3 rev 3**: the day is the family/travel-city calendar day, `tzOffsetMin` = that city's offset (§4.8, §5.7) |
| F2 #21 | `lastServerTime` to the engine; sync at episode end and app background | applied (§4.8) |
| F2 #22 | ⟂VERIFY re-sign keeps SQLite; `preInstallSync` | applied (§3.3: ⟂VERIFY + `--pre-install-sync` fallback + unsynced counter in `GET /devices`) |
| F2 #23 | DO emits `boss_defeated{participants}` once; 15-min rejoin | applied (§10.2; 00 E5 rev 3 binds the 15 min / 30 min) |
| F2 #24 | `asset.missing{assetId, kind}` | applied (§4.11, §5.8) |
| F4 #28 | `family.calmMode`, `family.hasChildUnder36m`, `consent.checkinNudge` | applied (§6.2 `families.prefs_json` comment, §7.3 PATCH /family; `hasChildUnder36m` server-derived nightly + on birthdate edit, override expiry in §7.1 step 4) — also R1-6, R4-1 |
| F4 #29 | `fs_local_counters` excluded from log/sync; only `rf_raised` syncs | applied (§5.7, §5.9, §6.1 table, §16 #17); `rf_raised` is a field of the encrypted `flags` doc, not a plain event — forwarded to 07 as wording |
| F4 #30 | `POST /fs/t2/request` chips-only; `POST /content/fact-report` | applied (§7.7, §7.6, `fact.reported` in §5.8) — also R4-1 |
| F5 #1 | missing events (`screen.enter/exit`, `app.background/foreground`, `behavior.signal`, `skill.level_reverted`, `benchmark.result`, `placement.finished`, `creation.saved`, `launch.blocked`, `parent.tonight_done`) | applied (§5.8) |
| F5 #2 | `item.answered` v2 fields | applied (§5.8, v1 upcast via 08's safe defaults) |
| F5 #3 | `session.ended` = 08 `SessionEndReason` + `plannedMinutes/blocksCompleted/focusedMs` | applied (§5.8, §4.16) with 08's **current** enum incl. `backgrounded`, `coop_ended` (R5 #3, R2) |
| F5 #4 | `checkin.mood` → `behavior.signal{emotion_checkin}`; `local_only` retention = not persisted + 14-day counters | applied (§5.7, §5.8, §5.9) |
| F5 #5 | crons/payload/S4 | already resolved |
| F5 #6 | `parent.input` event + API-13 reattribute route + audit | applied (§5.8, §7.10a API-13) |
| F5 #7 | clock skew: `clockSkewMs` on `device_self`, `/sync` meta, `GET /devices`, corrected `ctx.now` | applied (§6.1, §6.2, §6.4, §6.5, §7.4, §16 #21) |
| F5 #8 | `engine_checkpoint` table, `model_epoch` | applied (§6.1, §6.2) |
| F5 #9 | Cloudflare Access optional sentence | applied (§5.10) |
| F5 #10 | export: `engine-state.v1`, `FlagState` opt-in, never `emotion_checkin` | applied (§7.12) |
| F7 #5 / GAP-14e / R1-5 | AI caps $3/$40, state machine hooks | applied (§4.11, §6.2, §7.1, §7.7, §8.4, §14, §15.1 #4, §16 #8); console limit ≥ soft cap default $50 per 00 OPD-08 rev 3, stored as `consoleCapCents` |
| F7 #6 / GAP-15 | photo 2,576 px q0.88, on-device blur module, orientation bake, double EXIF strip | applied (§2.3, §8.3, §9.4) |
| F7 #7 / GAP-15b / GAP-52 | `photo_jobs` states, `capture_queue`, `clientJobId`, pHash, 24 h auto-delete, `photo.rerender_variants` | applied (§6.1, §6.2, §7.9; `/photos` + `/ai/photo/*` removed). F9's 4 jobs / 48 h → 12's 20 pages / 60 MB / 7 d (12 owns) |
| F7 #8 | `revoked` packs | applied (§4.4 #7, §6.2, §7.6, E-25, §16 #20) |
| F7 #9 | `VoIntent` enum | applied (§4.11) |
| F7 #10 | Friday batch 06:05 / fallback 11:00 | applied (§7.1 step 2) |
| F8 #1–#18 | fonts, deps, `remove()`, Hermes floor, Reanimated flags, nikud tokens, memory tiers, viewports, test table, CI jobs, `child_resume_bak`, canvas boundary, observability, Split View policy, spike gate, `PQ-DEV`, lint list | already resolved (round 1) **except** #15's consequence: `app.config.ts` still had `UIRequiresFullScreen: false` → applied `true` (§11.1; R1-1) |
| F9 GAP-01 | `ftueCompletedAt` in `child_resume` | applied (§4.16) + `ftue.completed` event (R2) |
| F9 GAP-02 | `cap.bonus_granted` + route | applied (§4.16, §5.8, §7.3 `POST /children/:id/cap-bonus` = 10 API-29, payload `{minutes, dayKey}`, mode L) |
| F9 GAP-04 | per-child + per-device bedtime, earliest wins | applied (§5.5) with 10 §6.13's `preBedtimeMin` semantics (R5) |
| F9 GAP-06 | idle → picker accounting | applied (§4.16: `inactivity`, caps count `focusedMs`) |
| F9 GAP-07 | picture-PIN miss/reset | applied (§4.16, `picturepin.miss`) |
| F9 GAP-09 | `neighborhood_layout` doc | applied (§6.1, §6.5 — field-level LWW **per tile**, per R2/02) |
| F9 GAP-11b | `child_settings.launchMode` | applied (§4.16, §5.8) |
| F9 GAP-21 | level-ups only from the DO summary | applied (§10.2 `coop.room_ended`) |
| F9 GAP-22 | `device_settings.childOverrides` | applied (§4.16) |
| F9 GAP-24 | `session.ended.reason` enum | applied (= F5 #3) |
| F9 GAP-26 | OTA apply for shared iPads | applied (§3.3 moments 2b episode boundary / profile switch and 2c foreground ≥ 30 min; `UpdateGate` sole caller) |
| F9 GAP-27 | `pending_app_update` parent surface | applied (§4.4 #6, §7.1 step 4, `GET /devices`) |
| F9 GAP-36 | co-admin bootstrap code | applied (§5.2, §7.12; 00 H2 rev 3) |
| F9 GAP-39 | travel timezone | applied (§4.8; `family_prefs.travel {city, until}`, `app.tz_mismatch`) — rewritten to 00 C3 rev 3 (family-city `DayKey`) |
| F9 GAP-40 | storage low, audio cache LRU | applied (§4.7, §7.1 step 4) |
| F9 GAP-41 | data at rest | applied (new §9.9) |
| F9 GAP-44 | 04:00 cap day | **rejected** — 00 C3 rev 3 / OPD-26: one `DayKey`, midnight in the family city (§4.16 says so) |
| F9 GAP-48 | Maestro layer, fixture deep link, testIDs | applied (§4.1 dev links, §12.1) |
| F9 GAP-51 | `child_remove`, define `resync` | applied (§5.1, §6.2, §7.3, §7.4) |
| F9 GAP-56 | family delete grace | applied (§7.3, E-26) |
| F9 GAP-57 | pairing on a device with another family's data | applied (S3, E-27) |
| F9 GAP-57b | unpair = push then revoke, `force` | applied (S5, §7.4, §16 #5) |
| F9 GAP-67 | parent voice clips | applied (new §7.13, `voice_clips`, `family.voice` pack) |
| F9 GAP-68 | deep links `olam://` | applied (§4.1 closed list; 10 §11.9 `familygame://` → forwarded) |
| F9 (guardian remove) | devices stay, sessions revoked | applied (§7.3) |
| F9 GAP-19 | car mode | applied as a P2 stub (`car.tsx`, SCR-21) — 01 keeps it as a parent option |
| R1-4 | §8.2 −16 LUFS | applied (−18 LUFS ±1, ≤ −1 dBTP; 00 E9) |
| R1-7 | `bootstrap-guardian --role coadmin` | applied (= GAP-36) |
| R2 | `turn.yielded`, `ftue.completed`, `tz_mismatch`, `coop_letter`, `travel.city`, `expo-print`, `dayKey.ts` family-city offset | applied (§5.8, §6.1, §7.3, §2.3, §6.4) |
| R3 | `behavior.signal{kind:'nikud_lens'}` | applied (§5.8); 08 to add the kind to its union (forwarded) |
| R4-1 | the nine items | applied (all above; `coop_letter` schema chips-only in §6.1, `campfire.lantern_lit` in §5.8) |
| R5 #3 / 10 §18.2 | `k3.parentPromptTimeout`, `PUT /admin/ai-caps`, `GET /photos/jobs/:id` `estimatedReadyAt`, API-29 payload, extended enum | applied (§5.8, §7.12 aliases, §7.9, §7.3) |
| self-found | §1 diagram / §7.1 skeleton `/photos` → `/photo-jobs`, `/fs`, `/voice-clips`; `evaluateFlags` "nightly" wording → on device (§6.2, §7.1); §18 #8–11 marked accepted by 00 rev 3 | applied |

### 1.2 Items for 09 — UX/UI design system

| Source | Item | Disposition |
|---|---|---|
| F1 #6 | `chart.timeDirection` token + ⟂VERIFY | already resolved (§3.11 #10) |
| F1 #12 / F4 #19 | `type.en.letter`, `type.en.k1.decode` | already resolved (§3.6) |
| F2 #16 | flame icon set, `dragOrTapTap`, chest auto-open under accessibility, captions default on for K1, counting voice with numerals | applied (§3.10 `icon.flame.*`, §7 `inputMode`, §10 motor row); captions default on for K1 and numerals in counting were already in §10/§9.1 |
| F2 #17 | 5 receipt-line templates ×{m,f,pl} ×K1/K2 | applied (§9.2 R1–R5) |
| F2 #18 | gestures for MX-M14/M19/M16 | applied (§4.7) |
| F2 9d / R3 | `CoopZoneLayout` contract + snapshot per device class | applied (§4.6, `UX-AC-20`; SCR-13 references it) |
| F3 #20a | ⟂VERIFY two consecutive pointed lines | applied (§3.6, GATE-UX-2; fallback token `multiline` 1.7) |
| F3 #20b | nikud marks at 120 % | **decided: not in v1** (§3.6: `fontVariant` stays banned; a derived font is the only path and enters only as a gate candidate; bubble ×1.5 instead) |
| F3 #21 | `handedness` prop + snapshot per hand | applied (§7 common props, §10) |
| F3 #22 | praise key namespaces (`praise.math.*`, 12 §2.6) | applied (§9.2) |
| F4 #20 | `region.englishIsle` vs 01 `palette.english` | applied (§2.5: UI accent `grape`; `#19B7B0` → `color.char.sunny`, character only) — 01 to point here (forwarded) |
| F4 #21 | CMP-22: exit button, pause visible, no `rtl` inside | applied (§7.4, incl. GAP-30 cap warning deferred to island exit) |
| F4 #22 | B9(a) code-strip note | applied (§3.11 #9) |
| F5 #1 | `type.parent.*` ↔ iOS text styles, AX mapping, AX1+ layout rules | applied (§3.6) |
| F5 #2 | child level glyphs from `levels.ts` | applied (§3.3) |
| F5 #3 | B9 ⟂VERIFY owner | already resolved |
| F7 #11 | G4b under GATE-UX-6 + 13 §16.7 as the single schedule | applied (§14) |
| F7 #12 | music LUFS −26 vs −23 | applied: −26 confirmed canonical (§6.7); 12 §6.9/§8 to change −23 → −26 (forwarded) |
| F8 09 #1–#6 | `window.__olam`, memory targets, Rive sizes, `dirX`/`useDir`, UX-AC-17/18, `FixedSlot` | already resolved (round 1) |
| F9 GAP-08 / R2 | chest "auto after 1.5 s" vs 02 §6.3/§6.5 | applied (§5.3: hand at 4 s; auto-open only via the accessibility setting / Reduce Motion) |
| F9 GAP-16 | photo-game surfacing wording | applied (§8.3 F4) |
| F9 GAP-19 | car mode screen | applied (SCR-21, P2) |
| F9 GAP-19b | safe-room cap accounting | applied (SCR-16, `mode:'safe_room'`) |
| F9 GAP-29 | iPhone hub orientation | applied (§8.1: portrait-first canonical) — 01 DD-20 to align (forwarded) |
| F9 GAP-43 / R2 | K1 "camp asleep" view | applied (SCR-01, SCR-07) |
| F9 GAP-48 | `testID` everywhere | applied (§7, §12 #17) |
| F9 GAP-60 | Guided Access passcode | applied (§10) |
| F9 (SCR-13) | K3 lobby parent-present gate | applied (SCR-13) |
| R2 | P26 persistence, P27 neutral chain escalation | applied (§9.2) |
| R5 #5 | picker tile "הגיבור כבר מתכונן לשינה", camp-asleep after the 30-min unlock | applied (SCR-01, cites 10 §6.13 (a)/(d)) |

### 1.3 Items for 14 — performance & quality

| Source | Item | Disposition |
|---|---|---|
| F7 #17 | pack split ownership | applied (`PQ-B-24`) |
| F7 #18 | quiet days in `calendar-guard` | applied (`PQ-REL-01` #6) |
| R1-2 | `UIRequiresFullScreen` assumptions | applied (`PQ-R-47`, `PQ-DEV-12`, §11, ⟂VERIFY #6) |
| self-found | splash 3 s target / 4 s hard (11 §4.7, 09 §11); parent Dynamic Type AX5 with AX3 gate (09 §3.6, 11 R-11); `PQ-X-20`/`PQ-REL-04` follow `UpdateGate` moments; `PQ-T-04` gate list; `PQ-T-06` fixture links + `testID`; stale canary note | applied |

## 2. New / forwarded issues for other owners (blockers only)

| # | Owner | Problem | Proposed fix |
|---|---|---|---|
| R6-1 | 01 (§2.2, DD-20, DD-31) | `palette.english = #19B7B0` conflicts with 09 tokens; DD-20 says iPhone hub landscape; DD-31 says `DayKey` by device tz. | §2.2: "UI accent = 09 `grape`; `#19B7B0` = `color.char.sunny` (character only)"; DD-20: hub portrait-first (09 §8.1); DD-31: `DayKey` = family/travel-city day (00 C3 rev 3). |
| R6-2 | 12 (§6.9, §8 music, §6.11) | Music loudness −23 LUFS vs 09 §6.7 canonical −26 (00 E9: 09 owns the mix table). | Change −23 → −26 in §6.9 and the music row of §8. |
| R6-3 | 10 (§11.9) | Deep-link scheme `familygame://report/{id}`. | `olam://report/{id}` (11 §4.1 closed list). |
| R6-4 | 08 (§1.9, §1.10) | 11 registers `behavior.signal{kind:'nikud_lens', wordId, lexId, msShown}` (R3) and the wire `mode:'safe_room'` (dropped by `toEngineEvent`, 0 cap minutes). | Add `nikud_lens` to the `BehaviorSignal` union; confirm `safe_room` stays outside `Session.mode`. |
| R6-5 | 07 (§2.7) | "`rf_raised {ruleId, week}` syncs as an event" — 11 carries it only as a field inside the encrypted `flags/<childId>` doc (C8a, 11 §5.7/§9.8), never as a plain event. | One sentence: "`rf_raised` = field of the encrypted flags doc (11 §9.8)". |
| R6-6 | 13 / 15 | Maestro flow path: 14 `PQ-T-06` and 11 §12.1 use `apps/mobile/e2e-maestro/*.yaml`; 15 §3 GAP-48 and 13 say `tools/e2e/maestro/*.yaml`. | Use `apps/mobile/e2e-maestro/` (next to the Playwright specs). |
| R6-7 | 02 (§2.1, UC-16) | `DayKey`/`tzOffsetMin` from the device offset. | Family/travel-city offset per 00 C3 rev 3 (same as R5 §C #1). |

## 3. Notes

- 00 C3 rev 3 (family-city `DayKey`) arrived mid-round; 11 §4.8/§5.7/§6.4 were rewritten to it after a first pass had used the device offset. 09/14 are unaffected.
- 11 grew to 4,018 lines; the largest additions are §5.8 (registry), §7.9/§7.13, §8.4, §9.9 and the §4.16 cap/bedtime rules. No section was restructured.
- ⟂VERIFY added this round: re-sign container preservation (11 §3.3), `expo-print` nikud rendering (11 §2.3), iOS file-protection class (11 §9.9), two-line nikud spacing (09 §3.6), Stage Manager with `UIRequiresFullScreen` (14 #6, owned by 11 §0).
