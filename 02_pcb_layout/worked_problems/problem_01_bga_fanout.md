# Problem 01: BGA Fanout for a Large FPGA

## Problem Statement

You are laying out a Xilinx Kintex-7 FPGA in a 676-ball FBGA package (FF676), 1.0 mm ball pitch, 26 × 26 ball matrix. The package measures 27 × 27 mm. Your target board is a 6-layer stackup with a finished thickness of 1.6 mm:

| Layer | Use |
|---|---|
| 1 (top) | Component placement, signal escape |
| 2 | Ground plane |
| 3 | Signal routing |
| 4 | Power plane |
| 5 | Signal routing |
| 6 (bottom) | Signal + ground pours |

**Part A:** Calculate whether standard through-hole via escape routing (one via between adjacent pads on the same pitch) is geometrically feasible for all rows of this FPGA. State the design rule requirements at the innermost rows.

**Part B:** Determine how many routing layers are accessible for each escape channel using dog-bone via escape routing from the outer rows. How does the number of escape channels change as you move inward, and what strategy do you use for the interior balls?

**Part C:** The FPGA has 32 high-speed SERDES differential pairs (lanes) in a dedicated ball region on the left side of the package. These pairs require 100 Ω differential impedance, matched length within ±0.1 mm of each other, and must be routed to an edge-mounted SFP+ connector 45 mm away. Describe the routing strategy for these lanes from the BGA to the connector.

**Part D:** The FPGA core supply (VCCINT = 1.0 V) is supplied by 24 balls arranged in a 4 × 6 cluster in the centre-left region of the package. Your PCB stackup has a dedicated VCCINT power plane on layer 4. Describe how you establish the VCCINT supply to these balls, including via routing and decoupling capacitor placement.

---

## Solution Approach

Work through each part in sequence. BGA fanout is a constraint-satisfaction problem: geometry, layer availability, signal integrity, and power delivery all compete for the same PCB real estate.

### Part A — Through-Hole Via Feasibility at 1.0 mm Pitch

At 1.0 mm ball pitch, the centre-to-centre distance between adjacent balls is 1000 µm. The BGA pad diameter for 1.0 mm pitch (IPC-7351B, SMD pads) is typically 0.5 mm, leaving a gap of:

```
Gap = 1.0 mm - 0.5 mm = 0.5 mm between adjacent pad edges
```

For a dog-bone via escape (trace exits pad, then via is placed between adjacent pads), the available space must accommodate:

```
Geometry check for via between pads:

   [pad 0.5 mm dia] ---trace--- [via pad] ---trace--- [pad 0.5 mm dia]
   |<----------- 1.0 mm pitch centre-to-centre ------------->|

   Via pad size (standard): drill 0.3 mm, annular ring 0.125 mm → pad dia 0.55 mm
   Trace width: 0.1 mm minimum (4 mil, standard fab)
   Clearance: 0.1 mm minimum

   Space required from edge of BGA pad to centre of via:
     Clearance (0.1) + half via pad (0.275) = 0.375 mm

   Available: 0.5 mm / 2 = 0.25 mm per side of gap

   0.375 mm required > 0.25 mm available — does not fit with standard vias.
```

**Result: Standard 0.3 mm drill through-hole vias do NOT fit between adjacent 1.0 mm pitch balls at standard fab rules.**

However, with a reduced via pad size — using a via annular ring of 0.075 mm (advanced fab, feasible at most HDI houses) — the via pad diameter drops to 0.45 mm:

```
Space required: 0.1 (clearance) + 0.225 (half of 0.45 mm pad) = 0.325 mm
Available: 0.25 mm — still too tight.
```

In practice, at 1.0 mm pitch the standard escape strategy is:

- **Outer two rows (rows A-B and Z-Y from edge):** Space exists between the pads and board edge for dog-bone vias with the via placed outside the BGA footprint (the "dog-bone" extends outward). This works for the outermost ~2 rows.
- **Rows 3-5 inward:** One via can be placed between each row and the next, using the via between two rows on the same column line. Only one signal per column can escape per routing layer without via conflicts.
- **Interior rows (beyond row 5):** Requires sequential layer fanout — escaping on successive layers — which requires that each deeper escape uses a layer reserved exclusively for inner-row routing.

### Part B — Accessible Routing Layers per Escape Channel

