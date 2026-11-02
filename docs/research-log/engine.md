# Research log: assessment, adaptive engine and child model

Spec served: `docs/spec/08-assessment-engine-child-model.md`. Date: 2026-11-02.

## Method and access limits

- WebSearch worked until this turn's shared search budget ran out. WebFetch reached only `github.com`. These hosts failed DNS (they are outside the session's network allowlist): `www.frontiersin.org`, `arxiv.org`, `www.justinmath.com`, `files.eric.ed.gov`. To read the primary PDFs below, those hosts have to be allowed in the session's network settings.
- Findings from those hosts come from search-result extracts. Formulas I derived myself, or took from background knowledge, are marked **[derived]** or **[unverified]**.
- Numeric test vectors in spec 08 §12 were computed with a scratch Python script (grid posterior, Elo, HSHS). They are reproducible from the formulas in the spec.

## 1. Elo with response time: Maris & van der Maas (2012) HSHS scoring rule

- Sources: Klinkenberg, Straatemeier & van der Maas 2011, *Computers & Education* 57(2):1813–1824 ([ScienceDirect](https://www.sciencedirect.com/science/article/abs/pii/S0360131511000418)). Ability and difficulty update after every item, there is no pre-calibration, and the scoring rule is explicit. Maris & van der Maas 2012, *Psychometrika* ([Springer](https://link.springer.com/article/10.1007/s11336-012-9288-y)).
- The secondary note [kilowatto/math-challenge](https://github.com/kilowatto/math-challenge/blob/main/docs/research/2026-07-31-mc-13-its-knowledge-tracing-elo.md) (fetched) gives the rule as `score = a_i·(d_i − RT)·(2·acc − 1)`, with an earlier form `acc·(d − RT)`. Its Elo expectation uses a generic 400-point logistic, not the HSHS expectation.
- **[derived]** Take the density f(x,t) ∝ exp(Δ·(2x−1)(1−t/D)) over normalized time u = t/D ∈ [0,1]. Integrating over u gives P(x=1) = σ(Δ). The log-partition is log(2·sinh Δ / Δ). The expected normalized score is therefore E[S] = coth(Δ) − 1/Δ (the Langevin function, ≈ Δ/3 near 0). The Elo gradient step is θ ← θ + K·(S − E[S]). This matches the published claim that the implied accuracy model is 2PL with the time limit acting as discrimination: in unnormalized time the logit is D·Δ.
- Later Math Garden practice replaced the RT-dependent penalty for wrong answers with a fixed penalty (2025 psychometrics paper, cited in `learning_science_workbooks.md`). We do not adopt this. HSHS is used only in K1 opt-in speed rounds.
- **Decision for the spec:** accuracy-only Elo by default. HSHS applies only when tier is K1, the skill is fluency-eligible, the skill is at level ≥ proficient, and the round is an opt-in speed round. This is reconciliation D11 in `gap_audit.md`.

## 2. Bolsinova et al. (2026) dynamic K

- Vermeiren, Hofman, Bolsinova, van der Maas & Van den Noortgate, "Balancing stability and flexibility: investigating a dynamic K value approach for the Elo rating system in adaptive learning environments", *UMUAI* 36(1), art. 4, DOI 10.1007/s11257-025-09439-z ([Springer](https://link.springer.com/article/10.1007/s11257-025-09439-z); [Tilburg](https://research.tilburguniversity.edu/en/publications/balancing-stability-and-flexibility-investigating-a-dynamic-k-val/)). The point is that shrinking K with the response count fails when ability changes a lot. A search snippet says K adjusts to observed rating-change trends, and that the method is implemented in Math Garden.
- **[unverified]** the exact K function. The spec therefore uses a transparent substitute: an uncertainty variable u ∈ [0,1], with K = Kmin + (Kmax − Kmin)·u. u falls with each scored response, rises with days since last practice (Glicko-like), and rises when the EWMA of residuals shows systematic drift. Drift is the trend signal the paper argues for.
- Pelánek's uncertainty function U(n) = α/(1+β·n) (Pelánek 2016 CAE; Pelánek et al. 2017 *UMUAI* 26(1):89–118, [PDF host](https://www.fi.muni.cz/~xpelanek/publications/umuai-adaptive-practice.pdf)). I could not confirm the parameter values (α≈0.8, β≈0.05) from search. The spec does not depend on them.

## 3. Knowledge Space Theory placement

- Falmagne & Doignon (1988) and Doignon & Falmagne (1999, ch. 10) describe stochastic assessment with per-item careless-error (β_q) and lucky-guess (η_q) parameters, a half-split or informative questioning rule, and Bayesian or multiplicative updating. These are implemented in R `kstMatrix::kmassess` ([rdrr](https://rdrr.io/cran/kstMatrix/man/kmassess.html)). Values ≥ 0.5 for either parameter are problematic.
- ALEKS Adventure (K–3) caps its knowledge check at 9 questions for K and 14 for grades 1–3. A knowledge state needs about log2(#states) questions (see `placement_assessment.md` §1).
- **Decision:** a single-family app has no data to estimate a full knowledge structure. The spec uses an IRT-equivalent of the half-split rule. Skills on a strand "spine" are ordered by a designer anchor. Phase B bisects the bracket of [highest success, lowest failure]. Phase R probes near the posterior mean. A grid posterior (EAP) over θ gives the SE stopping rule. Known / unknown / uncertain classes come from posterior predictive P per skill (≥0.85 / ≤0.40). The classes are then closed over the prerequisite DAG (downward and upward closure), as in KST. Careless and lucky errors are absorbed by the logistic model and the guess floor c = 1/nOptions.
- Simulation (scratch script): with prior N(0, 0.8²), item discrimination a = 1.5 and a spine spaced 0.5, SE ≤ 0.45 is reached in 8–11 items for K1. For K2 (prior SD 1.0, cap 8), the run stops at the cap with SE ≈ 0.53. With a = 1 (Rasch), about 18–21 items would be needed, which is too many for children. That is why a = 1.5 is the default for open/drag/build items, and a = 1.2 with a guess floor for tap-choice items.

## 4. Spaced repetition: Leitner, SM-2, FSRS

- FSRS-6 (fetched from the [awesome-fsrs wiki](https://github.com/open-spaced-repetition/awesome-fsrs/wiki/The-Algorithm)): R(t,S) = (1 + factor·t/S)^(−w20), factor = 0.9^(−1/w20) − 1, so R(S,S) = 0.9. Same-day update S' = S·e^{w17(G−3+w18)}·S^{−w19}. There are 21 default parameters [0.212, 1.2931, 2.3065, 8.2956, 6.4133, 0.8334, 3.0194, 0.001, 1.8722, 0.1666, 0.796, 1.4835, 0.0614, 0.2629, 1.6483, 0.6014, 1.8729, 0.5425, 0.0912, 0.0658, 0.1542]. The interval formula on the wiki is shown for FSRS-4.5 only.
- SM-2: I(1)=1, I(2)=6, I(n)=I(n−1)·EF, EF₀=2.5, min 1.3 (`learning_science_workbooks.md` §3).
- Latimier et al. 2021: expanding and uniform schedules do not differ significantly. Having spacing matters more than the exact schedule.
- **Decision:** a Leitner-style ladder, 1/3/7/14/30/60 days (D9). FSRS and HLR are out of scope with 3 users (gap audit §4.4). FSRS's R(t,S) shape is borrowed only to estimate recall for review priority: R = 0.9^(t/I).

## 5. Math Academy FIRe (Fractional Implicit Repetition)

- Skycak, "Optimized, Individualized Spaced Repetition in Hierarchical Knowledge Structures" ([justinmath.com](https://www.justinmath.com/individualized-spaced-repetition-in-hierarchical-knowledge-structures/); [Hecker review](https://frankhecker.com/2025/02/14/math-academy-part-7/)). A successful review of an advanced topic sends fractional credit down through "encompassings" (weights are often fractional). A failure sends a penalty. Review selection "compresses" reviews by picking tasks whose implicit credit covers other due reviews. The credit model also depends on student-topic learning speed. All evaluation is self-reported; there is no independent study.
- **[unverified]** the exact credit and penalty functions and the early-review discount. The full page could not be fetched.
- **Decision (FIRe-lite):** credit = product of encompassing weights, at most depth 3, cut off when the product falls below 0.2. Credit is discounted ×0.25 if the prerequisite's review is less than half its interval old. When accumulated credit reaches 1.0, it counts as a passed review. A failure sends no automatic demotion. If the chosen distractor's error tag maps to a prerequisite, that prerequisite gets a `checkSoon` probe.

## 6. ZPDES (Clement, Roy, Oudeyer & Lopes)

- JEDM 7(2):20–48, 2015 ([JEDM](https://jedm.educationaldatamining.org/index.php/JEDM/article/view/111); arXiv 1310.3174). The reward is learning progress: the correctness rate in the recent half of a window minus the older half. Mastered and impossible activities both score about 0. The ZPD set expands when an activity's success passes a threshold λ_ZPD. The method uses exploration rate ζ/γ and an expert-supplied coarse graph only. A 2024 study with 265 children ([arXiv 2402.01669](https://arxiv.org/html/2402.01669v2)) found ZPDES beat an expert linear curriculum, and choice helped only when combined with ZPDES.
- One tuned parameter set (hierarchical-bandit paper 2024, [arXiv 2408.07208](https://arxiv.org/html/2408.07208v1)): γ = 0.1, w₀ = 0.5. It was tuned on simulations and is not canonical.
- **Decision:** ZPDES-lite for choosing among *ready* (fringe) skills. LP_k = |mean(x, last 4) − mean(x, previous 4)|. Selection weight = priority + LP, with ε = 0.2 uniform exploration. The "ZPD" is the KST fringe plus in-progress skills.

## 7. BKT parameters

- 4 parameters per skill: P(L0), P(T), P(G), P(S). Illustrative values: 0.36, 0.1, 0.3, 0.05 (van de Sande 2013, via kilowatto). Mastery when P(L) ≥ 0.95 (Corbett & Anderson 1995). Degeneracy: Baker, Corbett & Aleven 2008 treat G or S > 0.5 as degenerate. A course deck attributes stricter bounds of G < 0.3 and S < 0.1 to Corbett & Anderson 1995 ([UPenn slides](https://learninganalytics.upenn.edu/MOOT/slides/W004V002v6.pdf)) **[secondhand]**.
- **Decision:** BKT runs in shadow mode only, for later comparison. D9's rule-based mastery is the single definition. Shadow parameters: P(L0) from placement, P(T)=0.15, P(G)=min(0.3, 1/nOptions), P(S)=0.1.

## 8. Cold start

- Math Garden starts items from arbitrary ratings and lets them self-calibrate over very large samples. That is not available to a family of 3, as the gap audit notes ("item difficulty will barely move").
- **Decision:** item β = skill anchor + template feature deltas (designer priors) + a learned per-template offset (bounded ±0.5) + an item residual (bounded ±0.6, K_item ≤ 0.04). All three children share item ratings. Child θ starts from the grade position, then parent-questionnaire pseudo-observations (weight 0.7), then placement.

## 9. Rapid-guess and disengagement thresholds

- Wise & Kong 2005 introduced response-time effort. The normative threshold NT10 is 10% of the item's mean RT; Wise & Ma 2012 report high classification accuracy. Effort-moderated scoring drops rapid guesses, and more liberal thresholds removed about 2/3 of the distortion in a retest study ([Frontiers 2023](https://www.frontiersin.org/articles/10.3389/feduc.2023.1127644); [Bohrium summary](https://www.bohrium.com/paper-details/using-retest-data-to-evaluate-and-improve-effort-moderated-scoring/812594362742472705-7571)). Grade-3 RTE was 0.991, so disengagement was rarer than at grade 8.
- NWEA MAP Growth: a rapid guess is an answer in < 10% of the item's average time, with a 10 s maximum. Three successive rapid guesses means disengaged ([NWEA blog](https://nwea.org/blog/?p=24854); [research brief](https://www.nwea.org/uploads/2020/03/researchbrief-what-happens-when-test-takers-disengage-understanding-and-addressing-rapid-guessing-2019.pdf)).
- I found no published thresholds for ages 5–9 on tablets. **Decision:** threshold = max(age floor, 0.1 × the child's own median RT for that template). Age floors: K1 600 ms (reading items 1200 ms), K2 800 ms. RT is measured from `answerableAt` (end of the spoken prompt), never from screen show. Three successive rapid guesses form a disengagement episode.

## 10. Frustration and wheel-spinning heuristics

- Wheel-spinning (Beck & Gong 2013, AIED): no 3 correct in a row within the first 10 practice opportunities. Others use 10 attempts without mastery ([ERIC ED594575](https://files.eric.ed.gov/fulltext/ED594575.pdf); [JEDM decision trees](https://jedm.educationaldatamining.org/index.php/JEDM/article/view/210)).
- Log-based frustration detection works at coarse grain from consecutive errors, time between attempts and error counts. Fine-grained detection is weaker (Rodrigo & Baker, programmers). Rapid repeated clicking is a common HCI frustration cue. I found no child-specific tapping or idle detector.
- **Decision:** a D10 deterministic guard (2 errors → hint + step down; 3 → worked example or switch; under 6 never more than 3 misses in a row), plus a transparent additive frustration score over the last 5 items (threshold 4), plus a wheel-spinning detector (≥10 scored attempts with no level gain and LP ≈ 0 → `stuck`). The detector gives coarse signals only: it drives in-the-moment adaptation and is never a trait label.

## 11. Israeli calendar (not verified this session)

- Search budget was exhausted before I could verify the Ministry's תשפ"ז holiday calendar. The spec treats calendar dates as a content-pack config (`IsraeliSchoolCalendar`) to verify at build time. Approximate dates from background: Hanukkah begins evening 4 Dec 2026; Pesach begins evening 21 Apr 2027; Shavuot 11 Jun 2027; elementary school ends around 20 Jun 2027.

## Open gaps carried into the spec

1. No validated Hebrew cut scores (letter knowledge, PA, RAN, grade-3 WCPM). Flag thresholds are provisional config values, tied to persistence over weeks plus parent observation.
2. Bolsinova dynamic-K formula, Math Garden uncertainty constants, ZPDES canonical thresholds and FIRe exact functions are unread. The spec uses transparent substitutes, each marked as such.
3. There is no validation of Elo or placement under age 6. K2 placement stops at the cap with a wider SE, which is shown to the parent as a range. K3 has no test.
