# Noise Analysis

## Prerequisites
- Signal-to-noise ratio concepts (see `adc_dac_interfaces.md`)
- Op-amp circuits: noise gain, bandwidth (see `op_amp_circuits.md`)
- Basic statistics: RMS, power spectral density, integration

---

## Concept Reference

### Fundamental Noise Sources

#### Thermal Noise (Johnson-Nyquist Noise)

Every resistor at a temperature above absolute zero generates voltage noise from random thermal motion of electrons. This is unavoidable — it is a thermodynamic consequence of resistor physics.

```
Thermal noise voltage spectral density:
  e_n = sqrt(4 × k_B × T × R)   [V/√Hz]

Where:
  k_B = Boltzmann constant = 1.38×10⁻²³ J/K
  T   = absolute temperature (K)
  R   = resistance (Ω)

At T = 300 K (27°C):
  e_n = sqrt(4 × 1.38e-23 × 300 × R)
      = sqrt(1.657e-20 × R)
      = 1.29e-10 × sqrt(R) V/√Hz

For common resistor values:
  R = 100 Ω:   e_n = 1.29 nV/√Hz
  R = 1 kΩ:    e_n = 4.07 nV/√Hz
  R = 10 kΩ:   e_n = 12.9 nV/√Hz
  R = 100 kΩ:  e_n = 40.7 nV/√Hz
  R = 1 MΩ:    e_n = 129 nV/√Hz
```

**RMS noise over a bandwidth BW:**

```
V_rms = e_n × sqrt(BW)

For R = 10 kΩ, BW = 10 kHz:
  V_rms = 12.9 nV/√Hz × sqrt(10,000 Hz)
        = 12.9 nV/√Hz × 100
        = 1.29 µV RMS
```

Thermal noise is white (flat spectrum) and Gaussian-distributed. Doubling the bandwidth increases noise power by 2× (voltage by sqrt(2)). Reducing temperature reduces noise — cryogenic amplifiers exploit this for extremely low-noise radio astronomy.

#### Shot Noise

Shot noise arises from the discrete nature of electric charge. Any current flowing across a junction (pn junction, vacuum tube) produces shot noise:

```
Current noise spectral density:
  i_n = sqrt(2 × q × I_DC)   [A/√Hz]

Where:
  q     = electron charge = 1.60×10⁻¹⁹ C
  I_DC  = DC current flowing through the junction

For a diode with I_DC = 1 mA:
  i_n = sqrt(2 × 1.60e-19 × 1e-3)
      = sqrt(3.20e-22)
      = 1.79e-11 A/√Hz = 17.9 pA/√Hz
```

Shot noise is most significant in:
- Bipolar transistor base current (IB creates shot noise i_n = sqrt(2qIB))
- Photodiode dark current amplification
- Low-leakage CMOS circuits (shot noise from ESD protection diode leakage)

For pure resistors (no junctions), shot noise does not apply — only thermal noise.

#### 1/f Noise (Flicker Noise, Pink Noise)

1/f noise is excess low-frequency noise whose power spectral density rises as 1/f (or 1/f² for voltage noise spectral density, depending on the model). It is present in all semiconductor devices and some resistors.

```
1/f noise model:
  e_n(f) = e_n_white × sqrt(f_c / f)   [V/√Hz]

Where:
  e_n_white = white noise floor (thermal + shot)
  f_c       = corner frequency (below which 1/f dominates)

Combined noise density:
  e_n_total(f) = e_n_white × sqrt(1 + f_c/f)

At f = f_c: 1/f noise equals the white noise floor → total noise is sqrt(2) × white noise
Below f_c:  1/f noise dominates
Above f_c:  white noise dominates
```

**Typical corner frequencies:**

```
BJT transistors (bipolar):   f_c ≈ 100 Hz to 1 kHz
JFET transistors:            f_c ≈ 100 Hz (slightly lower than BJT)
MOSFET transistors:          f_c ≈ 1 kHz to 10 MHz (generally worse than BJT/JFET)
Metal film resistors:        f_c < 10 Hz (very low — excellent for precision)
Carbon composition resistors: f_c > 100 kHz (very noisy at LF — avoid in precision circuits)
Precision op-amps (bipolar): f_c ≈ 100 Hz to 1 kHz
Chopper-stabilised op-amps:  f_c ≈ 0.1 Hz (extremely low — practical elimination of 1/f)
```

**Why 1/f noise matters for precision DC measurement:**

