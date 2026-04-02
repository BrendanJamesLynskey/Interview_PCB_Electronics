# Analogue Quiz

Fifteen multiple-choice questions covering operational amplifier circuits, ADC and DAC
interfaces, sensor signal conditioning, and noise analysis. Each question has four
options. Explanations are provided in the Answer Key at the end.

---

## Questions

---

### Question 1

An inverting amplifier uses a 10 kΩ feedback resistor (RF) and a 1 kΩ input resistor
(RIN). The op-amp is powered from ±15 V rails. An input signal of +1.5 V DC is applied.
What is the expected output voltage, and what component would you add to minimise input
bias current error?

A) Output = -15 V (saturated at the negative supply); add a capacitor in parallel with
   RF to reduce bias current.

B) Output = -15 V (saturated); no additional component is needed because ideal op-amps
   have zero bias current.

C) Output = -(RF/RIN) × VIN = -10 × 1.5 = -15 V. For a ±15 V supply, this is at the
   negative rail — the output saturates. To minimise bias current error, add a resistor
   equal to RF ∥ RIN = (10k × 1k) / (10k + 1k) ≈ 909 Ω from the non-inverting input
   to ground.

D) Output = -(RF/RIN) × VIN = -10 × 1.5 = -15 V. The output is in the linear region
   because the gain is exactly 10 and the output equals the supply voltage.

---

### Question 2

A single-supply 5 V op-amp (not rail-to-rail output) is configured as a voltage follower
(unity-gain buffer) for a sensor output that swings between 0.1 V and 4.9 V. The op-amp
datasheet specifies "output voltage range: 0.5 V to VCC - 0.5 V" (0.5 V to 4.5 V).
What happens at the extreme ends of the sensor output range?

A) The op-amp output accurately tracks the input from 0.1 V to 4.9 V because the unity
   gain configuration does not limit the output swing.

B) When the input reaches 0.1 V, the output saturates at 0.5 V. When the input reaches
   4.9 V, the output saturates at 4.5 V. The sensor range beyond these limits is clipped
   and the downstream circuit receives incorrect data.

C) The op-amp enters a latch-up condition below 0.5 V and above 4.5 V, requiring a
   power cycle to recover.

D) The output clips only at the upper end (4.5 V) because single-supply op-amps can
   swing to GND (0 V) on their output without error.

---

### Question 3

An instrumentation amplifier (INA) has a gain set to 100 by an external resistor. The
common-mode rejection ratio (CMRR) is specified as 100 dB at DC and 80 dB at 1 kHz.
A 50 Hz mains interference signal of 2 V peak is present on both inputs. What is the
approximate interference voltage at the INA output at 50 Hz?

A) 200 V — the INA amplifies the common-mode signal by the gain of 100.

B) 0 V — the INA completely rejects all common-mode signals regardless of frequency.

C) Approximately 2 mV — the CMRR at 50 Hz (between DC and 1 kHz, likely better than
   80 dB) means the 2 V common-mode signal appears as approximately 2 V / 10^(CMRR_dB/20)
   at the differential input. Using 100 dB at 50 Hz: 2 V / 100000 = 20 µV referred to
   input, × gain 100 = 2 mV at output.

D) Approximately 200 µV — CMRR of 100 dB means only 1 µV appears at the input; with
   gain 100 the output sees 100 µV.

---

### Question 4

A 16-bit ADC has a full-scale input range of ±2.5 V and is measuring a signal with
a bandwidth of 10 kHz. The ADC's ENOB (Effective Number of Bits) is 14.5 bits. What
is the signal-to-noise ratio of the ADC and what is its dynamic range limitation?

A) SNR = 6.02 × 16 + 1.76 = 98.1 dB (ideal 16-bit); ENOB of 14.5 bits means the
   actual SNR is 6.02 × 14.5 + 1.76 ≈ 89 dB.

B) SNR = 6.02 × 14.5 + 1.76 ≈ 89 dB; the ADC performs as a 14.5-bit converter in
   terms of noise performance. The remaining 1.5 bits are lost to ADC noise (thermal
   noise, quantisation noise, and harmonic distortion), not to the signal bandwidth.

