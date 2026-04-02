# Assembly Considerations

## Overview

PCB assembly transforms a bare board and a bill of materials into a functioning product. The assembly process introduces its own set of failure modes — tombstoning, solder bridging, cold joints, and component misalignment — many of which can be designed out before the boards are built. Design for assembly (DFA) is the discipline of making layout and component choices that result in high first-pass yield.

This file covers the surface-mount assembly process, tombstoning prevention, solder paste stencil design, and reflow profile engineering.

---

## Fundamentals

### The Surface-Mount Assembly Process

Standard SMT assembly follows a defined sequence:

1. **Solder paste printing**: A squeegee pushes paste through a laser-cut stencil aperture onto the bare PCB pads
2. **Component placement**: A pick-and-place machine positions components onto the wet paste
3. **Reflow soldering**: The assembly passes through a reflow oven; the paste flux activates, solvents evaporate, and tin-lead or SAC solder melts and wets to the pad and component terminal
4. **Inspection**: AOI (automated optical inspection) and, for critical boards, X-ray inspection for BGA joints
5. **Through-hole soldering** (if present): Wave soldering or selective soldering for THT components placed on the bottom side after SMT reflow

Each stage has its own yield drivers. Understanding all of them is necessary to design a board that assembles reliably at volume.

### Why First-Pass Yield Matters

In a production environment, rework is expensive — far more expensive than the component cost. A board with a 98% first-pass yield at 500 components has a probability of requiring rework of approximately:

```
P(at least one defect) = 1 - (0.98)^500 ≈ 1 - 0.000045 ≈ 99.995%

This means nearly every board requires rework.

At 99.9% per component:
P(at least one defect) = 1 - (0.999)^500 ≈ 39.5%
```

Good DFA practices are the primary driver of first-pass yield at the layout level.

### Component Orientation Rules

- **Electrolytic capacitors**: Mark polarity clearly on silkscreen with a "+" symbol and a filled/unfilled pad convention. Verify orientation with schematic review.
- **Polarised components (diodes, LEDs, tantalums)**: Use a consistent polarity convention across the entire board. Cathode bar or anode triangle should always face the same direction relative to the reference designator.
- **ICs**: Pin 1 should be identifiable from both the silkscreen and the PCB pad geometry. A chamfered corner on the courtyard outline or a dot on the pad layer is the standard convention.
- **Connectors**: Orient headers so the mating direction is accessible after assembly. A connector pinned in the direction of the board edge is far easier to assemble than one pointing toward the centre.

---

## Intermediate

### Tombstoning: Causes and Prevention

Tombstoning is a defect where a small passive component (typically 0402 or smaller) stands up on one end during reflow, forming a tombstone shape. One end of the component lifts off its pad, leaving an open circuit.

**Root cause:**

Tombstoning is driven by an imbalance in surface tension forces acting on the two solder joints during the moment the solder melts. If one pad wets before the other, the surface tension of the molten solder on the early-wetting pad exerts a rotational torque large enough to pull the component upright.

Causes of unequal wetting timing:
- Unequal pad sizes (asymmetric copper area, one pad heats faster)
- Unequal thermal mass at each end (adjacent ground plane on one side, open air on the other)
- Non-uniform paste volume applied to the two pads
- Unequal spacing from large thermal masses (heatsinks, connectors)

**Prevention strategies:**

1. **Symmetric pad geometry**: Both pads of a two-terminal passive must be identical in area and copper connection. If one pad connects to a large pour and the other to a thin trace, split the thermal connection to the pour using a thermal relief pattern.

2. **Thermal relief on ground pads**: When a passive pad connects directly to a copper pour, use a thermal relief (cross-hatch connection) rather than a flood fill connection. This slows heat transfer to the pad, allowing both pads to reach reflow temperature simultaneously.

3. **Component orientation relative to reflow direction**: Orient passives so their long axis is perpendicular to the reflow oven conveyor direction. This ensures both pads enter the peak temperature zone simultaneously. A common layout guideline is to orient all 0402s and smaller in the same direction on a given board side.

4. **Use IPC-7351 land pattern guidelines**: The IPC land pattern database defines pad dimensions that are validated for low-tombstone risk. Using pad dimensions significantly larger than IPC recommendations increases tombstone risk on small passives.

