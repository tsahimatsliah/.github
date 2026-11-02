# Research log: Economy & progression

> Per 00-F3: research done while writing a spec is logged here. Feeds `docs/spec/02-economy-progression.md`.
> Session date: 2026-11-02. Author: spec agent (game-economy designer role).

## Access conditions this session (read first)

- **WebFetch blocked** (DNS not resolvable in this session): `animalcrossing.fandom.com`, `timesaver.gg`. Every other
  finding below comes from **search-result snippets**, not full pages.
- **Search budget:** the shared WebSearch budget ran out partway through the session. Two planned queries were
  **not run**: (a) Astro Bot rescue counts / Gatcha Lab / puzzle pieces, (b) the goal-gradient and endowed-progress
  primary papers. Spec claims on those two topics are tagged **[BK]**.
- **Action for parent/devops (one line):** to verify the [BK] items, allow `animalcrossing.fandom.com`,
  `timesaver.gg`, `bulbapedia.bulbagarden.net`, `nookipedia.com` and `jamanetwork.com` in the session's network
  settings, then re-run the "Open verification tasks" below.

Evidence tags: **[V]** verified in a source this session (search snippet), **[N]** from an earlier research note in
this repo, **[BK]** background knowledge, not verified this session, **[D]** design decision derived from the above.

---

## 1. XP curve formulas used in games

