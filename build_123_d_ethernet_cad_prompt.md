# Build123d Prompt — Parametric Yellow Ethernet Patch Cable + RJ45 Female Coupler

Create a **fully parametric Build123d script** that models a **short yellow Ethernet patch cable** with transparent RJ45 male connectors on both ends, plus a **dark RJ45 female-to-female inline coupler** with metallic internals. The right plug of the cable should be **fully inserted** into the coupler's left (-X) port.

The model should be **simulation-ready** and **visually realistic**, with **74 separate named parts** suitable for CAD visualization, robotic grasping, and reinforcement-learning insertion tasks.

Do **not** model the environment, table, or any other object — only the cable assembly and coupler.

## General requirements
- Use **millimeters** throughout.
- All critical dimensions must be **named symbolic parameters** at the top of the script.
- Structure the code into **helper functions** that each return parts:
  - `make_boot()` — lofted strain-relief boot with rib grooves
  - `make_plug_body()` — transparent housing with insertion nose chamfers
  - `make_latch()` — mechanically plausible latch with root, spring, snap, catch, and release pad
  - `make_internals()` — contact block, 8 gold pins, 4 wire pairs
  - `make_rear_sleeve()` — yellow transition sleeve
  - `make_guide_rail()` — bottom alignment rail
  - `build_end_assembly(name_prefix)` — assembles all sub-parts for one end
  - `align_to_dir(d)` — rotation helper mapping +X to a direction vector
  - `make_socket_cutout(face_x, direction)` — keystone-profile cutout for coupler port
  - `make_port_metalwork(face_x, direction)` — metal shield frame, floor plate, and contact pins for one coupler port
- Track parts with a `parts[]` list and `part_names[]` list for manifest output.
- Prefer robust primitives and boolean ops over fragile sketch complexity.
- Use `Box`, `Cylinder`, `Circle`, `Spline`, `Polyline`, `RectangleRounded`, `extrude`, `sweep`, `loft`, `chamfer`, and boolean `+` / `-`.
- Wrap edge selection + chamfer/fillet in `try/except` to handle edge cases.
- **Never use black or near-black colors.** Use light/medium tones.

---

## 1. Complete parameter block

```python
# ── Global ──
cable_total_length = 180.0
cable_spline_y_offset = 16.0
cable_spline_z_sag = 2.0

# ── Cable ──
cable_diameter = 5.0

# ── Boot ──
boot_length = 18.0
boot_max_width = 13.2
boot_max_height = 9.2
boot_cable_entry_diameter = 6.2
boot_front_fillet = 1.5
boot_rib_count = 5
boot_rib_pitch = 2.0
boot_rib_depth = 0.5

# ── Male RJ45 Plug ──
plug_length = 22.0
plug_width = 12.0
plug_height = 8.0
plug_nose_chamfer_top = 1.5
plug_nose_chamfer_side = 0.75
plug_guide_rail_width = 2.0
plug_guide_rail_height = 0.4

# ── Latch Tab ──
latch_length = 13.0
latch_root_length = 2.5
latch_root_thickness = 1.0
latch_width = 5.8
latch_peak_rise = 3.2
latch_catch_height = 0.8
latch_catch_length = 1.5
latch_release_pad_length = 3.0
latch_release_pad_height = 0.3

# ── Visual-only Internals ──
wire_bundle_diameter = 0.5
wire_count = 4
contact_pin_count = 8
contact_pin_pitch = 1.02
contact_pin_length = 5.0
contact_pin_width = 0.6
contact_pin_thickness = 0.18
contact_block_length = 8.0
contact_block_width = 9.0
contact_block_height = 2.5

# ── Coupler Block ──
socket_depth = 22.0                  # matches plug_length — full insertion
center_wall = 2.0                    # thin divider between sockets
coup_len = 2 * socket_depth + center_wall  # 46mm total
coup_w = 16.0
coup_h = 16.0
port_main_w = 12.4                   # plug_width (12.0) + 0.4 clearance
port_main_h = 8.4                    # plug_height (8.0) + 0.4 clearance
port_tab_w = 6.2                     # latch_width (5.8) + 0.4 clearance
port_tab_h = 3.5                     # latch_peak_rise (3.2) + 0.3 clearance
sock_depth = 2.5                     # outer chamfered entry bevel
sock_inner_depth = socket_depth - sock_depth  # 19.5mm deep inner bore
shield_th = 0.6                      # metal shield frame wall thickness
shield_depth = 10.0                  # shield frame depth inside socket
```

