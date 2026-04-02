# Bring-Up Quiz

Fifteen multiple-choice questions covering board bring-up methodology, power-on sequence
debugging, oscilloscope and thermal imaging techniques, rework procedures, ECN process,
EMC fundamentals, and safety compliance. Each question has four options. Explanations
are provided in the Answer Key at the end.

---

## Questions

---

### Question 1

A new PCB is powered for the first time using a bench supply set to 5 V with a 200 mA
current limit. The supply immediately current-limits, the voltage drops to 1.8 V, and
the current meter reads 200 mA. No smoke or burning smell is observed. What is the
correct first diagnostic step?

A) Increase the current limit to 1 A to allow the board to complete its power-up sequence,
   then measure all rails with a multimeter.

B) Remove power immediately. With the board unpowered, measure the resistance from the
   5 V input rail to ground with a multimeter to determine whether a short or low-impedance
   fault is present before reapplying any power.

C) Probe each power rail with an oscilloscope while the supply is current-limiting to
   determine which rail is faulted.

D) Replace the bench supply with the board's intended wall adapter at 5 V/2 A to provide
   adequate current and observe whether the board powers on normally.

---

### Question 2

A board bring-up checklist requires measuring resistance from each power rail to ground
before applying power. The 3.3 V rail measures 18 Ω. The 1.8 V rail measures 320 Ω.
The 5 V rail measures 4.7 kΩ. Which reading warrants further investigation before
power is applied?

A) The 5 V rail at 4.7 kΩ — this is too high, suggesting an open circuit in the power
   path.

B) The 3.3 V rail at 18 Ω — this is suspiciously low and may indicate a short or
   overly low-impedance fault that will cause excessive current draw on power-up.

C) The 1.8 V rail at 320 Ω — this is the expected reading for a rail with a large number
   of decoupling capacitors and no ICs powered.

D) All three readings are within normal range for a populated board.

---

### Question 3

During board bring-up, you observe that a 1.0 V core rail powers up correctly when
tested in isolation, but when the full sequencing circuit is active, the 1.0 V rail
rises to 0.8 V and then collapses. The 1.8 V upstream rail remains at the correct
voltage throughout. What is the MOST likely cause?

A) The 1.8 V upstream rail voltage is too low, starving the 1.0 V buck converter's
   input.

B) The 1.0 V rail output capacitance is too small, causing the output to droop under
   load.

C) The 1.0 V buck converter is triggering its overcurrent protection during start-up,
   either because the load inrush exceeds the current limit or because the soft-start
   ramp is too fast for the output capacitance.

D) The PMIC enable signal for the 1.0 V rail is being driven by the 1.8 V PGOOD line,
   which asserts before the 1.8 V rail has fully stabilised.

---

### Question 4

A processor requires power supply sequencing where the 1.0 V core must rise before the
1.8 V I/O supply. To verify this during bring-up, you set up a four-channel oscilloscope.
Describe the MINIMUM measurement configuration.

A) Connect one oscilloscope channel to the 1.0 V rail and another to the 1.8 V rail.
   Trigger on the 1.0 V rising edge. Capture both waveforms and verify that the 1.0 V
   rail begins to rise before the 1.8 V rail.

B) Connect one channel to the PGOOD output of the 1.0 V regulator and another to the
   ENABLE input of the 1.8 V regulator. Trigger on PGOOD. Verify the ENABLE signal
   is asserted after the PGOOD signal.

C) Connect all four channels to the four supply rails on the processor (VCCCORE, VCCIO,
   VCCPLL, VCCRESET). Trigger on the first rising edge. Verify all four rails rise in
   the correct order.

D) Use a single oscilloscope channel with a differential probe from the 1.0 V rail
   to the 1.8 V rail. A positive voltage at the start of the capture confirms the
   1.0 V rail rose first.

---

### Question 5

An oscilloscope measures 80 mV peak-to-peak ripple on a 3.3 V switching regulator
output at the regulator's switching frequency. The measurement is taken with the probe
tip at a test point 15 mm from the output capacitor, with the ground clip attached to
a ground pin 25 mm away. What is wrong with this measurement setup, and what is the
corrected value likely to be?

A) The probe is at a 10:1 attenuation setting; the actual ripple is 800 mV. Switching
   to a 1:1 probe setting gives the correct reading.

