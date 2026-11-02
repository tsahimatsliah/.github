# 08 — מבחן מיקום, מנוע אדפטיבי ומודל הילד (`packages/engine`)

> מסמך מימוש. כפוף ל-`00-foundation-decisions.md` (בעיקר C2, C3, C5, C8, C9, E6, E8, E11, F1) ולהכרעות D3, D9, D10 ו-D11 ב-`research_notes/תוכנית הוליסטית לאפליקציית הלמידה/gap_audit.md`.
> הפרוזה בעברית, המזהים באנגלית. כל מספר שאין לידו מקור הוא **ברירת מחדל לכיוון**, והוא מרוכז ב-`EngineConfig` (§11).
> יומן המחקר: `docs/research-log/engine.md`.

---

## 0. מטרה, היקף ועקרונות

**מה המנוע עושה.** לכל ילד ולכל מקצוע הוא (1) קובע נקודת התחלה בעזרת שאלון הורה ו"מסע מיפוי" משחקי, (2) מעדכן אחרי כל ניסיון אומדן יכולת לכל מיומנות, (3) מגדיר רמות שליטה ומתזמן חזרה מרווחת, (4) מרכיב כל פרק (episode) מחדש, (5) מנהל מודל עניין ופרופיל התנהגותי תלוי-הקשר, (6) מעריך כללי "אזור לתשומת לב" ומציג אותם להורה בלבד, ו-(7) מתזמן בדיקות חוזרות.

**מה הוא לא עושה.** הוא לא מצייר UI, לא משמיע קול ולא ניגש לרשת. הוא לא מאבחן, לא משווה בין אחים ולא משתמש במגדר הילד.

**עקרונות מימוש (מחייבים):**

1. **TypeScript טהור** (`strict`, בלי תלויות runtime מלבד ספריית עזר מתמטית קטנה, ובלי `Date.now()` או `Math.random()` בתוך הלוגיקה). הזמן (`now: EpochMs`), אזור הזמן (`Asia/Jerusalem`) ומחולל אקראיות זרוע (`rng: () => number`, ‏mulberry32 עם seed = hash(childId, sessionId)) מוזרקים מבחוץ. אותו קלט נותן תמיד אותו פלט.
2. **Event-sourced** (E6). המצב (`EngineState`) הוא קיפול (fold) של יומן אירועים: `Attempt`, `BehaviorSignal`, `ParentInput` ו-`SessionEvent`. הסדר הקנוני הוא `(ts, deviceId, eventId)`. אחרי סנכרון שמוסיף אירועים "באמצע", המנוע מחשב מחדש מנקודת הביקורת (checkpoint) האחרונה שלפני האירוע המוקדם ביותר. Elo רגיש לסדר, ולכן אין "מיזוג מצבים", רק מיזוג אירועים.
3. **רץ על המכשיר.** כל החישובים מקומיים. גם הנתונים הרגישים הנגזרים (דגלים, נטיות, כיול EF) נשארים במכשירי המשפחה (§9.6).
4. **שקיפות.** כל החלטה של המרכיב מחזירה `reasons[]` קריא, כדי שאזור ההורה יוכל להציג "מה התאמנו ולמה".
5. **שלושה ילדים בלבד.** קושי הפריטים נשען על הערכות מעצב (priors) ועל כללים פשוטים ועמידים. ה-Elo של הפריטים הוא תיקון קטן ותחום, לא כיול אמיתי (gap audit §4.4).

**ארכיטקטורת מודולים:**

```
packages/engine/src/
  types.ts            // §1
  scale.ts            // §2  סולם, עוגנים, priors לפריטים
  placement/          // §3  שאלון, מסעות, חיפוש בינארי, KST
  estimate/           // §4  Elo, HSHS, מסנני RT, K דינמי
  mastery/            // §5  רמות, בדיקה מושהית, דעיכה, סולם חזרות, FIRe-lite
  compose/            // §6  מרכיב פרקים, בורר פריטים, שומר תסכול, בקר רצועת הצלחה
  interest/           // §7  מודל עניין ובנדיט
  profile/            // §8  נטיות תלויות-הקשר, כיול EF, קלט לנרטיב
  flags/              // §9  כללי "אזור לתשומת לב"
  cadence/            // §10 בוס חודשי, חלונות בנצ'מרק, חזרה מחופשה
  config.ts           // §11
  index.ts            // API ציבורי
```

**API ציבורי:**

```ts
export interface Engine {
  reduce(state: EngineState, event: EngineEvent): EngineState;            // טהור
  startPlacement(state: EngineState, childId: ChildId, subject: SubjectId, ctx: Ctx): PlacementRun;
  nextPlacementStep(run: PlacementRun, state: EngineState, ctx: Ctx): PlacementStep; // item | breather | closing | done
  composeEpisode(state: EngineState, childId: ChildId, ctx: Ctx): EpisodePlan;
  nextItem(state: EngineState, rt: EpisodeRuntime, ctx: Ctx): ItemPick;     // כולל החלטות שומר התסכול
  evaluateFlags(state: EngineState, childId: ChildId, ctx: Ctx): FlagState[];
  buildNarrativeInput(state: EngineState, childId: ChildId, month: string, ctx: Ctx): StrengthsNarrativeInput;
  dueCheckpoints(state: EngineState, childId: ChildId, ctx: Ctx): Checkpoint[]; // בוס, בנצ'מרק, חזרה מחופשה
}
export interface Ctx { now: EpochMs; tz: 'Asia/Jerusalem'; rng: () => number; config: EngineConfig; calendar: IsraeliSchoolCalendar; }
```

---

## 1. מודל נתונים (TypeScript)

### 1.1 מזהים ובסיס

```ts
export type ChildId = string; export type SkillId = string; export type ItemId = string;
export type TemplateId = string; export type TopicId = string; export type SessionId = string;
export type StrandId = string;            // ראו את הרשימה הקנונית בהמשך הסעיף
export type EpochMs = number;
export type DayKey = string;              // 'YYYY-MM-DD' לפי Asia/Jerusalem. ימים קלנדריים נמדדים לפי DayKey בלבד
export type SubjectId = 'math' | 'hebrew' | 'english' | 'science' | 'world' | 'ai_literacy' | 'sel' | 'logic' | 'art' | 'music' | 'motor';
export type Tier = 'K1' | 'K2' | 'K3';    // שכבת הגיל של הפרופיל (A2), לא זהות
export type Band = 'T' | 'G' | 'S';       // 00 A6: רצועת גיל של *תוכן* (T 2–4, G 5–6, S 7–10). מיוצא מ-packages/shared/bands.ts
export type Grade = 'toddler' | 'gan_trom' | 'gan_chova' | 'g1' | 'g2' | 'g3' | 'g4' | 'g5' | 'g6';
export type LevelId = 'new' | 'beginner' | 'advancing' | 'proficient' | 'master';
export type ExtraGateId = 'O.REP' | 'O.STEP';   // שכבות-על ממקצוע (03 §3.2). הסמנטיקה ב-§5.1 וב-§4.1
```

**רצועות (A6).** מיומנויות מתויגות ברצועה (`Skill.band`, ועוד `gradeTag` אופציונלי), לא בילד. הילד מקבל רצועה **נגזרת**: `bandOf(child) = grade ∈ {toddler, gan_trom} ? 'T' : grade === 'gan_chova' ? 'G' : 'S'` (ברירת מחדל לפי `tier`: K3→T, K2→G, K1→S), מחושבת מחדש ביום הולדת ובשינוי כיתה (§10.5.7). `bandOf` חי ב-`packages/shared/bands.ts`. המנוע משתמש ברצועה רק כמסנן תוכן (`fringe` לא מציע מיומנות ברצועה שמעל רצועת הילד + 1; מיומנות T לילד S מותרת רק כ-warmup/fun), **לא** כפרמטר של הקושי (θ כבר מכסה את זה). כל הכללים "לפי שכבה" במסמך (`cfg.episode[tier]`, סולמות, RT) נשארים לפי `Tier`, כי הם כללים על הילד ולא על התוכן.

**`SubjectId` סגור.** אין `coding` ו-`finance` (הערת 06 §2): תכנות ממופה ל-`logic` וכסף ל-`world`, וההבחנה לדיווח להורה נשמרת ב-**ציר** (`logic.coding`, `world.finance`). הדשבורד (10) מקבץ לפי `StrandId`, ולכן לא צריך מקצוע נפרד. כך `subjectWeights` נשאר קצר (10 §6.13, סכום 100).

**רשימת הצירים הקנונית (`StrandId`)** — כל ציר הוא סולם לוגיטים אחד (§2.1). הבעלים של התוכן בכל ציר הוא מסמך הקוריקולום, ו-08 מחזיק רק את המזהה:

| מקצוע | צירים | בעלים |
|---|---|---|
| `math` | `math.counting`, `math.magnitude`, `math.number`, `math.operations`, `math.word_problems`, **`math.facts`, `math.fractions`, `math.geometry`, `math.measurement`, `math.patterns_data`** (03 §6.0א) | 03 |
| `hebrew` | `hebrew.pa`, `hebrew.letters`, `hebrew.listening`, `hebrew.decoding`, `hebrew.comprehension`, `hebrew.spelling_morph`, `hebrew.writing` | 04 ⟂VERIFY: 04 עדיין לא מפרסם רשימת צירים ועמוד שדרה. עד אז המזהים כאן הם הקנוניים, ו-04 ממפה אליהם |
| `english` | `english.oral_vocab`, `english.listening`, `english.letter_sounds`, `english.decoding`, `english.speaking` | 05 (§3) |
| `science`, `world`, `logic`, `art`, `music`, `motor` | לפי 06 §2: `science.*`, `world.*` (כולל `world.finance`), `logic.*` (כולל `logic.coding`), `art.*`, `music.*`, `motor.tracing`, `motor.fine`, `motor.gross` | 06 |
| `ai_literacy`, `sel` | `ai_literacy.*`, `sel.*` (ב-`sel` אין רמות מוצגות, C8) | 07 |

**בדיקת תוכן:** חבילת תוכן שבה `Skill.strand` לא ברשימה נדחית בטעינה (`ContentIndex` validator).

### 1.1א שמות שלבים: ההגדרה הקנונית והמיפוי ל-02

| `LevelId` | תווית קנונית | תצוגה להורה | תצוגה לילד (00 C15; 09 מצייר) |
|---|---|---|---|
| `new` | חדש | "שלב: חדש" | זרע |
| `beginner` | מתחיל | "שלב: מתחיל" | נבט |
| `advancing` | מתקדם | "שלב: מתקדם" | שתיל |
| `proficient` | שולט | "שלב: שולט" | עץ |
| `master` | מאסטר | "שלב: מאסטר" | עץ פורח |

התווית מתייחסת **לשלב של המיומנות** ומתאימה לשם העצם "שלב" (זכר דקדוקי), ולכן היא לא תלויה במגדר הילד. לילד מוצגים שמות של שלבי צמיחה, שאין בהם מגדר (B3). **00 C15 קובע את חמשת השמות האלה, גם לילד.** 02 §4.2 (ניצן/נבט/שתיל/פרח/פרי) חייב להתיישר אליהם (fable-issues-F5 → 02): הצמח הוא הרמה; **התג** (ארד/כסף/זהב) הוא תוספת של 02 על `proficient`/`master`/`retained`, לא שלב צמיחה נוסף.

**02 משתמש בשבעה "מצבי טריגר" פנימיים** (`locked → ready → introduced → practicing → proficient → mastered → retained`, 00 C15: "internal economy triggers"). הם **נגזרים** מהרמה ומהגרף, ולא רמות. המיפוי הקנוני לפי C15 (`introduced→new`, `practicing→beginner|advancing`, `proficient→proficient`, `mastered|retained→master`; `locked/ready` = קדם-מצבים בלי רמה), ממומש פעם אחת ב-`packages/shared/levels.ts` ומיובא על ידי המנוע, הכלכלה (02) והדשבורד (10):

| מצב 02 | נגזרת מ-08 | הערה |
|---|---|---|
| `locked` | `level === 'new' && !ready(k)` | קדם-מצב. `ready(k)` = כל הקדם-דרישות הקשות ≥ `proficient` (§3.5, "השוליים") |
| `ready` | `level === 'new' && ready(k) && !firstSeenAt` | קדם-מצב: בשוליים, עוד לא הוצגה |
| `introduced` | `level === 'new' && firstSeenAt` | הוצגה (למשל בלוק מודרך/דוגמה) ועדיין אין תצפית מנוקדת. ← C15 `introduced→new` |
| `practicing` | `level ∈ {beginner, advancing}` | ← C15 `practicing→beginner\|advancing` |
| `proficient` | `level === 'proficient'` | תג ארד (02) |
| `mastered` | `level === 'master' && ladder[rung] < 30` | "Mastered" של C11/D9. תג כסף ויהלום (D1) |
| `retained` | `level === 'master' && ladder[rung] ≥ 30` | עבר את שלב החזרה של 30 יום (02 §4.1). תג זהב |

```ts
// packages/shared/levels.ts — מקור אחד לשני הסולמות (00 C15, E11)
export type LevelId = 'new' | 'beginner' | 'advancing' | 'proficient' | 'master';
export type EconomySkillState = 'locked' | 'ready' | 'introduced' | 'practicing' | 'proficient' | 'mastered' | 'retained';
export const LEVEL_ORDER: readonly LevelId[] = ['new', 'beginner', 'advancing', 'proficient', 'master'];
export const LEVEL_LABEL_HE: Record<LevelId, string> = { new: 'חדש', beginner: 'מתחיל', advancing: 'מתקדם', proficient: 'שולט', master: 'מאסטר' };
export const LEVEL_CHILD_HE: Record<LevelId, string> = { new: 'זרע', beginner: 'נבט', advancing: 'שתיל', proficient: 'עץ', master: 'עץ פורח' };
export function toEconomyState(level: LevelId, ready: boolean, seen: boolean, reviewRungDays: number | undefined): EconomySkillState {
  switch (level) {
    case 'new':        return !ready ? 'locked' : !seen ? 'ready' : 'introduced';
    case 'beginner':
    case 'advancing':  return 'practicing';
    case 'proficient': return 'proficient';
    case 'master':     return (reviewRungDays ?? 0) >= 30 ? 'retained' : 'mastered';
  }
}
export function fromEconomyState(s: EconomySkillState): LevelId | null {   // לייבוא נתונים ישנים ולבדיקות בלבד. null = קדם-מצב בלי רמה
  return ({ locked: null, ready: null, introduced: 'new', practicing: 'beginner', proficient: 'proficient', mastered: 'master', retained: 'master' } as const)[s];
}
```

* **כיוון אחד:** המנוע הוא מקור האמת ל-`level`; 02 ו-10 לא מחשבים רמה בעצמם (AC-05 ב-10). `skill_state_changed` של 02 §2.1 נושא `LevelId` של 08 (`from`, `to`), ו-02 גוזר ממנו את מצבו.
* **הורדת רמה והתג:** `level` יכול לרדת (§5.3), אבל התג של 02 לא יורד לעולם (`badgeNeverDowngrades`). זה לא סתירה: התג הוא היסטוריה ("הגיע פעם ל-master"), והרמה היא מצב נוכחי. `toEconomyState` מחזיר את המצב הנוכחי; 02 שומר בעצמו `maxStateEver`.

### 1.2 Skill

```ts
export interface Skill {
  id: SkillId;
  subject: SubjectId;
  strand: StrandId;
  band: Band;                               // 00 A6: רצועת התוכן (T/G/S). חובה. gradeTag מדייק בתוכה
  titleHe: string;                          // להורה
  kind: 'fact' | 'procedure' | 'concept' | 'discrimination' | 'knowledge' | 'motor' | 'fluency' | 'language';
  prereqs: SkillId[];                       // DAG קשיח: מגדיר "מוכן ללמידה"
  encompasses: { skillId: SkillId; weight: number }[]; // FIRe: אילו מיומנויות מתורגלות באופן חלקי (0..1)
  anchor: number;                           // מיקום בתוכנית בשנות-כיתה: 0 = תחילת גן חובה, 3.0 = תחילת כיתה ג', 3.5 ≈ פברואר בכיתה ג'
  spineOrder?: number;                      // מיקום על "עמוד השדרה" של הציר לצורך מיקום. undefined = לא צומת בדיקה
  gradeTag: Grade;
  monthHint?: number;                       // 1..12: החודש שבו נלמד בדרך כלל
  reviewMode: 'skill' | 'item';             // 'item' = Leitner לכל פריט (אותיות, עובדות, מילים)
  fluencyEligible: boolean;                 // ל-K1 מותר להשתמש ב-RT (סבב מהירות), רק אחרי proficient
  fluencyTargetMs?: number;                 // למשל עובדות כפל: 3000 (ברירת מחדל עיצובית)
  ageMinMonths: number;
  curriculumSource: 'moe_new_2026' | 'moe_old' | 'evidence' | 'unknown'; // gap audit §4.8
  errorTags?: Record<string, SkillId>;      // תג טעות אבחוני → קדם-דרישה חשודה (למשל 'place_value_reversal' → 'math.pv.tens')
  placementEligible: boolean;
  discriminationPairs?: SkillId[];          // לשזירה מבחינה (ב/כ, ד/ר)
  extraGates?: ExtraGateId[];               // 03 §17.2: 'O.REP' = שער ייצוגים ל-proficient (§5.1). 'O.STEP' = ניקוד שלבים (§4.1)
  placementRole?: 'primary' | 'continuation'; // 'continuation' = ציר שלא נבדק במסע, מתחיל ב-prior מ-pos ומיקום עצל (§3.7)
}
```

* **ולידציה בטעינת חבילה:** מיומנות עם `O.REP` חייבת להחזיק לפחות 2 תבניות `reviewed` עם `representation:'pictorial'` ולפחות 2 עם `'abstract'`, אחרת החבילה נדחית (אחרת השער לא ניתן למעבר). מיומנות עם `O.STEP` חייבת להחזיק תבניות שמדווחות `steps` (§1.4).

### 1.3 ItemTemplate ו-Item

```ts
export interface FeatureSpec { key: string; deltas: Record<string, number> }   // תכונה → תוספת קושי בלוגיטים
export interface ItemTemplate {
  id: TemplateId; skillId: SkillId;
  interaction: 'tap_choice' | 'drag' | 'build' | 'trace' | 'speak' | 'type_number' | 'sort' | 'number_line' | 'free_play';
  nOptions?: number;                        // לפריטי בחירה: רצפת ניחוש c = 1/nOptions
  discrimination: number;                   // a. ברירת מחדל 1.5 (פתוח/גרירה/בנייה), 1.2 (בחירה)
  baseDelta: number;                        // היסט קושי מעוגן המיומנות
  features: FeatureSpec[];
  representation?: 'concrete' | 'pictorial' | 'abstract';  // CPA
  minPlausibleMs: number;                   // רצפת ניחוש מהיר לתבנית (§4.4)
  idleMs?: number;                          // דורס את ברירת המחדל של השכבה
  timeLimitMs?: number;                     // D לסבב מהירות (HSHS) בלבד
  answerableAfter: 'prompt_end' | 'target_word' | 'immediate';
  wrapperSlots: string[];                   // מקומות שבהם עטיפת עניין יכולה להחליף סקין
  scored: boolean;                          // false לפעילות יצירה/משחק חופשי
}
export interface Item {
  id: ItemId; templateId: TemplateId; skillId: SkillId;
  featureValues: Record<string, string>;
  betaPrior: number;                        // מחושב ב-scale.ts (§2.3), נשמר בחבילת התוכן
  distractors?: { value: string; errorTag?: string }[];
  promptAudioId: string;
  source: 'authored' | 'generated' | 'photo';
  reviewed: boolean;                        // C7: רק פריטים מאושרים מגיעים לילד
  isProbe: boolean;                         // מתאים למיקום ולבנצ'מרק (חציון קושי של המיומנות)
}
export interface ItemCalibration { itemId: ItemId; beta: number; n: number }        // משותף לשלושת הילדים
export interface TemplateCalibration { templateId: TemplateId; offset: number; n: number }
```

### 1.4 Attempt (אירוע ביומן)

```ts
export type BlockRole = 'warmup' | 'new' | 'review' | 'stretch' | 'fun' | 'closing' | 'boss' | 'probe' | 'breather' | 'delayed_check';
export type AttemptMode = 'placement' | 'practice' | 'boss' | 'benchmark' | 'speed_round' | 'coop' | 'playroom';
export interface Attempt {
  type: 'attempt'; id: string; deviceId: string;
  childId: ChildId; sessionId: SessionId; blockId: string;
  itemId: ItemId; skillId: SkillId; templateId: TemplateId;
  shownAt: EpochMs; answerableAt: EpochMs;  // answerableAt = סוף ההנחיה הקולית/מילת היעד
  firstResponseAt?: EpochMs; completedAt: EpochMs; dayKey: DayKey;
  outcome: 'correct' | 'wrong' | 'too_tricky' | 'timeout' | 'abandoned';
  firstTryCorrect: boolean;
  attemptsCount: number; hintsUsed: number; workedExampleShown: boolean; selfCorrected: boolean;
  chosenErrorTag?: string;                  // התג של המסיח שנבחר בניסיון הראשון
  touchesBeforeAnswer: number;              // לזיהוי הקשות זעם
  idleMaxMs: number;
  appBackgrounded: boolean;
  steps?: { total: number; correct: number; firstErrorTag?: string };   // O.STEP (03 §3.2): פרוצדורה רב-שלבית, K1. firstTryCorrect ⇔ correct === total
  representation?: 'concrete' | 'pictorial' | 'abstract';               // מועתק מהתבנית בזמן ההצגה, לשער O.REP (§5.1)
  context: { role: BlockRole; mode: AttemptMode; assisted: 'none' | 'parent' | 'sibling'; wrapperTopic?: TopicId; afterWorkedExample: boolean };
}
// שם האירוע על החוט (11 §5.8): `item.answered` v2 = בדיוק המבנה הזה. השדות שחסרים ב-v1 של 11 (outcome, role, mode, chosenErrorTag, touchesBeforeAnswer, idleMaxMs, appBackgrounded, assisted, steps, representation) מתווספים (ראו fable-issues-F5 → 11).
// נגזר, לא נשמר:
export interface ScoredObservation { x: number; w: number; useRt: boolean; excluded?: ExclusionReason; responseMs?: number }
export type ExclusionReason = 'rapid_guess' | 'timeout' | 'idle' | 'backgrounded' | 'abandoned' | 'not_scored_template' | 'disengaged_block';
```

### 1.5 SkillState ו-StrandState

```ts
export interface RecentObs { dayKey: DayKey; ts: EpochMs; sessionId: SessionId; firstTryCorrect: boolean; hintFree: boolean; x: number; p: number; role: BlockRole; mixedContext: boolean;
  representation?: 'concrete' | 'pictorial' | 'abstract' }   // ל-O.REP
export interface SkillState {
  childId: ChildId; skillId: SkillId;
  theta: number;                            // יכולת על סולם הציר (לוגיטים)
  uncertainty: number;                      // u ∈ [0,1] → K (§4.3)
  nScored: number; driftEwma: number; driftCount: number;
  level: LevelId; levelSince: EpochMs;
  provenance: 'none' | 'placement' | 'practice' | 'parent' | 'skip_challenge';
  recent: RecentObs[];                      // 12 התצפיות המנוקדות האחרונות
  proficientAt?: EpochMs; proficientDay?: DayKey;
  delayedCheck?: { eligibleFromDay: DayKey; asked: number; correct: number; status: 'pending' | 'passed' | 'failed' };
  firstSeenAt?: EpochMs; lastPracticedAt?: EpochMs; lastPracticedDay?: DayKey;
  fluency?: { medianCorrectMs: number; n: number; personalBestMs?: number; fluent: boolean };
  errorTagStats: Record<string, { count: number; sessions: number; lastAt: EpochMs }>;
  learningProgress: number;                 // ZPDES-lite (§6.4)
  attemptsSinceLevelChange: number; stuck: boolean;
  consecutiveFailedReviews: number;
  needsReview: boolean;
}
export interface StrandState { childId: ChildId; strand: StrandId; theta: number; se: number; updatedAt: EpochMs; placedAt?: EpochMs; ceilingHit?: boolean }
```

### 1.6 ReviewSchedule

```ts
export interface ReviewCard {
  key: string;                              // `${skillId}` או `${skillId}#${itemKey}` (reviewMode='item')
  skillId: SkillId; itemKey?: string;
  ladder: number[];                         // ימים, לפי שכבה (§5.4)
  rung: number;                             // אינדקס ב-ladder
  lastReviewAt: EpochMs; lastReviewDay: DayKey; dueDay: DayKey;
  implicitCredit: number;                   // FIRe-lite ∈ [0,1)
  lapses: number; checkSoon: boolean;
}
export interface ReviewSchedule { childId: ChildId; cards: Record<string, ReviewCard> }
```

### 1.7 ChildProfile

```ts
export interface ChildProfile {
  id: ChildId; displayName: string;
  grammaticalGender: 'm' | 'f' | 'n';       // B3: למחרוזות בלבד. לעולם לא עובר למנוע (ראו EngineChildView)
  birthMonth: string;                       // 'YYYY-MM'
  tier: Tier; grade: Grade;
  readingMode: 'pre_reader' | 'emerging' | 'fluent_pointed' | 'fluent_unpointed';
  successTarget: number;                    // נגזר מ-tier ומ-challengeDial (§6.6)
  challengeDial?: 'easy' | 'just_right' | 'hard';   // K1 בלבד
  stretchZoneOptIn: boolean;                // K1 בלבד (C2)
  rtPolicy: 'ignore' | 'fluency_only';      // נגזר: grade < g1 → 'ignore' (§4.4)
  consent: { adaptive: true; interestInference: boolean; efGames: boolean; narrativeReports: boolean; earlyFlags: boolean;
             checkinNudge: boolean;         // T4b ב-10 §8: דגל EMO_CHECKIN (§9.2). ברירת מחדל false
             parentConcern: { attention: boolean; language: boolean; motor: boolean; emotional: boolean } };
  parentSeeds: { interests: TopicId[]; sensitivities: string[]; temperamentHints: TemperamentHint[] };
  questionnaire: QuestionnaireAnswer[];
  dispositions: DispositionMap;             // §8
  efCalibration?: EFCalibration;            // §8.3
  placement: Partial<Record<SubjectId, PlacementStatus>>;
  modelEpoch: number;                       // §10.5.6: "איפוס התקדמות" מעלה את המונה. אירועים עם epoch קטן יותר לא נכנסים ל-fold
  schoolYear: string;                       // 'תשפ"ז' / '2026-27'. מתעדכן ב-rollover (§10.5.7)
}
export type EngineChildView = Omit<ChildProfile, 'grammaticalGender' | 'displayName'>;  // כל פונקציות המנוע מקבלות רק את זה
export type TemperamentHint = 'slow_to_warm' | 'high_energy' | 'sound_sensitive' | 'needs_movement_breaks' | 'perfectionist_signs';
export interface QuestionnaireAnswer { questionId: string; answer: 'yes' | 'partly' | 'no' | 'unsure'; answeredAt: EpochMs }
export interface PlacementStatus {
  state: 'not_started' | 'in_progress' | 'done'; expeditionsDone: number; strandsDone: StrandId[]; finishedAt?: EpochMs;
  startedAt?: EpochMs; lastStepAt?: EpochMs;
  quality: 'ok' | 'low' | 'inconsistent';   // §3.8: 'low' = SE > 0.6 בציר כלשהו; 'inconsistent' = דפוס תשובות אקראי
  resumable?: { strand: StrandId; posterior: number[]; obs: number; lo: number; hi: number; prev: number; savedAt: EpochMs }; // §3.6 כלל 6
}
```

### 1.8 InterestMap

```ts
export type InterestDomain = 'stem' | 'arts' | 'language_stories' | 'nature_animals' | 'people_places' | 'body_movement';
export type InterestPhase = 'unexplored' | 'sparked' | 'sustained' | 'emerging_passion' | 'dormant';
export interface InterestTopicState {
  topic: TopicId; domain: InterestDomain;
  score: number; peakScore: number; lastDecayAt: EpochMs;
  evidenceDays: DayKey[];                   // ימים עם ראיה חיובית (חלון של 28 יום)
  selfInitiatedDays: DayKey[];              // בחירה חופשית, חזרה לא מונחית, "אני תוהה", "עוד מזה"
  explicitConfirmations: number;            // בחירת תמונה של הילד, "עוד מזה", אישור הורה
  phase: InterestPhase;
  alpha: number; beta: number;              // Beta-Bernoulli לבנדיט
  exposures28d: number; lastExposedAt?: EpochMs;
  snoozedUntil?: EpochMs;                   // "לא מעניין אותי" (K1)
  parentOverride?: 'loves' | 'not_really';
  favorite: boolean;                        // K3: נבחר ≥3 פעמים ב-≥2 ימים
}
export interface InterestMap { childId: ChildId; topics: Record<TopicId, InterestTopicState>; recentChoicePositions: number[] }
```

### 1.9 BehaviorSignal

```ts
interface SignalBase { type: 'signal'; id: string; deviceId: string; childId: ChildId; sessionId: SessionId; ts: EpochMs; dayKey: DayKey;
  context: { subject?: SubjectId; activityKind?: string; topic?: TopicId; itemId?: ItemId } }
