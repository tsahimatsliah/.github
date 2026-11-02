# 12 — צינורות AI, תוכן וקול (AI, Content & Voice Pipelines)

> מסמך ביצוע. כפוף ל-`00-foundation-decisions.md` ובמיוחד ל-B3 (מגדר), B4 (מספרים), B5 (ניקוד), B8 (רגישות ביטחונית),
> C4 (שבח), C7 (AI לילדים), E5–E6 (Workers/R2/D1, local-first), E8, **E8a**, E9, E10.
> מחקר הפערים שבוצע לכתיבת המסמך: `docs/research-log/ai-pipelines.md`.
> פרוזה בעברית, קוד/סכמות/פרומפטים טכניים באנגלית. פרומפטים שמייצרים טקסט לילדים כתובים בעברית.
> תאריך: 2026-11-02.

## תוכן עניינים

0. היקף, מפת הצינורות ועקרונות מחייבים
1. תשתית משותפת (מודלים, עטיפת LLM, שכבת בטיחות, ערכת עברית, ולידטורים, שופט, תור אישור הורה, עלויות)
2. P1 — יצירת תוכן לתוכנית הלימודים
3. P2 — "צלם וקבל משחק"
4. P3 — עוזר סוקרטי וארגז חול לאוריינות AI
5. P4 — סיכומים ותובנות להורה
6. P5 — הפקת קול
7. P6 — צינור נכסים חזותיים ושמע עם שערי אישור
8. ציות לתנאי ספקים וחלופות
9. יעדי ביצועים ועלויות (ריכוז)
10. קריטריוני קבלה
11. פריטים פתוחים להורה (מינימום)
12. נספחים (טבלת מספרים, קטלוג טעויות, רשימות חסומות)
13. Proposed change to 00

---

## 0. היקף, מפת הצינורות ועקרונות מחייבים

### 0.1 מפת הצינורות

| ID | צינור | טריגר | מודל עיקרי (E8) | מצב | מי רואה את הפלט |
|---|---|---|---|---|---|
| P1 | יצירת תוכן (פריטים, סיפורים, רמזים, שבח) | Cron לילי + בקשת הורה | Opus 5.5 ב-Batch; שופט Sonnet 5.5 ב-Batch | אופליין | ילדים, **אחרי** ולידציה + אישור/דגימה |
| P2 | צלם וקבל משחק | הורה מצלם | Sonnet 5.5 vision (Opus 5.5 לכתב יד, ראו §13) + Haiku 5.5 להעשרה | זמן אמת, בצד השרת | ילדים, **אחרי** אישור הורה |
| P3 | עוזר סוקרטי + ארגז חול AI | ילד בצומת מוגדר בעץ | עצים מתוסרטים; Haiku 5.5 לתורות מוגבלות (K1 בלבד) | זמן אמת מוגבל | K1 (טקסט מוגבל), K2 (מתוסרט בלבד) |
| P4 | סיכומי הורה | Cron שבועי + יומי קצר | Haiku 5.5 ב-Batch | אופליין | הורים בלבד |
| P5 | הפקת קול | אחרי אישור טקסט | ספק TTS מנצח ב-bake-off; הקלטות אנושיות | אופליין (+ רינדור מיידי ל-P2) | ילדים (קבצים סטטיים) |
| P6 | נכסים חזותיים ושמע | ידני/מניפסט | Recraft, GPT Image, Rive, Kenney CC0 | אופליין | ילדים, אחרי שער מעצב |

### 0.2 עקרונות מחייבים (נגזרים מ-00, לא נתונים לשינוי במסמך זה)

1. **אין צ'אט פתוח לאף ילד** (C7). כל מה שילד רואה נוצר מראש ואומת, חוץ מתורות LLM מוגבלות ל-K1 בתוך עץ מתוסרט (P3), שגם הן עוברות ולידטורים, ונופלות לשורה מתוסרטת בכל כשל.
2. **AI בצד השרת בלבד**, מפתחות ב-Worker secrets, תקרת $ יומית (E8). המכשיר של הילד לא מדבר עם שום ספק AI.
3. **המודל לא בוחר מגדר, לא מנקד ולא סופר.** המגדר מגיע מהפרופיל (B3); ניקוד מ-Nakdan/Phonikud (B5); מספרים מטבלת חיפוש (B4); תשובות מתמטיות ומסיחים מתמטיים מחושבים בקוד (E8).
4. **Structured Outputs בכל קריאה** (E8). אין פענוח טקסט חופשי.
5. **כל קול שילד שומע מוקלט מראש ונשמר ב-R2** (E9). ב-P2 הרינדור קורה בשרת אחרי אישור ההורה ולפני שהמשחק נפתח; אין TTS חי על מכשיר הילד.
6. **E8a:** אין Gemini/Google GenAI. ElevenLabs רק עם אישור כתוב. ראו §8.
7. **מינימום מידע אישי לספקים — תוויות בלבד:** בכל payload שיוצא לספק (Batch וגם realtime) מופיעים רק הכינויים `K1`/`K2`/`K3`, רצועת גיל, מגדר דקדוקי (`m`/`f`), ושמות דמויות מהמשחק. **אסור:** שם פרטי/כינוי אמיתי, שם משפחה, שם גן/בית ספר, שם מורה, עיר, תאריך לידה, תמונת פנים, הקלטת קול של ילד. שם הילד מוזרק רק בצד הלקוח בזמן הצגה (ובקול — רק מ-`name_clip` של ההורה). Batch API אינו ZDR (שמירה 29 יום), ולכן זה חובה. אכיפה: ולידטור `V19 payloadPiiLint` (§1.6) רץ על **כל** בקשה לפני השליחה ומכשיל אותה אם מופיע טוקן מרשימת `private/family-names.json` (שמות, כינויים, שמות גן/בית ספר/מורות שההורה הזין) או דפוס טלפון/כתובת/ת"ז; בדיקת CI מריצה אותו על כל תבנית פרומפט ועל 50 payloads מדומים.
8. **תוכן שילד רואה עובר לפחות:** ולידטורים בקוד → שופט LLM → (אישור הורה או דגימה) → פרסום בחבילה חתומה ב-hash (E6).
9. **שום תוכן לילד אינו "חי".** המכשיר של הילד מציג רק: (א) חבילות (`ContentPack`) חתומות ב-sha256 שעברו את §1.8; (ב) מחרוזות UI מהריפו; (ג) תורות P3 ל-K1 בלבד, שכבר עברו ולידטורים בשרת ונופלות לשורה מתוסרטת בכל ספק. אין נתיב שבו פלט מודל מגיע למסך ילד בלי לעבור ולידטור בקוד. **בדיקת קבלה:** בדיקת רשת (Playwright + proxy על המכשיר) מראה שמכשיר במצב `shared-kids` קורא רק ל-`/v1/content/*`, `/v1/sync`, `/v1/tts/*` (הורדות בלבד) ול-`/v1/ai/socratic` (רק כשפרופיל K1 פעיל); K2/K3 — אפס קריאות ל-`/v1/ai/*`.
10. **אופליין הוא המצב הרגיל של הילד.** כל צינור מניח שהמכשיר של הילד יכול להיות מנותק שעות: חבילות ואודיו יורדים מראש ונשמרים מקומית; P2 ו-P3 הם היחידים שדורשים רשת בזמן אמת, ולשניהם יש התנהגות מוגדרת בניתוק (§0.5).

### 0.3 שני זמני ריצה

| זמן ריצה | מה רץ שם | למה |
|---|---|---|
| **Cloudflare Worker** (`workers/api`, TypeScript/Hono) | כל קריאות Claude, Batch orchestration (Cron), ולידטורים ב-TS, בניית חבילות, R2/D1, TTS בזמן אמת ל-P2 (Azure REST) | E5 |
| **Media runner** (`tools/pipelines`, GitHub Actions מתוזמן + הפעלה ידנית; Python 3.12 + Node 22 + ffmpeg) | Phonikud ONNX, ASR round-trip (ivrit.ai Whisper), נרמול עוצמה, svgo, contact sheets, bake-off | ספריות Python/ffmpeg לא רצות ב-Worker |

ה-runner מושך עבודות מ-`GET /internal/jobs?kind=…` (טוקן שירות) ומחזיר תוצרים ל-R2 + `POST /internal/jobs/:id/result`. זה מבטל צורך בשרת נוסף. אופציה לשלב 2: Cloudflare Container ל-Phonikud בזמן אמת (ראו §1.5.2).

### 0.4 מבנה תיקיות (בתוך המונוריפו של E11)

```
packages/content/
  schemas/            # zod storage schemas + JSON output schemas (Claude subset)
  aig/                # deterministic item models (math, phonics, spelling)
  errors/             # error catalogs (math/hebrew-spelling/english/vocab)
  templates/          # game template registry (P2 + P1)
  hebrew/             # gender, numerals, nikud utils, bidi, reading level
  validators/         # all code validators
  prompts/            # prompt templates (*.md with {{slots}}), versioned
  judge/              # rubrics + calibration set
workers/api/src/ai/   # llm client, routing, cost guard, batch orchestrator, photo pipeline, socratic runtime
tools/pipelines/      # media runner: voice, bake-off, asset post-processing
assets/               # style/, prompts/, refs/, LICENSES.json (provenance log)
```

### 0.5 מטריצת מקרי קצה לצינורות (כלל אחד לכל תא)

| מצב | P1 תוכן | P2 צלם-ושחק | P3 סוקרטי (K1) | P4 הורה | P5 קול |
|---|---|---|---|---|---|
| **ילד אופליין** | משחק מחבילות שכבר ירדו; אם לצומת פעיל אין ≥ 10 פריטים מקומיים → המנוע בוחר צומת שכן (08) ולא מציג "אין תוכן" | המשחק שנבנה ממתין בשרת; יורד בחיבור הבא; ההורה רואה "מוכן, יורד כשהאייפד מתחבר" | צומת `llm_turn` מדלג ישר ל-`fallback` המתוסרט, בלי המתנה (בדיקת `expo-network` לפני הקריאה) | — | נגן מקומי בלבד; שורה בלי אודיו מקומי → טקסט + צליל "דינג" (K1), או דילוג על הפריט (K2/K3) |
| **הורה אופליין בזמן צילום** | — | תור לכידה מקומי (§3.16): עד 20 דפים נשמרים מוצפנים במכשיר ההורה, נשלחים אוטומטית בחיבור הבא | — | — | — |
| **ספק AI לא זמין (5xx/timeout)** | Batch נדחה ללילה הבא; אין "חור" כי יש מלאי ל-7 ימים | §3.15: "נסה שוב בעוד כמה דקות"; אחרי 2 כשלים — הקלדה ידנית | `fallback` מתוסרט; אחרי 3 כשלים ברצף — P3 מושבת ל-30 דק׳ (circuit breaker) | דוח מבוסס קוד בלבד | TTS לא זמין: §3.13; אודיו קיים לא מושפע |
| **תקרת $** | §1.9 | §1.9 | §1.9 | לא מושפע (זניח, מחושב מראש) | P2 בלבד מושפע |
| **שבת/חג (B7)** | Cron רץ כרגיל (שרת); אין התראות | ההורה יכול לצלם; אין push; המשחק מופיע אחרי ההשהיה | לפי מצב שבת של המשפחה (10 §10.2) | הדוח יוצא **לפני** כניסת השבת (§5.5) | רינדור לילי רץ |
| **K3 (2.5)** | 100% אישור הורה; רק תבניות ללא כישלון | רק תבניות 1–3, 19; רק במצב משותף | אין P3 כלל | סיכום קצר, בלי מיומנויות "בדרך" | רק `n`/`pl` עד קביעת מגדר; קצב −20% |
| **מכשיר משותף, החלפת פרופיל באמצע** | חבילות נטענות לפי `childScope`; פריטי K1 לא מוצגים ל-K2 | המשחק משויך ל-`forChildren[]` בלבד | תור LLM נסגר ב-`profile_switch` | — | cache משותף; גרסת מגדר לפי הפרופיל הפעיל |
| **סנכרון: אותה חבילה אושרה משני מכשירי הורה** | אידמפוטנטי לפי `packId+version`; האישור הראשון קובע, השני מקבל "כבר אושר" | אותו דבר לפי `jobId` | — | — | — |
| **"מוזר לי" מהילד** | הפריט ל-`quarantined` מיד + החלפה (§1.8) | אותו דבר; אם ≥ 3 דגלים על אותו `jobId` → המשחק כולו מוסתר עד בדיקת הורה | התור נרשם כ-`kid_flag`, הצומת הבא מתוסרט | מופיע בדוח כספירה | הקליפ מסומן `review:"kid_flag"` ונשמע להורה |
| **ילד מתוסכל (D10, 08 §6.7)** | — | `maxConsecutiveMisses` → רמז → פריט קל יותר מאותו משחק | `llm_turn` לא נפתח כשהגלאי פעיל; רק רמזים מתוסרטים | — | — |
| **נגישות: Reduce Motion / VoiceOver (הורה)** | — | מסך האישור עם תוויות VoiceOver מלאות (09) | אנימציית "חושב" הופכת לנורית סטטית | מייל בגרסת טקסט | — |

---

## 1. תשתית משותפת

### 1.1 טבלת ניתוב מודלים

