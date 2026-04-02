# Decoupling Strategy

## Prerequisites
- Capacitor parasitics: ESR, ESL, and self-resonant frequency (see `component_selection.md`)
- Basic impedance and frequency-domain concepts (XC, XL, resonance)
- Power distribution network (PDN) fundamentals
- MLCC dielectric types and their voltage coefficient behaviour

---

## Concept Reference

### Why Decoupling Is Necessary

Every digital switching event — a logic gate toggling, a MOSFET turning on, a DDR burst write — demands a sudden change in supply current. The power delivery network (PCB traces, vias, and planes) has finite inductance. By Faraday's law, any inductance resists instantaneous current change, causing a supply voltage droop:

```
V_droop = L_PDN × dI/dt

Example: 10 nH trace inductance, 1 A current step in 1 ns:
  V_droop = 10×10⁻⁹ × 1 / 1×10⁻⁹ = 10 V  (catastrophic — logic fails immediately)

With a 100 µF decoupling cap 2 cm away (L_PDN = 2 nH to cap):
  During the first 1 ns, cap supplies current; droop ≈ 2 nH × 1 A / 1 ns = 2 V (still bad)

With 100 nF cap 1 mm away (L_PDN = 0.1 nH):
  V_droop = 0.1 nH × 1 A / 1 ns = 0.1 V (acceptable)
```

The key insight: **the decoupling capacitor closest to the load, with the shortest possible current loop, has the lowest effective inductance and provides charge fastest.** Larger, farther-away capacitors provide charge over longer timescales.

---

### Impedance Target and the PDN Model

A PDN is designed so that the impedance at the load's power pins stays below a target value across the entire relevant frequency band:

```
Z_target = ΔV_allowed / ΔI_max

Example: 1 V core supply, 5% ripple tolerance, 10 A transient:
  Z_target = (1.0 × 0.05) / 10 = 5 mΩ

This 5 mΩ target must be maintained from DC to the highest frequency
current harmonic of the load (often up to 500 MHz for modern processors).
```

A single capacitor cannot achieve low impedance across many decades of frequency. The PDN uses a hierarchy of capacitors, each covering a frequency range:

```
Frequency range      Capacitor type        Location
------------------   -----------------     --------------------------
DC to ~1 kHz         Bulk electrolytic     Near PCB connector / VRM
1 kHz to 1 MHz       10–100 µF MLCC        On board, within 5 cm of load
1 MHz to 100 MHz     100 nF MLCC           Within 5 mm of each VCC pin
100 MHz to 1 GHz     10–100 pF C0G         Within 0.5 mm, or under the IC (via-in-pad)
>1 GHz               Package capacitance   Inside the IC package / die capacitance
```

The objective is a flat impedance profile (no resonant peaks above Z_target) across the full frequency range. Resonant peaks arise from the anti-resonance between adjacent capacitor tiers.

---

### Anti-Resonance Between Capacitor Tiers

When two capacitors are placed in parallel and their values differ significantly, their series-resonant frequencies differ. Between these frequencies, the combined impedance can be higher than either capacitor alone — an anti-resonance peak.

```
Tier 1: 10 µF cap, ESL = 2 nH → SRF₁ = 1/(2π√(LC)) = 1/(2π√(2n×10µ)) = 1.13 MHz
Tier 2: 100 nF cap, ESL = 1 nH → SRF₂ = 1/(2π√(1n×100n)) = 15.9 MHz

Between 1.13 MHz and 15.9 MHz:
  10 µF cap is above its SRF → appears inductive (ZL = jωL)
  100 nF cap is below its SRF → appears capacitive (ZC = 1/jωC)
  These two impedances form an LC parallel resonance with a peak
  Peak frequency ≈ √(SRF₁ × SRF₂) geometric mean ≈ 4.2 MHz
```

**Mitigation strategies for anti-resonance:**

1. **Overlap frequency coverage:** Use values close enough that SRFs overlap or the gap between tiers is small.
2. **Increase ESR of larger capacitors deliberately:** Adding a small resistor in series damps the resonance peak.
3. **Use multiple values of the mid-tier:** Instead of one 100 nF cap, use 47 nF + 220 nF to spread coverage.
4. **Power integrity simulation:** Use tools (SIwave, Cadence Sigrity, or even LTspice) to simulate the PDN impedance profile across frequency before layout.

---

### Decoupling Per Pin vs Shared Decoupling

**Per-pin decoupling:**

