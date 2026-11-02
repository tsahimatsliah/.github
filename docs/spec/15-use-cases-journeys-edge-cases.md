# 15 — Use Cases, Journeys & Edge Cases (end-to-end catalog for E2E tests and acceptance)

> **Status:** v1.0, 2026-11-02 (Fable reviewer F9). **Subordinate to** `00-foundation-decisions.md`; where this file and an owning spec disagree, the owning spec wins and the disagreement is listed in §3 (gap list) and in `docs/review/fable-issues-F9.md`.
> **Purpose:** one place where a developer (or the agent) can read every journey a real person or the system goes through, step by step, with the edge cases, the expected behavior, the spec section that defines it, and a test sketch that can be turned directly into a Playwright (web export), Maestro (device) or Jest/sim test.
> **Language:** English prose; Hebrew UI strings, IDs, routes and spec IDs as they appear in the owning specs.

---

## 0. Conventions

### 0.1 IDs and references

- Journeys are `UC-nnn`. Edge cases inside a journey are `UC-nnn/En`. Gaps are `GAP-nn` (§3). Test sketches are `E2E-UC-nnn-x`.
- Spec references use `NN §x.y` (file number + section), e.g. `10 §7.1 ON-07` = `10-parent-area-dashboard.md`, section 7.1, screen ON-07. `00 H2` = decision H2 in the foundation file.
- Placeholder names follow 10 §6: **נועה** = K1 (f), **איתי** = K2 (m), **{K3}** = K3 (`n` until the parent confirms gender, 00 A2, 01 DD-05).

### 0.2 Test layers and harness hooks (what each sketch assumes)

| Tag | Layer | Tool | Harness it relies on |
|---|---|---|---|
| `[PW]` | Web export of `apps/mobile`, deterministic mode | Playwright (11 §12.2) | `EXPO_PUBLIC_TEST=1`: frozen clock `2026-11-08T16:00:00+02:00`, seeded RNG, animations off, `window.__olam.ready`, `window.__olam.world.moveTo()`, `window.__olam.motion.feed()`, `window.__perf`; route interception for offline; fixture families (§0.3) |
| `[MAE]` | Dev/family build on a real iPad/iPhone or simulator via the parent's Mac (Remote Control) | Maestro flows (`tools/e2e/maestro/*.yaml`) — **proposed, see GAP-48** | `testID`s on every interactive element (09 §12 polish checklist), a debug deep link `olam://test/fixture?id=FX-..` (dev builds only) that loads a fixture and sets the injected clock |
| `[API]` | Worker `olam-api` against a scratch D1 | Vitest + `wrangler dev` / miniflare (11 §12.1) | seeded family; `Idempotency-Key`; cron handlers invoked directly with a fake `now` |
| `[SIM]` | Pure engine/economy/alerts logic | Jest + fast-check (08 §12, 02 §14.1, 10 §16) | `packages/engine`, `packages/engine/economy`, `packages/engine/alerts`, `packages/engine/metrics` |
| `[📱]` | Needs a human eye/hand (parent) | 11 §12.4 / 13 §10.3 checklist | — |

Every sketch lists: **Given** (fixture + clock), **When** (steps), **Then** (asserts). A sketch marked `⟂VERIFY` depends on a harness hook that is proposed here and not yet in a spec.

### 0.3 Fixture catalogue (proposed names, used by sketches)

| ID | Contents |
|---|---|
| `FX-FRESH` | No device token; nothing installed-state. First run. |
| `FX-FAM-NEW` | Family created, 3 children, owner PIN set, one `shared_kids` iPad paired, placement not started (ES-0). |
| `FX-FAM-W1` | Same, end of week 1 (01 §14.3): each child ≥8 friends, K1 level 3, K2 level 2, campfire level 2, 1 family treasure. |
| `FX-FAM-W6` | Six weeks of play: K1 ~240 friends, mastered skills in 3 subjects, pets at "young", 2 chests on the shelf, wishlist with a savings goal at 60 %. |
| `FX-FAM-OFFLINE-3D` | `FX-FAM-W6` plus 3 days of unsynced events on the iPad (≈400 events) and 2 unsynced `child_settings` docs. |
| `FX-ALERTS-6` | A week of events producing 6 alert candidates per child (10 AC-10). |
| `FX-PHOTO-PRINTED` / `FX-PHOTO-HANDWRITTEN` / `FX-PHOTO-STORY` / `FX-PHOTO-NOT-LEARNING` | Worksheet images for 12 §3 cases (synthetic, no real children). |
| `FX-CAL-5787` | Full Hebrew calendar table for 5787 incl. Tel Aviv + Jerusalem candle times (10 AC-11, 01 AC-EV-1). |

### 0.4 Environment matrix every journey is checked against

child (K1/K2/K3) × device (iPad shared / iPhone single-child / parent iPhone / web) × online/offline × first-run/returning × solo/co-op × weekday/Shabbat/holiday/quiet day × failure (vendor, network, crash) × parent action in parallel × accessibility setting (Reduce Motion, Dynamic Type, VoiceOver). A journey lists only the cells where behavior differs.

---

## 1. Actors

| Actor | Who | What they can do (summary) | Primary specs |
|---|---|---|---|
| **K1** | נועה, girl 8.5, grade 3 | Full economy (coins, gems, chests, shop, season track), text + voice UI, joystick option, Stretch/Mystery flames, guide mode, picture-PIN on by default, K1-only AI (גלגלון) live turns, time-trial vs. own ghost, ASR reading check (parent present) | 00 A2 D5; 01 §5.3; 02; 08 §6.6; 12 §4.2 |
| **K2** | איתי, boy 5.5, gan chova | Coins + wood/gold chests + pet, voice-only UI, tap-to-move + jump button, 2 plan choices, scripted hints only, no reading required, English "sandwich" | 00 A2 D5 B2; 01 §5.3; 02 §2.3; 12 §4.2 |
| **K3** | {K3}, toddler 2.5, gender `n` | Lap mode only (parent opens via gate), stickers only, no fail states, playroom diorama, "copy me" movement, no ASR/camera/currency; profile launchable only parent-present | 00 C3 D5 H3; 01 §15.3; 11 §4.16 |
| **Owner parent** | First guardian (`owner`) | Everything in 11 §5.4: family, children, settings, approvals, rewards, devices, invites (co-admin + viewer), export, delete, ops | 00 H1 H4; 10 §2; 11 §5.4 |
| **Co-admin parent** | Second guardian (`coadmin`) | Same as owner except: cannot delete child/family, cannot invite co-admins, cannot change roles/ownership, no `admin.ops` | 10 §2; 11 §5.4 |
| **Viewer** | Grandparent (`viewer`) | Celebrations feed + share cards + viewer weekly mail (no numbers); reactions ❤️👏🌟; optional voice note (parent-approved); web only in phase 1 (GAP-35) | 10 §2 RB-2 RB-4, §13; 11 §5.10 |
| **L (local parent)** | Anyone who passes the gate on a `shared_kids` device without a guardian session | Device-local caps/quiet hours, lock/unlock this device, read reports of this device's children | 10 RB-0; 11 §5.3 S4, §5.4 |
| **Developer agent (ops)** | Claude via CI / Remote Control on the parent's Mac | Bootstrap owner code, builds + 7-day re-sign, OTA promotion, nightly cron ownership, cost caps, backups, restore drills | 11 §3, §5.2, §11, §15.2; 13 §3 |
| **System** | Cron Triggers, Durable Objects, engine | Nightly content batch, alerts queue every 15 min, Friday report window, monthly rollups, pack signing, co-op rooms, local engine (placement, SRS, flags) | 10 §15.3; 11 §7.1, §10.2; 12 §2; 08 |

---

## 2. Journeys

Each journey: **Actors · Preconditions · Happy path · Alternative paths · Edge cases · Expected behavior / acceptance · Spec refs · E2E sketch.**

### 2.A Install, registration, identity, devices

#### UC-001 First install on the family iPad (free personal team, cable via the parent's Mac)

- **Actors:** Owner, developer agent, system.
- **Preconditions:** Parent has a Mac with Xcode + Apple ID (personal team), Claude desktop Remote Control folder set (11 §3.3 one-time setup); iPad in Developer Mode and trusted; no paid program (`OLAM_SIGNING=free`).
- **Happy path:**
  1. Agent runs `APP_VARIANT=family npx expo prebuild -p ios --clean` then `npx expo run:ios --configuration Release --device <UDID>` over Remote Control (11 §3.3).
  2. iOS prompts "Untrusted Developer" on first launch → parent trusts in Settings → General → VPN & Device Management (once per team per device).
  3. App launches to `/(setup)/welcome` ("הגדרת מכשיר חדש") with two buttons: "אני הורה – הקמת משפחה" / "צירוף מכשיר למשפחה" (11 §4.1).
  4. Agent records `tools/mac/devices.json` entry and `~/.olam/resign-state.json` (installedAt); LaunchAgent re-sign routine is scheduled (11 §3.3).
  5. `app.start` telemetry carries `runtimeVersion`, `updateId`, `signedAt` (11 §5.8, §6.5 device block).
- **Alternative paths:** (a) Concept stage: PATH 0 web export opened in iPad Safari, "Add to Home Screen" — concept/taste only, not for kids' daily use (11 §3.2). (b) Paid program later → PATH 2, team-migration checklist (11 §3.4, UC-076/E3).
- **Edge cases:**
  - E1: Free team already has 3 apps on the device → install refused; agent must remove another app or use the `family` variant only (11 §3.3 "Free-team app slots").
  - E2: iPad locked/asleep during network install → routine retries next day; at ≥6 days posts `POST /v1/ops/notify` + macOS notification (11 §3.3).
  - E3: Signing team changed → iOS may force delete-and-reinstall, wiping SQLite → must run full sync first; recover via UC-081 (11 §3.3 warning).
  - E4: Expo Go attempted → not a path for SDK 58 (11 §3.1).
- **Expected behavior:** Launch ≤2.0 s cold to picker on the oldest iPad (09 §11); `extra.buildSignedAt` injected and visible later as "תוקף ההתקנה: עוד N ימים" in the parent area (11 §3.3). Kids never see expiry.
- **Spec refs:** 00 E1a; 11 §3.1–3.3, §4.1, §12.4 #1 #15; 13 §3.1–3.3.
- **E2E sketch:** `E2E-UC-001-a [MAE]` Given fresh device + `FX-FRESH` → When app opens → Then welcome screen shows both buttons in Hebrew RTL, no child UI reachable, `testID=setup.welcome`. `E2E-UC-001-b [📱]` 11 §12.4 row 1 and 15.

#### UC-002 Owner registration and family onboarding (ON-01 → ON-12)

- **Actors:** Owner, developer agent (bootstrap code), system (Worker).
- **Preconditions:** UC-001 done; agent ran `pnpm ops bootstrap-owner --email <owner>` and gave the parent the one-time code `XXXX-XXXX-XXXX` (24 h TTL) in chat (11 §5.2).
- **Happy path:**
  1. ON-01 welcome (three promises) → "להתחיל".
  2. ON-02 "יש לי קוד הקמה" → `POST /v1/auth/bootstrap/redeem {code, guardianName, email}` → 201 with tokens; family + owner created; `audit(family.created)` (11 §5.3 S1).
  3. ON-03 family name, **city** (mandatory; drives Shabbat times), Shabbat mode (default "מנוחה"), Hebrew date toggle → `PATCH /v1/family`.
  4. ON-04 ×3 children (UC-003). ON-05 parent questionnaire per child (skippable; K3 ~5 min, K1/K2 2–3 min; 08 §3.3).
  5. ON-06 PIN ×2 + Face ID + shared-device question (UC-005).
  6. ON-07 this device registered implicitly `POST /v1/devices/self`; ON-08 mode `shared-kids` + allowed children (K3 only with lap mode).
  7. ON-09 quick defaults (caps, bedtime, quiet hours, report email). ON-10 consent layers, **no toggle pre-checked, each must be answered** (10 §8 PRIV-02). ON-11 invite second parent (skippable). ON-12 "המפות מתחילות" explanation.
  8. `done.tsx` → placement expeditions start per child at each child's next entry (UC-009).
- **Alternative paths:** Stop after ON-06 and resume later (10 §7.1, AC-23): every step persisted; app reopened → continues at the next incomplete step. Email-code sign-in instead of bootstrap once a sending domain exists (11 §5.2 `email_code`).
- **Edge cases:**
  - E1: Bootstrap code expired/used → error `{code:'bootstrap_invalid'}` in Hebrew, no enumeration; agent issues a new one.
  - E2: Owner leaves ON-04 with only one child → allowed (AC-23); other children added later in FAM-03.
  - E3: City not chosen → ON-03 blocks with inline error (10 §7.1 "חובה לבחור עיר").
  - E4: No sending domain yet (Q-05) → email code and emails unavailable; onboarding still completes via bootstrap; weekly report lands in PA-INBOX only (10 §6.16).
  - E5: Onboarding on a `parent_only` iPhone first → device registered as `parent_only`; kids' iPad paired later (UC-006).
  - E6: K3 grammatical gender "עוד לא החלטנו" → `n`; all K3 strings render neutral; DD-05 alternation only where neutral impossible.
- **Expected behavior:** ~12 min total; every step saved; placeholders where personal info missing (00 F2); audit rows for family.created, child.created, settings changed.
- **Spec refs:** 00 H2; 10 §7.1 (ON-01…ON-12), AC-23; 11 §5.2, §5.3 S1, §4.1 `(setup)/`.
- **E2E sketch:** `E2E-UC-002-a [PW]` Given `FX-FRESH` + intercepted API → When walk ON-01…ON-06, kill page, reload → Then resumes at ON-07, PIN hash was sent once (`PUT /v1/guardians/me/pin` called 1×). `E2E-UC-002-b [API]` redeem twice → second returns 4xx; audit has exactly one `family.created`. `E2E-UC-002-c [PW]` ON-10 "המשך" disabled until all consent toggles answered.

#### UC-003 Adding a child (ON-04 / FAM-03)

- **Actors:** Owner / Co-admin.
- **Preconditions:** Signed-in guardian.
- **Happy path:** name, nickname, birthdate, grammatical gender (f/m/n), grade/gan suggested from birthdate and **confirmed by parent**, active subjects, avatar now or "the child builds it at first entry" (recommended), picture-PIN (K1 default on) → `POST /v1/children` → audit `child.created` → child appears in picker on allowed devices after sync.
- **Alternative paths:** Edit later in FAM-03 (gender change warning: all texts/narration switch, "no download needed" — see GAP-52 for photo packs); archive; reset profile model (interests/dispositions only).
- **Edge cases:**
  - E1: Birthdate implies a band (T/G/S) different from the chosen grade → parent's grade wins; band derived from grade for curriculum, age for touch targets (09 §4.2) ⟂VERIFY which drives `child.band` — see GAP-04 note in 07/06 adoption.
  - E2: Fourth child added → schema supports N (11 §5.1); hub has 3 neighborhoods (01 §10.1) → GAP-53.
  - E3: Same name twice → allowed; nickname disambiguates in picker.
  - E4: Gender changed while the child is mid-session on another device → doc LWW; the running session keeps old variants until next screen; next sync swaps (10 AC-25 "within one refresh").
- **Expected behavior:** Child row has no credentials (11 §5.1); K3 profile only launchable in lap mode (00 H3).
- **Spec refs:** 10 §7.1 ON-04, §7.2 FAM-03, AC-25; 11 §5.1, §7.3; 00 B3.
- **E2E sketch:** `E2E-UC-003-a [PW]` add child with gender `n` → picker tile + first greeting rendered neutral; switch to `f` → greeting feminine in one reload; snapshot both.

#### UC-004 Avatar builder (parent-built likeness, child-edited cosmetics)

- **Actors:** Parent, child.
- **Preconditions:** Child exists.
- **Happy path:** Parent (gate) opens "יוצרים את X שלי" wizard (~5 min): skin tone, hair, eyes…, free offset/scale sliders (Mii approach) → save. Child later edits only cosmetics (clothes, accessories, party hair color) in SCR-09; K3 does not edit (parent builds; child has 3 "surprise" buttons).
- **Alternative paths:** "The child builds at first entry" (ON-04): FTUE includes the פנסון lab (01 §14.2) — note the **hero** (פנסון) parts are child-chosen in FTUE while avatar face structure stays parent-only.
- **Edge cases:** E1: Child tries to change skin tone → control not present for child (01 §3.3 step 3). E2: Parent uploads photo → not possible by design (no photos, no face scan). E3: Hero parts from child drawings (01 §3.4) go through the photo pipeline with on-device processing only; poor quality → "sticker" filter (DD-22), never "unsuitable".
- **Expected behavior:** No item gender-locked; catalog never filtered by gender (01 §3.3; 02 §7.1).
- **Spec refs:** 01 §3.3–3.4, DD-22; 09 §8.4 SCR-09; 00 G4.
- **E2E sketch:** `E2E-UC-004-a [PW]` as K2 open SCR-09 → assert no skin-tone/face-shape controls; as parent (gate) → assert they exist. `E2E-UC-004-b [PW]` catalog identical for K1 and K2 profiles of different gender (diff of item ids).

#### UC-005 Parent PIN, Face ID and the shared-device check (ON-06)

