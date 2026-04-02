# Component Placement

## Prerequisites
- Understanding of PCB stackup and layer assignments
- Signal integrity fundamentals: return currents, propagation delays
- BGA package construction: ball pitch, ball map, thermal pad
- Power distribution basics: decoupling capacitor placement

---

## Concept Reference

### Why Placement Drives Everything

Component placement is the single most consequential step in PCB layout. Routing is largely determined by placement: a well-placed board often routes itself with few compromises; a poorly placed board cannot be fixed by clever routing. Every signal path, every return current, every thermal gradient, and every manufacturing constraint is a downstream consequence of how components are positioned.

The placement process follows a priority order:

```
Priority 1  Mechanical constraints  -- connectors, mounting holes, chassis cutouts
Priority 2  Thermal constraints     -- heat sources, airflow direction, thermal relief
Priority 3  High-speed signal flow  -- minimise trace length and layer changes
Priority 4  Power delivery          -- bypass caps adjacent to ICs
Priority 5  Low-speed / general I/O -- fill remaining space efficiently
```

### Signal Flow and Functional Blocks

Group components by function. Each functional block occupies a defined region of the board, and signals flow directionally between blocks rather than criss-crossing the board.

```
Board layout schematic (example):

 +--------------+   +--------------+   +-----------+
 | Power Supply |-->| Microcontroller|-->| Peripherals|
 |  (left side) |   | (centre)     |   | (right)   |
 +--------------+   +--------------+   +-----------+
        |                   |
        v                   v
  Bulk caps near        Decoupling
  converter output       caps on each
                         VCC pin
```

Benefits:
- Power rails run in one direction, reducing conducted noise coupling paths
- Traces between related components are short
- Isolation between noisy analogue and digital blocks is easier to maintain

### BGA Breakout Principles

A BGA (Ball Grid Array) package presents the densest routing challenge in PCB layout. The escape routing strategy must be chosen before any other placement decision around the BGA.

**Ball pitch and escape geometry:**

| Ball pitch | Gap between pads | Max trace width for 1-trace escape | Via type required |
|---|---|---|---|
| 1.0 mm | 650 µm | 100 µm (4 mil) | Through-hole via |
| 0.8 mm | 480 µm | 75 µm | Through-hole via (tight) |
| 0.65 mm | 300 µm | 50 µm | Blind via |
| 0.5 mm | 200 µm | Not feasible | Via-in-pad required |
| 0.4 mm | 120 µm | Not feasible | HDI microvia-in-pad |

**Dog-bone vs via-in-pad fanout:**

Dog-bone fanout (standard): The BGA pad connects via a short trace stub to a via pad placed between the pads. This works for 1.0 mm and 0.8 mm pitch.

```
Dog-bone (1.0 mm pitch, outer two rows):

  [o]   [o]   [o]   BGA pads
   |     |     |
  [via] [via] [via]  vias placed between pads
```

Via-in-pad (0.5 mm pitch, inner balls): The via drill sits directly under the BGA pad. The via must be filled and copper-capped per IPC-4761 Type VII to prevent solder wicking.

```
Via-in-pad (0.5 mm pitch):

  [o/v] [o/v] [o/v]  pad and via are coincident
```

**Layer assignment for BGA fanout:**

For a large BGA (>400 balls), plan escape routing across multiple layers:
- Layer 1 (top): Outer ring escape — signals that can be routed on L1 before encountering adjacent pad obstructions
- Layer 2 (first inner): Signals from second ring that dog-bone to a via and continue on L2
- Layer 3 and below: Inner ball signals via blind or through-hole vias

### Placement Priority for Complex Boards

**Step 1: Fixed placements**

Start with components that cannot be moved:
- Board-mounted connectors (position determined by chassis/system)
- Crystals and oscillators (must be close to the IC they serve, with short traces)
- Mounting holes and PCB outline

**Step 2: Critical ICs**

Place the board's primary ICs — FPGAs, processors, high-speed transceivers. The orientation of these ICs determines the routing topology for the entire board. For a BGA device, consider:
- Which side of the BGA do the high-speed signals exit? Orient so those balls face the interface connector.
- Where does the FPGA's DDR memory interface sit? Place DDR devices directly adjacent to the FPGA's DDR bank balls.
- Where is the power supply? Keep VCC_CORE and VCC_IO power domains close to the ball rows they serve.

