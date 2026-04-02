# Power Supply Architecture

## Prerequisites
- Basic circuit theory: voltage dividers, Ohm's law, power dissipation
- Feedback control fundamentals: gain, stability, loop compensation
- Passive components: inductors, capacitors, their parasitics
- MOSFET switching concepts (see `component_selection.md`)

---

## Concept Reference

### Voltage Regulator Selection: LDO vs Switching

#### Linear Regulators (LDO)

A low-dropout (LDO) linear regulator passes current through a series pass element (typically a PMOS transistor) whose gate voltage is controlled by a feedback loop to maintain a fixed output voltage.

```
    VIN ──┬────[Pass transistor]────┬── VOUT
          |                         |
          |    ┌──────────────┐     ├── COUT
          └────┤  Error amp   ├─────┘
               │              │
               │  VREF        │  Feedback (R1/R2 divider)
               └──────────────┘

Power dissipation: P = (VIN - VOUT) × IOUT
Efficiency:        η = VOUT / VIN  (ignores quiescent current)
```

**LDO characteristics:**

```
Parameter          Typical value          Notes
---------          -------------          ------------------------------------
Dropout voltage    50-300 mV              Minimum VIN - VOUT for regulation
Quiescent current  1 µA to 10 mA          Important for battery applications
Output noise       1-100 µV RMS           PSRR drops at high frequency
PSRR at 100 kHz    20-40 dB typical       Switching noise from upstream reaches output
Transient response Fast (no inductor)     Responds in microseconds
PCB area           Very small             Minimal external components
Cost               Very low               Simple silicon process
```

**When to use an LDO:**

- Input-to-output voltage differential is small (< 1-2 V) — dropout voltage is low, efficiency is acceptable.
- Low current loads (< 500 mA) where heat dissipation is manageable.
- Noise-sensitive analogue circuits (ADC reference, PLL, RF VCO) that cannot tolerate switching ripple.
- Post-regulation: an LDO after a switching regulator removes ripple while handling only the small VIN-VOUT differential.
- Point-of-load regulation where a clean, stable voltage is needed from a slightly higher rail.

**LDO power dissipation example:**

```
VIN = 5 V, VOUT = 3.3 V, IOUT = 500 mA:
  P_dissipated = (5.0 - 3.3) × 0.5 = 850 mW
  Efficiency = 3.3 / 5.0 = 66%
  Package: SOT-223 or D2PAK likely required for thermal reasons

VIN = 3.6 V, VOUT = 3.3 V, IOUT = 500 mA:
  P_dissipated = (3.6 - 3.3) × 0.5 = 150 mW
  Efficiency = 91.7%
  Package: SOT-23-5 adequate
```

**Common LDO mistake — minimum load current:** Many LDOs require a minimum load current (typically 1-10 mA) to maintain regulation and stability. With no load or a very light load, the feedback loop may lose control. Check the datasheet minimum load specification.

**PSRR (Power Supply Rejection Ratio):** LDOs attenuate upstream noise, but only at lower frequencies. PSRR typically decreases above 1-10 kHz. A switching regulator feeding an LDO must place the switching frequency where the LDO still provides adequate PSRR. Using an additional low-pass RC filter before the LDO input can help for very sensitive analogue rails.

---

#### Switching Regulators

Switching regulators store energy in inductors or capacitors and transfer it to the output efficiently, with typical efficiency of 85-95%.

**Buck (step-down) converter:**

```
    VIN ──[HS MOSFET]──┬──[L]──┬── VOUT
                       |       |
                  [LS MOSFET]  C   Load
                       |       |
                      GND     GND

VOUT = VIN × D   (D = duty cycle = ton / T)
Output ripple: ΔV = ΔIL / (8 × fsw × COUT)  (for CCM)
              where ΔIL = (VIN - VOUT) × D / (fsw × L)
```

**Boost (step-up) converter:**

```
    VIN ──[L]──┬──[Diode]──┬── VOUT
               |            |
            [MOSFET]       C   Load
               |            |
              GND          GND

VOUT = VIN / (1 - D)
```

**Buck-boost and SEPIC:** Used when VOUT may be above or below VIN. Common in battery-powered devices where the battery voltage spans the load voltage.

**Switching regulator key parameters:**

