# Brawl Stars Game Design Teardown (state: early November 2026) and a Blueprint for "מחנה הגחליליות"

> Research note, written 2026-11-04. **Design analysis only.** Nothing here comes from decompiling, datamining or extracting assets. Everything is public writing (Supercell posts, guides, wikis, talks, news) or careful observation-level background knowledge.
>
> **Purpose:** take apart Brawl Stars (the kids' favourite, 00 A4) system by system. Then sort every mechanic into ADOPT, ADAPT or REJECT against our guardrails (00 §C, §D, §G; 02 §0) and turn the result into a concrete blueprint for our game.
>
> **What this note avoids repeating.** Visual and UI anatomy (lobby layout, buttons, fonts, rarity colours, HUD colours, reveal timing tables) is in `supercell_ui_style_guide.md`. Art and 3D look is in `supercell_art_3d_style_guide.md`. SDT, flow and white-hat versus black-hat drives are in `astrobot_joy_gamification_psychology.md`. The 2022–2026 loot-box history summary is in `../אפליקציית למידה משחקית לילדים/hit_games_engagement.md` §1. This note covers **systems, numbers, rules, pacing, flows and the ethics mapping.**
>
> **Method and limits.** In this cloud session, direct page fetches to `supercell.com`, `brawlstars.fandom.com` and `naavik.co` failed DNS (they are outside the network allowlist). Every fact therefore comes from web-search result summaries of those and other pages. The cited URL is the page the summary describes. Exact wording should be checked before quoting. **To re-verify, the parent can add `supercell.com`, `brawlstars.fandom.com` and `naavik.co` to the session's allowed network domains.**
>
> **Evidence tags:**
> - **[S]** Supercell's own post, notes or talk (seen through a search summary).
> - **[S3]** A third-party guide or news source.
> - **[C]** Sources conflict; the range is given.
> - **[BK]** Long-standing background knowledge of the game, not re-verified this session.
> - **[INF]** My inference or design analysis.
>
> Our own spec numbers are always quoted from 02 and 01. Proposed changes are marked **⟦PROPOSED⟧**.

---

## 0. TL;DR

1. **Brawl Stars is three nested clocks.** The *match* is 2–3 minutes. The *daily* cycle is Daily Wins plus quests. The *seasonal* cycle is a Brawl Pass of about 28 days, a monthly Ranked season and a Pro Pass of about 4 months. Each clock has its own visible bar. The "one more game" pull comes from always being one bar-tick away from a reward. [INF from §1–§2, §8]
2. **The match is the product.** Supercell rebuilt the core twice in a year-long beta. Two control models were tried, then a third, and landscape twin-stick won. Progression was also redesigned more than once ([GDC 2019, Summala](https://www.gamedeveloper.com/design/video-designing-the-control-and-progression-schemes-of-i-brawl-stars-i-)) [S]. "Tap to auto-aim, drag to aim" is the single most important accessibility trick for kids ([PocketGamer guide](https://www.pocketgamer.com/brawl-stars/brawl-stars-tips-and-tricks-a-guide-for-the-beginner-brawler/)) [S3].
3. **Progression is a stack of throttles:** trophies → Trophy Road → Credits → brawler unlock → Power Points plus Coins → Power 1–11 → gadgets (P7) → star powers (P9) → gears → hypercharge (P11) → buffies. **Coins are the bottleneck.** Maxing one brawler costs about **3,740 PP + 7,765 coins** for levels alone, plus about 5,000 coins of abilities and a 5,000-coin hypercharge ([Sports360](https://sports360news.com/en/esports/brawl-stars-upgrade-costs-explained/11143150), [Simple Game Guide](https://simplegameguide.com/brawl-stars-upgrade-calculator/)) [S3].
4. **Randomness was removed, then re-added in a nicer form.** Brawl Boxes were removed in Dec 2022 ([Game Developer](https://gamedeveloper.com/business/supercell-pulls-loot-boxes-from-brawl-stars-in-favor-of-deterministic-rewards-)) [S]. Revenue then fell about 14% over 5 months ([Deconstructor of Fun](https://www.deconstructoroffun.com/blog/why-removing-loot-boxes-in-brawlstars-failed)) [S3]. Free, earned, tap-to-upgrade **Starr Drops** came in June 2023, and from June 2023 to Feb 2024 MAU rose ×2.4, DAU ×3.9 and revenue ×8.8 ([mobilegamer.biz on the GDC 2024 talk](https://mobilegamer.biz/supercell-explains-brawl-stars-big-comeback-from-an-all-time-low-to-8-8x-revenue/)) [S]. **Lesson for us:** the *reveal ritual* drives engagement more than the randomness does. 02 §6 already keeps the ritual and drops the randomness.
5. **Supercell keeps removing complexity and loss.** Club League, club quests and the club shop were removed and replaced by a monthly Mega Pig weekend ([Supercell club changes](https://supercell.com/en/games/brawlstars/blog/news/important-club-changes-are-coming/)) [S]. Masteries were removed in June 2025: "only ~12% of players ever got a title" ([gamingonphone](https://gamingonphone.com/news/brawl-stars-developer-team-explains-why-masteries-were-removed)) [S3]. The trophy-season reset was removed in Feb 2026 ([Supercell Threads post](https://www.threads.com/@brawlstars/post/DUJALjXjD95/new-trophy-system-were-applying-changes-to-the-current-trophy-system-after-the)) [S]. The Ranked reset was softened from −6 to −2 ranks in Nov 2026 ([Release Notes Nov 2026](https://supercell.com/en/games/brawlstars/blog/game-updates/release-notes-november-2026/)) [S]. **The industry leader is moving toward our C5, more slowly.**
6. **What kids love:** short matches, characters with big personality, playing with friends, the reveal moment. **What they hate:** pay-to-win, matchmaking, bad teammates, losing trophies ([Kimola App Store analysis](https://kimola.com/reports/unlock-brawl-stars-success-exclusive-feedback-analysis-report-app-store-us-150573)) [S3]. Every one of those complaints comes from PvP plus money, and our game has neither.
7. **The core lessons for our game:** a 1-tap "Play"; a short, finite, clearly ending unit; a visible next-reward bar on every screen; one escalating reveal per session; a roster of collectible characters with personality; new verbs unlocked one at a time; season themes that re-skin the hub. All of this is already allowed by our specs (§12, §13).
8. **Never copy:** PvP loss, daily-win obligations (the 6th-win Chaos Drop), timers on quests, events and shop, paid pass tracks, gems for money, random paid rewards, ranked resets, MVP comparison, open chat.

---

## 1. Core loop: the 2–3 minute match

### 1.1 Match formats

| Mode | Team | Win rule | Length / rounds | Respawn | Source |
|---|---|---|---|---|---|
| **Gem Grab** | 3v3 | Hold 10+ gems, which starts a countdown; dropping below 10 stops it | ~2–3 min; countdown ~15 s [BK] | Yes, a few seconds [BK]; a defeated carrier drops their gems (built-in comeback) [BK] | [Mobile Matters modes](https://mobilematters.gg/brawl-stars/game-modes-explained) [S3] |
| **Brawl Ball** | 3v3 (5v5 variants) | First to 2 goals | ~2:30 plus overtime [BK] | Yes [BK] | [brawls.wiki](https://brawls.wiki/game-modes/) [S3] |
| **Heist** | 3v3 | Destroy the enemy safe first | ~2:30 [BK] | Yes | [Liquipedia](https://liquipedia.net/brawlstars/Brawl_Stars) [S3] |
| **Bounty** | 3v3 | 20 stars first, or the most at time-out; each kill raises your own bounty by 1, up to 7 | ~2 min [BK] | Yes | [Liquipedia](https://liquipedia.net/brawlstars/Brawl_Stars) [S3] |
| **Hot Zone** | 3v3 | Zone control: 100 s combined per Liquipedia; 60 s per another source [C] | ~2 min | Yes | [Liquipedia](https://liquipedia.net/brawlstars/Brawl_Stars) [C] |
| **Knockout** | 3v3 | Best of 3 rounds; wipe the enemy team; **no respawn** | 3 short rounds | No | [Liquipedia](https://liquipedia.net/brawlstars/Brawl_Stars) [S3] |
| **Showdown** | Solo 10 / Duo 5×2; **20-player Showdown** announced Oct 2026 | Last one standing; power cubes grow your brawler; a poison cloud closes in | ~2–3 min | No (Duo: the partner revives) [BK] | [gamingonphone Oct 2026 Brawl Talk](https://gamingonphone.com/news/brawl-stars-october-2026-brawl-talk-ranked-rework-new-brawlers-game-modes-buffies-and-more/) [S3] |
| **PvE / boss events** | Co-op (incl. the first-ever **20-player Boss Fight**, Duolingo Sept 2026) | Beat a giant boss together; in the Duolingo event, **quiz questions mid-battle** | Event | — | [GamesBeat](https://gamesbeat.com/brawl-stars-and-duolingos-mascots-face-off-in-new-collaboration/) [S3] |

**[INF] Why 2–3 minutes works:**
- **Time to first action is about 5 s** after spawn [BK]. No build phase, no shopping, no lane farming. Naavik calls the game "the MOBA reimagined" for exactly this compression ([Naavik](https://naavik.co/digest/brawl-stars-moba-reimagined/)) [S3].
- **Each match is a full story arc:** spawn → skirmish → objective swing → climax (countdown, overtime, poison) → banner.
- **Built-in comebacks** (gem drop on death, ball turnovers, Super charge) keep losing teams hopeful for most of the match.
- **Low stakes per match.** One loss costs little and the next match is about 10 s away.
- **Variety without a rules lesson.** Every mode reuses the same 3 verbs (move, attack, Super). Only the *goal* changes.
- **Depth unfolds slowly:** "Brawl Stars gradually exposes depth as players level up each individual Brawler and unlock new ones" ([Deconstructor of Fun](https://deconstructoroffun.com/blog/2024/9/30/4-x-reasons-why-squad-busters-suffers-while-brawl-stars-soars)) [S3].

### 1.2 Controls

- **Left stick = move.** **Right red stick = attack: tap fires at the nearest target (auto-aim); drag to aim, release to fire** ([PocketGamer](https://www.pocketgamer.com/brawl-stars/brawl-stars-tips-and-tricks-a-guide-for-the-beginner-brawler/)) [S3].
- **Yellow skull stick = Super**, charged by landing hits. It also supports tap or drag ([PocketGamer](https://www.pocketgamer.com/brawl-stars/brawl-stars-tips-and-tricks-a-guide-for-the-beginner-brawler/)) [S3].
- **Gadget button:** limited uses per match, 3 by default [BK]. **Hypercharge (purple):** charges more slowly than the Super from the same actions, then is pressed to activate a short power-up of the Super and stats ([Mobile Matters](https://mobilematters.gg/brawl-stars/hypercharge-explained)) [S3].
- **Every button can be moved** (Settings → Edit Controls) ([ar-pay](https://ar-pay.com/blog/en/articles/brawl-stars/)) [S3]. Default colours and positions are in `supercell_ui_style_guide.md` §6.5.
- **Design history:** the team tried tap versus stick, then moved to landscape, which made a second stick possible. Controls were "much more demanding" than earlier Supercell games, so touch simplicity got heavy prototyping ([GDC news](https://gdconf.com/news/go-inside-design-supercells-new-hit-game-brawl-stars-gdc-2019); [DTF write-up](https://dtf.ru/gamedev/57819-razrabotka-sistemy-upravleniya-i-progressa-opyt-sozdatelei-brawl-stars)) [S].
- **Ammo:** about 3 segments that reload over time. It throttles spam and teaches rhythm [BK].

**[INF] The kid-accessibility pattern:** *one default that always works* (tap = auto) **plus** *an expert layer that pays off later* (drag aim, leading shots, Super timing). A 5-year-old can play by tapping; an 8-year-old can master aiming.

### 1.3 Readability

- Top-down 3/4 camera with the whole local fight in view [BK]. The camera question (orthographic vs perspective) is in `supercell_art_3d_style_guide.md` §5.1.
- **Team is shown by colour** (blue = us, red = them) on rings and health bars. Ammo sits under your own brawler. A Super-ready glow appears on the button [BK].
- **Bushes** hide brawlers and **walls** block shots. One grid language is used across all maps [BK].
- Each brawler has a **distinct silhouette and attack shape** (cone, line, lob, area), so you can read the projectile type at a glance [BK]. The GDC 2025 talk "Making of a Brawler" covers the character-creation rules ([GDC schedule](https://schedule.gdconf.com/session/making-of-a-brawler-character-creation-for-brawl-stars/907875)) [S].

### 1.4 Matchmaking and bots

- Supercell says solo matchmaking uses the selected brawler's trophies, and to a lesser degree total trophies, "to protect less-skilled players" ([Gamertweak](https://gamertweak.com/brawl-stars-matchmaking-explained)) [S3].
- Community reports describe **bots at low trophies** ("0–1000 super easy"). These are fan-written and unofficial ([speedrun.com post](https://www.speedrun.com/sv-SE/posts/nd70r)) [S3]. A 2021 Duels bug with bots was confirmed by a community manager ([PiunikaWeb](https://piunikaweb.com/2021/12/30/brawl-stars-duels-bug-biggest-xploiters-or-abusers-could-suffer-a-ban)) [S3].
- **[INF]** Undisclosed bots make a beginner feel competent. Our version is *disclosed* scripted helpers (§12, row 54).

---

## 2. Session design

| Measure | Value | Source |
|---|---|---|
| Match length | "Rarely exceeds 3 minutes" | [Kimola](https://kimola.com/reports/unlock-brawl-stars-success-exclusive-feedback-analysis-report-app-store-us-150573) [S3] |
| Session length | About 22 min per session, about 4 sessions per day (third-party estimates) | [Udonis](https://www.blog.udonis.co/mobile-marketing/mobile-games/brawl-stars-player-count) [S3, weak] |
| Genre benchmark | Midcore: 6–7 sessions per day; top-tier session length 8–9 min | [GameAnalytics 2025 benchmarks](https://gameindustrylibrary.com/documents/gameanalytics-mobile-gaming-benchmarks-2025) [S3] |
| DAU / MAU | About 18.2M DAU (Jan 2025) → about 16.9M (Jun 2025), estimates; DAU decline "stopped" by update 63 (Sept 2025) | [Udonis](https://www.blog.udonis.co/mobile-marketing/mobile-games/brawl-stars-player-count); [PocketGamer.biz](https://www.pocketgamer.biz/march-2026-mobile-game-charts-brawl-stars-revenue-rises-nearly-50/) [S3] |

**[INF] A typical session is about 7 matches**: 22 min ÷ about 3 min per match plus menus. That is close to our K1 episode of 15–20 min (00 C3). The *duration* is not the problem; the *hooks that pull you back* are.

### 2.1 "One more game" hooks, ranked by strength [INF]

1. **Daily Wins track.** Six win slots per day, each with a reward. **The 6th win of each day always includes a Chaos Drop** ([timesaver Starr Drops](https://timesaver.gg/blog/brawl-stars-starr-drops-july-2026)) [S3]. The mix of the other 5 differs by source and season [C] ([BrawlZone](https://brawlzone.net/starr-drops)). Before 2026 it was 3 Starr Drops on the 1st, 4th and 8th/9th win ([mobilegamer.biz](https://mobilegamer.biz/supercell-explains-brawl-stars-big-comeback-from-an-all-time-low-to-8-8x-revenue/); [Game World Observer coverage via search](https://gameworldobserver.com/2022/12/13/brawl-stars-loot-boxes-removed-supercell)) [S]. *Note: once the 6 are done, the daily reward pull drops sharply. That is a soft natural stopping point (good design) wrapped in a daily obligation (bad for kids).*
2. **Win streaks:** up to **+10 extra trophies per win** on the same brawler ([timesaver trophy push](https://timesaver.gg/blog/brawl-stars-trophy-push-guide-august-2026)) [S3]. Streaking makes stopping feel like "wasting" it.
3. **Trophy Road next milestone**, always shown with a progress bar.
4. **Brawl Pass tier bar plus quests.** Quests are split into **Daily / Season / Event, each with a timer** ([Supercell UX notes](https://supercell.com/en/games/brawlstars/blog/release-notes/optional-update)) [S].
5. **Limited events**: Mega Quests (Easter 2026: earn 100 trophies per day for 4 days) ([Sportsdunia](https://www.sportsdunia.com/gaming/brawl-stars-easter-mega-quests-2026)) [S3]; Brawl-O-Ween phases ([MMO Culture](https://mmoculture.com/news/brawl-stars-brawl-o-ween-2026-events/)) [S3].
6. **Club Mega Pig weekend**: tickets per member, wins fill a shared pig ([Guild Order](https://guildorder.com/games/brawl_stars/wiki/mega-pig-club-event)) [S3, weak].
7. **Daily Freebie** on the Supercell Store web shop. It is often a 24-hour item ([Sportsdunia](https://www.sportsdunia.com/gaming/brawl-stars-free-rewards)) [S3].
8. **The team lobby "play again" loop with friends** [BK]: it is socially costly to leave.

### 2.2 The daily rhythm [INF reconstruction]

```
open app → claim pending (Trophy Road / Pass / freebie) ─┐
  → Play (1 tap) → match 2–3 min → results → drop reveal ┤ ×6 "Daily Wins"
  → quest ticks → pass bar → maybe upgrade a brawler ─────┘
  → 6th win: Chaos Drop → (soft stop) or continue for trophies/streak
```

---

## 3. FTUE and onboarding

### 3.1 What is documented

- A **mandatory tutorial as Shelly**: move, attack, aim (auto and manual) and Super. Then a **practice match against AI bots** ([PlayerVault](https://www.playervault.gg/guides/brawl-stars/beginner-guide); [Gamer-Choice](https://gamer-choice.com/brawl-stars-beginners-guide/)) [S3].
- **Gating by trophies:**
  - Starr Drops at 50 trophies ([Mobile Matters](https://mobilematters.gg/brawl-stars/brawl-stars-starr-drops-guide)) [S3, possibly dated].
  - Records at 350 ([Mobile Matters](https://mobilematters.gg/news/brawl-stars/masteries-removal-explained)) [S3].
  - **Ranked at 5,000, up from 1,000 in Nov 2026** ([Release Notes Nov 2026](https://supercell.com/en/games/brawlstars/blog/game-updates/release-notes-november-2026/)) [S].
  - Game modes unlock along the road; a Dec 2023 tweak moved them earlier ([Trophies wiki via search](https://brawlstars.fandom.com/wiki/Trophies)) [S3].
- **Early Trophy Road was buffed "up to 25k trophies for new players so they can catch up"** ([Supercell blog via search](https://supercell.com/en/games/brawlstars/blog/news/new-power-brawl-pass-changes-and-a-new-starr-drop-2/)) [S].
- **Power-level gating:**
  - Gadgets at P7, Star Powers at P9, Hypercharge at P11 ([Simple Game Guide](https://simplegameguide.com/brawl-stars-upgrade-calculator/)) [S3].
  - Gears at P8 and P10, or only at P10 [C] ([timesaver gears](https://timesaver.gg/blog/brawl-stars-gears-guide-september-2026)).

### 3.2 First 30 minutes, minute by minute [BK + INF reconstruction; no official source]

| Min | What happens | Design job |
|---|---|---|
| 0:00–0:30 | Splash, loading, age/terms prompt, language | Get to play quickly |
| 0:30–2:30 | Tutorial island as Shelly: move to a marker → shoot dummies (tap = auto) → drag to aim → fill the Super → fire the Super | **One verb per step, learned by doing** |
| 2:30–5:00 | First real match (Showdown or Gem Grab) against easy bots, a near-certain win, Victory banner | **The first win is guaranteed** |
| 5:00–6:00 | Results: trophies count up; first Trophy Road reward claimed with a guided tap (coins/PP); the lobby is revealed | First reward ≤6 min |
| 6:00–15:00 | 3–4 more matches; modes/brawlers unlock on the road; first Starr Drop at about 50 trophies (escalating tap reveal); first upgrade prompt (P1→P2 is cheap: 20 PP + 20 coins, §4.2) | **A new thing on almost every match** |
| 15:00–30:00 | About 8–12 matches total; 100–250 trophies; first new brawler(s) via the road or Brawler Blast; Brawl Pass and quests introduced with badges; friend/club prompts | Show the long roads (Trophy Road, Pass) so the player sees a future |

**[INF] First-week complexity gating.** Days 1–2 are mostly *verbs and modes*. Days 3–5 bring *collection* (new brawlers, the first P7 gadget). The end of week 1 brings *optimisation* (choosing whom to upgrade, quests). Ranked, Pro Pass and buffies are weeks to months away. **One new system is exposed per play session, and each arrives with a badge and a guided first use.**

---

## 4. Progression systems: rules and pacing

### 4.1 Trophies, Trophy Road and Prestige

- **Per-brawler trophies.** You win trophies with a brawler and the account total is the sum [BK]. Wins give fewer trophies and losses cost more as a brawler climbs ([Trophies wiki](https://brawlstars.fandom.com/wiki/Trophies)) [S3].
- **Base win:** about +10, up from +8. **Win streak:** up to +10 extra ([timesaver](https://timesaver.gg/blog/brawl-stars-trophy-push-guide-august-2026)) [S3].
- **Underdog matches:**
  - Underdog trophies are on only below 2,000 trophies, per the Feb 2026 notes ([Release Notes Feb 2026](https://supercell.com/en/games/brawlstars/blog/release-notes/release-notes-february-2026/)) [S].
  - **No trophies are lost when losing an Underdog match** ([Trophies wiki change log](https://brawlstars.fandom.com/wiki/Trophies)) [S3].
- **Trophy Road** (account-level, all brawlers' trophies combined):
  - The June 2025 rework turned the road into Starr Park "Worlds" and **extended it from 70,000 to 100,000**. It added Legendary and Hypercharge Starr Drops and moved Mastery's resources into the road ([Release Notes June 2025](https://supercell.com/en/games/brawlstars/blog/game-updates/release-notes-june-2025/)) [S].
  - World 5 was added later and rewards along the whole road were raised on 16/12/25 ([Trophies wiki](https://brawlstars.fandom.com/wiki/Trophies)) [S3].
  - Legendary Starr Drops sit at 13k, 18k and 23k ([hit_games note §1](../אפליקציית%20למידה%20משחקית%20לילדים/hit_games_engagement.md)) [S3].
- **Prestige (since Feb 2026):**
  - The seasonal trophy reset and Trophy Boxes were **removed**. Leaderboard Prestige became a legacy stat ([Supercell Threads](https://www.threads.com/@brawlstars/post/DUJALjXjD95/new-trophy-system-were-applying-changes-to-the-current-trophy-system-after-the)) [S].
  - Each brawler hits Prestige 1 at 1,000 trophies, then a new Prestige every 1,000, with **no cap**. Rewards include a golden title at P1, a neon icon at P2 and a neon title at P3 ([PlayNews](https://www.playnews.gg/en/guides/how-to-get-prestige-in-brawl-stars-how-many-trophies-and-what-rewards); [timesaver Prestige](https://timesaver.gg/blog/brawl-stars-prestige-guide)) [S3].
- **[INF] Pacing.** Without resets, a brawler's trophies only rise in the long run. Daily losses still exist.

### 4.2 Power levels 1–11 (per brawler)

| Step | Power Points | Coins | Unlocks at the new level |
|---|---|---|---|
| 1→2 | 20 | 20 | — |
| 2→3 | 30 | 35 | — |
| 3→4 | 50 | 75 | — |
| 4→5 | 80 | 140 | — |
| 5→6 | 130 | 290 | — |
| 6→7 | 210 | 480 | **Gadget slot** (buy for 1,000 coins) |
| 7→8 | 340 | 800 | Gear slot 1 [C] |
| 8→9 | 550 | 1,250 | **Star Power** (2,000 coins) |
| 9→10 | 890 | 1,875 | Gear slot (2) [C] |
| 10→11 | 1,440 | 2,800 | **Hypercharge** (5,000 coins); **Buffies** |
| **Total** | **3,740** | **7,765** | |

- **Sources.** The totals and the 9→10 and 10→11 steps are sourced ([Sports360](https://sports360news.com/en/esports/brawl-stars-upgrade-costs-explained/11143150); [Brawls wiki](https://brawls.wiki/upgrade-calculator/); [CTBS](https://ctbsdb.com/ctbs/en/tools/upgrade)) [S3]. The earlier steps are [BK]. **They add up exactly to both sourced totals**, which supports them.
- **Ability prices:** gadgets 1,000; star powers 2,000; hypercharge 5,000 ([Simple Game Guide](https://simplegameguide.com/brawl-stars-upgrade-calculator/)) [S3]. Gears cost 1,000 (Super Rare), 1,500 (Epic) or 2,000 (Mythic) ([timesaver gears Sept 2026](https://timesaver.gg/blog/brawl-stars-gears-guide-september-2026)) [S3].
- **Full max, 2 gadgets + 2 star powers + hypercharge:** about **3,740 PP + 12,740 coins** before gears and buffies ([everycalculators](https://everycalculators.com/brawl-stars-upgrade-calculator.html)) [S3]. The step curve grows about ×1.6 per level [INF]. Early levels are trivial and the last two cost 57% of all coins.
- **Gems → Power Points:** the default rate went from 1 gem = 5 PP to **1 gem = 10 PP on 16/12/25** ([Power Points wiki](https://brawlstars.fandom.com/wiki/Power_Points)) [S3].

### 4.3 Abilities: gadgets, star powers, gears, hypercharges, buffies

| System | Rule | Source |
|---|---|---|
| Gadget | P7; 2 per brawler, equip 1; limited uses per match [BK] | [Simple Game Guide](https://simplegameguide.com/brawl-stars-upgrade-calculator/) [S3] |
| Star Power | P9; 2 per brawler, equip 1; passive | same [S3] |
| Gear | Slots at P8/P10 [C]; Super Rare/Epic/Mythic at 1,000/1,500/2,000 coins; reports of Epic/Mythic removal are unclear [C] | [timesaver Jul 2026](https://timesaver.gg/blog/brawl-stars-gears-guide-july-2026); [Sept 2026](https://timesaver.gg/blog/brawl-stars-gears-guide-september-2026) |
| Hypercharge | P11; 5,000 coins, purchasable about 2 weeks after release (paid offers first); charges more slowly than the Super; per-brawler effect (e.g., Penny's doubles firepower) | [Mobile Matters](https://mobilematters.gg/brawl-stars/hypercharge-explained) [S3] |
| Buffies (Dec 2025 →) | Power-ups *for* a gadget, star power or hypercharge. A **Claw Machine** pull costs **1,000 coins + 2,000 PP**. Once a pool is cleared, Bling chroma pulls cost 10,000. 3 per brawler. They come in batches per season (6 brawlers × 3 in Sept 2026; 6 more announced Oct 2026). Pass **Buffie Keys** give another source | [Mobile Matters](https://mobilematters.gg/guides/brawl-stars/buffies-explained); [timesaver Sept 2026](https://timesaver.gg/blog/brawl-stars-buffies-explained-september-2026); [esports.gg](https://esports.gg/news/mobile/brawl-talk-buffies-chaos-drops/) [S3] |

- **Hypercharge discourse.** At launch, Supercell's Frank Keienburg said "some of our players were concerned that we'd introduce an 'I win!'-button, but Hypercharge is far from that" ([PocketGamer interview](https://pocketgamer.com/brawl-stars/frank-keineburg-interview-hypercharge-update/)) [S]. PocketGamer.biz credits Starr Drops plus Hypercharges with the 2024 revenue rise ([PocketGamer.biz](https://www.pocketgamer.biz/have-supercells-changes-supercharged-its-revenue/)) [S3].
- **[INF] The pattern.** Each new power layer (gadget → SP → gear → HC → buffy) is (a) a fresh "verb" that renews interest in brawlers players already own and (b) a new coin sink. It is also where the pay-to-win perception comes from.

### 4.4 Mastery (removed) → Records

- **Mastery (Feb 2023 to June 24, 2025):**
  - Points came only from wins, scaled by trophies, with +20% for Star Player and a cap of 6,000 per day ([Mobile Matters](https://mobilematters.gg/brawl-stars/mastery-ranks-points-rewards-brawl-stars)) [S3].
  - It was removed with a "#1000Masteries" farewell event ([Supercell "RIP Masteries"](https://supercell.com/en/games/brawlstars/blog/news/rip-masteries)) [S].
  - Supercell said it "wasn't working well from a design perspective". Many players didn't understand the trophy-to-mastery scaling, and only about 12% ever got a title ([gamingonphone](https://gamingonphone.com/news/brawl-stars-developer-team-explains-why-masteries-were-removed); [TechIssuesToday](https://techissuestoday.com/brawl-stars-masteries-replaced-with-records-backlash)) [S3].
- **Records** (achievements per brawler and per account; unlock at 350 trophies):
  - They took over Mastery's icons, pins and titles ([Supercell Records post](https://supercell.com/en/games/brawlstars/blog/game-updates/about-the-update-records-changes)) [S].
  - Completing all Records on one brawler takes about 300 wins ([timesaver Records 2026](https://timesaver.gg/blog/brawl-stars-records-guide-2026)) [S3, weak].
  - Supercell said Records "went through many design pivots… we made a call to ship the simplest version we could". Its general manager wrote that "we didn't only fail players with Records; we failed the whole Brawl Stars team" ([2025 in review](https://supercell.com/en/games/brawlstars/blog/community/2025-in-review-franks-blog-post)) [S].
- **[INF] Lesson:** an opaque conversion formula (trophies → mastery points) kills a progression system. **One legible scale wins.** This is our C15 (one level vocabulary).

### 4.5 Ranked, Pro Pass and esports

- **Ranks:** 8 tiers (Bronze, Silver, Gold, Diamond, Mythic, Legendary, Masters, Pro). Bronze through Masters have divisions I–III, so 22 ranks in total. It is draft-based (pick and ban) and requires P11 ([trophycoach](https://trophycoach.com/brawl-stars/guides/brawl-stars-ranked-mode-guide); [timesaver Jul 2026](https://timesaver.gg/blog/brawl-stars-ranked-guide-july-2026)) [S3].
- **Nov 2026 "Ranked Refresh"** ([Release Notes Nov 2026](https://supercell.com/en/games/brawlstars/blog/game-updates/release-notes-november-2026/); [APKPure summary](https://apkpure.com/news/brawl-stars-november-2026-what-the-ranked-refresh-changes); [timesaver recap](https://timesaver.gg/blog/brawl-stars-brawl-talk-recap)) [S]:
  - Unlock raised from 1,000 to 5,000 trophies. Players who already had it keep it and get a Chaos Drop instead.
  - No teams at Masters I and above.
  - Free maxed brawlers per season raised from 3 to 6.
  - Larger Elo gains in early and middle ranks, more points for a 2–0.
  - **The season reset drops 2 ranks instead of 6.**
  - The pick-and-ban screen gained class sorting, ban suggestions, quick chat and a 3D map preview.
- **Pro Pass:**
  - 100 tiers over about 4 months. Free track plus a **paid track at US$24.99**.
  - Pro skin upgrades at tiers 10/30/60/100 are claimable only during the active season.
  - XP comes from ranked wins, rank-ups **and watching esports**. Premium doubles XP and the weekly cap ([Mobile Matters](https://mobilematters.gg/guides/brawl-stars/ranked-pro-pass-explainer); [esports.gg](https://esports.gg/news/mobile/brawl-stars-ranked-pro-pass-details/)) [S3].
- The **Pro rank unlocks the esports button** in-game ([esports.gg](https://esports.gg/news/mobile/brawl-stars-ranked-pro-pass-details/)) [S3].

### 4.6 Account level, XP and the passes

- **Brawl Pass XP** comes from quests and events. Tiers are the account-level "level" players actually track [BK]. The pass-era "Experience Level" was replaced by Mastery in 2023 ([Mobile Matters](https://mobilematters.gg/brawl-stars/mastery-ranks-points-rewards-brawl-stars)) [S3].
- **Brawl Pass 2026:** see §5.4.

### 4.7 Brawler unlocks: Starr Road → Brawler Blast; Credits

- **Dec 2022:** boxes were removed and the **Starr Road** arrived: pick the next brawler and fill it with Credits. Frank Keienburg: "no more probabilities, no more random rewards" ([Deconstructor of Fun summary](https://www.deconstructoroffun.com/blog/was-brawl-stars-ditching-loot-a-good-move)) [S].
- **June 2025:** Rare and Super Rare brawlers became Trophy-Road-only, no longer for gems ([Release Notes June 2025](https://supercell.com/en/games/brawlstars/blog/game-updates/release-notes-june-2025/)) [S].
- **Sept 2026: Brawler Blast** replaced the Starr Road ([Release Notes Aug 2026](https://supercell.com/en/games/brawlstars/blog/release-notes/release-notes-august-2026/); [Brawlmetrics](https://brawlmetrics.gg/news/brawl-stars-brawler-blast); [TrophyCoach](https://trophycoach.com/brawl-stars/guides/brawl-stars-brawler-blast-guide)) [S]:
  - Credits fill the next rarity in a fixed order: **Rare 160, Super Rare 430, Epic 925, Mythic 1,900, Legendary 3,800, Ultra Legendary 5,500**.
  - The opening is a **shooting gallery** of up to 10 unowned brawlers of that rarity. You shoot targets and **the last one standing is unlocked**. Some sources say there is occasionally a choice [C].
  - Each target hit gives 0–4 **Gem Fragments**, which turn into a progression bonus for the new brawler, up to P11 plus extras [C on totals].
  - 1,500 free Credits were given as transition compensation.
- **Credits** come from the Brawl Pass, Trophy Road, Starr Drops and the Daily Freebie ([Mobile Matters currencies](https://mobilematters.gg/brawl-stars/currencies-overview)) [S3].
- **Rarities:** Starting, Rare, Super Rare, Epic, Mythic, Legendary, Ultra Legendary ([Supercell support](https://support.supercell.com/brawl-stars/en/articles/brawler-rarities-3.html)) [S]. Ultra Legendary started with Kaze, in early access May 2025 ([esports.net](https://www.esports.net/news/mobile-games/kaze-value-pack-brawl-stars/)) [S3].
- **[INF] The arc of this design:** random box (2017–22) → *chosen* path (2022–26) → *a random pick inside a fixed rarity, wrapped in a mini-game* (2026). Supercell keeps putting a little luck back into unlocks. For kids, the **chosen** version (Starr Road) is the one to copy.

---

## 5. Economy

### 5.1 Currencies and resources

| Resource | Sources | Sinks | Notes |
|---|---|---|---|
| **Coins** | Trophy Road, Pass, Starr Drops (Rare: 50 coins at 41.9%), Records, events, Mega Pig | Power levels; gadgets, SPs, gears, HCs; buffies (with PP) | **The bottleneck** [INF]. About 150 coins ≈ 20 gems in the shop [S3, weak] ([Mobile Matters](https://mobilematters.gg/brawl-stars/currencies-overview)) |
| **Power Points** | Starr Drops (Rare: 25 PP at 32.6%), Pass, Road, freebies (e.g., 75 PP) | Levels; buffies (2,000 per pull) | Gem rate 1 = 10 PP since Dec 2025 ([PP wiki](https://brawlstars.fandom.com/wiki/Power_Points)); an old wiki says 2 coins per PP in the shop [S3, dated] |
| **Credits** | Pass, Road, Starr Drops (Rare: 10 credits at 2.3%), freebie | Brawler Blast (160 → 5,500) | Collection currency |
| **Gems** (premium) | **Bought with money**; small free amounts from the Pass and Pro Pass | Skins (29–299), the Pass, offers, conversion to coins/PP | A 15-gem Mega Quest was removed from Nov 2026 ([Version History 2026](https://brawlstars.fandom.com/wiki/Version_History/2026)) [S3] |
| **Bling** | Pass, Road, quests, challenges, drops, Mega Pig | Cosmetics below Mythic, pins, sprays, icons; chroma buffies (10,000) | Cap of about 7,000 [S3, weak] ([Bling wiki](https://brawlstars.fandom.com/wiki/Bling)) |
| **Vault Keys** (Jan 2026 →) | **Only from Brawl Pass tiers**; they persist | 4 vaults: Resource / Brawler / Skin / Buffie. **Choose** the reward (e.g., Ultra Legendary Kaze = 6 Brawler Keys; Legendary Spike, Crow or Leon = 4) | Resource keys are on the free track, the others mainly paid ([Sportsdunia](https://www.sportsdunia.com/gaming/brawl-stars-how-to-get-keys)) [S3] |
| **Gem Fragments** | Brawler Blast targets | Bonus progression for the unlocked brawler | Sept 2026 [S] |
| **Pass XP / Pro XP / tickets / event currency** | Quests, ranked, esports viewing, Mega Pig tickets (6 per member) | Tiers | Event currency was added in update 63 (Sept 2025) ([PocketGamer.biz](https://www.pocketgamer.biz/march-2026-mobile-game-charts-brawl-stars-revenue-rises-nearly-50/)) [S3] |
| *Removed* | Chroma Credits, Club Coins, Tokens/Star Tokens, Brawl Boxes, Trophy Boxes | — | ([Mobile Matters](https://mobilematters.gg/brawl-stars/currencies-overview); [Supercell club post](https://supercell.com/en/games/brawlstars/blog/news/important-club-changes-are-coming/)) |

**[INF] Economy shape:**
- **About 8 live resources.** That is many, but each has one job.
- **Coins and PP are the progression throttles. Gems are the bypass.** Bling and Credits are the free-to-play cosmetic and collection lanes.
- **The free-to-play coin supply is deliberately scarcer than demand.** Every new layer (HC, gears, buffies) widens that gap, and that gap is what sells offers.

### 5.2 Shop

- **April 2026 redesign:** tabs (up to 7, empty tabs hidden, tabs shrink on scroll) and a tap-to-expand offer view ([Release Notes Apr 2026](https://supercell.com/en/games/brawlstars/blog/release-notes/release-notes-april-2026/)) [S].
- **Content:**
  - Daily rotating items and offers [BK].
  - Skin catalogue by rarity: **Rare 29 gems / 1,000 bling; Super Rare 79 / 2,750; Epic 149 / 5,000; Mythic 199; Legendary 299** (Mythic and Legendary are gems only) ([Mobile Matters skin rarities](https://mobilematters.gg/brawl-stars/brawl-stars-guide-to-skin-rarities)) [S3].
  - Hypercharge skins come with offers [BK].
  - **Value packs**, e.g., Kaze: maxed brawler + 18 pins, about $26 or $49.99 [C] ([esports.net](https://www.esports.net/news/mobile-games/kaze-value-pack-brawl-stars/); [Sportskeeda](https://www.sportskeeda.com/mobile-games/is-kaze-brawl-stars-worth-49-99)).
- **Daily Freebie** on the Supercell Store web shop (claim once per account, often 24 h). July 2026 "Freebie Week" gave 7 NanoDrops through logins ([Sportsdunia](https://www.sportsdunia.com/gaming/brawl-stars-freebie-week-july-2026)) [S3].

### 5.3 Starr Drops, Chaos Drops and Mega Boxes

**Starr Drop rules**
- Introduced June 2023 as a free, earned and not directly purchasable chance reward ([Naavik](https://naavik.co/deep-dives/brawl-stars-deconstruction/)) [S3].
- **Tapping gives a chance to raise the rarity, up to 4 upgrade chances.** The outcome is decided before the animation ([AFK Gaming](https://afkgaming.com/mobileesports/guide/brawl-stars-starr-drops-rewards-rarities-drop-chance-how-to-get-more); [Mobile Matters](https://mobilematters.gg/brawl-stars/brawl-stars-starr-drops-guide)) [S3].
- The number of wins matters, not the streak ([Mobile Matters](https://mobilematters.gg/brawl-stars/brawl-stars-starr-drops-guide)) [S3].

**Final rarity odds** (one calculator; unofficial): Rare 50% · Super Rare 28% · Epic 15% · Mythic 5% · Legendary 2% ([west-games](https://west-games.com/brawl-stars-drop-rate-calculator/)). Legendary at about 2% is the one figure all sources agree on ([timesaver Jul 2026](https://timesaver.gg/blog/brawl-stars-starr-drops-july-2026)) [S3/C].

**Contents**
- **Rare:** 50 coins (41.9%), 25 PP (32.6%), 10 Credits (2.3%), and others ([Sportskeeda](https://www.sportskeeda.com/esports/starr-drop-rates-brawl-stars-explained)) [S3].
- **Legendary:** Star Power 42.68%, Epic brawler 18.29%, Hypercharge 18.29%, skin 12.20%, Mythic brawler 6.10%, **Legendary brawler 2.44%**. A Legendary drop guarantees the rarity, not a Legendary brawler ([Sportsdunia via search](https://www.sportsdunia.com/gaming/brawl-stars-starr-drop-update-rare-skins-unattainable); [TopUpList](https://topuplist.com/blogs/detail/brawl-stars-legendary-starr-drops-brawler-chances)) [S3].
- **Feb 25, 2026:** rare skins were removed from Epic/Mythic Starr Drops and Chaos Drops ([hit_games note](../אפליקציית%20למידה%20משחקית%20לילדים/hit_games_engagement.md)) [S3].

**Chaos Drops** (Dec 2025)
- An upgraded drop with an "Ultra" ceiling (hypercharges, high-rarity brawlers, skins, buffies), worth "an average of 4× a standard Starr Drop".
- Sources: **the 6th Daily Win**, the Brawl Pass and events ([esports.gg](https://esports.gg/news/mobile/brawl-talk-buffies-chaos-drops/); [timesaver](https://timesaver.gg/blog/brawl-stars-starr-drops-july-2026)) [S3].

**Other drops and boxes**
- **Mega Box:** returned as an *earned* reward (pass, events, quests). The old version was 80 gems = 10 Brawl Boxes ([AllClash](https://www.allclash.com/brawl-stars-drop-rates-explained/); [esports.net](https://www.esports.net/news/mobile-games/mega-box-starr-drops-brawl-stars/)) [S3].
- **Themed drops:** Smoothie Drops, NanoDrops, Coffin Box (Brawl-O-Ween), Starr Nova Box, Damian Box ([Sportsdunia](https://www.sportsdunia.com/gaming/brawl-stars-starr-nova-box-drop-chances); [MMO Culture](https://mmoculture.com/news/brawl-stars-brawl-o-ween-2026-events/)) [S3].
- **[INF]** Every season re-skins the same reveal ritual. That is cheap novelty.

### 5.4 Brawl Pass (Jan 2026 rework)

- **60 tiers** (up from 50) and **53,250 XP** in total. Tiers 1–9 cost 50 → 350 XP, tiers 10–50 cost **950** each, tiers 51–60 cost **1,250** each ([timesaver Brawl Pass](https://timesaver.gg/blog/brawl-stars-brawl-pass)) [S3].
- **Free / Brawl Pass $8.99 / Plus $12.99** (old prices were $6.99 / $9.99). Plus adds +20% XP and 7–8 extra tiers [C] ([ldshop](https://www.ldshop.gg/blog/brawl-stars-gp/brawl-stars-brawl-pass.html); [esportsguides](https://esportsguides.co.uk/2026/02/10/brawl-stars-plus-pass/)) [S3].
- **Free track, Season 55:** 18 Starr Drops, 6 Chaos Drops, 1 Legendary Starr Drop and 2 Mega Boxes. Credits and skin keys sit mainly on paid tracks ([timesaver](https://timesaver.gg/blog/brawl-stars-brawl-pass)) [S3].
- **Season length:** about 28 days, though some run 35 ([seasontimer](https://seasontimer.live/blog/brawl-stars-all-seasons-dates)) [S3].
- Each season pays out in **Vault Keys** (§5.1). Keys persist after the season ([Sportsdunia](https://www.sportsdunia.com/gaming/brawl-stars-how-to-get-keys)) [S3].
- **[INF] Curve shape:** front-loaded cheap tiers (1–9) give quick early wins, then a long flat stretch with a steeper tail. We borrow this shape in §13.6 ⟦PROPOSED P7⟧.

### 5.5 Mega Pig (club)

- A monthly Friday–Monday event.
- Each member has tickets (6 per cycle reported). Best-of-1 wins add tickets to the pig, which has **5 stages × 16 wins = 80 wins**.
- Members who win at least 1 match claim the rewards. Members are **ranked by wins inside the club** ([Guild Order](https://guildorder.com/games/brawl_stars/wiki/mega-pig-club-event); [Mobile Matters](https://mobilematters.gg/brawl-stars/brawl-stars-mega-pig-event-explained)) [S3, weak].

### 5.6 Free versus paid, and kid-relevant facts

- **Free:** every brawler is reachable (Credits, Trophy Road, Brawler Blast), as are Starr Drops, Bling cosmetics, the free Pass track and the free Pro Pass track.
- **Paid only:** Mythic and Legendary skins, Pass and Plus extra tracks and keys, Pro Pass paid track, value packs, early access (Kaze), and *speed* (gems → coins/PP).
- **Age rules:** the ToS requires age 13+ (or parental consent), while the App Store rating is 9+. Accounts under 16 need a trusted adult's email for Supercell ID and have restricted chat and club features ([Supercell younger players](https://support.supercell.com/clash-of-clans/en/articles/younger-players.html); [Qustodio](https://www.qustodio.com/en/blog/brawl-stars-video-game-guide-parents/)) [S/S3].

---

## 6. Collection and customization

- **Roster:** about 104–105 brawlers by mid-2026 [C] ([1v9](https://1v9.gg/blog/how-many-brawlers-are-in-brawl-stars); [esports.gg Apr 2026](https://esports.gg/news/mobile/brawl-talk-april-2026-new-brawlers/)).
  - Release cadence is about 1–2 per season. Each Brawl Talk reveals the next 2–3 seasons' brawlers ([esports.gg Aug 2026](https://esports.gg/news/mobile/brawl-talk-aug-2026-brawler-unlock-more/)) [S3].
  - 2026 additions include Pierce and Glowbert (Dec 2025), Damian (Apr), Cosmo and Vince (Sept), Thunder Frogs (Legendary, next season) and Rey and Ruby (Dec; **pick one free during Brawlidays**) ([timesaver recap](https://timesaver.gg/blog/brawl-stars-brawl-talk-recap); [PiunikaWeb Damian](https://piunikaweb.com/2026/04/27/brawl-stars-community-annoyed-new-brawler-damian/)) [S3].
- **Skins:**
  - Tiers Rare → Legendary plus Hypercharge skins, with the prices in §5.2.
  - **Chromas** are colour variants of pass skins ([gamingonphone pass coverage](https://gamingonphone.com/news/brawl-stars-brawlentines-brawl-pass-brings-heartbroken-rico-and-brawlentines-angelo-skins-along-with-chromas-and-more)) [S3].
  - Ranked and Pro skins upgrade in steps ([Mobile Matters](https://mobilematters.gg/guides/brawl-stars/ranked-pro-pass-explainer)) [S3].
- **Expression items:**
  - **Pins:** emote stickers used in matches and chat; sold in sets (Kaze pack: 18 pins) ([esports.net](https://www.esports.net/news/mobile-games/kaze-value-pack-brawl-stars/)) [S3].
  - **Sprays:** spray on the ground in a match [BK].
  - **Profile icons and titles** (Records, Prestige golden/neon, Ranked frames, the 2026 Ranked frame designs) ([Version History 2026](https://brawlstars.fandom.com/wiki/Version_History/2026)) [S3].
- **Display:**
  - The Brawlers screen is a grid with **class sorting as the default (Nov 2026 for pick & ban) and trophy sorting**. Locked brawlers show as silhouettes [BK].
  - The brawler detail page shows power, abilities, skins, Records and Prestige.
  - The Brawler Blast section shows the fixed unlock order ([Release Notes Aug 2026](https://supercell.com/en/games/brawlstars/blog/release-notes/release-notes-august-2026/)) [S].
  - The player profile shows a battle card with the ranked icon and a legacy Prestige stat ([Threads post](https://www.threads.com/@brawlstars/post/DUJALjXjD95/new-trophy-system-were-applying-changes-to-the-current-trophy-system-after-the)) [S].
- **[INF]** Naavik calls "collecting the Brawler cast" **the biggest retention driver** ([Naavik](https://naavik.co/deep-dives/brawl-stars-deconstruction/)) [S3]. Characters, not currencies, keep people playing. That matches our rescued-friends loop (02 §5).

---

## 7. Social

- **Friends:** add by Supercell ID or QR code ([theglobalgaming](https://theglobalgaming.com/brawl-stars/how-to-1v1)) [S3].
- **Team up:** team codes, then queue together [BK].
- **Friendly Battles:** private lobbies with no trophies at stake, bots on or off, up to 6 players ([GamePressure](https://www.gamepressure.com/brawl-stars/friendly-game-custom-match/zdc63a)) [S3].
- **Clubs:**
  - Chat, with match replays shareable in chat, easier invites and club search. Club League, club quests and the club shop were removed and Club Coins converted.
  - **Mega Pig** is the only club event ([Supercell](https://supercell.com/en/games/brawlstars/blog/news/important-club-changes-are-coming/); [Mobile Matters club rework](https://mobilematters.gg/brawl-stars/brawl-stars-club-rework)) [S]. Naavik lists social systems as a key long-term retention driver ([Naavik](https://naavik.co/deep-dives/brawl-stars-deconstruction/)) [S3].
- **In-match communication:** **pins and quick chat** (no free text inside a match) [BK]. In Nov 2026, quick chat was added to pick and ban ([Release Notes Nov 2026](https://supercell.com/en/games/brawlstars/blog/game-updates/release-notes-november-2026/)) [S].
- **Spectating friends, replays, Star Player:** Star Player is the MVP of the winning 3v3 team. It was a +20% mastery bonus in the Mastery era ([Mobile Matters](https://mobilematters.gg/brawl-stars/mastery-ranks-points-rewards-brawl-stars)) [S3].
- **Community goals:** server-wide targets, e.g., 700 billion mastery points for a shared bonus ([Supercell RIP Masteries](https://supercell.com/en/games/brawlstars/blog/news/rip-masteries)) [S].
- **Co-op PvE:** boss fights, including 20-player ones ([GamesBeat](https://gamesbeat.com/brawl-stars-and-duolingos-mascots-face-off-in-new-collaboration/)) [S3].
- **[INF] How cooperation is encouraged:**
  - 3v3 objectives *require* role spread (tank, support, damage).
  - Duo Showdown revives create interdependence.
  - Mega Pig makes your wins fill a shared container.
  - The cost: Mega Pig ranks members and solo queue blames teammates, the second most common complaint ([Kimola](https://kimola.com/reports/unlock-brawl-stars-success-exclusive-feedback-analysis-report-app-store-us-150573)) [S3].

---

## 8. Live ops

| Cadence | What | Source |
|---|---|---|
| About 28 days | Brawl Pass season: theme, 2 pass skins plus chromas, lobby background and music, themed drops | [seasontimer](https://seasontimer.live/blog/brawl-stars-all-seasons-dates); [UI guide §1.1](supercell_ui_style_guide.md) [S3] |
| Monthly | Ranked season; club Mega Pig weekend | [timesaver](https://timesaver.gg/blog/brawl-stars-ranked-guide-july-2026) [S3] |
| About 4 months | Pro Pass, tied to esports (Brawl Cup, World Finals) | [Mobile Matters](https://mobilematters.gg/guides/brawl-stars/ranked-pro-pass-explainer) [S3] |
| About 2–3 months | **Brawl Talk** video (reveals the next 2–3 seasons) plus a large update; balance patches in between (e.g., Sept 16, 2026: 18 brawlers changed) | [esports.gg](https://esports.gg/news/mobile/brawl-talk-aug-2026-brawler-unlock-more/); [PlayNews](https://www.playnews.gg/en/guides/brawl-stars-the-meta-tier-list-after-the-september-16-maintenance-18-brawlers-adjusted) [S3] |
| Daily / rotating | Event slots by mode with map rotation. Exact slot timers weren't found this session; historically main 3v3 and Showdown maps rotate about daily, with special and weekend slots on 2–3-day cycles [BK] | — |
| Multi-week arcs | Phased events (Brawl-O-Ween "The Last Coffin" Oct 29 – Nov 5, 2026), multi-month events (Battle for Katana Kingdom praised in the 2025 review) | [timesaver](https://timesaver.gg/blog/brawl-stars-next-brawl-talk); [Mobile Matters roadmap](https://mobilematters.gg/news/brawl-stars/brawl-stars-roadmap-2026-details) [S3] |

- **2026 seasons:**
  - Steampunk (Jan) → Brawlentines (Feb 5) → Sands of Time (Mar) → Dragons & Faeries (Apr).
  - → **Starr Patrol = My Hero Academia** (May 7) → **Brawl Strikers** (football, World Cup; Jun 4).
  - → Royal Academy / Back 2 School with **Duolingo** (Sept 19–30: Duo as a 20-player boss, quiz questions mid-battle; Spike inside Duolingo lessons with a 7-day track).
  - → Brawl-O-Ween (Oct) → Season 56 (Nov 5) → Brawlidays (Dec).
  - Sources: [Sportsdunia](https://www.sportsdunia.com/gaming/brawl-stars-starr-patrol-brawl-pass-may-2026); [esports.gg](https://esports.gg/news/mobile/brawl-strikers-season-explained/); [PlayNews Duolingo](https://www.playnews.gg/en/guides/brawl-stars-the-duolingo-event-and-boss-fight-against-duo-12-rewards-until-september-30); [Duolingo guides](https://duolingoguides.com/duolingo-brawl-stars-collab/) [S3].
  - Other sponsor events include the adidas STARR CUP (Jun 18 – Jul 1).
- **Esports:**
  - BLAST runs the Brawl Stars Championship 2026 ([Supercell BSC 2026](https://supercell.com/en/games/brawlstars/blog/esports/first-look-at-bsc-2026)) [S].
  - Brawl Cup in Berlin, May 15–17 ([AppSpy](https://www.appspy.com/brawl-stars/brawl-cup-2026/)) [S3]. World Finals in Tokyo ([players.com.ua](https://players.com.ua/en/news/tokyo-to-host-the-brawl-stars-world-finals-2026/)) [S3].
  - Watching earns Pro Pass XP and sometimes freebies, e.g., a free Hypercharge link during World Finals ([Sportskeeda](https://sportskeeda.com/mobile-games/how-win-free-hypercharge-rewards-via-brawl-stars-world-finals)) [S3].
- **2026 organisation:** two separate teams, one for features and one for limited-time events, plus monthly updates trialled in H2 2026 and more narrative ([Mobile Matters roadmap](https://mobilematters.gg/news/brawl-stars/brawl-stars-roadmap-2026-details)) [S3].
- **Borrowed formats:** the 2023–24 turnaround borrowed event and offer formats from social casino and puzzle games ([PocketGamer.biz](https://www.pocketgamer.biz/game-analysis-how-supercell-sparked-a-brawl-stars-resurgence)) [S3].

---

## 9. UX flows

### 9.1 Screen map [BK + `supercell_ui_style_guide.md` §1.1 for positions]

```
LOBBY (center: selected brawler 3D; top-left: profile; top-right: currencies, friends/club, menu;
       left column: Shop · Brawlers · (Pass/Road/Records entry); bottom-right: MODE CARD + PLAY)
 ├─ PLAY (1 tap) → matchmaking (≈5–20 s; cancel) → loading → MATCH → RESULTS → DROP REVEAL(S) → LOBBY
 ├─ MODE CARD → Events screen (slots by mode, map, rotation timer) → pick → back to lobby
 ├─ BRAWLERS → grid (sort: class/trophies/power/rarity) → BRAWLER PAGE
 │      ├─ upgrade (confirm popup with cost) → level-up ceremony
 │      ├─ abilities (gadget/SP/gear/HC/buffy equip & buy) · skins (try-on, buy) · records · prestige
 │      └─ Brawler Blast section (unlock order) → gallery mini-game → unlock ceremony
 ├─ SHOP → tabs (≤7) → offer → expanded view → purchase popup (gems/bling/money)
 ├─ BRAWL PASS → tiers track (free/premium rows) → claim · Quests tab (Daily/Season/Event + timers) · Vaults (keys → choose)
 ├─ TROPHY ROAD → worlds → claim milestones (one tap each, or claim-all)
 ├─ RANKED (≥5,000 trophies) → pick & ban → match → rank bar · Pro Pass
 ├─ SOCIAL → friends · club (chat, Mega Pig) · team code · friendly battle
 ├─ PROFILE → battle card, icons, titles, stats, legacy prestige
 └─ CATALOG / COLLECTION → skins, pins, sprays, icons (bling/gem prices)
```

- **Navigation depth:** at most 3 levels for nearly everything (lobby → section → item → popup) [INF].
- **Taps to a match:** **1** (Play, same mode) or **3** (mode card → slot → Play) [BK].

### 9.2 Results and reward sequence [BK + INF; fits the Star Player sourcing above]

1. "VICTORY" or "DEFEAT" banner with the team lineup. The winners animate a pose; on a loss, the brawlers look sad [BK].
2. **Star Player** highlight (winners) [S3].
3. Per-brawler **trophy delta counts up** (+streak bonus, Underdog tag) [S3]; the Trophy Road bar advances.
4. Daily Wins slot fills, then the drop arrives → **tap-to-upgrade reveal** (§5.3).
5. Pass XP or quest ticks, if any.
6. Back in the lobby: the team stays together, "play again" goes to the same mode, and a red badge appears on any new claimables.

### 9.3 Reward-claim and badge logic [INF from observation patterns]

- **Claimables pile up as badges** (red count) on Pass, Road, Shop (free item), Brawlers (upgrade affordable) and Records.
- Claiming is **one tap per reward**, with an escalating reveal for drops. "Claim all" exists for bulk resources.
- **The badge rules work as nudges:** "an upgrade is affordable" badges push spending, and shop freebie badges pull players into the shop, where the offers are. This is where engagement shades into monetisation. Our 02 §6.3 rule 5 (a quiet number, no flashing red) is the right counter-rule.

---

## 10. Feedback and juice

Mostly **[BK]** (no primary Supercell craft source was reachable). Our implementation numbers are owned by 01 §16 and 09 §5–§6.

- **Hits:** a white flash on the target, floating damage numbers, a small knockback on some attacks, and HP bars that drain with a lag.
- **Kills:** a burst plus a skull icon and a kill-feed entry. Your own death gets a short greyed respawn timer.
- **Super ready:** the button pulses, plus a sound cue and a ring glow. **Hypercharge** adds a coloured aura and screen-edge tint on activation.
- **Objective moments:** a big countdown numeral in Gem Grab, plus "overtime" banners and a goal explosion.
- **Victory / defeat:** a fanfare or a deflating sting; the losing banner is short. **Count-ups** cover trophies, coins and tier XP. Rank-ups get a shield animation with a ribbon.
- **Starr Drop reveal:** a fixed-rarity star appears. Each tap shakes it harder with a rising pitch and may **change its colour up a tier** (green → blue → purple → red → gold, roughly), then a burst and the item card. **This is the most cited "dopamine" moment** ([ClutchPoints via UI guide](supercell_ui_style_guide.md)). For us, D2 forbids colour change at reveal; 02 §6.5 keeps everything except the change.
- **Audio:** an in-house composer and audio team, plus season-specific lobby music and agency-made event audio ([DevTrackers](https://devtrackers.gg/brawl-stars/p/6c2de745-i-m-not-even-joking); [Antfood](https://antfood.com/work/brawl-stars)) [S3]. Every brawler has voice barks for spawn, Super, kill and victory [BK].
- **Haptics:** light, optional, togglable [BK].
- **[INF] Underlying principle:** *every* state change has at least two of picture, sound and motion, and the most valuable moments (Super, drop, rank-up) have all three. Our C14 celebration budget keeps this rule but caps the total time.

---

## 11. What players love versus hate, and controversies

### 11.1 Love / hate

- **App Store rating:** 4.7 from about 2.1–2.9M reviews ([AppFollow US](https://apps.appfollow.io/ios/brawl-stars/1229016807?country=us); [App Store](https://apps.apple.com/us/app/-/id1229016807)) [S3].
- **Love:** fun, quick matches, characters, friends and community, an "addictive" loop ([Kimola](https://kimola.com/reports/unlock-brawl-stars-success-exclusive-feedback-analysis-report-app-store-us-150573)) [S3].
- **Hate:**
  - Pay-to-win, matchmaking, teammates and lag ([Kimola](https://kimola.com/reports/unlock-brawl-stars-success-exclusive-feedback-analysis-report-app-store-us-150573)) [S3].
  - Character acquisition ([Kimola TR](https://kimola.com/reports/unlock-insights-with-our-brawl-stars-feedback-analysis-report-app-store-tr-152905)) [S3].
  - Broken patches (April 28, 2026: login and shop failures) ([SSBCrack](https://news.ssbcrack.com/brawl-stars-players-face-glitch-in-oddities-store-after-maintenance-update/)) [S3].
  - Unhappiness with new brawler designs (Damian) ([PiunikaWeb](https://piunikaweb.com/2026/04/27/brawl-stars-community-annoyed-new-brawler-damian/)) [S3].

### 11.2 Controversy timeline and how Supercell responded

| When | What | Response / outcome |
|---|---|---|
| 2018 (beta) | Longest beta in Supercell history (522 days); portrait → landscape; "is it good enough?" doubts ([DoF 2018](https://deconstructoroffun.com/blog/2018/5/11/bs)) | Reworked twice, shipped Dec 2018 [S] |
| 2021–22 | KPIs fell through 2022 ([Naavik](https://naavik.co/digest/brawl-stars-moba-reimagined/)) | — |
| **Dec 12, 2022** | Brawl Boxes removed for "fair and predictable"; Starr Road added ([Game World Observer](https://gameworldobserver.com/2022/12/13/brawl-stars-loot-boxes-removed-supercell)) | Revenue −14% over 5 months; "removed the thrill"; "the idea wasn't wrong, the execution was" ([DoF](https://www.deconstructoroffun.com/blog/why-removing-loot-boxes-in-brawlstars-failed)) |
| **June 2023** | Free Starr Drops (luck is back) | Recovery: ×8.8 revenue, ×3.9 DAU (Jun 2023 – Feb 2024) ([mobilegamer.biz](https://mobilegamer.biz/supercell-explains-brawl-stars-big-comeback-from-an-all-time-low-to-8-8x-revenue/)); talked through openly at GDC 2024 ([GDC](https://schedule.gdconf.com/session/brawl-stars-learnings-from-the-removal-of-loot-boxes/902677)) [S] |
| 2023 → | Hypercharge: "I win button" fears; paid early offers | Interview reassurance ([PocketGamer](https://pocketgamer.com/brawl-stars/frank-keineburg-interview-hypercharge-update/)); free HC giveaways and events ([gamingonphone](https://gamingonphone.com/news/brawl-stars-hypercharge-unleashed-event-explained-heres-how-to-get-a-free-hypercharge)) [S/S3] |
| 2024 | Record year; Supercell revenue €2.8B / $3B, +77% ([mobilegamer.biz](https://mobilegamer.biz/squad-busters-disappoints-but-brawl-stars-soars-in-record-year-for-supercell/)) | — |
| Apr–May 2025 | "Play to Win" uncapped event raised P2W fears ([Sportskeeda](https://sportskeeda.com/mobile-games/is-brawl-stars-pay-to-win-2025)); **Ultra Legendary Kaze paid early access** ([DevTrackers thread](https://devtrackers.gg/brawl-stars/p/9b267d27-scummy-things-about-the-l-l-early-access-offer)) | Credits route a week later (Jun 5, 2025) ([esports.net](https://www.esports.net/news/mobile-games/kaze-value-pack-brawl-stars/)) [S3] |
| June 2025 | Masteries removed → Records; backlash | Public "we failed" post-mortem ([2025 in review](https://supercell.com/en/games/brawlstars/blog/community/2025-in-review-franks-blog-post)) [S] |
| 2025 | Revenue and DAU dipped ([PocketGamer.biz](https://www.pocketgamer.biz/march-2026-mobile-game-charts-brawl-stars-revenue-rises-nearly-50/)) | Update 63 events stabilised DAU [S3] |
| Jan 2026 | Pass price $6.99 → $8.99; 60 tiers; keys | Vault choice framed as player agency [S3] |
| Feb 2026 | Trophy resets removed | Well received [INF] |
| Sept 2026 | Starr Road → Brawler Blast (pick is random) | 1,500 free credits as compensation [S] |
| Nov 2026 | Ranked unlock raised to 5,000 and reset softened −6 → −2 | Players who already had Ranked keep it and get a Chaos Drop instead [S] |

**[INF] Supercell's response pattern:**
1. Explain in public (blogs, Brawl Talk, GDC).
2. Compensate (credits, drops, conversions such as Club Coins → coins/PP).
3. Grandfather access ("players who already unlocked Ranked keep it").
4. Simplify (remove systems).
5. Put the fun of luck back in a "free" form.

Points 1–4 are good models for our parent-facing change notes (`10` "what's new"). Point 5 we reject; we keep the ritual instead.

---

## 12. Ethics map: every mechanic → ADOPT / ADAPT / REJECT

**Key:**
- **ADOPT:** healthy as it is.
- **ADAPT:** keep the feel, change the rules.
- **REJECT:** manipulative, FOMO, random-paid or PvP loss.

**Spec references:**
- 01 = game design. 01 §12 is co-op; the brief said "07 co-op", but under 00 J1 co-op belongs to 01 §12 and 07 owns team-talk/SEL.
- 02 = economy. 09 = UX. 00 refers to the clause.

| # | Brawl Stars mechanic | Verdict | Our concrete version | Maps to |
|---|---|---|---|---|
| 1 | 2–3 min match, clear end | **ADAPT** | A unit for each age: a Kishōtenketsu beat (0:45–2:00) with a checkpoint and a rescue; a level K2 3–6 / K1 6–9 min; finite episode with wind-down, no autoplay | 01 §5.1, 00 C3 |
| 2 | 1-tap Play from lobby | **ADOPT** | The hub CTA starts the *recommended* node directly; the map is optional (⟦PROPOSED P1⟧) | 09 §8.3 F1 |
| 3 | Twin-stick, landscape | **ADAPT** | Tap-to-move (K2), optional floating joystick (K1 only); action buttons not mirrored in RTL | 01 §6.3, 09 §4.5, UI guide §10 |
| 4 | Tap = auto-aim, drag = precise | **ADOPT** | Tap the target in the world = the learning action; K1 can drag-aim the light beam for flair (same result) | 01 §9.1 |
| 5 | Super charged by hits | **ADAPT** | Correct answers fill the פנסון's core and a full core = "light wave" (K1 opt-in combo, proficient skills only). **The charge never decays and errors never drain it** | 01 §9.1, 00 C5 |
| 6 | Gadgets (limited uses) | **ADAPT** | Tools from TOOLBELT: unlimited uses, hints free forever, never sold | 07 §2.2, 02 §5.2 r7 |
| 7 | Hypercharge | **ADAPT** | Same as #5; no purchase, no power gap between children | 01 §9.1 |
| 8 | HP, death, respawn | **REJECT → ADAPT** | No HP; shadowlings have layers; a wrong beam lights a flower (funny) | 01 §9.1, §6.5 |
| 9 | PvP trophies won/lost | **REJECT** | No PvP, no loss, no sibling comparison | 00 C5, C6 |
| 10 | Trophy Road (long linear milestone track) | **ADAPT** | The Map/Journey path plus the Level 1–50 table with milestone gifts; **monotonic, earned by learning XP**; upcoming rewards shown as silhouettes | 02 §3, 09 SCR-04 |
| 11 | Per-brawler trophies, Prestige without a cap | **ADAPT** | Per-domain skill garden plus "stars" past Level 50 (uncapped, 7,000 XP each) | 02 §3.1, §4.2 |
| 12 | Power levels bought with coins + PP | **REJECT** (pay-to-progress) → **ADAPT** | Visible "getting stronger" tied to **mastery only**: pet evolution, plus ⟦PROPOSED P2⟧ hero-core glow tiers. Cosmetic power fantasy, no gameplay advantage | 02 §8.2, 01 §9.1 |
| 13 | Power Points as a throttle | **REJECT** | No throttle currency; learning is never gated by spendables | 02 §0.2 |
| 14 | Loadout choice (gadget/SP/gear) | **ADAPT** | K1 picks an attack *style* (archer/builder/singer) in Plan, with **no power difference**; equipping cosmetics | 01 §9.1 |
| 15 | Buffies Claw Machine (coins + PP grind) | **REJECT** mechanics, **ADAPT** ritual | Capsule machine: the next capsule is always visible, no duplicates, fixed price (K1 60 / K2 6) | 02 §5.7, 00 D2 |
| 16 | Starr Drop (random rarity, tap-upgrade) | **ADAPT** | Deterministic chest: tier from the trigger, shown from the first frame; taps add shake and pitch only; preview before opening | 02 §6.3, §6.5; 00 D2 |
| 17 | Chaos Drop on the **6th daily win** | **REJECT** | No daily-count rewards, no login rewards | 01 §5.5, 00 C5 |
| 18 | Daily Wins track (6/day, soft stop) | **ADAPT** | The **episode path of the day**: 4–5 "lanterns" (Plan → Morning → Level(s) → Camp → Evening). The last lantern = receipt + goodnight. **Finite per episode, no carry-over pressure, nothing for coming back daily** (⟦PROPOSED P3⟧, presentation only) | 01 §5.3, 09 |
| 19 | Win-streak bonus trophies | **ADAPT** | Pentatonic streak notes (up to 8); the chain resets silently; no reward is ever lost | 01 §16.1 |
| 20 | Underdog: no loss, catch-up road buff | **ADOPT principle** | Already stronger: nothing is ever lost; no catch-up needed because nothing expires (02 §9.3 overflow) | 02 §9.3 |
| 21 | Brawl Pass free/paid, 28-day expiry | **REJECT** paid/expiry → **ADAPT** | Adventure Album: free, 30/20 tiers, **never expires**, past seasons stay completable | 02 §9 |
| 22 | Vault Keys: *choose* your reward | **ADOPT** (choice) | Gold chest = pick 1 of 3 shown in advance; wishlist plus save-goal | 02 §6.3 r2, §7.4 |
| 23 | Quests with timers (Daily/Season/Event) | **REJECT** timers → **ADAPT** | K1's **Plan goal and prediction** ("I'll rescue 6/7") plus a Calibration Star for an accurate prediction; optional curiosity tasks that never expire | 01 §5.2 |
| 24 | Daily shop, offers, value packs, early access | **REJECT** | Fixed catalogue; weekly Sunday showcase; no discounts, countdowns or "expires" | 00 D3, 02 §7.3 |
| 25 | Gems bought with money | **REJECT** | Gems only at milestones (K1), weekly cap 10, never purchasable | 00 D1, 02 §2.5 |
| 26 | Gems → coins/PP conversion (pay to skip) | **REJECT** | No conversions except the declared overflow (gems above the cap → 20 coins each, shown openly) | 02 §2.5 |
| 27 | Bling (free cosmetic lane) | **ADOPT concept** | Coins buy only cosmetics, building and pet treats | 02 §7.1 |
| 28 | Skin rarity priced by gems | **ADAPT** | Price bands A–D; rarity = *what earned it* (chest tier), never chance | 02 §7.2, 00 D2 |
| 29 | Pins (emotes) | **ADOPT** | Emotes (unlock K1 Lv7 / K2 Lv8); in co-op they are the non-verbal channel | 01 §10.6, 02 §7.1 |
| 30 | Sprays | **ADOPT** | Placeable stickers in the neighborhood and camp; K3 sticker scenes | 02 §2.4, 01 §10 |
| 31 | Titles and icons | **ADAPT** | Titles describe **actions**, never traits ("100 friends came home") | 02 §5.4, 00 C4 |
| 32 | Brawler roster of about 105 with rarities | **ADAPT** | Rescued friends by species, plus hero parts; species sets → badge plus pet accessory; no power rarity | 02 §5, 01 §4.11 |
| 33 | Brawler Blast (random pick within rarity) | **ADAPT** | ⟦PROPOSED P4⟧ "Lantern gallery": in a gold-chest or capsule-series choice the child *knocks down* the options they don't want and **keeps the one they choose**. The gallery is the ritual and the choice is theirs | 02 §6.3, §5.7 |
| 34 | Records (achievements) | **ADOPT** | Skill badges named after the skill; per-tool states; collection-room walls | 02 §4.1, 07 TOOLBELT |
| 35 | Mastery (opaque formula, removed) | **LESSON** | One level vocabulary (C15); the child sees one scale per system | 00 C15 |
| 36 | Ranked plus a reset (−2 ranks) | **REJECT** | Opt-in Stretch Zone and K1 ghost time-trial against **her own** record only | 00 C2, 01 §5.3 |
| 37 | Pro Pass, esports viewing XP | **REJECT** | — | 00 C5 |
| 38 | Clubs and chat | **REJECT** open chat → **ADAPT** | The family camp is the "club"; messages only from chips (`coop_letter`) | 00 C7a, 01 §12.1 CO-4 |
| 39 | Mega Pig (shared container, tickets, in-club ranking) | **ADAPT** | Weekly family goal: logs fill, only the total is shown, `overshootAllowance 1`, **no per-child ranking**; nothing happens if it's missed | 02 §10 |
| 40 | Team codes, friendly battles | **ADAPT** | Same-device and family multi-device co-op (CO-1/2/3), drop-in/drop-out | 01 §12.1 |
| 41 | Star Player MVP | **REJECT** | Every role gets its own hero moment card; no MVP among siblings | 00 C6, 01 §12.2 |
| 42 | Quick chat | **ADOPT** | Chip-only phrases and emotes in co-op | 00 C7a |
| 43 | Spectate and replays | **ADAPT** | Show&Tell recording (≤30 s, on device) for the parent; parent "watch" in lap mode | 01 §5.2, MX-T05 |
| 44 | Brawl Talk reveal show | **ADAPT** | ניצי's "news" in camp: silhouettes of the next region and friends; **no countdown** | 01 §13 |
| 45 | 28-day themed seasons, collaborations | **ADAPT** | Calendar seasons of 5–9 weeks tied to the Hebrew year; original IP only (no licensed collabs) | 01 §13, 02 §9.1 |
| 46 | Limited-time events and modes | **ADAPT** | Holiday events stay completable forever ("holiday box"); no event currency; matched sink | 02 §10.3 |
| 47 | Map rotation timers | **REJECT** timers → **ADAPT** | Novelty through the daily surprise (max 1 per day, **accumulates**, nothing is lost) and one new gimmick per level | 01 §5.5, 00 G3 |
| 48 | Daily freebie on the web store | **REJECT** | No login rewards | 01 §5.5 |
| 49 | Push notifications and red badges | **REJECT** for kids | No child notifications; quiet count, no flashing red; badges only for *earned and unseen* items, never for offers | 02 §6.3 r5, §8.3 |
| 50 | Results count-ups, banners | **ADOPT** within budget | T2/T3 tiers; celebrations at most 15% of the episode | 00 C14, 01 §16.1 |
| 51 | DEFEAT screen | **REJECT** | No defeat screen; "almost!" ("כמעט!") with the same layout; the rescue cannot fail | 02 §5.2 r2, UI guide §6.4 |
| 52 | Seasonal lobby background and music | **ADOPT** | The hub re-dresses per season and holiday | 01 §13, 09 §6.5 |
| 53 | Brawler personality and voice barks | **ADOPT** | Cast with voice lines; the hero is silent (G4); approved humor guidelines | 01 §2.6, §4 |
| 54 | Undisclosed bots in low-trophy matches | **REJECT** deception → **ADAPT** | A scripted ניצוץ takes over a missing sibling's role, **clearly a game character** | 01 §12.1 |
| 55 | Comeback mechanics (gem drop) | **ADOPT** | In the co-op evening the fire only dims ("tomorrow we'll add logs"); nothing is lost | 02 §5.5 |
| 56 | Paid early access | **REJECT** | — | 00 D1 |
| 57 | Transparent change notes, compensation, grandfathering | **ADOPT** (parent-facing) | "What's new" to the parent; never remove an owned item; refunds return coins with a neutral note | 02 §6.3 r6, §7.3 |
| 58 | Removing systems players invested in (Masteries) | **LESSON** | Never remove earned progress; counters only go up | 00 C5, 02 §0.2 |
| 59 | Progressive feature unlock by trophies | **ADOPT** | Unlocks by level, one system at a time (shop Lv3, capsule Lv4, emotes Lv7…) | 02 §3.2 |
| 60 | Free maxed brawlers for ranked (equalising) | **ADOPT principle** | No child is ever under-powered: all co-op roles are equally strong (handicapping) | 01 §12.4 |

---

## 13. Blueprint for our game (derived from the teardown)

> Constraints: consistent with 00, 01 and 02. Every number below is from 02 unless marked ⟦PROPOSED⟧. Proposals list the spec section that would change.

### 13.1 Core loop ("the match" = a beat)

```
see a goal in the world (caged friend / shadowling / gate)   ← readable at a glance (§1.3)
   → ONE learning action = the game action (tap/aim/build/say) ← tap-default, drag-expert (§1.2)
   → T0 juice ≤0.8 s (light beam, layer peels, pentatonic note) ← 3 channels (§10)
   → core fills (never drains) → optional K1 "light wave"       ← Super/Hypercharge feel, no loss
   → beat ends 0:45–2:00 with a rescue + checkpoint             ← the 2–3 min "match" arc
```

Brawl's arc maps onto our beats: spawn → skirmish → climax → banner becomes **Ki → Shō → Ten → Ketsu**. The Ten twist is our "overtime". Ketsu always lands on success ("land the plane", 01 §5.3).

### 13.2 Session loop (one episode = Brawl's "Daily Wins", without the obligation)

```
Profile tap → HUB: "path of the day" (4–5 lanterns, all visible)          ⟦PROPOSED P3⟧
  L1 Plan (choose + predict)  → L2 Morning review (wake 3–8 friends)
  → L3 Level (beats ×4)       → [K1: L3b Block B]
  → L4 Camp: friends move in, receipt, build/shop/wishlist
  → L5 Evening (co-op natural point) → Goodnight + offline mission → app sleeps
```

- Each lit lantern = a T1 tick, like Brawl's Daily-Win slot filling.
- The last lantern gives the **receipt** (02 §2.8) and, when one was earned, the chest reveal (the "Starr Drop moment", deterministic).
- **Differences from Brawl:**
  - The path belongs to the *episode*, not the day. Unlit lanterns never carry over as "missed".
  - There is no reward for daily return.
  - The parent cap ends the day (00 C3).
- K1 at 15–20 min against Brawl's ~22-minute session (§2) shows that **our length is already genre-normal. Only the pull-back hooks are removed.**

### 13.3 Meta loops (Brawl structure → our structure)

| Brawl Stars | Ours (owner) | Clock |
|---|---|---|
| Trophy Road (100k, worlds) | Map/Journey + Level 1–50 + stars (02 §3) | Months |
| Brawler roster + Brawler Blast | Rescued friends by species + collection room + capsule series (02 §5) | Weeks |
| Power 1–11 + abilities | Pet evolution by mastery (02 §8.2) + ⟦P2⟧ hero-core tiers | Weeks |
| Gadgets / SP / HC "new verbs" | One gimmick per level (G3) + tools unlocked per region (07 TOOLBELT) | Every level |
| Brawl Pass (28 d, paid) | Adventure Album (5–9 weeks, free, never expires) (02 §9) | Season |
| Records | Skill badges + Calibration Stars (02 §4, 01 §5.2) | Ongoing |
| Club + Mega Pig | Family camp + weekly goal + campfire projects (02 §10, §7.6) | Week |
| Brawl Talk | ניצי's news (silhouettes, no timers) (01 §13) | Season |
| Prestige | Stars beyond 50, monotonic | Long tail |

### 13.4 First-week FTUE plan (Brawl's "something new almost every match", within 01 §14)

01 §14.2–14.3 already handles the first 60 s and the day-by-day rhythm. **The Brawl Stars lessons to layer on:**

1. **First action ≤15 s, first reward ≤45 s, first "win" guaranteed.** Brawl's practice match against bots maps to our single-target first rescue (09 F6, 01 §14.2). ✔ Already specified.
2. **One new system per session, each with a guided first use.** Brawl shows a badge, then a pointer, then a one-tap claim. Map onto 01 §14.3:

| Day | K1 new system (guided first use) | K2 new system | Brawl Stars analogue |
|---|---|---|---|
| Sun | Hero lab + first rescue + diary page | Hero lab + first rescue | Tutorial island |
| Mon | Collection room (3 *real* rescued friends) | Collection room | First Trophy Road claim |
| Tue | First Build Mission (arrays) | Movement moment + Build | First upgrade |
| Wed | First regular level + evening + **pet egg** | Same | First new brawler |
| Thu | First family evening boss + fire 1→2 + family treasure (**first chest reveal**) | Same | First Starr Drop |
| Fri | Daily surprise + shop/wishlist (if Lv3) | Daily surprise | Shop tab badge |

3. **⟦PROPOSED P5⟧ Show the road ahead.** Add a "**next 3 unlocks**" silhouette strip on the hub, like a Trophy Road preview: "פותחים בקרוב: 🛒 חנות · 🎁 מכונת קפסולות · 💃 ריקודים". Each unlock shows its condition as information ("ברמה 4"), never a countdown. *Changes 09 (hub skeleton SCR-03) and reads `levels.unlocks` from 02 §3.2; no economic change.*
4. **⟦GAP G-BS-1, flagged⟧ Week-1 XP drought.** Placement expeditions pay 0 XP (02 §2.7), and K1 does them Sun–Wed. So the 02 §3.3 "Level 3 at week 0.2" projection may not hold in week 1. The level-gated unlocks (shop at Lv3, capsule at Lv4) could slip to the end of week 1, exactly when Brawl Stars front-loads unlocks. **Two options for the 02 owner:**
   - **(a)** Let `episodeComplete` XP (50) and `rescue` XP (5) still pay in placement episodes. They are not measurement data; 02 §2.1 already counts rescues for the collection.
   - **(b)** Key week-1 unlocks to FTUE episode count, not XP.
   - **Recommendation: (a).** Simulate it in 02 §12.
5. **K1 "skip" for babyish moments** (01 §14.2) mirrors Brawl letting veterans skip tutorials. ✔ Already specified.

### 13.5 Progression tree (per child; all counters monotonic)

```
LEARNING ACTIONS (engine events)
 ├─► XP ─► Level 1–50 ─► Stars (∞)          ─► unlocks one system at a time (02 §3.2)
 │        └► Adventure Album tier (season)    ─► seasonal cosmetics, chests (02 §9)
 ├─► skill level (08) ─► badges (02 §4) ─► pet stage (02 §8.2) ─► ⟦P2⟧ hero-core tier
 ├─► rescues ─► neighborhood buildings (02 §5.5) ─► secret levels (§5.3) ─► collection milestones
 ├─► pieces  ─► regional mosaic ─► secretB + capsule series (02 §5.7)
 └─► milestones ─► chests (deterministic tier) ─► coins / gems(K1) / known item
                                  └► spend (choice): shop · building · pet treats · capsule · campfire logs
FAMILY: weekly goal ─► fire level (∞, never resets) ─► region early-unlock + family projects (02 §7.6, §10)
```

### 13.6 Economy sketch and proposed changes (02 owns every number)

**What stays exactly as in 02:**
- Coins and gems are pocket money, not throttles.
- Chests are deterministic.
- The shop is cosmetic and fixed, with a weekly showcase.
- No random rewards, no paid anything, no expiry.

The Brawl lesson that *supports* 02: Brawl's coin bottleneck sells offers, and we have nothing to sell, so **our coin supply can be generous and predictable**. 02's band A at "1–2 episodes" and band D at "1.5–3 weeks" (02 §7.2) is a healthy free-to-play-like wish horizon without scarcity pressure.

**Proposed changes** (each is small, deterministic and C5/D2-safe):

| ID | Proposal | Brawl Stars evidence | Spec change | Numbers |
|---|---|---|---|---|
| **P1** | Hub CTA → start the recommended node directly (≤2 taps from profile to play); map stays one tap away | 1-tap Play (§9.1) | 09 §8.3 F1 steps 3–4 | Taps: profile → CTA = 2 (now 3) |
| **P2** | **Hero-core glow tiers** keyed to *total skills mastered* across domains: a cosmetic beam and aura upgrade (the "getting stronger" fantasy, 00 A4 "heroes and battles") with **no gameplay effect** | Power levels as visible strength (§4.2) | 02 §4/§1 new non-spendable counter `heroCoreTier`; 01 §3.2 visuals; **check against** 01 §9.1 ("no difference in power") | Thresholds at 3 / 8 / 15 / 25 / 40 mastered (these reuse the pet thresholds 3/8/15) |
| **P3** | "Path of the day" lantern strip (presentation of the episode structure, §13.2) | Daily Wins slots (§2.1) | 09 hub/episode HUD; 01 §5.3 | No grants; each lantern ≤T1 (1.5 s) within the C14 budget |
| **P4** | "Lantern gallery" choice ritual for gold-chest 1-of-3 and new capsule series: knock down the ones you *don't* pick, keep your choice (K1; K2 a simple tap-to-pick) | Brawler Blast gallery (§4.7), Vault choice (§5.1) | 01 §16.2 choreography, 02 §6.3 r2 (choice unchanged) | ≤3 s extra; skippable after first view |
| **P5** | "Next 3 unlocks" silhouette strip | Trophy Road preview (§4.1) | 09 SCR-03 | — |
| **P6** | Weekly family goal shown in **3 visual stages** (logs ⅓ / ⅔ / full → fire grows a notch each stage); **still one treasure, only when met**, nothing if missed | Mega Pig 5 stages (§5.5) | 02 §10.2 display only | No new grants (keeps 02 §12 simulation intact) |
| **P7** | **Front-loaded Adventure Album tiers:** tiers 1–5 cost 50% so the first tier lands in the first regular episode; the difference is spread over the rest so the total is unchanged | Brawl Pass tiers 1–9 cheap (§5.4) | 02 §9.2 `tierXp` formula gets a `frontLoad` block | Example K1 S0 (30 × 250 = 7,500): tiers 1–5 = 125 (625 total); tiers 6–30 = 275 (6,875). K2 S0 (20 × 350 = 7,000): tiers 1–5 = 175 (875); tiers 6–20 ≈ 410 (6,125 ÷ 15 = 408.3 → round10 = 410; clamp [350, 650] OK) |
| **P8** | Week-1 XP from placement episodes (episode complete + rescues only) | Dense early unlocks (§3.2) | 02 §2.7 `placementProbe` exception list | +50 + 5×rescues per placement episode; still 0 per item |
| **P9** | Badge policy: a quiet numeric badge only for **earned and unseen** items (chest on the shelf, new friend in the collection room, an unlocked system); **never** for "affordable", shop or wishlist | Brawl's red affordability and offer badges (§9.3) | 09 §7.4 badge component rule; 02 §6.3 r5 | At most 1 badge per hub cluster |

### 13.7 What we deliberately leave behind, and why the kids will still love it [INF]

- Kids' reviews praise **quick matches, characters, friends and the reveal** (§11.1). We keep all four.
- They complain about **P2W, matchmaking, teammates and losing**. We remove all four by design: no PvP, no money, role-asymmetric sibling co-op (01 §12.2), no loss (C5).
- Brawl Stars' 2022–2023 dip shows that **removing the reveal thrill hurts**. We keep a full escalating reveal (02 §6.5: 2.9–5.8 s per tier, three channels) and drop only the colour-upgrade gamble.

---

## 14. Gaps and next steps

- **Re-verify primary sources once allowed:** `supercell.com` (Release Notes Jun 2025, Feb/Apr/Aug/Nov 2026; RIP Masteries; 2025 in review), `brawlstars.fandom.com` (Trophy Road milestone table, Daily Wins schedule, Starr Drop odds page), `naavik.co` (full deconstruction, especially any FTUE section).
- **Not found this session:**
  - Official Starr Drop and Mega Box odds tables (Supercell support "Starr Drop Chances").
  - Map rotation slot timers.
  - Quest counts and XP per quest.
  - Exact Daily Wins mix after Jan 2026.
  - Results-screen timings.
  - Any Supercell craft talk on hit feedback or audio.
- **Suggested in-house study** (a parent or designer with the game): record one FTUE and one Starr Drop reveal on an iPhone, then frame-step them to measure beat durations, matching `gap_audit.md` G14. Observation only, no extraction.
- **Hand-offs to the spec owners:**
  - P1, P3, P5, P9 → 09.
  - P2, P4 → 01 (+02 for the `heroCoreTier` counter).
  - P6, P7, P8 → 02, with the §12 simulation rerun.
  - GAP G-BS-1 → 02 §2.7 / 01 §14.3.