**Step 3: Memory and high-speed peripherals**

DDR, LPDDR, SDRAM, and Flash devices must be placed within the length-matching budget for the interface. DDR4-3200 has a maximum fly-by trace length budget; placing memory more than 30–40 mm from the processor makes length matching difficult without excessive serpentine routing.

**Step 4: Power supply components**

Switching regulators, inductors, and bulk capacitors should be placed:
- Inductors close to the switch node (minimise the loop area of the inductor, MOSFET, and input capacitor)
- Output capacitors at the load end of the power trace, not the source end
- Input capacitors on the hot side of the switch, between VIN and the converter, minimising lead inductance

**Step 5: Passive components**

Decoupling capacitors: Place each bypass capacitor as close as possible to the VCC pin it decouples. For BGA packages, place decoupling on the same side as the BGA, directly adjacent to the package body. For fine-pitch BGAs, use via-in-pad to place capacitors between the BGA pads if necessary.

```
BGA decoupling placement rule:

  GOOD:            BAD:
  [cap][BGA][cap]  [BGA] ---- 5 mm ---- [cap]
  (caps adjacent)  (distant caps have high inductance path)
```

---

## Tier 1 -- Fundamentals

### Question F1
**Why is component placement considered more important than routing in PCB design? What problems cannot be fixed by re-routing if placement is wrong?**

**Answer:**

Routing is constrained by placement. Once components are placed, the trace lengths, layer assignments, and proximity relationships between nets are largely fixed. There are problems that routing cannot compensate for:

1. **Return current paths:** If a noisy switching regulator is placed next to a sensitive ADC input, the return currents from the regulator will flow through the ground plane beneath both components. No amount of routing changes the ground plane topology under fixed components. The only fix is to move the regulator.

2. **Trace length budgets:** DDR4 fly-by topology has a total trace length budget of ~150 mm. If the memory is placed 80 mm from the processor, the traces to far-side balls are already over budget before any routing begins.

3. **BGA escape routing:** If a BGA is oriented so that its high-speed SerDes balls face away from the SerDes connector, every signal must cross the body of the BGA and route around it to reach the connector — adding length and layer changes.

4. **Thermal management:** If a high-power component is placed in a region of poor airflow or adjacent to another heat source, the thermal design cannot be fixed by rerouting.

5. **Decoupling effectiveness:** A bypass capacitor placed 20 mm from its VCC pin has a series inductance of ~20 nH (approximately 1 nH/mm for a via and trace combination). At 100 MHz, this is 12.6 Ω, rendering the capacitor nearly useless. Routing the capacitor closer to the pin is not possible once the IC is fixed — only the capacitor position can be moved.

---

### Question F2
**Explain the purpose of grouping components into functional blocks. What problems arise when functional blocks are mixed together on a board?**

**Answer:**

Grouping components into functional blocks keeps the signals, power rails, and return currents associated with each function in a defined area. Each block has its own supply decoupling, its own local return plane, and short interconnects between components that belong together.

**Problems when blocks are mixed:**

1. **Ground current coupling:** A switching power supply passes large pulsed currents through the ground plane. If an ADC is placed nearby, those return currents flow through the same ground plane area as the ADC's reference ground, injecting noise directly into the analogue reference.

2. **Long shared traces:** Signals from mixed blocks must route across the board, passing near unrelated circuits. These long traces act as antennas, coupling noise from adjacent signal layers.

3. **Decoupling conflicts:** A single power rail that supplies both a high-speed digital IC and a sensitive analogue IC will have switching noise from the digital IC conducted onto the analogue supply, even with capacitor bypassing, because both devices share the same physical copper trace segment.

4. **Difficult isolation:** Split ground planes or ferrite bead isolation between analogue and digital sections are only practical when the analogue and digital blocks occupy separate physical board regions. Mixed placement makes this impossible.

**Example:** On a mixed-signal board with an MCU, ADC, and buck converter:
- Place the buck converter in one corner with its switching loop area minimised.
- Place the MCU in the centre with full digital supply decoupling.
- Place the ADC on the opposite side from the buck converter, with a separate low-noise supply derived from an LDO, not directly from the buck output.

---

### Question F3
**What is a "critical placement" decision and why must it be resolved before routing begins?**

**Answer:**

A critical placement decision is one where the physical relationship between two or more components directly determines whether a design specification (signal integrity, EMC, thermal, or functionality) can be met.

