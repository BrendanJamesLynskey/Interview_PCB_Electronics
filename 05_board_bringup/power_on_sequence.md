# Power-On Sequence

## Prerequisites
- Understanding of MOSFET and BJT operation in switching applications
- Familiarity with power supply topologies: LDO, synchronous buck, PMIC
- Basic understanding of CMOS latch-up mechanism
- Ability to read IC datasheet absolute maximum ratings and power sequencing sections

---

## Concept Reference

### Why Sequencing Matters

Modern SoCs, FPGAs, and microprocessors contain multiple internal voltage domains. The
I/O interface cells (which communicate with other chips) often operate at a higher
voltage (3.3 V, 2.5 V, 1.8 V) than the core logic (1.0 V, 0.85 V, 0.7 V). If the I/O
supply is present but the core supply is absent, the I/O ESD protection diodes can
become forward biased, injecting minority carriers into the silicon substrate.

In CMOS technology, the substrate contains parasitic bipolar transistors formed by
adjacent P-N junctions. If these parasitic transistors are triggered simultaneously,
a low-impedance path between VCC and GND is formed — this is **latch-up**. Latch-up
is self-sustaining: the device remains latched even after the triggering event ends,
and the only way to release it is to remove power. If latch-up current is high enough,
the device is permanently destroyed.

Power sequencing requirements prevent the conditions that cause latch-up.

### Common Sequencing Requirements

Most datasheets specify sequencing in one of three ways:

```
1. Simultaneous: All rails must rise within a specified window of each other
   (e.g., "all supplies must be within 100 ms of each other")

2. Ordered: Rails must start in a defined sequence, with or without a minimum
   interval between them
   (e.g., "VCCINT must reach 0.6 V before VCCAUX begins to rise")

3. Ratio-based: The ratio of two supply voltages must never exceed a defined value
   during rise and fall
   (e.g., "VCCIO / VCCCORE must never exceed 2.0 during any transient")
```

A datasheet that states no sequencing requirement does not mean no sequencing is needed
— it means the device is internally protected against out-of-sequence power-on. Verify
this explicitly before assuming.

### Sequencing Methods

**Method 1 — PMIC with integrated sequencer**

A dedicated PMIC provides the simplest and most reliable sequencing. The PMIC
sequences its output rails according to a configuration stored in OTP or an I2C
register map. Each channel has programmable:
- Start delay after the previous rail asserts PGOOD
- Soft-start ramp rate
- Enable/disable order for power-down

```
Advantages: All timing defined in one place, integrated fault monitoring,
            I2C visibility into operating state
Disadvantages: Requires PMIC with sufficient channel count, I2C configuration
               complexity, single-point fault risk
```

**Method 2 — Enable chain (daisy-chained enable signals)**

The PGOOD output of the first regulator drives the ENABLE input of the second, the
PGOOD of the second drives the ENABLE of the third, and so on. The power-up sequence
is determined by the physical connection order.

```
3.3V LDO ─── PGOOD ──► EN[1.8V Buck] ─── PGOOD ──► EN[1.0V Buck]
   │                         │                          │
   └── 3.3V rail             └── 1.8V rail              └── 1.0V rail
```

```
Advantages: Simple, requires no external controller, robust to I2C failure
Disadvantages: Difficult to change sequencing in hardware, delays are fixed by
               regulator soft-start and PGOOD assertion threshold only
```

**Method 3 — Supervisor IC with sequencer outputs**

A dedicated power supervisor (e.g., TI TPS3700, Maxim MAX16046) monitors all input
rails and asserts sequenced enable signals with programmable delays. This separates
the power conversion function from the sequencing control function.

**Method 4 — Discrete RC delay on ENABLE pin**

A simple RC network delays the ENABLE signal of a downstream regulator. This is
the lowest-cost method but the timing is poorly controlled (±20% on RC values in
production, plus regulator ENABLE threshold variation). Use only when timing
requirements are loose (hundreds of milliseconds, not microseconds).

```
         R1
VCC ──── 10kΩ ──── ENABLE
              │
             C1    (RC time constant sets delay)
              │
             GND
```

---

### Power-Down Sequencing

Power-down sequencing is the reverse of power-up sequencing in most designs. The reason
is the same: avoiding the condition where a high-voltage I/O supply is present while
the core supply is absent.

**Critical mistake:** Many designers correctly implement power-up sequencing but neglect
power-down. If the 1.0 V core collapses first (due to overcurrent protection or fault)
while the 3.3 V I/O remains up, the device experiences an out-of-sequence power-down
that is equivalent to an out-of-sequence power-up — latch-up is equally possible.