export type BehaviorSignal = SignalBase & (
  | { kind: 'error'; retried: boolean; quitWithinMs?: number }
  | { kind: 'hint_request'; msSinceAnswerable: number; attemptsBefore: number }
  | { kind: 'hint_available_unused'; errorsOnItem: number }
  | { kind: 'challenge_choice'; offered: ('easy' | 'medium' | 'hard')[]; chosen: 'easy' | 'medium' | 'hard' }
  | { kind: 'topic_choice'; offered: TopicId[]; chosen: TopicId; position: number }
  | { kind: 'revisit'; topic: TopicId; prompted: boolean }
  | { kind: 'i_wonder'; topic: TopicId }
  | { kind: 'optional_extra'; topic: TopicId; ms: number }
  | { kind: 'reaction'; value: 'up' | 'meh' | 'down'; topic?: TopicId }
  | { kind: 'dwell_after_complete'; topic: TopicId; ms: number }
  | { kind: 'early_quit'; topic?: TopicId; afterError: boolean }
  | { kind: 'idle'; ms: number; afterError: boolean }
  | { kind: 'rage_tap'; taps: number; windowMs: number }
  | { kind: 'rapid_guess_burst'; count: number }
  | { kind: 'break_request' } | { kind: 'calm_tool'; tool: 'dragon_breath' | 'squeeze' | 'count5' | 'hug' }
  | { kind: 'recovery' }                    // חזר והצליח אחרי אפיזודת תסכול באותו מפגש
  | { kind: 'novelty_offer'; accepted: boolean }
  | { kind: 'not_into_this'; topic: TopicId } | { kind: 'want_more'; topic: TopicId }
  | { kind: 'emotion_checkin'; value: 'happy' | 'sad' | 'angry' | 'scared' | 'calm' }   // C8a: `local_only`. התווית לא נשמרת מעבר לפרק; נשאר רק מונה מצטבר ל-14 יום (`checkinCounters`) לצורך EMO_CHECKIN (§9.2)
  | { kind: 'parent_correction'; target: string; value: unknown }
);
```

* **על החוט** (11 §5.8) כל `BehaviorSignal` נשלח כאירוע `behavior.signal` (cls `learn`, פרט ל-`emotion_checkin` שהוא `local_only`). 11 עדיין לא רושם את הסוג הזה (ראו fable-issues-F5).

### 1.9א ParentInput (אירוע ביומן, מקורו באזור ההורה)

```ts
export type ParentInput = { type: 'parent_input'; id: string; deviceId: string; guardianId: string; childId: ChildId; ts: EpochMs; dayKey: DayKey; hlc: string } & (
  | { kind: 'questionnaire'; answers: QuestionnaireAnswer[] }
  | { kind: 'session_reattribute'; sessionId: SessionId; to: ChildId | 'unknown' }      // §10.5.3: "זה לא היה {name}"
  | { kind: 'reset_profile_model' }                                                        // §10.5.6: עניין ונטיות בלבד
  | { kind: 'reset_progress'; subjects: SubjectId[] | 'all' }                              // §10.5.6: מעלה modelEpoch
  | { kind: 'skill_action'; skillId: SkillId; action: 'classTaught' | 'prefer' | 'lock' | 'unlock' | 'skipChallenge' | 'notRelevant' | 'interfaceIssue'; reason?: string }  // 10 API-12
  | { kind: 'flag_action'; ruleId: string; action: 'talkedToTeacher' | 'notRelevant' | 'snooze' | 'print'; reason?: string }   // §9.7
  | { kind: 'class_topic'; topic: ClassTopic }
  | { kind: 'grade_rollover'; from: Grade; to: Grade; schoolYear: string }               // §10.5.7
  | { kind: 'interest_override'; topic: TopicId; value: 'loves' | 'not_really' | 'clear' }
  | { kind: 'disposition_correction'; key: DispositionKey; context?: string; value: unknown }
  | { kind: 'external_evidence'; evidence: 'paperProbe' | 'teacherNote' | 'transfer'; subject?: SubjectId; valueJson: unknown }
);
```

* **סדר וסתירות:** `ParentInput` ממוין יחד עם שאר האירועים בסדר הקנוני. כששני הורים שולחים קלט סותר לאותו יעד (למשל `lock` ו-`unlock` לאותה מיומנות), **האחרון לפי `hlc` קובע** (E6, 10 RB-5), ושניהם נשארים ביומן. אין "מיזוג" של קלטים.
* `session_reattribute` ו-`reset_progress` הם האירועים היחידים שמשנים אירועים **קודמים** (דרך הקיפול: הם מסננים, לא מוחקים). לכן הם מפעילים חישוב מחדש מנקודת הביקורת שלפני האירוע המושפע המוקדם ביותר (§10.5.4).

### 1.10 Session, EpisodePlan, Plan

```ts
export interface Session {
  id: SessionId; childId: ChildId; startedAt: EpochMs; endedAt?: EpochMs; dayKey: DayKey;
  mode: 'episode' | 'mini' | 'placement' | 'boss' | 'benchmark' | 'reentry' | 'playroom' | 'coop';
  plannedMinutes: number; focusedMs: number; coPlay: boolean;
  plan: EpisodePlan; blocksCompleted: number;
  endReason?: SessionEndReason;
  attributedTo?: ChildId | 'unknown';       // §10.5.3: ברירת מחדל = childId. אחרי `session_reattribute` שונה, והניסיונות לא נספרים לילד הזה
  anomaly?: { kind: 'identity_suspect'; score: number; reasons: string[] };   // §10.5.3, מחושב בסיום המפגש
  stats: { scored: number; firstTryCorrect: number; rapidGuesses: number; timeouts: number; frustrationEpisodes: number; celebrationMs: number };
}
// הרשימה הקנונית של סיבות סיום. 11 §5.8 (`session.ended{reason}`), 10 §4.1 ו-02 משתמשים בה כפי שהיא (אין 'childQuit'/'natural'/'windDown' מקבילים):
export type SessionEndReason =
  | 'completed'          // הפרק הסתיים בבלוק closing (10: "טבעי")
  | 'daily_cap'          // תקרה יומית (§6.10)
  | 'child_exit'         // הילד יצא (10 AL-04: "childQuit")
  | 'parent_exit'        // הורה סגר / החלפת שחקן (11 §4.16: 'switch')
  | 'frustration_break'  // הצעת הפסקה התקבלה (§6.7)
  | 'disengaged'         // ניחושים/idle (§4.4)
  | 'bedtime_lock'       // נעילת שינה (11 §5.5)
  | 'inactivity'         // 5 דקות בלי מגע → חזרה לבורר (11 §4.16)
  | 'app_killed';        // לא התקבל session.ended; נסגר בפתיחה הבאה עם endedAt = ts האירוע האחרון + 30 ש'
export interface Block {
  id: string; role: BlockRole; subject: SubjectId; skillIds: SkillId[];
  itemCount: number; targetP: number; scored: boolean;
  scaffold: 'model_guided_independent' | 'independent';
  activityKind: string; wrapperTopic?: TopicId; wrapperIsExploration: boolean;
  reasons: string[];
}
export interface EpisodePlan { childId: ChildId; minutes: number; itemBudget: number; blocks: Block[]; targetP: number; reasons: string[] }
export interface ClassTopic { id: string; skillIds: SkillId[]; phase: 'upcoming' | 'teaching' | 'taught'; setAt: EpochMs; expiresAt: EpochMs; source: 'parent_pick' | 'photo_tag' }
export interface Plan {
  childId: ChildId; schoolYear: '2026-27';
  yearScaffold: Record<number, SkillId[]>;  // חודש → מיומנויות יעד (מ-gradeTag/monthHint)
  classTopics: ClassTopic[];                // "מה למדו החודש"
  subjectWeights: Partial<Record<SubjectId, number>>;
  episodeTargetMin: number; dailyCapMin: number;
  restDays: DayKey[];                       // שבת וחגים (B7), נגזר מהלוח
  speedRoundsEnabled: boolean;              // K1 בלבד
}
```

### 1.11 מצב המנוע

```ts
export interface EngineState {
  content: ContentIndex;                    // skills, templates, items (קריאה בלבד, מחבילת תוכן מאומתת)
  itemCal: Record<ItemId, ItemCalibration>; templateCal: Record<TemplateId, TemplateCalibration>;
  children: Record<ChildId, {
    profile: EngineChildView; plan: Plan;
    skills: Record<SkillId, SkillState>; strands: Record<StrandId, StrandState>;
    reviews: ReviewSchedule; interests: InterestMap;
    flags: Record<string, FlagState>;
    bandController: { adj: number; ewmaSuccess: number; n: number };
    itemsPerMinute: number;                 // EWMA, ‏prior K1/K2 = 2.0
    lastActiveDay?: DayKey; sessions: SessionSummary[];  // 60 האחרונים
    contentExhausted: Partial<Record<StrandId, DayKey>>;  // §10.5.9
  }>;
  checkpointSeq: number;
  lastEventKey?: { ts: EpochMs; deviceId: string; eventId: string };   // המפתח הקנוני האחרון שנקלט
}
```

**נקודות ביקורת (checkpoints):** `EngineState` נשמר ב-SQLite אחרי כל מפגש שהסתיים וגם כל 200 אירועים, עם `lastEventKey`. נשמרות 30 נקודות אחרונות לכל ילד (≈ חודש). כשסנכרון מביא אירוע עם מפתח קטן מ-`lastEventKey` (איחור), או `session_reattribute`/`reset_progress`, המנוע טוען את נקודת הביקורת האחרונה שלפני המפתח המוקדם ביותר שהושפע ומקפל מחדש. אם אין נקודה כזו (האירוע ישן מ-30 נקודות), מקפלים מאפס (S-4: ≤ 1 ms לאירוע, כלומר ≤ 10 ש' לשנה של אירועים, ברקע עם מסך "מסדרים את המפה").

### 1.12 טיפוסי עזר

```ts
export type DispositionKey = 'persistence_after_error' | 'help_seeking' | 'challenge_choice' | 'novelty_approach' | 'curiosity' | 'recovery' | 'session_rhythm';
export interface DispositionEstimate { value: number; n: number; sessions: number; days: number; label: 'hidden' | 'early' | 'consistent';
  trend: '↑' | '→' | '↓'; windowDays: 30; parentCorrected?: { value: unknown; at: EpochMs } }
export type DispositionMap = Record<DispositionKey, { overall: DispositionEstimate; byContext: Record<string /* `${subject}/${activityKind}` */, DispositionEstimate> }>;
export interface EFCalibration { measuredAt: EpochMs; instructionMaxSteps: 1 | 2 | 3; rulesPerActivity: 1 | 2; distractorDensity: 'low' | 'normal'; avoidGoNoGo: boolean }
export interface FlagState { ruleId: string; childId: ChildId; status: 'none' | 'watch' | 'parent_note' | 'referral_suggested' | 'dismissed';
  weeksPersisting: number; firstObservedAt?: EpochMs; lastNotifiedAt?: EpochMs; snoozedUntil?: EpochMs; notify: boolean;
  evidence: { observationHe: string; exposureSessions: number; periodWeeks: number } }
export interface IsraeliSchoolCalendar { schoolYear: string; schoolStart: DayKey; restDays: DayKey[]; vacations: { from: DayKey; to: DayKey; nameHe: string }[];
  benchmarkWindows: { id: 'BOY' | 'MOY' | 'EOY'; from: DayKey; to: DayKey }[]; memorialDays: DayKey[]; verified: boolean }
export interface SessionSummary { id: SessionId; dayKey: DayKey; mode: Session['mode']; focusedMs: number; firstTryRate: number; scored: number;
  frustrationEpisodes: number; lastThirdDrop: number; rapidGuessRate: number; timeouts: number }
