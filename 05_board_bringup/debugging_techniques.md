# Debugging Techniques

## Prerequisites
- Oscilloscope operation: triggering, probing, channel math, FFT mode
- Basic bench equipment: multimeter, function generator, bench power supply
- Understanding of digital logic levels and communication protocols
- Ability to navigate a schematic and PCB layout simultaneously

---

## Concept Reference

### The Principle of Least Disruption

Every measurement perturbs the circuit under test. The goal of board debugging is to
measure accurately while perturbing as little as possible. Choosing the right tool and
the right probing technique is not optional — a probe with wrong impedance or a ground
lead that is too long will create artefacts that look exactly like real circuit failures.

The three sources of measurement error in PCB debugging:
1. **Probe loading:** The probe's input impedance (resistance and capacitance) forms a
   voltage divider with the circuit's source impedance, attenuating or distorting the
   signal.
2. **Ground inductance:** A long probe ground lead adds inductance in series with the
   measurement ground return, causing resonance peaks and ringing on fast edges.
3. **Common-mode pickup:** An unshielded probe near a high-dV/dt node picks up
   capacitively coupled noise that appears on the measurement.

---

### Oscilloscope Probing Techniques

**Probe types and their appropriate applications:**

| Probe type | Attenuation | Bandwidth | Input impedance | Use case |
|-----------|-------------|-----------|-----------------|---------|
| 10x passive | 10:1 | 100-500 MHz | 10 MΩ ∥ 10-15 pF | General-purpose board debugging |
| 1x passive | 1:1 | 10-20 MHz | 1 MΩ ∥ 50-100 pF | Low-frequency, low-impedance nodes only |
| Active (FET) | 1:1 | 1-6 GHz | 1 MΩ ∥ 0.5-1 pF | High-frequency, high-impedance nodes |
| Differential | Configurable | 100 MHz-2 GHz | High, floating | Floating or high-voltage signals |
| Current (clamp) | N/A | 100 kHz-100 MHz | Non-contact | Non-invasive current measurement |

**The single most important probing technique: use a short ground spring.**

A standard oscilloscope probe comes with a long (~15 cm) ground flying lead with a
clip. For signals above approximately 5 MHz, never use this ground lead. Instead, use
the short ground spring (the small coil accessory that attaches to the probe tip,
shorting the probe barrel to the probe tip with a spring contact less than 5 mm long).

Why: The inductance of a 15 cm ground lead is approximately 150 nH. At 10 MHz, this
has an impedance of ~9.4 Ω. When combined with the probe's tip capacitance (~15 pF),
it forms an LC resonance near 100 MHz. This resonance appears as ringing on any fast
edge, and can be mistaken for actual circuit ringing.

```
Long ground lead:   L ≈ 150 nH, resonance with 15 pF ≈ 100 MHz
Short ground spring: L ≈ 5 nH, resonance with 15 pF ≈ 600 MHz
Ideally: PCB-mounted socket (e.g., Pomona clip) with 2 mm ground connection
```

**Probe compensation:**
Every 10x passive probe has an adjustable compensation capacitor at its BNC connector.
Compensation must be verified every time the probe is used on a different oscilloscope
or after a temperature change. An uncompensated probe over- or under-compensates
high-frequency content, causing edges to appear sharper or duller than they are.

To compensate: connect the probe to the oscilloscope's calibration output (typically a
1 kHz square wave at 3.3 V). Adjust the compensation trimmer until the square wave
corners are flat with no overshoot or droop.

**Probing QFN/BGA/small-pitch devices:**
Direct probing of fine-pitch device pins with a standard probe tip often requires
holding the probe at an angle or using a solder-attached test hook. For production-grade
measurements, use a probe designed for fine-pitch work with a sharp tungsten tip, or
use dedicated test point pads near the pin of interest.

---

### Oscilloscope Setup for Common Measurements

**Power rail noise measurement:**
```
- AC coupling (removes DC component, shows only AC ripple/noise)
- 20 MHz bandwidth limit OFF (to capture switching noise up to full probe bandwidth)
- Vertical: 50 mV/div
- Horizontal: 1-10 µs/div (to capture switching frequency ripple)
- Trigger: channel trigger on the noise peak, or use a second channel triggered on
  the converter's switching node
- Expected: < 1% of rail voltage for a well-designed supply (< 33 mV on a 3.3 V rail)
```

