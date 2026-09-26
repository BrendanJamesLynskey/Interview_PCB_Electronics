# Problem 03: Thermal Issue

## Problem Statement

A fully functional board has passed bring-up and is running a representative workload
inside its product enclosure. The design includes:
- A 12 V input, 3.3 V output synchronous buck converter rated at 3 A continuous
  (Texas Instruments LMR33630ADDA, 8-pin HSOIC PowerPAD package with an exposed
  thermal pad). It supplies the MCU, sensors, status LEDs and a display.
- A 1.8 V/300 mA LDO (Diodes Inc. AP7331-18, SOT25 package), fed from the 3.3 V rail,
  which supplies a 1.8 V sensor domain
- An STM32F4 microcontroller running at 168 MHz, full load (100% CPU utilisation)
- The PCB is a two-layer design with 1 oz copper on both sides
- Room temperature: 25°C, still air (no forced convection); the enclosure is sealed

After about 20 minutes at full load, the board resets unexpectedly. The reset is not
triggered by software: the MCU's RCC_CSR register shows the PORRSTF (power-on reset)
flag, meaning the supply voltage dropped below the power-on reset threshold. The board
restarts, but while it is still hot it resets again every few seconds to few tens of
seconds. It only runs normally again after being powered off and left to cool.

**Task:** Diagnose the thermal root cause and provide a quantified remedy.

---

## Solution Approach

### Step 1 — Characterise the Failure

The PORRSTF flag shows that the MCU's supply (VDD = 3.3 V) dropped below the internal POR
threshold (~1.8 V for STM32F4). This is a power supply failure, not a firmware error. The
board recovers, which rules out permanent device damage.

Three features point to a **thermally induced power supply failure**:
- It appears only after sustained operation.
- It repeats quickly once the board is hot.
- It clears after cooling.

Regulators with thermal shutdown behave exactly like this. They switch off at a junction
temperature threshold and restart once the die has cooled through a hysteresis band.

The supply chain is: 12 V input → buck (3.3 V) → MCU. The LDO hangs off the 3.3 V rail
and feeds only the 1.8 V sensor domain.

### Step 2 — Thermal Survey with Thermal Camera and Thermocouple

Open the enclosure lid only long enough to take a reading after 18 minutes at full load,
with a thermocouple taped inside the enclosure:

```
Readings after 18 minutes:
  Enclosure air (thermocouple):          44°C
  LMR33630 (buck), package top:          160°C  ← far above anything else
  AP7331 (LDO, SOT25), package top:       85°C
  STM32F4 (MCU):                          60°C
```

Two things stand out:
1. The enclosure air has risen 19°C above the room. This is the self-heating that makes
   the failure take 20 minutes to appear.
2. Both regulators are hot, but the buck is extreme.

**Thermal camera emissivity note:** Black epoxy package bodies have emissivity ε ≈ 0.95,
so the surface readings are reasonably accurate.

### Step 3 — Rule the LDO In or Out

LDO power dissipation:
```
P_LDO = (V_in - V_out) × I_out + V_in × I_q

V_in  = 3.3 V (from the buck)
V_out = 1.8 V
I_out = 150 mA (1.8 V sensor domain, measured with a current probe)
I_q   = 65 µA typical (AP7331 datasheet) — negligible

P_LDO = (3.3 - 1.8) × 0.150 + 3.3 × 65 µA = 225 mW
```

Junction temperature:
```
T_j = T_enclosure + P_LDO × Rθja
From AP7331 datasheet: Rθja = 190°C/W (SOT25, minimum pad, single-sided 2 oz board)

T_j = 44 + 0.225 × 190 = 44 + 42.8 = 86.8°C
```

This is consistent with the 85°C surface reading. It is below the AP7331's 125°C maximum
operating junction temperature, and well below its 140°C thermal shutdown threshold. The
LDO is hot but working as designed. It also does not supply the MCU, so it cannot cause a
VDD power-on reset. **The LDO is a red herring.** The buck is the suspect.

### Step 4 — Scope the Power Rails at the Moment of Reset

Set up the oscilloscope to capture the 3.3 V rail with a long capture window (10
minutes/div if possible, or use a data logger). Trigger on a rail drop below 2.5 V.

A digital oscilloscope with acquisition memory can be set to capture the event and
roll back to the pre-trigger state.

**Result:** At the moment of reset, the 3.3 V rail drops from 3.28 V to below 1.5 V
over approximately 50 µs. The buck's power-good (PG) output goes low at the same instant,
while the 12 V input stays solid. The rail comes back as the converter soft-starts,
and the cycle repeats.

A collapse with the input still present, followed by a clean restart, is the signature of
the regulator itself shutting down. The candidates are overcurrent (hiccup mode) and over-
temperature.

### Step 5 — Estimate the Buck Converter Junction Temperature (First Pass)

The design-time current budget for the 3.3 V rail:
```
MCU 120 mA + 2× MEMS sensors at 15 mA + 3× status LEDs at 10 mA = 180 mA
```

