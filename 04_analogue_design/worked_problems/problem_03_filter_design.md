# Problem 03: Filter Design

## Problem Statement

You are designing the signal chain for an industrial vibration monitor. After the sensor and gain stage (from Problem 02), you need three separate filter functions in the same signal path:

1. **Anti-aliasing filter:** Attenuate signals above 12.5 kHz (Nyquist for 25 kSPS ADC) by at least 72 dB (12-bit ADC dynamic range). The passband must be flat to within ±0.5 dB up to 10 kHz.

2. **High-pass filter:** Remove DC offset and low-frequency drift below 10 Hz from the vibration signal. The -3 dB corner must be at exactly 10 Hz with ±10% tolerance. Below 1 Hz the attenuation must be > 20 dB.

3. **Notch filter (60 Hz):** Attenuate the 60 Hz mains interference by at least 40 dB. The notch must not disturb signals outside the 50-70 Hz band by more than 3 dB.

**Part A:** Design the anti-aliasing filter. Select an appropriate filter type (Butterworth, Chebyshev, Bessel), determine the minimum order, and calculate component values for one complete filter stage.

**Part B:** Design the high-pass filter. Calculate the component values for a 1st-order and 2nd-order implementation. Justify which is more appropriate for this application.

**Part C:** Design the 60 Hz notch filter using a twin-T network. Calculate the component values, state the sensitivity of the notch frequency to component tolerance, and propose a method to tune the notch frequency in production.

**Part D:** Determine the correct ordering of these three filter sections in the signal chain and justify the order. Discuss how op-amp slew rate and bandwidth interact with the filter chain.

---

## Solution Approach

### Part A — Anti-Aliasing Filter Design

**Specifications:**

```
Passband: DC to 10 kHz, ±0.5 dB ripple maximum
Stopband: at 12.5 kHz, ≥ 72 dB attenuation
Stopband-to-passband ratio: 12.5 / 10 = 1.25 (very narrow transition band)
```

**Filter type comparison:**

```
Butterworth: maximally flat passband (no ripple), -20N dB/decade rolloff.
  Most predictable behaviour. Preferred when passband flatness is critical.

Chebyshev Type I: equiripple in passband, steeper rolloff for same order.
  Can achieve the stopband requirement at lower order for the same ripple spec.

Chebyshev Type II: maximally flat passband, equiripple in stopband.
  Better rejection behaviour in the stopband at the cost of stopband equiripple.

Bessel: maximally flat group delay (linear phase). Rolloff is much less steep.
  Not appropriate here — the narrow transition band requires steep rolloff.

Elliptic: equiripple in both passband and stopband. Steepest rolloff.
  Minimum component count for this specification, but complex to design.
```

**Order calculation for Butterworth (passband edge = 10 kHz, ±0.5 dB tolerance):**

The Butterworth passband ripple constraint sets the cutoff frequency:

```
±0.5 dB at 10 kHz means gain ≥ -0.5 dB at 10 kHz (i.e., gain ≥ 0.944 at passband edge)

For a Butterworth with -3 dB at fc:
  At f = 10 kHz: gain = 1 / sqrt(1 + (10/fc)^(2N)) ≥ 0.944

  (10/fc)^(2N) ≤ 1/0.944² - 1 = 1.122 - 1 = 0.122

  Set fc > 10 kHz to meet the passband flatness requirement.
  Optimise by setting fc such that both passband and stopband conditions are met.
```

**Choosing Chebyshev Type I (0.5 dB ripple):**

For equiripple ±0.5 dB in the passband up to 10 kHz:

```
The -0.5 dB point is the passband edge. The stopband requirement is 72 dB at 12.5 kHz.

Selectivity factor: k = f_stop / f_pass = 12.5 / 10 = 1.25

For Chebyshev order estimation:
  N ≥ cosh⁻¹(sqrt((10^(As/10) - 1) / (10^(Ap/10) - 1))) / cosh⁻¹(k)

  Where As = 72 dB (stopband), Ap = 0.5 dB (passband ripple):
  
  epsilon_p = sqrt(10^(0.5/10) - 1) = sqrt(0.1220) = 0.3493
  epsilon_s = sqrt(10^(72/10) - 1) ≈ sqrt(1.585×10^7) = 3981

  N ≥ cosh⁻¹(3981 / 0.3493) / cosh⁻¹(1.25)
    = cosh⁻¹(11,397) / cosh⁻¹(1.25)

  cosh⁻¹(x) = ln(x + sqrt(x²-1))
  cosh⁻¹(11397) = ln(11397 + sqrt(11397²-1)) = ln(22794) = 10.03
  cosh⁻¹(1.25)  = ln(1.25 + sqrt(1.25²-1)) = ln(1.25 + 0.75) = ln(2.0) = 0.693

  N ≥ 10.03 / 0.693 = 14.5 → N = 15
```

