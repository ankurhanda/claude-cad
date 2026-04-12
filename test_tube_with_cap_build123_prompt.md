# Test Tube with Screw Cap — Build123d CAD Prompt

## Goal
Create a **clear plastic test tube** with a **matching screw cap**. The cap must fit the tube correctly and screw on smoothly. Model the two parts as separate solids, but dimension them as a matched pair. The overall style should match a small sample tube: a long slender transparent body with a rounded closed end, an externally threaded neck, and a short purple cap with vertical grip ribs.

---

## General modeling requirements
- Units: **millimeters**
- Build as **two separate parametric parts**:
  1. `test_tube`
  2. `cap`
- Keep all major dimensions exposed as variables.
- Use clean constructive geometry with revolved profiles for axisymmetric parts.
- Use **revolved ring thread approximations** (triangular cross-section revolved 360°), not helical sweeps. Helical sweeps are fragile in build123d and often fail or run slowly. Ring threads are robust and visually convincing.
- **Internal and external thread rings must be offset by half a pitch** so they interleave rather than collide.
- Ensure the cap fully mates with the tube.
- Add realistic clearances so the cap fits rather than intersecting.
- Skip fragile operations (fillets, chamfers, face-based sketches on complex solids) unless strictly necessary. These are the most common source of build123d failures on complex geometry.

---

## Visual description of the test tube
The tube is:
- long and narrow
- cylindrical for most of its body
- closed at one end with a smooth rounded dome
- open at the top
- threaded externally near the opening
- transparent and thin-walled
- graduations omitted (fragile to model, low visual value at this scale)

The cap is:
- short
- cylindrical
- slightly wider than the test tube neck
- internally threaded
- closed on top
- externally ribbed with many fine vertical grip lines
- proportioned so that when fully screwed on, it covers the neck and sits naturally on the tube

---

## Suggested dimensions

### Tube main body
- Outer diameter of cylindrical body: **16.0**
- Wall thickness: **1.2**
- Inner diameter of body: **13.6**
- Straight cylindrical body length: **88.0**
- Closed-end dome radius: **8.0**
- Overall tube length including dome and neck: about **108 to 112**

### Tube neck and thread region
- Neck outside major diameter: **18.0** (radius `neck_major_r = 9.0`)
- Neck minor diameter under thread: **16.4** (radius `neck_minor_r = 8.2`)
- Threaded length: **10.0**
- Lead-in: **0.5 mm** unthreaded gap before first ring (built into Z offset formula)
- Chamfers and fillets: **omitted** (fragile on this geometry; add inside `try/except` if desired)

### Cap
- Cap outer diameter: **22.0**
- Cap overall height: **17.0 to 18.0**
- Top thickness: **2.2 to 2.8**
- Internal cavity sized to cover the threaded neck
- Internal thread matched to tube thread with clearance
- Ribbed grip band around most of the cap height

### Thread parameters
Use matching **revolved ring** threads on tube and cap (not helical):
- Pitch: **2.0**
- External thread radial depth on tube: **0.8** (use 85% of full depth for truncated crests → 0.68 effective)
- Internal thread clearance relative to tube thread: **0.15 radial minimum**
- Number of turns: **4** (computed as `int((thread_len - 1) / thread_pitch)`)
- Each ring is a triangular cross-section (3 points) revolved 360° around Z
- **Critical: internal rings must be offset by half a pitch** relative to external rings so they interleave. Without this offset, the ring crests overlap radially and the parts cannot mate.

---

## Part 1: test tube

### Shape intent
The test tube should look like a common clear plastic lab sample tube:
- thin-walled
- smooth
- nearly constant body diameter
- gently rounded closed bottom
- open threaded neck at top

### Construction plan
1. In a single `BuildPart` context, create the **outer solid** by sketching a closed half-profile on `Plane.XZ` and revolving around `Axis.Z`:
   - The profile is a closed loop: up the centerline (axis) from `(0, 0)` to `(0, z_top)`, right to `(neck_minor_r, z_top)`, down the neck to `(neck_minor_r, z_body_end)`, step inward to `(tube_or, z_body_end)`, down the body to `(tube_or, dome_cz)`, then a `RadiusArc` back to `(0, 0)` with radius `tube_or` (positive radius gives the correct quarter-circle dome).
