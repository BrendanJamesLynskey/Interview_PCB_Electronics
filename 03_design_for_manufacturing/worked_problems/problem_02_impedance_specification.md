# Problem 02: Impedance Specification

## Problem Statement

You are designing a 6-layer PCB that carries the following interfaces:

- PCIe Gen 3 x4 (differential pairs, 100 Ω differential / 50 Ω single-ended)
- USB 3.2 Gen 1 (differential pairs, 90 Ω differential)
- A single-ended SPI bus at 3.3 V logic (no impedance control required)

The stackup is:

| Layer | Use | Copper | Dielectric |
|---|---|---|---|
| 1 (top) | Signal + power | 1 oz | 0.1 mm prepreg (Er = 4.2) to L2 |
| 2 | Ground plane | 1 oz | 0.36 mm core (Er = 4.5) to L3 |
| 3 | Signal | 1 oz | 0.36 mm core (Er = 4.5) to L4 |
| 4 | Power plane | 1 oz | 0.1 mm prepreg (Er = 4.2) to L5 |
| 5 | Signal | 1 oz | 0.1 mm prepreg (Er = 4.2) to L6 |
| 6 (bottom) | Signal + ground | 1 oz | — |

**Part A:** Calculate the trace width required on Layer 1 to achieve 50 Ω single-ended impedance (microstrip). Use the IPC-2141A approximation formula.

**Part B:** Calculate the trace width and gap required on Layer 1 for a 100 Ω differential impedance pair (edge-coupled microstrip). Confirm whether the calculated geometry satisfies fabrication minimums for a standard fab (4 mil trace / 4 mil space).

**Part C:** Calculate the trace width required on Layer 3 to achieve 50 Ω single-ended impedance (stripline, symmetrically placed between L2 and L4).

**Part D:** Write the controlled impedance specification block that must appear in the PCB fabrication notes for this board.

---

## Solution Approach

Work through the transmission line impedance formulae systematically. The IPC-2141A (microstrip) and IPC-2141A (stripline) approximations are accurate to within ±5% for standard PCB geometries and are the industry-standard calculation method referenced by most fab houses.

### Part A — Layer 1 Microstrip, 50 Ω Single-Ended

The IPC-2141A microstrip impedance formula:

```
Z0 = (87 / sqrt(Er_eff + 1.41)) * ln(5.98 * H / (0.8 * W + T))

Where:
  Z0   = characteristic impedance (Ω)
  Er   = relative permittivity of dielectric (use 4.2 for prepreg to L2)
  H    = dielectric height from trace to reference plane (m or consistent units)
  W    = trace width
  T    = trace thickness (copper)
  Er_eff = Er (simplified; full formula uses Er_eff slightly less than Er for outer layers)
```

Given values:
- H = 0.1 mm (prepreg to L2 ground plane)
- T = 35 µm = 0.035 mm (1 oz copper)
- Er = 4.2
- Target Z0 = 50 Ω

Rearranging for W (iterative, since W appears inside ln):

**Starting estimate** using W ≈ 2H = 0.2 mm:

```
Z0 = (87 / sqrt(4.2 + 1.41)) * ln(5.98 * 0.1 / (0.8 * 0.2 + 0.035))
   = (87 / sqrt(5.61)) * ln(0.598 / 0.195)
   = (87 / 2.368) * ln(3.067)
   = 36.7 * 1.121
   = 41.2 Ω   (too low — need narrower trace)
```

**Second estimate** W = 0.15 mm:

```
Z0 = (87 / 2.368) * ln(5.98 * 0.1 / (0.8 * 0.15 + 0.035))
   = 36.7 * ln(0.598 / 0.155)
   = 36.7 * ln(3.858)
   = 36.7 * 1.351
   = 49.6 Ω   ≈ 50 Ω (within 1%)
```

**Result: W ≈ 0.15 mm (6 mil) for 50 Ω microstrip on Layer 1**

This is above the 4 mil minimum trace width — fabrication is feasible at standard process.

### Part B — Layer 1 Edge-Coupled Differential Pair, 100 Ω