```

---

## 2. סולם וקושי

### 2.1 הסולם

* כל ציר (strand) הוא סולם לוגיטים אחד. **יחידה אחת ≈ שנת לימודים אחת** בגילים האלה (`SCALE = 1.0` לוגיט לשנת-כיתה, ניתן לדריסה לכל ציר). כך SE של 0.45 שווה בערך חצי שנת לימודים, וזה דיוק מספיק לבחירת נקודת התחלה.
* עוגן מיומנות: `b_k = SCALE · skill.anchor`.
* מיקום הילד הצפוי בתוכנית: `pos(child, date) = gradeStart(grade) + min(monthsSinceSep1, 10)/10`, בתחום [0, 1] בתוך השנה (ספטמבר–יוני; יולי–אוגוסט נעצרים על 1.0). ‏K1 בנובמבר 2026: ‏`3 + 0.2 = 3.2`. ‏K2: ‏`0.2`. לפעוטה: ‏`(ageMonths − 66)/12`.
* `gradeStart`: ‏`toddler` ו-`gan_trom` לפי נוסחת הגיל, ‏`gan_chova` 0, ‏`g1` 1, ‏`g2` 2, ‏`g3` 3, ‏`g4` 4, ‏`g5` 5, ‏`g6` 6. ה-`grade` מגיע מפרופיל הילד ומתעדכן רק ב-`grade_rollover` (§10.5.7), אף פעם לא לפי תאריך לבד.

### 2.2 מודל התשובה

$$P_i(\theta) = c_i + (1 - c_i)\,\sigma\big(a_i(\theta - \beta_i)\big),\qquad \sigma(z)=\frac{1}{1+e^{-z}}$$

* `c_i = 1/nOptions` בפריטי בחירה, 0 בשאר. ‏`a_i = template.discrimination`.
* למה a = 1.5 ולא Rasch (a = 1)? בסימולציה, עם a = 1 נדרשים 18–21 פריטים כדי להגיע ל-SE ≤ 0.45. עם a = 1.5 מספיקים 8–11. פריטים פתוחים לילדים קרובים לדפוס גוטמן (KST מניח טעויות רשלנות וניחושי מזל קטנים), ולכן a גבוה מ-1 סביר. הפרמטר מכויל לכל תבנית, וזו הנחה עיצובית.

### 2.3 Priors לפריטים מתוך תבניות (cold start)

$$\beta_i = b_k + \underbrace{\text{baseDelta}_T + \sum_f \text{delta}_f(\text{value}_{i,f})}_{\delta_i,\ \text{clamp}[-1.5,\,1.5]} + \tau_T + \varepsilon_i$$

* `δ_i` מוגדר בידי המעצב. דוגמה, תבנית "עובדת כפל a×b":

| תכונה | ערכים → δ |
|---|---|
| `table` | ×1: ‎−1.0, ×10: ‎−0.8, ×2: ‎−0.6, ×5: ‎−0.4, ×3: 0, ×4: ‎+0.1, ×9: ‎+0.3, ×6: ‎+0.4, ×7: ‎+0.6, ×8: ‎+0.6 |
| `format` | בחירה מ-3: ‎−0.3 (ו-c = 1/3), הקלדה: 0 |
| `representation` | מוחשי: ‎−0.4, ציורי: ‎−0.2, מופשט: 0 |
| `commuted` (8×3 אחרי 3×8) | ‎−0.1 |

* `τ_T`: היסט נלמד לכל תבנית, משותף לשלושת הילדים, בתחום ±0.5. פריטים שנוצרו (מ-LLM או מצילום) יורשים אותו מיד.
* `ε_i`: שארית נלמדת לכל פריט, בתחום ±0.6.
* פריטי צילום (photo pipeline) מקבלים `skillId` ותכונות (טווח מספרים, פורמט) מאישור ההורה, ו-β מחושב מאותה נוסחה.

### 2.4 עדכון קושי (קטן ותחום)

אחרי ניסיון מנוקד עם `w > 0`, ורק אם `u_child,k ≤ 0.5` (אומדן הילד אמין דיו):

```
K_item = 0.04 / (1 + n_i/50);           ε_i ← clamp(ε_i − K_item·w·(x − P), −0.6, 0.6);   n_i += 1
K_tmpl = 0.01;                           τ_T ← clamp(τ_T − K_tmpl·w·(x − P), −0.5, 0.5)
```

עם שלושה ילדים, הזזה של 0.1 לוגיט דורשת עשרות ניסיונות. זו התנהגות מכוונת: ה-priors הם הליבה.

---

## 3. מבחן מיקום ("מסע מיפוי המפה")

### 3.1 זרימה לפי שכבה

| | K3 (2.5) | K2 (5.5, גן חובה) | K1 (8.5, כיתה ג') |
|---|---|---|---|
| שאלון הורה | 12 פריטים (~4 דק') | 15 פריטים (~5 דק') | 16 פריטים (~5 דק') |
| מסע משחקי | **אין.** טלמטריית חדר משחקים בלבד | 3 מסעות × 3–5 דק' (רביעי אופציונלי) | 3–4 מסעות × 8–10 דק' |
| פריסה | — | 2–4 ימים, ≤2 מסעות ביום | 2–4 ימים, עדיף מסע אחד ביום |
| פריטים לציר | — | מינ' 4, מקס' 8 | מינ' 6, מקס' 12 |
| פריטים למקצוע | — | ≤14 | 15–25 (מקס' 25) |
| כלל עצירה לציר | — | SE ≤ 0.45 או מקס' או ניתוק | SE ≤ 0.45 או מקס' או ניתוק |
| יעד הצלחה בשלב R | — | ~0.70 (offset 0.55) | ~0.63 (offset 0.35) |

המספרים מעוגנים ב-ALEKS Adventure (≤9 פריטים לגן, ≤14 לכיתות א'–ג') וב-`placement_assessment.md` §1. עם prior של שאלון, K1 מגיע בסימולציה ל-SE ≤ 0.45 אחרי 8–11 פריטים לציר. ‏K2 נעצר לרוב במקסימום, עם SE של 0.50–0.55. במקרה כזה מוצג להורה טווח ("בערך…") עד שהמשחק השוטף מצמצם אותו (בנוסח IXL).

### 3.2 מבנה המסעות

**K2 ("הרפתקת היכרות").** הדמות מבקשת עזרה למצוא אותיות שאבדו ולספור כוכבים לחללית. הכול מונחה קולית ואין צורך לקרוא.

| מסע | צירים | הערות |
|---|---|---|
| E1 מספרים | `math.counting` (ספירה, עוקב, ספירה ממשיכה), `math.magnitude` (סוביטיזציה 1–5 בהבזק של ~1 ש', השוואת כמויות/ספרות, ישר מספרים 0–10) | פריטים בהקשה פשוטה קודם (Anthony et al.: ילדים מחטיאים מטרות מגע) |
| E2 צלילים ואותיות | `hebrew.pa` (צליל פותח, חרוז, מחיאת הברות), `hebrew.letters` (צליל-אות ושם-אות מתוך 3) | בלי פענוח (B6: פענוח פורמלי מתחיל בכיתה א') |
| E3 סיפור | `hebrew.listening` (הבנת הנשמע של סיפור קצר), `english.oral_vocab` (שמע-והקש, אופציונלי) | אנגלית באי LTR (B2) |
| E4 (אופציונלי) | `motor.tracing` | תצפיתי בלבד, לא מנוקד |

**K1 ("מסע בחירת גיבור").** כל תחנה פותחת כלי.

| מסע | צירים | הערות |
|---|---|---|
| E1 חשבון א' | `math.number` (ערך מקום עד 1000, ישר מספרים 0–1000), `math.operations` (חיבור/חיסור), `math.facts` (3–4 עובדות מייצגות לכל קבוצה, 03 §14.1; מקס' 6 פריטים) | RT נרשם בשקט, לא משפיע (§4.4). צירים `primary` |
| E2 עברית א' | `hebrew.decoding` (מילים ופסאודו-מילים מנוקדות + 3 פריטים לא מנוקדים לפער הניקוד), `hebrew.comprehension` | אותו טקסט מנוקד ולא מנוקד (B5) |
| E3 חשבון ב' + אנגלית | `math.word_problems` (מוקראות בקול כדי להפריד בין קריאה לחשבון), `math.fractions` (3 פריטי `isProbe` בלבד, ציר `continuation`), `english.listening`, `english.letter_sounds` | אי LTR. תקרת 25 פריטי חשבון למקצוע נשמרת: E1 ≤ 16, ‏E3 חשבון ≤ 9 |
| E4 עברית ב' | `hebrew.spelling_morph` (הכתבה של הומופונים ט/ת, כ/ח, ק/כ, א/ע; שורש משותף) | קטע קריאה בקול (ASR, E9) **אופציונלי**, רק בנוכחות הורה |
| — (ללא מסע) | `math.geometry`, `math.measurement`, `math.patterns_data` | צירי `continuation`: מיקום עצל בתרגול (§3.7) |

### 3.3 שאלון ההורה

כל שאלה ממופה למיומנות בעוגן ידוע, והתשובות נכנסות לפוסטריור **כתצפיות-פסאודו**:

| תשובה | תצפית |
|---|---|
| `yes` | x = 1, w = 0.7 |
| `partly` | x = 1, w = 0.35 + x = 0, w = 0.35 |
| `no` | x = 0, w = 0.7 |
| `unsure` | אין |

המשקל 0.7 משקף שדיווח הורים פחות אמין מתצפית. הניסוח מקורי, בהשראת התחומים של ASQ-3/CDI, ולא מועתק מהם (הם כלים ברישיון).

**K3** (`{m|f}` = גרסת זכר/נקבה לפי B3):

| ID | שאלה | מיפוי |
|---|---|---|
| T1 | {name} {אומר\|אומרת} לפחות 50 מילים שונות? | `lang.vocab_expressive` |
| T2 | {name} {מחבר\|מחברת} שתי מילים יחד ("עוד מים", "אבא הלך")? | `lang.combinations` |
| T3 | {name} {מבצע\|מבצעת} בקשה של שני שלבים ("קח את הכדור ושים בסל")? | `lang.receptive` |
| T4 | {name} {מצביע\|מצביעה} על חלקי גוף כשמבקשים? | `lang.receptive_body` |
| T5 | {name} {מזהה\|מזהה} בעלי חיים או מחקה את הקולות שלהם? | `world.animals` |
| T6 | {name} {מתאים\|מתאימה} צבעים או {קורא\|קוראת} להם בשם? | `math.colors_shapes` |
| T7 | {name} {סופר\|סופרת} "אחת, שתיים, שלוש" ומצביע/ה? | `math.counting_rote3` |
| T8 | {name} {מזהה\|מזהה} את האות הראשונה של השם שלו/ה? | `hebrew.name_letter` |
| T9 | {name} {בונה\|בונה} מגדל של 4 קוביות או יותר? | `motor.fine` |
| T10 | {name} {מצליח\|מצליחה} להקיש ולגרור על המסך? | כיול אינטראקציה (לא מיומנות) |
| T11 | {name} {מקשיב\|מקשיבה} לסיפור קצר 3–5 דקות? | `TemperamentHint` / אורך פעילות |
| T12 | יש רגישות לרעשים חזקים או למשהו אחר שכדאי שנדע? | `sensitivities` |

**K2:**

| ID | שאלה | מיפוי |
|---|---|---|
| G1 | כמה אותיות {name} {מכיר\|מכירה} בשם? (אף אחת / כמה / רובן) | `hebrew.letter_names` (רובן = yes) |
| G2 | {name} {יודע\|יודעת} איזה צליל עושה אות (למשל מ' = /m/)? | `hebrew.letter_sounds` |
| G3 | {name} {מזהה\|מזהה} את השם שלו/ה כתוב? | `hebrew.name_reading` |
| G4 | {name} {כותב\|כותבת} את השם שלו/ה? | `motor.name_writing` |
| G5 | {name} {ממציא\|ממציאה} חרוזים ("חתול–שתול")? | `hebrew.pa.rhyme` |
| G6 | {name} {יודע\|יודעת} באיזה צליל מתחילה מילה ("שמש" מתחילה ב-ש)? | `hebrew.pa.first_sound` |
| G7 | {name} {סופר\|סופרת} בעל פה עד 10 / 20 / 30? | `math.rote_count` (שלוש תצפיות, לכל סף) |
| G8 | {name} {סופר\|סופרת} נכון עד 10 חפצים ויודע/ת "כמה יש"? | `math.cardinality10` |
| G9 | {name} {מזהה\|מזהה} את הספרות 1–10? | `math.numerals10` |
| G10 | {name} {יודע\|יודעת} איפה יש יותר (7 או 4)? | `math.compare` |
| G11 | {name} {פותר\|פותרת} "יש לך 2 ועוד 1, כמה יש?" בלי חפצים? | `math.add_within5` |
| G12 | {name} {מבין\|מבינה} סיפור קצר ועונה על שאלות עליו? | `hebrew.listening` |
| G13 | {name} {מכיר\|מכירה} מילים באנגלית (צבעים, מספרים, בעלי חיים)? | `english.oral_vocab` |
| G14 | {name} {מחזיק\|מחזיקה} עיפרון ומעתיק/ה צורה פשוטה? | `motor.tracing` |
| G15 | מה {name} {אוהב\|אוהבת} במיוחד ומדבר/ת על זה בלי סוף? (בחירה מרשימת נושאים) | `parentSeeds.interests` |

**K1:**

| ID | שאלה | מיפוי |
|---|---|---|
| S1 | {name} {קורא\|קוראת} טקסט מנוקד ברצף, בלי לעצור על כל מילה? | `hebrew.decoding_pointed` |
| S2 | {name} {קורא\|קוראת} מילים בלי ניקוד? | `hebrew.decoding_unpointed` |
| S3 | {name} {קורא\|קוראת} ספרים בעצמו/ה? | `hebrew.comprehension` |
| S4 | {name} {נהנה\|נהנית} לקרוא? | `interest: reading` (לא מיומנות) |
| S5 | בכתיבה יש הרבה בלבול בין ט/ת, כ/ח, ק/כ, א/ע? (yes = תצפית שלילית) | `hebrew.spelling_homophones` |
| S6 | {name} {כותב\|כותבת} משפטים שלמים לבד? | `hebrew.writing_sentence` |
| S7 | לוח הכפל עד 5? | `math.facts_x_to5` |
| S8 | לוח הכפל עד 10? | `math.facts_x_to10` |
| S9 | חיבור וחיסור עד 100 בראש? | `math.add_sub100` |
| S10 | מספרים עד 1000 (מאות, עשרות, יחידות)? | `math.pv1000` |
| S11 | בעיות מילוליות קצרות? | `math.word_problems` |
| S12 | שברים פשוטים (חצי, רבע)? | `math.fractions_intro` |
| S13 | אותיות וצלילים באנגלית? | `english.letter_sounds` |
| S14 | {name} {קורא\|קוראת} מילים פשוטות באנגלית? | `english.decoding_cvc` |
| S15 | מה למדו בכיתה החודש? (בחירה מרשימה או צילום דף) | `Plan.classTopics` |
| S16 | יש משהו בשיעורי הבית שמתסכל לאחרונה? | הקשר לפרופיל, לא תצפית |

**בדיקה צולבת:** אם `|θ̂_placement − μ_questionnaire| > 1.0` לוגיט, נרשם `questionnaire_mismatch`, ולהורה מוצג משפט רך: "ראינו במשחק תמונה קצת שונה ממה שסימנתם. זה טבעי, ונמשיך לכייל תוך כדי משחק."

**שאלון שדולג:** `prior = N(SCALE·(pos − 0.2), σ0 + 0.2)` ו-`maxItems` של הציר +2 (בתוך תקרת המקצוע). להורה מוצג "אפשר למלא אחר כך" ב-PA-ASM, ותשובות מאוחרות נכנסות כתצפיות-פסאודו **רק** למיומנויות שעדיין `u ≥ 0.5` (אחרת הראיה מהמשחק כבר טובה יותר).

### 3.3א מזהי שאלון קנוניים (מחליפים את המזהים הזמניים)

המזהים בטבלאות T/G/S למעלה הם **מזהי שאלון** (`questionId`). המיפוי למיומנויות אמיתיות בחבילות התוכן נמצא בקובץ `packages/content/questionnaire-map.json`, שהתוכן שלו הוא הטבלה הזו. שאלה יכולה למפות לכמה מיומנויות, כל אחת עם משקל (ברירת מחדל 1.0, מוכפל ב-`questionnaireWeight`):

| `questionId` | מיומנויות (מסמך) |
|---|---|
| T6 `math.colors_shapes` | `M.GEO.SHAPE.TYPICAL`, `M.DATA.SORT.ONE` (03) |
| T7 `math.counting_rote3` | `M.NUM.COUNT.ROTE3` (03) |
| G7 `math.rote_count` 10/20/30 | `M.NUM.COUNT.ROTE10` / `ROTE20` / `ROTE30` (03) — שלוש תצפיות, אחת לכל סף |
| G8 `math.cardinality10` | `M.NUM.COUNT.OBJ10` (03) |
| G9 `math.numerals10` | `M.NUM.NUMERAL.ID10` (03) |
| G10 `math.compare` | `M.NUM.COMP.SYM10` (1.0), `M.NUM.COMP.COUNT10` (0.5) (03) |
| G11 `math.add_within5` | `M.OPS.JOIN.C5` (03) |
| S7 `math.facts_x_to5` | `M.FACT.MUL.X2`, `X5`, `X3`, `X4` (03) |
| S8 `math.facts_x_to10` | `M.FACT.MUL.MIXED` (03) |
| S9 `math.add_sub100` | `M.OPS.ADD.2D2D`, `M.OPS.SUB.2D2D` (03) |
| S10 `math.pv1000` | `M.PV.HUND1000` (03) |
| S11 `math.word_problems` | `M.WP.BAR.PPW`, `M.WP.BAR.CHANGE` (03) |
| S12 `math.fractions_intro` | `M.FRAC.UNIT` (03) |
| G13 `english.oral_vocab` | `E.VO.COL.REC`, `E.VO.NUM.REC`, `E.VO.ANI.REC` (05 §3; `english.oral_vocab` = `E.VO.*.REC`) |
| S13 `english.letter_sounds` | `E.PH.SAT.GPC` (05; `english.letter_sounds` = `E.PH.*.GPC`, היחידה הראשונה) |
| S14 `english.decoding_cvc` | `E.PH.SAT.BLEND` (05; `english.decoding_cvc` = `E.PH.*.BLEND`) |
| T1–T5, T8–T9, G1–G6, G12, G14, S1–S6 | `H.*` (04), `lang.*`/`motor.*`/`world.*` (06) — ⟂VERIFY: 04 ו-06 טרם פרסמו טבלת מיפוי. עד אז הקובץ מחזיק את המזהה הזמני, והטעינה מציגה אזהרת dev "unmapped questionId" (לא כשל) |

* **צירי מיקום באנגלית** (05 §3): ‏`english.oral_vocab` = עמוד שדרה מ-`E.VO.*.REC`; ‏`english.listening` = `E.OR.L.*.REC`; ‏`english.letter_sounds` = `E.PH.*.GPC` לפי סדר היחידות; ‏`english.decoding` = `E.PH.*.BLEND`. השם `english.listening_vocab` שהופיע בטבלת E3 של K1 הוחלף ב-`english.listening`.
* **בדיקה:** `questionnaire-map.json` עובר ולידציה מול `ContentIndex` ב-CI (כל מיומנות קיימת, או מסומנת `provisional`). כל מיומנות שבשאלון ואינה `placementEligible` עדיין מקבלת תצפית-פסאודו.

### 3.4 האלגוריתם (לכל ציר)

**מבנה.** לכל ציר יש עמוד שדרה: רשימת מיומנויות `placementEligible` ממוינות לפי `b_k`, במרווח טיפוסי של 0.5 (כחצי שנה). הפוסטריור על θ נשמר ברשת (grid) של `[-4, 4]` בצעד 0.05 (161 נקודות).

```ts
function placeStrand(strand, child, prior: {mu: number; sd: number}, cfg): PlacementResult {
  // 0. Prior: נורמלי, ועליו תצפיות-הפסאודו מהשאלון
  let post = gridPrior(prior.mu, prior.sd);
  for (const q of questionnairePseudoObs(child, strand)) post = update(post, q.b, q.x, q.w, q.a ?? 1.5, 0);

  const spine = spineOf(strand);                  // מערך b ממוין
  const obs: Obs[] = []; let consecMiss = 0; let prev: number;

  // 1. חימום (התחלה קלה): 2 פריטים בצומת הגבוה ביותר עם P ≥ 0.90
  const w = argmaxIndex(spine, i => predictive(post, spine[i]) >= 0.90) ?? 0;
  let lo = w, hi = firstIndex(spine, i => predictive(post, spine[i]) <= 0.10) ?? spine.length;
  for (let k = 0; k < 2; k++) ask(w, 'warmup');
  if (lastTwoBothFailed()) { lo = -1; hi = w; }   // רצפה: חיפוש מתחת לנקודת החימום
  prev = w;

  // 2. לולאה: חיפוש בינארי (B), ואחריו עידון (R)
  while (true) {
    const { mean, sd } = moments(post);
    if (obs.length >= cfg.minItems && sd <= cfg.seStop) return finish('SE');
    if (obs.length >= cfg.maxItems) return finish('cap');
    if (disengagedStreak() >= 3) return finish('disengaged');           // §4.4
    if (child.tier === 'K2' && breatherUsed && missedRightAfterBreather && obs.length >= cfg.minItems)
      return finish('frustration');                                     // K2: 2 החטאות → מנוחה → החטאה נוספת = סיום הציר

    let i: number, role: BlockRole;
    if (consecMiss >= 2) { i = argmaxIndex(spine, j => predictive(post, spine[j]) >= 0.90) ?? 0; role = 'breather'; }
    else if (hi - lo > 1) {                       // B: חציה של [ההצלחה הגבוהה ביותר, הכישלון הנמוך ביותר]
      i = Math.floor((lo + hi) / 2);
      while (Math.abs(spine[i] - spine[prev]) > cfg.maxJump) i += i > prev ? -1 : 1;   // maxJump = 1.5
      role = 'probe';
    } else {                                       // R: בדיקה ליד μ פחות היסט
      i = argminIndex(spine, j => [Math.abs(spine[j] - (mean - cfg.offset)), j]);
      role = 'probe';
    }
    const r = ask(i, role);                        // פריט isProbe, קרוב לחציון הקושי של המיומנות
    if (role === 'probe' && hi - lo > 1) { if (r.x === 1) lo = Math.max(lo, i); else hi = Math.min(hi, i); }
    consecMiss = role === 'breather' ? 0 : (r.x === 0 ? consecMiss + 1 : 0);
    prev = i;
  }

  function ask(i, role) {
    const item = pickProbeItem(spine[i]);
    const a = render(item, { feedback: 'neutral_map_reveal', timerVisible: false });   // §3.6
    const o = score(a);                            // §4.1, כולל מסנני RT (§4.4)
    if (!o.excluded) { post = update(post, item.beta, o.x, o.w, item.a, item.c); obs.push(o); }
    return o;
  }
  function finish(reason) {
    closingWinItem();                              // פריט ניצחון עם P ≥ 0.90. לא נכנס לפוסטריור, כן נכנס ל-Elo הרגיל
    const { mean, sd } = moments(post);
    return { theta: mean, se: sd, reason,
             ceilingHit: reason === 'cap' && lastNProbesAllCorrect(4),   // המסע הבא ממשיך כלפי מעלה עם prior = הפוסטריור
             classes: classifySkills(strand, post, obs) };               // §3.5
  }
}
```

* **עדכון הפוסטריור:** `post(θ) ∝ post(θ) · P(θ)^{w·x} · (1−P(θ))^{w·(1−x)}`, עם P מ-§2.2. ‏"קשה לי, אפשר אחר?" נספר כ-x = 0 עם w = 0.5.
* **Prior לציר:** `μ0 = SCALE·(pos − 0.2)`, כלומר מתחילים מעט מתחת לרמה הצפויה. ‏`σ0 = 0.8` ל-K1 ו-`1.0` ל-K2. לציר השני והלאה באותו מקצוע: `μ0 += 0.5·(θ̂_prev − μ0_prev)` ו-`σ0 = 0.9`.
* **ריבוי מקצועות:** תוצאת ציר אחד מזיזה את ה-prior של הציר הבא באותו מקצוע. בין מקצועות אין העברה.
* **מינימום לציר:** חימום + 4 בדיקות (K2) או חימום + 6 בדיקות (K1), אלא אם עצירה בגלל ניתוק או תסכול.

### 3.5 סיווג בסגנון KST וסגירה על הגרף

```ts
function classifySkills(strand, post, obs): Record<SkillId, 'known' | 'uncertain' | 'unknown'> {
  const out = {};
  for (const k of skillsOfStrand(strand)) {                 // כל המיומנויות בציר, לא רק עמוד השדרה
    const p = predictive(post, b(k));                        // Σ post(θ)·σ(a(θ − b_k))
    out[k] = p >= 0.85 ? 'known' : p <= 0.40 ? 'unknown' : 'uncertain';
    const d = directEvidence(obs, k);                        // הצלחות/כישלונות ישירים במיומנות
    if (d.fail >= d.success && d.fail > 0 && out[k] === 'known') out[k] = 'uncertain';
    if (d.success >= 2 && d.fail === 0 && p >= 0.75) out[k] = 'known';
  }
  // סגירה כלפי מטה: known ⇒ כל הקדם-דרישות (טרנזיטיבית) known, אם אינן מנוגדות לראיה ישירה
  for (const k of topoDesc(strandSkills)) if (out[k] === 'known') for (const j of ancestors(k)) {
    if (directEvidence(obs, j).fail >= 2) { out[k] = 'uncertain'; out[j] = 'uncertain'; } else if (out[j] !== 'known') out[j] = 'known';
  }
  // סגירה כלפי מעלה: unknown ⇒ כל הצאצאים unknown
  for (const k of topoAsc(strandSkills)) if (out[k] === 'unknown') for (const s of descendants(k)) out[s] = 'unknown';
  return out;
}
```

**אתחול `SkillState` מהמיקום:**

| class | level | theta | u | חזרה |
|---|---|---|---|---|
| `known` | `proficient`, ‏`provenance:'placement'` | θ̂_s | 0.4 | כרטיס ב-rung 1 (3 ימים) כבדיקת אישור. כישלון מוריד ל-`advancing` בשקט |
| `uncertain` | `beginner` | θ̂_s | 0.6 | — |
| `unknown` | `new` | θ̂_s | 0.5 | — |

`delayedCheck.eligibleFromDay = placementDay + 3` למיומנויות `known`. כך "מאסטר" מגיע רק אחרי בדיקה מושהית אמיתית (D9). **השוליים** (fringe, מה שמוכן ללמידה) הם מיומנויות `uncertain`, ומיומנויות `unknown` שכל הקדם-דרישות שלהן `known`/`proficient`.

### 3.6 כללי UI למיקום (מחייבים)

1. **בלי משוב נכון/לא נכון.** כל תשובה, נכונה או שגויה, חושפת חלק מהמפה עם אותה אנימציה ניטרלית-חיובית ("תודה, המפה גדלה!", ≤0.8 ש' לפי D7). בלי ניקוד, כוכבים או לבבות.
2. **התחלה קלה וסיום קל.** שני פריטי חימום עם P ≥ 0.9, ופריט סיום עם P ≥ 0.9 שלאחריו "המפה מוכנה!".
3. **"קשה לי, אפשר אחר?"** הדמות מושכת בכתפיים ("בוא/י נדלג", בגרסת מגדר). אין עונש גלוי.
4. **בלי טיימר גלוי.** RT נרשם בשקט.
5. **פריט מנוחה** אחרי 2 החטאות רצופות.
6. **עד 10 דקות למסע** ל-K1 ו-5 דקות ל-K2. אם הזמן נגמר באמצע ציר, הציר נשמר כ-`in_progress` ומתחדש מהפוסטריור בפעם הבאה. **כללי הפסקה באמצע ציר** (כל יציאה: זמן, `child_exit`, `app_killed`, `bedtime_lock`, רקע > 60 ש'):
   * הפוסטריור, `obs`, `lo/hi/prev` נשמרים ב-`PlacementStatus.resumable` **אחרי כל פריט** (אירוע `placement.step` הוא מקור האמת; `resumable` הוא נגזרת שלו, ולכן גם סנכרון ממכשיר אחר משחזר אותו).
   * חידוש בתוך **14 יום**: ממשיכים מאותו מקום, עם פריט חימום אחד (P ≥ 0.9) לפני ההמשך, ו-`maxItems` לא מתאפס.
   * אחרי 14 יום, או אם הילד שיחק בינתיים ≥ 3 פרקים במקצוע: הציר מתחיל מחדש עם `prior = N(mean(post), max(sd(post), 0.7))`, כי מה שנלמד בינתיים שינה את התמונה. התצפיות הישנות נשארות ביומן ולא נכנסות שוב.
   * ציר שנעצר אחרי `minItems` נסגר כ-`finish('interrupted')` (כמו `cap`), ולא נפתח מחדש.
   * מסע שלא הושלם ב-**21 יום** מהפתיחה: המקצוע מסומן `placement.state='done'` עם `quality:'low'`, הצירים שלא נבדקו מקבלים prior מ-`pos` (כמו `continuation`, §3.7), והמשחק השוטף ממשיך. להורה: "סיימנו את המיפוי עם מה שיש; המשחק ימשיך לדייק."
7. **אחים לא רואים זה את זה.** המיקום מתבצע בפרופיל האישי. מסע מיקום נפתח רק אם `session.childId` זהה לפרופיל שנבחר בבורר **ואין** `anomaly.identity_suspect` במפגש הקודם (§10.5.3); אחרת המסע נדחה לפעם הבאה בלי הסבר לילד.
8. **פריט אחד במסך**, הנחיה קולית (B1) ואייקון. ‏K2 לא צריך לקרוא.
9. **ירח דבש:** שני הפרקים הראשונים אחרי המיקום רצים עם `targetP = 0.90`, כדי שהשיעור הראשון יצליח.
10. **בלי עזרה מההורה** (10 ON-12). אם בכל זאת `assisted ≠ 'none'` (ההורה לחץ "עזרתי"), הפריט נכנס לפוסטריור ב-w × 0.3 כמו ב-§4.1.

### 3.7 צירי המשך (`placementRole:'continuation'`) ומיקום עצל

צירים שאין להם מקום במסעות (K1: `math.geometry`, `math.measurement`, `math.patterns_data`, וגם `math.fractions` כשהזמן ב-E3 נגמר; K2: כל צירי `math.*` החדשים; כל המקצועות של 06/07) **לא נבדקים במסע**. במקום זה:

1. `StrandState` מאותחל ל-`theta = SCALE·(pos − 0.2)`, ‏`se = 1.0`, ‏`placedAt = undefined`. תצפיות-פסאודו מהשאלון נכנסות כרגיל.
2. כל המיומנויות בציר מקבלות `level:'new'`, ‏`u = 1.0` (K = 0.50), ‏`provenance:'none'`. השוליים נגזרים מהגרף בלבד.
3. **מיקום עצל:** בפעם הראשונה שהמרכיב בוחר מיומנות מהציר כמוקד, הבלוק `new` שלה מתחיל ב-**3 פריטי `isProbe`** ברמת המיומנות (role `probe`, ‏`targetP` רגיל, **עם** משוב רגיל, כי זה תרגול ולא מיקום). הם נכנסים ל-Elo עם K = 0.50, ולכן 3 פריטים מזיזים θ עד ±0.75 לוגיט. אם שלושתם נכונים → המיומנות מסומנת `uncertain→known` לפי §3.5 (סגירה כלפי מטה חלה), והמוקד עובר לצאצא. אם שלושתם שגויים → קדם-דרישה (כמו `stuck`).
4. `placedAt` נקבע כשהציר צבר ≥ 8 תצפיות מנוקדות ו-`se ≤ 0.6` (SE של הציר מחושב כ-`1/sqrt(Σ a²·P(1−P))` על 20 התצפיות האחרונות, עם רצפה 0.3).
5. **אין "מקדים/בדרך/בבנייה" לציר המשך** ב-PA-ASM (10 W-ASM-05) עד `placedAt`; מוצג "עוד לא מופה, המשחק מכייל תוך כדי".

### 3.8 תשובות אקראיות במיקום (ילד שמקיש בלי לחשוב)

הסימנים (בודקים אחרי `minItems`): (א) `disengagedStreak ≥ 3` כבר מסיים את הציר (§3.4). (ב) **חוסר עקביות גוטמן:** ≥ 2 הצלחות בצמתים שמעל ≥ 2 כישלונות (b גבוה יותר ב-≥ 1.0 מהכישלון), או `sd(post)` שלא קטן ב-≥ 0.05 במשך 4 פריטים רצופים אחרי `minItems`. (ג) `responseMs` מתחת לרצפת הגיל ב-≥ 50% מהפריטים (גם אם לא ברצף).

כשמתקיים (ב) או (ג): `finish('inconsistent')`, ‏`quality:'inconsistent'` למקצוע, ‏`se = max(se, 0.7)`, וכל מיומנויות הציר מאותחלות כ-`uncertain` (לא `known` ולא `unknown`) עם `u = 0.8`. אין חזרה על המסע באותו יום. המסע לציר הזה נפתח שוב **פעם אחת** אחרי ≥ 2 ימים, כ"מסע קצר" (`maxItems` = `minItems` + 2). אם גם הוא `inconsistent`, המקצוע נסגר ב-`quality:'low'` והתרגול הרגיל מכייל (K גבוה). להורה ב-PA-ASM: "המיפוי בחשבון יצא לא ברור (אולי {name} {מיהר|מיהרה}). המשחק ממשיך לכייל תוך כדי, ואפשר להציע לנסות שוב בזמן רגוע." (לא מוצג לילד).

---

## 4. הערכה שוטפת (גרסת Elo)

### 4.1 ניקוד ניסיון (`score(attempt) → ScoredObservation`)

| מצב | x | w |
|---|---|---|
| נכון בניסיון ראשון, בלי רמז | 1 | 1 |
| נכון אחרי תיקון עצמי (K1, ניסיון שני מותר) | 0.5 | 1 |
| נכון אחרי רמז | 0.3 | 0.5 |
| שגוי בניסיון ראשון (כולל אחרי רמז או דוגמה פתורה) | 0 | 1 |
| "קשה לי" | 0 | 0.5 |
| פריט "כמעט זהה" מיד אחרי דוגמה פתורה | כרגיל | ×0.5 |
| בסיוע הורה או אח (`assisted ≠ 'none'`) | כרגיל | ×0.3 |
| K3 (כל פריט, משחק משותף) | כרגיל | ×0.5 |
| **פרוצדורה רב-שלבית (`O.STEP`, ‏K1, ‏03 §3.2)** בניסיון ראשון, בלי רמז | `correctSteps / steps` | 1 |
| ניחוש מהיר / timeout / idle / רקע / נטישה / תבנית לא מנוקדת | — | 0 (`excluded`) |
| מפגש שהורה סימן "לא {name}" (`attributedTo ≠ childId`, §10.5.3) | — | 0 (`excluded:'reattributed'`) |

ספירות השליטה (§5) משתמשות רק ב-`firstTryCorrect && hintsUsed === 0` על תצפיות לא מוחרגות. **ב-`O.STEP`:** `firstTryCorrect ⇔ steps.correct === steps.total`; ‏x חלקי נכנס ל-Elo (הנוסחה ב-§4.2 מקבלת x ∈ [0,1]), ל-`learningProgress` ול-BKT הצל (שם x ≥ 0.999 נחשב נכון). `steps.firstErrorTag` ממלא את `chosenErrorTag` ומזין את §5.5. תיקון עצמי בפרוצדורה (K1, ניסיון שני): x = 0.5 × (correctSteps₂/steps), כלומר לכל היותר 0.5.

### 4.2 עדכון (מצב ברירת מחדל: דיוק בלבד)

```ts
function eloUpdate(s: SkillState, st: StrandState, item: ItemResolved, o: ScoredObservation, cfg): void {
  if (o.excluded || o.w === 0) return;
  const P  = item.c + (1 - item.c) * sigmoid(item.a * (s.theta  - item.beta));
  const Ps = item.c + (1 - item.c) * sigmoid(item.a * (st.theta - item.beta));
  const K  = cfg.kMin + (cfg.kMax - cfg.kMin) * s.uncertainty;       // kMin 0.12, kMax 0.50
  s.theta  = clamp(s.theta  + o.w * K * (o.x - P), -6, 6);
  st.theta = clamp(st.theta + o.w * cfg.strandShare * K * (o.x - Ps), -6, 6);   // strandShare 0.25
  updateUncertainty(s, o, P);                                         // §4.3
  maybeUpdateItem(item, o, P, s.uncertainty);                        // §2.4
  pushRecent(s, o, P);
}
```

* מיומנות שעוד לא נראתה: `s.theta ← st.theta` בחשיפה הראשונה (היררכיה ציר → מיומנות, בהשראת Elo גלובלי+מקומי של Pelánek).
* חיזוי הצלחה לבחירת פריט: P מאותה נוסחה עם `s.theta`.

### 4.3 לוח K: אי-ודאות דינמית (בהשראת Bolsinova et al. 2026)

```ts
function updateUncertainty(s, o, P) {
  s.uncertainty = Math.max(0, s.uncertainty - 0.05 * o.w);           // ~20 תשובות מ-1 ל-0
  s.driftEwma = 0.8 * s.driftEwma + 0.2 * (o.x - P); s.driftCount++;
  if (Math.abs(s.driftEwma) > 0.25 && s.driftCount >= 6) {          // חיזוי מוטה באופן שיטתי: היכולת השתנתה
    s.uncertainty = Math.min(1, s.uncertainty + 0.15); s.driftCount = 0;
  }
}
function onFirstAttemptAfterGap(s, gapDays) {                         // דמוי Glicko: הפסקה מגדילה אי-ודאות
  if (gapDays >= 3) s.uncertainty = Math.min(1, s.uncertainty + gapDays / 60);
}
```

* ערכי פתיחה: ללא מיקום `u = 1.0` (K = 0.50). אחרי מיקום: 0.4–0.6 (§3.5).
* **ההיגיון:** K שרק יורד עם מספר התשובות נכשל כשהיכולת משתנה מהר, וזה המצב אצל ילדים. לכן K עולה שוב בשני מקרים: כשהשאריות סוטות באופן עקבי, וכשיש הפסקה ארוכה. את הנוסחה המדויקת של המאמר לא הצלחתי לקרוא, וזה תחליף שקוף (יומן מחקר §2).

### 4.4 זמן תגובה לפי גיל וסינון ניחושים

**מדיניות RT** (D11): `rtPolicy = grade < 'g1' ? 'ignore' : 'fluency_only'`. כלומר K3 ו-K2 (כל שנת גן חובה, גם אחרי גיל 6) מתעלמים מ-RT בניקוד. ל-K1 יש RT רק במיומנויות `fluencyEligible`, ורק מרמת `proficient`.

**מדידה:** `responseMs = firstResponseAt − answerableAt`. אפשרויות התשובה פעילות רק מ-`answerableAt` (סוף ההנחיה, או אחרי מילת היעד). לכן "תשובה לפני השאלה" לא יכולה לקרות, וערך שלילי הוא באג (assert).

**ספי הדרה** (בהשראת NT10 של Wise & Kong / NWEA, עם רצפות לגיל):

| | ניחוש מהיר | idle / timeout | רצף ניתוק |
|---|---|---|---|
| K3 | לא רלוונטי (לא מנקדים כישלון) | 20 ש' בלי מגע → הנחיה חוזרת | ≥4 הקשות בשנייה על לא-מטרות → הנחיה עדינה בלבד |
| K2 | `responseMs < max(800, 0.1·median_T)` | 15 ש' → הנחיה חוזרת. ‏30 ש' → timeout (מוחרג) | 3 ניחושים מהירים רצופים → `rapid_guess_burst` |
| K1 | `responseMs < max(600, 0.1·median_T)`. פריטי קריאה: `max(1200, 0.1·median_T)` | 20 ש' → הנחיה עדינה. ‏45 ש' (90 ש' בקטע קריאה) → timeout | אותו דבר |

* `median_T` הוא החציון של הילד עצמו בתבנית (≥10 תצפיות. עד אז רק הרצפה). `template.minPlausibleMs`, אם הוגדר, דורס כלפי מעלה.
* הספים לא עולים על 10 ש' (מקסימום NWEA).
* האפליקציה עברה לרקע → `backgrounded`. הפריט הראשון אחרי חזרה מהשהיה מוחרג.
* `rapid_guess_burst` → ניקוד הבלוק מושהה (`disengaged_block`) והמרכיב מציע "הפסקה או משהו אחר" (§6.7). זה נספר למדד מעורבות, לא ליכולת.

**מדד שטף (K1, מוצג להורה בלבד):** חציון RT על תשובות נכונות בניסיון ראשון, ב-20 האחרונות, במיומנויות `fluencyEligible` מרמת `proficient`. ‏`fluent = median ≤ fluencyTargetMs && accuracy ≥ 0.9`. זה תג, לא רמה.

### 4.5 סבב מהירות (HSHS, ‏K1 בלבד, opt-in)

זה המקום היחיד שבו RT נכנס לעדכון. הכלל מבוסס על Maris & van der Maas 2012, עם זמן מנורמל:

$$S = (2x-1)\left(1 - \frac{\min(t, D)}{D}\right) \in [-1, 1],\qquad E[S] = \coth(\Delta) - \frac{1}{\Delta},\quad \Delta = a(\theta-\beta)$$

```ts
function hshsUpdate(s, item, x: 0 | 1, tMs: number, cfg) {
  const D = item.timeLimitMs!;                                // למשל 12000 לעובדות. אין טיימר מאיים: מוצג "שיא אישי"
  const S = (2 * x - 1) * (1 - Math.min(tMs, D) / D);
  const d = item.a * (s.theta - item.beta);
  const E = Math.abs(d) < 1e-6 ? d / 3 : 1 / Math.tanh(d) - 1 / d;   // פונקציית לנז'בן
  const K = 0.5 * (cfg.kMin + (cfg.kMax - cfg.kMin) * s.uncertainty);  // S בטווח רחב פי 2 מ-x
  s.theta += K * (S - E);
}
```

* **גידור:** `tier==='K1' && skill.fluencyEligible && level ∈ {proficient, master} && plan.speedRoundsEnabled && mode==='speed_round'`.
* בסבב: תשובה ב-`t ≥ D`, או אי-תשובה עד D, נותנת S = 0 ונספרת. אחרי שתי אי-תשובות רצופות הסבב נעצר ("עדיין כאן?"), ושתיהן מוחרגות.
* ההסבר לנוסחת התוחלת (גזירה עצמית): הצפיפות ∝ `exp(Δ·S)` על זמן מנורמל נותנת `P(x=1) = σ(Δ)`, כך שהסולם זהה למודל הדיוק. התוחלת היא נגזרת לוג-פונקציית החלוקה `log(2·sinh Δ / Δ)`.

---

## 5. שליטה, דעיכה וחזרה מרווחת

### 5.1 הגדרת רמות (D9: הגדרה אחת לכל המערכת)

"ניסיון נקי" = `firstTryCorrect && hintsUsed === 0 && !excluded`. ‏`last8` = 8 התצפיות המנוקדות האחרונות של המיומנות. ‏`P̂` = חיזוי ההצלחה על פריט בחציון הקושי של המיומנות.

| רמה | תנאי כניסה |
|---|---|
| `new` חדש | אין תצפית מנוקדת |
| `beginner` מתחיל | ≥1 תצפית מנוקדת |
| `advancing` מתקדם | ≥5 תצפיות, דיוק ניסיון-ראשון ב-last8 (או בכולן אם יש פחות) ≥ 0.70, וגם `P̂ ≥ 0.70` |
| `proficient` שולט | ≥7/8 נקיים ב-last8 (≥85%), **3 האחרונות נקיות**, ה-8 פרושות על **≥2 ימים שונים** (DayKey), וגם `P̂ ≥ 0.80`. פותח את הצאצאים בגרף |
| `master` מאסטר | `proficient` + **בדיקה מושהית** שעברה (§5.2). זה "Mastered" של D9. מכאן מגיעים תג ו-יהלום (D1) |

* K3: הרמות מחושבות פנימית לבחירת פריטים בלבד ולא מוצגות. מוצגות רק מדבקות.
* `attemptsSinceLevelChange ≥ 10 && learningProgress < 0.05 && level ∈ {beginner, advancing}` → `stuck = true` (wheel-spinning, ‏Beck & Gong). המרכיב יורד לבדיקת קדם-דרישה (§6.4), ואם זה נמשך יותר מ-5 מפגשים מוצג להורה כרטיס "תקוע".
* **שער ייצוגים `O.REP`** (03 §17.2, מיומנויות מושגיות במתמטיקה; EEF 2020, concreteness fading): כשב-`skill.extraGates` יש `'O.REP'`, הכניסה ל-`proficient` דורשת **בנוסף** ≥ 2 תצפיות נקיות עם `representation:'pictorial'` **וגם** ≥ 2 עם `'abstract'` בתוך last8. תצפית בלי `representation` (תבנית ישנה) לא נספרת לשער. עד שהשער מתקיים הרמה נשארת `advancing` עם `reasons: ['needs_pictorial' | 'needs_abstract']`, והבורר (§6.5) מעדיף את הייצוג החסר. השער לא חל על `master` (הבדיקה המושהית היא מעורבבת ממילא) ולא על הורדת רמה. `extraGates` נבדקים אחרי כל תנאי §5.1 הרגילים, ב-`packages/engine/mastery/gates.ts`, כדי ש-D9 יישאר הגדרה אחת עם תוספות מוצהרות.

### 5.2 בדיקה מושהית ומעורבבת

```ts
onReachProficient(s, day) { s.proficientDay = day; s.delayedCheck = { eligibleFromDay: addDays(day, 3), asked: 0, correct: 0, status: 'pending' };
                            ensureReviewCard(s.skillId, rung = 0); }
