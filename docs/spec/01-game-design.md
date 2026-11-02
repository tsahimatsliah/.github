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
| Hero | A lantern-being piloted by the child, co-designed with the children. | — | A specific robot (IP not used). |

No names, silhouettes, sounds or story framing are taken from existing games. Rules:
- The hero is never white with blue eyes.
- No "crash site" or spaceship.
- No PlayStation-style controller imagery.

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
| R-08 | **Sunny Harbor** | English (oral for K2, phonics + vocab for K1) | Turquoise-yellow harbor, lighthouse, English signs only (LTR) | `MX-E01` echo shell |
| R-09 | **מוסך גלגלון** | AI literacy, computational thinking, debugging | A workshop of bots, conveyor belts, screens | `MX-A01` remote controller |
| R-10 | **עץ השקט** | SEL: emotions, breathing, problem solving, perspective-taking | A giant tree with a treehouse, soft light, a creek | `MX-S02` lantern breath |
| R-11 | **שביל הגחליליות** | Mixed review in runner form (all subjects) | A twisting path through the forest at dusk | auto-run (`MX-R01`) |
| R-12 | **חדר הגחליליות** | K3: colors, shapes, animals, 1–3, body parts, first words | A soft playroom of toys (2.5D isometric) | — (open play) |

**Region unlocks**
- Regions open through **the family campfire level** (see §10.4) together with each child's skill readiness. This follows 99 Nights' Missing Kids Board (R12).
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
  - Accent color `palette.english = #19B7B0` (turquoise) plus yellow.
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
| 6 | "ספינה מחו"ל" | The English island opens, Sunny arrives | Mr. Mix-Up (mixes up English words) |
| 7 | "הסדנה של ברגית" | Big building, the bridge to the island | צריח עקום (a tower built crooked) |
| 8 | "למה הבלגניסט מבולגן?" | Revealed: the Muddler sneezed because he was cold and alone outside the camp | The Muddler himself (puzzles with no "defeat"; he joins as the camp's MC) |

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

- **The פנסון** is a small lantern-being with a robot-ish feel. It is a mechanical vessel the child "pilots". The child's avatar sits inside a glass dome on its head, so the face is visible.
- Design goals:
  - Original (no shape, color or detail from Astro Bot or any other game).
  - Lovable and very animatable.
  - Drawable by K2.
- **The פנסון does not talk and is not AI.**
  - It has no voice lines and no opinions. It expresses itself only through animation and short sounds (beeps, a mini-flute).
  - So it never blurs with the AI helper גלגלון (00 C7). The two must look different (see the CH-03 card): the פנסון is round and warm with a lantern core, while גלגלון is boxy with a screen face and gears.
- **Name options for the hero type** (the children can give *their* פנסון a personal name in onboarding):
  1. **פָּנָסוֹן** (default)
  2. נִיצוֹצוֹן
  3. זוֹהֲרוֹן

### 3.2 Hero anatomy (parts system)

| Slot | Options at launch | Notes |
|---|---|---|
| `chassis` | 3: `chassis.round` (K3 default, a wobbly ball), `chassis.explorer` (K2 default, egg-shaped with short legs), `chassis.ranger` (K1 default, slimmer, with a backpack) | Each chassis shares one skeleton of ≤30 bones (E1b) |
| `dome` | 4 shapes (round, square-ish, bubble, double) | The avatar's face shows inside, rendered as a 2D/texture projection onto the dome |
| `core` | 12 colors of "inner flame" (the lantern core) | Lights up with each correct answer (the core juice moment) |
| `antenna` | 8 (spring, flower, star, leaf, flag, ear, horn, none) | Sways with secondary motion |
| `arms` | 4 (short, mechanical claw, gloves, grabber) | The visual for interacting with power-ups |
| `feet` | 4 (legs, wheels, springs, suction-cup feet) | Changes only the walking animation, not speed or ability |
| `decal` | Unlimited: **drawings by the children** (scanned, see 3.4) plus 30 earned stickers | Placed on the chassis in 3 positions |
| `trail` | 10 (sparks, bubbles, leaves, notes, rainbow…) | Cosmetic, from the shop or milestones |

- **Rule:** no part changes gameplay ability (00 D3). Power-ups belong to the level, not to the hero.
- **3D pipeline:**
  - Spike: Quaternius CC0 robot base (E1b).
  - Final: a custom design (Meshy/Tripo → Blender ≤5k tris → rig).
- **Mandatory animation clips (8):** `idle`, `walk`, `run`, `jump`, `celebrate`, `wave`, `found_it`, `oops_cute` (a cute stumble, never "sad").

### 3.3 Avatar: a child who looks like themselves without a photo (00 D14 rules, Mii approach)

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
- **No photos, no face scanning, no image generation from a photo** (00 D14).
- **Avatar emotions:** `happy`, `proud`, `surprised`, `thinking`, `silly`, `sleepy`. The avatar never "cries" in the game.
- **Gender:** no item is gender-locked. The catalog is never filtered by gender (`gap_audit` §4.14).

### 3.4 Co-design with the children (G4)

1. **"מעבדת הפנסון"** (onboarding plus whenever they like): the child picks parts. The choices count as "design".
2. **"תכנון אמיתי" (offline mission):** the app prints (00 C10) an "outline of a פנסון" page.
   - The child colors and draws "what my פנסון has".
   - The parent photographs it (parent gate). The image is processed on the device: crop, background removal, palette quantization to 8 colors.
   - It becomes a `decal`, or a new `antenna` from a template if the parent picks "turn into an antenna".
   - No cloud processing is needed. If quality is poor, a filter turns it into a "sticker".
3. **"הצבעה משפחתית" (once a season):** ברגית presents 3 new part designs (prepared from the children's ideas: the parent notes "she suggested a pineapple antenna" and the content pipeline produces 3 options). Each child votes with an emoji. All three are added to the catalog; the vote only decides the order they release in, so there's no losing.

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
| CH-04 | **הַבַּלְגָּנִיסְט** (מר בלבול, הקוסם בלגן) | Shadow magician (comic villain) | m | Purple-gray + gold hat | Weekly boss, season arc, becomes a friend | Every subject (mixes them up) |
| CH-05 | **הַצְּלָלוֹנִים** | Round shadow blobs, a group | pl (each one m/f at random, fixed per individual) | Soft gray-purple → glowing when lit | "Enemies" that become friends | Every subject |
| CH-06 | **בְּרָגִית** (קוֹצִית, פַּטִּישׁוֹנָה) | Hedgehog engineer | f | Red-orange + blue overalls | Building, measuring, geometry, works in progress | Math (space), building |
| CH-07 | **דֻּבְדּוּבָה** (מיכמש, דובית) | Large soft bear | f | Pink-lilac | Calm, emotions, Tree of Quiet | SEL |
| CH-08 | **פְּרוֹפֶסוֹר לָמָּה** (למה-למה, זיקיתוש) | Chameleon scientist | m | Green that changes with "hypotheses" | Curiosity, experiments, science | Science, world |
| CH-09 | **משפחה** (parents, grandparents, family pet) | NPCs built by the parent | per person | — | "Camp elders", recorded voices | Personal support |
| CH-10 | **נִיצוֹצוֹת** (the rescued friends) | Species of light creatures | per individual | By species | The collectibles, filling the hub | By species (see 4.11) |

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

### 4.12 Companions (pets): "גוזלי אור" (00 D4)

- Every child has **1 active companion**, plus a collection that grows at milestones (3 at launch, then +1 per mastered skill cluster).
- **It hatches from an "egg of light"** that the child received at a milestone. The egg's contents are known in advance (silhouette, D2).
- **3 evolution stages:** `baby` → `young` → `grown`. They are fixed by mastery of the cluster the pet is tied to (gap D9). Example: "the dragon of light" is tied to the letter cluster א–ל; "young" means 50% of the cluster mastered, "grown" means 100%.
- **Never dies, never gets sick, is never hungry.**
  - Feeding (1 tap per episode, using coins or a free daily treat) is a show of affection, not a need.
  - If not fed, the pet is fine.
- **In-game role:**
  - Follows the פנסון in levels.
  - **Points with its nose at a nearby secret** (a 3-second hint, after 20 seconds of no progress).
  - Cheers on a correct answer.
  - Sleeps at the end of the episode.
- **K3:** the pet is "פּוּפִּי" (a soft round creature). It reacts to every touch with a laugh or a sound and doesn't evolve. Instead, it gets a hat or a color at every sticker milestone.
- **Launch catalog (9):** light dragon, crystal turtle, fog cat, bubble fish, cloud fox, note owl, leaf hedgehog, star rabbit, mini gear robot (K1, AI-literacy cluster).

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
| 0:00–0:40 | **Arrival + Plan** | The פנסון wakes up in the tent, ניצי says "אור בוקר". Optional feelings check-in (1 tap, every other day). Lost Friends Board → choice of 2 levels | — | Voice only, big pictures |
| 0:40–2:00 | **Warm-up review: "בוקר טוב לחברים"** | Walk through the neighborhood: 3–4 friends rescued earlier are "asleep". Wake each one by doing its item ("which friend says /b/?", tap the right one; "show me 4 crystals") | 3–4 due items from the SRS ladder (gap D9) | Short, easy (≥90% expected) |
| 2:00–2:45 | **Level: Ki (introduce)** | Entering the level through the gate. ניצי demonstrates the gimmick with the new item, in a safe place | New skill: 1 item, worked example | No mistake possible (an action with a single target) |
| 2:45–4:15 | **Level: Shō (develop)** | The gimmick plus movement/height. 2 friends rescued. **Movement moment** (MV-*) ≤45s | 4–6 items of the new skill | Hint after 2 errors (gap D10) |
| 4:15–5:45 | **Level: Ten (twist)** | A twist that requires understanding (the gate swaps the order, a shadowling scrambles). 1–2 friends + interleaved practice | 4–6 items: 60% new, 40% older mixed | Secret room available |
| 5:45–6:30 | **Level: Ketsu (conclude)** | An easy celebratory finish: running to the gate, the last friend + T2 | 1–2 easy items | Lands on success ("land the plane") |
| 6:30–8:00 | **Camp: Show + Build** | The friends move in (T1 each, quick). Coin receipt. Place one decoration or build mission (`MX-M10` simple), feed the pet | Optional build mission: 2–3 counting/shape items | The coins appear *after* the action (00 D1, Lepper) |
| 8:00–9:20 | **Short evening by the campfire** | 3–4 sleepy shadowlings come to warm themselves. Light each one with an item (review) → they become friends | 4–5 mixed review items | Calm, dusk palette, not dark |
| 9:20–10:00 | **Goodnight + Tell + offline mission** | Tell (1 tap). The friends yawn and go to sleep. Offline mission card (picture + voice). ניצי: "שיחקנו יופי…" → the app sleeps | — | No autoplay; reopening goes through the parent gate (00, `child_psychology` §7D) |

#### K1 (8.5): `episode.k1.default`, 18:00 (range 15–20)

| Time | Segment | What happens | Content |
|---|---|---|---|
| 0:00–1:00 | **Arrival + Plan** | Lost Friends Board (map of regions with silhouettes). Choice of quest + flame (Cozy/Stretch/Mystery) + goal and prediction | — |
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
| 0:00–0:30 | **Hello** | The parent opens (gate). פופי peeks out of a box: "!קוּקוּ" + the child's name (recorded by the parent) |
| 0:30–1:00 | **Plan** | 3 big doors with pictures (animals / colors / music). The child points, the parent taps, or the child taps |
| 1:00–2:40 | **Peekaboo level** | 3 friends hiding in a playroom diorama (2.5D isometric, fixed camera). Tap a bush → it shakes → a friend appears and says its item ("!פָּרָה! מוּ!" / "!אָדֹם!" / "!שְׁנַיִם!"). **There are no wrong answers**: every tap does something fun. A parent prompt on screen: "שאלו: איפה הפרה?" |
| 2:40–3:20 | **"Copy me" (MV-01)** | A 30–40s song with movements (clap, arms up, stomp), parent with the child, no sensing |
| 3:20–3:45 | **The friends come home** | The friends hop into the playroom, a sticker in the album (T2 sensory, short) |
| 3:45–4:00 | **Goodnight** | The friends go to sleep under a blanket, a lullaby of 2 notes. A parent prompt: "שאלו: מה מצאת היום?" → the app sleeps |

### 5.4 Meta loop per child

| | K3 | K2 | K1 |
|---|---|---|---|
| **Long-term "map"** | Sticker album (12 pages of 6) + a playroom that fills with toy friends | Lost Friends Board + hub neighborhood that fills + collection room | Same plus Adventure Album (season track, 30 tiers, never expires) |
| **Collections** | Stickers, toy friends | Friends by species, pieces (mosaic), 1 pet | Same plus diary pages, emotes, hero parts, seasonal mosaic |
| **Currencies** (00 D5) | None (stickers only) | Coins + treasures + pet | Coins + gems + treasures + shop + season track |
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

- The world is **not mirrored in RTL** (E1b). Only the 2D HUD is RTL.
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

That is 51 mechanics; ≥25 are required. The **MVP** (first vertical slice) is: MX-M01, MX-M02, MX-H01, MX-H02, MX-H03, MX-E01, MX-B01, MX-B02, MX-P01, MX-S02, MX-R01.

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
- **Failure:** the goggles "lend" the nikud for 2 seconds (a hint) without lowering the track. Only the engine moves the track.

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
  - Optional at the start of an episode (default every other day, can be turned off).
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
   - **Never jumping with a device in the hand.**
   - One active player in the camera zone at a time.
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
- **Combo (K1 only, opt-in):** 3 in a row = a "light wave" that peels 1 layer from all the shadowlings in the area. It only shows after accuracy mastery (D11).
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
- **Defeating it:** the boss shrinks, laughs, and gives back a "big lost friend" (VIP) plus a gold-frame treasure (§16.2).
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
- **If they didn't make it:** the boss "falls asleep" at the edge of the camp with its meter frozen, and carries over to next week. No loss, and no "the boss beat you".
- **A parent can take part:**
  - a recorded clue ("רמז מאבא") that opens the finale, plus 1 tap;
  - or an "Abba/Ima" role in the finale (an extra player in same-device mode, giving light to the weakest one, `assist`).
- **Reward:**
  - A family treasure (prism frame).
  - +1 family fire level (if the weekly goal was also met).
  - K1: 2 gems.
  - Everyone: the boss joins as a VIP for that week (a cute version wanders the camp).

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
  - **3 neighborhoods**, one per child (K1 on the right, K2 in the middle-left, K3 = an entrance to the playroom). The neighborhood's color is the child's chosen color.
  - Gates to regions around the edges.
  - Buildings that unlock.
- **The rescued friends live in the neighborhood of the child who rescued them:**
  - They walk around, play, cheer when the child walks by, and sometimes "help" at a building (an animation).
  - A friend you poke says its item (light retrieval practice).
  - Above 60 friends, **instancing** keeps the scene within budget (E1b: ≤80 draw calls); only 30 friends are "awake" at a time and the rest are "inside the houses".
- **Start empty and dim** (Kirby's ruins): only the campfire and the tents. Every return makes the camp visibly fuller and brighter (P-3).

### 10.2 Neighborhood building thresholds (per child, fixed and visible, Astro model)

| Rescued friends (K2) | K1 | Building that opens | What it adds |
|---|---|---|---|
| 5 | 8 | **בית החברים** (Friends' House) | The friends have a home + the collection room (10.3) |
| 15 | 25 | **צריף המוזיקה** | Letter/number songs to listen to (a calm menu, no rewards) |
| 30 | 50 | **אוהל התחפושות** | The avatar and פנסון wardrobe (cosmetics) |
| 50 | 80 | **גן הגוזלים** | The pets' home, the egg nest |
| 75 | 120 | **תיאטרון הסיפורים** | A bedtime story (§15.5) + K1's diary pages |
| 100 | 160 | **מגדל הכוכבים** | A "Then vs. Now" view (the journey over the months) |
| 150 | 240 | **סדנת היוצרים** (K1: design mode) | K1 designs a small room/level from chunks (Roblox-like expression) |

- **Visible milestones:** a sign with a silhouette of the next building and "עוד 4 חברים" (goal-gradient). No prices, and nothing for sale.
- **K3:** every 5 toy friends → a new toy in the playroom.

### 10.3 The collection room ("חדר האוצרות")

- A 3D shelf/wall per species, with **a silhouette of every friend not yet found** (from the curriculum map, Pokédex style).
- **Endowed progress:** the room opens with 3 friends already present (from onboarding).
- **Goal-gradient:**
  - Near completion of a species, a soft glow appears on the last silhouettes plus "עוד 3!".
  - The engine also makes the last friends easier to find: they appear more often in levels.
- **Poke a friend** → it says its item. **Poke a silhouette** → ניצי gives a hint about where it is ("הוא מחכה במכרה").
- **A completed set of a species** = a T3 celebration + a badge **named after the skill** ("את מכירה את כל אותיות הקבוצה הראשונה", 00 D4) + an evolution for the tied pet.
- **Pieces (mosaic):** each region has a mosaic of 30–40 pieces that slowly reveals a picture of the region's story. A completed mosaic = a diary page + a decoration for the campfire.

### 10.4 The family campfire (shared sink)

- **Fire level 1–10 per season. It goes up only.** One level per:
  - (a) a weekly family goal met (00 C5);
  - (b) a weekly boss defeated;
  - (c) a seasonal event completed.
- **Each level:** a bigger, more colorful fire (99 Nights' 6 levels → our 10), a wider lit zone in the evening, and it **unlocks the next region / Lost Friends quest** (R12).
- **Campfire decorations:**
  - Benches, lights, a family flag, a hanukkiah at Hanukkah, a sukkah at Sukkot.
  - Bought with gems (K1) or **coins from the "give jar"** of K2/K1 (the "spend/save/give" model; a contribution to the family goal, without comparing who gave how much).
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

### 11.2 Default numbers (tunable in `economy.config.ts`)

> **Precedence:** `02-economy-progression.md` is the canonical owner of currency amounts, caps, chest tiers (wood/silver/gold/legendary), chest triggers, the capsule machine and XP. Where the numbers in this section differ from 02, **02 wins**. They are kept here only as the game-design intent: payment for mastery and not time; sparks are not a currency; nothing is announced in advance. A reconciliation pass should align the tables.


| Event | K2 coins | K1 coins | K1 gems | Notes |
|---|---|---|---|---|
| New item correct on the first try (first time) | 2 | 2 | — | Diminishing returns on repeats: 0 after the item is mastered |
| Passing a delayed review (SRS rung) | 3 | 3 | — | The receipt says "you remembered after 7 days" |
| Recovering after a mistake (correct after an error on the same item) | 2 | 2 | — | Rewards persistence (C4) |
| Completing a level (one-time) | 10 | 15 | — | Replays: 3 |
| Skill mastered (gap D9) | 20 | 25 | 1 | + a badge named after the skill |
| Region boss | 25 | 30 | 2 | + a gold-frame treasure |
| Weekly family boss | 15 | 20 | 2 | + a family treasure |
| Weekly family goal | — | — | 3 | + a fire level |
| Seasonal event completed | 20 | 30 | 3 | + a prism treasure |
| **Daily cap** | **60** | **100** | — | Above the cap: a "receipt" with 0, and nothing is said about it |

- **No pre-announcement** of prizes for tasks (Lepper). The receipt appears *after* the action in the camp.
- **Shop (cosmetic only, fixed catalog; items never leave once they've appeared):**
  - an avatar shirt 30–60;
  - a פנסון part 40–120;
  - a neighborhood decoration 20–150;
  - an emote 80;
  - a trail 100;
  - a "special" item for gems (K1) 3–8 gems.
  - A wishlist plus a "save-up jar" with a progress bar.
- **Spend/save/give jars** (K2: two, K1: three).
  - The "save" jar gets 5% weekly interest (up to 10 coins).
  - The "give" jar goes to the family campfire.
- **K3:** stickers only (00 D5). No coin counter.

---

## 12. Sibling co-op

### 12.1 Co-op modes

| ID | Mode | Devices | Who | Length | Content |
|---|---|---|---|---|---|
| CO-1 | **"שני צדי השולחן"** (Face-to-face) | 1 iPad landscape, split into 2 halves, each half **rotated toward its player** (180°) | K1+K2, K2+K3 (with a parent), K1+K3 | 5–8 min | Campfire evening, boss finale, building together |
| CO-2 | **"כתף אל כתף"** (Side-by-side) | 1 iPad, one shared screen, roles with separate zones (right/left) | Any pair or three, + a parent | 5–8 min | The runner (one runs, one lights), the evening, a level in co-op |
| CO-3 | **"מכשירים מחוברים"** (Multi-device) | 2–3 devices on the same family account, a session over WebSocket (see Proposed change to 00) | K1+K2(+K3 on a parent's phone/iPad) | 5–10 min | The weekly boss, the evening, Mission Control (K1 commander → K2 artist-bot) |
| CO-4 | **"מכתב לאח"** (Async) | Any | K1 → K2 | — | K1 builds a riddle or puzzle for her brother (Walkington problem-posing); K2 solves it in his episode. A heart emote goes back |
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

### 12.6 K1 guide mode ("מצב מדריכה")

- **Opt-in per session.** K1 is offered (never required) "do you want to be your brother's guide today?"
- **Cap:** 1 guide session per day, ≤10 minutes (gap §4.12).
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
| S0 | **"המחנה מתעורר"** | from launch (target 2026-11-08) → 2026-12-03 | Onboarding, R-03 + R-05 + R-12 | Weeks 1–3 of the arc (§2.4) |
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
- **No "event coins"** and no special currency. Rewards come from the regular tables (§11.2).
- **"Event treasure"** (prism frame) after completing ≥60% of the event levels. Its contents are known in advance.

---

## 14. Onboarding and first week

### 14.1 Setting up the family (parent, behind the gate, ~10 minutes, before the children)

1. Names, grammatical gender, birth year.
2. Building avatars (§3.3).
3. Recording the name + 3 lines (opening, goodnight, milestone).
4. Shabbat mode and observance level (§15.4).
5. Daily caps.
6. Device placement for movement (optional).
7. A "letter of the week from gan" / "this month in class" (optional).

### 14.2 The first 60 seconds per child

**K2** (the reference flow; voice only, no text):

| s | What happens |
|---|---|
| 0–8 | Darkness of dusk → one firefly (ניצי) lights up. "!אוֹר בּוֹקֶר, [name in the parent's voice]" |
| 8–20 | A 12s movie: the Muddler's sneeze (funny), the friends fly away. ניצי: "עזור לי להחזיר אותם?" |
| 20–35 | The **פנסון lab**: 3 big pulsing parts (chassis/color/antenna). Each tap changes it immediately, plus a laugh. A big "!יוצאים" button appears after 2 choices |
| 35–50 | The פנסון wakes up in the camp. **The first tap on the ground → it rolls there** (teaching tap-to-move by doing, no tutorial). Glints lead to a stone |
| 50–60 | A rune on the stone hums "/m/". ניצי: "איזו אבן אומרת 'מָ'?" Two stones. A correct tap → the first friend (a letter-friend) comes out, T1 + "!מָצָאתָ אוֹתִי". (If wrong: the stone says its sound, and the right one glints. Nothing to fail) |

**K1** (with text + voice):

| s | What happens |
|---|---|
| 0–10 | Same opening, plus a short title in the HUD "מחנה הגחליליות". "Skip" appears after 3s (she doesn't like "babyish") |
| 10–25 | The פנסון lab: Ranger chassis, 6 parts + **giving it a name** (a keyboard or 6 suggestions) |
| 25–40 | Tap-to-move + the joystick shown as an option ("רוצה ג'ויסטיק?") |
| 40–60 | A first lock: "החבר תקוע מאחורי שער. כדי לפתוח: 6 × 4 קורות." The array hammer (MX-M05) at an easy level (placement in the background) → the first friend + a short diary page |

**K3** (lap mode, with a parent):

| s | What happens |
|---|---|
| 0–10 | A soft playroom, פופי peeks out: "!קוּקוּ, [name]" |
| 10–30 | "!גַּע בִּי" → every touch makes פופי laugh and bounce (cause and effect) |
| 30–50 | Two bushes are shaking. Every tap reveals an animal ("!פָּרָה! מוּ!") |
| 50–60 | The animal hops into the playroom. A first sticker (T2 sensory). Parent prompt: "שאלו: מי זה?" |

- **There is no explicit "placement test".** The first week includes **play-based placement**: the items are chosen with an adaptive staircase within regular levels (see the placement spec / `placement_assessment.md`).

### 14.3 First week (day by day)

| Day | K3 | K2 | K1 | Family |
|---|---|---|---|---|
| Sun | The playroom (3 toys) + peekaboo | Onboarding + LV R-03 #1 (rune lens) + the first evening | Onboarding + LV R-05 #1 (array hammer) + diary | — |
| Mon | + a 4th toy, "copy me" | R-05 #1 (number jet) + the collection room opens (3 friends from onboarding, endowed) | R-03 #1 + the collection room | — |
| Tue | Peekaboo with colors | R-03 #2 + first movement moment (MV-02 clapping syllables) | R-04 taste (the fog) + the first Build Mission | — |
| Wed | Rest / playroom | Lantern breath (MX-S02) in a level + first pet egg (known in advance) | First evening defense with towers + egg | — |
| Thu | Family: "lighter" role | **First family evening: the "ענן ההפתעות" boss** (CO-2, short, 5 min, guaranteed win) | Same + optional guide mode | Fire level 1→2, the first family treasure |
| Fri | Optional (before Shabbat) | Short episode: daily surprise + Build | Same | Recorded "Shabbat shalom" from a parent (optional) |
| Sat | Shabbat (§15.4) | Shabbat | Shabbat | Nothing counts |

**Goal of week 1:**
- Each child has rescued ≥8 friends (K3: 6 toy friends).
- The family has seen its fire grow.
- Each child has had ≥1 "show Abba/Ima" moment.

---

## 15. Stopping, Shabbat, holidays, toddler

### 15.1 Designed to end (00 C3, C5, `risks` §4)

- **A visible progress strip** in the HUD: "היום במחנה": 4–5 icons (morning, level, camp, evening, goodnight) that fill in as you go.
- **A soft warning:** when 2:00 remain of the cap or the episode, ניצי says "עוד הרפתקה אחת קטנה ואז הולכים לישון".
- **Never cut in the middle of a beat:**
  - If the cap ends in the middle of a level, the game finishes the current beat (≤1:30 overtime) and then **"lands"** in Ketsu: a shortcut to the gate and the last friend.
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
  - The app shows only "the camp is asleep" (a slowly animated drawing) plus the parent-gate button.
  - **No "play again" button** for the child. Re-entry for K2/K3 is through the parent gate. K1 can open the "חדר האוצרות" (view only, no rewards) and the music hut.
- **Caps (parent-set, defaults):**
  - K3: 2 episodes / 10 minutes per day.
  - K2: 2 / 25 minutes.
  - K1: 2 / 40 minutes.
- **Bedtime lock:** from a parent-set hour (default 19:30 for K3/K2 and 20:30 for K1). The app shows "the camp is asleep".
- **Media-free contexts:**
  - The parent marks meals or travel in a schedule; the app "sleeps".
  - Car: an **audio only** mode (songs, riddles, no screen). This is an option for the parent, not a default.
- **Offline mission at the end** (every episode, by age and by the content learned):
  - "מצאו 3 דברים בבית שמתחילים ב-מ"
  - "ספרו כמה מדרגות יש בבניין"
  - "Sunny asks: find something *blue*"
  - "תכנני את הפנסון שלך על דף" (printable)
  - Done/not done is marked by the parent or not marked at all; nothing is lost.
- **Exit in the middle:** a "back to camp" button (in the pause menu) at any time, with no cost. A level that was left is saved by checkpoint.

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

- **Detection:** the time of candle lighting and havdalah is computed by an algorithm from sunset (a NOAA solar algorithm) for the city chosen by the parent (default: Tel Aviv, lighting 20 min before sunset; Jerusalem 40 min; havdalah = the appearance of 3 stars ≈ sunset + 40 min, a configurable parameter).
- **Holidays** from the calendar config (§13). Yom Tov days get the same treatment as Shabbat.
- **Modes (family choice, an open item for the parent):**

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
| **T2** medium | Level complete, a new item/sticker, treasure, a building rises | **≤3.0s** (plus the taps on the treasure, which the child controls) | Yes, skippable after 1.0s | Reveal (16.2), the friends dance, the item rotates into the album | Sensory-only version 1.5s | Yes | Yes |
| **T3** large | Skill mastered, a species set complete, pet evolution, region boss | **≤6.0s**, rare | Yes, skippable after 1.0s | Confetti (not flashing), ניצי dance, a fanfare of 1.5–3s, sometimes a recorded parent line, a "תראו לאבא ולאמא" moment | Doesn't exist (instead: T2) | Yes | Yes |
| **T4** finale | Weekly family boss, season finale | **≤10s**, at most 1 per week | Yes, skippable after 2.0s | A short cinematic of the friends + the boss turns into a friend + the fire grows | Shared participation | Yes | Yes |

- **Time budget:** celebrations + transitions **≤15% of the episode's time** (gap §4.5). Measured in telemetry (`celebration_ms_total / episode_ms`). If the budget is exceeded, T0 shortens to 0.25s and T1 merges.
- **Screen shake:** K3 0, K2 ≤4px, K1 ≤8px. **Flashing:** never >3 Hz (photosensitivity). Reduce Motion (OS) → no shake, no confetti, fades only.
- **Streak notes:** each consecutive correct answer plays the next note on the C-major pentatonic scale (up to 8). A broken chain resets quietly with no sound (R `hit_games` §4).
- **Error sound:** a soft low "בּוֹפּ" of 150ms, **never a buzzer** and never a red X for K2/K3.

### 16.2 Treasure reveal choreography (deterministic, 00 D2, R3)

> **Precedence:** the chest tiers, the taps per tier and the per-tap timings in `02-economy-progression.md` (§ chest reveal) are canonical. This section keeps two game-design requirements that 02 must satisfy (or, if it doesn't, record a decision in `docs/decisions/`): (a) the contents are fixed and previewed before opening; (b) the total reveal time is ≤3.0s excluding the child's taps (T2 budget). The "effort marks" idea below is a *proposal* for how extra taps can mean something. Adopt it only if 02 agrees.


- **The frame type is known when it's earned and shown in advance:**
  - `bronze` = a level/episode
  - `silver` = a skill mastered
  - `gold` = a region boss
  - `prism` = an event or the family boss
  - The frame differs in **shape** as well as color (color-blind safe): round / hexagon / star / prism.
- **The contents are shown as a silhouette before opening.**
- **"Effort steps":** the number of taps to open = 1 + the number of **effort marks** earned in the activity (max 3):
  - `firstTry` (≥80% correct on the first try)
  - `hintFree` (no hints in the last beat)
  - `braveChoice` (chose Stretch / 🌶🌶)
  - (K1) `mentor` (helped a sibling in guide mode)
  - Each step shows the badge of the mark ("ניסית שוב!"). **This is a summary of real effort, not a lottery.**
- **Timings per step (tokens):**
  ```
  reveal.tap.wobbleMs = 280      (the wobble grows with each step: amplitude ×1.0, ×1.4, ×1.8)
  reveal.tap.glowStepMs = 180    (+ one rising note; mark badge flies in)
  reveal.burstMs = 400           (light burst, 24 particles, hit-stop 60ms)
  reveal.itemRiseSpinMs = 700    (the item rises and spins 1.25 turns, ease-out-back)
  reveal.settleMs = 300          (lands in its slot + click + haptic success)
  reveal.voMs ≤ 1200             (the item's name / ניצי, in parallel to rise+settle)
  total (excluding child tap latency) ≤ 3000ms
  ```
- **Verification:** a frame-step study on the device after the vertical slice (CEL-AC-6). The tokens are tuned for "feel", without exceeding the T2 budget.
- **There is no randomness of value.** "Rarity" = what earned it (00 D2). For K1 the frame shows "קיבלת זהב כי ניצחת את הבוס של המכרות".

### 16.3 Praise phrase sets (the game's voice)

The UX spec is the canonical owner of the phrase list (00 C4). These are the game's voice lines and must be merged into it. Each category has ≥8 variants per age in the content pack; examples below.

| Category | Example K2 `{m|f|pl}` | Example K1 `{m|f|pl}` |
|---|---|---|
| **Describing the action** | `{מָצָאתָ|מָצָאת|מְצָאתֶם} אֶת הַ-מָ!` | `{בָּנִיתָ|בָּנִית|בְּנִיתֶם} קִיר שֶׁל 6 שׁוּרוֹת וְ-4 טוּרִים.` |
| **Strategy** | `{סָפַרְתָּ|סָפַרְתְּ|סְפַרְתֶּם} הָלְאָה מֵחָמֵשׁ!` | `{פֵּרַקְתָּ|פֵּרַקְתְּ|פֵּרַקְתֶּם} אֶת 7×8 לִשְׁנֵי חֲלָקִים.` |
| **Persistence** | `{נִסִּיתָ|נִסִּית|נִסִּיתֶם} שׁוּב, וְזֶה עָבַד!` | `{הִמְשַׁכְתָּ|הִמְשַׁכְתְּ|הִמְשַׁכְתֶּם} גַּם כְּשֶׁהָיָה קָשֶׁה.` |
| **Improvement** | `הַיּוֹם זֶה הָלַךְ יוֹתֵר מַהֵר!` | `{זָכַרְתָּ|זָכַרְתְּ|זְכַרְתֶּם} אֶת זֶה גַּם אַחֲרֵי שָׁבוּעַ.` |
| **Brave choice** | `{בָּחַרְתָּ|בָּחַרְתְּ|בְּחַרְתֶּם} בַּשְּׁבִיל הַמְּאַתְגֵּר, אַמִּיץ!`* | `{בָּחַרְתָּ|בָּחַרְתְּ|בְּחַרְתֶּם} לְהִתְמוֹדֵד עִם הָאֶתְגָּר.` |
| **Checking/verifying** | `{בָּדַקְתָּ וְצָדַקְתָּ|בָּדַקְתְּ וְצָדַקְתְּ|בְּדַקְתֶּם וּצְדַקְתֶּם}` | `{שָׁאַלְתָּ|שָׁאַלְתְּ|שְׁאַלְתֶּם} אֶת גַּלְגַּלּוֹן וְאָז {בָּדַקְתָּ|בָּדַקְתְּ|בְּדַקְתֶּם} לְבַד.` |
| **Helping (co-op)** | `{עָזַרְתָּ|עָזַרְתְּ|עֲזַרְתֶּם} לַמְּדוּרָה לִגְדֹּל.` | `{חִכִּיתָ|חִכִּית|חִכִּיתֶם} שֶׁהוּא יְנַסֶּה קֹדֶם. זֶה עוֹזֵר לוֹ לִלְמֹד.` |
| **Neutral escalation in a chain** (no gender) | "!שָׁלוֹשׁ בְּרֶצֶף" · "!הַשְּׁבִיל נִבְנֶה" · "!עוֹד חָבֵר" | "!חֲמִשָּׁה בְּרֶצֶף" · "!הַמַּסְלוּל נִפְתַּח" |
| **After a mistake** | `עוֹד לֹא. {בּוֹא|בּוֹאִי|בּוֹאוּ} נִסְתַּכֵּל שׁוּב.` | `עוֹד לֹא. {רוֹצֶה|רוֹצָה|רוֹצִים} רֶמֶז אוֹ עוֹד נִסָּיוֹן?` |

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

- **AC-WORLD-1:** The hub is a walkable scene (R3F or the Skia fallback) with a campfire, 3 neighborhoods and ≥3 gates. Tap-to-move reaches any walkable point within ≤3s of walking.
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
- **AC-MX-3:** English: every English line plays only inside the English frame (`englishContext=true`) with an entry/exit jingle. Phonics sounds come from the recordings list only.
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
- **AC-EVE-1:** Evening: shadowlings don't approach the camera (path z ≥ fire.z − 2 tiles). In K3 only `sleepy` appears.
- **AC-RUN-1:** The runner always ends at a destination within ≤90s (K2) / ≤120s (K1). There is no death state or revive (a test).

### Co-op

- **AC-CO-1:** In CO-1 the two halves of the screen are rotated toward their player, and each half has a separate RTL HUD (screenshot).
- **AC-CO-2:** Fairness: in telemetry, each child's share of light is within ±25% of an even share in ≥80% of co-op sessions. If not, a closed-loop correction runs.
- **AC-CO-3:** Drop-in/out: a player who leaves doesn't stop the session (≤1s of disruption). A player who joins gets a role within ≤3s.
- **AC-CO-4:** No screen in co-op shows separate data per child to the children (screenshot review).
- **AC-CO-5:** Guide mode: K1 has no input on K2's items. There is a 5s delay before a hint token. There is a 10-minute cap and 1 per day (a test).
- **AC-CO-6:** Co-op time is counted at 50% toward each child's cap (a test).

### Economy and celebrations

- **AC-ECO-1:** No coins are paid per raw time or per spark. Every coin event is tied to an event from §11.2 (a test of the payment table).
- **AC-ECO-2:** There is no pre-announcement of a reward for a task (lint on strings: "תקבל/י … אם").
- **AC-ECO-3:** Treasure: the contents and frame are known before opening (silhouette). The number of taps = 1 + effortMarks (a test). There is no RNG in the reward path (a lint rule bans `Math.random` in `economy/`).
- **AC-CEL-1:** T0 ≤800ms, T1 ≤1500ms, T2 ≤3000ms, T3 ≤6000ms, T4 ≤10000ms (a timer test on each animation sequence).
- **AC-CEL-2:** Celebrations + transitions ≤15% of the episode time on average over a week (telemetry).
- **AC-CEL-3:** No flashing >3 Hz. Shake 0 for K3. Reduce Motion is respected (an animation test).
- **AC-CEL-4:** Lint on banned phrases (§16.3) across all content and scripts: 0 hits. Gender parity check passes.
- **AC-CEL-5:** Every child-facing string has `m` and `f` variants (and `pl` where relevant). Playwright snapshots for each profile.
- **AC-CEL-6:** A frame-step study on the device after the vertical slice (📱). The reveal tokens are updated and documented in `docs/decisions/`.

### Stopping, Shabbat, toddler

- **AC-STOP-1:** Every episode ends with the goodnight ritual (30–45s) plus an offline mission card. After it there is no "play again" button for K2/K3 (screenshot).
- **AC-STOP-2:** The cap never cuts off mid-beat. Overtime is ≤1:30 at most, or the state is saved for tomorrow (a test).
- **AC-STOP-3:** No exit line from §15.2's banned list appears (lint).
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
| DD-06 | Weekly boss day | Thursday, 17:00–19:00 is the default "opening hour" for the finale (the finale is available from Thursday until Friday at noon) |
| DD-07 | Is Saturday night a new week? | Yes. The week runs Sunday → Saturday night |
| DD-08 | Two children want the same device | The turn ring decides by last use. If both want to, a CO-2 suggestion is shown |
| DD-09 | A child who wants to keep going after the cap | No. The goodnight ritual, then "the camp is asleep". The parent can grant +10 min through the gate (a "bonus" button, not shown to the child) |
| DD-10 | Number of choices in Plan | K2: 2, K1: 3 + a flame, K3: 3 doors |
| DD-11 | Default letter order (K2) | The default in the Hebrew spec (sound-first, high utility, confusables separated); the parent's "letter of the week" overrides |
| DD-12 | Which numbers K3 hears | 1–3 only (Give-N), in the lookup forms (feminine for rote counting; noun agreement as in the lexicon) |
| DD-13 | Level difficulty in co-op | Each child at their own band; co-op doesn't change the band |
| DD-14 | Content that a parent hasn't approved (generated) | Doesn't reach the child. Fallback to the static pack |
| DD-15 | Offline (no network) | Everything works. Co-op is limited to same-device. Sync later (00 E6) |
| DD-16 | An event that came out before the content was ready | It is postponed by days in the config without changing its dates on the calendar (it appears in the box as a "late gift") |
| DD-17 | A secret not found after 3 visits | On the 4th visit the pet points to it right away (goal-gradient) |
| DD-18 | Moving up a grade (Sept 2027) | Profile: K2 → grade 1 (moves to the "Explorer+" setting, more text with nikud); K1 → grade 4; K3 → gan (3.5) moves from lap mode to "co-play-first" with short solo periods. A light placement test runs again |
| DD-19 | Volume and music | Music −8 dB under voice-over (ducking). Separate sliders for music/SFX/VO for the parent |
| DD-20 | iPhone orientation | Game scenes in landscape. The hub on iPhone is landscape too (a 3D scene); menus/collection room portrait-agnostic |
| DD-21 | What the parent sees from co-op | Only "who took part" + positive behaviors (from guide mode) |
| DD-22 | A child's drawing failed in processing | It becomes a "sticker" with an artistic filter. Never "unsuitable" |
| DD-23 | Two siblings rescued the same friend in co-op | Each one gets "their" copy in their neighborhood (no splitting, no fights) |
| DD-24 | Content that is too easy for K1 (boredom) | Mystery/Stretch flame offered automatically after 3 levels with ≥95% |
| DD-25 | Daily surprise when the child skipped days | They accumulate up to 7, and then "a big letter" that bundles them |

---

## 19. Open items for the parent (minimal; nothing here blocks: placeholders + defaults are in use)

1. **The children's names** and **K3's grammatical gender** (00 A2). Default: DD-05.
2. **Shabbat mode:** `soft` / `full` / `off` (§15.4) + the city for computing times. Default: `soft`, Tel Aviv.
3. **Recording parent voices:** the child's name ×3 children + 3 short lines per parent (≈10 minutes, a guided in-app recording). Default: no recording → TTS is not used for names; ניצי says "!אור בוקר" without a name.
4. **Taste approval (E10):** the world name, the hero (פנסון), the cast (a sheet of 8 characters). The first defaults are used until approval.
5. **Grandparents** in the game? (Yes/no + names.) Default: no.
6. **The family boss day:** default Thursday.

---

## 20. Proposed change to 00

> The decisions in 00 remain binding until updated. Each proposal includes the reason.

1. **E5 (Backend) — real-time co-op:** add **Cloudflare Durable Objects** (WebSocket hibernation; part of the Workers Paid plan) as the transport for CO-3 sessions. Each "family room" is one DO: authoritative event ordering, low latency, and an automatic close after 15 minutes without activity.
   - **Reason:** D1/HTTP sync (E6) is enough for async but not for evenings or the boss on 2–3 devices.
   - **Fallback without approval:** CO-3 is disabled and co-op is same-device only (CO-1/CO-2).
2. **B7 — explicit list of quiet days:** Yom HaShoah, Yom HaZikaron, Tisha B'Av, Yom Kippur = no events, no boss, celebrations up to T1. **Yom Yerushalayim** = a neutral day with no event (political sensitivity).
   - **Reason:** "memorial days" in 00 isn't specific enough for a calendar config.
3. **B8 — add:** no fireworks (including Yom HaAtzmaut); no darkness >40% of the screen and a minimum luminance in evening scenes; for K3, dusk only.
   - **Reason:** R8 (age of onset of fear of the dark), and fireworks are explosions.
4. **D1 — clarification:** "sparks" (`spark`) in levels are a **non-persistent in-episode resource** that doesn't convert to coins. They are not a third currency.
   - **Reason:** they serve as breadcrumbs (G1) without breaking the "mastery, not time" rule.
5. **D2 — clarification of "upgrade steps" in the reveal:** a multi-tap with visible "upgrades" is allowed **only** when each step represents an effort mark that has already been earned (§16.2).
   - **Reason:** to keep the beloved Starr Drop/Lucky Chest ritual without randomness (R3).
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
| `free_choice_unrewarded` | activity (music hut, collection room, playroom) | The overjustification metric (report §"how we know we won") |
| `stop_ritual` | completed, overtimeMs, capReason | AC-STOP-* |

**Product metrics (for the parent, not for the children):**
- "learning seconds per minute of play" (target ≥0.6 for K1/K2);
- delayed review after a week;
- free choice of activities with no rewards;
- persistence after a mistake;
- number of creations.