**Digital signal integrity measurement:**
```
- DC coupling
- Trigger: edge trigger on the signal under test
- For rise/fall time: use maximum bandwidth and full vertical range
  (rise time measurement requires bandwidth > 0.35 / rise_time)
- For setup/hold timing: use cursors or automatic measurements
- For eye diagrams: use infinite persistence with a period trigger
```

**Switching regulator waveform measurement:**
```
Switching node (VSW):
  - 10x probe with short ground spring
  - Vertical: 2-5 V/div (captures full swing from PGND to VIN)
  - Horizontal: 200 ns/div for a 1 MHz switcher
  - Beware: the switching node swings at the full input voltage (10-20 V for some
    converters). Ensure probe is rated for the voltage.

Output ripple:
  - See power rail noise measurement above
  - Place probe tip directly at the output capacitor's positive terminal with the
    ground spring clipped to the capacitor's negative terminal — not on a remote pad
```

**I2C/SPI bus decode:**
Most modern oscilloscopes include serial decode firmware. Setup:
```
Protocol: I2C
  - CH1: SCL, threshold at 1.5 V (for 3.3 V logic)
  - CH2: SDA, threshold at 1.5 V
  - Trigger: I2C Start condition or specific address

Protocol: SPI
  - CH1: SCLK
  - CH2: MOSI
  - CH3: MISO
  - CH4: CS# (active low)
  - Bit order: MSB first (verify against device datasheet)
```

---

### Thermal Imaging

A thermal camera is one of the most powerful tools for rapid fault localisation. It
provides a non-invasive view of the entire board simultaneously, allowing identification
of hot components within seconds of power-on.

**Equipment options:**
- Benchtop thermal camera (FLIR E series, Optris): 640x480 pixels, ±2°C accuracy,
  ~0.1°C NETD. Professional tool for precision work.
- USB/smartphone thermal module (FLIR One, Topdon TC001): 160x120 pixels, ±3°C.
  Sufficient for most bring-up fault localisation.
- Thermal paper (Thermochromic paper): Qualitative only, low cost, no equipment needed
  — pressed against the board, hotspots stain the paper.

**Calibration and emissivity:**

PCB surfaces (FR4, solder mask, copper) have different thermal emissivity values:
```
Green solder mask (epoxy):  ε ≈ 0.90 (good thermal emitter, accurate reading)
Bare copper (shiny):         ε ≈ 0.05 (poor emitter, reads much colder than actual)
Black anodised aluminium:   ε ≈ 0.97
Silver solder (shiny):      ε ≈ 0.10
```

A bare copper pad or exposed via will read much colder than its actual temperature
in a thermal image. Set the camera's emissivity to 0.9 for solder-mask areas, and be
sceptical of temperature readings on exposed copper.

**Thermal imaging procedure during bring-up:**
```
1. Power up the board in a controlled environment (still air, room temperature ambient)
2. Allow 30-60 seconds for thermal steady state (most components reach 90% of their
   steady-state temperature within 30 seconds)
3. Capture a thermal image
4. Look for:
   a. Components with temperature > ambient + 30°C (candidates for investigation)
   b. Components significantly hotter than identical nearby components
      (e.g., one decoupling capacitor at 60°C while all others are at 30°C)
   c. Any component above its datasheet maximum junction temperature (Tj,max)
5. For a suspected fault: apply power, watch the thermal image in real time.
   A shorted component heats extremely rapidly (seconds) vs. a normally warm component
   (minutes to reach steady state).
```

**Thermal imaging for intermittent faults:**
Apply a cooling spray (freeze spray / component cooler) to a suspected hot component.
If the fault disappears when the component is cooled and returns when it warms up, this
confirms a thermal-coefficient-related failure (solder joint that opens when heated,
component that fails above a threshold temperature).

---

### Logic Analysers and Protocol Analysers

A logic analyser captures many digital signals simultaneously at high sample rates,
with deep memory and protocol decode firmware. Essential for debugging multi-signal
buses where an oscilloscope runs out of channels.

**When to use a logic analyser vs. an oscilloscope:**