### Coupler positioning (for plug insertion)

```python
# Right cable endpoint -> plug tip = endpoint + (boot_length + plug_length) along +X
# endpoint = (35, -38, 8), plug tip = 35 + 18 + 22 = 75
# -X face at coup_pos.X - coup_len/2; for full insertion: face at 75 - 22 = 53
# center = 53 + coup_len/2 = 76
coup_pos = Vector(76, -38, 0)
```

---

## 2. Coordinate conventions

- **X** = cable length axis / insertion axis
- **Z** = up
- **Y** = lateral
- Cable midpoint centered near origin, lying near Z = 0
- Plug tips face **outward** along cable tangent at each end
- Coupler sits to the right of the cable with its -X port facing the right plug

---

## 3. Color palette

```python
C_YELLOW        = Color(0.95, 0.82, 0.12)          # cable, boots, latch, rear sleeve
C_CLEAR         = Color(0.80, 0.83, 0.80, 0.55)    # transparent plug housing, guide rail
C_GOLD          = Color(0.83, 0.70, 0.22)          # contact pins (male and female)
C_CONTACT_BLOCK = Color(0.90, 0.88, 0.82, 0.4)    # internal contact block
C_DARK          = Color(0.22, 0.22, 0.24)          # coupler body
C_METAL         = Color(0.72, 0.73, 0.75)          # coupler shield frames, floor plates
WIRE_COLORS     = [
    Color(0.93, 0.55, 0.15),   # orange pair
    Color(0.25, 0.65, 0.30),   # green pair
    Color(0.30, 0.45, 0.85),   # blue pair
    Color(0.60, 0.42, 0.22),   # brown pair
]
```

---

## 4. Cable outer jacket

### Path
Construct a smooth cable centerline using `Spline` through **6 control points**. The left end is free; the right end curves toward the coupler with the last two points sharing the same Y and Z so the outward direction is pure +X (aligned with coupler port):

```python
cable_pts = [
    Vector(-90, 15, 2),                    # left end, free
    Vector(-55, 18, 0.3),                  # gentle curve
    Vector(-15, 5, 0),                     # middle, low
    Vector(5, -20, 4),                     # curving toward coupler
    Vector(18, -38, coup_h / 2),           # approach — aligned Y,Z with endpoint
    Vector(35, -38, coup_h / 2),           # endpoint — plug points +X into coupler
]
```

### Sweep
Create a circular profile on a plane normal to the path start tangent (`cable_path.tangent_at(0)`), then `sweep` along the spline.

### Color
Yellow (`C_YELLOW`).

### Part name
`"cable_jacket"`

---

## 5. Strain-relief boot

### Construction
`loft` with **4 cross-sections** at increasing X positions, all perpendicular to X axis (`x_dir=(0,1,0)`, `z_dir=(1,0,0)`):

| Position | Shape | Size |
|----------|-------|------|
| x = 0 | `Circle` | radius = `boot_cable_entry_diameter / 2` |
| x = `boot_length × 0.35` | `RectangleRounded` | interpolated width/height, radius = `min(w,h) × 0.35` |
| x = `boot_length × 0.70` | `RectangleRounded` | interpolated width/height, radius = 1.8 |
| x = `boot_length` | `RectangleRounded` | `boot_max_width × boot_max_height`, radius = `boot_front_fillet` |

Interpolation formula for each cross-section at fraction `f`:
```python
w = boot_cable_entry_diameter + (boot_max_width - boot_cable_entry_diameter) * f
h = boot_cable_entry_diameter + (boot_max_height - boot_cable_entry_diameter) * f
```

