# Tiny Turbo — updated master prompt

Build a complete, playable 3D browser game called “Tiny Turbo.”

It is a three-lane endless driving game in which a tiny toy car races across household surfaces. Everyday objects become oversized obstacles: pencils, mugs, books, building blocks, and cereal boxes.

If a concept image is attached, use it as the visual reference. If none is attached, proceed from this written direction: a coral-red toy car with a white racing stripe, a warm wooden kitchen table, blue tape marking the track, cereal-ring collectibles, and ruler ramps. Do not block implementation because an image is missing.

Deliver working code and a playable preview, not just a plan or static mockup.

## CORE EXPERIENCE

The car drives forward automatically. Players switch lanes, jump low obstacles, collect cereal rings, use boosts, and try to beat their best distance.

Make the game:
- Immediately understandable.
- Responsive on keyboard and touch.
- Challenging but fair.
- Charming and visually coherent.
- Smooth on ordinary laptops and mid-range phones.

Prioritize a complete, reliable gameplay loop before adding decorative complexity.

## PRIORITY FIXES FROM THE RECORDED BUILD

Build these requirements into the first implementation:
1. Wheels roll around their own axles in the correct direction; they must not spin sideways, wobble, or orbit the car.
2. The environment scrolls smoothly without persistent lag or repeated stutters during normal play, boosts, recycling, or zone transitions.
3. The controls and power-up UI sit in a compact right-side corner layout and never block the car or the approaching driving path.

These are acceptance requirements based on problems observed during the recorded build. Do not assume that a visually plausible first render proves they are solved.

## TECHNICAL IMPLEMENTATION

- Use React, TypeScript, and Three.js or React Three Fiber.
- Use the build setup supported by this environment.
- Generate initial assets with procedural geometry and materials.
- Do not require downloaded models, paid assets, API keys, accounts, or a backend.
- Structure gameplay, rendering, UI, generation, audio, and persistence into understandable modules.
- Keep high-frequency gameplay updates outside React state.
- Use a fixed physics timestep with an accumulator and a bounded number of substeps.
- Handle rendering independently from physics.
- Prevent large time jumps after tab switching.
- Use reusable road sections and object pools.
- Keep the player near the world origin by moving the environment or periodically recentering it.
- Clean up event listeners, animation loops, and resources correctly.

## ART DIRECTION

Aim for a stylized miniature toy world, not a photorealistic racing simulator.

Visual qualities:
- Warm wood, soft daylight, matte plastic, paper, rubber, and ceramic.
- A coral-red toy car with a white stripe and chunky black wheels.
- Simple, readable shapes and restrained textures.
- Blue masking tape defining three clearly readable lanes.
- Large household objects establishing the tiny scale.
- Gentle atmospheric depth without hiding hazards.
- Natural contact shadows.
- A slightly elevated third-person chase camera.

The concept image is a mood and composition reference. Create an efficient real-time interpretation rather than attempting expensive photographic detail.

Avoid:
- Neon overload.
- Excessive bloom.
- Heavy motion blur.
- Distracting particles.
- Branded household packaging.
- Random clutter inside the playable path.
- Visual effects that hide obstacles.

## PLAYER CAR

Create a recognizable toy car with:
- Rounded or beveled body shapes where practical.
- Four visible wheels.
- Dark windows.
- A white racing stripe.
- Small headlights and taillights.
- A believable toy-scale silhouette.

Separate gameplay and visuals:
- playerRoot controls lane position and physical jump height.
- carVisual controls body lean, pitch, and suspension movement.
- Collision bounds are explicit and independent of decorative geometry.
- Keep the visual model replaceable with a GLB later.

Rotate the wheels with travel speed. Lean the body subtly during lane changes. Do not allow visual animation to alter collision dimensions unpredictably.

## WHEEL ROTATION — REQUIRED CORRECTION

- Document the car's local forward, up, and lateral axes before building the wheel hierarchy.
- Each wheel must spin around its local axle through its center. Keep the axle pivot separate from the initial orientation of the wheel mesh.
- For example, if the car's longitudinal direction is local Z and its axles run along local X, spin the axle pivots around local X. Orient the wheel geometry to that axle once; do not blindly animate a mesh axis without accounting for its initial orientation.
- Derive wheel rotation from signed virtual distance traveled divided by wheel radius. The sign must visually match forward rolling.
- If the car stays near the origin while the environment moves, use gameplay travel distance, not the car's nearly stationary world position.
- Do not rotate the entire car, move wheel centers, or spin around world origin to simulate rolling.
- Keep left and right wheels visually consistent even when their meshes or orientations are mirrored.
- Keep steering yaw, body lean, suspension, and axle rotation in separate transforms so they do not overwrite each other.
- Freeze wheel rotation when paused or stopped. Reset its state consistently on restart.
- Verify the wheels from a temporary side or three-quarter debug view as well as the chase camera. Remove intrusive debug helpers from the normal game.
- Check slow driving, maximum speed, boost, lane changes, airborne movement, pause/resume, and restart. Do not claim the issue is fixed based on code inspection alone.

