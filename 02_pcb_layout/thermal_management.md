# Thermal Management

## Prerequisites
- Basic heat transfer: conduction, convection, radiation
- Ohm's law analogy for thermal circuits: power = temperature difference / thermal resistance
- IPC-2152 standard for trace current capacity
- Via thermal resistance (covered in `via_types_and_selection.md`)

---

## Concept Reference

### Thermal Resistance Network

Every thermal design problem reduces to a resistance network. Power flows from junction (die) to ambient through a chain of thermal resistances in series:

```
P_dissipated
     |
  T_junction
     |   R_theta_jc  (junction-to-case, from datasheet)
  T_case
     |   R_theta_cs  (case-to-heatsink, thermal interface material)
  T_heatsink
     |   R_theta_sa  (heatsink-to-ambient)
  T_ambient

T_junction = T_ambient + P * (R_theta_jc + R_theta_cs + R_theta_sa)
```

The goal is always to keep T_junction below the maximum rated value (typically 125°C for silicon ICs, 150°C absolute maximum). A commonly used reliability target is 125°C even when the datasheet permits 150°C.

**Thermal resistance units:** Degrees Celsius per Watt (°C/W). A resistance of 10°C/W with 1 W dissipation produces a 10°C temperature rise.

### Copper as a Heat Spreader

PCB copper acts as a horizontal heat spreader before the heat reaches a heatsink or is convected away from the board surface. A 2 oz copper pour has a sheet thermal resistance:

```
Thermal sheet resistance of copper:
  R_sheet = t_copper / k_copper (per unit area)

  2 oz copper: thickness = 70 µm = 0.07 mm
  k_copper = 385 W/(mK)

  R_sheet = 0.00007 / 385 = 1.82e-7 m²K/W = 0.182 mm²K/W

For a 10 × 10 mm copper pour (100 mm²):
  R_thermal = 0.182 / 100 = 0.00182 K/W ≈ 1.82 mK/W per mm thickness

Spreading resistance is much lower in plane than through the thickness of the board.
```

Copper pours directly beneath and adjacent to hot components dramatically reduce thermal spreading resistance. PCB layout should maximise copper area under power components and route heat to areas of the board that are less thermally loaded.

### IPC-2152 — Trace Current Capacity

IPC-2152 (2009) superseded the older IPC-2221 chart method and provides a more accurate model of trace current capacity as a function of:

- Trace cross-section (width × copper thickness)
- Temperature rise above ambient
- Whether the trace is internal (less cooling) or external
- Board construction (with or without planes nearby)

**Key formula (simplified approximation from IPC-2152 data):**

For an external trace (top or bottom layer) with no nearby planes, at 10°C temperature rise:

```
I_max ≈ 0.048 * ΔT^0.44 * A^0.725

Where:
  ΔT   = allowed temperature rise above ambient (°C)
  A    = trace cross-section area (mils²) = width(mils) × copper_thickness(mils)
  I_max = maximum current (A)

Example: 0.5 mm (19.7 mil) wide trace, 1 oz copper (1.38 mil thick), 20°C rise:
  A = 19.7 × 1.38 = 27.2 mils²
  I_max = 0.048 × 20^0.44 × 27.2^0.725
        = 0.048 × 5.17 × 10.6
        = 2.63 A
```

**IPC-2152 vs IPC-2221 comparison:**

IPC-2221 charts (still widely referenced in older textbooks and tools) are known to be overly conservative — they underestimate current capacity significantly because they were based on 1950s-era measurements. IPC-2152 uses a rigorous empirical model. The difference can be 50% or more for traces on boards with copper planes nearby, which act as heat sinks.

**Critical distinction — internal traces:**

Internal traces (embedded between planes) see much less convective cooling. IPC-2152 derate internal traces by approximately 50% compared to external traces for the same cross-section and temperature rise. Use the internal trace chart or formula when the trace is on an inner layer, even if it is adjacent to a power plane.

```
Rule of thumb (internal traces):
  I_internal ≈ 0.5 × I_external for same ΔT target
```

**Temperature rise budget:**

Always design for the worst-case ambient temperature, not 25°C. For a product operating to 70°C ambient with a 30°C trace temperature rise budget:

