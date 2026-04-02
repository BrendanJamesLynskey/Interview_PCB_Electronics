# Stackup Design

## Prerequisites
- Understanding of transmission line theory: characteristic impedance, propagation velocity
- Basic dielectric properties: relative permittivity (Dk), loss tangent (Df)
- PCB fabrication process: laminate, prepreg, copper foil, etching
- Signal integrity fundamentals: return current paths, crosstalk

---

## Concept Reference

### Why Stackup Matters

The stackup defines the physical structure of every signal layer in the board. Every impedance-controlled trace, every return current path, every power delivery plane, and every EMC characteristic is a direct consequence of the stackup. Getting the stackup wrong at the start of a project cannot be fully compensated by routing. Getting it right enables disciplined layout with predictable electrical behaviour.

### The Four Functions of a Stackup

```
1. Signal routing layers  -- carry traces; must have controlled impedance
2. Reference planes       -- GND and power planes; provide return paths and shielding
3. Power distribution     -- PDN planes; provide low-inductance power delivery
4. Mechanical structure   -- board thickness, rigidity, via drill aspect ratio
```

### Dielectric Materials

| Property    | FR-4 (standard) | Rogers 4350B   | Megtron 6      |
|-------------|-----------------|----------------|----------------|
| Dk (at 1GHz)| 4.3 - 4.6       | 3.48           | 3.4 - 3.7      |
| Df (at 1GHz)| 0.018 - 0.025   | 0.0037         | 0.002          |
| Tg          | 130 - 170 degC  | ~280 degC      | ~185 degC      |
| Use case    | General purpose | RF/microwave   | >10 GHz SerDes |

**Dk** directly sets the propagation velocity and, combined with trace geometry, sets the characteristic impedance. Lower Dk = faster signals = wider traces for the same impedance.

**Df** (dissipation factor, loss tangent) sets insertion loss. At high frequencies, Df dominates losses over copper skin-effect losses. For PCIe Gen 4/5, NVMe, or 56 Gbps SerDes, low-Df materials are necessary to meet the channel loss budget.

### Characteristic Impedance: Microstrip and Stripline

**Microstrip** (trace on outer layer, single reference plane below):

```
     w
  ________
  ________   <-- trace (width w, thickness t)
  |      |   <-- dielectric (thickness h, Dk = Er)
  --------   <-- reference plane
```

Approximate formula (IPC-2141A):
```
Z0 = (87 / sqrt(Er + 1.41)) * ln(5.98*h / (0.8*w + t))
```

**Embedded microstrip**: Microstrip buried under a solder mask or additional dielectric layer. The effective Dk is between the substrate Dk and air (Dk=1), lowering the effective Dk slightly.

**Stripline** (trace buried between two reference planes):

```
  --------   <-- reference plane (GND or power)
  |      |   <-- dielectric (b1)
  ________   <-- trace
  |      |   <-- dielectric (b2)
  --------   <-- reference plane
```

Approximate formula:
```
Z0 = (60 / sqrt(Er)) * ln(4*b / (0.67*pi*(0.8*w + t)))
where b = b1 + b2 (total distance between planes)
```

Stripline advantages: better shielding, no radiation, symmetric reference planes eliminate odd/even mode splitting in differential pairs. Disadvantage: harder to fabricate, via stubs on through-hole vias can cause resonances.

**Differential impedance** (edge-coupled, common case):

For a target of 100 ohm differential (50 ohm single-ended each):
```
Zdiff = 2 * Z0 * (1 - 0.347 * exp(-2.9 * s/h))
where s = gap between traces, h = height above plane
```

Tighter spacing (smaller s) lowers differential impedance. A practical 100 ohm differential pair requires careful coordination of trace width and spacing during stackup planning.

### 4-Layer Stackup

The most common low-cost stackup. Suitable for boards up to ~100 MHz clocks with disciplined layout.

```
Layer 1  SIG      Top signal (microstrip)        35 um copper
         Core     0.18 mm FR-4
Layer 2  GND      Ground plane (reference)        35 um copper
         Prepreg  0.36 mm FR-4
Layer 3  PWR      Power plane (PDN)              35 um copper
         Core     0.18 mm FR-4
Layer 4  SIG      Bottom signal (microstrip)     35 um copper
         Total thickness: ~1.0 mm
```