### Solder Paste Stencil Design

The stencil is a laser-cut stainless steel foil (typically 0.12–0.15 mm thick for standard SMT) through which solder paste is deposited. The aperture defines how much paste goes onto each pad.

**Area ratio — the fundamental stencil rule:**

```
Area ratio = Aperture area / Aperture wall area
           = (Length x Width) / (2 x Stencil thickness x (Length + Width))

The area ratio must be > 0.66 for reliable paste release.

Example — 0402 component (1.0 mm x 0.5 mm pad, 0.12 mm stencil):
  Area ratio = (1.0 x 0.5) / (2 x 0.12 x (1.0 + 0.5))
             = 0.5 / 0.36
             = 1.39   (good — well above 0.66)

Example — 0201 component (0.5 mm x 0.3 mm pad, 0.12 mm stencil):
  Area ratio = (0.5 x 0.3) / (2 x 0.12 x (0.5 + 0.3))
             = 0.15 / 0.192
             = 0.78   (marginal — acceptable but watch for inconsistency)
```

If the area ratio is below 0.66, paste sticks to the aperture wall rather than releasing cleanly onto the pad. This produces inconsistent paste volume, leading to solder balls, bridges, or starved joints.

**Step stencils:**

When a board contains both fine-pitch components (requiring thin paste deposit) and large components (requiring thick paste deposit), a stepped stencil is used. The foil is chemically etched or electroformed to be thinner in the fine-pitch regions (e.g., 0.1 mm) and full thickness elsewhere (e.g., 0.15 mm). Step stencils add cost but are often necessary for mixed-technology boards.

**Aperture reduction for fine-pitch:**

For component pitches below 0.5 mm (e.g., 0.4 mm pitch QFN, 0.4 mm BGA), aperture area is typically reduced to 80–90% of pad area to reduce solder bridging between adjacent pads. The paste volume is intentionally under-printed relative to the pad size.

### Reflow Profile

The reflow profile is the temperature-time curve the assembly follows through the oven. A correctly designed profile is critical for reliable joints and component survival.

**Standard SAC305 (lead-free) reflow profile zones:**

```
Temperature
    |                                      *** Peak (235-245°C)
    |                                    **   **
    |                                  **       **
245 |                                **           **
    |                              **
    |          *****************                    **  (Cooling)
200 |         *                 *
    |        *   (Soak zone)
183 |       * (Liquidus)
    |      *
150 |     *
    |    *
    |   *
    | **  (Preheat ramp)
    |
    +--+---+-------+----------+-----+----> Time
      30s  60s    120s       180s  240s
```

**Zone descriptions:**

| Zone | Typical Range | Purpose |
|---|---|---|
| Preheat ramp | 25°C → 150°C at 1–3°C/s | Evaporate solvents from paste, minimise thermal shock |
| Soak / activation | 150–200°C for 60–120 s | Activate flux, equalise temperature across PCB, remove oxides |
| Reflow | 200°C → peak → 200°C | Melt solder, wet to pads and component terminals |
| Peak temperature | 235–245°C (SAC305) | Above liquidus long enough for complete wetting |
| Time above liquidus (TAL) | 45–75 s above 217°C | Too short: cold joints; too long: component damage, excessive intermetallic growth |
| Cooling | 4–6°C/s | Solidify solder; too slow causes coarse grain structure; too fast can cause warpage |

**Sn63/Pb37 (tin-lead) profile for comparison:**

Peak 205–215°C, liquidus at 183°C, TAL 45–60 s. The lower peak temperature makes tin-lead more forgiving for temperature-sensitive components, which is why lead-free exemptions exist in aerospace and medical sectors under RoHS Article 5.

**Common profile mistakes:**

- **Ramp too fast**: Components crack, paste splatters (solder beading)
- **Soak too short**: Inadequate flux activation, poor wetting, solder balls
- **Peak too high**: Component damage, PCB delamination, excessive intermetallic compound (IMC) growth
- **Peak too low**: Insufficient reflow, cold joints, poor wetting
- **TAL too long**: Excessive IMC growth weakens joint long-term, board discolouration

---

## Advanced