Each VCC/VDD pin on an IC has its own dedicated decoupling capacitor, placed directly adjacent to that pin.

```
Advantages:
  Lowest effective inductance from cap to pin (shortest current loop)
  Failures or misplacement affect only that pin, not others
  Required by almost all high-speed IC datasheets

Disadvantages:
  More components (BOM count)
  Requires careful placement — a cap placed 5 mm away has much higher ESL
  Potentially redundant if multiple VCC pins are on the same internal power rail
```

**Shared decoupling:**

One larger capacitor serves multiple nearby VCC pins, or pins on different ICs that are close to each other.

```
Acceptable when:
  Multiple VCC pins are tied internally (many small MCUs have one internal VDD
  bus; all VCC pins can share 2-3 caps placed centrally among them)
  The shared capacitor is very close to all served pins (< 1-2 mm away each)

Not acceptable when:
  High-speed pins with fast dI/dt — shared path length increases effective inductance
  Differential signal pins where per-pin symmetry is required
  The IC datasheet specifies dedicated caps per pin
```

**Rule of thumb:** For any IC with slew rate > 1 V/ns or switching currents > 500 mA per pin, use dedicated per-pin decoupling. For simple 3.3 V logic running at < 50 MHz, shared decoupling is usually acceptable if placement is tight.

---

### Capacitor Placement Principles

#### Distance and Loop Area

The inductance of a current loop is proportional to the area enclosed by the loop. The decoupling current path is:

```
VCC plane → via → capacitor → via → GND plane (for a capacitor between planes)

Or:

VCC trace → pad → capacitor → pad → GND trace → GND via → GND plane → VCC plane → VCC via → back to IC pin
```

Minimise loop area by:
- Placing the capacitor as close as possible to the VCC/GND via pair of the IC pin it decouples.
- Routing the capacitor directly between the IC VCC via and a GND via directly beside it — not via a long trace first.
- On dense boards, use via-in-pad or back-side capacitors to eliminate trace length entirely.

```
Good placement (minimises loop):
  IC VCC pad → short trace (< 0.5 mm) → cap → short trace → GND via
  |_________________ ~1 mm loop ___________________________|

Poor placement:
  IC VCC pad → 5 mm trace → cap → 5 mm trace → GND via
  (The 10 mm loop has inductance roughly 10× higher)
```

#### Via Placement

For capacitors on the same side as the IC (most common), the via connecting to the supply plane goes through the board. Inductance of a PCB via is approximately:

```
L_via ≈ 5.08 × h × [ln(4h/d) + 1] (nH, h and d in inches)
  h = via length (board thickness)
  d = via drill diameter

Typical: 1.6 mm board, 0.3 mm drill:
  L_via ≈ 5.08 × 0.063 × [ln(4×0.063/0.0118) + 1] ≈ 0.32 × [ln(21.4) + 1]
         ≈ 0.32 × 4.07 ≈ 1.3 nH per via

Two vias in parallel: ≈ 0.65 nH — use two GND vias per capacitor where inductance budget is tight.
```

**Recommended via topology for decoupling caps:**

- Place one via directly at each end pad of the capacitor (one VCC via, one GND via).
- Do not route a trace from the cap to a via located further away — this adds trace inductance in series.
- For the highest-frequency decoupling (under-BGA caps), use dog-bone via-in-pad or embedded caps.

---

### ESR and ESL Trade-offs

**ESR (Equivalent Series Resistance):**

ESR sets the floor of impedance at resonance — a capacitor cannot present lower impedance than its ESR, no matter how large C is:

```
Z_min = ESR (at the self-resonant frequency)

A 10 µF MLCC with ESR = 5 mΩ presents 5 mΩ minimum impedance.
A 100 µF wet electrolytic with ESR = 100 mΩ presents 100 mΩ minimum.

For a Z_target = 5 mΩ PDN, only low-ESR MLCCs meet the target.
```

**ESR also damps anti-resonance peaks:** Higher ESR broadens the resonant dip and reduces anti-resonance peaks. In some PDN designs, a deliberate high-ESR bulk capacitor is placed in parallel with low-ESR MLCCs to damp the anti-resonance between tiers. Polymer electrolytic capacitors (ESR 5-50 mΩ) are useful for this role.

**ESL (Equivalent Series Inductance):**

ESL is dominated by package geometry:

```
Package     Approximate ESL
-------     ---------------
0402 MLCC   0.5-1.0 nH
0201 MLCC   0.3-0.5 nH
01005 MLCC  0.1-0.2 nH
Large can electrolytic  10-30 nH
Through-hole cap  30-100 nH (avoid for high-frequency decoupling)
```

Placing two capacitors in parallel halves the effective ESL:

```
N capacitors in parallel: ESL_eff = ESL_single / N
```

This is one justification for multiple small caps instead of one large cap: same total capacitance, lower total ESL, lower SRF.

---

### SPICE Simulation of the PDN

A simple LTspice model of a PDN tier:

```spice
* Two-tier PDN: 10uF bulk + 100nF local decoupling + IC current demand
* Bulk cap tier
L_bulk 1 2 5nH        ; inductance from PCB trace to bulk cap
R_esr_bulk 2 3 50mΩ   ; ESR of bulk cap
C_bulk 3 0 10µF       ; bulk capacitance

* Local decoupling tier
L_local 1 4 0.5nH     ; short trace inductance to local cap
R_esr_local 4 5 3mΩ   ; ESR of local MLCC
C_local 5 0 100nF     ; local MLCC

* Load current source (step transient)
I_load 1 0 PULSE(0 2 1n 100p 100p 10n 20n)  ; 2A step in 100 ps

* Supply (ideal voltage source with series inductance)
V_supply 0 10 1.8V
L_supply 10 1 10nH    ; PDN inductance from VRM to load

.tran 100p 100n
.meas TRAN V_droop MIN V(1)
.end
```

Run this simulation, plot V(1), and observe the voltage droop. Iterate component values and placement (inductance) to meet the target.

---

## Tier 1 — Fundamentals

### Question F1
**Why is a 10 µF decoupling capacitor placed 10 cm from the IC less effective than a 100 nF capacitor placed 1 mm from the IC at frequencies above 10 MHz?**

**Answer:**

The effectiveness of a decoupling capacitor depends on how quickly it can respond to a current demand — which is governed by the inductance of the current loop from capacitor to load, not the capacitance value alone.

The 10 µF capacitor 10 cm away has a supply path inductance of roughly 10-50 nH (10 cm of trace ≈ 10-20 nH + via inductances). Its self-resonant frequency is:

```
SRF = 1 / (2π × √(L × C)) = 1 / (2π × √(20n × 10µ)) = 356 kHz
```

Above 356 kHz, this capacitor appears inductive — increasing its impedance with frequency. At 10 MHz it presents:

```
Z ≈ 2π × 10 MHz × 20 nH = 1.26 Ω  (purely inductive, useless as a decoupler)
```

The 100 nF capacitor 1 mm away has trace and via inductance of approximately 0.3-1 nH. Its SRF:

```
SRF = 1 / (2π × √(0.5n × 100n)) = 22.5 MHz
```

At 10 MHz (below its SRF), this cap is still capacitive and presents:

```
Z ≈ 1 / (2π × 10 MHz × 100 nF) = 0.16 Ω  (capacitive, actively decoupling)
```

The 100 nF cap is 8× lower impedance at 10 MHz despite being 100× smaller in capacitance value. This illustrates that **placement and resulting loop inductance — not capacitance value — determine high-frequency decoupling effectiveness.**

---

### Question F2
**A design has a single 100 µF electrolytic capacitor at the VCC connector and nothing else. What is wrong with this, and what would a correct decoupling strategy look like for a board with a 1 GHz microcontroller?**

**Answer:**

The single 100 µF electrolytic fails for three reasons:

1. **High ESL:** A typical through-hole electrolytic has 30-100 nH of inductance. At frequencies above a few hundred kHz, it appears inductive and its impedance rises with frequency rather than falling.

2. **High ESR:** A wet electrolytic may have ESR of 100-500 mΩ — far above the < 10 mΩ target for a processor core rail.

3. **Far from the load:** The capacitor at the connector is connected to the processor through centimetres of PCB trace with additional inductance.

A correct strategy for a 1 GHz microcontroller:

```
Tier 1 — Bulk energy reservoir (handles slow load steps, holds charge during
          power sequencing):
  2× 47 µF polymer electrolytic (ESR ~20 mΩ each) placed within 1 cm of the
  processor. Also provides 100 µF near connector for bulk energy.

Tier 2 — Mid-frequency decoupling (1 kHz to 10 MHz):
  4-8× 10 µF 0402 X7R MLCC (16 V rating to avoid DC bias loss at 1 V core)
  Placed within 3-5 mm of VCC ball group or VCC pin cluster.

Tier 3 — High-frequency decoupling (10 MHz to 200 MHz):
  1× 100 nF 0402 X7R per VCC pin, placed within 1 mm of each VCC via.
  Caps connect from VCC via to an adjacent GND via — not to a shared GND trace.

Tier 4 — Very high frequency (200 MHz to 1 GHz):
  1× 10 nF 0201 or via-in-pad cap per VCC/GND pair, directly under the BGA
  package. C0G preferred at this small value for frequency stability.
  10-100 pF 0201 C0G on critical VCC pins if datasheet recommends.

Result: Flat PDN impedance from DC to > 500 MHz, with all peaks below Z_target.
```

The key principle: multiple tiers, each sized and placed to be effective in its specific frequency range.

---

### Question F3
**What is the self-resonant frequency (SRF) of a capacitor and why does it matter for decoupling?**

**Answer:**

The self-resonant frequency is the frequency at which a real capacitor's inductive reactance (from its equivalent series inductance, ESL) equals its capacitive reactance. At the SRF, the impedance is at a minimum, equal to the ESR alone. Above the SRF, the capacitor behaves as an inductor — its impedance increases with frequency.

```
f_SRF = 1 / (2π × √(ESL × C))

Example: 100 nF 0402 MLCC, ESL = 0.7 nH:
  f_SRF = 1 / (2π × √(0.7×10⁻⁹ × 100×10⁻⁹))
        = 1 / (2π × 8.37×10⁻⁹) = 19 MHz

At 10 MHz (below SRF): behaves as capacitor, low impedance — effective decoupling
At 19 MHz: minimum impedance = ESR only — best decoupling performance
At 50 MHz (above SRF): behaves as inductor, rising impedance — ineffective
```

**Why it matters:** A 100 nF capacitor does not decouple above ~50 MHz in a typical 0402 package. If the IC draws current at 100 MHz, this capacitor is not helping. Smaller capacitors (10 nF, 1 nF) with lower ESL and higher SRF must be placed even closer to serve the high-frequency range.

**Common mistake:** Assuming a large-value capacitor is always better. A 10 µF MLCC has SRF of ~1-3 MHz; it is entirely inductive at 100 MHz. It must be supplemented by smaller-value, physically smaller capacitors placed right at the IC pins.

---

## Tier 2 — Intermediate

### Question I1
**Your 1.2 V DDR5 memory rail has a Z_target of 2 mΩ from 100 kHz to 500 MHz. You have chosen 100 nF 0402 X7R MLCCs (ESL = 0.7 nH, ESR = 4 mΩ) for local decoupling. How many do you need in parallel to meet the 2 mΩ impedance target at the SRF of the array? What limits how many you can use effectively?**

**Answer:**

**Step 1 — Impedance at SRF for N capacitors in parallel:**

When N identical capacitors are placed in parallel:
- Total capacitance: C_total = N × C
- Total ESR: ESR_total = ESR / N (parallel resistance)
- Total ESL: ESL_total = ESL / N (parallel inductance)

The SRF of the array is unchanged (since both L and C scale by N, √(LC) = √((L/N)(NC)) = √(LC)):

```
f_SRF = 1 / (2π × √(ESL/N × N×C)) = 1 / (2π × √(ESL × C)) — same as single cap
```

At the SRF, impedance = ESR_total = ESR / N:

```
For Z_target = 2 mΩ, ESR = 4 mΩ:
  2 mΩ = 4 mΩ / N → N = 2 minimum to meet impedance at resonance
```

**Step 2 — Check impedance at 500 MHz (above SRF for single cap):**

Above SRF, impedance is dominated by ESL:

```
For N = 2 caps in parallel:
  ESL_total = 0.7 nH / 2 = 0.35 nH
  Z at 500 MHz = 2π × 500 MHz × 0.35 nH = 1.1 Ω  (way too high!)
```

This shows that 100 nF caps alone cannot meet 2 mΩ at 500 MHz. Smaller-value caps with lower ESL (e.g., 1 nF 0201 with ESL = 0.2 nH) are needed for the > 200 MHz range.

**Step 3 — Practical limits on parallelising:**

Diminishing returns occur because:

1. **Physical placement:** Each capacitor must be close to the IC pins it serves. If N = 20 caps, they cannot all be within 0.5 mm — some will be further away, increasing their effective ESL and negating the benefit of paralleling.