A 15th-order filter is still impractical for a narrow guard band. This confirms the conclusion from Problem 02: a 25 kSPS sample rate with a 10 kHz bandwidth requires an unrealistically high-order filter. The practical solution is to either oversample at 100 kSPS (reducing the filter order to ~7) or use a delta-sigma ADC.

**Revised design: 7th-order Chebyshev at 100 kSPS (oversampled):**

```
With 100 kSPS: Nyquist = 50 kHz
Selectivity: k = 50 / 10 = 5

N ≥ cosh⁻¹(11397 / 0.3493) / cosh⁻¹(5)
  = cosh⁻¹(32623) / cosh⁻¹(5)
  = 10.78 / 2.29
  = 4.7 → N = 5 (5th-order Chebyshev gives > 72 dB at 50 kHz with 0.5 dB ripple)
```

**5th-order Chebyshev active filter implementation:**

```
Structure: two 2nd-order Sallen-Key stages + one 1st-order RC

Pole frequencies and Q values (0.5 dB ripple Chebyshev, 5th order, fc = 10 kHz):
  Stage 1 (2nd order): f01 = 10.86 kHz, Q1 = 0.707  (from pole tables)
  Stage 2 (2nd order): f02 = 8.02 kHz, Q2 = 1.777
  Stage 3 (1st order): f03 = 6.64 kHz (single pole RC)
  (Values from Williams & Taylor, "Electronic Filter Design Handbook", 4th ed.)

Component values for Stage 2 (highest Q, most critical):
  Target: f0 = 8.02 kHz, Q = 1.777

  Using Sallen-Key with equal R: C1 = C, C2 = C×m, R1 = R2 = R
  Q = sqrt(m) / (3 - K), f0 = 1 / (2π × R × C × sqrt(m))

  Choose C = 10 nF, m = Q² × (3-K)²...
  Simplified design (using design tables for Sallen-Key):
    Choose C1 = 10 nF, C2 = 10 nF (equal), then adjust R for each section.

  For equal-C Sallen-Key with unity gain:
    Q = 0.5 (only) → not suitable for Q = 1.777.

  For Q = 1.777 with Sallen-Key gain K = 3 - 1/Q = 3 - 0.563 = 2.437:
    Buffer gain K = 2.437 → non-inverting buffer with Rf/Ri = K-1 = 1.437
    Select Ri = 10 kΩ, Rf = 14.37 kΩ → use 14.3 kΩ E96

    R = 1 / (2π × f0 × C × sqrt(C1/C2)) — for equal C:
    R = 1 / (2π × 8020 × 10e-9) = 1987 Ω → use 2.0 kΩ E96

    Actual f0 with R = 2.0 kΩ: f0 = 1/(2π × 2000 × 10e-9) = 7.96 kHz (0.7% error — acceptable)
```

### Part B — High-Pass Filter Design

**1st-order passive RC high-pass:**

```
Target: fc = 10 Hz (-3 dB corner)
Select C = 10 µF (electrolytic or film), then R = 1/(2π × fc × C) = 1/(2π × 10 × 10e-6)
  = 1592 Ω → use 1.6 kΩ

Below 1 Hz: attenuation = 20 × log(f/fc) = 20 × log(1/10) = -20 dB
  At 1 Hz: -20 dB ✓ (meets the > 20 dB requirement at 1 Hz)

Passband flatness:
  At 10 kHz: attenuation = 20 × log(10000/10) = 60 dB gain (high-pass → passes 10 kHz)
  The high-pass filter has negligible effect above its corner (< 0.01 dB at 10 kHz).
```

**2nd-order Sallen-Key high-pass (Butterworth, 2 poles at fc):**

```
Two poles at 10 Hz gives steeper roll-off below fc:
  At 1 Hz: attenuation = 40 × log(1/10) = -40 dB (vs -20 dB for 1st-order)
  Better rejection of very low frequency drift (0.1-1 Hz baseline wander).

Component values (equal component Sallen-Key HPF, Butterworth Q = 0.707):
  C1 = C2 = C = 10 µF
  R1 = R2 = R = 1/(2π × 10 × 10e-6) = 1592 Ω → use 1.6 kΩ
  Buffer gain K = 3 - 1/Q = 3 - 1/0.707 = 3 - 1.414 = 1.586
  Rf/Ri = 0.586 → Ri = 10 kΩ, Rf = 5.86 kΩ → use 5.9 kΩ E96
```

