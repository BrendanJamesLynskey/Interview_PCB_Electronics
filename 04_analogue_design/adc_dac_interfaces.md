# ADC/DAC Interfaces

## Prerequisites
- Sampling theory: Nyquist theorem, aliasing
- Basic op-amp circuits (see `op_amp_circuits.md`)
- Signal-to-noise ratio, decibels
- PCB layout for mixed-signal designs (see `../02_pcb_layout/return_current_paths.md`)

---

## Concept Reference

### ADC Static Performance Parameters

#### Resolution and LSB Size

The fundamental unit of a converter is the Least Significant Bit (LSB):

```
LSB = V_FSR / 2^N

Where:
  V_FSR = full-scale range (e.g., 5 V, 4.096 V)
  N     = number of bits

For a 12-bit ADC, 4.096 V range:
  LSB = 4.096 / 4096 = 1 mV/LSB

For a 16-bit ADC, 4.096 V range:
  LSB = 4.096 / 65536 = 62.5 µV/LSB
```

#### Integral Non-Linearity (INL)

INL measures the worst-case deviation of the actual ADC transfer function from a straight line through the two endpoints:

```
INL = max|V_actual(code) - V_ideal(code)| / LSB

A +1 LSB INL means one code maps to a voltage 1 LSB higher than expected.

Typical ADC INL:
  General purpose 12-bit SAR: ±1 to ±4 LSB
  Precision 16-bit SAR:       ±1 to ±4 LSB (at 16-bit = ±62-250 µV)
  High-performance delta-sigma: ±0.5 to ±2 LSB
```

INL cannot be removed by calibration — it is the irreducible nonlinearity.

#### Differential Non-Linearity (DNL)

DNL measures the deviation of each code width from the ideal 1 LSB:

```
DNL(n) = (actual code width for code n) - 1 LSB

DNL = -1 LSB → "missing code" (converter skips from n-1 to n+1)
DNL > -1 LSB → converter is monotonic (no missing codes)

Typical: ±0.5 to ±1 LSB for 12-16 bit converters
```

Poor DNL generates harmonic distortion even when INL is acceptable.

#### Offset and Gain Error

```
Offset error: fixed shift of entire transfer curve → removed by single-point calibration
Gain error:   slope error growing with amplitude → removed by two-point calibration
INL/DNL:      residual after calibration — fundamental accuracy floor
```

### ADC Dynamic Performance Parameters

#### Signal-to-Noise Ratio (SNR)

```
Ideal SNR = 6.02 × N + 1.76 dB   (N = number of bits, sinusoidal input)

12-bit ideal: 74.0 dB
16-bit ideal: 98.1 dB
24-bit ideal: 146.0 dB (theoretical only — far exceeds thermal limits)
```

Actual SNR is lower due to internal noise, reference noise, and aperture jitter.

#### Effective Number of Bits (ENOB)

ENOB converts actual measured SNR back to equivalent resolution:

```
ENOB = (SNR_measured - 1.76) / 6.02

Example: 12-bit ADC with measured SNR = 68 dB:
  ENOB = (68 - 1.76) / 6.02 = 11.0 bits

The converter resolves 11 effective bits despite having 12-bit resolution.
```

ENOB is the primary figure of merit for comparing ADC dynamic performance.

#### Spurious Free Dynamic Range (SFDR)

```
SFDR = 20 × log(V_fundamental / V_largest_spur)   [dBc]

Typical values:
  General purpose 12-bit:    75-80 dBc
  Precision 16-bit SAR:      90-100 dBc
  High-speed 12-bit (GSPS):  65-70 dBc
```

SFDR matters most for spectral analysis, RF sampling, and audio applications where spurs must not appear above the signal of interest.

#### Total Harmonic Distortion (THD)

THD and THD+N (noise included) characterise harmonic content. SINAD (Signal-to-Noise and Distortion Ratio) relates directly to ENOB:

```
SINAD = signal power / (harmonic distortion + noise)
ENOB  = (SINAD - 1.76) / 6.02
```

### ADC Architectures

```
Architecture     Resolution      Speed range       Application
-----------      ----------      -----------       -----------
SAR              12-18 bit       10 kSPS-5 MSPS    Precision measurement,
                                                    data acquisition
Delta-Sigma      16-32 bit       10 SPS-10 MSPS    Instrumentation, audio,
                                                    slow precision sensing
Flash            6-10 bit        >1 GSPS           Oscilloscopes, radar
Pipeline         10-16 bit       10-500 MSPS       Video, wideband receivers
```

**SAR (Successive Approximation Register):**

The SAR performs a binary search. At each clock cycle it sets one bit, compares, then moves to the next. After N cycles, the conversion is complete.