**Signal-plane pairing:** L1 references L2 (GND). L4 references L3 (PWR). L3 provides a noisy reference for L4 signals because switching currents flow through it. Better: swap L3 to GND and use L2 as the power plane, making both signal layers reference GND. Power delivery then relies on wide pours and careful capacitor placement rather than a dedicated plane.

**Improved 4-layer:**
```
Layer 1  SIG      Top signal (references L2 GND)
Layer 2  GND      Ground plane
Layer 3  GND      Ground plane (or split GND/PWR)
Layer 4  SIG      Bottom signal (references L3 GND)
```

This gives both signal layers a solid GND reference. PDN uses copper pours on L1/L4 plus good decoupling capacitor placement.

### 6-Layer Stackup

Adds two additional routing layers and allows a more complete PDN. Suitable for FPGAs, multi-gigabit interfaces, and complex mixed-signal boards.

```
Layer 1  SIG      Top component/signal layer (microstrip)
         Prepreg  0.10 mm
Layer 2  GND      Ground plane (reference for L1 and L3)
         Core     0.36 mm
Layer 3  SIG      Inner signal (stripline, references L2 and L4)
         Prepreg  0.10 mm
Layer 4  PWR/GND  Power plane or second GND
         Core     0.36 mm
Layer 5  SIG      Inner signal (stripline, references L4 and L6)
         Prepreg  0.10 mm
Layer 6  SIG      Bottom component/signal layer (microstrip)
         Total: ~1.6 mm
```

**Key choice:** Should L4 be GND or PWR? If L4 is GND, you get excellent signal integrity on L3 and L5 (both sandwiched between GND planes). Power delivery is via planes on L3 or L5 with reduced area. If L4 is PWR, the EMC is slightly worse but PDN is stronger. For high-speed digital, L4 = GND is preferred; power distribution is handled by pours and decoupling.

### 8-Layer Stackup

```
Layer 1  SIG      Top (microstrip, refs L2)
Layer 2  GND      Ground reference
Layer 3  SIG      Inner signal (stripline, refs L2 and L4)
Layer 4  PWR      Power plane (3.3V or core)
Layer 5  GND      Ground plane
Layer 6  SIG      Inner signal (stripline, refs L5 and L7)
Layer 7  PWR      Power plane (1.8V or I/O)
Layer 8  SIG      Bottom (microstrip, refs L7)
```

Multiple power planes allow different voltage domains to have dedicated planes. L4/L5 together form a tight capacitive PDN pair (low inductance between them). The L2/L3/L4 triplet and L5/L6/L7 triplet each provide signal-between-planes routing.

**Signal layer assignment guidelines for 8+ layers:**
- Assign high-speed SerDes (PCIe, USB 3.x, SERDES) to inner stripline layers (L3, L6) -- lower EMI radiation, better return path.
- Use outer microstrip layers for lower-speed I/O, power/ground pour connections, and accessible probe points.
- Route clocks on inner layers adjacent to GND planes.
- Never route critical signals across a layer transition that crosses a plane split.

### 10+ Layer Stackups

High-complexity ASICs, FPGAs with many BGA balls, and boards with multiple high-speed interfaces (multiple PCIe slots, DDR5, 100G Ethernet) require 10-16 layer stackups.

```
Example 12-layer FPGA carrier:
L1   SIG     Top (BGA breakout, short traces)
L2   GND     Reference for L1
L3   SIG     Inner (high-speed SerDes)
L4   GND     Reference for L3
L5   SIG     Inner (DDR data/address)
L6   PWR     VCC core power plane
L7   GND     Ground midstack
L8   SIG     Inner (control signals, clocks)
L9   PWR     VCC I/O plane
L10  SIG     Inner (less critical signals)
L11  GND     Reference for L12
L12  SIG     Bottom (connectors, test points)
```

**Design rule for 10+ layers:** Every signal layer should have an adjacent (or nearby) reference plane of the correct voltage (GND preferred, power acceptable). Never stack two signal layers against each other without a reference plane between them -- this creates broadside-coupled transmission lines with uncontrolled impedance and crosstalk.

