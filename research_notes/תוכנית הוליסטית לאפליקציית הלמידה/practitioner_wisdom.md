# Practitioner Wisdom: Lessons from People Who Built Kids' Learning Apps and Games

Scope: hands-on lessons, processes and rules of thumb from kids' content studios and edtech builders, turned into actionable lessons for a UX-designer parent building a Hebrew-first, gamified iPhone/iPad learning game for three kids: a girl, 8.5, in grade 3; a boy, 5.5, in gan chova; and a toddler, 2.5.

Method note: this session's network policy blocked page fetches (WebFetch) for most hosts: ycombinator.com, joanganzcooneycenter.org, qrf.org, sketch.com and others. All findings come from web-search result summaries of the cited pages. They are not full-text reads, so treat quotes as near-verbatim snippets. Where a claim comes from secondary or weak sources, the note says so. The report writer should note that full-text verification was not possible.

Correction to the brief: Pok Pok's co-founders are **Esther Huybreghts** and **Melissa Cash** (CEO; formerly at Disney). Esther's husband **Mathijs Demaeght** is also named as a co-founder. "Melinda Byerley" is not associated with Pok Pok.

---

## 1. How do the best kids' content studios work? (Sesame, PBS KIDS, Khan Academy Kids, Duolingo ABC, Toca Boca, Sago Mini, Originator, Pok Pok, DragonBox, TinyTap)

### Takeaway
The studios that succeed share one loop: (1) set learning goals and kid-perspective goals in writing before production, (2) observe real children early, before anything digital exists, and again at every stage, and (3) put researchers or "play designers" next to producers rather than in a separate department. They also make a few early philosophical decisions (no points, no levels, open-ended play, child at the center) that steer every later design choice.

### Cited Findings

