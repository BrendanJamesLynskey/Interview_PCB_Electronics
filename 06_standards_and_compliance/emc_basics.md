# EMC Basics

## Prerequisites
- Understanding of electromagnetic fields and wave propagation
- Familiarity with Fourier analysis and frequency-domain thinking
- Basic RF concepts: transmission lines, shielding, impedance
- Knowledge of PCB stackup and power distribution network design

---

## Concept Reference

### What EMC Is and Why It Matters

Electromagnetic Compatibility (EMC) is the ability of equipment to function correctly
in its electromagnetic environment without introducing intolerable electromagnetic
disturbance to other equipment in that environment. It has two aspects:

- **Emissions:** The equipment must not radiate or conduct electromagnetic noise above
  prescribed limits. This protects other equipment from interference.
- **Immunity (susceptibility):** The equipment must continue to function correctly
  when exposed to external electromagnetic disturbances up to defined levels.

Regulatory bodies require both to be demonstrated before a product can be legally
sold in most markets.

---

### Regulatory Frameworks

#### CE Marking (European Economic Area)

CE marking is a self-declaration by the manufacturer that the product complies with
applicable EU directives. For electronic equipment, the relevant directive is:

**Radio Equipment Directive (RED) 2014/53/EU** — for products with intentional radio
transmitters/receivers.

**Electromagnetic Compatibility Directive (EMCD) 2014/30/EU** — for all other
electronic equipment. Requires conformity with harmonised standards.

Key harmonised EMC standards under the EMCD:
```
EN 55032      Emissions from multimedia equipment (replaces EN 55022 for IT equipment)
EN 55035      Immunity for multimedia equipment (replaces EN 55024)
EN 61000-3-2  Harmonic current emissions (mains-connected equipment)
EN 61000-3-3  Voltage fluctuations and flicker
EN 61000-4-x  Immunity test series:
  EN 61000-4-2  ESD immunity (contact and air discharge)
  EN 61000-4-3  Radiated RF immunity
  EN 61000-4-4  Electrical fast transient / burst (EFT/B)
  EN 61000-4-5  Surge immunity
  EN 61000-4-6  Conducted RF immunity
  EN 61000-4-8  Power frequency magnetic field immunity
  EN 61000-4-11 Voltage dips and interruptions
```

**Technical File (TF):** The manufacturer must compile a Technical File demonstrating
compliance. For EMC, this includes: test reports from an accredited laboratory (or
in-house testing with documented procedures), risk assessment, and a Declaration of
Conformity (DoC). Third-party testing is not mandatory for most products (unlike
safety certification) but in-house testing must be demonstrably rigorous.

#### FCC (United States — Federal Communications Commission)

The FCC regulates electromagnetic emissions under **Part 15** of Title 47 CFR for
unintentional emitters. Equipment falls into two classes:

```
Class A: Commercial/industrial use (offices, factories, not residential)
          Limits are less stringent than Class B
          Self-declaration with test report; SDoC (Supplier's Declaration of Conformity)

Class B: Residential use (or equipment marketed for residential use)
          More stringent limits
          Most consumer products must be Class B
          May require third-party test lab for certification (FCC ID) if using
          intentional radiators (Wi-Fi, Bluetooth, cellular)
```

For intentional radiators, FCC certification requires testing at an FCC-recognised
laboratory and registration of an FCC ID. This applies to any product with a wireless
radio that transmits in a licensed or unlicensed frequency band.

**FCC vs. CE limits comparison:**
```
Frequency range    CE Class B limit (V/m)    FCC Class B limit (µV/m at 3 m)
10 MHz             (conducted range)          (conducted range)
30 MHz             30 dBµV/m at 10 m          100 µV/m at 3 m ≈ 40 dBµV/m at 3 m
100 MHz            37 dBµV/m at 10 m          150 µV/m at 3 m
230 MHz            37 dBµV/m at 10 m          150 µV/m at 3 m
1 GHz              47 dBµV/m at 10 m          500 µV/m at 3 m
```

FCC Class B limits are numerically similar to EN 55032 Class B but measured at 3 m
vs. 10 m (6 dB difference due to far-field inverse distance law).

#### UKCA Marking (United Kingdom)