```
Max trace temperature = 70 + 30 = 100°C

If the PCB temperature rating is 130°C (FR-4), this provides 30°C margin —
acceptable, but margins should be checked at maximum ambient.
```

### Thermal Via Arrays

Thermal vias conduct heat vertically through the PCB from a hot component pad to a copper pour or heatsink on the opposite side or on an inner plane.

```
Thermal resistance of a single copper-plated via (not filled):

  R_via = L / (k_copper × A_barrel)

  For 1.6 mm board, 0.3 mm drill, 25 µm plating:
    Barrel OD = 0.3 mm, barrel ID = 0.25 mm
    A_barrel = pi/4 × (0.3² - 0.25²) mm² = 0.0216 mm² = 2.16e-8 m²
    R_via = 0.0016 / (385 × 2.16e-8) ≈ 192 °C/W per via

For copper-filled vias (solid copper barrel):
    A_filled = pi/4 × 0.3² mm² = 0.0707 mm² = 7.07e-8 m²
    R_via = 0.0016 / (385 × 7.07e-8) ≈ 59 °C/W per via
```

In practice, a single via contributes little. Via arrays of 9–36 vias under a power component achieve:

```
25 copper-plated vias in parallel: R_array = 192 / 25 ≈ 7.7 °C/W
25 copper-filled vias in parallel: R_array = 59 / 25 ≈ 2.4 °C/W
```

**Via spacing in thermal arrays:**

Place vias on a 1.0–1.2 mm grid within the thermal pad footprint. Vias too close together (< 0.6 mm) are difficult to solder without bridging. Vias too far apart leave hot spots between them.

**Solder filling of open vias:**

If via-in-pad vias under a thermal pad are not filled and capped (see `via_types_and_selection.md`), solder flows into the barrels during reflow. This starves the solder joint, reduces via count effectiveness, and can produce solder balls on the far side. Always specify filled-and-capped for via-in-pad thermal arrays.

### PCB Thermal Interface to Heatsink

When attaching a heatsink to a PCB (for a TO-220 or similar package), the contact interface resistance depends on:

```
R_theta_cs = t_TIM / (k_TIM × A_contact)

Where:
  t_TIM = thickness of thermal interface material (m)
  k_TIM = thermal conductivity of TIM (W/mK)
  A_contact = contact area (m²)

Common TIMs:
  Thermal grease (Shin-Etsu X-23): k = 4-8 W/mK, t = 0.05-0.1 mm
  Phase-change pad:                 k = 3-6 W/mK, t = 0.1-0.2 mm
  Thermal pad (e.g., Bergquist GP3000): k = 3 W/mK, t = 0.25 mm

Example: 20 mm × 20 mm contact area, 0.1 mm grease, k = 6 W/mK:
  A = 4e-4 m²
  R_theta_cs = 0.0001 / (6 × 4e-4) = 0.042 °C/W
```

For board-level applications without a separate heatsink (BGA, QFN components), the PCB copper and vias form the thermal interface. In these cases the relevant parameter is the junction-to-board resistance (R_theta_jb or Psi_jb), which is specified in most IC datasheets and represents heat flow into the PCB rather than into a heatsink.

---

## Tier 1 — Fundamentals

### Question F1
**A component has Tjmax = 125°C, R_theta_jc = 10°C/W, and R_theta_cs = 1.5°C/W (thermal pad + heatsink). The ambient temperature is 50°C and the component dissipates 5 W. What is the maximum acceptable heatsink-to-ambient thermal resistance (R_theta_sa)?**

**Answer:**

The full thermal chain is:

```
T_junction = T_ambient + P × (R_theta_jc + R_theta_cs + R_theta_sa)

Rearranging for R_theta_sa:

R_theta_sa = (T_junction_max - T_ambient) / P - R_theta_jc - R_theta_cs
           = (125 - 50) / 5 - 10 - 1.5
           = 75 / 5 - 11.5
           = 15 - 11.5
           = 3.5 °C/W
```

The heatsink must have a thermal resistance of 3.5°C/W or less.

**Design check:** A small TO-220 heatsink in natural convection typically achieves 10–30°C/W. 3.5°C/W requires a medium-sized heatsink with forced airflow, or a larger natural convection heatsink. The designer would need to consult heatsink vendor data at the given airflow and orientation.

