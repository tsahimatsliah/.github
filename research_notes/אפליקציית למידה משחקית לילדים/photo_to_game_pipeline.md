# Snap a Photo → Instant Learning Game: Feasibility, Templates, Pipeline, Pedagogy, Safety

Research date: 2026-11-02. Scope: Israeli family (kids 2.5 / 5.5 / 8.5), iPhone/iPad, Hebrew-first (printed, handwritten, nikud), plus English and math.

Method note: many primary vendor-doc hosts could not be fetched from this research session (wordwall.net, learn.microsoft.com, docs.cloud.google.com, zenn.dev failed DNS in the sandbox; developer.apple.com returned an empty body). Findings for those vendors come from search-result snippets. They are marked as such and listed under Gaps for verification. GitHub and platform.claude.com loaded and were read in full.

---

## Q1. How well do current vision LLMs and OCR services handle Hebrew print, Hebrew handwriting (esp. children's), nikud, math notation, mixed RTL/LTR?

### Takeaway
Hebrew handwriting is still a weak spot for every general model. On the only Hebrew-specific handwriting leaderboard I found (ivrit.ai, Sept 2026), the best LLM full-page word coverage is about 46%, against 89% for a second human reader. Claude models rank low there. Printed Hebrew is a solved problem by comparison. So the product should treat handwriting as "best-effort plus a parent confirm/edit step", should prefer printed worksheets, and should never trust model output for answers without checks. I found no public benchmark at all for children's Hebrew handwriting or nikud OCR.