B) The long ground clip introduces inductance (~25 nH) that resonates with the probe
   tip capacitance, adding a ringing artefact to the measurement. The probe should be
   fitted with a short ground spring and positioned at the output capacitor terminals.
   The true ripple is likely 30-50 mV.

C) The 25 mm ground clip is adequate for all frequencies below 10 MHz; since the
   switching frequency of the regulator is typically below 10 MHz, the measurement
   is correct.

D) The test point placement 15 mm from the capacitor adds trace inductance that increases
   the apparent ripple; the true ripple at the capacitor is higher than 80 mV.

---

### Question 6

A JTAG connection to a new board returns an unexpected device ID. The ID corresponds
to a completely different microcontroller family from the one on the board. What are the
two MOST likely explanations?

A) The JTAG TCK frequency is too high for the device; reduce TCK and retry, and check
   whether the device was damaged during ESD handling.

B) The JTAG scan chain is not connected correctly (TDO and TDI may be swapped), or a
   wrong device is populated on the board (incorrect component from the BOM or assembly
   error placing the wrong IC in the correct footprint).

C) The JTAG adapter power supply is different from the I/O voltage of the target device,
   causing logic level translation errors.

D) The device requires a specific boot mode pin configuration before JTAG enumeration;
   verify the boot mode pins are in their correct state.

---

### Question 7

During bring-up, a thermal camera image taken 60 seconds after power-on shows a 0402
ceramic capacitor on the 5 V input rail at 78°C while all adjacent components are at
32°C. The ambient temperature is 25°C. The component is rated at 125°C maximum. What
is the correct action?

A) Continue the bring-up; 78°C is below the component's rated maximum of 125°C and
   represents acceptable operation.

B) Remove power immediately. A 0402 ceramic capacitor under normal conditions should
   not exceed ambient by more than a few degrees Celsius. A 53°C rise above ambient
   on a passive component indicates it is dissipating anomalous power — most likely due
   to being shorted internally or being over-rated in voltage. Investigate before
   re-applying power.

C) Apply a freeze spray to the capacitor to cool it, then power on again and monitor
   whether the failure recurs.

D) Replace the capacitor with a higher-rated 150°C version before restarting bring-up,
   as this type of temperature rise is normal for input rail decoupling capacitors under
   transient conditions.

---

### Question 8

A rework engineer proposes to fix a net connectivity error (a signal trace connected
to the wrong pad) on a prototype board by cutting the trace with a scalpel and
attaching a 30 AWG PTFE wire to bridge the correct connection. The proposed rework
has not been written up as an ECN. Under what circumstance is proceeding without an
ECN acceptable?

A) When the rework is minor (less than two cut traces and one wire) and the engineer
   is experienced with PCB rework.

B) When the board is the only available prototype and the project timeline requires
   the fix to be tested today; the ECN can be written retroactively once the fix
   is confirmed to work.

C) It is never acceptable to perform rework on a board that will be used for any
   formal test or evaluation without an ECN. However, for an informal early-stage
   prototype that will not be used for formal testing, documentation in the engineering
   notebook (date, description of change, photograph) is a minimum acceptable substitute.

D) Rework without an ECN is acceptable as long as the engineer verbally informs their
   manager and another team member before beginning.

---

### Question 9

A product fails FCC Part 15 Class B radiated emissions at 96 MHz. The PCB contains
a microcontroller clocked at 96 MHz with a crystal oscillator. A spectrum scan shows
a broadband elevation from 100 MHz to 500 MHz in addition to the 96 MHz fundamental
tone. What two design changes should be investigated first?

A) Add a ferrite bead in series on the crystal oscillator's power supply pin, and
   replace the crystal with a TCXO (temperature-compensated crystal oscillator).

B) Add a 33 Ω series resistor at the MCU clock output pin to reduce the edge rate and
   harmonic content, and ensure a continuous ground plane under the clock trace with
   no breaks or slots.

C) Increase the clock frequency to 100 MHz to move the fundamental tone away from
   the FCC test boundary at 96 MHz.

D) Apply a ferrite shield can over the MCU and route all clock traces inside this
   shielded area.

---

### Question 10

A conformal-coated PCB is being reworked to replace a failed component. The coating
is identified as an acrylic type. What is the correct preparation step before
desoldering the failed component?

A) The conformal coating does not need to be removed before desoldering SMT components;
   the reflow heat during removal will soften the acrylic and it will self-clear.