**Common mistake:** Forgetting the thermal interface material (TIM) resistance. Specifying a heatsink at exactly the calculated maximum R_theta_sa with no margin leaves zero headroom for TIM degradation over time, manufacturing variation, or worse-case power dissipation.

---

### Question F2
**Using IPC-2152, estimate the minimum trace width needed to carry 3 A on a 1 oz copper external trace with a 20°C temperature rise budget. The ambient temperature is 40°C.**

**Answer:**

Using the IPC-2152 simplified formula for external traces:

```
I_max = 0.048 × ΔT^0.44 × A^0.725

Rearranging for A:

A = (I / (0.048 × ΔT^0.44))^(1/0.725)
  = (3 / (0.048 × 20^0.44))^(1.379)
  = (3 / (0.048 × 5.17))^(1.379)
  = (3 / 0.248)^(1.379)
  = (12.1)^(1.379)
  = 38.8 mils²
```

1 oz copper = 1.38 mil thick. Required width:

```
W = A / t = 38.8 / 1.38 = 28.1 mils ≈ 0.71 mm
```

**Answer: approximately 0.75 mm (30 mil) wide trace is appropriate, providing a small margin.**

**Verify:** Maximum trace temperature = 40°C ambient + 20°C rise = 60°C — well below FR-4 Tg of ~135°C and within board temperature limits.

---

### Question F3
**What is the difference between R_theta_jc (junction-to-case) and R_theta_ja (junction-to-ambient)? Which should be used when sizing a heatsink, and which is relevant for a BGA soldered directly to a PCB without a heatsink?**

**Answer:**

**R_theta_jc (junction-to-case):**
Thermal resistance from the silicon die (junction) to the package case (the external surface of the component in direct contact with any heatsink). This is a material property of the component itself, independent of the board or heatsink. It is the correct parameter to use when a heatsink is attached directly to the package — because the heatsink attaches at the "case" reference point.

**R_theta_ja (junction-to-ambient):**
Total thermal resistance from die to ambient air, measured on a standardised JEDEC 2S2P test board in still air. It rolls together R_theta_jc, case-to-board, and board-to-ambient into one number. It is a useful first-order estimate for free-air operation without a heatsink, but it is board-dependent and cannot be used to select a heatsink.

**Which to use:**

- Heatsink attached to component: use R_theta_jc. The heatsink starts at the case; calculate R_theta_cs (TIM) + R_theta_sa (heatsink) separately.
- BGA or QFN on PCB without heatsink: use Psi_jb (junction-to-board thermal characterisation parameter) or R_theta_jb, which represents heat flowing into the PCB. This is the primary heat path for most low-to-medium power BGAs, where the bulk of heat leaves through the solder balls and PCB copper rather than through the package lid into air.
- Quick estimate only: R_theta_ja can be used to bound the problem: T_j = T_amb + P × R_theta_ja gives the worst-case junction temperature in still air.

---

## Tier 2 — Intermediate

### Question I1
**A 3.3 V power trace carries 4 A for a 200 mm length on an internal layer (1 oz copper) in a 4-layer stackup. The adjacent ground plane is 0.1 mm away. Calculate: (a) the minimum trace width using IPC-2152 for 20°C temperature rise; (b) the IR voltage drop across the trace; (c) whether the IR drop is acceptable if the load requires 3.3 V ± 3%.**

**Answer:**

**Part (a) — Minimum width, internal trace:**

Internal traces derate by approximately 50% compared to external traces (less convective cooling). The required current capacity of an external trace to support 4 A internal:

Equivalent external I_target = 4 A / 0.5 = 8 A (to find the cross-section, then apply to internal)

More precisely, use the IPC-2152 internal factor directly:

```
A_internal = (I / (0.048 × 0.5 × ΔT^0.44))^(1/0.725)
           = (4 / (0.048 × 0.5 × 20^0.44))^(1.379)
           = (4 / 0.124)^(1.379)
           = 32.3^(1.379)
           = 282 mils²

Width = 282 / 1.38 = 204 mils ≈ 5.2 mm
```

This is a very wide trace — 5.2 mm. This illustrates why high-current internal traces are impractical without copper planes. In practice, a power plane or heavily poured copper (multiple oz) would be used instead of a routed trace for 4 A on an inner layer.

**Alternative: use 2 oz copper:**