PMICs should be configured with power-down sequencing that is the inverse of power-up:
1.0 V core last to rise → 1.0 V core first to fall.

---

### Latch-Up: Mechanism and Prevention

**Physical mechanism:**

In a standard CMOS process, each NMOS transistor is built inside a p-type well, and
each PMOS transistor is built inside an n-type well. These wells form parasitic PNP
and NPN bipolar transistors with the substrate. When both transistors are simultaneously
turned on, they form a regenerative feedback loop (PNPN thyristor structure):

```
VCC
 │
 ├── P+  (PMOS source/drain)  ─── Parasitic PNP collector
 │                                        │
 │   N-well                    Parasitic NPN base
 │                                        │
 ├── N+  (NMOS source/drain)  ─── Parasitic NPN collector
 │
GND
```

When the collector current of either BJT exceeds the holding current of the structure,
both transistors saturate simultaneously, creating a direct VCC-to-GND path.

**Triggering conditions:**
- Voltage applied to an I/O pin exceeds VCC + 0.5 V (forward-biases protection diode)
- Voltage on I/O pin is below GND - 0.5 V
- High dV/dt on I/O pin displaces current through junction capacitances
- I/O supply present but core supply absent (the condition sequencing prevents)

**Prevention in design:**
1. Enforce power-up and power-down sequencing
2. Add series resistors (e.g., 22-100 Ω) on I/O signals that could be driven before
   the device's supply is present — limits injection current
3. Select devices with "latch-up tested to ±100 mA injection" for interfaces exposed
   to out-of-sequence signals (JEDEC JESD78 test standard)
4. Ensure ESD protection clamps limit I/O voltage excursions

**Prevention in bring-up:**
- Never apply signals to a device's I/O pins before its supply is active
- When using JTAG during bring-up, verify TCK, TDI, TMS are not driving the device
  before VCCIO is present (some JTAG adapters drive these signals at all times)
- If a board shows symptoms of latch-up (extremely high current draw that persists
  after removing the triggering condition), cycle the supply completely off for
  several seconds before re-applying

---

### Soft-Start and Inrush Current

Every power supply that drives a significant output capacitance experiences an inrush
current at start-up. Without soft-start control, the inrush current to charge output
capacitance C through resistance R follows:

```
I_inrush = V_in / R_total  (limited only by source impedance and ESR)
```

For a 3.3 V rail with 100 µF of output capacitance and a source impedance of 0.5 Ω,
the inrush current would be 6.6 A without soft-start — likely exceeding the regulator's
current limit and causing a fault condition.

**Soft-start** ramps the output voltage slowly (typically 1-10 ms), limiting the
charging current:
```
I_inrush_controlled = C × dV/dt  =  100 µF × (3.3 V / 5 ms) = 66 mA
```

During bring-up, soft-start ramp rates may need adjustment if:
- The current-limited bench supply triggers its limit during the inrush period
  (increase soft-start time or increase current limit)
- Rails fail to start because PGOOD is not asserted before the sequencing timeout
  (slow soft-start causes the upstream PGOOD to assert too late)
- EMI conducted emissions testing shows excessive low-frequency noise content
  (soft-start duration affects sub-MHz spectral content of the power bus)

---

## Sequencing Principles

### Determining the Required Sequence from Datasheets

The bring-up engineer must extract sequencing requirements before the board arrives:

1. Read the "Power-On Reset" or "Power Supply Sequencing" section of every IC datasheet
2. Record the requirement type: simultaneous / ordered / ratio-based
3. Draw a sequencing diagram showing every rail, its dependencies, and time relationships
4. Verify the PMIC or sequencer configuration implements the diagram exactly

Example sequencing diagram for a Xilinx Kintex-7 FPGA:
```
VCCINT (1.0 V) ─── starts ──► VCCAUX (1.8 V) ─── starts ──► VCCO_xx (variable I/O)
                               (after VCCINT > 0.6 V)         (after VCCAUX > 1.5 V)
```

### Common Sequencing Pitfalls

**Pitfall 1 — Ignoring the power-down sequence.**
Design the power-down sequence during schematic design, not as an afterthought.

**Pitfall 2 — PGOOD threshold mismatch.**
The PGOOD assertion threshold of one regulator may be 90% of nominal. If the next rail
needs 95% of the previous rail before starting, a 90% PGOOD threshold creates an
undefined timing margin. Verify the full chain with measured thresholds.

**Pitfall 3 — Load-dependent rail droop.**
A rail that is within tolerance at no load may sag below the PGOOD deassertion
threshold under full load. If this occurs, the sequencer interprets it as a fault and
shuts down all downstream rails in sequence. Measure rail voltage under full load before
declaring the sequence correct.

