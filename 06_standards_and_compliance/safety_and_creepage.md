# Safety and Creepage

## Prerequisites
- Understanding of mains voltage levels and hazardous voltage thresholds
- Familiarity with PCB fabrication: laminate materials, solder mask, surface finishes
- Basic knowledge of insulation classes and dielectric breakdown
- Ability to read IPC-2221 and IEC 62368-1 design tables

---

## Concept Reference

### Why Creepage and Clearance Matter

When mains voltage (or any hazardous voltage above 50 Vrms AC or 75 V DC) is present
on a PCB, inadequate spacing between conductors can result in:

- **Dielectric breakdown (clearance failure):** The electric field across an air gap
  exceeds the dielectric strength of air (~3 MV/m at sea level). An arc flashes over
  between conductors, causing damage or fire. This is a clearance issue — it depends on
  the distance through air.
- **Tracking (creepage failure):** Conductive paths form across the surface of the PCB
  laminate due to contamination (dust, moisture, ionic flux residues) or degradation
  of the laminate surface under sustained electric field stress. This is a creepage
  issue — it depends on the distance along the surface.

These two failure modes are physically different and require separate distance
specifications. Both must be satisfied simultaneously.

---

### Definitions

**Clearance:** The shortest distance through air between two conductors.

**Creepage distance:** The shortest distance along the surface of an insulating material
between two conductors.

**Working voltage (V_working):** The highest peak or RMS voltage that can appear across
an insulation system under normal operating conditions (not including transients).

**Pollution degree:** A classification of the likely contamination environment:
```
Pollution Degree 1: No pollution or only dry non-conductive pollution
                    (sealed enclosure, cleanroom). Negligible effect.
Pollution Degree 2: Normally non-conductive pollution; occasionally temporary
                    conductivity due to condensation. Typical for most equipment
                    designed for residential/commercial use.
Pollution Degree 3: Conductive pollution, or dry non-conductive pollution that
                    can become conductive due to condensation. Industrial environments.
Pollution Degree 4: Continuous conductive pollution (outdoor, marine). Not normally
                    addressed by PCB spacing alone.
```

Most consumer electronics are designed for Pollution Degree 2.

**Overvoltage category:** A classification of the transient overvoltage severity
on power distribution systems:
```
Category I:  Equipment protected by a transient limiting device (e.g., SPD)
Category II: Equipment plugged into a fixed installation (most mains-connected
             consumer equipment)
Category III: Fixed installation equipment (distribution boards, wiring)
Category IV: Equipment at the origin of the installation (meters, primary overcurrent
             protection)
```

---

### IPC-2221 Spacing Requirements

IPC-2221 is the generic standard for PCB design including spacing requirements.
Table 6-1 provides the minimum spacing requirements based on conductor type and
environment:

**B1 (External conductors, uncoated, sea level to 3050 m):**
```
Voltage (DC or AC peak)   Minimum spacing
0-15 V                    0.1 mm
16-30 V                   0.1 mm
31-50 V                   0.6 mm
51-100 V                  0.6 mm
101-150 V                 0.6 mm
151-170 V                 1.25 mm
171-250 V                 1.25 mm
251-300 V                 1.25 mm
301-500 V                 2.5 mm
> 500 V                   0.005 mm/V
```

**B2 (External conductors, with conformal coating, sea level to 3050 m):**
```
0-30 V    0.05 mm
31-50 V   0.13 mm
51-150 V  0.4 mm
151-300 V 0.4 mm
301-500 V 0.8 mm
> 500 V   0.003 mm/V
```

**A (Internal conductors):**
```
0-100 V   0.1 mm
101-300 V 0.1 mm
> 300 V   0.1 mm (but limited by fabrication capability and dielectric thickness)
```

Note: IPC-2221 specifies functional isolation spacing. For safety isolation (IEC 62368-1
and IEC 60950-1), the requirements are more stringent and depend on reinforced vs.
basic insulation classification.

---

### IEC 62368-1: The Current Safety Standard

