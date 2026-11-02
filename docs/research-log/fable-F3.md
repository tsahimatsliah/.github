# Research log — Fable review F3 (specs 03 math, 04 Hebrew)

> Session 2026-11-02. Targeted checks only; the hosts `pop.education.gov.il`, `meyda.education.gov.il` and `hashbacha.org` are outside the session allowlist (DNS refused), so Ministry PDFs and the Hashbacha year plan were read **only through search-result summaries**. Grades as in `math.md`: A/B/C/BK/I.

## R1. Grade-3 math order under the new curriculum (תשפ"ז)

- **Found:** Hashbacha published "פריסת תוכנית הלימודים במתמטיקה לכיתה ג' לפי תוכנית הלימודים החדשה, תשפ"ז" (`hashbacha.org/wp-content/uploads/2026/08/…כיתה-ג-1.pdf`). The search summary says the arithmetic sequence **opens with representations of numbers up to 10,000 and addition/subtraction within 10,000**, that geometry is taught **throughout the year alongside arithmetic**, and that the plan assumes **≥5 weekly hours, one of them geometry**. The old grade-3 program (`kita3.pdf`, `math.md` R3) also opens with "המספרים הטבעיים בתחום הרבבה". **Grade C** (publisher plan via search summary; not the Ministry milestones document).
- **Not found:** whether column multiplication, division with remainder and the fraction scope stay in grade 3 (the Ministry grade-3 page and `curriculum-third-grade.pdf` could not be opened). `math.md` R3's "I" tags stand.
- **Decision (03 §7.2):** moved "numbers to 10,000" (`M.PV.THOU10K`… `M.NUM.ROUND`) from February to October–November; February became "column add/sub to 10,000 + word problems". Marked ⟂VERIFY. The 30%-geometry parallel strand (03 §7.2, §13.3) is consistent with "one hour of five". Since the app starts in November, units 01–02 are `presumedTaught` and are checked in placement rather than taught.
- **Sources:** [Hashbacha תשפ"ז grade-3 plan (PDF)](https://hashbacha.org/wp-content/uploads/2026/08/%D7%A4%D7%A8%D7%99%D7%A1%D7%AA-%D7%AA%D7%95%D7%9B%D7%A0%D7%99%D7%AA-%D7%94%D7%9C%D7%99%D7%9E%D7%95%D7%93%D7%99%D7%9D-%D7%9C%D7%9E%D7%95%D7%A8%D7%99-%D7%94%D7%A9%D7%91%D7%97%D7%94-%D7%AA%D7%A9%D7%A4%D7%96-%D7%9B%D7%99%D7%AA%D7%94-%D7%92-1.pdf); [Ministry: new curriculum page](https://pop.education.gov.il/tchumey_daat/matmatika/yesodi/oraat-math/new-curriculum/); [Ministry: elementary math curriculum page](https://pop.education.gov.il/tchumey_daat/matmatika/yesodi/oraat-math/tohnit-limudim/); [Ministry inspector circular 5787 (PDF, unread)](https://meyda.education.gov.il/files/Mazkirut_Pedagogit/math/primary-school/math2025/mafmar_yesodi_tashpaz.pdf).
- **Re-run when allowed:** open the Hashbacha PDF and list units with weeks; open `curriculum-third-grade.pdf` and confirm column multiplication / remainder / fractions.

## R2. Letter sounds before names (04 P2) — re-check

- **Found:** Levin, Shatil-Carmon & Asif-Rave (2006, *JECP* 93, 139–165) exists as cited; a Hebrew-literacy review notes Israeli classroom practice emphasises names over sounds, so children know names better than sounds, and that most Hebrew letter names are acrophonic and longer (≥3 phonemes). Treiman, Levin & Kessler (2007) found fewer phonologically based confusions than in English. **Grade B/C** (abstracts and a review, not the 2006 full text).
- **Decision:** 04 P2 stands (sound as primary label; name secondary from block ב). No change.
- **Sources:** [WUSTL: Linking shapes of letters to sounds — Hebrew](https://profiles.wustl.edu/en/publications/linking-the-shapes-of-alphabet-letters-to-their-sounds-the-case-o/); [PMC3278080](https://pmc.ncbi.nlm.nih.gov/articles/PMC3278080); [Applied Psycholinguistics: letter names in Hebrew](https://core-prod.cambridgecore.org/core/journals/applied-psycholinguistics/article/letter-names-effect-on-letter-saying-spelling-and-word-recognition-in-hebrew/F5EA025ABE16F199E59C984E188A78B6).

## R3. Pointed → unpointed transition and partial vowelization (04 §6)

- **Found:** Schiff 2012 (grades 2/4/6): grade 2 reads vowelized text more accurately and faster; by grade 4 unvowelized reading has developed; the vowelized-accuracy mediation disappears by grade 6. Shany, Bar-On & Katzir 2011: children gradually begin using unpointed script **by grade 3**. Bar-Kochva & Breznitz's triplex model: the transition shifts reliance from vowel points to morphological and lexical knowledge. A dyslexia study (grade 4) found no vowelized/unvowelized difference for dyslexic readers, suggesting the transition differs for struggling readers. **No study of partial vowelization for children** was found. **Grade B** for the developmental shift; **I** for the fade schedule.
- **Decision:** 04 §6 stays adaptive per word with gates, morphology as the lever (§10), and stop rules; added explicit regression rules (§6.4) and a `watch` flag (`HEB_K1_FADE_STALL`). The dyslexia finding supports "freeze fade at F0 for a child far behind" (04 §2.1א).
- **Sources:** [Schiff 2012, BIU](https://education.biu.ac.il/en/node/4541); [Shany, Bar-On & Katzir 2011 (Springer)](https://springerlink.bibliotecabuap.elogim.com/content/pdf/10.1007/s11145-011-9314-y.pdf); [Triplex model](https://www.bohrium.com/paper-details/learning-to-read-a-semitic-abjad-the-triplex-model-of-hebrew-reading-development/813069170513018881-9328); [Haifa: role of vowel signs](https://cris.haifa.ac.il/en/publications/the-role-of-vowel-signs-in-hebrew-beyond-word-recognition/).

## R4. Hebrew numerals and pointing (03 §10.4), native-speaker review

- Checked by reading, not by search (budget): the base table 1–10 (f/m, absolute/construct), teens, tens, hundreds, thousands and ordinals are correct. Fixes applied: the vav-conjunction rule (no וָ inside numerals; וּ before shva and בומ"ף; וַ before chataf), "עשרים שנה" allowed by the Academy but the app standardises on plural, "חֲלָקֵי" (normative) vs "חֶלְקֵי" (colloquial) marked ⟂VERIFY against the Academy term bank before recording. Nikud in 04's word lists: three errors fixed (הַחָתוּל→הֶחָתוּל; אִישָׁה→אִשָּׁה; גָּשָׁם→גֶּשֶׁם, moved to stage D because of /e/), two "pseudowords" that are real words replaced.

## Open re-verification queue (adds to `math.md` R13 and `hebrew.md`)

1. Hashbacha grade-3 plan: unit order and weeks (R1).
2. Ministry `curriculum-third-grade.pdf`: column multiplication, remainder, fractions in grade 3 or 4.
3. Academy of the Hebrew Language: normative reading of ":" (חֲלָקֵי / חֶלְקֵי).
4. Two consecutive pointed lines at line-height 1.55–1.6 in Noto Sans Hebrew (09 font gate; 04 §7.4).
