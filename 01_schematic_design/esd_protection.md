# ESD Protection

## Prerequisites
- Diode and Zener diode fundamentals (forward voltage, breakdown, reverse current)
- Basic circuit protection: clamping, crowbar, filtering
- Interface standard voltages (3.3 V, 5 V, 12 V, USB, Ethernet)
- Overvoltage and transient concepts

---

## Concept Reference

### What Is ESD and Why Does It Damage Electronics?

Electrostatic discharge (ESD) is a rapid, high-voltage current pulse generated when two objects at different electrostatic potentials come into contact. The human body can accumulate 25,000 V; walking on carpet in dry conditions easily generates 5,000-15,000 V. When this charge reaches an IC pin, the pulse is:

```
Human Body Model (HBM — IEC 61000-4-2 / ESDA HBM):
  Source voltage: 0 V to 8,000 V (test levels)
  Source impedance: 1.5 kΩ (HBM equivalent circuit)
  Duration: ~100 ns
  Peak current: V / 1.5k = 8,000 / 1,500 = 5.3 A

Charged Device Model (CDM):
  The IC itself accumulates charge and discharges through its pins to ground.
  Rise time: < 200 ps (extremely fast)
  Peak current: 1-5 A
  Most destructive to thin gate oxides and fine-pitch interconnects.
```

ESD damages ICs through:

1. **Dielectric breakdown:** Gate oxide of a MOSFET (2-10 nm thick) can be punctured by as little as 15-40 V. A single ESD event can create a permanent conductive path through the gate oxide, causing a hard failure or latent degradation.

2. **Metal melting:** The transient current density in narrow metal interconnects can locally melt aluminium or copper traces, causing an open or intermittent connection.

3. **Junction damage:** Reverse-biasing a PN junction beyond its avalanche capability causes localised heating and can destroy the junction.

4. **Latch-up:** CMOS parasitic thyristor structures (PNPN) can be triggered by ESD, causing sustained low-impedance conduction that destroys the IC unless power is removed.

---

### IEC 61000-4-2: The System-Level ESD Standard

IEC 61000-4-2 specifies ESD immunity requirements for complete equipment (not individual ICs). It defines:

```
Test type       Contact discharge    Air discharge
----------      ----------------     -------------
Level 1         ±2 kV                ±2 kV
Level 2         ±4 kV                ±4 kV
Level 3         ±6 kV                ±8 kV
Level 4         ±8 kV                ±15 kV

Consumer CE marking typically requires Level 2 (±4 kV contact, ±8 kV air).
Industrial and medical products often require Level 3 or Level 4.

Test waveform (contact discharge into 2 Ω):
  Rise time: 0.7-1 ns
  Peak current: ±4 kV / 2 Ω = 2,000 A (!) for Level 2 contact discharge
  Duration (to 50%): ~60 ns
  Duration (to 10%): ~100-200 ns
```

The extremely fast rise time (< 1 ns) means the ESD pulse has significant energy at frequencies up to 1 GHz. Effective ESD protection must clamp the voltage before the fast edge reaches the IC.

**Failure criteria for IEC 61000-4-2:**

```
Criterion A: Equipment continues to operate normally during test — full compliance
Criterion B: Temporary degradation allowed, but self-recovery without intervention
Criterion C: Temporary degradation allowed; manual reset required
Criterion D: Permanent damage — test failure
```

---

### ESD Protection Devices

#### TVS Diodes (Transient Voltage Suppressor)

TVS diodes are specifically engineered for clamping transient overvoltages. They are characterised by:

```
Parameter          Symbol    Meaning
---------          ------    -------
Standoff voltage   VRWM      Maximum continuous working voltage — no conduction
Breakdown voltage  VBR       Voltage at which the device enters avalanche (at test current IT)
Clamping voltage   VC        Voltage across device at peak pulse current IPP
Peak pulse power   PPPM      Maximum instantaneous power (at specified pulse duration)
Peak pulse current IPP       Maximum transient current the device can handle
Junction capacitance  CJ     Parasitic capacitance — limits use on high-speed signal lines
Response time         ns     Time to enter conduction after overvoltage appears
```

**Unidirectional vs bidirectional:**

```
Unidirectional TVS: One Zener + one forward diode in series (one direction clamped,
                    other direction clamps at forward voltage ~0.7 V)
                    Use for: single-polarity lines (power supply rails, single-ended signals)
                    Benefit: lower capacitance, lower clamping voltage in reverse

Bidirectional TVS: Two Zener diodes back-to-back
                   Use for: AC lines, differential signals, RS-485, USB
                   Both polarities clamped to VC
```