IEC 62368-1 (Audio/Video, Information and Communication Technology Equipment) replaced
IEC 60950-1 (IT equipment) and IEC 60065 (audio/video) in 2020. It uses a hazard-based
safety engineering (HBSE) approach.

**Energy source classes:**
```
ES1 (Energy Source Class 1): Not hazardous under normal and abnormal conditions
                               ≤ 15 Vrms / ≤ 21.2 V peak / ≤ 2 J or ≤ 240 VA
ES2: Hazardous under abnormal conditions only (single fault)
     15-120 Vrms / 21.2-170 V peak
ES3: Hazardous under normal conditions — requires double/reinforced insulation
     > 120 Vrms / > 170 V peak
```

Mains voltage in Europe (230 Vrms, 325 V peak) is an ES3 source. All accessible parts
must be protected from ES3 sources by reinforced insulation or double insulation.

**Insulation classes under IEC 62368-1:**

| Class | Description | Min clearance (250 Vrms, PD2, OV II) |
|-------|------------|---------------------------------------|
| Functional insulation | Needed for correct operation only | 0.2 mm (varies) |
| Basic insulation | Single protection against electric shock | 2.0 mm |
| Supplementary insulation | Independent protection in addition to basic | 2.0 mm |
| Double insulation | Basic + supplementary (sum) | 4.0 mm |
| Reinforced insulation | Single system equivalent to double | 4.0 mm |

For a mains-operated product with accessible user-touchable outputs, the isolation
between mains and output must be reinforced insulation (or double insulation).
A typical 4 mm clearance between mains copper and output copper is the result.

---

### Creepage Distance Calculation

Creepage distance depends on working voltage, pollution degree, and material group.

**PCB material groups:**
```
Group I:   CTI ≥ 600 V    (PTFE, polycarbonate, some glass-filled materials)
Group II:  400 ≤ CTI < 600 V    (FR4 standard — actually Group IIIa!)
Group IIIa: 175 ≤ CTI < 400 V   (Standard FR4 epoxy laminate, CTI ≈ 175-250 V)
Group IIIb: 100 ≤ CTI < 175 V   (Some FR4 grades, phenolic materials)
```

CTI = Comparative Tracking Index: the voltage at which a standard leakage current
path forms after 50 drops of contamination solution. Higher CTI = more track-resistant.

FR4 epoxy laminate typically falls in Group IIIa (CTI 175-249 V). This is important:
FR4 is not as track-resistant as might be assumed.

**Creepage table (IEC 62368-1, Table G.13, Basic insulation, Pollution Degree 2):**
```
Working voltage    Group I       Group II      Group IIIa/IIIb
(Vrms)             (CTI ≥ 600)   (400-599)     (100-399)
50                 1.0 mm        1.0 mm        1.4 mm
100                1.4 mm        1.6 mm        2.0 mm
125                1.6 mm        1.8 mm        2.5 mm
250                2.5 mm        3.2 mm        4.0 mm
300                3.2 mm        4.0 mm        5.0 mm
400                4.0 mm        5.0 mm        6.3 mm
500                5.0 mm        6.3 mm        8.0 mm
```

For **reinforced insulation**, multiply basic insulation values by 2.

**Example calculation for a 230 Vrms mains-operated PCB:**

Assume: Basic insulation, Pollution Degree 2, FR4 laminate (Group IIIa), 250 V working.

```
Required creepage distance = 4.0 mm (from table, 250 V, Group IIIa, basic)

For reinforced insulation (mains to accessible output):
  Required creepage distance = 4.0 × 2 = 8.0 mm minimum
```

This means the copper of the mains live terminal must be at least 8.0 mm (along
any surface path, including through solder mask) from any accessible output copper.

---

### PCB Layout Techniques to Achieve Creepage

**Slots (isolation slots/cuts):**
A physical slot cut through the PCB laminate forces the creepage path to go around
the slot rather than directly across the surface. This is the most effective way to
increase creepage distance without increasing PCB size.

```
Without slot: creepage measured directly across FR4 surface
With slot:    creepage path goes down one side of slot + along the slot bottom
              + up the other side = greatly increased path length

Slot width: minimum 0.5 mm (limited by drill or routing bit minimum size)
Slot depth: through the full board thickness (both layers isolated)
```