## CONTROLS

Desktop:
- A / Left Arrow: move one lane left.
- D / Right Arrow: move one lane right.
- Space / Up Arrow: jump.
- Shift / N: activate boost.
- Escape / P: pause or resume.

Mobile:
- Swipe left or right to change lanes.
- Swipe up to jump.
- Accessible on-screen steering, jump, and boost buttons.
- One gesture triggers one action.
- Classify diagonal swipes by their dominant direction.
- Prevent page scrolling inside the active game area.

Inputs must respond immediately. Clamp movement to the outer lanes and prevent accidental double jumps from held keys.

## MOVEMENT AND JUMPING

- Start at an approachable speed.
- Increase difficulty gradually to a reasonable maximum.
- Animate lane changes smoothly using the same timing assumptions used by obstacle generation.
- Permit steering while airborne.
- Use vertical velocity and gravity for jumps.
- Tune the jump to last around 0.8–1.0 seconds and clear low obstacles comfortably.
- Allow jumping only while grounded.
- Include a small input buffer so a press just before landing can trigger after landing.
- Add subtle takeoff pitch, landing compression, and rebound.
- Use a small dust puff on takeoff and landing.
- Avoid large camera shakes.

Pause must freeze movement, jump physics, timers, particles, and scoring.

Restart must reset all movement and effect state.

## TRACK AND WORLD

Use reusable tabletop segments with coherent decoration placement.

Create three zones:

1. Breakfast Table
- Warm wood.
- Cereal boxes, mugs, napkins, and cereal rings.
- Pencils and small building blocks as hazards.
- Ruler ramps.

2. Study Desk
- Notebooks, erasers, pen holders, and sticky notes.
- Low stationery obstacles.
- Book stacks outside the track.
- Cardboard ramps.

3. Playroom Floor
- Wooden flooring.
- Toy blocks, toy boxes, and large puzzle pieces.
- Simple toy tunnels with sufficient clearance.
- Playful color accents.

Maintain the same three-lane coordinate system across zones. Transition scenery and surface materials smoothly. Do not introduce unsupported table-edge drops or gaps between zones.

Place large decorative props outside the playable corridor unless they are deliberate, validated obstacles.

## FAIR OBSTACLE GENERATION — CRITICAL

Never generate an unavoidable crash.

Every obstacle sequence must preserve at least one reachable driving route that does not require jumping, boost, a shield, or another power-up.

An empty lane is not sufficient if the player cannot reach it in time.

Implement a reachability-based generator:
- Represent obstacles by lane occupancy, longitudinal interval, and height.
- Track reachable lanes through successive upcoming intervals.
- Account for player speed, car length, obstacle length, lane-change duration, and a reaction-time buffer.
- Validate lane transitions themselves, not only their endpoints.
- Reject transitions that cross occupied space.
- Consider obstacles already on the track when placing new ones.
- Account for speed increases while approaching hazards.
- Never force two lane changes in less time than the movement system permits.
- Validate patterns across segment boundaries.
- Use bounded generation retries.
- If no candidate passes validation, insert a simple pattern or recovery gap.

Do not solve fairness by making the track empty. Use varied combinations of:
- Single-lane obstacles.
- Staggered obstacles with sufficient spacing.
- Safe optional jump shortcuts.
- Coin trails along alternative valid routes.
- Occasional recovery sections.

Introduce each obstacle type gently before combining it with others.

## OBSTACLES AND RAMPS

Include:
- Pencils: low obstacles that can be jumped.
- Erasers: medium obstacles.
- Building blocks: tall obstacles that must be avoided.
- Small book stacks: clearly blocked lanes.
- Ruler and cardboard ramps: optional jump opportunities.

Ramp behavior:
- A ramp launches the car only when the grounded car enters its defined trigger.
- Trigger each ramp once per encounter.
- Use the same jump physics system with a tuned launch velocity.
- Generate a safe landing zone after every ramp.
- Do not launch an already airborne car again.
- Make ramps visually distinct from barriers.

