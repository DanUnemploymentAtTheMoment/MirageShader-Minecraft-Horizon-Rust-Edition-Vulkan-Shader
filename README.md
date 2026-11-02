<h1 align="center">Mirage Shader Horizon</h1>

<p align="center">
  <strong>Look like a path tracer. Cost like Chocapic13 V6 Lite.</strong><br>
  A performance-first shader family for Horizon, the Rust rewrite of Minecraft: Java Edition 26.3.
</p>

<p align="center">
  <img alt="Game" src="https://img.shields.io/badge/Minecraft%20Java-26.3-62b47a">
  <img alt="Engine" src="https://img.shields.io/badge/engine-Horizon%20(Rust)-b7410e">
  <img alt="Platform" src="https://img.shields.io/badge/primary%20target-Android-3ddc84">
  <img alt="Edition" src="https://img.shields.io/badge/edition-Light%20v0.2-8e6bd8">
  <img alt="Status" src="https://img.shields.io/badge/status-experimental-orange">
</p>

---

## At a glance

| | |
|---|---|
| **What it is** | A shader pack for Horizon built around one idea: reproduce the *look* of expensive rendering with cheap maths. |
| **Who it is for** | Players on mid-range and low-end Android GPUs who still want atmosphere, depth and lighting. |
| **Current edition** | **Mirage Light v0.2**, the baseline all other editions are built on. |
| **Pipeline** | Optional shadow pass, forward lighting, one composite pass, one final pass. One colour buffer. |
| **Does not need** | Compute shaders, geometry shaders, custom textures, normal maps, specular maps, a G-buffer. |
| **Dimensions** | Separate Overworld, Nether and End rendering. |

---

## What is Mirage?

Most shader packs get their quality by doing more work: more samples, more passes, more accurate light transport. That works on a desktop GPU. On a phone, with a small GPU and a shared memory bus, it falls apart quickly.

Mirage takes the opposite approach. For every effect it asks one question:

> *What is the cheapest thing that still looks right?*

It is not trying to be physically correct. It is trying to be **perceptually convincing** at a cost a phone can afford. The philosophy is inherited from Chocapic13 V6 Lite, which gets a lot of mileage out of imitation instead of simulation, and the visual ambition comes from the path-traced packs that show what the result *should* feel like.

### The Mirage substitutions

| The expensive way | The Mirage way |
|---|---|
| Global illumination | Sky/ground hemisphere ambient, a fill light opposite the sun for colour bounce, and a warm torch curve |
| Screen-space or ray-traced reflections | Water reflects the **analytic sky** along the reflection direction. No tracing at all |
| Volumetric fog and scattering | Exponential plus height haze, tinted by the sky colour in that direction, with a three-lobe sun glow |
| Soft shadow filtering with many taps | A distorted shadow map with two hardware-filtered taps, plus a skylight-based fake shadow beyond range |
| Full ambient occlusion | Four depth samples, close-range only |
| Fluid water simulation | Three travelling sine waves with analytic slopes |
| Texture-based sky, stars and clouds | Fully procedural: gradient, discs, hashed stars, one-plane noise clouds with a sun-shifted tap for self-shadowing |
| Heavy post-processing | One composite and one final pass, with an optional motion blur that costs nothing while you stand still |

---

## Editions

Mirage is a **family**, not a single pack. Every edition shares the same philosophy and the same core architecture. They differ in how much *budget* they spend.

### Mirage Light: the baseline

**Light is not a cut-down version of something bigger. It is the foundation.**

Light is the reference implementation of the Mirage philosophy: the leanest pipeline, the cheapest approximation for every effect, and the architecture every other edition inherits. Features are only added to the project if they can be justified against Light's cost. If an effect cannot be built cheaply, it does not go into the baseline.

Light's contract:

- One colour buffer, no G-buffer, no extra render targets
- Two post passes (composite and final), plus an optional shadow pass
- Work that is constant per frame is computed per vertex, never per pixel
- Every expensive effect is independently switchable
- Night, the Nether and the End skip the shadow pass entirely

### The edition roadmap

> These tiers describe direction, not promised dates or contents. Names may change.

| Edition | Status | Intent |
|---|---|---|
| **Mirage Light** | **Current (v0.2)** | The baseline. Maximum efficiency, widest hardware reach. |
| **Mirage Micro** | Planned | The same architecture under a stricter budget, for the weakest GPUs: fewer post effects, shorter ranges, simpler sky. |
| **Mirage Standard** | Idea | Light plus carefully chosen enhancements for mid-range devices, each justified by its measured cost. |
| **Mirage Plus** | Idea | Higher visual targets for capable hardware, still built on Light's cheap-first principles. |

**The rule for every edition:** it inherits Light's architecture, and anything it adds must be measured against Light's baseline. If an enhancement does not earn its cost, it does not ship.

---

## Features in Mirage Light v0.2

