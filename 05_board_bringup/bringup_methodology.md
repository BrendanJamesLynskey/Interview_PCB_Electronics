# Bring-Up Methodology

## Prerequisites
- Basic bench measurement skills: multimeter, oscilloscope, bench power supply
- Familiarity with schematic and PCB layout navigation
- Understanding of power supply topologies (LDO, DCDC) and digital bus protocols
- Knowledge of component datasheets and absolute maximum ratings

---

## Concept Reference

### What Board Bring-Up Is

Board bring-up is the structured process of verifying a newly assembled PCB for the first
time — confirming that the hardware behaves as designed before firmware or software
development depends on it. It is distinct from production test (which is automated and
repeatable) and from debugging a previously working board (which begins from a known-good
baseline). Bring-up begins with zero assumptions about board health.

The overriding goal is to **expand the verified envelope incrementally**, never applying
conditions (voltage, current, clock, firmware) that the next unverified subsystem cannot
yet safely handle. Each phase confirms a set of preconditions that make the next phase safe.

### Why a Phased Approach is Essential

A board that fails in an unconstrained power-on has typically destroyed evidence of the
failure. Inrush current may have blown a fuse that was also the only path to a test point.
Thermal runaway may have desoldered components before temperatures were read. A latch-up
event caused by incorrect sequencing may present identically to a short circuit from an
assembly defect.

Phased bring-up:
1. Limits the energy available to any single fault (current-limited supply, isolation
   of power domains)
2. Creates decision gates where measurements either confirm the design or stop progress
3. Preserves the board in a state where the fault can be diagnosed and corrected

### The Standard Bring-Up Phases

```
Phase 0: Pre-Power Inspection
Phase 1: Power Rail Verification (current-limited, no IC power)
Phase 2: Passive Network and Peripheral Verification
Phase 3: Core Logic Power-On (processor, FPGA, memory)
Phase 4: Clock and Reset Verification
Phase 5: Communication Bus Verification
Phase 6: Peripheral and Firmware Bring-Up
Phase 7: Full Functional Test
```

---

### Phase 0 — Pre-Power Inspection

Never apply power to a board that has not passed visual inspection. Inspection checklist:

**Visual inspection (naked eye and magnification):**
- Solder bridges, especially on fine-pitch ICs (QFP, BGA, QFN)
- Missing components — compare physical board against assembly drawing or BOM
- Incorrect orientation — polarised components (electrolytic capacitors, diodes, LEDs,
  tantalum capacitors) have a marking for pin 1 or anode; verify against silkscreen
- Damaged components — cracked packages, discolouration, bent leads
- Cold joints — dull, grainy solder surface rather than shiny and concave

**Resistance checks (multimeter, power off):**
- Measure resistance from each power rail to ground
- A short (< 1 Ω) indicates either a bridge or a failed component
- Very low resistance (< 10 Ω) on a rail with many bulk capacitors is normal; true
  shorts read at or below the meter's lead resistance (~0.1-0.5 Ω)
- Typical acceptable values: VCCIO rails 50-500 Ω, core rails 10-100 Ω depending on
  capacitance and bypass count

**Polarity check:**
- Confirm input connector pinout with a continuity check against the schematic
- Confirm ground net connectivity from connector to local ground plane

---

### Phase 1 — Power Rail Verification

Apply power for the first time with a **current-limited bench supply**, not with the
board's own input from a wall adapter or battery.

**Current limit settings:**
- Set the compliance current to approximately 150% of the expected quiescent current
  for the subsystem being powered. If the expected quiescent draw is unknown, start at
  50 mA and increase cautiously.
- A supply that immediately current-limits indicates a fault. Remove power immediately.

