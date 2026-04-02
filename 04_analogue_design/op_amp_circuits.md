# Op-Amp Circuits

## Prerequisites
- Basic circuit analysis: KVL, KCL, voltage divider, superposition
- Ideal op-amp assumptions: infinite gain, infinite input impedance, zero output impedance, zero offset
- Frequency response: Bode plots, gain-bandwidth product
- Feedback theory: negative feedback, loop gain, stability

---

## Concept Reference

### The Ideal Op-Amp Model

An ideal op-amp has two rules that apply when negative feedback is present:

```
Rule 1: V+ = V–  (virtual short — the differential input voltage is zero)
Rule 2: I+ = I– = 0  (no current flows into either input terminal)

These two rules are sufficient to analyse all standard op-amp topologies.
```

In practice, departures from these ideal rules set the limits of real op-amp performance:

```
Departure from Rule 1:
  Input offset voltage (Vos): the DC differential voltage that makes the output zero.
  Typical: 0.1–10 mV for general-purpose, <10 µV for precision devices.
  Effect: adds a DC error term to every configuration.

Departure from Rule 2:
  Input bias current (Ib): the small current that must flow into each input.
  Typical: 1 pA (JFET/CMOS) to 100 nA (bipolar input).
  Effect: creates voltage drops across source impedances, causing offset.
  Mitigation: match source impedances at both input terminals.

Input noise:
  Voltage noise spectral density: en (nV/√Hz)
  Current noise spectral density: in (pA/√Hz)
  Noise corner frequency (1/f noise): f_c (kHz range for bipolar, Hz range for CMOS)
```

### Inverting Amplifier

```
Circuit:
              Rf
    +---------/\/\---------+
    |                      |
    |   Ri                 |
Vin --/\/\-- V– –[Op-Amp]–+-- Vout
                 |
                V+
                |
               GND

Gain: Vout/Vin = -Rf/Ri

Input impedance: Rin = Ri
Output impedance: ~0 Ω (ideal)

Design rules:
  Rf and Ri set gain and input impedance.
  To minimise bias current offset: add Rcomp = Rf || Ri at V+ input.
  Total noise gain = 1 + Rf/Ri (noise referred to input is amplified by noise gain).
```

**When to use:** When the source can drive a defined load impedance (Ri), when phase inversion is acceptable, or when a summing amplifier topology is needed.

### Non-Inverting Amplifier

```
Circuit:
                       Rf
          +------+----/\/\--+-- Vout
          |      |          |
Vin --+--V+  V– –+---[Ri]--+
      |
     GND

Gain: Vout/Vin = 1 + Rf/Ri

Input impedance: very high (approaches open-circuit)
Output impedance: ~0 Ω (ideal)

Noise gain = signal gain = 1 + Rf/Ri
```

**When to use:** When source impedance is high and must not be loaded, or when a high input impedance buffer is needed.

### Voltage Follower (Unity Gain Buffer)

```
Vout/Vin = 1 (Rf = 0, Ri = open, or simply short from output to V–)
Input impedance: very high
Output impedance: ~0 Ω (ideal)
Noise gain = 1 (lowest noise gain of any topology)
```

Used to isolate a high-impedance source from a low-impedance load. Critical in ADC input buffering, sensor signal conditioning, and reference voltage distribution.

### Summing Amplifier (Inverting)

```
         R1             Rf
V1 --/\/\--+---/\/\---- Vout
           |
         R2 +
V2 --/\/\--+
           |
         R3 +
V3 --/\/\--+-- V–

Vout = -Rf × (V1/R1 + V2/R2 + V3/R3)

If R1 = R2 = R3 = Rin:
  Vout = -(Rf/Rin) × (V1 + V2 + V3)
```

Used in DAC circuits (R-2R ladders drive a summing amplifier), audio mixing, and signal combining.

### Difference Amplifier

