# Sensor Signal Conditioning

## Prerequisites
- Op-amp circuits: inverting, non-inverting, instrumentation amplifier (see `op_amp_circuits.md`)
- ADC interface requirements: input range, input impedance (see `adc_dac_interfaces.md`)
- Basic sensor physics: Wheatstone bridge, thermocouple, RTD, current loop
- Noise fundamentals: SNR budget, noise referred to input (see `noise_analysis.md`)

---

## Concept Reference

### What Signal Conditioning Achieves

A sensor produces a raw output that is mismatched to an ADC or processing system in one or more of:
- Signal amplitude (µV to mV from sensors vs 0-5 V ADC range)
- Signal type (differential, single-ended, current loop)
- Output impedance (bridge output may be 350 Ω; ADC needs < 200 Ω drive)
- Common-mode voltage (sensor output may be riding on a reference voltage)
- Frequency content (noise and interference must be filtered)

Signal conditioning transforms the sensor output into the form the ADC requires, while preserving signal accuracy and minimising noise contribution from the conditioning stage itself.

### Wheatstone Bridge

The Wheatstone bridge is the fundamental transducer circuit for resistive sensors (strain gauges, load cells, pressure transducers, RTDs in some configurations):

```
         Vexc
           |
      R1       R2
    ---/\/\---+---/\/\---
    |         |         |
   Vout+      +        Vout–
    |         |         |
    ---/\/\---+---/\/\---
      R3       R4
           |
          GND

Balanced condition (R1/R2 = R3/R4): Vout = 0

For a sensor strain gauge where R4 changes by ΔR:
  Vout = Vexc × ΔR / (4R + 2ΔR) ≈ Vexc × ΔR / (4R)  for small ΔR

Quarter-bridge (one active gauge):
  Sensitivity = Vexc × GF × ε / 4
  Where GF = gauge factor (typically 2 for metallic strain gauges)
              ε = strain

For Vexc = 5 V, GF = 2, strain = 1000 µε (1000 microstrain):
  Vout = 5 × 2 × 1000e-6 / 4 = 2.5 mV
```

**Key bridge characteristics:**

- Output is a small differential voltage (µV to mV range) on top of a large common-mode voltage (Vexc/2 for a balanced bridge).
- The output impedance is R (the bridge resistor value) seen differentially.
- Requires an instrumentation amplifier with high CMRR to reject the common-mode voltage.

### RTD Signal Conditioning

Platinum resistance thermometers (PT100, PT1000) change resistance linearly with temperature:

```
R(T) = R0 × (1 + α × T)   (simplified Callendar-Van Dusen for -50 to +150°C)

Where:
  R0  = resistance at 0°C (100 Ω for PT100, 1000 Ω for PT1000)
  α   = 0.00385 Ω/Ω/°C (IEC 60751 standard)

PT100 at 0°C:  100 Ω
PT100 at 100°C: 138.5 Ω (38.5 Ω change over 100°C)
PT1000 at 100°C: 1385 Ω (385 Ω change over 100°C)
```

**Conditioning approach:**

```
Method 1 — Ratiometric bridge:
  Place RTD as one arm of a Wheatstone bridge. Bridge output → INA → ADC.
  Advantage: Vexc noise cancels (ratiometric — both signal and reference scale with Vexc).
  Disadvantage: nonlinearity from bridge unbalancing.

Method 2 — Constant current excitation:
  Excite RTD with a precision constant current Iexc (e.g., 1 mA).
  V_RTD = Iexc × R(T) → amplify and digitise.
  Advantage: linear output. Disadvantage: self-heating: P = I² × R = 0.1 mW at PT100 — raises T_measured by ~0.025°C in still air.

Method 3 — 3-wire or 4-wire connection:
  2-wire: lead resistance adds directly to RTD measurement.
  3-wire: lead resistance partially cancelled (two reference wires, one sense wire).
  4-wire (Kelvin): force current through two wires, sense voltage on two separate wires — lead resistance completely eliminated. Required for ±0.1°C accuracy over long cable runs.
```

### Thermocouple Signal Conditioning

Thermocouples generate a voltage from the Seebeck effect at the junction of two dissimilar metals:

```
Common thermocouple types:
  Type K (NiCr-NiAl): 41 µV/°C, -200 to +1260°C, most common industrial
  Type J (Fe-Const):  51 µV/°C, -40 to +750°C
  Type T (Cu-Const):  43 µV/°C, -200 to +350°C, good for low temperature
  Type S (Pt-Rh/Pt):  7-10 µV/°C, 0 to +1750°C, primary standard

Output range example — Type K, 0 to 500°C:
  V_out = 41 µV/°C × 500°C = 20.5 mV
```

**Cold junction compensation (CJC):**

A thermocouple measures the temperature difference between the hot junction (sensing point) and the cold junction (where the thermocouple wire connects to the copper PCB trace). The cold junction temperature must be independently measured (typically with a local temperature sensor like NTC thermistor or a dedicated CJC IC) and added to the thermocouple voltage reading:

```
T_hot = T_cold + V_thermocouple / Sensitivity

If T_cold varies ±2°C and CJC is not compensated: measurement error ±2°C.
```

**Conditioning chain for thermocouple:**

```
[Thermocouple] → low-pass filter (60 Hz rejection) → high-gain INA (gain ~100-500)
              → ADC → CJC correction (in software or dedicated IC)

Gain calculation: For Type K, 0-500°C = 0-20.5 mV:
  ADC range 0-3.3 V → gain = 3.3 / 0.0205 = 161
  Select INA gain = 128 or 200 (adjust reference offset to centre the range).
```

### Current Loop (4-20 mA) Signal Conditioning

The 4-20 mA current loop is the dominant sensor interface in industrial process control. A 4 mA current represents 0% of process range; 20 mA represents 100%. The current-mode signal is immune to voltage drops in long cable runs.

```
Signal conditioning:
  Precision sense resistor Rs (typically 100-250 Ω) → voltage across Rs
  → amplify to ADC range

  For Rs = 100 Ω:
    4 mA → 0.4 V
    20 mA → 2.0 V
    Range: 0.4 V to 2.0 V (span = 1.6 V)

  For ADC range 0-5 V:
    Gain: 5 / 1.6 = 3.125 (set to ~3, offset adjust for 0.4 V at 0%)
    
  Alternatively: Rs = 250 Ω → 1.0 V to 5.0 V (convenient, full ADC range on 5 V ADC)
```

**Open-loop fault detection:** At 4 mA live zero, a reading below 4 mA (< 0.4 V across 100 Ω) indicates a broken wire or failed transmitter — the loop has failed open.

### The Signal Conditioning Chain Design Procedure

```
1. Characterise the sensor output:
   - Output type (voltage, current, resistance, charge)
   - Output range (full scale in correct units)
   - Output impedance
   - Common-mode voltage
   - Frequency content (signal bandwidth, noise, interference)

2. Define the ADC requirements:
   - Input range and reference
   - Input impedance
   - Maximum input frequency (Nyquist)

3. Design each stage:
   a. Input protection (ESD, overvoltage)
   b. Amplification to match ADC range
   c. Offset adjustment if needed
   d. Anti-aliasing / noise filtering
   e. Buffering (output impedance matching to ADC)

4. Perform noise budget:
   - Refer all noise sources to the input
   - Sum (in quadrature for uncorrelated sources)
   - Compare to required SNR

5. Select op-amps / INA based on noise and accuracy budget:
   - Vos for DC accuracy
   - en (input voltage noise) for AC/noise performance
   - in (input current noise) × source impedance for current noise term
```

---

## Tier 1 — Fundamentals

### Question F1
**A PT100 RTD is excited by a 1 mA constant current source. At 20°C, R(20°C) = 107.7 Ω. The voltage across the RTD is amplified by an INA with gain = 10 and fed to a 12-bit ADC with 3.3 V full-scale range. What is the ADC input voltage at 20°C? What does 1 ADC code represent in temperature?**

**Answer:**

```
Voltage across RTD at 20°C:
  V_RTD = Iexc × R(T) = 1 mA × 107.7 Ω = 107.7 mV

After INA (gain = 10):
  V_ADC = 107.7 mV × 10 = 1.077 V

ADC code at 20°C (12-bit, 3.3 V range):
  Code = V_ADC / LSB = 1.077 V / (3.3/4096) = 1.077 / 0.000806 = 1336
```

Temperature resolution (1 ADC code):

```
1 LSB = 3.3 V / 4096 = 0.806 mV at the ADC input
At INA output: 0.806 mV
Referred to INA input: 0.806 mV / 10 = 80.6 µV

Change in RTD voltage per °C:
  dV/dT = Iexc × R0 × α = 1 mA × 100 Ω × 0.00385 = 0.385 mV/°C

Temperature per code:
  ΔT = 80.6 µV / 0.385 mV/°C = 0.209°C/code ≈ 0.2°C/LSB
```

