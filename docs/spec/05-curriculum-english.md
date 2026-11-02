# 05 — תוכנית לימודים וגרף מיומנויות: אנגלית (EFL)

> **סטטוס:** טיוטה ליישום, v1, 2026-11-02. כפוף ל-`00-foundation-decisions.md`. אם יש סתירה, 00 גובר.
> הצעות לשינוי ב-00 מופיעות בסוף המסמך (§20).
> **מקורות:** דוח v1 (פרק האנגלית); `english_efl.md`, `learning_science_workbooks.md`,
> `hebrew_first_language_voice.md`, `voice_tech_he_en.md`, `placement_assessment.md`, `gap_audit.md`
> (בעיקר D3, D4, D9, D10, D18), `content_world_phygital.md`; יומן המחקר `docs/research-log/english.md`.
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
4. **שפה אחת לכל דמות (OPOL).** הדמות **Pip** (שם זמני) מדברת רק אנגלית, נכנסת מצד שמאל ומסומנת בצבע ובג'ינגל
   משלה. מספר עברי מתרגם, נכנס מצד ימין, ותמיד בתבנית "סנדוויץ'" (EN → HE → EN). תמיכת העברית דועכת **לפי מילה**,
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
| EN-9 | **שילוב פנימי (00-C1)** | פענוח הוא החישול: מילה שפוענחה הופכת לחפץ במחנה. האזנה היא המסחר: Pip מבקש ואת/ה מביא/ה. | [N] Habgood & Ainsworth |
| EN-10 | **בלי עונש** | אין "לא נכון!" ואין צליל כישלון. Pip חוזר על הצורה הנכונה (recast). מגן התסכול של D10 חל על כל פעילות. | 00-C2/C4/C5 |
| EN-11 | **יישור עם בית הספר בלי לרוץ קדימה בעיוורון** | קצב K1 נקבע לפי שליטה. ההורה מזין פעם בחודש מה נלמד בכיתה (§17), והנושאים מתיישרים לפי זה. | [N] gap_audit #8 |
| EN-12 | **מה שלא נמדד בנייר לא נחשב** | בדיקות (probes) קצרות כל 4 שבועות, ומבדק בסגנון Starters בסוף השנה. | 00-C10 |

---

## 2. שלושה ילדים, שלושה מסלולים: יעדים לסוף השנה (יוני 2027)