C) SNR = 20 × log10(2^16) = 96.3 dB; ENOB has no effect on SNR because it relates
   to linearity, not noise.

D) SNR = 6.02 × 14.5 = 87.3 dB; the +1.76 dB term only applies to ideal converters.

---

### Question 5

A photodiode transimpedance amplifier (TIA) uses a 1 MΩ feedback resistor and a 1 pF
feedback capacitor in parallel. The photodiode has a junction capacitance of 5 pF. What
is the -3 dB bandwidth of this amplifier, and what is the function of the feedback
capacitor?

A) Bandwidth = 1/(2π × RF × CF) = 1/(2π × 10⁶ × 10⁻¹²) = 159 kHz. The feedback
   capacitor stabilises the amplifier by adding a zero in the feedback network that
   compensates for the pole introduced by the photodiode junction capacitance.

B) Bandwidth = 1/(2π × RF × (CF + CD)) = 1/(2π × 10⁶ × 6 × 10⁻¹²) = 26.5 kHz.
   The photodiode capacitance adds in series with CF and reduces bandwidth.

C) Bandwidth is determined only by the op-amp's gain-bandwidth product; the RC
   components do not limit bandwidth in a TIA configuration.

D) Bandwidth = 1/(2π × RF × CD) = 1/(2π × 10⁶ × 5 × 10⁻¹²) = 31.8 kHz. The feedback
   capacitor is used to measure the photodiode capacitance from the circuit.

---

### Question 6

A designer uses a DAC with 12-bit resolution and a 3.3 V full-scale output to control a
precision servo motor. The motor requires 0.01% accuracy. Is the DAC resolution adequate?

A) Yes — 12-bit resolution gives 4096 levels across 3.3 V, providing a step size of
   3.3/4096 = 0.806 mV, which is 0.806/3300 = 0.024% resolution. This marginally
   exceeds the 0.01% requirement; a 14-bit or 16-bit DAC is more appropriate.

B) Yes — 12-bit resolution always provides 0.01% accuracy because accuracy and resolution
   are the same parameter.

C) No — a 12-bit DAC has only 12 discrete levels, which is insufficient for precision
   control applications.

D) Yes — 12-bit resolution provides 1 LSB = 3.3 V / 4096 = 0.806 mV. The 0.01%
   accuracy requirement corresponds to 3.3 V × 0.0001 = 0.33 mV, which is less than
   1 LSB. The DAC resolution is inadequate regardless of converter quality.

---

### Question 7

A precision 24-bit ADC is used to measure a thermocouple signal (5 µV/°C, range 0-1000°C).
The thermocouple output requires amplification before the ADC input. The ADC has an
internal reference of 2.048 V and an input range of ±2.048 V. The instrumentation
amplifier is set to a gain of 200. What is the measurement resolution in °C?

A) Resolution = 2.048 V / (200 × 2^24) / (5 µV/°C) ≈ 0.24 m°C per LSB.

B) Resolution = (2.048 V / 2^24) / 200 / (5 µV/°C) = 0.24 m°C per LSB.

C) Resolution = 2.048 V / 2^24 × 200 / (5 µV/°C) ≈ 4.88°C per LSB.

D) Resolution = (2.048 V / 2^24) × (1/200) × (1/5 µV per °C) = full-scale range
   in degrees ÷ 2^24.

---

### Question 8

A voltage reference IC is rated at 2.5 V ±0.1% initial accuracy and has a temperature
coefficient of 5 ppm/°C. Over an operating temperature range of -40°C to +85°C (a
span of 125°C), what is the total accuracy budget for the reference voltage?

A) Total error = initial accuracy only = ±0.1%; the temperature coefficient is a
   separate specification and does not add to the initial accuracy budget.

B) Total error ≈ ±0.1% + (5 ppm/°C × 125°C) = ±0.1% + 625 ppm = ±0.1% + 0.0625%
   = ±0.1625% of full-scale. At 2.5 V, this is ±4.06 mV total.

C) Total error = 5 ppm/°C × 125°C = 625 ppm = 0.0625%; the initial accuracy cancels
   out because it represents a fixed offset that can be calibrated.

