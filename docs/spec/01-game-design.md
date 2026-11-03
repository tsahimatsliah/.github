# 01 — Game Design Spec: "מחנה הגחליליות" (Firefly Camp)

> **Status:** v1.0. Implementation-ready, with defaults for every ambiguity. Written 2026-11-02.
> **Binding on:** world, cast, loops, mechanics, battles, building, co-op, seasons, onboarding, stopping, celebrations.
> **Consistent with:** `00-foundation-decisions.md`, including §G (Experience pillars) and E1b (3D layer). Where this spec suggests a change to 00, the suggestion is at the end of the document ("Proposed change to 00"). Until it is accepted, 00 wins.
> **Research:** `docs/research-log/game-design.md` (R1–R12) plus the notes listed there.
> **Revision dependency:** the 3D-vs-2.5D tech spike (`tech_3d_world.md`, 4 go/no-go gates). This spec is **camera-agnostic**: every level is defined on a tile grid with elevation, so it works in R3F 3D and in Skia isometric 2.5D. Sections marked `⟲REVISE-AFTER-SPIKE` need a review after the spike.

---

## 0. Conventions

**IDs**

| Prefix | Meaning |
|---|---|
| `P-*` | Pillar |
| `R-*` | World region |
| `CH-*` | Character |
| `MX-*` | Learning mechanic / gimmick |
| `MV-*` | Movement moment |
| `LV-*` | Level template |
| `EV-*` | Event |
| `AC-*` | Acceptance criterion |
| `DD-*` | Default decision |

**Gendered strings.** Every string addressed to a child is written as `{m|f|pl}`. Example: `{מצאת אותו!|מצאת אותו!|מצאתם אותו!}` (the same form in some tenses), `{נסה|נסי|נסו}`.
- Neutral wording is preferred: the child's name, an infinitive, or "בואו נ…".
- Gender comes from `profile.grammaticalGender` (00 B3).
- K3's gender is still TBD, so every K3 line exists in both forms.
- Speech by a character about itself uses that character's fixed gender (see the cast).

**Age shorthand.** K1 = the girl, 8.5, grade 3. K2 = the boy, 5.5, gan chova. K3 = the toddler, 2.5, lap-mode.

**Age bands (00 A6).** Content and mechanics are tagged by band, never by child: `T` = 2–4 (today K3), `G` = 5–6 (today K2), `S` = 7–10 (today K1). `packages/shared` exports `Band` and `bandOf(child)` (`K3→T, K2→G, K1→S`, re-evaluated on birthdays/grade changes, DD-18). Where this document says "K2" in a mechanic card, read "band G"; the K-names are kept for readability only.

**Skill levels (00 C15).** The only level names that children and parents see are 08's: `new / beginner / advancing / proficient / master` (חדש / מתחיל / מתקדם / שולט / מאסטר). "Mastered" in this document = 08 `master` (D9). 02's internal economy trigger states are mapped once in `packages/shared/levels.ts` (02 §4.1).

**Phrases**
- "Learning action" = an action whose correctness depends on the skill (00 C1).
- "Episode" = one finite session (00 C3).

**Durations** are written `0:45` (minutes:seconds).

**Code identifiers**
- Code identifiers are in English, camelCase.
- Content IDs are kebab-case.
- Event names are snake_case.

---

## 1. Vision and pillars

**Vision in one sentence.** A small, warm family camp at the edge of a magical forest. Every evening, the light gathered there brings back lost friends. Each child explores, rescues and builds at their own level. The light, the rescue and the building are made of letters, numbers, English words and ideas, so the learning is literally the power (00 C1, G1–G7).

**The core feeling we're after.** "Astro Bot meets a cozy 99 Nights evening, in Hebrew, with our family inside it." It should feel like a console adventure that happens to teach, not an app with exercises.

| ID | Pillar | Test the developer can run (if the answer is "no", the design is wrong) |
|---|---|---|
| P-1 | **Learning = action** (C1) | Can you remove the learning content and still have a "game"? If yes, the game has degraded into a quiz. Every collect, rescue, attack and build depends on a correct learning action. |
| P-2 | **Joy of discovery** (G1) | In every 30 seconds of exploration, is there something to discover (a glint, a sound, a strange bush, a silhouette behind a gate)? Target: ≥1 interactable per 6×6 tiles of open space. |
| P-3 | **Rescue, don't score** (G2) | Is there a numeric score on screen? Forbidden. Progress is shown as rescued friends who appear in the world and in the hub. |
| P-4 | **One fresh idea per level** (G3) | Does the level introduce one gimmick and develop it in 4 beats? Two new gimmicks in one level is a bug. |
| P-5 | **Family together, without comparison** (C6, G5) | Is there a screen where one child sees a sibling's data next to their own? Forbidden. Co-op means shared goals and different roles. |
| P-6 | **Short, warm, finite** (C3, C5) | Does every episode end in a goodnight ritual plus an offline mission, with no autoplay and no "one more"? |
| P-7 | **Console-grade feel** (G7) | Does every touch get a response within ≤100ms, with sound plus animation (plus haptics on iPhone)? |
| P-8 | **The body is a controller too** (G6) | Does every episode for K2/K1 include ≥1 movement moment that has a seated alternative? |

**Two hard principles that cut across everything**
- **"Don't eject on a find"** (Odyssey): a rescue or collect never cuts to a results screen. The friend flies to the jar and play continues.
- **"Juice around the item, not on the item":** jokes and celebrations happen in transitions. While the child is solving, the screen is calm (an attention finding, R4).

---

## 2. The world

### 2.1 Name and premise (original)

**Name options for the world** (taste approval, E10). Default first:
1. **מַחֲנֵה הַגַּחֲלִילִיּוֹת** (Firefly Camp)
2. עֵמֶק הָאוֹר (Valley of Light)
3. מַחֲנֶה נִיצוֹץ

**Premise (shown to the children in a 40-second opening scene, with no text for K2/K3)**
- In the valley at the edge of "יער הלוחשים" there was once a happy camp.
- Its light came from **הָאוֹר הַכָּתוּב**, the written light: every lantern held a letter, every star held a number, and every song held words.
- One night **הַבַּלְגָּנִיסְט** (the Muddler) arrived. He is a theatrical shadow-magician in a top hat who loves surprises a bit too much. He sneezed one huge sneeze ("אַפְּצִ'יייי!"). The light-friends, called **נִיצוֹצוֹת** (the Sparklings), were scattered all over the forest.
- Now they are stuck in hard-to-reach places: inside mixed-up crystals, behind gates with jumbled letters, on top of trees, under a lake of fog.
- The camp went dim. Only the old campfire still glows quietly.
- The family arrives with **ניצי**, the last firefly left at the camp. They build a little hero out of an old lantern, called **the פָּנָסוֹן**, and set out to bring the friends home.
- Each friend who comes back adds light, life and new buildings to the camp.

**Why it is original and not a copy (R2, R12)**

| Element | Here | 99 Nights | Astro Bot |
|---|---|---|---|
| Fire | Never goes out. It grows with the family's learning. | Fire that goes out means the run is over. | — |
| Enemies | Lonely, prank-playing shadows that turn into friends. No horror. | Horror creatures (the Deer, cultists). | — |
| Rescued | Light creatures made of letters, numbers and words. Each rescued friend is a curriculum item. | 4 missing children. | 300 robots, a crew. |
| Hero | A **toy-suit with a living spark** (a lantern-being) piloted by the child, co-designed with the children. Never called a robot. | — | A specific robot (IP not used). |

No names, silhouettes, sounds or story framing are taken from existing games. Rules:
- The hero is never white with blue eyes.
- No "crash site" or spaceship.
- No PlayStation-style controller imagery.
- The hero is **never described as a robot** in any string, script, asset name or prompt (see §3.1 and AC-HERO-1). The only robot in the world is גלגלון (CH-03).

### 2.2 World structure

```
                 [R-08 מגדלור Sunny — English island, LTR]
                         ⇣ rope bridge (the language changes)
[R-05 מכרות הגביש]  [R-03 גבעת האותיות]  [R-04 אגם הערפל]
        \                |                 /
         \        [R-00 המחנה — HUB]       /
          \   (campfire · neighborhoods · trophy room)
[R-06 סדנת ברגית]  [R-07 מצפה הפליאה]  [R-09 מוסך גלגלון]
                \        |        /
            [R-10 עץ השקט]   [R-11 שביל הגחליליות — runner]
                     [R-12 חדר הגחליליות — K3 playroom]
```

- **The hub (R-00) is a physical place you walk around in** (3D/2.5D), not a menu.
- From the hub, **gates** (decorated arches) lead to regions.
- **A region** = a cluster of 6–10 short levels plus 1 region boss, connected by a small "field area" of 1–2 minutes of open exploration with secrets.

**Region map**

| ID | Name (Hebrew) | Subjects / skills | Signature visuals | Primary power-up |
|---|---|---|---|---|
| R-00 | **המחנה** (hub) | everything (review, show, build) | Warm campfire, tents, neighborhoods per child | — |
| R-03 | **גבעת האותיות** | K2: letter sounds, nikud, blending, phonological awareness. K1: roots, morphology, spelling | Hills of stone runes that glow when read | `MX-H01` rune lens, `MX-H02` syllable forge |
| R-04 | **אגם הערפל** | K1: nikud-fade → unpointed reading, comprehension, fluency | A lake whose fog *is* the nikud: reading clears it | `MX-H06` fog goggles |
| R-05 | **מכרות הגביש** | K2: counting, 10-frame, addition/subtraction to 20. K1: multiplication/division, place value to 10,000, fractions | Caves of crystal clusters, mine carts, rails | `MX-M01` number jet, `MX-M05` array hammer |
| R-06 | **סדנת ברגית** | geometry, measurement, patterns, area/perimeter, Build Missions | A huge workshop yard, scaffolding, cranes | `MX-M10` measuring rope |
| R-07 | **מצפה הפליאה** | science, world knowledge, experiments, geography of Israel | Observatory, greenhouses, map of Israel as a 3D diorama | `MX-X01` wonder telescope |
| R-08 | **Sunny Harbor** | English (oral for K2, phonics + vocab for K1) | Harbor and lighthouse in 09's `region.englishIsle` palette (purple-teal, §2.3); Sunny herself stays turquoise + yellow; English signs only (LTR) | `MX-E01` echo shell |
| R-09 | **מוסך גלגלון** | AI literacy, computational thinking, debugging | A workshop of bots, conveyor belts, screens | `MX-A01` remote controller |
| R-10 | **עץ השקט** | SEL: emotions, breathing, problem solving, perspective-taking | A giant tree with a treehouse, soft light, a creek | `MX-S02` lantern breath |
| R-11 | **שביל הגחליליות** | Mixed review in runner form (all subjects) | A twisting path through the forest at dusk | auto-run (`MX-R01`) |
| R-12 | **חדר הגחליליות** | K3: colors, shapes, animals, 1–3, body parts, first words | A soft playroom of toys (2.5D isometric) | — (open play) |

**Region unlocks**
- Regions open through **the story calendar or the family campfire level, whichever comes first** (the binding rule and the per-region table are in §10.4), together with each child's skill readiness. This follows 99 Nights' Missing Kids Board (R12) without ever gating learning behind a shared counter (02 §5.2 rule 5).
- A region a child is not ready for yet appears as "work in progress" with ברגית waving. It never shows a lock and never says "you're not good enough."

### 2.3 Visual tone and palette `⟲REVISE-AFTER-SPIKE`

- **Style:** toon shading with 3–4 steps, inverted-hull outline, bright saturated colors, blob shadows (E1b).
- **Shapes:**
  - Round and soft for "good" things.
  - Shadowlings are round too, never sharp-edged.
  - Every character can be drawn by K2 with 4–6 shapes (important for the coloring pages, 00 C10).
- **"Cozy-tense" evening palette**
  - Palette tokens: `palette.day` (warm cream-green), `palette.dusk` (orange-pink-purple), `palette.evening` (deep blue-purple with fire light).
  - Minimum luminance: `palette.dusk.minLuminance = 0.28` (relative luminance of the average frame). Never below it for K2/K1. For K3, `0.45`, and K3 sees dusk only, never evening (R8).
  - The fire is visible in every evening frame. Darkness never covers more than 40% of the screen area.
- **English island** (00 B2):
  - **Colour tokens are 09's (00 J1):** the island frame, signs and region tint use `region.englishIsle` (`#5F37C0 / #8C5CF0 / #A8F0E9`, 09 §3) and the subject colour `subject.english = grape`. The earlier value `palette.english = #19B7B0` is **withdrawn** (it conflicted with 09; 05 references tokens only). **Sunny (CH-02) keeps her character colours, turquoise + yellow** (§4.3): the island is purple-teal, the otter in it is turquoise-yellow, so she reads clearly against it. The "English frame" that wraps English moments outside the island (bottles, Sunny Says) also uses `region.englishIsle`.
  - All signs are in English (LTR), as pre-rendered textures.
  - Entry jingle `sfx.english.enter` (3 ukulele notes) and exit `sfx.english.exit` (the reverse).
- **Text in the world:** no 3D text (E1b).
  - A letter or syllable that appears in the world, such as a rune on a stone, is a **pre-rasterized glyph texture**. It is generated at build time from the approved font, with nikud verified (`tools/glyph-baker`), and is an image, not text.
  - Words and sentences appear only in the 2D HUD, positioned by projecting 3D→screen.

### 2.4 Lore bible (for content generators and scripts)

**World laws (never break)**
1. Light is made of knowledge. Reading, counting, saying a word in English, or understanding something new creates light. Light can't be bought.
2. Nobody dies, nobody is hurt, and nothing is lost permanently. Whatever was scattered can be found.
3. Shadowlings are not evil. They are cold, lonely and mischievous. Light warms them, and then they become friends.
4. The campfire never goes out. At worst it "naps" (glows quietly) and wakes happily.
5. The camp is a family. Everyone helps, and each person helps in a different way.
6. Machines (גלגלון and the bots) are machines. They help, compute and make mistakes. Only creatures feel.

**Places:** see §2.2. Each region has 3 "landmarks" that are visible from afar (sightline landmarks, R12) to guide exploration.

**Season 1 arc: "האור חוזר"**

S0 and S1 are 8 weekly chapters, each ending in a family boss (§9.3).

