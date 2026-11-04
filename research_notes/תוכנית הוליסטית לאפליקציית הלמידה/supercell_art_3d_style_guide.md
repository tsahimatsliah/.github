# Stylized 3D Character & Environment Style Guide (Supercell-tier look, original IP)

For the family learning game: **Spark-pilot heroes** (kids piloting "toy-suit" lantern beings that hold a living spark), **rescued light creatures**, a **comic shadow villain**, a **campfire camp hub**, and regions (crystal mines, letter hills, English lighthouse island).
Target stack (from `tech_3d_world.md`): three.js + R3F v9 on expo-gl, `MeshToonMaterial`, no post-processing on native, hero about 3–6k tris. Offline "lobby/UI renders" come from Blender. Researched Nov 2026.

**Legend.** Inline links back factual claims. **[INF]** marks my inference or practitioner rule of thumb, not a sourced fact. **[OBS]** marks an observation from looking at the games or their public art. It is not a published spec, so treat it as a starting value to tune.
**Research limits.** WebFetch could not reach `supercell.com` or `80.lv` from this sandbox (DNS blocked), so Supercell job posts and 80.lv articles are cited through search-result excerpts and reposts. I did not watch the GDC Vault talks. No Supercell source publishes poly counts, shader code or light values. All numbers below are [INF]/[OBS] unless linked.

---

## 0. The look in 12 rules (TL;DR)