| | **K1** (8.5, כיתה ג', Pre-A1) | **K2** (5.5, גן חובה) | **K3** (2.5) |
|---|---|---|---|
| מטרה | קוראת ומאייתת מילים פענוחיות (עד e שקטה), מזהה 25 מילים מסובכות, מבינה ומשתתפת בשיחות Pre-A1 | מבין הוראות ושירים, מצביע על כ-120 מילים, אומר כ-40 מילים ו-15 צירופים | מכיר 8–10 שירים עם תנועות, מגיב לכ-30 מילים |
| ענפים פעילים | כולם (§3) | OR, VO, PA בעל-פה, SG, PR (האזנה פשוטה בלבד) | SG, VO (חשיפה), OR.L בסיסי |
| קרוא וכתוב | פונטיקה S1–S10 ליבה, S11 אתגר; אות גדולה/קטנה; שם באנגלית; משפט עם נקודה | **אין.** אותיות מופיעות רק אגב (שיר ה-ABC, כרטיס השם של Pip) (D18) | אין |
| דקות קלט בשבוע (יעד) | 60–80 באפליקציה + 20–30 בשגרות משפחה | 35–45 באפליקציה + 20–30 שירים ושגרות | 10–15 במשחק משותף + שירים בבית, בלי הגבלה |
| מבנה שבועי | 3 פרקי אנגלית של 15–20 דק' + 2 "חטיפים" של 4–5 דק' בתוך פרקים אחרים | 4 פרקים של 8–10 דק' + שיר יומי (מצב שמע ברכב) | 3–4 פרקים של 3–5 דק' בחיק הורה |
| יעד הצלחה (00-C2) | 75–85% + Stretch Zone | ≈85% | ≈90–95%, בלי כישלון |
| עברית | הסברים בעברית, איים באנגלית בלבד שמתארכים מ-60 שנ' ל-4–5 דק' | סנדוויץ' לכל מילה חדשה, הוראות בעברית, פקודות אנגליות נלמדות | המספר העברי או ההורה ממסגרים, השיר באנגלית |
| ASR | לא ב-v1. שלב 2: אופציונלי, בנוכחות הורה, מגביל בלבד (§13) | לא | לא |

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
  "bands": ["K1"],                       // K1 | K2 | K3
  "kind": "mastery",                     // mastery | familiarity | exposure
  "prereqs": [
    { "id": "E.PH.SAT.GPC", "min": "mastered" },
    { "id": "E.PA.PHON.BLEND", "min": "mastered" },
    { "id": "E.DIR.LTR.TRACK", "min": "practicing", "soft": true }   // soft = מומלץ, לא חוסם
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
| **M-STD** | K1 (וגם K2 בצמתי קליטה) | ההגדרה האחידה של gap_audit D9: לפחות 85% בשמונה הניסיונות הראשונים האחרונים, שלושת האחרונים בלי רמז, על פני יומיים לפחות, **וגם** בדיקה מושהית נכונה אחרי 3 ימים לפחות. סולם החזרה: 1, 3, 7, 14, 30, 60 יום. (אם מפרט המנוע יקבע אחרת, הוא גובר.) |
| **M-REC** | K2: צמתי `VO.*.REC` ו-`OR.L.*` | M-STD עם 6 ניסיונות במקום 8 ויעד של 80%. גם שימוש ב"עזרה בעברית" נחשב רמז. |
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
| `E.PA.PHON.BLEND` | מיזוג בעל-פה (Pip אומר "c-a-t", את/ה בוחר/ת תמונה) | K2 (אתגר), K1 | `E.PA.ONSET.ID` | M-REC / M-STD | קדם הכרחי ל-`E.PH.*.BLEND` |
| `E.PA.PHON.SEG` | פירוק לצלילים (סופרים צלילים בקפיצות צפרדע) | K1 | `E.PA.PHON.BLEND` | M-STD | "sun": שלוש קפיצות |

#### 3.4.4 PH: פוניקה (K1). הצמתים נגזרים מהרצף ב-§5

לכל UNIT ‏`U` ברצף נוצרים הצמתים הבאים אוטומטית:

| תבנית ID | מיומנות | קדם | שליטה | מכניקה |
|---|---|---|---|---|
| `E.PH.U.GPC` | צליל ← גרפמה (שומעת /m/ ובוחרת m) **וגם** גרפמה ← צליל (רואה m ובוחרת תמונה שמתחילה ב-/m/) | `E.PH.(U-1).GPC` נשלט, ו-`E.PH.(U-1).BLEND` לפחות 70% | M-STD **לכל גרפמה בנפרד** | פנסי צלילים (T03/T04) |
| `E.PH.U.BLEND` | פענוח מילים אמיתיות ומילים מומצאות של ה-UNIT ושל מה שלפניו | `E.PH.U.GPC`, `E.PA.PHON.BLEND` | M-STD (לפחות 20% מהפריטים מילים מומצאות) | מחשלת המילים (T06), אמיתי או טיפשי (T08) |
| `E.PH.U.SEG` | איות מהכתבה במילים של ה-UNIT | `E.PH.U.BLEND` 70% | M-STD | ערבוב הצללונים (T07) |
| `E.PH.U.READ` | קריאת משפט פענוחי ובחירת תמונה מתאימה | `E.PH.U.BLEND`, `E.TW.U.READ` | M-STD | מגילות סיפור (T11) |
| `E.TW.U.READ` | זיהוי המילים המסובכות של ה-UNIT | `E.PH.U.GPC` | M-STD | גחליליות (T10) |
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
| `E.PR.MP.IEE` | /ɪ/–/iː/ | ship/sheep, sit/seat, chip/cheap | K1 | `E.PH.MGE.BLEND` (אוזן קודמת לקריאה: הצומת נפתח בהאזנה כבר אחרי `INM`) | M-PR |
| `E.PR.MP.UU` | /ʊ/–/uː/ | full/fool, look/Luke, pull/pool | K1 (אתגר) | `E.PR.MP.IEE` | M-PR |
| `E.PR.MP.LS` | תנועה קצרה–ארוכה (e שקטה) | cap/cape, kit/kite, hop/hope, cub/cube | K1 | `E.PH.MGE.GPC` | M-PR |
| `E.PR.STRESS.TAP` | הטעמה: איפה החלק "החזק"? | BA-by, ba-NA-na, TI-ger | K1 (אתגר) | — | M-STD |
| `E.PR.ECHO.SAY` | חוזר/ת אחרי Pip, הקלט והשווה | כל מילה | K1, K2 | — | **F-FAM בלבד, אף פעם לא ציון** |

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
| `E.RD.TEXT.COMP` | סיפור פענוחי של 3–6 עמודים ושאלת הבנה | `E.PH.GOK.READ` | M-STD | מגילת סיפור, ואחריה "מה Pip מצא?" |
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
| `ERR.PH.LETTERNAME` | מיזוג בשמות אותיות ("see-ay-tee") | — | (ASR לא קיים) הורה מסמן במצב "שמעתי"; T06: בחירת "סיי" ב-T04 | מסיח: מילה שמתחילה בשם האות (C → "see" ← sea) | "האות עושה צליל": Pip אומר את הצליל, ושם האות נשמר לשיר |
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
| `ERR.GR.COPULA` | "She happy" (בלי is/am/are) | ✓ (אין אוגד בהווה) | T12 (בחירת בועה) | בועה בלי am/is | Pip חוזר על המשפט בצורה הנכונה, ותבנית קבועה: I'm / It's |
| `ERR.GR.ARTICLE` | "I have dog" (בלי a) | ✓ | T12/T16 | בועה בלי a | recast: "A dog! I have a dog." |
| `ERR.GR.ADJ_ORDER` | "a ball red" | ✓ (שם עצם ואחריו תואר) | T16 | אריח בסדר עברי | שיר צבעים: "a red ball" |
| `ERR.GR.LIGHT` | "Open/close the light" | ✓ (פתח את האור) | T12 | — | צירוף קבוע: "Turn on/off the light" |
| `ERR.GR.AGE` | "I have eight years" | ✓ | T12 | בועה מסיחה | צירוף: "I'm eight." |
| `ERR.GR.IT` | he/she לחפצים | ✓ | T12 | — | "It's a…" |
| `ERR.GR.PLURAL` | "two cat" | חלקי | T16/T12 | — | שיר ספירה עם -s מודגש |
| `ERR.VO.SEMANTIC` | בחירת שכן סמנטי (cat במקום dog) | — | T01 | מסיחים מאותו נושא ברמה 2 | ידע חלקי: מתועד, לא "שגיאה" בדוח להורה |
| `ERR.VO.LOAN_STRESS` | מילה שאולה בהגייה עברית (baNAna, KOMputer) | ✓ | (הפקה, לא נמדד) | — | הקלט והשווה, ו-"Pip says it like this" |
| `SIG.GLOSS_DEPEND` | שימוש ב"עזרה בעברית" ביותר מ-40% מהתורות | — | תדירות לחיצה | — | **אות למנוע**, לא שגיאה: איים קצרים יותר, חזרה על מילים |
| `SIG.RAPID_TAP` | הקשות מהירות ואקראיות | — | 3 תשובות בפחות מ-0.6 שנ' | — | מגן עייפות (learning_science F19) |

**חוק:** קודי `ERR.PH.R`, ‏`ERR.PH.H_DROP` ו-`ERR.PH.NG_G` (ר גרונית, h חלשה, "sing-g") **לא נאכפים בכלל**,
כי הם לא פוגעים בהבנה. הם מופיעים רק כהערה לכותבי התוכן: אל תבנו פריט סביבם.

---

## 4. איך האנגלית חיה בעולם (שילוב פנימי, 00-C1)

אזור האנגלית במחנה נקרא **מזח הבלון של Pip** (Pip's Balloon Dock): המקום שבו בלון האוויר החם של Pip נוחת.
הכניסה מלווה בג'ינגל קבוע של 1.2 שנ', בגוון האנגלית ובמעבר עמוד LTR (§10.4). כל מכניקה כאן היא הלמידה
עצמה, ואף אחת מהן לא מתפקדת כשער חידון.

| מכניקה (שם במשחק) | פעולת משחק = פעולת למידה | צמתים | תבנית | ילדים |
|---|---|---|---|---|
| **פנסי צלילים** (Sound Lanterns) | צללונים כיבו את הפנסים. שומעים צליל ומקישים על הפנס עם הגרפמה, והפנס נדלק. | `PH.*.GPC`, `AL.NAME.ID` | T03, T04 | K1 |
| **מחשלת המילים** (Word Forge) | רונות־אותיות מונחות על הסדן משמאל לימין. הפטיש מכה בכל רונה (צליל), החלקה ימינה ממזגת, **והחפץ נוצר**: ‏net תופסת צללון, box מאחסנת, hat הופך לפריט קוסמטי. מילה מומצאת הופכת ל"יצור טיפשי" שנכנס לאלבום. | `PH.*.BLEND`, `PA.PHON.BLEND` | T06, T08 | K1 |
| **ערבוב הצללונים** (Mumble Scramble) | צללון ערבב אותיות במילה. שומעים את המילה ומסדרים את האותיות במשבצות משמאל לימין, והצללון חוזר להיות חבר. | `PH.*.SEG`, `WR.DICT.SPELL` | T07 | K1 |
| **מערת ההד** (Echo Cave) | שתי דלתות עם שתי תמונות. ההד אומר מילה, וכשבוחרים נכון נפתח שביל. דובר אחר בכל פעם. | `PR.MP.*` | T09 | K1 (K2 גרסה של 2 דלתות בלי ציון) |
| **עמדת המסחר של Pip** (Trading Post) | Pip מבקש פריטים באנגלית ("I need a red apple!"). מוצאים ומגישים, ומקבלים חומרי בנייה. | `VO.*.REC`, `OR.L.*` | T01 | K2, K1 (K3 גרסת "גלה") |
| **אימון "Pip Says"** | פקודות תנועה. הילד זז, וההורה או הילד מאשר. ב-K1 יש מצב "Pip Says" עם מלכודת (בלי "Pip says" לא זזים). | `OR.L.CMD`, `VO.VRB` | T02 | כולם |
| **גחליליות** (Tricky Fireflies) | גחליליות עם מילים מסובכות. תופסים את המילה ש-Pip אומר. | `TW.*.READ` | T10 | K1 |
| **מגילות סיפור** (Story Scrolls) | סיפור פענוחי מאיר את השביל ביער. מצב "קראי לאח" (מדריכה, 00-C6) | `RD.*`, `PH.*.READ` | T11 | K1 (K2 מאזין) |
| **מדורת קריוקי** | שירה ותנועה סביב המדורה, וקריוקי השלמת מילה | `SG.*` | T13 | כולם |
| **שיחה עם Pip** (Pip Chat) | תורות דיאלוג כתובות מראש. Pip שואל, והילד בוחר בועה או אומר (הקלט והשווה) | `OR.S.*` | T12 | K2, K1 |

**חומרים במשחק המשפחתי (המדורה):** כל פעולת אנגלית מוסיפה לדלק המדורה המשותפת, בדיוק כמו מתמטיקה ועברית.
ב-K3 התפקיד הוא "שומר/ת האש": הקשה על חיה או צבע שהשיר מזכיר מוסיפה ניצוץ, ואין כישלון.

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
| S10 | `MGE` | a_e, i_e, o_e, u_e, e_e | /eɪ/ /aɪ/ /oʊ/ /juː/ /iː/ | two, who, does, love, your | 3 | **Boss סוף שנה: "מגילת המחנה"** |
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
  mad, dad, sad, pad, tins, pins, maps, mats, naps, dips; ✱ Pip, Sam, Tim, Pam, Nan, Dan, Mia\*\*
- pseudo: tid, nim, dap, sim, mip, nad, pid
- sentences: `Pip sat.` · `Sam is sad.` · `I am Pip.` · `Tim and Pam sit.` · `Dad naps.` · `The map is in the tin.`

\*\* **Mia** היא דוגמה לשם שאינו פענוחי (ia). שמות הילדים באנגלית יוגדרו מראש כמילים מסובכות אישיות (§19).

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
- סה"כ עד סוף S10 יש **42 מילים** (כולל and/is/his/has, שהופכות לפענוחיות ב-S9 ועוברות לרשימת "פוענחו!" כחגיגה
  קטנה, לא כנטישה).
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
// 4. טקסט מותר אם 100% מהטוקנים מותרים; מילת תוכן אחת לא-פענוחית מותרת רק עם תמונה + tap-to-hear ומסומנת "picture word"
function isDecodable(word: string, unit: Unit, child: ChildId): Verdict;
```

CI: כל פריט שמסומן `decodable: true` עובר את הבודק. אם הוא נכשל, ה-build נכשל (AC2).

### 5.7 כללים לטקסטים פענוחיים (גם לפרומפט של ה-LLM)

- אורך: S2–S4 עד 2 משפטים של 3–5 מילים לעמוד, 3 עמודים. S5–S8 עד 3 משפטים, 4–5 עמודים. S9–S10 עד 4 משפטים,
  6 עמודים.
- כל סיפור כולל את **Pip** ולפחות אחד מגיבורי המחנה, ומסתיים בשינוי קטן במצב (נמצא משהו, נבנה משהו). אין
  אלימות. צללונים מתהפכים לחברים.
- לכל סיפור יש שאלת הבנה אחת לפחות שאפשר לענות עליה בתמונה (`E.RD.TEXT.COMP`).
- כל מילה ניתנת להקשה לשמיעה (Pip). הקשה לא פוגעת בשליטת `PH.*.READ` של מילה פענוחית, אבל נרשמת כ"נעזרה".
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
