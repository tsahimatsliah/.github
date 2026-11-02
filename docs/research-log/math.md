# Research log: math curriculum (spec 03)

> This log backs `docs/spec/03-curriculum-math.md`. Each entry gives the question, what was found, the source, an evidence grade, and the decision it supports.
> Grades: **A** = RCT/meta-analysis · **B** = small study, official document or primary design source · **C** = secondary source (publisher, school site, search summary of an official document) · **BK** = the author's background knowledge of the published literature, not re-verified in this session · **I** = design inference.
> **Session constraints (2026-11-02):** these hosts all refused the connection or failed DNS, so their pages could not be read: `meyda.education.gov.il`, `pop.education.gov.il`, `hashbacha.org`, `ymath.haifa.ac.il`, `portal.macam.ac.il`, `sites.google.com`, `math-center.org`, `matic.co.il`, `web.archive.org` and Google cache. Only WebSearch result summaries were available. The shared WebSearch budget ran out partway through, so several planned searches (fact-strategy sequences, misconception catalogs, a check of Hebrew math vocabulary) were not run. Those sections rely on **BK** and are marked as such. To verify the Israeli curriculum line by line, `meyda.education.gov.il` and `pop.education.gov.il` must be allowed in the session's network settings. This is the single most valuable open research task (it matches `gap_audit.md` §3 G3).

---

## R1. Does grade 3 really move to the new curriculum in 5787?

- **Found:** yes. The Ministry's elementary math curriculum page says the gradual replacement began in 5785. In 5786 grades 1–2 learned on the new curriculum and grades 3–6 stayed on the old one, and "בשנת הלימודים תשפ"ז תלמידי כיתה ג' יצטרפו". The program's principles are a balance between computational fluency and deep understanding, mathematical discourse, and critical and creative thinking. — [pop: תוכנית הלימודים במתמטיקה ליסודי](https://pop.education.gov.il/tchumey_daat/matmatika/yesodi/oraat-math/tohnit-limudim/); [pop: התוכנית החדשה](https://pop.education.gov.il/tchumey_daat/matmatika/yesodi/oraat-math/new-curriculum/) (search summaries). **Grade C** (official source, read through a search summary).
- **Implication:** K1 is in the **first grade-3 cohort** on the new program. In 5786 she studied grade 2 on the new program too, so her grade 2 knowledge base matches the new curriculum.
- **Decision:** the year plan for grade 3 is a skeleton that the "this month in class" input re-orders (spec §7). Placement checks the grade 2 base, because grade 2 was also new material.

## R2. Which documents define the grade 3 topics?

