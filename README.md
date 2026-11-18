# Last Stable Orbit

A real-time, physically grounded black hole you can fly around, fall into, orbit, and land beside, running in the browser on WebGPU. Every pixel follows a ray of light through curved spacetime, so what you see is what a camera there would record.

The whole project is one self-contained `index.html` of about 69 KB (29 KB gzipped). There is no engine, no library, and no texture, model or audio file: the starfield, galaxy, accretion disk, station, planet, star and music are all generated in code.

**Live demo:** https://thelaststableorbit.edgeone.dev/

## What you are looking at

A spinning black hole of 4.3 million solar masses, like the one at the centre of our galaxy. Its shadow is about 65 million km across.

You start riding alongside a 4,000 km ring station that orbits at 16 Schwarzschild radii, about 18% of the speed of light. Further out, an Earth-sized planet orbits with one face always toward the hole, and a Sun-like star circles nearby; both are specks next to the shadow. On-screen readouts give distances in km and AU, your speed as a fraction of light speed, your altitude, and how fast your clock runs compared with a distant one.

## Physics

- **Spinning (Kerr) black hole** with adjustable spin from 0 to 0.99. Light paths are integrated per pixel as a Hamiltonian flow in Kerr–Schild coordinates, which stay regular across the event horizon, so the camera can be rendered from inside it.
- **Accretion disk** with the Page–Thorne temperature profile of a thin relativistic disk, an inner edge at the innermost stable circular orbit for the current spin, and combined Doppler and gravitational shifts applied to colour and brightness.
- **A violent environment.** The disk churns rather than just turning, magnetic flares erupt from its inner edge, rise and fade, and gas spirals into the hole in thin streams inside the last stable orbit. A hot, thin plasma glows around the inner disk.
- **Light-travel time.** Everything is drawn as it was when its light left, so each lensed image of a flaring hot spot lights up at its own moment (light echoes), and from deep in the well the outside world runs fast-forward.
- **Relativistic jets** with knots and bright blobs moving at 0.9 c and Doppler beaming: the jet aimed at you is bright, the other dim, and near the axis the knots appear to move faster than light.
- **Observers done properly:**
  - Hovering uses a zero-angular-momentum frame, so frame dragging carries you around.
  - Flying fast applies a Lorentz boost, which produces aberration and the searchlight effect.
  - Riding with the station or planet matches its orbit, so you see the sky as an orbiting observer does.
  - Cutting the engines follows a true free-fall path through the horizon. The fall resets at the singularity, or at the inner horizon of a spinning hole, where the physics becomes genuinely uncertain.

## The planet

Fly within about 127,000 km and your ship matches the planet's orbit automatically; then fly down. Your speed scales with altitude, from hundreds of km/s high up to walking pace near the ground.

The planet is tidally locked, so the black hole hangs fixed in its sky like a permanent sunset. It has continents with ridged, snow-capped mountains, ice caps and a frozen night side, oceans with drifting waves and a glitter path, a cloud layer, and an atmosphere that scatters the disk's light and reddens the hole near the horizon. Viewpoint 7 puts you on its shore.

## Rendering

HDR throughout, with temporal anti-aliasing, bloom, automatic eye adaptation, filmic tone mapping and dithering. The sky is the core of a galaxy: a bright band all around, dark dust lanes, and pink emission nebulae. The default colour is a cinematic grade; press C for true blackbody colour. Small objects near the camera are rendered in camera-relative coordinates, so they stay precise at true scale, down to waves on the planet's sea. Resolution can be lowered or supersampled up to 150%.

## Controls

| Key | Action |
|---|---|
| W A S D, mouse | Fly and look (click "Take the controls" for pointer lock) |
| Q / E | Down / up |
| Shift, mouse wheel | Faster; the wheel sets base speed |
| O | Match the orbit of the station or planet nearby, or let go |
| G | Cut the engines and fall freely; press again to hover |
| Z / X | Zoom in / out |
| [ / ] | Less / more spin |
| T | Change how fast time runs |
| C | Cinematic or true blackbody colour |
| V | Spacetime panel: curved-space funnel and your light cone |
| L | Labels: what am I looking at? |
| P | Photo mode: 64 frames averaged, saved as PNG |
| 1 to 7 | Jump to a viewpoint (7 stands on the planet) |
| F | Start or stop the 60-second tour |
| - / = | Lower / raise resolution |
| M | Sound on / off |
| H | Frame timings |
| Esc | Back to the start screen |