```
         R1       R3
Vin+ --/\/\--+--/\/\-- Vout
             |
            V+
             |
         R2      R4
Vin– --/\/\--+--/\/\-- V–
                       |
                      GND

With matched resistors R1=R2=R, R3=R4=Rf:
  Vout = (Rf/R) × (Vin+ – Vin–)
  CMRR = 20 × log(Rf/R × (1+Rf/R+R) / ΔR_mismatch)
```

CMRR (Common Mode Rejection Ratio) is very sensitive to resistor matching. For a 0.1% resistor mismatch, CMRR is limited to approximately 66 dB — far below what a three-op-amp instrumentation amplifier achieves.

### Integrator and Differentiator

**Inverting integrator:**

```
Replace Rf with capacitor C: Vout = -(1/RC) × ∫Vin dt
Low-pass characteristic: -20 dB/dec roll-off, 90° phase shift
Instability risk: DC offset at input integrates without bound (adds R in parallel with C to stabilise)
```

**Differentiator:**

```
Replace Ri with capacitor C: Vout = -RC × dVin/dt
High-pass characteristic: amplifies high-frequency noise
Real differentiators require a series resistor to limit gain at high frequencies
```

### Op-Amp Stability and Phase Margin

An op-amp in feedback is a closed-loop system. Stability requires that at the frequency where loop gain = 0 dB (gain crossover frequency), the phase shift of the loop is less than 180°. The phase margin is:

```
Phase margin = 180° – |loop phase at gain crossover|
Target: ≥ 45° for stability (≥ 60° for low peaking and good transient response)
```

**Unity-gain stability:** Many op-amps are internally compensated for unity-gain stability (noise gain = 1). This means they are stable in any configuration with noise gain ≥ 1 and standard capacitive loads.

**Capacitive load instability:** A capacitive load at the output creates an additional pole in the loop. This can reduce phase margin to zero, causing oscillation. Mitigation: add a series "snubber" resistor (10–100 Ω) between the op-amp output and the capacitive load. This isolates the capacitive pole from the feedback path.

```
Rout_snubber + CL creates a pole at: f_p = 1 / (2π × Rout × CL)

For 100 Ω series resistance and 1 nF load: f_p = 1.59 MHz
This places the additional pole well above the gain crossover frequency of
most general-purpose op-amps — stability is restored.
```

---

## Tier 1 — Fundamentals

### Question F1
**Design an inverting amplifier with a gain of -10, input impedance of 10 kΩ, and noise gain that does not exceed 11. Select resistor values and add the bias current compensation resistor. What is Rcomp?**

**Answer:**

From gain and input impedance requirements:

```
Ri = 10 kΩ  (sets input impedance)
Gain = -Rf/Ri = -10 → Rf = 10 × 10 kΩ = 100 kΩ

Noise gain check: 1 + Rf/Ri = 1 + 10 = 11 ✓ (meets requirement)
```

Bias current compensation resistor at the non-inverting input (connected to GND):

```
Rcomp = Rf || Ri = 100 kΩ || 10 kΩ = (100 × 10) / (100 + 10) = 9.09 kΩ

Use nearest E96 preferred value: 9.09 kΩ → select 9.09 kΩ or 9.1 kΩ
```

**Why Rcomp is necessary:** If both input terminals see the same source impedance, the bias current (which is the same for both inputs in bipolar op-amps) creates equal voltage drops, and the differential offset cancels to first order. Without Rcomp, the non-inverting input is at GND (zero impedance) while the inverting input sees Rf || Ri ≈ 9 kΩ. The resulting input offset is:

```
ΔVos = Ib × (Rf || Ri) = Ib × 9 kΩ

For Ib = 100 nA (bipolar op-amp): ΔVos = 100 nA × 9 kΩ = 0.9 mV
This appears at the output as: ΔVout = ΔVos × 11 = 9.9 mV
With Rcomp, this error reduces by ≈ 50× (limited by Ib matching, not Ib magnitude)
```