### Controlled Impedance Planning

Impedance targets are set early in design and communicated to the PCB fabricator via a controlled impedance note on the fab drawing. Standard targets:

| Interface     | Topology         | Target Impedance |
|---------------|------------------|-----------------|
| Single-ended  | Microstrip/Stripline | 50 ohm      |
| LVDS          | Edge-coupled diff pair | 100 ohm diff |
| PCIe (>= Gen1)| Edge-coupled diff pair | 85 ohm diff  |
| USB 2.0       | Edge-coupled diff pair | 90 ohm diff  |
| USB 3.x/HDMI  | Edge-coupled diff pair | 90 ohm diff  |
| DDR4/DDR5     | Single-ended      | 40-50 ohm    |

**Fabricator collaboration:** Share the stackup with the fabricator early. They will adjust core/prepreg thicknesses and copper weights to hit your impedance targets within +/-10% (or +/-5% for tight specs). Never finalise trace widths before the stackup is confirmed by the fabricator.

---

## Tier 1 -- Fundamentals

### Question F1
**What is a PCB stackup and why must it be defined before routing begins?**

**Answer:**

A PCB stackup is the ordered sequence of copper layers and dielectric materials that make up the board. It specifies: the number of layers, the thickness and material (Dk, Df) of each dielectric layer, the copper weight of each layer, and which layers are signal, ground, or power.

The stackup must be defined before routing because:

1. **Characteristic impedance** depends on trace width, dielectric thickness (h), and Dk. If you route a 50 ohm trace before knowing h and Dk, the trace width you choose will produce the wrong impedance on the actual board.

2. **Via drill-to-layer assignments** depend on how many layers exist and their order. Blind and buried via drill pairs are determined by the stackup.

3. **Return current paths** are determined by which planes are adjacent to which signal layers. Routing a signal before knowing which reference plane it sees can produce unintended return path voids.

4. **Differential pair geometry** (trace width and spacing for 100 ohm or 85 ohm differential) is calculated from the stackup dielectric height. The spacing and width combination changes with each stackup variant.

A board routed without a confirmed stackup will require trace width changes, layer reassignments, and potentially re-routing after the stackup is finalised -- wasting significant time.

---

### Question F2
**What is the difference between microstrip and stripline? Which gives better signal integrity and why?**

**Answer:**

**Microstrip:** A trace on an outer layer of the PCB, with a reference plane below it and air above it. The dielectric is a combination of the PCB material and air (approximated by an effective Dk).

**Stripline:** A trace on an inner layer, sandwiched between two reference planes above and below it. The dielectric is entirely the PCB material.

**Comparison:**

| Property             | Microstrip           | Stripline            |
|----------------------|----------------------|----------------------|
| Reference planes     | One (below)          | Two (above and below)|
| Dielectric           | Mixed (PCB + air)    | PCB only (uniform Dk)|
| EMI radiation        | Higher (radiates up) | Lower (shielded)     |
| Propagation velocity | Higher (lower Dk_eff)| Lower (full Dk)      |
| Crosstalk            | Higher               | Lower                |
| Impedance tolerance  | Moderate (+/-10%)    | Better controlled    |
| Via stub resonance   | Not an issue         | Can be an issue      |

Stripline gives better signal integrity for high-speed interfaces because:
- Both reference planes suppress radiation and confine the field
- Symmetric reference planes eliminate the odd/even mode impedance difference for differential pairs
- Lower crosstalk because the fields are fully contained

Microstrip is used on outer layers for component fanout, test points, and lower-frequency signals where the ease of access outweighs the SI disadvantage.

---

### Question F3
**A 4-layer board uses the stackup L1=SIG, L2=GND, L3=PWR, L4=SIG. Which signal layer has better signal integrity and why? What would you change?**

**Answer:**

**L1 has better signal integrity** than L4.

L1 signals reference L2 (GND). L2 is a solid, quiet reference plane. Return currents flow through GND, which is the same net connected throughout the board, providing a consistent low-impedance return path.

