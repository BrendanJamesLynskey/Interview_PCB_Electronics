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

Additionally, the RTC keeps poor time. After 24 hours it is about 36 seconds ahead of an
NTP-synchronised PC, and it gains the same amount each day. The 32.768 kHz crystal is an
Abracon ABS07-32.768KHZ-7 (CL = 7 pF, ±20 ppm at 25 °C).

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

The RTC gains 36 s per day:

```
Error = 36 s / 86 400 s = 4.2 × 10⁻⁴ ≈ +417 ppm
f_actual ≈ 32 768 Hz × (1 + 417 ppm) ≈ 32 781.7 Hz
```

This is about 20× the crystal's ±20 ppm tolerance. A stopwatch check would never find it,
because over 60 seconds the RTC gains only 60 × 417 ppm = 0.025 s.

#### Hypothesis Formation

A crystal running hundreds of ppm fast, not percent, points to the oscillator circuit
rather than a wrong part:
1. Load capacitance too small: the crystal is load-pulled to a higher frequency
2. Wrong crystal CL variant fitted (e.g. a 12.5 pF crystal in a circuit designed for 7 pF
   would run slow, not fast, so this does not fit)
3. RTC prescaler or calibration register misconfigured (a firmware check)

#### Step 1 — Measure the LSE Frequency Without Loading It

Do not probe the 32.768 kHz crystal pins. A 10x probe adds 10-15 pF, which is more than
the crystal's whole load capacitance. It would pull the frequency down, hiding the fault,
or stop the oscillator altogether. Instead, route the RTC calibration output (512 Hz, the
LSE divided by 64) to its pin and measure it with a frequency counter:

```
Measured: 512.211 Hz  →  (512.211 / 512 − 1) = +412 ppm
LSE      = 64 × 512.211 = 32 781.5 Hz
```

This agrees with the 24-hour drift. The prescaler and calibration register are at their
defaults, which rules out hypothesis 3.

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

#### Step 3 — Confirm With the Pulling Calculation

A crystal's load-resonant frequency sits above its series resonance fs by:

```
Δf/f (CL) = C1 / (2 × (C0 + CL))
```

The ABS07 datasheet gives C0 = 0.9-1.2 pF. It does not list C1; Abracon's tuning-fork
crystals are typically 1-4 fF (AB26T datasheet), so take C0 = 1.0 pF and C1 = 3 fF:

```
At rated CL = 7 pF:     3 fF / (2 × 8.0 pF)  = 188 ppm above fs  (the calibrated point)
At actual CL = 1.5 pF:  3 fF / (2 × 2.5 pF)  = 600 ppm above fs
Pulling = 600 − 188 = +412 ppm  →  32 781.5 Hz, +35.6 s/day
```

This matches the measurement. Over the datasheet ranges of C1 (1-4 fF) and C0
(0.9-1.2 pF) the same fault gives +124 to +580 ppm, so a missing-load-capacitor fault
always shows up as hundreds of ppm, never as percent. The upper bound is
fp − fs = C1 / (2 × C0) ≈ 1500 ppm.

The wrong capacitors also make the oscillator about ten times more sensitive to stray
capacitance. The sensitivity is C1 / (2 × (C0 + CL)²): 240 ppm/pF at 1.5 pF, against
23 ppm/pF at the rated 7 pF.

#### Root Cause of Fault 2: Wrong Capacitor Value (12× Too Small)

BOM review reveals CX3 and CX4 were specified as "12p" (12 pF) but the contract
manufacturer picked 1 pF components. The most likely cause is a BOM formatting issue:
the "p" suffix was lost during BOM export, leaving the value field reading "12" which
was ambiguous. Some CM systems default to the smallest common available value when
a unit suffix is missing.

#### Fix for Fault 2

Replace CX3 and CX4 with 12 pF C0G 0402 capacitors.

After replacement: CL = (12 × 12) / (12 + 12) + 1 = 7 pF. The 512 Hz calibration output
should now read within 512 Hz ± 20 ppm (±0.010 Hz), and the RTC should drift less than
±1.7 s/day.

For long-term accuracy, use the STM32H743's RTC calibration register to apply a
trim value that compensates for the residual frequency offset:

```
Available calibration range: −487.1 ppm to +488.5 ppm (in 0.954 ppm steps;
STM32 reference manual, RTC smooth digital calibration)
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

With too little load capacitance: operating point moves toward fp → frequency increases
With too much load capacitance: operating point moves toward fs → frequency decreases
Correct CL: operating point at the specified load-resonant frequency (between fs and fp)
```

---

## Key Takeaways

1. **Always specify C0G (NP0) capacitors for crystal load.** X7R capacitors introduce
   frequency variation with temperature, voltage, and ageing that directly translates
   to oscillator frequency drift.

2. **Crystal load capacitance errors cause both startup failures (too large) and
   frequency errors (too small or too large).** The direction of frequency error is:
   too much load capacitance → frequency too low; too little → frequency too high.

3. **Errors far outside the crystal's tolerance are a hardware issue, not a software trim
   issue.** The +412 ppm error happens to fit inside the RTC's −487 to +489 ppm trim
   range, but it would use 84 % of that range. It would also leave an oscillator ten
   times more sensitive to stray capacitance, humidity and temperature. Firmware
   calibration is for fine adjustment of correctly operating hardware (tens of ppm), not
   for correcting wrong component values.

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
  Adding load capacitance pulls the operating frequency down from fp toward fs.
  Higher load → lower frequency. Lower load → higher frequency (by at most
  about C1/(2 C0), i.e. hundreds to a few thousand ppm). Operating
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