The number of signals that can escape to a given routing layer decreases as you move inward, because outer-row traces physically block inner-row escape paths to that layer. This is the "layer budget" problem in BGA routing.

```
Outer row (row A, outermost):
  Via exits BGA footprint into open board area → connects to any layer (L1-L6).
  Effective routing layers available: L1, L3, L5 (signal layers, minus plane layers).

Row B (second from outside):
  Dog-bone via placed beside the row-A escape trace — fits in remaining gap.
  One trace per column already consumed on the inner layer from row A.
  Routing layers available: L1, L3, L5 (minus row-A channels already used).

Row C (third):
  Two traces per column already exiting from A and B rows.
  Routing channels become tight — one signal layer may be full at this column.

Row D and inward:
  Without via-in-pad (VIP) or blind vias, escape requires routing under existing
  via pads, which is generally not permitted (via pad keepout clearance).
  Interior rows require either:
  (a) Via-in-pad — via directly under the BGA ball pad.
  (b) Blind vias — from layer 1 down to layer 2 only, freeing layers 3-6 for routing.
  (c) Microvia HDI build-up — enables any-layer routing with 0.1 mm vias.
```

**Strategy for interior balls:**

For a standard 6-layer board at 1.0 mm pitch, the accepted practice is:

1. Route outer 3 rings of balls using dog-bone escape, fanning out to the perimeter of the BGA footprint.
2. For interior balls (rows D through W, approximately), use via-in-pad with through-hole vias filled and capped — this allows each ball to have its own via without requiring trace escape space.
3. Route interior ball vias to dedicated inner routing channels on L3 and L5, which are not blocked by the outer-row dog-bone vias.
4. Group signals by function: SERDES lanes on one side, configuration pins on another, general I/O on the remaining sides — this reduces route congestion.

### Part C — SERDES Differential Pair Routing Strategy

**Routing constraints for SERDES:**

```
Impedance: 100 Ω differential (50 Ω single-ended odd-mode)
Length matching: ±0.1 mm intra-pair (between P and N of same pair)
                 ±2 mm inter-pair (between all 32 lanes) — typical for XAUI/SFP+
Layer: preferably stripline (inner layer) for EMI shielding and impedance stability
Reference plane: continuous ground plane on both sides of the routing layer
```

**Step 1 — BGA escape for SERDES balls:**

The SERDES transmitter (TX) and receiver (RX) balls are typically on the left edge of the FPGA package in groups. At 1.0 mm pitch, the differential pairs occupy adjacent columns:

```
SERDES lane 0:  TX_P ball at (col 2, row 3), TX_N ball at (col 2, row 4)
                RX_P ball at (col 3, row 3), RX_N ball at (col 3, row 4)
...and so on for all 32 lanes.
```

Escape the SERDES balls using via-in-pad directly from the BGA pad. Route via to layer 3 (stripline, between L2 GND and L4 power — the power plane provides AC ground). Layer 3 provides a controlled 50 Ω single-ended / 100 Ω differential environment.

**Step 2 — Intra-pair length matching in the BGA escape:**

The P and N balls of each pair are adjacent (one ball apart = 1 mm offset). The via-in-pad drills for P and N are at slightly different x-y positions. After exiting the BGA region on layer 3, the P and N traces will have slightly different lengths due to the offset.

Add a serpentine tuning section (accordion tune) on the shorter trace of each pair immediately after clearing the BGA keepout zone. Tune to within ±0.1 mm before any connector routing.

**Step 3 — Routing to SFP+ connector:**

```
Route all 32 differential pairs in parallel on layer 3 (stripline).
Maintain 100 Ω differential geometry: W = 0.127 mm (5 mil), S = 0.127 mm gap
(calculated for the specific stackup — use IPC-2141A or field solver).

Separation between adjacent lanes: ≥ 5× trace width (to avoid crosstalk coupling).
  5 × 0.127 mm = 0.635 mm minimum between pairs.

Group lanes in bundles of 4 (one SERDES quad per group) with 0.3 mm guard spacing
between groups.

Avoid routing parallel to high-speed clock traces on adjacent layers — use the
plane layers (L2, L4) to provide isolation.
```

**Step 4 — SFP+ connector interface:**

SFP+ connectors have through-hole signal pins at 0.8 mm pitch. Use via-in-pad or dog-bone escape at the connector footprint. Route the final 5 mm to the connector on L1 (microstrip) if via-to-connector distance is short — this avoids an additional via transition and its stub.