D) Total error = √(0.1%² + 0.0625%²) = ±0.118% (RSS combination), because initial
   accuracy and temperature drift are independent error sources.

---

### Question 9

A sensor output has a 0-100 mV range with 0.5 Ω source impedance. It is connected to
a 12-bit SAR ADC with a 1 V reference and a 10 kΩ input impedance. What is the primary
measurement error mechanism and what is its magnitude?

A) The sensor source impedance (0.5 Ω) forms a voltage divider with the ADC input
   impedance (10 kΩ); the divider ratio is 10000/(10000 + 0.5) ≈ 0.99995 — a
   negligible error of 0.005%.

B) The ADC input impedance is too low: the 10 kΩ input impedance loads the 0.5 Ω
   source and causes the measured voltage to be 100 × (10000/10000.5) ≈ 99.995 mV
   instead of 100 mV — an error of 0.005%, which is less than 1 LSB of a 12-bit ADC
   on a 1 V reference (1 LSB = 1000 mV / 4096 = 0.244 mV). Source loading is not
   the primary concern.

C) The 100 mV signal uses only 10% of the ADC's 1 V full-scale range. The effective
   resolution is reduced from 12 bits to approximately 9 bits (1/10th of full scale
   = 3.3 bits of wasted resolution), which is the primary error mechanism.

D) The ADC will fail to convert correctly because its reference (1 V) exceeds the
   sensor output range (0-100 mV).

---

### Question 10

An op-amp integrator has an input resistor of 10 kΩ and a feedback capacitor of 10 nF.
A DC input offset voltage of 2 mV is present at the op-amp's input. What happens to
the integrator output over time?

A) The output remains stable at 0 V because the feedback capacitor blocks DC and
   prevents the offset from integrating.

B) The output ramps at a rate of VOS / (RIN × CF) = 2 mV / (10 kΩ × 10 nF) = 20 V/s
   until the output saturates at the supply rail, even when no intended input signal
   is present.

C) The output oscillates because the integrator has 90° phase shift and the offset
   voltage provides positive feedback.

D) The output settles to a steady-state value of VOS × RF / RIN because the integrator
   acts as a low-pass filter at DC.

---

### Question 11

A 16-bit ADC is sampling a 1 kHz sinusoidal signal at 100 kSamples/s. The signal
amplitude spans 80% of the ADC's full-scale range. The sampling frequency satisfies
Nyquist. What is the MOST effective technique to improve the effective noise floor of
the measurement by 6 dB?

A) Increase the ADC sampling rate to 200 kSamples/s without changing the signal
   bandwidth; the additional samples are averaged (decimation), reducing noise by
   3 dB per factor of two in sample rate.

B) Average four consecutive ADC samples together. Averaging N samples reduces the
   white noise power by N and the noise amplitude by √N. Averaging 4 samples reduces
   noise amplitude by a factor of 2, improving SNR by 6 dB (= 20 log10(2)).

C) Apply a 100 Hz low-pass digital filter after the ADC to reject noise above the
   signal bandwidth. Reducing the measurement bandwidth by 10× improves the noise floor
   by 10 dB.

D) Increase the ADC reference voltage from 1 V to 2 V to use more of the supply range,
   which reduces the noise floor by 6 dB.

---

### Question 12

A resistive temperature detector (RTD) with a nominal resistance of 100 Ω at 0°C
(PT100) is measured using a two-wire configuration with a 1 mA constant current source.
The connecting cable has 2 Ω total resistance. What temperature measurement error
results, and how is it corrected?

A) The cable resistance adds 2 Ω to the measured RTD resistance. Since PT100 has a
   sensitivity of 0.385 Ω/°C, the error is 2/0.385 = 5.2°C. Correction requires either
   a four-wire (Kelvin) measurement or knowledge of the exact cable resistance.

B) The cable resistance adds a voltage offset equal to 1 mA × 2 Ω = 2 mV, which is
   negligible at measurement voltages of 100 mV.

C) The cable resistance causes a non-linearity error because the current source is
   not ideal, but no systematic offset error occurs because the current is constant.

