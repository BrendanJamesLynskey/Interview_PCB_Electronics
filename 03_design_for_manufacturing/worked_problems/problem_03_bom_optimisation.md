# Problem 03: BOM Optimisation

## Problem Statement

You are reviewing the Bill of Materials (BOM) for a new wireless sensor node going into production at 10,000 units per year. The BOM currently contains 187 unique component references across 312 total placements. A design review has flagged the following observations:

- 14 different resistor values between 4.7 kΩ and 10 kΩ, all used as pull-up resistors
- 6 different MLCC decoupling capacitors with values 100 nF, but three different voltage ratings (6.3 V, 10 V, 16 V) and two different case sizes (0402 and 0603)
- 3 different linear regulators from 3 different manufacturers, each supplying 3.3 V
- 2 different NPN BJTs used as switch transistors in identical circuit topologies, rated 100 mA and 150 mA respectively

**Part A:** Define "BOM rationalisation" and explain its cost impact at 10,000 units/year. Why does unique component count matter beyond direct component cost?

**Part B:** Propose a rationalised value for the pull-up resistor family. Justify your choice technically (what pull-up resistance values are acceptable) and commercially (preferred value series).

**Part C:** Consolidate the decoupling capacitor variants. Which single variant should replace all six? Justify on electrical and commercial grounds.

**Part D:** Consolidate the three 3.3 V regulators to a single part. What criteria govern the selection, and what design changes (if any) are required to use the same part everywhere?

**Part E:** Can the two BJT switch transistors be consolidated? State any constraints and the correct selection approach.

---

## Solution Approach

### Part A — BOM Rationalisation: Definition and Cost Impact

**Definition:**

BOM rationalisation is the systematic reduction of unique component count by replacing multiple components of similar function and different part numbers with a single standardised part — without degrading circuit performance, reliability, or regulatory compliance.

**Direct cost impact at 10,000 units/year:**

Volume pricing for passive components follows a steep curve. Each unique part number in the BOM requires:

```
1. Setup fee amortisation: Kitting, reel setup, and pick-and-place programming
   for each part. Typical CM setup cost per unique part: £20-100 per production
   batch. At 10 batches/year: £200-1000 per unique line per year.

2. Minimum order quantity (MOQ): Most resistors and capacitors have MOQs of
   1000-10,000 pieces per value. Unique low-volume values may sit in inventory
   for years, tying up capital.

3. Volume pricing penalty: Ordering 10,000 × 14 different resistor values =
   714 pieces of each. At this quantity, unit price is typically 10-20× higher
   than ordering 10,000 × 10,000 pieces of a single value.

   Example:
     14 resistor values × 10,000 units = 714 pieces each
     Unit price at 714 pcs: £0.015 per resistor
     Total cost: 14 × 714 × £0.015 = £150

   With 1 rationalised value × 140,000 pieces:
     Unit price at 140,000 pcs: £0.002 per resistor
     Total cost: 1 × 140,000 × £0.002 = £280

   Cost is slightly higher in this case, but:
```

**Indirect cost savings of rationalisation (often larger than direct savings):**

```
(a) Engineering time: Each unique part requires verification, datasheet review,
    and DFM checking. Fewer unique parts = less engineering overhead.

(b) Supply chain risk: Each unique part is a potential supply chain failure point.
    A shortage of one unique pull-up resistor value can halt production.
    With a single consolidated value, all 14 positions are covered by one SKU.

(c) Inventory carrying cost: Fewer SKUs in stock = less warehouse cost and
    less risk of component obsolescence.

(d) Pick-and-place efficiency: The CM charges per component type as well as per
    placement. Fewer types means fewer feeder changeovers and faster setup.

(e) Incoming inspection: Each unique part is a separate inspection line item.
    Fewer unique parts reduces QC overhead.
```

At 10,000 units/year, the combined indirect savings from reducing from 187 to ~140 unique parts could easily amount to £5,000-20,000 per year, even if direct component costs do not decrease dramatically.

### Part B — Pull-Up Resistor Rationalisation

**Technical constraints on pull-up resistance value:**

Pull-up resistors serve two functions:

1. Logic level pull-up: Ensure a signal is at a defined logic HIGH when no driver is active.
2. Current limiting for open-drain/collector outputs (I2C, SMBus).

For a 3.3 V logic family:

```
Minimum value (maximum current): Set by the sink capability of open-drain drivers.
  I2C specification: max sink current 3 mA for standard mode/fast mode.
  R_min = V_DD / I_max = 3.3 V / 3 mA = 1.1 kΩ

Maximum value (rise time): Set by the RC time constant with line capacitance.
  For I2C at 100 kHz: t_rise ≤ 1 µs (standard mode), line cap ≈ 50 pF typical.
  R_max = t_rise / (0.847 × C) = 1 µs / (0.847 × 50 pF) = 23.6 kΩ

For general GPIO pull-ups (not I2C):
  100 Ω to 100 kΩ is technically valid for most microcontroller inputs (CMOS,
  very high input impedance). The constraint is leakage current and static power.
  At 3.3 V, 10 kΩ draws 330 µA per pull-up — for a battery-powered design,
  the number of always-on pull-ups is a power budget concern.
```

