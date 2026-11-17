# blackholesimulator
A real-time, physically grounded black hole you can fly around, fall into, and photograph, running in the browser on WebGPU.
Last Stable Orbit

A real-time, physically grounded black hole you can fly around, fall into, and photograph, running in the browser on WebGPU. Every pixel follows a ray of light through curved spacetime, so what you see is what a camera there would actually record.

It is one self-contained index.html of about 52 KB (22 KB gzipped). There is no engine, no library, and no texture, model or audio file: the starfield, nebulae, accretion disk, station, planet, star and music are all generated in code.

Six viewpoints and the in-app spacetime panel

What you are looking at

A spinning black hole of 4.3 million solar masses, like the one at the centre of our galaxy. Its shadow is about 65 million km across. You start 15,000 km from a 4,000 km ring station; an Earth-sized planet and a Sun-like star orbit nearby, and both are specks next to the hole. On-screen readouts give distances in km and AU, your speed as a fraction of light speed, and how fast your clock runs compared with a distant one.

Physics
Spinning (Kerr) black hole with adjustable spin from 0 to 0.99. Light paths are integrated per pixel as a Hamiltonian flow in Kerr–Schild coordinates, which stay regular across the event horizon, so the camera can be rendered from inside it.
Accretion disk with the Page–Thorne temperature profile of a thin relativistic disk, an inner edge at the innermost stable circular orbit for the current spin, Keplerian rotation, and combined Doppler and gravitational shifts applied to both colour and brightness.
Observers done properly. Hovering uses a zero-angular-momentum frame, so frame dragging carries you (and the station) around. Flying fast applies a Lorentz boost, which produces aberration and the searchlight effect. Pressing G switches to a true free-fall geodesic through the horizon to the inner horizon.
Light-travel time. Everything is drawn as it was when its light left. Each lensed image of a flaring hot spot lights up at its own moment (light echoes), and seen from deep in the well the outside world runs fast-forward.
Relativistic jets with knots moving at 0.9 c and Doppler beaming: the jet aimed at you is bright, the other is dim, and near the axis the knots appear to move faster than light.
Rendering

HDR throughout, with temporal anti-aliasing (Catmull–Rom history), bloom, automatic eye adaptation, filmic tone mapping and dithering. Small objects near the camera (the station and planet) are rendered in camera-relative coordinates so they stay precise at true scale. Resolution can be lowered or supersampled up to 150%.

Controls
Key	Action
W A S D, mouse	Fly and look (click "Take the controls" for pointer lock)
Q / E	Down / up
Shift, mouse wheel	Faster; wheel sets base speed
G	Cut the engines and fall freely; press again to hover
[ / ]	Less / more spin
T	Change how fast time runs
V	Spacetime panel: curved-space funnel and your light cone
L	Labels: what am I looking at?
P	Photo mode: 64 frames averaged, saved as PNG
1 to 6	Jump to a viewpoint
F	Start or stop the 60-second tour
- / =	Lower / raise resolution
M	Sound on / off
H	Frame timings
Esc	Back to the start screen
Running it

Open index.html in a current Chrome or Edge with WebGPU and hardware acceleration enabled. A discrete GPU is recommended.

Useful URL parameters:

Parameter	Effect
?bench	Runs the 60 s scripted fly-through and reports GPU and frame times (average, 99th percentile, max). Press F11 for fullscreen and reload first.
?pose=1..6	Start at a given viewpoint
?spin=0.9	Set the spin
?scale=0.5	Render resolution as a fraction of the screen
?still=16	Freeze time and accumulate 16 jittered frames (used for test renders)
?cam=x,y,z&at=x,y,z	Place the camera (units of the Schwarzschild radius)
Verification

All checks below ran in headless Chrome on a software WebGPU adapter, using the renderer's own shader code.

Test	Result
Shadow size, non-spinning, camera at 30, 10 and 3 rs, vs the exact formula	+0.005%, 0.000%, +0.001%
Shadow shape, spin 0.9 seen edge-on, vs Bardeen's analytic curve	area −0.13%, edges −0.09% / −0.13%, sideways offset −0.16%
Light bending, 1,000 rays at spin 0.9, vs an independent float64 reference integrator	worst error 0.0021° (99th percentile 0.0010°)
In-app tests: free fall through the horizon, labels, photo download, station co-rotation	pass

Performance has not yet been measured on real GPU hardware. The target is 2560×1440 with average GPU frame time ≤ 11 ms and no frame over 16.7 ms; run ?bench to measure it on your machine.

Known simplifications
Disk brightness scales with temperature to the power 2.2 instead of 4, to compress the dynamic range; colours and the approaching/receding asymmetry are unaffected.
The station, planet and star are test bodies on simple circular orbits and do not curve spacetime themselves.
The free fall stops at the inner horizon of a spinning hole, where the physics becomes genuinely uncertain and these coordinates stop following your path.
The jet's shape is artistic; its beaming and apparent superluminal motion are physical.
Roadmap
A star torn apart by tides, feeding a fresh disk
A traversable wormhole to a second universe
Motion-aware anti-aliasing
Sound driven by the physics (orbital frequencies at your radius)
Full-resolution performance measurements and a blind visual review
Credits

Built with the help of Claude (Anthropic).

License

No license has been chosen yet.
