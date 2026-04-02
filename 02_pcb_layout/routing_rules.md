# Routing Rules

## Prerequisites
- Transmission line theory: characteristic impedance, propagation velocity, reflection
- PCB stackup design: signal layer assignment, reference planes
- Differential pair fundamentals: odd mode, even mode, skew
- Signal integrity: crosstalk, return path inductance

---

## Concept Reference

### Why Routing Rules Exist

Every trace on a PCB is a transmission line. Without enforced routing rules, traces become unterminated stubs, differential pairs accumulate skew, return currents take high-impedance detours, and crosstalk degrades signal margins. Routing rules encode the electrical physics of high-speed signalling into geometric constraints that EDA tools can check and enforce.

A well-configured DRC rule set catches the majority of signal integrity and EMC issues before a single board is built.

### Differential Pair Impedance Fundamentals

A differential pair consists of two traces carrying complementary signals. The differential impedance (tip-to-tip) is determined by the geometry of both traces relative to their reference plane.

**Edge-coupled microstrip (outer layer):**

```
     w      s      w
  _______         _______
  _______   gap   _______   traces (width w, gap s)
  |                      |
  ========================   reference plane (height h below)
```

Differential impedance (approximation):

```
Zdiff = 2 * Z0_single * (1 - 0.347 * exp(-2.9 * s/h))

Where:
  Z0_single = single-ended impedance of one isolated trace
  s = gap between trace edges (not centre-to-centre)
  h = trace height above reference plane
```

For loose coupling (s/h > 2), the coupling term is negligible and Zdiff ≈ 2 × Z0_single. For tight coupling (s/h < 0.5), the differential impedance is meaningfully lower than 2 × Z0_single.

**Edge-coupled stripline (inner layer):**

```
========================   reference plane
  |                    |
  _______   gap   _______   traces between planes
  |                    |
========================   reference plane
```

Symmetric stripline differential impedance (approximate):

```
Zdiff_stripline = 2 * Z0_odd

where Z0_odd is calculated using the stripline formula with effective
dielectric height reduced by coupling factor.
```

**Common differential impedance targets:**

| Interface | Target Zdiff | Tolerance |
|---|---|---|
| PCIe Gen 1-5 | 85 Ω | ±15% (spec), ±10% (best practice) |
| USB 2.0 | 90 Ω | ±15% |
| USB 3.x | 90 Ω | ±10% |
| HDMI | 100 Ω | ±15% |
| LVDS, general | 100 Ω | ±10% |
| 1000BASE-T Ethernet | 100 Ω | ±10% |
| SATA | 100 Ω | ±10% |
| DisplayPort | 100 Ω | ±10% |

### Length Matching

Length matching ensures that signals within a group (differential pair, byte lane, bus) arrive at the receiver simultaneously. Skew causes inter-symbol interference, setup/hold violations, and compliance failures.

**Intra-pair skew (within a differential pair):**

```
Skew = length_difference × (propagation delay per unit length)
     = ΔL × (sqrt(epsilon_r_eff) / c)

For FR4 (Er_eff ≈ 4.0), propagation delay ≈ 167 ps/inch (6.7 ps/mm)

Example: 1 mm skew on a 100 Ω LVDS pair:
  Skew = 1 mm × 6.7 ps/mm = 6.7 ps
  For 1.25 Gbps LVDS (800 ps bit period), this is 0.8% of a bit -- acceptable
  For 6.25 Gbps (160 ps bit period), this is 4.2% of a bit -- marginal
```

**Common length-matching budgets:**

| Interface | Intra-pair (P to N) | Inter-pair (lane to lane) | Byte lane to byte lane |
|---|---|---|---|
| PCIe Gen 3 | ≤ 5 mil (0.127 mm) | ≤ 20 mil (0.508 mm) | ≤ 500 mil (12.7 mm) |
| PCIe Gen 4/5 | ≤ 2.5 mil (0.064 mm) | ≤ 10 mil (0.254 mm) | ≤ 250 mil (6.35 mm) |
| USB 3.2 | ≤ 5 mil | ≤ 50 mil | N/A |
| DDR4-3200 | ≤ 5 mil | Byte lane: ≤ 25 mil | ≤ 100 mil |
| 100G Ethernet | ≤ 2.5 mil | ≤ 10 mil | N/A |
| LVDS (general) | ≤ 10 mil | ≤ 25 mil | N/A |

