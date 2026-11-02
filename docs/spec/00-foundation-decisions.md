# 00 — Foundation Decisions (Engineering Decision Record)

> Source of truth for all spec documents in `docs/spec/`. Every spec MUST be consistent with this file.
> If a spec author finds strong evidence to change a decision here, they keep the decision, and add a
> "Proposed change to 00" note at the end of their spec with the evidence. Do not silently diverge.
>
> **Revision 2 (Fable review, 2026-11-02):** every "Proposed change to 00" in specs 01–13 was adjudicated
> (see Appendix A). Accepted proposals are applied in place below. IDs are stable; clarifications that
> refine an existing decision use a letter suffix (`C7a`, `E6a`); genuinely new decisions take the next
> free number. Where two specs proposed the same ID for different things, the ID was assigned once and
> the other spec is told in `docs/review/fable-issues-F1.md`.
>
> Precedence when documents disagree: **parent's explicit requests > this file > the owning spec of the
> topic (see §J) > evidence in research notes.**

Inputs: `reports/אפליקציית למידה משחקית לילדים.md` (v1 report), notes in
`research_notes/אפליקציית למידה משחקית לילדים/` (round 1–2) and
`research_notes/תוכנית הוליסטית לאפליקציית הלמידה/` (round 3–4, incl. `gap_audit.md` with 25 reconciliations),
and the "Proposed change to 00" sections of `01`–`13`.

Marker: `⟂VERIFY` = not yet verified; the line says how to verify.

---

## Decision index

