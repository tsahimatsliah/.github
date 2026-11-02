# Research log — UX/UI design system (spec 09)

> Written 2026-11-02 for `docs/spec/09-ux-ui-design-system.md`, per F3. Each entry gives the question, the method, the result, the confidence, and what the spec does with it.
> **Session limits.** The shared WebSearch budget for this turn ran out before the gap searches started. Direct fetches of `docs.swmansion.com` (Reanimated) and `rive.app` failed: these hosts are outside the session's allowed network domains and must be allowed before re-checking. Because of that, items marked **[BK]** come from background knowledge (pre-2026 training) and were *not* re-verified in this session. Each one is flagged for re-verification at build time (see the "Re-verify" list at the end). The font findings, contrast checks, colour-blind simulation and spring maths were **measured or computed in this session** and are reproducible (scripts described below).

---

## R1. Hebrew fonts with good nikud rendering (free / OFL, Google Fonts)

**Method (measured).** I downloaded each family as TTF via the Google Fonts CSS2 API (`fonts.googleapis.com/css2?family=…`, TTF served to a non-browser UA). With `fontTools` I checked:
- the cmap coverage of all 27 Hebrew letters and 18 nikud marks (U+05B0–U+05BD, U+05BF, U+05C1, U+05C2, U+05C7);
- the GPOS `mark`/`mkmk` features and GSUB `ccmp` under the `hebr` script.

I then shaped 16 hard test clusters with HarfBuzz (`uharfbuzz`), including שָׁלוֹם, ךָ, קֻ, ףְ, שׂוֹ, זִּ, וּ, תּ, לֹא, מֹשֶׁה and צְּ, and rendered specimen sheets with Pillow + libraqm at 56–110 px. Scripts are in the session scratchpad (`nikud_check.py`, `render_sheet.py`, `render_zoom.py`, `metrics.py`). Caveat: HarfBuzz is not CoreText. iOS rendering must still be checked on a device (GATE-UX-2).

**Coverage result** (letters / missing marks / Hebrew GPOS features):

| Family | Letters | Missing marks | GPOS (hebr) | Notes from render |
|---|---|---|---|---|
| **Noto Sans Hebrew** | 27/27 | none | kern, mark, mkmk | All test clusters correct. Qamats sits inside ךָ, shva inside ףְ, qubuts beside ק. Ink height with nikud ≈ 0.98 em. |
| **Noto Serif Hebrew** | 27/27 | none | kern, mark, mkmk | Correct. Serif "book" look. |
| Noto Rashi Hebrew | 27/27 | none | kern, mark, mkmk | Rashi script. Not for teaching. |
| **Rubik** | 27/27 | meteg, rafe | kern, mark, mkmk | Correct placement. Static weights 300–900 (variable on GF). Latin + tabular numerals (`tnum`). Ink ≈ 0.99 em. |
| **Fredoka** | 27/27 | meteg | kern, mark, mkmk | Correct placement. Rounded and friendly. Ink ≈ 1.03 em. ו/ז distinction is weaker. |
| Varela Round | 27/27 | meteg, rafe | kern, mark, mkmk | Correct placement, but ש/ט shapes are non-standard (U-like). Single weight. Not for learning glyphs. |
| Assistant | 27/27 | meteg | kern, mark, mkmk | Correct. Light and narrow, so smaller at the same pt. |
| Heebo | 27/27 | meteg | mark, mkmk | ףְ renders oddly (marks crowd inside). Deepest descent with nikud (0.34 em). Avoid for nikud. |
| Secular One | 27/27 | meteg, rafe | kern, mark, mkmk | Heavy display. Marks stay legible. Single weight. |
| Suez One | 27/27 | meteg, rafe | kern, mark, mkmk | Heavy serif display. |
| Karantina | 27/27 | meteg | kern, mark | Condensed display only. Marks crowd. |
| Open Sans (Hebrew) | 27/27 | rafe | mark, mkmk | **Bug seen:** qamats on ךָ renders below/at the descender instead of inside the letter. Avoid for nikud. |
| Alef | 27/27 | meteg, rafe | mark (no mkmk) | Acceptable. Lacks mkmk (stacked marks). |
| Frank Ruhl Libre | 27/27 | none | kern, mark, mkmk | Classic school-book serif with the clearest ב/כ and ד/ר. Ink ≈ 1.10 em. |
| David Libre | 27/27 | meteg | kern, mark, mkmk | Fine. |
| Miriam Libre | 27/27 | meteg, rafe | kern, mark, mkmk | Fine. |
| IBM Plex Sans Hebrew | 27/27 | meteg, qamats-qatan | kern, mark | No mkmk. |
| Bellefair | 27/27 | meteg, rafe | kern, mark | Display serif. |

