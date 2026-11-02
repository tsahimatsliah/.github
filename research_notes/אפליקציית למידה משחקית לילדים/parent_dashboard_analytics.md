# Parent Dashboard & Learning Analytics for a Kids' Learning App (Israeli family; kids 2.5 / 5.5 / 8.5; iPhone/iPad)

Research date: 2026-11-02. Scope: what metrics matter, how leading products present them, evidence on informing parents, learning-analytics pitfalls, actionable features, privacy, and a concrete recommended dashboard spec.
Method note: about 19 search/fetch calls. Several vendor help centers were reached only through search snippets. uk.ixl.com was blocked by the session network, so the IXL parent guide PDF could not be fetched directly. Claims are marked with their source. Design recommendations are labelled as inferences.

---

## 1. How leading products design parent/teacher views

### Takeaway
The best products don't lead with time or questions answered. They lead with a mastery score per skill, a "where they struggle" list that links to the exact missed items, and a periodic report card that compares this period with the last one. Time and activity sit in a secondary "activity" view. For preschool apps such as Lingokids and Khan Academy Kids, parent reporting is thin (activities and skills practised) or missing altogether. That gap is an opportunity.

### Cited Findings
**IXL (the most detailed parent analytics)**
- The SmartScore is a 0–100 per-skill score that is explicitly **not a percentage**. 80 = proficiency, 90 = excellent, 100 = mastery. — [IXL blog: using IXL Analytics at home](https://blog.ixl.com/2018/09/21/using-ixl-analytics-at-home/); [IXL blog](https://blog.ixl.com/?p=13740)
- The **Trouble Spots** report "pinpoints the concepts your child is struggling with, and you can even see the exact questions your child received and missed." — [IXL Analytics Guide for Parents (PDF)](https://uk.ixl.com/materials/Analytics_Guide_for_Parents.pdf) (quoted in a search snippet; direct fetch blocked)
- A teacher-side rule: a trouble spot is flagged when a student misses **3 questions within a specific item type in a skill**. I found no parent-specific threshold. — [IXL blog: 3 popular teacher reports](https://blog.ixl.com/?p=12993)
- Drilling into a skill shows the SmartScore progression for each practice session, plus the exact questions and the answers given. — [IXL blog](https://blog.ixl.com/?p=8752)
- IXL tells parents to set SmartScore goals of 80 or 90 on Trouble Spot skills, which turns a diagnosis into a next action. — [IXL blog: how to spot and address learning gaps](https://blog.ixl.com/?p=12836)
- Other parent reports: the Score Chart (whether assigned goals were hit) and the Student Summary (overview). — [IXL blog: fostering parent-teacher collaboration](https://blog.ixl.com/2020/04/30/fostering-parent-teacher-collaboration/)

**Khan Academy (main site)**
- The newer Parent Dashboard gives "one clear view of their growth–learning minutes, skills practiced, and lessons completed." — [Khan Academy blog: 2025 updates every parent should know](https://blog.khanacademy.org/?p=19766)
- The Activity tab lets parents filter by time, content, and activity type (exercises, videos, articles, quizzes). — [Khan help (via search)](https://support.khanacademy.org/hc/bg/articles/360032987591)
- Mastery mechanics: answering 5 in a row correctly marks a skill as "practiced", then mastery challenges level it up. Activity Overview counts *skills leveled up*. The Progress report counts *level changes*, so one skill leveling up three times shows as 3 there and as 1 in Activity Overview. That discrepancy is a known source of parent confusion. — [Khan support (via search)](https://kvzcldemo.zendesk.com/hc/en-us/articles/4407079934743-How-do-I-view-my-child-s-profile); [Khan Learning Dashboard blog](https://blog.khanacademy.org/introducingthe-learning-dashboard/)

**Khan Academy Kids (ages 2–8)**
- Kids-app progress **cannot** be viewed in the main Khan Parent Dashboard, because they are separate platforms. The help center describes teacher-facing reports (Assignments Report, All Progress Report). Parents have asked for a parent dashboard and I found no confirmation that one exists. — [Khan community thread](https://support.khanacademy.org/hc/nl/community/posts/360059050591/comments/360009725152); [Khan Kids help](https://khankids.zendesk.com/hc/en-us/articles/4403614100109); [community post](https://support.khanacademy.org/hc/hy/community/posts/33107105886221/comments/33291151152269)

**Prodigy Math**
- A **monthly Report Card** breaks down math skills, with a month-over-month comparison ("better or worse in specific areas"). Parents are emailed when a new one is ready. Grade and date selectors give access to past cards. — [Prodigy blog: report card tool](https://www.prodigygame.com/blog/report-card-tool)
- **Curriculum Progress** shows the % of curriculum finished at each grade. It can *drop* when new skills are added. Answering more questions doesn't necessarily raise it, because a skill counts only when mastered. — [Prodigy help: Curriculum Progress](https://prodigygame.zendesk.com/hc/en-us/articles/115004773966); [Prodigy help: view curriculum progress](https://prodigygame.zendesk.com/hc/en-us/articles/4407414190484)
- One third-party source gives the mastery rule as about 7 correct in a row in a skill. This is unverified against Prodigy's own docs. — [third-party deep dive](https://aistage.thestempedia.com/deep-dives/prodigy-dashboard-data-that-actually-helps-learning-456886)
- Other parent widgets: goals, Recent Activity, Grade Level, weekly questions answered. There is a separate dashboard per child. — [Prodigy blog: learning progress support](https://prodigygame.com/blog/learning-progress-support)

**Lingokids (ages 2–8, the closest analogue for the 2.5- and 5.5-year-olds)**
- The Progress Center is inside the Parents Area. It fills only after **5 completed activities** and shows activities finished plus "top-practised skills", meaning the skills where the child spent the most time and the activities that practised them. — [Lingokids help: Progress reports](https://help.lingokids.com/hc/en-us/articles/9569447484177-Progress-reports)
- Parental gate: the parent enters their birth year to get into the Parents Area. — [Lingokids help: Parents Area](https://help.lingokids.com/hc/en-us/articles/115005129325-Parents-Area)
- Lingokids publishes "apply in-app progress to everyday life" content, which links app progress to offline activities. — [Lingokids blog](https://lingokids.com/blog/posts/how-to-apply-your-childs-in-app-progress-to-everyday-life)
- Weekly cadence: third-party reviews say weekly reports or "last two weeks". The official help pages don't state a cadence. Treat it as unconfirmed. — [ling-app review](https://ling-app.com/blog/lingokids-review/)

**Zearn / DreamBox (teacher-first)**
- Zearn's **Tower Alerts** report shows which digital lesson a student struggled in, when, and whether they later completed it and moved on. The **Pace Report** shows lessons completed per student. Both are teacher-facing. — [Zearn help: Tower Alerts](https://help.zearn.org/hc/en-us/articles/236239748); [Zearn help: Pace Report](https://help.zearn.org/hc/en-us/articles/236279187-Pace-Report)
- Zearn has a "share progress with families" workflow. I couldn't retrieve its contents. — [Zearn help](https://help.zearn.org/hc/en-us/articles/360021586433-Share-progress-with-families)
- The DreamBox Parent Dashboard gives "a detailed current view of your child's academic and story progress". An older (2015) district document describes alerts for "where attention is needed and when to congratulate success". — [DreamBox parent reports](https://www.dreambox.com/parent-reports); [district letter](https://carmeldelmar.dmusd.org/documents/Parents--Community/Parent-Technology-Portal/Dreambox%20Parent%20letter%20Carmel%20Del%20Mar.pdf)

**Math Academy**
- XP is the core effort currency: "each XP being roughly equivalent to 1-minute of focused effort", with a daily XP goal. XP "enforces both correct answers and good math habits". — [Math Academy: How it works](https://mathacademy.com/how-it-works); [Justin Skycak](https://justinmath.com/what-is-the-highest-sustainable-daily-xp-on-math-academy/)
- One parent reports that 1 XP averages 2–3 real minutes for their family, so the calibration is approximate. — [Well-Trained Mind forum](https://forums.welltrainedmind.com/topic/733445-my-ongoing-review-of-mathacademycom-updated-62824)

### Inferences
- Common design pattern: (1) a per-skill mastery state with a non-percentage scale and clear named thresholds, (2) a struggle list with drill-down to the actual items, (3) a periodic report card compared with the previous period, (4) an activity log kept secondary. A "focused effort" unit, like Math Academy's XP, is better than raw minutes because idle time and guessing don't inflate it.
- Under-6 products give parents little beyond "what they played". Covering a curriculum plan, mastery, and offline tie-ins for pre-readers would be a differentiator for the 2.5- and 5.5-year-olds.
- Prodigy's Curriculum Progress dropping and Khan's double-counted level-ups both confuse parents. Define every metric in a tooltip, and never let a headline number go down for reasons unrelated to the child.

### Gaps
- I found no parent-facing documentation (from this search budget) for Duolingo Family Plan / Duolingo for Schools, ClassDojo, Seesaw, Epic, Beast Academy Online, or Brainly. Their designs are uncaptured here, and a report writer shouldn't describe them from these notes.
- I couldn't confirm IXL Diagnostic parent views, or the exact IXL "Progress & Improvement" and "Usage" charts, because the PDF fetch was blocked.
- Math Academy has no confirmed parent dashboard and no official guessing-penalty rule.

---

## 2. Research: does informing parents improve child outcomes? And what about pressure?

### Takeaway
Strong RCT evidence shows that frequent, specific, low-cost information to parents raises engagement and achievement, with the biggest gains for lower-performing children. Messages about *what to improve, with specifics* beat generic praise. Small actionable nudges (READY4K, PACT) change parent behavior a lot. *How* parents act on the information decides whether it helps: autonomy-supportive, failure-as-learning responses help, and controlling or pressuring ones hurt. So the dashboard has to coach the parent's response, not just report scores.

### Cited Findings
**Information to parents (school-age)**
- Bergman (Los Angeles RCT): parents got extra information on missing assignments and grades several times a month by email, text, and phone. The summary concludes that more information to parents may be a cost-effective way to raise achievement. The middle-school arm was contaminated because teachers replicated the treatment. — [Bergman CESifo WP 5391](https://www.cesifo.org/DocDL/cesifo1_wp5391.pdf); [NBER conf. version](https://conference.nber.org/conf_papers/f68410/f68410.pdf); [MSU summary](https://ippsr.msu.edu/node/525); [AEA registry](https://www.socialscienceregistry.org/trials/286)
- Bergman & Chan ("Leveraging Parents through Low-Cost Technology"): automated texts about grades, missed assignments, and absences in 22 middle and high schools. Effects were larger for below-median-GPA students. About 32,000 messages cost about $63 in total. — [Bergman & Chan 2020 PDF](https://static1.squarespace.com/static/60d0c05ace34212ef5a1131b/t/640570ea6622fd4805ea4a91/1678078189109/Bergman_Chan_Leveraging-Parents-through-Low-Cost-Technology_2020.pdf)
- Bergman, Edmond-Verley & Notario-Risk (2018, *Econ. of Ed. Review* 66): a three-arm trial comparing information alone with information plus home visits on parent skills. — [EconPapers](https://econpapers.repec.org/RePEc:eee:ecoedu:v:66:y:2018:i:c:p:92-103)
- World Bank synthesis: four recent papers (including Chile and Malawi) show that giving parents information can improve student outcomes. — [World Bank Development Impact blog](https://blogs.worldbank.org/impactevaluations/can-providing-information-parents-improve-student-outcomes-4-recent-papers-show-it-can-chile-malawi)
- Kraft & Rogers (2015, *Econ. of Ed. Review* 47:49–63): N=435 high-schoolers in summer credit recovery. Weekly one-sentence teacher-to-parent messages cut failure-to-earn-credit from **15.8% to 9.3% (−41%)**, mainly by preventing dropout. **Messages about what the student could improve beat messages about what they did well.** Cost per extra credit was under 1/10 of the district's typical cost per credit. — [Annenberg/Brown](https://annenberg.brown.edu/publications/underutilized-potential-teacher-parent-communication-evidence-field-experiment); [Boston Fed](https://www.bostonfed.org/publications/communities-and-banking/2016/spring/the-underutilized-potential-of-teacher-parent-communication.aspx); [EdWeek](https://edweek.org/leadership/study-teacher-outreach-to-parents-has-under-explored-potential-in-schools/2015/05)

**Preschool nudges (directly relevant to ages 2.5 and 5.5)**
- York, Loeb & Doss, "One Step at a Time" (*J. Human Resources* 2019, 54(3):537–566). READY4K! sent 8 months of texts to parents of preschoolers: a fact, a tip, and a growth message each week, aimed at behavioral barriers. Parent involvement at home and school rose **0.15–0.29 SD**, and early literacy rose **about 0.11 SD**. An earlier draft reports larger gains (**about 0.31 SD**) for children starting below the median. — [NBER W20659](https://www.nber.org/papers/W20659); [JHR](https://jhr.uwpress.org/content/54/3/537); [Stanford draft PDF](https://wayback.stanford.edu/was/20180311093300mp_/http://cepa.stanford.edu/sites/default/files/One%20Step%20At%20A%20Time%203_1_17_0.pdf)
- Mayer, Kalil, Oreopoulos & Gallegos, PACT (NBER w21602; *JHR* 54(4):900–925). Low-income preschool parents used a reading app. Reminders, goal-setting, and social rewards **more than doubled reading time (about 1 SD)** over 6 weeks. Information alone did **not** explain the effect. Gains were largest for the most present-oriented parents. — [NBER w21602](https://www.nber.org/papers/w21602.pdf); [EconPapers](https://econpapers.repec.org/RePEc:nbr:nberwo:21602)

**Growth- vs performance-oriented framing; praise; pressure**
- Haimovitz & Dweck (2016, *Psychological Science*): **parents' failure mindsets** (failure as debilitating vs enhancing), not their intelligence mindsets, predicted children's beliefs about intelligence. Children could perceive their parents' failure beliefs, and parents' *reactions to failure* carried the effect (experiment with 132 parents). The sample was mostly college-educated Bay Area mothers. — [ScienceDaily](https://www.sciencedaily.com/releases/2016/04/160428132136.htm); [BPS Research Digest](https://bps.org.uk/research-digest/parents-who-think-failure-harmful-learning-have-children-who-think-ability-fixed)
- Gunderson et al. (2013, *Child Development*): parents' **process praise** (effort, strategy) to children aged 14–38 months predicted incremental (growth) motivational frameworks at ages 7–8 (N=53). Person praise did not predict fixed beliefs. — [PMC3655123](https://pmc.ncbi.nlm.nih.gov/articles/PMC3655123)
- Pomerantz (Illinois studies): when mothers helped in a **controlling** way (taking over, giving commands), initially low-performing children disengaged. **Autonomy-supportive** help improved their performance. Controlling reactions to real failures, such as punishment, predicted further poor performance and lower grades 6 months later. Discussing failures supportively predicted improvement. — [Illinois News Bureau](https://news.illinois.edu/if-you-want-to-help-your-children-with-homework-take-it-easy-studies-suggest/); [Illinois LAS](https://las.illinois.edu/news/2004-07-01/helping-homework)
- Grolnick & Pomerantz (2022 review): parent involvement helps motivation and learning "particularly when such involvement is autonomy supportive and affectively positive". Homework is where controlling, negative involvement is most likely to happen. — [Grolnick & Pomerantz 2022 PDF](https://selfdeterminationtheory.org/wp-content/uploads/2025/09/2022_GrolnickPomerantz_ShouldParents.pdf)

### Inferences
- Push the *actionable improvement* message ("Noa is working on carrying in addition, try this 5-minute game") over the *score* message. Kraft & Rogers and IXL's goal-setting both support this.
- Frame every struggle alert as "productive struggle, normal on the path to mastery" (Haimovitz & Dweck). Pair it with a suggested autonomy-supportive script ("Ask her to show you how she solved it") and an explicit "don't" ("avoid taking over or quizzing").
- Weekly, short, specific messages give the best dose-response for the cost (READY4K, Kraft & Rogers). Daily score pushes risk pressure and notification fatigue. This is inference; no study here compared cadences directly.
- The biggest effects are for lower-performing children and less-engaged parents. So a "struggling" state should trigger *more* concrete parent tips, not a red warning.

### Gaps
- None of these RCTs studied a game app's dashboard directly. They cover school messages or texting programs, so transfer is inferred.
- I found no Israel-specific RCT on parent-information interventions.
- No study found here quantifies harm from parents seeing *too much* fine-grained data (as opposed to controlling *responses*).

---

## 3. Learning-analytics research: meaningful metrics, misleading dashboards, visual design, norms, sibling comparison

### Takeaway
Dashboards by themselves haven't reliably improved learning outcomes. Value comes from metrics grounded in learning theory (mastery, growth, error patterns, self-regulation and persistence) that lead to an action, not from activity counts. Show growth against the child's own past and against a curriculum or grade expectation. Don't put siblings next to each other.

### Cited Findings
- Kaliisa & Saqr's systematic review (38 studies): **no evidence that learning-analytics dashboards reliably improve learning outcomes**, although they're the most common LA intervention. — [Springer, Educ. & Inf. Tech. 2023](https://link.springer.com/article/10.1007/s10639-023-12401-4); [arXiv 2312.15042](https://arxiv.org/pdf/2312.15042)
- Paulsen & Lindsay's review: a shift toward learner-focused dashboards grounded in learning theory and using data beyond activity logs. — [UPM summary](https://iptc.upm.es/publications/have-learning-analytics-dashboards-lived-up-to-the-hype-a-systematic-review-of-impact-on-students-achievement-motivation-participation-and-attitude)
- Valle et al. ("Staying on target"): self-regulated learning is the main lens, and **affect and motivation are largely overlooked** as dashboard outcomes. — [Bohrium summary](https://bohrium.dp.tech/paper/arxiv/c3d882a9ce92ea5411e8c2a9fea0d3fca381cb3daf6f5e3e3a57bd61c9e87262)
- Schwendimann/Verbert et al. 2016 LAK review (346 candidate papers): calls for longitudinal studies and head-to-head design comparisons. — [EPFL Infoscience](https://infoscience.epfl.ch/record/216917?ln=en)
- Volume ≠ learning, from product docs: Prodigy says more questions answered doesn't necessarily raise Curriculum Progress, because only mastery counts. — [Prodigy help](https://prodigygame.zendesk.com/hc/en-us/articles/4407414190484)
- Effort ≠ minutes: Math Academy calibrates XP to *focused* minutes, and parents report 2–3 real minutes per XP. — [Math Academy](https://mathacademy.com/how-it-works); [WTM forum](https://forums.welltrainedmind.com/topic/733445-my-ongoing-review-of-mathacademycom-updated-62824)
- Error patterns: IXL Trouble Spots group misses by *item type* within a skill (3 misses) and show the actual items. That is a practical misconception detector. — [IXL blog](https://blog.ixl.com/?p=12993)
- Sibling comparison: comparing a child's achievement with a sibling's, and maternal differential treatment, predicted academic self-concept, with differential treatment mediating (Korean thesis). A 3-wave study of 384 adolescent sibling dyads found **reciprocal links between differential parenting, perceived partiality, and lower self-worth**. Large-sample Add Health work found weak direct effects of differential treatment on achievement. Evidence is mostly correlational. — [Ewha thesis](https://dspace.ewha.ac.kr/handle/2015.oak/203908); [PubMed 16402879](https://pubmed.ncbi.nlm.nih.gov/16402879/); [Add Health dissertation](https://addhealth.cpc.unc.edu/publications/bib/7476)

### Inferences (metric recommendations, not directly sourced)
- **Meaningful:** skill mastery state and its change over time; accuracy *trend* per skill (rolling, not lifetime); first-attempt accuracy (more diagnostic than eventual accuracy with hints); median response time on mastered skills as a *fluency* signal (school-age only, and only after accuracy is stable; for the 2.5-year-old, response time mostly measures motor and attention); error clusters by item type (misconceptions); persistence (retries after an error, finishing a session after a miss); help/hint use; productive-struggle ratio (items solved after at least 1 error).
- **Vanity or misleading:** total minutes, total stars or coins, streak length as a goal in itself, questions answered, lifetime accuracy (it rises when kids replay easy content), % of curriculum "played" when not mastered.
- **Visual forms:** a skill map (curriculum graph or tree, nodes coloured by mastery state: not started / practising / proficient / mastered / needs review); a week heatmap of *focused* sessions (a calendar, not a streak counter); sparkline accuracy trends per strand; a "growth since last week" delta.
- **Norms:** show placement against the *curriculum plan* and an approximate grade-level band ("on track for גן חובה / כיתה ב׳ expectations"). Avoid percentiles against other children at ages 2–10. For the 8.5-year-old (כיתה ג׳), align strands to the Israeli Ministry of Education math and Hebrew-language curricula. Meitzav (Israel's national school assessment) is given in grades 5 and 8, so it isn't directly relevant at these ages. That is an unverified recollection, so check it.
- **Sibling comparison:** one child per screen. No cross-child leaderboards. A family view shows only *family* collective goals (such as minutes read together) or a per-child "on track" status without scores.

### Gaps
- I found no peer-reviewed source that defines response time as a fluency indicator for young children within this search budget (it's well established in curriculum-based measurement, but uncited here).
- I didn't retrieve sources on help-seeking or "gaming the system" detection (Baker et al.).
- No source confirmed the Israeli grade-level expectations or meitzav grades and subjects. A separate check of the Ministry of Education (משרד החינוך / ראמ"ה) is needed.
- No evidence found on how visual formats (heatmap vs list) affect *parent* understanding specifically.

---

## 4. Actionable insights and parent controls

### Takeaway
Effective parent-facing programs turn data into a tiny, specific, doable action: a fact, a tip, and a growth message (READY4K), plus goals, reminders, and social reward (PACT). Products like IXL (goal-setting on trouble spots), Lingokids (everyday-life application), and Prodigy (monthly emailed report card) follow this pattern.

### Cited Findings
- READY4K's weekly text bundle: a fact, a tip (a concrete activity built into existing routines such as bath time or grocery shopping), and a growth/reinforcement message. It raised home involvement 0.15–0.29 SD. — [NBER W20659](https://www.nber.org/papers/W20659); [JHR](https://jhr.uwpress.org/content/54/3/537)
- PACT: goal-setting, reminders, and social rewards (not information) doubled parent–child reading time. — [NBER w21602](https://www.nber.org/papers/w21602.pdf)
- IXL tells parents to set SmartScore goals (80/90) on Trouble Spot skills. — [IXL blog](https://blog.ixl.com/?p=12836); [IXL blog: active role](https://blog.ixl.com/2022/03/02/how-to-take-an-active-role-in-your-childs-learning/)
- Prodigy emails parents a monthly Report Card. — [Prodigy blog](https://www.prodigygame.com/blog/report-card-tool)
- Lingokids publishes guidance on carrying in-app skills into everyday life. — [Lingokids blog](https://lingokids.com/blog/posts/how-to-apply-your-childs-in-app-progress-to-everyday-life)
- Specific improvement-focused messages beat general praise. — [Kraft & Rogers via Annenberg](https://annenberg.brown.edu/publications/underutilized-potential-teacher-parent-communication-evidence-field-experiment)

### Inferences
- **"Tonight" card:** one offline activity per child per week, tied to the current or struggling skill, built into a Hebrew-family routine (cooking: count and halve; Shabbat dinner: set the table, one-to-one correspondence; car ride: rhyme or first-sound games in Hebrew). It takes 5 minutes and needs no materials.
- **Conversation starters** with process-praise wording: "What was the trickiest puzzle today? How did you figure it out?" Avoid "Did you get them all right?"
- **Weekly summary** (push plus optional email on Friday afternoon, before Shabbat): per child, one win, one "working on", one "tonight" activity, and focused time. Optional monthly report card with month-over-month comparison, like Prodigy.
- **Celebration sharing** (with grandparents and family): parent-initiated only. Share an *achievement card* (a badge or artwork, "learned the letter מ"), never raw scores or accuracy. The child should approve, at least for the 8.5-year-old.
- **Goals and rewards:** parent sets a weekly *effort* goal (focused minutes or sessions) and optional mastery goals for the 8.5-year-old. Rewards are in-app cosmetics or a family activity, not money. Keep rewards tied to effort or process, not to accuracy.
- **Controls:** daily session limit per child; quiet hours (bedtime, Shabbat/holiday mode); subject weights (for example more Hebrew reading vs math for the 5.5-year-old); content approval and locks (skip ahead or lock content); toggle for English/Hebrew UI and audio; a parental gate on settings (adult challenge, as with Lingokids' birth-year gate).

### Gaps
- No experimental evidence found on grandparent sharing, or on reward management in kids' apps specifically.
- I didn't find evidence on the best push timing for Israeli families. The Friday timing is a cultural inference.

---

## 5. Privacy considerations for child performance data

### Takeaway
Treat performance data as sensitive personal data about minors. Collect the minimum, keep it on-device or in an encrypted family account, publish a written retention schedule, get verifiable parental consent, and never use it for ads or third-party sharing. The amended COPPA Rule (full compliance due 22 Apr 2026) and Israel's Privacy Protection Law Amendment 13 (in force 14 Aug 2025) are the relevant regimes.

### Cited Findings
- Amended COPPA Rule: effective 23 Jun 2025; **full compliance by 22 Apr 2026**. Operators need a **written data-retention policy** stating why children's data is collected, the business need for keeping it, and the deletion timeframe, published in the privacy notice. Indefinite retention is prohibited, but no fixed retention period is set. — [BBB National Programs](https://bbbprograms.org/media/insights/blog/coppa-amended); [Toy Association 2026](https://toyassociation.org/PressRoom2/News/2026-News/updated-coppa-rule-requirements-take-effect-april-22.aspx); [Mondaq / Ashurst Perkins Coie](https://www.ashurstperkinscoie.com/en/insights/ftc-finally-publishes-amended-coppa-rule-compliance-deadlines-set/); [Sheppard](https://www.sheppard.com/insights/blogs/ftc-coppa-updates-provide-new-protections-for-children)
- Israel Privacy Protection Law Amendment 13: approved 5 Aug 2024 and **in force 14 Aug 2025**. It adds GDPR-like definitions, stronger enforcement powers for the Privacy Protection Authority including financial sanctions, and a privacy protection officer requirement for some organizations. — [Pearl Cohen](https://www.pearlcohen.com/the-knesset-enacts-a-comprehensive-amendment-to-the-privacy-protection-law/); [Digital Policy Alert](https://digitalpolicyalert.org/event/32795-privacy-protection-bill-amendment-no-13-formerly-no-14-entered-into-force); [Goldfarb](https://www.goldfarb.com/amendment-13-to-the-israeli-privacy-protection-law/)
- Israeli privacy law does not specifically address children's privacy. An Israel Internet Association report recommends adding it, and that's a recommendation, not enacted law. — [search summary of law-firm commentary; see Lipa Meir](https://lipameir.co.il/en/?p=5313)

### Inferences
- For a family-built app: store per-item attempt data locally (Core Data/SwiftData with iOS Data Protection), sync through the parent's iCloud (CloudKit private database) rather than a third-party server, and keep no analytics SDKs in the kids' side. Apple's Kids Category rules (no third-party analytics or ads) apply if it's published. This is uncited here, so verify against the App Store Review Guidelines.
- Retention: keep raw item-level logs for about 90 days and roll them up into weekly per-skill aggregates kept until the parent deletes them. Provide "export" and "delete child profile".
- Shared grandparent cards should be rendered images with no account linkage and no scores.

### Gaps
- I didn't verify current Apple Kids Category or App Store guideline text, or GDPR-K applicability (Israel has EU adequacy).
- No Israeli PPA guidance specific to children's edtech was found.

---

## 6. Concrete recommended dashboard spec (screens, widgets, metric definitions, alert rules)

### Takeaway
A five-screen, parent-gated area: Family Home → Child Overview → Skill Map/Curriculum Plan → Skill Detail → Activity & Settings. There's also a weekly report and a small set of growth-framed alerts. Every widget answers "what should I do next?" and the design is age-tiered (2.5 / 5.5 / 8.5).

### Cited Findings (design anchors)
- Per-skill mastery scale with named thresholds, not percentages — [IXL](https://blog.ixl.com/2018/09/21/using-ixl-analytics-at-home/)
- Trouble Spots with exact missed items, using a 3-misses-per-item-type rule — [IXL](https://blog.ixl.com/?p=12993)
- Curriculum-progress-by-mastery, plus a monthly comparative report card — [Prodigy](https://www.prodigygame.com/blog/report-card-tool)
- Focused-effort unit (XP ≈ 1 focused minute) — [Math Academy](https://mathacademy.com/how-it-works)
- Minimum-data gate before reports appear (5 activities) — [Lingokids](https://help.lingokids.com/hc/en-us/articles/9569447484177-Progress-reports)
- Improvement-specific messages; fact, tip, and growth message; goals and reminders — [Kraft & Rogers](https://annenberg.brown.edu/publications/underutilized-potential-teacher-parent-communication-evidence-field-experiment); [READY4K](https://www.nber.org/papers/W20659); [PACT](https://www.nber.org/papers/w21602.pdf)

### Inferences — the spec

**Screens & widgets**
1. **Family Home** (behind the parent gate): one card per child (avatar, focused minutes this week vs goal as a ring, one-line status "On track / Working on X / New win!"). No scores side by side. "This week's 'tonight' ideas" carousel. Family goal (such as shared reading minutes).
2. **Child Overview:** (a) *This week*: focused minutes, sessions, days active (calendar heatmap); (b) *Wins*: newly mastered skills; (c) *Working on*: up to 3 skills in "practising" with a mini accuracy sparkline; (d) *Needs a hand*: trouble spots, framed as productive struggle; (e) *Tonight* card plus 2 conversation starters; (f) *Persistence badge* ("kept going after mistakes 12 times").
3. **Curriculum Plan / Skill Map:** strands per age. 2.5 years: colours and shapes, counting 1–5, Hebrew letter sounds, vocabulary, fine motor. 5.5 years: numbers to 20, early addition, phonemic awareness, letter recognition, first reading (ניקוד). 8.5 years: multiplication, fractions, place value, reading comprehension, English as a foreign language. Nodes are coloured by state, with the "current focus" highlighted and the "next up" queue. Parents can reorder or lock and approve upcoming units.
4. **Skill Detail:** mastery score history, first-try accuracy trend, median response time (8.5-year-old only, after proficiency), error clusters with example items replayed exactly as the child saw them, hint use, related offline activities.
5. **Activity log & Settings:** session list (date, duration, focused minutes, skills touched); controls for daily limit, quiet hours / Shabbat mode, subject weights, content approval, UI and audio language, notification preferences, data export and delete.
6. **Weekly report** (push plus email, Friday): per child one win, one working-on, one tonight activity, focused minutes vs goal, and a growth-framed sentence. Monthly report card: per-strand mastery change vs last month.

**Metric definitions** (age-adjusted)
- *Focused minute:* time with input activity in the last 30 s, excluding idle, menus, and video. Rapid-fire guessing doesn't count (for example 3 or more answers under 1.5 s that are wrong, on the 5.5- and 8.5-year-old tiers).
- *Mastery score (0–100, non-percentage):* rises with correct first attempts weighted by difficulty and falls with errors, more so at higher levels. States: Practising (<60), Proficient (60–79), Strong (80–89), Mastered (≥90 plus correct on a delayed review ≥3 days later). Decays to "needs review" after 21 days without correct practice.
- *First-try accuracy:* % of items correct on the first attempt, over a rolling window of the last 20 items per skill.
- *Fluency:* median response time on correct items for Proficient+ skills, shown as a trend (8.5-year-old only).
- *Persistence:* % of items with ≥1 error that were then solved, plus sessions continued after an error.
- *Help-seeking:* hints per 10 items, and whether hints were followed by success.
- *Curriculum coverage:* % of the plan's skills at Proficient+, which can only go down if the parent adds content (shown with an explanation).
- Minimum data: no skill score shown until ≥8 items. No child report until ≥5 activities.

**Alert rules** (max 2 pushes per child per week, plus the weekly report; none during quiet hours or Shabbat)
- *Win:* skill reaches Mastered → celebration card (shareable by the parent).
- *Trouble spot:* ≥3 errors on the same item type within a skill across ≥2 sessions → "Working on" card with a tip, not a red alert.
- *Stuck:* a skill stays in Practising for more than 5 sessions with first-try accuracy flat or falling → suggest an easier prerequisite and an offline activity.
- *Frustration:* ≥3 sessions in a week end within 2 min of an error streak, or rage-tap patterns → suggest shorter sessions or co-play.
- *Disengagement:* no focused session for 7 days → gentle nudge with a suggested 5-minute game, never guilt.
- *Over-use:* the daily limit is hit 5 days a week, or play happens during quiet hours → a notice to the parent only.
- *Review due:* ≥3 mastered skills decayed to "needs review" → the app prioritises them automatically; mentioned in the weekly report only.

### Gaps
- All thresholds are proposed defaults based on the product patterns above, not validated values. They should be tuned on real family data.
- I found no source on age-appropriate response-time norms or rage-tap detection for toddlers.
