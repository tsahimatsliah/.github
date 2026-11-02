# Learning Through Movement (Embodied Learning) for Ages 2-10 in an iPhone/iPad Expo/React Native Game

Context: Israeli family, girl 8.5, boy 5.5, toddler 2.5. Target stack: Expo SDK 58 / React Native, iOS. Research date: Nov 2026. 16 search calls; most findings come from abstracts, search snippets and official doc index pages, not full papers. Treat exact numbers as "verify before quoting in product copy".

## 1. Evidence: does movement-based / embodied learning work for 2-10 year olds?

### Takeaway
The evidence is real but modest. Movement that is *integrated with the content* (walk the number line, act out the word) does better than either sitting still or doing unrelated exercise. Physically active lessons have a small effect on learning overall (d≈0.36) and a large effect on in-lesson outcomes (d≈0.81). Short exercise bursts improve attention (g≈0.43). Exergames give small-to-moderate executive-function gains but do not clearly beat ordinary exercise. Design rule: make the movement *carry meaning* (it maps to number, letter or word), not just a reward break.

### Cited Findings

**Physically active lessons / general PA → learning**
- Norris et al. 2020 (Br J Sports Med) meta-analysis of physically active lessons, 42 studies. It found a large significant effect on lesson-time educational outcomes (d=0.81, 95% CI 0.47-1.14, k=7) and a small significant effect on overall educational outcomes (d=0.36, 95% CI 0.09-0.63, k=25). Lesson-time PA rose a lot (d=2.33) and overall PA rose a little (d=0.32). There was no significant effect on cognition (k=3) or health (k=3). 25 of 42 studies had high risk of bias in ≥2 domains. Most studies were in preschool or elementary settings — [Norris et al. post-print record](https://katalog.lib.cas.cz/KNAV/EdsRecord/edsair,edsair.doi.dedup.....5348f4ba8c941ec1503853f3048f0751); [UCL Discovery](https://discovery-pp.ucl.ac.uk/id/eprint/10083551)
  - Discrepancy: the US Community Preventive Services Task Force summary lists the overall educational k as 23 (not 25) and the health effect as 0.03. The author's seminar slides show −0.03 — [Community Guide TFFRS](https://www.thecommunityguide.org/pages/tffrs-physical-activity-classroom-based-physically-active-lesson-interventions.html); [Norris NOPREN seminar slides](https://nopren.ucsf.edu/sites/g/files/tkssra5936/f/Dec-19_Emma%20Norris%20NOPREN%20Seminar.pdf)
- EEF Teaching & Learning Toolkit, "Physical activity" strand: small positive impact on attainment (+2 months), very low cost, based on 136 studies. The page is internally inconsistent on evidence security: it calls the evidence "high" in one place and says it "lost a padlock" because many studies weren't independently evaluated in another. EEF announced a comprehensive update to this strand, and the +2 figure may since have changed — [EEF Physical activity](https://educationendowmentfoundation.org.uk/education-evidence/teaching-learning-toolkit/physical-activity); [EEF toolkit updates](https://educationendowmentfoundation.org.uk/news/updates-ten-strands-toolkit)
- EEF also has a separate *Early Years* toolkit strand, "Physical development approaches", with a technical appendix — [EEF Early Years physical development](https://educationendowmentfoundation.org.uk/early-years/toolkit/physical-development-approaches/technical-appendix)

**Acute exercise → attention / executive function (EF)**
- A 2018 systematic review and meta-analysis of preadolescents (6-12) found that acute (single-bout) PA improves attention (g=0.43, 95% CI 0.09-0.77, 6 studies). Longer-term PA programs improve executive functions (g=0.24), and the largest effects are expected from regular PA over several weeks — [PubMed 29054748](https://pubmed.ncbi.nlm.nih.gov/29054748/)
- Secondary summary of active-break literature: small-to-moderate attention effect (d≈0.43). Evidence for academic outcomes is weaker: of 4 academic studies, only 1 found a significant effect, in maths — [Multilit "Brain breaks" note](https://info.multilit.com/hubfs/Nomanis%20Notes_%2317_Brain%20breaks-1.pdf?hsLang=en)
- Egger et al. 2018, "The effect of acute cognitively engaging physical activity breaks on children's executive functions: Too much of a good thing?", raises the possibility that combining high cognitive and high physical load can backfire. I did not retrieve its results — [UGR record](https://produccioncientifica.ugr.es/documentos/68cc49f7a9fb570f27aaa90b?lang=de)

**Embodied math: number line, spatial-numerical movement**
- Fischer et al. 2011 (Psychonomic Bulletin & Review): kindergartners compared number magnitudes on a digital dance mat by stepping left or right. This spatial-numerical training beat a non-spatial control on number-line estimation and on a subtest of a standardized math achievement battery — [Konstanz KOPS record](https://kops.uni-konstanz.de/handle/123456789/45026)
- Link et al. 2013, "Walk the number line" (Trends in Neuroscience & Education): children walked to positions on a floor number line. They showed larger training effects than with an identical number-line training that had no task-specific whole-body experience — [Konstanz KOPS](https://kops.uni-konstanz.de/entities/publication/878bea3c-a4aa-4c9f-afe0-7ddbfdf86ca8)
- Fischer, Moeller, Huber, Cress & Nuerk 2015 (Int J Serious Games): pilot study (N=27) where children moved left or right in front of an interactive whiteboard to do number-line estimation. Full-body movement helped them maintain performance in multi-digit addition, compared with two control conditions — [IJSG PDF](https://journal.seriousgamessociety.org/index.php/IJSG/article/download/93/pdf_29/601); [DOAJ](https://doaj.org/article/bb559fe12cc24f4abc92cbd4c82a217f)
- Review-level framing of embodied numerical training (Educational Psychology Review, 2015) — [Springer s10648-015-9324-4](https://link.springer.com/doi/10.1007/s10648-015-9324-4)

**Gesture and math (Goldin-Meadow lab)**
- Cook & Goldin-Meadow 2006 (49 third and fourth graders): when instruction included gesture, children produced their own gestures, and those gestures were linked to learning. Adding gesture to speech made instruction more effective — [PMC2265003](https://pmc.ncbi.nlm.nih.gov/articles/PMC2265003)
- Cook, Mitchell & Goldin-Meadow 2008, "Gesturing makes learning last" (Cognition): children told to gesture while learning math equivalence kept their learning better. Speaking without gesturing did not consolidate learning — [UChicago news](https://news.uchicago.edu/story/gesturing-hands-powerful-tool-childrens-math-learning); [NSF PAR](https://par.nsf.gov/servlets/purl/10025977)
- Novack et al. 2014 (Psychological Science, third graders): abstract gesture generalized best, concrete gesture came next, and physical action on objects was least effective. Acting on objects gave shallower understanding — [PMC3984351](https://pmc.ncbi.nlm.nih.gov/articles/PMC3984351); [ScienceDaily](https://sciencedaily.com/releases/2014/03/140311124251.htm)

**Embodied vocabulary / foreign language (relevant to English for Hebrew speakers)**
- Mavilidi et al. 2015 (Educational Psychology Review): 111 preschoolers from 15 childcare centres learned 14 Italian words over 4 weeks. Of four conditions (integrated exercise, non-integrated exercise, seated gesturing, verbal repetition), integrated exercise gave the best learning — [EUR RePub 85532](https://repub.eur.nl/pub/85532); [SCU portal](https://researchportal.scu.edu.au/esploro/outputs/journalArticle/Preschool-Childrens-Foreign-Language-Vocabulary-Learning/991013335328502368)
- Evidence for Learning summary: the integrated-PA preschool group had the best free recall, still visible up to 6 weeks later — [Evidence for Learning, Physical development approaches](https://evidenceforlearning.org.au/education-evidence/australasian-research-summaries/early-childhood-education-toolkit/physical-development-approaches)
- Toumpaniari, Loyens, Mavilidi & Paas 2015: embodying words through enactment gestures plus physical activity was preferred by children and gave higher learning than gestures alone or no movement — [EUR RePub 84596 PDF](https://repub.eur.nl/pub/84596/REPUB_84596_OA.pdf)
- Schmidt et al. ran a follow-up with 8-10 year olds (20 words, 2 weeks; embodied vs task-irrelevant PA vs sedentary). Results were not visible in my sources — [UOW record](https://ro.uow.edu.au/sspapers/1850)

**Exergames**
- Chen et al. 2023 (Archives of Public Health), 11 studies, 508 children: exergaming improved cognitive flexibility (SMD 0.34), inhibitory control (SMD 0.57) and working memory (SMD 0.26). Publication bias was noted — [PMC10571260](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10571260/)
- Wu et al. 2023 (JMIR Serious Games), 8 RCTs, 435 children and adolescents: *commercial* exergames showed no significant EF advantage over conventional exercise. Only commercial titles were included — [PMC10623224](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10623224/); [JMIR PDF](https://games.jmir.org/2023/1/e42697/PDF)
- PLOS ONE 2024 review of 37 RCTs on exergaming, EF and motor ability. It notes one study with no difference vs traditional sport in healthy children — [PMC11379181](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC11379181/)
- A 2026 meta-analysis of exergaming and EF in children with ADHD exists. I saw only its abstract, not the pooled effects — [PMC12818307](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12818307/)

**GoNoodle-style movement videos**
- 2024 simulated-classroom study (n=71): researcher-designed PAAC-3 breaks reached ~5.7-5.8 METs vs 3.3 METs for GoNoodle videos. Some GoNoodle videos can still reach moderate-to-vigorous activity — [Boise State IJPAH](https://scholarworks.boisestate.edu/ijpah/vol4/iss1/4)
- 2022 real-world off-the-shelf use (non-randomized pre/post): GoNoodle was the only product whose daily use rose. For GoNoodle, changes in sedentary time, light PA and MVPA were not significant. Authors: off-the-shelf products need extra support to integrate — [Augusta/Elsevier Pure](https://augusta.elsevierpure.com/en/publications/classroom-teachers-off-the-shelf-use-of-movement-integration-prod)
- A small 2023 thesis found students did *better* on the d2 attention test after a generic PA video than after GoNoodle — [WCU thesis](https://wcu.hykuup.com/concern/scholarly_works/6b09ebb4-b61e-4238-9274-af914c4e1337)
- Trial NCT03394846 (GoNoodle vs Take10 vs ABC for Fitness, 60 teachers) posted results in Jan 2025. I did not retrieve them — [ClinicalTrials protocol](https://cdn.clinicaltrials.gov/large-docs/46/NCT03394846/Prot_SAP_000.pdf)

**Screen time + activity guidance (toddler 2.5, boy 5.5)**
- WHO 2019 guidelines for under-5s: replace restrained or sedentary screen time with active play, and protect sleep. For 2-year-olds, sedentary screen time should be ≤1 h/day ("less is better"); same for 3-4 year olds. Children should not be restrained (stroller, high chair) for more than 1 h at a time. Ages 1-2: ≥180 min/day of PA at any intensity. Sleep: 11-14 h (ages 1-2), 10-13 h (ages 3-4). For sedentary time, the WHO recommends reading, storytelling, singing and puzzles *with a caregiver* — [WHO guideline page](https://www.who.int/publications/i/item/9789241550536); [PAHO copy](https://www.paho.org/en/documents/guidelines-physical-activity-sedentary-behaviour-and-sleep-children-under-5-years-age)
- Secondary sources state 3-4 year olds need ≥180 min PA, of which ≥60 min is moderate-to-vigorous. Not verified against the WHO PDF in this session — [Active for Life summary](https://activeforlife.com/world-health-guidelines/)

### Inferences
- WHO's wording is "*sedentary* screen time". A movement game where the toddler is standing and jumping is arguably not sedentary, but it is still screen exposure. The safest framing is a co-play, caregiver-present activity with short sessions (a few minutes) inside the ≤1 h budget.
- The strongest design pattern from the literature is *task-relevant* movement: the body position or action encodes the answer (number magnitude → left/right, word → enactment). Exercise used only as a "reward break" helps attention but adds little to learning.
- Novack 2014 suggests a progression for the 8.5-year-old: move from concrete actions to more abstract gestures, such as tracing "balance" with arms for equations, so knowledge generalizes.
- The exergame and GoNoodle data say generic dance videos produce modest intensity and modest cognitive benefit. Value comes from coupling movement to the curriculum and from the child's own active responding, not from passive "follow the video".
- The Egger 2018 "too much of a good thing" question suggests not combining maximal intensity with hard cognitive problems for the 5.5-year-old. Use moderate movement with easy retrieval, or vigorous bursts followed by a calm task.

### Gaps
- No full-text check of Link 2013 sample size or effect sizes.
- Finger gnosis and counting (Noël; Gracia-Bafalluy & Noël 2008; Fischer's finger-counting work) was not searched. No citable findings here.
- Letter formation with the body (whole-body letter making, sky writing) and handwriting embodiment: not searched. TPR (Asher) original evidence: not searched beyond the Mavilidi line.
- Donnelly & Lambourne (TAKE 10! / PAAC classroom studies): only indirectly referenced (PAAC-3 in the GoNoodle study). Primary results were not retrieved.
- Kinect, Just Dance and Ring Fit studies with kids: not specifically retrieved beyond the exergame meta-analyses.
- No Hebrew-specific embodied literacy research found (not searched).
- WHO 5-17 guideline (60 min/day MVPA average, limit recreational screen time) was not re-verified in this session.
- Current (post-update) EEF physical activity figure unknown.

## 2. Mechanics: safe, fun, age-appropriate movement games for this family

### Takeaway
Map each movement to meaning: position = number, pose = letter, action = verb. Keep bursts of 1-3 minutes alternating with calm tasks. Tier by age: imitation and songs for the toddler, counting jumps and simple choices for the 5.5-year-old, timed agility and math challenges plus co-op "missions" for the 8.5-year-old. The research supports the *principles* (integrated movement, gesture, short bursts). The specific mechanics below are design inferences built on them.

### Cited Findings
- Stepping left/right to judge number magnitude (dance-mat style) improved kindergartners' number-line estimation. This directly supports a "jump left if smaller, right if bigger" mechanic — [Fischer et al. 2011 / KOPS](https://kops.uni-konstanz.de/handle/123456789/45026)
- Walking to a position on a floor number line beat the same training without full-body movement. This supports a "hop to where 7 lives on the hallway number line" mechanic — [Link et al. 2013 / KOPS](https://kops.uni-konstanz.de/entities/publication/878bea3c-a4aa-4c9f-afe0-7ddbfdf86ca8)
- Moving left or right in front of a large screen (whiteboard) was feasible for number-line estimation in a pilot (N=27). This supports camera-tracked lateral movement in front of an iPad — [Fischer et al. 2015 IJSG](https://journal.seriousgamessociety.org/index.php/IJSG/article/download/93/pdf_29/601)
- Enacting foreign words with whole-body actions (preschool, Italian words) beat seated gesture and verbal repetition, with recall up to 6 weeks. This supports "Simon says" / act-out-the-English-verb for the 5.5 and 8.5 year olds — [Mavilidi 2015](https://repub.eur.nl/pub/85532); [Evidence for Learning](https://evidenceforlearning.org.au/education-evidence/australasian-research-summaries/early-childhood-education-toolkit/physical-development-approaches)
- Asking children to gesture while learning makes learning last, and abstract gesture generalizes better than acting on objects — [Cook/Mitchell/Goldin-Meadow via UChicago](https://news.uchicago.edu/story/gesturing-hands-powerful-tool-childrens-math-learning); [Novack 2014 PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC3984351)
- Single short PA bouts improve attention in 6-12 year olds (g=0.43). This supports brief movement bursts before or between focused tasks — [PubMed 29054748](https://pubmed.ncbi.nlm.nih.gov/29054748/)
- WHO under-5 guidance favours interactive play and caregiver-led singing and storytelling for quiet time. This supports song and imitation modes for the toddler with a parent present — [WHO](https://www.who.int/publications/i/item/9789241550536)

### Inferences (design proposals, not evidence-tested in this exact form)
- **Toddler (2.5):** "Copy me" imitation (arms up, clap, stomp, animal walks) with a song. No scoring and no failure states. The parent is the co-player. Sessions 2-5 min. Use only simple sensing (sound, big motion) or none, and let the parent tap "done". Hebrew songs with actions, such as body-part songs, as content.
- **Boy (5.5):**
  - "Jump the count": jump N times and the camera or accelerometer counts aloud.
  - Left/right magnitude jumps.
  - Freeze dance that stops on a letter and asks "find something that starts with ב".
  - Clap syllables of Hebrew words (ra-ke-vet = 3 claps), detected by audio onset.
  - Show N fingers (hand pose) for numbers 1-10.
  - Home scavenger hunt: "photograph something red / round / starting with M". Use on-device classification, or just a parent confirm.
- **Girl (8.5):**
  - Timed agility math: answer zones left, right or up; lean or step to choose.
  - Floor number line with tape: hop to fractions, negatives, or ×-table multiples.
  - English verbs via "Simon says", with tense twists ("Simon says: jumped!" = do nothing).
  - Gesture-based equation balance, using arms as a balance scale.
  - Co-op family challenges: together reach 100 jumps, where each sibling's jumps count toward a shared goal.
  - "Missions" between sessions: count stairs, find 5 rectangles.
- **Calm-down / regulation:** 30-60 s breathing (inflate a balloon on screen in time with breath) or a yoga pose (tree, cat) after vigorous bursts, before any harder cognitive task.
- **Session pacing:** a 1-3 min movement burst, then a 2-5 min calm or learning task, repeated. Whole session ~10-20 min for the older kids. For the toddler, a few minutes total, inside the WHO ≤1 h screen budget.
- **Seated or inclusive mode:** every mechanic needs a seated equivalent (arm-only gestures, head tilt, finger counting, tap fallback) for injury, limited space, or sick days.

### Gaps
- No studies found on specific in-app mechanics like "freeze dance with letters" or "clap syllables" for Hebrew. Rhythm and syllable clapping evidence (phonological awareness) was not searched.
- No direct evidence found on optimal burst length (1 vs 3 min) for 5-8 year olds in app contexts.

## 3. Technology: sensing movement on iOS from Expo / React Native

### Takeaway
There are three tiers:
1. **expo-sensors**: accelerometer, gyroscope, DeviceMotion, Pedometer. Works in Expo Go, needs an `NSMotionUsageDescription` string. Good for "hold the phone and jump/shake/tilt", but you cannot use it when the iPad is on a stand.
2. **Camera pose or hand tracking** with react-native-vision-camera frame processors. Needs a dev build. v5 (Nitro-based, released Apr 2026) changed the plugin API, and many community pose plugins still target v4.
3. **Native Apple Vision** (`DetectHumanBodyPoseRequest` / `DetectHumanHandPoseRequest`, iOS 18+ Swift API; `VN…` legacy API) or ARKit body tracking (A12+), through a custom Expo native module.

All camera processing should run on-device with no frames stored.

### Cited Findings

**expo-sensors**
- expo-sensors provides Accelerometer, Gyroscope, DeviceMotion (all platforms) and Pedometer (Android and iOS). It is included in Expo Go. On iOS the motion permission is `NSMotionUsageDescription`, which you set via the config plugin's `motionPermission` option (or set to false to disable). Projects without CNG must configure it manually. The 200 Hz cap applies to Android 12+, not iOS — [Expo sensors docs (latest)](https://docs.expo.dev/versions/latest/sdk/sensors.md); [v57 docs](https://docs.expo.dev/versions/v57.0.0/sdk/sensors.md)

**react-native-vision-camera (VisionCamera)**
- VisionCamera V5 was announced Apr 16, 2026. It was fully rewritten on Nitro Modules, and adds a Constraints API, in-memory photos, depth streaming, RAW, multi-cam and modular plugins — [Margelo blog: What's new in V5](https://margelo.com/blog/whats-new-in-visioncamera-v5); [V5 guide](https://visioncamera4.margelo.com/docs/guides/vision-camera-v5)
- In v5, native frame processor plugins are typed Nitro HybridObjects. The old `FrameProcessorPlugin` base class, the `VISION_EXPORT_SWIFT_FRAME_PROCESSOR` macro and `VisionCameraProxy.addFrameProcessorPlugin` are gone. Frame processors need `react-native-vision-camera-worklets` plus Software Mansion's `react-native-worklets` (not `-core`) — [Margelo react-native-skills: VisionCamera skill](https://skills.sh/margelo/react-native-skills/react-native-vision-camera)
- Release cadence: 5.1.0 (2026-07-01; notes include "Frame Output is now Frame Processors"), 5.2.0 (2026-07-27), 5.2.1 (2026-07-31; Nitro 0.36.4 ANR fix). Hook naming (e.g., `useFrameOutput`) appears to be in flux, so check current docs — [newreleases 5.1.0](https://newreleases.io/project/yarn/react-native-vision-camera/release/5.1.0); [5.2.1](https://newreleases.io/project/yarn/react-native-vision-camera/release/5.2.1)
- Ecosystem plugins are being re-released for v5 as breaking changes (e.g., vision-camera-ocr-plus v2) — [DEV Community](https://dev.to/jamenamcinteer/react-native-vision-camera-ocr-plus-v2-released-with-vision-camera-v5-nitro-modules-compatibility-5c32)
- react-native-executorch (Software Mansion) documents real-time camera integration as another route for on-device ML on frames — [RN ExecuTorch camera integration](https://docs.swmansion.com/react-native-executorch/docs/extensions/camera-integration)

**Pose plugins (community)**
- Google ML Kit-based options:
  - `react-native-mlkit-pose-detection`: iOS only, via VisionCamera frame processors — [GitHub](https://github.com/swittk/react-native-mlkit-pose-detection)
  - `react-native-vision-camera-v3-pose-detection`: targets VC 3.9 — [GitHub](https://github.com/gev2002/react-native-vision-camera-v3-pose-detection)
  - `react-native-esanusi-sensor-pose`: VC v4, 33 landmarks, iOS ≥15.5 — [pkgstats](https://www.pkgstats.com/pkg:react-native-esanusi-sensor-pose)
- MediaPipe-based options:
  - `react-native-mediapipe-pose-plugin`: Swift and Kotlin, 33 3D landmarks with visibility. Tested with Xcode 26, RN 0.81.5, VC ^4.7.3, MediaPipeTasksVision ~>0.10.14. You add `pose_landmarker_full.task` yourself. The "production-validated" claim is self-reported — [pkgstats](https://www.pkgstats.com/pkg:react-native-mediapipe-pose-plugin)
  - `react-native-mediapipe-posedetection` v0.4.0: VC ^4, New Arch, worklets-core, GPU, ~357 weekly downloads — [socket.dev](https://socket.dev/npm/package/react-native-mediapipe-posedetection)
  - `vision-camera-pose-landmarks-plugin` v0.1.0: thin docs — [pkgstats](https://www.pkgstats.com/pkg:vision-camera-pose-landmarks-plugin)
- My search found no maintained VisionCamera plugin wrapping *Apple Vision* body pose. Using Apple Vision likely means writing your own native frame processor plugin or Expo module — [search summary, see ML Kit plugin list above](https://github.com/swittk/react-native-mlkit-pose-detection)

**Apple Vision (native)**
- `DetectHumanBodyPoseRequest` is the Swift struct API (iOS 18+, macOS 15, visionOS 2). It exposes `supportedJointNames` and joint groups, plus a `detectsHands` flag for including hands — [Apple: DetectHumanBodyPoseRequest](https://developer.apple.com/documentation/vision/detecthumanbodyposerequest.md)
- `DetectHumanHandPoseRequest` has `maximumHandCount` and supported joint names and groups. It outputs `HumanHandPoseObservation`. This is the basis for finger counting — [Apple: DetectHumanHandPoseRequest](https://developer.apple.com/documentation/vision/detecthumanhandposerequest.md)
- Legacy and Obj-C equivalents: `VNDetectHumanBodyPoseRequest` and `VNDetectHumanHandPoseRequest` (output `VNIdentifiedPointsObservation`-type points) — [Apple: VNDetectHumanBodyPoseRequest](https://developer.apple.com/documentation/vision/vndetecthumanbodyposerequest.md); [VNDetectHumanHandPoseRequest](https://developer.apple.com/documentation/vision/vndetecthumanhandposerequest.md)
- `VNDetectHumanBodyPose3DRequest` detects body points in 3D relative to the camera — [Apple: VNDetectHumanBodyPose3DRequest](https://developer.apple.com/documentation/vision/vndetecthumanbodypose3drequest.md)

**ARKit body tracking**
- Developer forum reports: ARKit body tracking needs an A12 chip or later (iPhone XR+). Check `ARBodyTrackingConfiguration.isSupported` at runtime and fall back. Using the `iphone-ipad-minimum-performance-a12` capability key had install problems on older iOS. Unofficial source — [Apple Dev Forums 127463](https://developer.apple.com/forums/thread/127463); [655144](https://developer.apple.com/forums/thread/655144)

### Inferences
- **Recommended architecture for the family's iPad on a stand:** front camera → VisionCamera v5 frame processor → pose or hand landmarks → simple heuristics in a worklet:
  - Jump: hip/shoulder keypoint vertical velocity spike plus both feet leaving the baseline.
  - Lean: shoulder-midpoint x offset.
  - Arms up: wrists above head.
  - Finger count: extended-finger test from hand joints.
  
  Emit only discrete events (e.g., `jump`, `lean_left`, `fingers=3`) to the JS thread.
- **Library choice:** if the team targets VisionCamera v5, most pose plugins listed above are v4-era and will need porting. The cleanest path may be a small custom Expo Module (Swift) that runs Apple Vision `DetectHumanBodyPoseRequest`/`DetectHumanHandPoseRequest` on frames (iOS 18+). This avoids shipping MediaPipe models and Google SDKs. The alternative is pinning VisionCamera v4 plus a MediaPipe plugin, at the cost of an upgrade path.
- **Phone-in-hand mechanics** (shake to roll dice, tilt to steer, pedometer missions) can use expo-sensors and are testable in Expo Go. Camera pose cannot be tested in Expo Go, because VisionCamera is a native module outside Expo Go. Plan an EAS development build from day one.
- **Clap and syllable detection:** use simple audio onset detection (RMS energy spike thresholding) on microphone input, processed on-device. Do not record or store audio.
- **Privacy:** process frames in memory only, never write them to disk or network, and show a visible "camera on" indicator. Parents enable the camera mode, with a parental gate. Pose landmarks need not be stored either, only aggregate counts (e.g., "42 jumps"). Children's data rules (COPPA and Apple Kids Category; Israeli Privacy Protection Law) favour this design.
- **Performance:** run pose at reduced resolution and frame rate (e.g., 15-30 fps at ≤720p), drop frames when busy, and keep the UI on the main thread. ARKit body tracking uses the rear camera only in world-tracking mode, which is impractical for a self-view game. Prefer front-camera 2D Vision pose.
- **Web fallback for testing:** expo-sensors DeviceMotion works on web. For pose on web, MediaPipe's JS Pose Landmarker or TensorFlow.js could stand in during development. I did not verify the details here.

### Gaps
- Expo SDK 58 specifics (release date, RN version, any changes to expo-sensors or expo-camera frame access) were not verified. The latest Expo sensors docs found were "latest/v57".
- Whether expo-camera (not VisionCamera) offers frame processing in SDK 58: not checked.
- Official Apple doc on ARBodyTrackingConfiguration device requirements and camera: not retrieved, only forum posts.
- No accuracy data found for Apple Vision or MediaPipe pose on *young children* (2-5 year olds have different body proportions). This should be tested in-house.
- Exact Apple hand-joint names and the finger-counting accuracy with small hands: not retrieved.
- Expo Go's inclusion or exclusion of VisionCamera: not explicitly cited this session. The dev-build requirement is standard knowledge but unverified here.

## 4. Safety & Design: space, placement, inclusivity, pacing, parent setup

### Takeaway
I found no formal safety standard for home exergames for young kids. The guidance below is inferred from the evidence above and standard practice. Clear a play zone, put the iPad on a stable stand at child chest height, never require holding the device while jumping (toddler), offer a seated mode, keep bursts short, and keep the toddler supervised.

### Cited Findings
- WHO under-5 guidance: prefer active play, limit sedentary screen time to ≤1 h for 2-4 year olds, and use caregiver co-engagement for quiet time — [WHO](https://www.who.int/publications/i/item/9789241550536)
- Off-the-shelf movement products did not increase MVPA without extra support in real classrooms. This argues for parent setup and guidance built into the app — [Augusta/Elsevier Pure 2022](https://augusta.elsevierpure.com/en/publications/classroom-teachers-off-the-shelf-use-of-movement-integration-prod)
- Short acute bouts are enough for attention benefits (g=0.43). Long sessions are not needed — [PubMed 29054748](https://pubmed.ncbi.nlm.nih.gov/29054748/)

### Inferences
- **Space:** about 2×2 m clear area per active child, away from furniture corners, glass and stairs. Use non-slip flooring or bare feet or grippy socks (no plain socks on tiles, which are common in Israeli homes). The app shows a camera "check your space" setup screen with the full body visible.
- **Device placement:** iPad on a stand or leaned against a wall on the floor or a low table, 2-3 m away, front camera at roughly child chest height. Never require a hand-held device during jumping. The phone-in-hand modes (shake, tilt) suit the 8.5-year-old, standing still or seated, with a wrist strap or case.
- **Multi-child:** one active player in the camera zone at a time, or side-by-side lanes with enough spacing. Turn-taking UI for siblings, plus co-op scoring.
- **Toddler:** always with a caregiver, no competitive timers, no fail sounds. Movements: imitation only (no fast direction changes or jumping off things).
- **Noise and neighbours:** "quiet mode" with no-jump alternatives (march in place, arm moves, tip-toe). Useful in apartments and at Shabbat or evening quiet times.
- **Inclusive / seated mode:** arm, head and finger gestures, plus a tap fallback for every movement prompt.
- **Pacing:** 1-3 min bursts, then a calm learning task. A cool-down breathing exercise ends each session. Daily caps visible to parents.
- **Parent onboarding:** a one-time checklist covering space, stand, footwear and camera privacy explanation, plus an optional per-session "is the space clear?" prompt.

### Gaps
- No peer-reviewed injury data for home exergaming in children under 10 was found (not searched).
- No official recommended camera distance or angle for Apple Vision body pose (not searched).