2. In the same `BuildPart`, create the **inner cavity** as a second sketch on `Plane.XZ` and revolve with `mode=Mode.SUBTRACT`:
   - Closed loop: from `(0, dome_cz - tube_ir)` up the axis to `(0, z_top + 0.5)` (overshoot ensures open top), right to `(tube_ir, z_top + 0.5)`, down to `(tube_ir, dome_cz)`, then `RadiusArc` back to start with radius `tube_ir`.
3. Extract the part with `tb.part`.
4. Add external thread rings as separate `BuildPart` + revolve operations, fused with `+`.
5. Do **not** attempt fillets or chamfers on the resulting solid — they are fragile on geometry this complex.

### Detailed geometric guidance
- The bottom dome uses a `RadiusArc` in the XZ profile with radius equal to `tube_or`. The arc center ends up at `(0, dome_cz)`. Positive radius in `RadiusArc` selects the shorter (90°) arc — this is correct for the dome.
- The neck base radius (`neck_minor_r = 8.2`) is slightly larger than the body radius (`tube_or = 8.0`), creating a 0.2 mm step at the body-to-neck transition. This is acceptable.
- The inner cavity has its dome center at the same Z as the outer dome (`dome_cz`) but with radius `tube_ir`, maintaining uniform wall thickness through the dome.
- The inner cavity overshoots the tube top by 0.5 mm (`z_top + 0.5`) to ensure a clean open top after subtraction.

### External thread rings
- Each thread ring is a **separate `BuildPart`** containing a triangular profile sketched on `Plane.XZ` and revolved around `Axis.Z`.
- Profile: 3-point `Polyline` with `close=True`:
  - `(neck_minor_r, z - pitch * 0.22)` — root, below center
  - `(neck_minor_r + depth * 0.85, z)` — truncated crest
  - `(neck_minor_r, z + pitch * 0.22)` — root, above center
- Ring center Z positions: `z_body_end + 0.5 + pitch * (0.5 + i)` for `i` in `range(n_turns)`.
- Each ring is fused onto the tube with `test_tube = test_tube + ring_b.part`.
- The 0.5 mm offset from `z_body_end` provides a short unthreaded lead-in.

### Graduations
Omitted. Shallow cosmetic marks are fragile to model and add little visual value at this scale. Can be added later if needed.

---

## Part 2: cap

### Shape intent
The cap should look like a small plastic screw cap for a sample tube:
- cylindrical body at reduced radius (`cap_body_r = cap_or - rib_depth`), with ribs extending to full `cap_or`
- closed on top (solid top of `cap_top_t` thickness)
- internally threaded with ring approximations (half-pitch offset from external rings)
- vertically ribbed on the outside for finger grip (36 ribs via `PolarLocations`)
- proportioned to cover the tube neck and part of the upper body

### Construction plan
1. In a single `BuildPart` context:
   a. Create the main body as a `Cylinder(cap_body_r, cap_h, align=(C, C, MIN))` where `cap_body_r = cap_or - rib_depth` (10.5 mm). The body is intentionally undersized — ribs will extend to the full `cap_or`.
   b. Subtract the bore: `Cylinder(cap_bore_r, cap_cav_h + 0.05, align=(C, C, MIN), mode=Mode.SUBTRACT)`. This hollows from the bottom, leaving the top solid.
   c. Add grip ribs using nested `Locations` + `PolarLocations`:
      ```
      with Locations((0, 0, 0.5 + rib_h / 2)):
          with PolarLocations(cap_body_r + rib_depth / 2, rib_count):
              Box(rib_depth, rib_w, rib_h)
      ```
      The `Locations` shifts the Z center; `PolarLocations` distributes and rotates each rib radially. Each `Box(rib_depth, rib_w, rib_h)` has its depth axis pointing radially outward (handled by PolarLocations rotation).
2. Extract the part with `cb.part`.
3. Add internal thread rings as separate `BuildPart` + revolve operations, fused with `+`.
4. Skip fillets — they fail on the complex rib geometry.

