# Problem 02: Clock Failure

## Problem Statement

A new board has been brought up through Phase 1 (power rails verified) and Phase 2
(visual inspection passed) successfully. The design uses:
- A 25 MHz crystal oscillator (XTAL) generating the reference clock for a system PLL
- An STM32H743 microcontroller consuming the 25 MHz reference via its HSE input
- The PLL is configured in firmware to multiply the 25 MHz reference to 480 MHz core clock
- A secondary 32.768 kHz crystal (LSE) for the real-time clock (RTC) function

During Phase 3 (processor boot), JTAG connection is successful and the device ID is
correct. However, when firmware attempts to configure the PLL and switch to the HSE
clock source, the microcontroller enters its clock fault handler (HSE timeout interrupt
fires, CSS — clock security system — asserts a fault flag).

Additionally, the RTC is initialised but immediately shows incorrect timekeeping —
the seconds register advances at approximately 1.08 seconds per real second, confirmed
with a stopwatch over 60 seconds.

**Task:** Diagnose both clock faults independently and identify the root cause of each.

---

## Solution Approach

### Fault 1: 25 MHz HSE Fails to Start

#### Hypothesis Formation

The CSS (Clock Security System) fires after HSE fails to achieve oscillation within
its startup timeout (~5 ms). The HSE failure has five plausible causes:

```
1. Crystal not oscillating at all (most common)
   → Wrong load capacitors, crystal damaged, oscillator circuit fault

2. Crystal oscillating but not reaching specification amplitude
   → Load capacitor value wrong, excessive PCB trace capacitance

3. Crystal oscillating but at wrong frequency
   → Wrong crystal value loaded (wrong BOM part), or extreme load mismatch

4. HSE input is correct but not connected to MCU pin
   → PCB routing error (net not connected to HSE_IN pin)

5. MCU HSE pin configuration error in firmware
   → Pin configured as GPIO instead of oscillator function (less likely if JTAG works)
```

#### Step 1 — Probe the Crystal Oscillator Circuit

Use an oscilloscope with a 10x probe and, critically, a short ground spring. Crystal
oscillators are high-impedance circuits — even the capacitance of a standard probe
can disrupt or kill the oscillation.

```
Probe setup for crystal measurement:
  - 10x passive probe (reduces probe capacitance from ~100 pF to ~10-15 pF)
  - Short ground spring (2-5 mm, reduces ground inductance to < 5 nH)
  - Vertical: 500 mV/div
  - Horizontal: 20 ns/div (for 25 MHz, period = 40 ns)
  - Trigger: edge, auto, on the XOUT pin of the crystal
```

