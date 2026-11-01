MirageLite v0.1 - a Horizon (Rust Minecraft 26.3) shader pack for low-end Android
==================================================================================
Idea: look like a path tracer, cost like Chocapic13 Lite. Every "expensive" effect is faked.

INSTALL
  Put this zip (unextracted) in Horizon's shaderpacks folder and select it in the shader pack menu.
  It does not need flat draws, custom textures, normal/specular maps, compute shaders or geometry stages.

WHAT IS FAKED, AND HOW
  Global illumination ... sky/ground hemisphere ambient + a fill light opposite the sun (colour bounce) + warm torch curve
  Soft shadows .......... 1 shadow map (1024, distorted), 4 hardware-filtered taps; beyond 72 blocks and when SHADOWS is off,
                          skylight in the lightmap stands in for shadows
  Ambient occlusion ..... 4 depth taps in the composite (no normals buffer, no G-buffer at all)
  Sky + atmosphere ...... analytic gradient, sunset band, 3-lobe Mie glow, sun/moon discs, procedural stars (zero textures)
  Clouds ................ one plane, 2 noise octaves, 2nd tap shifted toward the sun for self-shadowing and silver lining
  Aerial perspective .... exponential + height haze tinted by the sky colour in that direction
  Water reflections ..... reflect vector -> analytic sky. No screen-space tracing. Sun glitter comes free from the sky's sun lobe
  Water waves ........... 3 travelling sines with analytic slope
  God rays .............. 6 depth taps, only evaluated when looking toward the sun
  Bloom ................. blurred mip levels, 3 texture reads
  Underwater ............ per-channel absorption + in-scatter
  Precision ............. one 8-bit buffer (sqrt-encoded, x4 headroom), IGN dither on output to hide banding

PASSES: shadow (optional) + one forward pass per draw + composite + final. One colour buffer. No other render targets.

KNOWN LIMITS / UNTESTED
  - NOT run in-game: I could not run Horizon here. Static checks only (includes resolve, braces balance, no undefined smoothstep).
    Expect a first-launch fix or two; send me the loader's error text.
  - Overworld only. Nether/End use the overworld sky colours and sun lighting.
  - Without block IDs: no leaf/grass sway, no per-block emissives (torches etc. are guessed from block light + brightness),
    water is told from glass by its biome tint.
  - Bloom relies on the loader honouring colortex0MipmapEnabled; if not, bloom looks sharper than intended (turn BLOOM off).
  - Alpha holds skylight for fog. If the loader ignores the separate-alpha blend lines in shaders.properties,
    fog inside caves behind water/particles may be slightly off.

IF IT IS STILL TOO SLOW (biggest wins first)
  1. SHADOWS off  2. GODRAYS off  3. CLOUDS off  4. AMBIENT_OCCLUSION off  5. lower the port's resolution scale
  All in shaders/lib/settings.glsl (comment out the #define lines) if the options screen is unavailable.

Written from scratch. No code from Chocapic13, SEUS, or Photon is included.