Avoid moving hazards in the first implementation unless the generator can validate their future positions.

## COLLISIONS

Use height-aware collisions:
- Low obstacles can be cleared only when the car’s underside is above their top.
- Tall obstacles remain dangerous during jumps.
- Check ascent, flight, descent, and landing.
- Use swept collision checks or bounded substeps to prevent tunneling.
- Use slightly forgiving collision bounds.
- Trigger an unprotected crash only once.
- Show a brief, readable impact response before game over.

Jumping does not grant invincibility.

## COLLECTIBLES AND SCORING

Use cereal rings as the main collectible.

- Give them a gentle floating or rotating animation.
- Arrange them in deliberate trails.
- Use an appropriate 3D pickup distance.
- Collect each ring only once.
- Make trails follow valid routes.
- Place optional airborne trails over safe ramps.
- Use a small pickup effect and optional sound.

Score combines:
- Distance traveled.
- Collected rings.
- Small bonuses for successful ramp landings.

Define scoring constants clearly and make scoring frame-rate independent.

Track separately:
- Current run score.
- Distance.
- Rings collected this run.
- Best score.
- Total spendable rings.

Commit run earnings exactly once when a run ends, including an explicit restart or exit. Do not duplicate rewards across repeated button presses or state changes.

## POWER-UPS

Implement three simple power-ups:

1. Ring Magnet
- Attracts nearby cereal rings for a limited duration.
- Shows an icon and countdown.
- Cannot collect the same ring twice.

2. Bubble Shield
- A subtle transparent bubble around the car.
- Absorbs one collision.
- Gives a brief grace period after breaking so the same obstacle cannot immediately kill the player.

3. Wind-Up Boost
- A meter charges through normal play.
- Activation briefly increases speed and score gain.
- Gives collision protection during the boost.
- Adds small speed lines and a restrained toy-like effect.
- Smoothly returns to normal speed.
- Keeps ramp landing zones safe throughout the boost.

Specify refresh behavior for repeated pickups. Use caps rather than unlimited stacking. Freeze all durations while paused.

## GAME STATES

Use clear states:
- Loading.
- Menu.
- Playing.
- Paused.
- Game over.

Prevent invalid transitions and duplicate animation loops.

When the browser becomes hidden:
- Pause automatically.
- Suspend or mute gameplay audio appropriately.
- Require explicit resume when the user returns.

## INTERFACE

Use a playful but clean interface:
- Warm cream, charcoal, coral, and blue accents.
- Readable typography.
- Rounded buttons with clear focus and pressed states.
- Minimal HUD.
- No visual clutter over the driving path.

## CONTROL PLACEMENT — REQUIRED CORRECTION

- Do not place keyboard hints, jump/boost buttons, power-up cards, or instruction panels directly above the player car in the lower-center driving view.
- On desktop, group control hints, boost information, and active power-up indicators in a compact upper-right corner panel. Keep the score and distance at the upper left, and leave the central driving corridor unobstructed.
- Keep the right-side panel inside the viewport and clear of the pause button. It must remain compact when several power-ups are active.
- On touch devices, put the action buttons in a compact lower-right control cluster, respecting safe-area insets. Steering buttons must remain reachable without covering the projected car position; swipes remain available across the unobstructed game area.
- Use responsive layouts, not fixed screen coordinates that only work at one resolution. Reflow for narrow portrait screens and short landscape screens.
- Keep button labels and touch targets usable; do not solve overlap by making the controls illegibly small.
- Passive HUD areas must not intercept gameplay gestures. Interactive controls should receive only the input intended for them.
- Define a protected screen-space area around the car and the near driving corridor, including its lane-change and jump positions. Keep persistent controls out of that area.
- Verify desktop, mobile portrait, and mobile landscape screenshots with boost and power-up indicators visible. Confirm that the car and approaching obstacles remain readable.

Start screen:
- Tiny Turbo title.
- A live or static view of the toy car and tabletop.
- Play.
- Garage.
- How to Play.
- Settings.
- Best score.

Gameplay HUD:
- Score.
- Distance.
- Run rings.
- Boost meter.
- Active power-ups.
- Pause button.

Pause:
- Resume.
- Restart.
- Settings.
- Return to menu.

Game over:
- Score.
- Best score.
- Distance.
- Rings earned.
- New-record message when appropriate.
- Play Again.
- Garage.
- Menu.

## GARAGE

