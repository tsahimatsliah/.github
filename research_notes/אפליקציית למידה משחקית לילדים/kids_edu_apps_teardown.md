# Teardown of Leading Children's Educational Apps: UX, UI, Onboarding, Progression and Engagement Lessons for a Personalized Learning Game (ages 2-10; Israeli family with kids aged 2.5, 5.5, 8.5)

Research note on method and limits: this round used about 25 web searches. Every WebFetch attempt failed because the cloud session's network allowlist blocks blog.duolingo.com, lennysnewsletter.com and sketch.com, and other hosts probably fail the same way. Most findings below therefore come from search-result summaries of the cited pages, not full-page reads. Numbers that only appeared in secondary write-ups are marked as such. Several apps on the requested list (Lingokids, Bini, Montessori Preschool, Reading Eggs/Mathseeds, Epic!, Monster Math, Moose Math, Marbotic, Osmo, PBS Kids, Matific, Kahoot! Kids, Speakaboos, Gus on the Go) got little or no sourced coverage. They are listed under Gaps.

## 1. Duolingo (main app): core loop, streaks, leagues, hearts to energy, path, notifications, and measured retention effects

### Takeaway
Duolingo's engagement engine has four parts: a linear path on the home screen, a daily streak with forgiveness tools (freeze, wager, amulet), weekly leagues of about 30 people, and carefully run notifications. Duolingo credits this engine with growing DAU 4.5x in four years. Its A/B tests show that each mechanic moves retention by small single-digit percentages, and that these gains compound. In 2025 the free-tier "hearts" penalty for mistakes was replaced by "energy", which is spent on every exercise. Duolingo says engagement and conversion rose, but free users protested strongly.