**Increased copper clearance:**
Simply increasing the copper-to-copper spacing on the layout is the simplest approach
but consumes PCB area.

**Slotted courtyard in the paste/mask layers:**
Even copper tracks that are far apart may have solder mask bridging them. Solder mask
adds approximately 25 µm of insulation but does not significantly contribute to
creepage because the mask can be damaged or absent. When measuring creepage, trace
the path along the bare FR4 surface — solder mask is not credited.

**No copper pour across the isolation boundary:**
A copper fill/pour that crosses the isolation zone (even if GND under both mains and
secondary copper) creates a path that reduces creepage to zero for that net. The
ground plane under mains components must be separated from the ground plane under
secondary components by at least the required creepage distance.

---

### Creepage vs. Clearance: Which Governs?

In practice, for mains-operated equipment, creepage usually governs because the surface
path is shorter than the air path. For very compact designs, both must be calculated
and the maximum of the two requirements used.

```
Example: 250 Vrms working voltage, reinforced insulation, FR4
  Required clearance (IEC 62368-1, OV II): 3.2 mm
  Required creepage (Group IIIa): 8.0 mm

Creepage governs: the copper spacing must be at least 8.0 mm
```

---

## Creepage Distances

### Practical Design Checklist

For a mains-operated product:

1. Identify all nets at mains potential (L, N, and any net connected without isolation)
2. Identify all nets at secondary (output/signal) potential
3. Determine the insulation class required (typically reinforced for mains-to-accessible)
4. Look up the required creepage and clearance for the working voltage, pollution
   degree, and PCB material group
5. In the PCB layout, measure the creepage distance (surface path) between every
   mains net and every secondary net. Use the EDA tool's creepage measurement function
   if available, or manually trace the path
6. Add isolation slots where the required distance cannot be achieved by spacing alone
7. Verify ground pours do not bridge the isolation barrier

---

## Safety Standards

### IEC 62368-1 vs. IEC 60950-1

IEC 60950-1 (withdrawn in 2020, superseded by IEC 62368-1) used a prescriptive approach:
specific minimum dimensions were tabulated. IEC 62368-1 uses HBSE (Hazard Based Safety
Engineering): identify the energy source class, identify the user's vulnerability, and
ensure adequate protection.

In practice, the creepage and clearance tables in both standards are similar. The main
differences are in the energy source classification framework and in the treatment of
audio power and RF energy sources.

For products sold into the EU, the relevant safety standard under the Low Voltage
Directive (LVD) 2014/35/EU is typically IEC 62368-1 (or IEC 60335-1 for household
appliances, IEC 60601-1 for medical devices).

---

## Best Practices

- Always measure creepage distance with a physical slot in the path — never assume
  solder mask provides creepage credit.
- Specify the PCB material's CTI in the fabrication drawings. Standard FR4 may vary
  from Group IIIa to Group IIIb depending on the grade. Use the more conservative
  value unless the specific CTI is confirmed by the laminate supplier.
- Include a creepage and clearance table in the PCB fabrication notes, identifying
  the minimum spacing required for each isolation boundary.
- Submit the PCB design to a safety review against IEC 62368-1 (or the applicable
  standard) before committing to fabrication. Creepage violations found after
  fabrication require a board spin.
- For medical devices, the applicable standard is IEC 60601-1, which has more stringent
  creepage and clearance requirements (2× the values of IEC 62368-1 for patient-applied
  parts).

---

## Tier 1 — Fundamentals

### Question F1
**Define creepage distance and clearance. Why must both be calculated for a mains-connected PCB?**

**Answer:**

**Clearance** is the shortest straight-line distance through air between two conductors.
It is relevant to breakdown via the air path — if the electric field exceeds the
dielectric strength of air (approximately 3 MV/m at sea level, lower at high altitude
and high humidity), an arc can form through the air.