L4 signals reference L3 (PWR). L3 is a power plane. Power planes carry switching currents from all active devices on the board, and are therefore noisy relative to GND. The return current for an L4 signal must flow through the power plane, which introduces power supply noise onto the signal return path. This creates common-mode noise coupling and degrades signal quality.

Additionally, L3 (PWR) is often split into multiple power domains with gaps between them. Any L4 trace that routes across a gap in the L3 plane loses its reference plane continuity, creating an impedance discontinuity, a return current detour, and a potential EMC emission source.

**What to change:**

Option 1 -- Swap L3 to a second GND plane:
```
L1 SIG  (references L2 GND)
L2 GND
L3 GND   <-- change from PWR to GND
L4 SIG  (references L3 GND)
```
PDN is handled by copper pours on L1/L4 and well-placed decoupling capacitors.

Option 2 -- Use a buried capacitance stackup where L2 and L3 are very close GND/PWR pairs:
```
L1 SIG
L2 GND
L3 PWR   (very thin prepreg between L2 and L3 for distributed capacitance)
L4 SIG
```
The thin dielectric between L2 and L3 forms a distributed capacitor that improves PDN without compromising either signal layer's reference.

For any board above ~50 MHz, Option 1 is the safer choice.

---

## Tier 2 -- Intermediate

### Question I1
**You are designing a 6-layer board with PCIe Gen 3 x4 (four differential pairs TX, four differential pairs RX). Which layers would you assign these signals to and why?**

**Answer:**

PCIe Gen 3 has a data rate of 8 GT/s, corresponding to a fundamental frequency of 4 GHz and significant harmonic content to 8 GHz and beyond. The requirements are:

- 85 ohm differential impedance (PCIe specification)
- Low insertion loss channel (stay within loss budget, typically -28 dB at Nyquist)
- Low crosstalk between adjacent pairs
- Minimal radiation (EMC compliance)
- Clean return path (no plane splits)

**Recommended 6-layer assignment:**

```
L1  SIG   -- Component breakout only (short stubs to BGA/connector)
L2  GND   -- Reference plane
L3  SIG   -- Assign PCIe TX pairs here (inner stripline)
L4  GND   -- Reference plane for L3 and L5
L5  SIG   -- Assign PCIe RX pairs here (inner stripline)
L6  SIG   -- Component breakout and low-speed signals
```

**Reasons for L3 and L5 (inner stripline):**

1. Stripline is shielded between two GND planes (L2/L4 for L3, L4/L6-area for L5), minimising EMI radiation from the 4 GHz+ signals.
2. Inner layers have no exposure to the uneven dielectric of microstrip, giving tighter impedance control -- important for PCIe's 85 ohm +/-15% tolerance.
3. With GND on L4 between TX pairs (L3) and RX pairs (L5), there is layer-to-layer isolation between TX and RX, reducing far-end crosstalk that could cause compliance failures.
4. FR-4 Dk of ~4.4 at 4 GHz and the inner layer geometry gives an 85 ohm differential pair with approximately 100 um trace width and 200 um spacing -- well within typical fabrication capability.

**Via treatment:** PCIe through-hole vias will have stubs from L3 or L5 down to the unused lower portion of the via barrel. At Gen 3 frequencies, these stubs create resonances that add insertion loss. Either use back-drilling to remove the stubs, or select a stackup where the signals are on layers that minimise stub length.

---

### Question I2
**A board manufacturer quotes two stackup options for your 8-layer design. Option A uses 1080 prepreg between L1 and L2 (0.0635 mm thick, Dk=4.4). Option B uses 2116 prepreg (0.114 mm thick, Dk=4.3). Your target is 50 ohm single-ended microstrip on L1. Calculate the approximate trace width required for each option. What other factors would influence your choice?**

**Answer:**

Using the IPC-2141A microstrip formula:
```
Z0 = (87 / sqrt(Er + 1.41)) * ln(5.98*h / (0.8*w + t))
```
Rearranged for width (iterative, but can be estimated):

Copper thickness t = 35 um (1 oz copper on outer layer)
Target Z0 = 50 ohm

**Option A (1080 prepreg): h = 0.0635 mm, Er = 4.4**

