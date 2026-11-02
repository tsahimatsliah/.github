# Research log: game design (spec 01)

> This log backs `docs/spec/01-game-design.md`. Each entry gives the question, what was found, the source, an evidence grade, and the decision it supports.
> Grades: **A** = RCT/meta-analysis, **B** = small study / primary design doc / official source, **C** = secondary or press coverage, **D** = fan wiki or SEO guide (weak; used only for "what kids already know" texture, never for numbers we depend on), **I** = my design inference.
> Session constraints (2026-11-02): WebFetch failed (DNS) for hebcal.com, timeanddate.com and almo-ai.com. Only WebSearch result summaries were available, so no full pages could be read. Holiday dates were computed locally with my own implementation of the arithmetic Hebrew calendar (Reingold-Dershowitz) and checked against search snippets (they match).

---

## R1. Jewish/Israeli calendar 5787 (needed for seasons and events)

- **Method:** I wrote my own code (`scratchpad/scripts/hebcal_own.py`, not committed) that computes the molad and the dechiyot rules and maps Hebrew dates to Gregorian.
- **5787 is a leap year** (385 days, with Adar I and Adar II), so Purim is in Adar II and Pesach falls late.

| Event | Date (daytime; the holiday starts the evening before) |
|---|---|
| Rosh Hashana 5787 | 2026-09-12 (Sat) |
| Yom Kippur | 2026-09-21 |
| Sukkot | 2026-09-26 |
| Simchat Torah (Israel) | 2026-10-03 |
| Hanukkah | first candle on the evening of **2026-12-04** (Fri); days 2026-12-05 to 2026-12-12 |
| Tu BiShvat | 2027-01-23 (Sat) |
| Purim Katan (14 Adar I) | 2027-02-21 |
| **Purim** | **2027-03-23** (Tue); Shushan Purim 2027-03-24 |
| **Pesach** | seder on the evening of 2027-04-21; day 1 2027-04-22; day 7 2027-04-28 |
| Yom HaShoah | 2027-05-04 (Tue; no shift needed) |
| Yom HaZikaron | 2027-05-11 (Tue) |
| Yom HaAtzmaut | 2027-05-12 (Wed) |
| Lag BaOmer | 2027-05-25 (Tue) |
| Yom Yerushalayim | 2027-06-04 (Fri; nominal date) |
| **Shavuot** | **2027-06-11** (Fri) |
| Tisha B'Av | 2027-08-12 (Thu) |
| Tu B'Av | 2027-08-18 |
| Rosh Hashana 5788 | 2027-10-02 |
| Yom Kippur 5788 | 2027-10-11 |
| Sukkot 5788 | 2027-10-16 |
| Simchat Torah 5788 | 2027-10-23 |
| Hanukkah 5788 | first day 2027-12-25 |

