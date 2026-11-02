# Voice Technologies for a Hebrew-first / English-learning Kids' App (iPhone/iPad, Israel, family-only) — state as of Nov 2026

Research method note: these are researcher notes for a report writer. Most vendor and doc pages (ai.google.dev, learn.microsoft.com, nisai.dev) could not be fetched directly from this session because the network blocked them. Findings below therefore come mostly from web-search syntheses of those pages, plus direct fetches of GitHub pages. Treat prices and dates as "verify on the vendor page before committing". Wherever I could not verify a claim, it is listed under Gaps.

---

## 1. Hebrew TTS landscape in 2026 (which engine sounds most native; nikud, stress, homographs, numbers)

### Takeaway
Six cloud engines now really support Hebrew: ElevenLabs (v3 and the new v4, Sept 2026), Google Gemini 3.8 Flash and Flash-Lite TTS, Google Chirp 3 HD he-IL, Azure (Hila and Avri only), Cartesia Sonic 3.6 and MiniMax Speech 2.8. Amazon Polly still has no Hebrew. OpenAI's gpt-4o-mini-tts is being shut down on 6 Jan 2027. No independent, rigorous Hebrew listening test from 2026 exists. The best evidence is a March 2025 snapshot plus some Israeli blog self-tests. In those, ElevenLabs v3 and Gemini come out on top, each with about 4% word errors on a round-trip test. Stress placement and nikud-free ambiguity are the main failures across all vendors. A hands-on bake-off is required.

### Cited Findings