**Pitfall 4 — ESD protection diode conduction during signal-before-supply.**
JTAG adapters, debug cables, and other external connections often drive signals onto
the board before the board's supply is enabled. Add 100 Ω series resistors on all
external interface signals if this cannot be prevented.

---

## Design Considerations

### Selecting a Sequencing Topology for New Designs

| Criterion | PMIC | Enable chain | Supervisor IC | RC delay |
|-----------|------|-------------|---------------|---------|
| Number of rails | Up to 8+ in one IC | Any (chain length) | Up to 6 typical | Any |
| Timing accuracy | ±5% (register-controlled) | ±20% (RC dominated) | ±10% | ±30% |
| Fault detection | Integrated | None | Optional | None |
| Design visibility | I2C register map | None | Limited | None |
| Cost | Medium-high | Low | Low-medium | Very low |
| Recommended for | Complex SoC boards | Simple 2-3 rail boards | Medium complexity | Prototypes only |

### Decoupling the Sequencer from the Converters

For high-reliability designs, use a dedicated sequencer IC separate from the power
converters. This decoupling means:
- A failure of one converter does not affect the sequencer's ability to shut down other rails
- The sequencer can be from a different supply domain and remain powered after the
  main rails have shut down, allowing fault logging
- The converter can be replaced with a different device without changing the sequencing
  logic

---

## Best Practices

- Always simulate or calculate the expected power-up time for each rail before the
  board arrives. Deviations from expected timing are diagnostic information.
- Test power-down sequence as well as power-up. Remove the input supply abruptly and
  capture all rails simultaneously on a four-channel scope.
- Test what happens when one rail faults during operation: does the board shut down in
  the correct sequence, or do some rails remain up after others have collapsed?
- For battery-powered designs, test power-up at minimum battery voltage (3.0 V for a
  single-cell Li-ion). Some regulators fail to start or exhibit wrong sequencing at
  low input voltages.
- Place test points on all PGOOD and ENABLE signals. These are the most useful signals
  for diagnosing sequencing failures.

---

## Tier 1 — Fundamentals

### Question F1
**What is CMOS latch-up and what power sequencing condition most commonly triggers it?**

**Answer:**

Latch-up is the inadvertent triggering of the parasitic PNPN thyristor structure
inherent in CMOS processes. When triggered, the device forms a low-impedance path
between VCC and GND that is self-sustaining — it continues to conduct even after the
triggering stimulus is removed. The device draws very high current and is likely to be
permanently damaged unless the supply is cycled.

The most common power sequencing trigger is the **I/O supply being present while the
core supply is absent**. In this condition:

- The I/O ESD clamping diodes (connected to VCCIO) are biased from the I/O supply
- The I/O cells attempt to pull current from the core supply through internal connections
- With the core supply absent (zero volts), these currents forward-bias the parasitic
  junctions in the substrate
- If injection current exceeds the holding current of the PNPN structure, latch-up occurs

Prevention: ensure the core supply (VCCINT, VCCCORE) rises before or simultaneously
with the I/O supplies.

---

### Question F2
**What is soft-start in a DC-DC converter, and why is it relevant to power sequencing?**

**Answer:**

Soft-start is a feature of regulated power supplies that ramps the output voltage
gradually from zero to the regulated value over a controlled time interval (typically
1-10 ms for board-level supplies). The ramp rate is controlled by an internal current
source charging an internal or external timing capacitor (SS pin).

Relevance to sequencing:

1. **Inrush current limiting:** Without soft-start, the output capacitor charges
   instantaneously, drawing a large spike of current that can exceed the converter's
   current limit, trigger upstream fuses, or cause adjacent rails to droop.

2. **PGOOD assertion timing:** A slow soft-start ramp delays the PGOOD assertion until
   the rail reaches regulation. In an enable-chain sequencer, this delay determines how
   long the downstream rail waits. If soft-start is too slow, the sequencing timeout
   may expire before PGOOD asserts, causing a false fault condition.

3. **Interaction with sequencing hold-off:** If a downstream enable must not assert
   until the upstream rail has been stable for a defined hold-off time (as some
   datasheet requirements specify), the soft-start time of the upstream rail plus the
   hold-off time sets the minimum total delay that must be accommodated.

---

## Tier 2 — Intermediate

### Question I1
**Draw a three-rail sequencing circuit using an enable chain. Label all PGOOD and ENABLE connections. What is the main limitation of this approach?**

**Answer:**