- **Fredoka has Hebrew**, which answers the question in the brief. Nunito and Baloo 2 have no Hebrew.
- Meteg and rafe are rare in children's text. The missing meteg does not matter for Dicta-Nakdan-style output. Qamats-qatan (U+05C7) matters only if the nakdan emits it. Normalise it to qamats U+05B8 in the content pipeline if the chosen font lacks it.

**Measured ink extents** (Pillow bbox relative to the baseline, at 100 px, with nikud strings such as "לֹא קֻ לְּךָ"):
- Noto Sans Hebrew: top −0.76 em, bottom +0.22 em
- Rubik: −0.72 / +0.27 em
- Fredoka: −0.75 / +0.28 em
- Heebo: −0.83 / +0.34 em
- Frank Ruhl Libre: −0.81 / +0.29 em

So pointed text needs roughly 1.0–1.05 em of ink per line. With a 0.45 em clear gap between the lowest mark of one line and the lamed/holam of the next, line-height ≈ **1.5–1.6** gives a safe result, and 1.35 is enough for unpointed text. This matches v1's "+20–35% leading".

**Confusable letter pairs** (render at 96 px; ב/כ, ד/ר/ך, ה/ח/ת, ו/ז/ן, ס/ם, ע/צ, ט/ש):
- Noto Sans Hebrew and Rubik: clear.
- Frank Ruhl: clearest (serif foot on ב, distinct ר shoulder).
- Fredoka: ו/ז weaker.
- Varela Round: ש/ט stylised.

**Decision used in spec 09:**
- Reading and learning text: **Noto Sans Hebrew** (complete marks, correct placement, nine weights).
- Display and UI: **Rubik** (chunky 800/900 weights for the Supercell-style register, tabular digits, Latin).
- English islands: **Andika** (see R1b).
- Candidate B for K2/K3 reading at the font gate: **Fredoka**. Candidate C for K1 story reading: **Frank Ruhl Libre**.
- All are OFL.
- Ship **static TTFs per weight**, not variable fonts. React Native's `fontWeight` mapping onto variable custom fonts on iOS is unreliable [BK].

**Confidence:** high for coverage and placement under HarfBuzz; medium for iOS CoreText parity until GATE-UX-2.

### R1b. English beginner font
**Method:** rendered "a g I l 1 b d p q cat dog" in Andika, Lexend, Rubik and Fredoka.

**Result:** Andika (SIL, built for beginning readers) has a single-storey "a", a distinct serifed "I", a tailed "l" and a distinct "1". Rubik and Fredoka make I/l nearly identical.

**Decision:** Andika for English-island learning text. Rubik for English UI chrome.

---

