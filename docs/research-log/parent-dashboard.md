# Research log — Parent area & dashboard (spec 10)

Date: 2026-11-02. Author: spec writer for `docs/spec/10-parent-area-dashboard.md`.
Purpose (F3): close the gaps left open by `research_notes/אפליקציית למידה משחקית לילדים/parent_dashboard_analytics.md`
and the round-3 notes before writing spec 10. About 22 web search/fetch calls.

Network note: `ideas.repec.org`, `cepa-aws.gse.stanford.edu` and `tipsbytext.stanford.edu` failed DNS
(outside the session allowlist). Findings from those sources come from search-engine extracts only.
To read the full documents, those hosts have to be allowed in the session's network settings.

---

## Q1. Parent-facing views of leading products (IXL, Khan, Prodigy, Zearn, Duolingo, Lingokids, ClassDojo)

Findings
- **IXL.** The Score Chart lists per-skill SmartScore, questions answered, time spent and last practice session. IXL suggests parents check it weekly or every two weeks. Usage Details shows time, questions and number of skills. Student Summary shows whether assigned SmartScore targets were met. A district guide asks for proficiency (SmartScore 80) on at least two skills a week from the study plan. — [IXL blog: fostering parent-teacher collaboration](https://blog.ixl.com/2020/04/30/fostering-parent-teacher-collaboration/); [IXL: using analytics at home](https://blog.ixl.com/2018/09/21/using-ixl-analytics-at-home/); [IXL blog p=8752](https://blog.ixl.com/?p=8752). I found no official description of an IXL weekly parent email.
- **Khan Academy.** A community post says skill progress was dropped from the weekly parent email after the "Khan Academy Reimagined" redesign. The Activity Overview breaks work down by course. Users still ask for the old **knowledge map** (a skill-tile grid that darkened with mastery), which was removed because it depended on the Google Maps API. — [Khan community: New Activity Report lacks Skill Levels](https://support.khanacademy.org/hc/de/community/posts/47404589461261/comments/47750016839821); [Khan community: What happened to the knowledge map?](https://support.khanacademy.org/hc/de/community/posts/360027982751/comments/360002975971); [Khan help: Activity tab](https://support.khanacademy.org/hc/bg/articles/360032987591)
- **Prodigy.** The monthly Report Card appears at the start of each month, and an email announces it. It shows skills worked on per grade, skills mastered per topic, strongest and weakest areas, and month-over-month comparison. A separate paid "Comparison Insights" adds questions, % correct, total time, **and comparison with peers**. Peer comparison is an anti-pattern for this family (C6). — [Prodigy: report card tool](https://www.prodigygame.com/blog/report-card-tool); [Prodigy help: Comparison Insights](https://prodigygame.zendesk.com/hc/en-us/articles/4421110038804-Comparison-Insights)
- **Zearn.** The family-facing material is only "Share progress with families", which I could not retrieve. Tower Alerts and the Pace Report are teacher-facing. — [Zearn help: Share progress with families](https://help.zearn.org/hc/en-us/articles/360021586433-Share-progress-with-families); [Zearn help: Progress Report](https://help.zearn.org/hc/en-us/articles/236279147)
- **Duolingo (ABC / family).** I found no official parent report for Duolingo ABC. Duolingo's weekly progress email (adult product) is praised in UX commentary for using personal data, and the same commentary suggests a social-comparison/FOMO angle. We reject that angle (C5, C6). — [uxdesign.cc review of Duolingo emails](https://uxdesign.cc/i-neglected-my-duolingo-for-20-days-after-signing-up-heres-every-email-they-sent-me-ecd0f32ce2ed); [opened.co Duolingo overview](https://opened.co/tools/duolingo)
- **ClassDojo Portfolios.** A teacher must approve student posts before families see them. After approval, only family members connected to that student see them, and families can add encouragement. This is a direct analogue for (a) the content-approval gate and (b) grandparent "Viewer" sharing with an approval step. — [ClassDojo help: Who can see portfolio posts](https://help.classdojo.com/hc/en-us/articles/360010117191-Who-Can-See-a-Student-s-Portfolio-Posts)

Decisions for spec 10
- Lead with "what to do this week". Mastery appears per skill as named levels, not percentages. Peer comparison, "weakest area" wording and lifetime % correct are cut.
- A "knowledge map"-style **tile grid** (Khan's old map, Mastery Grids) is the main skill-map view. The dependency graph is an optional iPad/web view.
- The monthly summary ("המבט החודשי") is announced inside the Friday weekly report. It is not a separate message, to keep message volume down (see Q3).
- Grandparents get celebration cards. A parent approves each one, and K1 also approves. This follows the ClassDojo approve-then-share pattern.

## Q2. Skill-map visualizations

Findings
- **Mastery Grids** (Loboda, Guerra, Hosseini & Brusilovsky, EC-TEL 2014): a grid where topic order runs along one axis and the colour density of a cell encodes knowledge level, with a finer concept-level bar chart underneath. In a large classroom study, open social student modelling helped lower-prior-knowledge students most. Its "social" comparison grids are exactly what we must drop (siblings). — [Mastery Grids chapter](https://www.springerprofessional.de/en/mastery-grids-an-open-source-social-educational-progress-visuali/2271048); [Brusilovsky et al. 2015 IEEE TLT (ADL mirror)](https://adlnet.gov/assets/uploads/2015%20---%20(IEEE%20Transitions)%20Brusilovsky%20et%20al.%20-%20Open%20Social%20Student%20Model.pdf)
- **Khan's knowledge map** (historical): skill tiles that darken as mastery challenges are passed. Users still miss it. — links above.

Decision: one tile per skill, ordered along the curriculum sequence in rows per strand. Fill uses a single-hue sequential scale plus a shape glyph (◌ ◔ ◑ ◕ ●) for colour-blind safety. Review-ready shows a sparkle overlay, never a fading or wilting state (D19-style). No red for child performance.

## Q3. Weekly parent-message design (Kraft & Rogers, READY4K/Tips-by-Text, dose)

Findings
- **Kraft & Rogers 2015** (*Econ. of Ed. Review* 47:49–63; N=435): weekly one-sentence individualized teacher→parent messages. A secondary source says messages about **what to improve** beat exclusively positive messages. Cost was just over $13 per student per course. — [Annenberg](https://annenberg.brown.edu/publications/underutilized-potential-teacher-parent-communication-evidence-field-experiment); [Chicago Policy Review](https://chicagopolicyreview.org/2016/01/06/messages-to-parents-can-help-low-performing-students/)
- **READY4K / Tips-by-Text structure:** FACT (Monday, builds buy-in), TIP (Wednesday, a concrete activity inside an existing routine), GROWTH (Friday, encouragement plus an extension of the tip). Real examples: "When you're bathing your child, point out the letters on shampoo bottles. Ask your child to name them & the sounds they make."; "Say two words to your child that start with the same sound, like happy & healthy."; grocery store: name all the fruits and vegetables, then rhymes (egg/leg). Children gained about 2–3 extra months of literacy. — [Stanford news](https://ed.stanford.edu/news/stanford-tips-text-program-helps-boost-literacy-preschoolers-study-finds); [NC DNCR](https://dncr.nc.gov/press-release/texts-prek-parents-build-reading-skills-children); [Stanford TBT PDF](https://tipsbytext.stanford.edu/sites/g/files/sbiybj27996/files/media/file/tbt_one_step_at_time.pdf)
- **Personalization works** (Doss, Fahle, Loeb & York, *JHR* 54(3):567–603): differentiated texts, which told parents how well their child knew a specific skill and adapted the tips, beat general literacy texts. Those children were **63% more likely to read at a higher level**. Effects were driven by children far from average, so the content mattered, not just the reminder. — [Stanford SCALE](https://scale.stanford.edu/publications/more-just-nudge-supporting-kindergarten-parents-differentiated-and-personalized-text); [ParentPowered summary](https://parentpowered.com/blog/research/more-than-just-a-nudge-supporting-kindergarten-parents-with-differentiated-and-personalized-text-messages)
- **Dose and opt-out** ("Too little or too much"; "It's too annoying"): 5 texts/week → 8% opt-out vs 3 texts/week → 5%. More texts and **more complex** texts raise opt-out. Adding context and encouragement to activities **reduces** opt-out. ParentPowered today sends 3 texts a week tailored to the child's age. — [Annenberg brief: Too little or too much](https://annenberg.brown.edu/sites/default/files/TBT_Brief_Too%20Little_or_Too_Much.pdf); [EdWeek](https://www.edweek.org/leadership/more-isnt-always-better-in-texting-preschool-parents-study-finds/2018/07); [CEPA: It's too annoying](https://cepa.stanford.edu/node/2133); [ParentPowered FAQ](https://s3.parentpowered.com/gelf/GELFParentPoweredFAQEnglish.pdf)

Decisions for spec 10
- Weekly report = for each child: one win, one focus (specific, improvement-oriented, *with what the app is doing about it*), one "tonight" activity inside a routine, one conversation starter, and one growth sentence. That is a FACT→TIP→GROWTH compression into one Friday message, because we already have in-app context.
- The **total parent message budget is ≤3 per week per parent**: the weekly report plus at most 2 alerts, bundled across children. The ≤2/child/week cap applies before bundling. A daily cap of 1 push per parent per day.
- Hard length budget: ≤70 Hebrew words per child block in the email, push ≤110 characters. The skill state is always included (personalization), never a raw score.

## Q4. Hebrew/Israeli parent-communication tone and context

Findings
- Israeli parents are saturated by class WhatsApp groups. 5787 (2026) back-to-school guides warn about irrelevant messages, and about **night and weekend (Shabbat) messages becoming a burden**. The Ministry has moved teacher–parent messaging to official channels (Mashov) and away from direct WhatsApp. — [Kipa: 5 rules for parent groups (Aug 2026)](https://www.kipa.co.il/לייף-סטייל/1230615-0/); [JNS Hebrew: WhatsApp in schools](https://www.jns.org/he/article/000001a0-a8e3-d8ef-abb5-f9efc6d00000); [Timeout: ten commandments of class WhatsApp](https://timeout.co.il/%d7%a2%d7%a9%d7%a8%d7%aa-%d7%94%d7%93%d7%99%d7%91%d7%a8%d7%95%d7%aa-%d7%9c%d7%95%d7%95%d7%90%d7%98%d7%a1%d7%90%d7%a4-%d7%94%d7%9b%d7%99%d7%aa%d7%aa%d7%99/)
- I found no research on the register of Hebrew parent messages from apps. The tone rules in spec 10 are practitioner judgment: short, warm, direct, informal "אנחנו", no exclamation inflation, and impersonal/infinitive address to avoid gendering the parent.

Decisions: parent copy uses infinitive/impersonal/plural constructions ("אפשר לנסות", "שווה לשאול"). No messages on Shabbat or holidays, or during parent quiet hours. Everything held is delivered the next weekday at 07:30. Friday send must land **before** candle lighting.

## Q5. Channels: push, email (Resend), Telegram

Findings
- **Expo Go:** remote push from `expo-notifications` is unavailable in Expo Go from SDK 53 (the docs disagree on whether this covers iOS or Android only). A development build is the supported path. Local notifications still work. — [Expo notifications docs v53](https://docs.expo.dev/versions/v53.0.0/sdk/notifications/); [latest](https://docs.expo.dev/versions/latest/sdk/notifications)
  → In the E1a prototype stage (Expo Go), assume **no remote push**. Email plus an optional Telegram bot plus the in-app inbox carry the reports.
- **Telegram Bot API HTML mode** supports only b/strong, i/em, u, s, tg-spoiler, a, code, pre, blockquote (and tg-emoji under conditions). `<`, `>`, `&` must be escaped. Per secondary sources, whitespace and newlines are handled loosely. The bot cannot message first, so each parent must /start it. I could not verify the 4096-char message and 1024-char caption limits from this session, so treat them as assumed limits and enforce them in code. — [grammY ParseMode ref](https://grammy.dev/ref/types/ParseMode); [teloxide ParseMode](https://docs.rs/teloxide/latest/teloxide/types/enum.ParseMode.html); backend note §5
- **RTL HTML email:** Gmail may strip `<style>` blocks and ignore `dir` in places. Practice is to put `dir="rtl"` and `lang="he"` on `<html>`, wrap content in `<div dir="rtl">`, and use **inline** `direction:rtl; text-align:right` on every block, then test in Gmail iOS and Apple Mail. — [Ghost PR 27357 (RTL newsletters)](https://github.com/TryGhost/Ghost/pull/27357); [Marketo community thread](https://nation.marketo.com/t5/product-discussions/how-do-i-code-for-hebrew-emails/m-p/197310); [tempmail RTL Gmail note](https://tempmail.us.com/en/rtl/resolving-rtl-text-alignment-issues-in-gmail-html-emails)
- **Shabbat timing:** `@hebcal/core` computes candle-lighting and havdalah per `Location`. Default candle lighting is 18 min before sunset (40 min in Jerusalem, 30 in Haifa), and havdalah uses tzeit. It can be overridden. Version numbers conflict across sources (5.9.x vs 6.5.x), so pin at build time. — [npm @hebcal/core](https://npmjs.com/package/@hebcal/core); [HebrewCalendar API](https://hebcal.github.io/api/core/classes/HebrewCalendar.html)

## Q6. Data-visualization practice: sparklines, small multiples, RTL charts, parent visualization literacy

Findings
- **Sparklines:** "small, intense, simple, word-sized graphics" with typographic resolution, placed in sentences and tables. Tufte gives little guidance on placement; Goffin et al. (TVCG 2014) studied word-scale placement. — [Tufte notebook](https://www.edwardtufte.com/notebook/sparkline-theory-and-practice-edward-tufte/); [Goffin et al.](https://vis.cs.ucdavis.edu/vis2014papers/TVCG/papers/2291_20tvcg12-goffin-2346435.pdf)
- **Small multiples** need a shared axis and scale so that only the subset differs. — [Georgia Tech Tufte notes](https://faculty.cc.gatech.edu/~stasko/4460/Notes/tufte.pdf)
- **RTL and charts:** current Material and Wikimedia Codex guidance says **do not mirror charts/graphs** when mirroring could change interpretation, and do not mirror time-progress elements. Older Material guidance mirrored time. — [Material bidirectionality](https://m2.material.io/go/design-bidirectionality); [Codex bidirectionality](https://doc.wikimedia.org/codex/latest/style-guide/bidirectionality.html)
- **Dashboard literacy:** teachers with lower visualization literacy relied on single charts. Some guidance helped and too much backfired (JLA 2025). Simpler visuals were liked but led to lower recall of the main takeaway (SFU thesis). No parent-specific study was found. — [JLA 8471](https://learning-analytics.info/index.php/JLA/article/view/8471); [SFU thesis](https://summit.sfu.ca/_flysystem/fedora/2022-08/input_data/21422/etd21348.pdf)

Decisions
- Evidence is split: current Material/Codex say do not mirror charts, while Apple HIG (adopted by `09-ux-ui-design-system.md` RTL rule 10) mirrors time (old on the right). Spec 10 therefore uses a single token `chart.timeDirection`, default `'rtl'` per 09, and the parent decides at GATE-UX-3. The `'ltr'` option stays available.
- Calendars are always mirrored (Sunday on the right, as in Israeli calendars).
- Every chart has a one-sentence Hebrew "takeaway" headline above it. That is the guidance layer, and it states the gist so the simplicity trade-off does not cost recall.
- Small multiples share a 0–100% y-axis and show the **target band** shaded. In an adaptive system accuracy is *held* near the target band by design, so the band explains why "flat ≈ healthy".

## Open gaps (not closable from this session)
- Full text of Kraft & Rogers (exact effect of improvement vs positive messages) and of Doss et al. (effect sizes beyond the headline 63%).
- Telegram length limits (assumed 4096/1024, enforced in code).
- Whether iOS Expo Go still delivers remote push under SDK 58. Assume no, and verify on device.
- Israeli 5787 school vacation dates (load from the Ministry calendar at build time, with parent confirmation).
- No study on parent understanding of skill-grid vs list views. Validate with the parent (A5) at the design checkpoint.