```
Parameter          Typical value       Notes
---------          -------------       ------------------------------------
Efficiency         85-95%              Peaks at mid-load, drops at light load
Switching frequency 100 kHz - 5 MHz   Higher fsw → smaller L, C; more switching loss
Output ripple       1-50 mV            Depends on L, C, ESR, frequency
Transient response  10-100 µs          Slower than LDO due to inductor
Output noise        High frequency     Requires careful filtering for analogue loads
PCB area            Moderate to large  Inductor is often the largest component
EMI                 Significant        Requires attention to layout and filtering
```

**Switching frequency trade-off:**

```
Lower fsw (100 kHz):        Higher fsw (2 MHz):
  Larger L and C required     Smaller L and C (smaller PCB area)
  Lower switching loss        Higher switching loss (more heat)
  Lower EMI frequency         Higher EMI frequency (easier to filter)
  Better efficiency           Lower peak efficiency
```

---

### Power Tree Design

A power tree is a hierarchical diagram showing all power rails in a system, their sources, conversion method, sequencing relationships, and current requirements.

**Power tree design process:**

**Step 1 — Inventory all loads and their requirements:**

```
Rail     Voltage    Current (max)   Tolerance   Notes
------   -------    -------------   ---------   -------------------------
VCORE    0.85 V     15 A            ±3%         Processor core
VDDR     1.1 V      8 A             ±5%         DDR4 termination
VIO      1.8 V      2 A             ±5%         I/O interfaces
VDDR_PLL 1.1 V      50 mA           ±1%         PLL supply (clean)
VANA     1.8 V      200 mA          ±5%         ADC analogue supply
VANAREF  1.25 V     10 mA           ±0.1%       ADC reference (precise)
VPERI    3.3 V      1 A             ±5%         Peripherals, GPIO
VSYS     5 V        3 A             ±10%        Fan, USB
```

**Step 2 — Select input source and first conversion stage:**

```
Input: 12 V wall adapter or battery
  → 12 V bus (system input rail, often directly from connector after protection/filtering)

12 V bus feeds:
  → Isolated DCDC (if isolation required, e.g., medical or industrial)
  → Non-isolated buck converters for each major voltage domain
```

**Step 3 — Group rails by voltage and noise sensitivity:**

```
High-current rails (>2 A): Dedicated synchronous buck converters
Low-current clean rails: LDO post-regulated from a nearby switching rail
Multiple rails at same voltage: Can share one converter with local LDOs or separate converters
```

**Example power tree for an embedded processing board (12 V input):**

```
12 V IN
   │
   ├── [EMI filter + bulk capacitors]
   │
   ├──[Buck 12V→5V, 5A]──── 5V_MAIN
   │                             │
   │                             ├──[LDO 5V→3.3V, 1A]──── 3.3V_IO (GPIO, peripherals)
   │                             │
   │                             └──[USB protection]──── USB_VBUS (5V, pass-through)
   │
   ├──[Buck 12V→1.8V, 5A]──── 1.8V_DDR
   │                             │
   │                             └──[LDO 1.8V→1.25V, 50mA]──── 1.25V_VREF (ADC reference)
   │
   └──[Buck 12V→0.85V, 20A]─── 0.85V_CORE
```

**Step 4 — Efficiency analysis:**

```
Power consumed by loads:
  P_load = 0.85×15 + 1.1×8 + 1.8×2 + 1.25×0.01 + 3.3×1 + 5×3
         = 12.75 + 8.8 + 3.6 + 0.0125 + 3.3 + 15 = 43.5 W

Power from 12 V supply (assuming 90% converter efficiency each stage):
  P_input ≈ 43.5 / 0.90 ≈ 48.3 W
  Input current ≈ 48.3 / 12 = 4.0 A
  Specify 6 A input connector and fuse for margin
```

---

### Power Sequencing

Many processors, FPGAs, and system-on-chips require their power rails to come up and go down in a specific sequence to avoid latch-up, undefined states, or damage to I/O buffers.

**Why sequencing matters:**

