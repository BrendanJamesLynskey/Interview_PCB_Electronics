# Problem 02: ADC SNR Budget

## Problem Statement

You are specifying an ADC for a vibration monitoring system. The sensor is a MEMS accelerometer with:

- Sensitivity: 100 mV/g
- Noise floor: 50 µg/√Hz (accelerometer self-noise)
- Output range: ±3 g (±300 mV differential)
- Output impedance: 200 Ω

The system requirements:

- Measurement bandwidth: 10 Hz to 10 kHz
- Dynamic range: > 80 dB (must resolve 50 µg signals in the presence of 3 g full-scale signals)
- Sample rate: 25 kSPS
- Single 3.3 V supply

**Part A:** Calculate the minimum required ENOB for the ADC to not degrade the system dynamic range below 80 dB. What commercial ADC bit count is appropriate?

**Part B:** The candidate ADC is a 16-bit SAR with the following specs:
- Measured SNR: 90 dB at 1 kHz input
- THD: -95 dBc at 1 kHz
- Input range: ±2.5 V differential (5 V span)
- Input capacitance (sampling): 20 pF

Determine SINAD, ENOB, and SFDR for this ADC. Does it meet the 80 dB dynamic range requirement?

**Part C:** Design the anti-aliasing filter for this ADC and accelerometer combination (25 kSPS, 10 kHz signal bandwidth). Specify the filter order, topology, and component values.

**Part D:** The accelerometer output is ±300 mV but the ADC input range is ±2.5 V. Design the gain stage to match the signal to the ADC. Specify the amplifier gain, the resulting ADC utilisation, and the impact on SNR.

---

## Solution Approach

### Part A — Required ADC Dynamic Range and Bit Count

**System dynamic range requirement:**

```
Dynamic range = maximum signal / minimum resolvable signal
             = 3 g / 50 µg = 60,000 → 95.6 dB

But the system requirement specifies > 80 dB dynamic range.
The accelerometer has a noise floor of 50 µg/√Hz.

Integrated sensor noise over 10 Hz to 10 kHz bandwidth:
  V_sensor_noise = 50 µg/√Hz × sqrt(10000 - 10) ≈ 50 × sqrt(9990) = 4997 µg RMS
               = 4.997 mg RMS ≈ 5 mg RMS
  Voltage: 5 mg × 100 mV/g = 0.5 mV RMS
```

**Required ADC noise floor:**

The ADC quantisation noise must not significantly degrade the sensor noise floor. A common engineering criterion: ADC noise < 1/3 of sensor noise (to contribute < 0.5 dB of noise figure degradation):

```
ADC noise floor < 0.5 mV / 3 = 0.167 mV RMS

In terms of dynamic range: (300 mV peak) / (0.167 mV RMS × √2) = 1273 : 1
Equivalent: 20 × log(1273) = 62 dB dynamic range from quantisation alone.

Combined sensor + ADC noise: sqrt(0.5² + 0.167²) mV RMS = 0.527 mV RMS
Dynamic range: 300 mV / 0.527 mV = 569 → 55 dB

This is below the 80 dB requirement — a problem arises because the sensor noise
itself is quite large when integrated over 10 kHz bandwidth.

Revisit: the 80 dB requirement applies to the system's ability to resolve a
50 µg/√Hz spectral component within a narrow band (e.g., 1 Hz resolution
bandwidth):
  Min detectable signal in 1 Hz BW: 50 µg/√Hz × sqrt(1 Hz) = 50 µg = 0.005 mV
  Max signal: 3 g = 300 mV
  Dynamic range: 20 × log(300 mV / 0.005 mV) = 95.6 dB
```

**ADC ENOB requirement for 80 dB system DR:**

```
Required ENOB: (80 - 1.76) / 6.02 = 13.0 bits

This is the minimum. The ADC must also not limit the system's 95 dB dynamic range:
  ENOB ≥ (95.6 - 1.76) / 6.02 = 15.6 bits

Conclusion: a 16-bit ADC (ideal ENOB = 15.9 bits) is the appropriate choice.
```

### Part B — ADC SINAD, ENOB, and SFDR Assessment

**Calculating SINAD from SNR and THD:**

```
SNR  = 90 dB (signal-to-noise ratio)
THD  = -95 dBc (total harmonic distortion relative to carrier)

Converting to linear power:
  Noise power = 10^(-90/10) = 10^-9 (normalised to signal power = 1)
  THD power   = 10^(-95/10) = 3.16×10^-10

SINAD = signal power / (noise power + distortion power)
     = 1 / (10^-9 + 3.16×10^-10)
     = 1 / 1.316×10^-9
     = 759.9×10^6

SINAD_dB = 10 × log(759.9×10^6) = 88.8 dB
```

**ENOB:**