**E96 preferred value analysis:**

The 14 existing values between 4.7 kΩ and 10 kΩ are likely spread across E24 or E96 steps. A single value of **10 kΩ** is the standard first choice because:

```
- 10 kΩ is an E3 / E6 / E12 / E24 preferred value — maximum availability
- 10 kΩ at 3.3 V: 330 µA per pull-up — acceptable for battery systems if minimised
- Satisfies I2C pull-up constraint (10 kΩ < 23.6 kΩ max for 100 kHz, 50 pF)
- Universal stock value — stocked by all distributors in all temperature grades
  and tolerances (1%, 5%) from multiple manufacturers
```

**Recommended consolidation:** Replace all 14 values with **10 kΩ 1% 0402 ±100 ppm/°C** (e.g., Vishay CRCW040210K0FKED or equivalent).

For any pull-ups where 10 kΩ was determined to be technically insufficient (e.g., a specific I2C bus with long traces and high line capacitance requiring lower resistance), verify the design technically and, if a different value is genuinely needed, add it as a second SKU rather than restoring the full original spread.

### Part C — Decoupling Capacitor Consolidation

**Current variants:**

```
Variant  Value    Voltage   Case    Usage
A        100 nF   6.3 V     0402    Low-voltage rails (1.8 V, 3.3 V)
B        100 nF   10 V      0402    General decoupling
C        100 nF   16 V      0402    5 V rail and above
D        100 nF   10 V      0603    Through-hole hand-assembly sections (if any)
E        100 nF   6.3 V     0603    Possibly older layout sections
F        100 nF   16 V      0603    Mixed legacy use
```

**Selection criteria:**

```
Voltage rating: Apply the 2× voltage derating rule for X7R ceramics.
  Maximum supply voltage on this board: 5 V.
  Required capacitor voltage rating: 5 V × 2 = 10 V minimum.
  A 16 V rated part applied at 5 V: 31% of rated voltage → excellent derating.
  A 16 V rated part applied at 3.3 V: 21% → even better (flat DC bias curve).
  A 6.3 V rated part at 3.3 V: 52% → significant capacitance reduction due to
  DC bias (voltage coefficient effect in X7R).

DC bias effect quantification:
  100 nF 6.3 V X7R at 3.3 V bias: effective capacitance ≈ 50-60 nF
  100 nF 16 V X7R at 3.3 V bias: effective capacitance ≈ 90-95 nF

Case size: 0402 is preferred over 0603 for modern designs.
  0402 occupies 50% less PCB area.
  0402 has slightly lower ESL (~0.7 nH vs ~1.0 nH for 0603).
  Any 0603 positions can be padded out to fit 0402 (0402 physically fits inside
  0603 pads with solder bridging — verify with assembly house) or the pads can
  be resized in a minor layout revision.
```

**Recommended single variant: 100 nF 16 V X7R 0402**

```
This variant:
  - Satisfies the 2× derating rule for all voltages up to 8 V on this board
  - Provides near-nominal effective capacitance at all supply voltages (minimal
    DC bias effect at < 50% of rated voltage)
  - Uses the preferred small case size
  - Is a very high-volume commodity part — widest availability, lowest price
    per unit, least supply chain risk

Example part: Murata GRM155R71C104KA88 (100 nF 16 V X7R 0402)
  or Yageo CC0402KRX7R9BB104

Price benefit: Consolidating 6 variants to 1 at 10,000 units/year typically
  yields 15-30% cost reduction vs blended average price of multiple variants,
  plus eliminates 5 kitting/setup line items per production batch.
```

### Part D — 3.3 V Regulator Consolidation

**Current situation:** Three different regulators (presumably LDO types) from three manufacturers, all providing 3.3 V. Potential causes: different engineers specified different parts, or different current and noise requirements in different sections of the design.

**Criteria for selecting a single replacement:**

```
1. Output current: Select a part that meets the highest current requirement
   across all three use cases. Sum all rail loads: if the three regulators
   supply 100 mA, 250 mA, and 500 mA respectively, the consolidated part
   must supply at least 500 mA (and ideally > 600 mA with margin).

2. Input voltage range: The single part must be compatible with the input
   voltage for all three positions. If two regulators take 5 V input and
   one takes 12 V input, a 12 V-capable part is required (or the 12 V
   supply is pre-regulated).

3. Quiescent current: For battery-powered sections, the quiescent current
   of the regulator matters. A high-current LDO may have high IQ.
   Verify this against the power budget for each section.

4. Noise: Analogue supply regulators may require lower output noise than
   digital supply regulators. Check the noise specification (output voltage
   noise in µV_rms) for the most noise-sensitive position and select a
   part meeting that requirement. A low-noise LDO (e.g., 10 µV_rms) can
   replace a standard LDO (e.g., 50 µV_rms) anywhere without penalty.

5. Stability: Check the output capacitor requirement. Some LDOs require
   a minimum ESR in the output capacitor to remain stable (older designs);
   others require a low-ESR capacitor. A regulator requiring a specific
   output capacitor ESR range may not be compatible with the ceramic output
   capacitors used elsewhere.

6. Package: Consolidate to a single package. If two regulators are in SOT-223
   and one is in TO-252, specify the SOT-223 (smaller, easier to assemble,
   lower thermal resistance to PCB for moderate currents).
```