- FPGAs: core voltage must come up before I/O voltage to prevent I/O buffers from becoming forward-biased through protective diodes into an unpowered core.
- Processors: if VIO (I/O) comes up before VCORE, I/O cells may draw excessive current trying to drive internal circuitry that has no supply.
- Memory: JEDEC DDR4 specifies that VDD (1.2 V) must come up before or simultaneously with VDDQ (termination, also 1.2 V) and both before VPP (2.5 V activation).
- Reverse-biasing: if VOUT > VIN for a regulator, internal ESD structures can be forward-biased, damaging the device.

**Sequencing methods:**

**Method 1 — RC delay:** Simple, low-cost, but not precise. A resistor-capacitor network delays the enable signal to the second converter.

```
                ENABLE_1
                   │
                   ├──[PMIC_1 EN]──(first rail)
                   │
                   └──[R]──[C]──[comparator]──[PMIC_2 EN]──(second rail)

Rise time of RC: τ = R × C
Example: R = 100 kΩ, C = 10 µF → τ = 1 s (too slow)
         R = 100 kΩ, C = 100 nF → τ = 10 ms (typical for processor sequencing)
```

**Method 2 — POWER GOOD (PG) chaining:** Each regulator has a POWER GOOD output that goes high when its output reaches regulation. This output enables the next regulator.

```
ENABLE → [Reg1] → PG1 → EN2 → [Reg2] → PG2 → EN3 → [Reg3]

Advantages: rail 2 cannot start until rail 1 is in regulation, regardless of temperature
Disadvantages: requires regulators with PG output; longest chain adds latency
```

**Method 3 — Power management IC (PMIC):** A dedicated PMIC handles sequencing internally with programmable timing, power-good monitoring, fault response, and often OVP/OCP/OTP protection for each rail.

```
Advantages: precise, flexible, handles complex sequences (interleaved, simultaneous)
Disadvantages: cost, complexity, single point of failure for all rails
```

**Powerdown sequencing:** Usually the reverse of power-up. The datasheet may specify exact maximum time between rails during powerdown. If a rail that must come down first remains high while a lower-voltage rail drops (e.g., VIO drops but VCORE is still high), damage can occur through I/O structures.

**FPGA sequencing example (Xilinx UltraScale+):**

```
Required sequence (power-up):
  1. VCCINT (core, 0.85 V)     ─── must come first
  2. VCCO_xxx (I/O banks, 1.8V/3.3V) ─── after VCCINT reaches 90% of nominal
  3. GTX transceivers (MGTAVCC, 0.85 V) ─── after VCCINT

Power-down: reverse of above, or ramp all simultaneously within 100 ms
```

---

## Tier 1 — Fundamentals

### Question F1
**What is the main advantage of a switching regulator over a linear regulator? What is the main disadvantage?**

**Answer:**

**Main advantage — efficiency:**

A linear regulator dissipates the input-output voltage differential as heat: `P_loss = (VIN - VOUT) × IOUT`. Converting 12 V to 3.3 V at 1 A loses 8.7 W as heat and achieves only 27.5% efficiency.

A switching regulator instead transfers energy in discrete packets via an inductor or capacitor, achieving 85-95% efficiency because the pass element switches between hard-on (low voltage drop, low loss) and hard-off (no current, no loss) states.

**Main disadvantage — noise and complexity:**

Switching creates voltage ripple on the output rail (typically 1-50 mV) and generates conducted and radiated electromagnetic interference (EMI) at the switching frequency and its harmonics. The switching waveforms contain high dV/dt and dI/dt edges that couple into nearby circuits. Additionally:

- More external components required (inductor, multiple capacitors, sometimes snubbers).
- Stability must be designed carefully (loop compensation).
- EMI filter at input is typically needed for CE/FCC compliance.
- More complex PCB layout required.

**When each is preferred:**

```
Use LDO when:                        Use switching regulator when:
  VIN - VOUT is small (< 1 V)          High current (> 500 mA)
  Current is low (< 100 mA)           VIN >> VOUT (large differential)
  Ultra-low noise required             Battery powered (efficiency critical)
  Simplest possible design            Cost of heat/power is prohibitive
```

---

### Question F2
**What is dropout voltage in an LDO? What happens if VIN drops below VOUT + V_dropout?**

**Answer:**

Dropout voltage is the minimum difference between VIN and VOUT at which the LDO can maintain output regulation. Below this voltage, the series pass transistor is fully saturated (fully on) and cannot regulate further.