B) Apply isopropyl alcohol (IPA) to the coated area and allow it to soak for 60 seconds;
   the acrylic will absorb the IPA and soften, allowing removal with a cotton swab before
   desoldering.

C) Remove the acrylic conformal coating from the rework area using an appropriate solvent
   (MEK or a dedicated acrylic stripper) applied with a swab, clearing the coating from
   the component and its adjacent pads. Then proceed with desoldering.

D) Use a hot air gun at 200°C to burn off the acrylic coating from the rework area
   before desoldering the component.

---

### Question 11

A creepage distance measurement on a new mains-operated PCB design shows 6.5 mm between
the live conductor and a secondary-side conductor. The design targets reinforced insulation
per IEC 62368-1 for a 250 Vrms working voltage on standard FR4 (Group IIIa). Is this
distance compliant?

A) Yes — 6.5 mm exceeds the IEC 62368-1 basic insulation requirement of 4.0 mm for
   Group IIIa at 250 Vrms.

B) No — IEC 62368-1 reinforced insulation at 250 Vrms for Group IIIa requires 8.0 mm
   (basic insulation 4.0 mm × 2). The 6.5 mm distance is non-compliant for reinforced
   insulation.

C) Yes — 6.5 mm is compliant because it exceeds the 5.0 mm clearance requirement for
   reinforced insulation.

D) The distance is compliant only if a conformal coating is applied over the entire
   board, which provides additional insulation credit.

---

### Question 12

A product incorporates a switching power supply module and a microcontroller board
as separate PCBs connected by a cable. During radiated EMC pre-compliance testing,
emissions at the switching frequency are found primarily when the cable is connected.
What is the most likely mechanism and the most effective mitigation?

A) The cable shield is acting as an antenna, radiating energy from differential-mode
   noise on the power supply output. Add a Y-capacitor from the shield to chassis.

B) The cable is acting as a common-mode (CM) antenna, driven by the voltage difference
   between the power supply's output ground and the chassis/earth reference. The most
   effective mitigation is a common-mode choke on the cable, placed at the exit point
   from the power supply enclosure, combined with a Y-capacitor to chassis at the
   same point.

C) The cable is radiating differential-mode noise from the microcontroller's clock signal.
   Add a ferrite bead in series on each signal conductor within the cable.

D) The emissions occur because the cable is too long; shorten the cable to below
   λ/4 at the switching frequency to prevent antenna resonance.

---

### Question 13

A board uses tantalum capacitors on several supply rails. The board design was
completed five years ago. A junior engineer notices the original tantalum capacitors
are no longer available and proposes substituting polymer electrolytic capacitors of
the same capacitance and voltage rating. What is the MOST significant difference
the design engineer must evaluate before approving the substitution?

A) Polymer electrolytic capacitors have lower ESR than wet tantalum capacitors. In
   an LDO circuit, the reduced ESR may eliminate the zero in the control loop that
   the original design relied upon for stability, potentially causing the LDO to
   oscillate.

B) Polymer electrolytic capacitors have a higher rated voltage; the existing voltage
   derating may be excessive for the new components.

C) Polymer electrolytic capacitors have a different physical size; the courtyard
   on the PCB may not accommodate them.

D) Polymer electrolytic capacitors are not RoHS compliant and cannot be substituted
   without a formal REACH assessment.

---

### Question 14

An ECN is raised to correct a wrong resistor value in the feedback divider of a buck
converter, which causes the output to be 3.4 V instead of the intended 3.3 V. The
corrective rework involves removing the top feedback resistor (R23, currently 100 kΩ)
and replacing it with a 95.3 kΩ resistor. Which ECN field is MOST critical for
ensuring the rework is performed correctly on the target boards?

A) The date of creation, because it establishes traceability to the point in the
   programme timeline when the error was discovered.

B) The affected board serial numbers, because applying the rework to an incorrect
   board (one already at the correct specification) would introduce an error where
   none existed.

C) The approver's name and signature, because this confirms the change has been
   reviewed and the engineering correctness has been verified.

D) The reason field, because documentation of the root cause prevents the same error
   in future designs.

---

### Question 15

A medical device PCB must meet IEC 60601-1 for a Type B patient-applied part with a
230 Vrms mains input. The board uses a transformer-isolated power supply. A Y-capacitor
(Y1 class, 4.7 nF) connects the primary earth to the secondary circuit for EMC purposes.
A safety engineer flags this as a potential compliance issue. What is the concern?