**Serpentine routing for length matching:**

When one trace of a pair is shorter than its partner, a serpentine (accordion) pattern adds length. Rules for serpentine routing:

```
Serpentine constraint: Minimum separation between adjacent serpentine segments
must exceed 3× the trace-to-reference-plane height (h) to prevent the segments
from coupling to each other and creating a local differential impedance anomaly.

For h = 0.1 mm (microstrip):  minimum serpentine spacing = 3 × 0.1 = 0.3 mm
For h = 0.3 mm (stripline):   minimum serpentine spacing = 3 × 0.3 = 0.9 mm

Do NOT serpentine a differential pair by adding length to one trace only.
Always match both traces from the last common-mode reference point (e.g.,
the BGA pad or the connector) to the receiver.
```

### Crosstalk Rules

Crosstalk occurs when the electromagnetic field of one trace induces a voltage or current on an adjacent trace. Forward crosstalk (FEXT) and backward crosstalk (NEXT) have different mechanisms and different sensitivity to trace geometry.

**Crosstalk reduction rules:**

```
3W rule: To reduce crosstalk to below 10%, maintain a trace-to-trace edge
         separation of at least 3 times the trace width (measuring centre-to-centre
         distance >= 3 × W).

Example: 0.15 mm trace width
  Centre-to-centre spacing: 3 × 0.15 = 0.45 mm minimum
  (Edge-to-edge gap: 0.45 - 0.15 = 0.30 mm)
```

The 3W rule is a practical approximation. For critical signals (clocks, high-speed buses), a 5W spacing is preferred.

**Layer-to-layer crosstalk (broadside coupling):**

Two signal layers directly adjacent to each other (with no reference plane between them) form a broadside-coupled pair with very strong crosstalk. This is why consecutive signal layers must always be separated by a reference plane.

**Common rule:** Never route two high-speed signal layers adjacent to each other without an intervening reference plane.

### Routing Direction Rules

To minimise coupling between adjacent signal layers, enforce alternating routing directions:

```
Layer 1 (SIG): Horizontal routing only
Layer 3 (SIG): Vertical routing only
Layer 5 (SIG): Horizontal routing only
Layer 6 (SIG): Vertical routing only
```

With this convention, adjacent signal layers cross at approximately 90° everywhere, minimising the length over which they run in parallel and therefore minimising inductive and capacitive coupling.

### Via Rules for High-Speed Signals

**Via stub resonance:** Through-hole vias leave an unused stub on layers below the signal layer. The stub resonates at:

```
f_resonance = c / (4 × L_stub × sqrt(Er))

For L_stub = 1 mm, Er = 4.1:
  f = 3e8 / (4 × 0.001 × 2.025) = 37 GHz -- no problem for PCIe Gen 4
For L_stub = 2.5 mm (thick backplane):
  f = 3e8 / (4 × 0.0025 × 2.025) = 14.8 GHz -- problem for PCIe Gen 5 (16 GHz)
```

Mitigation:
- Use blind vias (eliminate stub entirely for outer-layer-to-inner-layer transitions)
- Specify back-drilling in fab notes to remove the stub after lamination
- Minimise board thickness in the signal layer region

**Via transition: keep to GND pairs:**

Every signal via transition should have a nearby (within 500 µm) GND via to provide a low-impedance return path for the return current. Place GND stitching vias immediately adjacent to every signal via used for a high-speed interface.

---

## Tier 1 -- Fundamentals

### Question F1
**What is the 3W rule and why does it exist? Give an example of when you would apply it.**

**Answer:**

The 3W rule states that the centre-to-centre distance between two adjacent traces should be at least 3 times the trace width to reduce inductive and capacitive crosstalk between them.

The physical basis is the electromagnetic field around a trace. The energy in the field is concentrated near the trace but extends outward. The field falls off with distance; at a centre-to-centre distance of 3W, the mutual coupling between two traces is reduced to approximately 10% of the self-coupling (roughly -20 dB isolation). Closer spacing means more coupling.

