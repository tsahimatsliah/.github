# 05 — תוכנית לימודים וגרף מיומנויות: אנגלית (EFL)

> **סטטוס:** טיוטה ליישום, v1, 2026-11-02. כפוף ל-`00-foundation-decisions.md`. אם יש סתירה, 00 גובר.
> הצעות לשינוי ב-00 מופיעות בסוף המסמך (§20).
> **מקורות:** דוח v1 (פרק האנגלית); `english_efl.md`, `learning_science_workbooks.md`,
> `hebrew_first_language_voice.md`, `voice_tech_he_en.md`, `placement_assessment.md`, `gap_audit.md`
> (בעיקר D3, D4, D9, D10, D18), `content_world_phygital.md`; יומן המחקר `docs/research-log/english.md`.
> **מיושר עם:** `01-game-design.md` (Sunny, ניצי, ‏R-08, ‏MX-E01..E04, עונות), `02-economy-progression.md` (ערכי
> מטבעות, `neverEarns`), `06`/`07` (רצועות T/G/S, תגי בטיחות, תקציב זמן משותף §2.1), `08-assessment-engine-child-model.md`
> (רמות שליטה `new → beginner → advancing → proficient → master`, משקלים), `09-ux-ui-design-system.md` (Andika, ‏`type.en.*`, ‏`CMP-22`),
> `12-ai-content-voice-pipelines.md` (רשימת ההקלטה §6.10, ‏TTS). פערים מול המפרטים האלה מרוכזים ב-§20.2.
> **רצועות גיל:** המסמך משתמש ב-`T` (פעוטות 2–4), `G` (גן 4–6), `S` (בית ספר, עם `gr`) כמו 06 ו-07. השנה: **K3 ≡ T, ‏K2 ≡ G, ‏K1 ≡ S (gr 3)**.
> בטבלאות שמציינות K1/K2/K3 הכוונה היא לרצועה, לא לילד; בקובצי התוכן השדה הוא תמיד `bands: ('T'|'G'|'S')[]`.
> **שפת המסמך:** ההסברים בעברית. תוכן באנגלית (מילים, משפטים, שירים, תבניות) כתוב באנגלית.
> **תגיות ראיה:** **[V]** אומת במקור בסבב הזה · **[N]** מגיע מהערת מחקר קודמת · **[BK]** ידע רקע שלא אומת בסבב הזה ·
> **[D]** החלטת תכנון שלי, עם נימוק.
> **מגבלת גישה:** אתרי Cambridge ומשרד החינוך היו חסומים בסבב הזה, ולכן רשימות המילים הרשמיות לא נקראו במלואן.
> בנק המילים (§7) בנוי על נושאי Starters ומסומן לאימות (משימה V1 ביומן המחקר).

---

## 0. תקציר החלטות (TL;DR)

