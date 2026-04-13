# Build123d Prompt: Realistic Cavendish Banana

Create a **single unpeeled Cavendish banana** — the common grocery-store variety — as a clean CAD solid in build123d. The banana must look **ordinary, thick, and slightly imperfect**, not stylized, iconic, or sculptural. It should read immediately as a real piece of fruit because of its chunky proportions, shallow curve, broad underside, subtle peel structure, and blunt ends.

---

## Overall design intent

The banana is a **gently curved, thick organic solid** whose body stays full for most of its length. It is not a sleek crescent, not a bent cylinder, and not a decorative fruit icon. It should feel like something you would pick up at a supermarket — dense, slightly asymmetric, and imperfect in a natural way.

---

## 1. Global proportions

Use realistic Cavendish banana dimensions:

| Parameter | Value | Notes |
|-----------|-------|-------|
| Chord length | 170 mm | Straight-line distance from stem to tip |
| Arch rise | 28 mm | Maximum height of curve above chord — shallow |
| Max body width | 40 mm | Side-to-side at thickest point |
| Max body thickness | 33 mm | Belly-to-back at thickest point |
| Stem stub length | ~7 mm | Short and chunky |
| Tip nub length | ~4 mm | Compact and blunt |

### Critical proportion rules
- The body should stay at **95%+ of maximum width** for roughly 40% of the total length (from about t=0.28 to t=0.64)
- Taper at both ends should be **short and blunt**, not gradual and elegant
- The banana should feel **front-heavy** — the stem side is thicker and more anchored than the tip side
- Maximum girth sits at approximately **40% of the length from the stem end**, not dead center

---

## 2. Centerline / spine geometry

The spine is a **shallow, broad, slightly asymmetric planar arch** in the XZ plane.

### Curve formula
Use a modified sine arch with asymmetric weighting:

```
z(t) = RISE × sin(π × t) × (1 + 0.55 × (0.43 − t)) / normalization
```

This produces:
- Peak near **t ≈ 0.43** (biased toward stem side)
- Gentler curvature on the stem half
- Slightly steeper descent toward the tip
- A broad, relaxed middle span — not a tight arc

### What the curve must NOT be
- A perfect circular arc
- A high dramatic crescent
- A boomerang shape
- Symmetric end-to-end

### What it should feel like
A lazy, practical grocery-banana curve — broad in the middle, barely rising, with a long shallow span through the body.

---

## 3. Cross-section geometry

This is the most important aspect. The cross-section must be a **soft rounded triangular-oval with a broader flatter underside**, not a circle and not a strongly lobed shape.

### Section character

At the thickest body stations, the cross-section should have:

- **Domed outer back** (convex side of curve): broad, smooth, gently rounded
- **Two soft side transitions**: full and gradual, not sharply faceted
- **Broad flat underside** (concave side): wider and less deeply curved than the back — the banana could visually "rest" on a table
- **Three subtle longitudinal ridges** where the peel panels meet: present but not prominent
- **Three shallow valleys** between ridges: barely perceptible from mid-distance

### Back vs. underside asymmetry

The underside (inner/concave side) should be **only 82% as deep** as the outer back is tall. This makes the underside broader and flatter while keeping the back properly domed. Achieve this by scaling the negative-Y (belly) portion of each cross-section:

```
if sin(angle) >= 0:
    py = rh × sin(angle) × ridge_mod + back_offset    # normal dome
else:
    py = rh × sin(angle) × 0.82 × ridge_mod + back_offset  # flatter underside
```

### Section offset

Shift each entire cross-section slightly toward the outer/back side by **12% of the local half-height** (scaled per station). This makes the back fuller and the belly tighter in absolute distance from the spine.

### Ridge modulation

Apply a **3-lobe cosine modulation** at 5.5% depth:

```
mod = 1.0 + ridge_depth × cos(3θ − π/2)
```

Phase of −π/2 places ridges at:
- **270°** (inner crease — the most identifiable banana seam)
- **30°** (upper-right side edge)
- **150°** (upper-left side edge)

And valleys at:
- **90°** (outer back center — smooth)
- **210°** and **330°** (lower sides)

This matches real banana peel anatomy: one visible seam on the inner curve, two side seams, and a smooth broad back.

### Left-right asymmetry

Add a **1.8% cosine width modulation** offset by ~0.55 radians, making one side of the banana very slightly fuller than the other.

### Profile resolution

Use **32 points per cross-section** with a periodic B-spline to ensure smooth silhouettes.

---

## 4. Thickness distribution along the length

The banana body must stay **thick for most of its length**, with short blunt tapers at both ends.

### Station-by-station progression

Use 12 cross-section stations with these proportions (fractions of maximum radius):