**Why it matters:**
- Inductive coupling (near-end crosstalk, NEXT): a current change on the aggressor induces a voltage on the victim trace at the source end.
- Capacitive coupling: a voltage change on the aggressor induces a current on the victim trace, creating differential noise.

At high frequencies, both mechanisms combine to create significant signal degradation on victim traces.

**Example application:**

A board carries an SPI bus (MOSI, MISO, SCK, CS) running at 50 MHz with 0.15 mm traces. The 3W rule says keep centre-to-centre spacing at least 0.45 mm. If the SPI bus runs adjacent to a UART RX line, applying 3W spacing prevents the fast SCK edge from coupling noise into the UART and causing framing errors.

For clock traces, which are the worst aggressors because they switch continuously, 5W spacing is preferred over 3W.

---

### Question F2
**What is intra-pair skew in a differential pair, and what causes it? How does it degrade signal integrity?**

**Answer:**

**Intra-pair skew** is the timing difference between the positive and negative traces of a differential pair caused by a difference in their physical lengths. If the positive trace is 1 mm longer than the negative trace, the positive signal arrives at the receiver approximately 6–7 ps later than the negative signal.

**Causes of skew:**

1. **Routing asymmetry:** The positive and negative traces take different paths around vias, pads, or obstructions, resulting in different lengths.
2. **Via asymmetry:** If only one trace of the pair passes through a via while the other does not, the via adds additional electrical length (~0.3–0.5 mm equivalent trace length).
3. **Dielectric variation (fibre weave effect):** If the two traces of the pair overlie different weave patterns in the PCB glass fibre, their effective Dk values differ slightly, causing different propagation velocities even with equal physical lengths.
4. **Length matching error:** The serpentine routing used to match lengths was applied to the wrong trace, making the longer trace even longer.

**How it degrades signal integrity:**

A differential receiver measures V(P) - V(N). In the ideal case, P and N switch simultaneously. If P arrives Δt later than N, then for a period of Δt at the receiver, both P and N are at the same level (both still at the old logic level), giving a differential voltage of zero. This momentary zero differential creates:
- Reduced eye height at the receiver (noise margin reduction)
- Common-mode noise at the timing offset (the differential signal becomes a pulse of common-mode voltage at the transition, which can radiate as EMI)
- At sufficient skew, violation of the receiver's differential input setup/hold time requirements

For PCIe Gen 5, the eye mask is very tight (approximately ±25 mV at the receiver). Intra-pair skew of even 2–3 ps can close the eye at 32 Gbps.

---

### Question F3
**A design calls for routing a DDR4 data byte lane with eight data signals (DQ[7:0]) and one DQS differential strobe pair. What length matching constraints apply within the byte lane, and what is the consequence of violating them?**

**Answer:**

**DDR4 byte lane length matching constraints:**

The DQS strobe is the timing reference for the byte lane. All eight DQ signals must arrive at the DRAM at the same time as the DQS edge that captures them.

```
Constraint 1: Intra-pair skew of DQS differential pair
  Requirement: DQS+ and DQS- must be length-matched to within ≤ 5 mil (0.127 mm)
  Reason: Imbalanced DQS pair creates common-mode noise at the DRAM clock input,
          increasing jitter on the capture clock.

Constraint 2: DQ-to-DQS matching within the byte lane
  Requirement: Each DQ trace must be matched to the DQS length within ≤ 25 mil (0.635 mm)
  Tolerance depends on FPGA/memory controller specification -- tighter values (≤ 10 mil)
  are common for DDR4-3200 and above.
  Reason: The DQS strobe captures data at its rising and falling edges.
          If a DQ signal arrives more than half a UI (unit interval) before or after
          the DQS edge, a setup or hold violation occurs.

Unit interval for DDR4-3200:
  UI = 1 / (3200e6 × 2) = 156 ps    (DDR4 is double-data-rate)
  Half UI = 78 ps
  At 6.7 ps/mm propagation delay, 78 ps = 11.6 mm maximum mismatching --
  BUT the memory controller requires setup and hold margins, so the effective
  budget is much less (typically ±10 mil = ±0.254 mm = ±1.7 ps).
```

**Consequence of violating constraints:**

