# Pool Shot Lab

A browser-based American pool shot simulator built to the refined specification.

## Run it

Open `pool-shot-lab.html` in a current desktop browser (Chrome, Edge, Firefox or Safari). There is no build step and no required dependency. Google Fonts (Barlow) is loaded if available; otherwise system fonts are used.

If your browser restricts local files, serve the folder instead:

```
python3 -m http.server 8000
# then open http://localhost:8000/pool-shot-lab.html
```

## Structure

The single file contains four separable parts, each in its own `<script>` block:

- **Physics core** (`PoolPhysicsCore`): deterministic, event-driven engine.
  - Table and pocket geometry, cue strike, ball–ball and cushion models.
  - Exact motion between events and polynomial contact-time solving.
  - Self-contained, so it can be serialised into a Web Worker.
- **Search engine** (`PoolSolverCore`): suggested-shot search.
  - Written as a generator, so it runs in a Web Worker or time-sliced on the main thread.
- **Renderer**: Canvas 2D drawing of the table, paths, markers, magnifier and minimap.
- **Scenario editor and UI**: layout editing, shot controls, analysis, variants, sweeps, validation and file handling.

## Several target balls

You can set up to three target balls, shot in order. Use **Shot > Targets > Add target**; it adds a new ball if none is free.

- **Target 1** is this shot.
- **Targets 2 and 3** are remaining targets. For each shot the simulator reports whether they are hit, moved or pocketed.
- **Next shot rating:** from where the cue ball stops, it rates the shot on the next target as good, playable, hard or blocked.
  - This is a geometric estimate covering cut angle, pocket angle, distance and blocking balls; spin and throw are ignored.
  - On the table it appears as a green, amber or red line.
- **Suggest** can rank shots by position for the next target, and can avoid disturbing the remaining targets.
- **Continue to ball n**, below the table, keeps the result and makes the next target active, once a shot pockets target 1.

Try the preset **Three-ball run-out: position for the next target**. Scenario JSON is now version 2, which adds `remainingTargets`; version 1 files still import.

## Validation

Open the **Checks** tab and choose **Run all checks**. All 19 checks pass on the reference run. They cover:

- table and pocket scale, and placement rules;
- lossless head-on transfer and 90° stun separation;
- stop, follow and draw from spin at impact;
- cushion speed and spin dependence;
- pocket capture and jaw rejection;
- the double-kiss example and its avoidance alternative;
- prediction and playback agreement, and frame-rate independence;
- frozen contacts, tunnelling and energy;
- export/import reproducibility through the real importer, including rejection of an invalid file, and cushion-integration convergence;
- remaining-target disturbance and next-shot blocking;
- ball–ball friction consistency (sticking means zero slip) and momentum in pressing contacts;
- preview time, and solver reproduction.

## Changes after code review

- **Pressing contacts:** touching balls pushed together by spin no longer stop instantly. The contact is integrated in 0.2 ms sub-steps with inelastic impulses and friction, which conserves the momentum exchanged between them. Transient overlap stays below 1 µm.
  - A detection gap was also fixed: balls separating extremely slowly after a contact are now caught when they close again.
- **Ball–ball friction:** the friction impulse now uses the same effective masses as the velocity update (no vertical ball motion), so "sticking" means the contact slip really reaches zero.
- **Residual slip:** slip below tolerance is now removed with a dissipative cloth-friction impulse instead of overwriting the spin.
- **Suggestions:**
  - Results are marked "Outdated" when any search input changes, including the targets, pockets, table, physics, elevation or settings.
  - The re-check now uses the search's own setup.
  - Finish-zone wording only appears when a zone exists.
- **Search coverage:** nine tip positions (including follow and draw combined with sidespin), four speeds, and local refinement of tip and speed around the best candidates. Searches now take roughly 3–5 s in the background worker.
- **Import:** scenarios are fully validated before anything changes, so an invalid file leaves the current setup untouched.
- **Not changed:** pocket capture is still the documented simplification. A ball is captured when its centre crosses the slate-hole edge, and the drop is not simulated.

## Calibration status

Dimensions follow the WPA equipment specification. Friction, restitution, squirt and cushion coefficients are literature-typical or assumed values, labelled "uncalibrated" in the Physics tab. No quantitative accuracy is claimed until they are fitted to measured trajectories. See **Checks > Model notes** for every model, approximation and unsupported case.