1. **A chunky toy you'd put on your desk.** Supercell's skin-creator guidance (as summarized from the 2022 Supercell MAKE campaign): *"A skin should be like a chunky toy that you want to have on your desk. Focus on a couple of details and make them oversized to really pop out. Avoid adding too much detail."* ([GamingOnPhone summary of MAKE guidelines](https://gamingonphone.com/news/supercell-make-campaign-how-to-make-custom-skins-in-brawl-stars/)). This is almost exactly our "toy-suit" hero brief. Treat it as rule #1.
2. **Colorful, and it must pop in-game.** The official MAKE skin guidelines say to avoid dark and washed-out palettes, because characters are small on screen ([Supercell MAKE skin submission guidelines](https://make.supercell.com/en/create/brawl-stars/skin/submission-guidelines)).
3. **Cartoony, with humor in the attitude.** The style pillars are "Cartoony" and "Match the attitude of the skin with the wacky Brawl Stars humor" ([TapTap mirror of MAKE guide](https://www.taptap.io/es/post/9586389)).
4. **Pitch each character as a joke, and keep it simple.** Supercell game artist Fernanda Oliveira's Brawler pillars include humour, "less is more," and timeless tribute. A good reference should feel familiar without being a copy ([PocketGamer.biz: 5 creative pillars](https://www.pocketgamer.biz/5-creative-pillars-for-designing-brawlers-in-supercells-brawl-stars/); [GDC 2025 session "Making of a Brawler"](https://schedule.gdconf.com/session/making-of-a-brawler-character-creation-for-brawl-stars/907875)).
5. **Stylized forms, boxy props, simple gradients.** The Clash style "comes from the way the characters are modeled and textured… props and buildings got more of a boxy look that we complemented with simpler color gradients". The team "uses the PBR workflow but breaks PBR in many ways", with realistic shader response but stylized color and roughness ([Adobe Substance 3D Magazine: Supercell Helsinki](https://www.adobe.com/products/substance3d/magazine/supercell-helsinki-creating-stylized-content-for-clash-of-clans-and-clash-royale.html)).
6. **Silhouette first.** Read the character at thumbnail size before you add any texture ([GameDev.net on the GDC clip](https://gamedev.net/news/making-of-a-brawler-character-creation-for-brawl-stars-clip-3-r4113/); [gamespace playbook: "If the asset dies under a neutral HDRI, fix the silhouette or values"](https://gamespace.com/all-articles/news/stylized-3d-renders-that-pop-a-practical-playbook-for-game-teams/)).
7. **Big readable shapes and bevelled edges that catch light** [INF]. Premium toy-like 3D reads as "molded vinyl/plastic" because every edge has a chamfer that picks up a highlight.
8. **2–3 main colors plus 1 accent, with clear value separation** between head, body and prop [INF; value rule echoed by a Blizzard artist in [80.lv](https://80.lv/articles/004adk-talking-about-stylized-character-art/)].
9. **Soft, warm-key and cool-fill lighting, a rim that lifts characters off the background, no pure-black shadows** ([Blender Studio: "almost always some kind of rim light on the characters"](https://studio.blender.org/blog/wing-it-from-concept-to-render/); ratios in §3).
10. **Gameplay pieces are loud, decoration is quiet.** Bold, simple visuals let players focus on play ([RetroStyle Games blog](https://retrostylegames.com/blog/game-art-design-like-brawl-stars/), a secondary source).
11. **Cartoon VFX are animated drawings on flat planes with a limited palette**, not simulations ([Supercell VFX artist on realtimevfx](https://realtimevfx.com/t/very-cartoonic-vfx-for-brawl-stars-made-with-flash/11032)).
12. **Snappy animation**: strong anticipation, squash and stretch, held poses ([BetweenTheFrames: Animation Fundamentals: Brawl Stars](https://medium.com/betweentheframes/animation-fundamentals-brawl-stars-34ffb10bd32c)).

---

## 1. What Supercell has actually said (evidence base)

| Topic | Finding | Source |
|---|---|---|
| Brawler creation | GDC 2025 lecture by Fernanda Oliveira (Game Artist, Supercell). She defined the game's "core pillars and creative rules": humor, nostalgia, simple yet appealing design | [GDC schedule](https://schedule.gdconf.com/session/making-of-a-brawler-character-creation-for-brawl-stars/907875); [clip](https://www.youtube.com/shorts/y1Kx5eghopA) |
| Mash-up references | A Brawler's "shared DNA" mixes two unrelated film references (Frank-N-Furter + The Fly). The Gray Brawler is black and white, with Buster Keaton-style physical gags used as mechanics | [PocketGamer.biz](https://www.pocketgamer.biz/5-creative-pillars-for-designing-brawlers-in-supercells-brawl-stars/); [GDC TikTok via search](https://gamedev.net/news/3430/) |
| No magic in Brawl Stars | A ghost Brawler must be "not actually a ghost", so designs are grounded in a twist or gag | [PocketGamer.biz](https://www.pocketgamer.biz/5-creative-pillars-for-designing-brawlers-in-supercells-brawl-stars/) |
| Talk video | "Fernanda Oliveira – Games First 2024 – Creating Characters for Brawl Stars" | [YouTube](https://www.youtube.com/watch?v=KUgUTVy-l2I) |
| 3D role | Senior 3D Character Artist: stylized characters optimized for mobile performance. Maya named. Most volume goes to external studios | [Supercell careers](https://supercell.com/en/careers/senior-3d-character-artist-brawl-stars/2132787/); [80.lv jobs repost](https://80.lv/jobs/senior-3d-character-artist-supercell); [Space Ape/Accel repost](https://jobs.accel.com/companies/space-ape-games-2/jobs/68144610-senior-3d-character-artist-brawl-stars) |
| Clash pipeline | ZBrush detail, then bake in Substance Painter, with a pre-made stylized smart-material library. UDIMs go to Maya/Arnold for hi-res marketing and UI renders. "Breaks PBR in many ways" | [Adobe Substance Magazine (2019)](https://www.adobe.com/products/substance3d/magazine/supercell-helsinki-creating-stylized-content-for-clash-of-clans-and-clash-royale.html) |
| CoC rendering | Original game art is **pre-rendered 3D**. References include SNES/arcade (Advance Wars, Gauntlet). The camera is set higher than in similar games for playability | [PocketGamer.biz: making of CoC](https://www.pocketgamer.biz/chart-rush-making-of-clash-of-clans/) |
| CoC now | Heroes and pets moved to real-time 3D as hardware improved | [PocketGamer: CoC 10-year interview](https://www.pocketgamer.com/clash-of-clans/interview-ten-year-anniversary/) |
| Clash Royale | Units come from 3D models. Community sources say the in-game units are pre-rendered sprites (unofficial) | [Quora (informal)](https://www.quora.com/Is-Clash-Royale-a-2D-or-3D-game) |
| Squad Busters | "Very chibi anime style… takes our toon-style characters and remoulds them into a slightly different form" (Rob Lowe) | [GamesIndustry.biz mirror](https://dcmpx.remotevs.com/biz/gamesindustry/www/SL/supercells-smash-bros-how-the-stars-aligned-for-squad-busters) |
| VFX role | Stylized **2D** VFX, particle editors, Adobe Animate + Photoshop for sequence drawings and key animation. Heavy use of external studios | [Supercell careers VFX](https://supercell.com/en/careers/senior-vfx-artist-brawl/2019291/); [Squad Busters VFX post](https://supercell.com/en/careers/senior-vfx-artist-squad-busters/1608067/) |
| VFX practice | A Brawl Stars VFX artist (Woo): mostly Flash, avoid pre-rendered sequences, no custom shaders or mesh particles at the time, basic blend shaders, flat planes, limited palette, heavy animation. Thin graphics read as noise | [realtimevfx](https://realtimevfx.com/t/very-cartoonic-vfx-for-brawl-stars-made-with-flash/11032) |
| External VFX studio | OCTOPO (Seoul) ships Brawl Stars and Squad Busters VFX and teaches "Western-style cartoon effects": masses vs effects, master shader, unlit/stencil shaders, Adobe Animate | [Coloso course](https://coloso.global/en/products/vfxstudio-octopostudio-us); [OCTOPO ArtStation](https://octopovfx.artstation.com/) |
| Animation role | Senior Animator: "give life to new Brawlers" with external animation studios | [Supercell careers](https://supercell.com/en/careers/senior-animator/1684312/) |
| Outsourcing credits | Airborn Studios (3D/textures), Ocellus (Brawler portraits), Piñata (illustration), Golden Wolf / Psyop (cinematics) | [Airborn: Road Rage Carl](https://airbornstudios.artstation.com/projects/8lDxKE); [Ocellus portraits](https://ocellusart.artstation.com/projects/kNQ00A); [Piñata](https://studiopinata.artstation.com/projects/rA5RD5); [Golden Wolf](https://www.goldenwolf.tv/work/brawl-stars-the-last-stand) |

**Takeaway [INF].** Supercell's quality comes from a few rules applied consistently (toy-like chunk, humor, color pop, simple silhouettes, broken-PBR stylization) and from very heavy iteration. Exotic tech is not the source. Every rule can be reproduced with Blender + three.js.

---

## 2. Character design language

### 2.1 Proportions [OBS/INF: measured by eye from public art; tune per character]

| Character class | Height (heads) | Head width vs shoulders | Hands | Feet | Notes |
|---|---|---|---|---|---|
| **Spark-pilot hero (toy-suit)** | **2.5–3.0** | head ≈ 0.9–1.1× shoulder width | oversized "mitten" hands, ≈ 0.5–0.7 head width, 3 fingers + thumb or mitt | chunky boots, length ≈ 0.6–0.8 head | Big readable torso "suit" panel, so the spark/lantern core sits mid-chest or in the helmet |
| Light creature (rescued) | **1.2–2.0** | body *is* the head (blob/bean) | nubs or none | nubs | 1 dominant shape (circle/teardrop); eyes ≥ 25% of face height |
| Shadow villain (comic) | **3–3.5**, or a top-heavy 2.5 | narrow head, big shoulders/cape, OR huge head on a skinny body | long fingers, 4 digits | pointy small feet | Triangles + crescents; theatrical poses; one loud accent color |
| Kid pilot (visible in cockpit/helmet) | 2.0–2.5 (chibi) | — | — | — | Only head and hands visible inside the suit window |

- General chibi range is 2–4 heads, with 1:2 or 1:3 head-to-body ratios ([chibi guide excerpt via search](https://tips.clip-studio.com/es-es/articles/4879?org=1)). Squad Busters is officially "chibi" ([GI mirror](https://dcmpx.remotevs.com/biz/gamesindustry/www/SL/supercells-smash-bros-how-the-stars-aligned-for-squad-busters)). Brawl Stars Brawlers sit at the taller end, about 2.5–3.5 heads [OBS].
- **Legs short, at about 0.6–0.9 head [INF].** Gameplay reads the top of the character from a high camera, so mass and detail go into head, shoulders and props.
- **Weapon/prop/tool ≈ 0.5–1× body height [OBS]** and carries most of the identity. For us the prop is the **lantern/spark tool**, e.g. a lantern-staff, a flashlight-cannon or a letter-launcher.

### 2.2 Silhouette and shape language
- **Squint test**: render solid black at **64 px tall**. Each hero must be identifiable among the cast [INF]. The skin-creation community also warns against straying from the base silhouette ([MAKE guidance summaries](https://gamingonphone.com/news/supercell-make-campaign-how-to-make-custom-skins-in-brawl-stars/)).
- **One "hook" shape per character, oversized** (MAKE: "a couple of details… oversized"). Examples: a huge lantern helmet, a giant antenna spark, a lighthouse-lamp backpack.
- Shape language ([general principle](https://nastyrodent.com/stylized-3d-characters-art-direction-principles/), [skill summary](https://skills.sh/arjun988/blender-skills/stylized-style)):
  - **Heroes**: rounded rectangles plus circles, which read sturdy, friendly and toy-like.
  - **Light creatures**: circles and teardrops, with soft gradients and emissive cores.
  - **Villain**: triangles, crescents and spikes, but with rounded tips (comic, not scary, for ages 2.5–8.5).
- **Big/medium/small rhythm [INF]**: about 60% big mass (suit/body), 30% medium (head/helmet, prop), 10% small (buttons, rivets, eyes). Cheap designs spread detail evenly.
- **Asymmetry in one place** (a tilted antenna, a single shoulder pad, a patch). This gives life without breaking readability [INF].

### 2.3 Faces and attitude
- **Eyes**: large, simple shapes (ovals or capsules) with 1 big + 1 small specular dot, painted or as separate geometry so they can blink and swap [INF/OBS]. For under-5 appeal, set eyes **at or below the head's vertical midline** and spaced about one eye-width apart [INF, classic "baby schema"].
- **Brows**: thick, separate shapes floating above the eyes. They carry 70% of the attitude (confident, grumpy, scheming) [OBS: Brawl Stars faces are brow-driven]. The villain's brows should be oversized and very mobile.
- **Mouth**: a big, graphic shape, often asymmetric (smirk). Use texture swaps or blend shapes for 4–6 expressions: neutral, happy, determined, surprised, sad, laugh [INF].
- **Personality in 3 layers [INF]**: (1) silhouette/prop, (2) default face plus idle pose (lean, weight shift), (3) one signature gesture in the lobby idle (spin the lantern, polish a letter, yawn).
- **Pitch each character as a one-line joke** (Oliveira). Examples: "A lighthouse who is afraid of the dark." "A shadow who wants to be a stand-up comedian but every joke switches the lights off."

### 2.4 Color blocking
- **2–3 main hues + 1 accent**. Values separated by at least 20–25% lightness between adjacent big masses. Check in grayscale [INF; grayscale rule from [AoE forum](https://forums.ageofempires.com/t/friend-foe-colors-request-colorblind-user/74446) and the Blizzard artist in [80.lv](https://80.lv/articles/004adk-talking-about-stylized-character-art/)].
- **Saturation**: mid-high on the character, lower on the ground. Never pure black: darkest shadow about L* 15–20, tinted blue or purple [INF].
- **Our IP color roles [INF proposal]**:
  - Spark/light (the "soul" of every hero and creature): warm gold-amber core `#FFC845` → `#FF8A1F` edge, emissive.
  - Hero toy-suits: one primary per hero (sky blue `#3BA8FF`, tomato `#FF5A4E`, mint `#3FD6A0`, grape `#9A6BFF`), plus cream `#FFF3DC` and graphite `#3A3F55` trim.
  - Light creatures: pastel bodies with bright emissive cores (they "light up" when rescued, going from desaturated to full color).
  - Shadow villain: deep indigo `#2E2457` plus violet `#6B4BC4` with **one comic accent** (lime `#B8F44A` or bubblegum `#FF6FB5`). Purple-blue, never black, keeps it friendly and readable on dark ground.
  - Team/semantic colors: friendly = blue/cyan, hazard = red/orange plus a shape cue (spikes, stripes) for color-blind safety ([color-blind patterns discussion](https://steamdb.info/patchnotes/18345997)).

### 2.5 Skins/variants of a base character (how Supercell varies Brawlers)
- **Keep**: silhouette class, proportions, rig, face layout, the "hook" location. **Change**: theme, materials, palette, prop shape within the same bounding box [INF from MAKE rules: no simple recolors, stay true to the Brawler ([MAKE](https://make.supercell.com/en/create/brawl-stars/skin/submission-guidelines))].
- **Tiers [INF]**: (1) recolor plus pattern (cheap reward), (2) themed shell swap (new helmet/prop, same body mesh), (3) full remodel with custom VFX and animations.
- For our game, **suit skins = "Spark Suits"** earned per region: a Crystal Miner suit (faceted gem helmet, pick-lantern), a Letter Knight suit (alphabet blocks armor), a Lighthouse Keeper suit (rotating lamp hat, rain slicker).

---

## 3. 3D modeling & texturing

### 3.1 Budgets (our stack; real-time on iPad A12+)
| Asset | Game LOD (tris) | Showcase/lobby LOD | Texture | Bones |
|---|---|---|---|---|
| Hero (suit + kid) | **3–6k** ([tech_3d_world.md budget](./tech_3d_world.md)) | 8–15k (lobby only) [INF] | 1× 512² (game), 1024² (lobby) | ≤ 30–40 |
| Light creature | 800–2.5k | 4k | 256²–512² or palette atlas | 8–15 |
| Villain | 4–7k | 15k | 512²/1024² | ≤ 40 |
| Props | 200–2k | — | shared palette atlas | — |

- Community references: mobile heroes are about 1.5–10k tris, with 512–1024² textures ([Polycount threads](https://polycount.com/discussion/comment/1765758/); [Polygon count guides](https://blog.neural4d.com/user-guide/polygon-count-for-3d-game-assets-printing-and-webar/)). Fan remakes of Brawlers run 13–45k tris but are not in-game data ([ArtStation fan Sprout](https://www.artstation.com/artwork/e0Eorw), [Sketchfab Finx 13.1k](https://sketchfab.com/3d-models/finx-brawl-stars-3d-model-8a72d8c3857c489496f08e83944600bd)).
- **Spend polys on the silhouette and on bevels**, not on flat interiors. Use 2-segment chamfers (≈ 2–4% of part size) on all hard toy edges. Use weighted/custom normals so bevels shade smoothly [INF; standard stylized-game technique].

### 3.2 Texturing approach (choose per asset)
1. **Gradient/palette atlas** (cheapest; props, environment, creatures): UVs placed on a 256² swatch texture with vertical gradients per swatch (light top, darker bottom). This matches Supercell's "simpler color gradients" on boxy props ([Adobe](https://www.adobe.com/products/substance3d/magazine/supercell-helsinki-creating-stylized-content-for-clash-of-clans-and-clash-royale.html)).
2. **Hand-painted-lite base color** (heroes, villain): flat fills plus a **baked AO multiply (20–35%)**, a soft top-down gradient (+8–12% value at the top of each part), and a thin **cavity/curvature edge highlight (+10–15% value)** on convex edges. Paint no lighting direction into it, because the toon shader does that [INF; diffuse-only/unlit mobile practice in [Polycount](https://polycount.com/discussion/194666/hand-painted-mobile-game-characters)].
3. **PBR-lite** (lobby/marketing renders only): base color + roughness. Toy plastic roughness 0.35–0.5, rubber/cloth 0.7–0.85, painted metal 0.3–0.45 with metalness 0–0.3 (stylized metal is usually *colored* non-metal plus a strong spec) [INF]. "Realistic shader, stylized colors and roughness" is Supercell's own phrasing ([Adobe](https://www.adobe.com/products/substance3d/magazine/supercell-helsinki-creating-stylized-content-for-clash-of-clans-and-clash-royale.html)).
- **AO**: bake into the texture or into **vertex color** (cheap on mobile, but needs even topology) ([Unity docs on vertex AO](https://docs.unity3d.com/Packages/com.unity.industry.toolkit@3.2/manual/Actions/Autogenerated/BakeAmbientOcclusion.html)). AO color should be tinted warm-brown or purple, not gray [INF].
- **Specular/highlights**: for real time, use a single **stepped spec blob** (toon) or matcap highlight on plastic parts only. Cheap mobile packs use a toon matcap plus one 512² texture ([Unity Asset Store example](https://assetstore.unity.com/packages/3d/characters/humanoids/fantasy/toon-humanoid-animal-characters-16-unique-humanoid-animals-209083)).
- **Outlines**: Brawl Stars in-game characters do **not** rely on heavy black ink lines [OBS]. Readability comes from the rim, value separation and the soft shadow under the feet. **Recommendation for us [INF]**: a thin, colored (darkened local hue, not black) inverted-hull outline about 1.5–2.5% of character height, on characters and interactables only. That outline doubles as a "you can touch this" signal for small kids. Environment gets no outline.
- **Texel density**: keep it consistent across a character set (e.g., 256 px/m in game) [INF]. Mixed density is a top "cheap" tell.

### 3.3 Toy-suit + living spark specifics [INF]
- Model the suit as **molded vinyl panels** with separate meshes for helmet glass (a transparent sphere with fake fresnel), the spark core (emissive mesh + billboard glow sprite) and panel seams (inset 1–2 mm grooves baked into AO).
- **The spark is the hero's "face of the soul"**: an emissive core (HDR color ×2–4 in lobby renders) with flicker animation (noise on scale ±5% and emissive ±15% at 6–10 Hz). Add a camera-facing glow quad (additive, 1.5–2.5× core size). In-game there is no bloom, so the glow quad *is* the bloom.

---

## 4. Lighting & rendering (lobby/showcase + UI renders)

### 4.1 Real-time in-game (three.js, no post-processing)
- **Lights [INF]**: 1 `DirectionalLight` key (warm `#FFF1DC`, intensity ≈ 2.0–3.0 in r155+ units, elevation 50–60°, azimuth from camera-left/front) + `HemisphereLight` (sky `#CFE6FF`, ground `#8A6A55`, 0.6–1.0). No real-time shadow maps on mobile: use a **blob shadow decal** (dark-blue radial gradient, 40–55% opacity) under each character ([tech_3d_world budget](./tech_3d_world.md)).
- **Toon ramp**: 3 bands (shadow / mid / light) at values ≈ 0.55 / 0.8 / 1.0, with the shadow band **hue-shifted** toward blue/purple via the material color or a custom ramp [INF].
- **Rim**: a fresnel rim of 0.3–0.6 strength, cool `#BFE3FF` (or warm in the campfire hub), threshold 0.55–0.7, soft edge 0.05–0.1 (code in §7.4).
- Lighting ratios from general practice: Key 1.0, Fill 0.3–0.5, Rim 0.5–1.2, warm key / cool fill ([Blender lighting guide](https://cursa.app/en/page/blender-basics-for-animation-lights-for-readable-animation-and-simple-materials); [Creative Bloq](https://www.creativebloq.com/3d/how-to-use-key-fill-and-rim-lighting-in-3d-art)).

### 4.2 Lobby / showcase (camp hub) [INF recipe]
- **Composition**: hero centered on a **round pedestal** (a campfire log ring / stone disc, 1.2–1.5× character footprint), camera at chest height looking slightly up (5–10°), 35–50 mm equivalent, the character filling 55–65% of screen height. Slow idle and ±15° auto-sway or drag-to-rotate.
- **Light rig (camp hub at dusk)**:
  - Key: campfire, warm orange `#FFA347`, low (20–30° elevation), from front-left, 1.0.
  - Fill: sky, cool `#7FA8FF`, hemisphere 0.35–0.45.
  - Rim/back: moonlight `#A9D4FF` from back-right, 0.8–1.2, giving a strong separation line on shoulders and head.
  - Practical: the hero's own spark (emissive core lighting the face via a small point light, 0.3–0.5, radius 1.5 m).
- **Background**: 2–3 depth layers, all **lower contrast and lower saturation** than the hero (background values within 30–60% lightness, no pure whites). Use a soft vignette (painted into the backdrop texture, since there is no post on native). Particles: 10–30 slow ember sprites rising.
- **Post (offline/web only)**: bloom threshold 0.9–1.0, intensity 0.3–0.6, small radius; a slight warm grade; vignette 15–25%. On native, fake it with glow quads and a vignette overlay image.

### 4.3 2D UI renders from 3D (icons, chest/reward cards, portraits)
Supercell lineage: CoC art was pre-rendered 3D ([PocketGamer.biz](https://www.pocketgamer.biz/chart-rush-making-of-clash-of-clans/)). Clash teams render hi-res models for UI and marketing ([Adobe](https://www.adobe.com/products/substance3d/magazine/supercell-helsinki-creating-stylized-content-for-clash-of-clans-and-clash-royale.html)). Brawler portraits are painted/illustrated by partner studios ([Ocellus](https://ocellusart.artstation.com/projects/kNQ00A)).
**Our recipe [INF]**
1. Blender scene `ui_render.blend` with a *locked* camera per category: portrait (head + shoulders, 3/4 view, 50 mm), full body (35 mm, slight low angle), item (orthographic, 3/4 top view at 30°).
2. The same 3-light rig for every render (consistency beats perfection): key area light 45° left/up, warm, size 1 m; fill 0.35× cool; rim 1.0× from back-right.
3. Film → Transparent, 1024² (icons) or 2048² (cards). View transform **Standard** (not AgX/Filmic), otherwise saturated toon colors get muted ([toon look notes](https://www.strayspark.studio/blog/how-to-get-anime-toon-look-blender)).
4. Compositor: Glare (Bloom/Fog Glow) on emissives, plus an alpha-dilate **outer outline** of 6–10 px in a dark local hue (the "sticker" look common in mobile UI), plus a 2–4% inner shadow.
5. Export PNG, trim, then optimize to WebP/PNG at 2×/3×.
6. For "chest/drop" images: render the object with an exaggerated perspective (24–28 mm, close) and a light burst behind it (separate 2D layer).

---

## 5. Environment style

### 5.1 Camera and grid [INF; Brawl Stars camera is not published, and forum views conflict between orthographic and perspective ([Godot forum](https://forum.godotengine.org/t/how-to-get-camera-perspective-view-like-brawl-stars/23689))]
- **Exploration/arena camera**: perspective, **FOV 30–40° (vertical)**, pitch **50–60° down**, height so that about 9–11 tiles are visible across the iPad's short side. A narrow FOV keeps verticals near-straight and the "diorama" look ([lower FOV reduces distortion](https://polycount.com/discussion/comment/2630433)).
- **Grid**: 1 m tiles. Walls and obstacles are full tiles. The character is about 0.9–1.1 tile tall (heads read big from above). Brawl Stars builds maps from tile types (walls, bushes, water, destructibles) ([Brawl Stars wiki: Map Maker tiles](https://brawlstars.fandom.com/wiki/Competition_Winner)).
- **Kids' twist**: ages 2.5–8.5 need a closer camera (≈ 7–9 tiles across) and bigger interactables (≥ 1.2 tile) [INF].

### 5.2 Readability rules (gameplay vs decoration)
| Layer | Value / saturation | Detail | Outline/glow | Examples |
|---|---|---|---|---|
| Ground (walkable) | mid-light value, **low saturation**, low-frequency pattern (tile seams, 2–3 tones) | lowest | none | path stones, grass tiles |
| Blocking (walls/cliffs) | darker sides + **bright top faces** (the top face is what you see from 55°) | medium, chunky bevels | none | crystal rock walls, hedge blocks |
| Interactables / learning objects | **highest saturation + emissive accent** | medium | colored outline + idle bob/sparkle | letter blocks, lantern posts, rescuable creatures |
| Hazards | warm red/orange + **shape cue** (spikes/stripes) | medium | pulse | shadow puddles |
| Decoration (outside the play area) | **−20–30% saturation, −15% contrast**, slight blue haze with distance | high (allowed) | none | trees, background lighthouse, distant hills |
- **Prop density [INF]**: 60–70% of the playable area clear, props clustered in 3–7 item groups, and a "frame" of dense decoration on the arena border (Brawl Stars style). Avoid noise in the walkable center.
- **Color zoning per region [INF proposal]**:
  - **Camp hub (campfire)**: warm amber/orange key, teal night sky, log and canvas browns. The safe, cozy anchor.
  - **Crystal mines**: cool violet/indigo rock with **saturated cyan, magenta and gold crystals as emissive pickups**. Ground is desaturated slate. Crystals are the only high-chroma objects.
  - **Letter hills**: soft green rolling tiles with **giant toy alphabet blocks** (primary colors, one letter per face, rounded bevels). Flowers are low-saturation.
  - **English lighthouse island**: sandy cream, sea teal, a red-white lighthouse as the landmark visible from anywhere; fog and seagulls. Signage in English letters is a learning hook.
  - **Shadow villain zones**: dark purple-blue overlay with comic "ink puddles" that the spark clears, restoring full color (desaturate to resaturate as the reward).
- Players recognize environments by silhouette first ([environment readability commentary](https://vsquad.art/blog/articles/eduard-kolokolov-stylized-environment-art/)). Give each region **one landmark silhouette** (giant crystal, giant letter "A", lighthouse) [INF].

---

## 6. VFX style
- **Sourced core**: cartoon look, flat planes, limited palette, heavy hand animation, avoid pre-rendered sim sequences, and beware thin graphics because they read as noise ([Woo, Supercell, realtimevfx](https://realtimevfx.com/t/very-cartoonic-vfx-for-brawl-stars-made-with-flash/11032)). Taught pipeline: concept → masses → keyframe animation, plus a master shader with unlit/stencil variants ([OCTOPO course](https://coloso.global/en/products/vfxstudio-octopostudio-us)).
- **Shapes [INF]**: circles, teardrops, 4–6-point stars, rings and chunky "puff" clouds with **3-tone cel shading** (white-hot core / hue / darker hue edge). Never wispy realistic smoke.
- **Colors**: our "spark" family = white core → gold `#FFD34D` → orange `#FF8A1F` edge. "Shadow" family = indigo with lime or pink highlights. Learning-success family = rainbow confetti + star bursts. Keep each effect to ≤ 3 hues + white.
- **Timing [INF; classic 2D FX timing]**: anticipation/charge 3–6 frames → impact flash 1–2 frames (white or full-bright) → main shape 4–8 frames → dissipate 8–15 frames, decelerating. Flipbooks on twos (12 fps) in a 30/60 fps game for a hand-drawn feel. Successful-answer celebration: total ≤ 1.2 s so it doesn't slow learning loops.
- **Tech for us [INF]**: flipbook sprite sheets (4×4 or 8×8 on 512²/1024²) drawn in Krita/Animate or generated and cleaned up. Additive or alpha-blended unlit `ShaderMaterial` on instanced quads. Use ≤ 300 particles on screen on A12. Flipbook smear technique: [Udemy stylized explosion overview](https://www.udemy.com/course/visual-effects-for-games-in-unity-stylized-explosion/).

---

## 7. Animation style
- **Principles shown in Brawl Stars' cinematic language**: squash & stretch, anticipation, exaggeration ([BetweenTheFrames](https://medium.com/betweentheframes/animation-fundamentals-brawl-stars-34ffb10bd32c)). Animators need "command of weight, timing and posing" ([Supercell animator posting via search](https://supercell.com/en/careers/senior-animator/1684312/)).
- **Numbers [INF, at 30 fps]**:
  | Clip | Length | Notes |
  |---|---|---|
  | Game idle | 1.5–2.5 s loop | breathing + spark flicker, 2–4% body squash |
  | Lobby idle | 4–6 s loop + a "fidget" every 8–15 s (2–3 s) | the personality gesture |
  | Run | 0.5–0.7 s cycle (≈ 16–20 frames) | big bounce, head lags 1–2 frames, arms pump |
  | Action/attack | anticipation 4–6 f → strike 2–3 f → hold 3–5 f → recover 8–12 f | pose-to-pose, stepped-feel ease |
  | Hit/oops | 6–10 f | squash 15–25%, eyes swap |
  | Victory | 1.5–2.5 s + held final pose | readable as a still frame (it becomes the UI pose) |
  | Correct-answer cheer | ≤ 1.0 s | must not block the next question |
- **Squash & stretch on the toy suit**: preserve volume (scale Y 0.85 ↔ X/Z 1.08). Use secondary motion on antenna, lantern and cape (spring bones or procedural damped spring in code).
- **Mixamo/auto-rig motions look "realistic-generic"**. To make them stylized [INF]: (1) speed up 1.2–1.5×, (2) cut in-betweens on the anticipation, (3) push key poses +20–30% further, (4) add a 2–3 frame hold at extremes, (5) add procedural squash on landing in code.
- **The villain moves theatrically**: slow sneaky anticipation, then fast comic "boing" exits. Silent-film/vaudeville references, in the spirit of Oliveira's Buster Keaton example ([PocketGamer.biz](https://www.pocketgamer.biz/5-creative-pillars-for-designing-brawlers-in-supercells-brawl-stars/)).

---

## 8. Practical pipeline (solo AI agent + designer parent)

### 8.1 Flow
```
Designer one-liner joke + mood refs
 → GPT Image 2.5 concept (front 3/4 hero render + turnaround sheet, §8.6)
 → parent picks/annotates → image edit iterations ("change only X")
 → Image-to-3D (Meshy 6 with quad/T-pose or Tripo for stylized) → GLB
 → Blender cleanup (remesh/retopo, bevel, symmetry, UV, AO bake, palette texture)
 → Rig (Meshy auto-rig / Mixamo / Rigify) → stylize animations (§7)
 → Export glTF (≤6k tris game LOD, 512² tex) → gltf-transform optimize
 → three.js toon material + rim + outline (§8.4)
 → Blender lobby/UI renders (§4.3) for 2D UI
```

### 8.2 Image-to-3D generation and cleanup
- **Tool choice (2026 comparisons, vendor-influenced)**: Tripo P1 is rated best for stylized/hand-painted in one test. Meshy 6 offers quad topology plus A/T-pose control, which drops into Mixamo without cleanup. Rodin is too realistic for cartoon ([StraySpark April 2026](https://www.strayspark.studio/blog/generative-3d-tools-comparison-meshy-rodin-tripo-csm-2026); [3D AI Studio comparison](https://www.3daistudio.com/blog/hitem3d-vs-meshy-vs-tripo-comparison); [rigging showdown](https://www.strayspark.studio/blog/ai-auto-rigging-showdown-2026-tripo-meshy-cascadeur-mixamo)). Meshy says "stylized/cartoon" prompts give cleaner topology ([Meshy docs](https://docs.meshy.ai/en/webapp/text-to-3d)). It also advises remeshing high-poly or irregular output before rigging ([Meshy auto-rig guide](https://www.meshy.ai/tutorials/character-auto-rigging-workflow)).
- **Input image for 3D gen [INF]**: a single character, **front or 3/4 view, A-pose, arms 30–45° away from body, fingers apart, neutral face, flat even lighting, no rim, no outline, plain mid-gray background**. Lighting and outlines in the input get baked into the generated texture as dirt.
- **Cleanup checklist in Blender [INF]**
  1. Apply scale and set the origin at the feet; character height 1.0 m (hero), consistent across the cast.
  2. Symmetrize (X), then Decimate (or QuadriFlow/Instant Meshes remesh) to the budget. Hand-fix the face loops around the eyes and mouth if blend shapes are needed.
  3. Separate parts the AI fused (helmet glass, spark core, eyes) into their own meshes/materials.
  4. Add a Bevel modifier (2 segments, limit by angle 30°) on hard toy parts, then Weighted Normal (keep sharp).
  5. **Replace the AI texture**: AI textures carry baked lighting and noise. Re-texture as flat fills + palette atlas + AO bake (Cycles AO bake, 64 samples, distance 0.2 m) multiplied at 25%.
  6. Eyes and brows as separate planes/meshes for texture-swap expressions.
  7. Check silhouette at 64 px and grayscale values (§2).

### 8.3 Rigging
- Meshy auto-rig or **Mixamo** (upload T/A-pose, place markers. Free with a large clip library) ([showdown](https://www.strayspark.studio/blog/ai-auto-rigging-showdown-2026-tripo-meshy-cascadeur-mixamo)). For non-humanoid creatures (blobs), hand-rig 5–12 bones in Blender (root, body, squash bone, eyes, ears/antenna spring chain).
- Add a **squash bone** (root child at the pelvis) so code can apply squash/stretch without retargeting [INF].
- Keep ≤ 40 bones and one skinned mesh per hero (budget). Bake animations to glTF actions; retarget Mixamo clips in Blender, then stylize (§7).

### 8.4 three.js / R3F toon material recipe
```js
import * as THREE from 'three';

// 1) 3-band toon ramp (NearestFilter is required by MeshToonMaterial)
//    https://threejs.org/docs/#api/en/materials/MeshToonMaterial ; https://sbcode.net/threejs/meshtoonmaterial/
function makeRamp(levels = [140, 205, 255]) {
  const data = new Uint8Array(levels.length * 4);
  levels.forEach((v, i) => data.set([v, v, v, 255], i * 4));
  const tex = new THREE.DataTexture(data, levels.length, 1, THREE.RGBAFormat);
  tex.minFilter = tex.magFilter = THREE.NearestFilter;
  tex.generateMipmaps = false;
  tex.needsUpdate = true;
  return tex;
}
const RAMP = makeRamp();

// 2) Toon + fresnel rim via onBeforeCompile (inject before final output chunk)
export function makeToon({ map, color = '#ffffff', rimColor = '#bfe3ff',
                           rimStrength = 0.45, rimThreshold = 0.6, rimSoft = 0.08,
                           emissiveMap = null, emissive = '#000000' } = {}) {
  const m = new THREE.MeshToonMaterial({ map, color, gradientMap: RAMP, emissiveMap, emissive });
  m.onBeforeCompile = (shader) => {
    shader.uniforms.uRimColor = { value: new THREE.Color(rimColor) };
    shader.uniforms.uRimStrength = { value: rimStrength };
    shader.uniforms.uRimThreshold = { value: rimThreshold };
    shader.uniforms.uRimSoft = { value: rimSoft };
    shader.fragmentShader = shader.fragmentShader
      .replace('#include <common>', `#include <common>
        uniform vec3 uRimColor; uniform float uRimStrength, uRimThreshold, uRimSoft;`)
      // r154+: <opaque_fragment>; older three: <output_fragment>
      .replace('#include <opaque_fragment>', `
        { vec3 V = normalize(vViewPosition);           // view-space, toward camera
          float f = 1.0 - clamp(dot(normal, V), 0.0, 1.0);
          f = smoothstep(uRimThreshold, uRimThreshold + uRimSoft, f);
          outgoingLight += uRimColor * f * uRimStrength; }
        #include <opaque_fragment>`);
  };
  m.customProgramCacheKey = () => `toonrim`;
  return m;
}
```
- Optional **light-side-only rim** [INF]: multiply `f` by `smoothstep(-0.2, 0.4, dot(normal, viewLightDir))` with a uniform for the key-light direction in view space. This gives the Brawl Stars-like "rim on the lit side" feel and keeps rim off the shadow side ([rim technique](https://lettier.github.io/3d-game-shaders-for-beginners/rim-lighting.html); [BlenderNation toon rim](https://www.blendernation.com/2020/07/15/how-to-get-perfect-rim-lights-toon-shader-tutorial/)).
- **Hue-shifted shadows [INF]**: replace the gray ramp with an RGB ramp (shadow texel e.g. `[120,120,170]`, mid `[205,200,215]`, light `[255,255,255]`). The shadow band then tints cool automatically.
- **Outline (inverted hull)**: in R3F use drei `<Outlines thickness={0.02} color="#2b2350" />` as a child of the `mesh`/`skinnedMesh`. Defaults: thickness 0.05, `angle` crease. Works without post-processing ([drei Outlines docs](https://drei.docs.pmnd.rs/abstractions/outlines)). Caveats: per mesh, and it can draw interior edges ([three.js forum](https://discourse.threejs.org/t/outlines-around-objects-cartoonish-effect/57206)). Hard-edged meshes split normals and produce gaps in the hull, so **use smooth (averaged) normals for the hull** or keep the toy bevels so normals stay continuous [INF]. Plain three.js equivalent: clone geometry → `MeshBasicMaterial({ color, side: THREE.BackSide })` + `onBeforeCompile` with `transformed += normal * uThickness;` after `#include <begin_vertex>`. For skinned meshes, the hull must be a `SkinnedMesh` bound to the same skeleton.
- **Spark glow**: emissive core (`emissive` color + `emissiveIntensity` 1.5–3) + additive sprite (`SpriteMaterial({ blending: AdditiveBlending, depthWrite: false })`) scaled 1.5–2.5×, flicker in `useFrame`.
- **Blob shadow**: a `planeGeometry` with a radial-gradient PNG, `transparent`, `depthWrite: false`, `polygonOffset`, opacity 0.45, scaled by jump height.
- **Renderer**: `toneMapping = THREE.NoToneMapping` or `ACESFilmic` with exposure tuned. ACES desaturates brights; for toon color fidelity prefer `NoToneMapping`/`LinearToneMapping` + sRGB output [INF]. Use `dpr={[1, 2]}`, antialias off on native (budget).

### 8.5 Blender stylized shader (for lobby/UI renders and for previewing)
- **EEVEE toon**: Principled/Diffuse BSDF → **Shader to RGB** → (Map Range to move the terminator) → **ColorRamp (Constant)**. Stops: 0.0 shadow color (hue-shifted, ~55% value), 0.45 mid, 0.75 light. Mix (Multiply) with the base color texture ([TehMerow wiki](https://github.com/TehMerow/Tutorials/wiki/Toon-Shading-with-Eevee); [gskinner on Shader to RGB](https://blog.gskinner.com/archives/2020/05/shader-rgb-node.html)). Shader to RGB is EEVEE-only.
- **Rim**: Layer Weight (Facing, blend 0.3) → ColorRamp (Constant at 0.6) → × rim color → Add ([BlenderNation rim tutorial](https://www.blendernation.com/2020/07/15/how-to-get-perfect-rim-lights-toon-shader-tutorial/)).
- **Spec blob**: Glossy/Specular → Shader to RGB → ColorRamp (Constant, 0.9) → Add white × 0.6 on plastic parts only.
- **Outline**: Solidify modifier (thickness −1.5% of height, Flip Normals, Material Offset → a black-tinted emission material with Backface Culling), or Grease Pencil Line Art for UI renders [INF; standard Blender technique].
- **Color management: View Transform = Standard**. AgX mutes saturated toon colors ([StraySpark note](https://www.strayspark.studio/blog/how-to-get-anime-toon-look-blender)).
- **Softer "premium toy" alternative (closer to Brawl Stars lobby renders) [INF]**: keep the Principled BSDF (roughness 0.4–0.55, subsurface 0.05 on soft parts), soft area lights (§4.2), AO enabled in EEVEE (distance 0.3 m), and let the rim and color do the stylization. This is "realistic shader, stylized colors" per Supercell ([Adobe](https://www.adobe.com/products/substance3d/magazine/supercell-helsinki-creating-stylized-content-for-clash-of-clans-and-clash-royale.html)).

### 8.6 GPT Image prompt templates (original designs, lobby quality)
Model notes: gpt-image-1 was shut down Oct 23 2026, so use **GPT Image 2.5 (Sunburst/Flare)**. Request transparency via the `background="transparent"` parameter with PNG/WebP, not only in the prompt ([OpenAI image prompting guide](https://developers.openai.com/api/docs/guides/image-prompting)). Transparency can punch holes in white areas of the subject; add "no transparency inside the character" ([OpenAI community](https://community.openai.com/t/gpt-image-1-transparency-remove-background-also-cuts-out-other-white-spots-of-the-image/1273481)). For edits: "Change only X. Keep everything else exactly the same". Use a small set of reference images, each referred to by order ([OpenAI Academy via search](https://openai.com/academy/image-generation)).
**IP hygiene [INF]**: never put game, studio or character names in prompts ("Brawl Stars", "Supercell", "Clash", any Brawler name). Describe the *attributes* instead. Names pull the model toward copying and can trigger refusals. Keep an "Avoid" block in every prompt (GPT Image has no negative-prompt parameter, so write exclusions in prose).

**Shared STYLE block (paste into every prompt)**
```
STYLE: premium stylized 3D mobile-game character render, chunky collectible-toy proportions
(2.5–3 heads tall, big head, oversized hands and boots, short legs), soft molded-vinyl and
matte-plastic materials with rounded bevelled edges catching small highlights, simple clean
color blocking (2–3 main colors + 1 accent), smooth hand-painted-feel gradients, subtle ambient
occlusion, soft warm key light from front-left, cool fill, crisp cool rim light separating the
silhouette, no harsh black shadows, saturated but harmonious colors, appealing expressive face
with large simple eyes and thick expressive eyebrows, playful confident attitude, readable
silhouette at small size, family-friendly, polished AAA mobile quality.
AVOID: any existing game, cartoon, movie or toy-brand character or logo; recognizable
trademarked designs; text, watermarks, signatures; realistic human anatomy or skin pores;
photorealism; anime screentone; thin noisy details; excessive greebles; guns or realistic
weapons; scary, gory or horror features; dark muddy palette; pure black outlines; extra
fingers; blurry or distorted hands; cropped feet.
```

**T1. Hero concept render (lobby quality)**
```
Create an ORIGINAL hero character: a "Spark Suit", a small toy-like walking lantern robot suit
piloted by a cheerful 7-year-old kid visible through a round glass helmet window. In the suit's
chest is a living spark: a glowing golden-orange flame creature with tiny happy eyes.
Suit theme: [Crystal Miner: faceted amethyst helmet, pick-shaped lantern tool, cyan crystal shoulder pads].
Main colors: [grape purple, cream], accent: [cyan crystal glow], spark: gold-orange.
Pose: confident heroic 3/4 stance, one hand raising the lantern tool, slight lean, weight on back leg.
Scene: standing on a round mossy stone pedestal by a cozy campfire at dusk, soft bokeh forest and
tents behind (low contrast, darker and less saturated than the character), floating embers.
Framing: full body, character fills 60% of the frame height, camera at chest height looking slightly up.
{STYLE block}
```
**T2. Turnaround sheet (input for image-to-3D)**
```
Character turnaround model sheet of the SAME character as image 1: front, side, back views and a
3/4 view, A-pose with arms 40° from the body, fingers spread, neutral friendly expression,
evenly lit flat studio lighting with NO rim light, NO cast shadows, NO outlines, plain mid-gray
background, orthographic-looking views aligned on one ground line, identical proportions and
colors in every view. {STYLE block, but replace lighting sentence with "flat even lighting"}
```
**T3. Rescued light creature**
```
ORIGINAL cute light creature, 1.5 heads tall bean/teardrop body, tiny nub arms, huge sparkling eyes,
soft translucent pastel [mint] body with a bright glowing core visible inside, tiny leaf antenna.
Two states side by side: left "lost" (desaturated gray-blue, sleepy eyes, dim core), right "rescued"
(full color, joyful jump, glowing core, little sparkles). Transparent background. {STYLE block}
```
**T4. Comic shadow villain**
```
ORIGINAL comic villain "the Shadow Prankster": a theatrical living shadow made of soft deep-indigo
ink with violet highlights, top-hat-like wobbly head shape, huge expressive eyebrows, mischievous
grin, long noodle arms with 4-finger gloves, little crescent-moon cape, ONE bright lime accent
(glowing bow tie). Silly, not scary: rounded spike tips, big cartoon eyes. Pose: sneaky tiptoe
with exaggerated anticipation. Purple-blue palette, never pure black. {STYLE block}
```
**T5. Environment tile/diorama (top-down game view)**
```
Top-down 55-degree perspective game arena diorama, narrow lens, square tile grid of 1-meter tiles,
region: [Letter Hills]: soft green rounded grass tiles, giant toy alphabet blocks as walls (rounded
bevelled cubes, primary colors, one embossed letter per face), small flower clusters at the borders,
a winding cream path through the middle. Gameplay area clean and readable; decoration dense only at
the border; walkable ground low saturation; blocking objects have bright top faces; interactable
objects most saturated with a soft glow. Warm afternoon key light, soft blue shadows.
{STYLE block adapted: "premium stylized 3D mobile game environment"}
```
**T6. UI icon / reward render**
```
Single game reward icon: an ORIGINAL treasure lantern-chest made of polished wood and gold trim with
a glowing spark inside the keyhole, dramatic slightly low 3/4 view, wide-angle toy perspective,
thick soft dark-violet sticker outline around the whole object, light burst behind, transparent
background, no text. {STYLE block}
```
- **Consistency loop [INF]**: lock T1's best output as "image 1". Pass it plus a palette swatch image to every later prompt. Generate turnarounds with T2. Keep a `style_refs/` folder of 4–6 approved images and always send the same ones. Have a vision check score new images against the refs ([consistency approach in ai_models_assets.md](./ai_models_assets.md)).

---

## 9. Premium stylized vs "cheap mobile clone" (checklist)

| Premium (aim) | Cheap clone tell |
|---|---|
| A documented rulebook: proportions, shape, texture, lighting applied to every asset ([art-direction consistency](https://nastyrodent.com/stylized-3d-characters-art-direction-principles/); [Rocketbrush](https://rocketbrush.com/blog/the-ultimate-character-art-style-guide-for-artists-and-developers)) | Mixed asset packs with different texel densities, line weights and lighting |
| Silhouette and values solved before texture ([gamespace](https://gamespace.com/all-articles/news/stylized-3d-renders-that-pop-a-practical-playbook-for-game-teams/)) | Detail used to hide a weak silhouette |
| One oversized hook detail, restraint elsewhere ("chunky toy") | Uniform greebles everywhere, noisy textures |
| Bevelled edges catching highlights, deliberate material separation (vinyl vs rubber vs glow) | Razor-sharp edges, or blobby melted AI meshes with no planes |
| Hue-shifted shadows, colored AO, no pure black | Gray/black AO, muddy desaturated shadows |
| Rim light and value separation lift the character off the background | Character the same value as the ground |
| A character concept that is a joke or a story (Oliveira) | A recolor of a genre archetype |
| Faces that act (brows, mouth swaps, blinks) | Static painted faces |
| Hand-tuned, snappy animation with holds and squash | Raw mocap/Mixamo floatiness, foot sliding |
| VFX as designed shapes with timing | Generic particle soup and realistic smoke |
| UI renders from one locked camera and light rig | Every icon lit differently |
| Environment value hierarchy (gameplay loud, decor quiet) | Busy ground textures, decoration inside the play space |

---

## 10. Best reference URLs (study list)
- Supercell MAKE skin guidelines: <https://make.supercell.com/en/create/brawl-stars/skin/submission-guidelines> · character guideline pages: <https://make.supercell.com/en/create/brawl-stars/skin/character-guidelines/Kit>
- GDC 2025 "Making of a Brawler" (Fernanda Oliveira): <https://schedule.gdconf.com/session/making-of-a-brawler-character-creation-for-brawl-stars/907875> (full video on GDC Vault)
- Games First 2024 talk: <https://www.youtube.com/watch?v=KUgUTVy-l2I> · write-up: <https://www.pocketgamer.biz/5-creative-pillars-for-designing-brawlers-in-supercells-brawl-stars/>
- Supercell Helsinki Substance interview (Clash style, broken PBR): <https://magazine.substance3d.com/supercell-helsinki-creating-stylized-content-for-clash-of-clans-and-clash-royale/>
- Making of Clash of Clans (pre-rendered 3D, camera): <https://www.pocketgamer.biz/chart-rush-making-of-clash-of-clans/>
- GDC 2022 Clash IP expansion talk: <https://gdcvault.com/play/1027594/Expanding-an-IP-Clash-of>
- Brawl Stars VFX with Flash (Supercell VFX artist): <https://realtimevfx.com/t/very-cartoonic-vfx-for-brawl-stars-made-with-flash/11032>
- OCTOPO (Brawl Stars/Squad Busters VFX vendor) portfolio and course: <https://octopovfx.artstation.com/> · <https://coloso.global/en/products/vfxstudio-octopostudio-us>
- Partner portfolios: Airborn <https://airbornstudios.artstation.com/projects/8lDxKE> · Ocellus portraits <https://ocellusart.artstation.com/projects/kNQ00A> · Piñata <https://studiopinata.artstation.com/projects/rA5RD5> · Squad Busters portraits <https://www.artstation.com/artwork/kNYwXz>
- Supercell ArtStation hub: <https://supercell.artstation.com/>
- Toy-aesthetic comparison (Astro Bot design evolution): <https://blog.playstation.com/2024/09/09/astro-bot-how-team-asobi-created-a-unified-vision-for-fun/>
- three.js MeshToonMaterial: <https://threejs.org/docs/#api/en/materials/MeshToonMaterial> · drei Outlines: <https://drei.docs.pmnd.rs/abstractions/outlines> · rim math: <https://lettier.github.io/3d-game-shaders-for-beginners/rim-lighting.html>
- Blender toon: <https://github.com/TehMerow/Tutorials/wiki/Toon-Shading-with-Eevee> · rim: <https://www.blendernation.com/2020/07/15/how-to-get-perfect-rim-lights-toon-shader-tutorial/>
- AI 3D tools 2026: <https://www.strayspark.studio/blog/generative-3d-tools-comparison-meshy-rodin-tripo-csm-2026> · Meshy prompting: <https://docs.meshy.ai/en/webapp/text-to-3d>
- OpenAI image prompting: <https://developers.openai.com/api/docs/guides/image-prompting>

## 11. Gaps / next steps
- **Not verified directly**: full text of Supercell job posts and 80.lv articles (host blocked in this sandbox), and the GDC Vault video content. Allow `supercell.com`, `80.lv`, `gdcvault.com` in network settings to deepen this.
- **No official numbers** for Brawl Stars poly counts, texture sizes, shader model, camera FOV/pitch or lobby light values. Everything numeric here is a tuned starting point [INF]. Validate with an on-device lookdev spike: 1 hero + 1 creature + 1 tile set, screenshot comparisons at 64 px / 256 px / full screen, and parent and kid preference tests.
- **Spike acceptance criteria [INF]**: hero identifiable in a black 64-px silhouette line-up; grayscale value separation between head, body and prop ≥ 20%; 60 fps on the oldest iPad with 3 heroes, 10 creatures and the full tile set; kids (2.5 / 5.5 / 8.5) can point to "what can I touch" within 3 s on a new screen.