| Station | t | Width scale | Height scale | Ridge scale | Character |
|---------|---|------------|--------------|-------------|-----------|
| 1 — stem tip | 0.005 | 0.28 | 0.26 | 0.02 | Tiny but wider than a sharp point |
| 2 — stem root | 0.03 | 0.46 | 0.44 | 0.06 | Chunky start |
| 3 — stem shoulder | 0.08 | 0.68 | 0.66 | 0.22 | Rapid expansion into body |
| 4 — early body | 0.16 | 0.88 | 0.86 | 0.55 | Already thick — peel emerging |
| 5 — proximal body | 0.28 | 0.96 | 0.95 | 0.80 | Nearly maximum — full body |
| 6 — max girth | 0.40 | 1.00 | 1.00 | 1.00 | Thickest point |
| 7 — post-peak | 0.52 | 0.99 | 0.98 | 0.98 | Still essentially at max |
| 8 — mid body | 0.64 | 0.96 | 0.94 | 0.92 | Barely tapering |
| 9 — distal body | 0.76 | 0.88 | 0.84 | 0.78 | Gentle taper begins |
| 10 — pre-tip | 0.87 | 0.68 | 0.62 | 0.48 | Still substantial |
| 11 — near tip | 0.95 | 0.38 | 0.34 | 0.18 | Compact |
| 12 — tip nub | 0.99 | 0.16 | 0.15 | 0.05 | Blunt rounded end |

### Key distribution rules
- Stations 5 through 8 (t = 0.28 to 0.64) remain within **4% of maximum** — this is the thick grocery-banana middle
- Taper only becomes obvious after t = 0.76 (last 24% of length)
- Stem end starts at 28% of max (not a tiny point) and fills to 88% by t = 0.16
- Tip end is still 16% of max at the very last station — blunt, not a needle

---

## 5. Stem end geometry

The stem must be **short, thick, and integrated** into a heavy shoulder — not a delicate neck or a clean cylinder.

### Stem stub
- A short tapered cone: ~7 mm long, 6 mm diameter at root tapering to ~4.4 mm
- Slightly twisted (~8° around its axis) for natural irregularity
- Positioned at the stem end of the loft, extending away from the body along the negative-tangent direction
- Overlaps ~1 mm into the body for visual blending

### Stem shoulder
- The body is already at 68% of max width by t = 0.08 — the shoulder fills rapidly
- By t = 0.16, the body is at 88% — the transition from stem to body is short and blunt
- No elegant neck or narrow waist between stem and body

### Color
- Greenish: RGB (0.50, 0.58, 0.22)

### What to avoid
- Long thin stem peg
- Clean cylindrical dowel
- Graceful taper into the stem
- Symmetric stem attachment

---

## 6. Blossom tip geometry

The tip is **compact, blunt, and smaller than the stem end** — not a sharp point.

### Tip nub
- A small tapered cone: ~4 mm long, 3.6 mm diameter at root tapering to ~1.6 mm
- Extends from the last loft section along the tangent direction
- Rounded and dry-looking

### Tip taper
- The body only narrows significantly in the last 13% of length (t = 0.87 to 0.99)
- Even at the very end (t = 0.99), the section is 16% of max — not a vanishing point
- The tip should feel like a blunt continuation of the body, not a drawn-out nose

### Color
- Brownish: RGB (0.44, 0.36, 0.20)

### What to avoid
- Sharp needle point
- Long graceful taper
- Chili-pepper or horn-like end

---

## 7. Surface character and peel structure

### Ridge prominence
- **5.5% of local radius** — visible in end view and under raking light, but the banana appears mostly smooth from a distance
- Ridges should influence highlight flow more than silhouette
- From side view: mostly smooth outline
- From 3/4 view: faint panel transitions
- From end view: clearly non-circular but not aggressively tri-lobed

### Ridge variation along length
- Near stem (ridge_scale 0.02–0.22): barely present, body still filling out
- Mid-body (ridge_scale 0.80–1.00): clearest peel structure
- Near tip (ridge_scale 0.05–0.18): softening, body too narrow for strong ridges

### What to avoid
- Deep peel grooves
- Visible star-shaped or strongly faceted sections
- Over-modeled botanical segmentation

---

## 8. Controlled natural irregularity

The banana must feel natural without being wildly deformed.

### Asymmetries present in the model
- **End-to-end**: stem side heavier and blunter; tip side leaner and more tapered
- **Back vs. belly**: outer back is fuller (domed), inner belly is flatter and broader
- **Left-right**: 1.8% width bias makes one side very slightly fuller
- **Spine peak offset**: maximum arch at t ≈ 0.43, not centered
- **Stem twist**: 8° axial twist on stem stub
- **Ridge phase**: peel seam pattern is offset, not aligned with section axes