```
Example: LDO with V_dropout = 200 mV, VOUT = 3.3 V:
  Minimum VIN for regulation = 3.3 + 0.2 = 3.5 V
  If VIN = 3.4 V: LDO drops out → VOUT follows VIN (= VIN - dropout ≈ 3.2 V)
  If VIN = 3.3 V: VOUT ≈ 3.1 V — output is below specification
```

**Why it matters in battery-powered systems:**

A lithium-ion cell discharges from 4.2 V (full) to 2.7 V (empty). A system needing 3.0 V regulated output from a single Li-ion cell must use an LDO with dropout voltage ≤ 2.7 - 3.0 = −0.3 V — impossible. Instead, either use a boost converter, or design the system to accept 2.7-4.2 V directly, or use two cells in series with a buck regulator.

**Common mistake:** Selecting an LDO based on its "output voltage 3.3 V" specification without checking the dropout voltage against the minimum expected input voltage.

---

### Question F3
**Draw a simple power-up sequencing diagram where Rail B must come up after Rail A has reached 90% of its nominal value. What circuit achieves this?**

**Answer:**

```
Rail A ────────────────────────────────────── VOUT_A (3.3 V)
                  90% threshold (2.97 V)
                     │
           ──────────┘  Rail A "settled"
           │
Rail B ────┼────────────────────────────────── VOUT_B (1.8 V)
           │
       [delayed start]

Timing:
  t=0:    System enable asserted
  t=2 ms: Rail A reaches 90% of 3.3 V (2.97 V)
  t=2 ms: Rail B enable is released
  t=4 ms: Rail B reaches 90% of 1.8 V
  t=4 ms: System POWER_GOOD asserted
```

**Circuit implementation using POWER GOOD chaining:**

```
VIN ─── [Regulator A] ─── RAIL_A
          │
          └─[PG pin, open-drain]─[R 100k to RAIL_A]─── PG_A signal
                                                            │
                                                    [EN pin of Reg B]
VIN ─── [Regulator B, EN controlled] ─── RAIL_B
```

Regulator A's POWER GOOD output (open-drain, active-low) is pulled high by a resistor to RAIL_A. When RAIL_A crosses the PG threshold (typically 90-95% of VOUT), PG_A goes high and enables Regulator B. This is fully automatic and temperature-independent, unlike an RC timer.

For powerdown, the process reverses: RAIL_A is disabled first. When RAIL_A drops below the UVLO threshold, PG_A pulls low, disabling Regulator B.

---

## Tier 2 — Intermediate

### Question I1
**A microprocessor datasheet requires that VCORE (0.85 V) and VIO (1.8 V) are within 10 ms of each other at power-up, and specifies that VCORE must not exceed VIO by more than 200 mV at any time during power-up or powerdown. Design a sequencing circuit to meet these requirements.**

**Answer:**

**Requirement analysis:**

```
Constraint 1: |t_VCORE_start - t_VIO_start| < 10 ms — both rails appear "together"
Constraint 2: VCORE ≤ VIO + 0.2 V at all times — VCORE must not lead VIO
              This implies VIO should come up first or simultaneously
```

**Proposed sequence:**

```
1. VIO (1.8 V) ramps up first (or simultaneously with VCORE)
2. VCORE (0.85 V) enabled within 5 ms of VIO reaching 80% of nominal
3. Both rails settled within 10 ms total of system enable
```

**Circuit design:**

```
                          5V_MAIN
                             │
SYS_EN ───[PMIC / supervisory IC]
             │              │
             ├──[EN_VIO]──[Buck 1.8V regulator]──VIO
             │                    │
             │               PG_VIO (open drain)
             │                    │ pulled up to 3.3V via 100kΩ
             │                    │
             └──[AND gate]─────────┘
             │         │
          SYS_EN       │
                       └──[EN_VCORE, delayed by R/C 1ms]──[Buck 0.85V]──VCORE
```

**Alternative — PMIC solution:**

A dedicated PMIC (e.g., TI TPS65094 or Renesas ISL68225) allows programming of:
- Individual enable sequence (VIO = step 1, VCORE = step 2)
- Individual ramp time (both set to 1 ms)
- Delay between enable events (set 2 ms delay VIO-to-VCORE enable)
- Power-good monitoring with configurable threshold (90% of nominal)
- Powerdown sequence (VCORE off first, then VIO, within 1 ms)