**TVS selection procedure:**

```
1. VRWM ≥ maximum steady-state signal voltage + margin (typically 10%)
   For 3.3 V rail: VRWM ≥ 3.6 V → use 3.3 V or 5 V standoff TVS

2. VC at IPP must be below the IC's absolute maximum rating:
   If IC ABS MAX = 6 V, and TVS VC = 5.5 V at IPP → acceptable
   If IC ABS MAX = 4 V, this TVS does not protect adequately

3. IPP ≥ peak ESD current (determine from source impedance):
   IEC 61000-4-2 Level 4 with 2 Ω source, ±8 kV: IPP > 8000/2 = 4000 A peak
   But: series resistance in protection path limits actual current into TVS.
   Realistic: IPP > 10-20 A for board-level protection behind ferrite/resistor.

4. CJ: for signal lines, CJ must be low enough not to degrade signal integrity.
   USB 2.0 (480 Mbps): CJ < 2 pF
   USB 3.0 (5 Gbps): CJ < 0.5 pF
   Ethernet 1G: CJ < 1.5 pF
   HDMI 2.0: CJ < 0.25 pF
```

#### ESD Suppressor Arrays

For multi-pin connectors (USB, HDMI, MIPI, Ethernet), rail-to-rail TVS arrays integrate multiple TVS diodes in a single package with a common bus. Examples:

```
PRTR5V0U2X (NXP): dual-line, 5.5 V VRWM, CJ = 0.35 pF — suitable for USB 2.0
TPD4S012 (TI):   4-line, 5.5 V VRWM, CJ = 0.25 pF, rail-to-rail clamp — USB 2.0
PRTR5V0U2KB (NXP): VRWM = 5 V, CJ < 0.5 pF — USB 3.0

Internal structure (rail-to-rail type):
  Each I/O line → unidirectional TVS to GND
  Each I/O line → unidirectional TVS to VCC bus (rail clamp)
  VCC bus → bidirectional TVS to GND
  
  This clamps both positive (VLINE > VC above VCC) and negative (VLINE < -0.7 V)
  transients while keeping the line-to-VCC differential within limits.
```

#### Polymer ESD Suppressors (PESD)

Polymer ESD suppressors (e.g., Littelfuse PESD series) use a conductive polymer material that has high impedance at normal voltages and becomes conductive above a threshold:

```
Advantages:
  Very low capacitance (0.05-0.3 pF) — suitable for RF, multi-GHz signals
  Can handle very high peak currents in a small package
  No hard breakdown mechanism → longer device life in repeated ESD events

Disadvantages:
  Higher trigger voltage (5-15 V) than TVS — not suitable for 3.3 V or lower supply rails
  Best used in conjunction with TVS for low-voltage protection
  Less precise clamping voltage than TVS
```

#### Schottky Diodes and Rail Clamps

Simple rail-clamp protection using Schottky diodes:

```
                VCC
                 │
              [Schottky D1]  (forward-biased when VPIN > VCC + VF)
                 │
Signal ─────────┤ PIN
                 │
              [Schottky D2]  (forward-biased when VPIN < GND - VF)
                 │
                GND

VF (Schottky) ≈ 0.2-0.4 V → clamps to VCC + 0.4 V and GND - 0.4 V
```

This structure is the basis of the on-chip ESD protection in most CMOS ICs. The problem for board-level protection is that the Schottky diodes are sized for the on-chip currents (milliamps), not for external ESD (amperes). If an external ESD event occurs, the on-chip diodes cannot handle the energy — which is why external TVS protection is needed.

---

### ESD Protection Placement and Layout

ESD protection is effective only if it intercepts the ESD pulse before it reaches the IC. This requires careful attention to placement:

```
INCORRECT layout — ESD current reaches IC before clamping:

Connector → [trace, 5 cm] → [IC pin] → [TVS, 3 cm away on a spur trace]
  The trace inductance (5 cm ≈ 5 nH) between connector and IC means that
  during the 1 ns rise time of the ESD pulse, the IC sees 5 nH × dI/dt
  voltage spike before the TVS has clamped:
  V_spike = 5n × 2A / 1n = 10 V (potentially damaging before TVS acts)

CORRECT layout — ESD current clamped at the entry point:

Connector → [TVS, as close to connector pin as possible] → [series resistor or ferrite]
                                                                    ↓
                                                             [IC pin, after filtering]

The TVS clamps at the connector. The series element limits the residual current
into the IC. Only attenuated transients reach the IC.
```