Post-Brexit, the UK uses its own **UKCA (UK Conformity Assessed)** marking for products
placed on the market in Great Britain (England, Scotland, Wales). Northern Ireland
continues to accept CE marking under the Windsor Framework.

UKCA requirements are essentially identical to CE requirements — the same test standards
(BS EN equivalents of EN standards) apply. Key practical difference: a UK-specific
Declaration of Conformity must be created, referencing UK legislation rather than EU
directives. CE marking alone is no longer sufficient for the GB market.

Compliance timeline:
- From 1 January 2025, UKCA marking is required for most regulated products on the
  GB market. CE marking accepted by transitional provisions for specific product types
  is being phased out.

---

### EMC Emissions Sources

Emissions arise from current loops and their associated electric/magnetic fields.
The two fundamental emission mechanisms are:

**Differential mode (DM) emissions:**
Current flowing in the forward and return conductors of a signal or power path. The
loop formed by these two conductors is the antenna. DM emissions are minimised by
minimising the loop area.

```
Radiated field from a small current loop (loop area A, current I, frequency f):
  E ≈ 1.32 × 10⁻¹⁴ × f² × A × I   (V/m at 3 m distance)

Key insight: E ∝ f² — doubling frequency quadruples radiated emission.
             E ∝ A  — halving loop area halves radiated emission.
```

**Common mode (CM) emissions:**
Current flowing in the same direction on both the forward and return conductors (on
a cable, this is current flowing through the cable shield to chassis ground). CM
currents are driven by the voltage difference between the PCB ground and the local
earth/chassis. CM emissions are often the dominant mechanism for conducted EMC failures.

```
Common mode cable current drivers:
  - Switching converter voltage at chassis connections
  - Signal ground potential differences between two connected boards
  - Cable shields connected to PCB ground at one end only
  - Cable routed near switching nodes without a guard trace
```

---

### EMC Shielding

Shielding attenuates electromagnetic fields between the shielded region and the
outside world. Shielding effectiveness (SE) is expressed in dB:

```
SE = 20 × log10(E_unshielded / E_shielded)

SE = 60 dB means the shielded field is 1000× weaker than the unshielded field
```

Shielding works by two mechanisms:
1. **Reflection:** The impedance mismatch at the shield surface reflects incident
   waves. Effective for low-impedance shields (conductors) against high-impedance
   (electric field) sources.
2. **Absorption:** Energy is absorbed in the shield material (converted to heat by
   eddy currents). The skin depth determines how thick the shield must be for a given
   frequency.

```
Skin depth δ = √(ρ / (π × f × µ))

Where:
  ρ = resistivity (Ω·m)
  f = frequency (Hz)
  µ = permeability = µ0 × µr

Skin depth examples at 100 MHz:
  Copper (ρ = 1.68×10⁻⁸, µr = 1): δ = 6.6 µm
  Aluminium (ρ = 2.65×10⁻⁸, µr = 1): δ = 8.3 µm
  Steel (ρ = 1.0×10⁻⁷, µr = 100): δ = 0.9 µm
```

For frequencies above 100 kHz, a 1 mm thick copper or aluminium shield provides
complete absorption attenuation. The limiting factor for practical shielding is
**aperture leakage** — any opening in the shield (connector cutout, seam, ventilation
hole) allows field penetration.

**Maximum aperture size rule:**
```
An aperture acts as a slot antenna and re-radiates energy above the frequency
where the aperture dimension equals λ/2:

f_max = c / (2 × L_aperture)

For a 10 cm aperture: f_max = 3×10⁸ / (2 × 0.1) = 1.5 GHz
For a 1 cm aperture: f_max = 15 GHz

Rule of thumb: limit longest aperture dimension to λ/20 at the highest frequency
of concern. At 1 GHz (λ = 30 cm), maximum aperture = 1.5 cm.
```

**PCB-level shielding (shield cans):**
Pressed metal or solderable shield cans over sensitive RF or digital circuits.
Typical applications:
- Wireless module isolation (Bluetooth, Wi-Fi, cellular modem)
- ADC and DAC shielding from nearby digital noise sources
- Crystal oscillator isolation

Requirements for PCB shield cans to be effective:
1. Continuous solder joint along the entire perimeter (no gaps)
2. Ground connection at the base of the shield (the fence must contact a low-impedance
   ground plane, not just a few isolated pads)