**Measurement sequence for each rail:**
```
1. Set bench supply to 0 V, enable output
2. Slowly ramp voltage to rated value while watching current
3. Current should be low and relatively flat across the ramp
   (capacitive charging gives a brief spike then settles)
4. At rated voltage, measure: output voltage at the regulator
   output pin, and at the far end of the distribution network
5. Voltage drop > 100 mV across the distribution network indicates
   excessive resistance in the power path (check for poor joint on
   power via, missing plane connection, or incorrect component value)
```

**Power sequencing awareness:**
Many ICs have specified power rail sequencing requirements (see `power_on_sequence.md`).
In Phase 1, do not bring up rails out of sequence. If the board cannot be powered with
a subset of rails (all rails share the same input), use a bench supply per rail or
insert series resistors to delay slower rails.

**Thermal scan after first power-on:**
After the first full power-on (or immediately if current draw is higher than expected),
touch-check or use a thermal camera to identify hot components. Any component that is
uncomfortably hot to the touch within 30 seconds of power-on warrants investigation.
ICs that are warm are usually normal; ICs that are hot to the point of causing pain
(> 60°C) within the first minute of quiescent operation are suspect.

---

### Phase 2 — Passive Network and Peripheral Verification

Before powering the core logic, verify the supporting infrastructure:

**Crystal oscillators and PLLs:**
- Probe the crystal with a high-impedance probe (10 MΩ, < 5 pF) or use a frequency
  counter on the clock output pin
- Verify oscillator output swing is within spec; XTALs should not be probed directly
  without a 10x probe — the probe capacitance will detune or kill the oscillation

**Reset network:**
- Confirm the reset signal is asserted (active) at the correct logic level on power-up
- Measure the reset release timing — it must exceed the IC's power-on reset hold time
- Simulate a manual reset with a jumper or button and observe the voltage waveform

**Power supervisor / PMIC:**
- Verify power-good output signals are asserted at the correct voltage thresholds
- Confirm enable signals are in the correct state to allow sequenced rail start-up

---

### Phase 3 — Core Logic Power-On

With power rails and clocks verified, power on the processor, FPGA, or ASIC:

**JTAG / boundary scan:**
- The first functional test of a new board is almost always a JTAG connectivity test
- A JTAG chain that enumerates correctly confirms: power and ground to the device,
  JTAG signal integrity, and basic oscillator functionality
- If JTAG fails to enumerate, do not proceed to firmware load

**FPGA configuration:**
- Attempt to load a minimal bitfile (blink an LED or output a known clock) before
  loading the full application
- Verify configuration pins (PROGRAM_B, INIT_B, DONE) toggle through their expected
  states during configuration

**Processor boot:**
- Provide a minimal boot image that runs from internal ROM or a simple test vector
- The goal is to confirm the device executes instructions, not to run the full
  application firmware

---

### Phase 4 onwards — Incremental Functional Expansion

Each subsequent phase adds one sub-system at a time:

| Phase | What is added | Key verification |
|-------|--------------|-----------------|
| 4 | Clock generation, PLLs, clocking ICs | Frequency accuracy, jitter, lock time |
| 5 | Communication buses (I2C, SPI, UART, PCIe) | Protocol analyser, loopback test |
| 6 | Peripherals (ADC, DAC, sensors, storage) | Read/write functional tests |
| 7 | Full application firmware | System-level functional verification |

At each phase, document: what was tested, what passed, what failed, and what corrective
action was taken. This log is invaluable when the same board design returns for a revision.

---

## Bring-Up Phases

### Planning Before Boards Arrive

Bring-up efficiency is determined largely by decisions made during design:

- **Test points**: Dedicated pads on every power rail, every clock, every reset, and
  every critical bus signal. Test points must be large enough for a scope probe clip
  (typically 1.0 mm pad, clearly labelled on silkscreen).
- **LEDs**: At minimum, one LED per power rail and one LED that firmware can toggle.
  Cost is negligible; diagnostic value is high.
- **Isolation jumpers**: 0 Ω resistors or solder jumpers that allow a sub-section to
  be disconnected from the main board. Essential for phased bring-up.