```
2 oz = 2.76 mil thick
A = 282 mils² → W = 282 / 2.76 = 102 mils ≈ 2.6 mm (still very wide)
```

For a signal/power hybrid design, 4 A internal traces are managed with 2 oz inner copper pour regions, not single routed traces.

**Part (b) — IR voltage drop:**

Copper resistivity: ρ = 1.72e-8 Ω·m

```
Trace resistance:
  R = ρ × L / A
  A = 5.2 mm × 0.035 mm = 0.182 mm² = 1.82e-7 m²  (at minimum width)
  L = 200 mm = 0.2 m
  R = 1.72e-8 × 0.2 / 1.82e-7 = 0.0189 Ω

  ΔV = I × R = 4 × 0.0189 = 75.6 mV
```

**Part (c) — Is the IR drop acceptable?**

```
3.3 V ± 3% = ±99 mV
75.6 mV drop is within the 99 mV budget — but only just.
```

With the load at the end of the trace, the received voltage is 3.3 - 0.076 = 3.224 V, which is within the ±3% band (3.201 V minimum). However this leaves only 23 mV of margin — not comfortable in practice. A wider trace, shorter route, or receiving-end regulation should be considered.

**Common mistake:** Calculating trace resistance at 25°C only. Copper resistance increases with temperature (TCR ≈ +0.39%/°C). At 45°C actual trace temperature, resistance is roughly 8% higher, making the drop ~82 mV.

---

### Question I2
**A QFN-16 IC dissipates 2 W. It has an exposed thermal pad (4 × 4 mm) on the bottom. The board is a 4-layer stackup, 1.6 mm thick. Design the thermal via array, specify the fill type, and estimate the thermal resistance from thermal pad to the layer-2 ground plane.**

**Answer:**

**Via array design:**

Place vias on a 1.0 mm grid within the 4 × 4 mm thermal pad, with 0.25 mm edge margin:

```
Available area: (4 - 2×0.25) × (4 - 2×0.25) = 3.5 × 3.5 mm

Via positions in X: 0.25, 1.25, 2.25, 3.25, 4.25 → 4 positions within 3.5 mm span
  Actually: floor(3.5/1.0) + 1 = 4 vias per row
Via positions in Y: same — 4 rows
Total: 4 × 4 = 16 vias
```

**Via specification:**
- Drill: 0.3 mm
- Pad: 0.5 mm
- Fill: copper-filled (solid copper) for maximum thermal conductivity
- Cap: copper-capped per IPC-4761 Type VII (required for via-in-pad)
- Grid: 1.0 mm pitch

**Thermal resistance calculation:**

For copper-filled vias (solid copper barrel):

```
A_per_via = π/4 × (0.3e-3)² = 7.07e-8 m²
L = 1.6 mm = 1.6e-3 m (board thickness to layer 2 ≈ 0.1 mm for outer prepreg,
    but using full board thickness for conservative estimate)

Actually: layer 2 is approximately 0.1 mm below top surface.
L_effective = 0.1 mm = 1e-4 m

R_per_via = L / (k × A) = 1e-4 / (385 × 7.07e-8) = 3.67 °C/W

16 vias in parallel:
  R_array = 3.67 / 16 = 0.23 °C/W
```

This is very low — the via array is not the thermal bottleneck at this depth. The bottleneck will be the spreading resistance in the layer-2 copper plane and the convection from the PCB surface.

**Stencil design for thermal pad:**

Use a 4 × 4 grid of apertures (0.75 × 0.75 mm each with 0.25 mm gaps) rather than a single 4 mm aperture. This provides:
- Outgassing channels for flux volatiles (reduces voiding)
- ~86% paste coverage of the pad area
- Consistent paste height across the pad

---

## Tier 3 — Advanced

### Question A1
**A PCB power stage uses a synchronous buck converter with a QFN-exposed-pad power MOSFET dissipating 1.5 W. The PCB is a 4-layer board (1.6 mm thick), ambient temperature is 70°C (inside a sealed enclosure with no forced airflow). The MOSFET's Psi_jb = 15°C/W and Rθjc = 2°C/W. Design a thermal strategy — via array, copper spreading, and supplementary heatsinking — to keep Tj below 125°C. Show all calculations.**

**Answer:**

**Step 1 — Thermal budget:**

