# UI/UX Polish Craft for a World-Class Kids' Learning Game (iPhone/iPad, Hebrew RTL + English, ages 2.5 / 5.5 / 8.5)

> Research note on method and limits (Nov 2026): the fetch tool could not reach blog.duolingo.com, design.duolingo.com, rive.app or nngroup.com from this session; those hosts would need to be allowed in the session's network settings. Claims from those sources are therefore taken from search-result excerpts, not full-page reads, and are marked "(snippet)". Apple HIG pages were read in full from developer.apple.com (.md versions). Wherever a claim comes from my own craft judgment rather than a source, it sits under **Inferences**, not Cited Findings.

---

## 1. Visual design language of best-in-class kids products (Duolingo, Pok Pok, Toca Boca, Khan Academy Kids, Supercell, Nintendo)

### Takeaway
The best kids' products run a small, strict visual grammar: a few repeated geometric primitives, rounded everything, flat color with a single shadow convention, and characters as the main interface. Duolingo writes this down publicly (shape language, character rules, pill shadows). Toca Boca and Pok Pok add a "toy, not game" philosophy: no text, no instructions, calm open-ended play. Supercell's Brawl Stars is the reference for "readable at a glance" and touch-first controls. Public, primary documentation of Supercell's *menu/button* styling is thin.

