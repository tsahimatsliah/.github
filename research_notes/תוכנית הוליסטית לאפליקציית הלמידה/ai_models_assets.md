# AI Models & Generative Services Map + Asset Pipeline (family-only Hebrew-first kids' learning game, Nov 2026)

> Research conditions: Done 2026-11-02. The session's network sandbox **blocked direct fetches of ai.google.dev and openai.com** (DNS not allowed), so Google and OpenAI prices below come from web-search summaries of third-party aggregators and are marked "unverified". To verify them, those domains need to be allowed in the session's network settings. Anthropic pricing and Anthropic's minors guidance were fetched from official pages. About 17 tool calls were used, so several sub-questions (Rive AI features, Lottie generation, Meshy/Tripo, Stable Audio, Midjourney API, Layer.ai, Leonardo) were **not researched** and appear under Gaps.

## 1. LLM jobs: which model per job, prices, caching/batch, safety and vendor policy on minors

### Takeaway
Anthropic's Nov-2026 lineup makes Claude the cheapest strong default for every text job at family scale. Use Opus 5.5 via the Batch API for offline Hebrew content generation, Sonnet 5.5 as the QA judge, and Haiku 5.5 ($0.10/$0.50 per MTok) for the live Socratic companion. Use Gemini 3 Flash for worksheet-photo extraction, with Claude as the vision fallback. At family scale, LLM spend is roughly **$5–25/month**. No reliable public benchmark compares frontier models on Hebrew-with-nikud, so the project must run its own small eval.

### Cited Findings

**Anthropic pricing (official, fetched)**
- Claude Opus 5.5: $4/MTok input, $20/MTok output. 5-min cache write $5, 1-h cache write $8, cache hit $0.20 (cache hits are 0.05x base input on Opus 5.5 and Sonnet 5.5) — [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing)
- Claude Sonnet 5.5: $2 input / $10 output, cache hit $0.10. Claude Sonnet 5: $2 / $10. The intro price became permanent; the planned rise to $3/$15 on Sep 1, 2026 "will not occur" — [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing)
- Claude Haiku 5.5: $0.10 input / $0.50 output / $0.01 cache hit for prompts up to 100K tokens. For prompts over 100K: $0.50 / $2.50 — [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing)
- Higher tiers: Claude Fable 5.1 and Mythos 5.1 at $10/$50 (Mythos is limited availability). Opus 5 / 4.x at $5/$25. Haiku 4.5 at $1/$5 — [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing)
- The Batch API takes 50% off input and output: Opus 5.5 batch $2/$10, Sonnet 5.5 batch $1/$5, Haiku 5.5 batch $0.05/$0.25. Batch and prompt-caching discounts stack — [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing)
- Claude 4.7+ uses a newer tokenizer that "produces approximately 30% more tokens for the same text". Claude 4.6+ (except Haiku 5.5) includes the 1M context at standard pricing. US-only inference (`inference_geo`) costs 1.1x — [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing)
- Fast mode for Opus 5.5 costs $8/$40 and is not available with Batch. Web search tool costs $10 per 1,000 searches — [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing)

