# Problem 01: No Power Debug

## Problem Statement

A first-article four-layer PCB has been assembled. It contains:
- Input: 12 V barrel jack connector
- A 5 V/2 A synchronous buck converter (Texas Instruments TPS54240)
- A 3.3 V/500 mA LDO (TI TLV1117-3.3), powered from the 5 V rail
- A 1.8 V/1 A synchronous buck converter (TI TLV62130), powered from the 3.3 V rail
- A Cortex-M4 microcontroller (STM32F405) on the 3.3 V core and 1.8 V I/O rails
- All passives: resistors, capacitors, decoupling network

On first power-on using a current-limited bench supply set to 12 V / 500 mA, the supply
immediately current-limits and the voltage collapses to approximately 2 V. No components
are visibly damaged. The board has not been powered before.

**Task:** Describe the complete diagnostic and resolution procedure.

---

## Solution Approach

### Step 0 — Stop and Think Before Probing

Resist the temptation to immediately start probing. Take 30 seconds to form hypotheses
based on what is known:

A collapse on first power-on with no prior visible damage means the fault is almost
certainly one of:
1. A hard short between the 12 V input rail and GND (solder bridge, wrong component,
   reversed polarised component)
2. A low-impedance path that is not a dead short but draws more current than the
   500 mA limit (a regulator output shorted to GND, a wrong component value that
   causes excessive quiescent current)
3. An inrush event that momentarily exceeds the current limit and causes the supply
   to fold back (less likely to cause sustained collapse — the supply would recover)

The fact that the voltage stabilises at 2 V (rather than zero) suggests the supply is
current-limiting and sharing the 12 V across the supply impedance and the load. This
pattern is consistent with a resistive-but-low short rather than a dead (0 Ω) short.

### Step 1 — Pre-Power Resistance Check (if not already done)

Remove power. Use a multimeter in resistance mode on the 12 V input rail to GND.

```
Expected range: 100 Ω to several kΩ (depending on number of input capacitors,
                reverse protection components, and MOSFET gate pulldowns)

< 1 Ω          → Dead short on input rail — find and fix before proceeding
1-10 Ω         → Low-impedance fault — likely cause of current limit
10-100 Ω       → Suspicious but could be normal depending on design
> 100 Ω        → Input rail looks healthy; fault may be on a downstream rail
```

In this scenario, the 12 V rail reads 8 Ω. This is suspicious — too low for a healthy
design with only input capacitors.

### Step 2 — Identify the Shorted Rail by Isolation

The board has three rails downstream of the 12 V input: 5 V buck output, 3.3 V LDO
output, 1.8 V LDO output. The 12 V input feeds only the buck converter's power stage.

**Method: successive isolation using 0 Ω resistors or physical component removal.**

If the schematic contains isolation jumpers (0 Ω resistors) on power rails for exactly
this purpose, remove each one and re-measure the input rail resistance:

```
Remove 0 Ω on 5 V output path → 12 V input resistance measurement: unchanged (still 8 Ω)
→ The short is not on the 5 V rail side; the fault is on the 12 V input side of the buck

If resistance changes on removing a jumper → the fault is on that rail
```

If no isolation jumpers exist, the alternative is to unsolder suspect components:

The TPS54240 buck converter input is directly on the 12 V bus. Key candidates:
- Input decoupling capacitors (MLCC): can fail shorted (rare, but possible with
  a quality issue or overvoltage during transport)
- Input EMI filter inductors: rarely fail short, but if wrong component placed
  (e.g., a ferrite bead rated for a different impedance) could cause issues
- The TPS54240 VIN pin itself: the MOSFET body diodes could be conducting if
  the device is damaged or installed at wrong orientation

### Step 3 — Power Injection Technique for Short Location

With power still removed, apply a small, safe current from a second bench supply
set to a low voltage (1-2 V maximum) directly onto the 12 V rail via the input
connector. Set the current limit to 200-300 mA.

The current will flow into the short and heat the fault location. Use a thermal
camera or IR thermometer to scan the board for localised heating within 10-15 seconds
of applying the injection current.

```
If a component heats rapidly (within 5 seconds):
  → That component is the short. Its temperature rise = I² × R × time/thermal_mass

For a 0.5 Ω short with 300 mA:
  P = 0.3² × 0.5 = 45 mW
  A small MLCC at 45 mW heats to detectable temperature in < 10 seconds
```

**In this scenario:** The thermal camera shows a single 100 nF MLCC on the 12 V
input decoupling network heating to 65°C while all adjacent components remain at
ambient. This is the faulty component.

### Step 4 — Confirm and Remove the Faulty Component

Remove the MLCC identified by thermal imaging:
1. Apply flux
2. Heat both pads simultaneously with a chisel iron tip
3. Slide the component off with tweezers

Re-measure the 12 V input rail resistance: now reads 2.7 kΩ — consistent with
a healthy rail dominated by the input capacitors and MOSFET gate pulldown resistors.

**Root cause of the failed capacitor:**
- X7R MLCCs can fail short if subjected to voltage stress during handling (ESD)
  or if a manufacturing defect (metal inclusion in the dielectric) causes early
  failure. In production, incoming inspection on high-voltage rated capacitors
  includes a 100% voltage screen.
- Another common cause: a 10 V rated 100 nF MLCC placed on a 12 V rail. At 12 V,
  this capacitor is operating above its rated voltage, which accelerates dielectric
  breakdown. Always check that each component's rated voltage exceeds the applied
  voltage by the derating margin (typically 1.5-2×).

