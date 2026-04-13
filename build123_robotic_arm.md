# Build123d Prompt: 4-Part Articulated Robotic Arm Linkage

Create a **parametric 4-part planar linkage assembly** in build123d that resembles a compact robotic arm mechanism. The assembly consists of an upper link with asymmetric forked ends, a lower rocker link with a pronounced clevis head, a pedestal base bracket, and reusable thumb-screw pivot pins.

---

## Overall Design Intent

The assembly is a **planar articulated mechanism** — all pivot axes are parallel. It should look like a small, robust prototype linkage suitable for 3D printing or mechanical demonstration. The style is clean functional CAD: mostly extruded prismatic solids with rounded ends, coaxial pivot holes, and realistic assembly clearances.

The mechanism chain is:
- **Base bracket** (fixed) → bottom pivot → **Lower rocker** → top pivot → **Upper link** (free end with gripper)
- **Thumb-screw pins** hold each pivot joint together

---

## Global Parameters

Use a parametric approach with these starter values:

| Parameter | Value | Description |
|-----------|-------|-------------|
| `w_upper` | 20 mm | Upper link width (cross-section height) |
| `t_link` | 15 mm | Nominal link thickness (along pivot axis) |
| `w_lower` | 18 mm | Lower rocker shaft width |
| `d_pivot` | 6 mm | Pivot hole diameter |
| `t_base` | 5 mm | Base flange thickness |
| `L_upper` | 130 mm | Upper link length (pivot-to-pivot reference) |
| `L_lower` | 85 mm | Lower rocker shaft length |
| `base_w` | 52 mm | Base flange width |
| `base_d` | 28 mm | Base flange depth |
| `clr` | 0.3 mm | Side clearance between mating fork parts |
| `clr_p` | 0.125 mm | Pin radial clearance |

### Derived clearance values
- Clevis gap = `t_link + 2 * clr`
- Base fork gap = `t_link + 2 * clr`
- Pin shank radius = `(d_pivot - 2 * clr_p) / 2`

---

## Part 1 — Upper Link with Asymmetric Forked Ends

The upper link is a **long flat bar** with **forked (bifurcated) ends** — but the two ends differ in character.

### Central body
- A rectangular bar: width = `w_upper`, thickness = `t_link`
- Long enough to span between the two fork heads

### Front end (gripper fork) — splayed wider
The free end of the arm has two parallel finger prongs that splay **wider than the body thickness**:
- Total fork spread: approximately **130% of `t_link`**
- This gives the front end a gripper-like appearance
- Each finger is a rectangular prong capped with a semicircular rounded tip (cylinder in the XZ plane)
- A coaxial pivot hole passes through both fingers
- A reinforced head block at the root slightly wider than the fork spread

### Back end (pivot fork) — pulled closer together
The end that connects to the lower rocker via a pin has fingers pulled **tighter than the body**:
- Total fork spread: approximately **65% of `t_link`**
- This creates a compact clevis suitable for a screw/pin passing through
- Same construction as front: rectangular prongs with rounded tips and a coaxial pivot hole
- Reinforced head block at the root

### Fork finger dimensions
- Each finger thickness: `max(3.5, t_link * 0.22)` — roughly 3.5 mm
- Finger projection length: ~16 mm beyond the head block
- Head block length: ~7 mm (reinforced root zone)
- Rounded tip: semicircle with radius = `w_upper / 2`

### Pivot hole placement
- Holes centered along the link's longitudinal axis
- Hole center offset from link center: `L_upper / 2 - w_upper * 0.5`

### Detail feature
- A subtle shallow rectangular boss on one broad face near the front fork region
- Small and non-dominant — just enough to break visual symmetry

### Color
- Light yellow: RGB (0.95, 0.90, 0.55)

---

## Part 2 — Lower Rocker Link with Pronounced Clevis

The lower rocker is a **vertical arm** built from bottom to top with four distinct zones.

### 2.1 Lower pivot eye
- A single rounded tab at the bottom
- Circular eye body with radius ~8 mm, capped with a semicircular end
- One pivot hole through the eye
- Fits between the base bracket's fork cheeks

### 2.2 Main shank
- A long straight rectangular bar
- Width = `w_lower`, thickness = `t_link`
- Length = `L_lower` (~85 mm)
- Clean prismatic shape

### 2.3 Transition zone
- Short region between shank top and clevis root
- Widens from shank thickness to clevis width
- **Faceted corner cuts**: rectangular subtractions at each corner create angled/wedge-like transitions
- Gives a machined/prototyped look, not a smooth blend

### 2.4 Top clevis head (pronounced fingers)
- Two parallel **ear prongs** separated by a central slot
- Slot gap sized to capture the upper link's back fork with clearance
- Each ear:
  - Rectangular body, width = `w_lower + 6` (~24 mm)
  - Thickness = ~3.5 mm each
  - Topped with a semicircular rounded cap (cylinder rotated into the XZ plane)
  - Ear round radius = half the clevis width (~12 mm)
