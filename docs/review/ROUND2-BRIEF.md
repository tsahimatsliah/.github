# Round 2 Brief — resolve all cross-file issues

Round 1 (F1–F9) is done. Each reviewer filed issues for files they did not own in `docs/review/fable-issues-F1.md … F9.md` (~760 lines total; F9's 64 GAPs in `docs/spec/15-use-cases-journeys-edge-cases.md` §3 are also mirrored in `fable-issues-F9.md`). Some issues were already processed in round 1 by the owning reviewer (each spec's "## Fable review changelog" says which), many were not.

Your job (one reviewer per file cluster):
1. Read `docs/review/REVIEW-BRIEF.md` (rules still apply: edit only your files, keep language/structure, parent > 00 > owning spec > evidence).
2. Read EVERY `docs/review/fable-issues-F*.md` and spec 15 §3 GAPs; extract every item whose target is one of your files.
3. For each item: check the current file — if already resolved, mark "already resolved"; otherwise apply a concrete fix (numbers, IDs, rules, ACs) or, if you disagree, write a short rationale and keep the current text consistent with 00.
4. Make sure your files are consistent with the CURRENT versions of the specs they reference (others are editing in parallel only within their own files — re-read referenced sections before finalizing). Canonical owners: economy numbers → 02; levels/bands/flags/engine → 08; identity/API/events/schemas/crons → 11; UX tokens/components/screens → 09; performance/quality → 14; pipelines/vendors/recordings → 12; roadmap/parent checklist → 13; decisions → 00.
5. Write `docs/review/round2-<YOUR-ID>.md`: a table of every issue (source file + item id → disposition: applied / already resolved / rejected+why / forwarded-to-owner) and any NEW issues for other owners (keep these few; only real blockers).
6. Append "### Round 2 changelog" under your files' "## Fable review changelog".
7. Do not commit.