3. Removal/access mechanism designed into the top (press-fit lid or removable top section)
4. No signal vias penetrating the shield from outside (a via from outside the shield
   to inside carries the external noise in)

---

### EMC Filtering

Filtering prevents noise from leaving or entering equipment through conducted paths
(power cables, signal cables).

**Power line filtering for conducted emissions:**

```
Typical AC mains filter structure:
                    L1 (CM choke)
Line ────── Cx ────┤├──── Line (to load)
                   │
           Cy     Cy       (Y-capacitors between line and earth/chassis)
                   │
Neutral ────────── Neutral (to load)

Cx = X-rated capacitor (between line and neutral, class X1 or X2)
Cy = Y-rated capacitor (between line/neutral and earth, class Y1 or Y2)
L1 = Common-mode choke (high impedance to CM current, low to DM)
```

**X capacitors** (connected across live conductors): must survive mains voltage plus
transients. Rated for continuous mains operation. Safety class X2 for most mains
applications, X1 for circuits connected to mains with severe transients.

**Y capacitors** (connected to protective earth): must not fail short (a failed-short
Y cap creates a shock hazard by connecting live mains to the chassis). Rated for
continuous mains-to-earth application. Safety class Y2 for 250 Vrms mains, Y1 for
higher voltage.

**Common-mode choke (CMC):**
A toroidal or EE-core inductor wound with both conductors in opposite directions (so
differential current produces cancelling flux). Result: high impedance to common-mode
current (both conductors carry current in same direction — flux adds), low impedance
to differential (normal) current (flux cancels).

```
Typical CMC characteristics:
  Impedance to CM current: 1 kΩ to 1 MΩ (depending on frequency and inductance)
  Insertion loss in DM path: < 0.5 dB (low insertion loss is essential)
  Rated current: must handle full load current without saturation
```

**DC-DC converter input filter:**
Every switching converter generates conducted noise on its input rail. The minimum
filter required:
```
L_input ─── C_bulk ─── Converter

L_input: series inductance (ferrite bead or inductor) to attenuate high-frequency
         noise from propagating back to the input
C_bulk:  bulk capacitor to provide local charge reservoir for switching transitions
         (reduces conducted noise at the switching frequency)

Ferrite bead selection: choose Rdc < 0.1 Ω to avoid excessive voltage drop under load.
At the switching frequency and harmonics (100 kHz - 10 MHz), the ferrite bead
should present > 200 Ω of impedance.
```

---

### PCB Layout for EMC

EMC performance is largely determined by PCB layout, not just by schematic components.

**Key layout rules for EMC:**

1. **Minimise switching loop areas:**
   The loop formed by the switching current path (converter input capacitor → MOSFET
   → inductor → output capacitor → back to input) must be as small as possible.
   This loop radiates at the switching frequency and its harmonics.

2. **Unbroken ground plane under high-speed signals:**
   Return current follows the path of least impedance, which at high frequency is
   directly under the signal trace. A split or cut in the ground plane forces the
   return current to find a longer path, increasing loop area and emissions.

3. **Filter components at cable/connector entry points:**
   EMC filter capacitors and ferrite beads must be placed at the point where the
   cable enters the board, before the noise can propagate to internal circuitry.
   Filter components placed far from the connector are less effective because the
   trace between the connector and the filter acts as a receiving antenna.

4. **Separate analogue, digital, and power ground areas:**
   While a unified ground plane is preferred, high-current switching ground currents
   must be kept away from sensitive analogue ground areas. Achieve this by controlling
   the current flow path through the ground plane geometry, not by splitting the plane.

5. **Place decoupling capacitors as close as possible to IC power pins:**
   Every digital IC switching event causes a current spike on the supply. Decoupling
   capacitors provide the charge locally, preventing the spike from propagating along
   the supply rail and radiating.

---

## Standards and Regulations Summary

```
Region    Mark    Directive/Act           Key EMC Standards
--------  ------  ----------------------  -----------------------------------------
EU/EEA    CE      EMCD 2014/30/EU         EN 55032, EN 55035, EN 61000-4-x series
                  RED 2014/53/EU          EN 300 328 (Wi-Fi/BT), EN 301 893 (5 GHz)
USA       FCC ID  47 CFR Part 15          ANSI C63.4 (radiated), CISPR 32 (multimedia)
          SDoC
UK        UKCA    UKEMCR 2016,            BS EN 55032, BS EN 55035 (UK equivalents)
                  RATEUR 2017
Japan     VCCI    Voluntary Control        Based on CISPR 22 / CISPR 32
Canada    ISED    ICES-003                 Based on ANSI C63.4 / CISPR 32
Australia RCM     ERAC, ACMA              Based on CISPR 32 / IEC 61000 series
```