### Cited Findings
**Duolingo illustration system**
- Duolingo publishes an illustration guide with dedicated "shape language" and "characters" pages at design.duolingo.com (snippet). — [Duolingo Design: Illustration](https://design.duolingo.com/illustration); [Characters](https://design.duolingo.com/illustration/characters)
- Shadow rule: shadows always sit below characters and objects as a **pill shape, never an oval, "because ovals imply perspective"**. This keeps the world flat and graphic (snippet). — [design.duolingo.com/illustration](https://design.duolingo.com/illustration)
- Character rule: "One way of keeping a character's shape language cohesive is to use **shape repetition** throughout their design. While this isn't always required, it's a good starting point" (snippet). — [design.duolingo.com/illustration/characters](https://design.duolingo.com/illustration/characters)
- Mouths: "Mouths are expressive and the **least geometric shape** in our style. **Most mouths favor one side.** This asymmetry is more lifelike and brings more emotion to the face" (snippet). — [design.duolingo.com/illustration/characters](https://design.duolingo.com/illustration/characters)
- Poses must fit the style, not just be anatomically right. In the guide's gorilla example, "both of these poses are very gorilla-like, but only one of them works within our style" (snippet). — [design.duolingo.com/illustration/characters](https://design.duolingo.com/illustration/characters)
- Production: Duolingo "moved to a minimalistic art style that was fast to produce without sacrificing design quality and clarity" and switched to vector illustration software to speed the pipeline (snippet). — [Duolingo blog: Shape language](https://blog.duolingo.com/shape-language-duolingos-art-style/)
- Compositional eye-direction: "To draw attention to UI or text on the screen, we **rotate lines and angles in the artwork to direct the eye**" (snippet). — [Duolingo blog: Shape language](https://blog.duolingo.com/shape-language-duolingos-art-style/)
- Secondary analysis says the style rests on three primitives: **rounded rectangle, circle, rounded triangle**. Art director Greg Hartman is quoted with a music analogy: "make each shape matter." These come from secondary write-ups, not Duolingo itself. — [Dawn Creative](https://www.dawncreative.co.uk/?p=11033); [madegooddesigns](https://madegooddesigns.com/duolingo-brand-guidelines/)
- Johnson Banks (2019 rebrand) wrote the guidelines because "the owl mascot (Duo), the core colour, and their illustration style were broadly working. But they lacked clear guidelines on how to use the brand 'away' from the app." — [Creative Review](https://www.creativereview.co.uk/duolingo-rebrand-johnson-banks/); [Johnson Banks](https://www.johnsonbanks.co.uk/templates/work/duolingo)

**Pok Pok (Apple Design Award 2021, "Delight and Fun")**
- Pok Pok Playroom won the 2021 ADA in "Delight and Fun" three weeks after launch. — [MacStories ADA interviews](https://www.macstories.net/linked/2021-apple-design-award-winners-the-appstories-interviews/); [Sketch blog: Pok Pok](https://sketch.com/blog/pok-pok)
- The team chose no levels or points because that "is usually what makes screen time so addictive". Kids "approach every single toy with a sense of freedom and curiosity". It is described as "handcrafted toys" for ages 2–6 with "soothing sounds to create a calm, immersive environment". — [Mom's Choice Awards](https://www.momschoiceawards.com/?p=25849)
- They tested "every iteration of every activity" with kids from the start, and toddlers were candid critics. — [Mom's Choice Awards](https://www.momschoiceawards.com/?p=25849); [Tank Talks with Melissa Cash](https://tanktalks.substack.com/p/building-an-award-winning-app-through)
- CEO Melissa Cash: "Kids under five learn best through play." — [BetaKit](https://betakit.com/?p=332907)

**Toca Boca**
- CEO Björn Jeffery: heavy text works poorly for young users. The studio deliberately leaves space for kids to fill in, and this also removes most localization cost. — [GamesBeat](https://gamesbeat.com/how-toca-boca-got-100m-mobile-downloads-by-putting-kids-first/)
- Designer Mellbratt: "a good toy doesn't need instructions," so there are no verbal tutorials. — [Kill Screen](https://www.killscreen.com/what-designers-can-learn-about-play-children/)
- "Toys, not games." Gender-neutral by design. Not built around competing and winning. — [PocketGamer](https://www.pocketgamer.com/articles/052067/r/); [Tech.eu](https://tech.eu/features/1368/toca-boca-kids-apps-profile-video-interview/); [Kidscreen](https://kidscreen.com/?p=59849)
- "Characters as an interface" is an established pattern in Toca Boca and kids' apps generally. — [MacStories: Characters As An Interface](https://www.macstories.net/linked/characters-as-an-interface/)

**Khan Academy Kids (Duck Duck Moose lineage)**
- Five original animal characters "encourage children to think, play, learn, and persist". Later, **50 human characters "inspired by real kids and families"** were added for stories about friendship, confidence and patience. — [Khan Kids Help: characters](https://khankids.zendesk.com/hc/en-us/articles/360049358751-Learn-more-about-the-characters-inside-Khan-Academy-Kids)
- Process: first watch how children play, then design lessons and characters with early-learning specialists, artists, educators, parents and kids. — [Hoot Reading interview](https://learn.hootreading.com/blog/app-spotlight-khan-academy-kids)
- Duck Duck Moose joined Khan Academy in 2016. Founders were Caroline Hu Flexer (ex-IDEO), Michael Flexer and designer Nicci Gabriel. — [TechCrunch](https://techcrunch.com/2016/08/26/kids-app-maker-duck-duck-moose-joins-khan-academy/)

**Supercell / Brawl Stars**
- GDC 2019 (Antti Summala): Brawl Stars' controls were "much more demanding" than earlier Supercell games. Heavy prototyping went into making them simple and touch-native. A second talk covered the many control and progression schemes they tried. — [GDC news](https://gdconf.com/news/go-inside-design-supercells-new-hit-game-brawl-stars-gdc-2019); [Game Developer video](https://gamedeveloper.com/design/video-designing-the-control-and-progression-schemes-of-i-brawl-stars-i-); [GDC Vault](https://gdcvault.com/browse/gdc-19/play/1025751)
- Supercell's stated UI priorities in job postings: improve "quality, consistency, readability, and polish" (Clash Royale). Brawl Stars' team vision is "the most accessible team game for mobile". — [Supercell careers: Senior UI artist](https://supercell.com/en/careers/senior-ui-artist/66476c5d-30b6-4772-aa41-94d251e3d73c/); [Senior UI/UX Designer, Brawl Stars](https://supercell.com/en/careers/senior-uiux-designer-brawl-stars/2251345)
- Supercell's "readability on the subway" principle: a game should read at a glance on a small phone in a distracting setting. — [Gigazine](https://gigazine.net/gsc_news/en/20180501-game-design-subway-legibility)
- Third-party reviews describe Brawl Stars UI as using strong contrast, oversized icons and simplified layouts. These are secondary sources, not Supercell. — [Pixune review](https://pixune.com/blog/best-examples-mobile-game-ui-design/); [Pratt IxD critique](https://ixd.prattsi.org/2025/02/design-critique-brawl-stars/)

### Inferences
- **Recommended design-system skeleton.** These are my synthesis, not cited specs:
  - **3 primitives** (circle, rounded rect, rounded triangle), as in Duolingo.
  - **1 shadow convention**: pill shadow under characters, plus a darker "lip" under buttons.
  - **1 outline rule**: either no outlines (Duolingo/Pok Pok calm) or a thick dark outline (Supercell punch).
  - **4–6 saturated brand hues**, each with a darker "shade" step for 3D lips and a lighter "highlight" step.
- **Two registers for two audiences.** The 8.5-year-old Brawl Stars fan will read "Supercell chunky" as cool, while the 2.5-year-old benefits from Pok Pok calm. Use one illustration grammar with two "energy" skins:
  - The battle/hero hub: thicker outlines, bevels, gold and purple rarity colors, more particles.
  - The toddler zone: fewer elements, softer motion, no timers.
- **"Supercell chunky button" anatomy.** This comes from observation of the genre; no primary spec was found:
  - Rounded-rect face.
  - A darker bottom lip of 4–8 pt that the face "presses down" into.
  - A thick dark stroke around the whole button.
  - Inner top highlight.
  - Heavy, rounded display type with its own outline or drop shadow.
  - An icon that breaks the button's bounding box for dynamism.
  - On press, the face translates down by the lip height and the lip disappears.
- Characters should *be* the UI: a mascot points, reacts and explains. This replaces text for pre-readers (Toca/Khan Kids pattern).

### Gaps
- Could not read the full Duolingo design guide (design.duolingo.com blocked). The color palette tokens, exact corner radii and eye/proportion rules were not verified.
- No Sago Mini, Lingokids, Headspace for Kids or Nintendo UI design talks were retrieved in this pass.
- No primary Supercell/GDC source on menu/button art direction (bevels, outlines, fonts) was found. GDC Vault talks on Brawl Stars "Making of a Brawler" and Clash Royale UI may exist but were not accessed.

---

## 2. Motion design and micro-interactions (Rive, Lottie, springs, press states, celebration and reward choreography)

### Takeaway
Duolingo uses **Rive state machines** so characters respond to inputs in real time, including lip-sync driven by audio timing data, rather than playing canned clips. On iOS, SwiftUI's `spring(duration:bounce:)` (iOS 17+) is the native motion primitive. Keep bounce at or below about 0.3–0.4 for UI, and use more only for celebratory character motion. Primary-source analysis of Clash Royale/Brawl Stars reward timing was not found. The craft consensus is "the build-up is the reward": anticipation, then a staged reveal, then a peak with layered particles, sound and haptics.

### Cited Findings
**Rive and Duolingo**
- Duolingo's characters use a state machine that **blends mouth shapes (visemes)**. When a challenge appears, the app fetches the audio *and timing data*, then uses those timings to drive the animation state machine in sync with the audio. The blog notes that hand-animating every mouth movement was a major challenge (snippet). — [Duolingo blog: World character visemes](https://blog.duolingo.com/world-character-visemes)
- For the AI video-call character Lily, "the Rive State Machine drives Lily's mouth positions, facial expressions, camera movements, and all other animations" (snippet). — [Rive blog: Duolingo's Lily](https://rive.app/blog/duolingo-s-ai-powered-video-call-brings-lily-to-life)
- Agency posts claim Duolingo uses inputs such as `isSpeaking`, `mouthShape`, `isThinking` and 15–20 mouth shapes. **This is unverified** and not confirmed by Duolingo or Rive. — [DEV Community (Rive agency)](https://dev.to/uianimation/how-duolingo-uses-rive-for-their-character-animation-and-how-you-can-build-a-similar-rive-mascot-5d19)
- Rive markets itself for interactive character animation. — [Rive: character animation](https://rive.app/use-cases/character-animation)

**SwiftUI springs (WWDC23 "Animate with springs", session 10158)**
- The modern spring takes two parameters, **duration** and **bounce**. Duration roughly equals settling time, but for very bouncy springs it is the oscillation period. — [Apple docs: spring(duration:bounce:blendDuration:)](https://developer.apple.com/documentation/swiftui/animation/spring(duration:bounce:blendduration:).md); [WWDC23 10158](https://developer.apple.com/videos/play/wwdc2023/10158/)
- Presets are `.smooth`, `.snappy` and `.bouncy`. An example from the session is `withAnimation(.spring(duration: 0.6, bounce: 0.2))`. When unsure, use bounce 0. Values above about 0.4 can look exaggerated for UI. — [WWDC Notes 10158](https://wwdcnotes.com/notes/wwdc23/10158)
- SwiftUI automatically tracks gesture velocity and feeds it into springs, so a flung element continues with momentum. — [WWDC Notes 10158](https://wwdcnotes.com/documentation/wwdc23-10158-animate-with-springs/)
- The `duration:bounce:` API is iOS 17+. — [Hacking with Swift](https://www.hackingwithswift.com/quick-start/swiftui/how-to-create-a-spring-animation)

**Reduce Motion (Apple HIG Accessibility)**
- When Reduce Motion is on, reduce automatic and repetitive animations, including zooming, scaling and peripheral motion. Specifically:
  - "Tightening animation springs to reduce bounce effects."
  - "Tracking animations directly with people's gestures."
  - "Avoiding animating depth changes in z-axis layers."
  - "Replacing transitions in x-, y-, and z-axes with fades."
  - "Avoiding animating into and out of blurs."
  — [Apple HIG: Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility)
- "Be cautious with fast-moving and blinking animations… in some cases even result in epileptic episodes." — [Apple HIG: Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility)

**Game feel and "juice"**
- Martin Jonasson and Petri Purho's GDC 2012 talk **"Juice it or lose it"** showed a plain Breakout clone becoming fun through flash, shake, floating text, sound and particles. — [eastondev summary](https://eastondev.com/blog/en/posts/dev/20260521-game-feedback-feel/); [GameAnalytics](https://www.gameanalytics.com/blog/squeezing-more-juice-out-of-your-game-design)
- Practitioner button-press tuning: about **80 ms press, scale to 0.92, restore on release**. Squash and stretch should conserve volume (stretch one axis while squashing the other), then spring back with overshoot. These are anecdotal starting values. — [eastondev](https://eastondev.com/blog/en/posts/dev/20260521-game-feedback-feel/); [itch.io blog](https://itch.io/blog/1059831/making-a-game-feel-juicy-with-simple-effects)
- Many mobile players play muted, so visual and haptic feedback must carry the meaning on their own. Juice should match the core mechanic; past a point it becomes noise. — [eastondev](https://eastondev.com/blog/en/posts/dev/20260521-game-feedback-feel/); [Game Developer: feedback](https://gamedeveloper.com/design/the-single-most-useful-advice-i-can-give-for-making-any-game-better-feedback)
- A Rive chest-opening case study argues that "the reveal isn't the point, the build-up is," and designs around anticipation. — [Contra: Treasure Chest in Rive](https://contra.com/p/cgBYLNQU-treasure-chest-opening-animation-in-rive)
- Clash Royale chests have per-chest animation and sound, plus a multi-tap reveal of cards from common to rarest. This comes from fan and simulator sources, not Supercell. — [Royale Tracker chest simulator guide](https://royaletracker.gg/guides/clash-royale-chest-simulator-guide)

### Inferences
**Timing tokens (my recommended starting set; tune in device testing)**

| Token | Value | Use |
|---|---|---|
| `tap.press` | 60–90 ms | scale 0.92–0.95 with lip compress, plus light impact haptic |
| `tap.release` | `.spring(duration: 0.3, bounce: 0.35)` | release overshoot |
| `ui.enter` | `.snappy` or `.spring(duration: 0.35, bounce: 0.15)` | elements entering |
| `ui.exit` | 0.2 s ease-in | exits run faster than entries |
| `character.react` | `.bouncy` / bounce 0.4–0.6 | characters only, never UI chrome |
| `stagger` | 30–50 ms | between list or grid items |
| `celebrate.total` | ≤ 2.5 s | per correct-answer celebration (repeated dozens of times a session); the big level-up can run 4–6 s but must be tap-to-skip |

**Reward reveal choreography**

1. **Anticipation**, 0.3–0.6 s: the chest wobbles and squashes and the music ducks. Repeated taps each add a shake and a rising tone.
2. **Burst**, about 0.1 s: white flash, heavy impact haptic and a light ray "god-ray" spin.
3. **Reveal**: the item scales from 0 to 1.15 to 1.0 with rarity-colored particles. Rarity color is a learned language (Brawl Stars/Clash Royale use rarity tiers).
4. **Settle**: the item drifts gently and idles.
5. **Collect**: the item flies to its inventory slot along an arc, the counter ticks up, and a counter "pop" plays.

- **Celebration layering order** for each beat (the Apple HIG principle that visual, audio and haptic intensity should match):
  - Visual: character reaction, particles, UI pop, background pulse.
  - Audio: stinger and musical layer.
  - Haptic: one success or impact.
- Fire all three channels within one frame of each other.
- **Rive vs Lottie:**
  - Rive suits interactive characters: state machines, inputs, blend states, lip-sync, runtime-tunable avatar parts.
  - Lottie suits linear "after-effects" flourishes such as confetti and badge stamps.
  - Native SwiftUI/SpriteKit suits UI chrome and particles.
- **Avatars:** for "avatar that looks like me" in Rive, plan bones and swappable components (hair, skin fill, glasses) driven by data-binding inputs, rather than separate artboards per combination.
- **Frame rate:** on ProMotion iPads and iPhone Pro models, budget for 120 fps. Avoid offscreen blur and heavy shadows on animating layers. Pre-warm particle emitters and audio before celebrations so the first frame of the reward never hitches.

### Gaps
- No primary Supercell data on chest/Starr Drop timing in milliseconds was found. The timings above are design inferences.
- No primary source confirmed Rive runtime performance numbers (file size, CPU) for Duolingo; rive.app was blocked.
- WWDC haptics sessions (WWDC19 520, WWDC21 10278) are referenced by the HIG but were not watched.

---

## 3. Sound design (UI palettes, musical feedback, voice-over, mixing, mute/accessibility)

### Takeaway
No official Duolingo sound-design write-up surfaced in search. Third-party commentary confirms the pattern: a bright "ding" for correct, a deflated falling tone for wrong, and a launch sound that reuses the correct-answer chime. Apple's HIG requires that audio is never the only carrier of meaning. Pair sounds with haptics and visuals, give people control over audio, and don't autoplay without controls.

### Cited Findings
- Duolingo's error cue is a deflated, falling tone, designed as a counterpart to its success cue. — [Out of Scope newsletter](https://out-of-scope-product.beehiiv.com/p/duolingo-has-it-venmo-has-it-even-slack-has-it-your-product-doesn-t)
- Duolingo's startup sound is the same as the correct-answer sound. The single "Sound effects" toggle mutes all effects (a coarse control). — [Stealth Optional](https://stealthoptional.com/apps/duolingo-startup-sound-how-to-turn-off/)
- Pok Pok uses "soothing sounds to create a calm, immersive environment" for ages 2–6. — [Mom's Choice Awards](https://www.momschoiceawards.com/?p=25849)
- Apple HIG on audio:
  - "Use haptics in addition to audio cues… pair that sound with matching haptics for people who can't perceive the audio or have their audio turned off."
  - "Augment audio cues with visual cues."
  - "Avoid autoplaying audio and video content without also providing controls."
  — [Apple HIG: Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility)
- Apple HIG on haptics:
  - Build "a clear, causal relationship between each haptic and the action."
  - Never reuse the failure pattern for success.
  - "Avoid overusing haptics."
  - "Make haptics optional."
  - Prefer short haptics for discrete events.
  - Custom haptics have **intensity and sharpness** parameters (soft and organic vs crisp and mechanical) and can be synced with audio.
  — [Apple HIG: Playing haptics](https://developer.apple.com/design/human-interface-guidelines/playing-haptics)
- iOS system haptic vocabulary:
  - Notification: success, warning, error.
  - Impact: light, medium, heavy, rigid, soft.
  - Selection.
  — [Apple HIG: Playing haptics](https://developer.apple.com/design/human-interface-guidelines/playing-haptics)
- Many mobile players play without audio. — [eastondev](https://eastondev.com/blog/en/posts/dev/20260521-game-feedback-feel/)

### Inferences
**Sound palette design.** Tune all UI sounds to one key, for example C or F major pentatonic, so any overlap sounds musical.
- **Combo / streak:** each consecutive correct answer plays the next pentatonic note up, resetting after a miss or a pause. This is the "pitch-rising correct" technique used widely in casual games, e.g. Peggle-style escalation (a well-known pattern, though I found no primary source this pass).
- **Wrong answer for kids:** use a soft, low, short "boop" or a character "hmm?", never a buzzer. Follow immediately with a hint. This matters most for the 2.5-year-old.

**Mix bus hierarchy.** Assign VO, SFX and Music their own buses.
- Duck music by about 6–10 dB under voice-over.
- VO is always the loudest element because it carries instructions for pre-readers.
- Offer separate parent-settings sliders for music, effects and voice. Don't copy Duolingo's single toggle.

**Voice-over direction for ages 2.5–8.5:**
- Warm, smiling delivery.
- Roughly 10–20% slower than adult narration.
- Pause before key words.
- Exaggerated intonation on the target sound or letter.
- Record Hebrew with full, careful vowel articulation so it matches nikud.
- Record the child-facing praise lines in many variants (8–15 per type) to avoid repetition fatigue.

**Personalization:** consider recording the child's name only via parent-recorded audio, or avoiding it altogether. TTS name pronunciation in Hebrew is risky. See also the privacy notes in section 6.

**Adaptive music:**
- Use stem-based layers: base loop, then a rhythm layer on streak, then a melody layer during battles.
- Use a quieter "focus" mix during reading tasks.

### Gaps
- No primary source on Duolingo's or Nintendo's sound-design philosophy, or on Monument Valley audio, was retrieved.
- No research-based dB mix targets for kids' apps were found. Values above are practitioner convention.
- No peer-reviewed guidance was found on voice-over speaking rate for children's learning apps.

---

## 4. Typography and Hebrew-specific design (nikud, fonts, sizes, RTL mirroring, bidi)

### Takeaway
Apple's HIG gives precise RTL rules:
- Flip progress, navigation and "forward-motion" icons.
- Don't flip logos, checkmarks, clocks or real-world objects.
- Never reverse digits within a number, but do reverse the order of numerals along a flipped progress or rating control.
- Bump Hebrew about 2 pt next to all-caps Latin.

Nikud quality depends heavily on the specific font and the rendering stack. Popular Google Hebrew sans fonts (Heebo, Varela Round) and Culmus Simple CLM support basic niqqud but have documented gaps or positioning errors. No authoritative source on "the" Israeli first-grade font was found.

### Cited Findings
**Apple HIG: Right to left** (read in full)
- "Adjust text alignment to match the interface direction."
- "Align a paragraph based on its language, not on the current context." A paragraph means three or more lines. One- and two-line blocks follow the UI direction.
- Use one consistent alignment for every item in a list, including items in another script.
- "Hebrew text uses Western Arabic numerals."
- "Don't reverse the order of numerals in a specific number" (e.g. 541 or a phone number).
- "Reverse the order of numerals that show progress or a counting direction; never flip the numerals themselves." A 5-star rating in Hebrew runs 1→5 from right to left.
- "Flip controls that show progress from one value to another" (sliders, progress bars), including their end-cap glyphs.
- "Flip controls that help people navigate… a back button must point to the right."
- "Preserve the direction of a control that refers to an actual direction" ("to the right" always points right).
- "Arabic or Hebrew text can appear too small when next to uppercased Latin text… increase the RTL font size by about **2 points**."
- "Avoid flipping images like photographs, illustrations, and general artwork." Instead, "reverse the positions of images when their order is meaningful."
- Flip icons that represent text or reading direction, and icons that show forward or backward motion (e.g. speaker waves). Localize icons that contain letters (SF Symbols ships Hebrew variants).
- "Don't flip logos or universal signs and marks" (checkmark). "Avoid flipping interface icons that depict real-world objects" (clock, pencil, game controller). Right-handed tools stay right-handed.
- For complex icons, consider each component separately (badges, slashes). Example: a cart badge moves to the top-left for visual balance.

— [Apple HIG: Right to left](https://developer.apple.com/design/human-interface-guidelines/right-to-left)

- UIKit's `UIImage.flipsForRightToLeftLayoutDirection` controls per-image flipping. — [Apple docs](https://developer.apple.com/documentation/uikit/uiimage/flipsforrighttoleftlayoutdirection.md)

**Type sizes and contrast (Apple HIG Accessibility)**
- iOS/iPadOS default text size is **17 pt; minimum 11 pt**. Thin weights need larger sizes.
- Support enlargement up to **200%** (Dynamic Type or custom).
- WCAG AA contrast: **4.5:1** up to 17 pt, **3:1** at 18 pt or larger, or for bold.

— [Apple HIG: Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility)

**Hebrew fonts and nikud**
- The Open Siddur font chart sorts open-source Hebrew fonts by diacritic support:
  - **Heebo**: niqqud only, missing varika and meteg.
  - **Varela Round**: niqqud only, missing varika, rafé, several meteg ligatures, qamats qatan and holam(-variant).
  - **Simple CLM**: niqqud, but with **rafé positioning errors** and no varika.
  - Google's **Noto Rashi Hebrew** has niqqud positioning errors.
  - **Open Sans Hebrew** is listed with full niqqud and te'amim support.
  — [Open Siddur: Open-source Unicode Hebrew fonts](https://opensiddur.org/help/fonts/)
- Varela Round Hebrew is by Avraham Cornfeld (Alef Alef Alef). It is designed for a "soft" feel, with no mention of nikud optimization. — [GitHub: Varela-Round-Hebrew](https://github.com/alefalefalef/Varela-Round-Hebrew/); [alefalefalef.co.il free fonts](https://alefalefalef.co.il/en/%D7%A4%D7%95%D7%A0%D7%98%D7%99%D7%9D-%D7%91%D7%97%D7%99%D7%A0%D7%9D-%D7%91%D7%A2%D7%91%D7%A8%D7%99%D7%AA-%D7%9E%D7%A8%D7%97%D7%91%D7%99-%D7%94%D7%A8%D7%A9%D7%AA/)
- Simple CLM is part of the open-source Culmus project. — [Fedora: clm-simple-fonts](https://packages.fedoraproject.org/pkgs/culmus-fonts/clm-simple-fonts); [openSUSE font info](https://fontinfo.opensuse.org/families/SimpleCLM.html)
- "The quality of the niqqud rendering directly depends on the font." Culmus David and Frank Ruehl render niqqud well. — [Linux-IL mailing list](https://mailman.cs.huji.ac.il/pipermail/linux-il/2012-February/008728.html)
- Noto Sans Hebrew failed to render niqqud on at least one e-reader stack. Support depends on the shaping engine as well as the font. — [crosspoint-reader issue #2312](https://upd.dev/daveallie/crosspoint-reader/issues/2312)
- The commercial font **Hebrew Rinat Kids** (Samtype) is marketed for teaching children to read, with nikud marks "25% bigger". This is a vendor claim. — [MyFonts: Hebrew Rinat Kids](https://www.myfonts.com/fonts/samtype/hebrew-rinat-kids)
- Israel's Ministry of Education guidance: nikud is maximal (full) in grade 1 and reduced gradually in later grades (snippet; the PDF was not fully parsed). — [Ministry of Education: nikod.pdf](https://meyda.education.gov.il/files/Mazkirut_Pedagogit/HebrewPrimary/learning-teaching/books/nikod.pdf); [acquisition-reading-writing.pdf](https://meyda.education.gov.il/files/Pop/0files/ivrit_hinuch_leshoni/Yesodi/acquisition-reading-writing.pdf)
- General beginner-font criteria (Latin context): no swashes, clear spacing, and easily distinguished similar letters. — [Adobe Express: best fonts for teachers](https://www.adobe.com/dk/express/learn/blog/best-fonts-for-teachers); [Picklebums](https://picklebums.com/fonts-for-beginning-writers/)

### Inferences
**Two-font Hebrew strategy:**
- **Display/UI font** (Varela Round, Rubik or Heebo) for buttons, counters and the Brawl-Stars-style titles, where nikud is not needed.
- **Reading font** for any text with nikud. Choose one with verified, complete niqqud anchoring. Candidates to test: Open Sans Hebrew, Taamey Frank / Frank Ruehl CLM (serif, classic school-book look), Simple CLM (check the rafé issue, which matters little since rafé is rare in kids' text), and commercial Rinat Kids.

**Nikud rendering acceptance test.** Render on device every letter × every vowel, plus dagesh and shin/sin dots, at the target size, and check for:
- Collisions with descenders (ק, ך, ן, ף, ץ).
- Dagesh legibility inside narrow letters (ו, ז, י).
- Holam male vs holam haser.
- Shin dot vs holam collision (שׁ with ֹ).
- Line-height clipping. Add 20–35% extra leading for nikud below the baseline.
- Behavior with CoreText vs SwiftUI `Text` vs any game engine text renderer (Unity/TextMeshPro needs a proper HarfBuzz/RTL shaper).

**Learning-target sizes** (my estimates):
- Age 5.5: about 34–48 pt for a single letter or syllable with nikud, about 28–32 pt for short words.
- Age 8.5: about 22–26 pt.
- Age 2.5: no reading text, only icons and voice.
- Scale nikud with size; a font whose marks are about 20–25% heavier than adult defaults is preferable, as in the Rinat Kids claim.

**Confusable letter pairs.** Test the font specifically on these pairs for beginning readers: ב/כ, ד/ר/ך, ה/ח/ת, ו/ז/ן, ס/ם, ע/צ, and ט/ש/ש. Prefer fonts with exaggerated distinguishing features (e.g. a clear serif foot on ב).

**Bidi in a bilingual game:**
- Wrap English inside Hebrew strings with Unicode isolates (FSI/PDI, U+2068/U+2069) to stop punctuation and number jumps.
- Never concatenate localized strings. Use full-sentence format strings with placeholders.
- Scores, timers and level numbers stay LTR digits, and their containers still mirror.
- Health bars, XP bars and progress paths fill right→left in Hebrew mode, and "next" arrows point left.
- Do not mirror character art, avatars or the game world. Do mirror the *order* of sequences, such as a level path that progresses right→left.

**Language switching:** let a child or parent switch language per profile (Hebrew for the 5.5 reader, English for EFL practice). Use iOS per-app language and `layoutDirection` environment overrides, not a device-wide setting.

### Gaps
- No authoritative source identified which typeface Israeli first-grade textbooks or readers (e.g. ministry-approved "ספרי קריאה") typically use. This needs a check with publishers such as Matach (מטח) or Kinneret.
- No peer-reviewed study on Hebrew letter-shape legibility for children was found in this pass.
- Rubik's and Assistant's niqqud completeness was not verified. Their Google Fonts glyph coverage pages were not fetched.
- iOS-specific nikud rendering quality of Apple's system Hebrew font (SF Hebrew / Arial Hebrew fallback) was not verified.

---

## 5. Avatar creator design (Mii, Bitmoji, Roblox, Brawl Stars skins, Animal Crossing, Memoji)

### Takeaway
Likeness comes from **free placement and scaling of a small set of stylized parts** (Nintendo's Mii insight), not from realism. Inclusivity comes from a **broad base set of skin tones, hair textures and accessories**, including head coverings and assistive devices (the Memoji trajectory), and gender-neutral item pools (Animal Crossing). Roblox shows the community cost of a limited skin-tone set.

### Cited Findings
- **Mii origin:**
  - The face editor began as a "Fukuwarai"-like tool (a traditional Japanese face puzzle) in which parts are placed on a face.
  - The first version had fixed part positions, until the team worried they "might not be able to make our boss, Sakamoto-san's face!"
  - Letting players **rotate, move and resize parts** made likenesses far closer. Sakamoto: "when we made it so that you could freely adjust those things, we were able to make the avatars (Mii) that really looked like the original."
  - Hosted on a fan archive; cross-checked with Nintendo Life and My Nintendo News.
  — [RC24 Mii story interview](https://interviews.rc24.xyz/story_mii.html); [Nintendo Life](https://www.nintendolife.com/news/2016/06/mii_characters_evolved_from_a_project_called_womens_fortune_telling_pocket_notebook); [My Nintendo News](https://mynintendonews.com/2016/06/30/a-multiple-part-interview-with-yoshio-sakamoto-shigesato-itoi-will-talk-about-miitomos-potential-mii-history/)
- The Mii concept took about 20 years and went through prototypes from the NES era onward. A canceled GameCube camera project ("Stage Debut") was part of that history. — [Unseen64](https://www.unseen64.net/2008/04/14/mii-nes-prototype/); [Engadget 2007](https://www.engadget.com/2007/03/16/the-past-and-future-of-miis/)
- **Memoji inclusivity timeline:**
  - Headscarf at launch (iOS 12).
  - 2020: 20 new hairstyles, more ages, more skin colors, face masks and more headwear.
  - iOS 15: hearing aids, cochlear implants, oxygen tubes and soft helmets.
  — [Buro247](https://buro247.me/?p=22095); [IPS News](https://ipsnews.net/business/?p=30298); [Malay Mail 2021](https://www.malaymail.com/news/tech-gadgets/2021/07/17/for-world-emoji-day-have-fun-with-apples-new-memoji-updates/1990550)
- **Animal Crossing: New Horizons:**
  - Skin-tone selection, confirmed at E3 2019.
  - Hairstyles not locked to gender.
  - The 2.0 update (Nov 5, 2021) added 11 hairstyles.
  - Players and press asked for more inclusive textured-hair options after launch.
  — [Nintendo Wire](https://nintendowire.com/news/2019/06/12/animal-crossing-new-horizons-to-feature-full-character-customization-including-skin-tones-gender-neutral-hairstyles); [Animal Crossing World](https://animalcrossingworld.com/2020/02/character-creator-and-editor-detailed-in-new-animal-crossing-new-horizons-videos-pray-for-harriet/); [Nylon](https://www.nylon.com/beauty/animal-crossing-new-horizons-players-want-more-inclusive-hairstyle-options)
- **Roblox:** a DevForum thread, "My skin color feels underrepresented on Roblox," asks for more skin-tone options. This shows the cost of a limited preset palette. — [Roblox DevForum](https://devforum.roblox.com/t/my-skin-color-feels-underrepresented-on-roblox-add-more-skin-color-options/1689297)
- Apple HIG: "Consider allowing people to customize color schemes such as… game characters so they can personalize your interface in a way that's comfortable for them." — [Apple HIG: Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility)
- **Khan Academy Kids** added 50 human characters "inspired by real kids and families" for representation. — [Khan Kids Help](https://khankids.zendesk.com/hc/en-us/articles/360049358751-Learn-more-about-the-characters-inside-Khan-Academy-Kids)

### Inferences
**Recommended avatar part system** (Mii-style, kid-proof):
- Skin: about 12–16 tones on a perceptually even ramp, plus fantasy colors for the 8.5-year-old's hero mode.
- Face shape: 4–6 options.
- Eyes: 8–12 shapes plus 8 colors.
- Brows: 6–8.
- Nose: 6.
- Mouth: 6–8, asymmetric as in Duolingo.
- Hair: 30+ styles covering straight, wavy, curly, coily/afro, braids, locs, ponytails, buzz and peyot/side-curls. Each style comes in about 12 natural plus fantasy colors.
- Facial details: freckles, dimples, birthmarks and missing front teeth. Missing teeth is a big age-5–8 identity marker.
- Accessories: glasses (several frames), hearing aids/cochlear implants, kippa (several styles), hijab/headscarf, cap, hood/hero helmet.
- Body: 3–4 sizes.

**Likeness without photos.** Put the parent in the loop: a "make my kid" flow with a few big choices (skin, then hair style and color, then eyes, then glasses yes/no) and an optional fine-tune with Mii-style move/scale sliders.
- Avoid any photo or face-scan feature. It adds biometric-data and COPPA/GDPR-K complexity (see App Store 5.1.4 in section 6).

**Child-facing editor UX:**
- Use a big live preview with idle animation, and have the avatar *react* to every change (laughs at a new hat, blinks at glasses).
- Use a horizontal part carousel with large swatches, randomize (die icon) and undo.
- Avoid text tabs; use icon tabs (face, hair, clothes).

**Wardrobe/cosmetics integration (Brawl Stars skins model).** The avatar is the kid's "brawler". Skins and outfits are earned by learning, never purchased by the child.
- Rarity colors (common gray/green → rare blue → epic purple → legendary gold) give the 8.5-year-old the Brawl Stars status language.
- Keep the toddler's earned items purely cosmetic and immediate.

**Siblings.** Three kids on one device means a profile picker built from avatar faces only (no names needed). Each sibling's avatar can appear in the others' worlds as a cameo, which is a strong motivator.

### Gaps
- No primary design article on Bitmoji's or Roblox's avatar editor UX was retrieved.
- No Nintendo primary source (e.g. Iwata Asks on the Mii Channel) was fetched. The Iwata Asks Wii Sports page exists but was not read.
- No research on how young children (2–5) recognize themselves in stylized avatars was found.

---

## 6. UX for kids (text-free navigation, world-as-home, onboarding, parent gates, errors, accessibility, touch targets, layouts)

### Takeaway
The hard numbers are:
- NN/g: about **2 cm × 2 cm targets for young children**, four times the 1 cm adult minimum.
- Apple: 44×44 pt default, 28×28 pt absolute minimum, with **12 pt padding around bezeled and 24 pt around unbezeled controls**.
- App Store 1.3: links out, purchases and "other distractions" must sit behind a parental gate.

Text-free design (Toca), world-as-hub, no-shame error handling and difficulty accommodation are the craft norms.

### Cited Findings
- **NN/g:** tested children aged 3–12 in the US and China. Swiping, tapping big targets and dragging "were easy even for the youngest users". The recommendation is **at least 2 cm × 2 cm touch targets for young children**, versus 1 cm × 1 cm for adults (snippet). — [NN/g: Children's UX: Physical Development](https://www.nngroup.com/articles/children-ux-physical-development/)
- The 1 cm adult minimum derives from Parhi, Karlson & Bederson (snippet). — [NN/g: Touch Target Size](https://www.nngroup.com/articles/touch-target-size/)
- Children have less motor control, so their effective contact patch is about the same size as an adult's despite smaller fingers. — [Medium: Size matters](https://medium.com/@zacdicko/size-matters-accessibility-and-touch-targets-56e942adc0cc)
- **Apple control sizing:**
  - iOS/iPadOS default control size **44×44 pt**, minimum **28×28 pt**.
  - "Consider spacing between controls as important as size". Use about **12 pt padding** around bezeled elements and about **24 pt** around elements without a bezel.
  - "Use the simplest gesture possible — avoid custom multifinger and multihand gestures."
  - "Offer alternatives to gestures."
  — [Apple HIG: Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility)
- **Apple on color, timing and difficulty:**
  - "Convey information with more than color alone… red-green and blue-orange." Example: green circle with a checkmark vs red octagon with an X.
  - "Minimize use of time-boxed interface elements… Prefer dismissing views with an explicit action."
  - "Consider offering difficulty accommodations in games… reduce the criteria for successfully completing a level, adjust reaction time, or enable control assistance."
  — [Apple HIG: Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility)
- **Assistive Access** (iOS 17+) is a streamlined mode for cognitive disabilities. Guidance: remove noncritical workflows, one interaction per screen, confirm twice for hard-to-undo actions. — [Apple HIG: Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility)
- **App Store 1.3 Kids Category:**
  - "Must not include links out of the app, purchasing opportunities, or other distractions to kids unless reserved for a designated area behind a parental gate."
  - Kids Category apps should not include third-party analytics or third-party advertising, with narrow exceptions.
  - Once in the category, the app must keep complying even if it later deselects it.
  — [App Store Review Guidelines 1.3](https://developer.apple.com/app-store/review/guidelines/)
- **App Store 5.1.4:**
  - A parental gate "is generally not the same as securing parental consent to collect personal data". COPPA/GDPR still apply.
  - Apps that collect names, photos, drawings and similar data from minors need a privacy policy.
  - "For Kids" wording in metadata is reserved for the Kids Category.
  — [App Store Review Guidelines 5.1.4](https://developer.apple.com/app-store/review/guidelines/)
- **Text-free design:** Toca Boca minimizes text and instructions ("a good toy doesn't need instructions"). — [Kill Screen](https://www.killscreen.com/what-designers-can-learn-about-play-children/); [GamesBeat](https://gamesbeat.com/how-toca-boca-got-100m-mobile-downloads-by-putting-kids-first/)
- **Gestures:** prefer tap over adult gestures such as pinch for kids. Children under 3 can imitate several adult touchscreen gestures. — [UserTesting: UX for kids](https://www.usertesting.com/blog/ux-for-kids); [UB thesis PDF](https://diposit.ub.edu/dspace/bitstream/2445/128111/1/684570.pdf)
- A Fast Company critique of NN/g's kids report argues that usability research on kids often ignores well-being impacts. — [Fast Company](https://www.fastcompany.com/90217009/design-is-never-neutral)

### Inferences
**Touch targets by age** (my synthesis of NN/g's 2 cm and Apple's 44 pt):

| Age | Primary targets | Spacing | Dragging |
|---|---|---|---|
| 2.5 | ≥ 2 cm, about 75–90 pt on iPhone / 90–120 pt on iPad | ≥ 24 pt | generous snap zones, about 1.5× the visual target; accept "close enough" drops |
| 5.5 | about 60–70 pt | | |
| 8.5 | ≥ 44–56 pt | | |

- Hit areas can exceed the visual art.
- Ignore palm and multi-touch rejection edge cases. Debounce double taps from toddlers.

**Home screen as a world:**
- A scrollable island or map where each learning area is a place (castle = Hebrew letters, arena = math battles, workshop = building).
- The child's avatar walks there.
- Locked areas show as "under construction" with a friendly character, not padlocks.
- Each child's world reflects their progress (buildings grow), giving a visible sense of building.

**First-time experience under 60 seconds:**
1. Parent sets up profiles once (behind the gate): age band, language, avatar quick-build.
2. Child sees their avatar wave and say their name.
3. A mascot demonstrates one tap.
4. The first activity starts within about 15 s.
5. First reward at about 45 s.

- No text, no tutorial screens; teach through the first interaction.

**Parent gate:**
- Use a press-and-hold of about 3 s on two points, or a multiplication question, or reading-required instructions ("tap the number seven", written out as words). Avoid simple "swipe" gates that 5-year-olds learn.
- For a 8.5-year-old, use arithmetic beyond their level, or a parent PIN.
- Placement: settings, store links, any purchase, external links.
- Use Apple Screen Time and Family Sharing "Ask to Buy" for any IAP.

**Errors without shame:**
- A wrong answer gets a soft sound and a character reaction ("hmm, let's try!"). The wrong option wiggles and fades rather than flashing red.
- The correct option gently glows after a second miss.
- Never show lives or hearts for under-6s.
- Never lose earned items.

**Empty states:** always show something alive (a character napping or a seed waiting to grow) plus one big obvious action. Never a blank screen or a text-only message.

**Layout by device:**
- iPhone: landscape-first for game screens (like Brawl Stars). Keep primary actions in the thumb zones, the lower corners, mirrored for RTL.
- iPad: two-hand reach, and a larger world view with more parallax.
- Respect safe areas.
- Never place tap targets under or right next to the Dynamic Island or the home indicator. Children swipe up accidentally, so consider Guided Access guidance for parents.

**Color-blind safety:**
- Correct means a green checkmark circle; wrong means a soft shape change, not red-vs-green alone.
- Rarity uses color *and* frame shape or sparkle count.

**Three-age-band adaptation within one app:**
- **2.5:** voice-only, toy-like, no fail states, no timers.
- **5.5:** emerging reading with nikud, light goals.
- **8.5:** heroes and battles, streaks, collections, light competition with siblings (co-op preferred).

### Gaps
- Could not read NN/g's full children's articles (nngroup.com blocked). Its detailed findings on age-specific gesture difficulty (drag-and-drop accuracy, accidental taps) were not verified.
- No primary research on parental-gate effectiveness by age was found. Apple's own kids-apps page (developer.apple.com/app-store/kids-apps/) was linked but not read.
- No source on Dynamic Island interaction conflicts in kids' games was found.

---

## 7. Polish checklist: what separates "good" from Apple Design Award quality

### Takeaway
ADA-level polish is systemic:
- A few strict tokens for timing, color, shape and sound.
- Every input answered in about 100 ms across visual, audio and haptic channels, at matched intensity.
- No dead or empty states.
- Accessibility honored (Reduce Motion, contrast, larger text, haptic and audio options).
- Relentless playtesting with real kids (Pok Pok tested every iteration).

Restraint matters as much as juice. Apple and game-feel practitioners both warn that overused feedback becomes noise.

### Cited Findings
- The ADA "Delight and Fun" category was won by Pok Pok in 2021, which emphasized calm, toy-like, playtested design. — [MacStories](https://www.macstories.net/linked/2021-apple-design-award-winners-the-appstories-interviews/); [Mom's Choice Awards](https://www.momschoiceawards.com/?p=25849)
- "Often, the best haptic experience is one that people may not be conscious of, but miss when it's turned off." Match haptic intensity and sharpness to the animation. — [Apple HIG: Playing haptics](https://developer.apple.com/design/human-interface-guidelines/playing-haptics)
- "When visual, auditory, and tactile feedback are in harmony… the user experience is more coherent." — [Apple HIG: Playing haptics](https://developer.apple.com/design/human-interface-guidelines/playing-haptics)
- Accessibility features Apple expects:
  - Dynamic Type up to 200%.
  - Contrast at AA.
  - Reduce Motion handling.
  - Don't rely on color alone.
  - Captions for audio.
  - Audio controls.
  - Accessibility Nutrition Labels on the App Store.
  — [Apple HIG: Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility)
- Supercell's stated craft bar: "quality, consistency, readability, and polish". — [Supercell careers](https://supercell.com/en/careers/senior-ui-artist/66476c5d-30b6-4772-aa41-94d251e3d73c/)
- "Juice it or lose it" (GDC 2012): feedback layering turns a functional game into a fun one. Practitioners warn against excess. — [eastondev](https://eastondev.com/blog/en/posts/dev/20260521-game-feedback-feel/); [Game Developer](https://gamedeveloper.com/design/the-single-most-useful-advice-i-can-give-for-making-any-game-better-feedback)
- Duolingo's documented, repeatable illustration rules let a large team scale one style, a polish multiplier. — [Creative Review](https://www.creativereview.co.uk/duolingo-rebrand-johnson-banks/)

### Inferences
**ADA-grade polish checklist** (synthesized from the above plus craft practice):

1. **Tokens:** motion durations and springs, colors with shade and highlight steps, radii, outline weights, shadow style, sound set (one key) and haptic map. Store them in code and in Figma/Rive, never hard-coded.
2. **Every tap responds within one frame:** a press state, plus sound and haptic on release. Disabled buttons still respond with a wiggle and a "not yet" sound.
3. **No dead states:**
   - Every screen has an idle animation (breathing characters, drifting clouds).
   - Every wait over 300 ms shows a playful loader.
   - Every wait over 2 s gives the child something to do, such as tapping bubbles.
4. **Perceived performance:**
   - Preload the next activity during the celebration.
   - Pre-warm audio, particles and Rive files.
   - Hold 60/120 fps steady. Profile with Instruments on the oldest supported iPad.
5. **Skippable and interruptible:** celebrations and transitions can be skipped by tap after the first viewing, and animations can be interrupted (springs retarget smoothly).
6. **Consistent cause and effect:** the same success sound and haptic always means success, and failure never shares success feedback (HIG).
7. **Accessibility pass:**
   - Reduce Motion variant (fades, no shake).
   - Color-blind sims.
   - Larger text support in reading UI.
   - Separate music, SFX and voice sliders.
   - Haptics toggle.
   - Switch Control and VoiceOver labels on core navigation.
   - Fill in Accessibility Nutrition Labels.
8. **RTL/LTR parity:**
   - Screenshot both languages for every screen.
   - Check icon mirroring against the HIG list.
   - Check numbers stay LTR.
   - Check progress paths reverse.
   - Run nikud rendering QA.
9. **Kids' edge cases:**
   - Mashing, multi-touch and palm rejection.
   - Rapid back-and-forth taps.
   - App backgrounding mid-celebration.
   - Silent switch.
   - Bluetooth headphones latency (sync haptic and audio).
   - Guided Access compatibility.
10. **Playtest cadence:** every build is tested with the three target kids. Watch silently and record where fingers hesitate (Pok Pok/Khan Kids practice).
11. **Restraint audit:** for each effect, ask whether it communicates something. Cut decorative motion in reading moments, which should be calm (Pok Pok register). Spend juice on rewards and battles (Supercell register).
12. **Privacy-by-design:**
    - No third-party analytics or ads in kids' flows (App Store 1.3).
    - Avatars built from parts, not photos.
    - Parental gate on settings, links and purchases.

### Gaps
- No Apple primary source lists explicit ADA judging criteria beyond category names. No ADA jury commentary on kids' apps was retrieved.
- No quantitative studies linking specific polish factors (latency thresholds, frame rate) to kids' engagement or learning outcomes were found.
- WWDC sessions on designing for kids, or Apple's "Designing for kids" developer page, were not retrieved in this pass.
