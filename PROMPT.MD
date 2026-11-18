Build "Last Stable Orbit": a real-time, physically grounded black hole you can fly
around, fall into, orbit, and land beside, running in the browser.

PRIORITIES (in this order when they conflict)
1. Visual quality
2. Physical correctness of light paths and observers
3. Stable frame pacing at the target spec
4. Small bundle size

CONSTRAINTS
- WebGPU only. No Three.js/Babylon.js or any other engine or library.
- Everything procedural: no texture, model or audio files. The deliverable is a
  single self-contained index.html. Aim under 100 KB minified; report raw and
  gzipped size.
- Zero per-frame JS allocations in your own code. Preallocate all buffers. The
  only exceptions are WebGPU's own per-frame objects (encoder, passes, command
  buffer, swap-chain view); report any others.
- The scene is a deterministic function of camera state and simulation time.

TARGETS
- Primary: Windows 11, current Chrome, RTX 5070 Ti, 2560x1440 at full resolution.
  GPU frame time <= 11 ms average, no frame > 16.7 ms during a scripted 60 s
  fly-through.
- Secondary: phones and tablets with WebGPU (recent Chrome on Android, Safari on
  iOS 26 or later), with automatic resolution scaling.

UNITS AND SCALE
- Schwarzschild radius rs = 1 (so M = 1/2), c = 1. The hole has 4.3 million
  solar masses: rs = 12.7 million km, one time unit rs/c = 42.4 s. Show real
  units (km, AU, fraction of c) in all readouts.
- The scene must read as enormous: small objects are tiny next to a shadow about
  65 million km across. Render objects near the camera in camera-relative
  coordinates computed in float64 on the CPU, so they stay precise at true scale.

SPACETIME AND LIGHT
- Spinning (Kerr) black hole, spin adjustable from 0 to 0.99, default 0.8.
- Every pixel integrates a null geodesic as a Hamiltonian flow
  (H = 1/2 g^uv p_u p_v) with RK4 in Kerr-Schild Cartesian coordinates, which are
  regular across the event horizon so the camera can be rendered from inside it.
  Use analytic gradients in the shader. Track coordinate time along each ray.
- The camera is a tetrad. Build its 4-velocity from the observer mode (below),
  build spatial legs by Gram-Schmidt against the metric, and launch each ray's
  covariant momentum from the local view direction. Aberration and Doppler must
  follow automatically.
- Capture rule: a ray is captured at the horizon, or when it moves inward below
  the innermost (prograde) photon orbit, from where no light returns. Rays
  launched inside the horizon are not captured until they have exited.
- Escape at large radius (about 3000 rs) along the final direction. Shift sky
  colour and brightness by g = 1/p_t (g^4, colour temperature times g).
- Everything is drawn at its retarded time: emission time = camera time + the
  ray's (negative) travel time. Lensed images of a flaring source therefore light
  up at different moments (light echoes).
- Everything is lensed by the same rays. Nothing may be drawn unlensed over a
  lensed background, except camera-relative near-field objects where the bending
  across that range is below a pixel.

ACCRETION DISK AND PLASMA
- Thin volumetric disk from the spin-dependent ISCO out to 14 rs, prograde
  Kerr circular orbits (angular velocity and u^t from the Kerr formulas).
- Temperature from the Page-Thorne flux profile (zero at the ISCO), peak about
  9,000 K so it glows in visible light. Brightness g^4 times T^2.2 (disclose the
  compressed exponent), observed temperature T*g with
  g = 1 / (k . u_emitter).
- Colour: a cinematic gold-orange grade by default, toggled with C to true
  blackbody colour. Say which is which in the UI.
- Structure that flows forever without winding up: shearing annuli of noise that
  each rotate rigidly at their own angular velocity and crossfade, a co-moving
  wobble that breaks rings into filaments, and a third noise layer that evolves
  in time so the disk boils.
- Violence:
  - orbiting hot spots, the innermost of which flares periodically;
  - magnetic flares that erupt from the inner disk, rise, swell and fade;
  - thin streams of gas spiralling in between the ISCO and the horizon;
  - a hot, thin plasma glow around the inner disk.
  All are Doppler shifted.
- Relativistic jets along the spin axis: knots and occasional bright blobs at
  0.9 c, positioned at retarded time (so they can appear superluminal), with
  Doppler beaming ~D^3. Cap the beaming of the jet aimed straight at the camera
  so it never hides the shadow, and disclose the cap.

SKY
- The galactic core seen from its centre: a bright band all round the sky, a
  warm bulge glow, filamentary dark dust lanes, pink emission nebulae, a faint
  blue reflection haze, and a dense multi-layer starfield of varied colour and
  brightness. Anti-alias stars and nebulae by ray footprint, not just TAA.