```
P_out  = 3.3 V × 0.180 A = 594 mW
η      ≈ 85%  (read the curve for your operating point from the datasheet;
                85% is assumed throughout this problem)
P_in   = 594 / 0.85 = 699 mW
P_loss = 699 - 594 = 105 mW
```

Thermal resistance: the LMR33630 datasheet quotes RθJA = 42.9°C/W for the DDA package,
but on a 4-layer JEDEC board. TI states that this figure is for comparing packages, not
for design. On this 2-layer, 1 oz board with limited copper around the part, take an
effective RθJA of **60°C/W** (illustrative; measure it or simulate it for a real design).

```
T_j = 25 + 0.105 × 60 = 31°C
```

**Discrepancy:** The calculation predicts a junction barely above ambient, but the
package top reads 160°C. Either the dissipation or the thermal resistance, or both,
is far larger than assumed.

### Step 6 — Re-Examine Current Draw

Measure the actual 3.3 V current with a current probe during normal operation:

```
Measured I_out: 1.38 A sustained, vs. estimated 180 mA.
```

That is 7.7× the estimate. Re-examine the schematic:

**Finding:** The 3.3 V rail also supplies the display's LED backlight boost converter. At
full brightness it draws 1.2 A from 3.3 V. It was not in the current budget.

```
Revised:
P_out  = 3.3 V × 1.38 A = 4.55 W
P_in   = 4.55 / 0.85 = 5.36 W
P_loss = 5.36 - 4.55 = 0.80 W

T_j (pad soldered properly, Rθja = 60°C/W):
  at 25°C:  25 + 0.80 × 60 = 73°C
  at 44°C:  44 + 0.80 × 60 = 92°C
```

That is still well short of the 165°C thermal shutdown threshold, and far below what the
camera shows. The current-budget error roughly doubles the junction temperature rise, but
it does not explain the resets on its own. The thermal path must be worse than assumed.

### Step 7 — Verify Exposed Pad Solder Joint (X-Ray or Physical Inspection)

X-ray the LMR33630. The image shows significant voiding in the exposed pad solder joint,
about 60% void coverage. IPC-7093 recommends a maximum of 25% voiding for thermally
critical exposed-pad devices.

Most of the heat leaves the HSOIC through its thermal pad: RθJC(bot) is 4.3°C/W, against
54°C/W through the top (LMR33630 datasheet). As a first-order (pessimistic) model, scale
the effective RθJA by the inverse of the soldered pad area:

```
With 25% voiding: Rθja ≈ 60°C/W / 0.75 = 80°C/W
With 60% voiding: Rθja ≈ 60°C/W / 0.40 = 150°C/W

T_j with the voided pad at P_loss = 0.80 W:
  at 25°C (power-on):          25 + 0.80 × 150 = 146°C
  at 44°C (after 18 minutes):  44 + 0.80 × 150 = 164°C
```

The LMR33630 shuts down when its junction reaches about 165°C and restarts at about 148°C
(datasheet). It crosses the shutdown threshold when the enclosure air reaches
165 − 0.80 × 150 = **44.5°C**, which is what the thermocouple read just before the failure.
The camera agrees as well: the datasheet's junction-to-top parameter ψJT = 4.3°C/W puts the
junction at 160 + 4.3 × 0.80 ≈ 163°C during the 18-minute reading. That is just below the trip point, and
in line with the model's 164°C.

After a shutdown the die cools only 17°C, to 148°C, before the converter restarts. With the
enclosure still hot, it reaches 165°C again within seconds. That is the repeated-reset
pattern in the problem statement.

### Root Cause Summary

Three issues compound:
1. **Incomplete current budget:** The display backlight boost converter was left out of the
   3.3 V current budget. The buck dissipates 0.80 W instead of the 0.105 W designed for.
2. **Exposed pad voiding (60%):** Poor solder paste deposition or stencil aperture design
   left the thermal pad largely unsoldered, raising the effective thermal resistance about
   2.5× (60 → 150°C/W).
3. **Enclosure self-heating:** In the sealed enclosure the air rises about 20°C over
   20 minutes, which removes the last of the thermal headroom.

No single issue trips the shutdown. With the pad soldered properly, the junction stays at
92°C even at the full 0.80 W and 44°C. With the pad voided but the original 0.105 W load,
it would be only 44 + 0.105 × 150 = 60°C.

---

## Analysis

### Calculating the Required Thermal Solution

The LMR33630 junction must not exceed 125°C, the die limit TI gives for design, at the
product's maximum ambient. The specified ambient is 40°C room temperature, but inside the
enclosure the air is about 19°C warmer, so design for 60°C.