### Rib grooves
On the rear half of the boot, subtract ring-shaped grooves:
- For `i` in `range(boot_rib_count)`, place groove at `x = 2.0 + i * boot_rib_pitch`
- Stop if `rx > boot_length * 0.5`
- Each groove = `Cylinder(lr + 1.5, rib_depth * 0.6)` minus `Cylinder(max(lr - rib_depth, 1.0), rib_depth * 0.7)`, both `rotation=(0, 90, 0)`
- `lr` = interpolated local radius at that X position

### Color
Yellow (`C_YELLOW`).

---

## 6. Male RJ45 plug body

### Housing
`Box(plug_length, plug_width, plug_height)` with `align=(Align.MIN, Align.CENTER, Align.CENTER)`, positioned at `x = boot_length`.

### Front insertion nose chamfers
- Top front Y-parallel edges: `chamfer(length=plug_nose_chamfer_top)` (1.5 mm)
- Front Z-parallel edges: `chamfer(length=plug_nose_chamfer_side)` (0.75 mm)

Select front edges via `.edges().filter_by(Axis.Y).group_by(Axis.X)[-1]` (top only: filter `Z > 0`).

### Color
Clear/transparent (`C_CLEAR`).

### Yellow rear sleeve
A slightly oversized box overlaid on the rear 35% of plug body, **rotated 90° around X** to lie horizontal:
- `Box(plug_length * 0.35, plug_width + 0.2, plug_height + 0.2)` centered at `x = boot_length + plug_length * 0.35 / 2`, then `Rot(90, 0, 0)` applied (swaps Y and Z dimensions)
- Color: `C_YELLOW`
- Creates the yellow-to-clear transition visible on real plugs

---

## 7. Latch tab — mechanically plausible

The latch is the most geometrically complex feature. Build a **side profile in the XZ plane** using `Polyline`, then `extrude` symmetrically by `latch_width / 2` in both Y directions.

### Profile regions

The latch has 5 distinct functional regions along its length:

1. **Root attachment** (x = 0 to `latch_root_length`): Thick base where latch connects to plug top. Thickness = `latch_root_thickness` (1.0 mm).

2. **Flexible spring** (x = root_end to `latch_length × 0.45`): Rises from root to peak height. This region flexes during insertion.

3. **Release plateau** (x = spring_end to `latch_length × 0.62`): Flat top at `latch_peak_rise` (3.2 mm). The surface a finger presses to release the latch.

4. **Inclined snap surface** (x = plateau_end to `latch_length × 0.82`): Angled slope from peak down to `latch_peak_rise * 0.55`. This face slides against the jack during insertion.

5. **Catch feature** (x = snap_end to snap_end + `latch_catch_length`): Vertical drop of `latch_catch_height` (0.8 mm) creating the ledge that locks into the jack. This is the retention feature.

### Profile points

```python
root_end = latch_root_length                    # 2.5
spring_end = latch_length * 0.45                # 5.85
plateau_end = latch_length * 0.62               # 8.06
snap_end = latch_length * 0.82                  # 10.66
catch_end = snap_end + latch_catch_length       # 12.16

Polyline([
    (0, 0),                                                   # base start
    (0, latch_root_thickness),                               # root top
    (root_end, latch_root_thickness + 0.3),                  # root-to-spring transition
    (spring_end, latch_peak_rise),                           # spring peak
    (plateau_end, latch_peak_rise),                          # release plateau
    (snap_end, latch_peak_rise * 0.55),                      # inclined snap surface end
    (snap_end, latch_peak_rise * 0.55 - latch_catch_height), # catch drop (vertical)
    (catch_end, latch_peak_rise * 0.55 - latch_catch_height),# catch ledge bottom
    (catch_end, latch_peak_rise * 0.2),                      # return slope
    (latch_length, latch_peak_rise * 0.15),                  # tip
    (latch_length, 0),                                       # base end
    (0, 0),                                                  # close
])
```

### Release press pad
A small raised box on top of the plateau region:
- `Box(latch_release_pad_length, latch_width - 0.5, latch_release_pad_height)`
- Centered on the plateau at `z = plug_height/2 + latch_peak_rise + pad_height/2`
- Fused to latch body with `+`

### Position
Starts at `x = boot_length + plug_length * 0.08`, at `z = plug_height / 2` (plug top). **Rotated 90° around X** (`Rot(90, 0, 0)`) so the latch lies flat/horizontal on the plug top surface.