---

## Best Practices

- Conduct a **pre-compliance scan** with a near-field probe and spectrum analyser before
  formal testing. This identifies problem frequencies early, when correction is low cost.
- Route **high-speed signals on inner layers** between reference planes to reduce radiated
  emissions and improve immunity.
- Use a **single earth point** for chassis ground connection to avoid ground loops.
- Add **ferrite beads on all cable shield connections** where the cable enters the enclosure.
- Ensure **all connector mounting flanges make solid electrical contact with the chassis.**
  A floating connector shell creates a gap in the shielding.
- Keep a **compliance test plan** as a living document from the start of design.
  Retroactively designing for EMC compliance is significantly more expensive than
  proactively designing for it.

---

## Tier 1 — Fundamentals

### Question F1
**What is the difference between conducted emissions and radiated emissions? Give a PCB example of each.**

**Answer:**

**Conducted emissions** are electromagnetic noise that propagates along a conductor —
a power supply cable, signal cable, or PCB trace — and can be measured by monitoring
the current or voltage on that conductor. Example: a switching buck converter on a PCB
generates high-frequency noise (at its switching frequency and harmonics) on its input
power rail. This noise propagates back through the power cable to the mains and can
interfere with other equipment connected to the same supply. Conducted emissions are
tested by measuring the noise voltage on the mains supply (LISN measurement, typically
150 kHz to 30 MHz).

**Radiated emissions** are electromagnetic noise that propagates through the air as
an electromagnetic wave, not via a conductor. Example: a clock signal at 100 MHz on
a PCB trace acts as a short monopole antenna and radiates into the surrounding space.
The signal and its harmonics can be received by radio equipment at some distance from
the board. Radiated emissions are tested by measuring the electric field strength at
a distance (typically 3 m or 10 m in an anechoic chamber, above 30 MHz).

The frequency boundary is not sharp: a PCB trace that conducts noise to a cable
connector effectively converts the conducted noise into radiated emissions via the
cable acting as an antenna. Both must be controlled.

---

### Question F2
**What is a common-mode choke and how does it differ from a normal inductor in a power line filter?**

**Answer:**

A common-mode choke (CMC) is a paired inductor wound on a shared magnetic core with
both conductors (live and neutral, or signal and return) wound together. The windings
are oriented so that:
- **Differential-mode current** (normal current, flowing out on one conductor and back
  on the other) produces equal and opposite magnetic flux — they cancel, leaving very
  low inductance. The CMC appears nearly transparent to normal operating current.
- **Common-mode current** (noise current flowing in the same direction on both conductors,
  returning via earth/chassis) produces additive magnetic flux — high inductance results.
  The CMC presents a high impedance to CM noise, attenuating it.

A normal inductor in series with one conductor would attenuate both differential and
common-mode currents, causing unacceptable voltage drop under load. A CMC selectively
attenuates only the unwanted CM noise while carrying the full load current with minimal
impedance.

---

## Tier 2 — Intermediate

### Question I1
**A product fails FCC Part 15 Class B radiated emissions at 144 MHz and 288 MHz. The PCB contains a microcontroller running at 144 MHz. What are the most likely causes and what are the remedial actions, from most likely to least invasive?**

**Answer:**

The 144 MHz fundamental and 288 MHz second harmonic are consistent with the 144 MHz
clock and its second harmonic radiating from the PCB or attached cables.

**Most likely causes and remedial actions in order of likely impact and invasiveness:**

1. **Long clock trace or cable acting as an antenna (most likely).**
   A 144 MHz signal has λ ≈ 2.08 m. A quarter-wave monopole at 144 MHz is 52 cm. Any
   cable longer than a few centimetres attached to a pin or trace carrying the 144 MHz
   clock will be an effective antenna.
   - Remedy: Add a ferrite bead (high impedance at 144 MHz) in series on the cable or
     at the cable attachment point. This is the lowest-invasive fix.

