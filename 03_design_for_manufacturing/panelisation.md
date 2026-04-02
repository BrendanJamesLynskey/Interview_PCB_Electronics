# Panelisation

## Overview

Panelisation is the process of grouping multiple PCB units — either identical copies of the same design, or different designs sharing a production run — onto a single larger panel for fabrication and assembly. The panel is the unit processed by the fab and the assembly line. Individual boards are separated afterwards by routing or scoring.

Getting panelisation right reduces assembly cost, improves yield, and prevents mechanical and thermal damage during depanelisation. Poor panelisation is a common cause of first-article delays and rework.

---

## Fundamentals

### Why Panels Exist

Pick-and-place machines, reflow ovens, and wave-solder machines are optimised for handling rectangular panels of a defined size — typically 250 × 330 mm or 18 × 24 inches depending on the contract manufacturer's equipment. Individual PCBs are often too small to run efficiently through this equipment:

- Very small boards cannot be reliably gripped and transported by the conveyor
- Very small boards have insufficient thermal mass and heat unevenly in reflow
- A single small board wasted as a test coupon represents a high cost-per-board
- Pick-and-place throughput is measured in placements per hour across the whole panel, so filling the panel maximises efficiency

**Types of panelisation:**

| Type | Description | Use Case |
|---|---|---|
| Array (repetitive) | Same PCB design tiled across the panel | Volume production of a single product |
| Multi-image (customer array) | Different designs on the same panel | Prototype batches, shared production runs |
| Rails only | Single large board with frame rails added | Large boards close to panel size |

### Panel Construction

A panel consists of:

1. **Individual PCB units** — the actual board designs
2. **Breakaway rails** — the border frame that holds units together during assembly and provides machine edges for conveyor handling
3. **Fiducial marks** — copper targets used by the pick-and-place machine for optical alignment
4. **Tooling holes** — mechanical registration holes used by stencil printers and fixture systems
5. **Test coupons** — controlled impedance strips, cross-section coupons, or thermal cycling samples included in the rail area for process verification

### Minimum Panel Frame Dimensions

The rails must be wide enough to:
- Clear the conveyor clamps (typically 5 mm minimum from the PCB edge to the board outer edge, 10 mm preferred)
- Accommodate fiducials and tooling holes without overlap
- Provide sufficient rigidity during assembly

Typical minimum rail widths:
- Lead edge (first into oven): 10–15 mm
- Trail edge: 5–10 mm
- Side rails (if used): 5–10 mm

---

## Intermediate

### Breakaway Tab Design

Breakaway tabs (also called mouse bites or break tabs) connect individual PCB units to each other and to the rails. They must be strong enough to hold the panel rigid during assembly but designed to break cleanly without damaging the board.

**V-score (V-groove) separation:**

A V-shaped groove is scored into both sides of the board to a depth of approximately one-third of the board thickness on each side. The groove defines a straight separation line along which the panel snaps apart after assembly.

```
Cross-section of V-score:
         60°
         /\
Top  ---/  \---
Board     ||    }  ~1/3 thickness each side
Bottom ---\  /---
          \/
```

Constraints:
- V-score produces a straight separation line only — curved board edges require tab routing
- Minimum board-edge-to-copper clearance of 0.3 mm on the score line (copper will be damaged if too close)
- Not suitable for boards with edge connectors or components near the cut line
- Snap force can stress components within 5 mm of the score line — avoid placing capacitors and other fragile components in this zone

**Tab routing (mouse-bite tabs):**

Individual units are separated by routed slots with small un-routed tabs (typically 1 mm wide) holding them in place. The tabs are perforated with a row of small drill holes ("mouse bites") to weaken them for clean breaking.

```
Routed slot:   ===== [tab] =====
Mouse bites:   ===== (ooo) =====
```

Tab design guidelines:
- Tab width: 2–3 mm for standard boards, 1 mm for lightweight boards
- Mouse bite drill diameter: 0.5–0.8 mm
- Number of drill holes in tab: 3–5 typical
- Copper-to-tab-edge clearance: 0.5 mm minimum
- Space mouse bite holes 0.2 mm apart (on-centre 0.7–1.0 mm with 0.5 mm drills)

After depanelisation, mouse-bite tabs leave a rough edge on the board perimeter. If a clean edge is required (e.g., for a precision enclosure fit), the design must account for this residual roughness — typically ±0.2 mm from the ideal board edge.

### Fiducial Marks

Fiducial marks are copper targets on the PCB surface that the pick-and-place machine's vision system uses to calculate the precise board position and orientation before placing components. Without fiducials, the machine relies on mechanical registration alone, which is less accurate.

**Types of fiducials:**

| Type | Location | Purpose |
|---|---|---|
| Global (panel) fiducials | Panel rail | Align the entire panel in the machine coordinate system |
| Local (board) fiducials | Each PCB unit | Compensate for unit-to-unit registration variation within the panel |
| Component fiducials | Adjacent to fine-pitch ICs | Sub-region alignment for QFPs and BGAs with pitch < 0.5 mm |

**Fiducial specifications (IPC-7351):**