### Lighting
- Hemispheric sky ambient with ground bounce, a directional sun and moon, and a colour-bounce fill light
- Warm torch light with a hot core, plus a held-light falloff
- Wrapped diffuse for a softer terminator
- Faked emissive glow for torches, lanterns and other full-brightness faces
- Wet-surface darkening in rain

### Shadows
- One shadow map (1024, distortion-warped), **daytime only**, two hardware-filtered taps
- Skylight-based fake shadows beyond the shadow distance, indoors, and when shadows are turned off
- The shadow pass is skipped entirely at night

### Sky and atmosphere
- Analytic sky with sunrise and sunset colour, a three-lobe sun glow, sun and moon discs
- Procedural stars, no textures
- Procedural clouds with approximate self-shadowing and a silver lining
- Aerial perspective haze with height falloff
- Sun shafts: six depth samples, evaluated only when you are facing the sun

### Water
- Sky reflections with a Fresnel blend (sun glitter comes free from the sky model)
- Travelling-sine wave normals that calm with distance
- Underwater light absorption and in-scatter

### Dimensions
Each dimension has its own full shader set, not a recoloured Overworld.

| Dimension | Approach |
|---|---|
| **Overworld** | Full day/night cycle, sun and moon, clouds, shadows, haze |
| **Nether** | No sun or sky. Warm ambient tinted by the biome's fog colour, a soft headlight so shapes stay readable, stronger torch and lava light, its own fog |
| **End** | The Overworld's night recipe without the moon: cool violet ambient, one faint fixed light, a violet haze sky with stars, no sun and no clouds |

### Environmental animation
- Leaves sway face by face, plants sway from the top of the sprite, with slow gusts rolling across the world and stronger movement in rain
- Needs block IDs from Horizon. Without them nothing moves, and nothing breaks

### Cinematic motion blur
- Camera motion blur from reprojection, for a smoother feel at modest frame rates
- Zero cost while the view is still; four extra texture reads while it moves
- Adjustable strength, with a capped blur length

### Output
- Filmic tone mapping, saturation control, light vignette, and noise dithering to hide banding in an 8-bit pipeline

---

## Architecture

```
        SHADOW PASS                (overworld, daytime only)
              |
              v
     FORWARD LIGHTING              (per draw; sun, ambient, torch, shadows, water)
              |
              v
          COMPOSITE                (sky, clouds, haze, AO, sun shafts, underwater)
              |
              v
           FINAL                   (motion blur, tone map, grade, dither)
```

Design choices that matter on mobile:

- **One colour buffer.** Skylight rides in the alpha channel, so fog does not need a second target.
- **Forward, not deferred.** No G-buffer means far less memory traffic on a tile-based GPU.
- **Normals from derivatives.** Surface normals come from screen-space derivatives instead of a normals buffer.
- **8-bit with a square-root curve.** A sqrt encoding with four times the headroom stretches 8 bits further, and dithering hides what is left.
- **Constants live at the vertex.** Sun colour, ambient and light direction are computed per vertex and handed to the pixel stage as varyings.
- **Dimension folders.** Nether and End get their own shader sets so the Overworld path never pays for them.

---

## Performance

Light is engineered for **tile-based mobile GPUs**, where the usual enemies are memory bandwidth, full-screen passes and overdraw, rather than raw arithmetic.

What that means in practice:

- Fewer full-screen passes and no mip-chain bloom by default
- Shadows only when the sun can actually cast them
- Pixel-stage maths kept to multiplies and adds wherever possible
- Range-limited effects (AO, shadows) so distant pixels stay cheap
- Conditional effects (sun shafts, motion blur) that cost nothing when you are not using them

### Measuring it

Performance claims in this project should be measurable. Releases ship with **benchmark builds** that remove features cumulatively, so the real cost of each stage can be read off a frame-time overlay:

| Build | What is removed |
|---|---|
| `bench0` (normal) | nothing |
| `bench1` | AO, sun shafts, motion blur |
| `bench2` | also fog, sky and clouds (flat sky) |
| `bench3` | also shadows (fake shadows only) |
| `bench4` | also all lighting (albedo only) |

**Test protocol:** same world, same spot, same time of day, same render distance, 100% render resolution. Record GPU frame time (ms) three times per build. The drop between consecutive builds is the cost of that stage.

### Reference device and baseline

| | |
|---|---|
| Device | Redmi Note 15 Pro 5G |
| SoC / GPU | Dimensity 7400 Ultra / Mali-G615 MC2 |
| Resolution | 100% |
| Render distance | 8 chunks |
| Graphics API | Vulkan |

| Version | Scene | GPU time | FPS |
|---|---|---|---|
| v0.1 | Overworld, forest at night | ~50 ms | ~19 |
| v0.1 | End (Overworld-style sky) | ~24 ms | ~38 |
| **v0.2** | Overworld, same spot | *to be measured* | *to be measured* |