### Color
Yellow (`C_YELLOW`).

---

## 8. Guide rail

Bottom alignment rail on plug:
- `Box(plug_length * 0.5, plug_guide_rail_width, plug_guide_rail_height)`
- At `x = boot_length + plug_length * 0.3`, `z = -plug_height/2 - plug_guide_rail_height/2`
- Color: `C_CLEAR`

---

## 9. Simplified internal contact region

### Contact block
Simplified internal structure visible through clear housing:
- `Box(contact_block_length, contact_block_width, contact_block_height)`
- Position: `x = boot_length + plug_length - contact_block_length - 2`
- Color: `C_CONTACT_BLOCK` (semi-transparent warm grey)

### 8 gold contact pin blades
Evenly spaced across plug width at `contact_pin_pitch` (1.02 mm):
- **Horizontal blade**: `Box(contact_pin_length, contact_pin_width, contact_pin_thickness)` near plug ceiling (`z = plug_height/2 - 1.3`)
- **Bent contact tip**: `Box(1.0, contact_pin_width, 0.8)` bending downward at plug front
- Position blades at `x = tip_x - contact_pin_length - 1.5`
- Color: `C_GOLD`

### 4 wire pairs (visual-only)
Colored cylinders visible through transparent housing:
- Diameter: `wire_bundle_diameter` (0.5 mm)
- Length: `plug_length - 4`
- At `z = -plug_height/2 + 2.0`, Y positions: -2.0, -0.7, 0.6, 1.9
- Colors: orange, green, blue, brown from `WIRE_COLORS`

---

## 10. End assembly function

`build_end_assembly(name_prefix)` creates one complete plug end and returns `[(part, name), ...]`:

```python
def build_end_assembly(name_prefix):
    end_parts = []
    end_parts.append((make_boot(), f"{name_prefix}_boot"))
    end_parts.append((make_plug_body(), f"{name_prefix}_plug_body"))
    end_parts.append((make_rear_sleeve(), f"{name_prefix}_rear_sleeve"))
    end_parts.append((make_latch(), f"{name_prefix}_latch_tab"))
    end_parts.append((make_guide_rail(), f"{name_prefix}_guide_rail"))
    for j, internal in enumerate(make_internals()):
        end_parts.append((internal, f"{name_prefix}_internal_{j}"))
    return end_parts
```

All sub-parts are built along +X starting at x=0 (boot rear). The assembly is then placed at the cable endpoint with the correct orientation.

---

## 11. Placing ends at cable endpoints

### Direction helper

```python
def align_to_dir(d):
    """Rotation mapping +X to direction d."""
    d = d.normalized()
    az = degrees(atan2(d.Y, d.X))
    ay = degrees(atan2(-d.Z, sqrt(d.X**2 + d.Y**2)))
    return Rot(0, ay, az)
```

### Left end placement
```python
pt_left = cable_pts[0]
dir_left = (pt_left - cable_pts[1]).normalized()  # outward from cable
rot_left = align_to_dir(dir_left)
for part, name in build_end_assembly("left"):
    placed = Pos(pt_left) * rot_left * part
    add_part(placed, name)
```

### Right end placement
```python
pt_right = cable_pts[-1]
dir_right = (pt_right - cable_pts[-2]).normalized()
rot_right = align_to_dir(dir_right)
for part, name in build_end_assembly("right"):
    placed = Pos(pt_right) * rot_right * part
    add_part(placed, name)
```

---

## 12. RJ45 Female-to-Female Coupler Block

The coupler is a dark rectangular block with two opposing RJ45 female ports (keystone profile). Each socket is deep enough to accept the full male plug length (22mm).

### Body
```python
body = Box(coup_len, coup_w, coup_h, align=(Align.CENTER, Align.CENTER, Align.MIN))
```
- Apply `fillet` to vertical edges (radius 1.0) and top edges (radius 0.6), wrapped in try/except.
- Color: `C_DARK`

### Socket cutouts — `make_socket_cutout(face_x, direction)`

Each port has a **keystone profile** (rectangle + latch tab notch), not a plain rectangle. Two-stage cutout:

1. **Outer chamfered entry** (depth = `sock_depth` = 2.5mm):
   - Rectangle: `(port_main_w + 2.0) × (port_main_h + 1.5)` — slightly oversized for visual bevel
   - Tab notch: `(port_tab_w + 1.0) × (port_tab_h + 0.3)` on top of rectangle
   - Positioned at face + `direction * sock_depth/2`

2. **Inner bore** (depth = `sock_inner_depth` = 19.5mm):
   - Rectangle: `port_main_w × port_main_h` — matches male plug with 0.4mm clearance
   - Tab notch: `port_tab_w × port_tab_h` — matches latch with 0.4mm clearance
   - Positioned deeper: face + `direction * (sock_depth + sock_inner_depth/2)`

Both cutouts are subtracted from the coupler body on each face (-X and +X).

**No tunnel between the two sockets** — each socket is a blind bore separated by a 2mm center wall.

### Port metalwork — `make_port_metalwork(face_x, direction)`

Each port contains 3 types of metal parts, all offset inward by `eps = 0.2mm` to prevent Z-fighting:

1. **Shield frame** (C_METAL):
   - Outer: `Box(shield_depth, mw, mh)` + tab box, where `mw = port_main_w - 0.4`, `mh = port_main_h - 0.4`
   - Inner (subtracted): `Box(shield_depth+1, iw, ih)` + tab, where `iw = mw - 2*shield_th`, `ih = mh - 2*shield_th`
   - Positioned at face + `direction * (shield_depth/2 + eps)`

2. **Floor plate** (C_METAL):
   - `Box(shield_depth + 2, iw - 0.3, shield_th)`
   - Positioned at bottom of inner bore + eps offset from all surfaces

3. **8 gold contact spring pins** (C_GOLD):
   - Horizontal blade: `Box(4.5, 0.5, 0.2)` near ceiling of port (`cz + mh/2 - 1.3`)
   - Bent contact tip: `Box(1.2, 0.5, 1.0)` bending downward
   - Spaced at `contact_pin_pitch` (1.02mm), same as male pins
   - Positioned deeper inside socket: face + `direction * (shield_depth + pin_len/2 + eps)`

### Coupler assembly
```python
# Add metalwork for both ports
for p in make_port_metalwork(-coup_len / 2, 1):
    add_part(Pos(coup_pos) * p, "coupler_metalwork")
for p in make_port_metalwork(coup_len / 2, -1):
    add_part(Pos(coup_pos) * p, "coupler_metalwork")

# Add body last (so metalwork is inside)
add_part(Pos(coup_pos) * body, "coupler_body")
```

---

## 13. Plug insertion geometry

The right cable plug is fully inserted into the coupler's -X port:

- Last two cable spline points share **same Y (-38) and Z (coup_h/2 = 8)** so the outward direction is pure **+X**
- Right cable endpoint at `(35, -38, 8)` → plug tip at `35 + boot_length(18) + plug_length(22) = 75`
- Coupler -X face at `coup_pos.X - coup_len/2 = 76 - 23 = 53`
- Plug tip (75) is `75 - 53 = 22mm` inside the socket = full insertion depth matching `socket_depth`

The cable curves naturally from upper-left, swooping down and right to approach the coupler horizontally.

---

## 14. Male/female dimension matching

| Dimension | Male (plug) | Female (port) | Clearance |
|-----------|-------------|---------------|-----------|
| Width     | 12.0 mm     | 12.4 mm       | 0.4 mm    |
| Height    | 8.0 mm      | 8.4 mm        | 0.4 mm    |
| Latch width | 5.8 mm   | 6.2 mm        | 0.4 mm    |
| Latch rise | 3.2 mm     | 3.5 mm (tab)  | 0.3 mm    |
| Plug length | 22.0 mm   | 22.0 mm (socket depth) | 0.0 mm |

All clearances are 0.3–0.4mm, sufficient for insertion without binding.

---

## 15. Expected part manifest (74 parts)