### Detailed cap geometry
- `cap_body_r = cap_or - rib_depth = 10.5`. Ribs protrude from 10.5 to 11.0, giving the full 22 mm OD silhouette.
- `cap_bore_r = neck_major_r + thread_clearance = 9.15`. The bore clears the tube's thread crests.
- `cap_cav_h = cap_h - cap_top_t = 14.5`. This is the bore depth; the remaining 2.5 mm is solid top.

### Internal thread rings
- Same triangular-ring technique as external, but **pointing inward** (crest toward centerline):
  - `(cap_bore_r, z - pitch * 0.22)` — root at bore surface
  - `(cap_bore_r - depth * 0.85, z)` — truncated crest (inward)
  - `(cap_bore_r, z + pitch * 0.22)` — root at bore surface
- **Half-pitch offset**: ring center Z = `pitch * (1.25 + i)`, not `pitch * (0.75 + i)`. This offsets them 1.0 mm (half pitch) from the external rings so the crests interleave rather than collide.
- Each ring is a separate `BuildPart` fused with `cap = cap + irb.part`.

### Grip ribs
- Rib count: **36**
- Rib depth: **0.5** (extends from `cap_body_r` to `cap_or`)
- Rib width: `2 * pi * cap_body_r / rib_count * 0.45` (45% of circumferential spacing)
- Rib height: `cap_h - 2.0` (1.0 mm margin top and bottom)
- Created via `PolarLocations` in `BuildPart` — this is the most robust approach for uniform radial patterns in build123d.

---

## Fit requirements

### Assembly relationship
The cap must fit the tube:
- threads must match in pitch and handedness
- cap inner thread must clear tube outer thread
- cap depth must be enough to fully engage the threaded neck
- when tightened, the cap should stop in a realistic position without passing through the tube

### Clearances
Use practical CAD clearances:
- radial thread clearance: **0.15** (applied to `cap_bore_r = neck_major_r + 0.15`)
- Resulting radial clearances when assembled:
  - external crest to internal root: **0.27 mm**
  - internal crest to external root: **0.27 mm**
- Axial clearance between interleaved rings: **0.12 mm** (half-pitch minus two half-spans)
- avoid perfectly coincident mating surfaces

### Threading guidance
For a robust generated CAD model:
- Use **revolved ring approximations**, not helical sweeps. Each thread ring is a triangular cross-section (`Polyline` with 3 points, `close=True`) sketched on `Plane.XZ` and revolved 360° around `Axis.Z`.
- Use 85% of full thread depth for the crest (`depth * 0.85`) to keep crests slightly truncated and avoid sharp tips.
- Each ring's axial span is `±pitch * 0.22` from its center Z.
- **Do not use** `Helix` + `sweep` — this approach is fragile in build123d (profile orientation at helix start is unpredictable, and sweep often fails or produces self-intersecting geometry).
- **Internal rings must be offset half a pitch** from external rings. If external rings sit at `z_base + pitch * (0.5 + i)`, internal rings must sit at `z_base + pitch * (1.25 + i)` (in cap-local coordinates). Without this offset the ring triangles overlap radially and the parts cannot assemble.

---

## Build123d-specific guidance

### Preferred strategy
- Use `BuildPart` + `BuildSketch(Plane.XZ)` + `BuildLine` + `revolve(axis=Axis.Z)` for all axisymmetric forms.
- Build the tube outer and inner profiles as **two separate sketch-revolve operations** within a single `BuildPart` context (second revolve with `mode=Mode.SUBTRACT`). Do not use `Shell` — it is unreliable on complex geometry.
- Use **revolved triangular rings** for threads (separate `BuildPart` per ring, fused with `+`). Do not use `Helix` + `sweep`.
- Use `PolarLocations` + `Box` inside `BuildPart` for the cap ribs.
- Keep the model fully parametric so diameters, pitch, wall thickness, and lengths can be edited easily.