This is the ideal temperature resolution — 0.2°C per ADC code. Actual accuracy depends on INA offset voltage, gain accuracy, and ADC INL, which must be evaluated in the noise/accuracy budget.

---

### Question F2
**A strain gauge bridge is excited with Vexc = 5 V. At full load (1000 µε), the output is ΔVout = 2.5 mV (differential). An INA must amplify this to 0-2.5 V for the ADC. What gain is required? What are the constraints on the INA's input offset voltage Vos if the system must resolve 10 µε (1% of full scale)?**

**Answer:**

```
Required gain:
  Gain = 2.5 V / 2.5 mV = 1000

This is a high gain (1000×). Most single-device INAs can achieve this, but
stability, bandwidth, and noise must be checked at this gain level.
```

**Offset voltage constraint:**

10 µε = 1% of 1000 µε → 1% of full-scale = 0.01 × 2.5 mV = 25 µV.

The INA output offset at gain = 1000:

```
Vout_offset = Vos × Gain = Vos × 1000

For this to be < 1% of full-scale output (0.01 × 2.5 V = 25 mV):
  Vos × 1000 ≤ 25 mV
  Vos ≤ 25 µV

The INA input offset voltage must be ≤ 25 µV.
```

This is a precision specification. Standard INAs (e.g., INA128) have Vos = 25-100 µV. For ≤ 25 µV, select a precision INA (e.g., INA819: Vos = 10 µV max, INA333: Vos = 25 µV max) or use an autozero/chopper device.

**Note:** Vos drifts with temperature. At 50 ppm/°C and 50°C temperature range: ΔVos = 10 µV × 50 ppm/°C × 50°C = 25 µV additional drift — this equals the full offset budget and must also be accounted for. In practice, system calibration at temperature, or a chopper-stabilised INA, is required for sub-10 µε accuracy over temperature.

---

## Tier 2 — Intermediate

### Question I1
**Design a complete signal conditioning chain for a Type K thermocouple measuring 0–500°C. Specify: (a) the amplifier gain; (b) the low-pass filter cutoff frequency if the measurement is updated at 10 Hz; (c) the cold junction compensation method; (d) the ADC resolution required to meet 1°C measurement accuracy.**

**Answer:**

**Part (a) — Amplifier gain:**

```
Type K sensitivity: 41 µV/°C
Output at 500°C: 41 µV/°C × 500°C = 20.5 mV

Target ADC input range: 0-3.3 V (single supply, reference 3.3 V)
  Gain = 3.3 V / 0.0205 V = 161

Use a gain of 128 or 200 (standard INA gain resistor options):
  At gain = 200: full-scale output = 20.5 mV × 200 = 4.1 V (slightly clipping with 3.3 V ADC)
  At gain = 128: full-scale output = 20.5 mV × 128 = 2.62 V (fits 3.3 V ADC with headroom)

Select gain = 128. This maps 0-20.5 mV (0-500°C) to 0-2.62 V.
Add an offset shift in software or via a reference input to the INA if the
ADC cannot accommodate the unused upper range.
```

**Part (b) — Anti-aliasing filter:**

```
Measurement rate: 10 Hz output, but the chain should be protected against
60 Hz mains interference (most common thermocouple noise source).

Anti-aliasing requirement:
  Nyquist: 5 Hz (10 Hz sample rate / 2)
  But 60 Hz must also be rejected (>74 dB for 12-bit ADC SNR).

Use a low-pass filter with:
  Cutoff: 1-3 Hz (removes 60 Hz and all higher-frequency noise)
  This is well below the 5 Hz Nyquist for 10 Hz measurement rate.

Single-pole RC filter:
  fc = 1 Hz → R × C = 1 / (2π × 1) = 0.159 s
  With C = 10 µF: R = 15.9 kΩ → use 16 kΩ

Two-pole (60 dB/decade) gives > 60 dB at 60 Hz:
  60 Hz / 1 Hz = 60 (ratio), 2-pole at -40 dB/decade:
  Attenuation = 40 × log(60) = 71 dB — marginally sufficient for 12-bit
  Add a notch at 60 Hz or use a 3rd-order filter for reliable 60 Hz rejection.
```