- **Actors:** Owner / Co-admin.
- **Happy path:** PIN (6 digits, typed twice, not a trivial sequence, different from the other parent's) → scrypt hash computed on device (N=2^15, r=8, p=1) → `PUT /v1/guardians/me/pin` → Face ID enabled only if `capabilities.faceId` → question "does a child have Face ID/Touch ID enrolled or know the device passcode?" → if yes, this device is PIN-only (SEC-02).
- **Edge cases:** E1: `123456`/`000000` rejected inline. E2: PIN equals the other guardian's → rejected (10 §7.1). E3: Web/PATH 0 → PIN only (11 §3.1 table). E4: Co-admin sets their own PIN at join (UC-007); hashes of **all** guardians sync to devices so the gate works offline (11 §4.8).
- **Spec refs:** 10 §7.1 ON-06, §9 SEC-02/03; 11 §5.3 S4, §7.3; 00 E7.
- **E2E sketch:** `E2E-UC-005-a [PW]` enter `123456` → inline error; enter valid → `PUT …/pin` body contains hash only (never the PIN). `E2E-UC-005-b [MAE]` 11 §12.4 row 10.

#### UC-006 Pairing a second device (QR or 6-digit code)

- **Actors:** Guardian (signed-in device), new device, Worker.
- **Preconditions:** New device has the app installed (UC-001 install step repeated per device).
- **Happy path:** Guardian device: devices → "הוספת מכשיר": name, type, mode, allowed children → `POST /v1/pairings` → QR (`olam://pair?p=<pairingId>&n=<nonce>&api=<base>`) + code "482 913", 10-min TTL; new device: "צירוף מכשיר למשפחה" → scan (expo-camera) or type → `POST /v1/pairings/claim` → device token in Keychain → initial pull sync (cursor 0) → picker; guardian screen polls every 2 s and shows "המכשיר צורף ✓"; audit `device.paired`.
- **Alternative paths:** Camera denied → type the code. Pair from the web dashboard (shows QR + code) (11 §5.10).
- **Edge cases:**
  - E1: Code expired (>10 min) → `pairing_expired`, retryable=false, Hebrew message; guardian generates a new one (13 §10.3 "קוד שפג תוקפו נדחה").
  - E2: 5 wrong attempts per pairing or 5/10 min/IP → rate-limited; audit `pairing.failed_burst`.
  - E3: New device already paired to **another** family (edge: reinstall by a relative) → claim must require local wipe first — GAP-57.
  - E4: Offline new device → pairing impossible (11 §4.8); UI says "צריך אינטרנט".
  - E5: K3 in allowed children on a `single_child` device → rejected (K3 only with lap mode on `shared_kids`/`parent_only`, 10 §7.2 FAM-04).
  - E6: Shared iPad paired with no guardian session by default → parent area runs in L mode until "התחברות הורה במכשיר זה" (11 §5.3 after S3).
- **Expected behavior:** QR scan < 3 s (11 §12.4 row 11); first pull expected ~6 MB/year of events (11 §6.6).
- **Spec refs:** 10 §7.1 ON-07/ON-08, §7.2 FAM-04; 11 §5.3 S3, §7.4, §4.11 error shape; 13 §10.3.
- **E2E sketch:** `E2E-UC-006-a [API]` create pairing, advance clock 11 min, claim → `pairing_expired`. `E2E-UC-006-b [PW]` two browser contexts: A creates pairing, B types code → A shows "המכשיר צורף ✓" within 3 polls; B lands on picker with the allowed children only. `E2E-UC-006-c [MAE]` scan QR with the camera.

#### UC-007 Inviting and joining a co-admin parent

- **Actors:** Owner, co-admin (own iPhone), Worker, email (Resend).
- **Happy path:** FAM-02 choose role `coadmin` + channel (QR / link / email) → `POST /v1/invites` → 8-char code, link `https://parent.<domain>/join?c=…`, 7-day single-use → shared outside the app → co-admin opens link/app "הצטרפות" → `POST /v1/invites/accept` → email code → `POST /v1/auth/email/verify {…, inviteCode}` → guardian created with role from invite → sets PIN + Face ID (UC-005) → pairs own iPhone as `parent_only` → owner gets email "X הצטרף/ה כמנהל/ת משנה"; audit `invite.created`, `guardian.joined`.
- **Alternative paths:** Phase 2 with Sign in with Apple. Resend/cancel pending invite (FAM-01).
- **Edge cases:**
  - E1: Link opened after 7 days or twice → "ההזמנה פגה" (AC-24).
  - E2: Co-admin tries to invite a co-admin → UI hidden + server 403 (`guardian.invite` viewer-only for C).
  - E3: No email domain yet → email code impossible → **GAP-36** (proposed ops bootstrap code for the second parent).
  - E4: Co-admin's PIN collides with owner's → rejected at ON-06.
  - E5: Co-admin on Android → app install impossible; web dashboard only (P1 read-only, P2 edit) (10 §14.3).
  - E6: Owner removes co-admin → their sessions revoked; devices they paired stay in the family (device belongs to family, not guardian) ⟂VERIFY in 11 §7.3 remove semantics.
- **Spec refs:** 00 H1 H2; 10 §7.1 (co-admin join), §7.2 FAM-01/02, AC-24; 11 §5.3 S2, §5.4, §7.3.
- **E2E sketch:** `E2E-UC-007-a [API]` accept same code twice → second 4xx; role of created guardian = `coadmin`; `guardian.invite(coadmin)` by that guardian → 403.

#### UC-008 Viewer (grandparent) invite, feed and reactions

- **Actors:** Owner or Co-admin, Viewer (web), K1 (consent), system.
- **Happy path:** FAM-02 role `viewer` by email magic link (no Apple account; grandparents may have Android) → viewer opens web `סקירה` with celebrations only → share card SC-01…06 approved by parent (and **K1 approves her own cards** at next entry: "רוצה שסבתא תראה? ✔ / לא עכשיו") → card appears in `GET /viewer/feed` via signed URL → viewer reacts ❤️/👏/🌟 → reaction reaches the child's hub mailbox immediately; voice note → PA-APPR A6 first.
- **Edge cases:**
  - E1: Viewer opens `/child/:id/overview` → 403 (AC-18).
  - E2: `viewer.showSkillNames` off (default) → no skill names in the viewer weekly mail T-W-VIEWER; on → names only, never numbers (RB-2).
  - E3: K1 says "לא עכשיו" → card stays pending, never sent; parent sees "ממתין לאישור של נועה" (10 §13).
  - E4: Card containing a number/level/minutes → impossible by construction (AC-28 lint on card template).
  - E5: Viewer tries app on a family iPad → no parent area access for viewers on kids' devices (RB-4).
- **Spec refs:** 10 §2 RB-2/RB-4, §7.1 (viewer join), §13, AC-18, AC-28; 11 §5.4, §5.10.
- **E2E sketch:** `E2E-UC-008-a [PW]` parent creates SC-01 for K1 → K1 picker entry shows consent card → tap "לא עכשיו" → viewer feed empty; tap ✔ → feed has 1 PNG 1080×1350 without EXIF (`[API]` check headers). `E2E-UC-008-b [API]` viewer token → `overview` 403.

#### UC-009 Placement expeditions ("מסע מיפוי המפה") per child

- **Actors:** K1, K2 (K3: questionnaire only), parent (hands-off), engine.
- **Preconditions:** ON-05 questionnaire answered or skipped; ON-12 shown to the parent ("don't help with answers").
- **Happy path (K2):** First entries over 2–4 days, ≤2 expeditions/day, each 3–5 min: E1 numbers (counting, magnitude), E2 sounds & letters, E3 story listening (+ optional English on the LTR island), E4 optional tracing; per axis min 4 / max 8 items; stop at SE ≤ 0.45 or max; two warm-up items P≥0.9 and a closing item P≥0.9 ("המפה מוכנה!"); every answer gives the same neutral-positive "תודה, המפה גדלה!" ≤0.8 s; no score, no timer, "קשה לי, אפשר אחר?" shrugs; rest item after 2 consecutive misses. Skills classified known/uncertain/unknown with closure; `known` → `proficient` with `provenance:'placement'` and a delayed check from day +3; honeymoon: first two episodes after placement at `targetP=0.90`.
- **Happy path (K1):** 3–4 expeditions × 8–10 min, ≤10 min hard stop; E1 math A, E2 Hebrew A (pointed + 3 unpointed items), E3 math B + English, E4 Hebrew B (spelling; read-aloud ASR only parent-present).
- **K3:** No expedition; parent questionnaire (12 items) + playroom telemetry only.
- **Alternative paths:** Time runs out mid-axis → axis saved `in_progress`, resumes from the posterior next time. Parent skipped the questionnaire → start by age, longer placement.
- **Edge cases:**
  - E1: Child quits the expedition (pause → back to camp) → `in_progress`, no penalty; PA-ASM shows "x/4".
  - E2: Sibling watches/answers → cannot be detected; ON-12 instructs; placement runs in the personal profile only (08 §3.6 #7).
  - E3: Expedition rescues: friends count toward the collection (01 §14.2 first friend), but **no coins/XP** (02 §2.7) → GAP-13 (needs an explicit `level.mode='placement'` flag).
  - E4: PA-HOME before any expedition → ES-0 "המפות של הילדים עוד נפרשות"; PA-MAP greyed (10 §6.1, §6.3).
  - E5: Placement window BOY 15.10–30.11 — launch 8.11 falls inside; later re-placement at MOY/EOY windows (08 §10.3) and after ≥21-day breaks (UC-048).
- **Expected behavior:** K1 reaches SE ≤ 0.45 after 8–11 items per axis with a questionnaire prior; K2 often stops at max with SE 0.50–0.55 → parent sees a range (08 §3.1). No economy events of class `econ` during placement.
- **Spec refs:** 08 §3.1–3.6, §10.3; 10 §7.1 ON-05/ON-12, §6.8 PA-ASM; 01 §14.2 (play-based), §14.3; 02 §2.7.
- **E2E sketch:** `E2E-UC-009-a [SIM]` 08 §12.4 tests + assert zero `econ.*` events in a placement session. `E2E-UC-009-b [PW]` K2 expedition: every answer (right or wrong) shows identical feedback class; no HUD coins; after 2 misses next item has `role=rest`. `E2E-UC-009-c [PW]` leave mid-axis → PA-ASM W-ASM-01 shows "1/3 הושלמו", re-enter → continues.

#### UC-010 Profile picker, picture-PIN and sibling impersonation

- **Actors:** K1, K2, K3 (via parent), device in `shared_kids` mode.
- **Happy path:** Picker shows only `device_children`; "turn ring" glow on whose turn is next by last use (01 §12.5); tap avatar → (K1) picture-PIN: 3-picture sequence from a 3×3 grid, verified locally (scrypt) → `session.started{mode:'solo'}` → hub ≤1.0 s (09 §11). Switching: long-press 1.5 s on the avatar badge → "החלפת שחקן" → resume doc `child_resume/<childId>` saved → picker; inactivity 5 min → picker; cap reached → wind-down → picker.
- **Edge cases:**
  - E1: K2 taps K1's avatar → picture-PIN prompt; 3 misses → "אפשר לבקש עזרה מאמא או אבא", no lockout timer, no punishment (11 §4.16) → **GAP-07** (what happens on the 4th tap; counting attempts for M-14).
  - E2: K2 taps K3's tile → parent-present gate required (hold-2 s / Face ID / PIN); K2 cannot start K3 (13 §10.3 "K3 נפתחת רק במצב משותף").
  - E3: Forced switch mid-level → nothing lost; the yielding child returns to the same beat later (01 §12.5) and gets a "Patience heart" + VIP friend surprise → **GAP-33** (rule for detecting a yield).
  - E4: `single_child` iPhone → straight into hub, switching hidden behind the gate (11 §5.5).
  - E5: Picture-PIN reset by parent (FAM-03) → next entry asks the child to set a new sequence ⟂VERIFY flow not written (GAP-07).
  - E6: Device offline → picker + picture-PIN work (hash local, 11 §4.8).
  - E7: Picker after goodnight for K2/K3 → tile shows "המחנה ישן"; re-entry only through the gate (01 §15.1); K1 → view-only camp (GAP-43).
- **Spec refs:** 00 H3; 01 §12.5, §15.1; 11 §4.16, §5.5; 09 SCR-01; 13 §10.3.
- **E2E sketch:** `E2E-UC-010-a [PW]` as device with 3 children: tap K1 → grid appears; wrong ×3 → helper text, grid still present, no timer; correct → hub. `E2E-UC-010-b [PW]` tap K3 → gate sheet appears; cancel → back to picker, no `session.started` emitted.

#### UC-011 K3 lap-mode launch and parent-present rules

- **Actors:** Parent + K3.
- **Happy path:** Parent passes the gate (or hold-2 s on a guardian-paired device) → `episode.k3.lap` 4:00 (3–5): פופי "קוקו" with the child's name in the parent's recorded voice → 3 picture doors → peekaboo level (no wrong answers) → "copy me" MV-01 → friends home + sticker → goodnight; on-screen **parent prompts** ("שאלו: איפה הפרה?") small, in a corner; ≥1 "do together" moment.
- **Alternative paths:** Playroom SCR-17 alone for a short time (safe: no exits, no text, no fail) → auto-stops after ≤5 min with a lullaby; still opened only through the gate.
- **Edge cases:**
  - E1: Multi-touch / palm → ignored; 12 mm edge margin has no actions; no swipe-to-exit; Guided Access recommended (01 §15.3, 13 §10.3).
  - E2: Volume: max −6 dB; no flashing >2 Hz; no shake; no dark scenes; no Muddler/approaching shadowlings (AC-K3-2).
  - E3: Second K3 episode in the day → cap: 2 episodes / 10 min (01 §15.1) vs 15 min (10 §6.13) → **GAP-03**.
  - E4: K3 solo minutes in learning episodes (not playroom) on 3 days → AL-06 (10 §10.1); M-14 target 0.
  - E5: Gender `n` → neutral VO; where impossible, alternate f/m per session (DD-05).
  - E6: Parent recorded no name → "אור בוקר" without a name (01 §19 item 3).
- **Spec refs:** 00 C3 H3; 01 §5.3 (K3), §15.3, AC-K3-1/2; 09 SCR-17; 10 §10.1 AL-06; 11 §4.16.
- **E2E sketch:** `E2E-UC-011-a [PW]` K3 episode: inject 2-finger touch → no action; tap at 6 mm from edge → no action; advance clock 5:00 in playroom → lullaby + sleep screen. `E2E-UC-011-b [PW]` scene config lint: no `muddler`/`shadowling.approach` in K3 scenes.

### 2.B Daily play (child side)

#### UC-012 K2 daily episode, minute by minute (`episode.k2.default`, 10:00)

- **Actors:** K2, engine, (parent nearby).
- **Preconditions:** Placement done or honeymoon; daily cap not reached; not bedtime/Shabbat-full; packs for the region present (bundled core works offline).
- **Happy path:** (01 §5.3 K2 table)
  1. 0:00–0:40 Arrival + Plan: פנסון wakes, ניצי "אור בוקר"; optional feelings check-in (UC-035); Lost Friends Board → 2 level choices + prediction 🌶/🌶🌶.
  2. 0:40–2:00 Warm-up "בוקר טוב לחברים": 3–4 due SRS items as sleeping friends (≥90 % expected).
  3. 2:00–6:30 Level Ki/Shō/Ten/Ketsu with one gimmick; movement moment ≤45 s inside Shō (UC-024); hint after 2 errors; secret room in Ten; lands on success.
  4. 6:30–8:00 Camp: friends move in (T1 each), coin receipt **after** the action (≤4 s, coins counted aloud, 02 §2.8), one decoration/build mission, feed the pet (1 free treat).
  5. 8:00–9:20 Short evening: 3–4 sleepy shadowlings, 4–5 mixed review items (UC-028).
  6. 9:20–10:00 Goodnight + Tell (😀/😐/😮‍💨 + "מה עזר?") + offline mission card → "camp asleep" (UC-039).
- **Alternative paths:** Second episode same day inside the cap; review node on the map (SCR-04 "חזרה" blooms like a bonus); co-op evening instead of solo evening when a sibling joins (UC-030).
- **Edge cases:**
  - E1: Engine has <8 scored items in a subject this week → raises that subject's weight (08 §10.1).
  - E2: 85 % of episode time passed → `closing` block; cap ends during closing → 60 s grace then warm end (08 §6.10) vs 01 "finish beat ≤1:30" → GAP-02 reconciliation.
  - E3: No due review items (day 1) → warm-up shortened, friends from onboarding are the "sleepers".
  - E4: Daily surprise present (max 1/day; accumulates up to 7 then "big letter", DD-25).
  - E5: Episode started 55 min before bedtime → 10 §6.13 "no new session from 60 min before bedtime" → GAP-04 precise rule.
  - E6: App backgrounded mid-level (call) → resume to "לגעת כדי להמשיך" at the checkpoint (11 §12.4 row 14).
- **Expected behavior:** Duration within 8–12 min in ≥95 % (AC-LV-2); celebrations ≤15 % of episode (AC-CEL-2); no coins in HUD during a level (01 §6.4).
- **Spec refs:** 01 §5.2–5.5, §6, §15.1; 08 §6.1–6.10; 02 §2.3, §2.8; 09 §8.3 F1.
- **E2E sketch:** `E2E-UC-012-a [PW]` deterministic K2 episode via `window.__olam.world.moveTo()` route + scripted answers → assert segment order, `celebration_ms_total/episode_ms ≤ 0.15`, HUD has no coin counter during `level/*`, receipt appears only on `hub` after the Ketsu T2. `E2E-UC-012-b [SIM]` 08 §12.6 composer tests with the K2 template.

#### UC-013 K1 daily episode (`episode.k1.default`, 18:00)

- **Happy path:** Plan (quest + flame Cozy/Stretch/Mystery + goal & prediction, DD-10: 3 choices + flame) → "סיור בוקר" 6–8 due items, optional time-trial vs. her own ghost (fluency only after mastery) → Level A (Ki/Shō/Ten with struggle-first 60–90 s in Stretch/Ketsu + maybe secret exit) → Block B (second subject level / region boss / secret level / runner) → Camp: Show + Build Mission + collection + shop/wishlist → Evening defense 2 waves with 2 light towers → Show&Tell: prediction vs. result → **Calibration Star** for an accurate prediction (not for success), "בפעם הבאה אני…" chips or 15 s voice note kept on device, optional 30 s Show&Tell recording for the parent, offline mission, diary page, goodnight.
- **Alternative paths:** Guide mode offer (UC-032); "Mystery/Stretch flame offered automatically after 3 levels ≥95 %" (DD-24); Cozy flame offered after <60 % over 8 items (01 §6.5).
- **Edge cases:**
  - E1: Interleaving: ≥2 subjects per ≥15-min episode (08 §6.8).
  - E2: Class topic with a test in 2 days → spaced retrieval ≤30 % of session (10 §6.13.1; UC-053).
  - E3: Voice notes (15 s / 30 s) never leave the device unless the parent saves a "family moment" ⟂VERIFY where the Show&Tell recording for the parent is stored (GAP-67-adjacent; see GAP-66).
  - E4: K1 chooses Stretch and hits 3 consecutive errors → offer "לנסות שוב / לעבור" (08 §6.7 stretch column).
  - E5: Episode cap 40 min/day: 2 episodes default (01 §15.1).
- **Spec refs:** 01 §5.2–5.3 (K1), §9.2, §9.4, §9.5, DD-10, DD-24; 08 §6.6–6.8; 02 §2.2, §2.8.
- **E2E sketch:** `E2E-UC-013-a [PW]` plan predicts "6/7", rescue 6 → Show&Tell shows Calibration Star; predict "7/7", rescue 6 → no star, process praise only (string ∉ banned list). `E2E-UC-013-b [SIM]` composer: K1 18-min episode contains ≥2 subjects; Stretch block targetP 0.65–0.70.

#### UC-014 K3 lap episode (details)

See UC-011. Additional acceptance: every K3 episode counts as an "adventure day" for the family goal (02 §10.1); the sticker album page fills (12 pages × 6); each 5 toy friends → a new toy; "מעטפת הפתעה" with 2–3 stickers at family goal and holiday events (02 §6.2 K3).

- **E2E sketch:** `E2E-UC-014-a [PW]` complete a lap episode → sticker count +1, `adventureDay(K3, today)=true`, zero `econ.earn` with `currency:'coins'` for K3.

#### UC-015 Level play: rescue on the path, secrets, checkpoints, falls

- **Actors:** K1/K2.
- **Happy path:** Gate iris (≤1.5 s) → Ki introduces the gimmick with one new item (no possible failure) → Shō: 2 friends (1 on path, 1 slightly off), movement moment → Ten: twist + secret room, interleaved items → Ketsu: last easy friend + T2; friends fly to the HUD jar (T1 ≤1.5 s, input stays active) → back to hub: arrival animation ≤2 s (AC-HUB-1); threshold sign "עוד N חברים" updates.
- **Secrets:** ≥4 of 8 secret types per level; glint every 3 s; pet points after 20 s without progress / 45 s without a secret; density ≥1 interactable per 6×6 tiles; secret friend = curiosity + an easy learning action.
- **Edge cases:**
  - E1: Fall into a gap → bounce 0.6 s (K2) / 0.8 s (K1) to the nearest checkpoint, no loss (AC-LV-5).
  - E2: Stuck 60 s (K2) / 90 s (K1) → pet points + glowing path.
  - E3: Secret not found after 3 visits → 4th visit the pet points immediately (DD-17).
  - E4: Replaying a finished level → friends already found don't pay again; K2 always gets a spark trail to remaining friends, K1 optional radar (02 §5.2 #7, §2.7).
  - E5: Secret exit (10 % K2 / 15 % K1 of levels) → hidden level; first completion: wood chest with a region-unique item (02 §5.3).
  - E6: Level file fails lint (missing beat/secret types) → CI blocks (AC-LV-1).
  - E7: Pack for a new region not downloaded (offline) → gate shows "בקרוב" ⟂VERIFY copy (GAP-40).
  - E8: Mini-game crash → error boundary returns to hub, attempt discarded, no penalty (11 §4.11).
- **Spec refs:** 01 §6.1–6.6, §10.1–10.2, DD-17, AC-LV-1…6, AC-HUB-1/2; 02 §5.1–5.3; 11 §4.11.
- **E2E sketch:** `E2E-UC-015-a [PW]` route through a seeded level: assert `world.rescue` events = friends on path only after `item.answered{correct:true}`; move during T1 → hero moves (AC-LV-4). `E2E-UC-015-b [PW]` idle 20 s → pet pointer visible; 45 s → secret glint intensifies. `E2E-UC-015-c [PW]` throw inside a mini-game via `window.__olam.test.throw()` ⟂VERIFY → hub, `app.error` event, friend jar unchanged.

#### UC-016 Failure handling, frustration guard and the break offer

- **Actors:** K1/K2/K3, engine.
- **Happy path (no-fail ladder):** wrong → informative bounce (no loss; "עוד לא" for K1); 2 consecutive first-try errors on a skill → hint (גלגלון step 1) + step down (`pTarget = min(0.97, targetP+0.10)`, one more concrete CPA representation); 3 → worked example by ניצי + near-identical item (K1) or switch to a familiar easy item P≥0.95 (K2); under age 6 never >3 misses in a row in any state; `consecAny=3` under 6 → switch to a favorite; 4 (K1) → switch activity.
- **Frustration score** over last 5 items: +2 per extra consecutive error, +2 rage taps (≥5 taps/2 s on inactive area), +1 idle >10 s after an error, +2 exit attempt ≤10 s after an error, +2 😠/😢 at check-in, +1 rapid guess → score ≥4 → `offer_break`: dragon breath / break / switch to a favorite topic (K1 also "another way"). Success after a break → `recovery` (reported to the parent as a strength).
- **Edge cases:**
  - E1: <60 % over 8 items → engine lowers level; ניצי "lantern breath"; next item surely easy; K1 offered Cozy flame.
  - E2: K1 in Stretch: 2 errors → hint only; 3 → choice "לנסות שוב / לעבור".
  - E3: Error sound: soft 150 ms "בופ", never buzzer/red X for K2/K3.
  - E4: After a switch, the skill doesn't return in the same episode unless in a review block.
  - E5: ≥3 sessions in 7 days ending `childQuit` ≤120 s after ≥2 errors → AL-04 (UC-061); `EMO_DISTRESS` after 3 weeks (UC-062).
  - E6: Rapid guess (K1, rt<800 ms wrong) → item pays 0, never shown as punishment (02 §2.6).
- **Spec refs:** 01 §6.5, §16.1 (error sound), AC-LV-6; 08 §6.7; 02 §2.6; 10 §10.1 AL-04.
- **E2E sketch:** `E2E-UC-016-a [SIM]` fast-check: for K2, no sequence of engine outputs yields 4 consecutive first-try misses on the same item. `E2E-UC-016-b [PW]` feed rage taps (6 taps in 1.5 s on background) → `offer_break` sheet with 3 options, no coin change.

#### UC-017 Chest earned, previewed, revealed, shelved (D2)

- **Actors:** K1/K2, (K3 surprise envelope).
- **Happy path:** Trigger (e.g. skill → `proficient` = wood; mastered = silver + 1 gem; region boss = gold choose 1 of 3) → `chest.earned{contents[] fixed}` at earn time → frame shape+color shown from the first frame (round/shield/star/crown) → preview silhouette + label ("בפנים: כובע פיראט + 40 מטבעות", read aloud for K2) → taps: 1/2/3/3+600 ms long-press by tier (02 §6.1) → burst (≤3 flashes/s; soft glow in calm mode) → reveal → settle: coin counter ticks ≤1 s → collect; total without taps ~2.9–5.8 s (02 §6.5).
- **Alternative paths:** "Keep on the treasure shelf" to open later (e.g. with Abba) — quiet number badge, no red dot; skip to beat 7 after first viewing of each tier; no tap for 4 s → animated hand; accessibility "auto-open".
- **Edge cases:**
  - E1: Track exhausted → fixed coins (K1 25 / K2 4) shown in advance (02 §6.3 #3).
  - E2: Sync collision: two offline devices earned the same track item → later chest keeps the item; reveal says "כבר יש לך את זה, קיבלת במקומו X מטבעות" (band A); collected items are never taken (02 §6.3 #6).
  - E3: Quiet day (QD) → chest goes to the shelf without auto-reveal; no fanfare/confetti (02 §6.5).
  - E4: Reduce Motion → flash off, particles ≤3 (09 §5.5).
  - E5: Chest reveal interrupted (lock screen mid-animation) → `chest.opened` already persisted at tap; resume shows the collected state (13 §10.3 "נעילת מסך באמצע חגיגה").
  - E6: 01 "effort marks = taps" vs 02 tiers; 09 "auto after 1.5 s" vs 02 "hand after 4 s" → GAP-08.
- **Spec refs:** 00 D2; 02 §6.1–6.5; 01 §16.2; 09 §5.3 (CHOREO chest), §5.5; 11 §5.8.
- **E2E sketch:** `E2E-UC-017-a [SIM]` lint `Math.random` banned in `economy/`; property: `chest.earned.contents` equals `chest.opened.contents`. `E2E-UC-017-b [PW]` fixed-frame screenshot of preview; tier label visible before any tap; QD date → chest lands on shelf, no reveal route.

#### UC-018 Shop, wishlist, savings goal, jars and refund

- **Actors:** K1 (full), K2 (simple), parent (refund).
- **Happy path:** Shop opens at its milestone (K1 after first 100 coins, 09 SCR-02 #6); showcase of 6 rotates **weekly on Sunday**; catalog permanent, fixed prices, bands A–D; "חסרים לך עוד ___" (K1) / stacks of 10 (K2); wishlist (K1 ≤5, K2 1 "המשאלה שלי"); savings goal shows "about N weeks" from the 4-week average, no date; jars spend/save/give with split 70/20/10 default (K2 70/30); weekly savings bonus Sunday; buy → item to inventory → (K1) "ללבוש עכשיו?" → SCR-09; **no auto-jump to the shop** after a chest (09 F3).
- **Edge cases:**
  - E1: Undo purchase within 10 min, one tap; parent refund any time from PA-ECO → equipped item case → GAP-38.
  - E2: Not enough coins for anything → "חוסכים יחד" empty state (09 SCR-08).
  - E3: Shop during child quiet windows/bedtime → "החנות סגורה" (10 W-ECO-06) — which quiet hours? GAP-42.
  - E4: Offline double spend on two devices → negative projected balance hidden as `max(0,coins)`; later earnings fill silently; `econ.overdraft` telemetry (11 §6.5); savings bonus on negative → GAP-10.
  - E5: Item "unlocks at level 12" shown as silhouette map, not pressure (02 §7.3).
  - E6: Shop/base time >30 % of play for 2 weeks → parent suggestion, not an alert (10 W-ECO-05).
  - E7: No transfers between siblings; "give" only to the campfire (02 §7.5).
- **Spec refs:** 00 D3; 02 §7.1–7.5; 09 SCR-08, F3; 10 §6.12; 11 §6.5.
- **E2E sketch:** `E2E-UC-018-a [PW]` clock Sunday 00:05 → showcase ids differ from Saturday; Monday → same as Sunday. `E2E-UC-018-b [SIM]` two devices offline buy band-D item each with 1,000 coins → merged balance negative, UI 0, no `econ.spend` revoked. `E2E-UC-018-c [PW]` no element with text "מבצע"/"מוגבל"/countdown in shop (lint + screenshot).

#### UC-019 Home base: blueprints and Build Missions

- **Actors:** K1 (14×14 grid), K2 (10×10).
- **Happy path:** SCR-10 "view" vs "build" toggle; blueprints bought with coins (fixed prices; K1 18 / K2 10 at launch) → placement with grid snap, 90° rotate, undo; build time 0 (2 s animation); Build Mission from ברגית ("גדר סביב גינה של 5×4") = the learning action (MX-M05/M10), costs 0 coins, creates a unique "project" item, pays like a learning item.
- **Edge cases:** E1: Sibling visiting → may leave a ❤️ only; cannot edit (01 §10.5). E2: Wrong build → structure wobbles gently and returns (09 06d). E3: Two devices place on the same tile offline → GAP-09. E4: K1 "סדנת היוצרים" free mode at 240 friends → no right/wrong, no coins (02 §2.7). E5: Nothing breaks/decays, no attacks (01 §10.5).
- **Spec refs:** 01 §10.5, §10.2; 02 §7.7, §2.7; 09 SCR-10, 06d.
- **E2E sketch:** `E2E-UC-019-a [PW]` Build Mission 5×4 fence: place 18 pieces → T2 + receipt; place 16 → wobble, no receipt, no error sound buzzer. `E2E-UC-019-b [PW]` as K2 visit K1's neighborhood → only ❤️ control present.

#### UC-020 Collection room, species sets and mosaics

- **Happy path:** Opens with ≥3 real rescues from onboarding (endowed, must be active rescues, 02 §5.2 #8); shelf per species with silhouettes; poke friend → says its item; poke silhouette → voiced location hint; near completion → glow + "עוד 3!" and the engine makes last friends appear more often; species set complete → T3 + skill-named badge + **pet accessory** (not evolution; evolution is mastery-only, 02 §5.4) + silver (K1) / wood (K2) chest; regional mosaic (30–40 pieces) complete → diary page + campfire decoration + `secretB` + capsule series.
- **Edge cases:** E1: No time-limited silhouettes; holiday items persist (SCR-12). E2: >60 friends in the hub → instancing, only 30 awake (01 §10.1); 150 friends must stay ≤80 draw calls, ≥55 fps p95 (AC-HUB-3 📱). E3: Every 100 rescued friends → action-titled title + hat (02 §5.4).
- **Spec refs:** 01 §10.3, AC-COLL-1, AC-HUB-3; 02 §5.4–5.6; 09 SCR-12.
- **E2E sketch:** `E2E-UC-020-a [PW]` fixture with species 7/8 → last silhouette glows + "עוד 1!"; rescue it → T3 ≤6 s, badge text contains the skill name, pet accessory slot filled, pet stage unchanged.

#### UC-021 Pets: egg, hatch, care, evolution

- **Happy path:** First egg at game start (K1 chooses 1 of 2; K2 gets his first-subject species); second egg (K1) at level 4 or "גן הגוזלים", whichever is later; evolution strictly by mastery (K1: baby at first `proficient`, young 3 mastered, adult 8, legendary 15 + 3 retained; K2: 1/2/5/9); care actions (feed/play/wash/dress) raise friendship 1–10 (max 3 points/day, only up) and unlock pet emotes; 1 free treat per episode end; ≤3 treats eaten/day, rest stored without decay.
- **Edge cases:** E1: Absence → pet sleeps/plays alone, wakes happy; **never** "התגעגעתי"/sadness (lint). E2: No hunger/sickness/death flags (`needsDecay:false`). E3: No notifications from the pet, ever. E4: K2 sees a ring without numbers; K1 sees "עוד 2 מיומנויות של אותיות" (information, not contract). E5: Holiday costumes are cosmetics, not species.
- **Spec refs:** 00 D4; 02 §8.1–8.3; 01 §4.12; 09 SCR-11.
- **E2E sketch:** `E2E-UC-021-a [SIM]` evolution function is pure in mastered-count; 100 care actions change friendship only. `E2E-UC-021-b [PW]` lint pet VO strings against the banned list ("התגעגעתי", "הייתי עצוב").

#### UC-022 Capsule machine (not gacha)

- **Happy path:** Opens at level 4 (K1) / 10 (K2); price 60 / 6 coins; **next capsule always visible** in the window; series of 8 in fixed order, no duplicates; new series when a regional mosaic completes.
- **Edge cases:** E1: Series finished → machine shows "סדרה חדשה כשהפסיפס יושלם" (no purchase possible) ⟂VERIFY copy. E2: Not enough coins → "חסרים לך עוד N".
- **Spec refs:** 00 D2; 02 §5.7.
- **E2E sketch:** `E2E-UC-022-a [PW]` window shows item n; buy → receive item n; window now shows n+1 (deterministic).

#### UC-023 Daily surprise and the mailbox

- **Happy path:** At first entry of the day (max 1/day) one new thing in camp (letter, visiting shadowling favor, new ברגית item, friend on the roof); accumulate when away (3 days → 3 letters), bundle at 7 into "a big letter" (DD-25); **no login streak, no daily login reward**.
- **Edge cases:** E1: Shabbat-soft → surprises don't open on Shabbat; open cumulatively after (01 §13.1 analog) ⟂VERIFY. E2: Viewer reactions and appreciation letters (UC-063, UC-008) arrive in the same mailbox.
- **Spec refs:** 01 §5.5, DD-25; 10 §6.11 W-RWD-04.
- **E2E sketch:** `E2E-UC-023-a [PW]` clock +3 days → mailbox badge "3"; +8 days → one "big letter" containing 7.

#### UC-024 Movement moments (MV-*), seated alternative, quiet mode, safety card

- **Actors:** K2/K1 (K3 "copy me" with parent).
- **Preconditions:** One-time parent safety setup behind the gate (2×2 m, stand 2–3 m away, grippy socks, camera explanation, "מצב שכנים").
- **Happy path:** Once per level (K2 ≤45 s, ≤2:00/episode) / 1–2 per K1 episode (≤60 s) after a reading task: `SafetyCard` 5 s → child taps "השטח פנוי!" (K3: parent) → character demos → "יאללה!" → count → "פריז!" → cooldown breathing balloon 30–60 s → calm retrieval task. Phase 1 sensing: self-report, iPhone tilt/shake (MV-07), mic onset for claps (MV-02, no recording).
- **Alternative paths:** `profile.movementMode ∈ {full, seated, tapOnly}` and a per-moment "sit" button; switching mid-moment loses nothing (AC-MV-1).
- **Edge cases:**
  - E1: Device held in hand when a jump is requested (DeviceMotion "holding") → "את המכשיר מניחים על המעמד" (neutral), moment waits (09 SCR-19).
  - E2: Quiet mode / after 20:00 / Shabbat-on / "apartment" → no jumping moments (AC-MV-2).
  - E3: iPhone → only in-hand mechanics; never jump with phone (AC-MV-3).
  - E4: Self-report gives no coins; moment never blocks progress (01 §8.2).
  - E5: Camera pose (Phase 2) → per-child parent opt-in, visible camera-on indicator, no frames stored (11 §4.17).
  - E6: Max 90 s continuous movement per activity (11 §4.17).
- **Spec refs:** 00 G6; 01 §8.1–8.2, AC-MV-1…4; 09 SCR-19, §6.10, UX-AC-12; 11 §4.17.
- **E2E sketch:** `E2E-UC-024-a [PW]` clock 20:30 → MV-03 replaced by seated variant; `window.__olam.motion.feed(holdingSamples)` during MV-03 request → stand prompt shown. `E2E-UC-024-b [SIM]` every `MovementActivity` has `seatedVariant` and `tapVariant` (UX-AC-12).

#### UC-025 English island (R-08 Sunny Harbor): enter, play, gloss, exit, length adaptation

- **Actors:** K1/K2/K3, Sunny (English only).
- **Happy path:** Rope bridge "where the language changes" → `sfx.english.enter` (3 ukulele notes) → page slides from the left, `palette.english`, `type.en.*` (Andika), `<Dir value="ltr">` wrapper; Sunny's boat docks left; fixed buttons "?בעברית", "Again, please", exit in the same place; mechanics MX-E01…E04 (+E05–E07 proposed); exit → `BYE` song, reverse jingle, back to RTL shell. K2 gets the EN→HE→EN sandwich; K1 islands grow (60–90 s in Nov → 4–5 min in June).
- **Adaptation:** island lengthens one step when, over the last 5 islands, "?בעברית" <20 % of turns and accuracy ≥80 %; shortens when help >40 % (`SIG.GLOSS_DEPEND`) or the child exits mid-island twice.
- **Edge cases:**
  - E1: Hebrew narration must never play inside the island (AC-MX-3 `englishContext=true`) → daily-cap warning from ניצי mid-island → GAP-30.
  - E2: Child answers in Hebrew → counted as comprehension success; Sunny recasts in English, never "Say it in English" (05 PP-9).
  - E3: Phonics sounds only from human recordings; TTS never for phonemes (00 E9).
  - E4: K2 reaches G0 gloss level → "?בעברית" button disappears (05 §10.5).
  - E5: Mr. Mix-Up makes the deliberate mistakes, never Sunny (05 §10.6).
  - E6: Playback rate 0.85× default; K1 slow-down button 0.75× (05 §10.6).
  - E7: Playwright matrix includes LTR screens + mixed bidi strings (11 §12.2).
- **Spec refs:** 00 B2; 05 §4, §10.2–10.6, PP-1…9; 01 §4.3, AC-MX-3; 11 §4.1 `english/[islandId]`, §12.2.
- **E2E sketch:** `E2E-UC-025-a [PW]` enter island → `dir=ltr` on the island root, progress bar fills left→right, RTL shell nav hidden; exit → `dir=rtl`. `E2E-UC-025-b [SIM]` 5 islands with gloss 10 % + acc 85 % → length step +1; gloss 45 % → −1. `E2E-UC-025-c [SIM]` audio-log assertion: no `he-*` VO file requested while `englishContext=true`.

#### UC-026 "Your turn!" speaking, record-and-compare, pronunciation policy

- **Happy path:** Sunny "Your turn!" → child says the word → (K1/K2) record & compare: Sunny then the child side by side; K2: parent taps "אמר!"; K1: taps 👍/🤔; data only to F-FAM; recordings deleted at session end unless the parent saves a "family moment".
- **Edge cases:** E1: No production score ever (PP-1); perception items (T09) do get scored, with mouth cards after an error and no fail sound (PP-2). E2: ASR for English not in v1; phase 2 only K1, parent present, limited word set, result not stored (PP-5). E3: Mic permission denied → mechanic falls back to "say it together" without recording ⟂VERIFY copy. E4: Accent features (guttural r, weak h) never corrected (PP-8).
- **Spec refs:** 05 §13 PP-1…PP-9; 00 E9.
- **E2E sketch:** `E2E-UC-026-a [PW]` after "Your turn" flow, no `item.answered{correct:false}` emitted for production; file system contains no audio after `session.ended`.

#### UC-027 The runner "מרדף הגחליליות" (MX-R01)

- **Happy path:** Shadowling grabs a friend → auto-run 3 lanes; K2 tap=jump (coyote 150 ms), K1 tap=jump + swipe=lane; answer gates every 8–12 s; wrong lane = 1.5 s detour; fixed length K2 60–90 s / K1 90–120 s; always ends at the destination; collision = 0.5 s stumble + 1.5 s slow; power-ups earned by bonus gates, never bought; K1 ghost of her own best (opt-in, shown to her only).
- **Edge cases:** E1: No death/revive/endless (AC-RUN-1). E2: Co-op: K1 navigator chooses lanes for both; K3 cheer meter → 3 s magnet (01 §12.3). E3: Reduce Motion → slower camera, no bob (09 §5.5). E4: iPhone → landscape locked (09 §8.1).
- **Spec refs:** 01 §9.4, §12.3, AC-RUN-1; 09 06f.
- **E2E sketch:** `E2E-UC-027-a [SIM]` runner state machine: from any state, `t ≥ maxLen` → `arrived`; no `dead` state exists. `E2E-UC-027-b [PW]` K2 runner with 100 % wrong gates ends within 90 s with the friend returned.

#### UC-028 Evening campfire defense (solo, inside the episode)

- **Happy path:** Dusk palette (luminance ≥0.28; K3 ≥0.45, AC-WORLD-3); shadowlings emerge slowly at the sides, never toward the camera; K2: 1 wave 4–5 (1:20); K1: 2 waves 4–6 (2:00) + 2 light towers (2 items each, auto-light every 6 s); a lit shadowling becomes a friend; a shadowling reaching a lantern takes it giggling — lantern returns next evening or by an easy chase; worst case "השמש זרחה והם הלכו לישון"; K1 flame modifier (blue slow / green double friends / gold big treasure + Stretch / purple surprise among predefined rules).
- **Edge cases:** E1: K3 "tuck-in" only sleepy shadowlings, no approach, no stealing (AC-EVE-1). E2: Family campfire level widens the lit zone (01 §10.4). E3: Quiet day → no boss; evening calm music, T0/T1 only. E4: Player has no HP; wrong answers subtract nothing (AC-BTL-1).
- **Spec refs:** 01 §9.1, §9.5, AC-BTL-1/2, AC-EVE-1, AC-WORLD-3; 09 06e.
- **E2E sketch:** `E2E-UC-028-a [PW]` 20 evening screenshots per region → mean relative luminance ≥0.28; shadowling paths satisfy `z ≥ fire.z − 2` tiles via `window.__olam.world.entities()` ⟂VERIFY hook.

#### UC-029 Weekly family boss ("ערב מדורה משפחתי", Thursday finale)

- **Actors:** All children present, parent (optional "Camp Elder"), system.
- **Happy path:** From Sunday the boss sits at the camp edge with a challenge banner; each child's evening segments and level rescues lower the muddle meter (`bossHP = Σ plannedEpisodes×300×0.8`, 300/episode regardless of age); Thursday (parent-chosen day; finale available Thursday 17:00 → Friday noon, DD-06) joint finale 6–8 min (CO-2 same device or CO-3) → finale damage = remaining HP → boss "becomes a friend" T4 (≤10 s, ≤1/week) → prism family treasure; +1 fire level if the weekly goal is also met; K1 +2 gems; boss wanders camp as a VIP that week.
- **Alternative paths:** Parent recorded clue opens the finale; parent "assist" role gives light to the weakest.
- **Edge cases:**
  - E1: Family under ~80 % of planned episodes → boss "falls asleep" with the meter frozen and carries over; no loss, no "the boss beat you" (AC-BOSS-1/2).
  - E2: One child absent at the finale → "finale always finishes the boss if all participants took part" ambiguous → GAP-63.
  - E3: Thursday is a holiday eve/quiet day → no boss that day; finale window shifts ⟂VERIFY rule (GAP-63).
  - E4: Finale on Friday after candle lighting → blocked by Shabbat mode (counters stop; `shabbat.full` sleeps).
- **Spec refs:** 01 §9.3, §12.2, DD-06, AC-BOSS-1/2; 02 §6.2 (weekly boss = silver + 1 gem for mastered… note: 01 says 2 gems; 02 §6.2 "בוס שבועי" silver +1 gem → GAP-14b), §7.6.
- **E2E sketch:** `E2E-UC-029-a [SIM]` AC-BOSS-1 simulation at 80 % and 50 %. `E2E-UC-029-b [PW]` finale with 2 children on one iPad → shared meter only; no per-child score element (AC-CO-4).

#### UC-030 Same-device co-op (CO-1 face-to-face, CO-2 side-by-side), drop-in/out, fairness

- **Happy path:** Hub (night) → campfire icon → SCR-13 lobby → choose children + roles (K3 Lightkeeper tap-only, K2 Scout-Hero sees glints, K1 Builder-Commander reads clues) → touch zones by `absoluteX` with separate RTL HUD per half (CO-1 rotates the far half 180°) → 5–8 min defense → shared meter only → "הצלחנו ביחד" `tier.family` → lullaby; `coop.joined`/`coop.contribution` events per child; co-op minutes count **50 %** toward each child's cap.
- **Fairness:** `lightPerCorrect = 100 / baselineCorrectPerMin` (median last 5 sessions; defaults K3 6, K2 4, K1 5); hinted 70 %; wrong 0; K3 taps rate-limited 1/0.8 s; synergy moment within 10 s; share within ±25 % of even in ≥80 % sessions else closed-loop `coop.balance.k=0.2`.
- **Edge cases:**
  - E1: A child leaves → replaced by a scripted ניצוץ within ≤1 s; joins → role within ≤3 s (AC-CO-3).
  - E2: K3 absent → fire lit automatically at base level (01 §12.2).
  - E3: Both want the same device → turn ring, or CO-2 suggestion (DD-08).
  - E4: Same friend rescued by two siblings → each gets a copy (DD-23).
  - E5: Co-op session appears in both children's PA-LOG as "משותף עם {name}" without the other's data (10 §6.5); parent sees only "who took part" + positive behaviors (DD-21).
  - E6: K3 in co-op still needs parent-present gate to join ⟂VERIFY (lobby must request the gate when K3 is selected).
- **Spec refs:** 00 C6 G5; 01 §12.1–12.5, AC-CO-1…6; 11 §10.1; 09 SCR-13; 10 §6.5.
- **E2E sketch:** `E2E-UC-030-a [PW]` CO-1 screenshot: two HUDs, far one rotated; touches at x<50 % routed to zone A only. `E2E-UC-030-b [SIM]` fairness simulation 1,000 sessions → ≥80 % within ±25 %. `E2E-UC-030-c [SIM]` cap accounting: 10 co-op minutes add 5 to each participant.

#### UC-031 Multi-device co-op (CO-3, `coop.multiDevice` flag, Durable Object room)

- **Happy path:** Device A `POST /coop/rooms` → joinCode; device B `POST /coop/rooms/join` (same family) → `wsTicket`; both connect `wss://…/v1/coop/rooms/:id/ws?ticket=…`; `hello` → `welcome{state}`; poses 10 Hz; learning answers evaluated on the client, `action{correct, power}` applied by the DO (idempotent by `action.id`, last 500); room ends when all leave / 2 min idle / 30 min cap → `coop_rooms.summary_json`.
- **Edge cases:**
  - E1: Socket drops >10 s → client continues in "solo co-op" and merges contributions as events afterwards; no loss (11 §10.2) → interaction with DO summary → GAP-21.
  - E2: Reconnect with `hello{lastTick}` → full `welcome`.
  - E3: Rate limit >20 msg/s → `ack{applied:false, reason:'rate'}`.
  - E4: Flag off or kill switch `coop` → lobby shows same-device only.
  - E5: Emotes are the only sibling channel (no chat, 00 C7); taunting emotes don't exist (01 §10.6).
  - E6: K3 on a parent's `parent_only` phone joins as Lightkeeper (11 §5.5 "parent-present mode").
- **Spec refs:** 11 §10.2, §7.11, §4.12; 01 §12.1 CO-3, §10.6; 00 C7.
- **E2E sketch:** `E2E-UC-031-a [API]` miniflare DO: two sockets, kill one 12 s, reconnect → `welcome.tick` ≥ last; duplicate `action.id` → `reason:'duplicate'`. `E2E-UC-031-b [PW]` two contexts in one room → shared meter equal on both within 1 tick.

#### UC-032 K1 guide mode ("מצב מדריכה")

- **Happy path:** Opt-in per session ("do you want to be your brother's guide today?"), ≤1/day, ≤10 min; K1 sees K2's screen + guide strip ("תני לו לנסות קודם" 5 s timer, "שאלי: איך ידעת?", 3 hint tokens); she cannot answer for him, mark him wrong, see his stats or compare; she earns Mentor Track progress only, never coins for his answers; parent sees "היא שאלה 'איך ידעת' 3 פעמים".
- **Edge cases:** E1: PA-CSET "מצב מדריך" off → never offered (10 §6.13). E2: Attempt to tap on K2's items from K1's zone → ignored (AC-CO-5). E3: Mentor mark `mentor` adds an effort step to her next chest (01 §16.2, if 02 adopts).
- **Spec refs:** 00 C6; 01 §12.6, AC-CO-5; 10 §6.13; 02 §6.2.
- **E2E sketch:** `E2E-UC-032-a [PW]` hint token tap within 5 s of item start → disabled; after 5 s → glow on K2's screen; 4th token → none left; 10:00 → session ends gracefully.

#### UC-033 Async "letter to a sibling" (CO-4)

- **Happy path:** K1 builds a riddle/puzzle for K2 from structured parts → appears in K2's next episode → a heart emote returns.
- **Edge cases:** E1: Free text by K1 → must not reach K2 unmoderated → GAP-46 (structured-only authoring). E2: K2 solves it → K1 Mentor Track credit ⟂VERIFY.
- **Spec refs:** 01 §12.1 CO-4; 00 C7.
- **E2E sketch:** `E2E-UC-033-a [PW]` K1 composes from chips only (no text input element present); K2 episode next day shows the letter.

#### UC-034 Turn-taking on one iPad and the "patience heart"

- **Happy path:** Turn ring by last use; instant save on switch; an episode stopped mid-way "for a sibling" gives the yielder a Patience heart and a VIP friend surprise next episode (unexpected, never pre-announced).
- **Edge cases:** E1: Detection rule for a yield missing → GAP-33. E2: Time visible to parents only, never to children (01 §12.5).
- **Spec refs:** 01 §12.5, DD-08; 11 §4.16.
- **E2E sketch:** `E2E-UC-034-a [SIM]` after rule from GAP-33 is adopted: `session.ended{reason:'switch'}` at 40 % + different child starts within 120 s → `turn.yielded` event; next session for the yielder contains `vipFriend` surprise.

#### UC-035 Emotional check-in (RIT-OPEN) and what is (not) stored

- **Actors:** K1 (zone → word of 9 → intensity 1–3 optional), K2 (quadrant then 1 of 3 words), K3 (optional 4 faces, parent-led, nothing stored).
- **Happy path:** ≤20 s at episode start: "איזה מזג אוויר יש בפנים עכשיו?" → zone ☀️⛅⛈️🌧️ → word → companion validates and offers a tool (storm: dragon breath before starting?); "לא רוצה להגיד" always available; only `checkin_done` (no label) and the on-device aggregate counters for RF-01/RF-02 update; `checkin.mood` is `local_only` and never syncs (11 §5.7; 10 AC-32).
- **Edge cases:**
  - E1: Intensity 3 → calming first, "detective" (SE.THINK) deferred (07 §5 timing rule).
  - E2: 😠/😢 adds +2 to the frustration score (08 §6.7).
  - E3: Storm/rain ≥5 of last 7 check-ins on ≥5 days within 14 days → RF-01 `consider` card to the parent (07 §10) — duplicate with 10's proposed `EMO_CHECKIN` (3 of 5 days) → GAP-32.
  - E4: Frequency: 01 says "every other day" for K2; 07 says every episode → GAP-31.
  - E5: No coins ever for check-in (07 AC-13).
  - E6: Check-in label never appears in share cards, reports, Push (07 §8; 10 AC-14/32).
- **Spec refs:** 00 C8; 07 §2.4, §5 SE.VOCAB.2/3, §10 RF-01/02; 01 §5.3; 08 §6.7; 10 §8 PRIV-01, AC-32; 11 §5.7.
- **E2E sketch:** `E2E-UC-035-a [PW]+[API]` choose ⛈️ "מתוסכל" → sync push body contains no event with `type:'checkin.mood'`; local DB has it with `cls:'local_only'`. `E2E-UC-035-b [SIM]` 5 storm days in 7 → RF-01 candidate; 4 → none; "לא רוצה להגיד" ×7 → none.

#### UC-036 AI helper גלגלון/גלגלי: brain token, scripted ladders, K1 live turns, refusals

- **Actors:** K1 (live turns), K2 (scripted trees only), K3 (none), Worker (Haiku 5.5), parent (log).
- **Happy path (K1):** Item error → classified in code → scripted hint ladder; `llm_turn` node opens **only after ≥1 child attempt** ("brain token" lights the robot's bulb) → chips + ≤60-char text → `piiScrub` + `riskLexicon` → `socratic.turn` (numbers masked; ≤20 words; move ∈ allowed) → validators (answerLeak with all numeral forms, genderCheck, bannedContent, praiseLint, live nikud ≤1.5 s) → text + fixed beep (no live TTS) → any failure → scripted `fallback` node, no retry; quotas 12 turns/day, 4/exercise; fixed opener "אני גלגלי. אני מכונה…".
- **Refusals (code before LLM):** answer begging → next hint; off-topic → redirect; personal details → not sent + privacy line + parent flag (no content); distress → not sent + "כדאי לספר לאמא או לאבא עכשיו" + "לקרוא להורה" button + gentle parent flag; security/news → redirect; insults → neutral (flag at 3); "are you alive?" → literacy line.
- **Edge cases:**
  - E1: Offline → scripted hints only, UI "צריך אינטרנט" for the robot (11 §4.8).
  - E2: Kill switch `ai` or daily $ cap >100 % → scripted only (12 §1.9).
  - E3: Timeout 6 s → fallback; waiting animation = gears, never "typing…".
  - E4: K2 never gets `llm_turn` nodes (12 §4.2); PA-CSET "מלווה AI" off → scripted only for that child.
  - E5: Homework-mode game → the helper receives only the clone item (12 §3.10).
  - E6: AL-10 "AI safety" Push always (high) even in quiet hours (10 Q-5).
  - E7: Every turn logged to `ai_turns` 90 days with "machine label shown" verification (10 AC-33).
- **Spec refs:** 00 C7 E8; 12 §1.3, §4.1–4.8; 07 §2.3; 10 §6.15, §10.1 AL-10, AC-33; 11 §4.8, §7.7, §8.4.
- **E2E sketch:** `E2E-UC-036-a [API]` mock Haiku returning "התשובה היא 12" → validator `answerLeak` fails → response is the scripted fallback node id; `ai_turns.used_fallback=1`. `E2E-UC-036-b [PW]` K1 types "אני גרה ברחוב הרצל 5" → request to `/ai/companion` never fires (route spy), privacy line shown, parent inbox gets AL-10 low. `E2E-UC-036-c [PW]` K2 item with 2 errors → hint nodes only; no text input exists.

#### UC-037 AI-literacy sandbox activities (pre-generated, combinatorial)

- **Happy path:** K2: prompt potion (300 approved images; unapproved cell → chip disabled), find the odd one (40); K1: better question (54 answers), role hats, AI detective (30 fact sets, all hand-checked), only-people-know, mission control / bug hunt, next word, camera trainer (on-device only). Credit: coins at mastery/effort events (07 §2.6), gem at branch end.
- **Edge cases:** E1: An image rejected at gate G5 → combination unavailable, never a blank (12 §4.5). E2: "Family AI charter" (AI.RESP.3) output goes to the parent card, not the LLM. E3: Child "bell" (AI.RESP.1) → RF-06 immediate parent info + content review (07 §10).
- **Spec refs:** 07 §3 (AI.*), §2.6; 12 §4.5; 01 MX-A03.
- **E2E sketch:** `E2E-UC-037-a [PW]` prompt potion: pick 5 chips → image id = deterministic hash of the tuple; disabled chip for a known-rejected tuple.

#### UC-038 "מוזר לי" button and the "something strange or scary" bell

- **Happy path:** Small ? icon on any generated item → item `quarantined` immediately, replaced by another from the same family, appears in the parent queue with reason `kid_flag` (12 §1.8). Bell (AI.RESP.1) on any screen → RF-06 card "{name} לחץ/ה על הפעמון במסך [מסך]" + screen marked for content review.
- **Edge cases:** E1: Not in PA-APPR types list (A1–A6) → GAP-47. E2: Offline → quarantine stored locally, synced later; replacement from local pack.
- **Spec refs:** 12 §1.3.2 #5, §1.8; 07 §3 AI.RESP.1, §10 RF-06; 10 §6.10.
- **E2E sketch:** `E2E-UC-038-a [PW]` tap "מוזר לי" → next item id ≠ flagged; `review_queue` row `reason:'kid_flag'` after sync.

#### UC-039 End of session: wind-down, goodnight ritual, offline mission, blocked re-entry

- **Happy path:** Progress strip "היום במחנה" fills; at 2:00 left ניצי: "עוד הרפתקה אחת קטנה ואז הולכים לישון"; goodnight ritual 30–45 s (friends gather, yawn, ניצי dims, child covers the פנסון, 4-note lullaby, offline mission card, camp-asleep picture); `session.ended{reason:'natural'}`; after goodnight: no "play again" for K2/K3 (re-entry via gate); K1 may open the collection room (view only) and the music hut; approved exit lines only.
- **Edge cases:** E1: Offline mission "done" marked by the parent (W-HOME-09) or not at all; nothing lost; wood chest for a confirmed mission (02 §6.2) → anti-grind cap → GAP-37. E2: Printable mission → AirPrint ⟂VERIFY `expo-print` (GAP-50). E3: Bedtime story optional ending (UC-042) inside the episode's time. E4: K1 "view-only camp" semantics → GAP-43. E5: Forbidden exit lines lint (AC-STOP-3).
- **Spec refs:** 00 C3 C5; 01 §15.1–15.2, AC-STOP-1…3; 09 SCR-07 WindDownCard, SCR-20; 10 W-HOME-09; 02 §6.2.
- **E2E sketch:** `E2E-UC-039-a [PW]` after wind-down as K2: picker tile tap → gate sheet, no episode; as K1 → camp-asleep view with collection/music only, no `session.started{mode:'solo'}` with level routes reachable. `E2E-UC-039-b [SIM]` lint of exit lines.

#### UC-040 Daily cap reached mid-level, parent +10 bonus, bedtime approaching

- **Happy path:** Cap (K3 10/15 ⟂, K2 25, K1 40 min; weekday vs Fri–Sat separately) → never cut mid-beat: finish the beat (≤1:30 overtime) and "land" in Ketsu; if >1:30 remains, level state saved to the checkpoint for tomorrow; then goodnight ritual; `session.ended{reason:'cap'}`.
- **Alternative paths:** Parent grants +10 min through the gate ("bonus" button not shown to the child, DD-09) → GAP-02 (no event/API).
- **Edge cases:**
  - E1: Cross-device cap: local events + server-merged total; offline overrun tolerated, never punished (11 §4.16).
  - E2: Cap hit on 5 of 7 days → AL-06 report-only.
  - E3: 60-min pre-bedtime rule (10 §6.13) → GAP-04 precise semantics.
  - E4: Bedtime schedule hits mid-level → 2-min warning "הגיבור הולך לישון", ends at a natural point, soft animation, nothing startling; `reason:'bedtime'` (10 FAM-04; AC-21).
  - E5: Day boundary for the cap (midnight vs. 04:00) undefined → GAP-44.
- **Spec refs:** 00 C3; 01 §15.1, DD-09; 08 §6.10; 10 §6.13, §7.2 FAM-04, AC-21; 11 §4.16, §5.5.
- **E2E sketch:** `E2E-UC-040-a [PW]` clock so that cap ends 40 s into Shō → beat completes, Ketsu shortcut, ritual; overtime ≤90 s; `session.ended.reason='cap'`. `E2E-UC-040-b [PW]` cap ends with 3:00 of level left → level saved; next day resumes at the checkpoint.

#### UC-041 Safe room ("החדר המוגן", B8)

- **Happy path:** Reachable from the picker (soft "home" icon, **no gate**) and from pause; works offline and in the dark (dim `dusk` palette, max brightness `paper.50` at 85 %); breathing games, 3 recorded short stories, familiar routines; giant K3-size buttons; no coins, scores or sharp sounds; `mus.safeRoom`.
- **Edge cases:** E1: During `shabbat.full`/bedtime → still allowed per setting (09 §6.10). E2: No siren-like SFX anywhere (AC-SHAB-2 spectral check). E3: Time in the safe room not counted toward caps ⟂VERIFY (GAP-19b).
- **Spec refs:** 00 B8; 09 SCR-16, §3.4 dusk, §6.4, §6.10, SCR-18; 01 AC-SHAB-2.
- **E2E sketch:** `E2E-UC-041-a [PW]` offline + bedtime screen → safe room reachable without the gate; no `econ.*` events; brightness of any frame ≤ threshold.

#### UC-042 Bedtime story (optional ending)

- **Happy path:** From "תיאטרון הסיפורים" (120 K2 / 75... per 01 §10.2 thresholds; 02 stretches), a 1–2 min pre-generated story from the day's friends and words, parent-previewed (A? manual story approval, 10 §6.10 A2 "ידני לסיפורים"); dark UI, no rewards, only "לילה טוב".
- **Edge cases:** E1: Unapproved story → static pack fallback (DD-14). E2: Story contains the child's name → injected locally, never in the Batch prompt (00 item 12; 12 §1.3).
- **Spec refs:** 01 §15.5, §10.2, DD-14; 12 §2.5, §1.8; 10 §6.10.
- **E2E sketch:** `E2E-UC-042-a [PW]` story screen has exactly one button; no coin/XP event emitted.

#### UC-043 Season track ("אלבום ההרפתקה"), tiers, overflow, never expires

- **Happy path:** XP from every learning action feeds level and the active track (K1 30 tiers, K2 20, K3 6-sticker page); tier XP constant within a season (K1 clamp 250–450); rewards per 02 §9.4; completed → overflow to the oldest unfinished season; all done → bonus tier every 800 XP; past seasons in the album, selectable as active; no countdown, no premium.
- **Edge cases:** E1: New child mid-season starts at tier 1 of the current season (02 §9.3). E2: Season certificate printed for home (01 §5.1) → print path GAP-50. E3: Campfire level must **not** reset per season; a new color ring per season instead (02 §7.6 [D]; review brief item 1).
- **Spec refs:** 00 D4 D5; 02 §9.1–9.4, §7.6; 01 §5.1, §13.
- **E2E sketch:** `E2E-UC-043-a [SIM]` XP overflow routing; seasons table matches 01 §13.2 dates.

#### UC-044 Seasonal events (Hanukkah first night on Shabbat eve, Purim, Pesach, Yom HaAtzmaut)

- **Happy path:** `calendar.events` config; event opens on its date with 1–3 event levels, 1 campfire decoration, 1 sticker/mosaic set, 1 offline mission/day, optional family evening; event treasure (prism) after ≥60 % of event levels, contents known in advance; after the event everything moves to "תיבת החגים" forever; holiday items join the catalog permanently; `matchedSinkRequired`.
- **Edge cases:**
  - E1: EV-01 night 1 = Friday 2026-12-04 → unlocks Saturday night together with night 2 (cumulative) (01 §13.1–13.2, AC-EV-3).
  - E2: Content not ready → postponed in config, appears as a "late gift" (DD-16).
  - E3: No fireworks/explosions on Yom HaAtzmaut; lantern parade (01 §13.2, 00 B8).
  - E4: Pesach/summer "extra episode allowed" parent setting (01 §13.2 EV-05, S5).
  - E5: Yom Yerushalayim / Tu B'Av: neutral, no event (01 §13.2 note).
- **Spec refs:** 00 B7 B8 D3; 01 §13.1–13.3, DD-16, AC-EV-1…3; 02 §10.3.
- **E2E sketch:** `E2E-UC-044-a [SIM]` `FX-CAL-5787`: 2026-12-04 19:00 Tel Aviv → EV-01 closed; 2026-12-05 18:30 → nights 1+2 open. `E2E-UC-044-b [PW]` 2027-01-30 → EV-02 levels present in the holiday box.

#### UC-045 Shabbat (soft / full / off), holidays and quiet days

- **Happy path (`shabbat.soft`, default):** From candle lighting (city-based: Tel Aviv 18/20 min, Jerusalem 40 — 01 §15.4 vs 10 SET-01: 18 vs 20 → GAP-14c) to havdalah (+40 min configurable): app usable as free practice; all counters stop; no coins/gems earned; no events open/close; no notifications; Shabbat scenery; SCR-20 for locked parts.
- **`shabbat.full`:** app sleeps (Shabbat-shalom picture), parent can open via the gate as an exception. **`shabbat.off`:** regular.
- **Quiet days (QD-1…4):** no events/boss; T0/T1 only; calm music; no siren-like SFX; chests shelved; alerts sent without celebratory emoji/"!"; AL-01 moved to the next day.
- **Edge cases:** E1: Weekly goal `availableDays` excludes Shabbat/Yom Tov/parent rest days; a week with 0 available days = "שבוע מנוחה" (02 §10.1). E2: Review ladder ignores rest days (`elapsedActiveDays`, 08 §5.3). E3: Playing on Shabbat when allowed counts as bonus, not required (02 §10.1). E4: Shabbat starts mid-episode → episode finishes normally, receipts 0 coins afterward ⟂VERIFY (GAP-14d). E5: Device clock/timezone abroad → GAP-39.
- **Spec refs:** 00 B7 B8; 01 §15.4, §13.1, AC-SHAB-1/2; 02 §10.3; 08 §5.3, §6.10; 09 SCR-20, §6.10; 10 SET-01, §10.2 Q-2/Q-4.
- **E2E sketch:** `E2E-UC-045-a [SIM]` AC-SHAB-1 time simulation Tel Aviv + Jerusalem for all 5787 Shabbatot: zero counter deltas, zero `econ.*`. `E2E-UC-045-b [PW]` QD-2 date: evening has no boss banner; SFX manifest played ∩ siren-list = ∅.

#### UC-046 Weekly family goal and the campfire

- **Happy path:** Week Sun 00:00 → Sat 23:59 Asia/Jerusalem; targets K1 3, K2 3, K3 2 adventure days; `effTarget = min(target, ceil(availableDays×0.6))`; `overshootAllowance=1`; "logs for the fire" fill in camp (sum only, no per-child, no days-left); met → fire flares, K1/K2 gold chest (K1 +2 gems), K3 envelope, +20 logs, parent gets a reward suggestion (≤1 per 14 days); not met → nothing happens; fire level +1 (never resets) only via goal/boss/event.
- **Edge cases:** E1: Parent removes K3 from the goal → target 0. E2: Sick day declared → excluded from `availableDays` (SET-03). E3: Co-op episode counts for every participant. E4: Give-jar donations buy decorations/projects only; level is never purchasable (02 §7.6).
- **Spec refs:** 00 C5 D4; 02 §10.1–10.3, §7.6; 01 §10.4; 10 W-HOME-01, AL-14.
- **E2E sketch:** `E2E-UC-046-a [SIM]` property tests on `effTarget`/`progress` incl. a week with Rosh Hashana (2 Yom Tov + Shabbat). `E2E-UC-046-b [PW]` PA-HOME shows "{x} מתוך {goal} עצים" and no per-child attribution (AC-01).

#### UC-047 Monthly boss and benchmark windows (BOY/MOY/EOY)

- **Happy path:** Monthly boss per subject when ≥28 days passed or ≥3 skills reached `proficient`, not on rest days; K2 6 / K1 10 items, 50 % delayed checks, no hints ("קשה לי" allowed); boss always retreats; win → gem; drift anchor `mean(x−P) < −0.15` → `u += 0.2`. Benchmark windows BOY 15.10–30.11, MOY 15.1–28.2 (K1 adds unpointed text), EOY 15.5–15.6; AL-12 report-only when a window opens; PA-ASM shows status vs. plan only when SE ≤ 0.5.
- **Edge cases:** E1: Benchmarks pay nothing (02 §2.7). E2: K1 read-aloud ASR only parent-present, never punitive (00 E9; AC-MX-5). E3: W-ASM-06 paper probe PDF + parent input (C10).
- **Spec refs:** 08 §10.2–10.3; 10 §6.8, AL-12; 02 §2.7; 01 AC-MX-5.
- **E2E sketch:** `E2E-UC-047-a [SIM]` boss composition shares; retreat in all terminal states. `E2E-UC-047-b [PW]` boss screen never renders a score.

#### UC-048 Return after a long break (≥21 days) and the skip challenge

- **Happy path:** `daysSinceLastActive ≥ 21` or a school break ≥14 days without play → first episode `mode:'reentry'`: 5-item warm-up per active axis (≤2 axes), `u += gap/60`, reviews ≤30 %, no demotion from this alone; AL-05 "long break" (once per break, ≤1 per 21 days, excludes Shabbat/holidays/sick/paused) with a no-guilt tone; the pet wakes happy.
- **Skip challenge:** axis with >95 % first-try over 14 days (≥30 items) → upward placement-style probe (max 8) → passing skills `proficient` with `provenance:'skip_challenge'`, delayed check still required; parent "כבר יודע/ת את זה" triggers the same.
- **Edge cases:** E1: Daily surprises accumulated (UC-023). E2: Downward check after <60 % for 7 days (08 §10.4).
- **Spec refs:** 08 §10.4; 10 §10.1 AL-05, §6.3 W-MAP-04, §6.8 W-ASM-07; 02 §8.3.
- **E2E sketch:** `E2E-UC-048-a [SIM]` 25-day gap → first plan `mode:'reentry'`, review share ≤0.3, no level change events.

#### UC-049 School year rollover (1 Sept 2027) and S6

- **Happy path:** FAM-03 shows "עלה/תה כיתה?" on 1 Sept; parent confirms; K2 → grade 1 ("Explorer+", more pointed text), K1 → grade 4, K3 → gan 3.5 (lap → "co-play-first" with short solo periods); a light play-based re-placement runs (DD-18); S6 season; content prepared for 5788 holidays.
- **Edge cases:** E1: Parent doesn't confirm → engine keeps the old grade; banner persists ⟂VERIFY. E2: K3 leaving lap-only means picker rules change (parent-present requirement relaxed) → GAP-11b (rule for "short solo periods"). E3: Curriculum mapping tags "טרם אומת" (10 §6.3).
- **Spec refs:** 01 §13.2 S6, DD-18; 10 §7.2 FAM-03, §6.3; 08 §10.3.
- **E2E sketch:** `E2E-UC-049-a [PW]` clock 2027-09-01 → FAM-03 banner; confirm → `child.updated` audit diff grade 3→4; next K1 entry runs a reduced placement.

### 2.C Parent area (app and web)

#### UC-050 Parent gate: hidden corner, Face ID, PIN cooldown, L mode

- **Actors:** Parent (or any adult), K1 trying to get in.
- **Happy path:** Long-press 2 s on the small parent icon (picker corner / hub) → GATE-01 → Face ID (if `capabilities.faceId` and the device is not flagged "kids know the passcode") → on success: guardian session present → Keychain refresh → JWT → `(parent)` routes; no session → **L mode** (device-local settings, lock/unlock this device, reports of this device's children) with a one-time "התחברות הורה במכשיר זה" to upgrade; after Face ID success, a guardian picker so the audit knows who entered; parent area locks after 3 min idle or on background; leaving returns to the **picker**, never to the last child.
- **PIN path:** 6 digits verified locally against synced scrypt hashes; 5 failures → 60 s wait, doubling; ≥5 consecutive failures audit-logged; no arithmetic gate (00 E7).
- **Edge cases:** E1: Shared iPad flagged at ON-06 → PIN only (SEC-02; AC-17). E2: `shared_kids` → PA-FLAG / PA-AI / PA-STR narrative blurred until "הצגה"; blur overlay on `inactive` for the app switcher (SEC-07). E3: Offline → gate works (hash synced). E4: Step-up re-auth for export/delete/roles/unpair/wipe/PIN change (SEC-06). E5: `parent_only` iPhone → Face ID every open or after 15 min by choice (SEC-05). E6: L-mode caps edits vs. per-child caps across devices → GAP-22.
- **Spec refs:** 00 E7 H6; 10 §9 SEC-01…07, RB-0, AC-17; 11 §5.3 S4, §5.4 (L), §4.1 nav rules.
- **E2E sketch:** `E2E-UC-050-a [PW]` 5 wrong PINs → pad disabled 60 s (clock-driven), 6th wrong after wait → 120 s; audit row `gate.failed_burst`. `E2E-UC-050-b [PW]` exit parent area → route is `/picker`. `E2E-UC-050-c [MAE]` 11 §12.4 row 10.

#### UC-051 PA-HOME browsing and "tonight" actions

- **Happy path:** Within 10 s: W-HOME-01 campfire (sum only), 3 child cards in a weekly-rotating order (seed = isoWeek) with one status line (priority selector §6.1.1) and a "הערב" chip, W-HOME-03 tonight carousel (5-min activity + context icon + why; "עשינו ✔" / "החלף רעיון"), approvals banner only if >0, last 2 alerts, upcoming (tests, holidays, benchmark windows, back from break), tip of the week, real-world missions awaiting confirmation (no reject button; "עוד לא, ננסה מחר"), health dot only on a fault.
- **Edge cases:** E1: ES-0 before placement; ES-1 "<5 activities"; ES-2 rest week. E2: Device filter is secondary and never changes mastery (AC-22). E3: iPad ≤300 ms from local DB with 90 days × 3 children (AC-30). E4: Never two children with a numeric metric in one component (AC-01 render-tree test). E5: No red for a child's performance (AC-07).
- **Spec refs:** 10 §6.1, §3.2, §5.4, AC-01, AC-02, AC-07, AC-22, AC-30, AC-31; 09 §7.4 parent components.
- **E2E sketch:** `E2E-UC-051-a [PW]` isoWeek 46 vs 47 → card order differs; render-tree scan: no component with one `metric` and two `childId`. `E2E-UC-051-b [PW]` "עשינו ✔" → `tonight.done` event; parent-engagement metric increments.

#### UC-052 Child overview, skill map, skill detail and "correct us" actions

- **Happy path:** PA-CHILD (period selector; device filter) → PA-MAP tabs (math, Hebrew, English (K1; K2 oral only), world & science, AI & thinking, SEL "מה פגשנו" only) with views game map / Ministry plan (source + verification tags) / by months (Sept→June right-to-left) / graph (P2) → tile → PA-SKILL (parent-language name, example item, stage chip, history, accuracy vs. target band, error clusters with the items exactly as shown, hints & persistence, next review, 2 off-screen activities) → actions: "נלמד בכיתה", "להעדיף עכשיו", "לנעול לבינתיים", "כבר יודע/ת" (skip challenge, not direct mastery), "לא רלוונטי", W-SK-10 "תקנו אותנו" (knows outside / interface issue → item type switched to tap / taught elsewhere) → `POST /children/:id/skills/:skillId/actions` (doc LWW) + audit.
- **Edge cases:** E1: <8 items → "עוד מוקדם" (AC-04/05). E2: Stage comes from the engine `LevelId`, never recomputed; R score hidden <8 obs, capped 79/89 (AC-05). E3: Reordering "next up" that violates a prerequisite → "קודם צריך את {prereq}" (W-MAP-05). E4: Raw events older than 24 months → "נשמרו רק הסיכומים החודשיים". E5: SEL tab never shows levels (C8).
- **Spec refs:** 10 §6.2–6.4, §4.2 M-04/05/06, AC-04/05/06, API-11/12; 08 §5.1, §10.4; 00 C8 C9.
- **E2E sketch:** `E2E-UC-052-a [PW]` skill with 7 observations → "עוד מוקדם"; 8 → chart. `E2E-UC-052-b [API]` action `skipChallenge` → audit row with `diff_json`; engine plan next session contains a skip-challenge probe (`[SIM]`).

#### UC-053 "Taught this month" input (class topics, test dates)

- **Happy path:** PA-CSET §6.13.1 → subject → topic by (a) skill-node search, (b) free text mapped by Sonnet (`POST /class-topics/map`) with parent confirmation of the nodes, or (c) photo of a notebook page via the photo pipeline → phase `upcoming`/`teaching`/`taught` → optional test date → result text "בשבועיים הקרובים המשחק יעדיף את זה. קודם נסגור את {prereq}" → engine priorities (+0.4/+0.5/×1.5) without overriding the graph; expires after 42 days unless renewed ("עדיין לומדים?"); K2: letter of the week / gan theme as content wrappers.
- **Edge cases:** E1: Test in 2 days → AL-13 Push + spaced retrieval ≤30 % of session; test tomorrow → GAP-72. E2: Mapping returns nodes the child isn't ready for → prerequisites scheduled first, topic never shown directly (08 §6.9). E3: Offline → free-text mapping unavailable; (a) search works. E4: Coverage denominator grows → automatic note (AC-06).
- **Spec refs:** 10 §6.13.1, AL-13, AC-06, API-23; 08 §6.9; 03 §7; 04 §13; 05 §15.
- **E2E sketch:** `E2E-UC-053-a [SIM]` `teaching` on a skill with an unready prerequisite → next 3 plans contain the prerequisite, not the skill. `E2E-UC-053-b [PW]` set then advance clock 43 days → topic shown as expired with "עדיין לומדים?".

#### UC-054 Photo-to-game: printed worksheet (happy path)

- **Actors:** Parent (gate), Worker (Sonnet 5.5 vision; Opus in parallel for handwriting), child.
- **Happy path:** SCR-14 capture (doc scanner, ≤4 pages) → crop (+ up to 3 exclusion rectangles) → on-device face blur (VisionKit) + optional name blur + orientation bake + EXIF strip + resize 2,576 px (12 §3.3; 11 says 1,600 → GAP-15) → tag source kind / mode / children → `POST /v1/photo-jobs` → R2 private (TTL ≤30 d) → extraction streamed to the confirm screen (≤15 s p50 to review) → parent fixes words, marks the answer key (parent-only), picks mode (UC-055) and children → confirm → template selection (code), distractors (code), Haiku enrichment per child (skip on failure), validators, TTS (server) → pack + sha256 → R2 → devices notified → child's map shows a glowing `secret` node "הפתעה מ{{parent}}" (09 F4) / "משחק חדש מהמחברת!" (12 §3.1) → GAP-16; items enter SRS (+1/+3/+7); raw photo **deleted immediately after approval/rejection** (AC-16: ≤5 min).
- **Edge cases:** E1: Child cannot start a photo flow (11 §9.4). E2: Names of other children replaced with placeholders ("הוחלפו 2 שמות"). E3: First round playable at 4 items with audio while the rest loads (12 §3.14). E4: Same source → different fill per child (K1 spell_builder, K2 first_sound, K3 tap_heard_picture). E5: Push unavailable on free signing → AL-08a via local notification/in-app (GAP-62). E6: "Keep photo for reference" toggle off by default.
- **Spec refs:** 00 E8 E5; 12 §3.1–3.9, §3.12–3.14; 11 §8.3, §9.4; 10 §6.10 A1, U4, AC-16; 09 SCR-14, F4.
- **E2E sketch:** `E2E-UC-054-a [API]` `FX-PHOTO-PRINTED` with mocked vision → job statuses `uploaded→extracting→needs_confirm→building→ready`; after confirm the R2 object is gone within 5 min (lifecycle + worker). `E2E-UC-054-b [PW]` parent review screen: low-confidence items highlighted; "לאשר הכול" enabled only after scrolling the sample to the end (12 §1.8). `E2E-UC-054-c [PW]` child map shows the new node only after approval (never before).

#### UC-055 Photo-to-game: homework mode (anti-cheating)

- **Happy path:** Mode "שיעורי בית להגשה" → child gets **skill clones** only (2 per original, never identical even reordered); originals + answer key visible to the parent only; reading comprehension → `story_sequence`/`true_false_shield` with **different** statements (V12); the AI helper sees only the clone; "הוגש" tap by the parent **or** next day 06:00 → `review` mode unlocks originals into SRS.
- **Edge cases:** E1: `study_list` (dictation words) → items themselves (practice is the goal). E2: Clone generator can't find an `ItemModel` → item falls out of the game, parent sees "לא הצלחנו לשכפל פריט 3" ⟂VERIFY copy. E3: Child never plays practice before 06:00 → review still opens next day (nothing lost).
- **Spec refs:** 12 §3.10, §3.7; 10 §6.10 A1 ("מתי נפתח 'מצב חזרה'").
- **E2E sketch:** `E2E-UC-055-a [SIM]` for 100 math originals: no clone equals an original under commutation; `review` unlock at 06:00 local. `E2E-UC-055-b [PW]` in homework game, hint ladder text never contains an original number.

#### UC-056 Photo-to-game: a child's own story → interactive story

- **Happy path:** Parent photographs K1's written story → on the child's device: "אבא/אמא צילמו את הסיפור שכתבת. רוצה שנהפוך אותו לסיפור במשחק?" → "כן"/"לא עכשיו" ("no" is honored, never re-asked for that story) → parent approval (mode "סיפור שהילד כתב") → sentences extracted verbatim (`authored_text`), silent spelling suggestion shown to the parent only; display "as written" (K1 default) or "corrected" (K2 default, dictated stories) → scenes from the P6 library (no live image generation), K1 gets 1–2 choice points marked "הצעה של הרובוט" that she can reject ("לא, הרעיון שלי!") → cover "סיפור מאת {{name}}" (name injected on the client; parent's recorded name audio if present) → optional share with a sibling (off by default; the author decides).
- **Edge cases:** E1: Spelling practice only from words the parent marked. E2: "Special illustration" (GPT Image from text only) is an offline parent request behind a gate, not in the 60 s path. E3: A5 approval with credit, no public spelling correction (10 §6.10). E4: Comic production (AI.CREATE.2) reuses this flow (07 §3).
- **Spec refs:** 12 §3.11; 10 §6.10 A5; 00 C7; 07 §3 AI.CREATE.2; 01 DD-22.
- **E2E sketch:** `E2E-UC-056-a [PW]` consent "לא עכשיו" → story job `declined_by_child`; reopening the app does not re-prompt. `E2E-UC-056-b [PW]` K1 rejects the robot suggestion → page text equals `authored_text` byte-for-byte.

#### UC-057 Photo-to-game failures and fallbacks

| Case | Expected behavior | Spec |
|---|---|---|
| `not_learning_material` | "לא מצאנו כאן תרגיל. אפשר לצלם שוב?" + tips; photo deleted now | 12 §3.15 |
| Fewer than a template's minimum | "נמצאו רק 2 מילים — להוסיף עוד מהמילים של {{ילד}}?" (expand from the child's lexicon) | 12 §3.9, §3.15 |
| Extraction failed twice | "אפשר להקליד את המילים" quick typing screen; pipeline resumes at step 7 | 12 §3.15 |
| >30 % `[?]` words | recommend re-shoot; may continue with certain words only | 12 §3.15 |
| Model refusal | logged; "לא הצלחנו לקרוא את הדף הזה"; no other model tried | 12 §3.15 |
| Daily $ cap >80 % | Batch deferred; P2/P3 continue to 100 % | 12 §1.9 |
| Daily $ cap >100 % | "המשחק יוכן מחר בבוקר"; parent override with Face ID up to +$5 | 12 §1.9; 11 §8.4 |
| Haiku enrichment fails/timeout 8 s | game built with generic scripted intro/hints and templates without lexical distractors | 12 §3.12 |
| TTS unavailable | K1 plays text-only templates; K2/K3 wait: "הקול יהיה מוכן בעוד כמה דקות" | 12 §3.13 |
| Nakdan unavailable | Phonikud fallback job | 12 §3.1, §8 |
| Offline parent device | UI "צריך אינטרנט"; capture allowed? → queue upload ⟂VERIFY (GAP-15b) | 11 §4.8 |
| Kill switch `photo` | camera hidden in the parent area (T7 off behaves the same) | 11 §4.12; 10 PRIV-02 |
| Time-to-game P95 > 60 s | telemetry `photo.ttg` alarm; not user-visible | 12 §3.14 |

- **E2E sketch:** `E2E-UC-057-a [API]` each row above as a parametrized test with mocked vendor responses; assert photo object deletion for every terminal state.

#### UC-058 Content approval queue: nightly packs, sampling policy, decisions

- **Happy path:** Nightly Batch (Opus 5.5) → validators → judge (Sonnet) → `review_queue`; first 3 packs per family/template/story type 100 % to the parent; then 10 % sample (≥3 items) + every `warn`/disagreement + every story + all P2; a rejection returns the family to 100 % for the next pack; K3 always 100 %. Parent card: pointed text in the child's gender variant with an m/f/pl toggle, audio, answer, distractors with error codes, judge scores; actions ✔ / ✎ (saved as `parent_fixed`, feeds calibration) / ✖ with reason code (factual / nikud / gender / age / tone / image) → "לאשר הכול" only after scrolling the whole sample; AL-08b reminder once after 48 h; packs signed (Ed25519) and only approved packs reach `GET /v1/content/catalog`.
- **Edge cases:** E1: SET-05 policy per type; A1/A3/A4 locked to manual. E2: Unapproved generated content never reaches a child; static pack fallback (DD-14). E3: `kid_flag` items appear (GAP-47). E4: Web dashboard side-by-side approvals (11 §5.10). E5: A pack needing a newer engine → `pending_app_update` (11 §4.4).
- **Spec refs:** 12 §1.8, §2; 10 §6.10 A2–A6, SET-05, AL-08b; 11 §4.4, §7.6, §8.1; 01 DD-14.
- **E2E sketch:** `E2E-UC-058-a [API]` 4th pack of a family → ≥3 sampled items; reject one → 5th pack fully queued. `E2E-UC-058-b [PW]` catalog on the child device excludes pack `pending_parent`; after approval the manifest signature verifies and the pack activates atomically.

#### UC-059 Weekly report (Friday)

- **Happy path:** Cron `*/15 6-12 * * *` UTC; per family `send_at = clamp(candleLighting − 3h, 12:00, 14:00)` Israel time; prepared 09:00; rolling window since the previous snapshot; idempotent `reportId = family + isoWeek`; structure: campfire line, per child (rotating order) 🏆 one win, 🧭 one "working on" incl. what the game does, 🌙 one 5-min tonight activity in a Friday context, 💬 one conversation question, quiet numbers in grey, ≤70 words/child; optional lines (approvals, monthly view link, reward suggestion, share card, overflow alerts); channels: in-app inbox (always), email (RTL HTML with inline `dir`, ≤600 px, `<bdi>` numbers, no tracking pixel, signed 👍/👎 links valid 14 days, deep link), Push "הסיכום השבועי מוכן…" (P1), Telegram (P2), viewer version without numbers.
- **Edge cases:** E1: Friday is Yom Tov → report on the first holiday eve of that week (AC-12). E2: Shabbat mode off → fixed Friday 12:00. E3: No domain → inbox only (10 Q-05). E4: Quiet week → ES-2 template. E5: K3 "beautiful moment" picked from parent-marked observation or telemetry. E6: Never includes flags (AC-14); payload excludes `flags[]`. E7: Activity repeated in the last 4 reports → excluded. E8: Deep-link scheme `familygame://` vs `olam://` → GAP-68.
- **Spec refs:** 10 §11.1–11.9, AC-12, AC-13, AC-14, AC-29, §15.3; 11 §7.1 (cron, to be replaced), §7.10; 12 §5.
- **E2E sketch:** `E2E-UC-059-a [API]` run the cron handler every 15 min across Friday 2026-12-04 (candle lighting 16:07 Tel Aviv) → exactly one send at the first slot ≥12:00 and ≤13:07; re-run → no duplicate. `E2E-UC-059-b [PW]` email HTML snapshot RTL in Playwright; lint: ≤70 words per child block, exactly one of each section (AC-13).

#### UC-060 Monthly view, strengths narrative (Haiku), observation feedback

- **Happy path:** Prepared on the 1st 03:00; attached to the next Friday report (link + short excerpt; no separate message); sections 1–10 (growth headline never negative; per-subject slope with automatic explanations; mastered list; vs. plan; "מה שמנו לב" narrative; interests; outside evidence prompt; next month; "what we adapted"; 3 observations to rate 👍/👎). Narrative: Haiku with `StrengthsNarrativeInput` (`{CHILD}` token, no name/gender; no flags/EF/check-in) → validators (COPY-3 words, gender lexicon, numbers equal to input, no sibling names, ≤180 words) → one retry → `T-M-FALLBACK` template; name/gender injected locally.
- **Edge cases:** E1: T3 consent off → templates only. E2: Dispositions `hidden` until 20 events/5 sessions/3 days; `consistent` ≥40 events/3 weeks. E3: Parent correction overrides inference, −50 % signal weight for 60 days. E4: Shared iPad → narrative blurred (SEC-07). E5: Monthly in the first 2 weeks of data → not generated.
- **Spec refs:** 10 §11.8, §6.7, §11.7, PRIV-02 T3; 08 §8.1–8.4; 12 §5.
- **E2E sketch:** `E2E-UC-060-a [API]` mocked Haiku output containing "גאון" → rejected → retry → fallback; output with "היא" outside `{f|m}` → rejected. `E2E-UC-060-b [SIM]` headline never < 0 under random month deltas.

#### UC-061 Alerts engine: rules, caps, quiet hours, Shabbat, bundling, feedback

- **Happy path:** Nightly evaluation (AL-02…06) + every-15-min dispatch; `COUNTS_TOWARD_CAP` rules ≤2 per child per week (overflow → report "עוד מהשבוע"); ≤1 Push per guardian per day (bundled T-AL-BUNDLE); parent quiet hours 21:00–07:30; Shabbat/holiday window [lighting −30 min, havdalah +30 min] with no sends on Saturday night (accumulated → Sunday 07:30); after the Friday report no further Push until Saturday night except AL-08a / AL-10; memorial days: no celebratory emoji/"!", AL-01 moved a day; expiry 5 days → report only; Push text ≤110 chars, never accuracy/flags/emotional content; in-app card with say/avoid script + CL link + 👍/👎; mute a type for 2 weeks.
- **Edge cases:** E1: AL-08a exempt from quiet hours only if the guardian started the photo in the last 10 min. E2: AL-10 `high` always sent; `low` respects quiet hours. E3: AL-09 flags never Push. E4: AL-11 system (device unsynced 48 h, backup failed, 80 % AI budget, photo delete failed, bad Push token) inbox + Push ≤1/day. E5: Free signing → no remote Push at all; inbox is the reliable channel (10 §6.16) and local notifications where the app is foregrounded ⟂VERIFY.
- **Spec refs:** 10 §10.1–10.3, AC-10, AC-11, §6.16, §15.3; 08 §5.1 (`stuck`), §6.7; 11 §3.1 (push capability).
- **E2E sketch:** `E2E-UC-061-a [SIM]` `FX-ALERTS-6` → ≤2 per child, ≤1 Push/day (AC-10). `E2E-UC-061-b [SIM]` full 5787 with `FX-CAL-5787` for Tel Aviv + Jerusalem → zero Push inside Shabbat/holiday windows except exempt rules (AC-11). `E2E-UC-061-c [PW]` card 👎 → `alerts.feedback` row; "להשתיק סוג זה לשבועיים" → candidate of that rule within 14 days routed to report.

#### UC-062 "לתשומת לב" (early flags) lifecycle

- **Happy path:** `evaluateFlags` nightly on device (consent T4) → `none → watch` (silent engine adaptation) `→ parent_note` (≥3 weeks persisting with exposure) `→ referral_suggested` (≥8 weeks, min exposure, referral gate e.g. `parentConcern.*`); notify only when rank increases and ≥28 days since the last flag notice for that child; shown only in PA-FLAG, inbox, monthly; shared iPad blurred; card: concrete observation, what the game does, home activity, what to ask the teacher; referral: route (CL-11), printable RTL A4 summary without θ, suggested wording; actions: "דיברנו עם המורה", "לא רלוונטי" + reason, "להשהות 4 שבועות", "הדפסה"; pattern gone 4 weeks → `none` + "נראה שזה הסתדר".
- **Edge cases:** E1: Consent off → area shows the static "מתי לפנות" guide only. E2: `ATTN_PATTERN` never names a disorder; referral only with `parentConcern.attention`. E3: Rules have activation dates (e.g. `HEB_K2_LETTERS_PA` from 1 Feb). E4: `LANG_K3_PARENT` from questionnaire answers only. E5: Flags sync only to parents' devices, encrypted; never to AI vendors; deleted on profile reset (08 §9.6). E6: Hotline numbers must be verified before release (08 §9.4; 07 §10). E7: RF-01/02 (07) overlap with `EMO_DISTRESS`/`EMO_CHECKIN` → GAP-32.
- **Spec refs:** 00 C8; 08 §9.1–9.6; 10 §6.9, AC-14, AC-15, AC-32; 07 §10; 04 §15.
- **E2E sketch:** `E2E-UC-062-a [SIM]` 08 §12.9 tests + "two notices within 28 days impossible". `E2E-UC-062-b [PW]` shared iPad → PA-FLAG content blurred until "הצגה"; web → not blurred. `E2E-UC-062-c [API]` child device token → `GET /children/:id/flags` 403 (AC-18).

#### UC-063 Rewards manager, surprise card, appreciation letter

- **Happy path:** PA-RWD: parent defines family activities (experiences by default; `pricedInCoins:false`); suggestions only to the parent, ≤1 per 14 days, triggered by family goal ×3 this season / season album done / 5 real-world missions / parent judgment; **never announced to a child in advance**; after deciding, parent may send "הפתעה מאבא ואמא: נאפה יחד!" card (unexpected reward); appreciation letter (voice or process-praise template; free text linted) → child's hub mailbox.
- **Edge cases:** E1: UI cannot create accuracy/score/sibling-comparison or task→prize triggers (AC-36). E2: No coin/gem grants from the parent area; no such endpoint (AC-37). E3: Suggestions are family-wide or equal for all children (02 §11.7).
- **Spec refs:** 00 D6; 02 §11; 10 §6.11, AC-36, AC-37, API-19.
- **E2E sketch:** `E2E-UC-063-a [API]` `POST /rewards` with `trigger:'accuracy'` → 400; search of API routes for `grant`/`coins` under `/children` → none. `E2E-UC-063-b [PW]` letter text "את גאונה" → lint error shown, cannot send.

#### UC-064 Economy oversight (PA-ECO): refund, sticker, closed shop

- **Happy path:** Coins by source (stacked), gems milestones, chest log with fixed contents, purchases/wishlist/savings, health checks ("מטבעות על זמן: 0" always), controls: "תן מדבקה" (K3 — see GAP-28), refund any purchase, "החנות סגורה בשעות שקט".
- **Edge cases:** E1: Refund of an equipped item → GAP-38. E2: Shop/base >30 % for 2 weeks → suggestion, not alert. E3: Diminishing-returns counter shown as information.
- **Spec refs:** 10 §6.12, API-20; 02 §2.6–2.7, §7.3; 00 D1.
- **E2E sketch:** `E2E-UC-064-a [SIM]` metric "coins per raw minute" is identically 0 for any event log.

#### UC-065 Share cards (SC-01…06) and the iOS share sheet

- **Happy path:** Celebration → "שיתוף" → card rendered on device (Skia) 1080×1350 PNG, no EXIF, `card.png`; forbidden fields impossible (score/percent/minutes/level/photo/school/full name/QR); consent K1 self / K2 "שאלנו את איתי" / K3 parent; channels: iOS share sheet (no account) or viewer feed; offered ≤1/week per child as a report line.
- **Edge cases:** E1: Holiday card SC-06 with family avatars only. E2: Viewer voice reply → PA-APPR A6 → child hears it after approval.
- **Spec refs:** 10 §13, AC-28; 00 C6.
- **E2E sketch:** `E2E-UC-065-a [PW]` render each SC-* for each profile → OCR/DOM check: no digits except in the skill name whitelist; PNG metadata empty.

#### UC-066 Child settings changes (caps, bedtime, subjects weights, nikud, accessibility, AI, picture-PIN)

- **Happy path:** PA-CSET edits write `child_settings` docs (LWW by HLC) → audit diff → synced to devices (foreground every 5 min / next open); effects: caps enforced per child per day across devices; subject weights 10–60 % summing to 100; exploration share 20–30 %; challenge dial (K1 90/80/70, Stretch opt-in); speed rounds (K1, after mastery only); nikud "auto: fade from February"; English sandwich/island length; accessibility (calm, timers off, read-aloud, dyslexia-friendly font, text size); guide mode; AI companion on/off; picture-PIN; 0–100 mastery score display (K1, off).
- **Edge cases:** E1: Two parents edit concurrently → LWW; screen shows "שונה לאחרונה על ידי {guardian}, {time}"; conflict visible in the audit (RB-5). E2: Edit while the child is mid-episode on another device → applies at the next episode (plan already composed) ⟂VERIFY rule (GAP-66). E3: Gender change → audio variants for P2 packs (GAP-52). E4: Weights sum ≠ 100 → inline error.
- **Spec refs:** 10 §6.13, RB-5, API-22; 11 §6.5 (docs LWW), §4.8; 08 §6.8 (default weights); 00 B5 C2 C9.
- **E2E sketch:** `E2E-UC-066-a [API]` two docs with HLCs differing only in deviceId suffix → deterministic winner; tombstone with lower HLC loses. `E2E-UC-066-b [PW]` set cap 25→30 → audit "אבא שינה את המגבלה היומית של איתי: 25 ← 30 דק'" with diff view.

#### UC-067 Device management: rename, mode, allowed children, usage, storage

- **Happy path:** FAM-04 card (name, type, mode, allowed avatars, last sync dot green ≤1 h / amber >24 h / grey >48 h "המכשיר לא התחבר", version + OTA, minutes per child this week as separate bars) → detail: rename (immediate, audited), mode change (parent-only hides the picker), allowed children (K3 only with lap mode), usage heatmap, storage & health (packs, audio cache, pending events, pending photos, gate type), battery-saving sync policy text, install expiry "תוקף ההתקנה: עוד N ימים".
- **Edge cases:** E1: Removing the only allowed child from a `shared_kids` device → blocked (ON-08 rule). E2: Device unsynced ≥48 h → AL-11 + health dot. E3: Mode change while a child session is active on that device → applies after the session ends ⟂VERIFY. E4: Web dashboard device actions require step-up for unpair/wipe (SEC-06).
- **Spec refs:** 10 §7.2 FAM-04, MD-01, AL-11; 11 §5.5, §7.4, §3.3 (expiry).
- **E2E sketch:** `E2E-UC-067-a [PW]` fixture with `lastSyncAt = now − 49h` → grey dot + copy; AL-11 in inbox. `E2E-UC-067-b [API]` `PATCH /devices/:id {allowedChildIds:[]}` on `shared_kids` → 400.

#### UC-068 Bedtime lock: schedule (offline) and "lock now" (remote)

- **Happy path:** Per-device schedule doc `device_settings/<deviceId>.bedtime {start:'19:30', end:'07:00', days}` enforced **locally, offline**; inside bedtime SCR-20 "לילה טוב" screen, no game access; parent gate unlocks for 30 min; remote "לנעול עכשיו עד…" → `POST /v1/devices/:id/commands {type:'bedtime_lock', until}` → delivered in the next `/v1/sync` response or ≤60 s via `/v1/devices/self/commands` polling while foregrounded → ack `applied` → UI "ננעל ✓ (לפני 2 דק')"; child side: 2-min warning, "הגיבור הולך לישון", ends at a natural point; `session.ended{reason:'bedtime'}`; "לפתוח" → `unlock`.
- **Edge cases:** E1: Airplane mode → schedule still enforced (AC-21). E2: Device offline when "lock now" sent → applied on next contact; UI shows pending. E3: Per-child bedtime (10 §6.13) vs. per-device schedule (11 §5.5) → GAP-04. E4: Unlock for 30 min → returns to picker or to the child? → GAP-05. E5: Safe room allowed during bedtime per setting (09 §6.10). E6: Re-entry attempts ≥3 after bedtime → RF-07 info (07 §10) / AL-06.
- **Spec refs:** 10 §7.2 FAM-04, AC-21, U6; 11 §5.3 S5, §5.5, §4.8; 09 SCR-20; 07 §10 RF-07.
- **E2E sketch:** `E2E-UC-068-a [PW]` offline + clock 19:31 → SCR-20; gate → 30-min unlock → 20:02 → SCR-20 again. `E2E-UC-068-b [API]+[PW]` command queued → device poll (60 s) → ack → parent UI text. `E2E-UC-068-c [MAE]` 11 §12.4 row 13.

#### UC-069 Unpair and wipe

- **Happy path (unpair):** step-up → `unpair` → token revoked immediately → next call 401 → device shows "המכשיר נותק מהמשפחה" → `(setup)/welcome`; local data kept until wipe/reinstall. **Wipe:** command delivered on next contact (token valid only for `/commands` + `/ack`) → device pushes unsynced events first → deletes SQLite + packs + Keychain → ack → token revoked; never online → `wipe_pending`, revoked after 7 days.
- **Edge cases:** E1: Child mid-session when unpair arrives → session ends gracefully, events pushed before the 401? (unpair revokes immediately → unsynced events of that device are lost unless wipe is used) → GAP-57b (recommend "unpair = wipe-lite": push then revoke). E2: Re-pair same device to the same family → restore (UC-081). E3: Lost device → UC-083.
- **Spec refs:** 11 §5.3 S5, §5.4, §7.4; 10 §7.2 FAM-04, AC-20, SEC-06.
- **E2E sketch:** `E2E-UC-069-a [API]` unpair → `/v1/sync` 401; wipe with 12 unsynced events → events present in D1 before `device_commands.status='acked'`; offline device → status `wipe_pending`, token invalid after +7 days.

#### UC-070 Audit log browsing and export

- **Happy path:** FAM-05 sentence rows ("אבא שינה את המגבלה היומית של נועה: 40 ← 30 דק'") with device/time → diff view; filters (guardian, action, child, device, dates); CSV export; logged: settings changes, approvals, pairing/unpair/lock/wipe, invites/roles, export/delete, ≥5 gate failures, PIN change, consents; not logged: viewing; append-only; 24 months; PIN hashes/tokens never logged.
- **Edge cases:** E1: Actor `device` for L-mode edits ("device-local parent", 11 S4) → attribution text ⟂VERIFY. E2: Viewer cannot open FAM-05.
- **Spec refs:** 10 §7.2 FAM-05, RB-3, AC-19; 11 §5.6, §7.12.
- **E2E sketch:** `E2E-UC-070-a [API]` every mutating route in API-02..04, 12, 14, 18..23 writes one `audit_log` row in the same batch (AC-19); grep of `diff_json` for `pin`/`token` → none.

#### UC-071 Data export

- **Happy path:** PRIV-03 → step-up → `POST /v1/admin/export` → ZIP of NDJSON per table (+ proposed per-child CSV and PDF summary) in R2 → emailed link valid 24 h → audit.
- **Edge cases:** E1: No email domain → link must be returned in-app → GAP-49. E2: Co-admin may export (11 §5.4). E3: `local_only` data (check-ins) is not in the export (never on the server) — the privacy table says so (PRIV-01).
- **Spec refs:** 10 §8 PRIV-03, AC-26; 11 §7.12, §5.4.
- **E2E sketch:** `E2E-UC-071-a [API]` export → ZIP contains `events.ndjson` with no `cls:'local_only'` rows; link 404 after 24 h + 1 min.

#### UC-072 Deleting a child (soft) and deleting the family

- **Happy path:** Child delete (owner only, step-up): soft delete; events purged after 30 days; "ביטול" possible meanwhile; devices get a command to remove the child locally (type missing → GAP-51). Family delete: `DELETE /v1/family {confirm:'<family name>'}` + 7-day grace; backups follow the backup cycle (disclosed).
- **Edge cases:** E1: Co-admin tries → 403 (AC-27). E2: During grace → what devices show → GAP-56. E3: Deleted child's neighborhood/friends in the hub → removed locally on command ⟂VERIFY.
- **Spec refs:** 10 §8 PRIV-04, AC-27; 11 §7.3, §5.9.
- **E2E sketch:** `E2E-UC-072-a [API]` delete child → `children.deleted_at` set, events still present day 29, gone day 31; undo day 10 → restored.

#### UC-073 Consent layers (PRIV-02) and their effects

| Layer | Off → expected effect | Test |
|---|---|---|
| T0 adaptive | cannot be turned off; explanation shown | toggle absent/disabled |
| T1 interest inference | interest graph deleted; parent seeds + explicit child choices remain; PA-INT shows seeds only | `[SIM]` graph cleared, seeds kept |
| T2 EF games | age defaults only | plan contains no EF calibration items |
| T3 narrative | templates only; no Haiku call | `[API]` no `summary` job for the family |
| T4 early flags | `evaluateFlags` returns []; PA-FLAG shows static guide | `[SIM]` + `[PW]` |
| T4b EMO_CHECKIN | off by default; on → boolean flag only (GAP-32 unify with RF-01) | `[SIM]` |
| T5 parent concerns | opens the referral stage of relevant rules | `[SIM]` referral gate |
| T6 AI companion | scripted hints only | `[PW]` no text input / no `/ai/companion` calls |
| T7 photo-to-game | camera hidden in the parent area | `[PW]` |

- **Spec refs:** 10 §8 PRIV-02, ON-10; 08 §1.7 consent, §7, §8.3, §9.1; 12 §4.2.

#### UC-074 Web dashboard (Cloudflare Pages) sign-in and viewing

- **Happy path:** `olam-parent.pages.dev` (or `parent.<domain>`) → email code (phase 1) → refresh cookie `__Host-olam_rt` (HttpOnly/Secure/SameSite=Strict, 30 d), access token in memory, `X-Olam-Client: web` on mutations, strict CSP; pages: overview, child progress/skills/settings, approvals side-by-side, devices (incl. add device QR + code), family, AI log, audit, data; same numbers as the app (AC-34, shared `packages/engine/metrics`); P1 read-only, P2 edit with step-up; 30-min idle lock; print sheets A4 RTL for monthly and flags.
- **Edge cases:** E1: Cloudflare Access in front during phase 1 (review brief item 9) ⟂VERIFY integration with the email-code flow. E2: <1024 px → iPad layout; <768 → iPhone layout. E3: Viewer sees overview only. E4: No PA-FLAG blur on web (10 §6.9).
- **Spec refs:** 00 H6; 11 §5.10; 10 §14.3, SEC-09, AC-34, AC-35.
- **E2E sketch:** `E2E-UC-074-a [PW]` sign in, open child overview → numbers equal the app's for the same fixture (metrics module snapshot); mutation without `X-Olam-Client` → 403. Axe checks pass; keyboard navigation reaches every chart with a Hebrew `aria-label`.

#### UC-075 Interests (PA-INT): corrections, seeds, exploration balance, child's "I want to learn" list

- **Happy path:** Top ≤5 topics with "why we think so" (3 moments), new sparks, dormant, parent seeds (interests + sensitivities), child-owned list (K1; parent view-only), exposure balance across 7 domains (<10 % → auto-rebalance "איזון חשיפה: מתוקן"), real-world step (Renzulli), exploration share slider 20–30 %; actions "נכון/לא מדויק/יותר/פחות" override inference with a date; stage promotion to `emerging_passion` needs ≥3 days of initiated evidence in 14 days **and** explicit child/parent confirmation.
- **Edge cases:** E1: Gender never a feature (C9). E2: K3 only "אהוב". E3: Passion Passport / deep-dive season (07 SD.PASS, 06 §14) surfaces here as the current deep-dive.
- **Spec refs:** 00 C9; 10 §6.6, API-14; 08 §7; 06 §14; 07 §6 SD.PASS.
- **E2E sketch:** `E2E-UC-075-a [SIM]` 30-day proposals with one domain at 8 % → next proposals raise it to ≥10 %; interest weights invariant to the gender field.

### 2.D Ops, system, failure and resilience

#### UC-076 Weekly 7-day re-sign routine and install expiry

- **Actors:** Developer agent (LaunchAgent on the Mac), parent, app.
- **Happy path:** `tools/mac/resign.sh` daily 18:40 + at login; per device with `now − installedAt ≥ 5 days`: rebuild if the cached `.app` is older than 5 days (`git checkout family-*` tag, `pnpm i --frozen-lockfile`, release build), install over Wi-Fi via `devicectl` (USB fallback), verify with `devicectl device info apps`, update state; smoke test: app launches, data present (13 §3.2); parent sees "תוקף ההתקנה: עוד N ימים" in FAM-04; at ≤1 day a gentle banner on the gate screen; kids never see expiry; weekly scheduled reminder on Friday leaves a reserve day; recommended cadence Thursday evening (13 Q-07).
- **Edge cases:**
  - E1: ≥6 days without success → `POST /v1/ops/notify` → guardian email "צריך לחבר את האייפד למק לחידוש האפליקציה" + macOS notification.
  - E2: Expired app → iOS refuses to launch ("Unable to Verify App"); **never delete the app** (13 §3.2); reinstall over it restores everything (same bundle ID + team); local data stays.
  - E3: Signing missed twice in a month → agent proposes the paid program (13 §3.4), never performs it.
  - E4: Locked device may refuse installs ⟂VERIFY (11 §3.3).
  - E5: New native dependency → fingerprint change → `needs-native-build` label; OTA waits (11 §11.3).
  - E6: Daily JSON export to Files/Mac during signing (VS phase, 13 §3.2) — superseded by D1 sync from MVP.
- **Spec refs:** 00 E1a; 11 §3.3, §3.4, §12.4 #15; 13 §3.2–3.4, §10.3; 10 §7.2 FAM-04.
- **E2E sketch:** `E2E-UC-076-a [SIM]` unit test of `resign.sh` state logic (dry-run mode): devices at 4/5/6 days → skip/build/notify. `E2E-UC-076-b [PW]` `buildSignedAt = now − 6d` → gate banner present; child routes contain no expiry element.

#### UC-077 OTA update: preview → family, rollback, never mid-episode

- **Happy path:** PR `ota-safe` → `ota.yml` publishes to `main` → `preview` channel (parent iPhone canary) → `promote.yml` manual (parent clicks "Run workflow") or automatic after 24 h without S0/S1 → `family` branch; app `checkAutomatically: ON_LOAD`, `fallbackToCacheTimeout: 0`; update **applies at the next cold start only**; `Updates.reloadAsync()` only from the parent area ("עדכון זמין — להפעיל מחדש?"); release checklist: CI green + screenshot diff approved, Hebrew changelog, content validators/judge passed + sample approved, canary first, backup before migration, **no OTA Friday afternoon/Shabbat/holiday except S0**.
- **Edge cases:**
  - E1: S0 (child safety/privacy/data loss/offensive content) → roll back within 1 h (`eas update:roll-back-to-embedded` or republish previous group) (13 §10.1).
  - E2: Shared iPad never cold-starts (kids don't kill apps) → update never applies → GAP-26 (proposed reload boundary).
  - E3: Fingerprint mismatch → CI does not publish to `family`; JS waits for the new binary (11 §11.3).
  - E4: `minRuntime` in config newer than the device → pack `pending_app_update`; app shows parent-only notice ⟂VERIFY (GAP-27).
  - E5: Gender bug → S2 always (13 §10.1).
- **Spec refs:** 11 §3.3 (OTA), §11.3–11.5; 13 §10.1–10.2; 00 E1.
- **E2E sketch:** `E2E-UC-077-a [API/CI]` workflow test: PR with a changed native dependency → label `needs-native-build`, no `eas update` step. `E2E-UC-077-b [MAE]` 13 §10.3 "עדכון OTA מגיע ומוחל באתחול".

#### UC-078 Nightly content generation (Batch), validators, judge, cost caps

- **Happy path:** Cron 02:10 Israel: rollups, alerts evaluation, `evaluateFlags`, backups, cleanup, **and** the night's content packs: Opus 5.5 Batch (K1/K2/K3 labels only, never names — 29-day retention) → schema validation (two layers) → validators (nikud, gender, line length per age, banned words, numbers by lookup) → judge (Sonnet, `safety` hard-fail) → review queue (UC-058) → TTS bake (cache by hash, LUFS normalize) → signed pack → catalog.
- **Edge cases:**
  - E1: Daily `DAILY_USD_CAP` ($3; 11 says $1.00 → GAP-14e) >80 % → Batch deferred to the next day; >100 % → P2 "tomorrow morning", P3 scripted only; AL-11 at 80 %; workspace monthly limit $30/$40 (11 §8.4 vs 12 §1.9 → same GAP).
  - E2: Batch failures/refusals → retries per 12 §2.7; family keeps existing packs (nothing visible to the child).
  - E3: Prompt prefix hash check in CI per `prompt_version` (12 §1.9).
  - E4: Vendor terms check >180 days → CI fails (12 §8).
  - E5: Phonikud/Nakdan down → the other; neither → pack waits, flagged `warn`.
- **Spec refs:** 00 E8 (+ brief item 12); 12 §1.2–1.9, §2, §6.7–6.9, §8; 11 §7.1, §8.1, §8.4; 10 §15.3.
- **E2E sketch:** `E2E-UC-078-a [API]` cron handler with mocked Batch: cost ledger at 85 % → job status `deferred`; `ai_log` rows never contain a child's name (regex over fixture names). `E2E-UC-078-b [SIM]` validator suite over a golden pack set (incl. a planted "גאון" and a masculine verb on an `f` item → both fail).

#### UC-079 AI / vendor outage, kill switches, degraded modes

| Failure | Child experience | Parent experience | Spec |
|---|---|---|---|
| Anthropic API down | Existing packs play; גלגלי scripted only; P2 "לא הצלחנו לקרוא… ננסה שוב" | AL-11 if persistent; SET-07 shows errors | 12 §8 last column; 11 §4.11 |
| `killSwitches.ai` | same as above, instantly after config refresh (ETag) | PA-AI shows "כבוי זמנית" | 11 §4.12, §7.7 |
| `killSwitches.photo` / `coop` | camera hidden / lobby same-device only | — | 11 §4.12 |
| TTS vendor down | new P2 lines: K1 text-only, K2/K3 wait; baked VO unaffected | "הקול יהיה מוכן בעוד כמה דקות" | 12 §3.13 |
| R2 unavailable | packs already on device play; new downloads retry with backoff (0.5→30 s, jitter) | health dot | 11 §4.11 |
| D1/Worker down | fully offline play; sync queue grows; backoff cap 15 min | banner "עודכן לפני {t}" | 11 §4.8, §6.5; 10 §14.4 |
| Resend down | report stays in inbox; email retried ⟂VERIFY retry policy (GAP-49b) | SET-07 | 10 §11.2 |
| Config endpoint down | bundled defaults + cached `remote_config` | — | 11 §4.12 |
| Cloudflare cron skipped | idempotent reports/alerts catch up next run; report window closes at 14:00 → report sent late? → rule: send at next run even after 14:00 same day ⟂VERIFY (GAP-23b) | — | 10 §11.1 |

- **E2E sketch:** `E2E-UC-079-a [PW]` intercept `/v1/ai/*` with 503 → K1 hint ladder shows scripted node ids; no spinner longer than 6 s. `E2E-UC-079-b [API]` set `killSwitches.ai=true` → `/v1/ai/companion` returns `{code:'ai_disabled'}` before any vendor call.

#### UC-080 Offline for days, then sync; conflicts

- **Happy path:** Everything plays offline (bundled core content; packs on device; PIN hashes; picture-PIN; caps from local events; bedtime schedule); events appended with device-local `seq`; on foreground / every 5 min online / episode end / "סנכרון עכשיו": `POST /v1/sync` push oldest-first ≤500/call ≤1 MB, pull pages, `commands[]`, `configVersion`, `serverTime`; events idempotent by id; docs LWW by HLC; identity server-authoritative; projections rebuilt if older events arrive.
- **Conflicts:** economy double-spend → negative hidden; chest track duplicate → coins instead (UC-017/E2); settings edited on two devices → HLC winner + audit note; clock skew >2 min → telemetry + HLC uses max(local, server); far-future clock → server clamps HLC to `serverNow + 5 min`.
- **Edge cases:**
  - E1: Invalid event (schema) → rejected individually, kept locally with `last_error`, visible in debug; never blocks the batch.
  - E2: `child_id` not allowed on this device → rejected (except `parent_only`).
  - E3: `local_only` class → never pushed (sync filter test) and rejected server-side (defense in depth).
  - E4: 3 days offline (`FX-FAM-OFFLINE-3D`) → cap totals from other devices merge after sync; overrun tolerated.
  - E5: Battery-saving policy: during a session push every 2 min or 50 events; background only via BGTask/next open; no polling when closed (10 FAM-04).
  - E6: Device reports `lastSyncAt` >48 h → AL-11 (T-AL-11 "ההתקדמות שמורה בו ותעלה כשיתחבר").
  - E7: Weekly report while a device is unsynced → report uses what the server has; the next report covers the rest (rolling window).
- **Spec refs:** 00 E6; 11 §4.8, §6.4–6.6, §5.7–5.8; 10 §7.2 FAM-04 (sync policy), AL-11, §11.1; 01 DD-15.
- **E2E sketch:** `E2E-UC-080-a [PW]+[API]` offline 3 days (route block) → play 2 episodes → online → `/v1/sync` called with ≤500 events per call, all accepted; second device pulls and shows the same friend count. `E2E-UC-080-b [API]` push the same batch twice → `accepted` both times, D1 row count unchanged. `E2E-UC-080-c [MAE]` 11 §12.4 row 12.

#### UC-081 Device restore / replacement (new iPad, reinstall, team change)

- **Happy path:** Pair as a new device (UC-006) → first pull with `cursor 0` → all family events (paged) + docs → `rebuildProjections` → resume; ~15k events/year ≈ 6 MB; packs re-downloaded from the catalog (hash-verified); resume docs per child restore position/pending chest.
- **Edge cases:** E1: Old device still paired → both active; unpair the old one. E2: Unsynced events on the lost old device are gone (see UC-083). E3: Restore after team change (11 §3.3) — sync first. E4: D1 Time Travel 30-day PITR; nightly R2 exports; quarterly restore drill `tools/ops/restore.ts` (11 §6.6, §9.6).
- **Spec refs:** 11 §6.6, §4.4, §9.6; 10 §7.2 FAM-04.
- **E2E sketch:** `E2E-UC-081-a [API]` fresh device cursor 0 on a 20k-event family → paged pulls (`hasMore`), final projections equal the server's `child_skill_week` rollup.

#### UC-082 Storage pressure, pack integrity failures, missing content

- **Happy path:** Old pack versions deleted after 7 days or immediately when storage is low; sha256 mismatch → delete + refetch ×3 → `failed`, previous good version kept; `minAppRuntime/minEngine` → `pending_app_update`; bundled core content guarantees a fully working fresh install offline.
- **Edge cases:** E1: No space for a new region pack → gate "בקרוב"? and parent alert → GAP-40 (no `storage_low` alert, no child copy). E2: Audio cache eviction policy for TTS (R2 cache mirrored on device) ⟂VERIFY LRU size (GAP-40). E3: `expo-image` disk cache growth bounded ⟂VERIFY. E4: Parent "storage & health" card shows pack volume and audio cache (FAM-04).
- **Spec refs:** 11 §4.4, §4.7, §4.11; 10 §7.2 FAM-04.
- **E2E sketch:** `E2E-UC-082-a [PW]` serve a manifest with a wrong sha256 → pack stays on the previous version; 3 retries logged; no child-visible error.

#### UC-083 Lost or stolen device

- **Happy path:** Parent (any device/web) → FAM-04 → `unpair` (immediate token revoke) or `wipe` (on next contact; `wipe_pending` → token revoked after 7 days); guardian refresh tokens are Keychain-bound to Face ID/passcode; PIN is a convenience gate only (scrypt slows offline brute force); admin actions need the guardian session.
- **Edge cases:** E1: Device never reconnects → local SQLite (learning events, hashed PINs, resume docs) stays on the device under iOS passcode protection → GAP-41 (specify `NSFileProtectionComplete` and what PII is at rest). E2: Unsynced events lost → parent sees a gap in PA-LOG; nothing else breaks. E3: Device found again after wipe → re-pair as new (UC-081).
- **Spec refs:** 11 §9.7 (threat model rows), §5.3 S5, §5.1; 10 AC-20.
- **E2E sketch:** `E2E-UC-083-a [API]` revoked device token → every route 401 incl. `/v1/content/*`; `wipe_pending` device → only `/commands` + `/ack` succeed.

#### UC-084 Interruptions: calls, backgrounding, lock screen mid-celebration, Bluetooth, silent switch

- **Expected behavior:** Audio session set once at boot (`playsInSilentMode: true`, interruption → resume); narration audible on silent switch; incoming call → game pauses, returns to "לגעת כדי להמשיך" at the checkpoint; background mid-level → resume doc saved; lock mid-celebration → rescue/chest already persisted (events written at the action, not at the animation end); Bluetooth headphones on/off → audio routes correctly; Control Center swipe → same as background; Guided Access → no in-app change; palm/10 fingers/rapid taps → no crash, no loss.
- **Spec refs:** 11 §4.5, §4.16, §12.4 row 14; 13 §10.3 "ילדים ומקרי קצה"; 09 §6.7.
- **E2E sketch:** `E2E-UC-084-a [PW]` dispatch `visibilitychange` hidden during T1 → `child_resume` doc written; return → overlay "לגעת כדי להמשיך"; friend count unchanged. `E2E-UC-084-b [MAE]` call simulation on device (📱 row 14).

#### UC-085 Accessibility paths

| Setting / need | Child side | Parent side | Test |
|---|---|---|---|
| Reduce Motion (OS or profile "רגוע") | springs without overshoot, fades 200 ms, no shake, ≤3 slow particles, chest flash off, 3D camera slow, no bob | same | `prefers-reduced-motion` snapshots (11 §12.2) |
| Photosensitivity | ≤1 flash/s, never >3/s, white flash ≤30 % opacity | — | animation test (AC-CEL-3) |
| Text size 100–150 % per profile; dyslexia-friendly font (PA-CSET) | layouts reflow; nikud not clipped at largest size | Dynamic Type up to AX3: grids become lists, nothing truncated (AC-35) | largest Dynamic Type snapshots |
| Captions for VO (default on for K1 and parent) | every VO line has an optional caption | — | string coverage lint |
| Motor | targets in mm by age (09 §4.2), "tap-tap" alternative for every drag, no mandatory multi-finger gesture, joystick side swap, long-press ×1.5, no timers except K1 opt-in | — | UX-AC-06 |
| Movement | seated variant for every MV-*, "מצב שכנים" | — | UX-AC-12 |
| VoiceOver | labels on navigation + primary buttons, RTL focus order; parent area fully accessible; each chart read as its takeaway sentence then "פירוט ←" | full | UX-AC-04, 10 AC-35 |
| Switch Control | focus ring; every action reachable by a single tap | — | manual |
| Cognitive | one instruction per screen; ≤3 choices for little ones; fixed HUD positions; Guided Access guide for parents | — | playtest |
| Auto-open chests (a11y setting) | opens without taps after preview | — | `[PW]` |
| Audio | 3 sliders (music/SFX/VO), quiet mode (music off, SFX 50 %, VO full), haptics toggle | — | `[PW]` |
| Color | never color alone; chest frames differ by shape; CVD simulation passes | no red for child performance | UX-AC-02; 10 AC-07 |

- **Spec refs:** 09 §3.2–3.4, §4, §5.5–5.6, §6.10, §10, §13.2; 10 §6.13, §14.4, AC-35; 11 §12.2; 01 §16.1; 02 §6.5.
- **E2E sketch:** `E2E-UC-085-a [PW]` matrix run with `prefers-reduced-motion` + largest Dynamic Type for every screen in 11 §12.2; axe on parent screens; CLS = 0.

#### UC-086 Backups, cron health and system alerts

- **Happy path:** Nightly 02:10 export per table to `olam-private/backups/d1/<date>/` (35 daily + 12 monthly); D1 Time Travel; AL-11 on backup failure; quarterly restore drill; retention purges (telemetry 30 d, `ai_log` 90 d unless pinned, events 24 months → monthly aggregates + R2 archive, audit 24 months, sessions +7 d, photos ≤30 d backstop).
- **Edge cases:** E1: Backup failure two nights → AL-11 cooldown 24 h → second alert. E2: Cron overlap (long Batch) → idempotent jobs. E3: Photo delete failure → AL-11 + lifecycle backstop.
- **Spec refs:** 11 §5.9, §6.6, §9.6, §13; 10 §10.1 AL-11, SET-07.
- **E2E sketch:** `E2E-UC-086-a [API]` fake R2 failure in the backup job → `alerts` row AL-11 `backup_failed`; run again next day → second row only after 24 h.

#### UC-087 Concept prototypes (PATH 0) and design approval gates

- **Happy path:** Web export → Claude artifact or `olam-proto.pages.dev` preview → parent opens on iPad Safari (COOP/COEP headers for SQLite; else in-memory DB) → taste gates (moodboard, character sheets, avatar parts, Rive rigs, audio review; 00 E10) → decisions logged.
- **Edge cases:** E1: Prototype must not be used for kids' daily play (11 §3.2). E2: No haptics/Rive native/Face ID on web (capabilities table).
- **Spec refs:** 11 §3.2; 00 E10 I1; 13 §4 (C1–C2); 09 §14.
- **E2E sketch:** `E2E-UC-087-a [PW]` `EXPO_PUBLIC_DB=memory` boot renders the hub first frame with `window.__olam.ready`.

#### UC-088 Content pack update arriving mid-session

- **Expected behavior:** Catalog check and downloads happen off the UI frames (idle callbacks, 1 MB chunks); the atomic version switch happens in one SQLite transaction; the **running session keeps its loaded items**; the new version is used from the next level load; the child never sees a "new content" interstitial; TTS replacements after nightly loudness normalization download silently (12 §3.13).
- **Spec refs:** 11 §4.4; 12 §3.13.
- **E2E sketch:** `E2E-UC-088-a [PW]` publish a new pack version while in a level → current item ids unchanged until the level ends; next level uses the new version.

#### UC-089 Crash and error boundaries

- **Expected behavior:** Root boundary "משהו השתבש. ננסה שוב?" with mascot; per-route boundaries for `(child)` and `(parent)`; mini-game boundary → hub with no loss (attempt discarded, no penalty); `app.error` telemetry (stack ≤4 KB, 30-day retention); API error shape `{error:{code,message,requestId,retryable}}` with Hebrew copy from i18n.
- **Edge cases:** E1: Crash during chest reveal → `chest.opened` persisted at the first tap → collected state on relaunch. E2: Crash during sync push → idempotent re-push. E3: Repeated crash loop on one route → feature flag per device to disable (`perChild`/device flags) ⟂VERIFY (11 §4.12 evaluation order supports per-device).
- **Spec refs:** 11 §4.11, §4.12, §5.9.
- **E2E sketch:** `E2E-UC-089-a [PW]` inject a render error in `game/[gameId]` → hub shown within 1 s; `app.error` event with `route`; friend jar unchanged.

#### UC-090 Clock, timezone and calendar edge cases

- **Expected behavior:** All calendar logic (Shabbat, holidays, report send time, week boundaries, cap day) uses **the family's city/timezone (`Asia/Jerusalem`)**, not the device's; device skew >2 min → telemetry; HLC clamp; DST transitions (Israel: late March / late October) do not double-count or skip a day; events carry `tzOffsetMin` for display.
- **Edge cases:** E1: Family travels abroad with the iPad → Shabbat per home city vs. local → GAP-39 (decision needed). E2: Device clock set far in the future by a child → server clamp; local cap day computed from `max(local, server)` ⟂VERIFY. E3: Cap "day" boundary → GAP-44.
- **Spec refs:** 11 §6.4–6.5; 10 §4 (Asia/Jerusalem), §11.1; 02 §10.1; 01 §15.4.
- **E2E sketch:** `E2E-UC-090-a [SIM]` week/day boundary tests across the 2027-03-26 and 2027-10-31 DST switches; `adventureDay` counted once.

#### UC-091 Parent voice recording (name ×3, 3 lines) and its use

- **Happy path:** Parent area → guided in-app recording (12 §6.11; 01 §14.1 step 3; 13 §16.5 script): child's name ×3 children + opening/goodnight/milestone lines → used by FTUE ("אור בוקר, [name]"), K3 "קוקו", story cover, milestone T3 moments; no clone unless consented later (00 E9).
- **Edge cases:** E1: No recording → lines play without the name (01 §19). E2: Re-record → replaces; old file deleted. E3: Where stored (device + R2 private?), download to other devices, size → GAP-67.
- **Spec refs:** 00 E9; 01 §14.1, §19; 12 §6.11; 13 §16.5.
- **E2E sketch:** `E2E-UC-091-a [MAE]` record on the parent iPhone → K2 FTUE on the iPad plays the name clip after sync (📱 11 §12.4 row 4 extension).

#### UC-092 Playtest and paper-probe routines (research workflow, not product)

- **Happy path:** Weekly playtests per 13 §11 (observation sheet per child, Fun Toolkit by age, distress signs stop the session); paper probes twice a week per 13 §12 (multiple-baseline); results entered in W-ASM-06; monthly paper check PDF; teacher notes.
- **Edge cases:** E1: Distress signs (13 §11.6) → stop, log, parent informed — not an in-app flag. E2: Probes pay nothing (02 §2.7).
- **Spec refs:** 13 §11–12; 10 §6.8 W-ASM-06; 00 C10.
- **E2E sketch:** `E2E-UC-092-a [PW]` enter a paper probe result → PA-ASM chart updates; no `econ.*` event.

### 2.E Cross-cutting edge-case groups (checklists)

#### EC-01 Gender and neutral forms

- Every child-facing and child-referring string has `m`/`f`/`pl` (+`n` where K3 pending); numbers spoken from the lookup table only (feminine rote counting); Playwright contact sheet groups each screen across genders; wrong gender is always S2; LLM output gender-checked against the lexicon; Sunny needs no variants (addresses "you"). **Tests:** AC-CEL-5, 10 AC-08, 11 §12.2 matrix, 12 validators.

#### EC-02 RTL / LTR

- World is never mirrored; 2D HUD RTL; forward progression right→left on boards/maps; English islands LTR with fixed buttons; parent sidebar on the right, iPhone back arrow points right; charts' time axis right→left (10 RTL-2); email inline `dir` per block. **Tests:** 11 §4.14, §12.2; 09 §3.11; 10 AC-29.

#### EC-03 No-loss invariants (C5)

- No hearts/energy/streak loss; nothing wilts/dies; chests never expire; friends/items/buildings/logs never removed; campfire level never resets; negative balances hidden; abandoned level saved; boss "sleeps"; pet never sad. **Tests:** 02 §12.4 invariants, 01 AC-BTL-1, AC-STOP-2, AC-EV-2.

#### EC-04 Comparison and privacy between siblings (C6)

- No side-by-side metrics anywhere (AC-01); co-op shows a shared meter only; K1 guide mode sees no stats; siblings cannot edit each other's neighborhoods; CO-4 letters structured; share cards per child with consent; weekly report rotates order. **Tests:** 10 AC-01/02, 01 AC-CO-4/5.

#### EC-05 Security situation (B8)

- No sirens/alarms/explosions/war content (Safety Core); SFX spectral check; safe room offline in the dark; Yom HaAtzmaut lantern parade; shofar only as real shofar quietly. **Tests:** AC-SHAB-2, 12 §1.3 validators.

#### EC-06 Praise and copy

- Process praise only; banned trait labels/superlatives/comparison/guilt lists (01 §16.3, 09 §9.3) linted on all `*.he.json`, scripts and stored LLM output (10 AC-09); gender parity test `praiseParity.test.ts`; exit lines whitelist. **Tests:** AC-CEL-4, AC-STOP-3.

#### EC-07 Time, caps and autoplay

- No autoplay, no "one more", no countdowns anywhere (shop, season, events, SCR-20); caps per child across devices; co-op 50 %; wind-down fixed; re-entry via gate (K2/K3); bedtime per device; quiet windows. **Tests:** AC-STOP-1/2, 10 AC-21, 11 §4.16.

---

## 3. Cross-spec gap list (journey steps no spec defines precisely enough)

Each row: the owning spec that should carry the rule, the journey that exposed it, the gap, and a proposed rule (a default that the developer can implement now; the owning spec's reviewer accepts or replaces it). The same rows, grouped by owning file, are in `docs/review/fable-issues-F9.md`. IDs are stable (they are referenced from §2) and therefore not sequential.

| ID | Owner | Journey | Gap | Proposed rule |
|---|---|---|---|---|
| GAP-01 | 01 §14.2 / 11 §4.16 | UC-012, UC-081 | FTUE is described per child but no state says it already ran; a restored/second device would replay the 60-second opening. | Add `ftueCompletedAt` to the `child_resume` doc (LWW). FTUE runs only when null; the onboarding rescues are the endowed 3 friends. |
| GAP-02 | 08 §6.10 / 01 §15.1 / 11 §7 | UC-040 | Cap overtime: 01 says finish the beat ≤1:30; 08 says 60 s grace at closing. DD-09 "+10 min bonus" has no event, API or audit entry. | Rule: at 85 % → `closing`; if the cap ends inside a beat, finish it (≤90 s) then land; 08's 60 s grace applies only inside `closing`. Add event `cap.bonus_granted {minutes:10, guardianId or 'device-local'}` (cls `admin`), gate-only button, audit row, max 1/child/day. |
| GAP-03 | 01 §15.1 vs 10 §6.13 | UC-011, UC-040 | K3 daily cap: 01 = 2 episodes / 10 min; 10 = 15 min. | Default 10 min / 2 episodes (00 C3: 3–5 min × 2); parent range 5–15. 10 updates the table. |
| GAP-04 | 10 §6.13 / 11 §5.5 / 01 §15.1 | UC-040, UC-068 | Bedtime is per child (10 PA-CSET, 01 defaults differ: K2 19:30 vs 20:00) and per device (11 schedule). The "no new session 60 min before bedtime" rule has no exact semantics. | Both exist: `child_settings.bedtime` and `device_settings.bedtime`; effective lock = earliest. Defaults K3 19:30, K2 20:00, K1 20:30 (10 wins). Pre-bedtime rule: a new episode may start only if `bedtime − now ≥ plannedEpisodeMin + 2`; otherwise the picker tile shows "הגיבור כבר מתכונן לשינה" and SCR-20 at bedtime. |
| GAP-05 | 11 §5.5 / 10 FAM-04 | UC-068 | After the gate unlocks bedtime for 30 min, where does the device go? | To the picker; the 30-min window is device-wide; at its end the running episode gets the 2-min warning and lands; `session.ended{reason:'bedtime'}`. |
| GAP-06 | 11 §4.16 / 08 §6.10 | UC-010, UC-012 | Inactivity 5 min → picker, but session accounting (cap, `session.ended` reason, checkpoint) is unspecified. | Idle ≥60 s is already excluded from M-01; at 5 min: save resume, `session.ended{reason:'idle'}`, the level resumes from its checkpoint next time; idle minutes do not count toward the cap. |
| GAP-07 | 11 §4.16 / 10 §6.13 | UC-010 | Picture-PIN: after 3 misses the hint text appears, but the 4th tap, attempt counting and the reset flow are undefined. | After 3 misses: grid hidden for 60 s (no countdown shown), tile shows the parent icon; misses emit `picturepin.miss` (telemetry) and feed M-14 "sibling access attempts"; parent reset → next entry runs "בחרי 3 תמונות" once, then verify. |
| GAP-08 | 09 §5.3 vs 02 §6.5 | UC-017 | Chest: 09 "auto after 1.5 s without tap"; 02 "animated hand after 4 s, auto-open only as an accessibility setting"; 01 "taps = 1 + effort marks". | 02 wins (owning spec): tier-based taps, hand at 4 s, auto-open only via a11y. 09 and 01 §16.2 align. |
| GAP-09 | 02 §7.7 / 11 §6.5 | UC-019 | Neighborhood layout edited offline on two devices (same tile). | `neighborhood_layout/<childId>` doc, LWW by HLC; an item displaced by the merge returns to the inventory with a mailbox note; nothing is lost. |
| GAP-10 | 02 §7.5 / 11 §6.5 | UC-018 | Savings bonus uses the week's minimum balance; a merged negative projection would make it negative. | Bonus base = `max(0, minBalance)`; overdraft weeks pay no bonus. |
| GAP-11b | 01 DD-18 / 11 §4.16 | UC-049 | K3 at 3.5 moves to "co-play-first with short solo periods" but the picker's parent-present rule for K3 is binary. | Add `child_settings.launchMode ∈ {lap, coplayFirst, solo}`; `coplayFirst` = gate required once per day, solo episodes capped at 5 min. |
| GAP-12 | 01 §14.2–14.3 vs 08 §3 vs 10 ON-12 | UC-009 | 01: "no explicit placement test, play-based inside regular levels"; 08: explicit expeditions (3–4 × 8–10 min); 10: "2–4 short days". | 08 wins on structure; the expeditions are story-wrapped levels with 08 §3.6 UI rules (no right/wrong). 01 §14.3 week-1 table should allocate expedition slots: K2 Sun–Tue one each (inside the level slot), K1 Sun–Wed. |
| GAP-13 | 08 §3 / 02 §2.7 / 01 §6 | UC-009 | Expedition levels rescue friends but must pay no coins/XP; no flag tells the economy. | `level.mode` ∈ {placement, normal, review, boss, event}; economy ignores `econ` sources when `mode='placement'`; collection still counts rescues; chests not triggered. |
| GAP-14b | 01 §9.3 vs 02 §6.2 | UC-029 | Weekly boss reward: 01 "K1: 2 gems"; 02 "בוס שבועי: silver + 1 gem". | 02 wins on numbers (brief item 1): silver + 1 gem; 01 §9.3 updates. |
| GAP-14c | 01 §15.4 vs 10 SET-01 | UC-045 | Candle-lighting offset: 01 "Tel Aviv 20 min"; 10 "18 min (other cities), 30 Haifa, 40 Jerusalem". | Use `@hebcal/core` city defaults (Jerusalem 40, Haifa 30, others 18) with manual override; 01 updates. |
| GAP-14d | 01 §15.4 / 02 §10.3 | UC-045 | Shabbat (soft) starting mid-episode: are the episode's receipts paid? | The episode that started before candle lighting is paid in full at its receipt; counters freeze only for sessions started after lighting. |
| GAP-14e | 11 §8.4 vs 12 §1.9 | UC-078 | AI daily cap: 11 "$1.00 per family-day" + $30/month; 12 "$3/day" + $40/month. | 12 wins (owning pipeline spec): `DAILY_USD_CAP=3`, `MONTHLY_USD_CAP=40`; 11 updates `ai_daily_cap_cents` default to 300. |
| GAP-15 | 11 §8.3 vs 12 §3.3 | UC-054 | Photo resize: 11 "1600 px long edge"; 12 "2576 px, q0.88"; 11 "face check later" vs 12 on-device face blur module. | 12 wins; 11 §8.3 and §9.4 update. |
| GAP-15b | 11 §4.8 / 12 §3 | UC-057 | Photo capture while offline: allowed and queued, or blocked? | Capture + crop + blur allowed offline; upload queued; the parent sees "יישלח כשיהיה חיבור"; queue max 4 jobs; job expires after 48 h unsent (photo deleted locally). |
| GAP-16 | 09 F4 vs 12 §3.1 | UC-054 | Child-side surfacing of a photo game: 09 "glowing `secret` node on the map 'הפתעה מ{{parent}}'"; 12 "appears in the child's island 'משחק חדש מהמחברת!'". | Both: map node (09) labelled per 12; for K2 the node is the "review" slot of the next episode's plan so it is reachable without map browsing. |
| GAP-19 | 01 §15.1 / 09 / 11 | UC-039 | "Audio-only car mode" exists in 01 only (no screen, launch path, cap accounting). | P2 feature: parent-gate toggle "מצב רכב" → playlist (music hut songs, riddles); no econ events; counts 0 toward caps; screen dimmed with one "עצור" button. |
| GAP-19b | 09 SCR-16 / 11 §4.16 | UC-041 | Safe-room time and cap accounting. | Safe room time never counts toward caps; `session.started{mode:'safeRoom'}` for PA-LOG category "סיפור/רוגע". |
| GAP-21 | 11 §10.2 / 02 §7.6 | UC-031 | Multi-device "solo co-op" after a socket drop merges contributions as events, but the DO summary is authoritative for campfire level-ups. | Level-ups come only from the DO summary; offline contributions add "logs" and light to the next session's meter; never a retroactive level. |
| GAP-22 | 11 §5.4 (L) / 10 RB-0 | UC-050, UC-066 | L-mode edits caps "on this device only" while caps are per child across devices. | `device_settings.childOverrides[childId].dailyCapMin`; effective cap on that device = `min(childCap, override)`; audit actor `device-local parent`. |
| GAP-23b | 10 §11.1 | UC-079 | Cron missed the 12:00–14:00 window (platform outage). | Send at the next run the same day even after 14:00 but ≥3 h before lighting; otherwise Sunday 07:30 with subject "השבוע שלנו (באיחור)". |
| GAP-24 | 11 §5.8 / 10 §6.5 / 08 | UC-039, UC-040, UC-061 | `session.ended.reason` is untyped; specs use natural/cap/exit/bedtime/switch/childQuit. | Enum in `packages/shared/events.ts`: {natural, cap, childQuit, switch, idle, bedtime, parentStop, crash, coopEnd, shabbat}; AL-04 uses `childQuit`. |
| GAP-25 | 01 §15.1 / 10 AL-04 | UC-016 | "Back to camp" from the pause menu: does it count as `childQuit`? | Yes, `reason:'childQuit'` with `secondsSinceLastError` in payload; AL-04 applies its 120-s rule. |
| GAP-26 | 11 §3.3, §11.3 | UC-077 | Updates apply only at cold start; a shared iPad is rarely cold-started. | Reload boundary: if an update is downloaded, the app was backgrounded ≥30 min and no child session is active → `reloadAsync()` on next foreground before the picker. |
| GAP-27 | 11 §4.4 / 10 | UC-077, UC-082 | Pack `pending_app_update` and expired signing have no parent-facing copy. | FAM-04 health: "חבילת {name} מחכה לעדכון אפליקציה"; AL-11 variant `app_update_needed` once per 7 days. |
| GAP-28 | 10 W-ECO-06 / 02 §2.4 | UC-064 | "תן מדבקה" without limit — for which children? | K3 only (D5). For K2/K1 the parent uses the appreciation letter (W-RWD-04). |
| GAP-29 | 01 DD-20 vs 09 §8.1 | UC-013 | iPhone hub orientation: 01 landscape; 09 portrait (landscape supported). | 09 wins (UX owner): hub portrait-first on iPhone, landscape allowed; 01 DD-20 updates. |
| GAP-30 | 05 §10.4 / 01 §15.1 / 01 AC-MX-3 | UC-025 | Cap warning (ניצי, Hebrew) would fire inside an English island. | Defer the warning to island exit; if the cap/bedtime ends inside the island, Sunny's "Bye-bye" + exit jingle first, then the Hebrew wind-down. |
| GAP-31 | 01 §5.3 vs 07 §2.4 | UC-035 | Check-in frequency: 01 "every other day" (K2); 07 every episode. | 07 wins: every episode, ≤20 s, skippable ("לא רוצה להגיד"); 01 §5.3 updates. |
| GAP-32 | 08 §9.2 / 07 §10 / 10 §6.9 | UC-035, UC-062 | Three overlapping emotional rules: 07 RF-01 (≥5/7 in 14 d), 10's proposed `EMO_CHECKIN` (3 of 5 days), 08 `EMO_DISTRESS`. | One rule in 08 §9.2 `EMO_CHECKIN` with 07's thresholds, consent T4b, computed on device, boolean out; 10 and 07 reference it. |
| GAP-33 | 01 §12.5 / 11 §5.8 | UC-034 | "Patience heart" needs a detection rule and an event. | `turn.yielded` when `session.ended{reason:'switch'}` at <80 % of the planned episode and a different child starts within 120 s on the same device; ≤1 per child per day; next episode includes the VIP surprise. |
| GAP-35 | 10 §7.1 / 11 §3 | UC-008 | Viewer "in the app in Viewer mode" is impossible on free signing (3 devices, cable). | Phase 1: viewers are web-only; app mode after the paid program. |
| GAP-36 | 11 §5.2 / 10 ON-02 | UC-007 | Without a sending domain, the co-admin cannot receive an email code. | Add `bootstrap` support for a second guardian: `pnpm ops bootstrap-guardian --email <coadmin> --role coadmin` → one-time code; documented in 11 §5.2. |
| GAP-37 | 02 §6.2 / 10 W-HOME-09 | UC-039 | Mission confirmation → wood chest: no cap, no child-side surfacing. | ≤1 mission chest per child per day; surfaced next episode as a mailbox letter + chest on the shelf. |
| GAP-38 | 02 §7.3 / 10 W-ECO-06 | UC-018, UC-064 | Parent refund of an equipped/placed item. | Unequip to the previous outfit / remove from the grid, coins returned, item back in the catalog; child sees a neutral mailbox note; never a "taken away" animation. |
| GAP-39 | 01 §15.4 / 10 §4 / 11 §6.4 | UC-045, UC-090 | Travel abroad / device timezone ≠ family city. | All calendar logic uses the family city; add `family_prefs.travelMode` (parent toggle) that switches to device-local sunset for Shabbat while on; telemetry `tz_mismatch`. |
| GAP-40 | 11 §4.4 / 10 AL-11 | UC-082 | No `storage_low` alert, no child-facing copy for a region whose pack is missing, no audio-cache bound. | AL-11 variant `storage_low` (<500 MB free); region gate shows "האזור עוד נטען" (no error); TTS cache LRU ≤400 MB. |
| GAP-41 | 11 §9.7 | UC-083 | Data at rest on a lost offline device. | `olam.db` and packs under `NSFileProtectionCompleteUntilFirstUserAuthentication`; Keychain items `WHEN_UNLOCKED_THIS_DEVICE_ONLY` (already); document PII at rest = names, events, PIN hashes. |
| GAP-42 | 10 W-ECO-06 / 02 §7 | UC-018 | "Shop closed in quiet hours" — parent Push quiet hours or child windows? | Child "app sleeps" windows and bedtime only. |
| GAP-43 | 01 §15.1 vs 09 SCR-07 | UC-039 | K1 after goodnight: 01 allows collection room + music hut; 09 sends everyone to the picker. | Picker tile for K1 after wind-down opens "camp asleep" view with collection room and music hut only (no session, no econ); K2/K3 tiles require the gate. |
| GAP-44 | 08 §6.10 / 11 §4.16 / 02 §10.1 | UC-040, UC-090 | Daily cap day boundary undefined. | Day = 04:00–04:00 family timezone for caps and surprises; the family-goal week stays Sunday 00:00. |
| GAP-46 | 01 §12.1 CO-4 / 00 C7 | UC-033 | Async sibling letter authored by K1 — moderation. | Structured authoring only (chips, numbers, pictures from the pack); no free text; stored as a `coop_letter` doc. |
| GAP-47 | 10 §6.10 / 12 §1.8 | UC-038, UC-058 | `kid_flag` quarantine items are not in PA-APPR's type list. | Add **A7 "סימון של ילד"**: manual, shows the item + replacement; decision codes as A2. |
| GAP-48 | 13 §10.1 / 11 §12 | all `[MAE]` sketches | No on-device E2E layer (Maestro) in the QA strategy; only Playwright web + manual device checklist. | Add Maestro flows (`tools/e2e/maestro/*.yaml`) run by the agent on the Mac against the `dev` build (simulator + one real iPad), triggered per native build and weekly; dev-build-only deep link `olam://test/fixture?id=` + injected clock; `testID` on all interactive elements (09 §12). |
| GAP-49 | 10 PRIV-03 / 11 §7.12 | UC-071 | Export link only by email; no domain in phase 1. | `POST /admin/export` returns the signed link in the response as well; shown in-app. |
| GAP-49b | 10 §11.2 / 11 | UC-079 | Email delivery retry policy (Resend failure). | Retry 3× with backoff over 2 h; then inbox-only with a health note in SET-07. |
| GAP-50 | 01 §5.1, §15.1 / 11 §2.3 | UC-039, UC-043 | Printing (season certificate, offline mission, flag summary) — no library/flow. | `expo-print` → PDF → iOS share sheet/AirPrint; all printables A4 RTL. |
| GAP-51 | 11 §5.1 / 10 PRIV-04 | UC-072 | Child soft-delete sends "a command to remove the child locally" but `DeviceCommand` has no such type. | Add `child_remove {childId}` and define `resync` (drop projections + full pull). |
| GAP-52 | 10 AC-25 / 12 §3.13 | UC-003, UC-066 | Gender change "no download needed", but P2 packs render TTS only for the selected children's variants. | Core packs ship all variants; on gender change the Worker re-renders missing P2 variants (queued job) and the device downloads silently; AC-25 scoped to core content. |
| GAP-53 | 01 §10.1 / 11 §5.1 | UC-003 | A 4th child: schema supports N; hub has 3 neighborhoods. | Not supported in v1 UI; FAM-03 blocks a 4th child with a message; recorded as a v2 item. |
| GAP-56 | 10 PRIV-04 / 11 §7.3 | UC-072 | Family delete 7-day grace: device behavior. | Sync returns 410 `family_deleting`; devices play offline-only with a parent-area banner "המשפחה בתהליך מחיקה"; after purge → 401 → setup. |
| GAP-57 | 11 §5.3 S3 | UC-006 | Claiming a pairing on a device that holds data of another family. | Claim compares local `familyId`; mismatch → require a local wipe confirm (step-up) before claiming. |
| GAP-57b | 11 §5.3 S5 | UC-069 | `unpair` revokes immediately → unsynced events on that device are lost. | `unpair` = command: push unsynced events, then revoke (like wipe but keeps local data); immediate revoke only via `unpair {force:true}` for lost devices. |
| GAP-60 | 09 §10 / 13 §3.3 | UC-011 | Guided Access exit uses the device passcode kids may know. | Parent guide: set a Guided Access passcode different from the device passcode; note in SEC-02 copy. |
| GAP-62 | 10 AL-08a / 11 §3.1 | UC-054, UC-061 | AL-08a "immediate Push" impossible on free signing. | Phase 1: in-app poll while the review screen is open + a **local notification** scheduled when the app is backgrounded during a photo job; Push once path 2. |
| GAP-63 | 01 §9.3, DD-06 | UC-029 | Finale with an absent child; boss day falling on a holiday eve/quiet day. | `bossHP` counts only children who played ≥1 episode this week; the finale completes with any present subset; if Thursday is a Yom Tov/QD, the window moves to the previous weekday evening. |
| GAP-66 | 10 §6.13 / 08 §6.3 | UC-066 | Settings changed while a child is mid-episode. | Caps/bedtime apply immediately (safety); content settings (weights, nikud, AI) apply from the next episode. |
| GAP-67 | 01 §14.1 / 12 §6.11 / 11 | UC-091 | Parent voice clips: storage, sync to other devices, format. | Record in the parent area → m4a ≤10 s each → R2 private `voice/<familyId>/<guardianId>/<clipId>.m4a` + device cache; distributed through the content catalog as a family pack; re-record replaces. |
| GAP-68 | 10 §11.9 vs 11 §4.1 | UC-059 | Deep-link scheme mismatch: `familygame://report/{id}` vs `olam://` (pair only). | One scheme `olam://`; allowed paths `pair`, `report/{id}`, `approvals`, `test/fixture` (dev only). |
| GAP-72 | 10 §6.13.1 / 08 §6.9 | UC-053 | Test date < 3 days away. | No AL-13; one review block in the next session; copy "המבחן קרוב, חזרה קצרה היום". |

---

## 4. Traceability matrix (journeys × spec sections)

Legend: ● primary source · ○ secondary/constraint. Files: 00 foundation, 01 game design, 02 economy, 03 math, 04 Hebrew, 05 English, 06 extra subjects, 07 future skills, 08 engine, 09 UX, 10 parent area, 11 tech, 12 AI pipelines, 13 delivery/QA.

| UC | 00 | 01 | 02 | 03 | 04 | 05 | 06 | 07 | 08 | 09 | 10 | 11 | 12 | 13 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| UC-001 install | E1a | | | | | | | | | | | ● §3 | | ● §3 |
| UC-002 onboarding | H2 | §14.1 | | | | | | | ○ §3.3 | | ● §7.1 | ● §5.2–5.3 | | |
| UC-003 add child | A2 B3 | | | | | | | | | | ● ON-04, FAM-03 | ○ §5.1 | | |
| UC-004 avatar | G4 | ● §3.3–3.4 | ○ §7.1 | | | | | | | ○ SCR-09 | | | | |
| UC-005 PIN/Face ID | E7 | | | | | | | | | | ● ON-06, §9 | ● S4 | | |
| UC-006 pairing | H1 | | | | | | | | | | ● ON-07/08 | ● S3, §7.4 | | ○ §10.3 |
| UC-007 co-admin | H1 H2 | | | | | | | | | | ● §7.1–7.2 | ● S2, §5.4 | | |
| UC-008 viewer | H1 H4 | | | | | | | | | | ● §13, RB-2 | ○ §5.10 | | |
| UC-009 placement | C2 | ○ §14.2 | ○ §2.7 | ○ §14 | | ○ §14 | | | ● §3 | | ○ ON-12, PA-ASM | | | |
| UC-010 picker/PIN | H3 | ○ §12.5 | | | | | | | | ○ SCR-01 | | ● §4.16 | | |
| UC-011 K3 lap | C3 H3 | ● §15.3 | ○ §2.4 | | ○ §12 | | | | | ○ SCR-17 | ○ AL-06 | ○ §4.16 | | |
| UC-012 K2 episode | C3 | ● §5.3 | ○ §2.3, §2.8 | ○ | ○ | ○ | ○ | ○ §12 | ● §6 | ○ F1 | | | | |
| UC-013 K1 episode | C2 C3 | ● §5.3 | ○ §2.2 | ○ | ○ | ○ | ○ | ○ §12 | ● §6 | | | | | |
| UC-014 K3 episode | D5 | ● §5.3 | ○ §6.2 | | | | | | | | | | | |
| UC-015 level/rescue | G1 G2 | ● §6 | ● §5 | | | | | | | ○ SCR-05 | | ○ §4.11 | | |
| UC-016 frustration | C2 | ● §6.5 | ○ §2.6 | | | | | | ● §6.7 | | ○ AL-04 | | | |
| UC-017 chest | D2 | ○ §16.2 | ● §6 | | | | | | | ○ §5.3 | | ○ §5.8 | | |
| UC-018 shop | D3 | | ● §7.1–7.5 | | | | ○ F.* | | | ○ SCR-08 | ○ PA-ECO | ○ §6.5 | | |
| UC-019 building | D4 | ● §10.5 | ● §7.7 | ○ M.MEAS | | | | | | ○ SCR-10 | | | | |
| UC-020 collection | G2 | ● §10.3 | ● §5.4 | | | | | | | ○ SCR-12 | | | | |
| UC-021 pets | D4 | ○ §4.12 | ● §8 | | | | | | | ○ SCR-11 | | | | |
| UC-022 capsules | D2 | | ● §5.7 | | | | | | | | | | | |
| UC-023 surprise | C5 | ● §5.5 | | | | | | | | | | | | |
| UC-024 movement | G6 | ● §8 | | | | ○ MV-08 | ○ H.* | | | ● SCR-19 | | ● §4.17 | | |
| UC-025 English island | B2 | ○ §4.3 | | | | ● §4, §10 | | | | ○ CMP-22 | | ○ §4.1 | | |
| UC-026 speaking | E9 | | | | | ● §13 | | | | | | | | |
| UC-027 runner | E1b | ● §9.4 | | | | | | | | ○ 06f | | | | |
| UC-028 evening | G5 B8 | ● §9.5 | | | | | | | | ○ 06e | | | | |
| UC-029 weekly boss | G5 | ● §9.3 | ○ §6.2 | | | | | | | | | | | |
| UC-030 co-op same device | C6 | ● §12 | | | | | | ○ SE.TEAM | | ○ SCR-13 | ○ PA-LOG | ● §10.1 | | |
| UC-031 co-op multi | C7 | ○ §12.1 | | | | | | | | | | ● §10.2 | | |
| UC-032 guide mode | C6 | ● §12.6 | | | | | | | | | ○ PA-CSET | | | |
| UC-033 async letter | C7 | ● CO-4 | | | | | | | | | | | | |
| UC-034 turn-taking | | ● §12.5 | | | | | | | | | | ○ §4.16 | | |
| UC-035 check-in | C8 | ○ §5.3 | | | | | | ● §5, §10 | ○ §6.7 | | ○ PRIV-01 | ○ §5.7 | | |
| UC-036 AI helper | C7 E8 | ○ CH-03 | | | | | | ○ §2.3 | | | ○ PA-AI | ○ §7.7 | ● §4 | |
| UC-037 AI literacy | C7 | ○ MX-A | | | | | | ● §3 | | | | | ● §4.5 | |
| UC-038 "מוזר לי" | | | | | | | | ○ RF-06 | | | ○ PA-APPR | | ● §1.8 | |
| UC-039 wind-down | C3 C5 | ● §15.1 | ○ §6.2 | | | | | ○ RIT-CLOSE | ○ §6.10 | ● SCR-07/20 | ○ W-HOME-09 | | | |
| UC-040 cap/bedtime | C3 | ● §15.1 | | | | | | | ● §6.10 | | ● §6.13, FAM-04 | ● §4.16, §5.5 | | |
| UC-041 safe room | B8 | | | | | | | | | ● SCR-16 | | | | |
| UC-042 bedtime story | C7 | ● §15.5 | | | | | | | | | ○ A2 | | ○ §2.5 | |
| UC-043 season track | D4 | ○ §13 | ● §9 | | | | | | | | | | | |
| UC-044 events | B7 B8 | ● §13 | ○ §10.3 | | | | | | | | | | | ○ §2.1 |
| UC-045 Shabbat/QD | B7 | ● §15.4 | ● §10.3 | | | | | | ○ §5.3 | ○ SCR-20 | ○ SET-01 | | | |
| UC-046 family goal | C5 | ○ §10.4 | ● §10 | | | | | | | | ○ W-HOME-01 | | | |
| UC-047 monthly boss | | | | ○ MECH.BOSS | ○ §8 | | | | ● §10.2–10.3 | | ○ PA-ASM | | | |
| UC-048 re-entry | | | | | | | | | ● §10.4 | | ○ AL-05 | | | |
| UC-049 rollover | | ● DD-18, S6 | | ○ §7 | ○ §13 | ○ §15 | | | | | ○ FAM-03 | | | |
| UC-050 gate | E7 | | | | | | | | | ○ SCR-15 | ● §9 | ● S4 | | |
| UC-051 PA-HOME | C6 | | | | | | | | | | ● §6.1 | | | |
| UC-052 map/skill | C8 C9 | | | ○ §6 | ○ §3 | ○ §3 | ○ | ○ | ○ §5.1 | | ● §6.2–6.4 | | | |
| UC-053 taught this month | | | | ○ §7 | ○ §13 | ○ §15 | | | ● §6.9 | | ● §6.13.1 | | | |
| UC-054 photo printed | E8 | | | ○ §8 | ○ §7 | ○ §11 | | | | ○ SCR-14 | ○ A1 | ○ §8.3 | ● §3 | |
| UC-055 homework | | | | | | | | | | | | | ● §3.10 | |
| UC-056 child story | C7 | | | | | | | ○ AI.CREATE | | | ○ A5 | | ● §3.11 | |
| UC-057 photo failures | | | | | | | | | | | | ○ §8.4 | ● §3.15, §1.9 | |
| UC-058 approvals | C7 | ○ DD-14 | | | | | | | | | ● §6.10 | ○ §4.4 | ● §1.8 | |
| UC-059 weekly report | | | | | | | | | | | ● §11 | ○ §7.10 | ○ §5 | |
| UC-060 monthly/narrative | C9 | | | | | | | | ● §8 | | ● §11.8, §6.7 | | ● §5 | |
| UC-061 alerts | | | | | | | | | ○ §5.1, §6.7 | | ● §10 | | | |
| UC-062 flags | C8 | | | | ○ §15 | ○ §17 | | ○ §10 | ● §9 | | ● §6.9 | | | |
| UC-063 rewards | D6 | | ● §11 | | | | | | | | ● §6.11 | | | |
| UC-064 economy oversight | D1 | | ○ §2.6 | | | | | | | | ● §6.12 | | | |
| UC-065 share cards | C6 | | | | | | | | | | ● §13 | | | |
| UC-066 child settings | B5 C9 | | | | | | | | ○ §6.8 | | ● §6.13 | ○ §6.5 | | |
| UC-067 devices | H5 | | | | | | | | | | ● FAM-04 | ○ §5.5 | | |
| UC-068 bedtime lock | B8 | | | | | | | ○ RF-07 | | ○ SCR-20 | ● FAM-04 | ● S5, §5.5 | | |
| UC-069 unpair/wipe | | | | | | | | | | | ○ FAM-04 | ● S5 | | |
| UC-070 audit | H4 | | | | | | | | | | ● FAM-05 | ● §5.6 | | |
| UC-071 export | | | | | | | | | | | ● PRIV-03 | ○ §7.12 | | |
| UC-072 delete | | | | | | | | | | | ● PRIV-04 | ○ §7.3 | | |
| UC-073 consents | C8 C9 | | | | | | | | ○ §1.7 | | ● PRIV-02 | | | |
| UC-074 web | H6 | | | | | | | | | | ● §14.3 | ● §5.10 | | |
| UC-075 interests | C9 | | | | | | ○ §14 | ○ SD.PASS | ● §7 | | ● §6.6 | | | |
| UC-076 re-sign | E1a | | | | | | | | | | | ● §3.3 | | ● §3.2 |
| UC-077 OTA | E1 | | | | | | | | | | | ● §11.3 | | ● §10.2 |
| UC-078 nightly batch | E8 | | | | | | | | | | | ○ §8.1 | ● §1–2 | |
| UC-079 outages | | | | | | | | | | | | ● §4.11–4.12 | ● §8 | |
| UC-080 offline/sync | E6 | ○ DD-15 | ○ §6.3 | | | | | | | | ○ FAM-04 | ● §4.8, §6.5 | | |
| UC-081 restore | | | | | | | | | | | | ● §6.6 | | |
| UC-082 storage | | | | | | | | | | | | ● §4.4 | | |
| UC-083 lost device | | | | | | | | | | | | ● §9.7 | | |
| UC-084 interruptions | | | | | | | | | | ○ §6.7 | | ● §4.5, §12.4 | | ○ §10.3 |
| UC-085 accessibility | | ○ §16.1 | ○ §6.5 | | | | | | | ● §5.5, §10 | ○ §14.4 | ○ §12.2 | | |
| UC-086 backups | | | | | | | | | | | ○ SET-07 | ● §5.9, §9.6 | | |
| UC-087 prototypes | E10 I1 | | | | | | | | | ○ §14 | | ● §3.2 | | ○ §4 |
| UC-088 pack mid-session | | | | | | | | | | | | ● §4.4 | ○ §3.13 | |
| UC-089 crash | | | | | | | | | | | | ● §4.11 | | |
| UC-090 clock/tz | B7 | ○ §15.4 | ○ §10.1 | | | | | | | | ○ §4 | ● §6.4 | | |
| UC-091 parent voice | E9 | ○ §14.1 | | | | | | | | | | | ● §6.11 | ○ §16.5 |
| UC-092 playtest/probes | C10 | | | | | | | | | | ○ W-ASM-06 | | | ● §11–12 |

**Coverage notes.** Curriculum specs 03–07 are reached mainly through the engine (08) and the episode templates (01); their journey-relevant surfaces are: class-topic schedules (03 §7, 04 §13, 05 §15), placement axes (08 §3.2 ↔ 03 §14, 05 §14), flags (04 §15, 05 §17, 07 §10), English island rules (05 §10), movement and SEL rituals (07 §2.4, §5), passion passport (06 §14, 07 §6). Every `[SIM]` sketch for these should import the curriculum pack fixtures rather than synthetic skills.

---

## 5. Open items for the parent surfaced by this catalog (batched; nothing blocks)

1. Confirm K3 daily cap 10 vs 15 min (GAP-03) and bedtime defaults (GAP-04).
2. Confirm the weekly boss day (Thursday) and the finale window (GAP-63).
3. Travel abroad behavior for Shabbat times (GAP-39).
4. Whether viewers (grandparents) exist in phase 1 at all (web-only, GAP-35).
5. Whether to buy the sending domain now (unblocks UC-007 email codes, UC-059 email, UC-071 links).

---

## Fable review changelog

- **2026-11-02 (F9):** Created this file. Read 00 and specs 01–13 (full for 00, 01 §3–§18, 02 §2–§11, 08 §3–§10, 09 §5–§10, 10 all, 11 §3–§12, 12 §1, §3–§5, §8, 13 §2–§3, §10; headings and targeted sections of 03–07). Wrote 92 journeys (UC-001…UC-092) plus 7 cross-cutting edge-case checklists (EC-01…EC-07), each with preconditions, happy path, alternatives, edge cases, expected behavior, spec references and an E2E sketch tagged by test layer (`[PW]`, `[MAE]`, `[API]`, `[SIM]`, `[📱]`). Added a fixture catalogue (§0.3) and the environment matrix (§0.4) so sketches are concrete. Collected 64 cross-spec gaps (GAP-01…GAP-72, non-sequential by design) with proposed default rules, mirrored into `docs/review/fable-issues-F9.md` grouped by owning spec. Added a traceability matrix (§4) of all journeys × spec sections and a short batched list of parent questions (§5). Marked unverified harness hooks and copy with `⟂VERIFY`.