**Design changes required:**

- Verify the output capacitor specification matches the new regulator's stability requirements at each position. Some LDOs are PMOS-based and stable with ceramic capacitors; others are NMOS-based and require specific ESR ranges.
- Check the enable pin logic — some LDOs have active-high enable, others active-low. If the enable signal polarity differs between the old and new parts, add an inversion or connect to VIN directly.
- Verify thermal management: if the consolidated part handles more current than the original parts at some positions, check that the PCB copper pour or heat sinking is adequate.

**Example:** Texas Instruments TLV755P (500 mA, 1.5-6.5 V input, 10 µV_rms noise, 2 µA IQ, ceramic cap stable) is a versatile commodity LDO suitable for consolidation in most battery-powered applications.

### Part E — BJT Switch Consolidation

**Current situation:** Two NPN BJTs in identical circuit topologies, rated 100 mA and 150 mA respectively.

**Consolidation feasibility:**

Yes, this is an ideal consolidation candidate.

```
The circuit topology is identical — only the current rating differs.
The 150 mA part can directly replace the 100 mA part in all positions:
  - VCE(sat) at 100 mA load is typically lower for the 150 mA part
    (larger die → lower RCE_sat) — performance is maintained or improved
  - The base drive circuit is unchanged if the current gain (hFE) range is
    the same or wider for the 150 mA part
  - Package is typically the same for both (SOT-23 for 100-200 mA devices)

Selection criteria for the single replacement:
  (a) IC_max ≥ 150 mA (covers the higher-rated position)
  (b) VCE_sat at 100 mA ≤ specification requirement for the load
  (c) hFE range compatible with existing base resistor values
  (d) VCE_max and VBE characteristics compatible with supply voltages
  (e) Single vendor, widely stocked commodity device
```

**Check hFE and base resistor calculation:**

```
Existing circuit: MCU GPIO (3.3 V) → R_base → NPN base → collector → load.

For the 150 mA device (replacing the 100 mA device):
  At saturation: IC = 150 mA (worst case)
  Required IB for hard saturation: IB = IC / (hFE_min × 0.1 derating factor)
    e.g., hFE_min = 100, IC = 150 mA:
    IB_required = 150 mA / 10 = 15 mA

  R_base = (V_GPIO - VBE) / IB = (3.3 - 0.7) / 0.015 = 173 Ω

  If the existing R_base is designed for 100 mA IC at the same GPIO voltage:
    IB = (3.3 - 0.7) / R_base_existing = 2.6 V / R_base

  As long as the new 150 mA device has similar or better hFE, the base resistor
  does not need to change — the device goes deeper into saturation at 100 mA
  load, which is acceptable.

  Verify VCE_sat at the new device at 150 mA meets the headroom requirement
  for the load at the 100 mA position.
```

**Recommended replacement:** MMBT3904 (SOT-23, 200 mA, 40 V, hFE 100-300 at 10 mA) or BC817-40 (SOT-23, 500 mA, 45 V) — both are commodity parts stocked worldwide from multiple manufacturers, covering both 100 mA and 150 mA positions with margin.

---

## Key Takeaways

- BOM rationalisation reduces unique component count, which reduces kitting cost, supply chain risk, inventory overhead, and pick-and-place setup time — often more significant than direct component cost reduction
- 10 kΩ is the standard first-choice pull-up resistor value: satisfies I2C timing at 100 kHz, E3 preferred value, maximum availability
- 100 nF 16 V X7R 0402 is the standard single decoupling capacitor for consolidation: correct voltage derating for all rails up to 8 V, minimal DC bias loss, preferred package
- LDO consolidation requires checking: maximum current, input voltage range, noise, output capacitor stability range, and package — select to the most demanding requirement across all positions
- BJT switch consolidation: always consolidate to the higher-rated device; verify hFE and base drive compatibility

---

## Interview Notes

BOM optimisation questions appear in PCB and hardware engineering interviews at companies that are cost- and supply-chain-sensitive — which is most companies in volume production. The interviewer is assessing:

1. Understanding of why BOM count matters beyond direct unit cost (supply chain, inventory, setup cost, risk)
2. Knowledge of how to apply derating rules to consolidate component ratings
3. Awareness of the DC bias (voltage coefficient) effect in MLCCs — the most commonly cited component characteristic in BOM rationalisation interviews
4. Practical judgment: when is it safe to replace a lower-rated part with a higher-rated part, and what checks are needed?

A weak answer names a single consolidation action without justifying it. A strong answer quantifies the impact (cost per unit, risk reduction), states the technical constraint being satisfied, and identifies the minimum number of checks required to validate the change.
