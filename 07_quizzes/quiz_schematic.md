# Schematic Quiz

Fifteen multiple-choice questions covering component selection, power supply design,
decoupling strategy, and ESD protection. Each question has four options. Explanations
are provided in the Answer Key at the end.

---

## Questions

---

### Question 1

A 10 µF X7R MLCC rated at 6.3 V is used as a decoupling capacitor on a 3.3 V rail.
What is the most significant reason this capacitor may not provide 10 µF of effective
capacitance in circuit?

A) The X7R dielectric ages logarithmically after soldering, reducing capacitance by up
   to 3% per decade of time.

B) The capacitance of an X7R dielectric decreases significantly with applied DC voltage,
   and at 3.3 V on a 6.3 V-rated capacitor the effective capacitance may be reduced by
   40-60%.

C) X7R capacitors exhibit a piezoelectric effect that reduces their capacitance at the
   DC bias level.

D) The package inductance (ESL) of the 0402 package resonates with the capacitance at
   the 3.3 V frequency, reducing effective capacitance.

---

### Question 2

A MOSFET gate driver outputs 5 V. The design uses an N-channel MOSFET with VGS(th)
= 3.5 V (maximum) and a datasheet note that RDS(on) is specified at VGS = 10 V. What
is the primary concern?

A) The MOSFET may not turn off reliably because 5 V exceeds the absolute maximum VGS.

B) The MOSFET will turn on (VGS = 5 V > VGS(th) = 3.5 V maximum) but may not be fully
   enhanced, resulting in significantly higher RDS(on) than the datasheet value.

C) The MOSFET will overheat because the 5 V gate drive voltage exceeds the rated
   gate voltage.

D) The switching losses will be higher than calculated because the Miller plateau occurs
   at exactly 5 V.

---

### Question 3

Which capacitor dielectric is most appropriate for the load capacitors on a 25 MHz
crystal oscillator?

A) X7R, because its ±15% temperature coefficient ensures the oscillator remains within
   the PLL lock range over the operating temperature range.

B) Y5V, because it provides the highest capacitance density in the smallest package,
   minimising parasitic inductance.

C) C0G (NP0), because its capacitance is stable with temperature, voltage, and over
   time — each of which would otherwise load-pull the crystal frequency.

D) Z5U, because its high permittivity allows very small package sizes compatible with
   fine-pitch BGA devices.

---

### Question 4

A tantalum capacitor is specified on the input of a linear regulator. The rail voltage
is 5 V and the capacitor is rated at 6.3 V. During a standard ESD test, the rail
spikes to 8 V for 50 ns. What is the most likely failure mode?

A) The dielectric will absorb the 8 V spike without issue because tantalum dielectric
   has a recovery mechanism that self-heals minor overvoltage events.

B) The capacitor may fail short-circuit due to dielectric puncture, potentially causing
   thermal runaway and fire if the power source can supply sustained current.

C) The electrostatic discharge will be absorbed by the positive temperature coefficient
   of the tantalum dielectric, limiting peak current.

D) The capacitor will fail open-circuit, causing the downstream regulator to lose its
   input bypass and oscillate.

---

### Question 5

A synchronous buck converter operates at 400 kHz with 12 V input and 3.3 V output.
Which parameter is MOST important when selecting the high-side MOSFET?

A) Maximum drain-to-source voltage (BVDSS), because the converter must survive load dump
   transients.

B) Total gate charge (Qg), because at 400 kHz the switching losses in the high-side device
   dominate over conduction losses for a short duty cycle (27.5%).

C) On-resistance (RDS(on)), because the high-side device conducts for the majority of each
   switching period at this duty cycle.

D) Maximum continuous drain current (ID), because the high-side switch sees the full
   inductor current on every cycle.

---

### Question 6

A designer wants to decouple a 3.3 V microcontroller power pin. The datasheet recommends
a 100 nF capacitor within 1 mm of the power pin and a 10 µF capacitor within 5 mm. The
designer places both capacitors but positions them 8 mm from the IC. What is the most
likely consequence?

A) The capacitors will not charge at all because the trace resistance between the
   capacitor and the IC is too high for current to flow.