### Cited Findings
- **The ivrit.ai Hebrew Handwriting OCR leaderboard** is maintainer-run, scored 2026-09-04, and has two tracks. Line mode is ranked by median CER. Full-page mode is ranked by order-independent word coverage. The test set is 225 lines plus 10 pages, gated and test-only. — [itayinbarr/heb-ocr (GitHub)](https://github.com/itayinbarr/heb-ocr)
  - **Line mode, median CER (lower is better):**
    - Human second read: 0.000
    - gemini-flash: 0.119 (13 blanks dropped)
    - Mishkefet-v1 (a 30M-parameter CTC model): 0.175
    - gemini-flash-lite: 0.280
    - gpt-5.6-sol: 0.440
    - gpt-5.6-terra: 0.585
    - claude-sonnet-5: 0.615
    - claude-opus-5: 0.692
    - gpt-5.6-luna: 0.735
    - claude-haiku-4-5: 0.905

    — [heb-ocr](https://github.com/itayinbarr/heb-ocr)
  - **Full-page mode, word coverage (higher is better):**
    - Human: 0.890
    - Mishkefet-v1: 0.463
    - gpt-5.6-sol: 0.462
    - gemini-flash-lite: 0.453
    - claude-opus-5: 0.335
    - gpt-5.6-terra: 0.270
    - gemini-flash: 0.215
    - claude-sonnet-5: 0.210
    - claude-haiku-4-5: 0.197

    Page CER for the best systems is about 0.33–0.40. — [heb-ocr](https://github.com/itayinbarr/heb-ocr)
  - **Caveats the repo itself states:**
    - The median metric flatters gemini-flash, whose micro CER is 2.19.
    - gemini-pro is unranked because 187 of 225 outputs were blank.
    - VLMs on this board have a "documented repetition-loop failure", which CTC models avoid.
    - The eval harness is private, so the metric is reconstructed.

    — [heb-ocr](https://github.com/itayinbarr/heb-ocr)
  - **Data scarcity:**
    - Published real Hebrew handwriting is about 10,219 lines in total: Pinkas has 943 cursive lines (CC-BY-4.0) and BiblIA has 9,276 lines of medieval square script.
    - `sivan22/hebrew-handwritten-dataset` is single-character classification (28 classes, about 4k glyphs), not line-level HHD.
    - The repo's own table is internally inconsistent on some dataset sizes.

    — [heb-ocr](https://github.com/itayinbarr/heb-ocr)
- **General handwriting (mostly English).** AIMultiple's cursive benchmark puts Gemini 3 Pro on top with GPT-5 close behind. The METATR multilingual benchmark (arXiv 2605.26712) reports Gemini 3 Pro strongest across many scripts. Its Hebrew row was not visible to me. — [AIMultiple handwriting benchmark](https://aimultiple.com/handwriting-recognition); [METATR](https://arxiv.org/pdf/2605.26712); [Inksight 31-system benchmark (includes Hebrew Pinkas ledger)](https://www.inksight-app.com/handwriting-ocr-benchmark)
- **Apple Vision (on-device):** I could not confirm Hebrew support.
  - Apple's docs say to query `supportedRecognitionLanguages(for:revision:)` / `supportedRecognitionLanguages()` at runtime. Support differs between the `.fast` and `.accurate` levels. — [Apple docs](https://developer.apple.com/documentation/vision/vnrecognizetextrequest/supportedrecognitionlanguages(for:revision:).md)
  - A 2020 forum thread lists early support as en, fr, it, de, es, pt, zh (no Hebrew). — [Apple Developer Forums](https://developer.apple.com/forums/thread/121048)
  - An iOS 18 test blog reportedly says 18 languages are supported. The visible list had no Hebrew, but it was truncated. — [zenn.dev (snippet only)](https://zenn.dev/shu223/articles/textrecognition)
  - A third-party React Native wrapper README claims Hebrew in the iOS 16+ revision-3 path. This is unverified. — [@dariyd/react-native-text-recognition](https://www.npmjs.com/package/@dariyd/react-native-text-recognition)
- **Apple Live Text / Live Translation (iOS 26):** no source lists Hebrew. — [MacRumors iOS 26.1 languages](https://www.macrumors.com/2025/09/22/ios-26-1-apple-intelligence-languages/)
- **Apple TTS:** Apple AVSpeechSynthesis has supported Hebrew voices since iOS 8. This matters for the TTS step. — [useyourloaf](https://useyourloaf.com/blog/ios-8-adds-hebrew-speech-synthesis/)
- **Google Cloud Vision and Azure Document Intelligence:** the language-support pages could not be fetched (host blocked in this session). See Gaps.

### Inferences
- **Recommended stack for Hebrew:**
  - **Primary extractor:** a frontier VLM (Gemini Flash class or GPT-5.x) on the whole page, asked for structured output. On the leaderboard, Gemini Flash leads lines and GPT-5.6-sol / Gemini Flash-Lite lead full pages.
  - **Optional second pass:** a dedicated Hebrew HTR model (Mishkefet-v1 is a candidate, 30M params, could run on server) on handwritten regions.
  - **Reconcile** the two outputs. Disagreements become "please confirm" tokens shown to the parent.
- **Claude's role:** it scores comparatively poorly on Hebrew handwriting transcription in this one benchmark. It is still a reasonable choice for the reasoning/generation step (classifying content, building items and distractors) once the text is extracted and confirmed.
- **Children's handwriting** (age 5.5–8.5: block letters, reversed letters, invented spelling) is out of distribution for every dataset above. Expect worse than the ~33–40% page CER of adult cursive on some samples. Mitigations: the confirm/edit screen, and a design that works even when only part of the page is read.
- **Spelling games need a guard.** For a child's own story, "OCR error" and "child's spelling error" are indistinguishable to the model. The system must not silently "correct" the child's spelling into the game's answer key. The parent confirms the answer key.
- **Nikud:** VLM OCR of nikud is likely unreliable, because the marks are tiny and photos are low-resolution. The safer design is:
  1. Extract consonantal text.
  2. If the source had nikud (gan / first grade), regenerate it with Dicta Nakdan.
  3. Let the parent compare against the photo.
- **Mixed RTL/LTR and math:** keep math as structured tokens (operands, operator, blank position), never as display strings. Render with explicit bidi isolation (`⁨…⁩` / SwiftUI `.environment(\.layoutDirection)`). Hebrew worksheets write `5 + 3 = __` LTR inside RTL text, and naive concatenation flips the operand order.

### Gaps
- No public benchmark of **children's** Hebrew handwriting OCR. Building a small internal eval set (50–100 photos of the family's own pages, with consent) is the realistic path.
- No benchmark found for **nikud OCR** accuracy from photos.
- Apple Vision Hebrew support is **unconfirmed**. Verify on-device with `supportedRecognitionLanguages` (iOS 26). Apple's doc page body did not render for me.
- Google Cloud Vision (`docs.cloud.google.com/vision/docs/languages`) and Azure Document Intelligence OCR language pages (`learn.microsoft.com/.../language-support/ocr`) were blocked in this sandbox.
  - My background knowledge, unverified here: both list Hebrew for **printed** text. Azure's handwritten support covers a short list of mostly Latin/CJK languages, and I believe Hebrew handwriting is not on it.
  - These hosts must be allowed to verify.
- No Hebrew math-notation benchmark found. OCRBench was not fetched.

---

## Q2. Existing products that generate learning activities from photos/text, and how they map content to templates

### Takeaway
The market pattern is "one content set (term/definition or Q/A list) → many interchangeable game skins". Wordwall is the clearest model, with about 30 templates and template switching that needs no re-entry. AI generation is usually text/PDF → quiz, not photo → multi-template game. TinyTap's AI is prompt-to-game, not photo-to-game. Kahoot's generator is PDF-based, English-first, and limited to quiz-like types. No product I found does photo of a Hebrew worksheet → several playable game templates for young kids. That is a gap the app can fill.

### Cited Findings
- **Wordwall** (from search snippets; the site was blocked for direct fetch):
  - **Templates:** Airplane, Anagram, Balloon pop, Complete the sentence, Crossword, Find the match, Flash cards, Flying fruit, Gameshow quiz, Group sort, Hangman, Image quiz, Labelled diagram, Match up, Matching pairs, Maze chase, Open the box, Pair or No Pair, Quiz, Rank order, Speed sorting, Spell the word, Spin the wheel, True or false, Type the answer, Unjumble, Watch and memorize, Whack-a-mole, Win or lose quiz, Wordsearch.
  - **Tiers:** Free has 12 templates. Pro has 34.
  - **Content constraints:** Whack-a-mole needs ≥5 correct and ≥5 incorrect answers. Anagram and Wordsearch need plain text only.
  - **AI:** "Generate With AI" (Standard/Pro) creates a Quiz from a prompt or an uploaded document.
  - **Template switching:** you can switch templates without re-entering content.

  — [Wordwall features (snippet)](https://wordwall.net/features); [Wordwall Quiz template page](https://wordwall.net/cb/about/template/quiz); [TeachersFirst on Wordwall](https://teachersfirst.org/blog/?p=12820)
- **Kahoot!:**
  - The AI generator takes PDF uploads of roughly 150 pages / 300k characters.
  - It outputs Quiz, True/False, Slider, and Type-answer questions.
  - "The main language of the Question generator is English."
  - It is a higher-tier plan feature.
  - I found no official confirmation of direct image upload; only third-party sites claim it.

  — [Kahoot support: generate questions with AI](https://support.kahoot.com/hc/en-us/articles/40988856361747); [Kahoot blog: PDF-to-kahoot](https://kahoot.com/blog/2024/01/17/ai-pdf-question-generator-for-educators/); [Kahoot AI tools](https://support.kahoot.com/hc/articles/17152945038355)
- **TinyTap:**
  - "Prompt-to-Game" AI beta launched August 2023: topic prompt → complete game.
  - A companion tool does text-to-image.
  - Photo use is manual: the teacher uploads or takes pictures, records a spoken question, and selects the region of the picture that is the answer.
  - I found no worksheet-photo → game feature.

  — [TechBriefly](https://techbriefly.com/2023/08/30/edtech-firm-tinytap-launches-ai-prompt-features-for-educational-games); [TinyTap AI page](https://www.start.tinytap.com/ai); [FreeTech4Teachers walkthrough](https://freetech4teach.teachermade.com/?p=15463)
- **Quizlet Magic Notes:**
  - Upload or paste notes, and it creates outlines, flashcards, practice tests, summaries, and essay prompts.
  - Input limit is 40,000 characters.
  - Reportedly renamed "study guides" (low-quality source).
  - Unlimited use is reportedly tied to Quizlet Plus (third-party).
  - I found no reliable confirmation of handwritten-photo input.

  — [EducatorsTechnology](https://www.educatorstechnology.com/2023/10/quizlets-latest-back-to-school-features.html); [Toolmage](https://www.toolmage.com/de/tool/quizlet/)
- **Photomath / Gauth and the cheating debate:**
  - Photomath said "We didn't develop PhotoMath as a cheating tool." — [TIME](https://time.com/3545695/homework-apps)
  - Teachers have had to safeguard tests against such apps. — [Fox5 NY](https://www.fox5ny.com/news/cheating-a-free-for-all-at-virtual-high-schools-teachers-claim)
  - Some teachers are unconcerned and shift toward real-world problems the app can't solve. — [Stanford GSE](https://ed.stanford.edu/node/4490)
  - Alice Keeler argues that if Photomath can solve it, the task isn't hitting mathematical practices. — [Alice Keeler](https://alicekeeler.com/2022/09/29/photomath-is-not-cheating/)
  - Gauth critics: students copy answers without reading the explanations. This comes from a commercial blog, Oct 2025, so treat it cautiously. — [gstory.ai](https://www.gstory.ai/blog/gauth-ai/)

### Inferences
- **Copy Wordwall's architecture.** Use a small number of canonical content shapes (pairs, Q/A with options, ordered sequence, categorised items, cloze sentence, single word to spell), and have every template declare which shapes and minimum counts it accepts. Wordwall's "Whack-a-mole needs ≥5 correct + ≥5 incorrect" and "Anagram needs plain text" are exactly that kind of per-template constraint.
- **Differentiators vs. existing tools:** Hebrew-first; photo of a handwritten page as input; ages 2.5–8.5, with audio-first templates for pre-readers; and generating *practice* instead of *answers*, which inverts the Photomath/Gauth model.

### Gaps
- Not verified this session: Blooket/Gimkit AI, H5P content types, Diffit, MagicSchool, Brisk, Khanmigo, Google NotebookLM quizzes/flashcards, Google Lens homework help, Duolingo exercise types. My searches returned nothing on NotebookLM. These come from background knowledge only and need checking before they are cited:
  - **H5P** has ~50 content types: Drag the Words, Mark the Words, Fill in the Blanks, Memory Game, Flashcards, Dialog Cards, Image Hotspots, Sort the Paragraphs, etc.
  - **NotebookLM** added flashcards and quizzes from sources in 2025.
  - **Duolingo** exercise types include select-the-translation, tap-the-pairs, listen-and-type, word bank, and speak.
- No official Kahoot image-upload confirmation.

---

## Q3. Template taxonomy: content type → template → age; auto-selection; distractors; adaptivity and spacing

### Takeaway
Map each extracted content type to a canonical item shape, then pick templates by item shape, age band, and modality (audio vs. reading). LLMs produce mathematically valid distractors but are weak at anticipating real misconceptions. So math distractors should come from a coded misconception library, and LLM distractors should be used mainly for language items. Every generated item should also be written to a spaced-repetition queue.

### Cited Findings
- **LLM distractors and misconceptions:** LLMs can generate mathematically valid distractors but are "less adept at anticipating common errors or misconceptions among real students". In-context learning with about 3 similar example MCQs (k=3) worked best among the methods tested. — [Feng et al., NAACL 2024 Findings, arXiv 2404.02124](https://arxiv.org/abs/2404.02124)
- **Per-template content constraints exist in practice:** Whack-a-mole needs ≥5 correct and ≥5 incorrect items; Anagram and Wordsearch need plain text. — [Wordwall (snippet)](https://wordwall.net/features)

### Inferences (design proposal; not externally sourced)
- **Content types → item shapes → templates (ages):**

| Content type (from photo) | Canonical item shape | Age 2.5 (pre-reader, audio/tap) | Age 5.5 (emergent reader, gan/K) | Age 8.5 (grade 3) |
|---|---|---|---|---|
| Vocabulary list (He/En) | pairs(word↔image/translation) | Tap the picture you hear; Memory pairs (images + TTS) | Match up, Balloon pop (hear word → pop image), Find the match | Match up, Flash cards, Speed sorting, Crossword |
| Spelling words | word + audio | – | Letter-first-sound (which letter starts "בית"?), Unjumble 2–3 letters | Spell the word (hear → arrange letters), Anagram, Hangman, Wordsearch |
| Letters / nikud (gan) | grapheme + sound | Find the letter you hear (big targets) | Whack-a-mole the letter/nikud sound; Group sort (קמץ vs. חיריק) | – |
| Math exercise set | expression with blank | Count-and-tap (dots), "more/less" | Number bonds, Balloon pop the answer, Maze chase | Quiz with misconception distractors, Speed round, True/False equations, Rank order |
| Reading passage | Q/A + sequence + cloze | Picture-story listen + "who/what" tap | Sequence pictures (Rank order), True/False read aloud | Comprehension quiz, Complete the sentence (cloze), Rank order events |
| Child's own story | characters, events, sequence | Listen to it as a picture story | Sequence the events, "who did what" | Comic with choices, cloze from own sentences, spelling from own words (parent-confirmed) |
| Facts (science/Torah/holidays) | Q/A, categories | Image quiz | Group sort, True/False | Quiz, Open the box, Gameshow |

- **Auto-select rule (deterministic, after the LLM classifies content):**
  1. Filter templates by item shape and minimum item count. For example, Whack-a-mole needs ≥5 correct and ≥5 incorrect, so generate extra distractor items, or skip the template.
  2. Filter by the child's age band and reading mode. Under 6 → audio-first, ≤4 options, no typing.
  3. Rank by novelty, preferring templates not played in the last N sessions.
  4. Offer 2–3 choices and default to the top one ("instant game").
- **Distractors by domain:**
  - **Math:** generate in code from a misconception catalogue:
    - operation swap (5+3 → 2)
    - off-by-one (7 / 9)
    - digit reversal (12 ↔ 21)
    - place-value error (25+7 → 22 or 95)
    - "add instead of subtract on the word problem" for 8.5-year-olds

    Validate that every distractor ≠ the answer and is in a sensible range.
  - **Hebrew spelling:** confusable-letter substitutions (ט/ת, כ/ח/ק, ס/ש, א/ע, ו/ב), final-form errors (מ/ם, נ/ן, צ/ץ, פ/ף, כ/ך), and missing/extra matres lectionis (י/ו). Generate these by rule, then have the LLM filter out ones that form real words when that would cause confusion.
  - **Vocabulary:** same semantic category as the answer (other animals, other colors). Use the LLM, constrained to words already in the child's known lexicon where possible.
- **Adaptive + spaced:**
  - Each atomic item (word, fact, math fact) becomes a card in the per-child SRS queue (FSRS or Leitner) with `source_photo_id`, `skill_tag`, and an `age_variant` for each sibling.
  - The first session is the immediate game (same-day retrieval). Then schedule reviews at roughly +1d, +3d, +7d…, and mix them into later games of any template.
  - Difficulty inside a session: more distractors, a timer, or typing instead of tapping after 2 consecutive correct answers. Fall back after 2 errors.

### Gaps
- No found source specifically validates which game templates work best per age for Hebrew-speaking children. The age mapping above is design judgment informed by general early-literacy practice.
- I did not fetch FSRS / spaced-repetition sources this session.

---

## Q4. Pipeline design, structured-output schemas, latency and cost

### Takeaway
A viable pipeline:

1. On-device capture and cleanup.
2. Optional on-device PII blur.
3. Server-side VLM extraction to strict JSON, using structured outputs (now stable on the Claude API, and available on OpenAI with `strict: true`).
4. Deterministic validation (math in code, Hebrew nikud via Dicta Nakdan, dedupe).
5. Template fill and TTS.
6. Parent confirm.
7. Play.

The schema must respect provider JSON-schema subsets. For example, Claude structured outputs do not support numeric min/max or minItems > 1, so enforce those constraints in code.

### Cited Findings
- **Claude structured outputs** (read from the docs page directly):
  - **Status:** stable / GA on the Claude API, AWS, Google Cloud, Bedrock, and Microsoft Foundry. No beta header needed.
  - **How to enable:** `output_config.format` with `{type: "json_schema", schema: …}`. Strict tool use is `strict: true` on tool definitions. `output_format` plus the `structured-outputs-2025-11-13` header is deprecated.
  - **Supported models:** include Opus 5.5 / 5, Sonnet 5.5 / 5, Haiku 5.5 / 4.5, Opus 4.5–4.8, Sonnet 4.5 / 4.6, and others.

  — [Claude docs: structured outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs)
- **Claude JSON-schema limits:**
  - **Not supported:** recursive schemas, numeric constraints (`minimum` / `maximum` / `multipleOf`), string length limits, array `minItems` / `maxItems` beyond 0/1, `additionalProperties` other than `false`, external `$ref`. Regex is partial (no lookarounds or `\b`).
  - **Complexity limits:** up to 20 strict tools, 24 optional params, 16 union-type params.
  - **Output caveats:** `stop_reason: "refusal"` or `"max_tokens"` can yield non-conforming output. Enum casing may vary.
  - **Images:** the docs list "extract data from images" as a use case.

  — [Claude docs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs)
- **OpenAI** offers `response_format: {type: "json_schema", strict: true}`, which requires all fields. Both providers support only a subset of JSON Schema. — [logic.inc structured outputs guide](https://logic.inc/resources/structured-outputs-guide); [levelup article](https://levelup.gitconnected.com/claude-api-structured-outputs-schemas-not-hope-952284782089)
- **Dicta Nakdan:**
  - Nakdan combines neural models with curated linguistic knowledge and dictionaries. — [Nakdan paper arXiv 2005.03312](https://arxiv.deeppaper.ai/papers/2005.03312v1?locale=en)
  - A hosted API exists; terms are at dicta.org.il/developers (unverified details). — [hebrew-nlp-toolkit guide](https://claudeskills.info/skills/skills-il/localization/hebrew-nlp-toolkit/)
  - The D-Nikud project trained on Dicta-diacritized text "subsequently manually corrected", which implies Dicta output still needs human correction. — [D-Nikud arXiv 2402.00075](https://arxiv.org/html/2402.00075)
  - Nakdimon is an open alternative model. — [Nakdimon on Replicate guide](https://dev.to/aimodels-fyi/a-beginners-guide-to-the-nakdimon-model-by-elazarg-on-replicate-fa4)
- **TTS:** Hebrew system TTS is available on iOS (AVSpeechSynthesizer). — [useyourloaf](https://useyourloaf.com/blog/ios-8-adds-hebrew-speech-synthesis/)

### Inferences (proposed design)
- **Pipeline stages:**
  1. **Capture (on-device):** VisionKit document camera (`VNDocumentCameraViewController`) for perspective correction. Multi-page allowed.
  2. **Privacy pre-pass (on-device):**
     - Face detection (`VNDetectFaceRectanglesRequest`) → blur.
     - Optional on-device text recognition (if Hebrew is supported) to find name-like tokens.
     - Crop to the exercise region the user selects.
  3. **Extraction (server VLM, structured output):** returns `content_type`, `language(s)`, `items[]`, a per-token confidence / `uncertain_spans`, and `has_nikud`, `handwritten`, `source_kind` (worksheet / notebook / story / homework).
  4. **Validation (code):**
     - Recompute math answers with a parser/evaluator. Reject the item if it disagrees with any LLM-provided answer.
     - Run nikud through Dicta Nakdan when needed.
     - Normalise final letters.
     - Drop items under a confidence threshold, or route them to the parent.
     - Check template minimum counts.
  5. **Generation (LLM, structured output):** per-age variants, distractors (language) plus coded distractors (math), short hints and praise lines (Hebrew), and an optional story → comic script.
  6. **TTS:** iOS on-device Hebrew voice for instant play. Optionally use a higher-quality cloud Hebrew voice, cached per item.
  7. **Parent review:** a 10-second screen with the photo side by side with the extracted items. Tap to fix. "Approve & play" with a per-child selection.
  8. **Play + SRS enqueue.**
- **Minimal schema sketch** (fits Claude's subset: `additionalProperties: false`, no numeric bounds, enums for types; counts enforced in code):

```json
{
  "type": "object",
  "additionalProperties": false,
  "required": ["source_kind", "content_type", "primary_language", "items", "pii_detected"],
  "properties": {
    "source_kind": {"type": "string", "enum": ["worksheet", "notebook", "homework", "child_story", "book_page", "other"]},
    "content_type": {"type": "string", "enum": ["vocabulary", "spelling", "letters_nikud", "math_facts", "word_problems", "reading_passage", "story", "facts"]},
    "primary_language": {"type": "string", "enum": ["he", "en", "mixed"]},
    "handwritten": {"type": "boolean"},
    "has_nikud": {"type": "boolean"},
    "pii_detected": {"type": "array", "items": {"type": "object", "additionalProperties": false, "required": ["kind", "text"], "properties": {"kind": {"type": "string", "enum": ["person_name", "school_name", "phone", "other"]}, "text": {"type": "string"}}}},
    "items": {"type": "array", "items": {"type": "object", "additionalProperties": false,
      "required": ["id", "shape", "prompt", "answer", "confidence"],
      "properties": {
        "id": {"type": "string"},
        "shape": {"type": "string", "enum": ["pair", "qa", "cloze", "sequence", "category", "spell", "math_expr"]},
        "prompt": {"type": "string"},
        "answer": {"type": "string"},
        "math": {"type": "object", "additionalProperties": false, "required": ["left", "op", "right"], "properties": {"left": {"type": "string"}, "op": {"type": "string", "enum": ["+", "-", "x", ":"]}, "right": {"type": "string"}}},
        "category": {"type": "string"},
        "order_index": {"type": "integer"},
        "confidence": {"type": "string", "enum": ["high", "medium", "low"]},
        "skill_tag": {"type": "string"}
      }}}
  }
}
```

- **Latency target:** under ~10–15 s from shutter to playable game. Ways to get there:
  - Start the game with the first validated items while the rest stream in.
  - Pre-built templates (no code generation).
  - On-device TTS first.
  - One extraction call plus one generation call, which can be merged into one call for simple worksheets.
- **Cost:** a single photo is roughly a few thousand input tokens plus 1–3k output tokens per call. At 2026 frontier "flash/haiku-class" pricing, that is likely well under $0.01–0.05 per photo.

### Gaps
- I did not retrieve current per-token pricing or image-token accounting for Claude, OpenAI, or Gemini this session, so the cost estimate above is an unsourced order-of-magnitude figure.
- No measured latency figures found for VLM page extraction.
- Dicta Nakdan API terms, rate limits, and commercial licensing are not verified. Check dicta.org.il/developers.
- OpenAI and Gemini official structured-output docs were not fetched directly.

---

## Q5. Pedagogy: practice instead of answers, same-day retrieval then spacing, child's story → personalized story, sibling adaptation

### Takeaway
The anti-cheating stance should be built into the product: the app never outputs the solutions to the photographed homework. It generates parallel practice items on the same skill, and it uses the photographed items themselves only after the homework is done, or for review. This answers the Photomath/Gauth critique, where students copy answers without engaging.

### Cited Findings
- **The critique of photo-solvers:** students copy answers without reviewing explanations (Gauth). — [gstory.ai](https://www.gstory.ai/blog/gauth-ai/)
- **The case for photo-solvers:** they can be a legitimate learning aid, while teachers adapt assessment. — [Fox5 NY](https://www.fox5ny.com/news/cheating-a-free-for-all-at-virtual-high-schools-teachers-claim); [Stanford GSE](https://ed.stanford.edu/node/4490)
- **Alice Keeler's argument:** if an app can solve the task, the task doesn't target mathematical practices. — [Alice Keeler](https://alicekeeler.com/2022/09/29/photomath-is-not-cheating/)

### Inferences
- **Rules:**
  1. **Homework mode:** clone the skill, not the item. Change the numbers or words. For example, 7+5 → 6+5, 8+4.
  2. **The answer key is parent-only.** The child never sees answers to the actual homework items before submitting them.
  3. **Review mode** (after school / the next day) can use the exact items: retrieval practice of today's lesson.
- **Scheduling:** same-day game (within hours) → SRS reviews (+1d, +3d, +1w), interleaved with older items.
- **Child's own story → interactive story/comic:** only with the child's and parent's permission. Keep the child's wording visible as "authored by". Generated illustrations should be generic characters, never faces from the photo. Never "correct" the story publicly. Spelling practice is drawn only from words the parent marks.
- **Sibling adaptation of one source, for example a grade-3 vocab sheet about animals:**
  - **2.5:** hear "כלב" → tap the dog picture.
  - **5.5:** first-letter sound game + memory pairs.
  - **8.5:** spell-the-word / anagram + cloze sentences.

  Store one `source_id` with per-child item variants.

### Gaps
- I did not fetch primary retrieval-practice or spacing research (Roediger & Karpicke; Cepeda et al.) in this session. The report writer should cite these from other researchers' notes if available.
- No sources found on outcomes of "AI-generated games from homework photos" for young children.

---

## Q6. Safety and privacy: classmates' names and faces, on-device first, parent approval

### Takeaway
Photos of gan and school pages routinely include other children's names, the teacher's name, the school name, and sometimes faces or class photos. The pipeline should crop, detect, and redact on-device before upload, minimise retention, and gate every game behind parent approval. I found no product-specific sources in this session, so this section is design guidance.

### Cited Findings
- Apple's Vision framework provides on-device text recognition, with the language list queryable at runtime, which enables an on-device first pass (Hebrew support unconfirmed). — [Apple docs](https://developer.apple.com/documentation/vision/vnrecognizetextrequest/supportedrecognitionlanguages().md)
- The extraction schema can carry a `pii_detected` field. Structured outputs guarantee its presence when the response isn't a refusal or truncation. — [Claude docs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs)

### Inferences
- **On-device:**
  - Face blur (Vision face detection).
  - The user crops to the exercise area.
  - Optional text-based name redaction once on-device Hebrew OCR is confirmed. Otherwise do it server-side with immediate deletion.
- **Server:**
  - Use the API with zero/limited data retention options.
  - Delete the raw photo after extraction by default. Keep only the parent-approved item JSON.
  - Replace detected person names with placeholders before any content reaches games, TTS, or illustrations. The child's own name is allowed only if the parent opts in.
- **Parent approval:** required before any generated game is playable (kids' accounts never trigger generation without approval), and before any child-story content becomes a comic.
- **Content safety:** run generated text through a kid-safety filter. Do not generate realistic images of real people from the photo.

### Gaps
- I did not research Israeli privacy law (Privacy Protection Law Amendment 13, in force Aug 2025), COPPA, or GDPR-K specifics, or Apple's Kids Category rules, in this session. Those need separate sourcing.
- I did not verify vendor data-retention terms (Anthropic, OpenAI, Google) for image inputs.
