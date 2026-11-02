# Hebrew-First Language, Voice and Localization for a Kids' Learning Game That Teaches English

Scope: Hebrew TTS, Hebrew ASR for children, Hebrew LLM quality, gendered address, Hebrew-as-medium EFL teaching, Israeli cultural localization (calendar, kids' culture, security-situation sensitivity), and Hebrew number words. Target: Israeli family, kids aged 2.5, 5.5 and 8.5, on iPhone/iPad. Date of research: 2 Nov 2026.

Method note: 16 searches and fetches. Much of the Hebrew-specific evidence is from community tests, vendor pages and arXiv preprints, not peer-reviewed or independent benchmarks. The confidence level is marked throughout. Statements under "Inferences" are my reasoning or general linguistic or industry knowledge without a fetched source. The report writer should keep them clearly separate from the cited facts.

---

## 1. Hebrew TTS for kids' content (naturalness, nikud, stress, homographs, numbers, gendered address, parent voice cloning)

### Takeaway
No vendor reliably gets Hebrew pronunciation right from unpointed text. Hebrew script leaves out vowels and stress, so every engine has to guess, and homographs and milra/mil'el stress are where the guesses fail. The best 2026 options for warm narration are:
- ElevenLabs v3, with `language_code: "he"` set explicitly.
- Google Chirp 3 HD, which added he-IL in late 2025.
- Gemini TTS.
- Azure Hila/Avri, which are older but stable.

For a kids' app with a fixed curriculum, the robust approach is to pre-generate all narration offline, drive pronunciation with selective nikud or a phonetic layer (Phonikud/Dicta Nakdan), have a native speaker do QA, and ship the audio as static files. Live TTS should be reserved for dynamic strings such as names.

### Cited Findings
- **ElevenLabs:** In a community comparison (snapshot 22 Mar 2025), ElevenLabs stock voices (Rachel, Adam, Bella) on `eleven_v3` were rated "Good" for Hebrew, but only with `language_code: "he"` set. The older Multilingual v2 model was "unintelligible" for Hebrew. — [danielrosehill/Hebrew-TTS-Providers](https://github.com/danielrosehill/Hebrew-TTS-Providers); samples at [Hebrew TTS Samples](https://danielrosehill-hebrew-tts-samples.static.hf.space/index.html)
- **Other providers in the same comparison:**
  - MiniMax T2A v2.6 Turbo voice clones were rated "Best", described as "most impressive results — cloned voices sounded natural in Hebrew". They were cloned from about one minute of English source audio.
  - Edge TTS (Microsoft Avri M and Hila F) was rated "Good".
  - Gemini 2.5 Flash Preview TTS (voices Puck and Zephyr) was rated "Good".
  - Chatterbox Multilingual clones were rated "Poor": cloning did not carry into Hebrew, and one run produced gibberish because a Dicta ONNX model was missing.
  - Resemble AI was rated "Poor": the stock voice Avigail was "usable but limited", custom clones were unintelligible, and it needed nekudot to be intelligible.

  — [danielrosehill/Hebrew-TTS-Providers](https://github.com/danielrosehill/Hebrew-TTS-Providers)
- **Age of that comparison:** It predates ElevenLabs' March 2026 model update. One 2026 article reports that the new voice model became the default in March 2026 with improved emotional range. — [DEV Community](https://dev.to/techsifted/elevenlabs-new-voice-model-in-march-2026-the-emotional-range-finally-caught-up-5619)
- **Nikud with ElevenLabs v3 (community guidance):** Add nikud selectively, only on words that can be read more than one way. Fully pointed text can degrade the output. — [openclawdir ElevenLabs TTS skill](https://openclawdir.com/skills/elevenlabs-tts-2z68ji)
- **Google Chirp 3 HD:** Hebrew (Israel) `he-IL` is listed in the supported languages. The release notes show Hebrew was added in a late-2025 update. Regional availability differs between doc versions: global/us/eu are GA, and some versions also list asia-southeast1, europe-west2 and asia-northeast1. Google's Speech-to-Text Chirp 3 uses the code `iw-IL` (Preview) for Hebrew, so the codes differ between TTS and STT. — [Google Chirp 3 HD docs](https://docs.cloud.google.com/text-to-speech/docs/chirp3-hd); [Cloud TTS release notes](https://docs.cloud.google.com/text-to-speech/docs/release-notes)
- **Azure:**
  - `he-IL-AvriNeural` (male) was part of Microsoft's December 2020 batch of 51 new neural voices. — [Microsoft Tech Community](https://techcommunity.microsoft.com/t5/azure-ai/azure-neural-text-to-speech-updates-51-new-voices-added-to-the/ba-p/1988418)
  - `he-IL-HilaNeural` (female) is listed at about 113 words per minute. — [json2video Hila](https://json2video.com/ai-voices/azure/voices/he-il-hilaneural); [json2video Avri](https://json2video.com/ai-voices/azure/voices/he-il-avrineural/)
  - My search found no evidence of a new Hebrew-native Azure voice in 2025–2026 beyond Hila and Avri. Users have reported Hebrew-support problems in Home Assistant integrations, which is anecdotal. — [Home Assistant forum](https://community.home-assistant.io/t/microsoft-tts-hebrew-support-broken/416488)
- **Phonikud (open-source Hebrew G2P, arXiv 2506.12311, June 2025, later retitled "Overcoming Phonetic Underspecification for Hebrew Text-To-Speech"):**
  - It adds standard nikud plus extra marks for stress, vocal shva and clitic boundaries, then converts the result to IPA with deterministic rules.
  - The authors report that it predicts phonemes more accurately than prior methods, and that small local TTS models given its input "approach large proprietary systems".
  - It releases the ILSpeech dataset (Hebrew speech with Hebrew and IPA transcriptions).
  - Example: `phonemize('שָׁלוֹם עוֹלָם')` returns `ʃalˈom olˈam`, with explicit stress.
  - A follow-up project, "Renikud", targets further improved G2P.

  — [arXiv 2506.12311](https://arxiv.org/abs/2506.12311); [PyPI phonikud](https://pypi.org/project/phonikud/0.4.0/); [phonikud.github.io](https://phonikud.github.io/)
- **Phonikud claims on its own pages:** The comparison page includes ElevenLabs v3 among the systems tested. The explore page claims a ChatterBox-based Phonikud system beats ElevenLabs in blind evaluations. These are the developers' own claims, not independent. — [phonikud paper explore page](https://thewh1teagle.github.io/phonikud-paper/explore.html)
- **Other open Hebrew TTS:** The comparison repo lists LightBlue TTS Hebrew, Phonikud TTS and ivrit.ai as resources but did not test them. — [danielrosehill/Hebrew-TTS-Providers](https://github.com/danielrosehill/Hebrew-TTS-Providers)
- **Dicta nikud quality:** DictaLM 3.0 24B is reported at 86.86% on automatic nikud on the Hebrew LLM leaderboard. This comes from a Hebrew launch article that I did not verify against the leaderboard. It suggests that even the best automatic pointing still needs human checking for kids' content. — [Gadgety (Hebrew)](https://www.gadgety.co.il/354176/dicta-lm-3-launched/)

### Inferences
- **Why Hebrew TTS is hard:** Unpointed Hebrew is massively ambiguous. ספר can be sefer, sapar or safar. Stress position is lexical and sometimes distinguishes meaning. Engines also must choose the correct gender form for numbers and for 2nd-person address. For a kids' app, the risk is mostly mispronounced words that young children then copy. Children are learning pronunciation from the app, so errors matter more than in adult content.
- **Recommended pipeline for fixed content (most of a curriculum app):**
  1. Write the script in Hebrew.
  2. Point it automatically with Dicta Nakdan or Phonikud.
  3. Have a native speaker correct the nikud and stress for ambiguous words.
  4. Send selectively pointed text, or phonemes/SSML where supported, to the TTS.
  5. Have a native speaker listen and QA the result.
  6. Cache the result as audio files bundled with the app.

  This also removes per-play API cost and network dependency, and avoids sending children's data at runtime.
- **Gendered 2nd-person address:** Build every child-directed line in separate feminine, masculine and plural variants, for example את מוכנה? / אתה מוכן? / אתם מוכנים?. Pre-render each variant, and never let the TTS or LLM choose the gender. This family has a girl and boys, so you need at least the F.SG, M.SG and plural (M.PL default for mixed groups) variants. Optionally offer F.PL, and an inclusive form such as מוכנים/ות for text-only contexts. Slash forms should not be spoken.
- **Numbers:** TTS engines read digits as cardinal numbers, often in the masculine or "counting" form. For math content, write the number out in words with the correct gender, for example שָׁלוֹשׁ תַּפּוּחִים, and do not pass digits.
- **Apple on-device voices:** iOS ships Hebrew voices for AVSpeechSynthesizer: Carmit, plus downloadable enhanced/premium variants. They are free, offline and private, but robotic compared with cloud neural voices. They suit fallback and accessibility, not character narration. I did not fetch an Apple source in this session, so verify the voice names and quality tiers on a device.
- **Parent voice cloning:**
  - *Technically feasible:* MiniMax clones from about one minute of audio sounded natural in Hebrew in the 2025 test. Cloning quality varies widely by vendor, and Chatterbox and Resemble failed.
  - *Ethical and privacy recommendations:*
    - Use it only with the explicit consent of the parent being cloned.
    - Keep the voice for family-only use.
    - Prefer vendors with clear voice-ownership and deletion terms.
    - Never use a cloned parent voice to deliver praise for engagement metrics or to give instructions that sound like real-world parental commands, such as "come to me now". For a 2.5-year-old that could blur reality.
    - Consider limiting the cloned voice to bedtime stories or recorded-style messages.
  - A simpler, warmer alternative is to have the parent record a small set of real clips: the child's name, "כל הכבוד", and holiday greetings.

### Gaps
- I found no independent 2026 blind MOS comparison of Hebrew TTS that includes ElevenLabs post-March-2026, Chirp 3 HD Hebrew, OpenAI gpt-4o-mini-tts Hebrew and Azure. The only side-by-side found is from March 2025.
- I found no vendor documentation on how ElevenLabs, Google or OpenAI handle nikud, stress or homographs, nor on whether Chirp 3 HD Hebrew supports SSML `<phoneme>`.
- I found no source on child-friendly or "kid-character" Hebrew voices, or on Israeli startups (Dubverse/Speechify-type Hebrew vendors). Robo-Shaul was not covered by the sources found.
- OpenAI TTS Hebrew quality was not evaluated in any source found.

---

## 2. Hebrew speech recognition for children's speech (reading-aloud checks)

### Takeaway
The ivrit.ai Whisper fine-tunes are the strongest open Hebrew ASR, with about 5% WER on adult podcast speech. They are trained only on adult speech, and I found no Hebrew child-speech ASR data. Apple's on-device SpeechAnalyzer Hebrew support is unconfirmed. For reading-aloud checks, use constrained, forgiving verification and not free transcription.

### Cited Findings
- **ivrit.ai Whisper Large v3:** This is a Hebrew continued training of OpenAI Whisper on about 300 h of crowd-transcribed audio, about 50 h of crowd-recited Wikipedia, and about 4,700 h of Knesset plenums. The model card points to the ivrit.ai Hebrew transcription leaderboard for WER. — [HF ivrit-ai/whisper-large-v3](https://huggingface.co/ivrit-ai/whisper-large-v3)
- **Third-party WER on ivrit.ai eval-d1 (podcasts):** ivrit-ai/whisper-large-v3-turbo scored 4.87 WER. A streaming Hebrew+English model, "kol-stream-he-en", scored 5.80 in batch mode. Both figures are self-reported by the model authors. — [HF MaorB/kol-stream-he-en](https://huggingface.co/MaorB/kol-stream-he-en)
- **Bilingual streaming model:** That Hebrew+English model is relevant for code-switched child utterances. — [HF MaorB/kol-stream-he-en](https://huggingface.co/MaorB/kol-stream-he-en)
- **Small models:** Community Whisper-small Hebrew fine-tunes report about 37.9 WER. Small, on-device-sized models are much weaker. — [HF mike249/whisper-small-he-3](https://huggingface.co/mike249/whisper-small-he-3/blob/main/README.md?code=true)
- **ivrit.ai talk (PyData Tel Aviv 2025):** Knesset-data fine-tunes of Whisper-base and Turbo gave about a 10% WER reduction compared with earlier models. — [PyData TLV 2025](https://cfp.pydata.org/pydata-tel-aviv-2025/talk/P9AKQD)
- **Independent 2026 evaluation on Holocaust testimonies:** Mean Whisper WER was 15.28%, and the ivrit.ai models were called "promising". — [ACL Anthology 2026.htres-2.10](https://aclanthology.org/2026.htres-2.10/)
- **Google STT Chirp 3:** Hebrew is listed as `iw-IL`, in Preview. — [Google Chirp 3 HD / STT docs](https://docs.cloud.google.com/text-to-speech/docs/chirp3-hd)
- **Apple SpeechAnalyzer/SpeechTranscriber (iOS 26):**
  - Apple exposes `supportedLocales`, including downloadable-but-not-installed locales. — [Apple docs](https://developer.apple.com/documentation/speech/speechtranscriber/supportedlocales.md)
  - One third-party launch list (Cantonese, Chinese, English, French, German, Italian, Japanese, Korean, Portuguese, Spanish) omits Hebrew. — [addpipe blog](https://blog.addpipe.com/apple-speechanalyzer-api.md)
  - A Japanese article says about 40 locales are supported. — [issoh.co.jp](https://www.issoh.co.jp/tech/details/8929/)
  - A forum thread reports a locale (Arabic) listed as supported whose model could not be downloaded. — [Apple Dev Forums](https://developer.apple.com/forums/thread/797835)

### Inferences
- **Child speech:** Children's speech has higher pitch, different formants, disfluencies and developing phonology, and this reliably degrades adult-trained ASR. For a 2.5-year-old, ASR is impractical. For a 5.5-year-old, expect high error rates. An 8.5-year-old reading aloud is the realistic use case.
- **Practical reading-aloud strategy:**
  - (a) Use a known target text, and score alignment or keyword spotting against it instead of open transcription.
  - (b) Accept near-matches, with lenient thresholds and phonetic matching via Phonikud IPA of the target.
  - (c) Never punish the child for an ASR failure. Fall back to "let's listen together", or have the parent tap to confirm.
  - (d) Prefer on-device processing (legacy SFSpeechRecognizer he-IL with `requiresOnDeviceRecognition` where available, or a bundled small model) for privacy.
  - (e) Use cloud ivrit.ai Turbo only with parental consent, for periodic, non-real-time assessment.
- **English reading checks:** Use English ASR, such as Apple's on-device en-US. Its child-speech accuracy is also imperfect, but it is better resourced than Hebrew.
- **On-device check:** Before committing, query `SpeechTranscriber.supportedLocales` on a real device for `he_IL`, and test the AssetInventory download.

### Gaps
- I found no Hebrew children's speech corpus or child-speech WER figures.
- Apple's Hebrew support in SpeechAnalyzer (iOS 26+) is not confirmed. Legacy SFSpeechRecognizer he-IL on-device availability was not verified in this session.
- I did not retrieve the current ivrit.ai leaderboard numbers directly.

---

## 3. Hebrew LLM quality (benchmarks, failure modes, mitigation)

### Takeaway
Hebrew LLM evaluation now exists (the DICTA/HF Hebrew LLM Leaderboard and DICTA's new chat suite), and DictaLM 3.0 is the leading open Hebrew model. No benchmark measures what a kids' app most needs: correct gender agreement in child-directed address, natural non-translationese phrasing, and correct nikud. Treat LLM output as a draft. Generate content offline, validate it with Nakdan, and get native-speaker review. Never let an LLM produce live, unreviewed Hebrew for a 2.5–8.5-year-old.

### Cited Findings
- **Dicta-LM 3.0 (arXiv 2602.02104, Feb 2026):**
  - Sizes: 24B (based on Mistral-Small 3.1), 12B (Nemotron-based) and 1.7B (Qwen3-based), under Apache 2.0.
  - The 24B model matched or exceeded larger models on the Hebrew LLM Leaderboard and achieved the top score in Israeli Trivia.
  - The paper introduces a 0-shot Hebrew chat benchmark covering translation, summarization, Winograd, Israeli trivia and diacritization (nikud). Summarization and translation are judged by GPT-4o against Gemini-2.5-Pro outputs.

  — [arXiv 2602.02104](https://arxiv.org/abs/2602.02104); [Moonlight review](https://www.themoonlight.io/review/dicta-lm-30-advancing-the-frontier-of-hebrew-sovereign-llms)
- **Reported 24B scores:** 86.86% nikud, 56.86% summarization and 78.06% contextual understanding. It is described as the best open Hebrew model up to 70B on the leaderboard. This is not verified against the leaderboard. — [Gadgety (Hebrew)](https://www.gadgety.co.il/354176/dicta-lm-3-launched/)
- **Leaderboard method:** The leaderboard evaluates base models few-shot, so its scores do not directly reflect chat-model behavior. — [Moonlight review](https://www.themoonlight.io/review/dicta-lm-30-advancing-the-frontier-of-hebrew-sovereign-llms)
- **Gender in MT/LLMs:**
  - The MiTTenS gender-mistranslation dataset (26 languages) — [arXiv 2401.06935](https://arxiv.org/html/2401.06935v3)
  - A coreference study found that in Arabic and Hebrew "alignment errors occur more often" for gender. — [arXiv 2302.08464](https://arxiv.org/pdf/2302.08464)
  - A March 2026 paper proposes Minimal Pair Accuracy to test whether models use contextual gender cues or fall back on stereotypes. — [arXiv 2603.17952](https://arxiv.org/html/2603.17952v1)
- **Translationese:** LLMs exhibit translationese and literalism traced to biases from supervised fine-tuning. This is from Li et al. 2025, as summarized in search results and not Hebrew-specific; I have no direct link fetched.

### Inferences
- **Typical Hebrew failure modes for frontier LLMs (Claude/GPT/Gemini)**, based on general practitioner knowledge and not on a fetched benchmark:
  - Defaulting to masculine 2nd person, or flipping gender mid-text.
  - Wrong numeral gender (שלושה ילדות).
  - Unnatural calques from English ("לקחת החלטה" is acceptable, but other calques are less so).
  - Overly formal or literary register that is unsuitable for kids.
  - Hallucinated or incorrect nikud when asked to point text.
  - Wrong stress cues.
  - English-centric cultural references.
- **Mitigations:**
  1. Write a system prompt and style guide that specifies the child's gender explicitly for every generation, along with register ("ספרות ילדים, משפטים קצרים, עברית מדוברת תקנית") and banned constructions.
  2. Generate F, M and PL variants as separate structured fields, and run automated checks: regex or morphological analysis of 2nd-person forms against the expected gender, using DictaBERT or the Dicta morphological tagger.
  3. Point text with Dicta Nakdan, not the LLM, and have a human verify ambiguous words.
  4. Run a native-speaker review of everything a child hears, ideally an Israeli early-childhood educator for the 2.5 and 5.5 year-olds.
  5. Use DictaLM 3.0 as a second-opinion checker or reranker for naturalness. It can run self-hosted, which keeps children's data local.
- **For live/dynamic features** (e.g., an LLM tutor for the 8.5-year-old), constrain output to templates or short responses, with a post-generation gender and profanity validator. Keep a fallback to a canned response.

### Gaps
- I found no published Hebrew benchmark scores for Claude, GPT or Gemini, and no Hebrew-specific gender-agreement or translationese benchmark.
- I did not fetch the current state of the "Open Hebrew LLM Leaderboard" (HF/DICTA) or the HeQ results.

---

## 4. Gendered language design (addressing each child correctly; plural and family)

### Takeaway
Hebrew requires gender marking on 2nd-person pronouns, verbs, adjectives and imperatives, so a Hebrew-first app must store each child's grammatical gender in the profile and render every string from gender-specific variants. This applies to text, audio and character dialogue. No authoritative published "Israeli app best practice" was found. The recommendations below are design inferences.

### Cited Findings
- Gender is a central organizing factor in children's acquisition of Hebrew noun and adjective plural morphology, and accuracy rises steadily from kindergarten into adulthood. Young children are therefore still acquiring these forms, and the app models them. — study summarized in search results (Language Learning, 2012); see also [CRIS IUCC](https://cris.iucc.ac.il/en/publications/from-dichotomy-to-divergence-numbergender-marking-on-hebrew-nouns-2/)

### Inferences
- **Data model:** Each profile has `grammatical_gender ∈ {f, m}`. Optionally add a parent-chosen override for gender-neutral phrasing. Each string key holds `{f, m, pl_m, pl_f}` variants, and the localization layer selects a variant by addressee. Multi-child or family screens use plural, with masculine plural as the grammatical default for a mixed group. Feminine-first inclusive text (e.g., "מוכנות ומוכנים") is optional for parent-facing text.
- **Audio:** Every voiced line needs all variants pre-recorded, so the audio budget roughly doubles or triples. To limit the cost, write lines that avoid gendered forms where natural:
  - Use the infinitive as an instruction ("לגעת בכוכב", "לשמוע שוב"), a pattern common in Israeli UI.
  - Use 1st-person plural ("בואו נספור").
  - Use the child's name as a vocative.
- **Imperatives:** Imperatives are strongly gendered (גע/געי/געו), so prefer the infinitive or "בוא/י" variants as needed.
- **Characters:** The characters themselves have genders that require agreement in narration (הדובה אמרה / הדוב אמר). Keep a character registry with gender metadata, so that generated content agrees with it.
- **Tests:** Add automated tests that render every screen for each profile and flag any 2nd-person form that mismatches. A native speaker should review a screenshot matrix.
- **The 2.5-year-old** will not read, so audio and visual cues dominate. The 8.5-year-old reads and will notice errors. Mis-gendering is a strong negative signal for Israeli kids and parents.

### Gaps
- I found no published UX guidelines or case studies from Israeli apps (e.g., Israeli ed-tech or government apps) on gendered address.
- Gender-neutral Hebrew conventions (e.g., the "נבינארית" proposals) were not researched.

---

## 5. Bilingual pedagogy: Hebrew as medium for teaching English to young learners

### Takeaway
Current applied-linguistics consensus supports principled, judicious own-language (L1) use rather than English-only. Hebrew should carry instructions, scaffolding, meaning checks and emotional support. English should be the target of short, protected immersion moments. Use sandwiching (EN → HE gloss → EN) for new items. Separate the languages clearly through character, voice and visual cues, with a "one character, one language" convention for the English speaker.

### Cited Findings
- **Hall's position:** Graham Hall prefers the term "own-language". He notes that teachers say "judicious/appropriate/optimal" without clear guidelines, and recommends "sandwiching": inserting a brief own-language translation into the discourse, followed by the English again. — [IATEFL 2018 report on Graham Hall](https://eim-ub.blogspot.com/2018/04/iatefl-2018-graham-hall.html)
- **Hall & Cook (2012):** They recommend that L1 be used "cautiously". Butzkamm (2003) describes good use as "systematic, selective and judicious". A bilingual view of language classrooms "now prevails in academic circles". — [Tekin & Garton 2020 (Scribd)](https://www.scribd.com/document/516305061/Tekin-and-Garton-2020); [hrcak.srce.hr](https://hrcak.srce.hr/file/247157)
- **Cognitive load:** Cook and Hall (2012) argue that L1 use reduces learners' processing load during cognitively difficult tasks. — [Leeds Language Scholar](https://languagescholar.leeds.ac.uk/the-role-of-l1-in-an-efl-classroom); [typeset: shared L1 reduces cognitive overload and anxiety](https://typeset.io/pdf/using-a-shared-l1-to-reduce-cognitive-overload-and-anxiety-3i3gmc0x93.pdf)
- **Sandwich stories / diglot weave:** These embed L2 words in L1 discourse. Yuhua (1999) argues that they move learners from the familiar to the unfamiliar. This is from a small-scale source, so treat it cautiously. — [readingmatrix.com](https://readingmatrix.com/files/21-347acl84.pdf)
- **Young learners:** Aminifard & Mehrpour (2019), a survey of 110 teachers of young EFL learners, call for "prudent" mother-tongue use. Most L1-use research focuses on adults, so evidence for young learners is thinner. — [ensani.ir](https://ensani.ir/fa/article/download/278120); [LASLab: L1 use among young EFL and CLIL learners](https://laslab.org/conference_paper/l1-use-among-young-efl-and-clil-learners-in-task-based-interaction)
- **Israeli context:**
  - Israeli state schooling has traditionally started formal English in grade 3. The source is undated. — [Forward](https://forward.com/fast-forward/381465/israel-launches-20m-program-to-teach-children-better-english/?amp=1)
  - There have been pilots and teacher training for English in kindergarten. — [Times of Israel](https://www.timesofisrael.com/english-to-be-taught-in-israeli-kindergartens/amp/)
  - The MoE English Curriculum 2020 (elementary) is a CEFR-aligned revision of the 2018 curriculum. — [Scribd: Curriculum 2020 Elementary](https://www.scribd.com/document/500196677/Curriculum-2020-Elementary)
  - Israel announced AI-driven personalized English learning in schools. — [JNS](https://www.jns.org/news/israel-news/israel-to-implement-ai-driven-personalized-english-learning-in-schools)

### Inferences
- **Age-banded design:**
  - **2.5 years:** Hebrew-dominant play, with English as songs, chants and single words: animals, colors, "hello/bye". Use rhythmic repetition and no reading. English exposure should be short and joyful.
  - **5.5 years:** Hebrew instructions, English vocabulary in themed sets, and listening/picture-matching. Start phonemic awareness in English orally. Use sandwiching for every new word: "Apple — תפוח — apple!".
  - **8.5 years:** Grade-3 Israeli curriculum alignment, with Latin-letter decoding, phonics and sight words. Hebrew is used for explaining rules and metacognition. Gradually lengthen English-only "immersion islands", for example a 60–120 s "English Corner" scene where the English character speaks only English and Hebrew help is available via a button.
- **Marking the language switch (recommended):**
  - (a) A dedicated English-speaking character (e.g., a visiting friend from abroad) who only speaks English, following the "one person, one language" principle. Hebrew characters may use the occasional English word but always frame it.
  - (b) A distinct voice timbre for English.
  - (c) A visual cue such as a flag-free badge, a color band or a speech-bubble style for English.
  - (d) An audio jingle into and out of English moments.
  - (e) A "?עזרה בעברית" button that gives a Hebrew gloss and then repeats the English, which is sandwiching on demand.
- **Script direction (RTL/LTR):**
  - The UI shell is RTL. English text blocks are LTR-isolated, using the Unicode bidi isolation or iOS `semanticContentAttribute` per view.
  - Never mix Hebrew and English inside one sentence-level text node for early readers.
  - For English reading tasks, show text on a visually distinct "English page" laid out LTR, with left-to-right finger-tracking animation. This teaches the direction explicitly; it is a known confusion point for Hebrew readers starting English.
  - Number direction and math layouts should follow Israeli school convention: digits are LTR even in RTL text.
- **Avoid making Hebrew a crutch for comprehension of every English utterance.** The research supports L1 for meaning-making and affect, but learners need comprehensible English input. Fade the Hebrew gloss as items become known, using spaced repetition.

### Gaps
- I found no empirical studies specifically on Hebrew-L1 preschool EFL apps, nor RCT evidence on sandwiching with under-6s.
- The current (2025–26) MoE starting grade for English and any kindergarten English mandate are not confirmed.
- "One person, one language" research is mainly from bilingual family studies. I did not fetch a source for its application to characters in apps.

---

## 6. Israeli cultural localization (calendar, Shabbat, kids' culture, geography, security-situation sensitivity)

### Takeaway
Base seasonal content on the Hebrew calendar. Use Israeli kids' media references carefully: Hop!, Rechov Sumsum and Yuval HaMevulbal are well established, but licensed characters cannot be used. Handle war and security content with an evidence-based "calm, routine, parent-mediated" approach that follows MoE/Shefi early-childhood resilience materials. Avoid sirens and alarm-like sounds in the app.

### Cited Findings
- **Hop! Channel:** An Israeli preschool channel launched in 2000 as the first commercial-free dedicated channel for kids under 7. Programs include Rechov Sumsum (Israeli Sesame Street, with Moshe Oofnik) and Yuval HaMevulbal ("Yuval the Confused", Yuval Shem-Tov, with a Hop! series running three seasons from about 2006). Other Hop! shows include Little Michal and the Monsters (2015) and Muli & Tsumi (2013). — [Wikipedia: Hop! Channel](https://en.wikipedia.org/wiki/Hop!_Channel); [Wikipedia: Yuval Shem-tov](https://en.wikipedia.org/wiki/Yuval_Shem-tov); [World Screen: Sesame Street returns to Israel](https://worldscreen.com/?p=86138)
- **Parents drive children's stress:** In a University of Haifa survey of 128 parents of young children, young children's stress tracked parents' emotional responses more than siren frequency. About 66% of parents reported three or more sirens a day, but only 13.3% of children showed high stress. — [JNS](https://www.jns.org/news/israel-news/parents-not-sirens-drive-childrens-stress-levels-during-war-researchers-say)
- **Distress prevalence:** The Israeli Pediatric Association reported that 84% of Israeli children suffered emotional distress after Oct 7. This is a secondary report, so use it cautiously. Therapeutic workbooks such as "Inner Space: My Resilience Workbook" and "I Feel That Way and That's Okay!" were distributed. — [JNS: Helping Israel's children](https://www.jns.org/feature/helping-israels-children-through-this-war)
- **Clinical guidance:**
  - Be honest, spare young children details, and offer hugs and reassurance.
  - An Ashkelon resilience center frames sirens like seatbelts ("they save lives").

  — [Jewish Press 2014](https://jpeditors.jewishpress.com/news/israel/families-grapple-with-lasting-effects-of-israel-gaza-conflicts/2014/07/16); [Fox News 2021](https://www.foxnews.com/world/israel-palestinian-conflict-traumatic-children)
- **NATAL helpline:** \*3362, available 24/7. — [JPost opinion](https://www.jpost.com/opinion/article-858180)
- **MoE early-childhood division / Shefi materials:**
  - An emergency outline for keeping contact with kindergartens, with home-activity suggestions for parents, dated 19.03.26. — [meyda.education.gov.il 19-3](https://meyda.education.gov.il/files/PreSchool/hirum/19-3.pdf)
  - "חוסן – היום שאחרי", a kindergarten program from the MoE and the Cohen-Harris Resilience Center (2021). It names rockets, sirens, protected spaces and disrupted routine as challenges. — [hosen-2021.pdf](https://meyda.education.gov.il/files/PreSchool/hosen-2021.pdf)
  - Kindergarten "keeping safe" material. — [keeping-safe.pdf](https://meyda.education.gov.il/files/Pop/0files/akademia-reshet/kindergarten/keeping-safe.pdf)
  - An emergency guidance pack with "messages of resilience and hope" ("שאגת הארי" file). — [shaagat-haari3.pdf](https://meyda.education.gov.il/files/Dovrut/shaagat-haari3.pdf)
- **Common recommendations across these materials:**
  - Keep a fixed routine even with siren disruptions.
  - Watch for regression, sleep problems, clinginess, separation difficulty and stomach aches.
  - Give honest, age-appropriate explanations.
  - Do not expose children of any age to news during a crisis.
  - Encourage activities that build a sense of control.

  — [ice.co.il](https://www.ice.co.il/health/news/article/1070961); [mako](https://www.mako.co.il/home-family-kids/Article-a3c10c142ea0791026.htm); [ALUT: התמודדות הורית בזמן לחימה (2024)](https://alut.org.il/wp-content/uploads/2024/01/התמודדות-הורית-בזמן-לחימה.pdf)

### Inferences
- **Calendar engine:** Use the Hebrew calendar (e.g., `Calendar(identifier: .hebrew)` on iOS) to drive the seasonal events below, each paired with English vocabulary themes:
  - Rosh Hashana: apple and honey; "Happy New Year", "apple", "sweet".
  - Yom Kippur: a quiet day; no gamified pushes.
  - Sukkot: sukkah decorating.
  - Hanukkah: candles counting 1–8, a natural math and English-numbers fit; "candle", "light".
  - Tu BiShvat: trees and fruits.
  - Purim: costumes; dress-up vocabulary, "I am a…".
  - Pesach: spring, cleaning, the 4 questions; the "Who knows one?" counting song fits math.
  - Yom HaShoah and Yom HaZikaron: no celebratory content, and pause or soften rewards and sounds on these days.
  - Yom Ha'atzmaut: flags, picnic/mangal, Israel geography.
  - Lag BaOmer: bonfire.
  - Shavuot: dairy and first fruits.
- **Shabbat-aware behavior:**
  - A parent setting can pause notifications and streaks from Friday afternoon to Saturday night. Streaks should not "break" over Shabbat or holidays.
  - Use the school-year rhythm: September start, summer break July–August, Hanukkah and Pesach breaks.
  - Use the Israeli weekend (Fri–Sat). The week starts on Sunday.
- **Israeli kids' culture:**
  - Reference genres and folk material rather than licensed characters: classic songs by Datia Ben-Dor, Naomi Shemer, Levin Kipnis and Miriam Yalan-Shteklis, and holiday songs.
  - Original characters can borrow the Israeli tone: warm, a bit cheeky, sabra-style directness, humor based on playful mishaps (as in Yuval HaMevulbal).
  - Do not use Hop!/Yuval/Sumsum IP without licensing.
- **Moledet geography:**
  - Cities and landmarks such as Jerusalem, Tel Aviv, Haifa, Eilat (Red Sea), the Dead Sea, the Kinneret, the Hermon (snow), Masada and the Negev/Ramon Crater.
  - Animals such as the hyrax (שפן סלע), ibex (יעל), gazelle and hoopoe (דוכיפת, the national bird).
  - Mixed English and Hebrew naming is a natural vocabulary bridge.
- **Security-situation sensitivity (recommendations):**
  - Never use rising/falling siren-like tones, explosion sounds, "alert" red flashes, or sudden loud noises.
  - Avoid themes of hiding from danger, rockets or soldiers-in-combat for under-6s.
  - Include an optional calm "safe room" (מרחב מוגן) mode with breathing games, short stories and familiar routines. It should be easy to open offline and in low light, and could be used in a shelter.
  - Content should emphasize safety, the trusted adults around the child, and agency ("what I can do"), in line with the Hosen/MoE framing.
  - Parent-facing content should note that parental calm matters most, and link NATAL \*3362 and the MoE/Shefi materials.
  - The app must not issue or simulate Home Front Command alerts.
  - Do not reference current events in content, because they date quickly and may trigger children.
- **Diversity:** Families may be secular, traditional or religious. Holiday content should be cultural and inclusive, with a parent toggle for religious-observance depth.

### Gaps
- "Shuli" and "Bli Ba" could not be verified as specific Israeli children's characters or shows. I did not retrieve current (2025–26) popular Israeli kids' YouTube/TV properties.
- No formal, 2025–26 Shefi/NATAL/ERAN guideline specifically about digital or app content for children in wartime was found. The MoE documents found are mainly 2021–2024, plus one dated March 2026.
- I did not research Israeli humor-research sources or Israeli ed-tech localization case studies.

---

## 7. Hebrew number words and math language

### Takeaway
Hebrew numerals are gendered with "polarity": the form ending in ‑ה is used with masculine nouns (שלושה ילדים), the form without it with feminine nouns (שלוש ילדות). Numerals also have construct forms (שלושת הילדים), and the bare counting form is feminine (אחת, שתיים, שלוש). Even native speakers neutralize the gender distinction, so a math app must model this carefully and consistently. There is no evidence that it harms math learning.

### Cited Findings
- **Neutralization study:** A study of 40 Israeli children (20 fourth graders and 20 seventh graders) in monitored versus unmonitored speech found evidence of the "disappearance of gender agreement in Modern Hebrew numerals" and a reanalysis of numeral suffixes by speakers. Children commonly do not produce the normative forms. — [Cambridge, Language Variation and Change: "Neutralization of gender distinctions in Modern Hebrew numerals"](https://www.cambridge.org/core/journals/language-variation-and-change/article/abs/neutralization-of-gender-distinctions-in-modern-hebrew-numerals/658F6A3A318E90D3A0D5FD27328E9729)
- **Anecdotal:** Native Israelis themselves often mix up numeral gender. — [r/hebrew thread](https://redlib.belloworld.it/r/hebrew/comments/1cua1rf/a_petty_rant_about_numbers)
- **Learning materials:** Basic resources present separate masculine and feminine number sets. — [morim.com numbers](https://www.morim.com/numbers.htm); [HebrewPod101 counting](https://www.hebrewpod101.com/lesson/basic-bootcamp-4-counting-from-one-to-one-hundred-in-hebrew?lp=27)

### Inferences
- **Design rules for math content:**
  - (a) Use the feminine/abstract counting form for rote counting ("אחת, שתיים, שלוש…"). This is how Israeli kindergartens count.
  - (b) When counting objects, match the noun's gender. Choose object nouns deliberately, and avoid mixing genders within one item.
  - (c) Write all spoken numbers in words with nikud for TTS, never digits.
  - (d) Give the 2.5 and 5.5 year-olds consistent, normative forms. Don't over-correct the child's own non-normative usage, because it is common among natives.
  - (e) Note the 2 forms: שניים/שתיים when standalone, and שני/שתי before a noun.
  - (f) Check numbers 11–19, which also have gendered forms (אחת־עשרה / אחד־עשר), and the construct forms (שלושת, ארבעת).
- **Bilingual math bonus:** English numbers have no gender, so the English module can teach "three" as a single form. Show Hebrew and English numerals side by side with the same digit, which is LTR in both scripts.
- **LLM-generated math word problems** are a high-risk area for numeral-gender errors. Add automated numeral-noun agreement checks using a morphological analyzer such as Dicta's tools, plus human review.

### Gaps
- I found no study that directly links Hebrew numeral gender to children's math performance or early numeracy, or compares it with languages without gendered numerals.
- The publication year of the Cambridge study was not captured (it appears to be older; LVC volume around 2000). The 2012 Language Learning plural study is cited via a search snippet only.