| Measurement | Tool | Reason |
|-------------|------|--------|
| Rise/fall time of a digital signal | Oscilloscope | Analogue amplitude information needed |
| Protocol timing (I2C, SPI, UART) with < 8 signals | Oscilloscope with decode | Fewer channels, lower setup time |
| Complex protocol (PCIe, USB, DDR) | Protocol analyser | Protocol-specific decode, compliance tools |
| > 8 simultaneous digital signals | Logic analyser | Channel count |
| Rare/intermittent event capture | Logic analyser | Deep memory, complex triggering |
| Power rail noise | Oscilloscope | Analogue resolution needed |

**Common logic analyser mistake:** Running the sample rate too low. A logic analyser
sampling at 4x the signal rate will miss glitches. For reliable capture of a 10 MHz
SPI clock, use at least 40 MHz sample rate; for glitch capture use 400 MHz or higher.

---

### In-Circuit Emulators and JTAG Debuggers

A JTAG debugger (J-Link, CMSIS-DAP, OpenOCD) connects to the device's debug port and
provides:
- CPU halt, step, and run control
- Memory read/write (RAM, flash, peripheral registers)
- Breakpoint and watchpoint hardware
- JTAG boundary scan (see `bringup_methodology.md`)

During bring-up, use the debugger to:
1. Read peripheral register maps (confirm clock generator is running, DMA is idle)
2. Test memory connectivity (write a walking-1 pattern to RAM, read back)
3. Single-step through startup code to find the instruction that triggers a fault
4. Read internal temperature sensor registers if available

---

## Measurement Tools

### Bench Power Supply Configuration

During debugging, configure the bench supply for maximum diagnostic value:
```
- Current limit: set to known-good operational current × 1.5
- Output voltage: set to nominal then measure at board
  (account for voltage drop across the cable — use a 4-wire (Kelvin) connection
   if the supply supports it)
- Enable/disable sequence: use front-panel key rather than connection/disconnection
  to avoid mechanical transients on the supply line
```

### Multimeter Usage

The multimeter is for DC and low-frequency AC measurements only. Do not use a
multimeter on high-frequency switching nodes.

Appropriate uses:
- Continuity check during Phase 0 inspection
- DC voltage measurement on supply rails (< 100 kHz)
- Resistance measurement (board unpowered)
- Diode drop measurement for fault identification

Inappropriate uses:
- Measuring switching node voltage (the meter will display a meaningless average)
- Measuring clock frequencies above a few hundred kHz
- Measuring transient events

### Bench LCR Meter

An LCR meter measures inductance, capacitance, and resistance of individual components.
Useful during bring-up for:
- Verifying a component is the correct value (measure before soldering if suspect)
- Confirming a capacitor is not internally shorted (read infinite capacitance or low
  resistance)
- Measuring the effective capacitance of a power rail (total by-pass capacitance)

---

## Debug Methods

### Signal Tracing

Signal tracing is the systematic process of following a signal path from its source to
its destination, measuring at each accessible point, to find where the signal deviates
from the expected value.

```
Start at the signal source (e.g., a crystal output or MCU pin):
  - Is the signal present here? If not, the source itself is suspect.
  - Does the signal have the correct frequency, amplitude, and waveshape?

Move along the signal path toward the destination:
  - Measure at each accessible test point or via
  - First deviation from expected values identifies the faulty segment

At the destination:
  - Is the signal correctly received? (correct logic level, correct threshold)
```

### Divide and Conquer

For a complex fault with many possible causes:

1. Identify the physical midpoint of the suspect signal path or power distribution
2. Measure at that midpoint — is the fault upstream or downstream?
3. Repeat, halving the search space each iteration
4. Result: binary search converges in log2(N) measurements

### Substitution

Replace a suspect component with a known-good component and observe whether the fault
resolves. This is the most definitive test but is invasive (requires desoldering).

Apply substitution only after signal tracing has identified the suspect component —
random substitution is slow and risks introducing new faults during rework.

### Signal Injection

Apply a known signal to a point in the circuit and observe the output. Useful for:
- Testing a signal path in isolation (bypass the normal signal source)
- Testing a filter or amplifier (inject a swept frequency, observe response)
- Testing a communication interface (inject a known waveform, observe if the receiving
  device decodes it correctly)