---

### Question F2
**An op-amp has a gain-bandwidth product (GBW) of 10 MHz. It is configured as a non-inverting amplifier with gain = 100. At what frequency does the amplifier's closed-loop gain drop to 3 dB below its DC gain? What is the gain at 1 MHz?**

**Answer:**

For a single-pole op-amp, the closed-loop bandwidth is:

```
BW_closed = GBW / Closed-loop gain
          = 10 MHz / 100
          = 100 kHz

The -3 dB frequency is 100 kHz.
```

At 1 MHz (which is 10× above the -3 dB frequency):

```
For a first-order roll-off (-20 dB/decade above -3 dB frequency):
  Gain at 1 MHz = DC gain / sqrt(1 + (f/f_3dB)²)
               = 100 / sqrt(1 + (1 MHz/100 kHz)²)
               = 100 / sqrt(1 + 100)
               = 100 / 10.05
               = 9.95 ≈ 10 (i.e., ~20 dB below the 100× DC gain)
```

In dB: gain falls from 40 dB at DC to 20 dB at 1 MHz — a 20 dB reduction per decade above the -3 dB frequency.

**Practical implication:** If the circuit must maintain gain ≥ 90 (within 1 dB of DC gain), it can only be used up to approximately 30 kHz with this op-amp at a gain of 100. To extend bandwidth, either reduce gain (increase feedback) or use an op-amp with a higher GBW product.

---

### Question F3
**What is the common-mode rejection ratio (CMRR) of an op-amp, and why does it matter in a difference amplifier application?**

**Answer:**

**CMRR** is the ratio of the op-amp's differential gain to its common-mode gain:

```
CMRR = 20 × log(A_diff / A_cm)   [in dB]

Typical values:
  General purpose: 80-100 dB
  Precision: 100-140 dB

Example: CMRR = 100 dB means A_cm = A_diff / 10^5
  For A_diff = 1000: A_cm = 0.01
  A 1 V common-mode signal appears as only 10 mV at the output.
```

**Why it matters in a difference amplifier:**

A difference amplifier is intended to amplify only the differential signal (Vin+ – Vin–) and reject any signal common to both inputs (common-mode voltage). In real circuits, common-mode signals arise from:

- Ground potential differences between the signal source and the amplifier
- Power supply noise coupled equally into both signal lines
- Electromagnetic interference picked up by the cable connecting the sensor to the amplifier

If the CMRR is finite, the amplifier adds a portion of the common-mode signal to the output as a differential error. For example, a 50 Hz mains-frequency pickup of 100 mV common-mode voltage, with CMRR = 80 dB:

```
Error at output = 100 mV / 10^(80/20) = 100 mV / 10,000 = 10 µV (referred to input)
                  Appears at output as 10 µV × gain

With differential gain = 100: output error = 1 mV — significant for a sensor
with a 10 mV full-scale output range.
```

In precision sensor applications (strain gauges, thermocouples, load cells), CMRR directly sets the floor for measurement accuracy in the presence of common-mode noise.

---

## Tier 2 — Intermediate

### Question I1
**Design a Sallen-Key second-order low-pass filter with a Butterworth response, cutoff frequency 10 kHz, using equal component values (R1 = R2 = R, C1 = C2 = C). Select R and C for C = 10 nF. Calculate the Q factor and confirm it matches the Butterworth requirement.**

**Answer:**

The Sallen-Key unity-gain low-pass filter:

```
         R1        R2
Vin --/\/\---+---/\/\--- V+ ---[buffer]--- Vout
             |           |
             C2          C1
             |           |
            GND         GND
                          \
                           +-- (feedback: Vout to V–, unity gain)
```

For equal component values (R1 = R2 = R, C1 = C2 = C):