- **JTAG/SWD header**: Every board with a programmable device should have an accessible
  JTAG or SWD header that does not require soldering.
- **Boot mode select**: Exposed boot mode pins (BOOT0/BOOT1, CFG pins) allow changing
  the boot source without board rework.

### Bring-Up Procedure Documentation

A written bring-up procedure must exist before the board arrives. The procedure includes:

1. Expected resistance values for each power rail (from simulation or prior design)
2. Expected quiescent current for each phase
3. Expected voltage for each rail with tolerance
4. Clock frequencies and expected waveform characteristics
5. Pass/fail criteria for each test step
6. Instructions for proceeding on pass and stopping on fail

---

## Planning Strategy

### Risk Prioritisation

Not all problems are equally likely on a first spin. Statistically, first-article failures
cluster around:

1. **Assembly errors** (wrong component, wrong orientation, solder bridge) — Phase 0
2. **Power supply design errors** (wrong feedback resistor, wrong enable polarity) — Phase 1
3. **Reset / power sequencing issues** — Phase 2
4. **Clock / PLL issues** (load capacitor value, XO selection, PLL loop filter) — Phase 4
5. **PCB routing errors** (signal swap, incorrect net connection) — Phase 5

Phases are ordered to hit the highest-probability, highest-impact failures first.

### Spare Boards and Components

Always order at least three PCBs for a first article. The first is for bring-up (may be
damaged during debug), the second is the clean reference board once bring-up is complete,
and the third is for rework and ECN verification. Order spare ICs for any component likely
to be damaged by a bring-up fault (especially MOSFETs, LDOs, and ESD-sensitive devices).

---

## Best Practices

- Never power a board for the first time from a fixed-voltage wall adapter. Always use a
  current-limited bench supply with visible current readout.
- Write down measurements as you take them. Memory is unreliable; measurements are not.
- Use a calibrated oscilloscope, not a multimeter, for any signal faster than 1 kHz.
  Multimeters alias fast signals and report misleading values.
- When a measurement fails, do not immediately assume a design error. Verify the
  measurement setup first: correct probe ground, correct probe compensation, correct
  measurement point.
- If a board is smoking or smells burnt, remove power immediately and do not apply
  power again until the damage is assessed and the root cause is identified.
- Keep a bring-up log. Record every measurement, every anomaly, and every corrective
  action. The log is the foundation of the ECN process if design changes are needed.

---

## Tier 1 — Fundamentals

### Question F1
**What is the purpose of using a current-limited bench supply during initial board power-on, and how should the current limit be set?**

**Answer:**

A current-limited bench supply serves two functions:

1. **Fault protection:** If a fault draws excessive current (dead short, latch-up,
   wrong polarity component), the supply current-limits and clamps the voltage rather
   than delivering unlimited fault energy. This prevents thermal damage, fire, and the
   destruction of diagnostic evidence.

2. **Fault detection:** The front-panel current meter shows the actual draw. If the
   current immediately hits the limit and the voltage collapses, a fault is confirmed
   before any damage propagates.

Current limit setting: Set the limit to approximately 150% of the expected quiescent
current for the sub-system being powered. For example, if a 3.3 V rail is expected to
draw 200 mA at quiescent, set the limit to 300 mA. Do not set the limit so high that a
fault could still cause damage before the supply responds.

For unknown designs, begin at 50 mA and increase slowly while watching the current
meter and touching the board for unexpected heat.

---

### Question F2
**List five things to check during a pre-power visual inspection.**

**Answer:**

1. **Solder bridges** on fine-pitch devices (QFP, QFN, BGA). Use a magnifying glass or
   USB microscope. Focus on the tightest-pitch components and any area that looks
   irregular in photos or X-ray.

2. **Component polarity** for electrolytic capacitors, tantalums, diodes, and LEDs.
   Electrolytic capacitors have a stripe on the negative lead. Tantalums have a stripe
   on the positive lead. The polarity convention difference between these two types is a
   common error.