B) The decoupling is less effective because the inductance of the trace between the IC
   power pin and the capacitors increases the impedance at the switching frequencies,
   reducing the capacitors' ability to supply instantaneous current.

C) The design will work identically because the PCB dielectric between the supply and
   ground planes provides sufficient high-frequency decoupling.

D) The 10 µF bulk capacitor at 8 mm distance will resonate with the trace inductance at
   an audio frequency, causing acoustic noise from the PCB.

---

### Question 7

A board uses a TVS diode for ESD protection on a USB data line. The TVS is a bidirectional
15 V clamping device. USB 2.0 high-speed signals swing between 0 V and 400 mV
differentially. What is the most critical selection error in this design?

A) A TVS diode should not be used on USB lines; only ESD protection arrays are permitted
   by the USB specification.

B) The 15 V clamping voltage is far too high to protect the USB transceiver, which has
   absolute maximum ratings of 3.6 V. A clamping voltage of 3.6 V or less is required.

C) A bidirectional TVS is the wrong polarity for USB data lines, which only carry
   positive voltages.

D) The TVS junction capacitance will be acceptable because TVS devices designed for
   signal lines have capacitance below 0.5 pF.

---

### Question 8

A power supervisor monitors a 3.3 V rail and asserts a reset signal if the rail
drops below a threshold. The supervisor is set to assert reset at 3.0 V (91% of 3.3 V).
What problem can occur if the threshold is set too high (for example, 3.25 V)?

A) A false reset may be triggered by normal load transients causing momentary rail droop,
   interrupting operation unnecessarily.

B) The supervisor will draw excessive quiescent current at voltages close to the threshold
   because the internal comparator hysteresis is reduced.

C) The supervised IC may exceed its maximum undershoot specification and latch up.

D) The reset signal will have a slow edge because the comparator output slew rate is
   inversely proportional to overdrive voltage.

---

### Question 9

A designer proposes using a 1 Ω series resistor in series with a 100 µF bulk tantalum
capacitor on the output of a 5 V supply. What is the primary purpose of this resistor?

A) To reduce output voltage ripple by forming an RC low-pass filter with the tantalum
   capacitor.

B) To limit inrush current to the tantalum capacitor at power-up, protecting the
   dielectric from the high initial charging current that can cause failure.

C) To compensate for the ESR of the tantalum capacitor, improving the transient response
   of the upstream regulator.

D) To measure the capacitor current by monitoring the voltage drop across the resistor
   with an oscilloscope.

---

### Question 10

A 3.3 V-powered CMOS IC has its reset pin connected directly to a 5 V signal from
another device without any level shifting or current limiting. The 5 V device drives
the reset pin to 5 V when asserted. What is the most likely long-term consequence?

A) The reset pin will not function because the input threshold requires a logic-high
   voltage above 2.4 V; 5 V satisfies this requirement.

B) The input ESD protection diode will clamp the 5 V to VCC + 0.5 V = 3.8 V and
   inject current into the VCC rail, potentially stressing the diode and causing
   gradual degradation or immediate latch-up if injection current is high.

C) The logic level will be inverted because the input voltage exceeds the maximum
   rated logic-high voltage for CMOS inputs at 3.3 V VCC.

D) There will be no problem because CMOS inputs are high-impedance and will not draw
   meaningful current from the 5 V driver.

---

### Question 11

What is the function of the Miller plateau during MOSFET turn-on, and why is it
relevant to gate driver design?

A) The Miller plateau is the region where the gate capacitance charges to the final gate
   voltage. Its duration is determined solely by the gate driver's output impedance.

B) The Miller plateau is the period during turn-on when VGS is constant while VDS falls.
   The gate charge consumed during this period (Qgd) is the primary driver of switching
   loss and determines the required gate driver current for a given switching speed.

C) The Miller plateau occurs when the MOSFET enters the saturation region and the drain
   current is limited by the load. It causes a step increase in gate voltage that can
   be used to detect load current.

D) The Miller plateau is a region of negative resistance in the gate circuit that causes
   oscillation unless a gate resistor of at least 10 Ω is used.

---

### Question 12

A 100 Ω resistor rated 0.125 W is used to pull up a 3.3 V signal line. The line is
driven low by an open-drain driver. What is the power dissipation in the resistor when
the driver asserts the line low, and is the resistor correctly derated?