Examples:

**Crystal oscillator placement:**
A crystal must be placed within a few millimetres of its driving IC load capacitors. Long crystal traces pick up noise and can shift the oscillation frequency. The crystal must also be positioned away from the edge of the board and away from other oscillating circuits (avoid mutual inductance coupling between crystal traces).

If the crystal is placed remotely and the board is fully routed, moving it requires ripping up potentially hundreds of traces that route over or around it. This is the kind of problem that forces a full re-spin on production boards.

**Differential pair connector alignment:**
If a PCIe connector is placed on one side of the board and the corresponding FPGA transceiver balls are on the opposite side, the differential pair route must cross the entire board. Length matching serpentines will consume large board areas, and the pairs will be exposed to crosstalk from other signals along the long route. Rotating the FPGA or choosing a different device package orientation could have avoided this.

**Decoupling capacitor placement (FPGA core supply):**
FPGA core power pins are distributed across the BGA footprint. If the designer places all decoupling capacitors on one edge of the BGA, the pins on the opposite side of the BGA have a long, inductive supply path. Decoupling must be spread around the BGA perimeter or placed in via-in-pad positions between BGA balls. This decision must be made before routing because it affects the BGA footprint design and via assignment.

---

## Tier 2 -- Intermediate

### Question I1
**You are placing a 784-ball FPGA in a 0.8 mm pitch BGA package (28 × 28 mm). The FPGA has four DDR4 memory interfaces on the south bank and PCIe Gen 4 x8 transceivers on the west bank. Describe your complete placement strategy for the FPGA and surrounding components.**

**Answer:**

**Step 1: Orient the FPGA**

Rotate the FPGA so that:
- The south bank (DDR4 balls) faces the board edge closest to the memory slots or the direction where DDR4 routing is shortest.
- The west bank (PCIe transceivers) faces the PCIe edge connector side.

This ensures DDR4 and PCIe routes travel in opposite directions from the FPGA and do not cross each other.

**Step 2: DDR4 placement**

Four DDR4 interfaces typically require 2 or 4 DRAM devices (x16 or x8 configuration). Place these:
- Directly below the south bank, within 20–30 mm to stay within the DDR4 fly-by length budget (~150 mm total for the command/address T-topology tree).
- All four DRAM devices in a row, oriented so their address/data balls face the FPGA.
- Place the DDR4 VTT termination resistors at the far end of the fly-by chain, not at the FPGA end.

**Step 3: PCIe connector**

Place the PCIe x8 edge connector on the west edge of the board. The FPGA transceiver balls will face this connector. Keep the differential pair route length as short as possible: target <50 mm per pair, with ≤10 mm intra-pair skew.

**Step 4: Power supplies**

The FPGA has multiple power domains:
- VCC_CORE (e.g., 0.95 V, ~5–10 A): Place a dedicated synchronous buck converter as close as possible to the core supply balls on the BGA. Place ceramic output capacitors (10 µF × 8–16) distributed around the BGA perimeter.
- VCC_IO (multiple banks, e.g., 1.8 V, 2.5 V): Place small LDOs or bucks for each bank, located adjacent to their respective bank's balls.
- Decoupling caps: Use via-in-pad placement or standard dog-bone escapes between BGA balls, targeting 100 nF ceramic caps within 2 mm of each power pin row.

**Step 5: Configuration and debug**

Place the configuration flash (SPI or BPI) adjacent to the FPGA's configuration bank (typically a dedicated corner bank). Place the JTAG header close to the JTAG pins. These low-speed signals do not constrain placement tightly but should not occupy space that blocks DDR or PCIe routing channels.

**Step 6: BGA escape routing pre-plan**

For a 0.8 mm pitch BGA, plan escape routing before finalising placement:
- Outer two rows: dog-bone via escape on Layer 1
- Next two rows: dog-bone via to Layer 3 (inner signal)
- Inner balls: through-hole vias to Layer 5 or Layer 7

If blind vias (L1–L2) are available, the first two inner rows can also escape to Layer 2, freeing up routing channel capacity on Layer 1.

---

### Question I2
**What is placement priority for decoupling capacitors, and how does the optimal placement differ between a small MCU and a large BGA FPGA?**

**Answer:**

**Decoupling capacitor placement principle:**