// המרכיב משבץ את הבדיקה רק בבלוק review/boss מעורבב, לא בבלוק שממוקד במיומנות הזו, ורק מ-eligibleFromDay
onDelayedCheckItem(s, cleanCorrect: boolean) {
  const dc = s.delayedCheck!; dc.asked++; if (cleanCorrect) dc.correct++;
  if (dc.correct === 2)                         { dc.status = 'passed'; promote(s, 'master'); }
  else if (dc.asked === 2 && dc.correct === 1)  { /* פריט שלישי שוקל */ }
  else if (dc.asked - dc.correct >= 2)          { dc.status = 'failed'; s.level = 'proficient';
      setRung(s, 0); schedulePractice(s, 2 /*items next session*/); s.delayedCheck = pending(addDays(today, 3)); }
}
```

כך מתקבלת שליטה שנבדקה גם על פני ≥2 ימים בתרגול וגם ≥3 ימים אחריו, בהקשר מעורבב ובלי רמזים (Khan: Mastered רק במבחן מצטבר. ‏D9).

### 5.3 דעיכה והורדת רמה (אף פעם לא ממפגש רע אחד)

* **"צריך חזרה"** (`needsReview`): חזרה שהגיע זמנה נכשלה, **או** `elapsedActiveDays > 1.5 × ladder[rung]`. ימי מנוחה (שבת וחגים, B7) לא נספרים ב-`elapsedActiveDays`.
* **הורדת רמה** (`master → proficient`, ‏`proficient → advancing`) רק אם אחד מאלה:
  (א) שתי חזרות נכשלו ברצף בימים שונים (`consecutiveFailedReviews ≥ 2`).
  (ב) דיוק ניסיון-ראשון ב-last8 < 0.60, כשה-8 פרושות על ≥2 מפגשים.
* אף פעם לא מתחת ל-`beginner`, ולא מחזרה אחרי חופשה בלבד (§10.4).
* הילד לא רואה "ירידה": הצמח "צמא", אף פעם לא נבול (C5).
* הערכת זכירה לצורך עדיפות: `R = 0.9^(elapsedActiveDays / ladder[rung])` (צורת FSRS: R = 0.9 כשהזמן שווה למרווח).

### 5.4 סולם החזרות

| שכבה | `ladder` (ימים) | הערה |
|---|---|---|
| K1 | 1, 3, 7, 14, 30, 60 | D9 |
| K2 | 1, 3, 7, 14, 30 | — |
| K3 | 1, 2, 4, 7, 14 | חזרה בין ימים דרך חזרה על האהובים (Childers & Tomasello) |

```ts
function onReviewResult(card: ReviewCard, first: boolean, second?: boolean, day: DayKey) {
  if (first)                        card.rung = Math.min(card.rung + 1, card.ladder.length - 1);       // עבר
  else if (second === true)         { /* "עבר רך": השלב נשאר */ }
  else { card.rung = Math.max(0, card.rung - (card.ladder[card.rung] >= 30 ? 2 : 1)); card.lapses++; }  // נכשל
  card.lastReviewDay = day; card.dueDay = addActiveDays(day, card.ladder[card.rung]); card.implicitCredit = 0;
}
```

* חזרה = פריט אחד. אם הוא שגוי, מגיע פריט שני מאותה מיומנות.
* `reviewMode:'item'` (אותיות, עובדות, מילים): כרטיס לכל פריט. ‏`'skill'`: כרטיס למיומנות, עם פריט אקראי מבין אלה שלא נראו לאחרונה.
* `dueDay` שנופל על יום מנוחה זז ליום הפעיל הבא.
* FSRS ו-HLR מחוץ להיקף, כי עם שלושה משתמשים הם לא יעלו על Leitner (gap audit §4.4). נוסחת FSRS שמורה ביומן המחקר לעתיד.

### 5.5 קרדיט חזרה עקיף (FIRe-lite)

```ts
function propagateImplicitCredit(k: SkillId, today: DayKey, success: boolean, errorTag?: string) {
  if (success) {
    walkEncompassed(k, /*depth*/ 3, (j, weightProduct) => {     // DFS עם מכפלת משקלים, עוצר אם המכפלה < 0.2
      const card = cardOf(j); if (!card) return;                // רק מיומנויות proficient ומעלה מחזיקות כרטיס
      const elapsed = activeDaysBetween(card.lastReviewDay, today), I = card.ladder[card.rung];
      const credit = weightProduct * (elapsed < 0.5 * I ? 0.25 : 1);   // חזרה מוקדמת שווה פחות (ריווח)
      card.implicitCredit += credit;
      if (card.implicitCredit >= 1) onReviewResult(card, true, undefined, today);   // נחשב כחזרה שעברה. אין העברת עודף
    });
  } else if (errorTag && skill(k).errorTags?.[errorTag]) {
    const j = skill(k).errorTags[errorTag]; markCheckSoon(j);   // פריט חזרה של j במפגש הבא. בלי הורדת רמה
  }
}
```

* מופעל רק על ניסיון נקי במיומנות k.
* **דחיסת חזרות:** בבחירת חזרות (§6.3), חזרה של מיומנות שמכסה הרבה חזרות אחרות שהגיע זמנן מקבלת בונוס עדיפות `+0.3 × Σ weightProduct(due covered)`.
* זו גרסה מפושטת של FIRe של Math Academy, שאין לה הערכה עצמאית. הפונקציות המדויקות לא נקראו (יומן מחקר §5).

### 5.6 BKT במצב צל (לא משפיע)

לכל מיומנות מחושב גם `P(L)` של BKT, ונרשם ביומן להשוואה עתידית בלבד. הפרמטרים: `P(L0)` ממיקום (known 0.8, uncertain 0.4, unknown 0.15), ‏`P(T)=0.15`, ‏`P(G)=min(0.3, c)` (רצפת הניחוש של התבנית), ‏`P(S)=0.1`. הכלל של §5.1 הוא ההגדרה היחידה (D9).

---

## 6. מרכיב הפרקים (Session composer)

### 6.1 תקציב

```ts
function budget(child, plan, today, cfg): { minutes: number; items: number; mode: 'episode' | 'mini' | 'none' } {
  const remaining = plan.dailyCapMin - focusedMinutesToday(child);     // כולל משחק משותף, נספר לשני הילדים
  const ep = cfg.episode[child.tier];                                  // K1 {target 18, min 12}, K2 {10, 6}, K3 {4, 3}
  if (remaining >= ep.min) { const m = Math.min(plan.episodeTargetMin ?? ep.target, remaining);
                             return { minutes: m, items: Math.round(m * child.itemsPerMinute), mode: 'episode' }; }
  if (remaining >= 3 && child.tier !== 'K3') return { minutes: 3, items: 6, mode: 'mini' };  // חימום אחד + ≤4 חזרות + ניצחון
  return { minutes: 0, items: 0, mode: 'none' };                       // מסך "העולם נח", בלי נדנוד (C5)
}
```

* תקרות יומיות כברירת מחדל: K3 15 דק', ‏K2 25 דק', ‏K1 40 דק' (D4). ההורה משנה אותן.
* `itemsPerMinute` הוא EWMA של פריטים מנוקדים לדקה ממוקדת (prior 2.0).

### 6.2 הרכב לפי שכבה

| שכבה | warmup | new (קצה היכולת) | review מעורבבת | stretch | fun / עניין | הערות |
|---|---|---|---|---|---|---|
| K1 | 15% | 40% | 25% | 10% (רק עם `stretchZoneOptIn`, אחרת עובר ל-new) | 10% | ≥2 מקצועות בפרק |
| K2 | 20% | 40% | 20% | 5% (רק אם הצלחת הפרק הקודם ≥ יעד) | 15% | 1 מיומנות חדשה בפרק |
| K3 | — | — | — | — | — | 50% "מוכר ואהוב", 50% "חידוש עדין". אין בלוקים מנוקדים כישלון |

* **שיתוף העניין** (C9: ‏20–30%): חל על **בחירת נושא העטיפה** בכל הבלוקים ועל בלוק ה-fun. ‏K1 25%, ‏K2 30%, ‏K3 50% (שם זה "החידוש העדין"). זה לא חל על רמת הקושי.
* **התאמות:** אין חזרות שהגיע זמנן → נתח ה-review עובר ל-new. הפרק הקודם הסתיים בפחות מ-יעד−0.10 → stretch = 0, ‏new −10%, ‏warmup +10%. פרק חזרה מחופשה → §10.4.
* **עיגול:** largest remainder. שוויון נשבר לפי סדר העמודות.

### 6.3 בניית הפרק

```ts
function composeEpisode(state, childId, ctx): EpisodePlan {
  const c = state.children[childId]; const b = budget(c, c.plan, today(ctx), ctx.config);
  if (b.mode === 'none') return emptyPlan('daily_cap');
  if (b.mode === 'mini') return miniPlan(c);
  const shares = adjustShares(ctx.config.composition[c.profile.tier], c);
  const n = largestRemainder(b.items, shares);

  // מאגרי מועמדים
  const due      = dueReviews(c, today).map(r => ({ r, pr: reviewPriority(r, c) }));   // (1−R)·(1+boost) + 0.3·coverage + checkSoon
  const ready    = fringe(c).concat(inProgress(c));                                     // §3.5 + beginner/advancing
  const focal    = pickFocalSkills(ready, c, ctx, c.profile.tier === 'K1' ? 2 : 1);     // §6.4
  const warmPool = skillsWithP(c, 0.90).filter(k => isFavoriteOrRecent(k));
  const stretch  = c.profile.tier === 'K1' ? nextFringeOrHarder(focal, c, 0.65) : nextFringeOrHarder(focal, c, 0.75);

  // בלוקים (בסדר הזה)
  const blocks: Block[] = [];
  blocks.push(block('warmup', warmPool, n.warmup, P = 0.93));
  for (const f of focal) blocks.push(block('new', [f], share(n.new, focal, f), targetP(c), scaffoldFor(f)));
  blocks.push(...interleavedReview(due, n.review, c));          // ≥2 מיומנויות, רק advancing ומעלה, זוגות הבחנה קודם
  if (n.stretch) blocks.push(block('stretch', stretch, n.stretch, c.profile.tier === 'K1' ? 0.68 : 0.75));
  blocks.push(block('fun', [], n.fun, NaN /*לא מנוקד או קל*/));
  blocks.push(block('closing', warmPool, 1, 0.95));             // תמיד מסיימים בהצלחה
  insertDelayedChecks(blocks, c, today);                        // §5.2, בתוך review/boss בלבד
  interleaveSubjects(blocks, c.plan.subjectWeights);            // §6.8
  assignWrappers(blocks, c.interests, ctx);                     // §7.4
  return { childId, minutes: b.minutes, itemBudget: b.items, blocks, targetP: targetP(c), reasons };
}
```

* `scaffoldFor(f)`: מיומנות שמוצגת לראשונה → `model_guided_independent` (דוגמה, ‏2–3 פריטים מודרכים, ואז עצמאי. Kalyuga). אחרת `independent`.
* **חוקי "הדף"**: ≤10 פריטים לבלוק. שני הראשונים בבלוק new קלים. פריט חדש שונה מקודמו **בתכונה אחת בלבד** (§6.5). כל 3–4 בלוקים יש חזרה מעורבבת.
* **ניקוד:** אסור שאותה מיומנות תופיע בשני בלוקים רצופים, חוץ ממבוא. זמן חגיגות ומעברים ≤15% מזמן הפרק (gap audit §4.5). הנתון נמדד ב-`stats.celebrationMs`.

### 6.4 בחירת מיומנויות "חדשות" (ZPDES-lite + עדיפויות)

```ts
priority(k) = 1.0·ready(k)
            + boost(k)                                   // §6.9 "מה למדו החודש"
            + 0.3·monthAlign(k)                          // monthHint ≈ החודש הנוכחי ב-yearScaffold
            + 0.5·learningProgress(k)                    // LP = |mean(x, 4 אחרונות) − mean(x, 4 שלפניהן)|
            − 0.4·[stuck(k)]
            + 0.2·[subjectUnderweighted(k.subject)]      // לפי subjectWeights ב-14 הימים האחרונים
pickFocalSkills: בהסתברות ε = 0.2 → בחירה אחידה מתוך ready (חקר).
                 אחרת → דגימה פרופורציונלית ל-max(priority, 0.05)^2. ‏rng זרוע.
stuck(k) ⇒ במקום k: בדיקת קדם-דרישה (הקדם-דרישה עם u הגבוה ביותר, או זו שתג הטעות מצביע עליה).
           ALEKS: יורדים צומת, ממלאים פער, חוזרים.