If the connector uses a through-hole via, back-drill the via stub to reduce stub length (SERDES is typically 6.25 Gb/s for SFP+; stub resonance check is required for the via barrel length in a 1.6 mm board — at ~30 GHz resonance this is not an issue, but confirm calculation).

**Inter-pair length matching:**

After routing all 32 lanes, identify the longest lane and add serpentine tuning to all shorter lanes so all arrive within ±2 mm at the SFP+ connector.

### Part D — VCCINT Power Distribution

**Via routing for 24 VCCINT balls:**

The VCCINT balls (24 balls, 4 × 6 cluster) occupy an interior region of the BGA. Via-in-pad is required. Each ball connects via a copper-filled via directly to the VCCINT power plane on layer 4.

```
Via specification:
  Drill: 0.3 mm (or 0.25 mm for better density)
  Fill: copper-filled via-in-pad (conductive fill for low resistance)
  Layer connection: ball pad (L1) → L4 power plane (VCCINT)
  Note: vias must NOT connect to L2 GND plane — add anti-pad on L2.
  Anti-pad diameter: via pad + 0.2 mm = 0.75 mm → clear GND plane on L2.
```

**Decoupling capacitor placement:**

Per the Kintex-7 PCB design guidelines (UG539), VCCINT requires:
- One 100 nF 0402 X7R per VCCINT ball (24 caps)
- One 10 µF 0402 X7R per 4 VCCINT balls (6 caps)
- One 100 µF polymer tantalum or polymer aluminium bulk cap

```
Placement rules:
  100 nF caps: Place on layer-6 (bottom) directly under the BGA, via-adjacent
               to each VCCINT via. Each cap: VCC via pad → cap → adjacent GND via.
               Bottom-side placement minimises via-to-cap trace length.

  10 µF caps:  Place on layer-1 top side, within 3 mm of the BGA edge near
               the VCCINT ball cluster. Use 0402 MLCCs on a dedicated VCCINT trace.

  100 µF bulk: Place within 10 mm of the VCCINT ball cluster, on the VCCINT plane
               connected by a short via to L4. Polymer cap, ESR ~20 mΩ.
```

**Anti-pad and plane integrity:**

Route VCCINT on L4 as a poured copper region dedicated to VCCINT. Adjacent power domains (VCCO, VCCAUX) on L4 are separated by plane splits. Ensure:

- Plane splits do not cross under any high-speed signal via that uses L4 as a reference — this breaks the return current path and causes EMI and SI issues.
- Split gaps are perpendicular to signal routing wherever possible.
- Decoupling caps bridge each plane split where crossing is unavoidable.

---

## Key Takeaways

- At 1.0 mm BGA pitch, standard through-hole via dog-bone escape works for the outer 2-3 rows; interior balls require via-in-pad with filled-and-capped vias
- Layer budget at the BGA perimeter limits how many signals can escape on each routing layer; plan the layer assignment before routing to avoid dead ends
- SERDES differential pairs require: (a) correct differential impedance — designed into the stackup and trace geometry; (b) intra-pair length matching within ±0.1 mm — enforced by serpentine tuning at the BGA exit; (c) continuous reference planes — verified by checking for plane splits under the route
- Power via-in-pad for VCCINT balls uses copper-filled vias to connect the ball directly to the power plane — anti-pads on intermediate plane layers are essential to avoid unintentional short circuits
- Decoupling placement under a BGA (bottom-side caps) minimises the via-to-cap inductance — critical for VCCINT at 200+ MHz FPGA operation

---

## Interview Notes

BGA fanout questions test spatial reasoning and knowledge of PCB process constraints simultaneously. Interviewers at FPGA-heavy companies (Xilinx/AMD, Intel, Lattice partner boards) will probe:

- Whether you understand *why* inner rows need via-in-pad (the geometric constraint — not just "it's the rule")
- Whether you know anti-pads are required when a power via passes through a GND plane on an intermediate layer
- Whether you can describe the length-matching procedure and where serpentine tuning sections go
- Whether you understand that VCCINT capacitors placed on the bottom under the BGA are more effective than top-side caps placed 15 mm away from the ball field

A strong answer for a senior role includes quantitative estimates (how many caps, what grid pitch for vias, what trace width for impedance) not just qualitative statements.