The goal of a decoupling capacitor is to supply instantaneous charge to an IC's power pins when the IC draws a current spike. The effectiveness is determined by the series inductance of the current loop from the capacitor's negative plate to the IC's GND pin, and from the IC's VCC pin back to the capacitor's positive plate.

**Series inductance in a PCB path:**
```
L_total = L_trace + L_via + L_pad ≈ 1 nH/mm trace + 1 nH per via

A capacitor 5 mm from the VCC pin (trace + via path): ~6-7 nH total
At 200 MHz: X_L = 2π × 200e6 × 7e-9 = 8.8 Ω -- nearly useless
```

The capacitor must therefore be as close as physically possible to the VCC pin.

**Small MCU (e.g., STM32 in QFN48):**

The package is ~7 × 7 mm with exposed VCC pins around the perimeter. Place one 100 nF C0G or X7R ceramic cap (0402 or 0603) on each VCC pin:
- Position each cap with its VCC pad directly adjacent to the VCC pin pad -- no trace between them, or a trace of ≤0.5 mm.
- The GND connection goes to the nearest GND via, which connects to the GND plane.
- One 10 µF bulk cap per supply domain (e.g., 3.3V and 1.2V core) placed within 5 mm.

**Large BGA FPGA (e.g., 784-ball 0.8 mm pitch):**

The VCC pins are distributed across the interior of the BGA footprint. Standard perimeter placement is inadequate for inner VCC pins because the trace must travel through the BGA escape routing area before reaching a capacitor.

Options:
1. **Via-in-pad decoupling**: Place small ceramic capacitors (100 nF, 0201) between BGA ball positions on Layer 1, with their supply via going to the power plane on Layer 2 and GND via to Layer 2. This achieves <1 mm electrical distance from VCC ball to capacitor plate.

2. **Underside placement**: If the BGA is on the top side, place capacitors on the bottom side directly beneath the BGA. The via path (L1 BGA pad → via → L4 inner plane → via → L8 bottom capacitor) is longer, but for bulk capacitance (10 µF) this is acceptable.

3. **Power plane proximity**: The power plane on Layer 2 (or 3) directly under the BGA provides distributed capacitance. A 100 µm thick prepreg between GND on Layer 2 and PWR on Layer 3 gives approximately 1 nF per cm², which across the BGA footprint (28 × 28 mm = 7.84 cm²) gives ~7.84 nF of intrinsic distributed capacitance at very low ESL.

**Key difference:** For a small MCU, perimeter placement is sufficient. For a large BGA, a combination of via-in-pad decoupling, underside capacitors, and tight plane pair capacitance is needed to keep impedance below target at frequencies >100 MHz.

---

### Question I3
**Describe the "keep-away" zones required around the following component types and explain the reason for each: (a) crystal oscillator, (b) RF antenna or RF transmission line, (c) high-current inductor, (d) electrolytic capacitor.**

**Answer:**

**(a) Crystal oscillator**

Keep-away requirements:
- No signal traces routed over or under the crystal body (L1 copper flood void, inner layer anti-pad area beneath the crystal footprint)
- Minimum 3 mm clearance from clock traces to any other switching signals
- Ground fill under the crystal to shield it from the board side
- No vias or copper in the area between the crystal and its load capacitors

Reason: The crystal oscillates mechanically. The crystal pins and traces are part of a resonant tank circuit with very high impedance. Any capacitive coupling from adjacent traces disturbs the tank impedance and can shift the oscillation frequency or prevent oscillation altogether. EMI radiated from the crystal traces can couple into sensitive analogue inputs.

**(b) RF antenna or transmission line**

Keep-away requirements:
- Clear ground plane below the transmission line (or specific coplanar waveguide geometry with ground strips on either side)
- No copper fills, pours, or traces within the calculated antenna exclusion zone (typically 3–5 mm for a PCB trace antenna)
- No components between the antenna and the board edge

Reason: A PCB trace antenna is deliberately designed to radiate. Any copper within its near field loads the antenna, shifting its resonant frequency and reducing radiation efficiency. The antenna ground plane must be exactly as specified in the antenna reference design — deviations change the input impedance and cause impedance mismatch at the feed point.

**(c) High-current power inductor**

Keep-away requirements:
- Sensitive analogue traces or components: minimum 5–10 mm from the inductor body
- No plane copper directly over the top of the inductor (some inductors have significant fringing flux from the top surface)
- Current-carrying traces to and from the inductor: as short as possible, on the same layer as the inductor to minimise loop area