- **Found:** every grade has a separate "ציוני דרך" (milestones) document, and there is a grade 3 file `curriculum-third-grade.pdf`. The milestones document "is based on the curriculum and includes its components: topics, teaching hours, clarifications and examples." The new grade 1 curriculum is at `math2023/Newprogramgrade1.pdf` and `curriculum-first-grade.pdf`. — [pop curriculum page](https://pop.education.gov.il/tchumey_daat/matmatika/yesodi/oraat-math/tohnit-limudim/); [curriculum-third-grade.pdf](https://meyda.education.gov.il/files/Pop/0files/matmatika/Yesodi/pedagogya/curriculum-third-grade.pdf) (not readable); [Newprogramgrade1.pdf](https://meyda.education.gov.il/files/Mazkirut_Pedagogit/math/primary-school/math2023/Newprogramgrade1.pdf) (search summary). **Grade C.**
- **New grade 1 (search summary):** numbers to 100 (about 45 hours, with counting to 100 practiced all year); addition and subtraction within 20, built up gradually from the 10 range; whole tens by analogy (20 + 60 = 80 because 2 + 6 = 8). **Grade C.**
- **Decision:** grade 1 content forms the K2 "readiness / G+" layer. K2 skills go up to counting to 100 and +/- within 10 as stretch goals, not requirements.

## R3. What is in grade 3?

- **Ministry emergency-learning page (grade 3 topic list, search summary):** ordering numbers (סדר בין מספרים); knowing the numbers up to 10,000 (הכרת המספרים עד 10,000); sorting triangles (מיון משולשים); area and perimeter of rectangles, grades 3–4 (שטח והיקף מלבנים). — [pop emergency-learning](https://pop.education.gov.il/tchumey_daat/matmatika/yesodi/oraat-math/emergency-learning/). **Grade C.**
- **Ministry estimation page:** a unit "כיתה ג - מתחילים שנה עם חוש למספרים", which makes number sense and estimation an explicit theme for the start of the year. — [pop omdan-tovana-misparit](https://pop.education.gov.il/tchumey_daat/matmatika/yesodi/noseem_nilmadim/omdan-tovana-misparit/). **Grade C.**
- **Old grade 3 curriculum (kita3.pdf):** opens with "המספרים הטבעיים בתחום הרבבה (10,000), א. המבנה העשרוני". A search summary of the same document mentions vertical (column) multiplication, division with remainder and angles. — [kita3.pdf](https://meyda.education.gov.il/files/mazkirut_pedagogit/matematika/tochnyotlemud/kita3.pdf). **Grade C.**
- **Test and worksheet sites for grade 3:** separate units for numbers to 10,000, vertical multiplication, division with remainder, fractions, angles and perimeter/area. — [מט"ח מבדקים](http://www.math.maty.manhi.org.il/BRPortal/br/P102.jsp?arc=919159); [KidsPlus grade 3](https://www.kidsplus.co.il/worksheets/third/multi-c/); [Math Center: חילוק עם שארית ג'](https://math-center.org/he-IL/learning/3rd/division-with-remainders/); [tirgulon](https://tirgulon.co.il/class-C.htm). **Grade C** (they follow the old program).
- **Earlier notes:** the multiplication-table focus (completing and consolidating it, with the distributive law), the laws of operations, ×10/×100, simple fractions and "חקר לוח הכפל" (`math_pedagogy.md` §1). **Grade C.**
- **Unresolved:** whether the new program moves column multiplication or division with remainder to grade 4, and the exact fraction scope. **Decision:** both are tagged `S` (grade 3) with confidence "I". The class-input mechanism (§7) and the photo pipeline correct the order. Column multiplication 2-digit×1-digit is placed at the end of grade 3, with mental distributive multiplication first, so the order is pedagogically right whichever grade the school uses.

## R4. Grade 2 content (for "review gaps")

- **Found:** grade 2 milestones cover operations within 100, about 18 hours for horizontal addition and subtraction, mastery of the addition/subtraction facts up to 9+9, and adding and subtracting whole tens within 100. They also include exploration and estimation tasks (compare exercises without solving them). — [march2026.pdf](https://meyda.education.gov.il/files/Mazkirut_Pedagogit/math/primary-school/math2025/march2026.pdf); [B.Peulot.pdf](https://meyda.education.gov.il/files/mazkirut_pedagogit/matematika/tochnyotlemud/B.Peulot.pdf) (search summaries). **Grade C.**
- **Central District grade 2 plan (old program, 5778):** the meaning of multiplication and division, the multiplication table up to 30, and multiplication and division up to 60. The Hashbacha grade 2 series spends about 38 hours on multiplication and division, including ×5 and ×10. — [tl-b.pdf](https://meyda.education.gov.il/files/merkaz/new/Disciplines/Math/tl-b.pdf); [Hashbacha grade 2](https://hashbacha.org/wp-content/uploads/2023/08/) (search summaries). **Grade C.**
- **Decision:** the K1 review layer `S-R` includes facts within 20, +/- within 100, the meanings of ×/÷ and ×2/×5/×10. Grade 3 completes the table.

## R5. The gan curriculum

- **Found:** three strands: מושג המספר; תחושה מרחבית וגיאומטריה; מושגים כמותיים בחיי היום-יום. Problem solving runs through all of them. Draft core document: `kdamyesodi/math1.pdf`; also `TochnitKdamYesodiHeb.pdf`. — [math1.pdf](https://meyda.education.gov.il/files/tochniyot_limudim/kdamyesodi/math1.pdf); [TochnitKdamYesodiHeb.pdf](https://meyda.education.gov.il/files/Mazkirut_Pedagogit/Matematika/TochnitKdamYesodiHeb.pdf) (search summaries). **Grade C.**
- **Not found:** the official per-strand detail (counting ranges, symbols). **Decision:** levels follow Clements & Sarama (R6), tagged to the three strands.

## R6. Clements & Sarama learning trajectories

- **Search confirmed** the framework (goal + developmental progression + activities). The arithmetic levels include Find Result, Find Change/missing addend (5 + _ = 7) and Part-Whole, up to flexible strategies. An LT-vs-teach-to-target RCT (ZDM 2020) favored the LT approach for addition and subtraction. — [LearningTrajectories.org counting PDF](https://www.learningtrajectories.org/documents/1596923897613.pdf); [ZDM 2020](https://link.springer.com/article/10.1007/s11858-019-01122-z); [DREME](https://dreme.stanford.edu/publication/efficacy-of-a-learning-trajectory-approach-compared-to-a-teach-to-target-approach-for-addition-and-subtraction/). **Grade B.**
- **Level names and typical ages (BK, from Clements & Sarama, *Learning and Teaching Early Math*, 2nd edition 2014; verify against the 4th edition):**
  - **Counting:**
    - Chanter (2); Reciter (3); Reciter (10) (3–4)
    - Corresponder (3); Counter (Small Numbers) (3); Producer (Small Numbers) (3–4)
    - Counter (10) (4); Counter and Producer (10+) (4); Counter Backward from 10 (4)
    - Counter from N (N+1, N−1) (5); Skip Counter by 10s (5–6)
    - Counter to 100 (6); Counter On Using Patterns (6); Skip Counter (6); Counter of Imagined Items (6); Counter On Keeping Track (6)
    - Counter of Quantitative Units / Place Value (7); Counter to 200 (7); Number Conserver (7)
  - **Subitizing:** Pre-Subitizer (2); Maker of Small Collections (3); Perceptual Subitizer to 4 (3–4); Perceptual Subitizer to 5 (4); Conceptual Subitizer to 5 (5); Conceptual Subitizer to 10 (6); Conceptual Subitizer to 20 (6); with place value and skip counting (7); with place value and multiplication (8).
  - **Comparing:** Perceptual Comparer (2); Matching Comparer (3); Counting Comparer (same size) (4); Counting Comparer (5) (4); Ordinal Counter (5); Mental Number Line to 10 (5–6); Counting Comparer (10) (5); Mental Number Line to 100 (7); Mental Number Line to 1000s (8).
  - **Adding and subtracting:** Nonverbal +/− (2–3); Small Number +/− (3); Find Result +/− (4); Find Change +/− (5); Make It N (5); Counting Strategies +/− (6); Part-Whole +/− (7); Numbers-in-Numbers (7); Deriver +/− (7); Problem Solver +/− (8).
  - **Composing number:** Pre-Part-Whole Recognizer (2); Inexact Part-Whole (4); Composer to 4, then 5 (5); Composer to 7 (6); Composer to 10 (7); Composer with Tens and Ones (7).
  - **Shapes:** Shape Matcher (2); Shape Recognizer–Typical (3); Shape Recognizer–circles, squares, triangles (4); Constructor of Shapes from Parts–Looks Like (4); Shape Identifier (5); Angle Recognizer (5); Shape Recognizer–All Rectangles (6); Side Recognizer / Parts of Shapes Identifier (6–7); Shape Class Identifier and Shape Property Identifier (7–8); Angle Size Comparer (8).
  - **Composing shapes:** Pre-Composer (2); Piece Assembler (3–4); Picture Maker (4); Shape Composer (5); Substitution Composer (6); Shape Composite Repeater (7); Shape Composer–Units of Units (8).
  - **Patterning:** Pre-Patterner (2); Pattern Recognizer (2); Pattern Fixer (3–4); Pattern Duplicator AB (3–4); Pattern Extender AB (4); Pattern Duplicator (4); Pattern Extender (5); Pattern Unit Recognizer (7).
  - **Length:** Length Quantity Recognizer (3); Length Direct Comparer (4); Indirect Length Comparer (5); Serial Orderer to 6+ (5); Length Unit Iterater (6); Length Unit Relater (7); Length Measurer (7); Conceptual Ruler Measurer (8).
- **Decision:** every skill in spec 03 carries a `C&S` tag with the nearest level, and the K3/K2 age bands follow these ages. **BK. Verify.**

## R7. Number-line board games (Siegler & Ramani)

- The earlier notes already cover this (`math_pedagogy.md` §3, §5): four 15-minute sessions; a linear board beats a circular one; colors in place of numbers give no effect; WWC rates it as meeting standards; effects persisted after 9 weeks (≈0.55–0.80, from a secondary source); counting on from the current square is better (Laski & Siegler 2013); a 2025 meta-analysis covers 18 studies. **Grade A/B.**
- **Decision:** `MECH.PATH_RACE` with a dosage rule. For each new range (0–10, 0–20, 0–100), run at least 4 races of about 5 minutes over 2 weeks. Saying the numbers aloud on every move is required (counting on, never counting from 1).

## R8. Fact-fluency strategy sequences (BK, search budget ran out)

- **Addition (Baroody; Henry & Brown 2008; Kling & Bay-Williams, *Math Fact Fluency* 2019, BK):**
  - Foundational facts first: +0, +1, +2, doubles, combinations of 10, 10+n.
  - Then derived facts: near-doubles, make-10 (bridging through 10), +9/+8 by compensation.
  - Subtraction through "think addition", then take-from-ten and back-through-ten.
- **Multiplication (Kling & Bay-Williams; Van de Walle, BK):**
  - Foundational: ×2, ×10, ×5, ×1, ×0 (and squares).
  - Derived: ×4 (double double), ×3 (×2 + one group), ×6 (×5 + one group, or double ×3), ×9 (×10 − one group), ×8 (double ×4), ×7 (×5 + ×2).
  - Commutativity halves the set, so 55 facts remain from 100. After the foundational facts, 21 facts with both factors in {3,4,6,7,8,9} remain (15 pairs + 6 squares; my own arithmetic).
- **Timed tests and anxiety:** see `math_pedagogy.md` §4 (Boaler vs. Hechinger; NCTM 2014). **Grade C.**
- **Decision:** untimed by default; response time is measured silently and only after accuracy is mastered (gap_audit D11); an opt-in "beat my record" mode; one strategy at a time; interleaving only after each set is ≥70% (`learning_science_workbooks.md` §7-12).

## R9. Misconception catalogs (BK, search budget ran out)

- **Sources (BK):**
  - Ashlock, *Error Patterns in Computation* (systematic "bugs": smaller-from-larger, zero errors, carry errors)
  - Brown & Burton 1978 (BUGGY)
  - Ryan & Williams 2007, *Children's Mathematics 4–15*
  - Kamii (place value, "the 1 in 16")
  - Knuth et al. 2006 (operational view of the equal sign: 8 + 4 = _ + 5 → 12)
  - Ni & Zhou 2005 (whole-number bias in fractions)
  - Carpenter, Fennema et al., CGI (problem types, the keyword trap)
  - Clements & Sarama (shape prototypes)
  - Kamii & Clark (rulers starting at 1, counting ticks)
  - Fuson (teen numbers)
- **Hebrew-specific (BK + I):**
  - Teen numbers are said "three-ten" (שלוש-עשרה), which leads to writing 31 for 13. This is the same direction as English, but the gendered forms add load.
  - Numbers with zeros in the middle (אלפיים וחמש → 25 / 2005 / 20005).
  - Mixing "פי" (times) with "יותר" (more than).
  - Division written with ":" in Israeli schools (BK; verify against a photographed worksheet).
- **Decision:** a global error catalog `E.*` (spec §4) with a deterministic distractor function for each error, plus a remediation path `R.*`.

## R10. Hebrew numerals in problem text

- The earlier notes already cover this (`hebrew_first_language_voice.md` §7):
  - polarity (masculine noun → the ‑ה form);
  - construct forms (שלושת);
  - the bare counting form is feminine;
  - שני/שתי before a noun, אחד/אחת after it;
  - teens and construct forms need care;
  - speakers neutralize the gender distinction (LVC study);
  - LLMs make numeral-gender errors.

  **Grade B/C.**
- **Decision:** spec 03 §10 defines a noun registry with gender, a numeral lookup table with nikud (only 1–10 written out by hand; the rest generated by rules → Nakdan → human check, B4/B5), and display text with digits / spoken text with words.

## R11. School calendar 5787

- **Holiday dates:** taken from `research-log/game-design.md` R1 (own calculation, cross-checked): Rosh Hashana 12.9.2026; Sukkot 26.9; Hanukkah 5–12.12.2026; Tu BiShvat 23.1.2027; Purim 23.3.2027; Pesach 22.4.2027; Yom HaAtzmaut 12.5.2027; Shavuot 11.6.2027. **Grade B.**
- **School vacations (third-party sites, they conflict):**
  - Hanukkah break 6–12.12.2026 (some sites say from 5.12);
  - Purim 23–24.3.2027;
  - Pesach break 13.4–28.4.2027, back 29.4;
  - end of elementary school 30.6.2027 according to one site, 20/21.6 according to another.

  — [mako](https://www.mako.co.il/home-family-kids/Article-df5ebe37893f281026.htm); [israelhayom](https://www.israelhayom.co.il/vacation-calendar); [edugo](https://edugo.co.il/vacations). **Grade C.**
- **Decision:** the year plan is organized by month, and vacations come from `calendar.schoolBreaks` (parent-editable, as in spec 01).

## R12. Hebrew math vocabulary (BK)

- **Grade 1–4 terms (BK, Israeli textbooks):**
  - Operations: מחובר, סכום, מחסר, מחוסר, הפרש, גורם, מכפלה, מחולק, מחלק, מנה, שארית.
  - Number structure: ספרה, יחידות, עשרות, מאות, אלפים, רבבה, ישר המספרים, זוגי/אי-זוגי.
  - Laws: חוק החילוף, חוק הקיבוץ, חוק הפילוג, סדר פעולות חשבון.
  - Fractions: שבר, מונה, מכנה.
  - Geometry: מצולע, צלע, קודקוד, זווית ישרה/חדה/קהה, אלכסון, מקבילים, מאונכים.
  - Measurement: היקף, שטח.
  - Writing and comparing: מאוזן, מאונך, גדול מ-/קטן מ-.
  - In the gan: כמה, יותר, פחות, שווה, ראשון/אחרון, לפני/אחרי, מעל/מתחת, ליד, בין, ימין/שמאל, ארוך/קצר, כבד/קל, מלא/ריק.
- **Decision:** a glossary with nikud in spec 03 §11. Each term has a skill ID where it is taught, and an audio clip prepared ahead of time.

## R13. Gaps carried forward

1. The ציוני דרך for grade 3 in the new program: not read. This needs network allow-listing, or a parent photo of the textbook's table of contents (spec §16, item 1).
2. Multiplication sign · or ×, and division ":" in the K1 classroom: BK. The app auto-detects it from the first photographed worksheet, with `notation.mulSign` as the default.
3. No validated Israeli norms for fact fluency (RT). The 3-second threshold is a BK convention (Kling & Bay-Williams / Van de Walle) and must be tuned on data.
4. Clements & Sarama ages: BK, needs checking against the book.
5. No Hebrew-specific research found on misconceptions or dyscalculia.
