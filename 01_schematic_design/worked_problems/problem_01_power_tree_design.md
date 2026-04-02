# Problem 01: Power Tree Design

## Problem Statement

You are designing a single-board computer (SBC) based on a hypothetical ARM Cortex-A55 quad-core processor. The board will accept a 12 V DC input from an external wall adapter. The following rails are required by the processor datasheet and peripheral components:

```
Rail name    Voltage   Max current   Tolerance   Consumer
----------   -------   -----------   ---------   ----------------------------------------
VCORE        0.9 V     12 A          ±3%         Processor core (dynamic voltage scaling)
VCPU_MEM     1.1 V     4 A           ±5%         CPU L3 cache and memory subsystem
VDDR         1.35 V    6 A           ±5%         LPDDR4X memory (4 × 512 MB)
VIO_1V8      1.8 V     2.5 A         ±5%         I/O banks: MIPI, USB, SDIO, SPI, I2C
VIO_3V3      3.3 V     1.5 A         ±5%         USB PHY VBUS detect, PCIe, GPIO, LEDs
VANA         1.8 V     300 mA        ±3%         PLL and analogue supply (clean)
VDAC_REF     1.25 V    20 mA         ±0.5%       Audio DAC reference (low noise)
VSYS_5V      5 V       3 A           ±10%        Fan, USB-A VBUS output
```

The processor datasheet specifies:
- VCORE must come up before VCPU_MEM.
- VCPU_MEM must come up before VDDR.
- VIO_1V8 must come up within 5 ms of VCORE.
- VANA must be derived from VIO_1V8 (same silicon supply domain), not directly from a switching regulator.
- All rails must be within regulation before the processor de-asserts its internal power-on reset (POR), which occurs 10 ms after VSYS_5V is stable.

Design the complete power tree for this board.

---

## Solution Approach

### Step 1 — Inventory and Group the Rails

Before selecting regulators, group rails by voltage, current, and noise sensitivity:

```
Group A — High-current core rails (require dedicated high-efficiency synchronous bucks):
  VCORE    0.9 V, 12 A  — highest current, must come up first
  VCPU_MEM 1.1 V, 4 A
  VDDR     1.35 V, 6 A

Group B — I/O and system rails (moderate current, less noise-sensitive):
  VIO_1V8  1.8 V, 2.5 A — feeds VANA via LDO; must be clean enough for PLL
  VIO_3V3  3.3 V, 1.5 A
  VSYS_5V  5 V, 3 A     — fan and USB-A; can be noisy

Group C — Precision/low-noise rails (post-regulated from adjacent Group B rail):
  VANA     1.8 V, 300 mA — LDO post-reg from VIO_1V8 (same voltage, different impedance)
  VDAC_REF 1.25 V, 20 mA — LDO from VIO_1V8, very tight tolerance
```

### Step 2 — Calculate Total Input Power

```
Load power at full current (worst case):

P_VCORE     = 0.9 × 12 = 10.8 W
P_VCPU_MEM  = 1.1 × 4  = 4.4 W
P_VDDR      = 1.35 × 6  = 8.1 W
P_VIO_1V8   = 1.8 × 2.5 = 4.5 W   (includes VANA and VDAC_REF upstream)
P_VIO_3V3   = 3.3 × 1.5 = 5.0 W
P_VSYS_5V   = 5.0 × 3.0 = 15.0 W

Total load power = 47.8 W

Assuming 90% efficiency for each synchronous buck converter stage:
  P_input ≈ 47.8 / 0.90 = 53.1 W
  Maximum input current from 12 V: 53.1 / 12 = 4.4 A

Wall adapter specification: 12 V, 6 A (60 W) — with 25% headroom over worst-case.
PCB input connector and traces: rate for 6 A continuous.
Input fuse or eFuse: set to 5.5 A trip threshold.
```

### Step 3 — Select Converter Topologies