Reason: A power inductor stores energy in its magnetic field. Unshielded (open-core) inductors radiate this field into adjacent components and traces. A 1 A, 10 µH inductor switching at 500 kHz generates a substantial AC field that can couple into nearby traces as common-mode or differential noise. Shielded inductors (ferrite core surrounds the winding) are preferred for mixed-signal boards; even so, a 5 mm clearance from the top surface is prudent.

**(d) Electrolytic capacitor**

Keep-away requirements:
- Minimum 2 mm from the top of the capacitor body to any other component (to allow the pressure vent to operate in case of failure)
- Polarity marking must be visible for inspection; do not allow silkscreen from adjacent components to obscure the polarity band
- Keep away from heat sources (inductor body, heat sink, power resistor) — elevated temperature significantly reduces electrolytic capacitor lifetime

Reason: Electrolytic capacitors fail by electrolyte evaporation at elevated temperatures. Every 10°C rise above rated temperature approximately halves the expected lifetime (Arrhenius relationship). The top vent must be clear — if the capacitor bulges or ruptures, the vent prevents catastrophic explosion; a clamped vent converts a safe failure into a destructive one.

---

## Tier 3 -- Advanced

### Question A1
**You are placing an 1156-ball FPGA (35 × 35 mm, 1.0 mm pitch) on a 10-layer board. The FPGA has the following functional banks: north bank: DDR5 (48 data bits + address/cmd), east bank: 100G Ethernet (4 × 25G SerDes), south bank: PCIe Gen 5 x16, west bank: general-purpose I/O. Walk through the complete placement decision process, including BGA escape routing assignment and decoupling strategy.**

**Answer:**

**Step 1: Board orientation and connector placement**

PCIe Gen 5 x16 requires a standard PCIe edge connector. This connector is large and must be at a board edge. Place it on the south edge. Orient the FPGA so that the PCIe bank (south) faces the PCIe connector.

100G Ethernet connects to a QSFP28 or QSFP-DD cage or an SFP cage array. These are tall components that typically sit at a board edge. Place Ethernet cages on the east edge. Orient the east bank toward the Ethernet cages.

DDR5 (north bank): Place DDR5 DRAM devices directly above the FPGA. The DDR5 bus has tight timing requirements; minimise trace lengths to ~40–60 mm.

**Step 2: Escape routing plan for 1.0 mm pitch BGA**

At 1.0 mm pitch, the gap between pads is ~650 µm. This allows one trace escape between pads with a standard 4/4 mil process.

```
Layer assignment by BGA zone:

Outer ring (rows 1-2 from edge):  Layer 1 dog-bone escape
Next ring (rows 3-4):             Layer 1 trace -> via to Layer 3
Next ring (rows 5-6):             Layer 1 trace -> via to Layer 5
Inner zone (rows 7+):             Via-in-pad or short trace to via, Layer 7 or Layer 9

SerDes balls (PCIe, Ethernet):    Blind vias to Layer 3 (inner stripline)
                                   -> direct connections, no stubs
Power balls (VCC_CORE, VCC_INT):  Short dog-bone to via, power plane Layer 4 or 6
GND balls:                        Short dog-bone to GND via, GND plane Layer 2
```

**Step 3: DDR5 placement and routing pre-plan**

DDR5 uses a point-to-point or short fly-by topology. For 48 data bits:
- Six ×8 DRAM devices in a row above the FPGA
- All address and command signals use a fly-by daisy-chain topology through all six devices
- Each DRAM must be within the trace length budget set by the FPGA's DDR5 controller PHY specification

Place the DRAMs at 25–35 mm from the FPGA's DDR5 bank edge. All DRAM VDD decoupling (typical: 8–12 × 100 nF per device) placed between the DRAM and the FPGA, with bulk 47 µF caps at the far end.

**Step 4: SerDes placement and keepout**

PCIe Gen 5 and 100G Ethernet SerDes lanes must follow strict routing rules:
- All 16 PCIe differential pairs must maintain 85 Ω ±10% differential impedance
- Lane-to-lane skew must be within ±20 ps within each group (separate tolerance from AC coupling caps)
- AC coupling capacitors (typically 100 nF C0G 0402 per lane) placed within 5 mm of the FPGA SerDes output, before the connector