**Key layout rules:**

1. **TVS directly at connector:** TVS pads should be within 5 mm of the connector pin, not routed around to a convenient location elsewhere on the board.

2. **GND via immediately at TVS:** The return path from TVS to GND must be via a short, direct GND via adjacent to the TVS pad. A long GND trace from TVS to GND via adds inductance and degrades clamping.

3. **Series resistance before IC:** A 33-100 Ω series resistor between the TVS and the IC pin:
   - Limits the residual current into the IC if the TVS clamping is imperfect.
   - Forms a low-pass filter with the IC input capacitance to attenuate the fast edge.
   - Does not consume board area and costs essentially nothing.

4. **Keep ESD protection off analogue signal paths:** A TVS diode with 2 pF capacitance on a 1 GHz signal degrades the signal. For analogue RF, use polymer suppressors or LC filter topologies rather than shunt TVS.

---

### ESD Protection for Common Interfaces

**USB 2.0 (Full Speed / High Speed 480 Mbps):**

```
D+ and D- lines: maximum capacitance per line = 2-3 pF (USB spec)
Use: Dual-line TVS array, CJ < 1 pF per line (accounts for parasitic from trace)
VRWM: 5 V (USB lines swing 0-3.6 V nominally)
Example: PRTR5V0U2X, USBLC6-2SC6 (ST Microelectronics, CJ = 0.4 pF)

VBUS (5 V): TVS with VRWM = 5.5 V, VC < 9 V (USB spec: < 5.5 V + 3 V tolerance)
```

**USB 3.0 / 3.1 (5 Gbps / 10 Gbps):**

```
SuperSpeed pairs (SSTX+/-, SSRX+/-): maximum capacitance < 0.5 pF per line
Use: PESD suppressor or silicon-based TVS with CJ < 0.25 pF
Example: ESDA14V2BP6 (ST), TPD4S014 (TI), PRTR5V0U2KB (NXP)
```

**HDMI / DisplayPort (multi-Gbps):**

```
Data lanes (TMDS, HBR): maximum capacitance < 0.5 pF
Use: ultra-low capacitance TVS array with differential protection
Example: ESDA25B1U5B (ST, 0.1 pF per channel)
HotPlug detect (+5 V): needs separate TVS with VRWM = 5 V
```

**RS-232 (±12 V swing):**

```
RS-232 signal levels reach ±15 V during data transmission.
TVS must have VRWM ≥ 15 V to avoid false clamping.
Use: ±15 V bidirectional TVS (SP3220E has integrated protection)
Or: SMAJ15A (15 V unidirectional per line)
IEC 61000-4-2 Level 4 testing is common for industrial RS-232.
```

**RS-485 (differential, ±7 V common mode):**

```
Bus voltage can swing ±7 V common mode. TVS must not clamp during normal operation.
Use: ±12 V bidirectional TVS on each bus line (e.g., SMAJ12CA or SP3485 with built-in)
Termination resistor (120 Ω) provides series impedance to limit ESD current.
```

**Ethernet (PoE capable, up to 57 V):**

```
RJ-45 jack → CMC (common mode choke) → transformer → PHY chip
The transformer provides galvanic isolation (handles 1.5 kV ESD in most cases).
PoE lines (PSE): TVS rated for 75-100 V between power pairs.
Signal lines: TVS array after transformer if required (many PHY chips are robust).
```

---

## Tier 1 — Fundamentals

### Question F1
**What is the purpose of a TVS diode and how does it differ from a Zener diode used as a clamp?**

**Answer:**

Both a TVS and a Zener clamp overvoltage by entering avalanche/breakdown and conducting, but they are engineered for fundamentally different purposes:

**Zener diode:** Designed for precise voltage reference at a continuous, stable current. Key characteristics:
- Tight breakdown voltage tolerance (±1-5%)
- Designed for continuous or near-continuous operation
- Low power dissipation (typically < 1 W)
- Small junction area → high junction resistance under transient

**TVS diode:** Designed for high-energy transient absorption. Key characteristics:
- Large junction area for high transient current handling (amperes, not milliamps)
- Very low dynamic impedance during conduction → clamping voltage close to VBR
- Very fast response time (picoseconds) — required for ESD and lightning
- High peak power dissipation (hundreds of watts for microseconds)
- Less precise breakdown voltage (±5-10% typical), but that is acceptable because the goal is protection, not reference accuracy

