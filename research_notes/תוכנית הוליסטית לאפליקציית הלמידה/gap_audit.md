# Gap Audit: Contradictions, Ranked Gaps, Gap-Closing Research and New Concerns Across the 15 Research Notes

Scope: a critical review of all 15 notes in `research_notes/אפליקציית למידה משחקית לילדים/`, written for the holistic plan. The product is a Hebrew-first, gamified, personalized learning game for one Israeli family (girl 8.5 in grade 3, boy 5.5 in gan chova, toddler 2.5) on iPhone and iPad.

Method note: the four top-level sections below are the "key questions" the coordinator asked for. Citations to other notes use the form [note: file §N] and point to the sibling folder. Web findings use ordinary links. About 19 web calls were made in this session. WebFetch failed with DNS errors on pop.education.gov.il, www.healthychildren.org and depts.washington.edu, which are outside this session's network allowlist. Findings from those sources therefore come from search-engine extracts only. To read the full documents, those hosts have to be allowed in the session's network settings. Gap-closing deliberately avoided topics that other researchers are covering: placement; Hebrew TTS/STT/LLM, bilingual pedagogy and Israeli culture; iOS architecture; risks and failure modes; and content production, OER and phygital.

---

## 1. Contradictions & reconciliations

### Takeaway
The notes agree on the ethics (no ads, no paid randomness, no guilt, no sibling ranking, all purchases behind a parent gate). They contradict each other on the reward economy, success-rate targets, session lengths, streaks, celebration timing, praise wording, the tech stack, live versus pre-generated AI, photo use, and several smaller numbers. One numeric error also appears in two notes: the mm-to-pt conversion for touch targets. Every contradiction below has a proposed reconciled position, and the plan should adopt these as single source-of-truth defaults.