**Creepage distance** is the shortest path along the surface of an insulating material
between two conductors. It is relevant to tracking — the formation of a conductive
carbon path on the surface of the PCB laminate due to contamination, moisture, and
sustained electric field stress. Surface tracking is a failure mode specific to solid
insulating materials and is not governed by air dielectric strength.

Both must be calculated because each addresses a different physical failure mechanism:
- Clearance guards against instantaneous breakdown (arcing)
- Creepage guards against slow degradation and eventual surface tracking

In practice for 250 Vrms mains-operated equipment on standard FR4, the required
creepage distance (typically 4-8 mm for basic to reinforced insulation) is usually
larger than the required clearance (~1.5-3.2 mm), so creepage governs the design.
However, both must be verified because at higher voltages or under transient conditions,
clearance can become the binding constraint.

---

### Question F2
**Why does FR4 laminate's CTI value affect the required creepage distance, and what is the typical CTI of standard FR4?**

**Answer:**

The Comparative Tracking Index (CTI) measures a material's resistance to surface
tracking under contamination. It is the voltage (in volts) at which the material
develops a conductive track within 50 drip cycles of a standardised electrolytic
solution (IEC 60112 test method).

Materials with higher CTI are more resistant to tracking, so they require less
creepage distance for the same working voltage:
- Materials with CTI ≥ 600 V (Group I): shortest required creepage distances
- Materials with CTI 175-399 V (Group IIIa): longest required creepage distances

Standard FR4 epoxy laminate has a typical CTI of 175-250 V, placing it in Group IIIa.
This is the most conservative material group (excluding Group IIIb) and requires the
largest creepage distances.

Practical implication: at 250 Vrms working voltage, basic insulation requires
4.0 mm creepage on FR4 (Group IIIa) but only 2.5 mm on a Group I material.
Specifying a higher-CTI laminate (e.g., some halogen-free or polyimide materials)
can reduce the required creepage distance and allow a more compact design.

---

## Tier 2 — Intermediate

### Question I1
**A compact mains-powered product uses a transformer-isolated flyback converter. The secondary output is 12 V DC. The PCB must comply with IEC 62368-1 reinforced insulation requirements between mains and the 12 V output. The available PCB width limits the mains-to-secondary copper spacing to 5 mm. Is this compliant? If not, what is the most space-efficient design solution?**

**Answer:**

Check the required clearance and creepage:

**Working voltage:** 250 Vrms (European mains, worst case)
**Insulation class:** Reinforced insulation (mains to accessible 12 V output)
**Pollution degree:** 2 (typical for indoor consumer product)
**PCB material:** Standard FR4, CTI ≈ 175-250 V → Group IIIa

Required clearance (IEC 62368-1, OV II, 250 Vrms, reinforced):
- Basic clearance at 250 Vrms: 1.6 mm (from table)
- Reinforced (×2 is not simply applied for clearance; use Table G.12 directly):
  approximately 3.2 mm. (5 mm available — clearance passes.)

Required creepage (IEC 62368-1, Table G.13, 250 Vrms, Group IIIa, basic = 4.0 mm):
- Reinforced = basic × 2 = 4.0 × 2 = **8.0 mm**

The available 5 mm spacing is insufficient for the required 8.0 mm creepage.

**Most space-efficient solution: Add an isolation slot**

A through-board slot between the mains copper and secondary copper increases the
creepage path without consuming additional lateral PCB space:

```
Without slot:
  Mains copper ─── 5 mm surface path ─── Secondary copper
  Creepage = 5 mm (FAIL)

With slot (1 mm wide, full depth):
  Mains copper ─── 1 mm ─── [slot side 1] ─── slot bottom (FR4 edge) ─── 
  [slot side 2] ─── 3 mm ─── Secondary copper
  Creepage = 1 + board_thickness + 3 mm

For a 1.6 mm thick PCB:
  Creepage = 1 + 1.6 + 3 = 5.6 mm (still FAIL)

Increase to 2.5 mm on each side:
  Creepage = 2.5 + 1.6 + 2.5 = 6.6 mm (still FAIL)

For a wider slot (2 mm wide, inside dimensions):
  Creepage = 2 + 1.6 + 2 = 5.6 mm (still FAIL)

The total creepage from mains copper to secondary copper, routed around the slot,
must total 8.0 mm. With a 1.6 mm board:
  If 3.2 mm on each side of the slot: 3.2 + 1.6 + 3.2 = 8.0 mm ✓
  Total lateral space consumed: 3.2 + 1 (slot) + 3.2 = 7.4 mm ← less than 8 mm
```