```
ENOB = (SINAD_dB - 1.76) / 6.02
     = (88.8 - 1.76) / 6.02
     = 87.04 / 6.02
     = 14.46 bits
```

**SFDR:**

SFDR is the ratio of the fundamental to the largest spurious tone. With THD = -95 dBc, the fundamental-to-largest-harmonic ratio is at least 95 dBc, so SFDR ≥ 95 dBc (assuming the worst harmonic is the largest spur).

**Assessment against 80 dB requirement:**

```
System ENOB = 14.46 bits → equivalent dynamic range = 6.02 × 14.46 + 1.76 = 88.8 dB

88.8 dB > 80 dB — the ADC meets the system requirement with 8.8 dB margin.

The ENOB at 1 kHz is 14.46 bits. This must also be verified at 10 kHz
(the maximum signal frequency). ENOB typically degrades with frequency
due to aperture jitter. Verify in the datasheet at 10 kHz.
```

### Part C — Anti-Aliasing Filter Design

**Requirements:**

```
Sample rate: 25 kSPS → Nyquist: 12.5 kHz
Signal bandwidth: 10 kHz
Guard band: 12.5 - 10 = 2.5 kHz
ADC dynamic range: 88.8 dB

The filter must attenuate signals above 12.5 kHz by ≥ 88.8 dB.
Frequency ratio at stopband edge: 12.5 kHz / 10 kHz = 1.25
```

**Filter order for Butterworth at f_pass = 10 kHz, f_stop = 12.5 kHz:**

```
Butterworth attenuation: A(r) = 10 × log(1 + r^(2N))  where r = f/fc

Set fc at corner frequency, then check stopband.

For fc = 10 kHz, at f = 12.5 kHz (r = 1.25):
  Required: 10 × log(1 + 1.25^(2N)) ≥ 88.8 dB
  1 + 1.25^(2N) ≥ 10^8.88 = 7.59×10^8
  1.25^(2N) ≈ 7.59×10^8
  2N × log(1.25) ≥ log(7.59×10^8)
  2N × 0.0969 ≥ 8.88
  N ≥ 45.8 → N = 46!
```

A 46th-order Butterworth is completely impractical. The problem is the very narrow guard band (1.25:1 ratio). Solutions:

**Solution 1 — Increase sample rate:**

If the sample rate is increased to 100 kSPS, the Nyquist becomes 50 kHz. Stopband ratio = 50/10 = 5. Required order:

```
1.25^(2N) ≥ 7.59×10^8 → replace with:
5^(2N) ≥ 7.59×10^8
2N × log(5) ≥ 8.88
2N × 0.699 ≥ 8.88
N ≥ 6.35 → 7th order Butterworth
```

A 7th-order filter is practical (four cascaded stages: 2-2-2-1). This is the preferred solution for high-resolution, low-aliasing ADC applications — oversample and decimate.

**Solution 2 — Use delta-sigma ADC:**

A delta-sigma ADC at 100 kSPS output rate with OSR = 256 oversamples internally at 25.6 MSPS. The analog anti-aliasing requirement reduces to attenuating signals above 25.6/2 = 12.8 MHz — a simple single-pole RC filter suffices.

**Practical design (Solution 1 with 7th-order Butterworth, fc = 10 kHz, 100 kSPS):**

```
7th-order active Butterworth:
  Implement as: three 2nd-order Sallen-Key stages + one 1st-order RC

  Pole frequencies and Q values (Butterworth 7th order):
  Stage 1: f01 = fc × 0.976 = 9.76 kHz, Q1 = 0.556  (real pole scaled to complex)
  Stage 2: f02 = fc × 0.879 = 8.79 kHz, Q2 = 0.802
  Stage 3: f03 = fc × 0.649 = 6.49 kHz, Q3 = 2.247
  Stage 4: 1st-order RC at fc × 1.0 = 10 kHz

  (Standard Butterworth pole table values — from filter design handbook)

Component values for Stage 1 (example, Sallen-Key, equal component):
  C = 1 nF (choose), R = 1/(2π × f01 × C) = 1/(2π × 9760 × 1e-9) = 16.3 kΩ → 16.2 kΩ E96
  Q adjustment: set buffer gain K = 3 - 1/Q1 = 3 - 1/0.556 = 1.20
  K = 1 + Rb/Ra → Rb/Ra = 0.20 → Ra = 10 kΩ, Rb = 2 kΩ
```

**Op-amp requirement for 7th-order filter:**

Each stage requires a stable, low-noise op-amp with GBW >> 10× noise gain × f0. For the stage with Q = 2.247, noise gain = 3 - 1/2.247 = 2.555. Required GBW ≥ 10 × 2.555 × 10 kHz = 255 kHz → use op-amp with GBW ≥ 10 MHz for margin.

### Part D — Gain Stage for Sensor-to-ADC Matching

**Gain requirement:**