### What to avoid
- Perfect mirror symmetry in any axis
- Mathematically identical end tapers
- Perfectly uniform section spacing

---

## 9. Construction strategy

### Recommended build123d workflow

1. **Define spine curve** in the XZ plane as a modified sine arch with asymmetric weighting, normalized so the peak equals RISE.
2. **Compute tangent** at each station via finite differences.
3. **Create section planes** normal to the tangent at each of the 12 stations. Use `Plane(origin=pos, x_dir=Vector(0,1,0), z_dir=tangent)` — the width direction is world Y, and the height direction is perpendicular to the tangent in the XZ plane.
4. **Sketch cross-sections** on each plane using a periodic B-spline through 32 polar points, with:
   - 3-lobe cosine modulation (5.5% depth, phase −π/2)
   - Left-right cosine asymmetry (1.8%)
   - Underside flattening (82% depth factor for sin(θ) < 0)
   - Back-offset shift (12% of local half-height)
5. **Loft** through all 12 sections inside a `BuildPart` context.
6. **Add stem stub** as a separate Cone solid, positioned at the stem end along the negative-tangent, with a slight twist.
7. **Add blossom nub** as a separate smaller Cone at the tip end along the tangent.
8. **Combine** body + stem + nub using `Compound(children=[...])` to preserve per-part colors.

### Important implementation notes
- All `BuildSketch` contexts must be **directly inside** the `BuildPart` block (not in a helper function) — build123d's context stack requires this for `loft()` to find the pending faces.
- Use `Spline(*points, periodic=True)` inside `BuildLine` for smooth closed profiles.
- Call `make_face()` after each `BuildLine` context to convert the wire to a face.
- The loft consumes all pending faces in order — ensure stations are iterated from stem to tip.

---

## 10. Color

| Part | RGB | Description |
|------|-----|-------------|
| Body | (0.95, 0.86, 0.18) | Warm ripe Cavendish yellow |
| Stem | (0.50, 0.58, 0.22) | Greenish stem |
| Tip nub | (0.44, 0.36, 0.20) | Dry brownish blossom end |

Do not use black or near-black anywhere.

---

## 11. Silhouette goals

### Side view
- Broad, shallow, relaxed arch — not a dramatic crescent
- Thick through most of the length
- Fuller on the stem side, slightly leaner toward the tip
- No sudden narrowing until the last ~15%

### End view
- Soft rounded triangular-oval — clearly not circular
- Broad flat underside, domed back
- Subtle ridge hints, not star-shaped

### 3/4 view
- Faint peel panel transitions
- Chunky, grocery-store-real proportions
- Visible stem shoulder and blunt tip

---

## 12. Strong negative constraints

Do **not** generate:
- A sleek crescent or stylized banana icon
- A bent cylinder or bent sausage
- A sweep of a circle along an arc
- A high dramatic arch
- Strongly faceted 3-lobed sections
- Deep peel grooves visible from distance
- A sharp pointed tip or long graceful taper
- A long thin stem neck
- Perfectly symmetric geometry
- A horn, boomerang, chili pepper, or crescent roll shape

Do **generate**:
- A shallow relaxed curve
- A thick dense body that stays full for 60%+ of its length
- A broad flatter underside
- A blunt heavy stem shoulder
- A compact blunt blossom tip
- Subtle peel seam structure
- Ordinary supermarket Cavendish banana proportions

---

## 13. Compact direct prompt

Create a realistic Cavendish banana with a shallow relaxed curve, a thick chunky body that stays full for most of its length, and blunt ends. Use a modified sine spine that peaks at about 43% from the stem with only 28mm of rise over a 170mm chord. Place 12 cross-section stations normal to the spine, each sketched as a soft rounded triangular-oval with subtle 3-lobe peel modulation at 5.5% depth. The outer back should be smoothly domed, the underside should be broader and flatter at 82% of the back depth, and the whole section shifted 12% toward the back. Keep the body at 95%+ of max width from t=0.28 to t=0.64, with rapid fill from a chunky stem shoulder and short blunt taper to a compact tip nub. Add a short tapered greenish stem cone and a small brownish blossom nub as separate solids. Combine all three parts in a Compound with per-part colors. The result should look like an ordinary thick grocery-store banana, not a stylized crescent.

---

## 14. Starter parameters

```python
CHORD    = 170      # mm chord length
RISE     = 28       # mm max arch height
MAX_RW   = 20       # mm max half-width
MAX_RH   = 16.5     # mm max half-height
RIDGE    = 0.055    # peel ridge depth (5.5%)
LOBE_PH  = -π/2    # ridge phase
BACK_OFF = 0.12     # back-belly offset factor
LR_ASYM  = 0.018    # left-right asymmetry
N_PTS    = 32       # profile resolution
```

These are starting values and can be scaled uniformly.