PMIC approach is preferred in production designs for its precision, testability, and ability to respond to fault conditions (OVP, OCP) on individual rails without requiring discrete logic.

**Verification checklist:**
- Simulate both power-up and power-down sequences.
- Verify that worst-case VIO sag (load transient) does not pull VIO below VCORE + 0.2 V during operation.
- Verify that VCORE turns off before VIO during powerdown.

---

### Question I2
**You have a 5 V input and need to power three rails: 3.3 V at 2 A, 1.8 V at 1 A, and 0.9 V at 3 A. Calculate the total input current required, assuming 90% efficiency for switching stages and 85% efficiency for any LDO stages. Compare an architecture using three independent buck converters vs one 5V→1.8V buck followed by an LDO to 0.9 V.**

**Answer:**

**Architecture A — Three independent buck converters from 5 V:**

```
Load power:
  P_3V3 = 3.3 × 2 = 6.6 W
  P_1V8 = 1.8 × 1 = 1.8 W
  P_0V9 = 0.9 × 3 = 2.7 W
  P_load_total = 11.1 W

Input power per rail (at 90% efficiency):
  P_in_3V3 = 6.6 / 0.90 = 7.33 W
  P_in_1V8 = 1.8 / 0.90 = 2.00 W
  P_in_0V9 = 2.7 / 0.90 = 3.00 W
  P_in_total = 12.33 W

Input current from 5 V: 12.33 / 5 = 2.47 A
Heat dissipated in converters: 12.33 - 11.1 = 1.23 W
```

**Architecture B — 5V→1.8V buck (90%), then 1.8V→0.9V LDO (85% = 0.9/1.8×100%):**

Wait — note that 0.9/1.8 = 50% efficiency, not 85%.

```
LDO efficiency is determined by the voltage ratio:
  η_LDO = VOUT/VIN = 0.9/1.8 = 50%  (not 85%)

Architecture B uses an LDO for the 0.9 V rail:
  P_0V9_delivered = 2.7 W
  P_LDO_input = 2.7 / 0.50 = 5.4 W  (from 1.8 V bus)
  P_LDO_heat = 5.4 - 2.7 = 2.7 W   (dissipated as heat in LDO)

  Total on 1.8 V bus: P_1V8_load + P_LDO_input = 1.8 + 5.4 = 7.2 W
  P_buck_input = 7.2 / 0.90 = 8.0 W  (from 5 V bus)
  P_3V3_buck_input = 6.6 / 0.90 = 7.33 W

  P_in_total = 8.0 + 7.33 = 15.33 W
  Input current from 5 V: 15.33 / 5 = 3.07 A
  Heat dissipated: 15.33 - 11.1 = 4.23 W
```

**Comparison:**

```
Architecture   Input current   Heat dissipated   Recommendation
-----------    -------------   ---------------   --------------------------------
A (3× buck)    2.47 A          1.23 W            Better efficiency, more PCB area
B (buck+LDO)   3.07 A          4.23 W            Poor efficiency, LDO gets very hot
                                                  (2.7 W in small package)
```

**Conclusion:** Using an LDO to step down 1.8 V to 0.9 V is thermally unacceptable at 3 A — the LDO would need to dissipate 2.7 W, requiring a large package and heatsink. Three independent buck converters are the correct architecture. The LDO architecture is only sensible when the current is small (< 100-200 mA) and when noise isolation justifies the efficiency penalty.

---

### Question I3
**What is a power tree "inrush current" problem and how do you mitigate it? Give a specific example involving a large bulk capacitor bank.**

**Answer:**

**Inrush current** is the large transient current that flows when a capacitor bank charges from zero (or a lower voltage) to a supply voltage at power-on. Because capacitors initially appear as a short circuit to an instantaneous voltage step:

```
I_inrush = C × dV/dt

For a low-impedance source with large capacitance:
  If C = 1000 µF, VIN = 12 V, source impedance = 0.1 Ω:
  Peak current = 12 / 0.1 = 120 A (for the first instant)
  Time constant τ = R × C = 0.1 × 1000×10⁻⁶ = 100 µs
```

**Consequences:**