3. **Missing components** by comparing the physical board to the assembly drawing or
   BOM. Pay attention to small passive values (100 nF, 10 kΩ) that are easily missed
   by automated placement and are not obviously absent to the eye.

4. **Resistance to ground on each power rail** using a multimeter in resistance mode
   with the board unpowered. A near-zero reading indicates a short that must be resolved
   before power is applied.

5. **Correct connector pinout** — verify the input power connector pin-to-net assignment
   by continuity test against the schematic. An inverted power connector destroys the
   board on first power-on.

---

### Question F3
**Why is JTAG connectivity verification typically the first active test on a new board?**

**Answer:**

JTAG (IEEE 1149.1) boundary scan confirms several conditions simultaneously:

- The device has valid power and ground on all supply pins (a device without power
  will not enumerate on JTAG)
- The JTAG clock (TCK), data in (TDI), data out (TDO), and mode select (TMS) signals
  are correctly routed and have adequate signal integrity
- The device's basic logic is functional enough to implement the TAP state machine
- The oscillator or reference clock connected to the device is active (some devices
  require a reference clock to operate their JTAG TAP)

A successful JTAG enumeration with the correct device ID is therefore a single test
that validates power delivery, basic signal integrity, clock generation, and device
health simultaneously — more diagnostic value per test step than any other single
measurement on a new board.

If JTAG fails, there is no point attempting a firmware download. JTAG failure must be
resolved first.

---

## Tier 2 — Intermediate

### Question I1
**You apply power to a new board for the first time. The bench supply immediately current-limits and the voltage collapses. Describe your diagnostic approach.**

**Answer:**

A current-limit on first power-on indicates a low-impedance fault. The diagnostic sequence:

**Step 1 — Confirm the fault is real, not a measurement artefact.**
Disconnect the board and measure the bench supply output directly. Verify the supply is
not in protection mode from a prior event. Reconnect and observe current and voltage
simultaneously.

**Step 2 — Identify which rail is faulting.**
If the board has a single input rail, probe the voltage at the regulator input, regulator
output, and key distribution points with the supply current-limiting. The voltage will
collapse near the fault point. If multiple rails share the same input, use isolation
jumpers to disconnect rails one at a time until the current drops to normal.

**Step 3 — Identify the short's location.**
With power removed, use a milliohm measurement or a power injection technique (inject
a small, safe current from a bench supply into the shorted rail, then use a thermal
camera or an IR thermometer to find the component heating disproportionately — that
is the fault location). Alternatively, use a four-wire resistance measurement to find
the lowest-resistance path from the rail to ground.

**Step 4 — Determine the fault type.**
- Bridge between rail and ground (probe both sides of suspected bridge with magnification)
- Wrong component value or type (verify ICs, regulators, resistors against BOM)
- Wrong component orientation (capacitor, diode polarity reversal)
- Failed component (an IC with an internal short — verify by removing the IC and
  re-measuring the rail resistance)

**Step 5 — Correct and re-verify.**
After correction, re-measure the rail-to-ground resistance before re-applying power.

---

### Question I2
**A processor board has three supply rails: 3.3 V (I/O), 1.8 V (DDR), and 1.0 V (core). The datasheet states that 1.0 V core must rise before 1.8 V, and 1.8 V must rise before 3.3 V. How do you verify the power sequence during bring-up?**

**Answer:**

**Measurement setup:**
Connect a four-channel oscilloscope with one channel per rail. Use passive 10x probes
with short ground springs (not the flying lead ground — high inductance ground leads
corrupt the low-frequency rise-time measurement). Trigger on the first rail to rise
(1.0 V core).