```
 1. cable_jacket
 2. left_boot
 3. left_plug_body
 4. left_rear_sleeve
 5. left_latch_tab
 6. left_guide_rail
 7. left_internal_0          (contact block)
 8. left_internal_1..8       (8 gold pins)
 9. left_internal_9..12      (4 wire pairs)
20. right_boot
21. right_plug_body
22. right_rear_sleeve
23. right_latch_tab
24. right_guide_rail
25. right_internal_0..12     (same as left)
38-73. coupler_metalwork     (2 ports × 18 metal parts each)
74. coupler_body
```

---

## 16. Assembly hierarchy

```
scene
├── cable_jacket
├── left_end
│   ├── left_boot
│   ├── left_plug_body
│   ├── left_rear_sleeve
│   ├── left_latch_tab
│   ├── left_guide_rail
│   ├── left_internal_0      (contact block)
│   ├── left_internal_1..8   (8 gold pins)
│   └── left_internal_9..12  (4 wire pairs)
├── right_end (inserted into coupler -X port)
│   └── (mirror of left_end)
└── coupler
    ├── coupler_metalwork × 36 (shields, plates, pins for both ports)
    └── coupler_body
```

---

## 17. Build strategy summary

| Component | Technique |
|-----------|-----------|
| Cable body | `sweep` circle along `Spline` (6 control points) |
| Boot | `loft` 4 cross-sections (circle → 3 rounded rects), subtract ring grooves |
| Plug body | `Box` with `chamfer` on front insertion edges |
| Rear sleeve | `Box` overlay, yellow, on rear 35% of plug |
| Latch tab | `Polyline` 12-point side profile → `make_face` → `extrude both=True` + release pad box |
| Guide rail | `Box` on plug bottom |
| Contact block | `Box`, semi-transparent |
| Gold pins (male) | `Box` blade + `Box` bent tip, 8× |
| Wire pairs | `Cylinder` rotation=(0,90,0), 4× colored |
| Coupler body | `Box` with fillets, subtract keystone-profile socket cutouts on both faces |
| Socket cutout | 2-stage: outer bevel `Box+Box` then inner bore `Box+Box` (rectangle + tab) |
| Shield frame | `Box+Box` outer minus `Box+Box` inner (hollow keystone frame) |
| Floor plate | `Box` at socket floor |
| Gold pins (female) | `Box` blade + `Box` bent tip, 8× per port |

---

## 18. Robustness constraints

- Avoid ultra-thin walls below 0.6 mm
- Avoid tiny fillets under 0.2 mm
- Avoid exact tangent/coincident ambiguity in loft sections
- Use simple sketch profiles over overly complex splines
- Wrap edge selection + chamfer/fillet in `try/except`
- Use `align=` parameters on `Box` for predictable positioning
- Use `rotation=(0, 90, 0)` on `Cylinder` to align along X axis
- Offset metalwork by `eps = 0.2mm` from coupler walls to prevent Z-fighting

---

## 19. Expected dimensions

### Cable
- Cable jacket diameter: **5.0 mm**
- Natural drape from upper-left, curving right into coupler

### RJ45 male plug
- 22 mm long body
- 12.0 mm wide, 8.0 mm tall
- Transparent clear housing with yellow rear sleeve
- Front insertion nose with 1.5 mm top chamfer and 0.75 mm side chamfer
- 8 gold contact pins at 1.02 mm pitch
- Guide rail on bottom

### Latch tab
- 13 mm long, rises 3.2 mm above plug top
- 5 functional regions: root, spring, release plateau, inclined snap surface, catch ledge
- Catch feature: 0.8 mm vertical drop for jack retention
- Release press pad on plateau top

### Boot
- 18 mm long
- 4-section loft with 5 rib grooves
- Circle (Ø6.2) to rounded rectangle (13.2 × 9.2) transition

### Coupler
- 46 mm long × 16 mm wide × 16 mm tall
- Two opposing keystone-profile female ports (each 22mm deep)
- 2mm center wall between sockets
- Metal shield frames (0.6mm thick, 10mm deep) inside each port
- 8 gold contact spring pins per port
- Metal floor plates at socket bottom

---

## 20. Render output

```python
scene = Compound(children=parts)
render("model", scene)
```

Print a part manifest at the end:
```python
for i, name in enumerate(part_names):
    print(f"  {i+1:2d}. {name}")
```
