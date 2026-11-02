# 09 — מערכת עיצוב UX/UI ומפרט אינטראקציה (UX/UI Design System & Interaction Spec)

> מסמך מימוש. כפוף ל-`00-foundation-decisions.md`, ובמקרה של סתירה 00 גובר. הצעות לשינוי ב-00 מרוכזות ב-§16.
> קהל: סוכן הפיתוח (טוקנים, כללים, מצבים, קריטריוני קבלה) וההורה-מעצב (שערי אישור, §14).
> מקורות: הדוח v1 (פרקי "ליטוש", "מה לגנוב", "כלכלה", טבלאות 5, 6 ו-12); הרשימות `ui_ux_polish_craft.md`, `hit_games_engagement.md`, `kids_edu_apps_teardown.md`, `child_psychology_motivation.md`, `coop_economy_ios.md`, `gap_audit.md`, `practitioner_wisdom.md`, `hebrew_first_language_voice.md`, `app_stack_ai_built_workflow.md`, `tech_3d_world.md`, `astrobot_joy_gamification_psychology.md`, `learning_through_movement.md`, `content_world_phygital.md`, `photo_to_game_pipeline.md`; יומן המחקר `docs/research-log/ux-ui.md` (מדידות גופנים, ניגודיות, עיוורון צבעים, קפיצים, המרות מ"מ).
> סטטוס: v1 · 2026-11-02. ערכים שסומנו **[מכויל]** הם ערכי פתיחה שנבדקים מול הילדים ועל המכשיר.

---

## 0. מוסכמות

### 0.1 מזהים
| קידומת | משמעות | דוגמה |
|---|---|---|
| `color.*`, `type.*`, `space.*`, `radius.*`, `stroke.*`, `bevel.*`, `shadow.*`, `dur.*`, `ease.*`, `spring.*`, `sfx.*`, `mus.*`, `hap.*`, `touch.*` | טוקנים (שמות באנגלית, נגזרים ישירות לקוד ב-`packages/ui/tokens`) | `spring.press.release` |
| `CMP-xx` | רכיב בספריית הרכיבים (§7) | `CMP-03 AnswerTile` |
| `SCR-xx` | מסך (§8) | `SCR-04 EpisodePlayer` |
| `CHOREO-xx` | כוריאוגרפיה (§5.3) | `CHOREO-05 ChestReveal` |
| `GATE-UX-n` | שער אישור הורה (§14) | `GATE-UX-2` גופן וניקוד |
| `UX-AC-nn` | קריטריון קבלה שנבדק אוטומטית או ידנית | `UX-AC-07` |

### 0.2 פרופילים ומגדר
- `K1` (בת, 8.5), `K2` (בן, 5.5), `K3` (פעוט/ה, 2.5, מגדר טרם נקבע): ראו A2 ב-00.
- כל מחרוזת שמופנית לילד נכתבת כ-`{זכר|נקבה|רבים}`, ובקוד היא `{m, f, pl}` (B3, תואם ל-07 §0.3). ל-K3 מנסחים בניוטרלי עד שההורה מאשר מגדר.
- **כלל ההומוגרף (חדש, חובה):** בעברית בלי ניקוד, צורות רבות של זכר ונקבה נכתבות אותו דבר אבל נהגות אחרת: מָצָאתָ/מָצָאת, רוֹצֶה/רוֹצָה, קִבַּלְתָּ/קִבַּלְתְּ. לכן:
  - גם כשהטקסט על המסך זהה, **קובץ השמע תמיד נפרד לכל מגדר**.
  - הטקסט שנשלח ל-TTS תמיד מנוקד.
  - הלינטר מסמן מחרוזת שיש לה `m == f` בטקסט אבל רק `audioId` אחד.

### 0.3 רגיסטרים
שני רגיסטרים חזותיים ושמיעתיים באותו דקדוק:
- **`register.hero`**: "Supercell שמנמן". קווי מתאר עבים, שפה (lip) כהה, נדירות בצבע, הרבה חלקיקים. מיועד ל-K1, ובגרסה מרוככת `register.heroLite` ל-K2.
- **`register.calm`**: "Pok Pok". קו מתאר דק ורך, בלי ניצנוצים, תנועה איטית, פחות אלמנטים. מיועד ל-K3, לרגעי קריאה, לחדר המוגן (B8) ולטקס הסיום.

הרגיסטר נגזר מהפרופיל ומההקשר: מסך קריאה של K1 הוא `calm` גם אצלה.

---

## 1. עקרונות עיצוב (10)

1. **הדמויות הן הממשק.** לפני כל תפריט יש דמות שמצביעה, מגיבה ומסבירה. טקסט הוא שכבה אופציונלית מעל הקול (Toca, Khan Kids).
2. **עולם, לא אפליקציה (G1).** כל מסך הוא מקום בעולם. ניווט = הליכה, דלת, גשר. אין "רשימות" לילדים.
3. **פעולת הלמידה היא פעולת המשחק (C1).** אין חלון קופץ של שאלה בתוך העולם. השאלה היא מנעול, גשר, פלטפורמה או חבר כלוא.
4. **תגובה בכל פריים.** כל מגע מקבל תגובה חזותית תוך פריים אחד (≤16ms), וצליל ומגע (haptics) תוך ≤50ms. גם כפתור מושבת מגיב ("עוד לא").
5. **סקרנות מתוגמלת תוך שניות (G1).** בכל מסך עולם יש לפחות "סוד" אחד שאפשר למצוא תוך 10 שניות: ניצוץ, צליל או חפץ שמגיב.
6. **ג'וס מדויק ומאופק.** הג'וס נשמר לרגעי הצלחה, חילוץ ותגמול. רגעי קריאה שקטים. כל אפקט צריך לתקשר משהו, ותקציב החגיגה הוא ≤15% מזמן הפרק (§5.4).
7. **אין עונש, אין לחץ (C5).** אין לבבות, אין שעון שרץ אחורה, אין אדום של "טעות", אין צליל באזר, ואין "כמעט זכית".
8. **צבע אף פעם לא לבד.** כל משמעות מקודדת בצבע + צורה + צליל. זה נמדד, ראו §3.4.
9. **RTL כשפת אם, לא כהיפוך.** כיווני קריאה והתקדמות זורמים מימין לשמאל. העולם התלת-ממדי, הדמויות, השליטה הפיזית ונוטציה מתמטית לא מתהפכים (§3.11).
10. **שלוש קבוצות גיל, עולם אחד.** אותו עולם ושלושה "עורות" של צפיפות, שליטה וקול. הבת לעולם לא רואה ממשק "של תינוקות" (Gelman: ילדי 6-8 לא רוצים להיראות "בייבי").

---

## 2. הנחיה אמנותית (Art Direction Brief)

### 2.1 שפת צורות
- **שלוש צורות יסוד:** עיגול, מלבן מעוגל ומשולש מעוגל (דקדוק Duolingo). כל דמות וכל רכיב בנויים מהן, עם חזרה על צורות בתוך הדמות.
- **הכול מעוגל:** אין פינה חדה בממשק או בעולם. רדיוס מינימלי לפי §3.8, ובתלת-ממד bevel בכל קצה.
- **כלל צל אחד:** צל "גלולה" שטוח מתחת לדמויות וחפצים צפים (לא אליפסה פרספקטיבית). בתלת-ממד: blob shadow (E1b).
- **כלל אור אחד:** האור בא **ישר מלמעלה, מעט מקדימה** (לא משמאל ולא מימין). כך היפוך אייקון ב-RTL לא הופך את ההבהקה. ההבהקה תמיד ברצועה העליונה של הצורה.
- **פרופורציות דמות:** ראש גדול (ראש:גוף ≈ 1:1 לגיבור, 1:1.3 לחברים), עיניים גדולות, פה א-סימטרי שנוטה לצד אחד.
- **"שבירת המסגרת":** אייקון או דמות חורגים מגבולות הכפתור או הכרטיס ב-10-20%. זה נותן דינמיות ב-`register.hero` בלבד.

### 2.2 קו מתאר, שפה והבהקה (2D)
| רגיסטר | קו מתאר | שפה (lip) | הבהקה פנימית | צללים |
|---|---|---|---|---|
| `hero` (K1) | `stroke.hero` 3pt, `color.ink.900` | `bevel.lip.k1` 6pt בגוון `*.700` | רצועה עליונה 30% בגובה, לבן 35% | `shadow.drop` קשה |
| `heroLite` (K2) | 3pt `ink.900` | `bevel.lip.k2` 8pt (עבה יותר כדי שיהיה ברור שאפשר ללחוץ) | 30%, לבן 30% | `shadow.drop` |
| `calm` (K3, קריאה) | `stroke.calm` 2pt `ink.700` | `bevel.lip.k3` 6pt רך | 20%, לבן 20% | `shadow.soft` |

הקו הכהה הוא **דרישת נגישות**, לא רק סגנון. פני הכפתורים הרוויים על רקע שמנת עומדים על ניגודיות של 1.46-4.06 בלבד, מתחת ל-3:1 שנדרש לרכיבי ממשק (WCAG 1.4.11). קו מתאר `ink.900` מגיע ל-4.1-11.4 מול כל פנים (מדידה ב-research log R5).

### 2.3 עולם תלת-ממדי / 2.5D: "צעצוע חי" (G1, G7, E1b)

**הקונספט:** דיורמה של צעצועים. ויניל רך, פלסטיק מט, לבד, עץ צבוע ופימו. העולם נראה כמו משהו שאפשר להחזיק ביד: חמים, מבריק במידה ו"נלחץ" למגע. כל אובייקט בעולם מגיב (עלים מתנדנדים, פטריות קופצות, אבנים מצפצפות). זו "צפיפות הצעצוע" של Astro Bot, בדקדוק מקורי לחלוטין (G4).

| נושא | כלל | הערות מימוש (E1b) |
|---|---|---|
| חומרים | `MeshToonMaterial` עם gradient map של 3 מדרגות (צל, בינוני, אור) + rim light רך; `MeshLambertMaterial` לסביבה רחוקה. בלי PBR | gradient map אחד לכל אזור (`toonRamp.<region>`), 3×1px |
| צבע | צבעי ורטקס או אטלס פלטה קטן (≤512²) לכל אזור, נגזרים מהפלטה בסעיף 3. רוויה גבוהה בחזית, נמוכה יותר ברקע (פרספקטיבה אווירית) | בלי טקסטורות מצולמות |
| קו מתאר | inverted hull: עובי 2-3% מגובה הדמות לגיבור, לחברים ולפריטי איסוף. **אין** קו מתאר לטרריין ולצמחייה | אין post-process (יקר, לא נתמך ב-expo-gl) |
| צל | blob shadow (דיסק/גלולה רך, `ink.900` ב-25%) תמיד מתחת לגיבור ולפריטים. זה קריטי לקריאת עומק בקפיצות | בלי shadow maps; לכל היותר 1024² על הגיבור |
| תאורה | אור כיווני אחד מלמעלה-קדימה + hemisphere (שמיים/אדמה בגוון האזור). AO אפוי בצבעי ורטקס | |
| גיאומטריה | עבה, שמנמנה ו-beveled. בלי קירות דקים ובלי חפצים חדים. גובה מדרגה ≤ חצי גובה הגיבור | גיבור 3-6k משולשים, ≤40 עצמות |
| מצלמה | מעקב גוף שלישי, pitch קבוע 40° (טווח 35-50°), FOV צר 35° לתחושת דיורמה, yaw אוטומטי לכיוון התנועה | `spring.camera.follow`; דהיית מסתירים |
| שמיים ואטמוספרה | כיפת גרדיאנט רכה + עננים מצוירים שנעים לאט. ערפל עומק בגוון האזור | |
| חלקיקים | quads ממוקרים (instanced): ניצוצות, כוכבים, עלים, קונפטי. צורות בלבד, בלי טקסט | ≤200 חלקיקים חיים |
| טקסט בעולם | **אסור טקסט עברי בתלת-ממד.** כל תווית, בועה, מונה ושלט היא שכבת 2D מעל העולם, ממוקמת בהטלה מנקודת עוגן (E1b). שלטים בעולם הם אייקונים או ציורים | אותיות *כתוכן לימודי* (אבן עם א') מצוירות כטקסטורה מוכנה מראש, ונבדקות ב-GATE-UX-2 |
| 2.5D (fallback 2) | ספרייטים מרונדרים מראש **מאותם מודלים, באותה תאורה ובאותו toon ramp**, במבט איזומטרי/פרלקסה ב-Skia. כך מעבר ל-2.5D לא משנה את השפה | אותו קובץ טוקנים |

**איפוק:**
- K3: אין רעידת מצלמה ואין חלקיקים מהירים.
- K2/K1: רעידת מצלמה רק באירוע "פיצוץ" (תיבה, בוס). משרעת ≤2pt, ≤150ms, ונכבית ב-Reduce Motion.

### 2.4 דמויות: משפחות שלעולם לא מתערבבות
| משפחה | תפקיד | שפה חזותית | צבע/סימן | הערות |
|---|---|---|---|---|
| **הגיבור/ה** (G4) | הדמות שהילד "מטיס" ומלביש | צעצוע-חליפה עגלגל מוויניל רך, ראש גדול, עיניים מצוירות אקספרסיביות, "ניצוץ-לב" זוהר בחזה, צעיף או שכמייה בצבע הילד | `child.theme.*` | **לא נקרא "רובוט" בטקסט.** ראו התנגשות עם `ROBOT` למטה. מעוצב יחד עם הילדים (ציורים → חלקים), GATE-UX-4 |
| `COMPANION` (07) | חבר סיפור חם ומתוסרט, מנחה, מדבר על התוכן | יצור עגול עם פרווה/לבד, אוזניים גדולות, פה א-סימטרי | צבע אזור הבית (`ember/sun`) | לא AI |
| `ROBOT` גַּלְגַּלּוֹן ו-`BOTS` (07) | עוזר-מכונה שטועה בכוונה | קופסאות מלבניות-מעוגלות, **גלגלי שיניים ונורות גלויים, בלי פנים אנושיות** (מסך עם מד במקום עיניים) | `color.machine.*` (אפור-פלדה + נורה צהובה) + תג גלגל-שיניים קבוע | **אייקון גלגל השיניים שמור רק למשפחה הזו** בכל המערכת |
| חברים אבודים (G2) | יצורים קטנים לחילוץ. כל אחד נושא אות, מספר, צבע או מילה | גחליליות/ניצוצות עם גוף קטן ועיניים גדולות. סילואטה ייחודית לכל אחד | צבע אזור + סמל התוכן שהם נושאים | אחרי חילוץ הם עוברים לגור במחנה ועובדים, מנגנים ובונים |
| "צללונים" (07/content) | "יריבים" של בלגן שגונבים אור ואותיות | עגולים, מתנדנדים וקומיים, עיניים עגולות לבנות (**לא אדומות**), בלי שיניים וטפרים | `dusk.500` + נצנוץ | כשמנצחים אותם הם מתפוגגים לנצנצים והופכים לחברים |

**התנגשות שזוהתה (G4 מול C7/D13):** גיבור "דמוי-רובוט" חמוד שיש לו רגשות, לצד `ROBOT` ש"אין לו רגשות", עלול לטשטש לילדים את ההבחנה בין דמות סיפור למכונה. **הכלל:**
- הגיבור מעוצב כ**צעצוע-חליפה עם ניצוץ חי**, בלי גלגלי שיניים גלויים, בלי מסך-פנים, ובלי תג מכונה.
- בקופי הוא נקרא בשמו או "הגיבור/ה", לעולם לא "רובוט" ולא "AI".
- הפרדה מלאה בצליל: לגיבור צלילי צעצוע, ל-ROBOT ביפים.

ראו הצעה ב-§16.

### 2.5 אזורי העולם ומצבי הצבע
שמות עבודה. אוצר העולם ושמותיו הסופיים בספק העולם; כאן מוגדרים הטוקנים בלבד.

| טוקן אזור | נושא | מצב רוח וצבע | ראמפ toon (צל/בינוני/אור) | מוזיקה (§6.5) | אלמנט חתימה |
|---|---|---|---|---|---|
| `region.camp` | ה-Hub, המדורה, חדר האוצרות | שקיעה זהובה, חם ומזמין | `#9E5A26 / #FF8A3D / #FFE590` | `mus.camp` 92 BPM | המדורה הגדולה, פנסים שנדלקים עם כל חילוץ |
| `region.letterForest` | עברית | יער ירוק עמוק, פטריות ופנסי אותיות זוהרים בגוון berry | `#1F6B30 / #4CC764 / #B5F0BE` + `berry.500` | `mus.forest` 100 BPM | אבני-אות, פנסי ניקוד |
| `region.numberPeaks` | מתמטיקה | הרים קפואים-מתוקים, קריסטלים כחולים, שמיים בהירים | `#15528C / #3BA4F5 / #E6F5FF` + `sun.500` | `mus.peaks` 110 BPM | מדרגות-מספר, גשרי קו-מספרים |
| `region.englishIsle` | אנגלית (LTR, B2) | אי טרופי סגול-טורקיז, שלטים מצוירים | `#5F37C0 / #8C5CF0 / #A8F0E9` | `mus.isle` 104 BPM, יוקלילי ושריקה (כלי חתימה של "אנגלית") | ספינה/מעבורת כניסה, ג'ינגל `sfx.english.enter` |
| `region.builderBay` | בנייה, מדידה, גאומטריה, יצירה | חוף חולי-שמשי, פיגומים צבעוניים | `#D99A0B / #FFC83D / #FFF8EC` + `sea.500` | `mus.bay` 96 BPM | מנוף, לבני-בנייה |
| `region.wonderCaves` | מדע, עולם, כישורי חשיבה | מערות ביולומינסצנטיות בטורקיז עם פנסי ענבר. חשוך-רך, **לא מפחיד** | `#11877D / #22C3B5 / #A8F0E9` + `ember.200` | `mus.caves` 84 BPM | גבישים זוהרים, מפות קדומות |
| `region.calmMeadow` | חדר משחקים של K3, "החדר המוגן" (B8), SEL | אחו פסטלי, ניגודיות רקע נמוכה, תנועה איטית | `#8FB9A0 / #B5F0BE / #FFF8EC` | `mus.meadow` 72 BPM | בועות, חיות רכות |
| `time.night` (G5) | ערב/הגנה על המדורה | `dusk` כחול-סגול עם אור מדורה חם. **ערב נעים, לא אימה** | `#221E45 / #37306B / #FFC83D` (אור אש) | `mus.campfire.night` 80 BPM אקוסטי | המדורה כמקור האור הראשי, גחליליות |

**כללי לילה:** בלי עיניים אדומות, בלי צללים שקופצים, בלי צלילי סירנה (B8), ובלי חושך מוחלט, כי המדורה תמיד נראית. "אש מתעממת" מוצגת כאש קטנה וישנונית עם פרצוף מנומנם, לא כמשהו שמת.

### 2.6 פרומפטים ללוח השראה ולייצור נכסים
לשימוש עם Nano Banana / GPT Image (E10, לא Gemini API לפי E8a), Recraft ל-SVG, ו-Meshy/Tripo לתלת-ממד (tech_3d_world). כל תוצר נרשם ב-`LICENSES/assets.json`.

**סיומת חובה לכל פרומפט:**
`original character design, no text, no letters, no logos, no existing franchise, not Astro Bot, not PlayStation, not Nintendo, not Supercell characters, child-friendly, soft vinyl toy aesthetic, toon shading with 3-step light ramp, rounded bevelled shapes, top-front key light, soft pill-shaped contact shadow, saturated but warm palette`

**פרומפט שלילי:** `realistic, photographic, horror, sharp teeth, red eyes, weapons, blood, guns, explosions, sirens, military, gloomy, thin lines, tiny details, text, watermark, brand`

| # | נכס | פרומפט (אנגלית, לכלי היצירה) |
|---|---|---|
| MB-1 | גיבור, קונספט | `A small brave toy-suit hero, chunky rounded body, oversized head with big expressive painted eyes and an asymmetric smile, a glowing star-shaped "spark heart" on the chest, a short scarf in {child color}, tiny boots with springs, 3/4 view, turnaround sheet front/side/back, white background` |
| MB-2 | גיבור, 6 פוזות | `Same hero: idle bounce, run, jump apex, landing squash, celebrate arms-up, curious peek; expression sheet: happy, surprised, determined, sleepy, giggle` |
| MB-3 | חבר אבוד | `A tiny lost firefly-sprite friend, round body the size of a plum, huge eyes, little antenna light, holding a carved wooden token, shy and hopeful, inside a soft glowing bubble cage, toy diorama` |
| MB-4 | Hub / מחנה | `A cozy campfire camp in a magical forest clearing at golden hour, toy diorama tilt-shift, tents, lanterns, a big friendly campfire with a sleepy face, small wooden buildings under construction with scaffolds, empty spots waiting to be filled, isometric 40-degree camera` |
| MB-5 | יער האותיות | `Lush toy forest, giant rounded mushrooms that glow softly, stepping stones carved with blank shapes, hanging lanterns, chunky trees, deep greens with pink-magenta glows, playful and safe` |
| MB-6 | הרי המספרים | `Candy-like icy mountains with blue crystal stepping blocks, a rainbow bridge made of segmented planks, fluffy clouds, bright sky, toy diorama` |
| MB-7 | אי האנגלית | `Small tropical toy island with purple-teal palette, a ferry boat dock, palm trees with round leaves, a wind-sock, a ukulele-playing parrot-like original creature` |
| MB-8 | מערות הפלא | `Gentle bioluminescent teal cave with warm amber lanterns, round crystals, soft mist, cozy not scary, small mine-cart rails` |
| MB-9 | ערב במדורה | `Evening campfire scene, deep blue-violet dusk, warm firelight circle, round wobbly comic shadow-blobs with white round eyes peeking from bushes looking more mischievous than scary, kids' heroes holding lanterns` |
| MB-10 | ערכת UI | `Mobile game UI kit, chunky rounded buttons with thick dark navy outline, darker bottom lip, top highlight band, bold rounded font placeholders as blank bars, coin and gem counters, chest in bronze/silver/gold frames with different frame shapes, cream background, flat hard drop shadows` |
| MB-11 | חלקי אווטאר | `Avatar parts sheet in the same style: 12 skin tones on an even ramp, 30 hairstyles including curly, coily, braids, locs, side curls, kippa, headscarf, glasses, hearing aid, missing front tooth smile, front-facing, separate parts on grid` |
| MB-12 | גלגלון (ROBOT) | `A friendly but clearly mechanical helper machine, boxy rounded steel-grey body, visible gears and light bulbs, a gauge display instead of a face, no human eyes or mouth, small gear badge, slightly clumsy` |

---