```
For a sensor measurement system with 0.1-10 Hz bandwidth:
  Most of the bandwidth is in the 1/f region for standard op-amps.
  The 1/f noise dominates over thermal noise.

  Example: OPA277 (precision bipolar):
    Voltage noise: 8 nV/√Hz at 1 kHz (white)
    Corner frequency: ~0.1 Hz
    At 1 Hz: e_n = 8 nV/√Hz × sqrt(0.1/1) × correction factor ≈ 15 nV/√Hz
    (1/f noise barely significant for this device at 1 Hz)

  Example: OPA27 (general precision):
    White noise: 3 nV/√Hz
    Corner frequency: ~2.7 Hz
    At 1 Hz: e_n ≈ 3 × sqrt(2.7/1) = 4.9 nV/√Hz (1/f significant)
    At 0.1 Hz: e_n ≈ 3 × sqrt(2.7/0.1) = 16 nV/√Hz (1/f dominates)
```

### Op-Amp Noise Model

A real op-amp is characterised by two noise sources referred to its input:

```
1. Input voltage noise: en [nV/√Hz]
   A voltage source in series with the non-inverting input.
   Amplified to output by the noise gain (1 + Rf/Ri for inverting amp).

2. Input current noise: in [pA/√Hz]
   A current source at each input terminal.
   Creates a noise voltage in × Zsource when flowing through source impedance.
   Significant for bipolar op-amps (in ~ 1-10 pA/√Hz) with high-Z sources.
   Negligible for CMOS/JFET op-amps (in < 0.1 pA/√Hz).
```

**Total input-referred noise (noise referred to input, RTI):**

```
e_ni² = en² + (in × Rsource)² + e_Rsource²

Where:
  en        = op-amp input voltage noise
  in        = op-amp input current noise
  Rsource   = source impedance
  e_Rsource = thermal noise of source resistance = sqrt(4kTRsource)

For en = 5 nV/√Hz, in = 2 pA/√Hz, Rsource = 10 kΩ, T = 300 K:

  e_Rsource = sqrt(4 × 1.38e-23 × 300 × 10e3) = 12.9 nV/√Hz
  in × Rsource = 2e-12 × 10e3 = 20 nV/√Hz

  e_ni = sqrt(5² + 20² + 12.9²) = sqrt(25 + 400 + 166) = sqrt(591) = 24.3 nV/√Hz

  The current noise dominates: a bipolar op-amp is poorly matched to a 10 kΩ source.
  Use a JFET/CMOS input op-amp (in < 0.1 pA/√Hz) for high-impedance sources.
```

**Noise gain:**

The op-amp's output noise = input-referred noise × noise gain:

```
Noise gain = 1 + Rf/Ri  (always ≥ 1, even for buffers)

For inverting amplifier with Av = -10 (Ri = 1 kΩ, Rf = 10 kΩ):
  Noise gain = 11
  Signal gain = 10
  The noise is amplified MORE than the signal (by ratio 11/10 = 1.1×)
  → Inverting amplifiers have slightly worse SNR than non-inverting for the same gain.
```

### Noise Budget Procedure

A noise budget is a systematic accounting of all noise sources in a signal chain:

```
Step 1: Identify all noise sources
  - Source resistance thermal noise
  - Sensor intrinsic noise (dark current for photodiodes, etc.)
  - Op-amp/INA voltage noise
  - Op-amp/INA current noise
  - Reference noise (for ADC)
  - ADC quantisation noise

Step 2: Refer all sources to a common reference point (input is most useful)
  Each source is divided by the gain from the reference point to that source's location.

Step 3: Combine uncorrelated sources in quadrature (RMS sum)
  e_total = sqrt(e1² + e2² + e3² + ...)
  (Only valid when sources are statistically independent, which they generally are.)

Step 4: Integrate over bandwidth to get RMS noise
  V_noise_rms = e_ni × sqrt(BW)   (for white noise)
  V_noise_rms = e_n_white × sqrt(f_c × ln(f_H/f_L) + (f_H - f_L)) for 1/f + white

Step 5: Compare to signal and compute SNR
  SNR = V_signal / V_noise_rms
  ENOB = (SNR_dB - 1.76) / 6.02
```

### Noise-Bandwidth Product

The noise bandwidth of a single-pole RC filter differs slightly from the -3 dB bandwidth:

```
Noise bandwidth = π/2 × f_-3dB ≈ 1.57 × f_-3dB

For a 10 kHz bandwidth low-pass filter:
  Effective noise bandwidth = 15.7 kHz (not 10 kHz)
  This 57% factor must be accounted for when computing RMS noise.

For a 2nd-order Butterworth filter:
  Noise bandwidth ≈ 1.11 × f_-3dB

For higher-order filters, the noise bandwidth approaches f_-3dB.
```

In practice, for order ≥ 3 filters, using the -3 dB bandwidth directly in noise calculations introduces < 10% error — often acceptable.

---

## Tier 1 — Fundamentals

### Question F1
**Calculate the thermal noise voltage (in µV RMS) of a 100 kΩ resistor in a bandwidth of 1 kHz at room temperature (27°C, 300 K).**

**Answer:**

```
Step 1 — Noise spectral density:
  e_n = sqrt(4 × k_B × T × R)
      = sqrt(4 × 1.38e-23 × 300 × 100,000)
      = sqrt(1.657e-15)
      = 40.7 nV/√Hz

Step 2 — Integrate over bandwidth:
  V_rms = e_n × sqrt(BW) = 40.7 nV/√Hz × sqrt(1000 Hz)
        = 40.7 × 31.62
        = 1.29 µV RMS
```

**Interpretation:** Any measurement system attempting to resolve signals below 1.29 µV through this resistor is limited by the fundamental thermal noise. This sets a hard floor — no amount of averaging or filtering can recover a signal below the noise floor set by the source resistance.

---

### Question F2
**An op-amp has en = 10 nV/√Hz (white noise, flat spectrum) and corner frequency fc = 200 Hz. Calculate the RMS input-referred noise in two frequency bands: (a) 0.1 Hz to 10 Hz (DC sensor measurement); (b) 1 kHz to 10 kHz (audio).**

**Answer:**

The combined noise density:

```
e_n(f) = e_n_white × sqrt(1 + fc/f) = 10 × sqrt(1 + 200/f)
```

**Part (a) — 0.1 Hz to 10 Hz:**

In the 1/f region (f << fc), the dominant term is sqrt(fc/f):

```
Noise in 1/f region, from f_L to f_H where f_H ≤ fc:
  V_rms_1f = e_n_white × sqrt(fc × ln(f_H/f_L))
           = 10 × sqrt(200 × ln(10/0.1))
           = 10 × sqrt(200 × ln(100))
           = 10 × sqrt(200 × 4.605)
           = 10 × sqrt(921)
           = 10 × 30.3
           = 303 nV RMS

Above fc (10 Hz is marginally below fc = 200 Hz — most of band is 1/f dominated):
Full calculation including white noise component:
  Total ≈ 10 × sqrt(fc × ln(10/0.1) + (10 - 0.1))
        = 10 × sqrt(921 + 9.9)
        = 10 × sqrt(930.9) = 305 nV RMS ≈ 305 nV RMS
```

**Part (b) — 1 kHz to 10 kHz:**

At 1-10 kHz, the 200 Hz corner frequency is well below this band — the noise is dominated by white noise:

```
V_rms_white = e_n_white × sqrt(BW) = 10 × sqrt(9000) = 10 × 94.9 = 949 nV RMS

1/f contribution at 1-10 kHz is negligible (fc/f = 200/1000 = 0.2 → 10% correction):
  Total ≈ 10 × sqrt(200 × ln(10000/1000) + (10000 - 1000))
        = 10 × sqrt(200 × 2.303 + 9000)
        = 10 × sqrt(460 + 9000)
        = 10 × sqrt(9460) = 972 nV RMS
```

**Comparison:** At DC (0.1-10 Hz), the 1/f noise produces 305 nV RMS despite the small bandwidth — almost 1/3 of the broadband noise at 1-10 kHz. Chopper-stabilised op-amps eliminate this 1/f contribution for precision DC measurements.

---

## Tier 2 — Intermediate

### Question I1
**An instrumentation amplifier (INA) with en = 8 nV/√Hz and in = 0.8 pA/√Hz is used to amplify a signal from a 350 Ω full bridge (Wheatstone bridge). The INA gain is 500. (a) Calculate the input-referred noise for a 100 Hz bandwidth; (b) identify the dominant noise source; (c) determine the total output-referred noise RMS; (d) calculate the resulting SNR if the bridge full-scale output is 10 mV.**

**Answer:**

**Part (a) — Input-referred noise sources:**