D) The two-wire configuration is standard for PT100 measurements below 500 Ω; no
   correction is necessary.

---

### Question 13

An audio ADC (24-bit, 96 kSamples/s) measures the acoustic output of a loudspeaker
system. The acoustic measurement microphone has a self-noise floor of -120 dBV in a
20 kHz bandwidth. The ADC has an input-referred noise floor of -105 dBV in a 20 kHz
bandwidth. Which component limits the system noise floor, and by how much does the
ADC noise degrade the overall measurement?

A) The microphone limits the system noise floor. The ADC noise (-105 dBV) is 15 dB
   above the microphone noise (-120 dBV), so the ADC dominates and the total noise floor
   is approximately -105 dBV — 15 dB above the microphone floor.

B) The microphone limits the system noise floor. With the microphone floor at -120 dBV
   and ADC at -105 dBV, the ADC is the limiting element because the system cannot be
   more sensitive than its loudest noise source.

C) Neither component limits the system; at 24 bits the ADC's theoretical SNR exceeds
   144 dB, which is below the microphone's -120 dBV noise floor.

D) The ADC limits the system noise floor at -105 dBV because the ADC noise is higher
   (louder) than the microphone noise. The total noise floor is determined by the
   dominant noise source, which is the ADC at -105 dBV.

---

### Question 14

A Sallen-Key low-pass filter is designed for 1 kHz cutoff frequency with Q = 0.707
(Butterworth response). The components are: R1 = R2 = 10 kΩ, C1 = 22 nF, C2 = 11 nF.
The op-amp has a gain-bandwidth product (GBW) of 1 MHz. At what frequency does the
op-amp's finite gain-bandwidth product begin to degrade the filter response significantly?

A) Above 1 kHz, because the filter cutoff frequency equals the GBW divided by the DC gain
   of the Sallen-Key stage.

B) At approximately 100 kHz, because the op-amp open-loop gain starts to roll off well
   below this frequency and the feedback loop of the Sallen-Key stage becomes gain-limited.

C) The GBW of 1 MHz is 1000× the filter cutoff of 1 kHz; the Sallen-Key stage has unity
   gain (voltage follower), so the op-amp must maintain adequate open-loop gain up to
   approximately 10× the cutoff frequency (10 kHz). At 10 kHz, the GBW = 1 MHz gives
   open-loop gain of 100 — adequate. Significant degradation typically begins when the
   open-loop gain falls below 40 dB (100×) above the cutoff.

D) The GBW does not affect filter response because the op-amp is used in a unity-gain
   configuration and the filter response is set entirely by passive components.

---

### Question 15

A current sense resistor of 10 mΩ is used to measure 10 A current in a DC power supply.
A difference amplifier measures the voltage across the sense resistor with a gain of
100. The sense voltage is 10 A × 10 mΩ = 100 mV; the amplifier output is 10 V. The
resistor has a temperature coefficient of 50 ppm/°C. At 100°C above calibration
temperature, what is the measurement error in milliamps?

A) Error = 50 ppm/°C × 100°C = 5000 ppm = 0.5%. At 10 A, this is 50 mA.

B) Error = 50 ppm/°C × 100°C = 0.5% of resistance change. The resistance changes from
   10 mΩ to 10.05 mΩ. At 10 A, the measured voltage changes from 100 mV to 100.5 mV.
   The amplifier output changes from 10 V to 10.05 V. The current measurement error is
   0.5% of 10 A = 50 mA.

C) Error = 50 ppm/°C × 100°C = 5000 ppm = 0.005 Ω. At 10 A, voltage error = 0.005 × 10
   = 0.05 V. Amplifier output error = 0.05 × 100 = 5 V. This represents a 50% gain error.

D) The temperature coefficient only applies above the maximum rated temperature of the
   resistor; at 100°C above calibration the resistor is operating within its normal range
   and no correction is needed.

---

## Answer Key

---

**Question 1 — Answer: C**

The inverting amplifier gain is -RF/RIN = -10k/1k = -10. With VIN = +1.5 V:

```
Vout = -10 × 1.5 = -15 V
```