## R2. Supercell UI style (chunky buttons, outlines)
- **No new primary source this session** (search budget exhausted). The existing evidence is in `ui_ux_polish_craft.md` §1: Supercell UI priorities "quality, consistency, readability, polish", readable on the subway, oversized icons, strong contrast, plus the genre-observed anatomy (rounded face, dark 4–8 pt lip, thick dark stroke, inner highlight, heavy outlined display type, icon breaking the bounds).
- **New quantitative support (computed, see R5):** on a cream background (#FFF8EC), the saturated button faces have only 1.46–4.06:1 contrast. That is below WCAG 1.4.11's 3:1 for UI components in most cases. A dark ink outline (#1B1530) gives 4.1–11.4:1 against every face. **So the Supercell-style dark outline is an accessibility requirement here, not only a style choice.** Spec 09 makes the outline mandatory on all interactive surfaces.

## R3. Reanimated springs equivalent to iOS presets
- Apple's `spring(duration:bounce:)` maps to a mass-spring system with mass 1: stiffness = (2π/duration)², damping = 4π(1−bounce)/duration, dampingRatio ≈ 1−bounce. This was confirmed via a WebFetch summary of Apple's API doc page; the page body did not render, so treat it as [BK]-plus-summary. Defaults: duration 0.5 s, bounce 0.
- iOS presets [BK]: `.smooth` = (0.5, 0); `.snappy` = (0.5, 0.15); `.bouncy` = (0.5, 0.3). Apple warns that bounce above about 0.4 looks exaggerated in UI (WWDC23, cited in `ui_ux_polish_craft.md` §2).
- **Reanimated 4 `withSpring` defaults could not be verified** (docs host blocked). The Reanimated 3 docs described `duration` as *perceptual*, with actual settle ≈1.5× [BK], and default physics changed across versions [BK].
- **Decision:** spec 09 defines every spring token as **explicit `{mass, stiffness, damping}`** computed from Apple's formula (table in spec §5.2). Results never depend on library defaults. Verify feel on device at GATE-UX-3.

## R4. Kids' UI sound design and mixing
- Evidence carried over:
  - Duolingo's paired correct/incorrect cues: a bright rising cue and a deflated falling one.
  - Apple HIG: never audio-only, pair audio with haptics and visuals, avoid autoplay without controls.
  - Pok Pok's "soothing sounds".
  - Practitioner conventions: success sounds 150–400 ms, major intervals, pitch-rising combo, random ±5–10% pitch on repeated sounds, music ducked 6–10 dB under VO.
  - Gelman: loud, constant feedback irritates the adults nearby.
  - Sources: `hit_games_engagement.md` §4, `ui_ux_polish_craft.md` §3, `practitioner_wisdom.md` §5.
- Loudness [BK]: Sony's ASWG-R001 recommends about **−24 LUFS integrated for console/home and −18 LUFS for portable/mobile**, with true peak ≤ −1 dBTP. The spec adopts −18 LUFS ±2 for the master on mobile, re-verify.
- `expo-audio` has no mixer bus or sidechain compressor, so ducking must be implemented as scripted gain ramps. Assets are loudness-normalised offline (ffmpeg `loudnorm`) [BK].

## R5. Colour-blind-safe palette for kids (computed)
**Method:**
- WCAG 2.x relative-luminance contrast for every text/background and UI pair.
- Colour-vision-deficiency simulation with Machado et al. 2009 matrices (severity 1.0) for protan, deutan and tritan.
- CIE76 ΔE between confusable pairs before and after simulation.
- Script: `palette.py`.

**Key results:**
- White text on saturated "500" faces fails: leaf 2.17, sea 2.69, berry 3.21, grape 4.28. Ink #1B1530 on every face passes 5.47–11.35. **Rule:** labels on colour use ink text, or white text with an ink stroke, at sizes of 18 pt or larger only.
- Face vs cream background is 1.46–4.06 (fails 3:1), and the ink outline vs face is ≥4.1. This is the outline rule (R2).
- Under CVD simulation, some pairs collapse:
  - leaf vs ember: ΔE 12.3 under protan
  - berry vs teal: ΔE 11.9 under deutan
  - leaf vs teal: ΔE 5.6 under tritan

  So subject and region colours are *never* the sole carrier of meaning: each has a glyph and a landmark.
- correct (leaf) vs try-again (stone) stays distinct under all three simulations (ΔE 43–67), but their luminance contrast is only 1.30, so shape (check vs "?" and wobble) remains mandatory.
- Tier colours (bronze/silver/gold) are separable (ΔE ≥26) but silver vs gold has luminance contrast 1.01, so tiers also use frame shape and star count (gap_audit D2).
- The Okabe–Ito CVD-safe set [BK] was considered. It is too muted for the Supercell register, so spec 09 keeps saturated brand hues plus mandatory shape coding, which is the approach Apple HIG recommends ("convey information with more than colour alone").

## R6. React Native RTL gotchas (2026)
- From `app_stack_ai_built_workflow.md` Q2 (cited there):
  - `forcesRTL` needs a native rebuild.
  - The RN 0.84 iOS ScrollView/FlatList invisible-on-revisit bug under `forceRTL`.
  - `<Text>` alignment divergence without explicit `textAlign`.
  - react-native-slider follows the device locale.
  - react-native-screens native headers ignore I18nManager.
- From `tech_3d_world.md` Q4/Q6: the 3D world is never mirrored. Hebrew lives only in the 2D HUD. Joystick left and actions right regardless of RTL.
- [BK], to verify on RN 0.88:
  - `transform: translateX` and Reanimated/gesture translations are not mirrored, so a `dirX()` helper is needed.
  - Skia canvases have no RTL concept.
  - Horizontal ScrollView initial offset in RTL has historically been buggy on iOS.
  - Variable custom fonts' `fontWeight` is unreliable.
  - `I18nManager.forceRTL` takes effect only after a reload.
  - On web, set `<html dir="rtl" lang="he">`.
- Spec 09 §3.11 and §15 turn these into lint rules and regression screenshots.

## R7. Rive state-machine best practices
- **Could not fetch rive.app** (blocked). [BK], re-verify:
  - Rive's recommended runtime API moved from per-input state-machine inputs to **Data Binding (View Models)**.
  - Use one state machine per artboard (`Main`), with layers for independent channels (body/face/mouth).
  - Use Rive Events to notify the runtime (e.g., `reveal_done`).
  - Keep nested artboards for swappable parts.
- From `app_stack_ai_built_workflow.md` Q2: `@rive-app/react-native` (Nitro) uses `useRiveFile` + `RiveView`. Web support is unconfirmed, so plan a `.web.tsx` split.
- 00 E1a: the Rive Nitro runtime is not in Expo Go, so a Skia/Lottie placeholder is needed until the dev build. Since E1a's update, the primary path is the dev build from day one, which lowers this risk, but web artifacts still need the fallback.
- **Spec decision:** every character is specified as a *state contract* (inputs/view-model properties + events) that both Rive and a Skia-sprite placeholder implement (spec §7, `CMP-CharacterRig`).

## R8. Duolingo / Brawl Stars onboarding teardowns
- No new sources (search budget). Carried over from the notes:
  - Duolingo's home path is "probably the most successful mechanic" (Mazal); a milestone animation gave +1.7% D7.
  - World 1-1 wordless teaching.
  - First-time experience under 60 s with the first reward at about 45 s (`ui_ux_polish_craft.md` §6 inference).
- [BK], not re-verified:
  - Duolingo lets learners take a first lesson before account creation ("gradual engagement").
  - Brawl Stars onboards with a short guided bot match teaching move/aim/super, then unlocks modes progressively along the Trophy Road. Each new system is introduced one at a time at a milestone.
- **Spec decision:** profile setup is parent-only (H2). The child's first session is play-first: the first interaction within 15 s, no tutorial screens, and one new system introduced per milestone (shop at the first chest, album at the first rescue, and so on).

## R9. 3D / 2.5D world, controls, movement (read from new notes, 2026-11-02)
- `tech_3d_world.md`:
  - Toon/Lambert materials, inverted-hull outlines and blob shadows.
  - Budgets: ≤80 draw calls, 100–150k tris, textures ≤1024², DPR ≤2.
  - Tap-to-move + auto camera (pitch 35–50°); joystick for K1 only; tilt as bonus only.
  - Hebrew only in the 2D HUD via projected anchors.
  - Pause the Canvas during 2D activities.
- `astrobot_joy_gamification_psychology.md`:
  - Astro Bot's toy-like philosophy, one verb per level, and rescue → hub thresholds (16/32/48/64/120).
  - Contrasting haptic textures; cheap failure.
  - **Conflict noted:** its "capsule machine with random cosmetics" contradicts 00 D2 (contents fixed at earn time), so spec 09 has no random capsule.
- `learning_through_movement.md`:
  - 1–3 min bursts; seated equivalents; iPad on a stand 2–3 m away; never hold the device while jumping.
  - 2×2 m clear space; grippy socks or barefoot on tiles; quiet mode with no jumps.
  - Camera-on indicator; toddler supervised and imitation-only.

## R10. Touch-target conversions (computed)
- pt per mm = (ppi / scale) / 25.4:
  - iPhone 6.1"/Pro/Pro Max (460 ppi @3x): **6.04 pt/mm**
  - iPhone 12/13 mini (476 @3x): 6.25
  - iPhone SE 2/3 (326 @2x): 6.42
  - iPad/Air/Pro (264 @2x): **5.20**
  - iPad mini (326 @2x): **6.42**
- This confirms the gap_audit correction: 20 mm = 121 pt on iPhone and 104 pt on iPad. Note that **iPad mini behaves like an iPhone** (128 pt for 20 mm). Sizes must be computed per device model, not per idiom.

---

## Re-verify at build time
1. The iOS CoreText nikud render of Noto Sans Hebrew, Rubik and Fredoka (GATE-UX-2 specimen page).
2. Reanimated 4 `withSpring` accepts `{mass, stiffness, damping}` and the duration semantics.
3. Rive Data Binding API in `@rive-app/react-native` on SDK 58.
4. ASWG loudness numbers.
5. RN 0.88 RTL bugs listed in R6.
6. expo-haptics on iPad is a silent no-op, not an error.
7. GameController / DualSense support path in Expo, if adopted (spec §16).