A) P = V²/R = (3.3)²/100 = 109 mW. This is within the 0.125 W rating, so the resistor
   is acceptable.

B) P = V²/R = (3.3)²/100 = 109 mW. This exceeds the recommended 50% derating limit of
   62.5 mW. A 0.25 W rated resistor should be used instead.

C) P = V/R = 3.3/100 = 33 mW. This is well within the rating and correctly derated.

D) P = 0 W when the line is driven low, because the driver shorts the resistor and
   no voltage appears across it.

---

### Question 13

A high-speed signal (risetime < 1 ns) is routed on a 3.3 V PCB without impedance
control, series termination, or attention to return paths. What phenomenon is most
likely to cause data errors?

A) Thermal noise from the trace resistance at room temperature, which has an RMS voltage
   of approximately 0.13 µV per √Hz — negligible at data rates below 10 GHz.

B) Reflections caused by impedance discontinuities along the unterminated trace, which
   can produce ringing on the signal edge that violates setup or hold time requirements.

C) Cross-talk from adjacent signal traces, which is the dominant error mechanism for all
   high-speed signals regardless of termination.

D) Skin effect increasing the trace resistance at high frequency to the point where the
   signal voltage is attenuated below the logic threshold.

---

### Question 14

A design requires monitoring three supply rails (3.3 V, 1.8 V, and 1.0 V) and
asserting a global reset if any rail drops below 90% of its nominal value. Which
implementation is MOST appropriate for a production design?

A) Monitor each rail with a separate comparator and a precision voltage reference;
   combine the comparator outputs with a three-input AND gate. This maximises accuracy
   but uses more components.

B) Use a dedicated multi-channel power supervisor IC (e.g., TI TPS3701) configured for
   each rail's threshold. These devices include hysteresis, timeout filtering, and
   open-drain outputs designed for exactly this purpose.

C) Use the MCU's internal ADC to measure each rail periodically and assert a reset in
   firmware if any reading is low. This eliminates external components.

D) Connect all three rails through a resistor divider network to a single comparator
   input; any rail dropping will reduce the combined voltage below the threshold.

---

### Question 15

A schematic shows a 0 Ω resistor (zero-ohm link) in series with a power rail. What
are two valid engineering reasons for including this component?

A) To provide a low-resistance connection that allows current measurement with a
   milliohm meter, and to allow the rail to be disconnected during bring-up by removing
   the link.

B) To provide a fuse function (the link will fail open on overcurrent) and to allow
   impedance matching on the power rail.

C) To provide pull-up/pull-down control for bus arbitration, and to act as a ferrite
   bead substitute at high frequency.

D) To compensate for voltage drop on long power traces, and to provide EMI filtering
   by introducing a known resistance.

---

## Answer Key

---

**Question 1 — Answer: B**

The DC bias voltage coefficient (voltage dependency of capacitance) is the dominant
effect for X7R capacitors on power rails. A 10 µF 6.3 V X7R at 3.3 V applied bias
can lose 40-60% of its nominal capacitance. This is the most significant and most
commonly overlooked effect.

- A is true (ageing does occur) but ageing loss (1-3% per decade) is far smaller than
  DC bias loss (40-60%) and therefore not the MOST significant reason.
- C is incorrect: the piezoelectric effect generates voltage in response to mechanical
  stress, it does not reduce capacitance at DC bias.
- D is incorrect: ESL causes the impedance to become inductive above the self-resonant
  frequency, but the capacitance value itself is not reduced by ESL in circuit.

---

**Question 2 — Answer: B**

A 5 V gate drive voltage exceeds the typical VGS(th) = 3.5 V (maximum), so the device
will turn on. However, RDS(on) is specified at VGS = 10 V. At VGS = 5 V, the MOSFET is
partially enhanced and its actual RDS(on) can be 3-10× higher than the datasheet value,
causing significant additional conduction loss and possible thermal failure.

- A is incorrect: 5 V does not exceed the absolute maximum VGS of a typical MOSFET
  (typically ±20 V). Concern about not turning off is not valid.
- C is incorrect: gate drive voltage does not directly cause overheating; conduction
  loss from insufficient enhancement does.