- Shape: Solid copper circle
- Diameter: 1.0 mm (nominal), 0.5 mm minimum acceptable
- Solder mask opening: 1 mm larger than copper diameter on each side (3 mm total opening for 1 mm fiducial)
- Copper-free keepout: 3 mm radius around each fiducial — no copper, traces, or text in this zone
- Minimum 3 fiducials per panel (two on one axis, one offset on the other axis) — three points uniquely determine position and rotation
- Fiducials should be as far apart as possible within the panel to maximise angular accuracy

**Common mistake:** Fiducials covered by solder mask or silkscreen text are invisible to the vision system. Verify the solder mask and silkscreen layers have correct clearances around all fiducials.

### Tooling Holes

Tooling holes (also called registration holes) provide mechanical registration for:
- Stencil printing (the stencil sits on locating pins through the tooling holes)
- In-circuit test fixtures
- Wave solder pallets

Typical specifications:
- Diameter: 2.0–3.0 mm, drilled through the board (not plated)
- Location: panel corners, in the rail area, not within the PCB units
- Tolerance: ±0.05 mm on hole position and diameter for stencil printing

Do not use tooling holes as fiducials — they serve different functions. Tooling holes are mechanical references; fiducials are optical references.

---

## Advanced

### Panelisation for Assembly Yield

The arrangement of units within a panel affects assembly yield in non-obvious ways:

**Board orientation and thermal uniformity:**

Reflow ovens have a temperature gradient across the width of the conveyor. Boards closer to the side walls may receive slightly more or less heat than boards in the centre. For temperature-sensitive applications, rotating alternate units 180° (a "flip" array) can average out thermal gradients across the population.

**Component clearance at panel boundaries:**

Components on one unit must not fall within the keepout zone of adjacent units or the rail. A common rule is to maintain 5 mm clearance from any component to the nearest V-score or tab edge. Tall components (connectors, heatsinks) require additional clearance.

**Panelisation of mixed-technology boards:**

If a board has both SMT components on both sides and through-hole components on the bottom, the panel must be compatible with:
1. Top-side SMT reflow (pass 1)
2. Bottom-side SMT reflow (pass 2, upside down)
3. Wave or selective soldering for bottom-side THT (pass 3)

The panel stiffness must be maintained through all three passes. Panels that have been V-scored but not yet snapped can lose rigidity after the first reflow if they are close to the snap threshold.

### Panelisation for Different Board Shapes

Irregular board shapes require careful panelisation to minimise waste panel area and maintain rigidity:

**Nesting irregular boards:**

Non-rectangular boards can sometimes be nested (rotated and interlocked) to improve panel utilisation. However, this complicates the panel Gerber generation and may complicate fiducial placement if units are at different orientations.

**L-shaped and cut-out boards:**

Boards with internal cut-outs or L-shapes are routed within the panel. The internal routing requires internal routing bridges or sacrificial tab material to maintain board rigidity until depanelisation.

### CAM Panelisation vs EDA Panelisation

Panelisation can be performed either in the EDA tool before Gerber generation, or by the fabricator's CAM (Computer Aided Manufacturing) software after Gerber receipt.

**EDA panelisation (preferred for assembly):**

The designer panelises the board in the PCB EDA tool, generates a single Gerber set for the complete panel, and the assembly house uses this directly. The designer has full control over unit spacing, tab locations, fiducial positions, and rail content.

**CAM panelisation (fabricator-performed):**

The designer sends individual board Gerbers; the fab creates the panel internally. This is fine for bare-board fabrication but does not account for assembly-specific requirements (fiducial placement, rail content for ICT, stencil step areas). For any board going through SMT assembly, EDA panelisation with assembly-specific content in the rails is strongly preferred.

### Common Panelisation Mistakes

- **No fiducials on the panel or on individual units**: Alignment relies on mechanical registration alone, causing fine-pitch placement errors
- **Copper too close to V-score line**: Depanelisation damages traces or pads
- **No keepout zone near breakaway tabs**: Components crack or lift during manual depanelisation
- **Panel too heavy or too large for the CM's conveyor**: The board sags in reflow, causing warpage and misalignment
- **Tooling holes plated**: Plated holes change diameter with copper deposition; use NPTH (non-plated through-hole) for tooling holes
- **Fiducials covered with silkscreen or solder mask**: Invisible to the pick-and-place vision system

---

## Best Practices

1. Always provide panel drawings to the assembly house for first-article review before committing to tooling.
2. Place global fiducials in diagonally opposite corners of the panel, as far apart as possible.
3. Place local (board-level) fiducials on every unit when the panel contains components with pitch below 0.5 mm.
4. Maintain 5 mm keepout from board edge copper to V-score lines and 5 mm from fragile components to any breakaway edge.
5. Specify tooling holes as non-plated (NPTH) and confirm the CM's tooling pin diameter matches.
6. For prototypes with mixed board shapes, use a multi-image panel to fill the frame and amortise setup costs.
7. Include a controlled-impedance coupon in the panel rail frame on all boards with controlled impedance nets.

---

## Related Topics

- [PCB Fabrication Constraints](pcb_fabrication_constraints.md)
- [Assembly Considerations](assembly_considerations.md)
- [Via Types and Selection](via_types_and_selection.md)
- [Problem 03: BOM Optimisation](worked_problems/problem_03_bom_optimisation.md)
