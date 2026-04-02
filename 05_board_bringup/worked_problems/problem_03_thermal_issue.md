# Problem 03: Thermal Issue

## Problem Statement

A fully functional board has passed bring-up and is running a representative workload.
The design includes:
- A 5 V input, 1.8 V output synchronous buck converter rated at 3 A continuous
  (Monolithic Power Systems MP2315S, 8-pin SOIC-8-EP package)
- A 3.3 V/500 mA LDO (Diodes Inc. AP7331) in a SOT-23-5 package
- An STM32F4 microcontroller running at 168 MHz, full load (100% CPU utilisation)
- The PCB is a two-layer design with 1 oz copper on both sides
- Ambient temperature: 25°C in still air (no forced convection)

After 20 minutes at full load, the board resets unexpectedly. The reset is not triggered
by software; the MCU's RCC_CSR register shows a PORST (power-on reset) flag, indicating
the supply voltage dropped below the power-on reset threshold. The board recovers after
the reset and resumes operation, only to reset again after approximately 15 minutes.

**Task:** Diagnose the thermal root cause and provide a quantified remedy.

---

## Solution Approach

### Step 1 — Characterise the Failure

The PORST flag indicates the MCU's supply voltage (VDD = 3.3 V) dropped below the
internal POR threshold (~1.8 V for STM32F4). This is a power supply failure, not a
firmware error. The board recovers, which rules out permanent device damage.

The fact that the failure occurs only after sustained operation at elevated temperature
strongly suggests a **thermally-induced power supply failure**.

The two candidates supplying the MCU are:
- The 3.3 V LDO (AP7331), which supplies the MCU
- The 5 V buck converter, which supplies the LDO input

Either could be failing thermally. Work from the supply input toward the load.

### Step 2 — Thermal Survey with Oscilloscope and Thermal Camera

**Initial thermal measurement (30-second exposure, full load):**

Set up a thermal camera (FLIR One or equivalent) and observe the board at full load.

```
Component temperatures observed:
  MP2315S (buck converter):   71°C
  AP7331 (LDO, SOT-23-5):    96°C  ← significantly elevated
  STM32F4 (MCU):              58°C
  Ambient:                    25°C
```

The AP7331 LDO at 96°C is immediately suspect. SOT-23-5 packages have a thermal
resistance of approximately 200-350°C/W (junction-to-ambient, on a 2-layer PCB with
minimal copper pour). The junction temperature is likely higher than the package surface
temperature seen by the camera.

**Thermal camera emissivity note:** The SOT-23-5 package body (black epoxy) has
emissivity ε ≈ 0.95, so the 96°C surface reading is reasonably accurate.

### Step 3 — Calculate Expected LDO Junction Temperature

LDO power dissipation:
```
P_LDO = (V_in - V_out) × I_out + V_in × I_quiescent

V_in  = 5 V (from buck output)
V_out = 3.3 V
I_out = ??? (measure with a current probe or calculate from MCU power consumption)
I_q   = 55 µA (from AP7331 datasheet) — negligible

STM32F4 at 168 MHz, full CPU utilisation:
  Typical current from datasheet: ~80 mA at 168 MHz, 3.3 V
  P_MCU = 3.3 V × 0.080 = 264 mW

Therefore:
  I_out ≈ 80 mA + peripherals ≈ 120 mA (estimate including I/O activity)

P_LDO = (5.0 - 3.3) × 0.120 = 1.7 V × 0.120 A = 204 mW
```

Junction temperature estimate:
```
T_j = T_ambient + P_LDO × Rθja
    = 25 + 0.204 × Rθja

From AP7331 datasheet: Rθja = 252°C/W for SOT-23-5 on a standard single-layer PCB
                              (two-layer PCB with no copper pour: ~220°C/W)

T_j = 25 + 0.204 × 220 = 25 + 44.9 = 69.9°C   (junction temperature estimate)

With actual measured surface temperature of 96°C, the junction is approximately:
  T_j ≈ T_surface + small internal resistance gradient ≈ 100-110°C
```

The AP7331 has an absolute maximum junction temperature of 125°C. At the calculated
and measured temperatures, the device is approaching its limit, but should not be
triggering its thermal shutdown (Tsd = 155°C typical) yet.