A) Y1-class capacitors are not rated for 230 Vrms mains; a Y2-class capacitor is
   required for compliance.

B) The Y-capacitor creates a leakage current path from mains to the secondary circuit
   and ultimately to the patient-applied part. IEC 60601-1 limits patient leakage
   current to ≤ 0.1 mA (NC) for Type B parts. At 230 Vrms and 50 Hz, a 4.7 nF Y-cap
   passes I = 2π × 50 × 4.7 nF × 230 ≈ 340 µA — exceeding the 0.1 mA patient leakage
   limit for a patient-connected circuit.

C) The Y-capacitor violates the reinforced insulation requirement between primary and
   secondary because it provides a direct electrical path through the isolation barrier.

D) Y-capacitors on the primary side are prohibited in IEC 60601-1; only CM chokes
   are permitted for primary-side conducted EMC attenuation in medical equipment.

---

## Answer Key

---

**Question 1 — Answer: B**

Removing power and measuring resistance is the correct first step. The current limit
has already protected the board from damage; applying more current (A) risks damaging
components by supplying sustained fault current. Probing live with an oscilloscope (C)
prolongs the fault condition. Using a wall adapter (D) removes the current-limiting
protection and risks catastrophic damage. The resistance measurement with power removed
is safe, non-destructive, and definitively identifies whether a fault exists and its
approximate impedance.

---

**Question 2 — Answer: B**

18 Ω from the 3.3 V rail to ground is unusually low. A healthy 3.3 V rail with MLCC
decoupling capacitors, IC power pins, and pull-down resistors typically measures 50 Ω
or more. An 18 Ω reading suggests either an abnormally large capacitance bank (which
would look like low resistance to a DC meter immediately after measurement begins),
a shorted capacitor, a component failure, or a solder bridge.

The 1.8 V rail at 320 Ω and the 5 V rail at 4.7 kΩ are both plausible for rails with
a small number of capacitors. The 3.3 V rail requires investigation.

---

**Question 3 — Answer: C**

The symptom — rail rises partway then collapses — is the signature of overcurrent
protection triggering. Specifically, the 1.0 V rail collapses to zero (the converter
shuts down or enters hiccup mode) after partially rising. In isolation the rail works,
meaning the converter is correctly configured. Under sequencing with the full load
(including the SoC or FPGA), the inrush current at start-up exceeds the current limit
of the 1.0 V converter. The soft-start ramp rate is too fast for the combination of
output capacitance and load inrush, causing OCP.

- A is ruled out because the 1.8 V rail "remains at correct voltage throughout."
- B is ruled out because a too-small output capacitor would cause droop under load,
  not a collapse at start-up before the load is active.
- D describes a possible sequencing issue but would more likely cause the rail not to
  start at all, rather than partially rising.

---

**Question 4 — Answer: A**

The minimum measurement configuration to verify sequencing is one channel per rail,
triggered on the first rail to rise. Channels 1 and 2 on the 1.0 V and 1.8 V rails
respectively, with a trigger on the 1.0 V rising edge, captures whether 1.0 V begins
to rise before 1.8 V. This is sufficient for basic sequence verification.

- B verifies the sequencing circuit control signals, which is useful for debugging
  but does not directly verify that the actual power rails are in sequence.
- C is more comprehensive but requires four available channels; the question asks for
  the MINIMUM.
- D is technically creative but a differential measurement between two rails only shows
  their relative value at each moment, not their individual absolute levels; this can
  be misinterpreted if both rails start simultaneously at the same rate.

---

**Question 5 — Answer: B**

A 25 mm ground clip has inductance of approximately 25 nH. Combined with the probe
tip capacitance of ~15 pF, this forms a resonant circuit at:

```
f = 1 / (2π × √(25 nH × 15 pF)) ≈ 232 MHz
```

At frequencies near this resonance, the probe ground return has very high impedance,
adding voltage noise to the measurement that appears as ringing. At a switching
frequency of 1-2 MHz, the fundamental is measured reasonably accurately, but harmonics
(at 5-10 MHz and above) are distorted. The ringing artefact from the long ground lead
adds to the apparent ripple amplitude.

The correct setup uses a short ground spring (L < 5 nH) directly at the output capacitor
terminals. The true ripple at the capacitor is typically lower because the test point
also includes the trace impedance between capacitor and test point.

---

**Question 6 — Answer: B**