If DQ traces are significantly longer than DQS, the data is not stable at the DRAM input when the strobe captures it -- a hold time violation. The DRAM may capture incorrect data. In a DRAM memory interface this manifests as:
- Sporadic memory errors (detected by ECC if enabled)
- Memory training failure during board bring-up (the DRAM controller cannot find a valid DQ window)
- Hard data corruption that does not reproduce under all temperature and voltage conditions (making debug extremely difficult)

---

## Tier 2 -- Intermediate

### Question I1
**You need to route a USB 3.2 Gen 1 differential pair (5 Gbps, 90 Ω differential) from a microcontroller on Layer 1 to a USB-C connector also on Layer 1. The route is 45 mm long. What specific routing rules apply and what routing mistakes would cause a compliance failure?**

**Answer:**

**Applicable routing rules:**

**Impedance:**
- Target: 90 Ω ±10% differential (81–99 Ω)
- Calculate trace width and gap from the stackup: for a typical 0.1 mm prepreg microstrip, a 90 Ω differential pair requires approximately W = 0.175 mm, S = 0.175 mm (see IPC-2141A calculation)
- Must maintain consistent width and spacing throughout the entire 45 mm route -- any taper, via, or component pad creates an impedance discontinuity

**Length matching:**
- Intra-pair: ≤ 5 mil (0.127 mm) between the USB3_DP and USB3_DN traces
- Total route: 45 mm is within USB 3.2 channel compliance (maximum recommended ~100 mm on FR-4 for Gen 1)

**Reference plane:**
- Route must stay over a continuous, solid reference plane for the full length
- If the route crosses any plane split, gap, or cutout, the impedance is undefined at that crossing point

**Separation from other signals:**
- Apply 3W rule (minimum spacing from all other signals): for 0.175 mm trace, 3W = 0.525 mm centre-to-centre from any other trace
- Keep away from clock, switcher, or other differential pair traces that could cause crosstalk

**Layer transitions:**
- Minimise layer transitions (vias) -- each via introduces approximately 0.1 pF capacitance and 1 nH inductance
- If a via is unavoidable, place it at a point where the reference plane is the same net (GND) on both sides; add a GND via immediately adjacent

**Routing mistakes that cause compliance failure:**

1. **Trace width varying along the route:** If the trace narrows from 0.175 mm to 0.1 mm to clear a pad, the local impedance rises from 90 Ω to approximately 110 Ω, creating a reflection. USB 3.2 eye mask allows very limited additive jitter; multiple small reflections accumulate and close the eye.

2. **Splitting the pair at a via:** If DP transitions through a via on Layer 1 to Layer 3 but DN stays on Layer 1, the two traces of the pair now reference different planes (L2 for DP on L3, and the outer air/dielectric for DN on L1). The mode impedance is mismatched, causing differential-to-common-mode conversion.

3. **Routing over a plane split:** The power plane beneath the USB traces has a split between 3.3V and 1.8V. Routing the USB pair across this gap causes the return current to take a long detour around the gap, creating an inductive impedance discontinuity that appears as a reflection in TDR analysis.

4. **Skew from asymmetric connector break-out:** The USB-C connector has short fanout traces on the PCB footprint. If these traces are asymmetric (different lengths for DP and DN), the skew accumulated before the equalisation point is already at the budget limit before the main route begins.

---

### Question I2
**What is the fibre weave effect, and how does it cause skew in differential pairs at high data rates? What mitigation strategies are available?**

**Answer:**

**The fibre weave effect:**

PCB dielectric is made of woven glass fibre bundles impregnated with epoxy resin. The glass fibre (Dk ~6.0) and epoxy resin (Dk ~3.2) have different permittivities. The woven pattern alternates between glass-rich and resin-rich regions at a spatial period typically in the range of 0.5–1.0 mm.

If one trace of a differential pair lies predominantly over glass bundles while the other lies predominantly over resin-rich regions, the two traces experience different effective Dk values:

```
Dk_glass ≈ 6.0
Dk_resin ≈ 3.2
Dk_FR4_nominal ≈ 4.1 (average)

If trace P sees Dk_eff = 4.4 (more glass) and trace N sees Dk_eff = 3.8 (more resin):

Propagation delay per mm:
  Delay_P = sqrt(4.4) / c × 1mm = 2.098 / (3e8 m/s) × 1e-3 = 6.99 ps/mm
  Delay_N = sqrt(3.8) / c × 1mm = 1.949 / (3e8 m/s) × 1e-3 = 6.50 ps/mm

Over a 50 mm trace:
  Skew = (6.99 - 6.50) × 50 = 24.5 ps
```

