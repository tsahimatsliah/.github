# Personalization Approaches and Technology Stack for a Family-Personalized Hebrew/English Learning Game (kids 2.5, 5.5, 8.5)

Research note: the session's network blocked direct page fetches from learn.microsoft.com and elevenlabs.io. Vendor claims below come from search-engine summaries of vendor pages (Google docs, ElevenLabs help center) and some third-party pages. Each one is flagged where it applies. Before committing to a vendor, re-check these pages directly: the Azure Speech language-support page, the ElevenLabs language page, and OpenAI's TTS guide.

## How personalization boosts children's learning and engagement (research + concrete feature ideas)

### Takeaway
The evidence that personalization helps is real but modest and context-dependent. Children's names and family details in print books improved preschool word learning and shared-reading enjoyment (Kucirkova; small samples). Matching math word problems to a student's interests improved speed and accuracy, and the gains persisted (Walkington). Research on AI-generated personalized stories in 2025 is still mostly small design studies. So personalization should be used to support the learning goal, not as decoration. Kucirkova warns about "self-focus," and personalization works best on tasks at the edge of the child's ability.

### Cited Findings
**Personalized books (Kucirkova and colleagues)**
- In a study of 18 preschoolers (mean age 3;10), children were read a picture book with both personalized and non-personalized sections. At later testing points they knew the new words from the personalized sections better. Published as Kucirkova, Messer & Sheehy (2014), "Reading personalized books with preschool children enhances their word acquisition," *First Language* 34(3). — [Open University ORO](https://oro.open.ac.uk/40613); [UCL Discovery](https://discovery.ucl.ac.uk/1536711/)
- The word-learning advantage appeared for print books. With digital books, children learned the same number of words from personalized and non-personalized versions. — [Lit Hub summary](https://lithub.com/remember-personalized-childrens-books-new-studies-show-they-might-have-scientific-benefits)
- When parents and children read personalized books together, both showed significantly more smiles and laughs (observational study led by Kucirkova). — [Lit Hub](https://lithub.com/remember-personalized-childrens-books-new-studies-show-they-might-have-scientific-benefits)
- Caveat from Kucirkova: personalization "encourages self-focus," seen as more self-referential speech when children read personalized books. — [Psychology Today (Kucirkova)](https://www.psychologytoday.com/gb/blog/just-for-you/201809/personalized-books-for-children-what-you-need-to-know)
- A separate study (Kruse et al.) found no difference in children's understanding of a story's moral between personalized and non-personalized versions. — [Lit Hub](https://lithub.com/remember-personalized-childrens-books-new-studies-show-they-might-have-scientific-benefits)
- Kucirkova's overall conclusion from about 10 years of research: personalized books can benefit children "only if used thoughtfully." — [Lit Hub](https://lithub.com/remember-personalized-childrens-books-new-studies-show-they-might-have-scientific-benefits); see also the MIT Open Learning talk "Personalization and Young Children: the Good, the Bad, and the Ugly" — [MIT Open Learning](https://openlearning.mit.edu/events/personalization-and-young-children-good-bad-and-ugly)

**Context personalization in math (Walkington)**
- 145 ninth graders were surveyed about their interests (sports, music, movies) and randomly assigned to standard or interest-personalized linear-equation problems. The personalized group solved problems faster and more accurately. The biggest gain was in translating story situations into symbolic equations. The math structure was identical; only the context changed. — [EdWeek 2012](https://www.edweek.org/ew/articles/2012/09/26/05personalize_ep.h32.html); [SMU news](https://www.smu.edu/news/archives/2012/candace-a-walkington-educationweek-04oct2012)
- Gains persisted after personalization stopped. Previously personalized students did better on harder, non-personalized problems. — [KQED MindShift](https://kqed.org/mindshift/32768/to-learn-algebra-the-not-so-secret-way-to-students-hearts); [EurekAlert release](https://www.eurekalert.org/news-releases/518360)
- Walkington frames personalization as a scaffold for linking the "situation model" to the "problem model." She hypothesized that it works when problems sit at the edge of the learner's ability. Earlier math-personalization research had mixed results. Scaling is hard because "the bigger you make it, the harder it is to tap into students' interests." — [EdSurge 2013](https://edsurge.com/n/2013-11-08-can-interest-based-personalization-deliver-better-learning-outcomes); [ICLS 2012 paper](https://smu.edu/-/media/site/simmons/research/rme/docs/cw/icls_submission_2012_final.pdf)
- A later study personalized four units and found higher situational interest and better problem-solving efficiency. Personalization had to match the problem's depth and specificity to affect both interest and learning. — [EdSurge](https://edsurge.com/n/2013-11-08-can-interest-based-personalization-deliver-better-learning-outcomes)
- Follow-on work had students write their own interest-based "algebra stories" (problem-posing). Walkington predicted that personalization plus choice would have more impact. — [UTB catalog record](https://vufind.katalog.k.utb.cz/SummonRecord/FETCH-LOGICAL-c306t-9df7fe4e033e3a190218a1ab204b641d84600e378e3a7e4ee48aa58064fee2fe3)
- Effect sizes: the sources found were news coverage and abstracts, so they are not reported here.

**AI-generated personalized stories (2025 research)**
- StoryMate (CHI 2025) is an LLM interactive story-reading tool, studied with 12 children, 14 parents and 13 experts. Its engagement techniques include addressing children by name and weaving their interests into the dialogue. The study is qualitative, about perceptions. — [arXiv 2503.00590](https://arxiv.org/html/2503.00590v2)
- StoryLab (AIED 2025) generates stories toward teacher-set vocabulary, phonics and grammar goals. Its stories matched target vocabulary better than reference corpora (TF-IDF). The user study had only 4 teachers, with no measure of child outcomes. — [NSF PAR](https://par.nsf.gov/servlets/purl/10653121)
- Tailored Tales (ICCAI 2025) preference-tunes an LLM so stories meet Early Grade Reading Assessment criteria, evaluated for readability, coherence and engagement. — [MBZUAI repository](https://irep.mbzuai.ac.ae/items/3e3b7fd2-7847-45c5-93aa-c6bbd2f8be8a)
- Colin (CHI EA 2025) is a story co-creation system, studied with 20 participants. It measured engagement, understanding of cause and effect, and the quality of new stories. — [arXiv 2405.06495](https://arxiv.org/html/2405.06495v4)
- Overall, the 2025 AI-story literature is mostly small design studies. Controlled trials of child comprehension against conventional stories appear limited. — (synthesis of the sources above)

### Inferences
- **Design principle:** keep the learning content fixed and personalize the surface (names, pets, places, interests), the way Walkington did. Use personalization most where comprehension is the bottleneck: word problems for the 8.5-year-old, new vocabulary for the 2.5- and 5.5-year-olds.
- **Print vs. digital:** Kucirkova found no word-learning benefit for personalized digital books. That suggests the benefit may come from shared parent-child reading and conversation, not from the child's name on screen alone. The game should prompt co-play: "read this together with Abba," or printable personalized mini-books.
- **Self-focus:** to limit it, make family members, pets and friends the main characters in many scenes, not always the child. That also fits the "family world" concept.
- **Feature ideas, mapped to the evidence:**
  - Name and family insertion in stories and vocabulary scenes (Kucirkova). Example: "Grandma Rina's cat ate 3 of the 7 cookies."
  - Interest-based word problems (Walkington). The parent or child picks interests (dinosaurs, football, Elsa), and the LLM rewrites a fixed problem template with Hebrew and English versions.
  - Child-authored problems and stories. The 8.5-year-old writes problems for younger siblings (Walkington's problem-posing, plus a sibling co-op mode).
  - Stylized avatars from family photos; family members as NPCs. "Family world" map with home, Grandma's house, the gan (kindergarten), school and the park.
  - Parent-recorded praise and instructions, used sparingly so they stay special. Real-world rewards set by parents in the dashboard.
  - The child's drawings scanned into game assets (stickers or characters). This is a "co-creation" angle similar to Colin and StoryMate.
  - Grandparents take part remotely: they record a story, send a challenge, or receive a weekly "artwork" postcard.
- **Age bands:**
  - 2.5: voice-first, no text dependence, big tap targets, naming family members and pets, colors and counting.
  - 5.5: letters (otiyot) with nikud, phonics, simple English words, counting to 20.
  - 8.5: reading fluency without nikud, math word problems, English reading.

### Gaps
- Large replications of Kucirkova's word-learning result (n=18) were not found.
- The search did not surface specific published outcome data on Wonderbly/"Lost My Name" or studies of "self-reference effect" in young children. No numbers on these are reported here.
- Sandra Calvert's work, for example on parasocial relationships with familiar characters, was not retrieved in this session. Any claim about familiar characters boosting learning needs a separate check.
- No research was found on personalization with real photos of the child's own toys, pets or home specifically.
- No Hebrew-language personalization studies were found.

## Hebrew-capable AI services (TTS, nikud, ASR, LLMs, image generation)

### Takeaway
As of late 2026, Hebrew TTS is available from Azure (he-IL Hila/Avri neural voices), Google Cloud (he-IL added to Chirp 3: HD in Dec 2025), and ElevenLabs v3 (Hebrew listed among 74 languages; v4 is now its newest model). OpenAI TTS Hebrew support is indicated only by third parties, and its quality is reportedly weaker. For correct pronunciation of children's text, run Hebrew through a diacritizer (Dicta Nakdan, or open-source Phonikud/phonikud-onnx) before TTS. For speech recognition, ivrit.ai's open-source Hebrew Whisper fine-tunes are the state of the art for adult speech. No Hebrew child-speech benchmarks were found. For consistent cartoon characters from photos, Gemini 2.5 Flash Image ("Nano Banana") leads image-editing leaderboards, but independent identity-drift tests are lacking.

### Cited Findings
**Text-to-speech**
- **Azure:**
  - Hebrew was added to Neural TTS in the Ignite 2020 update with the female voice he-IL-HilaNeural. — [Microsoft Tech Community, Ignite 2020](https://techcommunity.microsoft.com/t5/azure-ai-blog/ignite-2020-neural-tts-updates-new-language-support-more-voices/ba-p/1698544)
  - The male voice he-IL-AvriNeural was added in December 2020. — [Microsoft Tech Community, 51 new voices](https://techcommunity.microsoft.com/t5/azure-ai/azure-neural-text-to-speech-updates-51-new-voices-added-to-the/ba-p/1988418)
  - The old standard voice (Asaf) is retired; standard voices were unavailable after Aug 31, 2024. — [Microsoft Learn migration guide](https://learn.microsoft.com/he-il/azure/ai-services/speech-service/how-to-migrate-to-prebuilt-neural-voice)
  - SSML (pauses, emphasis) works with Hila and Avri (third-party). — [JSON2Video](https://json2video.com/ai-voices/azure/voices/he-il-hilaneural)
  - Whether newer he-IL voices exist (e.g., HD/multilingual voices) was NOT verified, because the language-support page fetch was blocked.
- **Google Cloud TTS:**
  - Chirp 3: HD lists Hebrew (Israel) he-IL. The release notes dated Dec 11, 2025 added he-IL along with bg-BG, cs-CZ, el-GR and others.
  - Regions are listed as global/us/eu GA.
  - Specific Hebrew voice names were not confirmed in this session. — [Google Chirp 3: HD docs](https://docs.cloud.google.com/text-to-speech/docs/chirp3-hd); [Cloud TTS release notes](https://docs.cloud.google.com/text-to-speech/docs/release-notes)
- **ElevenLabs:**
  - The help center says Eleven v3 supports 74 languages, and the language list includes Hebrew. — [ElevenLabs: What languages do you support?](https://elevenlabs.io/docs/help-center/other/what-languages-do-you-support); [ElevenLabs models help article](https://help.elevenlabs.io/hc/en-us/articles/17883183930129-What-models-do-you-offer-and-what-is-the-difference-between-them)
  - Current docs describe Eleven v4 as the latest model and v3 as "previous." v4's Hebrew coverage was not verified. — same sources (search summary)
- **OpenAI TTS:**
  - Retell AI's language-support docs list he-IL as supported for OpenAI voices, including tts-1 and gpt-4o-mini-tts. — [Retell AI language support](https://docs.retellai.com/build/language-support)
  - Third-party reviews describe OpenAI voices as English-centric. Earlier models' non-English output "was not that good" and had an American accent. — [CallMissed](https://www.callmissed.com/en/models/gpt-4o-mini-tts)
  - The official OpenAI language list was not retrieved.

**Automatic nikud and pronunciation**
- **Dicta Nakdan:**
  - It is a hybrid neural plus curated-lexicon diacritizer (Shmidman et al., ACL 2020) supporting Modern, Rabbinic and Poetic Hebrew. — [arXiv 2005.03312](https://arxiv.org/pdf/2005.03312)
  - Reported Modern Hebrew accuracy: 88.23% word-level and 95.12% character-level, ahead of Morfix and Snopi. — [Liner review of the paper](https://liner.com/review/nakdan-professional-hebrew-diacritizer)
  - A web tool exists (nakdan.dicta.org.il) and a developer page (dicta.org.il/developers). Endpoint format, rate limits and terms were not verified. — [hebrew-nlp-toolkit guide](https://claudeskills.info/skills/skills-il/localization/hebrew-nlp-toolkit/)
  - Other papers report correcting Dicta outputs via "Dicta's API." — [arXiv 2402.00075 (D-Nikud)](https://arxiv.org/html/2402.00075)
  - Dicta also publishes an open character-level model, dicta-il/dictabert-large-char-menaked. Per one guide, it is generally outperformed by the hosted Nakdan for sentence-level vocalization. — [hebrew-nlp-toolkit guide](https://claudeskills.info/skills/skills-il/localization/hebrew-nlp-toolkit/)
  - Nakdan-based tools warn of errors on rare or ambiguous words and work best on full sentences. — [Milon Avneyon nikud tool](https://milononline.net/%D7%9B%D7%9C%D7%99%D7%9D/%D7%94%D7%95%D7%A1%D7%A4%D7%AA-%D7%A0%D7%99%D7%A7%D7%95%D7%93?lang=en)
- **Phonikud:**
  - An open-source Hebrew grapheme-to-phoneme system for real-time TTS (arXiv 2506.12311, June 2025).
  - It adds nikud to unvocalized text, plus enhanced marks for stress and vocal shva, then outputs IPA.
  - Its authors report more accurate phoneme prediction than prior methods. — [arXiv 2506.12311](https://arxiv.org/abs/2506.12311); [project page](https://thewh1teagle.github.io/phonikud-paper)
  - PyPI 0.4.0: `phonemize('שָׁלוֹם עוֹלָם')` returns `ʃalˈom olˈam`. Diacritization uses phonikud-onnx.
  - Known issues: names and non-Hebrew words are sometimes wrong, and stress can vary for identical nikud. Inline phoneme overrides are supported. The G2P code is CC BY 4.0. — [PyPI phonikud](https://pypi.org/project/phonikud/0.4.0/)
  - A live Hebrew TTS demo exists. — [HF Space](https://thewh1teagle-phonikud-tts.hf.space)

**Speech recognition (Hebrew)**
- ivrit.ai's Interspeech 2025 paper reports a new open-source Hebrew Whisper model with up to 29% lower error rates than existing Whisper solutions, trained on manually corrected crowd transcriptions. — [Interspeech 2025 (Marmor et al.)](https://www.isca-archive.org/interspeech_2025/marmor25_interspeech.pdf)
- ivrit-ai/whisper-large-v3 is a Hebrew continued-training model (Apache-2.0, trained April 2025). Data: about 300 h crowd-transcribed audio, about 50 h of read Wikipedia, and about 4,700 h of Knesset plenum recordings. — [HF model card](https://huggingface.co/ivrit-ai/whisper-large-v3)
- ivrit-ai/whisper-large-v3-turbo-ct2 is a faster-whisper/CTranslate2 build trained on 295 h volunteer plus 93 h professional transcriptions. — [HF model card](https://huggingface.co/ivrit-ai/whisper-large-v3-turbo-ct2)
- The training data is adult speech (Knesset, volunteers). No child-speech data or benchmarks were mentioned. — (same model cards)

**Image generation and character consistency**
- Gemini 2.5 Flash Image ("Nano Banana") is reported to top LMArena image-editing rankings, about 170 Elo above Flux Kontext per one blog. It is described as strong at keeping a character consistent across scenes. — [xagi-labs blog](https://xagi-labs.github.io/blog/nano-banana-is-gemini25flashimage-beating-flux-kontext-by-170-elo-with-sota-consistency-editing-and-multi-image-fusion); [laozhang.ai comparison](https://blog.laozhang.ai/api-comparison/gemini-vs-flux-image/)
- A practitioner view: Flux is better for pose and layout control; Gemini is better for consistent identity across styles and multi-image characters. The reported Gemini price is $0.039 per image. — [laozhang.ai comparison](https://blog.laozhang.ai/api-comparison/gemini-vs-flux-image/)
- These are leaderboard and blog claims. No independent, reproducible identity-drift test was found. — (same sources)

### Inferences
- **Recommended Hebrew audio pipeline:**
  1. The LLM generates Hebrew text with nikud requested. Do not trust LLM nikud blindly.
  2. Run the text through Dicta Nakdan, or Phonikud for an offline/open option.
  3. Have a parent review it once.
  4. Synthesize with Azure he-IL or Google Chirp 3 HD he-IL. Both use cloud APIs with stable he-IL locale codes.
  5. **Pre-generate and cache audio files** rather than calling TTS live. This makes the game work offline, cuts cost, and lets a parent QA pronunciation.
  For a 5.5-year-old learning letters, phoneme-level control matters, so prefer SSML (Azure) or Phonikud IPA overrides.
- **Parent voice:** record praise and instructions natively in the app. Don't clone the parent's voice with a third-party service, which would send biometric voice data off-device (see the privacy section). ElevenLabs voice cloning is technically possible but adds privacy exposure.
- **Child ASR:** Whisper-family models trained on adult speech will likely do poorly on a 2.5-year-old's speech. Design voice features as "parent-confirmed" or use simple keyword matching, not graded pronunciation scoring. Running ivrit.ai Whisper on-device in the browser (e.g., via WebGPU/transformers.js or whisper.cpp) is plausible, but browser feasibility for large-v3 was not verified.
- **LLM content:** generate content offline, in batches, at "authoring time." The parent reviews it, and the game ships static JSON. Children never chat with a live LLM. This also avoids sending child data to an LLM vendor at runtime.
- **Character art:** create avatars once from a small number of photos, or from the child's own drawings, then reuse the stylized avatar as the reference image for all future generations. That way the real photo is uploaded only once, or never if you start from a drawing or a description.

### Gaps
- Hebrew-generation quality comparisons of Claude, GPT, Gemini and DictaLM were not retrieved this session; no benchmark claims are made here. The DictaLM version and license are unverified.
- The current Azure he-IL voice list (any HD or "multilingual" voices added after 2021) is unverified because learn.microsoft.com was blocked.
- ElevenLabs v4's Hebrew support and ElevenLabs Scribe Hebrew accuracy are unverified.
- OpenAI's official TTS language list is not confirmed; only third-party evidence was found.
- No Hebrew child-speech ASR benchmark (any vendor) was found.
- Pricing for Azure, Google and ElevenLabs was not collected.
- GPT Image character consistency and data-retention terms were not researched.

## App platform choice for a solo parent builder (PWA vs Expo vs Flutter vs Unity/Godot; RTL; web game libraries; storage)

### Takeaway
Only React Native/Expo RTL guidance was retrieved; it confirms RTL works there but carries restart and platform quirks. The rest of this section is inference. For a solo parent with AI coding help, building one family's tablets, a **PWA (or plain web app) with a web game library** is the lowest-friction path. CSS `dir="rtl"` and logical properties handle Hebrew layout natively. Expo is the fallback if native app-store distribution or deeper device APIs are needed.

### Cited Findings
- **Expo/React Native RTL:**
  - Typical setup: i18next for translations, AsyncStorage for the chosen language, and `I18nManager.allowRTL`/`forceRTL` for direction. — [DEV Community / GeekyAnts](https://dev.to/geekyants-inc/implementing-rtl-right-to-left-in-react-native-expo-a-step-by-step-guide-m01)
  - Changing direction usually needs an app restart (`RNRestart.Restart()` after `forceRTL`). A no-restart approach re-renders the tree with a changed `key`. — [GeekyAnts](https://geekyants.com/blog/implementing-rtl-right-to-left-in-react-native-expo---a-step-by-step-guide); [GeekyAnts no-restart](https://geekyants.com/blog/implementing-right-to-left-rtl-support-in-expo-without-restarting-the-app)
  - iOS needs a full restart; the guide says this was addressed in RN 0.79.0, with patch-package needed for older versions. This is a community claim. — [GeekyAnts](https://geekyants.com/en-ae/blog/implementing-rtl-right-to-left-in-react-native-expo---a-step-by-step-guide)
  - Use logical style properties (`paddingStart`/`paddingEnd`) and flip `flexDirection`/`textAlign` based on `I18nManager.isRTL`. — [GeekyAnts techblog](https://techblog.geekyants.com/implementing-rtl-right-to-left-in-react-native-expo-a-step-by-step-guide.md)

### Inferences
These come from general knowledge, NOT verified in this session.
- **Bilingual layout:** this app mixes Hebrew (RTL) and English (LTR). A web stack's per-element `dir` attribute makes mixed-direction screens easy. React Native's app-global `I18nManager` model is clumsier for switching mid-session.
- **Game layer:**
  - Phaser 3 (2D game framework) or PixiJS (renderer) for game scenes.
  - Rive or Lottie for character animation.
  - Howler.js for audio sprites of the cached TTS and parent recordings.
  - Draw Hebrew text in DOM overlays rather than canvas text, to avoid bidi and nikud rendering issues.
- **PWA caveats to verify:**
  - iOS/iPadOS storage eviction for PWAs.
  - Microphone permission inside an installed PWA.
  - Audio autoplay rules (sound needs a user gesture to unlock).
- **Unity/Godot:** heavier for a solo dev. Both have historically had weak RTL/Hebrew text shaping (Unity needs plugins; Godot 4 improved with its TextServer). Only worth it if the game needs complex physics or 3D.
- **Flutter:** has solid built-in RTL (`Directionality`). It is a reasonable alternative, but AI coding assistants tend to have the most training data for React/TS web stacks.
- **Storage:**
  - Local-first: IndexedDB in the browser for child profiles, progress and cached media. Photos and voice recordings never leave the device by default.
  - Optional sync: Supabase (Postgres plus row-level security) or Firebase for the parent dashboard across devices and for grandparents' remote participation.
  - A family-only app can use one family account with child sub-profiles. No child logins are needed.

### Gaps
- No official docs for Phaser, PixiJS, Rive, Howler, MDN PWA, Flutter RTL or Godot RTL were fetched. Those platform claims are unsourced inferences and need checking.
- 2026 iOS PWA capabilities (storage quotas, push, install UX) were not verified.

## Privacy and safety for children's photos and voice data

### Takeaway
Children's faces and voices count as regulated biometric/personal data. The amended US COPPA Rule (compliance deadline April 22, 2026) adds voiceprints and faceprints to "personal information." Israel's Amendment 13 to the Privacy Protection Law took effect Aug 14, 2025, with GDPR-like enforcement and fines. A private family app built by a parent for their own kids is likely outside the scope of these commercial-operator rules (inference, not legal advice). But the safest design is the same either way: process locally, avoid sending faces or voices to third parties, and use vendor terms that exclude training on inputs.

### Cited Findings
- **US COPPA:**
  - The FTC published the final amendments on April 22, 2025. They took effect June 23, 2025, with a compliance deadline of April 22, 2026 for most provisions. — [Hunton](https://hunton.com/privacy-and-information-security-law/ftc-publishes-final-coppa-rule-amendments); [BBB Programs](https://bbbprograms.org/media/insights/blog/coppa-amended)
  - One source gives Jan 16, 2025 as the date the FTC finalized the amendments. That was the vote/announcement; publication in the Federal Register followed in April. — [JD Supra](https://jdsupra.com/legalnews/the-ftc-finalizes-changes-to-the-coppa-5661286)
  - Biometric identifiers are added to "personal information": fingerprints, retina/iris patterns, genetic data, voiceprints, gait, facial templates/faceprints, when usable for automated recognition. — [PRIVO](https://www.privo.com/blog/understanding-new-coppa-amendments); [National Law Review](https://natlawreview.com/article/ftc-publishes-final-coppa-rule-amendments)
- **Israel:**
  - Amendment 13 entered into force Aug 14, 2025, after the Knesset passed it in August 2024. It moves Israeli law closer to the GDPR. — [Digital Policy Alert](https://digitalpolicyalert.org/event/32795-privacy-protection-bill-amendment-no-13-formerly-no-14-entered-into-force); [IAPP](https://iapp.org/news/a/israel-marks-a-new-era-in-privacy-law-amendment-13-ushers-in-sweeping-reform)
  - It strengthens the Privacy Protection Authority with administrative fines and requires privacy protection officers in some cases. — [Pearl Cohen](https://www.pearlcohen.com/israel-significant-amendment-to-the-privacy-law-takes-effect/); [Goldfarb](https://www.goldfarb.com/amendment-13-to-the-israeli-privacy-protection-law/)
  - Biometric data under Amendment 13 includes facial geometry and voiceprints, per a secondary vendor source. — [Kiteworks](https://www.kiteworks.com/regulatory-compliance/israeli-hmos-amendment-13-compliance/)

### Inferences
- **Architecture:** default to on-device for photos, drawings and voice recordings (IndexedDB, never uploaded).
- **Photo-based avatars:** if cloud image generation is needed, either:
  - (a) upload once, via a paid API tier whose terms exclude training and limit retention, then delete; or
  - (b) skip the photo entirely by generating from a text description or the child's drawing.
- **Parent voice:** record it directly instead of cloning it, which avoids creating a voiceprint with a vendor.
- **If the app is ever shared beyond the family**, COPPA, Amendment 13 and GDPR Art. 8 (consent age 13 to 16 depending on country) obligations apply: verifiable parental consent, data minimization, and retention limits.

### Gaps
- No children-specific provisions of Israel's Amendment 13 (minors' consent age, special protections) were found. Check gov.il/privacy.
- GDPR-K / UK Age Appropriate Design Code details were not researched.
- Retention and training policies of Google (Gemini API), OpenAI, ElevenLabs and Azure for image and voice inputs were not verified.

## Parent dashboard best practices (metrics, reports, goals, rewards)

### Takeaway
No sources on parent-dashboard best practices were retrieved in this session; everything below is design inference built on the findings above. Show parents mastery by skill, not just time or streaks. Surface the skills a child is struggling with, along with concrete "do this at home" prompts that reinforce the shared-reading benefit Kucirkova found. Let parents set real-world rewards tied to goals.

### Cited Findings
- None retrieved specifically on dashboards. The relevant indirect evidence comes from the first section: the personalization benefit appeared in shared parent-child print reading, not solo digital reading. — [Lit Hub](https://lithub.com/remember-personalized-childrens-books-new-studies-show-they-might-have-scientific-benefits)

### Inferences
- **Per-child mastery map** by skill:
  - 2.5: colors, counting to 5, family names, animal sounds.
  - 5.5: Hebrew letters and nikud vowels, English letters, counting and adding to 10.
  - 8.5: reading without nikud, word problems, English vocabulary.
  Show each skill as not started / practicing / mastered, based on accuracy over the last N attempts.
- **"Needs help" list:** skills with repeated errors, plus an offline activity suggestion (e.g., "practice ש/ס with a bedtime book").
- **Time played** per day or week, with parent-set daily limits. Use soft streaks that don't punish missed days, since the children are very young.
- **Weekly report:** a short summary plus one highlight artifact (a drawing, a recorded sentence). It can be shared with grandparents.
- **Goals and rewards:** the parent sets goals (e.g., "master 5 new letters") and a real reward (park trip, ice cream). The child sees progress toward it in the game.
- **Content review queue:** the parent approves AI-generated stories and problems, and the nikud and audio, before they reach the child.

### Gaps
- No research or vendor guidance was gathered on effective parent dashboards, on reward schemes and intrinsic motivation (e.g., the overjustification effect) for these ages, or on screen-time recommendations. A dedicated search is needed.
