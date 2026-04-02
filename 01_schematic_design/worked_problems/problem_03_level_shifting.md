# Problem 03: Level Shifting

## Problem Statement

A mixed-voltage embedded system requires bidirectional and unidirectional signal translation between four voltage domains:

```
Domain    Voltage   ICs in domain
------    -------   -------------
A         3.3 V     ARM Cortex-M4 MCU (VIH_min = 2.0 V, VIL_max = 0.8 V, IOH = 8 mA)
B         1.8 V     Application processor (VIH_min = 1.26 V, VIL_max = 0.54 V, IOH = 4 mA)
C         5.0 V     Legacy SPI ADC (VIH_min = 3.5 V, VIL_max = 1.0 V, TTL compatible)
D         1.2 V     LPDDR5 memory (requires strict 1.2 V VOH/VOL — not directly controllable by MCU)
```

The following interfaces must be implemented:

1. **SPI bus (SCLK, MOSI): MCU (Domain A, 3.3 V) → ADC (Domain C, 5 V).** Unidirectional, push-pull, 10 MHz max clock rate.
2. **I2C bus (SDA, SCL): MCU (Domain A, 3.3 V) ↔ Application processor (Domain B, 1.8 V).** Bidirectional, open-drain, 400 kHz (Fast-mode).
3. **GPIO interrupt line: Application processor (Domain B, 1.8 V) → MCU (Domain A, 3.3 V).** Unidirectional, push-pull, low speed.
4. **LPDDR5 data interface: Application processor (Domain B, 1.8 V) → memory (Domain D, 1.2 V).** Note: state whether a discrete level shifter is needed.

For each interface, design an appropriate level shifting solution and justify the choice.

---

## Solution Approach

### Background: Why Level Shifting Fails Without Proper Design

Before solving each interface, it is important to understand the failure modes of naive approaches:

```
Naive approach 1 — Direct connection:
  3.3 V MCU GPIO → 1.8 V processor GPIO
  Problem: 3.3 V HIGH signal exceeds 1.8 V domain's absolute maximum input voltage
           (typically VDD + 0.3 V = 2.1 V). The 3.3 V input will forward-bias the
           1.8 V IC's input protection diode into VDD_1V8, causing:
             a) Latch-up if the parasitic thyristor triggers
             b) Permanent damage to the protection diode at sufficient current
             c) Power consumption through the protection diode path

Naive approach 2 — Resistor divider (unidirectional step-down only):
  3.3 V → R1 → junction → R2 → GND; junction → 1.8 V input
  Works for DC or slow signals, but:
    - Drive impedance is high (R1 + R2 || load); signal edge speed limited
    - Cannot be bidirectional
    - VOH at the junction is VDD × R2/(R1 + R2) — must be re-verified for VIH

Naive approach 3 — Using a logic gate from the low-voltage domain:
  Drive the 5 V input from a 3.3 V output, relying on TTL compatibility.
  Only works if the high-voltage device is truly TTL-compatible (VIH ≤ 2.0 V).
  The Domain C ADC has VIH_min = 3.5 V — not TTL compatible. Direct connection fails.
```

---

### Interface 1 — SPI (3.3 V MCU → 5 V ADC), Unidirectional, Push-Pull, 10 MHz

**Why direct connection fails:**

The ADC requires VIH_min = 3.5 V. The MCU's 3.3 V VOH (minimum typically 2.4 V) does not meet this threshold. The 3.3 V HIGH level would be interpreted as undefined (between VIL_max = 1.0 V and VIH_min = 3.5 V — a prohibited region for some ADC inputs).

**Solution options:**

**Option A — Dedicated logic-level translator IC (recommended):**

```
Use: Texas Instruments SN74LVC1T45 (single-bit, unidirectional, up to 420 Mbps)
  or SN74LVC8T245 (8-bit bus, unidirectional)

Operating principle:
  The device has two separate supply pins (VCCA = 3.3 V, VCCB = 5 V).
  Input threshold referenced to VCCA (3.3 V): standard 3.3 V logic levels.
  Output VOH referenced to VCCB (5 V): typically 0.9 × VCCB = 4.5 V.
  This comfortably exceeds the ADC's VIH_min = 3.5 V. ✓

Speed: SN74LVC1T45 propagation delay ≈ 2 ns — compatible with 10 MHz (100 ns period).

Schematic:
  VCCA ──┐                ┌── VCCB (5 V)
  (3.3V) │                │
         │  SN74LVC1T45   │
MCU_SCLK ─┤A             B├── ADC_SCLK
MCU_MOSI ─┤A             B├── ADC_MOSI
         │  DIR tied HIGH │   (A→B direction permanently)
         └────────────────┘

Series resistor (33 Ω) on MCU side to limit current during power-sequencing:
  If 5 V ADC powers up before 3.3 V MCU, VCCB internal clamp diodes may conduct.
  The series resistor limits clamp diode current to (5 - 3.3) / 33 = 52 mA peak.
  (Verify this is within the translator's absolute maximum.)
```