OBJECTS
- Ring station: about 4,000 km across, derelict, a broken ring with hub, spokes,
  panels and a few warm window lights. It orbits on a circular prograde orbit at
  16 rs, inclined 14 degrees, about 18% of c, tidally locked with one face to the
  hole. It spins about its axis, is lit by the disk (soft wrap for the extended
  source, hard shadows, Fresnel rim), and emits a faint radio beacon.
- Earth-sized planet (radius 6,371 km) on a circular orbit at 45 rs, inclined
  12 degrees, tidally locked, so the black hole hangs fixed in its sky. From afar
  it is a footprint-widened point of light. Up close, ray-march it in planet
  radii and camera-relative coordinates:
  - terrain: continents plus ridged, snow-capped mountains (up to about 12 km),
    ice caps, and a frozen night side;
  - oceans: drifting waves, Fresnel reflection, and a glitter path from the disk;
  - a cloud layer;
  - a single-scattering Rayleigh and Mie atmosphere (Earth-like coefficients and
    scale heights) lit by the disk direction, which reddens and dims the hole
    near the horizon and casts the planet's shadow.
  Expose the same height function on the CPU (sampling the same noise data) for
  ground collision.
- Sun-like star (radius about 0.055 rs) on a circular equatorial orbit at 38 rs:
  limb-darkened, granulated, Doppler shifted, timed so it sits exactly behind the
  hole at viewpoint 5 and forms an Einstein ring.

OBSERVER MODES
- Hover: the zero-angular-momentum observer, boosted by your velocity in its
  frame (up to 0.97 c). Speed scales so you cover about half the distance to the
  nearest object per second; the mouse wheel sets base speed, Shift multiplies.
  You cannot hover closer than just outside the horizon.
- Ride along (orbit matching): within about 76,000 km of the station or
  127,000 km of the planet, the ship matches that body's orbit automatically, or
  on O. You move relative to the body in its rotating frame, with the 4-velocity
  built from the orbital motion plus yours. Near the planet the camera's "up" is
  the local vertical. Speed scales with altitude, from hundreds of km/s high up
  to walking pace near the ground, and the ground stops you 1.2 km up. O or G
  releases you into free fall beside the body.
- Free fall (G): integrate the timelike geodesic on the CPU in float64 with
  adaptive substeps. Check horizon crossing and the end condition every substep,
  not once per frame, because at high time rates one frame can cover the whole
  interior. Reset to the start immediately on reaching the singularity
  (spin 0) or the inner horizon (spin > 0, where the coordinates stop following
  the path), with a message giving the proper time since crossing.
- Time rate presets x11, x42, x169 (default), x678 real time. The camera's
  coordinate time advances by u^t times its proper time, so the outside world
  runs fast-forward deep in the well.

INTERFACE
- Start screen with title, a short description that conveys the scale, a "Take
  the controls" button, a spin slider, a time-rate menu, and a resolution menu:
  Auto, 35% to 100%, and 125% or 150% supersampled.
  - Each control explains itself on hover. Time must say plainly what fast-forward
    means.
  - The start screen must stay usable on a landscape phone: scrollable, nothing
    cut off.
- Spacetime panel (V): a curved-space funnel with horizon and ISCO rings and your
  position, and your light cone in the t-r plane (tipping inward near the
  horizon, both edges inward inside it) with your worldline. Underneath:
  distance, speed, clock rate versus a distant clock, mode, spin, time rate, and
  the nearest object or your altitude.
- Labels (L): name, size and distance of the hole, disk, jet, station, planet and
  star, with dark outlines for legibility, flipping side near screen edges.
- Photo mode (P): freeze time, average 64 jittered frames, save a PNG.
- Toast messages for every state change.
- Frame timings (H): GPU timestamp queries.
- Sound (M): an original procedural score built on the first interaction, with
  no audio files:
  - slow organ chords through a long reverb;
  - pitch and tempo that sink with the gravitational redshift at your radius;
  - a rumble that grows near the hole and a roar inside the disk;
  - sparse bell tones;
  - the station's beacon, panned to its side.
- Mobile mode, automatic on touch screens:
  - drag on the left to fly and on the right to look;
  - buttons for Up, Down, Boost, Fall, View (cycles viewpoints) and Tour, plus a
    top bar for Labels, Panel, Photo, Match and Menu;
  - fullscreen on start, panel hidden by default, adaptive resolution, a reduced
    step budget.

