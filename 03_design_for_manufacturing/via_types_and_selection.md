# Via Types and Selection

## Overview

A via is a plated hole that creates an electrical connection between copper layers. The choice of via type has significant implications for board layer count, fabrication cost, assembly reliability, and signal integrity. In HDI (High Density Interconnect) designs and fine-pitch BGA fanout, via selection is a primary design driver.

This file covers through-hole vias, blind vias, buried vias, microvias, and the via-in-pad technique, including when each is appropriate and what it costs.

---

## Fundamentals

### Through-Hole Via

A through-hole via (THV) penetrates the entire board stack from top surface to bottom surface. It is the simplest and cheapest via type to manufacture because it requires only a single drilling and plating operation.

```
Top layer  ----[  ]----
Layer 2    ----[  ]----   } Via barrel connects all layers
...              |
Bottom     ----[  ]----
```

**Characteristics:**
- Drilled after lamination using a mechanical drill
- Plated with copper during the main electroplating operation
- No additional process steps beyond standard fabrication
- Connects all layers simultaneously — any unused internal layers must be isolated from the via using anti-pads

**Limitations:**
- Consumes routing channels on every layer, even layers where the connection is not needed
- Stubs on unused layers can cause signal integrity issues at high frequencies (see Advanced section)
- Minimum drill diameter limited by aspect ratio (board thickness / drill diameter ≤ 10:1 for standard process)
- Cannot be used under a component — the hole prevents soldering on the opposite side unless filled and capped

**Typical specifications:**
- Drill diameter: 0.3 mm (standard) to 0.2 mm (fine)
- Pad diameter: drill + 0.25 mm (giving 0.125 mm annular ring)
- Anti-pad diameter: pad + 0.1 mm minimum

### Blind Via

A blind via connects an outer layer to one or more inner layers, but does not penetrate to the opposite outer layer. It is "blind" because one end is not visible from the other side of the board.

```
Top layer  ----[  ]----
Layer 2    ----[  ]----   } Blind via (top to layer 2)
Layer 3    ----   ----   (no via connection)
Bottom     ----   ----   (no via connection)
```

**How they are fabricated:**

Blind vias are drilled into a partially-assembled stackup (individual copper-clad laminate sub-layers) before the full board is laminated together. This requires additional lamination cycles and is therefore more expensive — typically 30–50% cost adder over a standard through-hole-only board.

**When to use blind vias:**
- Dense BGA fanout where through-vias consume routing channels on inner layers
- When the blind via connection is between adjacent layers (most common case)
- HDI boards where component density precludes through-hole via escape routing

### Buried Via

A buried via connects two or more inner layers without reaching either outer surface. It is fabricated by drilling and plating a sub-stack before outer layers are added.

```
Top layer  ----   ----
Layer 2    ----[  ]----   } Buried via (layer 2 to layer 3)
Layer 3    ----[  ]----
Bottom     ----   ----
```

**Cost impact:**

Buried vias require the most complex fabrication because they necessitate drilling and plating inner layer sub-stacks, then laminating additional layers over them. Expect a 50–80% cost adder over a through-hole-only board. Buried vias are mainly used in the most density-constrained designs: server processors, FPGA boards with large BGAs, and high-layer-count backplanes.

---

## Intermediate

### Microvia (HDI Via)

A microvia is a small-diameter via formed by laser drilling rather than mechanical drilling, connecting adjacent layers only. The laser (typically a CO2 laser for glass/resin or UV-YAG for copper direct drilling) ablates material without the mechanical stress of a drill bit.

**Key characteristics:**
- Diameter: 0.05–0.15 mm (50–150 µm)
- Connects only two adjacent layers (one layer pair per microvia)
- Formed in the outermost laminate before or after lamination, depending on build-up type
- Can be stacked (microvia on microvia) or staggered across multiple lamination cycles

**HDI build-up layers:**

The IPC-2226 HDI design standard defines structures by the number of build-up layers:

| Structure | Description |
|---|---|
| 1+N+1 | One build-up layer on each side of a core |
| 2+N+2 | Two build-up layers on each side |
| Any-layer HDI | Microvias on every layer — maximum density |

**Stacked vs staggered microvias:**

Stacked microvias sit directly on top of each other, requiring copper fill at each level to provide a mechanical and electrical connection point. Staggered microvias are offset horizontally at each layer pair — easier to manufacture but consumes more XY board area.

**Interview tip:** Stacked microvias require copper fill (confirmed by X-ray) at each level. An unfilled stacked microvia collapses under solder joint stress. The additional fill step adds cost but is required for reliability in Class 3 designs.

### Via-in-Pad

Via-in-pad (VIP) places the via drill directly under a surface-mount component pad. This is the key technique that enables BGA fanout for fine-pitch packages where there is insufficient space for escape routing with conventional vias between pads.

**Without via-in-pad:**
```
[BGA pad] -- trace -- [via pad]   requires space for trace escape
```

**With via-in-pad:**
```
[BGA pad = via pad]               via is directly under the component
```

**Fabrication requirements for via-in-pad:**

A via-in-pad must be filled and capped to prevent solder wicking into the via barrel during reflow. If the barrel is open, molten solder flows into the hole by capillary action, starving the component joint and potentially creating a solder ball on the far side of the board.

The process is:
1. Drill and plate the via normally
2. Fill the via barrel with epoxy resin (non-conductive fill) or copper (conductive fill)
3. Planarize (sand or etch back) to restore a flat pad surface
4. Plate additional copper over the filled via to seal the surface
5. The pad surface is now flat and solderable

**Conductive vs non-conductive fill:**