**On phones and tablets** a touch mode switches on automatically: drag on the left to fly and on the right to look. Buttons cover climb, dive, boost, fall, orbit matching, viewpoints, the tour, labels, the panel, and photos. Resolution adapts automatically.

**Hover over Spin, Time and Resolution** on the start screen for an explanation. Time is fast-forward: at ×169 each real second shows almost 3 minutes at the black hole.

## Running it

Open `index.html` in a current Chrome or Edge with WebGPU and hardware acceleration enabled, or use the live demo. A discrete GPU is recommended. On phones you need recent Chrome on Android, or Safari on iOS 26 or later.

Useful URL parameters:

| Parameter | Effect |
|---|---|
| `?bench` | Runs the 60 s scripted fly-through and reports GPU and frame times (average, 99th percentile, max). Press F11 for fullscreen and reload first. |
| `?pose=1..7` | Start at a given viewpoint |
| `?spin=0.9` | Set the spin |
| `?scale=0.5` | Render resolution as a fraction of the screen |
| `?mobile`, `?desktop` | Force touch or desktop mode |
| `?still=16` | Freeze time and accumulate 16 jittered frames (used for test renders) |
| `?cam=x,y,z&at=x,y,z` | Place the camera (units of the Schwarzschild radius) |

## Verification

These checks ran in headless Chrome on a software WebGPU adapter, using the renderer's own shader code. The light-path code they test has not changed since.

| Test | Result |
|---|---|
| Shadow size, non-spinning, camera at 30, 10 and 3 rs, vs the exact formula | +0.005%, 0.000%, +0.001% |
| Shadow shape, spin 0.9 seen edge-on, vs Bardeen's analytic curve | area −0.13%, edges −0.09% / −0.13%, sideways offset −0.16% |
| Light bending, 1,000 rays at spin 0.9, vs an independent float64 reference integrator | worst error 0.0021° (99th percentile 0.0010°) |
| In-app tests: free fall through the horizon, labels, photo download, touch controls | pass |

Not yet verified: flying down to land interactively, and the newest effects (flares, plunging gas, orbit matching) in motion or on a real phone.

**Performance has not yet been measured on real GPU hardware.** The target is 2560×1440 with average GPU frame time ≤ 11 ms and no frame over 16.7 ms; run `?bench` to measure it on your machine. The planet surface is the most expensive view.

## Known simplifications

- The default colour is a cinematic grade rather than true blackbody colour (press C to compare).
- The disk's temperature is chosen to glow in visible light; a real disk around a hole this size would peak in the ultraviolet.
- Disk brightness scales with temperature to the power 2.2 instead of 4.
- The shapes of the plasma glow, flares and jets are artistic; their beaming and motion are physical. The beaming of the jet aimed straight at you is capped so it does not hide the hole.
- The station, planet and star follow simple circular orbits without the slow wobble a spinning hole would add, and do not curve spacetime themselves. Riding with them is station-keeping alongside the orbit.
- A real planet this close would be cooked by the disk's X-rays.
- Anti-aliasing does not yet account for camera movement, so very fast flight near the station can smear.

## Roadmap

- A star torn apart by tides, feeding a fresh disk
- A traversable wormhole to a second universe
- Drop a probe and watch it redden and freeze at the horizon
- A laser pointer whose beam bends, and can circle the hole
- Twin clocks: compare how much you aged against the station
- Mass presets (our galaxy's hole, M87*, a 10-sun hole) with tidal forces
- A telescope mode that shows the hole as the Event Horizon Telescope does
- Motion-aware anti-aliasing
- Sound driven by the physics
- Video capture
- Full-resolution performance measurements and a blind visual review

## Credits

Built with the help of Claude (Anthropic).

## License

No license has been chosen yet.