- D is incorrect: the Miller plateau occurs at the gate threshold of the transconductance
  region, not at the supply voltage level; it does not affect switching losses in the way
  described.

---

**Question 3 — Answer: C**

Crystal load capacitors must use C0G (NP0) dielectric. C0G has:
- Temperature coefficient: 0 ± 30 ppm/°C (effectively zero variation)
- No DC voltage coefficient
- No ageing
- No piezoelectric effect

Any variation in load capacitance directly load-pulls the crystal frequency. X7R (A)
has ±15% capacitance variation over temperature — this translates directly to a
frequency error at the extremes. Y5V (B) has +22/-82% variation and is entirely
unsuitable. Z5U (D) has +22/-56% variation; also unsuitable.

---

**Question 4 — Answer: B**

Tantalum capacitors fail short-circuit when their dielectric is punctured. An 8 V spike
on a 6.3 V-rated capacitor exceeds its voltage rating, risking dielectric breakdown.
Once punctured, a tantalum forms a low-resistance short. If the power source can supply
sustained current into this short, thermal runaway occurs and the component can ignite.

- A is incorrect: tantalum does NOT self-heal. It lacks the oxide regrowth mechanism of
  aluminium electrolytic capacitors.
- C is incorrect: tantalum does not have a positive temperature coefficient that limits
  current; its failure mode is destructive shorting.
- D is incorrect: tantalum failure is short-circuit, not open-circuit. A polymer tantalum
  fails open, but standard wet tantalum fails short.

---

**Question 5 — Answer: B**

At a duty cycle of 27.5% (3.3/12 = 0.275), the high-side switch conducts for only 27.5%
of each period. Its RMS current is IL × √D ≈ 0.52 × IL. At 400 kHz, the switching
losses are proportional to Qgd × Vds × Iload × fsw, and because the duty cycle is low,
the per-period conduction contribution is small compared to the switching events.
Therefore Qgd (and related Qg) is the primary selection criterion.

- A is correct that BVDSS must be verified (and is important), but it is not the MOST
  important parameter for selection among otherwise suitable candidates; it is a filter
  criterion, not the optimisation target.
- C is incorrect: the low-side switch (conducting 72.5% of the period) is where RDS(on)
  dominates. For the high-side at 27.5% duty, conduction loss is secondary.
- D is incorrect: maximum continuous ID is a filter criterion to ensure the device is
  not overloaded, not the primary selection parameter.

---

**Question 6 — Answer: B**

PCB trace inductance is approximately 0.8-1 nH/mm for typical geometry. At 8 mm, the
trace from the IC to the capacitors has approximately 6-8 nH of inductance. At the
frequencies where the IC creates current transients (100 MHz and above for digital ICs),
this inductance has an impedance of:

```
Z = 2π × f × L = 2π × 100 MHz × 7 nH ≈ 4.4 Ω
```

This effectively disconnects the capacitor from the IC at high frequency, making the
decoupling ineffective.

- A is incorrect: the trace resistance is only a few milliohms; this is not the issue.
- C is incorrect: the distributed capacitance of the PCB power plane is relevant but
  cannot replace a physically placed decoupling capacitor; and this design has no power
  plane (two-layer board implied by context).
- D describes a real phenomenon (acoustic resonance of X7R MLCCs) but it occurs in the
  audible band, not as a consequence of placement distance.

---

**Question 7 — Answer: B**

The USB 2.0 transceiver has an absolute maximum rating of approximately 3.6 V on its
data pins. A TVS clamping at 15 V provides no protection whatsoever against an ESD event
— the 15 V clamp voltage is far above the 3.6 V limit of the protected device. The TVS
will not conduct until the voltage reaches 15 V; by then the transceiver has been
destroyed.

The correct ESD protection device for USB 2.0 high-speed must have:
- Clamping voltage < 3.6 V
- Junction capacitance < 0.5 pF per pin pair (to avoid degrading signal integrity at
  480 Mbit/s)

- A is incorrect: TVS diodes and ESD protection arrays are both valid; there is no blanket
  prohibition on TVS for USB.
- C is incorrect: USB differential data pins carry both positive and negative differential
  voltages during normal operation, so a bidirectional device is appropriate.
- D is incorrect: many standard TVS devices for signal lines do have capacitance below
  0.5 pF; this is a valid design consideration, not the primary error.