An incorrect device ID from JTAG most likely indicates one of two problems: a wrong
device is physically on the board (assembly error — wrong IC in the correct footprint),
or the JTAG scan chain connections are incorrect (TDI and TDO swapped, causing the
shift register to read an unexpected pattern). Both should be checked: visually inspect
the IC against the BOM and schematic, and verify the JTAG pin connections.

- A (TCK frequency) affects scan chain reliability but typically causes scan failures
  or timeouts, not a coherent but incorrect device ID.
- C (logic level mismatch) causes JTAG communication errors but not a coherent wrong ID.
- D (boot mode) affects the device's behaviour after enumeration but not the JTAG device
  ID which is read from a hard-coded register.

---

**Question 7 — Answer: B**

A ceramic capacitor (0402 MLCC) under normal decoupling duty — passing AC currents
with negligible DC current — should not heat detectably above ambient. Its ESR at
typical switching frequencies is on the order of milliohms, producing microwatts of
heat. A 53°C rise above ambient indicates the capacitor is dissipating meaningful power,
most likely because it has failed short-circuit. A failed-short MLCC draws current from
the 5 V supply, heating through I²R in its residual resistance.

Continuing to operate a suspected shorted capacitor risks thermal runaway (particularly
for tantalum; less for ceramic, which typically fails gracefully) and will not provide
diagnostic information — the fault source is already identified. Remove power and
investigate.

---

**Question 8 — Answer: C**

For an informal early-stage prototype that will not be used for formal regulatory
testing, formal customer evaluation, or CE/FCC testing, a lightweight documentation
approach (engineering notebook entry with date, description, and photograph) is
acceptable as a minimum. However, the practice of documenting changes must be maintained.

A, B, and D all represent rationalised non-documentation: these approaches leave no
reliable record if the board is later used for formal testing or if the rework causes
a secondary fault. A board with undocumented rework cannot be used for any formal
test activity.

---

**Question 9 — Answer: B**

Two evidence-based first actions for an MCU clock causing broadband and fundamental-tone
emissions:

1. **Series resistor at the clock output:** Adding 22-47 Ω in series at the MCU clock
   output pin slows the edge rate (increases rise/fall time), which directly reduces
   harmonic content above the fundamental. The fundamental at 96 MHz is unaffected in
   frequency; only the harmonic amplitudes decrease.

2. **Ground plane continuity:** Broadband elevation from 100-500 MHz is a classic
   signature of a clock trace crossing a ground plane slot or routing over a split,
   which increases the antenna loop area across all harmonics simultaneously. Restoring
   ground plane continuity under the clock trace is the most effective structural fix.

- A (ferrite on VDD and TCXO replacement) addresses power supply noise injection into
  the oscillator but does not address the primary radiation mechanism.
- C is incorrect: moving the clock frequency does not reduce emissions; it moves the
  problem frequency.
- D (shield can) is a valid late-stage mitigation but is expensive and is not the
  "first investigation" — root cause analysis should precede shielding.

---

**Question 10 — Answer: C**

Acrylic conformal coating is solvent-removable. The correct solvent strippers for acrylic
are MEK (methyl ethyl ketone), acetone, or a purpose-formulated acrylic conformal
coating remover. The coating is removed from the rework area by applying solvent with
a swab and allowing it to act, then clearing the softened coating. The board must then
be cleaned and dried before desoldering and re-coating.

- A is incorrect: reflow heat does not reliably clear acrylic and may cause it to bubble
  and flow onto adjacent areas, causing contamination.
- B is partially correct (IPA softens some acrylics) but IPA is a less effective acrylic
  stripper than MEK or dedicated strippers; the 60-second soak is too short for most
  formulations.
- D is incorrect: burning conformal coating with a hot air gun at 200°C releases toxic
  fumes and may damage the PCB and adjacent components.

---

**Question 11 — Answer: B**

IEC 62368-1 reinforced insulation is defined as providing isolation equivalent to
double the basic insulation distance. For 250 Vrms working voltage, Group IIIa (FR4),
Pollution Degree 2:

```
Basic insulation creepage = 4.0 mm
Reinforced insulation creepage = 4.0 × 2 = 8.0 mm
```

6.5 mm is compliant for basic insulation (> 4.0 mm) but is NOT compliant for reinforced
insulation (< 8.0 mm). For a mains-operated product where the secondary output is
accessible by users, reinforced insulation is required between mains and the accessible
conductors.