- Large inrush trips overcurrent protection on the source (e.g., power supply, circuit breaker, PCB fuse).
- Voltage droop on the supply bus affects other loads already running.
- Mechanical stress and contact arcing in connectors and relays (can weld contacts).
- Electromigration stress in PCB traces (brief but intense current).

**Mitigation techniques:**

**1. NTC thermistor (passive):**

```
NTC in series: high resistance (5-50 Ω) at cold → limits inrush
               low resistance (< 1 Ω) when warm → low loss in steady state

Limitation: recovery time after powerdown is slow (must cool); not suitable
for frequent hot-plugging or power cycling.
```

**2. Series resistor + bypass relay or MOSFET:**

```
VIN ──[R_limit 2Ω]──┬──[MOSFET bypass]── VOUT/COUT
                    │
                    └──[COUT charges via R_limit]

Sequence:
  t=0:  R_limit limits inrush; C charges with τ = R × C
  t=5τ: COUT fully charged; bypass MOSFET turned on → R_limit bypassed
  Steady state: zero voltage drop across bypass

Inrush with R=2 Ω, C=1000 µF, VIN=12 V:
  I_peak = 12 / 2 = 6 A (vs 120 A without)
  τ = 2 Ω × 1000 µF = 2 ms → MOSFET turned on after ~10 ms
```

**3. Active inrush limiter / eFuse:**

Integrated circuits (TI TPS259xx, Maxim MAX17526) provide programmable current limiting with soft-start ramp:

```
Features:
  - Programmable inrush current limit (set by external resistor)
  - Soft-start: output voltage ramps at controlled dV/dt
  - Overcurrent protection in steady state
  - Reverse polarity and overvoltage protection
  - Fault reporting via FLAG pin

Design: set I_limit = C × dV/dt → choose dV/dt to limit I to acceptable value
  For C = 1000 µF, I_limit = 5 A: dV/dt = 5 A / 1000 µF = 5000 V/s = 5 V/ms
  Rise time to 12 V: 12 / 5 = 2.4 ms
```

**4. Soft-start on DCDC converter:**

Most switching regulators include an internal soft-start function that ramps the output voltage over 1-10 ms, preventing full bus voltage from appearing instantaneously at the converter output capacitance.

---

## Tier 3 — Advanced

### Question A1
**A multi-rail system requires five rails. You have identified two possible architectures: a discrete multi-output architecture and a PMIC. Walk through the trade-offs in terms of BOM cost, PCB area, sequencing precision, fault handling, and reliability.**

**Answer:**

**Discrete multi-output architecture:**

Each rail has its own dedicated regulator IC (buck controller + FETs, or integrated buck), with sequencing implemented via POWER GOOD chaining, RC delays, or supervisory logic.

```
Advantages:
  Fault isolation: one regulator fails without affecting others
  Flexibility: each rail optimised independently (frequency, compensation, topology)
  No single point of failure in the power system
  Replacement parts are standard, widely available
  Layout can be optimised per rail (distribute inductors across the board)

Disadvantages:
  PCB area: 5× inductor footprint + 5× input/output capacitor sets
  BOM cost: 5 regulator ICs, 5 inductors, 5-10 capacitor sets, gate drivers,
            supervisor IC for sequencing
  Design effort: each loop must be compensated individually
  Sequencing: requires external supervisory IC or logic for precise control
  Parametric variation: each regulator has its own tolerance and trimming
```

**PMIC architecture:**

A single PMIC IC contains multiple switching converter cores, LDOs, sequencing logic, and protection.

```
Advantages:
  PCB area: single IC, smaller inductors due to high internal fsw
  BOM cost: fewer ICs, potentially fewer external components
  Sequencing: built-in, programmable via I2C or OTP fuses — precisely controlled
  Fault handling: integrated OVP, OCP, OTP, UVP, fault logs readable via I2C
  Monitoring: real-time telemetry (current, voltage, temperature) via PMBus
  Validated solution: characterised across temperature and process variation

Disadvantages:
  Single point of failure: PMIC failure can take all rails down simultaneously
  Flexibility: cannot easily add a non-standard topology
  Availability risk: PMIC is a custom or semi-custom part; second-source is harder
  Cost: PMIC may be more expensive than discrete parts at high volume
  Design lock-in: changing the PMIC requires re-evaluation of all rails
  Thermal concentration: all power conversion heat in one package footprint
```