```
For a 12-bit SAR at 1 MSPS:
  Requires 12 internal clock cycles per sample (minimum 12 MHz clock)
  Input must be stable during the full 1 µs conversion window
  → Requires a sample-and-hold (usually internal) or an input buffer
    with settling time < one acquisition window
```

The SAR input is typically a switched-capacitor sample-and-hold. This creates charge kickback on the driving source at the sampling instant. Sources with impedance > 100-200 Ω must be buffered to allow the sample capacitor to settle within the acquisition window.

**Delta-Sigma:**

A delta-sigma oversamples at OSR × the output rate, using a 1-bit or multi-bit quantiser, and applies a digital decimation filter. Noise shaping pushes quantisation noise out of the signal band.

```
Oversampling benefit (per octave): +3 dB SNR
Noise shaping benefit: OSR^(2L+1) for an L-th order modulator

1st order, OSR=256: SNR improvement = 10×log(256^3) = 72 dB above 1-bit
2nd order, OSR=256: SNR improvement = 10×log(256^5) = 120 dB above 1-bit
```

The digital decimation filter introduces group delay. Step inputs require waiting for the filter to settle before the output is valid — makes delta-sigma unsuitable for fast multiplexed channel switching.

### DAC Performance Parameters

The same static accuracy parameters apply (INL, DNL, offset, gain). Dynamic DAC parameters:

```
Settling time: time for output to reach final value within ±0.5 LSB
              after a code step. Critical for waveform generation and
              multiplexed outputs.

Glitch impulse: transient spike at major code transitions (e.g., 0111...1 → 1000...0).
               Caused by differential switching timing. Measured in nV·s.
               Mitigated with a deglitch circuit (sample-and-hold after the DAC).

Output noise spectral density: limits achievable dynamic range. Filter output
               to the signal bandwidth to reduce integrated noise.
```

### Anti-Aliasing Filter Design

The Nyquist theorem: input must be band-limited to below fs/2.

```
Alias frequency: f_alias = |f_signal - k × fs|

Example: fs = 10 kHz, f_signal = 8 kHz:
  f_alias = |8000 - 10000| = 2000 Hz (appears as genuine 2 kHz component)
```

**Filter specification:**

The filter must attenuate signals at fs/2 to below the converter's noise floor:

```
Required attenuation at fs/2 ≥ converter dynamic range (ENOB × 6.02 + 1.76 dB)

For a 12-bit ADC (74 dB) with fs = 100 kSPS, signal bandwidth = 10 kHz:
  Stopband: 50 kHz
  Passband: 10 kHz
  Frequency ratio: 50/10 = 5
  Required roll-off: 74 dB over this frequency range

  4th-order Butterworth at fc = 15 kHz provides ~68 dB at 50 kHz — close.
  5th-order Chebyshev (0.5 dB ripple) at fc = 15 kHz provides ~80 dB — sufficient.
```

Delta-sigma ADCs need only a simple RC filter for anti-aliasing because oversampling places the first alias at OSR × (output data rate), far above the signal band.

---

## Tier 1 — Fundamentals

### Question F1
**A 12-bit ADC has a full-scale range of 3.3 V. What is 1 LSB in mV? If the ADC has INL = ±2 LSB, what is the maximum absolute voltage error for any given code? Can calibration remove this error?**

**Answer:**

```
1 LSB = FSR / 2^N = 3.3 V / 4096 = 0.806 mV ≈ 0.8 mV

Maximum absolute voltage error from INL = ±2 LSB:
  ±2 × 0.806 mV = ±1.61 mV
```

**Calibration:** Single-point (offset) and two-point (offset + gain) calibration removes offset and gain error but does NOT remove INL. INL is the nonlinearity remaining after the best-fit straight line is subtracted. It is the intrinsic nonlinearity of the converter and sets the floor for absolute accuracy regardless of calibration.

For a 3.3 V full-scale instrument, ±1.61 mV corresponds to ±0.049% full-scale accuracy — a fundamental limit of this converter in any calibrated system.

---

### Question F2
**Define ENOB. A 16-bit ADC measures 92 dB SNR at 1 kHz. Calculate ENOB and state what this means for the system designer.**

**Answer:**

```
ENOB = (SNR_measured - 1.76) / 6.02
     = (92 - 1.76) / 6.02
     = 90.24 / 6.02
     = 15.0 bits
```

Ideal 16-bit SNR = 6.02 × 16 + 1.76 = 98.1 dB. The measured 92 dB is 6.1 dB below ideal, corresponding to approximately 1 bit of effective resolution lost to noise and distortion.

**Meaning:** The converter effectively resolves 15 bits of signal at 1 kHz. For the system designer:

1. The system cannot achieve 16-bit accuracy using this converter alone — it delivers 15-bit accuracy at 1 kHz.
2. ENOB typically degrades with increasing input frequency. Verify ENOB at the maximum frequency of interest, not just at 1 kHz.
3. If 16-bit accuracy is required, either select a converter with higher ENOB, reduce noise by averaging/oversampling, or revise the accuracy specification.