**Reassessment:** The 15-20 minute time delay before failure suggests the thermal
failure is not at the LDO itself. The MCU VDD supply might be drooping due to an
associated thermal effect elsewhere.

### Step 4 — Scope the Power Rails at the Moment of Reset

Set up the oscilloscope to capture the 3.3 V rail with a long capture window (10
minutes/div if possible, or use a data logger). Trigger on a rail drop below 2.5 V.

A digital oscilloscope with acquisition memory can be set to capture the event and
roll back to the pre-trigger state.

**Result:** At the moment of reset, the 3.3 V rail drops from 3.28 V to below 1.5 V
over approximately 50 µs, then recovers to 3.28 V within 2 ms.

The drop characteristic — fast collapse, fast recovery — is consistent with the LDO
(or its input supply) briefly entering an over-temperature shutdown, toggling its
thermal protection, and recovering.

### Step 5 — Investigate the Buck Converter Thermal State

Return to the thermal camera. After 20 minutes of operation:

```
MP2315S surface temperature: 89°C  (was 71°C at 30 seconds)
```

The buck converter has been heating for 20 minutes. At 89°C on the package surface,
the junction temperature of the internal MOSFETs is higher.

Calculate buck converter power dissipation:
```
Vin  = 5 V, Vout = 1.8 V (note: this buck is also supplying a 1.8 V rail elsewhere
  on the board — also loaded)
  Wait: recheck the schematic.

Schematic review reveals: the MP2315S is set to Vout = 3.3 V (not 1.8 V), and the
AP7331 LDO is a post-regulator from the 3.3 V buck output to a separate 3.3 V domain.
The MCU is powered from the buck output directly (3.3 V), not from the LDO.
```

**This changes the analysis significantly.** The LDO (AP7331) was a red herring in
the thermal survey — its high temperature is expected given its power dissipation and
package, but it is not supplying the MCU.

The MCU VDD is powered from the buck converter output. The buck is the suspect.

### Step 6 — Verify Buck Converter Thermal Performance

MP2315S datasheet: Rθja = 45°C/W (SOIC-8-EP, assuming exposed pad soldered to copper)

```
P_buck = (1 - efficiency) × P_out

Vout = 3.3 V, Iout = 120 mA (MCU) + other loads...

Check schematic more carefully: total 3.3 V load = 120 mA (MCU) + 2× MEMS sensors
at 15 mA each + 3× status LEDs at 10 mA each = 120 + 30 + 30 = 180 mA

P_out = 3.3 × 0.180 = 594 mW

Typical efficiency of MP2315S at 5V → 3.3V, 180 mA: ~88%

P_in  = P_out / η = 594 / 0.88 = 675 mW
P_loss = P_in - P_out = 675 - 594 = 81 mW

T_j_buck = T_amb + P_loss × Rθja = 25 + 0.081 × 45 = 28.6°C
```

This looks very cool — much cooler than the 89°C surface measurement.

**Discrepancy:** The calculated junction temperature is 28.6°C above ambient (53.6°C
junction) but the measured surface is 89°C. This discrepancy is large enough to
indicate that either:
1. The exposed pad (EP) is not properly soldered to the thermal relief copper, so the
   actual Rθja is much higher than 45°C/W
2. There is another heat source in the vicinity being picked up by the camera

### Step 7 — Verify Exposed Pad Solder Joint (X-Ray or Physical Inspection)

X-ray the MP2315S. The X-ray image shows significant voiding in the exposed pad solder
joint — approximately 60% void coverage. IPC-7093 recommends a maximum of 25% voiding
for thermal-critical exposed pad devices.

```
Impact of voiding on thermal resistance:
  With 25% voiding: Rθja ≈ 45°C/W × 1.33 = 60°C/W
  With 60% voiding: Rθja ≈ 45°C/W × 2.5  = 112°C/W

T_j with voided pad = 25 + 0.081 × 112 = 34°C   ... still only 59°C

Still too low to explain 89°C surface temperature. Continue investigating.
```

### Step 8 — Re-Examine Current Draw

Measure the actual 3.3 V current with a current probe during the failure window:

```
Measured I_out: peaks at 920 mA during the failure event, vs. estimated 180 mA.
```

