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
  - "בָּדַקְתָּ{|ְ|ֶם} אוֹתִי? מְצֻיָּן. מְכוֹנוֹת טוֹעוֹת."
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
  - "אֵיךְ הַגּוּף שֶׁלְּךָ{|ְ|ֶם} מַרְגִּישׁ עַכְשָׁו?"
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
| `sparkling.wonder` | **פְּלִיאוֹנִים** | A science or world fact | A spiral shell | "הַאִם יָדַעְתָּ{|ְ|ֶם}…?" |
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