- A reinforced solid root block connects both ear bases
- Coaxial pivot hole through both ears
- Hole positioned slightly below the rounded top region

### Color
- Light yellow: RGB (0.95, 0.90, 0.55) — same as upper link

---

## Part 3 — Base Pedestal Bracket

A fixed support bracket with three zones stacked vertically.

### 3.1 Bottom mounting flange
- Flat rectangular plate: `base_w × base_d × t_base`
- Two small vertical mounting holes near each end (much smaller than pivot holes, ~2 mm radius)

### 3.2 Central pedestal
- A rectangular upright block centered on the flange
- Width = `w_lower + 6`, height = ~15 mm
- Raises the fork above the mounting surface

### 3.3 Upper fork cheeks
- Two parallel vertical lugs extending upward from the pedestal
- Gap between cheeks sized to fit the lower rocker's bottom eye with clearance
- Slot cut into the upper region to create the fork gap
- One coaxial pivot hole through both cheeks
- Cheek height = ~22 mm above pedestal

### Color
- Medium grey: RGB (0.55, 0.55, 0.58)

---

## Part 4 — Thumb-Screw Pivot Pin

A reusable hand-tightened pin used at both pivot joints. Two instances in the assembly.

### 4.1 Shank
- Smooth cylinder
- Diameter slightly smaller than pivot hole (accounting for `clr_p`)
- Length sufficient to pass through the full fork + captured part stack, plus ~4 mm clearance

### 4.2 Collar
- Short cylinder with slightly larger diameter than shank (+1.2 mm radius)
- Sits between the knob and the fork face
- Height ~2 mm

### 4.3 Knob head
- Larger cylinder: radius = `d_pivot * 1.3`
- Height ~6 mm
- 7 evenly spaced shallow axial grooves cut around the circumference for finger grip
- Groove radius ~30% of knob radius
- Grooves positioned slightly outside the knob edge to create lobed indentations
- Result is a fluted thumb wheel — hand-friendly, not sharp

### Color
- Medium-light grey: RGB (0.62, 0.62, 0.65)

---

## Assembly

Position the four parts into an articulated pose:

### Base
- Placed at the origin, flange on the XY ground plane

### Lower rocker
- Bottom eye aligned with the base bracket's pivot hole
- Tilted from vertical by a **rocker tilt angle** (~12°)
- Transform: translate to base pivot, rotate by tilt, offset so the eye center lands on the pivot

### Upper link
- Back fork aligned with the lower rocker's top clevis pivot
- Slight droop angle (~-6° from horizontal)
- Transform: translate to top pivot position, rotate by droop, offset so back pivot hole aligns

### Pins
- One pin at each pivot location (base pivot and top pivot)
- Oriented along the Y axis (pivot axis direction)

### Assembly transform pattern
```
Pos(pivot_world) * Rot(0, angle, 0) * Pos(0, 0, -local_pivot_offset) * part
```

---

## Proportion Rules

Maintain these relationships unless explicitly overridden:

- Upper link length: `6.0–7.5 × w_upper`
- Link thickness: body is `t_link`, fork fingers are ~22% of `t_link` each
- Front fork spread: ~130% of `t_link` (gripper feel)
- Back fork spread: ~65% of `t_link` (tight pin clevis)
- Lower rocker shaft length: `4.0–5.5 × w_lower`
- Clevis ear height above root: ~15 mm with 12 mm round caps
- Base flange width: `2.5–3.5 × w_lower`
- Pin knob diameter: `2.6 × d_pivot`
- All pivot holes share the same nominal diameter

---

## Visual and Manufacturing Style

- Mostly extruded prismatic solids — no organic sculpting
- Rounded ends on links and clevis ears (semicircular caps)
- Faceted transition zones (rectangular boolean cuts, not smooth fillets)
- Realistic mechanical clearances at all joints
- All solids watertight and suitable for CAD export
- Light yellow for structural links, grey tones for base and fasteners
- No black or near-black colors anywhere
- No decorative text, embossing, or surface texture

---

## Key Modeling Techniques

- **Boolean algebra mode**: `Box + Cylinder` for unions, `part - Cylinder` for holes
- **Rot(90, 0, 0) * Cylinder(r, h)**: Creates Y-axis aligned cylinders for pivot holes and pins
- **Semicircular caps**: `Rot(90, 0, 0) * Cylinder(radius, thickness)` placed at prong tips
- **Faceted transitions**: Boolean subtraction of corner blocks to create wedge-like chamfers
- **Compound(children=[...])**: Final assembly preserving per-part colors in GLB export
- **Parametric lobing**: Polar array of cylinder subtractions around the knob for grip flutes