For a ±15 V supply, -15 V is at the negative rail. Most op-amps cannot reach the supply
rail (typically 1-2 V headroom), so the output will saturate slightly above -15 V. The
calculation shows the design is close to saturation with a 1.5 V input and ×10 gain on
a ±15 V supply.

For bias current error minimisation: the compensation resistor at the non-inverting
input should equal the parallel combination of RF and RIN: RF ∥ RIN = (10k × 1k) /
(10k + 1k) = 909 Ω. This ensures the bias current flowing through each input sees the
same impedance and the resulting offset voltages cancel.

- D is incorrect: -15 V is at the supply rail, which is the saturation boundary, not
  the linear region.

---

**Question 2 — Answer: B**

The op-amp output voltage range specification is a hard limit. When the ideal output
would be 0.1 V (tracking the 0.1 V input), the op-amp can only output as low as 0.5 V.
Similarly, it can only output as high as 4.5 V. Signals outside this range are clipped.
Rail-to-rail output op-amps reduce this headroom to 10-50 mV, allowing output closer
to the supplies.

- A is incorrect: the unity-gain configuration does not extend the output range; the
  op-amp's output voltage range is a device characteristic independent of the feedback
  configuration.
- C is incorrect: latch-up in op-amps typically requires the input or output to exceed
  the supply voltage, not just approach the output range limit.
- D is incorrect: single-supply op-amps generally cannot swing to 0 V unless specifically
  rated as rail-to-rail output.

---

**Question 3 — Answer: C**

CMRR is defined as the ratio of differential gain to common-mode gain, expressed in dB.
A CMRR of 100 dB at 50 Hz means the common-mode rejection is 10^(100/20) = 100,000.
The common-mode-to-differential referred signal is:

```
VCMRR_input = VCM / CMRR = 2 V / 100,000 = 20 µV (referred to differential input)
```

Amplified by the gain of 100:

```
Vout_CM = 20 µV × 100 = 2 mV
```

This is the residual 50 Hz interference at the output.

- A is incorrect: the CMRR prevents common-mode gain. The INA does not amplify the
  common-mode signal by the differential gain of 100.
- B is incorrect: the INA has finite CMRR; perfect rejection is not achieved at any
  frequency.
- D results from a calculation error: 100 dB = 100,000 (not 1,000,000); 2 V / 100,000
  = 20 µV, not 1 µV.

---

**Question 4 — Answer: B**

ENOB (Effective Number of Bits) incorporates all noise and distortion mechanisms
(quantisation noise, thermal noise, jitter, harmonic distortion) into a single figure.
The achievable SNR based on ENOB is:

```
SNR = 6.02 × ENOB + 1.76 dB = 6.02 × 14.5 + 1.76 = 87.29 + 1.76 = 89.05 dB
```

The 1.5-bit reduction from ideal (16 bits) to effective (14.5 bits) represents noise
and distortion. This is the actual performance limitation.

- A is technically correct but unnecessarily verbose; the key answer is that ENOB gives
  the actual SNR and represents the converter's practical performance.
- C is incorrect: ENOB includes both linearity errors AND noise; it is not solely a
  linearity metric.
- D is incorrect: the +1.76 dB Schreier constant (derived from quantisation noise
  statistics of a full-scale sinusoid) applies to all real ADCs; it is not omitted for
  non-ideal converters.

---

**Question 5 — Answer: A**

In a transimpedance amplifier, the -3 dB bandwidth is set by the feedback RC:

```
f_-3dB = 1 / (2π × RF × CF) = 1 / (2π × 10⁶ × 10⁻¹²) = 159 kHz
```

The function of CF is to stabilise the amplifier. Without CF, the photodiode junction
capacitance CD creates a pole with the op-amp's open-loop impedance. This pole is inside
the amplifier's feedback loop and causes phase margin degradation, often leading to
oscillation. CF adds a zero in the feedback network at f = 1/(2π × RF × CF), which
reduces the loop gain at high frequencies and restores phase margin.

The optimal CF for stability while maximising bandwidth is:

```
CF = √(CD / (2π × RF × GBW))
```