### Cited Findings
**Overall growth and the metric they optimize**
- Former CPO Jorge Mazal says a handful of product changes, together with other Product and Marketing efforts, grew DAU 4.5x over four years. He started from a slowdown in user growth, when Duolingo was already the most downloaded education app. — [Lenny's Newsletter, "How Duolingo reignited user growth" (via search summary)](https://www.lennysnewsletter.com/p/how-duolingo-reignited-user-growth); same 4.5x figure in [Reach Capital](https://www.reachcapital.com/?p=5907)
- Mazal's first retention attempt, a free month of Super Duolingo, failed because the best and most active users were already subscribers. — [Lenny's Newsletter (via search summary)](https://www.lennysnewsletter.com/p/how-duolingo-reignited-user-growth)
- The team's North Star metric was CURR (current user retention rate): the probability that a user who visited every week for the last two weeks visits again this week. Secondary write-ups report CURR rose 21% and daily churn fell more than 40% over four years. — [Marisha Lakhiani Substack summarizing Mazal](https://marishalakhiani.substack.com/p/breaking-down-duolingos-growth-model) (secondary)
- Mazal said that "probably the most successful mechanic in Duolingo is the home screen", meaning the path that shows the learner advancing through the curriculum. In an Amplitude AMA he said the homescreen redesign "created a new growth hill" and a route to personalization. — [Reach Capital](https://www.reachcapital.com/?p=5907); [Amplitude AMA](https://amplitude.com/community/events/ama-jorge-mazal-duolingo)

**Leagues / leaderboards**
- Duolingo's own blog says leaderboards raised time spent learning by about 17%. It notes that "typically if our experiments can move a major metric by 1%, that's a really good outcome". The current system was the fourth iteration and was modeled on leaderboards from other games. — [Duolingo blog, "Time spent learning well" (via search summary)](https://blog.duolingo.com/time-spent-learning-well)
- Secondary sources say the league design was modeled on FarmVille 2. They also say it tripled "highly engaged learners", defined as at least 1 hour a day, 5 days a week. — [Elite Game Developers Substack](https://elitegamedevelopers.substack.com/p/investing-in-language-learning-apps?open=false) (secondary; the tripling is not confirmed in Duolingo's own blog text that I could see)
- Duolingo says the growth in total learning time after competitive features like Leaderboards came mostly from very active learners. — [Duolingo blog, "Time spent learning well"](https://blog.duolingo.com/time-spent-learning-well)
- Conflict: one secondary source says leaderboards raised DAU by 17%. That looks like a confusion with the 17% learning-time figure. — noted in search summary of [marishalakhiani Substack](https://marishalakhiani.substack.com/p/breaking-down-duolingos-growth-model)
- Third-party claims that leagues raised lesson completion by 25% have no primary Duolingo source. — [StriveCloud](https://www.strivecloud.io/blog/gamification-examples-boost-user-retention-duolingo) (unverified)

**Streaks**
- In Duolingo's Streak Wager A/B test, Day-7 retention showed the largest improvement, +14%. In the Weekend Amulet test, learners with the amulet were about 4% more likely to return a week later and about 5% less likely to lose their streak. — [Duolingo blog, "How Streaks keep Duolingo learners committed" (via search summary)](https://blog.duolingo.com/how-streaks-keep-duolingo-learners-committed-to-their-language-goals/)
- Duolingo reports that a streak milestone animation improved Day-7 retention of brand-new learners by 1.7%. — [Duolingo blog, "streak milestone design animation" (via search summary)](https://blog.duolingo.com/streak-milestone-design-animation)
- A third-party write-up says a test that let one completed lesson extend the streak (lowering the bar) gave +3.3% relative D14 retention and +1% DAU. — [Okara blog](https://okara.ai/blog/how-duolingo-grew) (unverified secondary)
- A design newsletter says Duolingo changed streak mechanics after users found the pressure discouraging. It credits the change with a 40%+ increase in learners holding streaks of 7 days or more. — [Designfolio Substack](https://designfolio.substack.com/p/crazy-ux-redesign-duolingo) (secondary)
- Duolingo has published a product talk on streaks by Jackson Shuttleworth (Group PM, Retention). — [video summary](https://videohighlight.com/v/_CCwoQZH5hI)
- Duolingo has A/B tested push-notification copy and timing around streaks. — [Taplytics case](https://taplytics.com/blog/how-duolingo-ran-an-experiment-on-their-streaks-feature)

**Criticism of streak pressure**
- A habit-design critique describes a guilt-based notification sent to a child who missed a lesson. It argues that users stop calling the streak "motivating" and start calling it "pressure". — [JTBD UX protocol](https://jtbdux-protocol.pages.dev/adoption-and-behavior/behavior-design-patterns/how-duolingo-engineered-50-million-daily-users-and-the-backlash-that-came-with-it)
- Critics argue that streaks drive high-frequency, low-depth engagement ("shallow learning trap"). — [DEV Community](https://dev.to/yaptech/duolingos-shallow-learning-trap-gamified-streaks-harmful-habits-4134); [Jiwon Yoo case study](https://jiwonvyoo.github.io/portfolio/case/duolingo/duo.html)

**Hearts replaced by Energy (2025)**
- The Q2 2025 investor materials describe Energy as a new pacing system for free users that replaces Hearts. Secondary summaries say the materials credit it with higher engagement and conversion. — [Duolingo Q2 2025 shareholder letter](https://investors.duolingo.com/static-files/0b55110c-2eb9-466d-8549-5459e0851290); [Finviz Q2 deep dive](https://finviz.com/news/136389/duol-q2-deep-dive-new-features-and-international-markets-fuel-growth)
- Mechanics: free users get a 25-unit energy bar instead of 5 hearts. Hearts were lost only on mistakes, but energy is spent on every exercise, so even a perfect lesson drains it. It fully recharges in about a day. Super/Max subscribers get unlimited energy. Duolingo says learners do more under Energy than under Hearts. — [Duolingo blog "Duolingo energy" (via search summary)](https://blog.duolingo.com/duolingo-energy/); [Duolingo Wiki – Energy](https://duolingo.fandom.com/wiki/Energy)
- Testing began around July 2025 and rolled out on iOS first. — [Duolingo Wiki](https://duolingo.fandom.com/wiki/Energy); [Nuclear Monster, May 2025](https://nuclearmonster.com/2025/05/duolingo-switches-from-hearts-to-energy-adds-chess) (the dates conflict: wiki says July 3 2025 testing, the Nuclear Monster headline is from May 2025)
- Backlash: several change.org petitions to restore Hearts, and press stories of users quitting. — [change.org](https://www.change.org/p/duolingo-hearts-not-energy); [Android Authority](https://www.androidauthority.com/quitting-duolingo-energy-system-3599842/); [TechIssuesToday](https://techissuestoday.com/duolingo-energy-system-user-backlash)

### Inferences
- The highest-leverage mechanic is the visible path, not the streak or the league. For kids, an always-visible "where I am / what's next" map probably matters more than any currency.
- Streak forgiveness (freeze, weekend amulet, one-lesson minimum) gave measurable gains. For children, whose schedules parents control (Shabbat, holidays, sick days), a streak needs built-in forgiveness, or it should be a weekly or "family" streak instead of a strict daily one.
- Leagues mostly boosted already-very-active users. Public competition is risky for ages 2-8 and probably useful only for the 8.5-year-old in opt-in, friendly forms (family or co-op leagues).
- Energy charges for practice itself, so it limits learning even when the child succeeds. A children's product should not copy it. If session pacing is needed, use a non-monetized "come back tomorrow" story beat.

### Gaps
- Full Duolingo blog texts, the Lenny's essay body and the 10-K could not be fetched (network blocked). Exact numbers for streak freeze impact, Streak Society, and notification "protect the channel" rules are unverified here.
- No primary source found for 2026 changes (AI features such as Video Call / Max, chess/music/math expansion beyond the chess mention).

## 2. Duolingo ABC, Duolingo Math, and Khan Academy Kids (free, mission-driven early learning)

### Takeaway
The two strongest free reference apps for the 2.5-6 band are Duolingo ABC and Khan Academy Kids. Both have no ads and no in-app purchases. Both use a mascot-guided, mostly linear path with short activities and audio-first instructions. Khan Kids adds an open "library" alongside the path. Neither has strong independent efficacy studies.

### Cited Findings
- Duolingo ABC: Common Sense Media rates it 3+. It has 127 units covering each letter, tracing, reading and rhyming. Kids cannot choose where to start on the learning path, and CSM suggests it is best in small doses. — [Common Sense Media – Duolingo ABC](https://www.commonsensemedia.org/app-reviews/duolingo-abc-learn-to-read)
- Duolingo confirmed no ads or in-app purchases in ABC. It is free, needs no login, works offline, and targets ages about 3-6 (some listings say 3-8 or preschool to grade 2). Speech recognition may be inaccurate for young children. — [TopTenReviews](https://toptenreviews.com/duolingo-abc-launches-early-helping-young-kids-learn-to-read-and-write-for-free); [Educational App Store](https://www.educationalappstore.com/app/duolingo-abc); [Inverse](https://www.inverse.com/input/tech/duolingos-new-ios-app-wants-to-teach-kids-how-to-read)
- A claimed EDC study showing a 28% literacy gain after 9 weeks is unverified. — reported in [GoodEReader](https://goodereader.com/blog/?p=351856) (treat as unconfirmed)
- Khan Academy Kids is "100% free with no ads or subscriptions". It targets ages 2-8, is aligned with Head Start and Common Core, and is COPPA compliant. — [App Store listing](https://apps.apple.com/app/1378467217)
- Characters: Kodi the bear is the guide who leads kids through activities. Ollo the elephant handles phonics and letter sounds (one source wrongly calls Ollo a dingo). — [App Store / APKMirror release notes](https://www.apkmirror.com/apk/khan-academy/khan-academy-kids)
- Khan Kids' release notes cite Common Sense Media ratings of 5 stars for Educational Value and 5 stars for Ease of Play. This is self-reported in the listing. — [APKMirror release 2.0](https://www.apkmirror.com/?p=628512)
- Khan Academy's efficacy evidence covers the main K-12 product, not Kids. It reports an effect size of 0.36 and about 20% higher-than-expected gains in a study of about 350K students in grades 3-8 who used it 30+ min/week. — [Khan Academy efficacy results 2023-24](https://2023-2024.annualreport.khanacademy.org/efficacy-results); [Khan blog](https://blog.khanacademy.org/multiple-studies-show-khan-academy-drives-learning-gains-evidence-for-our-platforms-effectiveness)
- Khan Kids vs ABCmouse comparison guides contrast free/no ads with a paid subscription. — [ScreenWise](https://screenwiseapp.com/guides/khan-academy-kids-vs-abcmouse-which-is-right-for-your-family)

### Inferences
- "Free, no ads, no IAP" is what makes parents trust Khan Kids and Duolingo ABC. A paid product needs a clear parent-side paywall and a child-side experience with no selling.
- A fixed starting point is a weakness CSM flags (kids "can't pick where to start"). A placement step or adaptive entry is a cheap win, especially for siblings of different ages sharing one device.

### Gaps
- No source found on Duolingo Math (kids/adult math app) design or its current status in 2026. It may have been folded into the main Duolingo app; not verified.
- No sourced details on Khan Kids' library-vs-path structure, the full character cast (Peck, Reya, Sandy, etc.), or celebration animations. The design-blog posts were not retrieved.

## 3. Prodigy Math: RPG battle loop and pay-to-win monetization criticism

### Takeaway
Prodigy wraps math in a fantasy RPG where answering questions powers battles, with pets and cosmetics. It is free in school but heavily upsells memberships at home. Watchdogs filed an FTC complaint about bait-and-switch and manipulation, and Common Sense Media calls it pay-to-win. It is the clearest case to avoid.

### Cited Findings
- On Feb 19, 2021, Fairplay (then CCFC) and 21 groups asked the FTC to investigate Prodigy for deceiving educators and parents and for targeting children. They also said it lacks substantiation for claims of boosting math ability. — [Fairplay press release](https://fairplayforkids.org/feb-19-2021-advocates-to-ftc-prodigy-math-game-preys-on-kids-and-families/); [Complaint PDF](https://fairplayforkids.org/wp-content/uploads/2021/02/Prodigy_Complaint_Feb21.pdf); [Axios](https://www.axios.com/2021/02/19/prodigy-math-game-ftc-complaint)
- The Premium annual membership cost up to $107.40. By September 2021 an "Ultimate" tier cost over $180/yr. Members-only rewards (sparkly treasure boxes) give advantages in battles against classmates who are not members. — [Fairplay](https://fairplayforkids.org/pf/prodigy/); [MediaPost](https://www.mediapost.com/publications/article/360719/ftc-urged-to-investigate-prodigy-math-game-over-al.html)
- Fairplay's test: in 19 minutes of play it saw 16 membership ads and only 4 math problems. — [Fairplay complaint](https://fairplayforkids.org/feb-19-2021-advocates-to-ftc-prodigy-math-game-preys-on-kids-and-families/)
- Common Sense Media says essential features, including evolving pets and basic content access, are locked behind several costly membership tiers, "transforming it into a pay-to-win model". — [Common Sense Media – Prodigy](https://www.commonsensemedia.org/app-reviews/prodigy-kids-math-game)
- Prodigy's spokesperson said the game notifies players about memberships "from time to time". I found no FTC enforcement outcome. — [Axios](https://www.axios.com/2021/02/19/prodigy-math-game-ftc-complaint)

### Inferences
- The RPG battle loop (correct answer = attack) is very motivating for 7-10-year-olds. The lesson is to keep that loop and remove the commercial pressure: no member-only power, and no upsell shown to the child.
- The ratio of math to non-math time is a real design metric. Track "learning seconds per session minute" and keep the meta-game from taking over.

### Gaps
- No sourced detail on Prodigy's adaptive placement engine, its curriculum mapping, or 2025-2026 changes (e.g., Prodigy English).

## 4. Open-ended, no-failure "digital toys": Toca Boca (Toca Life World), Pok Pok, Sago Mini

### Takeaway
The most-awarded design philosophy for ages 2-6 removes goals, scores, failure, text and menus entirely ("digital toys"). Children explore cause and effect on their own, without a parent. Pok Pok won a 2021 Apple Design Award (Delight & Fun) on this basis. Toca Boca built its brand on "no winning, no losing" and gender-neutral design.

### Cited Findings
- Pok Pok Playroom has "no goals, no winning — just play". Its founders built it because existing apps were "extremely gamified or very pedagogical" and heavy on language and UI, so kids constantly needed a parent's help. — [Sketch blog – Pok Pok](https://sketch.com/blog/pok-pok) (via search summary)
- Pok Pok's press kit: "no rules, instructions, menus or language so kids can play and learn all on their own". — [Pok Pok press kit](https://playpokpok.com/press/press-kit/)
- Founders: Esther Huybreghts (CCO), Mathijs Demaeght (VP Design), Melissa Cash (CEO). Pok Pok won the 2021 Apple Design Award in Delight & Fun about three weeks after launch, and a 2023 App Store Award. It was named to TIME's 100 Most Influential Companies of 2025. — [Pok Pok blog – ADA](https://blog.playpokpok.com/pok-pok-playroom-is-a-202-1-apple-design-award-winner-cf07e4b1f450); [Pok Pok – App Store Award](https://playpokpok.com/blog/pok-pok-wins-an-app-store-award/); [VRT NWS](https://www.vrt.be/vrtnws/nl/2025/06/26/belgische-kinderapp-pokpok-bij-100-meest-invloedrijke-bedrijven/) (TechCrunch's tag page dates the ADA to 2023, which conflicts with the company's own 2021 account; the 2023 date matches the App Store Award)
- MacStories interviewed Pok Pok as a 2021 ADA winner. — [MacStories AppStories ep. 227](https://www.macstories.net/podcasts/appstories/227/)
- Toca Boca describes itself as "digital toys": no winning, no losing, no stress, no high scores, no rules. Its head of consumer products, Mathilda Engman, frames the mission as open-ended play in games that do not have to be "won". — [Kidscreen](https://kidscreen.com/?p=59849); [Killscreen – Toca play designer](https://www.killscreen.com/what-designers-can-learn-about-play-children/)
- Toca Boca's design is child-centered, gender-neutral and open-ended. Its classic apps carried no IAP or third-party ads (e.g., Toca Nature, 2014). — [Kidscreen – Toca Nature](https://kidscreen.com/?p=60776); [Motionographer – design process](https://motionographer.com/2016/04/27/the-design-process-behind-toca-bocas-infectious-apps/)
- Reviewers see the lack of a "right way to play" as a feature that lets kids explore consequences without judgment and without social comparison. — [ScreenWise – best Toca Boca games](https://screenwiseapp.com/guides/the-best-toca-boca-games-for-kids-and-creativity)
- Sago Mini School (ages 2-5) was built with child development experts and covers early reading, math, science and problem solving. Sago Mini World offers subscription access to hundreds of activities. — [Sago Mini School](https://sagosago.com/school/); [App Store – Sago Mini World](https://apps.apple.com/tm/app/id874425722?l=en-gb)

### Inferences
- For the 2.5-year-old, the right mode is a no-fail sandbox: tap anything and something delightful happens, with no wrong answers, no timers and no text. Learning (colors, counting, Hebrew letter sounds) should come in through play, not through quizzes.
- One product can host both philosophies. A "playroom" mode with no goals for toddlers and free play, and a "journey" mode with structure for the 5.5- and 8.5-year-olds. Rewards earned on the journey can unlock new toys in the playroom.

### Gaps
- Could not retrieve Pok Pok's or Toca's detailed playtesting methods, sound-design notes, or monetization numbers (Toca Life World's shift to IAP/free-to-play and parent complaints about it are not sourced here).

## 5. Literacy apps: Endless Alphabet/Reader/Numbers (Originator), Teach Your Monster to Read, Homer, Reading Eggs

### Takeaway
The best-loved pre-reader literacy apps rely on charming characters (monsters) and animated words or letters that act out their meaning or sound. There is no failure state. Kids drag letters into slots and hear their phonics sounds. Evidence ranges from expert approval of systematic phonics (TYMTR) to small, company-hosted RCTs (Homer). None has strong independent proof.

### Cited Findings
- Endless Alphabet (CSM 3+) works more as a vocabulary tool than an alphabet app. Kids go at their own pace with "no objective or goals beyond exploring the words". The monsters are engaging, but kids can't track which words they've mastered. — [Common Sense Media – Endless Alphabet](https://www.commonsensemedia.org/app-reviews/endless-alphabet); [Common Sense Education](https://www.commonsense.org/education/app/endless-alphabet)
- Endless Reader (Originator, formerly Callaway Digital Arts): a stampede of monsters scatters the letters, and the child drags them back into order to spell the word. The monsters carry over from Endless Alphabet. — [Horn Book – Endless Reader](https://www.hbook.com/story/endless-reader-app-review); [Horn Book – Endless Numbers](https://www.hbook.com/story/endless-numbers-app-review)
- Children and Media Australia flags in-app purchasing and online interactivity in Endless Alphabet. — [CMA](https://childrenandmedia.org.au/app-reviews/apps/endless-alphabet)
- Endless Spanish: "super cute monsters help kids learn Spanish phonics" (CSM 4+). — [CSM – Endless Spanish](https://www.commonsensemedia.org/app-reviews/endless-spanish)
- Teach Your Monster to Read: Common Sense Education calls it an enjoyable, engaging way to practice phonics, but it lacks higher-order thinking and does not use groundbreaking game design ("Phonics adventure oozes charm, has limited learning potential"), Pre-K to 2. — [Common Sense Education – TYMTR](https://www.commonsense.org/education/app/teach-your-monster-to-read)
- A literacy teacher's review calls TYMTR one of the few programs closely aligned with the Science of Reading, with systematic, sequential letter-sound teaching. It should still be only part of a plan for dyslexic children. — [Modulo review](https://www.modulo.app/all-resources/teachyourmonstertoread)
- Homer claims a "74% increase in early reading scores". The underlying study (Susan Neuman) was a randomized study in 7 Head Start classrooms, 82 children, 6 weeks, 15 min/day. The 74% is a pre-post gain in phonological awareness in the treatment group, and the control used a math app. A critic notes the control group started ahead and then declined. The study is hosted on Homer's own site. — [Homer efficacy PDF (Neuman, NYU)](https://site-static.learnwithhomer.com/static/pdf/can-an-app-close-the-literacy-gap-efficacy-learn-with-homer-susan-neuman-nyu.pdf); [Homer study PDF](https://static.learnwithhomer.com/deployedassets/1db00f93/static/pdf/Efficacy%20Study%20HOMER.pdf)

### Inferences
- "Animated word" (a word that acts out its meaning) and "letter that says its sound when touched" are the core interaction patterns for Hebrew. They map well to letters with nikud, where each consonant-plus-vowel combination makes one sound.
- A gap in Endless Alphabet (no mastery tracking) is an opportunity: keep the free, joyful exploration, but quietly log mastery to a parent dashboard.
- Efficacy marketing claims in this category are often inflated. Plan to measure your own learning outcomes simply (pre/post skill probes inside the game).

### Gaps
- Reading Eggs/Mathseeds (map, golden eggs, critters) returned no sourced details. Epic! (digital library, reading logs) and Speakaboos were not covered.

## 6. Adaptive math: DragonBox (Kahoot!), Math Garden (Rekentuin)

### Takeaway
Math Garden is the best-documented adaptive engine. It uses an Elo-style rating that updates both child ability and item difficulty after every answer, factors in response time, and aims for about 75% success. DragonBox shows that hiding the math inside a puzzle is very engaging. Its transfer to standard school tests is weak or mixed.

### Cited Findings
- Math Garden updates child ability and item difficulty after each answer using an Elo-based model that also factors in response time. Items are sampled at a mean success probability of 0.75 ("challenging but not too hard"). — [Math Garden: A new educational and scientific instrument (UvA)](https://dare.uva.nl/id/67ab8977-5e36-4f9f-962d-be294f2b8c09); [UvA DARE paper](https://dare.uva.nl/id/de65bb65-d198-4dbc-ac7b-bc25ca23e627)
- A 2023 study of adaptive math programs reports the same average of about 75% correct, with corrective feedback after each task. It examined effects on performance, self-concept and anxiety. — [PMC10299571](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10299571/)
- Math Garden coins earned in the math games buy decorative trophies in the menu system. — [CORE paper](https://core.ac.uk/download/161527697.pdf)
- The related Dutch Snappet software also re-estimates ability after each item and picks items with about a 75% chance of success ("chess rating for sums"). — [Scientias.nl](https://scientias.nl/schaakrating-voor-rekensommen-wat-levert-adaptieve-software-op-de-basisschool-op/)
- DragonBox Algebra: in an RCT with 190 7th/8th graders over 5 periods, DragonBox students solved more problems and enjoyed it more, but the Lynnette tutor group scored significantly higher on the post-test. A second study (267 students) found no greater gains than the tutoring system despite higher enjoyment. A Carnegie Mellon finding called it "ineffective" for standard equation solving, and in-game learning "does not transfer". — [ACM paper (unpaywall)](https://unpaywall.org/10.1145%2F3057889); [EdSurge](https://edsurge.com/news/2016-03-13-enter-the-dragonbox-can-a-game-really-teach-third-graders-algebra)
- A 2023 AERA Open RCT: both DragonBox and From Here to There! improved 7th graders' conceptual understanding of equivalence, and FH2T's effect was larger for higher-pretest students. — [AERA Open / DOAJ](https://doaj.org/article/28d05a10d8ea487f80d0c3b6811e3e9d); [FH2T project](https://sites.google.com/view/from-here-to-there/research-findings_1/summary-of-findings)
- The DragonBox developers' Algebra Challenge: after 1.5 hours, 93% of students solved three algebra problems in a row. This measures in-game mastery, not transfer. — [EdSurge](https://edsurge.com/news/2016-03-13-enter-the-dragonbox-can-a-game-really-teach-third-graders-algebra)
- Kahoot! Numbers by DragonBox targets ages 4-8 and is subscription-based. Common Sense lists its learning rating as "not yet reviewed", and I found no independent efficacy study. — [Common Sense Education – Kahoot! Numbers](https://www.commonsense.org/education/reviews/kahoot-numbers-by-dragonbox); [CSM – DragonBox Numbers](https://www.commonsensemedia.org/app-reviews/dragonbox-numbers)

### Inferences
- An Elo-style dual rating (child skill × item difficulty) aiming for about 75-80% success is a proven, cheap personalization core. The same item bank can serve all three kids at different difficulty levels.
- Hidden-math puzzles (DragonBox) need an explicit "bridge" step that re-expresses the puzzle in school notation. Otherwise skills stay stuck in the game.
- The 75% target means a child fails about 1 in 4 items. With no penalty and with gentle corrective feedback (no lost lives), that failure becomes a learning moment.

### Gaps
- Math Garden's garden metaphor (plants growing or wilting with practice) could not be confirmed from sources. Matific, Monster Math, Moose Math (Duck Duck Moose/Khan) and Osmo/Marbotic (tangible play) were not covered.

## 7. Pre-reader and early-reader UI patterns (audio prompts, big buttons, no text, parental gates)

### Takeaway
Award-winning apps for pre-readers remove text, menus and instructions entirely. The child navigates by tapping characters and objects, with audio narration giving every prompt (Pok Pok, Toca). Kid-safe navigation is meant to work without supervision (Lingokids). Parent-facing functions such as purchases and settings sit behind gates. Billing and cancellation flows that fail parents are one of the category's largest sources of complaints.

### Cited Findings
- Pok Pok's design has no rules, instructions, menus or language, so kids can play on their own. The founders identified language- and UI-heavy apps as the reason kids need parents' help. — [Pok Pok press kit](https://playpokpok.com/press/press-kit/); [Sketch blog](https://sketch.com/blog/pok-pok)
- Lingokids cites "Safety" as a core design principle that allows "a kid-friendly navigation system" designed for play without supervision. The app mixes videos, songs, activities and games that adapt to the child's level. — [Lingokids](https://lingokids.com/?p=16482); [Learning Counsel](https://thelearningcounsel.com/articles/app-of-the-week/the-playlearningtm-app-for-kids-ages-2-8-years-old/)
- Duolingo ABC relies on audio and tracing for 3-6-year-olds, and its speech recognition can struggle with young children's voices. — [Educational App Store](https://www.educationalappstore.com/app/duolingo-abc)
- ABCmouse parents reported being routed to an upgrade screen when trying to reach the parent section to cancel. — [Trustpilot – ABCmouse](https://trustpilot.com/review/www.abcmouse.com?page=2)

### Inferences
- Pre-readers (2.5 y/o): every screen needs audio, big touch targets, no text, no time limits and no failure. The character speaks the prompt and repeats it on tap. Early readers (5.5): text with audio highlighting (karaoke-style) and nikud. Fluent readers (8.5): text first, with optional audio.
- Speech recognition for young children is unreliable. Use it only for no-fail "say it" moments, never as a gate to progress.

### Gaps
- No sourced specifics on parental-gate designs (e.g., "hold for 3 seconds", math question, "swipe with two fingers"). Apple's Kids Category guidelines (which require parental gates for links and purchases) were not retrieved in this round.
- Bini, Montessori Preschool and Gus on the Go were not covered.

## 8. Journey/map design, unlocking, and celebration moments

### Takeaway
A visible linear path on the home screen ("where I am, what's next") was Duolingo's single most successful mechanic. Celebration animations measurably raise retention (+1.7% D7 from a streak milestone animation). Kid apps either use a guided path (Duolingo ABC, Khan Kids) or deliberately avoid a map and goals (Pok Pok, Toca, Endless Alphabet).

### Cited Findings
- Mazal: the home-screen path showing advancement through the curriculum is "probably the most successful mechanic in Duolingo". — [Reach Capital](https://www.reachcapital.com/?p=5907)
- The homescreen redesign "created a new growth hill and a path for personalization". — [Amplitude AMA](https://amplitude.com/community/events/ama-jorge-mazal-duolingo)
- A streak milestone celebration animation improved D7 retention of new learners by 1.7%. — [Duolingo blog](https://blog.duolingo.com/streak-milestone-design-animation)
- Duolingo ABC uses a fixed path of 127 units, and kids cannot choose their entry point. — [CSM](https://www.commonsensemedia.org/app-reviews/duolingo-abc-learn-to-read)
- Endless Alphabet has no goals beyond exploration, and there is no visible mastery. — [CSM](https://www.commonsensemedia.org/app-reviews/endless-alphabet)
- Math Garden: coins buy decorative trophies in the menu. — [CORE](https://core.ac.uk/download/161527697.pdf)

### Inferences
- Path plus side-quests: a single linear "journey" per child (adapted to their skill) with optional exploration areas. For the 2.5-year-old the "path" can be a short sequence of 3 activities followed by a reward, not a long map.
- Celebrations should be short (2-3 s), skippable, character-driven (the mascot dances or cheers, with a sound sting), and scaled to the achievement (small sparkle for a correct answer, big party for completing a unit). Avoid reward inflation.

### Gaps
- No sourced specifics for Khan Kids, Reading Eggs or Lingokids map design or celebration timings.

## 9. Monetization, parent complaints and anti-patterns (ads, paywalls, cancellation, too easy/hard)

### Takeaway
The biggest documented parent grievances are commercial, not pedagogical. The worst offenders: kid-facing upsells and pay-to-win (Prodigy), billing that is hard to cancel (ABCmouse; $10M FTC settlement), and paywalls that feel like "subscription issues" (Lingokids). Free, ad-free apps (Khan Kids, Duolingo ABC) earn trust. Charging for practice itself (Duolingo energy) produced adult backlash.

### Cited Findings
- ABCmouse paid $10M to settle FTC allegations that, from 2015 to 2018, it billed tens of thousands of customers and made cancellation too difficult. Some who cancelled had only cancelled the base membership. The order requires clearer disclosure of cost, renewal and cancellation terms. — [TMJ4 / FTC settlement](https://www.tmj4.com/news/i-team/abcmouse-pays-10-million-to-ftc-over-illegal-marketing-billing-allegations); [FTC consumer page](https://consumer.ftc.gov/comment/90974); [AOL](https://www.aol.com/abcmouse-trapping-working-parents-not-202249870.html)
- ABCmouse's CSM parent reviews (247) feature billing issues, fee warnings and poor customer service. — [CSM – ABCmouse parent reviews](https://www.commonsensemedia.org/website-reviews/abcmousecom/user-reviews/adult)
- A review-analysis site says the main 1-3-star complaint about ABCmouse is that cancellation doesn't take effect. — [unstar.app](https://unstar.app/blog/abcmouse-khan-academy-kids-duolingo-abc-homer-outschool-learning-apps-ranked-2026) (methodology unverified)
- Lingokids' CSM parent-review tags (only 12 reviews): "strong foundational skills", "limited critical thinking", "subscription issues", "colorful and engaging". — [CSM – Lingokids parent reviews](https://www.commonsensemedia.org/app-reviews/lingokids-play-and-learn/user-reviews/adult); [CSM – Lingokids](https://www.commonsensemedia.org/app-reviews/lingokids-play-and-learn)
- Prodigy: 16 membership ads vs 4 math problems in 19 minutes, and member-only battle advantages. — [Fairplay](https://fairplayforkids.org/feb-19-2021-advocates-to-ftc-prodigy-math-game-preys-on-kids-and-families/)
- TinyTap's model: creators mark games as premium (for subscribers) and are paid according to usage. — [eLearning Inside](https://news.elearninginside.com/tinytap-allows-teachers-to-make-their-own-educational-games)

### Inferences
- Never show offers, ads or currency purchases to the child. Put all payment in the parent area, with one-tap cancellation and clear renewal reminders. This is both an ethical and a trust differentiator.
- No paid advantage in any competitive element, and no locking of the "core loop" (e.g., pet evolution) behind payment.

### Gaps
- No sourced data on "too easy / too hard" complaints for specific apps, on "paywall mid-play" anecdotes for Lingokids, Sago or Toca Life World, or on ad-supported kids' apps.

## 10. Hebrew / Israeli kids' learning apps (TinyTap and Hebrew reading apps)

### Takeaway
TinyTap (Tel Aviv, 2012) is the notable Israeli platform: a marketplace of user- and teacher-created quiz/tap games (180K+ games, 24 languages) monetized by subscription with revenue share to creators. Hebrew-native reading apps are small indie products, often with nikud-based word reading. The market has room for a polished, adaptive Hebrew-first product.

### Cited Findings
- TinyTap was founded in 2012 by Yogev Shelley and Oren Elbaz. Teachers and parents build interactive games without code, from photos and voice recordings. Over 150K games in 24 languages by 2018, and later over 180K. — [eLearning Inside](https://news.elearninginside.com/tinytap-allows-teachers-to-make-their-own-educational-games); [EdSurge 2012](https://www.edsurge.com/news/2012-11-17-tinytap-nabs-500k-seed-round-for-kids-diy-gaming-app); [PR Newswire 2018](https://prnewswire.com/news-releases/tinytap-raises-5m-to-enable-limitless-learning-for-children-while-helping-teachers-get-paid-300668551.html)
- Funding: $500K seed (2012), $1.5M (2017), $5M led by Aleph (2018), and a reported $8.5M from Sequoia China (2023). — [Globes](https://en.globes.co.il/en/article-israeli-educational-games-co-tinytap-raises-5m-1001242223); [EdSurge 2017](https://edsurge.com/news/2017-02-14-tinytap-raises-1-5m-to-expand-marketplace-of-teacher-created-apps); [Startupim](https://startupim.com/company/tinytap/raw)
- TinyTap has promoted an approach to limit children's screen time. — [Jerusalem Post via PressReader, 2020](https://www.pressreader.com/israel/jerusalem-post/20200724/281951725137264)
- Kriakala – קריאה קלה עם הילה: a Hebrew reading app (ages 3-8) with games, riddles and interactive activities for learning the alef-bet. $9.99, with IAP, and the developer says it collects no data. — [App Store – Kriakala](https://apps.apple.com/us/app/id6752532036)
- שובי – הכנה לכיתה א: 7 Hebrew reading/writing games (letter recognition, letter sounds, word reading, sentence writing). Designed by a psychologist, with difficulty tuned to be "challenging but not frustrating". Free with IAP, 4.5★ from 10 ratings. — [App Store](https://apps.apple.com/us/app/id1135190119)
- Baby Hebrew: 130+ illustrated words with nikud for toddlers to first grade. Free with IAP. — [App Store – Baby Hebrew](https://apps.apple.com/il/app/baby-hebrew/id575495231)

### Inferences
- Hebrew specifics matter: right-to-left layout, nikud as a scaffold that is faded with age (2.5 → audio only, 5.5 → full nikud, 8.5 → texts without nikud), and Israeli context (Shabbat and holidays, which suit a weekly rather than daily streak, and local cultural content).
- A TinyTap-style "parent creates a mini-game from family photos and voice" feature could be a strong personalization hook (e.g., a parent records "who is this? it's Grandma!").

### Gaps
- No verification of whether TinyTap has substantial Hebrew content in 2026, or of its post-2023 ownership. Other Israeli products (e.g., Hebrew versions of Lingokids or Khan Kids, Israeli Ministry of Education apps, "Galileo", "Kidum" kids content) were not researched.

## 11. Cross-cutting synthesis: best patterns and anti-patterns for a personalized game for ages 2-10

### Takeaway
The evidence points to a hybrid design. For toddlers: a no-fail, text-free, audio-first sandbox (Pok Pok/Toca). For 5-6: a guided mascot path with animated letters, words and numbers (Duolingo ABC/Khan Kids/Endless). For 8-10: an adaptive challenge loop with light RPG and meta-progression (Prodigy loop minus monetization; Math Garden's Elo at about 75% success). Wrap it in forgiving habit mechanics (Duolingo streak freeze/amulet), keep all commerce in the parent area, and treat celebration and visible progress as first-class features.

### Cited Findings
- A visible progression path is Duolingo's most successful mechanic. — [Reach Capital](https://www.reachcapital.com/?p=5907)
- Forgiveness mechanics raise retention: Streak Wager +14% D7, Weekend Amulet +4% one-week return and -5% streak loss. — [Duolingo blog](https://blog.duolingo.com/how-streaks-keep-duolingo-learners-committed-to-their-language-goals/)
- Celebration animations move retention (+1.7% D7). — [Duolingo blog](https://blog.duolingo.com/streak-milestone-design-animation)
- Competition (leagues) boosted learning time about 17%, mostly among very active users. — [Duolingo blog](https://blog.duolingo.com/time-spent-learning-well)
- Adaptive difficulty at about 75% success, with feedback after each item. — [UvA](https://dare.uva.nl/id/de65bb65-d198-4dbc-ac7b-bc25ca23e627); [PMC](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10299571/)
- No text, menus or rules lets young kids play alone (an ADA-winning approach). — [Pok Pok press kit](https://playpokpok.com/press/press-kit/)
- Engagement does not guarantee transfer (DragonBox). — [EdSurge](https://edsurge.com/news/2016-03-13-enter-the-dragonbox-can-a-game-really-teach-third-graders-algebra)
- Commercial anti-patterns draw regulatory and reputational damage: Prodigy (FTC complaint) and ABCmouse ($10M FTC). — [Fairplay](https://fairplayforkids.org/feb-19-2021-advocates-to-ftc-prodigy-math-game-preys-on-kids-and-families/); [TMJ4](https://www.tmj4.com/news/i-team/abcmouse-pays-10-million-to-ftc-over-illegal-marketing-billing-allegations)
- Guilt-based notifications aimed at children are criticized as manipulative. — [JTBD UX](https://jtbdux-protocol.pages.dev/adoption-and-behavior/behavior-design-patterns/how-duolingo-engineered-50-million-daily-users-and-the-backlash-that-came-with-it)

### Inferences
**Best patterns (recommended)**
1. One mascot cast with an age-adaptive voice. It narrates every prompt for pre-readers and fades narration as reading grows.
2. Per-child profiles with picture or avatar selection (no typing), so three siblings share one device.
3. A dual-rating adaptive engine (Elo-like) targeting about 75-80% success, with response time as a signal. Placement is invisible, done by playing, not by a test.
4. Short sessions: 3-5 activities followed by a celebration and a natural stopping point. A "come back tomorrow, the character is sleeping" ending replaces energy or hearts.
5. Forgiving streaks: a weekly goal (e.g., 4 of 7 days), automatic freeze on Shabbat and holidays, and an optional family streak shared by the siblings.
6. Celebrations that are short, character-driven, scaled to the achievement, with distinct sound stings. The big parties are saved for milestones.
7. Collectibles and cosmetics (avatar outfits, room decorations, pets that grow with practice) earned only through learning, never purchased.
8. A no-fail sandbox mode for the toddler, and sandbox items unlocked through the older kids' journeys.
9. A parent dashboard showing mastery per skill (a gap Endless Alphabet leaves open), plus parent-created mini-games from family photos and voice (TinyTap-inspired).
10. Bridge activities that move in-game representations into school notation, to secure transfer (the DragonBox lesson).

**Anti-patterns (avoid)**
1. Losing lives or hearts on mistakes, or energy that limits practice.
2. Any purchase, ad or "members only" item visible to the child. Pay-to-win.
3. Hard-to-cancel billing. Upsells placed in the parental-gate path.
4. Guilt-based or nagging notifications aimed at children (address the parent instead).
5. Public leaderboards for young children. Competition only opt-in, for ages 8+, preferably co-op or family.
6. A fixed starting point that ignores the child's level (a CSM criticism of Duolingo ABC).
7. Speech recognition as a gate to progress for young children.
8. A meta-game that crowds out learning time (Prodigy's 4 problems in 19 minutes).

### Gaps
- No direct child-development research was gathered in this round on the optimal session length or reward schedules for ages 2-4 (e.g., AAP screen-time guidance).
- No sources found for Apple Design Award insights other than Pok Pok's, GDC talks by Duolingo designers, or Khan Kids design-blog posts (all likely relevant but unretrieved due to blocked fetches).