### Voiding in Solder Joints

Voids are gas pockets trapped within the solder joint, visible on X-ray inspection. They form from:
- Flux outgassing that cannot escape before the solder solidifies
- Moisture in the paste or PCB laminate (hygroscopic PCBs must be baked before assembly)
- Via outgassing when via-in-pad is not properly filled and capped

IPC-7093 specifies acceptable void levels for BGA joints: typically no more than 25% void area per joint for Class 2, and 10% for Class 3 thermally critical applications. QFN exposed pads are particularly prone to voiding because the large flat joint traps gas; a stencil design with a grid pattern of apertures (rather than a single large aperture) allows outgassing channels.

### Double-Sided Reflow

When both board sides carry SMT components, the board goes through the oven twice — top side first, then bottom side. Components on the bottom side are remelted during the second reflow pass. Critical constraints:

- **Component weight limit for second-pass bottom side**: Gravity plus inertial forces during conveyor vibration must not exceed the surface tension holding components on the bottom side after solder melts. The rule of thumb is that bottom-side components must have a weight-to-pad-area ratio below 30 mg/mm²; heavy components (large BGAs, connectors, large inductors) cannot be reliably held upside-down during reflow and must be placed on the top side.
- **Mechanical wave soldering for bottom-side THT**: If through-hole components must be on the bottom side, they are typically hand-soldered or selectively soldered after SMT reflow, since wave soldering a board that already has top-side SMT risks disturbing top-side joints.

### Moisture-Sensitive Devices

Many components — particularly BGAs, QFNs, and MLCCs with certain dielectric formulations — absorb atmospheric moisture after opening. If reflowed when wet, internal moisture flashes to steam, causing **popcorning**: micro-cracks in the package body, delamination at the die-attach interface, or bond wire failure.

IPC/JEDEC J-STD-020 defines Moisture Sensitivity Levels (MSL):

| MSL | Floor Life (25°C, 60% RH) |
|---|---|
| 1 | Unlimited |
| 2 | 1 year |
| 3 | 168 hours |
| 4 | 72 hours |
| 5 | 48 hours |
| 5a | 24 hours |
| 6 | Bake before use, use within 6 hours |

Components that have exceeded floor life must be baked (typically 125°C for 24 hours, or 40°C for 96 hours for moisture-sensitive packages) before assembly.

### Common Mistakes to Know for Interviews

- **No thermal reliefs on ground pad connections for passives**: The most common tombstoning cause in student and early-career designs.
- **Identical stencil aperture for all pads regardless of size**: Ignoring area ratio leads to paste release failures on fine-pitch devices.
- **Not specifying reflow profile in assembly drawing**: The CM uses a generic profile, which may not suit the specific component mix.
- **Placing large components between the fine-pitch IC and the board edge**: This creates a thermal shadow — the large component blocks hot air from reaching the fine-pitch pads, causing an uneven reflow profile.
- **Ignoring MSL requirements**: Components left open on the floor for weeks then assembled without baking. Failure is invisible until thermal cycling or environmental stress testing.

---

## Best Practices

1. Orient all passive components 0402 and smaller in the same direction on each board side, aligned perpendicular to the reflow conveyor direction.
2. Apply thermal reliefs to all passive pads connecting to copper pours, even on power nets.
3. Calculate stencil area ratio for every component type below 0.5 mm pitch; flag any value below 0.66 for step-stencil or aperture redesign.
4. Specify the reflow profile in the assembly drawing and review it with the contract manufacturer before the first build.
5. Provide MSL labels and handling instructions for all MSL 3+ devices in the assembly kit.
6. Use a grid-pattern stencil aperture for QFN exposed ground pads to allow flux outgassing channels.
7. Review X-ray images from first-article boards before approving production — BGA and QFN voiding is invisible to AOI.

---

## Related Topics

- [PCB Fabrication Constraints](pcb_fabrication_constraints.md)
- [Via Types and Selection](via_types_and_selection.md)
- [Panelisation](panelisation.md)
- [Problem 01: Via in Pad](worked_problems/problem_01_via_in_pad.md)
- [Environmental, REACH, RoHS](../06_standards_and_compliance/environmental_reach_rohs.md)
