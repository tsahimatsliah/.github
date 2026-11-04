# Supercell-Genre UI Style Guide for a Hebrew-First (RTL) Family Learning Game on iPhone Landscape (932×430 pt)

> Research note, written 2026-11-04. The goal is to extract principles, anatomy, proportions and techniques from Brawl Stars (primary), Clash Royale, Clash of Clans and Squad Busters, plus Stumble Guys, Subway Surfers, Rush Royale, Archero and Pokémon UNITE. A designer or agent can then build **original** screens in the same genre at the same quality bar. Nothing here asks you to copy proprietary art, fonts, icons or layouts pixel for pixel.
>
> **Network limits (important for trust).** WebFetch and curl could not reach gameuidatabase.com, interfaceingame.com, supercell.com, fandom, reddit, youtube, medium, artstation, behance, itch.io, samurai-gamers.com, fontsinuse.com, comicraft.com or ixd.prattsi.org. DNS/proxy blocked all of them, so those hosts would have to be allowed in the session's network settings. Facts from those sites come from **search-engine result summaries**, marked "(via search summary)". Only **GitHub (raw files and code search) and Google Fonts** could be read directly. I could **not view screenshots**, so I measured no Supercell screen. Every px/pt number for *our* screens is a design recommendation derived from the cited ratios plus genre conventions, and is labeled **[Inference]**. Two things are my own measurements, labeled **[Measured]**: font metrics and Hebrew-coverage checks, run locally with fontTools and Pillow+RAQM on font files downloaded from Google Fonts, and the CSS of an open-source Clash Royale-style button.
>
> **Ethics guardrails inherited from the plan** (see `gap_audit.md`, decisions D2, D7, D15, D16). Chests and reveals are **deterministic**, and "rarity" encodes *what earned it*, not chance. Per-answer feedback is ≤0.5–0.8 s and milestones are 4–6 s, skippable. Game screens on iPhone are landscape. Touch targets follow the age-corrected physical sizes in `gap_audit.md` G9: ≥60 pt for the 8.5-year-old, ≥90 pt for the 5.5-year-old and ≥120 pt for the toddler on iPhone. Those sizes override the 44 pt platform minimum quoted below.

---

## 0. TL;DR: the 12 rules that make it read as "Supercell-genre"

