# This is the Largest Header (H1)
MirageShader Horizon

A Performance-First Shader Project for Horizon — Minecraft Java Edition 26.3 (Rust Rewrite)

<p align="center">
  <strong>Look like a path tracer. Cost like Chocapic13 V6 Lite.</strong>
</p>MirageLite is an experimental, performance-oriented shader pack designed for Horizon, the Rust-based reimplementation of Minecraft Java Edition 26.3. Its primary objective is to bring visually sophisticated lighting, atmospheric rendering, and environmental effects to lower-end Android devices without relying on computationally expensive rendering techniques.

The project follows an approximation-first rendering philosophy: instead of calculating every graphical effect through physically accurate or computationally intensive methods, MirageLite attempts to reproduce their visual characteristics through cheaper mathematical models, simplified lighting algorithms, and carefully optimized shader operations.

The inspiration behind this philosophy comes from the remarkable efficiency of Chocapic13 V6 Lite, combined with the visual ambitions demonstrated by more advanced shaders such as Horizon_PathTracer.

MirageLite is not intended to be another conventional shader that achieves visual quality through increasingly expensive rendering operations. It is an attempt to rethink how those visual results can be achieved in the first place.

---

Table of Contents

- "Project Vision" (#project-vision)
- "Core Philosophy" (#core-philosophy)
- "Project Objectives" (#project-objectives)
- "Current Development" (#current-development)
- "Rendering Architecture" (#rendering-architecture)
- "Implemented Rendering Techniques" (#implemented-rendering-techniques)
- "Performance Engineering" (#performance-engineering)
- "Dimension-Specific Rendering" (#dimension-specific-rendering)
- "Environmental Animation" (#environmental-animation)
- "Compatibility and Requirements" (#compatibility-and-requirements)
- "Known Limitations" (#known-limitations)
- "Development Roadmap" (#development-roadmap)
- "Installation" (#installation)
- "Performance Recommendations" (#performance-recommendations)
- "Research and Development Philosophy" (#research-and-development-philosophy)
- "Contributing and Feedback" (#contributing-and-feedback)
- "Project Status" (#project-status)
- "Acknowledgements" (#acknowledgements)

---

1. Project Vision

The central idea behind MirageLite is relatively simple:

What if we could recreate the appearance of advanced rendering techniques without actually implementing their computationally expensive counterparts?

Modern shaders often rely on sophisticated rendering techniques to produce realistic lighting, atmospheric depth, soft shadows, reflections, and other graphical effects. While these approaches can produce impressive results, their computational requirements can make them impractical for devices with limited GPU resources.

MirageLite approaches this problem differently.

Rather than prioritizing physical accuracy, it prioritizes perceptual quality: how convincing an effect looks to the player relative to the amount of processing power required to produce it.

For example:

- Instead of implementing full global illumination, approximate indirect lighting through ambient hemispherical illumination, directional fill lighting, and color-temperature adjustments.
- Instead of implementing expensive real-time reflections, reconstruct environmental reflections analytically using the sky-rendering model.
- Instead of using complex volumetric atmospheric simulations, approximate atmospheric depth through inexpensive exponential and height-based haze.
- Instead of implementing computationally demanding ambient occlusion systems, estimate local occlusion using a small number of depth samples.
- Instead of simulating physically accurate water waves, construct convincing surface movement through combinations of mathematical sine functions.

These are examples of a broader development philosophy: reproduce the visual result, not necessarily the expensive process traditionally used to obtain it.

The ultimate objective is to make sophisticated graphical effects accessible to hardware that would otherwise struggle with conventional high-end shader implementations.

2. Core Philosophy

MirageLite is built around five fundamental principles.

2.1. Perceptual Fidelity Over Physical Accuracy

A rendering technique does not necessarily need to be physically accurate to produce a convincing visual result.

If an approximation can reproduce the important visual characteristics of an effect while requiring substantially fewer resources, it should be considered a legitimate alternative.

2.2. Performance Is a Design Requirement

Optimization should not be something added at the end of development.

Every rendering technique should be evaluated according to its computational cost, memory requirements, sampling overhead, and contribution to the final image.

Performance is part of the architecture itself.

2.3. Minimal Rendering Complexity

Avoid unnecessary rendering passes, redundant calculations, excessive texture sampling, and additional render targets whenever possible.

A smaller rendering pipeline is generally easier to optimize, maintain, and adapt to different hardware configurations.

2.4. Intelligent Approximation

Whenever possible, expensive graphical operations should be replaced with mathematically cheaper alternatives.

This includes analytical lighting models, procedural effects, simplified reflection models, reduced sampling strategies, and temporal or spatial approximations where appropriate.

2.5. Visual Consistency

Performance optimization should not result in a collection of disconnected graphical effects.

Lighting, atmospheric scattering, shadows, reflections, and environmental coloration should work together to create a coherent visual environment.

---

3. Project Objectives

MirageLite is being developed with the following objectives:

Objective| Description
Low-end optimization| Prioritize performance on Android devices with limited GPU capabilities.
Visual fidelity| Deliver convincing lighting, atmosphere, shadows, and environmental effects.
Minimal overhead| Reduce unnecessary rendering operations and resource consumption.
Horizon compatibility| Develop specifically around Horizon's shader-pack architecture.
Vulkan-oriented rendering| Target the graphics environment used by Horizon while respecting its actual shader interfaces.
Dimensional accuracy| Implement separate environmental configurations for the Overworld, Nether, and End.
Extensibility| Maintain an architecture that can accommodate future graphical improvements.
Configurability| Allow demanding effects to be disabled or adjusted independently.

The project is primarily intended for lower-end Android hardware, although compatibility and performance on other supported platforms are also desirable.

4. Current Development

MirageLite is currently in its early experimental development stage.

The initial implementation focuses on establishing a functional rendering pipeline while exploring how advanced graphical effects can be recreated using relatively inexpensive techniques.

The first development milestone is to establish a visually coherent shader with a minimal rendering architecture, followed by iterative optimization and compatibility testing.

Current priorities

1. Performance optimization

The current rendering techniques need further profiling and optimization. Particular attention is being given to shadow sampling, depth-based effects, cloud rendering, god rays, and post-processing operations.

2. Environmental lighting

The existing lighting system needs further refinement, particularly its behavior across different dimensions.

3. Dimension-specific rendering

The current implementation primarily follows Overworld environmental assumptions. Separate Nether and End configurations are planned.

4. Environmental animation

Subtle vegetation movement is being considered as an additional visual feature, provided it can be implemented efficiently within Horizon's available shader interfaces.

5. Compatibility verification

Real-world testing is necessary to verify shader loading, rendering behavior, blending configurations, and performance on actual Android hardware.

5. Rendering Architecture

MirageLite uses a deliberately minimal rendering architecture.

Its initial implementation avoids unnecessary rendering complexity and relies primarily on forward rendering and a small number of post-processing operations.

Current pipeline

             SHADOW PASS
              (Optional)
                  |
                  v
          FORWARD RENDERING
          (Per Draw Operation)
                  |
                  v
             COMPOSITE
        AO / GOD RAYS / BLOOM
                  |
                  v
               FINAL
          Output / Dithering

Architectural characteristics

The current implementation is designed around:

- One primary color buffer.
- An optional shadow-rendering pass.
- Forward rendering rather than deferred rendering.
- Depth-based post-processing.
- No conventional G-buffer.
- No separate normals buffer.
- No compute shader dependency.
- No geometry shader dependency.
- No custom texture-pack dependency.
- No external normal or specular texture requirements.

The objective is to keep the rendering pipeline relatively compact while extracting as much visual quality as possible from the available rendering resources.

Buffer and precision management

The initial implementation uses an 8-bit color buffer with square-root encoding to preserve additional effective precision for selected values.

Output dithering uses Interleaved Gradient Noise (IGN) to help reduce visible banding.

This approach is intended to make more efficient use of limited buffer precision without introducing additional render targets.

6. Implemented Rendering Techniques

The following techniques describe the initial v0.1 implementation. They are approximations of more sophisticated rendering effects, not physically accurate replacements.

Effect| Implementation approach
Global illumination| Sky/ground hemisphere ambient lighting, directional fill light, and warm torch illumination.
Soft shadows| Single distorted 1024-resolution shadow map with four hardware-filtered taps.
Shadow fallback| Lightmap skylight approximation when shadows are disabled or beyond the shadow distance.
Ambient occlusion| Four depth samples in the composite stage.
Sky rendering| Analytic sky gradient, sunset coloration, and procedural celestial effects.
Atmospheric scattering| Three-lobe Mie glow approximation.
Stars| Procedurally generated without external textures.
Clouds| Single-plane procedural clouds using two noise octaves.
Cloud lighting| Additional shifted sampling for approximate self-shadowing and silver lining.
Aerial perspective| Exponential distance haze and height-based atmospheric attenuation.
Water reflections| Analytic sky evaluation using the reflection direction.
Water waves| Three traveling sine functions with analytically calculated slopes.
God rays| Six depth samples, conditionally evaluated toward the sun.
Bloom| Blurred mip levels with a minimal sampling approach.
Underwater rendering| Per-channel absorption and in-scattering.
Dithering| IGN-based output dithering to reduce visible banding.

Why these techniques?

The intention is to obtain the visual characteristics of advanced rendering effects without implementing their most expensive computational requirements.

For instance, MirageLite does not perform screen-space reflection tracing to render water reflections. Instead, it evaluates the analytical sky model along the reflection direction.

Similarly, its global illumination approximation does not attempt to calculate indirect light transport through the entire scene. It uses environmental illumination and directional fill lighting to reproduce some of the visual characteristics of bounced light.

These techniques are deliberately selected to balance visual quality and computational efficiency.

7. Performance Engineering

Performance optimization is one of the most important aspects of MirageLite.

The project specifically targets devices where GPU performance, memory bandwidth, and rendering resources may be considerably limited.

Optimization efforts focus on several areas.

7.1. Sampling Reduction

Minimize texture reads and depth-buffer sampling wherever possible.

Sampling operations should be justified by their actual contribution to the final image.

7.2. Simplified Mathematical Models

Replace unnecessarily complicated calculations with analytical approximations whenever the resulting visual difference is acceptable.

7.3. Conditional Rendering

Effects that are not visually relevant should avoid performing unnecessary work.

For example, god rays should only be evaluated when the camera is oriented toward the sun.

7.4. Render Target Reduction

Avoid introducing additional buffers unless they provide a meaningful visual or computational advantage.

7.5. Resolution-Aware Effects

Future development may explore reduced-resolution rendering for selected post-processing effects, provided the implementation is compatible with Horizon's rendering architecture.

7.6. Configurable Quality

Expensive effects should remain independently configurable so users can prioritize performance over visual complexity when necessary.

The objective is not simply to reduce the number of graphical effects, but to make each effect as efficient as reasonably possible.

8. Dimension-Specific Rendering

One of the planned improvements is to introduce dedicated environmental configurations for Minecraft's three major dimensions.

The current v0.1 implementation primarily follows Overworld lighting and atmospheric assumptions. This results in environmental inconsistencies when entering the Nether and End.

Future versions will address these differences.

Overworld

The Overworld serves as the primary reference environment for MirageLite.

Its rendering configuration includes:

- Day-and-night transitions.
- Sun and moon rendering.
- Atmospheric coloration.
- Procedural clouds.
- Environmental ambient lighting.
- Directional sunlight and shadows.
- Atmospheric perspective.

Nether

The Nether requires an independent environmental lighting model.

Unlike the Overworld, its lighting should not depend on conventional sunlight or a standard celestial day-and-night cycle.

Planned improvements include:

- Dedicated ambient illumination.
- Adjusted environmental brightness.
- Dimension-specific atmospheric coloration.
- Removal of inappropriate Overworld celestial assumptions.
- Better integration with existing block and lightmap information.

The objective is to preserve the Nether's distinctive atmosphere without making the environment unnecessarily dark.

End

The End also requires its own environmental configuration.

Planned improvements include:

- Dedicated sky rendering.
- Removal of inappropriate Overworld sun and cloud behavior.
- Dimension-specific ambient lighting.
- Adaptation of suitable nighttime lighting techniques.
- A distinct atmospheric appearance.

Where possible, existing rendering functions will be reused rather than duplicated.

9. Environmental Animation

Another planned feature is subtle wind-based vegetation movement.

The objective is to introduce additional environmental life without introducing expensive simulation systems.

Potential implementations include:

- Gentle grass movement.
- Subtle leaf displacement.
- Flower and plant animation.
- Time-based vertex displacement.
- Procedural wind variation.

The implementation will depend on the information and rendering interfaces exposed by Horizon.

In particular, block identification and vertex attribute availability may determine how accurately individual vegetation types can be animated.

This feature is planned rather than implemented in the initial release.

10. Compatibility and Requirements

MirageLite is designed specifically for Horizon's shader-pack environment.

Target environment

- Game: Horizon.
- Game compatibility target: Minecraft Java Edition 26.3.
- Engine implementation: Rust.
- Rendering target: Horizon's supported graphics pipeline.
- Primary platform: Android.
- Primary hardware target: Lower-end mobile devices.

Rendering dependencies

The initial implementation does not require:

- Compute shader stages.
- Geometry shader stages.
- Custom texture packs.
- External normal maps.
- External specular maps.
- Conventional deferred-rendering buffers.

Actual graphics API compatibility depends on Horizon's shader loader, supported shader language, and rendering interfaces.

Important compatibility note

MirageLite is an experimental shader pack. Compatibility with all supported devices and graphics configurations has not yet been established.

11. Known Limitations

The following limitations are known in the initial v0.1 release.

Area| Current limitation
In-game testing| Not yet verified in an actual Horizon game session during initial development.
Dimensions| Nether and End currently reuse Overworld environmental assumptions.
Vegetation| No grass or leaf movement.
Block identification| No reliable per-block emissive identification.
Torch lighting| Artificial illumination is approximated using block light and brightness information.
Water identification| Water and glass differentiation relies on available biome tint information.
Bloom| Depends on the loader honoring "colortex0MipmapEnabled".
Fog blending| Separate-alpha blending behavior may vary depending on loader support.

These limitations are documented so that users and contributors understand the current state of the project.

12. Development Roadmap

MirageLite is intended to evolve through incremental development rather than introducing numerous expensive features at once.

The following roadmap represents development intentions, not guaranteed release dates.

Phase 1 — Initial Foundation

- [x] Establish the initial shader architecture.
- [x] Implement forward rendering.
- [x] Implement approximate global illumination.
- [x] Implement basic shadow mapping.
- [x] Implement depth-based ambient occlusion.
- [x] Implement procedural sky and clouds.
- [x] Implement analytical water reflections.
- [x] Implement atmospheric perspective.
- [x] Implement basic post-processing.
- [ ] Complete real-world Horizon testing.

Phase 2 — Performance and Stability

- [ ] Identify the most expensive rendering operations.
- [ ] Optimize shadow sampling.
- [ ] Reduce unnecessary texture reads.
- [ ] Improve post-processing efficiency.
- [ ] Investigate cheaper alternatives for expensive visual effects.
- [ ] Verify buffer precision and blending behavior.
- [ ] Establish consistent performance benchmarks.
- [ ] Improve compatibility across supported Android hardware.

Phase 3 — Environmental Accuracy

- [ ] Implement dedicated Nether lighting.
- [ ] Implement dedicated End lighting.
- [ ] Correct dimension-specific sky behavior.
- [ ] Improve environmental ambient illumination.
- [ ] Refine atmospheric effects.
- [ ] Improve water and transparency interactions.

Phase 4 — Environmental Enhancements

- [ ] Investigate lightweight vegetation animation.
- [ ] Introduce procedural wind movement.
- [ ] Improve cloud appearance and lighting.
- [ ] Refine water surface behavior.
- [ ] Explore additional inexpensive atmospheric effects.

Phase 5 — Architectural Refinement

- [ ] Research alternative rendering techniques.
- [ ] Investigate shader implementations designed for Vulkan and similar rendering environments.
- [ ] Evaluate more efficient mathematical approximations.
- [ ] Revisit the rendering architecture where meaningful performance improvements are possible.
- [ ] Improve modularity and maintainability.
- [ ] Expand quality and performance configuration options.

13. Installation

1. Download the MirageLite release ZIP file.
2. Do not extract the archive.
3. Place the ZIP file inside Horizon's shaderpacks directory.
4. Launch Horizon.
5. Open the shader-pack selection menu.
6. Select MirageLite.
7. Apply the shader pack.

The shader pack is designed to be loaded directly from its ZIP archive.

If the shader fails to load, please provide the complete loader error message when reporting the issue.

---

14. Performance Recommendations

If MirageLite performs poorly on your device, the following settings can help reduce rendering overhead.

Recommended order of adjustment:

1. Disable "SHADOWS".
2. Disable "GODRAYS".
3. Disable "CLOUDS".
4. Disable "AMBIENT_OCCLUSION".
5. Reduce the rendering resolution scale through Horizon's graphics settings.

If the shader options menu is unavailable, the corresponding definitions can be adjusted in:

"shaders/lib/settings.glsl"

These adjustments are intended to help identify which effects have the greatest impact on performance.

---

15. Research and Development Philosophy

MirageLite is influenced by the broader shader-development community, particularly projects that demonstrate how sophisticated graphical effects can be achieved through efficient rendering techniques.

Chocapic13 V6 Lite is a particularly important source of conceptual inspiration because of its performance-oriented approach to shader development.

Other shader projects, including Horizon_PathTracer, Photon, and various OpenGL ES and Vulkan-based implementations, provide useful examples of different rendering techniques and visual approaches.

However, MirageLite is developed around independently implemented rendering solutions.

The development process emphasizes:

1. Identifying a desirable graphical effect.
2. Understanding how conventional implementations produce that effect.
3. Investigating alternative mathematical and rendering techniques.
4. Evaluating their computational requirements.
5. Developing an independent implementation.
6. Comparing visual quality and performance.
7. Refining the implementation based on measurable results.

The objective is to learn from existing rendering research without treating expensive implementations as the only possible solutions.

A visually convincing approximation can be more valuable than a physically accurate implementation that exceeds the available hardware budget.

---

16. Contributing and Feedback

MirageLite is an experimental project, and feedback from actual users is especially valuable during its early development.

If you encounter problems, please provide as much relevant information as possible.

Useful information includes:

- Horizon version.
- Android device and GPU.
- Graphics API and rendering configuration.
- Shader-pack version.
- Rendering resolution scale.
- Chunk render distance.
- Enabled graphical effects.
- Approximate performance or frame rate.
- Screenshots or recordings.
- Complete shader-loader error messages.

Bug reports

Please distinguish between:

- Shader compilation errors.
- Shader loading failures.
- Visual rendering bugs.
- Dimension-specific problems.
- Performance issues.
- Device-specific compatibility problems.

Detailed reports make it easier to identify whether an issue originates from the shader source, the rendering pipeline, or the environment in which it is running.

---

17. Project Status

Current release: v0.1

Development stage: Experimental / Early Development

Primary target: Low-end Android devices running Horizon.

Testing status: Static validation performed; real-world in-game verification remains necessary.

MirageLite should currently be considered an experimental implementation rather than a fully validated, production-ready shader.

Features and compatibility may change as Horizon's rendering interfaces and the project itself continue to evolve.

---

18. Acknowledgements

MirageLite is an independently developed project informed by ideas and techniques from the shader-development community.

Special conceptual inspiration comes from:

- Chocapic13 V6 Lite — For its performance-first rendering philosophy and efficient approach to visual effects.
- Horizon_PathTracer — For demonstrating the visual possibilities of advanced rendering techniques within Horizon.
- Photon — For additional inspiration and reference when exploring shader rendering techniques.

No source code from Chocapic13, SEUS, or Photon is included in MirageLite.

All shader implementations are developed independently. References to other projects acknowledge conceptual and technical inspiration, not code reuse or endorsement.

---

<p align="center">
  <strong>MirageLite</strong><br>
  <em>Advanced-looking visuals. Minimal computational cost.</em>
</p>