**Procedure:**
```
1. Set all four channels to DC coupling, 1 V/div or 2 V/div
2. Set timebase to capture the full power-on transient: typically 10-50 ms/div
3. Power-on the board (or press the enable signal while the supply is already running)
4. Capture the waveform
5. Use the oscilloscope's horizontal cursors to measure:
   - T1: time at which 1.0 V core reaches 10% of its nominal value
   - T2: time at which 1.8 V DDR reaches 10% of its nominal value
   - T3: time at which 3.3 V I/O reaches 10% of its nominal value
   - Verify T1 < T2 < T3
6. Measure the final settled values at steady state and compare to nominal
7. Measure overshoot: any rail that overshoots by > 5% of its nominal value
   warrants investigation
```

**Pass criteria:**
- Sequence order: 1.0 V rising before 1.8 V, 1.8 V rising before 3.3 V (by at least
  the margin specified in the datasheet, typically 0 to a few milliseconds)
- Final voltages within ±5% of nominal (or within datasheet absolute maximum limits)
- No rail oscillating or repeatedly trying to start and collapsing

**Common issues found:**
- Sequence inversion due to incorrect enable signal routing
- One rail that starts, triggers an overcurrent condition on the supply, and collapses
  before the others start — indicates a sequencing fault or excessive inrush

---

## Tier 3 — Advanced

### Question A1
**You are bringing up a new board that uses a PMIC to sequence six power rails. After power-on, five rails are correct but the sixth (1.8 V DDR termination) does not start. The PMIC STATUS register (readable via I2C) reports a FAULT condition on that rail. Describe your diagnostic process.**

**Answer:**

A PMIC fault flag on a single rail narrows the problem to either the rail load, the
PMIC configuration, or the PMIC hardware for that channel.

**Step 1 — Read the full PMIC register map.**
Read all fault registers, not just STATUS. Most PMICs distinguish between:
- PGOOD fault (rail did not reach regulation within timeout)
- OCP fault (overcurrent protection triggered)
- OVP fault (overvoltage protection triggered)
- UVP fault (undervoltage protection — rail sagged below threshold after starting)
- TSD (thermal shutdown of the PMIC itself)

The specific fault type gives immediate direction.

**Step 2 — If OCP fault (most common for a new design):**
Probe the rail with a current probe and oscilloscope. Determine whether the overcurrent
event is:
- Inrush during soft-start (PMIC soft-start ramp too fast for the load capacitance —
  reduce the ramp rate register or increase the current limit register)
- Steady-state overcurrent (load draws more than the PMIC channel rating — check for
  a short on the 1.8 V rail, or a load that is higher than expected)
- Spurious fault due to an PMIC setting error (compare I2C register content to the
  intended configuration in the design files — a single wrong bit can halve the
  current limit)

**Step 3 — If PGOOD fault:**
The rail attempted to start but did not reach its regulation voltage within the
PMIC's soft-start timeout window. Probe the rail during start-up attempt. Look for:
- Rail rising partially then collapsing (load overcurrent or output capacitor too large)
- Rail not rising at all (PMIC enable signal not asserted, missing or wrong feedback
  resistor divider on an adjustable channel, inductor or output capacitor wrong value)

**Step 4 — Verify the PMIC configuration against the design.**
Dump the entire PMIC I2C register map and compare against the intended register
configuration table from the design files. One wrong bit in the DDR termination
channel configuration can explain the fault.

**Step 5 — Isolate the load.**
With the 1.8 V DDR termination rail disabled (force the PMIC channel off), measure the
resistance from the rail net to ground with the board powered (other rails active).
A resistance substantially below the expected value indicates a short in the DDR
termination network (often a wrong pull-down resistor value or a bridge near the DDR
termination resistors).

---

## Related Topics

- [Power-On Sequence](power_on_sequence.md)
- [Debugging Techniques](debugging_techniques.md)
- [Rework and ECN](rework_and_ecn.md)
- [Design for Test](../01_schematic_design/design_for_test.md)
- [Worked Problem: No Power Debug](worked_problems/problem_01_no_power_debug.md)