**Which is more appropriate?**

For vibration monitoring where baseline wander and DC offset are concerns:

```
The 2nd-order design is preferred because:
(a) -40 dB at 1 Hz better rejects low-frequency mechanical drift and temperature
    effects on the sensor, which typically appear in the 0.01-1 Hz range.
(b) The Butterworth response gives maximally flat passband above 10 Hz —
    no amplitude ripple in the vibration frequency band.
(c) The 2nd-order only requires one additional op-amp stage compared to 1st-order.

Concern: the 10 µF capacitors required for a 10 Hz cutoff are large.
  Use film capacitors (polyester or polypropylene) for stability and low leakage.
  Electrolytic capacitors have large tolerance (±20%) and degrade with age.
  With 20% capacitor tolerance, fc varies ±20% = 8-12 Hz (within the ±10% spec:
  9-11 Hz). Tighter spec requires 5% or 1% film capacitors.
```

### Part C — 60 Hz Twin-T Notch Filter

**Twin-T topology:**

The passive twin-T network is a null network at the notch frequency:

```
Circuit (twin-T):
         C        C
Vin --||---+---||-- Vout
           |
           R/2
           |
          GND

     R        R
     |         |
     +----||---+
          2C

Notch frequency: f_notch = 1 / (2π × R × C)
```

**Component values for 60 Hz notch:**

```
Select C = 100 nF (film capacitor, 1% tolerance):
R = 1 / (2π × 60 × 100e-9)
  = 1 / (3.770e-5)
  = 26.53 kΩ

Use E96 value: 26.7 kΩ (0.64% error → f_notch = 59.6 Hz)

Full twin-T network:
  Two R = 26.7 kΩ (series arms)
  One R/2 = 13.3 kΩ (shunt arm) → use 13.3 kΩ E96
  Two C = 100 nF (series arms)
  One 2C = 200 nF (shunt arm) → use 200 nF (two 100 nF in parallel)
```

**Sensitivity to component tolerance:**

The notch frequency is directly proportional to 1/(RC). Any tolerance in R or C shifts f_notch:

```
Δf_notch / f_notch = sqrt((ΔR/R)² + (ΔC/C)²)

For 1% R (ΔR/R = 0.01) and 1% C (ΔC/C = 0.01):
  Δf_notch / f_notch = sqrt(0.01² + 0.01²) = sqrt(2) × 0.01 = 1.41%

At 60 Hz: Δf_notch = ±0.85 Hz → notch between 59.2 and 60.9 Hz
```

The passive twin-T has very high Q at the notch but finite depth due to component mismatch:

```
Notch depth with 1% resistor mismatch: approximately 34-40 dB (1% mismatch)
With 0.1% matched resistors: notch depth > 60 dB
```

For 40 dB notch depth with 1% components, the passive twin-T is marginally acceptable. Use 0.1% tolerance or tighter components for more reliable 40 dB rejection.

**Active twin-T for sharper Q:**

Wrap the twin-T network in a feedback loop to increase Q and notch depth:

```
Vout → R_feedback → Vref_input of twin-T (shunt node)

This creates an active notch filter with adjustable Q. Higher Q makes the notch
narrower — better for a sharp 60 Hz notch without affecting 50-70 Hz range.
```

**Production tuning method:**

1. **Fixed trim resistor:** Replace one of the series R values with a fixed resistor in series with a small trimpot (e.g., 26.1 kΩ + 1 kΩ trimpot). Measure f_notch during production test and adjust the pot for maximum attenuation. Lock with thread-locking compound.

2. **Component sorting:** Sort 1% resistors and capacitors into ±0.1% bins at incoming inspection. Use matched sets for the twin-T. This achieves 0.1% effective tolerance without premium pricing.

3. **Software-based 60 Hz rejection:** If a DSP or microcontroller is in the processing chain, implement a digital notch filter (IIR notch) instead of an analog twin-T. A second-order digital IIR notch provides >60 dB rejection, is tunable in firmware, and requires no analog components. This is the preferred approach for modern digital systems.

### Part D — Filter Ordering and Op-Amp Considerations

**Correct filter order:**

```
[Sensor] → [High-pass filter] → [Gain stage] → [Anti-aliasing LPF] → [Notch filter] → [ADC]
```

**Justification:**

