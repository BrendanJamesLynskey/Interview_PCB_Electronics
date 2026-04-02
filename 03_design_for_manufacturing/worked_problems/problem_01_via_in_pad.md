# Problem 01: Via in Pad

## Problem Statement

You are laying out a PCB that uses a Renesas RZ/G2L applications processor in a 0.5 mm pitch BGA package with 449 balls. The package measures 15 × 15 mm. The board is a 6-layer stackup with a target overall thickness of 1.2 mm.

**Part A:** Explain why standard escape routing (via placed on a trace escaping from the BGA pad) is not feasible for all signals on this package, and why via-in-pad must be considered.

**Part B:** A signal on the BGA has its pad at position D6 (row D, column 6). Describe the via-in-pad process the fab must perform to make this pad solderable. What specification must appear in the PCB fab notes?

**Part C:** The processor has a 3 × 3 mm exposed thermal pad on the bottom of the package that is electrically and thermally connected to the internal ground plane. You need to conduct heat from this pad into a 2 oz copper ground plane on layer 2. Design the via array under this thermal pad. What fill type would you specify and why?

**Part D:** What is the risk if the fab omits the fill-and-cap process on the via-in-pad, and how would you detect this failure at incoming inspection?

---

## Solution Approach

The problem tests four interconnected skills: BGA escape routing constraints, via-in-pad process knowledge, thermal via array design, and defect detection. Work through each part systematically.

### Part A — Why Standard Escape Fails at 0.5 mm Pitch

At 0.5 mm (500 µm) ball pitch, the centre-to-centre distance between adjacent BGA pads is 500 µm. The pad diameter for 0.5 mm pitch is typically 0.3 mm (300 µm) per IPC-7351B, leaving a gap of 200 µm (0.2 mm) between pad edges.

For a standard escape route — trace from pad to via — three objects must fit within that 200 µm gap:

```
Gap available = 200 µm

Requirement to route a trace and via between adjacent pads:

  [via annular ring]  [trace]  [spacing]
  
  Via drill: 0.2 mm → pad: 0.45 mm (annular ring 0.125 mm each side)
  
  Half of via pad:   0.45/2 = 0.225 mm = 225 µm
  
  This is already larger than the 200 µm gap available.
```

Even a minimal via placed adjacent to the pad extends beyond the clearance available between neighbouring pads. This means that in the body of the BGA, there is not enough space to route traces between pads and land a conventional via. Only the outermost one or two rows of balls can use trace-escape to a nearby via.

The inner balls must use via-in-pad so that the via occupies the same XY footprint as the pad itself, requiring no additional lateral space.

### Part B — Via-in-Pad Fabrication Process

The required fabrication sequence for a via-in-pad is:

1. **Standard drilling and plating**: The via hole is drilled (typically 0.2 mm laser or 0.25 mm mechanical for 0.5 mm pitch BGA) and copper-plated to form the barrel connection between layers

2. **Via fill**: The barrel is filled with either:
   - Non-conductive epoxy resin (for signal vias where thermal performance is not critical)
   - Electrically conductive silver epoxy
   - Copper (electroplated solid fill — highest cost, best thermal/electrical performance)

3. **Planarisation**: The overfill is sanded, lapped, or etched back until the via surface is flush with the surrounding copper pad surface. Any recess greater than 50 µm will cause solder paste to sink into the recess rather than sitting on the pad, leading to an unreliable joint.

4. **Cap plating**: A thin copper layer is electroplated over the filled and planarised via surface to seal the fill and create a continuous, homogeneous solderable copper surface.

5. **Standard surface finish**: ENIG, HASL, or other surface finish is applied over the capped pad as normal.

**Required fab note language:**

> "Via-in-pad required on all vias located within BGA land pattern (reference designator U1). Vias shall be filled with non-conductive epoxy, planarised flush, and copper-capped per IPC-4761 Type VII. Surface finish per board specification. Verify fill and cap by cross-section on first article."

**IPC-4761 via fill type reference:**

| Type | Fill | Cap |
|---|---|---|
| I | None | None |
| II | Tented (mask only) | — |
| IV b | Plugged | — |
| VI a/b | Filled | No cap |
| VII | Filled | Copper capped |

Type VII is the only type suitable for via-in-pad under SMT components.

### Part C — Thermal Via Array Design

The 3 × 3 mm exposed thermal pad requires a via array to conduct heat to the layer-2 ground plane.

**Thermal resistance target:**

A typical power dissipation for the RZ/G2L running application workloads is 2–4 W. The junction-to-board thermal resistance target is approximately 5–10°C/W (set by the package datasheet). The via array thermal resistance must be significantly lower than this to not be a bottleneck.

**Via array geometry:**

Standard guidance calls for vias on 1.0–1.2 mm grid under a thermal pad. For a 3 × 3 mm pad:

