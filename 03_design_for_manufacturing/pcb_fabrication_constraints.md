# PCB Fabrication Constraints

## Overview

Fabrication constraints define the physical limits within which a PCB can be reliably manufactured. Violating them produces boards that cannot be fabricated at all, or that suffer from systematic defects: open circuits, shorts between adjacent features, or structural delamination. Understanding the boundary between what a fabricator achieves at standard cost versus what requires a premium process is a core DFM skill and a common interview topic.

This file covers minimum trace and space rules, drill and annular ring constraints, copper weight considerations, solder mask rules, and the practical difference between IPC Class 2 and Class 3 requirements.

---

## Fundamentals

### What Are Design Rules?

Design rules are the minimum geometric dimensions that a fabrication process can reliably produce. They are derived from the fab house's equipment capability, photolithography resolution, and process controls. Every project should begin with a DRC profile loaded from the target fab's specification sheet — not from an EDA tool's default values, which are often more permissive than a real process.

**Core fabrication parameters — typical values:**

| Parameter | Standard Fab | Advanced / HDI Fab |
|---|---|---|
| Minimum trace width | 0.1 mm (4 mil) | 0.075 mm (3 mil) |
| Minimum trace space | 0.1 mm (4 mil) | 0.075 mm (3 mil) |
| Minimum through-hole drill | 0.3 mm | 0.15 mm |
| Minimum annular ring (external) | 0.125 mm | 0.05 mm |
| Minimum via drill | 0.3 mm | 0.2 mm |
| Board edge copper clearance | 0.3 mm | 0.2 mm |

### Why 4/4 mil Is the Standard Threshold

The trace/space rule arises from photoresist resolution. At 4 mil (0.1 mm), standard subtractive copper processes have sufficient control to consistently resolve features without bridging (short caused by under-resolved space) or excessive undercut (width loss caused by over-etching). Going below this demands tighter photolithography, more expensive chemistry, and increased panel scrap rates — all of which translate into cost and lead-time penalties. Most prototype fabs (JLCPCB, PCBWay, OSH Park) quote 6/6 mil as safe and 4/4 mil as available; below that they escalate to a different process tier.

### Annular Ring

The annular ring is the copper pad material remaining around a drilled hole after drilling. Drills wander due to bit flex and fibre composition variation in the laminate. The annular ring provides the geometric tolerance buffer.

```
Annular ring = (Pad diameter - Finished hole diameter) / 2

Example:
  Copper pad diameter: 0.6 mm
  Finished hole diameter: 0.3 mm
  Annular ring = (0.6 - 0.3) / 2 = 0.15 mm
```

If drill wander exceeds the annular ring, a **breakout** occurs: the drill exits the pad copper, leaving a mechanically weak or completely open connection. IPC standards define minimum acceptable annular ring by class (see Intermediate section).

### Drill Diameter vs Finished Hole Size

The fab drills slightly oversized to account for copper plating deposited on the barrel wall after drilling. A finished hole of 0.3 mm typically requires a drill of approximately 0.35 mm.

**Always specify finished hole size in the fab notes, not drill diameter.** Gerber files specify finished holes; the fab compensates. If the designer specifies drill size instead of finished size, the hole will be undersized after plating.

---

## Intermediate

### Copper Weight and Its Trade-offs

Copper weight is expressed in ounces per square foot (oz/ft²). It directly sets the copper foil thickness before etching.

| Copper Weight | Foil Thickness | Minimum Reliable Trace | Typical Application |
|---|---|---|---|
| 0.5 oz | 17.5 µm | 0.075 mm | Fine-pitch signal layers, HDI |
| 1 oz | 35 µm | 0.1 mm | General purpose signal |
| 2 oz | 70 µm | 0.15 mm | Power planes, high-current traces |
| 3 oz | 105 µm | 0.2 mm | Power conversion, motor drives |

Thicker copper enables narrower traces for a given current rating but complicates fine-pitch routing because undercutting is more severe as etch depth increases. The standard solution is an asymmetric stackup: 0.5 oz or 1 oz on signal layers and 2 oz on dedicated power layers.

### IPC Class 2 vs Class 3

IPC-6011 defines three classes of electronic hardware based on the consequences of failure:

**Class 1 — General Electronic Products**
Consumer electronics, toys, and disposable devices where cosmetic imperfection is acceptable and service interruption is non-critical. Rarely specified in professional engineering.

**Class 2 — Dedicated Service Electronic Products**
The default for industrial, commercial, medical non-life-supporting, and most professional electronics. Continued performance is required but service interruption is tolerable. The vast majority of commercial PCBs are Class 2.

**Class 3 — High Reliability Electronic Products**
Safety-critical, life-support, aerospace, defence, and implantable devices. Equipment downtime is not acceptable. Requires tighter process controls, 100% inspection, and tighter acceptance criteria throughout fabrication and assembly.

**Key differences between Class 2 and Class 3:**

| Criterion | IPC Class 2 | IPC Class 3 |
|---|---|---|
| Min. annular ring (external) | 0.05 mm | 0.075 mm |
| Min. annular ring (internal) | 0.05 mm | 0.075 mm |
| Hole wall plating thickness (avg.) | 20 µm | 25 µm |
| Hole wall plating thickness (min.) | 18 µm | 20 µm |
| Board bow and twist | ≤ 0.75% | ≤ 0.50% |
| Via fill inspection | Not required | ≥ 75% barrel fill |
| Laminate voids | Small voids acceptable | Tighter acceptance |
| Conductor width reduction | ≤ 30% of min. specified | ≤ 20% of min. specified |