CONTROLS
WASD move, mouse look (pointer lock, with drag-to-look fallback), Q/E down/up,
Shift faster, wheel base speed, O match/release orbit, G fall/hover,
Z/X zoom, [ ] spin, T time rate, C colour grade, V panel, L labels, P photo,
1-7 viewpoints, F 60 s tour, -/= resolution, M sound, H timings, Esc menu.

RENDERING PIPELINE
HDR ray-trace pass, then:
- temporal AA with Catmull-Rom history and a neighbourhood clamp;
- a mip-chain bloom;
- eye adaptation from the log-average of the smallest bloom level;
- filmic tone map, vignette and dithering.

VIEWPOINTS (fixed camera and time; used by the critic and the tour)
1. Riding with the station, 15,000 km from it, at 16 rs. The hole looms on the
   left, and the station's lit face is visible on the right. Recompose until both
   are fully in frame.
2. Edge-on to the disk at 20 rs.
3. Looking down the spin axis at 25 rs.
4. Hovering at 2 rs, looking along the orbit.
5. The star directly behind the hole, forming an Einstein ring.
6. Beside the planet in close orbit: a lit crescent and the hole in the same
   frame.
7. On the planet's shore, the hole about 15 degrees above the ocean horizon.

VERIFICATION
- Physics gate (automated, with debug flags that hide the disk and objects and
  make the sky white):
  a) Non-spinning shadow size from a static camera at 30, 10 and 3 rs must match
     sin(alpha) = (3*sqrt(3)/2)(1/r)sqrt(1 - 1/r) within 1%. Measure the
     sub-pixel area of the shadow over accumulated jittered frames.
  b) Spinning shadow shape at spin 0.9, edge-on, camera at 1000 rs: area, both
     horizontal edges and the sideways offset within 1% of Bardeen's analytic
     curve. The flattened edge must be on the approaching side.
  c) Light bending: write an independent float64 reference that shares no
     formulas with the renderer (inverse metric as an explicit 4x4 matrix,
     forces by central differences, RK4 with small steps). For 1,000 escaping
     rays at spin 0.9, with half of them aimed to pass close to the hole and
     each ray still escaping with 5% less sideways motion, the renderer's own
     shader code (in a compute pass) must agree within 0.1 degree. Report the
     error binned by closest approach.
  d) The approaching side of the disk is measurably brighter and bluer. Report
     the ratio.
- Functional tests in headless Chrome:
  - free fall through the horizon to the reset, with messages in order;
  - photo download;
  - labels drawn;
  - orbit matching;
  - landing collision;
  - touch joystick and buttons in an emulated phone.
- Visual critic: a separate subagent that sees ONLY your renders and the
  descriptions above, never your code. Give it the 7 viewpoints at 2560x1440
  plus a recording of the tour, and have it return prioritized defects. Iterate
  until nothing is above "minor".
- Performance: ?bench runs the 60 s tour through viewpoints 1-6 at full
  resolution and reports GPU and frame times (average, 99th percentile, max)
  with PASS/FAIL. Also measure the planet surface separately; it is the most
  expensive view.

KNOWN PITFALLS (avoid these)
- A WGSL diagnostic directive must come before every declaration in the
  combined shader module.
- Test cameras must bypass the world's outer distance limit. Otherwise the
  camera is silently moved and every measurement is wrong.
- In these coordinates, light grazing the horizon spirals around it many times.
  Finer steps there without the capture rule exhaust the step budget and leak
  light into the shadow.
- The remaining bending beyond the escape radius depends on the coordinates. Push
  the escape radius out and let the bending test tell you if it matters.
- A hovering derelict station is impossible; it must orbit.
- Viewpoint compositions drift once objects orbit, so compose in each body's
  rotating frame.

RULES OF ENGAGEMENT
- Never loosen a gate or a viewpoint to make it pass. If a gate cannot be met,
  stop and report what blocks it, with measurements.
- Report only numbers you measured on real hardware. If you cannot run the
  target spec, say so instead of estimating.
- Deliver working builds in stages, in this order:
  1. light paths and the shadow gates;
  2. disk, sky and post-processing;
  3. observers and the free fall;
  4. station, star and orbit matching;
  5. the planet;
  6. interface, sound and mobile;
  7. polish against the critic.
  Run the physics gates again after any change to light-path code.

DONE WHEN: all verification gates pass. Deliver index.html, a README (features,
controls, URL parameters, verification table, known simplifications), the bundle
size, the physics report, the performance report, the 7 final renders and the
critic's final review.

STRETCH (only after DONE): a star torn apart by tides feeding a fresh disk; a
traversable wormhole to a second universe; a probe that reddens and freezes at
the horizon; mass presets with tidal forces.