**Part (c) — Cold junction compensation:**

```
Use a dedicated thermocouple front-end IC (e.g., MAX31855K or AD8495) that:
  - Integrates a precision INA (gain ~122× for Type K)
  - Includes an on-chip temperature sensor for cold junction temperature
  - Provides digital output with hardware CJC performed internally

Alternatively: place a precision NTC thermistor or silicon temperature sensor
(e.g., LM35, TMP37) at the PCB location where the thermocouple terminals
connect. Read the CJC sensor on a separate ADC channel and add the
equivalent voltage (41 µV/°C × T_cjc) to the thermocouple voltage in firmware.
```

**Part (d) — Required ADC resolution:**

```
Temperature span: 500°C
Required temperature resolution: 1°C → 1/500 = 0.2% of full scale

ADC full-scale (at gain = 128): 2.62 V
Required voltage resolution: 0.002 × 2.62 V = 5.24 mV
Required voltage at INA input: 5.24 mV / 128 = 40.9 µV

1 ADC code = ADC_range / 2^N must be ≤ 40.9 µV at the INA input
After gain of 128: 1 code at INA input = ADC_FSR / (2^N × 128)

For ADC_FSR = 3.3 V:
  3.3 / (2^N × 128) ≤ 40.9 µV
  2^N ≥ 3.3 / (40.9e-6 × 128) = 630
  N ≥ log2(630) = 9.3 → minimum 10-bit ADC

In practice, a 12-bit ADC provides comfortable margin (4× overspecification),
accommodating INA gain tolerance, noise, and temperature drift of components.
```

---

## Tier 3 — Advanced

### Question A1
**A load cell with 350 Ω full bridge has a rated output of 2 mV/V at 5 kg full scale. The bridge is excited by a 5 V precision reference. Design the complete signal conditioning chain to interface with a 12-bit SAR ADC (0-5 V input range). Include: gain calculation, INA selection criteria (specify required Vos, CMRR, noise), anti-aliasing filter design for 100 Hz data rate, and a noise budget showing SNR at the ADC.**

**Answer:**

**Signal characterisation:**

```
Bridge output at full scale (5 kg):
  V_diff = Vexc × sensitivity = 5 V × 2 mV/V = 10 mV differential

Common-mode voltage: Vexc/2 = 2.5 V (balanced bridge at null)

Output impedance: 350 Ω (one pair) || 350 Ω (other pair) = 175 Ω differential
```

**Gain design:**

```
ADC range: 0-5 V
Required gain: 5 V / 10 mV = 500

INA gain: set to 500. Most precision INAs accommodate this.
(e.g., INA128 gain = 500 → R_G = 49.4 kΩ / (500 - 1) ≈ 99 Ω)
```

**INA selection criteria:**

```
CMRR requirement:
  Common-mode voltage = 2.5 V. For a 12-bit ADC (74 dB SNR), the CMRR must
  attenuate the 2.5 V CM to below the noise floor:
  Required CMRR > 20 × log(2.5 V / (5 V / 4096)) = 20 × log(2048) = 66 dB
  At gain = 500, INA CMRR is typically 100-130 dB — this requirement is easily met.

Vos requirement:
  1 LSB at ADC output = 5 V / 4096 = 1.22 mV
  Referred to INA input: 1.22 mV / 500 = 2.44 µV
  INA Vos ≤ 2.44 µV for < 1 LSB offset error

  This is extremely tight. Standard INAs (INA128: 25 µV) do not meet this.
  Options:
  (a) Chopper-stabilised INA: INA333 (25 µV), LTC1051-based custom INA (Vos < 5 µV)
  (b) Accept offset error and calibrate: 25 µV × 500 = 12.5 mV offset at output
      = 12.5/5000 × 100% = 0.25% of full-scale → removed by zero calibration

Noise requirement (en):
  At gain = 500, the INA input noise is amplified 500× to the ADC input.
  For a noise floor < 0.5 LSB in the 100 Hz bandwidth:
  Noise floor = 0.5 LSB / gain = 0.61 mV / 500 = 1.22 µV (peak) in 100 Hz BW
  RMS requirement: 1.22 µV / 3.3 = 0.37 µV RMS (assuming normal distribution, 3-sigma)

  Thermal noise of 350 Ω source: V_th = sqrt(4kT × R × BW)
  = sqrt(4 × 1.38e-23 × 300 × 175 × 100) = sqrt(2.90e-16) = 17 nV RMS
  (negligible compared to INA noise)

  Required INA voltage noise: en × sqrt(BW) ≤ 0.37 µV
  en ≤ 0.37 µV / sqrt(100) = 37 nV/√Hz

  INA128: en = 8 nV/√Hz at 1 kHz → 8 × sqrt(100) = 80 nV RMS — borderline
  INA333: en = 55 nV/√Hz → 550 nV RMS — too noisy at 100 Hz bandwidth
  AD8221: en = 7 nV/√Hz → 70 nV RMS — acceptable with some margin
```

