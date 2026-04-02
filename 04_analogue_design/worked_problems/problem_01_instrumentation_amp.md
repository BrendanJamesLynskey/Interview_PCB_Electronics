# Problem 01: Instrumentation Amplifier Design

## Problem Statement

You are designing the front-end amplifier for a precision pressure sensor based on a piezoresistive Wheatstone bridge. The bridge has the following characteristics:

- Supply: 5 V excitation
- Bridge resistance: 5 kΩ nominal (all four arms)
- Full-scale output: ±10 mV differential at full pressure range (1 MPa)
- Common-mode voltage: Vexc/2 = 2.5 V
- Signal bandwidth: DC to 500 Hz
- Sensitivity: 10 mV / MPa

The signal must be digitised by a 16-bit SAR ADC with 0-5 V input range.

**Part A:** Design the INA stage. Calculate: the required gain, the INA output voltage range, and specify the gain resistor value if using an INA with the formula G = 1 + 49.4 kΩ / RG.

**Part B:** The INA has the following specifications: en = 8 nV/√Hz (white noise), in = 0.5 pA/√Hz (white noise), fc = 20 Hz (1/f corner), Vos = 15 µV. Calculate the total input-referred noise in the signal bandwidth (DC to 500 Hz, treat the lower cutoff as 0.01 Hz) and determine the SNR for a full-scale ±10 mV input.

**Part C:** Calculate the CMRR requirement for the INA if the common-mode voltage can vary ±0.5 V due to excitation supply noise. The ADC requires the common-mode error at the INA output to be < 0.5 LSB.

**Part D:** The INA output must swing between 0 V and 5 V to use the full ADC range, but the INA runs from a single 5 V supply. The bridge output is bipolar (±10 mV). Design the output offset circuit to shift the INA mid-scale output to 2.5 V, mapping -10 mV → 0 V and +10 mV → 5 V.

---

## Solution Approach

### Part A — Gain and INA Output Range

**Gain requirement:**

```
INA input range: ±10 mV differential
Target ADC range: 0 to 5 V (using the full 16-bit range)

The signal is bipolar: -10 mV to +10 mV, which is 20 mV span.
ADC span: 5 V.

Required gain: G = ADC_span / Signal_span = 5 V / 20 mV = 250
```

**Output range:**

```
At -10 mV input:  Vout = -10 mV × 250 = -2.5 V
At +10 mV input:  Vout = +10 mV × 250 = +2.5 V

These are differential outputs relative to the mid-rail reference.
With a 2.5 V offset applied (Part D): 0 V to 5 V at ADC.
```

**Gain resistor calculation (INA128 or INA228 family):**

```
G = 1 + 49.4 kΩ / RG

Solving for RG:
  250 = 1 + 49.4 kΩ / RG
  249 = 49.4 kΩ / RG
  RG = 49.4 kΩ / 249 = 198.4 Ω

Use E96 preferred value: 196 Ω (nearest to 198.4 Ω).

Actual gain with 196 Ω: G = 1 + 49,400/196 = 1 + 252 = 253
Output at full scale: ±10 mV × 253 = ±2.53 V — slightly over the 2.5 V target.
```

**Trimming:** Use a 200 Ω fixed resistor in series with a 50 Ω trimpot at RG. Adjust the trimpot at calibration to set the exact gain. Alternatively, accept the 1.2% gain error and correct it in firmware via a gain calibration coefficient.

### Part B — Noise Analysis

**Identifying noise sources:**

```
1. INA input voltage noise (en = 8 nV/√Hz, fc = 20 Hz)
2. INA input current noise (in = 0.5 pA/√Hz, white)
3. Bridge source resistance thermal noise
```

**Source impedance:**

The differential source impedance seen by the INA inputs is:

```
V+ input sees: R1 || R3 = 5k || 5k = 2.5 kΩ to Vexc and GND
V– input sees: R2 || R4 = 5k || 5k = 2.5 kΩ

Differential source resistance: 2.5 kΩ + 2.5 kΩ = 5 kΩ
```

**Noise calculations (bandwidth 0.01 Hz to 500 Hz):**

For 1/f + white noise, integrated from f_L to f_H:

```
V_rms = e_n_white × sqrt(fc × ln(f_H/f_L) + (f_H - f_L))

1. INA voltage noise:
   e_n_white = 8 nV/√Hz, fc = 20 Hz, f_L = 0.01 Hz, f_H = 500 Hz

   V_INA_rms = 8 × sqrt(20 × ln(500/0.01) + (500 - 0.01))
             = 8 × sqrt(20 × ln(50000) + 499.99)
             = 8 × sqrt(20 × 10.82 + 499.99)
             = 8 × sqrt(216.4 + 499.99)
             = 8 × sqrt(716.4)
             = 8 × 26.77
             = 214 nV RMS

2. INA current noise × source impedance:
   in = 0.5 pA/√Hz (white noise only — current noise 1/f not specified, assume white)
   in × Rsource = 0.5e-12 × 5000 = 2.5 nV/√Hz (referred to input)

   V_in_rms = 2.5 × sqrt(500 - 0.01) ≈ 2.5 × sqrt(500) = 2.5 × 22.4 = 56 nV RMS

3. Bridge thermal noise (5 kΩ differential source):
   e_Rsource = sqrt(4 × 1.38e-23 × 300 × 5000) = sqrt(8.28e-17) = 9.1 nV/√Hz

   V_bridge_rms = 9.1 × sqrt(500) = 9.1 × 22.4 = 204 nV RMS

Total input-referred noise:
  e_total = sqrt(214² + 56² + 204²) nV RMS
          = sqrt(45,796 + 3,136 + 41,616) nV RMS
          = sqrt(90,548) nV RMS
          = 301 nV RMS
```

**SNR at full-scale input:**

```
Full-scale input signal: ±10 mV peak = 10 mV / sqrt(2) = 7.07 mV RMS
  (sinusoidal — but for a step/DC measurement, use full scale directly)

For DC measurement accuracy: use peak error model:
  Full scale: 10 mV
  Noise (RMS): 0.301 µV

  SNR = 20 × log(10 mV / 0.301 µV) = 20 × log(33,222) = 90.4 dB

  ENOB = (90.4 - 1.76) / 6.02 = 14.7 bits
```

The noise performance is approximately 14.7 bits effective — suitable for a 16-bit ADC system, where the front-end noise does not limit the overall resolution below the ADC's capability.

### Part C — CMRR Requirement

**ADC LSB calculation:**

```
16-bit ADC, 5 V range:
  1 LSB = 5 V / 65536 = 76.3 µV

  0.5 LSB = 38.2 µV
```

**Required CMRR:**

```
Common-mode variation: ΔVcm = ±0.5 V → 1 V peak-to-peak worst case

The INA output must show < 0.5 LSB of error from this CM variation:
  At the INA output: 38.2 µV maximum error at gain = 253
  Input-referred CM error: 38.2 µV / 253 = 0.151 µV

  CMRR required = 20 × log(ΔVcm / V_cm_error)
                = 20 × log(0.5 V / 0.151 µV)
                = 20 × log(3,311,258)
                = 130.4 dB
```

This is an extremely stringent CMRR requirement. Check against the INA datasheet.

**Real INA CMRR at gain = 253:**

INA128 datasheet specifies CMRR at gain = 100: typically 110 dB (minimum), at DC. At gain = 253, CMRR is typically slightly better (110-120 dB) due to the larger differential gain.

**Conclusion:** The INA128 at gain = 253 falls about 10 dB short of the 130 dB requirement at DC. Options:

1. **Reduce the CM variation:** Add a low-noise LDO to power the bridge excitation, reducing the supply noise from ±0.5 V to ±50 mV. This reduces the CMRR requirement by 20 dB to 110 dB — achievable.

2. **Use a precision INA with higher CMRR:** AD8221 specifies 130 dB minimum CMRR at gain = 1000. At gain = 253, slightly lower but closer to the requirement.

3. **Accept the error and calibrate:** If the CM variation is deterministic (power supply frequency), notch-filter the CM noise on the excitation supply.

In practice, option 1 (precision voltage reference for bridge excitation) is the standard approach in precision sensor design. The excitation supply is always a precision voltage reference, not directly from a noisy system supply.

