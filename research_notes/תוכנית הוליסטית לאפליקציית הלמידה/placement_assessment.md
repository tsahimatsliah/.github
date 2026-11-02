# Playful Placement and Stealth Assessment for a Hebrew-First Kids' Learning App (girl 8.5 / grade 3, boy 5.5 / gan chova, toddler 2.5)

Research notes, November 2026. Method note: the web search tool worked, but fetching full pages failed for www.mheducation.com and pmc.ncbi.nlm.nih.gov because those hosts are not in the session's allowed network domains. Findings from those hosts come from search-result snippets, not full-text reads. Anything that comes from background knowledge and was not confirmed by a source this session is marked "unverified" and is placed in Inferences or Gaps, not Cited Findings.

---

## 1. Computerized adaptive testing (CAT) for young children: IRT basics, items needed, and how commercial placement systems work (ALEKS, i-Ready, IXL, MAP Growth K-2, Khan, Math Garden, Duolingo); cold-start strategies

### Takeaway
Commercial K-3 placement systems are much shorter for young children than for older students. ALEKS Adventure (K-3) caps the initial check at 9 questions for kindergarten and 14 for grades 1-3. Knowledge-space methods need roughly log2(number of feasible knowledge states) questions because each answer also tells you about skills that were not tested. Elo-style systems such as Math Garden skip the separate test: ability and item difficulty update after every answer, and using speed as well as accuracy improves the estimate. So a kids' app does not need a long "test". A short adaptive probe of about 8-15 items per subject, followed by continuous Elo/IRT updating during play, matches what the field does.