```
VCORE (12 V → 0.9 V, 12 A):
  Ratio: 0.9/12 = 7.5% duty cycle — very low duty cycle.
  Use: Multiphase synchronous buck (2 or 4 phases) to reduce input/output ripple
  and improve transient response. The processor core is typically the most
  dynamic load, with current slew rates of 10-100 A/µs during workload changes.
  Recommended IC: Texas Instruments TPS53681 (multiphase controller, up to 16 A/phase),
  or Renesas ISL6364 (dual-phase, 12 A).

VCPU_MEM (12 V → 1.1 V, 4 A):
  Use: Single-phase synchronous buck (e.g., TI TPS62130, up to 3 A, or TPS53819A).

VDDR (12 V → 1.35 V, 6 A):
  Use: Single-phase synchronous buck (e.g., TI TPS56221, 6 A, 2 MHz).

VIO_1V8 (12 V → 1.8 V, 2.5 A):
  Use: Single-phase synchronous buck. Must be clean enough for VANA LDO to post-regulate.

VIO_3V3 (12 V → 3.3 V, 1.5 A):
  Use: Single-phase synchronous buck.

VSYS_5V (12 V → 5 V, 3 A):
  Use: Single-phase synchronous buck. Can share the same switching frequency with
  VIO_3V3 if they are on different channels of a dual-output controller.

VANA (1.8 V → 1.8 V, 300 mA):
  Use: Low-dropout LDO, post-regulated from VIO_1V8.
  The LDO provides:
    - Very low output noise (< 10 µV RMS with good LDO selection)
    - High-frequency attenuation of switching regulator noise
    - Tight load regulation for PLL supply
  Dropout: < 100 mV at 300 mA — use an LDO with Vdrop ≤ 50 mV at this current.
  The LDO input and output are at the same nominal voltage. The LDO regulates
  precisely at 1.800 V while VIO_1V8 may be anywhere from 1.71 V to 1.89 V
  (±5% tolerance). With VANA requiring ±3%, the LDO must regulate across the
  ±5% VIO_1V8 range. Verify: minimum VIO_1V8 (1.71 V) > VANA (1.8 V × 0.97 = 1.746 V)
  plus LDO dropout (0.05 V) = 1.796 V.
  Margin: 1.71 V is less than 1.796 V — there is insufficient headroom!

  Resolution: Tighten VIO_1V8 tolerance to ±3% (so minimum = 1.746 V), or
  increase VIO_1V8 nominal to 1.85 V (above VANA by 50 mV minimum).
  This is a common oversight in system-level power tree design.

VDAC_REF (1.8 V → 1.25 V, 20 mA):
  Use: Low-dropout precision LDO from VANA (not VIO_1V8 directly).
  Reason: VANA is already low-noise; taking VDAC_REF from VANA instead of
  VIO_1V8 avoids any switching noise coupling from the buck converter.
  ±0.5% tolerance requires an LDO with < 5 mV accuracy (e.g., TI TPS7A49xx,
  0.1% initial accuracy, 10 µV/√Hz noise).
```

### Step 4 — Power Tree Diagram

```
12 V DC INPUT
    │
    ├─[EMI filter: 10 µH CMC + 100 µF ceramic + TVS]
    │
    ├──[2-phase Buck, 12V→0.9V, 12A]──── VCORE
    │   (TI TPS53681 or equiv)               │
    │                                        └─[PG_VCORE]──────────────────┐
    │                                                                       │
    ├──[Buck, 12V→1.1V, 4A]──────────── VCPU_MEM                          │
    │                                        │                              │
    │                                        └─[PG_VCPU_MEM]──────────┐   │
    │                                                                   │   │
    ├──[Buck, 12V→1.35V, 6A]─────────── VDDR                          │   │
    │                                        │                          │   │
    │                                        └─[PG_VDDR]──────────┐   │   │
    │                                                               │   │   │
    ├──[Buck, 12V→1.8V, 2.5A]───────── VIO_1V8                    │   │   │
    │   (clean, low-ripple, ±3%)              │                     │   │   │
    │                                         ├──[LDO 1.8V→1.8V    │   │   │
    │                                         │   300mA]── VANA    │   │   │
    │                                         │       │             │   │   │
    │                                         │       └──[LDO       │   │   │
    │                                         │          1.8V→1.25V │   │   │
    │                                         │          20mA]──VDAC_REF│  │
    │                                         └─[PG_VIO_1V8]───────┘   │   │
    │                                                                    │   │
    ├──[Buck, 12V→3.3V, 1.5A]───────── VIO_3V3                         │   │
    │                                                                    │   │
    └──[Buck, 12V→5V, 3A]────────────── VSYS_5V                         │   │
         │                                                               │   │
         └─[PG_VSYS_5V]──────────────────────────────────────────┐      │   │
                                                                   │      │   │
                                           [Supervisory IC / PMIC] │      │   │
                                           Monitors: PG_VSYS_5V ◄──┘      │   │
                                                      PG_VCORE ◄───────────┘   │
                                                      PG_VCPU_MEM ◄────────────┘
                                                      PG_VDDR ◄─── etc.
                                           Asserts:  PROC_POR_N when all PG high
```

