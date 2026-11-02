# Research log — Fable reviewer F4 (05 English, 06 extra subjects, 07 future skills)

Date: 2026-11-02. No web access was used in this pass; items below are checks from background knowledge, each marked with how to verify. Anything marked ⟂VERIFY is also marked in the spec.

## Songs (05 §9.2 licensing)

| Song | Check | Status |
|---|---|---|
| Head, Shoulders, Knees and Toes | Tune = "There Is a Tavern in the Town" (1883, PD). The familiar **lyrics** are first attested 1961 (Rev. Robert Hall "Head, Shoulders…" is sometimes claimed; earlier printed sources unknown). | Kept TRAD-verify; 05 now uses ORIG lyrics on the PD tune until a pre-1927 print is found. Verify: search Gutenberg/HathiTrust for "head shoulders knees and toes" before 1927. |
| If You're Happy and You Know It | Disputed origin (Latvian folk "Kad man bija" claims; 1957 Jonico Music registration). | TRAD-verify; ORIG alternative "If you're ready…" already in 05. |
| Teddy Bear, Teddy Bear, Turn Around | Skipping rhyme; printed versions from the 1920s–30s; author unknown. | TRAD-verify kept. |
| Ten in the Bed | Traditional, 20th century; no clear author. | TRAD-verify kept. |
| Michael Finnegan (tune for ORIG "Eight Little Candles") | Earliest print I know of is 1920s; may not clear a strict pre-1927 bar. | ⟂VERIFY added in 05; fallback tune "This Old Man" (1906). |
| Itsy Bitsy Spider | Published 1910 ("Camp and Camino in Lower California", as "Spider Song"). | PD-safe OK. |
| This Old Man | First published 1906 (Anne Gilchrist, Journal of the Folk-Song Society). | PD-safe OK. |
| Happy Birthday to You | US PD after the 2016 Warner/Chappell settlement; EU since 2017. | OK. |
| Alphabet Song | Bradlee 1835 on the "Ah! vous dirai-je" tune. | OK. |
| Old MacDonald | Modern form in "Tommy's Tunes" (1917). | PD (US) OK. |

## Phonics sequence (05 §5)

Letters and Sounds Phase 2 sets (s a t p / i n m d / g o c k / ck e u r / h b f ff l ll ss) and Phase 3 (j v w x / y z zz qu / ch sh th ng / vowel digraphs) match 05 S1–S8 with US adaptations. Decodable sentences S1–S10 were hand-checked against the cumulative grapheme + tricky-word sets: no violations found once `-s` (plural and 3rd-person) and proper names are allowed (rule added to §5.6). Tricky-word count through S10 recounted = **40** (05 said 42; fixed). K3 core word list recounted = **46** (05 §7.1 said 40; fixed).

## Knowledge-pack facts (06 §12–13) spot-check

| Claim | Check | Status |
|---|---|---|
| Synodic month ≈ 29.53 days; Hebrew month 29/30 days | NASA / standard | OK (noted in 06 §12.5) |
| Israeli astronaut on the ISS 2022 | Eytan Stibbe, Ax-1, April 2022 | OK, keep † until `source` filled |
| Dinosaur footprints near Jerusalem | Beit Zayit site | OK, † |
| ~500 million birds migrate over Israel per season | Common figure (Hula/SPNI) | "מאות מיליוני" OK, † |
| Dead Sea = lowest land point | OK | — |
| Drip irrigation: Simcha Blass, Netafim/Hatzerim 1965 | OK | † |
| Anemone = national flower (2013 public vote) | OK | † |
| Golan volcanoes extinct/dormant | OK | wording "כבויים" fine |
| Makhtesh formed by erosion, not impact/volcano | OK | — |
| >200 bones (adult 206); 20 milk teeth | OK | — |

## Safety

- CPSC small-parts rule: parts that fit a 1.25" × 2.25" cylinder are a choking hazard for children under 3 → 06 §16.1 uses "< 32 mm" and the 00-C13 "under 36 months" derivation.
- Small high-powered magnets: ingestion of ≥2 magnets causes bowel injury; 06 restricts magnets to simulation while a child under 3 is at home.

## Open items for the next pass (web needed)

1. Michael Finnegan first print date (05 CANDLES).
2. Exact field name for the calm/"safe room" switch in 10/11 (`family.calmMode` used as a working name).
3. Whether 08 accepts per-skill `masteryProfile` and `kind: performance|exposure` (05 M-REC, 06 M-PERF/M-EXPO depend on it; both have fallbacks).
