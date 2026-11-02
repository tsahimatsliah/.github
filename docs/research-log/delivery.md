# Research log — Delivery, roadmap & QA (feeds `docs/spec/13-delivery-roadmap-qa.md`)

Date: 2026-11-02. Author: delivery-spec agent. Format follows F3: question, findings with sources, confidence, and what changed in the spec.

**Network note:** these hosts did not resolve from the cloud sandbox (not on the session's allow-list): `www.karamel.co.il`, `meyda.education.gov.il`, `www.hebcal.com`. I worked from search-result extracts. Each date marked "verify" should be checked against the official Ministry of Education calendar after `meyda.education.gov.il` and `hebcal.com` are added to the allow-list. That is a story in Phase 0 (P0-E8-S3).

---

## Q1. Israeli school calendar, 5787 (2026–27)

| Event | Date | Source | Confidence |
|---|---|---|---|
| School year opens | Tue 1.9.2026 | search extracts (Srugim / Walla / Israel Hayom) | High |
| Hanukkah break | Sun 6.12.2026 – Sat 12.12.2026, back Sun 13.12.2026 | [mako](https://www.mako.co.il/home-family-recreation/Article-33b7e14799af651006.htm), [Israel Hayom](https://www.israelhayom.co.il/vacation-calendar) | Medium-high |
| Purim break | Tue–Wed 23–24.3.2027, back Thu 25.3 | [karamel](https://www.karamel.co.il/לוח_חופשות_משרד_החינוך.asp) (search extract) | Medium |
| Pesach break | Tue 13.4.2027 – Wed 28.4.2027, back Thu 29.4 | karamel + mako (extracts) | Medium-high |
| Erev Pesach | 21.4.2027 | [Technion calendar](https://hr.technion.ac.il/wp-content/uploads/2026/06/לוח-חופשות-תשפז-2027-2026.pdf) (academic, used only for the Hebrew-calendar date) | High (calendar date) |
| Yom HaAtzmaut | 12.5.2027 (Yom HaZikaron 11.5) | Technion | High (calendar date) |
| Lag BaOmer | Tue 25.5.2027, a regular school day | karamel extract | Medium |
| Erev Shavuot | 10.6.2027 | Technion | High |
| End of year, elementary and kindergarten | 30.6.2027 (middle and high school 20.6) | karamel extract; source text is "slightly garbled" | Medium; **verify** |
| Tu BiShvat | 23.1.2027 (Shabbat), from background calendar knowledge | not verified | Medium; **verify** |
| Yom HaShoah | ~4.5.2027, from background knowledge | not verified | Medium; **verify** |

**Change to the spec:** a calendar table in §2. The Hanukkah break becomes the first "playtest window" of the Vertical Slice. The Pesach break is the end-of-MVP evaluation window. The summer is reserved for preparing K2 for grade 1 (5788).

## Q2. When does K1 move to unpointed reading?

- The Ministry's grade-3 fluency tool (Mazkirut Pedagogit) has three assessment windows: start of the year (Sep–Oct), **mid-year (Jan–Feb) for formative intervention**, and end of year. In the Ministry's table, start-of-grade-3 texts are pointed and **end-of-grade-3 texts are unpointed**. The main transition happens "during grade 3". Sources: search extracts of [fluency-assessment-2021.pdf](https://meyda.education.gov.il/files/Mazkirut_Pedagogit/HebrewPrimary/assessment/fluency-assessment-2021.pdf) and [fluency-assessment-end-of-classB.pdf](https://meyda.education.gov.il/files/Mazkirut_Pedagogit/HebrewPrimary/assessment/fluency-assessment-end-of-classB.pdf). The full text could not be read (host blocked).
- Confidence: medium. **Interpretation:** "~Feb 2027" (B5) is the *starting point and checkpoint* of the transition, not the date by which it must be finished. Unpointed reading is expected to be established by end of year (June 2027).
- **Change to the spec:** the nikud-fade track has to be in K1's hands by early January 2027, so that the in-app partial-nikud stage coincides with the mid-year assessment. The first Vertical Slice already includes a text layer with an adjustable nikud level. A "Proposed change to 00" note clarifies B5.

## Q3. Vertical slice in game development

- A vertical slice is a short, representative piece of the game, polished to "ship quality", that cuts through every layer (art, code, design, audio). It is usually one level. It sets the quality bar for the rest of production. Caveat: its development pace is usually *not* a reliable predictor of production pace. Sources: [Wayline — Ultimate guide](https://www.wayline.io/blog/ultimate-guide-vertical-slices-indie-devs), [MUNI PV266 course doc](https://is.muni.cz/el/fi/jaro2018/PV266/um/PV266_-_Horizontal_and_Vertical_Slice_Descriptions.pdf?kod=PV255), [Game Developer — "why we should stop saying vertical slices"](https://www.gamedeveloper.com/design/why-we-should-stop-saying-vertical-slices-), [bugnet](https://bugnet.io/blog/what-is-a-vertical-slice). Confidence: medium (blogs and courses, no formal standard).
- **Change to the spec:** the VS is defined as one world region, one mechanic per child and one full session loop (entry, play, reward, soft ending), at final quality. Exit criteria are about *quality and fun*, not breadth. Estimates for later phases are adjusted upward by ×1.5 relative to VS pace.

## Q4. Playtesting with young children: Fun Toolkit, observation, assent

- Fun Toolkit (Read & MacFarlane): Smileyometer (5 faces), Again-Again table (yes/maybe/no), Fun Sorter (ranking). It is intended for ages ~5–10. Children skew toward the happiest face, and the order of activities affects ratings. Some researchers think a VAS fits only from age ~7. Sources: [Aalto Fun Toolkit](https://experience.aalto.fi/fun-toolkit/), [Read & Horton 2025](https://clok.uclan.ac.uk/54967/1/54967%20Read%20%26%20Horton%20VOR.pdf), [Read 2008](https://dx.doi.org/10.1007/s10111-007-0069-9). Confidence: medium-high.
- Assent is an ongoing, multimodal process, not a one-off form (e.g., an illustrated "assent mat"). A child's refusal is honoured even when a parent has consented. Assent from toddlers is barely addressed in the literature. A toddler can agree to play but cannot meaningfully assent to data use. Sources: [White Rose eprint 240595](https://eprints.whiterose.ac.uk/id/eprint/240595/), [Digital Child ethics working paper](https://digitalchild.org.au/research/publications/working-paper/ethical-considerations-in-research-on-digital-childhoods), [Stellenbosch assent template](https://files.su.ac.za/public/ethics/documents/2025-10/informed-assent-template-children.pdf). Confidence: medium.
- No source was found on the dual role of a parent researching their own children. The protocol adds safeguards based on judgement (separating play from testing, observation over self-report, a child-initiated "stop" signal).
- **Change to the spec:** §8 adapts the protocol per age. K3 gets observation only. K2 gets a 3-face Smileyometer plus Again-Again in pictures. K1 gets the full Fun Toolkit plus a "This or That" comparison. Every session has a picture-based "I want to stop" card.

## Q5. Outcome evaluation for N=3: single-case designs

- WWC standards for single-case designs: at least 3 demonstrations of an effect at 3 different points in time. For a multiple baseline: ≥3 tiers (staggered introduction) and ≥5 data points per phase for "meets without reservations" (≥3 "with reservations"). Lanovaz & Turgeon (2020) argue the 3-demonstration rule is not empirically grounded. In their simulation, 3 tiers with ≥2 clear changes keep Type I error acceptable. Sources: [WWC SCD brief](https://ies.ed.gov/ncee/WWC/Docs/ReferenceResources/wwc_scd_understanding_IR_011017.pdf), [Perspectives on Behavior Science 2020](https://link.springer.com/article/10.1007/s40614-020-00263-x), [CEC-DR position statement](https://cecdr.org/sites/default/files/2021-01/_DR_Position_Statement_5_data_points_WWC_SCD_final.pdf). Confidence: high on the principles, medium on the exact numbers (the current handbook was not read).
- Effect-size measures that supplement visual analysis: Tau-U (Parker et al. 2010) and NAP. Source: [PMC8070073](https://pmc.ncbi.nlm.nih.gov/articles/PMC8070073) and studies that pair visual analysis with Tau-U. Confidence: medium.
- **Change to the spec:** §9 uses a multiple-baseline design *across skills within each child* (3 tiers per child), with brief paper probes (1–2 minutes) twice a week. School instruction is recorded as a confounder, and an untaught control skill is added. The goal is "good enough for a family decision", not a scientific claim.

## Q6. Local install without the App Store (E1a)

- **Expo Go:** Reanimated and Skia are marked "Included in Expo Go" in the SDK 57 docs. Expo Go runs only the native modules built into it, so any other library needs a development build. Sources: [Expo Reanimated v57](https://docs.expo.dev/versions/v57.0.0/sdk/reanimated.md), [Expo Skia v57](https://docs.expo.dev/versions/v57.0.0/sdk/skia.md), [third-party overview v58](https://docs.expo.dev/versions/v58.0.0/sdk/third-party-overview.md). Confidence: medium-high. **Verify on SDK 58** in P0.
- **Rive does not work in Expo Go.** It contains custom native code and fails with a `requireNativeComponent` error, so a development build is required. Source: [Rive docs: Adding Rive to Expo](https://rive.app/docs/runtimes/react-native/adding-rive-to-expo). Confidence: high.
- **Tunnel:** `npx expo start --tunnel`. From SDK 58 a new tunnel is the default (the `EXPO_UNSTABLE_TUNNEL_V2` variable is ignored). The docs list intermittent connection failures and say the tunnel URL is public. Sources: [Expo CLI](https://docs.expo.dev/more/expo-cli/), [Expo Go simulators page](https://docs.expo.dev/preview/simulators/works-with/expo-go.md). Confidence: medium. Each Expo Go version supports a single SDK (by background knowledge and the "New Expo Go supports only SDK 58" statement in `app_stack_ai_built_workflow.md`). That is a version-drift risk when Expo Go updates in the App Store.
- **Free Apple ID (Personal Team):** provisioning profiles expire **7 days** after issue, and then the app must be rebuilt and reinstalled. Limits: up to 3 apps per device and up to 10 App IDs, which also expire after 7 days. One user reports that an expired app cannot be launched, only deleted. A paid profile lasts a year. Sources: [Apple compare memberships (JP)](https://developer.apple.com/support/compare-memberships/jp/), [Apple forums 71283](https://developer.apple.com/forums/thread/71283), [Apple forums 724896](https://developer.apple.com/forums/thread/724896). Confidence: high on the 7 days and 3 apps; **medium on a cap of "3 devices"** (appears in an extract only; verify).
- **Personal Team does not support every capability.** Apple DTS refers to the "Supported capabilities" table. Confirmed unsupported: Core NFC, Group Activities, Family Controls. **Not verified:** Sign in with Apple, Push Notifications, App Attest, iCloud. Sources: [Apple forums 93090](https://developer.apple.com/forums/thread/93090), [forums 727575](https://developer.apple.com/forums/thread/727575), [forums 764682](https://developer.apple.com/forums/thread/764682). Confidence: medium. **Action:** the spec assumes these capabilities are *unavailable* until the paid program, and proposes a change to E7/E3 in 00.
- **Change to the spec:** a 3-stage install ladder (§3), a weekly re-sign routine, a capability matrix per stage, an explicit trigger for upgrading to the paid program, and risks R-INST-1..7.

## Q7. ADR templates

- Michael Nygard's format (Title, Status, Context, Decision, Consequences) is the de facto standard. MADR adds "Considered options" and "Decision drivers". Sources: background knowledge (Nygard 2011 "Documenting Architecture Decisions"; adr.github.io / MADR). Not re-verified this session; low risk.
- **Change to the spec:** §12.3 gives a short Nygard+MADR-based template with added fields: "evidence level (F1 1–5)", "reversibility", "did the parent need to be asked? why not", and "review trigger".

## Q8. Working with an AI agent: CLAUDE.md and task breakdown

- Community guides (not official) agree: keep CLAUDE.md short (~<200 lines), specific and true for the whole project. Include build/test commands, structure, key conventions and 5–10 non-negotiable rules. Layer it (project / local / subfolders) and iterate after `/init`. Sources: [DEV — CLAUDE.md best practices 2026](https://dev.to/nishilbhave/claudemd-best-practices-the-complete-2026-guide-435j), [Danny Willems — memory system](https://dannywillems.github.io/2026/01/16/claude-code-memory-system.html), [eesel — 7 best practices](https://eesel.ai/blog/claude-code-best-practices). Confidence: medium (unofficial sources).
- **Change to the spec:** §14 proposes a root CLAUDE.md under 200 lines, with long rules pushed to `docs/` and to subfolder CLAUDE.md files (`apps/mobile/`, `packages/content/`). Tasks are broken into stories of at most one agent-day each, with written acceptance criteria and a screenshot as proof.

## Q9. Risk register for kids' apps

- Built on the register in `research_notes/…/risks_failure_modes.md` §10 (R1–R21), the new concerns in `gap_audit.md` §4, the vendor terms in `voice_tech_he_en.md` §2 and the new install findings (Q6). No new external source was needed. The register in the spec (§11) has 40 items with owner and trigger.

## Open items (for later research, not blocking)
1. The official Ministry 5787 calendar (meyda host), and whether kindergartens end on 30.6 or have a "summer kindergarten" in July.
2. The current Apple capabilities table for a Personal Team (Sign in with Apple, Push, App Attest), to be checked on device in P0-E2.
3. Expo Go SDK 58: confirm that Skia, Reanimated 4, expo-audio and expo-sqlite are bundled.
4. Whether reinstalling a free-signed app over an expired version keeps its data container (to be checked on device in P0-E2-S5).