- A is correct that 6.5 mm exceeds basic insulation but the design requires reinforced.
- C quotes incorrect distances; there is no 5.0 mm requirement for reinforced at 250 Vrms.
- D is incorrect: conformal coating is not credited as insulation in IPC-2221 or
  IEC 62368-1 creepage calculations.

---

**Question 12 — Answer: B**

When a cable connects two PCBs, the cable can act as a common-mode antenna if there
is a voltage difference (at the switching frequency or its harmonics) between the board
ground and the chassis earth at the cable connection point. This is the most common
cable-related EMC failure mode. The CM choke attenuates the common-mode current without
affecting differential (signal) current; the Y-capacitor provides a low-impedance CM
current return path to chassis at the cable entry point.

- A (differential mode) is possible but less common for switching frequencies appearing
  when a cable is connected.
- C may contribute but is a secondary mechanism; the signal conductors are at lower
  amplitude than the power switching noise.
- D shortening the cable is sometimes effective but is not a design solution; the
  root cause is common-mode current, not antenna length per se.

---

**Question 13 — Answer: A**

Many LDO voltage regulators use a minimum ESR specification for their output capacitor
to maintain loop stability. The LDO error amplifier zero, created by the output
capacitor's ESR, provides phase boost that maintains phase margin. Wet tantalum
capacitors have ESR in the range 50-500 mΩ; polymer electrolytics have ESR as low
as 5-50 mΩ. If the LDO stability condition requires ESR > 50 mΩ and the polymer
electrolytic provides only 20 mΩ, the LDO may oscillate or have inadequate phase margin.

Always verify the LDO datasheet's stability requirements before approving a capacitor
substitution, particularly any change in ESR.

- B is not the most significant concern: lower ESR components do not cause derating
  issues; derating applies to voltage, not ESR.
- C (physical size) is a valid check but is a straightforward verification, not the
  most significant engineering risk.
- D is incorrect: polymer electrolytic capacitors are generally RoHS compliant; the
  concern raised is not substantiated.

---

**Question 14 — Answer: B**

The most critical field for preventing the rework from being applied to the wrong board
is the **affected board serial numbers**. If the rework is applied to a board that is
already at the correct specification (perhaps from a later batch where the BOM was
already corrected), it would introduce an error. The serial number list ensures that
only the affected boards receive the modification.

- A (date) is important for traceability but does not prevent incorrect application.
- C (approver) is essential for the ECN process but does not identify which boards to rework.
- D (reason) is important for root-cause documentation and prevention in future designs
  but does not directly ensure the rework is applied to the correct boards.

---

**Question 15 — Answer: B**

IEC 60601-1 imposes strict patient leakage current limits. For a Type B patient-applied
part, the maximum patient leakage current (Normal Conditions) is 0.1 mA. The leakage
current through a Y-capacitor at mains frequency is:

```
I = V × 2π × f × C = 230 × 2π × 50 × 4.7 × 10⁻⁹ = 0.34 mA
```

This 0.34 mA exceeds the 0.1 mA limit. To comply, the Y-capacitor value must be reduced
or eliminated, or the circuit topology must route the leakage current to chassis ground
rather than to the patient-connected secondary circuit.

```
Maximum allowable C for Type B: I_max / (V × 2π × f) = 0.1 mA / (230 × 314) = 1.38 nF
```

The 4.7 nF capacitor exceeds the maximum permissible value by 3.4×.

- A is incorrect: Y1-class capacitors are rated for operation across primary to earth
  at full mains voltage; a Y1 is more robust than Y2, not less.
- C is incorrect: the Y-capacitor provides EMC function (CM noise bypass) and is
  explicitly permitted in most safety standards through rated Y-capacitors; it is the
  leakage current, not the presence of an electrical path, that is the concern.
- D is incorrect: Y-capacitors are permitted in IEC 60601-1 primary circuits; the
  restriction is on the leakage current magnitude, not on the component type.

---

## Related Topics

- [Bring-Up Methodology](../05_board_bringup/bringup_methodology.md)
- [Power-On Sequence](../05_board_bringup/power_on_sequence.md)
- [Debugging Techniques](../05_board_bringup/debugging_techniques.md)
- [Rework and ECN](../05_board_bringup/rework_and_ecn.md)
- [EMC Basics](../06_standards_and_compliance/emc_basics.md)
- [Safety and Creepage](../06_standards_and_compliance/safety_and_creepage.md)
- [Conformal Coating](../06_standards_and_compliance/conformal_coating.md)
