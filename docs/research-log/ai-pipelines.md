# Research log — AI, content & voice pipelines (for spec 12)

Date: 2026-11-02. Author: spec-12 writer (Claude). Per F3, this log records targeted gap research done while writing
`docs/spec/12-ai-content-voice-pipelines.md`. "Verified" = read on the primary page this session. "Secondary" = search
snippet / third-party mirror. "Blocked" = host not reachable from this sandbox (needs allow-listing to re-verify).

Blocked hosts this session: `dicta.org.il`, `elevenlabs.io`, `www.recraft.ai`, `docs.cartesia.ai`, `learn.microsoft.com`
(plus `ai.google.dev`, `openai.com` per earlier notes). Everything about those vendors below is secondary and is listed
in §12 "Re-verify at build time".

---

## 1. Anthropic structured outputs — syntax & limits (Verified)
Source: https://platform.claude.com/docs/en/build-with-claude/structured-outputs ; bundled claude-api skill (2026-10-06 cache).
- Enable with `output_config: { format: { type: "json_schema", schema } }`. `output_format` is deprecated; no beta header.
  TS SDK helper: `zodOutputFormat()` / `messages.parse()`.
- Supported: object/array/string/integer/number/boolean/null, `enum` (primitives only), `const`, `anyOf`, `allOf`
  (not with `$ref`), internal `$ref/$defs`, `default`, string formats (date-time, date, uri, uuid, email, …).
  `additionalProperties` must be `false`. Array `minItems` only 0 or 1.
- NOT supported: recursive schemas, `minimum/maximum/multipleOf`, `minLength/maxLength`, array constraints beyond minItems 0/1,
  external `$ref`, complex enums. Regex: no lookarounds, no `\b`, no backreferences, avoid large `{n,m}`.
- Complexity limits per request (summed across all strict schemas): **20 strict tools, 24 optional parameters, 16 parameters
  with union types** (`anyOf` or `["string","null"]`). Grammar compile timeout 180 s; "Schema is too complex" 400.
- Compiled grammar cached **24 h from last use**; changing `name`/`description` doesn't invalidate, structure does.
- Property ordering: required first, then optional → make every property required.
- `stop_reason: "refusal"` or `"max_tokens"` may yield non-conforming JSON → always check stop_reason before parse.
- Works with Batch (50% off), streaming, token counting. Changing `output_config.format` invalidates the prompt cache.
- Model-specific (skill): Opus 5.5 and Sonnet 5.5 reject forced `tool_choice` (`any`/`tool`) → use structured outputs;
  assistant prefill → 400; from API version `2026-10-21` any `temperature/top_p/top_k` → 400; Opus 5.5 thinking cannot be
  disabled (effort is the only control, default `medium`); Sonnet 5.5 thinking-off = `{type:"between_tools"}`;
  Haiku 5.5 effort default `medium`, refusals have **no server-side fallback**. Opus/Sonnet 5.5 support
  `fallbacks: "default"` with beta `server-side-fallback-2026-07-01`.
- Design consequence (spec §1.4): two schema layers — a strict "LLM output schema" (Claude subset, all-required,
  no nullables) and a richer zod "storage schema" validated in code.

## 2. Batch API usage & pricing (Verified)
Source: https://platform.claude.com/docs/en/build-with-claude/batch-processing ; https://platform.claude.com/docs/en/about-claude/pricing
- ≤100,000 requests or 256 MB per batch; most finish < 1 h; expire after 24 h (expired requests not billed);
  results downloadable 29 days. Results arrive in any order → key by `custom_id`.
- **Not ZDR-eligible (29-day retention).** → no child-identifying data in batch prompts (pseudonyms only).
- `max_tokens: 0` pre-warm not allowed in batch; docs recommend the **1-hour cache TTL** for shared context in batches.
- Batch prices (per MTok in/out): Opus 5.5 $2/$10; Sonnet 5.5 $1/$5; Haiku 5.5 $0.05/$0.25 (≤100K prompt).
  Discounts stack with caching.

