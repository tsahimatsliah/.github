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
 CHILD DEVICE: 13 game appears in child's island ("משחק חדש מהמחברת!") · items → SRS queue (+1/+3/+7 …, D9)
                                                                               nightly: loudness re-normalize ──► (runner)
```

### 3.2 טיפים לצילום (טקסט UX להורה, מוצג בפעם הראשונה ובכפתור "?")

> **איך מצלמים כדי שיצא משחק טוב**
> 1. דף אחד בכל צילום, ישר מלמעלה, באור יום או מנורה — בלי צל של הטלפון.
> 2. דף מודפס עובד הכי טוב. כתב יד של ילדים — אפשר, אבל תצטרכו לבדוק את המילים.
> 3. אפשר לחתוך רק את התרגיל. כדאי להשאיר בחוץ שמות של ילדים אחרים, המורה ותמונות.
> 4. אם בדף יש תשובות שהילד כבר כתב — זה בסדר, נשתמש בהן רק בשבילכם.
> 5. ניקוד בתמונה לא נקרא — אנחנו מוסיפים ניקוד מחדש ואתם מאשרים.
> 6. עד 4 דפים לאותו משחק.

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
- `handwritten=true` או שהמודל דיווח `handwriting_ratio ≥ 0.3`: **שתי קריאות מקבילות** (Sonnet 5.5 + Opus 5.5) על אותו דף; פיוס בקוד (§3.5.5). (בלוח heb-ocr, Opus 5 הוביל את Sonnet 5 בכיסוי מילים בעמוד כתב יד מלא: 0.335 מול 0.210; ראו §13.)
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

| תחום | קודים (פירוט ב-§12.2) |
|---|---|
| חשבון | `MATH_OP_SWAP` (חיבור↔חיסור), `MATH_OFF_BY_ONE`, `MATH_DIGIT_REVERSAL` (12↔21), `MATH_PLACE_VALUE` (25+7→22/95), `MATH_CARRY_OMIT`, `MATH_BORROW_OMIT` (כולל "קטן מגדול"), `MATH_TABLE_NEIGHBOR` (7×8→49/63), `MATH_ADD_FOR_MULT` (7×8→15), `MATH_COUNT_ALL_AGAIN` (K2 counting-on), `MATH_HALF_DOUBLE` |
| כתיב עברי | `HE_SOUNDALIKE_T` (ט/ת), `HE_SOUNDALIKE_K` (כ/ק/ח), `HE_SOUNDALIKE_S` (ס/שׂ), `HE_SOUNDALIKE_V` (ב/ו), `HE_GUTTURAL` (א/ע, ה/א בסוף), `HE_FINAL_FORM` (מ/ם, נ/ן, צ/ץ, פ/ף, כ/ך), `HE_MATER_OMIT` (חסר י/ו), `HE_MATER_EXTRA`, `HE_VISUAL` (ב/כ, ד/ר, ה/ח, ו/ז, ס/ם) |
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
- אם ה-TTS לא זמין: K1 משחקת בתבניות טקסט בלבד (בתצוגה לפי רמת ניקוד); K2/K3 ממתינים (הודעה להורה: "הקול יהיה מוכן בעוד כמה דקות").

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

### 3.15 כשלים ונפילות

| מצב | התנהגות |
|---|---|
| `not_learning_material` | "לא מצאנו כאן תרגיל. אפשר לצלם שוב?" + טיפים. התמונה נמחקת מיד |
| < מינימום של כל תבנית | הצעה: "נמצאו רק 2 מילים — להוסיף עוד מהמילים של {{ילד}}?" |
| חילוץ נכשל פעמיים | "אפשר להקליד את המילים" — מסך הקלדה מהיר (הורה), ואותו צינור מתחיל משלב 7 |
| הרבה `[?]` (> 30% מהמילים) | ממליצים על צילום חוזר; אפשר להמשיך עם המילים הוודאיות בלבד |
| Refusal | נרשם; מוצג "לא הצלחנו לקרוא את הדף הזה"; לא מנסים מודל אחר |
| תקרת $ | §1.9 |

### 3.16 עלות לתמונה (הערכה)

| רכיב | טוקנים | מחיר |
|---|---|---|
| Sonnet 5.5 חילוץ: תמונה ~4.8K + system 2.5K (cache read) + פלט ~2K | | ≈ $0.010 + $0.0003 + $0.02 = **$0.03** |
| + Opus 5.5 (כתב יד בלבד) | אותו דבר | ≈ **$0.06** |
| Haiku 5.5 העשרה: 3K קלט + 3K פלט | | ≈ **$0.002** |
| Azure TTS ~1,500 תווים | | ≈ **$0.02** (או 0 בתוך free tier) |
| **סה"כ לתמונה** | | **$0.05 (מודפס) – $0.11 (כתב יד)** → 30 תמונות/חודש ≈ $2–3 |