At PCIe Gen 5 (32 Gbps, UI = 31.25 ps), 24.5 ps is nearly one entire unit interval of skew -- a complete compliance failure.

**Mitigation strategies:**

1. **Route differential pairs at 45 degrees to the glass weave:** The standard weave runs at 0° and 90°. Routing at 45° causes each trace to cross both glass bundles and resin-rich regions alternately along its length, averaging out the Dk variation rather than having one trace stay in glass and the other in resin.

2. **Specify spread-glass (or low-skew) PCB material:** Materials such as Isola 370HR with spread glass weave (woven to have uniform glass density rather than concentrated bundles) significantly reduce the Dk variation from glass to resin. The Dk variation across the surface is typically ±0.1 instead of ±0.5.

3. **Tight differential pair coupling (small gap):** With tight coupling (small gap relative to dielectric height), the odd-mode field is predominantly between the traces, not between each trace and the reference plane. The fibre weave effect modulates the each-trace-to-reference-plane capacitance; tight coupling reduces the proportion of field energy that interacts with the weave.

4. **Use Megtron 6 or similar high-speed laminate:** Ultra-low-Df materials engineered for >10 Gbps SerDes typically have finer glass weave pitch, reducing the Dk spatial variation.

5. **Measure and characterise:** At Gen 4/5 speeds, include TDR coupons on the panel and verify intra-pair skew. If fibre weave effect is observed, rotate the board orientation relative to the weave direction or specify spread-glass as a procurement requirement.

---

### Question I3
**You are routing four PCIe Gen 4 differential pairs. Explain the difference between intra-pair matching, inter-pair matching, and the AC coupling capacitor placement requirement. What happens electrically if any of these rules are violated?**

**Answer:**

**Intra-pair matching (P to N within one lane):**

The positive and negative traces of a single differential pair must be equal in length from the transmitter output to the receiver input.

Requirement: ≤ 2.5 mil (0.064 mm) for PCIe Gen 4

Physical cause of violation: If the designer routes P and N traces around obstructions differently, one trace becomes longer. Even 2.5 mil (0.064 mm) represents about 0.4 ps of skew at FR-4 propagation velocity. PCIe Gen 4 has a Unit Interval of 62.5 ps; the budget is tight.

**Electrical consequence:** A skew between P and N converts some fraction of the differential signal energy into common-mode. At 16 Gbps, common-mode noise at that frequency radiates efficiently from the cable or connector assembly and causes EMC failures. The PCIe 4.0 specification places a maximum on transmitter intra-pair skew at 0.5 ps (transmitter end); total channel skew including PCB is tighter still.

**Inter-pair matching (lane to lane within the same direction):**

All TX lanes (or all RX lanes) within a PCIe x4 or x16 link must arrive within a timing window of each other. The PCIe receiver uses elastic buffers to absorb inter-pair skew, but these buffers have finite capacity.

Requirement: ≤ 10 mil (0.254 mm) for PCIe Gen 4 (some implementations allow more; check the FPGA/PCH spec)

Physical cause of violation: Lane 3 routes from the BGA in a straight line while Lane 0 must detour around a via cluster, making it 10 mm longer.

**Electrical consequence:** If one lane is delayed by more than the elastic buffer depth, that lane's data bits are permanently skewed relative to the other lanes. The receiver cannot de-skew beyond its buffer limit, causing loss of synchronisation on that lane and ultimately PCIe link training failure (the link trains at a lower width, reducing bandwidth).

**AC coupling capacitor placement:**

PCIe uses AC coupling capacitors (typically 100 nF, C0G, 0402) on every TX differential pair to block DC voltage differences between transmitter and receiver. The capacitor must be placed:
- On the transmitter side of the channel (close to the FPGA or processor output)
- Within 5 mm of the transmitter pad
- The two capacitors of the pair must be placed symmetrically (equal distances from the differential pair axis)

**Electrical consequence of poor capacitor placement:**