---

**Question 8 — Answer: A**

If the supervisor threshold is set too close to the nominal rail voltage (e.g., 3.25 V
when the rail is 3.3 V), normal load transients — which commonly cause ±1-3% rail droop —
can momentarily push the rail below the threshold and assert reset. This creates spurious
resets during normal operation that are extremely difficult to diagnose, as they are
reproducible only under specific load conditions.

- B is incorrect: quiescent current of a supervisor is determined by its architecture,
  not by the proximity of the monitored voltage to the threshold.
- C describes a real concern (undervoltage can cause latch-up) but the question asks
  about the consequence of a threshold that is too HIGH, not too low.
- D is incorrect: the comparator slew rate is not significantly affected by the overdrive
  voltage in the range described.

---

**Question 9 — Answer: B**

Wet tantalum capacitors are vulnerable to destructive failure when subjected to high
inrush current at power-up. When a power supply first activates, the low ESR of the
tantalum capacitor and the low output impedance of the supply create a large inrush
current that can stress the dielectric and cause premature failure. A 1 Ω series
resistor limits this inrush:

```
I_max = V_rail / (R_series + ESR) = 5 / (1 + 0.05) ≈ 4.8 A  (with 1 Ω)
vs.  I_max = 5 / 0.05 = 100 A  (without 1 Ω)
```

- A (RC filtering) is technically true but is not the primary purpose; the 1 Ω value
  is far too small to provide meaningful filtering at useful frequencies.
- C is incorrect: adding external resistance degrades transient response, not improves it.
- D describes a valid use of a series resistor but at 1 Ω the voltage drop is too small
  for accurate current measurement; a shunt resistor with a known precision value is used
  for current measurement, not an arbitrary protection resistor.

---

**Question 10 — Answer: B**

Most 3.3 V CMOS ICs have ESD protection diodes from each I/O pin to VCC and GND.
When a 5 V signal is applied to a pin on a 3.3 V device, the protection diode from
the pin to VCC (3.3 V) becomes forward biased at V_pin − VCC − V_diode_forward =
5 − 3.3 − 0.6 = 1.1 V forward bias. This injects current into the VCC rail, which
can raise VCC above its rated voltage, stress the protection diode, and — if the
injection current exceeds the holding current of the parasitic PNPN structure — trigger
latch-up. At 5 V driven into 3.3 V CMOS, the injection current can be several milliamps
to tens of milliamps depending on the source impedance.

- A is partially true (the signal will be recognised as logic-high) but ignores the
  protection diode forward bias and its consequences.
- C is incorrect: the logic level interpretation is not inverted. CMOS inputs above
  the VIH threshold are logic-high regardless of whether that exceeds VCC.
- D is incorrect: the high impedance of the CMOS input applies in the linear region,
  but once the protection diode is forward biased, current flows through a low-impedance
  path.

---

**Question 11 — Answer: B**

During MOSFET turn-on, the gate charge profile has three regions. In the plateau region,
VGS is held constant by the Miller effect while VDS falls (as the drain voltage transitions
from the input bus voltage to the output voltage). During this period, the gate driver
current is entirely consumed by charging the Miller capacitance (Cgd). The plateau duration
is Qgd / IG_driver. Shortening the plateau (by increasing driver current or reducing
gate resistance) reduces switching loss.

- A is incorrect: the plateau is not where VGS charges to its final voltage; VGS is
  constant during the plateau, which is the defining characteristic.
- C is incorrect: the drain current is determined by load inductance during the plateau,
  not by MOSFET saturation; and the plateau does not produce a step in gate voltage.
- D is incorrect: while gate resistors are used to control switching speed and prevent
  ringing, the Miller plateau itself is not a negative resistance phenomenon.

---

**Question 12 — Answer: B**

When the open-drain driver pulls the line low, the full 3.3 V appears across the resistor.
Power dissipation is:

```
P = V² / R = (3.3)² / 100 = 10.89 / 100 = 0.109 W = 109 mW
```

Industry standard derating for resistors is 50% of rated power (MIL-STD-975, industry
practice). The 50% derating limit of a 0.125 W resistor is 62.5 mW. The 109 mW
dissipation exceeds this limit by 74%. A 0.25 W resistor derated to 50% = 125 mW
threshold is marginal; a 0.5 W resistor derated to 50% = 250 mW would be the correct
specification.