**Anti-aliasing filter:**

```
Nyquist for 100 Hz data: 50 Hz
60 Hz mains rejection recommended (weighing scales in industrial environments)

2nd-order Butterworth low-pass, fc = 10 Hz:
  At 50 Hz: -40 dB × log(50/10) = -40 × 0.7 = -28 dB → insufficient for 74 dB ADC

Use 3rd-order at fc = 5 Hz (impractical with large RC):
  Alternatively: use the INA's internal bandwidth (if it has a FILTR pin)
  plus an external 2-pole passive filter.

Practical choice: 4th-order Butterworth, fc = 10 Hz
  Attenuation at 50 Hz: (50/10)^4 = 625 → 56 dB — marginal
  At 100 Hz: 80 dB — sufficient. Data rate of 100 Hz with 10 Hz filter causes
  aliasing of 50-100 Hz range.

  Better approach: use a delta-sigma ADC (not SAR) for this application.
  A 24-bit delta-sigma with 100 SPS output rate provides built-in 60 Hz notch
  and 6th-order sinc filter — optimal for load cell applications.
  (This is why all precision scales use sigma-delta, not SAR.)
```

**Noise budget:**

```
Source              RMS noise (referred to INA input, 100 Hz BW)
--------------      -----------------------------------------------
INA voltage noise   AD8221: 7 nV/√Hz × √100 Hz = 70 nV
INA current noise   10 pA/√Hz × 175 Ω × √100 Hz = 175 pA × 175 Ω = 0.03 nV (negligible)
Source thermal      17 nV (calculated above, negligible)
ADC quantisation    1.22 mV / 500 gain = 2.44 µV RTI

Total noise RTI = sqrt(70e-9² + 2.44e-6²) ≈ 2.44 µV  (ADC quantisation dominates)

SNR = 5 mV (half full-scale signal) / 2.44 µV = 2049 → 66 dB
      vs ideal 12-bit: 74 dB → ENOB ≈ 10.7 bits at 100 Hz bandwidth

The system achieves approximately 11 effective bits, limited by ADC quantisation.
To improve: increase ADC resolution to 16-bit, or use a delta-sigma ADC.
```

---

## Best Practices

1. For any resistive bridge sensor, use 4-wire (Kelvin) excitation and sense when lead resistance exceeds 0.1 Ω or cables are longer than 5 m — 2-wire connections introduce systematic temperature-dependent offset.
2. Set INA gain so the maximum sensor output maps to at least 80% of the ADC full-scale range. Underutilising the ADC range wastes dynamic range — the full-scale signal should not be less than 50% of ADC FSR.
3. Always include a 60 Hz (and 50 Hz for European equipment) rejection filter in industrial sensor chains — mains-coupled interference is the most common source of measurement error in factory environments.
4. Chopper-stabilised or auto-zero op-amps are mandatory for high-gain (> 100×) DC sensor amplifiers if temperature drift is a concern — standard bipolar op-amps drift tens of µV/°C at high gain.
5. Specify CJC thermal coupling carefully: the CJC sensor must be thermally connected to the thermocouple terminal block, not just placed on the same PCB. A copper trace connecting the terminal block and the CJC sensor footprint improves thermal coupling.
6. For 4-20 mA current loop inputs, always protect the sense resistor input with TVS and current-limiting resistors — the current loop may carry shorts or inductive transients from valve actuators and motors.

---

## Related Topics

- [Op-Amp Circuits](op_amp_circuits.md)
- [ADC/DAC Interfaces](adc_dac_interfaces.md)
- [Noise Analysis](noise_analysis.md)
- [Problem 01: Instrumentation Amplifier](worked_problems/problem_01_instrumentation_amp.md)
- [ESD Protection](../01_schematic_design/esd_protection.md)