| ID | One-line summary |
|---|---|
| A1 | Family-only app: 3 kids, 2 parents; no public release, no monetization |
| A2 | Children K1 (girl 8.5, grade 3), K2 (boy 5.5, gan chova), K3 (toddler 2.5, gender TBD) |
| A3 | iPad-first, iPhone-adapted; orientation rules |
| A4 | What the kids love (Astro Bot, 99 Nights, Roblox, Brawl Stars…) and self-like avatars |
| A5 | Parent = senior UX designer holding taste gates; developer brings domain expertise |
| A6 | Age bands `T`/`G`/`S` are the shared curriculum vocabulary (skills tagged by band, not by child) |
| A7 | Touch targets defined in millimetres per child (20/15/10 mm), converted per device model |
| B1 | Hebrew-first, RTL everywhere |
| B2 | English in LTR "islands", one-person-one-language, sandwich gloss; Hebrew TTS never says English |
| B2a | K2 English is oral only this year; English letters only incidentally |
| B3 | Grammatical-gender variants for every string that addresses or refers to a child |
| B4 | Spoken numbers from a hand-checked table; written numbers as digits (LTR isolate) |
| B5 | Nikud: full for K2/K1 default; ktiv male canonical; per-word adaptive fade F0–F4 with a free nikud lens |
| B5a | Fonts: Noto Sans Hebrew (reading), Rubik (display), Andika (English); OFL only |
| B6 | Evidence beats local convention |
| B7 | Jewish calendar; Shabbat window via `@hebcal/core`; modes soft/full/off; weekly report Friday 12:00–14:00 |
| B7a | Quiet days list (Yom HaShoah, Yom HaZikaron, Tisha B'Av, Yom Kippur; Yom Yerushalayim neutral) |
| B8 | Security-situation sensitivity: no sirens/explosions/fireworks; darkness limits; K3 dusk only |
| B8a | Bravery ladders never touch security topics; "safe room" is not exposure and needs professional review |
| B9 | RTL direction conventions: code strips, arrows, time axes, number lines |
| C1 | Intrinsic integration: the learning action is the game action |
| C2 | Success bands per child; K1 dial 0.85/0.80/0.75, 0.70 only in opt-in Stretch Zone |
| C3 | Episode lengths; K1 level 6–9 min, K2 level 3–6 min; parent caps; no autoplay |
| C4 | Process praise only; banned trait-praise list |
| C5 | No loss mechanics; weekly family goal instead of streaks |
| C6 | No sibling comparison; co-op with different roles |
| C7 | No open-ended AI chat for children; scripted, sandboxed, labelled as a machine |
| C7a | AI tiers T0–T3; closed-vocabulary (chips) input only; nothing shown to a child before parent approval |
| C8 | Not therapy; red flags to parents only, non-diagnostic, with referral paths |
| C8a | Emotional data: never leaves the episode except local 14-day counters; only one boolean flag may sync |
| C9 | Strengths-based profiling; 20–30% exploration applies to topics/wrappers, not difficulty or skills |
| C10 | Phygital bridge; paper probes are the real success metric |
| C11 | One definition of mastery and one review ladder (1/3/7/14/30/60) |
| C12 | No visible clocks; response time scored only for K1 fluency skills in an opt-in speed round |
| C13 | Physical-safety tags on every real-world mission; small-parts rule while a child < 3 lives at home |
| C14 | Celebration budget ≤15% of episode time; answer feedback ≤0.8 s and non-blocking |
| C15 | Canonical skill-level names from 08 (`new/beginner/advancing/proficient/master`) |
| D1 | Coins (mastery & effort) + gems (milestones only); no real money; sparks are not a currency |
| D1a | No coins for emotional check-ins, kindness, bravery steps or reflection |
| D2 | Chest contents fixed at earn time; reveal "upgrade steps" only for already-earned effort marks |
| D3 | Cosmetic-only shop; catalogue fixed; weekly showcase rotates on Sunday; nothing ever leaves |
| D4 | XP/levels, skill badges, pets that evolve with mastery, season album, home base, campfire (no season reset) |
| D5 | Economy per child: K3 stickers (+hidden parent-only XP), K2 simple, K1 full |
| D6 | Real-world rewards set by parent, never priced in coins, never announced as contingent prizes |
| D7 | 02 owns every economic number; 01 and others reference 02's config |
| E1 | App stack: Expo SDK 58 / RN 0.88 / TS strict / expo-router; phased build (Mac now, EAS later); EAS Update OTA |
| E1a | Install paths: dev build via parent's Mac (free team, 7-day re-sign) now; EAS ad hoc after paid program |
| E1b | 3D layer: three.js + R3F 9 + expo-gl; budgets; fallbacks; glyphs only as pre-baked textures |
| E2 | Rendering/animation: Reanimated 4, Gesture Handler, Skia, Rive; no game engine lib |
| E3 | Device modules incl. `expo-sensors`, `expo-sqlite` + kv-store (no MMKV), local-only notifications in phase 1 |
| E4 | Testing from Linux: web export + Playwright (pinned Docker), Jest; Maestro on the Mac simulator; device checks by parent |
| E5 | Backend: Cloudflare Workers Paid, D1, R2, Cron, Durable Objects (`CoopRoom`), Pages; staging at C3, prod from MVP sprint 3 |
| E6 | Local-first sync; append-only event log; LWW for settings; content packs by hash manifest |
| E6a | Sensitive derived data stays in the family: on-device, encrypted to guardian devices only, never to AI vendors |
| E7 | Auth: email one-time code (phase 1) → Sign in with Apple added in phase 2; children are profiles; parent gate rules |
| E8 | AI server-side only; Claude models per job; Batch is not ZDR → labels K1/K2/K3 only; child-safety prompt slot |
| E8a | Vendor terms: no Gemini/Google GenAI; ElevenLabs only with written approval |
| E9 | Voice: pre-generated narration; TTS bake-off; human-recorded Hebrew (~300) and English (~130 + songs) banks; no scoring of child speech |
| E10 | Assets: Recraft (paid) + GPT Image (offline) + hand drawing; Rive; CC0 audio; taste gates; provenance log |
| E11 | Monorepo layout incl. `packages/shared`, `packages/i18n`, `packages/api-client`, `apps/parent-web` |
| E11a | Engine is deterministic and event-sourced; canonical order `(ts, deviceId, eventId)` |
| E12 | Performance and quality playbooks are binding references |
| E13 | Cloud-environment network allowlist (hosts list); CI deploys regardless |
| G1 | Joy of discovery first: explorable world with dense secrets |
| G2 | Rescue & collect loop; rescued friends/pieces are non-spendable and never gate the curriculum |
| G3 | One gimmick per level; short levels; kishōtenketsu |
| G4 | Original hero = "toy-suit with a living spark"; never called a robot; gears reserved for the machine |
| G5 | Day/night co-op, non-scary |
| G6 | Learning through movement with seated alternative and safety rules |
| G7 | Console-game feel; white-hat motivation only |
| H1 | Household model: Family → Guardians (Owner/Co-admin/Viewer) → Child profiles → Devices |
| H2 | Onboarding flow once; email code now, Sign in with Apple when paid |
| H3 | Child sign-in: profile picker, optional picture-PIN, K3 lap-mode only |
| H4 | Parent RBAC; Viewer sees approved cards only; audit log |
| H5 | Self-hosted analytics per child/device/guardian; no third-party SDK |
| H6 | Parent area via gate on devices + web dashboard on Cloudflare Pages proxying `/api/*` |
| I1 | Checkpoints where parent approval saves work; batched questions; never block |
| I2 | Separate streams a–e; one orchestrator session talks to the parent; ≤3 active sessions |
| I3 | Checkpoint list C1–C7 with target dates (owned by 13 §4) |
| F1 | Decision hierarchy: this file → RCT evidence → best practice → consensus → ADR |
| F2 | What to ask the parent (and only that); batch; placeholders |
| F3 | Missing research → targeted research → log → update spec → continue |
| J1 | Ownership map: which spec owns which topic |

---

## A. Product & audience

| ID | Decision |
|---|---|
| A1 | Family-only app. 3 kids, 2 parents. Not public, not scalable, no monetization. Optimize for polish, low cost, low ops. **Local install only, no App Store publishing** (see E1a). |
| A2 | Kids (placeholders until parent provides names): **K1** = girl, 8.5, grade 3 (כיתה ג', school year תשפ"ז / 2026-27). **K2** = boy, 5.5, gan chova (גן חובה). **K3** = toddler, 2.5 (grammatical gender TBD — parent will confirm; design must support both; until then every `T`-band line is written neutrally and the content linter enforces it). |
| A3 | Devices: iPad (shared/primary) + iPhones. Design iPad-first, iPhone-adapted. Portrait + landscape on iPad; iPhone portrait-first for menus, landscape allowed for game scenes. **iPad mini is treated as an iPhone-class device** for layout and touch targets. |
| A4 | Kids love: heroes & battles, building & creating, Roblox, Brawl Stars, Clash Royale, 99 Nights in the Forest, Temple Run, and K2 finished Astro Bot + DLC. Want avatars that look like themselves. |
| A5 | Parent (primary user of parent area) is a senior UX/product designer: holds taste approvals; not an edtech domain expert. The developer (Claude) is expected to bring domain expertise and decide autonomously. |
| A6 | **Age bands are the shared curriculum vocabulary** (from 07 §0.2, adopted by 06): `T` = 2–4 (today K3; co-play only, no text, no fail states, 3–5 min), `G` = 5–6 (today K2; voice + pictures, 8–12 min), `S` = 7–10 (today K1; short Hebrew text with voice, 15–20 min). Skills are tagged by band (+ optional grade tag), never by child; children move through the graph, nodes do not move. `packages/shared` exports `Band = 'T'|'G'|'S'` and the `Tier → Band` mapping (`K3→T, K2→G, K1→S`, re-evaluated on birthday/grade change). All curriculum specs (03–07) use this vocabulary. |
| A7 | **Touch targets are defined in millimetres and converted per device model** (`ptPerMm(modelId)` via `expo-device`; unknown model → conservative 6.42 pt/mm and a telemetry note). Visual minimum: **K3 20 mm, K2 15 mm, K1 10 mm**, parent 44 pt (HIG). Hit area (≥ visual): 25 / 18 / 12 mm. Gap between visual targets 8 / 6 / 4 mm; edge padding beyond the safe area 10 / 6 / 4 mm; no targets within 10 mm of the Dynamic Island or home indicator; system back-swipe disabled on child screens. Automated check `UX-AC-06` (Playwright) fails the build on violations. Detail tables: 09 §4. |

## B. Language & culture

| ID | Decision |
|---|---|
| B1 | **Hebrew-first.** Entire UI, narration, characters, parent area in Hebrew, RTL. English is taught as a subject inside the Hebrew app. |
| B2 | English content lives in **LTR "English islands"** (separate pages/cards), voiced by a dedicated English-only character ("one person, one language"), with a distinct visual + audio cue (jingle, color accent) when entering English. Hebrew gloss via "sandwich" (EN → HE → EN) for K2; English-only islands grow over time for K1. **Clarifications (from 05):** (a) K3: English = songs and single words in co-play only; the sandwich gloss is fixed, no fading. (b) **The Hebrew TTS voice never pronounces English**; an English word inside a Hebrew line is a clip by the English character. (c) Spatial convention: the English character stands on the **left**, the Hebrew mascot on the **right** of a shared screen (09 owns the visual detail). |
| B2a | **K2: English is oral only this year (listening, speaking, songs, TPR); English letters appear only incidentally** (never as a mastery target in K2's skill map). Decoding in English starts for K2 only after Hebrew decoding is `proficient` (C15) and not before grade 1. Rationale: gap_audit D18; K2 learns Hebrew letters this year. |
| B3 | **Grammatical gender:** every string that **addresses or refers to** a child — child screens, parent screens, weekly reports, emails, cards, and LLM (Haiku) summaries — has variants `m`/`f`/`pl` (and `n` neutral, used while K3's gender is pending). Prefer neutral constructions (infinitive, "בואו נ…", child's name). Strings addressed **to a parent** are written in impersonal/neutral Hebrew (10 COPY-1). Gender is a profile field; LLM/TTS never choose gender. Snapshot tests render every screen per profile; the content linter rejects a child-referring string without all variants. |
| B4 | **Numbers:** spoken numbers come from a hand-checked lookup table (gendered numerals, construct forms, agreement with the counted noun; `packages/content/hebrew/numerals.json`, parent spot-check Q-14), never from free TTS/LLM. Rote counting uses feminine forms (as in Israeli gan). **Written numbers in Hebrew text are digits inside an LTR isolate** (never spelled out by the LLM); word problems generated by LLM must obey the same rule. |
| B5 | **Nikud:** full nikud for all K2 text and K1 by default. **Canonical spelling for K1 texts is ktiv male (כתיב מלא, Academy rules); nikud is placed on the ktiv-male form** so word shapes stay stable through fading. K1 follows a **per-word adaptive fade track F0–F4** (04 §6: pointed → partial → unpointed per word, driven by that child's decoding evidence), with a **free "nikud lens"** (tap to reveal, never costs anything). Calendar envelope (from the Ministry fluency tool, 13 §19 #5): partial stage reached by ~mid-January 2027, most K1 text unpointed by June 2027 — the envelope guides pacing, the per-word evidence governs; a word the child cannot yet read unpointed is never faded. K2 texts are built only from lexicon IDs (04 §5). Nikud generated by Dicta Nakdan (or Phonikud), validated, human-spot-checked. Font must render nikud correctly on iOS (test early on device). `⟂VERIFY` Dicta Nakdan accepts ktiv-male input and returns it pointed without changing letters — run 50 K1 words through the API and diff letters; if not, write an alignment adapter (04 PC-04-2). |
| B5a | **Fonts (OFL only):** `font.read` = **Noto Sans Hebrew** (the only tested sans with full mark coverage incl. meteg, rafe, qamats qatan); `font.display` = **Rubik** (headings, labels, counters with `tnum`); `font.en` = **Andika** (beginning readers: single-storey a/g, distinct I/l/1). **Open Sans, Heebo and Varela Round are excluded for pointed text** (ךָ bug, missing marks — 09 research log R1). Fallback chain `font.read → Rubik → system Arial Hebrew`; a missing glyph (`.notdef`) fails the content linter (`UX-AC-03`), not the runtime. Web prototypes embed the same fonts (E1a). |
| B6 | **Best practice beats local convention.** Align topics/timing with the Israeli curriculum and school calendar, but when evidence-based global practice conflicts with common Israeli methods, follow the evidence (e.g., letter sounds before names, systematic phonics, CPA math, number-line games, spaced retrieval). |
| B7 | **Israeli/Jewish calendar.** Week starts Sunday. **Shabbat/Yom Tov pause window = [candle lighting − 30 min, havdalah + 30 min]**, computed by `@hebcal/core` (zmanim) for the parent's city (default Tel Aviv; one library for the whole app — no second NOAA implementation). **Family modes** (parent choice, default **`soft`**): `soft` = app usable, all counters/goals frozen, no events open or close, no notifications, no coins/gems earned (no "Shabbat grinding"), camp shows Shabbat scenery, levels open as free practice; `full` = app sleeps for children from candle lighting to havdalah (parent gate can open it); `off` = regular days. Holiday seasonal events (Rosh Hashana, Sukkot, Hanukkah, Tu BiShvat, Purim, Pesach, Yom HaAtzmaut, Shavuot). **The weekly parent report goes out on Friday between 12:00 and 14:00 and at least 3 h before candle lighting** (never on Shabbat); the alert queue runs every 15 min outside the pause window. Calendar file `packages/content/calendar/5787.json` is generated from hebcal and cross-checked against the Ministry calendar (13 P0-E8-S3). |
| B7a | **Quiet days** (`QD-*`, explicit list for the calendar config): **Yom HaShoah** (2027-05-04), **Yom HaZikaron** (from the evening of 2027-05-10 to 05-11), **Tisha B'Av** (2027-08-12), **Yom Kippur** (from the evening of 2027-10-10 to 10-11; if the family chose `full`, the app sleeps). Rules on a quiet day: the day **counts** toward goals; no events, no boss; celebrations capped at **T1**; no automatic chest opening; calm music only; nothing that resembles a siren. **Yom Yerushalayim = neutral day, no event** (political sensitivity). Other memorial days may be added by the parent in settings. `⟂VERIFY` dates against `@hebcal/core` output at P0-E8-S3. |
| B8 | **Security-situation sensitivity:** no siren-like sounds, explosions, alarms, war/current-events content, and **no fireworks** (including Yom HaAtzmaut — fireworks are explosions). Evening scenes: **darkness covers ≤40% of the screen** and keeps a minimum luminance (09 tokens); **for K3 only dusk, never night**. Blocked-word lists live in 12 §12.3. Optional offline calm "safe room" mode (see B8a). |
| B8a | **Bravery ladders (07 `SE.BRAVE`) never include security-situation topics** (sirens, shelters, ממ"ד). The "safe room" calm mode is **not** an exposure mechanism and its content requires professional review (OPD-19) before release. |
| B9 | **RTL direction conventions (one rule for every spec):** (a) coding/sequence strips in band `G` flow **RTL** (first block on the right); band `S` uses a vertical stack; (b) movement arrows mean **absolute screen direction**; (c) **time axes** (homeland timelines, parent charts in app, email and web) flow **RTL with explicit arrows** — past on the right, now on the left — the same rule everywhere (`chart.timeDirection = 'rtl'`); (d) **number lines, equations and math expressions are always LTR** (isolated), as in Israeli schools. `⟂VERIFY` (c) against two Israeli grade-3 homeland textbooks and the Hebrew-locale iOS Health/Screen Time charts; if both run LTR, flip (c) to `ltr` in one token and keep the rule global. |

## C. Pedagogy & ethics guardrails

| ID | Decision |
|---|---|
| C1 | **Intrinsic integration:** the learning action IS the game action (e.g., attack = solve; build = measure/multiply). Avoid quiz-between-levels. |
| C2 | **Target success bands:** K3 ≈ 90–95% (no fail states), K2 ≈ 85%, K1 75–85% with opt-in Stretch Zone. **K1's challenge dial is 0.85 / 0.80 / 0.75; 0.70 exists only inside opt-in Stretch Zone blocks** (08 §14 #5), never the 90/80/70 of the research notes. |
| C3 | **Episode lengths:** K3 3–5 min (co-play only, "lap mode"), K2 8–12 min, K1 15–20 min. **A level is 3–6 min for K2 (G3) and 6–9 min for K1**; a K1 episode = 1–2 levels + camp + evening. Parent-set daily caps. Sessions are finite episodes with a wind-down; no autoplay. |
| C4 | **Praise:** process/strategy praise only. No trait praise ("גאון", "אלוף", "חכם"), no inflated superlatives. Approved/banned phrase lists live in 09 (UX) and 12 §12.3 (`praise_trait`). |
| C5 | **No loss mechanics:** no hearts/energy, no streak loss, nothing wilts or dies, no fake timers, no nagging, no FOMO, **no resets that feel like loss** (campfire level never resets, see D4). Weekly **family goal** instead of daily streaks; auto rest days on Shabbat/holidays. |
| C6 | **No sibling comparison:** no leaderboards/side-by-side scores. Co-op with different roles per child. K1 "guide/teacher" mode is opt-in, capped, and rewards her own progress. |
| C7 | **AI for kids:** no open-ended chat for any child, at any tier. Structured, scripted, sandboxed AI interactions only; "try first, then AI" (the child commits to an attempt before the machine responds); hints not answers; AI clearly labeled as a machine ("AI • מכונה"), separate from the scripted mascot and from the player's hero (G4). Content kids see is pre-generated and validated (and parent-approvable). |
| C7a | **AI tiers (07 §0.5) are the only four ways AI touches a child:** `T0` scripted simulation (pure code/content, no network; label "machine" stays); `T1` build-time generation (Opus batch → Sonnet judge → gender/nikud linter → pack; parent approves the pack; every item tagged `machine_made`); `T2` deferred generation on parent approval (child composes a request from **closed-vocabulary chips only**, server generates overnight, parent approval queue, appears next episode; off by default in MVP, on in v1); `T3` adult co-pilot (parent types in their own AI tool from a script the game shows; the app sends nothing). **No free text from any child ever reaches the server — not even K1. No live generation is ever shown to a child without parent approval.** |
| C8 | **Not therapy:** SEL/CBT-informed activities only; emotional check-ins stored on device or not at all (see C8a); red flags surface to parents only, non-diagnostic, with referral paths (יועצת, אבחון דידקטי, קלינאית תקשורת, ריפוי בעיסוק, רופא/ת ילדים). |
| C8a | **Emotional data storage:** emotion labels are **not persisted beyond the episode**. Single exception: **local aggregate counters for 14 days** feeding red-flag rules (`RF-*`, `EMO_CHECKIN`), which are never synced raw. The only thing that may leave the device is **one derived boolean per rule ("flag raised")**, synced encrypted to the guardians' devices, and only with explicit parent opt-in (10 consent `T4b`, default off). **No ASR or LLM analysis of children's voice recordings in SEL content.** Rationale: Amendment 13 to the Israeli Privacy Protection Law treats personality assessment and mental-health data as especially sensitive. |
| C9 | **Profiling:** strengths-based, dynamic, context-tagged (not trait labels), parent-visible and correctable; kids see only a "my strengths" view; recommendations never by gender. **The 20–30% exploration share applies to the choice of interest topics and wrappers only** — not to difficulty and not to skill selection (skill selection has its own ε = 0.2 exploration inside the success band; the C2 controller is never bypassed). |
| C10 | Phygital bridge: printable missions/worksheets (free handwriting for K2 letters), offline missions, family rituals. In-app accuracy is not the success metric; periodic paper probes + teacher feedback are. |
| C11 | **One definition of mastery, one review ladder (all subjects).** A skill is `proficient` when: ≥85% correct over the last 8 attempts, the last 3 without hints, spread over ≥2 calendar days (`DayKey`, Asia/Jerusalem), **and** a delayed check ≥3 days later is correct. Review ladder **1 / 3 / 7 / 14 / 30 / 60 days**; a miss drops one step (two steps from the 30-day step and above); **no level is ever lowered from a single session**. 08 §5.1 is the algorithmic implementation (Elo + BKT logging) and wins on detail; this row is the observable contract that 03–07, 10 and the photo pipeline all use. Subject specs may add entry gates (e.g., 03 `O.REP` pictorial + abstract) but never a weaker rule. |
| C12 | **No visible clock for any child, ever.** Response time is measured silently. It enters scoring **only for K1, only in fluency skills already `proficient`, only inside an opt-in speed round whose result is shown at the end** (08 §4.5 HSHS). RT is off in scoring for anyone still in gan (`grade < g1`, a grade rule, not an age rule — K2 turns 6 mid-year). RT is still used silently for rapid-guess filtering (08 §4.4). The photo-to-game pipeline and all generators obey this. |
| C13 | **Physical safety for real-world/offline missions:** every mission carries safety tags from a fixed set — `adult-required`, `no-flame`, `no-mains`, `choking-hazard-T`, `outdoor-with-adult`, `food-allergy-check`, `photo-on-device`. No open flame, no mains electricity, nothing that could be swallowed. **While a child under 3 lives at home (derived from K3's birthdate — with the placeholder age 2.5 at 2026-11 that is until ~May 2027), no small parts or small magnets in any mission for any child.** Content linter rejects a mission without tags. |
| C14 | **Celebration budget:** celebrations + transitions ≤ **15%** of episode time (telemetry `celebration_played`, `stop_ritual`; AC in 01/09). **Answer feedback ≤ 0.8 s and never blocks input.** Tiers and caps: T0 0.8 s, T1 1.5 s, T2 3 s, T3 6 s, T4 10 s; skippable after 1 s (T2/T3) or 2 s (T4); flash ≤ 3 Hz; screen shake K3 0 px / K2 4 px / K1 8 px. 02's micro/small/medium/large = T0/T1/T2/T3. |
| C15 | **Canonical skill-level names are 08's:** `new / beginner / advancing / proficient / master` = חדש / מתחיל / מתקדם / שולט / מאסטר (child view: זרע / נבט / שתיל / עץ / עץ פורח). 02's economy states (`locked/ready/introduced/practicing/proficient/mastered/retained`) are **internal economy triggers**, mapped once in `packages/shared/levels.ts` (`introduced→new`, `practicing→beginner|advancing`, `proficient→proficient`, `mastered|retained→master`; `locked/ready` are pre-states with no level). Parents and children never see two scales. |

## D. Gamification & economy (parent's explicit request — keep, design ethically)

| ID | Decision |
|---|---|
| D1 | Currencies: **מטבעות (coins)** = soft currency earned from mastery events & effort (not raw time); **יהלומים (gems)** = rare, earned only at milestones: **skill mastered, boss beaten, area completed, area boss, weekly family goal, holiday event, end of a season track** (02 owns the amounts; **levels 25/50 etc. do not give gems**). **No real money, ever:** no transactions and no real monetary value inside the app; no purchases of currency. Learning **about** real money (₪ coins and notes, grocery play, price comparison; 03/06) is allowed and encouraged. **The only time-dependent source in the wallet is the small, capped weekly savings bonus (02 §7.5), a declared pedagogical exception; nothing else grows with time.** **"Sparks" (`spark`) are a non-persistent in-episode resource** (breadcrumbs for G1): they reset at episode end and never convert to coins — not a third currency. |
| D1a | **No coins or gems for emotional check-ins, kindness missions, bravery-ladder steps or reflections.** These are recognised by skill-named badges, the family garden, or a gem at the **end** of a ladder that was never announced in advance (Deci et al. 1999; prevents "emotion faking for coins"). |
| D2 | **Chests/treasures (תיבות/אוצרות):** contents are fixed at earn time and shown (or guaranteed) before opening; "rarity" reflects what earned it, never chance. Big juicy reveal animation is allowed and encouraged. **Multi-tap reveals with visible "upgrade steps" are allowed only when each step represents an effort mark already earned** (01 §16.2). **The random "capsule machine with random cosmetics" recommendation in `astrobot_joy_gamification_psychology.md` §Q6 is rejected**; the capsule machine shows the next (deterministic) capsule. |
| D3 | **Cosmetic-only shop** (avatar outfits, emotes, base decorations, pet accessories, trails). **The catalogue is fixed; only a weekly showcase (changes on Sunday) rotates; no item ever leaves the catalogue.** Wishlist + save-up goals; no countdowns, no discounts, no "expires" (02 config keys `discount/expires/availableUntil/streak` are forbidden). |
| D4 | Levels/XP per child, skill badges named after the skill learned, collectible pets that **evolve by the number of skills mastered in a domain** (never die; a completed species set gives an accessory + chest, not evolution), season album that never expires, home base building as reward sink, family campfire upgrades as shared sink. **The campfire level never resets between seasons** (`resetsPerSeason: false`; a new season adds a colour ring). |
| D5 | K3: stickers only (no currencies, no visible levels; **a hidden XP counter exists for the parent dashboard only**). K2: coins + simple chests + pet. K1: full economy (coins, gems, chests, shop, pass-like season track without paywall). |
| D6 | Real-world rewards set by parent, behind parent gate, framed as family activities tied to effort goals; never announced in advance as contingent prizes for specific tasks; **never priced in coins** (the v1 report's "park trip = 500 coins" example is withdrawn). |
| D7 | **02 owns every economic number** (earn values, caps, prices, chest contents, XP/level pace, season tiers, savings bonus) in `packages/engine/economy/economy.config.json` + zod schema + simulation; 01 §11.2 and any other spec refer to 02 instead of restating numbers. Building thresholds are 02's stretched values (§5.5). |

## E. Technology stack

| ID | Decision |
|---|---|
| E1 | **App:** Expo SDK 58 (React Native 0.88, New Architecture, Hermes), TypeScript strict, `expo-router`. **Build is phased:** **Phase 1 (now):** built on the parent's Mac (`npx expo run:ios --device`, free personal-team signing, through Claude Remote Control); **Phase 2 (after the paid Apple Developer Program):** EAS Build in the cloud + EAS internal/ad hoc distribution, optional TestFlight for parents. **OTA via EAS Update in both phases** (free Expo account; channels `preview` → `family`; `runtimeVersion.policy = "fingerprint"`). |
| E1a | **Local-first install (parent request): no App Store publishing.** Parent answer (2026-11-02): has a Mac; no paid Apple Developer Program yet. **Primary path from day one = development build (`expo-dev-client`) over cable via the parent's Mac through Claude Remote Control, free personal-team signing** (7-day expiry, max 3 apps/device; re-signing automated by `tools/mac/resign.sh` ≤ every 6 days, launchd daily check). After install, JS/asset changes flow via the dev client / EAS Update without re-cabling until the next re-sign. **Expo Go is not a path** for an SDK 58 project (App Store Expo Go ≤ SDK 54; `⟂VERIFY` at build time by checking the Expo Go App Store listing's supported SDK — even if compatible it lacks Rive/Nitro). **Phase 2** = EAS internal distribution (ad hoc, 1-year provisioning) once the parent buys the paid program (OPD-25; trigger: re-signing becomes a burden or push/Sign in with Apple is needed). **Phase 1 has no APNs, no Sign in with Apple, no App Attest** (free team lacks the entitlements) — see E3, E7, H2. **Concept prototypes before and between device builds are web artifacts (Expo web export) viewed on the iPad in Safari; they must set `dir="rtl"`, embed the B5a fonts, and ship a sprite fallback for Rive**, otherwise the parent sees wrong nikud/direction. |
| E1b | **3D layer:** three.js + `@react-three/fiber@9` (native entry) + `expo-gl` + `expo-asset` (same code on web for Playwright); custom kinematics + `three-mesh-bvh` (no Rapier/WASM on Hermes); fallbacks: three.js in `react-native-webview`, then 2.5D Skia isometric; `react-native-filament` as native-only plan B for perf. Budgets: ≤80 draw calls, 100–150k visible tris, **WebP/PNG textures ≤1024² (hero ≤2048²); no KTX2/Draco/meshopt on native**, toon/Lambert, blob shadows, pixel ratio ≤2. Controls: tap-to-move + auto follow camera; auto-run tap-to-jump mini-levels; floating joystick K1 only; tilt as bonus. **Hebrew text only in the 2D RN HUD — never as text inside GL/Skia canvases on child screens** (nikud rendering is verified only in RN text). **In-world glyphs (a single letter/syllable as a learning object, e.g. a rune on a stone) are allowed only as pre-baked image textures** from `tools/glyph-baker` (nikud validated at bake time). ~10-day tech spike with 4 go/no-go gates (see `tech_3d_world.md`, 13 C2); must verify R3F v9 on RN 0.88 on day 1. |
| E2 | **Rendering/animation:** `react-native-reanimated` 4 + `react-native-worklets`, `react-native-gesture-handler`, `@shopify/react-native-skia` (game canvases, tracing, particles), `@rive-app/react-native` (characters, mascots, state machines; web fallback `@rive-app/react-canvas`), Lottie only for one-shots if needed. Mini-games = Skia canvas + Reanimated frame loop (no game engine lib). |
| E3 | **Device:** `expo-haptics`, `expo-audio` (layered music/SFX/VO), `expo-camera`/`expo-image-picker` (photo-to-game), `expo-image`, `expo-localization` (`forcesRTL: true`), `expo-device` (model → `ptPerMm`, A7), **`expo-sensors` (DeviceMotion: tilt, shake, "device in hand" detection before any jump moment — G6 safety)**, **`expo-sqlite` + `expo-sqlite/kv-store` as the only local stores (no MMKV — Nitro surface without need)**, `expo-local-authentication` (Face ID parent gate), `expo-notifications` **local-only in phase 1** (no remote push without APNs; parent alerts go by email and the in-app inbox until phase 2). **Camera pose** (on-device Vision module) only in a dev build, later phase. **Optional game-controller support** (iOS GameController: DualSense/MFi via a small Expo module; stick = move, X = jump, haptics → controller rumble) is allowed **after the vertical slice (C4)**, never required to play, and never a prerequisite for any mechanic. |
| E4 | **Testing from Linux:** web export + Playwright screenshot tests (iPhone + iPad viewports, RTL, each child profile/gender, A7 touch-target check `UX-AC-06`), Jest + Testing Library for logic, engine determinism replays (E11a). **Playwright baselines are generated only inside the pinned Playwright Docker image.** **Maestro golden-path flows run on the Mac's iOS Simulator via Remote Control.** Real-device checks by parent only for haptics, Rive, camera, nikud rendering, audio. Pin versions; RTL screenshot regression suite. |
| E5 | **Backend:** Cloudflare Workers Paid ($5/mo) — one Worker `olam-api` (TypeScript, Hono), **D1** (SQLite) for family data & event log, **R2** for content packs, TTS audio cache, photos (private bucket, EXIF stripped, auto-delete 30 days), **Cron Triggers** (nightly content generation; weekly report scheduler `*/15 6-12 * * *` computing the Friday `send_at` per B7; alert queue every 15 min), **Durable Objects** (`CoopRoom`, WebSocket Hibernation API, one room per multi-device co-op session, server-authoritative order, auto-close after 15 min idle — Workers Paid includes it), Rate Limiting binding, **Cloudflare Pages** for the parent web dashboard, **Resend** for email codes and reports. Deployed by `wrangler` from GitHub Actions CI (E13). **Timing:** the vertical slice (C4) runs **fully local** (SQLite, packed content, JSON export); Cloudflare **staging is deployed at C3 with no child data**; production from MVP sprint 3 (~February 2027). Without Cloudflare approval, multi-device co-op (01 CO-3) is disabled and co-op is same-device only. |
| E6 | **Sync:** local-first. Each device has SQLite + append-only event log; push/pull via Worker; last-write-wins for profile/settings; events merged by ID and replayed in canonical order (E11a). Content packs + audio are read-only downloads verified by a hash manifest. |
| E6a | **Sensitive derived data stays in the family.** Early red flags, context-dependent tendencies, EF calibration and emotional check-ins are computed **on device**, synced **encrypted only to guardian devices** (never in plain text through analytics tables), and **never sent to any AI vendor** — not even to Haiku for narrative summaries (summaries receive only the non-sensitive progress facts). Haiku/Opus/Sonnet prompts carry labels `K1/K2/K3`, never names (E8). |
| E7 | **Auth:** **Phase 1 (free team):** guardians authenticate by **email one-time code** (6-digit code + link via Resend, 10-min TTL, 5 attempts, rate-limited, always-202 to avoid enumeration); the owner's very first sign-in uses a **one-time bootstrap/setup code** the agent hands over in chat. **Phase 2 (paid program): Sign in with Apple is added**; all providers map to the same Guardian identity. Children are profile rows (avatar + picture-PIN), not accounts; devices paired by a guardian (pairing token/QR, 10-min TTL, rate limits); **App Attest deferred to phase 2**. **Parent gate = Face ID or the app's own parent PIN (no arithmetic gates — K1 solves them). On `shared-kids` devices the device passcode is never accepted as a fallback (`disableDeviceFallback: true`) because the kids know the iPad passcode**; `parent-only` devices may allow it. |
| E8 | **AI (server-side only, keys in Worker secrets, per-day $ cap):** Claude Opus 5.5 (Batch API) for Hebrew content generation; Claude Sonnet 5.5 for QA judging and for worksheet/photo extraction (vision) with mandatory parent confirm; Claude Haiku 5.5 for scripted Socratic companion turns and parent summaries; Dicta Nakdan / Phonikud for nikud. **Handwritten worksheet pages:** default single route Sonnet 5.5; a parallel Opus 5.5 call with code-side reconciliation is enabled **only if** the 30-page family evaluation set (OPD-20, 12 §10) shows it improves word coverage; parent confirm is mandatory either way; optional phase-2 second pass with Mishkefet-v1 in a container (private use, CC-BY-NC-SA weights). **The Batch API is not zero-data-retention (29-day retention): batch requests never contain real names, photos or children's free text — labels K1/K2/K3 only.** Anthropic usage policy for minor-facing products applies: child-safety system prompt **if/when Anthropic provides one (a slot exists in the prompt system); until then the internal Safety Core + validators + parent queue (12 §8)**, safeguards, disclose AI. Structured outputs (JSON schema) everywhere; math answers computed in code; distractors from error catalogs in code. |
| E8a | **Vendor terms constraint (child-directed apps):** Gemini API / Google GenAI terms bar apps "directed towards or likely to be accessed by" under-18s; ElevenLabs prohibits under-18-targeted apps without written approval and bans child-like voices. Therefore: **no Gemini / Google GenAI / Vertex anywhere (including "Nano Banana" image models and Gemini TTS)**; ElevenLabs (TTS, SFX, Music) only if the parent obtains written approval (OPD-16); otherwise terms-compatible vendors. Re-verify terms at build time. |
| E9 | **Voice:** all narration pre-generated & cached in R2 (no live TTS for kids; **no `AVSpeechSynthesizer` for child narration**). **Photo-to-game (12 P2) audio is rendered server-side after the parent approves the extracted content and before the game opens**, stored in R2, loudness-normalised nightly. Hebrew TTS chosen by a scripted **bake-off** (protocol in `voice_tech_he_en.md`) among terms-compatible candidates: Azure he-IL (Hila/Avri), Cartesia Sonic 3.6, MiniMax Speech 2.8, ElevenLabs v4 (only with approval), Google Chirp 3 HD (**excluded**, E8a). Avoid OpenAI gpt-4o-mini-tts (shuts down 6 Jan 2027). Text preprocessed with Phonikud (nikud + stress), numbers from the B4 table, gender variants pre-rendered. English = native US voice from the same vendor; the Hebrew voice never pronounces English (B2). **Human-recorded audio banks (never TTS):** **Hebrew `HREC` ~300 clips** (23 consonants, 5 vowels, ~120 CV combinations at two speeds, common closed syllables; recorded by the parent or a native narrator, ~45–60 min, in-app script; temporary TTS stand-in for combinations only, flagged `temporary`); **English session ~2.5 h by a native en-US speaker** (~44 phonemes, 26 letter names, ~60 nonsense words for NWF, vocals for ~27 songs + sound chants; fallback Wikimedia CC BY-SA IPA clips for phonemes, spoken-not-sung songs until recorded). **No production score for any child's speech, ever.** Hebrew reading-aloud check for K1 only: on-device ivrit.ai Whisper large-v3-turbo fine-tune, matched against known target text, never punitive, never stored as a mastery datum. No ASR for K3; none required for K2. **English ASR (phase 2, K1, parent present) drives only the character's reaction, is not stored as mastery data, and no child audio is ever sent to a cloud pronunciation-assessment service.** No live voice AI for K2/K3. Parent voice: recorded in-app (~20 min script); optional consented clone later. |
| E10 | **Assets:** **Recraft (paid subscription — free tier makes images public)** for SVG icons/UI/avatar parts; **GPT Image** (offline, developer account, **no child data, photos or drawings in prompts unless OPD-10 consents**) for scenes/characters; hand drawing by the parent-designer as the alternative; Rive for rigging/animation; audio from Kenney.nl CC0 + Freesound (CC0 only) + home foley recordings; **ElevenLabs SFX/Music only with written approval (E8a)**. Designer approval gates (G1–G7 in 12 §9): moodboard, character sheets, avatar parts sheet, Rive rigs, **rigged 3D hero model (added if ADR-0005 selects 3D; at VS-E9-S3)**, audio review. Asset provenance/license log (`generator`, `promptHash`, `plan: paid`, license) feeds the in-app credits screen. |
| E11 | Repo: monorepo in this repository's branch or a new repo (parent to choose; default new private repo `family-learning-game`), pnpm workspaces: `apps/mobile`, `apps/parent-web`, `workers/api`, `packages/engine` (adaptive engine, pure TS), `packages/content` (schemas, packs, generators), `packages/ui` (design system), **`packages/shared` (wire contracts, RBAC, event types, band and level mappings — A6, C15)**, **`packages/i18n`**, **`packages/api-client`**, `tools/` (pipelines, `glyph-baker`, `mac/resign.sh`). |
| E11a | **The engine is deterministic and event-sourced.** Time and RNG are injected; events are replayed in the canonical order `(ts, deviceId, eventId)`; after a merge the state is recomputed from the last checkpoint. Two devices replaying the same log must produce the identical child model (θ) — a CI replay test asserts it. |
| E12 | **Binding playbooks:** `research_notes/תוכנית הוליסטית לאפליקציית הלמידה/performance_reliability_playbook.md` (the 25 performance rules §11A, lint bans §11D, release checks §11E) and `quality_bug_prevention_playbook.md` are binding engineering references for 11 and 13; CI enforces the lint bans and the memory/frame-time smoke per SDK bump. A spec may tighten them, never loosen them. |
| E13 | **Cloud-environment network allowlist** (parent action, optional: CI deploys regardless). Hosts the agent needs for verification and CLIs: `docs.expo.dev`, `expo.dev`, `api.expo.dev`, `u.expo.dev`, `qr.expo.dev`; `developers.cloudflare.com`, `api.cloudflare.com`, `*.workers.dev`, `*.pages.dev`; `appleid.apple.com`, `rive.app`, `hono.dev`; `playwright.azureedge.net`, `cdn.playwright.dev`; `registry.npmjs.org`; the chosen TTS vendor API host; `api.anthropic.com` (only for the Worker, via secrets); content verification: `pop.education.gov.il`, `meyda.education.gov.il`, `benyehuda.org`, `www.hebcal.com`. A blocked host is logged in `docs/research-log/` and listed under "allowlist requests" in the weekly status; never bypassed. |

## H. Identity, users, devices & parent admin (added 2026-11-02)

| ID | Decision |
|---|---|
| H1 | **Household model:** one `Family` (tenant) → `Guardian` users (2 parents; the first = **Owner/Admin**, can invite the 2nd as **Co-admin**; optional read-only **Viewer** roles e.g., grandparents) → `Child` profiles (K1–K3; no credentials, no email) → `Device`s (each paired to the family by a guardian; device type iPad/iPhone/iPad-mini; mode: `shared-kids`, `single-child`, `parent-only`). |
| H2 | **Onboarding/registration (once):** owner installs → signs in with the **one-time setup code**, then **email one-time code** (phase 1; **Sign in with Apple becomes an additional option in phase 2**, E7) → creates family → adds children (name, nickname, birthdate, grammatical gender or `unset`, grade/gan, dominant hand, avatar builder) → sets parent PIN + Face ID → pairs devices via QR/6-digit code → chooses per-device allowed children → placement expeditions start per child. Co-admin joins by invite link/QR + email code. ≤5 min for the parent; resumable mid-way. |
| H3 | **Child sign-in on device:** profile picker with avatars (no passwords); optional picture-PIN per child (K1 only by default) to stop siblings playing as each other; K3 profile only launchable in lap mode/parent-present (Guided Access recommended on the shared iPad). Fast user switching on shared iPad; per-child resume state. |
| H4 | **Parent controls (RBAC):** Owner/Co-admin can edit children, settings, caps, quiet hours, Shabbat mode, subject weights, content approval, rewards, device management (rename, restrict children, remote lock "bedtime", unpair/wipe), data export/delete. **Viewer: sees only cards a guardian (or K1 for her own) has approved; no scores by default; a Viewer's voice reply reaches a child only after guardian approval** (ClassDojo approve-before-share pattern). All admin actions audit-logged. |
| H5 | **Analytics per child, per device, per guardian view:** every event carries `familyId, childId, deviceId, sessionId, appVersion`; dashboard filters by child (primary) and device (secondary: usage per device, last sync, battery-safe). Self-hosted events only (D1), no third-party analytics SDK. Retention policy defined in 11/10 (flags never in the weekly payload; raw emotional data never synced — C8a). |
| H6 | **Parent area access:** from any family device via parent gate (Face ID/app PIN), plus a lightweight **web dashboard** (`apps/parent-web` on Cloudflare Pages, **proxying `/api/*` to the Worker through a service binding so cookies are same-origin**), using the same Worker session (email code now, Sign in with Apple later). **Cloudflare Access in front of the dashboard is an optional extra layer, not required** (one login, not two). Weekly report delivery in phase 1 = email (Resend) + dashboard + in-app inbox; push only in phase 2. |

## I. Delivery checkpoints & thread strategy (added 2026-11-02)

| ID | Decision |
|---|---|
| I1 | Build proceeds through **natural checkpoints** where the parent's approval saves wasted work/tokens; between checkpoints the agent works autonomously without ping-pong. Each checkpoint ends with a short artifact (interactive prototype/page, screenshots, or a decision list) and at most 5 batched questions, each with a default (see "Open parent decisions" below). Never block on an answer. |
| I2 | Separate work threads/sessions per stream to keep context clean: (a) Product concept & creative direction (world, hero, characters, style frames) — prototypes as artifacts; (b) Game-type prototypes (mechanic playables); (c) Tech foundation (repo, CI, backend, auth/identity); (d) Content production (curriculum packs, audio); (e) Integration & polish. Streams (a) and (c) run in parallel. **The parent talks to one orchestrator session only** (holds `STATUS.md`, batches questions per checkpoint, sends the weekly status); **at most 3 sessions are active at once**; a new session boots per 13 §6.2. |
| I3 | **Checkpoints (owned in detail by 13 §4):** C1 concept & creative direction (~11.11.2026) · C2 mechanic prototypes + 3D spike with 4 gates (~20.11.2026) · C3 tech foundation & identity on device (~27.11.2026) · C4 vertical slice on device (alpha 6.12.2026, done 3.1.2027) · C5 content production sample (~24.1.2027) · C6 MVP in daily use (~29.4.2027, Pesach evaluation) · C7 v1 polish (30.6.2027). Dates slip; order and gates do not. |

## G. Experience pillars (creative direction — added 2026-11-02 after parent input)

K2 finished Astro Bot (PS5) + DLC and adores it; all kids love 99 Nights in the Forest. The game must feel like a joyful, explorable adventure — not an app with exercises.

| ID | Decision |
|---|---|
| G1 | **Joy of discovery first.** An explorable world (3D or convincing 2.5D — decided by the tech spike in `tech_3d_world.md`, ADR-0005 at C2) with dense hidden secrets: puzzle pieces, lost friends to **rescue**, hidden rooms, secret levels. Curiosity is always rewarded within seconds. |
| G2 | **Rescue & collect loop:** each level hides N lost friends (original characters) + collectible pieces; finding/freeing them requires a learning action (solve, read, count, say in English, sort). Rescued friends return to the **hub**, which visibly fills with life (they build, play, cheer) — the hub is the trophy room. Collection display room with silhouettes of missing items (endowed progress / goal-gradient, never FOMO). **Rescued friends and mosaic pieces are non-spendable counters: they unlock bonus content only (camp scenes, secret levels, capsules) and never gate the curriculum path** — a child who struggles to find secrets never gets stuck in learning. |
| G3 | **One gimmick per level:** short levels (K2: 3–6 min; K1: 6–9 min, C3) each built around one fresh power-up/gimmick tied to a skill (e.g., a "number-jet" that flies exactly N tiles, a "letter-magnet" that pulls only the right sound). Variety > length. Kishōtenketsu: introduce → develop → twist → conclude. |
| G4 | **Original hero:** a lovable small **"toy-suit with a living spark"** (working name פנסון; original IP, never Astro Bot or any existing character). **It is never called a robot or an AI, has no gears and no screen-face, and is a silent vessel the child's avatar pilots/customizes** — no dialogue of its own. The **gear motif, boxy shape, screen-face and the "AI • מכונה" label are reserved for the machine family** (07 `ROBOT`/`BOTS`, the AI helper גלגלון) so that C7's "AI is a labelled machine" never blurs with the beloved hero for K2/K3. Hero and cast co-designed with the kids (their drawings/ideas become parts). |
| G5 | **Day/night co-op (99-Nights-inspired, non-scary):** day = explore, gather, rescue; evening = family defends/upgrades the campfire together with learning-powered abilities. Lost friends to find in the forest. Evening scenes obey B8 (≤40% darkness, K3 dusk only). |
| G6 | **Learning through movement:** regular embodied moments (jump/lean/clap/act-out/hop a floor number line, tilt to steer), device-sensor (`expo-sensors`, E3) and (later) on-device camera pose, always with a seated alternative and safety rules (device-in-hand check before jump moments; ≤2 movement moments per episode for K1/K2, 1 for K3; home constraints from OPD-14). See `learning_through_movement.md`. |
| G7 | **Feel ("juice") like a console game:** tight controls, satisfying physics, haptics on iPhone, layered sound, playful animation; every interaction delightful within the C14 celebration budget. Gamification grounded in SDT, flow, curiosity, goal-gradient, collection, ownership — white-hat drives only (see `astrobot_joy_gamification_psychology.md`). |

## F. Autonomy protocol (how the developer decides)

| ID | Decision |
|---|---|
| F1 | Decide autonomously using this hierarchy: (1) this file + specs (ownership per §J), (2) RCT/meta-analytic evidence, (3) best-in-class product practice, (4) practitioner consensus, (5) developer judgment with a logged rationale in `docs/decisions/` (ADR format). |
| F2 | Ask the parent ONLY for: personal assets/info (names, appearance, voices, photos of school material), account creation / payments / API keys, designer taste approval gates (E10), irreversible or outward-facing actions, and genuine value judgments about their family (e.g., religious observance level). Batch questions (≤5 per checkpoint); never block on them — use the defaults in "Open parent decisions" and `TODO-PARENT:OPD-nn` placeholders and proceed. |
| F3 | When research is missing mid-build: run targeted research, record findings in `docs/research-log/`, update the relevant spec, continue. |

## J. Cross-spec ownership map (added by Fable review)

| ID | Decision |
|---|---|
| J1 | **One owner per topic; others reference, never restate.** World, levels, areas, hub/camp, event calendar, boss story, mechanics catalogue (`MX-*`), celebration choreography → **01**. Every economic number, chest contents, shop bands, season tiers, savings bonus → **02**. Subject skill graphs, item templates, error catalogues, placement seeds → **03 math, 04 Hebrew, 05 English, 06 extra subjects, 07 future skills/SEL/AI literacy**. Engine algorithms, level names (C15), subject weights (`EngineConfig`), red-flag rule engine → **08**. Tokens (fonts, touch, colour, motion), copy rules, accessibility → **09**. Parent screens, reports, consent model (`T4b` etc.), notification channels → **10**. Stack versions, schemas, API, auth, CI, install paths → **11**. AI/voice pipelines, vendor terms, asset generation, prompts → **12**. Checkpoints, sprints, QA, parent question batching → **13**. When two specs disagree inside a topic, the owner wins unless this file says otherwise. |

---

## Open parent decisions (deduplicated across 00–13; each has a default so nothing blocks)

| OPD | Question | Default until answered | Needed by | Source refs |
|---|---|---|---|---|
| OPD-01 | Children's names, nicknames, birthdates, grade/gan, dominant hand; a 3-second recording of how each name is said; English spelling of the names | Placeholders `{K1}/{K2}/{K3}`; mascot greets without a name | Week 1 | A2; 01 §19 #1; 04 §17 #1; 05 §19 #3; 10 Q-01; 13 Q-01 |
| OPD-02 | K3's grammatical gender | `n` (neutral) everywhere in band `T` | Week 1 | A2, B3; 13 Q-16 |
| OPD-03 | Repo choice: this repo's branch or new private repo | New private repo `family-learning-game` | Week 1 | E11; 13 Q-03 |
| OPD-04 | Shabbat/holiday mode (`soft`/`full`/`off`) + city for zmanim + religious-observance level for content (Shabbat Queen, blessings) | `soft`, Tel Aviv; content framing cultural-traditional, neutral | Week 1 | B7; 01 §19 #2; 02 §15 #1; 04 §17 #5; 06 P-06-5; 07 P-03; 10 Q-02/Q-04; 13 Q-10 |
| OPD-05 | Owner's and co-admin's emails; buy a domain (~$10/yr) for Resend email codes and reports | No domain → `*.workers.dev`, no email until bought; bootstrap code for sign-in | Week 1 (domain by C3) | E7; 10 Q-05; 11 §15.1 #5; 13 Q-28 |
| OPD-06 | Mac details (model, macOS, ≥40 GB free) and device inventory (models, iOS, shared vs single, Pencil, headphones) | Perf budgets assume iPad (264 ppi) + iPhone 2020+; shared iPad = `shared-kids` | Week 1 | E1a, A7; 13 Q-22/Q-27 |
| OPD-07 | Weekly sign-off day/time for the parent; family boss day | Sign-off Friday 12:00 (with the report); boss day Thursday | Week 1 | B7; 01 §19 #6; 13 Q-07/Q-24 |
| OPD-08 | Accounts and keys: Expo (free) token; Cloudflare Workers Paid + scoped API token; Anthropic workspace with monthly spend limit (~$30) + key; Azure Speech (optional Cartesia/MiniMax) for the bake-off; Recraft paid subscription; OpenAI (assets only); Rive; Resend | Phase 0 runs without them: local-only VS, web artifacts, TTS stand-ins | Expo ~12.11; Azure ~12.11; Anthropic ~16.11; Cloudflare ~20.11 (staging), must by 1.2.2027 | E5, E8, E9, E10; 11 §15.1; 12 §11 #1; 13 Q-04/Q-09/Q-11/Q-13 |
| OPD-09 | Children's interests: favourite characters (private use), animals, topics, what exactly they love in Roblox/Brawl Stars/99 Nights/Astro Bot, what is funny, what is scary (esp. K3) | Generic interest seeds; C9 exploration finds the rest | Before C1 (~9.11) | C9, A4; 13 Q-08 |
| OPD-10 | Appearance description per child (no face photos) for avatars; children's drawings (3–5 each, hero + lost friend) and consent to send drawings to the image model | Default avatar parts; drawings not sent (consent = no) | Drawings by C1; appearance by C3 | G4, E10; 13 Q-02/Q-21/Q-30 |
| OPD-11 | Taste gates (E10 / 12 G1–G7): world name, hero, 8-character cast, moodboard, chest sheet, 6 pets × 5 stages, treasure room & camp style, Rive rigs, audio, parent-area screens (PA-HOME/CHILD/MAP) and email template | First defaults used until approved | Per checkpoint schedule | A5, E10; 01 §19 #4; 02 §15 #4; 10 Q-10; 12 §11 #7 |
| OPD-12 | Parent voice recordings (~20 min script: names, 15 praise lines, good-night; optional Hebrew sounds) | No recording → TTS is never used for names; lines without a name | By C3 | E9; 01 §19 #3; 12 §11 #4; 13 Q-20 |
| OPD-13 | Hebrew phonics bank `HREC` (~300 clips, ~45–60 min) recorded by the parent or a native narrator | TTS stand-in for combinations only, flagged `temporary`; isolated consonants unavailable until recorded | By C5 | E9; 04 §17 #2 |
| OPD-14 | Movement constraints at home (space, neighbours, hours) | Seated alternative always offered; jump moments off on iPhone-in-hand | By C2 | G6; 13 Q-29 |
| OPD-15 | Native en-US speaker for the ~2.5 h English recording session (acquaintance or paid ~$100–250), or approve the Wikimedia fallback | Wikimedia CC BY-SA phoneme clips; songs spoken, not sung | By C5 | E9; 05 §19 #5; 12 §11 #5 |
| OPD-16 | ElevenLabs: send the written-approval request (template in 12 §11 #2)? | Not sent → no ElevenLabs anywhere | Day 1 (answer takes time) | E8a; 12 §11 #2; 13 Q-12 |
| OPD-17 | Bake-off listening session (2–3 h, both parents; 10-min A/B with K1 and K2) | Agent picks by objective metrics (MOS proxy, nikud accuracy) and flags "unheard by parent" | ~12–16.11 | E9; 12 §11 #3 |
| OPD-18 | Current levels & concerns: does K1 read unpointed? times-table status? which letters K2 knows? what K3 says/counts? concerns about speech/attention/dyslexia | Placement expeditions start from `unknown` priors | By baseline (~10.11) | C11, 08; 13 Q-23 |
| OPD-19 | A child psychologist (CBT, Israeli) to review SEL content and the Hebrew emotion-word lists; the "safe room" mode | SEL only in co-play mode (07 AC-20); lists as proposed; safe room not shipped | By C5 | C8, B8a; 07 P-01/P-02; 10 Q-09 |
| OPD-20 | What is taught this month in class/gan (math, Hebrew, English, gan topic, letter of the week), photos of 5–10 workbook pages (no names/faces) + TOC of K1's math book and series name; 30 real worksheet photos (20 printed, 10 handwritten) for the P2 evaluation set | Default yearly plans in 03–06 §"what they learned this month"; ×/: notation defaults; P2 eval deferred | ~16.11 then monthly | C10, E8; 03 §16 #1; 04 §17 #3–4; 05 §19 #1–2; 06 P-06-7; 12 §11 #6; 13 Q-05/Q-06 |
| OPD-21 | Teacher inputs: K1 reading/fluency program and start-of-year fluency result; a date for "times table by heart"; K2's future school: print vs cursive in grade 1 | 04 §8.4 defaults; times table by end of May; print | Pesach (teacher summary only, 13 Q-31) | 03 §16 #2; 04 §17 #3–4 |
| OPD-22 | Family: grandparents as Viewers (names, channel email/app) and in the game?; family members as the kids call them; pets (name, kind, look, sounds); places (home, rooms, grandparents', gan, school, park — photos without people) | No Viewers; generic "family member"; no pets; generic places | VS / MVP | H1, H4; 01 §19 #5; 06 P-06-2/3; 10 Q-07; 13 Q-17/18/19 |
| OPD-23 | Real-world family-reward menu (5–8 activities) and whether K1 has real pocket money; include K3 in the family goal? | Placeholder activities from v1 report table 15; no pocket money (F missions in-game only); K3 included (goal = 2 co-play days) | By MVP | D6; 02 §15 #2–3; 06 P-06-6; 07 P-07; 13 Q-25 |
| OPD-24 | Sibling dynamics (who competes with whom, what frustrates, sensitive topics) | Co-op roles by age only | By VS | C6; 13 Q-26 |
| OPD-25 | Paid Apple Developer Program ($99/yr) — only when a trigger fires (re-signing burden, push, Sign in with Apple, App Attest) | Free personal team; phase 1 constraints (E1a/E7) | On trigger | E1, E1a; 13 Q-15 |
| OPD-26 | Weekly goals and daily caps per child (10 §4.2, §6.13); bedtimes per child | 10's values; 19:30 / 20:00 / 20:30 | By VS | C3; 10 Q-04/Q-08 |
| OPD-27 | Consent switches: local voice recordings (acting, wonderings, "my idea"; auto-delete 30 d); red-flag nudge and "flag-only" sync between parents' phones (`T4b`); "my body is mine" module (H.BSAFE.01) and the trusted adults to name; family photos for G.FAM.01 (on device) | Recordings on (local only); nudge on in parent area, sync off; body-safety module off; avatars instead of photos | By MVP | C8a, E6a; 06 P-06-2/4; 07 P-05/P-06; 10 §18 Δ3 |
| OPD-28 | Town/neighbourhood (for G.TOWN) and family origins / interviewee for G.FAM.02 | Generic "our town" content; generic family member | v1 | 06 P-06-1/3 |
| OPD-29 | Report wording: "ארוחת שישי" vs "ארוחת שבת"; show the Hebrew date?; Telegram channel? | "ארוחת שישי"; Hebrew date off; Telegram no (P2) | By MVP | 10 Q-03/Q-06 |
| OPD-30 | Which AI tool the parent uses for the T3 co-pilot sessions, and is that wanted? | Generic script, 10 min/month, optional | v1 | C7a; 07 P-04 |
| OPD-31 | Spot-check 20 rows of the numerals table and 20 pointed words; Dicta Nakdan registration if the API requires it | Rows marked `verified:false`; Phonikud only | ~18.11 | B4, B5; 12 §11 #8; 13 Q-14 |
| OPD-32 | Life-skills topics in K1's class (for "what they learned this month") | Sequence inferred from grades 2 and 5 (07 R10) | Monthly | 07 P-08 |
| OPD-33 | Cloud-environment network allowlist (E13 hosts) | Not allowed → CI deploys; verification via secondary sources, logged | Optional | E13; 04 §17 #6; 11 §15.1 #11 |
| OPD-34 | 2–3 family routines from 05 §16.2 to adopt | FR-1, FR-2, FR-5 | v1 | 05 §19 #6 |

Specs keep their local question lists for context but must cite the `OPD-nn` ID; 13's `docs/parent/questions.md` template is generated from this table.

---

## Appendix A — Proposal adjudication log (Fable review F1, 2026-11-02)

Decision values: **accept** (applied as proposed), **accept-modified** (applied with a change, stated), **reject** (not applied, reason), **n/a-forward** (not a 00 matter; forwarded in `docs/review/fable-issues-F1.md`).

| Source | Proposal (short) | Decision | Applied to | Rationale |
|---|---|---|---|---|
| 01 §20 #1 | Durable Objects for multi-device co-op (CO-3) | accept | E5 | 11 already designs `CoopRoom`; included in Workers Paid; fallback stated |
| 01 §20 #2 | Explicit quiet-days list; Yom Yerushalayim neutral | accept | B7a | Calendar config needs concrete dates; merged with 02 §16 #6 |
| 01 §20 #3 | B8: no fireworks, ≤40% darkness, K3 dusk only | accept | B8, G5 | Fireworks = explosions; fear-of-dark onset evidence (01 R8) |
| 01 §20 #4 | Sparks are a non-persistent in-episode resource | accept | D1 | Keeps "mastery, not time"; 02 already lists sparks under `neverEarns` |
| 01 §20 #5 | Reveal "upgrade steps" only for earned effort marks | accept | D2 | Preserves the ritual without chance |
| 01 §20 #6 | In-world glyphs as pre-baked textures | accept | E1b | Compatible with 11 §18 #4 (no live text in GL/Skia); nikud validated at bake time |
| 01 §20 #7 | Hero = silent vessel; AI helper has distinct shape language | accept | G4, C7 | Same as 09 PC-4; prevents hero/AI blur for K2/K3 |
| 01 §20 #8 | K1 level 6–9 min; G3 sets K2 only | accept | C3, G3 | Consistent with episode 15–20 min |
| 02 §16 #1 | Catalogue fixed; weekly Sunday showcase; nothing leaves | accept | D3 | Stricter reading of D3; removes daily-rotation hook (Radesky) |
| 02 §16 #2 | Real rewards never priced in coins | accept | D6 | v1 table 8 example contradicted D6 |
| 02 §16 #3 | Gem milestones list; levels do not give gems | accept | D1 | Amounts stay in 02 (D7) |
| 02 §16 #4 | K3 hidden parent-only XP, no visible levels | accept | D5 | Reconciles D4 and D5 |
| 02 §16 #5 | Rescued friends/pieces non-spendable; never gate curriculum | accept | G2 | Curiosity must not become an obligation |
| 02 §16 #6 | "Quiet day" definition; add Tisha B'Av | accept | B7a | Merged with 01 §20 #2 |
| 02 §17 (01↔02 table) | 02 owns numbers; campfire no reset; pet evolution by mastery | accept | D4, D7 | Brief issue #1; C5 "no loss" |
| 03 §17 #1 | C11 mastery definition + review ladder | accept | C11 | gap_audit D9 unified five definitions; 08 §5.1 remains the implementation |
| 03 §17 #2 | C12 no clocks; RT policy | accept | C12 | Merged with 08 §14 #4 (grade rule, not age) |
| 03 §17 #3 | B4: digits in display, words in speech | accept | B4 | Needed by the LLM word-problem pipeline |
| 03 §17.2 | Requests to 08 (extra gates, step scoring, 5 strands, 2 flags, ID map) | n/a-forward | — | 08 owner; C11 explicitly allows subject entry gates |
| 04 §18 PC-04-1 | Hebrew `HREC` ~300-clip human bank | accept | E9, OPD-13 | TTS cannot produce isolated consonants; brief #13 |
| 04 §18 PC-04-2 | B5: ktiv male canonical, adaptive per-word fade, K2 from lexicon IDs | accept | B5 | ⟂VERIFY Nakdan on ktiv male kept |
| 04 §18 PC-04-3 | PD status of Bergstein/Tchernichovsky (notes file) | n/a-forward | — | Not a 00 matter; logged for the notes owner |
| 05 §20 #1 | Extend English recording: letter names, nonsense words, song vocals | accept | E9, OPD-15 | NWF needs real nonsense words; AI singing blocked by E8a |
| 05 §20 #2 | B2: K3 songs only, Hebrew TTS never says English, spatial convention | accept-modified | B2 | Spatial convention adopted as default; 09 owns the visual detail |
| 05 §20 #3 | K2 oral only; English letters incidental | accept | B2a | gap_audit D18 was missing from 00 |
| 05 §20 #4 | No production score for any child; ASR only for reaction; no cloud pronunciation | accept | E9 | Child WER 15–35%; C5 |
| 05 §20.2 | Requests to 01 (pairs mode, MX-E05–E07, Pip, pre-island phase), 08 (25%/15% share, ID map), 09 (type tokens) | n/a-forward | — | Owners 01/08/09; recommendation in issues file |
| 06 §20 #1 | D1 wording: real-money learning allowed; savings bonus declared exception | accept | D1 | Removes a silent contradiction with 02 §7.5 |
| 06 §20 #2 | Physical-safety tags (proposed as C11) | accept-modified | **C13** | C11 was taken by 03's mastery rule; same content, new ID |
| 06 §20 #3 | B9 RTL conventions | accept-modified | B9 | Time-axis rule made global (incl. parent charts, 10 §18.2) with ⟂VERIFY |
| 06 §20 #4 | A6 bands T/G/S as shared vocabulary | accept | A6 | Brief #4; 07 §0.2 definitions |
| 07 §15 #1 | C7a AI tiers T0–T3; chips-only input | accept | C7a | Matches gap_audit D13 |
| 07 §15 #2 | C8a emotional storage rules | accept | C8a | Merged with 08 §14 #1 and 10 Δ3 (one boolean flag with consent) |
| 07 §15 #3 | D1a no coins for SEL | accept | D1a | Deci 1999; emotion-faking risk |
| 07 §15 #4 | B8a bravery ladders exclude security topics; safe room needs review | accept | B8a, OPD-19 | R14, ERASE-Stress precedent |
| 08 §14 #1 | E6a sensitive derived data never to AI vendors | accept | E6a | Amendment 13; C8/E8 were silent |
| 08 §14 #2 | E11a deterministic event-sourced engine | accept | E11a | Elo is order-sensitive; multi-device sync |
| 08 §14 #3 | C9 exploration share applies to topics/wrappers | accept | C9 | Otherwise 20–30% of items would leave the C2 band |
| 08 §14 #4 | RT off below grade 1; K1 fluency opt-in only | accept | C12 | Grade rule rather than age rule |
| 08 §14 #5 | K1 dial 0.85/0.80/0.75; 0.70 only in Stretch | accept | C2 | Within C2's 75–85% |
| 09 §16 PC-1 | `expo-sensors`; camera pose later in dev build | accept | E3, G6 | G6 safety (device-in-hand check) |
| 09 §16 PC-2 | Optional game-controller support | accept-modified | E3 | Allowed after C4, never required, never a mechanic prerequisite |
| 09 §16 PC-3 | Fonts Noto Sans Hebrew / Rubik / Andika; exclude Open Sans, Heebo, Varela Round | accept | B5a | Measured in 09 research log R1 |
| 09 §16 PC-4 | Hero wording "toy-suit with a living spark"; gears for the machine | accept | G4 | Same as 01 §20 #7; brief #2 |
| 09 §16 PC-5 | Celebration budget ≤15%; feedback ≤0.8 s | accept | C14 | gap_audit #5; 01 Appendix A tokens |
| 09 §16 PC-6 | Touch targets in mm; iPad mini = iPhone | accept | A7, A3 | gap_audit #7; 09 §4 tables |
| 09 §16 PC-7 | Record rejection of random capsule machine | accept | D2, this log | D2 forbids chance |
| 09 §16 PC-8 | Web prototypes: `dir="rtl"`, embedded fonts, Rive sprite fallback | accept | E1a | Parent would otherwise review wrong nikud/direction |
| 10 §18.1 Δ1 | B3 covers strings referring to a child anywhere; parent strings impersonal | accept | B3 | Weekly report has gendered verbs about the child |
| 10 §18.1 Δ2 | Shabbat window ±30 min via hebcal; Friday 12:00–14:00 report | accept | B7, E5 | Parent's explicit request; weekend message load |
| 10 §18.1 Δ3 | One boolean `EMO_CHECKIN` flag may sync with consent T4b | accept | C8a | Minimal nudge on the other parent's device |
| 10 §18.1 Δ4 | No remote push until paid program; inbox + email | accept | E3, H6 | No `aps-environment` on a free team |
| 10 §18.1 Δ5 | Viewer sees approved cards only; voice replies gated | accept | H4 | ClassDojo pattern; safety |
| 10 §18.2 | Requests to 11 (cron, flags[], disableDeviceFallback, focus.span, tables), 08 (EMO_CHECKIN), 02↔08 names, 09 chart direction | accept-modified / n/a-forward | E5 (cron), E7 (`disableDeviceFallback`), C15 (names), B9 (chart direction); rest forwarded | Identity/admin items bound in 00; implementation details to 11/08 |
| 11 §18 #1 | E1 phased build wording | accept | E1 | E1 and E1a contradicted each other |
| 11 §18 #2 | Email one-time code primary; SIWA/App Attest/push in phase 2 | accept | E7, H2, H6, E3 | Free team lacks entitlements; `⟂VERIFY` folded into OPD-25 trigger |
| 11 §18 #3 | `expo-sqlite` + kv-store; no MMKV | accept | E3 | Avoids Nitro surface |
| 11 §18 #4 | No Hebrew text in GL/Skia on kid screens; no KTX2/Draco/meshopt; texture caps | accept | E1b | Consistent with 01 §20 #6 |
| 11 §18 #5 | Pages proxies `/api/*` via service binding | accept | H6 | Same-origin cookies |
| 11 §18 #6 | Packages `shared`, `i18n`, `api-client`, `apps/parent-web` | accept | E11 | Additive; `shared` also hosts A6/C15 mappings |
| 11 §18 #7 | Maestro on Mac simulator; Playwright baselines in pinned Docker | accept | E4 | Reproducible baselines |
| 12 §13 #1 | Remove Nano Banana from E10 | accept | E10, E8a | E10 contradicted E8a; brief #11 |
| 12 §13 #2 | ElevenLabs SFX/Music conditional on approval | accept | E10 | E8a |
| 12 §13 #3 | Dual-route Opus+Sonnet for handwriting | accept-modified | E8 | Enabled only if the 30-page family eval shows gain; default single route |
| 12 §13 #4 | Batch API not ZDR → labels only | accept | E8, E6a | Brief #12 |
| 12 §13 #5 | P2 audio rendered server-side after approval; no AVSpeechSynthesizer | accept | E9 | Keeps "no live TTS for kids" without silencing P2 |
| 12 §13 #6 | Child-safety system prompt slot until Anthropic provides text | accept | E8 | Factual state as of 2026-11-02 |
| 13 §19 #1 | Identity without SIWA; Cloudflare Access on web dashboard | accept-modified | E7, H6 | Wording unified to 11's "email one-time code" (= magic link + 6-digit code); Cloudflare Access optional, not required (avoid double login) |
| 13 §19 #2 | E1 aligned to E1a | accept | E1 | Same as 11 §18 #1 |
| 13 §19 #3 | Notifications by email until paid | accept | E3, H6 | Same as 10 Δ4 |
| 13 §19 #4 | VS fully local; staging at C3 without child data; prod Feb 2027 | accept | E5 | Reduces account/privacy dependencies during the quality phase |
| 13 §19 #5 | Nikud-fade calendar (mid-Jan partial, June unpointed) | accept-modified | B5 | Kept as an envelope; 04's per-word adaptive fade governs |
| 13 §19 #6 | Orchestrator session; ≤3 active sessions; boot order | accept | I2 | I1's anti-ping-pong intent |
| 13 §19 #7 | Extra taste gate: rigged 3D hero model | accept | E10 | Conditional on ADR-0005 choosing 3D |
| Brief #3 | Skill-level names: 08 canonical; mapping in `packages/shared` | accept | C15 | 02's states are economy triggers |
| Brief #14 | Playbooks binding | accept | E12 | Already cited by 11; now binding for 13 too |
| Brief #15 | Network allowlist hosts | accept | E13, OPD-33 | Union of 11 §15.1 #11 and 04 §17 #6 |

---

## Fable review changelog (F1, 2026-11-02)

- Added a **Decision index** (every ID, one line) and a **Cross-spec ownership map** (§J) so the developer agent can resolve conflicts without asking.
- Added **Open parent decisions** (OPD-01…34): deduplicated across 00–13, each with a default and a due checkpoint; specs are asked to cite OPD IDs (issues file).
- Added **Appendix A** adjudicating all 70+ proposals from 01–13 (accept / accept-modified / reject / forward) with rationale; applied every accepted one in place.
- **Fixed internal contradictions:** E1 vs E1a (phased build, Mac now / EAS later); E3 MMKV → `expo-sqlite` + kv-store; E10 Nano Banana removed and ElevenLabs SFX/Music made conditional (E8a); Sign in with Apple / push / App Attest moved to phase 2 (E7, H2, H6, E3); Google Chirp excluded from the bake-off (E9).
- **New rows:** A6 (bands T/G/S), A7 (touch targets in mm), B2a (K2 oral English), B5a (fonts), B7a (quiet days), B8a (bravery ladders), B9 (RTL conventions), C7a (AI tiers), C8a (emotional data), C11 (mastery + review ladder), C12 (clocks/RT), C13 (physical safety), C14 (celebration budget), C15 (level names), D1a (no coins for SEL), D7 (02 owns numbers), E6a (sensitive data never to AI vendors), E11a (deterministic engine), E12 (binding playbooks), E13 (network allowlist), I3 (checkpoint list), J1 (ownership map).
- **Edited rows:** A1, A3, A4, B2, B3, B4, B5, B7, B8, C2, C3, C4, C5, C7, C8, C9, D1–D6, E1, E1a, E1b, E3, E4, E5, E6, E7, E8, E8a, E9, E10, E11, H1, H2, H3, H4, H5, H6, I1, I2, G2, G3, G4, G5, G6, G7, F1, F2 — see Appendix A for the source of each change.
- Parent-mandated decisions preserved verbatim in intent: coins + gems + chests + cosmetic shop (D1–D5), Hebrew-first RTL (B1), family-only local install (A1, E1a), Astro-Bot-like joyful explorable world with rescues (G1–G3), learning through movement (G6), identity/admin model (H1–H6), checkpoints (I1–I3).
- `⟂VERIFY` markers kept/added: Expo Go SDK ceiling (E1a), Dicta Nakdan ktiv-male handling (B5), quiet-day dates vs hebcal (B7a), time-axis direction (B9).
- Cross-file issues written to `docs/review/fable-issues-F1.md`.