```
1. INA voltage noise:
   e_INA = 8 nV/√Hz × sqrt(100 Hz) = 8 × 10 = 80 nV RMS (RTI)

2. Source resistance thermal noise:
   Differential source impedance of the bridge:
   From V+ terminal: R1||R2 = 350||350 = 175 Ω
   From V– terminal: R3||R4 = 175 Ω
   Total differential: 350 Ω

   e_Rsource = sqrt(4kTR) = sqrt(4 × 1.38e-23 × 300 × 350)
             = sqrt(5.796e-18) = 2.41e-9 V/√Hz = 2.41 nV/√Hz

   Over 100 Hz: e_Rsource_rms = 2.41 × sqrt(100) = 24.1 nV RMS (RTI)

3. INA current noise × source impedance:
   in × Rsource = 0.8e-12 × 350 = 0.28 nV/√Hz
   Over 100 Hz: 0.28 × sqrt(100) = 2.8 nV RMS (RTI) — negligible

Total RTI noise:
  e_total_RTI = sqrt(80² + 24.1² + 2.8²) nV RMS
              = sqrt(6400 + 581 + 8) nV RMS
              = sqrt(6989) nV RMS
              = 83.6 nV RMS
```

**Part (b) — Dominant noise source:**

The INA voltage noise (80 nV RMS) dominates, contributing 96% of the total noise power. The source resistance thermal noise (24.1 nV) is secondary. Current noise is negligible — confirming that low-impedance bridge sources (350 Ω) are well-matched to bipolar INAs with low in.

**Part (c) — Output-referred noise:**

```
V_out_noise = e_total_RTI × Gain = 83.6 nV × 500 = 41.8 µV RMS
```

**Part (d) — SNR:**

```
Full-scale bridge output: 10 mV differential
INA output at full scale: 10 mV × 500 = 5 V

SNR = V_signal / V_noise_rms = 5 V / 41.8 µV = 119,617
SNR_dB = 20 × log(119617) = 101.6 dB

ENOB = (101.6 - 1.76) / 6.02 = 16.6 bits
```

This system achieves approximately 16.6 bits of effective resolution over a 100 Hz bandwidth — excellent for a strain gauge weighing application. In practice, the INA gain accuracy, offset temperature drift, and bridge balance error would reduce the effective accuracy, but the noise floor is not the limiting factor.

---

## Tier 3 — Advanced

### Question A1
**Design a noise budget for the front end of a DC-coupled ECG (electrocardiogram) amplifier. The electrode impedance is 50 kΩ (each electrode). The ECG signal is 1 mV peak, frequency content 0.05-150 Hz. The system must achieve better than 40 dB SNR (clinical requirement). Specify: (a) the maximum acceptable op-amp voltage noise; (b) the maximum acceptable op-amp current noise; (c) whether a bipolar or CMOS-input INA is preferred; (d) the 1/f noise concern and mitigation.**

**Answer:**

**Signal characterisation:**

```
V_signal = 1 mV peak = 0.354 mV RMS (sinusoidal approximation)
Bandwidth: 0.05-150 Hz
Source impedance: 50 kΩ per electrode
Differential source impedance: ~100 kΩ (two electrodes in series for differential signal)
```

**SNR requirement:**

```
Required SNR ≥ 40 dB → ratio = 100
Allowable noise floor: V_signal / SNR = 0.354 mV / 100 = 3.54 µV RMS
```

**Part (a) — Maximum voltage noise:**

Allocate 50% of noise budget to op-amp voltage noise:

```
e_n_op_amp_rms ≤ 0.5 × 3.54 µV = 1.77 µV RMS

For bandwidth 0.05-150 Hz (noise bandwidth ≈ 1.57 × 150 = 235 Hz):
  e_n (spectral density) ≤ 1.77 µV / sqrt(235) = 115 nV/√Hz
```

This is a generous specification — most op-amps achieve this. However, the 1/f noise in the 0.05-150 Hz band adds significantly (see Part d).

**Part (b) — Maximum current noise:**

The current noise flowing through the 100 kΩ differential source impedance creates a voltage noise:

```
Allocate remaining 50% of noise budget: 1.77 µV RMS for current noise term

in × Rsource ≤ 1.77 µV / sqrt(235) = 115 nV/√Hz
in ≤ 115 nV/√Hz / 100 kΩ = 1.15 pA/√Hz
```

This is a strict current noise requirement. Bipolar op-amps typically have in = 1-10 pA/√Hz at these frequencies. CMOS/JFET input op-amps have in < 0.1 pA/√Hz.