```
Input supply
    │
    ▼
┌─────────────┐
│ 3.3 V LDO   │── 3.3 V rail (always enabled first)
│  VOUT=3.3V  │
│  PGOOD ─────┼──────────────────────────────┐
└─────────────┘                              │
                                             │ ENABLE
                                    ┌────────▼──────────┐
                                    │ 1.8 V Buck        │── 1.8 V rail
                                    │  VOUT=1.8V        │
                                    │  PGOOD ───────────┼──────────────┐
                                    └───────────────────┘             │ ENABLE
                                                                ┌─────▼──────────┐
                                                                │ 1.0 V Buck     │── 1.0 V rail
                                                                │  VOUT=1.0V     │
                                                                └────────────────┘
```

Operation: 3.3 V LDO starts (always enabled). When 3.3 V reaches regulation, PGOOD
asserts and enables the 1.8 V buck. When 1.8 V reaches regulation, its PGOOD asserts
and enables the 1.0 V buck.

**Main limitations:**
- **No fault visibility:** If the 1.8 V rail trips its overcurrent protection and
  disables, the 1.0 V rail will also shut down (its ENABLE is removed). The root cause
  of the fault (1.8 V OCP) is not distinguishable from a normal shutdown based on
  ENABLE state alone. There is no register map to read.
- **No power-down sequence control:** Power-down is determined by which rail collapses
  first, not by a designed sequence. Adding power-down sequencing requires additional
  circuitry.
- **Timing accuracy:** The delay between PGOOD assertion and the downstream rail
  reaching regulation is set by the regulator's soft-start ramp — not a precise digital
  timer. Timing accuracy is ±20% in practice.

---

## Tier 3 — Advanced

### Question A1
**You are debugging a sequencing issue where a 1.0 V core rail on a new SoC board powers up correctly in isolation (bench supply, no load), but when the full sequencing circuit is active, the 1.0 V rail tries to start, partially rises to 0.6 V, then collapses. The other rails remain up. Describe your diagnostic methodology.**

**Answer:**

The symptom — partial rise then collapse of the 1.0 V rail while upstream rails remain
up — indicates that the 1.0 V buck converter is starting normally but experiencing an
overcurrent or fault condition that triggers its protection circuit.

**Step 1 — Characterise the waveform precisely.**
Capture the 1.0 V rail, the 1.8 V rail, and the switching node of the 1.0 V buck
simultaneously. Use a current probe on the 1.0 V output path if available.

Key observations:
- Does the rail rise steadily then collapse suddenly (OCP), or does it rise and then
  oscillate (loop stability issue)?
- Is the 1.8 V input to the buck drooping during the 1.0 V startup attempt? If yes,
  the 1.8 V source is insufficient and the 1.0 V converter is being starved.
- Does the switching frequency change during the collapse (hiccup mode OCP vs.
  immediate shutdown OCP)?

**Step 2 — Check the load.**
The board is populated in the scenario (the SoC is present). The SoC's 1.0 V core
supply current at startup can be significantly higher than at quiescent:
- FPGA and SoC devices often have high inrush current as internal capacitances charge
- Some devices perform power-on self-test (POST) routines that are higher-power than
  quiescent operation
- Check the datasheet for "power-on current" specifications — these are often different
  from (higher than) the operational current figures

**Step 3 — Check the buck converter configuration.**
- Verify the current limit setting: compare the sense resistor or current limit pin
  resistor value on the board against the design intent. A wrong value can set the
  OCP threshold 50% too low.
- Verify the soft-start capacitor: too small a CSS causes too fast a ramp, exceeding
  the current limit during startup.
- Verify the output capacitor: too much output capacitance requires more current to
  charge, potentially exceeding the soft-start-constrained current.

**Step 4 — Temporarily isolate the SoC load.**
Cut the enable pin of the 1.0 V rail or remove the SoC (if socketed) and observe
whether the 1.0 V rail now rises correctly to 1.0 V. If it does, the fault is
load-side (SoC inrush exceeds converter OCP limit). If it still collapses, the fault
is in the converter circuit itself.

**Step 5 — Resolution paths.**
- If SoC inrush: increase the OCP threshold (change sense resistor), or increase
  soft-start time (increase CSS), or add pre-charge to output capacitors.
- If converter fault: check for a bridge on the switching node to PGND, wrong inductor
  value, or misconfigured feedback network (wrong output voltage causing the converter
  to try to regulate to a lower than expected voltage and cycle into OCP).

---

## Related Topics

- [Bring-Up Methodology](bringup_methodology.md)
- [Debugging Techniques](debugging_techniques.md)
- [Power Supply Architecture](../01_schematic_design/power_supply_architecture.md)
- [Worked Problem: No Power Debug](worked_problems/problem_01_no_power_debug.md)