A 1 mm wide isolation slot with 3.2 mm copper setback on each side uses 7.4 mm
total lateral space and achieves exactly 8.0 mm creepage on a 1.6 mm board. This
is more compact than a direct 8.0 mm copper gap (no slot) and the slot itself
also breaks any surface moisture bridge.

Alternative: Use a higher-CTI laminate (Group II or Group I) to reduce the required
creepage to 6.4 mm or 5.0 mm respectively, potentially eliminating the need for a slot.

---

## Tier 3 — Advanced

### Question A1
**A medical device PCB must meet IEC 60601-1 third edition for a Type B patient-applied part. The equipment operates at 230 Vrms mains input. What creepage and clearance distances are required, and how do they differ from IEC 62368-1 requirements for the same PCB?**

**Answer:**

IEC 60601-1 is the safety standard for medical electrical equipment and introduces
the concept of **means of protection (MOP)** rather than basic/reinforced insulation.

**Means of Protection Classification:**
- **MOOP (Means of Operator Protection):** protection for the operator/user not in
  patient contact
- **MOPP (Means of Patient Protection):** protection for patient-applied parts

Each means is classified:
- 1 MOPP = basic insulation level for medical (patient)
- 2 MOPP = reinforced insulation level for medical (patient) = double protection

**Type B patient-applied part:** The part is applied to the patient but is not directly
connected to the patient's heart. Type B (Body) applies a moderately stringent
requirement.

**Required isolation:** 2 MOPP (mains to Type B patient-applied part, IEC 60601-1
Table 4 and Table 16).

**Clearance for 2 MOPP (250 Vrms, Overvoltage Category II, Pollution Degree 2):**
```
IEC 60601-1 Table 16, 2 MOPP, 250 V working:
  Required clearance: 4.0 mm
```

**Creepage for 2 MOPP (250 Vrms, Pollution Degree 2, PCB material Group IIIa):**
```
IEC 60601-1 Table 16, 2 MOPP, 250 Vrms, Group IIIa:
  Required creepage: 8.0 mm

(Note: same value as IEC 62368-1 reinforced insulation at 250 V, Group IIIa)
```

**Comparison with IEC 62368-1 (reinforced insulation, 250 Vrms, Group IIIa):**
```
Metric             IEC 62368-1 (reinforced)    IEC 60601-1 (2 MOPP, Type B)
Clearance          3.2 mm                       4.0 mm
Creepage           8.0 mm                       8.0 mm
```

The IEC 60601-1 clearance requirement is more stringent (4.0 mm vs. 3.2 mm).
Creepage requirements are the same in this case.

**Additional IEC 60601-1 requirements not present in IEC 62368-1:**
- Leakage current limits are more stringent: ≤ 0.5 mA earth leakage, ≤ 0.1 mA
  patient leakage current (NC), ≤ 0.05 mA for type CF (patient directly cardiac)
- The 2 MOPP requirement is mandatory for **any** connection between mains and
  patient-applied parts, including through Y-capacitors (which are permitted in
  IEC 62368-1 but must be safety-rated Y-capacitors in IEC 60601-1)
- Input and output test voltages for dielectric strength tests (hi-pot tests) are
  higher in IEC 60601-1 (4000 V for 2 MOPP vs. 3000 V in IEC 62368-1)

---

## Related Topics

- [EMC Basics](emc_basics.md)
- [Environmental, REACH, RoHS](environmental_reach_rohs.md)
- [Conformal Coating](conformal_coating.md)
- [PCB Fabrication Constraints](../03_design_for_manufacturing/pcb_fabrication_constraints.md)
- [Power Supply Architecture](../01_schematic_design/power_supply_architecture.md)