2. **Anti-resonance between tiers:** Adding more 100 nF caps increases the anti-resonance peak between the 100 nF tier and the next smaller tier. At some point, adding more caps of one value worsens the peak more than it helps at resonance.

3. **BOM and area:** Each capacitor costs money and occupies PCB space.

**Conclusion:** Use N = 2-4 of the 100 nF caps for ESR improvement in the 10-100 MHz band, then supplement with a separate tier of 1-10 nF caps in 0201 package to cover 100-500 MHz. Power integrity simulation with actual via inductances gives the definitive answer.

---

### Question I2
**Describe the difference between a "bypass" capacitor and a "decoupling" capacitor. Are these different design problems?**

**Answer:**

The terms are used interchangeably in common usage, but there is a useful distinction:

**Bypass capacitor:** Provides a low-impedance path to ground for AC noise on a DC supply rail, bypassing it away from the load. The goal is to prevent noise from entering the IC. The focus is on filtering incoming supply noise.

```
Supply with noise → [Z_source] → VCC_pin
                                      |
                                  [Bypass cap]
                                      |
                                     GND

The bypass cap + source impedance forms a low-pass filter.
Corner frequency: f_c = 1 / (2π × Z_source × C)
```

**Decoupling capacitor:** Provides local charge storage to supply instantaneous current demand by the IC, preventing the IC's switching currents from propagating back through the supply trace inductance and causing VCC droop. The focus is on supplying the IC's own transient demand.

```
VCC_plane → [L_trace] → VCC_pin
                              |
                          [Decoupling cap] ← supplies ΔI during switching transient
                              |
                             GND

The cap provides dI/dt that the inductor would otherwise prevent.
```

**Are they the same design problem?** In practice, yes — a well-placed low-ESL/ESR capacitor directly at the IC pin solves both:
- It decouples the IC's transient demand (local charge reservoir).
- It bypasses incoming supply noise to ground.

The same capacitor achieves both goals if it is placed correctly and sized correctly. Where they diverge as separate concerns: a bypass cap for a noise-sensitive analogue circuit (e.g., ADC reference) may need a high-frequency LC filter rather than just a capacitor, to prevent switching regulator harmonics at 2 MHz from reaching the reference pin. A decoupling cap for a digital core is more about charge reservoir than attenuation.

---

### Question I3
**A designer proposes to save PCB area by removing all the 100 nF per-pin decoupling capacitors from a 32-pin 3.3 V microcontroller and instead increasing the bulk 10 µF cap from 1 to 10 µF. Critique this proposal.**

**Answer:**

This proposal conflates capacitance value with decoupling effectiveness and will likely cause problems. The critique proceeds on three grounds:

**1. Frequency coverage:**

The 100 nF per-pin caps serve the 10-200 MHz frequency range where the 10 µF bulk cap (SRF ~1-3 MHz) is already inductive. Replacing them with a larger bulk cap does not address high-frequency decoupling at all — the SRF of the 10 µF cap does not change meaningfully, and it remains inductive above a few MHz.

**2. Loop inductance:**

A per-pin 100 nF cap placed 0.5 mm from the VCC pin has a current loop inductance of ~0.1-0.3 nH to that specific pin. The bulk 10 µF cap, even if increased to 100 µF, is several centimetres away with 5-20 nH of effective loop inductance to any individual pin. The transient current for that pin's switching events must traverse the full loop inductance to the bulk cap — causing significant voltage droop on that pin.

**3. Per-pin isolation:**

If pin 12 draws a fast transient current, its local cap absorbs it without disturbing pins 5, 19, or 28. Without per-pin caps, all pins share the bulk cap, and a fast event on any pin creates a voltage disturbance visible on all other pins — potential cross-talk through the supply rail.

**Correct area-saving approach:**

If area is genuinely constrained, consider:
- Using 0201 package instead of 0402 for per-pin caps (saves ~50% area).
- Sharing one 100 nF cap between two adjacent VCC pins if their switching events are decorrelated and the cap can be positioned centrally between the two pins.
- Checking whether adjacent VCC pins are internally connected — if so, they can share one cap.

The MCU datasheet's recommended decoupling circuit should always be followed as the minimum starting point.

---

## Tier 3 — Advanced