```
Comparison at peak ESD current:

Zener 3.6 V, 500 mW:
  IPP capability: ~100 mA continuous → the Zener is destroyed by ESD
  Dynamic impedance: ~5-20 Ω → at 1 A, clamping voltage = 3.6 + (1 × 10) = 13.6 V
  This does not protect a 5 V-rated IC!

TVS SMBJ3V3A (3.3 V unidirectional, 600 W):
  IPP capability: 600 W / 5.5 V clamp ≈ 109 A for 1 ms (10/1000 µs waveform)
  For ESD (100 ns pulse): much higher peak current allowed
  Clamping voltage at 10 A: ~5.5 V (from datasheet) — IC protected
```

**When to use a Zener for protection:** A Zener can be acceptable as an overvoltage clamp on a slow, current-limited signal (e.g., slow ADC input with a 10 kΩ series resistor) where the Zener handles only milliamps and the series resistor limits current. For any direct ESD exposure (I/O pins, connectors), a TVS rated for the expected peak current is required.

---

### Question F2
**Why is the TVS diode's capacitance a concern on high-speed signal lines, and how would you protect a USB 2.0 D+ pin from ESD?**

**Answer:**

**Why capacitance matters:**

A shunt TVS diode appears as a capacitor in parallel with the signal line at normal operating voltages. This capacitance, combined with the source impedance of the signal driver and the trace inductance, forms a low-pass filter that attenuates high-frequency signal content:

```
Signal bandwidth degradation (-3 dB frequency):
  f_3dB ≈ 1 / (2π × Z_source × C_TVS)

For USB 2.0 (480 Mbps, requires bandwidth > 240 MHz):
  Z_source ≈ 45 Ω (USB line impedance)
  Allowable C_TVS for < 3 dB loss at 240 MHz:
    C_TVS < 1 / (2π × 240 MHz × 45) = 14.7 pF

USB 2.0 spec allows 2 pF total (trace + TVS).
  Required C_TVS < 2 pF - 0.5 pF (trace) = 1.5 pF
```

A standard TVS (e.g., P6KE series) has CJ = 100-500 pF — completely unsuitable for USB 2.0.

**Correct approach for USB 2.0 D+:**

```
Connector
  │
  ├── VBUS (5 V): [TVS VRWM=5.5 V, standard capacitance acceptable]
  │
  ├── D+ : [Series ferrite bead, ~600 Ω @ 100 MHz, ~1 nH at signal frequency]
  │          │
  │          ├── [TVS array element, CJ < 0.5 pF, VRWM = 5 V]
  │          │              (e.g., one channel of PRTR5V0U2X)
  │          │              TVS GND pin → GND via < 3 mm away
  │          │
  │          └── D+ to USB controller IC
  │
  └── D- : [Same topology as D+, matched component to maintain differential balance]

Note: D+ and D- TVS must be matched capacitances to avoid differential-to-common-mode conversion.
Unmatched CJ creates differential skew → signal eye degradation.
```

The ferrite bead serves two purposes: limits the peak ESD current through the TVS and adds series impedance to further attenuate the ESD pulse residual before it reaches the USB controller.

---

### Question F3
**What is the difference between contact discharge and air discharge in IEC 61000-4-2, and which is typically more severe?**

**Answer:**

**Contact discharge:** The ESD gun tip is placed in direct electrical contact with the item under test before the discharge is triggered. The energy is delivered through the well-defined gun circuit (which has a 330 Ω / 150 pF discharge network, simulating a human body model). The discharge is deterministic and repeatable.

**Air discharge:** The gun tip is brought toward the item from a distance, and the discharge occurs when the air gap breaks down. The breakdown voltage depends on humidity, air pressure, geometry, and surface condition — all variable. The rise time is typically faster than contact discharge because the arc has lower impedance than the 330 Ω gun resistance in contact discharge.

```
Typical comparison at same test level:

Contact discharge (±4 kV Level 2):
  Peak current: ~2 A (through 330 Ω gun resistance)
  Rise time: ~0.7-1 ns
  Reproducible and well-defined

Air discharge (±8 kV Level 2 — note: air discharge levels are higher):
  Peak current: variable, can reach 30-40 A in arc
  Rise time: < 500 ps (can be very fast with narrow arc)
  Less reproducible — may vary ±50% between shots
```

