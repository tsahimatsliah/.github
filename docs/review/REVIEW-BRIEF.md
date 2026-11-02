# Fable Review Brief (shared by all reviewers)

Goal (from the parent): make the plan tighter, more precise, more complete — cover every use case, edge case, gap and need — so that when the parent says "build it", the developer agent can execute end-to-end with almost no questions.

Corpus:
- Binding decisions: `docs/spec/00-foundation-decisions.md` (owned ONLY by reviewer F1).
- Specs: `docs/spec/01..13-*.md` (each reviewer owns specific files — edit ONLY those).
- Research: `research_notes/אפליקציית למידה משחקית לילדים/*.md`, `research_notes/תוכנית הוליסטית לאפליקציית הלמידה/*.md`, `docs/research-log/*.md`, v1 report `reports/אפליקציית למידה משחקית לילדים.md`.

Known cross-spec issues already reported by spec authors (resolve those touching your files):
1. 01 vs 02 economy numbers: 02 wins on numbers; 01 must point to 02; campfire level must not reset per season; pet evolution tied to mastery; building thresholds stretched (02 config).
2. Hero vs machine robot: hero = "toy-suit with a living spark" (פנסון in 01), never called a robot; gear icon reserved for the machine robot (גלגלון / 07 ROBOT). Random capsule machine rejected (D2) — capsule machine shows next capsule.
3. Skill-level names: 08 (חדש/מתחיל/מתקדם/שולט/מאסטר; new…master) is canonical; 02 uses introduced…retained → needs one mapping in `packages/shared`.
4. Age bands T/G/S (07, 06) should be adopted across curriculum specs.
5. English time share: 05 asks 08 to raise English to 25% (K1) / 15% (K2).
6. Requests from 03 to 08 (pictorial+abstract mastery condition, partial credit, 5 math strands, 2 flags, ID map) and to 01 (new mechanics: balance scale, clock, shop, symmetry, number talk…).
7. Requests from 05 to 01 (minimal-pairs echo mode, MX-E05–E07, warm-up phase, Pip crab), to 09 (two larger English type sizes).
8. Requests from 10 to 11 (weekly report cron → Friday 12:00–14:00 window; alert queue every 15 min; drop flags[] from weekly payload; no device passcode at parent gate on shared kids' iPads; focus.span event; new tables/endpoints), to 08 (opt-in EMO_CHECKIN rule).
9. Identity: Sign in with Apple / push / App Attest likely unavailable on free Personal Team → phase-1 email-code / one-time setup code; Cloudflare Access for web dashboard; email for reports until paid program.
10. Install: Expo Go unavailable for SDK 58 → dev build via parent's Mac (free signing, 7-day re-sign); web artifacts for concept prototypes.
11. Vendor terms: no Gemini (incl. "Nano Banana") → GPT Image + Recraft (paid); ElevenLabs only with written approval.
12. Batch API 29-day retention → only K1/K2/K3 labels in prompts, never names.
13. Human-recorded audio banks: Hebrew ~300 phonics clips (04), English ~44 phonemes + letter names + ~60 nonsense words + song vocals (05).
14. New playbooks to make binding: `research_notes/תוכנית הוליסטית לאפליקציית הלמידה/{performance_reliability_playbook.md,quality_bug_prevention_playbook.md}`.
15. Network allowlist blocks expo.dev, api.cloudflare.com, etc. → deploy via GitHub Actions until parent allows hosts.

Rules for every reviewer:
- Edit ONLY your owned files (in place). Keep the file's language (Hebrew or English prose) and structure; improve, don't rewrite for style.
- Read 00 and every spec your files depend on; resolve contradictions in favor of: parent's explicit requests > 00 > the owning spec of that topic > evidence.
- For your domain, systematically enumerate use cases and edge cases (child × age × device × online/offline × first-run/returning × co-op × Shabbat/holiday × failure/frustration × parent actions × data loss/sync conflicts × accessibility × vendor/API failure) and make sure each is handled with a concrete rule, default, or acceptance criterion.
- Replace vague words ("appropriate", "some", "etc.") with numbers, IDs, rules. Add missing acceptance criteria. Mark anything still unverified with `⟂VERIFY` and a one-line how-to-verify.
- You may do targeted web research for gaps (log findings in `docs/research-log/fable-<your-id>.md`).
- Append a section "## Fable review changelog" at the end of each owned file: what you changed and why (bullets).
- Issues you find in files you do NOT own: write them to `docs/review/fable-issues-<your-id>.md` (file, section, problem, proposed fix). Do not edit others' files.
- Do not commit; the coordinator commits.