Include four cosmetic paint options:
- Coral Racer: unlocked by default.
- Ocean Blue.
- Lemon Sprint.
- Mint Mini.

Use earned rings to unlock the additional colors.

- Show a rotatable or clearly visible car preview.
- Display prices and locked/unlocked states.
- Disable purchases when funds are insufficient.
- Prevent duplicate purchases.
- Persist ownership and selection locally.
- Keep every car mechanically equivalent.
- Change only intended paint surfaces, preserving tires, windows, and stripe materials.

## AUDIO

Use simple locally generated Web Audio effects:
- Ring pickup.
- Jump.
- Landing.
- Shield break.
- Boost.
- Crash.
- Button confirmation.

Initialize audio only after user interaction.

Provide a working sound toggle. Include a music toggle only if music is actually implemented.

Keep volume comfortable. Avoid harsh or repetitive sounds.

## ACCESSIBILITY AND RESPONSIVENESS

- Support desktop, tablet, portrait mobile, and landscape mobile.
- Respect mobile safe areas.
- Use large touch targets.
- Make menus keyboard operable.
- Provide visible focus.
- Label icon buttons accessibly.
- Maintain readable contrast.
- Do not rely only on color for important states.
- Respect prefers-reduced-motion and provide a remembered override.
- Reduce camera movement and particles in reduced-motion mode.
- Do not trap keyboard focus.
- Provide a clear fallback message when WebGL is unavailable.

## LIGHT AND DARK THEMES

Implement light, dark, and system themes for menus and supporting pages.

- Remember the user’s selection.
- Use accessible controls.
- Preserve gameplay readability.
- The theme does not need to change the physical lighting of the game world.

## SUPPORTING WEBSITE PAGES

Outside active gameplay, include:
- Responsive navigation.
- Footer with the current year.
- About.
- Contact.
- Privacy Policy.
- Terms and Conditions.
- Working links back to the game.

Write content specific to the actual implementation.

Privacy must explain:
- Local browser storage for preferences, scores, currency, and cosmetic unlocks.
- Whether any data leaves the browser.
- No analytics or tracking unless actually added.
- Clearing browser data removes local progress.

Do not invent:
- An operator’s legal name.
- An email address.
- A business address.
- A jurisdiction.
- A support inbox.
- Cloud saves or online accounts.

If real operator/contact details are unavailable, identify them as launch requirements while completing the game. Do not create a fake contact form or claim messages are delivered.

Do not add ads, purchases with real money, analytics, or account registration.

## PERSISTENCE

Use a small versioned localStorage record.

- Validate loaded data.
- Handle missing or corrupted records.
- Clamp invalid values.
- Recover gracefully when storage is unavailable.
- Save settings and purchases at appropriate events.
- Avoid writing every frame.
- Explain that progress is saved only in the current browser.

## PERFORMANCE

- Pool track segments, obstacles, rings, and effects.
- Use instancing for repeated objects where useful.
- Keep polygon counts and material counts modest.
- Limit shadow-casting objects and shadow distance.
- Cap rendering pixel ratio.
- Avoid unnecessary allocations in the game loop.
- Avoid loading an entire household environment.
- Ensure recycled objects fully reset.
- Keep a stable memory footprint over long runs and repeated restarts.

## ENVIRONMENT PERFORMANCE — REQUIRED CORRECTION

Do not treat corrected wheel animation as proof that environment lag is fixed. Investigate and verify both separately.

