# iOS Technical Architecture (Nov 2026) for a Hebrew-first Kids' Learning Game, Built by a Designer-Parent with AI Coding Agents

Research notes, as of 2026-11-02. Method note: the session's network blocked direct fetches of apple.com, 9to5mac.com and the JS-rendered pages on developer.apple.com, so Apple's feature-availability pages and the framework language lists could not be read first-hand. Language-support claims below come from search-result summaries of those pages plus third-party write-ups, and each is marked by confidence. Anything not backed by a source is under Inferences (my engineering judgement) or Gaps. Context: by Nov 2026 the shipping OS is iOS/iPadOS 27 (released 14 Sep 2026 per [tbreak](https://tbreak.com/apple-intelligence-arabic-uae-watch-hypertension/)), so "iOS 26 era" APIs (Foundation Models, SpeechAnalyzer) are now one release old and are the baseline.

---

## 1. Apple-platform frameworks and Hebrew language support (UI/animation, game scenes, Rive/Lottie, haptics, audio, PencilKit, Vision OCR, SpeechAnalyzer, Foundation Models, Apple Intelligence)

### Takeaway
Plan for **no on-device Apple LLM in Hebrew**. Apple Intelligence, and so the Foundation Models on-device model, still doesn't support Hebrew in iOS 27. **Speech-to-text in Hebrew looks available** through SpeechTranscriber (he_IL reported in supportedLocales), with DictationTranscriber as the fallback. **Hebrew OCR through Vision is not confirmed**: older lists don't include it, so don't build core gameplay on it. Build the game in SwiftUI with SpriteKit/Canvas scenes, add Rive for designer-made interactive characters, and use the Apple-native haptics, audio and PencilKit stacks.

### Cited Findings
**Apple Intelligence / Foundation Models: Hebrew**
- Apple's support page for the next generation of Apple Intelligence lists English, Danish, Dutch, French, German, Italian, Norwegian, Portuguese, Spanish, Swedish, Turkish, Vietnamese, Chinese, Japanese and Korean. Hebrew is not on the list. — [Apple Support 121115 (via search summary)](https://support.apple.com/en-il/121115)
- At the iOS 27 launch (14 Sep 2026), Apple Intelligence supported 16 languages. Arabic was not among them, and Apple has not said whether or when it will be added. I found no coverage of Hebrew being added. — [tbreak](https://tbreak.com/apple-intelligence-arabic-uae-watch-hypertension/)
- An Apple Community thread asks for Hebrew support in Siri and Apple Intelligence. That's a user request with no Apple commitment. — [Apple Community](https://discussions.apple.com/thread/256012004)
- Hebrew arrived in the **Translate app** in iOS 27 (one of 9 new languages in the beta; first-ever Hebrew support in Translate). — [Cult of Mac](https://www.cultofmac.com/news/apple-translate-ios-27-9-new-language-support); [heise](https://www.heise.de/en/news/iOS-27-Apple-expands-languages-in-Translate-app-11359469.html); [itsbaba](https://news.itsbaba.com/story/39245/apple-adds-hebrew-language-support-to-ios-27-translation-fea-39912)
- Apple's iOS 27 feature-availability page lists Hebrew (Israel) for VoiceOver, Voice Control, Live Speech and Read & Speak, which are accessibility features, not Apple Intelligence. — [Apple iOS feature availability (via search summary)](https://www.apple.com/ios/feature-availability/)
- Foundation Models API shape: `SystemLanguageModel.default.availability` must be checked (it's unavailable on ineligible devices, when Apple Intelligence is off, or while the model downloads). `supportedLanguages` is queried at runtime. Guided generation uses `@Generable`/`@Guide`. There is a `Tool` protocol for tool calling, and the context window is about 4K tokens. — [Apple docs: SystemLanguageModel](https://developer.apple.com/documentation/foundationmodels/systemlanguagemodel). The page didn't render; these details came from the fetch tool's model knowledge, so treat them as medium confidence.

**SpeechAnalyzer / SpeechTranscriber**
- `SpeechTranscriber.supportedLocales` lists the locales the transcriber can handle, including downloadable ones not yet installed. The array is empty if the device doesn't support the transcriber. `installedLocales` is separate. — [Apple docs: supportedLocales](https://developer.apple.com/documentation/speech/speechtranscriber/supportedlocales.md)
- Use `SpeechTranscriber.supportedLocale(equivalentTo:)` to map a locale. Developers reported empty lists on some betas and on the Simulator. — [Apple Developer Forums](https://developer.apple.com/forums/thread/790108?page=2)
- At launch the list was short (Cantonese, Chinese, English, French, German, Italian, Japanese, Korean, Portuguese, Spanish). — [DEV Community](https://dev.to/szp2005/speechtranscriber-covers-about-ten-languages-and-we-were-telling-everyone-else-their-language-4dg9)
- A Substack locale snapshot includes **he_IL** in SpeechTranscriber's supported locales. — [Anton Gubarenko](https://antongubarenko.substack.com/p/ios-26-speechanalyzer-guide)
- As of Aug 2026, supportedLocales reportedly returns 42 locales across 22 languages, **including Hebrew**. — [LoroNote](https://loronote.com/en/blog/apple-speechanalyzer-vs-whisper). Third-party source; these two sources conflict with the launch-era list, which suggests the list grew.
- DictationTranscriber is Apple's documented fallback for unsupported languages or devices. One app labels dictation-only languages "(basic quality)". — [DEV Community](https://dev.to/szp2005/speechtranscriber-covers-about-ten-languages-and-we-were-telling-everyone-else-their-language-4dg9); [Apple docs: DictationTranscriber](https://developer.apple.com/documentation/speech/dictationtranscriber.md)
- WWDC25 session 277 introduced SpeechAnalyzer. — [WWDC25 277](https://developer.apple.com/videos/play/wwdc2025/277/)

**Vision text recognition**
- `VNRecognizeTextRequest.supportedRecognitionLanguages()` returns the supported list, which depends on the revision and recognition level. — [Apple docs](https://developer.apple.com/documentation/vision/vnrecognizetextrequest/supportedrecognitionlanguages().md)
- A printed list from a pre-iOS 26 snapshot has en, fr, it, de, es, pt-BR, zh-Hans/Hant, yue, ko, ja, ru, uk. **No Hebrew.** — [zenn.dev](https://zenn.dev/_ds_store/articles/0bb40f96eca3e7)
- iOS 26 added `RecognizeDocumentsRequest` with a `supportedRecognitionLanguages` property. I couldn't retrieve its actual list. — [Apple docs](https://developer.apple.com/documentation/vision/recognizedocumentsrequest/supportedrecognitionlanguages.md)

**Rive iOS runtime**
- Rive's Apple runtime now has two API generations. The legacy one is `RiveViewModel`/`RiveView` on the main thread. The new one uses explicit `File` + `Rive` objects with `Worker` objects for background processing, and APIs that throw. In SwiftUI it uses `RiveUIViewRepresentable` / `AsyncRiveUIViewRepresentable`. — [Rive docs: migrating from legacy](https://rive.app/docs/runtimes/apple/migrating-from-legacy)
- **Data binding**: by default a Rive object binds to the default view-model instance for its artboard, and properties can be read and updated at runtime. Global View Models allow shared bindings. — [Rive Apple runtime docs](https://rive.app/docs/runtimes/apple)
- Supports iOS 14+, visionOS 1+, tvOS 16+, macOS 13.1+. — [Rive docs](https://rive.app/docs/runtimes/apple/apple)

### Inferences
- **UI/animation layer (high confidence, general platform knowledge, not re-verified this session):** SwiftUI's `PhaseAnimator`/`KeyframeAnimator` (iOS 17+), spring animations, `Canvas` + `TimelineView` for particle and trace effects, and `matchedGeometryEffect` cover most "juicy" kid-UI motion without a game engine. RTL comes almost free in SwiftUI with `.environment(\.layoutDirection, .rightToLeft)` / the he locale, but audit custom `Canvas` drawing, SpriteKit coordinates and directional arrows by hand, because those don't mirror.
- **Game scenes:** use **SpriteKit embedded through `SpriteView`** for drag/drop, physics, letter-tracing playfields and sprite atlases. It's native, small, works offline, and Claude Code knows it well. RealityKit is overkill for 2D. Unity adds a large binary, a C# toolchain and a license, and SwiftUI interop is awkward. Godot 4 iOS export works but splits the codebase and makes it harder for AI agents to stay in one language. For a designer-parent working with an AI agent, **one language (Swift) and one IDE** is the biggest productivity lever.
- **Rive vs Lottie:** Rive fits interactive mascots that react to correct/incorrect answers (state machines plus data binding to drive, say, a score or the mascot's mood from Swift). Lottie fits one-shot celebratory animations exported from After Effects. Since the builder is a designer, the Rive editor lets them own the animation logic without code. Pin the new Rive runtime API, because the legacy one is being superseded.
- **Haptics/audio:** Core Haptics `CHHapticEngine` with AHAP files for custom patterns, and `UIImpactFeedbackGenerator`/SwiftUI `.sensoryFeedback` for simple taps. `AVAudioEngine` with several `AVAudioPlayerNode`s (music bed, SFX, voice-over), each with its own mixer volume, handles ducking music under narration. Set the session to `.playback` with `.mixWithOthers` decided per policy.
- **PencilKit:** `PKCanvasView` captures strokes for Hebrew letter tracing. Score tracing **geometrically yourself** (compare strokes against reference paths and stroke order) instead of relying on OCR, since Vision Hebrew OCR isn't confirmed.
- **Hebrew AI features must be server-side:** an LLM for generating content, any "conversation" with the mascot, and OCR fallback if needed. You can't count on the on-device Foundation Models for Hebrew. Use it only for optional English features, and check `supportedLanguages` at runtime in case Hebrew arrives later.
- **Speech:** a "say the word" pronunciation game is feasible on-device with SpeechTranscriber he_IL, but children's speech accuracy is unknown. Test on the actual kids, and design the game so that misrecognition never punishes the child (soft feedback, parent override).

### Gaps
- Couldn't read Apple's official feature-availability page or Live Text language list directly (network block). Whether Live Text/Vision supports Hebrew in iOS 27 is **unverified**. Run `supportedRecognitionLanguages()` on a device.
- No official Apple list of SpeechTranscriber locales was retrievable. Hebrew support rests on two third-party snapshots. Check `supportedLocales` on device.
- No Apple roadmap for Hebrew in Apple Intelligence was found.
- I didn't find whether the iOS 27 Translate Hebrew support is exposed to developers through the Translation framework (`TranslationSession`).

---

## 2. Data and sync for a family (SwiftData + CloudKit vs Supabase/Firebase; profiles; offline-first)

### Takeaway
For 5 people on Apple devices, use **SwiftData (local, offline-first) mirrored to the parent's private CloudKit database**, with child profiles stored as records inside one family "owner" account. Keep the content packs and the AI proxy on a separate small server or CDN. A Supabase/Firebase backend only pays off if Android or web clients or cross-Apple-ID real-time features become necessary.

### Cited Findings
- I couldn't retrieve CloudKit quotas or pricing, or Supabase 2026 free-tier details, this session (the search returned Cloudflare results only). — [search attempt; no primary source retrieved](https://developers.cloudflare.com/workers/platform/pricing/index.md)

### Inferences
- **SwiftData + CloudKit** (`ModelConfiguration(cloudKitDatabase: .private("iCloud.<id>"))`) syncs automatically across devices **signed into the same Apple ID**. It's free to the developer for private-database storage (counted against the user's iCloud quota), needs no server, and works offline-first. Constraints: SwiftData CloudKit sync needs every property optional or defaulted, no unique constraints, and optional relationships. It also only syncs the **private** database. Sharing across different Apple IDs, such as each child's own Apple ID under Family Sharing, needs `CKShare`/the shared database, which SwiftData doesn't support natively. You'd need Core Data + `NSPersistentCloudKitContainer` sharing, or raw CloudKit. This is from general platform knowledge, not verified in a source this session.
- **Recommended profile model:** one app-level "family" owned by a parent's Apple ID, and `ChildProfile` entities (avatar, age, reading level, progress) chosen from a kid-friendly profile picker. A shared iPad works naturally. On a child's own device under a separate child Apple ID, either (a) sign that device's app into a family sync through your own backend, or (b) implement CKShare via Core Data. For 3 kids, the simplest MVP is a **profile picker on each device plus progress synced through the parent's iCloud**, which works when the kids' iPads use the family-managed account or when progress sync goes through the small backend instead.
- **Offline-first:** content packs download into the app container and are versioned. Gameplay never blocks on the network, and AI-generated content is pre-generated and cached, never generated live while a child waits.
- **Supabase/Firebase:** add these only if you need cross-Apple-ID sync without CKShare complexity, a web parent dashboard, or Android. Supabase gives Postgres + auth + storage + edge functions in one place. The cost is that child data then sits on a third-party server, which raises privacy and Kids Category questions.

### Gaps
- Current CloudKit quotas/pricing and Supabase/Firebase 2026 pricing weren't retrieved.
- I didn't verify whether SwiftData gained native CKShare/shared-database support in iOS 26/27.

---

## 3. AI backend (proxy, models, structured output, caching, batching, TTS caching, CDN, monthly cost)

### Takeaway
Use a **Cloudflare Worker ($5/mo paid plan) as an authenticated proxy** holding the API keys, with **R2 (zero egress) as the CDN for content packs and cached TTS audio**. Generate content in batches with Claude (Sonnet 5.5 for quality, Haiku 5.5 for volume), using structured outputs and prompt caching. Pre-render Hebrew TTS once and cache it forever. Realistic family cost: **about $10–40/month** plus the $99/yr developer fee.

### Cited Findings
**Claude API pricing (Anthropic first-party, cached 2026-10-06 in Anthropic's claude-api reference; per million tokens)** — [Anthropic pricing docs](https://platform.claude.com/docs/en/about-claude/pricing)
- Claude Haiku 5.5 (`claude-haiku-5-5`): $0.10 input / $0.50 output, for prompts up to 100K tokens ($0.50/$2.50 beyond).
- Claude Sonnet 5.5 (`claude-sonnet-5-5`): $2 / $10, cache reads $0.20, 1M context.
- Claude Opus 5.5 (`claude-opus-5-5`): $4 / $20, cache reads $0.20.
- Haiku 4.5 is still listed at $1 / $5.
- Message Batches API: asynchronous, about 50% cost; results come back in any order and are keyed by `custom_id`. — [Anthropic docs](https://platform.claude.com/docs/en/about-claude/pricing); third-party confirmations of the 50% batch discount and of stacking with caching: [morphllm](https://www.morphllm.com/anthropic-claude-api-pricing), [finout](https://finout.io/blog/claude-pricing-in-2026-for-individuals-organizations-and-developers)
- Prompt caching is a prefix match: keep the stable system prompt and schema first. The minimum cacheable prefix is 512–4096 tokens depending on the model. Cache reads are about a 90% discount (Haiku 4.5 example: write $1.25, read $0.10/MTok). — [Anthropic docs](https://platform.claude.com/docs/en/about-claude/pricing); [third-party summary](https://www.morphllm.com/anthropic-claude-api-pricing)
- Structured outputs: `output_config: {format: {...}}` on Messages (the old `output_format` is deprecated), or `strict: true` on tools. SDK `messages.parse()` validates against the schema. — [Anthropic docs](https://platform.claude.com/docs/en/about-claude/pricing)
- Assistant prefill now returns 400 on current models, so use structured outputs for JSON. — [Anthropic docs](https://platform.claude.com/docs/en/about-claude/pricing)

**Hebrew TTS**
- A Hebrew TTS comparison (Mar 2025, small and dated) found MiniMax best, Edge TTS and Gemini good, and ElevenLabs good on v3. **ElevenLabs needs a Hebrew language code: multilingual v2 was unintelligible in Hebrew.** — [danielrosehill/Hebrew-TTS-Providers](https://github.com/danielrosehill/Hebrew-TTS-Providers)
- ElevenLabs subscription tiers: Free 10K chars/mo, Starter $5 (30K), Creator $22 (100K, commercial), Pro $99 (500K), Scale $330 (2M). Sources disagree on per-character API rates. — [forasoft](https://www.forasoft.com/blog/article/synthetic-voice-library-apps); [ElevenLabs Hebrew TTS page](https://elevenlabs.io/text-to-speech/hebrew)
- Azure AI Speech Neural: about $15–16 per 1M characters, 500K characters/month free for neural voices (sources vary: $15 vs $16). — [dev.to TTS pricing 2026](https://dev.to/leanvox/tts-api-pricing-in-2026-i-went-through-every-provider-so-you-dont-have-to-bem); [LiteLLM Azure docs](https://docs.litellm.ai/docs/providers/azure_ai_speech)
- Gemini TTS (gemini-2.5-flash-preview-tts) has 30 prebuilt voices. I found no Hebrew-specific pricing. — [dev.to](https://dev.to/leanvox/tts-api-pricing-in-2026-i-went-through-every-provider-so-you-dont-have-to-bem)

**Cloudflare**
- Workers Paid plan: $5/month minimum, covering Workers, Pages Functions, KV, Hyperdrive and Durable Objects. Includes about 10M requests and 30M CPU-ms per month. — [Cloudflare Workers pricing](https://developers.cloudflare.com/workers/platform/pricing/index.md)
- Overage is about $0.30 per million requests and $0.02 per million CPU-ms (one source says $0.50/M requests). Free plan: 100K requests/day. R2 storage: 10 GB included, $0.015/GB-month beyond, zero egress. — [makerkit calculator](https://makerkit.dev/pricing-calculator/cloudflare); [costbench](https://costbench.com/software/cdn-edge/cloudflare/hidden-costs/) (secondary sources)

### Inferences
- **Architecture:** iOS app → (App Attest / DeviceCheck token plus a per-family secret issued at onboarding) → Cloudflare Worker → Claude API (key held as a Worker secret, never in the app). The Worker validates a JSON schema, rate-limits per device (KV/Durable Object), and writes outputs to R2. The app pulls packs from R2 through a CDN URL. Supabase Edge Functions or Vercel would work equally well. Cloudflare wins on zero egress for audio and on its $5 flat price.
- **Generation pattern:** a parent asks for "a week of animal-themed letter games for a 5-year-old". The Worker submits a **Batch** job, using structured output with the item-bank schema and a cached system prompt that holds the curriculum, style guide and Hebrew niqqud rules. The parent reviews and approves in a parent mode, and approved items are published to the pack. Live, real-time generation is reserved for optional parent-triggered features.
- **Hebrew quality:** LLM Hebrew output with **niqqud (vowel points)** for early readers is error-prone. Use the best model (Sonnet 5.5 or Opus 5.5) for content, add an automated second-pass validation, and consider Dicta's Hebrew tools/models for niqqud checking ([DictaLM paper](https://arxiv.org/abs/2309.14568v1) shows Hebrew-specific models exist). Parent review stays the final gate.
- **TTS caching:** hash (text + voice + model + niqqud) into an R2 key and store the mp3/aac. Most kid-app audio is a small reusable vocabulary (letters, numbers, praise phrases, instructions), so after the first month TTS spend falls to near zero. Narration needs niqqud-aware or phonetically spelled input, because Hebrew TTS mispronounces unvoweled homographs.
- **Monthly cost estimate (my calculation using the prices above):**
  - Assume 30 new mini-games/day × (3K input tokens, about 2.5K of them cached system prompt, + 2K output). On Sonnet 5.5 that's about $0.78/day list, about $23/mo. With batch (−50%) and caching it's about **$10–12/mo**. On Haiku 5.5 it's about **$1–2/mo**.
  - TTS: 30 games × ~500 new chars = ~450K chars/mo, which fits inside Azure's 500K free neural tier (or about $7/mo beyond it). ElevenLabs would need Pro ($99) at that volume, or Creator ($22) if caching brings unique characters under 100K.
  - Cloudflare Workers + R2: $5/mo.
  - **Total: about $6–40/mo** depending on the model and TTS vendor, plus the Apple Developer Program fee.

### Gaps
- Official ElevenLabs API per-character rates and v3 Hebrew support level, Azure he-IL voice names (e.g., HilaNeural/AvriNeural, from training memory and not verified), Google Cloud TTS he-IL voice pricing, and OpenAI TTS Hebrew quality weren't verified from primary pages.
- Gemini/OpenAI LLM pricing for comparison wasn't retrieved.
- I found no benchmark of LLM Hebrew niqqud accuracy.

---

## 4. Distribution for family-only use (TestFlight, ad hoc, custom apps, Kids Category)

### Takeaway
**TestFlight internal testing under a paid Apple Developer account** is the practical channel for 5 family members. Builds expire, so plan a re-upload at least every 90 days (automate it with Xcode Cloud or fastlane). If the app ever goes public in the Kids Category, **third-party analytics and ads are effectively banned**, and that obligation sticks even if you later leave the category, so design privacy-clean from day one.

### Cited Findings
- Apple's guidelines say Kids Category apps should not include third-party analytics or third-party advertising. Once customers expect the app to follow Kids Category requirements, it must keep meeting them in later updates even if the category is deselected. — [App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)
- Limited exceptions: third-party analytics is allowed if no child's PII or device info is sent to a third party. An older version allowed contextual ads from providers with kids policies and human-reviewed creatives. — [App Store Review Guidelines History](https://www.appstorereviewguidelineshistory.com/articles/2019-09-14-kids-apps-ads-analytics-and-sign-in-with-apple/); [Apple news 2019](https://developer.apple.com/news/?id=06032019j)
- Enforcement example: a rejection for "third-party analytics or third-party advertising with the ability to collect, transmit or share identifiable information, including, for example, IDFA". — [Apple Developer Forums 130863](https://developer.apple.com/forums/thread/130863); [forum 117320](https://developer.apple.com/forums/thread/117320)

### Inferences
- **Options ranked for this family:**
  1. **TestFlight internal testers** (up to 100 members of the App Store Connect team, no beta review). Parents are added as team users. The kids' devices install via the parents' Apple IDs or the kids' own (internal testers need App Store Connect roles, so for child Apple IDs use **external testing** with a public or invite link, which needs a light Beta App Review). Builds expire after 90 days, so use an Xcode Cloud workflow that rebuilds every few weeks.
  2. **Ad hoc / development provisioning** (register up to 100 devices per device type per year): no expiry worries for a year, sideloaded through Xcode or Apple Configurator. Fine for 5–6 devices.
  3. **Free personal team (no paid account):** apps expire after 7 days and there's no CloudKit, so it's not viable.
  4. **Custom Apps (Apple Business Manager)** target organizations and are overkill.
  5. **Unlisted App Store distribution** (by request) is an option if friends want it later. A public Kids Category listing means a parental gate before any outward link or purchase, no third-party analytics or ads, and COPPA/GDPR-K-style privacy duties.
- CloudKit and App Attest need a paid account (the $99/yr figure is from training knowledge and wasn't verified this session).
- Design for Kids-Category compliance from the start (no third-party SDKs, a parental gate on settings and AI features, no external links in kid mode), so the app can go public later without a rewrite.

### Gaps
- I didn't verify the current TestFlight build validity (90 days), internal/external tester limits, ad-hoc device limits, or the developer fee against Apple's 2026 pages (apple.com was unreachable from this session).

---

## 5. Developer workflow with AI coding agents (Claude Code, Xcode 26.x agentic coding, previews, snapshot tests, Figma MCP, Rive)

### Takeaway
As of Xcode 26.3 (Feb 2026), **Claude Agent is built into Xcode** (Claude Agent SDK, the same harness as Claude Code) and can capture SwiftUI Previews to check its own UI work. Xcode also exposes an MCP server, so terminal Claude Code can drive builds and previews. A designer-parent can iterate on "describe → agent builds → preview screenshot → tweak" loops without being a Swift expert.

### Cited Findings
- Xcode 26.3, released late Feb 2026, added agentic coding with Anthropic's Claude Agent and OpenAI's Codex built in. Agents can search docs, explore files, update project settings, and **check their own work visually by capturing Xcode Previews**, iterating through builds and fixes. — [9to5Mac](https://9to5mac.com/2026/02/26/apple-releases-xcode-26-3-with-support-for-agentic-coding/); [MacTech](https://www.mactech.com/2026/02/27/apple-releases-xcode-26-3-with-support-for-agentic-coding); [SD Times](https://sdtimes.com/ai/xcode-26-3-adds-agentic-coding-support/)
- Xcode 26.3 integrates the Claude Agent SDK, giving subagents and background tasks. It exposes its capabilities over MCP so other agents can be used. A Hacker News user reported MCP responses not matching their schema early on. — [InfoQ](https://infoq.com/news/2026/02/xcode-26-3-agentic-coding); [AlternativeTo](https://alternativeto.net/news/2026/2/xcode-26-3-adds-agentic-coding-with-claude-agent-codex-and-other-ai-tools-via-mcp)
- Setup: Xcode Settings → Intelligence → Anthropic. One blog says the Claude Agent needs a paid Claude subscription (single source). AI coding requires macOS 26 (Tahoe). — [Medium](https://medium.com/@chinthaka01/part-1-agentic-coding-in-xcode-26-3-with-claude-agent-for-swift-swiftui-2dda7ec7272e); [heyuan110](https://www.heyuan110.com/posts/ai/2026-02-20-xcode-agentic-coding/)

### Inferences
- **Recommended loop:**
  1. The designer sketches in Figma with a shared design-token file (colors, type scale, spacing, radii).
  2. Claude Code (terminal) or Xcode's Claude Agent reads the tokens, through the Figma Dev Mode MCP server if available or exported JSON, and generates a SwiftUI `DesignSystem` package.
  3. Every screen gets `#Preview` variants (RTL Hebrew, large Dynamic Type, iPad/iPhone, each child profile) so the agent and the designer review visually.
  4. Snapshot tests (pointfreeco `swift-snapshot-testing`) lock in approved looks so agent refactors don't silently break visuals.
  5. A `CLAUDE.md` in the repo records the conventions: Swift 6 strict concurrency, SwiftUI + SpriteKit only, no third-party analytics, RTL rules, content-schema location.
- **Rive for designers:** the designer authors mascot state machines and view models in the Rive editor. The agent only wires Swift values to Rive view-model properties. Keep the property names in a shared contract file.
- **Fast iteration on content:** make game templates data-driven (JSON), so new games are new JSON, not new code, and a debug "content playground" screen loads packs from a local folder or URL.
- **Builds:** Xcode Cloud (or fastlane on the Mac) handles TestFlight uploads. Claude Code can run `xcodebuild` and tests from the terminal on the Mac.

### Gaps
- Figma Dev Mode MCP server's 2026 status/pricing and its SwiftUI output quality weren't researched this session.
- Xcode 27 (WWDC26) agent features beyond 26.3 weren't found in search results.

---

## 6. Asset pipeline (illustration consistency, vectorization, SFX, music licensing)

### Takeaway
Not deeply researched this session (tool budget and network blocks). Only engineering recommendations are offered here. Verify the vendor terms before relying on them.

### Cited Findings
- ElevenLabs offers Hebrew TTS voices and markets a Hebrew page. Commercial use starts at the Creator tier ($22/mo). — [ElevenLabs Hebrew](https://elevenlabs.io/text-to-speech/hebrew); [forasoft](https://www.forasoft.com/blog/article/synthetic-voice-library-apps)

### Inferences
- **Illustration consistency:** define a style bible (palette, line weight, character turnarounds). Generate with an image model that accepts style or character reference images (Gemini image models, GPT image, and Midjourney's style/character references all support reference-conditioned generation, per general knowledge). The designer then cleans up the output. For an app built from a small fixed cast (a mascot plus about 50 objects), it's often faster for the designer to **hand-finish vector masters** and use AI only for exploration and backgrounds.
- **Vectorization:** for SpriteKit, raster @2x/@3x PNG/WebP in texture atlases is simpler than vectors. Use vectors (SVG → PDF assets, or Rive) for UI and mascots.
- **SFX:** licensed libraries or AI SFX generation (ElevenLabs Sound Effects). Keep a `LICENSES.md` per asset.
- **Music:** for family-only use the risk is low, but if the app goes public, read the AI music terms closely (free-tier outputs on Suno/Udio have historically been non-commercial).

### Gaps
- Current Suno/Udio/ElevenLabs Music terms, ElevenLabs SFX pricing, Midjourney/Gemini/GPT image licensing and style-reference capabilities: **not verified this session**.

---

## 7. Content model, remote config, privacy-preserving analytics

### Takeaway
Use **versioned JSON content packs** (manifest plus item bank plus asset references) on R2/CDN, validated by the same JSON Schema that Claude's structured outputs use. Remote config is a small signed JSON file. Analytics are first-party events stored locally and synced to the parent's CloudKit or the family Worker. No third-party SDKs, which also keeps the app compatible with the Kids Category.

### Cited Findings
- Kids Category forbids third-party analytics that transmit identifiable or device info. — [App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)
- Claude structured outputs can constrain a response to a JSON Schema (`output_config.format`, or `strict: true` tools). — [Anthropic docs](https://platform.claude.com/docs/en/about-claude/pricing)

### Inferences
- **Suggested schema (sketch):**
  - `manifest.json`: `{packId, version (semver), minAppVersion, locale:"he-IL", ageBand, skills[], items[], assets[{id, url, sha256}], publishedAt}`
  - `item`: `{id, type: "letter_trace"|"match_picture"|"syllable_build"|"count"|..., skill: "he.letters.bet", difficulty 1–5, prompt: {text, textNiqqud, audioId}, options[...], answer, distractorRationale, assets[], tags[], reviewStatus: "draft"|"approved"}`
  - The app has a renderer per `type`, so the LLM only fills slots and never writes code.
- **Versioning:** use immutable pack versions with content-hashed assets. The app keeps the last good pack and can roll back. `minAppVersion` stops new item types from reaching old builds.
- **Remote config / live events:** a `config.json` on R2 with feature flags, event windows (e.g., a Hanukkah or Purim themed pack), and per-child overrides, cached with ETags.
- **Adaptive learning:** keep per-child per-skill mastery locally (Elo-like or a Leitner box), synced through CloudKit. That data feeds the next content-generation prompt (weak skills → more items).
- **Analytics:** `Event {ts, profileId (local UUID), packId, itemId, outcome, latencyMs}` goes to SwiftData, then syncs to CloudKit, with a parent dashboard in-app. Optionally the Worker aggregates anonymised counts. Never send child names, voice or drawings off-device unless a parent opts in.

### Gaps
- No external sources on item-bank standards (e.g., QTI) were retrieved. The schema above is a design proposal, not a sourced standard.

---

## 8. Performance and accessibility (120 Hz ProMotion, battery, older-iPad memory, VoiceOver, Dynamic Type)

### Takeaway
Not deeply researched with sources this session. Apple confirms Hebrew support for VoiceOver and Live Speech in iOS 27, so native accessibility in Hebrew is viable.

### Cited Findings
- Hebrew (Israel) is listed for VoiceOver, Voice Control, Live Speech and Read & Speak in iOS 27. — [Apple iOS feature availability (via search summary)](https://www.apple.com/ios/feature-availability/); [tbreak](https://tbreak.com/apple-intelligence-arabic-uae-watch-hypertension/)

### Inferences
- **120 Hz:** SwiftUI animations and SpriteKit (`preferredFramesPerSecond = 120`) follow ProMotion. Cap game scenes at 60 fps on non-ProMotion devices and pause `SKView`/`TimelineView` when they're off-screen to save battery. Avoid per-frame SwiftUI view rebuilds, and prefer `Canvas`/SpriteKit for many moving objects.
- **Memory on older iPads** (e.g., 3 GB RAM models): use texture atlases, compressed audio (AAC), preload only the current pack, and unload Rive files between scenes.
- **Accessibility for pre-readers:** a voice prompt on every instruction (TTS cache), big touch targets (≥60 pt for small kids), no time pressure by default, color-blind-safe feedback (shape plus color), reduced-motion support (`accessibilityReduceMotion`), and VoiceOver labels in Hebrew. Dynamic Type matters mostly in parent screens, and kid screens can use fixed large type.
- **Target OS:** iOS/iPadOS 26+ (Liquid Glass design, SpeechAnalyzer) is reasonable for family devices. Check that every kid's iPad can run it.

### Gaps
- No sourced device-support list for iOS 26/27 or ProMotion API changes was retrieved.

---

## 9. Synthesis: recommended architecture, build/distribution workflow, costs, and risks

### Takeaway
**Native Swift app (SwiftUI + SpriteKit + Rive) with offline JSON content packs, SwiftData/CloudKit for family progress, and a $5 Cloudflare Worker proxy calling Claude (batched, structured, cached) plus Azure/ElevenLabs Hebrew TTS cached on R2. Ship through TestFlight/ad hoc. Expect about $10–40/mo plus the developer fee.** The main risks are Hebrew-specific: no Apple on-device LLM or confirmed OCR in Hebrew, niqqud accuracy from LLMs and TTS, and children's speech recognition.

### Cited Findings
- See sections 1–8. The key load-bearing facts: Apple Intelligence has no Hebrew in iOS 27 ([Apple Support](https://support.apple.com/en-il/121115); [tbreak](https://tbreak.com/apple-intelligence-arabic-uae-watch-hypertension/)). SpeechTranscriber reportedly includes he_IL ([LoroNote](https://loronote.com/en/blog/apple-speechanalyzer-vs-whisper); [Gubarenko](https://antongubarenko.substack.com/p/ios-26-speechanalyzer-guide)). Claude Haiku 5.5 is $0.10/$0.50 and Sonnet 5.5 is $2/$10 per MTok, with batch at about 50% off ([Anthropic](https://platform.claude.com/docs/en/about-claude/pricing)). The Workers Paid plan is $5/mo ([Cloudflare](https://developers.cloudflare.com/workers/platform/pricing/index.md)). Xcode 26.3 has a built-in Claude Agent ([InfoQ](https://infoq.com/news/2026/02/xcode-26-3-agentic-coding)). Kids Category bans third-party analytics and ads ([Apple](https://developer.apple.com/app-store/review/guidelines/)).

### Inferences
**Recommended stack**
| Layer | Choice | Why |
|---|---|---|
| App UI | SwiftUI (iOS/iPadOS 26+), RTL Hebrew | one language, previews, agent-friendly |
| Game scenes | SpriteKit via `SpriteView`; `Canvas`/`TimelineView` for light effects | native 2D, small, offline |
| Character animation | Rive (new Apple runtime, data binding), Lottie for one-shots | designer-owned interactivity |
| Input | PencilKit plus custom stroke scoring; SpeechTranscriber he_IL with DictationTranscriber fallback | Hebrew OCR unconfirmed |
| Feedback | Core Haptics / `.sensoryFeedback`; AVAudioEngine 3-bus mix | polish |
| Local data | SwiftData, offline-first | |
| Family sync | CloudKit private DB (same family account); upgrade to CKShare (Core Data) or Supabase only if needed | zero server for progress |
| Content | Versioned JSON packs + hashed assets on Cloudflare R2 | zero egress |
| AI proxy | Cloudflare Worker + App Attest + KV rate limits; keys as secrets | never ship keys |
| LLM | Claude Sonnet 5.5 (content quality), Haiku 5.5 (bulk/variants), Batch API + prompt caching + structured outputs; parent approval gate | cheap, schema-safe |
| TTS | Azure Neural he-IL (500K free chars/mo) or ElevenLabs v3 with Hebrew language code; cache by hash on R2 | Hebrew quality varies; A/B with the kids |
| Analytics | First-party events → SwiftData → CloudKit; in-app parent dashboard | Kids-Category-safe |
| Distribution | Paid dev account → Xcode Cloud → TestFlight (rebuild under 90 days) or ad hoc | family-only |
| Dev workflow | Claude Code + Xcode 26.3+ Claude Agent, `#Preview` matrix, snapshot tests, CLAUDE.md conventions, Figma tokens → SwiftUI design system | designer-led iteration |

**Phased plan**
1. **Weeks 1–2:** design system and shell (profile picker, home map), 2 game templates (letter tracing, picture–word match) with hand-made content, TestFlight to the family.
2. **Weeks 3–4:** Worker proxy, Claude batch generation into the JSON schema, parent review screen, TTS cache.
3. **Weeks 5–6:** CloudKit sync, mastery model, Rive mascot, speech game prototype (test on the kids).
4. **Later:** live events through remote config, more templates, and optional App Store Kids Category hardening.

**Risks and mitigations**
- **Hebrew niqqud errors** in LLM text and TTS pronunciation: a second-pass validator, a parent approval gate, and phonetic overrides in the TTS cache.
- **No Hebrew on-device LLM** (Foundation Models): all generation goes server-side and is pre-generated, so it works offline after sync.
- **Kids' speech recognition accuracy** in Hebrew is unknown: make speech games optional, keep soft feedback, and allow a parent override.
- **TestFlight expiry** and certificate renewal friction: automate with Xcode Cloud and set a calendar reminder.
- **Vendor and API drift** (Rive's runtime API change, Claude model retirements, Xcode MCP bugs): pin versions and keep model IDs in remote config.
- **Privacy:** child voice and drawings stay on device. The Worker logs no PII.
- **Cost runaway** from a bug in a generation loop: per-device rate limits in the Worker plus an Anthropic workspace spend limit.
- **SwiftData + CloudKit limits** (no unique constraints, no native sharing): design the models for this from day one.

### Gaps
- All Apple language-support claims need on-device verification (`supportedLocales`, `supportedRecognitionLanguages()`, `SystemLanguageModel.supportedLanguages`) on iOS 27, because primary Apple pages were unreachable this session (apple.com, 9to5mac.com and the developer.apple.com rendered pages need to be allowed in the session's network settings for a re-check).
- The asset-pipeline vendor terms (Suno/Udio/ElevenLabs Music/SFX, image models) and Figma MCP were not verified.