---

### Question F3
**What is aliasing and why is it harmful? Give a numerical example with fs = 5 kHz and a 4.2 kHz input. What is the circuit remedy?**

**Answer:**

Aliasing occurs when a signal above the Nyquist frequency (fs/2) is sampled, appearing in the digitised output at a lower, spurious frequency. The aliased signal is indistinguishable from a real in-band signal.

**Numerical example:**

```
Sampling rate: fs = 5 kHz → Nyquist = 2.5 kHz
Input signal: 4.2 kHz (above Nyquist)

f_alias = |f_signal - fs| = |4.2 - 5.0| = 0.8 kHz

The 4.2 kHz input appears as a 0.8 kHz tone in the digital output.
If the system is measuring 0-2.5 kHz signals, a 0.8 kHz alias corrupts any
genuine signal near 0.8 kHz — the error cannot be removed in digital processing.
```

**Why harmful:** Once aliased, the spurious frequency cannot be separated from genuine signal. The only remedy is to prevent it before sampling.

**Circuit remedy:** An anti-aliasing low-pass filter placed before the ADC input, with its cutoff frequency set below fs/2. The filter attenuates all signals above fs/2 to below the ADC noise floor before they reach the sampling input. The filter must be in the analog domain — digital filtering after sampling cannot recover aliased information.

---

## Tier 2 — Intermediate

### Question I1
**A SAR ADC has an input sample capacitor of 30 pF and requires the input to settle within 0.5 LSB in a 50 ns acquisition window. The converter is 12-bit. What is the maximum source resistance that can drive the ADC directly, without an input buffer? If the sensor source impedance is 10 kΩ, what circuit is required?**

**Answer:**

**Maximum settling condition:**

The input circuit is a first-order RC: source impedance Rs drives the sample capacitor Cs = 30 pF. The input must settle to within 0.5 LSB = 1/2^(N+1) of final value.

```
Settling condition: V_final × (1 - exp(-t_acq / (Rs × Cs))) ≥ V_final × (1 - 0.5 LSB/V_final)

Time constant τ = Rs × Cs must satisfy:
  exp(-t_acq / τ) ≤ 0.5 / 2^N   (error ≤ 0.5 LSB as fraction of FSR)

For N = 12, t_acq = 50 ns:
  exp(-50 ns / τ) ≤ 0.5 / 4096 = 1.22e-4
  -50 ns / τ ≤ ln(1.22e-4) = -9.01
  τ ≤ 50 ns / 9.01 = 5.55 ns

  Rs_max = τ / Cs = 5.55 ns / 30 pF = 185 Ω
```

The maximum source resistance is **approximately 185 Ω**.

**With 10 kΩ source impedance:**

A unity-gain op-amp buffer (voltage follower) is required between the sensor and the ADC. The buffer presents a very low output impedance (< 1 Ω closed-loop) to the ADC sampling input, while the sensor drives the op-amp input at high impedance (virtually no current drawn).

Buffer selection criteria:
- Input impedance > 10× source impedance (to minimise loading): > 100 kΩ → use CMOS input op-amp
- Output current drive: must supply peak charging current to Cs within t_acq
  I_peak = Cs × V_FSR / t_acq = 30 pF × 5 V / 50 ns = 3 mA → standard op-amp easily handles this
- Settling time of buffer itself: the buffer must settle within the ADC acquisition window
  → verify the slew rate and bandwidth specifications of the selected op-amp

---

### Question I2
**Compare SAR and delta-sigma ADCs for the following two applications: (a) 8-channel multiplexed temperature measurement at 1 SPS per channel; (b) a precision oscilloscope front end sampling at 500 MSPS with 10-bit resolution. Justify the architecture choice for each.**

**Answer:**

**Application (a) — 8-channel temperature at 1 SPS:**

Winner: **Delta-sigma.**

```
Reasons:
  (1) Resolution: thermocouples and RTDs require 16-20 bit resolution to resolve
      temperature to 0.1°C or better. Delta-sigma delivers this naturally.
  (2) Bandwidth: 1 SPS per channel is very slow — delta-sigma's slow but precise
      architecture is ideal.
  (3) Anti-aliasing: inherent oversampling makes the anti-aliasing filter trivial.
  (4) Integration: many delta-sigma ADCs integrate multiplexers, PGA, reference,
      and CRC for sensor interfaces (e.g., ADS1248, MAX31865).

Caution with multiplexing:
  After switching channels, allow the delta-sigma filter to settle (1-10 conversions
  at the output data rate). At 1 SPS per channel with 8 channels: the system can
  afford to discard the first 1-2 conversions after each channel switch if the
  output data rate is 10-20 SPS.
```

**Application (b) — 500 MSPS, 10-bit oscilloscope:**

