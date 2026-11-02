# Research log: English (EFL) curriculum

> Per 00-F3: research done while writing a spec is logged here. Feeds `docs/spec/05-curriculum-english.md`.
> Session date: 2026-11-02. Author: spec agent (curriculum designer role).

## Access conditions this session (read first)

- **Blocked hosts:** `www.cambridgeenglish.org`, `flyer.us` (DNS not resolvable via WebFetch), and `curl` CONNECT
  to `meyda.education.gov.il`, `katzir.org.il` and `cambridgeenglish.org` returned proxy 403. The official word lists
  (Cambridge Pre A1 Starters 2018 list, MoE "Lexical Pre-Band I & Band I") could **not** be opened in full.
- **Search budget:** the shared WebSearch budget ran out mid-session. Two planned queries (HVPT meta-analysis,
  Hebrew-L1 English vowel perception) were not run. Claims on those topics in the spec are flagged as
  *background knowledge, unverified this session*.
- **Action for parent/devops (one line):** to verify the word lists, allow `cambridgeenglish.org`,
  `meyda.education.gov.il`, `pop.education.gov.il` in the session's network settings, then re-run the
  verification task in "Open verification tasks" below.

Evidence tags used: **[V]** verified in a source this session (search snippet or note), **[N]** from an earlier
research note in this repo, **[BK]** background knowledge, not verified this session, **[D]** design decision.

---

## 1. Israeli MoE curriculum (Pre-A1 / Pre-Band I)