```
Cutoff frequency:
  f0 = 1 / (2π × R × C)

  With C = 10 nF, f0 = 10 kHz:
  R = 1 / (2π × 10 kHz × 10 nF)
    = 1 / (2π × 10^4 × 10^-8)
    = 1 / 6.283e-4
    = 1591 Ω → use 1.6 kΩ (E96: 1.60 kΩ)

Quality factor for equal-component unity-gain Sallen-Key:
  Q = 0.5 (for equal R and C, unity gain)

Butterworth requirement:
  For a 2nd order Butterworth: Q = 1/√2 = 0.707
```

With equal components and unity gain, Q = 0.5, which corresponds to a **Bessel response** (maximally flat group delay) — not Butterworth.

**Correction to achieve Butterworth Q = 0.707:**

Provide a gain in the buffer:

```
Sallen-Key non-inverting gain K:
  Q = 1 / (3 - K)   (for equal R and C)
  0.707 = 1 / (3 - K)
  3 - K = 1.414
  K = 1.586

Implement with a non-inverting amplifier: K = 1 + R_b/R_a = 1.586
  R_b/R_a = 0.586 → e.g., R_a = 10 kΩ, R_b = 5.86 kΩ (use 5.9 kΩ E96)
```

**Revised design:**

```
R = 1.6 kΩ, C = 10 nF (both filter stages)
Non-inverting buffer gain K = 1.586 (set by R_a = 10 kΩ, R_b = 5.9 kΩ)
f0 = 1 / (2π × 1600 × 10e-9) = 9.95 kHz ≈ 10 kHz
Q  = 1 / (3 - 1.586) = 1 / 1.414 = 0.707 (Butterworth) ✓
```

---

### Question I2
**An inverting op-amp circuit is used to amplify a ±100 mV input signal from a thermocouple amplifier to a ±5 V range. The op-amp runs on a ±15 V supply. Ri = 1 kΩ, Rf = 50 kΩ. Calculate: (a) the closed-loop gain; (b) the output voltage range; (c) the maximum input signal before output clipping (assuming the op-amp can swing to ±13 V); (d) whether the op-amp can drive a 600 Ω load at full output.**

**Answer:**

**Part (a) — Gain:**

```
Vout/Vin = -Rf/Ri = -50 kΩ / 1 kΩ = -50
```

**Part (b) — Output voltage range:**

```
For Vin = ±100 mV: Vout = -50 × (±100 mV) = ∓5 V
Output range: +5 V to -5 V (with input range ±100 mV)
```

**Part (c) — Maximum input before clipping:**

```
Vout_max = ±13 V (given op-amp rail-to-rail limitation)
Vin_max = Vout_max / |Gain| = 13 V / 50 = 260 mV

For inputs above ±260 mV, the output clips at ±13 V.
The specified ±100 mV input gives ±5 V output — there is 160 mV of headroom
before clipping, which provides adequate protection against input overdrive.
```

**Part (d) — Output drive capability:**

Most general-purpose op-amps can source/sink 10–25 mA short-circuit current. At full output:

```
V_out = 5 V (worst case)
R_load = 600 Ω
I_out = 5 V / 600 Ω = 8.3 mA

Check datasheet for the specified op-amp. If the datasheet lists short-circuit
current ≥ 10 mA, the amplifier can drive the 600 Ω load.

However: check the output voltage under load. If the op-amp has a non-zero
output resistance Rout, the output will drop:
  Vout_loaded = Vout_unloaded × Rload / (Rload + Rout_effective)

For a well-designed feedback amplifier, Rout_effective ≈ open-loop Rout / loop gain
  ≈ few Ω / few thousand = milliohms — negligible for 600 Ω loads.
```

If the op-amp cannot drive 600 Ω directly (some precision op-amps have limited output current), add a unity-gain buffer (voltage follower) at the output using a higher-current op-amp.

---

## Tier 3 — Advanced

### Question A1
**Design a precision current source that delivers exactly 1 mA into a variable load impedance (0–10 kΩ), using an op-amp and a reference voltage of 1.0 V. The current must be accurate to ±0.1%. Derive the transfer function, identify the dominant error sources, and specify the required op-amp parameters.**