### Question A1
**Design the PDN for a FPGA (Xilinx Artix-7) with VCCINT = 1.0 V at 3 A peak, VCCO_0 = 3.3 V at 500 mA, and VCCAUX = 1.8 V at 200 mA. The FPGA switching activity reaches its maximum at an internal clock of 200 MHz. Specify the decoupling strategy with justification for each tier, and identify the anti-resonance concern between the VCCINT tiers.**

**Answer:**

**Step 1 — Impedance targets:**

```
Z_target(VCCINT)  = ΔV_allowed / ΔI = (1.0 × 0.03) / 3 = 10 mΩ
  (30 mV noise budget, 3 A transient at 200 MHz fundamental → harmonics to ~1 GHz)

Z_target(VCCO_0)  = (3.3 × 0.05) / 0.5 = 330 mΩ  (more relaxed)

Z_target(VCCAUX)  = (1.8 × 0.05) / 0.2 = 450 mΩ   (most relaxed)
```

The VCCINT rail drives the primary concern: 10 mΩ from 100 kHz to > 500 MHz.

**Step 2 — VCCINT decoupling tiers:**

```
Tier 1 — Bulk (DC to 500 kHz):
  2× 100 µF polymer electrolytic (ESR ~20 mΩ each → parallel ESR = 10 mΩ)
  Placed within 10 mm of FPGA package, connected to VCCINT plane via short traces.

Tier 2 — Mid-frequency (500 kHz to 10 MHz):
  8× 10 µF 0402 X7R, 10 V rated (ESL ~0.7 nH each → parallel ESL = 0.088 nH)
  ESR parallel = ~2 mΩ → meets 10 mΩ well below SRF
  SRF of array: 1/(2π×√(0.088n×80µ)) ≈ 1.9 MHz
  Distributed in clusters of 2 caps near each VCCINT via group on FPGA.

Tier 3 — High-frequency (10 MHz to 200 MHz):
  Xilinx UG475 recommends 1× 100 nF per VCCINT pin (Artix-7 has 12 VCCINT balls).
  12× 100 nF 0402 X7R, 10 V rated.
  ESL single: 0.7 nH → 12 in parallel: 0.058 nH
  ESR parallel: ~0.33 mΩ → well below 10 mΩ
  SRF array: 1/(2π×√(0.058n×1.2µ)) ≈ 19 MHz
  Placed within 1-2 mm of each VCCINT via, on same side as FPGA.

Tier 4 — Very high frequency (200 MHz to 1 GHz):
  Under-BGA 10 nF 0201 C0G caps (via-in-pad).
  Target: 4-6 caps for the VCCINT ball group.
  ESL ~0.15 nH each → 5 in parallel: 0.03 nH
  SRF: 1/(2π×√(0.03n×50n)) ≈ 130 MHz
  Above SRF becomes inductive; rely on package capacitance above 500 MHz.
```

**Step 3 — Anti-resonance analysis between Tier 2 and Tier 3:**

```
Tier 2 array SRF: 1.9 MHz
Tier 3 array SRF: 19 MHz

Between 1.9 MHz and 19 MHz, Tier 2 is inductive and Tier 3 is capacitive.
They form a parallel LC resonance (anti-resonance peak).
Peak frequency ≈ √(1.9 × 19) ≈ 6 MHz

Estimated peak impedance:
  Z_peak ≈ √(L_Tier2 / C_Tier3) = √(0.088n / 1.2µ) ≈ 8.6 mΩ
  (This is actually below the 10 mΩ target — just barely acceptable)

If the peak exceeds Z_target, mitigations:
  a) Add a 1-10 mΩ resistor in series with one Tier 2 cap to damp the resonance.
  b) Move Tier 2 SRF upward by reducing inductance (better placement, 0201 packages).
  c) Add a "bridging" tier of 1 µF 0402 MLCCs to fill the gap.
```

**Step 4 — VCCO_0 and VCCAUX (simpler — higher Z_target):**

```
VCCO_0 (3.3 V, 500 mA):
  Xilinx recommends: 1× 10 µF per bank + 1× 100 nF per VCCO pin.
  Z_target = 330 mΩ → easily met with 4× 100 nF and 1× 10 µF.

VCCAUX (1.8 V, 200 mA):
  1× 10 µF + 4× 100 nF per power domain.
  Shared LDO output cap (polymer, 10 µF) counts as Tier 1.
  VCCAUX is used by SERDES and config logic — some sensitivity to noise.
  Add 1× 10 nF 0201 C0G close to each VCCAUX pin if SERDES are active.
```