### Part D — Output Offset to Map ±2.5 V Swing to 0-5 V

**Approach:**

The INA output swings ±2.5 V relative to an internal reference. To shift this to 0-5 V for the ADC:

Connect the INA's VREF pin to 2.5 V (mid-rail). The VREF pin sets the output midpoint: Vout = Gain × Vdiff + VREF.

```
INA with VREF = 2.5 V:
  At Vdiff = -10 mV: Vout = 253 × (-10 mV) + 2.5 V = -2.53 + 2.5 = -0.03 V ≈ 0 V
  At Vdiff = 0:       Vout = 0 + 2.5 V = 2.5 V
  At Vdiff = +10 mV: Vout = +2.53 + 2.5 = 5.03 V ≈ 5 V
```

**2.5 V reference generation:**

```
Method 1 — Resistor divider from 5 V (adequate for moderate CMRR):
  Two 10 kΩ resistors from 5 V to GND. Midpoint = 2.5 V.
  Buffer with a unity-gain op-amp to provide low-output impedance for VREF pin.
  Source impedance of 10 kΩ divider: 5 kΩ. The buffer op-amp eliminates this.

Method 2 — Precision 2.5 V reference:
  Use a precision 2.5 V voltage reference IC (e.g., REF25, ADR2520: 0.1% initial
  accuracy, 10 ppm/°C). This decouples the output reference from supply noise,
  ensuring the midpoint is accurate and stable.

  Preferred for the high-accuracy system designed here.

Method 3 — Internal VREF from ADC:
  Some ADC ICs provide a precision midpoint reference output. If the ADC's REFOUT/2
  pin is available, use it directly as VREF for the INA — this creates a ratiometric
  reference where both the INA output and ADC reference track the same reference voltage.
```

**Complete output circuit:**

```
INA VREF pin ← 2.5 V precision reference (REF2025 or equivalent)
INA output → direct to ADC input (no additional amplification needed)

ADC code mapping:
  Code 0:     0 V   = -10 mV input (0 MPa)
  Code 32768: 2.5 V = 0 mV input (null balance point)
  Code 65535: 5 V   = +10 mV input (1 MPa full scale)
```

---

## Key Takeaways

- At 5 kΩ bridge source impedance, the INA voltage noise and bridge thermal noise are comparable in magnitude — neither dominates overwhelmingly. This is an important regime where both terms must be calculated
- CMRR requirements become very stringent (>120 dB) when the common-mode voltage is large (2.5 V) and the required accuracy is high (0.5 LSB at 16 bits = 38 µV). The primary solution is to regulate the bridge excitation with a precision reference, not to rely solely on INA CMRR
- The VREF pin of an INA is the correct way to implement output offset — it shifts the output midpoint without adding a separate summing stage. It also creates a pseudo-ratiometric reference if the same voltage references both VREF and the ADC
- RG tolerancing matters: a 1% error in RG gives a 1% gain error at high gain settings. Either trim the gain resistor at calibration or correct the gain error in firmware using a stored calibration coefficient

---

## Interview Notes

Instrumentation amplifier problems are a staple of analogue hardware interviews. This problem tests the full design chain, from sensor characteristics to ADC interface. Key discriminators between strong and weak answers:

- **Strong:** Knows the CMRR requirement comes from the bridge excitation noise, not just from a generic INA spec — and proposes regulating the excitation supply as the primary fix
- **Strong:** Calculates noise budget rigorously — identifies that bridge source resistance thermal noise is comparable to INA voltage noise and must be included
- **Strong:** Uses the VREF pin to shift the INA output, not a summing amplifier (adding a summing stage would add noise and complexity)
- **Weak:** Specifies an INA gain of 250 without checking that the gain resistor calculation yields a standard value, or without verifying single-supply output swing
- **Weak:** Forgets to calculate the gain resistor value, just stating "set the gain to 250"

A follow-up question often asked: "How would you implement a zero-pressure calibration?" Answer: apply zero pressure, read the ADC code, and store it as the zero offset in firmware. Subtract this from all subsequent readings. This removes the offset from bridge imbalance, INA Vos, and reference inaccuracy in a single calibration step.