**Practical cost impact of Class 3:**

Specifying Class 3 increases fabrication cost by 20–50% due to:
- More frequent microsection coupon testing per panel
- Tighter yield targets (higher scrap rate)
- Extended inspection time per board
- Certified operator requirements at certain process steps
- Full traceability documentation

Do not specify Class 3 unless the application genuinely requires it. Medical implantables, pacemakers, avionics flight computers, and munitions fuzing are legitimate Class 3 candidates. A commercial Wi-Fi router is not.

### Solder Mask Constraints

Solder mask is the polymer coating covering copper not intended for soldering. Key design constraints:

- **Solder mask expansion**: The mask opening is typically 0.05–0.1 mm larger than the copper pad on each side to compensate for registration tolerance. Too small an expansion and mask creeps onto the pad, inhibiting solder wetting. Too large and adjacent pads lose their electrical isolation.

- **Minimum solder mask web**: The sliver of mask between two adjacent pad openings must be at least 0.1 mm wide. Narrower slivers lift or crack during thermal cycling, causing solder bridging in subsequent assemblies.

- **SMD vs NSMD pads**: Solder-mask-defined (SMD) pads use the mask opening to define the solderable area — the copper pad is larger and the mask constrains the joint. Non-solder-mask-defined (NSMD) pads rely solely on the copper shape. For fine-pitch BGAs, NSMD is preferred because it gives more consistent solder joint geometry and is less sensitive to mask registration errors.

---

## Advanced

### Controlled Impedance and Fabrication Tolerance

Controlled impedance traces require the fab to hold trace width and dielectric thickness within tight tolerances. Standard fabrication achieves ±10% on impedance; premium Class 3-aligned processes achieve ±5%.

The fabricator achieves controlled impedance by:
1. Receiving the target impedance in the PCB fab notes — for example: "Layer 2: 50 Ω ±10%, single-ended, 1 oz copper, trace width 0.127 mm, dielectric to GND plane 0.1 mm"
2. Running the stackup through an impedance calculator to back out trace widths
3. Measuring impedance using TDR on a dedicated test coupon included in the panel frame
4. Adjusting etch time or requesting a modified prepreg on subsequent production runs if coupon measurements fall outside tolerance

The critical point: **a fab cannot infer controlled impedance requirements from Gerber files alone**. If the fab notes do not explicitly state layer, reference plane, target value, and tolerance, the fab will not apply any special controls and impedance will be determined solely by nominal stackup variation.

### Aspect Ratio and Via Reliability

The aspect ratio of a via is the ratio of board thickness to drill diameter. High aspect ratios are difficult to plate reliably because electroplating solution cannot adequately penetrate and circulate in a narrow, deep hole.

```
Aspect ratio = Board thickness / Via drill diameter

Example A — standard 1.6 mm board, 0.3 mm drill:
  Aspect ratio = 1.6 / 0.3 = 5.3 : 1   (well within limits)

Example B — 3.2 mm board, 0.2 mm drill:
  Aspect ratio = 3.2 / 0.2 = 16 : 1    (specialised process required)
```

IPC-6012 guidelines:
- Up to 10:1 — standard production process
- Up to 8:1 — recommended for Class 3 high-reliability
- Up to 20:1 — achievable with pulse-reverse plating, significant cost premium

Exceeding these limits produces thin plating at the barrel centre. During thermal cycling, CTE mismatch between the FR4 laminate (expanding along Z-axis ~60 ppm/°C) and the copper barrel (~17 ppm/°C) stresses the thin region. **Barrel fracture** is the primary through-via failure mode in aerospace and automotive hardware exposed to wide temperature cycling.

### Common Design Mistakes to Know for Interviews

- **Designing to EDA defaults**: KiCad and Altium ship with permissive default DRC values. Always replace them with the actual fab's profile before routing.
- **Confusing drill diameter with finished hole size**: Specify finished hole size in Gerber/fab notes; the fab adds the plating allowance.
- **Ignoring copper-to-edge clearance**: Copper closer than 0.3 mm to the board edge is damaged during routing and depanelisation. This is a systematic first-article failure on inexperienced designs.
- **Not accounting for etch compensation**: Fine traces must be drawn slightly wider in Gerber to compensate for undercut during etching. Most EDA tools apply etch compensation automatically when configured correctly — verify this is enabled.
- **Specifying Class 3 by default**: Some designers specify Class 3 on all products "to be safe." This doubles cost and lead time with no engineering benefit for non-critical applications.

---

## Best Practices

1. Download the target fab's DRC file and load it into the EDA tool before routing begins — not after.
2. Specify copper weight, surface finish, IPC class, controlled impedance, and board finish explicitly in the fab notes layer of the Gerber package.
3. Keep all copper at least 0.3 mm from the board edge, increasing to 0.5 mm near tab-route lines.
4. Use the fab's online impedance calculator to pre-check trace widths before laying out high-speed nets.
5. Request a DFM review from the fab on first articles — they will flag marginal annular rings, minimum web violations, and aspect ratio concerns before committing to production quantities.
6. Always cross-check current-carrying capacity using IPC-2152 curves, not rule-of-thumb estimates.

---

## Related Topics

- [Via Types and Selection](via_types_and_selection.md)
- [Assembly Considerations](assembly_considerations.md)
- [Panelisation](panelisation.md)
- [Stackup Design](../02_pcb_layout/stackup_design.md)
- [Problem 02: Impedance Specification](worked_problems/problem_02_impedance_specification.md)