### Critical build123d API notes
- **`Compound` constructor**: `Compound([list])` (positional) silently produces an empty compound with 0 children, losing all geometry and colors. Always use `Compound(children=[list])` (keyword argument). This is the single most common cause of missing colors or geometry in multi-part models.
- **`RadiusArc(start, end, radius)`**: Positive radius selects the shorter arc. For the dome, `RadiusArc((tube_or, dome_cz), (0, 0), tube_or)` gives the correct quarter-circle. The arc center ends up at `(0, dome_cz)`.
- **Colors**: Set `.color` after all boolean operations. Colors survive `Pos`/`Rot` transforms. Use `Color(r, g, b, a)` with floats 0–1. Values are exported as linear-space in GLTF; the viewer handles the sRGB conversion.
- **`PolarLocations(radius, count)`** in `BuildPart`: rotates each child so its local X points radially outward. So `Box(depth, width, height)` has `depth` in the radial direction automatically.

### Suggested parameter names
- `tube_od`, `tube_wall`, `tube_body_len`
- `neck_major_r`, `neck_minor_r` (use radii, not diameters — avoids `/2` everywhere)
- `thread_pitch`, `thread_len`, `thread_depth` (derived: `neck_major_r - neck_minor_r`)
- `cap_od`, `cap_h`, `cap_top_t`
- `thread_clr` (radial clearance)
- `rib_count`, `rib_depth`
- Derived: `tube_or`, `tube_ir`, `cap_or`, `cap_body_r`, `cap_bore_r`, `cap_cav_h`
- Z coordinates: `dome_cz`, `z_body_end`, `z_top`, `n_turns`

### Modeling order
1. Define all parameters and derived dimensions.
2. `BuildPart`: revolve outer profile, then revolve inner cavity with `mode=Mode.SUBTRACT`.
3. Loop: create external thread rings (separate `BuildPart` + revolve each), fuse onto tube.
4. Set tube color.
5. `BuildPart`: cap body cylinder, subtract bore, add ribs via `PolarLocations`.
6. Loop: create internal thread rings (half-pitch offset), fuse onto cap.
7. Set cap color.
8. Position parts for display using `Pos`/`Rot` transforms.
9. `Compound(children=[tube, cap])` — **keyword argument is mandatory**.
10. `render("model", result)`.

---

## Appearance targets
- Tube: `Color(0.72, 0.88, 0.96, 0.45)` — translucent light blue (clear plastic look)
- Cap: `Color(0.6, 0.2, 0.8)` — vivid opaque purple
- Colors must be set **after** all boolean operations (fusing thread rings etc.) or they are overwritten.
- After `Pos`/`Rot` transforms, re-set the color on the transformed copy to be safe.
- The assembled pair should resemble a small lab sample container.

---

## Constraints to preserve
- Tube body must remain slender and cylindrical
- Bottom must remain rounded, not flat
- Tube wall must remain hollow and manufacturable
- Cap must be wider than the tube neck
- Cap must visibly screw onto the tube rather than merely press-fit
- The cap and tube should look like parts from the same product

---

## What to avoid
- Do not make the tube bottom flat
- Do not make the cap a plain smooth cylinder without ribs
- Do not leave the cap unthreaded
- Do not create zero-clearance mating threads
- Do not use excessively aggressive thread depth that self-intersects
- Do not make proportions chunky or toy-like
- **Do not use `Helix` + `sweep`** for threads — use revolved ring approximations
- **Do not use `Compound([list])`** — always use `Compound(children=[list])`
- **Do not use `Shell`** — use a second revolved profile with `mode=Mode.SUBTRACT`
- **Do not attempt fillets/chamfers** on the final complex solid — they almost always fail
- **Do not place internal and external thread rings at the same Z positions** — they must be offset by half a pitch to interleave

---

## Expected result
A clean parametric CAD model of:
1. a **clear rounded-bottom threaded test tube**
2. a **purple ribbed screw cap**
3. a matched assembly where the cap fits the tube correctly

The result should be suitable for:
- CAD visualization
- 3D printing after minor tolerance adjustment
- downstream conversion to STEP/STL
- Build123d scripting and refinement

---

## Optional enhancement ideas
- Add embossed volume numbers (use extruded text on the tube body — fragile, test carefully)
- Add a sealing inner plug on the cap underside
- Add a tamper band variant
- Try helical threads if the ring approximation is insufficient (use `Helix` + `sweep` with `BuildSketch(helix ^ 0)` for profile orientation — but expect failures)
- Add fillets to the body-neck transition (try `fillet(edges, radius=0.5)` inside a `try/except`)
- Add an assembled vs exploded view toggle for rendering
