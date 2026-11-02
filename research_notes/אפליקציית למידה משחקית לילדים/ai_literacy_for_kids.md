# AI Literacy & "AI-Native" Thinking for Kids 2.5–10: Curricula, Research, Skill Tree, In-Game Activities, Safety/Policy

Research context: Nov 2026. Family = Israeli, kids 2.5 / 5.5 / 8.5, iPhone/iPad.
Method note: WebFetch could not reach several primary hosts in this session (ai4k12.org, raise.mit.edu, jns.org failed DNS in the sandbox, so they need to be allowlisted). Findings therefore rest on search-result extracts. Where only a secondary source was available, that is noted. Anything I know from background but could not verify this session is listed under Gaps, not presented as fact.

---

## Q1. Existing K-5 AI literacy curricula and national initiatives: what they teach and how they progress

### Takeaway
The mature K-5 frameworks (AI4K12 "Five Big Ideas", UNESCO's 2024 student framework, MIT Day of AI) agree on three things. They use a spiral of Understand → Apply → Create. At K-2 they rely on concrete, embodied, unplugged modeling such as building a model, using a decision tree, or sorting. Hands-on "train a classifier" activities (AI for Oceans, Teachable-Machine style) start around age 8. Almost none of them yet teach generative-AI prompting or agent direction to under-10s, which leaves a real design gap for the game to fill. Governments (Israel, Beijing, Estonia, US) moved fast in 2025–26, but mostly for grades 3/4+ and secondary.

### Cited Findings
**AI4K12 (US, AAAI/CSTA)**
- Five Big Ideas: (1) Perception, (2) Representation & Reasoning, (3) Learning, (4) Natural Interaction, (5) Societal Impact. Societal Impact sits at the center of the poster because it connects to all the other ideas — [AI4K12 gradeband progression charts](https://ai4k12.org/gradeband-progression-charts/)
- Guidelines cover K-2, 3-5, 6-8 and 9-12 bands. Each big idea is unpacked into concepts and subconcepts per band. The charts were posted as drafts for public comment — [AI4K12](https://ai4k12.org/gradeband-progression-charts/); [AI4K12 "AI Thinking in K-12"](https://ai4k12.org/artificial-intelligence-thinking-in-k-12/)
- Example progression for Big Idea 2 (Representation & Reasoning). K-2: "Construct a model of something and compare it to the thing being modeled; Use a decision tree to make a decision." 3-5: "Create/design a representation of…" — [AI4K12 ISTE 2019 presentation](https://dcmpx.remotevs.com/com/github/SL/touretzkyds/ai4k12/raw/master/documents/ISTE_2019_Presentation_website_final.pdf)

**MIT RAISE — Day of AI**
- Day of AI added an all-new K-2 curriculum alongside its 3-5+ materials — [MIT Open Learning](https://openlearning.mit.edu/news/teachers-bring-mits-day-ai-your-classroom)
- Each grade-band curriculum is about 4 hours of 30–60-minute lessons, built for teachers with little or no tech background. It is free under Creative Commons — [THE Journal](https://thejournal.com/articles/2023/06/06/day-of-ai-k12-curriculum-freely-available-online-anytime.aspx); [Day of AI](https://dayofai.org/?p=20)
- Recommended sequence: everyone starts with "What is AI?" lessons, then picks further units. The program describes itself as a spiral curriculum aligned with the UNESCO AI Competency Framework for Students — [Day of AI](https://dayofai.org/?p=20)

**UNESCO AI Competency Framework for Students (2024)**
- 12 competencies across 4 dimensions: human-centred mindset, ethics of AI, AI techniques & applications, AI system design. There are 3 progression levels: Understand → Apply → Create. It was released at Digital Learning Week 2024 — [UNESCO](https://www.unesco.org/en/articles/ai-competency-framework-students); [Oide (Ireland)](https://www.oidetechnologyineducation.ie/ai-hub/projects-research-and-insights/ai-frameworks-and-guidance-for-education/the-unesco-competency-framework-for-students/)
- Stated aims: critical judgement of AI solutions, citizenship responsibility, foundational AI knowledge, and inclusive, sustainable design — [UNESCO](https://www.unesco.org/en/articles/ai-competency-framework-students)
- Do not confuse it with UNESCO's separate teacher framework (15 blocks, 5 aspects × 3 levels) — [ICHEI](https://en.ichei.org/en/news/information/924.html)

**Code.org**
- AI for Oceans: students train a bot to sort fish from trash, which teaches machine learning, training data and bias. It targets ages 8+. Code.org pages list the grade range inconsistently as 3-8 or 3-12. It was launched for Hour of Code 2019 — [Code.org Medium](https://codeorg.medium.com/ai-for-oceans-a-csforgood-activity-14ef6ebd8178); [GitHub ml-activities](https://github.com/code-dot-org/ml-activities); [i-programmer](https://www.i-programmer.info/news/150-training-a-education/13315-ai-for-oceans-kids-use-computer-science-for-good.html)
- Code.org now runs an "Hour of AI" and a CodeAI hub — [Code.org Hour of AI](https://code.org/en-US/hour-of-ai); [Code.org AI](https://code.org/en-US/artificial-intelligence)

**Raspberry Pi Foundation — Experience AI (with Google DeepMind)**
- Current lessons are for ages 11–14. For younger learners the Foundation points to Code Club AI/ML projects, which are short and beginner-friendly — [Raspberry Pi Foundation blog](https://www.raspberrypi.org/blog/ai-education-resources-ai-awareness-day/)

**Common Sense Media / Common Sense Education**
- The AI Literacy collection's K-5 section is "Foundational Media Literacy" with 12 resources: fact vs. opinion, using tech to learn (grade 2), searching online. The explicitly AI-titled lessons ("What Is AI?", "How Is AI Trained?") are listed for grades 6+ — [Common Sense AI Literacy Lessons K-12](https://www.commonsense.org/education/collections/ai-literacy-lessons-for-grades-k-12); [What Is AI? lesson](https://www.commonsense.org/education/digital-citizenship/lesson/what-is-ai); [How Does AI Learn?](https://www.commonsense.org/education/digital-literacy/how-does-ai-learn)
- Common Sense also publishes an "AI Literacy Toolkit for Families", "Common Sense Rules for the AI Era", and a newer curriculum framed as building "the human skills AI is quietly eroding" — [Families toolkit](https://www.commonsense.org/education/families-ai-literacy-toolkit); [Rules for the AI Era](https://www.commonsense.org/education/articles/common-sense-rules-for-the-ai-era); [Press release](https://www.commonsensemedia.org/press-releases/common-sense-medias-new-curriculum-builds-the-human-skills-ai-is-quietly-eroding)

**Israel (Ministry of Education)**
- Feb 2025: a national AI-in-schools program for grades 4–12 across all sectors. Plans covered 70,000 trained teachers and 3,000 mentors from 400+ tech companies, with Google, Microsoft, Apple and Nvidia as partners — [JNS](https://www.jns.org/israel-unveils-groundbreaking-ai-education-plan-for-2025/); [Calcalist](https://www.calcalistech.com/ctechnews/article/lde52kasn); [Madan](https://madan.org.il/en/news/israel-unveils-ai-powered-education-plan)
- Tools named: a training bot "Q", a Gemini-based education chat assistant, MagicSchool for lesson planning, and a customized Minecraft interface for younger students — [JNS](https://www.jns.org/israel-unveils-groundbreaking-ai-education-plan-for-2025/); [JFeed](https://www.jfeed.com/tech/sr3shn)
- "Project 720": a 28-school pilot in 2025–26 across 17 authorities, expanded to 180+ schools with about 300 more applying. A first-year external evaluation covered 716 students in 21 schools. Critics cite the teacher shortage and question whether the program can be sustained — [JNS feature](https://www.jns.org/feature/israels-ai-classroom-experiment-is-going-national)
- Plan for all high-schoolers to learn English through personalized AI lessons — [Jerusalem Post](https://www.jpost.com/israel-news/article-904365)

**Beijing / China**
- Beijing's 2025–27 school AI plan (issued 6 Mar 2025) requires at least 8 class hours of AI per academic year from fall 2025 (1 Sept), in primary and secondary schools. Lessons can be standalone or integrated, for example into IT. Primary schools focus on experiential courses, junior high on applying AI in study and daily life, and high school on AI applications and innovation. A curriculum outline (trial) followed in late June 2025 — [gov.cn](https://english.www.gov.cn/news/202503/12/content_WS67d18e9ec6d0868f4e8f0c40.html); [China Daily, Jul 2025](https://mobile.chinadaily.com.cn/cn/html5/2025-07/04/content_005_6866ef5ded50ccabe1515119.htm)
- By Oct 2025, 1,400+ Beijing schools were to provide at least 8 hours, described as China's first provincial-level compulsory comprehensive AI education — [China Daily](https://govt.chinadaily.com.cn/s/202510/21/WS694a49c7498e368550339296/beijing-makes-ai-education-compulsory-in-public-schools.html)

**Estonia — AI Leap (TI-Hüpe)**
- Pilot year 2025–26 covers 154 of 156 upper-secondary schools, 4,800 teachers and 20,000 students (grades 10–11 from 1 Sept 2025). Eurydice reports 3,000 teachers instead. Expansion to 58,000 students is planned by 2027. A University of Tartu team (Jaan Aru) is studying the student app's impact, with results expected in 2026 — [TI-Hüpe](https://tihupe.ee/en/); [Eurydice](https://eurydice.eacea.ec.europa.eu/news/estonia-ai-leap-initiative-enhance-learning-and-teaching); [TI-Hüpe study](https://tihupe.ee/en/extensive-study-to-determine-impact-of-ai-leap-app-on-learners/)

**United States**
- EO 14277, "Advancing Artificial Intelligence Education for American Youth" (23 Apr 2025), created a White House Task Force on AI Education chaired by the OSTP Director. It also ordered a Presidential AI Challenge with multiple age categories, to be held within 12 months, plus K-12 public-private resources and teacher-training grants — [White House](https://www.whitehouse.gov/presidential-actions/2025/04/Advancing-Artificial-Intelligence-Education-For-American-youth/); [Akin Gump](https://www.akingump.com/en/insights/ai-law-and-regulation-tracker/president-trump-signs-executive-order-on-advancing-ai-education); [Holland & Knight](https://www.hklaw.com/en/insights/publications/2025/04/executive-order-advancing-ai-education-for-american-youth)

### Inferences
- The consensus progression across AI4K12, UNESCO and Day of AI can be mapped straight onto game "worlds": Understand (what AI is and isn't; how it "sees", "learns" and "talks") → Apply (use it well: ask, check, fix) → Create (make things with it and design tiny AI systems). Societal impact and ethics run through every level, not as a separate world.
- Israel's official school program starts at grade 4. The 8.5yo (grade 3) is about to meet AI in school, so the game can pre-empt that. The 5.5yo and 2.5yo are below every national program found, so the game would be ahead of school for them.
- None of the K-5 curricula found explicitly teaches prompting, multi-agent delegation or "AI-native" workflows. Those skills have to be derived from computational thinking (decomposition, algorithms, debugging) and from co-creation research (Q4). This is the game's differentiator, and also its main area of unvalidated design risk.
- AI4K12's K-2 "decision tree" and "build a model and compare it to the real thing" objectives are cheap, unplugged and turn naturally into game mechanics (see Q5).

### Gaps
- The exact K-2/3-5 statements for each Big Idea (AI4K12 charts) and the actual Day of AI K-2 lesson titles could not be fetched because the hosts were blocked. I could not verify the "Pickles the dog" K-2 lesson named in the brief. My background recollection is that a Day of AI lesson uses a dog character to teach how AI learns from examples, but this is unverified.
- Teachable Machine (Google), Machine Learning for Kids (Dale Lane), Cognimates (Druga), MIT App Inventor AI units and Elements of AI for kids were not covered by any retrieved source. From background knowledge, all are free browser-based "train a classifier on your own images/sounds/text" tools. They are all web-first and not designed for under-8 iPad use, but this is unverified here.
- No primary Israeli MoE document on elementary-grade (K-3) AI literacy, or on rules for student chatbot use, was retrievable. Hebrew-language MoE sources (e.g., pop.education.gov.il) should be checked.
- The China national (MoE-wide) policy, as opposed to Beijing's, was not confirmed. I did not find Finland-specific 2025 school AI initiatives.
- The Presidential AI Challenge's 2026 status and age categories were not retrieved.

---

## Q2. Research on how children perceive, trust, learn with and create with AI (and the risks)

### Takeaway
Young children readily attribute minds and cognitive abilities to talking AI. Their trust becomes more selective with age, shifting toward trusting devices for facts and humans for personal or social knowledge. They still notice errors when given the chance. Parents strongly shape how children understand AI, and family co-use works best with tangible, embodied activities. Conversational agents that ask questions can match a human partner on story comprehension for ages 3–6. The main risk is cognitive offloading ("cognitive debt"), which argues for designs where the child thinks first and the AI only assists.

### Cited Findings
**Anthropomorphism and perception**
- Most preschool-aged children perceived conversational agents as having cognitive abilities that let them "understand" speech — [Xu & Warschauer, CHI 2020](https://dl.acm.org/doi/fullHtml/10.1145/3313831.3376416)
- Druga, Williams & Breazeal (2018) found that parents' views of machine intelligence shaped how their children judged smart toys' intelligence — [Raspberry Pi Foundation research summary](https://www.raspberrypi.org/blog/ai-literacy-children-families-working-together-ai-education-research/)
- Druga & Ko (2021) studied how children's perceptions of machine intelligence change when they train and code "smart" programs. The work appears under the heading "Growing up with AI" in Druga's seminar — [RPF seminar slides](https://www.raspberrypi.org/app/uploads/2022/03/Family-as-a-Third-Space-for-AI-Literacy.pdf)
- Title of one study, "Alexa doesn't have that many feelings: Children's understanding of AI", suggests children reason explicitly about AI feelings and minds — [arXiv 2305.05597](https://arxiv.org/pdf/2305.05597)

**Trust and verification**
- Girouard-Hallam & Danovitch (2022, *Developmental Psychology*): with age, children trust the voice assistant more for factual information and the human more for personal information. Trust depends on age and on information type — [ResearchGate](https://www.researchgate.net/publication/359544774_Children's_trust_in_and_learning_from_voice-assistants); [Semantic Scholar](https://www.semanticscholar.org/paper/Children's-trust-in-and-learning-from-voice-Girouard-Hallam-Danovitch/0f6a04862795925a70484d1ac4b0a953144b5a2f)
- Streble, Gregg, Daugherty, Girouard & Danovitch: older children were significantly more likely to trust a device's stable-fact answers and a human's personal-fact answers. Familiarity in that sample: Alexa 48.3%, Siri 33.6%, Google Home 18.2% — [U. Louisville](https://ir.library.louisville.edu/uars/31/)
- "Hey Curio" (2025, 22 children): children generally trusted information from AI but stayed sensitive to its accuracy. Their main home uses were factual questions and music or stories — [ACM IDC 2025](https://dl.acm.org/doi/full/10.1145/3713043.3727059)
- A 2026 review notes that younger children's reliance on AI may be driven by social preferences, while older children's reliance is guided more by explicit judgements of reliability — [Computers in Human Behavior Reports, 2026](https://www.sciencedirect.com/science/article/pii/S2451958826003738)

**Learning with conversational agents (Ying Xu)**
- A storytelling conversational agent for book reading: children responded actively to its prompts, reacted to its feedback with strong emotion, and quickly learned how to interact with a "digital interlocutor" — [Xu & Warschauer, CHI 2019 EA](https://dl.acm.org/doi/fullHtml/10.1145/3290607.3299035)
- RCT with 90 children aged 3–6: dialogic reading with a smart-speaker agent that asked questions and gave contingent feedback was compared with reading with an adult. The title reports that dialogue with the agent improved story comprehension by increasing engagement — [Xu et al., Child Development 2022](https://onlinelibrary.wiley.com/doi/10.1111/cdev.13708)
- A companion paper reports "same benefits, different communication patterns" for an agent vs. a human reading partner — [Computers & Education](https://www.sciencedirect.com/science/article/abs/pii/S0360131520302578)

**Family co-use (Druga)**
- "Family as a Third Space for AI Literacies" (Druga, Christoph & Ko, CHI 2022) ran a 5-week in-home program with families (18 children aged 5–11 and 16 parents). Parents took multiple distinct roles, and tangible/embodied activities worked well for joint learning — [Paper PDF](https://faculty.washington.edu/ajko/papers/Druga2022FamilyAILiteracy.pdf); [ACM](https://dl.acm.org/doi/fullHtml/10.1145/3491102.3502031); [RPF blog](https://www.raspberrypi.org/blog/ai-literacy-children-families-working-together-ai-education-research/)
- The "4As" family AI-literacy framework is Ask, Adapt, Author, Analyze — [Druga et al., JoDS draft](https://bigyipper.com/wp-content/uploads/2021/04/JODS_Author_Draft_The_4As__Ask__Adapt__Author__Analyze___AI_Literacy_Framework_for_Families.pdf)

**Co-creation and creativity**
- 8 children aged 5–6 co-created stories with ChatGPT alongside parents, prompting it and responding to AI story elements. Children aged 7–13 revised prompts iteratively to refine stories, images and dialogue, which built creative self-efficacy ("I want it to talk like Darth Vader") — [CHI 2024](https://dl.acm.org/doi/10.1145/3613904.3642492)
- Children aged 8–13 accepted some AI contributions and rejected others. They resisted AI "content expansion" that went beyond their own imagination. Recommendation: make AI reinterpretations visible and editable, and preserve the child's intent — [CHI 2026 "Generative AI in Children's Creative Collaboration"](https://dl.acm.org/doi/full/10.1145/3772318.3790907); [CHI 2026 collaborative interaction patterns](https://dl.acm.org/doi/10.1145/3772318.3791831)
- A scoping review of 20 child-AI co-creation studies identified four AI roles, including "Generator" (child gives a broad prompt and the AI fills in the rest). It warns that heavy AI scaffolding can limit children's free exploration — [IDC 2025 review](https://dl.acm.org/doi/10.1145/3713043.3731506)
- Youth prompting with Microsoft Copilot at 4-H STEM camps has been studied — [ACM 2026](https://dl.acm.org/doi/10.1145/3773077.3812166)

**Over-reliance / cognitive offloading**
- MIT Media Lab, "Your Brain on ChatGPT" (2025): 54 university students wrote essays with an LLM, with search, or with no tools ("brain-only"). Brain-only writers showed the strongest and most distributed EEG connectivity, search users were intermediate, and LLM users the weakest. LLM users reported low ownership and struggled to quote their own essays. In the crossover session, the LLM→brain group stayed under-engaged, while the brain→LLM group showed stronger recall. The authors call this "cognitive debt" — [LIRNEasia journal club PDF](https://lirneasia.net/wp-content/uploads/2025/12/Your-Brain-on-ChatGPT_Final.pdf); [LIRNEasia](https://lirneasia.net/2025/12/lirneasia-journal-club-your-brain-on-chatgpt-exploring-the-cognitive-debt-of-ai-assisted-writing)
- Caveats: the sample is small, adults only, essays only, and the paper was not peer-reviewed at release. The authors warn against claims that ChatGPT "damages the brain". Popular coverage overstated the findings — [LIRNEasia](https://lirneasia.net/2025/12/lirneasia-journal-club-your-brain-on-chatgpt-exploring-the-cognitive-debt-of-ai-assisted-writing); [C2ST breakdown](https://www.c2st.org/?p=20545)

### Inferences
- For the 2.5yo and 5.5yo, the agent persona is a powerful engagement lever (Xu), but it will be anthropomorphized (Xu & Warschauer 2020). The game should state plainly that the helper is a machine. For example, a robot with visible "gears", which "doesn't have feelings, it follows patterns", and which openly makes mistakes. It should not have a human-like friend persona.
- Parents' framing transfers to children (Druga 2018, 2022). A short parent prompt card per session ("ask your child: how did the robot know that?") is likely to have outsized impact. Parent-child co-play should be a core mode, not an add-on.
- Trust research shows that "AI is good at facts" is the default belief that grows with age. The verification curriculum therefore matters most for the 8.5yo. It should teach that AI can be confidently wrong about facts too, and that AI is not the source for personal or family knowledge ("Only Mom knows what Grandma likes").
- Cognitive-debt evidence (adult, preliminary) plus the co-creation findings support a "you think first, then the AI" rule as a game mechanic: the child draws, plans or guesses before the AI acts. AI output should be editable and rejectable, and the child keeps authorship.

### Gaps
- No longitudinal or large-sample evidence was found on generative-AI use effects for children under 8.
- I could not retrieve specific Cognimates study findings, or the full methods and effect sizes for the Girouard-Hallam/Danovitch studies.
- I found no direct research on toddlers (2–3) and generative AI. Guidance for the 2.5yo must be inferred from general early-childhood media research, which is outside this scope and likely covered by another researcher.

---

## Q3. Prompting and "agentic" thinking for kids: specificity, context, iteration, decomposition, delegation, verification

### Takeaway
No retrieved curriculum teaches prompting or agent orchestration to under-10s. The closest evidence is the co-creation research (iterative prompt revision builds self-efficacy, and children want to keep their own intent), Druga's 4As, and AI4K12's K-2 decision-tree and modeling objectives. The practical approach is to recast prompting as computational thinking: precise instructions (algorithms), breaking goals into steps (decomposition), checking and fixing (debugging), and assigning jobs to helpers (delegation). Teach it unplugged first with a parent, then through a sandboxed in-game AI.

### Cited Findings
- Iterative prompt revision by children aged 7–13 improved creative self-efficacy and confidence in shaping AI outputs — [CHI 2024 "I want it to talk like Darth Vader"](https://dl.acm.org/doi/10.1145/3613904.3642492)
- Children resist AI over-expansion and want their intent kept. Designs should make AI changes visible and editable — [CHI 2026](https://dl.acm.org/doi/full/10.1145/3772318.3790907)
- A second- to fourth-grade visual storytelling platform had children define character traits and emotions and choose scenes. AI-generated keywords then guided them in building each scene's plot. This is structured "prompt slots" rather than a free text box — [IDC 2025 co-creation review](https://dl.acm.org/doi/10.1145/3713043.3731506)
- Druga's family framework of Ask → Adapt → Author → Analyze maps naturally onto prompt, iterate, create and evaluate — [4As JoDS draft](https://bigyipper.com/wp-content/uploads/2021/04/JODS_Author_Draft_The_4As__Ask__Adapt__Author__Analyze___AI_Literacy_Framework_for_Families.pdf)
- AI4K12 K-2 objectives include using a decision tree to make a decision, and building a model and comparing it with the real thing — [AI4K12 ISTE 2019](https://dcmpx.remotevs.com/com/github/SL/touretzkyds/ai4k12/raw/master/documents/ISTE_2019_Presentation_website_final.pdf)
- Druga's family study found tangible, embodied activities effective for joint parent-child AI learning — [RPF blog](https://www.raspberrypi.org/blog/ai-literacy-children-families-working-together-ai-education-research/)

### Inferences — Prompting/Agentic micro-skills translated for kids (design proposal, not sourced curriculum)
| Adult "AI-native" skill | Kid framing | First taught at |
|---|---|---|
| Be specific | "Robot takes you literally" — say exactly what you want | 5.5 (unplugged "Parent Robot"); 2.5 as simple 2-choice selection |
| Give context | "Tell the helper who it's for and why" | 5.5–6 |
| Show an example | "Show, don't just tell" (pick a sample picture) | 5.5 |
| Constrain format | "How many? How big? What colour? How long?" | 6–7 |
| Iterate | "Make it better: change one ingredient and try again" | 5.5 (pictures), 8 (stories) |
| Ask the AI to ask you | "Let the helper ask you 3 questions first" | 7–8 |
| Role prompt | "Put a hat on the helper: be a pirate / a teacher" | 6 (story voices) → 8 |
| Decompose | "Big mission → small steps" (cards in order) | 5.5 with 3 steps; 8.5 with 5–8 steps |
| Delegate to agents | "Each helper bot has one job" (researcher, artist, checker) | 8–9 |
| Verify | "Detective: find the mistake; check with a second source/a grown-up" | 5.5 (visual oddities) → 8.5 (facts) |
| Give feedback | "Tell the helper what to fix, kindly and precisely" | 6–8 |
| Know when not to use AI | "Brain first", "Some things only people know" | All ages |

- Unplugged pairing first: the "Parent Robot" game, where the child gives step-by-step instructions to a parent who follows them hyper-literally (e.g., making a sandwich). It directly builds specificity and debugging, and it fits Druga's finding that embodied family activities work.
- Using structured prompt slots ("ingredients") instead of free text suits pre-readers and Hebrew/English bilingual children, reduces the moderation surface, and matches the 2nd–4th-grade platform's keyword scaffolding.

### Gaps
- I found no peer-reviewed study evaluating explicit "prompt engineering" or multi-agent delegation instruction for ages 5–10. The skill table above is a design synthesis from computational thinking and co-creation research, not a validated curriculum.
- I did not verify sources for classic unplugged computational-thinking activities (CS Unplugged, the "PB&J sandwich" exercise). They are widely used but were not cited here.

---

## Q4. Platform policies and safe access for children (constrains game architecture)

### Takeaway
No mainstream general-purpose chatbot is permitted for the 5.5yo or 8.5yo directly. ChatGPT is 13+ (with teen parental controls since Sept 2025). Claude consumer is 18+, but Anthropic's Usage Policy allows products built for minors that add age-appropriate safeguards. Gemini allows under-13s only through supervised Google Family Link accounts, a parent opt-in that is unavailable in the EEA and UK. Character.AI removed open-ended chat for under-18s in Nov 2025. The game must therefore run a parent-mediated, sandboxed, structured AI layer of its own: constrained inputs, filtered outputs, no free open-ended chat for young kids, a parent dashboard, and parent consent under COPPA/GDPR-style rules.

### Cited Findings
- **OpenAI/ChatGPT:** minimum age is 13, with parental permission required under 18, and there is no under-13 product. An age-prediction system applies teen safeguards to likely minors — [Ban the Bots explainer](https://www.banthebots.org/explainers/ai-chatbot-age-requirements); [heyotto](https://www.heyotto.app/blog/how-old-to-use-chatgpt-ai-age-requirements) (secondary sources)
- **ChatGPT parental controls (launched 29 Sept 2025):** linking requires mutual acceptance of an invite. Parents can set quiet hours and turn off voice mode, memory and image generation, and opt the teen out of model training. Parents cannot see chat transcripts. A later "study hours" feature was added — [OpenAI Help Center](https://help.openai.com/en/articles/12315553-parental-controls-on-chatgpt-faq); [BetaNews](https://betanews.com/2025/09/29/openai-introduces-parental-controls-for-chatgpt/); [Windows Central](https://www.windowscentral.com/artificial-intelligence/openai-chatgpt/openai-introduces-parental-controls)
- **Anthropic/Claude:** the Consumer Terms require users to be 18 or older, or the local age of consent if that is higher. The Privacy Policy says data from users under 18 is deleted once discovered — [Anthropic Consumer Terms (archive)](https://www.anthropic.com/legal/archive/687101e2-9c3d-4ab3-8933-470b8520b3a1); [ConductAtlas tracker](https://conductatlas.com/platform/anthropic/anthropic-consumer-terms/minimum-age-requirement-for-services/)
- The Usage Policy lets organizations build products for minors on Claude if they add age-appropriate safeguards and comply with child-safety laws — [ConductAtlas – Usage Policy, products serving minors](https://conductatlas.com/platform/anthropic/anthropic-usage-policy/products-serving-minors-additional-requirements/); [Website Planet](https://websiteplanet.com/news/anthropic-defines-ai-kids-rules) (secondary trackers; verify against the current Anthropic page)
- **Google Gemini:** under-13s can use it only through a supervised Family Link account that the parent enables, and the parent is notified on first use. Image generation is reported disabled for minors, and under-13 access is reported unavailable in the EEA, Switzerland and the UK. Sources disagree about how much activity visibility parents get — [Kinzoo](https://www.kinzoo.com/blog/a-parents-guide-to-google-gemini-for-kids-everything-you-need-to-know); [kidsaitools](https://www.kidsaitools.com/en/articles/is-gemini-safe-for-kids) (secondary)
- **Character.AI:** on 29 Oct 2025 it announced removal of open-ended chat for under-18s. Enforcement began around 24–25 Nov 2025, using in-house behavioral age assurance plus Persona. Teens keep structured features such as Stories. The move was driven by lawsuits and lawmakers' pressure — [Character.AI blog](https://blog.character.ai/an-update-on-changes-to-our-under-18-experience/); [TechCrunch](https://techcrunch.com/2025/10/29/character-ai-is-killing-the-chatbot-experience-for-minors/?rand=23331); [Bloomberg](https://www.bloomberg.com/news/articles/2025-10-29/character-ai-to-ban-children-under-18-from-talking-to-its-chatbots); [CNBC](https://www.cnbc.com/make-it/2025/11/24/characterai-to-ban-teens-from-open-ended-chats-human-interaction-is-crucial-psychotherapist-says.html)
- **Israel:** the national school program uses a Gemini-based assistant adapted for education, i.e., a school-mediated deployment rather than consumer accounts — [JNS](https://www.jns.org/israel-unveils-groundbreaking-ai-education-plan-for-2025/)

### Inferences
- **Architecture:** the app should call LLM and image APIs server-side under the developer's (or parent's) account, wrapped in a child-safety layer: input whitelists or ingredient chips, output moderation, no personal data, no persistent "friend" memory. Children should never hold their own chatbot accounts. If Claude is used, this is the "product serving minors" path, which needs documented safeguards.
- Character.AI's retreat to "structured, non-open-ended" features for minors is a strong industry signal. The game's structured prompt-crafting (chips, cards, voice choices), rather than free chat, matches where the industry and regulators have landed.
- **Teaching real-tool skills without real-tool access:** simulate ChatGPT/Claude-like interfaces inside the game, with an obviously fictional helper character. Add a "Grown-up Co-pilot" mode where a parent uses their own account with the child beside them, following game-provided scripts. This mirrors the ChatGPT model, where parents are linked but cannot see transcripts. For under-13s the parent's presence is the safeguard.
- Bake in quiet hours, session limits and a parent dashboard, following ChatGPT's teen-control feature set.

### Gaps
- COPPA specifics (including the FTC's amended COPPA Rule, 2025) and Israeli Privacy Protection Law / Amendment 13 obligations for a children's app were not retrieved. Legal review is needed.
- Microsoft Copilot age rules, Khanmigo's under-13 model, Miko, PBS KIDS AI features and "Pinwheel" were not researched within the tool budget.
- I could not fetch Anthropic's current Usage Policy and Consumer Terms text directly. Policy summaries came from third-party trackers, and one news item suggests Anthropic set out a minors framework whose exact scope is unclear.

---

## Q5. Age-graded AI-literacy skill tree with concrete in-game activities

### Takeaway
Build one shared "AI Helpers Island" world with three difficulty lanes. The 2.5yo gets cause-and-effect plus "machine vs. person". The 5.5yo gets literal-robot instructions, sorting and training, spotting mistakes, and ingredient-based picture prompting. The 8.5yo gets decomposition, multi-agent missions, fact-checking and iterative story and image creation. Every activity follows "brain first → AI helps → you check → you decide", and parent co-play is built in.

### Cited Findings (evidence anchoring the activities)
- K-2 objectives: decision trees and build-and-compare models — [AI4K12 ISTE 2019](https://dcmpx.remotevs.com/com/github/SL/touretzkyds/ai4k12/raw/master/documents/ISTE_2019_Presentation_website_final.pdf)
- Train-a-sorter (fish vs. trash), covering training data and bias, for ages 8+ — [Code.org AI for Oceans](https://codeorg.medium.com/ai-for-oceans-a-csforgood-activity-14ef6ebd8178)
- Ages 3–6 learn well from a question-asking agent that gives contingent feedback — [Xu et al. 2022](https://onlinelibrary.wiley.com/doi/10.1111/cdev.13708)
- Ages 5–6 can co-create stories with ChatGPT alongside parents. Ages 7–13 gain self-efficacy from iterative prompting — [CHI 2024](https://dl.acm.org/doi/10.1145/3613904.3642492); [IDC 2025 review](https://dl.acm.org/doi/10.1145/3713043.3731506)
- Children want control over the AI's additions — [CHI 2026](https://dl.acm.org/doi/full/10.1145/3772318.3790907)
- Families learn AI best through tangible joint activities, and parents' views shape children's — [Druga et al. 2022](https://faculty.washington.edu/ajko/papers/Druga2022FamilyAILiteracy.pdf); [RPF](https://www.raspberrypi.org/blog/ai-literacy-children-families-working-together-ai-education-research/)
- Older children's trust tracks reliability judgements. The default belief is "devices = facts" — [Girouard-Hallam & Danovitch 2022](https://www.researchgate.net/publication/359544774_Children's_trust_in_and_learning_from_voice-assistants); [CHB Reports 2026](https://www.sciencedirect.com/science/article/pii/S2451958826003738)
- Spiral progression Understand → Apply → Create — [UNESCO](https://www.unesco.org/en/articles/ai-competency-framework-students); [Day of AI](https://dayofai.org/?p=20)

### Inferences — the skill tree (design proposal)

**Branch A: What is AI? (Understand)**
- A0 (2.5): "Who's talking?" Sort pictures into person, animal, machine. Tap the robot to see it "work" with lights and gears. Message: machines help, people feel.
- A1 (5.5): "Is it AI?" Sort everyday things (calculator, smart speaker, teddy bear, auto-correct, cartoon robot). "Robot learns from examples": show 5 cats → it guesses a new cat → show it a weird cat and it fails.
- A2 (8.5): "How the helper guesses". The "predict the next word" game: the child fills in the blank, the AI fills in the blank, compare. Message: it predicts patterns and does not "know". Explain why it can sound sure and still be wrong.

**Branch B: Training and data (AI4K12 Big Idea 3, Learning)**
- B1 (5.5): "Teach the Sorting Bot". Drag 6–10 examples (fruit vs. toys, fish vs. trash) and test the bot. If you only teach red apples, the bot thinks green apples are not fruit. This is a gentle introduction to bias.
- B2 (8.5): "Camera trainer" (Teachable-Machine-like, running on-device on iPad). Train gestures or drawings, find what fools it, then fix it with better and more varied data.

**Branch C: Talking to AI / prompting (Apply)**
- C0 (2.5): "Robot, please!" Choose between two picture-instructions (blue ball vs. red ball) and watch the robot do exactly that. Cause and effect, and polite asking. Voice-only, no reading.
- C1 (5.5): "Literal Robot Kitchen". Order picture-cards so the robot makes a sandwich. Missing steps cause comic failures (bread on head). Unplugged twin: "Program the Parent Robot".
- C2 (5.5–6.5): "Prompt Potion". Combine ingredient chips WHO + DOING WHAT + WHERE + STYLE + one DETAIL to brew a picture. Then "change one ingredient" to iterate. Later levels unlock "how many", "colour" and "mood" chips.
- C3 (8.5): "Better Question Quest". The helper answers vague questions vaguely. Upgrade the question with context ("for a 3rd grader", "in 3 bullet points", "with an example") and see the difference. Unlock the "Ask me questions first" spell, where the helper interviews the child before writing.
- C4 (8.5): "Role Hats". Give the helper a role (tour guide in Jerusalem, pirate, science teacher) and compare answers. Discuss: the facts should not change because of the hat.

**Branch D: Checking AI (Verify — critical for 8.5)**
- D1 (5.5): "Spot the Silly". AI pictures with a deliberate oddity (six-fingered hand, cat with three ears, upside-down clock) to find.
- D2 (8.5): "AI Detective". The helper gives 5 facts, 1–2 of them confidently wrong (e.g., about Israel's geography or animals). The child checks them with an in-game "library book", a map, or asking a parent. Earn badges for "two sources agree" and "asked a human".
- D3 (8.5): "Facts vs. personal knowledge". Questions the AI can't know ("What's Grandma's favourite song?") lead the child to learn to route them to people. This builds on Girouard-Hallam's fact vs. personal trust distinction.

**Branch E: Creating with AI (Create)**
- E1 (5.5, with parent): "Our Story Machine". The child picks characters and the problem, AI proposes the next scene, and the child accepts, changes or rejects it with a big "No, my idea!" button that keeps authorship. The parent reads aloud.
- E2 (8.5): "Comic Studio". The child scripts the panels (brain first), AI drafts the art, and the child edits and redoes until it is right. A checker step ensures the story makes sense.

**Branch F: Directing agents (Agentic thinking — 8.5 advanced, 5.5 light)**
- F1 (5.5): "Helper Jobs". Three bots each have one job (Fetch-bot, Paint-bot, Tidy-bot). Give the right job to the right bot.
- F2 (8.5): "Mission Control". For a goal like "Plan a family picnic in the Galilee" or "Make a poster about dolphins", the child breaks the mission into steps on cards, assigns Researcher-bot, Artist-bot, Writer-bot and Checker-bot, reviews each output, sends back feedback ("too long", "wrong animal"), and approves the final product. Score on clear instructions, sensible order, and checks done.
- F3 (8.5+): "Bug Hunt". A pre-made agent plan fails. Find which bot got an unclear instruction and fix it (debugging).

**Branch G: Responsible use (Societal Impact; woven through every branch)**
- "Brain first" token: some levels require the child to draw or guess before the AI helps, with a reward for trying.
- "When to ask a grown-up": safety scenarios like a stranger message, a scary image, or the helper asking for personal info, where the right move is to tell a parent.
- Fairness: the sorting-bot bias examples from B1/B2.
- Privacy: "Never tell the helper your address, school or password." This is a Hebrew/English voice line repeated across levels.

**Parent-child co-play layer (all ages)**
- Each session ends with a 1-minute parent prompt card in Hebrew ("Ask: how did the robot know?", "Find one mistake together"). This follows Druga's finding that parents shape children's AI concepts.
- "Grown-up Co-pilot" mode for the 8.5yo: the game gives a scripted mission to do on the parent's own ChatGPT/Claude/Gemini, with the child directing and the parent typing. The child practises real tools within platform rules.
- Sibling mode: the 8.5yo becomes "Mission Commander" and assigns a picture-only task to the 5.5yo, who plays "Artist-bot". This is a role-play of agent delegation, and the 2.5yo gets a "button-pressing robot" role.

**Indicative progression gates**
- 2.5: A0, C0 (voice and pictures, about 3-minute sessions, always with a parent).
- 5.5: A1, B1, C1, C2, D1, E1, F1 (pre-reader friendly, chips and voice).
- 8.5: A2, B2, C3, C4, D2, D3, E2, F2, F3 (reading in Hebrew/English, short typed or voice prompts within a moderated sandbox).

### Gaps
- None of the activity designs above has been tested with children. They are syntheses from the cited research and curricula and should be playtested.
- I found no research-validated age at which children can do multi-agent delegation. Placing F2 at 8.5 is a judgement based on the 3-5 grade-band shift toward "design/create" in AI4K12 and on the age range (7–13) of the iterative-prompting studies.
- I found no Hebrew-language child AI-literacy materials. Hebrew speech recognition and TTS quality for preschool voices should be checked separately.