Winner: **Flash or pipeline ADC.**

```
Reasons:
  (1) Speed: delta-sigma is fundamentally limited to ~10 MSPS output rate. SAR tops out
      at ~5 MSPS for 12-bit. Neither can approach 500 MSPS.
  (2) 10-bit resolution at 500 MSPS requires a pipeline or flash architecture:
      Pipeline: 10-14 bit at 100-500 MSPS (e.g., AD9648, LTC2209)
      Flash: 6-8 bit at >1 GSPS (e.g., HMCAD1511)
  (3) Pipeline ADC with 10-bit resolution at 500 MSPS is the standard choice for
      oscilloscopes in this range.
  (4) Dynamic specifications matter most: ENOB at 250 MHz signal frequency
      (Nyquist), SFDR (spurious tones corrupt waveform display).
      Pipeline converters have ENOB of 8-9 bits at high input frequencies.
```

---

## Tier 3 — Advanced

### Question A1
**A 16-bit SAR ADC running at 250 kSPS has a clock jitter of 50 ps RMS. Calculate the SNR degradation due to jitter at: (a) 100 Hz input frequency; (b) 100 kHz input frequency. At what frequency does jitter become the dominant noise source, assuming the ideal quantisation noise floor of a 16-bit converter?**

**Answer:**

**SNR degradation due to aperture jitter:**

```
SNR_jitter = -20 × log(2π × f_in × σ_jitter)

Ideal 16-bit SNR (quantisation limited): 6.02 × 16 + 1.76 = 98.1 dB
```

**Part (a) — 100 Hz:**

```
SNR_jitter = -20 × log(2π × 100 × 50e-12)
           = -20 × log(3.14e-8)
           = -20 × (-7.5)
           = 150 dB

150 dB >> 98 dB — jitter is completely negligible at 100 Hz.
Quantisation noise dominates.
```

**Part (b) — 100 kHz:**

```
SNR_jitter = -20 × log(2π × 100e3 × 50e-12)
           = -20 × log(3.14e-5)
           = -20 × (-4.5)
           = 90 dB

90 dB < 98 dB — jitter is now the dominant limitation.
ENOB set by jitter: (90 - 1.76) / 6.02 = 14.6 bits
```

**Crossover frequency (where jitter = quantisation noise):**

```
Set SNR_jitter = 98.1 dB:
  98.1 = -20 × log(2π × f_cross × 50e-12)
  log(2π × f_cross × 50e-12) = -4.905
  2π × f_cross × 50e-12 = 1.245e-5
  f_cross = 1.245e-5 / (2π × 50e-12) = 39.6 kHz
```

Jitter dominates above approximately **40 kHz** for this clock quality. This is well below the 125 kHz Nyquist of the 250 kSPS ADC — confirming that even a moderate 50 ps jitter significantly limits 16-bit dynamic range at frequencies above 40 kHz.

**Design implication:** For precision AC measurements above 40 kHz with a 16-bit converter, the sample clock must have sub-10 ps RMS jitter. Use a crystal oscillator or a PLL-derived clock with a low-phase-noise VCO. Do not use a system microcontroller clock (typical jitter 100-500 ps RMS) to drive a precision ADC directly.

---

## Best Practices

1. Check ENOB at the highest frequency of interest — it degrades with frequency due to aperture jitter and bandwidth limits. The published "bits of resolution" is the DC ideal; ENOB at signal frequency determines actual AC performance.
2. Buffer SAR ADC inputs: if source impedance exceeds 100-200 Ω, add a unity-gain op-amp buffer to allow the internal sample capacitor to settle within the acquisition window.
3. Design anti-aliasing filters for the actual ENOB-derived SNR (not nominal bit count) at the signal frequency. A 12-bit ADC at high frequency may have only 10 ENOB, requiring only 60 dB of anti-aliasing attenuation.
4. Use delta-sigma ADCs for precision DC and audio-rate measurements; use SAR or pipeline for multi-channel acquisition and higher-speed requirements.
5. Clock the ADC from a low-jitter source. For precision 16-bit measurements above 10 kHz, clock jitter must be < 10 ps RMS.
6. Keep ADC VREF lines short and heavily decoupled; reference noise directly adds to conversion noise at full gain. A 100 nF + 10 µF directly at the VREF pin is the minimum decoupling.

---

## Related Topics

- [Op-Amp Circuits](op_amp_circuits.md)
- [Sensor Signal Conditioning](sensor_signal_conditioning.md)
- [Noise Analysis](noise_analysis.md)
- [Return Current Paths](../02_pcb_layout/return_current_paths.md)
- [Problem 02: ADC SNR](worked_problems/problem_02_adc_snr.md)
- [Problem 01: Instrumentation Amplifier](worked_problems/problem_01_instrumentation_amp.md)