For a differential pair, the odd-mode impedance (half the differential impedance) must be 50 Ω. The coupling between traces slightly reduces the single-ended impedance compared to an isolated trace, so the trace width is slightly narrower than Part A for the same impedance target.

A simplified rule of thumb for edge-coupled microstrip differential impedance:

```
Zdiff ≈ 2 * Z0_single * (1 - 0.347 * exp(-2.9 * S/H))

Where:
  S = gap (space) between the two traces
  H = dielectric height
  Z0_single = single-ended impedance of one trace (isolated)
```

Working backward: to achieve Zdiff = 100 Ω with the traces slightly coupling:

Start with W = 0.15 mm (giving Z0_single ≈ 50 Ω in isolation), and test with S = 0.15 mm gap:

```
Zdiff = 2 * 50 * (1 - 0.347 * exp(-2.9 * 0.15/0.1))
      = 100 * (1 - 0.347 * exp(-4.35))
      = 100 * (1 - 0.347 * 0.0129)
      = 100 * (1 - 0.00447)
      = 99.6 Ω ≈ 100 Ω
```

**Result: W = 0.15 mm trace, S = 0.15 mm gap for 100 Ω differential on Layer 1**

Converting to mils: 0.15 mm = 5.9 mil ≈ 6 mil

This is above the 4/4 mil minimum — feasible at standard process.

**USB 3.2 — 90 Ω differential:**

USB 3.2 specifies 90 Ω differential. Repeating with a slightly wider trace or narrower gap to reduce impedance:

Try W = 0.175 mm, S = 0.175 mm:

```
Z0_single with W=0.175:
  Z0 = 36.7 * ln(5.98 * 0.1 / (0.8 * 0.175 + 0.035))
     = 36.7 * ln(0.598 / 0.175)
     = 36.7 * ln(3.42)
     = 36.7 * 1.23
     = 45.1 Ω

Zdiff = 2 * 45.1 * (1 - 0.347 * exp(-2.9 * 0.175/0.1))
      = 90.2 * (1 - 0.347 * exp(-5.075))
      = 90.2 * (1 - 0.347 * 0.00626)
      = 90.2 * 0.9978
      ≈ 90.0 Ω
```

**Result: W = 0.175 mm, S = 0.175 mm for 90 Ω differential on Layer 1**

### Part C — Layer 3 Stripline, 50 Ω Single-Ended

Layer 3 is centred between L2 (ground) and L4 (power). The power plane acts as an AC reference for signal integrity purposes (decoupling capacitors tie it to ground for high-frequency signals). Layer 3 is therefore a symmetric stripline: reference planes on both sides at equal distance.

Stackup geometry:
- L2 to L3: 0.36 mm core
- L3 to L4: 0.36 mm core
- Total H1 = H2 = 0.36 mm
- T = 0.035 mm (1 oz)
- Er = 4.5 (core material)

IPC-2141A symmetric stripline formula:

```
Z0 = (60 / sqrt(Er)) * ln(4 * (2*H - 0.8*T) / (0.67 * pi * (0.8*W + T)))

Simplified for symmetric stripline where H = distance to each plane:
  B = 2H = total distance between reference planes = 2 * 0.36 = 0.72 mm

Z0 = (60 / sqrt(Er)) * ln(4*B / (pi * (0.8*W + T) * 0.67))
```

Solving for W = 0.2 mm:

```
Z0 = (60 / sqrt(4.5)) * ln(4 * 0.72 / (pi * (0.8*0.2 + 0.035) * 0.67))
   = (60 / 2.121) * ln(2.88 / (pi * 0.195 * 0.67))
   = 28.29 * ln(2.88 / 0.410)
   = 28.29 * ln(7.02)
   = 28.29 * 1.949
   = 55.1 Ω   (too high)

Try W = 0.25 mm:

Z0 = 28.29 * ln(2.88 / (pi * (0.8*0.25 + 0.035) * 0.67))
   = 28.29 * ln(2.88 / (pi * 0.235 * 0.67))
   = 28.29 * ln(2.88 / 0.494)
   = 28.29 * ln(5.83)
   = 28.29 * 1.763
   = 49.9 Ω ≈ 50 Ω
```