**Which is more severe:** Air discharge can produce faster rise times and higher peak currents for the same test level, potentially more damaging to fast logic. However, because IEC 61000-4-2 specifies higher voltages for air discharge (±8 kV air vs ±4 kV contact for Level 2), the delivered energy is comparable. Contact discharge is used on conductive surfaces (connector pins, metal housings); air discharge is used on non-conductive surfaces (plastic enclosures, gaps). Both must be passed for CE marking.

---

## Tier 2 — Intermediate

### Question I1
**Design ESD protection for a 3.3 V RS-232 interface (±15 V signal swing) entering a PCB from an external connector. The system must pass IEC 61000-4-2 Level 3 (±6 kV contact). Specify the TVS selection and placement.**

**Answer:**

**Step 1 — Characterise the signal environment:**

RS-232 signal lines legitimately swing to ±15 V (MARK = -3 to -15 V, SPACE = +3 to +15 V). The TVS must not clamp during normal signal operation:

```
VRWM ≥ 15 V (to avoid TVS conduction on peak RS-232 swings)
→ Use bidirectional TVS, VRWM = 15-18 V
```

**Step 2 — Peak current requirement for IEC 61000-4-2 Level 3:**

```
Level 3 contact discharge: ±6 kV into 330 Ω gun impedance + PCB impedance.
Assume PCB series resistance before TVS: ~10 Ω (trace + any series resistor).
Peak current = 6,000 / (330 + 10) = 17.6 A

TVS must handle IPP > 20 A (with margin).
```

**Step 3 — Maximum allowed clamping voltage (VC):**

RS-232 transceiver IC (e.g., SP3232) has absolute maximum on RS-232 pin of typically ±25 V.

```
Required: VC at IPP < 25 V (with some margin — use < 22 V)
```

**Step 4 — Component selection:**

```
SMBJ15CA (bidirectional TVS, SMB package):
  VRWM = 15 V, VBR = 16.7-20 V, VC = 24.4 V at IPP = 24.6 A (600 W)
  CJ ≈ 80 pF — acceptable for RS-232 (baud rates up to 115,200 bps = ~115 kHz)
  IPP = 24.6 A — exceeds 20 A requirement ✓
  VC = 24.4 V < 25 V absolute max ✓  (just marginal — verify with actual IC spec)

Alternative: SMAJ15CA (SMA package, 400 W, IPP = 16 A) — marginal on current.
Better: P6SMB15CA (600 W) or SMCJ15CA (1500 W, DO-214AB) for more margin.

For 4-line RS-232 (TxD, RxD, RTS, CTS):
  1× quad TVS array or 4× individual TVS
  e.g., NUP4114 (ON Semi): quad bidirectional, 12 V VRWM (check against RS-232 levels)
  Or: SP3200 series (Sipex/MaxLinear): RS-232 transceiver with integrated TVS
```

**Step 5 — Placement and schematic:**

```
DB9 Connector
  │
  Pin 2 (RxD)  ──────[R 33Ω]──── ──────────────────── RxD_IC
  Pin 3 (TxD)  ──────[R 33Ω]──── ──────────────────── TxD_IC
  Pin 7 (RTS)  ──────[R 33Ω]──── ──────────────────── RTS_IC
  Pin 8 (CTS)  ──────[R 33Ω]──── ──────────────────── CTS_IC
  Pin 5 (GND)  ──────────────────────────────────────── GND
  │                     │
  │                     ├── [SMBJ15CA bidirectional TVS] → GND  (one per line)
  │                     │    placed within 5 mm of connector pin
  │
  └──[Chassis GND strap (if applicable)]

The 33 Ω series resistors:
  - Limit peak TVS current: I_peak = 6000 / (330 + 33) ≈ 16.5 A
  - Limit transient current into IC if TVS slightly misses clamping
  - Form RC filter with IC input capacitance (~10 pF): f_3dB = 1/(2π×33×10p) = 480 MHz
    (well above 115,200 bps requirement)
  - RS-232 signal integrity not affected at normal baud rates

TVS layout:
  TVS pad directly connected to connector pad via short wide trace (< 5 mm, ≥ 0.5 mm wide)
  GND via immediately adjacent to TVS GND pad (< 2 mm)
  GND via connects to ground plane — not to trace running across board
  Series resistor on IC side of TVS (after TVS, before IC pin)
```

---

### Question I2
**A system passes IEC 61000-4-2 Level 2 testing with SMAJ5.0A TVS diodes on all ports. The customer then requests Level 4 (±8 kV contact) compliance. Describe what changes may be required and why simply swapping to a higher-wattage TVS of the same voltage may not be sufficient.**

**Answer:**

**Why a higher-wattage TVS alone may not be sufficient:**

