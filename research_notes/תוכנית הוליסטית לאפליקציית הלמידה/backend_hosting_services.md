# Backend, Hosting and Services for a Family-Only Kids' Learning Game (Nov 2026)

> Method note: In this cloud session, WebFetch could not resolve the official vendor domains (developers.cloudflare.com, supabase.com and firebase.google.com all returned ENOTFOUND, so those domains need to be allowed in the session's network settings before full pages can be fetched). Findings below come from WebSearch result extracts. Where an extract cited an official page (Cloudflare docs, Firebase FAQ, Expo docs, Anthropic docs, Meta docs, Telegram docs), it is marked as official. Third-party aggregator figures are marked as such and should be re-verified before money is committed. 18 search calls were made.

## 1. Which backend fits best? (Supabase vs Firebase vs CloudKit vs Cloudflare vs Convex vs PocketBase/VPS vs InstantDB/Turso)

### Takeaway
**Recommendation: Cloudflare Workers Paid ($5/mo) with Workers + D1 (SQLite) + R2 + Cron Triggers, all deployed by the agent with `wrangler` from Linux.** It is the only option that is cheap, never pauses, has no egress fees, keeps AI keys server-side, includes 30-day point-in-time recovery for the DB, and can be managed entirely from the CLI and a config file. Supabase Free pauses after 7 days of low activity (a real risk for a family app during holidays), and Supabase Pro costs $25/mo. Firebase Cloud Storage now requires the Blaze plan. CloudKit is free and private, but it is Apple-only and awkward for a server-side AI proxy and for cron jobs, and an agent cannot easily operate it headless from Linux.

### Cited Findings
**Cloudflare (recommended)**
- Workers Paid: $5/month minimum. Includes 10M requests and 30M CPU-ms per month. Overage is $0.30 per additional 1M requests and $0.02 per additional 1M CPU-ms (official) — [Cloudflare Workers pricing](https://developers.cloudflare.com/workers/platform/pricing/)
- D1 on Workers Paid: 25 billion rows read and 50 million rows written per month included. Overage is $0.001 per 1M rows read and $1.00 per 1M rows written (official) — [Cloudflare D1 pricing](https://developers.cloudflare.com/d1/platform/pricing/)
- D1 limits: max DB size is 10 GB on Paid (500 MB on Free). Max storage per account is 1 TB on Paid (5 GB on Free) (official) — [D1 limits](https://developers.cloudflare.com/d1/platform/limits/)
- D1 Time Travel (point-in-time restore): 30 days on Paid, 7 days on Free. No extra cost, does not count against storage, 10 restores per 10 minutes. Restores are destructive and overwrite the DB in place (official) — [D1 limits](https://developers.cloudflare.com/d1/platform/limits/); [D1 blog "turning it up to 11"](https://blog.cloudflare.com/d1-turning-it-up-to-11)
- D1 Free daily caps: 5M rows read/day, 100K rows written/day, 5 GB total (third-party tracker) — [cloud-pricing-tracker](https://cloud-pricing-tracker.pages.dev/services)
- Workers Free: 100,000 requests/day and 10 ms CPU per invocation. Paid allows up to 5 minutes of CPU (third-party) — [apicostcalc](https://apicostcalc.com/cloudflare.html); [findutils](https://findutils.pages.dev/developers/cloudflare-cost-calculator/)
- R2: Standard storage $0.015/GB-month. Free tier is 10 GB-month, 1M Class A ops and 10M Class B ops per month. Egress to the internet is free. Class A costs $4.50/M and Class B $0.36/M on Standard (official) — [R2 pricing](https://developers.cloudflare.com/r2/pricing/); [R2 pricing md](https://developers.cloudflare.com/r2/pricing/index.md)
- Conflict: one tracker lists R2 Class A at $9.00/M and Class B at $0.90/M. Those are actually the Infrequent Access rates on the official page, so the official Standard rates of $4.50 and $0.36 apply — [cloud-pricing-tracker](https://cloud-pricing-tracker.pages.dev/services) vs [R2 pricing](https://developers.cloudflare.com/r2/pricing/)
- SQLite-backed Durable Objects use the same row pricing as D1 (official) — [Durable Objects pricing](https://developers.cloudflare.com/durable-objects/platform/pricing/)
- Workers no longer have the 1,000-subrequest limit (changelog, Feb 2026) — [Cloudflare changelog](https://developers.cloudflare.com/changelog/post/2026-02-11-subrequests-limit/)

**Supabase**
- Free projects are paused after "low activity over a 7-day period". Data is not deleted and can be restored from the dashboard. Paid projects are not paused (official docs) — [Supabase Project Pausing](https://supabase.com/docs/guides/platform/free-project-pausing)
- Free plan (third-party 2026 guides): 2 active projects, 500 MB DB per project, 50,000 MAU, 5 GB bandwidth, 1 GB file storage. Pro: 8 GB DB, 100K MAU, 100 GB storage, daily backups kept 7 days, no pausing — [jetadmin](https://www.jetadmin.io/blog/supabase-pricing-2026-guide-to-plans-limits-and-real-world-costs/); [automationatlas](https://automationatlas.io/answers/supabase-free-tier-limits-2026/)
- Workaround articles exist for keeping a Free project awake with a scheduled GitHub Actions ping. This shows the pause is a known pain point — [levelup.gitconnected](https://levelup.gitconnected.com/supabase-free-tier-will-pause-your-app-heres-the-github-actions-fix-8c1fd35b49ca)

**Firebase**
- Cloud Storage for Firebase, including default buckets, now requires the Blaze pay-as-you-go plan. On Spark, bucket API calls return 402/403. Spark projects lost read/write access to default buckets starting Feb 3, 2026. Default buckets can no longer be provisioned with Terraform (official FAQ) — [Firebase storage changes FAQ](https://firebase.google.com/docs/storage/faqs-storage-changes-announced-sept-2024?authuser=4); [mirror](https://firebase.google.cn/docs/storage/faqs-storage-changes-announced-sept-2024)
- "Usage at no cost is still possible on Blaze" (official FAQ, same source)

**Apple CloudKit**
- Data in the private database counts against the *user's* iCloud quota, not the developer's — [Apple docs: privateCloudDatabase](https://developer.apple.com/documentation/cloudkit/ckcontainer/privateclouddatabase.md); [Apple forum 2025](https://developer.apple.com/forums/thread/773582)
- When a quota is hit, apps must handle `CKError.quotaExceeded` — [Apple forum](https://developer.apple.com/forums/thread/665612)
- I found no current 2026 public-database quota figures. Figures from about 2016 (100 MB DB, 10 GB assets, 40 req/s) are outdated — [Apple forum (old)](https://developer.apple.com/forums/thread/35633)

**Convex**
- Free tier is "meant for one or two developers building a personal project". Exceeding limits for an extended period may cause HTTP errors (official FAQ) — [Convex pricing FAQ](https://www.convex.dev/pricing/faq)
- Aggregators list 1M function calls/mo, 0.5 GB DB and 1 GB file storage free. Pro is $25/dev/mo (unverified, sources inconsistent) — [costbench](https://costbench.com/software/database-as-service/convex/free-plan/); [agentdeals](https://agentdeals.dev/vendor/convex)

**PocketBase on a VPS (Hetzner)**
- Hetzner raised prices twice in 2026 (Apr 1 and Jun 15). After both rounds, a CAX11 (2 Arm vCPU/4 GB) costs about €5.99/mo in DE/FI, excluding IPv4 and VAT. The CX22 was replaced by the CX23 at about €5.49 — [Northflank](https://northflank.com/blog/hetzner-cloud-server-price-increases); [webhosting.today](https://webhosting.today/2026/06/18/hetzners-price-increases-reached-209-the-30-headline-applied-to-a-different-tier/); [Tom's Hardware](https://www.tomshardware.com/tech-industry/hetzner-to-raise-prices-by-up-to-37-percent-from-april-1)

**PowerSync (sync layer)**
- Official: Free plan is $0 with soft limits and no card required. A mirror of the pricing page says Free projects are deactivated after 1 week of inactivity. Third-party sources list 2 GB synced and 50 peak connections, and Pro from $49/mo (unverified) — [PowerSync pricing](https://powersync.com/pricing); [toolradar](https://toolradar.com/tools/powersync/pricing)

### Inferences
- **Fit ranking for this family (5 users, about 6 devices, low writes):**
  1. **Cloudflare Workers Paid + D1 + R2**, about $5/mo. It never pauses, has PITR, needs no egress budget, and has a first-class CLI (`wrangler deploy`, `wrangler d1 migrations apply`, `wrangler r2 object`). Config lives in `wrangler.toml`/`wrangler.jsonc` in git, and `wrangler dev` runs locally (Miniflare) on Linux. That makes it the best fit for an AI agent.
  2. **Supabase Pro**, $25/mo. Real Postgres, auth, storage, edge functions and RLS make it very capable. The Free tier is unsuitable because of the 7-day pause.
  3. **Firebase Blaze**, about $0 at this scale. A card is mandatory, and the Firestore/Functions setup carries Google Cloud IAM overhead. It is good for FCM, but that is not needed on iOS-only.
  4. **CloudKit**, free and the most private (data stays in parents' iCloud). It is Apple-ID-only, needs Swift/native modules, and cannot hold AI keys or run cron jobs, so a Worker would be needed anyway. That makes it two systems.
  5. **PocketBase on Hetzner**, about €6/mo. It is simple, but the agent becomes the sysadmin (OS patches, TLS, backups, uptime).
  6. **Convex/InstantDB/Turso**, all viable, but they add a vendor without removing the need for Workers for cron/AI proxy. InstantDB/Turso 2026 limits were not verified.
- Free Workers (no $5) would technically suffice for traffic, but the 10 ms CPU cap and 7-day Time Travel make the $5 plan worth it. The $5 also covers Durable Objects if real-time features are wanted later.
- D1 is SQLite, which pairs naturally with on-device SQLite (expo-sqlite) for a simple sync design.

### Gaps
- Official Cloudflare free-plan limits for Cron Triggers count, Queues and KV were not retrieved (fetch blocked). Cron Triggers exist on both plans per general knowledge, but the counts are unverified.
- Official Supabase/Firebase pricing pages could not be fetched. Supabase figures are from aggregators.
- InstantDB and Turso 2026 free-tier limits were not found.
- R2 encryption at rest is not confirmed from 2026 docs. Cloudflare has historically stated that R2 encrypts all objects at rest, but that is unverified here.

## 2. Auth for a family: Sign in with Apple, picture-PIN child profiles, device trust, parental gate

### Takeaway
Parents sign in with Sign in with Apple. The Worker verifies Apple's identity token and issues its own session JWT. Children never get accounts: they are rows in a `child_profiles` table belonging to a `family`, and the child picks an avatar plus a picture-PIN on a device already trusted by a parent. Device trust comes from a parent "pairing" the device (QR or 6-digit code), backed by App Attest.

### Cited Findings
- Expo's `@expo/app-integrity` uses Apple App Attest on iOS. The flow: the app generates a hardware-backed key pair per user per device, attests it once, then signs ("asserts") sensitive requests. The server verifies using Apple's validation steps and stores the key ID (official Expo docs/blog) — [Expo app-integrity docs](https://docs.expo.dev/versions/latest/sdk/app-integrity/); [Expo blog](https://expo.dev/blog/expo-app-integrity)
- The app sends `attestationObject` and `keyId` to the server once, then assertion plus client data with a server-issued challenge on later requests. If an assertion fails, the developer decides how to handle it — [Expo app-integrity docs](https://docs.expo.dev/versions/latest/sdk/app-integrity/)

### Inferences
- **Design:** `families` (1 row), `parents` (2 rows, keyed by Apple `sub`), `devices` (paired, with App Attest key ID, last seen, revocable), `child_profiles` (3 rows: name, avatar, picture-PIN hash, age band).
- The picture-PIN is a convenience lock that stops siblings swapping profiles, not a security boundary. Store it as a salted hash and verify it on the device or in the Worker.
- **Parental gate:** For Apple's Kids category guidelines, use a gate before parent areas, purchases and external links. A Hebrew math question or press-and-hold works, and Face ID/passcode via `expo-local-authentication` is stronger.
- **No email/password and no child accounts.** That minimises personal data and removes password resets.
- App Attest verification in a Worker needs CBOR parsing, an X.509 chain check against Apple's App Attest root, and ECDSA via WebCrypto. This is feasible but unverified for Workers in sources. A simpler first version is device pairing tokens plus Cloudflare rate limiting, with App Attest added later.

### Gaps
- Apple's "Validating apps that connect to your server" page and the Sign in with Apple REST token-verification docs were not retrieved.
- No source confirmed a ready-made Workers-compatible App Attest verification library.

## 3. Offline-first sync options

### Takeaway
For 5 users and a handful of devices, **skip sync frameworks**. Use expo-sqlite (or MMKV/Legend-State for small state) on the device, plus a simple **append-only event log** synced to D1 through the Worker (`POST /sync` with events since cursor X, last-write-wins per field for profile settings, and progress events that are naturally mergeable as sums or max). PowerSync, ElectricSQL and Zero are overkill. PowerSync Free also deactivates after 1 week of inactivity, per a mirror of its pricing page.

### Cited Findings
- PowerSync Free: $0, soft limits. Projects are reportedly deactivated after 1 week of inactivity (mirror of official page). Paid tiers start at about $49/mo (unverified) — [PowerSync pricing](https://powersync.com/pricing); [toolradar](https://toolradar.com/tools/powersync/pricing)
- Convex offers real-time sync, but its free tier is aimed at personal projects and paid is $25/dev — [Convex FAQ](https://www.convex.dev/pricing/faq)
- D1 is SQLite with row-based pricing that is effectively free at family scale (25B reads/50M writes included on Paid) — [D1 pricing](https://developers.cloudflare.com/d1/platform/pricing/)

### Inferences
- **Event-log pattern:** each device writes `events(id uuid, device_id, child_id, type, payload, created_at)` locally, then uploads unsynced events on foreground or reconnect and pulls `events where seq > lastSeq`. Derived tables (stars, mastered words) are recomputed on the device. It is idempotent via UUIDs, costs nothing, and is easy for an agent to test.
- Content packs and TTS audio are read-only on the device. Download a manifest (JSON with hashes) from R2 via the Worker, then cache files in `expo-file-system`.
- If the app is native Swift instead of Expo, SwiftData plus CloudKit sync is the zero-backend alternative, but then the AI proxy still needs a Worker.

### Gaps
- No 2026 sources retrieved on ElectricSQL, Replicache/Zero, Legend-State, WatermelonDB or TinyBase pricing or maturity.

## 4. Secrets and AI proxy: Worker patterns, rate limits, App Attest, prompt logging, cost caps

### Takeaway
All AI calls go through `/ai/*` routes on the Worker. Keys are stored as Worker secrets (`wrangler secret put ANTHROPIC_API_KEY`), never in the app. The Worker checks the device token or assertion, applies per-child and per-day quotas tracked in D1 or a Durable Object, and logs prompts and responses to D1 for parent review (photos are referenced by R2 key only). It also enforces a hard daily dollar cap in code. Provider-side caps sit underneath as a second layer: Anthropic offers organization/workspace monthly spend limits.

### Cited Findings
- Anthropic: spend limits exist at the organization and workspace level, monthly. There is an API (`POST /v1/organizations/spend_limits`) with daily/weekly/monthly periods, in early-access preview (official docs) — [Anthropic spend limits API](https://platform.claude.com/docs/en/api/beta/organization/spend_limits.md)
- Workspaces give per-workspace monthly spend limits (official announcement) — [Anthropic workspaces](https://anthropic.com/news/workspaces); [Claude workspaces article](https://claude.com/resources/articles/workspaces)
- Per-API-key monthly caps are not officially documented. The practical approach is one workspace per use, with a workspace-scoped key — [requesty blog](https://www.requesty.ai/blog/llm-spend-limits-per-api-key-and-customer)
- Workers Paid CPU allowance is up to 5 min per invocation, which covers streaming LLM responses and long TTS calls (third-party) — [findutils](https://findutils.pages.dev/developers/cloudflare-cost-calculator/)
- App Attest flow (see section 2) — [Expo app-integrity](https://docs.expo.dev/versions/latest/sdk/app-integrity/)

### Inferences
- **Layers of cost protection:** (1) a Worker counter in D1, e.g. `ai_spend_cents` per day with a refusal above $1/day; (2) per-child request quotas; (3) an Anthropic workspace "family-app" with a monthly limit; (4) Google AI Studio, OpenAI and ElevenLabs budget alerts or limits in each console.
- **Logging:** a `ai_log` table (child_id, model, prompt, response, tokens, cost estimate, flagged) feeds the parent dashboard "what did the AI say to my kid" view. Set retention to 90 days and purge with a cron job.
- **Caching:** hash (text, voice, model) and store ElevenLabs/Gemini TTS output in R2, so each phrase is paid for once. Egress is free on R2.
- Cloudflare AI Gateway (caching, rate limiting and logging in front of LLM providers) is a possible add-on. It was not verified in this research.

### Gaps
- Pricing and spend-cap controls for OpenAI, Google AI Studio (Gemini) and ElevenLabs were not researched here. Another researcher may cover AI APIs.
- Cloudflare's Rate Limiting binding and AI Gateway pricing were not verified.

## 5. Push notifications and parent messaging (APNs vs Expo Push vs FCM; email; Telegram vs WhatsApp)

### Takeaway
Use **Expo Push** (free) if the app is built with Expo. It wraps APNs, and the Worker POSTs to Expo's API from a cron job. For a native Swift app, send APNs directly from the Worker with a .p8 key. FCM adds nothing on iOS-only. Send the **weekly report by email via Resend** (free tier of 3,000/mo and 100/day is far more than needed). If chat delivery is wanted, a **Telegram bot** is free and trivial, but each parent must message the bot first. **WhatsApp Business Cloud API** is paid per template message (about $0.005–0.006 per utility message in Israel) and requires Meta Business verification. That is too much friction for 2 parents.

### Cited Findings
- "There is no cost associated with sending notifications through Expo push notification service". It can be used, or FCM/APNs directly (official FAQ) — [Expo push FAQ](https://docs.expo.dev/push-notifications/faq/)
- Expo/SDK are free (MIT). EAS (builds, submit, updates) is where pricing applies. The Free plan includes a limited number of low-priority builds and free EAS Update (official) — [Expo pricing](https://expo.dev/pricing.md); [Expo billing plans](https://docs.expo.dev/billing/plans/)
- Resend free: 3,000 emails/mo, 100/day cap, 1 domain, 30-day log retention. Postmark free: 100/mo (third-party 2026) — [automationatlas Resend](https://automationatlas.io/answers/resend-free-tier-explained-2026/); [agentdeals Resend](https://agentdeals.dev/vendor/resend); [agentdeals Postmark](https://agentdeals.dev/vendor/postmark)
- Meta moved WhatsApp to per-message pricing on Jul 1, 2025. Utility templates are free inside an open customer-service window and charged outside it (official) — [Meta WhatsApp pricing](https://developers.facebook.com/docs/whatsapp/pricing)
- Israel rate cards: one lists marketing $0.0406 and utility $0.0061 (effective Jan 1, 2026), another lists 0.0353/0.0053. A blog claims Meta cut Israel utility pricing about 45% in Q2 2026 (unverified) — [chakrahq](https://chakrahq.com/article/whatsapp-api-pricing-guide/); [Times of Israel blog](https://blogs.timesofisrael.com/israels-quiet-q2-2026-bot-pricing-reset/)
- Some guides claim service messages will be charged from Oct 1, 2026. This conflicts with Meta docs and is unverified — [agentdeals WhatsApp](https://agentdeals.dev/whatsapp-business-api-pricing-2026)
- Telegram Bot API is free with no per-message fee. A bot cannot message a user first, so the user must start the chat. Baseline limit is 30 msg/s (official FAQ plus third-party) — [Telegram Bots FAQ](https://core.telegram.org/bots/faq); [sleekflow](https://sleekflow.io/blog/telegram-api)

### Inferences
- **Recommended:** a gentle push (max 1/day, parent devices only, quiet hours), a Sunday-morning Hebrew HTML email report via Resend, and optionally a Telegram bot.
- **WhatsApp route if strongly desired:** the cost would be cents per month (about 8 utility messages/mo × about $0.006), but setup is the real cost: a Meta Business account, a dedicated phone number, template approval in Hebrew, and token management. A middle ground is a weekly report email containing a `wa.me` share link.
- Resend needs a verified sending domain, which is a reason to buy a domain (~$10/yr). Cloudflare Registrar sells at cost (unverified here).
- Push from a family app needs no third-party SDK beyond expo-notifications, so there are no analytics or tracking concerns.

### Gaps
- APNs token-based auth from Workers (JWT ES256 with WebCrypto, HTTP/2 to api.push.apple.com) is not verified for Workers' fetch. Expo Push avoids the question.
- Official Resend and Postmark pricing pages were not fetched.

## 6. Photo storage privacy and legal (Israeli PPL Amendment 13, COPPA)

### Takeaway
Store homework photos in a **private R2 bucket**. Never make objects public. Serve them only via the Worker with parent/device auth or short-lived signed URLs. Add **client-side encryption** (AES-GCM key held in the iOS Keychain and shared between family devices via pairing) if "even the cloud provider can't see it" is wanted. Set **auto-delete** with R2 lifecycle rules or a nightly cron (e.g., 30 days after the AI has extracted the text), and strip EXIF/GPS before upload. Legally, a private, non-commercial, family-only app is very likely outside both COPPA (which applies to operators acting "for commercial purposes") and Israel's PPL (which has a personal/household-use exemption). Amendment 13 took effect Aug 14, 2025. This is an inference, not legal advice. Sending children's photos to third-party AI APIs is the real privacy exposure, so choose providers and settings that do not train on API data.

### Cited Findings
- Israel PPL Amendment 13 entered into force Aug 14, 2025 — [IAPP](https://iapp.org/news/a/israel-marks-a-new-era-in-privacy-law-amendment-13-ushers-in-sweeping-reform); [Goldfarb](https://www.goldfarb.com/amendment-13-to-the-israeli-privacy-protection-law/)
- The PPL carves out personal/household activities from its requirements (secondary/vendor guides) — [cookie-script](https://cookie-script.com/privacy-laws/israel-privacy-protection-law-ppl); [Clym](https://www.clym.io/regulations/protection-of-privacy-law-ppl-5741-1981-israel)
- COPPA's "operator" definition covers someone operating a website or online service that collects personal info "for commercial purposes". Services "directed to children" are judged by content, language and imagery (FTC guidance) — [FTC: COPPA not just kids sites](https://ftc.gov/business-guidance/resources/childrens-online-privacy-protection-rule-not-just-kids-sites); [LexisNexis guide](https://www.lexisnexis.com/community/insights/legal/practical-guidance-journal/b/pa/posts/a-guide-to-protecting-children-s-privacy-online)
- R2 egress is free, so serving photos to the parent dashboard costs nothing beyond operations — [R2 pricing](https://developers.cloudflare.com/r2/pricing/)

### Inferences
- **Practical privacy checklist:**
  - private bucket
  - Worker-mediated access
  - EXIF strip
  - optional on-device face blur (Apple Vision face detection before upload is feasible but beyond homework needs)
  - 30-day auto-delete
  - AI request logs retained for 90 days
  - no third-party analytics SDKs
  - data export button
  - "delete everything" script
- If the app is distributed beyond the family (App Store public listing in the Kids category), COPPA, Apple Kids-category rules and PPL obligations (database registration and security regulations) could start to apply. Keep it on TestFlight or ad-hoc/internal distribution.
- The PPL household exemption likely covers the parent's own processing. Third-party processors (Cloudflare, Anthropic, Google) are covered by their own terms. Prefer the EU/ISR-friendly options where available (R2 location hint "eeur"/"weur"; unverified).

### Gaps
- The exact statutory wording of the PPL household exemption and whether it covers a parent processing their children's data was not retrieved from the statute. A legal review is recommended if the app is ever shared.
- No source specific to a "family-only non-commercial app" under COPPA was found. The commercial-purpose reasoning is inference.
- R2 server-side encryption at rest and lifecycle-rule syntax were not verified in 2026 docs.

## 7. Observability without third-party analytics; backups, export, domain, costs; accounts checklist

### Takeaway
Log self-defined events to D1 (the same event log used for sync, plus an `app_logs` table), and view them in the parent dashboard. Sentry's free Developer plan (5,000 errors/mo, 1 user) is fine for crash reporting in a family app. Alternatively, rely on Xcode Organizer/TestFlight crash reports at zero setup. Backups combine D1 Time Travel (30 days), a nightly cron that exports D1 to R2 as SQL/JSON, and optionally a weekly mirror to a parent's iCloud/Google Drive. **Expected monthly cost: about $5 for infrastructure plus about $0–10 for AI APIs, so roughly $5–15/month.** One-time costs are the Apple Developer Program ($99/yr ≈ $8.25/mo) and an optional domain (about $10/yr).

### Cited Findings
- Sentry free Developer plan: 5,000 errors/month, 1 user. Span/replay figures vary by source. Team is about $26–29/mo (third-party) — [costbench Sentry](https://costbench.com/software/developer-tools/sentry/free-plan/); [toolradar Sentry](https://toolradar.com/tools/sentry/pricing)
- `@sentry/react-native` supports Expo SDK 50+ via `npx expo install @sentry/react-native` and the config plugin. `sentry-expo` is deprecated (official docs) — [Sentry Expo guide](https://docs.sentry.io/platforms/react-native/guides/expo/)
- D1 Time Travel: 30-day PITR on Paid at no extra cost (official) — [D1 limits](https://developers.cloudflare.com/d1/platform/limits/)
- EAS Free plan: limited low-priority builds and free EAS Update (official) — [Expo billing](https://docs.expo.dev/billing/plans/)

### Inferences

**Architecture (text diagram)**
```
 iPhone/iPad (Expo RN app, Hebrew RTL)                Parent web dashboard
 ├─ expo-sqlite: local event log + content cache       (Cloudflare Pages/Workers
 ├─ expo-file-system: TTS mp3 + content packs           static assets, Sign in
 ├─ Sign in with Apple (parents) / picture-PIN (kids)    with Apple JS)
 ├─ App Attest (expo app-integrity)                            │
 └─ expo-notifications (parent devices only)                   │
            │ HTTPS (JWT + device assertion)                    │
            ▼                                                  ▼
 ┌──────────────── Cloudflare (Workers Paid $5/mo) ─────────────────┐
 │ Worker "api" (Hono/TS):                                          │
 │   /auth (verify Apple id_token → session JWT)                     │
 │   /sync (push/pull events)   /photos (signed upload/download)    │
 │   /ai/* proxy → Anthropic / Gemini / OpenAI / ElevenLabs         │
 │        (secrets, quotas, $/day cap, prompt log)                   │
 │ D1 (SQLite): families, parents, devices, children, events,       │
 │              ai_log, app_logs, push_tokens  [Time Travel 30d]     │
 │ R2 (private): photos/ (30d auto-delete), tts-cache/, packs/,     │
 │              backups/ (nightly D1 export)                         │
 │ Cron Triggers: 02:00 nightly content gen + backup + purge;       │
 │                Sun 08:00 weekly report → Resend email + Expo Push │
 └──────────────────────────────────────────────────────────────────┘
            │                         │                    │
       Expo Push API → APNs     Resend (email)     (optional) Telegram bot
```

**Monthly cost estimate (family of 5)**

| Item | Cost |
|---|---|
| Cloudflare Workers Paid (incl. D1, DOs; R2 within 10 GB free) | $5.00 |
| R2 beyond 10 GB (unlikely; e.g. 20 GB) | ~$0.15 |
| Expo (EAS Free) + Expo Push | $0 |
| Resend Free | $0 |
| Sentry Developer | $0 |
| Apple Developer Program ($99/yr) | ~$8.25 |
| Domain (optional, ~$10/yr) | ~$0.85 |
| AI APIs (depends on usage; cached TTS, nightly batch gen) | ~$2–10 (not verified here) |
| **Total** | **~$16–24/mo incl. Apple fee; ~$5–15 excl. Apple fee** |

**Accounts the parent must create (truly necessary)**
1. **Apple Developer Program** ($99/yr). Required for TestFlight/device install, Sign in with Apple, push (APNs key) and App Attest. The parent must enroll personally (identity verification).
2. **GitHub**. Code repo plus Actions CI. It is free and lets the agent work in the repo.
3. **Cloudflare**. Add a card, subscribe to Workers Paid $5, and create a scoped API token (Workers, D1, R2, Pages edit) for the agent.
4. **Anthropic Console (API)**. Create a workspace "family-app" with a monthly spend limit and a key.
5. **Expo account**. Free. Needed for EAS Build/Submit (building iOS from Linux requires EAS cloud builds) and Expo Push. The parent creates an access token for the agent.

Optional, depending on features:
- **Google AI Studio** (Gemini key; free tier possible), only if Gemini is used for vision/TTS.
- **ElevenLabs**, only if premium Hebrew voices are needed.
- **OpenAI**, only if a specific model is needed. Recommend skipping to minimise accounts.
- **Resend**, for the weekly email (needs a domain). Alternatively, skip email and use push only.
- **Domain** (via Cloudflare Registrar), for email sending and a dashboard URL. Otherwise `*.workers.dev` is free.
- **Sentry**, optional, since TestFlight crash logs may suffice.
- **Telegram bot** via @BotFather, optional, free and takes 2 minutes.
- Not needed: Supabase, Firebase, Fly.io, Hetzner, PowerSync, WhatsApp Business.

**What the agent can do alone (Linux container)**
- Write all code (Expo app, Worker, dashboard).
- Run `wrangler dev` locally, write and run D1 migrations, deploy Workers and Pages, create R2 buckets, set secrets (`wrangler secret put`), and configure cron.
- Write tests, CI (GitHub Actions) and backup/restore scripts.
- Run EAS cloud builds (`eas build -p ios`) and submit to TestFlight (`eas submit`), once the Apple credentials/API key are provided.
- Generate content packs and TTS caches.

**What only the parent can do**
- Enroll in Apple Developer and accept agreements.
- Create the App Store Connect API key (or give the agent the credentials).
- Enter payment cards.
- Create provider API keys and paste them as secrets (ideally into Cloudflare via the dashboard, or hand them to the agent once).
- Add family members as TestFlight testers.
- Verify the domain DNS for Resend (the agent can do this if the domain is on Cloudflare and the token has DNS edit).

### Gaps
- AI API usage cost was not modelled here (depends on model choice and volumes). Likely covered by another researcher.
- TestFlight build expiry (90 days, historically) and ad-hoc/internal distribution limits for 2026 were not verified. These matter for a long-lived family app, since EAS or a GitHub Action would need to rebuild periodically.
- EAS Free plan exact build quotas for 2026 were not retrieved (page truncated).
- Cloudflare Registrar pricing and Workers Static Assets/Pages free limits were not verified.