**Result: W ≈ 0.25 mm (10 mil) for 50 Ω symmetric stripline on Layer 3**

Note that the inner layer trace is wider than the outer layer trace for the same impedance — because the dielectric height is much larger (0.36 mm vs 0.1 mm), the trace must be wider to maintain the same impedance ratio.

### Part D — Controlled Impedance Specification for Fab Notes

The fab notes must include a controlled impedance specification table. This is typically placed on a dedicated "fab notes" layer in the PCB Gerber package and repeated on the assembly/fab drawing.

```
CONTROLLED IMPEDANCE SPECIFICATION

Board: [Project Name / PCB Part Number]
Revision: A
Standard: IPC-2141A (impedance calculation basis)
Fabrication class: IPC Class 2
Impedance tolerance: ±10% (standard); upgrade to ±5% if required

Coupon: Include TDR test coupon (50 Ω and 100 Ω structures) in panel rail.
        Measure coupon per IPC-2141A. Retain coupon measurements with
        traveller documentation.

---------------------------------------------------------------------
No. | Layer | Type         | Target (Ω) | Tol.  | Trace W | Gap    | Reference Plane
---------------------------------------------------------------------
1   | L1    | Microstrip   | 50 Ω SE    | ±10%  | 0.15 mm | —      | L2 (GND)
    |       |              |            |       | (6 mil) |        |
2   | L1    | Diff. Pair   | 100 Ω DIFF | ±10%  | 0.15 mm | 0.15 mm| L2 (GND)
    |       | (PCIe)       |            |       | (6 mil) | (6 mil)|
3   | L1    | Diff. Pair   | 90 Ω DIFF  | ±10%  | 0.175 mm| 0.175mm| L2 (GND)
    |       | (USB 3.2)    |            |       | (7 mil) | (7 mil)|
4   | L3    | Stripline    | 50 Ω SE    | ±10%  | 0.25 mm | —      | L2 (GND), L4 (PWR)
    |       |              |            |       | (10 mil)|        |
---------------------------------------------------------------------

Notes:
1. All impedances are odd-mode (single-ended equivalent) unless noted as DIFF
   (differential). Differential values are measured tip-to-tip at the connector.
2. Stackup: See stackup callout drawing [reference]. Do not substitute prepreg
   or core materials without written approval and impedance re-verification.
3. Dielectric constants: prepreg Er = 4.2 @ 1 GHz, core Er = 4.5 @ 1 GHz.
4. Copper: 1 oz (35 µm finished) all layers.
5. Any deviation from the above requiring design changes must be raised as a
   PCB fabrication deviation request before first article approval.
```

---

## Key Takeaways

- Outer layer microstrip traces are narrower than inner layer stripline traces for the same impedance target, because the dielectric height to the reference plane is much smaller on outer layers
- The IPC-2141A formula is an approximation — it is accurate to ±5% for standard PCB geometries; use a field solver (Polar Si9000, Saturn PCB Toolkit) for precision-critical designs
- USB 3.2 (90 Ω) and PCIe (100 Ω) have different target impedances — the trace geometry must be different for each interface
- Controlled impedance MUST be specified in the fab notes — it cannot be inferred from Gerber geometry alone
- Always request TDR coupon measurements from the fab and retain them with the board documentation for traceability

---

## Interview Notes

Impedance calculation questions are extremely common in hardware engineering interviews. The key things interviewers look for:

1. Understanding of the physical basis — impedance is set by trace width, dielectric height, and dielectric constant. It is NOT set by trace length.
2. Knowing the difference between microstrip and stripline — and that inner layers give wider traces for the same impedance.
3. Knowing that the difference between differential and single-ended impedance is not simply 2× — coupling between the pair modifies the value.
4. Understanding that the spec has to be communicated to the fab explicitly.

Common trick question: "If I make the trace twice as long, does the impedance double?" No — impedance is a per-unit-length characteristic, independent of length. Length affects delay and loss, not impedance.
