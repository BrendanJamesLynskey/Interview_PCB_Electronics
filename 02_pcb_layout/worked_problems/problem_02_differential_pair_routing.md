# Problem 02: Differential Pair Routing with Constraints

## Problem Statement

You are routing a PCIe Gen 3 x1 link on a 6-layer PCB. The link connects an SoC on the left side of the board to an M.2 connector 80 mm away on the right side. The stackup is:

| Layer | Use | Copper | Dielectric to next layer |
|---|---|---|---|
| 1 (top) | Signal, components | 1 oz | 0.1 mm prepreg (Er = 4.2) to L2 |
| 2 | Ground reference | 1 oz | 0.36 mm core (Er = 4.5) |
| 3 | Signal | 1 oz | 0.36 mm core (Er = 4.5) |
| 4 | Power (3.3 V) | 1 oz | 0.1 mm prepreg (Er = 4.2) to L5 |
| 5 | Signal | 1 oz | 0.1 mm prepreg (Er = 4.2) to L6 |
| 6 (bottom) | Ground + signal | 1 oz | — |

PCIe Gen 3 electrical specifications:
- Differential impedance: 85 Ω ± 15% (per PCIe base spec 3.0, Table 4-11)
- Maximum intra-pair skew: 0.1 UI at 8 GT/s = 12.5 ps
- Maximum inter-pair skew (TX-to-RX): 20 ns
- Maximum insertion loss: 10 dB at 4 GHz (Nyquist for 8 GT/s NRZ)

The SoC TX pair exits at pad coordinates (5, 14) and (5, 15) in mm. The M.2 connector RX pads are at (85, 14) and (85, 15) mm.

**Part A:** Calculate the trace width and gap required for 85 Ω differential impedance on Layer 3 (edge-coupled stripline between L2 ground and L4 power planes).

**Part B:** The route must pass through a region where a DDR4 bus occupies Layer 3. Describe how you handle the layer transition for the differential pair, and what constraints govern the via placement.

**Part C:** After routing, you measure the P trace at 82.4 mm and the N trace at 81.9 mm. Calculate the intra-pair skew and determine whether it meets the PCIe specification. If not, describe exactly how to correct it.

**Part D:** The route passes over a power plane split between the 3.3 V VCCM and 1.8 V VCCIO regions on Layer 4. The split gap is 0.25 mm wide and runs perpendicular to the differential pair route. Assess the signal integrity risk and state what you would do.

---

## Solution Approach

### Part A — 85 Ω Differential Stripline Geometry

Layer 3 is a symmetric stripline between the L2 ground plane and the L4 power plane. For AC signal purposes, the decoupled power plane acts as an AC ground reference.

**Dielectric geometry:**

```
H1 (L3 to L2) = 0.36 mm, Er = 4.5 (core)
H2 (L3 to L4) = 0.36 mm, Er = 4.5 (core)  — symmetric
T = 35 µm = 0.035 mm (1 oz copper)
B = H1 + H2 = 0.72 mm (total separation between reference planes)
```

**IPC-2141A symmetric stripline — iterative solution:**

Starting with W = 0.25 mm, S = 0.12 mm:

```
Z0_se = (60 / sqrt(4.5)) × ln(4 × 0.72 / (pi × (0.8×0.25 + 0.035) × 0.67))
      = 28.29 × ln(2.88 / (pi × 0.235 × 0.67))
      = 28.29 × ln(2.88 / 0.494)
      = 28.29 × ln(5.83)
      = 28.29 × 1.763
      = 49.9 Ω

Coupling correction (S = 0.12 mm, H = 0.36 mm):
  Zdiff = 2 × 49.9 × (1 - 0.347 × exp(-2.9 × 0.12/0.36))
        = 99.8 × (1 - 0.347 × exp(-0.967))
        = 99.8 × (1 - 0.347 × 0.380)
        = 99.8 × (1 - 0.132)
        = 99.8 × 0.868
        = 86.6 Ω  ≈ 85 Ω target (within 2%)
```