- B incorrectly places CD in series with CF; CD is the photodiode capacitance at the
  virtual ground input node, not in series with the feedback.
- C is incorrect: the RC in the feedback definitely limits bandwidth; the op-amp GBW
  also limits bandwidth but the dominant constraint for a low-noise TIA with 1 MΩ
  feedback is the RF × CF product.
- D is incorrect: the bandwidth formula CD formula would apply only if there were no
  CF present (CD alone with RF sets a pole without CF).

---

**Question 6 — Answer: D**

A 12-bit DAC provides 2^12 = 4096 output levels. For a 3.3 V full-scale range:

```
1 LSB = 3.3 V / 4096 = 0.806 mV
LSB as percentage = 0.806 mV / 3300 mV = 0.024% of full scale
```

The accuracy requirement is 0.01% of full scale:

```
0.01% × 3.3 V = 0.33 mV
```

Since 1 LSB (0.806 mV) > the required accuracy (0.33 mV), the quantisation step is
larger than the required accuracy step. The DAC resolution is insufficient.

A 14-bit DAC: 1 LSB = 3.3 V / 16384 = 0.201 mV (0.006%) — adequate.

- A is correct in its calculation (0.024% per LSB) but incorrectly concludes the DAC
  is adequate; 0.024% exceeds the 0.01% requirement.
- B is incorrect: resolution and accuracy are distinct. Resolution defines the smallest
  possible output step; accuracy also includes nonlinearity, gain error, and offset.
- C is incorrect: a 12-bit DAC has 4096 levels, not 12.

---

**Question 7 — Answer: B**

The calculation proceeds from ADC code to temperature:

```
ADC LSB voltage = (2 × 2.048 V) / 2^24 = 4.096 V / 16,777,216 = 244 nV per LSB

After gain of 200 INA, 244 nV ADC LSB corresponds to:
  Signal voltage = 244 nV / 200 = 1.22 nV per LSB at the INA input

Thermocouple sensitivity = 5 µV/°C, so:
  Temperature per LSB = 1.22 nV / 5 µV per °C = 0.000244°C / LSB ≈ 0.24 m°C per LSB
```

This is the theoretical quantisation-limited resolution. Actual resolution may be
degraded by noise in the INA, the ADC, and the thermocouple cold-junction compensation.

- A and B arrive at the same numerical answer (0.24 m°C/LSB) through slightly different
  but equivalent calculation paths. Both are correct. B more explicitly shows the step
  from ADC LSB through gain to temperature.
- C is incorrect: multiplying by gain rather than dividing gives the wrong sign of the
  effect; gain of 200 improves resolution by 200×, not worsens it.
- D is not wrong in principle but is an incomplete calculation.

---

**Question 8 — Answer: B**

The total accuracy budget for a precision reference over a temperature range is the
root-sum-square (RSS) or worst-case sum of the contributing error terms. For a
conservative (worst-case) analysis:

```
Initial accuracy: ±0.1% = ±1000 ppm
Temperature drift: 5 ppm/°C × 125°C = 625 ppm = 0.0625%

Worst-case sum: ±0.1% + ±0.0625% = ±0.1625%
At 2.5 V: ±4.06 mV
```

The RSS combination (D) is used when errors are statistically independent, which they
may be in practice. But for a guaranteed worst-case design margin, the arithmetic sum
(B) is the safe approach.

- A is incorrect: temperature coefficient is an additional error source that must be
  added to the initial accuracy for operation over temperature.
- C is incorrect: initial accuracy is a fixed offset error that may be calibrated out
  at a single temperature, but the temperature drift cannot be calibrated unless the
  device is calibrated at every operating temperature.
- D (RSS) is used when the error sources are independent and random; for a systematic
  temperature drift error, worst-case sum is more appropriate.

---

**Question 9 — Answer: C**

The ADC has a full-scale range of 1 V and the sensor output is 0-100 mV. The sensor
signal uses only 10% of the ADC's input range. Each ADC output code represents:

```
Voltage per LSB = 1 V / 4096 = 0.244 mV (12-bit on 1 V FS)
```

