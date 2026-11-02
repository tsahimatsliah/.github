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
7. **מינימום מידע אישי לספקים:** בפרומפטים משתמשים בכינויים (`K1`, `K2`, `K3`) ובשמות דמויות במשחק; שם הילד האמיתי מוזרק רק בצד הלקוח בזמן הצגה (ובקול — רק מהקלטת ההורה). Batch API אינו ZDR (שמירה 29 יום), ולכן זה חובה.
8. **תוכן שילד רואה עובר לפחות:** ולידטורים בקוד → שופט LLM → (אישור הורה או דגימה) → פרסום בחבילה חתומה ב-hash (E6).

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
| `photo.extract.hw` | חילוץ כתב יד (מקבילי) | `claude-opus-5-5` (+ Sonnet במקביל) | Realtime | `medium` | 8000 | system 5m | הורה מקליד |
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

- בכל טקסט שנוצר ע"י LLM מספרים מופיעים כ-slots: `{{num:a|noun:תפוח}}`. הקוד שולף מגדר שם-עצם מ-`nounLexicon` (שדה `gender`, נבדק ידנית), ממלא צורה נכונה, ומכניס ספרות לתצוגה מתמטית ומילים ל-TTS.
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

**תקרה (E8):** `DAILY_USD_CAP` (ברירת מחדל $3) ו-`MONTHLY_USD_CAP` ($40). מעל 80% יומי: עבודות Batch נדחות ליום הבא; P2 ו-P3 ממשיכים עד 100%; מעל 100%: P2 מציג "המשחק יוכן מחר בבוקר" (ההורה יכול לעקוף ב-Face ID עד $5 נוספים), P3 עובר לשורות מתוסרטות בלבד.

---

## 2. P1 — יצירת תוכן לתוכנית הלימודים

### 2.1 ארכיטקטורה

```
 [Skill graph + "מה למדו החודש" (parent input) + coverage gaps from engine]
                     │  nightly Cron 01:30 IL (Worker)
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
4. השתמש/י רק בשמות עצם מרשימת NOUNS שבהודעה, כדי שהמערכת תדע את המין הדקדוקי שלהם.
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

**עיבוד:** V08 decodability (K2 — כל מילה ⊆ ALLOWED_WORDS ∪ FUNCTION_WORDS אחרי הסרת קידומות; כשל אחד = הסיפור חוזר ל-revise עם רשימת המילים הבעייתיות), V09, V10; `scene` ממופה ל-`imageRef` מתוך ספריית רקעים ומדבקות (P6) — **אין יצירת תמונה בזמן ריצה**.

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

### 2.8 עלות וזמן

- **נפח צפוי (חודש ייצור):** ~3,000 פריטים + 60 סיפורים + 40 סולמות רמזים. Skins: 300 בקשות × (3K טוקני system ב-cache-hit + 0.5K קלט + 2.5K פלט) ≈ 0.15M קלט + 0.9M cache-read + 0.75M פלט → ≈ $0.3 + $0.2 + $7.5 = **≈ $8**. סיפורים: 60 × 6K פלט ≈ $3.6. שופט: 3,000 × (2K cache + 1K קלט + 0.4K פלט) ≈ $3 + $6 = **≈ $9**. סה"כ P1 ≈ **$20/חודש ייצור, $5–8 במצב יציב** (תואם ai_models_assets).
- **Latency:** אין יעד מקוון. יעד תפעולי: חבילה ממתינה לאישור עד 07:00 בבוקר; זמינה במכשיר תוך ≤ 10 דקות מאישור.