### Step 5 — Sequencing Design

The processor datasheet requires:

```
Required order:   VCORE → VCPU_MEM → VDDR → VIO_1V8 (within 5 ms of VCORE)
All rails ready before PROC_POR_N de-asserts (10 ms after VSYS_5V stable)
```

**Implementation using POWER GOOD chaining and supervisory IC:**

```
t = 0 ms:       12 V applied; input capacitors charge
t = 1 ms:       VSYS_5V Buck enabled (always-on rail); rises to regulation
t = 2 ms:       PG_VSYS_5V asserts → enables VCORE Buck
t = 3 ms:       VCORE reaches 90% of 0.9 V (0.81 V); PG_VCORE asserts
                → enables VCPU_MEM Buck
t = 4 ms:       PG_VCPU_MEM asserts → enables VDDR Buck simultaneously with
                start of VIO_1V8 Buck (must be within 5 ms of VCORE)
t = 5.5 ms:     VDDR and VIO_1V8 both in regulation
t = 5.5 ms:     VIO_1V8 feeds VANA LDO; VANA reaches regulation within 0.5 ms
                VANA feeds VDAC_REF LDO; reaches regulation within 0.5 ms
t = 6 ms:       All rails in regulation; supervisory IC asserts PROC_POR_N
t = 10 ms:      Processor begins execution (internal POR hold-time expires)

Constraint check:
  VIO_1V8 start at ~3 ms (same time as VCPU_MEM enable)
  VIO_1V8 in regulation at ~5.5 ms
  VCORE reached regulation at ~3 ms
  Δt(VCORE_start → VIO_1V8_start) = 1 ms < 5 ms required ✓

  Total time to all rails regulated: ~6 ms < 10 ms POR hold-time ✓
```

**Powerdown sequence (reverse):**
```
Disable PROC_POR_N (processor asserts reset to itself)
Disable VDDR → Disable VCPU_MEM → Disable VCORE
Disable VIO_1V8 (which drops VANA and VDAC_REF)
Disable VIO_3V3
VSYS_5V last (it may power USB/fan still running)
```

---

## Analysis

### Efficiency Calculation

```
Full-load input power (all rails at max, 90% buck efficiency, 85% LDO efficiency):

Buck stages:
  P_in_bucks = (10.8 + 4.4 + 8.1 + 4.5 + 5.0 + 15.0) / 0.90 = 53.1 W
  
LDO VANA (1.8 V → 1.8 V, 300 mA):
  Efficiency ≈ 1.800/1.850 = 97% (assuming VIO_1V8 set to 1.85 V to ensure dropout margin)
  P_LDO_VANA = 0.3 × (1.85 - 1.80) = 15 mW heat

LDO VDAC_REF (1.8 V → 1.25 V, 20 mA):
  Efficiency = 1.25/1.8 = 69%
  P_LDO_VDAC = 0.020 × (1.8 - 1.25) = 11 mW heat (negligible)

Total system efficiency:
  η_system = P_load / P_input = 47.8 / 53.1 = 90.0%

Heat in power conversion: 53.1 - 47.8 = 5.3 W total
  Distributed across all converters on the PCB.
  Each buck dissipates roughly 0.5-1.5 W depending on load.
  No single converter thermal hotspot — good thermal design.
```

### The VANA LDO Headroom Problem — Revisited

This is a real problem that causes integration failures and is worth exploring in depth:

```
VIO_1V8 nominal: 1.80 V, tolerance ±5% → range: 1.71 V to 1.89 V
VANA target:     1.80 V, tolerance ±3% → range: 1.746 V to 1.854 V
LDO dropout:     50 mV (typical low-dropout device at 300 mA)

LDO requires: VIN ≥ VOUT + Vdropout = 1.80 + 0.05 = 1.85 V minimum input

Minimum VIO_1V8 (worst case): 1.71 V
1.71 V < 1.85 V → LDO drops out at minimum VIO_1V8 → VANA goes below 1.746 V
                  → VANA falls outside its ±3% tolerance
                  → PLL and analogue circuits may malfunction

Three solutions:
  Option A: Trim VIO_1V8 to 1.85 V nominal, ±3% tolerance
            Range: 1.794 V to 1.905 V
            Minimum: 1.794 V < 1.85 V — still marginal. Need tighter tolerance.

  Option B: Trim VIO_1V8 to 1.85 V, ±2% tolerance
            Range: 1.813 V to 1.887 V
            Minimum: 1.813 V < 1.85 V — still fails worst case!

  Option C (correct): Raise VIO_1V8 nominal to 1.95 V, ±3%
            Minimum: 1.95 × 0.97 = 1.892 V > 1.85 V ✓
            Maximum: 1.95 × 1.03 = 2.009 V — must verify all VIO_1V8 consumers
            can tolerate 2.0 V maximum. Check GPIO and MIPI datasheets.

  Option D: Use a different supply for VANA (e.g., step it down from VIO_3V3
            through a dedicated LDO). More headroom, but VANA is no longer on
            the same supply domain as the VIO_1V8 digital circuitry.
```

### Multiphase Buck for VCORE — Why It Matters

```
Single-phase 12 V → 0.9 V, 12 A at 500 kHz:
  Duty cycle: D = 0.9/12 = 7.5%
  Inductor current ripple: ΔIL = (12 - 0.9) × 0.075 / (500 kHz × L)
    For L = 300 nH: ΔIL = 11.1 × 0.075 / (500k × 300n) = 5.55 A ripple
  Input RMS ripple current = IL × √(D(1-D)) = 12 × √(0.075 × 0.925) = 3.24 A RMS
  → Large input capacitor required to handle this current without excessive ripple

2-phase interleaved buck (phases 180° apart):
  Each phase runs at 500 kHz with D = 7.5%.
  Input current ripple largely cancels between phases.
  Effective ripple frequency at input = 2 × 500 kHz = 1 MHz.
  Each inductor carries 6 A average (half total load).
  Transient response: two inductors demagnetise in parallel → 2× faster current slew.
  Smaller output capacitance required for same transient performance.
```

---

## Key Takeaways

1. **Always verify LDO headroom at worst-case input and output tolerances.** The VANA case above is a very common system-level error that only shows up in testing at extreme temperature or supply tolerance corners.

2. **Group rails before selecting topologies.** High-current core rails need dedicated converters. Precision rails need LDO post-regulation. Low-current precision rails should be derived from already-clean intermediate rails, not directly from the switching stage.

3. **Sequencing must be specified with timing numbers, not just order.** "VCORE before VCPU_MEM" is incomplete. The requirement must include: what defines "before" (reaching what voltage threshold), the maximum allowed gap, and what the processor's internal POR hold time is.

4. **Multiphase converters are justified for the highest-current rails.** The benefits — reduced ripple, faster transient response, lower per-inductor current — outweigh the added controller complexity above roughly 8-10 A.

5. **Input power budgeting determines the wall adapter rating.** Always add ≥ 20% margin over calculated worst-case input power to allow for efficiency variation, startup transients, and future feature additions.

---

## Interview Notes

**Common follow-up questions:**

- "How would you adjust the power tree if the board needed to run from a 5 V USB-C power delivery input instead of 12 V?" → The 0.9 V VCORE buck now has a 5:1 input-to-output ratio rather than 12:1 — duty cycle rises to 18%, which is more favourable. But the VSYS_5V rail can no longer be generated from a step-down converter from 12 V — it becomes the input. USB-PD negotiation circuitry is needed.

- "How does dynamic voltage scaling (DVS) on VCORE affect your design?" → The processor adjusts VCORE between a minimum (idle, ~0.75 V) and maximum (peak performance, ~1.05 V) by sending a voltage adjust command to the converter via I2C/PMBus. The converter must support this protocol (PMBus, AVS, or VID). Sequencing constraints apply at both ends of the voltage range. The power consumption calculation must use the maximum, not nominal, voltage.

- "What happens if VDDR comes up before VCORE?" → The processor's I/O cells are connected to VDDR rail but the core logic (powered by VCORE) has no supply. Current flows from VDDR through the I/O input protection diodes into an unpowered VCORE rail, potentially damaging the core logic and causing latch-up. The sequencing constraint exists precisely to prevent this.