```
Via grid pitch: 0.8 mm (for 0.2 mm drill, gives reasonable density)

Number of vias in 3 × 3 mm area:
  X direction: floor((3 - 0.5) / 0.8) + 1 = 4 vias
  Y direction: floor((3 - 0.5) / 0.8) + 1 = 4 vias
  Total: 16 vias (4 × 4 array)

(Edge margin of 0.25 mm from pad edge to first via centre)
```

**Fill specification — copper fill:**

For a thermal pad application, specify **copper-filled (solid copper) via-in-pad** for the following reasons:

- Copper thermal conductivity: ~385 W/mK
- Epoxy fill thermal conductivity: ~0.5 W/mK
- Ratio: 770× improvement per via cross-section

With epoxy-filled thermal vias, the barrel copper plating carries heat but the fill material (being thermally insulating) is wasted cross-section. Copper fill ensures the entire via barrel area contributes to heat transfer.

**Thermal resistance of the array:**

```
Per copper-filled via (0.2 mm drill, 1.2 mm board):
  A_copper = pi/4 * (0.2e-3)^2 = 3.14e-8 m²
  R_per_via = L / (k * A) = 0.0012 / (385 * 3.14e-8)
            = 0.0012 / 1.21e-5
            ≈ 99 °C/W

16 vias in parallel:
  R_array = 99 / 16 ≈ 6.2 °C/W
```

This is at the upper end of acceptable. In practice, solder filling the via from the component thermal pad adds solder thermal conductivity (~50 W/mK) in parallel, improving performance. Many designs use 25–36 vias for a 3 × 3 mm thermal pad, which would reduce R_array to 2–4°C/W.

**Stencil aperture for thermal pad:**

Use a grid of 9 apertures (3 × 3) rather than a single aperture covering the full pad. Each aperture is approximately 0.8 × 0.8 mm with 0.2 mm gaps between them. This grid pattern:
- Allows flux outgassing channels to prevent voiding
- Maintains adequate paste volume (80–90% coverage)
- Reduces void area in the solder joint per IPC-7093

### Part D — Risk of Omitting Fill and Cap

**Failure mode:**

If the via barrel is open (unfilled) beneath the BGA thermal pad or signal pad during reflow:

1. Molten solder at 245°C has very low viscosity and high surface energy
2. Capillary action draws solder into the via barrel
3. The BGA solder ball above the via is starved of solder
4. The resulting joint has insufficient solder volume — thin, brittle, or fully open depending on severity
5. On the far side of the board, solder may emerge as a solder ball, which is a short-circuit risk and an IPC-A-610 workmanship violation

**Detection methods:**

- **X-ray inspection**: The definitive test. Cross-sectional X-ray or 2.5D/3D CT X-ray shows solder distribution within the joint and the via barrel. An unfilled via appears as a solder spike or visible hollow cylinder. Perform on first-article boards before approving production.
- **Cross-section metallography**: Destructive test on a sample board — epoxy-pot, cross-section through the via-in-pad, and examine under optical microscope at 100×. Confirms fill continuity, cap thickness, and planarisation quality.
- **AOI (automated optical inspection)**: Cannot see inside joints or vias; AOI alone is insufficient for via-in-pad verification.
- **Continuity testing (ICT or flying probe)**: Will detect an open joint but not a weak or marginal solder joint that will fail under vibration or thermal cycling.

---

## Key Takeaways

- Via-in-pad is necessary at BGA pitches of 0.5 mm and below because there is insufficient gap between pads for conventional trace-escape plus via
- IPC-4761 Type VII (filled and copper-capped) is the only suitable specification for via-in-pad under SMT pads — this must be explicitly stated in fab notes
- Thermal via arrays must use copper fill (not epoxy) to exploit the full cross-section for heat conduction; 16–36 vias per 3 × 3 mm thermal pad is typical
- Thermal pads require a grid stencil aperture to allow outgassing; a single large aperture leads to excessive voiding
- The only reliable detection method for via-in-pad fill quality is X-ray inspection — always perform on first-article builds

---

## Interview Notes

This problem type comes up frequently at companies designing high-density digital hardware (processors, FPGAs, large SoCs). Interviewers want to see that you understand the root cause constraint (geometric — the gap between pads is smaller than a via pad), not just the solution ("use via-in-pad").

Common follow-up questions:
- "What are the cost implications of via-in-pad?" (Answer: adds 10–20% to bare board cost due to fill, planarisation, and cap plating steps)
- "Can you use laser microvias instead of via-in-pad for this BGA?" (Answer: microvias are also placed in-pad, but only span one layer pair; they are the HDI alternative but require a build-up lamination process)
- "How do you handle the solder paste volume over a filled via-in-pad?" (Answer: the cap creates a flat surface; paste volume is determined by stencil aperture size, not affected by the via below)