Probe the XOUT pin (the output of the crystal — the pin connected to the MCU's HSE_IN).

**Result observed:** No oscillation present. The XOUT pin is DC-stable at approximately
1.6 V — the DC bias voltage set by the internal oscillator amplifier bias network.
No sinusoidal or clipped waveform is visible.

The crystal is not oscillating. This is a startup failure.

#### Step 2 — Verify Load Capacitors

The crystal datasheet specifies a load capacitance (CL) of 12 pF. The circuit uses two
load capacitors (CX1 and CX2) in a Pi configuration:

```
MCU OSC_IN  ─── CX1 ─── GND
MCU OSC_OUT ─── CX2 ─── GND
Crystal pin 1 ─ MCU OSC_IN
Crystal pin 2 ─ MCU OSC_OUT

The load capacitance seen by the crystal:
  CL = (CX1 × CX2) / (CX1 + CX2) + C_stray
```

For CL = 12 pF target and assuming C_stray ≈ 2 pF from PCB traces:

```
Required: (CX1 × CX2) / (CX1 + CX2) = 12 - 2 = 10 pF
For equal capacitors: CX1 = CX2 = 20 pF
```

Remove CX1 and measure it with an LCR meter: reads **100 pF**. Check CX2: also 100 pF.

The load capacitors are 5× the correct value. The circuit has CX1 = CX2 = 100 pF
instead of the required 20 pF.

#### Root Cause of Fault 1: Wrong Capacitor Value

100 pF load capacitors present a load capacitance of:

```
CL_actual = (100 × 100) / (100 + 100) + 2 = 50 + 2 = 52 pF
  vs. required CL = 12 pF
```

With 52 pF of load capacitance instead of 12 pF, the Pierce oscillator circuit (used
internally by the STM32 HSE) cannot start reliably. The total capacitive load is too
high for the oscillator's transconductance to overcome, and the crystal cannot build
up oscillation.

Additionally, even if oscillation did start, the frequency would be significantly pulled
below specification, causing PLL lock failure.

#### Fix for Fault 1

Replace CX1 and CX2 with 20 pF C0G (NP0) capacitors.

**Why C0G (NP0)?**
Load capacitors for crystal oscillators must use C0G dielectric. X7R capacitors:
- Have ±15% capacitance variation over temperature — directly varies oscillator
  frequency by load-pulling the crystal over the operating temperature range
- Have a piezoelectric effect that introduces phase noise
- Have a DC voltage coefficient (capacitance varies with bias voltage)

C0G capacitors have < ±0.3% variation over temperature, no piezoelectric effect, and
no DC bias dependence — essential for a stable reference clock.

After replacing the load capacitors, probe the crystal again:

```
Result: 25 MHz sinusoidal waveform visible at XOUT pin, approximately 600 mVpp
(the XOUT amplitude should be a clipped sine — this is normal for a Pierce oscillator
driving into the MCU's inverting amplifier)
```

Firmware confirms: HSE starts, PLL locks, 480 MHz core clock verified by toggling a
GPIO at a measured rate with the scope.

---

### Fault 2: 32.768 kHz RTC Advancing Too Fast

The RTC is advancing at approximately 1.08 seconds per real second (8% fast over a
60-second test period).

#### Hypothesis Formation

The RTC frequency error of +8% implies the 32.768 kHz crystal is oscillating at a
frequency approximately 8% higher than nominal: approximately 35.4 kHz.

```
f_actual = 32.768 kHz × 1.08 = 35.39 kHz
Deviation = +8%
```

A crystal running 8% fast is not a calibration or trim issue — that is a gross error.
Possible causes:
1. Wrong crystal value loaded — but no standard crystal value falls 8% above 32.768 kHz
2. Load capacitors are much too small (crystal is load-pulled to a higher frequency)
3. PCB routing issue causing a parasitic resonance at a different frequency

#### Step 1 — Probe the 32.768 kHz Crystal

```
Probe setup:
  - 10x passive probe with short ground spring
  - Vertical: 500 mV/div
  - Horizontal: 10 µs/div (32.768 kHz period ≈ 30.5 µs)
  - Use oscilloscope frequency counter function
```

Probe the LSE_OUT pin. The oscilloscope confirms oscillation at 35.4 kHz.

The crystal is oscillating above its nominal frequency.

#### Step 2 — Check Load Capacitors

The 32.768 kHz crystal datasheet specifies CL = 7 pF.

```
Required (equal capacitors): CX1 = CX2 = 2 × (CL_target - C_stray)
Assuming C_stray ≈ 1 pF for short traces to MCU LSE pin:
  CX1 = CX2 = 2 × (7 - 1) = 12 pF
```

Measure the installed load capacitors with an LCR meter: CX3 = CX4 = **1 pF**.

Load capacitance presented to the crystal:
```
CL_actual = (1 × 1) / (1 + 1) + 1 = 0.5 + 1 = 1.5 pF
  vs. required CL = 7 pF
```

With only 1.5 pF of load capacitance instead of the required 7 pF, the crystal is
load-pulled toward its series resonance frequency. The series resonant frequency is
slightly higher than the parallel resonance at rated CL, so the crystal runs fast.

#### Root Cause of Fault 2: Wrong Capacitor Value (12× Too Small)

BOM review reveals CX3 and CX4 were specified as "12p" (12 pF) but the contract
manufacturer picked 1 pF components. The most likely cause is a BOM formatting issue:
the "p" suffix was lost during BOM export, leaving the value field reading "12" which
was ambiguous. Some CM systems default to the smallest common available value when
a unit suffix is missing.

#### Fix for Fault 2

Replace CX3 and CX4 with 12 pF C0G 0402 capacitors.

After replacement: RTC measured at 1.001 seconds per real second over 60 seconds
(within the crystal's ±20 ppm specification at room temperature).

For long-term accuracy, use the STM32H743's RTC calibration register to apply a
trim value that compensates for the residual frequency offset:

```
Available calibration range: ±512 ppm (in 0.954 ppm steps)
Measurement method: compare RTC output to a GPS 1-PPS signal over 24 hours,
then apply the correction factor to the CALR register
```

---

## Analysis

### Why Crystal Oscillator Failures Are Common on First Articles

Crystal oscillator circuits are sensitive analogue sub-circuits that are frequently
treated as trivial by schematic designers. The key parameters that must be verified:

```
Parameter               Consequence of error
--------------------    --------------------------------------------------
Load capacitor value    Frequency error, failure to start
Load capacitor type     Temperature drift (X7R vs C0G), phase noise
Trace length to XTAL    Parasitic capacitance pulls frequency; XOUT trace
                        also acts as a low-level RF transmitter (EMI source)
Supply decoupling       Noise on VDD couples into oscillator output as jitter
Probe loading during    Even a 10x probe can disrupt a 32 kHz crystal — never
debug                   probe LSE pins directly if avoidable
```

### Load Capacitance Formula for Interview Recall

```
CL_effective = (C1 × C2) / (C1 + C2) + C_stray

For two equal capacitors C1 = C2 = C:
  C = 2 × (CL_target - C_stray)

Where:
  CL_target = specified in crystal datasheet
  C_stray   = 1-3 pF from PCB traces (estimate from layout; measure empirically)
```

### Series vs. Parallel Resonance

A crystal oscillator operates between two resonant frequencies:

```
fs (series resonance): crystal impedance is minimum (purely resistive)
                       → frequency is slightly lower than fp
fp (parallel resonance, anti-resonance): crystal impedance is maximum
                       → operating point with correct CL is between fs and fp

With too little load capacitance: operating point moves toward fs → frequency increases
With too much load capacitance: operating point moves toward fp → frequency decreases
Correct CL: operating point at the specified parallel resonant frequency
```

---

## Key Takeaways

1. **Always specify C0G (NP0) capacitors for crystal load.** X7R capacitors introduce
   frequency variation with temperature, voltage, and ageing that directly translates
   to oscillator frequency drift.

2. **Crystal load capacitance errors cause both startup failures (too large) and
   frequency errors (too small or too large).** The direction of frequency error is:
   too much load capacitance → frequency too low; too little → frequency too high.

3. **Gross frequency errors (> 0.1%) are a hardware issue, not a software trim issue.**
   An 8% RTC frequency error cannot be fixed in software. Firmware calibration is for
   fine adjustment of correctly operating hardware, not for correcting wrong component
   values.

4. **Probing technique is critical for crystal measurements.** A standard 1x probe with
   a flying ground lead will typically kill a 32.768 kHz tuning fork crystal oscillation
   and significantly perturb a 25 MHz crystal circuit. Use a 10x probe with a short
   ground spring and minimise probe contact time.

5. **BOM formatting is a frequent source of component value errors.** Engineering ECOs
   must include explicit units in all component value fields, and the CM BOM must include
   the Manufacturer Part Number (MPN) as the authoritative specification — not just
   the value field.

---

## Interview Notes

**What types of questions this problem covers:**

- "Why did my crystal fail to oscillate?"
  Load capacitance too high, wrong dielectric, trace too long, probe loading, or damaged
  crystal. Verify load capacitors first — they are the most common cause.

- "How does load capacitance affect crystal frequency?"
  Crystal series resonant frequency is pulled toward parallel resonance by load
  capacitance. Higher load → lower frequency. Lower load → higher frequency. Operating
  outside the specified load range causes both frequency error and potential instability.

- "What is the difference between series and parallel resonance in a crystal?"
  Series resonance (fs): minimum impedance, purely resistive.
  Parallel resonance (fp): maximum impedance. Normal crystal oscillators (Pierce circuit)
  operate between fs and fp, at a point determined by the load capacitance.

- "Why use C0G and not X7R for crystal load capacitors?"
  C0G has near-zero temperature coefficient, no voltage coefficient, no ageing, and no
  piezoelectric effect. X7R has all of these — each introduces oscillator frequency
  variation and phase noise.

- "What would you check first if a processor's PLL fails to lock?"
  1. Verify the reference clock (HSE) is running at the correct frequency
  2. Verify PLL divider settings (M, N, R) in firmware match the clock plan
  3. Verify PLL lock time is within the firmware timeout period
  4. Check the PLL supply (e.g., VCAP pins on STM32, which require specific external
     capacitor values to stabilise the internal voltage regulator for the PLL)