**Step 5 — Layout requirements:**

```
VCCINT ball group: place Tier 3 caps within 1 mm of each ball.
                   Tier 2 caps within 5 mm.
                   Tier 1 polymer caps within 10 mm.
Each cap: GND via adjacent to cap pad — not shared long GND trace.
Use dedicated VCCINT plane poured on an inner layer directly under the FPGA.
Minimise number of vias between plane and ball: use micro-vias for high density.
```

---

### Question A2
**You observe a 120 mV peak-to-peak ripple on a 3.3 V rail at 1 MHz (the switching regulator frequency). The tolerance is ±50 mV. You have already placed 4× 10 µF MLCC and 8× 100 nF MLCC capacitors at the load. Describe a systematic methodology to identify whether the problem is insufficient capacitance, excessive ESR, a layout issue, or a regulator compensation issue.**

**Answer:**

**Step 1 — Identify the ripple character:**

```
With oscilloscope (10 MHz BW, AC coupled, ground clip very close):
  Is the ripple frequency exactly the switching frequency (1 MHz)? → switching ripple
  Is there an envelope modulation at a lower frequency (10-100 kHz)? → loop instability
  Is the ripple on all rails simultaneously? → common-mode, likely layout/ground issue
  Does ripple change with load current? → regulator compensation / current mode

120 mV at 1 MHz, no envelope modulation → switching ripple, not instability.
```

**Step 2 — Estimate expected ripple from output capacitance:**

```
For a buck converter, output voltage ripple:
  ΔV_C = ΔIL / (8 × fsw × C_out)  (CCM, assumes capacitance-limited ripple)
  ΔV_ESR = ΔIL × ESR_total         (ESR-limited ripple)

Total capacitance: 4×10µF (effective at 1 MHz?) + 8×100nF = ~42 µF
But: 10 µF MLCCs — check SRF. If ESL=0.7 nH: SRF = 0.6 MHz.
At 1 MHz the 10 µF caps are above their SRF → they are inductive at 1 MHz!
Effective capacitance at 1 MHz comes only from the 100 nF caps: 8×100 nF = 800 nF.

ΔV_C = ΔIL / (8 × 1 MHz × 800 nF) = ΔIL × 0.156 Ω

For ΔIL = 500 mA: ΔV_C = 78 mV  (already close to 50 mV budget, before ESR)
```

**Step 3 — Measure ESR contribution:**

```
Probe at the capacitor pads directly (not at the IC, to exclude trace inductance):
  If ripple at capacitor pads << ripple at IC pins → trace inductance is the cause
  If ripple at capacitor pads ≈ ripple at IC pins → ESR or insufficient C

ESR contribution: ΔV_ESR = ΔIL × ESR_total
  8× 100 nF MLCCs in parallel: ESR_total ≈ 4 mΩ / 8 = 0.5 mΩ → negligible
  Check if any series ferrite bead or trace resistance adds ESR
```

**Step 4 — Differentiate layout vs component issue:**

```
Replace scope probe ground clip with a direct SMD tip probe at the IC pin:
  If ripple drops significantly → the scope ground lead was acting as an antenna
    (false reading). Actual ripple may be acceptable.
  If ripple remains high → genuine PDN issue.

Measure ripple right at regulator output vs at IC:
  If regulator output is clean (< 20 mV) but IC sees 120 mV → trace inductance
  in the PCB path is responsible. Voltage droop = L × dI/dt.

Calculate trace inductance:
  120 mV droop, ΔI = 500 mA, rise time = 20 ns (1 MHz switching edge):
  L = V × dt / dI = 0.120 × 20n / 0.5 = 4.8 nH → ~5 cm of trace inductance
```

**Step 5 — Fixes based on diagnosis:**

```
If capacitor SRF is the issue:
  Add 4× 1 µF 0402 MLCC (SRF ~5 MHz for 0402) to bridge between 10 µF and 100 nF tiers
  This covers 1 MHz effectively.

If trace inductance is the issue:
  Re-route to shorten the path from regulator output cap to load caps.
  Widen the supply trace or use a copper pour on the signal layer.
  Move the load caps closer to the IC.

If regulator output ripple is excessive (> 20 mV at regulator):
  Increase output inductor value (reduces ΔIL).
  Increase output capacitance at the regulator output (not at the load).
  Verify compensation network matches the output capacitor ESR.
  Increase switching frequency (if adjustable) to shift ripple out of concern band.
```