---

## Best Practices

- Label all probe grounds. Ground loops between probe ground clips cause errors that
  look like the circuit is misbehaving when the measurement setup is the problem.
- Measure twice before cutting. Verify a measurement from at least two different probe
  points before concluding a fault exists.
- Keep a fault log. Record each measurement result, whether it was expected, and what
  action was taken. Debugging without a log leads to testing the same hypotheses
  multiple times.
- A digital logic signal that reads correct voltage but causes protocol errors usually
  has a signal integrity problem (reflections, crosstalk, timing margin). Switch from
  multimeter or logic analyser to an oscilloscope.
- When in doubt, remove power before probing. Accidental short circuits with a probe
  tip on a live board cause more damage than the original fault in many cases.

---

## Tier 1 — Fundamentals

### Question F1
**Why should you never use the long flying-lead ground clip when probing a signal above 10 MHz? What should you use instead?**

**Answer:**

The long flying-lead ground clip (typically 15 cm) has an inductance of approximately
150 nH. This inductance, combined with the probe's tip capacitance (~15 pF), forms a
series LC resonant circuit with a resonant frequency of:

```
f_resonance = 1 / (2π × √(LC)) = 1 / (2π × √(150nH × 15pF)) ≈ 106 MHz
```

At and near this frequency, the ground return impedance becomes very high, causing the
probe to measure the signal poorly and introduce ringing artefacts on fast edges. Below
about 5 MHz the effect is negligible; above 10 MHz it becomes significant; near 100 MHz
it dominates the measurement.

**Use instead:**
- The short ground spring accessory (2-5 mm spring, L < 5 nH)
- A PCB-mounted test socket with a coaxial ground return
- For high-frequency measurements: a probe designed for coaxial SMA test points

---

### Question F2
**An oscilloscope shows your 3.3 V power rail has 150 mV peak-to-peak ripple at the switcher frequency. Is this a problem? How was the measurement taken correctly?**

**Answer:**

Whether 150 mV peak-to-peak on a 3.3 V rail is a problem depends on the load:

- 150 mV is 4.5% of 3.3 V. Many digital ICs specify a ±5% tolerance on their supply
  rail, making 150 mV ripple marginal but technically within spec.
- For analogue circuits (ADC reference, precision op-amp supply), 150 mV of ripple at
  the switching frequency is almost certainly problematic and will appear as tones in
  the output spectrum.
- For high-speed digital logic with tight timing margins, supply ripple modulates
  propagation delay and should ideally be < 50 mV (< 1.5%).

**Correct measurement procedure:**
1. AC-couple the oscilloscope input (removes the 3.3 V DC component, shows only the
   AC component on a sensitive vertical scale)
2. Set vertical to 50 mV/div or 100 mV/div
3. Bandwidth limit OFF (to see the full spectral content of the ripple)
4. Place probe tip at the regulator output capacitor terminal and ground spring at
   the capacitor's ground terminal — not at a remote test point (routing resistance
   adds voltage drop that inflates the reading)

If the ripple is higher than expected: check output capacitor value and ESR, switching
frequency, load transient current, and loop compensation.

---

## Tier 2 — Intermediate

### Question I1
**You suspect an I2C communication failure between a microcontroller and a sensor. Describe the full diagnostic approach, from initial check to root cause identification.**

**Answer:**

**Step 1 — Physical layer verification.**
Probe SCL and SDA simultaneously on an oscilloscope (10x probe, short ground spring,
DC coupled, 1 V/div, 10 µs/div). Observe whether:
- Both lines idle at their pulled-up high voltage (3.3 V or 1.8 V as appropriate)
  when no communication is occurring. If either line is stuck low, there is a bus
  contention or a device that is holding the line.
- Edges are clean: rise time should be < 300 ns for 100 kHz I2C, < 120 ns for 400 kHz.
  If rise time is too slow, the pull-up resistor is too large or the bus capacitance
  is too high. If edges are distorted or have overshoot, the pull-up is too small.