The sensor spans 100 mV / 0.244 mV = 410 codes out of 4096. This is equivalent to
using only log2(410) = 8.7 bits of effective resolution for the 0-100 mV range.

The engineer should either use an ADC with a 0.1 V reference (or programmable gain
front end) to use the full 12-bit resolution for this signal range.

- A and B both correctly calculate that source loading error is negligible (< 0.005%)
  and identify it as not the primary concern; they agree that the range mismatch is the
  primary issue.
- D is incorrect: most ADCs function correctly when the input is well within the
  reference range; there is no conversion failure from a 100 mV signal on a 1 V reference.

---

**Question 10 — Answer: B**

An ideal integrator integrates any DC signal, including the input offset voltage. The
output ramps at:

```
dVout/dt = VOS / (RIN × CF) = 2 mV / (10 kΩ × 10 nF) = 2 × 10⁻³ / (10⁻⁴) = 20 V/s
```

Within 0.75 s, the output reaches a ±15 V supply rail. This is called "integrator
wind-up" or "latch-up" and is the fundamental problem with open-loop DC integrators.

Solution: Add a reset switch in parallel with CF (to discharge the integrator on demand),
or add a large resistor in parallel with CF (converting the ideal integrator into a
lossy integrator / first-order low-pass filter with a time constant RF × CF).

- A is incorrect: the feedback capacitor does NOT block DC in this configuration. In an
  integrator, DC at the input drives a charging current through the feedback capacitor
  indefinitely.
- C is incorrect: an integrator does not oscillate from a DC offset; it saturates.
- D is incorrect: the integrator output is not a stable DC value proportional to VOS;
  it ramps continuously.

---

**Question 11 — Answer: B**

Averaging N samples reduces the noise power by N (assuming white, uncorrelated noise),
which reduces the noise amplitude by √N. For a 6 dB improvement (factor of 2 in
amplitude, factor of 4 in power), N = 4 samples must be averaged.

```
SNR improvement = 20 log10(√N) = 10 log10(N)
For N = 4: improvement = 20 log10(2) = 6.02 dB ≈ 6 dB
```

This requires the sampling rate to be at least 4× the Nyquist rate for the signal
bandwidth (to have excess samples available for averaging).

- A is partially correct in mechanism (oversampling plus decimation) but states 3 dB
  per factor of two in sample rate. Oversampling by 4× (not 2×) gives 6 dB improvement.
  The phrasing "3 dB per factor of two" is the correct incremental rate, but the question
  asks for a 6 dB improvement, requiring 4× oversampling.
- C is correct that reducing bandwidth improves the noise floor (10 dB for 10× bandwidth
  reduction), but the question asks about averaging ADC samples, not digital filtering.
  Both approaches achieve noise floor improvement but through different mechanisms.
- D is incorrect: increasing the reference voltage increases the full-scale range and
  reduces the signal-to-full-scale ratio if the signal amplitude is unchanged — this
  worsens the SNR for a fixed-amplitude signal.

---

**Question 12 — Answer: A**

In a two-wire RTD configuration, the lead resistance RL is in series with the RTD
resistance RRTD. The measurement system cannot distinguish between the RTD resistance
and the cable resistance, so it measures RRTD + RL = 100 + 2 = 102 Ω. The PT100
temperature coefficient is approximately 0.385 Ω/°C near 0°C, giving:

```
Apparent temperature = 102 / 0.385 ≈ 264.9°C  (instead of 259.7°C for 100 Ω alone)
Error = 2 Ω / 0.385 Ω per °C = 5.19°C ≈ 5.2°C
```

This is a systematic offset error that remains constant if the cable resistance is
constant with temperature (it is not — copper resistance increases with temperature,
adding a secondary error). A four-wire (Kelvin) measurement eliminates this error by
using separate sense and source leads.

- B is incorrect: 2 mV across 100 mV is a 2% error — significant for precision
  temperature measurement, not negligible.
- C is incorrect: the error is a systematic offset, not a non-linearity. A constant
  current source does not eliminate the lead resistance offset.
- D is incorrect: two-wire is not standard for precision RTD measurements; it is
  adequate only when cable resistance is known and stable or the accuracy requirement
  allows for it.

---