**Sesame Workshop (the original "model")**
- The Sesame Workshop model is the planning, production and evaluation system that came out of the first season. Research was split in two: in-house **formative** research that informed and improved production while it was under way, and independent **summative** evaluation done by the Educational Testing Service (ETS). — [Wikipedia: Sesame Street research](https://en.wikipedia.org/wiki/Sesame_Street_research)
- Ed Palmer built "the distractor" in the 1960s. A child watched a test segment while a slideshow ran beside the screen, and researchers recorded whether the child's eyes stayed on the segment, as a measure of its attention-holding power. Researchers Lewis Bernstein and Valeria Lovelace later turned it into an "eyes-on-screen" method that collected data from larger groups of children at once. — [Wikipedia: Sesame Street research](https://en.wikipedia.org/wiki/Sesame_Street_research); [Stefy substack: Sesame Street and the Distractor](https://stefy.substack.com/p/sesame-street-and-the-distractor)
- The method measured only whether the child was watching the screen, not where on the screen the child looked. — [Stefy substack](https://stefy.substack.com/p/sesame-street-and-the-distractor)
- A secondary blog reports that segments kept attention ratings of 85–90%, with some at 100%. This is unverified against primary archives. — [Stefy substack](https://stefy.substack.com/p/sesame-street-and-the-distractor)
- In 1968 Gerald Lesser (Harvard) ran curriculum-planning seminars that brought psychologists, sociologists, teachers, TV producers, children's book authors and entertainment executives together. Accounts differ on the format: "five three-day seminars" in one, "four two-and-a-half-day seminars" in another. — [Wikipedia: Educational goals of Sesame Street](https://en.wikipedia.org/wiki/Educational_goals_of_Sesame_Street); [Harvard Gazette 2019](https://news.harvard.edu/gazette/story/2019/09/harvards-hand-in-helping-to-launch-sesame-street-which-celebrates-50-years)
- Lesser understood that getting child-development research into production required a close working relationship between researchers and producers. Researchers wrote a **"Writer's Notebook"** for writers and producers that bridged curriculum goals and scripts. Research staff and producers kept running internal reviews and seminars to check that curriculum goals were being met. — [Wikipedia: Educational goals of Sesame Street](https://en.wikipedia.org/wiki/Educational_goals_of_Sesame_Street)
- A 2024 academic paper on Sesame's formative research says the real experts are the children who take part in it. — [Journal of Children and Media, 2024 (via informahealthcare/T&F)](https://informahealthcare.com/doi/full/10.1080/17482798.2024.2356957?src=exp-la)

**PBS KIDS / Ready To Learn**
- For about two decades under the Ready To Learn grant, Rockman et al. ran formative evaluations for PBS KIDS. These included youth and family playtesting of digital games at different design stages, plus extended-play studies. — [Rockman et al.](https://rockman.com/topic/childrens-youth-media/)
- PBS KIDS uses playtesting to find out whether mechanics are too easy or too hard, whether instructions and prompts are adequate, and how well the overall concept works (DeWitt & Simensky 2019). Facilitators logged the moments when children said the inquiry process out loud. — [Zero to Three: The PBS KIDS Approach](https://www.zerotothree.org/resource/journal/getting-the-most-out-of-screen-time-the-pbs-kids-approach-to-learning-through-media/)
- In SRI's 2012 study, children often left the PBS KIDS Lab site to play games with characters they already liked. Known characters pull kids in. — [SRI PBS KIDS Transmedia Gaming Study 2012](https://www.sri.com/wp-content/uploads/2021/12/gaming_body_feb_13_0.pdf)
- With CRESST, PBS KIDS built a Learning Analytics Platform that processed telemetry from 20,000+ gameplay sessions. Gameplay data correlated with the TEMA-3 math assessment, and the analysis showed which game elements were most effective for learning. — [CRESST](https://cresst.org/portfolio/how-cresst-helped-pbs-kids-improve-childrens-learning-through-gameplay/)
- Wikipedia says the Ready To Learn grant ran from 1992 until it was terminated in 2025. This is not verified against an official source. — [Wikipedia: Ready to Learn](https://en.wikipedia.org/wiki/Ready_to_Learn_(grant_program))

**Khan Academy Kids (Duck Duck Moose heritage)**
- Duck Duck Moose's nine-person team joined Khan Academy in 2016. Their earlier apps had won 22 Parents' Choice Awards, 19 Children's Technology Review Awards and a KAPi award. — [Duck Duck Moose announcement](https://www.duckduckmoose.com/2016/08/leading-early-learning-app-maker-duck-duck-moose-joins-khan-academy/); [TechCrunch 2016](https://techcrunch.com/2016/08/26/kids-app-maker-duck-duck-moose-joins-khan-academy/)
- The team play-tested in a wide range of child-care centers to see whether a single design element (a background, a sound, a graphic) helped or hurt a child's performance on an assessment task. One concrete finding: uneven backgrounds kept kids from making accurate height comparisons. Audio was treated as a design problem of its own, and answer-choice designs were tested with kids on an ongoing basis. — [Khan Academy Blog: Prototyping Playful and Nimble Pre-K Assessments](https://blog.khanacademy.org/prototyping-playful-and-nimble-pre-k-assessments/)
- The team worked with Stanford researchers, aligned content to the Head Start Early Learning Outcomes Framework, and spent several years prototyping, play-testing and refining before the 2018 launch. — [Khan Academy Blog](https://blog.khanacademy.org/khan-academy-adds-apps-for-young-children/) (via search summary)

**Duolingo ABC**
- The app relies heavily on audio narration and animation. Engineers describe animations as tricky because they can run asynchronously. A modular structure lets one feature, such as the reading lessons, be tested on its own. Android "can be a fickle platform… especially for kids devices." — [Duolingo Blog: A good read: building Duolingo ABC for Android](https://blog.duolingo.com/a-good-read-building-duolingo-abc-for-android/)
- The structure is a map of a city where each building holds a set of lessons, with separate flows for parents and children. — [Screensdesign UI breakdown](https://screensdesign.com/showcase/learn-to-read-duolingo-abc)
- Critique from a reading-focused reviewer: many exercises match whole words to spoken words, and the app lacks sentence and book reading. — [Phonics.org review](https://www.phonics.org/learn-to-read-duoling-abc-review/)

**Toca Boca**
- Testing starts before anything digital exists. The team first watches how kids play within a play pattern. Bjorn Jeffery described having kids play with a physical doll's house to find where the fun is, then building digital prototypes around it. — [GamesBeat: How Toca Boca got 100M downloads](https://gamesbeat.com/how-toca-boca-got-100m-mobile-downloads-by-putting-kids-first/); [Joan Ganz Cooney Center App Fairy podcast transcript](https://joanganzcooneycenter.org/?p=18640)
- Caroline Ingeborn (2018): kids come into the Stockholm and San Francisco studios, and the team also visits schools and homes. Once they think they understand a play pattern, they prototype and go back to kids repeatedly during production. — [JGCC App Fairy transcript](https://joanganzcooneycenter.org/?p=18640)
- Jeffery: if kids don't like something, they simply leave and play with something else, which makes the process very humbling. — [GamesBeat](https://gamesbeat.com/how-toca-boca-got-100m-mobile-downloads-by-putting-kids-first/)
- Toca employs **play designers**, not game designers, and their job is to keep each project anchored in the kids' perspective. Play researcher Chris Lindgren asks kids during playtests whether they see a character who is like them, often with surprising results. — [QRF interview with Chris Lindgren](https://qrf.org/en/media-center/our-latest-blogs/interview-chris-lindgren-lessons-learnt-designing-games-children)
- The Kill Screen headline "The secret of smart kids' entertainment: give them a toy, not a game" sums up Toca's toy-first philosophy. — [Kill Screen](https://www.killscreen.com/secret-smart-kids-entertainment-give-them-toy-not-game/)

**Sago Mini (Spin Master; Originator's apps moved here)**
- Weekly playtesting with kids. Director of Product Brennan Clark says the key, beyond weekly testing, is "to bring the parents into the equation". Sago built a parent app that shows children's creations, so parents keep seeing the value. — [RevenueCat Sub Club podcast with Brennan Clark](https://www.revenuecat.com/blog/growth/how-to-build-a-great-kids-app-with-minimal-data-podcast-with-sago-minis-brennan-clark)
- "Most of the purchases are made in those first 48 hours". The first session has to win the parent. Apple drives 80% of revenue for Sago's three preschool subscription apps. Kids' privacy rules rule out IDFA/ATT, so analytics needed a custom SDK built with AppsFlyer. — [RevenueCat podcast](https://www.revenuecat.com/blog/growth/how-to-build-a-great-kids-app-with-minimal-data-podcast-with-sago-minis-brennan-clark); [RevenueCat: monetizing kids apps](https://www.revenuecat.com/blog/growth/whats-the-best-way-to-monetize-kids-apps)
- Sago's CEO says its products have no artificial incentives, formal rules or scripted characters, "because we want kids to have ownership of the ways they play." — [Sago Mini: Message from the CEO](https://sagomini.com/article/message-from-the-ceo/)

**Originator (Endless Alphabet / Reader / Numbers)**
- Spin Master reportedly bought Originator in July 2021 for $29M. The same source says the studio shut down in July 2026 after 13 years, with layoffs, and that its apps moved to Sago Mini/PikNik. This comes from a user-edited fandom wiki and is **weakly sourced**. — [Endless fandom wiki: Originator](https://endless-20.fandom.com/wiki/Originator)

**Pok Pok (Esther Huybreghts, Melissa Cash, Mathijs Demaeght)**
- "There is no winning in the app, so there's no sort of dopamine that comes with the winning or beating a level that keeps making you want to play" (Melissa Cash). The app has no levels, no points and nothing to win or lose, and the team wanted a space where mistakes don't cost anything. — [Globe and Mail](https://www.theglobeandmail.com/business/article-pok-pok-app-provides-a-non-addictive-virtual-experience-for-young/); [Mom's Choice Awards interview](https://momschoiceawards.com/blog/interview-with-moms-choice-award-winner-pok-pok-montessori-preschool-app/)
- Those early decisions steer the product. The team rarely has to remove overstimulating features, because avoiding levels pushes them toward calm play, which "sometimes needs to be *more* engaging." — [Parent Tech: Behind the Scenes with Pok Pok](https://www.parent.tech/p/behind-the-scenes-redefining-kids-screen-time-with-pok-pok)
- The UI is mostly language-free because many users can't read. It supports multi-touch for two-handed play, the art is hand-crafted, and there are no ads. The team iterated with family, friends and schools, and Cash calls young testers "brutally honest." Pok Pok won the 2021 Apple Design Award three weeks after launch. — [Sketch blog: Pok Pok](https://sketch.com/blog/pok-pok); [Hampton blog](https://joinhampton.com/blog/from-disney-designer-to-app-store-royalty); [Parenting, Apparently podcast](https://podcasts.apple.com/al/podcast/parenting-apparently/id1855989506)
- Each toy is designed to "grow with kids" from ages 2–6. Another source gives 2–7. — [Sketch blog](https://sketch.com/blog/pok-pok); [Mom's Choice](https://momschoiceawards.com/blog/interview-with-moms-choice-award-winner-pok-pok-montessori-preschool-app/)

**DragonBox (WeWantToKnow; Jean-Baptiste Huynh)**
- Huynh, a former teacher, co-founded DragonBox with cognitive scientist Patrick Marchal. In his words: "We have a Nordic philosophy that puts the child at the center and focuses on exploration, play, discovery and building intrinsic motivation." Kahoot! bought DragonBox in 2019 for about $18M. At that point its six apps had about 1.5M downloads. — [EdSurge 2019](https://www.edsurge.com/news/2019-05-08-game-on-kahoot-snaps-up-dragonbox-for-18-million-for-its-first-acquisition); [TechCrunch 2019](https://techcrunch.com/2019/05/08/educational-gaming-platform-kahoot-acquires-math-app-maker-dragonbox-for-18m/)
- DragonBox strips the formal language out of algebra and reduces it to its basic elements. Rules first appear as "monster cards" and dice, and the monster cards are slowly replaced by letters and numbers. This is concrete-to-abstract fading. — [Getting Smart: DragonBox: This Is How You Gamify](https://www.gettingsmart.com/2013/05/24/dragonbox-this-is-how-you-gamify/)
- "It's not an algebra app." Huynh frames the product around speed and imagination. — [Hechinger Report](https://hechingerreport.org/video-games-that-work-better-than-school/)

**TinyTap (Israeli; Yogev Shelly & Oren Elbaz, Bezalel graduates)**
- Founded in 2012 in Tel Aviv. Shelly built it "out of my own personal needs". One account says he started by turning family photos into activities for his father, who had dementia. Teachers build games and keep a revenue split, reported early as 70/30 in the creator's favor. The company raised $5M in 2018, led by Aleph. Aggregators say it is now an Animoca Brands subsidiary (unverified). — [Globes 2018](https://en.globes.co.il/en/article-israeli-educational-games-co-tinytap-raises-5m-1001242223); [PR Newswire 2018](https://prnewswire.com/news-releases/tinytap-raises-5m-to-enable-limitless-learning-for-children-while-helping-teachers-get-paid-300668551.html); [Jerusalem Post via PressReader](https://www.pressreader.com/israel/jerusalem-post/20200724/281951725137264)

### Inferences
- **Process to copy at home: a mini Sesame model.** (a) Write a one-page "Writer's Notebook" per age: the learning goals for Hebrew letters, niqqud, phonics and early math, and what each child already finds funny or loves. (b) Before any code, run a non-digital play session (paper cards, magnetic letters, a doll's house) to find "where the fun is," as Toca does. (c) Prototype, then playtest weekly, as Sago does. (d) Periodically run a separate "did they learn?" check (summative), apart from "do they like it?" (formative).
- **Run a home version of the distractor.** Play the new mini-game while a competing attraction (a toy, a sibling's show) is available nearby, and note how often and when the child looks away. That is a cheap, honest appeal test, and it beats asking "did you like it?"
- **Make the philosophy decision early.** Pok Pok shows that "no points, no levels" shapes everything that follows. The brief says "highly gamified," so decide on purpose which gamification layers you accept and write it down: for example, no loss states, no timers for the 5.5-year-old, and progress shown as collection or growth rather than scores.
- **Use familiar characters.** SRI's finding that kids go after characters they already know suggests building the game around characters the kids already love, such as family pets, the kids themselves as avatars, or locally beloved figures where licensing allows.
- **TinyTap is a local precedent** for Hebrew-language educational content made by parents and teachers. It is worth studying as competition and as inspiration for a "parent-authored levels" feature, for example family photos turned into activities.

### Gaps
- No full-text practitioner accounts were retrieved for Lingokids, Homer, Prodigy's early learnings, Kahoot! kids, Osmo, or Nintendo Brain Age / Ring Fit. These were not searched within the tool budget.
- The precise Sesame "Writer's Notebook" format and current Sesame Workshop formative methods (2020s) were not obtained.
- Originator's 2026 closure is unconfirmed by any primary source.
- Toca Boca session sizes and protocols (how many kids per test, how long) were not found.

---

## 2. Edtech postmortems and failures: why kids' edtech startups fail and what designers underestimate

### Takeaway
Practitioners agree on a few failure modes. The parent (or teacher), not the child, is the buyer. Downloads are not the success metric. Content and quality cost far more than founders expect, and buyers have historically cared more about cost than quality. The App Store kids' market trained parents to expect free or $0.99 apps, which pushes studios toward bundles and subscriptions. For a personal family project, the economics problem disappears, but the lessons about content volume and parents as gatekeepers still apply.

### Cited Findings
- A founder whose first edtech startup failed said: "even though the end consumer is the student, decisions are always driven by the influencer whether it is the teacher or the parent." — [Entrepreneur India](https://www.entrepreneur.com/article/276350)
- Download counts are not the right key metric for an edtech startup. With kids' apps, the parents decide whether to buy. — [OnGraph: 9 reasons edtech startups fail](https://ongraph.com/9-reasons-why-edtech-startups-fail-and-tips-to-avoid-them) (vendor blog; low authority)
- Feature bloat is common: "fancy features that don't align with the app's core purpose. This drives up development costs and confuses users." — [OnGraph](https://ongraph.com/9-reasons-why-edtech-startups-fail-and-tips-to-avoid-them) (vendor blog)
- The YC podcast "Cost vs Quality in Edtech" (Keith Schacht of Mystery Science, Avichal Garg, Geoff Ralston, 2018) argues that mainstream education buyers have cared far more about cost than quality, while founders tend to prioritize quality. — [YC Blog](https://www.ycombinator.com/blog/cost-vs-quality-in-edtech-keith-schacht-avichal-garg-and-geoff-ralston); [YouTube](https://www.youtube.com/watch?v=ajO-h8NEcqQ). **Correction:** the brief links Schacht to MindSnacks, but the search ties him to Mystery Science (K-8 inquiry-based science, acquired). No MindSnacks postmortem was found.
- The Ekid Studio AR flashcards app won awards but was listed as out of business by March 2025. It competed with ABCmouse's "hundreds of lessons," which shows that the cost of content volume decides who survives. — [Geeks Around Globe](https://next.geeksaroundglobe.com/ekid-studio-ar-flashcards-failure-edtech) (secondary)
- Kids' app economics: parents grew used to free or $0.99 apps, so charging $4.99 became hard even for good games. A 2026 audit of 34 toddler apps found 19 selling subscriptions, at an average of $60.40 a year. — [Little Wheels: toddler app costs](https://littlewheels.app/statistics/toddler-app-costs) (small self-published site; indicative only)
- Spin Master launched PikNik in 2023, a subscription bundling its preschool apps that replaced single-app purchases. One consumer blog dates the shift to 2020 and says one-time purchases "couldn't fund new content fast enough". The two dates conflict. — [Wikipedia: Sago Mini](https://en.wikipedia.org/wiki/Sago_Mini); [Thinking in Educating blog](https://www.thinkingineducating.com/the-hidden-reasons-behind-subscription-models-in-kids-mobile-games-and-what-parents-can-do/)
- Privacy and analytics: kids' apps cannot use IDFA/ATT and are losing Google's Ad ID, so measurement must be done in creative ways. Sago, for example, built a custom SDK with AppsFlyer. — [RevenueCat podcast](https://www.revenuecat.com/blog/growth/how-to-build-a-great-kids-app-with-minimal-data-podcast-with-sago-minis-brennan-clark)
- Michael Feldstein's "When ed tech startups fail" essay exists as a reference on the topic. It was not retrieved in detail. — [e-Literate](https://eliterate.us/when-ed-tech-startups-fail/)

### Inferences
- **Content volume is the hidden boss.** Studios needed "hundreds of lessons," and Sago reportedly moved to subscriptions because single purchases couldn't pay for new content fast enough. For a solo parent, that means the engine and templates matter more than any single level. Design a few reusable mechanics (for example drag-letter-to-sound, build-a-word, count-and-feed) that can be filled with content from data files, rather than many bespoke mini-games.
- **Audio is a production pipeline, not an asset.** Duolingo ABC's "heavy reliance on audio narration" and Khan Kids treating audio as its own design problem imply that every prompt, letter sound, niqqud combination and piece of praise needs recorded Hebrew audio. Budget for it, script it in a spreadsheet, and record it in batches. A parent's own voice is a strong option for home use.
- **The parent is still a user even at home.** Sago's parent app and Duolingo ABC's separate parent flow show that the adult needs a view of progress. For a family project, the parent is the curriculum specialist, analyst and gatekeeper all at once. Build a simple parent dashboard (what each child practiced, where they got stuck) early.
- **Localization trap.** Hebrew-first avoids the retrofit problem most studios face, but RTL layouts, niqqud rendering and gendered Hebrew forms of address (for a girl vs. a boy) multiply the audio and text variants. Plan the data model for gendered strings from day one. This inference comes from Hebrew grammar, not from a cited practitioner.

### Gaps
- No primary postmortems were retrieved for MindSnacks, AltSchool, Amplify or Originator. The a16z, Reach Capital, Owl Ventures and Brighteye essays were not searched within the budget.
- No reliable churn or retention benchmarks for kids' subscription apps were found.

---

## 3. Learning engineering as a discipline (Learning Engineering Toolkit, design-based research, learning objectives first, curriculum specialist vs. designer)

### Takeaway
Learning engineering means applying the learning sciences through human-centered engineering design and data-informed iteration. Its seven principles are process, learning sciences, human-centered design, engineering, data instrumentation, analytics and ethics. The best-supported practical rule for educational games is **intrinsic integration**: build the learning content into the core mechanic, so that mastering the content is how you win. Bolting a game onto a quiz ("chocolate-covered broccoli") does not work.

### Cited Findings
- *Learning Engineering Toolkit* (Goodell & Kolodner, eds., Routledge 2022) defines learning engineering as a process and practice that applies the learning sciences using human-centered engineering design and data-informed decision-making. The foundations are organized around process, applying the learning sciences, human-centered design, engineering, data instrumentation, analytics and ethics. The tools section covers understanding the challenge, teaming, lean-agile development and human-centered design. — [ResearchGate listing](https://www.researchgate.net/publication/361424680_Learning_Engineering_Toolkit_Evidence-Based_Practices_from_the_Learning_Sciences_Instructional_Design_and_Beyond); [O'Reilly](https://www.oreilly.com/library/view/learning-engineering-toolkit/9781000683257/)
- Learning engineering is described as constant iteration and co-production with learners, with data informing continuous improvement. — [Fensie et al., "Conceptualizing Learning Engineering"](https://digitalcommons.pepperdine.edu/gsepedu/389/)
- Habgood's PhD (Nottingham, 2007) traces the "chocolate-covered broccoli" label for early edutainment to Bruckman (1999) and argues these methods "have often proved ineffective" because they rely on extrinsic motivation. — [Habgood thesis PDF](https://eprints.nottingham.ac.uk/10385/1/Habgood_2007_Final.pdf)
- Habgood's test game *Zombie Division* (division taught through sword-fighting) compared an **intrinsic** version, where math mastery produced in-game success, with an **extrinsic** version that put a quiz after play. The intrinsic version was more effective both for learning and for motivation. A secondary write-up says that when pupils could choose, they played the intrinsic version **7x longer**. — [Habgood thesis](https://eprints.nottingham.ac.uk/10385); [KQED MindShift: secret sauce of a great educational game](https://www.kqed.org/mindshift/20765/whats-the-secret-sauce-to-a-great-educational-game)
- PBS KIDS + CRESST used gameplay telemetry (20,000+ sessions) to check that in-game behavior correlates with an external math assessment (TEMA-3), which validates that the game measures learning. — [CRESST](https://cresst.org/portfolio/how-cresst-helped-pbs-kids-improve-childrens-learning-through-gameplay/)
- Khan Academy Kids aligned content to a published framework (the Head Start Early Learning Outcomes Framework) and partnered with Stanford researchers. That is the "learning objectives first" pattern with a curriculum partner. — [Khan Academy Blog](https://blog.khanacademy.org/khan-academy-adds-apps-for-young-children/)
- At Sesame, curriculum experts and producers worked side by side, with researchers translating goals into a Writer's Notebook. The curriculum specialist role was a partner to the creative team, not a gatekeeper at the end. — [Wikipedia: Educational goals of Sesame Street](https://en.wikipedia.org/wiki/Educational_goals_of_Sesame_Street)

### Inferences
- **Practical recipe for the parent:**
  1. List the learning objectives per child. Examples: grade 3 (girl), reading fluency without niqqud and multiplication tables; gan chova (boy), letter-sound correspondence, phonemic awareness, numbers to 20; toddler, colors, shapes, vocabulary, cause and effect. Align them to the Israeli Ministry of Education curriculum for gan chova and grade 3, which serves as the "Head Start framework" equivalent.
  2. For each objective, design the *mechanic* so that performing the skill is the action that moves the game forward, as in DragonBox and Zombie Division. A quiz between rewards is not enough.
  3. Instrument a few events (attempts, errors per item, time to answer, hint use) to see real mastery. Then compare against a short offline check now and then, such as reading 10 words aloud to a parent, as a home version of PBS KIDS' TEMA-3 validation.
  4. Use DragonBox-style fading: start with concrete, playful tokens and gradually swap in real symbols (letters, niqqud, digits).
- Design-based research at home means iterating in short cycles with the same few learners and changing one variable at a time where possible. Khan Kids tested single elements such as backgrounds to see their effect on task performance.

### Gaps
- The Toolkit's full chapter text (for example evidence-centered design and A/B testing guidance) was not accessible. Carnegie Learning's specific practices were not researched within the budget.

---

## 4. Practical methods for testing with kids (observation protocols, how many kids and what ages, reading boredom and frustration, co-design, think-aloud limits, ethics with your own kids)

### Takeaway
Keep sessions short and age-matched: about 30 minutes or less for children up to 8, and 45–60 minutes for older children. Give verbal instructions to pre-readers. Observe behavior rather than asking for opinions, because preschoolers cannot reliably report satisfaction. Children aged 6–8 do better tested in pairs. A child can always decline, and they will simply leave if bored, which is itself the clearest signal. For deeper involvement, use Druin's roles (user, tester, informant, design partner) and low-tech prototyping such as "Bags of Stuff."

### Cited Findings
- **Session length:** about 30 minutes maximum for children up to about 8 and 45 minutes for older children (People for Research). Other sources cap sessions at 60 minutes (Research Collective), and NN/G ran 60-minute sessions with ages 3–12. — [NN/G: Usability Testing with Minors: 16 Tips](https://www.nngroup.com/articles/usability-testing-minors/); [Research Collective](https://research-collective.com/conducting-usability-studies-with-children/)
- **Pairs for ages 6–8:** NN/G found paired testing more effective for 6–8-year-olds. Children over 8 were usually comfortable enough with devices not to need a partner. — [NN/G](https://www.nngroup.com/articles/usability-testing-minors/)
- Use simple language, give instructions verbally to pre-readers, and **prepare extra tasks** so a session doesn't end early. — [NN/G](https://www.nngroup.com/articles/usability-testing-minors/)
- Preschoolers (2–5) have short concentration spans and are too young to clearly express how satisfied they are. Children aged 6–10 can follow a task and hold attention longer. — [AufaitUX](https://www.aufaitux.com/blog/usability-testing-with-children/); [arXiv: Take Care: usability evaluation methods for children](https://arxiv.org/pdf/1212.0647)
- Research on think-aloud and collaboration with children in context suggests verbalization works differently with kids than with adults. — [arXiv 1306.4094: Exploring Verbalization and Collaboration during Usability Evaluation with Children](https://arxiv.org/pdf/1306.4094) (title-level only; details not retrieved)
- Let children decline even when a parent has consented. — [The Story: usability testing with minors](https://thestory.is/en/journal/usability-testing-with-minors/) / People for Research (via search summary)
- **Signals:** kids who don't like something simply leave and play with something else (Toca). PBS KIDS facilitators logged when children spontaneously said the target concept out loud, which is a positive learning signal. The distractor method used eyes-on-screen versus looking away as the attention measure. — [GamesBeat](https://gamesbeat.com/how-toca-boca-got-100m-mobile-downloads-by-putting-kids-first/); [Zero to Three](https://www.zerotothree.org/resource/journal/getting-the-most-out-of-screen-time-the-pbs-kids-approach-to-learning-through-media/); [Wikipedia: Sesame Street research](https://en.wikipedia.org/wiki/Sesame_Street_research)
- **Cooperative Inquiry (Allison Druin, 1998, UMD HCIL KidsTeam):** adapts participatory design, contextual inquiry and in-situ low-fidelity prototyping. It asks for a long-term partnership between adult and child team members and offers techniques that support intergenerational communication and produce actionable design feedback. — [UMD HCIL KidsTeam](https://hcil.umd.edu/children-as-design-partners/); [Druin 1999 CHI paper PDF](http://www.cs.umd.edu/hcil/trs/99-14/99-14.pdf)
- Druin's four roles for children are **user, tester, informant and design partner**. — [Cooperative Inquiry revisited](https://www.researchgate.net/publication/257745975_Cooperative_Inquiry_revisited_Reflections_of_the_past_and_guidelines_for_the_future_of_intergenerational_co-design); [JGCC: Designing with Kids (2023)](https://joanganzcooneycenter.org/2023/03/22/designing-with-kids/)
- **"Bags of Stuff"** is a low-tech prototyping technique in which kids build ideas from art supplies. It was used with KidsTeam members. "Sticky noting" as a named technique could not be confirmed in this session. — [ResearchGate: Cooperative Inquiry](https://www.researchgate.net/publication/221517509_Cooperative_Inquiry_Developing_new_Technologies_for_Children_with_Children)
- Toca asks kids whether they see a character "like them," and Lindgren says the answers are often surprising. That is a cheap representation check. — [QRF: Chris Lindgren interview](https://qrf.org/en/media-center/our-latest-blogs/interview-chris-lindgren-lessons-learnt-designing-games-children)
- Sago playtests **weekly**. — [RevenueCat podcast](https://www.revenuecat.com/blog/growth/how-to-build-a-great-kids-app-with-minimal-data-podcast-with-sago-minis-brennan-clark)

### Inferences
- **Home protocol based on the above:**
  - **Cadence:** a short session every week, as Sago does. Toddler: 5–10 minutes, observation only. 5.5-year-old: 10–15 minutes. 8.5-year-old: 15–20 minutes, plus a 2–3 minute chat afterward.
  - **Pair the 5.5 and 8.5** occasionally (NN/G's pairs for ages 6–8). The older sister explaining the game to her brother shows you the gaps in your onboarding.
  - **Observe, don't ask.** Log looks-away, sighs, random tapping, asking for help, quitting, and returning unprompted the next day (the strongest appeal signal). Count "unprompted returns" over a week rather than taking "I like it" at face value.
  - **Don't teach during the test.** Give one verbal prompt only (as for pre-readers), then stay quiet. Have extra tasks ready.
  - **Make the 8.5-year-old a design partner** (Druin's highest role). Give her a "Bags of Stuff" session with paper, stickers and markers to invent a level or character for her brother. Older kids at this age like to know the rules and do things right, which suits an "assistant designer" role.
  - **Ethics with your own kids:** the right to decline applies even more strongly at home. Keep testing optional, never tie it to screen-time privileges, and stop at the first signs of fatigue. Children want to please a parent, so you will get positive bias. Weight behavior over words even more than usual, and if possible include one or two friends or cousins who have no reason to please you.
  - **Sample size:** home testing gives you N=3 at three different developmental stages. That is fine for formative work (Toca and Khan Kids iterate with small groups) but not for claims about learning. Add cousins, classmates or gan friends occasionally to avoid overfitting to your own kids.

### Gaps
- No practitioner-specific checklist of boredom and frustration cues for ages 2–8 was found, such as a published coding scheme. The cues above are partly inferred.
- Hanna, Risden & Alexander's classic 1997 guidelines on usability testing with children were not surfaced by search.

---

## 5. Rules of thumb practitioners repeat

### Takeaway
The rules with real practitioner backing: find the fun first in physical play, then build the learning into the mechanic. Use voice and minimal text for pre-readers. Prefer feedback over "more fun" and keep rewards small. Never make mistakes costly. Design per two-year age band. Win the parent in the first sessions. Test with real kids weekly. Several popular rules, such as "audio is 50% of the experience" and "first 30 seconds," were not traced to a citable first-hand source.

### Cited Findings
- **Find the fun first, physically:** Toca uses doll's-house play before any prototype. — [GamesBeat](https://gamesbeat.com/how-toca-boca-got-100m-mobile-downloads-by-putting-kids-first/)
- **Integrate learning into the mechanic** (no chocolate-covered broccoli). Kids played the intrinsic version about 7x longer. — [Habgood thesis](https://eprints.nottingham.ac.uk/10385); [KQED MindShift](https://www.kqed.org/mindshift/20765/whats-the-secret-sauce-to-a-great-educational-game)
- **Design per two-year band:** Gelman's chapters cover ages 2–4, 4–6 and 6–8. "The app you're designing [for a 3-year-old] probably won't work for a 6-year-old." — [Rosenfeld Media: Design for Kids](https://rosenfeldmedia.com/books/design-for-kids/); [Vicky Teinaki summary](https://www.vickyteinaki.com/blog/born-digital-app-design-kids/)
- **Ages 2–4:** autoplay and minimal UI (secondary summary of Gelman). — [Vicky Teinaki](https://www.vickyteinaki.com/blog/born-digital-app-design-kids/)
- **Ages 4–6, the "muddy middle":** when kids drift, don't make the app more fun. Identify feedback opportunities inside the experience. Avoid overstressing winning, and keep rewards "small and low-key, if used at all." Kids like strong feedback, but intense or constant feedback irritates the adults nearby. Kids aged 4–6 want feedback for doing things right and like showing their work to others. — [A List Apart: Kids 4–6: The Muddy Middle (Gelman excerpt)](https://alistapart.com/article/kids-4-6-the-muddy-middle/)
- **Ages 6–8:** "Don't want to be seen as babyish, like to know the rules and do things right". Give 6–8s instructions up front (secondary summary of Gelman). — [Vicky Teinaki](https://www.vickyteinaki.com/blog/born-digital-app-design-kids/)
- **Language-light UI for pre-readers, plus multi-touch** (Pok Pok). — [Sketch blog](https://sketch.com/blog/pok-pok)
- **Voice for every instruction for pre-readers:** give instructions verbally. — [NN/G](https://www.nngroup.com/articles/usability-testing-minors/); Duolingo ABC's heavy use of narration — [Duolingo Blog](https://blog.duolingo.com/a-good-read-building-duolingo-abc-for-android/)
- **Make mistakes cheap:** in Pok Pok, a mistake "doesn't cost anything," and there is no win or lose state. — [Mom's Choice](https://momschoiceawards.com/blog/interview-with-moms-choice-award-winner-pok-pok-montessori-preschool-app/)
- **Kids have ownership:** Sago avoids artificial incentives, formal rules and scripted characters. — [Sago CEO message](https://sagomini.com/article/message-from-the-ceo/)
- **Toy, not game**, for young children (Toca). — [Kill Screen](https://www.killscreen.com/secret-smart-kids-entertainment-give-them-toy-not-game/)
- **Parents are the gatekeeper:** purchases happen in the first 48 hours, and the parent app reinforces value. — [RevenueCat podcast](https://www.revenuecat.com/blog/growth/how-to-build-a-great-kids-app-with-minimal-data-podcast-with-sago-minis-brennan-clark); [Entrepreneur](https://www.entrepreneur.com/article/276350)
- **Test weekly with real kids** (Sago), and **at every design stage** (PBS KIDS/Rockman). — [RevenueCat](https://www.revenuecat.com/blog/growth/how-to-build-a-great-kids-app-with-minimal-data-podcast-with-sago-minis-brennan-clark); [Rockman](https://rockman.com/topic/childrens-youth-media/)
- **Test single visual and audio elements:** a busy background can break the learning task (Khan Kids height comparison). — [Khan Academy Blog](https://blog.khanacademy.org/prototyping-playful-and-nimble-pre-k-assessments/)
- **Concrete to abstract fading** (DragonBox's monster cards becoming letters and numbers). — [Getting Smart](https://www.gettingsmart.com/2013/05/24/dragonbox-this-is-how-you-gamify/)
- **Modular features you can test in isolation** (Duolingo ABC). — [Duolingo Blog](https://blog.duolingo.com/a-good-read-building-duolingo-abc-for-android/)

### Inferences
- **Tension to resolve: "highly gamified" vs. practitioner consensus.** Gelman (small, low-key rewards for ages 4–6), Pok Pok (no points or levels), Sago (no artificial incentives) and Habgood (intrinsic beats extrinsic) all push against heavy extrinsic gamification for young children. Resolve it by tiering per child:
  - Toddler: a pure toy. No goals, no fail states, cause and effect, autoplay voice.
  - 5.5-year-old: gentle feedback-rich play with collectibles and visible growth (a garden, a zoo), but no scores, timers or loss.
  - 8.5-year-old: more game structure is fine. Rules, quests, streaks and challenges suit 6–8s who "like to know the rules and do things right" and dislike anything babyish. Keep the core mechanic as the learning act.
- **One shared world, three age skins.** The two-year-band rule means one interface cannot serve 2.5, 5.5 and 8.5. A shared world with per-child profiles that change the UI density, the text-vs-voice mix and the challenge level is the practical answer. The older sister will reject anything that looks like her toddler brother's app.
- **Calibrate feedback intensity for the adults too.** Gelman's warning that loud, constant feedback irritates adults matters in a family home. Offer a "quiet mode" and keep celebratory sounds short.
- Rules from the brief that are probably true but not sourced here ("first 30 seconds," "kids skip tutorials," "audio is 50% of the experience," "build a vertical slice") are listed in Gaps. A sensible working version: teach by doing in the first interaction, with no tutorial screens for pre-readers, and build one complete vertical slice (one mechanic, one age, with real Hebrew audio) before going wide.

### Gaps
- No first-hand practitioner source was found for "audio is 50% of the experience," "first 30 seconds," "every screen one goal," or "kids skip tutorials." They are widely repeated in industry, but no citable origin was located in this session.
- Nintendo Brain Age / Ring Fit lessons were not researched.

---

## 6. Books and resources a newcomer should read

### Takeaway
Start with Gelman's *Design for Kids* (practical, organized by age band), Habgood's intrinsic-integration work (why gamification fails or succeeds for learning), Druin's Cooperative Inquiry papers (testing and co-design), the Sesame Street research literature (formative research model), and the *Learning Engineering Toolkit* (process and data). Add NN/G's kids-testing tips and the Joan Ganz Cooney Center's design-with-kids pieces for method.

### Cited Findings
- *Design for Kids: Digital Products for Playing and Learning*, Debra Levin Gelman (Rosenfeld Media, 2014). It is organized by age band (2–4, 4–6, 6–8, and older), and Gelman led research on PNC's Planet Orange. — [Rosenfeld Media](https://rosenfeldmedia.com/books/design-for-kids/); [UXmatters review](https://www.uxmatters.com/mt/archives/2014/07/design-for-kids-digital-products-for-playing-and-learning.php); [UXPod interview](https://uxpod.com/episodes/design-for-kids-an-interview-with-debra-levin-gelman.html); free excerpt at [A List Apart](https://alistapart.com/article/kids-4-6-the-muddy-middle/)
- *Learning Engineering Toolkit*, Goodell & Kolodner (Routledge, 2022). — [O'Reilly](https://www.oreilly.com/library/view/learning-engineering-toolkit/9781000683257/)
- Habgood, *The Effective Integration of Digital Games and Learning Content* (PhD, Nottingham, 2007), free PDF. — [Nottingham ePrints](https://eprints.nottingham.ac.uk/10385/1/Habgood_2007_Final.pdf)
- Druin, "Cooperative Inquiry: Developing New Technologies for Children with Children" (CHI 1999), free tech report; plus "The role of children in the design of new technology" (2002). — [UMD HCIL TR 99-14](http://www.cs.umd.edu/hcil/trs/99-14/99-14.pdf)
- Gerald Lesser, *Children and Television: Lessons from Sesame Street* (1974). This is the origin text of the Sesame model. — [Wikipedia entry](https://en.wikipedia.org/wiki/Children_and_Television:_Lessons_from_Sesame_Street)
- NN/G, "Usability Testing with Minors: 16 Tips." — [NN/G](https://www.nngroup.com/articles/usability-testing-minors/)
- Joan Ganz Cooney Center, "Designing with Kids: How Children and Adults Can Co-create New Technology" (2023), and the App Fairy podcast interviews with Toca Boca. — [JGCC 2023](https://joanganzcooneycenter.org/2023/03/22/designing-with-kids/); [JGCC Toca transcript](https://joanganzcooneycenter.org/?p=18640)
- Podcasts: RevenueCat Sub Club with Sago Mini's Brennan Clark; Tank Talks episode 202 with Pok Pok's Melissa Cash; *Parenting, Apparently* (hosted by Cash and Huybreghts); Forward Obsessed with Melissa Cash. — [RevenueCat](https://www.revenuecat.com/blog/growth/how-to-build-a-great-kids-app-with-minimal-data-podcast-with-sago-minis-brennan-clark); [Tank Talks](https://tanktalks.substack.com/p/building-an-award-winning-app-through); [Apple Podcasts](https://podcasts.apple.com/al/podcast/parenting-apparently/id1855989506); [Forward Obsessed](https://www.forwardobsessed.com/episodes/melissa-cash)
- SRI and CRESST PBS KIDS reports, for examples of measuring learning from gameplay. — [SRI 2012](https://www.sri.com/wp-content/uploads/2021/12/gaming_body_feb_13_0.pdf); [CRESST](https://cresst.org/portfolio/how-cresst-helped-pbs-kids-improve-childrens-learning-through-gameplay/)

### Inferences
- A suggested reading order for a busy UX-designer parent: (1) the Gelman A List Apart excerpt and the age chapters for 2–4, 4–6 and 6–8, which map directly onto the three kids; (2) the Habgood thesis summary and the KQED article; (3) NN/G's 16 tips; (4) Druin 1999; (5) the Sesame Street research overview; (6) the Learning Engineering Toolkit chapters on process and data instrumentation. Listen to the Sago and Pok Pok podcasts on commutes.

### Gaps
- Not verified this session: *The Art of Game Design* (Schell; its kids-related chapters), *A Theory of Fun* (Koster), *Designing Digital Products for Kids* (Rubens Cantuni, Apress 2019), *Mind in the Making* (Ellen Galinsky), *Learning by Design*, the Dust or Magic conference, and the "Design for Kids" and "Kids Media Club" podcasts. These are widely recommended, but no source was retrieved for them in this session, so the report writer should mention them as unverified suggestions only.