A 5× underestimate of the current draw. Re-examine the schematic:

**Finding:** The 3.3 V rail also supplies a 128 × 64 OLED display via a boost converter.
The boost converter is drawing 700 mA from 3.3 V when active (OLED at maximum brightness).
This was not captured in the initial current estimate.

```
Revised:
P_out = 3.3 V × 0.920 A = 3.04 W
P_in  = 3.04 / 0.88 = 3.45 W
P_loss = 3.45 - 3.04 = 410 mW

T_j = 25 + 0.410 × 112 (with voided pad) = 25 + 45.9 = 70.9°C  (junction)

Still lower than 89°C surface. But the junction temperature of the IC MOSFET
(not the package surface) may be higher. The thermal camera sees the package
surface, not the silicon. With poor exposed pad bonding:

  T_j_MOSFET = T_j_calc + (internal junction-to-case resistance × P_loss)
```

After long thermal soak (20 minutes), the board temperature everywhere has risen
due to the limited heat-sinking. The ambient within the board enclosure may have
reached 40-50°C even though the test room is at 25°C. Recalculate:

```
With ambient_effective = 45°C:
T_j = 45 + 0.410 × 112 = 45 + 45.9 = 90.9°C

This aligns with the 89°C surface measurement.
```

The MP2315S is approaching its thermal shutdown threshold (typically 150°C). Given
the 60% void coverage, any load increase or ambient temperature rise would push it
into thermal shutdown.

### Root Cause Summary

Three compounding issues:
1. **Underloaded current estimate:** The OLED boost converter was not included in the
   3.3 V current budget during design, causing the buck converter to be under-specified.
2. **Exposed pad voiding (60%):** Poor solder paste deposition or aperture design in
   the exposed pad stencil caused excessive voids, tripling the thermal resistance.
3. **Thermal self-heating:** In a sealed or poorly ventilated enclosure, ambient
   temperature rises over time, reducing the available thermal headroom.

---

## Analysis

### Calculating the Required Thermal Solution

The MP2315S must not exceed T_j = 125°C at maximum ambient (specified as 40°C for
this product).

```
Maximum allowable power dissipation at T_ambient = 40°C:
  P_max = (T_j_max - T_ambient) / Rθja
        = (125 - 40) / 45 = 1.89 W   (with properly soldered exposed pad)

Actual dissipation: 410 mW — well within the 1.89 W limit.

But with voided pad (Rθja = 112°C/W):
  P_max = (125 - 40) / 112 = 0.76 W
  Actual 410 mW is within this, but with enclosure self-heating, effective
  ambient rises further. At T_ambient = 60°C:
  P_max = (125 - 60) / 112 = 0.58 W  — below the 410 mW dissipation.
```

This confirms the voided pad is the primary hardware fault. The secondary issue is
the current budget oversight.

### Stencil Design for Exposed Pad Components

Exposed pad voiding is caused by flux volatiles trapped during soldering. Prevention:

```
Stencil aperture design for exposed pad:
  1. Use a segmented aperture (array of small squares) rather than a single solid opening
  2. Total aperture area = 50-80% of exposed pad area
  3. Individual segments separated by 0.2-0.5 mm webs (allow flux to outgas)
  4. Typical segment: 0.8 mm × 0.8 mm square
  5. Maximum stencil thickness: 0.13 mm for small exposed pads

Example for MP2315S (2 mm × 2 mm exposed pad):
  3×3 array of 0.55 mm × 0.55 mm squares
  Web width: 0.17 mm
  Coverage: 9 × (0.55²) / 4 = 68% of pad area
```

### Remedial Actions

**Immediate (hardware rework):**
1. Remove the MP2315S using a hot air rework station with a thermocouple profile
2. Clean the exposed pad with solder wick
3. Apply fresh solder paste using a stencil with the segmented aperture design
4. Re-flow the MP2315S using a controlled reflow profile
5. X-ray verify void coverage < 25%

**Schematic/design correction:**
1. Update the current budget to include the OLED boost converter
2. Verify the MP2315S current rating (3 A) is sufficient for the 920 mA peak load
   — it is, but with only 3× margin. For the next spin, consider a 5 A rated device
   for derating.

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