If the capacitor is placed far from the transmitter, the transmission line stub between the transmitter and the capacitor forms a resonator. At PCIe Gen 4 data rates, this stub resonance can fall within the signal band, causing a frequency notch in the insertion loss response that closes the eye. Additionally, an asymmetrically placed capacitor pair adds differential trace length (one signal travels further to its capacitor than the other), introducing intra-pair skew at the component transition.

---

## Tier 3 -- Advanced

### Question A1
**You are performing a pre-route signal integrity analysis on a 100G Ethernet (4 × 25 Gbps) layout. The differential pairs are 55 mm long on an inner stripline layer between two GND reference planes. The fabricator's stackup shows the signal layer is asymmetrically positioned: 0.08 mm from GND above, 0.12 mm from GND below. What are the SI implications of this asymmetry, and how do you model and mitigate it?**

**Answer:**

**Impedance asymmetry:**

An asymmetrically placed stripline trace sits closer to the upper GND plane than the lower. The capacitance to each plane is inversely proportional to distance: the upper plane contributes more capacitance than the lower. The total capacitance is set by the parallel combination of both planes' contributions:

```
Using the asymmetric stripline formula:
  b = H1 + H2 = 0.08 + 0.12 = 0.20 mm (total distance between planes)
  H1 = 0.08 mm (closer plane)
  H2 = 0.12 mm (further plane)

For a symmetric stripline (H1 = H2 = 0.1 mm):
  Effective h = b/2 = 0.1 mm

For the asymmetric case, the effective height (using the Wadell approximation) is:
  h_eff = 2 * H1 * H2 / (H1 + H2) = 2 * 0.08 * 0.12 / 0.20 = 0.096 mm

The impedance for the asymmetric case is slightly higher than the symmetric case
with the same total separation, because h_eff < b/2 means slightly less capacitance
to one plane.
```

The single-ended impedance shift from ideal symmetric is approximately +2–3 Ω in this example -- within the ±10% fabrication tolerance, so this alone is not a compliance failure.

**Mode splitting:**

The more significant effect is different even-mode and odd-mode impedance values for the differential pair. In a symmetric stripline with equal distance to both planes, the odd-mode current loops are symmetric about the pair's midplane. In an asymmetric structure:

- The odd-mode field (the one that carries the differential signal) has unequal image currents in the two GND planes
- The even-mode field (common-mode) is also asymmetric
- This causes a difference between Z_diff as measured from the upper plane side vs the lower plane side

For a differential pair on this asymmetric layer, the differential impedance is well-defined (because the differential field is largely between the traces, less so to the reference planes), but any asymmetric external disturbance (a nearby trace above the pair versus below) will see different susceptibility from the two planes.

**Common-mode to differential conversion:**

100G Ethernet (25GBASE-R per lane) uses PAM4 modulation, which is particularly sensitive to common-mode noise. The asymmetric stackup means:
- Common-mode suppression is slightly different from the two planes
- If common-mode noise enters from the upper plane (e.g., plane resonance from a nearby power via), the odd suppression from the lower plane is slightly different, allowing a small fraction to convert to differential mode

**Modelling and mitigation:**

1. **Field solver verification:** Use a 2D field solver (Ansys HFSS 2D Extractor, Polar Si9000, or similar) to extract the actual characteristic impedance matrix for the asymmetric geometry. Input H1 = 0.08 mm, H2 = 0.12 mm, trace dimensions, and Er values. Verify that Zdiff falls within the 100 Ω ±10% requirement for 100GBASE-SR4.

2. **Request symmetric stackup from fabricator:** Specify in the fab notes that the signal layer must be centred between the two reference planes (H1 = H2 within ±5 µm). Use a symmetric core/prepreg construction (equal material above and below the signal layer). The fabricator's CAM engineer can achieve this with careful material selection.

3. **Use spread-glass laminate:** At 25 Gbps per lane, fibre weave effect is a concern independent of the stackup asymmetry. Specifying a spread-glass material (such as Panasonic Megtron 6 or Isola Astra MT77) addresses both the Df loss budget and the Dk uniformity simultaneously.

4. **Differential pair routing at 45 degrees:** As discussed in I2, routing at 45° to the glass weave averages out the Dk variation and partially compensates for the asymmetric coupling by averaging the position of each trace relative to the weave over its length.