```
50 = (87 / sqrt(4.4 + 1.41)) * ln(5.98*0.0635 / (0.8*w + 0.035))
50 = (87 / sqrt(5.81)) * ln(0.3797 / (0.8*w + 0.035))
50 = (87 / 2.41) * ln(0.3797 / (0.8*w + 0.035))
50 = 36.1 * ln(0.3797 / (0.8*w + 0.035))
1.385 = ln(0.3797 / (0.8*w + 0.035))
0.3797 / (0.8*w + 0.035) = e^1.385 = 3.994
0.8*w + 0.035 = 0.0951
0.8*w = 0.0601
w = 0.075 mm = 75 um
```

**Option B (2116 prepreg): h = 0.114 mm, Er = 4.3**

```
50 = (87 / sqrt(4.3 + 1.41)) * ln(5.98*0.114 / (0.8*w + 0.035))
50 = (87 / sqrt(5.71)) * ln(0.682 / (0.8*w + 0.035))
50 = (87 / 2.389) * ln(0.682 / (0.8*w + 0.035))
50 = 36.4 * ln(0.682 / (0.8*w + 0.035))
1.374 = ln(0.682 / (0.8*w + 0.035))
0.682 / (0.8*w + 0.035) = e^1.374 = 3.95
0.8*w + 0.035 = 0.1727
0.8*w = 0.1377
w = 0.172 mm = 172 um
```

**Summary:**

| Option | h        | Dk  | Required trace width |
|--------|----------|-----|----------------------|
| A      | 63.5 um  | 4.4 | ~75 um               |
| B      | 114 um   | 4.3 | ~172 um              |

**Other factors influencing the choice:**

- **Manufacturing yield:** 75 um traces (Option A) approach the capability limit of standard PCB fabrication (minimum 100 um is typical for class 2). A 75 um trace increases cost and risk of open-circuit defects. Option B at 172 um is well within standard capability.
- **BGA breakout density:** If L1 is used for BGA breakout with 0.5 mm or 0.4 mm pitch, thinner traces (Option A) enable more traces between pads.
- **Impedance tolerance:** Thinner prepreg (Option A) has greater percentage thickness variation due to resin flow during lamination, potentially giving worse impedance tolerance (+/-12% vs +/-8% for Option B).
- **Signal loss:** 1080 prepreg can have slightly higher Df than 2116 due to glass weave density. For high-frequency operation (>5 GHz), confirm the loss tangent specification.
- **Conclusion:** For a standard fabrication run, Option B (172 um traces) is the better choice unless BGA breakout density specifically requires the finer geometry of Option A.

---

### Question I3
**Explain signal-plane pairing. What happens if a high-speed signal transitions between layers that reference different planes?**

**Answer:**

**Signal-plane pairing** is the practice of ensuring that each signal layer has a clearly defined, continuous, single-net reference plane directly adjacent to it. Every trace on a layer "pairs with" its adjacent reference plane.

When a via transitions a signal from one layer to another, the return current must also transition from one reference plane to the other. If both reference planes are GND, the return current can flow between the two GND planes through nearby bypass capacitors or stitching vias. The inductance of this transition is low if stitching vias or capacitors are within 1/20 wavelength of the signal via.

**What happens when reference planes are different nets:**

```
Example:
  L1 SIG references L2 GND
  Via transitions signal from L1 to L3
  L3 SIG references L4 PWR (3.3V)

Return current path at L1: flows along L2 GND
Via transition: return current must jump from L2 GND to L4 PWR
                This path goes through a bypass capacitor or directly through the power rail
                That capacitor has significant ESL (>1 nH typically)
```

**Consequences:**

1. **Return path inductance spike:** The capacitor ESL introduces an inductive impedance discontinuity. At 1 GHz, 1 nH = 6.3 ohms. The signal sees this as a stub impedance that reflects energy and adds to insertion loss.

2. **Ground-to-power noise coupling:** The return current now flows through the PDN. Any switching noise in the 3.3V plane modulates the signal's reference, adding common-mode noise to the signal.

3. **EMC emission:** The detoured return current creates a larger current loop. Loop area is proportional to magnetic radiation. Even a 1 mm diameter loop at 1 GHz can radiate significantly.