```
Available temperature rise: 125 - 70 = 55°C
Power: 1.5 W
Maximum total thermal resistance from junction to ambient: 55 / 1.5 = 36.7°C/W
```

**Step 2 — Heat flow paths:**

For a QFN with exposed pad in a sealed enclosure, the dominant heat path is through the PCB into the enclosure base (conduction-coupled). There is minimal free convection in a sealed box. Assume:

```
R_junction_to_board = Psi_jb = 15°C/W   (per datasheet, JEDEC board)

Board temperature = T_ambient + P × R_board_to_ambient

For a sealed enclosure with good PCB-to-enclosure thermal coupling (standoffs,
thermal pad on bottom of PCB to aluminium enclosure):
  R_board_to_enclosure ≈ 5°C/W (estimated for a 50 cm² board, aluminium standoffs
                                  + thermal pad to 3 mm aluminium base plate)
  R_enclosure_to_ambient ≈ 10°C/W (natural convection, 50 cm² enclosure side area)

Total R_junction_to_ambient = Psi_jb + R_board + R_enc = 15 + 5 + 10 = 30°C/W
T_junction = 70 + 1.5 × 30 = 70 + 45 = 115°C  (5°C below the 120°C reliability limit)
```

This is feasible but marginal. Optimise each stage:

**Step 3 — Via array under QFN:**

Exposed thermal pad: 3 × 3 mm. Using 16 copper-filled vias on 0.8 mm grid:

```
R_via_array = 59 °C/W per filled via (for 1.6 mm board) / 16 vias
            = 3.7 °C/W

This is in series with the solder joint resistance (~1°C/W) and improves
heat spreading from the component pad to the layer-2 plane significantly.
```

**Step 4 — Copper pour spreading:**

Extend the layer-1 ground pour to at least 20 × 20 mm around the component. Also flood layer-2 ground in the same area. This reduces the effective thermal resistance from the via exits to the board edge:

```
Spreading resistance (approximate for a uniform heat source on a square copper
pour of side a = 20 mm, copper thickness 70 µm):

R_spread ≈ 1 / (2 × k_cu × t_cu × a)
         = 1 / (2 × 385 × 0.00007 × 0.02)
         = 1 / 1.078
         ≈ 0.93°C/W (per layer)

Two layers in parallel: 0.47°C/W — negligible. Copper spreading is very effective.
```

**Step 5 — PCB-to-enclosure interface:**

Attach the PCB to an aluminium enclosure base plate using:
- 4× M3 standoffs with spring washers to ensure good mechanical contact
- A 1.5 mm silicone thermal pad (k = 3 W/mK) between PCB bottom and enclosure

```
Thermal pad resistance:
  R_pad = t / (k × A) = 0.0015 / (3 × 0.04) = 0.0125°C/W ≈ negligible

Standoff conduction provides additional path in parallel.
```

**Step 6 — Final thermal budget check:**

```
R_junction_to_board  = 15°C/W (Psi_jb from datasheet)
R_via_array          ≈ 4°C/W  (via array adds to Psi_jb improvement)
R_PCB_spread         ≈ 0.5°C/W
R_PCB_to_enclosure   ≈ 1°C/W
R_enclosure_to_amb   ≈ 8°C/W  (enclosure with large area, fins optional)

Total R = 15 + 4 + 0.5 + 1 + 8 = 28.5°C/W
T_junction = 70 + 1.5 × 28.5 = 70 + 42.8 = 112.8°C  (12°C margin)
```

This is acceptable. If margin were tighter, adding a small aluminium extrusion clip over the QFN (even without active cooling) would reduce enclosure-to-ambient resistance significantly.

**Key lesson:** In a sealed enclosure, thermal design is dominated by the board-to-enclosure and enclosure-to-ambient path. Optimising the via array alone without addressing the enclosure interface gives diminishing returns.

---

### Question A2
**An IPC-2152 analysis shows a 0.4 mm (16 mil) wide 2 oz copper trace on an internal layer can carry a maximum of 2.8 A for a 30°C temperature rise. The trace is 50 mm long, at 3.3 V. Calculate: (a) the trace DC resistance at 85°C trace temperature; (b) the power dissipated in the trace; (c) whether a parallel thermal model incorporating the adjacent GND plane (0.15 mm away, covering the full trace length with 2 oz copper) materially changes the temperature rise. Explain the modelling trade-off.**