```
Accelerometer output: ±300 mV (at ±3 g)
ADC input range: ±2.5 V

Gain = 2.5 V / 300 mV = 8.33

Use gain of 8 (exact with standard resistors) or 8.33 (requiring a trimpot).
For gain = 8: ADC utilisation = 300 mV × 8 / 2500 mV = 96% → good.
```

**Gain stage design (non-inverting amplifier):**

```
G = 1 + Rf/Ri = 8

Rf/Ri = 7

Select Ri = 1 kΩ, Rf = 7 kΩ (use 6.81 kΩ E96 + 196 Ω E96 in series = 7.006 kΩ)
Or: Ri = 10 kΩ, Rf = 70 kΩ (use 68 kΩ + 2 kΩ = 70 kΩ)

Noise gain (unity for buffer) is also 8 — the op-amp voltage noise is amplified 8×.

Op-amp noise requirement:
  Accelerometer noise: 50 µg/√Hz × 100 mV/g = 5 µV/√Hz (at output of sensor)
  
  Op-amp voltage noise referred to output: en_opamp × Gain = en × 8
  For the op-amp to contribute < 1/3 of sensor noise at the gain stage output:
  en × 8 ≤ 5 µV/√Hz / 3 = 1.67 µV/√Hz
  en ≤ 1.67 µV / 8 = 0.208 µV/√Hz = 208 nV/√Hz

  This is a very relaxed requirement — standard precision op-amps (5-30 nV/√Hz)
  easily meet this. Even a general-purpose LM358 (40 nV/√Hz) meets it.
```

**ADC input settling:**

With a 20 pF ADC sampling capacitor and 10 µs acquisition window at 100 kSPS:

```
Driving impedance from gain stage: essentially 0 Ω (closed-loop op-amp output)
Plus a 33 Ω series protection/isolation resistor (recommended for op-amps
driving ADC sampling inputs): 33 Ω + 20 pF → RC = 0.66 ns

Settling to 16-bit accuracy requires 11 × τ = 7.3 ns << 10 µs acquisition time.
The source impedance constraint is easily met.
```

**SNR impact of gain stage:**

```
Op-amp noise referred to ADC input (after gain = 8):
  e_opamp_total = 20 nV/√Hz × 8 = 160 nV/√Hz (at ADC input)

Sensor noise referred to ADC input:
  5 µV/√Hz × 8 = 40 µV/√Hz (at ADC input)

The op-amp contributes only 0.4% of the noise variance — negligible.
The sensor noise dominates the noise budget, as intended.

ADC quantisation noise referred to ADC input:
  For 16-bit, ±2.5 V: LSB = 5 V / 65536 = 76.3 µV
  Quantisation noise RMS = LSB / sqrt(12) = 22 µV

Sensor noise at ADC input (1 Hz bandwidth): 40 µV/√Hz → 40 µV RMS (in 1 Hz BW)
ADC quantisation noise: 22 µV RMS
  Combined: sqrt(40² + 22²) = 45.7 µV RMS

The ADC quantisation is 55% of the sensor noise in a 1 Hz bandwidth. This limits
performance — a 24-bit ADC would contribute only 2.2 µV RMS quantisation noise,
well below the 40 µV sensor noise floor.
```

---

## Key Takeaways

- ENOB is the correct figure of merit for ADC dynamic range comparison — it integrates SNR and THD into a single number that directly gives effective bits
- For high-resolution applications with narrow guard bands, a sharp analog anti-aliasing filter is impractical — oversample and decimate (use a higher sample rate or a delta-sigma ADC with internal oversampling)
- The gain stage should be designed so that the sensor noise dominates the noise budget at the ADC input — if ADC quantisation noise is larger than sensor noise, the ADC is underspecified
- Op-amp noise in the gain stage is typically negligible when the stage gain is 10× or less and the source (sensor) noise floor is >> 10 nV/√Hz — verify this quickly with a noise budget before spending time on elaborate op-amp selection

---

## Interview Notes

ADC selection and SNR budget problems appear frequently in mixed-signal hardware interviews. The key points interviewers check:

1. Whether you can work through ENOB from SNR + THD (not just quote ENOB from the datasheet)
2. Whether you recognise that a narrow guard band (25 kSPS with 10 kHz signal bandwidth) makes a practical anti-aliasing filter impractical — the correct response is to increase the sample rate
3. Whether you check that the sensor noise dominates the ADC noise floor (if the ADC is underspecified, you are wasting dynamic range that the sensor provides)
4. Whether you verify the ADC input settling constraint — a common oversight that causes mysterious first-article failures

A senior candidate also mentions: "I would validate the ENOB at 10 kHz input frequency, not just at 1 kHz — ENOB degrades with input frequency and the datasheet spec at 1 kHz may not represent the worst case for a 10 kHz signal."