4. **Crosstalk:** The return current spreading across the reference plane (instead of following tightly under the trace) couples to adjacent signals.

**Mitigation:**

- Route signals on layers that pair with GND planes only, whenever possible.
- Place stitching vias (GND-to-GND) within 500 um of any signal via that transitions between two GND-referenced layers.
- If a layer transition across a power plane is unavoidable, place a 100 nF bypass capacitor (GND-to-PWR) immediately adjacent to the signal via.
- Consider back-drilling and careful via placement to minimise stub lengths that interact with the return path discontinuity.

---

## Tier 3 -- Advanced

### Question A1
**You are designing a 12-layer board for a Xilinx UltraScale+ FPGA with PCIe Gen 4 x16, DDR4-3200, and 100G Ethernet. Walk through the complete stackup design process, explaining every layer assignment decision.**

**Answer:**

**Step 1: Enumerate interfaces and their requirements**

```
Interface        Layers needed    Impedance      Notes
PCIe Gen 4 x16   4 routing lyrs   85 ohm diff    8 pairs TX, 8 pairs RX
DDR4-3200        3 routing lyrs   40 ohm SE       Address, data (byte lanes), clocks
100G Ethernet    1 routing lyr    100 ohm diff    4x25G SERDES pairs
FPGA power       2 plane pairs    N/A            VCC_CORE, VCC_INT (sub-1V)
Board I/O        2 routing lyrs   50 ohm SE      JTAG, UART, GPIO, config
```

**Step 2: Determine layer count**

SerDes (PCIe + 100G) needs inner stripline layers for shielding. DDR4 wants to be close to the FPGA with consistent Dk. I/O can use outer layers. With two power plane pairs needed, 12 layers is appropriate:

```
L1   SIG    Top (FPGA/BGA breakout, short stub layer)
L2   GND    Primary reference -- GND (solid)
L3   SIG    Inner A (PCIe TX, 100G -- stripline between L2 and L4)
L4   GND    Inner reference -- GND (solid)
L5   SIG    Inner B (PCIe RX -- stripline between L4 and L6)
L6   PWR    VCC_INT plane (FPGA core power -- very thin prepreg above)
L7   GND    PDN reference -- GND (thin prepreg below L6 for cap effect)
L8   SIG    Inner C (DDR4 data -- stripline between L7 and L9)
L9   PWR    VCC_DDR plane
L10  GND    Bottom reference -- GND
L11  SIG    Inner D (DDR4 address/control -- stripline between L10 and L12)
L12  SIG    Bottom (board connectors, power bypass placement)
```

**Step 3: Justify each pairing**

- L1 (SIG) pairs with L2 (GND): FPGA BGA breakout traces are short. Microstrip on L1 is acceptable because lengths are <5 mm before vias transition signals to inner layers.
- L3 (SIG) pairs with L2 and L4 (GND/GND): PCIe TX and 100G SerDes in symmetric stripline. Both reference planes are GND -- best possible SI. Low EMI. Clean return transition via stitching vias at signal vias.
- L5 (SIG) pairs with L4 and L6 (GND/PWR): PCIe RX pairs here. L6 is a power plane (VCC_INT), slightly worse reference than GND. Mitigation: keep L6 continuous with no splits under L5 routing channels. Place GND-to-VCC_INT bypass caps within 500 um of PCIe RX via transitions.
- L6/L7 (PWR/GND): Chosen as a tightly coupled plane pair. If the fabricator uses 75 um prepreg between L6 and L7, the plane-to-plane capacitance is approximately 1 nF per cm^2, providing significant distributed PDN capacitance for VCC_INT.
- L8 (SIG) pairs with L7 and L9 (GND/PWR): DDR4 data byte lanes. DDR4 runs at 1600 MHz data rate (3200 MT/s), fundamental 800 MHz. This is tolerable on a GND/PWR sandwich. All DDR4 data routing stays within one layer; all byte lanes are on L8.
- L11 (SIG) pairs with L10 and L12 area: DDR4 address/command/clocks. These are lower frequency and less sensitive to the mixed GND/signal boundary at L12.