```
Maximum allowable power dissipation, T_enclosure = 60°C:
  P_max = (T_j_max - T_enclosure) / Rθja

  Properly soldered pad (60°C/W):   (125 - 60) / 60  = 1.08 W   vs 0.80 W actual → OK
  Pad at the 25% void limit (80°C/W): (125 - 60) / 80  = 0.81 W   vs 0.80 W actual → no margin
  60% voided pad (150°C/W):          (125 - 60) / 150 = 0.43 W   vs 0.80 W actual → fails
```

So the rework must bring voiding well under the 25% limit, and the next spin must lower
the effective Rθja (thermal vias and more copper) to recover real margin.

### Stencil Design for Exposed Pad Components

Exposed pad voiding is caused by flux volatiles trapped during soldering. Prevention:

```
Stencil aperture design for exposed pad:
  1. Use a segmented aperture (array of small squares) rather than a single solid opening
  2. Total aperture area = 50-80% of exposed pad area
  3. Individual segments separated by 0.2-0.5 mm webs (allow flux to outgas)
  4. Typical segment: 0.8 mm × 0.8 mm square
  5. Maximum stencil thickness: 0.13 mm for small exposed pads

Example for a 2 mm × 2 mm exposed pad (illustrative):
  3×3 array of 0.55 mm × 0.55 mm squares
  Web width: 0.17 mm
  Coverage: 9 × (0.55²) / 4 = 68% of pad area
```

### Remedial Actions

**Immediate (hardware rework):**
1. Remove the LMR33630 using a hot air rework station with a thermocouple profile
2. Clean the exposed pad with solder wick
3. Apply fresh solder paste using a stencil with the segmented aperture design
4. Re-flow the LMR33630 using a controlled reflow profile
5. X-ray verify void coverage < 25%

**Schematic/design correction:**
1. Update the current budget to include the display backlight boost converter
2. Verify the LMR33630 current rating (3 A) is sufficient for the 1.38 A load — it is,
   with 2.2× margin. Current is not the limit here; dissipation is, so the fix is
   thermal, not a bigger regulator.

**PCB layout correction (for next spin):**
1. Add thermal vias under the exposed pad to conduct heat to the inner copper or
   bottom copper layer
2. Increase the copper pour area on both sides of the board under the buck converter
3. Use the segmented stencil aperture in the fabrication notes

---

## Key Takeaways

1. **Thermal failures have long time constants.** A component may be within its limits
   for the first few minutes but enter thermal shutdown after 15-30 minutes as the
   board equilibrium temperature rises. Always run thermal testing for at least 30
   minutes at maximum load before declaring a design thermally compliant.

2. **Exposed pad voiding is a critical manufacturing defect for power components.**
   A package with a poor exposed pad solder joint can have Rθja 2-3× higher than the
   datasheet value. Always X-ray power components in critical applications, and specify
   maximum void percentage (typically 25%) in the assembly drawing.

3. **Current budgets must be complete.** All loads on every power rail must be estimated
   before selecting power components. A single omitted load (like a display driver)
   can invalidate the entire thermal analysis.

4. **The thermal camera is a qualitative first step, not a quantitative final answer.**
   Camera readings are affected by emissivity of different surfaces. The camera shows
   where to look; calculations and datasheets confirm whether it is a real problem.

5. **Thermal shutdown produces a characteristic failure signature:** sudden reset or
   loss of function after a warm-up period, with recovery after a brief power-off
   that allows the device to cool below the thermal shutdown hysteresis threshold.

---

## Interview Notes

**Common question variants:**

- "How do you diagnose an intermittent reset that only happens after the board warms up?"
  → Use the RESET cause register (CSR/RSTCTL in most MCUs) to identify whether the
  reset was a power-on reset (supply drooped), a watchdog reset, or a software reset.
  Then monitor the supply rail with an oscilloscope in a long-capture mode to catch
  the voltage transient at the moment of reset.

- "What is the impact of solder voids on an exposed pad device?"
  → Voids are air pockets under the exposed pad that increase thermal resistance.
  The thermal path from the IC junction to the PCB copper is blocked wherever a void
  exists. Even 50% voiding can double the effective thermal resistance.

- "How do you calculate the junction temperature of a component?"
  → T_j = T_ambient + P_dissipated × Rθja (junction-to-ambient)
  For components on a heatsink: T_j = T_ambient + P × (Rθjc + Rθcs + Rθsa)
  where Rθjc = junction-to-case, Rθcs = case-to-heatsink, Rθsa = heatsink-to-ambient.

- "What is derating and why does it apply to temperature?"
  → Derating is reducing applied stress below the rated maximum. For thermal derating,
  a common rule is to target T_j < 125°C for silicon devices with T_j_max = 150°C,
  giving a 25°C margin. This margin accommodates measurement uncertainty, ambient
  temperature variation, and ageing effects.

- "How do you design a stencil for an exposed pad component?"
  → Use a segmented aperture (array of small squares or a cross pattern) covering
  50-80% of the pad area. The webs between segments allow flux volatiles to escape
  during reflow, reducing void formation. Stencil thickness should be ≤ 0.15 mm for
  fine-pitch devices to control paste volume.