| Fill Type | Thermal Conductivity | Cost | Use Case |
|---|---|---|---|
| Epoxy (non-conductive) | ~0.5 W/mK | Lower | Signal BGAs, standard applications |
| Copper (conductive) | ~390 W/mK | Higher | Thermal via-in-pad under power components |

Thermal via-in-pad under a power component (e.g., a QFN with exposed thermal pad) uses copper-filled, copper-capped vias to conduct heat from the component directly into the PCB thermal stack. This is the primary heat extraction mechanism for many modern power ICs and RF amplifiers.

---

## Advanced

### Via Stub Resonance in High-Speed Signals

In through-hole vias, the unused portion of the barrel below the signal layer forms a transmission line stub. This stub presents a capacitive/inductive discontinuity that can cause signal reflections and insertion loss at frequencies where the stub length is a quarter wavelength.

```
Critical frequency for stub resonance:

f = c / (4 * L_stub * sqrt(epsilon_r))

Where:
  c = speed of light (3e8 m/s)
  L_stub = stub length (m)
  epsilon_r = relative permittivity of PCB laminate (typically 3.8–4.5 for FR4)

Example:
  8-layer board, signal on layer 2, board thickness 1.6 mm
  Stub length ≈ 1.6 mm - (2 * layer separation) ≈ ~1.2 mm

  f = (3e8) / (4 * 0.0012 * sqrt(4.1))
    = (3e8) / (0.00974)
    ≈ 30.8 GHz

For PCIe Gen 4 at 16 Gb/s (8 GHz fundamental):
  Stub resonance at 30 GHz is not a concern.

For PCIe Gen 5 at 32 Gb/s (16 GHz fundamental):
  Third harmonic at 48 GHz — stub still acceptable.

But for a thicker backplane (3.2 mm) on layer 2:
  Stub ≈ 2.8 mm
  f ≈ 13.2 GHz — directly impacts Gen 5 signal integrity
```

**Back-drilling (controlled depth drilling):**

Back-drilling removes the via stub by drilling from the opposite side of the board to a controlled depth, leaving only the plated portion of the barrel that carries the signal. This eliminates the stub resonance at the cost of:
- Additional drilling operation (cost adder ~15–25%)
- Minimum stub length limited by mechanical drill tolerance (~0.25 mm residual stub achievable)
- Must be called out explicitly in fab notes with target back-drill depth per net

Back-drilling is standard practice for PCIe Gen 4 and above, 100G Ethernet, and other high-speed serial links routed through thick multilayer backplanes.

### Via Thermal Resistance

Vias in thermal management applications must be analysed for thermal resistance. A single plated through-hole via has a thermal resistance determined by the copper barrel geometry:

```
R_thermal_via = L / (k_cu * A_barrel)

Where:
  L = via length (board thickness, m)
  k_cu = copper thermal conductivity (385 W/mK)
  A_barrel = cross-sectional area of copper barrel (m²)

For a 1.6 mm board, 0.3 mm drill, 25 µm plating:
  Barrel OD = 0.3 mm, barrel ID = 0.3 - 2(0.025) = 0.25 mm
  A = pi/4 * (0.3^2 - 0.25^2) = pi/4 * (0.09 - 0.0625) mm²
    = pi/4 * 0.0275 = 0.0216 mm² = 2.16e-8 m²

  R = 0.0016 / (385 * 2.16e-8)
    = 0.0016 / 8.32e-6
    ≈ 192 °C/W per via
```

This is a very high thermal resistance for a single via. In practice, thermal management requires **via arrays** — often 25–100 vias under a power component — to achieve a useful parallel thermal path. For copper-filled vias the barrel is solid copper, dramatically reducing resistance per via.

### Via Selection Summary

| Via Type | Cost Multiplier | Use Case | Key Constraint |
|---|---|---|---|
| Through-hole | 1× (baseline) | General purpose, low density | Consumes all-layer routing; stub at high speed |
| Blind | 1.3–1.5× | BGA fanout, HDI layer 1 to 2 | Additional lamination cycle |
| Buried | 1.5–1.8× | High-density inner layer routing | Most complex lamination |
| Microvia (HDI) | 1.4–2.0× | Fine-pitch BGA, any-layer HDI | Adjacent layers only; must fill if stacked |
| Via-in-pad | +10–20% over base | Fine-pitch BGA, thermal management | Must be filled and capped |
| Back-drilled | +15–25% over base | High-speed serial >10 Gbps | Residual stub; minimum back-drill depth |

---

## Best Practices

1. Use through-hole vias as the default for all non-critical signals and power routing — they are the lowest cost and most reliable.
2. Reserve blind and buried vias for designs where through-hole via density genuinely blocks routing; confirm with the fab that the build-up structure is within their process capability.
3. Specify via-in-pad only when BGA pitch or thermal requirements demand it — always specify "filled and capped" in the fab notes and verify on X-ray.
4. For boards above 10 Gbps per channel, evaluate via stub length and consider back-drilling or through-hole via minimisation strategies (use blind vias to avoid the stub entirely).
5. When using thermal via arrays, size the array to achieve the target thermal resistance at assembly level, accounting for solder joint voiding that reduces effective fill.
6. Check the fab's microvia design rules — stacked microvias require a specific build-up sequence and copper fill that must be agreed with the fab before layout begins.

---

## Related Topics

- [PCB Fabrication Constraints](pcb_fabrication_constraints.md)
- [Panelisation](panelisation.md)
- [Stackup Design](../02_pcb_layout/stackup_design.md)
- [Problem 01: Via in Pad](worked_problems/problem_01_via_in_pad.md)
- [Problem 02: Impedance Specification](worked_problems/problem_02_impedance_specification.md)