Upgrading from SMAJ (400 W) to SMCJ (1500 W) or SM6T (600 W) increases the peak current capability (IPP), which is necessary. However, the system-level ESD protection problem has three separate dimensions:

**1. Clamping voltage (VC) at higher peak current:**

Higher-wattage TVS devices have lower dynamic impedance (Rdyn), so their clamping voltage at higher IPP rises proportionally less. But the relationship is:

```
VC = VBR + Rdyn × IPP

SMAJ5.0A: Rdyn ≈ 0.1 Ω, VBR = 6.4 V, VC at 30 A = 6.4 + 0.1×30 = 9.4 V
SMCJ5.0A: Rdyn ≈ 0.05 Ω, VBR = 6.4 V, VC at 100 A = 6.4 + 0.05×100 = 11.4 V

The IC's absolute maximum input voltage (ABS MAX) may be 6 V or 7 V.
If VC = 9.4 V exceeds ABS MAX → the IC is damaged even with TVS present.
```

At Level 4, the peak current is 2× higher than Level 2. The higher VC may exceed IC ABS MAX even when the TVS survives.

**2. Series impedance in the ESD path:**

If the Level 4 pulse (8 kV / 330 Ω = 24 A peak) causes the clamping voltage to exceed ABS MAX, the fix is to increase the series impedance before the IC (not just the TVS rating):

```
With 100 Ω series resistor between TVS and IC:
  IEC 61000-4-2 gun: 330 Ω source
  Series R: 100 Ω
  Total: 430 Ω
  Peak current through TVS: 8000 / 430 = 18.6 A (reduced)
  Residual voltage at IC: VC - (100 Ω × I_residual_into_IC_cap)
  IC input capacitance: 5 pF → current into IC ≈ C × dV/dt ≈ tiny

The series resistor substantially limits IC exposure. This is often required
for Level 3-4 compliance where Level 1-2 only needed the TVS.
```

**3. Board-level current path (layout):**

At higher ESD levels, the induced ground bounce through the PCB can couple energy into supposedly protected paths. At Level 4, the ground plane potential near the ESD injection point may rise by several volts relative to the IC's local ground reference, effectively injecting a voltage stress from the ground side. This requires:

- Solid, low-inductance ground plane directly under the protection devices.
- Chassis ground connection via spark gap or TVS at board entry point for systems in metal enclosures.
- Ground bonding between PCB ground and chassis at multiple points.

**Summary of Level 2 → Level 4 upgrade steps:**

```
1. Verify IPP capability: upgrade TVS package (SMAJ → SMCJ or DO-214AB)
2. Verify VC vs IC ABS MAX at Level 4 current — add series resistance if needed
3. Check PCB layout: TVS GND via must handle Level 4 current without trace voltage drop
4. Add or increase series resistance on all protected lines
5. Verify chassis/enclosure bonding — significant at Level 4 air discharge
6. Re-test: ESD can have non-linear, path-dependent failure modes
```

---

## Tier 3 — Advanced

### Question A1
**A mixed-signal PCB has a 16-bit ADC with a 1.8 V reference input (ABS MAX: 2.5 V) and a ±5 V analogue input range. The analogue input arrives from a detachable sensor cable (up to 3 m long, therefore susceptible to ESD and cable transients). The ADC input network has a 10 kΩ input resistor, a 47 nF anti-aliasing filter capacitor, and an op-amp buffer. Design a complete ESD protection strategy that does not compromise the 16-bit noise performance.**

**Answer:**

**Step 1 — Constraints analysis:**

```
Signal path requirements:
  ADC input range: ±5 V → input protection must allow ±5 V
  ADC resolution: 16 bits on 10 V range → 1 LSB = 10 V / 65536 = 153 µV
  Noise budget: < 1 LSB RMS preferred → < 153 µV noise floor
  Anti-aliasing filter: 10 kΩ + 47 nF → f_3dB = 1/(2π × 10k × 47n) = 339 Hz

ESD protection constraints:
  Cannot introduce > 1 µV RMS noise (which ESD TVS leakage current through 10 kΩ can cause)
  Cannot add capacitance that materially changes the 339 Hz filter pole
    (47 nF total → max 0.5% change → max 235 pF additional)
  TVS capacitance × 10 kΩ must not create a new pole below 339 Hz
    → C_TVS < 1/(2π × 10k × 339) = 47 µF (not the binding constraint)
  TVS leakage current through 10 kΩ must not create DC offset > 1 mV
    → I_leakage < 1 mV / 10 kΩ = 100 nA (very demanding)
```