### Cited Findings
**C1. Currency and economy: "no currency" vs. "coins + gems + chests + shop"**
- Psychology note: "**No currency** (or, if any, non-purchasable, non-gambled, spent on creative self-expression like decorating a room)". Also "No 'do X to get Y' announcements". Also "no rarity tiers, no gacha, no duplicates", and "show a skills tree/garden growing rather than a coin balance". Tangible-like in-app rewards (coins, stickers) are "functionally tangible for a 5-year-old" [note: child_psychology_motivation.md §1, §7A].
- Economy note: coins earned "per mastery event", rare gems, chests, a "small rotating daily shop (3 to 4 items, Fortnite and Brawl Stars style)", crafting, and a parent "reward store" priced in coins ("trip to the park = 500 coins") [note: coop_economy_ios.md §2].
- Learning-science note: "stickers and coins mark effort and progress… unpredictable and celebratory rather than a per-answer payment" [note: learning_science_workbooks.md §7G-25].
- Deci, Koestner & Ryan 1999 (128 experiments). Engagement-, completion- and performance-contingent rewards reduced free-choice intrinsic motivation (d = −0.40, −0.36, −0.28). Positive verbal feedback raised it (d = +0.33 free-choice, +0.31 self-report). Tangible rewards were *more* detrimental for children than for college students, and verbal rewards were *less* enhancing for children — [PubMed abstract](https://pubmed.ncbi.nlm.nih.gov/10589297/); [paper PDF mirror](https://leeds-faculty.colorado.edu/dahe7472/deci%201999.pdf). Note: the psychology note lists completion-contingent as −0.44 and engagement-contingent as −0.40. The search extract gives completion-contingent as −0.36 on free choice. The table values still need a check against the full text.
- Cameron, Banko & Pierce (2001) argue that negative effects show up mainly when rewards are tangible, *expected* and loosely tied to performance on high-interest tasks — [summary via search of Deci et al. sources](https://pubmed.ncbi.nlm.nih.gov/10589297/).

**C2. Randomness inside chests**
- Economy note: chests may be random within a tier, with odds shown to the 8-year-old ("1 in 10 chests has a golden hat") [note: coop_economy_ios.md §2].
- Hit-games note: "Escalating anticipation animation for a guaranteed earned item" and avoid "random value… odds" [note: hit_games_engagement.md §7 matrix]. Psychology note: "no rarity tiers, no gacha" [note: child_psychology_motivation.md §7A-3].

**C3. Rarity tiers**
- UI note: rarity colors "common gray/green → rare blue → epic purple → legendary gold" give the 8.5-year-old "Brawl Stars status language" [note: ui_ux_polish_craft.md §5]. Hit-games note: "rarity reflects effort/mastery" [note: hit_games_engagement.md §1, §7]. Psychology note bans rarity tiers [note: child_psychology_motivation.md §3, §7A].

**C4. Real-world rewards**
- Economy note: a parent "reward store" priced in coins [note: coop_economy_ios.md §2]. Personalization note: "the parent sets goals… and a real reward (park trip, ice cream)" [note: personalization_tech.md, dashboard section]. Dashboard note: "Rewards are in-app cosmetics or a family activity, not money. Keep rewards tied to effort or process, not to accuracy" [note: parent_dashboard_analytics.md §4]. Psychology note: expected tangible rewards for an enjoyed activity are "the riskiest" [note: child_psychology_motivation.md §1].

**C5. Target success rate**
- Teardown note: about 75–80% (Elo) [note: kids_edu_apps_teardown.md §6, §11]. Math note: about 70–85% [note: math_pedagogy.md §6]. Curriculum-engine note: 70–85%, with item selection at about 75% [note: extra_subjects_adaptive_curriculum.md §B2, §B3]. Psychology note: 70–85% "heuristic, not sourced" [note: child_psychology_motivation.md §3]. Learning-science note: age-graded bands of 90–95% (2.5), 85–90% (5.5) and 75–85% (8.5), plus a challenge dial of 90/80/70 [note: learning_science_workbooks.md §7A-1].
- The anchors behind these numbers: Wilson et al. 2019 (≈85%, in machine learners); Rosenshine (≈80% in guided practice); Math Garden's 75% default with 60/75/90 options; and Jansen et al. 2013, where the easier condition (≈90%) led children to practice more [note: learning_science_workbooks.md §3, §7 anchors].

**C6. Session length**

| Source note | 2.5 | 5.5 | 8.5 |
|---|---|---|---|
| child_psychology §4 table | 3–5 min | 10–15 min | 15–25 min |
| child_psychology §6 "health limits" | ≤10–15 min | ≤15–20 min | ≤20–30 min |
| learning_science §4 / §7E | 3–5 min (cap ~5) | 8–12 min (cap ~12–15) | 15–20 min (cap ~20–25) |
| extra_subjects §B2 | 5–8 min | 10–15 min | 10–15 min |
| math_pedagogy §6 | — | 10–15 min/day | 10–15 min/day |
| ai_literacy §Q5 | ~3 min | — | — |

The psychology note contradicts itself (3–5 min vs. ≤10–15 min for the toddler). All the sources agree that their numbers are unvalidated heuristics [note: learning_science_workbooks.md §4 Gaps].

**C7. Streaks**
- Psychology note: weekly target, no loss framing, no streaks for the toddler [note: child_psychology_motivation.md §7C]. Teardown note: weekly 4-of-7 goal, Shabbat freeze, optional family streak [note: kids_edu_apps_teardown.md §11]. Math note: "habit streaks" [note: math_pedagogy.md §6]. EFL note: "short daily sessions with a streak" [note: english_efl.md §7]. UI note: for the 8.5-year-old, "streaks, collections, light competition with siblings" [note: ui_ux_polish_craft.md §6]. The hit-games "streak ladder" is a different thing, an *in-session* combo of rising notes [note: hit_games_engagement.md §4].

**C8. Sibling competition**
- Psychology note: hard rule of no side-by-side scores and no leaderboards [note: child_psychology_motivation.md §5]. UI note: "light competition with siblings (co-op preferred)" for the 8.5-year-old [note: ui_ux_polish_craft.md §6]. Economy note: "If the siblings do compete, normalize scores" [note: coop_economy_ios.md §1]. Teardown note: competition "opt-in, ages 8+" [note: kids_edu_apps_teardown.md §11].

**C9. Celebration duration**
- UI note: `celebrate.total ≤ 2.5 s` *per correct-answer celebration (repeated dozens of times a session)*; level-up 4–6 s [note: ui_ux_polish_craft.md §2]. Hit-games note: micro (per answer) <0.5 s; small (end of round) ~1.5 s; medium 2–3 s; large 4–6 s [note: hit_games_engagement.md §4]. Teardown note: celebrations "short (2–3 s)" [note: kids_edu_apps_teardown.md §8].

**C10. Praise wording**
- Psychology note bans superlatives ("incredible", "perfect", "best") and trait labels ("smart", "genius") [note: child_psychology_motivation.md §2, §7B]. Hit-games note proposes escalating Hebrew voice lines "!יופי", "!מדהים" (amazing), "!אלוף" (champion), modeled on Candy Crush's "Sweet/Tasty/Delicious/Divine" [note: hit_games_engagement.md §4]. "!אלוף" is a person/trait label and "!מדהים" is a superlative.
- Learning-science note: "Mistakes help your brain grow" as an explicit message [note: learning_science_workbooks.md §7G-28]. Psychology and world-leading notes: growth-mindset messaging has small effects; "keep it light", don't build around it [note: child_psychology_motivation.md §2; world_leading_and_ai_pedagogy.md §2].

**C11. Mastery definition and review ladder**
- Learning-science note: BKT ≥0.95, or ≥85% of the last 8 over ≥2 days with the last 3 hint-free. Ladder 1/3/7/14/30/60 days; a lapse demotes two rungs [note: learning_science_workbooks.md §7B-7, §7C-10].
- Curriculum-engine note: Khan ladder, Mastered needs a delayed mixed checkpoint ≥1 day later. Leitner 1/2/4/8/16 [note: extra_subjects_adaptive_curriculum.md §B1, §B3].
- Dashboard note: mastery score ≥90 plus a correct delayed review ≥3 days later; decays to "needs review" after 21 days [note: parent_dashboard_analytics.md §6].
- Photo note: +1d, +3d, +7d [note: photo_to_game_pipeline.md §Q3, §Q5].

**C12. Frustration and difficulty-drop rules**
- Learning-science note: after 2 consecutive errors drop a step and hint; after 3 on a skill switch skill or show a worked example; "never let a 2.5- or 5.5-year-old fail more than 3 times in a row" [note: learning_science_workbooks.md §7A-4]. Curriculum-engine note: trigger at "<~60% over the last 5–8 items" [note: extra_subjects_adaptive_curriculum.md §B2]. Psychology note: hint after 2 tries, auto-solve after 3 [note: child_psychology_motivation.md §7B]. Photo note: *raise* difficulty with "more distractors, **a timer**, or typing" after 2 correct [note: photo_to_game_pipeline.md §Q3].

**C13. Timers**
- Every other note bans visible countdowns, or allows them only as an opt-in "beat your own time" for age 8+ [note: math_pedagogy.md §4; learning_science_workbooks.md §7A-6; child_psychology_motivation.md §7F]. The photo pipeline's automatic timer escalation contradicts this (C12).

**C14. Response time in the Elo update**
- Curriculum-engine note: the Math Garden score S = (2·correct − 1)·(1 − rt/deadline) is applied generically [note: extra_subjects_adaptive_curriculum.md §B3]. Learning-science note: "ignore RT for scoring under age 6" because motor speed confounds it [note: learning_science_workbooks.md §3, §7A-6]. Dashboard note: RT only for the 8.5-year-old after proficiency [note: parent_dashboard_analytics.md §3].

**C15. Tech stack**
- Personalization note: a "PWA (or plain web app) with a web game library is the lowest-friction path", with IndexedDB and Supabase/Firebase [note: personalization_tech.md, platform section, explicitly "NOT verified"]. Economy/iOS note: native SwiftUI + SpriteKit, CloudKit; "A web PWA loses Pencil Scribble integration, haptics, Game Center and SharePlay" [note: coop_economy_ios.md §3]. UI and photo notes assume native SwiftUI, Core Haptics, VisionKit and Rive [note: ui_ux_polish_craft.md §2; photo_to_game_pipeline.md §Q4].

**C16. Live AI vs. pre-generated content**
- Personalization note: "generate content offline, in batches, at authoring time… Children never chat with a live LLM" [note: personalization_tech.md, AI section].
- World-leading note: a live "Why? Owl" Socratic companion and "voice AI with tightly scripted Socratic turns" for the 5.5-year-old [note: world_leading_and_ai_pedagogy.md §6].
- AI-literacy note: a server-side sandboxed LLM with chip inputs and output moderation [note: ai_literacy_for_kids.md §Q4].
- Photo note: live server VLM calls per photo [note: photo_to_game_pipeline.md §Q4].

**C17. Companion persona**
- Psychology note: "one consistent, warm companion"; a parasocial bond improves learning (Calvert) [note: child_psychology_motivation.md §3].
- AI-literacy note: the helper should be a robot with visible gears that "doesn't have feelings", "not a human-like friend persona" [note: ai_literacy_for_kids.md §Q2].
- World-leading note: "No anthropomorphic friend persona" [note: world_leading_and_ai_pedagogy.md §3, §6-13].

**C18. Photos of family and faces**
- Personalization note: "stylized avatars from family photos" using Gemini image editing [note: personalization_tech.md]. UI note: "Avoid any photo or face-scan feature" for avatars [note: ui_ux_polish_craft.md §5]. EFL and teardown notes: family-photo peekaboo and TinyTap-style parent photo games [note: english_efl.md §7; kids_edu_apps_teardown.md §10].

**C19. iPhone orientation**
- Economy/iOS note: "iPhone: one-handed, **portrait**, bottom-third controls" [note: coop_economy_ios.md §3]. UI note: "iPhone: **landscape-first** for game screens (like Brawl Stars)" [note: ui_ux_polish_craft.md §6].

**C20. Toddler solo vs. co-play**
- Teardown note: digital toys where "children explore… without a parent"; kid-safe navigation "meant to work without supervision" [note: kids_edu_apps_teardown.md §4, §7]. Psychology note: "toddler mode should ideally *require* lap-play" [note: child_psychology_motivation.md §6]. AI-literacy and EFL notes: "always with a parent" [note: ai_literacy_for_kids.md §Q5; english_efl.md §6].

**C21. Nikud for the 8.5-year-old**
- Teardown note: "8.5 → texts without nikud" [note: kids_edu_apps_teardown.md §10]. Personalization note: "8.5: reading fluency without nikud" [note: personalization_tech.md]. Hebrew note: grade 3 is *consolidating pointed fluency*, with unpointed work starting mid-grade-3 [note: hebrew_literacy.md §1, §6].
- New Ministry evidence (Section 3): the grade-3 fluency tool assesses at start and end of year, with *both* pointed and unpointed versions at the end of grade 3. The unpointed tool was "still in development" — [Ministry fluency assessment docs (search extract)](https://meyda.education.gov.il/files/Mazkirut_Pedagogit/HebrewPrimary/assessment/fluency-assessment-2021.pdf).

**C22. English for the 5.5-year-old**
- EFL note: oral only, "formal decoding is probably premature while Hebrew reading is also just starting" [note: english_efl.md §3]. Personalization and dashboard notes list "English letters" in the 5.5 mastery map [note: personalization_tech.md; parent_dashboard_analytics.md §6 lists EFL only at 8.5].

**C23. Garden and fire "decay" visuals**
- Learning-science note: plants "droop when a review is due" [note: learning_science_workbooks.md §7C-13]. Economy note: "the fire dims; tomorrow it needs more wood" [note: coop_economy_ios.md §1]. Hit-games note: "Plants must never wither as punishment" [note: hit_games_engagement.md §2]. Psychology note: growth "rests" instead [note: child_psychology_motivation.md §7C].

**C24. Rotating daily shop**
- Economy note: rotating 3–4 item daily shop with a wishlist [note: coop_economy_ios.md §2]. Hit-games matrix: avoid "limited-time exclusive items that vanish" and Fortnite item-shop rotations [note: hit_games_engagement.md §3, §7].

**C25. Touch-target sizes, plus a unit error**
- Psychology note: ≥2–3 cm (2.5), ≥1.5–2 cm (5.5), ≥1 cm (8.5) [note: child_psychology_motivation.md §4]. Economy/iOS note: "≥ 15 to 20 mm (roughly 60 to 80 pt)" [note: coop_economy_ios.md §3]. UI note: 2.5 "≥ 2 cm, about 75–90 pt on iPhone / 90–120 pt on iPad"; 5.5 60–70 pt; 8.5 ≥44–56 pt [note: ui_ux_polish_craft.md §6].
- **The conversions are wrong.** iPhone 16 is about 461 ppi at @3x, which is ≈153.7 pt per inch, so 1 pt ≈ 0.165 mm. iPad (11th gen) is 264 ppi at @2x, which is 132 pt per inch, so 1 pt ≈ 0.192 mm — [PhoneArena iPhone 16 vs iPad 11th gen specs](https://www.phonearena.com/phones/compare/Apple-iPhone-16,Apple-iPad-11th-Gen-A16-2025/phones/12240,12656); [iosref scale table](https://iosref.com/res). On that basis, 2 cm ≈ **121 pt on iPhone** and ≈ **104 pt on iPad**, not 75–90 pt. And "15–20 mm ≈ 60–80 pt" is really ≈ 91–121 pt on iPhone. The derivation is my own arithmetic from the published ppi figures.

### Inferences
Each reconciled position below is proposed as the plan's default (D-number = decision).

- **D1 (economy, C1/C4/C24).** Build a **minimal, single-currency, mastery-paid, cosmetic-only economy for the 5.5 and 8.5 profiles only, with no announced per-task price.**
  - **One soft currency** (call it "כוכבים/stars" or a themed item). It is paid out by the system at *mastery events*: a skill leveling up, a review passed after a gap, finishing an episode. It is never paid per correct answer, and never pre-announced as "do 5 to get 10."
  - **No second "gem" currency.** Rare unlocks come from deterministic milestones on the map, which makes a second currency unnecessary. Two currencies plus XP plus stickers plus pets plus a garden plus a campfire plus a trophy road is systems bloat for a solo builder and for a 5-year-old.
  - **Spending is only on self-expression and the shared family space.** That matches the psychology note's own "if any" exception.
  - **Toddler: no currency at all.** Sensory feedback is the reward, which all notes agree on.
  - **Real-world rewards are not priced in coins.** If parents want real rewards, the dashboard offers *family-activity* rewards tied to *effort/consistency goals*: "3 adventure days this week → we bake together." This follows the dashboard note and the Cameron/Deci shared ground (expected tangible rewards are riskiest).
  - **Shop: a fixed catalog** that grows by unlocking (new "arrivals" appear permanently, nothing leaves). A wishlist and save-up jar provide the financial-literacy layer. There is no daily rotation, because rotation is a daily-return lever even with a wishlist.
- **D2 (chests and rarity, C2/C3).** **Contents are deterministic, and "rarity" encodes effort, not chance.** A chest's contents are fixed at the moment it is earned and shown as a silhouette before opening. The reveal ritual (wobble → burst → item) is kept, because the ritual is the fun, not the odds. Tier colors map to *what earned it*: a bronze frame for an episode, silver for a skill mastered, gold for a unit or boss. Use frame shape plus color for color-blind safety. No odds are displayed, because there are none.
- **D3 (success rates, C5).** Adopt the **age-graded bands**: about 90–95% (2.5, which is mostly recognition with no fail state), about 85% (5.5) and about 75–85% (8.5). The 8.5-year-old gets the 90/80/70 challenge dial. The Elo "75%" defaults in the teardown and curriculum-engine notes are the *8.5 default*, not a family-wide target. Rationale: Jansen 2013 (easier → more practice) is the only child RCT among the anchors, and Wilson's 85% is from machine learners.
- **D4 (session length, C6).** Defaults are **2.5: 3–5 min episodes, co-played, at most about 2/day. 5.5: 8–12 min. 8.5: 15–20 min.** These are *episode* lengths. Parent-adjustable daily caps sit above them: 2.5 ≤15 min, 5.5 ≤25 min, 8.5 ≤40 min, all well within the 2–5 "≈1 h total" guidance still quoted by secondary coverage. The psychology note's "≤10–15 min per session" for the toddler should be treated as a *daily* ceiling, not an episode length.
- **D5 (streaks, C7).** **No daily streak for anyone.** Use a **weekly family goal** (for example, the family garden grows on any 3–4 days per child per week). Shabbat and holidays are auto-rest days. Celebrate returns rather than consecutive days. The in-session "combo ladder" of rising notes is allowed: it is feedback, not a habit lever.
- **D6 (sibling competition, C8).** **No sibling competition of any kind visible to children by default.** The 8.5-year-old's competitive appetite is met by ghost races against her own past self and by co-op bosses. A parent-enabled "family challenge" might be considered later, but only one that is *team vs. the game*.
- **D7 (celebration timing, C9).** Adopt the hit-games tiers. **Per-answer feedback is ≤0.5–0.8 s and non-blocking**: the next item can load underneath. End-of-round is ~1.5 s, a new item 2–3 s, and milestones 4–6 s, rare and tap-to-skip. The UI note's 2.5 s per correct answer would eat about 50 s of a 5-minute session if there were 20 items (my arithmetic). That hurts the "learning seconds per minute" metric the teardown note itself recommends [note: kids_edu_apps_teardown.md §3].
- **D8 (praise, C10).** **Process-praise scripts only.** Replace "!אלוף/!מדהים" with escalating *descriptive* lines: "!יופי של ניסיון", "!מצאת את הדרך", "!שלוש ברצף", "!בדקת וצדקת". Keep escalation in the *music and visuals*, not in trait labels. Growth-mindset phrases are allowed occasionally as light encouragement, not as a recurring doctrine.
- **D9 (mastery and review, C11).** Use **one definition everywhere.** That covers the engine, the dashboard and the photo SRS.
  - "Mastered" means ≥85% on the last 8 first attempts, hint-free on the last 3, across ≥2 days, **and** a correct delayed check ≥3 days later.
  - The review ladder is **1, 3, 7, 14, 30, 60 days**. The photo pipeline's +1/+3/+7 is just the first three rungs.
  - A lapse demotes one rung, or two after a ≥30-day rung.
  - "Needs review" in the dashboard means "a due review has been failed" (or the item is overdue past 1.5× its interval), not a fixed 21 days.
- **D10 (frustration, C12/C13).** Use **one frustration guard**: 2 consecutive errors → hint and a step down; 3 → worked example or a switch of activity; under 6, never more than 3 misses in a row. For the photo pipeline, difficulty rises by more distractors or typing, **never an automatic timer**. Timers exist only as an 8.5 opt-in "speed round" after mastery.
- **D11 (RT, C14).** Response time is **off in the Elo update for the 2.5 and 5.5 profiles**. For the 8.5 profile it is on only for fact-fluency skills, after accuracy mastery.
- **D12 (stack, C15).** The PWA recommendation is the outlier. It was explicitly unverified, and it conflicts with the brief (iPhone/iPad, extreme polish, Pencil, haptics). **Native SwiftUI + SpriteKit/Rive is the default**, pending the iOS-architecture researcher's findings. The personalization note's bidi argument for the web is answered by SwiftUI's per-view `layoutDirection` [note: coop_economy_ios.md §3].
- **D13 (AI tiers, C16/C17).** Use **three AI tiers**:
  - (a) **Authoring time** (most content): stories, item variants and audio are generated by the parent's pipeline, parent-reviewed, and shipped as static data.
  - (b) **Bounded runtime**: the photo pipeline and a chip-input AI-literacy sandbox. These are server-side, moderated, parent-approved before play, with no free text for under-8s.
  - (c) **No open-ended chat** for any child. The "Why? Owl" becomes a *scripted* Socratic character with branching and pre-written hint ladders. A live LLM may be used only in the parent-present "Grown-up Co-pilot" mode for the 8.5-year-old.
  - On persona: keep **two distinct character types**. The warm mascot cast is scripted content characters, where parasocial warmth is fine because there is no AI pretending to be a person. The AI helper is a clearly labelled machine, a robot with gears that makes mistakes on purpose. Never merge them.
- **D14 (photos, C18).** Family photos may be used **on-device as content**: peekaboo, "who is this?" and vocabulary. They are **never sent to an image model to make avatars**. Avatars come from Mii-style parts. The only cloud photo flow is homework or worksheets, with on-device face blur and crop first.
- **D15 (orientation, C19).** **iPad landscape is the primary device for game and co-op screens.** On iPhone, game screens are landscape (matching Brawl Stars muscle memory) and the hub and map are portrait-agnostic. This supports one-handed portrait only for quick 8.5 review "snacks." Pick one per screen type and document it.
- **D16 (toddler, C20).** The toddler space is a **no-fail "playroom" that is safe to use alone for short stints**: it has no exits, no text and no fail states. The *learning episodes* for the toddler (counting, words) are **co-play-first**, with parent prompts on screen. Re-entry after an episode ends needs the parent gate.
- **D17 (nikud, C21).** For the 8.5-year-old, **pointed text is the default through mid-year.** Nikud-fade and unpointed tasks ramp from about February, matching the Ministry's mid-grade-3 assessment timing. Both scripts are measured at year-end, mirroring the Ministry tool.
- **D18 (English for the 5.5-year-old, C22).** **Oral English only** (songs, TPR, listen-and-tap). Letters appear incidentally. Remove "English letters" from the 5.5 dashboard mastery map.
- **D19 (decay visuals, C23).** **No dimming, drooping or wilting.** A due review shows as a *positive* cue: a bud ready to bloom, or a glowing "ready" sparkle. Absence shows a resting, sleeping state that wakes happily.

### Gaps
- The Deci et al. 1999 table values for children vs. college students and for expected vs. unexpected rewards were not retrieved, because the full-text PDF host was blocked. The two notes disagree slightly on completion-contingent d (−0.44 vs. −0.36). Someone should read Table 2 of the PDF directly.
- No source resolves the session-length question empirically. D4 is still a judgment call.

---

## 2. Top gaps ranked

### Takeaway
Across the 15 notes' Gaps sections, the gaps that most threaten the product are platform capability gaps: Hebrew speech, handwriting and OCR on Apple. These are being covered elsewhere. The highest-impact gaps that are *not* covered elsewhere are: the exact Israeli grade-3 curricula, especially the brand-new math program this year; the Hebrew letter-order and workbook conventions the kids actually meet in class; the evidence on digital points and coins for young children; and the sibling co-play evidence. The ranking below scores each gap by how many design decisions depend on it and how costly a wrong guess would be.

### Cited Findings
Ranked list. "Owner" says whether another researcher is covering it. Impact: H = high, M = medium, L = low.

| # | Gap (where it appeared) | Impact | Owner / status |
|---|---|---|---|
| 1 | Hebrew support in Apple Speech (SpeechTranscriber, he-IL), Scribble/PencilKit, Vision OCR; Foundation Models Hebrew (coop_economy §3; photo §Q1; ui §4) | H: gates voice, tracing and photo features | iOS researcher |
| 2 | Child Hebrew ASR accuracy, no benchmark (english §5; hebrew §5, §7; personalization; world §6) | H: gates fluency sprints and "say it" | Hebrew TTS/STT researcher |
| 3 | **New grade-3 math curriculum content (5787 cohort)**: times-table mastery timing, 1,000 vs 10,000, fractions scope (math §1) | H: the 8.5-year-old is in the first cohort | **This audit, partially closed (§3)** |
| 4 | **Grade-3 Hebrew milestones**: unpointed timing, fluency norms, grammar topics (hebrew §1, §6) | H | **This audit, partially closed** |
| 5 | **Evidence on points, coins and badges for under-10s** (psych §1 Gaps; coop §2 Gaps) | H: decides the whole economy | **This audit, partially closed** |
| 6 | Placement validity for under-6s (extra §B2) | H | Placement researcher |
| 7 | **Hebrew letter-introduction order and the Israeli programs and workbooks in use** (hebrew §1, §3; learning_science §5) | M-H: content order for the 5.5-year-old | **This audit, closed negatively (no official order)** |
| 8 | **AAP 2026 statement specifics** (psych §6) | M: drives defaults and parent copy | **This audit, partially closed** |
| 9 | **Touch-target research by age; mm↔pt** (psych §4; coop §3; ui §6) | M: affects every screen | **This audit, closed with a correction** |
| 10 | **Sibling co-play and tutoring evidence** (psych §5; coop §1; english §7) | M: co-op is a headline feature | **This audit, partially closed** |
| 11 | Nikud font rendering in iOS fonts (ui §4) | M | Partly iOS researcher; open |
| 12 | Israeli Privacy Law Amendment 13 child provisions (psych §6; personalization; photo §Q6) | M if published, L for a family-only app | **This audit, partially closed** |
| 13 | **Radesky 2022 details** (psych §6) | L-M: the audit checklist exists already | **This audit, closed** |
| 14 | Supercell reveal timing; Duolingo sound process (hit §1, §4; ui §2, §3) | M for polish | **This audit, not closable from public sources** |
| 15 | Hebrew numeral gender in counting (math §1) | M for the 2.5/5.5 counting voice-over | **This audit, partially closed** |
| 16 | Novelty decay, transfer, long-term engagement (many notes) | H | Risks researcher |
| 17 | Optimal celebration duration and reward density by age (hit §4; psych §7) | M | Open, no literature found by anyone |
| 18 | Persistence of Math Garden's "garden/wilting" mechanic (teardown §6; learning §3) | L | Open, irrelevant after D19 |
| 19 | Grade 1–3 science and moledet unit lists; gan core curriculum (extra §A1) | M | Open (Ministry hosts blocked) |
| 20 | PEGI 2026 loot-box rating; Israeli loot-box law (hit §7) | L: the product has no paid randomness | Open, low priority |

### Inferences
- Gaps 1, 2 and 6 are the true "kill switches", but they are owned elsewhere. The plan should treat voice and tracing as *progressive enhancement* until they are confirmed.
- Gap 5 is high-impact because three notes built an economy on an assumption that the psychology note rejects. Section 3 narrows the risk but cannot eliminate it. The plan should therefore instrument intrinsic interest: free-choice play in the playroom, and opting into optional content with no reward.

### Gaps
- No gap list mentions **measuring learning**: pre/post probes and delayed retention, and how a family of 3 would detect any effect. That is probably in the risks researcher's scope (playtesting), so it is flagged and not researched here.

---

## 3. Gap-closing findings

### Takeaway
Targeted searches closed or narrowed nine gaps.
- **Math:** the new math curriculum really does reach grade 3 in 5787 (this year), but its grade-3 topic list was not retrievable.
- **Hebrew:** the Ministry's grade-3 fluency tool confirms the pointed→unpointed timing (mid-year and end of year, both scripts). There is **no official letter-introduction order**: programs and teachers choose.
- **AAP 2026:** the statement drops individual hour caps as the frame and targets *design*. The specific numbers in secondary coverage remain unconfirmed.
- **Gamification:** the most relevant meta-analysis finds the *smallest* gamification effects in primary school (g ≈ 0.31) and larger effects on extrinsic than intrinsic motivation.
- **Siblings:** children taught by older siblings outperformed those taught by older peers (Azmitia & Hesser 1993).
- **Touch targets:** two notes contain a pt-conversion error.
- **Radesky 2022, Amendment 13 and Hebrew numeral gender:** each got useful partial detail.

### Cited Findings
**G3. Israeli grade-3 math, new curriculum**
- The Ministry's math teachers' portal says that in 5786 (2025-26) grades 1–2 learned under the new curriculum and grades 3–6 stayed on the old one; "בשנת הלימודים תשפ"ז תלמידי כיתה ג' יצטרפו" (in 5787, grade 3 joins). The program aims at a balance between computational fluency and deep understanding, plus mathematical discourse and critical and creative thinking — [Ministry math curriculum page (search extract)](https://pop.education.gov.il/tchumey_daat/matmatika/yesodi/oraat-math/tohnit-limudim/).
- The Ministry publishes a "מסמך ציוני דרך" (milestones document) built on the curriculum. It lists topics, teaching hours, clarifications and examples, and is the authoritative source for grade-3 topics — [same page (search extract)](https://pop.education.gov.il/tchumey_daat/matmatika/yesodi/oraat-math/tohnit-limudim/).
- An August 2026 (5787) elementary guidance letter from the national math supervisor says the year continues implementation of the new curriculum, mentions the milestones document, and mentions the "ישראל ריאלית" program — [mafmar yesodi tashpaz PDF (search extract)](https://meyda.education.gov.il/files/Mazkirut_Pedagogit/math/primary-school/math2025/mafmar_yesodi_tashpaz.pdf).
- A grade-3 curriculum PDF exists at meyda.education.gov.il (`curriculum-third-grade.pdf`), but its content was not visible in search — [curriculum-third-grade.pdf](https://meyda.education.gov.il/files/Pop/0files/matmatika/Yesodi/pedagogya/curriculum-third-grade.pdf).
- Workbook evidence from school supply lists: one school's 5786 grade-3 list requires the "פלוס ג' שבילים" math set (four booklets) plus math and geometry manipulative kits — [school book list (search extract)](https://docs.google.com/document/d/13QwAXgtstmkVo5KexObGNNB4s_n_3D6W/export?format=pdf). The "פשוט חשבון" series carries Ministry approval stamps. The approved book found was for grade 5, with a grade-1 booklet hosted on the Ministry's free-books site — [פשוט חשבון on Ministry free-books](https://meyda.education.gov.il/files/free%20books/%D7%A4%D7%A9%D7%95%D7%98%20%D7%97%D7%A9%D7%91%D7%95%D7%9F%20%D7%953.pdf). "מסלולים" appears in the Ministry's approved-book price regulations, for grade 1 — [takanot 8266](https://meyda.education.gov.il/files/SifreyLimud/takanot-8266.pdf).

**G4. Grade-3 Hebrew milestones (fluency, pointed→unpointed)**
- The Ministry's grade-3 reading-fluency tool has a 45-second word-reading task plus oral reading of a short text. It is given at two points (beginning and end of year). At the end of grade 3 it comes in two script versions, pointed and unpointed. A mid-grade-3 text ("הכדור הקטן") is designated for February. A note says the tool for **unpointed** fluency was "still in development" — [Ministry fluency assessment docs (search extract)](https://meyda.education.gov.il/files/Mazkirut_Pedagogit/HebrewPrimary/assessment/fluency-assessment-2021.pdf); [end-of-grade-2 tool](https://meyda.education.gov.il/files/Mazkirut_Pedagogit/HebrewPrimary/assessment/fluency-assessment-end-of-classB.pdf).
- The documents say most of the move to skilled reading happens during grade 3, toward reading unpointed script. Some pupils struggle with that transition, so assessment on both text types is recommended for early intervention — [fluency-assessment-2021 (search extract)](https://meyda.education.gov.il/files/Mazkirut_Pedagogit/HebrewPrimary/assessment/fluency-assessment-2021.pdf).
- The documentation form records words per minute, reading time in seconds, and accuracy bands: **≤4 errors = medium-high; 5–9 = medium-low; ≥10 = low** — [same source (search extract)](https://meyda.education.gov.il/files/Mazkirut_Pedagogit/HebrewPrimary/assessment/fluency-assessment-2021.pdf). No numeric wpm targets were found.
- The Ministry also published a 2019 program to promote reading fluency in elementary school — [reading-fluency.pdf](https://meyda.education.gov.il/files/Mazkirut_Pedagogit/HebrewPrimary/program/reading-fluency.pdf).

**G7. Hebrew letter order and reading programs**
- No official Ministry letter-introduction order was found. Search results (including an older Ministry-adjacent report) describe the Ministry as opposing a single imposed teaching method — [Walla news](https://news.walla.co.il/item/90664); [Ynet](https://www.ynet.co.il/articles/0,7340,L-2730540,00.html). These are older news items without clear dates.
- "קסם וחברים" is described by a JCC partner document as a Ministry-approved systematic program in which all letters and the nikud signs (kamatz, patach, shva, chirik, chirik male) are learned in grade 1, in print and script in parallel, through play and creation — [JCC partner PDF](https://www.jccotp.org/wp-content/uploads/2024/05/1st-and-2nd-1.pdf). This is a secondary description, and no letter order is given.
- A RAMA-era (תשס"ב) document surveyed the common methods of its time and found that 34% of teachers did not document pupils' reading level at the start of grade 1 — [Ministry Hemed/RAMA doc (search extract)](https://meyda.education.gov.il/files/Hemed/yesodi/chol/tash1.pdf).
- The Ministry provides a literacy-profile tool used at the start of grade 1 to look at early literacy — [evaluation-tools.pdf](https://meyda.education.gov.il/files/Pop/0files/ivrit_hinuch_leshoni/Yesodi/evaluation-tools.pdf).

**G5. Points, coins and badges in young children**
- Kurnaz & Koçtürk (2025), *Psychology in the Schools* 62(12):4997–5009, a meta-analysis of gamification and K-12 motivation (31 studies, 41 comparisons). Overall g = 0.654 (I² = 88.9%). Effects were larger on **extrinsic** (g = 0.713) than **intrinsic** (g = 0.638) motivation. By grade band: secondary g = 1.015, high school g = 0.821, and **primary school g = 0.309 (the smallest)** — [Hacettepe record](https://research.hacettepe.edu.tr/en/publications/a-meta-analysis-of-gamifications-impact-on-student-motivation-in--2/); [search extract of DOI 10.1002/pits.70056](https://avesis.hacettepe.edu.tr/yayin/61244cfa-5cf5-4504-aa02-e82577965c19/a-meta-analysis-of-gamifications-impact-on-student-motivation-in-k-12-education).
- A 2024 Springer meta-analysis focused on **intrinsic** motivation (35 interventions, ~2,500 participants) found an overall g = 0.257 — [search extract](https://springerlink.bibliotecabuap.elogim.com/content/pdf/10.1007/s11423-020-09807-z.pdf). The exact citation was not confirmed; the link returned is to a related 2020 ETR&D paper.
- Huang et al. 2020 (ETR&D) found an overall g = .464 across 14 gamification design elements on learning outcomes — [ETR&D PDF](https://springerlink.bibliotecabuap.elogim.com/content/pdf/10.1007/s11423-020-09807-z.pdf).
- None of these isolated points or badges for under-8s. The search found no such study.

**G8. AAP 2026 policy statement**
- "Digital Ecosystems, Children, and Adolescents" was released 20 Jan 2026 in *Pediatrics* (Feb 2026), DOI 10.1542/peds.2025-075320, with a technical report. It replaces earlier AAP media guidance — [AAP news release](https://www.aap.org/en/news-room/news-releases/aap/2026/american-academy-of-pediatrics-calls-for-creating-a-child-friendly-digital-world/); [Contemporary Pediatrics](https://www.contemporarypediatrics.com/view/aap-calls-for-system-level-changes-to-improve-children-s-digital-media-environments).
- It uses a socioecological model and says media use "cannot be viewed solely through the lens of individual child behaviors or screen limits alone". **Platform design is the central issue**: engagement-driven vs. child-centered design. Designs that "prioritize engagement and commercialization often encourage prolonged use, which can displace movement and sleep" — [AAP News "Beyond screen time"](https://publications.aap.org/aapnews/news/34088/Beyond-screen-time-Policy-discusses-how-to); [Children and Screens summary](https://www.childrenandscreens.org/newsroom/news/digital-ecosystems-children-and-adolescents-policy-statement/).
- For families: a family media plan, modeling, media-free spaces (meals, car), and the "5 Cs" (child, content, calm/self-regulation, crowded-out activities, communication). For industry: safety and privacy by default, limited data collection and targeted ads, transparent algorithms. Age groups are 0–5, 6–12 and 13–18 — [HealthyChildren.org summary (search extract)](https://www.healthychildren.org/English/news/Pages/creating-a-child-friendly-digital-world-AAP-releases-new-media-recommendations.aspx); [AAP News](https://publications.aap.org/aapnews/news/34088/Beyond-screen-time-Policy-discusses-how-to).
- Strong caregiver–child relationships are associated with positive digital experiences — [AAP News](https://publications.aap.org/aapnews/news/34088/Beyond-screen-time-Policy-discusses-how-to).
- **Unconfirmed:** secondary coverage cites "under one hour a day for toddlers and preschoolers". No search result confirmed this, or any 18-month threshold or rule on autoplay or rewards, from the statement itself.

**G9. Touch targets**
- Children's touches missed targets 46% of the time vs. 32% for adults (Anthony et al.), as already cited [note: coop_economy_ios.md §3]. NN/g recommends ≥2 cm × 2 cm for young children [note: ui_ux_polish_craft.md §6].
- Shaw & Anthony, "Toward a Systematic Understanding of Children's Touchscreen Gestures" (CHI 2016 EA, pp. 1752–1759), is about gestures, not target sizing. No mm-level per-age recommendation was found in any source — [researchr record](https://researchr.org/publication/ShawA16).
- Unit conversion facts: iPhone 16 is 461 ppi (@3x) and iPad 11th gen is 264 ppi (@2x), so 1 pt ≈ 0.165 mm on iPhone and ≈ 0.19 mm on iPad — [PhoneArena](https://www.phonearena.com/phones/compare/Apple-iPhone-16,Apple-iPad-11th-Gen-A16-2025/phones/12240,12656); [iosref](https://iosref.com/res).

**G10. Sibling tutoring**
- Azmitia & Hesser (1993), *Child Development* 64:430–444 (DOI 10.2307/1131260):
  - Younger children watched, copied and questioned older **siblings** more than older **peers**, and siblings offered more spontaneous guidance.
  - In structured teaching, siblings gave more explanations and positive feedback and handed over more control.
  - Children taught by siblings scored **higher on posttests** than those taught by peers.
  - Younger children actively elicited explanations.
  - [Abstract mirror](https://data.zhenzhi.com/details/1/1004950058).

**G13. Radesky 2022 details**
- JAMA Network Open 5(6):e2217641 (June 2022), apps used by 160 children aged 3–5. Four in five apps had manipulative design, and about 99% of children had at least one in a top-used app. Examples of parasocial pressure: a character crying when a child skipped a free-trial prompt, and "Don't just stand there, do something!" when idle. Countdown clocks at pauses. Lures: stickers, trophies and **accumulating daily rewards**. A 20-second pop-up with a hidden close button. Lower-SES children had more exposure — [Michigan Medicine](https://www.michiganmedicine.org/health-lab/design-tricks-commonly-used-monetize-young-childrens-app-use); [EurekAlert](https://www.eurekalert.org/news-releases/956153); [Deceptive Patterns summary](https://deceptive.design/articles/prevalence-and-characteristics-of-manipulative-design-in-mobile-applications-used-by-children).

**G12. Israel Amendment 13 and minors**
- Amendment 13 took effect 14 Aug 2025 — [IAPP](https://iapp.org/news/a/israel-marks-a-new-era-in-privacy-law-amendment-13-ushers-in-sweeping-reform).
- Commentary says there is no dedicated child-consent age. Minors' consent runs through the general framework, and the Legal Capacity and Guardianship Law (1962) means parental consent governs — [EdLaw summary](https://edlaw.eduverse.com/docs/jurisdiction/nat/il/protection-of-privacy-law).
- **Data on minors counts as "specially sensitive."** Databases with such data on more than 100,000 people must notify the regulator — [Global Law Experts](https://globallawexperts.com/israels-privacy-reforms-bite-2026-expanded/).
- The Privacy Protection Authority has issued **draft** age-assurance guidance on a risk-based, proportionality approach — [same source](https://globallawexperts.com/israels-privacy-reforms-bite-2026-expanded/). These are law-firm and aggregator summaries, not the statute.

**G15. Hebrew numerals and counting**
- Research documents neutralization of gender distinctions in Modern Hebrew numerals: speakers reanalyze numeral suffixes and agreement is disappearing — [Language Variation and Change](https://www.cambridge.org/core/journals/language-variation-and-change/article/abs/neutralization-of-gender-distinctions-in-modern-hebrew-numerals/658F6A3A318E90D3A0D5FD27328E9729).
- Hebrew children aged 4–17 showed no mass/count sensitivity before about 7;2, vs. about age 4 for English learners. The authors link this to relaxed number-marking in cardinality contexts — [UvA record](https://dare.uva.nl/id/58e31300-d16a-4e14-94f7-7a27a9ce6002).
- No study was found directly on gendered counting words and cardinality acquisition.

**G14. Supercell reveal timing and Duolingo sound**
- No Supercell or GDC primary source on chest or Starr Drop reveal timing was found. The only design commentary is critical (Clash Royale's old chest timers of 3+ hours as a monetization lever) — [PocketGamer.biz](https://www.pocketgamer.biz/genius-why-clash-royale-is-for-losers).
- No official Duolingo sound-design write-up was found. Secondary commentary describes a "bouncy chime" for correct answers and a deflated descending tone for wrong ones — [Out of Scope newsletter](https://out-of-scope-product.beehiiv.com/p/duolingo-has-it-venmo-has-it-even-slack-has-it-your-product-doesn-t).

### Inferences
- **Grade-3 math.** Because the 8.5-year-old is in the *first* grade-3 cohort on the new program, older grade-3 workbooks and online worksheets may not match her class sequence. The game should not hard-code a grade-3 sequence from old materials. Use the parent's "this month in class" input [note: extra_subjects_adaptive_curriculum.md §B2], and the photo-to-game pipeline on her actual worksheets, as the alignment mechanism. Fetching the ציוני דרך document (needs pop/meyda allow-listing) is the single most valuable curriculum task left.
- **Hebrew fluency metric.** The game's fluency sprint should mirror the Ministry: a 45-second word list plus a short passage, wpm plus an error count, with the same error bands. Run it at start, mid-year (February) and end of year, and from mid-year in both pointed and unpointed scripts. This makes in-app numbers legible to the class teacher.
- **Letter order.** With no official order, the app should (a) offer a sensible default order, and (b) let the parent pick "the letter of the week from gan or class" to override it, the same way math works. The default should follow the Hebrew note's principles: sound-first, high-utility, acrophonic, confusables separated [note: hebrew_literacy.md §3, §7]. The boy is in gan chova, so formal letter order matters less than phonological awareness and name letters.
- **Economy.** The primary-school g ≈ 0.31 (vs. ~1.0 in middle school), and the larger extrinsic than intrinsic effect, support D1. Gamification is a modest lever at these ages. The extrinsic tilt is a warning, not a feature. Don't expect coins to carry engagement. The story, characters, creation and mastery map have to.
- **AAP.** Parent-facing copy should *not* claim "AAP says ≤1 hour" until the primary text is read. The AAP 2026 emphasis on design aligns with the plan's ethics checklist. The family media plan and media-free spaces suggest a feature: "app sleeps during meals and the car", which parents can set.
- **Touch targets.** The corrected mapping (proposed): 2.5 ≥2 cm ≈ **≥120 pt iPhone / ≥104 pt iPad**; 5.5 ≥1.5 cm ≈ ≥90 pt iPhone / ≥80 pt iPad; 8.5 ≥1 cm ≈ ≥60 pt iPhone / ≥52 pt iPad. Hit areas are larger than visuals. Implement sizes in *physical* units through a per-device pt-per-mm constant, not raw pt.
- **Sibling tutoring.** Azmitia & Hesser support making the 8.5-year-old the "guide" for the 5.5-year-old in co-op. It is the strongest direct evidence in the whole corpus for the sibling-teacher mode. Note the finding is about *task mastery in a lab building task* and about siblings' *style* (explanations, handing over control). The co-op design should prompt exactly those behaviors ("explain to Itai why", "let him try first").
- **Radesky.** "Accumulating daily rewards" are explicitly on the list of lures. This supports dropping daily login rewards and daily shop rotation (D1, D5).
- **Amendment 13.** For a family-only app the practical impact is low. If ever published, child data is "specially sensitive" under Israeli law, which reinforces on-device and CloudKit-private defaults.
- **Hebrew counting voice-over.** Gender agreement in numerals is unstable even among adult speakers. Two consequences: (a) the *recorded or TTS* counting lines must be correct and consistent, since Hebrew children default to feminine abstract counting (אחת, שתיים, שלוש); and (b) generated items like "שלושה כלבים / שלוש בובות" are a likely LLM and TTS error source. Count lines should come from a hand-checked lookup table, not free generation.
- **Reveal timing and sound.** Public sources won't supply these numbers. The plan should budget an in-house "frame-step study": record Brawl Stars Starr Drop and Duolingo lesson-end sequences on device, step frame by frame, and measure anticipation, burst and settle durations and the sound layers. Then set the tokens. That is legal analysis of timing, not copying assets.

### Gaps
- Grade-3 topics in the new math curriculum: still not retrieved. The ציוני דרך and curriculum-third-grade PDFs need allow-listing of meyda.education.gov.il and pop.education.gov.il.
- Hebrew wpm targets for grade 3, and the 2025 Hebrew curriculum's coverage of grade 3: not found.
- An official or common letter order in widely used grade-1 programs: not found.
- Points and badges for under-8s specifically: no study found.
- AAP 2026 numeric limits: unconfirmed.
- Deci et al. child-only and expected/unexpected values: not retrieved.
- Supercell and Duolingo primary timing and sound sources: none public.

---

## 4. New ideas & concerns not yet covered anywhere

### Takeaway
Several issues fall between the notes:
- Hebrew **grammatical gender in every child-facing line**.
- **Systems bloat** from 10+ overlapping progression systems.
- **Data starvation** of Elo/FSRS with three users.
- The **cumulative time cost of celebrations**.
- **Device-sharing fairness** between siblings.
- **Wrong pt math**.
- The new-curriculum cohort risk.
- A missing **evaluation plan**.

Each is concrete and would change the build.

### Cited Findings
- Hebrew numeral gender agreement is unstable in Modern Hebrew — [Language Variation and Change](https://www.cambridge.org/core/journals/language-variation-and-change/article/abs/neutralization-of-gender-distinctions-in-modern-hebrew-numerals/658F6A3A318E90D3A0D5FD27328E9729).
- The curriculum-engine note itself flags that item difficulty "won't calibrate from 3 kids alone" and that HLR/FSRS may never beat Leitner with 3 users [note: extra_subjects_adaptive_curriculum.md §B3 Gaps].
- The teardown note recommends tracking "learning seconds per session minute" [note: kids_edu_apps_teardown.md §3].
- Primary-school gamification effects are the smallest by grade band (g = 0.309) — [Kurnaz & Koçtürk 2025](https://research.hacettepe.edu.tr/en/publications/a-meta-analysis-of-gamifications-impact-on-student-motivation-in--2/).
- Perceived differential treatment between siblings is linked to lower self-worth, and fairness is the moderator [note: child_psychology_motivation.md §5].

### Inferences
1. **Gendered Hebrew address (high impact; not raised in any note).**
   - Hebrew imperatives and many praise lines are gendered: "לחץ/לחצי", "נסה/נסי", "אתה/את", "מוכן/מוכנה".
   - With a girl and a boy, every scripted line, TTS clip, LLM prompt and template needs per-child gender variants. An option for neutral phrasing (plural or infinitive: "בואו ננסה", "אפשר ללחוץ") also helps.
   - Getting this wrong (addressing the girl in masculine forms, which is the default in many Hebrew apps) is a constant, noticeable irritant and a gender-parity issue that ties to the Gunderson process-praise concern [note: child_psychology_motivation.md §2].
   - Store a `gender` field per profile, write scripts in a variant format (`{m: "…", f: "…"}`), and QA every audio line twice.
   - The Israeli-culture researcher may touch on this. It is flagged here because it touches every content pipeline (photo, LLM, TTS).
2. **Hebrew counting lines from a lookup table.** Use a hand-verified table of numeral + noun-gender forms (1–20, plus tens) for all counting voice-over and generated math stories. Do not trust the LLM or TTS for "שלושה/שלוש".
3. **Systems bloat (high impact for a solo builder).** Across the notes there are proposals for:
   - coins
   - gems
   - XP
   - stars per round
   - stickers
   - an album with silhouettes
   - evolving pets
   - a skill garden
   - a family campfire
   - a family village or island
   - Trophy Road
   - a season album
   - a savings jar
   - a reward store
   - a wardrobe

   That is about 15 systems. Recommendation: consolidate into **three visible structures**. (a) **The Map/Path**: the curriculum road with milestone gifts, where Trophy Road, the level map and the album live. (b) **My Space**: avatar plus room or island, the *only* sink for the one currency, where cosmetics live. (c) **The Family Garden**: shared and cooperative, fed by everyone's weekly activity, where co-op bosses and family quests pay out. Pets and companions live inside My Space and evolve with mastery. Everything else is cut or merged.
4. **Adaptive engine reality check.** With 3 children and a hand-authored item bank, Elo item difficulty will barely move. Plan for **designer-assigned difficulty levels plus a simple per-skill rule engine** (D3, D9, D10) as the real core, with Elo on *child ability only*, against fixed item difficulties. FSRS/HLR is out of scope. The photo pipeline's items arrive without difficulty data, so they inherit a level from content type and number range. This simplification also makes the dashboard explainable.
5. **Celebration time budget.** Add a hard metric: celebration plus transition time ≤15% of session time (proposed). Measure it in telemetry, since the D7 arithmetic shows how fast juice eats a 5-minute toddler episode.
6. **Device-sharing fairness.**
   - Three kids, likely one or two iPads. Nobody covered turn-taking.
   - Proposed: a visible "turn ring" in the family hub showing whose turn is next. Equal daily budgets are shown to the parent only.
   - A co-op mode (one iPad, both sides of the table) counts toward both kids' time.
   - The profile picker uses avatar faces only. Switching saves state instantly, so a forced switch never loses progress.
   - Fairness matters because perceived differential treatment is the harm mechanism [note: child_psychology_motivation.md §5].
7. **Physical-unit sizing.** Fix the conversion error before any design tokens are written. Express targets in mm and convert per device.
8. **New-curriculum cohort risk.**
   - The 8.5-year-old's class follows a curriculum that started this year for grade 3.
   - Older apps and worksheets, and LLM "grade 3 Israel" knowledge, reflect the *old* program.
   - Mitigations: the parent's monthly class-topic input; using photos of her actual homework as ground truth; tagging every skill node with "source: new curriculum / old / unknown".
9. **Fluency instrument parity with school.** Mirror the Ministry fluency tool's format and error bands (Section 3), so parents can talk to the teacher in shared terms. Only report wpm for pointed text until mid-year.
10. **Evaluation plan is missing.** No note specifies how the family will know whether the app teaches anything. Proposed minimal protocol:
    - per-skill in-game probes, untimed and unrewarded, at baseline and every 4 weeks;
    - delayed-retention checks (D9's ≥3-day rule doubles as this);
    - one free-choice measure of intrinsic interest: does the child choose the playroom or optional content with no reward attached?
    - one parent-observed transfer check per month (for example, reads a sign in the street, uses multiplication while shopping).
    - This is the honest counterweight to the small expected effects (math note: ~0.1–0.4 SD [note: math_pedagogy.md §5]; gamification at primary g ≈ 0.31).
11. **"App sleeps" contexts (from AAP 2026).** Beyond bedtime and Shabbat, add parent-set "media-free spaces": meals and car rides, following the AAP family-media-plan guidance. For the car, offer an *audio-only* Hebrew/English song-and-riddle mode as an alternative, since screens can displace sleep and movement.
12. **Older-sibling burden.** Azmitia & Hesser support sibling teaching. The psychology note warns against making her the "permanent babysitter" [note: child_psychology_motivation.md §5]. Proposed: guide mode is opt-in per session, capped (for example 1 co-op episode per day), and pays *her own* progress (a "mentor" skill track with its own milestones), so it's never a cost to her.
13. **Parent-gate collision with the 8.5-year-old.** Multiplication-question gates [note: ui_ux_polish_craft.md §6] will be solved by a grade-3 child who is literally learning multiplication this year. Use a parent PIN or Face ID for the parent area, not arithmetic.
14. **Color-blind and gender-coded rarity.** Rarity colors (gold/purple/pink) and avatar defaults can encode gender stereotypes. Keep item pools gender-neutral (Animal Crossing model) and never pre-filter cosmetics by the profile's gender.

### Gaps
- No data on how often Israeli apps mis-gender girls in Hebrew UI. Concern 1 rests on language structure, not a measured study.
- No source on turn-taking or device-sharing conflicts among siblings with tablets.
- The 15% celebration-time budget and the probe cadence are proposals with no validation.