**Answer:**

**Part (a) — Trace DC resistance at 85°C:**

```
Copper resistivity at 25°C: ρ_25 = 1.72e-8 Ω·m
Temperature coefficient: α = 0.00393 /°C

ρ_85 = ρ_25 × (1 + α × (85 - 25))
      = 1.72e-8 × (1 + 0.00393 × 60)
      = 1.72e-8 × 1.2358
      = 2.13e-8 Ω·m

Trace dimensions:
  Width = 0.4 mm = 4e-4 m
  Thickness = 2 oz = 70 µm = 7e-5 m
  Length = 50 mm = 0.05 m
  A = 4e-4 × 7e-5 = 2.8e-8 m²

R = ρ × L / A = 2.13e-8 × 0.05 / 2.8e-8 = 0.038 Ω
```

**Part (b) — Power dissipated at 2.8 A:**

```
P = I² × R = 2.8² × 0.038 = 7.84 × 0.038 = 0.298 W ≈ 300 mW
```

This dissipation heats the trace, which in turn slightly increases resistance — iterative calculation would converge within 1-2% for this case.

**Part (c) — Thermal model with adjacent ground plane:**

The adjacent ground plane at 0.15 mm spacing acts as a thermal conductor running in parallel with the FR-4 dielectric. FR-4 has thermal conductivity ~0.3 W/mK; copper is ~385 W/mK.

```
Heat from trace must travel:
  (a) Upward through 0.15 mm FR-4 to the copper plane above
  (b) Laterally through the copper trace itself
  (c) Via the copper plane and then convection from the board surface

Thermal resistance from trace top surface to adjacent plane (per unit area):
  R_fr4 = t / (k × A_per_unit_length)
         = 0.15e-3 / (0.3 × W_trace × L)  [per unit length]

For a 0.4 mm trace, 50 mm length:
  A_contact = 0.4e-3 × 0.05 = 2e-5 m²
  R_fr4 = 0.15e-3 / (0.3 × 2e-5) = 25°C/W

This is in parallel with heat flowing out through the trace ends.
The parallel plane provides a non-negligible thermal path for long traces,
lowering the effective thermal resistance and reducing temperature rise by
approximately 10-20% compared to the IPC-2152 model which does not assume
an adjacent plane.
```

**Modelling trade-off:**

IPC-2152 internal trace data is measured on a test board with a specified construction (2S2P test board, 2-signal, 2-plane). If the designer's board has a ground plane closer than the test board geometry, the actual current capacity will be slightly higher than IPC-2152 predicts — the adjacent plane helps cool the trace. Conversely, if the trace is deeply embedded with planes far away, performance may be worse.

The correct engineering approach is to use IPC-2152 as a conservative baseline, verify with thermal simulation (e.g., IEC 60287 or ANSYS thermal model) for critical power traces, and confirm with first-article thermal measurements. Never use a trace carrying 2.8 A at the IPC-2152 maximum without at least 20% current margin in the design specification.

---

## Best Practices

1. Always model the full thermal chain from junction to ambient — identify the dominant thermal resistance (usually board-to-ambient in sealed enclosures, not junction-to-case) and focus design effort there.
2. Use IPC-2152 for trace current capacity, not IPC-2221. Note whether the trace is internal or external and apply the appropriate derating.
3. For any trace carrying > 1 A, calculate both the current capacity (IPC-2152) and the IR drop. Both constraints must be satisfied simultaneously.
4. Specify via fill type in fabrication notes for thermal via arrays. Non-conductive epoxy fill wastes cross-section for heat conduction — specify copper fill when thermal performance matters.
5. Under QFN and IC thermal pads, use a grid stencil aperture pattern to allow flux outgassing and minimise voiding in the solder joint.
6. Account for copper temperature coefficient when calculating IR drop at operating temperature — use ρ at the worst-case trace temperature, not at 25°C.

---

## Related Topics

- [Via Types and Selection](../03_design_for_manufacturing/via_types_and_selection.md)
- [Stackup Design](stackup_design.md)
- [Component Placement](component_placement.md)
- [Problem 01: BGA Fanout](worked_problems/problem_01_bga_fanout.md)
- [PCB Fabrication Constraints](../03_design_for_manufacturing/pcb_fabrication_constraints.md)