**Findings**
- [V] The MoE Pre-A1 can-do list (pop.education.gov.il, "Can dos: Pre-basic user (Pre-A1)") includes:
  understanding short, very simple questions and statements delivered slowly and clearly with visual or gesture
  support; recognizing Pre-Band I vocabulary and instructions; repeating short phrases; asking someone's name or the
  time; asking for a spelling; getting attention with a fixed expression; making a few basic requests; **reciting a
  short rhyme or chant**; reading familiar irregular words; reading sentences built from Pre-Band I vocabulary;
  tracing and copying individual letters **in the correct direction**; using a full stop to end a sentence.
  — [Can dos Pre-A1 (pop)](https://pop.education.gov.il/tchumey_daat/english/yesodi/curriculum/basic-user-a2/);
  [candospreA1.pdf](https://meyda.education.gov.il/files/Mazkirut_Pedagogit/English/CurriculumFilesAugust21/candospreA1.pdf)
- [V] Curriculum 2020: Pre-A1 instruction focuses mainly on **extensive listening and speaking before literacy** is
  introduced. — [English Curriculum 2020](https://meyda.education.gov.il/files/Mazkirut_Pedagogit/English/Curriculum2020.pdf)
- [V] The MoE "Lexical Pre-Band I" list (updated 26 Jan 2023, typos corrected) tags each entry with part of speech
  and **Rec/Prod** (recognition vs production). It is labeled "elementary school"; no grade split was visible.
  — [LexicalBand1.pdf](https://meyda.education.gov.il/files/Mazkirut_Pedagogit/English/CurriculumFilesAugust21/LexicalBand1.pdf);
  mirror [katzir.org.il](https://katzir.org.il/wp-content/uploads/2022/06/LexicalBand1.pdf)
- [V] An unofficial page offers a Band I table with Hebrew translations ("MOE English Word Lists Updating to 2020").
  Not official; useful for cross-checking glosses only. — [educatingisrael.com](http://www.educatingisrael.com/vocab.php)
- [V] Pre-A1 draws partly on a forthcoming "Teaching of English at the Pre-Foundation Level" document (not found).
  — [CurriculumIntermediate2.pdf](https://meyda.education.gov.il/files/ChinuchMukar/CurriculumIntermediate2.pdf)
- [V] ENJOY ENGLISH: continued in kindergarten and grade 1 with teacher mentoring (Knesset background paper). A 2022
  report describes NIS 70M for experiential exposure K–2. No weekly hours found. —
  [Knesset](https://fs.knesset.gov.il/25/Committees/25_cs_bg_5723711.pdf);
  [Jewish Press 2022](https://jewishpress.com/news/on-campus/israeli-schoolchildren-start-summer-vacation-teachers-still-wrestling-with-finance-ministry/2022/06/30)
- [N] Grade 3 = first compulsory year; Foundation (A1, Band I, ≥1,200 items) = grades 4–6 (english_efl.md §1).

**Implications for the spec**
- Rec/Prod split adopted as a per-word field (`use: rec|prod`) in the vocab bank.
- Chant/rhyme recitation, letter direction, full stop, asking for spelling and "what time is it" are explicit
  can-do nodes, because they are named in the MoE list.
- Word list: Cambridge Starters topics used as the backbone (see §2) with MoE Pre-Band I as the target to verify.

## 2. Cambridge Pre A1 Starters vocabulary

**Findings**
- [V] Official sources exist: A–Z list (2018) and a picture word-list book (A–Z on pp. 25–30); a newer edition
  (April 2025) is referenced by third parties. — [Starters word list 2018](https://www.cambridgeenglish.org/images/351849-pre-a1-starters-word-list-2018.pdf);
  [picture book](https://www.cambridgeenglish.org/Images/starters-word-list-picture-book.pdf);
  [TES flashcards note](https://www.tes.com/teaching-resource/yle-pre-a1-starters-flashcards-a-to-c-13294293)
- [V] Third-party topic snippets confirm body words (foot, hair, hand, head, leg, mouth, nose) and colors (black,
  brown, green, grey, orange, pink, purple, red, yellow). — [flyer.us snippet](https://flyer.us/?p=469)
- [N] Test format: Listening ~20 min / 4 parts; Reading & Writing 5 parts; Speaking 3–5 min; shields (max 5);
  ages 7–12 (placement_assessment.md §3).
- [BK] Starters topic list (2018): animals; the body and the face; clothes; colours; family and friends; food and
  drink; the home; numbers 1–20; places and directions; school; sports and leisure; time; toys; transport; weather;
  the world around us; plus names list and grammar structures (there is/are, can, have got, like + -ing, imperatives,
  prepositions in/on/under/next to/behind/between/in front of, question words). UK/US variants listed in pairs
  (e.g., eraser/rubber, fries/chips, candy/sweets, apartment/flat, soccer/football, truck/lorry).

**Implications**
- The spec's vocab bank (≥400 words) is organized by these topics, marked P1/P2/P3. **US forms** are primary (00-E9:
  native US voice); UK forms stored as `alt`.
- Verification task V1 below: diff the bank against the official 2018/2025 list and MoE Pre-Band I once the hosts
  are allowed.

## 3. Synthetic phonics sequences

**Findings**
- [V] Jolly Phonics: 42 sounds in 7 groups: (1) s a t i p n; (2) c/k e h r m d; (3) g o u l f b; (4) ai j oa ie ee or;
  (5) z w ng v oo; (6) y x ch sh th; (7) qu ou oi ue er ar. Group 1 chosen because it makes more simple 3-letter
  words than any other six letters. One guide has a likely typo for group 3. —
  [smarterlearningguide](https://smarterlearningguide.com/jolly-phonics-review);
  [learnphonics chart](https://learnphonics-3et.pages.dev/jolly-phonics-sounds);
  [Inspire workshop slides](https://www.slideshare.net/slideshow/reception-inspire-workshop-presentation/773302)
- [V] UFLI Foundations: Lesson 1 /ă/, 2 /m/, 3 /s/; all lowercase letters, single consonants and short vowels by
  lesson 34 (Phase 1 "The Alphabet", lessons 1–34); 128 lessons in 14 phases; lesson 13 = d, 20 = -s, 27 = l;
  later "Alphabet Review & Longer Words" (CVC, CCVC, CVCC, CCVCC, CCCVC). —
  [EdReports UFLI K](https://edreports.org/reports/detail/ufli-ela-k-2-2025/kindergarten/gateway-one);
  [Scholastic alignment](https://education.scholastic.com/content/dam/education/programs/marketing/ready4reading/short-reads-decodables/pdfs/ufli-aliginment.pdf);
  [spellingtestbuddy guide](https://spellingtestbuddy.com/resources/ufli-scope-and-sequence/)
- [N] UFLI materials forbid alteration and derivatives (content_world_phygital.md §a3): **use the idea of a
  sequence, never UFLI text/decodables.**
- [BK] UK DfES "Letters and Sounds" (2007): Phase 2 sets s a t p / i n m d / g o c k / ck e u r / h b f ff l ll ss;
  Phase 3 j v w x / y z zz qu / ch sh th ng / ai ee igh oa oo ar or ur ow oi ear air ure er; Phase 4 adjacent
  consonants; Phase 5 alternative graphemes (ay ou ie ea oy ir ue aw wh ph ew oe au, split digraphs).
  Tricky words Phase 2: the to I no go into; Phase 3: he she we me be was you they all are my her; Phase 4: said have
  like so do some come were there little one when out what.

**Decision [D]:** the spec uses a sequence that follows the Letters-and-Sounds/Jolly order of the first
graphemes (s a t p → i n m d → …) because it yields real CVC words from step 1, and adapts it for (a) a US accent
(drop non-rhotic `ear/air/ure` as units; r-controlled taught as ar/or/er-ir-ur), (b) Hebrew-L1 pressure points
(w/v and th introduced with minimal-pair listening; short a/u/o triad flagged), (c) an 8.5-year-old pace.

## 4. High-frequency words

- [V] Dolch: 220 service words (pre-primer 40, primer 52, grade 1 41, grade 2 46, grade 3 41) + 95 nouns = 315;
  claimed 50–75% of typical text. Origin date reported as 1936 or 1948 (conflict). —
  [Wikipedia: Dolch word list](https://en.wikipedia.org/wiki/Dolch_word_list);
  [k12reader](https://www.k12reader.com/kindergarten-sight-words/)
- [BK] Fry: 1,000 words, first 100 ≈ 50% of text. Not verified this session.
- [D] Policy: high-frequency words are split into **decodable-later** (taught by phonics when their step arrives)
  and **tricky** (irregular at the current step). Only tricky words get whole-word ("heart word") treatment, with
  the regular part decoded and only the irregular part marked.

## 5. Phonemes and Hebrew-L1 transfer

- [V] Popular/teacher sources (not peer-reviewed): Hebrew has ~5 vowels; ship/sheep and bit/beat blur; no /θ ð/;
  some speakers say V for W ("west"→"vest"), but **not universal: screen, don't assume**; final-syllable stress habit;
  full vowels in unstressed syllables. Final devoicing **not documented** for Hebrew speakers. —
  [BoldVoice](https://boldvoice.com/blog/english-pronunciation-hebrew-speakers);
  [funfluen](https://funfluen.com/learn/guides/english-pronunciation-for-hebrew-speakers-w-v-th-english-vowels-and-rhythm/);
  [inpronunci](https://inpronunci.com/israeli-accent-in-english/);
  [Wikipedia: Non-native pronunciations of English](https://en.wikipedia.org/wiki/Non-native_pronunciations_of_English)
- [N] Kahn-Horwitz et al.: English letter knowledge at the start of the first EFL year is the strongest predictor of
  later EFL reading/spelling in Israeli children; Hebrew readers need an "approach shift" because English vowels are
  full letters (english_efl.md §3).
- [V] "~44 phonemes" is the standard figure, varying by accent. —
  [Reading Rockets 44 phonemes](https://Www.readingrockets.org/sites/default/files/migrated/the-44-phonemes-of-english.pdf);
  [Victoria DoE 44 sounds](https://www.education.vic.gov.au/Documents/school/teachers/teachingresources/discipline/english/literacy/44SoundsofAusEnglish.docx)
- [BK] Analysis used in the spec: Hebrew vowels /a e i o u/ → US short /æ/, /ɑ/, /ʌ/ all tend to map onto Hebrew
  /a/ (cat–cot–cut triad); /ɪ/→/i/ (ship–sheep); /ʊ/→/u/ (full–fool); /ɝ/ and schwa unfamiliar; /ɹ/ replaced by
  uvular /ʁ/; /ŋ/ produced as /ŋg/; /h/ often weak in Israeli speech; /tʃ dʒ ʒ/ present via loanwords (צ׳ ג׳ ז׳).
  Grammar transfer at Pre-A1: copula omission ("She happy"), article omission ("I have dog"), noun–adjective order
  ("ball red"), "open/close the light", "he/she" for objects, "I have 8 years". Spelling: vowel-letter omission
  (abjad habit), right-to-left letter order in early spelling, b/d/p/q confusion, no letter case in Hebrew.
  **All [BK]; not verified against peer-reviewed Hebrew-L1 studies this session.** Treated as hypotheses the error
  catalog *detects* rather than assumes.
- [BK] High Variability Phonetic Training (multiple talkers, minimal pairs, immediate feedback) improves L2
  perception and transfers partly to production (e.g., Thomson 2018 review; Sakai & Moorman 2018 meta-analysis).
  **Search not run (budget).** Spec uses ≥4 TTS talkers for minimal pairs on that basis.

## 6. Audio sources

- [V] No CC0/CC-BY phonics-audio set for children was found. Wikimedia Commons hits were not audio. —
  (search 2026-11-02; see 05 §audio)
- [N] Wikimedia IPA clips are mostly CC BY-SA 3.0/4.0; an MIT project bundles 44 English phonemes from them (35/36
  CC BY-SA 3.0, one PD); IPA Handbook audio is personal use only (voice_tech_he_en.md §3).
- [N] TTS cannot reliably produce isolated phonemes; record ~44 sounds with a native speaker (00-E9).

## 7. Songs (public domain)

- [V] Traditional nursery rhymes are generally PD; specific arrangements, illustrations and **recordings** may be
  protected. Gutenberg hosts PD collections (e.g., Moffat & Willebeek le Mair, *Our Old Nursery Rhymes*, 1911;
  *Little Songs of Long Ago*; *Traditional Nursery Songs of England*, 1843). —
  [Gutenberg 61009](https://gutenberg.org/ebooks/61009); [Gutenberg 55107](https://m.gutenberg.org/ebooks/55107);
  [Gutenberg 30418](https://www.gutenberg.org/ebooks/30418);
  [PD Info nursery rhymes](https://mail.pdinfo.com/pd-music-books/nursery-rhymes-and-nursery-songs.php)
- [V] "The Wheels on the Bus": credited to Verna Hills (1898–1990), first published in *American Childhood*
  (1937 or 1939). Copyright status unsettled; Wikipedia talk notes uncertainty. —
  [Wikipedia](https://www.wikipedia.org/wiki/The_Wheels_on_the_Bus);
  [Classic FM](https://www.classicfm.com/discover-music/wheels-on-the-bus-lyrics-composer-history/)
- [BK] "Happy Birthday to You": US court (Rupa Marya v. Warner/Chappell, 2015; settlement 2016) → lyrics treated as
  PD in the US. "Baby Shark" (Pinkfong arrangement) is protected: avoid. "If You're Happy and You Know It": authorship
  disputed; status not verified. "Head, Shoulders, Knees and Toes": playground song to the tune of "There Is a Tavern
  in the Town" (1883); treated as traditional, not verified.
- [D] Policy in spec: song tiers **PD-safe** (published before 1927 or traditional with an old printed source),
  **TRAD-verify** (folk but a modern author is claimed: use only after check, else replace), **ORIG** (our own lyrics
  on a PD tune). All recordings are produced by us; no third-party recordings.

## 8. Israeli school calendar 5787 (2026–27)

- [V] Start Tue 1 Sep 2026. Rosh Hashana closed 11–13 Sep (back 14 Sep). Yom Kippur 20–21 Sep. Sukkot break
  25 Sep–3 Oct (back 4 Oct). Hanukkah break ~6–12 Dec 2026 (back 13 Dec; one source 5–11 Dec, provisional).
  Purim ~22–24 Mar 2027 (non-state). Pesach (state) 13–28 Apr 2027. Yom HaZikaron 11 May 2027 (ends 12:00).
  Shavuot 10–11 Jun 2027. Summer 1 Jul–31 Aug 2027 (elementary). Sources disagree on exact days. —
  [Shemesh calendar](https://www.shemesh.co.il/en/highlights/complete-school-holiday-and-vacation-calendar-for-2026-2027/);
  [axcalendar](https://axcalendar.info/eng/vacations/); [beevago](https://www.beevago.com/en/school-holidays/israel);
  [timeanddate](https://www.timeanddate.com/holidays/israel/first-day-of-school-year)
- [D] The app's calendar module (00-B7) must read dates from a data file, not hard-code them; the spec's year plan
  uses school weeks and lists dates as "approx., verify against MoE circular".

## Open verification tasks

| ID | Task | Blocked by | Owner |
|---|---|---|---|
| V1 | Diff `packages/content/en/vocab.json` against Cambridge Starters (2018 + 2025) and MoE Pre-Band I; tag `src`. | hosts blocked | dev, when hosts allowed |
| V2 | Read MoE Pre-A1 can-do PDF in full; map each statement to an `E.*` node id. | host blocked | dev |
| V3 | HVPT and Hebrew-L1 vowel perception: find 2–3 peer-reviewed sources; adjust PR.* nodes if needed. | search budget | dev |
| V4 | Copyright check for TRAD-verify songs before audio production. | — | dev (+ parent sign-off is not needed: family-only, A1) |
| V5 | Confirm 5787 calendar from the MoE circular; update `calendar/5787.json`. | host blocked | dev |

## Cross-spec alignment (same session)

- `01-game-design.md` already fixed the English-only character as **Sunny** (CH-02, sea otter, f) with **ניצי** as
  sandwich translator, region R-08 Sunny Harbor, mechanics MX-E01..E04, MV-08 "Sunny Says", Wordlings, boss
  Mr. Mix-Up, and English island opening in arc week 6 (~2026-12-20). 05 adopts all of these. It proposes
  "bottles from the sea" before the opening, a minimal-pairs mode for MX-E01, MX-E05..E07, and Pip the crab
  (a decodable, non-speaking sidekick).
- `08-assessment-engine-child-model.md`: mastery levels `proficient`/`master` (D9). Default subject weights are
  K1 English 15% and K2 10%. 05 requests 25% and 15%.
- `09-ux-ui-design-system.md`: `font.en` = Andika, `type.en.k1` 26pt. 05 requests decode-word and letter tokens.