**Answer:**

**Howland current pump (improved):**

The standard Howland current pump is the most common op-amp current source topology for grounded loads:

```
                 R1          R2
Vref --/\/\--+--[V–]--+--/\/\--+-- Vout_op
             |         |        |
            R3        LOAD     R2
             |         |        |
            GND       GND      [V+]
                               |
             R4 (= R1 || R2, or omit if precision trimmed)

Improved Howland with matching condition R1=R2=R3=R4=R:
  I_load = Vref / R
```

**Simpler implementation — Voltage-to-current converter for grounded load:**

```
       Rf        Rsense
Vref --+--[V–]--[Op-Amp]--+---[Rsense]---+--- Vload (load connects to ground)
       |                   |               |
      [Ri]                 +-- feedback ---+
       |
      GND

Transfer function:
  The op-amp forces V– = V+ = Vref.
  Current through Rsense: I = Vref / Rsense (regardless of load impedance)

  For I = 1 mA, Vref = 1.0 V:
    Rsense = Vref / I = 1.0 / 0.001 = 1000 Ω = 1 kΩ

Compliance voltage (max load voltage before op-amp saturates):
  V_load_max = V_supply+ – V_sat – I × Rsense
  For ±15 V supply, Vsat = 2 V, Rsense drop = 1 V:
  V_load_max = 15 – 2 – 1 = 12 V → max load = 12 V / 1 mA = 12 kΩ
  Meets the 0–10 kΩ requirement with margin.
```

**Dominant error sources:**

```
1. Vref accuracy:
   Error in current = Error in Vref / Rsense
   For ±0.1% current accuracy at 1 mA: ΔVref ≤ ±1 µV × 1000 = ±1 mV
   → Use a precision voltage reference (e.g., REF2025 or LT6654): Vref
     accuracy ≤ ±0.05% (0.5 mV) at 25°C.

2. Rsense tolerance and temperature coefficient:
   0.1% tolerance resistor (metal film) → 0.1% current error
   Temperature coefficient: 25 ppm/°C standard metal film
   Over 50°C range: 1250 ppm = 0.125% additional error
   → Use a precision low-TC resistor (Vishay Z-foil: <2 ppm/°C) or
     a 0.02% tolerance precision wirewound resistor.

3. Op-amp input offset voltage (Vos):
   Vos appears in series with Vref. For 1 kΩ Rsense:
   Error from Vos: ΔI = Vos / Rsense
   At 0.1% = 1 µA: Vos ≤ 1 µA × 1000 Ω = 1 mV
   → Specify op-amp Vos ≤ 100 µV for margin (e.g., OPA2188: Vos = 5 µV typ).

4. Op-amp bias current:
   IB flows through Rsense even with Vref = 0, causing an offset current.
   ΔI = IB (appears as a fixed offset).
   For 0.1% = 1 µA: IB ≤ 1 µA
   → Use a JFET or CMOS input op-amp (IB < 10 pA), not bipolar.
```

**Required op-amp parameters:**

```
Vos:   ≤ 100 µV
IB:    ≤ 1 nA (JFET/CMOS input)
GBW:   ≥ 1 MHz (for stable closed-loop behaviour driving resistive/capacitive loads)
Vout_swing: must reach 11 V for 10 kΩ load at 1 mA + 1 V Rsense drop

Recommended: OPA2188 (Vos = 5 µV, IB = 20 pA, GBW = 2 MHz, ±18 V supply)
or OPA277 (Vos = 10 µV, IB = 1 nA, GBW = 1 MHz)
```

---

### Question A2
**An op-amp is connected as an integrator with R = 100 kΩ and C = 10 nF. The op-amp has input offset voltage Vos = 2 mV and bias current IB = 10 nA. Calculate the output drift rate due to each error source and specify a stabilisation scheme.**