- **Cross-check:** search snippets agree. They give Hanukkah as Dec 4–12 2026, Purim as Mar 23 2027, Pesach as Apr 21–29 (diaspora 8th day), Yom HaAtzmaut as Wed May 12 2027 and Shavuot as Jun 11 2027 (Israel) ([jcal 5787](https://jcal.com/holidays.aspx?hyear=5787); [calendarific IL 2027](https://calendarific.com/holidays/2027/IL); [hebcal 5787 IL PDF](https://www.hebcal.com/holidays/hebcal-5787-il.pdf)). One search summary called June Shavuot "wrong". That claim is false: in a leap year Shavuot falls in June. **Grade B** (computed and cross-checked).
- **School-vacation dates for תשפ"ז:** the official Ministry calendar was not retrievable. The Ministry preschool PDFs exist ([meyda not-jewish 2026-27](https://meyda.education.gov.il/files/PreSchool/maonot/not-jewish2026-2027.pdf)), and third-party sites conflict. **Decision:** the app reads vacation ranges from a parent-editable config, `calendar.schoolBreaks`, with defaults marked "unverified". It never hard-codes them.
- **Decision:** Yom Yerushalayim gets no event, because it is politically loaded. It is a neutral day. Tisha B'Av and Yom HaShoah are added to the "quiet days" list (see Proposed change to 00).

## R2. 99 Nights in the Forest: loop details

- Per guides, the campfire has **6 upgrade levels**. The top tier ("Eternal flame") doubles night visibility and reveals chests in a wider radius ([rblxguide campfire](https://rblxguide.com/games/99-nights-in-the-forest/campfire); [campfire upgrade path](https://rblxguide.com/games/99-nights-in-the-forest/updates/campfire-upgrade-path)). **Grade D.**
- **"Modifier flames":** a run-level modifier is chosen early (Day 1 or before Day 3; sources conflict) and changes later events ([Beebom](https://beebom.com/99-nights-in-the-forest-modifier-flames-guide); [TechWiser flame types](https://techwiser.com/99-nights-in-the-forest-all-flame-types/)). **Grade D.**
- **Rescue goal:** 4 missing children (the Koala Kid among them) ([itemlevel guide](https://itemlevel.net/ultimate-guide-to-roblox-99-nights-in-the-forest/)). **Grade D.**
- Earlier notes already cover the main loop: day = gather fuel, night = defend, classes, and a fire whose going out ends the run (`coop_economy_ios.md` §1).
- **What we take:** (a) one shared, visibly upgradable fire whose levels widen the lit zone; (b) a "modifier" chosen at the start of a run (our Plan step: the child picks the evening's "flame color" that tweaks the evening); (c) a rescue/collection goal (we rescue *lost lanterns*, and the shadowlings become friends).
- **What we drop:** the run ending when the fire goes out; horror creatures (the Deer, cultists); the paid daily class shop.
- **Originality check (I):** our fire never goes out. The enemies are lonely, not hostile, and defeating one converts it into a friend. Light is literally made of letters and numbers. There are no forest-horror tropes. The structure is common to tower defense and survival games and is not specific to 99 Nights.

## R3. Reveal choreography (Brawl Stars Starr Drops, Clash Royale Lucky Chests)

- **Starr Drops:** a base rarity, then a hidden upgrade chance before the reveal; some guides say the outcome is fixed when the drop is earned and the animation only shows it. Odds differ between guides (e.g., Rare 50% / Super Rare 28% / Epic 15%; Legendary ~2%) ([mobilematters](https://mobilematters.gg/brawl-stars/brawl-stars-starr-drops-guide); [sportskeeda](https://www.sportskeeda.com/esports/starr-drop-rates-brawl-stars-explained); [timesaver.gg](https://timesaver.gg/blog/brawl-stars-starr-drops-july-2026)). **Grade D.**
- **Clash Royale Lucky Chests (Oct 2025):** these replaced Lucky Drops. They use 1–5 stars with usually **4 "spins"**, and each spin can upgrade the tier. Supercell publishes the odds: for a 1★ Magic chest, ≈31.6% stays 1★ and 0.3% (raised to 1% in Dec 2025) reaches 5★ ([RoyaleAPI](https://royaleapi.com/blog/clash-royale-2025-q3-update); [mobilematters overview](https://mobilematters.gg/guides/clash-royale/lucky-chests-overview); [Supercell Dec 2025 notes](https://supercell.com/en/games/clashroyale/blog/release-notes/december-update-2025); [Supercell drop-rate page](https://support.clashroyale.com/hc/en-us/articles/49484957630619-Information-About-Discounts-Offers-and-Drop-Rates)). **Grade B/C.**
- **What we take (I):** the multi-tap escalation ritual (tap → wobble grows → color step → burst). Our version is **deterministic**: the number of "upgrade steps" shown equals the number of things the child actually did (e.g., hint-free, first-try, Stretch chosen). The upgrades are a *recap* of earned effort, not a roll. That satisfies D2 (fixed at earn time, rarity = what earned it) and keeps the beloved "spin-up" feel. The per-step timing tokens are my own (no public frame data, as `gap_audit.md` §3 says). They are to be checked with a frame-step study on device.

## R4. Humor development by age

- McGhee's stage model links humor to cognitive development:
  - incongruous actions toward objects (toddlers);
  - misnaming and nonsense words (≈3–5);
  - conceptual incongruity;
  - the final stage, riddles and wordplay based on sound and word ambiguity (≈7+). McGhee notes riddles are funniest in grades 1–2.
  - Lexical/phonological ambiguity is easier for young children than syntactic or idiomatic ambiguity.

  ([LitNet review of McGhee](https://www.litnet.co.za/a-contemplative-investigation-into-humour-development-sense-of-humour-and-a-few-mechanisms-underlying-humour/); [Laughter Remedy (McGhee)](https://www.laughterremedy.com/?page_id=32); [Children's humor 0–3](https://www.laughterremedy.com/2010/12/children%e2%80%99s-humor-infancy-to-age-three/)). A Turkish replication found children's humor broadly consistent with McGhee ([CEEOL](https://ceeol.com/search/article-detail?id=1128481)). **Grade B/C.**
- A study of 3-year-olds proposes this order: incongruity → transgression → others' mistaken mental states (cited in the same review). **Grade C.**
- Humor in instruction: Zillmann et al. (1980) found that humorous inserts raised children's attention to and learning from educational TV. Critiques say humor works mainly through vigilance and can over-arouse. Off-topic or inappropriate humor disrupts learning ([PMC instructional humor review](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC11814177/); [ERIC EJ307779](https://searchworks.jioinstitute.edu.in/articles/eric__EJ307779)). **Grade B/C.**
- **Decisions:**
  - Humor tiers: K3 = object/action incongruity (a sock on the head); K2 = nonsense words, slapstick, mild "burp" humor, misnaming; K1 = Hebrew wordplay, riddles, puns, gentle irony.
  - Jokes happen in transitions and in feedback around the item, **never over the item while the child is solving** (attention finding, consistent with Iacovides et al. in `risks_failure_modes.md` §1).

## R5. Co-op handicapping and asymmetric family play

- **Super Mario Odyssey:** P2 controls Cappy, who is **invincible**, can attack and manipulate objects, and can drop in at any time. Nintendo pitched it as a way to introduce players to games ([GameSpot](https://www.gamespot.com/articles/nintendo-e3-207-super-mario-odyssey-local-co-op-co/1100-6450983/); [Game Informer](https://gameinformer.com/b/news/archive/2017/06/15/see-how-local-co-op-works-in-super-mario-odyssey); [Destructoid](https://destructoid.com/super-mario-odyssey-has-a-limited-co-op-mode)). Super Mario Galaxy had a pointer "co-star" helper. **Grade C.**
- **Super Mario Bros. Wonder:** Yoshi and Nabbit take no damage and are explicitly for younger players. A parent critique says kids *want the "real" characters*, and a lives-off option would be better than segregated easy characters ([Shacknews](https://shacknews.com/article/136864/super-mario-bros-wonder-playable-characters); [Escapist](https://www.escapistmagazine.com/yoshi-nabbit-are-super-mario-bros-wonders-easy-mode/); [Nintendo Life opinion](https://nintendolife.com/features/opinion-my-daughter-made-me-realise-that-mario-wonders-difficulty-options-need-work); [Cubed3](https://www.cubed3.com/games/reviews/nintendo-switch/super-mario-bros-wonder)). **Grade C.**
  - **Decision:** the toddler's role must *look* as heroic as the others (same lit-up hero visuals and a named role, "מדליק/ת האור"). Handicapping is invisible: difficulty, not a badge of being "the baby".
- **LEGO games:** seamless drop-in/drop-out 2-player co-op, puzzles rescaled so novices and experts both contribute, and invulnerability while building ([Game Informer](https://gameinformer.com/b/features/archive/2013/01/14/ready-player-2-traveller-39-s-tales-and-co-op); [Gamepressure](https://www.gamepressure.com/editorials/the-best-couch-coop-games-for-single-screen/z82b4-3)). **Grade C.**
  - **Decision:** drop-in/drop-out in every co-op mode; nobody's state blocks the others.
- **Pais et al., CSCW 2024, "Promoting Family Play through Asymmetric Game Design"** ([DOI 10.1145/3637392](https://dl.acm.org/doi/10.1145/3637392); [summary](https://www.universityxp.com/research/2026/7/23/promoting-family-play-through-asymmetric-game-design); [Pais 2022 thesis](https://run.unl.pt/bitstream/10362/138804/1/Pais_2022.pdf)):
  - Setting: adult–child pairs with an age gap of 10+ years and tailored roles.
  - Asymmetry was *essential* to the interactions (sharing information, learning roles together).
  - **Asymmetry of time can unbalance the experience.**
  - Same-mechanics games force one player into a less engaging style.

  **Grade B (small n).**
  - **Decision:** roles carry *information asymmetry*: K1 reads clues that K2 needs, and K2 sees things that K1 can't. Each child's time-on-task per co-op minute is balanced to within ±25% (acceptance criterion).
- **CHI 2012 parent–child co-located co-op** ([unpaywall](https://unpaywall.org/10.1145%2F2307096.2307147)): designers must plan *progression* in relation to balance between players. Figame (2025, ages 8–12) supports role switching ([arXiv 2503.14178](https://arxiv.org/abs/2503.14178)). **Grade B.**

## R6. Temple Run-style runner (for a co-op learning runner)

- Temple Run 2 uses swipes for turns, jumps and slides, and tilt to change lane. Swiping too early makes the runner trip and swiping too late is fatal. Coins buy upgrades. Power-ups are a shield, a coin magnet and a boost. Gems revive the runner at an escalating price ([Pocket Gamer tips](https://www.pocketgamer.com/temple-run-2/temple-run-2-hints-tips-and-tricks-how-to-stay-on-your-feet/); [Giant Bomb](https://www.giantbomb.com/games/3030-41247/); [Udonis](https://blog.udonis.co/mobile-marketing/mobile-games/temple-run-2)). **Grade C.**
- **Decisions:**
  - Keep: auto-run, 3 lanes, magnet/shield fantasy, the junction "choice" moment (it becomes the answer choice).
  - Drop: death, escalating revive cost, endless length, tilt controls (unreliable for small kids; an iPad on a table can't tilt).
  - Runs are fixed length (60–120 s) and end at a destination. A collision is a 0.5 s stumble, never a death.

## R7. Open play (Pok Pok / Toca) for K3's playroom

- Pok Pok: "a playroom floor filled with toys, only digital"; "no goal…no winning or losing, no levels". Nearly every sound was recorded by hand to be calming, and the app is designed to keep young kids emotionally regulated ([Apple Behind the Design](https://developer.apple.com/news/?id=5bcex7xf); [Sketch](https://www.sketch.com/blog/pok-pok/); [GamesBeat](https://gamesbeat.com/pok-pok-playroom-mobile-app-lets-kids-engage-in-open-ended-play/); [9to5Mac](https://9to5mac.com/2021/05/20/hands-on-with-pok-pok-playroom-kids-app/)). **Grade B.**
- **Decisions:**
  - K3's "חדר הגחליליות" playroom has no goals and no scores.
  - Sounds are soft, organic and recorded (foley-style, from the Kenney/recorded library). There are no stingers.
  - Learning content (colors, counting to 3, animal words) appears only as *labels the toy speaks* when touched (e.g., the "three ducks" toy says "שָׁלוֹשׁ בַּרְוָזִים" from the numeral lookup table).

## R8. Toddler fear of dark and "monsters" (night theme safety)

- Parenting and clinical sources agree that fear of the dark and of monsters typically emerges around **24–36 months**, as imagination outpaces reasoning, and often ties to bedtime separation ([Romper](https://www.romper.com/p/when-do-kids-become-afraid-of-the-dark-heres-how-to-help-your-little-one-be-brave-16949663); [Huckleberry](https://huckleberrycare.com/blog/dealing-with-nighttime-fears-in-children-overcoming-fear-of-the-dark-and); [Ovia](https://oviahealth.com/?p=103429)). **Grade C/D** (no primary study retrieved).
- **Decisions.** K3 is exactly in the onset window, so K3 never sees:
  - the "evening" palette darker than dusk;
  - shadowlings approaching;
  - any creature emerging from darkness.

  K3's shadowlings are already-friendly "sleepy" ones that want a blanket. The night-defense mode for K3 is "tuck-in": cover the sleepy shadowling with a blanket so it glows. Global rules:
  - no darkness covering more than 40% of the screen for any child;
  - evening light level never below a defined token (`palette.dusk.minLuminance`);
  - the fire is always visible in the evening scenes.

## R9. Emotes (Brawl Stars pins)

- Pins are battle emotes visible to team and opponents. They are the main expressive channel in a game without chat, used to celebrate, thank or taunt. Guides advise restraint (one well-timed pin beats spam). There are 3 brawler-specific slots plus 2 universal ones ([mobilematters](https://mobilematters.gg/brawl-stars/how-to-change-pins-in-brawl-stars); [esports.net](https://www.esports.net/news/mobile-games/how-to-change-emotes-in-brawl-stars/); [Brawl Stars etiquette blog](https://pug-heron-a4h3.squarespace.com/blog/communication-and-mentality-thoughts-on-brawl-stars-social-etiquette)). **Grade D.**
- **Decisions:**
  - Emote wheel with 4 slots and a 3 s cooldown.
  - Emotes are the only sibling-to-sibling channel in multi-device co-op (no chat; matches C7 and the App Store 5.1.4 spirit).
  - **No taunt emotes exist.** The catalog is limited to celebrate / thanks / help-me / look-here / silly.

## R10. Daily return without punishment (Animal Crossing)

- New Horizons "remembers your absence without punishing it" (villagers comment, a few cockroaches). Daily surprises make the routine feel alive ([GeekChamp](https://geekchamp.com/the-nostalgia-and-surprise-of-returning-to-animal-crossing-new-horizons/)). Counter-evidence: island rating drops without upkeep, and Nintendo has sent "your island misses you" emails ([ScreenRant](https://screenrant.com/animal-crossing-daily-task-guide-2022/); [TheGamer](https://www.thegamer.com/nintendo-animal-crossing-new-horizons-hiatus-guilt-trip-email/)). **Grade C.**
- **Decisions:**
  - A daily "something new at camp" surprise (a letter in the mailbox, a new visiting shadowling friend). The surprise accumulates: after 3 days away, there are 3 letters waiting.
  - **Zero decay of any kind:** no rating drop, no weeds or cockroaches. The return greeting is happy, never guilt: "התגעגענו! תראו מה קרה במחנה".
  - No "misses you" messages to children ever. Parent notifications follow 00 E3.

## R11. Gaps not closed (carry forward)

- No primary frame-timing data for Supercell or Duolingo reveals. Our timing tokens are proposals to be set by an on-device frame-step study (spec acceptance criterion CEL-AC-6).
- No study on multi-age (2.5/5.5/8.5) sibling co-op learning games. The handicapping formulas in spec §9 are design inference and must be tuned from telemetry (time-on-task balance, help-request counts).
- The official תשפ"ז school vacation calendar could not be retrieved. It is parent-configurable.
- I could not retrieve 99 Nights primary or developer sources. Fan wikis were used only for texture.