**Step 2 — Primary protection at cable entry (handles the bulk of ESD energy):**

The op-amp buffer isolates the ADC from the input network — ESD must be arrested before it reaches the buffer input. Two stages:

```
Cable entry → primary clamp → 10 kΩ resistor → secondary clamp → op-amp

Primary clamp: bilateral transient suppressor, ±6.5 V clamp (allows ±5 V signal,
  clamps anything beyond ±6 V)

Suitable device: Bourns CDSOD323-T05LC (bilateral, 5 V VRWM, VC = 9.2 V at 5 A)
  CJ = 200 pF → with 10 kΩ input resistor: RC pole = 1/(2π × 10k × 200p) = 79.6 kHz
  (far above 339 Hz anti-aliasing pole — does not affect ADC bandwidth)
  Leakage at 5 V signal: < 1 µA typical at 25°C → 1 µA × 10 kΩ = 10 mV offset (too high!)

Leakage is a problem: need a device with very low leakage at ±5 V working voltage.

Better choice: two back-to-back Schottky diodes (BAT54 or similar) to +5 V and -5 V rails:
  Leakage: 10-100 nA → 100 nA × 10 kΩ = 1 mV maximum (borderline)
  Or: use precision, low-leakage clamp such as ADI ADP7142 input protection network
  Or: PRTR5V0U2X (NXP) used as bilateral clamp: leakage < 1 µA at 5 V

Practical solution for ±5 V with < 100 nA leakage:
  Two BAV99 dual Schottky + split ±5.5 V TVS rails:
  D1 cathode → +5.5 V TVS → GND (clamps positive transient to +5.5 V)
  D2 anode → GND → -5.5 V TVS → anode (clamps negative transient to -5.5 V)
  Signal flows through without forward-biasing any diode within ±5 V range.
```

**Step 3 — Secondary protection at op-amp input (residual energy after 10 kΩ):**

The 10 kΩ resistor drops most of the transient voltage, limiting current to:

```
I_secondary = V_clamp_primary / 10 kΩ = 9.2 V / 10 kΩ = 0.92 mA peak
(after primary TVS, the voltage at the op-amp side is limited)

Op-amp ABS MAX: typically ±VS + 1 V for precision input-clamp op-amps
  e.g., OPA2228: input ±VS (must not exceed supply rails)

Secondary protection: pair of BAV99 Schottky diodes to ±5 V supply rails
  Forward voltage clamp to ±5.7 V → below op-amp ABS MAX of ±6 V
  Leakage: < 50 nA at room temperature — negligible

Capacitance of BAV99: ~3 pF per diode → pair in parallel: ~6 pF
  Additional RC pole: 1/(2π × 10k × 6p) = 2.65 MHz — no effect on 339 Hz system
```

**Step 4 — Complete protection schematic:**

```
Cable
connector
  │
  ├── [PE/chassis GND via spark gap or 1812 safety cap 4.7 nF 250 V Y-cap]
  │
Signal ──[FB1 ferrite 600Ω@100MHz]─────────────────────────────────────
                                  │
                             [TVS clamp ±6 V, primary, low leakage, CJ<300pF]
                                  │
                                 GND
                                  ↓
                          ─[R_in 10 kΩ 1%]─────────────────────────────────
                                              │
                                         [BAV99 to ±5V rails, secondary]
                                              │
                                             GND
                                              ↓
                                    ─[C_AAF 47 nF C0G]─ GND
                                              │
                                         [Op-amp buffer input]
                                              │
                                    ─[R_ADC 100 Ω]─[ADC input]
```

**Step 5 — Noise performance verification:**

```
Thermal noise of 10 kΩ at 25°C in 339 Hz bandwidth:
  Vn = √(4kTRB) = √(4 × 1.38×10⁻²³ × 298 × 10000 × 339) = 0.24 µV RMS

This is below 1 LSB (153 µV) → the resistor noise is acceptable.
ESD clamp leakage: < 100 nA at 5 V maximum → DC offset < 1 mV

Both constraints met. The primary ESD protection handles the bulk of ESD energy
(> 99% of the ESD voltage dropped across the clamp). The 10 kΩ resistor is the
current-limiting element. The secondary Schottky clamp protects the op-amp input
from the residual voltage under high ESD currents.

For IEC 61000-4-2 Level 3 (±6 kV contact through 330 Ω + 10 kΩ):
  Peak current = 6000 / (330 + 10000 + source) ≈ 0.58 A into primary TVS
  Primary TVS rated for > 5 A (easily handled)
  After primary clamp, voltage at op-amp side ≈ primary VC clamped to ~6-7 V
  Secondary Schottky clamps op-amp input to ±5.7 V — within ABS MAX
```