**Part (c) — Bipolar vs CMOS input:**

```
Thermal noise of source resistor (100 kΩ, 300 K, 235 Hz bandwidth):
  e_Rsource = sqrt(4kT × 100e3) × sqrt(235)
            = 40.7 nV/√Hz × 15.33
            = 624 nV RMS

This alone represents 624/3540 = 17.6% of the SNR budget — already significant.

For a bipolar INA with in = 5 pA/√Hz:
  Current noise contribution: 5e-12 × 100e3 × sqrt(235) = 5e-12 × 100e3 × 15.33
                             = 7.67 µV RMS — this EXCEEDS the entire noise budget!

Conclusion: A bipolar-input INA is unsuitable for 50 kΩ electrode impedance.
            CMOS or JFET input is mandatory.

For JFET INA (in = 10 fA/√Hz, e.g., INA116 or AD8221 at low frequency):
  Current noise: 10e-15 × 100e3 × 15.33 = 15.3 pV RMS — completely negligible.
```

**Part (d) — 1/f noise and mitigation:**

The ECG bandwidth extends down to 0.05 Hz. Virtually all op-amps have significant 1/f noise in this range.

```
For a standard precision INA with fc = 100 Hz:
  1/f noise in 0.05-150 Hz band:
  V_1f = e_n_white × sqrt(fc × ln(150/0.05) + (150 - 0.05))
       = 50 nV/√Hz × sqrt(100 × ln(3000) + 150)
       = 50 × sqrt(100 × 8.01 + 150)
       = 50 × sqrt(951) = 1542 nV RMS = 1.54 µV RMS

  This alone uses 1.54/3.54 = 43% of the noise budget — problematic.
```

**Mitigations:**

1. **Chopper-stabilised (auto-zero) INA:** Eliminates 1/f noise below the chopper frequency (typically 10-50 kHz). Effective fc becomes < 0.1 Hz. Example: ADA2200 (auto-zero, 7.5 nV/√Hz, effectively zero 1/f in the ECG band).

2. **AC coupling with baseline wander filter:** Most clinical ECG systems AC-couple the electrode input with a cutoff at 0.05 Hz (10 µF, 330 kΩ RC). This removes the DC drift from the measurement band. The AC coupling high-pass filter shifts the lower edge of sensitivity from near-DC to 0.05 Hz, reducing the total bandwidth and thus the integrated 1/f noise.

3. **Driven-right-leg circuit:** In clinical ECG, the common-mode voltage (patient body voltage) is measured and fed back via the right-leg electrode to reduce the common-mode interference at the amplifier inputs. This is not a noise reduction technique per se, but reduces the common-mode signal that stresses the amplifier CMRR.

**Recommended device:** INA333 (CMOS, en = 55 nV/√Hz, in = 200 fA/√Hz, Vos = 25 µV max, fc = ~0.1 Hz due to chopper stabilisation) or custom auto-zero design for the most demanding clinical use.

---

## Best Practices

1. Always refer all noise sources to the input (RTI) before summing. Adding noise at different points in the chain without referencing to a common point gives incorrect results.
2. Sum uncorrelated noise sources in quadrature (RMS sum): e_total = sqrt(e1² + e2² + ...). Never sum linearly — that is the worst-case bound, not the statistical expectation.
3. For source impedances > 10 kΩ, use CMOS or JFET input op-amps to avoid current noise × source impedance terms dominating the noise budget.
4. Chopper-stabilised (auto-zero) op-amps are mandatory for DC and sub-Hz measurements requiring < 1 µV noise — the 1/f noise of standard precision op-amps dominates this band.
5. Reduce noise by reducing bandwidth: every factor of 4× reduction in bandwidth halves the integrated noise. Always define the minimum acceptable bandwidth and filter aggressively to it.
6. Use 1% metal film resistors rather than carbon composition in precision analogue circuits — metal film has negligible excess (1/f) noise; carbon composition can add significant 1/f noise at low frequencies.

---

## Related Topics

- [Op-Amp Circuits](op_amp_circuits.md)
- [ADC/DAC Interfaces](adc_dac_interfaces.md)
- [Sensor Signal Conditioning](sensor_signal_conditioning.md)
- [Problem 01: Instrumentation Amplifier](worked_problems/problem_01_instrumentation_amp.md)
- [Problem 02: ADC SNR](worked_problems/problem_02_adc_snr.md)
- [Component Selection](../01_schematic_design/component_selection.md)