**Question 13 — Answer: D**

The system noise floor is determined by the highest (loudest) noise source in the
signal chain. The microphone noise (-120 dBV) is quieter than the ADC noise (-105 dBV).
Therefore, the ADC noise dominates. Any signal below the ADC noise floor (-105 dBV)
cannot be recovered, regardless of the microphone's lower self-noise.

The total system noise floor is approximately:

```
Vtotal = √(Vmic² + VADC²)
       = √(10^(-120/10) + 10^(-105/10))  [power addition]
       ≈ Vmax term = -105 dBV (ADC dominates)
```

The ADC limits the system noise floor. To exploit the microphone's lower noise floor,
a higher-performance ADC is needed (ADC noise < -120 dBV).

- A and B both contain the correct identification (ADC limits performance) but arrive
  at it through different reasoning paths. D is the most precisely stated answer that
  correctly identifies the ADC as the limiting element.
- C is incorrect: 24-bit theoretical SNR ≈ 6.02 × 24 + 1.76 = 146 dB but this is the
  theoretical quantisation-limited floor; actual ADC noise floors are higher. The
  measured -105 dBV is the specified input-referred noise, not the theoretical limit.

---

**Question 14 — Answer: C**

The Sallen-Key filter in unity-gain configuration requires the op-amp to have sufficient
open-loop gain above the cutoff frequency to act as an ideal voltage follower. As the
op-amp's open-loop gain rolls off (at GBW / frequency Hz), the closed-loop gain
deviation from the ideal becomes:

```
At frequency f above cutoff: op-amp open-loop gain ≈ GBW / f = 1 MHz / f
```

For significant filter response degradation, a rule of thumb requires open-loop gain >
100 (40 dB) at the frequencies of interest. At f = 10 kHz, open-loop gain = 100 —
borderline. At f = 1 kHz (cutoff), gain = 1000 — adequate.

For a practical design with GBW = 1 MHz and cutoff = 1 kHz, the filter response is
accurate to within the passband and begins to deviate in the stopband above approximately
100 kHz where the op-amp's feedback correction is insufficient. For audio frequencies
(up to 20 kHz), a 1 MHz GBW op-amp is adequate.

- A is incorrect: the filter cutoff is not GBW divided by DC gain for a unity-gain stage.
- B is a reasonable practical estimate; above 100 kHz degradation becomes measurable
  for a 1 MHz GBW op-amp.
- D is incorrect: the op-amp's GBW absolutely affects the filter response. The passive
  components set the ideal filter shape, but the op-amp's gain limitations modify the
  actual response at higher frequencies.

---

**Question 15 — Answer: B**

A temperature coefficient of 50 ppm/°C over 100°C causes a resistance change of:

```
ΔR/R = 50 ppm/°C × 100°C = 5000 ppm = 0.5%
New resistance = 10 mΩ × 1.005 = 10.05 mΩ
```

At 10 A, the sense voltage becomes:

```
Vsense_new = 10 A × 10.05 mΩ = 100.5 mV (instead of 100 mV)
```

The amplifier output increases from 10 V to 10.05 V. The current reading appears as:

```
I_apparent = 10.05 V / (100 V/A gain) = 10.05 A (error of +50 mA)
```

This confirms option A and B give the same result (50 mA), but B shows the full
calculation correctly. The error is 0.5% of 10 A = 50 mA.

- C is incorrect: the calculation 0.005 Ω × 10 A = 0.05 V is the voltage error, not
  5 V. The amplifier gain of 100 gives 0.05 V × 100 = 5 V error only if the resistance
  change were 10× larger; the arithmetic in C is incorrect.
- D is incorrect: the temperature coefficient applies whenever the temperature deviates
  from the calibration temperature, regardless of the rated operating temperature.

---

## Related Topics

- [Op-Amp Circuits](../04_analogue_design/op_amp_circuits.md)
- [ADC/DAC Interfaces](../04_analogue_design/adc_dac_interfaces.md)
- [Sensor Signal Conditioning](../04_analogue_design/sensor_signal_conditioning.md)
- [Noise Analysis](../04_analogue_design/noise_analysis.md)