5. **Channel simulation:** Run a complete channel simulation (IBIS-AMI simulation with the transmitter and receiver models, channel S-parameters, and the equalisation settings of the SerDes) to confirm that the asymmetric stackup does not close the 100GBASE-R eye mask after equalisation.

---

### Question A2
**Explain the concept of via stub resonance in detail. For a 16-layer 3.2 mm thick backplane carrying PCIe Gen 5 (32 Gbps) signal vias from Layer 1 to Layer 3, calculate the stub resonant frequency, determine whether it falls within the signal band, and specify the back-drill requirement to move it out of band.**

**Answer:**

**Via stub physics:**

A through-hole via penetrates the full board thickness. When the signal only uses the via to transition from Layer 1 to Layer 3, the portion of the via barrel from Layer 3 to the bottom of the board forms an unterminated stub. This stub is a short section of transmission line terminated in an open circuit.

An open-circuit transmission line stub resonates at a quarter wavelength:

```
f_resonance = c / (4 × L_stub × sqrt(Er_effective))

Where:
  c = 3e8 m/s (speed of light)
  L_stub = length of the unused via barrel
  Er_effective = effective permittivity experienced by the via field
                 (approximately 4.0-4.5 for FR-4, use 4.2 as representative)
```

**Calculating stub length:**

For a 16-layer board, 3.2 mm thick:
- Layer spacing: 3.2 mm / 15 gaps = 0.213 mm per layer pair (approximate, varies by stackup)
- Layer 1 to Layer 3: 2 layer pairs × 0.213 mm = 0.427 mm of used via
- Stub length = 3.2 mm - 0.427 mm = 2.773 mm

**Resonant frequency of the stub:**

```
f_resonance = (3e8) / (4 × 0.002773 × sqrt(4.2))
            = (3e8) / (4 × 0.002773 × 2.049)
            = (3e8) / 0.02272
            = 13.20 GHz
```

**Is this within the PCIe Gen 5 signal band?**

PCIe Gen 5 data rate: 32 Gbps
Nyquist frequency: 16 GHz
The fundamental frequency of the data signal: 16 GHz
Significant harmonic content: up to 32 GHz and beyond

The stub resonance at 13.2 GHz is directly within the signal band. At resonance, the stub presents a very low impedance shunt (near-short circuit) at 13.2 GHz, creating a transmission null (deep insertion loss notch) that significantly degrades the channel. The PCIe Gen 5 receiver's equaliser cannot compensate for a deep notch at 13.2 GHz — this causes a hard compliance failure.

**Back-drilling requirement:**

Back-drilling removes the stub by drilling from the bottom side to a controlled depth, leaving only the used portion of the via barrel plus a small residual stub.

Target: Move the resonance above 40 GHz (well above 2× the Nyquist frequency for PCIe Gen 5).

Required stub length for 40 GHz resonance:

```
L_stub_max = c / (4 × f_target × sqrt(Er))
           = (3e8) / (4 × 40e9 × 2.049)
           = (3e8) / 3.278e11
           = 0.000915 m
           = 0.915 mm
```

The residual stub must be less than 0.915 mm.

Back-drill specification for fab notes:

```
Back-drill specification:
  Net: PCIe_TX[n]_P and PCIe_TX[n]_N (all 16 lane pairs, both directions)
  Direction: From bottom side (Layer 16)
  Target depth: Stop at Layer 4 (leaving via barrel used from Layer 1 to Layer 3)
  Maximum residual stub: 0.25 mm (limited by mechanical drill tolerance)
  Back-drill diameter: 0.1 mm larger than original via drill (to clear plated barrel)
  Verification: TDR coupon on panel with identical via geometry; measure S11 null
  frequency; confirm > 40 GHz.

  Note: Back-drill diameter must not exceed anti-pad diameter on layers that are
  not to be connected to the via.
```

**Result with back-drilling:**

With 0.25 mm residual stub (mechanical drill tolerance):

```
f_resonance = (3e8) / (4 × 0.00025 × 2.049)
            = (3e8) / 0.002049
            = 146 GHz
```

Completely out of band for PCIe Gen 5. The insertion loss improvement from back-drilling at 13 GHz is typically 3–8 dB, which is often the difference between passing and failing the PCIe Gen 5 loss budget.