**Option B — MOSFET clamp with pull-up (simple, cheap):**

```
MCU_SCLK ──[R1, 1 kΩ]──┬── ADC_SCLK
                         │
                     [R_pull, 10 kΩ to 5 V]
                         │
                      [Schottky D, VCCB to junction — clamp]

This is a half-measure. The Schottky clamp prevents the 5 V from back-driving
the MCU to dangerous levels, but the voltage at ADC_SCLK is still limited by
the MCU drive voltage (3.3 V VOH) unless a pull-up is used.

With 10 kΩ pull-up to 5 V and 1 kΩ from MCU:
  VOH at ADC_SCLK when MCU drives HIGH (3.3 V): the resistor divider
  3.3 V drives through 1 kΩ, pull-up 10 kΩ from 5 V:
  V_junction = (3.3/1k + 5/10k) / (1/1k + 1/10k) = (3.3 + 0.5) / 1.1 = 3.45 V

  3.45 V < 3.5 V VIH_min of ADC — marginal failure condition.

This approach is inadequate without careful impedance engineering.
Recommendation: Use the dedicated translator IC (Option A) for reliability.
```

**Chosen solution: SN74LVC1T45 or SN74LVC8T245 (multi-bit) translator.**

---

### Interface 2 — I2C (3.3 V MCU ↔ 1.8 V Processor), Bidirectional, Open-Drain, 400 kHz

I2C is inherently an open-drain bus. Each device can only pull the line low — the HIGH state is set by the pull-up resistors. This makes bidirectional level translation simpler because neither device is actively driving HIGH.

**Why the 3.3 V MCU pull-up cannot directly drive the 1.8 V processor:**

If the I2C bus pull-up is connected to 3.3 V, the SDA/SCL lines rest at 3.3 V when idle. This exceeds the 1.8 V processor's maximum input voltage (1.8 + 0.3 = 2.1 V absolute maximum). The bus cannot be pulled up to 3.3 V.

If the pull-up is connected to 1.8 V, the HIGH level is 1.8 V. The MCU's VIH_min = 2.0 V — it will not reliably recognise 1.8 V as a HIGH. The MCU may interpret an idle bus as a stuck-LOW, causing it to attempt bus recovery.

**Solution — BSS138 MOSFET bidirectional level shifter:**

This is the classic discrete bidirectional I2C level shifter used in millions of designs. It exploits the body diode and gate threshold of a small N-channel MOSFET.

```
Circuit (per signal, SDA shown):

    VDD_3V3                  VDD_1V8
       │                       │
    [4.7 kΩ pull-up]        [4.7 kΩ pull-up]
       │                       │
       ├─── SDA_3V3            ├─── SDA_1V8
       │         \             │         \
       │          [BSS138 NMOS]           (1.8 V processor)
       │          Gate tied to VDD_1V8 (1.8 V)
       │          (VGS = 1.8 V, NMOS enhancement threshold ≈ 0.8-1.5 V)
       │
   (MCU, 3.3 V)

Detailed operation — LOW driven from 3.3 V side (MCU pulls SDA low):
  MCU open-drain output pulls SDA_3V3 to GND.
  VGS = VDD_1V8 - 0 V = 1.8 V > VGS_threshold (0.8 V typical).
  MOSFET turns on, pulling SDA_1V8 to GND through the channel.
  Both sides see LOW. ✓

Detailed operation — LOW driven from 1.8 V side (processor pulls SDA low):
  Processor pulls SDA_1V8 to GND.
  VGS = 1.8 V (gate) - 0 V (source, now pulled to GND through processor) = 1.8 V.
  Wait — the source is now at GND (pulled by processor), not at VDD_1V8.
  The gate is still tied to VDD_1V8 = 1.8 V.
  VGS = 1.8 V > threshold → MOSFET turns on strongly.
  SDA_3V3 is pulled low through the channel. ✓

  Additionally, if the MOSFET does not turn on fast enough, the body diode
  conducts: VD = VDD_1V8 (1.8 V), VS = SDA_3V3 side.
  Body diode conducts when VS > VD + 0.7 V → if SDA_3V3 > 2.5 V, body diode
  pulls it toward 1.8 V + 0.7 = 2.5 V.
  Then MOSFET channel takes over, pulling SDA_3V3 fully to GND. ✓

HIGH state (neither side pulling low):
  Both pull-up resistors hold respective sides high: SDA_3V3 = 3.3 V, SDA_1V8 = 1.8 V.
  MOSFET: Gate = 1.8 V, Source = SDA_3V3 = 3.3 V.
  VGS = 1.8 - 3.3 = -1.5 V → NMOS is OFF (VGS is negative).
  The two sides are isolated — each floats to its own pull-up voltage. ✓
```