### Cited Findings
**ALEKS (knowledge space theory)**
- The ALEKS Initial Knowledge Check (IKC) for higher-ed and secondary courses now has at most 25 questions, down from 30 — [McGraw Hill "Shorter, smarter knowledge checks"](https://www.mheducation.com/content/dam/mhe/highered/documents/aleks/innovation-shorter-smarter-knowledge-checks.pdf) (search snippet); an ASU course page gives 25-30 adaptive questions — [ASU ALEKS intro](https://sites.google.com/asu.edu/mat142introductiontoalekshonor/aleks-intro-and-initial-knowledge-check)
- **ALEKS Adventure (the K-3 product): the knowledge check has at most 9 questions for kindergarten and 14 for grades 1-3** — [McGraw Hill ALEKS Adventure Knowledge Checks](https://www.mheducation.com/prek-12/support/knowledge/aleks-adventure-knowledge-checks.html) (search snippet; the full page could not be fetched)
- In knowledge space theory, a student's knowledge state can be found with about log2(number of feasible knowledge states) questions. The learning-space model infers mastery of concepts that were never directly tested, which cuts the number of questions needed — [Falmagne/Doignon et al., "Learning Sequences" (arXiv 0803.4030)](https://arxiv.org/pdf/0803.4030)

**i-Ready Diagnostic / MAP Growth K-2 / IXL**
- Curriculum Associates guidance puts the i-Ready Diagnostic at about 45 minutes per subject, varying by grade — [i-Ready FAQ Diagnostic Duration 2022](https://www.curriculumassociates.com/-/media/mainsite/files/i-ready/iready-faq-diagnostic-duration-2022pdf.pdf); it is adaptive across reading and math, K-12 — [Ohio i-Ready AIF](https://education.ohio.gov/getattachment/Topics/List-of-Approved-Assessments/iReadyAIF25-26.pdf.aspx?lang=en-US)
- MAP Growth K-2 is computer-adaptive with audio support, but not every item has audio (directions may have audio when a reading passage does not) — [NWEA: Do all MPG items have audio](https://connection.nwea.org/articles/Knowledge/Do-all-MPG-test-items-have-audio). It is untimed, and breaks are encouraged — [NWEA Average MAP Growth Test Durations](https://nwea.org/uploads/Average-MAP-Growth-Test-Durations.pdf). An older NWEA sheet lists an "Early Literacy (33 items)" strand, and NWEA says the number of K-2 items was cut starting fall 2017 — [NWEA MAP Growth K-2 Content](https://www.nwea.org/resource-center/fact-sheet/48194/MAP-Growth-K-2-reading-and-mathematics-content_NWEA_onesheet.pdf)
- IXL Real-Time Diagnostic: about 45 minutes for full results in one subject. After that, 10-15 diagnostic questions a week keep it current. A level shows as a range until enough answers pin it down (then a "star"). Levels map to grade, for example 500 means ready for 5th grade — [IXL for Diagnostic Assessment guide](https://au.ixl.com/materials/i_guides/IXL_for_Diagnostic_Assessment.pdf); [IXL blog: personalized learning plan](https://blog.ixl.com/2023/05/25/create-a-personalized-learning-plan-for-your-child/)

**Math Garden (Rekentuin): Elo-based adaptive practice that doubles as monitoring**
- Math Garden is a web-based computer-adaptive practice and monitoring system. It tracks accuracy and response time, matches items to each child's ability, and reports progress to teachers automatically — [Klinkenberg profile / Math Garden publications, UvA](https://www.uva.nl/en/profile/k/l/s.klinkenberg/s.klinkenberg.html); [Math Garden: a new educational and scientific instrument](https://dare.uva.nl/id/67ab8977-5e36-4f9f-962d-be294f2b8c09)
- The model extends the Elo chess rating system. Person ability and item difficulty are both updated after every item-person interaction (Klinkenberg, Straatemeier & van der Maas, 2011, Computers & Education) — [UvA profile](https://www.uva.nl/en/profile/k/l/s.klinkenberg/s.klinkenberg.html); [Pelánek, Elo in education (CAE)](https://www.fi.muni.cz/~xpelanek/publications/CAE-elo.pdf)
- A 2019 Frontiers paper on real Math Garden data found that modeling speed and accuracy together beats modeling accuracy alone — [PMC6450197](https://pmc.ncbi.nlm.nih.gov/articles/PMC6450197) (search snippet)
- Bolsinova et al. (2026) argue that the usual Elo fix (shrink K as responses pile up) fails when ability changes a lot, as it does in digital learning. They propose a dynamic K and implement it in Math Garden — [Tilburg University: Balancing stability and flexibility, dynamic K](https://research.tilburguniversity.edu/en/publications/balancing-stability-and-flexibility-investigating-a-dynamic-k-val/)
- Klinkenberg's publications include a 2015 study of how experiencing success in math affects math anxiety and performance — [UvA profile](https://www.uva.nl/en/profile/k/l/s.klinkenberg/s.klinkenberg.html)

**Duolingo**
- The Duolingo English Test (the high-stakes exam, not the in-app placement) adapts so test-takers do not spend time on items far above or below their level. Adaptive items have objectively correct answers and are scored in real time. ML-based linguistic models estimate item difficulty for the CAT — [Duolingo blog: Is the DET hard?](https://blog.duolingo.com/is-the-duolingo-english-test-hard/); [DET Technical Manual](https://www.peb.ufrj.br/images/documentos/DET-technical_manual-V01.pdf); [Duolingo scoring whitepaper 2024](https://duolingo-papers.s3.amazonaws.com/reports/Duolingo_whitepaper_test_scoring_2024_v1.pdf)

**IRT stopping rules / precision**
- The most common CAT stopping rule ends the test once the trait estimate's standard error (SE) reaches a preset value. A fixed-length rule can hurt efficiency, and hybrid rules (e.g., PSER) try to balance precision and length — [Meadows Center: Stopping rules for computerized adaptive screening](https://meadowscenter.org/library/a-comparison-of-stopping-rules-for-computerized-adaptive-screening-measures-using-the-rating-scale-model/); [PROMIS Assessment Center: Stopping Rules](https://www.assessmentcenter.net/ac_api/2012-01/Docs/Stopping_Rules.docx)
- Simulation studies compare SE<0.30, SE<0.50 and SE<0.70 against fixed lengths of 10 and 20 items — [EJMSTE: CAT termination rules](https://www.ejmste.com/article/the-impact-of-computerized-adaptive-test-termination-rules-on-accuracy-across-different-ability-15897)
- A child-relevant example: CAT simulations of the CDI vocabulary inventory (Wordbank) used a minimum of 25 items, a maximum of 50, and stopped at SE = 0.1 — [RPubs CDI-CAT analysis](https://rstudio-pubs-static.s3.amazonaws.com/1141036_591ab51a65ec4ce7abafcb12497df484.html)
- A general IRT framework reference — [arXiv 2108.08604, IRT statistical framework](https://arxiv.org/pdf/2108.08604)

### Inferences
- **Arithmetic:** on a standard-normal theta scale, reliability is about 1 − SE². So SE 0.30 gives about 0.91, SE 0.40 about 0.84, and SE 0.50 about 0.75. For placement (choosing a starting node on the map, not making high-stakes decisions), SE ≈ 0.4-0.5 is enough, and continued play will tighten it.
- **Item budget per child per subject strand (proposed):** ages 8-9: 10-15 items per strand (cap 20). Age 5-6: 6-10 items per strand (cap 12, mirroring ALEKS Adventure's K cap of 9). Age 2.5: no item test (see section 4). Split this across 2-4 short sessions of 5-8 minutes. Do not run one 45-minute sitting the way i-Ready, IXL and MAP do; those are school products given under supervision.
- **Recommended hybrid algorithm:**
  1. **Prior from age and grade.** Start theta at the grade-typical level, or one level below to guarantee early successes. Use the Israeli grade the child is actually in (grade 3 and gan chova) and the parent's quick self-report ("reads short words / reads books alone").
  2. **Coarse binary search over a prerequisite skill graph** (ALEKS/KST style). Probe a mid-level node. A success marks its prerequisites as likely mastered; a failure marks its successors as likely not mastered. Then jump half the remaining distance. This gives about log2(nodes) probes per strand.
  3. **Fine estimation with Elo/1PL (Rasch).** Per strand, update θ ← θ + K·(score − expected). Use larger K early (cold start) and decay it with count, but bring it back up after long gaps or when residuals drift (the Bolsinova dynamic-K argument). Use a Math Garden-style score that rewards fast correct answers, but only for fluency strands (fact fluency, decoding speed). For comprehension and conceptual strands, score accuracy only, so slow, careful thinkers are not penalized.
  4. **Stop rule** = SE < 0.45 OR the item cap is reached OR 3 consecutive disengagement signals (section 2). Always end on an easy item.
  5. **Continuous calibration afterwards.** Item difficulties start from authored "grade levels". Once the family has played enough, re-estimate difficulties from the household's own data plus any shared data (Elo item updates). With only 3 children, item calibration will mostly have to come from authored priors or a wider user base. This is a real limitation for a single-family app.
- **What vendors converge on:** (a) an initial diagnostic, (b) continuous updating from ongoing work (IXL's 10-15 questions a week; Math Garden's per-item updates), (c) reporting a range until precision is reached (IXL's range vs. star). The app should copy (c) in the parent dashboard: show "about level 2.3-2.8" until the range is tight.
- **"I don't know / skip" option:** KST systems treat "I don't know" as informative (it reduces guessing). For kids, a friendly "too tricky, show me another" button is a good fit. Count it as a non-mastery signal with lower weight than a wrong answer. (Design reasoning; I could not fetch the ALEKS page to confirm how ALEKS itself handles it.)

### Gaps
- Exact Math Garden constants are unverified: the "high speed, high stakes" scoring rule S = (2x−1)(a·d − a·t) with a ~20-second deadline, the default target success probability (~0.75), and how it starts new players. The PMC page could not be fetched.
- I found no official item count for the i-Ready K-2 Diagnostic, and none for Khan Academy Course Challenges / Mastery Challenges. Unverified background: Khan uses mastery levels Attempted → Familiar → Proficient → Mastered, and course/unit challenges can level up several skills at once.
- I found no sources on how the Duolingo app's in-app placement test works (as opposed to the DET).
- I found no published reliability figures for very short (<10-item) CATs specifically in children aged 5-6.

---

## 2. Stealth assessment (Shute; evidence-centered design) and measuring from gameplay telemetry

### Takeaway
Stealth assessment builds evidence-centered design (ECD) into gameplay: a competency model (what to measure), an evidence model (which observables update which competency, and how, often with Bayesian networks), and a task model (game situations that elicit the evidence). The published evidence, mostly from Shute's own group, suggests it can be valid and reliable and can support learning. That makes it the right architecture for ongoing assessment once placement is done. Telemetry such as response time, retries, hints and error type are the observables.

### Cited Findings
- Stealth assessment measures competencies while supporting their development during play, and blurs the lines between play, learning and assessment. ECD supplies the steps to design it, embed it in a digital environment, and evaluate it. Aligning the task and evidence models to the competency model is key to validity — [Shute & Rahimi, Stealth Assessment Primer (2022)](https://myweb.fsu.edu/vshute/pdf/SA_Primer.pdf); [Shute, Lu & Rahimi, Stealth assessment handbook chapter](https://myweb.fsu.edu/vshute/pdf/sa_handbook.pdf)
- Shute's foundational stealth-assessment work and argument — [Shute 2008/2011 paper](https://myweb.fsu.edu/vshute/pdf/shute%202008_e.pdf); [TICL 8(2) 2010, pp.137-161](https://www.oldcitypublishing.com/journals/ticl-home/ticl-issue-contents/ticl-volume-8-number-2-2010/ticl-8-2-p-137-161/)
- Traditional tests can demotivate lower-performing students. Stealth assessment is offered as a remedy — [SA Primer](https://myweb.fsu.edu/vshute/pdf/SA_Primer.pdf)
- Worked examples: Physics Playground (physics knowledge, creativity, persistence) and Plants vs. Zombies 2 (problem solving), both built with ECD — [SA handbook](https://myweb.fsu.edu/vshute/pdf/sa_handbook.pdf); [KQED MindShift: Can a child's creativity and persistence be assessed by a game?](https://www.kqed.org/mindshift/38725/can-a-childs-creativity-and-persistence-be-assessed-by-a-game)
- The 2022 primer reports that current evidence suggests game-based stealth assessment is valid and reliable and supports learning. KQED reports significant, consistent correlations with externally validated tests — [SA Primer](https://myweb.fsu.edu/vshute/pdf/SA_Primer.pdf); [KQED](https://www.kqed.org/mindshift/38725/can-a-childs-creativity-and-persistence-be-assessed-by-a-game). **Caveat:** most of the evidence comes from Shute's team. I found no independent replications in this search.
- Response time adds information beyond accuracy (Math Garden data) — [Frontiers 2019 via PMC6450197](https://pmc.ncbi.nlm.nih.gov/articles/PMC6450197)

### Inferences
- **Competency model (per child):** a skill graph per subject (section 3 lists the nodes). Each node holds (θ, SE, last-seen, fluency index). Add cross-cutting "learner-state" variables that are not academic: persistence, attention/engagement, frustration.
- **Evidence model: observables to log for every item event:**
  - correctness; latency (time to first touch, time to submit); number of attempts; changes of answer; hint usage; "skip/too tricky" taps
  - **error type** (distractor chosen). Design distractors to be diagnostic. Math: off-by-one counting; place-value reversal (e.g., 41 for 14); adding instead of subtracting. Hebrew: visually similar letters such as ב/כ, ד/ר/ך, ה/ח/ת, ו/ז/ן, ס/ם; vowel (niqqud) confusions such as kamatz vs. patach (same sound), or segol/tsere vs. chirik
  - for tracing: path deviation, stroke order, stroke direction, lifts, completion time
  - for audio and speech: ASR confidence on read-aloud words, words correct per minute (WCPM) on short read-aloud passages (grade 3)
  - session-level: time on task, mid-item idle gaps, rapid-guess rate (answers faster than a plausible minimum, e.g., <0.8 s on a reading item), tapping randomly, quitting points
- **Telemetry scoring rules:** (a) rapid guesses and idle-timeout items are **excluded** from θ updates (they reflect disengagement, not ability) but counted toward an engagement score; (b) latency feeds a fluency index only for fluency-type nodes; (c) after an error, a short "try again with a scaffold" turns the item into a learning moment, and only the first attempt is scored.
- **Task model:** every mini-game type declares which nodes it gives evidence for, so ordinary play keeps updating the model. This removes the need for frequent formal tests.
- **Bayesian network vs. Elo:** Shute's group uses Bayes nets to aggregate evidence. For a small iOS app, per-node Elo plus prerequisite propagation (a success on a child node nudges its parents up; a failure on a parent nudges its children down) is a practical approximation. This is a design choice, not sourced.

### Gaps
- I found no peer-reviewed thresholds for "hesitation" latencies in 5-9-year-olds on tablet tasks. Thresholds will need to be set empirically from the children's own baselines (z-scores relative to self).
- I found no validated stealth-assessment model specifically for Hebrew literacy.

---

## 3. Established screeners per subject and what they measure (DIBELS/Acadience; Israeli Hebrew tools; early numeracy; English YL / Cambridge Pre A1 Starters)

### Takeaway
The early-literacy screeners agree on a small set of indicators: phonemic awareness (first sound, segmentation), letter knowledge, alphabetic principle/decoding (nonsense words), and oral reading fluency (accuracy + rate, later comprehension). Israeli Ministry tools assess Hebrew reading fluency on three dimensions (accuracy, rate, prosody) using a word list and a story text, with dedicated tools at grade 1 (reading-writing test, "Yesodot"/early literacy) and grades 2-3 (fluency). Early numeracy screeners centre on counting, magnitude comparison, nonverbal calculation, story problems and number combinations. Kindergarten number sense predicts first-grade math well. Cambridge Pre A1 Starters (ages 7-12) gives an age-appropriate model for English: picture-based listening, reading/writing and short speaking tasks.

### Cited Findings
**DIBELS / Acadience (English-language model of the indicators)**
- First Sound Fluency (FSF) measures phonemic awareness: the child says the first sound of an orally presented word. Phoneme Segmentation Fluency (PSF) directly measures phonemic awareness. Nonsense Word Fluency (NWF) measures the alphabetic principle and basic phonics with VC/CVC pseudowords such as "sig" and "rav" — [Acadience Reading K-6 Fact Sheet](https://acadiencelearning.org/wp-content/uploads/2025/03/AcadienceReadingK-6_FactSheet.pdf); [Acadience Reading Crosswalk](https://acadiencelearning.org/wp-content/uploads/2026/01/Acadience-Reading-Crosswalk.pdf)
- Grade spans: FSF is given beginning to middle of K; PSF middle of K to beginning of grade 1; NWF middle of K to beginning of grade 2 (fact sheet). Other documents list FSF as K only, PSF as K-1, and NWF as K-2 (sources differ slightly) — [Acadience Fact Sheet](https://acadiencelearning.org/wp-content/uploads/2025/03/AcadienceReadingK-6_FactSheet.pdf); [California DoE review](https://www.cde.ca.gov/be/cc/rd/documents/rdrsspitem5adec2024.docx)
- Benchmarks are given 3 times a year (BOY/MOY/EOY). Acadience is criterion-referenced against research-derived benchmark goals — [Acadience Crosswalk](https://www.in.gov/doe/files/Acadience-Reading-Crosswalk.pdf); [DIBELS description, Colorado DoE](https://www.cde.state.co.us/accountability/dibels_assessment_instrument_description_072319)

**Israeli Ministry of Education Hebrew tools**
- The Ministry's Pedagogical Secretariat assesses reading fluency on accuracy, rate and prosody, using two tasks: reading a word list aloud and reading a story text aloud. It supports both class-level and individual mapping — [Ministry fluency assessment 2021](https://meyda.education.gov.il/files/Mazkirut_Pedagogit/HebrewPrimary/assessment/fluency-assessment-2021.pdf)
- There is a dedicated end-of-grade-2 fluency tool and a grade-3 fluency tool — [Fluency assessment end of grade 2](https://meyda.education.gov.il/files/Mazkirut_Pedagogit/HebrewPrimary/assessment/fluency-assessment-end-of-classB.pdf); [Ministry evaluation tools list](https://meyda.education.gov.il/files/Pop/0files/ivrit_hinuch_leshoni/Yesodi/evaluation-tools.pdf)
- Grade 1 has a reading-and-writing test, and the Ministry's list of start-of-grade-1 tools includes an early-literacy ("Yesodot") test and a writing test — [Ministry Reading-Writing Tools](https://meyda.education.gov.il/files/Pop/0files/ivrit_hinuch_leshoni/Yesodi/Reading-Writing-Tools-26.pdf); [evaluation-tools.pdf](https://meyda.education.gov.il/files/Pop/0files/ivrit_hinuch_leshoni/Yesodi/evaluation-tools.pdf)
- Ministry research describes the fluency measure as a valid and reliable predictor of reading performance — [Ministry gaps-conference article](https://meyda.education.gov.il/files/LishcatMadaan/gapsconference/article8.pdf)

**Hebrew-specific predictors (Haifa Edmond J. Safra cohort, n = 515)**
- In 515 Hebrew-speaking children (mean age 5.9), kindergarten letter knowledge (OR ≈ 4.75) and phonological awareness (OR ≈ 4.17) predicted grade 1 dyslexia. Letter knowledge, **morphological awareness** and RAN predicted grade 4 dyslexia over and above grade 1 status (Yinon et al., JAMA Network Open, 2026) — [HealthDay summary](https://www.healthday.com/healthpro-news/child-health/cognitive-linguistic-deficits-in-kindergarten-linked-to-dyslexia-risk); [INTED2025 abstract](https://library.iated.org/view/YINON2025PRE); [Safra Center](https://ejsafra.haifa.ac.il/wp-content/uploads/2022/01/research14.pdf) (the odds ratios come from the press summary and the abstract, not the full text)
- In Hebrew kindergarteners (n = 41), letter naming and executive functioning were the strongest predictors of phonological awareness mid-year, and letter naming was strongest at year end (Wasserstein & Lipka, 2019) — [Frontiers in Psychology 2019](https://www.frontiersin.org/journals/psychology/articles/10.3389/fpsyg.2019.01809/full)

**Early numeracy**
- Jordan et al.'s kindergarten number sense battery covers counting, number knowledge (including magnitude comparison), nonverbal calculation, story problems and number combinations. Kindergarten number sense plus its growth explained 66% of the variance in grade 1 math achievement — [IES REL Southeast summary](https://ies.ed.gov/ncee/edlabs/regions/southeast/aar/m_04-2017.asp); [PMC3655274](https://pmc.ncbi.nlm.nih.gov/articles/PMC3655274) (search snippet)
- As a screen, kindergarten number sense with at-risk cutoffs correctly ruled out 84% of children who did not later have calculation-fluency difficulties, but caught only 52% of those who did. That is good specificity and modest sensitivity — [PMC3935894](https://pmc.ncbi.nlm.nih.gov/articles/PMC3935894) (search snippet)
- Magnitude comparison appears in most operational definitions of number sense and in nearly all early math screeners (Berch, 2005, as cited) — [Screening for Mathematics Difficulties in K-3 (RTI report)](https://dwwlibrary.wested.org/files/5293918.pdf%3Frp_log_id=2xlt3B0e)
- Symbolic and non-symbolic magnitude processing were more tightly linked within the subitizing range (about 1-4). Non-symbolic processing at kindergarten entry predicted later symbolic development only within that range (Developmental Science, 2019) — [desc.12884 copy](https://www.scribd.com/document/508122918/10-1111-desc-12884)

**English as a foreign language: Cambridge Pre A1 Starters**
- Three papers. Listening: about 20 minutes, 4 parts / 20 questions. Reading & Writing: 5 parts, each with examples, correct spelling required, about 20 minutes. Speaking: 4 parts, child alone with an examiner, about 3-5 minutes. Results are reported in "shields" (max 5). The target age is 7-12 — [Cambridge English Starters test format](https://www.cambridgeenglish.org/exams-and-tests/starters/test-format/); [British Council Cyprus](https://www.britishcouncil.com.cy/exam/cambridge/english-levels/pre-a1-starters-yle)
- The digital version: Listening up to 40 minutes, 5 tasks / 20 questions; Reading & Writing up to 25 minutes, 5 tasks / 25 questions; Speaking 3-5 minutes, 4 tasks — [Exams Catalunya Starters Digital](https://exams-catalunya.com/en/exam/cambridge-pre-a1-starters-digital-yle/format/)

### Inferences
**Proposed skill map and item types per child.** Item types are adapted from the screeners above. Hebrew items use pointed (menukad) text through grade 2-3.

| Subject / strand | Girl 8.5 (grade 3) | Boy 5.5 (gan chova) | Toddler 2.5 |
|---|---|---|---|
| Hebrew phonological awareness | Phoneme deletion/manipulation ("say שמש without /ʃ/"), syllable counting with niqqud | First sound (FSF analogue: "which picture starts with /m/?"), rhyme, syllable clapping, final sound | none tested (play only) |
| Letter knowledge | not needed unless flagged | Letter-sound and letter-name tap-the-letter (22 letters + 5 final forms), timed letter naming for RAN-style evidence | letter "toys", exposure only |
| Decoding | Pointed and unpointed word reading; pseudoword reading (NWF analogue in Hebrew CV/CVC with niqqud); speed | CV syllable decoding with niqqud (e.g., בָּ, מִ) only if letter knowledge is present | none |
| Fluency | Read-aloud passage, WCPM + accuracy (Ministry grade-3 fluency structure: word list + story); prosody rated coarsely | n/a | n/a |
| Comprehension | Short text + literal/inferential questions (picture or multiple-choice), listening comprehension as a separate strand | Listening comprehension of a short story read aloud | parent report |
| Morphology (Hebrew-specific) | Root/pattern tasks (find the shared root in כתב/מכתב/כתבה), plural/gender agreement | Oral morphology games ("one ילד, many ___") | none |
| Writing / spelling | Dictation of words with homophonous letters (ט/ת, כ/ח, ק/כ, א/ע), sentence building | Write own name, letter tracing, phonetic spelling of a simple word | n/a |
| Math number sense | Place value to 1000s, number-line estimation 0-1000, fraction intro | Subitizing 1-5 (flash dots ~1 s), dot/digit magnitude comparison, counting on, number-line 0-10/0-20, what-comes-after | counting games only |
| Math operations / fluency | Multiplication facts (grade 3 Israeli focus), +/- within 1000, word problems (read aloud optional to separate reading from math) | Nonverbal addition/subtraction with objects, story problems told aloud, number combinations within 5-10 | n/a |
| English (EFL) | Pre A1-style: listen-and-point to pictures, letter-sound phonics (Israeli English formally starts around grade 3; check), basic vocabulary (colors, animals, numbers) | Listen-and-point vocabulary only, songs | exposure only |
| Science / general knowledge | Picture-sort categories, cause-effect "what happens next", simple prediction | Same at a simpler level, naming animals/body/weather | parent report |
| AI literacy | "Is this a robot or a person?", "computers can make mistakes", sorting tasks that teach patterns/classification | Sorting/pattern games, "robot follows instructions" sequencing | n/a |
| Fine motor | Letter formation quality for cursive (Israeli schools teach כתב יד) | Tracing paths, letter tracing (stroke order), drag precision | Drag/tap/pinch only (observational) |

- **Separate modalities to avoid confounds.** Read math items aloud by default for the 5.5-year-old (and optionally for the 8.5-year-old), so reading difficulty does not get scored as math difficulty. Score "listening comprehension" and "reading comprehension" as separate nodes. A gap between them (good listening, poor reading) is a classic decoding-difficulty signature (simple view of reading; design reasoning).
- **RAN on a tablet.** Classic RAN is naming a grid of familiar items (colors, objects, letters, digits) aloud as fast as possible. On iPad it could be done with speech recognition or with the parent tapping along. Even a rough version helps, because RAN predicted grade 4 Hebrew dyslexia in the Haifa cohort. Background knowledge, not verified this session.
- **The English level target for the 8.5-year-old** should be anchored to Pre A1 descriptors (picture-based, listening-first, minimal writing). Use a Starters-like picture vocabulary "listen and tap" task of 10-15 items as the EFL placement probe.

### Gaps
- I could not verify the content or availability of: the Shany/Breznitz tools; the "Aleph-Taf" (א-ת) reading/writing diagnostic battery (Shany, Lachman & Shalem, et al.; publisher and norms not checked); the Ministry's "מבדק אבחוני בקריאה כיתה א'" items and norms; Israeli WCPM norms for grade 3. The Ministry search found fluency tools but no national numeric benchmark table.
- TEMA-3 (Test of Early Mathematics Ability, ages 3-8) and AIMSweb Test of Early Numeracy (oral counting, number identification, quantity discrimination, missing number) were not checked this session. Their content is from background knowledge only.
- I found no item-level description of the Jordan screener's subitizing items.
- Whether English instruction formally begins in grade 1, 2 or 3 in Israel's current curriculum (it has shifted over the years) was not verified.

---

## 4. Toddler (2.5) assessment: parent report rather than testing (MB-CDI Hebrew, ASQ-3)

### Takeaway
At 2.5, valid "assessment" means structured parent report plus passive observation of play. Testing does not work at this age. The Hebrew MB-CDI is the established vocabulary checklist (web-normed on 881 toddlers, but only for ages 12-24 months). The ASQ-3 (1-66 months, 30 items, 6 per domain, 10-15 minutes for the parent) covers communication, gross motor, fine motor, problem solving and personal-social.

### Cited Findings
- Hebrew CDI: Maital et al. (2000) adapted the MB-CDI to Hebrew. The original sample was too small to establish norms — [Journal of Child Language: Hebrew CDI (Maital et al.)](https://www-cambridge-org.ezproxyberklee.flo.org/core/services/aop-cambridge-core/content/view/49F45CA24B467985EBA713DB65F11698/S0305000999004006a.pdf/the-hebrew-communicative-development-inventory-language-specific-properties-and-cross-linguistic-generalizations.pdf)
- Gendler-Shalev & Dromi built web-based Hebrew MB-CDI Words & Gestures norms on 881 typically developing toddlers (417 girls, 464 boys) aged 1;0-2;0. The form lists 428 words in 18 categories, and the authors recommend its growth curves as norms for the second year of life — [J Child Lang 2021, Hebrew Web MB-CDI growth curves](https://www.cambridge.org/core/services/aop-cambridge-core/content/view/80A8AFEEB4ED71E3682560A77FBD7941/S0305000921000179a.pdf/the-hebrew-web-communicative-development-inventory-mb-cdi-lexical-development-growth-curves.pdf); [Norms record](https://katalog.lib.cas.cz/KNAV/EdsRecord/edsair,edsair.doi...........33cd9b763571047e75fdc29c02e366e7)
- A CAT version of the CDI is feasible (simulated, 25-50 items, SE = 0.1 stop) — [RPubs CDI-CAT](https://rstudio-pubs-static.s3.amazonaws.com/1141036_591ab51a65ec4ce7abafcb12497df484.html)
- ASQ-3: age range 1-66 months; 30 items per questionnaire (6 per area) across communication, gross motor, fine motor, problem solving and personal-social; 10-15 minutes for the parent — [ASQ-3 At a Glance](https://llm.agesandstages.com/content/wp-content/uploads/2019/01/asq-3-at-a-glance-pdf.md); [AAP Screening Tool Finder: ASQ-3](https://aap.org/en/patient-care/screening-technical-assistance-and-resource-center/screening-tool-finder/ages-and-stages-questionnaire-asq-3). One source gives 20 minutes to administer — [Shirley Ryan AbilityLab](https://www.sralab.org/rehabilitation-measures/ages-and-stages-questionnaires)

### Inferences
- **Toddler spec:** (1) a one-time 5-minute parent questionnaire at setup: "does she say ~50+ words? 2-word combinations? follow 2-step instructions? name colors/animals? scribble/stack blocks?". Write these items in your own words, modeled on CDI/ASQ domains. **Do not copy ASQ-3 or MB-CDI items verbatim; both are copyrighted/licensed instruments.** (2) Passive play telemetry: tap accuracy, drag success, attention span per session, whether she responds to audio prompts ("where's the dog?"). (3) Re-ask a few parent questions every ~3 months.
- **Never score or label the toddler.** The app's role is enrichment (vocabulary, songs, cause-effect toys). The only "flag" is a gentle note to the parent: if the parent report suggests very limited expressive vocabulary or no word combinations around age 2;6, suggest discussing it at Tipat Halav or with the pediatrician, or a speech-language evaluation (קלינאית תקשורת). The Hebrew CDI norms stop at 24 months, so the app has no valid Hebrew norm for age 30 months. That is another reason to refer rather than score.
- **Session length for a 2.5-year-old:** very short (2-5 minutes). No timers, no right/wrong buzzers. Design reasoning; I found no source.

### Gaps
- No Hebrew MB-CDI Words & Sentences (16-30 months) norms were verified this session. I don't know if a Hebrew W&S normed form exists for 2;6.
- I could not confirm whether the ASQ-3 has an officially validated Hebrew translation and Israeli norms.
- I found no sources on licensing terms for digital use of ASQ-3 or the CDI in a consumer app. This needs checking (Brookes Publishing for ASQ; the MB-CDI is licensed via Brookes / the CDI Advisory Board).

---

## 5. Early flags for learning difficulties (dyslexia in Hebrew, dyscalculia, attention) and how to surface concerns responsibly in the Israeli system

### Takeaway
The most predictive kindergarten markers of later Hebrew dyslexia are weak letter knowledge, weak phonological awareness, and (for later, rate-based dyslexia) slow RAN and weak morphological awareness. For math, the markers are weak magnitude comparison, counting, subitizing and number combinations. Early math screens are specific but only moderately sensitive (52% hit rate), so one low score must never be treated as a diagnosis. The app should track persistent patterns over weeks, present them as "areas to watch", and point the parent to the Israeli pathway: the class teacher / school counselor (יועצת), then a didactic evaluation (אבחון דידקטי, age 6+, by a Ministry-recognised diagnostician) or psycho-didactic evaluation, or a speech-language evaluation for language and phonology.

### Cited Findings
- In the Haifa cohort, kindergarten letter knowledge (OR ≈ 4.75) and PA (OR ≈ 4.17) predicted grade 1 dyslexia. Letter knowledge, morphological awareness and RAN predicted grade 4 dyslexia beyond grade 1 status. The authors argue for a multi-component early screen — [HealthDay](https://www.healthday.com/healthpro-news/child-health/cognitive-linguistic-deficits-in-kindergarten-linked-to-dyslexia-risk); [INTED2025 abstract](https://library.iated.org/view/YINON2025PRE)
- An ongoing Haifa project (Gott, under David Share) tests whether kindergarten RAN deficits and PA/MA deficits dissociate and predict accuracy-based vs. rate-based reading problems. It is described as the first longitudinal dyslexia-subtyping study in Hebrew — [Edmond J. Safra Center seminar](https://ejsafra.haifa.ac.il/?p=28644)
- In older Hebrew readers (10-13), phonological awareness was the strongest predictor across reading measures — [Reading & Writing, s11145-005-5461-3](https://link.springer.com/doi/10.1007/s11145-005-5461-3)
- Kindergarten number-sense screening: 84% specificity, 52% sensitivity for later calculation-fluency difficulty — [PMC3935894](https://pmc.ncbi.nlm.nih.gov/articles/PMC3935894)
- Magnitude comparison is nearly universal in early math screeners. Within the subitizing range, non-symbolic magnitude at K entry predicts later symbolic skill — [RTI report](https://dwwlibrary.wested.org/files/5293918.pdf%3Frp_log_id=2xlt3B0e); [Dev Sci 2019](https://www.scribd.com/document/508122918/10-1111-desc-12884)
- **The Israeli pathway:** a didactic evaluation (אבחון דידקטי) identifies learning difficulties and strengths and is intended for children over 6. It should be done by a diagnostician recognised by the Ministry of Education (M.A. in learning disabilities and/or a diagnostician certificate). A psycho-didactic evaluation adds a psychologist. The Ministry funds evaluations for low-income families, and some local authorities run evaluation and support centres. Parents are advised to start with the school counselor (יועצת) or a local evaluation centre — [Kol Zchut: Didactic evaluation](https://www.kolzchut.org.il/he/אבחון_דידקטי); [Kol Zchut: Psycho-didactic evaluation](https://www.kolzchut.org.il/he/%D7%90%D7%91%D7%97%D7%95%D7%9F_%D7%A4%D7%A1%D7%99%D7%9B%D7%95%D7%93%D7%99%D7%93%D7%A7%D7%98%D7%99)
- Recent Ministry guidance allows some accommodations (e.g., extra time) without a professional evaluation; others require a documented school-committee decision based on an evaluation — [Ministry SHEFI circular on learning-disability accommodations](https://meyda.education.gov.il/files/shefi/liikoheylemida/Hathamot_Tashpag/Hozer_Nehalim_Tashpag.pdf) (as summarised in search results)

### Inferences
**Flag rules (proposed; persistent-pattern based, never from a single session):**
- **Hebrew reading risk, boy 5.5:** letter-sound knowledge well below expected for the second half of gan chova (e.g., <~10 of 22 letter-sounds by spring), plus weak first-sound / rhyme PA, persisting across 3+ weeks of practice → "watch" flag. Add slow letter/object naming speed (RAN-like) to the profile. **Note:** formal decoding instruction in Israel starts in grade 1, so poor decoding at 5.5 is NOT a flag by itself.
- **Hebrew reading risk, girl 8.5:** accuracy is fine but rate is very slow (rate-based profile), or persistent errors with visually or phonologically similar letters, or a large listening-vs-reading comprehension gap, or spelling errors that are not phonologically plausible → suggest talking to the teacher and the יועצת, and possibly a didactic evaluation (she is past the age-6 threshold).
- **Math, both:** a persistent subitizing failure for 1-4 dots, slow or inaccurate symbolic magnitude comparison (which is bigger, 7 or 4?), counting all instead of counting on at grade 3 level, or no retrieval of basic facts (all finger counting) despite practice → "watch" flag. With 52% sensitivity, also prompt the parent with observation questions.
- **Attention:** in-app data can only show patterns: a high rapid-guess rate, frequent idle timeouts, a steep accuracy drop late in a session, large response-time variability. The app should never mention ADHD by name or imply a diagnosis. Instead, adapt (shorter sessions, more breaks). Only if the parent independently raises concerns, suggest talking to the pediatrician or the school.
- **Fine motor / grapho-motor:** persistent difficulty tracing simple strokes at 5.5 → suggest the gan teacher, or an occupational therapy (ריפוי בעיסוק) evaluation via Kupat Holim. (Pathway from background knowledge; not verified this session.)
- **Language (any age):** weak vocabulary, poor listening comprehension, or (toddler) limited words → speech-language pathologist (קלינאית תקשורת), usually through Kupat Holim, or the gan's speech-therapy services. Background knowledge; not verified.

**How to surface concerns (responsible UX):**
- Use a parent-only area, behind a parental gate. The child never sees it.
- Wording: "an area we're watching", not "risk" or "disorder". Give concrete observations ("Noa is accurate but reads about X words/minute; most grade-3 readers in our data read faster") plus what the app is doing about it and suggested at-home activities.
- After N weeks of a persistent pattern despite targeted practice (an RTI-like logic of "response to intervention"), add: "consider sharing this with the teacher / school counselor; a didactic evaluation can clarify." Include a printable Hebrew summary the parent can bring.
- Use clear disclaimers: not a diagnostic tool, not normed on Israeli children (unless it becomes so), possible false alarms and misses.
- Privacy: keep the data on-device or in family-only storage. This is sensitive information about minors (Israeli Privacy Protection Law, amendment 13, and the App Store kids-category rules). Not researched this session.

### Gaps
- I found no validated Hebrew cut-scores for letter knowledge, PA or RAN in gan chova that an app could use. The Haifa studies report odds ratios, not public cut-points.
- I found no Hebrew-specific dyscalculia screening research this session.
- I did not verify the Kupat Holim referral process for speech and OT evaluations, or the role of the שפ"ח (educational psychological services) in the gan versus school years.
- I found no evidence on the accuracy of app-based ADHD-like attention indicators in children. Caution is warranted.

---

## 6. Making placement fun and low-anxiety (calibration as an adventure, no failure feedback, short sessions, parent observation questions, ending on success)

### Takeaway
The evidence base supports two levers. (1) Embed assessment in play (stealth assessment), because test framing demotivates lower performers. (2) Keep success rates high: Math Garden research looks specifically at how experiencing success affects math anxiety, and its adaptive engine keeps children at a target success rate. Commercial K-2 tests are untimed, use audio, and allow breaks. Concrete UX rules (no "wrong" buzzers during placement, multiple short sessions, end on an easy item) are design inferences consistent with these sources.

### Cited Findings
- Traditional tests can demotivate lower-performing students. Stealth assessment within games is offered as a remedy — [SA Primer](https://myweb.fsu.edu/vshute/pdf/SA_Primer.pdf)
- Klinkenberg et al. (2015) studied how experiencing success in math affects math anxiety and performance in Math Garden — [UvA profile](https://www.uva.nl/en/profile/k/l/s.klinkenberg/s.klinkenberg.html)
- MAP Growth K-2 is untimed with breaks encouraged, and offers audio support for K-2 — [NWEA durations](https://nwea.org/uploads/Average-MAP-Growth-Test-Durations.pdf); [NWEA audio](https://connection.nwea.org/articles/Knowledge/Do-all-MPG-test-items-have-audio)
- An adaptive design avoids wasting time on items far too hard or too easy — [Duolingo blog](https://blog.duolingo.com/is-the-duolingo-english-test-hard/)
- Cambridge Starters, built for 7-12-year-olds, uses picture-based tasks, examples before each part, a familiar adult to introduce the child in speaking, and positive "shields" reporting, not pass/fail — [Cambridge Starters format](https://www.cambridgeenglish.org/exams-and-tests/starters/test-format/); [British Council](https://www.britishcouncil.com.cy/exam/cambridge/english-levels/pre-a1-starters-yle)

### Inferences
**Placement UX spec ("drawing the adventure map" / "מיפוי המפה"):**
- **Narrative:** the guide character "needs help lighting up the map". Each answered item reveals a piece of the map, whether it is right or wrong. Correctness is never shown during placement: every answer gets the same neutral, positive animation ("thanks, the map grows!"). No score, no stars lost, no hearts/lives.
- **Structure:** 3-4 "expeditions" of 5-8 minutes over 2-4 days, with one subject or strand per expedition, so the 8.5-year-old does math one day and Hebrew the next. For the 5.5-year-old, use 3-5-minute expeditions, all audio-led, no reading needed for instructions.
- **Start easy:** begin 1 level below the age-typical level, giving 2-3 near-certain successes to warm up (these also help verify that the child understands the interaction). Use binary-search jumps after that.
- **Hard-item buffering:** after 2 consecutive misses, insert an easy "breather" item (it does not update θ much) and then continue. Always end an expedition on an item the model predicts is ≥90% likely to succeed.
- **"Too tricky" button:** shown as the guide character shrugging ("let's skip this one"). No penalty is shown to the child.
- **Parent observation questions** (2-3 minutes per child, at setup): "Does she read chapter books alone? Read without niqqud? Know multiplication tables up to 5/10? Does he know letter names / sounds? Count to 20? Write his name?" These set the prior for θ and cut the item count by roughly half (fewer binary-search steps). They are also cross-checked against performance; a large mismatch is itself informative.
- **No timers visible.** Latency is recorded silently, and speed is rewarded only in later fluency games, never in placement.
- **Separate children:** each child has their own profile and avatar, and siblings should not see each other's levels. This avoids comparison between the 8.5- and 5.5-year-olds.
- **Ending:** a reveal ("your map is ready!") with the first unlocked areas set at an easy level, so the first real lesson succeeds.

### Gaps
- I did not retrieve the effect sizes from the Math Garden success/anxiety study (Jansen et al., 2013, and Klinkenberg et al., 2015). Unverified background: lower target difficulty increased how often children played, without hurting learning.
- I found no direct experimental evidence comparing "no-feedback placement" with "feedback placement" for young children's anxiety.

---

## 7. Ongoing re-assessment and progress measurement (boss levels, growth reports, grade-level comparison) and a consolidated spec

### Takeaway
Industry practice combines continuous updating from everyday practice (IXL: 10-15 diagnostic questions a week; Math Garden: an update on every item) with periodic benchmark windows (Acadience/DIBELS: three times a year, BOY/MOY/EOY). Results are reported against grade-level expectations (IXL maps levels to grade; Acadience is criterion-referenced to benchmark goals). Elo's step size must stay flexible because children's ability changes quickly. For a home app: continuous stealth updating, monthly "boss levels" as short formal checkpoints, and seasonal benchmark windows aligned to the Israeli school year.

### Cited Findings
- IXL recommends 10-15 diagnostic questions a week to keep levels current, shows a range until the level is pinpointed, and maps scores to grade (500 = ready for grade 5) — [IXL Diagnostic guide](https://au.ixl.com/materials/i_guides/IXL_for_Diagnostic_Assessment.pdf)
- Acadience benchmarks three times a year (BOY/MOY/EOY), criterion-referenced to benchmark goals, with progress monitoring in between — [Acadience Crosswalk](https://www.in.gov/doe/files/Acadience-Reading-Crosswalk.pdf); [Acadience Fact Sheet](https://acadiencelearning.org/wp-content/uploads/2025/03/AcadienceReadingK-6_FactSheet.pdf)
- Math Garden combines adaptive practice and monitoring in one self-organising tool. Teachers get progress automatically — [IAEA: Combining computerized adaptive practice and monitoring](https://iaea.info/?p=8114)
- Decreasing-K Elo is inadequate when ability fluctuates. A dynamic K is proposed — [Tilburg: dynamic K](https://research.tilburguniversity.edu/en/publications/balancing-stability-and-flexibility-investigating-a-dynamic-k-val/)
- Israeli Ministry fluency tools exist for the end of grade 2 and for grade 3 (word list + story; accuracy, rate, prosody) and support individual mapping — [Ministry fluency 2021](https://meyda.education.gov.il/files/Mazkirut_Pedagogit/HebrewPrimary/assessment/fluency-assessment-2021.pdf); [end of grade 2](https://meyda.education.gov.il/files/Mazkirut_Pedagogit/HebrewPrimary/assessment/fluency-assessment-end-of-classB.pdf)
- Kindergarten number sense **growth**, not only status, contributes to predicting grade 1 math (66% variance with status + growth) — [IES REL Southeast](https://ies.ed.gov/ncee/edlabs/regions/southeast/aar/m_04-2017.asp)

### Inferences
**Re-assessment cadence (proposed):**
- **Continuous:** every scored item updates node θ through Elo with dynamic K. Mastery of a node is declared when θ − item difficulty gives P(correct) ≥ 0.85 across ≥2 sessions on ≥2 different days (spaced, to guard against same-day cramming).
- **Spaced retention probes:** 1-2 review items from mastered nodes per session (about 10% of items). A failure reopens the node, which keeps the estimate honest.
- **"Boss levels" (monthly, ~5 minutes per subject):** a mixed-skill challenge drawn from recently mastered and next-up nodes, scored without hints. This is the app's formal progress-monitoring point. Frame it as a celebration ("defeat the dragon of the 7× table"). A loss just means "come back stronger": no lost progress, and the boss returns later.
- **Seasonal benchmarks (3 times a year, aligned to the Israeli school year: September-October, January-February, May-June):** for the grade-3 girl, a short read-aloud fluency probe modeled on the Ministry's word list + story structure (WCPM, accuracy), plus a math checkpoint. For the gan chova boy: letter-sound knowledge, PA and number sense (counting, subitizing, comparison), which matches the predictors above. Before grade 1 entry (summer), run a "ready for first grade" checkpoint.
- **Growth reports for parents:** per subject, a level band (range until precise, like IXL), growth since last month or season, "next 3 skills", and the time-on-task trend. Compare to grade expectations as "on track / ahead / building", based on the app's own mapping of nodes to the Israeli curriculum grade (תכנית הלימודים). Do not invent percentiles without norms.
- **Re-placement triggers:** after more than 3 weeks inactive (summer break), run a light 5-item warm-up re-probe per active strand (with an increased K), because skills may have decayed. If a child succeeds at >95% for 2 weeks, jump them ahead with a binary-search probe up the graph. If they succeed at <60% for a week, probe down.

**Consolidated assessment spec (summary for the report writer):**
1. **What to measure:** per child, per subject strand, a node-level θ + SE in a prerequisite skill graph. Strands: Hebrew (PA, letter knowledge, decoding, fluency, morphology, listening comprehension, reading comprehension, spelling/writing); math (subitizing, counting, magnitude comparison/number line, place value, operations, fact fluency, word problems); English (listening vocabulary, phonics, simple reading); science/general knowledge (vocabulary/categories, cause-effect); AI literacy (patterns/classification, "machines can be wrong", instructions/sequencing); fine motor (tracing quality). Plus learner-state variables (engagement, persistence).
2. **Item types per age:** see the table in section 3. The 5.5-year-old's items are all audio-led, picture or tap based, with tracing. The 8.5-year-old adds reading items, read-aloud with ASR, dictation, and number line. The 2.5-year-old gets a parent questionnaire plus passive play telemetry only.
3. **How many items:** placement ≈ 10-15 items per strand for the 8.5-year-old and 6-10 for the 5.5-year-old, about 40-60 items in total per child across 3-4 short sessions. Fewer if parent priors are strong (KST log2 logic; ALEKS Adventure caps of 9 for K and 14 for grades 1-3). Ongoing: about 10-15 scored items a week per subject, built into play (IXL benchmark).
4. **Algorithm:** age/grade + parent-report prior → binary search on the skill graph → Elo/Rasch refinement with dynamic K and SE stopping (~0.45) → continuous stealth updates with telemetry filters (exclude rapid guesses and timeouts; speed only for fluency nodes).
5. **Game feel:** map-revealing narrative, no correctness feedback in placement, short expeditions, easy start and easy end, a "too tricky" skip, no visible timers, separate sibling profiles.
6. **Early detection:** persistent multi-week patterns on research-backed markers (letter knowledge, PA, RAN, morphology for Hebrew reading; subitizing, magnitude comparison, counting strategy, fact retrieval for math). Parent-only, non-diagnostic wording, with referral to teacher/יועצת → אבחון דידקטי (6+), קלינאית תקשורת, ריפוי בעיסוק, or the pediatrician.
7. **Re-assessment:** continuous, plus monthly boss levels, plus 3 benchmark windows a year on the Israeli school calendar, plus re-probes after breaks.

### Gaps
- I found no Israeli grade-level numeric norms (e.g., grade-3 WCPM in Hebrew) to anchor "on track". The app will need to rely on curriculum-based criteria or gather its own data.
- I did not verify the Israeli Ministry's national assessments (RAMA Meitzav, grades 2/5/8) for the timing of grade-level expectations, or the current Israeli math curriculum milestones by grade (e.g., when the multiplication table is expected to be mastered).
- Validity of ASR (speech recognition) for children's Hebrew read-aloud scoring was not researched. This is a key technical risk for fluency measurement.