**Step 2 — Protocol decode.**
Use the oscilloscope's I2C decode or connect a logic analyser with I2C protocol decode.
Look for:
- Does the master generate valid START and STOP conditions?
- Is the target address correct (7-bit address + R/W bit)?
- Does the slave ACK the address? (ACK = SDA pulled low during 9th clock)
  No ACK on address = slave not responding, which indicates either wrong address,
  slave not powered, slave in reset, or hardware address pins in wrong state.
- Are data bytes ACKed or NACKed?

**Step 3 — Address verification.**
Check the hardware address pins (A0, A1, A2) on the sensor IC. The I2C address is
typically composed of a fixed 7-bit prefix plus bits set by these pins. If any address
pin is not connected to its expected level (floating, or connected to wrong rail), the
sensor will respond to a different address than expected.

**Step 4 — Power and reset state.**
Verify the sensor is powered. Check the sensor's /RESET or ENABLE pin is in the
active (not held in reset) state. Some sensors require a software reset command before
they respond to normal read commands.

**Step 5 — Timing margins.**
If the protocol decode shows framing but intermittent NACKs: the bus timing may be
marginally too fast for the sensor. Check the sensor's maximum I2C clock frequency.
Reduce the firmware-configured clock frequency and observe whether errors decrease.

---

## Tier 3 — Advanced

### Question A1
**You are debugging an intermittent failure that only occurs after the board has been operating for 20-30 minutes. The failure manifests as incorrect data from a 16-bit ADC connected via SPI. How do you diagnose whether the root cause is thermal, power supply, or signal integrity?**

**Answer:**

Intermittent failures that appear after thermal soak are among the most challenging
to diagnose because the fault condition cannot be induced on demand in the early phase.

**Step 1 — Characterise the failure mode precisely.**
Log the ADC output values continuously to a file (via UART or USB from the
microcontroller). Note: is the fault a fixed offset (systematic), a random error
(random noise), or a single stuck bit (digital fault)? The failure mode type guides
the subsequent diagnostic direction.

- Fixed offset after warmup: thermal coefficient of a reference component
- Random errors: noise floor issue, possibly thermal noise in reference or source
  impedance of the signal being measured
- Single stuck bit: digital signal integrity or internal ADC failure

**Step 2 — Assign a scope to run unattended.**
Set up an oscilloscope in persistent or infinite persistence mode on the SPI clock and
MISO lines, triggered on the fault condition if a digital protocol error is detectable.
Some ADC ICs assert an alert pin on conversion error — trigger on that. Leave the scope
running. When the fault occurs, the screen captures the waveform at the moment of failure.

**Step 3 — Monitor power rail simultaneously.**
Add a second oscilloscope monitoring the AVDD and DVDD rails of the ADC (AC-coupled,
50 mV/div) and the ADC reference voltage (DC-coupled). Timestamp the power measurements
against the fault log. If the power rail noise floor increases as the board warms up,
this is a power supply thermal issue.

**Step 4 — Apply thermal stress selectively.**
Use a hot air gun or freeze spray to change the temperature of individual components
while the board is running and the fault log is active. Heat the voltage reference,
then cool it. Heat the ADC. Heat the decoupling capacitors near the ADC power supply
pins. Correlate the timing of any fault occurrences with the thermal intervention.

**Step 5 — Check the physical connection quality.**
Inspect the ADC solder joints under magnification. If the fault correlates with thermal
cycling (appears as the board warms up, disappears when cooled), suspect a solder joint
with a thermal coefficient that causes it to open when hot. Resolder all pins on the
ADC and any series resistors in the SPI path, then retest.

**Step 6 — Check signal integrity at temperature.**
If the scope capture at failure shows degraded SPI signal quality (slow edges, reduced
swing, overshoot), the problem is signal integrity that worsens with temperature. This
could be:
- Trace impedance changing with temperature (minor effect, rarely significant)
- Series resistors in the SPI path with high temperature coefficients
- The source impedance of the MISO driver increasing at high temperature (common in
  some CMOS output stages operating near supply limits)

---

## Related Topics

- [Bring-Up Methodology](bringup_methodology.md)
- [Power-On Sequence](power_on_sequence.md)
- [Rework and ECN](rework_and_ecn.md)
- [Worked Problem: Clock Failure](worked_problems/problem_02_clock_failure.md)
- [Worked Problem: Thermal Issue](worked_problems/problem_03_thermal_issue.md)