2. **Incomplete ground plane / ground plane slots near the clock signal.**
   If the clock signal trace crosses a gap in the ground plane, the return current is
   forced along a longer path, creating a larger loop that radiates more effectively.
   - Remedy: For an in-production board, add copper tape bridging the slot (temporary
     fix to confirm hypothesis). For the next PCB spin, eliminate ground plane cuts.

3. **Inadequate decoupling at the MCU clock output driver.**
   If the clock signal has fast edges (short rise time), its harmonics have more energy.
   Slowing the edge rate by adding a series resistor (22-47 Ω) at the MCU clock output
   pin reduces harmonic content.
   - Remedy: Add a 33 Ω series resistor close to the MCU clock output pin. This slows
     the edge and reduces harmonic content without affecting the clock function.

4. **No spread-spectrum clocking (SSC) enabled.**
   If the MCU supports spread-spectrum clock modulation (dithers the clock frequency
   by ±0.5-1%), the spectral power is spread over a wider bandwidth, reducing the peak
   at 144 MHz by 10-15 dB.
   - Remedy: Enable SSC in the MCU clock configuration register (if supported).

5. **Chassis and enclosure apertures coinciding with λ/2 at 144 MHz.**
   A chassis aperture of λ/2 = 104 cm would resonate; at λ/4 = 52 cm, it re-radiates.
   - Remedy: Add conductive gaskets to seams; reduce aperture size below λ/20 = 10 cm.

---

## Tier 3 — Advanced

### Question A1
**A power supply designer proposes placing the Y-capacitors (line-to-earth) on the secondary (output) side of an isolated DC-DC converter rather than on the primary (mains input) side. Explain the EMC and safety implications of this decision.**

**Answer:**

**EMC implication:**

In an isolated converter, common-mode noise is generated by high dV/dt transitions
across the isolation barrier — specifically, the primary switching voltage couples
through the transformer interwinding capacitance to the secondary side. This capacitive
coupling drives a common-mode current that flows from the secondary output, through
the chassis/earth, and back to the primary mains.

The standard Y-capacitor placement is on the **primary side**, connected between live
or neutral and the primary earth ground. This provides a low-impedance return path for
the CM current on the primary side, before it reaches the mains.

If Y-capacitors are placed on the secondary side only:
- The primary-to-secondary interwinding coupled CM current still flows, but now the
  secondary Y-caps must handle the return path. The isolation barrier CM impedance
  (dominated by the interwinding capacitance Cps) is in series with this path.
- This may be effective for controlling secondary-side CM noise (e.g., noise from
  secondary switching or load transients coupling to earth), but it does not address
  primary-to-secondary coupled CM noise as effectively as primary-side Y-caps.
- Optimal EMC performance uses Y-caps on both sides — primary Y-caps for primary
  CM noise return path, secondary Y-caps for secondary-side noise.

**Safety implication:**

Y-capacitors connected to protective earth must meet IEC 60384-14 safety ratings:
- Class Y1: up to 250 V peak, reinforced isolation (for equipment where failure of
  the capacitor would connect live mains to accessible parts)
- Class Y2: up to 250 Vrms, basic isolation

On the secondary side of an isolated converter, if the secondary output is accessible
to users (it is the product's output terminals), a failed-short Y-capacitor would
connect the secondary output to the mains earth. Depending on the secondary voltage
and the system grounding arrangement, this could create a leakage current path.

IEC 60950-1 / IEC 62368-1 limit the earth leakage current from secondary circuits
to the earth to typically ≤ 0.5 mA (< 0.25 mA for portable equipment) to prevent
electric shock risk. Over-specifying Y-capacitor values in pursuit of EMC performance
can push the leakage current above the safety limit.

**Conclusion:**
Secondary-side Y-caps are valid and common, but must be verified against leakage
current safety limits. A balanced approach uses both primary and secondary Y-caps,
with values determined by the required CM attenuation and the maximum allowable
leakage current.

---

## Related Topics

- [Safety and Creepage](safety_and_creepage.md)
- [Conformal Coating](conformal_coating.md)
- [Stackup Design](../02_pcb_layout/stackup_design.md)
- [Return Current Paths](../02_pcb_layout/return_current_paths.md)
- [Bring-Up Methodology](../05_board_bringup/bringup_methodology.md)