**Prior baseline (March 2025 snapshot, still the only side-by-side Hebrew test found)**
- Ratings were: MiniMax (voice clone, T2A v2.6 Turbo with Hebrew language boost) "Best"; Edge TTS / Azure Avri (M) and Hila (F) "Good"; Gemini 2.5 Flash Preview TTS (Puck, Zephyr) "Good"; ElevenLabs `eleven_v3` with `language_code: "he"` (Rachel, Adam, Bella) "Good". `eleven_multilingual_v2` gave "unintelligible Hebrew". Chatterbox Multilingual (he) was "Poor" (generic sounding, gibberish in run 2). Resemble AI was "Poor": the stock voice Avigail was usable, but custom clones were "unintelligible without nekudot". The repo lists no updates after 22 March 2025. — [danielrosehill/Hebrew-TTS-Providers](https://github.com/danielrosehill/Hebrew-TTS-Providers)

**ElevenLabs**
- Eleven v3 reached general availability on 14 March 2026. ElevenLabs says it is slower and not meant for real time, and points to Flash v2.5 for real-time use. — [Inworld review of v3 GA](https://inworld.ai/resources/elevenlabs-v3-review)
- Eleven v4 and v4 Turbo launched on 28 Sept 2026 according to most sources (OpenRouter lists 7 Oct 2026). They cover 90+ languages, against 70+ for v3, 32 for Flash v2.5 and 29 for Multilingual v2. They use inline audio tags such as [laughs] and [whispers] and do not support SSML. The Style and Speed sliders were removed in v4, while Stability and Similarity remain. Requests are capped at 10,000 characters. Turbo claims about 100 ms median latency. — [Unite.ai](https://www.unite.ai/elevenlabs-launches-eleven-v4-with-low-latency-turbo-variant/); [ElevenLabs v4 docs](https://elevenlabs.io/docs/overview/capabilities/text-to-speech/eleven-v4); [eesel v4 explainer](https://www.eesel.ai/blog/eleven-v4)
- Hebrew (`he`) is listed among v4's languages on the ElevenLabs models page. The docs warn that "some languages are handled more fluently than others". They also say that when a cloned voice speaks a different language, v4 aims for fluent target-language speech rather than keeping the reference accent. — [ElevenLabs Models](https://elevenlabs.io/docs/overview/models); [Eleven v4 docs](https://elevenlabs.io/docs/overview/capabilities/text-to-speech/eleven-v4)
- An independent 2026 guide (as summarized by search; the page was not fetchable) calls ElevenLabs Hebrew intelligible and reasonably natural for narration. It says stress often falls on the wrong syllable in multi-syllable words, that mixed Hebrew-English text causes problems, and it rates Google's Hebrew voices as "more mature". — [NisAI TTS 2026 guide](https://nisai.dev/guides/ai-voice-tts-2026/)
- Israeli Hebrew guides from 2026 (search summary) say ElevenLabs v3 has full Hebrew with 30+ Hebrew voices. One rates its Hebrew 7–8/10 and finds male voices generally more natural than female ones. Which guide made which claim was unclear in the synthesis. — [kolbo.ai Hebrew TTS guide](https://kolbo.ai/he/blog/text-to-speech-hebrew-complete-guide); [tzedek.me ElevenLabs guide](https://tzedek.me/blog/elevenlabs-guide)
- In an Israeli self-test, bintechai "measured 4% word errors in Gemini and in ElevenLabs v3" and said Google's narration engine is free through Google AI Studio. They name nikud as "the one thing" that doesn't work well in Hebrew. — [bintechai – קול AI בעברית](https://bintechai.com/learn/ai-voice)
- Pronunciation control: ElevenLabs pronunciation-dictionary phoneme tags work only with eleven_flash_v2 and eleven_v3; a newer Agents page adds eleven_v4 and v4_turbo. IPA or CMU notation for languages other than English requires eleven_v3. v3 "natively understands IPA symbols when wrapped in forward slashes" in text, but only for individual words and not with 100% consistency. Other models use alias (respelling) rules instead. — [ElevenLabs pronunciation dictionary](https://elevenlabs.io/docs/eleven-agents/customization/voice/pronunciation-dictionary); [ElevenLabs best practices – pronunciation](https://elevenlabs.io/docs/best-practices/prompting/pronunciation)
- v3 tag craft, per the official guide: Stability is the most important setting. Use "Creative" or "Natural" for expressive tags; "Robust" dampens direction, and "Creative" can hallucinate. Use prompts longer than 250 characters, generate 2–3 takes and pick one, and match tags to a voice that suits them. SSML is ignored. Professional Voice Clones are not fully optimized for v3. — [Prompting Eleven v3](https://elevenlabs.io/docs/best-practices/prompting)
- Generic (non-Hebrew) quality rankings: a 2026 Gradium comparison ranks Eleven v3 behind Cartesia Sonic 3.6 and Inworld on blind-preference Elo. On a hard-case pronunciation test it gives Eleven v3 Conversational 65.4%, Gradium 81.0% and Cartesia 75.1%. Gradium is a competing vendor. — [Gradium alternatives 2026](https://gradium.ai/content/best-elevenlabs-alternatives-2026-tts-apis-voice-quality-price)

**Google**
- Chirp 3 HD added he-IL on 10 Nov 2025, together with bg-BG, cs-CZ, el-GR and others. The Chirp 3 HD page excludes he-IL from pause control and custom pronunciations, so Hebrew Chirp voices cannot be steered with phoneme overrides. — [Cloud TTS release notes](https://docs.cloud.google.com/text-to-speech/docs/release-notes); [Chirp 3 HD docs](https://docs.cloud.google.com/text-to-speech/docs/chirp3-hd)
- Gemini-TTS in Cloud Text-to-Speech lists Hebrew (he-IL) as Preview. — [Gemini-TTS Cloud docs](https://docs.cloud.google.com/text-to-speech/docs/gemini-tts)
- The current Gemini API TTS models are Gemini 3.8 Flash TTS (130+ languages) and Gemini 3.8 Flash-Lite TTS (100+ languages, model page dated Sept 2026, replaces gemini-3.1-flash-tts-preview). Both list Hebrew and auto-detect the input language. — [Gemini 3.8 Flash TTS model page](https://ai.google.dev/gemini-api/docs/models/gemini-3.8-flash-tts); [Gemini 3.8 Flash-Lite TTS](https://ai.google.dev/gemini-api/docs/models/gemini-3.8-flash-lite-tts)
- Features: 30 prebuilt voices, two-speaker dialogue, and Voice Design (a persistent custom voice from a text description; Preview on Google Cloud). Style, accent, pace, tone and emotion are steered by natural-language prompts. Firebase suggests a prompt structure of "audio profile + scene + director's notes". — [OpenRouter Gemini 3.8 Flash TTS](https://openrouter.ai/google/gemini-3.8-flash-tts); [Gemini Voice design docs](https://ai.google.dev/gemini-api/docs/voice-design); [Firebase AI Logic speech](https://firebase.google.com/docs/ai-logic/generate-speech); [Gemini-TTS Cloud docs](https://docs.cloud.google.com/text-to-speech/docs/gemini-tts)
- Reliability: a May 2026 forum bug reports that gemini-3.1-flash-tts-preview returned HTTP 200 with empty content and finishReason OTHER for a Hebrew prompt. The thread shows no resolution, and it concerns the older model. — [Google AI forum bug](https://discuss.ai.google.dev/t/bug-gemini-3-1-flash-tts-preview-returns-http-200-with-empty-content-and-finishreason-other-for-hebrew-tts-prompt/144297)
- No 2026 test of Israeli-accented Hebrew narration via Gemini style prompts was found. Google's 3.8 page claims regional-accent support, which is unverified for Hebrew. — [Gemini 3.8 Flash TTS model page](https://ai.google.dev/gemini-api/docs/models/gemini-3.8-flash-tts)

**Azure**
- Hebrew voices are he-IL-HilaNeural (F) and he-IL-AvriNeural (M). The Feb 2025 Microsoft blog lists them and says they "switch languages based on the text input". A third-party directory also lists only these two. No Hebrew HD or Dragon HD voice was found. Dragon HD Omni (preview) offers 500–700+ multilingual voices with styles, but Hebrew support for them is unconfirmed. — [Azure TTS Feb 2025 updates](https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/azure-ai-speech-text-to-speech-feb-2025-updates-new-hd-voices-and-more/4387263); [json2video Azure Hebrew voices](https://json2video.com/ai-voices/azure/languages/hebrew/); [Dragon HD Omni announcement](https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/introducing-dragon-hd-omni-azure-speech-new-voice-type-now-in-preview-via-micros/4481288); [Azure language support](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/language-support)
- Azure release notes mention a he-IL punctuation fix. — [Azure Speech What's new](https://docs.azure.cn/en-us/ai-services/speech-service/releasenotes)

**OpenAI**
- No Hebrew-specific quality review of gpt-4o-mini-tts exists. An older OpenAI forum thread says the earlier TTS models produced Hebrew "with a heavy American accent". A poster there found the newer realtime voices "much better" for Hebrew, which is an anecdote. — [OpenAI forum – Hebrew realtime voices](https://community.openai.com/t/real-time-voices-for-tts-api-specifically-hebrew-support/1064592)
- Deprecation: secondary reports of an OpenAI notice dated 1 Oct 2026 set 6 Jan 2027 as the shutdown date for tts-1, tts-1-hd and the dated gpt-4o-mini-tts snapshots. The named replacement is gpt-realtime-2.1-mini, which does not support `/v1/audio/speech`, so migrating means moving to the Realtime API. — [mixed-news](https://mixed-news.com/en/openai-tts-replacement-gpt-realtime-2-1-mini-audio-speech-not-supported/); [kingy.ai tracker](https://kingy.ai/ai-launch-tracker/openai-text-to-speech-model-retirement-january-6-2027/); [OpenAI TTS guide](https://developers.openai.com/api/docs/guides/text-to-speech)
- One forum report says output from the same gpt-4o-mini-tts snapshot got noticeably worse between January and February. This is anecdotal, but it is an argument for pinning versions and caching audio. — [OpenAI forum quality decrease](https://community.openai.com/t/audio-quality-decrease-in-gpt-4o-mini-tts/1373888)
- OpenAI custom voices can be created from a text description (accent, pace) or a consent recording. No Hebrew results have been reported. — [OpenAI custom voices](https://developers.openai.com/api/docs/guides/custom-voices)

**Others**
- Cartesia: Sonic 3.6 is the current model and lists Hebrew among 44 languages. The changelog says Sonic 3.5 brought a "step-change in multilingual performance, particularly Hebrew", but the same changelog also lists Hebrew among languages that did not improve, which is inconsistent. — [Cartesia Sonic 3.6 docs](https://docs.cartesia.ai/build-with-cartesia/tts-models/latest); [Cartesia changelog 2026](https://docs.cartesia.ai/changelog/2026); [Cartesia Hebrew page](https://www.cartesia.ai/languages/hebrew)
- MiniMax: Speech 2.6 HD/Turbo `language_boost` includes Hebrew (40 languages). 2.6 is deprecated in favour of Speech 2.8 HD/Turbo, which also lists Hebrew. — [AIML API MiniMax 2.6 HD](https://docs.aimlapi.com/api-references/speech-models/voice-chat/minimax/speech-2.6-hd); [MiniMax API overview](https://platform.minimax.io/docs/api-reference/api-overview)
- Amazon Polly has no Hebrew. A long-running AWS re:Post request is still open, and the Oct 2026 generative-engine update added Hindi and Finnish, not Hebrew. — [AWS re:Post – request for Hebrew](https://repost.aws/questions/QU0xlQcZ8ZQqSHvU3PqhMRnQ/request-for-hebrew); [aws-news Oct 2026](https://aws-news.com/article/2026-10-20-polly-generative-engine-upgraded-with-more-natural-speech-14-new-voices-and-2-new-languages)
- Apple on-device: Hebrew TTS (Carmit, a Nuance-derived voice) has been in AVSpeechSynthesizer since iOS 8, and early testers called it default quality. I found no info on an Enhanced or Premium Hebrew voice or Hebrew Personal Voice in iOS 26. — [Use Your Loaf – iOS 8 Hebrew](https://useyourloaf.com/blog/ios-8-adds-hebrew-speech-synthesis/)

**Open source and Israeli research**
- Phonikud (Interspeech 2026, arXiv 2506.12311) is an open-source Hebrew G2P. A nikud model, recommended via `phonikud-onnx` with an int8 ONNX file, adds vowel marks plus invented marks for vocal shva (U+05BD) and stress/hat'ama (U+05AB). Example: "טחינה" (tahini) vs "טחינ֫ה" (grinding). A rule-based FST then produces IPA and expands numbers and dates, with an English fallback for mixed text. The authors say small local TTS models with Phonikud input get close to proprietary systems. Known limits: formal-sounding nikud, inconsistent hat'ama, and errors on names and foreign words. Code is CC BY 4.0. The ILSpeech and SASPEECH datasets are non-commercial. The companion repo `phonikud-tts` exists, and Renikud is a newer, more accurate G2P. — [phonikud GitHub](https://github.com/phonikud/phonikud); [arXiv 2506.12311](https://arxiv.org/abs/2506.12311); [ReNikud arXiv 2606.20179](https://arxiv.org/pdf/2606.20179)
- ReNikud notes that standard nikud leaves lexical stress unspecified and that traditional nikud doesn't match spoken Hebrew. Leading open systems predict vowel marks before synthesis. — [ReNikud](https://arxiv.org/pdf/2606.20179)
- HebTTS (Hebrew University/Technion, Interspeech 2024) is a diacritic-free, language-model-based TTS. — [HebTTS page](https://pages.cs.huji.ac.il/adiyoss-lab/HebTTS/)
- The catalog of Hebrew AI models (updated 1 Apr 2026) says "no production-grade standalone [Hebrew TTS] model yet". It lists HebTTS, SIMS-Llama3.2-3B and SIMS-7B, israwave, Zonos-Hebrew, two Hebrew Orpheus-3B fine-tunes, mms-tts-heb and phonikud. Dicta's `dictabert-large-char-menaked` is a nikud model useful for TTS preprocessing. — [Hebrew-AI-Models catalog](https://github.com/danielrosehill/Hebrew-AI-Models)
- Zonos-Hebrew is licensed cc-by-nc-4.0, outputs 44 kHz, and has rate, pitch and emotion controls. MamreTTS (same author) claims Phonikud integration. No independent quality evaluation exists. — [mixpeek Zonos-Hebrew card](https://mixpeek.com/model/notmax123/Zonos-Hebrew); [MamreTTS listing](https://app.alphaneural.io/models/notmax123/MamreTTS)
- No source was found for "Robo-Shaul", a Dicta TTS product, or an ivrit.ai TTS model. ivrit.ai appears to be ASR-only. — [Hebrew-AI-Models catalog](https://github.com/danielrosehill/Hebrew-AI-Models)

### Inferences
- The realistic contenders for "most native Hebrew" are ElevenLabs v4 and v3, Gemini 3.8 Flash TTS, and Google Chirp 3 HD he-IL. Azure Hila/Avri is the stable, cheap baseline but the least expressive. MiniMax 2.8 and Cartesia 3.6 are worth one round in a bake-off. Open source is not production-ready for warm, kid-directed narration, but Phonikud is very useful as a preprocessing and QA tool.
- Hebrew-specific risks to test: (a) stress errors, (b) homographs in unvocalized text, (c) numbers, dates and gendered number agreement, (d) English words inside Hebrew sentences, (e) whether a vendor honours nikud when it is supplied (unverified for every vendor), and (f) whether supplying full nikud makes the speech sound formal or "biblical".
- Chirp 3 HD he-IL cannot take custom pronunciations, so the only fix for a wrong word is respelling or partial nikud. ElevenLabs offers alias dictionaries plus v3 IPA, which is the most controllable.
- Avoid OpenAI TTS for the Hebrew narration pipeline because the model is being retired. Treat any model as perishable: always cache rendered audio and keep the model ID in the cache key.

### Gaps
- No 2026 blind listening test (MOS or preference) of Hebrew across vendors exists. Only a hands-on bake-off can decide (protocol in section 7).
- It is unknown whether ElevenLabs v4, Gemini 3.8 and Chirp 3 HD use nikud and stress marks when supplied in the input.
- The full list of Chirp 3 HD he-IL voice names, and their count, was not retrieved.
- No Hebrew results were found for Hume Octave, Fish Audio, Resemble's 2026 models, Chatterbox Multilingual (newer versions) or F5-TTS Hebrew fine-tunes.
- No Israeli forum consensus was found on GeekTime, Reddit r/israel or Facebook. The Israeli material found was blogs and guides.

---

## 2. Kid-directed voice quality, child-voice policies, vendor age terms, consistency, pricing and caching

### Takeaway
Pricing is trivial at family scale; under $20/month on any engine. The real constraints are vendor policy and consistency. ElevenLabs bans child-like voices from its Voice Library. Its Prohibited Use Policy forbids apps aimed at under-18s without written approval. The Gemini API and Google Cloud generative AI terms bar use in services "directed towards or likely to be accessed by" under-18s. OpenAI allows it but discourages processing under-13 data without zero data retention. The user must consciously accept or resolve this; Azure is the cleanest option on terms found so far. For consistency, use fixed voice IDs, adult-sounding "playful" character voices, multiple takes, automated ASR QA and permanent caching.

### Cited Findings

**Child-like voices**
- ElevenLabs: children's voices, and adult voices designed to mimic children, cannot be added to the Voice Library. The Prohibited Use Policy forbids submitting recordings of anyone under 18 to train a voice, and creating a voice imitating a specific real minor. — [ElevenLabs help – child voices](https://elevenlabs.io/docs/help-center/product/voices/voice-library/can-childrens-or-child-like-voices-be-added-to-the-voice-library); [ElevenLabs Prohibited Use Policy](https://elevenlabs.io/use-policy)
- No official ElevenLabs text was found that bans child-like voices in private Voice Design. Third-party kids' vendors claim a platform-wide ban, but they have a commercial interest. — [kidsstoryteller.ai claim](https://kidsstoryteller.ai/elevenlabs-kids-voices)
- Gemini: a developer forum question about fictional child characters in Voice Design shows no official answer. One Gemini voice is described as "Leda: Youthful". — [Google AI forum – Voice Design child characters](https://discuss.ai.google.dev/t/voice-design-gemini-tts-are-fictional-child-characters-supported-minimum-age-and-approved-path/188899); [Vertex Gemini TTS docs](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/speech/text-to-speech)
- Voice Design guidance: put permanent traits (age, gender, timbre, accent, baseline cadence) in the voice description, and temporary emotion in the style or prompt. Use a clear one- or two-sentence description. — [Gemini Voice design](https://ai.google.dev/gemini-api/docs/voice-design); [AI Studio voice design](https://aistudio.google.com/docs/voice-design)

**Vendor age and audience terms (critical for a kids' app)**
- ElevenLabs Prohibited Use Policy: apps, products or experiences aimed at people under 18 are prohibited unless ElevenLabs has reviewed and approved them in writing. Account holders must be 18+. The privacy policy prohibits uploading voice data from or about anyone under 18. — [ElevenLabs Prohibited Use Policy](https://elevenlabs.io/use-policy); [ElevenLabs Privacy Policy](https://elevenlabs.io/privacy-policy)
- The Gemini API Additional Terms require users to be 18+. They bar use of the Services "as part of a website, application, or other online service that is directed towards or is likely to be accessed by individuals under the age of 18". The clause has been in place since at least Feb 2024. — [Gemini API Additional Terms](https://ai.google.dev/gemini-api/terms); [archived Feb 2024 terms](https://ai.google.dev/terms-archive/terms_02_15_24)
- Google Cloud Service Specific Terms have a similar bar for "Generative AI Services", which a forum post cites as §20(d). A Google developer-forum reply (not official policy) says the restriction applies via Vertex AI too. Whether Cloud TTS or Chirp 3 HD counts as a "Generative AI Service" was not confirmed; the list is at cloud.google.com/terms/services. — [Google Cloud Service Specific Terms](https://cloud.google.com/terms/service-terms); [Google dev forum](https://discuss.google.dev/t/question-regarding-age-restrictions-for-the-gemini-api/188060)
- Developers have asked Google directly whether static TTS audio, generated offline by an adult and shipped inside a children's app, is allowed. No official answer was found. — [forum: static TTS audio by an adult](https://discuss.ai.google.dev/t/gemini-api-age-restriction-may-static-tts-audio-generated-offline-by-an-adult-ship-inside-a-childrens-learning-app/186036); [forum: pre-generated TTS for under-18 audience](https://discuss.ai.google.dev/t/gemini-api-terms-pre-generated-tts-audio-in-an-app-whose-audience-includes-under-18s/196200)
- OpenAI: developers serving minors must follow COPPA and similar laws and add under-18 safeguards. Processing personal data of children under 13 is discouraged unless zero data retention is implemented first. Developers are expected to use OpenAI's safety tools and current flagship models. The Terms of Use require users to be 13+, and under-18s need parental permission. — [OpenAI under-18 API guidance](https://developers.openai.com/api/docs/guides/safety-checks/under-18-api-guidance); [OpenAI Terms of Use](https://openai.com/policies/row-terms-of-use/)

**Pricing**
- ElevenLabs API (sources checked Oct–Nov 2026): Eleven v3 and Multilingual v2 cost $0.08 per 1k characters; Flash/Turbo and v3 Conversational cost $0.04. v4 is reportedly $0.08 and v4 Turbo $0.04. Mid-2026 sources gave $0.10 and $0.05, which suggests a price drop. Subscriptions use credits at about one credit per character, roughly 1,000 characters per minute of audio. — [stork.ai](https://www.stork.ai/blog/elevenlabs-api-key-and-pricing-2026); [websitevoice.com](https://websitevoice.com/blog/elevenlabs-pricing-plans/); [goodvibecode (older rates)](https://www.goodvibecode.com/text-to-speech/elevenlabs-api-pricing-explained); [Cartesia's ElevenLabs pricing explainer](https://www.cartesia.ai/learn/elevenlabs-pricing)
- Gemini 3.8 Flash TTS: $0.50 per 1M input tokens and $9 per 1M audio output tokens. Google's table shows $1 and $18 from 1 Jan 2027, when introductory rates end. Audio is 25 tokens per second, about $0.81 per hour of speech at $9. Batch/Flex is $0.25 and $4.50. Flash-Lite TTS is $0.50 in and $6 out through 31 Dec 2026. The free tier covers both TTS models. Treat the 2027 doubling as needing verification. — [eesel Gemini 3.8 TTS pricing](https://www.eesel.ai/blog/gemini-3-8-flash-tts-pricing); [Gemini API pricing](https://ai.google.dev/gemini-api/docs/pricing); [cloudprice](https://cloudprice.net/models/google-gemini-3-8-flash-tts)
- Google Chirp 3 HD costs $30 per 1M characters. Aggregators report the first 1M characters per month free, but this conflicts with other aggregator pages and was not verified on Google's page. — [Google TTS pricing](https://cloud.google.com/text-to-speech/pricing); [costbench free plan](https://costbench.com/software/ai-voice-tools/google-cloud-text-to-speech/free-plan/)
- Azure neural TTS is about $15 per 1M characters, and Neural HD about $22 per 1M (reported cut in March 2026). The free tier is reportedly 0.5M neural characters per month. Custom Neural Voice is about $24 per 1M. These are secondary sources. — [beri.net comparison](https://www.beri.net/article/elevenlabs-vs-cartesia-vs-azure-tts-latency-pricing-voice-cloning-2026); [Azure speech pricing](https://azure.microsoft.com/en-us/pricing/details/cognitive-services/speech-services/)

**Hebrew-specific production tips found**
- Hebrew guides note that TTS engines get numbers, names and tone wrong. They recommend rephrasing items like "ה-AI" so they read naturally. — [shivuknet Hebrew AI voiceover 2026](https://shivuknet.co.il/blog/hebrew-ai-voiceover/); [bintechai](https://bintechai.com/learn/ai-voice)
- Google suggests writing natural spoken transcripts, including disfluencies, for more natural Gemini TTS output. — [Gemini speech generation docs](https://ai.google.dev/gemini-api/docs/speech-generation)

### Inferences
- **Cost at 200k characters per month** (Hebrew narration, assuming about 900–1,000 characters per minute, so about 3.3–3.7 hours of audio):
  - ElevenLabs v3 or v4: about $16 per month at $0.08 per 1k. Generating 2–3 takes per line raises this to about $32–48.
  - ElevenLabs Flash or v4 Turbo: about $8.
  - Gemini 3.8 Flash TTS: about $3 at the introductory rate, about $6 from 2027, or about $1.50 via Batch.
  - Chirp 3 HD: $6, or $0 if the 1M free tier holds.
  - Azure neural: $3, or $0 inside the free tier. Azure HD would be about $4.40, but there is no Hebrew HD voice.
  - Cost will not decide the vendor; quality and terms will.
- Nikud characters are Unicode combining marks and probably count as billable characters, which may raise character counts by roughly 30–60%. This is unverified; see Gaps.
- Terms strategy for a family-only app:
  - Option A: use Azure, which had no comparable under-18 clause in my research, though this was not exhaustively verified. Also use Chirp 3 HD if Cloud TTS is not a "Generative AI Service".
  - Option B: ask ElevenLabs for written approval, which is a realistic request for a tiny family app.
  - Option C: the parent, as the adult account holder, pre-generates static audio offline and bundles files only. No API calls happen from the child's device. This lowers exposure but does not clearly satisfy the "likely to be accessed by under-18" wording. The user should make this decision knowingly; the report should not present it as settled.
- Kid-directed narration production process (recommended):
  1. **Voice bible.** Set 1–2 narrators (warm adult female and male) and 3–5 characters. Use adult voice actors' timbre with playful delivery rather than child-like voices, which respects ElevenLabs policy. Fix the voice IDs, model version, stability and speed for each.
  2. **Script rules for 2.5–5.5-year-olds.** Short sentences, one idea per sentence, repetition, exclamations, and explicit pauses through punctuation or line breaks. Use a slower rate for the 2.5-year-old track.
  3. **Hebrew normalization pass.** Write numbers as gender-agreed words. Add nikud only to ambiguous words, using Phonikud or Dicta Nakdan as a suggestion and a parent to verify. Add hat'ama where needed. Keep a family glossary of names and ambiguous words, mapped to ElevenLabs alias rules.
  4. **Generate 2–3 takes** with tags such as [excited], [whispers] and [laughs], used sparingly (at most two per line).
  5. **Automated QA.** Transcribe each take with ivrit.ai Whisper and diff it against the script, which catches hallucinated or dropped words. Phonikud's IPA output can flag stress mismatches. Keep the take with the lowest error.
  6. **Parent listening pass**, with a three-way verdict: OK, re-roll or fix the text.
  7. **Normalize loudness**, trim silences and export AAC.
  8. **Cache permanently**, keyed by hash(model + voice + settings + normalized text). Ship the files in the app bundle or iCloud/CDN so playback is offline and needs no API calls at runtime.
- Batch rendering, such as Gemini Batch at half price or ElevenLabs Projects/Studio, suits this offline workflow.

### Gaps
- Whether ElevenLabs Voice Design may produce child-like voices for private, non-Library use is unclear.
- Whether Google Cloud Chirp 3 HD falls under the Generative AI Services under-18 bar was not confirmed.
- Azure's terms were not checked for a minors clause, and no Azure OpenAI or Speech minors restriction surfaced.
- Whether nikud marks count toward ElevenLabs or Google billed characters was not found.
- ElevenLabs subscription plan credit amounts were not verified.
- The kidsstoryteller.ai claim to sell "real child voices" via API was not evaluated, and is likely English-centric.

---

## 3. English TTS for teaching: native accents, slow and clear speech, isolated phonics sounds, minimal pairs

### Takeaway
Whole words, sentences and minimal pairs in US or UK English are solved by any top engine. Azure en-US/en-GB, Chirp 3 HD en-US/en-GB, ElevenLabs and Gemini all have native voices, and IPA phoneme tags can force word pronunciations. Isolated phonics sounds such as /b/ with no "buh", or /æ/ alone, are not reliably producible by TTS: SSML phoneme tags are designed per word, and consonant-only strings fail. Use human recordings for the about 44 phonemes, from a native-English speaker or a licensed or CC library. Use TTS for everything else.

### Cited Findings
- Google's SSML `<phoneme>` applies to a single word. Yandex states that its SSML phoneme markup "is not intended for generating separate sounds". Microsoft's legacy docs say an unrecognized phone makes the engine reject the whole SSML document. — [Google SSML reference](https://developers.google.com/assistant/conversational/ssml-beta); [Yandex SpeechKit SSML](https://yandex.cloud/en-ru/docs/speechkit/tts/markup/ssml); [Microsoft legacy SSML phoneme](https://learn.microsoft.com/en-us/previous-versions/office/developer/speech-technologies/hh378516(v=office.14))
- Practitioners report that TTS "works best with full words" and that sounds like "sh" fail because there is no vowel. — [Articulate community thread](https://community.articulate.com/discussions/discuss/how-to-get-text-to-speech-to-pronounce-phonetics-and-all-the-individual-letters-/933907)
- On-device engines such as Piper do not use SSML. — [Picovoice SSML guide 2026](https://picovoice.ai/blog/ssml-text-to-speech/)
- ElevenLabs v3 accepts IPA between slashes for individual words, not 100% consistently. — [ElevenLabs pronunciation best practices](https://elevenlabs.io/docs/best-practices/prompting/pronunciation)
- Open phoneme audio: Wikimedia Commons IPA recordings are reusable, mostly under CC BY-SA 3.0 or 4.0, which requires attribution and share-alike. An MIT-licensed project bundles 44 English phonemes using them (35 of 36 files CC BY-SA 3.0, one public domain). The IPA Handbook audio is for personal use only and cannot be put in products without permission. — [ipa-sound-lab](https://github.com/Yun-lab7/ipa-sound-lab); [Wikimedia Commons example](https://commons.wikimedia.org/wiki/File:%C9%A4%CC%9E_IPA_sound.opus); [IPA Handbook downloads](https://www.internationalphoneticassociation.org/content/ipa-handbook-downloads)

### Inferences
- **Recommended phonics audio plan.**
  - Primary: commission one recording session with a native English speaker, US or UK to match the chosen curriculum. Record the about 44 phonemes as pure sounds in a synthetic-phonics style, plus common blends and digraphs, each twice: "normal" and "stretched" for continuants (/sss/, /mmm/, /fff/). For stops (/b/, /t/, /k/), record a clipped burst with minimal schwa.
  - Fallback: Wikimedia IPA clips. These are adult phoneticians with generic IPA qualities, not always English allophones, and they carry CC BY-SA attribution duties. That is acceptable for a private family app, but the sound is not kid-friendly.
  - Last resort: trim continuants out of TTS words. This works for /s/, /m/, /f/ and /ʃ/, but not for stops.
- For minimal pairs (ship/sheep, bat/bet, think/sink) and decodable words, use TTS with IPA phoneme tags where available (Azure SSML `phoneme alphabet="ipa"` for en-US, or ElevenLabs v3 IPA in slashes) and verify by ear. Pick one accent (US or UK) and keep it consistent across all English audio.
- For "slow, clear" teaching speech: ElevenLabs v4 removed the Speed slider, so slow delivery must come from the text (punctuation, tags) or from post-processing time-stretch. Azure SSML `prosody rate` and Chirp 3 HD speaking-rate controls are more deterministic for slow English.
- A parent who is a non-native English speaker should not be the phoneme source, but could be cloned for warm English encouragement lines if desired. See section 6.

### Gaps
- No commercial licensed phonics audio library with clear app-embedding licence terms was found. Jolly Phonics and others were not researched successfully.
- No TTS engine was found that reliably produces pure isolated phonemes.
- Azure en-US IPA phoneme support was not re-verified for 2026 HD and Dragon voices; it is assumed from the long-standing SSML feature.

---

## 4. Speech recognition: Hebrew (incl. the 8.5-year-old reading aloud), children's English, pronunciation assessment, reading tracking

### Takeaway
For Hebrew, the open ivrit.ai Whisper large-v3-turbo fine-tune is the strongest verifiable option. It cuts word errors by about 37–50% versus stock Whisper on FLEURS, is Apache-2.0, and has quantized GGUF builds that can run on device. ElevenLabs Scribe and Soniox claim lower Hebrew WER, but only in their own tests. Children's speech is much harder, with WER 70–76% for preschoolers and 13–17% for grades 3–5 with Whisper. Reading-aloud tracking should therefore be done as alignment against the known text, not open transcription. For English pronunciation, Azure Pronunciation Assessment (en-US, phoneme-level, miscue detection) is the default. Speechace is the alternative. Hebrew pronunciation assessment as a service was not found. SoapBox Labs is no longer available to third parties.

### Cited Findings

**Hebrew ASR**
- ivrit.ai whisper-large-v3-turbo: on the full FLEURS Hebrew test (792 utterances), a Q8_0 GGUF conversion scores 18.46% WER (95% CI 16.94–20.02) against 29.51% for stock turbo. A PR dated 1 Nov 2026 reproduces 18.45% against 29.69%. A 40-utterance whisper.cpp test found 14.37%, or about 14.8% after quantizing to 452 MB. The model is Apache-2.0, trained on ivrit.ai crowd-transcribe, crowd-recital and Knesset data. — [adolan GGUF card](https://huggingface.co/adolan/ivrit-whisper-large-v3-turbo-gguf); [hex PR #755](https://github.com/anomalyco/hex/pull/755); [Handy discussion #1975](https://github.com/cjpais/Handy/discussions/1975); [ivrit-ai model card](https://huggingface.co/ivrit-ai/whisper-large-v3-turbo/blob/main/README.md)
- ivrit.ai's Interspeech 2025 paper describes an open-source, state-of-the-art Hebrew Whisper built through crowdsourcing, with a new Hebrew test set. — [Marmor et al., Interspeech 2025](https://www.isca-archive.org/interspeech_2025/marmor25_interspeech.pdf)
- Related models: `whisper-heb-ipa` outputs Hebrew IPA phonemes, and `WhiStress` is a stress- and prosody-aware Hebrew ASR. — [Hebrew-AI-Models catalog](https://github.com/danielrosehill/Hebrew-AI-Models)
- ElevenLabs Scribe (vendor figures): the Hebrew page claims 3.1% WER on FLEURS and 5.5% on Common Voice, model version unstated. The older Scribe v1 table shows 15.2% for Scribe against 20.3% for Gemini Flash 2 and 32.9% for Whisper large-v3. Hebrew sits in ElevenLabs' "Good" tier (>10–20% WER). Diarization is batch-only. — [ElevenLabs Hebrew STT](https://elevenlabs.io/speech-to-text/hebrew); [Soniox vs ElevenLabs Hebrew](https://soniox.com/compare-stt/soniox-vs-elevenlabs/hebrew)
- Soniox (vendor claims): 1.25% WER against 3.24% for OpenAI on its 2026 real-world Hebrew benchmark, and 7.5% Hebrew WER in a 2025 60-language study. Price is about $0.10 per hour async and $0.12 per hour streaming, with real-time Hebrew streaming. — [Soniox vs OpenAI Hebrew](https://soniox.com/compare-stt/soniox-vs-openai/hebrew); [Soniox vs Cartesia Hebrew](https://soniox.com/compare-stt/soniox-vs-cartesia/hebrew)
- Per Soniox's comparison page, gpt-4o-mini-transcribe costs about $0.18 per hour and gpt-4o-transcribe about $0.36 per hour. No independent Hebrew WER exists for either model or for Google Chirp 3. — [Soniox vs OpenAI Hebrew](https://soniox.com/compare-stt/soniox-vs-openai/hebrew)
- OpenAI's Whisper and GPT-4o transcription models also face a deprecation on 26 Feb 2027 (secondary report). — [mixed-news / dev.to deprecations](https://dev.to/max_bob/openai-api-deprecations-2026-2027-every-shutdown-date-and-recommended-replacement-gpt-4o-2ffm)
- Google Chirp 3 (STT) uses the code `iw-IL` for Hebrew. Pricing is about $0.016 per minute for the first 500k minutes, and dynamic batch about $0.004 per minute. — [Chirp 3 STT docs](https://docs.cloud.google.com/speech-to-text/docs/models/chirp-3); [Google STT pricing (es)](https://cloud.google.com/speech-to-text/pricing?hl=es-419); [costbench](https://costbench.com/software/ai-transcription-apis/google-speech-to-text)
- Azure STT lists he-IL, including fast transcription and custom speech. he-IL was added as a speech-to-text locale in a March 2025 release. — [Azure language support](https://learn.microsoft.com/azure/ai-services/speech-service/language-support?tabs=stt); [Azure release notes](https://docs.azure.cn/en-us/ai-services/speech-service/releasenotes)
- Apple SpeechAnalyzer/SpeechTranscriber (iOS 26, on device): a third-party post from August 2026 says `SpeechTranscriber.supportedLocales` returns 42 locales across 22 languages including Hebrew. Other sources say about 30 at launch. Check `supportedLocales`, `installedLocales` and `isAvailable` at runtime, and fall back to `DictationTranscriber`. — [LoroNote](https://loronote.com/en/blog/apple-speechanalyzer-vs-whisper); [onestack SpeechTranscriber ref](https://onestack.dev/ios-reference/Speech/SpeechTranscriber)

**Children's speech**
- DrivenData (Oct 2026): Whisper WER on child speech runs from 13–17% for grades 3–5 to 70–76% for toddlers and preschoolers. Text normalization changes WER substantially. — [DrivenData child ASR benchmarking](https://drivendata.co/blog/child-asr-benchmarking)
- Whisper-largeV3 zero-shot WER is 12.3 on MyST and 14.9 on OGI kids' corpora. Prompting Whisper and post-processing with an LLM cut child read-speech WER from 9.4% to 5.1% (Interspeech 2025). — [Benchmarking children's ASR 2024](https://arxiv.org/pdf/2406.10507); [Gao et al. Interspeech 2025](https://www.isca-archive.org/interspeech_2025/gao25c_interspeech.pdf)
- SoapBox Labs was acquired by Curriculum Associates in late 2023 and became an internal "AI Labs" incubator. Its partnerships with other providers ended, and it is being integrated into i-Ready. Its engine is described as recognizing children's speech from age 2. No 2026 third-party API access was found. — [Voicebot.ai](https://voicebot.ai/2023/11/29/voice-ai-for-kids-startup-soapbox-labs-acquired-by-edtech-firm-curriculum-associates/); [Curriculum Associates Apr 2025 release](https://www.curriculumassociates.com/about/press-releases/2025/04/digital-promise-certification); [dealroom](https://app.dealroom.co/companies/soapbox_labs)

**Pronunciation assessment and reading tracking**
- Azure Pronunciation Assessment returns word and phoneme results. Syllable groups are supported only for en-US with IPA/SAPI. en-GB is still preview and returns empty phoneme labels. Release notes cite 24 GA languages plus 7 in preview; the counts conflict between notes. Hebrew was not confirmed. — [Azure how-to pronunciation assessment](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/how-to-pronunciation-assessment); [Q&A en-GB preview](https://learn.microsoft.com/en-us/answers/questions/5966025/when-will-azure-pronunciation-assessment-for-en-gb); [Azure Speech release notes](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/releasenotes)
- Miscue detection needs `EnableMiscue=true`. Word tags include Omission, Insertion, Mispronunciation, UnexpectedBreak, MissingBreak and Monotone. Microsoft's transparency note says grading "for children's learning might not be as strict" as for adults. Developers report that streaming mode may miss omissions and insertions, that inserted words are sometimes not flagged, and that substitutions need custom comparison. Teams Reading Progress uses the same engine. — [Azure PA transparency note](https://learn.microsoft.com/en-us/azure/foundry/responsible-ai/speech-service/pronunciation-assessment/transparency-note-pronunciation-assessment); [Q&A streaming miscues](https://learn.microsoft.com/en-au/answers/questions/2157732/how-to-get-omissions-and-insertions-from-scripted); [Q&A word/phoneme inaccuracies](https://learn.microsoft.com/en-us/answers/questions/5707709/azure-speech-pronunciation-assessment-word-and-pho)
- Speechace returns utterance, word, syllable and phoneme scores (0–100 plus CEFR) and can score custom phoneme sequences. A third-party listing shows a Basic plan at about $40 per month, a 30-second audio cap, and English (US and UK), French and Spanish. — [Speechace API docs](https://api-docs.speechace.com/features); [fitgap listing](https://us.fitgap.com/products/speechace-api)

### Inferences
- **Hebrew reading-aloud tracker for the 8.5-year-old.** Treat it as a "known text" task:
  1. Run ivrit.ai Whisper turbo, on device via whisper.cpp or CoreML-converted GGUF, or on a home Mac. Prompt it with the expected passage.
  2. Align the output to the reference with nikud-insensitive, normalized fuzzy matching. This flags skipped and substituted words and gives words-correct-per-minute.
  3. Optionally use `whisper-heb-ipa` for phoneme-level decoding feedback.
  4. Use a cloud fallback (Scribe v2, Soniox or Google Chirp 3) when on-device accuracy is insufficient.
  5. Apple SpeechTranscriber can give free, live, low-latency "follow the finger" highlighting if Hebrew is installed on the iPad.
- An 8.5-year-old reading prepared text should land near the grades 3–5 regime (about 13–17% open WER, lower with reference prompting). That is usable for progress tracking, but not for harsh word-by-word judgments. The UX should be forgiving, and parent review should handle disputes.
- Do not use ASR for the 2.5-year-old at all; error rates of 70–76% make it unworkable. For the 5.5-year-old, use only constrained tasks ("say one of these 3 words") with lenient scoring.
- **English pronunciation for the 5.5- and 8.5-year-olds.** Azure PA en-US with a reference text gives phoneme scores, and scoring should be lenient (show "try again" rather than numbers). Use single-shot rather than streaming when miscues matter. Speechace is the fallback.
- **Cost estimate.** At 300 min/month of reading aloud (10 min/day): Google Chirp 3 about $4.80, Soniox about $0.60, gpt-4o-transcribe about $1.80, ivrit.ai on device $0.

### Gaps
- Azure Pronunciation Assessment pricing, and whether he-IL is supported, were not verified on the official page.
- There is no independent Hebrew WER for Scribe v2, Soniox, gpt-4o-transcribe, Google Chirp 3 or Apple SpeechTranscriber, and no Hebrew children's speech benchmark at all.
- Deepgram Nova-3 Hebrew support was not found in searches.
- ivrit.ai's official leaderboard numbers could not be fetched.
- An ELSA Speak API for developers was not researched successfully.

---

## 5. Real-time conversational voice (Socratic companion, English conversation practice)

### Takeaway
Technically, Hebrew live voice is available and cheap: Gemini Live lists Hebrew, OpenAI gpt-realtime-2.1 and -mini are multilingual, and ElevenLabs Agents run at about $0.08/min. For this family, the case against it is strong. Gemini's terms bar under-18 audiences, ElevenLabs needs written approval, and OpenAI discourages under-13 data without zero data retention. Common Sense Media advises against voice-based AI companions for ages 5 and under and "extreme caution" for ages 6–12. Recommendation: no open-ended live voice for the 2.5- and 5.5-year-olds. For the 8.5-year-old, at most a parent-present, time-boxed, turn-based English practice mode.

### Cited Findings
- OpenAI Realtime pricing per 1M audio tokens: gpt-realtime-2.1 costs $32 in, $64 out and $0.40 cached; gpt-realtime-2.1-mini costs $10 in, $20 out and $0.30 cached. Third-party estimates put a realistic conversation at about $0.05–0.15 per minute on 2.1 and $0.02–0.05 on mini. gpt-realtime-2 launched 7 May 2026; the 2.1 date conflicts between July and September 2026. — [OpenAI pricing](https://developers.openai.com/api/docs/pricing); [layer3labs per-minute math](https://www.layer3labs.io/guides/openai-realtime-api-pricing); [forasoft](https://www.forasoft.com/blog/article/openai-realtime-api-pricing)
- No Hebrew-specific evaluation of gpt-realtime exists. A Microsoft Q&A answer says Realtime voices' accent is "often neutralized… US-leaning". A forum thread reports that gpt-realtime-1.5 regressed in accent range and expressiveness compared with v1. — [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/5698273/gpt-realtime-api-how-to-specify-the-voice); [OpenAI forum regression](https://community.openai.com/t/gpt-realtime-1-5-major-regression-in-voice-expressiveness-and-accent-quality/1377222)
- Gemini Live API: the capabilities guide lists Hebrew (`he`) among 99 languages (other sources say 70, 97 or 24). Native-audio models pick the language automatically. Third-party pricing is about $3 per 1M audio input tokens (about $0.005/min) and $12 per 1M output tokens (about $0.018/min). Gemini 3.5 Live Translate also lists Hebrew at about $0.037/min. — [Live API capabilities](https://ai.google.dev/gemini-api/docs/live-api/capabilities); [developersdigest](https://www.developersdigest.tech/voice-agents/gemini-live-api); [Gemini 3.5 Live Translate](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/gemini/3-5-live-translate)
- ElevenLabs Agents cost about $0.08/min on all tiers ($0.16 burst), with LLM costs billed separately; one review says 70+ languages. Agent speech can use v4 Turbo (about 100 ms) and Scribe v2 Realtime (90+ languages). No Hebrew agent quality reports were found. — [cloudzero](https://www.cloudzero.com/blog/elevenlabs-pricing/); [a8gent](https://a8gent.com/platforms/elevenlabs-agents); [Scribe v2 Realtime in Agents](https://elevenlabs.io/blog/scribe-v2-realtime-in-elevenlabs-agents)
- Child safety: Common Sense Media (Jan 2026) advises parents to avoid voice-based AI toys for ages 5 and under and to use extreme caution for ages 6–12. It cites recognition failures, inappropriate activations, inaccurate answers, bonding mechanisms designed to create companion relationships, and collection of voice, transcript and behavioural data. — [Common Sense Media press release](https://www.commonsensemedia.org/press-releases/common-sense-media-warns-against-ai-toy-companions-after-research-reveals-safety-risks); [KQED](https://www.kqed.org/news/12070850/steer-clear-of-ai-companion-toys-for-kids-another-advocacy-group-warns)
- UNICEF's 2026 policy brief "When AI becomes a friend" calls for guidance on AI companions for children. The Children and Screens Evidence Council voted 7–2 (Feb 2026) for parental consent for under-18 use of AI companions. A Stanford Medicine psychiatrist argues that friend-like chatbots should not be used by children and teens. — [UNICEF brief](https://www.unicef.org/media/181131/file/UNICEF-When-AI-becomes-friend-policy-brief-2026.pdf); [Children and Screens](https://www.childrenandscreens.org/newsroom/news/q-should-parental-permission-be-required-for-minors-to-use-ai-companions/); [Stanford Medicine](https://med.stanford.edu/news/insights/2025/08/ai-chatbots-kids-teens-artificial-intelligence.html)
- Vendor terms: see section 2, covering the Gemini under-18 bar, ElevenLabs written approval and OpenAI's under-18 guidance and zero data retention. — [Gemini API terms](https://ai.google.dev/gemini-api/terms); [ElevenLabs use policy](https://elevenlabs.io/use-policy); [OpenAI under-18 guidance](https://developers.openai.com/api/docs/guides/safety-checks/under-18-api-guidance)

### Inferences
- **Preferred architecture if live voice is used at all.** Use a cascaded, turn-based pipeline instead of speech-to-speech:
  1. Push-to-talk input.
  2. Apple SpeechTranscriber or a cloud STT.
  3. An LLM text turn with a strict system prompt, a topic whitelist and a moderation check on both input and output.
  4. TTS output, preferring pre-cached phrases plus streaming TTS for novel lines.
  5. Logging that the parent can review.
  
  This gives a text checkpoint for safety, lets the cached "house voice" stay consistent, and avoids companion-like, always-listening behaviour. Latency of about 1–2 seconds is acceptable for kids' turn-taking practice.
- If speech-to-speech is wanted for the 8.5-year-old's English practice, OpenAI gpt-realtime-2.1-mini is the most terms-compatible of the three, given zero data retention, a parent account and safeguards. It costs about $1.20–3 per month for 60 minutes. Gemini Live is excluded by its terms; ElevenLabs Agents need written approval.
- Design rules: time-box sessions (e.g., 10 minutes), keep the persona tutor-like rather than "friend", give no memory of personal disclosures, have the parent present, use no wake word or always-on mic, and offer a visible "stop" control.

### Gaps
- No Hebrew conversational-quality evaluation exists for any live API. This needs a bake-off (section 7).
- Official Gemini Live pricing was not verified.
- OpenAI's "GPT-Live-1" product (about $0.05/min, per one third-party source) was not verified.
- Whether OpenAI's zero data retention is available to an individual or small account was not checked.

---

## 6. Parent voice cloning (Hebrew quality, consent)

### Takeaway
A parent can legitimately clone their own voice. ElevenLabs Professional Voice Clone allows only your own voice and requires spoken verification; Instant Voice Clone needs a consent attestation. Children's voices cannot be cloned on ElevenLabs. There are no published Hebrew clone-quality data for 2026. The March 2025 snapshot rated MiniMax clones "Best" for Hebrew, and noted that Resemble clones without nikud were unintelligible. Test the parent's Instant Voice Clone on v4 and v3 for Hebrew, and on v4 for English, in the bake-off.

### Cited Findings
- ElevenLabs Instant Voice Clone requires a consent checkbox. Professional Voice Clone requires reading verification lines aloud and is limited to your own voice "even with their consent". Failed verification brings a 24-hour wait. The docs recommend matching the training recording setup during verification and reading each line once. — [ElevenLabs IVC vs PVC](https://elevenlabs.io/blog/instant-vs-professional-voice-cloning); [ElevenLabs voice cloning docs](https://elevenlabs.io/docs/eleven-creative/voices/voice-cloning); [stork.ai cloning guide](https://www.stork.ai/blog/elevenlabs-voice-cloning-guide)
- ElevenLabs prohibits voice recordings of under-18s for training, and voices imitating a specific real minor. — [ElevenLabs Prohibited Use Policy](https://elevenlabs.io/use-policy)
- v4 aims for fluent target-language speech for cross-language clones rather than carrying the reference accent. This is relevant to a Hebrew-native parent's clone speaking English. Professional Voice Clones are "not fully optimized" for v3. — [Eleven v4 docs](https://elevenlabs.io/docs/overview/capabilities/text-to-speech/eleven-v4); [Prompting Eleven v3](https://elevenlabs.io/docs/best-practices/prompting)
- Hebrew clone results from March 2025: MiniMax clones via Replicate with the Hebrew boost were rated "Best" and natural sounding. Resemble custom clones were unintelligible without nekudot. A Chatterbox Multilingual clone was poor. — [Hebrew-TTS-Providers](https://github.com/danielrosehill/Hebrew-TTS-Providers)
- OpenAI also supports custom voices from a consent recording. — [OpenAI custom voices](https://developers.openai.com/api/docs/guides/custom-voices)

### Inferences
- **Use cases worth cloning for:**
  - Short, personal encouragement lines ("כל הכבוד, נועה!").
  - Bedtime-style story intros.
  - "Abba/Ima reads" moments.
  
  Keep core narration on a stable professional voice, since clones vary more.
- **Recording process.**
  - Record 2–5 minutes in a quiet room with a phone mic for an Instant Voice Clone, or 30+ minutes for a Professional Voice Clone.
  - Speak warm, kid-directed Hebrew, so the clone learns the target register.
  - Keep consent documented, which is trivial here because the parent is the account holder.
- Kids' own voices must never be uploaded to ElevenLabs, and should not be uploaded to any vendor.
- Apple Personal Voice (on device) would be the most private option, but Hebrew support is unknown.

### Gaps
- There is no 2026 Hebrew voice-clone quality comparison: ElevenLabs IVC/PVC on v4 vs MiniMax 2.8 vs Cartesia vs OpenAI custom voices.
- The ElevenLabs Professional Voice Clone language list, and whether it includes Hebrew, was unresolved; sources say 39 or 70+ languages.
- Apple Personal Voice Hebrew support is unknown.

---

## 7. Recommended stack per job (primary and fallback), cost at about 200k characters/month, and the bake-off protocol

### Takeaway
Pre-render nearly everything offline, by a parent account, cached forever, and use live AI voice minimally. Shortlist ElevenLabs v4 (with v3 as an alternate) and Gemini 3.8 Flash TTS for expressive Hebrew. Keep Google Chirp 3 HD he-IL and Azure Hila/Avri as cheap, terms-safer fallbacks. Use human recordings for English phonemes. Use ivrit.ai Whisper on device for Hebrew reading-aloud and Azure Pronunciation Assessment for English. Total voice cost should be about $10–40 per month. The final Hebrew TTS choice must come from a listening bake-off, and the terms question must be decided consciously by the user.

### Cited Findings
- The cost inputs are ElevenLabs at $0.08 and $0.04 per 1k characters; Gemini 3.8 Flash TTS at $9 per 1M audio tokens, 25 tokens per second, about $0.81/hour, possibly $18 from 2027; Chirp 3 HD at $30 per 1M with a possible 1M free; and Azure neural at $15 per 1M with 0.5M free. — [stork.ai](https://www.stork.ai/blog/elevenlabs-api-key-and-pricing-2026); [eesel Gemini TTS pricing](https://www.eesel.ai/blog/gemini-3-8-flash-tts-pricing); [Google TTS pricing](https://cloud.google.com/text-to-speech/pricing); [beri.net](https://www.beri.net/article/elevenlabs-vs-cartesia-vs-azure-tts-latency-pricing-voice-cloning-2026)
- STT and live cost inputs are Chirp 3 STT at $0.016/min, Soniox at about $0.10–0.12/hour, gpt-realtime-2.1-mini at about $0.02–0.05/min, and ElevenLabs Agents at $0.08/min plus the LLM. — [Google STT pricing](https://cloud.google.com/speech-to-text/pricing?hl=es-419); [Soniox](https://soniox.com/compare-stt/soniox-vs-openai/hebrew); [layer3labs](https://www.layer3labs.io/guides/openai-realtime-api-pricing); [a8gent](https://a8gent.com/platforms/elevenlabs-agents)
- An Israeli self-test methodology used round-trip ASR word errors to compare Hebrew TTS, finding 4% for both Gemini and ElevenLabs v3. This supports ASR round-trip as one automated metric. — [bintechai](https://bintechai.com/learn/ai-voice)

### Inferences

**Recommended stack (Nov 2026)**

| Job | Primary | Fallback(s) | Notes / est. monthly cost |
|---|---|---|---|
| Hebrew narration (pre-rendered) | Winner of bake-off between **ElevenLabs v4** (audio tags, alias dictionary) and **Gemini 3.8 Flash TTS** (style prompts, cheap, Batch) | **Google Chirp 3 HD he-IL**; then **Azure Hila/Avri** as the always-available baseline | 200k characters: about $16 (11L, ×2–3 for takes) / about $3–6 (Gemini) / $0–6 (Chirp) / $0–3 (Azure). Terms: 11L needs written approval for an under-18 app; Gemini/Vertex has an under-18 bar; decide consciously (sec. 2). |
| Hebrew character voices | **ElevenLabs Voice Design + v4 tags**, with adult-sounding playful characters and no child-like voices | **Gemini Voice Design + 2-speaker dialogue** | Fix voice IDs in a "voice bible"; render dialogues in one request for consistency; included in the cost above. |
| English narration (US or UK, pick one) | **ElevenLabs v4** or **Gemini 3.8 Flash TTS**, the same vendor as Hebrew for consistency | **Azure en-US/en-GB neural or HD** (SSML rate and IPA control, $15–22 per 1M); **Chirp 3 HD en-US** | 50k characters/month: under $5 anywhere. |
| English phonics (isolated sounds) | **Human recording session** of about 44 phonemes and blends by a native speaker, one-off cost | **Wikimedia Commons IPA clips** (CC BY-SA); trimmed TTS continuants | TTS only for words and minimal pairs, with IPA phoneme tags (Azure SSML or 11L v3 IPA). |
| Hebrew STT, 8.5-year-old reading aloud | **ivrit.ai whisper-large-v3-turbo** (Apache-2.0) on device via whisper.cpp/GGUF (about 450 MB quantized) or on a home Mac, with reference-text alignment | **ElevenLabs Scribe v2**, **Soniox** (about $0.12/hour) or **Google Chirp 3 iw-IL** ($0.016/min); **Apple SpeechTranscriber** for live highlighting if Hebrew is installed | 300 min/month: $0 on device; about $0.60–4.80 in the cloud. Lenient scoring; parent review. |
| English STT and pronunciation (5.5 and 8.5) | **Azure Pronunciation Assessment en-US**, scripted, single-shot, `EnableMiscue` | **Speechace** (about $40/month basic, 30-second cap); Apple SpeechTranscriber en-US for free recognition | Under $5/month at family scale (Azure PA price unverified). Constrained prompts for the 5.5-year-old. |
| Live conversation | **Don't**, for the 2.5- and 5.5-year-olds. For the 8.5-year-old: **cascaded turn-based** (Apple STT → LLM with moderation → cached or streamed TTS), parent present | **OpenAI gpt-realtime-2.1-mini** with zero data retention, if speech-to-speech is wanted | 60 min/month: about $1–3 (mini). Gemini Live is excluded by its terms; ElevenLabs Agents need approval. |
| Parent voice | **ElevenLabs Instant Voice Clone** of the parent (own voice), v4 | **MiniMax 2.8 clone** (rated best for Hebrew in 2025) | Short personal lines only; never children's voices. |

Overall at about 200k Hebrew plus 50k English characters per month, with STT and light live use: about $10–40 per month on the ElevenLabs path, or about $5–15 per month on the Google/Azure path. Once the content library is rendered and cached, the ongoing cost falls to near zero.

**Bake-off protocol (runnable once API keys exist; about 2–3 hours of parent time)**
1. **Test set.**
   - **Hebrew, 30 lines:**
     - 6 kid-directed narration lines (questions, exclamations, praise);
     - 6 homograph or stress traps, e.g. ספר, בוקר/בקר, טחינה, כלבי, the child's name in a sentence, שמונה as a number and as a word;
     - 4 lines with numbers, dates, times and gendered numerals ("3 בנות", "שלוש עשרה");
     - 4 Hebrew sentences with English words inside ("המילה dog באנגלית");
     - 4 slow "teaching" lines;
     - 3 dialogue lines between two characters;
     - 3 lines given both unvocalized and with nikud added by Phonikud, to test whether the vendor honours nikud.
   - **English, 20 lines:** narration, minimal pairs (ship/sheep, bat/bet, thin/sin), CVC words, slow-rate lines, and attempts at isolated /s/, /m/, /b/ and /æ/.
2. **Engines.**
   - Hebrew: ElevenLabs v4, v4 Turbo and v3 (2 voices each, with `language_code he`); Gemini 3.8 Flash TTS and Flash-Lite (2 voices each, with a style prompt "warm Israeli kindergarten teacher, native Israeli accent, slow and clear"); Chirp 3 HD he-IL (all voices); Azure Hila and Avri; Cartesia Sonic 3.6; MiniMax Speech 2.8 HD.
   - English: ElevenLabs v4, Gemini 3.8, Azure en-US HD, Chirp 3 HD en-US.
3. **Rendering.** A script renders each line per engine and voice, normalizes loudness, randomizes file names and writes a hidden key file. It also logs latency, characters billed and cost.
4. **Automated metrics.**
   - (a) Round-trip WER/CER: transcribe each clip with ivrit.ai Whisper (Hebrew) or Whisper/Apple (English) against the script.
   - (b) Stress check: compare phonemized ASR or `whisper-heb-ipa` output with Phonikud's IPA of the script on the trap words.
   - (c) Count failures: empty or cut audio, hallucinations, wrong language.
5. **Human blind listening.**
   - Both parents rate each clip 1–5 on "sounds like a native Israeli", warmth and kid-appeal, and mark every mispronounced word.
   - The 8.5-year-old (and the 5.5-year-old for a few clips) does A/B "which voice do you like" on the narration lines.
6. **Decision rule.** Drop any engine with a stress or homograph error rate above 10% or any hallucination. Among the rest, take the highest native-ness and warmth score, then the lowest cost. Keep the runner-up as the fallback. Re-run a 10-line mini bake-off whenever a vendor ships a new model (e.g., v4 updates, Gemini 2027 models).
7. **STT bake-off.**
   - Record the 8.5-year-old reading 10 Hebrew passages (about 60 seconds each) and 10 English passages. Keep the recordings locally only and do not upload them to vendors whose terms forbid minors' data.
   - Hand-correct transcripts, then compute WER for ivrit.ai turbo on device, Apple SpeechTranscriber, and any cloud engine whose terms allow it.
   - For English, also compare Azure PA phoneme scores with parent judgments on 20 minimal-pair words.

### Gaps
- Every quality ranking above is provisional until the bake-off runs; no 2026 Hebrew benchmark exists.
- Azure Pronunciation Assessment pricing, Gemini Live official pricing, Cartesia and MiniMax per-character pricing, and the Chirp 3 HD free-tier size were not verified.
- The legal interpretation of the under-18 clauses for a private, family-only app has no official vendor answer. It needs either written approval (ElevenLabs) or a conscious risk decision by the user.