1. **High-pass filter first (before gain):** The sensor output may have a significant DC offset or low-frequency drift. Amplifying this offset at gain = 8 (Part D of Problem 02) would saturate the gain stage op-amp before the signal reaches the anti-aliasing filter. Remove the DC and drift before amplification.

2. **Gain stage second:** After the high-pass filter has removed unwanted low-frequency content, the signal is amplified to match the ADC input range. This maximises SNR in subsequent stages.

3. **Anti-aliasing filter third:** The LPF operates on the amplified signal — it must handle the full signal swing (±2.5 V). Placing it before the gain stage would require a higher-gain amplifier afterwards (increasing noise from the gain stage).

4. **Notch filter last:** The notch filter operates at the ADC input level. Placing it after anti-aliasing ensures the LPF has already attenuated broadband noise that might otherwise couple into the notch filter and degrade its depth.

**Op-amp slew rate and bandwidth constraints:**

The anti-aliasing filter and notch filter operate with signals up to the ADC full scale (5 V peak-to-peak at 10 kHz):

```
Minimum slew rate:
  SR_min = 2π × f_max × V_peak = 2π × 10,000 × 2.5 V = 157,000 V/s = 0.157 V/µs

  Use op-amp with SR ≥ 1 V/µs (10× margin). Example: OPA350 (SR = 22 V/µs), LMC6482 (SR = 10 V/µs).

Minimum GBW:
  For the highest-Q stage (Q = 1.777, f0 = 8 kHz, noise gain = K = 2.437):
  GBW ≥ 50 × Q² × f0 = 50 × 3.16 × 8000 = 1.264 MHz
  (Rule: GBW ≥ 50 × Q² × f0 for stable Sallen-Key with gain)

  Use op-amp with GBW ≥ 10 MHz for 5× margin. Example: LMC6482 (GBW = 1.5 MHz) — borderline.
  Better: OPA2134 (GBW = 8 MHz, SR = 20 V/µs) for the critical high-Q stage.
```

**Single-supply considerations:**

With a 3.3 V single supply, the signal chain must operate within 0 to 3.3 V (or use a virtual ground at 1.65 V). Key requirements:

```
Input and output: use rail-to-rail input/output (RRIO) op-amps.
  Example: OPA350 (3-5.5 V single supply, RRIO), MCP6002 (1.8-6 V, RRIO)

High-pass filter capacitor bias: the AC coupling capacitor blocks DC.
  Establish a virtual mid-supply reference (1.65 V) using a resistor divider + buffer.
  Connect the bottom of the high-pass filter to this 1.65 V reference, not to ground.

ADC reference: 3.3 V single-supply ADC with external VREF = 3.3 V or internal 2.5 V.
```

---

## Key Takeaways

- A narrow transition band (12.5:10 kHz ratio = 1.25) makes a practical anti-aliasing filter impossible at the Nyquist of 25 kSPS — the correct solution is to oversample at 4× or more and use a higher-order filter with a more practical transition band ratio
- Filter section ordering matters: high-pass (DC rejection) before amplification prevents saturation from DC offset; anti-aliasing before the notch ensures the LPF sees the full signal swing while the notch operates on a band-limited signal
- The twin-T passive notch filter is sensitive to component matching — for production reliability, prefer 0.1% components or a DSP-based digital notch filter implemented in firmware
- Op-amp GBW must be >> noise gain × filter frequency, especially for high-Q stages — underpowered op-amps cause gain peaking and stability issues that appear as unexpected ripple in the passband

---

## Interview Notes

Filter design problems test the depth of analogue knowledge at hardware engineering interviews. The key discriminators:

1. **Order calculation:** Many candidates say "use a low-pass filter" without quantifying the order or recognising the transition band problem. Calculating that a 1.25:1 transition requires 15th-order Butterworth (and then proposing oversampling as the fix) demonstrates rigour.

2. **Twin-T sensitivity:** Knowing that a twin-T notch is highly sensitive to component matching, and proposing matched components or a digital alternative, is a senior-level answer.

3. **Filter ordering:** Stating the correct sequence (HPF → gain → LPF → notch) with clear justification separates engineers who have implemented real signal chains from those who know only textbook topologies.

4. **Op-amp selection:** Mentioning slew rate (not just GBW) for large-signal, high-frequency filter stages is a practical detail that interviewers at signal processing companies value highly.

A follow-up question often asked: "How would you test that the anti-aliasing filter meets specification?" Answer: inject a known-frequency, known-amplitude sinusoid into the filter input, measure the output amplitude at frequencies across the passband and stopband, and compare to the specification. Use a network analyser for automated frequency sweeps, or a precision signal generator + ADC measurement for a simple bench test.