In this case, review the BOM: the input capacitor was specified as 100 nF, 10 V, X7R.
On a 12 V rail, the 10 V capacitor is over-rated — the applied voltage exceeds the
rating. This is a design error. The correct part is 100 nF, 25 V (or higher), X7R.

### Step 5 — Replace the Component and Retest

Replace with a 100 nF 25 V X7R MLCC. Re-measure input rail resistance: 2.7 kΩ.
Apply 12 V via the bench supply with 500 mA limit.

```
Result: supply does not current-limit. Measures:
  12 V input:  12.01 V
  5 V output:  4.97 V  (within ±5%)
  3.3 V output: 3.29 V (within ±5%)
  1.8 V output: 1.81 V (within ±5%)
  Input current: 42 mA quiescent
```

All rails within specification. Continue with the standard bring-up procedure.

---

## Analysis

### Why the Capacitor Failed in This Way

An X7R MLCC internal short circuit fails to low (but non-zero) resistance, typically
1-50 Ω depending on the severity of the failure. This explains the bench supply
behaviour:

```
V_supply × (R_fault / (R_fault + R_supply_internal)) = V_measured

12 V × (8 / (8 + 8)) ≈ 6 V  — doesn't match the observed 2 V

A more accurate model includes the current-limiting behaviour:
At 500 mA limit, V_output = V_ocv - I_limit × R_supply_internal
If R_supply_internal = 20 Ω (aggressive current limit): V = 12 - 0.5×20 = 2 V ✓
```

The bench supply, once current-limiting, folds back its output voltage as it attempts
to maintain I_out = I_limit into a resistive load. This is entirely consistent with
the observation of V = 2 V at the 500 mA limit.

### Why the Voltage Did Not Collapse to Zero

A dead short (0 Ω) would cause the supply to fold back to near 0 V. The 2 V reading
indicates a resistive fault, not a direct conductor-to-conductor bridge. This is a
useful diagnostic distinction — a solder bridge between adjacent copper features would
likely produce near-zero voltage, while a component internal fault produces a non-zero
resistive path.

### Design Error Identified: Wrong Capacitor Voltage Rating

The root cause is a component on the BOM specified below the required voltage rating.
The correct corrective actions are:

1. **Immediate:** Replace all units of the under-rated capacitor on this board and
   all boards from the same batch.
2. **BOM correction:** Change C47 (the input decoupling capacitor) from 100 nF 10 V
   to 100 nF 25 V X7R 0402.
3. **ECN:** Raise ECN-0001 documenting the change with reason, affected units, and
   verification method.
4. **Schematic note:** Add a note to the input decoupling capacitor specifying minimum
   voltage rating = V_rail × 2, to prevent recurrence.
5. **Design review checklist:** Add a design rule: "Verify voltage rating of all
   capacitors with applied voltage derating check before release."

---

## Key Takeaways

1. **Always check component voltage ratings against applied voltages during schematic
   review.** A 10 V capacitor on a 12 V rail is an error that a competent design review
   would catch. Derating guidelines require capacitors to be rated at a minimum of
   1.5-2× the applied voltage.

2. **The power injection technique is the fastest way to locate a resistive short.**
   Injecting a small, safe current and using a thermal camera to find the heating
   component is faster and more definitive than resistance measurements alone, which
   can only tell you the total fault impedance, not its location.

3. **Thermal imaging in the first 10-15 seconds is more useful than steady-state thermal
   imaging for finding resistive faults.** The fault location heats faster (less thermal
   mass) than the rest of the board. Wait too long and the heat spreads, obscuring the
   fault location.

4. **A bench supply current limit is a diagnostic instrument, not just a safety device.**
   The voltage at which the supply stabilises when current-limiting gives you information
   about the impedance of the fault.

5. **First-article board failures are often assembly or BOM errors, not design errors.**
   In this case, both conditions were present: a wrong-value component (design error in
   the BOM) and a component that failed as a result (assembly outcome). The response
   to a bring-up failure must always trace back to root cause — stopping at "bad cap"
   without identifying why the cap failed would allow the same error to recur.

---

## Interview Notes

**Common question variants:**

- "Walk me through how you would debug a board that doesn't power up."
  - Answer structure: stop and think, pre-power inspection, isolation, power injection,
    confirm, replace, retest, root-cause and correct the underlying design error.

- "How do you find a short circuit on a PCB without a thermal camera?"
  - Use the four-wire (Kelvin) resistance measurement: measure resistance between the
    shorted rail and GND at multiple points on the board. The resistance decreases as
    you move the probe toward the fault location. This is a voltage-divider technique
    using the distributed resistance of the copper traces.

- "What would you do differently if this happened on a production board vs. a prototype?"
  - Production: do not attempt to rework without a formal ECN. Quarantine all boards
    from the affected batch. Determine if the issue is systemic (wrong part loaded at
    the CM) or isolated (single component failure). Test all boards before shipment.
  - Prototype: document the fault and fix, but rework is acceptable with a log entry.

- "How would you prevent this fault in future designs?"
  - Design review checklist item: voltage derating check on all capacitors
  - Schematic review tool: use ERC rules to flag capacitors where rated voltage is
    less than the net voltage
  - BOM check: automated comparison of component voltage rating vs. net voltage
    (many EDA tools support this as a component attribute check)