1. **One huge, warm-yellow primary CTA per screen** in a bottom corner. Everything else is quieter. Clash Royale reserves yellow for its most important buttons (Battle, Request, Use Card, Invite) — [The Rookies breakdown](https://discover.therookies.co/2020/02/24/game-design-ux-best-practices-detailed-breakdown-of-clash-royale) (via search summary). Brawl Stars' Play button is "big and bold in bright yellow" — [Pratt IxD critique](https://ixd.prattsi.org/2025/02/design-critique-brawl-stars/) (via search summary).
2. **A strict role-color code:** yellow = main action, green = secondary or "spend/upgrade", red = notifications, blue = backgrounds and low-importance UI — [Górnicki, UX in Clash Royale pt.1](https://www.gornicki.me/blog/90l/ux-in-clash-royale-part-1) (via search summary).
3. **Keep the center free for the hero character.** Group many actions at the edges by theme — [Medium/SWLH "Brawl Stars — What makes it great"](https://medium.com/swlh/brawl-stars-what-makes-it-great-3861dfb169fb) (via search summary).
4. **Stack every button in 4–5 layers:** dark outline, deep bottom lip, saturated face, light top band and a small specular "shine". Ratios are in §2 — [Dabolus/cr-button source](https://raw.githubusercontent.com/Dabolus/cr-button/master/cr-button.html) [Measured from source].
5. **All display text is heavy, rounded and outlined**, with a hard (0-blur) downward shadow. Ratios are in §3.
6. **Use one outline rule everywhere.** Outlines must stay the same weight across buttons, icons, panels and text at a given scale. Inconsistency is the #1 "clone" tell (§9).
7. **Popups over full screens:** keep the previous screen visible behind a dark scrim so the user stays oriented — [The Rookies](https://discover.therookies.co/2020/02/24/game-design-ux-best-practices-detailed-breakdown-of-clash-royale) (via search summary).
8. **Put key controls within thumb reach** — [The Rookies](https://discover.therookies.co/2020/02/24/game-design-ux-best-practices-detailed-breakdown-of-clash-royale) (via search summary). On 932×430 landscape, the bottom ~110 pt of each side is prime real estate [Inference].
9. **Consolidate buttons into hubs instead of adding buttons.** Supercell's Oct 2026 Clash of Clans UI revamp gives the reason: the old layout's clutter grew with each system and confused new players. It moved functions into a bottom-left Battle hub, a bottom-right Build hub and a top-left Profile hub — [Supercell: We've revamped the CoC UI](https://supercell.com/en/games/clashofclans/blog/news/weve-revamped-the-clash-of-clans-user-interface/) (via search summary).
10. **Minimal text and big numbers.** Icons carry meaning, numbers are the hero of counters, and labels are 1–2 words [Inference, consistent with Supercell's "readable at a glance" priority — see `ui_ux_polish_craft.md` §1].
11. **Juice every state change:** a press-squash, an overshoot on appear, count-ups, particles at reward moments — [Juice It or Lose It, GDC Europe 2012](https://gdcvault.com/play/1016789/Juice-It-or-Lose) (via search summary). Keep durations inside the D7 budget.
12. **For Hebrew:** mirror the *shell* (reading order) but **not** the in-game control scheme, clocks or media controls (§10). Set Hebrew labels about **15–20% larger** than the equivalent Latin all-caps label [Measured, §3].

---

## 1. Lobby / home screen composition

### 1.1 What Brawl Stars' lobby contains (cited) and how it is arranged

**Cited facts**
- The **center** shows the currently selected brawler with its rank, power and trophies — [Samurai Gamers main-menu guide](https://samurai-gamers.com/brawl-stars/main-menu-guide/) (via search summary; may predate recent updates).
- **Currencies plus chat and settings** sit at the **top-right** — [Samurai Gamers](https://samurai-gamers.com/brawl-stars/main-menu-guide/) (via search summary).
- A **left-side menu group** includes **Brawlers** (also where skins are viewed) and **Shop**. A Social menu covers Club and friends — [Samurai Gamers](https://samurai-gamers.com/?p=126011) (via search summary).
- The **profile name sits top-left** and opens the profile/icon screen — [Sportskeeda player-icon guide](https://www.sportskeeda.com/mobile-games/how-get-brawl-stars-player-icon-free-april-23-2025) (via search summary).
- The **Starr Road** (brawler unlock path) is "prominently featured on the main game screen just beneath 'Brawlers'" — [Wikipedia mirror](https://wiki.riteme.site/wiki/Brawl_stars) (via search summary).
- The **Play** button is the most prominent element: large, bold, bright yellow. Limited-time events get themed artwork and a themed UI on the home screen. The critique also warns that the many buttons can confuse new or returning players — [Pratt IxD critique, Feb 2025](https://ixd.prattsi.org/2025/02/design-critique-brawl-stars/) (via search summary).
- The menu uses "visual hierarchy and compartmentation to display quickly and intuitively the various actions, while leaving enough space in the centre" — [Medium/SWLH](https://medium.com/swlh/brawl-stars-what-makes-it-great-3861dfb169fb) (via search summary).
- Seasons get their own **main-menu background and music**, e.g. Brawl-O-Ween — [TheSixthAxis](https://www.thesixthaxis.com/?p=367498) (via search summary). Lunar New Year brought a "UI redecoration" and a new palette — [Dot Esports](https://dotesports.com/mobile/news/lunar-new-year-begins-on-brawl-stars-with-free-gifts) (via search summary).
- The Brawl Pass UI separates quests into daily, season and event categories with a counter — [Supercell (ES) update post](https://supercell.com/en/games/brawlstars/es/blog/game-updates/ya-disponible) (via search summary). In 2026 the pass grew from 50 to 60 tiers, and the Plus track front-loads rewards in tiers 0–7 — [Timesaver](https://timesaver.gg/blog/brawl-stars-brawl-pass) (via search summary; secondary).
- The brand itself was simplified in Sept 2025. The reasons given were "aged design, complexity, weak adaptability, inconsistency between the app and game logo, lack of emphasis on 'Brawl'", while the skull, colors and font style were kept — [Supercell blog: logo is changing](https://supercell.com/en/games/brawlstars/blog/news/brawl-stars-logo-is-changing/) (via search summary). Lesson: Supercell keeps *simplifying for adaptability*.

**Typical arrangement (LTR original) [Inference: from genre familiarity; verify against a current screenshot before relying on exact positions]**
- **Top-left:** a profile chip (avatar, name, total trophies) with a progression strip just below it (Trophy/Starr Road or Pass).
- **Top-right:** currency pills, then a menu/settings button in the corner. Social and chat entry points sit near the left edge.
- **Left column:** a vertical stack of square nav tiles (Shop, Brawlers, and others) with an icon over a short label. Each has a red count badge.
- **Center:** the 3D brawler on a stage/pedestal, slightly right of center, with the name and power level near it. When teammates join, their characters appear beside it.
- **Bottom-right:** a **mode/event selector card** (mode icon, map name, a timer if it is limited) placed **immediately to the left of the Play button**, so the "what" and the "go" form one cluster under the right thumb.

**Comparable 2025–2026 Supercell lobbies (cited)**
- **Clash Royale (Summer 2025):** a tab redesign. The Battle button now routes into Trophy Road and Challenges, with a **Game Mode Switcher** beside it — [Sportskeeda](https://www.sportskeeda.com/mobile-games/clash-royale-summer-update-2025-game-mode-switcher-trophy-road-rework) (via search summary). Chests were removed in 2025 — [RoyaleAPI](https://royaleapi.com/blog/rip-chests-2025-q1-update) (via search summary).
- **Clash of Clans (Oct 2026):** **Battle bottom-left, Build bottom-right**, given equal weight. **Profile top-left** absorbs Settings, Inbox and Supercell ID. Build has 4 tabs. "Almost nothing was removed" and there is no switching back — [Supercell CoC UI revamp post](https://supercell.com/en/games/clashofclans/blog/news/weve-revamped-the-clash-of-clans-user-interface/) (via search summary); [trophycoach guide](https://trophycoach.com/clash-of-clans/guides/clash-of-clans-new-ui-2026-where-did-my-buttons-go) (via search summary).
- **Stumble Guys (June 2026 revamp):** "refreshed button graphics, a new dynamic background, and a cleaner visual hierarchy", with equipped abilities shown on the main menu — [Stumble Guys news](https://www.stumbleguys.com/news/upcoming-changes-coming-to-ability-progression-and-main-menu) (via search summary). The home gives quick access to Shop, Customize, Friends, Clubs, Missions, Party and Play — [Stumble Guys help center](https://stumbleguys.helpshift.com/hc/en/4-stumble-guys/faq/66-what-can-i-find-in-the-home-screen/) (via search summary).
- **Pokémon UNITE:** uses a **pre-set phrase roster instead of free chat, "like in Clash Royale"** — [Telangana Today](https://telanganatoday.com/pokemon-unite-an-entirely-new-genre) (via search summary). This is the right pattern for a family app: no free text chat.
- **Subway Surfers:** an App Store reviewer complained that "the many events make it impossible to see the menu correctly" — [App Store (HU)](https://apps.apple.com/hu/app/subway-surfers/id512939461) (via search summary). This is a cautionary tale about event-banner clutter.

### 1.2 Our lobby: RTL wireframe at 932×430 pt [Inference: recommended spec]

Screen geometry: iPhone 14 Pro Max, 15 Plus and 15 Pro Max share 430×932 pt. **Landscape safe-area insets are left 59, right 59, bottom 21, top 0** — [useyourloaf: iPhone 15 screen sizes](https://useyourloaf.com/blog/iphone-15-screen-sizes/) (via search summary). That leaves a usable **814×409 pt** area (x 59→873, y 0→409). Add a **10 pt inner gutter**, which gives a content box of **x 69→863, y 10→399** [Inference]. Backgrounds, the character and the stage **bleed to all four screen edges**. Only interactive UI and text respect the safe area.

```
x: 0   59 69                         466(center)                       863 873 932
   ┌───┬──────────────────────────────────────────────────────────────────────┬───┐ y=0
   │   │ [≡][⚙]  [💎 1,250 +][🪙 8,430 +]          (season banner)  [שם ★12 (◉)]│   │ 10–56  top bar (h 46)
   │ D │                                                      [▓▓▓▓▓░░ דרך הלמידה ⭐]│ D │ 62–84  progress strip under profile
   │ y │ ┌───────────────┐                                         ┌──────┐     │ y │
   │ n │ │ משימה יומית ✓ │                  ✦  hero name ribbon     │ חנות │ ●3  │ n │ 96–160 right nav rail
   │ . │ │  2/3  ▓▓▓░    │                 ╭───────────╮           ├──────┤     │ . │
   │   │ └───────────────┘                 │  HERO 3D  │           │גיבורים│ ●1  │   │ 168–232
   │ I │ ┌───────────────┐                 │  ~230 pt  │           ├──────┤     │ I │
   │ s │ │  event card    │                 │   tall    │           │ אלבום│     │ s │ 240–304
   │ l │ └───────────────┘                 ╰───────────╯           └──────┘     │ l │
   │   │ ┌─────────────────────┐┌──────────────┐  ◖══ pedestal ══◗            │   │
   │   │ │      ! שַׂחֵק       ││ 🌋 עולם המספרים│                     [👨‍👩‍👧 משפחה]│   │ 319–395 bottom row
   │   │ └─────────────────────┘└──────────────┘                              │   │
   └───┴──────────────────────────────────────────────────────────────────────┴───┘ y=409 (home indicator below)
```

| Element | Position (x,y,w,h in pt) | Notes [Inference] |
|---|---|---|
| Profile chip | 663,10,200,46 | Avatar 46×46 at the **right** end (RTL start), with a level shield of 24 pt overlapping its lower-left corner. Name in Rubik 700, 16 pt. A sub-line with a star/trophy count in 14 pt. Tap opens Profile, which also hosts Settings and Inbox, following the CoC 2026 hub idea. |
| Progress strip (learning road / pass) | 663,62,200,22 | Track is 14 pt tall inside a 2 pt dark outline. A 32 pt star/chest icon overlaps the **left** end (the bar fills right→left in RTL, §10). |
| Menu + parent gate | 69,10,44,44 and 117,10,44,44 | Gear and parent-lock. These are secondary buttons, so blue/gray. |
| Currency pills | 169,16,118,34 and 295,16,118,34 | Icon 40×40 overlapping the pill's **right** edge by about 12 pt, value right-aligned next to it, and a small green "+" 26×26 at the left end (parent-gated). **Max 2 currencies.** |
| Right nav rail | 799,96,64,64 (+72 step) | 3 tiles: Shop, Heroes, Album/Map. Icon 40 pt over a label of 12–13 pt. Red badge 22 pt at the tile's top-left (outer) corner. |
| Daily mission card | 69,70,180,90 | Title, a progress bar and a "claim" state that turns the card into a green button. |
| Event/season card | 69,168,180,110 | Themed art with a timer pill. **Max 1–2 cards**, to avoid Subway-Surfers clutter. |
| Hero stage | center x≈560, feet y≈340 | Character about **230 pt tall (≈53% of screen height)**. Pedestal ellipse 220×44. Name ribbon 160×28 above the head or under the pedestal. Tap: hero plays a reaction. Long-press: hero detail. |
| **PLAY (primary CTA)** | **69,319,220,76** | Yellow. Label 34–38 pt Hebrew. **RTL reading path ends bottom-left.** The thumb case for bottom-left is that CoC 2026 puts a primary action (Battle) there. See the open decision in §10. |
| World/mode selector | 297,323,170,72 | Blue card: world icon 52 pt, world name 18 pt, a "change" chevron that points **left** in RTL. It sits right next to Play, as Brawl Stars pairs the mode card with Play. |
| Family/co-op button | 799,331,64,64 | Replaces chat/club. No free chat (UNITE/Clash pattern). |

**Hierarchy budget [Inference]:** 1 yellow element (Play) and at most 1 green element (a claimable reward) at any moment. Red appears only as badges. Everything else is blue, white or neutral. Squint test: at 25% scale, the first three things you see must be the **hero**, **Play** and **the profile/progress**.

### 1.3 Staging the character (pedestal, light, background) [Inference: genre craft]
- **Pedestal:** a low, wide ellipse (width ≈ 2.2× the character's shoulder width) with a 4–6 pt darker rim and a soft contact shadow (40–55% black, blur ≈ 12 pt) directly under the feet. Add a subtle rim glow in the seasonal accent color.
- **Lighting:** a warm key light from the upper side, a cool fill from the other side, and a **strong rim/back light** so the silhouette pops against a busy background. Use a slightly low camera (about 10–15° below eye level) so the hero feels heroic.
- **Background:** a 3-plane parallax: (1) blurred seasonal diorama, (2) a mid-plane prop set, (3) foreground stage. Add a **radial gradient** brightest behind the hero (center 100% → edges 55–65% luminance), a soft vignette (edges darkened 25–35%), and optional slow-rotating light rays behind the hero (period 20–30 s, 8–12% opacity). Seasonal re-skins swap only planes 1–2 and the accent color, following the Brawl Stars precedent of seasonal menu backgrounds and music cited in §1.1.
- **Idle life:** the hero breathes (scale-Y 1.00↔1.02, 2.4 s loop), blinks every 3–6 s at random, and plays a "fidget" every 8–15 s. Tapping triggers a reaction plus a voice line. Ambient particles in the background (dust motes, fireflies) at ≤20 on screen.

---

## 2. Button anatomy

### 2.1 Measured reference: an open-source Clash Royale-style button (community recreation, not Supercell)
This comes from `cr-button` (Polymer 2), "A Clash Royale styled button" — [GitHub Dabolus/cr-button](https://github.com/Dabolus/cr-button); [raw source](https://raw.githubusercontent.com/Dabolus/cr-button/master/cr-button.html). These are the CSS values read directly from the source [Measured]:

| Layer (outer→inner) | Value (default/yellow variant) | Ratio vs 72 px height |
|---|---|---|
| L1 outer shell | `#000` fill, radius 11 px, height 72; drop `0 11px 2px #301c09` (very dark brown) | radius ≈ 15%; outer drop ≈ 15% |
| L2 base/lip | `#FDA80D` inset 3 px on the sides and 2 px on top; **bottom lip** `0 9px 0 #af6b06` | dark border ≈ 3 px; lip ≈ **12.5%** |
| L3 top inner glow | `box-shadow: 0 -4px 5px #ffdc33` | light bleed at the top edge |
| L4 face | `#ffbb2a`, inner bottom shading `0 4px 3px #fe8906` | face slightly lighter than the base |
| L5 top highlight band | `#ffd053`, **height 45%** of the face, inset 3–4 px | highlight = **top 45%** |
| Shine | 7×9 px rounded blob `#fff1b6`, top-right corner | specular "sticker" glint |
| Label | 24 px "Supercell Magic", color `#ffc` (cream); outline faked with text-shadows at ±1 px plus a **3 px downward** offset in `#582e00` | label ≈ **33%** of height; drop ≈ **12.5%** of font size |
| Pressed | `transform: scale(.95)`, transition **75 ms** | |
| Variants | success: base `#09cc48`, lip `#0d8f23`, face `#30ed45`, band `#50ff65` · info: `#2199ff` / `#004fa5` / `#4cadff` / `#67bcff` · error: `#fc3632` / `#9c1814` / `#fc4262` / `#f66` · disabled: `#858585` / `#5a5a5a` / `#949494` / `#a9a9a9` | |

Takeaways [Inference]: (a) every color role has **4 tones**: lip (darkest, about −35% L), base, face, highlight (+15–20% L). (b) The lip and the outer drop together give about 25% of the height as "thickness", which is what makes it feel like a physical, pressable slab. (c) Label color is **cream or white** with a dark outline in the button's own dark hue (brown on yellow, dark green on green, navy on blue), never pure black.

### 2.2 Our button spec (tokens) [Inference: derived from 2.1 plus genre conventions]

**Geometry (iPhone, pt)**

| Size | W×H | Radius | Outline | Lip | Label (Latin / Hebrew) | Use |
|---|---|---|---|---|---|---|
| XL CTA | 220×76 | 18 | 3 | 8 | 30 / 36 | Play, Continue on results |
| L | 180×64 | 16 | 3 | 7 | 24 / 28 | popup confirm, claim |
| M | 140×52 | 14 | 2.5 | 6 | 20 / 24 | secondary actions |
| Icon tile | 64×64 | 14 | 2.5 | 6 | label 12 / 13 | nav rail |
| Round close (X) | 44×44 visual (hit 60) | 50% | 2.5 | 4 | — | popups |

- Minimum hit area: Apple's recommended 44×44 pt — [secondary summary of Apple HIG](https://smashingmagazine.com/2012/02/finger-friendly-design-ideal-mobile-touchscreen-target-sizes/) (via search summary). Material's 48×48 dp and WCAG 2.2's 24×24 CSS px are lower floors (same search). **Our floor is higher, per child age (G9 in `gap_audit.md`).** Visual size can be smaller than the hit box, so extend hit areas by an invisible 8–12 pt margin.
- **Layer stack (top to bottom in paint order):** (1) drop shadow `0 4 0` at 35% black, no blur. (2) Outer outline 3 pt, color = role-dark at about L 15–20%. (3) Lip: role-dark at about L 35%, offset down 8 pt. (4) Face: a vertical gradient from `role-light` (top) to `role` (bottom), stops 0%→100%. (5) Top highlight band: top 42–45% of the face, white at 25–35% opacity, inset 4 pt, same corner radius minus 2. (6) Shine: a 10×7 pt white ellipse at 80% opacity in the top-**start** corner (top-right in RTL). (7) Inner bottom shade: 3 pt, role-dark at 30%. (8) Label plus an optional icon.
- **Icon + label:** the icon is 60–70% of the button height and may **break the top edge by 15–25%** for dynamism. Gap to label = 6 pt. In RTL the icon sits on the **right** (start).
- **Notification badge:** a red circle of 22 pt (or a 22 pt-high pill for 2+ digits), with a 2 pt white or dark outline and a 13 pt white number. It sits on the button's top-outer corner, overlapping by 30–40%. On appear, pop 0→1.2→1.0 in 250 ms, then a gentle pulse every 4 s. Red is used **only** for badges, alerts and destructive actions, following the Clash role code in §0.

**Role palette (button 4-tone sets) [Inference, seeded from 2.1]**

| Role | Lip | Base | Face top | Highlight | Label outline |
|---|---|---|---|---|---|
| Primary (yellow) | `#B86A00` | `#FFB000` | `#FFD23A` | `#FFF1A8` | `#5A2E00` |
| Confirm/claim (green) | `#13852B` | `#2BCB48` | `#5EF06B` | `#B9FFB5` | `#0B4A14` |
| Secondary/info (blue) | `#0B4FA8` | `#2A8BFF` | `#59B0FF` | `#BFE3FF` | `#0A2D66` |
| Destructive/close (red) | `#9C1B18` | `#F23B36` | `#FF6A60` | `#FFC2BC` | `#5C0A08` |
| Special (purple) | `#5B21A6` | `#8E44FF` | `#B07BFF` | `#E2CCFF` | `#2E0F5C` |
| Disabled | `#5A5A5A` | `#8A8A8A` | `#A3A3A3` | `#D6D6D6` | `#2C2C2C` (label at 70%) |

**States [Inference; durations seeded from cited ranges]**
- **Pressed:** scale to 0.92–0.95, the face moves **down by the lip height** (lip collapses to 2 pt), and the face darkens 6%. Press-in takes 60–80 ms ease-out. Release springs to 1.0 with a 5–15% overshoot over 150–220 ms (ease-out-back). Cited ranges: 0.92 scale / 150 ms ease-out-back ("Duolingo-style") — [nylo ButtonAnimationStyle](https://pub.dev/documentation/nylo_support/latest/helpers_src_button_animation_style/ButtonAnimationStyle-class.html) (via search summary); 5–15% overshoot — [bugnet](https://bugnet.io/blog/how-to-fix-ui-button-pop-easing-feels-stiff) (via search summary); press feedback about 100–160 ms — [instagit duration budgets](https://instagit.com/emilkowalski/skills/what-are-recommended-duration-budgets-for-ui-animations.md) (via search summary); the cr-button uses .95 at 75 ms (2.1). Add a light haptic on press-down and a "thock" sound on release.
- **Disabled:** gray set, no shine, label at 70%, no press animation. A tap gives a short "nope" wiggle (±4 pt, 3 cycles, 240 ms) plus a tooltip saying *why*. Lesson from Clash Royale: removing the red "not enough" text made players tap items they couldn't afford — [The Rookies](https://discover.therookies.co/2020/02/24/game-design-ux-best-practices-detailed-breakdown-of-clash-royale) (via search summary). So show the unaffordable price **in red** on the button itself.
- **Attention (idle CTA):** every 3–5 s, a diagonal shine sweep (a white band at 40% opacity, 30° skew, 500 ms) plus a 1.00→1.04→1.00 breathe. Clash key controls get "an extra effect such as an animated highlight" — [Górnicki pt.1](https://www.gornicki.me/blog/90l/ux-in-clash-royale-part-1) (via search summary).

---

## 3. Typography

### 3.1 What Supercell actually uses (cited)
- **Clash of Clans** used Comicraft's commercial **You Blockhead** "to define the branding and game graphics … since its inception". In 2017 Comicraft made a custom "mideval" version for Clash Royale — [Comicraft](https://comicraft.com/?p=1082) (via search summary). Supercell's Latin in-game title font is **Supercell-Magic**, a custom You Blockhead — [fontsinuse: Clash Royale](https://www.fontsinuse.com/uses/66357/clash-royale-mobile-game) (via search summary).
- Body/message text in Clash Royale is **"SC CCBackBeat"**, a Comicraft **Back Beat** variant, according to a fan component library — [Dabolus/cr-fonts README](https://raw.githubusercontent.com/Dabolus/cr-fonts/master/README.md) [Measured from source]; [Back Beat on MyFonts](https://www.myfonts.com/fonts/comicraft/back-beat) (via search summary).
- That library's "title" mixin: Supercell Magic 16 px, white, outline built from 1 px text-shadows on all sides plus a **2 px downward** offset in black. Its "message" mixin: CCBackBeat 14 px white with no outline — [cr-fonts.html](https://raw.githubusercontent.com/Dabolus/cr-fonts/master/cr-fonts.html) [Measured from source]. The cr-card canvas draws card titles at 70 px Supercell Magic with `shadowOffsetY = 5` (≈7%) — [cr-card source](https://raw.githubusercontent.com/Dabolus/cr-card/master/cr-card.html) [Measured].
- **Brawl Stars' in-game font is not documented publicly.** Fan sites trace the *logo* lettering to the free font "Nougat" — [fontmeme](https://fontmeme.com/brawl-stars-font/) (via search summary). Community comments call **Lilita One** the closest free look-alike for the UI — [itch.io comment](https://itch.io/post/4398951) (via search summary). Treat both as "looks like", not "is".

### 3.2 Free Latin approximations [Measured + cited]
- **Lilita One** (Juan Montoreano, OFL, Google Fonts since 2012-01-11): **single weight 400**, Latin + Latin-ext, **0 of 27 Hebrew letters** — [google/fonts METADATA](https://raw.githubusercontent.com/google/fonts/main/ofl/lilitaone/METADATA.pb) [Measured with fontTools]. Ink density in a "PLAY" bounding box = **0.583**, the heaviest of all fonts tested [Measured].
- Other free heavy-rounded Latin options on Google Fonts: Baloo 2 (400–800), Madimi One, Bagel Fat One, Tilt Warp. None of them has a Hebrew subset — [google/fonts METADATA files](https://github.com/google/fonts/tree/main/ofl) [Measured]. **Recommendation: don't mix in a Latin display font at all.** Use the Hebrew font's own Latin glyphs and digits, so one outline/weight system covers everything (see 3.4).

### 3.3 Hebrew display candidates (all OFL on Google Fonts) [Measured]
Method: TTFs downloaded from fonts.googleapis.com. Hebrew letter coverage (U+05D0–05EA), core nikud coverage and the GPOS `mark` feature were checked with fontTools. "Ink density" = filled pixels ÷ bounding box for the word "שחק" at 100 px (higher = heavier). Glyph heights are in em.

| Font (weight) | Designer (METADATA) | Hebrew 27/27 | Core nikud | Ink density | Hebrew body height / Latin cap height | Character |
|---|---|---|---|---|---|---|
| **Rubik 900** (variable 300–900) | Hubert & Fischer, **Meir Sadan**, … | ✓ | all but rafe; GPOS mark ✓ | **0.564** | ש 0.581 / H 0.700 | Slightly rounded corners, compact, the punchiest. **Closest to "Supercell chunky".** |
| **Fredoka 700** (wght 300–700, wdth 75–125) | Milena Brandão, **Hafontia** | ✓ | complete; GPOS mark ✓ | 0.540 | ש 0.606 / H 0.699 | Fully round terminals, friendly, **closest to Lilita's feel**. Max weight 700, so add outline weight to compensate. |
| M PLUS Rounded 1c 900 | Coji Morishita, M+ | ✓ | missing qamats qatan and holam-haser-for-vav | 0.467 | ש 0.52 / H 0.73 | Rounded but light-looking Hebrew. 3.6 MB file (CJK). |
| Heebo 900 | Oded Ezer | ✓ | complete | 0.488 | — | Not rounded (Roboto-based). Good for body text, not display. |
| Secular One | Michal Sahar | ✓ | all but rafe | 0.448 | ש 0.554 / H 0.666 | Single weight, squarer. |
| Playpen Sans Hebrew 800 | TypeTogether | ✓ | — | 0.401 | — | Handwriting-ish, playful. Good for character speech bubbles. |
| Varela Round 400 | Joe Prince (Hebrew: Cornfeld/AlefAlefAlef) | ✓ | — | 0.280 | — | Too light for display. Body text only. |
| Gveret Levin | AlefAlefAlef (added 2025-11-14) | ✓ | — | — | — | A **handwriting** script — [GitHub AlefAlefAlef/gveret-levin](https://github.com/AlefAlefAlef/gveret-levin). Use for "notes from a character", never for buttons. |

Sources: METADATA for each family in [google/fonts](https://github.com/google/fonts) (e.g. [rubik](https://raw.githubusercontent.com/google/fonts/main/ofl/rubik/METADATA.pb), [fredoka](https://raw.githubusercontent.com/google/fonts/main/ofl/fredoka/METADATA.pb), [gveretlevin](https://raw.githubusercontent.com/google/fonts/main/ofl/gveretlevin/METADATA.pb)) [Measured]. Fredoka's Hebrew came from Hafontia, Ben Nathan's foundry, and later became the variable "Fredoka" — [Google Fonts discussion](https://groups.google.com/g/googlefonts-discuss/c/k_uu5mlBLpg) (via search summary). AlefAlefAlef's "Yiddishkeit" is rounded, but only its Bold weight is free (CC BY-SA 2.5), not Heavy — [alefalefalef free fonts](https://alefalefalef.co.il/en/?p=3941) (via search summary). The coverage check above **partly closes gap item 296 in `ui_ux_polish_craft.md`**: Rubik has nikud glyphs and GPOS mark positioning, and only rafe is missing. Mark-positioning *quality* on iOS CoreText still needs a device test.

**Recommendation [Inference from the measurements]**
- **Display/UI (buttons, titles, counters, badges): Rubik 800–900.** It is the heaviest Hebrew option, has a variable weight axis and full Latin plus digits, so one family covers mixed strings. Its Hebrew co-designer is an Israeli type designer (Meir Sadan).
- **Kid-facing friendly headers / character names: Fredoka 700** with a thicker outline (10–12%) to match Rubik's visual weight.
- **Reading text with nikud:** keep the separate reading font decided in `ui_ux_polish_craft.md` §4. **Do not outline nikud text.** In my Pillow+RAQM render, a 6 px stroke at 64 px (9%) visibly thickens and nearly merges patah/qamats with the outline under ע and ש [Measured visually]. If emphasis is needed on nikud text, use color plus a soft shadow only.

### 3.4 Type tokens (932×430, pt) [Inference]

| Token | Hebrew size | Latin/digits size | Weight | Fill | Outline (stroke) | Hard drop | Use |
|---|---|---|---|---|---|---|---|
| `title.xl` | 44 | 38 | Rubik 900 | white→`#FFF3C4` vertical gradient | 4 pt (≈9–10%) role-dark | 0,4 (≈10%) | Victory, popup headers |
| `title.l` | 30 | 26 | 900 | white | 3 pt | 0,3 | panel headers, ribbons |
| `button.xl` | 36 | 30 | 900 | `#FFFBEA` | 3.5 pt | 0,3.5 | Play |
| `button.m` | 24 | 20 | 800 | white | 2.5 pt | 0,2.5 | secondary buttons |
| `counter.l` | — | 26 | 900 | white | 3 pt | 0,3 | currency, trophies |
| `label.s` | 14 | 12 | 700 | white | 2 pt | 0,1.5 | tile labels, badges |
| `body` | 17 | 15 | Rubik 500 | `#1E2A3A` on light panels / white on dark | none | none | descriptions (≤ 2 lines) |

Rules:
- **Outline ≈ 8–10% of font size** (min 2 pt), **hard drop ≈ 10–12% straight down, 0 blur**, both in the *role-dark* hue rather than black. The cited community values are 1 px outline + 2–3 px drop at 16–24 px, i.e. 4–6% outline and 12% drop (§2.1, §3.1). Real Supercell text reads heavier on device, so this guide rounds the outline up [Inference].
- **Hebrew has no capital letters.** Latin UI in this genre is often all-caps. Hebrew gets its hierarchy from **size, weight, color and outline** instead. The Hebrew letter body is ≈0.58 em vs a Latin cap height of ≈0.70 em in Rubik [Measured], so **Hebrew labels need about +15–20% size** to match an all-caps Latin label. This agrees with Apple's "+about 2 pt for RTL next to uppercase Latin" in `ui_ux_polish_craft.md` §4.
- **Letter-spacing:** 0 for Hebrew (it is already wide). Latin all-caps +2–4%. Digits use tabular figures when counting up, so width doesn't jitter.
- **Numbers:** thousands separators (`8,430`). Abbreviate from 10k (`12.5K`). Digits read LTR inside RTL strings, so wrap mixed strings in FSI/PDI isolates. This pattern appears in a real RTL HUD PR — [woogitsugames/marchi #57](https://github.com/woogitsugames/marchi/pull/57) (via search summary).
- **Web/RN implementation:** `-webkit-text-stroke` plus `paint-order: stroke fill`, so the stroke sits *outside* the fill. HTML-text support exists in Firefox 60+ and Safari 11+, and Chromium arrived around March 2024 — [MDN paint-order](https://developer.mozilla.org/docs/Web/CSS/paint-order); [BCD issue 17951](https://github.com/mdn/browser-compat-data/issues/17951) (via search summary). For native SwiftUI, render the stroke as 8 offset copies or use a Core Text stroke with a fill pass on top [Inference].
- **Licensing:** Supercell-Magic and You Blockhead are commercial/proprietary (You Blockhead from $29 on [MyFonts](https://myfonts.com/fonts/comicraft/you-blockhead), via search summary). **Do not use any "free download" copies.**

---

## 4. Color system, rarity, backgrounds, panels, pills, bars, timers, tabs

### 4.1 Role colors (cited)
- Clash Royale uses **yellow** for the most important actions, **green** for secondary actions (Donate, Buy, Upgrade), **red** for notifications and **blue** for backgrounds and less important elements — [Górnicki pt.1](https://www.gornicki.me/blog/90l/ux-in-clash-royale-part-1) (via search summary); [The Rookies](https://discover.therookies.co/2020/02/24/game-design-ux-best-practices-detailed-breakdown-of-clash-royale) (via search summary).
- In the Brawl Stars battle HUD, the movement joystick is **blue**, attack **red**, Super **yellow** and Hypercharge **purple** — [ar-pay controls guide](https://ar-pay.com/blog/en/articles/brawl-stars/) (via search summary; third-party).
- Advice from a UI/UX art director: have **one "hot-action" color**, and pick **two fonts** (title + body) — [Medium/Design Bootcamp: 7 HUD mistakes](https://medium.com/design-bootcamp/7-obvious-beginner-mistakes-with-your-games-hud-from-a-ui-ux-art-director-d852e255184a) (via search summary).

### 4.2 Rarity coding (cited, community data)
- **Brawl Stars tiers:** Starting, Rare, Super Rare, Epic, Mythic, Legendary, Ultra Legendary — [Supercell support](https://support.supercell.com/brawl-stars/en/articles/brawler-rarities-3.html) (via search summary). Community hex values (mirroring the Brawlify API): Common/Starting `#b9eaff`, Rare `#68fd58`, Super Rare `#5ab3ff`, Epic `#d850ff`, Mythic `#fe5e72`, Legendary `#fff11e` — [Elomolina/brawl-stars-app interface](https://github.com/Elomolina/brawl-stars-app/blob/17599f7442bbed2718ed9e56b64feac1e9e8a8f5/src/app/brawl-stars/interfaces/brawlers.interface.ts) [Measured from source]. Ultra Legendary `#ff9d2e` comes from one fan project only — [DeniTutic/brawldle constants](https://github.com/DeniTutic/brawldle/blob/58f3d7f2e7ce3c0fbd7c6c6d6c10fdf272f21074/src/lib/constants.ts) [Measured; low confidence].
- **Clash Royale:** Common = pale/dark blue, Rare = orange/pale yellow, Epic = magenta/purple, Legendary = **rainbow gradient**, Champion = aureolin (gold-yellow) — [Clash Royale fandom: Cards](https://clashroyale.fandom.com/wiki/Cards) (via search summary). A fan card renderer uses common `#666`, rare `#FDA03C`, epic `#C3F`, legendary `#65BABA` — [cr-card](https://raw.githubusercontent.com/Dabolus/cr-card/master/cr-card.html) [Measured].
- **The genre-wide rule [Inference]:** cool/low → warm/high. Light blue or gray → green → blue → purple → red/pink → gold → animated multicolor at the top. Escalation is in **hue + saturation + motion**: higher tiers get animated shimmer, particles and rays.

**For our game (aligned with D2, "rarity encodes effort")** [Inference]: use a 3–4 tier *achievement* frame system rather than random rarity. Episode = **bronze/teal** `#3CC8C8`, skill mastered = **silver-blue** `#5AB3FF`, unit/boss = **purple** `#B05CFF`, year milestone = **gold animated** `#FFC21A`→`#FFF1A8` shimmer. Always pair the color with a **frame shape** (circle, shield, star, crown) for color-blind safety, as the gap audit requires (item 14). Avoid pink/blue gender coding.

### 4.3 Backgrounds [Inference: genre technique]
- **Lobby:** the seasonal 3D/2.5D diorama (§1.3).
- **Menus/popups backdrop:** a full-screen **radial gradient** with a saturated mid-blue center (`#2F7BFF`) and deep navy edges (`#0B2160`), plus a **subtle repeating pattern** (diagonal stripes, tiny stars or a brand motif) at 6–10% opacity scrolling diagonally at 6–10 pt/s. Add a vignette (edges −30% luminance).
- **Reward moments:** rotating **light rays** (12–16 wedges, alternating 0%/18% white, 1 rev per 8–12 s) behind the item, plus a soft bloom/glow in the tier color.
- **Never** use a flat solid color or photographic backgrounds behind UI.

### 4.4 Panels and popups [Inference unless cited]
- Popups over full screens, with the previous UI visible behind — [The Rookies](https://discover.therookies.co/2020/02/24/game-design-ux-best-practices-detailed-breakdown-of-clash-royale) (via search summary).
- **Scrim:** black at 55–65%, fading in over 150 ms.
- **Popup panel:** max 640×340 (≈79% × 83% of the usable area). Radius 20, outline 3 pt navy `#0A2D66`, body fill a light cream/blue gradient (`#F4F8FF`→`#DCE8FA`) or deep blue for "dark" panels. 4 pt inner light stroke at 30% white.
- **Header ribbon:** a banner wider than the panel, overhanging each side by 12–16 pt with folded tails. Height 52, filled with the role color, title in `title.l`, centered.
- **Close X:** red round button, 44 pt visual / ≥60 pt hit, at the **top outer corner**: top-**left** in RTL (see §10), offset half outside the panel (−12, −12).
- **Dark translucent info bars:** black at 45–55% with radius 12, for tooltips and "time left" strips over art.
- **Content density:** ≤ 2 lines of body text per popup, with 1 primary and 1 secondary button at most.

### 4.5 Currency pills [Inference]
- Pill 118×34, radius 17, dark translucent fill (`#0B1A3A` at 70%) with a 2 pt outline.
- **Icon 40×40 overlaps the pill's start edge** (right in RTL) by about 12 pt and is vertically centered, so the icon looks like it sits *on top of* the bar.
- Value in `counter.l` (26 pt) right after the icon, plus an optional green "+" mini-button (26 pt) at the end edge.
- Value changes **count up** (§7), and the icon **pulses** when a reward "flies" into it. Flying coin particles arc from the source to the pill icon over 450–700 ms, staggered by 40 ms.

### 4.6 Progress bars (trophy road, pass, mastery) [Inference]
- Track: height 14–18 pt inside a 2–3 pt dark outline with radius = h/2, and a dark inset trough (`#0A1F44`).
- Fill: role color gradient with a top 40% highlight band (same as buttons) and a moving diagonal stripe texture at 15% opacity.
- **Milestone nodes** sit *on* the bar as icons (28–36 pt) with a tier frame. Reached nodes glow. The next node bobs.
- **End cap:** a bigger reward icon (40 pt) overlapping the end.
- Fill direction: **right→left in RTL** (§10). Animate fills with ease-out over 600–900 ms, then show a "+N" popping text.
- **Long tracks** (pass/road) scroll horizontally with snap-to-tier. The current tier is centered, past tiers are dimmed 30%, and locked premium tiers have a lock plus purple tint.

### 4.7 Timers and tabs [cited + Inference]
- **Timers:** a dark pill with a clock icon and `1ש 23ד` format (2 units max, e.g., "2d 4h"). Turn it red under 1 h. **Don't use countdown pressure on kids** (`gap_audit.md` D10). Timers only label seasonal content.
- **Tabs:** Brawl Stars' April 2026 shop moved to **tabs; empty tabs are hidden; up to 7 tabs at a time; tabs shrink as you scroll; tapping an offer opens an expanded view with descriptions** — [Supercell release notes April 2026](https://supercell.com/en/games/brawlstars/blog/release-notes/release-notes-april-2026/) (via search summary). Our tab spec: height 44–48. The selected tab is raised by 4 pt, uses the role color and bright text, and is connected to the panel. Unselected tabs are darker blue with text at 70%. Each tab can carry a badge. Order tabs **right→left** in RTL.

---

## 5. Iconography and currency icons [Inference unless cited]
- The genre style is **chunky pseudo-3D painted/rendered icons with a thick dark outline**, not flat line icons. Game UI Database classifies Brawl Stars' style as **flat with some skeuomorphic and 3D elements** — [Game UI Database: Brawl Stars](https://www.gameuidatabase.com/gameData.php?id=465) (via search summary). Note that its **terms of use prohibit AI/ML use of its content**, so a human designer should browse it, not an agent scraping it. Squad Busters' principal artist lists **Icons/UI, VFX, BG/Props, splash, style guide and app icon** as one "visual code" led by the art director — [Squad Busters VFX credits/ArtStation](https://www.artstation.com/artwork/x3yalX) (via search summary). Icons are art-directed with the world, not sourced from an icon font.
- **Icon recipe:** a 3/4 top-down view (≈20–30° tilt). Exaggerated silhouette with a chunky base. 3–4 value bands per material: shadow, base, light, specular "sticker" highlight. **Outline 2.5–3 pt at 40 pt size (≈7%)** in a darkened hue of the local color (dark brown on gold, navy on blue). Optional 1 pt outer white "die-cut" stroke on busy backgrounds. A cast shadow (ellipse, 30% black) for standalone items.
- **Currency icons:** soft currency = **gold coin** (thick rim, embossed emblem, 2 specular glints). Premium = **faceted gem** (green or purple, 5–7 facets, a strong white glint plus a sparkle loop every 3 s). Chest/drop = a box/capsule silhouette whose **frame color = tier**. For our game: **max 2 currencies** (stars = earned, keys/gems = rare milestone). The psychology note prefers no currency or a non-purchasable one ([`gap_audit.md`](./gap_audit.md) C1).
- **Nav icons** (Shop, Heroes, Album) are illustrated **objects** (a market stall, a helmet, a book), not glyphs. Use a consistent light direction (top-left) and the same outline width across the whole set.
- **Readability test:** each icon must be recognizable at 24 pt **in silhouette** (solid black fill).

---

## 6. Key screens

### 6.1 Character select grid and character detail
- **Grid [Inference]:** cards 112×140 (4:5) with a 10 pt gap, so **6 columns** in the 794 pt content box (722 pt used plus a 72 pt side filter column). Card anatomy: tier-colored background with a radial highlight behind the portrait; portrait breaking the top edge by 8–10 pt; name banner at the bottom (`label.s`/16 pt); level/mastery badge top-start; "new"/"upgrade available" badges top-end. **Locked:** dark silhouette plus a lock, with the unlock condition in one line ("סיים את עולם המספרים").
- Sort by tier/unlock, then progress. The selected card gets a 3 pt white glow outline plus a 1.05 scale.
- **Detail [Inference]:** the large character model fills about 45% of the width on the **start side (right in RTL)** and rotates by drag. On the other side: name (`title.l`), role tag, skill family, a mastery bar, and 2–3 ability cards (icon + 1 line). Bottom end corner: the green "upgrade/equip" CTA. Top: back button (chevron points **right** in RTL) and currency pills.
- The Brawlers screen is also where skins are viewed — [Samurai Gamers](https://samurai-gamers.com/?p=126011) (via search summary). Rarity shows on tap — [Supercell support](https://support.supercell.com/brawl-stars/en/articles/brawler-rarities-3.html) (via search summary).

### 6.2 Shop/offer cards [cited + Inference]
- Brawl Stars 2026 shop: tabbed, empty tabs hidden, ≤7 tabs, an expanded view with item descriptions — [Supercell April 2026 notes](https://supercell.com/en/games/brawlstars/blog/release-notes/release-notes-april-2026/) (via search summary).
- **Card [Inference]:** 180×260 in a horizontal scroller (≈4 visible). Layers: tier-color frame → art area (60%) with rays → **diagonal corner ribbon** ("חדש!", "הכי שווה") → content icons with ×N counts in `counter.l` → **price button** at the bottom (green M button with a currency icon plus number; red number if unaffordable). Timer pill at the top. "Value" bubble (e.g., "x3") as a starburst sticker overlapping the top-end corner at 8° rotation.
- **For the family app:** the "shop" spends earned stars only. Any real-money item is behind the parent gate, and there are **no randomized offers** (D2).

### 6.3 Reward / drop opening sequence
**Cited:**
- Brawl Stars Starr Drops open after matches. "Each time you tap, you gain a small chance to increase the rarity" — [ClutchPoints](https://clutchpoints.com/gaming/brawl-stars-starr-drops-explained) (via search summary). The upgrade outcome is decided before the animation — [AFK Gaming](https://afkgaming.com/mobileesports/guide/brawl-stars-starr-drops-rewards-rarities-drop-chance-how-to-get-more) (via search summary).
- Clash Royale chest opening had **slow and fast modes**, which saves time when opening many chests — [The Rookies](https://discover.therookies.co/2020/02/24/game-design-ux-best-practices-detailed-breakdown-of-clash-royale) (via search summary).
- A commercial chest-opening asset that cites Clash Royale as the archetype lists opening particles and **rotating light rays** — [arastei chest animation](https://arastei.itch.io/classic-chest-opening-animation) (via search summary).
- No primary Supercell timing data exists publicly. `gap_audit.md` G14 proposes an in-house frame-step study.

**Our deterministic sequence (milestone tier, ≤4–6 s total, skippable after beat 2) [Inference]:**

| Beat | t (ms) | What happens | Sound/haptic |
|---|---|---|---|
| 0 Enter | 0–300 | Scrim fades in. The container drops in from above with squash on landing (scale-Y 0.85→1.08→1.0). | thud + light haptic |
| 1 Anticipation | 300–1300 | Wobble every 1 s, growing (±3°→±6°). Rays fade in. Glow pulses in the **tier color, already known** (D2: contents shown as a silhouette). "Tap!" hint after 1.5 s idle. | rising shimmer |
| 2 Tap(s) | per tap | Each tap: punch scale 1.12, 6–10 sparkles, camera micro-shake 2–3 pt for 80 ms. For a higher tier, the 2nd–3rd tap adds an **extra flourish**, but the tier never changes randomly. | tap "tink", ascending pitch |
| 3 Burst | +0–250 | Lid pops, white flash (0→80%→0 in 200 ms), radial particle burst (40–80), rays speed up ×3. | burst + success haptic |
| 4 Reveal | +250–1200 | Item flies up and scales 0→1.15→1.0 (ease-out-back). Tier frame and name ribbon slide in. Counter counts up "×3". | tier fanfare (longer for higher tier) |
| 5 Settle | +1200–2000 | Item idles (bob 4 pt). Green "אסוף!" (collect) button appears. Tap flies it to its home icon (pill/album) with trail particles. | collect chime |

For multiple items, reveal **lowest to highest** so it ends on the peak (the Clash Royale convention per the fan sources in `ui_ux_polish_craft.md` §3), with a "skip to summary" grid.

### 6.4 Results / victory screen [Inference; cited mechanics]
- Brawl Stars applies trophy gains/losses per brawler at match end, and rank-ups give rewards — [Samurai Gamers: Trophies](https://samurai-gamers.com/brawl-stars/trophies/) (via search summary); [esports.net ranks](https://www.esports.net/news/mobile-games/brawl-stars-ranks/) (via search summary).
- **Layout:** banner (`title.xl`, "!ניצחון" or for learning "!כל הכבוד") drops in at the top center (y 24–84), with ribbon tails and rays behind it. Center: the hero(es) on the stage doing a victory emote (the team lined up for co-op, a "star player" crown over the MVP). Below them: a **reward row** of 2–4 chips (stars +12, mastery +1, new sticker), each counting up in sequence with 150 ms stagger. Bottom: **"Play again"** yellow XL at the RTL end (bottom-left) and **"Home"** blue M at the other corner.
- **Losing / "not yet" for kids:** no "Defeat" red screen. Use a warm "!כמעט" ("almost!") with the same layout, a smaller reward and an encouraging character line (`gap_audit.md` D10 frustration guard).

### 6.5 Battle/activity HUD
- **Cited Brawl Stars defaults:** blue joystick bottom-left. Red attack button right. Yellow Super below or near attack. Purple Hypercharge on the right below the Super area. All positions are user-editable in Settings → Edit Controls — [ar-pay](https://ar-pay.com/blog/en/articles/brawl-stars/) (via search summary); [Brawl Stars wiki via search](https://brawlstars.fandom.com/wiki/Mega_Box) (via search summary). Brawl Stars controls were heavily prototyped to be touch-native — GDC 2019, cited in `ui_ux_polish_craft.md` §1. Health/ammo bars were restyled in a later update — [Pocket Gamer](https://www.pocketgamer.com/articles/076234/r/) (via search summary).
- **Our HUD rules [Inference]:**
  - **Movement-type activities:** joystick zone = the whole lower-left 40% of the screen, as a floating stick (appears where the thumb lands; base 120 pt, knob 56 pt, 30% opacity at rest). Action buttons in the lower-right arc: main 96 pt, secondary 64–72 pt at 20–30° steps around it. **Do not mirror these for RTL** (§10).
  - **Quiz/learning activities:** answer buttons are large (≥ the age floor) in a 2×2 or 1×3 row across the bottom 45% of the screen. Prompt/visual in the top-center 50%. The voice-replay button sits beside the prompt.
  - **Top strip** (y 10–50): round progress (dots or bar) in the center, pause at the **top-start corner (top-right in RTL)**, and a timer only in opt-in speed rounds (D10).
  - **Health bars over characters:** 60×8 pt with a 2 pt dark outline, green→yellow→red. Ally/enemy is identified by the **outline color** (blue/red). The name sits above in `label.s`.
  - **Text in HUD:** none except numbers and ≤2-word callouts. All feedback is iconic, voiced and animated.

### 6.6 Map / world-mode select [Inference]
- Clash Royale's Summer 2025 **Game Mode Switcher** sits beside the Battle button — [Sportskeeda](https://www.sportskeeda.com/mobile-games/clash-royale-summer-update-2025-game-mode-switcher-trophy-road-rework) (via search summary). Brawl Stars has a dedicated **game mode screen / Event screen** — [Brawl Stars wiki version history](https://brawlstars.fandom.com/wiki/Version_History) (via search summary).
- **Our select screen:** a horizontal carousel of world cards (200×240), the current one centered and 1.1× scale. Each card has art, name, a mastery bar, a lock state with its condition, and "new" or "ready to review ✨" badges (D19 uses positive "ready" cues, never decay). Tapping a card selects it and returns to the lobby with the **mode card updated next to Play**, which keeps the Brawl Stars "pick → Play" two-step.
- **For a node-path map** (the learning road): nodes 56–72 pt on a winding path. The current node is a bouncing hero token. Completed nodes show tier frames. The path scrolls in the RTL direction (progress goes right→left).

---

## 7. Motion and juice

**Cited principles**
- Juice = tweening/easing, screen shake, particles, sound, layered on without new mechanics — [Juice It or Lose It (GDC Vault)](https://gdcvault.com/play/1016789/Juice-It-or-Lose); [gamejuice summary](https://gamejuice.pages.dev/articles/juice-it-or-lose-it-lessons) (via search summary). Martin Jonasson later warned that too much polish can cost immersion — same search.
- Supercell's 2026 Clash Royale UI-artist role explicitly asks for "polished UI animations and motion treatments that make the interface feel responsive, satisfying, and alive" — [Supercell careers: Senior UI Artist](https://supercell.com/en/careers/senior-ui-artist/66476c5d-30b6-4772-aa41-94d251e3d73c/) (via search summary).
- Clash Royale's key controls get extra animated highlights — [Górnicki pt.1](https://www.gornicki.me/blog/90l/ux-in-clash-royale-part-1) (via search summary).

**Motion tokens [Inference; press values seeded from §2.2 sources]**

| Token | Value |
|---|---|
| `press` | scale 0.93, 70 ms ease-out; release 180 ms ease-out-back (overshoot s≈1.7, about 8%) |
| `pop.in` (badges, icons) | scale 0→1.2→1.0, 250 ms, back-out |
| `popup.open` | scrim 150 ms; panel scale 0.85→1.04→1.0 + fade, 260 ms |
| `popup.close` | scale 1→0.92 + fade, 140 ms ease-in |
| `screen.transition` | 200–300 ms: outgoing UI slides off toward the screen edges (rails out sideways, bars up/down) while the background crossfades. The hero stays; it is the anchor. Or an iris/cloud wipe of 350 ms for world changes. |
| `stagger` | 40–60 ms between list items/tiles appearing |
| `countup` | 0.6 s (small) to 1.2 s (big), ease-out-quart, tabular digits, a tick sound every ~60 ms with rising pitch, final punch scale 1.2→1.0 plus a sparkle |
| `idle.breathe` | scale 1.00↔1.03, 2–2.4 s sine loop (CTA, hero) |
| `shine.sweep` | every 3–5 s on the CTA, 500 ms |
| `shake.small` | 2–3 pt, 80–120 ms, decaying (taps on chests, impacts) |
| `shake.big` | 6–8 pt, 250 ms (boss hit only; **off for the toddler profile**) |
| `fly.to.counter` | 450–700 ms arc (quadratic Bézier, apex 80–120 pt), 40 ms stagger, ends in a pill pulse |
| `particles` | sparkles 6–10 per tap, 40–80 per burst. Max lifetime 0.8 s. Two-tone (white + tier color). Star/diamond shapes, not circles. |

**Budgets (from `gap_audit.md` D7):** per answer ≤0.5–0.8 s, non-blocking. End of round ~1.5 s. New item 2–3 s. Milestone 4–6 s, skippable. Total celebration + transition ≤15% of session time.

**Sound cues [Inference]:** every press gets a soft "thock" (pitch varies ±5% to avoid fatigue). Popup open "whoosh-pop". Coin/star collect "tink" layered and pitched up in sequence. Reward fanfare scales with tier. Error is a soft low "bonk", never a harsh buzzer. Ambient lobby music has a seasonal variant (cited precedent: seasonal menu music, §1.1). Each sound pairs with a haptic (light / medium / success).

---

## 8. Phone landscape layout rules

- **Safe area (932×430):** left/right 59 pt, bottom 21 pt, top 0 — [useyourloaf](https://useyourloaf.com/blog/iphone-15-screen-sizes/) (via search summary). The Dynamic Island sits on one *side* in landscape (which side depends on the rotation direction), so keep both sides' 59 pt free of interactive UI and important text. Backgrounds and art bleed. Some engines report wrong safe areas after rotation, so read insets on every layout pass — [Unity issue tracker](https://issuetracker.unity3d.com/issues/ios-sometimes-the-safe-area-is-incorrect-when-the-player-is-in-the-landscape-mode) (via search summary).
- **Thumb zones (two-hand landscape grip) [Inference]:** the easiest zones are bottom-left and bottom-right quarter-circles of ≈180 pt radius from each lower corner. Next comes the bottom-center band. The hardest zone is the top-center (both thumbs must stretch), so use it only for passive info (round progress, banners). Clash Royale keeps "all key elements … within reach of your finger" even one-handed — [The Rookies](https://discover.therookies.co/2020/02/24/game-design-ux-best-practices-detailed-breakdown-of-clash-royale) (via search summary).
- **Corner logic:** primary CTAs go in **bottom corners** (Brawl Stars Play bottom-right; CoC 2026 Battle bottom-left and Build bottom-right). Identity/profile goes in a **top corner** (CoC 2026 profile top-left). Currencies go in the **opposite top corner** (cited in §1.1).
- **Touch targets:** a 44×44 pt platform minimum ([secondary summary of Apple HIG](https://smashingmagazine.com/2012/02/finger-friendly-design-ideal-mobile-touchscreen-target-sizes/), via search summary). **Our age floors (G9):** 8.5 ≥60 pt, 5.5 ≥90 pt, 2.5 ≥120 pt on iPhone. Keep ≥8 pt between adjacent targets.
- **Text density [Inference]:** ≤ 7 text strings visible on the lobby (excluding numbers). Labels ≤ 2 words. Descriptions ≤ 2 lines and only inside popups. Prefer **icon + number** over sentences. Every instruction is also **voiced** for pre-readers.
- **Grid [Inference]:** 8 pt base unit, 10 pt gutters, a 12-column grid over the 794 pt content box (each column ≈ 57 pt + 10 gutter).

---

## 9. "Premium" vs "cheap imitation": the tells

Cited general advice: half-stylized art "looks cheap", button styles changing between screens is noticed, define states, text hierarchy and animation early — [Medium/Design Bootcamp: 8 mistakes](https://medium.com/design-bootcamp/the-8-obvious-ui-ux-design-mistakes-in-your-portfolio-game-or-app-a-practical-guide-for-83e352134c7b); [A few common pitfalls](https://medium.com/@doandaniel/a-few-common-pitfalls-to-avoid-in-indie-game-development-944a5713425d) (via search summary). Thin lines "turn damn-near translucent on mobile", and you should use two fonts and one hot-action color — [7 HUD mistakes](https://medium.com/design-bootcamp/7-obvious-beginner-mistakes-with-your-games-hud-from-a-ui-ux-art-director-d852e255184a) (via search summary). A generic "mobile-style UI" clashing with the game's art looks out of place — [itch.io comment](https://itch.io/post/12997684) (via search summary).

**Checklist [Inference: practitioner synthesis]**

| Cheap tell | Premium fix |
|---|---|
| Thin (1 px) or missing outlines; outline width varies between elements | One stroke scale: 3 pt on L/XL elements, 2–2.5 pt on M, 2 pt on S, always role-dark (not black) |
| Muddy gradients (gray midtones, low saturation, gradient through brown) | Gradients move **in hue and lightness together** (yellow→orange, not yellow→gray). Keep saturation high in the midtones. Use 4 tones per role (§2.2). |
| Flat buttons without a lip, or a soft blurry drop shadow | A hard lip (≈10–12% of height) plus a 0-blur drop. Light comes from above, everywhere. |
| Wrong font weight (Regular/Bold system fonts) or many fonts | One heavy rounded display family + one body family. Outline and hard shadow on all display text. |
| Too much text; sentences on buttons | 1–2-word labels, icons, numbers, voice-over |
| Flat lighting on characters and icons; no rim light | Key + fill + **rim light**. 3–4 value bands. Specular "sticker" glints. |
| Inconsistent icon styles (stock flat icons beside painted ones) | One art-directed icon set, same angle, light and outline |
| Every element equally loud (5 yellow buttons) | One hot CTA. Everything else stepped down in saturation and size. |
| Static UI: no press feedback, no idle life | Press squash, overshoot, count-ups, idle breathe/shine |
| UI hugging the screen edges or under the Dynamic Island | Safe-area-aware layout, 10 pt inner gutter, art bleeds |
| Pixel-soft or blurry assets | Vector or @3x assets. Strokes snapped to whole points. |
| Generic "free UI kit" look | Shapes and motifs derived from *your* world: the ribbon tails, pattern and badge shapes echo the game's theme |
| Copying Supercell layouts and fonts outright | Same *principles*, original art, your own fonts (OFL) and world motifs. Supercell's lettering is a commercial custom font. |

---

## 10. RTL (Hebrew) adaptation

**Cited rules**
- Mirror back/forward navigation and icons that show forward movement. Linear time is mirrored, but **clocks still turn clockwise**. **Media playback controls and media progress are not mirrored** (they represent tape direction). Don't mirror icons with a right-handed design (e.g., the search handle) or slashes — [Material Design bidirectionality (m2)](https://m2.material.io/design/usability/bidirectionality.html); [m1](https://m1.material.io/usability/bidirectionality.html) (via search summary). Mozilla: mirror **progress bars** and back/forward, but not A/V controls — [Firefox RTL guidelines](https://firefox-source-docs.mozilla.org/code-quality/coding-style/rtl_guidelines.html) (via search summary). Wikimedia Codex: don't mirror clocks, calendars or play/pause — [Codex bidirectionality](https://doc.wikimedia.org/codex/latest/style-guide/bidirectionality.html) (via search summary).
- RTL readers expect the **whole layout** to flip, not just the text. Retrofitting RTL is one of the costliest localization jobs, so plan it from the start — [The Gaming Nest](https://thegamingnest.com/en/articles/077a1a77-498a-41df-84fd-6953fc5fab8f); [A+ Games](https://aplusgamesstudio.com/blog/arabic-game-development-rtl) (via search summary). An RTL HUD PR kept playback-like speed controls LTR and isolated mixed number strings with FSI/PDI — [marchi PR #57](https://github.com/woogitsugames/marchi/pull/57) (via search summary). No authoritative rule on joysticks was found.

**Mirror / don't-mirror table for this game [Inference applying the cited rules]**

| Element | RTL treatment |
|---|---|
| Lobby shell: profile chip, currency side, nav rail side, mission cards | **Mirror** (profile top-right, currencies top-left, nav rail on the right) |
| Reading order of rows, tabs, carousels, grids | **Mirror** (first item at the right) |
| Progress bars, pass/road tracks, step dots, map path progress | **Mirror** (fill right→left) |
| Back/next chevrons, "go" arrows, swipe hints | **Mirror** |
| Icon + label inside buttons | **Mirror** (icon on the right) |
| Currency pill icon side | **Mirror** (icon overlaps the right edge) |
| Close X on popups | **Mirror** to top-left (consistent with "start = right"). Keep it in the same place on every popup. |
| Badge on tiles | **Mirror** to the outer-top corner |
| Text alignment | Right-aligned. Centered for titles and buttons. |
| **Numbers, timers, scores** | **Do not mirror digits.** They stay LTR inside RTL text (use bidi isolates). |
| Clocks, circular timers, loading spinners | **Do not mirror** (clockwise) |
| Media/voice replay ▶, audio scrubbers | **Do not mirror** |
| **Gameplay controls (joystick left, actions right)** | **Do not mirror.** It is a motor convention shared with Brawl Stars and every action game the kids know. Offer a left-handed swap setting instead. |
| Character art, facing direction, logos, the hero's weapon hand | **Do not mirror** (asymmetric art looks wrong flipped). Author poses to face the screen center. |
| Light direction in art | **Do not mirror** (keep top-left for asset reuse) |
| Shine sweep direction | Optional: sweep right→left to follow the reading flow |
| **Primary CTA (Play) corner** | **Open decision.** The default here is **bottom-left** (end of the RTL reading path; CoC 2026 shows a primary action works there). Alternative: keep bottom-right for right-thumb dominance, as in Brawl Stars. **Playtest both with the 8.5- and 5.5-year-olds** and log time-to-tap and mis-taps. Whichever wins, keep the mode card adjacent to Play on the screen-center side. |

**Hebrew-specific type notes:** +15–20% size over Latin caps (§3.4). Line-height ≥1.25 for unpointed and ≥1.4 for pointed text. Never outline pointed text. Bidi isolate any string mixing Hebrew and digits/Latin. Test the confusable letter pairs listed in `ui_ux_polish_craft.md` §4 in the display font as well (ב/כ, ד/ר, ה/ח/ת, ו/ז/ן, ס/ם).

---

## 11. Reference URLs by screen type (for a human designer to browse)

Many of these hosts were blocked from this container. A designer should open them in a browser. Note the Game UI Database AI/ML terms in §5.

**Whole-game UI galleries**
- Game UI Database – Brawl Stars (screens tagged by type: tile menus, tabs, radial) — https://www.gameuidatabase.com/gameData.php?id=465 (search the same site for Clash Royale, Clash of Clans, Squad Busters, Stumble Guys, Subway Surfers, Archero, Rush Royale, Pokémon UNITE)
- Interface In Game – Brawl Stars — https://interfaceingame.com/games/brawl-stars/ (use the site search for the other titles)

**Lobby / home**
- Pratt IxD design critique of the Brawl Stars home — https://ixd.prattsi.org/2025/02/design-critique-brawl-stars/
- Medium/SWLH, "Brawl Stars — What makes it great" (menu hierarchy) — https://medium.com/swlh/brawl-stars-what-makes-it-great-3861dfb169fb
- Samurai Gamers Brawl Stars main-menu map (older version) — https://samurai-gamers.com/brawl-stars/main-menu-guide/
- Supercell: CoC UI revamp, Oct 2026 (hub consolidation rationale) — https://supercell.com/en/games/clashofclans/blog/news/weve-revamped-the-clash-of-clans-user-interface/
- Clash Royale Summer 2025 (Game Mode Switcher, tab redesign) — https://www.sportskeeda.com/mobile-games/clash-royale-summer-update-2025-game-mode-switcher-trophy-road-rework
- Stumble Guys main-menu revamp, June 2026 — https://www.stumbleguys.com/news/upcoming-changes-coming-to-ability-progression-and-main-menu
- Student recreation of the Brawl Stars main menu (process) — https://thewolfg.itch.io/uiux-braw

**Buttons / typography**
- cr-button (layer CSS of a Clash Royale-style button) — https://github.com/Dabolus/cr-button
- cr-fonts (title/message text styling) — https://github.com/Dabolus/cr-fonts
- Comicraft on Supercell fonts — https://comicraft.com/?p=1082
- Construct tutorial: designing professional mobile game buttons — https://construct.net/en/tutorials/design-professional-looking-911
- Google Fonts Hebrew: Rubik https://fonts.google.com/specimen/Rubik · Fredoka https://fonts.google.com/specimen/Fredoka · curated list https://github.com/danielrosehill/Awesome-Hebrew-Fonts

**Color / rarity**
- Brawl Stars rarities (official) — https://support.supercell.com/brawl-stars/en/articles/brawler-rarities-3.html
- Clash Royale card rarities and colors (fandom) — https://clashroyale.fandom.com/wiki/Cards
- UX in Clash Royale pt.1–3 (color roles, popups, tutorial, loading) — https://www.gornicki.me/blog/90l/ux-in-clash-royale-part-1 · https://www.gornicki.me/blog/Woq/ux-in-clash-royale-part-2 · https://gornicki.me/blog/Bd87/ux-in-clash-royale-part-3
- The Rookies Clash Royale UX breakdown — https://discover.therookies.co/2020/02/24/game-design-ux-best-practices-detailed-breakdown-of-clash-royale

**Shop**
- Brawl Stars April 2026 release notes (tabbed shop) — https://supercell.com/en/games/brawlstars/blog/release-notes/release-notes-april-2026/

**Reward / reveal**
- Starr Drops mechanics (tap-to-upgrade) — https://clutchpoints.com/gaming/brawl-stars-starr-drops-explained
- Clash Royale deconstruction (session and chest pacing) — https://mobilefreetoplay.com/deconstructing-clash-royale/
- Frame-step your own device recordings (`gap_audit.md` G14). No public timing source exists.

**Battle HUD**
- Brawl Stars control layout guide — https://ar-pay.com/blog/en/articles/brawl-stars/
- GDC 2019, Designing Brawl Stars controls — https://gdcvault.com/browse/gdc-19/play/1025751

**Squad Busters (art direction, VFX)**
- Deconstructor of Fun deconstruction — https://www.deconstructoroffun.com/blog/2024/5/26/squad-busters-supercell-hits-the-apex-or-spins-off-track
- OCTOPO VFX on ArtStation — https://www.artstation.com/artwork/x3yalX
- Note: Supercell announced the game's sunset in Oct 2025 — [Wikipedia](https://en.wikipedia.org/wiki/Squad_Busters) (via search summary).

**Juice / motion**
- Juice It or Lose It — https://gdcvault.com/play/1016789/Juice-It-or-Lose

**RTL**
- Material bidirectionality — https://m2.material.io/design/usability/bidirectionality.html
- Mozilla RTL guidelines — https://firefox-source-docs.mozilla.org/code-quality/coding-style/rtl_guidelines.html

---

## 12. Gaps and what to do next

- **No screenshots were viewable.** Every Supercell layout position here comes from text sources plus genre familiarity. **Next step:** capture current Brawl Stars, Clash Royale and CoC lobbies on an iPhone with a 932×430 display. Overlay this guide's grid, measure element boxes (outline px, lip %, font cap height vs button height) and update the [Inference] numbers. That is measurement for principles, not asset copying.
- **No primary Supercell spec** (GDC talk or blog) on button layers, outline ratios or font sizes was found. The only measurable spec is the community `cr-button` recreation.
- **Brawl Stars' in-game UI font is unconfirmed.** Lilita One is a community look-alike only.
- **Reward-reveal timings and sound layers** remain unsourced (G14). Run the frame-step study.
- **The Play-corner decision in RTL** (bottom-left vs bottom-right) needs a playtest with the children.
- **Rubik/Fredoka nikud rendering on iOS CoreText** was not device-tested. Coverage and GPOS presence were verified only on the desktop with fontTools/RAQM.
- Rush Royale, Archero and Pokémon UNITE lobby specifics were **not found** in accessible sources. Their screens should be reviewed manually on Game UI Database / Interface In Game.