**Result: W = 0.25 mm (10 mil), S = 0.12 mm (5 mil) for 85 Ω differential on Layer 3.**

Both dimensions exceed the 4/4 mil standard fab minimum — fabrication is feasible. Confirm with the fab's own calculator and request a TDR coupon.

### Part B — Layer Transition Through DDR4 Bus Region

A layer change on a differential pair introduces three potential problems: via capacitance/inductance discontinuity, reference plane change, and via stub resonance.

**Via placement rules:**

```
1. Matched via pair:
   Place TX_P and TX_N vias side-by-side, separated by the same gap S = 0.12 mm
   between via pad edges. Identical drill diameter and annular ring for both.
   Asymmetric vias cause mode conversion — differential signal becomes
   partially common-mode, generating EMI and reducing noise margin.

2. Adjacent GND return vias:
   Place one GND via within 0.5 mm of each signal via. The return current
   for the differential signal travels on the reference plane; at the layer
   transition, it must jump planes through a via. Without a nearby GND via,
   the return current detours a long path, increasing loop inductance and
   radiating as an antenna.

3. Reference plane verification:
   Layer 3 references L2 (GND) and L4 (PWR/AC-GND).
   Layer 5 references L4 (PWR/AC-GND) and L6 (GND).
   The L4 power plane is shared as an AC reference on both layers — the
   return current path is continuous. The transition is valid as long as
   no plane split exists on L4 under the via pair.

4. Via stub length check:
   Board thickness: 1.6 mm. L3 is at approximately 0.5 mm depth.
   Via transitions from L3 to L5: stub below L5 ≈ 1.6 - 1.3 = 0.3 mm.
   Stub resonance:
     f = c / (4 × L_stub × sqrt(Er))
       = 3e8 / (4 × 0.0003 × sqrt(4.5))
       = 3e8 / 0.00254 = 118 GHz
   Not a concern for PCIe Gen 3 at 4 GHz.
```

**Routing the transition:**

Exit the DDR4 blocking region by transitioning the PCIe pair from L3 to L5 before the DDR4 bus, route the pair on L5 across the blocked region, then transition back to L3 after clearing the obstruction. Keep the pair tightly coupled (constant gap S) through both transitions and maintain the same impedance on L5 as on L3 — verify the L5 asymmetric stripline geometry gives the same 85 Ω.

### Part C — Intra-Pair Skew Calculation and Correction

**Propagation velocity in FR-4 stripline:**

```
v = c / sqrt(Er_eff) = 3e8 / sqrt(4.5) = 1.414e8 m/s

Delay per mm:
  t_d = 1 / (1.414e8 m/s × 1000 mm/m) = 7.07 ps/mm
```

**Intra-pair skew:**

```
ΔL = 82.4 - 81.9 = 0.5 mm (P trace is 0.5 mm longer)
Δt = 0.5 × 7.07 = 3.54 ps
```

**PCIe Gen 3 intra-pair skew limit: 12.5 ps**

3.54 ps < 12.5 ps — **the pair passes the intra-pair skew specification with comfortable margin.**

**If the skew had exceeded 12.5 ps (worked correction example):**

Suppose ΔL = 2.0 mm → Δt = 14.1 ps (fails by 1.6 ps). Correction:

```
Required additional length on N trace:
  ΔL_correction = 2.0 - (12.5 / 7.07) = 2.0 - 1.77 = 0.23 mm

  Add a 0.23 mm serpentine (accordion tune) to the N trace only.

Rules for the serpentine:
  (a) Add the tuning section in the region where P and N are tightly coupled
      (adjacent traces at gap S) — not in a region where they are separated.
      Differential mode coupling must be maintained through the tuning section.
  (b) Meander amplitude: 2-3× trace width (0.5-0.75 mm), gap between meander
      turns: ≥ 3× trace width to avoid self-coupling.
  (c) Do NOT add serpentines near connectors or ICs — the electromagnetic
      coupling of the serpentine to the near field of the component creates
      additional noise. Tune in the middle of a straight routing section.
```