v0.2 numbers will be published here once the benchmark builds have been run on the reference device. They are not estimated.

---

## Requirements and installation

**Requirements**
- Horizon (Minecraft: Java Edition 26.3, Rust rewrite) with shader-pack support
- A GPU that runs Horizon's renderer; developed on Android with Vulkan
- No texture pack or extra resources needed

**Installation**
1. Download the latest release zip. Do **not** extract it.
2. Place it in Horizon's `shaderpacks` folder.
3. Select **Mirage Light** in the shader menu.

**Options** live in `shaders/lib/settings.glsl` (and in the in-game options screen if your build exposes it):

| Option | Effect |
|---|---|
| `SHADOWS` | Real sun shadows |
| `AMBIENT_OCCLUSION` | Four-tap depth AO |
| `CLOUDS` | Procedural clouds |
| `GODRAYS` | Sun shafts |
| `WIND`, `WIND_STRENGTH` | Vegetation sway |
| `CINEMATIC_BLUR`, `BLUR_STRENGTH` | Camera motion blur |
| `BLOOM` | Mip-based bloom (off by default; costs a full-screen mip chain) |
| `EXPOSURE`, `SATURATION`, `FOG_DENSITY` | Look adjustments |

---

## Status and known limitations

Mirage is **experimental**. It has been run in Horizon on a real device, but compatibility across the wider range of hardware and Horizon builds is not yet established.

| Area | Limitation |
|---|---|
| Dimensions | Relies on Horizon loading per-dimension shader folders. If a dimension looks like the Overworld, that support is missing |
| Wind | Needs block IDs from Horizon. Without them vegetation stays still |
| Emissives | No per-block emissive data; glow is estimated from light level and brightness |
| Water vs. glass | Told apart by biome tint |
| Bloom | Needs the loader to honour buffer mipmaps; disabled by default |
| Blending | Fog accuracy behind translucent surfaces depends on the loader supporting separate-alpha blending |
| Motion blur | Needs previous-frame matrices from the loader |

---

## Roadmap

**Now: Light**
- [x] Forward-lit baseline with approximate GI, shadows, AO, sky, clouds, water, haze
- [x] Real-device testing on the reference phone
- [x] Dedicated Nether and End rendering
- [x] Lightweight vegetation sway and motion blur
- [ ] Publish measured v0.2 performance on the reference device
- [ ] Use the benchmark ladder to find and remove the most expensive stages
- [ ] Confirm per-dimension folders and block IDs on current Horizon builds

**Next: widen the family**
- [ ] Define the Micro budget and build it as a stricter preset of Light
- [ ] Evaluate reduced-resolution effects where Horizon allows it
- [ ] Prototype Standard enhancements against Light's measured baseline
- [ ] Improve water, cloud and transparency quality within budget

**Later**
- [ ] Revisit the architecture where measurement shows a real win
- [ ] Better modularity so editions share code instead of copying it

---

## Philosophy

1. **Look first, physics second.** If an approximation convinces the eye, it is a valid technique.
2. **Performance is a requirement.** Cost is considered when an effect is designed, not after.
3. **Measure, then optimise.** Changes are judged by frame time on a real device, not by feel.
4. **Keep the pipeline small.** Every pass, buffer and texture read has to justify itself.
5. **Stay coherent.** Lighting, sky, haze, water and shadows should read as one world.
6. **Borrow ideas, not code.** Techniques are studied, understood and re-implemented independently.

---

## Credits and licensing

- **Chocapic13**, for the performance-first philosophy this project is inspired by
- Path-traced shader packs for Horizon, for showing the look worth chasing
- **Horizon**, the Rust rewrite of Minecraft: Java Edition 26.3, and its community Android port
- Mirage's shader code is original. No source code from other shader packs is included

**License:** *(Credits and Licensing

Credits

- Chocapic13 — Inspiration for the performance-first rendering philosophy behind Mirage.
- Horizon — The rendering environment for which Mirage Shader Horizon is developed.
- The Shader Community — Inspiration, feedback, testing, and continued support.

License

Mirage Shader Horizon is licensed under the GNU General Public License v3.0 (GPL-3.0).

See the ""LICENSE"" (LICENSE) file for the complete license terms.

Copyright © 2026 Dan / DanUnemploymentAtTheMoment.

Mirage Shader Horizon is an independent community project and is not affiliated with, endorsed by, or associated with Mojang, Microsoft, or the Horizon developers. Minecraft is a trademark of Mojang Synergies AB.*

---

## Contributing and feedback

The most useful contribution is a **measurement**. When reporting performance, please include:

- Device and SoC
- Horizon version and graphics API
- Render distance and render resolution
- Which Mirage build you ran (including any benchmark build)
- GPU frame time (ms), not just FPS, and the scene you measured

Bug reports with the loader's error text, and screenshots of dimension-specific problems, help most of all.