**Step 4: Impedance calculations**

For PCIe Gen 4 on L3/L5 (symmetric stripline):
- Target: 85 ohm differential
- With L3 at 0.1 mm between L2 and L4 (total separation 0.2 mm): symmetric stripline
- Single-ended target Z0 = 42.5 ohm
- Trace width ~100 um, spacing ~175 um (verified with field solver)
- Back-drill all through-vias from bottom after L5 to remove stubs that would resonate below 14 GHz

For DDR4 on L8:
- Target: 40 ohm single-ended (typical for DDR4 with discrete termination)
- Asymmetric stripline (closer to L7 than L9): adjust width accordingly (~200 um)

**Step 5: Via strategy**

- PCIe/SerDes: blind vias L1-L3 and L1-L5 with back-drilling, or through-vias with back-drilling from bottom
- DDR4: through-vias L1-L8 or L1-L11 (back-drill optional at DDR4 speeds)
- Power delivery: thermal/power vias on L6 and L9 arrays under FPGA footprint
- Stitching vias: GND stitching vias (L2-L4-L7-L10) grid at 2-3 mm spacing across the board

---

### Question A2
**During stackup review, your signal integrity engineer runs a field solver simulation and reports that your 100 ohm differential PCIe pair on a proposed inner stripline layer shows a common-mode impedance of 28 ohm instead of the expected 25 ohm (half the differential). What could cause this and what does it mean for signal integrity?**

**Answer:**

**Expected relationship:** For a perfectly symmetric differential pair in a homogeneous medium:
```
Z_diff = 2 * Z_odd
Z_common = Z_even / 2
Z_odd ≠ Z_even in general
For loosely coupled pairs: Z_diff ≈ 2 * Z_single_ended
For tightly coupled pairs: Z_diff < 2 * Z_SE, Z_common > Z_SE / 2
```

**Why common-mode impedance = 28 ohm instead of 25 ohm:**

The discrepancy (28 ohm vs 25 ohm expected) indicates **asymmetric coupling to the reference planes**. Possible causes:

1. **Asymmetric prepreg thickness above and below the trace layer:** If the prepreg above the trace is thicker than the core below, the trace is closer to one reference plane than the other. The even-mode (common-mode) impedance changes because the coupling asymmetry shifts the capacitance balance.

2. **Trace routing close to an adjacent power plane split:** Even-mode currents spread more widely than odd-mode currents. If one side of the differential pair is nearer a plane gap, its coupling to the reference changes asymmetrically.

3. **Glass weave effect (fibre weave effect):** The glass weave in the prepreg creates periodic variations in local Dk. If the two traces of the pair sit over different weave patterns (one over glass bundle, one over resin-rich region), their individual Dk values differ. This causes a Dk mismatch between the two traces, splitting the odd-mode and even-mode impedances from their expected symmetric values.

**What it means for signal integrity:**

1. **Mode conversion:** If the transmitter drives a pure differential signal, a common-mode impedance mismatch relative to the receiver's common-mode termination will reflect common-mode energy. This reflected common-mode energy appears as a noise floor elevation on the receive side.

2. **Differential-to-common-mode conversion:** Any asymmetry that makes the two traces electrically different (different Dk, different path length, different coupling) converts differential signal energy into common-mode energy. Common-mode energy is the primary source of radiated EMI from differential pair routing.

3. **EMC implication:** PCIe Gen 4 at 16 GT/s has significant harmonic content above 8 GHz. Even a few percent of mode conversion at these frequencies can cause EMC test failures.

**Mitigation strategies:**

- Use a symmetric stackup with equal prepreg thickness above and below the trace layer.
- Specify low-weave or spread-glass prepreg (e.g., 1067 or spread-glass variants) for SerDes layers to reduce fibre weave effect.
- Route differential pairs at a 45-degree angle relative to the glass weave direction (or specify cross-hatched weave materials).
- Run differential pairs with tight coupling (small gap) so odd-mode coupling dominates and mode conversion from asymmetry is reduced.
- Accept the 28 ohm common-mode impedance but add a common-mode choke on the PCIe pair near the connector to attenuate any common-mode conversion before it reaches the cable.