```

### 6.5 בורר הפריטים ובקר רצועת ההצלחה

```ts
function pickItem(skill, block, c, rt, rng): Item {
  const pTarget = clamp(block.targetP + c.bandController.adj, block.targetP - 0.07, block.targetP + 0.07);
  const cands = itemsOf(skill).filter(i => i.reviewed && !seenInLast(i, 10) && i.id !== rt.prevItemId);
  // 0) O.REP: אם השער ממתין (§5.1) ויש מועמד בייצוג החסר עם |P_i − pTarget| ≤ 0.12, מצמצמים אליו
  // 1) |P_i − pTarget| מינימלי; 2) שוני בתכונה אחת לכל היותר מהפריט הקודם של המיומנות; 3) הכי פחות נראה לאחרונה; 4) rng
  const need = missingRepresentation(skill, c);                // 'pictorial' | 'abstract' | undefined
  const pool = need && cands.some(i => i.representation === need && Math.abs(P(i) - pTarget) <= 0.12) ? cands.filter(i => i.representation === need) : cands;
  return lexMin(pool, i => [round2(Math.abs(P(i) - pTarget)), featureDiff(i, rt.prevOfSkill) > 1 ? 1 : 0, lastSeen(i), rng()]);
}
// הלולאה האיטית: בקר רצועת ההצלחה, אחרי כל פרק
function updateBandController(bc, sessionFirstTry: number[], target: number) {
  for (const x of sessionFirstTry) { bc.ewmaSuccess = bc.n === 0 ? x : 0.9 * bc.ewmaSuccess + 0.1 * x; bc.n++; }   // ~20 פריטים
  if (bc.n >= 10) bc.adj = clamp(bc.adj + 0.5 * (target - bc.ewmaSuccess), -0.07, 0.07);
}
```

הבקר מתקן הטיה שיטתית (למשל priors שגויים לתבנית שלמה). הלולאה המהירה היא שומר התסכול (§6.7).

### 6.6 יעדי הצלחה (C2, ‏D3)

| שכבה | `targetP` | חוגה |
|---|---|---|
| K3 | 0.92 | — (אין מצב כישלון. "שגוי" = הנפשה שמראה את הדרך) |
| K2 | 0.85 | — |
| K1 | 0.80 | `easy` 0.85, ‏`just_right` 0.80, ‏`hard` 0.75. בלוקי Stretch Zone ‏0.65–0.70 (opt-in) |

### 6.7 שומר התסכול (D10) וגלאי התסכול

**השומר הדטרמיניסטי** (בכל פריט; `consec` = שגיאות ניסיון-ראשון רצופות במיומנות הנוכחית, `consecAny` = בכל מיומנות):

| מצב | K3 | K2 | K1 | K1 בבלוק stretch |
|---|---|---|---|---|
| שגיאה 1 | הנפשה מראה את הדרך, פריט דומה | אותו דבר | אותו דבר + ניסיון תיקון עצמי אחד | אותו דבר |
| `consec = 2` | רמז + צעד למטה | רמז + צעד למטה: `pTarget = min(0.97, targetP+0.10)` וייצוג אחד מוחשי יותר (CPA) | אותו דבר | רמז בלבד |
| `consec = 3` | **מעבר** לפריט אהוב (P ≥ 0.95) | **מעבר** לפריט מוכר וקל (P ≥ 0.95) | דוגמה פתורה ← פריט כמעט זהה (w = 0.5) | הצעת בחירה: "לנסות שוב / לעבור" |
| `consec = 4` | — | — | מעבר פעילות | מעבר פעילות |
| `consecAny = 3` (מתחת לגיל 6) | מעבר לאהוב | מעבר לאהוב | — | — |

* מתחת לגיל 6 אין יותר משלוש החטאות ברצף, בכל מצב.
* אחרי מעבר, המיומנות לא חוזרת באותו פרק, אלא אם היא בבלוק חזרה.

**ציון תסכול** (חלון של 5 פריטים אחרונים, לא מתאפס בין מיומנויות):

```
+2 לכל שגיאה רצופה מעבר לראשונה      +2 הקשות זעם (≥5 הקשות ב-2 ש' על אזור לא פעיל או על אותה תשובה שגויה)
+1 idle > 10 ש' מיד אחרי שגיאה        +2 ניסיון יציאה ≤10 ש' אחרי שגיאה
+2 בחירת 😠/😢 בצ'ק-אין                 +1 ניחוש מהיר אחרי שגיאה
ציון ≥ 4 → offer_break: נשימת דרקון / הפסקה / מעבר לנושא אהוב (בחירה של הילד). ‏K1 גם "לנסות דרך אחרת".
```

הצלחה אחרי הפסקה רושמת `recovery`. זה מדווח להורה כחוזקה: "חזרה ליצירה אחרי רגעים קשים, 8 פעמים החודש" (§8). ‏`rapid_guess_burst` מפעיל את אותה הצעה.

### 6.8 שזירה בין מקצועות

* K1, פרק של ≥15 דק': ≥2 מקצועות. בלוקים עוקבים מתחלפים במקצוע כשאפשר. המבוא של מיומנות חדשה נשאר רציף.
* K2: ≥2 מקצועות בתוך עטיפת סיפור אחת (למשל ספירה במסע לחלל, ואז צליל פותח של "חללית").
* בלוק review מערבב ≥2 מיומנויות מרמת `advancing` ומעלה בלבד (שזירה רק אחרי ~70%, Brunmair & Richter). זוגות `discriminationPairs` (ב/כ, ד/ר, +/− בבעיות מילוליות) מקבלים עדיפות.
* **משקלי מקצועות ברירת מחדל** (`EngineConfig.subjectWeights`, ההורה משנה ב-10 §6.13 בטווח 10–60 לכל מקצוע פעיל, סכום 100):

| | חשבון | עברית | אנגלית | אחר (06+07) |
|---|---|---|---|---|
| K1 | 30 | 30 | **25** | 15 |
| K2 | 30 | 35 | **15** | 20 |
| K3 | אין משקלים: 50% "מוכר ואהוב" / 50% "חידוש עדין" (§6.2) | | | |

  **ההכרעה (בקשת 05 §17, שורה 08):** 05 ביקש להעלות אנגלית ל-25% (K1) ו-15% (K2), כי כמות הקלט היא המנבא העיקרי של רכישת שפה זרה, ו-15 דק' בשבוע לא מצטברות לכלום. קיבלתי, עם שני סייגים: (א) **עברית ≥ אנגלית תמיד** (B1, Hebrew-first), ולכן ב-K2 העלייה באנגלית באה מהחשבון (35→30) ולא מהעברית, כי מודעות פונולוגית וצלילי אותיות לפני כיתה א' הם המנבא החזק ביותר לקריאה (Yinon et al. 2026, B6), ותחושת המספר של K2 כבר חזקה לפי השאלון; ב-K1 הירידה מתחלקת שווה (35→30 בשני המקצועות). (ב) **לפני פתיחת "האי האנגלי"** (05 §15.1: פרק 6, ~20.12; עד אז שלב "בקבוקים מהים") משקל האנגלית מוגבל ל-**5** (הבקבוקים), והיתרה מתחלקת פרופורציונלית בין שאר המקצועות. תוצאה צפויה ל-K1: ~3 פרקים × 18 דק' × 25% ≈ 14 דק' אנגלית בשבוע באפליקציה, פחות מה-35–40 ש-05 רוצה; את ההפרש 05 מספק משמע ושגרות מחוץ למסך, כפי שכתב. **שיר TPR באנגלית ב-K2 נספר כפול** (00 fable-issues-F1 #11): גם ל-`english` וגם ל-`music`/`motor` בחשבון `subjectUnderweighted` (§6.4), וממלא גם את בלוק ה-fun (§6.2), כך שהאנגלית לא "גונבת" מהמשחק החופשי ומהתנועה. 00 J1: המשקלים הם של 08 (`EngineConfig`), ניתנים לעריכת הורה (H4).
* **מקצוע כבוי** (ההורה כיבה ב-PA-CSET): המשקל שלו 0, והיתרה מתחלקת פרופורציונלית. מקצוע עם `contentExhausted` בכל הצירים (§10.5.9): כנ"ל, עם הודעה להורה.
* **תת-משקל:** `subjectUnderweighted(s)` ב-§6.4 מתקיים כשהחלק של s ב-14 הימים האחרונים קטן מ-`subjectWeights[s] − 10` נקודות.

### 6.9 "מה למדו החודש"

| `phase` | אפקט |
|---|---|
| `upcoming` (שבועיים לפני) | הוראה מקדימה: קדם-דרישות מתחת ל-`proficient` +0.4. המיומנות עצמה +0.2 אם מוכנה |
| `teaching` | המיומנות +0.5 אם מוכנה. אחרת קדם-דרישות +0.5 (סגירת פער קודם) |
| `taught` | עדיפות החזרה ×1.5, ושליפה בתוך 1–3 ימים |

* `expiresAt = setAt + 42 ימים` אלא אם חודש. ההעדפה לא דורסת את הגרף: מיומנות לא מוכנה אף פעם לא מוצגת ישירות.

### 6.10 תקרה יומית וסיום

* לא חותכים באמצע פריט. כשעברו 85% מזמן הפרק, עוברים לבלוק `closing`.
* אם נגמרה התקרה בזמן הסיום: עד 60 ש' חסד, ואז סיום חם (תיבה, סיכום, "נתראה מחר").
* בלי autoplay ובלי "עוד אחד" (C3). פרקים נוספים באותו יום מותרים רק בתוך התקרה.
* **שבת וחגים (00 B7, מצב משפחתי `soft` | `full` | `off`, ברירת מחדל `soft`, OPD-04):** בחלון [הדלקת נרות − 30 דק', הבדלה + 30 דק'] לפי `@hebcal/core` ועיר המשפחה:
  * `soft`: המנוע **כן** מרכיב פרקים (תרגול חופשי), ו-θ/u/רמות **כן** מתעדכנים (ראיה היא ראיה). **לא** מתוזמנים: בוס, בנצ'מרק, בדיקה מושהית (`insertDelayedChecks` מדלג), אתגר דילוג. היום לא נספר ב-`elapsedActiveDays` ולא ב-`availableDays` (02 §10.1), `dueDay` שנופל בו זז (§5.4). מטבעות/יהלומים: 0 (02). הודעות: אין (10 Q-2).
  * `full`: האפליקציה ישנה לילדים; המנוע לא נקרא. יום מנוחה לכל דבר.
  * `off`: יום רגיל.
  * **ימים שקטים (B7a):** יום רגיל למנוע (נספר ב-`elapsedActiveDays`), **אבל** בלי בוס (§10.2), בלי אתגר דילוג, ובלי חגיגות מעל T1 (C14; ה-`EpisodePlan.reasons` מסמן `quiet_day` כדי שה-UI ידע).

---

## 7. מודל העניין

### 7.1 אותות ומשקלים

| אות | משקל |
|---|---|
| בחירה חופשית מ-≥3 אפשרויות (מיקום אקראי) | +3 |
| בחירה מ-2 אפשרויות | +1.5 |
| חזרה לא מונחית ביום אחר | +3 |
| חזרה מונחית | +1 |
| "אני תוהה" / "ספר לי עוד" | +2 |
| חקירת תוספות אופציונליות (≥20 ש') | +2 |
| "עוד מזה" (K1) | +3, ונחשב אישור מפורש |
| 👍 | +1 |
| שהייה אחרי סיום ≥20 ש' | +1 |
| עזיבה מוקדמת | ‎−1 (‎−0.5 אם מיד אחרי שגיאה: זה קושי, לא חוסר עניין) |
| 👎 | ‎−2 |
| "לא מעניין אותי" (K1) | ‎−4, ונושא בהשהיה 14 יום ל-exploit בלבד |
| זרע הורה | התחלה s = 3, ‏`source:'parent'` |
| תיקון הורה | גובר על כל היסק |

**הטיית מיקום:** אם ≥80% מ-10 הבחירות האחרונות היו באותו מיקום על המסך, משקלי בחירה ×0.33 עד שהדפוס נשבר.

### 7.2 עדכון ודעיכה

```ts
decay(t):  s ← s · 2^(−Δdays/25)                        // זמן מחצית 25 יום (3–4 שבועות). זרע הורה: 50 יום
add(w):    s ← s + w;  peak ← max(peak, s)
interest:  I = 1 − exp(−max(s,0)/5)  ∈ [0,1)
confidence: 'low' (<2 ימי ראיה) | 'medium' (2–3) | 'high' (≥4 ב-28 יום)
```

### 7.3 שלבים (Hidi & Renninger)

| שלב | תנאי |
|---|---|
| `unexplored` | אין חשיפה |
| `sparked` (נדלק) | ≥1 אות חיובי |
| `sustained` (נמשך) | ראיה חיובית ב-≥3 ימים שונים ב-21 הימים האחרונים |
| `emerging_passion` (תשוקה מתפתחת) | ראיה **ביוזמת הילד** ב-≥3 ימים שונים ב-14 יום **וגם** ≥1 אישור מפורש (בחירת תמונה "זה או זה", "עוד מזה" או אישור הורה) |
| `dormant` (רדום) | היה לפחות `sparked`, אין ראיה 28 יום, ו-`s < 0.25·peak` |

* "עניין אישי מפותח" לא מוסק. ההורה יכול לסמן אותו (`parentOverride:'loves'`).
* K3: רק `favorite` (נבחר ≥3 פעמים ב-≥2 ימים). השלבים לא מוצגים.

**מנגנון לפי שלב:** `sparked` → עוד חשיפה. ‏`sustained` → חיבור למיומנויות ובחירה. ‏`emerging_passion` → הצעת פרויקט + העברה להורה (Renzulli Type III).

### 7.4 בנדיט ועטיפות

```ts
function assignWrapper(slot, im: InterestMap, cfg, rng): { topic: TopicId; exploration: boolean } {
  if (rng() < cfg.explorationShare[tier]) {                     // K1 0.25, K2 0.30, K3 0.50
    const d = argmin(DOMAINS, dom => [exposures28d(im, dom), DOMAIN_ORDER.indexOf(dom)]);   // איזון בין 6 תחומים
    return { topic: argmin(topicsOf(d).filter(notSnoozed), t => [t.exposures28d, t.topic]), exploration: true };
  }
  const pool = topics(im).filter(t => (t.phase !== 'unexplored' && t.phase !== 'dormant') || t.parentOverride === 'loves');
  return { topic: argmax(pool, t => betaSample(rng, t.alpha, t.beta)), exploration: false };   // Thompson
}
// תגמול, 7 ימים אחרי חשיפה כעטיפה: r = 0.7·reengaged + 0.3·learnedWell
//   reengaged = בחירה או חזרה וולונטרית, 👍, "עוד מזה" או תוספת אופציונלית
//   learnedWell = הצלחת ניסיון-ראשון בבלוקים העטופים ≥ targetP
// α += r; β += 1 − r; דעיכה: α ← 1 + (α−1)·2^(−Δ/25), אותו דבר ל-β
```

* **תגמול על למידה וחזרה וולונטרית, אף פעם לא על זמן מסך** (ZPDES משתמש בהתקדמות כתגמול. אופטימיזציה לזמן היא דפוס פסול).
* **ארכיטקטורת בחירה:** כשמוצעת בחירה, יש 3 אפשרויות: ‏exploit, ‏"שכן" (אותו תחום, נושא אחר) ו-wildcard (חקר). המיקום אקראי.
* **עומק:** העטיפה משנה את תוכן הבעיה (מדידת דינוזאורים), לא רק החלפת שם. נושאי העניין החזקים עוטפים דווקא מקצועות שבהם העניין נמוך (Høgheim & Reber).

### 7.5 מגדר: אסור

* `InterestMap`, `assignWrapper` ו-`composeEpisode` מקבלים `EngineChildView` בלבד. אין בו `grammaticalGender` (אכיפה ב-type, ובדיקה U-INT-6).
* ביקורת חודשית: לכל ילד, כל אחד מ-6 התחומים מקבל ≥10% מחריצי החקר ב-28 יום. כשל מוצג בלוג המפתח.

---

## 8. פרופיל הילד

### 8.1 נטיות תלויות-הקשר (לא תכונות)

ההקשר הוא `subject × activityKind` (למשל "חשבון/בנייה"), ויש גם ערך כללי.

| נטייה | הגדרה | גיל |
|---|---|---|
| `persistence_after_error` | שיעור השגיאות שאחריהן ניסיון נוסף או המשך לפריט הבא בלי יציאה, מתוך כל השגיאות | 4+ |
| `help_seeking` | התפלגות בקשות רמז: `immediate` (< 5 ש' מ-answerable, בלי ניסיון), `after_try` (≥1 ניסיון), `none_struggle` (≥2 שגיאות כשרמז זמין ולא נבקש). מוצג הסגנון השולט אם ≥50% | 5+ |
| `challenge_choice` | שיעור בחירת "חריף" כשמוצע, ודיוק בבחירה הזו | 5+ |
| `novelty_approach` | שיעור קבלת wildcard / פעילות חדשה | 3+ |
| `curiosity` | "אני תוהה" לכל מפגש | 4+ |
| `recovery` | אפיזודות תסכול שאחריהן חזרה והצלחה באותו מפגש, מתוך כל האפיזודות | 4+ |
| `session_rhythm` | שעות מועדפות, הדקה שבה הדיוק מתחיל לרדת | 5+ |

**אומדן עם כיווץ** (shrinkage) לבסיס של הילד עצמו, לא לנורמה:

```
overall = (k + m·prior)/(n + m),          prior = 0.7, m = 10
context = (k_c + m·overall)/(n_c + m),    m = 10
trend   = השוואה בין שני חצאי חלון של 30 יום: ↑ / → / ↓ (|Δ| ≥ 0.1)
```

### 8.2 ספי ראיה לפני הצגה

| תווית | תנאי |
|---|---|
| `hidden` | < 20 אירועים רלוונטיים, או < 5 מפגשים, או < 3 ימים |
| `early` ("תצפית ראשונה") | ≥20 אירועים ב-≥5 מפגשים ב-≥3 ימים |
| `consistent` ("דפוס עקבי") | ≥40 אירועים על פני ≥3 שבועות, ואותו כיוון בשני חצאי החלון |

**מה מוצג:**

* להורה מוצגות רק חוזקות: ערך מעל הבסיס של הילד באותו הקשר, או מעל 0.6. בתוספת "הרפתקה הבאה" אחת, בניסוח חיובי.
* ניסוח: "ב{הקשר} שמנו לב ש…", אף פעם "{name} הוא/היא…".
* לילד מוצג רק עמוד "הכוחות שלי".
* K3: אין נטיות מוצגות.
* כל פריט ניתן לתיקון ("תקנו אותנו"). התיקון גובר על ההיסק ומפחית את משקל האות הזה ב-50% למשך 60 יום.
* **אימות:** ב-8 השבועות הראשונים ההורה מדרג כל תצפית (👍/👎). מבנה שמקבל פחות מ-70% "מדויק" מוסתר.

### 8.3 כיול EF אופציונלי (Tier B, דורש opt-in)

| משחק | גיל | תדירות | פלט פנימי |
|---|---|---|---|
| "לבבות ופרחים" (inhibition/flexibility) | 5+ | כל 1–3 חודשים | דיוק בבלוק המעורב < 0.6 → בלי מכניקת go/no-go במשימות למידה, ו-`distractorDensity:'low'` |
| מיון קלפים DCCS | 4+ | אותו דבר | כישלון במעבר → `rulesPerActivity = 1` |
| הקשות קורסי (זיכרון עבודה) | 5+ | אותו דבר | טווח ≤2 → `instructionMaxSteps = 1`. ‏3 → 2. ‏≥4 → 3 |

* **אף פעם לא מוצג כמספר.** להורה מוצג רק "התאמנו את אורך ההוראות".
* הכלים בהשראת Hearts & Flowers, ‏DCCS ו-Corsi. אין להם תכונות פסיכומטריות ידועות.

### 8.4 קלט לנרטיב החוזקות (חודשי, להורה)

```ts
export interface StrengthsNarrativeInput {
  childToken: '{CHILD}';                       // השם ומגדר הניסוח מוזרקים מקומית אחרי היצירה. לא נשלחים לשרת
  month: string; tier: Tier; consentNarrative: true;
  highlights: { kind: 'mastered' | 'recovery' | 'curiosity' | 'project' | 'challenge'; detailHe: string; dayKey: DayKey }[]; // ≤3 רגעים קונקרטיים
  masteredThisMonth: { skillTitleHe: string; subject: SubjectId }[];
  interests: { topicHe: string; phase: InterestPhase; confidence: 'low' | 'medium' | 'high' }[];  // ≤3
  dispositions: { key: string; contextHe: string; label: 'early' | 'consistent'; trend: '↑' | '→' | '↓' }[]; // חוזקות בלבד
  growingEdge?: { skillTitleHe: string; nextStepHe: string };
  adjustmentsMade: string[];                   // "קיצרנו את ההוראות", "הוספנו הפסקות תנועה"
  suggestions: { renzulliType: 'I' | 'II' | 'III'; topicHe: string }[];
  conversationSeeds: string[];
  constraints: { bannedPhrases: string[]; noComparisons: true; noScores: true; maxWords: 180 };
}
```

* הנרטיב נוצר בצד השרת (Haiku, ‏E8) מהקלט המצטבר בלבד.
* הוא עובר ולידציית סכמה, בדיקת ביטויים אסורים (C4: "גאון", "אלוף", "חכם") והחלפת אסימונים מקומית.
* דגלים (§9), כיול EF וצ'ק-אין רגשי **לא נכנסים** לקלט הזה.

---

## 9. דגלים מוקדמים ("אזורים לתשומת לב")

### 9.1 עקרונות

* **להורה בלבד**, מאחורי שער הורה. הילד לא רואה, לא שומע ולא מקבל רמז לכך.
* **לא אבחון** (C8). לכל הודעה מצורפת הסתייגות.
* **דפוס מתמשך** של ≥3 שבועות **אחרי תרגול ממוקד** (בהיגיון של RTI), ואף פעם לא ממפגש אחד.
* **מדרג:** `none → watch` (פנימי: המנוע מתאים את עצמו, אין הודעה) `→ parent_note` (הודעת תצפית) `→ referral_suggested` (הצעת פנייה). שלב ההפניה דורש ≥8 שבועות והיקף תרגול מינימלי.
* **קצב:** לכל היותר הודעת דגל אחת לילד ב-4 שבועות. לא בהתראת push, רק באזור "לתשומת לב" ובדוח החודשי. ההורה יכול לבטל או להשהות. כיבוי `consent.earlyFlags` עוצר הכול.
* **ספים זמניים.** אין נורמות ישראליות מאומתות. כל ספי המספר הם `provisional: true` ב-config, ולכל כלל מצורפות שאלות תצפית להורה. רגישות ותקפות לא ידועות: מסנני גן בחשבון תופסים רק כ-52% (PMC3935894).

### 9.2 הכללים

| ID | שכבה | פעיל מ- | סמנים וספים | `parent_note` | `referral_suggested` |
|---|---|---|---|---|---|
| `HEB_K2_LETTERS_PA` | K2 | 1 בפברואר | צלילי אותיות ב-`proficient`+ < 10/22 **וגם** דיוק צליל-פותח/חרוז < 60% ב-≥20 פריטים | ≥3 שבועות, ≥6 מפגשים עם תוכן אותיות | ≥8 שבועות, ≥12 מפגשים ממוקדים, בלי שיפור (שיפוע θ < 0.1/חודש) |
| `HEB_K2_LISTENING` | K2 | 1 בדצמבר | הבנת הנשמע ≤ 0.5 לוגיט מתחת למיקום הצפוי ב-≥2 בדיקות | ≥4 שבועות | + `parentConcern.language` → קלינאית תקשורת |
| `HEB_K1_RATE` | K1 | מיד | דיוק ≥90% בקריאה מנוקדת **אבל** WCPM לא משתפר (שיפוע ≤0) 6 שבועות **וגם** מתחת לסף זמני (`provisional`, ‏`wcpmFloorPointed`) | 6 שבועות | 10 שבועות → מחנכת, יועצת, אבחון דידקטי |
| `HEB_K1_LETTER_CONFUSION` | K1 | מיד | ≥25% מהשגיאות בהבחנה הן בזוגות דומים (ב/כ, ד/ר/ך, ה/ח/ת, ו/ז/ן, ס/ם) ב-≥30 פריטים | 4 שבועות של תרגול הבחנה | 8 שבועות |
| `HEB_K1_LISTEN_READ_GAP` | K1 | מיד | θ(הבנת הנשמע) − θ(הבנת הנקרא) ≥ 1.0 לוגיט, SE ≤ 0.5 בשניהם | 4 שבועות | 8 שבועות |
| `HEB_K1_SPELLING_NONPHON` | K1 | מיד | ≥30% משגיאות ההכתבה לא פונולוגיות (לא הומופון) ב-≥30 מילים | 4 שבועות | 8 שבועות |
| `MATH_K2_NUMBER_SENSE` | K2 | 1 בינואר | סוביטיזציה 1–4 < 70% **או** השוואת ספרות 1–9 < 75% ב-≥30 פריטים | 3 שבועות | 8 שבועות |
| `MATH_K1_RETRIEVAL` | K1 | מיד | עובדות ×2/×5/×10: דיוק ≥90% אבל חציון RT > 6 ש' (ספירה ולא שליפה) אחרי 6 שבועות תרגול, **או** "ספירת הכול" בפריטי ספירה-ממשיכה | 6 שבועות | 10 שבועות |
| `ATTN_PATTERN` | K1, K2 | מיד | ניחושים מהירים > 20% מהפריטים ב-≥50% מהמפגשים **או** ירידת דיוק בשליש האחרון ≥ 25 נק' ב-≥60% מהמפגשים **או** ≥3 timeouts למפגש | 3 שבועות. ההודעה מתארת דפוס והתאמה, **בלי** שם הפרעה | **רק** אם `parentConcern.attention` |
| `MOTOR_K2_TRACING` | K2 | מיד | סטיית מסלול חציונית מעל סף התבנית ב-≥60% מהניסיונות | 6 שבועות | 10 שבועות |
| `LANG_K3_PARENT` | K3 | שאלון | T1 = `no` **וגם** T2 = `no` בגיל ≥ 2;6 | מיד, הודעה עדינה | אותה הודעה כוללת את מסלול הפנייה |
| `EMO_DISTRESS` | כולם | מיד | ≥3 אפיזודות תסכול בשבוע, 3 שבועות **או** ≥3 יציאות אחרי שגיאה בשבוע | 3 שבועות: על לחץ, הפסקות ומשחק משותף | אם `parentConcern.emotional`: מדריך הדגלים האדומים (C8) |
| `EMO_CHECKIN` (בקשת 10 §6.9; 00 C8a) | K2, K1 | רק עם `consent.checkinNudge` (T4b, ברירת מחדל כבוי) | **מונה מקומי ל-14 יום** (C8a): לכל `dayKey` נשמר רק `{checkins: n, sadOrScared: m}`, בלי התוויות עצמן, ונמחק אחרי 14 יום. הכלל: "עצוב"/"מפחד" ב-**≥3 מתוך 5 ימי הצ'ק-אין האחרונים** בתוך החלון (ימים עם צ'ק-אין, לא ימים קלנדריים). מחושב **במכשיר שבו נעשה הצ'ק-אין** בלבד; יוצא ממנו **רק** `{ruleId, status:'parent_note', dayKey}` (בוליאני אחד, C8a) למכשירי ההורים, מוצפן | **מיד** (זה לא דגל RTI אלא "דחיפה עדינה"): "אולי זה זמן טוב לשיחה" + CL-10. לא נכנס למדרג `watch`. קצב: לכל היותר פעם ב-**14 יום**, ולא נספר במכסת 28 הימים של שאר הדגלים | **אין.** לעולם לא `referral_suggested` מצ'ק-אין לבד. אם במקביל `EMO_DISTRESS` פעיל, ההודעה מאוחדת |
| `MATH_K1_FACT_PACE` (בקשת 03 §14.3) | K1 | מיד, `maxStatus:'watch'` עד אישור ההורה בנקודת הבדיקה (I1) | אחרי ≥10 שבועות של תרגול עובדות: < 30% מעובדות הכפל (`M.FACT.MUL.*`) ב-`proficient`+, **בזמן ש**דיוק במושגי הכפל (`M.MUL.*`) ≥ 85% ב-≥30 פריטים | 10 שבועות, ≥12 מפגשים עם עובדות. **זה קצב, לא קושי:** ההודעה מציעה לשוחח עם המורה על יעד לוח הכפל (03 §16) | **אין** (לעולם לא `referral_suggested`) |
| `MATH_K1_READ_VS_LISTEN` (בקשת 03 §14.3) | K1 | מיד, `maxStatus:'watch'` עד אישור | בבעיות מילוליות: דיוק נמוך ב-≥15 נקודות כש-`audioPrompt` כבוי (קוראת לבד) לעומת מופעל, ≥20 פריטים בכל מצב, ב-28 יום | **אין הודעה משלו.** הסמן מזין את `HEB_K1_LISTEN_READ_GAP` כראיה נוספת (נספר כ"שבוע מתקיים" גם אם ה-θ-gap קטן מ-1.0 אבל ≥ 0.7). המנוע ב-`watch`: בעיות מילוליות עם הקראה כברירת מחדל | דרך `HEB_K1_LISTEN_READ_GAP` בלבד |

**הערה:** פענוח חלש בגיל 5.5 **אינו** דגל, כי פענוח פורמלי מתחיל בכיתה א'.

**`maxStatus`:** לכל כלל יש `maxStatus` ב-config (`'watch' | 'parent_note' | 'referral_suggested'`). כלל שנוסף אחרי הגרסה הראשונה של המסמך מתחיל ב-`'watch'` (המנוע מתאים את עצמו, ההורה לא רואה כלום) עד שההורה מאשר את הנוסח שלו ב-10 PA-FLAG ("כללים חדשים ממתינים לאישור"). האישור נרשם ב-`ParentInput.flag_action` ומעלה את `maxStatus`.

### 9.3 פסאודוקוד

```ts
function evaluateFlags(state, childId, ctx): FlagState[] {
  if (!profile.consent.earlyFlags) return [];
  return RULES.filter(r => r.tiers.includes(tier) && ctx.now >= r.activeFrom(ctx.calendar)).map(r => {
    const weekly = r.weeklyMarkers(state, childId, ctx);            // מערך שבועי: { met: boolean; exposure: number }
    const persisting = trailingWeeks(weekly, w => w.met && w.exposure >= r.minWeeklyExposure);
    const prev = flags[r.id] ?? { status: 'none' };
    let status = persisting >= r.noteWeeks ? 'parent_note' : persisting >= 1 ? 'watch' : 'none';
    if (persisting >= r.referralWeeks && r.referralGate(profile) && cumulativeExposure(weekly) >= r.minExposure)
      status = 'referral_suggested';
    if (prev.status === 'dismissed' && ctx.now < prev.snoozedUntil) status = 'dismissed';
    const notify = rank(status) > rank(prev.status) && daysSince(lastFlagNotice(childId)) >= 28;
    return { ...prev, ruleId: r.id, status, weeksPersisting: persisting, notify, evidence: r.summarize(weekly) };
  });
}
```

* `watch` מפעיל התאמות מיידיות: עוד תרגול ממוקד בסמן, צעדים קטנים יותר ופרקים קצרים יותר (ATTN).
* כשהדפוס נעלם 4 שבועות, הסטטוס יורד ל-`none` בשקט. להורה מוצג "נראה שזה הסתדר".

### 9.4 נוסחי הודעות (עברית, להורה)

כללים: שם הילד במקום כינוי, ניסוח ניטרלי (B3), תצפית קונקרטית, מה האפליקציה עושה, מה אפשר לעשות בבית. אין "סיכון", "הפרעה" או "לקות".

**הסתייגות קבועה (בכל הודעה):**
> "זו תצפית מתוך המשחק, לא אבחון. האפליקציה לא נבדקה מול ילדים בישראל, ולכן יכולות להיות גם התרעות מיותרות וגם דברים שהיא לא תראה. אתם מכירים את {name} הכי טוב."

**`HEB_K2_LETTERS_PA`, ‏parent_note:**
> "בשבועות האחרונים {name} {מתרגל|מתרגלת} צלילי אותיות ומשחקי חרוזים. שמנו לב שזה עדיין לוקח יותר זמן ממה שאנחנו רואים בדרך כלל באמצע גן חובה. כרגע {name} {מזהה|מזהה} בביטחון {n} צלילי אותיות. מה אנחנו עושים: יותר משחקי צלילים קצרים, אות אחת בכל פעם. מה אפשר בבית: לשחק 'באיזה צליל זה מתחיל?' על דברים בבית, 3 דקות ביום."

**`HEB_K2_LETTERS_PA`, ‏referral_suggested:**
> "אחרי חודשיים של תרגול, ההתקדמות בצלילי האותיות עדיין איטית. כדאי לשאול את הגננת מה היא רואה בגן. אם גם שם עולה קושי, אפשר להתייעץ עם קלינאית תקשורת דרך קופת החולים. הערכה מוקדמת בגיל הזה נעשית בעדינות, ויכולה לחסוך תסכול בכיתה א'. אפשר להדפיס סיכום קצר ולהביא לשיחה."

**`HEB_K1_RATE` / `LETTER_CONFUSION` / `LISTEN_READ_GAP` / `SPELLING_NONPHON`, ‏parent_note:**
> "{name} {קורא|קוראת} בדיוק טוב, אבל הקצב עדיין איטי ולא השתפר בשבועות האחרונים." (או: "יש בלבול חוזר בין אותיות דומות כמו ב/כ ו-ד/ר" / "{name} {מבין|מבינה} סיפור שמקריאים הרבה יותר טוב מסיפור שקורא/ת לבד" / "בהכתבה יש טעויות שלא נובעות מאותיות שנשמעות אותו דבר") "מה אנחנו עושים: תרגול קריאה קצר וחוזר, עם ניקוד מלא. מה אפשר בבית: לקרוא יחד 10 דקות, לסירוגין שורה-שורה."

**כנ"ל, ‏referral_suggested:**
> "כדאי לשתף את המחנכת ואת יועצת בית הספר במה שראינו (אפשר להדפיס סיכום). אם גם הן רואות את זה, אבחון דידקטי אצל מאבחנ/ת מוסמכ/ת מטעם משרד החינוך (מגיל 6) יכול לעזור להבין איך הכי נכון ללמד את {name}. אבחון לא מדביק תווית. הוא מסביר מה עוזר."

**`MATH_*`, ‏parent_note:**
> "ב{סוביטיזציה/השוואת מספרים/עובדות כפל} {name} עדיין {סופר|סופרת} אחד-אחד במקום לזהות או לשלוף. זה שלב טבעי בדרך, ולפעמים הוא נמשך זמן. מה אנחנו עושים: משחקי הבזק קצרים וישר מספרים. מה אפשר בבית: משחקי קופסה עם קוביה ומסלול מספרים (נמצא יעיל לגילאי גן)."
> **referral:** "כדאי לשוחח עם {הגננת|המחנכת}. אם הקושי נמשך גם שם, אבחון דידקטי (מגיל 6) יכול להבהיר."

**`ATTN_PATTERN`, ‏parent_note** (בלי שם של הפרעה):
> "שמנו לב שבאמצע הפרקים הריכוז של {name} יורד מהר, ויש יותר תשובות מהירות 'על הדרך'. מה אנחנו עושים: קיצרנו את הפרקים והוספנו הפסקות תנועה. מה אפשר: לשחק בשעה שבה {name} {רענן|רעננה}, ולא לפני השינה."
> **referral (רק אם ההורה סימן דאגה):** "אם גם אתם רואים קושי דומה בבית ובגן או בבית הספר, אפשר לשוחח עם רופא/ת הילדים או עם הצוות החינוכי."

**`MOTOR_K2_TRACING`:**
> "העתקת קווים ואותיות עדיין מאתגרת את {name}. כדאי לשאול את הגננת. אם גם היא רואה את זה, אפשר לבקש הפניה לריפוי בעיסוק דרך רופא/ת הילדים בקופת החולים."

**`LANG_K3_PARENT`:**
> "סימנתם ש{name} עדיין לא {אומר|אומרת} הרבה מילים ולא {מחבר|מחברת} שתי מילים יחד. ילדים מתפתחים בקצב שונה, ובכל זאת בגיל הזה כדאי להזכיר את זה בטיפת חלב או אצל רופא/ת הילדים. אם צריך, הם יפנו לקלינאית תקשורת. בינתיים: לדבר ולתאר הרבה, לשיר ולקרוא יחד."

**`EMO_DISTRESS`:**
> "בשבועות האחרונים {name} {מתוסכל|מתוסכלת} לא מעט כשמשהו לא מצליח. התאמנו קושי והוספנו כלי הרגעה. מה אפשר: לשחק לפעמים יחד, ולשים לב שהמשחק לא מרגיש כמו מבחן. אם התסכול מופיע גם מחוץ למשחק ומפריע ביום-יום, היועצת בבית הספר או בגן היא כתובת טובה." + קישור למדריך הדגלים האדומים (C8). מספרי קווי סיוע (ער"ן, נט"ל) מאומתים לפני שחרור.

### 9.5 הדפסה

לכל `parent_note` ו-`referral_suggested` יש "סיכום להדפסה": התצפיות (בלי מספרי θ), תקופה, היקף תרגול, מה נעשה, וההסתייגות. מיועד לשיחה עם הגננת או המחנכת.

### 9.6 פרטיות

* `FlagState`, ‏`DispositionMap`, ‏`EFCalibration` וצ'ק-אין רגשי מחושבים ונשמרים במכשיר.
* מסונכרנים רק למכשירי ההורים, מוצפנים (E6). לא נשלחים לאף ספק AI.
* נמחקים עם איפוס פרופיל. הנתונים "רגישים במיוחד" לפי תיקון 13.
* **ייצוא (10 PRIV-03):** `FlagState` נכלל בייצוא רק כשההורה מסמן "כולל 'לתשומת לב'" (ברירת מחדל כבוי), ואז כ-JSON נפרד עם ההסתייגות. `emotion_checkin` לא נכלל בייצוא לעולם (הוא במכשיר הילד, לא אצל ההורה).

### 9.7 התרעות שווא ותיקון הורה

הרגישות והסגוליות של הכללים לא ידועות (§9.1), ולכן ההתנהגות אחרי "לא רלוונטי" חשובה יותר מהספים עצמם:

| פעולת הורה (10 PA-FLAG) | אפקט במנוע |
|---|---|
| **"לא רלוונטי"** + סיבה (`notRelevant`) | הכלל לילד הזה עובר ל-`dismissed` ל-**8 שבועות** (`snoozedUntil`), **ו**הסף של הכלל לילד הזה מחמיר ב-20% (למשל `< 10/22` → `< 8/22`; `≥25%` → `≥30%`) לשארית השנה. הסיבה נשמרת ב-`FlagState.calibration[]` ומוצגת למפתח (לא להורה). שתי דחיות של אותו כלל → הכלל כבוי לילד עד rollover (§10.5.7), ומוצג ב-PA-FLAG כ"כיביתם" עם אפשרות להדליק |
| **"דיברנו עם המורה"** (`talkedToTeacher`) | נרשם בראיות; הכלל לא מתקדם ל-`referral_suggested` ב-4 השבועות הבאים (נותנים זמן לתשובת המורה) |
| **"להשהות 4 שבועות"** (`snooze`) | `dismissed` ל-28 יום, בלי שינוי ספים |
| **הסתייגות מתוכנית** (`external_evidence` עם הערת מורה "בסדר גמור") | כמו `notRelevant` אבל בלי החמרת סף, ומוצג "המורה רואה תמונה אחרת" |
| כיבוי `consent.earlyFlags` | הכול נעצר, ו-`FlagState` נמחק במכשירים תוך 24 ש' |

**שני סמנים בלתי תלויים:** כלל שמוגדר עם "וגם" (למשל `HEB_K2_LETTERS_PA`) דורש את שניהם בכל שבוע שנספר. כלל עם "או" דורש שהסמן **אותו סמן** יתקיים ברצף השבועות (לא לסירוגין). זה מוריד התרעות שווא מרעש.

**הטיית חשיפה:** `minWeeklyExposure` (מספר פריטים רלוונטיים בשבוע) ברירת מחדל 15. שבוע עם פחות מזה לא נספר ולא שובר את הרצף (`persisting` לא מתאפס, רק לא עולה). ילד שמשחק מעט לא יקבל דגל רק מזה שהוא משחק מעט.

---

## 10. קצב הערכה מחדש

### 10.1 שוטף

* כל פריט מנוקד מעדכן את המצב (§4).
* בפועל נאספים 10–15 פריטים מנוקדים בשבוע לכל מקצוע, בתוך המשחק (בנוסח IXL). אם נאספו פחות מ-8 בשבוע במקצוע פעיל, המרכיב מעלה את משקלו.

### 10.2 בוס חודשי

* **טריגר:** עברו ≥28 ימים מהבוס הקודם במקצוע, **או** ≥3 מיומנויות הגיעו ל-`proficient` מאז. לא ביום מנוחה (B7, גם ב-`soft`), ולא ביום שקט (B7a). אחד למקצוע בחודש לכל היותר. יום הבוס המשפחתי המועדף: חמישי (OPD-07), ולכן כשהטריגר מתקיים הבוס ממתין ליום הזה אם הוא בתוך 6 ימים.
* **הרכב:** K2 6 פריטים, ‏K1 10 פריטים. 50% מיומנויות שממתינות לבדיקה מושהית (proficient ≥3 ימים), 30% חזרות בעדיפות גבוהה ומיומנויות שנשלטו לאחרונה, 20% "הצצה" לשוליים (מנוקד, לא נספר לבדיקה המושהית).
* **בלי רמזים** ("קשה לי" מותר).
* **מסגור:** קרב בוס. סרגל החיים של הבוס יורד עם תשובות נכונות **ומאמץ**. הבוס תמיד "נסוג" בסוף, ואין הפסד. ניצחון → יהלום (D1).
* **עוגן נגד סחיפה:** אם `mean(x − P)` על פריטי הבוס < ‎−0.15 (≥8 פריטים), אז `u += 0.2` למיומנויות המעורבות ובקר הרצועה מעודכן. זו בדיקת אמת שבה הילד מצליח פחות מהתחזית.

### 10.3 שלושה חלונות בנצ'מרק (לוח בתי הספר בישראל)

| חלון | תאריכים (תשפ"ז) | K1 | K2 | K3 |
|---|---|---|---|---|
| BOY | 15.10–30.11 (אחרי סוכות. השנה = מבחן המיקום) | מיקום מלא | מיקום מלא | שאלון |
| MOY | 15.1–28.2 | קריאה בקול: רשימת מילים 45 ש' + טקסט קצר מנוקד, ונוסף **אותו טקסט לא מנוקד** (B5, מבנה כלי משרד החינוך); חשבון 12 פריטים מעורבבים; אנגלית 10 שמע-והקש | צלילי אותיות (כל 22 + 5 סופיות, בלי זמן, בשני מפגשים), מודעות פונולוגית 8, תחושת מספר 10 | שאלון (ינואר) |
| EOY | 15.5–15.6 | כמו MOY, מנוקד + לא מנוקד | כמו MOY + "מוכנים לכיתה א'" (יוני–יולי) | שאלון (אפריל) + (יולי) |

* האלגוריתם הוא אלגוריתם המיקום (§3.4), עם `prior = { mu: θ_s, sd: max(0.5, se_s) }` ומקסימום פריטים מוקטן (K1 8, ‏K2 6 לציר).
* **פלט:** "מאז החלון הקודם" (Δθ), וסטטוס מול התוכנית: `θ_s − SCALE·pos ≥ +0.5` = "מקדים", `≥ −0.5` = "בדרך הנכונה", אחרת "בבנייה". מוצג רק אם SE ≤ 0.5. אחרת מוצג טווח.
* **אין אחוזונים ואין השוואה לילדים אחרים.**
* WCPM מדווח רק לטקסט מנוקד עד MOY (gap audit §4.9). ‏ASR רק ל-K1 ולעולם לא כעונש (E9).
* התאריכים הם config ב-`IsraeliSchoolCalendar` (חבילת תוכן), ומאומתים מול לוח משרד החינוך לפני שחרור.

### 10.4 אחרי הפסקה ושינויי קצב

* **חזרה מחופשה:** `daysSinceLastActive ≥ 21` (למשל קיץ), **או** חופשה מלוח בתי הספר ≥14 יום בלי משחק (חנוכה ארוכה, פסח). במקרה כזה:
  * הפרק הראשון הוא `mode:'reentry'`: חימום של 5 פריטים לכל ציר פעיל (≤2 צירים לפרק), עם `u += gap/60`.
  * החזרות מוגבלות ל-30% מהפרק, לפי עדיפות.
  * אין הורדת רמה על סמך הבדיקה הזו לבדה.
* **אתגר דילוג (האצה):** בציר שבו הצלחת ניסיון-ראשון > 95% ב-14 יום (≥30 פריטים) מוצע "אתגר דילוג". זו בדיקה בסגנון מיקום כלפי מעלה מהשוליים (מקס' 8). מיומנויות שעוברות מסומנות `proficient` עם `provenance:'skip_challenge'`, ועדיין דורשות בדיקה מושהית. אין תקרת כיתה.
* **בדיקה כלפי מטה:** הצלחה < 60% במשך 7 ימים (≥20 פריטים) → בדיקת קדם-דרישות בציר (מקס' 6).

### 10.5 מקרי קצה (כללים מחייבים)

#### 10.5.1 מיקום שנקטע באמצע
לפי §3.6 כלל 6: שמירה אחרי כל פריט, חידוש עד 14 יום, התחלה מחדש עם prior מורחב אחרי כן, סגירה כפויה אחרי 21 יום. **בדיקה:** U-PLR-1..4.

#### 10.5.2 ילד שעונה אקראית במיקום
לפי §3.8: `finish('inconsistent')`, אתחול `uncertain`, מסע קצר חוזר פעם אחת. **בדיקה:** U-RAND-1..3.

#### 10.5.3 אח ששיחק בפרופיל של אח (זיהוי ותיקון)
* **זיהוי (היוריסטי, בסיום כל מפגש, `anomaly.identity_suspect`).** ציון = סכום:
  * +2 אם `firstTryRate` של המפגש (≥10 פריטים מנוקדים) רחוק מ-`ewmaSuccess` של הילד ב-≥0.35 **לכל כיוון**.
  * +2 אם `mean(x − P)` במפגש ≥ +0.35 או ≤ −0.35 (≥10 פריטים): הילד "הרבה יותר טוב/גרוע מהתחזית" בבת אחת.
  * +1 אם חציון `responseMs` במפגש נמוך מ-0.5× או גבוה מ-2× מחציון הילד (≥10 תצפיות, אחרי סינון RT).
  * +1 אם המפגש החל תוך 3 דקות מ-`session.ended{reason:'parent_exit'|'inactivity'}` של **אח אחר** באותו מכשיר (11 §4.16).
  * +1 אם בחירות הנושא במפגש כולן מחוץ ל-5 הנושאים המובילים של הילד (≥3 בחירות).
  * ציון ≥ 4 → `identity_suspect`. ציון ≥ 3 במפגש **ראשון אחרי מיקום** לא נספר (ירח דבש משנה את ההתנהגות).
* **מה המנוע עושה לבד:** במפגש חשוד, העדכונים **כן נכנסים** (אין ראיה מספיקה לחסום), אבל `u` של המיומנויות המעורבות לא יורד (K לא קטן), כדי שמפגש אחד לא "ינעל" θ שגוי. אותות עניין מהמפגש נכנסים במשקל ×0.5. דגלים (§9) לא סופרים את המפגש הזה בשבוע. **לא נשלחת התראה לילד**, ולא נעשה שום דבר גלוי בזמן המשחק (C5).
* **כלי התיקון להורה (10 PA-LOG, W-LOG-05):** בשורת המפגש "לא {name} {שיחק|שיחקה}?" → בחירה: אח אחר / "לא יודע" → `ParentInput.session_reattribute {sessionId, to}`. המנוע: (1) `Session.attributedTo = to`; (2) כל ה-`Attempt` וה-`BehaviorSignal` של המפגש מוחרגים (`excluded:'reattributed'`) **אצל הילד המקורי**; (3) חישוב מחדש מנקודת הביקורת שלפני המפגש (§1.11), כך ש-θ, u, רמות, כרטיסי חזרה, עניין ונטיות חוזרים למצב "כאילו המפגש לא קרה" (זה מה ש-event sourcing נותן בחינם); (4) אם `to` הוא אח, האירועים **לא** מועברים אליו אוטומטית (ה-DAG, הרמות והקושי שלו שונים; הניסיונות על פריטים שלא מתאימים לו היו מטים את ה-Elo שלו). הם נשארים ביומן תחת `attributedTo` לצורך שקיפות בלבד. (5) `skill.mastered` ו-`skill.level_changed` שנפלטו במפגש מקבלים אירוע `skill.level_reverted` (02 **לא** לוקח חזרה מטבעות/תגים, C5; 02 §4.1 `badgeNeverDowngrades`). (6) ההחלטה הפוכה אפשרית ("בעצם כן {name}") עד 30 יום; שני הקלטים ביומן, האחרון לפי hlc קובע.
* **מניעה:** תמונת-PIN (H3, ברירת מחדל ל-K1); הצעה אוטומטית להורה להפעיל תמונת-PIN גם ל-K2 אחרי 2 מפגשים `identity_suspect` ב-14 יום (10 AL-15).

#### 10.5.4 נתונים משני מכשירים שסותרים
* **אירועים לא סותרים** (11 §6.5): אותו ילד ששיחק על שני מכשירים (גם במקביל, גם offline) נותן שני רצפי אירועים, והמצב הוא הקיפול שלהם בסדר הקנוני `(ts, deviceId, eventId)`. שני המכשירים מגיעים לאותו `EngineState` אחרי סנכרון (S-3).
* **מפגשים חופפים בזמן** (שני מכשירים, אותו ילד, אותו רגע): שניהם תקפים. התקרה היומית נספרת על שניהם (11 §4.16, חריגה offline נסלחת). חזרה שהגיע זמנה ותורגלה בשני המכשירים: הראשונה בסדר הקנוני מעדכנת את הכרטיס, והשנייה היא "חזרה מוקדמת" (§5.5, קרדיט ×0.25). פריט שהוצג בשניהם: שני הניסיונות נספרים (הם ניסיונות אמיתיים).
* **`ParentInput`/docs סותרים:** last-writer-wins לפי hlc (E6), שניהם ביומן.
* **אירוע שמגיע באיחור** (ts ישן מנקודת הביקורת): חישוב מחדש (§1.11). אם זה קורה באמצע מפגש פעיל, המנוע מסיים את הפריט הנוכחי, מחשב מחדש ברקע, וממשיך עם `EpisodeRuntime` הקיים (התוכנית לא נבנית מחדש באמצע פרק).
* **אירוע כפול** (אותו `id` משני מכשירים אחרי שחזור): `reduce` אידמפוטנטי לפי `id` (קבוצת מזהים שנראו ב-checkpoint).

#### 10.5.5 סטיית שעון
* `dayKey` מחושב **במכשיר בזמן האירוע** מ-`ts + tzOffsetMin` (11 §5.7), ונשמר באירוע. המנוע לא מחשב אותו מחדש. לכן אירוע ממכשיר עם שעון שגוי נכנס ליום השגוי, אבל **באופן עקבי** בכל המכשירים.
* הסדר הקנוני לפי `ts` נשאר גם אם הוא "שקרי". אין ניסיון לתקן ts של אירועים (הם עובדות).
* **הגנות:** (א) `gapDays` ו-`elapsedActiveDays` מחושבים על DayKey ומוגבלים ל-`max(0, …)`: שעון שחזר אחורה לא יוצר "הפסקה שלילית" ולא מעלה u. (ב) `onFirstAttemptAfterGap` לא מופעל אם `gapDays > 400` (שעון מקולקל, לא חופשה) אלא רושם `clock_anomaly` בטלמטריה. (ג) `dueDay` בעתיד הרחוק (> 90 יום מ-`max(dayKey)` שנראה) נחתך ל-90. (ד) חלונות בנצ'מרק ו-`activeFrom` של דגלים מחושבים מ-`ctx.now` של המכשיר שמריץ, ולא מאירועים. (ה) אם 11 מדווח `|skew| > 2 min`, המנוע מקבל `ctx.now = serverTime + (localNow − localAtSync)` (זמן שרת מתוקן), כך שהחלטות "היום" עקביות עם הדוחות ב-Worker.
* **בדיקה:** U-SKEW-1..3.

#### 10.5.6 איפוס על ידי הורה (10 FAM-03, PRIV-04)
| פעולה | מה מתאפס | מה נשאר |
|---|---|---|
| **"איפוס מודל הפרופיל"** (`reset_profile_model`) | `InterestMap` (חוץ מזרעי הורה ו-`parentOverride`), `DispositionMap`, `EFCalibration`, `FlagState` (כולל `calibration`), צ'ק-אין רגשי במכשירים | θ, רמות, כרטיסי חזרה, מיקום, כלכלה |
| **"איפוס התקדמות"** (`reset_progress`, לפי מקצוע או הכול; Owner בלבד, step-up) | `SkillState`, `StrandState`, `ReviewSchedule`, `PlacementStatus` של המקצועות שנבחרו, `bandController`, `itemsPerMinute` (אם הכול). ממומש כהעלאת `modelEpoch` (או `subjectEpoch[s]`): הקיפול מתעלם מ-`Attempt`/`BehaviorSignal` של המקצוע עם epoch קודם. **האירועים לא נמחקים** (היסטוריה, ייצוא, 11 §5.9) | כלכלה (02: מטבעות, תגים, חילוצים לא נלקחים, C5), עניין ונטיות (אלא אם גם איפוס פרופיל), `ItemCalibration` (משותף ל-3 הילדים, לא שייך לילד) |
| **"מחיקת ילד"** (10 PRIV-04, רכה, 30 יום) | המנוע מסיר את `children[childId]` מהמצב ומנקודות הביקורת בכל מכשיר מיד עם קבלת הפקודה; האירועים נמחקים ב-D1 אחרי 30 יום (11 §7.3). ביטול בתוך 30 יום → קיפול מחדש מאפס | — |
| **ביטול איפוס** | אין. איפוס התקדמות הוא בלתי הפיך ברמת המודל (האירועים נשמרים, אבל `modelEpoch` לא יורד). ההודעה להורה אומרת זאת, ומוצעת "הורדת רמה ידנית" למיומנות בודדת (W-SK-10) כחלופה עדינה |

אחרי `reset_progress` המקצוע חוזר ל-`placement.state='not_started'` ומסע מיקום חדש מתוזמן (עם prior מהשאלון, שלא התאפס).

#### 10.5.7 עליית כיתה (rollover, 1.9.2027 ולכל שנה)
* ב-**1 בספטמבר** (`calendar.schoolStart` של השנה החדשה) האפליקציה מציעה להורה ב-FAM-03 "עלה/תה כיתה?" עם הצעה (`grade+1`; `gan_chova → g1`; `gan_trom → gan_chova`; `toddler` לפי גיל ≥ 3;0 → `gan_trom`). **עד שההורה מאשר**, המנוע ממשיך עם ה-`grade` הישן ו-`pos` נעצר על 1.0 (§2.1). אין rollover אוטומטי (A5: ההורה יודע; ילד יכול להישאר שנה נוספת בגן).
* אישור → `ParentInput.grade_rollover`. המנוע:
  1. `profile.grade`, `profile.schoolYear` ← חדשים. `tier` **לא** משתנה (K1/K2/K3 הם מזהי פרופיל, A2), אבל כל הכללים שתלויים ב-`grade` (למשל `rtPolicy = grade < 'g1' ? 'ignore' : 'fluency_only'`) מתעדכנים: **K2 בכיתה א' מקבל `rtPolicy:'fluency_only'`** ואת סולם החזרות של K2 (הסולם לפי `tier`, לא לפי כיתה: 1/3/7/14/30; מעבר ל-60 יום רק כש-`tier` משתנה, וזה לא קורה).
  2. `Plan.yearScaffold` ו-`monthHint` נבנים מחדש מה-`gradeTag` החדש. `ClassTopic`-ים ישנים פגים (`expiresAt = now`).
  3. `IsraeliSchoolCalendar` של השנה החדשה (חבילת תוכן; ⟂VERIFY מול לוח משרד החינוך תשפ"ח, שמתפרסם בדרך כלל ביוני).
  4. **מיקום חוזר:** לא מסע מלא. הפרק הראשון אחרי הקיץ הוא `reentry` (§10.4, כי `daysSinceLastActive ≥ 21` כמעט בטוח), ובנוסף **"מסע קצר"** (`maxItems = minItems`) לכל ציר `primary` שבו `θ_s − SCALE·pos_new < −0.8` או `> +0.8`, כדי לכייל מול התוכנית החדשה. התאריכים: חלון BOY של השנה החדשה.
  5. `anchor` של מיומנויות **לא משתנה** (הוא תלוי תוכנית, לא ילד). רמות וכרטיסי חזרה נשמרים. דגלים: `weeksPersisting` מתאפס (שנה חדשה, כיתה חדשה), `calibration` נשמר, `activeFrom` מחושב מהלוח החדש.
  6. `ContentIndex` חייב להכיל מיומנויות ל-`gradeTag` החדש (K1 → `g4`: רצועת S+ של 03). אם אין → `contentExhausted` (§10.5.9) והודעה למפתח.
* **ילד שעלה כיתה באמצע השנה** (מעבר בית ספר, הקפצה): אותו מסלול, בלי התניה לספטמבר. **ילד שנשאר שנה נוספת:** ההורה דוחה את ההצעה, `schoolYear` מתעדכן, `grade` לא.
* **בדיקה:** U-ROLL-1..4.

#### 10.5.8 היעדרות ארוכה
* 21–89 יום: §10.4 (`reentry`).
* **≥ 90 יום** (למשל K2 שהפסיק באביב וחזר בספטמבר): `reentry` + "מסע קצר" לכל ציר `primary` (כמו ב-rollover). `u ← min(1, u + 1.0)` (כלומר 1.0). כרטיסי חזרה: כולם `needsReview`, ה-rung יורד ל-`max(0, rung − 2)` **בלי** הורדת רמה, ולא יותר מ-30% מהפרק לחזרות (כמו 10.4) למשך 5 פרקים. דגלים: `weeksPersisting` מתאפס.
* **ילד שההורה "השהה"** (10 AL-05 מזכיר "ילד מושהה"): `profile.paused: true` → המנוע לא מתזמן, דגלים ובנצ'מרק לא רצים, ימי ההשהיה לא נספרים ב-`elapsedActiveDays` (כמו ימי מנוחה). חזרה מהשהיה = כמו היעדרות לפי אורכה.

#### 10.5.9 לומד מהיר מאוד שמגיע לתקרה
* `ceilingHit` במיקום (§3.4) → המסע הבא ממשיך כלפי מעלה עם `prior = posterior`, בלי תקרת כיתה. אתגר דילוג (§10.4) ממשיך משם.
* **תוכן נגמר בציר** (אין מיומנות `new`/`beginner` שמוכנה, וכל מה שקיים ≥ `proficient`): `contentExhausted[strand] = today`. המרכיב: (א) חלק ה-new של הציר עובר לצירים אחרים באותו מקצוע; (ב) אם כל המקצוע מוצה → המשקל שלו מתחלק (§6.8) ובלוק ה-fun מקבל פעילויות העשרה `MP.X` (03 §3.2) ופרויקטים (§7.3, Renzulli III); (ג) להורה ב-PA-MAP: "הגענו לסוף התוכן שלנו ב-{strand} לשנה הזו. המשחק ממשיך לחזור ולהעמיק." (ד) למפתח: אירוע `content.exhausted` בטלמטריה, כדי להוסיף רצועת S+/G+ (03). **אין** "המצאת" תוכן על ידי LLM כתגובה (C7: תוכן מאושר בלבד).
* **בנצ'מרק לילד כזה:** הסטטוס "מקדים" מוצג רק ב-PA-ASM (10 R14), וה-SE מחושב כרגיל. `skip_challenge` לא מעניק `master` (עדיין בדיקה מושהית).
* **בדיקה:** U-CEIL-1..2.

#### 10.5.10 ייצוא ומחיקה (10 PRIV-03/04)
* ייצוא: `EngineState.children[childId]` מיוצא כ-JSON (schema `engine-state.v1`), בלי `ItemCalibration` (משותף) ובלי `FlagState` אלא אם סומן (§9.6). מצורף `levels.ts` mapping כדי שה-JSON קריא. הקיפול ניתן לשחזור מהאירועים בלבד, ולכן ה-JSON הוא נוחות, לא מקור.
* מחיקת משפחה: המנוע לא עושה דבר מיוחד; המכשירים מקבלים `wipe` (11 S5).

#### 10.5.11 הורים חולקים
* שני `ParentInput` סותרים → hlc (§1.9א). **חריג:** `reset_progress` ו-`child.delete` דורשים Owner (10 §2), ולכן אין סתירה. `lock`/`unlock` על מיומנות: האחרון קובע; ב-PA-MAP מוצג "שונה לאחרונה על ידי {guardian}" (10 RB-5).
* `questionnaire` שמולא על ידי שני ההורים (שניהם ענו): התשובות ממוזגות **לפי שאלה**: `unsure` נדחה מפני תשובה; `yes` ו-`no` סותרים → `partly`. זה נרשם ב-`questionnaire_mismatch` ומוצג ב-W-ASM-03 כ"סימנתם תשובות שונות".

#### 10.5.12 מעט מאוד נתונים (שבוע ראשון)
* לפני מיקום: `composeEpisode` מחזיר `EpisodePlan` עם `mode:'placement'` בלבד (אין פרק רגיל לפני שהמקצוע הראשון מופה, חוץ מ-K3). ההורה רואה ES-0 (10 §5.4).
* אחרי מיקום, עד 8 תצפיות למיומנות: `P̂` ו-FTA לא מוצגים (10 PD-P4); `level` כן (הוא מוגדר מהתצפית הראשונה). `reasons[]` של כל בלוק מסבירים "עדיין מכיילים".
* `bandController` לא פעיל עד `n ≥ 10` (§6.5), ו-`itemsPerMinute` הוא ה-prior 2.0 → תקציב הפריטים בשבוע הראשון עלול להיות גדול מדי: לכן ב-3 הפרקים הראשונים `itemBudget = round(0.8 × minutes × itemsPerMinute)` (פחות פריטים, יותר חגיגה), ו-`closing` מגיע ב-80% מהזמן במקום 85%.

---

## 11. פרמטרים (`EngineConfig`, ברירות מחדל)

```ts
export const DEFAULT_CONFIG: EngineConfig = {
  scale: 1.0,                                  // לוגיט לשנת-כיתה
  grid: { min: -4, max: 4, step: 0.05 },
  placement: {
    K1: { minItems: 6, maxItems: 12, subjectMax: 25, seStop: 0.45, offset: 0.35, priorSd: 0.8, maxJump: 1.5 },
    K2: { minItems: 4, maxItems: 8,  subjectMax: 14, seStop: 0.45, offset: 0.55, priorSd: 1.0, maxJump: 1.5 },
    questionnaireWeight: 0.7, tooTrickyWeight: 0.5, knownP: 0.85, unknownP: 0.40, mismatchLogit: 1.0,
  },
  elo: { kMin: 0.12, kMax: 0.50, strandShare: 0.25, uPerObs: 0.05, gapDaysToU: 60, driftAlpha: 0.2, driftThreshold: 0.25, driftMinObs: 6, driftBump: 0.15,
         kItem0: 0.04, kItemHalfN: 50, itemBound: 0.6, kTemplate: 0.01, templateBound: 0.5, itemUpdateMaxU: 0.5, hshsKFactor: 0.5 },
  rt: { K1: { floorMs: 600, readingFloorMs: 1200, idlePromptMs: 20000, timeoutMs: 45000, readingTimeoutMs: 90000 },
        K2: { floorMs: 800, idlePromptMs: 15000, timeoutMs: 30000 }, nt: 0.10, capMs: 10000, burst: 3, minObsForMedian: 10 },
  mastery: { advancingAcc: 0.70, advancingP: 0.70, proficientClean: 7, window: 8, lastClean: 3, minDays: 2, proficientP: 0.80, delayedDays: 3,
             demoteAcc: 0.60, stuckAttempts: 10, stuckLp: 0.05 },
  ladders: { K1: [1, 3, 7, 14, 30, 60], K2: [1, 3, 7, 14, 30], K3: [1, 2, 4, 7, 14] }, needsReviewFactor: 1.5,
  fire: { maxDepth: 3, minProduct: 0.2, earlyFactor: 0.25, earlyFraction: 0.5, coverageBonus: 0.3 },
  episode: { K1: { target: 18, min: 12 }, K2: { target: 10, min: 6 }, K3: { target: 4, min: 3 } },
  dailyCap: { K1: 40, K2: 25, K3: 15 },
  composition: { K1: { warmup: .15, new: .40, review: .25, stretch: .10, fun: .10 },
                 K2: { warmup: .20, new: .40, review: .20, stretch: .05, fun: .15 } },
  explorationShare: { K1: 0.25, K2: 0.30, K3: 0.50 }, skillExplorationEps: 0.2,
  targetP: { K3: 0.92, K2: 0.85, K1: { easy: 0.85, just_right: 0.80, hard: 0.75 }, stretchK1: 0.68 },
  band: { gain: 0.5, bound: 0.07, ewma: 0.1, minN: 10 },
  frustration: { scoreThreshold: 4, window: 5, idleAfterErrorMs: 10000, rageTaps: 5, rageWindowMs: 2000 },
  interest: { halfLifeDays: 25, parentSeedHalfLifeDays: 50, positionBias: 0.8, positionBiasFactor: 0.33, snoozeDays: 14 },
  profile: { shrinkM: 10, prior: 0.7, earlyEvents: 20, earlySessions: 5, earlyDays: 3, consistentEvents: 40, consistentWeeks: 3, hideBelowAccuracy: 0.7 },
  subjectWeights: { K1: { math: 30, hebrew: 30, english: 25, other: 15 }, K2: { math: 30, hebrew: 35, english: 15, other: 20 } },   // §6.8
  englishPreIslandWeight: 5,                   // §6.8: עד פתיחת האי האנגלי (05 §15.1)
  gates: { 'O.REP': { pictorialClean: 2, abstractClean: 2, pickTolerance: 0.12 } },   // §5.1, §6.5
  lazyPlacement: { probeItems: 3, placedAtMinObs: 8, placedAtMaxSe: 0.6 },            // §3.7
  placementResume: { resumeDays: 14, restartSdFloor: 0.7, forceCloseDays: 21, episodesBeforeRestart: 3 },   // §3.6 כלל 6
  inconsistency: { guttmanGap: 1.0, sdStallItems: 4, sdStallDelta: 0.05, fastShare: 0.5, retryAfterDays: 2 },  // §3.8
  anomaly: { rateDelta: 0.35, residualDelta: 0.35, rtRatioLow: 0.5, rtRatioHigh: 2.0, siblingSwitchMin: 3, minScored: 10, threshold: 4, interestWeight: 0.5 },  // §10.5.3
  flags: { minNoticeGapDays: 28, minWeeklyExposure: 15, notRelevantSnoozeWeeks: 8, notRelevantTighten: 0.2, teacherGraceWeeks: 4,
           checkin: { window: 5, hits: 3, cooldownDays: 14 },
           maxStatus: { MATH_K1_FACT_PACE: 'watch', MATH_K1_READ_VS_LISTEN: 'watch', EMO_CHECKIN: 'parent_note' /* השאר: 'referral_suggested' */ },
           provisional: { wcpmFloorPointed: null /* חייב להיקבע מכלי משרד החינוך או ממשוב מורה */ } },
  cadence: { bossEveryDays: 28, bossAfterProficient: 3, reentryGapDays: 21, vacationGapDays: 14, longAbsenceDays: 90, skipChallengeAcc: 0.95, probeDownAcc: 0.60,
             rolloverShortTripDelta: 0.8 },
  clock: { maxGapDays: 400, maxDueAheadDays: 90 },           // §10.5.5
  checkpoints: { everyEvents: 200, keep: 30 },               // §1.11
  firstWeek: { episodes: 3, budgetFactor: 0.8, closingAt: 0.80 },   // §10.5.12
  celebrationMaxShare: 0.15, honeymoonEpisodes: 2,
};
```

---

## 12. בדיקות יחידה (קלט → פלט צפוי)

**פורמט:** Jest. מספרים עם `toBeCloseTo(v, 2)` אלא אם צוין אחרת. וקטורי המיקום חושבו בסקריפט עזר עם אותן נוסחאות (רשת ‎−4..4, צעד 0.05, ‏a = 1.5, ‏c = 0).

### 12.1 סולם ו-priors

| ID | קלט | צפוי |
|---|---|---|
| U-SCL-1 | `pos(K1, 2026-11-02)` | 3.2 |
| U-SCL-2 | תבנית כפל: `baseDelta=0`, ‏`table=×7`, ‏`format=choice3`, ‏`representation=abstract`, ‏`b_k=3.4`, ‏`τ=0` | β = 3.4 + 0.6 − 0.3 = 3.7, ‏c = 1/3 |
| U-SCL-3 | δ מצטבר = 2.1 | clamp ל-1.5 |

### 12.2 Elo ו-HSHS

| ID | קלט | צפוי |
|---|---|---|
| U-ELO-1 | θ=0, β=0, a=1.5, c=0, u=1, x=1, w=1 | P=0.5, K=0.50, θ'=0.25, u'=0.95 |
| U-ELO-2 | אותו דבר, c=1/3, x=0 | P=0.6667, θ'=‎−0.3333 |
| U-ELO-3 | נכון אחרי רמז: x=0.3, w=0.5, P=0.5, K=0.5 | Δθ=‎−0.05 |
| U-ELO-4 | ניחוש מהיר (excluded) | θ, u, ε_i ללא שינוי, `excluded:'rapid_guess'` |
| U-ELO-5 | u=0.2, הפסקה של 30 יום | u=0.70 |
| U-ELO-6 | 6 שאריות רצופות של +0.5, driftEwma=0 | EWMA=0.3689 > 0.25 → u += 0.15, driftCount=0 |
| U-ELO-7 | עדכון פריט: n=0, x=1, P=0.5, u_child=0.4 | ε ← ‎−0.02. עם u_child=0.6 → ללא שינוי |
| U-ELO-8 | ε=0.59, x=0, P=0.9, 10 עדכונים | ε ≤ 0.6 תמיד |
| U-ELO-9 | strand: θ_s=0, K=0.5, x=1, P_s=0.5 | θ_s' = 0.0625 |
| U-HSHS-1 | K1, θ=0.5, β=0, a=1.5, u=0.2 (K=0.196, K_h=0.098), x=1, t=3000, D=12000 | S=0.75, E=L(0.75)=0.2411, θ'=0.5499 |
| U-HSHS-2 | אותו דבר, x=0, t=2000 | S=‎−0.8333, θ'=0.3947 |
| U-HSHS-3 | אותו דבר, אי-תשובה עד D | S=0, θ'=0.4764 |
| U-HSHS-4 | Δ=1e‎−9 | E=Δ/3, בלי NaN |
| U-HSHS-5 | K2, כל מיומנות, `mode:'speed_round'` | נדחה (שגיאת גידור). ‏RT לא נכנס |
| U-HSHS-6 | K1, מיומנות לא `fluencyEligible`, או `level='advancing'` | עדכון דיוק בלבד |

### 12.3 סינון RT

| ID | קלט | צפוי |
|---|---|---|
| U-RT-1 | K1, תבנית לא-קריאה, median=4000, responseMs=450 | סף max(600,400)=600 → `rapid_guess` |
| U-RT-2 | K1, פריט קריאה, median=5000, responseMs=900 | סף 1200 → `rapid_guess` |
| U-RT-3 | K2, median=6000, responseMs=900 | סף 800 → מנוקד |
| U-RT-4 | K1, median=150000 (קטע), responseMs=9000 | הסף לא עולה על 10000 → `rapid_guess` |
| U-RT-5 | 3 ניחושים מהירים רצופים | אירוע `rapid_guess_burst`, ‏`disengaged_block` לשאר הבלוק, הצעת הפסקה |
| U-RT-6 | K2, idle 15 ש' / 30 ש' | הנחיה חוזרת / `timeout` מוחרג |
| U-RT-7 | responseMs < 0 | assert נכשל (באג UI) |
| U-RT-8 | K2 עם גיל 6;1 בגן חובה | `rtPolicy='ignore'` |

### 12.4 מיקום

| ID | קלט | צפוי |
|---|---|---|
| U-PL-1 | prior N(0,1), שאלון: `yes` על b=‎−1, ‏`no` על b=0.5 (w=0.7) | μ=‎−0.085, SD=0.813 |
| U-PL-2 | עמוד שדרה b=[‎−2…3] בצעד 0.5; K1 prior N(0,0.8); הילד מצליח אם b ≤ 0.5 | רצף: חימום ‎−2, ‎−2; B ‎−0.5✓, 0.5✓, 1.0✗; R 0✓, 0.5✓×4, 1.0✗. עצירה `SE` אחרי 11, ‏θ̂=0.99±0.03, SE=0.43±0.02. ‏known={‎−2,‎−1.5,‎−1,‎−0.5}, ‏unknown={1.5…3}, ‏uncertain={0,0.5,1} |
| U-PL-3 | אותו דבר, הילד מצליח תמיד | עצירה `cap` אחרי 12, ‏θ̂=2.07, SE=0.46, ‏`ceilingHit=true` |
| U-PL-4 | אותו דבר, מצליח רק אם b ≤ ‎−1.5 | B ‎−0.5✗, ‎−1.5✓, ‎−1✗; R ‎−1✗; **breather** ‎−2✓; R ‎−1✗. עצירה `SE` אחרי 8, ‏θ̂=‎−0.96, SE=0.45. סיווג אחרי ראיה ישירה: ‎−2 known (3 הצלחות, p=0.81 ≥ 0.75) |
| U-PL-5 | K2 prior N(0,1), offset 0.55, מצליח אם b ≤ ‎−0.5 | חימום ‎−2, ‎−2; B ‎−0.5✓, 0.5✗, 0✗; breather ‎−2✓; R ‎−0.5✓, ‎−0.5✓. עצירה `cap` אחרי 8, ‏θ̂=0.15±0.03, SE=0.54±0.02 |
| U-PL-6 | אותו דבר, שני פריטי החימום נכשלים | `lo=-1, hi=w`: החיפוש יורד מתחת |
| U-PL-7 | K2: שתי החטאות → breather → החטאה נוספת, n ≥ min | `finish('frustration')` + פריט ניצחון |
| U-PL-8 | DAG ‏A→B→C. ‏C known, ‏B uncertain | B ← known, ‏A ← known |
| U-PL-9 | C known, ‏B עם 2 כישלונות ישירים | C, B ← uncertain |
| U-PL-10 | A unknown | B, C ← unknown |
| U-PL-11 | עמוד שדרה b=[‎−2, ‏2] בלבד, prev=‎−2, mid=2 | maxJump 1.5 מגביל, ונבחר הצומת הקרוב המותר |
| U-PL-12 | לכל ריצה | הפריט הראשון והאחרון עם P ≥ 0.9. אפס אירועי `feedback:'correctness'` |

### 12.5 שליטה וחזרה

| ID | קלט | צפוי |
|---|---|---|
| U-MST-1 | last8 = [1,1,1,0,1,1,1,1] נקיים, 2 ימים, P̂=0.86 | `proficient` |
| U-MST-2 | אותו דבר, יום אחד | `advancing`, ‏reason `needs_second_day` |
| U-MST-3 | 7/8, אבל הניסיון השלישי מהסוף עם רמז | `advancing` |
| U-MST-4 | proficient ב-D. בדיקה מושהית ב-D+2 | לא משובץ (לא זכאי) |
| U-MST-5 | ב-D+3, בבלוק review מעורבב: 2/2 נקיים | `master` |
| U-MST-6 | בדיקה מושהית: ✓✗✗ | `failed`. נשאר `proficient`, ‏rung=0, בדיקה חדשה מ-היום+3 |
| U-MST-7 | בדיקה מושהית בבלוק `new` של אותה מיומנות | לא נספרת |
| U-MST-8 | master, חזרה נכשלת פעם אחת | נשאר master, ‏`needsReview=true`, ‏rung יורד |
| U-MST-9 | master, שתי חזרות נכשלות ברצף בימים שונים | → proficient |
| U-MST-10 | last8 דיוק 0.5, כולם באותו מפגש | אין הורדה |
| U-MST-11 | 10 ניסיונות, ‏beginner, ‏LP=0.02 | `stuck=true` |
| U-REV-1 | K1, rung 0 עבר | rung 1, ‏due = +3 ימים פעילים |
| U-REV-2 | rung 4 (30) נכשל | rung 2 (7) |
| U-REV-3 | rung 2 נכשל | rung 1 |
| U-REV-4 | rung 2: ✗ ואז ✓ | rung 2 ("עבר רך"), lapses ללא שינוי |
| U-REV-5 | K2 rung 4 (30) עבר | נשאר 4 |
| U-REV-6 | rung 2 (7), ‏lastReview 1.11, היום 20.11, ימי מנוחה 7, 14 (שבתות) | elapsedActive=17 → 17/7=2.43 > 1.5 → `needsReview` |
| U-REV-7 | due שנופל בשבת | זז ליום ראשון |
| U-FIRe-1 | k→j (0.6), ‏j→l (0.5). הצלחה נקייה ב-k. ‏j בפיגור, ‏l בזמן | j += 0.6. ‏l: 0.3 |
| U-FIRe-2 | אותו דבר, j חזר לפני יומיים, I=7 | j += 0.6×0.25 = 0.15 |
| U-FIRe-3 | j.credit = 0.5 + 0.6 | עבר (rung+1), ‏credit=0 |
| U-FIRe-4 | שרשרת 0.6×0.3 = 0.18 | לא מופץ (< 0.2) |
| U-FIRe-5 | כישלון ב-k עם `chosenErrorTag` שממופה ל-j | `j.checkSoon=true`, אין שינוי rung או רמה |

### 12.6 מרכיב הפרקים

| ID | קלט | צפוי |
|---|---|---|
| U-CMP-1 | K2, 10 דק', ‏itemsPerMinute=2 → 20 | {warmup 4, new 8, review 4, stretch 1, fun 3} |
| U-CMP-2 | K1, 18 דק' → 36, ‏stretchZoneOptIn | {5, 14, 9, 4, 4} |
| U-CMP-3 | K2, 20 פריטים, אין חזרות | {4, 12, 0, 1, 3} |
| U-CMP-4 | K2, נותרו 4 דק' (min 6) | `mini`: 6 פריטים (חימום 1, חזרות ≤4, ניצחון 1) |
| U-CMP-5 | נותרו 2 דק' | `none` |
| U-CMP-6 | כל תוכנית | בלוק אחרון `closing` עם targetP ≥ 0.95, ראשון `warmup` |
| U-CMP-7 | K1, 18 דק' | ≥2 מקצועות. אין שני בלוקים רצופים לאותה מיומנות (מלבד מבוא) |
| U-CMP-8 | review: מיומנות ב-`beginner` | לא נכנסת לבלוק מעורבב |
| U-CMP-9 | classTopic `teaching` למיומנות X שהקדם-דרישה שלה ב-`advancing` | X לא מוצגת. הקדם-דרישה מקבלת +0.5 ונבחרת כמוקד |
| U-CMP-10 | classTopic שנקבע לפני 43 יום | ללא בוסט |
| U-CMP-11 | אותו seed ואותו מצב, פעמיים | תוכנית זהה (דטרמיניזם) |
| U-CMP-12 | `bandController`: יעד 0.85, ‏ewma 0.70 | adj=+0.07 (חסום). ‏pTarget=0.92 |
| U-CMP-13 | יעד 0.85, ‏ewma 0.95 | adj=‎−0.05 → pTarget=0.80 |
| U-CMP-14 | `pickItem` | אף פעם לא אותו פריט פעמיים ברצף. מעדיף שוני של תכונה אחת לכל היותר |
| U-FRU-1 | K2: ✗, ✗ | `hint_and_step_down`, ‏pTarget=0.95, ייצוג אחד מוחשי יותר |
| U-FRU-2 | K2: ✗✗✗ | `switch_to_easy_favorite` (P ≥ 0.95) |
| U-FRU-3 | K1: ✗✗✗ / ✗✗✗✗ | `worked_example_then_near_identical` / `switch_activity` |
| U-FRU-4 | K1 stretch: ✗✗✗ | `offer_choice` |
| U-FRU-5 | K2: ✗ במיומנות A, ✗ ב-B, ✗ ב-C | `consecAny=3` → `switch_to_easy_favorite` |
| U-FRU-6 | ✗✗ + rage-tap + idle 12 ש' אחרי שגיאה | ציון 2+2+1=5 ≥ 4 → `offer_break` |
| U-FRU-7 | הצלחה אחרי הפסקה באותו מפגש | אות `recovery` |

### 12.7 עניין

| ID | קלט | צפוי |
|---|---|---|
| U-INT-1 | s=6, אחרי 25 יום | s=3.0, ‏I=0.451 |
| U-INT-2 | בחירה חופשית מ-3 ב-3 ימים שונים בתוך 14 יום + זרע הורה | `emerging_passion` |
| U-INT-3 | אותו דבר בלי אישור מפורש (בלי זרע) | `sustained` |
| U-INT-4 | 9 מתוך 10 בחירות אחרונות במיקום 0 | משקל בחירה 3×0.33=0.99 |
| U-INT-5 | rng → 0.1 < 0.25 (K1) | חקר: התחום עם הכי מעט חשיפות ב-28 יום |
| U-INT-6 | שני מצבים זהים, רק `grammaticalGender` שונה (`f`/`m`) | `composeEpisode` ו-`assignWrapper` מחזירים פלט זהה (בדיקת תכונה, 1000 seeds) |
| U-INT-7 | עזיבה מוקדמת מיד אחרי שגיאה | ‎−0.5 (לא ‎−1) |
| U-INT-8 | "לא מעניין אותי" | נושא לא נבחר ב-exploit 14 יום. בחקר מותר |
| U-INT-9 | תגמול: בחירה וולונטרית תוך 7 ימים, ובלוקים מתחת ליעד | r=0.7 → α+=0.7, ‏β+=0.3 |

### 12.8 פרופיל

| ID | קלט | צפוי |
|---|---|---|
| U-PRF-1 | 19 שגיאות ב-6 מפגשים | `hidden` |
| U-PRF-2 | 22 שגיאות, 5 מפגשים, 3 ימים | `early` |
| U-PRF-3 | context: k_c=8, n_c=10, overall=0.6 | (8+6)/20 = 0.70 |
| U-PRF-4 | נטייה מתחת לבסיס הילד ומתחת ל-0.6 | לא מוצגת כחוזקה. לכל היותר "הרפתקה הבאה" |
| U-PRF-5 | קורסי טווח 2 | `instructionMaxSteps=1`. אין שדה מספרי ב-DTO להורה |
| U-PRF-6 | `buildNarrativeInput` | אין שדות flags, ‏EF, ‏emotion, ‏displayName, ‏gender |

### 12.9 דגלים

| ID | קלט | צפוי |
|---|---|---|
| U-FLG-1 | K2, 1 במרץ: 8/22 אותיות proficient, ‏PA 55% (24 פריטים), 4 שבועות × ≥6 מפגשים | `parent_note` |
| U-FLG-2 | אותו דבר ב-1 בדצמבר | `none` (לא פעיל) |
| U-FLG-3 | אותו דבר, 8 שבועות, 12 מפגשים ממוקדים, שיפוע 0.05 | `referral_suggested` |
| U-FLG-4 | דפוס בשבוע אחד בלבד | `watch` (בלי הודעה) |
| U-FLG-5 | ATTN מתקיים 3 שבועות, ‏`parentConcern.attention=false` | `parent_note`, לעולם לא `referral_suggested` |
| U-FLG-6 | דגל שני תוך 20 יום מהקודם | `notify=false` |
| U-FLG-7 | `consent.earlyFlags=false` | `[]` |
| U-FLG-8 | K3 T1=no, ‏T2=no, גיל 2;7 | `LANG_K3_PARENT` → הודעה |
| U-FLG-9 | כל מחרוזת הודעה | אין "סיכון", "הפרעה", ‏"ADHD", "לקות", "דיסלקציה". יש הסתייגות. מוצגת בגרסת m/f נכונה |
| U-FLG-10 | הדפוס נעלם 4 שבועות | `none` + "נראה שזה הסתדר" |

### 12.10 קצב

| ID | קלט | צפוי |
|---|---|---|
| U-CAD-1 | 2027-01-20 | חלון MOY |
| U-CAD-2 | 2027-03-10 | אין חלון |
| U-CAD-3 | פעילות אחרונה 2026-12-01, היום 2027-01-05 | `reentry`, ‏u += 35/60 לכל המיומנויות הפעילות, חזרות ≤30% |
| U-CAD-4 | 3 מיומנויות הגיעו ל-proficient מאז הבוס, 10 ימים | בוס זמין |
| U-CAD-5 | בוס: mean(x−P) = ‎−0.2 על 10 פריטים | u += 0.2 במיומנויות המעורבות |
| U-CAD-6 | 32 פריטים, 96% ב-14 יום | הצעת אתגר דילוג |

### 12.10א מיפוי רמות (`packages/shared/levels.ts`)

| ID | קלט | צפוי |
|---|---|---|
| U-LVL-1 | `new`, `ready=false` | `locked` |
| U-LVL-2 | `new`, `ready=true`, `seen=false` | `ready` |
| U-LVL-2b | `new`, `ready=true`, `seen=true` | `introduced` |
| U-LVL-2c | `beginner` / `advancing` | `practicing` (שניהם) |
| U-LVL-3 | `master`, rung ימים 14 | `mastered` |
| U-LVL-4 | `master`, rung ימים 30 | `retained` |
| U-LVL-5 | `master` → הורדה ל-`proficient` (§5.3) | `toEconomyState` = `proficient`; 02 שומר `maxStateEver='mastered'` (בדיקה ב-02) |
| U-LVL-6 | `fromEconomyState` | `locked`/`ready` → null; `introduced` → `new`; `practicing` → `beginner`; זהה לטבלת C15 (snapshot test) |
| U-BND-1 | `grade='gan_chova'` | `bandOf='G'`; אחרי rollover ל-`g1` → `'S'` |
| U-BND-2 | ילד G, מיומנות `band:'S'` עם כל הקדם-דרישות proficient | **כן** ב-fringe (S = G+1, מותר). ילד T עם מיומנות `band:'S'` → לא ב-fringe (S = T+2) |
| U-BND-3 | ילד S, מיומנות `band:'T'` | לא כמוקד `new`; מותרת רק ב-warmup/fun |

### 12.10ב שכבות-על ממקצוע, צירי המשך, משקלים

| ID | קלט | צפוי |
|---|---|---|
| U-STEP-1 | `O.STEP`: steps {total 3, correct 2}, ניסיון ראשון בלי רמז | x = 0.667, w = 1, `firstTryCorrect=false`, לא נספר כנקי |
| U-STEP-2 | אותו דבר, correct 3 | x = 1, נקי |
| U-STEP-3 | θ=0, β=0, a=1.5, u=1, x=0.667 | Δθ = 0.5·(0.667−0.5) = +0.0833 |
| U-STEP-4 | תיקון עצמי: ניסיון 2 עם 3/3 | x = 0.5 |
| U-STEP-5 | `firstErrorTag='E.NO_CARRY'` ממופה ל-j | `j.checkSoon=true` |
| U-REP-1 | `O.REP`: last8 = 7/8 נקיים, 2 ימים, P̂=0.86, ייצוגים: 6 pictorial + 2 concrete | נשאר `advancing`, reason `needs_abstract` |
| U-REP-2 | אותו דבר + 2 abstract נקיים (במקום 2 pictorial) | `proficient` |
| U-REP-3 | `pickItem` כשחסר abstract ויש מועמד abstract עם \|P−pTarget\| = 0.10 | נבחר abstract |
| U-REP-4 | כנ"ל, המועמד הקרוב ביותר באבסטרקט ב-0.20 | הבחירה הרגילה (לא כופים קושי לא מתאים) |
| U-REP-5 | חבילה עם `O.REP` ובלי תבניות abstract | הטעינה נדחית (`ContentIndex` validator) |
| U-REP-6 | מיומנות בלי `extraGates` | §5.1 כרגיל (רגרסיה: U-MST-1 עובר) |
| U-LAZY-1 | ציר `continuation`, מיומנות נבחרת לראשונה | 3 פריטי probe בתחילת בלוק new, K=0.50, משוב רגיל |
| U-LAZY-2 | 3/3 נכונים | המיומנות `known`; הקדם-דרישות `known` (סגירה) |
| U-LAZY-3 | 8 תצפיות, se=0.55 | `placedAt` נקבע |
| U-WGT-1 | K1, ברירת מחדל | {math 30, hebrew 30, english 25, other 15} |
| U-WGT-2 | K1 לפני פתיחת האי (2026-12-01) | english 5; השאר פרופורציונלי: math 37, hebrew 37, other 21 (עיגול largest remainder, סכום 100) |
| U-WGT-3 | K2, אנגלית כבויה | {math 35.3→35, hebrew 41, other 24} (largest remainder) |
| U-WGT-4 | הורה קובע hebrew 20, english 40 | נדחה בוולידציה של 10 (עברית ≥ אנגלית); המנוע מקבל רק config תקין (assert) |

### 12.10ג מקרי קצה (§10.5)

| ID | קלט | צפוי |
|---|---|---|
| U-PLR-1 | ציר נקטע אחרי 3 פריטים, חזרה אחרי 5 ימים | ממשיך מ-`resumable`, פריט חימום אחד, `obs.length` ממשיך מ-3 |
| U-PLR-2 | נקטע, חזרה אחרי 15 יום | התחלה מחדש, prior = N(mean, max(sd, 0.7)) |
| U-PLR-3 | נקטע אחרי `minItems`, חזרה | `finish('interrupted')`, לא נפתח שוב |
| U-PLR-4 | מסע פתוח 22 יום | `done`, `quality:'low'`, צירים לא נבדקים = continuation |
| U-PLR-5 | `resumable` נבנה מחדש מאירועי `placement.step` במכשיר אחר | זהה בתים-לבתים |
| U-RAND-1 | ✓ ב-b=1.5, ✓ ב-b=1.0, ✗ ב-b=−0.5, ✗ ב-b=0 אחרי minItems | `finish('inconsistent')`, se ≥ 0.7, כל המיומנויות `uncertain`, u=0.8 |
| U-RAND-2 | 60% מהפריטים מתחת לרצפת RT (לא ברצף) | `inconsistent` |
| U-RAND-3 | מסע קצר חוזר אחרי יומיים גם `inconsistent` | `quality:'low'`, אין מסע שלישי |
| U-ANM-1 | ילד עם ewma 0.80, מפגש 12 פריטים ב-0.33, חציון RT 0.4× | ציון 2+2+1=5 → `identity_suspect`; u לא יורד במיומנויות המפגש |
| U-ANM-2 | מפגש ראשון אחרי מיקום, ציון 3 | אין anomaly |
| U-ANM-3 | `session_reattribute` למפגש שהעלה מיומנות ל-`proficient` | אחרי חישוב מחדש: הרמה כפי שהייתה לפני המפגש; אירוע `skill.level_reverted`; θ זהה למצב "בלי המפגש" (בתים-לבתים מול fold ללא המפגש) |
| U-ANM-4 | reattribute ל-K2, ואז "בעצם כן" אחרי 10 ימים | שני הקלטים ביומן; המצב הסופי שווה למצב בלי reattribute |
| U-ANM-5 | מפגש reattributed | אותות עניין ודגלים לא סופרים אותו |
| U-SYNC-1 | שני מכשירים, אותו ילד, מפגשים חופפים, אותה חזרה בשניהם | הראשון בסדר הקנוני מעדכן rung; השני קרדיט ×0.25 |
| U-SYNC-2 | אירוע כפול (אותו id) | `reduce` מחזיר אותו מצב (אידמפוטנטי) |
| U-SYNC-3 | אירוע באיחור באמצע פרק פעיל | חישוב מחדש; `EpisodeRuntime` לא משתנה עד סוף הפרק |
| U-SKEW-1 | אירוע עם dayKey "אתמול" אחרי אירוע "היום" (שעון אחורה) | `elapsedActiveDays` = 0, אין עלייה ב-u |
| U-SKEW-2 | `gapDays = 2000` | אין `onFirstAttemptAfterGap`; `clock_anomaly` |
| U-SKEW-3 | `dueDay` מחושב ל-+200 יום | נחתך ל-+90 |
| U-RST-1 | `reset_profile_model` | InterestMap ריק פרט לזרעים; θ ללא שינוי |
| U-RST-2 | `reset_progress {subjects:['math']}` | SkillState של math ריק, placement math `not_started`; hebrew ללא שינוי; האירועים ביומן נשמרים; `modelEpoch`/`subjectEpoch.math` +1 |
| U-RST-3 | אחרי reset, ניסיון ישן של math מגיע בסנכרון (epoch ישן) | לא נכנס לקיפול |
| U-ROLL-1 | K2 `gan_chova → g1`, 1.9.2027 | `rtPolicy='fluency_only'`, ladder נשאר K2, `pos` = 1.0 (ספטמבר) |
| U-ROLL-2 | K1 `g3 → g4`, ציר `math.number` עם θ_s − 4.0 = −1.0 | "מסע קצר" מתוזמן לציר |
| U-ROLL-3 | ההורה דוחה את ההצעה | `grade` נשאר, `schoolYear` מתעדכן, `pos` נעצר על gradeStart+1.0 |
| U-ROLL-4 | ClassTopic פעיל בזמן rollover | פג תוקף |
| U-ABS-1 | 95 יום בלי פעילות | reentry + מסע קצר, u = 1.0, rung −2, אין הורדת רמה |
| U-ABS-2 | `profile.paused` 40 יום | ימי ההשהיה לא נספרים; חזרה = כמו 10.4 (לא 10.5.8) אם ימי המשחק הפעילים שחלפו < 90 |
| U-CEIL-1 | ציר שבו כל המיומנויות ≥ proficient | `contentExhausted[strand]` = today; חלק ה-new עובר לצירים אחרים |
| U-CEIL-2 | מקצוע שלם מוצה | משקל מתחלק; בלוק fun = MP.X/פרויקט; אין פריט לא מאושר |
| U-FLG-11 | `EMO_CHECKIN`: עצוב ב-3 מ-5 ימי צ'ק-אין, `checkinNudge=true` | `parent_note` מיד; payload = `{ruleId, status, dayKey}` בלבד |
| U-FLG-12 | כנ"ל, `checkinNudge=false` | כלום. ה-mood לא יוצא מהמכשיר (בדיקת sync pusher) |
| U-FLG-13 | `MATH_K1_FACT_PACE` מתקיים 10 שבועות, `maxStatus='watch'` | `watch` בלבד, `notify=false`; אחרי `flag_action` אישור → `parent_note` |
| U-FLG-14 | `notRelevant` על `HEB_K1_LETTER_CONFUSION` | `dismissed` 8 שבועות; סף 25%→30%; `calibration[]` נרשם |
| U-FLG-15 | שבוע עם 9 פריטים רלוונטיים (< 15) באמצע רצף של 3 | `persisting` לא עולה ולא מתאפס |
| U-FLG-16 | כלל "או": סמן A בשבוע 1, סמן B בשבוע 2, A בשבוע 3 | `persisting` = 1 (לא 3) |
| U-Q-1 | שני הורים: ש' G2 `yes` ו-`no` | `partly`; `questionnaire_mismatch` |
| U-FW-1 | פרק מס' 2 של ילד | `itemBudget` × 0.8, closing ב-80% |

### 12.11 אינטגרציה וסימולציה (`test/sim`)

* **S-1 (בקר):** ילד סינתטי עם θ אמיתי שעולה ב-0.5 לוגיט ב-200 פריטים, ו-priors שגויים ב-+0.4 לתבנית. ההצלחה הנצפית ב-50 הפריטים האחרונים: `targetP ± 0.05`.
* **S-2 (מיקום):** 500 ילדי גוטמן-רועשים (a=1.5) עם θ ~ U(‎−2, 2). ‏|θ̂ − θ| ≤ 0.5 ב-≥90% מהריצות (K1). ממוצע פריטים ≤ 11.
* **S-3 (replay):** יומן של 1000 אירועים בסדר אקראי מ-3 מכשירים. אחרי מיון קנוני, המצב זהה בתים-לבתים למצב שנבנה ברצף.
* **S-4 (ביצועים):** `composeEpisode` עם 1500 מיומנויות ו-20,000 פריטים < 30 ms ב-iPhone 12 (Hermes). ‏`reduce` < 1 ms לאירוע. קיפול מחדש מאפס של שנת אירועים של ילד אחד (≈ 60,000 אירועים) < 60 ש' ברקע.
* **S-5 (reattribute):** יומן של 3 ילדים × 200 מפגשים; בוחרים 20 מפגשים אקראיים ושולחים `session_reattribute`. המצב אחרי החישוב מחדש זהה בתים-לבתים למצב שנבנה מהיומן בלי המפגשים האלה.
* **S-6 (anomaly):** 1,000 מפגשים סינתטיים של "הילד הנכון" ו-1,000 של "אח אחר" (θ שונה ב-≥1.5 לוגיט). `identity_suspect` תופס ≥ 70% מהאחים ו-≤ 5% מהנכונים. המספרים הם יעדי עיצוב ל-config, לא תקפות אמפירית.
* **S-7 (rollover):** סימולציה של שנה + rollover + שנה: אין NaN, אין `dueDay` בעבר, כל ציר `primary` מקבל "מסע קצר" לכל היותר פעם אחת.

---

## 13. קריטריוני קבלה

1. **טוהר ודטרמיניזם:** אין `Date`, ‏`Math.random`, ‏I/O או גלובלים ב-`packages/engine/src` (כלל ESLint). ‏S-3 עובר.
2. **כיסוי:** ≥90% שורות ו-≥85% ענפים. כל בדיקות §12 עוברות.
3. **מגדר:** `grammaticalGender` לא נגיש לקוד המנוע (type + ESLint `no-restricted-properties`). ‏U-INT-6 עובר.
4. **D9/D10/D11:** הגדרת השליטה, שומר התסכול ומדיניות ה-RT ממומשות פעם אחת ומיובאות על ידי הדשבורד וצנרת הצילום (אין העתקות).
5. **מיקום:** K1 ≤25 פריטים למקצוע, ‏K2 ≤14. ‏0 אירועי משוב נכונות בזמן מיקום. כל ציר מתחיל ומסתיים בפריט עם P ≥ 0.9 (U-PL-12).
6. **בטיחות רגשית:** מתחת לגיל 6 אין יותר משלוש החטאות רצופות (בדיקת תכונה על 10,000 רצפים אקראיים). אין טיימר נראה מחוץ לסבב מהירות (K1, opt-in).
7. **דגלים:** אין נתיב קוד שמחזיר מחרוזת דגל ל-UI של ילד (בדיקת import graph: `flags/*` מיובא רק ע"י `apps/mobile/app/(parent)/**`). כל ההודעות עוברות U-FLG-9.
8. **פרטיות:** הקלט לנרטיב (§8.4) עובר ולידציית סכמה, ואין בו שדות אסורים (U-PRF-6). דגלים, ‏EF ורגש לא נכללים ב-payload לשרת מלבד סנכרון מוצפן להורים.
9. **שקיפות:** לכל `Block` יש `reasons.length ≥ 1` שמוצג באזור ההורה.
10. **חגיגות:** `celebrationMs / sessionMs ≤ 0.15` בפרקים מדומים (מדידה ב-E2E).
11. **סימולציה:** S-1, ‏S-2, ‏S-4, ‏S-5 ו-S-7 עוברים ב-CI. S-6 רץ ב-CI ומדווח (לא חוסם) עד כיול.
12. **סולם אחד:** `packages/shared/levels.ts` הוא הייבוא היחיד של שמות שלבים ב-`packages/engine`, `packages/engine/economy` ו-`apps/*` (ESLint `no-restricted-imports` על מחרוזות `'introduced'|'practicing'|'mastered'|'retained'` מחוץ לקובץ הזה). U-LVL-* עוברים.
13. **שכבות-על:** `O.REP` ו-`O.STEP` ממומשים ב-`mastery/gates.ts` ו-`estimate/score.ts` בלבד; MATH-AC-12 של 03 עובר עם הגרף המתמטי. U-STEP-*, U-REP-* עוברים.
14. **צירים:** כל `Skill.strand` בחבילות התוכן שייך לרשימה ב-§1.1; חבילה עם ציר לא מוכר נדחית (בדיקת טעינה).
15. **משקלים:** ברירות המחדל של §6.8 ב-config; לפני פתיחת האי האנגלי אנגלית ≤ 5; עברית ≥ אנגלית נאכף בוולידציה (U-WGT-*).
16. **מקרי קצה:** כל בדיקות §12.10ג עוברות. `session_reattribute` משחזר מצב זהה למצב ללא המפגש (S-5). `reset_progress` לא מוחק אירועים.
17. **דגלים חדשים:** `maxStatus` נאכף; `EMO_CHECKIN` לא מייצא דבר מלבד `{ruleId, status, dayKey}` (בדיקת payload של ה-sync pusher, יחד עם 11 §12.1 סעיף 6).
18. **עמידות לשעון:** U-SKEW-* עוברים; אין NaN/Infinity במצב אחרי אירועים עם ts שרירותי (property test, 10,000 יומנים).

---

## 14. Proposed change to 00

> **סטטוס (סקירת Fable, 00 גרסה 2, נספח A):** כל חמש ההצעות **התקבלו** ויושמו ב-00: ‏#1 → E6a, ‏#2 → E11a, ‏#3 → C9, ‏#4 → C12 (כלל כיתה, לא גיל), ‏#5 → C2. הטקסט נשמר כאן לתיעוד הנימוק בלבד; 00 הוא המקור.
>
> לפי ההוראה ב-00, ההחלטות הקיימות נשמרות. אלה הצעות להוספה או הבהרה, עם נימוק.

1. **הוספת E6a: "נתונים נגזרים רגישים נשארים במשפחה."** דגלים מוקדמים, נטיות תלויות-הקשר, כיול EF וצ'ק-אין רגשי מחושבים במכשיר, מסונכרנים מוצפנים רק למכשירי ההורים, ולעולם לא נשלחים לספקי AI (גם לא ל-Haiku לנרטיב). *נימוק:* תיקון 13 מגדיר הערכות אישיות ומידע רפואי ונפשי כ"רגישים במיוחד" (`cbt_sel_social_emotional.md` §6). ‏C8 ו-E8 לא אומרים זאת במפורש.
2. **הוספת E11a: "המנוע דטרמיניסטי ו-event-sourced."** זמן ו-rng מוזרקים. סדר קנוני `(ts, deviceId, eventId)`. חישוב מחדש מנקודת ביקורת אחרי מיזוג. *נימוק:* Elo רגיש לסדר, ו-E6 (local-first, כמה מכשירים) מבטיח שאירועים יגיעו באיחור. בלי כלל כזה שני מכשירים יחשבו θ שונה לאותו ילד.
3. **הבהרה ל-C9 ("20–30% exploration share"):** השיעור חל על **בחירת נושאי עניין ועטיפות**, לא על רמת הקושי ולא על בחירת מיומנויות. לבחירת מיומנויות יש חקר נפרד של ε=0.2 בתוך השוליים, ובקר רצועת ההצלחה לא נפגע. *נימוק:* פרשנות אחרת הייתה מכניסה 20–30% פריטים מחוץ לרצועת C2.
4. **הבהרה למדיניות RT (D11 ב-gap audit, לא ב-00):** להוסיף ל-C2/C3 ש"RT כבוי בניקוד לכל מי שעדיין בגן (grade < א'), ול-K1 רק במיומנויות שטף אחרי שליטה, בסבב מהירות opt-in". *נימוק:* K2 יעבור את גיל 6 במהלך השנה. כלל לפי גיל היה מפעיל RT באמצע גן חובה, בניגוד לנימוק המוטורי של D11.
5. **הבהרה ל-C2:** חוגת האתגר של K1 היא 0.85/0.80/0.75. ‏0.70 קיים רק בבלוקי Stretch Zone (opt-in). זה עומד בטווח 75–85% של C2, בניגוד ל-90/80/70 שבהערות המחקר.

---

## 15. מקורות עיקריים

(הפירוט, הסטטוס והפערים ב-`docs/research-log/engine.md`.)

* Klinkenberg, Straatemeier & van der Maas 2011, *Computers & Education* 57(2) — Elo ב-Math Garden. ‏Maris & van der Maas 2012, *Psychometrika* — כלל HSHS. ‏Vermeiren, Hofman, Bolsinova et al. 2025/26, *UMUAI* 36(1) — K דינמי.
* Pelánek 2016 (CAE) ו-Pelánek et al. 2017 (*UMUAI* 26(1)) — Elo באי-ודאות, Elo גלובלי+מקומי, תיקון ניחוש.
* Doignon & Falmagne 1999 (פרק 10); ALEKS EDM 2022/2024; ‏ALEKS Adventure (≤9/≤14 פריטים).
* Skycak, FIRe (Math Academy). ‏Settles & Meeder 2016 (HLR). ‏FSRS-6 wiki. ‏SM-2. ‏Latimier et al. 2021.
* Clement, Roy, Oudeyer & Lopes 2015, *JEDM* 7(2) (ZPDES); ‏arXiv 2402.01669 (2024).
* Corbett & Anderson 1995; Baker, Corbett & Aleven 2008 (BKT ופרמטרים מנוונים).
* Wise & Kong 2005; ‏Wise & Ma 2012 (NT10); ‏NWEA rapid-guessing (10%, ≤10 ש', 3 רצופים).
* Beck & Gong 2013 (wheel-spinning). ‏Rodrigo & Baker (זיהוי תסכול מלוגים).
* Jansen et al. 2013 (קל → יותר תרגול); ‏Hidi & Renninger 2006; ‏מטה-אנליזה 2024 על התאמה אישית לפי עניין (g≈0.48 שימור); ‏Bian, Leslie & Cimpian 2017.
* Yinon et al. 2026 (קוהורט חיפה: ידע אותיות, PA, RAN ומורפולוגיה); ‏PMC3935894 (רגישות 52% / סגוליות 84%); ‏כל זכות: אבחון דידקטי.
* הערות פנימיות: `placement_assessment.md`, ‏`learning_science_workbooks.md`, ‏`extra_subjects_adaptive_curriculum.md`, ‏`interest_detection_child_profile.md`, ‏`parent_dashboard_analytics.md`, ‏`gap_audit.md`, ‏`risks_failure_modes.md`, ‏`cbt_sel_social_emotional.md`.

---

## Fable review changelog

סקירת F5 (2026-11-02). כל השינויים בתוך המסמך, בלי שינוי מבנה. בקשות למסמכים אחרים ב-`docs/review/fable-issues-F5.md`.

* **יישור ל-00 גרסה 2 (F1):** `Band` + `bandOf` ו-`Skill.band` (A6); המיפוי ל-02 בדיוק לפי C15 (`introduced→new`, `practicing→beginner|advancing`; `locked/ready` קדם-מצבים) והחזרת שמות הצמיחה לילד (זרע…עץ פורח) ש-C15 קובע; `EMO_CHECKIN` על מונה מקומי ל-14 יום (C8a); מצבי שבת `soft/full/off` וימים שקטים במנוע (B7/B7a, §6.10, §10.2); ספירה כפולה של TPR (F1 #11); §14 מסומן "התקבל ב-00".
* **§1.1 — `SubjectId` סגור, רשימת צירים קנונית.** הוחלט **לא** להוסיף `coding`/`finance` (הערת 06): ההבחנה נשמרת בציר (`logic.coding`, `world.finance`), והדשבורד מקבץ לפי ציר. נוספה טבלת כל הצירים לפי מקצוע (כולל 5 הצירים החדשים של 03) ובדיקת טעינה. הצירים של 04 מסומנים ⟂VERIFY.
* **§1.1א — שמות שלבים: ההגדרה הקנונית והמיפוי ל-02** (בעיה חוצת-מסמכים 3 בבריף). 08 נשאר המקור (`new…master`); 7 מצבי 02 ממופים כנגזרות (`locked/ready` = גרף, `retained` = rung 30), עם `packages/shared/levels.ts` (`toEconomyState`/`fromEconomyState`). עמודת "תצוגה לילד" הועברה ל-02 §4.2 (הבעלים של המיפוי החזותי) במקום שני סולמות צמיחה סותרים.
* **§1.2, §4.1, §5.1, §6.5 — בקשות 03 §17.2 התקבלו:** `Skill.extraGates` (`O.REP`, `O.STEP`); ניקוד שלבים `x = correctSteps/steps` כולל תיקון עצמי ו-BKT צל; שער ייצוגים ל-`proficient` (≥2 pictorial + ≥2 abstract ב-last8) עם ולידציית חבילה ובורר פריטים שמעדיף את הייצוג החסר.
* **§1.1, §3.2, §3.7 — 5 צירי מתמטיקה חדשים** (03): `math.facts` נכנס ל-E1 (עובדות מייצגות), `math.fractions` ל-E3 (3 פריטים), והשאר הם צירי `continuation` עם **מיקום עצל** (3 פריטי probe בבלוק new הראשון), כדי לשמור על תקרת 25 הפריטים למקצוע.
* **§3.3א — מפת מזהי השאלון** (03 §6.0א, 05 §17): המזהים הזמניים הם `questionId` וממופים ב-`questionnaire-map.json` ל-`M.*` ו-`E.*`; 04/06 ⟂VERIFY. `english.listening_vocab` → `english.listening`.
* **§6.8 — משקלי מקצועות:** התקבלה בקשת 05 (K1 אנגלית 25, K2 15) עם נימוק ושני סייגים: עברית ≥ אנגלית (B1; ב-K2 הירידה מהחשבון), ואנגלית ≤ 5 עד פתיחת האי האנגלי. נוסף כלל למקצוע כבוי/מוצה.
* **§9.2 — שלושה כללים חדשים:** `EMO_CHECKIN` (בקשת 10, opt-in T4b/`consent.checkinNudge`, מחושב במכשיר, רק בוליאני יוצא, פעם ב-14 יום, בלי הפניה); `MATH_K1_FACT_PACE` ו-`MATH_K1_READ_VS_LISTEN` (03 §14.3) ב-`maxStatus:'watch'` עד אישור הורה. נוסף מנגנון `maxStatus`.
* **§9.6–9.7 — התרעות שווא:** אפקט מדויק לכל פעולת הורה ("לא רלוונטי" = השהיה 8 שבועות + החמרת סף 20%; "דיברנו עם המורה" = 4 שבועות חסד), כלל "אותו סמן ברצף", `minWeeklyExposure` כדי שילד שמשחק מעט לא יקבל דגל מרעש, וכללי ייצוא.
* **§1.4, §1.9, §1.9א, §1.10, §1.11 — טיפוסים:** `Attempt.steps/representation` (ו-`item.answered` v2 על החוט), `behavior.signal` על החוט, **`ParentInput` הוגדר לראשונה** (היה מוזכר בלי טיפוס), `Session.attributedTo/anomaly`, **`SessionEndReason` קנוני** (מאחד את 08/10/11), `modelEpoch`, `PlacementStatus.quality/resumable`, נקודות ביקורת מוגדרות (כל 200 אירועים, 30 נשמרות).
* **§2.1 — `pos` נעצר על 1.0 בקיץ, `gradeStart` מפורש, ו-`grade` משתנה רק ב-rollover.**
* **§3.6 כלל 6, §3.7, §3.8 — מיקום שנקטע, צירי המשך, תשובות אקראיות:** כללי חידוש (14 יום), התחלה מחדש, סגירה כפויה (21 יום), גילוי חוסר עקביות גוטמן, מסע קצר חוזר פעם אחת. נוסף כלל 10 (עזרת הורה במיקום) וסייג אחים לכלל 7.
* **§10.5 — מקרי קצה (חדש, 12 סעיפים):** מיקום נקטע, תשובות אקראיות, **אח בפרופיל של אח** (היוריסטיקת `identity_suspect` + כלי תיקון `session_reattribute` שמחזיר את המצב דרך חישוב מחדש), שני מכשירים, סטיית שעון, שני סוגי איפוס (מודל פרופיל / התקדמות דרך `modelEpoch`, בלי מחיקת אירועים), **עליית כיתה 1.9.2027** (אישור הורה, rtPolicy ל-K2 בכיתה א', מסע קצר), היעדרות ≥90 יום והשהיה, לומד מהיר ותוכן שמוצה, ייצוא, הורים חולקים (hlc; מיזוג שאלון), שבוע ראשון.
* **§11 — config:** כל המספרים החדשים מרוכזים (`subjectWeights`, `gates`, `lazyPlacement`, `placementResume`, `inconsistency`, `anomaly`, `flags.maxStatus/checkin/minWeeklyExposure`, `clock`, `checkpoints`, `firstWeek`, `cadence.longAbsenceDays`).
* **§12.10א–ג, §12.11, §13 — בדיקות וקבלה:** ~60 בדיקות יחידה חדשות (U-LVL, U-STEP, U-REP, U-LAZY, U-WGT, U-PLR, U-RAND, U-ANM, U-SYNC, U-SKEW, U-RST, U-ROLL, U-ABS, U-CEIL, U-FLG-11..16, U-Q, U-FW), סימולציות S-5..S-7, וקריטריוני קבלה 12–18.
* **לא שונה:** הנוסחאות של Elo/HSHS/מיקום, ספי השליטה (D9), סולמות החזרה, מודל העניין, הפרופיל, נוסחי הדגלים הקיימים.