**Google Gemini pricing (unverified; aggregators only, official page blocked)**
- Gemini 3 Flash: most sources list $0.50 input / $3 output per MTok, with 50% batch ($0.25/$1.50) — [eesel.ai](https://www.eesel.ai/blog/google-gemini-3-pricing), [puter](https://developer.puter.com/tutorials/gemini-api-pricing/), [benchlm](https://benchlm.ai/google/api-pricing)
- Gemini 3.1 Pro: $2 / $12 (≤200K prompt) and $4 / $18 (>200K). Batch is $1/$6 — [metacto](https://www.metacto.com/blogs/the-true-cost-of-google-gemini-a-guide-to-api-pricing-and-integration), [morphllm](https://www.morphllm.com/gemini-api-pricing)
- Some aggregators mention newer "Gemini 3.5–3.8 Flash" and a "Gemini 4 Argon". I could not confirm these. Conflicting batch figures ($0.375/$1.875) appear to refer to a "3.8 Flash" — [morphllm](https://www.morphllm.com/gemini-api-pricing)

**OpenAI pricing (unverified; aggregators only, official page blocked)**
- GPT-5.5: $5 input / $30 output per MTok. One source says it matches "GPT-5.6 Sol". Sources disagree on the current flagship naming (GPT-5.6 Sol/Terra/Luna vs "GPT-6 Astra") — [Finout](https://www.finout.io/blog/openai-pricing-in-2026), [morphllm](https://www.morphllm.com/openai-api-pricing), [devtk](https://devtk.ai/en/blog/openai-api-pricing-guide-2026/)
- GPT-5.4 mini $0.75/$4.50 and GPT-5.4 nano $0.20/$1.25. Another source lists gpt-5-nano at $0.05/$0.40. These conflict — [G2](https://www.g2.com/articles/openai-api-pricing), [OpenAI model page gpt-5-mini](https://developers.openai.com/api/docs/models/gpt-5-mini)

**Hebrew quality**
- Dicta-LM 3.0 (arXiv 2602.02104, Feb 2026) is a Hebrew "sovereign" LLM with base and chat variants, tool calling, and 65K native context — [arXiv](https://arxiv.org/abs/2602.02104), [Dicta tech report](https://dicta.org.il/publications/DictaLM_3_0___Techincal_Report.pdf)
- The Hebrew chat leaderboard includes a **Nikud (diacritization)** task. DictaLM-3.0-24B-Thinking scores 86.87 on it — [HF hebrew-llm-leaderboard chat-results](https://huggingface.co/datasets/hebrew-llm-leaderboard/chat-results), [Moonlight review](https://www.themoonlight.io/en/review/dicta-lm-30-advancing-the-frontier-of-hebrew-sovereign-llms)
- The authors claim instruct-tuned DictaLM 2.0/3.0 outperform other open-weight and commercial LLMs on most Hebrew tasks. This is a self-reported claim, and no per-model figures for Claude/GPT/Gemini were retrievable — [emergentmind summary](https://www.emergentmind.com/topics/hebrew-llm-benchmark-suite)
- Dicta's eval judges summarization and translation with GPT-4o against Gemini-2.5-Pro outputs, so it does not test the 2026 frontier models — [Moonlight review](https://www.themoonlight.io/en/review/dicta-lm-30-advancing-the-frontier-of-hebrew-sovereign-llms)

**Photo-to-game (worksheet vision / Hebrew handwriting)**
- No benchmark directly compares Gemini, Claude and GPT on **Hebrew** handwriting. HebOCRBench v1 covers printed text only, and its maintainers say its scores must not be presented as handwriting results — [HebOCRBench](https://huggingface.co/datasets/ssdataanalysis/hebocrbench-v1)
- An English-handwriting test by AIMultiple (100 samples, updated July 2026) reports Gemini 3 Pro at 100% accuracy, with Claude Sonnet 4.5 also in the lineup — [AIMultiple](https://aimultiple.com/de/handwriting-recognition)
- HebVL (a fine-tuned Qwen3-VL-8B) beats Kraken on historical Genizah handwriting (F1 0.807 vs 0.730). This is a different domain from kids' worksheets — [HebVL](https://friendli.ai/models/isaacmg/qwen3-vl-8b-hebrew-v18a-ckpt)
- OmniHandwritingOCR (arXiv 2608.18586) is a new multimodal handwriting and handwritten-math benchmark. Whether it covers Hebrew is unverified — [arXiv](https://arxiv.org/pdf/2608.18586)

**Vendor policies on minors**
- Anthropic (official help center): organizations giving minors access must (a) add safeguards such as age verification, content moderation/filtering, monitoring/reporting and educational resources; (b) implement Anthropic's **child-safety system prompt** where provided; (c) comply with COPPA and similar laws and state this publicly; (d) **disclose to users that they are talking to an AI**. Anthropic audits periodically and may suspend or terminate accounts with high violation rates that refuse to adopt safeguards. No minimum age is set — [Anthropic: Guidelines for organizations serving minors](https://support.claude.com/en/articles/9307344-responsible-use-of-anthropic-s-models-guidelines-for-organizations-serving-minors)
- Anthropic also publishes separate child-safety guidance for developers. Developers are responsible for safeguards, and Anthropic monitors API usage. Claude's own consumer products are 18+ — [Anthropic child safety guidance for developers](https://support.claude.com/en/articles/15591275-child-safety-guidance-for-developers)
- OpenAI "Under 18 API Guidance": add safeguards beyond the usage policies, comply with COPPA, make content "safe and age appropriate". OpenAI may audit. **Do not process personal data of children under 13 without first implementing Zero Data Retention (ZDR)**. The universal usage policies took effect Oct 29, 2025, with a "Keep minors safe" section — [OpenAI Under-18 API guidance](https://developers.openai.com/api/docs/guides/safety-checks/under-18-api-guidance)
- Google/Gemini policy on minors: not researched (see Gaps).

### Inferences
- **Recommended map (LLM):**
  | Job | Primary | Fallback | Est. cost/mo (family) | Notes |
  |---|---|---|---|---|
  | Offline Hebrew content (stories with nikud, word problems, quizzes) | Claude Opus 5.5 via Batch API ($2/$10) with a cached style guide and few-shot examples | Gemini 3.1 Pro batch (~$1/$6, unverified); GPT-5.5 | ~$5–15 (example: 500 items × 3K in / 1.5K out ≈ 1.5M in + 0.75M out ≈ $3 + $7.5 before caching) | Use structured JSON outputs (schema: text, nikud text, vocab, difficulty, age band). Generate nikud either by asking the LLM directly or by running a **separate diacritization pass with Dicta's nakdan tooling or DictaLM 3.0**, then have the judge compare. A human parent spot-checks a sample. |
  | Photo-to-game extraction | Gemini 3 Flash (cheap; matches the earlier "Flash best on lines" finding) | Claude Sonnet 5.5 vision; Gemini 3.1 Pro for hard pages | <$1 (≈30 photos/mo) | Re-test on the family's real worksheets (20–30 Hebrew handwriting samples) before committing. No published Hebrew-handwriting comparison of frontier models exists. |
  | Socratic companion (hints, not answers) | Claude Haiku 5.5 ($0.10/$0.50; cache hits $0.01) | Sonnet 5.5 for the 8.5-year-old's harder math | <$1 (Haiku) to ~$3–10 (Sonnet) for ~1,800 turns/mo | Keep a long cached system prompt (persona, "never give the final answer", age band, Hebrew register) plus Anthropic's child-safety system prompt. Disclose that it is an AI. Add an output check that blocks replies containing the answer key. |
  | Parent dashboard summaries | Claude Haiku 5.5 or Sonnet 5.5, run nightly in batch | Gemini 3 Flash | <$1 | Send aggregated event logs only, never raw child voice or photos. |
  | LLM-as-judge QA | Claude Sonnet 5.5 batch ($1/$5) | A different vendor (Gemini 3.1 Pro) for cross-family judging | ~$1–3 | Use a rubric (age appropriateness, Hebrew grammar/nikud accuracy, math correctness, safety). Judge with a model from a different family than the generator to reduce self-preference bias. |
- **Total LLM cost at family scale: roughly $5–25/month.** It is dominated by the content-generation volume chosen, not by live use.
- Because the app is private and family-only, COPPA's "operator" duties may not formally apply. The vendor terms still apply, though: Anthropic's AI-disclosure and safeguard expectations, and OpenAI's under-13 ZDR prerequisite. The safest design keeps live child-facing LLM calls minimal and stateless (no PII). Most content is pre-generated offline and parent-approved.
- Safety layers: (1) the vendor's own guardrails plus Anthropic's child-safety prompt; (2) a cheap classifier pass on every child-facing output (Haiku 5.5 as a classifier, or OpenAI moderation / Llama Guard; pricing and availability **unverified** here); (3) a whitelist-style UX where the companion only discusses the current exercise; (4) logging for parent review.

### Gaps
- Official Gemini and OpenAI price pages could not be fetched (sandbox DNS block). All Google and OpenAI prices are unverified and the aggregators conflict (e.g., gpt-5-nano $0.05/$0.40 vs GPT-5.4 nano $0.20/$1.25).
- No found source compares Claude 5.x, GPT-5.x, Gemini 3.x and DictaLM 3.0 on Hebrew generation or nikud accuracy. A per-model nikud table may exist in the Dicta tech report PDF or the HF leaderboard, but it wasn't retrieved.
- Not verified: OpenAI moderation API price (historically free), Llama Guard 4 availability, Google's policy on minors for the Gemini API, and rate limits per vendor tier.
- Whether Anthropic's child-safety system prompt is published, and where, was not confirmed.

## 2. Image generation for a consistent house style (characters, UI, avatar parts)

### Takeaway
Use **Recraft V4 Styles (custom style; SVG/vector output)** for icons, UI and flat illustration. Use **Gemini Nano Banana 2 / Pro** (reference-image editing, consistent characters) or **GPT Image 2/2.5** (top of the leaderboards) for character art and scenes. **Scenario** is an option if a trained, private style model for game assets is wanted. At family scale, image spend is roughly **$10–40/month** during production and near zero afterwards.

### Cited Findings
- Artificial Analysis Text-to-Image Arena (current snapshot): GPT Image 2.5 Sunburst (max) leads at Elo 1196, GPT Image 2.5 Flare (max) at 1190, GPT Image 2 (high) at 1171. The open-weights leader is Qwen-Image-2.1 (1035), followed by Ideogram 4.0 — [AA text-to-image leaderboard](https://artificialanalysis.ai/image/leaderboard/text-to-image?category=uiux_design)
- AA Image-Editing v2.1: GPT Image 2.5 Sunburst (max) at 1199, MAI-Image-2.7-Preview at 1193, GPT Image 2.5 Flare at 1182, then MAI-Image-2.6 and Krea 3 (Ultra) at 1151. Open-weights editing leaders are Fibo 2.0 (1088), Qwen-Image-2.1 (1081) and FLUX 3 Image Klein (1072) — [AA editing leaderboard](https://artificialanalysis.ai/image/leaderboard/editing), [AA open-weights editing](https://artificialanalysis.ai/image/leaderboard/editing/open-weights)
- Earlier, GPT Image 2 (high) debuted at #1, ahead of Nano Banana 2, FLUX.2 [max] and Seedream 4.0 — [Artificial Analysis on X](https://x.com/ArtificialAnlys/status/2047184062012706980)
- GPT Image 2.5 was released Sep 8, 2026 — [Wikipedia: GPT Image](https://en.wikipedia.org/wiki/GPT_Image)
- OpenAI image pricing is per token: gpt-image-2 $8/MTok in and $30/MTok out; gpt-image-1-mini $2.50/$8, or about $0.005–0.036 per 1024² image (unverified) — [Modellix](https://www.modellix.ai/blog/openai-image-generation-api-pricing)
- Nano Banana Pro (gemini-3-pro-image): $0.134 per 1K/2K image and $0.24 per 4K, with batch at $0.067 per 1K. Nano Banana 2 (gemini-3.1-flash-image): $0.045 per 0.5K, $0.067 per 1K, $0.151 per 4K. Nano Banana 2.1 is reported at $0.0336 per 1K. All unverified via aggregators — [benchlm](https://benchlm.ai/media-pricing/nano-banana), [renderahouse](https://www.renderahouse.com/blog/nano-banana-pricing), [curlscape](https://curlscape.com/blog/google-gemini-api-pricing-guide-2026)
- Recraft V4 Styles: $0.035/image. V4 Styles Pro: $0.10/image. V4 Styles Pro Vector (SVG): $0.12/image. Style creation costs $0.005 per request (OpenRouter reseller prices). Custom style training keeps a brand look across hundreds of generations, and the output is true vector — [OpenRouter Recraft v4 styles pro](https://openrouter.ai/recraft/recraft-v4-styles-pro), [shopot Recraft v4 styles pro vector](https://api3.shopot.ai/recraft/recraft-v4-styles-pro-vector), [tooldirectory](https://tooldirectory.ai/tools/recraft)
- Scenario: custom-model training is a one-time Compute Unit (CU) charge, "often an order of magnitude or more" above one generation. Generations then cost the standard rate, about 2–15 CU per image. `?dryRun=true` previews cost. Models are private to the org by default. The Starter plan is reportedly 1,500 credits for $15/mo; Pro is reportedly $45/mo (conflicting). The tool focuses on 2D assets and textures — [Scenario help: model costs](https://help.scenario.com/articles/8811829644-model-costs-for-asset-generation), [Scenario help: API credits](https://help.scenario.com/articles/7934059476-api-usage-and-credits-compute-units), [Scenario pricing](https://www.scenario.com/pricing), [aitoolsatlas](https://aitoolsatlas.ai/tools/scenario/review)

### Inferences
- **Recommended map (images):**
  | Job | Primary | Fallback | Cost |
  |---|---|---|---|
  | House-style "bible" (character turnarounds, palette, line weight) | GPT Image 2.5 or Nano Banana Pro, with designer iteration | FLUX.2/3 (BFL) | ~$5 one-off |
  | Icons, UI, buttons, badges (SVG) | Recraft V4 Styles Pro Vector with a custom style built from 5–10 approved images | Recraft raster, then vectorize | $0.12 per icon, ~$10–15 for 100 |
  | Scenes, story illustrations, cards | Nano Banana 2 batch (~$0.034–0.067 per 1K image) with style-reference images | GPT Image 2 / 2.5 | ~$10–25 for 300 images |
  | Avatar creator parts (hair/eyes/outfits as layers) | Draw a base body template, then use an image-editing model (Nano Banana / GPT Image edit) on a fixed canvas with "only change X". Better: **generate parts as Recraft SVG** so layers align and recolor via CSS/SwiftUI | Scenario trained style model | ~$10–20 |
  | Transparent backgrounds | Request transparency where the model supports it, or run a background-removal pass | — | negligible |
- Consistency comes from a pipeline, not a single model: a locked style guide prompt, 3–6 reference images passed on every call, a fixed seed/canvas size where supported, and an LLM or vision judge that compares new outputs with the references.
- Licensing for a private family app is low-risk with all the API vendors above. Still, store each vendor's ToS snapshot alongside the assets.

### Gaps
- Not researched: Midjourney v7/v8 API status, Ideogram API pricing, Leonardo, Layer.ai, BFL FLUX Kontext/FLUX.2 pricing, LoRA training costs, and native transparent-PNG support per model.
- Official Google, OpenAI and Recraft price pages were not fetched (reseller and aggregator figures only).

## 3. Animation and video cutscenes

### Takeaway
AI video (Veo 3.1, Sora 2, Kling 3.0) costs about $0.05–0.70 per second and has little value for a learning game for toddlers to 8-year-olds. A few 5–8 s celebratory cutscenes would cost under $10. Character animation should be done in **Rive** (rigged, state-machine-driven, tiny runtime on iOS), not video. Rive's AI features were not verified.

### Cited Findings
- Sora 2 costs about $0.10/s at 720p. Sora 2 Pro costs $0.30/s (720p) to $0.70/s (1080p). One source says **Sora 2's Videos API shuts down Sep 24, 2026**, a migration risk (unverified) — [CometAPI](https://www.cometapi.com/ai-video-api-pricing/), [modelslab](https://modelslab.com/blog/api/veo-3-1-vs-kling-3-sora-2-ai-video-api-cost-2026)
- Veo 3.1 Lite costs $0.05/s (720p) to $0.08/s (1080p). Veo 3.1 Fast costs about $0.15/s and includes native audio. Veo 3.1 Standard costs $0.40/s, or $0.60/s at 4K — [fluxnote](https://fluxnote.io/guides/ai-video-model-pricing-comparison-2026), [crazyrouter](https://crazyrouter.com/en/blog/ai-video-generation-api-pricing-comparison-2026)
- Kling 3.0's official developer API starts at about $0.084/s and rises with quality, audio and 4K — [modelslab](https://modelslab.com/blog/api/veo-3-1-vs-kling-3-sora-2-ai-video-api-cost-2026), [atlascloud](https://www.atlascloud.ai/blog/tips/cheapest-ai-video-generation-api-2026)

### Inferences
- Recommendation: use Rive for all interactive character animation (idle/blink/cheer/wrong-answer states), driven by the state machine from Swift. Use Lottie only for simple UI micro-animations. Optionally generate 3–5 cutscenes with Veo 3.1 Fast or Lite from approved key art: 5 × 8 s × $0.15 ≈ $6.
- Rigging AI art: generate characters as separated parts (head, torso, arms, eyes, mouth shapes), preferably SVG from Recraft, then import into the Rive editor for bones and mesh. This is a designer step and has no reliable AI shortcut confirmed here.

### Gaps
- Rive AI features (any 2026 "AI agent" or auto-rigging), AI Lottie generation tools, and Runway pricing were not researched. A search returned nothing on Rive.

## 4. Audio: SFX, music, voice, UI sounds

### Takeaway
Use free CC0 packs (Kenney UI Audio and Interface Sounds) as the base for UI sounds. Use ElevenLabs for custom SFX and loopable music: Eleven Music costs $0.15/min through the API, with commercial licensing on Starter and above. Avoid Suno and Udio for anything beyond private use, because they have no official API and unsettled training-data litigation.

### Cited Findings
- ElevenLabs API pricing (official page via search): Eleven Music costs $0.15 per minute. Music and SFX are billed per generation. Commercial licensing is on the Starter plan and above — [ElevenLabs API pricing](https://elevenlabs.io/pricing/api)
- One summary says Eleven Music's terms exclude film/TV/radio and **studio games** on self-serve plans, with only Enterprise allowing all commercial use. The free tier requires crediting Eleven Music. Both points are secondary-source claims — [stork.ai](https://www.stork.ai/blog/elevenlabs-music-2026), [teamday](https://www.teamday.ai/blog/best-ai-music-models-2026)
- Suno and Udio have no official public API; access goes through aggregators. Suno's free tier is non-commercial; songs made on a paid tier stay commercial after cancellation. Training-data lawsuits by major labels were still in progress as of Apr 2026 — [musicapi](https://musicapi.ai/blog/best-ai-music-api-2026), [glad-ia-tor](https://glad-ia-tor.com/blog/industry-news/sell-ai-music-suno-udio-licensing), [digitalapplied](https://www.digitalapplied.com/blog/ai-music-generation-platforms-suno-udio-elevenlabs-2026)
- Kenney UI Audio: 50 files, license Creative Commons CC0. Attribution is optional, and the assets are free for commercial use — [kenney.nl UI Audio](https://kenney.nl/assets/ui-audio), [Godot forum](https://forum.godotengine.org/t/kenneys-assets-free-and-creative-commons-cc0/36658)

### Inferences
- **Recommended map (audio):** UI clicks and pops come from Kenney CC0 (free). Custom SFX (sparkle, correct and wrong chimes, animal sounds) come from ElevenLabs SFX. Background music (3–5 loopable 60–90 s tracks plus a jingle) comes from ElevenLabs Music, about 10 min × $0.15 = $1.50 plus a Starter subscription. Suno is an acceptable fallback for private-only family use on a paid tier.
- Make loops seamless by generating 90 s, then cutting to bar boundaries and crossfading in a DAW or with ffmpeg. An agent can automate this with ffmpeg plus loudness normalization to about -16 LUFS.
- Hebrew TTS voice is outside this note's scope, but ElevenLabs is the natural vendor for it if needed.

### Gaps
- Not researched: ElevenLabs SFX v2 per-generation price, the Starter plan's monthly price, Stable Audio 2.5, Adobe Firefly audio, and freesound CC0 filtering. Whether the "studio games" exclusion applies to a private, non-distributed family app is untested; it most likely does not matter for non-commercial private use.

## 5. 3D, CC0 libraries, monthly cost and the agent-run asset pipeline

### Takeaway
3D (Meshy, Tripo) isn't needed. Kenney CC0 covers UI sounds and placeholder art. Total recurring AI cost at family scale is roughly **$15–60/month in production months and $5–15/month at steady state**. A 10-step agent pipeline with designer approval gates produces a consistent library.

### Cited Findings
- Kenney assets are CC0 with optional attribution — [kenney.nl](https://kenney.nl/assets/ui-audio)
- Price inputs: Claude [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing); Nano Banana [benchlm](https://benchlm.ai/media-pricing/nano-banana); Recraft [OpenRouter](https://openrouter.ai/recraft/recraft-v4-styles-pro); ElevenLabs [ElevenLabs API pricing](https://elevenlabs.io/pricing/api); Veo [fluxnote](https://fluxnote.io/guides/ai-video-model-pricing-comparison-2026)

### Inferences
- **Monthly cost estimate (family scale, 3 kids):**
  | Item | Production month | Steady state |
  |---|---|---|
  | Content generation (Opus 5.5 batch) + judge (Sonnet 5.5 batch) | $8–20 | $3–8 |
  | Companion (Haiku 5.5) + parent summaries + photo extraction (Gemini 3 Flash) | $1–5 | $1–5 |
  | Images (Nano Banana 2 / GPT Image + Recraft vector) | $15–40 | $0–5 |
  | Audio (ElevenLabs Starter + usage) | ~$5–10 (subscription price unverified) | $0–5 (can cancel) |
  | Video cutscenes (optional, Veo 3.1 Fast) | $0–10 one-off | $0 |
  | **Total** | **~$30–85** | **~$5–20** |
- **Asset pipeline the agent can run (designer approves at the gates):**
  1. **Style brief:** the agent drafts `style_guide.md` (palette hex codes, line weight, shape language, "rounded, flat, warm, Israeli-kid-friendly", nikud-safe typography such as Noto Sans Hebrew or Assistant, and a mirrored RTL layout).
  2. **Moodboard:** 20 candidates from GPT Image 2.5 and Nano Banana Pro. **Gate 1:** the designer picks 4–6 reference images.
  3. **Character bible:** turnarounds of the mascot and recurring characters in front, side, back and 6 expressions, generated with reference images. **Gate 2.**
  4. **Lock style:** create a Recraft custom style from the approved references for UI and icons. Optionally train a Scenario model for scenes. Save the prompt templates and references in the repo (`assets/style/`).
  5. **Manifest-driven generation:** the agent reads an `assets.yaml` listing each asset's ID, type, size, prompt variables and target format (SVG, PNG@3x, or Rive part), then runs batch generation with fixed canvas sizes.
  6. **Automatic QA:** a vision judge (Claude Sonnet 5.5 or Gemini) scores each output against the references on style match, no text artifacts, age-appropriateness and transparent background. Anything below threshold is regenerated, up to N tries.
  7. **Post-processing:** background removal, trim and pad, SVG optimization (svgo), sprite-sheet packing, Xcode asset catalog export (@2x/@3x), and dark-mode variants.
  8. **Avatar parts:** generate the parts on a shared template canvas with anchor points, verify alignment by compositing test avatars, and render a contact sheet. **Gate 3.**
  9. **Animation:** export character parts to Rive. The designer rigs them and sets the state machines (idle, cheer, think, oops). Optional Veo cutscenes are made from the approved key art. **Gate 4.**
  10. **Audio:** pull Kenney CC0 UI sounds; generate SFX and music with ElevenLabs; loop-trim and normalize to -16 LUFS with ffmpeg; play a review in a web contact page. **Gate 5.** Log each asset's provenance (model, prompt, date, license) in `assets/LICENSES.json`.
- Licensing note: in a private, non-distributed family app, every option above (Suno paid, ElevenLabs Starter, Recraft, Google/OpenAI images, CC0 packs) is low-risk. Keep the provenance log in case the app is ever published.

### Gaps
- Meshy and Tripo pricing and quality were not researched. The recommendation to skip 3D is a judgment call.
- Other CC0 libraries (OpenGameArt CC0 filter, freesound CC0, Quaternius, game-icons.net, which is CC-BY) were not verified.