---

### Question A2
**Explain the concept of a "guard ring" or "isolation moat" for ESD protection on a PCB. In what scenarios is this technique used, and how does it interact with the grounding strategy?**

**Answer:**

A guard ring (isolation moat in some contexts) is a continuous ring of copper connected to a defined potential — typically ground or a local shield potential — that surrounds a sensitive circuit area on the PCB. Its purpose and implementation differ depending on the application:

**Context 1 — High-voltage isolation (creepage and clearance):**

For circuits with high-voltage interfaces, a guard ring provides a controlled equipotential surface between the high-voltage and low-voltage regions:

```
HV region          Guard ring (tied to GND or floating)          LV region
   ○──────────────[moat cut in PCB copper]────────────────────────○
   │                                                               │
   └── no copper pour between HV net and guard ring               └── sensitive IC

The guard ring intercepts surface leakage currents before they reach the sensitive IC.
It reduces the effective leakage resistance between HV and LV.
Often combined with a slot or cut in the PCB substrate to increase creepage distance.
```

**Context 2 — Analogue guard ring (op-amp applications):**

For high-impedance analogue inputs (femtoamp current measurement, pH probe, pH electrode amplifiers), a guard ring is driven at the same potential as the signal to eliminate leakage from adjacent nets:

```
Signal pin (10 GΩ source impedance)
   │
   [Op-amp inverting input]
   │
   Surrounding PCB pads and traces ── Guard ring driven at same voltage as signal
                                     (from op-amp output or unity-gain buffer)

Without guard ring: a 10 GΩ source has 10 mV error for every 1 pA of leakage
  (V = I × R = 1 pA × 10 GΩ = 10 mV — significant!)
With guard ring at same potential as signal: no voltage difference → no leakage current
```

**Context 3 — ESD isolation moat:**

At the PCB level, when an external connector is the ESD injection point, a ground moat separates the "dirty" ESD-exposed region from the "clean" internal circuitry:

```
                ┌─────────────────────────────────────────────────────┐
                │  BOARD EDGE ZONE                                    │
                │  Connectors, ESD clamps, EMI filters                │
                │  Connected to chassis/shield ground (dirty GND)     │
                │                                                     │
                │  ─────────── ISOLATION MOAT ───────────────────     │
                │  (PCB copper removed, or slot cut in board)         │
                │  ─────────────────────────────────────────────      │
                │                                                     │
                │  INTERNAL ZONE                                      │
                │  Processor, DDR, analogue circuits                  │
                │  Connected to clean digital GND                     │
                └─────────────────────────────────────────────────────┘

Signal crosses the moat via:
  - Protected traces (after TVS + filter, only attenuated signals cross)
  - The two GND domains are connected at a single point (or through a ferrite bead)
    to prevent ground loops while keeping the dirty and clean grounds separated.
```

**Interaction with grounding strategy:**

The critical design choice is how the dirty GND (chassis, connector shield, ESD return) and clean GND (signal ground, ADC reference ground) are connected:

```
Single-point ground connection:
  Dirty GND and clean GND connected at one point only (a "star point" or through ferrite).
  ESD current returns to chassis via the dirty GND without flowing through the clean GND plane.
  Risk: if the star point has any inductance, ESD ground bounce appears at the star point
  and couples into the clean zone.

Multi-point ground with ferrite:
  Multiple connections between dirty and clean GND, each through a ferrite bead (600 Ω @ 100 MHz).
  DC and low-frequency current can flow across zones (acceptable for return currents).
  ESD (high-frequency) is blocked by the ferrite, forcing it to return via the dirty GND path.
  This is the approach recommended in IEC 61000-4-2 application notes for passing Level 3-4.

PCB slot (physical moat):
  A slot cut in the PCB substrate prevents surface-layer leakage across the moat boundary.
  Used for high-voltage isolation (safety standards IEC 62368-1, UL 60950).
  Not typically required for ESD at < 8 kV unless the board operates in high-humidity
  environments where surface tracking is a concern.
```

**Practical warning:** A guard ring or moat that is not connected to a well-defined potential (floating moat) can accumulate charge and act as an ESD aggressor rather than a protector. Every guard ring must be explicitly tied to either chassis ground, local GND, or a driven potential.