- Profile CPU frame work, rendering cost, draw calls, geometry/material counts, and allocation or garbage-collection spikes before choosing an optimization.
- Record the viewport, device pixel ratio, browser, hardware when available, and quality settings with each measurement.
- Aim for approximately 60 FPS on a typical laptop and a stable playable experience around 30 FPS or better on a representative mid-range phone. These are targets to test, not guaranteed results or claimed measurements.
- Maintain only the visible track plus a bounded buffer ahead and behind. Recycle sections instead of continuously growing the scene.
- Reuse geometries and materials; instance repeated props, rings, and obstacles when appropriate. Do not create new meshes, materials, textures, vectors, or arrays unnecessarily every frame.
- Prewarm object pools. Recycling must reset transform, collision, visibility, collection, and effect state without producing a noticeable pause.
- Cull or hide distant decorative props and avoid simulating off-screen effects unnecessarily.
- Limit real-time lights and shadow casters. Keep shadow maps and their camera coverage modest; use lightweight contact-shadow approximations when appropriate.
- Cap device pixel ratio and offer a lower-quality fallback that reduces resolution, shadows, decorative density, and particles while preserving gameplay rules and obstacle readability.
- Avoid expensive full-screen post-processing. Do not add bloom, motion blur, or high-detail procedural materials before the performance budget is verified.
- Keep physics and world scrolling frame-rate independent. Use interpolation where needed to avoid visible stepping between fixed physics updates.
- Avoid per-frame React state updates for simulation. Refresh HUD values at a modest cadence or when meaningful values change.
- Avoid synchronous storage writes, repeated layout reads/writes, and unbounded logging in the hot path.
- Keep world coordinates bounded to avoid precision jitter as distance increases.
- Ensure mounting, pausing, resuming, and restarting cannot create duplicate animation loops or listeners.
- If automatic quality adjustment is added, use averaging and hysteresis so quality does not oscillate rapidly.
- Measure a sustained run after warm-up, including a boost, segment recycling, and a zone transition. Compare frame-time distributions before and after fixes; do not rely only on an average FPS counter.
- If access to a real phone or profiler is unavailable, document that limitation and provide the checks to run later. Do not claim "lag-free" or verified device performance without evidence.

## BUILD ORDER

1. Working car, track, camera, lane switching, and verified wheel-axis animation.
2. Jump physics and height-aware collisions.
3. Fair procedural generation.
4. Rings, scoring, and game-over/restart flow.
5. Ramps and power-ups.
6. Garage and persistence.
7. Zones, audio, and visual polish within a measured performance budget; verify right-corner controls before adding decorative effects.
8. Supporting pages and accessibility.
9. Verification and fixes.

Complete each stage before expanding scope.

## VERIFICATION

Use focused automated tests where supported for:
- Generator reachability across many seeded sequences.
- Starting, intermediate, and maximum speeds.
- Long obstacles and segment boundaries.
- Lane changes across occupied intermediate space.
- Safe ramp landing zones.
- Jump behavior at different rendering frame rates.
- Single collection per ring.
- Shield and boost interactions.
- Pause/resume and hidden-tab behavior.
- Exactly-once run earnings.
- Purchases and insufficient funds.
- Corrupted or unavailable storage.
- Restarting during a jump or boost.
- Wheel-angle progression for a known travel distance and wheel radius, independent of render frame rate.
- Wheel transforms remaining centered on their axles during steering, jumping, and body lean.
- Object-pool reuse without stale collision, collection, or effect state.
- No duplicate animation loop after repeated start/pause/resume/restart sequences.

Manually verify when the environment permits:
- Keyboard controls.
- Touch controls and diagonal gestures.
- Responsive layouts.
- Menu navigation and focus.
- Loading and error states.
- Page routes and internal links.
- Sound toggles.
- Repeated restarts.
- Long-run performance.
- Correct wheel rotation in side/three-quarter and gameplay views, including boost and pause.
- Environment scrolling without repeated frame-time spikes during recycling or zone changes.
- Right-corner controls that do not overlap the car or near driving corridor at supported viewport sizes.
- HUD layout with multiple power-ups and the boost indicator visible at the same time.

Keep a short CHECKLIST.md separating:
- Completed work.
- Remaining launch checks.
- External blockers.

Record actual test evidence in QA.md. Put optional future ideas in a separate backlog. Do not claim tests or device checks that were not performed.

## VISUAL DEBUGGING WORKFLOW

When a screenshot or gameplay recording is available:
- Inspect the complete evidence before proposing a fix.
- For wheel defects, compare wheel orientation and motion against the car's travel direction.
- For control overlap, identify the obstructed car/road area and check the UI at the supplied viewport.
- For lag, inspect the relevant sequence and collect actual runtime measurements where possible. A screen recording alone does not establish the root cause.
- Explain the observed symptom, the likely cause supported by evidence, the specific change, and how it was verified.
- Re-test all three priority issues after a fix. A wheel correction must not be reported as resolving unrelated performance or layout problems.
- Preserve working gameplay, scoring, persistence, and controls while making targeted fixes.

## FINAL DELIVERY

Provide:
- Complete working source.
- A playable preview.
- Controls.
- A concise list of implemented features.
- Actual verification results.
- Remaining limitations or launch requirements.

Do not publish the game or send real emails.

If the coding environment is in Plan Mode, produce an implementation plan and concrete verification steps first, then wait for the user's implementation instruction. Once implementation is authorized, start building. Make reasonable implementation decisions, keep the game playable throughout development, and iterate on observed failures until the required checks pass or a real external limitation is documented.