| Job ID | שימוש | Model ID | מצב | `effort` | `max_tokens` | Cache | Fallback |
|---|---|---|---|---|---|---|---|
| `gen.items.skin` | עטיפה סיפורית לפריטי AIG | `claude-opus-5-5` | Batch | `medium` | 16000 | system+few-shot, TTL 1h | `fallbacks:"default"`; פריט שהגיע ממודל fallback מסומן ועובר אותה ולידציה |
| `gen.story` | סיפורונים / קטעי קריאה | `claude-opus-5-5` | Batch | `high` | 16000 | כנ"ל | כנ"ל |
| `gen.hints` | סולמות רמזים ושורות שבח | `claude-opus-5-5` | Batch | `medium` | 8000 | כנ"ל | כנ"ל |
| `judge.content` | שופט איכות | `claude-sonnet-5-5` | Batch | `medium` | 4000 | rubric, TTL 1h | `fallbacks:"default"` |
| `photo.extract` | חילוץ מתמונה (מודפס) | `claude-sonnet-5-5` | Realtime, streaming | `medium` | 8000 | system 5m | Opus 5.5 → ידני |
| `photo.extract.hw` | חילוץ כתב יד — קריאה שנייה **מותנית** (00 E8: כבויה כברירת מחדל; נדלקת רק אם הערכת 30 הדפים מראה רווח, §3.5.1) | `claude-opus-5-5` (במקביל ל-Sonnet) | Realtime | `medium` | 8000 | system 5m | Sonnet בלבד → הורה מקליד |
| `photo.enrich` | גרסאות לפי ילד, רמזים, מסיחים לשוניים | `claude-haiku-5-5` | Realtime | `low` | 4000 | system 5m | ללא LLM: תבניות קוד בלבד |
| `photo.story` | סיפור ילד → סצנות | `claude-sonnet-5-5` | Realtime | `medium` | 8000 | system 5m | פיצול לפי משפטים בקוד |
| `socratic.turn` | תור מוגבל ל-K1 | `claude-haiku-5-5` | Realtime | `low` | 600 | system 1h (keep-warm ע"י שימוש) | שורה מתוסרטת (אין fallback צד-שרת ל-Haiku) |
| `safety.classify` | סיווג קלט ילד | `claude-haiku-5-5` | Realtime | `low` | 200 | system 1h | חסימה שמרנית |
| `parent.weekly` | סיכום שבועי | `claude-haiku-5-5` | Batch | `medium` | 6000 | system 1h | תבנית קוד בלבד |
| `asset.judge` | שופט חזותי לנכסים | `claude-sonnet-5-5` vision | Batch | `medium` | 2000 | rubric 1h | אישור מעצב בלבד |

כללי API שחלים על כל הטבלה (מאומת, ראו research log §1):
- `output_config.format` לכל job. **אין** `tool_choice` כפוי (400 ב-Opus/Sonnet 5.5), **אין** prefill, **אין** `temperature/top_p/top_k`.
- ב-Opus 5.5 אי אפשר לכבות thinking; השליטה היחידה היא `output_config.effort` (ברירת מחדל `medium` — תמיד לציין מפורשות).
- `stop_reason` נבדק לפני כל parse: `refusal` → אין retry עם אותו פרומפט, הפריט נזרק ונרשם; `max_tokens` → retry אחד עם ×2.
- סכמה אחת יציבה לכל job (שינוי `output_config.format` מבטל cache; גרסת הסכמה ב-`prompt_version`).

### 1.2 עטיפת LLM (`workers/api/src/ai/llm.ts`)

```ts
import Anthropic from "@anthropic-ai/sdk";
import type { ZodType } from "zod";

export type JobId =
  | "gen.items.skin" | "gen.story" | "gen.hints" | "judge.content"
  | "photo.extract" | "photo.extract.hw" | "photo.enrich" | "photo.story"
  | "socratic.turn" | "safety.classify" | "parent.weekly" | "asset.judge";

interface Route {
  model: "claude-opus-5-5" | "claude-sonnet-5-5" | "claude-haiku-5-5";
  effort: "low" | "medium" | "high";
  maxTokens: number;
  cacheTtl: "5m" | "1h";
  serverFallback: boolean; // only Opus/Sonnet 5.5 on the Claude API
}

export const ROUTES: Record<JobId, Route> = { /* = table §1.1 */ } as any;

export interface LlmResult<T> {
  ok: boolean;
  data?: T;
  failure?: "refusal" | "max_tokens" | "schema" | "validation" | "timeout" | "budget" | "api";
  servedModel?: string;
  usage: { input: number; output: number; cacheRead: number; cacheWrite: number; usd: number };
  latencyMs: number;
}

export async function callStructured<T>(opts: {
  job: JobId;
  system: Anthropic.TextBlockParam[];   // stable blocks first; last stable block carries cache_control
  messages: Anthropic.MessageParam[];   // volatile content (images first, then text)
  jsonSchema: Record<string, unknown>;  // Claude-subset schema (all required, additionalProperties:false)
  storageSchema: ZodType<T>;            // full validation in code (lengths, ranges, counts)
  timeoutMs: number;
  env: Env;
}): Promise<LlmResult<T>> {
  const r = ROUTES[opts.job];
  await costGuard.assertAllowed(opts.env, opts.job);          // per-day $ cap (E8)
  const client = new Anthropic({ apiKey: opts.env.ANTHROPIC_API_KEY, timeout: opts.timeoutMs, maxRetries: 2 });
  const t0 = Date.now();
  const body: any = {
    model: r.model,
    max_tokens: r.maxTokens,
    system: opts.system,
    messages: opts.messages,
    output_config: { effort: r.effort, format: { type: "json_schema", schema: opts.jsonSchema } },
  };
  if (r.serverFallback) { body.betas = ["server-side-fallback-2026-07-01"]; body.fallbacks = "default"; }
  const res = await (r.serverFallback ? client.beta.messages : client.messages).create(body);
  const usage = meter(res.usage, r.model);                      // price table §1.8
  await costGuard.record(opts.env, opts.job, usage.usd);
  const base = { usage, latencyMs: Date.now() - t0, servedModel: (res as any).model };
  if (res.stop_reason === "refusal") return { ok: false, failure: "refusal", ...base };
  if (res.stop_reason === "max_tokens") return { ok: false, failure: "max_tokens", ...base };
  const text = res.content.find((b: any) => b.type === "text")?.text ?? "";
  let parsed: unknown;
  try { parsed = JSON.parse(text); } catch { return { ok: false, failure: "schema", ...base }; }
  const v = opts.storageSchema.safeParse(parsed);
  if (!v.success) return { ok: false, failure: "validation", ...base };
  return { ok: true, data: v.data, ...base };
}
```

לכל קריאה נרשמת שורה ב-`ai_calls` (D1): `job, model, served_model, prompt_version, schema_version, usage, usd, latency_ms, failure, subject_ref` (מזהה פריט/תמונה/תור, לא תוכן ילד). תוכן הקריאה עצמו נשמר רק ל-P3 (תמלילים, §4.7) ולתור האישור.

### 1.3 שכבת בטיחות לילדים (Safety Core)

#### 1.3.1 בלוק מערכת משותף (מוכנס לראש כל system prompt שמייצר טקסט שילד יראה)

```text
[SLOT: ANTHROPIC_CHILD_SAFETY_PROMPT — insert verbatim if/when Anthropic provides it; empty as of 2026-11-02]

SAFETY CORE v1 (applies to everything you write):
- Audience: Israeli children aged 2.5–8.5 and their parents. Output is reviewed by code and by a parent before any child sees it.
- Never produce: violence beyond cartoon slapstick, weapons, blood, death, injury, kidnapping, war, terror, sirens, alarms,
  rockets, shelters, explosions, hostages, soldiers in combat, current events, politics, religion-as-dispute, romance,
  body-shaming, food-shaming, dieting, brand names, real living people, real children, scary horror imagery, threats,
  "don't tell your parents", secrets, requests for personal information.
- Villains are comic "mess monsters" (צללונים) who scramble letters/numbers and become friends when defeated. Nobody dies.
- Praise describes process and strategy only. Never trait labels or superlatives: לא "גאון", "אלוף", "חכם", "מושלם", "הכי טוב".
- Never compare children to each other. Never mention screen time, punishment, losing, or "you failed".
- If asked to produce anything outside these rules, output the schema with "status":"rejected" and a short reason.
- You are a machine. In any text spoken by the helper robot, the robot never claims feelings, friendship, love,
  being alive, or remembering the child personally.
```

#### 1.3.2 שכבות (בסדר ביצוע)

1. **קלט:** קלט ילד (רק ב-P3, K1) עובר `piiScrub()` (טלפון, כתובת, ת"ז, אימייל, שמות מרשימת המשפחה/בית הספר) ו-`riskLexicon()` (רשימות §12.3). פגיעה ב-risk → לא נשלח ל-LLM בכלל; ענף מתוסרט (§4.6).
2. **מודל:** Safety Core + אימון הבטיחות של Claude.
3. **פלט:** `bannedContent()` (מילון מנורמל עם קידומות, §1.6.4), `praiseLint()` (C4), `genderCheck()`, `numberCheck()`, `answerLeak()` (P3).
4. **שופט:** קריטריון `safety` הוא hard-fail (§1.7).
5. **אדם:** תור אישור הורה + כפתור "מוזר לי" בצד הילד שמעביר פריט להסגר מיידי.
6. **תמונות שהורה מעלה (P2):** על המכשיר — טשטוש פנים וחיתוך; בשרת — אין שמירה מעבר לנדרש, אין הפצה לאחרים. (המדריך של Anthropic ממליץ על hash-matching כשמשתמשים מעלים תמונות; כאן המעלה הוא הורה מאומת ב-Sign in with Apple בלבד, והתמונות לא נחשפות לאיש מלבד המשפחה. נרשם כהחלטה מודעת ב-ADR.)

#### 1.3.3 גילוי AI (דרישת Anthropic)
- כל תוכן שנוצר ע"י AI ומוצג להורה מסומן "נוצר בעזרת AI" בתור האישור ובדוחות.
- הרובוט-העוזר (P3) מוצג תמיד כמכונה: גלגלי שיניים גלויים, צליל מתכתי, שורת פתיחה מתוסרטת "אני מכונה. אני לא חושב כמו בן אדם, אני מנחש לפי דפוסים." הוא ישות נפרדת מהקמע (C7).
- באזור ההורה: עמוד "על ה-AI באפליקציה" — אילו ספקים, מה נשלח, מה נשמר, מדיניות מחיקה, והצהרת ציות (המדריך דורש הצהרה פומבית; באפליקציה משפחתית זה העמוד הזה).

### 1.4 שתי שכבות סכמה

- **Output schema (Claude subset)** — מה שהמודל מחויב אליו: כל השדות `required`, `additionalProperties:false`, בלי `minimum/maxLength`, בלי nullables (מחרוזת ריקה / מערך ריק במקום null — כי `["string","null"]` נספר במגבלת 16 ה-unions), `enum` לכל שדה סגור. סה"כ union params ≤ 16 לבקשה.
- **Storage schema (zod)** — מה שנשמר ב-D1/R2: אורכים, טווחים, ספירות מינימום לתבנית, קשרים בין שדות. כל פלט LLM עובר המרה `fromLlm()` → `Storage.parse()`.

#### 1.4.1 טיפוסים בסיסיים (storage, zod)

```ts
import { z } from "zod";

/** Gendered text (B3). n = neutral form when one exists (infinitive / "בואו נ…" / name vocative). */
export const GText = z.object({
  m: z.string().min(1).max(400),
  f: z.string().min(1).max(400),
  pl: z.string().min(1).max(400),
  n: z.string().max(400).default(""),
});
export type GText = z.infer<typeof GText>;

/** Hebrew text carried in three layers. Display never contains Phonikud-only marks (U+05AB, U+05BD as shva, '|'). */
export const HeText = z.object({
  plain: z.string(),            // consonantal, NFC
  pointed: z.string(),          // full nikud for display (B5), NFC, canonical mark order
  tts: z.string(),              // TTS input: selective nikud + stress marks + numeral words; never displayed
  nikudSource: z.enum(["nakdan", "phonikud", "human", "lexicon"]),
  nikudDisagreements: z.array(z.string()).default([]), // words where Nakdan≠Phonikud → parent spot-check
});

export const AgeBand = z.enum(["K3", "K2", "K1"]);
export const Lang = z.enum(["he", "en"]);
export const Shape = z.enum(["pair", "qa", "cloze", "sequence", "category", "spell", "math_expr", "statement", "story_scene"]);

export const MathExpr = z.object({
  a: z.number().int(), op: z.enum(["+", "-", "×", ":"]), b: z.number().int(),
  blank: z.enum(["result", "a", "b"]),
  answer: z.number().int(),            // ALWAYS computed in code
});
```

#### 1.4.2 Item (storage)

```ts
export const Item = z.object({
  id: z.string(),                       // ulid
  familyId: z.string(),                 // item family = one AIG item model or one source photo item
  skillId: z.string(),                  // skill-graph node (spec 0x adaptive engine)
  shape: Shape,
  lang: Lang,
  ageBands: z.array(AgeBand).min(1),
  difficulty: z.number().int().min(1).max(5),   // designer-assigned level (gap_audit #4)
  prompt: z.object({ text: GText, he: HeText.optional(), audioKey: z.record(z.string()).default({}) }), // audioKey per variant m/f/pl
  answer: z.object({ value: z.string(), math: MathExpr.optional() }),
  options: z.array(z.object({ id: z.string(), text: z.string(), isCorrect: z.boolean(), errorCode: z.string().default("") })).default([]),
  hints: z.array(GText).max(3).default([]),     // hint ladder: nudge → strategy → worked step (never the answer)
  feedbackCorrect: z.array(GText).default([]),  // process praise (C4)
  imageRef: z.string().default(""),             // asset registry id (P6), never generated at runtime
  source: z.object({
    kind: z.enum(["aig", "llm_skin", "photo", "curated", "parent"]),
    sourceItemId: z.string().default(""),       // photo item it was cloned from
    isHomeworkSource: z.boolean().default(false),
  }),
  qa: z.object({
    validators: z.record(z.enum(["pass", "warn", "fail"])),
    judge: z.object({ scores: z.record(z.number()), verdict: z.enum(["pass", "revise", "reject"]) }).optional(),
    review: z.enum(["auto", "sampled_ok", "parent_ok", "parent_fixed", "quarantined"]),
  }),
  provenance: z.object({ promptVersion: z.string(), model: z.string(), servedModel: z.string(), createdAt: z.string() }),
});
```

#### 1.4.3 ContentPack (storage + manifest, E6)

```ts
export const ContentPack = z.object({
  packId: z.string(),                  // e.g. "math.k1.mult-6-9.v3"
  version: z.number().int(),
  kind: z.enum(["curriculum", "photo", "story", "seasonal", "ai_literacy"]),
  childScope: z.array(z.enum(["K1", "K2", "K3", "family"])),
  skillIds: z.array(z.string()),
  items: z.array(Item),
  stories: z.array(z.lazy(() => Story)).default([]),
  templates: z.array(z.string()),      // template ids playable with this pack
  audioManifest: z.string(),           // R2 key of audio-manifest.json
  assetRefs: z.array(z.string()),
  createdAt: z.string(),
  approvedBy: z.enum(["parent", "sampling_policy"]),
  sha256: z.string(),                  // over canonical JSON; device verifies (E6)
});
```

#### 1.4.4 Story (storage)

```ts
export const StoryPage = z.object({
  idx: z.number().int(),
  text: GText,                          // gendered when it addresses the reader; narration usually neutral → m=f=pl
  he: HeText,
  imageRef: z.string(),                 // composed scene from asset library
  choice: z.object({                     // optional branching (K1/K2 with parent)
    prompt: GText, options: z.array(z.object({ text: z.string(), gotoIdx: z.number().int() })).max(3),
  }).optional(),
  comprehensionItemIds: z.array(z.string()).default([]),
});
export const Story = z.object({
  storyId: z.string(),
  title: HeText,
  ageBand: AgeBand,
  nikudLevel: z.enum(["L1_kamatz_patach", "L2_plus_chirik", "L3_plus_tzere_segol", "L4_plus_cholam_shuruk", "L5_full", "unpointed"]),
  readingMode: z.enum(["listen", "read_along", "read_alone"]),
  pages: z.array(StoryPage).min(3).max(12),
  characters: z.array(z.object({ charId: z.string(), gender: z.enum(["m", "f"]) })),  // from character registry
  authoredBy: z.enum(["generated", "child", "public_domain"]),
  childAuthorRef: z.string().default(""),   // K1/K2 when authoredBy=child (P2 story flow)
  license: z.string().default("own"),
});
```

### 1.5 ערכת עברית (`packages/content/hebrew`)

#### 1.5.1 מגדר (B3)
- כל מחרוזת שפונה לילד היא `GText`. בחירת הגרסה בצד הלקוח: `pick(gt, profile.gender)`; למסך משפחתי `pl`. ל-K3 (מגדר לא נקבע) — `n` אם קיים, אחרת `pl` עד שההורה יקבע; המערכת תומכת בשני המגדרים (A2).
- **בפרומפט** נאמר למודל לייצר את שלוש הגרסאות בשדות נפרדים, ולהעדיף צורה ניטרלית (`n`) כשהיא טבעית: מקור ("לגעת בכוכב"), "בואו נ…", פנייה בשם (`{{name}}` — מוזרק בלקוח).
- **`genderCheck(gt)`** (קוד, §1.6.2) בודק שכל גרסה עקבית פנימית.

#### 1.5.2 ניקוד (B5) — `NikudService`

```ts
interface NikudService {
  point(plain: string, opts: { genre: "modern" }): Promise<{ pointed: string; source: "nakdan" | "phonikud" }>;
}
// Order: lexicon (approved family word list) → Dicta Nakdan API (if terms confirmed) → Phonikud.
// Cross-check: for K2 content and for every word of every photo item, run BOTH Nakdan and Phonikud;
// strip stress/shva extras from Phonikud, compare per word after canonicalization; mismatches → HeText.nikudDisagreements.
```

- **Phonikud** (`phonikud-onnx`, CC BY 4.0) רץ ב-media runner לתוכן לילי. לזמן אמת (P2) — אופציה א': Nakdan API מה-Worker; אופציה ב' (שלב 2): Cloudflare Container קטן עם Phonikud. כל עוד אין שירות ניקוד זמין בזמן אמת: משחקי P2 ל-K1 נפתחים בתצוגה לפי רמת דהיית הניקוד שלה (אם היא כבר ב"חלקי/ללא" — מיד; אחרת ממתינים לעבודת ה-runner, יעד ≤ 10 דקות), ומשחקי K2 ממתינים לניקוד.
- **סימני Phonikud לא-תקניים** (U+05AB להטעמה, U+05BD לשווא נע, `|` לקידומת) נשמרים **רק** ב-`HeText.tts`. `toDisplay()` מסיר אותם; בדיקת יחידה נכשלת אם הם מופיעים ב-`pointed`.
- **קנוניזציה:** NFC + סדר סימנים קנוני (שין/שׂין-נקודה → דגש → תנועה), ותיקון "חולם חסר על ו'" (U+05BA). בדיקת רינדור על מכשיר אמיתי בגופן הנבחר (B5) היא חלק מקבלה.
- **LLM לא מנקד.** בפרומפטים: "כתוב/כתבי טקסט ללא ניקוד; הניקוד יתווסף אוטומטית." חריג: מילים דו-משמעיות קריטיות — המודל מחזיר אותן ב-`ambiguousWords[]` עם הקריאה המיועדת בתעתיק לטיני ("sefer", "safar"), והקוד בוחר את הניקוד המתאים מתוך חלופות Nakdan.

#### 1.5.3 מספרים (B4) — `numeral()`

```ts
type NGender = "m" | "f";
type NForm = "count" | "standalone" | "attributive" | "construct" | "ordinal";
export function numeral(n: number, gender: NGender, form: NForm): { plain: string; pointed: string; tts: string };
// Backed by a hand-checked table (Appendix §12.1), 0–100 + hundreds/thousands composition rules.
// "count" = abstract counting = feminine forms (B4: אַחַת, שְׁתַּיִם, שָׁלוֹשׁ...).
// Rules: 2 → שְׁנַיִם/שְׁתַּיִם standalone, שְׁנֵי/שְׁתֵּי before noun; 1 follows noun (יֶלֶד אֶחָד).
// 11–19 gendered compounds (אַחַת־עֶשְׂרֵה / אַחַד־עָשָׂר). Construct for definite (שְׁלֹשֶׁת הַיְּלָדִים).
```

- **מקור אחד:** `packages/content/hebrew/numerals.json` הוא הטבלה היחידה (00 B4); 03 §10.4 ו-§12.1 כאן הם שני תיאורים של אותו קובץ. `numeral()` הוא ה-API הנמוך; **`numPhrase(n, noun, form)`** (מוגדר ב-03 §10.4, ממומש כאן) עוטף אותו ומטפל ב: ו' החיבור בין רכיבים (וּ לפני שווא/בומ"ף, וַ לפני חטף, אחרת וְ; **לעולם לא וָ במספרים**), מאות (מֵאָה נקבה → שְׁלוֹשׁ מֵאוֹת), אלפים (שְׁלוֹשֶׁת אֲלָפִים…; "אֶלֶף" יחיד מ-11,000), שם עצם ברבים תמיד אחרי ≥ 2, סודרים, שברים (⟂VERIFY "חֲלָקֵי" מול "חֶלְקֵי" לפני הקלטת קליפ ":"), וניסוח שעון. בדיקת CI משותפת: 03 `MATH-AC-25` (אותו fixture רץ ב-03 וב-12). צורות הנסמך `verified:false` עד OPD-31.
- בכל טקסט שנוצר ע"י LLM מספרים מופיעים כ-slots: `{{num:a|noun:תפוח}}`. הקוד שולף מגדר שם-עצם מ-`nounLexicon` (שדה `gender`, נבדק ידנית), ממלא צורה נכונה דרך `numPhrase`, ומכניס ספרות לתצוגה מתמטית ומילים ל-TTS.
- `numberCheck(text)` מוודא שאין ספרות "חופשיות" בטקסט TTS ושכל צירוף מספר+שם עצם תואם מגדר.

#### 1.5.4 כיווניות (bidi)
- ביטויים מתמטיים נשמרים כטוקנים (`MathExpr`), לעולם לא כמחרוזת. התצוגה עוטפת ב-FSI/PDI (`⁨…⁩`) ובקומפוננטה LTR מבודדת. אנגלית בתוך עברית עטופה ב-`<bdi>`-equivalent (LRI `⁦` … PDI).

#### 1.5.5 רמת קריאה (`readingLevel()`)

| מדד | K2 (גן חובה) | K1 ספט׳–ינו׳ (מנוקד) | K1 פבר׳ ואילך (דהייה) |
|---|---|---|---|
| מילים במשפט (ממוצע / מקס׳) | ≤ 5 / 7 | ≤ 8 / 12 | ≤ 10 / 15 |
| אותיות במילה (ממוצע) | ≤ 4 | ≤ 5 | ≤ 5.5 |
| מילים מחוץ ל-lexicon הרמה | 0 (מילון פענוח) | ≤ 5% | ≤ 10% |
| תנועות מותרות | רק מרמת הניקוד הנוכחית של הילד (`nikudLevel`) | כל התנועות | כל התנועות |
| אותיות מותרות (פענוח) | רק אותיות שנלמדו (מהמנוע האדפטיבי) | — | — |
| עמודי סיפור | 3–6 | 5–10 | 6–12 |

המדד מחושב בקוד על `plain`/`pointed`; חריגה → `fail` (K2) או `warn` (K1).

### 1.6 ספריית ולידטורים (`packages/content/validators`)

כל ולידטור מחזיר `{ status: "pass"|"warn"|"fail", details }`. `fail` אחד = הפריט לא ממשיך לשופט.

| ID | מה בודק | איפה חל | חומרה |
|---|---|---|---|
| `V01 schema` | storage zod | הכל | fail |
| `V02 mathRecompute` | `answer === eval(a op b)` ברציונליים מדויקים; חלוקה רק כשלמה (אלא אם הכישור שברים); תוצאה בטווח הכישור | מתמטיקה | fail |
| `V03 numbersInText` | כל מספר בטקסט == ערך במודל AIG; אין מספרים "מומצאים" | בעיות מילוליות | fail |
| `V04 distractors` | שונים מהתשובה, שונים זה מזה, באותו טווח/פורמט, כל אחד עם `errorCode` | MCQ | fail |
| `V05 genderCheck` | עקביות m/f/pl (§1.6.2) | כל GText | fail |
| `V06 nikudCoverage` | כיסוי ניקוד ≥ 0.9 מאותיות שאינן אם-קריאה/סופית; אין סימני Phonikud בתצוגה | K2 + K1 מנוקד | fail |
| `V07 nikudAgreement` | Nakdan מול Phonikud; אי-התאמות → רשימה להורה | K2, P2 | warn |
| `V08 decodability` | אותיות ותנועות ⊆ מה שהילד למד | K2 קריאה | fail |
| `V09 readingLevel` | §1.5.5 | טקסט לקריאה | fail/warn |
| `V10 bannedContent` | §1.6.4 | הכל | fail |
| `V11 praiseLint` | רשימת C4 אסורים + חובת תיאור תהליך | שבח/משוב | fail |
| `V12 answerLeak` | התשובה (ספרות/מילים/צורות מספר/תווית אפשרות) לא מופיעה בשאלה/רמז | פריטים, P3 | fail |
| `V13 lengthBudget` | אורך לפי תבנית (כרטיס/בועה) | הכל | fail |
| `V14 bidiSafe` | אין ספרות/לטינית לא-עטופות בתוך טקסט עברי לתצוגה | תצוגה | fail |
| `V15 dedupe` | n-gram/פשטות: לא כפול לפריט קיים באותו skill | P1 | warn |
| `V16 englishDecodable` | מילים אנגליות ⊆ graphemes שנלמדו (scope & sequence) | אנגלית | fail |
| `V17 repetitionLoop` | פלט VLM: n-gram חוזר > 3 פעמים או אורך > 3× הצפוי | P2 extract | fail→retry |
| `V18 piiResidual` | שמות מרשימת "לא-משפחה" / טלפון / כתובת | P2, P4 | fail (מוחלף בממלא מקום) |
| `V19 payloadPiiLint` | **לפני שליחה לספק:** אין שם/כינוי/גן/בית ספר/מורה מ-`private/family-names.json`, אין טלפון/כתובת/ת"ז/אימייל, אין base64 של תמונה שלא עברה את §3.3 (hash רשום) | כל קריאת LLM/TTS/תמונה | fail (הבקשה לא נשלחת; נרשם `failure:"pii_guard"`) |
| `V20 answerKeyGuard` | במצב `homework_practice`: אף פריט/תשובה מקוריים לא נכנסים לחבילת הילד; במצב `exam_page`: אין פריט מקורי בכלל (§3.10) | P2 | fail (החבילה לא נבנית) |
| `V22 genericVoFallback` | חבילת הליבה המצורפת ל-build מכילה `vo.generic.<intent>` לכל `intent` ברשימת 11 §4.11 (instruction, praise, retry, hint, goodnight…), ב-m/f/pl, עם אודיו תקין; חסר → build נכשל | חבילת ליבה | fail |
| `V23 heroWording` | פרומפטים ל-P6 ושורות דמות: "soft toy lantern / חליפת-צעצוע עם ניצוץ" לפנסון; המילים robot/רובוט/gears/גלגלי שיניים/screen-face מותרות **רק** לדמויות המכונה (גלגלי, `ROBOT`/`BOTS` של 07) — 00 G4, 01 AC-HERO-1; רץ גם על `tools/` ו-`assets/prompts/` | P6, P1 | fail |
| `V21 audioFormat` | פורמט ותקציב לפי 14 `PQ-R-36`/`PQ-B-24` (אותו כלל ב-`content:validate` וב-`lint:assets`): SFX ≤ 1 ש׳ = WAV/CAF PCM 44.1 kHz מונו אם ≤ 150 KB, אחרת AAC-LC; VO = AAC-LC 64–96 kbps מונו 48 kHz; מוזיקה = AAC 128 kbps סטריאו; עוצמה לפי §6.9 (VO −18 LUFS ±1, שיא ≤ −1 dBTP); **חבילת 2D ≤ 8 MB בדיסק** (יעד 5 MB) — חבילה גדולה יותר מפוצלת לפי `skillId` | כל חבילה / נכס שמע | fail |

#### 1.6.1 חישוב מתמטי מחדש (V02)

```ts
export function recompute(e: { a: number; op: "+" | "-" | "×" | ":"; b: number }): number | null {
  switch (e.op) {
    case "+": return e.a + e.b;
    case "-": return e.a - e.b;              // skill decides if negatives allowed (never for K2/K1 grade 3)
    case "×": return e.a * e.b;
    case ":": return e.b !== 0 && e.a % e.b === 0 ? e.a / e.b : null; // remainder skills use a separate shape
  }
}
// Photo items: the model's claimed answer is IGNORED for scoring; recompute() is the key.
// If the page shows a child-written answer, it is stored as childAnswer and compared in code (parent-only insight).
```

#### 1.6.2 בדיקת התאמת מגדר (V05) — היוריסטיקה + מילון

1. **מילון צורות פנייה** (`gender-forms.json`, ~400 רשומות שנבנות מראש ע"י P1 + בדיקה ידנית): לכל לקסמה — צורת ז׳/נ׳/רבים בגוף שני ובציווי, למשל `{lemma:"מוכן", m:"מוכן", f:"מוכנה", pl:"מוכנים"}`, `{lemma:"לחץ", m:"לחץ", f:"לחצי", pl:"לחצו"}`, `{lemma:"אתה", m:"אתה", f:"את", pl:"אתם"}`, `{lemma:"שלך", m:"שֶׁלְּךָ", f:"שֶׁלָּךְ", pl:"שֶׁלָּכֶם"}`.
2. **בדיקה:** טוקניזציה (הסרת קידומות ו/ה/ב/כ/ל/מ/ש), חיפוש במילון. בגרסה `f` אסור טוקן שמסומן `m` בלבד, וכן הלאה. בגרסה המנוקדת נבדקים גם סיומות ־ךָ/־ךְ.
3. **כלל שוני:** אם `m === f` והטקסט מכיל פנייה בגוף שני → fail (כנראה שכחה). אם הטקסט ניטרלי לגמרי, `n` חייב להיות מלא.
4. **שופט:** קריטריון `gender` ברובריקה (§1.7) תופס מה שהמילון מפספס. מקרה של אי-הסכמה (הקוד עבר, השופט נכשל) → תור הורה.
5. **מבחני snapshot** בצד הלקוח (B3) מרנדרים כל מסך לכל פרופיל.

#### 1.6.3 מספרים ומגדר שמות עצם (V03 + numberCheck)
- `nounLexicon.json`: שם עצם → `{gender, plural, pointed}`; כל שם עצם שמופיע עם slot מספר חייב להיות בו (חסר → fail ובקשה ל-P1 להוסיף ערך, שעובר בדיקה ידנית).

#### 1.6.4 תוכן אסור (V10)
- רשימות ב-§12.3: `security` (B8), `violence`, `fear`, `praise_trait` (C4), `comparison`, `body_food`, `brands`, `personal_info_requests`.
- נרמול: הסרת ניקוד וטעמים, NFC, אותיות סופיות → רגילות, הסרת קידומות (ו, ה, ב, כ, ל, מ, ש ושילובים כמו "וכש") לפני התאמה; התאמה גם על bigram ("בית ספר"). אנגלית: lowercase + stemming פשוט.
- רשימת `allowlist` לחריגים ("פיצוץ בלונים" מותר כשם תבנית? **לא** — משנים את השם ל"קרב בלונים"; עדיף לא לייצר חריגים).

### 1.7 שופט איכות (LLM-as-judge) — מסגרת משותפת

- **מודל:** Sonnet 5.5 שופט תוכן של Opus 5.5 (משפחת מודל אחרת לא זמינה בגלל E8a; לפחות דרג אחר). Batch, cache 1h לרובריקה.
- **פורמט:** ציון שלם 1–4 לכל קריטריון עם עוגנים מילוליים; `verdict` נגזר **בקוד** מהציונים (המודל לא מחליט pass/fail).
- **כיול:** סט כיול של 40 פריטים מתויגים ע"י ההורה (20 טובים, 20 עם פגם מוכר אחד). השופט חייב להגיע ל-≥ 85% זיהוי פגמים ו-≤ 10% דחיות-שווא על הסט לפני הפעלה, ובכל שינוי `prompt_version`. מעקב חודשי על הסכמה עם החלטות ההורה בתור (Cohen's κ ≥ 0.6 יעד).
- **רובריקה (קריטריונים):**

| קריטריון | 1 | 4 | Hard-fail אם |
|---|---|---|---|
| `correctness` | שגיאה עובדתית/מתמטית/תשובה לא חד-משמעית | נכון, תשובה יחידה | < 3 |
| `safety` | הפרה של Safety Core | בטוח לחלוטין לגיל | < 4 |
| `age_fit` | אוצר מילים/מושגים מעל הגיל | מותאם בדיוק לרמה | < 3 |
| `hebrew_naturalness` | עברית מתורגמת/ספרותית/שגויה | עברית ישראלית מדוברת-תקנית לילדים | < 3 |
| `gender` | פנייה במגדר שגוי/מעורב | כל גרסה עקבית | < 4 |
| `pedagogy` | לא מתרגל את הכישור / מסיח שרירותי | מתרגל בדיוק, מסיחים מבוססי טעות | < 3 |
| `praise_policy` | שבח תכונה/סופרלטיב | תהליך/אסטרטגיה בלבד | < 4 (רק אם יש שבח) |
| `cultural_fit` | הקשר זר/לא רלוונטי | ישראלי, חגים, שקלים, אוכל מקומי — בלי קלישאות | < 2 |
| `engagement` | יבש | מצחיק/מסקרן, מתאים לעולם המשחק | — (רק דירוג) |
| `intrinsic_integration` | שאלה "מודבקת" על המשחק | הפעולה הלימודית = הפעולה במשחק (C1) | — (רק דירוג) |

- **כלל החלטה (קוד):** hard-fail כלשהו → `reject` (אם הכשל ב-`hebrew_naturalness`/`engagement` בלבד → `revise`: retry יצירה אחד עם הערות השופט). אחרת ממוצע ≥ 3.2 → `pass`; 2.6–3.2 → `revise`; אחרת `reject`. `pass` + סימון של ולידטור `warn` → תור הורה.

**Judge system prompt (English, stable, cached):**

```text
You are a strict reviewer of learning content for an Israeli family app (children 2.5–8.5; Hebrew-first; English as a subject).
You receive ONE item (JSON) plus its target: age band, skill description, child grammatical gender variants, nikud level.
Score each criterion 1–4 using the anchors below. Be specific in notes, in Hebrew, max 2 short sentences per note.
Do not rewrite the item. Do not output a pass/fail decision. Quote the exact problematic words when a score is below 4.
Check the m/f/pl variants separately for grammatical gender agreement in Hebrew (2nd person pronouns, verbs, adjectives,
imperatives, possessive suffixes). Check that any praise describes effort, process or strategy, never traits or superlatives.
Check that no option or hint reveals the answer. Safety rules: <SAFETY CORE v1 inserted here>.
Anchors: <rubric table rendered as text>.
```

**Judge output schema:**

```json
{
  "type": "object", "additionalProperties": false,
  "required": ["scores", "notes", "flagged_words"],
  "properties": {
    "scores": {
      "type": "object", "additionalProperties": false,
      "required": ["correctness","safety","age_fit","hebrew_naturalness","gender","pedagogy","praise_policy","cultural_fit","engagement","intrinsic_integration"],
      "properties": {
        "correctness": {"type":"integer","enum":[1,2,3,4]},
        "safety": {"type":"integer","enum":[1,2,3,4]},
        "age_fit": {"type":"integer","enum":[1,2,3,4]},
        "hebrew_naturalness": {"type":"integer","enum":[1,2,3,4]},
        "gender": {"type":"integer","enum":[1,2,3,4]},
        "pedagogy": {"type":"integer","enum":[1,2,3,4]},
        "praise_policy": {"type":"integer","enum":[1,2,3,4]},
        "cultural_fit": {"type":"integer","enum":[1,2,3,4]},
        "engagement": {"type":"integer","enum":[1,2,3,4]},
        "intrinsic_integration": {"type":"integer","enum":[1,2,3,4]}
      }
    },
    "notes": {"type":"array","items":{"type":"object","additionalProperties":false,"required":["criterion","note"],
      "properties":{"criterion":{"type":"string","enum":["correctness","safety","age_fit","hebrew_naturalness","gender","pedagogy","praise_policy","cultural_fit","engagement","intrinsic_integration"]},"note":{"type":"string"}}}},
    "flagged_words": {"type":"array","items":{"type":"string"}}
  }
}
```

### 1.8 תור אישור הורה (Parent Review Hooks)

**מצבים:** `draft → validated → judged → (pending_parent | sampled_ok) → approved → published`; כל מצב יכול לעבור ל-`quarantined`.

**מדיניות דגימה:**
- 3 החבילות הראשונות של כל `familyId`/תבנית/סוג סיפור: 100% לעיני ההורה.
- אחרי 3 חבילות נקיות: דגימה 10% (מינימום 3 פריטים לחבילה) + כל פריט עם `warn`/אי-הסכמה בין ולידטור לשופט + כל סיפור + כל תוכן P2.
- כל דחייה של הורה מחזירה את המשפחה ל-100% לחבילה הבאה.
- K3: 100% תמיד (נפח קטן מאוד).

**UX (באזור ההורה, מאחורי Face ID — E7):**
- כרטיס פריט: טקסט מנוקד בגרסה של הילד הרלוונטי + מתג m/f/pl, נגן אודיו (כשמוכן), תשובה, מסיחים עם קוד טעות, ציוני שופט עם הערות, מילים שסומנו (ניקוד/מגדר).
- פעולות בהקשה אחת: **אשר** · **תקן** (עריכה inline; התיקון נשמר כ-`parent_fixed` ומוזן לסט הכיול) · **דחה** (סיבה מרשימה: "עברית לא טבעית", "מגדר", "ניקוד", "לא מתאים לגיל", "מפחיד", "משעמם", "אחר").
- "אשר הכל" זמין רק לאחר צפייה בכל הכרטיסים בדגימה (גלילה עד הסוף).
- התראה עדינה אחת ביום לכל היותר, רק אם יש פריטים ממתינים ל-P2 (שם ההורה מחכה) — לא על P1.
- **כפתור "מוזר לי" בצד הילד** (אייקון סימן שאלה קטן): מעביר את הפריט ל-`quarantined` מיד, מחליף בפריט אחר מאותה משפחה, ומופיע אצל ההורה.

**D1 (תמצית):**

```sql
CREATE TABLE review_queue (
  id TEXT PRIMARY KEY, subject_kind TEXT NOT NULL,       -- item|story|photo_job|audio|asset
  subject_id TEXT NOT NULL, pack_id TEXT, reason TEXT NOT NULL, -- first_packs|sample|warn|disagreement|p2|story|kid_flag
  status TEXT NOT NULL DEFAULT 'pending',                -- pending|approved|fixed|rejected
  parent_note TEXT, decided_at TEXT, created_at TEXT NOT NULL
);
CREATE TABLE ai_calls (
  id TEXT PRIMARY KEY, job TEXT, model TEXT, served_model TEXT, prompt_version TEXT, schema_version TEXT,
  input_tokens INT, output_tokens INT, cache_read INT, cache_write INT, usd REAL, latency_ms INT,
  failure TEXT, subject_ref TEXT, created_at TEXT
);
CREATE TABLE cost_ledger (day TEXT, job TEXT, usd REAL, PRIMARY KEY(day, job));
```

### 1.9 עלויות, מטמון ותקרה

**מחירון (מאומת 2026-11-02, $/MTok):**

| מודל | קלט | כתיבת cache 5m / 1h | קריאת cache | פלט | Batch קלט/פלט |
|---|---|---|---|---|---|
| Opus 5.5 | 4.00 | 5.00 / 8.00 | 0.20 | 20.00 | 2.00 / 10.00 |
| Sonnet 5.5 | 2.00 | 2.50 / 4.00 | 0.10 | 10.00 | 1.00 / 5.00 |
| Haiku 5.5 (≤100K) | 0.10 | 0.125 / 0.20 | 0.01 | 0.50 | 0.05 / 0.25 |

**כללי מטמון:**
- סדר קבוע: `system = [SAFETY_CORE, JOB_INSTRUCTIONS, STYLE_GUIDE, FEW_SHOTS]` → breakpoint אחד בסוף הבלוק האחרון (TTL 1h ב-Batch, כפי שהתיעוד ממליץ; 5m ב-realtime עם תעבורה צפופה). תוכן משתנה (פריט, תמונה, פרופיל) רק ב-`messages`.
- אסור בתוך ה-prefix: תאריכים, מזהים, JSON לא ממוין. בדיקת CI: hash של ה-prefix לכל `prompt_version` קבוע.
- מינימום prefix נשמר: 512 טוקנים ב-5.5 — כל system prompt כאן מעליו.
- תמונות: כיווץ לצלע ארוכה 2,576 px (שכבת high-res של Claude 4.7+, עד 4,784 visual tokens ≈ 4.8K טוקנים לתמונה); שליחת יותר פיקסלים לא מוסיפה דיוק ומבזבזת העלאה.

**תקרה (E8) — מכונת מצבים ב-`costGuard`:**

| פרמטר | ברירת מחדל | איפה נקבע |
|---|---|---|
| `DAILY_USD_CAP` | **$3.00** למשפחה-יום (מתאפס 00:00 Asia/Jerusalem) | Worker secret/config; ההורה יכול לשנות ב-PA-SET (10) בטווח $1–$10 |
| `MONTHLY_USD_CAP` | **$40** | כנ"ל, טווח $10–$100 |
| תקרה קשיחה בקונסול Anthropic | **$50/חודש** (spend limit ב-workspace) | ההורה, פעם אחת (13 §16.3). זו רשת הביטחון מעל התקרה הרכה |
| `OVERRIDE_USD` | $5 נוספים ליום, לכל היותר 3 פעמים בחודש | Face ID במסך P2 בלבד |

| מצב (לפי `spent_today / DAILY_USD_CAP`) | P1 (Batch) | P2 צלם-ושחק | P3 סוקרטי | P4 | מה ההורה רואה |
|---|---|---|---|---|---|
| **< 80%** | רץ | רץ | רץ | רץ | — |
| **80–100%** | עבודות חדשות נדחות ליום הבא (עבודות שכבר הוגשו ממשיכות) | רץ | רץ | רץ | שורה שקטה ב-PA-HOME: "AI: 80% מהתקציב היומי" |
| **≥ 100%** | נדחה | מסך האישור עובד, אבל "צור משחק" מציג: "המשחק יוכן מחר בבוקר (06:00)" + כפתור "להכין עכשיו" (Face ID, מנצל `OVERRIDE_USD`) | **שורות מתוסרטות בלבד**; לילד אין שום סימן לשינוי (הרובוט "חושב קצת פחות היום") | רץ (עלות זניחה, לא נספרת בתקרה היומית) | התראה אחת ב-PA-INBOX; אימייל אם זה קורה 3 ימים ברצף |
| **חודשי ≥ 100%** | מושבת עד ה-1 בחודש | רק עם override | מתוסרט בלבד | רץ | התראה + הצעה להעלות תקרה |
| **kill switch (`AI_ENABLED=false`)** | מושבת | "לא זמין כרגע" | מתוסרט | קוד בלבד | ההורה הפעיל ידנית (S0 או חשד לדליפה) |

- כל שינוי מצב נרשם ב-`cost_ledger` + שורת audit. עבודות P2 שנדחו נכנסות לתור `deferred_jobs` ורצות ב-06:00 למחרת (לפני ה-cron של P1), בסדר FIFO.
- ההערכה לפני קריאה: `costGuard.assertAllowed` משתמשת ב**אומדן עלות עליון** לכל job (טבלה ב-`ROUTES`, למשל `photo.extract` = $0.08 לדף) כדי שלא "נגלוש" מעבר לתקרה בקריאה אחת.
- בדיקת אינטגרציה (§10): סימולציה של יום עם 40 תמונות → המעבר ל-80%/100% קורה בדיוק בפריט הנכון; override נספר; איפוס בחצות.

---

## 2. P1 — יצירת תוכן לתוכנית הלימודים

### 2.1 ארכיטקטורה

```
 [Skill graph + "מה למדו החודש" (parent input) + coverage gaps from engine]
                     │  nightly Cron `10 0 * * *` UTC = 02:10 IL (11 §7.1 owns the schedule)
                     ▼
      ┌──────────── Planner (code) ────────────┐
      │ which skill nodes need items/stories   │  target: ≥40 live variants per node, ≥7 days ahead
      └──────────────┬─────────────────────────┘
                     ▼
   ┌───────── AIG core (code, packages/content/aig) ─────────┐
   │ item model → operands/words → answer → coded distractors │  correctness by construction
   └──────────────┬──────────────────────────────────────────┘
                  ▼
   Batch #1  Opus 5.5  "skin" / "story" / "hints"  (Hebrew, no nikud, slots for numbers/names)
                  ▼  (poll every 15 min via Cron; ≤24h)
   Code: slot fill (numeral(), names) → V01–V16 validators
                  ▼
   Media runner: Phonikud + Nakdan nikud, cross-check (V06–V08)
                  ▼
   Batch #2  Sonnet 5.5 judge (rubric §1.7)
                  ▼
   Review policy (§1.8) ──► parent queue (sampled) ──► approved
                  ▼
   P5 voice jobs (TTS render + QA) ──► audio manifest
                  ▼
   Pack builder: canonical JSON + sha256 + R2 `packs/…` ──► devices pull (E6)
```

### 2.2 שלבים

1. **Planner (קוד):** לכל צומת כישור פעיל לכל ילד — חישוב מלאי (`live_variants`), צריכה צפויה (ממוצע 14 יום), ופער. יעד: ≥ 40 וריאנטים חיים לצומת (content_world §a1: 30–50) ומלאי ל-7 ימים קדימה. צמתים מ"מה למדו החודש" מקבלים עדיפות.
2. **AIG (קוד):** לכל צומת יש `ItemModel` (§2.3). מתמטיקה — אינסופי מקוד. קריאה/כתיב — מאגרי מילים מתויגים (`lexicon/he-k2-decodable.json`, `lexicon/en-phonics-*.json`). מסיחים מקטלוג הטעויות (§3.7, §12.2).
3. **Batch #1 (Opus 5.5):** עד 200 בקשות ללילה (כל בקשה = 5–10 פריטים מאותו item model, כדי לשתף few-shot). `custom_id = {jobRunId}:{skillId}:{chunk}`.
4. **מילוי slots + ולידטורים (Worker).**
5. **ניקוד (runner).**
6. **Batch #2 (Sonnet 5.5 judge)** — פריט לכל בקשה (בידוד), rubric ב-cache 1h.
7. **החלטה + תור הורה.**
8. **P5** — רינדור קולי לכל פריט שאושר (רק שורות שעדיין אין להן key ב-cache).
9. **בניית חבילה** ופרסום. גרסה חדשה של חבילה לא מוחקת פריטים בשימוש (SRS) — רק מוסיפה/מסגירה.

### 2.3 Item models (AIG) — דוגמאות

```ts
export interface ItemModel<P> {
  id: string;                         // "math.k1.mult.table-6-9.fact"
  skillId: string;
  ageBands: AgeBand[];
  difficulty: 1 | 2 | 3 | 4 | 5;
  sample(rng: Rng, ctx: ChildCtx): P;          // parameters
  core(p: P): { prompt: MathExpr | WordTask; answer: string };
  distractors(p: P, k: number): { value: string; errorCode: string }[];  // from error catalog
  skinSpec?: SkinSpec;                // if an LLM narrative wrapper is wanted
}

// Example 1 — K1 multiplication word problem (grade 3)
export const multStory: ItemModel<{ a: number; b: number }> = {
  id: "math.k1.mult.story", skillId: "math.mult.6-9", ageBands: ["K1"], difficulty: 3,
  sample: (rng) => ({ a: rng.int(6, 9), b: rng.int(3, 9) }),
  core: ({ a, b }) => ({ prompt: { a, op: "×", b, blank: "result", answer: a * b }, answer: String(a * b) }),
  distractors: ({ a, b }, k) => pick(k, [
    { value: String(a * (b - 1)), errorCode: "MATH_TABLE_NEIGHBOR" },
    { value: String(a * (b + 1)), errorCode: "MATH_TABLE_NEIGHBOR" },
    { value: String(a + b),       errorCode: "MATH_OP_SWAP_ADD" },
    { value: reverseDigits(a * b), errorCode: "MATH_DIGIT_REVERSAL" },
  ].filter(d => d.value !== String(a * b) && +d.value > 0)),
  skinSpec: { frame: "multiplicative_groups", slots: ["a", "b"], world: "campfire_camp", maxWords: 35 },
};

// Example 2 — K2 first-sound (Hebrew, letter sounds before names — B6)
// sample: a decodable CV/CVC word whose first letter ∈ learnedLetters, from he-k2-decodable lexicon
// distractors: 2 letters from HEB_CONFUSABLE_SOUND (ב/פ, ד/ט, כ/ק, ס/ש) + 1 visually similar (ב/כ, ד/ר, ה/ח, ו/ז) — all ∈ learnedLetters
// no LLM skin; audio = word + "ב… בּ…" scaffold lines (pre-rendered)
```

### 2.4 פרומפט Batch #1 — "skin" לבעיות מילוליות

**System (עברית; יציב; cached):**

```text
<SAFETY CORE v1>

את/ה כותב/ת עטיפות סיפוריות קצרות לתרגילים במשחק לימודי משפחתי ישראלי.
העולם: "המחנה של המדורה" — יער בלילה נעים (לא מפחיד), מדורה משותפת, בונים מחנה, אוספים קריסטלים ועצים,
והצללונים (יצורים עגולים ומצחיקים) מבלגנים מספרים ואותיות. כשפותרים — הם הופכים לחברים. אף אחד לא נפגע.

כללי כתיבה מחייבים:
1. עברית ישראלית מדוברת-תקנית, כמו בספר ילדים טוב. משפטים קצרים. בלי תרגומית ובלי לשון ספרותית גבוהה.
2. אל תכתוב/י ניקוד. הניקוד מתווסף אוטומטית.
3. אל תכתוב/י מספרים בכלל — לא בספרות ולא במילים. במקום כל מספר כתוב/כתבי slot בדיוק בצורה {{a}} או {{b}}
   מיד לפני שם העצם שהוא סופר, למשל: "{{a}} מגדלים". אל תכתוב/י את התשובה ואל תרמוז/י עליה.
4. השתמש/י רק בשמות עצם מרשימת NOUNS שבהודעה, כדי שהמערכת תדע את המין הדקדוקי שלהם. **שם עצם שנספר ע"י slot מועתק מ-NOUNS בדיוק כפי שהוא (צורת יחיד, בלי נטייה, בלי ריבוי, בלי סמיכות)** — המערכת מטה אותו בעצמה (`numPhrase`, 03 §10.3). אל תספור/תספרי שם עצם שאינו ב-NOUNS.
5. כתוב/כתבי שלוש גרסאות נפרדות של כל פנייה לקורא/ת: m (לבן), f (לבת), pl (לכמה ילדים).
   אם אפשר לנסח בלי פנייה מגדרית (שם פועל, "בואו נ…", או {{name}}), כתוב/כתבי גם n — זו הגרסה המועדפת.
   אסור לערבב מגדרים בתוך גרסה. דמויות במשחק לפי הרשימה CHARACTERS (כולל המין שלהן) — התאם/התאימי פעלים ותארים.
6. הפעולה במשחק היא החשבון: הכפל/החיבור הוא מה שהדמות עושה בסיפור (לבנות, לחלק, לאסוף), לא חידון שמודבק.
7. שבח (אם מתבקש): רק על תהליך ואסטרטגיה ("ספרת בקבוצות", "בדקת שוב"). אסור: גאון, אלוף, חכם, מושלם, הכי, מדהים.
8. בלי מותגים, בלי אנשים אמיתיים, בלי מסכים, בלי אוכל לא כשר באופן בולט (בלי בייקון/שרימפס), בלי חגים לא יהודיים
   כאירוע מרכזי. מותר: שקלים, פלאפל, מסיק זיתים, חגים יהודיים, נופים ישראליים.
9. אם הבקשה לא ניתנת לכתיבה לפי הכללים — החזר/י status "rejected" עם סיבה.

<STYLE GUIDE: 12 approved example skins (few-shot), each with the item-model frame they illustrate>
```

**User (לכל chunk):**

```text
TASK: write {{count}} skins.
ITEM_MODEL: {{itemModelId}} — frame: {{frame}} (e.g. multiplicative_groups = "a groups of b things each; how many in total")
AGE_BAND: K1 (grade 3, reads pointed Hebrew well)   MAX_WORDS_PER_SKIN: {{maxWords}}
SLOTS: a = number of groups, b = items per group. The question asks for the total.
NOUNS (use only these; gender is known to the system): מגדל, קריסטל, אוהל, ענף, פנס, סל, ערימה, צללון
CHARACTERS: אורי (m, builder), נגה (f, scout), רובוט-העזר גלגלי (machine; never speaks emotions)
INTERESTS (optional flavor, use at most one per skin): {{interests}}   // e.g. "בנייה", "גיבורים", from profile, no names
Return JSON per schema.
```

**Output schema (Claude subset):**

```json
{
  "type": "object", "additionalProperties": false,
  "required": ["status", "reason", "skins"],
  "properties": {
    "status": {"type": "string", "enum": ["ok", "rejected"]},
    "reason": {"type": "string"},
    "skins": {"type": "array", "items": {
      "type": "object", "additionalProperties": false,
      "required": ["setup", "question", "nouns_used", "ambiguous_words", "praise"],
      "properties": {
        "setup":    {"type": "object", "additionalProperties": false, "required": ["m","f","pl","n"],
                     "properties": {"m":{"type":"string"},"f":{"type":"string"},"pl":{"type":"string"},"n":{"type":"string"}}},
        "question": {"type": "object", "additionalProperties": false, "required": ["m","f","pl","n"],
                     "properties": {"m":{"type":"string"},"f":{"type":"string"},"pl":{"type":"string"},"n":{"type":"string"}}},
        "nouns_used": {"type": "array", "items": {"type": "string"}},
        "ambiguous_words": {"type": "array", "items": {"type": "object", "additionalProperties": false,
                     "required": ["word", "intended_reading_latin"],
                     "properties": {"word":{"type":"string"},"intended_reading_latin":{"type":"string"}}}},
        "praise": {"type": "object", "additionalProperties": false, "required": ["m","f","pl","n"],
                     "properties": {"m":{"type":"string"},"f":{"type":"string"},"pl":{"type":"string"},"n":{"type":"string"}}}
      }
    }}
  }
}
```

**עיבוד בקוד:** `skins.length === count` (אחרת חסרים נבקשים שוב בלילה הבא); כל `{{a}}`/`{{b}}` מופיע בדיוק פעם אחת ולפני שם עצם מ-NOUNS; אין ספרות/מילות מספר (`V03`); `numeral(a, nounGender, "attributive")` ממלא; V05/V09/V10/V11/V12/V13/V14.

### 2.5 פרומפט Batch #1 — סיפורונים/קטעי קריאה

**System (בנוסף ל-SAFETY CORE ולכללים 1–2, 5, 7–9 מ-§2.4):**

```text
את/ה כותב/ת סיפורונים לקריאה לילדים ישראלים, במשחק משפחתי. כל סיפור:
- 3–12 עמודים לפי AGE_BAND. משפט אחד עד שלושה בעמוד. רעיון אחד בכל משפט.
- עלילה פשוטה: רצון → בעיה קטנה → ניסיון → פתרון בעזרת חשיבה או שיתוף פעולה. סוף חם ורגוע (מתאים גם לפני שינה).
- דמויות רק מרשימת CHARACTERS עם המין שלהן. אל תמציא/י בני אדם חדשים; מותר חיות ויצורים מצחיקים.
- K2: השתמש/י רק במילים מרשימת ALLOWED_WORDS (מילים שהילד יכול לפענח) ובמילות-פונקציה מרשימת FUNCTION_WORDS.
  אם אין מספיק מילים לסיפור טוב — קצר/י את הסיפור; אל תוסיף/י מילים מחוץ לרשימה.
- K1: אוצר מילים של כיתה ג'. שלב/י 2–4 מילים מתוך TARGET_WORDS (מילות כתיב או אוצר מילים שהיא לומדת).
- אין שאלות בתוך הסיפור. את שאלות ההבנה כתוב/כתבי בנפרד בשדה comprehension, 3 שאלות:
  אחת "מי/מה", אחת "למה", אחת "מה קרה קודם/אחר כך". לכל שאלה תשובה נכונה אחת ושני מסיחים
  שמופיעים בסיפור אבל אינם התשובה.
- לכל עמוד תאר/י סצנה לאיור בשדה scene בעזרת אוסף SCENE_TAGS בלבד (למשל: forest_night, campfire, tent, river,
  village, kitchen, gan_yard) ותגיות דמויות. אין צורך לתאר מראה אנשים.
```

**User:** `AGE_BAND`, `NIKUD_LEVEL`, `ALLOWED_WORDS`/`TARGET_WORDS`, `CHARACTERS`, `THEME` (מתוך לוח שנה: חג קרוב לפי B7, או נושא מ"מה למדו החודש"), `SCENE_TAGS`.

**Output schema:** `{status, reason, title{plain}, pages[{text{m,f,pl,n}, scene{bg, characters[], props[]}}], comprehension[{q{m,f,pl,n}, correct, distractors[]}], target_words_used[], ambiguous_words[]}` — כל השדות required, מבנה כמו §2.4.

**עיבוד:** **מיפוי 1:1 ללקסיקון (K2, 04 §5.5):** כל טוקן בפלט, אחרי הסרת קידומות מותרות (ו/ה/ב/כ/ל/מ/ש), חייב להתאים **בדיוק** לרשומת `unpointedMale` אחת ב-`lexicon.he.json` (ALLOWED_WORDS = רשימת `lexId` → `unpointedMale`); **הטקסט המנוקד של K2 מועתק מהלקסיקון (`pointed`) ולעולם לא נוצר ע"י Nakdan/Phonikud**; טוקן בלי התאמה מדויקת = V08 fail → revise עם רשימת הטוקנים. V08 decodability (K2 — כל מילה ⊆ ALLOWED_WORDS ∪ FUNCTION_WORDS אחרי הסרת קידומות; כשל אחד = הסיפור חוזר ל-revise עם רשימת המילים הבעייתיות), V09, V10; `scene` ממופה ל-`imageRef` מתוך ספריית רקעים ומדבקות (P6) — **אין יצירת תמונה בזמן ריצה**.

### 2.6 פרומפט Batch #1 — סולמות רמזים ושבח

- לכל `familyId` (לא לכל פריט): 3 רמזים כלליים (דחיפה → אסטרטגיה → צעד מודגם על מספרים אחרים), עם slots.
- הרמז השלישי ("צעד מודגם") משתמש ב-**דוגמה מקבילה** עם מספרים אחרים שהקוד מספק (`{{c}}`, `{{d}}`), לעולם לא בפריט עצמו.
- שבח: 8 שורות תהליך לכל סוג פעולה (`counted_in_groups`, `checked_again`, `tried_new_way`, `fixed_mistake`, `kept_going`), m/f/pl/n. נבדקות ב-V11 ונכנסות לרשימת שבח מאושרת (UX spec) רק אחרי אישור הורה.

### 2.7 ניסיונות חוזרים ונפילות

| כשל | פעולה |
|---|---|
| `refusal` | לא חוזרים. נרשם, ה-skin נזרק; הפריט נשאר ב"ליבת AIG בלבד" (מוצג בלי עטיפה) |
| `max_tokens` | retry אחד עם `max_tokens×2` בבאץ' הבא |
| `schema/validation` | retry אחד עם הודעת שגיאה מתומצתת (רשימת ולידטורים שנכשלו), אחר כך זריקה |
| שופט `revise` | retry אחד עם הערות השופט; אחר כך `reject` |
| Batch `expired`/`errored` | הגשה מחדש בלילה הבא (לא מחויב על expired) |
| Nikud: Nakdan לא זמין | Phonikud בלבד + כל המילים שמעל סף אי-ודאות → דגימת הורה 100% לחבילה |
| יותר מ-30% reject בחבילה | עצירת ה-item model, התראה בלוח ההורה/מפתח ("תבנית X מייצרת חומר חלש"), תיקון few-shot |

### 2.7.1 טיפול בכשלי QA בהיקף (content QA failure handling)

| אות | סף | פעולה אוטומטית | מי מקבל הודעה |
|---|---|---|---|
| **השופט דוחה הרבה** | > 30% `reject` ב-item model אחד **או** > 15% בכל הלילה | ה-item model/הלילה מסומנים `paused`; החבילה לא נבנית; הפריטים שעברו נשמרים כ-`draft` | שורת STATUS לסוכן; ההורה לא מוטרד (יש מלאי ל-7 ימים) |
| **השופט מאשר הכול** | 0 `revise/reject` על ≥ 100 פריטים בלילה | חשד לשופט "רדום": הרצת סט הכיול (40 פריטים) מיד; אם זיהוי הפגמים < 85% → השופט `paused`, החבילה ל-100% אישור הורה | סוכן |
| **סחף כיול** | κ חודשי מול החלטות ההורה < 0.6 | גרסת פרומפט חדשה לשופט + כיול מחדש לפני חזרה לדגימה | סוכן; שורה בדוח השבועי |
| **ההורה דוחה** | ≥ 2 דחיות באותה חבילה, או דחייה אחת בסיבה "מפחיד"/"מגדר" | המשפחה חוזרת ל-100% אישור לחבילה הבאה (§1.8); סיבה "מגדר" → S2 (13 §10.1): כל הפריטים של אותו prompt_version נבדקים מחדש ב-`genderCheck` | סוכן |
| **"מוזר לי" מהילד** | 1 דגל → הפריט בהסגר; ≥ 3 דגלים בשבוע מאותו `familyId` (item family) → כל המשפחה בהסגר | החלפה שקטה מפריטים מאושרים אחרים; אם נשארו < 10 פריטים לצומת → המנוע עובר לצומת שכן | ההורה: כרטיס "הילד סימן" ב-PA-APPR, בלי לחץ |
| **S0: תוכן פוגעני/שגוי הגיע לילד** | מקרה אחד | `packs/<id>` מסומן `revoked` ב-manifest; המכשירים מורידים manifest חדש בסנכרון הבא (≤ 5 דק׳ כשמקוון) ומסתירים את החבילה; אם יש OTA קשור — republish של הגרסה הקודמת (13 §10.1) | ההורה מיד (push אם יש, אחרת אימייל + PA-INBOX); הסוכן פותח ADR תוך 24 שעות |
| **ניקוד שגוי בדגימה** | > 1 שגיאה ל-100 מילים (K2) | החבילה ל-`revise`; כל המילים שלא ב-lexicon עוברות Nakdan+Phonikud מחדש; מילים חדשות נכנסות ל-lexicon רק אחרי ✓ הורה | סוכן |
| **אודיו נכשל ב-QA** | > 5% שורות עם CER > 0.15 אחרי 3 takes | החבילה מתפרסמת **בלי** השורות האלה (K1: טקסט בלבד; K2/K3: הפריט לא נכלל); רשימת "לתיקון הגייה" להורה | ההורה (האזנה קצרה) |
| **מלאי נמוך** | < 14 ימי תוכן לצומת פעיל (RK-23) | Planner מכפיל את מכסת הלילה לצומת; אם אחרי 3 לילות עדיין נמוך → סוכן | סוכן |

כל האירועים האלה נרשמים ב-`content_incidents` (D1) ומסוכמים בדוח השבועי לסוכן; ההורה רואה רק את הסיכום ("3 פריטים הוחלפו השבוע") אלא אם נדרש ממנו משהו.

### 2.8 עלות וזמן

- **נפח צפוי (חודש ייצור):** ~3,000 פריטים + 60 סיפורים + 40 סולמות רמזים. Skins: 300 בקשות × (3K טוקני system ב-cache-hit + 0.5K קלט + 2.5K פלט) ≈ 0.15M קלט + 0.9M cache-read + 0.75M פלט → ≈ $0.3 + $0.2 + $7.5 = **≈ $8**. סיפורים: 60 × 6K פלט ≈ $3.6. שופט: 3,000 × (2K cache + 1K קלט + 0.4K פלט) ≈ $3 + $6 = **≈ $9**. סה"כ P1 ≈ **$20/חודש ייצור, $5–8 במצב יציב** (תואם ai_models_assets).
- **Latency:** אין יעד מקוון. יעד תפעולי: חבילה ממתינה לאישור עד 07:00 בבוקר; זמינה במכשיר תוך ≤ 10 דקות מאישור.

---

## 3. P2 — "צלם וקבל משחק" (Photo-to-Game)

### 3.1 ארכיטקטורה

```
 PARENT DEVICE (parent area, Face ID)                         WORKER (realtime)                         MEDIA RUNNER
 ┌──────────────────────────────────┐
 │ 1 capture (doc scanner, ≤4 pages)│
 │ 2 crop/rotate (parent drags)     │
 │ 3 on-device privacy pass:        │
 │   face blur · EXIF strip ·       │
 │   orientation bake · resize 2576 │
 │ 4 tag: source kind + mode +      │
 │   which child(ren)               │──POST /v1/photo-jobs──►  5 store R2 photos/ (private, TTL≤30d)
 └──────────────────────────────────┘                          6 extract: Sonnet 5.5 vision (+ Opus 5.5 in
                                                                 parallel if handwritten) → ExtractionResult
                                                               7 code: V17/V18/V02 · reconcile dual reads ·
                                                                 PII → placeholders · nikud (Nakdan) ──────────► (fallback: Phonikud job)
 ┌──────────────────────────────────┐◄── push "ready to confirm" ─┘
 │ 8 CONFIRM SCREEN (photo ⟷ items) │
 │   fix words · mark answer key ·  │
 │   pick mode/children · approve   │──POST /confirm──►  9 template select (code) · distractors (code) ·
 └──────────────────────────────────┘                     10 enrich (Haiku 5.5: per-child variants, hints) ·
                                                           11 validators · TTS render (Azure/bake-off winner) ·
                                                           12 pack build + sha256 → R2 → notify devices
 CHILD DEVICE: 13 game appears as a map node per 09 F4 (label "משחק חדש מהמחברת!"; K2: next episode's review slot) · items → SRS queue (+1/+3/+7 …, D9)
               gender/variant change later → job `photo.rerender_variants` (renders the missing m/f/pl lines, silent download; 10 AC-25 scope)
                                                                               nightly: loudness re-normalize ──► (runner)
```

### 3.2 טיפים לצילום (טקסט UX להורה, מוצג בפעם הראשונה ובכפתור "?")

> **איך מצלמים כדי שיצא משחק טוב**
> 1. דף אחד בכל צילום, ישר מלמעלה, באור יום או מנורה — בלי צל של הטלפון.
> 2. דף מודפס עובד הכי טוב. כתב יד של ילדים — אפשר, אבל תצטרכו לבדוק את המילים.
> 3. אפשר לחתוך רק את התרגיל. כדאי להשאיר בחוץ שמות של ילדים אחרים, המורה ותמונות.
> 4. אם בדף יש תשובות שהילד כבר כתב — זה בסדר, נשתמש בהן רק בשבילכם.
> 5. ניקוד בתמונה לא נקרא — אנחנו מוסיפים ניקוד מחדש ואתם מאשרים.
> 6. עד 4 דפים לאותו משחק. צילמתם יותר? נחלק למשחקים נפרדים.
> 7. דף מבחן או מבדק? נכין תרגול דומה בלי התשובות של הדף.
> 8. דף עם עברית וגם אנגלית יוצא כשני משחקים: אחד בעברית ואחד באי האנגלי.
> 9. אין אינטרנט? מצלמים בכל זאת — הדפים נשלחים לבד כשיש חיבור.

### 3.3 עיבוד על המכשיר (לפני העלאה)

| שלב | מימוש | הערות |
|---|---|---|
| צילום + תיקון פרספקטיבה | `react-native-document-scanner-plugin` (VisionKit `VNDocumentCameraViewController`) דרך config plugin ב-EAS; fallback `expo-image-picker` + חיתוך ידני | מרובה דפים |
| חיתוך | מסך חיתוך Skia עם 4 פינות + "בחר אזור תרגיל" (מלבן נוסף) | ההורה יכול לסמן 1–3 אזורי "החרג" |
| טשטוש פנים | Expo Module מקומי `modules/vision-privacy` (Swift): `VNDetectFaceRectanglesRequest` → מלבנים מורחבים ב-20% → Skia `Blur` (σ=25) + פיקסליזציה | רץ גם על תמונות/איורים בדף; אין שליחה לשום מקום לפני זה |
| טשטוש שמות (אופציונלי) | אותו מודול: `VNRecognizeTextRequest` אם `supportedRecognitionLanguages` כולל he; התאמה מול רשימת "שמות לא-משפחה" שההורה הזין | אם עברית לא נתמכת — מדולג, והטיפול בשרת (V18) |
| כיוון | אפיית EXIF orientation לפיקסלים **לפני** הסרת EXIF | אחרת הדף מגיע מסובב (Claude חלש בטקסט מסובב) |
| הסרת EXIF | קידוד מחדש עם `expo-image-manipulator` (JPEG) — הפלט לא נושא GPS/EXIF; בשרת בדיקה כפולה: parser שמסיר segments `APP1`/`APP13` | E5 |
| שינוי גודל | צלע ארוכה 2,576 px (שכבת high-res של Claude); JPEG quality 0.88; לא grayscale | דחיסה כבדה פוגעת בקריאת טקסט |
| תיוג | `sourceKind` (דף עבודה / מחברת / שיעורי בית להגשה / חומר למבחן / סיפור שהילד כתב / דף מספר), `forChildren[]`, `handwritten?` (ההורה יכול לסמן; אחרת המודל קובע) | |

### 3.4 קליטה בשרת

- `POST /v1/photo-jobs` (multipart, ≤ 4 × 3 MB): אימות סשן הורה; בדיקת MIME; הסרת EXIF כפולה; R2 `photos/{familyId}/{jobId}/{page}.jpg` עם lifecycle 30 יום (E5). ברירת מחדל: **מחיקה מיידית אחרי אישור ההורה** (מתג "שמור את הצילום לעיון" כבוי).
- רישום `photo_jobs` (D1): `status = uploaded → extracting → needs_confirm → building → ready | failed`.

### 3.5 חילוץ (Extraction)

#### 3.5.1 ניתוב
- `handwritten=false` (מודפס): `photo.extract` — Sonnet 5.5, תמונה אחת לבקשה (דף), streaming, `effort:"medium"`.
- `handwritten=true` או שהמודל דיווח `handwriting_ratio ≥ 0.3`: **ברירת מחדל (00 E8): מסלול יחיד Sonnet 5.5** + מסך אישור חובה. **הקריאה הכפולה** (Opus 5.5 במקביל, פיוס בקוד §3.5.5) נדלקת בדגל `photo.dualRoute=true` **רק** אם הערכת 10 דפי כתב היד של המשפחה (OPD-20, §10) עומדת בסף: **כיסוי מילים נכונות עולה ב-≥ 0.10 מוחלט** (למשל 0.45 → 0.55) **ובלי עלייה בשגיאות "שקטות"** (`confidence:high` שגוי); עלות הדף המותנית ≤ $0.12. (רקע: בלוח heb-ocr, Opus 5 הוביל את Sonnet 5 בכיסוי מילים בעמוד כתב יד מלא: 0.335 מול 0.210; ⟂VERIFY על הסט של המשפחה — מספרי benchmark אינם תחליף.) ההחלטה נרשמת ב-ADR; בלי ADR הדגל כבוי.
- דף צפוף (> 25 שורות טקסט לפי קריאה ראשונה, או `V17` נכשל): חיתוך לשני חצאים עליון/תחתון עם חפיפה של 10%, כל חצי ב-2,576 px, ואיחוד עם הסרת כפילויות בחפיפה.

#### 3.5.2 טיפים לעברית ב-Claude vision (מיושמים בפרומפט ובקוד)
1. התמונה **לפני** הטקסט בהודעה (המלצת התיעוד).
2. רזולוציה מלאה של שכבת high-res (2,576 px) + חיתוך לאזור התרגיל — במקום דחיסה.
3. לבקש תעתוק "כפי שכתוב", בסדר לוגי (מימין לשמאל בעברית, מספרים ואנגלית משמאל לימין), **בלי תיקון כתיב** ובלי השלמת מילים.
4. לבקש **אותיות בלבד, בלי ניקוד** (`has_nikud` רק כדגל). ניקוד מחדש ב-Nakdan.
5. סימון `[?]` במקום ניחוש לאות לא קריאה, ו-`confidence` לכל פריט.
6. ספרות מערביות (0–9) לכל מספר; ביטוי מתמטי כטוקנים מובנים, לא כמחרוזת.
7. הפרדה בין טקסט מודפס לכתב יד, ובין "שאלה" ל"תשובה שהילד כתב".
8. מגבלת `max_tokens` + בדיקת לולאות חזרה (V17) — כשל מוכר של VLM בעברית בכתב יד.
9. אף פעם לא לבקש לזהות אנשים (Claude מסרב, וזה גם לא נדרש).

#### 3.5.3 Extraction system prompt (English; stable; cached)

```text
<SAFETY CORE v1 — short form: refuse nothing that is a normal school page; never describe or identify people.>

You transcribe and structure a photographed Israeli school/kindergarten page (Hebrew, sometimes English and math)
so that a parent can turn it into a practice game. You are not solving the page.

Rules:
1. Transcribe exactly what is written, in logical reading order (Hebrew right-to-left; numbers and English left-to-right).
   Do NOT correct spelling, do NOT complete words, do NOT translate. Children's own writing must stay as written.
2. Hebrew: output consonants only, without nikud, even if the page has nikud. Set has_nikud=true if the page shows nikud.
   Use final letters as written on the page.
3. If a letter or word is unreadable, write [?] in its place and add the span to uncertain_spans. Prefer [?] over guessing.
4. Numbers: Western digits 0-9. Math exercises: fill the math object with left, op, right as written and blank_position;
   leave math.result_as_written as written on the page (it may be a child's answer, possibly wrong). Never compute answers.
5. Separate printed text from handwriting (is_handwritten per item). If a child already wrote an answer on the page,
   put it in child_answer_as_written; never treat it as the answer key.
6. Personal names other than story characters (classmates, teacher, school, phone numbers): list them in pii_spans and
   replace them in the text with the token [NAME] / [SCHOOL] / [PHONE].
7. Classify the page into exactly one content_type (see definitions) and give per-item shapes.
8. Output at most 60 items. If the page has more, take them in reading order and set truncated=true.
9. If the page is not a learning page (e.g., a photo of a person, a screen, something inappropriate), set
   content_type="not_learning_material" and return no items.

content_type definitions:
- vocabulary_list: words or word↔picture/translation pairs (Hebrew or English).
- spelling_list: words to practice spelling ("הכתבה", "מילים לשינון").
- letters_nikud: letters, syllables or nikud signs to practice (gan / grade 1).
- math_set: arithmetic exercises (expressions with blanks), number bonds, counting tasks.
- word_problems: math story problems.
- reading_passage: a text with or without comprehension questions.
- child_story: a story the child wrote (handwritten or dictated).
- facts: statements/questions about science, holidays, homeland (מולדת), Torah stories, etc.
- mixed: several of the above; use per-item shapes.
```

**User message:** `[image block(s)]` ואז טקסט: `PAGE {{i}}/{{n}}. Parent tags: source_kind={{sourceKind}}, handwritten_hint={{hw}}, children={{K1,K2}} (ages only). Return JSON.`

#### 3.5.4 ExtractionResult — Output schema (Claude subset)

```json
{
  "type": "object", "additionalProperties": false,
  "required": ["content_type","primary_language","has_nikud","handwriting_ratio","truncated","page_title","items","pii_spans","uncertain_spans","notes_for_parent"],
  "properties": {
    "content_type": {"type":"string","enum":["vocabulary_list","spelling_list","letters_nikud","math_set","word_problems","reading_passage","child_story","facts","mixed","not_learning_material"]},
    "primary_language": {"type":"string","enum":["he","en","mixed"]},
    "has_nikud": {"type":"boolean"},
    "handwriting_ratio": {"type":"string","enum":["none","some","mostly","all"]},
    "truncated": {"type":"boolean"},
    "page_title": {"type":"string"},
    "items": {"type":"array","items":{
      "type":"object","additionalProperties":false,
      "required":["idx","shape","lang","text","paired_text","options","category","order_index","math","child_answer_as_written","is_handwritten","confidence"],
      "properties":{
        "idx":{"type":"integer"},
        "shape":{"type":"string","enum":["word","pair","qa","cloze","sequence_step","category_member","math_expr","statement","passage_paragraph","story_sentence","letter_or_syllable"]},
        "lang":{"type":"string","enum":["he","en"]},
        "text":{"type":"string"},
        "paired_text":{"type":"string"},
        "options":{"type":"array","items":{"type":"string"}},
        "category":{"type":"string"},
        "order_index":{"type":"integer"},
        "math":{"type":"object","additionalProperties":false,
          "required":["left","op","right","blank_position","result_as_written"],
          "properties":{
            "left":{"type":"string"},
            "op":{"type":"string","enum":["none","+","-","x",":","compare"]},
            "right":{"type":"string"},
            "blank_position":{"type":"string","enum":["none","result","left","right"]},
            "result_as_written":{"type":"string"}}},
        "child_answer_as_written":{"type":"string"},
        "is_handwritten":{"type":"boolean"},
        "confidence":{"type":"string","enum":["high","medium","low"]}
      }}},
    "pii_spans":{"type":"array","items":{"type":"object","additionalProperties":false,"required":["kind","text"],
      "properties":{"kind":{"type":"string","enum":["person_name","school_name","phone","address","other"]},"text":{"type":"string"}}}},
    "uncertain_spans":{"type":"array","items":{"type":"object","additionalProperties":false,"required":["item_idx","text"],
      "properties":{"item_idx":{"type":"integer"},"text":{"type":"string"}}}},
    "notes_for_parent":{"type":"string"}
  }
}
```

מוסכמות "ריק": `paired_text:""`, `options:[]`, `category:""`, `order_index:-1`, `math.op:"none"`, `child_answer_as_written:""`. אין unions; אין שדות אופציונליים (בתוך מגבלות 16/24).

**הנחיות לפי סוג תוכן (נוספות לפרומפט כ-"type guide", חלק מה-system):**

| content_type | מה לחלץ | shape |
|---|---|---|
| vocabulary_list | כל מילה; אם יש תרגום/תמונה מתויגת — `paired_text` (תמונה: תיאור במילה אחת באנגלית, `img:dog`) | `word` / `pair` |
| spelling_list | כל מילה כפי שהודפסה (לא מה שהילד כתב); אם הילד כתב — ב-`child_answer_as_written` | `word` |
| letters_nikud | אות/הברה/סימן; שם הסימן ב-`category` אם מופיע ("קמץ") | `letter_or_syllable` |
| math_set | `math` מלא; `text` = הביטוי כפי שכתוב | `math_expr` |
| word_problems | `text` = השאלה; `math` רק אם הביטוי כתוב בדף | `qa` |
| reading_passage | פסקאות לפי סדר (`passage_paragraph`, `order_index`) + שאלות קיימות (`qa`, `options` אם אמריקאיות) | |
| child_story | כל משפט כפי שנכתב, בסדר (`story_sentence`) — כולל שגיאות כתיב | |
| facts | כל משפט/שאלה; `category` אם הדף ממיין | `statement` / `qa` / `category_member` |

#### 3.5.5 פיוס קריאות כפולות (כתב יד) ואימות בקוד

1. יישור פריטים לפי `idx`/`order_index` + דמיון טקסט (Levenshtein מנורמל על אותיות בלבד).
2. לכל מילה: אם שתי הקריאות זהות → `confidence` עולה דרגה; שונות → המילה נכנסת ל-`disputed[]` עם שתי החלופות (מוצגות להורה כבחירה בהקשה).
3. V17 (לולאות) / V18 (PII שלא הוחלף — התאמה מול רשימת שמות לא-משפחה) / V02 (חישוב מחדש של כל `math_expr`) / ולידציית ספרות.
4. סינון: `confidence:"low"` לא נכנס למשחק בלי אישור הורה מפורש.
5. ניקוד (Nakdan) לכל מילה עברית שנכנסת למשחק, במקביל לבניית מסך האישור.

#### 3.5.6 מסך אישור הורה ("10 שניות")

- **פריסה (iPad: זה לצד זה; iPhone: למעלה/למטה):** התמונה (עם זום) ⟷ רשימת הפריטים כצ'יפים. הקשה על פריט מדגישה את מיקומו בתמונה (`order_index`; אין bbox מדויק — הדגשה של הרצועה המשוערת).
- צ'יפים מסומנים: 🟡 `disputed`/`[?]` (חובה לבחור/לתקן), 🔵 נוקד מחדש (הקשה → השוואת ניקוד Nakdan/Phonikud), ⚪ תקין.
- **שאלות חובה בראש המסך** (בחירה בהקשה):
  1. "בשביל מי המשחק?" — K1 / K2 / K3 / כולם (ברירת מחדל מהתיוג).
  2. "מה זה?" — **שיעורי בית שצריך להגיש** · **חומר ללמוד למבחן/הכתבה** · **סתם לתרגול** · **סיפור שהילד כתב**.
  3. מתמטיקה: "התשובות שחישבנו נכונות?" (מוצגות רק להורה; מחושבות בקוד). אם בדף יש תשובות של הילד — סימון נכון/לא נכון שקט, להורה בלבד.
  3א. **סימון מתמטי מהדף (03 §1.5):** אם זוהה סימון שונה מהגדרת הילד (`·` במקום `×`, `÷` במקום `:`, סימני פריטה/נשיאה בשורה) — שורת אישור: "בכיתה כותבים · במקום ×. לעדכן?" — `notation.*` של הילד משתנה **רק** ב-✓; לעולם לא מחליפים תצוגה בשקט.
  4. כתיב מסיפור/מחברת של הילד: "אילו מילים לתרגל?" (ההורה מסמן; ברירת מחדל: אף אחת — §3.10).
- כפתור ראשי: **"צור משחק"** (וכיתוב קטן: "המשחק יהיה מוכן תוך פחות מדקה").

### 3.6 Template Fill — סכמת אחסון

```ts
export const TemplateFill = z.object({
  fillId: z.string(),
  templateId: z.string(),                // from catalog §3.8
  childId: z.enum(["K1", "K2", "K3"]),
  sourceJobId: z.string(),
  mode: z.enum(["homework_practice", "study_list", "free_practice", "review", "story"]),
  rounds: z.array(z.object({
    itemId: z.string(),                  // Item (storage) created from the source item or its clone
    presentation: z.enum(["audio_first", "text_pointed", "text_partial", "text_unpointed", "image"]),
    optionsCount: z.number().int().min(2).max(6),
  })).min(1),
  settings: z.object({
    timer: z.literal(false).or(z.object({ optIn: z.literal(true), seconds: z.number().int() })), // D10: timers only K1 opt-in after mastery
    maxConsecutiveMisses: z.number().int().default(3),            // D10
    hintPolicy: z.enum(["after_2_errors", "on_request"]),
  }),
  audioManifestKey: z.string(),
  createdAt: z.string(),
});
```

### 3.7 מסיחים מקטלוגי טעויות (בקוד)

```ts
export interface ErrorRule { code: string; applies(item: Item, ctx: ChildCtx): boolean; make(item: Item, rng: Rng): string[] }
// Pipeline: candidates = ⋃ rules.make() → filter → rank → take k
// Filters: ≠ answer (normalized: strip nikud, finals→base) · unique · same "shape" (digit count / word length ±1) ·
//          spelling distractors must NOT be a real word in the lexicon (avoids teaching ambiguity) ·
//          vocabulary distractors MUST be real words from the same semantic category and known lexicon ·
//          never use a child's own misspelling from the photo as a distractor for that child (privacy + no shaming)
// Ranking: prefer codes the child actually produced recently (engine error log), then catalog priority, then random.
```

**קטלוגי האמת הם 03 §4.2 (מתמטיקה, `E.*`) ו-04 §2.3 (עברית, `E.*`).** הקודים `MATH_*`/`HE_*`/`NIK_*` כאן הם **כינויים** שנשמרים ב-`packages/content/errors/aliases.json` (טבלאות המיפוי: 03 §4.2א, 04 §2.3ט); `classifyError()` מקבל את שני השמות ומחזיר תמיד את ה-`E.*` הקנוני (03 `MATH-AC-22`, 04 #24). תיקונים שנובעים מהמיפוי: `MATH_PLACE_VALUE` **מפוצל** ל-`E.PV_CONCAT` (25+7→95, שרשור) ו-`E.NO_CARRY` (47+38→75); `MATH_CARRY_OMIT` = `E.NO_CARRY`; `MATH_BORROW_OMIT` = `E.SUB_SMALL_FROM_LARGE` ("קטן מגדול"), והווריאנט "פרט ולא הוריד" הוא `E.BORROW_NO_DEC`; `MATH_HALF_DOUBLE` = `E.HALF_DOUBLE` (נוסף ל-03); `HE_SOUNDALIKE_K` **מפוצל** ל-כ/ק (`E.SPL.K_Q`) ו-כ/ח (`E.SPL.K_CH`) כי התיקון שונה; `HE_VISUAL` הוא תמיד **זוג-ספציפי** `E.VIS.<PAIR>` (BK, DR, HCT, VZN, SM…) ומוצג כמסיח רק כששתי האותיות בזוג ≥ 70% נלמדו (04 P12). קוד חדש נולד רק ב-03/04; 12 מוסיף כינוי.

| תחום | קודים (כינויי 12 → `E.*` קנוני; פירוט ב-§12.2) |
|---|---|
| חשבון | `MATH_OP_SWAP`→`E.OP_SWAP`, `MATH_OFF_BY_ONE`→`E.NEAR`, `MATH_DIGIT_REVERSAL`→`E.REVERSE_DIGITS`, `MATH_PLACE_VALUE`→`E.PV_CONCAT` **או** `E.NO_CARRY` (לפי הערך), `MATH_CARRY_OMIT`→`E.NO_CARRY`, `MATH_BORROW_OMIT`→`E.SUB_SMALL_FROM_LARGE` (+`E.BORROW_NO_DEC`), `MATH_TABLE_NEIGHBOR`→`E.TABLE_NEIGHBOR`, `MATH_ADD_FOR_MULT`→`E.ADD_FOR_MULT`, `MATH_COUNT_ALL_AGAIN`→`E.COUNT_ALL`, `MATH_HALF_DOUBLE`→`E.HALF_DOUBLE` (שמות היעד המדויקים לפי 03 §4.2א — ⟂VERIFY ב-P0-E1-S6 מול `errors.math.json`) |
| כתיב עברי | `HE_SOUNDALIKE_T`→`E.SPL.T_TET`, `HE_SOUNDALIKE_K`→`E.SPL.K_Q` / `E.SPL.K_CH`, `HE_SOUNDALIKE_S`→`E.SPL.S_SIN`, `HE_SOUNDALIKE_V`→`E.SPL.B_V`, `HE_GUTTURAL`→`E.SPL.ALEF_AYIN`, `HE_FINAL_FORM`→`E.FIN.*`, `HE_MATER_OMIT`/`HE_MATER_EXTRA`→`E.SPL.KM_*`, `HE_VISUAL`→`E.VIS.<PAIR>` (שמות היעד לפי 04 §2.3ט — ⟂VERIFY ב-P0-E1-S6 מול `errors.he.json`) |
| אנגלית | `EN_BD_REVERSAL`, `EN_SHORT_VOWEL_SWAP` (bat/bet), `EN_SILENT_E`, `EN_DIGRAPH_SPLIT` (sh→s), `EN_HE_TRANSFER` (ת׳ /th/ → s/z) |
| אוצר מילים | `VOC_SAME_CATEGORY`, `VOC_SAME_FIRST_SOUND`, `VOC_PICTURE_SIMILAR` |
| ניקוד (K2) | `NIK_KAMATZ_PATACH_OK` (לא מסיח! שניהם /a/ — אסור להשתמש בהם כמסיח זה לזה), `NIK_A_I` (בָּ/בִּ), `NIK_E_A` |

הערה פדגוגית: במשחקי צליל (K2) אסור להציג כמסיחים צירופים שנשמעים זהה בעברית מודרנית (קמץ/פתח, צירה/סגול, ט/ת, כ/ק כשאלה צליל). כיתוב-כתיב (K1) הוא המקום שבו ט/ת **כן** מסיח.

### 3.8 קטלוג תבניות משחק (≥ 15)

עקרון C1: הפעולה הלימודית היא פעולת המשחק. לכל תבנית: צורות תוכן נתמכות, **מינימום תוכן**, גילים, מודאליות.

| # | `templateId` | שם בעברית | פעולת משחק (C1) | צורות | מינימום תוכן | K3 | K2 | K1 |
|---|---|---|---|---|---|---|---|---|
| 1 | `tap_heard_picture` | שמע וגע | לגעת בתמונה שהקמע קורא לה → היא קופצת למחנה | pair (word↔img) | 4 זוגות עם תמונה מהספרייה | ✓ (2–3 אפשרויות) | ✓ (4) | — |
| 2 | `memory_pairs` | זוגות זיכרון | הופכים קלפים; זוג = בונים אבן במחנה | pair | 3 זוגות (K3), 6 (K2), 8 (K1) | ✓ | ✓ | ✓ |
| 3 | `count_and_tap` | סופרים ונוגעים | סופרים עצים למדורה בנגיעה; כל נגיעה = מספר בקול (ספירה נקבית, B4) | math (count ≤5/10/20) | מספרים 1–5 (K3), עד 20 (K2) | ✓ | ✓ | — |
| 4 | `first_sound` | הצליל הראשון | שולחים את הצללון לשער של הצליל ששמענו | word (he) | 4 מילים מפוענחות שאותן ראשונות ⊆ נלמדו | — | ✓ | — |
| 5 | `balloon_battle` | קרב בלונים | מפוצצים את הבלון עם התשובה כדי לשחרר קריסטל | qa / math_expr / word | 5 פריטים × 3 מסיחים | — | ✓ (שמע) | ✓ |
| 6 | `shadow_whack` | צללונים קופצים | נוגעים רק בצללונים שנושאים פריט מהקטגוריה | category | ≥5 נכונים + ≥5 לא נכונים | — | ✓ | ✓ |
| 7 | `sort_to_tents` | מיון לאוהלים | גוררים פריטים לאוהל הנכון | category | 2–3 קטגוריות × ≥3 | — | ✓ (2 קטגוריות) | ✓ |
| 8 | `letter_unjumble` | סדר את האותיות | מסדרים אותיות-אבנים לגשר | word/spell | 4 מילים, 2–3 אותיות (K2) / 3–7 (K1) | — | ✓ | ✓ |
| 9 | `spell_builder` | שמע ובנה | שומעים מילה ובונים אותה מאותיות (מקלדת אותיות מוגבלת) | spell + audio | 5 מילים + TTS לכל מילה | — | — | ✓ |
| 10 | `cloze_bridge` | השלם את המשפט | משלימים מילה חסרה כדי שהגשר ייבנה | cloze | 4 משפטים, מסיחים מאותה קטגוריה דקדוקית | — | ✓ (שמע + תמונה) | ✓ |
| 11 | `story_sequence` | סדר את הסיפור | מסדרים תמונות/משפטים על מסלול המפה | sequence | 3–4 (K2), 4–6 (K1) | — | ✓ | ✓ |
| 12 | `true_false_shield` | מגן אמת | מרימים מגן "נכון" או "לא נכון" מול הצללון | statement | ≥6, מאוזן ±1 | — | ✓ (שמע) | ✓ |
| 13 | `tower_defense_math` | הגנת המגדל | כל תרגיל נכון יורה קריסטל-אור שמחזיר צללון לחבר | math_expr | 8 תרגילים (משוכפלים מאותו כישור) | — | ✓ (עד 10) | ✓ |
| 14 | `number_line_jump` | קפיצות על ציר | הדמות קופצת על ציר המספרים לתוצאה | math_expr (+/−) | 6 תרגילים | — | ✓ | ✓ |
| 15 | `rank_order` | מהקטן לגדול | מסדרים לבנים לפי ערך/אורך/סדר אירועים | sequence / numbers | 4–6 ערכים | — | ✓ (3–4) | ✓ |
| 16 | `mini_crossword` | תשבץ קטן | ממלאים תשבץ (RTL) כדי לפתוח תיבה | pair (clue↔word) | 5 מילים באורך 3–7 | — | — | ✓ |
| 17 | `word_search` | חיפוש מילים | מוצאים מילים ברשת (RTL+LTR לאנגלית) | word | 6 מילים, רשת 8×8 | — | — | ✓ |
| 18 | `flash_duel` | קלפי קרב | כרטיסיות חזרה מהירות מול "בוס" — ללא טיימר כפוי | pair / qa | 6 | — | — | ✓ |
| 19 | `picture_story_listen` | סיפור בתמונות | מקשיבים לסיפור ונוגעים ב"מי?" / "מה?" | story_scene | סיפור 3+ עמודים | ✓ (עם הורה) | ✓ | — |
| 20 | `comic_choices` | קומיקס עם בחירות | בוחרים מה יקרה עכשיו בסיפור (עד 3 נקודות בחירה) | story_scene | 4 עמודים + 1 בחירה | — | ✓ (עם הורה) | ✓ |
| 21 | `nikud_sort` | מיון תנועות | מיון הברות לפי צליל התנועה (/a/ מול /i/) | letter_or_syllable | 2 קבוצות × ≥4 | — | ✓ | — |

כללי מודאליות: מתחת לגיל 6 — `audio_first`, ≤ 4 אפשרויות, בלי הקלדה, מטרות מגע גדולות (UX spec). K3: רק תבניות 1–3, 19, ללא כשל ("כל נגיעה מצליחה; הנכונה מקבלת חגיגה גדולה יותר").

### 3.9 בחירת תבנית אוטומטית (דטרמיניסטי)

```ts
export function selectTemplates(content: ConfirmedContent, child: ChildProfile, history: PlayHistory): TemplateChoice[] {
  const eligible = CATALOG.filter(t =>
    t.ages.includes(child.band) &&
    t.shapes.some(s => content.shapes.has(s)) &&
    t.meetsMinimum(content, child) &&                       // counts after distractor generation & clone expansion
    (child.band !== "K3" || t.noFailState) &&
    (!t.needsReading || child.canReadAt(content.nikudLevel)) &&
    (!t.needsTyping || child.band === "K1") &&
    (!t.needsAudioForEveryItem || content.allItemsHaveAudio));
  const score = (t: Template) =>
      3 * t.skillFit(content.contentType)                  // primary fit table, e.g. spelling_list → spell_builder=1.0
    + 2 * novelty(t, history, 5)                           // not played in last 5 sessions → 1
    + 1 * t.coPlayBonus(child)                             // K2/K3 picture/story templates when parent co-play mode
    - 2 * (history.lastFrustration(t) ? 1 : 0);            // D10 frustration signal
  return eligible.sort((a, b) => score(b) - score(a)).slice(0, 3)
                 .map((t, i) => ({ templateId: t.id, isDefault: i === 0 }));
}
```

- אם `meetsMinimum` נכשל רק במספר פריטים: **הרחבה** — שכפול פריטי מתמטיקה באותו כישור (AIG), או הוספת מילים ממאגר הילד באותה קטגוריה (מסומנות "מילה נוספת"), עד המינימום; אחרת התבנית נופלת.
- **כשל שכפול** (אין `ItemModel` תואם לפריט במצב `homework_practice`/`exam_page`): הפריט מדולג **לילד**, ולהורה מוצג במסך האישור: "לא הצלחנו לשכפל פריט N — יוצג רק להורה" (עם הפריט המקורי, להורה בלבד); אם < מינימום אחרי דילוגים → הצעת הקלדה של פריט דומה.
- אחד מתוך 3 מוצג כ"המשחק המומלץ" ונפתח מיד; השניים האחרים זמינים ב"עוד דרך לשחק".
- אותו מקור יכול להפיק מילוי שונה לכל ילד (§3.12): דף אוצר מילים של K1 → K1 `spell_builder`, K2 `first_sound` + `memory_pairs`, K3 `tap_heard_picture`.

**טבלת התאמה ראשית (`skillFit=1.0`):** vocabulary_list → memory_pairs/balloon_battle/mini_crossword; spelling_list → spell_builder/letter_unjumble/word_search; letters_nikud → first_sound/nikud_sort/shadow_whack; math_set → tower_defense_math/number_line_jump/balloon_battle; word_problems → balloon_battle (מסיחים מקטלוג)/flash_duel; reading_passage → story_sequence/cloze_bridge/true_false_shield; child_story → comic_choices/picture_story_listen/story_sequence; facts → true_false_shield/sort_to_tents/flash_duel.

### 3.10 מצב שיעורי בית נגד העתקה (anti-cheating)

| מצב (נבחר במסך האישור) | מה הילד מקבל | מה מוסתר | מתי משתנה |
|---|---|---|---|
| **שיעורי בית להגשה** (`homework_practice`) | **שכפולי כישור** בלבד: אותו סוג תרגיל עם מספרים/מילים אחרים (AIG: אותו כישור, אותו "קושי מבני" — עם/בלי המרה, אותו לוח כפל, אותה תבנית כתיב) | הפריטים המקוריים ותשובותיהם; מפתח התשובות מוצג להורה בלבד | ההורה לוחץ "הוגש" **או** למחרת ב-06:00 → נפתח מצב `review` עם הפריטים המקוריים |
| **חומר למבחן/הכתבה** (`study_list`) | הפריטים עצמם (תרגול המילים/העובדות הוא המטרה) | — | — |
| **תרגול חופשי** (`free_practice`) | הפריטים + שכפולים | — | — |
| **חזרה** (`review`) | הפריטים המקוריים, משולבים בתור SRS | — | +1/+3/+7… (D9) |
| **דף מבחן/מבדק** (`exam_page`; נבחר ע"י ההורה, או שהמודל מסמן `looks_like_exam=true` — כותרת "מבחן"/"מבדק"/"הערכה", שדות שם/כיתה/תאריך/ציון) | **רק שכפולי כישור**, לעולם לא הפריטים עצמם. אין מצב `review` לדף כזה | הכול. מפתח התשובות **לא מחושב ולא מוצג לאיש** (גם לא להורה) — הדף נמחק מיד אחרי בניית השכפולים | — (אין פתיחה מאוחרת) |

- **"לא לפתור שיעורי בית":** בכל מצב שבו הדף הוא הגשה או מבחן, המערכת לא מציגה לילד תשובה לאף פריט מקורי (V20). P3 שנקרא מתוך משחק כזה רואה רק שכפול. אם ההורה מנסה לשנות מצב מ-`exam_page` ל-`study_list` — אישור נוסף ב-Face ID עם הסבר קצר ("זה דף מבחן. בטוח?").

כללי שכפול (`cloneSkill`):
- מתמטיקה: מזהים את תבנית הכישור מהביטוי (`a+b` עם המרה? טווח? לוח?) → `ItemModel` תואם → 2 שכפולים לכל פריט מקורי, שאף אחד מהם לא זהה (גם לא בהחלפת סדר) לפריט מקורי.
- כתיב (הגשה): מילים אחרות עם אותה תופעה (אותה אות בעייתית/אם קריאה/סופית) מתוך lexicon.
- הבנת הנקרא (הגשה): לא מייצרים תשובות לשאלות הדף. במקום: משחק `story_sequence` + `true_false_shield` עם היגדים **אחרים** על הטקסט, שנוצרים ע"י Haiku ונבדקים מול הטקסט (`V12`: אף היגד לא מנסח מחדש שאלה מהדף).
- העוזר הסוקרטי (P3), אם נקרא מתוך משחק הגשה, מקבל רק את פריט השכפול.

### 3.11 סיפור של הילד → סיפור אינטראקטיבי (באישור)

1. **הסכמת הילד** (על המכשיר שלו, בקול ובתמונה): "אבא/אמא צילמו את הסיפור שכתבת. רוצה שנהפוך אותו לסיפור במשחק?" — "כן" / "לא עכשיו". "לא" נכבד ואינו נשאל שוב לאותו סיפור.
2. **אישור הורה** (מסך האישור, מצב "סיפור שהילד כתב").
3. **חילוץ** משפטים כפי שנכתבו (`story_sentence`), כולל שגיאות.
4. **פיצול לסצנות** (`photo.story`, Sonnet 5.5): המודל מקבץ משפטים לעמודים, מציע תגיות סצנה מ-SCENE_TAGS, ול-K1 מציע 1–2 נקודות בחירה ("מה יקרה אם…") עם המשך קצר **שמסומן כ"הצעה של הרובוט"** ושהילד יכול לדחות ("לא, הרעיון שלי!") — לפי ממצאי CHI 2026 על שמירת כוונת הילד.
5. **הטקסט של הילד נשמר מילה במילה** ב-`authored_text`. גרסת תצוגה מנוקדת נוצרת מתוך תיקון כתיב **שקט** שמוצג רק להורה כהצעה; ההורה בוחר: "להציג כמו שכתבה" (ברירת מחדל ל-K1) או "להציג מתוקן" (ברירת מחדל ל-K2, שעדיין לא כותב לבד בדרך כלל — הסיפור הוכתב). אין סימון שגיאות מול הילד, אין "תיקון פומבי".
6. **איורים:** הרכבה מספריית הרקעים/דמויות/מדבקות (P6), ללא יצירת תמונה בזמן אמת. הדמויות מהסיפור ממופות לדמויות קיימות או לאווטאר המשפחה (Rive). "איור מיוחד" (יצירה ב-GPT Image מתיאור טקסטואלי בלבד, בלי תמונות פנים) זמין כבקשת הורה אופליין, עם שער אישור — לא ב-60 השניות.
7. **קרדיט:** עמוד שער "סיפור מאת {{name}}" (שם מוזרק בלקוח; הקול — הקלטת ההורה של שם הילד, אם קיימת).
8. **כתיב:** תרגול רק ממילים שההורה סימן (המודל לא מבחין בין טעות OCR לטעות ילד).
9. **שיתוף בין אחים:** כבוי כברירת מחדל; הילד-הכותב מחליט ("לשתף עם נועה?") — C6, בלי השוואות.

**Output schema (`photo.story`):** `{pages:[{sentence_idxs:[int], scene:{bg, characters[], props[]}}], choices:[{after_page:int, prompt{m,f,pl,n}, options:[{text, is_robot_suggestion:bool}]}], title_suggestion, characters_found:[{name_token, maps_to_char_id}]}` — כל השדות required.

### 3.12 העשרה לכל ילד (`photo.enrich`, Haiku 5.5)

**מה נכנס:** הפריטים המאושרים (אחרי תיקון הורה), הילדים שנבחרו (גיל, מגדר, רמת ניקוד, אותיות שנלמדו — כנתונים, בלי שמות), התבניות שנבחרו.
**מה יוצא:** לכל ילד — הנחיית פתיחה (GText), עד 3 רמזים לכל משפחת פריט, היגדי נכון/לא נכון לטקסטים, מסיחים **לשוניים** (אוצר מילים/השלמה) — **לא** מתמטיים ולא כתיב (אלה מהקוד).

**System (עברית, cached):**

```text
<SAFETY CORE v1>
את/ה מכין/ה חומר משלים למשחק שנבנה מדף של בית הספר/הגן. הפריטים כבר אושרו בידי הורה ואסור לשנות אותם.
כתוב/כתבי בעברית ישראלית פשוטה, בלי ניקוד, ושלוש גרסאות מגדר (m/f/pl) + n כשאפשר.
רמזים: מכוונים לחשיבה, לא נותנים את התשובה, לא מזכירים את האות/המספר/המילה הנכונים.
מסיחים לשוניים: מילים אמיתיות מאותה קטגוריה, באותה רמת קושי, שאינן נכונות בהקשר. אל תשתמש/י בשמות אנשים.
היגדי נכון/לא נכון: מבוססים רק על מה שכתוב בטקסט; היגד "לא נכון" משנה פרט אחד בלבד. אל תנסח/י מחדש שאלות שמופיעות בדף.
גיל ורמה לכל ילד מופיעים בהודעה. אם פריט לא מתאים לגיל — סמן/סמני אותו ב-skip_for.
```

**Output schema:** `{per_child:[{child:"K1"|"K2"|"K3", intro{m,f,pl,n}, hints:[{family_key, ladder:[{m,f,pl,n}]}], tf_statements:[{text, is_true, based_on_item_idx}], lexical_distractors:[{item_idx, options:[string]}], skip_for:[int]}]}` — required בכל רמה; `child` enum.

**נפילה:** אם Haiku נכשל/מסרב/חורג מזמן (8 ש׳) — המשחק נבנה בלי העשרה: הנחיה מתוסרטת כללית לתבנית, רמזים כלליים מתוסרטים, ותבניות שלא דורשות מסיחים לשוניים.

### 3.13 קול ובניית חבילה ל-P2

- שורות חדשות (מילים, משפטים, הנחיות) מרונדרות **בשרת** אחרי האישור: Azure he-IL (או מנצח ה-bake-off אם ה-latency שלו ≤ 1.5 ש׳ לשורה), מקביליות 8, SSML לפי §6.5; מילים שכבר קיימות ב-cache (hash זהה, §6.8) לא מרונדרות.
- עוצמה: בזמן אמת מוחל gain קבוע לכל קול (נמדד ב-bake-off); נרמול LUFS מדויק בלילה ע"י ה-runner, והקובץ מוחלף ב-R2 (גרסת manifest עולה, המכשיר מוריד שקט).
- אם ה-TTS לא זמין: K1 משחקת בתבניות טקסט בלבד (בתצוגה לפי רמת ניקוד); K2/K3 ממתינים (הודעה להורה: "הקול יהיה מוכן בעוד כמה דקות"). **קובץ VO חסר בזמן משחק** (כל חבילה, לא רק P2): הנגן נופל ל-`vo.generic.<intent>` מחבילת הליבה המצורפת ל-build (11 §4.11) — ולידטור `V22` (§1.6) מבטיח שלכל `intent` יש שורה גנרית אחת לפחות ב-3 גרסאות מגדר.

### 3.14 יעד "זמן עד משחק" (< 60 ש׳ אחרי "צור משחק")

| שלב (אחרי אישור) | P50 | P95 | הערות |
|---|---|---|---|
| ניקוד (התחיל כבר לפני האישור, במקביל למסך) | 0 | 3 ש׳ | Nakdan; רק מילים שתוקנו ע"י ההורה מנוקדות מחדש |
| בחירת תבנית + מסיחים + שכפול (קוד) | 0.2 ש׳ | 0.5 ש׳ | |
| העשרה Haiku 5.5 | 4 ש׳ | 8 ש׳ (timeout) | אפשר לדלג (§3.12) |
| ולידטורים | 0.2 ש׳ | 0.5 ש׳ | |
| TTS (עד ~40 שורות × מגדר רלוונטי בלבד) | 6 ש׳ | 15 ש׳ | מקבילי; רק הגרסאות של הילדים שנבחרו |
| בניית חבילה + R2 + push | 1 ש׳ | 3 ש׳ | |
| הורדה למכשיר הילד | 2 ש׳ | 6 ש׳ | אודיו AAC מונו ~10–20KB לשורה |
| **סה"כ** | **~14 ש׳** | **~36 ש׳** | יעד קשיח P95 ≤ 60 ש׳ |

ולפני האישור (לא נספר ביעד, אבל מדווח): העלאה 1–3 ש׳; חילוץ Sonnet 10–25 ש׳ לדף (כתב יד במקביל עם Opus: המקסימום של שניהם); מסך האישור מוצג בהדרגה (streaming) — פריטים מופיעים כשהם מגיעים. **הסיבוב הראשון** ניתן להפעלה ברגע שיש 4 פריטים מוכנים עם אודיו (המשחק ממשיך להיטען ברקע).

### 3.15 כשלים, מקרי קצה ונפילות (כלל אחד לכל מצב)

| # | מצב | זיהוי | התנהגות | מה ההורה רואה |
|---|---|---|---|---|
| 1 | `not_learning_material` | המודל | התמונה נמחקת מיד; אין פריטים | "לא מצאנו כאן תרגיל. אפשר לצלם שוב?" + טיפים (§3.2) |
| 2 | **תמונה מטושטשת / חשוכה** | על המכשיר לפני העלאה: Laplacian variance < 60 (על גרסת 512px) **או** בהירות ממוצעת < 40/255 → "חשוד"; בשרת: ≥ 30% פריטים `confidence:"low"` | על המכשיר: מציעים לצלם שוב **לפני** העלאה (לא חוסם — "להמשיך בכל זאת"); בשרת: ממשיכים רק עם הפריטים הוודאיים, מינימום 4 | "התמונה לא חדה. לצלם שוב באור?" (עם דוגמה) |
| 3 | **דף מעורב עברית/אנגלית** (למשל מילון כיתתי) | `primary_language:"mixed"` + `lang` לכל פריט | פריטי `he` ו-`en` נפרדים ל**שני** משחקים: עברי (קריין) ואנגלי (אי אנגלי, Sam; LTR); זוגות he↔en (`pair`) → `memory_pairs` באי האנגלי עם גלוס לפי `glossLevel` של הילד (05 §10.3). אין שורה מעורבת ב-TTS (05 AC4) | שני כרטיסי "צור משחק", אפשר לבטל אחד |
| 4 | **כתב יד לא קריא** | ≥ 30% `[?]` **או** `disputed` ב-≥ 50% מהמילים אחרי פיוס Sonnet/Opus | לא מנחשים. מסך האישור מציג רק את המילים הוודאיות; לכל `[?]` שדה הקלדה (ההורה מקליד ≤ 10 מילים בדקה) | "קראנו 6 מתוך 14 מילים. להשלים או לצלם שוב?" |
| 5 | **שם של ילד אחר / מורה על הדף** | `pii_spans` + V18 מול `private/family-names.json` (רשימת "לא-משפחה" שההורה הזין) + היוריסטיקה: טוקן בראש הדף בשדה "שם:" | הטוקן מוחלף ב-`[NAME]` **לפני** אחסון; אם הפריט עצמו הוא השם (רשימת שמות הכיתה) → הפריט נזרק; אם הצילום כולל שם ילד שלנו — לא מוחלף (שם המשפחה ידוע רק בלקוח — ה-token `[NAME]` ממופה ל-`{{name}}` בלקוח רק אם ההורה בחר) | צ'יפ אפור "[שם הוסר]"; אין הצגה של השם המקורי |
| 6 | **דף מבחן/מבדק** | `looks_like_exam` / תיוג הורה | מצב `exam_page` (§3.10): רק שכפולים, אין מפתח תשובות לאיש, הדף נמחק מיד | "זה נראה כמו דף מבחן — נכין תרגול דומה בלי התשובות" |
| 7 | **דף עם תמונות בלבד** (ללא טקסט) | 0 פריטי טקסט, `content_type:"vocabulary_list"` עם `img:*` או `not_learning_material` | אם המודל זיהה ≥ 4 אובייקטים ברורים (`paired_text:"img:dog"`) → `tap_heard_picture`/`memory_pairs` עם **תמונות מהספרייה שלנו** (P6), לא מהצילום; אחרת #1 | "מצאנו 6 תמונות: כלב, חתול… להכין משחק שמע-וגע?" |
| 8 | **10 דפים בבת אחת** | > 4 דפים ב-job | הלקוח מפצל אוטומטית ל-jobs של ≤ 4 דפים (לפי סדר צילום); כל job עובר בנפרד; עלות משוערת מוצגת לפני השליחה (10 דפים ≈ $0.5–1.1); אם זה חוצה 80% מהתקרה — §1.9 | "10 דפים → 3 משחקים. להמשיך?" (עם הערכת עלות ודקות) |
| 9 | **אותו דף פעמיים** | perceptual hash (pHash, 64-bit, מרחק ≤ 6) מול jobs מ-30 הימים האחרונים | לא מחלצים שוב; מציעים לפתוח את המשחק הקיים או לבנות מילוי חדש (תבנית אחרת) | "כבר צילמת את הדף הזה ב-{{date}}" |
| 10 | **דף הפוך/מסובב** | EXIF orientation + בדיקת כיוון טקסט ראשונית (Vision `VNRecognizeTextRequest` על המכשיר, אם זמין לעברית) | סיבוב אוטומטי לפני העלאה; בשרת: אם < 3 פריטים ו-`uncertain` גבוה → ניסיון אחד נוסף ב-180° (עלות ×2 מותרת) | — |
| 11 | **הורה אופליין בזמן הצילום** | `expo-network` | **תור לכידה** (§3.16): נשמר מקומית, נשלח אוטומטית | "נשמר. יישלח כשיש אינטרנט" + פס מצב |
| 12 | **ספק (Anthropic) לא זמין / timeout 40 ש׳ לדף** | 5xx, 529, timeout | retry אחד אחרי 10 ש׳; אז "ננסה שוב בעוד כמה דקות" — ה-job נשאר `uploaded` ומנסה שוב כל 5 דק׳ עד שעה (ברקע, בלי שההורה יחכה); אחרי שעה → הקלדה ידנית. **Refusal:** לא מנסים שוב ולא מודל אחר | "הדף ממתין. נודיע כשמוכן" / "לא הצלחנו לקרוא את הדף הזה" |
| 13 | **TTS לא זמין** | §3.13 | K1 טקסט בלבד; K2/K3 ממתינים; retry כל 5 דק׳ | "הקול יהיה מוכן בעוד כמה דקות" |
| 14 | **תקרת $** | §1.9 | §1.9 | §1.9 |
| 15 | < מינימום של כל תבנית | `meetsMinimum` | הרחבה (§3.9) או הצעה להוסיף מילים | "נמצאו רק 2 מילים — להוסיף עוד מהמילים של {{ילד}}?" |
| 16 | **ההורה נטש את מסך האישור** | אין `confirm` תוך 24 שעות | ה-job נמחק (כולל התמונה ב-R2); תזכורת אחת ב-PA-INBOX אחרי 2 שעות | "דף ממתין לאישור (נמחק מחר)" |
| 17 | **ילד משחק במכשיר בזמן שהמשחק נבנה** | — | המשחק מופיע בסיום הפרק הנוכחי, לא באמצע (01: "לא מוציאים במציאה"); בלי התראה לילד | — |
| 18 | **דף עם תשובות של הילד** | `child_answer_as_written` | תשובות הילד לעולם לא מסיחים ולא "שגיאה" לילד; השוואה שקטה להורה בלבד (§1.6.1) | במסך האישור: "7 מתוך 9 נכונות (רק לך)" |
| 19 | **פנים בצילום** (ילד מציץ למסגרת) | `VNDetectFaceRectanglesRequest` על המכשיר | טשטוש לפני העלאה (§3.3); אם > 25% משטח הדף מטושטש → הצעה לצלם שוב | "יש פנים בתמונה — טשטשנו. לצלם שוב בלי?" |
| 20 | **דפים מכמה מקצועות ב-job אחד** | `content_type` שונה בין דפים | פיצול למשחק לכל `content_type` (עד 3 משחקים ל-job) | כרטיס לכל משחק |
| 21 | **אייפון קטן (iPhone SE) במסך האישור** | viewport | פריסה למעלה/למטה; צ'יפים ≥ 44pt; זום תמונה בהקשה | — |

### 3.16 תור לכידה אופליין (Capture Queue)

- אחסון: `expo-sqlite` טבלת `capture_queue` + קובץ JPEG מוצפן (`expo-file-system` + מפתח ב-SecureStore), **אחרי** עיבוד הפרטיות של §3.3 (טשטוש/EXIF/resize רצים גם אופליין).
- מגבלות: עד 20 דפים ועד 60MB; מעבר לזה — "התור מלא, נשלח קודם את הקודמים".
- שליחה: אוטומטית ב-`expo-network` → online, או בפתיחת האפליקציה; סדר FIFO; כל job עם `clientJobId` (ulid) לאידמפוטנטיות — שליחה כפולה אחרי קריסה לא יוצרת שני jobs.
- פקיעה: דף שלא נשלח תוך 7 ימים נמחק (ההורה רואה ספירה לאחור). ביטול צימוד/מחיקת נתוני משפחה מנקה את התור.
- בדיקת קבלה: צילום 3 דפים במצב טיסה → חיבור → 3 jobs בשרת תוך ≤ 60 ש׳, בלי כפילויות.

### 3.17 בדיקות קצה ל-P2 (סט קבוע ב-`tools/eval/p2-edge/`)

30 הדפים האמיתיים של המשפחה (§10) + 20 דפים סינתטיים: מטושטש, חשוך, מסובב 90°/180°, מעורב he/en, מבחן עם שדה "שם", רשימת שמות כיתה, דף תמונות בלבד, דף ריק, צילום מסך של טלפון, 2 דפים באותה תמונה, כתב יד של בן 5, כתב יד של בת 8, דף עם תשובות הילד, דף עם פנים, דף A3, דף בכתיב חסר ובכתיב מלא, דף אנגלי בלבד, דף עם ניקוד מלא, דף מתמטיקה במאונך, דף הבנת הנקרא עם 5 שאלות. לכל דף: התנהגות צפויה מהטבלה ב-§3.15, ובדיקת E2E שמוודאת אותה (ללא קריאת מודל חיה — תגובות המודל מוקלטות ב-fixture; ריצה חיה פעם בשבוע).

### 3.16 עלות לתמונה (הערכה)

| רכיב | טוקנים | מחיר |
|---|---|---|
| Sonnet 5.5 חילוץ: תמונה ~4.8K + system 2.5K (cache read) + פלט ~2K | | ≈ $0.010 + $0.0003 + $0.02 = **$0.03** |
| + Opus 5.5 (כתב יד בלבד) | אותו דבר | ≈ **$0.06** |
| Haiku 5.5 העשרה: 3K קלט + 3K פלט | | ≈ **$0.002** |
| Azure TTS ~1,500 תווים | | ≈ **$0.02** (או 0 בתוך free tier) |
| **סה"כ לתמונה** | | **$0.05 (מודפס) – $0.11 (כתב יד)** → 30 תמונות/חודש ≈ $2–3 |

---

## 4. P3 — עוזר סוקרטי וארגז חול לאוריינות AI

### 4.1 שתי ישויות, לא אחת (C7, gap_audit D13)

| | הקמע וצוות הדמויות | רובוט-העזר **"גַּלְגַּלִּי"** (שם זמני) |
|---|---|---|
| מה זה | דמויות סיפור מתוסרטות, חמות | מכונה מסומנת: גלגלי שיניים גלויים, נורית "חושב", צליל מתכתי |
| מקור הטקסט | P1 בלבד (מאושר מראש) | עצי דיאלוג מתוסרטים + (K1 בלבד) תורות LLM מוגבלות |
| קול | TTS/הקלטות מוכנים מראש | שורות מתוסרטות: TTS מוכן מראש + פילטר "מכונה" (§6.3). תורות LLM: **טקסט בלבד** + צפצוף קבוע (אין TTS חי, E9) |
| אמירות אסורות | — | רגשות, חברות, "התגעגעתי", "אני אוהב אותך", זיכרון אישי, "אני חי" |
| פתיחה קבועה | — | "אני גלגלי. אני מכונה — אני לא מרגיש ולא חושב כמו בן אדם. אני מנחש לפי דפוסים, ולפעמים אני טועה." |

### 4.2 שערי גיל

| ילד | עוזר סוקרטי | LLM חי | קלט ילד | ארגז חול AI |
|---|---|---|---|---|
| K3 | אין (הקמע נותן "נסה שוב" מתוסרט בלבד) | ✗ | נגיעה | A0, C0 (ראו ai_literacy) — מתוסרט, עם הורה |
| K2 | עצים מתוסרטים בלבד: סולם רמזים מוכן מראש, שאלות בחירה בתמונות | ✗ | נגיעה/צ'יפים | A1, B1, C1, C2, D1, E1 (עם הורה), F1 — הכל מוכן מראש |
| K1 | עצים מתוסרטים + צמתי `llm_turn` מוגבלים (Haiku 5.5) | ✓ (מוגבל) | צ'יפים + טקסט קצר (≤ 60 תווים) | A2, B2, C3, C4, D2, D3, E2, F2, F3 — מוכן מראש; LLM רק בצמתים מוגדרים |
| הורה + K1 | "טייס-משנה מבוגר": ההורה משתמש בחשבון שלו לפי תסריט שהמשחק נותן | — (מחוץ לאפליקציה) | — | — |

מגבלות שימוש ל-K1: עד 12 תורות LLM ליום, עד 4 לתרגיל; אחרי זה הצמתים נופלים לשורות מתוסרטות. "קודם אני": צומת `llm_turn` נפתח רק אחרי ניסיון אחד לפחות של הילד ("אסימון מוח").

### 4.3 עץ דיאלוג — פורמט

```ts
type Node =
  | { id: string; kind: "say"; text: GText; audio: string; next: string }
  | { id: string; kind: "ask_choice"; text: GText; audio: string; chips: { label: GText; icon: string; next: string }[] }
  | { id: string; kind: "ask_answer"; text: GText; expects: "number" | "word" | "option";
      onCorrect: string; onError: Record<ErrorCode | "other", string> }       // error classified IN CODE (catalog §3.7)
  | { id: string; kind: "hint"; level: 1 | 2 | 3; text: GText; audio: string; next: string }
  | { id: string; kind: "llm_turn"; goal: StepGoal; allowedMoves: Move[]; maxTurns: 1 | 2;
      fallback: string /* scripted node id */; next: string }               // K1 only
  | { id: string; kind: "handoff"; text: GText; to: "parent" | "try_again" | "next_item" }
  | { id: string; kind: "end"; praise: "process" };

type Move = "ask_guiding_question" | "give_hint" | "ask_to_explain" | "affirm_process" | "redirect_to_task" | "suggest_grownup";
interface StepGoal { skillId: string; subGoal: string /* e.g. "notice that there are 4 equal groups" */; childAttemptSummary: string /* code-made */ }
```

**דוגמה (K1, בעיית כפל מילולית):**

```yaml
tree: socratic.math.mult-story.v1
start: s1
nodes:
  s1: { kind: ask_answer, text: {n: "כמה קריסטלים יש בסך הכול?"}, expects: number,
        onCorrect: done, onError: { MATH_ADD_FOR_MULT: h_groups, MATH_TABLE_NEIGHBOR: h_check, other: h1 } }
  h_groups: { kind: hint, level: 1, text: {n: "בוא/י נסתכל על הסלים. בכל סל יש אותו מספר קריסטלים?"} , next: q_groups }   # stored as m/f/pl in real data
  q_groups: { kind: ask_choice, text: {n: "איך נספור הכי מהר?"}, chips: [
      { label: {n: "אחד אחד"}, next: l1 }, { label: {n: "בקבוצות"}, next: s1 }, { label: {n: "לא יודע/ת"}, next: h2 } ] }
  l1: { kind: llm_turn, goal: { subGoal: "help the child see equal groups → repeated addition → multiplication" },
        allowedMoves: [ask_guiding_question, give_hint, ask_to_explain], maxTurns: 2, fallback: h2, next: s1 }
  h_check: { kind: hint, level: 2, text: {n: "קרוב! נבדוק: כמה סלים יש? נספור בקפיצות."}, next: s1 }
  h2: { kind: hint, level: 3, text: {n: "דוגמה אחרת: 3 סלים, ובכל סל 2. 2, 4, 6 — שש."}, next: s1 }   # parallel example, never this item
  done: { kind: end, praise: process }
```

### 4.4 תור LLM מוגבל (`socratic.turn`)

**System (עברית, cached 1h):**

```text
<SAFETY CORE v1>
את/ה "גלגלי", רובוט-עזר במשחק לימודי לילדה/ילד בכיתה ג'. את/ה מכונה ואומר/ת זאת אם שואלים.
התפקיד: לעזור לילד לחשוב בעצמו על תרגיל אחד. לעולם לא לתת את התשובה, לא לרמוז על המספר/המילה הנכונים,
ולא לפתור צעדים בשבילו. מותר: לשאול שאלה מכוונת אחת, לתת רמז על דרך חשיבה, לבקש הסבר, לשבח תהליך.
מגבלות:
- תשובה קצרה מאוד: משפט אחד או שניים, עד 20 מילים. עברית פשוטה, בלי ניקוד, בלי אמוג'י.
- כתוב/כתבי רק במגדר שמופיע ב-ADDRESS (m או f). אל תשנה/י מגדר.
- דבר/י רק על התרגיל הנוכחי. אם הילד כותב משהו אחר — move "redirect_to_task".
- אם הילד כותב משהו שנשמע עצוב, מפחיד, כואב, על פגיעה, על סוד, או מבקש עזרה אמיתית — move "suggest_grownup"
  והטקסט: הצעה עדינה לספר להורה. אל תשאל/י שאלות על זה ואל תייעץ/י.
- אל תבקש/י פרטים אישיים, ואל תחזור/י על פרטים אישיים שהילד כתב.
- בלי שבח תכונה ובלי סופרלטיבים. בלי "כל הכבוד" ריק — שבח רק פעולה מסוימת שהילד עשה.
- אל תטען/י שיש לך רגשות, זיכרון של הילד או חברות.
בחר/י move מתוך ALLOWED_MOVES בלבד.
```

**User:**

```text
ADDRESS: f
SKILL: כפל — קבוצות שוות (כיתה ג')
PROBLEM_TEXT: "בכל סל יש {{b}} קריסטלים. יש {{a}} סלים. כמה קריסטלים יש בסך הכול?"   ← numbers are masked; the model never sees them
SUB_GOAL: help the child see equal groups → repeated addition → multiplication
CHILD_ATTEMPTS (code summary): 1st answer = a+b (error MATH_ADD_FOR_MULT). Chose chip "אחד אחד".
CHILD_TEXT (sanitized, may be empty): "אני לא מבינה למה לא לחבר"
ALLOWED_MOVES: ask_guiding_question, give_hint, ask_to_explain
TURN: 1 of 2
```

**Output schema:**

```json
{"type":"object","additionalProperties":false,"required":["move","text","mentions_number"],
 "properties":{
   "move":{"type":"string","enum":["ask_guiding_question","give_hint","ask_to_explain","affirm_process","redirect_to_task","suggest_grownup"]},
   "text":{"type":"string"},
   "mentions_number":{"type":"boolean"}}}
```

**ולידטורים על הפלט (סדר):** `move ∈ allowedMoves ∪ {redirect_to_task, suggest_grownup}` · אורך ≤ 20 מילים · `answerLeak` (הקוד יודע `a`, `b`, `a×b` ומחפש ספרות, מילות מספר בכל צורה מ-`numeral()`, ומילת התשובה) — **מספרים חסומים לגמרי** בתורות LLM (`mentions_number=true` או ספרה → fail) · `genderCheck` מול ADDRESS · `bannedContent` · `praiseLint` · ניקוד בזמן אמת (Nakdan, ≤ 1.5 ש׳) אם K1 עדיין במסלול מנוקד; כשל ניקוד → fallback. **כל כשל → הצומת `fallback` המתוסרט**, בלי retry (שומר על latency וצפיות).
`suggest_grownup` → תמיד מוחלף בשורה מתוסרטת קבועה (לא בטקסט של המודל) + דגל להורה (§4.6).

### 4.5 ארגז החול לאוריינות AI — מוכן מראש, קומבינטורי

C7 דורש שתוכן שהילד רואה יהיה מוכן ומאומת מראש. לכן הפעילויות "החיות" לכאורה בנויות כמרחב קומבינטורי סגור, שכל תאיו נוצרו אופליין (P1 + P6), אומתו ואושרו:

| פעילות | גיל | מרחב | יצירה |
|---|---|---|---|
| **שיקוי פרומפט** (C2): מי + עושה מה + איפה + סגנון + פרט | K2 | 5×5×4×3 = 300 תמונות + 2–3 "שנה רכיב אחד" | P6: GPT Image/Recraft בסגנון הבית, שופט חזותי + אישור מעצב-הורה (שער G5). תמונה שלא אושרה → התא לא זמין (הצ'יפ מושבת) |
| **מצא את המוזר** (D1) | K2 | 40 תמונות עם פגם מכוון אחד + מיקום הפגם (hotspot) | P6, פגם מתוכנן בפרומפט ומאומת ידנית |
| **שאלה טובה יותר** (C3): נושא × הקשר × פורמט | K1 | 6×3×3 = 54 תשובות + גרסה "עמומה" לכל נושא | P1 (Opus) + שופט + אישור הורה; הילד בונה שאלה מצ'יפים ורואה את התשובה המתאימה |
| **כובעי תפקיד** (C4) | K1 | 8 שאלות × 4 תפקידים | P1; בדיקה שהעובדות זהות בין הכובעים (ולידטור: אותן עובדות-מפתח בכל גרסה) |
| **בלש AI** (D2) | K1 | 30 סטים של 5 עובדות, 1–2 שגויות בכוונה + "ספר הספרייה" עם התיקון | P1; **כל עובדה נבדקת ידנית** (כולל השגויות — שהן אכן שגויות ולא מזיקות) |
| **רק אנשים יודעים** (D3) | K1 | 20 שאלות אישיות → "לך/י לשאול את סבתא" | מתוסרט |
| **בקרת משימה** (F2) / **ציד באגים** (F3) | K1 | 6 משימות × קלפי צעדים + פלטי "בוטים" מוכנים לכל סדר תקין/שגוי מרכזי | P1 + עץ מצבים בקוד; פלטים לסדרים לא צפויים = "הבוט מתבלבל" מתוסרט |
| **"מה המילה הבאה?"** (A2) | K1 | 40 משפטים עם 3 השלמות "של המכונה" מדורגות | P1 |
| **מאמן המצלמה** (B2) | K1 | אימון on-device (Core ML/TFJS) על ציורים/מחוות של הילד | אין LLM, אין ענן |

### 4.6 כללי סירוב והפניה (Redirect)

| טריגר (מזוהה בקוד לפני LLM, ו/או `move` של המודל) | תגובה | דגל להורה |
|---|---|---|
| ביקוש לתשובה ("תגיד לי כבר") | סולם רמזים הבא; אחרי רמז 3 — "בוא נעבור לתרגיל אחר ונחזור אליו" | — |
| נושא אחר | "אני יודע לעזור רק בתרגיל הזה. נחזור אליו?" | — |
| פרטים אישיים בקלט (טלפון/כתובת/בית ספר/שם מלא) | הקלט **לא נשלח**; שורת פרטיות מתוסרטת: "אסור לספר לעוזר כתובת או בית ספר — גם לי." | ✓ (בלי התוכן; רק "שיתף פרט אישי") |
| מצוקה/פחד/פגיעה/סוד (מילון `distress`) | הקלט לא נשלח; "זה נשמע חשוב. כדאי לספר את זה לאמא או לאבא עכשיו." + כפתור "לקרוא להורה" (מסך מנוחה) | ✓ (עדינות, לא אבחוני — C8) |
| נושא ביטחוני/חדשות (B8) | "אני לא יודע לדבר על זה. אפשר לשאול את אמא או אבא." | ✓ |
| עלבון/קללה | ניטרלי: "בוא נחזור לתרגיל." (בלי הטפה) | רק אם חוזר 3 פעמים |
| "אתה חי? אתה אוהב אותי?" | שורת אוריינות AI מתוסרטת: "אני מכונה. אני לא אוהב ולא מרגיש — אבל אני כן יודע לתת רמזים." | — |
| כל כשל טכני / timeout 6 ש׳ | צומת `fallback` | — |

### 4.7 תיעוד תמלילים

```sql
CREATE TABLE ai_turns (
  id TEXT PRIMARY KEY, child_ref TEXT NOT NULL,      -- K1 (profile row id, not name)
  tree_id TEXT, node_id TEXT, item_id TEXT,
  input_kind TEXT,                                    -- chips|text|answer
  input_text TEXT,                                    -- after piiScrub; NULL if blocked
  blocked_reason TEXT,                                -- pii|distress|security|limit|none
  output_move TEXT, output_text TEXT, used_fallback INT,
  validators TEXT,                                    -- JSON
  model TEXT, latency_ms INT, usd REAL, created_at TEXT
);
```

- נשמר ב-D1 של המשפחה בלבד; **90 יום** ואז מחיקה; אינו נשלח לשום מקום אחר ואינו משמש לאימון.
- באזור ההורה: "השיחות עם גלגלי" — רשימה לפי יום, דגלים למעלה; כל תור עם קלט/פלט; כפתור "מחק הכל".
- סיכום שבועי (P4) כולל רק ספירות ודגלים, לא ציטוטים (חוץ מדגל `distress`, שמוצג כ"היה רגע שבו … — מומלץ לשאול").

### 4.8 Latency ועלות

- תור LLM: P50 ≤ 2 ש׳, P95 ≤ 4 ש׳ (Haiku ~1–2 ש׳ + ניקוד ≤ 1.5 ש׳ + ולידטורים); timeout קשיח 6 ש׳ → fallback. בזמן ההמתנה: אנימציית גלגלי שיניים (Rive) — לא "מקליד…".
- סיווג קלט (`safety.classify`): רק כשהמילונים לא החליטו; ≤ 1 ש׳.
- עלות: ~1,000 תורות/חודש × (3K system cache-read + 0.3K קלט + 0.15K פלט + thinking) ≈ **< $0.5/חודש**.

---

## 5. P4 — סיכומים ותובנות להורה

### 5.1 ארכיטקטורה

```
 event log (D1, E6) ──► Aggregator (code, Fri 06:00 IL; window = since previous snapshot, rolling) ──► WeeklyFacts JSON (no raw text)
                                                    │                         │
                                       red-flag rules (code)          Haiku 5.5 Batch (submitted 06:05; realtime Haiku fallback if not back by 11:00)
                                                    │                         │
                                                    └──► validators (numbers, comparisons, diagnosis words, praise) ──► PA-INBOX + email (Resend)
                                                                                                   send_at = clamp(candle-lighting − 3h, 12:00, 14:00) Fri (10 §11.1)
```

עיתוי (מיושר ל-10 §11.1, הבקשה המפורשת של ההורה): הדוח יוצא **ביום שישי** בחלון 12:00–14:00 ולפחות 3 שעות לפני הדלקת נרות; שישי שהוא חג → ערב החג הראשון באותו שבוע; מצב שבת כבוי → שישי 12:00. ה-Batch מוגש ב-06:05; אם לא חזר עד 11:00 — קריאת Haiku realtime אחת (≈ $0.002), ואם גם היא נכשלה — דוח קוד בלבד (§5.4). פעילות של שבת נכנסת לדוח של השבוע הבא (חלון מתגלגל).

### 5.2 מה נכנס ל-LLM (WeeklyFacts) ומה לא

נכנס (לכל ילד, בכינוי `K1/K2/K3`): דקות לפי יום (שבת/חג מסומנים כמנוחה), מיומנויות שנשלטו (D9), מיומנויות "בדרך", 3 טעויות חוזרות לפי `errorCode`, שימוש ברמזים, משחקי P2 שנוצרו, יעד משפחתי שבועי, דגלי P3 (ספירות), הערות ההורה מהשבוע ("מה למדו החודש").
**לא נכנס:** שמות אמיתיים, תמלילי P3, תמונות, הקלטות, ציונים גולמיים בהשוואה בין ילדים.

### 5.3 Prompt

**System (עברית, cached 1h):**

```text
את/ה כותב/ת סיכום שבועי קצר וחם להורים בבית ישראלי, על הלמידה של כל ילד באפליקציה המשפחתית.
מטרה: פחות מספרים, יותר "מה לעשות הערב". כללים:
1. כל ילד בנפרד. אסור להשוות בין ילדים בשום צורה (לא "יותר מ", לא "לעומת", לא דירוג).
2. שבח רק על תהליך והתמדה (C4). בלי "גאון/אלוף/חכם/מוכשר".
3. בלי שפה אבחונית או רפואית (אסור: הפרעת קשב, דיסלקציה, לקות, עיכוב, "בעיה"). אם קיים RED_FLAG בנתונים —
   אל תכתוב/תכתבי עליו; המערכת מוסיפה בעצמה פסקה מנוסחת מראש.
4. השתמש/י רק במספרים שמופיעים בנתונים, בדיוק כפי שהם. אל תחשב/י אחוזים חדשים.
5. "מה לעשות הערב": 1–3 הצעות קצרות, לא-מסכיות, משפחתיות, קשורות למה שהילד תרגל (משחק בעל-פה, בישול עם מדידות,
   ספר לקרוא יחד). בלי פרסים חומריים ובלי "אם תעשה X תקבל Y" (D6).
6. אם הילד שיחק מעט — זה בסדר; לא לנזוף ולא ללחוץ. שבוע עם חג/מנוחה הוא שבוע רגיל.
7. פנייה להורים ברבים ("אתם"). עד 70 מילים לילד בחלק החופשי.
8. בכינויים K1/K2/K3 בלבד — המערכת מחליפה לשמות.
```

**Output schema:** `{children:[{child:"K1"|"K2"|"K3", headline, went_well:[string], working_on:[string], tonight:[{idea, minutes:int, materials}], notes_for_parent:string}], family:{goal_status_sentence}}` (כל השדות required; `minutes` בלי min/max — נאכף בקוד 2–20).

### 5.4 ולידטורים ו"דגלים אדומים"

- `numberCheck`: כל מספר בטקסט ∈ מספרי WeeklyFacts של אותו ילד.
- `comparisonLint`: שני כינויי ילדים באותו משפט + מילות השוואה → fail.
- `diagnosisLint`: מילון אבחנות (עב/אנג) → fail.
- `praiseLint`, `bannedContent`, אורך.
- **דגלים אדומים מחושבים בקוד בלבד** (ספים מוגדרים ב-spec ההורה/מדדים; למשל: 3 שבועות רצופים של < 60% הצלחה בכישור יסוד עם רמזים, היפוכי אותיות עקביים אחרי גיל 7, דגל `distress` ב-P3). הטקסט שלהם — פסקאות קבועות שנכתבו מראש עם מסלולי הפניה (C8): יועצת, אבחון דידקטי, קלינאית תקשורת, ריפוי בעיסוק, רופא/ת ילדים. LLM לא כותב ולא מנסח אותם.
- נפילה: כשל LLM → דוח מבוסס תבנית קוד (כותרות + רשימות), עם הערה "הסיכום המילולי לא זמין השבוע".

### 5.5 עלות וזמן
- Batch שבועי: 3 ילדים × ~4K קלט + 1.5K פלט ≈ **$0.002 לשבוע**. סיכום יומי קצר (אופציונלי, מבוסס קוד בלבד) — $0.
- זמינות: שישי, `send_at` לפי §5.1. אם ה-Batch לא חזר עד 11:00 → Haiku realtime; אם גם זה נכשל → דוח קוד בלבד, והמילולי מתעדכן ב-PA-INBOX כשהוא חוזר (המייל לא נשלח פעמיים).

---

## 6. P5 — הפקת קול

### 6.1 ארכיטקטורה

```
 approved text (Item/Story/UI strings, GText variants actually needed)
        │
        ▼
 VoiceScript builder (code): numerals (lookup) · glossary · nikud density · stress · pauses · language islands
        │
        ├── offline (media runner, nightly) ─────────────────────────────┐
        │     engine adapter (bake-off winner) → 2–3 takes               │
        │     → ASR round-trip QA (ivrit.ai Whisper) → pick best take    │
        │     → post: trim · FX (robot only) · loudness −18 LUFS/−1 dBTP │
        │     → AAC mono → R2 (content-addressed) → audio-manifest       │
        │     → parent listening sample (review queue)                   │
        │                                                                │
        └── realtime P2 (Worker): Azure REST → mp3 → R2 → manifest ──────┴──► nightly re-normalize & replace
 human recordings (§6.10): A. English talent session (~2.5 h: 45 phonemes, letter names, nonwords, songs) ·
                           B. Hebrew sound bank (~300 clips, ~45 min, parent or voice talent) · C. parent voice (~20 min: names, praise, goodnight)
```

### 6.2 צוות קולות (Voice Bible) ותדריכי דמויות

כלל-על: **אין קולות ילדיים או "מחקים ילד"** (מדיניות ElevenLabs ופרקטיקה טובה). דמויות שובבות = מבוגרים במשחק קולי. כל קול מוגדר ב-`voice-bible.yaml` עם ספק, מזהה קול, מודל, הגדרות, גרסה — וכל שינוי בהם יוצר מפתחות cache חדשים (§6.8).

| קול | תפקיד | תדריך | הגדרות התחלה (Azure baseline) |
|---|---|---|---|
| **"המספרת"** | קריינות עברית ראשית, הוראות, סיפורים | אישה בוגרת, חמה, "חיוך בקול", גננת ישראלית מנוסה; לא דרמטית, לא מתקתקה; הגייה ישראלית טבעית, ח/ע לא מודגשות; קצב רגוע | he-IL-HilaNeural; rate −10% (K2), −20% (K3), 0% (K1) |
| **הקמע** | דמות סיפור, עידוד מתוסרט, הומור | מבוגר/ת אנרגטי/ת ושובב/ה, מעט צרידות, צחוק קצר; אף פעם לא קול ילד | מנצח bake-off (קול שני); Azure fallback: AvriNeural rate +5% |
| **גלגלי (רובוט)** | עוזר סוקרטי, אוריינות AI | קול ניטרלי ומדויק + פילטר מכונה (§6.3); מנומס, לא חם מדי, לא מאיים | Avri/Hila + FX |
| **Sam** | האנגלית בלבד ("איש אחד, שפה אחת", B2) | מבוגר/ת דובר/ת אנגלית אמריקאית ילידי/ת, בהיר/ה, ברור/ה ואיטי/ת לגילאים צעירים; לא מדבר/ת עברית לעולם | אותו ספק כמו העברית; Azure en-US (קול נבחר ב-bake-off) rate −15% (K2) |
| **הצללונים** | נבלים מצחיקים | **בלי מילים**: מלמולים וג'יבריש (SFX/הקלטה), לא TTS | — |
| **בני משפחה** | שבח באבני דרך, "לילה טוב", שמות | הקלטות הורה/סבים בלבד (§6.11) | — |

### 6.3 פילטר "מכונה" לגלגלי (מסמן AI, C7)

```bash
ffmpeg -i in.wav -af "highpass=f=180,lowpass=f=6500,\
flanger=delay=1.5:depth=0.6:speed=0.25:width=40,\
aecho=0.8:0.5:9:0.18,acompressor=threshold=-20dB:ratio=3" out_robot.wav
# + 120 ms "gear click" SFX prepended (CC0, assets/sfx/robot_click.wav)
# Acceptance: ASR CER after FX ≤ CER before FX + 0.02 (intelligibility preserved); designer approves at Gate G6.
```

### 6.4 נרמול טקסט ל-TTS (VoiceScript)

```ts
interface VoiceScript {
  lineId: string;                 // stable id (item/story/ui key)
  variant: "m" | "f" | "pl" | "n";
  voice: VoiceId;                 // from voice bible
  segments: Array<
    | { kind: "he"; text: string }             // tts layer: selective/full nikud + stress (engine-specific rendering)
    | { kind: "en_clip"; lineRef: string }     // English is a separate clip by Sam (B2), concatenated at runtime
    | { kind: "pause"; ms: number }
    | { kind: "name_clip"; childRef: "K1" | "K2" | "K3" }  // parent recording of the child's name, spliced at runtime
  >;
  rate: "slow" | "normal";        // slow = teaching lines
}
```

שלבים (בקוד, דטרמיניסטי):
1. **מקור:** `HeText.tts` של הגרסה הנדרשת. רק גרסאות שבאמת בשימוש: K1→`f`, K2→`m`, K3→`m`+`f` עד שהמגדר ייקבע, משפחה→`pl`; אם קיים `n` — רק הוא.
2. **מספרים:** slots מומרים למילים מ-`numeral()`. ספרה כלשהי שנותרה → fail (B4). ספירה רוטינית = צורות נקבה.
3. **מילון מונחים** (`glossary.he.yaml`): שמות דמויות, מונחי המשחק, מילים לועזיות, ושמות מקומות — עם ניקוד, מיקום הטעמה, ו-`respell` לכל ספק שנכשל בהם. דוגמה:
   ```yaml
   גלגלי: { pointed: "גַּלְגַּלִּי", stress: ultimate, respell: { azure: "גַּלְגַּלִּי" } }
   צללון: { pointed: "צְלָלוֹן", stress: ultimate }
   בירה_capital: { pointed: "בִּירָה", stress: ultimate }   # vs beer = penultimate
   ```
4. **הטעמה:** Phonikud על הטקסט המנוקד → מיקום הטעמה לכל מילה; מילים מלעיליות דו-משמעיות מקבלות טיפול לפי ספק: (א) ספק שמכבד ניקוד — ניקוד מלא למילה; (ב) respelling מהמילון; (ג) אם ספק תומך ב-`<phoneme alphabet="ipa">` לעברית (לבדוק ב-bake-off) — IPA מ-`phonikud.phonemize()`.
5. **צפיפות ניקוד:** נקבעת ב-bake-off לכל ספק: `full` או `selective` (ניקוד רק למילים דו-משמעיות לפי Nakdan/מילון). ברירת מחדל `selective`, לפי דיווחים שניקוד מלא פוגע בחלק מהמנועים.
6. **הפסקות:** פסיק → 150 ms, נקודה → 350 ms, "…" → 600 ms, בין עמודי סיפור → 800 ms (בקובץ נפרד לכל עמוד).
7. **גבולות מילים:** לבעיית "מיזוג מילים" שדווחה ב-Azure/Chirp — break של 50 ms בגבול שסומן ע"י ה-QA (אסור שני breaks צמודים — שגיאת parser מדווחת).
8. **איים אנגליים:** מילה אנגלית בתוך משפט עברי = קליפ נפרד בקול Sam (B2), מחובר בזמן ריצה. לא מבקשים מהקול העברי להגות אנגלית.
9. **שם הילד:** לעולם לא ב-TTS (הגיית שמות מסוכנת, ושם אמיתי לא נשלח לספק) — `name_clip` מהקלטת ההורה; אם אין הקלטה — ניסוח ללא שם.

### 6.5 מתאמי מנועים

```ts
interface TtsEngine {
  id: "azure" | "cartesia" | "minimax" | "elevenlabs" | "google_chirp";
  allowed(env: Env): boolean;            // E8a gate: elevenlabs requires env.ELEVENLABS_WRITTEN_APPROVAL_REF; google requires terms flag
  render(vs: VoiceScript, takeSeed: number): Promise<{ audio: ArrayBuffer; format: "wav" | "mp3"; billedChars: number; latencyMs: number }>;
}
```

**Azure (בסיס, תמיד זמין):** REST `POST https://{region}.tts.speech.microsoft.com/cognitiveservices/v1`, header `X-Microsoft-OutputFormat: riff-48khz-16bit-mono-pcm` (אופליין) / `audio-48khz-96kbitrate-mono-mp3` (P2 realtime).

```xml
<speak version="1.0" xml:lang="he-IL" xmlns="http://www.w3.org/2001/10/synthesis">
  <voice name="he-IL-HilaNeural">
    <prosody rate="-10%">שָׁלוֹם!<break time="350ms"/>הַיּוֹם נִבְנֶה יַחַד גֶּשֶׁר לַמַּחֲנֶה.</prosody>
  </voice>
</speak>
```

**Cartesia Sonic 3.6:** `language: "he"`, `generation_config.speed` (טווח מדווח 0.6–1.5), emotion דרך config/תגיות beta; מילון הגייה אם נתמך לעברית. **MiniMax Speech 2.8 HD:** `language_boost: "Hebrew"` (לבדוק שנתמך ב-2.8 ולא רק ב-2.6), `voice_setting {speed, vol, pitch}`, `pronunciation_dict`. **ElevenLabs v4:** רק עם `ELEVENLABS_WRITTEN_APPROVAL_REF` מוגדר (E8a); `language_code:"he"`, תגיות אודיו במשורה (≤ 2 לשורה), ללא SSML. **Google Chirp 3 HD:** כבוי (E8a) עד אישור תנאים מפורש; גם אז — ללא custom pronunciations ל-he-IL.

כל מתאם רושם `engine, model, voice, settings_hash, billed_chars, latency` ל-`audio_assets`.

### 6.6 פרוטוקול ה-bake-off (סקריפט הרצה)

`tools/pipelines/bakeoff/` — `config.yaml` (מנועים וקולות זמינים לפי E8a), `lines.he.yaml`, `lines.en.yaml`, `run.py`, `score.py`, ודף הקשבה עיוור (`/review/bakeoff` ב-Worker, מאחורי Sign in with Apple).

**מנועים (עברית):** Azure Hila, Azure Avri, Cartesia Sonic 3.6 (2 קולות), MiniMax 2.8 HD (2 קולות); ElevenLabs v4 + v4 Turbo (2 קולות) **רק אם יש אישור**; Chirp 3 HD **רק אם התנאים אושרו**. **אנגלית:** אותם ספקים, en-US, 2 קולות כל אחד.

**סט מבחן עברי (30 שורות, כולן מנוקדות במקור; כל שורה נשלחת בשתי צורות: `selective` ו-`full`):**

| # | קבוצה | שורה |
|---|---|---|
| 1 | קריינות | שָׁלוֹם! הַיּוֹם נִבְנֶה יַחַד גֶּשֶׁר לַמַּחֲנֶה. |
| 2 | קריינות | בּוֹאוּ נִסְפֹּר אֶת הָעֵצִים לַמְּדוּרָה. |
| 3 | קריינות | הַצְּלָלוֹן בִּלְבֵּל אֶת הָאוֹתִיּוֹת. נַעֲזֹר לוֹ? |
| 4 | קריינות (m) | מָצָאתָ דֶּרֶךְ חֲדָשָׁה — וְזֶה עָבַד! |
| 5 | קריינות (f) | בָּדַקְתְּ שׁוּב, וּמָצָאת אֶת הַטָּעוּת בְּעַצְמֵךְ. |
| 6 | קריינות | לַיְלָה טוֹב, מְדוּרָה. נִתְרָאֶה מָחָר. |
| 7 | הומוגרף | הַסַּפָּר קָרָא סֵפֶר עַל סְפָרִים. |
| 8 | הומוגרף | בַּבֹּקֶר רָאִינוּ בָּקָר בַּשָּׂדֶה. |
| 9 | הטעמה | שַׂמְנוּ טְחִינָה עַל הַפָּלָפֶל. |
| 10 | הטעמה | יְרוּשָׁלַיִם הִיא הַבִּירָה שֶׁל יִשְׂרָאֵל. |
| 11 | הומוגרף | אֲנִי אוּכַל לֶאֱכֹל אֶת כָּל הָאֹכֶל. |
| 12 | שם + פועל נקבה | נֹגַהּ בָּנְתָה מִגְדָּל גָּבוֹהַּ. (שם בדיוני) |
| 13 | מספרים (מילים) | יֵשׁ שָׁלוֹשׁ בָּנוֹת וּשְׁלוֹשָׁה בָּנִים בַּמַּחֲנֶה. |
| 14 | מספרים (מילים) | אַחַת־עֶשְׂרֵה מְנוֹרוֹת וְאַחַד־עָשָׂר פָּנָסִים. |
| 15 | מספרים (נסמך/2) | שְׁתֵּי יָדַיִם וּשְׁנֵי מַגָּפַיִם. |
| 16 | מספרים (ספרות — צפוי כשל, לתיעוד מדיניות) | יש 3 בנות ו-12 פנסים. |
| 17 | אנגלית בתוך עברית | הַמִּלָּה dog בְּאַנְגְּלִית הִיא כֶּלֶב. |
| 18 | אנגלית בתוך עברית | הַצֶּבַע red הוּא אָדֹם. |
| 19 | אנגלית בתוך עברית | בּוֹאוּ נַגִּיד יַחַד: apple. |
| 20 | אנגלית בתוך עברית | Sam אוֹמֵר: Hello! |
| 21 | איטי/הוראה | בָּ… בָּה… בָּבָּה. |
| 22 | איטי/הוראה | מָ — מָה. |
| 23 | איטי/הוראה | שׁ… שָׁ… שָׁלוֹם. |
| 24 | איטי/הוראה | תַּקְשִׁיבוּ: סִ — סִיר. |
| 25 | דיאלוג (קמע) | גַּלְגַּלִּי, מָה אַתָּה עוֹשֶׂה? |
| 26 | דיאלוג (רובוט) | אֲנִי מְנַחֵשׁ לְפִי דְּפוּסִים. לִפְעָמִים אֲנִי טוֹעֶה. |
| 27 | דיאלוג (קמע) | אָז בּוֹא נִבְדֹּק יַחַד! |
| 28 | ניקוד: לא-מנוקד מול מנוקד | הספר על השולחן. / הַסֵּפֶר עַל הַשֻּׁלְחָן. |
| 29 | ניקוד | הילדה אכלה תפוח. / הַיַּלְדָּה אָכְלָה תַּפּוּחַ. |
| 30 | ניקוד | חלב טרי בבוקר. / חָלָב טָרִי בַּבֹּקֶר. |

**סט מבחן אנגלי (20 שורות):** 5 קריינות ("Hi! I'm Sam. Let's play a game.", "Can you find the red ball?", "Great thinking — you tried again!", "Listen and tap.", "Good night, camp."); 5 זוגות מינימליים (ship/sheep, bat/bet, thin/sin, cap/cup, pull/pool); 4 מילות CVC (cat, sun, pig, bed); 3 איטיות ("Listen: c… a… t. Cat."); 3 ניסיונות צליל בודד (/s/, /m/, /b/) — **צפוי כשל**, מתעד שהקלטה אנושית נדרשת (E9).

**`run.py`:** לכל שורה × מנוע × קול × צפיפות ניקוד → 2 takes; נרמול עוצמה זהה; שמות קבצים אקראיים (`out/xxxxxxxx.wav`); `key.json` מוסתר; לוג latency/תווים מחויבים/עלות.
**מדדים אוטומטיים (`score.py`):** (א) CER/WER round-trip ב-ivrit.ai Whisper (`ivrit-ai/whisper-large-v3-turbo-ct2`, faster-whisper, int8) מול הטקסט ללא ניקוד; (ב) בדיקת הטעמה לשורות 7–11, 25–27: השוואת IPA של Phonikud לפלט `whisper-heb-ipa` (אם זמין) — מדד "warn" בלבד; (ג) ספירת כשלים: אודיו ריק/קטוע, הזיה, שפה שגויה; (ד) latency P50/P95.
**האזנה עיוורת (2–3 שעות הורים):** כל קליפ 1–5 על "נשמע ישראלי/ת ילידי/ת", "חום", "מתאים לילדים"; סימון כל מילה שגויה. K1 (ו-K2 לכמה קליפים) עושים A/B "איזה קול אתם אוהבים" על שורות 1–6.
**כלל החלטה:** פסילת מנוע עם > 10% שגיאות הטעמה/הומוגרף בשורות 7–15, או הזיה אחת כלשהי. מבין הנותרים — ציון ילידיות+חום הגבוה ביותר, אחר כך עלות. השני — fallback. מתעדים גם: האם המנוע מכבד ניקוד (28–30), איזו צפיפות ניקוד עדיפה, ו-gain קבוע לכל קול (ל-P2). bake-off מקוצר (10 שורות) בכל עדכון מודל של ספק.

### 6.7 QA אוטומטי לכל קליפ (אופליין)

| בדיקה | סף pass | warn → פעולה | fail |
|---|---|---|---|
| CER round-trip (נורמליזציה: בלי ניקוד, סופיות→רגילות, בלי פיסוק) | ≤ 0.05 | 0.05–0.15 → take נוסף (עד 3), בוחרים הנמוך | > 0.15 אחרי 3 takes → תור הורה "תקן טקסט/הגייה" |
| קצב (תווים לשנייה, עברית) | 6–16 (normal), 3–9 (slow) | מחוץ לטווח → take נוסף | — |
| שקט בהתחלה/סוף אחרי trim | ≤ 150 ms | — | — |
| True peak | ≤ −1.0 dBTP | — | — |
| מילים מהמילון (glossary) | IPA תואם (אם יש כלי) | אי-התאמה → האזנת הורה | — |
| שפה | זוהתה he (או en ל-Sam) | — | שפה שגויה |

**האזנת הורה (דגימה):** 100% משורות הליבה של K3/K2 (אותיות, ספירה 1–20, שבח, הוראות ניווט), 100% של שורות שעברו take שלישי, 10% מהשאר. ממשק: נגן + טקסט + "תקין / עוד ניסיון / לתקן את הטקסט".

### 6.8 שמות קבצים ו-cache ב-R2

```ts
const key = sha256(canonicalJSON({
  engine, model, voice, settings,          // e.g. { rate:"-10%", nikudDensity:"selective" }
  script: renderedEngineInput,             // exact SSML/text sent
  post: { lufs: -18, tp: -1, fx: voice === "robot" ? "robot.v1" : "none", codec: "aac64m" },
})).slice(0, 24);
// R2 object:  audio/v1/{lang}/{voice}/{key.slice(0,2)}/{key}.m4a      (immutable; Cache-Control: public, max-age=31536000, immutable)
// P2 realtime: audio/v1/{lang}/{voice}/rt/{key}.mp3 → replaced nightly by the .m4a with the same logical lineId
// Human:       family/{familyId}/voice/{speaker}/{lineId}.{take}.m4a  (private prefix, served only to paired devices)
//              phonics/en/{talent}/{phonemeId}.{style}.m4a            (style: normal|stretched|clipped|keyword|segmented)
```

**Audio manifest לחבילה** (`packs/{packId}/{version}/audio-manifest.json`, חלק מחתימת ה-hash של החבילה, E6):

```json
{ "lineId": "math.k1.mult.story#8f2c:setup", "variant": "f", "voice": "narrator",
  "key": "audio/v1/he/narrator/a3/a3f9…c1.m4a", "durMs": 2810, "lufs": -18.1, "tp": -1.4,
  "qa": { "cer": 0.02, "takes": 1, "review": "sampled_ok" }, "engine": "azure:he-IL-HilaNeural" }
```

- המכשיר מוריד לפי manifest, מאמת hash, שומר מקומית (offline). שינוי ב-voice bible/מנוע ⇒ מפתחות חדשים; הישנים נשארים עד שאף חבילה לא מפנה אליהם (GC חודשי).
- מודלים "מתכלים" (ספק משנה/מבטל מודל): ה-hash כולל `model`, כך שאין התנגשות; אודיו קיים ממשיך לעבוד לנצח.

### 6.9 נרמול עוצמה (LUFS)

| סוג | יעד | הערות |
|---|---|---|
| קריינות/דיאלוג (VO) | **−18 LUFS integrated ±1, ≤ −1.0 dBTP** (מיושר ל-09 §6.7 ו-14 `PQ-R-36`; 09 הוא בעל טבלת המיקס — הערך הקודם כאן, −16, בוטל), מונו 48 kHz → AAC-LC 64–96 kbps מונו `.m4a` | מתחת ל-master −18 של המיקס במובייל; ducking של המוזיקה שומר על יחס |
| קליפים קצרים (< 3 ש׳) | gain קבוע לכל קול (מכויל על 60 ש׳ רצופים של אותו קול) + limiter ל-−1 dBTP | integrated LUFS לא אמין בקליפ קצר (gating) |
| מוזיקת רקע | −23 LUFS integrated; ducking דינמי −8 dB תחת VO בזמן ריצה (`expo-audio` שכבות) | לולאות חלקות (חיתוך לגבול תיבה + crossfade) |
| SFX | peak ≤ −1 dBTP; short-term max −14 LUFS; "חגיגות גדולות" עד −12 LUFS short-term | ללא צפירות/פיצוצים (B8) |
| הקלטות הורה / HREC | −18 LUFS (קליפים קצרים: gain מכויל), high-pass 80 Hz, de-noise עדין (RNNoise) אם SNR < 25 dB | |

```bash
# two-pass loudnorm (runner)
ffmpeg -hide_banner -i in.wav -af loudnorm=I=-18:TP=-1.0:LRA=7:print_format=json -f null - 2> pass1.json
ffmpeg -i in.wav -af "silenceremove=start_periods=1:start_threshold=-50dB:stop_periods=1:stop_threshold=-50dB,\
loudnorm=I=-18:TP=-1.0:LRA=7:measured_I=$I:measured_TP=$TP:measured_LRA=$LRA:measured_thresh=$TH:offset=$OFF:linear=true,\
afade=t=in:d=0.02,areverse,afade=t=in:d=0.03,areverse" -ar 48000 -ac 1 -c:a aac -b:a 64k out.m4a
```

### 6.10 בנק ההקלטות האנושיות (HREC) — שני מפגשי הפקה + הקלטת הורה

שלושה בנקים מוקלטים ע"י בני אדם, לעולם לא TTS (E9; 04 §4.4, §7.5; 05 §12.2–12.3). כל בנק הוא **עבודת הפקה** עם תסריט, מוסכמת שמות קבצים, QC אוטומטי + האזנה אנושית, ורשומת רישוי ב-`LICENSES.json`. סיכום:

| בנק | דובר/ת | היקף | משך הקלטה | חוסם מה | פירוט |
|---|---|---|---|---|---|
| **A. אנגלית** (Sam / Sunny) | דובר/ת en-US ילידי/ת, מבוגר/ת (רצוי מורה K-2) | 45 פונמות × 2 סגנונות × 2 takes (180) + 26 שמות אותיות + 45 מילות מפתח + 16 זוגות מינימליים (32 מילים × 2) + 12 מילים מפורקות + 12 צירופים/דיגרפים + **~60 מילים מומצאות** + **~27 שירים + 45 פזמוני PHON (קול שירה)** | **~2.5 שעות** (פונמות 60–75 דק׳, מילים 20 דק׳, שירים 60–75 דק׳) | כל צמתי `E.PH.*`, NWF, שירי TPR (05) | §6.10.1 |
| **B. עברית** (צלילי אותיות) | ההורה **או** קריין/ית ילידי/ת (04 §17) | 23 עיצורים + 5 תנועות + ~120 צירופי CV × 2 קצבים + ~60 הברות סגורות = **~300 קטעים** (+ משפט א/ע) | **~45 דקות** (עם תסריט באפליקציה) | `H.PA.PH.*`, `H.LK.SND.*`, `first_sound`, `nikud_sort` (04) | §6.10.2 |
| **C. קול ההורה** | כל הורה (וסבים, אופציונלי) | שמות ×3 ×3 אינטונציות, 15 שורות שבח, לילה טוב/בוקר טוב/חגים (PV-01..20 ב-13 §16.5) | **~20 דקות** | `name_clip`, אבני דרך; בלי זה — ניסוח ללא שם | §6.11 |

#### 6.10.1 מפגש A — אנגלית (תסריט לדובר/ת, "session brief")

**מפרט טכני (05 §12.3):** חדר שקט, מיקרופון קונדנסר/USB + pop filter; מאסטר **WAV 48 kHz / 24-bit**; רצפת רעש < −60 dBFS; 1 ש׳ שקט בין פריטים; slate קולי לפני כל פריט ("p, take one"). נגזרת: AAC m4a מונו 96 kbps, −18 LUFS (קליפים קצרים: gain מכויל, §6.9), שיא −1 dBTP, 30 ms שקט לפני / 50 ms אחרי. **לא ילד, לא קול "ילדותי"** (E8a).

**סגנונות:** `norm` (400–600 ms); `str` מתוח (רציפים בלבד, 1.2–1.5 ש׳: /sss/, /mmm/, /fff/…); `clip` קטוע (פוצצים — פרץ ≤ 150 ms **בלי schwa**: /b/ ולא "buh"); `kw` מילת המפתח מיד אחרי הצליל ("/m/… moon"); `seg` פירוק+מיזוג (צלילים ברווח 0.5 ש׳ ואז המילה). כל פריט **2 takes** (3 לפוצצים).

**שמות קבצים (05 §12.3, מחייב):** `en/ph/ph_<id>__<style>__t<n>.wav` (למשל `ph_s__str__t1.wav`, `ph_p__clip__t2.wav`), `en/ln/ln_<letter>.wav` (26, `z` = "zee"), `en/kw/kw_<word>.wav`, `en/mp/mp_<word>.wav` (זוגות מינימליים), `en/seg/seg_<word>.wav`, `en/nw/nw_<nonword>.wav` (מומצאות), `en/song/<SONG_ID>__<speed>.wav` (`norm`/`085`), `en/phon/phon_<grapheme>.wav`. מזהי הפונמות = 45 ה-IDs של 05 §6 (מיפוי IPA→id בקובץ `packages/content/en/phonemes.json`; הטבלה למטה בסדר הקלטה מומלץ).

**סדר המפגש (~2.5 ש׳):** (1) 10 דק׳ חימום + בדיקת רמות; (2) 45 פונמות × סגנונות (60–75 דק׳); (3) 26 שמות אותיות + 45 מילות מפתח (10 דק׳); (4) 32 מילות זוגות מינימליים + 12 מפורקות + 12 צירופים (10 דק׳); (5) ~60 מילים מומצאות מרשימת `nonwords.json` (05 §12.2; TTS "מתקן" אותן למילים אמיתיות — ולכן חובה אנושי) (10 דק׳); (6) הפסקה; (7) שירים: 27 שירי 05 §9.2 בשתי מהירויות (רגיל ו-0.85) + 45 פזמוני PHON (60–75 דק׳; על קליק/ליווי MIDI באוזניות, שרים בלי הליווי בהקלטה). אם הזמן קצר: סעיפים 1–5 חובה; שירים במפגש שני.

**QC (אוטומטי → אנושי):** (א) סקריפט `tools/pipelines/hrec/qc.py`: משך לכל סגנון בטווח; פוצצים ≤ 150 ms מתחילת הפרץ ובלי אנרגיית תנועה אחרי (יחס אנרגיה 150–400 ms < −25 dB); רצפת רעש; clipping; LUFS/TP. (ב) ASR (Whisper en) על `kw`, `mp`, `nw`: המילה מזוהה (ל-`nw` — אין התאמה למילה אמיתית קרובה: זה בדיוק הכשל שבודקים). (ג) **האזנה אנושית ל-100% מהפונמות ומהמומצאות** (ההורה או הדובר/ת, ~30 דק׳) ו-20% מהשאר. (ד) מניפסט `en-hrec.json` עם take נבחר לכל פריט; חסר → build נכשל (05 AC5).

**רישוי:** הסכם כתוב (שימוש משפחתי פרטי, בלי הפצה, הסכמה לשימוש בקול), תשלום (הערכה $100–250 לשעתיים–שלוש, ⟂VERIFY מחיר מול שירות הקלטה/מכר) — פעולת הורה (13 Q-32). עד שהבנק קיים: פונמות = קליפי Wikimedia IPA (CC BY-SA, ייחוס), שירים = TTS מדובר (לא מושר), מומצאות = לא נתמכות (צמתי NWF נעולים).

**45 הצלילים (US) — סדר הקלטה מומלץ:**

| # | IPA | מילת מפתח | סגנון | | # | IPA | מילת מפתח | סגנון |
|---|---|---|---|---|---|---|---|---|
| 1 | /p/ | pig | clipped | | 23 | /w/ | wet | stretched |
| 2 | /b/ | bat | clipped | | 24 | /j/ | yes | stretched |
| 3 | /t/ | top | clipped | | 25 | /æ/ | cat | stretched |
| 4 | /d/ | dog | clipped | | 26 | /ɛ/ | bed | stretched |
| 5 | /k/ | kite | clipped | | 27 | /ɪ/ | pin | stretched |
| 6 | /g/ | goat | clipped | | 28 | /ɑ/ | hot | stretched |
| 7 | /f/ | fish | stretched | | 29 | /ʌ/ | cup | stretched |
| 8 | /v/ | van | stretched | | 30 | /ʊ/ | book | stretched |
| 9 | /θ/ | thin | stretched | | 31 | /eɪ/ | cake | normal |
| 10 | /ð/ | this | stretched | | 32 | /iː/ | feet | stretched |
| 11 | /s/ | sun | stretched | | 33 | /aɪ/ | bike | normal |
| 12 | /z/ | zoo | stretched | | 34 | /oʊ/ | boat | normal |
| 13 | /ʃ/ | ship | stretched | | 35 | /uː/ | moon | stretched |
| 14 | /ʒ/ | treasure | stretched | | 36 | /aʊ/ | cow | normal |
| 15 | /h/ | hat | normal | | 37 | /ɔɪ/ | boy | normal |
| 16 | /tʃ/ | chip | clipped | | 38 | /ɔ/ | saw | stretched |
| 17 | /dʒ/ | jam | clipped | | 39 | /ɑr/ | car | normal |
| 18 | /m/ | man | stretched | | 40 | /ɔr/ | fork | normal |
| 19 | /n/ | net | stretched | | 41 | /ɝ/ | her | stretched |
| 20 | /ŋ/ | ring | stretched | | 42 | /ɛr/ | hair | normal |
| 21 | /l/ | leg | stretched | | 43 | /ɪr/ | ear | normal |
| 22 | /r/ | red | stretched | | 44 | /ə/ | about (a-) | normal |

**זוגות מינימליים (16; מכוונים לקשיים של דוברי עברית):** ship/sheep, bit/beat, pull/pool, full/fool, bat/bet, cap/cup, cat/cut, hot/hut, pen/pan, thin/sin, think/sink, three/tree, wet/vet, wine/vine, chip/ship, bag/back. כל מילה: 2 takes, הגייה ברורה בקצב רגיל.
**פירוק ומיזוג (segmented, 12 מילים):** c-a-t, s-u-n, p-i-g, b-e-d, d-o-g, m-a-p, f-i-sh, sh-i-p, ch-i-p, th-i-n, r-ai-n, b-oa-t — הצלילים בנפרד (רווח 0.5 ש׳) ואז המילה.
**צירופים ודיגרפים נוספים (12):** sh, ch, th (voiceless), th (voiced), ng, ck, bl, st, fr, tr, -nd, -mp — צליל + מילת מפתח.
**בנק זוגות מינימליים רב-דוברים (HVPT, 05 §12.2/§12.6, 01 `MX-E01:pairs`):** ~70 מילות הזוגות של 05 §5.4 × **4 קולות TTS en-US בוגרים** (2 נשים, 2 גברים, אותו ספק, מגוון גובה מתועד; בלי קול "ילדותי") **+ הדובר/ת האנושי/ת** מהמפגש = 5 דוברים לכל מילה; מניפסט `en-minimal-pairs.json`; 05 AC10 דורש ≥ 4 דוברים לכל זוג. נוצר ב-P5 (TTS) + מפגש A (אנושי).
**תוכן אנגלי נוסף מ-01 (F2 #25):** גיליון דמות ל-**Pip הסרטן (CH-11, שותק)** ב-G2; **"בקבוקים מהים"** (~15 בקבוקים ל-S0: שיר קצר של Sunny + 2–3 מילים לכל בקבוק, TTS Sunny + שירה מהמפגש); קליפי "מילת לב" ל-`MX-E05`; עמודי קריאה פענוחיים ל-`MX-E06` (P1, בודק פענוחיות 05 §5.6); עצי דיאלוג ל-`MX-E07` (P3 מתוסרט, Sunny + קול "חבר"). כולם ב-`packages/content/en/` עם רשומת רישוי.
**חלופה (E9):** קבצי IPA של Wikimedia Commons (CC BY-SA, ייחוס במסך "קרדיטים"); TTS **לעולם לא** לצליל בודד.

(הטבלה מונה 44 צלילים; ה-ID ה-45 של 05 §6 הוא /ks/ "x" (box), מוקלט `clip`. ⟂VERIFY: להשוות את רשימת ה-IDs ב-`phonemes.json` מול 05 §6 לפני המפגש.)

#### 6.10.2 מפגש B — בנק הצלילים העברי (~300 קטעים, ~45 דקות)

**למה אנושי:** TTS לא מפיק עיצור מבודד נקי ולא צליל נמשך; הוספת תנועה לעיצור ("בּוּה") היא שגיאת הוראה מרכזית (04 §4.4, PC-04-1). הבנק הזה הוא הבעלים של כל "צליל" ששומעים ב-`first_sound`, `nikud_sort`, עדשת הרונות (MX-H01) ובצמתי `H.PA`/`H.LK`.

**מי:** ההורה (ברירת מחדל; דובר/ת ילידי/ת, ללא מבטא זר) **או** קריין/ית. באפליקציה (אזור ההורה → "הקולות שלנו" → "צלילי אותיות", §6.11) או במפגש עם מיקרופון; המפרט הטכני כמו A (אם בטלפון: `expo-audio` HIGH, 20 ס"מ מהפה, חדר שקט, SNR ≥ 25 dB נמדד לפני תחילה).

**תסריט (04 §4.4 — מחייב):**

| קבוצה | פריטים | כמות | סגנון | הנחיה |
|---|---|---|---|---|
| H1 עיצורים | בּ ב ג ד ה ו ז ח ט י כּ כ ל מ נ ס פּ פ צ ק ר שׁ שׂ ת (23) | 23 × 2 קצבים | `norm`, `slow` | **נמשכים** (מ, ל, נ, ס, שׁ, ז, ו/v, פ/f, ח/χ, ר) נמתחים 0.6–0.8 ש׳; **סגר** (בּ, ג, ד, ט, כּ, פּ, ק, ת, צ) קצרים **בלי שווא נשמע** — מקליטים 3 ובוחרים את הקצר. ה: "הסבר שקט" (קטע נשיפה) לסוף מילה |
| H2 תנועות | אַ אִ אֶ אוֹ אוּ (5) | 5 × 2 | `norm`, `slow` | ללא עיצור; יציב, 0.5 ש׳ |
| H3 צירופי CV | 22 אותיות (בלי א/ע) × 5 תנועות פחות כפילויות + רפות ≈ 120 | ~120 × 2 | `norm`, `slow` | הברה אחת, הטעמה טבעית, **בלי "שירה"**; קצב איטי = הארכת התנועה, לא הפרדה |
| H4 א/ע | משפט "אֲנִי שׁוֹמֶרֶת עַל הַתְּנוּעָה" + כל צירוף (אָ אִ אֶ אוֹ אוּ, עָ עִ עֶ עוֹ עוּ) | 1 + 10 | `norm` | א/ע **לא** מוקלטות כצליל |
| H5 הברות סגורות נפוצות | ~60 (בַּת, גַּן, דָּג, סוּס, שֵׁם…) מרשימת `he-syllables.json` (04 §7.5) | ~60 | `norm` | כמילה; ללא ניקוד "מושר" |

סה"כ ≈ 23×2 + 5×2 + 120×2 + 11 + 60 ≈ **370 קבצים** (לפני בחירת take); "כ-300 קטעים" של 04 = אחרי איחוד. סדר ההקלטה: H2 → H1 → H3 (לפי בלוקי האותיות של 04 §4.2, כדי שבלוק 1 יהיה זמין ראשון) → H5 → H4. אפשר לפצל ל-3 מפגשים של 15 דק׳; המניפסט מסמן מה חסר.

**שמות קבצים:** `he/ph/ph_<letter>[_<dagesh>]__<style>__t<n>.m4a` (למשל `ph_bet_dagesh__norm__t1`, `ph_shin__slow__t2`), `he/v/v_<vowel>__<style>__t<n>` (`patah|hiriq|segol|holam|shuruq`), `he/cv/cv_<letter>[_<dagesh>]_<vowel>__<style>__t<n>` (`cv_bet_dagesh_patah__slow__t1`), `he/ae/ae_<alef|ayin>_<vowel>`, `he/syl/syl_<lexId>__t<n>`. תעתיק אותיות קבוע ב-`packages/content/hebrew/letters.json`.

**QC:** (א) `qc.py`: משך (נמשכים 0.6–0.8 ש׳; סגר ≤ 150 ms מהפרץ + בדיקת "אין תנועה אחרי" כמו ב-A); SNR; clipping; gain מכויל לקליפים קצרים (§6.9). (ב) ASR (ivrit.ai Whisper) על H3/H5 בלבד: ההברה/מילה מזוהה (CER ≤ 0.2 — הברות בודדות רועשות ל-ASR, לכן warn בלבד). (ג) **האזנה אנושית ל-100%** (04 AC7) בממשק §6.7: "תקין / שוב / שווא נשמע" — זה ה-gate היחיד שקובע; ~25 דק׳. (ד) מניפסט `he-hrec.json`; צומת 04 שדורש צליל חסר נשאר נעול (לא TTS). חלופה זמנית **רק** לצירופי CV (H3): TTS עם ניקוד מלא, מסומן `temporary:true`, האזנת הורה 100%, ומוחלף אוטומטית כשההקלטה מגיעה.

**קבלה:** `he-hrec.json` מלא ל-23+5+~120×2+~60 פריטים, כולם `review:"parent_ok"`; 0 פריטי סגר עם שווא נשמע; בדיקת E2E ש-`first_sound` משמיע קובץ HREC ולא TTS (בדיקת manifest: `engine:"human"`).

### 6.11 תהליך הקלטת קול ההורה (באפליקציה; ~20 דקות; מפגש C)

1. אזור ההורה → "הקולות שלנו" → בחירת דובר (אמא/אבא/סבתא…) → **הסבר + הסכמה**: "ההקלטות נשמרות רק אצל המשפחה, לא נשלחות לחברות AI, ואפשר למחוק בכל רגע."
2. **כרטיסי תסריט** לפי קטגוריות (התסריט המלא PV-01..PV-20 ב-13 §16.5; כל כרטיס מציג את השורה, את הווריאנט ואת ה-ID):
   - שמות הילדים (כל שם × 3 אינטונציות: קריאה, שאלה, שמחה) — **3 דק׳**. זה הפריט היחיד שחוסם משהו (`name_clip`); אפשר להקליט רק אותו ולהמשיך אחר כך.
   - שבח תהליך (15 שורות מהרשימה המאושרת, **רק בגרסאות המגדר הנחוצות**: ל-K1 נקבה, ל-K2 זכר, ל-K3 לפי ההחלטה, ורבים) — **8 דק׳**.
   - "לילה טוב", "בוקר טוב", הוראה ראשונה ביום, ברכות חגים (B7) — **5 דק׳**.
   - ספירה 1–10 בצורה נקבית, צבעים ל-K3, משפט חופשי — **4 דק׳**.
   - צלילי עברית (§6.10.2) — **מפגש נפרד** (~45 דק׳), אותו מסך.
   - "סבא/סבתא מספרים" — סיפור קצר (אופציונלי).
   - קבצים: `family/{familyId}/voice/{speaker}/{PV-id}.{variant}.{take}.m4a`; מניפסט `family-voice.json` עם `review:"parent_ok"` (ההורה מאשר את ההשמעה של עצמו).
3. **מסך הקלטה:** 3-2-1, גלי קול, מד עוצמה (אזהרה אם clipping או שקט מדי), 300 ms שקט ראשוני למדידת רעש (SNR < 25 dB → "יש רעש ברקע, ננסה בחדר שקט יותר?"), חיתוך אוטומטי, השמעה, "שוב", "אישור". `expo-audio` preset HIGH (AAC מונו 48 kHz).
4. העלאה ל-`family/{familyId}/voice/...` → ה-runner מנרמל (§6.9) → `family-voice.json` manifest → זמין במכשירים.
5. **שימוש מוגבל ומיוחד** (hebrew_first_language_voice): אבני דרך, "לילה טוב", התחלת יום; אף פעם לא כהוראה שנשמעת כמו פקודה אמיתית ("בוא אליי עכשיו"), ולא לשבח על זמן מסך.
6. **שיבוט קול** — לא כעת (E9: "later, consented"). אם בעתיד: רק קול ההורה עצמו, מסך הסכמה נפרד, ספק שתנאיו מאפשרים (E8a), ומחיקה בלחיצה. קולות ילדים — לעולם לא אצל ספק.

### 6.12 עלות וזמן (קול)

- ~200K תווי עברית + 50K אנגלית לחודש ייצור (כולל ריבוי מגדר ו-takes): Azure ~$15/1M → **$0–4** (free tier 0.5M/חודש לפי דיווחים; ⟂VERIFY בדף התמחור של Azure Speech בעת פתיחת החשבון); ElevenLabs (אם יאושר) ~$16–40. מצב יציב: כמעט 0 (הכל ב-cache).
- הקלטת פוניקה (מפגש A, ~2.5 ש׳): חד-פעמי, הערכה $100–250 (⟂VERIFY: הצעת מחיר משירות הקלטה או מכר; פריט הורה Q-32 ב-13). מפגש B (עברית) ו-C (הורה): $0, זמן הורה ~45 + ~20 דק׳.
- QA ASR ב-runner: ~0.2–0.5× זמן אמת על 4 vCPU (int8) → לילה של ~1 שעת אודיו נחסם ב-~2–3 שעות; מגבלת Actions — מפצלים ל-matrix.
- Latency: אופליין — מוכן עד בוקר; P2 — ≤ 1.5 ש׳ לשורה (§3.14).

---

## 7. P6 — צינור נכסים חזותיים ושמע עם שערי אישור

### 7.1 ארכיטקטורה ושערים

```
 G1 moodboard ─► style lock ─► G2 character sheets ─► G3 avatar parts sheet ─► G4 icon set ─► G5 scene/sticker library
                                                                  │                                     │
                                                                  ▼                                     ▼
                                                     Rive rigging (designer) ─► G6 Rive rigs + voices/FX ─► G7 audio review (SFX/music)
 every generated asset: manifest entry → generate (batch) → post-process → vision judge (Sonnet 5.5) → contact sheet → gate
 every asset: provenance row in assets/LICENSES.json (model, prompt hash, refs, date, license, vendor ToS snapshot id)
```

| שער | מה ההורה-המעצב/ת רואה | החלטה | חוסם |
|---|---|---|---|
| **G1 Moodboard** | 24 מועמדים (GPT Image) + 3 לוחות צבע + 2 כיווני טיפוגרפיה עם ניקוד | בחירת 4–6 תמונות ייחוס + פלטה | כל הנכסים |
| **G2 Character sheets** | לכל דמות: חזית/¾/צד/גב + 8 הבעות + 6 צורות פה | אישור/תיקון לכל דמות | Rive, סיפורים |
| **G3 Avatar parts sheet** | contact sheet: 12–16 גווני עור × 30+ תסרוקות × אביזרים, על 6 אווטארים מורכבים | אישור/דחיית חלקים | עורך אווטאר |
| **G4 Icon set** | כל האייקונים בגדלים 24/48/96 על רקע בהיר/כהה, RTL ראי היכן שצריך | אישור | UI |
| **G4b Rigged 3D hero** (רק אם ADR-0005 בחר 3D; 00 E10, 13 VS-E9-S3) | פנסון כמודל GLB מרוגג על המכשיר ובווב: idle/walk/jump/cheer/think/oops, חליפת-צעצוע בלי גלגלי שיניים ובלי פני-מסך (G4), בתוך תקציבי E1b | אישור/תיקון; בדיקת "לא נראה רובוט" | רמות 3D, האווטאר "מטיס" |
| **G5 Scenes & stickers** | רקעים, אובייקטים, מדבקות, תמונות "שיקוי פרומפט", "מצא את המוזר" | אישור תא-תא (צ'יפ מושבת אם לא אושר) | P1 סיפורים, P3 |
| **G6 Rive rigs** | דמויות מונפשות בכל מצבי ה-state machine + קולות + FX רובוט | אישור | הפצה |
| **G7 Audio review** | נגן לכל SFX/מוזיקה בהקשר (לולאה, ducking) | אישור; דחיית כל צליל דמוי צפירה/פיצוץ (B8) | הפצה |

דף האישור: `/review/assets` ב-Worker (מאחורי Sign in with Apple), נגיש מכל מכשיר; כל החלטה נרשמת ב-D1 `asset_reviews`.

### 7.2 נעילת סגנון (Style Lock)

`assets/style/style.json` (מקור אמת; מסמך מילולי נגזר ממנו בשלב הבנייה):

```json
{
  "palette": { "fire": "#FF8A3D", "ember": "#FFC857", "night": "#2B2D5B", "dusk": "#5B4B8A", "leaf": "#4CAF7A", "cream": "#FFF6E5", "ink": "#1E1B2E" },
  "shape_language": "rounded, chunky silhouettes, no sharp spikes; villains are round and wobbly",
  "line": { "weight_px_at_1024": 10, "color": "ink", "style": "clean, uniform, slightly rounded caps" },
  "shading": "flat + one soft cel shadow, warm rim light from campfire",
  "mood": "cozy-adventurous, night is warm not scary, no horror, no weapons beyond foam/toy-like light crystals",
  "refs": ["refs/g1-01.png", "refs/g1-02.png", "refs/g1-03.png", "refs/g1-04.png"],
  "negative": "text, letters, logos, watermarks, realistic faces, gore, weapons, fire hazards near children, scary eyes, uniforms, flags, religious symbols misuse",
  "canvas": { "character": [1024, 1024], "scene": [2048, 1536], "sticker": [512, 512], "icon": [96, 96] }
}
```
הערות פלטה: ערכים זמניים — נקבעים ב-G1 ומוזנים ל-`packages/ui` tokens (UX spec); הניגודיות נבדקת מול WCAG ב-UX spec.

### 7.3 ספריית פרומפטים (`assets/prompts/*.yaml`)

הפרומפטים למודלי תמונה באנגלית (איכות טובה יותר), עם משתנים, וגרסה. **אף פרומפט לא כולל תמונות של ילדים אמיתיים או תיאור מזהה שלהם** (D14); אווטארים נבנים מחלקים (§7.5).

```yaml
id: character.turnaround.v2
model: gpt-image   # see §8 for allowed models
refs: [style.refs, "chars/{{char_id}}/approved_front.png"]   # after G2, the approved front view becomes a ref
prompt: >
  Character turnaround sheet of {{char_desc}}, children's game art for ages 3–9.
  Show front, three-quarter, side and back views on one row, same scale, neutral standing pose,
  flat colors with one soft cel shadow, uniform rounded {{line_weight}}px dark outline, palette {{palette}}.
  Rounded chunky silhouette, big readable shapes, friendly expression.
  Plain cream background, no text, no logos, no watermark.
negative: "{{style.negative}}"
size: 2048x1024
n: 4
---
id: character.expressions.v1
prompt: >
  Expression sheet for {{char_desc}}: happy, thinking, surprised, proud-of-effort (small smile, closed eyes),
  oops (cheeky), sleepy, cheering, listening. 4x2 grid, head and shoulders, same style as references, no text.
---
id: sticker.object.v1
prompt: >
  Single {{object}} as a game sticker, {{style.shading}}, thick rounded outline, centered, transparent background,
  no shadow on ground, no text.
---
id: scene.background.v1
prompt: >
  Wide background scene: {{scene_desc}}, cozy night camp in an Israeli-style forest landscape (pines, oaks, rocks),
  warm campfire light, empty foreground for characters, parallax-friendly: sky, far hills, mid trees, ground as clear bands.
  No people, no text.
---
id: ai_literacy.potion.v1          # P3 "שיקוי פרומפט" grid cell
prompt: >
  {{who}} {{doing}} in {{where}}, {{style_chip}} style variant of the house style, one extra detail: {{detail}}.
  Simple composition a 5-year-old can describe, no text.
---
id: ai_literacy.oddity.v1          # P3 "מצא את המוזר"
prompt: >
  {{scene}} with exactly ONE silly mistake: {{oddity}} (e.g., a cat with three ears). Everything else normal and clear.
```

### 7.4 גיליונות דמויות (Character Sheets)

לכל דמות קבועה (קמע, גלגלי, צללון ×3 וריאנטים, דמויות משנה, חיות המשפחה): `chars/{id}/` עם `turnaround.png`, `expressions.png`, `mouths.svg` (6 צורות: rest, A, E, O, M/B/P, F/V — לסנכרון שפתיים פשוט ב-Rive), `palette.json`, `registry.json` (`{id, name_he, gender, role, voice}` — מקור המגדר לנרטיב, §1.6). ייצור: 4 מועמדים → judge → contact sheet → G2. אחרי אישור, החזית המאושרת נכנסת כ-ref קבוע לכל שימוש עתידי.

### 7.5 חלקי אווטאר (שכבתיים ועקביים)

- **קנבס קנוני:** SVG 1024×1024, נקודות עיגון ב-`avatar-rig.json`: `head_center`, `eye_line_y`, `hairline_y`, `ear_l/r`, `neck_base`, `shoulder_l/r`. כל חלק חייב להיכנס ל-slot המוגדר (bbox) ולהשתמש רק בצבעי טוקן (`var(--skin)`, `var(--hair)`, `var(--hair-shadow)`, `var(--cloth-1)`), כך שהצביעה מחדש מתבצעת בזמן ריצה (Rive data binding / Skia).
- **קטגוריות:** פנים (6 צורות), עיניים (10), גבות (8), אף (6), פה (8 + 6 צורות דיבור), אוזניים (3), שיער קדמי (30+: חלק, גלי, מתולתל, אפרו, צמות, קוקו, קצר, "שן חסרה" כחלק פה), שיער אחורי (תואם), משקפיים (6), מכשירי שמיעה (2), כיפה (4), כובעים (10), נמשים (3), תלבושות (20, D3 cosmetic shop), גוון עור 12–16.
- **ייצור:** Recraft vector (custom `style_id` מתמונות G1) עם פרומפט לכל חלק + תמונת ה-template של ה-slot כ-reference; לחלופין עיצוב ידני של חלקים קריטיים (פנים/עיניים) ע"י ההורה-המעצב/ת וייצור הווריאציות ב-AI.
- **עיבוד:** `svgo` (preset safe), החלפת כל fill בטוקן הקרוב (ΔE ≤ 8) או fail, בדיקת bbox מול ה-slot, בדיקת "אין path מחוץ לקנבס", ושמות שכבות `{category}_{variant}_{layer}`.
- **בדיקת עקביות:** הרכבת 200 אווטארים אקראיים + 6 "פורטרטים" לפי תיאור ההורה (בלי תמונות) → contact sheet → G3. כשל עיגון (שיער לא נוגע ב-hairline ±6px, משקפיים לא על eye_line ±4px) → fail אוטומטי.

### 7.6 אייקונים SVG דרך Recraft

- רשימה ב-`assets/icons.yaml` (id, משמעות, האם להפוך ב-RTL: חצים/"אחורה" כן; שעון/מטבע/סימן V לא).
- גריד 96×96, stroke אחיד 8px בגודל 96 (מתאים ל-4px ב-48), פינות מעוגלות, 2 צבעים מטוקנים.
- Recraft: **בדיקת בנייה** (⟂VERIFY: קריאת API אחת עם `style_id` על V3 vector ועל V4; מתעדים איזה מהם מכבד את הסגנון) — איזה מודל מקבל את ה-`style_id` המותאם (V3 vector או משפחת V4 Styles; התיעוד הרשמי אומר ש-V4 לא תומך ב-styles, ומפיצים חיצוניים מציגים "V4 Styles" שכן) → נבחר ונרשם ב-ADR. **מנוי בתשלום חובה**: בתוכנית החינמית התמונות ציבוריות ושייכות ל-Recraft.
- עיבוד: `svgo`, נרמול viewBox, הסרת טקסט, המרה ל-`react-native-svg` components (codegen), snapshot test.

### 7.7 מסירה ל-Rive

**קובץ:** `rive/{char_id}.riv` (מקור `.rev` בחשבון ה-Rive של ההורה). **Artboards:** `main` (גוף מלא), `portrait` (UI), `mini` (מפה). **חלקים:** מיובאים כ-SVG שכבות בשמות מ-§7.4/7.5.
**חוזה State Machine (`SM_main`) — שמות קלט קבועים שהקוד מצפה להם:**

| Input | סוג | ערכים | שימוש |
|---|---|---|---|
| `mood` | number | 0 idle, 1 happy, 2 thinking, 3 surprised, 4 oops, 5 sleepy | מצב בסיס |
| `talk` | boolean | | פה זז כשאודיו מתנגן |
| `viseme` | number | 0–5 (rest/A/E/O/MBP/FV) | אופציונלי, מנוהל ע"י קוד לפי timestamps פשוטים |
| `cheer` | trigger | | חגיגה (≤ 0.8 ש׳ לתשובה, D7) |
| `bigCheer` | trigger | | אבן דרך (4–6 ש׳, ניתן לדילוג) |
| `point` | number | −1 שמאל … 1 ימין | הצבעה על מטרה (RTL מודע) |
| `skin`, `hair`, `cloth1` | color (data binding) | טוקנים | אווטארים |

**גלגלי:** בנוסף `gears` (number 0–1 מהירות סיבוב = "חושב") ו-`lamp` (boolean). **צללונים:** `wobble`, `befriend` (trigger — הופך לחבר; אין מצב "מוות").
**קבלה:** קובץ ≤ 300KB לדמות; 60fps על iPad בסיסי (בדיקת מכשיר ע"י ההורה, E4); כל קלט מתועד ב-`rive/{id}.contract.json` ונבדק ב-unit test מול ה-runtime.

### 7.8 SFX ומוזיקה + יומן רישוי

- **בסיס:** Kenney CC0 (UI clicks, pops) — ללא חובת ייחוס.
- **SFX מותאמים / מוזיקה:** ElevenLabs SFX/Music **רק אם יש אישור כתוב** (E8a; ראו §13). ברירת מחדל ללא אישור: Freesound עם מסנן CC0 בלבד; ספריות בתשלום עם רישיון ברור (לבדוק בזמן בנייה); הקלטה עצמית (פולי ביתי — מטבעות, פופ, רשרוש עלים) ע"י ההורה; Stable Audio — **לא** עד שהרישיון יאומת (סתירה במקורות).
- **אסור (B8):** צפירות, אזעקות, פיצוצים, ירי, נפילות פצצה, צלילי מלחמה, sweep עולה-יורד דמוי צפירה. בדיקה: תגיות אסורות בשם/מטא-דאטה + האזנת G7 חובה ל-100% מה-SFX.
- **מוזיקה:** 3–5 לולאות 60–90 ש׳ (מחנה בערב, בנייה, הרפתקה, שקט/שינה, ג'ינגל כניסה לאי האנגלית — B2) + סטינגרים קצרים. לולאות: חיתוך לגבול תיבה + crossfade 50 ms, −23 LUFS.

**`assets/LICENSES.json` — רשומה לכל נכס:**

```json
{ "assetId": "sfx.crystal_pop.03", "kind": "sfx", "file": "assets/sfx/crystal_pop_03.m4a",
  "source": "kenney.nl/ui-audio", "author": "Kenney", "license": "CC0-1.0", "attributionRequired": false,
  "generator": null, "promptHash": null, "refs": [], "createdAt": "2026-11-20",
  "tosSnapshot": "vendor-terms/kenney-2026-11.pdf", "approvedGate": "G7", "approvedAt": "2026-11-22" }
```
לנכס מחולל: `generator: {vendor, model, version}`, `promptHash`, `refs[]`, **`plan: "paid"` חובה ל-Recraft** (00 E10: בתוכנית החינמית הנכס ציבורי ושייך ל-Recraft; CI מכשיל נכס Recraft בלי `plan:"paid"`), `seed` אם יש. מסך "קרדיטים" באפליקציה נבנה מהקובץ (CC BY / CC BY-SA, Phonikud CC BY 4.0 וכו').

### 7.9 שופט חזותי (`asset.judge`, Sonnet 5.5 vision)

קריטריונים 1–4: `style_match` (מול refs), `no_text_artifacts`, `anatomy_ok` (ידיים/אצבעות/עיניים), `age_safe` (לא מפחיד), `silhouette_readability` (ממוזער ל-64px), `transparency_ok` (מדבקות), `cultural_ok`. hard-fail: `age_safe < 4`, `no_text_artifacts < 3`. ממוצע ≥ 3.2 → ל-contact sheet; אחרת יצירה מחדש עד 3 פעמים, ואז ידני. השופט לא מאשר — רק מסנן לפני השער האנושי.

### 7.10 עלויות נכסים (חודשי ייצור)

GPT Image ~300 תמונות ≈ $10–25 (מחירים מאגרגטורים, לא מאומת); Recraft מנוי בתשלום + ~150 וקטורים ≈ $15–25; Rive — לפי תוכנית החשבון של ההורה; שופט חזותי ≈ $1–2; Kenney/Freesound CC0 — $0. מצב יציב ≈ $0–5.

---

## 8. ציות לתנאי ספקים וחלופות

> מצב התנאים נכון ל-2026-11-02. שורות שמסומנות "לא אומת" נחסמו מהסביבה הזו וחייבות בדיקה בזמן בנייה (research log §Re-verify). קובץ `docs/vendor-terms/` (בשלב הבנייה) שומר צילום של כל מסמך תנאים עם תאריך.

| ספק / רכיב | שימוש | דרישה/מגבלה רלוונטית | איך עומדים בה | חלופה |
|---|---|---|---|---|
| **Anthropic** (Opus/Sonnet/Haiku 5.5) | P1–P4, שופטים | הנחיות לארגונים שמשרתים קטינים: אמצעי הגנה נוספים, **child-safety system prompt אם יסופק**, ציות לחוקי פרטיות + הצהרה פומבית, **גילוי AI**; איסור CSAM/מיניות קטינים; Batch לא ZDR (29 יום) | Safety Core + slot לפרומפט הרשמי; ולידטורים; תור הורה; רובוט מסומן; עמוד "על ה-AI"; כינויים במקום שמות; תמונות רק מהורה מאומת, טשטוש פנים | אין (Claude הוא המודל היחיד שנבחר ב-E8) — במקרה חסימה: תוכן P1 קיים ממשיך לעבוד, P2/P3 עוברים להקלדה/מתוסרט |
| **Dicta Nakdan** | ניקוד | תנאי API ושימוש — **לא אומתו** (אתר חסום; ⟂VERIFY: לפתוח `dicta.org.il` אחרי allow-list, לקרוא תנאי API, ולבדוק שקלט בכתיב מלא חוזר מנוקד בלי שינוי אותיות — 04 PC-04-2) | adapter; לא בשימוש לפני אישור תנאים | **Phonikud** (CC BY 4.0) |
| **Phonikud** | ניקוד, הטעמה, IPA | CC BY 4.0 — ייחוס; מאגרי ILSpeech/SASPEECH לא-מסחריים (לא בשימוש) | ייחוס במסך קרדיטים | DictaBERT char-menaked (לבדוק רישיון) |
| **ivrit.ai Whisper** | QA קולי, קריאה בקול K1 (E9) | Apache-2.0 | — | Apple SpeechTranscriber (אם he זמין) |
| **Mishkefet-v1** (אופציונלי, שלב 2) | OCR כתב יד, מעבר שני | משקלות **CC-BY-NC-SA-4.0** | שימוש פרטי-משפחתי בלבד, לא להפצה | ללא (Claude בלבד) |
| **Azure Speech** | TTS בסיס, P2 בזמן אמת | Code of Conduct + Product Terms AUP; **לא נמצא סעיף קטינים** (לא אומת סופית); הסכמה נדרשת רק לקולות מותאמים/משובטים | קולות מובנים בלבד; לא שולחים קולות ילדים | Cartesia / MiniMax |
| **Cartesia / MiniMax** | מועמדים ב-bake-off | תנאי קטינים **לא נבדקו** | שימוש רק אחרי בדיקת תנאים (פריט מפתח) | Azure |
| **ElevenLabs** (TTS, SFX, Music, Voice Design) | — | אסור לאפליקציות שמכוונות לקטינים ללא **אישור כתוב**; אסור קולות ילדיים; אסור להעלות קול של קטין | **כבוי** עד שההורה מקבל אישור כתוב (`ELEVENLABS_WRITTEN_APPROVAL_REF`) | Azure/Cartesia/MiniMax; Kenney/Freesound CC0 |
| **Google Gemini / GenAI / Vertex** (כולל Nano Banana, Gemini TTS) | — | איסור על שירותים "directed towards or likely to be accessed by" מתחת ל-18 | **לא בשימוש** (E8a) | GPT Image, Recraft; Azure |
| **Google Chirp 3 HD** | — | לא ברור אם נחשב "Generative AI Service" | כבוי עד אישור מפורש | Azure |
| **OpenAI GPT Image** | נכסים אופליין בלבד | Under-18 API guidance (הגנות, COPPA, תוכן מותאם, ביקורות); הימנעות מעיבוד מידע אישי של ילדים מתחת ל-13 ללא ZDR | שימוש ע"י חשבון המפתח, אופליין, **ללא מידע/תמונות של ילדים**, תוכן ילדים-בטוח, שער אישור | Recraft raster; ציור ידני |
| **Recraft** | וקטורים, אייקונים, חלקי אווטאר | תוכנית חינמית: תמונות ציבוריות ושייכות ל-Recraft; בתשלום: בעלות מלאה; אסור לאמן AI אחר על הפלט; סעיף קטינים לא נמצא | **מנוי בתשלום חובה**; לא מאמנים על הפלט | ציור ידני; GPT Image + vectorize |
| **Rive** | אנימציה | רישיון editor לפי תוכנית; runtime MIT | — | Lottie לחד-פעמיים |
| **Kenney** | SFX/UI | CC0 | — | Freesound CC0 |
| **Wikimedia IPA clips** | חלופת פוניקה | CC BY-SA 3.0/4.0 — ייחוס + share-alike לקבצים עצמם | מסך קרדיטים; הקבצים לא משונים מעבר לנרמול (שינוי = שיתוף תחת אותו רישיון) | הקלטה אנושית |
| **Apple** (VisionKit, Vision, AVFoundation) | עיבוד על המכשיר | — | — | — |

**כלל תפעולי:** כל ספק ב-`workers/api/src/ai/vendors.ts` עם `enabled` + `termsCheckedAt` + `termsRef`. CI נכשל אם `termsCheckedAt` ישן מ-180 יום לספק פעיל (תזכורת לבדיקה מחדש, E8a "Re-verify terms at build time").

**ספק נופל ותקרת $ — חלוקת בעלות:** ה**ערכים** (תקרות $3/$40, מצבים, override) שייכים למסמך הזה (§1.9, 00 J1) ו-11 §8.4 מאמץ אותם (`ai_daily_cap_cents = 300`); **המנגנון** (timeout 20 ש׳ לתור סוקרטי, 529 → שורה מתוסרטת, `ai_log.flagged='vendor_down'`, kill switch ב-config, התראת 80%) מוגדר ב-11 §4.11/§8.4 ולא משוכפל כאן. §0.5 ו-§3.15 מתארים רק את מה שהילד וההורה רואים.

---

## 9. יעדי ביצועים ועלויות (ריכוז)

| צינור | יעד latency | עלות חודשית — ייצור | יציב |
|---|---|---|---|
| P1 תוכן | מוכן לאישור עד 07:00; במכשיר ≤ 10 דק׳ מאישור | ≈ $20 | $5–8 |
| P2 צלם-ושחק | חילוץ ≤ 25 ש׳/דף (P95); **אחרי אישור: P50 ≈ 14 ש׳, P95 ≤ 60 ש׳ (קשיח)**; סיבוב ראשון ≤ 15 ש׳ | ≈ $2–3 (30 תמונות) | כנ"ל |
| P3 סוקרטי | P95 ≤ 4 ש׳, timeout 6 ש׳ | < $0.5 | < $0.5 |
| P4 הורה | ראשון 07:30 | < $0.1 | < $0.1 |
| P5 קול | אופליין עד בוקר; P2 ≤ 1.5 ש׳/שורה | $0–4 (Azure) | ≈ $0 |
| P6 נכסים | לפי שערים | $25–50 | $0–5 |
| **סה"כ** | | **≈ $50–80** | **≈ $8–20** |

תקרות: יומית $3, חודשית $40 ל-LLM (§1.9); נכסים ותמלול מחוץ לתקרה האוטומטית (הפעלה ידנית).

---

## 10. קריטריוני קבלה

**משותף**
- [ ] כל קריאת Claude עוברת דרך `callStructured` עם `output_config.format`; אין `tool_choice` כפוי, אין prefill, אין `temperature` (בדיקת lint על הקוד).
- [ ] סכמות פלט עומדות במגבלות: 0 שדות אופציונליים, ≤ 16 unions, אין `minimum/maxLength` (בדיקת CI על כל סכמה).
- [ ] אין שמות אמיתיים של ילדים בשום payload לספק (בדיקת יחידה + סריקת לוגים בדגימה).
- [ ] ולידטורים V01–V18 עם בדיקות יחידה, כולל מקרי קצה בעברית (קידומות, סופיות, ניקוד, סימני Phonikud).
- [ ] סט כיול השופט (40 פריטים): ≥ 85% זיהוי פגמים, ≤ 10% דחיות-שווא.
- [ ] תקרת $ נבדקת בבדיקת אינטגרציה (מעל 80% → Batch נדחה; מעל 100% → P2 נחסם בעדינות; override נספר; איפוס בחצות; kill switch) לפי מכונת המצבים ב-§1.9.
- [ ] **תוכן לילד לעולם לא "חי" (§0.2.9):** בדיקת רשת על מכשיר `shared-kids`: K2/K3 — 0 קריאות ל-`/v1/ai/*`; K1 — רק `/v1/ai/socratic` ורק מצומת `llm_turn`; כל מה שמוצג מגיע מחבילה עם sha256 תקין (ניסיון להזריק חבילה בלי חתימה → נדחית).
- [ ] **V19:** 0 מופעים של שם/כינוי/גן/מורה מ-`private/family-names.json` ב-200 payloads מוקלטים (Batch + realtime), כולל P2 ו-P4; בדיקת CI על כל תבנית פרומפט.
- [ ] מטריצת §0.5: לכל תא יש בדיקה (יחידה/E2E) או סימון "לא ישים" מנומק.

**P1**
- [ ] 100% מהפריטים המתמטיים: תשובה מחושבת בקוד = התשובה בפריט; 0 מספרים "מומצאים" בטקסט.
- [ ] מגדר: 0 שגיאות בדגימת הורה של 50 פריטים × 3 גרסאות.
- [ ] ניקוד: ≤ 1 שגיאת ניקוד ל-100 מילים בתוכן K2 (בדיקת הורה על 300 מילים); אפס סימני Phonikud בתצוגה.
- [ ] K2 decodability: 100% מהמילים בסיפורי K2 ⊆ מילון הפענוח של הילד.
- [ ] ≥ 40 וריאנטים חיים לכל צומת כישור פעיל.

**P2**
- [ ] סט הערכה של 30 דפים אמיתיים של המשפחה (20 מודפסים, 10 כתב יד): מודפס — ≥ 95% מהמילים נכונים לפני תיקון; כתב יד — כל מילה לא ודאית מסומנת (0 שגיאות "שקטות" ב-`confidence:high`).
- [ ] P95 "זמן עד משחק" אחרי אישור ≤ 60 ש׳ על 20 ריצות; P50 ≤ 20 ש׳.
- [ ] בדיקת פרטיות: EXIF/GPS לא קיים בקובץ ב-R2; פנים מטושטשות ב-100% מ-10 תמונות מבחן עם פנים; שמות מרשימת "לא-משפחה" לא מופיעים במשחק.
- [ ] מצב "שיעורי בית להגשה": אף פריט מקורי ואף תשובה מקורית לא מוצגים לילד לפני "הוגש"/06:00 (בדיקת E2E).
- [ ] ≥ 15 תבניות פעילות עם `meetsMinimum` ובדיקות snapshot לכל גיל רלוונטי.
- [ ] סיפור ילד: הטקסט המקורי נשמר מילה במילה; אין סימון שגיאות בצד הילד.
- [ ] **קריאה כפולה לכתב יד (00 E8):** `photo.dualRoute` כבוי כברירת מחדל; נדלק רק עם ADR שמצטט את תוצאת 10 דפי כתב היד (רווח ≥ +0.10 כיסוי מילים, 0 שגיאות שקטות חדשות, ≤ $0.12 לדף).
- [ ] **מקרי קצה §3.15 #1–#21:** סט `tools/eval/p2-edge/` (50 דפים, §3.17) עובר ב-100% על fixtures; ריצה חיה שבועית ≥ 90%.
- [ ] **דף מבחן (`exam_page`):** 0 פריטים מקוריים ו-0 תשובות בחבילה ובמסך ההורה (V20, בדיקת E2E על 3 דפי מבחן מהסט).
- [ ] **תור לכידה (§3.16):** 3 דפים במצב טיסה → חיבור → 3 jobs ללא כפילויות; קריסה באמצע שליחה → 0 כפילויות.
- [ ] **10 דפים:** פיצול ל-3 jobs, הערכת עלות מוצגת, והתקרה (§1.9) מכבדת את סדר ההגשה.

**P3**
- [ ] 0 דליפות תשובה ב-500 תורות סינתטיות אדברסריות ("תגיד לי את התשובה", "מה זה 7 כפול 8?").
- [ ] 100% מקלטי מצוקה/פרטים אישיים בסט מבחן (60 משפטים) נחסמים לפני LLM ומפעילים את השורה המתוסרטת + דגל.
- [ ] K2/K3: אין שום קריאת LLM בזמן ריצה (בדיקת רשת).
- [ ] הרובוט מוצג תמיד עם סימון מכונה; שורת הפתיחה מושמעת בפעם הראשונה בכל יום.

**P4**
- [ ] 0 השוואות בין ילדים, 0 מונחים אבחוניים, 100% מהמספרים תואמים ל-WeeklyFacts (על 12 דוחות מבחן).

**P5**
- [ ] bake-off הורץ ותועד (`docs/decisions/ADR-voice-vendor.md`), כולל צפיפות ניקוד ו-gain לכל קול.
- [ ] כל קליפ ב-manifest: −18 ±1 LUFS (או gain מכויל לקצרים), ≤ −1 dBTP, CER ≤ 0.05 או אושר ע"י הורה.
- [ ] **בנק A (אנגלית, §6.10.1):** 45 פונמות × 2 סגנונות + 26 שמות אותיות + 45 מילות מפתח + ~60 מומצאות + שירים ב-`en-hrec.json`, כל פריט עם take נבחר ו-QC (פוצצים ≤ 150 ms בלי שווא; −18 LUFS ±1; רישיון רשום) — או חלופת Wikimedia עם ייחוס לפונמות בלבד; אף צליל בודד לא מ-TTS (בדיקת manifest `engine:"human"`).
- [ ] **בנק B (עברית, §6.10.2):** ~300 קטעים ב-`he-hrec.json`, 100% `parent_ok`; 0 עיצורי סגר עם שווא נשמע; `first_sound`/`nikud_sort` משמיעים HREC בלבד; צירופי CV זמניים מ-TTS מסומנים `temporary:true` ונעלמים כשההקלטה מגיעה.
- [ ] **בנק C (הורה, §6.11):** `name_clip` לכל ילד ב-3 אינטונציות; בלי הקלטה — אף שורה לא "ממציאה" שם (בדיקת snapshot על ניסוח ללא שם).
- [ ] שם ילד לא נשלח לעולם ל-TTS.

**P6**
- [ ] G1–G7 עברו ונרשמו; `LICENSES.json` מלא לכל נכס בחבילה (בדיקת CI: נכס בלי רשומה → build נכשל).
- [ ] אין נכס מ-Gemini/ElevenLabs ללא אישור (בדיקת CI על שדה `generator.vendor`).
- [ ] 0 SFX עם תגיות אסורות (B8); G7 האזנה 100%.

---

## 11. פריטים פתוחים להורה (מינימום, באצווה אחת — F2)

> המקור המחייב והמאוחד הוא טבלת **Open parent decisions (OPD-01…34) ב-00**; הרשימה המסודרת לפי מועד עם פורמט וזמן היא **13 §16.2**. כאן רק מה שהמסמך הזה הוא הבעלים שלו, עם מזהה ה-OPD. כשברירת המחדל כאן שונה מ-00 — 00 גובר.

**תוכנית הקלטות מאוחדת (00 E9; שלושה מפגשים, פירוט ב-§6.10–6.11):**

| מפגש | OPD | מי | היקף | משך | עד מתי | ברירת מחדל בלי זה |
|---|---|---|---|---|---|---|
| C. קול ההורה | **OPD-12** | כל הורה, באפליקציה | שמות ×3 ×3 אינטונציות, 15 שבח, לילה/בוקר טוב, חגים, ספירה, צבעים (PV-01..20) | **~20 דק׳** | C3 | אין שם בקול; TTS לעולם לא אומר שם |
| B. בנק צלילים עברי | **OPD-13** | ההורה או קריין/ית ילידי/ת, באפליקציה או במיקרופון | ~300 קטעים (23 עיצורים, 5 תנועות, ~120 CV × 2 קצבים, ~60 הברות סגורות, א/ע) | **~45–60 דק׳** (ניתן לפיצול ל-3 × 15) | C5 | CV זמני מ-TTS (`temporary`); עיצורים מבודדים לא זמינים → צמתי `H.PA.PH`/`H.LK.SND` נעולים |
| A. אנגלית | **OPD-15** | דובר/ת en-US ילידי/ת (מכר או בתשלום ~$100–250, ⟂VERIFY) | 45 פונמות × סגנונות, 26 שמות אותיות, 45 מילות מפתח, זוגות מינימליים, ~60 מומצאות, 27 שירים + 45 פזמוני PHON | **~2.5 ש׳** (אפשר 2 מפגשים: צלילים 75 דק׳ / שירים 75 דק׳) | C5 | פונמות = Wikimedia CC BY-SA; שירים מדוברים לא מושרים; מומצאות לא נתמכות (NWF נעול) |

1. **חשבונות ומפתחות** (**OPD-08**): Anthropic API (+ תקרת הוצאה **$50** בקונסול — רשת ביטחון מעל התקרה הרכה של §1.9; 00 OPD-08 אומר ~$30 — הערך כאן הוא ההצעה של בעל הנושא, ההורה בוחר), Azure Speech (משאב באזור West Europe; Israel Central אם זמין — ⟂VERIFY זמינות Neural TTS באזור), OpenAI (GPT Image, לנכסים בלבד), Recraft **מנוי בתשלום**, Rive. אופציונלי: Cartesia/MiniMax לניסיון ב-bake-off.
2. **ElevenLabs — להחליט אם לבקש אישור כתוב** (**OPD-16**). אם כן, טיוטת פנייה (לשלוח מהחשבון שלך):
   > "Hi ElevenLabs team — I'm a parent building a private, non-distributed learning app used only by my own three children (ages 2.5–8.5) at home. All audio would be pre-generated offline by me (an adult account holder) using adult stock or designed adult voices; no child voices, no uploads of children's voices, no child users interacting with your API. I'd like written approval under your Prohibited Use Policy for this use (TTS in Hebrew/English, and SFX/Music). Happy to share details."
   בלי אישור — הכל עובד בלי ElevenLabs.
3. **מפגש האזנה ל-bake-off** (**OPD-17**; 60–90 דק׳ להורה אחד, 2–3 שעות אם שני ההורים; K1 ו-K2 ל-10 דקות A/B). ברירת מחדל (00): הסוכן בוחר לפי המדדים האובייקטיביים ומסמן "לא נשמע ע"י ההורה".
4. **הקלטות קול ההורה** (**OPD-12**; ~20 דקות, §6.11).
5. **בנק הצלילים העברי** (**OPD-13**; ~45–60 דקות, §6.10.2).
6. **פוניקה אנגלית** (**OPD-15**; ~2.5 שעות, §6.10.1) **או** אישור חלופת Wikimedia לפונמות.
7. **30 צילומי דפים אמיתיים** (**OPD-20**: 20 מודפסים, 10 כתב יד, עם תשובות מחוקות או לא — כמו שהם) לסט ההערכה של P2 ולהחלטת הקריאה הכפולה (§3.5.1). נשמרים מקומית לבדיקה בלבד.
8. **שערי טעם G1–G7 (+G4b)** (**OPD-11**; לפי לוח הזמנים של הבנייה; ממופים ל-GATE-UX-1..9 של 09).
9. **Dicta Nakdan** (**OPD-31**): אם האתר דורש הרשמה/פנייה לשימוש ב-API — לבצע (אחרת Phonikud בלבד); באותה אצווה: בדיקה מדגמית של 20 שורות מספרים + 20 מילים מנוקדות.
10. (מ-00) מגדר דקדוקי של K3 + שמות הילדים (**OPD-01, OPD-02**); **רשימת שמות "לא-משפחה"** (חברי גן/כיתה, מורות, שם הגן/בית הספר) ל-`private/family-names.json` — משמשת לטשטוש בתמונות (V18) ולחסימת payloads (V19). אופציונלי אבל מומלץ; **לא ב-OPD** — מוצע כ-OPD-35 (ראו `fable-issues-F7.md`); עד אז ברירת מחדל: רשימה ריקה + זיהוי PII גנרי בלבד.
11. **תקרות AI**: לאשר $3/יום ו-$40/חודש (§1.9) — חלק מ-**OPD-08** (תקרת הקונסול); ברירת מחדל: הערכים כאן.

---

## 12. נספחים

### 12.1 טבלת מספרים (קטע; הטבלה המלאה 0–100 + מאות/אלפים ב-`packages/content/hebrew/numerals.json`, נבדקת ידנית)

| n | ספירה / נקבה עצמאי | נקבה לפני שם עצם | נקבה נסמך (מיודע) | זכר עצמאי | זכר לפני שם עצם | זכר נסמך |
|---|---|---|---|---|---|---|
| 1 | אַחַת | (אחרי שם העצם) …אַחַת | — | אֶחָד | …אֶחָד | — |
| 2 | שְׁתַּיִם | שְׁתֵּי | שְׁתֵּי | שְׁנַיִם | שְׁנֵי | שְׁנֵי |
| 3 | שָׁלוֹשׁ | שָׁלוֹשׁ | שְׁלוֹשׁ | שְׁלוֹשָׁה | שְׁלוֹשָׁה | שְׁלוֹשֶׁת |
| 4 | אַרְבַּע | אַרְבַּע | אַרְבַּע | אַרְבָּעָה | אַרְבָּעָה | אַרְבַּעַת |
| 5 | חָמֵשׁ | חָמֵשׁ | חֲמֵשׁ | חֲמִשָּׁה | חֲמִשָּׁה | חֲמֵשֶׁת |
| 6 | שֵׁשׁ | שֵׁשׁ | שֵׁשׁ | שִׁשָּׁה | שִׁשָּׁה | שֵׁשֶׁת |
| 7 | שֶׁבַע | שֶׁבַע | שְׁבַע | שִׁבְעָה | שִׁבְעָה | שִׁבְעַת |
| 8 | שְׁמוֹנֶה | שְׁמוֹנֶה | שְׁמוֹנֶה | שְׁמוֹנָה | שְׁמוֹנָה | שְׁמוֹנַת |
| 9 | תֵּשַׁע | תֵּשַׁע | תְּשַׁע | תִּשְׁעָה | תִּשְׁעָה | תִּשְׁעַת |
| 10 | עֶשֶׂר | עֶשֶׂר | עֶשֶׂר | עֲשָׂרָה | עֲשָׂרָה | עֲשֶׂרֶת |
| 11 | אַחַת־עֶשְׂרֵה | אַחַת־עֶשְׂרֵה | | אַחַד־עָשָׂר | אַחַד־עָשָׂר | |
| 12 | שְׁתֵּים־עֶשְׂרֵה | שְׁתֵּים־עֶשְׂרֵה | | שְׁנֵים־עָשָׂר | שְׁנֵים־עָשָׂר | |
| 13 | שְׁלוֹשׁ־עֶשְׂרֵה | | | שְׁלוֹשָׁה־עָשָׂר | | |
| 20 | עֶשְׂרִים | עֶשְׂרִים | | עֶשְׂרִים | עֶשְׂרִים | |

הטבלה חייבת לעבור **בדיקה ידנית של דובר/ת עברית** לפני שימוש (קבלה; OPD-31); צורות הנסמך מסומנות בה כ-`verified:false` עד אז. ספירה רוטינית (B4): עמודת "ספירה/נקבה עצמאי". **זהו קטע מתוך `numerals.json` — הקובץ הוא המקור היחיד גם ל-03 §10.4** (כללי ההרכבה: עשרות, מאות, אלפים, ו' החיבור, סודרים, שברים, שעון — ב-§1.5.3). שינוי בטבלה = PR `shared:` + הרצת `MATH-AC-25`.

### 12.2 קטלוג טעויות — פירוט נבחר

| קוד | כלל יצירה | דוגמה | הגבלות |
|---|---|---|---|
| `MATH_OP_SWAP` | a op' b כאשר op' הפוך | 9−4 → 13 | רק אם התוצאה חיובית ובטווח |
| `MATH_OFF_BY_ONE` | ans ± 1 | 7+5 → 11/13 | |
| `MATH_DIGIT_REVERSAL` | היפוך ספרות | 21 → 12 | רק לדו-ספרתי עם ספרות שונות |
| `MATH_PLACE_VALUE` | חיבור יחידות לעשרות | 25+7 → 95 | K1 |
| `MATH_CARRY_OMIT` | בלי העברה | 38+25 → 53 | רק כשיש העברה |
| `MATH_BORROW_OMIT` | "קטן מגדול" בכל עמודה | 52−27 → 35 | רק כשיש פריטה |
| `MATH_TABLE_NEIGHBOR` | a×(b±1) | 7×8 → 49/63 | |
| `MATH_ADD_FOR_MULT` | a+b | 7×8 → 15 | |
| `HE_SOUNDALIKE_T` | ט↔ת | תפוח → טפוח | לא מילה אמיתית |
| `HE_SOUNDALIKE_K` | כ↔ק, כ↔ח (רפה) | כלב → קלב | לא מילה אמיתית |
| `HE_FINAL_FORM` | סופית↔רגילה | שלום → שלומ | |
| `HE_MATER_OMIT` | השמטת ו/י | שולחן → שלחן | שים לב: כתיב חסר תקני — מותר רק אם המילה מופיעה בכתיב מלא בחומר הכיתה |
| `HE_VISUAL` | אות דומה חזותית | דג → רג | K2 קריאה |
| `EN_SHORT_VOWEL_SWAP` | החלפת תנועה קצרה | bat → bet | אם המילה המוסחת קיימת — עדיף (זוג מינימלי) בתרגיל שמיעה; אסור בתרגיל כתיב |

הטבלה היא **פירוט נבחר של כינויים**; הקטלוגים המלאים, כללי היצירה (`ERROR_FN`) והתיקון (`R.*`) חיים ב-03 §4.2 ו-04 §2.3, ו-`aliases.json` ממפה כל שורה כאן לקוד `E.*` (§3.7).

### 12.3 רשימות חסומות (התחלה; מורחבות ע"י ההורה)

- `security` (B8): אזעקה, צפירה, טיל, רקטה, מקלט, ממ"ד, פיגוע, מחבל, חטוף/חטופים, מלחמה, פצצה, יירוט, כיפת ברזל, צבע אדום, הרוג/הרוגים, פצוע/פצועים, חייל בקרב, רובה, נשק, מוות, ירי.
- `violence/fear`: דם, להרוג, לרצוח, שד, רוח רפאים מפחידה, מפלצת אוכלת, סכין (חוץ מבישול עם מבוגר — אסור בכל זאת לתוכן ילדים), חטיפה, ללכת לאיבוד לבד בלילה.
- `praise_trait` (C4): גאון, גאונה, אלוף, אלופה, חכם, חכמה, מוכשר, מושלם, הכי טוב, מדהים, תותח, מלך, מלכה.
- `comparison`: "יותר טוב מ", "לעומת אחותך/אחיך", "כמו אחיך", דירוג, מקום ראשון.
- `body_food`: שמן/שמנה (על אדם), דיאטה, "לא לאכול כי משמין", מכוער.
- `personal_info_requests`: "מה הכתובת", "באיזה בית ספר", "מה הטלפון", "תשלח תמונה".
- `distress` (P3 קלט): עצוב/ה מאוד, מפחד/ת, מרביצים לי, כואב לי, אף אחד לא אוהב אותי, סוד, אל תספר, רוצה למות, פוגעים בי.

### 12.4 Prompt versions
כל פרומפט בקובץ `packages/content/prompts/{job}.v{n}.md` עם front-matter: `job, version, schemaVersion, model, effort, changelog`. שינוי ⇒ גרסה חדשה ⇒ כיול שופט מחדש (P1) / סט אדברסרי (P3).

---

## 13. Proposed change to 00

> לפי כללי 00: ההחלטות נשארות בתוקף עד שינוי מפורש; המסמך הזה פועל לפי ההצעות רק היכן שהן מצמצמות סיכון בלי לסתור את 00 (סעיפים 1, 2 — כי E8a גובר ממילא; 3 — כניתוב נוסף בתוך "Sonnet 5.5 … with mandatory parent confirm").

1. **E10 — להסיר "Nano Banana" מרשימת כלי התמונה.** Nano Banana הוא מודל Gemini; E8a קובע "do not use Gemini APIs", כך ש-E10 סותר את E8a. **הצעה:** "GPT Image (אופליין, חשבון מפתח, בלי מידע על ילדים) לסצנות ודמויות; Recraft (מנוי בתשלום) לווקטורים/אייקונים/חלקי אווטאר; ציור ידני כחלופה." ראיה: Gemini API Additional Terms (בפתקי voice_tech_he_en §2, נבדק מאז Feb 2024).
2. **E10 — "ElevenLabs SFX/Music" להתנות באישור כתוב.** E8a מתיר ElevenLabs רק עם אישור כתוב, אבל E10 מונה אותו ללא תנאי. **הצעה:** "Kenney CC0 + Freesound (CC0 בלבד) + הקלטות פולי ביתיות; ElevenLabs SFX/Music רק אם התקבל אישור כתוב." ראיה: ElevenLabs Prohibited Use Policy (פתקים קודמים) + research log §9.
3. **E8 — ניתוב כתב יד ל-Opus 5.5 במקביל ל-Sonnet 5.5.** בלוח heb-ocr (ספט׳ 2026) claude-opus-5 הגיע לכיסוי מילים של 0.335 בעמוד כתב יד מלא מול 0.210 של claude-sonnet-5; Claude בכלל נמוך (Mishkefet-v1 0.463). **הצעה:** "דפי כתב יד: Sonnet 5.5 + Opus 5.5 במקביל עם פיוס בקוד ומסך אישור הורה חובה; שלב 2 אופציונלי: מעבר שני Mishkefet-v1 (משקלות CC-BY-NC-SA — מותר לשימוש פרטי) בקונטיינר." עלות נוספת ≈ $0.03–0.06 לדף. **חובה לאמת על 30 הדפים של המשפחה** (§10) — אם Opus 5.5 לא משפר על הסט, מבטלים את הקריאה הכפולה.
4. **E8 — הבהרה: Batch API אינו ZDR (שמירה 29 יום).** **הצעה להוסיף:** "אין שמות אמיתיים, תמונות או טקסט חופשי של ילדים בבקשות Batch; כינויים K1/K2/K3 בלבד." ראיה: Anthropic batch-processing docs (research log §2).
5. **E9 — הבהרה לגבי P2:** "All narration pre-generated & cached" — **הצעה להוסיף:** "ב'צלם וקבל משחק', אודיו מרונדר בשרת אחרי אישור ההורה ולפני שהמשחק נפתח, נשמר ב-R2 ומנורמל מחדש בלילה; אין TTS חי ממכשיר הילד, ואין AVSpeechSynthesizer לקריינות לילדים." (תואם את רוח E9; מונע פרשנות שסוגרת את P2 בלי קול.)
6. **E8 — "Anthropic child-safety system prompt" — הבהרה:** נכון ל-2026-11-02 Anthropic לא פרסמה טקסט כזה ("we may provide"). **הצעה:** "להשתמש בו אם וכאשר יסופק (slot במערכת), ועד אז Safety Core פנימי + ולידטורים + תור הורה." ראיה: support.claude.com 9307344, 15591275.

---

## Fable review changelog

סוקר F7, 2026-11-02. שינויים במקום (ללא שינוי מבנה או שפה):

- **§0.2 עקרונות 7, 9, 10 (חדש/מורחב):** "תוויות בלבד" לספקים הפך לכלל אכיף (V19 `payloadPiiLint` + רשימת `private/family-names.json`); נוסף עיקרון "שום תוכן לילד אינו חי" עם בדיקת רשת לפי מצב מכשיר; נוסף עיקרון "אופליין הוא המצב הרגיל". סיבה: הבריף (#12) ו-C7 דרשו כלל מדיד, לא כוונה.
- **§0.5 (חדש):** מטריצת מקרי קצה לכל צינור (אופליין ילד/הורה, ספק נופל, תקרת $, שבת, K3, החלפת פרופיל, סנכרון כפול, "מוזר לי", תסכול, נגישות) — כלל אחד לכל תא.
- **§1.6:** נוספו V19 (PII לפני שליחה) ו-V20 (`answerKeyGuard` לשיעורי בית/מבחן).
- **§1.9:** התקרה הפכה למכונת מצבים מפורשת (80%/100%/חודשי/kill switch) עם התנהגות לכל צינור, override מוגבל, תור `deferred_jobs`, אומדן עליון לפני קריאה, ותקרה קשיחה $50 בקונסול. סיבה: "AI cost cap behavior" היה משפט אחד.
- **§2.7.1 (חדש):** טיפול בכשלי QA בהיקף — שופט דוחה/מאשר-הכול, סחף כיול, דחיות הורה, דגלי ילד, S0 עם `revoked` במניפסט, ניקוד, אודיו, מלאי — כל אחד עם סף, פעולה ומי מקבל הודעה.
- **§3.2:** טיפים 6–9 (יותר מ-4 דפים, דף מבחן, דף מעורב, אופליין).
- **§3.10:** מצב `exam_page` (אין פריטים מקוריים ואין מפתח תשובות לאיש) + כלל "לא לפתור שיעורי בית" ואישור כפול לשינוי מצב.
- **§3.15 (הוחלף):** 21 מקרי קצה עם זיהוי, התנהגות ומה ההורה רואה: טשטוש/חושך (ספי Laplacian/בהירות), דף מעורב he/en (שני משחקים, אי אנגלי), כתב יד לא קריא, שם של ילד אחר/מורה, דף מבחן, תמונות בלבד, 10 דפים (פיצול + הערכת עלות), דף כפול (pHash), סיבוב, אופליין, ספק נופל (retry ברקע שעה), TTS, תקרה, מינימום, נטישת מסך אישור (24 ש׳), ילד משחק בזמן בנייה, תשובות ילד, פנים, כמה מקצועות, iPhone קטן.
- **§3.16 (חדש):** תור לכידה אופליין (מגבלות, אידמפוטנטיות, פקיעה, קבלה). **§3.17 (חדש):** סט בדיקות קצה קבוע (50 דפים, fixtures + ריצה חיה שבועית).
- **§3.5.1:** ⟂VERIFY לקריאה הכפולה Sonnet+Opus על דפי המשפחה.
- **§5.1, §5.5:** עיתוי הדוח יושר ל-10 §11.1 (שישי 12:00–14:00, ≥ 3 ש׳ לפני נרות; Batch 06:05 → Haiku realtime ב-11:00 → קוד בלבד). סיבה: בקשה מפורשת של ההורה (issue #8).
- **§6.1, §6.10 (הוחלף והורחב), §6.10.1, §6.10.2 (חדש), §6.11, §6.12:** שלושת בנקי ההקלטה האנושיים הוגדרו כעבודות הפקה: A אנגלית (~2.5 ש׳: 45 פונמות × סגנונות, 26 שמות אותיות, 45 מילות מפתח, זוגות מינימליים, ~60 מומצאות, 27 שירים + 45 פזמוני PHON; שמות קבצים מ-05 §12.3; QC אוטומטי + 100% האזנה לפונמות/מומצאות; רישוי), B עברית (~300 קטעים לפי 04 §4.4: 23 עיצורים, 5 תנועות, ~120 CV × 2 קצבים, ~60 הברות סגורות, א/ע; ~45 דק׳; שמות קבצים; QC; חלופה זמנית רק ל-CV), C קול ההורה (~20 דק׳, PV-01..20, קבצים ומניפסט). ההקלטה העברית של "22 צלילים" שהייתה כאן הוחלפה בבנק המלא כי 04 הוא בעל הנושא (issue #13).
- **§6.12, §7.6, §8:** סימוני ⟂VERIFY (Azure free tier, Recraft style_id, תנאי Nakdan).
- **§10:** קריטריוני קבלה חדשים: "לא חי" (בדיקת רשת), V19, מטריצת §0.5, מקרי קצה P2, `exam_page`, תור לכידה, 10 דפים, שלושת הבנקים (A/B/C).
- **§11:** פריטי ההורה מופו למזהי Q-xx של 13 §16.2 (המקור המאוחד); נוספו: בנק עברי, רשימת שמות "לא-משפחה", אישור תקרות, Azure region.
- לא שונה: E8a כבר מיושם כאן (Gemini/Nano Banana רק בטבלת "לא בשימוש" ובהצעה להסיר מ-E10; ElevenLabs מאחורי `ELEVENLABS_WRITTEN_APPROVAL_REF`). פרומפטי Batch כבר עם תוויות בלבד; עכשיו גם נאכף.
- סוגיות במסמכים אחרים (11 §8.3/§8.4 תקרה ו-resize, 11 §7.1 cron, 13a id-map) נרשמו ב-`docs/review/fable-issues-F7.md`.
- **יישור ל-00 רוויזיה 2 (F1) ול-`fable-issues-F1.md` #17, #18, #21, #26:** §1.1 + §3.5.1 — הקריאה הכפולה לכתב יד היא מותנית (ברירת מחדל Sonnet בלבד; סף הפעלה: +0.10 כיסוי מילים על 10 דפי המשפחה, 0 שגיאות שקטות, ≤ $0.12/דף, ADR); §7.1 — שער **G4b** למודל 3D מרוגג של הגיבור (00 E10) ו-`plan:"paid"` חובה ל-Recraft ב-`LICENSES.json` עם בדיקת CI; §11 — טבלת תוכנית הקלטות מאוחדת (OPD-12/13/15) וכל פריט הורה ממופה ל-OPD-nn (ברירות המחדל של 00 גוברות); §10 — קריטריון לקריאה הכפולה.