1. **שלושה מסלולים, מנוע אחד.** ל-K1 (כיתה ג') יש פוניקה סינתטית שיטתית, אוצר מילים בעל-פה ותקשורת לפי
   Pre-A1. ל-K2 (גן חובה) יש אנגלית בעל-פה בלבד: שירים, TPR ו"שמע וגע" (gap_audit D18). ל-K3 (2.5) יש
   שירים ומילים בודדות, תמיד במשחק משותף עם הורה.
2. **כמות קלט לפני הכול** [N, Muñoz]. בכל שבוע יש יעד של "דקות קלט באנגלית" לכל ילד, והוא נמדד ומוצג להורה.
   מנות קצרות ותכופות עדיפות על מפגשים ארוכים.
3. **ידע אותיות (צליל ← אות) הוא המנוף המרכזי של K1** [N, Kahn-Horwitz]. הצליל נלמד לפני שם האות (00-B6),
   ושם האות מצטרף אחרי שהצליל נשלט. רצף הגרפמות הוא s a t p → i n m d → … (§5).
4. **שפה אחת לכל דמות (OPOL).** **Sunny** (CH-02 ב-`01-game-design.md`: לוטרת ים ושומרת מגדלור) מדברת רק
   אנגלית, נכנסת משמאל ומסומנת בטורקיז ובג'ינגל משלה. **ניצי** (CH-01) מתרגם, נכנס מימין, ותמיד בתבנית
   "סנדוויץ'" (EN → HE → EN). תמיכת העברית דועכת **לפי מילה**,
   לפי כללים מוגדרים (§10).
5. **הגייה: קודם האזנה.** זוגות מינימליים (w/v, th, ship/sheep, cat/cut/cot) נבחנים בהאזנה, עם כמה דוברים
   (§13). אף ילד לא מקבל ציון על ההפקה שלו. אין ASR עונשי.
6. **אנגלית אמריקאית** (00-E9). 45 הפונמות מוקלטות על ידי דובר ילידי. כל השאר מוקלט מראש ב-TTS. אין TTS חי.
7. **כל טקסט פענוחי עובר בודק פענוחיות בקוד**: מותר להשתמש רק בגרפמות שכבר נלמדו ובמילים "מסובכות" (tricky)
   שכבר נלמדו. ה-LLM כותב, והקוד מאשר.

---

## 1. עקרונות־על (חוקי מוצר לאנגלית)

| # | עיקרון | מה זה אומר בפועל | ראיה |
|---|---|---|---|
| EN-1 | **קלט רב, יומי וקצר** | יעד דקות קלט שבועי לכל ילד (§2). המדד כולל זמן באפליקציה, שירים בשמע ושגרות משפחה שההורה מסמן. | [N] Muñoz/BAF |
| EN-2 | **קודם בעל-פה** | מילה נשמעת ומובנת לפני שהיא נקראת. ב-K1 כל מילה בסיפור פענוחי כבר מוכרת בעל-פה, או שהיא מגיעה עם תמונה. | [V] Curriculum 2020: "extensive listening and speaking before literacy" |
| EN-3 | **שירים + תנועה (TPR)** | לכל שיר יש מערך תנועות. ההוראות הראשונות באנגלית נלמדות כפקודות תנועה. | [N] Magnussen & Sukying 2021 |
| EN-4 | **פוניקה סינתטית שיטתית ל-K1** | רצף קבוע, מיזוג משמאל לימין, מילים פענוחיות, ומילים מסובכות רק כשצריך. | [N] EEF ‎+5 חודשים; Ehri |
| EN-5 | **ידע אותיות** | אות גדולה וקטנה, צליל ושם. אבחנה מפורשת בין b/d/p/q. | [N] Kahn-Horwitz 2006/2012; [V] MoE Pre-A1 |
| EN-6 | **כיוון כתיבה הוא מיומנות** | צומת E.DIR נפרד. האנימציה תמיד זזה משמאל לימין, ושגיאות RTL מזוהות ומתועדות. | [V] MoE: "in the correct direction" |
| EN-7 | **האזנה לפני הפקה** | זוגות מינימליים עם כמה דוברים. אין ציון הפקה. | [N] ASR ילדים 15–35% WER; [BK] HVPT |
| EN-8 | **עברית כפיגום, לא כקביים** | סנדוויץ' למילים חדשות, ודעיכה לפי מילה. הסברי כללים בעברית ל-K1. | [N] Hall & Cook; Butzkamm |
| EN-9 | **שילוב פנימי (00-C1)** | פענוח מחייה: מילה שפוענחה קמה לחיים ומצטרפת כ-Wordling (01 §4.11). האזנה היא הנמל: Sunny קוראת מילה, ואת/ה מכוון/ת את הסירה הנכונה. | [N] Habgood & Ainsworth |
| EN-10 | **בלי עונש** | אין "לא נכון!" ואין צליל כישלון. Sunny חוזרת על הצורה הנכונה (recast). מגן התסכול של D10 חל על כל פעילות. | 00-C2/C4/C5 |
| EN-11 | **יישור עם בית הספר בלי לרוץ קדימה בעיוורון** | קצב K1 נקבע לפי שליטה. ההורה מזין פעם בחודש מה נלמד בכיתה (§17), והנושאים מתיישרים לפי זה. | [N] gap_audit #8 |
| EN-12 | **מה שלא נמדד בנייר לא נחשב** | בדיקות (probes) קצרות כל 4 שבועות, ומבדק בסגנון Starters בסוף השנה. | 00-C10 |

---

## 2. שלושה ילדים, שלושה מסלולים: יעדים לסוף השנה (יוני 2027)

| | **K1** (8.5, כיתה ג', Pre-A1) | **K2** (5.5, גן חובה) | **K3** (2.5) |
|---|---|---|---|
| מטרה | קוראת ומאייתת מילים פענוחיות (מסלול בסיס עד צירופים S9, מסלול מהיר עד e שקטה S10), מזהה 35 (בסיס) עד 40 (מהיר) מילים מסובכות מתוך בנק של 40 (§5.5), מבינה ומשתתפת בשיחות Pre-A1 | מבין הוראות ושירים, מצביע על כ-120 מילים, אומר כ-40 מילים ו-15 צירופים | מכיר 8–10 שירים עם תנועות, מגיב (X-EXP ≥10) לכ-30 מילים מתוך רשימת הליבה של 46 (§15.3) |
| ענפים פעילים | כולם (§3) | OR, VO, PA בעל-פה, SG, PR (האזנה פשוטה בלבד) | SG, VO (חשיפה), OR.L בסיסי |
| קרוא וכתוב | פוניקה S1–S9 ליבה (S10 במסלול מהיר), S11 אתגר; אות גדולה/קטנה; שם באנגלית; משפט עם נקודה | **אין.** אותיות מופיעות רק אגב (שיר ה-ABC, כרטיס השם של Sunny) (D18) | אין |
| דקות קלט בשבוע (יעד) | **20–25 באפליקציה** (§2.1) + 30–45 בשמע (רכב) ובשגרות משפחה | **8–10 באפליקציה** + 30–45 שירים ושגרות | 10–15 במשחק משותף + שירים בבית, בלי הגבלה |
| מבנה שבועי | 2 **בלוקי אנגלית** של 8–9 דק' בתוך פרקים מעורבים (08 §6.8) + חטיף אחד של 4 דק' בפרק שלישי (§15.1) | בלוק אחד של 8 דק' **או** 2 רגעי אנגלית של 4 דק' (שיר + נמל המילים), + שיר יומי (מצב שמע ברכב) | 3–4 פרקים של 3–5 דק' בחיק הורה, שיר אחד בכל פרק |
| חלק מזמן המסך (`subjectWeights`, 08 §6.8) | **20%** (מבוקש; ברירת המחדל של 08 היא 15%, §20.2) | **15%** (מבוקש; 08: 10%) | — (אין משקלים ב-T; הכול משחק משותף) |
| יעד הצלחה (00-C2) | 75–85% + Stretch Zone | ≈85% | ≈90–95%, בלי כישלון |
| עברית | הסברים בעברית, איים באנגלית בלבד שמתארכים מ-60 שנ' ל-4–5 דק' | סנדוויץ' לכל מילה חדשה, הוראות בעברית, פקודות אנגליות נלמדות | ניצי או ההורה ממסגרים, השיר באנגלית |
| ASR | לא ב-v1. שלב 2: אופציונלי, בנוכחות הורה, מגביל בלבד (§13) | לא | לא |

### 2.1 תקציב זמן שבועי משותף (05 + 06 + 07; הטבלה זהה בשלושת המסמכים)

הבסיס לחישוב הוא הפרופיל ה"טיפוסי" של 02 §12.1: K1 **5 פרקים × ~18 דק' ≈ 90 דק' בשבוע**, K2 **6 × ~10 ≈ 60 דק'**, K3 **4 × ~4 ≈ 16 דק'** (תקרה יומית
08 §11: K1 40, ‏K2 25, ‏K3 15 דק'). המשקלים הם **בקשה משותפת ל-08 §6.8 / `EngineConfig.subjectWeights`** (08 הוא המקור הקובע; עד שיעודכן, ברירות
המחדל של 08 תקפות: K1 חשבון 35 / עברית 35 / אנגלית 15 / אחר 15; K2 ‏35/35/10/20). הבקשה מחליפה את "אחר" בשני דליים מפורשים:

| דלי (`SubjectId` של 08) | K1 (S) | K2 (G) | מה נכנס | בעלים |
|---|---|---|---|---|
| `math` | 30% ≈ 27 דק' | 30% ≈ 18 דק' | 03 | 03 |
| `hebrew` | 28% ≈ 25 דק' | 30% ≈ 18 דק' | 04 (+ קרדיט חוצה של חבילות ידע, 06 §2.6) | 04 |
| `english` | **20% ≈ 18–20 דק'** | **15% ≈ 8–9 דק'** | 2 בלוקים + חטיף (K1); בלוק אחד או 2 רגעים (K2) | 05 |
| העשרה = `science`·`world`·`logic`·`art`·`music`·`motor` | **12% ≈ 11 דק'** | **15% ≈ 9 דק'** | חבילת ידע אחת + מעבדה/סטודיו אחד + חידת בונה (06 §2.5) | 06 |
| כישורי עתיד = `ai_literacy`·`sel` | **10% ≈ 9 דק'** | **10% ≈ 6 דק'** | `RIT-MID` + בלוק FS שבועי (07 §12.1). `RIT-OPEN/CLOSE` הם תקורה (≤10% מהפרק) ולא נספרים כאן | 07 |
| K3 (T) | — | — | אין משקלים. כל פרק הוא משחק משותף: שיר אנגלי אחד (05), פעילות העשרה אחת (06) או טקס FS (07), לפי בחירת ההורה/הילד (`shelf-3`) | — |

**כלל:** מסמך שרוצה יותר דקות ממה שבדלי שלו מוציא אותן **מחוץ למסך** (שמע ברכב, שגרות, דפים) ולא מדלי של מסמך אחר. הערך הקובע בקוד הוא `EngineConfig.subjectWeights` (08), והבדיקה היא AC18.

---

## 3. גרף המיומנויות

### 3.1 מוסכמת מזהים

`E.<STRAND>.<UNIT>.<SKILL>`: אותיות גדולות, ספרות ו-`_`. המזהה **יציב**: לא משנים מזהה אחרי שחרור, רק מוסיפים.

| STRAND | ענף | UNIT (דוגמאות) | SKILL (סיומות קבועות) |
|---|---|---|---|
| `DIR` | מושגי דפוס וכיוון | `LTR`, `CASE`, `WORD`, `PUNCT` | `TRACK`, `WRITE`, `MATCH`, `USE`, `SPACE`, `STOP` |
| `AL` | ידע אלפבית (שמות, צורה, אבחנה) | `NAME`, `VIS`, `FORM` | `SONG`, `ID`, `SAY`, `BDPQ`, `CFAM`… |
| `PA` | מודעות פונולוגית (בעל-פה) | `RHYME`, `SYL`, `ONSET`, `FINAL`, `MED`, `PHON` | `HEAR`, `MAKE`, `CLAP`, `ID`, `BLEND`, `SEG` |
| `PH` | פוניקה: גרפמה–פונמה, מיזוג, איות | `SAT`, `INM`, `GOK`, `EUR`, `HBF`, `JVW`, `YZQ`, `DIG`, `CLU`, `MGE`, `VTM`, `RCT`, `EXT` | `GPC`, `BLEND`, `SEG`, `READ`, `NAME` |
| `TW` | מילים מסובכות (tricky / heart words) | אותו UNIT כמו PH | `READ`, `SPELL` |
| `VO` | אוצר מילים לפי נושא | `ANI`, `BOD`, `CLO`, `COL`, `FAM`, `FOO`, `HOM`, `NUM`, `SCH`, `PLY`, `TOY`, `TRN`, `WLD`, `TIM`, `ADJ`, `VRB`, `FNC`, `HOL` | `REC` (קליטה), `PROD` (הפקה), `READ` (קריאה, K1) |
| `OR` | האזנה (`L`) ודיבור (`S`) | `L.CMD`, `S.GREET`… (UNIT מורכב משני חלקים) | `REC`, `RESP`, `SAY` |
| `PR` | תפיסה והגייה | `MP` (זוגות מינימליים), `STRESS`, `ECHO` | `WV`, `TH`, `DH`, `IEE`, `AUO`, `IEH`, `UU`, `LS` |
| `RD` | קריאה והבנה | `SENT`, `TEXT`, `PICT`, `YN` | `READ`, `COMP`, `MATCH` |
| `WR` | כתיבה ואיות | `COPY`, `NAME`, `SENT`, `DICT` | `WRITE`, `BUILD`, `SPELL` |
| `SG` | שירים ופזמונים | מזהה שיר (`HSKT`, `TTLS`…) | `JOIN` (הצטרפות לתנועה), `SING`, `FILL` |

דוגמאות: `E.PH.SAT.BLEND` (מיזוג מילים בצעד s-a-t-p), `E.PR.MP.WV` (האזנה w/v),
`E.VO.ANI.REC` (קליטת שמות חיות), `E.OR.S.LIKE.SAY` (אומר/ת "I like…").

### 3.2 סכמת צומת (pseudo-JSON, ב-`packages/content/en/skills.json`)

```jsonc
{
  "id": "E.PH.SAT.BLEND",
  "strand": "PH",
  "nameHe": "מיזוג מילים עם s a t p",
  "bands": ["S"],                        // T | G | S (06/07). השנה: K3=T, K2=G, K1=S. אין להשתמש ב-K1/K2/K3 בקבצים
  "grade": [3],                          // אופציונלי, ל-S בלבד (יישור לתוכנית הלימודים)
  "subject": "english",                  // SubjectId של 08
  "kind": "mastery",                     // mastery | familiarity | exposure
  "prereqs": [
    { "id": "E.PH.SAT.GPC", "min": "proficient" },      // רמות לפי 08 §5.1: new|beginner|advancing|proficient|master
    { "id": "E.PA.PHON.BLEND", "min": "proficient" },
    { "id": "E.DIR.LTR.TRACK", "min": "advancing", "soft": true }   // soft = מומלץ, לא חוסם. ("practicing" של 02 ≡ "advancing" של 08)
  ],
  "unlocksAt": { "prereqAccuracy": 0.7 }, // פותח לתרגול כשהקדם ≥70% (חפיפה מבוקרת)
  "mastery": "M-STD",                    // §3.3
  "templates": ["EN.T06", "EN.T08"],     // §11
  "errorCodes": ["ERR.DIR.RTL_SCAN", "ERR.PH.LETTERNAME", "ERR.PH.ADDED_SCHWA"],
  "content": { "graphemes": ["s","a","t","p"], "wordList": "PH.SAT.words" },
  "audio": ["ph_s","ph_a","ph_t","ph_p"],
  "difficulty": 1,                       // רמה שהמעצב קבע (gap_audit #4): Elo רק על יכולת הילד
  "source": ["L&S Ph2 Set1", "Jolly G1"], // מעקב מקור/יישור
  "curriculumTag": "MoE-PreA1/early-literacy"
}
```

### 3.3 הגדרות שליטה לפי סוג צומת

| קוד | למי | הגדרה |
|---|---|---|
| **M-STD** | K1 (וגם K2 בצמתי קליטה) | **= הרמות `proficient` ו-`master` של `08-assessment-engine-child-model.md` §5.1.** ‏`proficient` פותח צאצאים בגרף, ו-`master` הוא "Mastered" של D9. ההגדרה האחידה של gap_audit D9: לפחות 85% בשמונה הניסיונות הראשונים האחרונים, שלושת האחרונים בלי רמז, על פני יומיים לפחות, **וגם** בדיקה מושהית נכונה אחרי 3 ימים לפחות. סולם החזרה: 1, 3, 7, 14, 30, 60 יום. (08 הוא המקור הקובע. כאן רק מוצגת ההגדרה.) |
| **M-REC** | K2: צמתי `VO.*.REC` ו-`OR.L.*` | M-STD עם 6 ניסיונות במקום 8 ויעד של 80% (`proficientClean: 5, window: 6, proficientP: 0.80`). גם שימוש ב"עזרה בעברית" נחשב רמז. **מימוש:** דורש ש-08 יתמוך ב-`masteryProfile` לכל צומת (בקשה ב-§20.2). **עד אז M-REC = M-STD** (ברירת המחדל של 08), כדי לא לחסום פיתוח. |
| **F-FAM** (היכרות) | K2: `OR.S.*`, `VO.*.PROD`, `SG.*.SING` | לפחות 3 ניסיונות הפקה על פני יומיים לפחות, שההורה או הילד אישרו. **לא נבדק אוטומטית**, ולכן לא נחשב "שליטה" אלא "מוכר". |
| **X-EXP** (חשיפה) | K3 כולו; `SG.*.JOIN` | ספירת חשיפות עם מעורבות: הקשה, תנועה או קול. היעד הוא 10 חשיפות לפחות על פני 4 ימים לפחות. אין כישלון ואין "לא נשלט". |
| **M-PR** | `PR.MP.*` | 85% לפחות ב-12 פריטי AX/ABX אחרונים, ב-3 דוברים שונים לפחות ועל פני יומיים לפחות. |

**יחידת המעקב לאוצר מילים היא המילה, לא הנושא.** הצומת `E.VO.ANI.REC` נשלט כשלפחות 80% ממילות היעד שלו
(לפי עדיפות וגיל) במצב "נשלט". לכל מילה נשמר `lexState` (`new → seen → familiar → known → reviewed`) ו-`glossLevel` (§10.3).

### 3.4 צמתי הגרף לפי ענף

הטבלאות הן המקור לבניית `skills.json`. ב"קדם", ‏`~` מסמן קדם רך. "גיל" הוא הרצועה שבה הצומת פעיל.

#### 3.4.1 DIR: מושגי דפוס וכיוון (K1; מעט K2 אגב)

| ID | מיומנות | גיל | קדם | שליטה | פריט לדוגמה |
|---|---|---|---|---|---|
| `E.DIR.LTR.TRACK` | עוקבת אחרי הדגשה משמאל לימין ומקישה על אריחים בסדר הזה | K1 (K2 אגב) | — | M-STD | "גע באותיות לפי הסדר": 3 אריחים, החץ מהבהב משמאל |
| `E.DIR.LTR.WRITE` | מתחילה כתיבה בצד שמאל של השורה, אות אחרי אות ימינה | K1 | `E.DIR.LTR.TRACK` | M-STD | גרירת אותיות למשבצות: המשבצת הראשונה ריקה משמאל |
| `E.DIR.CASE.MATCH` | מתאימה אות גדולה לאות קטנה | K1 | `~E.PH.SAT.GPC` | M-STD | "מצאי את הזוג של G": ‏g q p |
| `E.DIR.CASE.USE` | אות גדולה בתחילת משפט, ב-I ובשמות | K1 | `E.DIR.CASE.MATCH`, `E.PH.HBF.GPC` | M-STD | "תקני את המשפט": ‏`pip is sad.` → `Pip is sad.` |
| `E.DIR.WORD.SPACE` | מזהה גבולות בין מילים | K1 | `E.PH.INM.BLEND` | M-STD | "כמה מילים יש פה?": ‏`Sam is sad.` → 3 |
| `E.DIR.PUNCT.STOP` | נקודה בסוף משפט | K1 | `E.DIR.WORD.SPACE` | M-STD | גרירת נקודה למקום הנכון |

#### 3.4.2 AL: אלפבית (שמות, צורה, אבחנה)

| ID | מיומנות | גיל | קדם | שליטה | הערות |
|---|---|---|---|---|---|
| `E.AL.NAME.SONG` | שר/ה את שיר ה-ABC | K1, K2 | — | F-FAM (K1) / X-EXP (K2) | שיר SG-ABC (§9). אותו לחן כמו Twinkle |
| `E.AL.NAME.ID` | מזהה אות לפי שמה ("tap the letter B") | K1 | שם האות נפתח רק אחרי ש-`GPC` של אותה אות נשלט | M-STD | שם אחרי צליל (00-B6) |
| `E.AL.NAME.SAY` | מאייתת בשמות אותיות (שם שלה, cat) | K1 | `E.AL.NAME.ID` | F-FAM | נדרש ב-MoE ("ask for a spelling") וב-Starters Listening |
| `E.AL.VIS.BDPQ` | מבחינה בין b/d/p/q, וגם n/u, m/w | K1 | `E.PH.HBF.GPC` (b) | M-STD | אסטרטגיית "bed": ‏b היא הקו ואז הבטן, d היא הבטן ואז הקו (§3.5, ERR.VIS.BD) |
| `E.AL.FORM.CFAM` | כתיבת משפחת c: ‏c a d g o q (e s) | K1 | `E.DIR.LTR.WRITE` | M-STD (+ דף מודפס) | משפחות כתיבה, כתיבה חופשית ולא רק העתקה (content_world c1) |
| `E.AL.FORM.LFAM` | משפחת l: ‏l t i j k y | K1 | `E.DIR.LTR.WRITE` | M-STD | |
| `E.AL.FORM.NFAM` | משפחת n: ‏n m h r b p | K1 | `E.DIR.LTR.WRITE` | M-STD | |
| `E.AL.FORM.ZFAM` | אלכסונים: ‏v w x z, ‏f u | K1 | `E.DIR.LTR.WRITE` | M-STD | |
| `E.AL.FORM.CAPS` | אותיות גדולות | K1 | `E.DIR.CASE.MATCH` | M-STD | |

#### 3.4.3 PA: מודעות פונולוגית באנגלית (בעל-פה, רק במילים מוכרות)

| ID | מיומנות | גיל | קדם | שליטה | פריט לדוגמה |
|---|---|---|---|---|---|
| `E.PA.RHYME.HEAR` | מזהה חרוז | K2, K1 | המילים ב-`lexState ≥ familiar` | M-REC / M-STD | "Cat… hat. Rhyme? 👍/👎". ב-K2 מתוך שירים |
| `E.PA.RHYME.MAKE` | משלים/ה חרוז (בחירה מ-3) | K2 (סוף שנה), K1 | `E.PA.RHYME.HEAR` | M-REC / M-STD | "A cat in a… (hat / dog / sun)" |
| `E.PA.SYL.CLAP` | מוחא/ת כפיים להברות | K2, K1 | — | F-FAM / M-STD | "ba-na-na": שלוש מחיאות. ההקשה על התוף נספרת |
| `E.PA.ONSET.ID` | צליל פותח (FSF) | K2 (מרץ ואילך), K1 | — | M-REC / M-STD | "Which one starts with /m/?": ‏moon, sun, cat |
| `E.PA.FINAL.ID` | צליל סוגר | K1 | `E.PA.ONSET.ID` | M-STD | "Which ends with /t/?" |
| `E.PA.MED.ID` | תנועה אמצעית | K1 | `E.PA.FINAL.ID` | M-STD | "cat / cut: which has /æ/?" **קשה במיוחד לדוברי עברית** |
| `E.PA.PHON.BLEND` | מיזוג בעל-פה (Sunny אומרת "c-a-t", את/ה בוחר/ת תמונה) | K2 (אתגר), K1 | `E.PA.ONSET.ID` | M-REC / M-STD | קדם הכרחי ל-`E.PH.*.BLEND` |
| `E.PA.PHON.SEG` | פירוק לצלילים (סופרים צלילים בקפיצות צפרדע) | K1 | `E.PA.PHON.BLEND` | M-STD | "sun": שלוש קפיצות |

#### 3.4.4 PH: פוניקה (K1). הצמתים נגזרים מהרצף ב-§5

לכל UNIT ‏`U` ברצף נוצרים הצמתים הבאים אוטומטית:

| תבנית ID | מיומנות | קדם | שליטה | מכניקה |
|---|---|---|---|---|
| `E.PH.U.GPC` | צליל ← גרפמה (שומעת /m/ ובוחרת m) **וגם** גרפמה ← צליל (רואה m ובוחרת תמונה שמתחילה ב-/m/) | `E.PH.(U-1).GPC` נשלט, ו-`E.PH.(U-1).BLEND` לפחות 70% | M-STD **לכל גרפמה בנפרד** | MX-E03 מצב צליל (T03/T04) |
| `E.PH.U.BLEND` | פענוח מילים אמיתיות ומילים מומצאות של ה-UNIT ושל מה שלפניו | `E.PH.U.GPC`, `E.PA.PHON.BLEND` | M-STD (לפחות 20% מהפריטים מילים מומצאות) | MX-E03 מצב מיזוג (T06), אמיתי או טיפשי (T08) |
| `E.PH.U.SEG` | איות מהכתבה במילים של ה-UNIT | `E.PH.U.BLEND` 70% | M-STD | MX-E03 מצב Mr. Mix-Up (T07) |
| `E.PH.U.READ` | קריאת משפט פענוחי ובחירת תמונה מתאימה | `E.PH.U.BLEND`, `E.TW.U.READ` | M-STD | יומן הספינה (T11) |
| `E.TW.U.READ` | זיהוי המילים המסובכות של ה-UNIT | `E.PH.U.GPC` | M-STD | אורות המגדלור (T10) |
| `E.PH.U.NAME` | שמות האותיות של ה-UNIT | `E.PH.U.GPC` | M-STD | T03 במצב שם |

UNITs (פירוט מלא ב-§5): ‏`SAT` → `INM` → `GOK` → `EUR` → `HBF` → [Boss א'] → `JVW` → `YZQ` → [Boss אלפבית] → `DIG` → `CLU` → `MGE` →
[Boss סוף שנה] → `VTM` (אתגר) → `RCT` (הכנה לכיתה ד') → `EXT` (כיתה ד').

#### 3.4.5 PR: תפיסה והגייה (קודם האזנה)

| ID | ניגוד | זוגות (דוגמה; רשימה מלאה ב-§5.4) | גיל | נפתח אחרי | שליטה |
|---|---|---|---|---|---|
| `E.PR.MP.WV` | /w/–/v/ | wet/vet, west/vest, wine/vine | K1 (K2: משחק "W like a kiss, V like a bunny bite" בלי ציון) | בדיקת בסיס בשיבוץ: אם הבסיס לפחות 90%, הצומת מסומן "לא נדרש" (סקר, לא הנחה [V]) | M-PR |
| `E.PR.MP.TH` | /θ/–/t/, ‏/θ/–/s/ | thin/tin, three/tree, thick/sick, bath/bat | K1 | `E.PH.DIG.GPC` (th) | M-PR |
| `E.PR.MP.DH` | /ð/–/d/ | then/den, they/day, though/dough | K1 | `E.PR.MP.TH` | M-PR |
| `E.PR.MP.AUO` | /æ/–/ʌ/–/ɑ/ | cat/cut/cot, hat/hut/hot, bag/bug, cap/cup/cop | K1 | `E.PH.EUR.GPC` (u) | M-PR |
| `E.PR.MP.IEH` | /ɪ/–/ɛ/ | pin/pen, sit/set, big/beg | K1 | `E.PH.EUR.GPC` (e) | M-PR |
| `E.PR.MP.IEE` | /ɪ/–/iː/ | ship/sheep, sit/seat, chip/cheap | K1 | `E.PH.INM.GPC` (האזנה בלבד, לא תלוי בקריאת ee. ההדגשה בלוח השנה היא במאי) | M-PR |
| `E.PR.MP.UU` | /ʊ/–/uː/ | full/fool, look/Luke, pull/pool | K1 (אתגר) | `E.PR.MP.IEE` | M-PR |
| `E.PR.MP.LS` | תנועה קצרה–ארוכה (e שקטה) | cap/cape, kit/kite, hop/hope, cub/cube | K1 | `E.PH.MGE.GPC` | M-PR |
| `E.PR.STRESS.TAP` | הטעמה: איפה החלק "החזק"? | BA-by, ba-NA-na, TI-ger | K1 (אתגר) | — | M-STD |
| `E.PR.ECHO.SAY` | חוזר/ת אחרי Sunny, הקלט והשווה | כל מילה | K1, K2 | — | **F-FAM בלבד, אף פעם לא ציון** |

#### 3.4.6 VO: אוצר מילים (כל הילדים; המילים עצמן ב-§7)

| ID | נושא | K3 | K2 | K1 |
|---|---|---|---|---|
| `E.VO.ANI.*` | חיות | X-EXP (10 מילים) | REC: M-REC · PROD: F-FAM | REC/PROD: M-STD · READ: לפי צעד הפענוח |
| `E.VO.BOD.*` | גוף ופנים | X-EXP | ✓ | ✓ |
| `E.VO.CLO.*` | בגדים | — | ✓ | ✓ |
| `E.VO.COL.*` | צבעים | X-EXP | ✓ | ✓ |
| `E.VO.FAM.*` | משפחה ואנשים | X-EXP (עם תמונות משפחה, D14) | ✓ | ✓ |
| `E.VO.FOO.*` | אוכל ושתייה | X-EXP | ✓ | ✓ |
| `E.VO.HOM.*` | בית וחפצים | X-EXP (bath, bed, cup) | ✓ | ✓ |
| `E.VO.NUM.*` | מספרים | 1–3 בשירים | 1–10 | 1–20 |
| `E.VO.SCH.*` | בית ספר וכיתה | — | חלקי | ✓ |
| `E.VO.PLY.*` | ספורט ופנאי | — | חלקי | ✓ |
| `E.VO.TOY.*` | צעצועים | X-EXP | ✓ | ✓ |
| `E.VO.TRN.*` | תחבורה | X-EXP (car, bus) | ✓ | ✓ |
| `E.VO.WLD.*` | העולם סביבנו: טבע, מזג אוויר, מקומות | — | חלקי | ✓ |
| `E.VO.TIM.*` | זמן וימים | — | — | ✓ |
| `E.VO.ADJ.*` | תארים ורגשות | X-EXP (happy, big) | ✓ | ✓ |
| `E.VO.VRB.*` | פעלים (פעולות TPR) | X-EXP (jump, clap) | ✓ | ✓ |
| `E.VO.FNC.*` | מילות יחס, כינויים ומילות תפקוד | — | in/on/under | ✓ |
| `E.VO.HOL.*` | חגים ועונות (הקשר ישראלי) | X-EXP | ✓ | ✓ |

קדמים: ב-K2 ‏`.PROD` דורש `.REC` במצב familiar. ב-K1 ‏`.READ` של מילה נפתח רק כשהמילה פענוחית בצעד הנוכחי
(הקוד מחשב את זה, §5.6) או כשהיא ברשימת המילים המסובכות שכבר נלמדו.

#### 3.4.7 OR: האזנה ודיבור (פירוט ההתקדמות ב-§8)

| ID | מיומנות | גיל | קדם | שליטה |
|---|---|---|---|---|
| `E.OR.L.CMD.REC` | מבין/ה פקודות כיתה ותנועה (§8.3) | K3 (5 פקודות, X-EXP), K2, K1 | — | M-REC / M-STD |
| `E.OR.L.Q.REC` | מבין/ה שאלות פשוטות (What's your name? How old…?) | K2, K1 | `E.OR.L.CMD.REC` | M-REC / M-STD |
| `E.OR.L.PREP.REC` | "Where's the ball? Under the bed." (שמע וגע) | K2 (in/on/under), K1 | `E.VO.FNC.REC` | M-REC / M-STD |
| `E.OR.L.COLOR.REC` | שמע/י וצבע/י (Starters Listening part 4) | K1 (K2 גרסה פשוטה) | `E.VO.COL.REC` | M-STD |
| `E.OR.L.NUM.REC` | מספרים בהאזנה, כולל 13/30 | K1 | `E.VO.NUM.REC` | M-STD |
| `E.OR.L.SPELL.REC` | מבינה שם שמאויית באותיות (S-A-M) | K1 | `E.AL.NAME.ID` | M-STD |
| `E.OR.L.STORY.COMP` | סיפור תמונות קצר, ואחריו שאלת תמונה | K2, K1 | `lexState` של מילות הסיפור | M-REC / M-STD |
| `E.OR.S.GREET.SAY` | Hello / Goodbye / How are you? I'm fine | K2, K1 | `E.OR.L.Q.REC` | F-FAM |
| `E.OR.S.NAME.SAY` | My name is… / I'm… | K2, K1 | — | F-FAM |
| `E.OR.S.AGE.SAY` | I'm five / eight. | K2, K1 | `E.VO.NUM.REC` | F-FAM |
| `E.OR.S.FEEL.SAY` | I'm happy / sad / tired / hungry. | K2, K1 | `E.VO.ADJ.REC` | F-FAM |
| `E.OR.S.LIKE.SAY` | I like… / I don't like… / Do you like…? | K2 (I like), K1 | `E.VO.FOO.REC` | F-FAM |
| `E.OR.S.WHATS.SAY` | What's this? It's a… | K2, K1 | `E.VO.*.REC` | F-FAM |
| `E.OR.S.HAVE.SAY` | I have a… / I have got… | K1 | — | F-FAM |
| `E.OR.S.CAN.SAY` | I can jump. Can you…? | K1 | `E.VO.VRB.REC` | F-FAM |
| `E.OR.S.ASK.SAY` | Can I have…, please? Here you are. Thank you. | K2 (please/thank you), K1 | — | F-FAM |
| `E.OR.S.SPELL.SAY` | How do you spell…? ומאיית/ת את שמו/ה | K1 | `E.AL.NAME.SAY` | F-FAM |
| `E.OR.S.TIME.SAY` | What time is it? It's three o'clock. | K1 | `E.VO.NUM.REC` | F-FAM |
| `E.OR.S.DESC.SAY` | There is a… / There are two… (תיאור תמונה, Starters Speaking) | K1 | `E.OR.S.WHATS.SAY` | F-FAM |
| `E.OR.S.CHANT.SAY` | אומר/ת פזמון קצר בעל-פה (MoE) | K2, K1 | `E.SG.*.JOIN` | F-FAM |

#### 3.4.8 RD ו-WR (K1)

| ID | מיומנות | קדם | שליטה | פריט לדוגמה |
|---|---|---|---|---|
| `E.RD.SENT.READ` | קוראת משפט פענוחי | `E.PH.INM.READ` | M-STD | `Pip sat on a mat.` ← בחירה מ-2 תמונות |
| `E.RD.PICT.MATCH` | קוראת משפט ובוחרת כן/לא לתמונה (Starters R&W 2) | `E.RD.SENT.READ` | M-STD | `The cat is on the bed.` 👍/👎 |
| `E.RD.TEXT.COMP` | סיפור פענוחי של 3–6 עמודים ושאלת הבנה | `E.PH.GOK.READ` | M-STD | עמוד ביומן הספינה, ואחריו "מה Pip מצא?" |
| `E.WR.COPY.WRITE` | מעתיקה מילה משמאל לימין | `E.DIR.LTR.WRITE` | M-STD | על המסך + דף מודפס |
| `E.WR.NAME.WRITE` | כותבת את שמה באנגלית | `E.AL.FORM.CAPS` | F-FAM + דף | שם הילדה מהפרופיל, איות שההורה אישר (§19) |
| `E.WR.DICT.SPELL` | מאייתת מהכתבה מילה פענוחית | `E.PH.U.SEG` | M-STD | T07 בלי מאגר אותיות מוגבל |
| `E.WR.SENT.BUILD` | בונה משפט מאריחי מילים, עם אות גדולה ונקודה | `E.DIR.PUNCT.STOP` | M-STD | `is / Sam / sad / .` |

#### 3.4.9 SG: שירים (צומת לכל שיר ב-§9)

`E.SG.<ID>.JOIN` (מצטרף/ת לתנועות, X-EXP), `E.SG.<ID>.SING` (שר/ה עם השיר, F-FAM), `E.SG.<ID>.FILL` (קריוקי
השלמת מילה, K2/K1, M-REC). השירים הם גם **מקור הקלט** של `VO` ו-`OR.L.CMD`: כל מילה בשיר שמסומנת `target`
מקבלת חשיפה ב-`lexState`.

### 3.5 קטלוג שגיאות (Error catalog)

כל שגיאה מתועדת עם קוד. הקוד משמש לשלושה דברים: **(1)** יצירת מסיחים בקוד (00-E8), **(2)** בחירת התערבות,
**(3)** אות להורה (רק כשיש דפוס עקבי: לפחות 4 מופעים ב-2 מפגשים). "העברה מעברית" מסמנת השערה שנבדקת ולא
מונחת מראש [BK].

| קוד | תיאור | העברה מעברית? | זיהוי בקוד | שימוש כמסיח | התערבות |
|---|---|---|---|---|---|
| `ERR.DIR.RTL_SCAN` | סריקה או הקשה מימין לשמאל; מיזוג "tac" במקום cat | ✓ (כיוון קריאה) | סדר ההקשות על האריחים יורד בציר x; בחירה של מילה הפוכה | ב-T08/T11 מוצגת גם המילה ההפוכה (tip/pit, pots/stop) | החץ הירוק מהבהב משמאל, האריח הראשון "קופץ". תרגול `E.DIR.LTR.TRACK` |
| `ERR.DIR.RTL_WRITE` | כתיבה שמתחילה מימין או משבצות שמתמלאות מימין | ✓ | מילוי משבצות T07 בסדר הפוך | — | אנימציית "השורה מתחילה ליד הלב השמאלי שלך" (רמז גוף) |
| `ERR.VIS.BD` | בלבול b/d | חלקי (בעברית בלבול ד/ר, ב/כ) | בחירה של b במקום d או להפך | ב-T03 כשהיעד b או d, האות השנייה תמיד מוצגת כמסיח | רמז "bed" (המיטה), סדר האנימציה: b = קו ואז בטן |
| `ERR.VIS.PQ` / `ERR.VIS.NU` / `ERR.VIS.MW` | p/q, n/u, m/w | — | כמו למעלה | כמו למעלה | השוואה צמודה (interleaving, ‏learning_science C12) |
| `ERR.CASE` | התאמת אות גדולה וקטנה שגויה; אות גדולה באמצע מילה | ✓ (בעברית אין אותיות גדולות) | T14, T16 | זוגות שנראים דומים: Gg/Qq, Bb/Dd, Nn/Hh | כרטיס "המשפחה של האות" |
| `ERR.SP.VOWEL_OMIT` | איות בלי תנועה ("bt" במקום bat) | ✓ (כתב עיצורי) | חסרה תנועה ב-T07 | — | "כל מילה באנגלית צריכה תנועה": התנועות בצבע, ומשבצת התנועה זוהרת |
| `ERR.PH.LETTERNAME` | מיזוג בשמות אותיות ("see-ay-tee") | — | (ASR לא קיים) הורה מסמן במצב "שמעתי"; T06: בחירת "סיי" ב-T04 | מסיח: מילה שמתחילה בשם האות (C → "see" ← sea) | "האות עושה צליל": Sunny אומרת את הצליל, ושם האות נשמר לשיר |
| `ERR.PH.ADDED_SCHWA` | "buh-a-tuh": שווא נוסף אחרי עיצור סותם, והמיזוג נכשל | — | (לא נמדד) | — | **מניעה**: הקלטות קטועות (§12.3), התחלה מעיצורים נמשכים (s, m), ומיזוג רציף |
| `ERR.PH.VOWEL_AUO` | /æ/ /ʌ/ /ɑ/ נשמעים כולם כ-/a/ | ✓ (בעברית יש /a/ אחת) | T09 / T04 | cat↔cut↔cot, bag↔bug | `E.PR.MP.AUO` עם מגוון דוברים. כרטיסי פה: "æ = smile wide", "ʌ = surprised uh" |
| `ERR.PH.VOWEL_IEH` | /ɪ/–/ɛ/ (pin/pen) | ✓ | T09 / T07 (pen במקום pin) | pin↔pen | `E.PR.MP.IEH` |
| `ERR.PH.VOWEL_IEE` | /ɪ/–/iː/ (ship/sheep) | ✓ | T09 | ship↔sheep | `E.PR.MP.IEE` |
| `ERR.PH.VOWEL_UU` | /ʊ/–/uː/ | ✓ | T09 | full↔fool | `E.PR.MP.UU` |
| `ERR.PH.TH_T` / `TH_S` / `DH_D` / `DH_Z` | th כ-t/s, ‏th קולי כ-d/z | ✓ (אין בעברית) | T09 | three↔tree, think↔sink, then↔den | תפיסה בלבד. הפקה: אנימציית "לשון נחש" (בלי ציון) |
| `ERR.PH.W_V` | w/v (שמיעה או כתיבה) | ✓ (בעברית שתיהן ו') | T09; T07: ‏v במקום w | wet↔vet | `E.PR.MP.WV`. כתיבה: "W = two V's holding hands" |
| `ERR.PH.SPLIT_DIGRAPH` | sh/ch/th נקראים כשתי אותיות | — | T06: הקשה על s ואז h בנפרד | ship ← "s-hip" | אריח מחובר אחד לדיגרף + אנימציית "חברים שלא נפרדים" |
| `ERR.PH.MAGIC_E` | מתעלמת מ-e השקטה (cake = "cak") או מכלילה אותה יתר על המידה | — | T09-LS / T06 | cap↔cape | קשת e הקסומה: ה-e "קופצת" ומאירה את התנועה |
| `ERR.PH.CK_SPELL` | c/k/ck: בחירה לא מדויקת באיות | — | T07 | — | **איות סביר מתקבל** (מסומן "plausible"), ומוצג המודל |
| `ERR.TW.DECODED` | מילה מסובכת פוענחה כרגילה ("sa-id") | — | T10 | said↔"sad" | "מילת לב": החלק הלא רגיל מסומן בלב, השאר מפוענח |
| `ERR.GR.COPULA` | "She happy" (בלי is/am/are) | ✓ (אין אוגד בהווה) | T12 (בחירת בועה) | בועה בלי am/is | Sunny חוזרת על המשפט בצורה הנכונה, ותבנית קבועה: I'm / It's |
| `ERR.GR.ARTICLE` | "I have dog" (בלי a) | ✓ | T12/T16 | בועה בלי a | recast: "A dog! I have a dog." |
| `ERR.GR.ADJ_ORDER` | "a ball red" | ✓ (שם עצם ואחריו תואר) | T16 | אריח בסדר עברי | שיר צבעים: "a red ball" |
| `ERR.GR.LIGHT` | "Open/close the light" | ✓ (פתח את האור) | T12 | — | צירוף קבוע: "Turn on/off the light" |
| `ERR.GR.AGE` | "I have eight years" | ✓ | T12 | בועה מסיחה | צירוף: "I'm eight." |
| `ERR.GR.IT` | he/she לחפצים | ✓ | T12 | — | "It's a…" |
| `ERR.GR.PLURAL` | "two cat" | חלקי | T16/T12 | — | שיר ספירה עם -s מודגש |
| `ERR.VO.SEMANTIC` | בחירת שכן סמנטי (cat במקום dog) | — | T01 | מסיחים מאותו נושא ברמה 2 | ידע חלקי: מתועד, לא "שגיאה" בדוח להורה |
| `ERR.VO.LOAN_STRESS` | מילה שאולה בהגייה עברית (baNAna, KOMputer) | ✓ | (הפקה, לא נמדד) | — | הקלט והשווה, ו-"Sunny says it like this" |
| `SIG.GLOSS_DEPEND` | שימוש ב"עזרה בעברית" ביותר מ-40% מהתורות | — | תדירות לחיצה | — | **אות למנוע**, לא שגיאה: איים קצרים יותר, חזרה על מילים |
| `SIG.RAPID_TAP` | הקשות מהירות ואקראיות | — | 3 תשובות בפחות מ-0.6 שנ' | — | מגן עייפות (learning_science F19) |

**חוק:** קודי `ERR.PH.R`, ‏`ERR.PH.H_DROP` ו-`ERR.PH.NG_G` (ר גרונית, h חלשה, "sing-g") **לא נאכפים בכלל**,
כי הם לא פוגעים בהבנה. הם מופיעים רק כהערה לכותבי התוכן: אל תבנו פריט סביבם.

---

## 4. איך האנגלית חיה בעולם (שילוב פנימי, 00-C1; מיושר ל-01)

אזור האנגלית הוא **R-08 ‏Sunny Harbor** (הנמל והמגדלור של Sunny, ‏`01-game-design.md` §2.2), מעבר לגשר החבלים
"שבו השפה מתחלפת". הכניסה: ג'ינגל `sfx.english.enter` (3 צלילי יוקלילי), צבע `palette.english` (טורקיז וצהוב)
ומעבר לעמוד LTR (§10.4). **חוק עולם (01 §2.4):** "אור עשוי מידע". מילה באנגלית שנאמרה, נשמעה או פוענחה יוצרת
אור. מילה שנלמדה הופכת ל-**Wordling** (`sparkling.word`), ידיד אור שחי רק על האי ואומר את המילה בקולה של Sunny.
אף מכניקה כאן לא מתפקדת כשער חידון. **הבוס של האי הוא Mr. Mix-Up**, שמבלבל מילים באנגלית (פרק 6 בעונה 1,
01 §2.4): הילדים "מסדרים לו" את המילים, והוא נהיה חבר. **זהות (01 §4.1):** Mr. Mix-Up הוא **הַבַּלְגָּנִיסְט (CH-04) בתחפושת האי**. כדי לא
לשבור את "אדם אחד, שפה אחת" (00-B2), **הקול שלו על האי הוא קול en-US נפרד** (קול "החבר" הגברי מ-§12.2; לא הקול העברי של CH-04), והוא **לא
אומר אף מילה באנגלית מחוץ לאי ואף מילה בעברית בתוך האי**. הטעויות המכוונות (protégé, §10.6) הן שלו בלבד; Sunny לא טועה. ⟂VERIFY מול בעלי 01
(§20.2): אם 01 מעדיף דמות נפרדת, המזהה כאן נשאר `MIXUP` וההחלפה היא רק בליהוק. **צבעי האי:** הקוד משתמש **רק בטוקנים** `palette.english` (01)
ו-`region.englishIsle` (09); הערכים שלהם סותרים כרגע (01: טורקיז `#19B7B0` + צהוב; 09: סגול-טורקיז + `grape`) ונפתרים ב-09 (בעל הטוקנים, §20.2).

| מכניקה (01) | פעולת משחק = פעולת למידה | צמתים | תבנית | ילדים |
|---|---|---|---|---|
| **MX-E03 צדפי הצלילים** (`phonics-shells`), מצב *צליל* | Sunny משמיעה צליל, ומקישים על הצדף עם הגרפמה, והצדף נפתח ומאיר | `PH.*.GPC`, `AL.NAME.ID` | T03, T04 | K1 |
| **MX-E03**, מצב *מיזוג* | גוררים צדפים לשורה משמאל לימין. Sunny ממזגת, החפץ קם לחיים ומצטרף כ-Wordling. טוויסט: מחליפים צדף (c→h) ומקבלים hat (01). מילה מומצאת הופכת ל"Silly-ling" מצחיק לאלבום | `PH.*.BLEND`, `PA.PHON.BLEND` | T06, T08 | K1 |
| **MX-E03**, מצב *Mr. Mix-Up* | Mr. Mix-Up ערבב את הצדפים. שומעים את המילה ומסדרים את האותיות במשבצות LTR | `PH.*.SEG`, `WR.DICT.SPELL` | T07 | K1 |
| **MX-E01 צדף ההד** (`echo-shell`), מצב *זוגות* (**הרחבה מוצעת ל-01**, §20.2) | שתי תמונות. הצדף מהדהד מילה בקולו של דובר משתנה, ובוחרים | `PR.MP.*` | T09 | K1 (K2 גרסה רכה בלי ציון) |
| **MX-E04 נמל המילים** (`word-harbor`) | סירות עם תמונות מגיעות. השלט (בקולה של Sunny, ב-K1 גם בכתב) אומר מילה, ומכוונים את הסירה הנכונה לרציף | `VO.*.REC`, `OR.L.*`, ‏K1: ‏`VO.*.READ` | T01 | K2, K1 |
| **MX-E01 צדף ההד**, מצב *TPR* + **MV-08 "Sunny Says"** | פקודות תנועה ("Jump! Touch the red boat!"), ואישור עצמי או של ההורה. טוויסט: בלי "Sunny says" לא זזים | `OR.L.CMD`, `VO.VRB` | T02 | כולם |
| **MX-E02 בועות שיר** (`song-bubbles`) + מצב קריוקי | פוקעים בועות בקצב, וכל בועה היא מילה עם תמונה. בועה שלא נפקעה צפה הלאה, ואין "החטאה". ב-K2 וב-K1 השיר נעצר וממלאים מילה | `SG.*` | T13 | K3, K2 (K1 בקריוקי) |
| **אורות המגדלור** (**מכניקה מוצעת MX-E05**, §20.2) | המגדלור מאיר מילים מסובכות על הגלים. Sunny אומרת מילה, ו"תופסים" אותה באלומה | `TW.*.READ` | T10 | K1 |
| **יומן הספינה של Sunny** (**מוצע MX-E06**) | סיפור פענוחי מעמודי היומן. כל עמוד שנקרא מדליק פנס בנמל. מצב "קראי לאח" (מדריכה, 00-C6) | `RD.*`, `PH.*.READ` | T11 | K1 (K2 מאזין) |
| **שיחה על הרציף** (**מוצע MX-E07**) | תורות דיאלוג כתובות מראש. Sunny שואלת, והילד בוחר בועה או אומר (הקלט והשווה) | `OR.S.*` | T12 | K2, K1 |
| **"גלה"** (K3, בחדר הגחליליות R-12 או בנמל, במשחק משותף) | מקישים על צדף או דלת, ויוצאות תמונה ומילה בקולה של Sunny. אין שאלה | `VO.*` (X-EXP) | T20 | K3 |

**דמות משנה לסיפורים פענוחיים (מוצעת ל-01):** ‏**Pip**, הסרטן הקטן של Sunny. Pip לא מדבר (אין לו קול, ולכן
אין הפרה של OPOL). הוא קיים כי השם שלו **פענוחי מצעד S2**, כך שהסיפור הראשון של K1 יכול להיות "Pip sat."
השם Sunny הוא מילה מסובכת אישית (‏y = /i/ נלמד רק ב-S13).

**חומרים במשחק המשפחתי (המדורה):** כל פעולת אנגלית מוסיפה לדלק המדורה המשותפת, בדיוק כמו מתמטיקה ועברית.
ב-K3 התפקיד הוא "שומר/ת האש": הקשה על חיה או צבע שהשיר מזכיר מוסיפה ניצוץ, ואין כישלון. **כל שורה באנגלית מושמעת רק בתוך מסגרת אנגלית** (`englishContext=true`, ‏AC-MX-3 ב-01).

---

## 5. פוניקה: רצף מלא (Scope & Sequence)

### 5.1 עקרונות הרצף

1. **רצף לפי Letters and Sounds / Jolly** [V/BK]: ‏s a t p כבר מאפשרות מילים אמיתיות בצעד הראשון. אין
   שימוש בטקסטים של UFLI, כי UFLI אוסרת יצירות נגזרות [N].
2. **התאמה לאנגלית אמריקאית:** short o היא /ɑ/ (hot, top). ‏r-controlled נלמד כ-ar, ‏or ו-er/ir/ur (/ɝ/), בלי
   יחידות ear/air/ure הבריטיות.
3. **התאמה לדוברי עברית:** w/v ו-th מוצגים יחד עם אימון האזנה בזוגות מינימליים. השלישייה a/u/o (/æ ʌ ɑ/)
   מקבלת צומת PR נפרד. e שקטה (MGE) מלמדת שאנגלית כותבת תנועות. לכל צעד יש מילים מומצאות (pseudowords) כדי
   למדוד פענוח אמיתי, בדומה ל-NWF [N].
4. **קצב:** צעד נפתח רק כשהצעד הקודם נשלט (§3.4.4). זמן היעד הוא 1.5–3 שבועות לצעד (§15).
5. **סדר הצגה בתוך צעד:** צליל (עם כרטיס פה ותנועת יד) ← גרפמה (גדולה וקטנה) ← מיזוג בעל-פה ← מיזוג מילים ←
   מילה מומצאת ← איות ← משפט ← סיפור. שם האות מגיע בסוף הצעד.
6. **עיצורים נמשכים קודם במיזוג:** ‏s, m, n, f, l נמתחים ("sssaaat"), ועיצורים סותמים מוקלטים קטועים (§12.3).

### 5.2 טבלת הרצף

| # | UNIT | גרפמות חדשות (בסדר ההוראה) | פונמות (US IPA) | מילים מסובכות חדשות | יעד שבועות | בוס/ציון דרך |
|---|---|---|---|---|---|---|
| S1 | `SAT` | s, a, t, p | /s/ /æ/ /t/ /p/ | I, a | 1.5 | — |
| S2 | `INM` | i, n, m, d + סיומת רבים ‎-s (‏taps, pins) | /ɪ/ /n/ /m/ /d/ | the, is, and\* | 2 | הסיפור הראשון: "Pip" |
| S3 | `GOK` | g, o, c, k | /g/ /ɑ/ /k/ /k/ | to, no, go | 2 | — |
| S4 | `EUR` | ck, e, u, r | /k/ /ɛ/ /ʌ/ /ɹ/ | you, my, of | 2 | — |
| S5 | `HBF` | h, b, f, ff, l, ll, ss | /h/ /b/ /f/ /l/ /s/ | he, she, we, me, be, was, his\*, has\* | 2.5 | **Boss א': "23 פנסים"** |
| S6 | `JVW` | j, v, w, x | /dʒ/ /v/ /w/ /ks/ | they, are, all | 2 | — |
| S7 | `YZQ` | y, z, zz, qu | /j/ /z/ /kw/ | do, her, what | 1.5 | **Boss אלפבית: 26 אותיות, גדולות וקטנות, צליל ושם** |
| S8 | `DIG` | sh, ch, th (/θ/ ו-/ð/), ng, nk | /ʃ/ /tʃ/ /θ/ /ð/ /ŋ/ /ŋk/ | there, some, come, one, said | 3 | — |
| S9 | `CLU` | אין גרפמה חדשה: צירופי עיצורים CVCC, ‏CCVC, ‏CCVCC; ‏s בתוך מילה כ-/z/ (is, his, has, as) | — | like, have, little, were, so | 3 | — |
| S10 | `MGE` | a_e, i_e, o_e, u_e, e_e | /eɪ/ /aɪ/ /oʊ/ /juː/ /iː/ | two, who, does, love, your | 3 | **Boss סוף שנה: "יומן המסע"** |
| S11 | `VTM` (אתגר, יוני) | ee, ea, ai, ay, oa, ow (snow), igh, y (/aɪ/) | /iː/ /eɪ/ /oʊ/ /aɪ/ | people, could, would, should, their, water | 4 | — |
| S12 | `RCT` (כיתה ד') | oo (moon), oo (book), ar, or, er, ir, ur | /uː/ /ʊ/ /ɑɹ/ /ɔɹ/ /ɝ/ | Mr, Mrs, oh, friend, because, school | — | — |
| S13 | `EXT` (כיתה ד') | ow (cow), ou, oi, oy, aw, au, ew, ue, wh, ph, ‎-ed, ‎-ing, soft c/g, ‎-le, y (/i/) | — | לפי Band I | — | — |

\* `and` מסובכת עד S9 (צירוף nd). ‏`is/his/has` מסובכות עד S9 (‏s = /z/ בתוך המילה). סיומת הרבים ‎-s היא מורפמה שנלמדת כבר ב-S2 ("one pin, two pins"), והבודק מתיר אותה כחריג מכלל הצירופים.

### 5.3 מילים פענוחיות לכל צעד

הרשימות **מצטברות**: בכל צעד מותר להשתמש גם בכל מה שלפניו. ‏`pseudo` הן מילים מומצאות למכניקת "אמיתי או
טיפשי". ✱ = שם (אות גדולה). הרשימות S1–S10 הורצו בבודק פענוחיות ראשוני בזמן כתיבת המסמך (סקריפט אב-טיפוס, 0 כשלים). מילים שמובנות לילד בעל-פה
מסומנות ב-VO (§7), והשאר מקבלות תמונה.

**S1 `SAT`** (s a t p)
- real: at, sat, pat, tap, sap, taps, pats, sats
- pseudo: tas, pas, aps, tat
- phrases: `I sat.` · `I tap.` · `Tap, tap, tap!` · `A pat.`

**S2 `INM`** (+ i n m d)
- real: it, in, an, am, sit, pit, tip, sip, nip, pin, tin, din, dip, dim, did, mid, man, mat, map, pan, nap, tan,
  mad, dad, sad, pad, tins, pins, maps, mats, naps, dips; ✱ Pip, Sam, Tim, Pam, Nan, Dan\*\*
- pseudo: tid, nim, dap, sim, mip, nad, pid
- sentences: `Pip sat.` · `Sam is sad.` · `Pip is in a pan.` · `Tim and Pam sit.` · `Dad naps.` · `The map is in the tin.`

\*\* שמות הילדים באנגלית נכנסים כ"מילים מסובכות אישיות" (§19), גם כשהם לא פענוחיים (למשל Mia, עם ia). כך הילדה קוראת את שמה כבר מהשבוע הראשון.

**S3 `GOK`** (+ g o c k)
- real: on, got, not, pot, top, dot, cot, cod, nod, pop, mop, cog, cap, cat, can, cop, kit, kid, kin, gap, gas,
  tag, nag, sag, dig, pig, gig, dog\*, cats, pigs, pots, tops, caps, kits; ✱ Tom, Kim, Kit
- pseudo: gom, kag, tog, cim, gop, dak
- sentences: `Pip got a pig.` · `The cat is on top.` · `Kim can tap a pot.` · `No, cat! Go, go, go!` · `It is not a dog.`

\* במבטא אמריקאי "dog" נוטה ל-/dɔg/. היא לא תשמש מילת מפתח לתנועה /ɑ/. מילת המפתח של o היא **pot** (§6).

**S4 `EUR`** (+ ck e u r)
- real: kick, sock, sack, tick, neck, deck, pick, pack, duck, tuck, rock, rack, sick, dock, pet, net, men, ten,
  pen, red, get, met, set, peg, den, up, us, cup, mud, sun, run, rug, nut, gum, pup, mom, cut, tug, rat, ram, rip,
  rid, rot, rim, rag, ducks, socks, pens, cups, nuts; ✱ Ted, Ken, Meg
- pseudo: rup, ret, muk, sut, nock, ded
- sentences: `Ted gets a red cup.` · `The sun is up.` · `Pip runs to the rock.` · `A duck in the mud.` · `You can pet my pup.`

**S5 `HBF`** (+ h b f ff l ll ss)
- real: hat, hot, hen, hug, hit, him, hum, hop, hut, had, bat, bed, big, bag, bun, bus, bib, bit, bet, but, bug,
  bad, bin, fan, fun, fig, fit, fog, fed, fin, off, puff, cuff, huff, lid, leg, log, lot, let, lip, lap, lit, lick,
  luck, lock, bell, doll, fill, hill, tell, sell, mess, kiss, less, boss, miss, hiss, toss, fuss, hats, bags, bells;
  ✱ Bob, Ben, Bill, Liz\*\*\*
- pseudo: hib, fom, lud, bek, fass, holl
- sentences: `Ben has a big red hat.` · `Bob fell off the log.` · `Fill the bag, Pip!` · `He hid in the hut.` · `She can hop on the bed.` · `We had fun in the sun.`

\*\*\* "Liz" נכנס רק ב-S7 (z).

**S6 `JVW`** (+ j v w x)
- real: jam, jet, jog, jug, job, jab, van, vet, vat, wet, web, wig, win, wag, wit, wed, will, well, box, fox, six,
  mix, fix, ox, tax, wax, fax, sax, jets, vans, webs, wigs; ✱ Jim, Jen, Max, Vic
- pseudo: jit, vop, wum, lix, vam, jeck
- sentences: `A fox sat in a box.` · `The vet has a big van.` · `Max will win.` · `They are all wet.` · `Jen fed six hens.`
- זוגות w/v: ‏wet/vet (S6). השאר מגיעים ב-S9 וב-S10

**S7 `YZQ`** (+ y z zz qu)
- real: yes, yet, yum, yap, yell, yak, zip, zap, buzz, fizz, jazz, fuzz, quiz, quit, quick, quack, quill;
  ✱ Zak, Liz
- pseudo: yot, zeb, quan, zuff, yick
- sentences: `Yes! Zak can zip the bag.` · `Quick, quick, quack!` · `The bug can buzz.` · `Let her do the quiz.` · `Quiz Liz!` · `What is in the box?`

**S8 `DIG`** (+ sh ch th ng nk)
- sh: ship, shop, shed, shut, shell, fish, dish, wish, rush, cash, mash, bash, gush, shock
- ch: chip, chin, chop, chat, chick, check, much, rich, such, chess, chum
- th /θ/: thin, thick, moth, math, bath, path, Beth, thud
- th /ð/: this, that, then, them, than, with\*
- ng / nk: ring, sing, king, long, song, bang, hang, sung, wing, thing, pink, sink, think, bank, thank, junk, honk, wink
- pseudo: shom, chib, thup, zing, shev, chock
- sentences: `The fish is in the dish.` · `Chip and Beth sing a song.` · `I think this is a ship.` · `Thank you!` · `Come and get the shell.` · `Some ducks run to the shed.`

\* במבטא אמריקאי "with" נשמעת /wɪθ/ או /wɪð/, ושתיהן מתקבלות.

**S9 `CLU`** (צירופים + ‎-s)
- CVCC: ants, hand, sand, band, land, milk, tent, best, nest, rest, test, west, vest, jump, lamp, camp, help,
  desk, gift, list, mist, belt, pond, wind, hunt, lift, soft, fast, mask, went, and, bump
- CCVC: frog, flag, stop, spot, swim, clap, crab, drum, grin, plum, step, snap, from, black, block, truck, trip,
  sniff, spell, still, twin, skip, trap, drop, glad, plan, slip, sled, spin, smell, snack, flip, grab, brick
- CCVCC ומעבר: stamp, crisp, blast, trunk, drink, stick, crash, fresh, brush, lunch, bench, shrimp\*
- ‏s = /z/: is, his, has, as (עוברות ממילים מסובכות למפוענחות)
- pseudo: blim, frap, nust, stog, crub, plint
- sentences: `The frog can jump in the pond.` · `Stop and help the crab!` · `Fred has a black tent.` · `Swim fast, frog!` · `I like milk and jam.` · `They were in the camp.`
- זוגות w/v: ‏west/vest, ‏went/vent

\* shrimp: ‏CCCVCC עם דיגרף, לשלב אתגר בלבד.

**S10 `MGE`** (+ a_e i_e o_e u_e e_e)
- a_e: cake, make, name, game, late, gate, tape, plane, lake, snake, grape, shake, came, same, made, cave, wave, safe
- i_e: bike, kite, five, time, line, ride, like, nine, pine, smile, white, slide, hide, fine, mine, bite, wide, prize
- o_e: bone, home, nose, rope, hole, note, joke, stone, smoke, those, hope, rose, globe
- u_e: cube, cute, tube, mule, June, flute, use, huge\*
- e_e: these, Pete, Steve
- pseudo: dake, fime, zote, nube, plome, vike
- sentences: `Kate has a cake.` · `I like my bike.` · `Five kites go up.` · `The cute mule is at home.` · `Who made this game?` · `Do you love your home?`
- זוגות e שקטה (`E.PR.MP.LS`): cap/cape, kit/kite, hop/hope, cub/cube, pin/pine, not/note, tap/tape, rip/ripe,
  mad/made, hat/hate, can/cane, rob/robe, tub/tube, slid/slide, win/wine; ‏w/v: ‏wine/vine

\* huge: ‏g רכה, אתגר בלבד.

**S11 `VTM`** (אתגר; + ee ea ai ay oa ow igh y)
- ee: see, bee, tree, feet, green, sheep, three, week, sleep, queen, teeth, need, seed
- ea: sea, eat, tea, read, meat, beach, team, clean, leaf, dream
- ai / ay: rain, tail, wait, paint, train, snail, mail · day, play, say, way, may, gray, stay
- oa / ow: boat, coat, road, soap, goat, toast · snow, slow, low, grow, show, yellow\*, window\*
- igh / y: night, light, right, high, bright · my, by, fly, cry, sky, why\*\*, try
- זוגות (`E.PR.MP.IEE`): ship/sheep, sit/seat, fit/feet, hit/heat, bin/bean, chip/cheap, lip/leap, still/steal

\* דו-הברתיות, אתגר. \*\* wh, מסובכת.

**S12 `RCT`** (כיתה ד'; + oo oo ar or er ir ur): moon, food, zoo, room, spoon, pool, cool, soon, tooth · book,
look, good, foot, cook, wood, hook · car, star, park, farm, arm, dark, shark, card, jar · for, fork, corn, horse,
short, sport, storm, born, horn · her, fern, bird, girl, first, shirt, turn, burn, fur, nurse.
זוגות (`E.PR.MP.UU`): full/fool, look/Luke, pull/pool.

### 5.4 בנק זוגות מינימליים (לתבנית T09)

כל זוג צריך שתי תמונות שאפשר לצייר בבירור, ולכן זוגות מופשטים מסומנים בסוגריים (לשימוש רק בשמע).

| צומת | זוגות |
|---|---|
| `E.PR.MP.WV` | wet/vet, west/vest, wine/vine, went/vent, (wail/veil), (wiper/viper) |
| `E.PR.MP.TH` | thin/tin, three/tree, thick/tick, bath/bat, math/mat, path/pat, thank/tank, thick/sick, think/sink, thing/sing, mouth/mouse, (thumb/sum) |
| `E.PR.MP.DH` | then/den, they/day, (though/dough), (breathe/breeze) |
| `E.PR.MP.AUO` | cat/cut/cot, hat/hut/hot, cap/cup/cop, bag/bug, ran/run, mad/mud, sack/suck/sock, rack/ruck/rock, (lack/luck/lock) |
| `E.PR.MP.IEH` | pin/pen, sit/set, big/beg, bit/bet, pig/peg, tin/ten, lid/led |
| `E.PR.MP.IEE` | ship/sheep, sit/seat, fit/feet, hit/heat, bin/bean, chip/cheap, lip/leap, still/steal, (live/leave) |
| `E.PR.MP.UU` | full/fool, pull/pool, look/Luke, (could/cooed) |
| `E.PR.MP.LS` | cap/cape, kit/kite, hop/hope, cub/cube, pin/pine, not/note, tap/tape, can/cane, tub/tube, rip/ripe |

### 5.5 מילים מסובכות: שיטת "מילת לב"

- כל מילה מסובכת מוצגת עם **החלק הרגיל מפוענח והחלק הלא רגיל מסומן בלב** (למשל ב-said: ‏s ו-d רגילות, ‏ai ❤).
- מקור הרשימה: Letters and Sounds Phase 2–5 [BK] ו-Dolch Pre-Primer/Primer [V], בסינון לפי שימושיות.
- סה"כ עד סוף S10 יש **40 מילים** בטבלת §5.2 (2+3+3+3+8+3+3+5+5+5; כולל and/is/his/has, שהופכות לפענוחיות ב-S9 ועוברות לרשימת
  "פוענחו!" כחגיגה קטנה, לא כנטישה), **ועוד מילים מסובכות אישיות**: `Sunny` ושמות הילדים (§19). עד S9 (מסלול בסיס) הבנק הוא 35, ולכן
  יעד K1 הוא 35 (בסיס) או 40 (מהיר), §15.1.
- מילת Dolch שמופיעה בצעד שבו היא פענוחית (למשל can, it, up, big, red) **לא** נלמדת כמילה מסובכת. היא מפוענחת.

### 5.6 בודק פענוחיות (קוד, `packages/content/en/decodable.ts`)

```ts
// pseudo-code
type Unit = "SAT"|"INM"|"GOK"|"EUR"|"HBF"|"JVW"|"YZQ"|"DIG"|"CLU"|"MGE"|"VTM"|"RCT"|"EXT";
interface Mapping { word: string; graphemes: string[]; source: "cmudict-align"|"manual" }
// 1. כל מילה מפורקת לגרפמות בעזרת מילון יישור גרפמה-פונמה (נגזר מ-CMUdict + עקיפות ידניות) [BK: CMUdict BSD]
// 2. מותרת אם: כל גרפמה ∈ allowed(unit) (מצטבר) ו-pattern(word) ∈ patterns(unit)
//    patterns: SAT..DIG = {VC, CVC, CVCC* only digraph/ck/ff/ll/ss/zz/nk endings}; CLU+ = +{CCVC, CVCC, CCVCC};
//    MGE+ = +{split digraph}; VTM+ = +{vowel teams}
// 3. או: word ∈ trickyTaught(unit) ∪ personalNames(child)
//    personalNames = שמות הילדים באנגלית (§19) + "Sunny" + "Pip" (פענוחי מ-INM) + ✱ שמות הסיפורים של ה-UNIT (§5.3)
// 3a. מורפמת -s (רבים "pins", גוף שלישי "runs") מותרת מ-INM גם כשהצירוף CVC+s חורג מ-patterns(unit); "s" סופית = /s/ או /z/ (שתיהן מתקבלות)
// 3b. אותיות גדולות: תחילת משפט, I, ושמות. אותן גרפמות; CASE לא משפיע על הפענוחיות
// 4. טקסט מותר אם 100% מהטוקנים מותרים; מילת תוכן אחת לא-פענוחית מותרת רק עם תמונה + tap-to-hear ומסומנת "picture word"
// 5. פיסוק מותר: . ! ? , ; גרש רק במילים מסובכות שנלמדו (it's, let's מ-S3; I'm מ-S2)
function isDecodable(word: string, unit: Unit, child: ChildId): Verdict;   // Verdict = { ok: boolean; reason?: 'grapheme'|'pattern'|'tricky'|'name' }
```

CI: כל פריט שמסומן `decodable: true` עובר את הבודק. אם הוא נכשל, ה-build נכשל (AC2).

### 5.7 כללים לטקסטים פענוחיים (גם לפרומפט של ה-LLM)

- אורך: S2–S4 עד 2 משפטים של 3–5 מילים לעמוד, 3 עמודים. S5–S8 עד 3 משפטים, 4–5 עמודים. S9–S10 עד 4 משפטים,
  6 עמודים.
- כל סיפור כולל את **Pip** הסרטן (פענוחי) או את Sunny (מילה מסובכת אישית), ולפחות אחד מגיבורי המחנה, ומסתיים בשינוי קטן במצב (נמצא משהו, נבנה משהו). אין
  אלימות. צללונים מתהפכים לחברים.
- לכל סיפור יש שאלת הבנה אחת לפחות שאפשר לענות עליה בתמונה (`E.RD.TEXT.COMP`).
- כל מילה ניתנת להקשה לשמיעה (Sunny). הקשה לא פוגעת בשליטת `PH.*.READ` של מילה פענוחית, אבל נרשמת כ"נעזרה".
- הטקסטים מיוצרים ב-Batch (00-E8), עוברים את הבודק, נבדקים ב-QA על ידי Sonnet ומחכים לאישור הורה אופציונלי.

---

## 6. מלאי הפונמות (General American, 45 מזהים)

הטבלה היא גם **רשימת ההקלטות** של הדובר הילידי (§12.3). "סיכון" = סיכון העברה לדוברי עברית [BK].
"סוג" = n (נמשך: מוקלט רגיל + מתוח) או s (סותם: מוקלט קטוע, בלי שווא).

| ID | IPA | גרפמה עיקרית | מילת מפתח | צעד | סוג | סיכון / הערה |
|---|---|---|---|---|---|---|
| `ph_s` | s | s, ss | sun | S1 | n | — |
| `ph_ae` | æ | a | apple | S1 | n | גבוה: מתמזג עם /ʌ/ /ɑ/ ← /a/ עברית |
| `ph_t` | t | t | top | S1 | s | — |
| `ph_p` | p | p | pig | S1 | s | — |
| `ph_ih` | ɪ | i | igloo | S2 | n | גבוה: ‏/ɪ/→/i/ |
| `ph_n` | n | n | net | S2 | n | — |
| `ph_m` | m | m | moon | S2 | n | — |
| `ph_d` | d | d | dog | S2 | s | — |
| `ph_g` | g | g | goat | S3 | s | — |
| `ph_aa` | ɑ | o | pot / octopus | S3 | n | גבוה: ‏/ɑ/→/a/ (עברית) או /o/ |
| `ph_k` | k | c, k, ck | cat, kite | S3 | s | — |
| `ph_eh` | ɛ | e | egg | S4 | n | נמוך: קרוב ל-/e/ עברית |
| `ph_uh` | ʌ | u | umbrella / up | S4 | n | גבוה |
| `ph_r` | ɹ | r | red | S4 | n | ר גרונית: **לא נאכף** |
| `ph_h` | h | h | hat | S5 | n (נשיפה) | h חלשה: **לא נאכף** |
| `ph_b` | b | b | bat | S5 | s | — |
| `ph_f` | f | f, ff | fish | S5 | n | — |
| `ph_l` | l | l, ll | leg | S5 | n | — |
| `ph_dzh` | dʒ | j | jam | S6 | s | נמוך (ג׳) |
| `ph_v` | v | v | van | S6 | n | ניגוד w/v |
| `ph_w` | w | w | web | S6 | n | בינוני: ‏w→v אצל חלק מהדוברים [V] |
| `ph_ks` | ks | x | box | S6 | s | צירוף, מוקלט כיחידה |
| `ph_j` | j | y | yes | S7 | n | — |
| `ph_z` | z | z, zz, s | zip | S7 | n | — |
| `ph_kw` | kw | qu | queen | S7 | s | צירוף, מוקלט כיחידה |
| `ph_sh` | ʃ | sh | ship | S8 | n | — |
| `ph_ch` | tʃ | ch | chip | S8 | s | נמוך (צ׳) |
| `ph_th` | θ | th | thumb / three | S8 | n | גבוה: ‏→t/s |
| `ph_dh` | ð | th | this / feather | S8 | n | גבוה: ‏→d/z |
| `ph_ng` | ŋ | ng, n(k) | ring | S8 | n | ‏"ring-g": **לא נאכף** |
| `ph_zh` | ʒ | s (treasure) | treasure | S13 | n | נמוך (ז׳). מוקלט עכשיו לשלמות |
| `ph_ey` | eɪ | a_e, ai, ay | cake | S10 | n | — |
| `ph_iy` | iː | e_e, ee, ea | tree | S10 | n | ניגוד /ɪ/ |
| `ph_ay` | aɪ | i_e, igh, y | kite | S10 | n | — |
| `ph_ow` | oʊ | o_e, oa, ow | boat | S10 | n | ‏/o/ עברית טהורה, בלי גלישה: לא נאכף |
| `ph_yuw` | juː | u_e | cube | S10 | n | — |
| `ph_uw` | uː | oo, ew, ue | moon | S12 | n | ניגוד /ʊ/ |
| `ph_uu` | ʊ | oo, u | book | S12 | n | גבוה |
| `ph_ar` | ɑɹ | ar | car | S12 | n | — |
| `ph_or` | ɔɹ | or | fork | S12 | n | — |
| `ph_er` | ɝ | er, ir, ur | bird | S12 | n | גבוה (אין בעברית) |
| `ph_aw` | aʊ | ow, ou | cow | S13 | n | — |
| `ph_oy` | ɔɪ | oi, oy | coin | S13 | n | — |
| `ph_ao` | ɔ | aw, au | saw | S13 | n | במבטא US רבים ממזגים עם /ɑ/. מוקלט, שימוש מינימלי |
| `ph_schwa` | ə | a (about), the | **לא נלמד כצליל בודד** | — | n | מוקלט לבניית "the" ו-"a" במודלים בלבד |

**מילת המפתח של `ph_ih`:** ‏**igloo** (נשמעת /ɪ/ בבירור, וקל לצייר אותה). insect ו-itch נפסלו כי קשה לצייר
אותן או כי הן לא נעימות. **הערה:** ‏`ph_ks` ו-`ph_kw` אינן פונמות. הן צירופים שמוקלטים כיחידה, ולכן יש 43 פונמות
ועוד 2 צירופים, סה"כ 45 קבצי צליל.

---

## 7. בנק אוצר המילים (534 מילים, לפי נושא)

### 7.1 כללים

- **מקור ועדיפות.** עמוד השדרה הוא נושאי Cambridge Pre A1 Starters [BK/V], בסדר עדיפות שמתאים לגיל ולהקשר
  הישראלי. **P1** = ליבה: מילים תכופות שמופיעות ב-Starters או בשירים, ונלמדות להפקה ב-K1. **P2** = שאר
  Starters ומילים שימושיות, לקליטה ול"הפקה כשמוכן". **P3** = אתגר, מילים מקומיות (pita, hummus) וחלק מ-Movers.
  המיפוי ל-Rec/Prod של משרד החינוך [V]: ב-K1, ‏P1 היא `prod`, ‏P2 היא `rec` עם `prod` אופציונלי, ו-P3 היא `rec`.
  **סטטוס אימות:** הבנק לא הושווה עדיין לרשימה הרשמית (V1 ביומן המחקר). כשההשוואה תתבצע, כל מילה תקבל
  `src: starters|moe-preband1|both|local`.
- **"מגיל"** הוא הגיל הצעיר ביותר שבו המילה מתאימה. **זה מאגר, לא יעד.** היעדים בפועל: K3 מקבל רשימת ליבה של 40
  מילים (§15.3), K2 מקבל כ-120 מילים לשנה לפי נושא חודשי (§15.2), ו-K1 מקבלת את כל P1 ו-P2 לאורך השנה, לפי
  תוכנית הנושאים (§15.1).
- **אנגלית אמריקאית** היא הצורה הראשית, והצורה הבריטית נשמרת ב-`alt`. אם בית הספר של K1 משתמש בצורה הבריטית
  (eraser/rubber), שתי הצורות מתקבלות בקליטה, ו-Sunny תמיד אומרת את הצורה האמריקאית.
- **★ = מילה שאולה או קוגנט** (banana, pizza, robot). אלה "ניצחונות מהירים" לביטחון ולמנוע ("מילים שאת/ה כבר
  יודע/ת!"), **אבל** ההגייה וההטעמה האנגליות שונות, ולכן הן מתורגלות ב"הקלט והשווה" (ERR.VO.LOAN_STRESS).
- **הגלוס העברי** נכתב כאן בלי ניקוד. הצינור מוסיף ניקוד (Dicta), והטקסט עובר בדיקה אנושית מדגמית (00-B5).
  בשמע, הגלוס נאמר על ידי ניצי בלבד (§10).
- **מגדר:** גלוס של שם עצם הוא צורת המילון. גלוס של מילה שתלויה במגדר הנמען (you, friend) נשמר עם וריאנטים
  `m/f/pl` (00-B3).
- **תמונה:** לכל מילה יש איור אחד בסגנון המחנה (00-E10). מילים מופשטות (the, of, very) מופיעות רק בצירופים, לא
  כפריט "שמע וגע".
- **מבנה רשומה** ב-`packages/content/en/vocab.json`:
  `{ "en":"apple", "he":{"base":"תפוח"}, "topic":"FOO", "p":1, "minBand":"K3", "cognate":false, "alt":null, "ipa":"ˈæpəl", "img":"foo_apple", "audio":{"pip":"w_apple_sunny", "talkers":[]}, "decodableAt":null, "src":"tbd" }`.
  ‏`decodableAt` מחושב אוטומטית על ידי בודק הפענוחיות (§5.6). למשל `cat` = `GOK`, ‏`apple` = null עד S13 (‏-le).

### 7.2 סיכום כמויות

| קוד | נושא | סה"כ | P1 | P2 | P3 |
|---|---|---|---|---|---|
| `ANI` | חיות | 44 | 21 | 20 | 3 |
| `BOD` | גוף ופנים | 21 | 16 | 5 | 0 |
| `CLO` | בגדים | 23 | 12 | 9 | 2 |
| `COL` | צבעים | 13 | 12 | 1 | 0 |
| `FAM` | משפחה ואנשים | 26 | 13 | 8 | 5 |
| `FOO` | אוכל ושתייה | 52 | 19 | 23 | 10 |
| `HOM` | בית וחפצים | 45 | 17 | 17 | 11 |
| `NUM` | מספרים | 22 | 11 | 11 | 0 |
| `SCH` | בית ספר וכיתה | 33 | 11 | 15 | 7 |
| `PLY` | ספורט, משחק ופנאי | 26 | 9 | 10 | 7 |
| `TOY` | צעצועים | 13 | 5 | 6 | 2 |
| `TRN` | תחבורה | 13 | 6 | 5 | 2 |
| `WLD` | העולם סביבנו: טבע, מקומות, מזג אוויר | 28 | 9 | 13 | 6 |
| `TIM` | זמן וימים | 18 | 4 | 13 | 1 |
| `ADJ` | תארים ורגשות | 36 | 14 | 15 | 7 |
| `VRB` | פעלים (רובם פקודות TPR) | 59 | 35 | 21 | 3 |
| `FNC` | מילות תפקוד: כינויים, יחס, שאלה | 54 | 33 | 21 | 0 |
| `HOL` | חגים ועונות (הקשר ישראלי) | 8 | 2 | 3 | 3 |
| | **סה"כ** | **534** | **249** | **216** | **69** |

### 7.3 צירופים קבועים (Chunks) שנלמדים כיחידה

הצירופים הם יחידות לקסיקליות לכל דבר, עם `id`, שמע ותמונה או תנועה (§8.2). הם לא נספרים ב-534.

### 7.4 הרשימות

#### 7.4.1 `ANI`: חיות (44)

| EN | עברית | P | מגיל | הערה |
|---|---|---|---|---|
| cat | חתול | P1 | K3 |  |
| dog | כלב | P1 | K3 |  |
| bird | ציפור | P1 | K3 |  |
| fish | דג | P1 | K3 |  |
| duck | ברווז | P1 | K3 |  |
| cow | פרה | P1 | K3 |  |
| horse | סוס | P1 | K3 |  |
| sheep | כבשה, כבשים | P1 | K3 | יחיד=רבים |
| frog | צפרדע | P1 | K3 |  |
| monkey | קוף | P1 | K3 |  |
| elephant | פיל | P1 | K3 |  |
| bear | דוב | P1 | K3 |  |
| puppy | כלבלב | P2 | K3 |  |
| chicken | תרנגולת; עוף (אוכל) | P1 | K2 | גם FOO |
| mouse (mice) | עכבר (עכברים) | P1 | K2 | רבים לא סדיר |
| lion | אריה | P2 | K2 |  |
| tiger | נמר | P1 | K2 |  |
| snake | נחש | P1 | K2 |  |
| spider | עכביש | P1 | K2 |  |
| giraffe | ג'ירפה | P1 | K2 | ★ |
| zebra | זברה | P2 | K2 | ★ |
| hippo | היפופוטם | P2 | K2 | ★ |
| crocodile | תנין | P2 | K2 | ★ (קרוקודיל) |
| goat | עז | P2 | K2 |  |
| donkey | חמור | P2 | K2 |  |
| pig | חזיר | P2 | K2 | Old MacDonald |
| rabbit | ארנב | P2 | K2 |  |
| bee | דבורה | P2 | K2 |  |
| butterfly | פרפר | P2 | K2 |  |
| turtle | צב | P2 | K2 |  |
| penguin | פינגווין | P2 | K2 | ★ |
| owl | ינשוף | P3 | K2 | דמות ידידותית בלבד |
| kitten | חתלתול | P3 | K2 |  |
| pet | חיית מחמד | P1 | K2 |  |
| animal | חיה, בעל חיים | P1 | K2 |  |
| zoo | גן חיות | P1 | K2 |  |
| fox | שועל | P2 | K1 | פענוחי S6 |
| hen | תרנגולת | P2 | K1 | פענוחי S5 |
| ant | נמלה | P2 | K1 | פענוחי S9 |
| lizard | לטאה | P2 | K1 |  |
| polar bear | דוב קוטב | P2 | K1 |  |
| jellyfish | מדוזה | P2 | K1 |  |
| tail | זנב | P2 | K1 |  |
| bat | עטלף; מחבט | P3 | K1 | פענוחי S5, דו-משמעי |

#### 7.4.2 `BOD`: גוף ופנים (21)

| EN | עברית | P | מגיל | הערה |
|---|---|---|---|---|
| head | ראש | P1 | K3 |  |
| shoulders | כתפיים | P1 | K3 | שיר HSKT |
| knees | ברכיים | P1 | K3 | שיר HSKT |
| toes | אצבעות רגליים | P1 | K3 | שיר HSKT |
| eyes | עיניים | P1 | K3 |  |
| ears | אוזניים | P1 | K3 |  |
| nose | אף | P1 | K3 |  |
| mouth | פה | P1 | K3 |  |
| hair | שיער | P1 | K3 |  |
| face | פנים | P1 | K3 |  |
| hand | יד, כף יד | P1 | K3 |  |
| foot (feet) | כף רגל (כפות רגליים) | P1 | K3 | רבים לא סדיר |
| leg | רגל | P1 | K3 |  |
| tummy | בטן | P2 | K3 |  |
| finger | אצבע | P2 | K3 |  |
| arm | זרוע | P1 | K2 |  |
| body | גוף | P1 | K2 |  |
| tooth (teeth) | שן (שיניים) | P2 | K2 |  |
| smile | חיוך; לחייך | P1 | K2 |  |
| back | גב | P2 | K1 |  |
| neck | צוואר | P2 | K1 |  |

#### 7.4.3 `CLO`: בגדים (23)

| EN | עברית | P | מגיל | הערה |
|---|---|---|---|---|
| hat | כובע | P1 | K3 |  |
| shoe | נעל | P1 | K3 |  |
| sock | גרב | P1 | K3 |  |
| pajamas | פיג'מה | P2 | K3 | ★ UK: pyjamas |
| shirt | חולצה | P1 | K2 |  |
| T-shirt | טי-שירט | P1 | K2 | ★ |
| pants | מכנסיים | P1 | K2 | UK: trousers |
| shorts | מכנסיים קצרים | P1 | K2 | ★ (שורטס) |
| skirt | חצאית | P1 | K2 |  |
| dress | שמלה | P1 | K2 |  |
| jacket | ז'קט, מעיל קל | P1 | K2 | ★ |
| coat | מעיל | P2 | K2 |  |
| bag | תיק | P1 | K2 |  |
| cap | כובע מצחייה | P2 | K2 | Starters: baseball cap |
| boots | מגפיים | P2 | K2 |  |
| jeans | ג'ינס | P2 | K2 | ★ |
| glasses | משקפיים | P2 | K2 |  |
| sweater | סוודר | P3 | K2 | ★ |
| costume | תחפושת | P2 | K2 | פורים |
| mask | מסכה | P2 | K2 | ★ פורים |
| clothes | בגדים | P2 | K1 |  |
| handbag | תיק יד | P3 | K1 |  |
| wear | ללבוש | P1 | K1 |  |

#### 7.4.4 `COL`: צבעים (13)

| EN | עברית | P | מגיל | הערה |
|---|---|---|---|---|
| red | אדום | P1 | K3 |  |
| blue | כחול | P1 | K3 |  |
| yellow | צהוב | P1 | K3 |  |
| green | ירוק | P1 | K3 |  |
| orange | כתום | P1 | K3 | גם FOO (תפוז) |
| pink | ורוד | P1 | K3 |  |
| purple | סגול | P1 | K2 |  |
| brown | חום | P1 | K2 |  |
| black | שחור | P1 | K2 |  |
| white | לבן | P1 | K2 |  |
| gray | אפור | P1 | K2 | UK: grey |
| color | צבע | P1 | K2 | UK: colour |
| rainbow | קשת בענן | P2 | K2 |  |

#### 7.4.5 `FAM`: משפחה ואנשים (26)

| EN | עברית | P | מגיל | הערה |
|---|---|---|---|---|
| mom | אמא | P1 | K3 | UK: mum; ב-K3 גם mommy |
| dad | אבא | P1 | K3 | ב-K3 גם daddy |
| baby | תינוק, תינוקת | P1 | K3 |  |
| brother | אח | P1 | K3 |  |
| sister | אחות | P1 | K3 |  |
| grandma | סבתא | P1 | K3 |  |
| grandpa | סבא | P1 | K3 |  |
| family | משפחה | P1 | K2 |  |
| friend | חבר, חברה | P1 | K2 |  |
| boy | ילד, בן | P1 | K2 |  |
| girl | ילדה, בת | P1 | K2 |  |
| man (men) | איש (אנשים) | P1 | K2 |  |
| name | שם | P1 | K2 |  |
| uncle | דוד | P3 | K2 |  |
| aunt | דודה | P3 | K2 |  |
| mother | אם | P2 | K1 |  |
| father | אב | P2 | K1 |  |
| grandmother | סבתא | P2 | K1 |  |
| grandfather | סבא | P2 | K1 |  |
| child (children) | ילד (ילדים) | P2 | K1 | רבים לא סדיר |
| kid | ילד, ילדה | P2 | K1 |  |
| woman (women) | אישה (נשים) | P2 | K1 |  |
| people | אנשים | P2 | K1 |  |
| person | אדם | P3 | K1 |  |
| cousin | בן דוד, בת דודה | P3 | K1 |  |
| classmate | חבר לכיתה, חברה לכיתה | P3 | K1 |  |

#### 7.4.6 `FOO`: אוכל ושתייה (52)

| EN | עברית | P | מגיל | הערה |
|---|---|---|---|---|
| apple | תפוח | P1 | K3 |  |
| banana | בננה | P1 | K3 | ★ הטעמה: ba-NA-na |
| bread | לחם | P1 | K3 |  |
| egg | ביצה | P1 | K3 |  |
| cake | עוגה | P1 | K3 |  |
| chocolate | שוקולד | P1 | K3 | ★ |
| ice cream | גלידה | P1 | K3 |  |
| milk | חלב | P1 | K3 |  |
| water | מים | P1 | K3 |  |
| juice | מיץ | P1 | K3 | ★ (ג'וס) |
| cookie | עוגייה | P2 | K3 | UK: biscuit |
| yummy | טעים | P1 | K3 |  |
| grapes | ענבים | P1 | K2 |  |
| watermelon | אבטיח | P1 | K2 |  |
| fruit | פרי, פירות | P1 | K2 |  |
| carrot | גזר | P1 | K2 |  |
| tomato | עגבנייה | P1 | K2 |  |
| pizza | פיצה | P1 | K2 | ★ |
| food | אוכל | P1 | K2 |  |
| drink | שתייה; לשתות | P1 | K2 |  |
| lemon | לימון | P2 | K2 | ★ |
| mango | מנגו | P2 | K2 | ★ |
| pear | אגס | P2 | K2 |  |
| pineapple | אננס | P2 | K2 | ★ |
| strawberry | תות | P2 | K2 |  |
| potato | תפוח אדמה | P2 | K2 |  |
| cucumber | מלפפון | P2 | K2 |  |
| salad | סלט | P2 | K2 | ★ |
| rice | אורז | P2 | K2 |  |
| cheese | גבינה | P2 | K2 |  |
| burger | המבורגר | P2 | K2 | ★ |
| fries | צ'יפס | P2 | K2 | ★ UK: chips |
| pasta | פסטה | P2 | K2 | ★ |
| sandwich | כריך, סנדוויץ' | P2 | K2 | ★ |
| candy | ממתק | P2 | K2 | UK: sweets |
| lemonade | לימונדה | P2 | K2 | ★ |
| pita | פיתה | P3 | K2 | ★ מקומי |
| hummus | חומוס | P3 | K2 | ★ מקומי |
| soup | מרק | P3 | K2 |  |
| kiwi | קיווי | P3 | K2 | ★ |
| honey | דבש | P3 | K2 | ראש השנה |
| peas | אפונה | P2 | K1 |  |
| beans | שעועית | P2 | K1 |  |
| meat | בשר | P2 | K1 |  |
| breakfast | ארוחת בוקר | P2 | K1 |  |
| lunch | ארוחת צהריים | P2 | K1 |  |
| dinner | ארוחת ערב | P2 | K1 |  |
| coconut | קוקוס | P3 | K1 | ★ |
| lime | ליים | P3 | K1 | ★ |
| onion | בצל | P3 | K1 |  |
| sausage | נקניקייה | P3 | K1 |  |
| tea | תה | P3 | K1 |  |

#### 7.4.7 `HOM`: בית וחפצים (45)

| EN | עברית | P | מגיל | הערה |
|---|---|---|---|---|
| door | דלת | P1 | K3 |  |
| bed | מיטה | P1 | K3 |  |
| table | שולחן | P1 | K3 |  |
| chair | כיסא | P1 | K3 |  |
| light | אור; מנורה | P1 | K3 | חנוכה; Turn on the light |
| bath | אמבטיה | P1 | K3 |  |
| cup | כוס | P1 | K3 |  |
| towel | מגבת | P3 | K3 | שגרת אמבטיה |
| toothbrush | מברשת שיניים | P2 | K3 |  |
| house | בית | P1 | K2 |  |
| home | בית (הבית שלי) | P1 | K2 |  |
| room | חדר | P1 | K2 |  |
| kitchen | מטבח | P1 | K2 |  |
| garden | גינה | P1 | K2 |  |
| window | חלון | P1 | K2 |  |
| picture | תמונה | P1 | K2 |  |
| phone | טלפון | P1 | K2 | ★ |
| computer | מחשב | P1 | K2 | ★ |
| box | קופסה, ארגז | P1 | K2 | פענוחי S6 |
| sofa | ספה | P2 | K2 | ★ |
| lamp | מנורה | P2 | K2 | ★ |
| mirror | מראה | P2 | K2 |  |
| TV | טלוויזיה | P2 | K2 | ★ television |
| plate | צלחת | P2 | K2 |  |
| spoon | כף | P2 | K2 |  |
| bowl | קערה | P3 | K2 |  |
| soap | סבון | P3 | K2 |  |
| bedroom | חדר שינה | P2 | K1 |  |
| bathroom | חדר אמבטיה | P2 | K1 |  |
| living room | סלון | P2 | K1 |  |
| apartment | דירה | P2 | K1 | UK: flat |
| wall | קיר | P2 | K1 |  |
| floor | רצפה | P2 | K1 |  |
| clock | שעון (קיר) | P2 | K1 |  |
| watch | שעון (יד); לצפות | P2 | K1 |  |
| camera | מצלמה | P2 | K1 |  |
| mat | שטיחון | P2 | K1 | פענוחי S2 |
| dining room | פינת אוכל | P3 | K1 |  |
| armchair | כורסה | P3 | K1 |  |
| radio | רדיו | P3 | K1 | ★ |
| cupboard | ארון | P3 | K1 |  |
| bookcase | כוננית ספרים | P3 | K1 |  |
| rug | שטיח | P3 | K1 | פענוחי S4 |
| fork | מזלג | P3 | K1 |  |
| key | מפתח | P3 | K1 |  |

#### 7.4.8 `NUM`: מספרים (22)

| EN | עברית | P | מגיל | הערה |
|---|---|---|---|---|
| one | אחת (1) | P1 | K3 | ספירה בנקבה, 00-B4 |
| two | שתיים (2) | P1 | K3 |  |
| three | שלוש (3) | P1 | K3 |  |
| four | ארבע (4) | P1 | K2 |  |
| five | חמש (5) | P1 | K2 |  |
| six | שש (6) | P1 | K2 |  |
| seven | שבע (7) | P1 | K2 |  |
| eight | שמונה (8) | P1 | K2 |  |
| nine | תשע (9) | P1 | K2 |  |
| ten | עשר (10) | P1 | K2 |  |
| number | מספר | P1 | K2 |  |
| zero | אפס | P2 | K1 |  |
| eleven | אחת-עשרה (11) | P2 | K1 |  |
| twelve | שתים-עשרה (12) | P2 | K1 |  |
| thirteen | שלוש-עשרה (13) | P2 | K1 | ניגוד 13/30 |
| fourteen | ארבע-עשרה (14) | P2 | K1 |  |
| fifteen | חמש-עשרה (15) | P2 | K1 |  |
| sixteen | שש-עשרה (16) | P2 | K1 |  |
| seventeen | שבע-עשרה (17) | P2 | K1 |  |
| eighteen | שמונה-עשרה (18) | P2 | K1 |  |
| nineteen | תשע-עשרה (19) | P2 | K1 |  |
| twenty | עשרים (20) | P2 | K1 |  |

#### 7.4.9 `SCH`: בית ספר וכיתה (33)

| EN | עברית | P | מגיל | הערה |
|---|---|---|---|---|
| book | ספר | P1 | K3 |  |
| school | בית ספר | P1 | K2 |  |
| teacher | מורה | P1 | K2 |  |
| pen | עט | P1 | K2 | פענוחי S4 |
| pencil | עיפרון | P1 | K2 |  |
| English | אנגלית | P1 | K2 |  |
| Hebrew | עברית | P1 | K2 |  |
| story | סיפור | P1 | K2 |  |
| song | שיר | P1 | K2 |  |
| crayon | צבע פנדה, עפרון צבעוני | P2 | K2 |  |
| eraser | מחק | P2 | K2 | UK: rubber |
| paper | נייר | P2 | K2 |  |
| playground | חצר משחקים | P2 | K2 |  |
| glue | דבק | P3 | K2 |  |
| scissors | מספריים | P3 | K2 |  |
| letter | אות; מכתב | P1 | K1 |  |
| word | מילה | P1 | K1 |  |
| class | כיתה (הקבוצה) | P2 | K1 |  |
| classroom | כיתה (החדר) | P2 | K1 |  |
| ruler | סרגל | P2 | K1 |  |
| desk | שולחן כתיבה | P2 | K1 | פענוחי S9 |
| board | לוח | P2 | K1 |  |
| page | עמוד | P2 | K1 |  |
| alphabet | אלפבית | P2 | K1 |  |
| question | שאלה | P2 | K1 |  |
| answer | תשובה; לענות | P2 | K1 |  |
| line | שורה, קו | P2 | K1 |  |
| right | נכון; ימין | P2 | K1 | Sunny לא אומרת "wrong" |
| sentence | משפט | P3 | K1 |  |
| example | דוגמה | P3 | K1 |  |
| poster | פוסטר | P3 | K1 | ★ |
| keyboard | מקלדת | P3 | K1 |  |
| homework | שיעורי בית | P3 | K1 |  |

#### 7.4.10 `PLY`: ספורט, משחק ופנאי (26)

| EN | עברית | P | מגיל | הערה |
|---|---|---|---|---|
| ball | כדור | P1 | K3 |  |
| dance | לרקוד; ריקוד | P1 | K3 |  |
| birthday | יום הולדת | P1 | K3 |  |
| game | משחק | P1 | K2 |  |
| soccer | כדורגל | P1 | K2 | UK: football |
| bike | אופניים | P1 | K2 |  |
| music | מוזיקה | P1 | K2 | ★ |
| fun | כיף | P1 | K2 |  |
| party | מסיבה | P1 | K2 | ★ (פארטי) |
| basketball | כדורסל | P2 | K2 |  |
| swimming | שחייה | P2 | K2 |  |
| kite | עפיפון | P2 | K2 |  |
| guitar | גיטרה | P2 | K2 | ★ |
| piano | פסנתר | P2 | K2 | ★ |
| drum | תוף | P2 | K2 |  |
| tennis | טניס | P2 | K1 | ★ |
| skateboard | סקייטבורד | P2 | K1 | ★ |
| video game | משחק מחשב | P2 | K1 |  |
| favorite | אהוב, מועדף | P2 | K1 | UK: favourite |
| photo | תמונה (צילום) | P3 | K1 | ★ |
| movie | סרט | P3 | K1 |  |
| team | קבוצה (בספורט) | P3 | K1 |  |
| hobby | תחביב | P3 | K1 | ★ |
| table tennis | פינג-פונג | P3 | K1 |  |
| badminton | בדמינטון | P3 | K1 | ★ |
| baseball | בייסבול | P3 | K1 | ★ |

#### 7.4.11 `TOY`: צעצועים (13)

| EN | עברית | P | מגיל | הערה |
|---|---|---|---|---|
| toy | צעצוע | P1 | K3 |  |
| doll | בובה | P1 | K3 |  |
| teddy bear | דובי | P1 | K3 |  |
| balloon | בלון | P1 | K3 | ★ |
| blocks | קוביות | P2 | K3 |  |
| bubbles | בועות | P2 | K3 |  |
| slide | מגלשה | P2 | K3 |  |
| swing | נדנדה | P2 | K3 |  |
| robot | רובוט | P1 | K2 | ★ |
| puzzle | פאזל | P2 | K2 | ★ |
| monster | מפלצת | P2 | K2 | ★ ידידותית בלבד |
| sandbox | ארגז חול | P3 | K2 |  |
| alien | חייזר | P3 | K1 |  |

#### 7.4.12 `TRN`: תחבורה (13)

| EN | עברית | P | מגיל | הערה |
|---|---|---|---|---|
| car | מכונית | P1 | K3 |  |
| bus | אוטובוס | P1 | K3 | ★ |
| train | רכבת | P1 | K3 |  |
| plane | מטוס | P1 | K2 |  |
| boat | סירה | P1 | K2 |  |
| truck | משאית | P1 | K2 | UK: lorry |
| helicopter | מסוק | P2 | K2 | ★ (הליקופטר) |
| taxi | מונית | P2 | K2 | ★ |
| hot-air balloon | כדור פורח | P2 | K2 | |
| tractor | טרקטור | P3 | K2 | ★ |
| scooter | קורקינט | P3 | K2 | ★ (סקוטר) |
| ship | אונייה | P2 | K1 | פענוחי S8 |
| motorcycle | אופנוע | P2 | K1 | Starters: motorbike |

#### 7.4.13 `WLD`: העולם סביבנו: טבע, מקומות, מזג אוויר (28)

| EN | עברית | P | מגיל | הערה |
|---|---|---|---|---|
| sun | שמש | P1 | K3 |  |
| moon | ירח | P1 | K3 |  |
| star | כוכב | P1 | K3 |  |
| tree | עץ | P1 | K3 |  |
| flower | פרח | P1 | K3 |  |
| rain | גשם | P1 | K2 |  |
| sea | ים | P1 | K2 |  |
| beach | חוף | P1 | K2 |  |
| park | פארק | P1 | K2 |  |
| sky | שמיים | P2 | K2 |  |
| cloud | ענן | P2 | K2 |  |
| snow | שלג | P2 | K2 |  |
| grass | דשא | P2 | K2 |  |
| leaf | עלה | P2 | K2 |  |
| sand | חול | P2 | K2 |  |
| shell | צדף | P2 | K2 |  |
| forest | יער | P2 | K2 | עולם המחנה |
| campfire | מדורה | P2 | K2 | עולם המחנה |
| rock | סלע, אבן | P2 | K2 | פענוחי S4 |
| wind | רוח | P2 | K1 |  |
| street | רחוב | P2 | K1 |  |
| store | חנות | P2 | K1 | UK: shop |
| weather | מזג אוויר | P3 | K1 |  |
| river | נהר | P3 | K1 |  |
| mountain | הר | P3 | K1 |  |
| city | עיר | P3 | K1 |  |
| world | עולם | P3 | K1 |  |
| place | מקום | P3 | K1 |  |

#### 7.4.14 `TIM`: זמן וימים (18)

| EN | עברית | P | מגיל | הערה |
|---|---|---|---|---|
| day | יום | P1 | K2 |  |
| night | לילה | P1 | K2 |  |
| morning | בוקר | P1 | K2 | Good morning |
| today | היום | P1 | K2 |  |
| afternoon | אחר הצהריים | P2 | K1 |  |
| evening | ערב | P2 | K1 |  |
| week | שבוע | P2 | K1 |  |
| time | זמן, שעה | P2 | K1 | What time is it? |
| o'clock | (שעה) עגולה | P2 | K1 | three o'clock |
| year | שנה | P2 | K1 |  |
| Sunday | יום ראשון | P2 | K1 | השבוע מתחיל ביום ראשון (00-B7) |
| Monday | יום שני | P2 | K1 |  |
| Tuesday | יום שלישי | P2 | K1 |  |
| Wednesday | יום רביעי | P2 | K1 |  |
| Thursday | יום חמישי | P2 | K1 |  |
| Friday | יום שישי | P2 | K1 |  |
| Saturday | שבת (היום) | P2 | K1 |  |
| tomorrow | מחר | P3 | K1 |  |

#### 7.4.15 `ADJ`: תארים ורגשות (36)

| EN | עברית | P | מגיל | הערה |
|---|---|---|---|---|
| big | גדול | P1 | K3 |  |
| small | קטן | P1 | K3 |  |
| happy | שמח | P1 | K3 |  |
| sad | עצוב | P1 | K3 |  |
| little | קטן, קטנטן | P1 | K2 |  |
| long | ארוך | P1 | K2 |  |
| new | חדש | P1 | K2 |  |
| tired | עייף | P1 | K2 |  |
| hungry | רעב | P1 | K2 |  |
| hot | חם | P1 | K2 |  |
| cold | קר | P1 | K2 |  |
| good | טוב | P1 | K2 |  |
| fine | בסדר | P1 | K2 | I'm fine |
| nice | נחמד | P1 | K2 |  |
| angry | כועס | P2 | K2 |  |
| scared | מפחד | P2 | K2 |  |
| funny | מצחיק | P2 | K2 |  |
| dirty | מלוכלך | P2 | K2 |  |
| fast | מהיר | P2 | K2 |  |
| slow | איטי | P2 | K2 |  |
| wet | רטוב | P2 | K2 | פענוחי S6 |
| open | פתוח; לפתוח | P2 | K2 |  |
| loud | רועש | P3 | K2 |  |
| quiet | שקט | P3 | K2 |  |
| short | קצר, נמוך | P2 | K1 |  |
| tall | גבוה (אדם) | P2 | K1 |  |
| old | ישן, זקן | P2 | K1 |  |
| bad | רע | P2 | K1 |  |
| beautiful | יפה | P2 | K1 |  |
| clean | נקי | P2 | K1 |  |
| thirsty | צמא | P2 | K1 |  |
| young | צעיר | P3 | K1 |  |
| dry | יבש | P3 | K1 |  |
| same | אותו דבר | P3 | K1 |  |
| easy | קל | P3 | K1 |  |
| hard | קשה | P3 | K1 |  |

#### 7.4.16 `VRB`: פעלים (רובם פקודות TPR) (59)

| EN | עברית | P | מגיל | הערה |
|---|---|---|---|---|
| jump | לקפוץ | P1 | K3 |  |
| run | לרוץ | P1 | K3 |  |
| walk | ללכת | P1 | K3 |  |
| clap | למחוא כפיים | P1 | K3 |  |
| stand up | לקום | P1 | K3 |  |
| sit down | לשבת | P1 | K3 |  |
| touch | לגעת | P1 | K3 |  |
| look | להסתכל | P1 | K3 |  |
| sing | לשיר | P1 | K3 |  |
| hop | לקפץ | P1 | K3 |  |
| turn around | להסתובב | P1 | K3 |  |
| stop | לעצור | P1 | K3 |  |
| eat | לאכול | P1 | K3 |  |
| sleep | לישון | P1 | K3 |  |
| wash | לרחוץ, לשטוף | P1 | K3 |  |
| play | לשחק | P1 | K3 |  |
| wave | לנופף | P1 | K3 |  |
| hug | לחבק | P1 | K3 |  |
| brush | להבריש, לצחצח | P2 | K3 |  |
| shake | לנער | P2 | K3 |  |
| stamp | לרקוע (ברגליים) | P2 | K3 |  |
| point | להצביע (על) | P1 | K2 |  |
| listen | להקשיב | P1 | K2 |  |
| say | להגיד | P1 | K2 |  |
| swim | לשחות | P1 | K2 |  |
| fly | לעוף | P1 | K2 |  |
| go | ללכת, לנסוע | P1 | K2 |  |
| come | לבוא | P1 | K2 |  |
| draw | לצייר | P1 | K2 |  |
| count | לספור | P1 | K2 |  |
| find | למצוא | P1 | K2 |  |
| have | יש לי, יש ל- | P1 | K2 |  |
| like | לאהוב, לחבב | P1 | K2 |  |
| can | יכול, יכולה | P1 | K2 |  |
| help | לעזור | P1 | K2 |  |
| make | להכין, ליצור | P1 | K2 |  |
| climb | לטפס | P2 | K2 |  |
| close | לסגור | P2 | K2 |  |
| paint | לצבוע | P2 | K2 |  |
| give | לתת | P2 | K2 |  |
| love | לאהוב | P2 | K2 |  |
| want | לרצות | P2 | K2 |  |
| throw | לזרוק | P2 | K2 |  |
| catch | לתפוס | P2 | K2 |  |
| kick | לבעוט | P2 | K2 |  |
| build | לבנות | P2 | K2 | עולם המחנה |
| laugh | לצחוק | P2 | K2 |  |
| smell | להריח | P3 | K2 |  |
| cry | לבכות | P3 | K2 |  |
| write | לכתוב | P1 | K1 |  |
| read | לקרוא | P1 | K1 |  |
| spell | לאיית | P2 | K1 |  |
| take | לקחת | P2 | K1 |  |
| know | לדעת | P2 | K1 |  |
| think | לחשוב | P2 | K1 |  |
| ride | לרכוב | P2 | K1 |  |
| live | לגור | P2 | K1 |  |
| put | לשים | P2 | K1 |  |
| show | להראות | P3 | K1 |  |

#### 7.4.17 `FNC`: מילות תפקוד: כינויים, יחס, שאלה (54)

| EN | עברית | P | מגיל | הערה |
|---|---|---|---|---|
| hello | שלום (בפגישה) | P1 | K3 |  |
| goodbye | להתראות | P1 | K3 | גם bye-bye |
| yes | כן | P1 | K3 |  |
| no | לא | P1 | K3 |  |
| more | עוד | P1 | K3 | K3: "More!" |
| all done | סיימנו | P1 | K3 | שגרת K3 |
| please | בבקשה | P1 | K2 |  |
| thank you | תודה | P1 | K2 |  |
| sorry | סליחה (התנצלות) | P1 | K2 |  |
| I | אני | P1 | K2 |  |
| you | את, אתה, אתם | P1 | K2 |  |
| my | שלי | P1 | K2 |  |
| your | שלך, שלכם | P1 | K2 |  |
| it | זה (לחפץ) | P1 | K2 | ERR.GR.IT |
| this | זה, זאת (קרוב) | P1 | K2 |  |
| in | בתוך | P1 | K2 |  |
| on | על | P1 | K2 |  |
| under | מתחת ל- | P1 | K2 |  |
| up | למעלה | P1 | K2 |  |
| down | למטה | P1 | K2 |  |
| what | מה | P1 | K2 |  |
| where | איפה | P1 | K2 |  |
| how many | כמה | P1 | K2 |  |
| and | ו- | P1 | K2 |  |
| a, an | (בלי תרגום: "אחד") | P1 | K2 | ERR.GR.ARTICLE |
| the | ה- (הידיעה) | P1 | K2 |  |
| he | הוא | P1 | K1 |  |
| she | היא | P1 | K1 |  |
| we | אנחנו | P1 | K1 |  |
| they | הם, הן | P1 | K1 |  |
| his | שלו | P2 | K1 |  |
| her | שלה; אותה | P2 | K1 |  |
| that | ההוא, ההיא; ש- | P2 | K1 |  |
| these | אלה | P2 | K1 |  |
| here | פה | P2 | K1 |  |
| there | שם | P2 | K1 | There is… |
| who | מי | P2 | K1 |  |
| how | איך | P2 | K1 |  |
| how old | בן כמה, בת כמה | P1 | K1 |  |
| next to | ליד | P2 | K1 |  |
| behind | מאחורי | P2 | K1 |  |
| in front of | מול, לפני | P2 | K1 |  |
| between | בין | P2 | K1 |  |
| not | לא (בשלילה) | P1 | K1 |  |
| or | או | P2 | K1 |  |
| but | אבל | P2 | K1 |  |
| with | עם | P2 | K1 |  |
| of | של | P2 | K1 |  |
| to | אל, ל- | P2 | K1 |  |
| some | כמה, קצת | P2 | K1 |  |
| lots of | הרבה | P2 | K1 |  |
| very | מאוד | P2 | K1 |  |
| too | גם | P2 | K1 |  |
| again | שוב | P1 | K2 | Sunny: "Again!" |

#### 7.4.18 `HOL`: חגים ועונות (הקשר ישראלי) (8)

| EN | עברית | P | מגיל | הערה |
|---|---|---|---|---|
| candle | נר | P1 | K3 | חנוכה, שבת |
| present | מתנה | P1 | K3 | US/UK; גם gift |
| holiday | חג | P2 | K2 |  |
| flag | דגל | P2 | K2 | יום העצמאות |
| Shabbat | שבת (החג) | P2 | K2 | מילה עברית שאולה |
| dreidel | סביבון | P3 | K2 | ★ (מיידיש) |
| doughnut | סופגנייה | P3 | K2 | US: donut |
| picnic | פיקניק | P3 | K2 | ★ ל"ג בעומר/עצמאות |

---

## 8. האזנה ודיבור: התקדמות Can-Do

### 8.1 סולמות

**האזנה (L)** ו**דיבור (S)** נמדדים בנפרד. **תקופה שקטה (S0) היא תקינה ומכובדת**: אף פריט לא דורש מילד
לדבר כדי להתקדם בהאזנה. MoE = הצהרת Pre-A1 של משרד החינוך [V]; ST = משימה בסגנון Starters [N].

| רמה | האזנה: הילד/ה יכול/ה… | דיבור: הילד/ה יכול/ה… | יישור | מי ומתי (יעד) |
|---|---|---|---|---|
| **0** | ליהנות משירים, לקשב לקול של Sunny, להגיב בגוף | (שקט) להקשיב בלבד, וזה תקין | — | K3 כל השנה; K2 בהתחלה |
| **1** | להגיב לשמו/ה, ל-hello/bye ול-5 פקודות תנועה עם הדגמה | להצטרף לשיר ולפזמון, ולחזור על מילה אחרי Sunny | MoE: "repeat short phrases"; "recite a short rhyme or chant" | K3 סוף שנה; K2 נובמבר–דצמבר |
| **2** | לבחור תמונה לפי שם (מתוך 2–4), ולבצע 10 פקודות בלי הדגמה | לקרוא בשם לתמונה במילה אחת, ולענות yes/no | MoE: "recognize Pre-Band I vocabulary and instructions" | K2 ינואר–מרץ; K1 בכניסה |
| **3** | להבין משפט קצר עם מילים מוכרות ומחווה ("Find the red ball"), ופקודה כפולה ("Touch your nose and jump") | צירופים קבועים: Hello! I'm Noa. I'm five. I like pizza. | MoE: "understand short, very simple questions… slowly, with visuals" | K2 סוף שנה; K1 דצמבר |
| **4** | שאלות על עצמו/ה, סיפור תמונות קצר (4–6 משפטים), מילות יחס, מספרים 1–20 | לענות בצירוף: It's a cat. Eight. Red. Yes, I do. ולבקש: Can I have…, please? | MoE: "ask someone's name / the time", "basic requests" | K1 פברואר–אפריל |
| **5** | משימות ST: שמע וחבר שמות, שמע וסמן, שמע וצבע, שם מאוית, מספרים | שיחה של 3–4 תורות, לשאול (What's your name? Do you like…?), לתאר תמונה (There is a…), לאיית את שמה | MoE: "ask for a spelling"; ST Speaking | K1 מאי–יוני |
| **6** | אי באנגלית בלבד של 2–4 דקות עם כפתור עזרה בלבד | — | — | K1 יוני (אתגר) |

### 8.2 בנק צירופים (`CH.*`)

| ID | Chunk | משמעות | מגיל | צומת |
|---|---|---|---|---|
| CH.HELLO | Hello! / Hi! | שלום | K3 | OR.S.GREET |
| CH.BYE | Bye-bye! / Goodbye! / See you! | להתראות | K3 | OR.S.GREET |
| CH.MORE | More, please! | עוד, בבקשה | K3 | OR.S.ASK |
| CH.ALLDONE | All done! | סיימנו! | K3 | — |
| CH.UHOH | Uh-oh! / Oops! | אופס | K3 | — |
| CH.YAY | Yay! / Hooray! | יש! | K3 | — |
| CH.MYTURN | My turn! / Your turn! | תורי / תורך | K2 | OR.S.ASK |
| CH.NAME | What's your name? — I'm ___ / My name is ___. | איך קוראים לך? | K2 | OR.S.NAME |
| CH.AGE | How old are you? — I'm five / eight. | בן/בת כמה? | K2 | OR.S.AGE |
| CH.HOWARE | How are you? — I'm fine / happy / tired / hungry. | מה שלומך? | K2 | OR.S.FEEL |
| CH.THANKS | Thank you! — You're welcome. | תודה / בבקשה | K2 | OR.S.ASK |
| CH.PLEASE | Can I have ___, please? — Here you are. | אפשר לקבל…? — הנה | K2 | OR.S.ASK |
| CH.WHATS | What's this? — It's a ___. | מה זה? — זה… | K2 | OR.S.WHATS |
| CH.COLOR | What color is it? — It's ___. | איזה צבע? | K2 | OR.S.WHATS |
| CH.LIKE | I like ___. / I don't like ___. | אני אוהב/ת… | K2 | OR.S.LIKE |
| CH.DOLIKE | Do you like ___? — Yes, I do. / No, I don't. | את/ה אוהב/ת…? | K1 | OR.S.LIKE |
| CH.HAVE | I have a ___. / I have got ___. | יש לי | K1 | OR.S.HAVE |
| CH.CAN | I can ___. / Can you ___? — Yes, I can. | אני יכול/ה | K1 | OR.S.CAN |
| CH.HOWMANY | How many ___? — Three ___. | כמה? | K2 | OR.L.NUM |
| CH.WHERE | Where is the ___? — It's in / on / under the ___. | איפה? | K2 | OR.L.PREP |
| CH.THEREIS | There is a ___. / There are two ___. | יש (שם)… | K1 | OR.S.DESC |
| CH.SPELL | How do you spell ___? — N-O-A. | איך מאייתים? | K1 | OR.S.SPELL |
| CH.TIME | What time is it? — It's three o'clock. | מה השעה? | K1 | OR.S.TIME |
| CH.LIGHT | Turn on / off the light. | תדליק/י / תכבה/י את האור | K1 | ERR.GR.LIGHT |
| CH.SORRY | Sorry! / Excuse me! | סליחה | K1 | OR.S.ASK |
| CH.DONTKNOW | I don't know. / Again, please. / Help, please! | לא יודע/ת / שוב, בבקשה / עזרה | K1 | OR.S.ASK (**כפתור "Again, please" מחליף את כפתור ההשמעה החוזרת באיים**) |
| CH.LETS | Let's go! / Let's play! | בואו נלך / נשחק | K2 | — |
| CH.GOODMORNING | Good morning! / Good night! | בוקר טוב / לילה טוב | K2 | שגרות משפחה |
| CH.SHABBAT | Shabbat Shalom! / Happy Hanukkah! / Happy birthday! | ברכות | K2 | HOL |

### 8.3 פקודות TPR ושפת כיתה (סדר הצגה)

פקודה נלמדת **קודם עם הדגמה של Sunny**, אחר כך בלי הדגמה, ובסוף במשחק "Sunny Says" עם מלכודת (K1).

| שלב | פקודות | מגיל |
|---|---|---|
| 1 | clap, jump, wave, stop, sit down, stand up | K3 |
| 2 | touch your (head/nose/toes…), turn around, hop, stamp, shake | K3 |
| 3 | look, listen, point to ___, find ___, show me ___ | K2 |
| 4 | walk, run (במקום), swim, fly, climb (פנטומימה), sleep, wake up | K2 |
| 5 | Give me ___. Put the ___ on/in/under the ___. Open/close (your eyes, the box) | K2 |
| 6 | Say ___. Again. Your turn. Draw a ___. Color it ___. Count. | K2/K1 |
| 7 | Read. Write. Spell. Circle. Tick. Match. Draw a line. (שפת מבחן Starters) | K1 |
| 8 | פקודות כפולות: "Touch your nose and jump." | K2 סוף שנה, K1 |

---

## 9. שירים ופזמונים

### 9.1 מדיניות רישוי

- **PD-safe:** פורסם לפני 1927, או מסורתי עם מקור מודפס ישן (Gutenberg: *Our Old Nursery Rhymes* 1911,
  *Traditional Nursery Songs of England* 1843) [V].
- **TRAD-verify:** שיר עם שורשים עממיים שיש עליו טענת מחבר מודרני, או שהסטטוס שלו לא ברור. משתמשים בו רק אחרי
  בדיקה (משימה V4). עד שהבדיקה מסתיימת משתמשים ב-**ORIG**: מילים שלנו על לחן PD.
- **ORIG:** מילים חדשות שאנחנו כותבים על לחן PD.
- **אסור:** Baby Shark (עיבוד Pinkfong), כל הקלטה של צד שלישי, ו-"Ten Little Indians" (פוגעני). במקומו: "Ten
  Little Fingers" או ORIG.
- **כל ההקלטות שלנו.** קול: דובר/ת ילידי/ת (§12.4). ליווי: עיבוד MIDI שלנו עם soundfont ברישיון CC0, או ספק
  מוזיקה שהתנאים שלו מתירים (00-E8a).
- **מקור הטקסט** של כל שיר נרשם ביומן הרישוי (00-E10), כולל כתובת המהדורה הישנה.

### 9.2 רשימת השירים

| ID | שיר | שכבה | מילות יעד | תנועות TPR | ילדים | עונה |
|---|---|---|---|---|---|---|
| `HELLO` | "Hello, Hello!" (ORIG, לחן *Frère Jacques*) | ORIG | hello, how are you, fine, thank you | נפנוף; הצבעה על עצמי; אגודל למעלה | K3 K2 K1 | פתיחה קבועה של כל אי |
| `BYE` | "Goodbye, Goodbye" (ORIG, לחן *Goodnight, Ladies*, 1847) | ORIG | goodbye, see you, friend | נפנוף, נשיקה באוויר | K3 K2 K1 | סגירה קבועה |
| `TTLS` | Twinkle, Twinkle, Little Star (Jane Taylor, 1806) | PD-safe | star, little, up, sky | פתיחה וסגירה של אצבעות; ידיים למעלה; צורת יהלום | K3 K2 | כל השנה, שיר ערב |
| `HSKT` | Head, Shoulders, Knees and Toes (לחן *There Is a Tavern in the Town*, 1883 = PD; **המילים** מתועדות רק מ-1961 → TRAD-verify; עד האימות: ORIG על אותו לחן, אותן 8 מילות גוף) | TRAD-verify | head, shoulders, knees, toes, eyes, ears, mouth, nose | נגיעה בכל איבר; בסיבוב השני מהר יותר | K3 K2 K1 | ינואר |
| `IYH` | If You're Happy and You Know It | TRAD-verify (חלופת ORIG: "If you're ready…") | happy, clap, stamp, hooray | מחיאה, רקיעה, "Hooray!" | K3 K2 | נובמבר |
| `RRRB` | Row, Row, Row Your Boat (1852) | PD-safe | row, boat, merrily | חתירה, הורה וילד פנים מול פנים ("lap mode") | K3 K2 | כל השנה |
| `ITSY` | Itsy Bitsy Spider (גרסת 1910) | PD-safe | spider, up, down, rain, sun | אצבעות מטפסות; גשם יורד; שמש בידיים | K3 K2 | חורף |
| `MULB` | "This Is the Way…" (לחן *Mulberry Bush*, המאה ה-19) | PD-safe + בתים ORIG | wash, brush, teeth, hands, eat, sleep | פנטומימה של השגרה | K3 K2 | שגרות אמבטיה ובוקר |
| `OMD` | Old MacDonald Had a Farm | PD-safe | cow, duck, pig, sheep, horse, dog, cat | חיקוי חיות וקולות | K3 K2 | פברואר |
| `PAT` | Pat-a-cake (1698) | PD-safe; "mark it with **N**" עם האות של הילד | cake, baby, oven (אגב) | מחיאות כפיים בזוג עם הורה | K3 | כל השנה |
| `BAA` | Baa, Baa, Black Sheep (1744) | PD-safe | sheep, black, three, yes | ספירה באצבעות; נדנוד ראש yes | K3 K2 | — |
| `TEDDY` | Teddy Bear, Teddy Bear, Turn Around | TRAD-verify | turn around, touch the ground, shoe | כל שורה היא פקודה | K3 K2 K1 | — |
| `ABC` | Alphabet Song (Bradlee, 1835, לחן Twinkle) + סיום ORIG: "…come and sing along with me!" | PD-safe + ORIG | שמות האותיות, **zee** | הצבעה על פסי אותיות שמוארים משמאל לימין | K2 (חשיפה) K1 | נובמבר ואילך |
| `BINGO` | B-I-N-G-O (שנות ה-1780) | PD-safe | שמות אותיות B I N G O, dog | מחיאה במקום אות שנעלמת | K2 K1 | דצמבר |
| `ONE2` | One, Two, Buckle My Shoe (1805) | PD-safe | 1–10, shoe, door, sticks | פנטומימה לכל זוג מספרים | K2 K1 | נובמבר |
| `THIS` | This Old Man (1906) | PD-safe | 1–10, חרוזים (one/thumb, two/shoe) | תיפוף על איבר לכל מספר | K1 (PA חרוז) | ינואר |
| `TEN` | Ten in the Bed | TRAD-verify | 10→1, roll over, little | גלגול ידיים, ספירה לאחור | K2 K1 | מרץ |
| `HICK` | Hickory Dickory Dock (1744) | PD-safe + בתים ORIG (two, three o'clock) | clock, mouse, one, o'clock | ידיים כמטוטלת; עכבר מטפס על היד | K1 | מאי (TIME) |
| `RAIN` | Rain, Rain, Go Away (המאה ה-17) | PD-safe | rain, go away, come again, play | גשם באצבעות; "go away" בדחיפה | K2 K1 | חורף |
| `SKIP` | Skip to My Lou (שנות ה-1840) + בתי פעולה ORIG ("Jump, jump, jump with me") | PD-safe + ORIG | jump, hop, run, clap | הפעולה עצמה | K2 K1 | אביב |
| `COLORS` | "Rainbow, Rainbow" (ORIG, לחן *Mary Had a Little Lamb*, 1830) | ORIG | צבעים | הרמת כרטיס או חפץ בצבע | K3 K2 | נובמבר |
| `DAYS` | "Sunday, Monday…" (ORIG, לחן *Clementine*, 1884). השבוע מתחיל ביום ראשון | ORIG | ימי השבוע | יד מטפסת יום-יום | K1 | פברואר |
| `CANDLES` | "Eight Little Candles" (ORIG, לחן *Michael Finnegan*; ⟂VERIFY תאריך הדפסה ראשון של הלחן לפני 1927, אחרת לחן *This Old Man* 1906) | ORIG | 1–8, candle, light, Happy Hanukkah | אצבע נדלקת לכל נר | K3 K2 K1 | חנוכה |
| `COSTUME` | "What Are You?" (ORIG, לחן *London Bridge*) "I am a lion / a robot…" | ORIG | I am a…, costume, mask | פוזה של הדמות | K2 K1 | פורים |
| `HBD` | Happy Birthday to You (PD בארה"ב מאז 2016) [BK] | PD (US) | happy birthday, you | נשיפה על נרות | כולם | ימי הולדת |
| `CLEAN` | "Clean Up" (ORIG, לחן *Mulberry Bush*) | ORIG | clean up, put away, toys | איסוף | K3 K2 | סיום פרק |
| `PHON-*` | 26+ פזמוני צלילים (ORIG). לכל גרפמה 2 שורות ותנועת יד | ORIG | גרפמה וצליל ו-3 מילים | תנועת יד ייחודית לכל צליל (§9.3) | K1 (K2 אגב) | לפי S1–S8 |

### 9.3 פזמוני צלילים (PHON) ותנועות

כל פזמון בנוי לפי התבנית: `/sss/, /sss/, the balloon says /sss/ · sun and sock and sand!`. אחריו באה תנועת
יד קבועה, **מקורית שלנו** (למשל s: לוחצים לאט על בלון דמיוני שמשחרר אוויר; a: נגיסה בתפוח עם פה פתוח לרווחה;
‏t: שתי נקישות אצבע על השולחן, "t-t"; p: פיצוץ בועה באצבע, "p!"). **לא מעתיקים** את הטקסטים או את התנועות
של Jolly Phonics (נחש, נמלים על הזרוע, צפייה בטניס, כיבוי נרות וכן הלאה). רעיון "תנועה לכל צליל" אינו מוגן,
אבל הביצוע שלהם כן, ולכן בדיקת QA משווה את רשימת התנועות שלנו לרשימה הידועה ופוסלת התאמות. רשימה מלאה של 45 תנועות
תיכתב בשלב התוכן, בפורמט `phon_actions.json`, עם אישור טעם של ההורה (00-E10).

---

## 10. הוראה בתיווך עברית: סנדוויץ', איים באנגלית ודעיכה

### 10.1 תפקידים קוליים (OPOL)

| דמות | שפה | מיקום על המסך | קול | תפקיד |
|---|---|---|---|---|
| **Sunny** (CH-02, לוטרת ים, נקבה, 01 §4.3) | **אנגלית בלבד, תמיד.** גם כשהילד מדבר עברית, Sunny עונה באנגלית, עם מחווה ותמונה | **שמאל** (הצד שבו מתחילה הקריאה באנגלית) | קול נשי en-US בוגר וחם, 0.85× (01), עיצורים ברורים; לא קול ילדותי (00-E8a) | מודל לשון, מבקשת, שרה |
| **ניצי** (CH-01, הקמע) | עברית | **ימין**, "עומד לידה" | קול ה-TTS העברי של ניצי | מתרגם בסנדוויץ', הוראות, הסברים, רגש ועידוד |
| **הורה** | מה שנוח להורה | — | — | משחק משותף, אישורים, שגרות. **לא חייב לדבר אנגלית "נכונה"**. המודל מגיע מהשמע |

**Sunny לא AI** (00-C7): כל השורות שלה כתובות מראש. בכל מקום שיש בו AI, הוא דמות נפרדת שמסומנת כמכונה.

### 10.2 כללי סנדוויץ' (SR)

| # | כלל |
|---|---|
| SR-1 | **חשיפה ראשונה למילה חדשה:** Sunny אומרת (EN, עם תמונה ומחווה) ← השהיה של 400–600 מ"ש ← ניצי אומר **רק את הגלוס**, בשקט יותר ובלי מסגרת ("תפוח") ← השהיה ← Sunny חוזרת ומזמין: "Apple! Your turn!" |
| SR-2 | **העברית לעולם לא יוצאת מפי Sunny, והאנגלית לעולם לא יוצאת מפי ניצי.** כשניצי צריך לציין מילה אנגלית ("המילה ‹cat› מתחילה ב…"), **משבצים קליפ של Sunny** בתוך המשפט העברי, עם "שבב" חזותי LTR מבודד. ה-TTS העברי לא מקבל אף פעם תווים לטיניים (AC4). |
| SR-3 | **הוראות בעברית** ל-K3 ול-K2, ובהן "מילת פקודה" אנגלית שנלמדת (`Listen!`, `Find!`, `Touch!`). אחרי שליטה ב-`E.OR.L.CMD.REC` הפקודה ניתנת באנגלית בלבד, וניצי שותק. |
| SR-4 | **אין תרגום בתוך שיר.** לפני שיר חדש ניצי אומר משפט אחד בעברית על נושא השיר. השיר עצמו באנגלית בלבד. |
| SR-5 | **הסברי כללים (K1)** בעברית, עד 12 שנ', עם דוגמאות אנגליות מבודדות ("כשבסוף המילה יש e שקטה, היא נותנת לתנועה להגיד את שמה: cap ← cape"). |
| SR-6 | **גלוס לפי מילה, לא לפי מסך.** הגלוס נקבע לפי `glossLevel` של המילה אצל הילד הספציפי (§10.3). |
| SR-7 | **"?בעברית" על פי דרישה** (K1 ו-K2): כפתור קבוע שמשמיע את הגלוס של המשפט האחרון, ואחריו Sunny חוזרת על המשפט. השימוש נרשם כ"נעזר". אין קנס, אבל השימוש נכלל בדרישת "בלי רמז" של שליטה. |
| SR-8 | **לא שואלים "מה זה בעברית?" כפריט נבחן.** המטרה היא הבנה, לא תרגום. הבדיקה תמיד מבוססת תמונה או פעולה. |
| SR-9 | **אין ערבוב בתוך צומת טקסט** אצל קוראים מתחילים: מחרוזת עברית ומחרוזת אנגלית הן שני רכיבים נפרדים (bidi isolation). |
| SR-10 | **רגש תמיד בעברית מותר**: כשהילד מתוסכל, ניצי נכנס ("רוצה שננסה יחד?") גם באמצע אי באנגלית. אחרי זה האי נסגר בשיר הסיום, ולא נקטע בפתאומיות. |

### 10.3 סולם הגלוס לפי מילה (`glossLevel`)

| רמה | התנהגות | מעבר למטה (דעיכה) | מעבר למעלה (חזרה) |
|---|---|---|---|
| **G3** סנדוויץ' אוטומטי | EN ← HE ← EN בכל חשיפה | אחרי 3 חשיפות עם תגובה נכונה (הקשה או תנועה) על פני יומיים לפחות | — |
| **G2** גלוס מושהה | EN ← המתנה של 2.5 שנ' ← אם אין תגובה או שהייתה טעות: HE ← EN | 2 תשובות נכונות בלי גלוס, בימים שונים (`lexState = familiar`) | 2 טעויות רצופות במילה |
| **G1** גלוס על פי דרישה | רק בכפתור "?בעברית" | המילה נשלטה (M-REC/M-STD) | טעות בבדיקה מושהית |
| **G0** בלי גלוס | הכפתור עדיין קיים ב-K1 ולא קיים ב-K2 | — | טעות בבדיקה מושהית: חזרה ל-G1 |

ברירת מחדל למילה חדשה: G3 ב-K3 וב-K2. ב-K1, ‏G3 למילים P2/P3 ו-**G2** למילים P1 שמופיעות כבר ברשימת
הבית-ספר שההורה הזין. **צירופים (CH)** עוברים את אותו סולם. **הוראות** עוברות סולם מקביל לפי `instrId`.

### 10.4 עיצוב "אי באנגלית" (English island)

- **כניסה:** `sfx.english.enter` (01), ואחריו מעבר "דף" שמחליק משמאל, טוקן `palette.english` (01) וטוקני הגופן `type.en.*` (09),
  והסירה של Sunny עוגנת משמאל. **יציאה:** שיר `BYE`, ג'ינגל הפוך, וחזרה למעטפת RTL.
- **בתוך האי:** כל הפריסה LTR (סדר הקשות, חיצים, פס התקדמות שמתמלא משמאל לימין), בתוך המכל `CMP-22 EnglishIsland` של 09 (`direction:'ltr'`).
  הכפתורים הקבועים ("?בעברית", "Again, please") נשארים באותו מקום בכל האיים; **כפתור היציאה לעברית נשאר במיקום ה-end של הממשק העברי**
  (09 §5 כלל 7), כדי שהילד ימצא "הביתה" תמיד באותו מקום. ניווט ה-shell מוסתר. כפתור ההפסקה של 01 §6.4 (דובדובה) נשאר זמין גם באי.
- **ילד שמערבב LTR/RTL:** האי לעולם לא מציג טקסט עברי ואנגלי באותו צומת (SR-9); פס ההתקדמות, החיצים והמשבצות מונפשים תמיד משמאל;
  סטודיו הקוד של 06 (רצועת G זורמת RTL) **לעולם לא מופיע בתוך האי** ולא באותו פרק עם בלוק אנגלית (AC19). שגיאות כיוון נרשמות כ-`ERR.DIR.*` (§3.5).
- **אורך אי לפי ילד** (אנגלית רציפה בלבד, בלי ניצי):

| | נובמבר | ינואר | מרץ | יוני |
|---|---|---|---|---|
| K1 | 60–90 שנ' | 2 דק' | 3 דק' | 4–5 דק' |
| K2 | 20–40 שנ' (שיר או דיאלוג) | 40–60 שנ' | 60 שנ' | 60–90 שנ' |
| K3 | שיר אחד (1–2 דק') | = | = | 2 שירים |

- **כלל התאמה:** האי מתארך בצעד אחד כשמתקיימים שני תנאים על פני 5 האיים האחרונים: שימוש ב"?בעברית" בפחות מ-20%
  מהתורות, ודיוק של לפחות 80%. האי מתקצר כשהשימוש בעזרה עולה על 40% (`SIG.GLOSS_DEPEND`), או כשהילד יוצא
  באמצע פעמיים.

### 10.5 סיכום לפי ילד

| | K1 | K2 | K3 |
|---|---|---|---|
| הוראות | עברית ← פקודות אנגליות בהדרגה; עד מרץ רוב ההוראות באי באנגלית | עברית + מילות פקודה אנגליות | עברית (מספר או הורה) |
| מילים חדשות | G2/G3 ← דעיכה מהירה | G3 ← דעיכה איטית | G3 תמיד (בלי דעיכה; ההורה יכול לכבות) |
| הסברים | עברית קצרה (SR-5) | אין הסברי כללים, רק משחק | אין |
| עזרה | "?בעברית" תמיד | "?בעברית" עד G0, ואז הכפתור נעלם | ההורה |
| מתי העברית נסוגה | לפי מילה (§10.3) ולפי אי (§10.4) | לפי מילה | לא נסוגה. בגיל הזה המטרה היא היכרות |

### 10.6 Sunny: דמות, סגנון תסריט וטיפוגרפיה

**אישיות (01 §4.3):** לוטרת ים שמחה ומוזיקלית שאוהבת סירות ושירים. סבלנית, ומרבה במחוות. תמיד מחזיקה חפץ
(צדף, ספר, יוקלילי). מעיל גשם צהוב וכובע מלחים עם סרט טורקיז. **טעויות מכוונות לתיקון (protégé)** שייכות ל-**Mr.
Mix-Up** ולא ל-Sunny: הוא "מתבלבל" ("Is this a hat? …No? Oh! It's a cat!"), והילדים מתקנים. Sunny פונה לילד
בשמו וב-you, ולכן אין צורך בווריאנטים של מגדר באנגלית.

**כללי סגנון תסריט (ל-writers ולפרומפט ה-LLM; נאכפים על ידי lint, AC14):**

| כלל | K3 | K2 | K1 |
|---|---|---|---|
| מילים במבע | 1–3 | עד 5 | עד 8 (באיים עד 10) |
| מילים חדשות במבע | 1 | 1 | 1, כל השאר מוכרות (i+1) |
| מבנים | שם עצם בודד, פקודה, "More?" | פקודה, It's a…, I like…, Where's…? | הווה פשוט וממושך, can, have, There is/are, שאלות wh, ‏yes/no |
| חזרה | כל מילת יעד 3 פעמים לפחות בפרק | 3 פעמים לפחות | 2 פעמים לפחות + מחזור בפרק הבא |
| קצב | 0.85× (ברירת המחדל של 01) | 0.85× | 0.85×, ועד 1.0 בהדרגה באיים ארוכים; כפתור האטה 0.75× |
| קיצורים | רק it's, let's | it's, I'm, let's, what's, where's | כל הקיצורים הנפוצים |
| אסור | ניבים, אירוניה, שאלה פתוחה בלי תמונה | כנ"ל | ניבים, slang |

**משפטי מפתח (מרונדרים מראש, תמיד אותו ביצוע; הרשימה של 01 §4.3 ועוד כמה):** "Ahoy, {name}!", "Your turn!",
"Listen… and do!", "You found it: *boat*!", "Let's sing!", "Bye-bye, see you tomorrow!", ובנוסף "Again!", "Let's
do it together.", "Look!"

**משוב (00-C4, תהליך בלבד):**

- **מאושר:** "You listened carefully!", "You sounded it out!", "You found it!", "You fixed it!", "You tried a new
  word!", "Left to right, just right!", "You kept trying!"
- **אסור:** "Smart girl/boy!", "Genius!", "You're the best!", "Perfect!", "Wrong!", "No!", "That's bad."
- **אחרי טעות** (recast, בלי שלילה): "This is a **cat**. Cat! Let's find the cat." האות הנכונה מהבהבת, ואחרי
  1–3 פריטים מוצג פריט דומה (learning_science D14).

**טיפוגרפיה (מסירה למפרט ה-UX):**

- גופן: **Andika** (SIL OFL 1.1), תואם ל-`font.en` ב-09. הגופן תוכנן לקוראים מתחילים: ‏a ו-g בקומה אחת, ו-I, ‏l ו-1 שונות זו מזו.
  הוא נטען דרך `expo-font`, ויש בדיקת רינדור על המכשיר.
- גדלים: לפי טוקני `type.en.*` של `09-ux-ui-design-system.md` (‏`type.en.k2.word` 44, ‏`type.en.k1` 26). **בקשה ל-09
  (§20.2):** טוקן נוסף `type.en.k1.decode` של 36pt לפחות למילה בודדת בפריטי פענוח, ו-`type.en.letter` של 64pt לפחות
  לאות בודדת. ריווח מילים רחב. בלי italics, בלי all-caps (מלבד אותיות גדולות בודדות), בלי יישור לשני הצדדים.
- **אריחי גרפמה:** דיגרף הוא אריח אחד (sh). אות כפולה היא אריח אחד (ll). ‏e שקטה מחוברת בקשת לתנועה. תנועות
  מסומנות בצבע תנועה עקבי, כדי לחזק את "לכל מילה יש תנועה" (ERR.SP.VOWEL_OMIT).
- כתב יד בדפים מודפסים: print manuscript ("ball and stick"), בהתאמה לכתב שבית הספר מלמד. ההורה מאמת מול המחברת
  (§19).

---

## 11. תבניות פריטים (Item Templates)

### 11.1 סכמה משותפת (pseudo-JSON, ‏`packages/content/en/templates/*.json`)

```jsonc
{
  "templateId": "EN.T01",
  "name": "listen_tap_picture",
  "skills": ["E.VO.*.REC", "E.OR.L.*.REC"],
  "bands": ["K3", "K2", "K1"],
  "mechanic": "MX-E04",                  // §4 (word-harbor)
  "params": {
    "choices": { "K3": 2, "K2": [2, 3, 4], "K1": [3, 4] },   // מתרחב עם הדיוק
    "distractorPolicy": ["sameTopic", "phonSimilar", "errorCode"],  // לפי רמת קושי
    "carrier": { "K3": "{word}!", "K2": "Find the {word}!", "K1": ["Find the {word}.", "Where's the {adj} {word}?"] }
  },
  "stimulus": { "audio": "sunny:{carrier}", "glossLevel": "per-word" },   // §10.3
  "response": { "type": "tap", "targetMinMm": { "K3": 20, "K2": 15, "K1": 12 } },
  "scoring": { "firstAttempt": true, "rt": "off<K1|softK1" },           // D11
  "scaffold": ["replay", "glossHe", "eliminateOne", "highlightTarget"],  // סדר רמזים; D10
  "feedback": { "correct": "sunny:praise.process", "incorrect": "recast" },
  "errorMap": { "sameTopicPick": "ERR.VO.SEMANTIC" },
  "telemetry": ["itemId", "choiceId", "attempt", "hintsUsed", "glossUsed", "rtMs"]
}
```

### 11.2 קטלוג התבניות

| ID | שם | מיומנויות | ילדים | תיאור | מסיחים מקוד |
|---|---|---|---|---|---|
| T01 | listen_tap_picture | VO.REC, OR.L | כולם | Sunny אומרת, הילד מקיש על תמונה | נושא זהה ← דמיון צלילי ← קוד שגיאה |
| T02 | tpr_command | OR.L.CMD, VO.VRB | כולם | Sunny נותנת פקודה, הילד זז, וההורה או הילד מאשר (👍). אין מצלמה ב-v1 | — |
| T03 | sound_to_letter | PH.GPC, AL.NAME | K1 | שומעים /m/ ומקישים על m (בחירה מ-3) | אותיות שנראות דומות (b/d) ו-PR (w/v) |
| T04 | letter_to_picture | PH.GPC | K1 | רואים m ובוחרים תמונה שמתחילה ב-/m/ | צליל פותח קרוב |
| T05 | first_sound_fishing | PA.ONSET/FINAL/MED | K2, K1 | "Which one starts with /s/?" (דגים מתמונות) | — |
| T06 | shell_blend | PH.BLEND, PA.PHON.BLEND | K1 | צדפים בשורה (MX-E03), הקשה על כל צדף משמאל לימין, החלקה, והמילה קמה לחיים כ-Wordling | — (פריט פתוח). אחרי מיזוג: בחירת תמונה מ-3, כולל המילה ההפוכה |
| T07 | mixup_spell | PH.SEG, WR.DICT | K1 | שומעים מילה וגוררים אותיות למשבצות LTR. במאגר יש +1–2 אותיות מסיחות | אות התנועה של זוג מינימלי; b/d |
| T08 | real_or_silly | PH.BLEND | K1 | מפענחים ומחליטים: מילה אמיתית (לחפץ) או "טיפשית" (ליצור) | — |
| T09 | echo_pairs | PR.MP.* | K1 (K2 גרסה רכה) | AX: שומעים מילה ובוחרים בין 2 תמונות. ABX ברמה גבוהה. דובר משתנה | בן הזוג המינימלי |
| T10 | lighthouse_tricky | TW.READ | K1 | Sunny אומרת מילה מסובכת, ותופסים אותה מתוך 3–4 | מילים מסובכות דומות (was/saw, of/off) |
| T11 | logbook_story | RD.*, PH.READ | K1 | סיפור פענוחי, הקשה לשמיעה, ושאלת תמונה | — |
| T12 | sunny_chat | OR.S.* | K2, K1 | Sunny שואלת, הילד בוחר בועה (2–3) **או** אומר (הקלט והשווה, בלי ציון) | בועה עם שגיאת העברה (ERR.GR.*) |
| T13 | song_karaoke | SG.FILL | K2, K1 | השיר נעצר, וממלאים מילה (תמונה) | מילה מאותו שיר |
| T14 | case_match | DIR.CASE.MATCH | K1 | מתאימים אות גדולה לאות קטנה (זיכרון או גרירה) | Gg/Qq, Bb/Dd |
| T15 | letter_trace_free | AL.FORM, DIR.LTR.WRITE | K1 | הדגמה ← 1 עקיבה ← כתיבה חופשית על המסך ← דף מודפס | — (בדיקת כיוון משיכה ונקודת התחלה) |
| T16 | sentence_build | WR.SENT, DIR.PUNCT | K1 | אריחי מילים למשפט, עם אות גדולה ונקודה | אריח בסדר עברי (ERR.GR.ADJ_ORDER) |
| T17 | rhyme_train | PA.RHYME | K2, K1 | קרונות מחורזים | — |
| T18 | listen_and_color | OR.L.COLOR | K1 (K2 פשוט) | "Color the ball red." (Starters Listening 4) | — |
| T19 | spell_name_listen | OR.L.SPELL, AL.NAME | K1 | Sunny מאייתת שם ("S-A-M"), ובוחרים כרטיס | שם עם אות דומה |
| T20 | peekaboo_discover | VO (X-EXP) | K3 | מקישים על דלת, יוצאת תמונה (גם תמונת משפחה, D14), ו-Sunny אומרת את המילה. אין שאלה | — |

### 11.3 פריטים לדוגמה

```jsonc
// T06 shell_blend — E.PH.GOK.BLEND
{ "itemId": "ph.gok.blend.0007", "templateId": "EN.T06", "skill": "E.PH.GOK.BLEND",
  "word": "pot", "graphemes": ["p","o","t"], "isReal": true, "decodableCheck": "pass@GOK",
  "audio": { "phonemes": ["ph_p","ph_aa","ph_t"], "word": "w_pot_sunny" },
  "afterBlend": { "pick": ["pot","top","pat"], "pictures": ["img_pot","img_top_toy","img_pat_dog"] },
  "errorMap": { "top": "ERR.DIR.RTL_SCAN", "pat": "ERR.PH.VOWEL_AUO" },   // top = קריאה הפוכה; pat = /ɑ/↔/æ/
  "reward": { "wordling": "pot", "campItem": "camp_pot" },     // שילוב פנימי: הסיר קם לחיים ונכנס למחנה
  "difficulty": 2 }

// T09 echo_pairs — E.PR.MP.WV
{ "itemId": "pr.wv.0003", "templateId": "EN.T09", "skill": "E.PR.MP.WV",
  "pair": ["wet","vet"], "target": "vet", "format": "AX",
  "talker": "en_us_m2",                     // מתחלף; M-PR דורש ≥3 דוברים
  "pictures": { "wet": "img_wet_dog", "vet": "img_vet" },
  "feedback": { "incorrect": "replayBoth+mouthCard:v" } }

// T12 sunny_chat — E.OR.S.AGE.SAY (K2)
{ "itemId": "or.age.0001", "templateId": "EN.T12", "skill": "E.OR.S.AGE.SAY",
  "pipLine": "How old are you?", "gesture": "fingers",
  "bubbles": [ { "text": "I'm five.", "audio": "ch_age_five", "correct": true },
               { "text": "I have five years.", "audio": "ch_age_err", "errorCode": "ERR.GR.AGE" } ],
  "speakOption": { "mode": "recordCompare", "score": false, "parentConfirm": true },
  "personalize": { "age": "profile.age" } }

// T11 logbook_story — E.PH.GOK.READ
{ "itemId": "rd.gok.story.002", "templateId": "EN.T11", "unit": "GOK",
  "pages": ["Pip got a pig.", "The pig is on top.", "No, pig! Go, go, go!"],
  "pictureWords": [], "trickyUsed": ["the","no","go"],
  "question": { "pip": "Where is the pig?", "choices": ["img_pig_top","img_pig_mud"], "answer": 0 },
  "readToSibling": true }
```

---

## 12. דרישות שמע

### 12.1 עקרונות

1. **הכול מוקלט או מרונדר מראש** ונשמר במטמון ב-R2 (00-E9). אין TTS חי לילדים.
2. **אנגלית אמריקאית (en-US) אחת ועקבית** בכל השמע האנגלי. ספק ה-TTS האנגלי הוא הספק שנבחר בתחרות הקולות (00-E9).
3. **פונמות בודדות לעולם לא ב-TTS.** הן מוקלטות על ידי דובר/ת ילידי/ת. גיבוי: קליפי Wikimedia IPA ‏(CC BY-SA)
   עם ייחוס [N].
4. **ה-TTS העברי לעולם לא מבטא אנגלית** (SR-2). מילים אנגליות בתוך שורה עברית הן קליפים של Sunny ששובצו בשורה.

### 12.2 מלאי קבצים

| קטגוריה | מקור | כמות (משוער) | הערות |
|---|---|---|---|
| פונמות (§6) | **הקלטה אנושית** | 45 IDs × 2 סגנונות (רגיל/מתוח או קטוע) × **3 טייקים** = 270 | ה-QA בוחר טייק אחד לכל סגנון. **טבלת §6 היא הרשימה הקובעת**; 12 §6.10 ("44 הצלילים") חייב לשקף אותה (ההבדל: שווא `ph_schwa` וצירופים `ph_ks`/`ph_kw`; §20.2) |
| שמות אותיות | **הקלטה אנושית** | 26 (z = "zee") | |
| מילות מפתח לפונמות | הקלטה אנושית | 45 | אותו דובר. נשמעות מיד אחרי הצליל ("/m/… moon") |
| שירים (§9.2) | **הקלטה אנושית** (קול) + ליווי | כ-27 שירים + 45 פזמוני צלילים | ראו §12.4 ו-§20 |
| מילים (§7) | TTS, ‏Sunny | 534 × 1 + נשאים ("Find the ___") | |
| זוגות מינימליים (§5.4) | TTS בכמה דוברים + הקלטה אנושית | כ-70 מילים × **4 דוברים** (2 נשים, 2 גברים; מבוגרים) + **16 זוגות (32 מילים) בהקלטה אנושית**, הרשימה של 12 §6.10 (ship/sheep, bit/beat, pull/pool, full/fool, bat/bet, cap/cup, cat/cut, hot/hut, pen/pan, thin/sin, think/sink, three/tree, wet/vet, wine/vine, chip/ship, bag/back) | HVPT [BK]. הדובר האנושי נספר כדובר חמישי |
| מילים פענוחיות (§5.3) | TTS, ‏Sunny | כ-450 | נבנות גם כ"פירוק" (§12.5) |
| מילים מומצאות | **הקלטה אנושית** | כ-60 | TTS נוטה "לתקן" מילים מומצאות למילים אמיתיות |
| שורות Sunny | TTS | כ-800 בשנה הראשונה | כולל וריאנטים של שמות (§19) |
| צירופים (§8.2) | TTS, ‏Sunny + קולות שיחה | 29 × 2 קולות | Sunny ועוד קול "חבר" אחד לדיאלוגים |
| גלוסים בעברית | TTS עברי (00-E9) | 534 + 29 | ניקוד, ‏m/f/pl לפי הצורך |

### 12.3 מפרט הקלטת הפונמות (session brief לדובר/ת)

- **דובר/ת:** ‏General American ילידי/ת, מבוגר/ת, קול חם וברור. רצוי ניסיון בהוראת פוניקה (מורה K-2). מבוגר
  ולא ילד, כמתחייב מתנאי הספקים ומ-00-E8a.
- **נמשכים** (s, m, n, f, l, v, z, sh, th, ‏ð, ‏ŋ, ‏r, ‏w, ‏j, תנועות): (א) רגיל, 400–600 מ"ש; (ב) מתוח, 1.2–1.5 שנ'.
- **סותמים** (p, b, t, d, k, g, ch, j, ks, kw): קטוע, **בלי שווא** ("p", לא "puh"). משך עד 150 מ"ש מתחילת הפרץ.
  **3 טייקים** לכל פריט (כמו 12 §6.10), ובדיקה שמיעתית של ה-QA.
- **תנועות קצרות:** אחרי הגייה בודדת מגיעה הגייה בתוך מילת המפתח (apple, egg, igloo, pot, up).
- **פורמט:** ‏WAV 48kHz / 24-bit מאסטר. נגזרת AAC ‏m4a 96kbps מונו. עוצמה ‎-16 LUFS לדיבור, שיא ‎-1 dBTP,
  רצפת רעש מתחת ל-‎-60 dBFS. שקט של 30 מ"ש לפני ו-50 מ"ש אחרי. חדר שקט, ‏pop filter.
- **שמות קבצים:** `en/ph/ph_s__norm__t1.wav`, ‏`en/ph/ph_s__str__t1.wav`, ‏`en/ph/ph_p__clip__t2.wav`,
  ‏`en/ln/ln_b.wav`, ‏`en/kw/kw_moon.wav`.
- **רישיון:** הסכם כתוב (שימוש משפחתי פרטי, בלי הפצה, הדובר מסכים לשימוש בקולו), ונרשם ביומן הרישוי.
  **פעולה של ההורה** (§19).
- **משך הסשן:** כ-2.5 שעות: פונמות + מילות מפתח + שמות אותיות ≈ 60–75 דק' (12 §6.10) · 16 זוגות מינימליים ≈ 10 דק' · כ-60 מילים
  מומצאות ≈ 15 דק' · 8 מילים "מפורקות" (segmented) ≈ 5 דק' · שירים (§12.4) ≈ 45–60 דק' · הפסקות ≈ 15 דק'.
  **אם אין דובר/ת** (§19 #5): פונמות מקליפי Wikimedia ‏(CC BY-SA, ייחוס במסך "קרדיטים"), מילים מומצאות ב-TTS עם IPA מפורש ובדיקת אוזן
  (מסומנות `tts_fallback:true`, ומוחלפות כשתהיה הקלטה), ושירים מדוברים (לא מושרים) בקול Sunny. אף אחד מהגיבויים לא חוסם את פתיחת האי.

### 12.4 שירים

- **קול:** אותו דובר/ת ילידי/ת או זמר/ת ילידי/ת, באותו סשן או בסשן נוסף. במשחק השיר מוצג כ"Sunny & friends".
  הקול השר לא צריך להיות זהה לקול ה-TTS של Sunny, כי OPOL דורש רק שהדמות לא תעבור לעברית.
- **ליווי:** עיבוד MIDI פשוט שלנו (כלי עץ ויוקללה בגוון המחנה) עם soundfont ברישיון CC0. לכל שיר יש שתי
  גרסאות: מהירות רגילה ו-0.85.
- **בלי צלילי אזעקה ובלי עליות טון שמזכירות סירנה** (00-B8).
- **מצב רכב ושמע בלבד** (gap_audit #11): רשימות השמעה לכל ילד, משירים שכבר נשלטו (X-EXP ≥ 10).

### 12.5 בניית שמע מיזוג

אודיו של "s… a… t → sat" נבנה בקוד: קליפי הפונמות עם רווח של 250 מ"ש ← 400 מ"ש ← המילה ב-TTS. בשלב הלימוד
הראשון (S1–S3) מגיעה גם גרסת **מיזוג רציף**: הנמשכים המתוחים מוצלבים (crossfade) לתוך התנועה ("ssssaaat").
כל פריט מיזוג עובר בדיקת אוזן אנושית ב-30 הפריטים הראשונים, ואחר כך בדגימה של 10%.

### 12.6 כללי TTS

- מילה בודדת מרונדרת **בתוך משפט נשא ונחתכת**, או עם תגי SSML/IPA לפי הספק, כדי למנוע "קטיעה" וטון שאלה.
- הגייה מכוונת ב-IPA לכל מילה בבנק, ונבדקת באוזן. מילים שאולות (banana, pizza) נבדקות במיוחד.
- קצב Sunny לפי §10.6. האטה ב-SSML ‏`prosody rate` או בהארכת זמן (time-stretch) בעיבוד, לפי הספק.
- דובר HVPT: 4 קולות en-US בוגרים מאותו ספק, ומגוון המין והגובה מתועד. אסור קול "ילדותי" (00-E8a).

---

## 13. מדיניות בדיקות הגייה

| # | כלל |
|---|---|
| PP-1 | **אין ציון הפקה לאף ילד.** אין מספרים, כוכבים, "נכון" או "לא נכון" על משהו שהילד **אמר**. |
| PP-2 | **תפיסה כן נבחנת**, כי היא מיומנות האזנה: ‏T09 עם M-PR. אחרי טעות משמיעים את שני המיקומים ומראים כרטיס פה. אין צליל כישלון. |
| PP-3 | **הפקה = השתתפות.** "Your turn!" ← הילד אומר ← (K1 ו-K2) הקלט והשווה: Sunny ואז הקול של הילד, זה לצד זה. ב-K2 ההורה מקיש "אמר!", ב-K1 הילדה מקישה 👍/🤔. הנתון נכנס ל-F-FAM בלבד. |
| PP-4 | **הקלטות לא יוצאות מהמכשיר** ונמחקות בסוף המפגש, אלא אם ההורה שומר "רגע משפחתי". אין העלאה לענן, אין שליחה ל-API ואין אימון מודלים. |
| PP-5 | **ASR לאנגלית: לא ב-v1.** שלב 2 (אופציונלי, K1 בלבד, בנוכחות הורה, 00-E9): זיהוי על המכשיר (Apple SpeechTranscriber en-US) **מוגבל לקבוצת מילים** ("say one of these 3"), ותוצאתו משמשת **רק** כדי ש-Sunny יגיב: אם המילה זוהתה, Sunny מבצע את הפעולה; אם לא, Sunny אומרת "Let's say it together!" ומשמיע את המודל. התוצאה לא נשמרת כנתון שליטה ולא מוצגת. |
| PP-6 | **אין שימוש ב-Azure Pronunciation Assessment או בשירותי ענן דומים על קול של ילד.** |
| PP-7 | **מה כן מאמנים בהפקה:** ‏th (אנימציית "לשון נחש", מראה דרך המצלמה הקדמית **בלי הקלטה**), ‏w ("שפתיים של נשיקה" לעומת "שן עליונה על השפה" ל-v), אורך תנועה (ship מול sheep, עם "מתיחת גומי" בידיים). הכול כמשחק, בלי מדידה. |
| PP-8 | **מה לא מתקנים בכלל:** ר גרונית, ‏h חלשה, ‏ŋg, מבטא כללי. הם לא פוגעים במובנות. ההורים מקבלים הסבר בדשבורד ("מבטא ישראלי הוא לא בעיה"). |
| PP-9 | **אם ילד מגיב בעברית לשאלה באנגלית,** זו הצלחה בהבנה. Sunny עושה recast באנגלית ("Yes! A cat!"), ולעולם לא אומר "Say it in English". |

---

## 14. שיבוץ ובדיקות התקדמות (ספציפי לאנגלית; המנגנון הכללי במפרט ההערכה)

### 14.1 שיבוץ K1 ("המכתב של Sunny", כ-10 דק', שני מפגשים לכל היותר)

| חלק | פריטים | מה קובע |
|---|---|---|
| א. אוצר מילים, "שמע וגע" | 15 (בסגנון Starters, נושאים מעורבים P1) | `lexState` ראשוני לפי נושא. מעל 80% בנושא: מילות P1 בנושא מתחילות ב-G2 |
| ב. צליל ← אות | 12 אותיות מדוגמות (s a t p i n m d g o c k ← ואם יש הצלחה, עוד 8) | UNIT כניסה: ה-UNIT הראשון שבו הדיוק מתחת ל-80% |
| ג. שם ← אות | 6 | `E.AL.NAME.ID` |
| ד. מיזוג | 6 מילים אמיתיות + 4 מומצאות (רק אם ב' לפחות 70%) | דילוג על BLEND ביחידות שעברו |
| ה. זוגות מינימליים | 8 (2 לכל ניגוד: WV, ‏TH, ‏AUO, ‏IEE) | `E.PR.MP.WV` "לא נדרש" אם הוא לפחות 90% (סקר, לא הנחה) |
| ו. שאלון הורה | 4 שאלות: מה נלמד בכיתה מספטמבר? ספר לימוד? האם יש חשיפה לאנגלית בבית או בחוגים? האם הילדה מכירה את שיר ה-ABC? | יישור נושאים (§17) |

הכללים מ-placement_assessment §6: בלי משוב כישלון, עצירה אחרי 3 טעויות רצופות בחלק, וסיום בהצלחה.

### 14.2 שיבוץ K2 (כ-4 דק')

‏T01 עם 10 פריטים (P1, 2 בחירות), ‏T02 עם 3 פקודות, ושאלון הורה ("האם יש אנגלית בגן? אילו שירים?"). המטרה רק
לקבוע `lexState` ראשוני. אין רמה.

### 14.3 K3

דיווח הורה בלבד: אילו מילים או שירים באנגלית הוא/היא מכיר/ה (00 ו-placement §4).

### 14.4 בדיקות כל 4 שבועות (00-C10), ללא זמן וללא תגמול, מוצגות כ"ביקור במגדלור"

| ילד | תוכן | משך |
|---|---|---|
| K1 | 10 GPC (מעורב, כולל b/d) · 5 מילים מומצאות (מקבילה ל-NWF) · 5 מילים מסובכות · 10 מילים מושהות (נלמדו לפני 7 ימים לפחות) · 6 זוגות מינימליים · 1 משפט פענוחי | 6–8 דק' |
| K2 | 10 מילים מושהות (T01) · 3 פקודות · 1 שאלת "שמע סיפור" | 3–4 דק' |
| K3 | אין. ההורה מסמן "מגיב/ה ל…" (5 מילים) | 1 דק' |

**סוף שנה K1 (יוני 2027): "מיני-Starters"** (לא רשמי, תחת מיתוג המשחק): האזנה, 10 פריטים בארבעה סוגים
(חיבור שמות, סימון תמונה, צביעה, שם מאוית); קריאה, 10 פריטים (כן/לא לתמונה, השלמת מילה מתמונה); דיבור, 4
הנחיות עם ההורה (מה זה, איפה, ספרי על עצמך, תארי תמונה). התוצאה מוצגת להורה כ"מגן" מתוך 5, ולילדה כ"מגדלור
שנדלק".

---

## 15. תוכניות שנתיות

> התאריכים לפי לוח תשפ"ז [V], **מקורב**: לאמת מול חוזר משרד החינוך (V5). מודול הלוח קורא את התאריכים
> מ-`calendar/5787.json` (00-B7). **שליטה גוברת על לוח השנה:** UNIT לא נפתח לפני הזמן ולא נכפה בגלל התאריך.
> היום הוא 2 בנובמבר 2026, וחודשי ספטמבר–אוקטובר כבר עברו בבית הספר. השיבוץ (§14) קובע את נקודת הכניסה.

### 15.1 K1: כיתה ג', תשפ"ז

**מבנה בלוק אנגלית של K1 (8–9 דק', 2 בשבוע, בתוך פרק מעורב של 08 §6.8; תקציב §2.1):**
- **בלוק א' (פוניקה):** `HELLO` גרסה קצרה, בית אחד (0:30) ← חימום, 3 פריטי חזרה ברמת הצלחה של כ-95% (1:00) ← צליל חדש: כרטיס
  פה, תנועה, אות גדולה וקטנה (2:00) ← צדפי הצלילים במצב מיזוג, ואמיתי או טיפשי (2:30) ← אורות המגדלור, 3–4 מילים מסובכות (1:00) ←
  `BYE` בית אחד + כרטיס משימה לבית (0:45). סה"כ ≈ 8 דק'.
- **בלוק ב' (שפה ואי):** `HELLO` (0:30) ← אי באנגלית: נמל המילים או שיחה על הרציף, לפי נושא החודש (2–4 דק', §10.4) ← צדף ההד, 6 זוגות
  (1:30) ← יומן הספינה, עמוד–שניים (2:00) ← `BYE` (0:30). סה"כ ≈ 8–9 דק'.
- **חטיף של 4 דק'** (פעם בשבוע, בפרק שלישי, בלי `HELLO`/`BYE` מלאים, רק הג'ינגל): חזרות שהגיע זמנן (6–8 פריטים) ושיר אחד.
- כל בלוק נפתח ונסגר בג'ינגל ובמסגרת האי (AC-MX-3 של 01). **שני הבלוקים לא באותו פרק** ולא באותו יום כברירת מחדל (המתזמן של 08 מפזר).
- הרכב בתוך הבלוקים: כ-60% קצה הלמידה, כ-30% חזרה מרווחת וכ-10% אתגר (learning_science C11). חופשות ומצב קל (§15.1 טבלה): בלוק ב' בלבד.

**תיאום עם עלילת העונה (01 §2.4, §13.2):** ההשקה מתוכננת ל-8.11.2026 (S0). **האי באנגלית נפתח בפרק 6, "ספינה
מחו"ל"**, בערך ב-20.12.2026 (בתוך S1), עם הבוס Mr. Mix-Up. **עד אז יש שלב "בקבוקים מהים"**: פעמיים או שלוש
בשבוע נסחף אל המחנה בקבוק עם הקלטה של Sunny (שיר ו-2–3 מילים, בתוך מסגרת אנגלית עם ג'ינגל וטורקיז), כרגע אנגלי
של 3–5 דק'. בשלב הזה יש קלט בעל-פה לכל הילדים, ו-K1 עושה את השיבוץ "המכתב של Sunny" (שבוע 3). השלב הזה הוא
הצעה ל-01 (§20.2), ואם תידחה, הקלט לפני פתיחת האי יגיע משגרות משפחה בלבד.

| חודש | שבועות לימוד (מקורב) | פוניקה (מסלול בסיס: כניסה ב-`SAT`) | מילים מסובכות | נושאי אוצר מילים | דיבור והאזנה | שירים | PR | אירועים | בדיקה |
|---|---|---|---|---|---|---|---|---|---|
| ספט'–אוק' | — | (בית הספר: היכרות ואותיות; לפי השאלון) | — | — | — | — | — | — | — |
| **נוב'** (מ-8.11) | 3 | **בקבוקים מהים:** אין עדיין UNIT. שיבוץ K1 בשבוע 3. בקבוקי ABC ופזמוני הצלילים s, a, t ו-p כחשיפה | — | FAM, COL, NUM 1–10, ברכות (בעל-פה) | L2 · ‏CH.HELLO/NAME/AGE | HELLO, BYE, ABC, ONE2, IYH, COLORS | בסיס בשיבוץ | — | שיבוץ |
| **דצמ'** | 2 + שבוע חנוכה (כ-6–12.12) | חנוכה: בקבוק CANDLES (1–8, candle, light, present). **כ-20.12: האי נפתח** ← `SAT` | I, a | TOY | CH.WHATS/COLOR | CANDLES, BINGO | — | חנוכה (EV-01), פתיחת האי, Mr. Mix-Up | — |
| **ינו'** | 4 | `INM` ← `GOK`; ‏`DIR.LTR.*`; אותיות גדולות וקטנות של מה שנלמד | the, is, and, to, no, go | FOO; ט"ו בשבט (EV-02, ‏23.1): tree, fruit, flower | CH.LIKE/PLEASE · L3 | HSKT, THIS | `MP.IEH` | ט"ו בשבט | בדיקה 1 |
| **פבר'** | 4 | `EUR` ← `HBF`; מיקוד b/d (`AL.VIS.BDPQ`); משפחות כתיבה c ו-l | you, my, of, he, she, we, me, be, was, his, has | BOD, ANI | CH.HOWMANY/WHERE | OMD, RAIN | `MP.AUO` | **בעברית מתחילה דעיכת הניקוד** (00-B5): בלי ענף אנגלית חדש החודש | בדיקה 2 |
| **מרץ** | 4 | **Boss א'** ("23 מנורות במגדלור") ← `JVW` | they, are, all | CLO; פורים (EV-04, ‏23.3): costume, mask, "I am a…" | CH.HAVE/CAN · L4 | COSTUME, TEN | `MP.WV` | פורים | בדיקה 3 |
| **אפר'** | 1.5 + חופשת פסח (13–28.4) | `YZQ` ← **Boss אלפבית** (26 אותיות: גדולה/קטנה, צליל ושם), לפני החופשה | do, her, what | HOM, TIM (ימי השבוע) | CH.SPELL (מאייתת את שמה) | DAYS, SKIP | `MP.WV` | **פסח (EV-05): מצב קל.** שירים ושגרות בלבד, וחזרה פעמיים בשבוע | בדיקה 4 (אחרי החג) |
| **מאי** | 4 | `DIG` (3–4 שבועות) | there, some, come, one, said | SCH, TRN, PLY; שעה (o'clock) | CH.DOLIKE/TIME · שיחה של 3–4 תורות | HICK | `MP.TH`, ‏`MP.DH` | יום השואה (4.5) ויום הזיכרון (11.5): **ימים שקטים** (QD-1/2), בלי אירוע ובלי בוס. יום העצמאות (12.5): flag, picnic. ל"ג בעומר (25.5): campfire | בדיקה 5 |
| **יוני** | 3 | `CLU` ← **Boss סוף שנה** ("יומן המסע"). ‏`MGE` אתגר, רק אם `CLU` נשלט | like, have, little, were, so | WLD, CH.THEREIS; שבועות (10–11.6): milk, cheese, cake | L5 · S4 | מועדפים | `MP.WV` (west/vest), ‏`MP.IEE` | **מיני-Starters** (13–17.6) | סוף שנה |
| **יולי–אוג'** | חופש | "קייטנת הקיץ של Sunny": 2–3 פעמים בשבוע, 10 דק', חזרה, שירים, ספרונים לפני השינה. `MGE` רק אם הילדה מבקשת | — | — | — | — | — | אין יעד | — |

**מסלול מהיר:** אם השיבוץ מראה שבית הספר כבר לימד את האותיות (סביר בכיתה ג' עד דצמבר), הכניסה היא ב-UNIT
הראשון שבו הדיוק מתחת ל-80%, וכל הטבלה "נדחסת" שמאלה. במסלול כזה `MGE` נכנס עד יוני ו-`VTM` לקיץ או לכיתה ד'.

**מרווח ביטחון:** החופשות (חנוכה ופסח) מתוכננות כשבועות קלים. אם הקצב איטי, `CLU` עובר לקיץ ו-`DIG` הופך ליעד
השנה. השליטה גוברת על לוח השנה.

**יעדי K1 לסוף יוני:** מסלול בסיס: ‏S1–S9 נשלטים (‏S10 אתגר); מסלול מהיר: עד S10 · כ-35 מילים מסובכות (מסלול בסיס) עד 40 (מסלול מהיר) · ‏P1 בקליטה ובהפקה, ו-P2 בקליטה, בכ-70%
מהנושאים · ‏L5 ו-S4 לפחות · ‏`MP.WV`, ‏`TH`, ‏`AUO`, ‏`IEE` לפחות 85% · כותבת את שמה ומשפט פענוחי עם אות גדולה
ונקודה.

### 15.2 K2: תוכנית בעל-פה (גן חובה)

**בלוק K2 (8 דק', פעם בשבוע; תקציב §2.1):** `HELLO` (0.5) ← נמל המילים T01: 3 מילים חדשות ו-3 לחזרה (3) ← "Sunny Says" T02 (1.5) ← שיר
או קריוקי (2) ← משחק PA או צדף הד רכה (0.5, מינואר) ← `BYE` (0.5). **חלופה** (ברירת המחדל של המתזמן כשהפרק קצר מ-10 דק'): **2 "רגעי
אנגלית" של 4 דק'** בשני פרקים שונים: רגע א' = ג'ינגל + נמל המילים (3) + `BYE` קצר; רגע ב' = ג'ינגל + שיר/TPR (3) + `BYE` קצר.
**בלי קריאה. אותיות רק אגב** (D18). השיר היומי מגיע ממצב השמע ברכב (FR-2), לא מזמן המסך.

| חודש | נושא | 15 מילות יעד (מהבנק) | צירופים | שירים | פקודות (§8.3) | PA |
|---|---|---|---|---|---|---|
| נוב' | שלום, אני, צבעים | hello, goodbye, yes, no, red, blue, yellow, green, happy, sad, boy, girl, mom, dad, name | CH.HELLO, CH.NAME, CH.AGE | HELLO, BYE, IYH, COLORS | שלבים 1–2 | — |
| דצמ' | אורות, מספרים, צעצועים | one, two, three, four, five, six, seven, eight, candle, light, present, ball, doll, teddy bear, balloon | CH.HOWMANY, CH.SHABBAT (Happy Hanukkah) | CANDLES, ONE2, TTLS | 2 | — |
| ינו' | הגוף שלי (+ ט"ו בשבט) | head, shoulders, knees, toes, eyes, ears, nose, mouth, hair, hand, leg, foot, tummy, tree, flower | CH.WHATS | HSKT, TEDDY | 3 | חרוז (מתוך השירים) |
| פבר' | חיות | cat, dog, bird, fish, duck, cow, horse, sheep, pig, frog, monkey, elephant, lion, snake, zoo | CH.LIKE | OMD, ITSY | 3–4 | חרוז |
| מרץ | בגדים ותחפושות | hat, shoe, sock, shirt, pants, dress, skirt, jacket, pajamas, costume, mask, robot, monster, big, small | CH.COLOR, "I am a…" | COSTUME, TEN | 4–5 | צליל פותח |
| אפר' | אוכל ובית (חודש קצר) | apple, banana, bread, egg, milk, water, juice, cake, pizza, cookie, ice cream, yummy, hungry, more, cup | CH.PLEASE, CH.THANKS | MULB, RAIN | 5 | צליל פותח |
| מאי | בחוץ ובתנועה | sun, rain, car, bus, train, plane, boat, bike, park, jump, run, swim, fly, stop, flag | CH.WHERE, CH.LETS | SKIP, RRRB | 6 | מיזוג בעל-פה (אתגר) |
| יוני | קיץ וחוף | sea, beach, sand, shell, hot, cold, fun, play, star, moon, family, friend, birthday, party, sleep | CH.GOODMORNING | חזרה על מועדפים, HBD | 8 | — |

**נובמבר עד אמצע דצמבר** מגיעים דרך "בקבוקים מהים" (שירים ומילים בעל-פה), עד שהאי נפתח בערך ב-20.12 (§15.1).
סה"כ 120 מילים. **יעדי K2 לסוף השנה:** כ-120 מילים בקליטה (M-REC), כ-40 "מוכרות בהפקה" (F-FAM), 15 צירופים,
12 שירים עם תנועות (X-EXP ≥ 10), ‏L3 ו-S2–S3. **ילד שבוחר לא לדבר מקבל את אותה התקדמות בהאזנה** (S0 תקין).
לקראת כיתה א' (תשפ"ח) ממשיכים בעל-פה ומוסיפים את `PA.PHON.*` ואת `E.PH.SAT` כאופציה, רק אחרי שהקריאה בעברית
יציבה.

### 15.3 K3: תוכנית שירים (2.5)

**פרק K3 (3–5 דק', בחיק הורה):** `HELLO` (0:45) ← "גלה" T20, ‏3–4 מילים (1–1.5 דק') ← שיר תנועה (1–2 דק') ←
`BYE` (0:30). **כל השורות של ההורה בעברית על מסך ההורה** (למשל: "הצביעו יחד על האף כש-Sunny שר nose", "חזרו על
המילה בקול שמח, בלי לבקש שיגיד/תגיד אותה").

עד פתיחת האי (~20.12) השירים מגיעים ב"בקבוקים מהים" (§15.1), ואחרי זה גם בבועות השיר (MX-E02).

| תקופה | שיר חדש (כל 3 שבועות בערך) | מילים בהדגשה |
|---|---|---|
| נוב' | HELLO + BYE (מסגרת קבועה), IYH | hello, bye-bye, clap, happy |
| סוף נוב'–דצמ' | TTLS, ‏CANDLES (חנוכה) | star, big, candle, one-two-three |
| ינו' | HSKT | head, eyes, nose, mouth, toes |
| פבר' | OMD | cow, duck, dog, cat |
| מרץ | RRRB (חתירה בחיק) | boat, row |
| אפר' | MULB (שגרת אמבטיה ושיניים, בבית בפסח) | bath, wash, hands, teeth |
| מאי | ITSY, PAT (ימי הולדת) | spider, rain, sun, cake |
| יוני | TEDDY, COLORS | teddy bear, red, blue, turn around |

**רשימת ליבה K3 (46 מילים, X-EXP):** hello, goodbye, more, all done, yes, no, mom, dad, baby, grandma, grandpa,
brother, sister (עם תמונות משפחה, D14), ball, car, bus, doll, teddy bear, balloon, cat, dog, duck, cow, fish,
bird, apple, banana, milk, water, cake, head, eyes, nose, mouth, toes, hand, bath, bed, shoe, hat, red, blue,
big, star, jump, clap.
**ציפייה מציאותית:** הבנה והצטרפות בתנועה. מילה שהוא/היא אומר/ת באנגלית היא בונוס ולא יעד [N, Kuhl].

---

## 16. גשר פיזי-דיגיטלי: דפים להדפסה ושגרות משפחה

### 16.1 דפים להדפסה (PDF A4, נוצרים אוטומטית; 00-C10)

לכל דף יש **הוראות להורה בעברית (RTL) בראש הדף**, **תוכן אנגלי LTR מבודד**, שם הילד, הגיבורים שלו, ו-QR
לשמע. התוכן נשאב מהחזרות שהגיע זמנן (המתזמן), כך שהדף תמיד רלוונטי.

| ID | דף | ילדים | מתי | הערה |
|---|---|---|---|---|
| PR-1 | **כרטיסי צליל-אות** (A6, ‏4 בעמוד): גרפמה גדולה וקטנה, תמונת מפתח, אייקון תנועה | K1 (K2 רק כרטיס שם) | בפתיחת כל UNIT | משמשים ל"ציד צלילים" בבית |
| PR-2 | **אריחי בניית מילים** לגזירה: דיגרפים מחוברים, תנועות צבועות | K1 | `SAT`, ‏`HBF`, ‏`DIG`, ‏`MGE` | אותו צבע תנועה כמו באפליקציה |
| PR-3 | **ספרון פענוחי** (8 עמודים מקיפול של A4 אחד) | K1, ומקריאה ל-K2/K3 | 2 לכל UNIT | מצב "מדריכה": פעם ביום לכל היותר (gap_audit #12) |
| PR-4 | **דף כתיבה למשפחת אותיות**: מודל ← 2 עקיבות ← **3 כתיבות חופשיות** ← ציור מילה ← "write your name". נקודת התחלה ירוקה משמאל וחץ LTR | K1 | עם `AL.FORM.*` | כתיבה חופשית ולא רק עקיבה (James & Engelhardt [N]) |
| PR-5 | **מילון התמונות של Sunny** (חוברת מדבקות) | K2 | עמוד לכל נושא חודשי | מדבקה כשהמילה נשמעה או נאמרה בבית (ההורה מסמן) |
| PR-6 | **חפיסת "Sunny Says"** (30 קלפי TPR) | כולם | נובמבר | משחק משפחתי בסוף שבוע |
| PR-7 | **מילת השבוע למקרר**: ‏K1 מילה פענוחית ומילה מסובכת, ‏K2 מילת תמונה | K1, K2 | שבועי | QR לשמע: גם הורה שאינו דובר אנגלית שומע את ההגייה |
| PR-8 | **מדבקות שמות לחפצים בבית** (door, window, table, chair, bed, lamp, mirror…) | K1 כותבת, K2 מדביק | ינואר (HOM) ומרץ | העתקה לפי מודל |
| PR-9 | **בינגו נושאים** | כולם | חודשי | ערב משחקים. אין מנצח יחיד, כולם ממלאים לוח משותף (00-C6) |
| PR-10 | **תעודת Boss** (שבח תהליך: "Noa read 23 lanterns, left to right!") | K1, K2 | אחרי Boss | |
| PR-11 | **קלפי הד** (זוגות תמונות מינימליים) | K1 | עם `PR.MP.*` | ההורה אומר (או מפעיל QR), והילדה מצביעה |
| PR-12 | **"הדרכון האנגלי שלי"**: חותמת לכל Can-Do ("I can say my name") | K1, K2 | מתעדכן | הערכה עצמית עם הורה, ממופה ל-`OR.S.*` |

### 16.2 שגרות משפחה (לכל שגרה יש כרטיס תסריט להורה בעברית)

| ID | שגרה | משך | ילדים | תסריט מקוצר |
|---|---|---|---|---|
| FR-1 | **Good morning English** בארוחת הבוקר | 2 דק' | כולם | "Good morning! How are you?" — "I'm… sleepy!" כל ילד עונה בדרכו |
| FR-2 | **אנגלית ברכב** (שמע בלבד) | 5–15 דק' | כולם | רשימת השמעה לכל ילד + "I spy something red" |
| FR-3 | **שיר גוף באמבטיה** | 2 דק' | K3, K2 | HSKT או MULB תוך כדי רחצה |
| FR-4 | **מילת ארוחת ערב** | 1 דק' | כולם | מילת אוכל אחת. K1 מאייתת, K2 אומר, K3 מצביע/ה |
| FR-5 | **כרטיס משימה של Sunny** (סוף פרק) | 2–5 דק' | כולם | K1: "Find 3 things that start with /s/". K2/K3: "Find something red" |
| FR-6 | **משחק חנות** | 5 דק' | K2, K1 | "Can I have an apple, please?" — "Here you are!" — "Thank you!" |
| FR-7 | **ספרון לפני השינה** | 3–5 דק' | K1 ← K2/K3 | K1 קוראת ספרון PR-3. מצב שינה: בלי תגמולים ובלי טיימר |
| FR-8 | **Sunny Says בסוף שבוע** | 5 דק' | כולם | בכל פעם ילד אחר הוא "Sunny". K3 בזוג עם הורה |
| FR-9 | **"רגע אנגלית" (OPOL-lite)** | גמיש | כולם | כשבובת הנייר של Sunny על השולחן, מדברים אנגלית "כמה שאפשר". הורה לא-ילידי הוא מודל מצוין לנכונות לנסות. **לא מתקנים הגייה**, רק recast |
| FR-10 | שבת וחגים | — | — | האפליקציה בהשהיה (00-B7). שירים מותרים לפי העדפת המשפחה (הגדרת הורה) |

**רישום:** ההורה מקיש "עשינו" באזור ההורה, והזמן נוסף ל"דקות קלט באנגלית" של כל ילד, לפי הערכה קבועה לכל
שגרה. **אין פרס שמוכרז מראש על שגרה** (00-D6). המשפחה כולה מקבלת תרומה למטרה המשפחתית השבועית.

---

## 17. הורה ובית ספר: דשבורד, סנכרון ודגלים

- **כרטיס אנגלית לכל ילד:** דקות קלט השבוע מול היעד (אפליקציה + שגרות) · UNIT נוכחי ומה הבא · מילים חדשות
  השבוע · "שאלו הערב" (למשל: "בקשו מ-{K1} לקרוא לכם ‹Pip got a pig›") · מצב הזוגות המינימליים (בניסוח חיובי).
  **בלי השוואה בין הילדים** (00-C6).
- **סנכרון חודשי עם בית הספר (K1):** 3 שאלות קצרות + צילום אופציונלי של עמוד במחברת או בספר. החילוץ ב-vision
  דורש אישור הורה (00-E8; הצינור של 12 §3, פלט כרשימת מילים/אותיות בצ'יפים שההורה מאשר). מה מתעדכן: (1) אותיות שנלמדו בכיתה מסומנות `schoolTaught`. הסימון מעלה עדיפות ולא
  נחשב שליטה (08 §6.9 `teaching` +0.5). (2) מילים מספר הלימוד נכנסות ל-G2 ומקבלות עדיפות בנושא. (3) **אם בית הספר מלמד שמות אותיות לפני
  צלילים**, האפליקציה ממשיכה "צליל קודם" (00-B6), ומחברת לשם בתוך אותו פריט ("This is **bee**. It says /b/").
  (4) **ספר לימוד שונה / לא מוכר:** אין רשימה מובנית לספרי לימוד. התוכנית מתיישרת לפי **הנושאים** (§15.1) ולא לפי הספר; מילים שהוזנו
  מהצילום מקבלות `src:'school'` ו-`alt` לצורה הבריטית אם זוהתה (§7.1). ספר שלא צולם = ברירת המחדל של §15.1, בלי שאלות נוספות להורה.
  (5) **"כבר מכירה את כל האותיות" (שיבוץ ב' ≥80% על כל 20 הדגימות):** מדלגים על `GPC` של SAT–YZQ (מסומנים `known`, ‏08 §2.4, בדיקה מושהית
  ב-rung 1) ונכנסים ישר ל-`BLEND` של ה-UNIT הראשון שבו המיזוג מתחת ל-80% (שיבוץ ד'). אם גם המיזוג ≥80% עד DIG, הכניסה היא `DIG`
  והטבלה נדחסת (מסלול מהיר). שמות אותיות (`NAME`) נבדקים בנפרד (שיבוץ ג') ולא מניחים אותם מהצלילים.
- **מצב רגוע / מצב ביטחוני (00-B8; ההגדרה המשפחתית `calmMode` של 10, ⟂VERIFY שם השדה):** באנגלית אין תוכן רגיש, ולכן האי ממשיך לפעול,
  עם שלושה שינויים: אין בוס (Mr. Mix-Up לא מופיע), חגיגות ברמת T0–T1 בלבד (01 §12), ופקודות TPR "רועשות" (stamp, shout, bang) לא
  נבחרות. שירים נשארים (הם המנגנון המרגיע ביותר שיש לנו). ימים שקטים (QD-1/2 של 01): אין בלוק אנגלית, רק שיר אחד אם ההורה בחר.
- **K2 שומע שירים באנגלית בגן (§19 #2):** שיר שההורה סימן "מכיר" מתחיל ב-`X-EXP = 5` (חצי מהיעד) ונכנס ראשון לקריוקי `FILL`; מילות
  היעד שלו מתחילות ב-`lexState = seen`. שיר שהגן שר **בגרסה אחרת** (מילים שונות): הגרסה שלנו לא "מתקנת" את הגן; ניצי אומר פעם אחת
  "בגן שרים את זה קצת אחרת, וזה בסדר", והשיר מסומן `altVersionKnown`.
- **הורה שלא דובר אנגלית:** כל כרטיס הורה וכל דף מודפס עם אנגלית נושא QR לשמע (PR-7); שורות ההורה בעברית בלבד; אין שגרה שדורשת
  מההורה לקרוא אנגלית בקול (FR-1..9 מנוסחות כך שההורה יכול להשמיע את Sunny במקום לומר בעצמו).
- **התראות להורה (לא אבחנה, 00-C8):**
  - **K1:** אחרי 12 שבועות לפחות של פוניקה, בשלושה פרקים או יותר בשבוע: דיוק צליל-אות מתחת ל-50% על אותיות
    שנלמדו, **או** מיזוג מתחת ל-40% במשך 4 שבועות למרות GPC של 80% ומעלה, **או** סריקה RTL עקבית (ERR.DIR) אחרי
    8 שבועות. אם יש דגל, ההודעה מציעה שיחה עם מורת האנגלית ועם היועצת, ומצליבה עם אותות הקריאה בעברית (ידע
    אותיות הוא מנבא משותף [N]). מסלולי הפנייה לפי 00-C8.
  - **K2:** אין דגלי אנגלית, כי מדובר בחשיפה בעל-פה. חריג: אם אין תגובה לשמע בשום שפה, הדגל הוא בדיקת שמיעה
    אצל רופא/ת ילדים.
  - **K3:** אין.
- **מבחן "האם זה עובד" (gap_audit #10):** בדיקות כל 4 שבועות (§14.4) · שימור מושהה · בחירה חופשית של אי
  אנגלית בלי תגמול · אות העברה חודשי ("קראה שלט באנגלית ברחוב", "שרה שיר באנגלית בלי האפליקציה"), שההורה מסמן.

---

## 18. קריטריוני קבלה

| # | קריטריון | איך בודקים |
|---|---|---|
| AC1 | לכל צומת `E.*` יש `id` יציב, ‏`prereqs`, ‏`mastery`, לפחות תבנית אחת וקוד שגיאה אחד, והגרף א-ציקלי | בדיקת CI על `skills.json` |
| AC2 | כל פריט `decodable: true` עובר את בודק הפענוחיות ב-100% (גרפמות ומילים מסובכות עד ה-UNIT הנוכחי), כולל טקסטי LLM | CI; build נכשל על שגיאה |
| AC3 | בנק המילים מכיל לפחות 400 רשומות (v1: 534) עם גלוס עברי מנוקד, נושא, P, ‏minBand, צורה אמריקאית ו-`alt`; ניקוד נבדק בדגימה של 10% | סקריפט + דגימה אנושית |
| AC4 | **אין תו לטיני בקלט ה-TTS העברי.** כל מילה אנגלית בשורה עברית היא קליפ Sunny משובץ | lint על כל מחרוזות ה-TTS |
| AC5 | 45 קבצי פונמה × 2 סגנונות, ‏26 שמות אותיות ו-45 מילות מפתח הוקלטו, עברו QA (עיצורים סותמים קטועים, עד 150 מ"ש, בלי שווא; עוצמה ‎-16 LUFS ±1), והרישיון מתועד | סקריפט מדידה + האזנה |
| AC6 | מנוע הסנדוויץ' שומר `glossLevel` לכל ילד ולכל מילה ומעביר רמות לפי §10.3 | בדיקות יחידה במנוע + snapshot של תרחישים |
| AC7 | **אין רכיב UI שמציג ציון הגייה.** הקלטות לא נשלחות מהמכשיר | grep לרכיבים אסורים + בדיקת רשת ב-Playwright (אין בקשת אודיו יוצאת) |
| AC8 | אי באנגלית מרונדר LTR בתוך מעטפת RTL. אין צומת טקסט מעורב בפריטי קריאה של K1/K2. Sunny משמאל וניצי מימין | צילומי מסך ב-Playwright לכל פרופיל (iPad ו-iPhone) |
| AC9 | לכל שיר יש שכבת רישוי, מקור, מילים, רשימת תנועות ושמע. אין שיר `TRAD-verify` בלי דגל `verified:true` | בדיקת CI על `songs.json` |
| AC10 | כל זוג מינימלי זמין בארבעה דוברים לפחות. ב-M-PR נספרים 3 דוברים לפחות | בדיקת מניפסט שמע |
| AC11 | משכי הפרקים תואמים ל-00-C3 (K3 3–5, ‏K2 8–12, ‏K1 15–20 דק'). זמן חגיגה ומעבר עד 15% | טלמטריה בבדיקת קבלה |
| AC12 | לוח תשפ"ז נטען מקובץ. אין אירוע חגיגי ביום הזיכרון. השהיה בשבת ובחג | בדיקת יחידה של מודול הלוח |
| AC13 | בדיקה כל 4 שבועות נוצרת לכל ילד פעיל ומוצגת בדשבורד | בדיקת אינטגרציה |
| AC14 | lint על שורות Sunny: אורך לפי גיל, מילון מותר (בנק + i+1), רשימת ביטויים אסורים (§10.6) | CI על `sunny_lines.json` |
| AC15 | כל דף PR-1 עד PR-12 נוצר כ-PDF תקין: עברית RTL ואנגלית LTR, בלי חיתוך ניקוד, גופן Andika משובץ | בדיקת רינדור PDF + צילום |
| AC16 | פזמוני הצלילים ותנועותיהם מקוריים. אין התאמה לרשימת התנועות הידועה של Jolly | בדיקת QA ידנית, מתועדת |
| AC17 | סדר הצלילים, המילים הפענוחיות והמילים המסובכות ב-`phonics.json` תואם ל-§5.2–§5.3; מספר המילים המסובכות עד S10 = 40 | בדיקת snapshot |
| AC18 | **תקציב זמן:** בסימולציית מתזמן של 1,000 שבועות (08 §12), דקות האנגלית ל-K1 הן 15–22 בשבוע ול-K2 6–10, לפי `subjectWeights` בתוקף; אף פרק K2 לא מכיל יותר מבלוק/רגע אנגלית אחד; שני בלוקי K1 לא באותו פרק | בדיקת סימולציה |
| AC19 | **כיוון:** בתוך `CMP-22` אין רכיב עם `direction:'rtl'`; סטודיו הקוד (06) לא מופיע באותו פרק עם בלוק אנגלית; כל אנימציית הדגשה/מילוי באי מתחילה ב-x המינימלי | בדיקת יחידה על המתזמן + Playwright |
| AC20 | **מגדר (00-B3):** כל שורה של ניצי באי (גלוס, הוראות SR-3, SR-5, SR-10) קיימת ב-`m`/`f`/`pl`; שורות לרצועת T ניטרליות עד אישור מגדר K3; שורות Sunny ללא וריאנטים (אנגלית, `{name}` בלבד) | לינטר `nitzi_lines.json` (כמו 07 AC-02/03) |
| AC21 | **רצועות:** כל `bands` ב-`skills.json`, ‏`vocab.json` (`minBand`) ו-`templates/*.json` ∈ {T,G,S}; אין K1/K2/K3 בקבצי תוכן | CI (סכמה) |
| AC22 | **בטיחות דפים ושגרות:** לכל PR-* ו-FR-* יש תגי בטיחות לפי 06 §16.1 (PR-2 אריחים לגזירה = `choking-hazard-T` + `adult-required`; FR-3 אמבטיה = `adult-required`; FR-5/FR-6 חפצים קטנים = `choking-hazard-T`); כשיש ילד/ה מתחת ל-36 חודשים במשפחה, דף/שגרה עם `choking-hazard-T` מוצגים להורה עם אישור חובה ולא מוצעים ל-K2 לבד | CI על `printables.json` + בדיקת יחידה |
| AC23 | **מצב רגוע:** כש-`calmMode=true` אין בוס באי, חגיגות ≤T1, ואין פקודות TPR מהרשימה "רועשות"; בימי QD-1/2 המתזמן לא משבץ בלוק אנגלית | בדיקת יחידה על המתזמן + snapshot |
| AC24 | **Mr. Mix-Up:** כל שורת `MIXUP` מתויגת `lang:'en'` ומושמעת רק כש-`englishContext=true`; אין שורת CH-04 בעברית בתוך האי ואין שורת `MIXUP` באנגלית מחוץ לו | לינטר על `lines.json` + בדיקת רכיב |
| AC25 | **אין קלט טקסט חופשי** באף תבנית EN.T01–T20 (קלט: הקשה, גרירה, בועות, הקלטה מקומית בלבד); אף בקשת רשת לא יוצאת מהאי בזמן פרק (תוכן ושמע מראש) | בדיקת UI + בדיקת רשת ב-Playwright |

---

## 19. נושאים פתוחים להורה (מינימלי, באצווה אחת; לא חוסם, 00-F2)

| # | שאלה | למה | ברירת מחדל עד שתהיה תשובה |
|---|---|---|---|
| 1 | **K1:** מה נלמד באנגלית בכיתה מספטמבר? מה שם ספר הלימוד? (אפשר לצלם עמוד במחברת) | שיבוץ ויישור נושאים (EN-11) | השיבוץ (§14) קובע לבד |
| 2 | **K2:** יש אנגלית בגן (ENJOY ENGLISH או חוג)? אילו שירים הוא מכיר? | שירים מוכרים מתחילים ב-G2 | התוכנית כמו שהיא |
| 3 | **איות השמות באנגלית** של שלושת הילדים (ושל בני משפחה למשחקי תמונות) | `WR.NAME`, מילים מסובכות אישיות, השורה "Ahoy, {name}!" | placeholder `{K1}/{K2}/{K3}` (כמו Q-01 במפרט 13) |
| 4 | **דובר/ת אנגלית ילידי/ת לסשן הקלטה** של כ-2.5 שעות (פונמות, שמות אותיות, מילים מומצאות, שירים): מכר, או שירות בתשלום | פעולה חיצונית או תשלום (00-F2) | גיבוי לפי §12.3: קליפי Wikimedia ‏(CC BY-SA) לפונמות; מילים מומצאות ב-TTS מסומנות; שירים ב-TTS מדובר (לא מושר) עד ההקלטה |
| 5 | (אופציונלי) בחירה של 2–3 שגרות משפחה מ-§16.2 שמתאימות לכם | התאמה לשגרת הבית | FR-1, ‏FR-2, ‏FR-5 |
| 6 | (אופציונלי) האם יש בבית מבוגר שמעדיף אנגלית בריטית? | בחירת `alt` לקליטה (§7.1) | אמריקאית בלבד |

---

## 20. הצעות לשינוי ב-00 (Proposed change to 00)

> לפי ההוראה ב-00, ההחלטות הקיימות לא שונו. אלה הצעות עם נימוק.

### 20.1 הצעות ל-00

1. **E9: הרחבת סשן ההקלטה האנושי.** כרגע 00 קובע הקלטה אנושית של כ-44 צלילים. **הצעה:** אותו סשן יכלול גם
   26 שמות אותיות, כ-60 מילים מומצאות ואת קולות השירים (כ-27 שירים + פזמוני צלילים). **נימוק:** (א) TTS נוטה
   "לתקן" מילים מומצאות למילים אמיתיות, והמבחן של NWF תלוי בהן [N]. (ב) שירה ב-TTS חלשה, ושירת AI דרך
   ElevenLabs Music חסומה לפי 00-E8a בלי אישור. (ג) שירים + TPR הם המנגנון עם הראיה הטובה ביותר ל-K2/K3 [N].
   התוספת היא כשעה בסשן.
2. **B2: שלוש הבהרות.** (א) **K3:** אנגלית = שירים ומילים בודדות במשחק משותף. הסנדוויץ' בגלוס G3 קבוע, בלי
   דעיכה. (ב) **ה-TTS העברי לעולם לא מבטא אנגלית.** מילה אנגלית בתוך שורה עברית היא קליפ של הדמות האנגלית
   (מונע הגייה עברית שגויה ומחזק OPOL). (ג) **מוסכמה מרחבית:** הדמות האנגלית משמאל וניצי מימין.
3. **שורה חדשה ב-B או C: "K2: אנגלית בעל-פה בלבד; אותיות אנגליות רק אגב."** ההחלטה כבר קיימת ב-gap_audit
   (D18) אבל חסרה ב-00. **נימוק:** K2 לומד אותיות עבריות השנה, ופענוח באנגלית "כנראה מוקדם מדי" [N]. הכנסת
   השורה ל-00 מונעת מצב שבו מפרטים אחרים (דשבורד, התאמה אישית) מחזירים "אותיות אנגליות" למפת השליטה של K2.
4. **E9 (ASR): ניסוח מחייב "אין ציון הפקה לאף ילד".** 00 כבר אוסר ASR עונשי בעברית ומגביל את האנגלית ל-K1
   בשלב מאוחר. **הצעה:** להוסיף שגם בשלב 2 ה-ASR האנגלי משמש רק לתגובה של הדמות (PP-5), לא נשמר כנתון שליטה,
   ואין שליחת קול ילד לשירותי Pronunciation Assessment בענן (PP-6). **נימוק:** WER של ילדים 15–35%, ודיבור
   במבטא לא-ילידי עוד פחות מדויק [N]. ציון שגוי פוגע במוטיבציה וב-C5.

### 20.2 תיאום עם מפרטים מקבילים (לא שינויים ב-00; לבדיקת בעלי 01, 08 ו-09)

| מפרט | נקודה | מה המסמך הזה עושה | בקשה |
|---|---|---|---|
| 01 (עולם) | Sunny (CH-02), ניצי כמתרגם, ‏R-08 ‏Sunny Harbor, ‏MX-E01..E04, ‏MV-08, ‏Wordlings, ‏Mr. Mix-Up | אומץ במלואו (§4, §10) | — |
| 01 | **זוגות מינימליים** | מצב חדש ב-MX-E01 צדף ההד (T09) | להוסיף ל-MX-E01 את המצב `pairs` |
| 01 | מילים מסובכות, סיפור פענוחי, דיאלוג | MX-E05 אורות המגדלור, ‏MX-E06 יומן הספינה, ‏MX-E07 שיחה על הרציף | לאשר שלוש מכניקות חדשות, או למפות אותן למכניקות קיימות |
| 01 | **Pip הסרטן** (לא מדבר) | דמות פענוחית לסיפורי K1 | לאשר ולהוסיף לליהוק כדמות משנה של Sunny |
| 01 | פתיחת האי בפרק 6 (~20.12) | שלב "בקבוקים מהים" מההשקה ועד הפתיחה (§15.1) | לאשר. **נימוק:** כמות קלט היא המנבא העיקרי [N, Muñoz], ו-6 שבועות בלי אנגלית הם כ-15% משנת הלימודים |
| 08 (מנוע) | רמות שליטה | M-STD ממופה ל-`proficient` ול-`master` (§3.3) | — |
| 08 | משקלי מקצועות (K1 אנגלית 15%, ‏K2 10%) | יעד באפליקציה של 35–40 דק' בשבוע ל-K1 (כ-25–30% מזמן המסך הצפוי) ו-15–20 ל-K2 (כ-15%). את שאר הקלט מספקים שמע ושגרות מחוץ למסך | להעלות את ברירת המחדל ב-`EngineConfig` ל-**K1: 25%, ‏K2: 15%**, או לספור שירי TPR באנגלית גם במשבצת "מוזיקה/תנועה" של K2 |
| 08 | מזהי מיומנות בשיבוץ (`english.oral_vocab`, ‏`english.letter_sounds`, ‏`english.decoding_cvc`) | מזהים מלאים `E.*` (§3) | למפות: ‏`english.oral_vocab` = ‏`E.VO.*.REC`; ‏`english.letter_sounds` = ‏`E.PH.*.GPC`; ‏`english.decoding_cvc` = ‏`E.PH.*.BLEND` |
| 09 (UX) | `font.en` ‏Andika, ‏`type.en.*` | אומץ | טוקנים נוספים: `type.en.k1.decode` (36pt לפחות) ו-`type.en.letter` (64pt לפחות) |