Place the PCIe AC coupling capacitors in a row between the FPGA south edge and the PCIe connector. This is a mandatory placement decision that must be made before routing.

**Step 5: Power delivery placement**

For a large FPGA, core power requirements are significant:
- VCC_CORE (~0.8 V, up to 30–50 A depending on FPGA configuration): requires a multiphase synchronous buck converter. Place the controller IC and all phase inductors within 20 mm of the FPGA's VCC_CORE ball cluster. Each phase current loop (switch MOSFET → inductor → output cap) must be as compact as possible.
- VCC_INT, VCC_AUX, VCCO_n (bank supplies): dedicated LDOs or bucks, each placed adjacent to their respective bank region.

Decoupling strategy:
- 100 nF × 32 (minimum) within 2 mm of VCC_CORE ball rows -- via-in-pad between ball pitches preferred
- 10 µF × 16 placed around FPGA perimeter for mid-frequency decoupling
- 47–100 µF × 4 placed at the output of each VRM, 10–20 mm from the FPGA, for low-frequency bulk reserve

**Step 6: Thermal planning**

A large FPGA running at full utilisation may dissipate 30–80 W. Confirm:
- The component placement allows the thermal solution (heatsink, heat pipe, or forced air) to make contact with the FPGA package lid.
- No tall components (vertical connectors, electrolytic caps) are taller than the heatsink retention mechanism height limit.
- Power inductors for VCC_CORE are placed such that they do not thermally load the FPGA -- keep them at least 10 mm from the FPGA body or use shielded types.

---

### Question A2
**During board review, a colleague points out that all the bypass capacitors for a BGA FPGA are placed on the board's bottom side, directly beneath the BGA footprint. They claim this is better than placing caps on the top side around the BGA perimeter. Evaluate this claim: under what conditions is it better, worse, or equivalent?**

**Answer:**

**The argument for bottom-side placement:**

Bottom-side placement beneath the BGA can be better than top-side perimeter placement when the following conditions hold:

1. **The via path is short**: If the BGA is on a 6-layer board and the power planes are on Layer 3 and Layer 4, a via from Layer 1 (BGA pad) to Layer 6 (bottom capacitor) passes through the power plane at Layer 3, connecting to the capacitor via the same plane. The electrical path from capacitor pad to power plane can be as short as one via (~1 nH) rather than a trace + two vias from a perimeter cap (~2–3 nH).

2. **VCC_CORE balls are in the interior**: For large BGAs, the core supply balls are often in the geometric centre of the package. A perimeter cap on Layer 1 must route through the escape routing congestion zone to reach those inner balls. A bottom-side cap beneath the inner balls has a direct vertical path.

3. **Assembly is double-sided**: If the board is designed for double-sided reflow, bottom-side caps under the BGA are assembled without additional process cost.

**Conditions where bottom-side placement is worse:**

1. **Via count and aspect ratio**: Routing a via from Layer 1 to Layer 6 (or 8) consumes routing channel on every intermediate signal layer the via crosses. In a dense routing area directly under the BGA, additional through-hole vias for capacitor connections compete for routing channels with signal vias. Blind vias could mitigate this but add cost.

2. **Long via path inductance**: On a thick board (2.4 mm, 10 layers), a via from top to bottom is approximately 2–3 nH. A top-side cap with a 0.5 mm trace and one via to a nearby plane might achieve 1–1.5 nH total loop inductance. In this case the bottom-side cap has higher inductance.

3. **Inspection and rework**: Bottom-side BGAs and their associated capacitors are inaccessible after the board is assembled. A failed capacitor under a large BGA is a board-level failure. Top-side perimeter caps are at least theoretically reworkable.

**When they are roughly equivalent:**

On a 4-layer board (1.6 mm thick), the via from Layer 1 to Layer 4 is ~1.6 mm long, giving approximately 1.6 nH via inductance. A perimeter cap 1.5 mm away also has approximately 1.5–2 nH total path inductance. The difference is small.

**Conclusion:**

The colleague's claim is partially correct. Bottom-side placement beneath the BGA is better when the BGA interior balls need decoupling and the via path to the power plane is short. Top-side perimeter placement is better when traces from perimeter caps to the power pins are short (outer ring balls) and when board thickness makes via inductance significant. The optimal strategy for a large FPGA combines both: via-in-pad or between-ball capacitors for inner critical power pins, and perimeter caps on the same side for the outer banks.