**Trade-off summary:**

```
Criterion          Discrete            PMIC
---------          --------            ----
BOM cost           Higher              Lower (mid-volume)
PCB area           Larger              Smaller
Sequencing         External logic      Built-in, programmable
Fault isolation    Good                Poor (all-or-nothing)
Time-to-market     Longer (design)     Shorter (PMIC handles it)
Long-term support  High (standard parts)  Risk if PMIC EOL
Reliability (MTBF) Distributed failures Single point of failure
Monitoring         Requires extra ICs  Built-in telemetry
```

**Industry practice:** In prototype and low-volume (< 1,000 units), PMICs save design time and are preferred. In high-volume consumer products, the BOM cost saving from PMICs is significant. In high-reliability aerospace or industrial systems, discrete architectures with redundant rails are preferred because a single PMIC failure must not destroy the entire system.

---

### Question A2
**A switching regulator exhibits instability (oscillation or limit cycling) during the load transient test. Describe the likely causes and how you would diagnose and fix the problem.**

**Answer:**

Instability in a switching regulator loop manifests as sustained oscillations, envelope modulation of the output ripple, or subharmonic oscillation visible on an oscilloscope.

**Cause 1 — Insufficient phase margin in the voltage feedback loop:**

The feedback network (compensation components: R and C from the error amplifier output to the feedback pin) sets the loop gain and phase response. If the phase margin falls below ~45°, the loop can oscillate.

```
Diagnostic:
  Inject a small AC signal at the feedback point (network analyser or Bode 100)
  Measure loop gain (T) as a function of frequency
  Look for phase at crossover frequency (where gain = 0 dB):
    Phase margin < 30°: probable instability
    Phase margin = 45-60°: target range
    Phase margin > 60°: well-damped but transient response may be sluggish

Fix:
  Adjust Type III compensation (zero placement)
  Reduce crossover frequency (slower loop, more conservative)
  Add additional zero to boost phase at crossover
```

**Cause 2 — ESR of output capacitor out of range:**

Peak current mode control regulators require a minimum output capacitor ESR to stabilise the current sampling ramp. If ESR is too low (all-ceramic output), the ramp modulation is insufficient.

```
Symptom: oscillation at fsw/2 (subharmonic oscillation)

Fix:
  Add slope compensation (most modern controllers have adjustable slope comp)
  OR add a small series resistance to output capacitors (e.g., 10-50 mΩ)
  OR switch to average current mode control topology
```

**Cause 3 — Right-half-plane (RHP) zero in boost or buck-boost:**

Boost converters have an RHP zero at `f_RHP = (1-D)² × Vin / (2π × L × Iout)`. The RHP zero increases gain but decreases phase, making compensation difficult at high bandwidth.

```
Diagnostic:
  Bode plot shows phase dropping unexpectedly fast above mid-frequency
  Instability occurs after load step that shifts operating point

Fix:
  Limit loop crossover frequency to < 1/3 of f_RHP
  This means slower transient response — a fundamental limitation of boost topology
  Use feedforward compensation to reduce RHP zero impact
```

**Cause 4 — Layout-induced instability:**

Long traces in the feedback path pick up switching noise, creating a spurious high-frequency signal in the error amplifier.

```
Diagnostic:
  Oscillation frequency is a non-integer multiple of fsw (not a harmonic)
  Oscillation disappears when a finger is placed near the feedback trace
  Bode plot on bench looks stable — oscillation is parasitic, not in the intended loop

Fix:
  Shorten feedback divider traces; place resistors and compensation caps close to IC
  Use a low-pass RC filter (10 kΩ + 10 pF) at the feedback pin to attenuate
  high-frequency noise injection
  Separate analogue ground plane under the compensation network
```

**Systematic diagnostic approach:**

```
1. Measure output ripple at steady state (no load transient) — is it clean?
2. Apply step load (e.g., 0% to 50% to 100% of rated load via active load)
3. Capture transient response on oscilloscope (AC coupled, 20 MHz BW)
4. If oscillation: measure frequency
   - At fsw/2: subharmonic — compensation / slope comp issue
   - At crossover frequency: loop phase margin issue
   - Random / noise-related: layout issue
5. Measure Bode plot if oscillation is reproducible
6. Adjust compensation, repeat
```