**Findings**
- [V] Old School RuneScape: the XP needed for each level is about 10% more than for the previous one. Level 2
  needs 83 XP and level 99 needs 13,034,431 XP in total. From about level 30 the exponential term dominates, so the
  XP required doubles about every 7 levels — [OSRS wiki: Exp table](https://oldschool.runescape.wiki/w/Exp_table);
  [OSRS wiki: XP](https://oldschool.runescape.wiki/w/XP).
- [V] RuneScape Dragonwilds uses a piecewise curve: exponential up to level 50, then linear —
  [Dragonwilds wiki](https://dragonwilds.runescape.wiki/w/Experience).
- [V] The common families are linear, polynomial (`base × L^p`; p=2 is the classic quadratic; 1.5 is gentle,
  2.5+ is "brutal") and exponential (impractical at high level caps) —
  [XP Curve Calculator (secondary)](https://toolhub-c14.pages.dev/xp-curve-calculator/);
  [Medium: Graphs for player progression](https://medium.com/@ipooprainbows/graphs-for-player-progression-part-ii-3807b25beee5).
- [BK] Pokémon "Medium Fast" growth group = n³ total XP; Fast = 0.8·n³; Slow = 1.25·n³. Not verified this session.

**Implications [D]**
- Exponential curves are a bad fit for children: every new level takes noticeably longer than the last, and the
  game slows down just when novelty decay sets in (risks note R1). Use a **gentle polynomial on XP-to-next**
  (`xpToNext(L) = round10(250 + 80·(L−1)^1.2)`). This gives a level in the first episode, a level about every 2–3
  days in the first month, and a level about every 2 weeks by level 30. Simulated pacing is in spec §3.

## 2. Battle/season passes: tier counts and pacing

**Findings**
- [V] Brawl Stars Brawl Pass after the January 2026 rework: **60 tiers** (up from 50), about **53,250 XP** to finish
  (≈ 890 XP per tier on average), season ≈ **one month** (down from two). Season examples: Starr Patrol 7 May–4 Jun
  2026; Sands of Time 4 Mar–2 Apr 2026. The free track includes Coins, Power Points, Credits, Starr Drops, Chaos
  Drops and 50 Gems. Paid tracks cost $8.99 / $12.99 —
  [timesaver.gg (snippet)](https://timesaver.gg/blog/brawl-stars-brawl-pass);
  [sportsdunia Starr Patrol](https://www.sportsdunia.com/gaming/brawl-stars-starr-patrol-brawl-pass-may-2026);
  [sportsdunia Sands of Time](https://www.sportsdunia.com/gaming/brawl-stars-sands-of-time-brawl-pass-march-2026);
  [mobilematters: rework](https://mobilematters.gg/brawl-stars/brawl-stars-brawl-pass-rework);
  [Supercell: incoming changes to the Brawl Pass](https://supercell.com/en/games/brawlstars/blog/news/incoming-changes-to-the-brawl-pass/).
- [V] Fortnite: weekly quests worth about 25K XP each (≈1.5 Battle Pass levels per week from quests in one
  Chapter 6 guide). Sources conflict across seasons on XP per level and on quest counts —
  [bo3.gg](https://bo3.gg/games/articles/how-to-get-fortnite-xp-fast-in-chapter-6);
  [GameSpot](https://www.gamespot.com/articles/fortnite-xp-gains-increased-for-the-fourth-time-in-two-weeks/1100-6497040/);
  [Sportskeeda C3S4](https://sportskeeda.com/fortnite/fortnite-chapter-3-season-4-how-much-xp-need-reach-level-100).
- [N] The FOMO levers in passes are the monthly expiry, the premium track and countdowns
  (hit_games_engagement.md §1, §7).

**Implications [D]**
- Keep the legible part of a pass (a linear track with a reward at every tier). Drop the expiry, the paid track and
  the timer. Use **30 tiers for K1 and 20 for K2**, at a **constant** XP cost per tier (400 / 600), so a typical
  week moves about 5–6 tiers. That gives the steady "a reward every session or two" rhythm without escalation.
- Seasons run **8–9 weeks** rather than Brawl Stars' 4. The track is designed to finish in about 5 weeks of typical
  play, so a child with a sick week still finishes, and it never expires anyway.

## 3. Chest/drop reveals: Clash Royale Lucky Chest, Brawl Stars Starr Drop

**Findings**
- [V] Clash Royale Lucky Chests (Oct 2025): Common = cards + Gold, **3 spins**; Magic = cards, Gold, Magic Items,
  cosmetics, **4 spins**; Seasonal chests rotate. Each tap is a chance to upgrade the star rarity (max 5-Star).
  The December 2025 update raised the 5-Star Magic chance from 0.3% to 1% —
  [Supercell Oct 2025 notes](https://supercell.com/en/games/clashroyale/blog/release-notes/october-update-2025);
  [Supercell support: Feelin' Lucky?](https://support.clashroyale.com/hc/en-us/articles/49484957072539);
  [Supercell Dec 2025 notes](https://supercell.com/en/games/clashroyale/blog/release-notes/december-update-2025);
  [mobilematters overview](https://mobilematters.gg/guides/clash-royale/lucky-chests-overview).
- [V] RoyaleAPI's community estimate of final Common-chest rarity: about 31% 1★ / 46% 2★ / 19% 3★ / 4% 4★ —
  [RoyaleAPI Q3 2025](https://royaleapi.com/blog/clash-royale-2025-q3-update?lang=nl).
- [V] **Brawl Starr Drops: the rarity is rolled when the drop is awarded. The tap-to-upgrade animation only reveals
  a result that is already decided.** One source says Legendary drops get "up to four spins". Base odds in one
  guide: Rare 50%, Super Rare 28%, Epic 15%, Legendary about 2% —
  [timesaver.gg July 2026 (snippet)](https://timesaver.gg/blog/brawl-stars-starr-drops-july-2026);
  [AFK Gaming](https://afkgaming.com/mobileesports/guide/brawl-stars-starr-drops-rewards-rarities-drop-chance-how-to-get-more);
  [Sportskeeda](https://www.sportskeeda.com/esports/starr-drop-rates-brawl-stars-explained).
- [N] No primary Supercell source on reveal timing in ms exists. The UI note gives the anticipation → burst →
  reveal → settle → collect choreography (ui_ux_polish_craft.md, "Reward reveal choreography"). The hit-games note
  says "the build-up is the reward".

**Implications [D]**
- The Starr Drop finding matters. The commercial "upgrade" animation is **theatre presented as chance**: the
  outcome is fixed but the child is made to feel it is being won by tapping. That is a near-miss pattern.
  Our chests keep the tapping ritual (taps add wobble and a rising tone), but **the tier is shown from the first
  frame and never changes**, and the contents are previewed. There is no colour-upgrade animation, ever.
- Tap counts follow Clash Royale's 3–4 spins: wood 1, silver 2, gold 3, legendary 3 + a hold. The ms budgets are
  derived from the celebration tiers (2 / 3 / 4.5 / 6 s).

## 4. Achievement-style currencies: Animal Crossing Nook Miles

**Findings**
- [V] Nook Miles achievements are tiered stamps. Example: "Angling for Perfection" pays **300 / 500 / 1,000 /
  2,000 / 3,000 Miles** for catching 10 / 100 / 500 / 2,000 / 5,000 fish, and each stamp grants a passport title.
  "Island Ichthyologist" tier 1 (10 unique fish) pays 300 —
  [Polygon table mirror](https://gamecytus.netlify.app/host-https-www.polygon.com/animal-crossing-new-horizons-switch-acnh-guide/2020/3/20/21186746/nook-mileage-rewards-titles-list-tasks-miles-chart-table);
  [Nookipedia: Titles](https://nookipedia.com/wiki/Titles);
  [GamerGuides: Nook Miles achievements](https://www.gamerguides.com/animal-crossing-new-horizons/guide/nook-miles/nook-miles-achievements).
- [V] Nook Miles+ (unlocked after the first 5,000-mile Resident Services fee) gives a set of daily tasks; some pay
  2× or 5×. There is reportedly no daily completion limit —
  [GamerGuides](https://gamerguides.com/animal-crossing-new-horizons/guide/nook-miles);
  [iMore](https://www.imore.com/animal-crossing-new-horizons-how-earn-nook-miles-fast).
- [N] Animal Crossing's "daily stimulus" (fossils, mail, shop stock) gives a reason to return through novelty, with
  no penalty for missing a day (hit_games_engagement.md §3).

**Implications [D]**
- Copy the **tiered, one-time achievement** structure: big payouts for cumulative real milestones (titles, badges,
  rescue milestones every 100). Do **not** copy Nook Miles+ rotating daily tasks: Radesky lists "accumulating daily
  rewards" as a lure (gap_audit.md G-Radesky), and Animal Crossing's daily shop rotation is a daily-return lever.

## 5. Duolingo XP and gems numbers

**Findings**
- [V] About **10 XP per lesson** is the consistent figure; Stories pay 14–30 XP. Gems: 3 daily quests with
  guaranteed gem rewards (sources conflict: 5–15 vs 20–100 gems). One source says achievements pay 25 gems. Finishing
  all 3 quests gives a triple-XP boost. Gems also come from watching ads (7 per ad). All of these come from
  low-authority secondary blogs —
  [Duolingo Wiki: Gem](https://duolingo.fandom.com/wiki/Gem);
  [happilyevertravels](https://happilyevertravels.com/duolingo-gems-lingots/);
  [duolingoguides](https://duolingoguides.com/?p=4901).
- [N] Duolingo: the path or home screen is "probably the most successful mechanic". Leagues added about 17% to time
  spent, mostly among already-heavy users. Energy charges for practice itself and caused a backlash
  (kids_edu_apps_teardown.md §1).

**Implications [D]**
- Rewarding **ads** and **XP boosts** ("triple XP for 15 minutes") are both time-pressure or attention levers;
  neither is used. The useful lesson is the *granularity*: a per-lesson XP number that is small, steady and
  legible. Our XP is per learning event with a 50-XP episode bonus.

## 6. Sources/sinks balancing methods

**Findings**
- [V] Machinations models an economy as resource flows: Source, Drain, Pool and Converter nodes. Balance is net
  weekly inflow minus outflow per pool, and excess accumulates weekly as inflation. The tool supports Monte-Carlo
  simulation of player journeys before (soft) launch —
  [Machinations: what is game economy design](https://machinations.io/articles/what-is-game-economy-design);
  [wikidocs (KR) chapter 8.2](https://wikidocs.net/367359);
  [AIIDE: Simulating mechanics to study emergence](https://ojs.aaai.org/index.php/AIIDE/article/view/12477);
  [Wooga case study](https://machinations.io/articles/how-wooga-uses-machinations-to-balance-junes-journeys-game-economy).
- [N] Only **destructive** sinks counter inflation, and event injections without a matching event sink cause
  lasting spikes (coop_economy_ios.md §2).

**Implications [D]**
- With 3 players, a full Machinations model is overkill. The spec uses an **expected-value weekly flow table per
  pool**, computed by a small script from the same `economy.config.json` defaults (spec §11). There are three
  automatic alarms: hoarding, overjustification and shop-time share. Every holiday faucet is paired with holiday
  sink items (`holidayEvents.matchedSinkRequired`).

## 7. Overjustification mitigation and token-economy fading

**Findings**
- [N] Deci, Koestner & Ryan 1999 (128 experiments): expected tangible rewards undermine free-choice intrinsic
  motivation, more so for children. Unexpected rewards and informational feedback do not. Cameron & Pierce argue
  that the effect is conditional (tangible, expected, loosely tied to performance, high-interest task)
  (child_psychology_motivation.md §1; gap_audit.md C1).
- [V] Secondary summaries agree that **performance-linked** rewards show smaller or no undermining. Rewards
  experienced as **informational** support competence; rewards experienced as **controlling** undermine it —
  [behavioraleconomics.com](https://www.behavioraleconomics.com/resources/mini-encyclopedia-of-be/overjustification-effect);
  [Cameron et al. 2001, RER](https://journals.sagepub.com/doi/10.3102/00346543071001001);
  [Deci et al. 2001 (Ewha record)](https://pure.ewha.ac.kr/en/publications/extrinsic-rewards-and-intrinsic-motivation-in-education-reconside/).
- [V] Token-economy practice (Vanderbilt TRIAD): start with low requirements and **thin gradually** (more tokens
  per backup reinforcer, longer intervals). **Never remove tokens** for misbehaviour. Maintenance after tokens are
  withdrawn works best when thinning is combined with social reinforcement or self-monitoring (Regnier et al.
  2022 review, via secondary summary). Gains can vanish without a planned fade-out —
  [VKC Token Economy Tip Sheet](https://vkc.vumc.org/assets/files/triad/ffp/Challenging_Behaviors/Token_Economy_Tip_Sheet.pdf);
  [VKC: how to design a token economy](https://familiesfirst.vkcsites.org/?p=158);
  [Behaviorist Book Club: maintenance review](https://behavioristbookclub.com/aba-research/aba-fundamentals/a-systematic-review-of-treatment-maintenance-strategies-in-token-econo/).
- [N] Gamification in primary school: g ≈ 0.31, with a larger effect on extrinsic than on intrinsic motivation
  (Kurnaz & Koçtürk 2025, gap_audit.md G5).

**Implications [D]**
- Coins are paid as an **end-of-episode receipt** that names what was learned (informational), never as an
  announced per-task price (D6 / Lepper).
- **Planned thinning.** Reward density falls with age (K3 sensory → K2 chunked → K1 sparse). Within a child it
  falls with mastery (mastered repeats pay 1 coin, then 0). The global `rewardDensityMultiplier` is lowered when the
  free-choice alarm fires.
- Coins never decrease except when the child spends them.
- Social reinforcement runs in parallel: "show Ima/Abba", campfire family moments, parent co-play scripts.

## 8. Kids' financial literacy via jars and game economies

**Findings**
- [V] Spend/save/share **three-jar** activities are standard in preschool financial education: Head Start (Illinois
  WILL), UW-Extension "Parenting the Preschooler", and Practical Money Skills, which suggests a **two-jar**
  save/spend version for under-5s —
  [Illinois WILL](https://will.illinois.edu/education/entry/using-play-and-positive-attention-to-teach-financial-basics-to-4-and-5-year);
  [UW-Extension PTP Save and Spend 2025](https://parenting.extension.wisc.edu/files/2025/06/PTP-Save-and-Spend-2025.pdf);
  [Practical Money Skills lesson](https://columbusmetro-pmscom.siuat.visa.com/content/dam/financial-literacy/practical-money-skills/pdfs/lessons/lev_1/1_2.pdf);
  [Sesame Workshop financial education](https://sesameworkshop.org/topics/financial-education/).
- [V] Children as young as 3 save tokens for later. In a savings board game they saved about 2 of every 3 tokens.
  Asking children to **budget** now vs later increased saving (Canadian research group, via press summary) —
  [PressReader / Cape Breton Post](https://www.pressreader.com/canada/cape-breton-post/20201207/281981790163613).
- [V] A parent-perspective preschool study (101 children, 36 parents) reported gains in money management and in
  enjoyment of donating savings. It is self-report with no control group —
  [JSSER](https://jsser.org/index.php/jsser/article/view/4593).
- [N] No empirical evidence was found that in-game economies teach real money management (v1 report, Table 8).

**Implications [D]**
- K2 gets 2 jars (spend/save) plus a one-tap "give a coin to the campfire". K1 gets 3 jars with child-chosen split
  presets.
- Save interest is concrete and capped: K2 gets +1 coin per full 10 held all week (which teaches tens); K1 gets 5%
  on the week's minimum balance, capped at 20.
- The financial-literacy claim stays modest: "a playful bridge to the real jars at home", not "teaches money".
  A printable 3-jar sheet goes into the phygital pack.

## 9. Manipulative-design taxonomy (Radesky 2022 and successors)

**Findings**
- [V] Radesky et al. 2022, JAMA Netw Open 5(6):e2217641. It is a coding scheme for manipulative design in apps
  used by 160 children aged 3–5. Each child's exposure was scored on **overall, gameplay-prolonging pressure,
  purchase pressure and ad-viewing pressure**. Nearly all children had at least one manipulative design in a
  top-used app, and lower-education households scored higher —
  [PMC8942378](https://pmc.ncbi.nlm.nih.gov/articles/PMC8942378);
  [Bohrium record](https://bohrium.dp.tech/paper/arxiv/d1ec878edcc78c03598107f0b657dd6c0de98822cdc6606af6f5f6b0a147ca8d).
- [N] Examples coded: parasocial pressure (a character crying, "Don't just stand there!"), countdown clocks,
  lures (stickers, trophies, **accumulating daily rewards**), hidden close buttons, navigation constraints
  (gap_audit.md; child_psychology_motivation.md §6).
- [V] Chen, Radesky, Hiniker et al. 2024 used a newer four-strategy scheme: **pressuring, enticing, trapping,
  lulling** — [Springer EM 2024](https://link.springer.com/article/10.1007/s12525-024-00699-y);
  [arXiv 2411.12083](https://arxiv.org/html/2411.12083). A 2025 study covers deceptive design and advertising in
  popular children's apps — [arXiv 2512.17819](https://arxiv.org/pdf/2512.17819).

**Implications [D]**
- The spec's anti-pattern checklist (§13) is organised by Radesky's 3 pressure families plus the 2024 four
  strategies, plus gambling-like, loss-aversion and social-comparison items. Each item has a test.

## 10. Israeli calendar 2026-27 (תשפ"ז) for season boundaries

**Findings**
- [V] School holidays (secondary sites; the official MoE PDFs appeared in results but were not extracted):
  - Hanukkah vacation 6–12 Dec 2026.
  - Purim 23–24 Mar 2027.
  - Pesach vacation 13–28 Apr 2027.
  - Yom HaZikaron 11 May 2027; Yom HaAtzmaut 12 May 2027.
  - Lag BaOmer 25 May 2027 (conflicting reports on whether it is a day off).
  - Shavuot 10–11 Jun 2027.
  Sources: [mako](https://www.mako.co.il/home-family-recreation/Article-33b7e14799af651006.htm);
  [karamel](https://www.karamel.co.il/לוח_חופשות_משרד_החינוך.asp);
  [rng.org.il](https://rng.org.il/uploads/n/1785678884.5499.pdf);
  [MoE (PDF, not extracted)](https://meyda.education.gov.il/files/PreSchool/maonot/jewish2026-2027.pdf).
- [V] Hanukkah 2026 first day is 5 Dec (it begins the evening of 4 Dec). Erev Pesach is 21 Apr 2027. Shavuot is
  11 Jun 2027 (some sources say 10 Jun) — [timeanddate](https://timeanddate.com/holidays/israel/2027);
  [calendarific](https://calendarific.com/holidays/2027/IL).
- [BK] Rosh Hashana 5787 was 12–13 Sep 2026; Tu BiShvat 23 Jan 2027; Yom HaShoah 4 May 2027 (verify).

**Implications [D]**
- The season and event calendar is owned by `01-game-design.md` §13.2: S0 "המחנה מתעורר" from launch on 8 Nov
  2026, then S1–S6 of 5–9 weeks each. Spec 02 scales tier XP to each season's length. Dates live in config, not in
  code. Holiday dates come from a bundled, hand-checked table (Hebcal-generated at build time, then reviewed),
  mirroring 00-B4's lookup-table principle.

## 11. Astro Bot rescue/collection model (search budget exhausted — background only)

- [BK] Astro Bot (Team Asobi, Sept 2024): about **300 bots** to rescue across about 80 levels in several galaxies.
  Each level hides several bots plus **puzzle pieces**. Rescued bots return to the **Crash Site** hub, which fills
  up with them. Some hub areas need a minimum number of rescued bots. Puzzle pieces unlock hub features and the
  **Gatcha Lab**, where coins buy capsule prizes. Not verified this session.
- [BK] Super Mario Odyssey Power Moons (non-spendable progress count gating the next kingdom) and Mario Odyssey
  coins (spendable on cosmetics) are the classic split between **"progress tokens" and "spending money"**.
- [BK] Goal-gradient: Kivetz, Urminsky & Zheng 2006 (JMR) found that coffee-card customers bought more often as they
  neared the free coffee. Endowed progress: Nunes & Drèze 2006 found that a 10-stamp card with 2 stamps
  pre-filled was completed more often than an 8-stamp blank card (≈34% vs 19%). Not re-verified this session.
- [N] 00-G1/G2 (parent input): dense hidden secrets, rescue loop, hub as trophy room, silhouettes, no FOMO.
- [N] `docs/spec/01-game-design.md` (written in parallel) owns the counts: 3/5/7 friends and 0/3/5 pieces per level, a 30–40 piece mosaic per region, per-child neighborhood thresholds, and the S0–S6 season calendar. Spec 02 adopts these and adds the economy layer (spec 02 §17).

**Implications [D]**
- Rescues and puzzle pieces are **non-spendable progress counts**. They are the hub's primary progression and they
  never decrease. Coins and gems stay the spending money.
- Collectibles gate only *bonus* content (hub areas, secret levels, capsule series), **never** the curriculum path.
- Astro Bot's random Gatcha becomes a **deterministic capsule machine** whose next capsule is visible.
- Endowed progress is used **honestly**: the region's guide friend is a real rescue the child performs in the
  intro beat, not a fake pre-fill.

---

## Open verification tasks

1. Read Deci, Koestner & Ryan 1999 Table 2 (children vs college; expected vs unexpected) once the PDF host is
   reachable; update spec §0 wording if the child-specific d values differ.
2. Verify the Astro Bot counts (bots, levels, puzzle pieces, hub gates) and the Gatcha duplicate rules.
3. Verify the official MoE 2026-27 holiday PDF and replace the secondary dates in `season.calendar` if needed.
4. Do the frame-step study: record a Brawl Stars Starr Drop and a Clash Royale Lucky Chest on device and step frame
   by frame (gap_audit.md G14). Compare with the `chests.revealMs` defaults and adjust by ±20% if they feel slow.
5. Verify Radesky 2022's exact category list from PMC8942378 and align the §13 checklist labels.