**Pull-up resistor selection for 400 kHz (Fast-mode):**

```
I2C Fast-mode capacitance budget: max 400 pF per line.
Rise time requirement: tr ≤ 300 ns (I2C Fast-mode specification).

tr = 0.8473 × R_pull × C_bus

For tr = 300 ns, C_bus = 50 pF (estimate for short PCB trace + device inputs):
  R_pull ≤ 300 ns / (0.8473 × 50 pF) = 7.1 kΩ

Use 4.7 kΩ on both sides — provides 190 ns rise time for 50 pF load.
If traces are long or there are multiple devices, re-measure C_bus and recalculate.

Power consumption: I_pullup = VDD / R_pullup
  3.3 V side: 3.3 / 4.7k = 0.70 mA per wire (when pulled low)
  1.8 V side: 1.8 / 4.7k = 0.38 mA per wire
  Total for SDA + SCL: (0.70 + 0.38) × 2 = 2.16 mA worst case
```

**Alternative — Dedicated I2C level shifter IC:**

```
NXP PCA9306: Dual-channel bidirectional I2C/SMBus translator.
  Advantages: Handles clock stretching correctly, faster than BSS138 solution,
  lower parasitic capacitance, simpler schematic.
  Use: Preferred in commercial products. BSS138 solution is preferred for
  prototyping and education due to zero IC cost and part availability.
```

**Chosen solution: BSS138 MOSFET bidirectional shifter (or PCA9306 for production).**

---

### Interface 3 — GPIO Interrupt (1.8 V Processor → 3.3 V MCU), Unidirectional, Push-Pull

This is a simple unidirectional, push-pull, low-speed signal. The 1.8 V processor drives a GPIO that must be read by the 3.3 V MCU.

**Check: Does the 3.3 V MCU recognise 1.8 V as HIGH?**

```
MCU VIH_min = 2.0 V.
1.8 V processor VOH_min = VDD × 0.9 = 1.8 × 0.9 = 1.62 V (worst case).

1.62 V < 2.0 V → The MCU will NOT reliably detect 1.8 V HIGH. ✗

Direct connection fails for the HIGH level. The LOW level is fine:
  Processor VOL_max = 0.2 × VDD = 0.36 V < MCU VIL_max = 0.8 V ✓
```

**Solution options:**

**Option A — Resistor pull-up to 3.3 V (simplest, slow signals only):**

```
1.8 V GPIO (open-drain mode) ──[R_series, 10 kΩ]──┬── MCU GPIO
                                                    │
                                              [10 kΩ pull-up to 3.3 V]

With processor GPIO in open-drain mode:
  Pulled LOW: processor sinks current through R_series and R_pullup.
  Released HIGH: R_pullup pulls to 3.3 V.

MCU sees 3.3 V HIGH (well above VIH_min = 2.0 V). ✓
MCU sees < 0.5 V LOW (well below VIL_max = 0.8 V). ✓

Limitation: Rise time limited by R_pullup × C_load.
  For C_load = 20 pF: tr = 2.2 × 10 kΩ × 20 pF = 440 ns.
  For an interrupt signal toggling at 1 kHz, this is perfectly acceptable.
  For speeds > 100 kHz, reduce R_pullup to 1 kΩ (more current, faster edge).

Risk: The 1.8 V GPIO must be configured as open-drain. If it is accidentally
configured as push-pull, the 3.3 V pull-up fights the processor's active-LOW
output at 1.8 V potential: current = (3.3 - 1.8) / (10 kΩ + output impedance).
With 10 kΩ, the current is (1.5 / 10k) = 0.15 mA — safe, but the voltage at
the MCU input will be between 1.8 V and 3.3 V (a resistor divider), which may
fall in the undefined region for the MCU.
```

**Option B — Single-gate translator (recommended for robustness):**