| Week | Chapter title | Story beat | Boss (a form of the Muddler) |
|---|---|---|---|
| 1 | "המחנה מתעורר" | Arrival, building the פנסון, the first friend | ענן ההפתעות (a cloud that sneezes out mixed-up objects) |
| 2 | "הגבעה שלוחשת" | Runes on the hill, the first sounds | מכונת הבלבול (swaps letters) |
| 3 | "בתוך המכרה" | Crystals, counting, carts | הקוסם הכפול (doubles and mixes up quantities) |
| 4 | "שמונה לילות" | Hanukkah (§13) | צל הסביבון (a spinning top that has lost its letters) |
| 5 | "הערפל עולה" | Fog on the lake, reading by context | ענן נמנום (a sleepy cloud that makes things blurry) |
| 6 | "ספינה מחו"ל" | The English island opens (R-08, week 6 from launch = the week of 2026-12-13; the gate opens on ~2026-12-20, 05 §15.1), Sunny arrives | **Mr. Mix-Up** = הבלגניסט (CH-04) in an island costume with a **separate en-US male voice** (§4.5; mixes up English words) |
| 7 | "הסדנה של ברגית" | Big building, the bridge to the island | צריח עקום (a tower built crooked) |
| 8 | "למה הבלגניסט מבולגן?" | Revealed: the Muddler sneezed because he was cold and alone outside the camp | The Muddler himself (puzzles with no "defeat"; he joins as the camp's MC) |

**Pre-island phase, "בקבוקים מהים" (weeks 1–5; requested by 05 §15.1, adopted).** From launch (2026-11-08) until R-08 opens (chapter 6, ~2026-12-20) English arrives only by sea: 2–3 times a week a bottle washes up at the camp shore (the daily-surprise slot, §5.5 / §5.3 `english_bottle`) with a recording of Sunny: one short song + 2–3 words, inside the English frame, 3–5 minutes, for every child (K3 with a parent). In the story the bottles are Sunny's letters from her ship on its way to the valley; the ship is the chapter-6 beat ("ספינה מחו"ל"). K1's English placement ("המכתב של Sunny", 05) arrives in week 3 as a bottle. Bottles stay in the mailbox forever (asset IDs in §7.4; dates in §13.2 S0).

**Lore discovered through play.** About 40 "pages from ניצי's diary" are hidden as secrets.
- K1 reads them (with nikud, according to her nikud-fade track).
- K2 hears them in voice-over.
- Each page is 2–4 sentences with one fact about the world plus one joke.

### 2.5 Tone of voice

- **Warm, curious, a little cheeky, never mocking.** Characters talk *about the content* (Calvert, `child_psychology_motivation.md` §3) and about what the child did.
- **No pressure:**
  - No "come back!"
  - No "you'll miss out."
  - No sad characters because a child left.
  - The goodbye line is fixed in spirit: "שיחקנו יופי. אני הולך לישון. נתראה מחר!"
- **Modeling frustration** (CBT): characters say "אוף, זה קשה לי" and then use a strategy.
- **Length:**
  - K3 lines: ≤4 words.
  - K2 lines: ≤10 words, 1–2 instructions.
  - K1 lines: ≤18 words.
- **Pacing:**
  - Voice-over at 0.9× natural speed for K2/K3.
  - A ≥250ms pause before a number or letter that is the content.

### 2.6 Humor guidelines (R4)

| Age | What's funny | Examples in the game | Frequency |
|---|---|---|---|
| K3 | Incongruity of objects and actions; peekaboo; funny sounds | A sock on the פנסון's head; a duck that "moos" and then laughs "לא! קְוָוה!"; a shadowling that sneezes bubbles | Every 30–60 seconds in the playroom, never during a learning question |
| K2 | Slapstick, nonsense words, misnaming, a little burp humor (burps, a big sneeze, but **no** poop or pee jokes), an animal that wears a hat backwards | ניצי calls a cat "כלב" and K2 corrects him (a learning moment!); the פנסון slips on a banana and does a perfect spin; the Muddler sneezes confetti | In every level transition, in feedback after a correct answer (not after a wrong one), in the entry gags of shadowlings |
| K1 | Hebrew wordplay, riddles, puns, mild irony, "breaking the fourth wall" in good taste | "למה הספר הלך לרופא? כי היו לו יותר מדי עמודים... כאב-עמוד!"; גלגלון: "חישבתי שיש 100% סיכוי לגשם. טעיתי. אני מכונה, לא מטאורולוג." | 1–2 jokes per episode, especially in diary pages and in גלגלון's lines |

**Humor rules (do / don't)**
- ✅ The character is the butt of the joke (ניצי, גלגלון, the Muddler), never the child.
- ✅ "A silly mistake by a character for the child to correct": the strongest humor plus protégé mechanic. At most 1 per level, and only on material the child has already mastered.
- ❌ Not funny:
  - a mistake the child made;
  - the body, weight, looks, gender, origin or religion;
  - war, soldiers, sirens or explosions (00 B8);
  - siblings ("your sister is slow").
- ❌ No sarcasm toward the child. No jump scares. No "frightening" surprises that look like humor.
- ❌ No humor on the item itself while the child is solving (R4). A joke lasts ≤2.0s and never blocks input.
- ✅ Humor through physics, such as squash & stretch and bounces, is preferred over verbal humor. It works for every age and in every language.

---

## 3. The hero: the פָּנָסוֹן, the avatar, and co-design (G4)

### 3.1 Concept

- **The פנסון** is a small lantern-being: a **toy-suit (חֲלִיפַת צַעֲצוּעַ) with a living spark (נִיצוֹץ חַי)** inside. The suit is a soft, stitched, lantern-shaped toy the child "wears"/pilots; the spark in its core is alive, warm and belongs to the child. The child's avatar sits inside a glass dome on its head, so the face is visible.
- **Naming rule (binding, parent's explicit request):** the hero is **never a robot**. Not "רובוט", not "robot", not "bot", not "mech", not "machine" — in strings, scripts, asset names, Rive artboard names, image-generation prompts or code comments. Allowed vocabulary: `toySuit`, `lanternHero`, "חליפת צעצוע", "פנסון", "הניצוץ החי". The gear/cog icon (⚙) is **reserved for the machine robot גלגלון** and for the "machine-made" tag on generated content (07 AC-06); it never appears on the פנסון or its parts. CI lint `hero-naming.lint` (AC-HERO-1).
  - Glossary note for writers: **הַנִּיצוֹצוֹת** (plural, CH-10) are the rescued friends; **הַנִּיצוֹץ** (singular, "הניצוץ של {name}") is the hero's living core. Scripts never use the bare plural for the hero's core.
- Design goals:
  - Original (no shape, color or detail from Astro Bot or any other game).
  - Lovable and very animatable; it reads as a **toy** (stitches, soft seams, a wind-up key as a decorative part option), not as a machine (no panels, rivets, antennas-as-sensors, LEDs, gears).
  - Drawable by K2.
- **The פנסון does not talk and is not AI.**
  - It has no voice lines and no opinions. It expresses itself only through animation and short sounds (a mini-flute, soft chimes; **never beeps**, beeps belong to גלגלון).
  - So it never blurs with the AI helper גלגלון (00 C7). The two must look different (see the CH-03 card): the פנסון is round, soft and warm with a lantern core, while גלגלון is boxy with a screen face and gears.
  - **The spark reacts to learning:** it brightens on each correct action (T0 "core juice"), pulses gently while the child thinks, and glows steadily at rest. It never dims below its resting glow (no loss, 00 C5).
- **Name options for the hero type** (the children can give *their* פנסון a personal name in onboarding):
  1. **פָּנָסוֹן** (default)
  2. נִיצוֹצוֹן
  3. זוֹהֲרוֹן

### 3.2 Hero anatomy (parts system)

| Slot | Options at launch | Notes |
|---|---|---|
| `chassis` | 3: `chassis.round` (K3 default, a wobbly ball), `chassis.explorer` (K2 default, egg-shaped with short legs), `chassis.ranger` (K1 default, slimmer, with a backpack) | Each chassis shares one skeleton of ≤30 bones (E1b) |
| `dome` | 4 shapes (round, square-ish, bubble, double) | The avatar's face shows inside, rendered as a 2D/texture projection onto the dome |
| `core` | 12 colors of the **living spark** (the lantern core) | Lights up with each correct answer (the core juice moment); never dims below rest |
| `topper` (was `antenna`) | 8 (spring, flower, star, leaf, flag, ear, horn, none) | Sways with secondary motion. Named "topper"/"כובעון" in code and strings, never "antenna", so the hero never reads as a machine |
| `arms` | 4 (short, mitten, gloves, grabber-paw) | The visual for interacting with power-ups. No claws, pistons or "mechanical" parts |
| `feet` | 4 (legs, rollers-in-socks, springs, suction-cup feet) | Changes only the walking animation, not speed or ability |
| `decal` | Unlimited: **drawings by the children** (scanned, see 3.4) plus 30 earned stickers | Placed on the chassis in 3 positions |
| `trail` | 10 (sparks, bubbles, leaves, notes, rainbow…) | Cosmetic, from the shop or milestones |

- **Rule:** no part changes gameplay ability (00 D3). Power-ups belong to the level, not to the hero.
- **3D pipeline:**
  - Spike: a CC0 base mesh with a **toy/plush silhouette** (e.g., a Quaternius CC0 "cute character" base re-skinned as a stitched lantern; no robot base, so the spike never ships robot proportions by accident) (E1b).
  - Final: a custom design (Meshy/Tripo → Blender ≤5k tris → rig). Prompts use "soft toy lantern, stitched fabric, glowing core"; the words "robot/mech/android" are banned in prompts (AC-HERO-1).
- **Mandatory animation clips (8):** `idle`, `walk`, `run`, `jump`, `celebrate`, `wave`, `found_it`, `oops_cute` (a cute stumble, never "sad").

### 3.3 Avatar: a child who looks like themselves without a photo (00 A4 self-like avatars; privacy per 00 E10 / OPD-10; Mii approach)

- The avatar is a 2D/2.5D face plus upper body (Rive/Skia; it is the face seen through the dome) and appears **full-body** in the hub (camp mode, walking next to the פנסון).
- **Face parts:**
  - 14 skin tones.
  - 34 hairstyles (straight, wavy, curly, afro, braids, ponytail, buzz, peyot, kippah as a separate item, headband, two buns…).
  - 10 hair colors.
  - 12 eyebrows, 16 eyes, 8 noses, 12 mouths.
  - A missing front tooth (on/off).
  - Freckles (3 densities), glasses (8), hearing aid, dimples.
- **Free placement and scaling (the Mii secret):** every part gets `offsetX/offsetY/scale/rotation` sliders within limits. This is what creates the likeness.
- **Process:**
  1. The parent builds the avatar behind the parent gate (~5 minutes) using a guided "יוצרים את X שלי" wizard with prompts such as "is her face rounder or longer?"
  2. The child gets to edit only the cosmetics (clothes, hair color for a party, accessories).
  3. Structural changes (skin tone, face shape) are parent-only, so a child doesn't "become someone else" by accident.
  4. K3 does not edit at all.
- **No photos, no face scanning, no image generation from a photo** (00 E10: no child photos in any image prompt unless OPD-10 consents; the avatar builder never uses the camera).
- **Avatar emotions:** `happy`, `proud`, `surprised`, `thinking`, `silly`, `sleepy`. The avatar never "cries" in the game.
- **Gender:** no item is gender-locked. The catalog is never filtered by gender (`gap_audit` §4.14).

### 3.4 Co-design with the children (G4)

1. **"מעבדת הפנסון"** (onboarding plus whenever they like): the child picks parts. The choices count as "design".
2. **"תכנון אמיתי" (offline mission):** the app prints (00 C10) an "outline of a פנסון" page.
   - The child colors and draws "what my פנסון has".
   - The parent photographs it (parent gate). The image is processed on the device: crop, background removal, palette quantization to 8 colors.
   - It becomes a `decal`, or a new `topper` from a template if the parent picks "turn into a topper".
   - No cloud processing is needed. If quality is poor, a filter turns it into a "sticker".
3. **"הצבעה משפחתית" (once a season):** ברגית presents 3 new part designs (prepared from the children's ideas: the parent notes "she suggested a pineapple topper" and the content pipeline produces 3 options). Each child votes with an emoji. All three are added to the catalog; the vote only decides the order they release in, so there's no losing.

---

## 4. Cast

### 4.0 Cast rules

- **Each character has:** one trait, one dominant color, one silhouette, one sentence of motto, and one skill family. This is Brawl Stars/Duolingo-style clarity (`content_world_phygital` §b1).
- **Character gender is fixed** (it determines how the character speaks about itself). The cast is balanced: 3 female, 3 male, 1 machine, plus a group.
- **Voices:**
  - All voices are pre-generated (00 E9). Voices are adult and warm. **No childlike voices** (00 E8a).
  - Each character gets a "sound signature" of 2 notes when entering a scene.
- **Characters never address the children to keep them in the game.** They never get sad because a child left, and never get angry.

### 4.1 Summary table

| ID | Name (default; alternatives) | Type | Gender (speech) | Color | Role | Skill family |
|---|---|---|---|---|---|---|
| CH-01 | **ניצי** (זיקי, גחלי) | Firefly, mascot | m | Warm yellow-orange | Main guide, scripted, everywhere | All (narration, guidance) |
| CH-02 | **Sunny** (Juniper, Pippa) | Sea otter, lighthouse keeper | f | Turquoise + yellow | **English only** | English |
| CH-03 | **גַּלְגַּלּוֹן** (בּוֹרְגִי, מְכוֹנִי) | Machine/robot, **clearly a machine** | m (grammatical) | Silver-steel + orange LED | AI helper: hints, AI-literacy | AI, thinking, hints |
| CH-04 | **הַבַּלְגָּנִיסְט** (מר בלבול, הקוסם בלגן) | Shadow magician (comic villain) | m | Purple-gray + gold hat | Weekly boss, season arc, becomes a friend. **On the English island he is "Mr. Mix-Up"** (island costume, separate en-US voice, §4.5) | Every subject (mixes them up) |
| CH-05 | **הַצְּלָלוֹנִים** | Round shadow blobs, a group | pl (each one m/f at random, fixed per individual) | Soft gray-purple → glowing when lit | "Enemies" that become friends | Every subject |
| CH-06 | **בְּרָגִית** (קוֹצִית, פַּטִּישׁוֹנָה) | Hedgehog engineer | f | Red-orange + blue overalls | Building, measuring, geometry, works in progress | Math (space), building |
| CH-07 | **דֻּבְדּוּבָה** (מיכמש, דובית) | Large soft bear | f | Pink-lilac | Calm, emotions, Tree of Quiet | SEL |
| CH-08 | **פְּרוֹפֶסוֹר לָמָּה** (למה-למה, זיקיתוש) | Chameleon scientist | m | Green that changes with "hypotheses" | Curiosity, experiments, science | Science, world |
| CH-09 | **משפחה** (parents, grandparents, family pet) | NPCs built by the parent | per person | — | "Camp elders", recorded voices | Personal support |
| CH-10 | **נִיצוֹצוֹת** (the rescued friends) | Species of light creatures | per individual | By species | The collectibles, filling the hub | By species (see 4.11) |
| CH-11 | **Pip** (Sunny's little crab) | Small crab, **silent** | — (no voice, no gender in speech) | Coral-orange + turquoise claws | Hero of K1's decodable English stories (05 §7); lives only on the English island | English reading (decodable from 05 stage S2: "Pip sat.") |

**Roles in the extra-subject and future-skills content (06/07; no new characters, F4):** 07's `COMPANION` is played by existing cast members — **ניצי (CH-01)** for AI literacy, computational thinking and self-direction (`AI.*`, `TK.*`, `SD.*`), **דובדובה (CH-07)** for every `SE.*` activity and the feelings check-in (`RIT-OPEN`), **ברגית (CH-06)** for `SD.MAKER`; 07's `ROBOT`/`BOTS` are **גלגלון (CH-03)** and his bots (the only machines, G4). 06 uses **פרופסור למה (CH-08)** for science (`S.*`) and ברגית for `S.TECH`. "Hats" in 07's scripts (detective hat, inventor hat) are **costume props** on those characters, never separate characters. The content pipeline's `characterId` field takes only `CH-*` ids (07 AC-33).

### 4.2 CH-01 ניצי (mascot)

- **Personality:** curious, optimistic, a little clumsy, a "big brother who gets things wrong on purpose". He loves questions and gets excited about small things.
- **Role:**
  - The scripted guide in every region.
  - Introduces each gimmick (the Ki beat).
  - Makes deliberate mistakes for the child to correct (protégé).
  - Leads Plan and Show&Tell.
  - Says goodnight.
  - **He is not AI** and is not labeled AI. He is a scripted story character (00 C7, gap D13).
- **Voice direction:** a warm adult tenor, energetic but not shrill, a natural Israeli accent. Smiling while speaking. Ends with a slight rise ("נו, בואו נבדוק!"). Pace 0.9×.
- **Visual brief:**
  - A round body (circle), 2 large almond-shaped eyes, a glowing tail-light at the back, 2 small wings, 2 antennae with dots on the ends.
  - **5 shapes** (K2 can draw him).
  - The tail light changes brightness with emotion: brighter when excited, pulsing gently when thinking.
- **Catchphrases (gendered variants):**
  - Opening: "!אוֹר בּוֹקֶר, {name}" ("good light-morning", our greeting; not gendered).
  - Before a gimmick: `{רוֹצֶה לִרְאוֹת קֶסֶם?|רוֹצָה לִרְאוֹת קֶסֶם?|רוֹצִים לִרְאוֹת קֶסֶם?}`
  - Deliberate mistake: "רֶגַע... זֶה 'מָ' אוֹ 'מִי'? אֲנִי מִתְבַּלְבֵּל. {תַּעֲזֹר לִי?|תַּעַזְרִי לִי?|תַּעַזְרוּ לִי?}"
  - After a rescue: "!עוֹד חָבֵר חָזַר הַבַּיְתָה"
  - Modeling frustration: "אוּף, זֶה קָשֶׁה לִי. נְשִׁימַת פָּנָס... עַכְשָׁו אֲנַסֶּה דֶּרֶךְ אַחֶרֶת."
  - Goodnight: "שִׂחַקְנוּ יֹפִי. אֲנִי הוֹלֵךְ לִישֹׁן. נִתְרָאֶה מָחָר!"
- **Do:** ask "how did you know?", get excited about the strategy, admit mistakes, give the child the last word.
- **Don't:**
  - Trait praise ("גאון", "חכם").
  - Superlatives ("מושלם").
  - Speaking during a solve (after 2.5s of silence, at most a "hmm…").
  - Asking the child to stay, getting sad, or yawning to make the child feel guilty.

### 4.3 CH-02 Sunny (English only)

- **Personality:** a sunny, musical sea otter who loves boats and songs. Patient. Uses lots of gestures.
- **Role:** the only voice of English (00 B2, "one person, one language"). She runs R-08 and appears in "English moments" in other regions, always with the entry jingle and turquoise frame.
- **Language rules:**
  - Never says a word in Hebrew.
  - For K2, the "sandwich" (EN → HE → EN) is done by **ניצי as the translator** standing next to her: Sunny: "Boat!" → ניצי: "סִירָה" → Sunny: "Boat!"
  - For K1, ניצי's share shrinks over time per `english.glossLevel` (3 → 0).
- **Voice direction:** a warm, natural US English female voice, slightly slower (0.85×), with clear consonants. Phonics sounds come from **recordings of a native speaker** (00 E9), never TTS.
- **Visual brief:** an otter in a yellow raincoat and a sailor's hat with a turquoise ribbon. Round shapes. Always holds an object (a shell, a book, a ukulele).
- **Catchphrases (English, no grammatical gender):**
  - "Ahoy, {name}!"
  - "Your turn!"
  - "Listen… and do!"
  - "You found it: *boat*!"
  - "Let's sing!"
  - "Bye-bye, see you tomorrow!"
- **Do:** gestures, repetition, songs, TPR, short sentences, 1–2 new words per encounter.
- **Don't:** correct pronunciation explicitly ("wrong!"), speak Hebrew, use a long sentence for K2, or play a sound that is not an approved native recording in phonics.

### 4.4 CH-03 גלגלון (AI helper, clearly a machine)

- **Personality:**
  - Precise, literal, slightly dry, funny by accident.
  - Proud to be a machine: "אני לא מרגיש, אני מחשב."
  - Makes mistakes (on purpose and sometimes for real) and asks to be checked.
- **Role:**
  1. The hint ladder in every mechanic (after "try first", 00 C7): hints, not answers.
  2. Runs R-09 (AI literacy).
  3. In K1 Mission Control, handles the "bots".
- **"Machine" rules (mandatory):**
  - Boxy shape.
  - A **screen** instead of a face, showing icons, "processing" dots and a gear animation.
  - Visible gears, a small "AI • מכונה" sticker on the chassis.
  - A slightly synthetic timbre (a light vocoder filter over the TTS).
  - Never says "I love you", "I'm happy", or "I'm sad".
  - Every hint opens with "לפי החישוב שלי…". Once a level he says "{תבדוק|תבדקי|תבדקו} אותי, אני יכול לטעות".
- **Voice direction:** a calm male voice, medium tempo, monotone with "beeps" between sentences. Light filter (`vo.filter.robot = bitcrush 12-bit + 3% chorus`).
- **Visual brief:**
  - A gray box on wheels, short telescopic arms, an antenna with an orange LED, a rectangular display face.
  - It must not look like the פנסון: no glass dome, no warm core, no round shape.
- **Catchphrases:**
  - "מְעַבֵּד... רֶמֶז מוּכָן."
  - "קֹדֶם {אַתָּה|אַתְּ|אַתֶּם}, אַחַר כָּךְ אֲנִי." (try first)
  - "זֶה רֶמֶז, לֹא תְּשׁוּבָה."
  - "{בָּדַקְתָּ|בָּדַקְתְּ|בְּדַקְתֶּם} אוֹתִי? מְצֻיָּן. מְכוֹנוֹת טוֹעוֹת."
  - Joke: "חִשַּׁבְתִּי שֶׁאֲנִי עָגֹל. מִסְתַּבֵּר שֶׁאֲנִי קֻפְסָה."
- **Do:** label himself as a machine, return the question to the child ("מה לדעתך הצעד הראשון?"), show the "thinking" visually, ask for verification.
- **Don't:**
  - Give a full answer before 2 attempts.
  - Appear before the child has tried.
  - Speak in free text: all of it is pre-generated and validated (00 C7).
  - Pretend to be a friend or a person.
  - Ask for personal information.

### 4.5 CH-04 הבלגניסט (comic villain who becomes a friend)

- **Personality:** a theatrical showman who loves surprises and attention. He jumbles because he's bored and lonely. He's not mean. He laughs loudly, sneezes, and gets offended in a funny way.
- **Role:**
  - The weekly boss (§9.3), each week in a different "magic form".
  - The face of the season arc.
  - In the finale he joins the camp as **MC of the evenings** and from then on presents the evening challenges.
  - **"Mr. Mix-Up" on the English island (binding, 05 §4 / 12 §6.4):** the boss of R-08 (chapter 6) and the "friend" voice of MX-E07 is הבלגניסט **in an island costume** (sailor's cap over the top hat, the smoke cloak tied back). He has a **second, separate en-US male voice** (12 voice bible: "חבר / Mr. Mix-Up"): one voice = one language (00 B2). **CH-04's Hebrew voice never speaks inside the island, and the Mr. Mix-Up voice never speaks outside it**; every `MIXUP` line is tagged `lang:'en'` and plays only with `englishContext=true` (05 AC24). He never says a Hebrew word on the island, not even his own catchphrases; his English catchphrase is "Is this a hat? …No? Oh! It's a cat!" (he "muddles", the children fix it). Asset ids under `en/char/mixup/*` (§7.4).
- **Voice direction:**
  - A theatrical male baritone, over-the-top "show" style, with circus-host rolled Rs.
  - Never threatening, never a low growl, no evil laugh.
  - His laugh is "חִי-חִי-חִי!", high and ridiculous.
- **Visual brief:**
  - Tall and thin (contrast with the round shadowlings), a cloak of soft smoke, a crooked golden top hat.
  - **Big, kind eyes**, a curly mustache.
  - No teeth, claws or red eyes.
  - When he "loses" he shrinks to a third of his size and laughs.
- **Catchphrases:**
  - "גְּבִירוֹתַי וְרַבּוֹתַי... בִּלְבּוּל!"
  - "אוֹי לֹא, {גִּלִּיתָ|גִּלִּית|גִּלִּיתֶם} אֶת הַטְּרִיק שֶׁלִּי!"
  - "אַפְּצִ'יייי! סְלִיחָה... זֶה קוֹרֶה לִי כְּשֶׁקַּר לִי."
  - Finale: "אֶפְשָׁר... לָשֶׁבֶת קְצָת לְיַד הַמְּדוּרָה?"
- **Do:** be funny, get defeated by learning, show a small soft spot (cold, lonely), be redeemed.
- **Don't:**
  - Appear in K3's playroom at all.
  - Taunt the child about mistakes.
  - Threaten to "take away" progress (he jumbles only material on the board in the scene, never the child's collection).

### 4.6 CH-05 the shadowlings

- **Description:**
  - Round, wobbly shadow blobs ("Jell-O made of smoke"), sized from a fist to a ball.
  - Two dot eyes, no mouth until they're lit.
  - When they get light they get a smile and a color, and turn into a **friend** (a ניצוץ of a matching species, or a shadowling-friend that stays in the camp).
- **Behavior:** they hide pieces, swap letters, stack crystals wrong, giggle ("קִיקִיקִי"), and run away clumsily.
- **"Strength" types (for the battle system, §9.1):**
  - `shadowling.tiny` (1 layer)
  - `shadowling.puffy` (3 layers)
  - `shadowling.stack` (a tower of 3 small ones, each needing a different item)
  - `shadowling.sleepy` (K3 only, wants a blanket)
- **Sound:** soft giggles, the "בּוֹפּ" of a bouncing ball, a little sneeze. No growl.
- **Don't:** approach the camera quickly, appear from the dark in K3 (K3 sees only `sleepy`), or have a pointy shape.

### 4.7 CH-06 ברגית (builder)

- **Personality:** a practical, enthusiastic hedgehog engineer, counter-stereotypical (female engineer). Loves "measure twice". Every building is a "project".
- **Role:**
  - Build Missions (`MX-M10`, `MX-M05`).
  - Works in progress (locked regions).
  - The shop and the decorations.
  - The seasonal part vote (3.4).
- **Voice:** an alto, quick, enthusiastic.
- **Visual:** spines like a ponytail, blue overalls, a measuring tape on her belt, a pencil behind her ear.
- **Catchphrases:**
  - "מוֹדְדִים פַּעֲמַיִם, בּוֹנִים פַּעַם אַחַת!"
  - "יֵשׁ לִי פְּרוֹיֶקְט, {אַתָּה אִתִּי?|אַתְּ אִתִּי?|אַתֶּם אִתִּי?}"
  - "אֲתַר בְּנִיָּה, בְּקָרוֹב כָּאן יִהְיֶה מַשֶּׁהוּ מַדְהִים." (a construction site, not a lock)
- **Don't:** say "you can't yet" or point to a skill gap. She says "עוד בבנייה" without blaming anyone.

### 4.8 CH-07 דובדובה (calm, SEL)

- **Personality:** a gentle, slow bear who listens a lot. "It's okay to feel."
- **Role:**
  - The feelings check-in (optional).
  - Lantern breathing (`MX-S02`).
  - The Tree of Quiet (R-10).
  - The always-available "need a break" button.
  - Team Talk (S06) for sibling conflicts in family mode.
- **Voice:** a deep, warm, slow alto (0.8×), lots of pauses.
- **Visual:** a large pink-lilac bear in a knitted cardigan, with a soft small lantern in her hand.
- **Catchphrases:**
  - "אֵיךְ {הַגּוּף שֶׁלְּךָ|הַגּוּף שֶׁלָּךְ|הַגּוּף שֶׁלָּכֶם} מַרְגִּישׁ עַכְשָׁו?"
  - "מֻתָּר לְהַרְגִּישׁ {מְתֻסְכָּל|מְתֻסְכֶּלֶת|מְתֻסְכָּלִים}."
  - "נְשִׁימָה אַחַת אִתִּי?"
  - "אֹמֶץ זֶה צַעַד קָטָן גַּם כְּשֶׁקְּצָת מְפַחֲדִים."
- **Don't (00 C8):** diagnose, ask "why are you sad?" repeatedly, store emotional data beyond the device, or present breathing as "a cure for worry".

### 4.9 CH-08 פרופסור למה (curiosity)

- **Personality:** a curious chameleon scientist who asks "what if?" and "how do we know?" He changes color when he changes his mind, which models intellectual humility ("I was wrong, my color changed!").
- **Role:** R-07, the experiment mechanics (`MX-X01`, `MX-X02`), Question Quests, and the "Mind-changer medal".
- **Voice:** a nasal, curious, quick tenor.
- **Visual:** a green chameleon in round glasses and a lab coat, with a long tail that curls into a question mark.
- **Catchphrases:**
  - "מָה {אַתָּה|אַתְּ|אַתֶּם} {חוֹשֵׁב|חוֹשֶׁבֶת|חוֹשְׁבִים} שֶׁיִּקְרֶה?"
  - "בּוֹאוּ נִבְדֹּק!"
  - "הוֹ! הֻפְתַּעְתִּי. הַצֶּבַע שֶׁלִּי הִשְׁתַּנָּה, שִׁנִּיתִי אֶת דַּעְתִּי!"
- **Don't:** state a "fact" without testing it, or mock a wrong prediction.

### 4.10 CH-09 family NPCs

- **"Camp elders":** every parent and grandparent the parent defines gets a simple avatar (same parts system) and a role:
  - Abba/Ima = "רָאשֵׁי הַמַּחֲנֶה".
  - Grandparents = "שׁוֹמְרֵי הַמִּגְדַּלּוֹר" or "הַגַּנָּנִים".
- **The family pet** (if there is one) becomes a camp creature that wanders the hub (the parent picks a species and color).
- **Voices: recorded only** (00 E9). Use sparingly:
  - the opening line on the first day;
  - "לילה טוב" at the end of the week;
  - milestone praise (T3 only);
  - a weekly "family clue" for the boss (optional).
  - Each recorded line plays at most 1 time per day. Clips: up to 12 per parent and 6 per grandparent.
  - **Format, storage and distribution are 12 §6.11's** (F9 GAP-67): recorded in the parent area ("הקולות שלנו"), `expo-audio` preset HIGH (AAC mono 48 kHz, `.m4a`), **each clip ≤10 s** (the recorder stops at 10 s), stored at `family/{familyId}/voice/{speaker}/{PV-id}.{variant}.{take}.m4a` in the private R2 bucket plus the device cache, listed in the `family-voice.json` manifest and shipped to every family device as a **family content pack** through the normal pack catalog (11 §4.4). **Re-recording replaces**: a new take is uploaded and the manifest's chosen take moves to it; the old take is deleted within 24 h. Only clips with `review:"parent_ok"` ever play. Nothing is sent to any AI vendor (00 E6a).
- **The child's name:** recorded by the parent (00 E9). It is never TTS.
- **Reducing self-focus** (Kucirkova): in 1 of every 3 stories, the hero of the scene is a family member ("Grandma Rina found a lost friend in the garden, help her count").
- **Do:** treat family members as heroes who help.
- **Don't:**
  - Use a family voice for discipline or to stop the game.
  - Any line in the voice of a family member that wasn't recorded by them.

### 4.11 CH-10 the ניצוצות (rescued friends) and their species

Each rescued friend is a **curriculum item** (content ID). In the hub it says its item when poked, so poking is retrieval practice.

| Species | Hebrew name | Carries | Shape | Sound |
|---|---|---|---|---|
| `sparkling.letter` | **אוֹתִיּוֹנִים** | A Hebrew letter or syllable (with nikud) | A drop shape with the letter glowing on the belly (pre-baked glyph texture) | Says its sound ("/m/… מָ!") |
| `sparkling.number` | **מִסְפָּרוֹנִים** | A number, fact or quantity | A crystal cube with N dots or a numeral | Counts itself (from the numerals lookup table, 00 B4) |
| `sparkling.word` | **Wordlings** | An English word (lives only on the island!) | A star with a picture of the word | Says the word in English (Sunny's voice) |
| `sparkling.wonder` | **פְּלִיאוֹנִים** | A science or world fact | A spiral shell | "{הַאִם יָדַעְתָּ|הַאִם יָדַעַתְּ|הַאִם יְדַעְתֶּם}…?" |
| `sparkling.feeling` | **רִגְשׁוֹנִים** | An emotion word | A soft cloud with a facial expression | Names the feeling plus a body cue |
| `sparkling.bot` | **בּוֹטוֹנִים** | A CT/AI concept | A tiny cube with an LED | Beeps plus a short line |
| `sparkling.toy` (K3) | **צַעֲצוּעוֹנִים** | Color, shape, animal, 1–3 | Big, soft, round shapes | Animal sound or the color's name |
| `vip.*` | **VIPs** | Story characters (redeemed shadowlings, cameos of family members as dolls) | Unique | Unique |

---

### 4.11a CH-11 Pip (silent crab, English island only; requested by 05)

- **Why he exists:** his name is decodable from 05's phonics stage S2 (s, a, t, p, i, n), so K1's very first English story can be "Pip sat." He is Sunny's pet-helper and the recurring hero of the ship's-log stories (MX-E06).
- **Rules:**
  - **Pip never speaks** (no voice lines in any language), so OPOL (00 B2) can never be broken by him. He communicates with claw gestures, bubbles and a 2-note "click-click" sound signature.
  - He appears **only** inside the English frame (R-08, bottles, English moments). Never in Hebrew scenes.
  - He is drawable by K2 with 4 shapes (a round body, two claws, two eye-stalks).
  - In 05's decodable sentences he does small things that end in a tiny change of state ("Pip got a pig.", "Fill the bag, Pip!"), so each page has a visible outcome in the scene.
- **Visual brief:** a small coral-orange crab with oversized turquoise claws, a tiny sailor's cap matching Sunny's, big round eyes. No teeth, no pincers that look sharp.
- **Don't:** give him Hebrew, give him a gendered voice, use him outside the English frame, or make him the butt of "slow/silly" jokes about reading.

### 4.12 Companions (pets): "גוזלי אור" (00 D4)

> **Precedence:** `02-economy-progression.md` §8 is the canonical owner of the pet catalog, stages, thresholds, treats and friendship. This section keeps the game-design role of the pet and mirrors 02's rules so the two never diverge.

- **One species per learning domain (02 §8.1), 6 at launch:** `fox_letters` שועלון האותיות (Hebrew reading), `dragon_numbers` דרקונון המספרים (math), `parrot_english` Poppy the parrot (English; **speaks only English**, 00 B2, voiced from Sunny's bank), `turtle_world` צב החוקר (world & science), `octopus_thinking` תמנון החידות (thinking/AI literacy), `puppy_heart` כלבלב הלב (SEL). No "gear robot" pet: the only machine in the cast is גלגלון.
- Every child has **1 active companion** (the one that follows the פנסון) and a collection in "גן הגוזלים". **Eggs** (02 §8.1): the first egg at onboarding (K1 chooses 1 of 2 shown; K2 receives the egg of his first domain); K1's second egg at level 4 or when גן הגוזלים opens, whichever is later; every further egg when a new domain starts. The egg's contents are known in advance (silhouette, D2).
- **5 stages, deterministic, by mastery only (02 §8.2):** `egg` → `baby` → `young` → `grown` → `legendary`. Thresholds are counts of **mastered skills in the pet's domain** (K1: first `proficient` / 3 / 8 / 15 mastered + 3 retained; K2: first `proficient` / 2 / 5 / 9). "Mastered" is 08's `master` level (D9). A completed species set in the collection room gives the pet an **accessory**, never a stage (02 §5.4). Care never affects evolution.
- **Never dies, never gets sick, is never hungry.**
  - Feeding = affection, not a need: 1 free treat per episode, special treats for coins (02 §8.3), at most 3 "eaten" per day, the rest wait in the pantry.
  - If not fed, the pet is fine. After an absence it is asleep or playing, and wakes happily ("הנה את! בואי נראה מה בנית"). Never "התגעגעתי".
  - "Friendship" 1–10 only goes up (3 care points/day max) and unlocks pet emotes ("tricks").
- **In-game role:**
  - Follows the פנסון in levels.
  - **Points with its nose at a nearby secret** (a 3-second hint, after 20 seconds of no progress or 45 seconds without a secret found; 02 `petNoseHintSeconds`).
  - Cheers on a correct answer.
  - Sleeps at the end of the episode.
- **K3:** the companion is "פּוּפִּי", a soft round bunny (02 `ארנבון`; the name is 01's, the species is 02's). It reacts to every touch with a laugh or a sound and doesn't evolve. Instead, it gets a hat or a color at every sticker milestone (every album page completed).
- **Holidays:** pets get holiday costumes, not new species (02 §8.1).

---

## 5. Loops

### 5.1 Loop hierarchy (shared by everyone, different parameters)

| Loop | Length | What happens | Reward/feedback |
|---|---|---|---|
| **Micro (action)** | 2–10 s | Learning action → game action (jump on the right syllable, fly exactly 7, measure a beam) | T0 juice (≤0.8s), light into the core, rising note on the pentatonic scale |
| **Beat (Kishōtenketsu beat)** | 0:45–2:00 | One beat of the level's gimmick (Ki/Shō/Ten/Ketsu) | Checkpoint, a friend rescued |
| **Level** | K3 1–3 min, K2 3–6 min, K1 6–9 min | One gimmick, 4 beats, 3–7 friends, 2–5 pieces, secrets | T2 at the end, friends move into the hub |
| **Episode = "a day at camp"** | K3 3–5, K2 8–12, K1 15–20 min (00 C3) | Plan → morning (review) → level/s → camp (show + build) → short evening (campfire) → goodnight + offline mission | Mastery-based coin receipt, sometimes a treasure, a diary page |
| **Week** | Sun–Fri (Shabbat is rest) | Family weekly goal plus "ערב מדורה משפחתי" with the weekly boss | Gems (K1), family fire level-up, family treasure |
| **Season** | 6–9 weeks, tied to the calendar (§13) | Story chapter, region, seasonal event, adventure album | A season album that never expires, a season certificate printed for home |

### 5.2 The Plan–Play–Show&Tell ritual (Plan-Do-Review, `future_skills` §5.1)

Built into every episode, in the hub.

| Step | K3 | K2 | K1 |
|---|---|---|---|
| **Plan** (≤40s) | The parent picks one of 3 picture "doors" together with the child ("what do we feel like today?") | Lost Friends Board: picks 1 of 2 levels (silhouettes of the friends waiting there); predicts "קַל 🌶 / מְאַתְגֵּר 🌶🌶" | Picks a quest + an optional "flame" (modifier: Stretch / Cozy / Mystery) + **a goal and prediction** ("I'll rescue 6 out of 7", "I'll try a new strategy") |
| **Play** | Peekaboo level | Level + camp | 1–2 levels / challenge + camp + evening |
| **Show** (in the camp, ≤60s) | The friends arrive in the playroom; the parent asks "מה מצאת?" (a prompt on screen) | The rescued friends arrive, each says its item; the child "introduces" them to ניצי by poking | Same, plus **calibration**: prediction vs. what happened → "Calibration Star" for an accurate prediction (not for success!) |
| **Tell** (≤40s) | — | 1 tap: 😀/😐/😮‍💨 "איך היה?" + "מה עזר?" (3 tool icons) | One-line sentence "בפעם הבאה אני…" (chips or 15s voice recording, kept on device) + optional 30s Show&Tell recording for the parent (MX-T05) |

### 5.3 Episode structure minute by minute (default template)

The adaptive engine fills the content slots. These are the default durations. `EpisodeTemplate` (Appendix A) makes them configurable.

#### K2 (5.5): `episode.k2.default`, 10:00 (range 8–12)

| Time | Segment | What happens | Content (engine) | Notes |
|---|---|---|---|---|
| 0:00–0:40 | **Arrival + Plan** | The פנסון wakes up in the tent, ניצי says "אור בוקר". Optional feelings check-in (MX-S01, 1 tap, ≤20 s, skippable with "לא רוצה להגיד"; **band G: every other day, in the first episode of that day only** — 07 §2.4 `RIT-OPEN` follows this cadence). Lost Friends Board → choice of 2 levels | — | Voice only, big pictures |
| 0:40–2:00 | **Warm-up review: "בוקר טוב לחברים"** | Walk through the neighborhood: 3–4 friends rescued earlier are "asleep". Wake each one by doing its item ("which friend says /b/?", tap the right one; "show me 4 crystals") | 3–4 due items from the SRS ladder (gap D9) | Short, easy (≥90% expected) |
| 2:00–2:45 | **Level: Ki (introduce)** | Entering the level through the gate. ניצי demonstrates the gimmick with the new item, in a safe place | New skill: 1 item, worked example | No mistake possible (an action with a single target) |
| 2:45–4:15 | **Level: Shō (develop)** | The gimmick plus movement/height. 2 friends rescued. **Movement moment** (MV-*) ≤45s | 4–6 items of the new skill | Hint after 2 errors (gap D10) |
| 4:15–5:45 | **Level: Ten (twist)** | A twist that requires understanding (the gate swaps the order, a shadowling scrambles). 1–2 friends + interleaved practice | 4–6 items: 60% new, 40% older mixed | Secret room available |
| 5:45–6:30 | **Level: Ketsu (conclude)** | An easy celebratory finish: running to the gate, the last friend + T2 | 1–2 easy items | Lands on success ("land the plane") |
| 6:30–8:00 | **Camp: Show + Build** | The friends move in (T1 each, quick). Coin receipt. Place one decoration or build mission (`MX-M10` simple), feed the pet | Optional build mission: 2–3 counting/shape items | The coins appear *after* the action (00 D1, Lepper) |
| 8:00–9:20 | **Short evening by the campfire** | 3–4 sleepy shadowlings come to warm themselves. Light each one with an item (review) → they become friends | 4–5 mixed review items | Calm, dusk palette, not dark |
| 9:20–10:00 | **Goodnight + Tell + offline mission** | Tell (1 tap). The friends yawn and go to sleep. Offline mission card (picture + voice). ניצי: "שיחקנו יופי…" → the app sleeps | — | No autoplay; reopening goes through the parent gate (00, `child_psychology` §7D) |

**Optional segments (any child, inserted by the planner, each ≤5:00, inside the episode budget):**
- `english_bottle` (S0 only, until R-08 opens; §13.2): a bottle from Sunny in the daily-surprise slot, 3–5 min inside the English frame.
- `ritual_mid` (07 `RIT-MID`, "דקת כלי"): 60–90s for K2 on 3 of 5 days, 90–120s for K1 every day; counts as learning time; one tool card from the unified tool belt (MX-T01). Skippable with one tap ("קדימה למשחק"); two skips in a row shorten it to a 5s version automatically (07 §12).
- `weather_tap` (06 §4.5): in the first episode of the day, one tap "what's the weather outside?" on the hub's family weather station; feeds the shared monthly chart. ≤10s, no reward.
- 07's `RIT-OPEN` = this document's Arrival + Plan (feelings check-in ≤20s + plan ≤15s); `RIT-CLOSE` = Goodnight + Tell. Open + close together ≤10% of episode time (07 AC-12; stricter than the 15% celebration budget and measured separately).

#### K1 (8.5): `episode.k1.default`, 18:00 (range 15–20)

| Time | Segment | What happens | Content |
|---|---|---|---|
| 0:00–1:00 | **Arrival + Plan** | Feelings check-in (MX-S01: area → word of 9 → intensity 1–3, ≤20 s, **every episode for band S**, skippable, 07 §2.4). Lost Friends Board (map of regions with silhouettes). Choice of quest + flame (Cozy/Stretch/Mystery) + goal and prediction | — |
| 1:00–3:00 | **Warm-up review: "סיור בוקר"** | A "guard patrol" in the neighborhood: wake 6–8 friends (fast fact fluency only after mastery, gap D11). Optional time-trial of the patrol against *her own* record (ghost) | 6–8 due items, mixed between subjects |
| 3:00–3:45 | **Level A: Ki** | Introduce the gimmick (worked example + "your turn") | New skill |
| 3:45–5:45 | **Level A: Shō** | Develop + movement moment (≤60s, optional seated) | 6–8 items |
| 5:45–7:45 | **Level A: Ten** | A twist + a "struggle-first" puzzle (in Stretch: 60–90s without a hint, then a hint ladder) | 5–7 items, 60/40 new/mixed |
| 7:45–8:30 | **Level A: Ketsu** | Finish + secret exit (sometimes) | 1–2 |
| 8:30–12:00 | **Block B (one of the following, by plan/engine)** | (a) a short second level in another subject (interleaving); (b) a region boss (§9.2); (c) a hidden secret level; (d) a runner chase (MX-R01) | 8–12 mixed items |
| 12:00–14:30 | **Camp: Show + Build + Shop** | Friends move in, coins/gems receipt, Build Mission (area/perimeter/arrays), decorating the neighborhood, collection room, shop/wishlist | 3–5 build items |
| 14:30–16:30 | **Evening at the campfire** | Defense: a wave of shadowlings (MX-B02), mixed review. If siblings are around, this is the natural co-op point | 6–10 mixed |
| 16:30–18:00 | **Show&Tell + goodnight** | Prediction vs. result, "next time I'll…", optional recording, offline mission, diary page (if found), goodnight | — |

#### K3 (2.5): `episode.k3.lap`, 4:00 (range 3–5), lap mode only (§15.3)

| Time | Segment | What happens |
|---|---|---|
| 0:00–0:30 | **Hello** | The parent opens (gate). פופי peeks out of a box: "!קוּקוּ" + the child's name (recorded by the parent). Optional parent-led check-in (4 faces, nothing stored; band T, 07 §2.4) |
| 0:30–1:00 | **Plan** | 3 big doors with pictures (animals / colors / music). The child points, the parent taps, or the child taps |
| 1:00–2:40 | **Peekaboo level** | 3 friends hiding in a playroom diorama (2.5D isometric, fixed camera). Tap a bush → it shakes → a friend appears and says its item ("!פָּרָה! מוּ!" / "!אָדֹם!" / "!שְׁנַיִם!"). **There are no wrong answers**: every tap does something fun. A parent prompt on screen: "שאלו: איפה הפרה?" |
| 2:40–3:20 | **"Copy me" (MV-01)** | A 30–40s song with movements (clap, arms up, stomp), parent with the child, no sensing |
| 3:20–3:45 | **The friends come home** | The friends hop into the playroom, a sticker in the album (T2 sensory, short) |
| 3:45–4:00 | **Goodnight** | The friends go to sleep under a blanket, a lullaby of 2 notes. A parent prompt: "שאלו: מה מצאת היום?" → the app sleeps |

### 5.4 Meta loop per child

| | K3 | K2 | K1 |
|---|---|---|---|
| **Long-term "map"** | Sticker album (12 pages of 6, +1 seasonal page per season, +2 pages/month; 02 §2.4) + a playroom that fills with toy friends | Lost Friends Board + hub neighborhood that fills + collection room + Adventure Album (season track, **20 tiers**, opens at level 6; 02 §9) | Same plus Adventure Album (season track, **30 tiers**, opens at level 3, never expires) |
| **Collections** | Stickers, toy friends | Friends by species, pieces (mosaic), pets (one per domain) | Same plus diary pages, emotes, hero parts, seasonal mosaic |
| **Currencies** (00 D5; all numbers in 02) | None (stickers only; hidden XP for the parent) | Coins (small "effort chunks", ≤90) + chests (wood/gold) + 2 jars (spend/save) + "give" button + pet | Coins + gems + chests (wood/silver/gold/legendary) + shop + 3 jars + season track |
| **Hub milestones** | Every 5 toy friends → a new toy in the playroom (sound/light) | Building thresholds (§10.2) | Same plus her own buildings (design mode) |
| **Challenge** | None | 🌶🌶 optional | Stretch / Mystery flames, struggle-first, time-trial against herself, Mentor Track (guide mode) |
| **Weekly** | Participates in the family evening (as "lighter") | Participates + personal goal | Participates + personal goal + Mentor Track |

### 5.5 Daily surprise (Animal Crossing done right, R10)

- At each entry (max 1/day), there is **one** new thing in the camp. Examples:
  - a letter from a friend in the mailbox (a diary page / a riddle);
  - a visiting shadowling friend who asks for a small favor (an optional bonus item);
  - a new item in ברגית's catalog;
  - a "lost" friend sitting on the camp's roof (an easy secret to find).
- **They accumulate:** after 3 days away, there are 3 letters waiting (no FOMO, nothing is lost).
- **There is no "you came 5 days in a row" reward** and no daily login reward (gap D5, Radesky).

---

## 6. Level structure

### 6.1 The level template: Kishōtenketsu + secrets

`LV-template` structure (each level is an instance; levels are assembled from **chunks** built by the designer + an engine-supplied content slot):

| Beat | Goal | Space | Friends | Pieces | Content |
|---|---|---|---|---|---|
| **Ki** (introduce) | Teach the gimmick safely | 1 small area, no possible failure (e.g., a single target) | 0–1 (demonstration) | 0 | 1 new item, worked example |
| **Shō** (develop) | The gimmick + movement/height/combination | 1–2 areas | 2 (1 on the path, 1 slightly off the path) | 1–2 | 4–8 items |
| **Ten** (twist) | Change that requires understanding | 1 area + a secret room | 1–2 + 1 secret | 1–2 | 4–7 items (interleaved) |
| **Ketsu** (conclude) | Easy finish and celebration | Runway to the exit gate | 1 (the last, easy) | 0–1 | 1–2 |

**Counts per age (default)**

| | K3 | K2 | K1 |
|---|---|---|---|
| Friends per level | 3 (all on the path) | 5 (3 on the path, 2 secret) | 7 (4 on the path, 3 secret) |
| Pieces | 0 | 3 | 5 |
| Secret exit (to a hidden level) | — | 10% of levels | 15% of levels |
| Checkpoints | Unnecessary (no falling) | Every 20–30s of play | Every 30–45s |
| Level length | 1:00–2:40 | 3:00–6:00 | 6:00–9:00 |

**Rule of "no level without its learning".** Friends on the path are rescued only through a learning action. A **secret friend** is found through curiosity (looking behind a waterfall, jumping on a strange stone, poking a talking flower) **plus** an easy learning action at the moment of the rescue. This keeps P-1 while also rewarding curiosity (`astrobot` §Q4: Gopnik).

### 6.2 The grammar of secrets (G1)

Every level must include ≥4 of the following 8 types (CI lint on the level file):
1. **Glint:** a shimmer every 3 seconds on a hidden object, within 6 tiles of the path.
2. **Sound tell:** a soft giggle or note from a hidden direction, spatial (`expo-audio` pan/volume by distance, E1b).
3. **Visible friend behind a gate:** the purest information gap. You see the friend, and the gate opens with the skill.
4. **Odd tile:** a tile with a different color; jumping or tapping it reveals a ledge.
5. **Talking background object:** a flower or rock that says a riddle when poked.
6. **Off-path ledge:** a ledge ≤3 tiles off the main path, reachable with the power-up.
7. **Breakable decoration:** a crate or bush that bursts open into sparks plus a piece.
8. **Pet's nose:** the pet points (after 20s without progress or 45s without a secret found).

**Density:** ≥1 interactable per 6×6 tiles, and ≥1 reward per 30s of walking (P-2).
**Breadcrumbs:** "sparks" (`spark`) arranged in a line lead to the next goal. Collecting them gives light to the core (an in-episode resource, see §11.1). They are not a currency.

### 6.3 Controls by age (E1b) `⟲REVISE-AFTER-SPIKE`

| | K3 | K2 | K1 |
|---|---|---|---|
| Movement | **Tap anywhere → the פנסון rolls there** (path find). Dragging = "follow the finger". Ignore a second touch, debounce of 300ms | Tap-to-move + **big jump button** (bottom right, ≥15 mm) or tap on the פנסון = jump | Tap-to-move **or** floating joystick (appears under the left thumb, deadzone 15%) + jump/action buttons on the right |
| Camera | Fixed (2.5D isometric diorama) | Automatic follow (pitch 40°, auto yaw, no rotation) | Automatic + optional swipe-to-orbit with auto recenter after 2s |
| Interaction | Tap on an object = go and interact (auto) | Tap on an object; drag-and-snap in mechanics (snap radius 20 mm) | Full: drag, trace, type a short word |
| Falling into a gap | Impossible (no gaps) | Bounces back to the edge with a cute animation (`oops_cute`), 0.6s, from the nearest checkpoint | Same, 0.8s |
| Auto-run mini-levels | — | Tap-to-jump (Super Mario Run model) | Tap-to-jump + swipe to change lane |
| Tilt | — | — | A bonus only in selected mini-levels (MV-07), never required |
| Min target size (physical, gap §4.7) | ≥20 mm (≈120pt iPhone / ≈104pt iPad) | ≥15 mm (≈90/80pt) | ≥10 mm (≈60/52pt) |
| Handedness | — | Toggle to mirror the jump button | Toggle to mirror joystick/buttons |

- The world is **not mirrored in RTL** (E1b). Only the 2D HUD is RTL. Number lines, equations and math expressions inside the HUD are always LTR (00 B9 (d)); coding/arrow strips for band G flow RTL (00 B9 (a)); movement arrows mean absolute screen direction (00 B9 (b)).
- **Game controller (00 E3):** optional after the vertical slice (C4); stick = move, X = jump, haptics → rumble. Never required by any mechanic; every control scheme above stays fully playable by touch (AC-CTRL-1).
- **"Forward" progression in the hub and regions is right to left.** For example, the first gate is on the right, and progress on the Lost Friends Board runs right to left (Apple HIG RTL).

### 6.4 HUD (2D, RTL, Reanimated)

- **Top right:** the friend jar in this level (silhouettes `●●○○○`; no numbers for K2/K3).
- **Top left:** a pause button with דובדובה (pause / "I need a break" / back to camp) + "שמע שוב" (replay the instruction).
- **Bottom:** prompt bubble (K1: short text + 🔊; K2/K3: icon + voice only).
- **No** score, no timer (except an opt-in K1 time-trial against herself), and no coins in the HUD during a level. Coins show only in the camp (an informational receipt).

### 6.5 Failure handling (no-fail, gap D10, 00 C2)

| Event | K3 | K2 | K1 |
|---|---|---|---|
| Wrong answer | No such thing (everything works) | A funny, informative bounce (the wrong stone "sinks" with a בּוֹפּ and comes back), no loss | Same + "עוד לא" |
| 2 errors in a row on the same item | — | Hint from גלגלון (step 1 of the ladder) + step down (fewer distractors) | Same; hint ladder of 3 steps |
| 3 errors | — | Worked example by ניצי ("בוא נעשה יחד") + an easier item; **never more than 3 in a row** | Worked example / switch to another tool |
| <60% over 8 items | — | The engine lowers a level, ניצי "takes a lantern breath" with the child, the next item is easy for sure | Same + an offer to switch to Cozy flame |
| Falling into a gap / bump | — | 0.6s and back to the checkpoint, no loss of friends | Same |
| "Stuck" (60s without progress) | — | The pet points + a glowing path | Same, after 90s |

### 6.6 Level schema (code)

```ts
// packages/content/src/schema/level.ts
export type Beat = 'ki' | 'sho' | 'ten' | 'ketsu';
export interface LevelTemplate {
  id: string;                    // 'lv-r05-numberjet-01'
  region: RegionId;              // 'R-05'
  gimmick: MechanicId;           // 'MX-M01' (exactly one; CI fails if >1 new gimmick)
  mode?: 'normal'|'placement'|'benchmark'|'review'|'boss'|'event'; // default 'normal'. Set per instance by the planner (08 §1.10 Session.mode / EpisodePlan.mode).
                                 // 'placement'|'benchmark' → 02 §2.7 neverEarns (no coins/XP/chests); rescues still count for the collection (G2); 08 §3.6 UI rules (no right/wrong feel).
  ages: ('k1'|'k2'|'k3')[];
  grid: { w: number; h: number; tileSize: number; elevations: number[][] }; // camera-agnostic
  chunks: Array<{ beat: Beat; chunkId: string; origin: [number, number] }>;
  friendSlots: Array<{ beat: Beat; pos: [number,number,number]; kind: 'path'|'secret'; secretType?: SecretType; itemSlot: ItemSlotRef }>;
  pieceSlots: Array<{ pos: [number,number,number]; secretType: SecretType }>;
  checkpoints: Array<[number,number,number]>;
  movementMoment?: { beat: 'sho'|'ten'; moment: MovementId };
  secretExit?: { pos: [number,number,number]; to: string };
  itemSlots: Record<ItemSlotRef, { skill: SkillId; role: 'new'|'review'|'mixed'; difficultyBand: 1|2|3|4|5 }>;
  durationTargetSec: { k1?: [number,number]; k2?: [number,number]; k3?: [number,number] };
  landmarks: Array<{ pos: [number,number,number]; assetId: string }>; // ≥1 visible from spawn
}
export type SecretType = 'glint'|'sound'|'visibleBehindGate'|'oddTile'|'talkingObject'|'offPathLedge'|'breakable'|'petNose';
```

**Production budget (realistic for a solo build):**
- **Year 1:** 12 chunks per gimmick × 24 gimmicks = a combinatorial library.
- **MVP:** 3 regions × 6 levels.
- Levels are assembled by `tools/level-assembler` (seed + gimmick + chunks). The designer approves before release (E10).
- Content (which letter/number) is injected at runtime from the engine, so one level can be replayed with different content.

---

## 7. Learning mechanics catalog (gimmicks)

### 7.0 Rules for the catalog

- **Each mechanic is a "verb"** that can carry a level (G3). It also has a **compact form** for evening, battle and review, so one item bank powers many modes (Blooket/Gimkit, `coop_economy` §1).
- **Generic fields:**
  - `ID` / `code`
  - target ages
  - skills
  - input
  - rules
  - **intrinsic integration** (how the learning *is* the action)
  - feedback
  - difficulty knobs (`knobs`)
  - failure handling
  - movement form (if any)
- **Asobi's reuse rule:** a mechanic that reappears in a third region must add a new twist (a `variant` field in the content).
- **Response time (RT):** RT is used only in K1 fluency mechanics after mastery (gap D11). For K2/K3, RT is not measured in the update.
- **Distractors:** distractors come from error catalogs in code, and math answers are computed in code (00 E8).

### 7.1 Master table

| ID | Name (Hebrew) | `code` | Ages | Domain | Primary input |
|---|---|---|---|---|---|
| MX-M01 | סילון המספרים | `number-jet` | K2, K1 | Counting, addition, skip counting, multiplication | tap/drag (choosing N), jump |
| MX-M02 | ניצוצות במבט | `subitize-sparks` | K3, K2 | Subitizing 1–6, 10-frame | tap |
| MX-M03 | להאכיל את הגוזל | `give-n` | K3, K2 | Give-N, cardinality | tap / drag |
| MX-M04 | נפחיית העשר | `ten-forge` | K2 | Composing/decomposing 10, number bonds | drag-snap |
| MX-M05 | פטיש המערכים | `array-hammer` | K1 | Multiplication as an array, area | drag rows×columns |
| MX-M06 | גשר הפסים | `bar-bridge` | K1 | Word problems with a bar model | drag pieces of a bar |
| MX-M07 | קסמי עובדות | `fact-spells` | K1 | Fluency of × ÷ facts (after mastery) | tap/keypad |
| MX-M08 | עגלות ערך המקום | `place-value-carts` | K1 | Place value up to 10,000 | drag/tap |
| MX-M09 | חלוקה הוגנת | `fair-share` | K1 (K2 light) | Division, fractions, equal sharing | drag |
| MX-M10 | חבל המדידה | `measure-rope` | K2, K1 | Length measurement (non-standard → cm), perimeter | drag/stretch |
| MX-M11 | שביל הדפוסים | `pattern-path` | K3, K2, K1 | Patterns AB/ABC/growing | jump on tile / tap |
| MX-M12 | קו המספרים בקפיצות | `line-hop` | K2, K1 | Number line 0–20 / 0–1000 / fractions | jump to a position (MV) / drag |
| MX-H01 | עדשת הרונות | `rune-lens` | K2 | Letter–sound | tap |
| MX-H02 | נפחיית ההברות | `syllable-forge` | K2, K1 | Letter + vowel → syllable (blending) | drag-snap |
| MX-H03 | גשר המילים | `word-bridge` | K2, K1 | Blending syllables → word, decoding | jump on stones in order |
| MX-H04 | ציד הצליל הראשון | `first-sound-hunt` | K3 (vocabulary), K2 | Phonological awareness, initial sound | magnet/tap |
| MX-H05 | תופי החריזה | `rhyme-drums` | K2 | Rhyme | tap drums / jump |
| MX-H06 | משקפת הערפל | `fog-goggles` | K1 | Nikud-fade, unpointed reading by context | tap a word / choose a meaning |
| MX-H07 | מגילת הדרכים | `story-scroll` | K1 | Reading comprehension: reading = navigation | reading + choosing a path |
| MX-H08 | קריאה ליד המדורה | `campfire-read` | K1 | Fluency, reading aloud (on-device ASR, not punitive) | voice |
| MX-H09 | שביל הזוהר | `glow-trace` | K2 | Letter-form tracing | trace (Skia) |
| MX-H10 | גן השורשים | `root-garden` | K1 | Roots and morphology | drag letters into a template |
| MX-H11 | בית הבד של האיות | `spell-press` | K1 | Spelling (confusables: ט/ת, כ/ח, א/ע) | tap/drag |
| MX-E01 | צדף ההד | `echo-shell` | K2, K1 | English listening, TPR, vocabulary | tap/movement |
| MX-E02 | בועות שיר | `song-bubbles` | K3, K2 | English songs, rhythm | tap |
| MX-E03 | צדפי הצלילים | `phonics-shells` | K1 | Phonics (recorded sounds), CVC blending | drag |
| MX-E04 | נמל המילים | `word-harbor` | K1 (K2 oral) | Word–picture, sandwich, gradual reading | tap/drag |
| MX-X01 | טלסקופ הפליאה | `wonder-scope` | K2, K1 | Observation, See-Think-Wonder, world knowledge | pan/tap |
| MX-X02 | מעבדת "בואו נבדוק" | `test-it-lab` | K2, K1 | Predict–test–revise, fair test | tap/drag |
| MX-X03 | חוקר המפה | `map-explorer` | K2 (right/left), K1 (Israel/world map) | Spatial orientation, geography | tap-to-move on a diorama |
| MX-X04 | תרמיל המיון | `sort-pack` | K3, K2, K1 | Classification (animals, living/non-living, habitats) | magnet/drag |
| MX-A01 | שלט הרובוט | `robot-remote` | K3 (2 choices), K2, K1 | Precise instructions, sequencing (CT) | tap cards |
| MX-A02 | ללמד את רובוט המיון | `teach-sortbot` | K2, K1 | Training data, bias | drag examples |
| MX-A03 | בלשי ה-AI | `ai-detective` | K2 (Spot the Silly), K1 | Checking AI output | tap / source |
| MX-A04 | חדר הבקרה | `mission-control` | K1 | Decomposition, delegation to agents, review | drag cards + review |
| MX-T01 | חגורת הכלים | `tool-belt` | K2, K1 | Choosing a strategy, solving two ways | choose an icon |
| MX-T02 | בלש הבאגים | `debug-path` | K2, K1 | Debugging an arrow program | tap/replace an arrow |
| MX-T03 | מי בצל? | `shadow-guess` | K2, K1 | Constraint-seeking questions | choose a question |
| MX-T04 | מוזיאון הטעויות | `mistake-museum` | K2, K1 | Learning from mistakes, intellectual humility | tap + recording |
| MX-T05 | במת ההצגה | `show-stage` | K2, K1 | Speaking, explaining (Show&Tell) | voice recording |
| MX-S01 | מד הרגשות | `feelings-meter` | all | Emotion labeling | tap |
| MX-S02 | נשימת פנס | `lantern-breath` | all (K3 with a parent) | Calming, slow breathing | tap-and-hold / mic (optional) |
| MX-S03 | בלש המחשבות | `thought-detective` | K1 (K2 light) | Thinking traps → balanced thought | tap |
| MX-S04 | מי בנעליים? | `whose-shoes` | K2, K1 | Perspective-taking | choose |
| MX-S05 | שיחת צוות | `team-talk` | Family | Resolving sibling conflicts | protocol on the parent's screen |
| MX-S06 | סולם האומץ | `brave-ladder` | K2, K1 (with a parent) | Graded steps of courage (offline) | parent tick |
| MX-R01 | מרדף הגחליליות | `firefly-chase` | K2, K1 (K3 as a passenger) | Mixed review in runner form | tap-to-jump / lane |
| MX-B01 | הדלקת צללונים | `shadow-light` | everyone | Battle: an answer = a light attack | by item |
| MX-B02 | הגנת המדורה | `campfire-defense` | everyone (by role) | Evening: mixed review, light strategy | by role |
| MX-P01 | צעצועי האור | `playroom-toys` | K3 | Open play: cause and effect, colors, 1–3, animals | tap |
| MX-M13 | מאזני הקסם | `balance-scale` | K2, K1 | Heavy/light, "=" as balance (a relation), missing-symbol puzzles (03 `MECH.BALANCE`, `R.BALANCE`) | drag onto pans |
| MX-M14 | שעון המחנה | `camp-clock` | K2, K1 | Daily sequence, reading/setting an analog clock (hour, half, quarter, 5 min), elapsed time (03 `MECH.CLOCK_CAMP`) | rotate hands |
| MX-M15 | המכולת של המחנה | `camp-shop` | K2, K1 | Shekels/agorot, paying, change (03 `MECH.MAKOLET`; **play money, never the game's coins**) | drag coins/notes |
| MX-M16 | קסם המראה | `mirror-lake` | K2, K1 | Symmetry: complete a shape across an axis, find axes (03 `MECH.MIRROR`) | tap tiles / draw |
| MX-M17 | שיחת מספרים ליד המדורה | `number-talk` | K1 (K2 oral) | One problem, 2–3 strategies, compare with "another child's way" (03 `MECH.NUMBER_TALK`) | choose / drag strategy cards |
| MX-M18 | מי חזק יותר? | `light-duel` | K2, K1 | Comparing quantities and numerals, <, >, = (03 `MECH.DUEL`) | tap |
| MX-M19 | צללית | `shadow-tangram` | K3 (T+), K2, K1 | Composing/decomposing shapes (03 `MECH.TANGRAM`) | drag / rotate |
| MX-M20 | דוח הסיירים | `scout-chart` | K2, K1 | Tables, pictographs, bar charts (03 `MECH.SCOUT_CHART`) | drag / choose |
| MX-E05 | אורות המגדלור | `lighthouse-lights` | K1 | Tricky (irregular) English words, sight reading (05 `TW.*.READ`) | tap (catch the word in the beam) |
| MX-E06 | יומן הספינה של Sunny | `ship-log` | K1 (K2 listens) | Decodable stories (Pip), comprehension; "read to your brother" (05 `RD.*`) | tap page / read |
| MX-E07 | שיחה על הרציף | `dock-talk` | K2, K1 | Scripted dialogue turns, speaking (05 `OR.S.*`) | choose a bubble / record-and-compare (K1) |
| MX-T06 | אסימון מוח | `brain-token` | K2, K1 | "Me first, then the machine": commit an attempt before any machine help (07 `BRAIN_TOKEN`) | choose / draw / guess |
| MX-X05 | ארגז החול הפיזיקלי | `build-sim` | K2, K1 | Ramps, wheels, bridges, shadows, circuits; plan-build-test-improve (06 §4.5 `build-sim`, S.TECH.02) | drag / place / run |
| MX-H12 | מפצל הרונות | `rune-splitter` | K2 (K1 review) | **The key K2 predictor:** CV → consonant + vowel (`H.PA.PH.SPLIT`), vowel identity/swap (`H.PA.PH.VOWEL`), CVC segmentation (`H.PA.PH.SEG`), phoneme deletion (`H.PA.PH.DEL`) (04 `rune-splitter`) | tap sound buttons / drag the vowel tile |
| MX-H13 | בלש התאומים | `twin-detective` | K2 (K1 handwriting) | Visual discrimination of look-alike letters (`H.LK.VIS.<PAIR>`), final forms (`H.LK.FIN`), dagesh/shin dot (`H.LK.DAG`, `H.LK.SHIN`), letter vs digit vs picture (`H.PR.UNIT`), print↔handwriting (K1 `H.HW.CURS.REC`) (04 `twin-detective`) | magnifier (drag) + tap |
| MX-H14 | סטודיו כתיבה | `writing-studio` | K2 | Invented spelling with tiles, short decodable words, labelling a drawing, **joint writing with a parent mediation script** (`H.SP.INV`, `H.SP.CVC`, `H.WC.EM.*`; 04 §14.4) — explicitly **not** tracing (04 P8) | drag tiles / parent-mediated |
| MX-H15 | מכונת המילים | `word-machine` | K2 (oral), K1 (written) | Oral morphology: singular→plural, m→f, diminutives, word families, verb agreement (`H.MO.OR.*`); K1 agreement-builder and sentence-lab (04 `word-machine-oral`, `agreement-builder`, `sentence-lab`) | tap / drag forms |

That is **68 mechanics** (51 original + 8 math from 03, 3 English from 05, 1 from 07, 1 from 06, 4 Hebrew from 04); ≥25 are required. In addition, existing mechanics gained **modes/variants** requested by other specs (defined in their cards): MX-E01 `pairs` mode (05), MX-X02 content "skins" (06), MX-T01 unified with 07's `TOOLBELT`, MX-M03 `lasso`, MX-M04 `cart`/`hut`, MX-M07 `table-map`, MX-M12 `board-game` (03 §5.4); from 04 §7.3א: MX-H05 `syllable-clap` (drum per syllable), MX-H07 `summary-chest` (three sentences into a previewed chest) and `story-campfire` (K2/K3 voiced e-book), MX-H08 `readers-theater` (family mode, roles + a recorded show) and `echo-cave`/`phrase-bridge` (repeated-reading stages), MX-H11 `prefix-peeler` (peel ו־ה־ב־כ־ל־מ־ש). **04 §7.3א's mapping of its logical mechanics onto MX-H01…H11 is confirmed as written** (letter-pet/name-banner→H01, rune-forge→H02, lantern-lighter/syllable-train→H03, sound-slingshot→H04, drum-clap/song-stage→H05, nikud-lens/fade-quest/homograph-detective/cloze-bridge→H06, story-campfire/story-map/question-forge/summary-chest→H07, fluency-spell/echo-cave/phrase-bridge/readers-theater→H08, trace-warmup→H09, root-tree/pattern-forge/word-sense→H10, spell-shield/prefix-peeler→H11, talking-book/fire-feeder→P01, speed-ladder→M07 word version, rapid-name = parent assessment screen, not a mechanic). The interim implementations 04 named (H02 reversed, H01 with visual distractors, H10) are superseded by the cards in §7.3.

**Build order (so 68 is a library, not a day-one scope):**
- **Tier A — MVP** (first vertical slice, unchanged): MX-M01, MX-M02, MX-H01, MX-H02, MX-H03, MX-E01, MX-B01, MX-B02, MX-P01, MX-S02, MX-R01 (11).
- **Tier B — first school term** (S0–S1): MX-M03, M04, M05, M11, M12, M18, MX-H04, H05, H09, **H12, H13, H14**, MX-E02, E03, E04, MX-X01, X04, MX-A01, MX-T01, T06, MX-S01, MX-M14, M16 (23). MX-H12 ships **in S0** (before Hanukkah) because `H.PA.PH.SPLIT` is the central K2 predictor (04).
- **Tier C — by S3**: everything else. A mechanic outside Tier A/B may ship as a `review`/`evening` compact form (§7.0) before its full level form exists.
- The production budget in §6.6 ("12 chunks per gimmick") applies to Tier A+B gimmicks in year 1 (34 × 12 = 408 chunks); Tier C gimmicks get 6 chunks each.

### 7.2 Mechanic cards

> Card format: **Ages · Skills · Input** / **Rules** / **Learning = action** / **Feedback** / **Knobs** / **Failure** / **Movement/Evening/Battle**.

#### MX-M01 `number-jet` (סילון המספרים) ★MVP
- **Ages · Skills · Input:**
  - K2: count 1–10, add within 10/20.
  - K1: skip counting, ×, ÷, add/subtract within 1000.
  - Input: K2 taps on dots or chooses a number; K1 uses a numeric keypad or chooses.
- **Rules:**
  - The פנסון gets a jet on its back that flies **exactly N tiles**.
  - The platform with the friend is N tiles away.
  - K2: "count the stones to the friend" → chooses 4 → flies 4.
  - Twist: a cloud covers part of the route, so K2 must count-on from a visible number (Laski & Siegler, count-on).
  - K1: "5 jumps of 3" → 15; a fuel gauge shows ×3 jumps.
- **Learning = action:** the number chosen *is* the flight distance. A wrong choice is a visible miss (landing 1 tile before or after), which turns into feedback.
- **Feedback:** each tile lights up with a counting voice (from the lookup table, feminine forms for rote counting, 00 B4). An exact landing gives T0 plus a "click". An inexact one gives a soft bounce back to the start and a voiced count of the tiles traveled ("טסת 5, החבר ב-6").
- **Knobs:**
  - range (5/10/20/100/1000)
  - number of hidden tiles (0–80%)
  - representation (dots → numeral → equation, CPA)
  - number of distractors (2–4)
  - K1: equation type (+, −, ×, ÷, missing addend)
- **Failure:** after 2 misses, the tiles are numbered. After 3, ניצי flies with the child and counts aloud.
- **Movement:** MV-03 "jump the count" (jump N times to charge the jet). Evening: "a jet of light" shoots exactly N sparks at a shadowling.

#### MX-M02 `subitize-sparks` (ניצוצות במבט) ★MVP
- **Ages · Skills · Input:** K3 1–3; K2 1–6 and 10-frame up to 10. Input: tap.
- **Rules:** a cluster of sparks lights up for a moment (K2: 1.2s; K3: no flash, it stays). Tap the box with the same number of dots/fingers.
  - K3 is a free version: "שָׁלוֹשׁ!" and all the 3s jump. Every tap produces a counting animation, with no right or wrong.
- **Learning = action:** the right box opens and the friend inside comes out.
- **Knobs:** quantity, arrangement (dice/random/10-frame), display time (K2 2.0s→0.8s), how alike the distractors are.
- **Failure:** the cluster comes back and is counted slowly together.
- **Evening:** a shadowling shows N sparks, and you light it with N.

#### MX-M03 `give-n` (להאכיל את הגוזל)
- **Ages · Skills · Input:** K3 1–2 (one-knower → two-knower), K2 up to 10. Input: tap items into a bowl / drag-snap.
- **Rules:** "the chick wants exactly 3 berries". The child puts in, then taps "done".
  - K3: the parent asks; every amount gets a happy "count with me" response, and the parent prompt shows the right amount.
- **Learning = action:** the chick grows into a bigger chick when given the exact amount (stage 1→2→3 within the level).
- **Feedback:** each berry put in triggers the counting voice. On "done" the chick counts its belly ("אחת, שתיים, שלוש: שלוש!"; cardinality by the last word, `math_pedagogy`).
- **Knobs:** N, a distracting item, two chicks with different N.
- **Failure:** if too many, the chick "returns" the extra one with a burp (K2 humor). If too few, it shows an empty mouth.

#### MX-M04 `ten-forge` (נפחיית העשר)
- **Ages · Skills · Input:** K2 bonds to 10 (later to 20). Input: drag-snap into a 10-frame.
- **Rules:** a crystal of 10 slots is needed for a bridge. There are 6; how many are missing? Drag a crystal with 4 dots.
- **Learning = action:** the crystal is filled → the bridge is lit → crossing → a friend.
- **Knobs:** the target (5/10/20), whether the full frame is visible, representation (dots → numeral).
- **Failure:** wrong crystals don't snap, and the empty slots flash with counting.

#### MX-M05 `array-hammer` (פטיש המערכים)
- **Ages · Skills · Input:** K1 × as an array, area, the distributive property (7×8 = 7×5 + 7×3). Input: drag a frame of rows×columns over a grid.
- **Rules:** ברגית needs a wall of "6×4" blocks. Drag to mark 6 rows and 4 columns → the hammer builds.
  - Twist: a ready-made wall with a hole, "how much is missing?"
  - Stretch: "build it in two parts" (decomposition), and the hammer splits the wall.
- **Learning = action:** the array marked *is* the building that rises and lets you climb to the friend.
- **Knobs:** numbers (2–10 ×), with or without a grid, decomposition required, area word problems.
- **Failure:** a wrong array builds a wall that is too short or too long, and you see the friend not reachable. A hint shows the row count.
- **Battle:** a "wall of light" that blocks a wave in the evening.

#### MX-M06 `bar-bridge` (גשר הפסים)
- **Ages · Skills · Input:** K1 word problems (+ − × ÷, comparison), Singapore bar model. Input: drag bar segments and numbers.
- **Rules:**
  - A word problem is read in the HUD, personalized by interests and family (Walkington; the content is fixed and only the surface changes).
  - The bar model is drawn → the segments become bridge planks → a numeric answer.
- **Learning = action:** building the model is building the bridge; the answer opens the gate.
- **Knobs:** number of steps (1–2), model type (part-whole / comparison), numbers, text length.
- **Failure:** after 2 errors, a hint ladder: (1) "what's the whole?" (2) a partially drawn model (3) a worked example.

#### MX-M07 `fact-spells` (קסמי עובדות)
- **Ages · Skills · Input:** K1 × ÷ facts, **only after the accuracy mastery threshold** (gap D11). Input: a 0–9 keypad or a choice.
- **Rules:** in battle or evening, each fact solved is a light spell. 3 in a row = a combo (a "light wave" hitting 3 shadowlings).
- **Learning = action:** the combo is the fluency.
- **Feedback:** rising pentatonic notes; a quiet reset if the chain breaks (no sad sound).
- **Knobs:** the set of facts, "speed round" (opt-in only, against her own record, never against siblings, gap D10).
- **Failure:** a wrong answer gives no penalty. The shadowling "giggles" and the right answer shows for 1.0s, then the item comes back after 2 more items.

#### MX-M08 `place-value-carts` (עגלות ערך המקום)
- **Ages · Skills · Input:** K1 place value to 10,000, reading and writing numbers. Input: drag crystals into carts labeled thousands/hundreds/tens/ones.
- **Rules:** to send 3,407 crystals to the camp, load the carts. Twist: an "exchange" of 10 tens for a hundred (regrouping).
- **Learning = action:** the train only leaves with the right number; the crystals reach the camp (they are visible in the hub as a pile).
- **Knobs:** range, zeros in the middle (3,047), regrouping.

#### MX-M09 `fair-share` (חלוקה הוגנת)
- **Ages · Skills · Input:** K1 division, unit fractions, equivalent fractions. K2 light: equal sharing among 2–3. Input: drag.
- **Rules:** split the treasure among 4 rescued friends so it's fair; cut a cake of light into 4.
  - The social stake: the friends are happy only when it's equal.
- **Knobs:** number of parts, remainder (K1), fractions on a number line (with MX-M12).
- **Failure:** an unequal split leads to a pouting friend (cute, not sad) and a prompt to compare piles.

#### MX-M10 `measure-rope` (חבל המדידה)
- **Ages · Skills · Input:** K2 non-standard measurement (how many steps / blocks), comparing length. K1 cm/m, perimeter, estimation. Input: stretch a rope / drag a unit.
- **Rules:** ברגית asks for a beam "6 blocks long". Stretch the rope, the units appear, cut → the beam is built.
- **Learning = action:** the beam you measured is placed into the bridge or house. Too short means it doesn't reach (visible).
- **Knobs:** unit, estimate-then-measure, perimeter (K1), the area of a room for the hub building.
- **Movement:** MV-05 "measure with your feet" (offline: how many steps long is the hallway).

#### MX-M11 `pattern-path` (שביל הדפוסים)
- **Ages · Skills · Input:**
  - K3: two colors alternating; any tap continues the pattern automatically, with a song.
  - K2: AB, ABB, ABC.
  - K1: growing patterns (2, 4, 6…) and rules.
  - Input: jump onto a tile / tap.
- **Rules:** the path is built from tiles following a pattern, and the next tile is missing. Choosing the right tile extends the path toward the friend.
- **Knobs:** pattern type, length, attributes (color / shape / size / sound).
- **Failure:** a wrong tile "sinks" and comes back. After 2, the pattern plays as a rhythm (sound + color).

#### MX-M12 `line-hop` (קו המספרים בקפיצות)
- **Ages · Skills · Input:** K2 a linear 0–10/0–20 board game (Siegler & Ramani). K1 0–1000, fractions on a line. Input: drag the פנסון to a position, or MV-04 (hop on a taped line on the floor).
- **Rules:**
  - "Where is 7?" Jump to the estimated spot; the tolerance narrows over time.
  - Dice game mode (K2): roll → advance by counting on.
- **Learning = action:** an accurate landing reveals a friend buried in the ground at that spot. An inaccurate one reveals "warm/cold" and a short trail to the right place.
- **Knobs:** range, tolerance (±15% → ±5%), marked anchors (0, 5, 10), fractions.

#### MX-H01 `rune-lens` (עדשת הרונות) ★MVP
- **Ages · Skills · Input:** K2 letter–sound (sound first, 00 B6). Input: tap.
- **Rules:**
  - The lens reveals runes (letters as pre-baked textures) on the stones.
  - ניצי says a sound ("/m/… מָ"). Point the lens at the rune that makes the sound; the rune lights up and the friend that was stuck inside comes out.
  - The letter order follows the default track, or the parent's "letter of the week" (gap §3).
- **Learning = action:** only the correct sound "unlocks" the lens.
- **Feedback:** the rune hums its sound; the friend says "!מָ… מָצָאתָ אוֹתִי" (gendered form).
- **Knobs:** number of runes (2–5), similarity of the distractors (visually confusable ב/כ, ד/ר, ה/ח, or acoustically confusable), with or without nikud.
- **Failure:** a wrong rune gives a soft "zzz" and its own sound is played (feedback with information). After 2 errors, the target rune glints.
- **Movement:** MV-06 "freeze dance": the music stops on a rune, and you find something in the room that starts with it.

#### MX-H02 `syllable-forge` (נפחיית ההברות) ★MVP
- **Ages · Skills · Input:** K2 letter + nikud → syllable (בָּ, בִּי, בוּ), blending. K1 review/complex nikud. Input: drag a letter onto an anvil + drag a vowel.
- **Rules:**
  - Drag "ב" and then "קמץ" onto the anvil, then hammer it → "בָּ" is heard.
  - The target: "build 'מוּ' for the cow on the hill."
- **Learning = action:** the forged syllable becomes a key or a tool that opens a cage.
- **Knobs:** set of vowels, number of letters, Ten twist: "the same letter, a different vowel".
- **Failure:** a wrong syllable is forged and said aloud ("בִּי… hmm, we needed בָּ"). Learning from the sound you heard.

#### MX-H03 `word-bridge` (גשר המילים) ★MVP
- **Ages · Skills · Input:** K2 blending 2–3 syllables (בָּ-בָּה), K1 decoding longer words. Input: jump on stones in order (right to left!).
- **Rules:**
  - A river with stones; each stone is a syllable. Jumping on them in order reads the word aloud, with each syllable said as you land.
  - At the end the word turns into an object (בָּבָּה → a doll comes to life and joins).
- **Learning = action:** the order of the jumps *is* the reading direction (RTL reinforcement) and the blending.
- **Knobs:** number of syllables, distracting stones, picture shown before or after.
- **Failure:** a wrong stone "wobbles" without sinking. After 2, the stones light up in order.

#### MX-H04 `first-sound-hunt` (ציד הצליל הראשון)
- **Ages · Skills · Input:** K3 vocabulary (the friend says the object's name, and all touches are fine). K2 initial sound. Input: magnet / tap.
- **Rules:** "the magnet only pulls things that start with /s/." Swing the magnet over 4 objects; only the right ones stick.
- **Learning = action:** the right magnet pulls the right pieces; the others "slide off" with a funny sound.
- **Offline:** "find 3 things at home that start with מ" (photo or a parent tick).

#### MX-H05 `rhyme-drums` (תופי החריזה)
- **Ages · Skills · Input:** K2 rhyme recognition. Input: tap a drum / jump on a drum.
- **Rules:** a drum says "חָתוּל". Jump on the drum that rhymes (כָּדוּר / סֵפֶר / תּוּת). The right drum plays a beat and the rhythm path continues.
- **Movement:** clap when it rhymes, sit when it doesn't (MV-02 variant).

#### MX-H06 `fog-goggles` (משקפת הערפל)
- **Ages · Skills · Input:** K1 the nikud-fade track (00 B5): pointed → partial → unpointed, on the ~Feb 2027 timeline. Input: tap a word / choose a meaning.
- **Rules:**
  - The lake fog is drawn as **nikud that drifts above the letters**. The fog level of the region tracks the child's position on the fade track.
  - In the level, an unpointed word appears (ספר). The context in the HUD sentence lets you decide (סֵפֶר / סַפָּר / סָפַר).
  - Choosing correctly clears the fog and reveals a friend.
- **Learning = action:** clearing the fog = reading without nikud.
- **Knobs:**
  - % nikud: 100 → 50 (hard vowels only) → 0
  - words with multiple readings
  - sentence length
- **Failure:** the goggles "lend" the nikud for 2 seconds (a hint) without lowering the track. Only the engine moves the track. **Every lend emits `hint:lens{wordId, lexId}`** so the engine can count lens taps per 100 words (04 §6.4 rule 4: ≥8 per 100 words is a difficulty signal, never a penalty).

#### MX-H07 `story-scroll` (מגילת הדרכים)
- **Ages · Skills · Input:** K1 comprehension (literal → inference). Input: reading a short passage + choosing a route or action in the world.
- **Rules:**
  - A diary page or letter describes where the friend went ("הוא הלך אחרי הציפור הכחולה, אבל פחד מהמים").
  - The child chooses a path in the world according to the text. Only a correct understanding gets you to the friend.
- **Learning = action:** reading = navigation (no question after the text).
- **Knobs:** length (40–150 words), nikud by the track, question type (literal / inference / vocabulary in context).
- **Failure:** a wrong path leads to a cute dead end ("רק צפרדע כאן") + a "read again" button with the key sentence highlighted.

#### MX-H08 `campfire-read` (קריאה ליד המדורה)
- **Ages · Skills · Input:** K1 reading aloud and fluency. Input: voice (on-device ASR, matched against known target text, 00 E9).
- **Rules:**
  - In the evening, K1 reads a short story to the friends by the fire. Each sentence read makes the fire grow a little.
  - A parent can listen in. A sibling (K2) can sit and listen: a co-op role!
- **Learning = action:** the friends react to the story (laugh, gasp) according to the sentence read.
- **Never punitive:** ASR does not "fail" a sentence. If there's no match, the sentence is marked "נקרא" anyway, and ניצי says "קראת, יופי". Measuring wpm and accuracy goes to the parent only, as an approximation, and only for pointed text until the middle of the year (gap §4.9).
- **Knobs:** text length, nikud by the track.
- **Fallback:** a "read silently" button → a short question about the story (accessibility, noise, Shabbat).

#### MX-H09 `glow-trace` (שביל הזוהר)
- **Ages · Skills · Input:** K2 letter forms (Hebrew print). Input: trace (Skia), tolerance of 8 mm.
- **Rules:**
  - The letter is drawn as a path of light in the forest; tracing it with a finger along the stroke directions lights up a route.
  - At the end the letter "comes to life" and becomes a friend.
- **Learning = action:** the finger's path *is* the hero's path.
- **Phygital:** a matching worksheet to print, for free handwriting (00 C10). In-app accuracy is not the measure.
- **Knobs:** tolerance, direction arrows, start dot, letters with similar forms.

#### MX-H10 `root-garden` (גן השורשים)
- **Ages · Skills · Input:** K1 roots (ש.מ.ר → שומר, שמירה, משמרת). Input: drag letters into a template (משקל).
- **Rules:** a root is planted as a seed. Templates are "watering cans"; each can grows a word-flower. Twist: a word that only looks like it shares the root.
- **Learning = action:** a correct word flowers and a friend comes out of it.

#### MX-H11 `spell-press` (בית הבד של האיות)
- **Ages · Skills · Input:** K1 spelling confusables (ט/ת, כ/ח, א/ע, ק/כ, ס/ש). Input: tap/drag.
- **Rules:** a missing letter in a word on a sign; choose the right letter to press it on. Twist: a word that is spelled differently from how it sounds.
- **Knobs:** pairs, word frequency, morphological hint ("from the root…").

#### MX-E01 `echo-shell` (צדף ההד) ★MVP
- **Ages · Skills · Input:** K2 oral English (TPR, vocabulary), K1 vocabulary + simple sentences. Input: tap / movement (MV-08).
- **Rules:**
  - Sunny says "Jump! / Touch the red boat!"
  - K2: the sandwich (Sunny-ניצי-Sunny). Choosing the right object with the shell makes it echo the word.
  - Twist: "Sunny says" vs. no Sunny (just an instruction without "Sunny says" means don't move).
- **Learning = action:** the right object comes to life and joins as a Wordling.
- **Knobs:** number of words (2–4 choices), new/review ratio (1:3), gloss level (K2 always; K1 3→0).
- **Rules:** only within the English island or an English frame (00 B2). No ASR for K2 (00 E9).

#### MX-E02 `song-bubbles` (בועות שיר)
- **Ages · Skills · Input:** K3 + K2 English songs, rhythm. Input: tap bubbles in rhythm.
- **Rules:** a short song (30–45s); each bubble popped on the beat plays a word with a picture. There is no "miss": a bubble that isn't popped floats away.

#### MX-E03 `phonics-shells` (צדפי הצלילים)
- **Ages · Skills · Input:** K1 English letter sounds (native recordings only), CVC blending. Input: drag shells into a row.
- **Rules:** 3 shells (c-a-t). Drag them into a row → Sunny blends them → the cat appears. Twist: change one shell (c→h) and get a hat.
- **Knobs:** set of letters, short vowels, digraphs (later).

#### MX-E04 `word-harbor` (נמל המילים)
- **Ages · Skills · Input:** K1 vocabulary (picture ↔ word, reading words), K2 oral only. Input: tap/drag.
- **Rules:** boats arrive with pictures; the harbor sign (in English) says a word, and you direct the right boat to the dock.
- **Knobs:** written word / spoken word, number of boats, semantic categories.

#### MX-X01 `wonder-scope` (טלסקופ הפליאה)
- **Ages · Skills · Input:** K2 observation (see–think–wonder in a light form), K1 world knowledge (animals, plants, space, the human body, Israel). Input: pan + tap.
- **Rules:**
  - פרופסור למה asks "what do you see?" Pan the telescope across the landscape and tap on details.
  - Then "what do you think?" (choosing from 2–3) and "what do you wonder?" (choosing a question).
  - Each detail found is a פליאון (a friend).
- **Learning = action:** finding details = rescues. Choosing a question = a "wondering coin".

#### MX-X02 `test-it-lab` (מעבדת "בואו נבדוק")
- **Ages · Skills · Input:** K2 a blicket detector ("which block lights the machine?"), K1 a fair test (change one variable). Input: tap/drag.
- **Rules:**
  - Predict, test, "were you surprised?"
  - Confounded trials invite a retest.
  - "Mind-changer medal" for updating a belief (intellectual humility, `future_skills` §5.6).
- **Learning = action:** a correct finding powers a machine that opens the way.

#### MX-X03 `map-explorer` (חוקר המפה)
- **Ages · Skills · Input:** K2 right/left/above/below/next to. K1 reading a map, directions, a map of Israel (regions, the Kinneret, Jerusalem, the Negev). Input: tap-to-move on a diorama.
- **Rules:** ניצי gives a route ("two steps right, then to the tree") / K1 reads a treasure map with a compass and scale.
- **Learning = action:** following the instructions accurately leads to the friend.

#### MX-X04 `sort-pack` (תרמיל המיון)
- **Ages · Skills · Input:**
  - K3: animals vs. vehicles. Any order is accepted; every placement plays the object's name and sound.
  - K2: living / non-living, land / sea / air.
  - K1: habitats, food chains.
  - Input: the backpack's magnet / drag.
- **Rules:** a backpack with 2–3 pockets; each object goes in the right pocket, and a full pocket "explodes" into sparks that release a friend.

#### MX-A01 `robot-remote` (שלט הרובוט)
- **Ages · Skills · Input:**
  - K3: 2 picture instructions (blue ball / red ball); the robot does exactly what was picked, and both are fun.
  - K2: arrow cards (sequence).
  - K1: loops ("repeat 3").
  - Input: tap cards.
- **Rules:**
  - Program a mini גלגלון to reach a friend.
  - Comic failures for incomplete instructions (bread on the head in the "literal robot kitchen", `ai_literacy` C1).
- **Message:** "the machine does exactly what you told it".
- **Offline:** "Program the Parent Robot".

#### MX-A02 `teach-sortbot` (ללמד את רובוט המיון)
- **Ages · Skills · Input:** K2/K1 training data and bias. Input: drag examples.
- **Rules:**
  - Teach the bot "fruit vs. toys" with 6–10 examples, then test it.
  - Only red apples → it thinks a green apple isn't fruit → add variety → it improves.
- **Learning = action:** better data = a bot that gets through the gate.

#### MX-A03 `ai-detective` (בלשי ה-AI)
- **Ages · Skills · Input:** K2 "Spot the Silly" (find the odd thing in an image generated in advance: a clock upside down, a cat with 3 ears). K1 verifying claims: גלגלון gives 5 "facts" with 1–2 mistakes; check them in the "library book" or the map, or ask a parent. Input: tap / choose a source.
- **Rules:** badges for "two sources agree" and "asked a person" (`ai_literacy` D2/D3). Questions the AI can't know ("what's Grandma's favorite song?") → route to a person.
- **Content:** pre-generated and validated, never live (00 C7).

#### MX-A04 `mission-control` (חדר הבקרה)
- **Ages · Skills · Input:** K1 decomposition, delegation, review. Input: drag cards + approve/send back.
- **Rules:**
  - A mission ("prepare a family picnic in the Galilee"). Break it into steps → assign Researcher-bot / Artist-bot / Checker-bot → review the outputs (prepared in advance with deliberate mistakes) → send back with feedback ("too long", "wrong animal") → approve.
- **Learning = action:** the planned picnic actually takes place in the camp (an animation of the friends).
- **Sibling form:** K1 is the "Mission Commander" who gives K2 a picture task as "Artist-bot" (`ai_literacy` sibling mode).

#### MX-T01 `tool-belt` (חגורת הכלים)
- **Ages · Skills · Input:**
  - K2: 3 math tools (count all / count on / draw) + 3 reading tools (sound out / look at the picture / ask).
  - K1: 6–8 tools (work backwards, a smaller case, a table, a person, ask the AI to explain, not to answer).
  - Input: choose an icon before solving.
- **Rules:**
  - "Was there another way?" A bonus for solving the same problem with 2 tools.
  - An "Efficiency badge" for finding the shorter one.
  - "Invent a tool" (K1) to name and add to the belt.
- **Integration:** the belt shows on the פנסון. The chosen tool changes the arms (cosmetic) in the following mechanic.

#### MX-T02 `debug-path` (בלש הבאגים)
- **Ages · Skills · Input:** K2/K1 debugging. Input: tap the wrong arrow and replace it.
- **Rules:** a ready-made program for גלגלון that "almost" works. Find the bug (an arrow in the wrong direction or a missing step). Loops for K1.

#### MX-T03 `shadow-guess` (מי בצל?)
- **Ages · Skills · Input:** K2/K1 constraint-seeking questions. Input: choose a question from cards.
- **Rules:** a shadowling hides one of 8 friends. Ask yes/no questions ("is it an animal?"). A good question (one that splits the group) eliminates more, and the cards fall with a sound. Find it in ≤3 questions (K1) or ≤5 (K2).

#### MX-T04 `mistake-museum` (מוזיאון הטעויות)
- **Ages · Skills · Input:** K2/K1. Input: tap + optional recording.
- **Rules:**
  - After a mistake and a correction, the child can "donate" the mistake to the museum (in the hub) with "what I thought / what I learned" (chips; K1 short text or recording).
  - Parents can add their own (via the parent area).
  - A weekly "most useful mistake" in the family evening (no voting between siblings; each child picks their own).

#### MX-T05 `show-stage` (במת ההצגה)
- **Ages · Skills · Input:** K2/K1. Input: voice recording (30–60s, on the device).
- **Rules:** "explain to ניצי how you built the bridge". It plays back to the parent in Show&Tell. A bravery ladder for the audience: the character → a parent → a grandparent (sent via the parent).

#### MX-S01 `feelings-meter` (מד הרגשות)
- **Ages · Skills · Input:**
  - K3: 4 faces, with a parent.
  - K2: a color + a word.
  - K1: a nuanced word + intensity 1–3.
  - Input: tap.
- **Rules:**
  - Optional at the start of an episode (07 `RIT-OPEN`). **Cadence by band (07 §2.4, in sync with §5.3):** S (K1) every episode; G (K2) every other day, first episode of that day only; T (K3) optional, parent-led. ≤20 s, always skippable ("לא רוצה להגיד" is a legitimate answer and is never flagged); the parent can turn it off.
  - דובדובה validates ("מותר להרגיש…") and offers a fitting tool.
  - **Data is not stored, or stays on the device only** (00 C8).

#### MX-S02 `lantern-breath` (נשימת פנס) ★MVP
- **Ages · Skills · Input:** everyone (K3 with a parent). Input: tap-and-hold = breathe in (3–4s), release = slow breath out (4–6s). Optional mic: detects exhaling without recording (MV-10).
- **Rules:** a paper lantern fills with light on the in-breath and glows steadily on a slow out-breath. 3–5 breaths, then the lantern rises to the sky and lights a star.
- **Integration:** this is the "pause" tool, available any time from the pause button. In the evening, a steady breath lights the lanterns around the camp.
- **Framing:** "a tool that helps the body calm down", not a cure (`cbt_sel` §3). The frame matches the fire world, but the action is "light a lantern", never "blow on fire".

#### MX-S03 `thought-detective` (בלש המחשבות)
- **Ages · Skills · Input:** K1 (K2 simplified). Input: tap.
- **Rules:**
  - A character says a thought ("אף אחד לא רוצה לשחק איתי אף פעם").
  - Identify the thinking trap (a mask: "always/never"), collect clues (fact vs. feeling), and choose a balanced thought.
  - The character gets light and a smile.
- **Content:** scripted, reviewed (00 C8).

#### MX-S04 `whose-shoes` (מי בנעליים?)
- **Ages · Skills · Input:** K2/K1 perspective-taking. Input: choose.
- **Rules:** a short story from two points of view (a shadowling who took a ball and a friend who lost it). Guess what each one feels and knows (false-belief-style for K2). A good choice means the two make up and a VIP friend joins.

#### MX-S05 `team-talk` (שיחת צוות)
- **Ages · Skills · Input:** family. Input: a protocol on the parent's screen plus a "talking stick" button.
- **Rules:** 5 steps (`cbt_sel` §4): lantern breath → I-message → repeat back → 2–3 solutions → choose + high five. K1 can be "team captain" in rotation. K3 is "keeper of the stick".
- **Use:** from the parent area only, not as a punishment.

#### MX-S06 `brave-ladder` (סולם האומץ)
- **Ages · Skills · Input:** K2/K1, together with a parent. Input: a parent tick.
- **Rules:** the parent sets 3–5 rungs (e.g., "ask the waiter for water myself"). Each rung completed in the real world is a star on a ladder in the hub. Celebrate the *attempt*.
- **Guardrail:** for a real fear, the parent gets a note about a referral path (00 C8).

#### MX-R01 `firefly-chase` (מרדף הגחליליות) ★MVP — see §9.4

#### MX-B01 `shadow-light` (הדלקת צללונים) ★MVP — see §9.1

#### MX-B02 `campfire-defense` (הגנת המדורה) ★MVP — see §9.5

#### MX-P01 `playroom-toys` (צעצועי האור) ★MVP (K3)
- **Ages · Skills · Input:** K3 cause and effect, color names, animals, 1–3, body parts. Input: tap only (any point on the object counts).
- **Rules:**
  - A 2.5D isometric playroom with 6 toys. Each toy does something different:
    - a drum says "בּוּם" and the number of beats;
    - a ball rolls and says its color;
    - a xylophone with 3 colors;
    - a farm with animals;
    - a cloud that rains bubbles;
    - a box of peekaboo.
  - No goals, no wrong answers, no text (Pok Pok, R7).
- **Learning = action:** every touch is labeled with a word (from the approved lexicon, both genders where needed), so the toy "talks".
- **Feedback:** soft recorded sounds (foley) with no harsh stingers. A bounce animation.
- **Knobs:** the number of toys rises (from 3 to 6), and new words are added over the weeks (the parent can choose a "word of the week").
- **Safety:** no exits, no menus. Exit is through the parent gate only. Sound volume is limited (`audio.k3.maxGainDb = -6`).

### 7.3 Mechanics requested by other specs (full cards)

> Every card below follows the same fields as §7.2. Skill IDs are the owning spec's (03 `M.*`, 05 `E.*`, 06 `S.*`, 07 `TK.*`/`AI.*`). Distractors and answers are computed in code from the owning spec's error catalogs (00 E8).

#### MX-M13 `balance-scale` (מאזני הקסם) — 03 `MECH.BALANCE`, remediation `R.BALANCE`
- **Ages · Skills · Input:** K2 heavy/light, "which pan goes down?" (`M.MEAS.WEIGHT.BALANCE`); K1 "=" as a relation (`M.OPS.EQ.MEANING`), missing-symbol puzzles (`M.PAT.BALANCE.EQ`). Input: drag crystals/objects onto two pans; tap "balanced?".
- **Rules:** the Muddler's scale guards a gate. K2: "which is heavier, the feather or the apple?" — drag, the pan sinks, the gate tilts open only when the child predicted correctly. K1: each pan shows an expression (`4 + 4` | `6 + 2`); the child balances by adding/removing crystals or decides "balanced / not balanced". Twist (Ten): symbols (🔺 + 🔺 = 12) — find the symbol's weight.
- **Learning = action:** the scale's tilt *is* the comparison; the gate opens when the scale is level. A wrong prediction shows the real tilt, which is informative, never a buzzer.
- **Feedback:** the beam animates with physics (overshoot, settle 400ms); the pan that is heavier "thuds"; balance = a chime + the gate lights.
- **Knobs:** object count (2–6), representation (objects → dots → numerals → expressions), symbol puzzles (K1), tolerance for "about equal" (K2 only: visibly different masses).
- **Failure:** after 2 errors the scale shows a translucent "ghost" of the correct tilt; after 3, ניצי weighs each side aloud (worked example).
- **Evening/Battle:** compact form — "balance the shadowling": choose the expression equal to the one on its belly.

#### MX-M14 `camp-clock` (שעון המחנה) — 03 `MECH.CLOCK_CAMP`
- **Ages · Skills · Input:** K2 daily sequence and "what comes before/after" (`M.MEAS.TIME.SEQ`, `M.MEAS.TIME.WEEK`); K1 analog clock on the hour/half/quarter (`M.MEAS.TIME.HOUR_HALF`), 5-minute precision (`M.MEAS.TIME.CLOCK5`), elapsed time (`M.MEAS.TIME.ELAPSED`). Input: rotate the hands (drag), or tap a time card.
- **Rules:** the camp's big clock stopped when the Muddler sneezed; the friends don't know when to wake up. Set the clock to "the time the shadowling wakes (7:00)" and the friend's door opens. Elapsed time (K1): "the bread needs 25 minutes; it went in at 3:10" — rotate forward and the oven dings. **The minute hand drives the hour hand** (geared), so quarter-past shows the hour hand slightly past (03: C stage "clock with gears").
- **Learning = action:** the set time *is* the event: the door opens, the oven dings, the lighthouse lights at the chosen time. A wrong time makes the friend yawn and point at the sun/moon (a context hint).
- **Feedback:** each 5-minute step clicks; a digital readout appears under the clock **after** the child sets it (analog → digital link).
- **Knobs:** precision (hour → half → quarter → 5 min → 1 min), hand-gearing on/off, digital shown before/after, elapsed-time spans (5–60 min).
- **Failure:** after 2 errors the target time appears as a faint ghost on the dial; after 3, the hands move themselves while ניצי says the time.
- **Never a visible timer:** this mechanic shows clock faces as *content* only; it never shows a countdown (03 §9.1, MATH-AC-06).
- **Offline:** "set the real clock at home to bath time" (03 `HOME.06`).

#### MX-M15 `camp-shop` (המכולת של המחנה) — 03 `MECH.MAKOLET`
- **Ages · Skills · Input:** K2 coins as objects, paying exact amounts to 10 (`M.NUM.*` money contexts); K1 shekels/agorot, paying and making change (`M.MEAS.MONEY.*`). Input: drag play coins/notes to the counter; tap "pay".
- **Rules:** ברגית runs a camp grocery where friends buy supplies with **play money (שקלים)** found in the level. "The bucket costs 7 ₪. Pay." Twist: pay with a 10 ₪ note and receive the change — the child counts it back ("8, 9, 10").
- **Hard separation from the game economy (binding):** play shekels are a **level prop**, not the game's coins. They appear only inside this mechanic, reset at level end, are never shown next to the coin receipt, and use a different icon (a shekel sign, not the camp coin). No shop item is ever priced in shekels and no play shekel converts to anything (AC-MX-7).
- **Learning = action:** the correct payment *is* what lifts the item off the shelf; over/under-payment makes ברגית hold the coins up and count aloud.
- **Knobs:** amounts (to 10 / 20 / 100), coin set (1, 2, 5, 10 ₪; agorot only for K1 S-band), change required, multiple items (addition).
- **Failure:** after 2 errors the coins needed glow; after 3, worked example.

#### MX-M16 `mirror-lake` (קסם המראה) — 03 `MECH.MIRROR`
- **Ages · Skills · Input:** K2 complete a symmetric picture across a vertical axis (`M.GEO.SYM.COMPLETE`); K1 vertical/horizontal/diagonal axes, find the axes of a shape, "is the diagonal an axis?" (`M.GEO.SYM.LINE`). Input: tap tiles in a grid / draw with Skia (tolerance 8 mm).
- **Rules:** the fog lake reflects half a bridge; build the other half so the friend can cross. K1 twist: the lake tilts the axis (diagonal). Purim/Yom HaAtzmaut skins: complete the flag (03 `U.K1.09`).
- **Learning = action:** the reflection becomes solid only where it is correct; the bridge holds only when both halves match.
- **Feedback:** a correct tile "ripples" into place; a wrong tile floats away (informative).
- **Knobs:** grid size (4×4 → 8×8), axis orientation, number of colors, "find the axis" vs "complete".
- **Failure:** after 2 errors a faint reflection appears for 2s; after 3, the axis is drawn and the tiles mirror one by one with ניצי.
- **Offline:** a folded-paper print (03 `PRN.K1.09.2`).

#### MX-M17 `number-talk` (שיחת מספרים ליד המדורה) — 03 `MECH.NUMBER_TALK`, P10
- **Ages · Skills · Input:** K1 (weekly, at least once): solve one problem in ≥2 ways and compare strategies (`M.WP.TALK.MULTI`, `M.WP.EXPLAIN`); K2 oral form with ניצי (how did you count?). Input: choose/drag strategy cards; optional 15s voice recording (on device).
- **Rules:** an evening scene by the fire. A problem (e.g., 25 + 19) appears. Step 1: the child solves with any tool from the tool belt (MX-T01). Step 2: "another way?" — a second strategy. Step 3: a scripted "other child" (a VIP friend, never a sibling) shows a third way; the child says which is faster and why (chips). **No speed, no timer, no score** (03 §9.1).
- **Learning = action:** each strategy lights one lantern around the fire; the "comparison" lights the big one. The friend wakes when ≥2 lanterns are lit.
- **Feedback:** process praise only ("פֵּרַקְתְּ אֶת 19 לְ-20 פָּחוֹת 1"). The recording plays back to the parent in Show&Tell.
- **Knobs:** number range, strategy set (count-on, make-ten, compensation, doubles, decomposition), number of ways required (2–3), K2 oral mode.
- **Failure:** none (no wrong strategy, only "did it work?"). If the first answer is wrong, the scripted friend models checking, not correcting.
- **Compact form:** one "another way?" prompt after any MX-M01/M04/M05 item (07 `another-way` hook).

#### MX-M18 `light-duel` (מי חזק יותר?) — 03 `MECH.DUEL`
- **Ages · Skills · Input:** K2 compare two quantities (more/fewer, 1–10), then numerals (`M.NUM.COMP.*`); K1 numerals to 1000, <, >, = symbols, fractions (`M.NUM.COMP.*`, `M.FRAC.COMPARE.*`). Input: tap the stronger torch / drag the crocodile mouth (the symbol).
- **Rules:** two torches of fireflies face a shadowling; the brighter one warms it. "Which torch is stronger: 7 or 9?" Tapping the bigger one sends its light. Twist: the quantities are shown in different representations (dots vs numeral). K1: drag the "mouth" (<, >) to point at the bigger number.
- **Learning = action:** only the correct comparison lights the shadowling; a wrong tap makes the weaker torch sputter (visible magnitude).
- **Feedback:** the two torches briefly show their quantities side by side after the answer (always informative).
- **Knobs:** range, representation mix, visual distance between quantities (03 "distance effect"), symbols on/off.
- **Failure:** after 2 errors the quantities line up as two rows (1:1 correspondence); after 3, counted aloud together.
- **Movement:** MV-09 (lean toward the bigger).

#### MX-M19 `shadow-tangram` (צללית) — 03 `MECH.TANGRAM`
- **Ages · Skills · Input:** K3 (T+): 2–3 big shapes into a silhouette, every placement accepted; K2 compose a house/boat from 3–5 shapes (`M.GEO.COMPOSE.SHAPE`); K1 7-piece tangram, rotations, decomposition (`M.GEO.DECOMPOSE`). Input: drag; rotate by a tap (90°/45°); no multi-finger gestures required (09 §10).
- **Rules:** a shadowling cast a strange shadow on the tent wall; fill the silhouette with shapes and it turns into a friend (a cat, a boat).
- **Learning = action:** the silhouette becomes a friend only when fully covered with no overlaps.
- **Feedback:** a shape snaps (20 mm) with a soft click; an overlap "wobbles" out.
- **Knobs:** number of pieces (2–7), rotation needed, outline shown (full / partial / none, K1), time-free.
- **Failure:** after 2 misplacements the next piece's outline glows; after 3, one piece places itself.
- **Offline:** a printable tangram (03 `PRN.K2.08`).

#### MX-M20 `scout-chart` (דוח הסיירים) — 03 `MECH.SCOUT_CHART`
- **Ages · Skills · Input:** K2 sort and count into a pictograph (`M.DATA.PICTO.G`); K1 tables, bar charts, "how many more?" questions (`M.DATA.*`). Input: drag tokens into columns; tap to answer.
- **Rules:** after an evening, ניצי asks for the scouts' report: "how many shadowlings became friends of each color?" The child builds the pictograph from the actual friends rescued in this episode (real data, no fake numbers), then answers 1–2 questions from the chart.
- **Learning = action:** the report is pinned on the camp board and the chart *is* the decoration; the question's answer opens the mailbox.
- **Knobs:** categories (2–5), scale (1 token = 1 / 2 / 5), question type (read / compare / difference).
- **Failure:** after 2 errors the relevant column highlights; after 3, counted together.
- **Hub hook (06 §4.5 weather station):** the same component renders the family's monthly weather chart, read-only, shared (00 C6-safe: the data is the weather, not a child).

#### MX-E01 `echo-shell` — added **`pairs` mode** (minimal pairs, 05 T09 `echo_pairs`)
- **Ages · Skills · Input:** K1 (K2 "soft" version without scoring): perception of English contrasts (05 `PR.MP.*`: w/v, th, ship/sheep, cat/cut/cot). Input: tap 1 of 2 pictures (AX); ABX at higher levels.
- **Rules:** the shell echoes a word in the voice of **one of 4 adult speakers + the human recording** (05 §12: 2 women, 2 men; never a childlike voice). Two pictures; tap the one you heard. **Listening before production** (05 EN-7): no pronunciation is ever scored.
- **Learning = action:** the right picture comes alive and joins as a Wordling; the wrong one just blinks.
- **Feedback:** after any answer the pair is re-played side by side with both pictures highlighted in turn (always informative).
- **Knobs:** contrast set, speaker variability (1 → 4 voices), AX → ABX, number of pairs per block (4–6).
- **Failure:** 2 errors → the two words play again slowly with the mouth-shape card (05); no third ladder step (perception training, not a test).
- **K2:** the same pairs as pure exposure, every tap "finds" something; no state is recorded beyond `exposures`.

#### MX-E05 `lighthouse-lights` (אורות המגדלור)
- **Ages · Skills · Input:** K1 tricky/irregular words (05 `TW.*.READ`: the, said, was…). Input: tap the word when the beam lights it.
- **Rules:** waves carry words past the lighthouse. Sunny says a tricky word; the beam sweeps; tap the right word while it is lit (the beam is slow: 1.5s per word, no "miss" penalty — it comes around again). Twist: the same word in two fonts/cases (Pip vs pip, 05 `E.DIR.CASE.*`).
- **Learning = action:** the caught word becomes a Wordling (a star, 4.11) and the lighthouse glows one notch brighter per word.
- **Feedback:** Sunny repeats the word and uses it in a 3-word sentence; the "tricky part" of the word is underlined after the catch (05 heart-word convention).
- **Knobs:** number of words on the waves (2–4), speed of the beam (1.5s → 1.0s only after `proficient`), case variation, decodable distractors.
- **Failure:** after 2 misses the target word stays lit; after 3, Sunny taps it with the child.

#### MX-E06 `ship-log` (יומן הספינה של Sunny)
- **Ages · Skills · Input:** K1 decodable stories of 3–6 pages (05 `RD.SENT.READ`, `RD.TEXT.COMP`); K2 listens (Sunny reads, pages turn). Input: tap a page to turn; tap the picture that matches; optional on-device ASR (05 PP-5: feedback only, never stored as mastery; never punitive).
- **Rules:** each page of Sunny's log is a decodable sentence with **Pip** (CH-11) doing something; reading the page **lights one lantern in the harbor**; at the end a comprehension choice ("what did Pip find?") from 2 pictures. **"Read to your brother" mode** (guide mode, §12.6): K1 reads aloud, K2 taps the picture; K1's Mentor Track progresses, K2 gets exposure only.
- **Learning = action:** the harbor lights up page by page; the story's outcome (a found object) appears in the scene.
- **Feedback:** after each page, Sunny re-reads it fluently (model), then "Next!".
- **Knobs:** pages (3–6), phonics stage of the text (05 S2 → S6), comprehension question type, ASR on/off.
- **Failure:** tapping a word plays it; after 2 wrong comprehension picks, the key sentence is highlighted and re-read. No "wrong" display ever (AC-MX-5 applies).

#### MX-E07 `dock-talk` (שיחה על הרציף)
- **Ages · Skills · Input:** K2 choose a reply bubble (picture + voice); K1 choose or **record-and-compare** (05 `OR.S.*`: name, age, likes, yes/no). Input: tap a bubble; K1 optional 5s recording played back next to Sunny's model (on device, never uploaded, never scored).
- **Rules:** Sunny asks a scripted question on the dock ("What's your name?" / "Do you like boats?"); 2–3 reply bubbles; the chosen reply is voiced by Sunny's bank (K2) or by the child (K1). Pre-written dialogue trees only (00 C7: no open-ended chat). K2 always gets the sandwich via ניצי standing on the dock's Hebrew side (right; 05 spatial convention: English character on the left).
- **Learning = action:** a fitting reply makes the boat cast off and a Wordling boards; an unfitting one makes Sunny tilt her head and ask again with a gesture (never "wrong").
- **Feedback:** Sunny repeats the child's choice in a full sentence.
- **Knobs:** number of bubbles (2–3), question set (05 `CH.*`), record-and-compare on/off (K1 only), gloss level.
- **Failure:** 2 unfitting picks → the gesture hint; 3 → Sunny models the answer and the boat casts off anyway (no fail).

#### MX-T01 `tool-belt` (חגורת הכלים) — **unified with 07 `TOOLBELT`** (replaces the earlier card)
- **Ages · Skills · Input:** K2: 3 math tools + 3 reading tools (count all / count on / draw; sound out / look at the picture / ask) plus 07's SEL/thinking tools as they unlock (dragon breath, I-message, idea cloud…). K1: 6–8 tools incl. "work backwards", "a smaller case", "a table", "ask a person", "ask the machine to explain, not answer". Input: choose an icon.
- **One structure for every strand (07 §2.2):** every tool is a **tool card** (illustration, name, "when to use it", a "used" counter — a count, not a score). Cards live in "המרחב שלי" (the neighborhood) and show on the פנסון's belt (cosmetic arms change in the next mechanic).
- **"Which tool?" moment (07 §2.2):** on 2 errors **or** 10s of no action after an error, a fan of 2–4 relevant tools opens (icons + voice for K2, text for K1). Choosing a tool is free and never costs anything.
- **Tool mastery states (07, replaces accuracy):** `seen` → `tried` (used when offered) → `used` → `chosen` (≥3 independent choices over ≥2 weeks) → `transferred` (a parent confirmed real-world use). `chosen` gives the card a spark; `transferred` a "world star". These are skill-name badges (00 D4), never coins.
- **Rules kept from the original card:** "another way?" bonus line for solving the same problem with 2 tools; an "Efficiency" line for finding the shorter one; "invent a tool" (K1) names and adds a card.
- **Ritual hook:** `ritual_mid` (§5.3) is a 60–120s "tool minute" that opens one card and uses it on one easy item.

#### MX-T06 `brain-token` (אסימון מוח) — 07 `BRAIN_TOKEN`; a **gate**, not a level mechanic
- **Ages · Skills · Input:** K2, K1. Input: deposit an attempt — an answer, a choice of 2–3, a guess, a plan, or a quick drawing.
- **Rules (binding across every mechanic with a גלגלון hint ladder):** the "help from the machine" button is **disabled until the child deposits a brain token**. Two wrong attempts on an item count as a deposit; so does any explicit choice/guess. The deposit lights the lamp on גלגלון's head; only then is hint step 1 available. Help order is always: a returned question ("מה לדעתך הצעד הראשון?") → a hint → an explanation; **never a final answer first** (07 §2.3, 00 C7).
- **Learning = action:** the lamp on גלגלון *is* the token; the hint ladder literally cannot start without it.
- **Feedback:** גלגלון: "ביפ! קודם המוח שלך, אחר כך הגלגלים שלי. מה הניחוש שלך?" → after the deposit: "קיבלתי את הניסיון שלך. רוצה רמז קטן או בדיקה?"
- **Knobs:** deposit type per mechanic (`answer` | `choice` | `drawing` | `plan`), K2 always `choice`.
- **Failure:** none; the token is never judged.
- **Guide mode:** K1's hint tokens (§12.6) obey the same gate on K2's deposit.
- **AC:** 07 AC-07 (machine help unavailable before a deposit) is implemented once in `packages/engine/hints/brainToken.ts` and reused by every mechanic.

#### MX-X02 `test-it-lab` — added **content skins** (06 §4.5)
- The round structure stays 07's (`blicket`, `TK.EXP.1`): 2 unambiguous trials → 1 confounded trial → "want to test again?" → a single test → "what did you find?", with the prediction recorded **before** each test.
- **Skins (06):** "בודק המוליכים" (which object closes the circuit → `S.ENG.05`), "גלאי המגנט" (→ `S.MAT.05`), "מה נובט?" (seeds with water/light, accelerated time → `S.LIFE.02`). K1 fair-test labs with the 3-locked-variables board: "עציצים", "מצנחים", "מטריות" (→ `S.LIFE.02/06`, `S.MAT.07`, `S.MAT.02`). Changing two variables → the advisor asks "איך נדע מה גרם לשינוי?" (a hint, not a failure).
- **Camp hook:** every circuit closed in the conductor skin lights a lantern in the camp (06 "פנס המחנה"; it adds **light**, not logs or coins).

#### MX-X05 `build-sim` (ארגז החול הפיזיקלי) — 06 §4.5
- **Ages · Skills · Input:** K2 push/pull, ramps, shadows (`S.ENG.01/02`); K1 bridges, wheels, pulleys, circuits, plan-build-test-improve with ≥2 versions (`S.TECH.02`, `S.TECH.03`, `S.ENG.05`). Input: drag/place parts; tap "run".
- **Rules:** a 2D physics sandbox (Skia + Reanimated, simple equations, no physics library, E2) in ברגית's yard. Each challenge has a goal ("bring the logs to the fire") and a **material limit** (8 beams) that forces planning. Run → watch → improve. K1 rubric: ≥2 versions tried, a test against the goal (06 M-PERF).
- **Learning = action:** the contraption works or visibly doesn't (the log stops, the bridge sags); success brings the goods to the camp (visible in the hub).
- **Feedback:** slow-motion replay of the failure point (0.5× for 2s); ברגית names the cause ("הרמפה תלולה מדי").
- **Knobs:** part set, material limit, goal distance, gravity/friction presets (fixed, not child-tunable), circuit parts (K1).
- **Failure:** no fail state; after 3 runs without success, one part glows as a suggestion; the child can keep tinkering indefinitely (creative mode earns nothing, 02 §2.7).
- **Safety:** circuits are a cartoon battery + bulb only; no mains sockets, no alarm buttons (06 S.ENG.05, 00 B8).

#### MX-H12 `rune-splitter` (מפצל הרונות) — 04 `rune-splitter`; the core K2 predictor `H.PA.PH.SPLIT`
- **Ages · Skills · Input:** K2 (K1 as review in R-03): split a CV into consonant + vowel (`H.PA.PH.SPLIT`: "מָ — what's the first sound?" → /m/), identify and swap the vowel (`H.PA.PH.VOWEL`: "מָ and מִ, what's different?"), segment CVC into phonemes with Elkonin boxes (`H.PA.PH.SEG`: דָּג → 3), delete a phoneme (`H.PA.PH.DEL`: "סַבָּא without /s/" → אַבָּא). Input: tap **sound buttons** (speaker icons that play /m/, /a/, /s/; never letters-as-text for the PA items), drag a **vowel tile** onto a rune, tap boxes.
- **Rules:** a CV rune (pre-baked glyph, §2.3) blocks a gate. Tap the sound button of its first sound → the rune **splits** into a consonant half and a vowel half with a satisfying crack; the gate opens. Shō: "which vowel is inside?" (drag the vowel tile to rebuild a different rune: מָ → מִ, the picture changes from "מָה?" to "מִי?"). Ten: Elkonin boxes — a CVC word's picture sits on 3 boxes; tap a box per sound in order (RTL); each box lights with its sound. Ketsu: deletion — "קוֹף without /k/" → the rune loses its first half and becomes אוֹף (nonsense is fine; ניצי giggles, the friend comes out anyway).
- **Learning = action:** the split *is* the key; a correct first sound physically breaks the rune. Blending (MX-H02) is the reverse motion, so the two mechanics share the anvil/rune art and the child sees "build" and "break" as one idea.
- **Feedback:** every sound button plays the human-recorded phoneme (00 E9 HREC, never TTS for isolated consonants); after a correct split both halves say their sound, then the whole rune says the CV again (segment → blend check). Wrong button: the rune hums its real first sound (informative), no buzzer.
- **Knobs:** CV set (from 04's letter blocks B1–B5 and vowel order), number of sound buttons (2 → 4), whole-CV vs. first-sound distractors (`EH.PA.CVWHOLE`, `EH.PA.ADDVOWEL` from 04's catalog), CVC length (2–3 boxes), deletion position (initial only for K2).
- **Failure:** after 2 errors the correct button pulses while ניצי stretches the sound ("mmm… מָ"); after 3, a worked example with the mouth-shape card (04) and an easier CV. Never more than 3 in a row (AC-LV-6).
- **Movement/Evening:** MV-02 clap variant for CV count; evening compact form: a shadowling shows a rune, tap its first sound to light it.
- **Priority:** Tier B, shipped in S0 — the first non-MVP mechanic to build.

#### MX-H13 `twin-detective` (בלש התאומים) — 04 `twin-detective`
- **Ages · Skills · Input:** K2: look-alike letter pairs (`H.LK.VIS.<PAIR>`: ד/ר, ב/כ, ה/ח, ו/ז, ג/נ…; a pair is served only when **both** letters are ≥70% known, 04), final forms (`H.LK.FIN`), dagesh and shin dot (`H.LK.DAG`, `H.LK.SHIN`), letter vs. digit vs. picture (`H.PR.UNIT`). K1: print ↔ handwriting (`H.HW.CURS.REC`). Input: drag a magnifier over a row (the "lens" from MX-H01 re-used as a detective's glass) + tap.
- **Rules:** the shadowlings swapped "twin" letters on the camp signs. A row of 6–8 runes; find **all** the ד (distractors ר). Each correct find pops the rune into the jar; the magnifier **animates the distinguishing feature** (the ד's corner vs. the ר's curve; the dot of שׁ on the right) when hovering (04: "animation that highlights the distinctive feature"). Ten: memory pairs (ד ↔ דָּג picture, ר ↔ רַכֶּבֶת). Ketsu: "fix the sign" (שָׁלוֹמ → שָׁלוֹם).
- **Learning = action:** only the real twin lifts under the glass; a wrong one "slides" and shows its own sound + its feature.
- **Feedback:** a found rune says its sound (sound-first, 00 B6); a wrong tap plays the distractor's sound and highlights the feature difference side by side for 1.0s.
- **Knobs:** pair (from 04 §2.3א), row length (4–8), number of targets (1–4), feature animation on/off, K1 handwriting fonts (2 hands).
- **Failure:** after 2 errors the feature animation stays on; after 3, ניצי points at one target and the row shrinks to 4. Pair-specific errors feed 04's `EH.VIS.<PAIR>` (never a merged "visual" error).
- **Evening:** compact form — a shadowling wears a letter mask; tap the matching twin on the sign.

#### MX-H14 `writing-studio` (סטודיו כתיבה) — 04 `writing-studio`; not tracing (04 P8)
- **Ages · Skills · Input:** K2: invented spelling with letter tiles (`H.SP.INV`: "how do we write מָה?" → the first sound's letter, then a letter per CV), short decodable words with final forms (`H.SP.CVC`: יָם, דָּג, שָׁם), labelling a drawing (`H.WC.EM.LABEL`), **joint writing with a parent** (`H.WC.EM.JOINT`, Aram & Levin: 7 weeks × 3/week). Input: drag tiles into slots; parent prompts on screen (same layer as K3 lap prompts, §15.3).
- **Rules:** a studio tent in the neighborhood. A purpose first, never "write this word": a birthday card for a friend, a shopping list for Shabbat, a sign for the room, a "letter to the campfire" (04 §14.4). The child **says** the word, taps the sound button per sound, then drags a tile for each sound. The parent-mediation script runs on screen for the adult ("שאלו: מה הצליל הראשון? תנו לו למצוא את האות"), one prompt per step, 04 §14.4 wording. **Invented spelling is expected and accepted** (`EH.SPL.PHON` is logged as "expected", never marked wrong in gan). The finished card/sign appears **in the world** (on the tent, in the mailbox of the friend, on the campfire).
- **Learning = action:** the written word *is* the object (the sign is readable in the neighborhood; the shopping list fills the camp kitchen).
- **Feedback:** the studio reads the child's tiles back exactly as written (phonetically, from the HREC bank) so the child hears what they wrote; no red marks. For K2-advanced CVC items with a final-form slot, the slot shape hints the final letter.
- **Knobs:** tile set (letters known ≥70%, plus 2 unknowns), slots shown/hidden, parent-mediated vs. solo (solo only after ≥3 mediated sessions), purpose template.
- **Failure:** no fail. If the child stalls 10s, the sound button of the next sound pulses; after 20s the parent prompt suggests "say it slowly together".
- **Phygital (00 C10):** every studio piece can be printed (rune-scroll) and the real handwriting happens on paper; in-app accuracy is never the measure. Safety tags per C13 on the paper mission.

#### MX-H15 `word-machine` (מכונת המילים) — 04 `word-machine-oral`, `agreement-builder`, `sentence-lab`
- **Ages · Skills · Input:** K2 **oral** morphology: one → many (`H.MO.OR.PL`), m → f (`H.MO.OR.GEN`), diminutives (`H.MO.OR.DIM`), word families/professions (`H.MO.OR.FAM`), verb agreement (`H.MO.OR.VAGR`). K1 **written**: agreement-builder (drag forms so the sentence "stands"), sentence-lab (grammar/punctuation). Input: tap the picture/choice (K2, voiced), drag word tiles (K1).
- **Rules:** ברגית's contraption has an input hopper and an output chute with a lever: put in "יֶלֶד", pull the "הַרְבֵּה" lever → "יְלָדִים" comes out (voiced) and the matching picture multiplies. Levers: הרבה / נקבה / קטנטן / "מי עושה את זה?" / הוא-היא-הם. K2 chooses the output from 2–3 voiced options (the machine "proposes", the child picks; a wrong pick makes the machine produce a silly object, e.g., one giant child instead of many). Twist: irregulars (אִשָּׁה → נָשִׁים) marked as "the machine's secret drawer". K1: a sentence on a bridge of planks; drag the agreeing form (גָּדוֹל/גְּדוֹלָה) so the plank fits and the bridge holds.
- **Learning = action:** the transformed word is what appears in the world (many children run out; the bridge holds).
- **Feedback:** the machine always says the full pair ("יֶלֶד — יְלָדִים") after the answer; K2 never sees text for this mechanic (oral skill), only pictures + voice. All forms come from the approved lexicon (00 B3/B4), never generated.
- **Knobs:** lever set, regular/irregular ratio, options (2–3), K1 sentence length, punctuation items.
- **Failure:** after 2 errors the correct output is said slowly and the child just taps it; after 3, worked example. `EH.MO.PLURAL_IRREG`, `EH.MO.GENDER_AGR` from 04's catalog.
- **Evening:** compact form — a shadowling holds "one"; feed it "many" to light it.

#### Names used by 06/07 that are not catalog IDs (mapping, F4 #15)

07 §7.1 and 06 §4.5 name activities by their own logical names. They map onto the catalog as **modes/variants** of existing mechanics, or are **FS screens** (07-owned UI that is not a level gimmick and therefore outside the "one gimmick per level" rule). `skills.json` / 07's activity files may use either name; `mechanics.aliases.json` (04's alias table, extended) carries the mapping.

| 06/07 name | Catalog | Notes |
|---|---|---|
| `blicket` | `MX-X02:blicket` | Content skin on the test-it lab (already adopted) |
| `fair-test-builder` | `MX-X02:fair-test` | The "change one thing" variant of MX-X02; band S |
| `breath-mic` | `MX-S02:mic` | Lantern breath with optional microphone onset (RMS only, no recording; AC-MV-4 applies) |
| `ladder` (bravery) | `MX-S06` | Parent-ticked steps; end-of-ladder gem per 02 (`braveryLadderComplete`, never announced) |
| `build-sim` | `MX-X05` | Adopted |
| `code-strip` / `code-stack` | `MX-T02:strip` (band G, RTL per 00 B9 (a)) / `MX-T02:stack` (band S, vertical); simple 3-card strips in K3/K2 = `MX-A01` | Arrows are absolute screen directions (00 B9 (b)) |
| `map-place` | `MX-X03:place` | Place a card on the Israel/world diorama |
| `shelf-3`, `route-planner`, `passport`, `season-track`, `rule-builder` | **FS screens** (`FS-SHELF3`, `FS-ROUTE`, `FS-PASSPORT`, `FS-SEASON`, `FS-RULES`), owned by 07 | Not level gimmicks; they appear inside `ritual_mid` / FS episodes and in the hub. A screen that later carries a level gets an `MX-*` id in this catalog first |

#### 7.4 English content asset IDs (what 12 §6.10.1 produces for this document; `LICENSES.json` and pack manifests use these exact ids)

All ids are kebab-case under `packages/content/en/` (§0 conventions). `<nn>` is zero-padded.

| Asset family | ID pattern | Count at S0 | Used by |
|---|---|---|---|
| Bottles from the sea | `en/bottles/bottle-<nn>` with clips `en/bottles/bottle-<nn>/song.m4a` and `en/bottles/bottle-<nn>/word-<k>.m4a` (k = 1..3), plus `card.png` (the bottle's picture) | 15 bottles (S0: `bottle-01…15`; CANDLES = `bottle-11`, ABC = `bottle-01`, K1 placement letter = `bottle-06`) | §13.2 S0, §5.3 `english_bottle` |
| Tricky-word clips (MX-E05) | `en/mx-e05/tw-<word>` (e.g., `tw-the`, `tw-said`) with `say.m4a`, `beam.png` | per 05 `TW.*.READ` list | MX-E05 lighthouse |
| Decodable ship-log pages (MX-E06) | `en/mx-e06/log-s<stage>-<nn>` (stage = 05 phonics stage, `log-s2-01` = "Pip sat.") with `page.png`, `read.m4a` (Sunny), `words.json` | per 05 `RD.*` | MX-E06 |
| Dock-talk dialogue trees (MX-E07) | `en/mx-e07/dock-<topic>-<nn>` (`dock-hello-01`) with `tree.json` and `lines/<lineId>.m4a` (speaker `sunny` or `mixup`) | per 05 `OR.S.*` | MX-E07 |
| Minimal-pairs bank (MX-E01 `pairs`) | `en/pairs/<word>.<voice>.m4a`, voice ∈ `h` (human, 12 session A), `f1`, `f2`, `m1`, `m2` (the 4 adult TTS talkers) | ~70 words × 5 | MX-E01 `pairs` (AC-MX-8) |
| Phonemes, letter names, nonsense words | `en/hrec/ph-<ipa-safe>` (e.g., `ph-th-voiceless`), `en/hrec/ln-<letter>`, `en/hrec/nw-<word>` | 45 / 26 / ~60 (05 §6) | MX-E03, MX-E01 |
| Songs | `en/songs/<song-id>` (`vocals.m4a`, `backing.m4a`, `lyrics.json`) | ~27 (05) | MX-E02, bottles |
| Pip (CH-11) | `en/char/pip/<state>` (`idle`, `wave`, `click`, `carry`) as Rive artboards + `sig.m4a` (2-note click) | 1 sheet | MX-E06, island |
| Mr. Mix-Up (CH-04 island form) | `en/char/mixup/lines/<lineId>.m4a` (en-US "friend" voice only), `en/char/mixup/costume` (Rive) | per 05 `MIXUP` lines | R-08 boss, MX-E07 |
| Wordlings (sparkling.word) | `en/wordlings/<word>` (`star.png`, `say.m4a` in Sunny's voice) | one per vocabulary word | §4.11, collection room |

Rules: every clip carries `engine:"human"` or `engine:"tts"` in the manifest (phonemes, letter names and nonsense words are human-only, AC-MX-3); no asset under `en/` is referenced from a Hebrew scene (AC-MX-3); a missing `en/` asset falls back per DD-38 and logs `asset.missing`.

---

## 8. Learning through movement (G6)

### 8.1 Principles

1. **Task-relevant movement:** the body action *encodes the answer* (left/right = smaller/bigger, a jump = a count, acting out = the meaning of a word). Exercise as a "break" alone isn't counted as a learning moment (`learning_through_movement` §1).
2. **Moderate movement + easy retrieval.** A vigorous burst (≤60s) is followed by a calm task (Egger 2018, via the note). Never a hard item during vigorous movement.
3. **Every movement moment has 2 fallbacks:**
   - `seated` (arms, head, fingers);
   - `tap` (tapping on the screen).
   - The selection is automatic per `profile.movementMode ∈ {full, seated, tapOnly}` and also per moment (a "sit" button in the HUD).
4. **Sensing by phase:**
   - **Phase 1:** none (self-report: the child taps "done", or a parent taps) plus `expo-sensors` for an iPhone in the hand (tilt/shake) plus microphone onset for claps (RMS, no recording).
   - **Phase 2:** front-camera pose (Apple Vision via a custom Expo Module, dev build; frames in memory only).
5. **Safety:**
   - A one-time setup screen for the parent (2×2 m of clear space, grippy socks or bare feet, the device on a stand 2–3 m away or leaning against a wall).
   - **Never jumping with a device in the hand.** Before any jump moment (MV-03, MV-04, MV-11) the app checks `expo-sensors` DeviceMotion for 1.5s (00 E3): if the device is moving (acceleration variance above `motion.inHandThreshold`), it shows "הניחו את המכשיר" and waits; after 10s without stillness it switches to the `seated` variant automatically. iPhone defaults to `seated` for all jump moments unless the parent marks "on a stand" (OPD-14).
   - One active player in the camera zone at a time.
   - **Real-world/offline missions carry safety tags** from 00 C13 (`adult-required`, `no-flame`, `no-mains`, `choking-hazard-T`, `outdoor-with-adult`, `food-allergy-check`, `photo-on-device`); the content linter rejects a mission without tags. While a child under 3 lives at home (K3 until ~May 2027), no mission for any child uses small parts or small magnets (MV-04 floor markers are printed paper, MX-M19 offline tangram is a printed sheet, MV-05 uses feet only).
   - Quiet mode: no jumps (marching in place / arms) by default from 20:00, on Shabbat if it is on, and in a parent-set "apartment" mode.
6. **Frequency:**
   - K2: 1 moment per level (≤45s), with a total of ≤2:00 per episode.
   - K1: 1–2 per episode (≤60s each).
   - K3: 1 "copy me" (30–40s) with the parent.

### 8.2 Catalog of movement moments

| ID | Name | Ages | What you do | What it encodes | Seated / Tap | Phase-1 sensing |
|---|---|---|---|---|---|---|
| MV-01 | "תעשו כמוני" | K3 (+K2) | Copy movements in a song (clap, arms up, stomp, a cat walk) | Body parts, actions, a word–action match | Arms only / tap on the matching picture | None (the parent taps "עשינו!") |
| MV-02 | "מחא כפיים להברות" | K2 | Clap the syllables of a word (רַ-כֶּ-בֶת = 3) | Phonological awareness | Taps on the knee / tap the drum | Microphone onset (RMS > threshold, debounce 180ms) |
| MV-03 | "קפוץ את המספר" | K2 | Jump N times to charge the jet | Counting, 1:1 correspondence | Raise your arms N times / tap N | Self-report (Phase 2: pose) |
| MV-04 | "קו מספרים על הרצפה" | K2, K1 | A taped line on the floor (printable markers); hop to the number / fraction | Number line, magnitude | Point on the screen | Self-report: tap the landing place |
| MV-05 | "מודדים ברגליים" | K2 | Walk heel-to-toe along a hallway and count | Non-standard measurement | Measure with the hand on a table | Self-report: type the number |
| MV-06 | "ריקוד הקפאה" | K2 | Dance; the music stops on a rune and you find something in the room that starts with it | Initial sound | Point at a picture | Parent confirm / photo (Phase 2) |
| MV-07 | "הטה כדי לנווט" | K1 (iPhone in hand, standing still or seated) | Tilt to guide a light ball through a maze toward the right answer | Choosing an answer, spatial orientation | Drag | `DeviceMotion` |
| MV-08 | "Sunny Says" | K2, K1 | Act out English verbs (jump, sit, wave, turn around), only after "Sunny says" | Comprehension of English verbs | Arm movements / tap on a picture | Self-report (Phase 2: pose) |
| MV-09 | "שמאלה קטן, ימינה גדול" | K2, K1 | Lean or step left/right: the smaller or bigger number (K1: bigger/smaller fraction) | Magnitude comparison | Tilt the head / tap a side | Phase 2: shoulder center x; Phase 1: tap |
| MV-10 | "נשימת פנס" (with the body) | everyone | Arms up on the in-breath, down on the out-breath | Calming (after a vigorous burst) | Same seated | Optional microphone |
| MV-11 | "100 קפיצות משפחתיות" | Family | Everyone together jumps or claps toward a shared goal | Counting to 100 (K1: by tens) | Claps | Self-report / microphone |

**Self-report** in Phase 1 isn't "cheating". The point is the body–content experience. Rules:
- The moment never blocks progress.
- The game doesn't check "did you really jump".
- Self-report gives no coins.

---

## 9. Battles, bosses, the runner and the evening

### 9.1 Battle system: lighting shadowlings (MX-B01)

**Principle:** the answer *is* the attack (Zombie Division, C1).
- The player has no HP and doesn't get "hurt".
- The shadowling has **shadow layers** (`layers`), not HP. Each correct answer peels off a layer with a beam of light from the פנסון's core.
- Once all the layers are gone, the shadowling lights up, smiles and **becomes a friend** who joins (redemption, `content_world` §b1).

| Type | Layers | Content | Behavior |
|---|---|---|---|
| `shadowling.tiny` | 1 | 1 item | Stands still, giggles |
| `shadowling.puffy` | K2 3 / K1 4 | Items of the same skill | Floats back and forth slowly (not toward the camera) |
| `shadowling.stack` | 3 small ones in a tower | 3 different items (interleaved) | Each one wobbles in turn |
| `shadowling.mirror` (K1) | 2 | Twist: the item comes back in a different representation (7×8 → 8×7 → 56÷7) | Shows a "mirror image" |
| `shadowling.sleepy` (K3) | 0 | No item | "Tuck in" with a blanket = 1 tap, and it glows |

- **How the "attack" works:** the item appears in the HUD (or in the world as a glyph texture or object).
  - Answering = aiming the light: tap on the right answer in the world, or on the keypad (K1).
  - The beam travels 0.25s, hit-stop 60ms, then the layer "peels off" (squash + particles).
- **Wrong answer:** the beam bends to the side and lights a nearby flower instead (funny, no penalty). The shadowling giggles. The same item comes back after 1–2 other items.
- **Combo (K1 only, opt-in):** 3 in a row = a "light wave" that peels 1 layer from all the shadowlings in the area. It only shows for skills at 08 level `proficient` or above and `fluencyEligible` (D11; 08 §4.4).
- **Hero "classes" (attack styles, cosmetic + verb flavor), K1 only, chosen in the Plan step:**
  - `style.archer` (a beam)
  - `style.builder` (building a wall of light in front of the shadowling)
  - `style.singer` (a sound wave)
  - Unlocked at milestones. **No difference in power or rate.**

### 9.2 Region bosses

- At the end of each region (after ~6 levels): **a boss of 3 phases**, each phase a "mini level" of the region's gimmick, lasting K2 3–4 min and K1 5–7 min in total.
- **The bosses are the Muddler's creatures** (an elemental "muddle" for each region):
  - R-03: "ענק העירבוביה", who swaps the order of letters.
  - R-05: "דרקון הגבישים העקום", whose crystals are mixed up.
  - and so on.
- **Structure:**
  - Phase 1: the region's gimmick alone.
  - Phase 2: the gimmick + a twist from the Ten beat.
  - Phase 3: mixed with an earlier region (interleaving).
- **Between phases:** 0:20 of "spectacle" (the boss laughs and stumbles), with checkpoints at the start of each phase.
- **Defeating it:** the boss shrinks, laughs, and gives back a "big lost friend" (VIP, the region's `bossFriend`, counted in the collection room) plus a **gold chest** (02 §6.2 `regionBossDefeated`: K1 80 coins + 1 gem + choice of 1 of 3 previewed items; K2 8 coins + choice of 1 of 2).
- **Rescue count per region (binding, mirrored in 02 §5.1):** `friendsInRegion = levels × friendsPerLevel + 1 (bossFriend)`. With 8 levels: K1 = 8×7+1 = **57**, K2 = 8×5+1 = **41**. The region card and the Lost Friends Board show `rescued/total` as silhouettes (K2: silhouettes only, no numerals; K1: "41/57"). Secret level `secretA` opens at ⌈50%⌉ of the region's friends, `secretB` at mosaic complete, `goldenRemix` (K1) at 100% friends + pieces (02 §5.3).
- **No-fail:**
  - A boss can't "win".
  - If the child struggles (≥3 errors in a phase), the boss "gets tired" and yawns, and the phase moves to Cozy.
  - The child can stop in the middle and progress is saved by phase.

### 9.3 The weekly family boss: "ערב מדורה משפחתי"

- **When:** the parent picks a weekday (default **Thursday**, so it doesn't collide with Shabbat). From the start of the week (Sunday), the boss appears at the edge of the camp with a "challenge banner".
- **Format:**
  - Damage accumulates all week. Each child's evening segments (§5.3) plus their level rescues each lower the boss's "muddle meter".
  - On boss night there is a **joint finale** of 6–8 minutes: a shared screen (same-device) or a session across devices (§12).
- **Muddle-meter formula (handicapping, §12.4):**
  ```
  bossHP = Σ_child ( plannedEpisodesThisWeek[child] × 300 ) × 0.8
  damagePerEpisode[child] ≈ 300 (fixed, regardless of age or speed)
  finaleDamage = remaining HP (the finale always finishes the boss if all the participants took part)
  ```
  So the family wins if each child completes ~80% of their planned episodes. Each child contributes **equally in display terms**.
- **Who counts (F9 GAP-63, binding):** `Σ_child` runs only over children who are **not paused** (10 FAM-03 `profile.paused`, 02 §10.1 `effTarget = 0`) **and** who have played ≥1 episode this week by the time the finale starts; a child who was absent all week (sick, away) is simply not in `bossHP` for that week and is never "missing". The **finale completes with any present subset** (even one child; even K3 alone with a parent as `assist`): `finaleDamage = remaining HP` for the children present. An absent child's rescues/evening damage from earlier in the week still count. The boss is never "waiting for" anyone.
- **Boss day on a Yom Tov / quiet day (DD-06):** if the chosen boss day (default Thursday) is a Yom Tov or a quiet day (00 B7a), the finale window moves to the **previous weekday evening** that is neither (e.g., Wednesday); if the whole Thursday–Friday-noon window is blocked, the boss sleeps and carries over (next bullet). No boss ever appears on a quiet day (AC-SHAB-2).
- **If they didn't make it:** the boss "falls asleep" at the edge of the camp with its meter frozen, and carries over to next week. No loss, and no "the boss beat you".
- **A parent can take part:**
  - a recorded clue ("רמז מאבא") that opens the finale, plus 1 tap;
  - or an "Abba/Ima" role in the finale (an extra player in same-device mode, giving light to the weakest one, `assist`).
- **Reward (numbers owned by 02 §6.2, trigger `bossDefeated`):**
  - K1: a **silver chest** (40 coins + 1 gem + the next item on the silver track). K2: a **wood chest** (3 coins + next item). K3: nothing extra (participation is the reward; the family envelope comes with the weekly goal).
  - +1 family fire level (the boss and the weekly goal are two separate level-ups; see §10.4). In CO-3 the `boss_defeated{participants}` event is emitted **once by the `CoopRoom` Durable Object** (00 E5, DD-34) with one id; in CO-1/CO-2 the host device emits it once with the shared `coop_session` id (§21 UC-06). Every child's chest derives from that single event (02 UC-17).
  - Everyone: the boss joins as a VIP for that week (a cute version wanders the camp) and the "family treasure" is that VIP plus the fire level, not a separate chest.

### 9.4 The runner: "מרדף הגחליליות" (MX-R01, Temple Run–like)

- **Context:** a shadowling (or the Muddler) grabs a lost friend and runs off. You chase him along the forest path (R-11 or a level's Ketsu).
- **Mechanics (E1b: auto-run, tap-to-jump):**
  - Automatic running on 3 lanes.
  - K2: tap = jump. Obstacles are low and every jump is "generous" (coyote time 150ms).
  - K1: tap = jump, swipe = change lane.
  - **Gates of answers:** every 8–12s a fork of 2–3 lanes, each with an answer (a glyph texture / picture / number). Choosing the right lane = speed + sparks; a wrong lane = a short detour (0:01.5) back to the path.
  - K2 also gets "flying letters": tap the letter that makes the sound → a light magnet.
- **Length:** fixed. K2 60–90s, K1 90–120s. It ends at a destination point: you catch the shadowling, it laughs and returns the friend.
- **No death:**
  - A collision is a 0.5s stumble (`oops_cute`) and slower running for 1.5s.
  - No "revive", no gems for continuing, no endless mode (R6).
- **Power-ups (from the item):** a magnet (pulls sparks), a shield of light (absorbs 1 collision), a jet (a short speed-up). They are earned by a "bonus question" gate, not bought.
- **Co-op:** see §12.3 (one runs, one shoots lights, K3 is a passenger who taps to "cheer" and so adds sparks).
- **Self-competition (K1, opt-in):** a ghost of her best previous run (a translucent figure). The record is shown to her only.

### 9.5 The evening: campfire defense (MX-B02; 99 Nights without fear)

- **Day/night rule (G5, binding):** the camp's time of day is driven by the **episode phase**, not by the wall clock: Arrival/Plan/Warm-up/Levels = `day`, Camp (Show + Build) = `dusk`, Evening = `evening`, Goodnight = `night`. Exceptions: Shabbat scenery (§15.4) and quiet days override the palette; K3 never sees `evening` (§2.3). A child who plays at 08:00 still gets the evening segment at the end of the episode (the fiction is "a day at camp", §5.1). In co-op (§12), the host device's phase is authoritative for all screens.
- **Atmosphere:** dusk–evening, a fire in the center, the rescued friends sitting around it. Shadowlings come out of the forest edge **slowly, at the sides of the screen, not toward the camera**. They're cold and want warmth.
- **The family's goal:** "warm" all the shadowlings (light them with items) before they "steal" a lantern.
  - If a shadowling reaches a lantern, it takes it, giggles and runs off to the forest. **The lantern comes back** the next evening, or with an easy chase.
  - There is nothing to lose: at worst "השמש זרחה והם הלכו לישון" (dawn arrives).
- **Solo structure (inside an episode):**
  - K2: 1 wave of 4–5 shadowlings (1:20).
  - K1: 2 waves of 4–6 (2:00), with a light strategy: place 2 "light towers" (Blooket Tower Defense style) for 2 items each. The tower lights automatically every 6s.
- **Family structure (boss night or a family evening):** 3–4 waves, roles (§12.2).
- **The family campfire (§10.4) widens the lit zone** with each level. This visibly shortens the shadowlings' path (99 Nights' "upgrade expands the safe area").
- **Evening "flame" (modifier, R2):**
  - In the Plan step, K1 can choose a flame color that changes the evening:
    - blue = the shadowlings are slow;
    - green = double friends;
    - gold = a big treasure, but Stretch items;
    - purple = surprise (a random choice among predefined rules, not a reward).
  - K2 gets a simple choice between 2 colors.
- **K3:** "tuck-in". Sleepy shadowlings sit near the fire; each tap covers one with a blanket, it glows and falls asleep. There's no approaching and no "stealing".

---

## 10. Hub, building, collection, campfire, emotes

### 10.1 The hub (R-00 המחנה): a place that fills with life (G2)

- **Structure:**
  - The campfire in the center.
  - **3 neighborhoods**, one per child (K1 on the right, K2 in the middle-left, K3 = an entrance to the playroom). The neighborhood's color is the child's chosen color. **v1 supports exactly 3 children** (F9 GAP-53): the hub layout, the turn ring (§12.5) and CO-1/CO-2 zones are built for three; 11's schema allows N children, so 10 FAM-03 **blocks adding a 4th child** with the message "בגרסה הזו יש מקום לשלושה ילדים" (a v2 item: a 4th neighborhood behind the campfire).
  - Gates to regions around the edges.
  - Buildings that unlock.
- **The rescued friends live in the neighborhood of the child who rescued them:**
  - They walk around, play, cheer when the child walks by, and sometimes "help" at a building (an animation).
  - A friend you poke says its item (light retrieval practice).
  - Above 60 friends, **instancing** keeps the scene within budget (E1b: ≤80 draw calls); only 30 friends are "awake" at a time and the rest are "inside the houses".
- **Start empty and dim** (Kirby's ruins): only the campfire and the tents. Every return makes the camp visibly fuller and brighter (P-3).

### 10.2 Neighborhood building thresholds (per child, fixed and visible, Astro model)

> **Precedence:** the thresholds live in `economy.config.json → collection.neighborhoodThresholds` (02 §5.5, paced by the economy simulation so the last building opens around week 9–13 instead of week 4–6). The table below mirrors the config; if they ever differ, **02 wins** and this table is updated.

| Rescued friends (K2) | K1 | `id` | Building that opens | What it adds |
|---|---|---|---|---|
| 5 | 8 | `friendsHouse` | **בית החברים** (Friends' House) | The friends have a home + the collection room (10.3) |
| 20 | 30 | `musicHut` | **צריף המוזיקה** | Letter/number songs to listen to (a calm menu, no rewards) |
| 50 | 80 | `costumeTent` | **אוהל התחפושות** | The avatar and פנסון wardrobe (cosmetics) |
| 100 | 150 | `chickGarden` | **גן הגוזלים** | The pets' home, the egg nest |
| 160 | 240 | `storyTheater` | **תיאטרון הסיפורים** | A bedtime story (§15.5) + K1's diary pages |
| 240 | 360 | `starTower` | **מגדל הכוכבים** | A "Then vs. Now" view (the journey over the months) |
| 350 | 520 | `creatorsWorkshop` | **סדנת היוצרים** (K1: design mode) | K1 designs a small room/level from chunks (Roblox-like expression) |
| +120 per season | +180 per season | `perSeasonNewBuilding` | **מבנה עונתי** (a new seasonal building each season, e.g., a Hanukkah lantern-house, a Purim stage) | Keeps rescues meaningful after the 7 core buildings |

- **Visible milestones:** a sign with a silhouette of the next building and "עוד 4 חברים" (goal-gradient). No prices, and nothing for sale. Only the child's **own** rescues count (co-op rescues count for each participant, DD-23).
- **Hub fill states (the hub as trophy room, G2).** The hub has 6 visible "fill states" driven by the **family total** of rescued friends across the three neighborhoods; each state adds ambient life, never removes anything:

| Fill state | Family rescued friends | What the camp looks like |
|---|---|---|
| `F0` dim | 0 | Campfire + tents only, mist at the edges, 1 lantern lit |
| `F1` waking | ≥10 | Paths appear, 3 lanterns, friends sit by the fire |
| `F2` lively | ≥60 | Friends walk and play, a mailbox, birds, 6 lanterns |
| `F3` busy | ≥150 | Friends "help" at buildings, music from the hut, bunting between tents |
| `F4` festival | ≥300 | A small stage, fireflies swarm at dusk, friends cheer when any child walks by |
| `F5` legendary | ≥600 | Permanent lantern parade path around the camp, the mist is gone entirely |

  The fill state is a visual layer; it never gates content and never shows per-child numbers (00 C6). Above 60 friends, instancing + "30 awake at a time" keep the frame budget (§10.1).
- **K3:** every 5 toy friends → a new toy in the playroom (02 `toyEveryNFriends`).

### 10.3 The collection room ("חדר האוצרות")

- A 3D shelf/wall per species, with **a silhouette of every friend not yet found** (from the curriculum map, Pokédex style).
- **Endowed progress:** the room opens with 3 friends already present (from onboarding).
- **Goal-gradient:**
  - Near completion of a species, a soft glow appears on the last silhouettes plus "עוד 3!".
  - The engine also makes the last friends easier to find: they appear more often in levels.
- **Poke a friend** → it says its item. **Poke a silhouette** → ניצי gives a hint about where it is ("הוא מחכה במכרה").
- **A completed set of a species** = a T3 celebration + a badge **named after the skill** ("את מכירה את כל אותיות הקבוצה הראשונה", 00 D4) + a chest (K1 silver, K2 wood; 02 §5.4) + an **accessory for the tied pet** (a scarf, a hat). Pet **evolution is by mastery only** (02 §8.2), never by collection, so "I found them all" and "I know them" stay two different things.
- **Capsule machine (02 §5.7):** a hand-cranked capsule machine stands next to בית החברים. It is **deterministic**: the window always shows the **next capsule** (`nextCapsuleVisible`), each series has 8 items in a fixed order with no duplicates, and a new series opens when a region mosaic is completed. The fun is the crank-and-pop ritual (≤2.0s, T2 budget), not chance. No "spin", no rarity colors, no odds. Opens at level 4 (K1) / 10 (K2); price in 02.
- **Pieces (mosaic):** each region has a mosaic of 30–40 pieces that slowly reveals a picture of the region's story. A completed mosaic = a diary page + a decoration for the campfire.

### 10.4 The family campfire (shared sink)

- **Fire level goes up only and never resets** — not per season, not ever (00 C5; 02 §7.6 `resetsPerSeason: false`). One level per:
  - (a) a weekly family goal met (00 C5);
  - (b) a weekly boss defeated;
  - (c) a seasonal event completed (by ≥1 child reaching the event treasure);
  - (d) Lag BaOmer (§13.2) gives +2.
  - **Only those four events raise the level** (02 §7.6, F9 GAP-21). Light contributed in co-op (`coop_light_share`), logs, lanterns (`campfire_lantern_lit`) and offline merges never produce a level-up by themselves. In CO-3 the level-up source is the `CoopRoom`'s single `boss_defeated`; contributions made while a device was disconnected are merged as light into the **next** evening's meter, never as a retroactive level (11 §10.2).
- **Visual ladder:** levels 1–10 are the **flame sizes** of the first "ring" (99 Nights' 6 levels → our 10). From level 11 on, every 10 levels add a **color ring** of embers around the fire (ring 2 = the second season's color, and so on), the fire keeps its maximum size and the lit zone stays at its maximum radius. A new season therefore **adds** a ring color to the fire, it never shrinks it. `fireLevel` is unbounded in config (`campfire.fireLevel.max: null`, `flameStepsPerRing: 10`).
- **Lit zone in the evening** widens with levels 1–10 (radius `6 + 0.6·level` tiles, capped at 12) and shortens the shadowlings' path (§9.5).
- **Region unlocks (binding rule; replaces the vaguer "fire opens regions"):** a region opens when **both** hold:
  1. **Story gate** — the earlier of: its chapter week has started on the calendar (§2.4 arc, `calendar.chapters`), **or** `fireLevel ≥ regionUnlocks.fireLevel[R]`. Fire can open a region **early**; the calendar guarantees it opens **at the latest** on its chapter week, so a family that misses weekly goals never loses access to learning (02 §5.2 rule 5: nothing in the curriculum path is gated behind collection or shared counters).
  2. **Readiness gate** — the child has ≥1 skill in the region's domain at 08 level `new`-with-prereqs-met or above (the engine's "fringe"). Until then the gate shows ברגית's "עוד בבנייה" (never a lock, never "not good enough").

  | Region | `fireLevel` early-open | Calendar guarantee (chapter) | Notes |
  |---|---|---|---|
  | R-00, R-12, R-10, R-11 | 1 (launch) | Week 1 | Hub, K3 playroom, Tree of Quiet (SEL always available), runner (as level Ketsu) |
  | R-03, R-05 | 1 (launch) | Week 1–3 | The two starting regions (S0) |
  | R-04 | 4 | Week 5 (S1) | Fog lake; K1 only until K2 starts grade 1 (DD-18) |
  | R-08 | 5 | Week 6 (S1) | English island; before it opens, English arrives via "bottles from the sea" (§13.2 S0, §7.2 MX-E01) |
  | R-06 | 6 | Week 7 (S1/S2) | ברגית's workshop |
  | R-07 | 8 | S3 start | Observatory |
  | R-09 | 9 | S3 mid | גלגלון's garage (AI literacy); MX-A01 simple cards are available earlier inside R-05/R-06 levels |

  The table is `calendar.regionUnlocks` in config; the parent can open any region manually from the parent area (audit-logged).
- **Campfire decorations:**
  - Benches, lights, a family flag, a hanukkiah at Hanukkah, a sukkah at Sukkot.
  - Bought with **"עצים למדורה" (logs)**, the shared unit of 02 §7.6: K1's give jar (10 coins = 1 log) and gems (1 gem = 5 logs), K2's give button (1 coin = 1 log), K3's episodes (3 logs each, automatic, the "lighter" role), and +20 logs per weekly goal. Family projects (benches → lantern string → flag → stage → swing → light tower) run in a fixed visible order, one active at a time (02 §7.6). Only the family total is shown to the children, never who gave how much (00 C6).
- **The family garden (07 §1 #8, 00 D1a) sits next to the campfire**, on the side away from the log pile: a small bed where a flower/stone/lantern appears for every **parent-confirmed kindness or team act** (`kindness_confirmed`). It is a **cosmetic layer only**: no coins, no logs, no fire level, no counter shown to the children (00 C6: a shared garden, never per child), nothing wilts (02 §2.7 `neverEarns`). It is rendered by the hub scene like a decoration; 06's weather station and lantern stand beside it.
- **The weekly family goal:** "3 adventure days per child this week" (the parent sets 2–5 per child; K3 default 2). Shabbat and holidays don't count and don't hurt. The meter is shared, and the display is "the family's light" without separating children.

### 10.5 Building in the neighborhood (building system)

- **The neighborhood grid:**
  - K1: 14×14 tiles.
  - K2: 10×10.
  - K3: none (the playroom is fixed, with toys).
- **Two kinds of building:**
  1. **Blueprints (catalog):** decorations and small buildings that open by milestones and are bought with coins (fixed prices, §11). Placement is free: grid snap, rotate in 90° steps, "undo".
  2. **Build Missions:** ברגית asks "build a bench 6 blocks long", "a fence around a garden of 5×4" (perimeter), "a floor of 24 tiles in 3 rows" (array).
     - **The building itself is the learning action** (MX-M05/M10). A Build Mission doesn't cost coins and creates a unique "project" item.
- **Free creative mode:** in K1's "סדנת היוצרים", the materials from Build Missions are free to use for construction without right/wrong.
- **Rules:**
  - Nothing breaks or decays.
  - No "attacks" on the neighborhood.
  - A sibling can't change another child's neighborhood. A sibling can *visit* and leave a "heart" emote.

### 10.6 Emotes (R9)

- **Wheel of 4 slots** (in co-op and the hub). Each emote is an avatar/פנסון animation of ≤1.2s + a sound.
  - Cooldown: 3s.
  - In multi-device co-op, the emote is shown to the siblings. **This is the only channel between siblings** (no chat, 00 C7).
- **Launch catalog (12):** 👋 hello, 👍 well done, 🙏 thanks, 🆘 help me, 👀 look here, 😂 silly dance, 🎉 hooray, 💡 I have an idea, ❤️ heart, 😴 tired, 🌀 spin, 🤝 high five (needs two).
- **Earned more:** at milestones and from treasures (~2 per month).
- **Banned:** taunting emotes ("ha-ha", a tongue, "noob", "you lost"), and anything that marks failure or ranks.
- **Teaching "how to use emotes":** ניצי models "תודה" after a sibling helps.

---

## 11. Economy hooks (00 D1–D6)

### 11.1 In-session resource: "sparks" (not a currency)

- Sparks (`spark`) are the breadcrumbs in levels. They light the core and in the evening "load" towers or attacks.
- **They reset at the end of the episode** and can't be accumulated. **No conversion to coins.** This keeps 00 D1 (coins for mastery, not time). See the Proposed change to 00 for the clarification.

### 11.2 Numbers: see 02 (no economy numbers live in this document)

> **Binding precedence:** `02-economy-progression.md` is the **sole owner of every economic number**: earn values per event (02 §2.2–2.5), diminishing returns and caps (02 §2.6), what never earns (02 §2.7), the end-of-episode receipt (02 §2.8), XP and levels (02 §3), chest tiers, triggers, contents and reveal timings (02 §6), shop catalog, price bands, showcase, wishlist, jars and save bonus (02 §7), pets (02 §8), season track (02 §9), weekly family goal (02 §10), real-world rewards (02 §11). Earlier drafts of this section carried a numbers table; it was removed so there is exactly one source. Anything a developer needs numerically is in `economy.config.json` (02 Appendix A).

What this document still **requires** of the economy (game-design intent, all already satisfied by 02):
1. **Mastery and effort pay, time never does** (00 D1; 02 §2.7 `neverEarns`). The engine emits learning events; the economy is a pure fold over them (02 §2.1).
2. **A rescue pays no coins.** The friend is the reward (02 §2.2 `rescue: 0`).
3. **Receipt after, never contract before** (Lepper; 02 §2.8). No coin counter in the HUD during a level (§6.4). The receipt's "process lines" include the effort marks from §16.2 (`firstTry`, `hintFree`, `braveChoice`, `mentor`) as **words, not value**.
4. **K2 coins are small "effort chunks"** that he can count aloud (≤4 per episode, feminine counting forms, 00 B4; 02 §2.3).
5. **K3: stickers only**, no coin counter, hidden XP for the parent only (02 §1).
6. **Milestones pay through chests** whose contents are fixed at earn time and previewed (00 D2; 02 §6.3).
7. **Sparks are not a currency** (§11.1).
8. **Shop is cosmetic only, catalog permanent, showcase weekly on Sunday**, no countdowns, no discounts (00 D3; 02 §7.3).
9. **Jars:** K1 spend/save/give, K2 spend/save + a "give" button; the give jar feeds the campfire's log pile (02 §7.5–7.6).

---

## 12. Sibling co-op

### 12.1 Co-op modes

| ID | Mode | Devices | Who | Length | Content |
|---|---|---|---|---|---|
| CO-1 | **"שני צדי השולחן"** (Face-to-face) | 1 iPad landscape, split into 2 halves, each half **rotated toward its player** (180°) | K1+K2, K2+K3 (with a parent), K1+K3 | 5–8 min | Campfire evening, boss finale, building together |
| CO-2 | **"כתף אל כתף"** (Side-by-side) | 1 iPad, one shared screen, roles with separate zones (right/left) | Any pair or three, + a parent | 5–8 min | The runner (one runs, one lights), the evening, a level in co-op |
| CO-3 | **"מכשירים מחוברים"** (Multi-device) | 2–3 devices on the same family account, a session over WebSocket (see Proposed change to 00) | K1+K2(+K3 on a parent's phone/iPad) | 5–10 min | The weekly boss, the evening, Mission Control (K1 commander → K2 artist-bot) |
| CO-4 | **"מכתב לאח"** (Async) | Any | K1 → K2 | — | K1 builds a riddle or puzzle for her brother (Walkington problem-posing); K2 solves it in his episode. A heart emote goes back. **Structured authoring only (00 C7a; 15 GAP-46):** the letter is composed from closed chips (question frames), numbers, and pictures/items from the installed pack — **no free text, no keyboard, no drawing** in the letter; it is stored as a `coop_letter` doc `{from, to, templateId, slots[], createdAt}` (11 §6.1) and rendered from the template on K2's device, so nothing unreviewed can reach him |
| CO-5 | **"הורה בחדר"** (Parent co-pilot) | 1 device + a parent prompt on screen | Every child (K3 always) | — | Lap-mode / co-play prompts (§15.3) |

- **Drop-in / drop-out** in every mode (LEGO, R5):
  - A sibling can join or leave at any moment without stopping the game.
  - A player who leaves is replaced by an "AI-free" friend (a scripted ניצוץ) that keeps their role at a basic level.
  - **Parent-time accounting:** co-op time counts toward each child's daily cap at **50%** (gap §4.6), so co-op is never "time stolen".

### 12.2 Role asymmetry (Hazelight + Pais et al., R5)

The roles are **different, not ranked**. Each has a heroic name and an equally impressive visual (the Mario Wonder lesson). There is **information asymmetry**: each child has something the other needs.

| Role | Child | What they do | Input | Content | Information only they have |
|---|---|---|---|---|---|
| **מַדְלִיק/ת הָאוֹר** (Lightkeeper) | K3 | Tap on glowing items → sparks to the fire; tuck in sleepy shadowlings | Tap only, anywhere | Colors/animals/1–3 (all correct) | — (pure contribution; *the fire's light* depends on them) |
| **הַגִּבּוֹר/ה הַסַּיָּר/ת** (Scout-Hero) | K2 | Moves the פנסון in the world, finds friends, attacks with letters/numbers | Tap-to-move, tap | Letter sounds, counting, adding within 20, English TPR | **Sees the hidden glints** (only his screen/zone shows the secrets) |
| **הַבּוֹנָה/ה הַמְּפַקֶּדֶת/ת** (Builder-Commander) | K1 | Reads clues (aloud to K2), builds towers and bridges, solves the "lock" | Full | Reading, ×÷, fractions, logic | **Reads the map and the clues** (only her zone shows text) |
| **רֹאשׁ/ת הַמַּחֲנֶה** (Camp Elder) | Parent (optional) | A recorded clue, "assist" (light to the player who is stuck), turning the page in a story | Tap | — | Co-play prompts |

**Example of "everyone is needed"** (the evening, CO-2):
1. K1 reads "the shadowling on the left wants the syllable מָ" and says it aloud.
2. K2 finds the מָ rune on his side and shoots light.
3. K3 taps the fire so that the light is strong enough.
4. If K3 isn't there, the fire is lit automatically at the base level (the game doesn't wait for anyone).

**Touch zones and `assisted` tagging (03 §13.7, 04 tutor mode; binding for CO-1/CO-2):**
- Every same-device co-op layout gives each child a **separate touch zone** (CO-1: the rotated half; CO-2: a named region ≥1/3 of the screen with a 12 mm dead strip between zones; K3's zone is always the one nearest the fire). A tap outside a child's own zone is **not an answer** for anyone: K3's taps on K2's items are ignored (they only feed K3's own glow targets), and K1's taps on K2's items are ignored unless K1 is in guide mode (and even then only hint tokens, §12.6).
- An item answered with help is tagged in its `item_answered` event: `assisted: 'parent' | 'sibling' | 'none'`. `sibling` is set only when the **parent marks it** on the co-play prompt ("עזרתי לו / אחותו עזרה"), never inferred from touch; `parent` is set by the parent's own tap on "I answered". The engine weights assisted items (08 §4.1, w ×0.3) and the economy pays them as `itemRecovered` (02: no penalty, no bonus). No flag is ever shown to a child.
- In `MX-B02` the shared light meter sums each child's own-zone correct answers only (§12.4); a child outside their zone contributes nothing and loses nothing.

### 12.3 Co-op in the runner (CO-2)

- **K2 = the runner** (tap-to-jump).
- **K1 = the "navigator":** chooses the lane at answer gates (×/reading). Her choice changes the lane for both of them, so they talk ("שמאלה!").
- **K3 = the "cheerleader":** each tap on their side of the screen fills a "cheer meter", which triggers a 3s magnet (always positive).
- **Without K1:** the gates are answered by K2 at his level (pictures/letters).

### 12.4 Handicapping (making the contributions fair)

**Principle:** **each child contributes "100 units of light" per active minute, regardless of age or speed.**

```
lightPerCorrect(child) = 100 / baselineCorrectPerMin(child)
baselineCorrectPerMin = median over the child's last 5 sessions (defaults: K3 6.0, K2 4.0, K1 5.0)
hintedCorrect → 70% of lightPerCorrect;   wrong → 0 (no subtraction ever)
K3: every tap on a glowing target counts as correct; rate-limited to 1 per 0.8s (so a toddler can't "spam" more than others)
sharedGoal(evening) = Σ_activeChildren(activeMinutes × 100) × 0.85
```

- **Item difficulty:** each child gets items at their own success band (00 C2: K3 90–95%, K2 ~85%, K1 75–85%), selected by the engine as usual. Co-op doesn't change the band.
- **"Synergy moment":** if all active children hit within a 10s window, the team gets a "united light" (a joint T1 + an extra spark). This encourages coordination, not competition.
- **No display of individual contribution** to the children. Only the shared meter is shown. Individual contribution goes to the parent only (and only in aggregate: "שלושתם השתתפו").
- **Fairness criterion (telemetry, AC):** in every co-op session, each child's share of light stays within ±25% of an even share (100/N%). If it deviates: the next time, `lightPerCorrect` is tuned (closed loop, `coop.balance.k = 0.2`).

### 12.5 Turn-taking (one device, three children)

- **"Turn ring"** on the profile-selection screen: a ring of 3 avatars, with a soft glow around "whose turn is next" (by order of last use). No time is shown to the children; the time is visible to the parent only.
- **Switching profiles saves instantly** (00 E6, local event log). A forced switch loses nothing; the child returns to the same beat.
- **Co-op counts for both** (at 50%, §12.1). **No "stealing" time:** an episode that was stopped mid-way "for a sibling" gives a "Patience heart" to the one who yielded, and their next episode starts with a VIP friend surprise (an unexpected reward, not a pre-announced one).
  - **Detection rule (F9 GAP-33, binding):** the device emits `turn_yielded{childId, sessionId, nextChildId}` when (a) a session ended with `reason:'parent_exit'` (08 §1.10's "switch") **before 80% of its planned minutes**, and (b) a **different** child started a session on the **same device within 120 s**. At most **1 per child per day**; never when the next child is the same child, never for `child_exit`/cap/bedtime endings. The next episode of the yielding child includes the VIP surprise in its Arrival segment (a `vip.*` friend, §4.11) and the heart is a mailbox item. **No coins, no XP** (02 UC-05: the heart is a world object, `neverEarns`). The event is cls `learn` on the wire (`turn.yielded`, 11 §5.8).

### 12.6 K1 guide mode ("מצב מדריכה")

- **Two-level opt-in.** (1) The parent enables guide mode in K1's settings (10 §6.13: default **off**). (2) When enabled, K1 is offered per session (never required) "do you want to be your brother's guide today?" The offer appears only when K2 is active on the same device or in a CO-3 room.
- **Cap:** 1 guide session per day, ≤10 minutes (gap §4.12). When the cap is reached mid-session the guide strip fades with "הדרכה להיום הסתיימה, מחר שוב" and K1 can keep playing her own content; K2 continues alone. A second offer never appears that day (§21 UC-18). Economy: one `mentor_session_completed` per day at most (02 §2.2); a session shorter than 3 minutes does not emit it.
- **Brain token first (07 §2.3):** K1's hint tokens are usable only **after K2 has deposited his own attempt** (an answer, a choice or a drawing). The 5s "let him try first" timer starts at that deposit.
- **What she does:** sits next to K2 in CO-2 or CO-3, sees on her screen what K2 sees plus a "guide strip":
  - "תני לו לנסות קודם" (a 5s timer before she can help);
  - "שאלי: איך ידעת?";
  - 3 "hint tokens" per session that she can give (a light glow).
- **What she can't do:**
  - mark him as wrong;
  - answer instead of him (no input for her on his items);
  - see his statistics;
  - compare.
- **What she gets:**
  - Progress on **her** Mentor Track (a separate track in the Adventure Album with its own milestones: "שאלת שאלה טובה", "חיכית בסבלנות", "הסברת בדרך אחרת").
  - Never coins "for his answers" (so his success isn't her payment).
- **Azmitia & Hesser (gap §3):** the prompts encourage explaining and handing over control, exactly the effective behaviors.
- **The parent sees** a short summary: "היא שאלה 'איך ידעת' 3 פעמים".

---

## 13. Seasons and live events: calendar 5787 (R1)

### 13.1 Rules

- **Each event:** a decoration in the camp, a special event level or region, a mosaic or sticker set, and 1 collectible set. **All content remains available forever** after the event (00 D3: "always return"; a seasonal album that never expires).
- **Events open on calendar dates but never close** in a way that loses anything. After the event ends, its level moves to the "תיבת החגים" in the hub.
- **Shabbat inside an event:** event days that fall on Shabbat open cumulatively after Shabbat (00 B7).
- **Memorial and quiet days:** no events, no boss, celebrations reduced to T0/T1, calm music (see §15.4).
- **No explosions or fireworks** (00 B8), including on Yom HaAtzmaut.
- **Dates:** daytime dates. A Jewish holiday starts the evening before. `calendar.events` is a config with these values, and the parent can move them (e.g., a school vacation).
- **School vacation dates** for תשפ"ז were not verified (R1): `calendar.schoolBreaks` defaults to empty, and the parent fills it in.

### 13.2 Season and event table

| ID | Name | Dates (Gregorian) | Theme | Content highlights per child |
|---|---|---|---|---|
| S0 | **"המחנה מתעורר"** | from launch (target 2026-11-08) → 2026-12-03 | Onboarding, R-03 + R-05 + R-12 | Weeks 1–3 of the arc (§2.4). **English warm-up "בקבוקים מהים" (requested by 05 §15.1, adopted):** until R-08 opens (chapter 6, ~2026-12-20) a bottle washes up at the camp shore **2–3 times a week** (daily surprise slot, §5.5) with a recording of Sunny: one song + 2–3 words, inside the English frame (turquoise + entry jingle), 3–5 minutes, for every child (K3 with a parent). K1's English placement ("המכתב של Sunny", 05) runs in week 3 through a bottle. Bottles stay in the mailbox and never expire. No rewards beyond the Wordling of each word |
| EV-01 | **"שמונה לילות של אור"** (Hanukkah) | first candle on the evening of **2026-12-04** (Shabbat eve!) → day 8 **2026-12-12** | A hanukkiah on the campfire; each night 1 candle and 1 friend | K3: count 1–8 candles with a song (feminine forms). K2: the spinning-top letters נ ג ה פ (MX-H01 on the dreidel). K1: how many candles altogether including shamash (2+3+…+9 = 44, MX-M06); sharing sufganiyot as fractions (MX-M09). Offline: light candles and count together. Night 1 is Friday evening → unlocks on Saturday night with night 2 (cumulative) |
| S1 | **"עונת הערפל"** | 2026-12-13 → 2027-01-30 | R-04 the fog lake + R-08 English island (week 6) | K1: start the gradual nikud-fade (prep before February); K2: the vowel series; the English island opens for everyone |
| EV-02 | **"חג השתילה"** (Tu BiShvat) | 2027-01-20 → **2027-01-26** (the holiday: 2027-01-23, Shabbat) | Planting trees in the camp (a permanent grove) | K3: tree/flower/bird names. K2: count 15 seedlings (Tu = 15). K1: an orchard plan in an array (MX-M05) + fruit names in English |
| S2 | **"עונת התחפושות"** | 2027-01-31 → 2027-03-27 | The Muddler prepares a festival; R-06 ברגית's workshop | Building, measuring, geometry; a costume tent |
| EV-03 | **"חצי-תחפושת"** (Purim Katan, small) | **2027-02-21** | A small surprise: one funny hat | A small daily surprise only |
| EV-04 | **"פסטיבל הבלבול"** (Purim) | 2027-03-16 → **2027-03-24** (Purim 03-23, Shushan Purim 03-24) | Everything is upside down! Costumes, a mishloach manot of light | K3: dress-up of friends (cause and effect). K2: put things back in their place (MX-X04), "the reverse day". K1: Hebrew riddles and wordplay (a festival of puns), sorting a mishloach manot by rules (logic). Costumes for the avatar and פנסון (cosmetic, permanent) |
| S3 | **"עונת האביב"** | 2027-03-28 → 2027-05-22 | R-07 the observatory, map of Israel | Science, experiments, geography |
| EV-05 | **"ניקיון האביב"** (Pesach) | 2027-04-15 → **2027-04-28** (seder on the evening of 04-21) | Cleaning the camp: crumb shadowlings (round crumbs, giggling) | K3: tidying toys (sort). K2: the 4 questions → a mechanic of questions (MX-T03), counting 4 cups. K1: a seder table for N guests (multiplication), a "family haggadah" of riddles. Offline: help set the seder table. **No plagues** (not appropriate). A vacation period: an extra 1/day episode allowed (parent setting) |
| QD-1 | **Yom HaShoah** | **2027-05-04** | Quiet day | No event or boss; T0/T1 only; calm music; K1: optional "a candle and a minute of quiet" (on the parent's screen, not in the game) |
| QD-2 | **Yom HaZikaron** | **2027-05-11** (from the evening of 05-10) | Quiet day | Same; **no sounds that resemble a siren**; the default for the day is "the camp is resting" with calm music only |
| EV-06 | **"יום הולדת למחנה"** (Yom HaAtzmaut) | from the evening of 2027-05-11 → **2027-05-12** + the weekend | A light parade of lanterns, a picnic, blue-white decorations, the map of Israel lights up | K2: the colors of the flag, shapes (Star of David = 2 triangles). K1: MX-X03 on the map of Israel (regions, cities). **No fireworks.** A lantern parade instead |
| EV-07 | **"ליל המדורות"** (Lag BaOmer) | from the evening of 2027-05-24 → **2027-05-25** | The highlight of the campfire game: a big family evening | A big family boss (the Muddler's "smoke cloud"); fire safety for K2/K1 (a scripted character: "מדורה רק עם מבוגר"); +2 fire levels |
| S4 | **"עונת הקציר"** | 2027-05-23 → 2027-06-30 | Gathering the friends from all the regions (mixed review), end of the school year | Mixed review, "Then vs. Now" |
| EV-08 | **"סל הביכורים"** (Shavuot) | 2027-06-06 → **2027-06-11** | Bikurim baskets, the seven species, a dairy picnic | K3: fruits and colors. K2: count fruit in a basket, sort the 7 species. K1: weight/measure (measurement), a basket recipe (fractions). Offline: make a basket at home |
| S5 | **"קייטנת הקיץ"** | 2027-07-01 → 2027-08-31 | The deep forest (a large new region), a summer reading challenge | K1: a summer reading program (MX-H07/H08); K2: preparing for grade 1 (blending, the full nikud set); a summer vacation setting (an extra episode allowed) |
| QD-3 | **Tisha B'Av** | **2027-08-12** | Quiet day | Like QD-1 |
| S6 | **"שנה חדשה במחנה"** | 2027-09-01 → 2027-10-23 | Back to school: **K2 starts grade 1, K1 starts grade 4, K3 to gan**; profile update (§DD-18) | Re-placement (a play-based placement test) |
| EV-09 | **"תפוח בדבש"** (Rosh Hashana 5788) | from the evening of 2027-10-01 → **2027-10-03** | A new year at camp: apples, honey, wishes | K2: counting pomegranate seeds (in tens, K1 multiplication), "wishes" (SEL: a goal for the year). Shofar sound: only the actual shofar sound (not a siren), quietly |
| QD-4 | **Yom Kippur** | **2027-10-11** (from the evening of 10-10) | Quiet day + "the camp is resting" | The default: the app sleeps (if the family chose "full Shabbat"), otherwise QD |
| EV-10 | **"בונים סוכה"** (Sukkot 5788) | from the evening of 2027-10-15 → **2027-10-23** (Simchat Torah) | The building system in a holiday version: a family sukkah by the campfire | K2: decorations in a pattern (MX-M11). K1: sukkah area/perimeter, the measurement rules (MX-M10, M05). Offline: decorations for the real sukkah. Simchat Torah: a joint circle dance in the hub (MV-01 for the family) |
| (note) | Yom Yerushalayim 2027-06-04, Tu B'Av 2027-08-18 | — | A neutral day, no event | (A politically loaded day; R1) |
| (note) | Rosh Hashana/Sukkot **5787** (2026-09-12 / 2026-09-26) | Already passed before launch | — | Their content is prepared for 5788 |

### 13.3 Structure of an event (template `EventTemplate`)

- **Length:** 1–9 days.
- **Contents:**
  - 1–3 event levels (one gimmick per level, the holiday version of an existing gimmick plus one twist);
  - 1 decoration for the campfire;
  - 1 sticker/mosaic set (8–12);
  - 1 offline mission per day;
  - 1 family evening (optional).
- **No "event coins"** and no special currency. Rewards come from the regular rules in 02 (`noEventCurrency`, `matchedSinkRequired`: every event ships with matching holiday shop items).
- **"Event treasure"** (02 trigger `holidayEventComplete`: K1 legendary chest with 3 gems, K2 gold chest, K3 surprise envelope) after completing ≥60% of the event levels (`eventTreasureAtLevelShare`). Its contents are known in advance.

---

## 14. Onboarding and first week

### 14.1 Setting up the family (parent, behind the gate, ~10 minutes, before the children)

1. Names, grammatical gender, birth year.
2. Building avatars (§3.3).
3. Recording the name + 3 lines (opening, goodnight, milestone) — the `name_clip` is the only item that blocks anything; format, storage and sync per 12 §6.11 (§4.10).
4. Shabbat mode and observance level (§15.4).
5. Daily caps.
6. Device placement for movement (optional).
7. A "letter of the week from gan" / "this month in class" (optional).

### 14.2 The first 60 seconds per child

**Runs once per child (F9 GAP-01):** the opening flow below runs only when `child_resume.ftueCompletedAt` is `null` (11 §4.16, LWW doc); it is set at the end of the flow (`ftue_completed{childId}` event, Appendix B). A second or restored device therefore never replays the opening; it lands on the Plan step. The parent can re-run it from FAM-03 ("להראות את הפתיחה שוב"), which clears the field. The three endowed friends (§14.3) are rescued only in the first run.

**K2** (the reference flow; voice only, no text):

| s | What happens |
|---|---|
| 0–8 | Darkness of dusk → one firefly (ניצי) lights up. "!אוֹר בּוֹקֶר, [name in the parent's voice]" |
| 8–20 | A 12s movie: the Muddler's sneeze (funny), the friends fly away. ניצי: "עזור לי להחזיר אותם?" |
| 20–35 | The **פנסון lab**: 3 big pulsing parts (chassis/color/topper). Each tap changes it immediately, plus a laugh. A big "!יוצאים" button appears after 2 choices |
| 35–50 | The פנסון wakes up in the camp. **The first tap on the ground → it rolls there** (teaching tap-to-move by doing, no tutorial). Glints lead to a stone |
| 50–60 | A rune on the stone hums "/m/". ניצי: "איזו אבן אומרת 'מָ'?" Two stones. A correct tap → the first friend (a letter-friend) comes out, T1 + "!מָצָאתָ אוֹתִי". (If wrong: the stone says its sound, and the right one glints. Nothing to fail) |

**K1** (with text + voice):

| s | What happens |
|---|---|
| 0–10 | Same opening, plus a short title in the HUD "מחנה הגחליליות". "Skip" appears after 3s (she doesn't like "babyish") |
| 10–25 | The פנסון lab: Ranger chassis, 6 parts + **giving it a name** (a keyboard or 6 suggestions) |
| 25–40 | Tap-to-move + the joystick shown as an option ("רוצה ג'ויסטיק?") |
| 40–60 | A first lock: "החבר תקוע מאחורי שער. כדי לפתוח: 6 × 4 קורות." The array hammer (MX-M05) as a single-target worked action (scripted, not a placement datum) → the first friend + a short diary page |

**K3** (lap mode, with a parent):

| s | What happens |
|---|---|
| 0–10 | A soft playroom, פופי peeks out: "!קוּקוּ, [name]" |
| 10–30 | "!גַּע בִּי" → every touch makes פופי laugh and bounce (cause and effect) |
| 30–50 | Two bushes are shaking. Every tap reveals an animal ("!פָּרָה! מוּ!") |
| 50–60 | The animal hops into the playroom. A first sticker (T2 sensory). Parent prompt: "שאלו: מי זה?" |

- **No *feeling* of a test — but the placement expeditions exist (00 H2, 08 §3.1; F9 GAP-12).** 08 owns the placement structure; this document owns only the child's experience of it. There *are* explicit expeditions — K2 "הרפתקת היכרות" (3 × 3–5 min, optional 4th), K1 "מסע בחירת גיבור" (3–4 × 8–10 min), spread over 2–4 days (08 §3.1–3.2) — but the child never experiences a test: each expedition is a **story-wrapped level** with `mode:'placement'` (§6.6) whose items follow 08 §3.6's UI rules (neutral "map reveal" feedback, no right/wrong, no timer, a closing win item with P ≥ 0.90), and **no coins, XP or chests** (02 §2.7 `placementProbe`); rescues made inside count for the collection (00 G2). Until the first subject is mapped, the planner issues placement plans only (08 §10.5.12; K3 has no expedition). 10 ON-12 shows the parent "2–4 short days". Lazy placement for continuation strands later happens inside regular levels (08 §3.7).

### 14.3 First week (day by day)

| Day | K3 | K2 | K1 | Family |
|---|---|---|---|---|
| Sun | The playroom (3 toys) + peekaboo | Onboarding (§14.2) + **expedition E1 "מספרים"** (08 §3.2) as the level slot, in R-05 scenery + camp Show (3 endowed friends) | Onboarding + **expedition E1 "חשבון א'"** in R-05 scenery + diary | — |
| Mon | + a 4th toy, "copy me" | **Expedition E2 "צלילים ואותיות"** in R-03 scenery + the collection room opens (3 friends from onboarding, endowed) | **Expedition E2 "עברית א'"** in R-03 scenery + the collection room | — |
| Tue | Peekaboo with colors | **Expedition E3 "סיפור"** (+ optional E4 tracing) + first movement moment (MV-02 clapping syllables) | **Expedition E3 "חשבון ב' + אנגלית"** (`english.listening` + `english.letter_sounds` probes run **here**, 08 §3.2; 05's week-3 bottle "המכתב של Sunny" (05 §14.1/§15.1) is the story wrapper for the lazy probes of `english.oral_vocab`/`english.decoding`, 08 §3.7 — one timing, reconciled in the final sweep) + the first Build Mission | — |
| Wed | Rest / playroom | First regular episode: LV R-03 #1 (rune lens) + lantern breath (MX-S02) + first pet egg (known in advance) + **first evening** | **Expedition E4 "עברית ב'"** + a short first regular level (R-05 #1 array hammer) + first evening defense with towers + egg | — |
| Thu | Family: "lighter" role | **First family evening: the "ענן ההפתעות" boss** (CO-2, short, 5 min, guaranteed win) | Same + optional guide mode | Fire level 1→2, the first family treasure |
| Fri | Optional (before Shabbat) | Short episode: daily surprise + Build | Same | Recorded "Shabbat shalom" from a parent (optional) |
| Sat | Shabbat (§15.4) | Shabbat | Shabbat | Nothing counts |

**Endowed progress must be real (02 §5.2 rule 8, `endowedProgress.mustBeActiveRescues`):** the 3 friends that the collection room opens with are rescued by the child in the **first episode** through real, easy learning actions: friend 1 in the first 60 seconds (§14.2), friends 2–3 inside expedition E1's story wrapper (08 §3.6: the breather and the closing win item, P ≥ 0.90, single-target, no failure possible; they are rescues, not scored placement data). The room never shows a friend the child did not rescue.

**Week-1 episode template:** days Sun–Tue (K2) / Sun–Wed (K1) use `episode.k2.placement` / `episode.k1.placement` = Arrival + Plan (no choice: "היום ניצי מראה לך את המפה") → expedition (the level slot, `mode:'placement'`) → Camp: Show (friends move in; **no receipt**, nothing to receive) → Goodnight + offline mission. The warm-up review, Block B and the evening defense start with the first regular episode (K2 Wed, K1 Wed after E4). 08 §10.5.12's first-week budget (×0.8 items, `closing` at 80%) applies to the first three regular episodes.

**Goal of week 1:**
- Each child has rescued ≥8 friends (K3: 6 toy friends).
- The family has seen its fire grow.
- Each child has had ≥1 "show Abba/Ima" moment.

---

## 15. Stopping, Shabbat, holidays, toddler

### 15.1 Designed to end (00 C3, C5, `risks` §4)

- **A visible progress strip** in the HUD: "היום במחנה": 4–5 icons (morning, level, camp, evening, goodnight) that fill in as you go.
- **A soft warning:** when 2:00 remain of the cap or the episode, ניצי says "עוד הרפתקה אחת קטנה ואז הולכים לישון". **Inside the English island (R-08 or any English frame) the Hebrew warning is deferred to the island exit** (00 B2, AC-MX-3; F9 GAP-30): if the cap or bedtime ends while the child is inside, Sunny says "Bye-bye, see you tomorrow!" + `sfx.english.exit` first, and only then does ניצי run the Hebrew wind-down; the island beat is finished within the same ≤1:30 rule below.
- **Never cut in the middle of a beat (one rule shared with 08 §6.10, F9 GAP-02):**
  - At **85%** of the episode budget the planner enters its `closing` block (08 §6.10; 80% in the first three episodes, 08 §10.5.12). That is the normal landing.
  - If the **daily cap** ends in the middle of a beat, the game finishes the current beat (**≤1:30 overtime**) and then **"lands"** in Ketsu: a shortcut to the gate and the last friend. 08's **60 s grace applies only inside `closing`** (the cap arriving while the closing block is already running); the two never add up — total overtime past the cap is ≤90 s in every case (AC-STOP-2).
  - If more than 1:30 remains in the level, its state is saved and it continues tomorrow from the checkpoint.
- **The goodnight ritual (fixed, 30–45s):**
  1. The friends gather around the fire.
  2. Each one yawns.
  3. ניצי dims his tail.
  4. The child "covers" the פנסון with a blanket (a tap).
  5. A lullaby of 4 notes.
  6. The offline mission card.
  7. The screen goes dark into a "the camp is asleep" picture.
- **After the goodnight:**
  - The app shows only "the camp is asleep" (a slowly animated drawing) plus the parent-gate button, then returns to the profile picker (09 SCR-07).
  - **No "play again" button** for the child. Re-entry for K2/K3 is through the parent gate.
  - **K1's "camp asleep" view (F9 GAP-43; 09 SCR-07 gets this state):** after her wind-down, K1's picker tile opens `camp_asleep` instead of a session: the dim camp with exactly two doors lit — the **collection room** (view/poke only) and the **music hut** (listen only); the shop tile shows "החנות ישנה" (02 §7.3), gates and levels are dark and unresponsive. **No `session.started`, no learning or economy events, 0 cap minutes**; only `free_choice_unrewarded{activity}` telemetry (Appendix B). The view auto-closes to the picker after 10 minutes of no touch or at bedtime. K2/K3 tiles never open it (gate only).
- **Caps (parent-set; defaults are owned by 10 §6.13 and mirrored here):**
  - K3: 2 episodes / **15** minutes per day (lap mode; the 5-minute solo playroom counts inside it).
  - K2: 2 / 25 minutes.
  - K1: 2 / 40 minutes.
  - Weekday and Friday/Shabbat caps are separate settings (10 §6.13). Co-op minutes count at 50% for each participant (§12.1).
- **Bedtime lock (semantics owned by 10 §6.13, mirrored here):** from a parent-set hour (`child_settings.bedtime`; defaults K3 19:30, K2 20:00, K1 20:30, OPD-26). No **new** episode starts once `now ≥ bedtime − preBedtimeMin` (**`preBedtimeMin` default 60, parent-editable 15–60**, 10 §6.13); the picker tile then says "הגיבור כבר מתכונן לשינה" (not a lock screen). If `bedtime − now < plannedEpisodeMin + 2` the episode is planned as a `mini` (08 §6.1) so it lands before bedtime; one already running finishes with the normal wind-down (`session.ended{reason:'bedtime_lock'}` if the lock arrives first). At bedtime itself the app shows "the camp is asleep" (09 SCR-20) and `launch.blocked{reason:'bedtime'}`. Where both a per-child bedtime and a per-device schedule (11 §5.5, `device_settings.bedtime`) exist, the **earlier one is the effective lock** (F9 GAP-04). A bedtime change applies immediately (safety), but a running episode lands at a natural point (08 §6.10).
- **"App sleeps" windows** (10 SET-02: school hours, meals, parent-declared): same screen as bedtime; the parent gate opens it as an exception (audit-logged).
- **Cap or window changed by the parent mid-session (§21 UC-11):** the new value is applied at the **next segment boundary** (Appendix A `segments`), never mid-beat. If the new cap is already exceeded, the soft warning fires immediately and the episode "lands" within ≤1:30 (above). If the cap was raised, the current episode simply gets the extra room. The child never sees a message about the change; the receipt is unaffected.
- **Parent "+10 min" bonus** (DD-09): a button behind the gate, max 1 per child per day, logged; it extends the cap, it does not add an episode. It is the event `cap_bonus_granted{childId, minutes: 10, grantedBy: guardianId | 'device-local', dayKey}` (cls `admin`, audit-logged, 11 §5.8; F9 GAP-02); the engine adds the minutes to today's cap at the next segment boundary; 02 ignores it (`neverEarns`).
- **Media-free contexts:**
  - The parent marks meals or travel in a schedule; the app "sleeps".
  - **Car: "מצב רכב", audio only — a P2 feature (F9 GAP-19), not in MVP/v1.** Spec for when it lands: a parent-gate toggle on the device (10 PA-CSET) that opens a **playlist** (music-hut songs, K2's daily English song from a bottle, riddles read by ניצי); the screen is dimmed to a single large "עצור" button (≥20 mm) and the current song's picture; **no learning items, no economy events, no `session.started`, 0 cap minutes**; it stops when the toggle is turned off or after 45 minutes. Until P2 the parent uses the music hut inside a normal session. 09 (SCR-21) and 11 own the screen and the launch path.
- **Offline mission at the end** (every episode, by age and by the content learned):
  - "מצאו 3 דברים בבית שמתחילים ב-מ"
  - "ספרו כמה מדרגות יש בבניין"
  - "Sunny asks: find something *blue*"
  - "תכנני את הפנסון שלך על דף" (printable)
  - Done/not done is marked by the parent or not marked at all; nothing is lost. A parent-confirmed mission is the economy event `offline_mission_confirmed` (02 §2.2: a wood chest, XP); an unconfirmed one costs nothing.
  - Every mission card carries `safety: SafetyTag[]` (00 C13) and the tags are printed on the card for the parent (e.g., "עם מבוגר", "בלי אש"). The content linter rejects a mission without tags (AC-STOP-4).
  - **Printing (F9 GAP-50; everything printable in this document — mission cards, the פנסון outline (§3.4), season certificates (§5.1), tangram/number-line sheets):** the app renders an **A4, RTL PDF** with `expo-print` (11 §2.3 module list) and hands it to the **iOS share sheet / AirPrint** from the parent area or from the mission card's "להדפיס" button (parent gate for K2/K3). Nothing prints silently; no printable contains a child's data beyond the name. Printables are static assets in the content pack (`print/<id>.pdf` or an HTML template rendered on device), never generated live.
- **Exit in the middle:** a "back to camp" button (in the pause menu) at any time, with no cost. A level that was left is saved by checkpoint. "Back to camp" ends the session as `session.ended{reason:'child_exit'}` (08 §1.10's canonical enum; 10 AL-04's "childQuit") with `secondsSinceLastError` in the payload, so AL-04's 120-s rule can run (F9 GAP-25); the pause itself (דובדובה's "need a break") does not end the session.

### 15.2 Character exit lines (approved)

- ✅ Approved:
  - "שִׂחַקְנוּ יֹפִי. אֲנִי הוֹלֵךְ לִישֹׁן. נִתְרָאֶה מָחָר!"
  - "הַחֲבֵרִים {שֶׁלְּךָ|שֶׁלָּךְ|שֶׁלָּכֶם} יְשֵׁנִים מְאֻשָּׁרִים."
  - "מָחָר יְחַכֶּה לָנוּ מִכְתָּב חָדָשׁ בַּתֵּבָה."
- ❌ Forbidden:
  - "אל תלך!"
  - "רק עוד אחד"
  - "אני אהיה עצוב בלעדיך"
  - "החברים יתגעגעו ויבכו"
  - "אם לא תחזור מחר…"

### 15.3 Toddler lap mode (K3, 00 C3, gap D16)

- **Default:** K3 episodes require a parent to open (Face ID/PIN) and include **parent prompts** on screen (in a corner, small text, *for the parent*): "שאלו: איזה צבע?", "תנו לו/לה לגעת לבד", "ספרו יחד: אחת, שתיים…".
- **Launch mode (`child_settings.launchMode`, F9 GAP-11b; field owned by 11 §4.16, default set by DD-18):** `lap` (today's K3: every launch through the gate, parent prompts on) · `coplayFirst` (from ~3.5 / gan: the gate is required **once per day**; after it, the child may start solo episodes from the picker, each capped at **5 minutes** = the playroom, with prompts off) · `solo` (band G and up: normal picker). K3's profile never launches from the sibling switcher without the gate in `lap` (§21 UC-05).
- **The playroom (MX-P01) is safe to use alone for a short time:** no exits, no text, no fail. It still always opens through the parent gate and stops automatically after 5 minutes (a gentle lullaby).
- **Interaction:** tap only, ≥20 mm, ignore multi-touch, edges of the screen have no actions (12 mm margin), no swipe-to-exit (Guided Access is recommended to the parent at setup).
- **Sound and light:**
  - Max volume -6 dB.
  - No flashing at >2 Hz.
  - Screen shake 0.
  - No dark scenes (§2.3).
- **No:** coins, numbers in the HUD, timers, the Muddler, approaching shadowlings, ASR, camera.
- **Rewards:** stickers (unlimited, immediate), toy friends, פופי's hats.
- **The parent is an active player:** in every K3 episode, ≥1 moment where the parent is asked to do something with the child ("עשו יחד: מחיאת כף").

### 15.4 Shabbat and holidays (00 B7)

- **Detection (00 B7, one library for the whole app):** candle-lighting and havdalah times come from **`@hebcal/core` zmanim** for the parent's city (OPD-04; default Tel Aviv; city-specific candle offsets as hebcal provides them, with a manual override in the parent area). The **pause window is `[candleLighting − 30 min, havdalah + 30 min]`**. There is no second (NOAA) implementation anywhere in the code; `packages/shared/calendar.ts` exposes `shabbatWindow(date, city)` and every consumer (counters, scenery, notifications, economy) calls it (AC-SHAB-3).
- **Holidays** from the calendar file `packages/content/calendar/5787.json` (generated from hebcal, cross-checked, 00 B7; §13). Yom Tov days get the same treatment as Shabbat.
- **The window applies by session start (F9 GAP-14d; 02 §10.3 `appliesBySessionStart`):** an episode whose `session.started` is **before** `candleLighting − 30 min` is a weekday episode to the end — it keeps its normal palette, its receipt is paid in full and its counters count, even if the receipt falls inside the window. The Shabbat scenery, frozen counters and "no coins" apply to sessions started **inside** the window. In `full` mode a running episode gets the normal 2-minute warning at the window start and lands (≤1:30), then the app sleeps.
- **Modes (family choice; default `soft` per 00 B7 / OPD-04):**

| Mode | Behavior |
|---|---|
| `shabbat.soft` (**default**) | The app can be used. All counters stop (the weekly goal and family goal are not affected). No events open or close, no notifications. The camp shows Shabbat scenery (candles, challah, the friends resting, quiet music). Coins/gems are not earned (to avoid "Shabbat grinding"). The levels are open as "free practice" |
| `shabbat.full` | The app sleeps from candle lighting to havdalah: a "Shabbat shalom" picture of the camp asleep. The parent can open it with the gate (an exception) |
| `shabbat.off` | Regular days (a secular family that wants it) |

- **Quiet days** (QD-1…QD-4, §13.2):
  - No events, no boss, T0/T1 celebrations only, calm music.
  - No sounds similar to sirens, ever (00 B8; a test on the SFX set).
- **Holiday eve:** the celebration of the event starts on the evening of the holiday only after Shabbat ends, if the holiday falls on Shabbat (e.g., night 1 of Hanukkah 2026-12-04).

### 15.5 Bedtime story (an optional end, `content_world` §c3)

- From "תיאטרון הסיפורים" (§10.2), optionally at the end of the day:
  - A story of 1–2 minutes made from the day's friends and words ("היום [name] הציל את מָ ואת 7, ו…"). It is pre-generated in the content pipeline, with parent preview (00 C7).
  - Dark UI, no rewards, no buttons apart from "לילה טוב".
  - It is part of the episode's time.

---

## 16. Celebrations, reveals and praise

### 16.1 Celebration tiers (gap D7)

| Tier | When | **Max duration** | Blocks input? | What happens | K3 | K2 | K1 |
|---|---|---|---|---|---|---|---|
| **T0** micro | Every correct action | **≤0.8s** (typical 0.35s) | No (the next item loads underneath) | Squash on the object, a burst of 8–12 particles, a note on the pentatonic scale (it rises in a chain), the פנסון's core lights up, light haptic (iPhone) | Yes, gentle, no shake | Yes | Yes |
| **T1** small | A friend is rescued in a level / end of a beat | **≤1.5s** | No ("don't eject": the friend flies to the jar while play continues) | The friend pops out, a voice line of ≤4 words, an arc into the HUD jar, a "click" | Yes | Yes | Yes |
| **T2** medium | Level complete, a new item/sticker, **wood chest** (02: ~2.9s), capsule pop, a building rises | **≤3.0s** (plus the taps on the chest, which the child controls) | Yes, skippable after 1.0s | Reveal (16.2 → 02 §6.5), the friends dance, the item rotates into the album | Sensory-only version 1.5s (K3 "surprise envelope") | Yes | Yes |
| **T3** large | Skill mastered, a species set complete, pet evolution, region boss, **silver / gold / legendary chest** (02: ~3.5 / ~4.4 / ~5.8s), level-up | **≤6.0s**, rare | Yes, skippable after 1.0s (chests: after the first viewing of that tier, 02) | Confetti (not flashing), ניצי dance, a fanfare of 1.5–3s, sometimes a recorded parent line, a "תראו לאבא ולאמא" moment | Doesn't exist (instead: T2) | Yes | Yes |
| **T4** finale | Weekly family boss, season finale | **≤10s**, at most 1 per week | Yes, skippable after 2.0s | A short cinematic of the friends + the boss turns into a friend + the fire grows | Shared participation | Yes | Yes |

- **Time budget:** celebrations + transitions **≤15% of the episode's time** (gap §4.5). Measured in telemetry (`celebration_ms_total / episode_ms`). If the budget is exceeded, T0 shortens to 0.25s and T1 merges.
- **Screen shake:** K3 0, K2 ≤4px, K1 ≤8px. **Flashing:** never >3 Hz (photosensitivity). Reduce Motion (OS) → no shake, no confetti, fades only.
- **Streak notes:** each consecutive correct answer plays the next note on the C-major pentatonic scale (up to 8). A broken chain resets quietly with no sound (R `hit_games` §4).
- **Error sound:** a soft low "בּוֹפּ" of 150ms, **never a buzzer** and never a red X for K2/K3.

### 16.2 Treasure reveal choreography (deterministic, 00 D2, R3)

> **Binding precedence:** `02-economy-progression.md` §6 is canonical for chest tiers, triggers, contents, taps per tier and the 8-beat reveal timings (`chests.revealMs`). The earlier "effort steps" proposal of this section (taps = 1 + effort marks) is **withdrawn**: 02's rule that the tier is visible from the first frame and the tap count is fixed per tier is simpler, deterministic and lint-able. What survives of the idea is the **effort marks as receipt lines** (below).

- **Tiers (02 §6.1), each a different shape as well as color (color-blind safe):** `wood` תיבת עץ (round, 1 tap, K1+K2) · `silver` תיבת כסף (shield, 2 taps, K1) · `gold` תיבת זהב (star, 3 taps, K1+K2) · `legendary` אוצר אגדי (crown, 3 taps + 600ms hold, K1) · K3 "מעטפת הפתעה" (envelope, 1 tap, stickers). The mapping from this document's older names: gold-frame (region boss) and family (weekly goal) → `gold`; prism (event, season final) → `legendary`; a level/episode gives **no chest** (friends and the receipt are the reward).
- **The tier and the contents are known when earned** (`chest_earned{chestId, contents}` is a **derived** output of the economy fold over the source learn event, `chestId = hash(sourceEventId)`, 02 §6.3 / 11 §5.8 — it is not a persisted event, so two devices always derive the same chest) and **previewed as a silhouette + label before opening** ("בפנים: כובע פיראט + 40 מטבעות"; read aloud for K2). The tier never changes during the reveal (no "upgrade" animation, 02 §6.5 כלל-על).
- **Effort marks** are computed per activity and shown as **process lines in the receipt** (02 §2.8), never as taps or value: `firstTry` (≥80% correct on first try), `hintFree` (no hints in the last beat), `braveChoice` (chose Stretch / 🌶🌶), `mentor` (K1 helped in guide mode), `movedBody` (completed a movement moment in `full` mode — informational only, the seated variant earns exactly the same, 02 §2.2). Lines use the §16.3 phrase sets.
- **Choreography:** 02 §6.5 (enter → preview → anticipation taps → charge → burst → reveal → settle → collect). Celebration-tier mapping: wood = T2, silver/gold/legendary = T3 (§16.1). Memorial days: the chest goes to the shelf, no auto-reveal, no fanfare (02 §6.5).
- **Idempotent reveal (data safety, §21 UC-14):** the chest exists as soon as its source event is in the log (derived, above); `chest_opened{chestId}` is the only persisted chest event and is written at the **start** of beat 6 (reveal). The fold grants contents on `chest_opened` only. If the app dies before beat 6, the chest is still on the shelf unopened; if after, the next launch plays only beat 8 (collect, ≤0.7s) for that chest. No path double-grants.
- **Verification:** a frame-step study on the device after the vertical slice (AC-CEL-6). Tokens are tuned for "feel" within 02's `revealMs` and the T2/T3 budgets.
- **There is no randomness of value.** "Rarity" = what earned it (00 D2). For K1 the label says "קיבלת זהב כי ניצחת את הבוס של המכרות".

### 16.3 Praise phrase sets (the game's voice)

The UX spec is the canonical owner of the phrase list (00 C4). **The lines below are merged into 09 §9.2 as P19–P25** (P19 action, P20 strategy, P21 improvement, P22 checking, P23 co-op help, P24 after a miss, P25 brave choice) and the banned words into 09 §9.3 (F6). **A new line is added to 09 first**, then referenced here; the content pack imports 09's list, never this table. Each category has ≥8 variants per age in the content pack; examples below (the 09 id is given per row where one exists).

| Category | Example K2 `{m|f|pl}` | Example K1 `{m|f|pl}` |
|---|---|---|
| **Describing the action** (09 P19) | `{מָצָאתָ|מָצָאת|מְצָאתֶם} אֶת הַ-מָ!` | `{בָּנִיתָ|בָּנִית|בְּנִיתֶם} קִיר שֶׁל 6 שׁוּרוֹת וְ-4 טוּרִים.` |
| **Strategy** (09 P20) | `{סָפַרְתָּ|סָפַרְתְּ|סְפַרְתֶּם} הָלְאָה מֵחָמֵשׁ!` | `{פֵּרַקְתָּ|פֵּרַקְתְּ|פֵּרַקְתֶּם} אֶת 7×8 לִשְׁנֵי חֲלָקִים.` |
| **Persistence** (09 P-set "persistence"; add to 09 before use) | `{נִסִּיתָ|נִסִּית|נִסִּיתֶם} שׁוּב, וְזֶה עָבַד!` | `{הִמְשַׁכְתָּ|הִמְשַׁכְתְּ|הִמְשַׁכְתֶּם} גַּם כְּשֶׁהָיָה קָשֶׁה.` |
| **Improvement** (09 P21) | `הַיּוֹם זֶה הָלַךְ יוֹתֵר מַהֵר!` | `{זָכַרְתָּ|זָכַרְתְּ|זְכַרְתֶּם} אֶת זֶה גַּם אַחֲרֵי שָׁבוּעַ.` |
| **Brave choice** (09 P25) | `{בָּחַרְתָּ|בָּחַרְתְּ|בְּחַרְתֶּם} בַּשְּׁבִיל הַמְּאַתְגֵּר, אַמִּיץ!`* | `{בָּחַרְתָּ|בָּחַרְתְּ|בְּחַרְתֶּם} לְהִתְמוֹדֵד עִם הָאֶתְגָּר.` |
| **Checking/verifying** (09 P22) | `{בָּדַקְתָּ וְצָדַקְתָּ|בָּדַקְתְּ וְצָדַקְתְּ|בְּדַקְתֶּם וּצְדַקְתֶּם}` | `{שָׁאַלְתָּ|שָׁאַלְתְּ|שְׁאַלְתֶּם} אֶת גַּלְגַּלּוֹן וְאָז {בָּדַקְתָּ|בָּדַקְתְּ|בְּדַקְתֶּם} לְבַד.` |
| **Helping (co-op)** (09 P23) | `{עָזַרְתָּ|עָזַרְתְּ|עֲזַרְתֶּם} לַמְּדוּרָה לִגְדֹּל.` | `{חִכִּיתָ|חִכִּית|חִכִּיתֶם} שֶׁהוּא יְנַסֶּה קֹדֶם. זֶה עוֹזֵר לוֹ לִלְמֹד.` |
| **Neutral escalation in a chain** (no gender; add to 09 before use) | "!שָׁלוֹשׁ בְּרֶצֶף" · "!הַשְּׁבִיל נִבְנֶה" · "!עוֹד חָבֵר" | "!חֲמִשָּׁה בְּרֶצֶף" · "!הַמַּסְלוּל נִפְתַּח" |
| **After a mistake** (09 P24) | `עוֹד לֹא. {בּוֹא|בּוֹאִי|בּוֹאוּ} נִסְתַּכֵּל שׁוּב.` | `עוֹד לֹא. {רוֹצֶה|רוֹצָה|רוֹצִים} רֶמֶז אוֹ עוֹד נִסָּיוֹן?` |

\* "אמיץ" describes the choice in context; the gendered form `{אַמִּיץ|אַמִּיצָה|אַמִּיצִים}` is allowed only right next to the act of choosing.

**Banned (CI lint on all content `packages/content` + scripts):**
- **Trait labels:** גאון/ה, חכם/ה, אלוף/ה, מוכשר/ת, "ילד/ה טוב/ה", "קורא/ת טוב/ה" (as an identity), "מספר אחת".
- **Inflated superlatives:** מושלם, מדהים, הכי טוב, אדיר, "פנומנלי", "לא ייאמן", "מטורף".
- **Comparison:** "יותר טוב מ…", "כמו אחותך", "לפני כולם", "ניצחת את…" (between people).
- **Conditional love:** "אני אוהב אותך כשאתה מצליח", "אני גאה רק…".
- **Guilt/pressure:** "אל תלך", "תחזור מחר או ש…", "פספסת", "החברים עצובים בגללך".
- **Negative-labeling errors:** "טעות!", "לא נכון!" (with a buzzer), "שוב טעית", "קל מדי בשבילך".

**Parity check:** the same number of process lines for the girl and the boy profiles, verified by a test (`praiseParity.test.ts`, Gunderson).

---

## 17. Acceptance criteria per feature

> Format: `AC-<area>-<n>`. Each criterion is testable: Jest for logic, Playwright for web screenshots, telemetry, or a manual check by the parent on a device (marked 📱).

### World and hub

- **AC-WORLD-1:** The hub (route `/(child)/[id]/map` → 09 SCR-04 "hub"; zones at `world/[zoneId]`, levels at `level/[levelId]`, results at `results` = 09 SCR-07 — 11 §4.1 names, used in every AC below) is a walkable scene (R3F or the Skia fallback) with a campfire, 3 neighborhoods and ≥3 gates. Tap-to-move reaches any walkable point within ≤3s of walking.
- **AC-WORLD-2:** No string in the 3D scene is rendered as text. Every Hebrew word is in the 2D HUD (lint: no `<Text>` from drei/troika in `world/`).
- **AC-WORLD-3:** Evening frames have average relative luminance ≥0.28 (K3 ≥0.45). Measured on 20 screenshots per region in Playwright.
- **AC-WORLD-4:** Every region has ≥3 landmarks visible from the spawn point of the region's field area.
- **AC-HUB-1:** A rescued friend appears in the child's neighborhood within ≤2s of returning to camp, with an "arrival" animation.
- **AC-HUB-2:** A threshold sign (§10.2) shows a silhouette of the next building and "עוד N". When the threshold is reached, the building rises with T3 (K2/K1).
- **AC-HUB-3:** At 150 friends the hub stays within the budget of ≤80 draw calls and ≥55fps p95 on the oldest iPad 📱.
- **AC-COLL-1:** The collection room shows silhouettes for all friends in the child's curriculum map. Poking a silhouette gives a voiced location hint. It opens with ≥3 friends (endowed).

### Levels

- **AC-LV-1:** Every level file passes lint:
  - exactly 1 `gimmick`;
  - the 4 beats in order;
  - ≥4 secret types (§6.2);
  - ≥1 landmark;
  - friend and piece counts by age (§6.1).
- **AC-LV-2:** In ≥95% of levels in playtest/telemetry, the duration is within the age range (K2 3–6 min).
- **AC-LV-3:** A friend on the path can't be rescued without a learning action (a unit test on the `rescue` logic: `requiresItemCorrect=true`).
- **AC-LV-4:** A rescue doesn't eject: input stays active during T1 (Playwright: a movement command during T1 is executed).
- **AC-LV-5:** A fall or bump returns to a checkpoint within ≤0.8s with no loss of friends.
- **AC-LV-6:** Frustration guard (gap D10): after 2 errors a hint; after 3 a worked example. There are never 4 errors in a row on the same item for K2 (engine + simulation test).

### Mechanics

- **AC-MX-1:** Each MVP mechanic (11) has a pure-TS implementation in `packages/engine/mechanics/*` with tests for: correctness, knobs, the failure ladder, and distractors from the error catalog.
- **AC-MX-2:** Counting voice-over comes from the numerals lookup table only (a test: no TTS call for a number).
- **AC-MX-3:** English: every English line plays only inside the English frame (`englishContext=true`) with an entry/exit jingle. Phonics sounds come from the recordings list only. No Hebrew line (incl. ניצי's cap/bedtime warning, §15.1) plays while `englishContext=true`; a cap ending inside the island produces Sunny's exit line + jingle **before** the Hebrew wind-down (test with a cap set to expire mid-island).
- **AC-MX-4:** RT is not included in the update for K2/K3 (a test on the engine adapter).
- **AC-MX-5:** ASR (MX-H08) never displays "wrong" to the child (a snapshot test of every result state).

### Movement

- **AC-MV-1:** Every MV-* has `seated` and `tap` variants that are fully playable. Switching mid-moment (the "sit" button) works with no loss.
- **AC-MV-2:** In quiet mode (or after 20:00 by default) no jumping moment appears (MV-03/MV-04 swap to the seated variant).
- **AC-MV-3:** No movement moment requires holding a device while jumping. MV-07 shows only on iPhone with a "stand still or sit" prompt.
- **AC-MV-4:** Clap detection on the microphone doesn't store audio (code review + a test that no file is written).

### Battles, bosses, evening, runner

- **AC-BTL-1:** The player has no HP. A wrong answer doesn't remove a layer and doesn't subtract anything (a test).
- **AC-BTL-2:** Every defeated shadowling becomes a friend or a visible light-mote. No "death" animation (asset review).
- **AC-BOSS-1:** The weekly boss formula (§9.3): a simulation of a week where every child completes 80% of plan → HP reaches 0 in the finale. With 50%, the boss "falls asleep" with the meter saved for next week.
- **AC-BOSS-2:** The finale never ends in a loss (a test of all states).
- **AC-BOSS-3:** `bossHP` excludes paused children and children with 0 episodes this week; a finale with any non-empty subset of children (incl. one child) reaches HP 0; a boss day falling on a quiet day / Yom Tov moves the window to the previous eligible weekday evening (calendar test with 2027-05-11 and Yom Kippur 5788).
- **AC-EVE-1:** Evening: shadowlings don't approach the camera (path z ≥ fire.z − 2 tiles). In K3 only `sleepy` appears.
- **AC-RUN-1:** The runner always ends at a destination within ≤90s (K2) / ≤120s (K1). There is no death state or revive (a test).

### Co-op

- **AC-CO-1:** In CO-1 the two halves of the screen are rotated toward their player, and each half has a separate RTL HUD (screenshot).
- **AC-CO-2:** Fairness: in telemetry, each child's share of light is within ±25% of an even share in ≥80% of co-op sessions. If not, a closed-loop correction runs.
- **AC-CO-3:** Drop-in/out: a player who leaves doesn't stop the session (≤1s of disruption). A player who joins gets a role within ≤3s.
- **AC-CO-4:** No screen in co-op shows separate data per child to the children (screenshot review).
- **AC-CO-5:** Guide mode: K1 has no input on K2's items. There is a 5s delay before a hint token. There is a 10-minute cap and 1 per day (a test).
- **AC-CO-6:** Co-op time is counted at 50% toward each child's cap (a test).
- **AC-CO-8:** `turn_yielded` (§12.5) fires only for `parent_exit` before 80% of planned minutes followed by a different child's session on the same device within 120 s; ≤1 per child per day; it produces no grant (fold test) and the next episode's Arrival contains a `vip.*` friend. CO-4 letters contain no free-text field (schema test on `coop_letter`).

### Economy and celebrations

- **AC-ECO-1:** No coins are paid per raw time or per spark. Every coin event is tied to an event from §11.2 (a test of the payment table).
- **AC-ECO-2:** There is no pre-announcement of a reward for a task (lint on strings: "תקבל/י … אם").
- **AC-ECO-3:** Chests: the contents and tier are known before opening (silhouette + label, 02 §6.3) and the tier is identical in all 8 reveal beats (snapshot). Taps per tier are 02's (wood 1, silver 2, gold 3, legendary 3 + hold). There is no RNG in the reward path (a lint rule bans `Math.random` in `economy/`). Effort marks appear only as receipt lines (test: `effortMarks` never changes any grant).
- **AC-ECO-4:** `chest_opened` is written at the start of reveal beat 6; killing the app at any beat and relaunching never grants twice and never loses a chest (a fold test over the three cut points: before `chest_earned`, between earned and opened, after opened).
- **AC-ECO-5:** Play shekels in MX-M15 never appear in any wallet, receipt or shop; a static test asserts the `camp-shop` module has no import from `economy/`.
- **AC-HERO-1:** `hero-naming.lint`: no string, asset name, Rive artboard, prompt file or code comment under `apps/mobile`, `packages/content` or `tools/` uses "robot/bot/mech/android/רובוט" for the hero; the gear icon asset is referenced only by `machine/*` components (גלגלון, bots, "machine-made" tag). 0 hits.
- **AC-REG-1:** Region unlock = (story calendar OR `fireLevel` threshold) AND readiness (§10.4). A simulation with a family that never meets a weekly goal still opens every region on its chapter week; a family at fire level 9 by week 3 opens R-04..R-09 early only if readiness holds.
- **AC-FIRE-1:** `fireLevel` is monotonic over the whole event log and across season boundaries (fold test with 3 seasons: never decreases; a new season adds a ring, never a reset).
- **AC-PET-1:** Pet stage is a pure function of `masteredSkillsInDomain` (and `retained` for legendary) per 02 §8.2; completing a species set changes `accessories`, not `stage` (unit test).
- **AC-MX-6:** Every mechanic in §7.3 has a card with all §7.0 fields and a `MechanicDef` with `compactForms` (test: catalog completeness).
- **AC-MX-7:** Brain-token gate: in every mechanic that exposes גלגלון's ladder, `hintAvailable === false` until a deposit event exists for that item (engine test across the catalog, implements 07 AC-07).
- **AC-MX-8:** MX-E01 `pairs`: no pronunciation score is ever computed or stored; the speaker pool rotates among the 4 adult voices + the human recording (test).
- **AC-MX-9:** Tool belt: `toolState` transitions follow 07 §2.2 (`chosen` requires ≥3 independent choices over ≥2 weeks; `transferred` requires a parent confirmation event).
- **AC-CO-7:** Touch zones: in CO-1/CO-2 a tap outside a child's own zone never produces an `item_answered` for another child (gesture test with synthetic taps per zone); `assisted:'sibling'` appears only after a parent mark event (engine test).
- **AC-MX-10:** MX-H12: PA items (`H.PA.*`) render sound buttons only, never letter text (snapshot); every isolated phoneme comes from the HREC recording list (test, as AC-MX-3 for Hebrew). MX-H13: a pair is served only when both letters are ≥70% known; errors are logged per pair (`EH.VIS.<PAIR>`). MX-H14: no accuracy score is computed for `H.SP.INV`/`H.WC.EM.*` items (`EH.SPL.PHON` logged as expected). MX-H15: K2 items show no text (snapshot).
- **AC-MX-11:** MX-H06 emits `hint:lens` on every lend; the engine's lens-tap rate per 100 words is derivable from the log (test).
- **AC-CTRL-1:** Every mechanic is completable with touch only; the optional controller never appears in any tutorial or prompt (snapshot).
- **AC-STOP-4:** Every offline mission card has ≥1 safety tag from 00 C13; the small-parts rule is enforced while `youngestChildAge < 3` (content lint).
- **AC-SHAB-3:** All Shabbat/holiday window checks go through `packages/shared/calendar.ts` (`@hebcal/core`); a grep test finds no other sunset/zmanim computation.
- **AC-UC-\*:** the acceptance criteria of §21 (one per use case) are part of this list.
- **AC-CEL-1:** T0 ≤800ms, T1 ≤1500ms, T2 ≤3000ms, T3 ≤6000ms, T4 ≤10000ms (a timer test on each animation sequence).
- **AC-CEL-2:** Celebrations + transitions ≤15% of the episode time on average over a week (telemetry).
- **AC-CEL-3:** No flashing >3 Hz. Shake 0 for K3. Reduce Motion is respected (an animation test).
- **AC-CEL-4:** Lint on banned phrases (§16.3) across all content and scripts: 0 hits. Gender parity check passes.
- **AC-CEL-5:** Every child-facing string has `m` and `f` variants (and `pl` where relevant). Playwright snapshots for each profile.
- **AC-CEL-6:** A frame-step study on the device after the vertical slice (📱). The reveal tokens are updated and documented in `docs/decisions/`.

### Stopping, Shabbat, toddler

- **AC-STOP-1:** Every episode ends with the goodnight ritual (30–45s) plus an offline mission card. After it there is no "play again" button for K2/K3 (screenshot).
- **AC-STOP-2:** The cap never cuts off mid-beat. Overtime is ≤1:30 at most (incl. 08's 60 s grace inside `closing`; the two never sum past 90 s), or the state is saved for tomorrow (a test with the cap expiring at every beat boundary and inside `closing`).
- **AC-STOP-3:** No exit line from §15.2's banned list appears (lint).
- **AC-STOP-5:** K1's `camp_asleep` view (§15.1) emits no `session.started`, no learn/econ event and adds 0 cap minutes; K2/K3 tiles after wind-down open only the gate (Playwright on SCR-07 → picker → tile). `cap_bonus_granted` is accepted at most once per child per `DayKey` and never creates a grant (fold test).
- **AC-FTUE-1:** With `child_resume.ftueCompletedAt` set, launching the child on a second device goes straight to Arrival + Plan with no opening movie (Playwright); with it null, the full §14.2 flow runs and sets it.
- **AC-PLC-1:** Expedition levels (`mode:'placement'`) produce rescues that appear in the collection and 0 coins/XP/chests (fold test over a week-1 log); in week 1 the planner emits only placement plans until the first subject's `placement.state='done'` (08), and the §14.3 day table holds for the simulated calendar (Sun–Tue K2, Sun–Wed K1).
- **AC-SHAB-1:** In `shabbat.soft` no counter changes between lighting and havdalah. There are no coin/gem events and no event opening (a time-simulation test with Tel Aviv / Jerusalem times).
- **AC-SHAB-2:** On QD days there is no event or boss, celebrations are capped at T1, and no SFX from the "siren-like" list is played (a spectral check of the SFX: no sweep 400–1500Hz >1s).
- **AC-K3-1:** A K3 episode opens only through the parent gate. Multi-touch is ignored. No action exists within a 12 mm margin. There are no texts for the child. The playroom auto-stops after ≤5 minutes.
- **AC-K3-2:** No K3 scene contains the Muddler, approaching shadowlings, or a dark palette (an asset/scene-config check).

### Events

- **AC-EV-1:** `calendar.events` contains every row in §13.2 with the right dates (a test comparing against the internal arithmetic Hebrew-calendar implementation).
- **AC-EV-2:** An event's content remains available after its end (the "holiday box" in the hub) (a test).
- **AC-EV-3:** An event day that falls on Shabbat opens cumulatively after havdalah (a test with Hanukkah 2026).

---

## 18. Default decisions (DD) for ambiguous cases

| ID | Ambiguity | Default |
|---|---|---|
| DD-01 | 3D or 2.5D? | Per the spike (E1b). Design is camera-agnostic. **K3's playroom is always 2.5D Skia isometric** (lighter, fixed camera, matches tap-only) |
| DD-02 | World and character names | The first option in each list (§2.1, §3.1, §4.1) until the taste gate (E10). Names are stored in a `names.ts` config, not in code |
| DD-03 | Hero gender | The פנסון has no voice, so no gender is needed. In speech about it, use "הפנסון" (m, grammatically) |
| DD-04 | AI robot's grammatical gender | m ("גלגלון"). First person in the present tense: "אני מחשב" |
| DD-05 | K3 gender until confirmed | `pl`/neutral forms where possible; otherwise `f` and `m` alternate per session (to avoid bias) until the parent sets it |
| DD-06 | Weekly boss day | Thursday, 17:00–19:00 is the default "opening hour" for the finale (the finale is available from Thursday until Friday at noon). If that day is a Yom Tov or a quiet day (00 B7a) the window moves to the previous eligible weekday evening; if none is left in the week, the boss sleeps and carries over (§9.3). `bossHP` counts only non-paused children with ≥1 episode this week; the finale completes with any present subset |
| DD-07 | Is Saturday night a new week? | Yes. The week runs Sunday → Saturday night |
| DD-08 | Two children want the same device | The turn ring decides by last use. If both want to, a CO-2 suggestion is shown |
| DD-09 | A child who wants to keep going after the cap | No. The goodnight ritual, then "the camp is asleep". The parent can grant +10 min through the gate (a "bonus" button, not shown to the child; max 1 per child per day; audit-logged as `cap_bonus_granted` (§15.1, 11 §5.8 cls `admin`); it extends minutes, never adds an episode or any reward; applied at the next segment boundary like any cap change) |
| DD-10 | Number of choices in Plan | K2: 2, K1: 3 + a flame, K3: 3 doors |
| DD-11 | Default letter order (K2) | The default in the Hebrew spec (sound-first, high utility, confusables separated); the parent's "letter of the week" overrides |
| DD-12 | Which numbers K3 hears | 1–3 only (Give-N), in the lookup forms (feminine for rote counting; noun agreement as in the lexicon) |
| DD-13 | Level difficulty in co-op | Each child at their own band; co-op doesn't change the band |
| DD-14 | Content that a parent hasn't approved (generated) | Doesn't reach the child. Fallback to the static pack |
| DD-15 | Offline (no network) | Everything works. Co-op is limited to same-device. Sync later (00 E6) |
| DD-16 | An event that came out before the content was ready | It is postponed by days in the config without changing its dates on the calendar (it appears in the box as a "late gift") |
| DD-17 | A secret not found after 3 visits | On the 4th visit the pet points to it right away (goal-gradient) |
| DD-18 | Moving up a grade (Sept 2027) | Profile: K2 → grade 1 (moves to the "Explorer+" setting, more text with nikud); K1 → grade 4; K3 → gan (3.5): `child_settings.launchMode` changes `lap → coplayFirst` (§15.3: gate once per day, solo episodes ≤5 min = the playroom) **on parent confirmation only** (08 §10.5.7, 10 FAM-03), never automatically. A light placement test (a short expedition, 08 §10.5.7) runs again |
| DD-19 | Volume and music | Music −8 dB under voice-over (ducking). Separate sliders for music/SFX/VO for the parent |
| DD-20 | iPhone orientation | **09 §8.1 owns orientation (00 A3, J1):** game scenes, mechanics, co-op and movement = landscape (locked on iPhone); **the hub, map, album, shop, avatar, base and pet on iPhone = portrait-first, landscape allowed** (the hub camera pulls back into a vertical frame, campfire in the top third, 09 SCR-04); profile picker, wind-down and parent area = portrait. The earlier "hub landscape too" default is withdrawn (F9 GAP-29) |
| DD-21 | What the parent sees from co-op | Only "who took part" + positive behaviors (from guide mode) |
| DD-22 | A child's drawing failed in processing | It becomes a "sticker" with an artistic filter. Never "unsuitable" |
| DD-23 | Two siblings rescued the same friend in co-op | Each one gets "their" copy in their neighborhood (no splitting, no fights) |
| DD-24 | Content that is too easy for K1 (boredom) | Mystery/Stretch flame offered automatically after 3 levels with ≥95% |
| DD-25 | Daily surprise when the child skipped days | They accumulate up to 7, and then "a big letter" that bundles them |
| DD-26 | A level left mid-way and not resumed | Checkpoint state is kept for **7 days**; after that the level restarts from Ki with the same content slots, and friends already rescued in it stay rescued (they are never "un-rescued") |
| DD-27 | An episode interrupted (profile switch, app killed, cap) and not resumed within 24 h | The engine auto-closes it as `episode_closed{reason:'abandoned'}`: item grants already earned are paid on the next receipt, `episodeComplete` is **not** paid, the family-goal day counts only if ≥1 full level was completed (02 §18 UC-03) |
| DD-28 | Child input faster than the UI can respond (tap spam) | Input is debounced per target (300 ms K3, 200 ms K2/K1); during T2+ celebrations taps are swallowed except "skip"; a 3rd wrong answer in <800 ms each (K1) triggers the rapid-guess rule (02 §2.6) and the engine steps down one band for the next 3 items; never a message about speed |
| DD-29 | Child stuck on the same level across 3 visits | Visit 4 starts with the pet pointing at the next friend immediately (DD-17), the engine serves the level's items one band lower, and ניצי "takes a lantern breath" first; the level is marked `assisted` for the parent report only. There is no "skip level" offered to the child; the parent can mark a skill `notRelevant` (10 API-12) which removes that content |
| DD-30 | Child avoids every learning action (wanders, pokes friends, visits the shop) | Allowed; nothing hostile happens. Free play earns nothing (02 §2.7). After 2 consecutive episodes with 0 `item_answered` events the parent gets an informational card (not an alert); the next Plan step offers only 2 levels and no camp-first option |
| DD-31 | Device time zone differs from the family's (travel) | **One `DayKey` everywhere (00 C3 rev 3):** day boundaries for caps, surprises and the economy are the **calendar day in the family's city** (`family.tz`, midnight), never the device offset; each event's `tzOffsetMin` is that city's offset at event time, computed on the device, monotonic per device (02 §2.1). Shabbat/holiday windows use the same city. While the parent sets a **"עיר נסיעה"** (`family_prefs.travel.city`, a hebcal city, 10 SET-01; clearing it returns to the family city) both the day and the Shabbat window follow the travel city. A banner for the parent appears when `deviceTz ≠ family.tz` for >24 h (telemetry `tz_mismatch`) |
| DD-32 | Device clock jumps backward or forward | Grants use the device `ts`; the server clamps far-future time (11 §6.4). `DayKey` for caps and diminishing returns is **monotonic per device** (never earlier than the last DayKey seen), so moving the clock back cannot reset a day's caps; moving it forward only ends the day early (no gain). Skew >2 min is telemetry for the parent (11 §6.5) |
| DD-33 | Two devices offline, same child, same chest track item | 02 §6.3 rule 6: the later chest keeps the item, the earlier one shows "כבר יש לך, קיבלת X מטבעות במקום"; nothing is removed |
| DD-34 | CO-3 room: a device disconnects mid-finale | Its player is replaced by the scripted friend within ≤1 s (§12.1); rejoin within 15 min resumes the role (the `CoopRoom` closes after 15 min idle and has a **30-minute hard cap** per session, 11 §10 / 00 E5); the Durable Object emits `boss_defeated{participants}` once to all devices, so each child's grant derives from the same event id |
| DD-35 | Parent lowers a cap below the minutes already played today | The soft warning fires at once and the episode lands within ≤1:30; tomorrow's cap is the new one; nothing earned today is removed |
| DD-36 | App killed during the goodnight ritual or receipt | `episode_completed` is written **before** the ritual starts; relaunch shows "the camp is asleep" directly; the receipt is shown once on the next entry if it was never displayed (`receipt_shown` event) |
| DD-37 | Free-signing install expired mid-week (00 E1a) | Kids see nothing (the app just won't launch); the parent area showed a banner from day 5 and the app syncs at every episode end and on background, so at most the current episode's unsynced events are at risk; on re-sign over cable the data container is preserved `⟂VERIFY` (install the re-signed build over the existing bundle id with `devicectl` on a test device and confirm SQLite survives; if not, the re-sign script must run "סנכרון עכשיו" first via the parent area) |
| DD-38 | Content pack or audio clip missing (vendor/API failure, offline) | The engine never blocks on content: a missing level chunk falls back to the static pack (DD-14); a missing VO clip plays the text via the HUD only (K1) or the character's 2-note signature + a picture prompt (K2/K3) and logs `asset.missing`; a missing numeral clip is a **hard error in CI**, never at runtime (the table ships complete) |
| DD-39 | Reduce Motion / calm mode on | No shake, no confetti, springs → `spring.reduced` (09), chest reveal: beats 3–5 collapse to a single 300 ms glow, auto-open after 4 s is the default; parallax and camera bob off; nothing is skipped in content |
| DD-40 | Color-only cues | Never color alone (09 §3.4): chest tiers differ by shape, evening flames by icon (blue = snowflake, green = leaf, gold = star, purple = question mark), jars by icon + label, lit/unlit runes by glow + a check mark, heat map "warm/cold" by sound + size |

---

## 19. Open items for the parent (minimal; nothing here blocks: placeholders + defaults are in use)

> IDs in brackets are 00's deduplicated **Open Parent Decisions** table; where a default here and in 00 differ, **00 wins**.

1. **The children's names** and **K3's grammatical gender** (00 A2) [OPD-01, OPD-02]. Default: placeholders; K3 `n` (neutral) everywhere in band T (00 overrides DD-05's alternation).
2. **Shabbat mode:** `soft` / `full` / `off` (§15.4) + the city for zmanim [OPD-04]. Default: `soft`, Tel Aviv.
3. **Recording parent voices:** names + lines (≈20 minutes, a guided in-app recording) [OPD-12]. Default: no recording → TTS is never used for names; ניצי says "!אור בוקר" without a name.
4. **Taste approval (E10):** the world name, the hero (פנסון), the cast (a sheet of 9 characters incl. Pip), 6 pets × 5 stages, chest sheet [OPD-11]. The first defaults are used until approval.
5. **Grandparents** in the game? (Yes/no + names.) [OPD-22] Default: no.
6. **The family boss day** [OPD-07]: default Thursday.
7. **Movement constraints at home** (space, neighbours, hours; is the iPad on a stand?) [OPD-14]. Default: seated alternative always offered; jump moments off on iPhone-in-hand.
8. **Children's drawings for co-design** (3–5 each, hero + lost friend) and consent to send them to the image model [OPD-10]. Default: drawings stay on device, processed locally only (§3.4).

---

## 20. Proposed change to 00

> The decisions in 00 remain binding until updated. Each proposal includes the reason.

1. **E5 (Backend) — real-time co-op: ACCEPTED (00 rev 2, Appendix A).** E5 now lists Durable Objects (`CoopRoom`, WebSocket Hibernation, auto-close after 15 min idle; 11 adds a 30-min hard cap). Each "family room" is one DO: authoritative event ordering, low latency.
   - **Fallback (still binding):** without Cloudflare approval CO-3 is disabled and co-op is same-device only (CO-1/CO-2); the vertical slice runs fully local (E5), so CO-3 is first testable at the MVP sprint that deploys staging.
2. **B7 — explicit list of quiet days:** Yom HaShoah, Yom HaZikaron, Tisha B'Av, Yom Kippur = no events, no boss, celebrations up to T1. **Yom Yerushalayim** = a neutral day with no event (political sensitivity).
   - **Reason:** "memorial days" in 00 isn't specific enough for a calendar config.
3. **B8 — add:** no fireworks (including Yom HaAtzmaut); no darkness >40% of the screen and a minimum luminance in evening scenes; for K3, dusk only.
   - **Reason:** R8 (age of onset of fear of the dark), and fireworks are explosions.
4. **D1 — clarification:** "sparks" (`spark`) in levels are a **non-persistent in-episode resource** that doesn't convert to coins. They are not a third currency.
   - **Reason:** they serve as breadcrumbs (G1) without breaking the "mastery, not time" rule.
5. **D2 — "upgrade steps" in the reveal: WITHDRAWN (Fable review).** 02 §6.5 and 00 D2 keep the tier visible from the first frame with a fixed tap count per tier; effort marks are receipt lines (§16.2). Nothing to change in 00.
6. **E1b — clarification of "Hebrew text only in the 2D HUD":** in-world glyphs (one letter/syllable as a learning object) are allowed as **a pre-baked image texture** (`tools/glyph-baker`, nikud validated), not text.
   - **Reason:** without this, "a rune on a stone" (MX-H01) and letter-friends can't exist in the world.
7. **C7 + G4 — rule:** the player's hero (פנסון) is a **silent vessel**, not AI and not a character with dialogue. The AI helper (גלגלון) must have a distinct shape language (boxy, a screen face, gears, an "AI • מכונה" label).
   - **Reason:** to prevent "robot hero" (G4) and "AI helper is a robot" (C7) from blurring together for K2/K3.
8. **C3 + G3 — clarification:** a K1 level is 6–9 minutes (an episode of 15–20 includes 1–2 levels + camp + evening). G3 only sets K2 (3–6).

---

## Appendix A — Schemas (for the developer)

```ts
// packages/content/src/schema/episode.ts
export type SegmentKind = 'arrival_plan'|'warmup_review'|'level'|'block_b'|'camp_show_build'|'evening'|'goodnight_tell'|'peekaboo'|'copy_me';
export interface EpisodeTemplate {
  id: 'episode.k1.default'|'episode.k2.default'|'episode.k3.lap'|string;
  child: 'k1'|'k2'|'k3';
  targetSec: [min: number, max: number];          // k2: [480, 720]
  segments: Array<{ kind: SegmentKind; targetSec: number; itemCount?: [number, number]; newRatio?: number; optional?: boolean }>;
  celebrationBudgetRatio: 0.15;
  movementMomentsMax: number;                      // k2: 2, k1: 2, k3: 1
}

// packages/engine/src/mechanics/types.ts
export interface MechanicDef {
  id: MechanicId;                 // 'MX-H01'
  code: string;                   // 'rune-lens'
  ages: Array<'k1'|'k2'|'k3'>;
  skills: SkillTag[];
  inputs: Array<'tap'|'drag'|'trace'|'voice'|'keypad'|'movement'|'tilt'>;
  knobs: Record<string, { min: number; max: number; default: number } | string[]>;
  failureLadder: Array<'retry'|'hint1'|'hint2'|'workedExample'|'stepDown'|'switchActivity'>;
  compactForms: Array<'battle'|'evening'|'runnerGate'|'review'>; // one item bank → many modes
  usesRT: false | 'k1AfterMastery';
}

// celebration tokens (packages/ui/src/tokens/celebration.ts)
export const celebration = {
  t0MaxMs: 800, t1MaxMs: 1500, t2MaxMs: 3000, t3MaxMs: 6000, t4MaxMs: 10000,
  skipAfterMs: { t2: 1000, t3: 1000, t4: 2000 },
  shakePx: { k3: 0, k2: 4, k1: 8 }, maxFlashHz: 3, budgetRatio: 0.15,
} as const;
```

## Appendix B — Telemetry events required for design metrics (on the device, sync per 00 E6)

| Event | Fields | Feeds |
|---|---|---|
| `level_beat_end` | levelId, beat, durationMs, itemsCorrect, itemsTotal, hints | AC-LV-2, difficulty |
| `friend_rescued` | friendId, kind(path/secret), secretType, msSinceLevelStart | Secret density, G1 |
| `celebration_played` | tier, durationMs, skipped | AC-CEL-2 |
| `coop_light_share` | sessionId, childId, lightUnits, activeMs | AC-CO-2 |
| `movement_moment` | momentId, mode(full/seated/tap), completed | G6 |
| `free_choice_unrewarded` | activity (music hut, collection room, playroom, `camp_asleep`) | The overjustification metric (report §"how we know we won") |
| `stop_ritual` | completed, overtimeMs, capReason | AC-STOP-* |
| `cap_bonus_granted` | childId, minutes (10), grantedBy (guardianId or `device-local`), dayKey | DD-09 audit (cls `admin`, 11 §5.8) |
| `turn_yielded` | childId, sessionId, nextChildId, plannedShare | §12.5 Patience heart; AC-CO-8 |
| `ftue_completed` | childId, deviceId | §14.2 (sets `child_resume.ftueCompletedAt`) |
| `tz_mismatch` | deviceTz, familyTz, hours | DD-31 banner |

**Product metrics (for the parent, not for the children):**
- "learning seconds per minute of play" (target ≥0.6 for K1/K2);
- delayed review after a week;
- free choice of activities with no rewards;
- persistence after a mistake;
- number of creations.

---

## 21. Use cases and edge cases (gameplay side; the money side of each is 02 §18 with the same UC id)

> Format: scenario → binding rule(s) → acceptance criterion `AC-UC-nn`. Rules reference the section that owns the mechanism; nothing here introduces a new number that 02 owns.

| ID | Scenario | Binding rules (gameplay) | AC-UC-nn (testable) |
|---|---|---|---|
| UC-01 | **First run, K2** | §14.2 K2 flow; 3 real endowed rescues in episode 1 (§14.3); first egg = his first domain (§4.12); no text; the Plan offers 2 levels only from R-03/R-05; the tutorial is "learn by doing", no modal | Playwright: from profile pick to first friend ≤60 s of scripted time with 0 text nodes in the child layer; `friend_rescued` ×3 by end of episode 1 |
| UC-02 | **First run, K1** | §14.2 K1 flow; "Skip" after 3 s; names her פנסון; placement = 08's expeditions as story-wrapped `mode:'placement'` levels (§14.2–14.3: E1 Sun … E4 Wed, 8–10 min each); egg choice 1 of 2; joystick offered once, remembered; FTUE never replays (`ftueCompletedAt`) | Snapshot: skip button visible at t=3 s; `placement` events never produce grants (02 §2.7); joystick preference persisted per child; AC-FTUE-1, AC-PLC-1 |
| UC-03 | **First run, K3** | §14.2 K3 flow; parent gate mandatory; פופי; no wrong answers; parent prompt on screen; 4:00 episode | AC-K3-1/2 hold on the first episode; the parent prompt layer is `parentOnly` (small text, not voiced) |
| UC-04 | **Returning after 3 weeks** | §5.5 surprises accumulate (max 7 → "big letter", DD-25); pet asleep → wakes happily (§4.12); campfire unchanged (never dims, §10.4); levels left mid-way restart after 7 days with rescued friends kept (DD-26); the Plan shows "ברוכים השבים" with a warm-up of **due reviews only** (no new skill in the first episode back); nothing says "you were away" | Fold over a log with a 21-day gap: no counter decreases (02 §14.1); the first episode back has `newRatio = 0`; the mailbox holds 7 items + 1 bundle |
| UC-05 | **Sibling grabs the device mid-level** | §12.5 instant save; switching profiles needs the picture-PIN if set (00 H3); the interrupted child's beat state is saved at the last checkpoint; `session.ended{reason:'parent_exit'}` + `turn_yielded` when the §12.5 rule holds → "Patience heart" + a VIP surprise next time; DD-27 auto-close after 24 h; the other child's episode starts normally; K3's profile cannot be launched this way (gate, `launchMode:'lap'`) | Test: switch at any beat → resume lands on the same beat with the same content slot; no grant is lost or duplicated (02 §18 UC-05); K3 launch from the switcher is refused without the gate; AC-CO-8 |
| UC-06 | **Two kids, one iPad, co-op turn-taking** | DD-08 turn ring; CO-1/CO-2 modes; each child's items at their own band (DD-13); shared meter only (§12.4); each gets "their" copy of a shared rescue (DD-23); co-op minutes count 50% to each cap; the session ends with **one** goodnight for both (each child's `episode_completed` written) | AC-CO-1..6; test: a CO-2 session emits one `episode_completed` per participant and one shared `coop_session` id |
| UC-07 | **K3 lap mode, parent walks away** | §15.3: the playroom (MX-P01) auto-stops after 5 min with a lullaby; no exits, no menus, 12 mm dead margin; no learning segment runs without a parent prompt answered ("עשינו!") within 60 s — otherwise the episode switches to playroom; `solo minutes in learning segments` reported to the parent (10 M-14, target 0) | AC-K3-1; test: no parent tap for 60 s in a lap segment → playroom; playroom hard-stops at 5:00 ± 5 s |
| UC-08 | **Offline for days** | DD-15 everything works; content from the installed packs; daily surprises generated locally from the seed; co-op same-device only; chests open offline (contents already in the event); sync on reconnect merges by event id (00 E6) | Playwright with network off for a simulated 5 days: all episodes playable; on reconnect `rejected = []` and no duplicate grants |
| UC-09 | **Shabbat / holiday / quiet day** | §15.4 modes (`soft` default); window via `@hebcal/core` (B7); `soft`: scenery, counters frozen, levels as free practice, no coins/gems, no events open/close; `full`: app sleeps for kids; quiet days: no boss/events, ≤T1, no siren-like SFX, chests go to the shelf (02 §6.5); Hanukkah night 1 on Friday opens after havdalah cumulatively (AC-EV-3) | AC-SHAB-1/2/3; AC-EV-3; test: a chest earned at 15:00 on a quiet day sits on the shelf with `autoReveal=false` |
| UC-10 | **Child spams taps / rapid guesses** | DD-28 debounce; K1 rapid-guess rule (02 §2.6, silent); after 3 rapid wrongs the engine steps down and ניצי models "lantern breath" once; K2/K3: no RT in scoring (AC-MX-4), the UI simply ignores taps during the 0.6 s bounce; K3 co-op taps rate-limited 1/0.8 s (§12.4) | Test: 20 taps in 2 s on a K2 item register ≤4 attempts; no string about speed exists (lint) |
| UC-11 | **Parent changes caps mid-session** | §15.1: applied at the next segment boundary; lower-than-elapsed → land within ≤1:30 (DD-35); raise → immediate room; settings doc LWW by HLC (11 §6.5); the child sees no message | Test: cap 25→10 at minute 12 → `stop_ritual{capReason:'parentChange'}` within 90 s of the boundary; cap 25→40 → no stop |
| UC-12 | **Child stuck on a level 3 times** | §6.5 frustration guard per item; DD-29 per level (pet points at once, band −1, `assisted` flag); boss phases move to Cozy at ≥3 errors (§9.2); never 4 errors in a row on one item (AC-LV-6) | Simulation: a 0%-accuracy bot completes any K2 level in ≤8 min with ≤3 errors per item and reaches Ketsu |
| UC-13 | **Refuses learning but wants rewards** | DD-30; no reward path without learning (P-1, AC-LV-3, AC-ECO-1); the shop/collection room are free to browse (02 `neverEarns`); camp-first entries >40% → the app opens on the map (02 §14.3) | Test: an episode with 0 `item_answered` yields 0 coins, 0 XP, 0 chests, and still ends with the goodnight ritual (no punishment, no lecture) |
| UC-14 | **App killed during a chest reveal** | §16.2 idempotent reveal (`chest_earned` → `chest_opened` at beat 6); relaunch plays beat 8 only or leaves the chest on the shelf | AC-ECO-4 |
| UC-15 | **Clock tampering** | DD-32 monotonic DayKey; server clamp; skew telemetry; Shabbat windows prefer `serverTime` when the last sync is <24 h old, else device time | Test: set device clock −2 days → caps unchanged; +10 days → `DayKey` advances once, no bonus paid early (02 §18 UC-15) |
| UC-16 | **Device time zone ≠ family** | DD-31 | Test: device in `America/New_York`, family Tel Aviv → episodes count toward the NY-local day; Shabbat scenery follows Tel Aviv until "travel city" is set |
| UC-17 | **CO-3 disconnect** | DD-34; §12.1 drop-in/out; Durable Object authoritative (00 E5 `CoopRoom`, accepted); offline → CO-3 unavailable, CO-1/2 offered; light contributed while disconnected merges into the next evening's meter, never a retroactive fire level (§10.4) | AC-CO-3; test: host drop during the finale → the finale completes on the DO and every device receives the same `boss_defeated` id on reconnect; `fireLevel` after the merge equals the DO's count |
| UC-18 | **K1 guide-mode overuse** | §12.6 two-level opt-in, 1/day ≤10 min, brain-token gate, no input on his items, no stats | AC-CO-5; test: a second offer on the same day never renders; mentor progress stops at 10:00 |
| UC-19 | **Accessibility: color blindness** | DD-40; 09 §3.4 | Playwright with a CVD filter: every state pair (lit/unlit, tier, flame, jar) remains distinguishable by shape/icon |
| UC-20 | **Accessibility: motor** | 09 §10: tap-tap alternative for every drag (`dragOrTapTap`), long-press ×1.5 setting, targets ≥ §6.3 sizes, no multi-finger gesture required, handedness mirror, chest auto-open setting, Switch Control focus ring | Test: every mechanic completes with single taps only (`inputMode='tapTap'`); 0 required multi-touch gestures (lint on gesture handlers) |
| UC-21 | **Accessibility: Reduce Motion** | DD-39; §16.1; 09 §5.5 | Animation test: with RM on, no `shake`, no `confetti`, all springs = `spring.reduced`, chest beats 3–5 ≤300 ms total |
| UC-22 | **Accessibility: hearing** | 09 §10: captions for every VO (default on for K1 and the parent; toggle for K2), every meaningful sound has a visual twin (sound tells → glints, counting voice → numerals shown, English phonics → mouth-shape card), haptics on iPhone as a third channel | Test: with audio muted, every secret type in §6.2 still has a visual cue; every K2 instruction has an icon prompt |
| UC-23 | **Vendor/API/content failure** | DD-14, DD-38; nothing kid-facing is generated live (00 C7, E9); the static pack is always installed | Test: delete a level chunk and a VO clip → the episode still completes; `asset.missing` telemetry recorded; numerals table completeness is a CI gate |
| UC-24 | **Grade change (Sept 2027)** | DD-18; bands re-evaluated by `bandOf(child)`; a light re-placement; nothing in the collection or hub changes | Test: K2's band `G→S` flips text rendering to band S without touching rescues, buildings or pets |

---

## Fable review changelog (reviewer F2, 2026-11-02)

- **Hero naming (00 G4, parent request):** §2.1, §3.1, §3.2 rewritten — the פנסון is a "toy-suit with a living spark", never a robot; vocabulary, part names (`antenna`→`topper`, no claws/wheels), spike base mesh and prompts scrubbed; gear icon reserved for גלגלון; new lint AC-HERO-1; glossary note ניצוץ vs ניצוצות.
- **01 ↔ 02 reconciliation (00 D4/D7):** §11.2 numbers table removed and replaced by a pointer + 9 design requirements; §16.2 "effort steps" withdrawn (02's fixed taps per tier, tier visible from frame 1), effort marks moved to receipt lines; chest tiers renamed wood/silver/gold/legendary with 02's shapes; §16.1 maps wood→T2 and silver/gold/legendary→T3; §9.2/§9.3/§13.3 rewards now cite 02 triggers; §10.2 thresholds replaced by 02's paced config values + per-season building; §10.3 species set → accessory + chest, not evolution; §4.12 pets rewritten to 02 §8 (6 species by domain, 5 stages, mastery thresholds, treats, friendship; K3 = פופי the bunny); §10.4 campfire **never resets** (unbounded level, color rings per 10, lit-zone cap), logs as the shared unit; §20 proposal 5 withdrawn.
- **Region unlocks made precise (§2.2, §10.4):** story-calendar-or-fire-level (earliest) AND readiness, with a per-region table; learning is never gated by a shared counter (00 G2).
- **Astro-Bot pillars completed:** rescue counts per region (§9.2: K1 57, K2 41 for 8 levels), hub fill states F0–F5 (§10.2), secret levels secretA/B/goldenRemix referenced to 02 §5.3, day/night rule by episode phase (§9.5), capsule machine (deterministic, next capsule visible) in §10.3.
- **Requests from other specs incorporated:** 03 → MX-M13..M20 + variants (lasso, cart/hut, table-map, board-game); 05 → MX-E01 `pairs` mode, MX-E05..E07, CH-11 Pip (silent crab), "bottles from the sea" warm-up in S0 (§13.2, §5.3); 07 → MX-T01 unified with `TOOLBELT` (tool cards, states, "which tool?" fan), MX-T06 brain-token gate across all hint ladders, ritual minutes mapped to episode segments (`ritual_mid`, open+close ≤10%); 06 → MX-X02 content skins, MX-X05 build-sim, hub weather station, camp lantern, never-wilting garden. Catalog now 64 mechanics with a Tier A/B/C build order (§7.1); full cards in new §7.3.
- **00 rev 2 applied:** A6 bands + C15 level names in §0; B7 `@hebcal/core` window and `soft` default replace the NOAA paragraph in §15.4 (AC-SHAB-3); C13 safety tags on offline missions (§8.1, §15.1, AC-STOP-4); E3 device-in-hand check before jump moments and optional controller (§6.3, §8.1, AC-CTRL-1); B9 direction conventions noted in §6.3; §19 open items carry OPD ids and defer to 00 defaults.
- **Caps/bedtime aligned to 10 §6.13** (K3 15 min, K2 bedtime 20:00), "app sleeps" windows, mid-session cap-change rule, +10 min bonus rule (§15.1, DD-09).
- **Guide mode** (§12.6): parent-enabled first (10 default off), cap behaviour mid-session, brain-token link.
- **Endowed progress must be real rescues** (§14.3, per 02 §5.2 rule 8).
- **New §21 use-case/edge-case matrix** (24 scenarios) with binding rules and AC-UC-nn; new DD-26..DD-40; new ACs (AC-ECO-3/4/5, AC-HERO-1, AC-REG-1, AC-FIRE-1, AC-PET-1, AC-MX-6..9, AC-CTRL-1, AC-STOP-4, AC-SHAB-3).
- **F3 (04 Hebrew) items processed:** MX-H12 `rune-splitter` (the key K2 predictor `H.PA.PH.SPLIT`, Tier B shipped in S0), MX-H13 `twin-detective`, MX-H14 `writing-studio` (parent-mediated, not tracing), MX-H15 `word-machine` added with full cards (§7.3); variants syllable-clap (H05), summary-chest/story-campfire (H07), readers-theater/echo-cave/phrase-bridge (H08), prefix-peeler (H11) recorded; 04 §7.3א mapping onto MX-H01…H11 confirmed (§7.1); `hint:lens` event on MX-H06 (AC-MX-11); co-op touch zones + `assisted:'parent'|'sibling'|'none'` tagging rule (§12.2, AC-CO-7); catalog now 68, Tier B 23. 03 §5.4's eight math proposals were already adopted as MX-M13…M20 (F3 #17).
- **Still unverified (`⟂VERIFY`):** DD-37 — whether a re-signed build installed over the existing bundle keeps the SQLite container (test with `devicectl` on one device before relying on it).

### Round 2 changelog (reviewer R2, 2026-11-02)

Disposition of every issue targeting this file is in `docs/review/round2-R2.md`. Changes:

- **Placement (F9 GAP-12/13):** §14.2 now states that 08's mapping expeditions exist as story-wrapped `mode:'placement'` levels (08 §3.6 UI rules, no grants, rescues count); §14.3 week-1 table allocates expedition slots (K2 Sun–Tue, K1 Sun–Wed) and a week-1 episode template; `mode` added to the level schema (§6.6); UC-02 updated; AC-PLC-1.
- **FTUE once (GAP-01):** `child_resume.ftueCompletedAt` rule + `ftue_completed` event; AC-FTUE-1.
- **Cap/closing/grace unified with 08 §6.10 (GAP-02):** 85% → `closing`; cap inside a beat → ≤1:30 landing; 08's 60 s grace only inside `closing`; `cap_bonus_granted` event for DD-09 (Appendix B); AC-STOP-2/5.
- **Caps/bedtime (GAP-03/04):** K3 15 min and 10's bedtimes kept (already aligned in round 1; rationale in the review file); effective lock = earliest of child/device bedtime.
- **English island:** `palette.english` replaced by 09's `region.englishIsle` tokens (F4 #18); Mr. Mix-Up = CH-04 in island costume with a separate en-US voice (F4 #13; §2.4, §4.1, §4.5); bottles phase in §2.4 with dates and week-6 confirmation (F4 #14); Hebrew cap warning deferred to island exit (GAP-30; AC-MX-3); **new §7.4 English asset-ID list** (F7 #16).
- **Cast roles for 06/07** after §4.1 (F4 #12); **06/07 mechanic-name mapping table** after §7.3 (F4 #15).
- **Check-in cadence** made band-specific and identical to 07 §2.4 (S every episode, G every other day first episode, T parent-led) in §5.3 and MX-S01 (GAP-31, F4 #16).
- **Weekly boss (GAP-63, DD-06):** absent/paused children excluded from `bossHP`, finale with any subset, quiet-day/Yom Tov window move; DO emits `boss_defeated` once; AC-BOSS-3.
- **Campfire:** level-ups only from the four events / DO summary (GAP-21); family garden placed next to the fire, cosmetic only (F4 #11/#17); 4th child not supported in v1 (GAP-53).
- **Co-op:** CO-4 structured authoring + `coop_letter` doc (GAP-46); `turn_yielded` detection rule for the Patience heart (GAP-33); AC-CO-8; DD-34 30-min room cap.
- **Stopping:** K1 `camp_asleep` view state (GAP-43); "back to camp" = `session.ended{reason:'child_exit'}` with `secondsSinceLastError` (GAP-25, 08's canonical enum); car mode marked P2 with its rules (GAP-19); printing via `expo-print` A4 RTL PDFs (GAP-50); `launchMode` lap/coplayFirst/solo (GAP-11b, DD-18).
- **Shabbat window applies by session start (GAP-14d);** DD-31 rewritten per 00 C3 rev 3: one `DayKey` = calendar day in the family city / travel city (`family_prefs.travel.city`), never the device offset (R5 blocker #1, GAP-39/44). **§15.1 bedtime** cites 10 §6.13's `preBedtimeMin` (60 default, 15–60), `mini` planning, effective lock = earliest (R5 #4b); **§14.2** reworded as child experience of 08's expeditions (R5 #4a).
- **Chests:** `chest_earned` is a derived fold output with `chestId = hash(sourceEventId)`; only `chest_opened` is persisted (F6 #2).
- **Praise:** §16.3 now points to 09 §9.2 P19–P25 per row; new lines go to 09 first (F6 #1). **Routes:** AC-WORLD-1 cites 11 §4.1 route names and 09 SCR ids (F6 #2).
- **DD-20:** iPhone hub portrait-first per 09 §8.1 (GAP-29). **§20 #1:** Durable Objects marked accepted (00 E5). **§4.10/§14.1:** parent voice clips per 12 §6.11 (GAP-67).

### Final sweep changelog

Final consistency sweep, 2026-11-02 (one editor for all files; details in `docs/review/final-sweep.md`, register in `16-verify-register-and-index.md`).

- **§7.3 MX-H12…MX-H15 cards and AC-MX-10:** Hebrew error codes renamed to 04's final namespace — `EH.PA.CVWHOLE`, `EH.PA.ADDVOWEL`, `EH.VIS.<PAIR>`, `EH.SPL.PHON`, `EH.MO.PLURAL_IRREG`, `EH.MO.GENDER_AGR` (round2-R3 #2; `E.*` is reserved for 05's skill ids).
- **§7.3 MX-M14 / MX-M19 skill ids:** `M.FRAC.COMPARE.*` (was a truncated id) and `M.DATA.PICTO.G` (03's id).
- **§14.3 K1 Tuesday row:** English placement timing reconciled with 05 §14.1 — `english.listening` + `english.letter_sounds` probes run in 08's E3 (week 1); 05's week-3 "המכתב של Sunny" bottle wraps the lazy probes.
- **Decision references:** §3.3 `00 D14` (no such decision) → `00 A4` + `E10`/OPD-10; §12.1 CO-4 "00 C7, F9 GAP-46" → "00 C7a; 15 GAP-46"; §6.4 and §7.3 cite B9's sub-rules as `00 B9 (a)/(b)/(d)`.