Additionally, if this is a bus line (like I2C SCL) with many drivers and receivers,
the duty cycle of the low state and ambient temperature must also be considered.

- A is incorrect: 109 mW is within the absolute rated power but violates the derating
  rule, which is the engineering standard, not the absolute limit.
- C is incorrect: the formula P = V/R is dimensionally incorrect; power is V²/R or I²R.
- D is incorrect: when the driver asserts low, the driver holds the output near 0 V but
  current flows through the resistor from VCC through the resistor to the driver output.
  The voltage across the resistor is VCC minus the driver saturation voltage ≈ 3.3 V.

---

**Question 13 — Answer: B**

A signal with a risetime of 1 ns has spectral content up to approximately 0.35/1 ns =
350 MHz. On an uncontrolled-impedance trace, the signal encounters impedance
discontinuities (connector transitions, via stubs, changes in reference plane). At each
discontinuity, a portion of the signal is reflected. The reflected wave travels back
toward the source and forward again, creating ringing on the received waveform. For a
high-speed parallel interface, this ringing can violate setup or hold time margins.

- A is incorrect: thermal noise at room temperature is approximately 0.9 nV/√Hz at a
  100 Ω source; for a 350 MHz bandwidth the integrated noise is ~17 µV RMS — negligible
  compared to the signal amplitude of hundreds of millivolts.
- C is incorrect: cross-talk is a real concern but is secondary to reflections in this
  scenario; unterminated impedance discontinuities are the immediate consequence of the
  stated design.
- D is incorrect: skin effect increases attenuation at high frequency, but for trace
  lengths of a few centimetres at 350 MHz, attenuation is typically less than 1 dB —
  not sufficient to bring the signal below the logic threshold.

---

**Question 14 — Answer: B**

A dedicated multi-channel power supervisor IC is the appropriate choice for a production
design. These devices are specifically engineered for rail monitoring with:
- Factory-calibrated threshold voltages with tight accuracy (typically ±1.5-2%)
- Integrated hysteresis to prevent oscillation near the threshold
- Timeout filter to reject short glitches and prevent false resets
- Open-drain output compatible with multi-source wired-AND reset structures
- Low quiescent current and small package

- A is technically valid but uses more components (three comparators + reference + gate)
  with higher component count, more variation in timing, and more design effort.
- C (MCU firmware monitoring) is fundamentally flawed: if the supply rail has drooped
  to a fault level, the MCU itself may be operating incorrectly and cannot be relied upon
  to detect and respond to the fault. A hardware supervisor is independent of firmware.
- D is incorrect: averaging three rails into a single comparator would allow one healthy
  rail to mask a failed rail. Each rail must be monitored independently against its own
  threshold.

---

**Question 15 — Answer: A**

A 0 Ω resistor (zero-ohm link) on a power rail serves two legitimate engineering purposes:

1. **Current measurement point:** By temporarily replacing the 0 Ω link with a known
   shunt resistor (e.g., 10 mΩ), the rail current can be measured by monitoring the
   voltage drop. The 0 Ω position is designed for this purpose.

2. **Isolation jumper:** The 0 Ω link can be removed during bring-up to isolate a
   downstream sub-circuit, allowing each power domain to be brought up incrementally.

- B is incorrect: a 0 Ω resistor is not a fuse — it does not fail reliably at a specified
  current. A fuse is a rated, controlled-failure device. 0 Ω links and impedance matching
  are unrelated functions.
- C is incorrect: pull-up/pull-down control and ferrite bead substitution are functions
  of specific component values, not zero-ohm links.
- D is incorrect: a 0 Ω resistor contributes negligible resistance and no EMI filtering
  by definition.

---

## Related Topics

- [Component Selection](../01_schematic_design/component_selection.md)
- [Power Supply Architecture](../01_schematic_design/power_supply_architecture.md)
- [Decoupling Strategy](../01_schematic_design/decoupling_strategy.md)
- [ESD Protection](../01_schematic_design/esd_protection.md)
- [Design for Test](../01_schematic_design/design_for_test.md)