### Part D — Power Plane Split Crossing Assessment

**Physical mechanism of the problem:**

A differential pair's return current travels as an image current on the nearest reference plane, directly under the trace. When the reference plane has a split, the image current cannot cross the gap and must detour around it.

```
Normal operation:
  Differential trace (forward current) + image in plane (return) = small loop, low inductance

At the split:
  Image current detours around gap ends — loop area increases by (gap_length × gap_width)
  Larger loop → more inductance → higher impedance discontinuity → signal reflection
  Larger loop → more radiation → EMI emission
```

**Quantitative estimate:**

```
Gap width: 0.25 mm
Trace speed: 7.07 ps/mm
Electrical length of gap: 0.25 × 7.07 = 1.77 ps

At 4 GHz (PCIe Gen 3 Nyquist): period = 250 ps
Gap as fraction of period: 1.77 / 250 = 0.7%

The discontinuity is electrically very short — the reflection coefficient will
be small (< 2%) in isolation.

PCIe Gen 3 total return loss budget at 4 GHz: > -15 dB (each discontinuity
contributes; a single crossing at -30 dB is within the link budget).
```

**Decision:**

A single perpendicular 0.25 mm split crossing is technically within PCIe Gen 3 tolerance — but it is poor practice and adds risk.

**Recommended mitigations (in priority order):**

1. **Reroute to avoid:** Route the differential pair above or below the plane split entirely. If the route has any routing freedom, this is always the preferred solution. A 5-10 mm detour is negligible in an 80 mm route.

2. **Stitching capacitor:** If rerouting is impractical, place a 100 nF 0402 MLCC bridging the split on L4, with vias connecting to each side of the split, positioned directly under the crossing trace. This provides a low-impedance AC return current path across the gap.

3. **Change routing layer:** If the split exists only on L4 and L2 (GND) is continuous, route the PCIe pair on Layer 1 (microstrip, referenced to L2 only) through the split region. The continuous L2 reference provides an uninterrupted image current path.

**Documentation:** Flag the plane split crossing in the design review package with the analysis above. If a field solver is available (SIwave, HyperLynx), simulate the crossing and confirm insertion loss and return loss meet the PCIe budget.

---

## Key Takeaways

- PCIe Gen 3 differential stripline: W ≈ 0.25 mm, S ≈ 0.12 mm for 85 Ω in symmetric FR-4 core at 0.36 mm dielectric height — inner layer traces are wider than outer layer for the same impedance because the dielectric height is larger
- Via transitions for differential pairs require a matched via pair (same geometry) and adjacent GND return vias — the return current path through the transition is as critical as the signal via itself
- Intra-pair skew at 3.5 ps is well within the 12.5 ps PCIe limit for 0.5 mm length mismatch; serpentine correction is added to the shorter trace within the coupled region
- Power plane splits generate return current detours that increase loop inductance and EMI — avoid by rerouting; mitigate with stitching capacitors when rerouting is impractical
- Propagation delay in FR-4 is approximately 7 ps/mm (stripline) — this number is used constantly in differential pair timing calculations

---

## Interview Notes

Differential pair routing is one of the most frequently tested PCB topics at hardware engineering interviews. The four parts of this problem map to the four most common sub-questions:

1. Impedance calculation — interviewers want to see you work through the formula or approximation, not just say "use the EDA tool"
2. Via transitions — the GND return via is the detail most often missed; mentioning it unprompted signals strong SI awareness
3. Length matching — stating where to add the serpentine (in the coupled region, not near connectors) distinguishes senior from junior answers
4. Plane split risk — a strong answer quantifies the electrical length and frames the decision in terms of the link budget, rather than simply asserting "plane splits are always bad"

For a senior or staff level role, add: "I would validate the impedance geometry and plane split crossing with a field solver and include the simulation results in the design review package before releasing to the fab."