**Answer:**

**DC transfer function of an ideal integrator:**

```
Vout(t) = -(1/RC) × ∫Vin dt

With Vin = 0 (no signal), any DC error at the input integrates without bound.
```

**Drift due to Vos:**

```
Time constant: RC = 100 kΩ × 10 nF = 1 ms
Drift rate due to Vos:
  dVout/dt = Vos / RC = 2 mV / 1 ms = 2 V/s

The output ramps at 2 V/s from the initial condition. A ±15 V supply saturates
in: (15 V - V_initial) / 2 V/s ≈ 7.5 s
```

**Drift due to IB:**

```
IB flows into the feedback capacitor C (not through R). The capacitor charges:
  dVout/dt = IB / C = 10 nA / 10 nF = 1 V/s

Total drift rate (assuming additive worst case): 2 + 1 = 3 V/s
```

**Stabilisation schemes:**

```
1. Add a large resistor Rf across the capacitor:
   Rf = 10 MΩ in parallel with C.
   This creates a real pole at f_p = 1/(2π × Rf × C) = 1.6 Hz.
   Below 1.6 Hz: circuit acts as an amplifier with gain = -Rf/R = -100 V/V.
   Above 1.6 Hz: circuit acts as an integrator.
   The DC gain is now limited to -100 V/V, preventing output saturation.
   Output offset due to Vos: Vout_dc = Vos × (Rf/R) = 2 mV × 100 = 200 mV
   — acceptable for most applications.

2. Automatic integrator reset:
   A CMOS switch in parallel with C, driven by a comparator that detects output
   approaching the supply rail. When the output exceeds ±10 V, the switch closes
   briefly to discharge C. Used in sample-and-hold and charge-measurement circuits.

3. Use an auto-zero (chopper-stabilised) op-amp:
   These devices internally cancel Vos by periodically auto-zeroing the input stage.
   Effective Vos < 1 µV, effectively eliminating offset-driven drift.
   Example: LTC2057 (Vos = 0.5 µV max, IB = 100 pA). Drift rate becomes:
   dVout/dt = 0.5 µV / 1 ms = 0.5 mV/s — orders of magnitude improvement.
```

**Best practice:** Use Rf stabilisation (option 1) as the standard circuit technique. Use auto-zero op-amps when the lowest drift is required and the chopper switching noise at the clock frequency is tolerable in the application.

---

## Best Practices

1. Always place a compensation resistor at the non-inverting input equal to Rf || Ri to minimise offset caused by bias current for bipolar-input op-amps. This is unnecessary for CMOS-input devices with IB < 1 pA.
2. For capacitive loads, add a 10–100 Ω series "snubber" resistor between the op-amp output pin and the capacitor to prevent phase-margin degradation and oscillation.
3. Verify the gain-bandwidth product is sufficient: the op-amp GBW must be at least 10× the maximum signal frequency times the noise gain (1 + Rf/Ri) to maintain accurate amplification.
4. Decouple op-amp supply pins with 100 nF ceramic + 10 µF electrolytic within 5 mm of the device. Op-amp supply rejection degrades with frequency; local decoupling is critical.
5. For integrators that must not saturate, always include a resistor across the feedback capacitor to stabilise the DC operating point.
6. Choose the input topology (inverting vs non-inverting) based on source impedance: non-inverting for high-impedance sources, inverting when a defined input impedance is required.

---

## Related Topics

- [ADC/DAC Interfaces](adc_dac_interfaces.md)
- [Sensor Signal Conditioning](sensor_signal_conditioning.md)
- [Noise Analysis](noise_analysis.md)
- [Problem 01: Instrumentation Amplifier](worked_problems/problem_01_instrumentation_amp.md)
- [Problem 03: Filter Design](worked_problems/problem_03_filter_design.md)
- [Decoupling Strategy](../01_schematic_design/decoupling_strategy.md)