## 3. Prompt caching (Verified)
Source: pricing page + skill `shared/prompt-caching.md`.
- Standard prices: Opus 5.5 $4 in / $5 (5m write) / $8 (1h write) / **$0.20 hit** / $20 out.
  Sonnet 5.5 $2 / $2.50 / $4 / **$0.10 hit** / $10. Haiku 5.5 (≤100K prompt) $0.10 / $0.125 / $0.20 / **$0.01 hit** / $0.50;
  >100K prompt $0.50/…/$2.50. Cache hits on Opus/Sonnet 5.5 = 0.05× input.
- Max 4 breakpoints; `cache_control: {type:"ephemeral", ttl:"1h"}`; longer-TTL entries must precede shorter ones.
- Minimum cacheable prefix on the 5.5 models: 512 tokens (skill table; "check docs before relying").
- Prefix match: tools → system → messages; any byte change invalidates. Caches are per workspace.
- `inference_geo: "us"` costs 1.1× (not needed).
- Vision (Verified, https://platform.claude.com/docs/en/build-with-claude/vision): Claude 4.7+ high-res tier = long edge
  **2,576 px / max 4,784 visual tokens**; tokens = ⌈w/28⌉×⌈h/28⌉; ≤8000×8000 px; ≤10 MB base64 per image; >20 images per
  request → ≤3000 px. Formats jpeg/png/gif/webp. Best practice: image before text; avoid heavy JPEG compression on text;
  weak on rotated / tiny (<200 px) images. Claude refuses to identify people in images.

## 4. Anthropic child-safety guidance for minor-facing products (Verified)
Sources: https://support.claude.com/en/articles/9307344-… (Guidelines for organizations serving minors);
https://support.claude.com/en/articles/15591275-child-safety-guidance-for-developers
- Organizations serving minors: additional safeguards (age verification, moderation/filtering, monitoring & reporting,
  educational resources); **implement Anthropic's child-safety system prompt "where provided"** ("we may provide");
  comply with COPPA-like laws and **state compliance publicly**; **disclose that users interact with an AI**.
  Anthropic audits; non-adoption + high violation rate → suspension.
- Developer guidance: prohibits CSAM/grooming/sexualization of minors in any context incl. fiction; developer is responsible
  for safeguards against uploads/elicitation; if users can upload images → run detection (hash-matching/classifiers,
  e.g. Thorn Safer/IWF) on own infra; logging for transparency; NCMEC reporting is a US legal duty (INHOPE for others).
- **No published text of the child-safety system prompt was found** (search + both pages). Spec reserves a slot
  `ANTHROPIC_CHILD_SAFETY_PROMPT` and ships its own safety core until/if Anthropic provides one.

## 5. Dicta Nakdan API usage/terms (Blocked — secondary only)
- dicta.org.il/developers unreachable. Nakdan = hybrid neural + lexicon diacritizer (ACL 2020; 88.23% word / 95.12% char
  accuracy, Modern Hebrew). A hosted API is referenced by papers (D-Nikud used "Dicta's API"). Endpoint shape, auth,
  rate limits and commercial/private terms **unverified**.
- Spec decision: Nakdan behind an adapter (`NikudService`), with Phonikud as the always-available fallback and as the
  cross-check. Parent open item: confirm Nakdan API terms (or contact Dicta) before relying on it in production.

## 6. Phonikud usage (Verified — GitHub README)
Source: https://github.com/phonikud/phonikud
- `pip install phonikud phonikud-onnx`; model `phonikud-1.0.int8.onnx` (HF).
  `Phonikud(path).add_diacritics(text)` → pointed text with extra marks; `phonikud.phonemize(vocalized)` → IPA
  (rule-based FST; `schema='modern'|'plain'`); `phonikud.normalize()`.
- **Non-standard marks it adds:** stress/hat'ama **U+05AB** (e.g. טח֫ינה vs טחינ֫ה), vocal shva **U+05BD**, prefix
  boundary `|` (ב|ירושלים); stress in IPA `ˈ`. Custom phonemes via `[word](/ipa/)` syntax.
  → These marks must never reach display text (spec §1.5.3).
- Limits: formal-sounding nikud, inconsistent hat'ama for same-nikud words, names/foreign words wrong, partial typo handling.
- License: code CC BY 4.0 (attribution); ILSpeech/SASPEECH datasets non-commercial (not used).
- Runtime: Python/ONNX → runs on the Actions "media runner" and in an optional container, not inside a Worker.

## 7. Azure TTS he-IL SSML (Blocked — secondary only)
- Voices: he-IL-HilaNeural (F), he-IL-AvriNeural (M) only; no Hebrew HD/Dragon HD found (prior notes).
- SSML `<break>`, `<prosody rate>` work (third-party). Whether `<phoneme>`/`<lexicon>` are honoured for he-IL: **unverified**.
- Field report (SynthBanshee PR #86, GitHub): Azure he-IL and Google Chirp can merge adjacent words; workaround = ~50 ms
  inter-word `<break>` at problem boundaries; adjacent `<break>` elements caused parser error 0x80045003 → merge them.
- Terms: TTS Code of Conduct + Microsoft Product Terms AUP; **no minors/under-18 clause found** (search); consent rules
  apply to custom/cloned voices. Re-verify.

## 8. Cartesia / MiniMax Hebrew (Blocked/secondary)
- Cartesia Sonic-3.x lists Hebrew `he`; `generation_config.speed` (0.6–1.5 on sonic-3 per LiveKit), emotion via config or
  beta SSML-like tags (`<emotion value='excited'/>`); pronunciation dictionaries on sonic-3 (syntax for Hebrew unknown).
  Terms re minors: **not checked** (docs host blocked).
- MiniMax Speech 2.8 HD/Turbo: `language_boost` lists Hebrew in one mirror; another mirror says language_boost only for
  2.6/02/01 models (conflict) → test non-streaming 2.8 with boost. `pronunciation_dict.tone[]` (plain/IPA replacements);
  `voice_setting {speed, vol, pitch}`. Terms re minors **not checked**.

## 9. ElevenLabs approval for child-directed apps (Blocked/secondary)
- No public approval process/form found. Third-party summaries of ToS: users must be 18+; privacy policy bars personal
  data of under-18s; Voice Library bans child/child-like voices; prior notes (fetched earlier) say the Prohibited Use Policy
  bans under-18-targeted apps without written approval. Contact found in FAQ: `team@elevenlabs.io` (secondary).
- Eleven Music has a separate "Music Commercial Rights" table (not retrievable); SFX commercial use requires paid plan
  (secondary). → spec keeps E8a: ElevenLabs (TTS, SFX, Music) only with written approval; draft request in spec §11.

## 10. Recraft API styles & terms (Blocked/secondary)
- Recraft's styles reference (snippet): "Styles are not supported for V4 models"; `style` and `style_id` are mutually
  exclusive; V2/V3 vector styles include "vector_illustration", "icon". Resellers (Picsart, Cloudflare Workers AI) expose
  `recraftv4-vector`, `recraftv4-pro-vector`, and a "V4 Styles" family that takes `style_id` / reference URLs.
  → conflict; spec requires a build-time probe: use whichever model accepts our custom `style_id` (V3 vector or V4 Styles).
- Ownership (Recraft docs snippet): **free-plan images are public and owned by Recraft, not licensed commercially;
  paid-plan images are owned by the user**, private, rights fixed at generation; outputs may not be used to train other AI.
  → paid plan required. Minors clause: not found.

## 11. Image-model terms for apps used by children (secondary)
- Gemini/Google GenAI: under-18 bar (prior notes, binding E8a) → no Gemini image models ("Nano Banana") — see Proposed
  change to 00 in spec §13.
- OpenAI: Under-18 API guidance (official page via search): extra safeguards, COPPA compliance, age-appropriate content,
  audits; prior notes: avoid processing under-13 personal data without ZDR. Image models listed: gpt-image-1.5/1/1-mini
  (official guide snippet) and GPT Image 2/2.5 per leaderboards. → use only for offline asset generation by the
  developer account, never with child photos/data.
- Recraft: see §10. FLUX/BFL: not researched.

## 12. Automatic item generation templates (secondary + prior notes)
- Gierl & Lai template AIG: cognitive model → item model (template with slots + constraints) → generator; one item model
  yields thousands of items (NCME Module 34 PDF snippet). SEAB Singapore: 264 math items from 56 templates.
  Sayin & Gierl 2024: template AIG + GPT-3.5 for reading items. Duolingo: template AIG limited to easy-to-model domains
  (math) → LLM hybrid for language.
- EDUMATH (ACL 2026), MATHWELL (2024) from prior notes: LLM word problems comparable for students but need validation.
- Spec: math "item models" in code (`packages/content/aig/*`), LLM only writes slot-filled narrative skins.

## 13. LLM-as-judge rubrics for educational content (secondary)
- AIME-Con 2025 (Henkel, Vanacore, Roberts): LLM scoring of children's story retells approaches human reliability with a
  well-designed rubric, worse on ambiguous items; models have systematic leniency/severity → calibrate.
- arXiv 2602.13243: multi-LLM evaluation against EQuIP rubric (9 criteria) with human validation of rationales.
- arXiv 2608.16131: ensemble judges, accept only if score range ≤ 1.
- EDM 2026 (Ehara): predicting where LLM difficulty ratings disagree with humans.
- MATHWELL criteria (prior notes): solvability, accuracy, appropriateness.
- Spec: per-criterion integer scores 1–4 with anchored descriptors; hard-fail criteria; calibration set of 40 parent-labelled
  items; judge model ≠ generator model (Sonnet judges Opus); disagreement-with-validators routes to parent.

## 14. Hebrew OCR with Claude vision (verified docs + heb-ocr repo)
- heb-ocr leaderboard (GitHub, fetched): full-page word coverage claude-opus-5 0.335 vs claude-sonnet-5 0.210 vs
  Mishkefet-v1 0.463; line CER claude-sonnet-5 0.615. VLM failure modes: repetition loops, blank outputs, overlong outputs.
  Tip: deskew and remove ruled lines before line segmentation; keep logical (not visual) order.
- Mishkefet-v1: code MIT, **weights CC-BY-NC-SA-4.0** (private family use OK, not redistributable commercially), 30.2M params,
  CPU, HF `itayinbar/Mishkefet-v1` (check ID spelling). → optional Phase-2 second pass in a container.
- No Claude-specific Hebrew prompting guidance exists; spec derives tips from the vision docs (image-first, 2,576 px,
  crop/tiling, light compression) + benchmark failure modes.

## 15. Loudness (secondary)
- Spoken word for mobile: −16 LUFS integrated, −1 dBTP (Apple podcast best practice per Auphonic); AES TD1008 cited as −18
  LUFS for speech by one source (unverified). Spec: VO −16 LUFS / −1 dBTP mono; music bed −23 LUFS; SFX peak-normalized.

## 16. SFX/music licensing fallbacks (secondary)
- Kenney CC0 (prior notes). Stable Audio Open licensing is **disputed** (community licence <$1M revenue vs non-commercial
  claims) → not used unless verified. Eleven Music/SFX: gated by E8a.

---

## Re-verify at build time (owner: developer, before first paid call)
1. Dicta Nakdan API endpoint/terms (dicta.org.il/developers).
2. Azure he-IL SSML `<phoneme>`/`<lexicon>` support; Azure terms (no minors clause).
3. Cartesia + MiniMax terms for apps used by minors; Hebrew quality in bake-off.
4. ElevenLabs written-approval path (only if parent chooses to ask).
5. Recraft model that accepts `style_id`; paid-plan ownership terms; minors clause.
6. OpenAI image terms + under-18 guidance (current text).
7. Anthropic child-safety system prompt availability (support article 9307344).
8. Phonikud/Mishkefet licences unchanged.