```
Use: Texas Instruments SN74LVC1T45 (1-bit, any voltage 1.2 V to 5.5 V per side).
Direction pin tied HIGH (B→A direction disabled; A→B direction only — but reversed:
use VCCA = 1.8 V, VCCB = 3.3 V for 1.8 V input, 3.3 V output).

This guarantees a proper logic translation, handles push-pull input correctly,
and is immune to GPIO configuration errors on the processor side.

Cost: approximately £0.15 in volume — justified for a clean interface.
```

**Chosen solution:** For this low-speed interrupt, the open-drain + 3.3 V pull-up approach is used (Option A) due to simplicity. The processor GPIO is configured in open-drain output mode in firmware.

If the GPIO must be push-pull (e.g., the processor GPIO cannot be reconfigured), use the SN74LVC1T45.

---

### Interface 4 — LPDDR5 Interface (1.8 V → 1.2 V)

**Does this require a discrete level shifter?**

No. For several important reasons:

```
Reason 1 — The application processor handles LPDDR5 natively:
  Modern application processors (e.g., Qualcomm Snapdragon, MediaTek Dimensity,
  Arm Neoverse) integrate LPDDR5 PHY (physical layer) directly on-die.
  The DDR PHY contains its own internal voltage translation between the core
  voltage domain and the LPDDR5 signalling domain (which uses VDDQ = 1.1 V or
  0.6 V differential for LPDDR5). No discrete level shifter exists in this path.

Reason 2 — LPDDR5 speed makes discrete level shifting impractical:
  LPDDR5 data rates: 6400-8533 Mbps per pin (LP5) or 3200-4266 MT/s (LPDDR5X).
  Any discrete level shifter in this path would add > 0.5 ns of propagation delay
  and > 0.2 pF of capacitance per pin.
  With 32 data pins: the additional capacitance would degrade signal integrity
  below JEDEC margins at these speeds. The timing budget at 6400 Mbps per pin
  is measured in picoseconds — no room for a discrete shifter.

Reason 3 — LPDDR5 uses a pseudo-open-drain interface:
  LPDDR5 uses a write-levelling and read-training calibration that requires
  the memory and PHY to cooperate at the silicon level. A discrete device
  in this path would disrupt the calibration protocol.

Reason 4 — The memory is directly connected to the SoC:
  In every production LPDDR5 design, the memory packages are placed directly
  adjacent to the SoC, connected by short traces (typically < 10 mm) on the
  same PCB. There is no "between" connection where a level shifter could sit.

Conclusion: No discrete level shifter for LPDDR5. The application processor's
integrated PHY handles voltage translation internally. The PCB designer's job is
to route the LPDDR5 traces with controlled impedance (40 Ω single-ended, 80 Ω
differential for differential pairs) and matched lengths within each byte lane.
```

---

## Complete Level Shifting Summary

```
Interface     From        To         Solution                   Direction   Speed OK?
---------     ----        --         --------                   ---------   ---------
SPI SCLK/MOSI 3.3 V MCU  5 V ADC   SN74LVC8T245               Uni (A→B)   Yes, 10 MHz
I2C SDA/SCL   3.3 V MCU  1.8 V App  2× BSS138 MOSFET shifter  Bidir       Yes, 400 kHz
GPIO IRQ      1.8 V App  3.3 V MCU  Open-drain + 10 kΩ pull-up Uni (A→B)  Yes, <100 kHz
LPDDR5        1.8 V App  1.2 V Mem  None (PHY internal)         N/A        N/A
```

---

## Analysis

### Speed vs Simplicity Trade-off for I2C

The BSS138 MOSFET solution is elegant but has a finite transition speed limited by the pull-up resistor charging the combined gate capacitance of the MOSFET and bus capacitance. For Fast-mode Plus (1 MHz) or higher, the BSS138 solution begins to fail:

```
At 1 MHz (Fast-mode Plus):
  Required rise time: tr ≤ 120 ns
  With 4.7 kΩ pull-up and 100 pF total bus capacitance:
  tr = 0.8473 × 4.7k × 100 pF = 398 ns → exceeds 120 ns limit. ✗

  Reducing pull-up to 1 kΩ:
  tr = 0.8473 × 1k × 100 pF = 84 ns → passes, but I_pullup = 3.3/1k = 3.3 mA.
  At 50% duty cycle, average current from 3.3 V rail = 1.65 mA per wire.
  Acceptable for most designs.

  However, 1 kΩ pull-up increases the settling time of the low-level assertion
  from the processor's current sink capability. Verify VOL at required sink current.

For 1 MHz+ I2C: use PCA9306 or NXP PCA9617 (FM+ capable, 1 MHz, true translator IC).
```

### Protection Against Power Sequencing Faults

When power supplies come up in an unexpected order, level shifter inputs may be driven before their supply is available. This can cause:

```
Scenario: VDD_5V (ADC) comes up before VDD_3V3 (MCU + translator VCCA).
  SN74LVC8T245: VCCA = 0 V, VCCB = 5 V.
  If any 5 V signal appears on the B-port, it forward-biases internal ESD
  diodes into the VCCA = 0 V rail.
  
  Risk: Current flows from 5 V ADC → VCCB ESD diode → VCCA rail → anything
  connected to the uninitialized 3.3 V rail.
  
  Mitigation: Series resistor (100-330 Ω) on each B-port trace limits
  the clamp diode current to (5 V / 330 Ω) = 15 mA — within device ratings.
  The power sequencing should also be reviewed: the 5 V ADC rail should come
  up after the 3.3 V rail where possible.
```

### Why Resistor Dividers Are Not Used for SPI

A resistor divider from 3.3 V to 5 V would require the output voltage to be above VIH_min = 3.5 V:

```
For ADC_SCLK = VDD_5V × R2 / (R1 + R2):
  When MCU drives HIGH (VOH = 3.0 V minimum):
  V_ADC = (3.0 / R1 + 5.0 / R2_pullup_from_5V) / (1/R1 + 1/R2)
  This is complex and varies with MCU output impedance.

For a clean SPI signal at 10 MHz, a resistor network's bandwidth is limited
by the capacitance at the intermediate node. Even 20 pF parasitic and 1 kΩ
resistors give f_3dB = 1/(2π × 1k × 20p) = 8 MHz — borderline for 10 MHz SPI.

Conclusion: For any signal above 1 MHz or with defined logic thresholds,
use a dedicated translator IC rather than a resistor network.
```

---

## Key Takeaways

1. **Always check VIH and VIL of both source and destination against the actual VOH and VOL of the driver.** A logic HIGH from a 1.8 V device (VOH ≈ 1.6 V) will NOT be reliably detected by a 3.3 V device with VIH_min = 2.0 V — even though 1.8 V and 3.3 V are both "LVTTL-family" devices.

2. **I2C's open-drain protocol enables a simple discrete MOSFET level shifter.** The BSS138 solution is well-proven and cost-effective for 400 kHz. At 1 MHz+ speeds, use a dedicated I2C translator IC.

3. **For high-speed interfaces (LPDDR, PCIe, SERDES), discrete level shifting is not feasible.** The SoC or FPGA must include a PHY that natively supports the target interface voltage. This is a system architecture constraint, not a PCB problem.

4. **Direction control in translator ICs matters.** Bidirectional translators with an explicit DIR pin must be correctly tied. Bidirectional auto-sensing translators (used in some I2C translators) infer direction from which side drives low first — verify the auto-sensing latency is acceptable for your protocol.

5. **Power sequencing can stress level shifters.** Include series resistors on translator inputs/outputs to limit clamp diode currents during power-up ordering violations, and specify the intended power-up order in the design documentation.

---

## Interview Notes

**Likely follow-up questions:**

- "Why can't you just connect the 3.3 V SPI clock directly to the 5 V ADC? The ADC probably has protection diodes." → The protection diodes are sized for ESD (milliamp-level transients), not steady-state current. Driving a 5 V input from a 3.3 V source means the internal clamp diode from the input to VDD_5V conducts continuously at (3.3 - 5 + Vf) V — which is reverse-biased, so it does not conduct. But the signal level (3.3 V VOH) does not meet the ADC's VIH_min = 3.5 V. Logic failure is the issue, not physical damage.

- "How does the BSS138 handle clock stretching in I2C?" → Clock stretching occurs when a slave holds SCL low to request a pause. With the BSS138, the slave on either voltage domain can hold SCL low; the MOSFET will pull the other side low correspondingly. Clock stretching is transparent to the level shifter — it simply holds the translated low state for as long as the slave pulls.

- "What is a 'voltage translator' vs a 'level shifter'?" → The terms are used interchangeably in industry. Technically, a level shifter may imply a single-supply solution (e.g., converting from push-pull to open-drain within the same voltage domain), while a voltage translator implies two different supply voltages. In interviews, use either term and clarify the operating voltages explicitly.

- "How would you debug a suspected level-shifting failure on an I2C bus?" → Probe both sides of the level shifter simultaneously with a logic analyser (using the correct voltage threshold on each channel). Look for asymmetric rise times (pull-up too weak) or for one side not toggling when the other does (MOSFET not turning on — check gate voltage and VGS threshold). Check that both sides have pull-up resistors to the correct supply voltage. Verify that neither device is driving the bus as push-pull into an open-drain pull-up.
