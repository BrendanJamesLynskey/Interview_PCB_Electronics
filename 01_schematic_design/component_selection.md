# Component Selection

## Prerequisites
- Basic passive component theory (capacitance, resistance, inductance)
- Voltage, current, and power relationships (Ohm's law, P = IV)
- MOSFET operating regions (cutoff, linear/triode, saturation)

---

## Concept Reference

### Component Derating

Derating is the deliberate reduction of a component's applied stress (voltage, current, temperature, power) below its rated maximum to improve reliability and extend service life. The failure rate of semiconductor devices and passive components rises steeply as stresses approach rated limits.

**Why components fail near their rated limits:**

- Electromigration in metal interconnects is exponentially dependent on current density and temperature.
- Dielectric breakdown in capacitors follows an inverse power law with applied voltage.
- Junction temperature rise in semiconductors accelerates diffusion-driven wear mechanisms.

**Typical derating guidelines (MIL-STD-975 and industry practice):**

```
Component type          Stress parameter       Typical derate to
------------------      -----------------      ------------------
Resistors               Power dissipation       50% of rated power
                        Voltage                 70% of rated voltage
Ceramic capacitors      DC voltage              50% of rated voltage
Tantalum capacitors     DC voltage              50% of rated voltage (safety-critical)
                                                60-70% for less critical apps
Electrolytic caps       Ripple current          80% of rated
                        Voltage                 80% of rated
MOSFETs                 VDS                     80% of BVDSS
                        ID                      75% of rated continuous
                        Power dissipation       75% considering junction temp
Diodes / TVS            Reverse voltage         80% of VRRM or VRWM
                        Forward current         75% of rated
IC supply pins          VCC                     Within absolute max, but
                        (follow datasheet)      allow headroom for transients
```

**Key interview point — temperature derating:** Most component ratings are specified at 25 °C. At elevated temperatures, maximum continuous current or power must be reduced. The derating factor is typically linear from the rated temperature up to the maximum junction temperature.

```
P_allowed(T) = P_max × (T_jmax - T_ambient) / (T_jmax - 25°C)

Example: A MOSFET rated 60 W at 25°C, Tjmax = 150°C, ambient = 70°C:
  P_allowed = 60 × (150 - 70) / (150 - 25) = 60 × 80/125 = 38.4 W
```

**Common mistake:** Applying a single derating factor without accounting for worst-case temperature. A component that looks correctly derated at 25 °C may be over-stressed at the system's maximum operating temperature.

---

### Capacitor Dielectric Types

#### MLCC (Multilayer Ceramic Capacitor) — Dielectric Classes

**Class I dielectrics — Temperature Compensating (C0G / NP0):**

```
EIA code   Temperature coefficient   Tolerance of TC    Typical capacitance range
--------   ----------------------   ---------------    -------------------------
C0G        0 ± 30 ppm/°C            ± 30 ppm/°C        1 pF to ~100 nF
NP0        (same as C0G, older name)
```

- Capacitance changes less than ±0.3% from -55 °C to +125 °C.
- No piezoelectric effect (no acoustic noise, no microphonics).
- No DC bias effect — capacitance does not drop with applied voltage.
- No ageing — capacitance is stable over time.
- Use for: oscillator timing circuits, filter networks, precision analogue, RF tuning.
- Limitation: low volumetric efficiency — a 100 nF C0G is much larger than 100 nF X7R.

**Class II dielectrics — High permittivity:**

```
EIA code   Operating temp range      Cap change over temp   Notes
--------   --------------------      --------------------   ----------------------
X5R        -55°C to +85°C           ±15%                   Consumer applications
X7R        -55°C to +125°C          ±15%                   Industrial/automotive standard
X8R        -55°C to +150°C          ±15%                   High-temp automotive
Y5V        -30°C to +85°C           +22/-82%               Avoid for precision work
Z5U        +10°C to +85°C           +22/-56%               Avoid for precision work
```

EIA code breakdown: Letter = lower temp limit, number = upper temp limit, letter = max capacitance change.

**Critical X7R characteristic — DC bias effect (voltage coefficient):**

```
A 10 µF 10 V X7R MLCC may have actual capacitance as follows:
  At 0 V bias:    10 µF (nominal)
  At 3.3 V bias:  ~7 µF  (30% loss)
  At 5 V bias:    ~4 µF  (60% loss)
  At 9 V bias:    ~1 µF  (90% loss!)
```

This is the single most commonly overlooked characteristic in digital power design. A decoupling capacitor rated at 10 µF but operating at 3.3 V on a 10 V capacitor provides significantly less charge reservoir than its label implies. The correct approach is to either:
1. Use a capacitor rated at 2× or more the operating voltage (e.g., 10 µF 25 V for a 3.3 V rail).
2. Use C0G/NP0 for values where it is practical.
3. Account for effective capacitance from vendor SPICE models or DC bias curves in the datasheet.

**Ageing in Class II MLCCs:** Barium titanate dielectric exhibits logarithmic capacitance loss after the last thermal cycle above the Curie point (~125 °C). Approximately 1-3% capacitance loss per decade of time from manufacture. Capacitors that have been through reflow soldering reset their ageing clock. This matters for precision timing or filter circuits with tight tolerance requirements.

**Piezoelectric effect in Class II MLCCs:**
- Mechanical vibration generates a small voltage (microphone effect).
- AC voltages generate mechanical stress and acoustic noise (speaker effect at switching frequencies).
- Sensitive analogue circuits should use C0G where practical, or locate X7R away from sensitive nodes.
- Audible whine from switching converters running in the audio band (20 Hz to 20 kHz) is often caused by X7R MLCCs resonating mechanically.

#### Tantalum Capacitors

```
Characteristic    Typical value        Notes
--------------    -------------        -------------------------------------------
Capacitance range  1 µF to 1000 µF    Higher volumetric density than MLCC for >10 µF
Voltage range      2.5 V to 50 V      Derate to 50% max voltage for reliability
ESR                50-500 mΩ          Higher than ceramic; useful for stability in LDOs
Temperature range  -55°C to +125°C    Capacitance varies ~15% over range
Failure mode       Short-circuit       Critical safety consideration
```

**Tantalum failure mechanism — thermal runaway:**
Tantalum pentoxide dielectric can be punctured by voltage spikes, overvoltage, excessive ripple current, or reverse voltage. The result is a low-resistance short that generates heat, which accelerates further dielectric breakdown — thermal runaway. The component can ignite in extreme cases.

Mitigation:
- Derate voltage to 50% in all applications. Use 35 V rated for a 12 V rail.
- Never apply reverse voltage. Protect against transient reversal.
- Include a series resistor (1-10 Ω) at the input to limit surge current on power-up.
- Consider polymer tantalum or polymer electrolytic as safer alternatives.

#### Polymer Electrolytic Capacitors

```
Type              ESR            Failure mode    Voltage range
-----------       --------       ------------    -------------
Wet electrolytic  High (>100 mΩ) Electrolyte dry Moderate voltage
Polymer Al        5-50 mΩ        Open-circuit    Low-medium voltage
Polymer Ta        10-100 mΩ      Open-circuit    Low-medium voltage
```

Polymer capacitors use a conductive polymer (polyethylene dioxythiophene, PEDOT) instead of liquid electrolyte. They offer:
- Very low ESR — often 5-50 mΩ vs hundreds of mΩ for wet electrolytic.
- Fail open rather than short (safer than wet tantalum).
- Extended lifetime at high temperature (no electrolyte evaporation).
- Better high-frequency performance than wet types.
- Used extensively as output capacitors on high-current switching regulators.

---

### MOSFET Selection for Power Switching

The four key parameters for a power switching MOSFET are: BVDSS (breakdown voltage), RDS(on) (on-resistance), QG/Qg (total gate charge), and thermal resistance.

**BVDSS selection:**
Select BVDSS ≥ (maximum VDS including transients) × derating factor (typically 1.25×).

```
Example: 12 V bus, inductive load, expected spike to 18 V:
  Required BVDSS ≥ 18 × 1.25 = 22.5 V → select 30 V or 40 V rated device
```

There is a fundamental trade-off: higher BVDSS devices have higher RDS(on) for the same die area (silicon on-resistance scales approximately as BVDSS^2.5 for vertical devices).

**RDS(on) and conduction loss:**

```
P_conduction = I_rms² × RDS(on)

Example: 5 A RMS, RDS(on) = 10 mΩ:
  P_conduction = 25 × 0.010 = 250 mW
```

RDS(on) increases with junction temperature (typically +0.4% to +0.7% per °C), so thermal calculations must iterate: more loss → higher temperature → higher RDS(on) → more loss.

**Gate charge and switching loss:**

Total gate charge Qg determines how much charge the gate driver must supply to fully enhance the MOSFET. Switching loss is:

```
P_switching = 0.5 × VDS × ID × (tr + tf) × fsw
            ≈ VDS × ID × QG × fsw / IG_avg

Where:
  tr, tf  = rise and fall times (function of Qg and driver current)
  fsw     = switching frequency
  IG_avg  = average gate current from driver

For a 100 kHz converter, 48 V, 5 A, Qg = 50 nC, driver = 1 A:
  tr ≈ Qg / IG = 50 nC / 1 A = 50 ns
  P_sw ≈ 0.5 × 48 × 5 × 2 × 50 ns × 100 kHz = 1.2 W
```

**Key MOSFET figure of merit:** RDS(on) × Qgd (gate-drain charge). Lower is better. Newer GaN devices offer dramatically lower FOM than silicon.

```
Technology        Typical RDS(on) × Qgd
-----------       ---------------------
Si planar         5000-20000 mΩ·nC
Si trench         500-2000 mΩ·nC
Si SJ (superjunction) 50-500 mΩ·nC
GaN               5-50 mΩ·nC
```

**N-channel vs P-channel:**
- N-channel MOSFETs have lower RDS(on) for same die size (3× better electron vs hole mobility).
- P-channel is convenient for high-side switching (gate-source referenced to source = output voltage) but costly in terms of performance per dollar.
- N-channel high-side switching requires a bootstrap or charge pump circuit to drive VGS above VDS.

**Thermal selection — calculating TjMax:**

```
T_j = T_ambient + P_total × (Rθjc + Rθcs + Rθsa)

Where:
  Rθjc = junction-to-case thermal resistance (from datasheet)
  Rθcs = case-to-heatsink (thermal interface material, ~0.1-0.5 °C/W)
  Rθsa = heatsink-to-ambient (heatsink selection)

Target: T_j < 125°C for silicon (absolute max typically 150-175°C,
        but derate to 125°C for reliability margin)
```

---

## Tier 1 — Fundamentals

### Question F1
**A designer specifies a 10 µF 0402 X7R MLCC for 3.3 V decoupling. Why might this not provide 10 µF of effective capacitance in circuit, and what is the correct fix?**

**Answer:**

Three effects reduce effective capacitance below the nameplate value:

1. **DC bias (voltage coefficient):** X7R MLCCs exhibit significant capacitance reduction with DC bias. A 10 µF 6.3 V X7R at 3.3 V bias may measure only 5-6 µF effective capacitance — a 40-50% loss. This is the dominant effect and is frequently missed by designers who do not consult the DC bias curve in the datasheet.

2. **Temperature coefficient:** X7R capacitance can vary ±15% over the operating temperature range (-55°C to +125°C).

3. **Ageing:** Class II MLCC dielectrics lose capacitance logarithmically with time after the last high-temperature thermal cycle.

**Correct fixes:**
- Use a 10 µF MLCC rated at 10 V or 16 V instead of 6.3 V. The voltage coefficient curve is much flatter at 3.3 V/16 V = 21% rated voltage vs 3.3 V/6.3 V = 52% rated voltage.
- Alternatively, use two 10 µF 6.3 V capacitors in parallel to compensate for the reduction, but this is less efficient.
- For precision timing or critical filtering, use a C0G (NP0) capacitor where the value is ≤ ~100 nF.

**Common mistake:** Relying on the capacitor body marking only. Always check the DC bias characteristic curve in the datasheet, not just the nominal value.

---

### Question F2
**Name the main failure mode of a wet tantalum capacitor. What design practices mitigate this risk?**

**Answer:**

The main failure mode is **short-circuit with potential thermal runaway and ignition.** The tantalum pentoxide (Ta₂O₅) dielectric is very thin and can be punctured by:
- Overvoltage (exceeding rated voltage)
- Voltage spikes and transients
- Excessive ripple current causing localised heating
- Reverse voltage (even brief reverse polarity destroys the dielectric)

Once punctured, the low-resistance short causes increased current, joule heating, further dielectric breakdown, and in severe cases combustion of the tantalum powder.

**Mitigation practices:**
1. **Voltage derating:** Apply no more than 50% of rated voltage in continuous use. For a 5 V rail, use a 10 V or 16 V rated tantalum.
2. **Series resistance:** Include a 0.5-10 Ω series resistor to limit inrush current at power-up. Power supplies have low output impedance at startup; capacitor inrush can cause dielectric stress.
3. **Polarity protection:** Never allow reverse voltage. Ensure PCB assembly orientation is unambiguous (polarity marked on silkscreen and courtyard).
4. **Consider polymer tantalum:** Fails open rather than short, eliminating the fire risk.
5. **Soft-start:** Ramp input voltage slowly when possible to reduce inrush stress.

---

### Question F3
**Define component derating. Why is it standard practice in aerospace and high-reliability electronics, and what are the trade-offs?**

**Answer:**

**Definition:** Derating is operating a component at a fraction of its maximum rated electrical, thermal, or mechanical stress to improve long-term reliability and reduce the probability of failure.

**Why it is standard practice:**

Failure rates of electronic components follow the Arrhenius equation for temperature-driven mechanisms and are proportional to stress levels for electrical mechanisms. Operating at 80% of rated voltage or power does not just provide a small safety margin — it can reduce failure rates by an order of magnitude. In aerospace (MIL-STD-975, ECSS-Q-ST-30-11), automotive (AEC-Q200), and medical applications, where field failure is not acceptable, derating is mandated.

For example, a ceramic capacitor operated at 50% of its rated voltage has a lifetime orders of magnitude longer than one at 90% rated voltage, because dielectric breakdown probability decreases steeply with reduced electric field.

**Trade-offs:**
- **Cost and size:** A capacitor or MOSFET derated to 50% voltage must be rated at 2× the operating voltage, increasing component cost and, for capacitors, physical size.
- **Weight:** In aerospace, this directly impacts vehicle mass.
- **Higher-rated components may have worse electrical characteristics:** A 100 V capacitor for a 48 V rail is larger and may have higher ESR than a 63 V capacitor used at its limit.

The design process balances the reliability requirement against cost and size constraints, and the derating factor is chosen based on mission criticality.

---

### Question F4
**You need a 100 nF decoupling capacitor on a 3.3 V I/O pin. Your PCB size forces you to use 0402. Should you use C0G or X7R? Justify your answer.**

**Answer:**

For a general 3.3 V digital I/O decoupling application, **X7R is the appropriate choice.** Here is the reasoning:

- A 100 nF C0G capacitor in 0402 size exists but is near the practical limit of C0G volumetric efficiency. It is more expensive (5-10× cost) and may have limited vendor availability.
- The DC bias voltage coefficient issue is negligible at 100 nF because the capacitor would be rated at 10 V or 16 V for a 3.3 V application — far below the voltage where the X7R curve shows significant derating.
- The ±15% temperature variation of X7R over the operating range is acceptable for digital decoupling, where the goal is charge reservoir, not precision filtering.
- A 100 nF X7R 0402 10 V is a commodity part available from multiple vendors at very low cost.

**When C0G would be the right choice at 100 nF:**
- Oscillator load capacitors (timing precision required, no capacitance variation allowed).
- RF matching networks and filters where the response centre frequency must be stable with temperature.
- Precision analogue filter poles where capacitance tolerance matters.

---

## Tier 2 — Intermediate

### Question I1
**Compare the switching loss and conduction loss mechanisms in a MOSFET used as a power switch. How does the choice of switching frequency affect which loss mechanism dominates, and how do you select a MOSFET accordingly?**

**Answer:**

**Conduction loss** occurs when the MOSFET is fully on (VGS >> VGS(th)). The channel behaves as a resistor:

```
P_cond = I_rms² × RDS(on)
```

This loss is independent of switching frequency. It depends only on current and on-resistance. To minimise it, select a MOSFET with the lowest possible RDS(on), which generally means a larger die area.

**Switching loss** occurs during transitions between on and off states. During transitions, both VDS and ID are simultaneously non-zero, creating instantaneous power dissipation:

```
P_sw = 0.5 × VDS × ID × (tr + tf) × fsw
```

Transition times (tr, tf) are controlled by the gate driver's ability to charge and discharge the gate capacitance, particularly the gate-drain (Miller) capacitance. A MOSFET with large Qgd has longer switching transitions and thus higher switching loss.

```
P_sw ∝ Qgd × fsw
```

**Effect of switching frequency on loss balance:**

```
fsw = 10 kHz:  Switching loss is small; minimise RDS(on) (use large die MOSFET)
fsw = 100 kHz: Both losses significant; use figure of merit RDS(on) × Qgd
fsw = 1 MHz:   Switching loss dominates; minimise Qgd (use small, fast MOSFET)
```

At very high frequencies (MHz range), a device with very low RDS(on) but large gate charge may perform worse than a device with higher RDS(on) but minimal gate charge.

**Selection procedure:**

1. Calculate maximum allowable total power dissipation from thermal model (Tjmax, heatsink).
2. Split budget between conduction and switching loss (often 50/50 as a starting point).
3. From conduction budget: max RDS(on) = P_cond / I_rms².
4. From switching budget: max Qgd = P_sw / (VDS × ID × fsw).
5. Find candidate MOSFETs satisfying both constraints.
6. Verify BVDSS with derating margin.
7. Verify gate drive current can fully enhance the device (VGS must reach 10 V for standard threshold, or 4.5-5 V for logic-level devices driven from 3.3 V/5 V logic).

---

### Question I2
**A 10 µF X7R MLCC is used as a bulk bypass capacitor on a 1.8 V DDR memory power rail. The datasheet shows 3 mΩ ESR and 0.5 nH ESL. At 100 MHz (approximately the DDR switching frequency harmonic), what is the impedance of this capacitor? Is it behaving as a capacitor, an inductor, or resistor at this frequency?**

**Answer:**

The impedance of a real capacitor is modelled as a series RLC circuit:

```
Z(f) = √(R² + (X_L - X_C)²)

Where:
  R   = ESR = 3 mΩ = 0.003 Ω
  X_L = 2π × f × ESL = 2π × 100×10⁶ × 0.5×10⁻⁹ = 0.314 Ω
  X_C = 1 / (2π × f × C) = 1 / (2π × 100×10⁶ × 10×10⁻⁶) = 0.000159 Ω
```

At 100 MHz, XL >> XC, so:

```
Z ≈ √(R² + X_L²) ≈ X_L ≈ 0.314 Ω
```

The capacitor is behaving as an **inductor** at 100 MHz. The 10 µF capacitance contributes negligibly; only the package inductance (ESL) matters.

**Self-resonant frequency (SRF):**

```
f_SRF = 1 / (2π × √(L × C)) = 1 / (2π × √(0.5×10⁻⁹ × 10×10⁻⁶))
       = 1 / (2π × √(5×10⁻¹⁵))
       = 1 / (2π × 7.07×10⁻⁸) ≈ 2.25 MHz
```

Below 2.25 MHz this capacitor is capacitive. Above 2.25 MHz it is inductive. At 100 MHz it presents ~314 mΩ of inductive impedance.

**Design implication:** For 100 MHz decoupling, smaller capacitors with lower ESL (smaller package, fewer turns of MLCC stack) are needed. A 100 pF 0402 C0G has ESL ~0.3 nH and SRF near 1 GHz — it provides low impedance at 100 MHz where the 10 µF bulk cap fails.

This is why decoupling strategies use multiple capacitor values in parallel: each handles a different frequency decade.

---

### Question I3
**What is the difference between a logic-level MOSFET and a standard-threshold MOSFET? When would you use each?**

**Answer:**

The distinction is in the gate threshold voltage (VGS(th)) and the gate voltage required for full enhancement:

```
Type                VGS(th) range    Full enhancement at    Typical VGS drive
-----------------   -------------    -------------------    -----------------
Standard threshold  2-4 V            VGS = 10 V             10-15 V gate drive
Logic-level         0.5-2 V          VGS = 4.5 V or 3.3 V  3.3 V or 5 V drive
```

**Standard-threshold MOSFET:**
- RDS(on) is specified at VGS = 10 V. The device is not fully on at VGS = 3.3 V or 5 V.
- Requires a dedicated gate driver with bootstrap or charge pump for high-side switching.
- Preferred in high-power applications (>20 W) where driver circuitry is already present.
- Generally offers lower RDS(on) for the same die size because the gate oxide is thicker (more robust).

**Logic-level MOSFET:**
- RDS(on) is specified at VGS = 4.5 V (or 2.5 V for ultra-logic-level).
- Can be driven directly from a microcontroller or 3.3 V/5 V digital logic output pin.
- Convenient for small load switching without dedicated gate drivers.
- The thinner gate oxide is more susceptible to damage if VGS exceeds 10-15 V (check absolute maximum).

**Selection rule:**
- MCU GPIO directly drives the gate: use logic-level (VGS(th) < 2.5 V for 3.3 V systems).
- Gate driver IC or half-bridge driver present: standard-threshold is safe and often preferred for better RDS(on) specification.
- Be cautious with partially enhanced logic-level MOSFETs: if VGS only reaches VGS(th) but not full enhancement, RDS(on) is much higher than the datasheet value, causing excess conduction loss and possible thermal failure.

---

## Tier 3 — Advanced

### Question A1
**A synchronous buck converter operates at 500 kHz with 12 V input, 1.8 V output, 10 A maximum load. Select appropriate high-side and low-side MOSFETs. Justify your selection criteria and calculate the expected power loss for each device.**

**Answer:**

**Step 1 — Duty cycle and current waveforms:**

```
D = Vout / Vin = 1.8 / 12 = 0.15  (15%)

Assume inductor ripple current ΔIL = 30% of load = 3 A peak-to-peak

High-side MOSFET (conducts for D = 15% of period):
  I_peak = 10 + 1.5 = 11.5 A
  I_valley = 10 - 1.5 = 8.5 A
  I_rms(HS) ≈ IL_avg × √D = 10 × √0.15 = 3.87 A

Low-side MOSFET (conducts for 1-D = 85% of period):
  I_rms(LS) ≈ IL_avg × √(1-D) = 10 × √0.85 = 9.22 A
```

**Step 2 — MOSFET requirements:**

```
High-side (HS):
  BVDSS: 12 V × 1.25 = 15 V minimum → select 20-30 V
  Key metric: Qgd (switching loss dominates at D=15%, low conduction time)
  Target: Qgd < 5 nC, RDS(on) < 20 mΩ at VGS = 10 V

Low-side (LS):
  BVDSS: same as HS = 20-30 V
  Key metric: RDS(on) (conduction loss dominates at 1-D=85%)
  Target: RDS(on) < 5 mΩ, Qgd < 15 nC acceptable
  Body diode: must handle dead-time conduction; check reverse recovery charge Qrr
```

**Step 3 — Example device selection:**

```
HS: Infineon BSZ016N04LSI (30 V, RDS(on) = 1.6 mΩ, Qg = 37 nC, Qgd = 8 nC)
    Or: consider GaN (e.g., EPC2036, 100 V, 1.4 mΩ, Qg = 5.4 nC, Qgd = 0.9 nC)

LS: Infineon BSZ040N03MS (30 V, RDS(on) = 4.0 mΩ, Qg = 52 nC)
    Criterion met: low RDS(on), acceptable Qgd for lower-speed LS transitions
```

**Step 4 — Loss calculation (Si example):**

```
High-side conduction loss:
  P_cond_HS = I_rms_HS² × RDS(on) × (1 + α × ΔTj)
             = 3.87² × 0.0016 × 1.3  (estimate 30% increase for Tj=90°C)
             = 14.98 × 0.0016 × 1.3 = 31 mW

High-side switching loss:
  Gate driver: 1 A source/sink
  tr ≈ Qgd / IG = 8 nC / 1 A = 8 ns
  P_sw_HS = 0.5 × 12 × 11.5 × 8 ns × 500 kHz = 276 mW

  (Switching loss dominates for the HS device — validates our Qgd priority)

Low-side conduction loss:
  P_cond_LS = I_rms_LS² × RDS(on) × (1 + α × ΔTj)
            = 9.22² × 0.004 × 1.3
            = 85 × 0.004 × 1.3 = 443 mW

Low-side switching loss:
  Body diode reverse recovery and hard switching losses are small for LS
  in a synchronous topology. Estimate Qrr loss:
  P_rr ≈ 0.5 × Qrr × VDS × fsw = 0.5 × 20 nC × 12 × 500 kHz = 60 mW

Total estimated losses: ~810 mW
```

**Step 5 — Thermal verification:**

```
T_j (HS) = T_amb + P_HS × Rθja ≈ 25 + 0.307 × 40 = 37.3°C (well within limit)
T_j (LS) = T_amb + P_LS × Rθja ≈ 25 + 0.503 × 35 = 42.6°C (well within limit)

(Rθja for small QFN packages: 30-50°C/W depending on copper pour area)
```

**Common interview pitfall:** Calculating RDS(on) loss at 25°C datasheet conditions without accounting for RDS(on) temperature coefficient, then being surprised by higher-than-expected losses in the actual design.

---

### Question A2
**Explain the Miller plateau in MOSFET gate charge characteristics. How does it affect the design of a gate driver circuit for a high-frequency converter?**

**Answer:**

When a MOSFET turns on into an inductive load (which maintains a nearly constant drain current), the gate charging process passes through three distinct regions:

```
Gate voltage vs. gate charge:

Vgs
 |                    _______________  VGS_final
 |               ____/
 |         _____|  <-- Miller plateau
 |    ____/
 |___/
 |____________________________ Qg
    ^     ^          ^
    |     |          Qg (total)
    |     Qgs2 (plateau start)
    Qgs1 (threshold)

Regions:
  0 → Qgs1: Gate capacitance Ciss charges; VGS rises to VGS(th)
  Qgs1 → Qgs2: VGS rises from VGS(th) to Miller plateau voltage;
                drain current rises, drain voltage begins to fall
  Qgs2 → Qgd: Miller plateau — VGS constant while Cgd (Miller cap) discharges;
               VDS falls; energy is transferred to/from inductor
               This region has zero dVGS/dt despite gate current flowing
  Qgd → Qg:   VGS rises to final gate drive voltage; device fully enhanced
```

**Physical explanation of Miller plateau:**

During the plateau, the Miller effect multiplies Cgd by the voltage gain of the switching transition. The effective capacitance seen at the gate is:

```
Ceff = Cgd × (1 + |dVDS/dVGS|) = Cgd × voltage amplification

Since dVDS/dVGS during the transition can be 5-20× for typical power MOSFETs,
the effective input capacitance during the plateau is 5-20× higher than Ciss.
This slows the gate voltage transition even though gate current is flowing.
```

**Gate driver design implications:**

1. **Driver current rating:** The plateau duration determines the switching loss. To shorten the plateau:
   ```
   t_plateau = Qgd / IG_driver
   IG_driver = (V_driver - V_plateau) / RG_total

   Where RG_total = internal driver impedance + external gate resistor
   ```
   Increasing driver current (or reducing gate resistance) shortens the plateau and reduces switching loss.

2. **Separate gate resistors for turn-on and turn-off:** A smaller gate resistor speeds switching (lower loss) but increases dV/dt and dI/dt, generating EMI and potentially causing ringing. Use a turn-on resistor and a parallel diode + smaller turn-off resistor to optimise independently:
   ```
   Faster turn-off: reduces body diode conduction time in synchronous rectifier
   Controlled turn-on: limits dV/dt to reduce EMI
   ```

3. **Miller clamp:** In a synchronous converter, the low-side MOSFET can be spuriously turned on by capacitive coupling through Cgd when the high-side switches (VDS ramps rapidly). A Miller clamp holds VGS low by sinking any Miller-coupled current to ground through a low-impedance path, preventing false turn-on.

4. **Dead-time optimisation:** The gate driver must provide sufficient dead-time between HS turn-off and LS turn-on to prevent shoot-through. Too long a dead-time causes body diode conduction loss; too short causes shoot-through. The minimum dead-time is determined by the plateau duration plus propagation delay.
