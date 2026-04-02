# Design for Test

## Prerequisites
- PCB fabrication basics: layer stackup, via types, drill tolerances
- JTAG / IEEE 1149.1 protocol fundamentals
- Digital logic states and tri-state buffers
- Basic manufacturing test concepts: continuity, shorts, functional testing

---

## Concept Reference

### Why Design for Test Matters

A PCB that cannot be efficiently tested is a liability throughout the product lifecycle. The cost of detecting a defect rises by roughly an order of magnitude at each stage of the manufacturing and deployment chain:

```
Stage                      Relative cost to detect and repair a defect
-----                      -------------------------------------------
PCB bare board inspection  1×   (AOI / electrical test at fab)
SMT assembly (pre-power)   10×  (AXI, flying probe, bed-of-nails)
Board power-on test        100× (functional testing, ICT)
System integration         1000×
Field return               10,000×
```

DFT practices shift defect detection as early as possible. The goal is to maximise fault coverage — the percentage of all possible manufacturing defects that can be detected by the test strategy — while minimising test time and fixture cost.

---

### Test Point Selection and Placement

A test point (TP) is a dedicated, accessible pad, via, or through-hole pin that provides a known, accessible node for probing by automated test equipment (ATE), flying probe, or manual debug.

**What to expose as test points:**

```
Priority 1 — Essential (always include):
  All power rails: VCC, VDD, VDDIO, and any regulated output
  All ground returns (at minimum, one GND TP per 10 cm² of board area)
  System reset lines (nRESET, POR)
  Programming/debug interfaces (JTAG, SWD, UART)
  Clock signals (XTAL, PLL output)

Priority 2 — High value:
  SPI/I2C bus lines (SCK, MOSI, MISO, SDA, SCL)
  CAN/RS-485 bus lines
  Key intermediate nodes in power conversion circuits
  Analogue reference voltages
  Interrupt lines from critical peripherals

Priority 3 — Useful when area permits:
  Individual I/O pin transitions for key signals
  Oscillator circuit nodes
  ADC input signals
```

**Test point physical specifications:**

```
Parameter                  Bed-of-nails minimum    Flying probe minimum
---------                  -------------------     -------------------
TP pad diameter            1.0 mm (0.040 in)       0.5 mm (0.020 in)
TP pad to TP pad pitch     2.54 mm grid preferred  0.75 mm minimum
Clearance to SMD           0.25 mm minimum         0.10 mm minimum
Through-hole TP diameter   1.0 mm drill minimum    N/A
SMD TP (no hole)           1.0 mm × 1.0 mm min     0.5 mm × 0.5 mm min
TP accessible side         Bottom preferred (ICT)   Either side
```

**Common DFT mistakes with test points:**
- Placing TPs under connectors or taller components where probes cannot reach.
- Using a via as a test point without ensuring the annular ring is exposed (no solder mask over via).
- Routing test points to nets that are buffered or isolated during test, making measurements unrepresentative.
- Insufficient ground test points — the probe return path matters as much as the signal probe.

---

### Bed-of-Nails Testing

A bed-of-nails (BoN) fixture is a custom test fixture with spring-loaded probes (Pogo pins) arranged to contact every test point on the PCB simultaneously. The PCB is pressed onto the fixture, all probes make contact, and the ATE can exercise every net in parallel.

**Advantages:**
- Very high throughput — entire board tested in one press cycle.
- Parallel access to all nodes simultaneously — no sequential scanning.
- Can perform in-circuit test (ICT): measures component values, continuity, shorts.

**Disadvantages:**
- Fixture cost: £5,000 to £50,000 per board design, amortised over production volume.
- Fixture lead time: 2-6 weeks.
- Not suited for prototyping or low-volume production.
- High-density fine-pitch boards require expensive micro-pogo pin fixtures.
- Components on both sides of the PCB require a dual-sided fixture (clamshell) or strategic TP placement on one side.

**ICT fault coverage using BoN:**

```
Defect type            ICT detection method          Typical coverage
-----------            --------------------          ----------------
Solder bridge (short)  Resistance between nodes      > 99%
Missing component      Opens test (drive/sense)      > 95%
Wrong value component  In-circuit measurement        90-95% (passive)
Wrong orientation      Functional + I/V curve        80-90%
Missing solder (open)  Continuity test               > 95%
Wrong device pinout    Functional test vectors       Varies by IC
```

**BoN layout requirements:**
- All test nodes must be on the same side (unless clamshell fixture).
- TP pads must be on a 100 mil (2.54 mm) grid where possible — non-grid placement increases fixture cost.
- Keep-out zones: mark areas where component height exceeds probe clearance.
- Panel registration: fiducials and tooling holes are used by the fixture to align the board.

---

### Flying Probe Testing

A flying probe tester uses two to eight individually programmable probes mounted on servo axes that move to contact test points sequentially, without a custom fixture.

**Advantages:**
- No fixture cost or lead time — reprogram in hours for a new board.
- Suited for prototyping, NPI (new product introduction), low-volume production.
- Can reach finer-pitch test points than BoN (probe tip as small as 100 µm).
- Both sides of PCB accessible without a dual fixture.

**Disadvantages:**
- Slow: sequential probe movements — a complex board may take 2-10 minutes per unit.
- Cannot simultaneously access all nodes (no ICT for components needing driven stimulus).
- Limited to continuity, opens, shorts, and some passive measurement.
- Not cost-effective above ~1,000 units/month (BoN fixture amortisation pays off).

**DFT for flying probe:**
- TPs still needed but can be smaller (0.5 mm SMD pad).
- Via-as-TP is more acceptable (no solder mask over vias reduces probe impedance).
- Expose both sides of the PCB to maximise accessible nodes.
- Flying probe cannot drive active components — functional test still requires a functional fixture or programming fixture.

---

### Boundary Scan (JTAG / IEEE 1149.1)

Boundary scan is a digital test methodology implemented within the IC itself. Each I/O pin of a JTAG-compliant device is surrounded by a boundary scan cell — a shift register stage that can capture the pin state or drive the pin to a known value under software control via the JTAG port.

**The JTAG TAP (Test Access Port) signals:**

```
Signal    Direction    Function
------    ---------    --------
TCK       Input        Test clock (shifts data through scan chain)
TMS       Input        Test Mode Select (state machine control)
TDI       Input        Test Data In (serial data into scan chain)
TDO       Output       Test Data Out (serial data out of scan chain)
nTRST     Input (opt)  Test Reset (asynchronous reset of TAP state machine)
```

**What boundary scan can test:**

```
Test type            Description                           Coverage
---------            -----------                           --------
EXTEST               Drive pin outputs, sample pin inputs  Board-level shorts/opens
INTEST               Test device's internal logic          Device-internal
SAMPLE/PRELOAD       Non-invasive capture of pin states    Observe live board
RUNBIST              Execute device's built-in self-test   Device-internal
Interconnect test    Check connections between two JTAG    Full shorts/opens on
                     devices (EXTEST on both)              JTAG-boundary nets
```

**Interconnect test procedure (two JTAG devices):**

```
Device A           Interconnect        Device B
--------           -----------         --------
JTAG output pin ──[trace]──[TVS?]──── JTAG input pin

1. Shift EXTEST instruction into Device A.
2. Preload a pattern (0b10101...) into boundary scan cells for Device A outputs.
3. Apply pattern: Device A drives its outputs.
4. Device B boundary scan captures its inputs.
5. Read back captured values via TDO.
6. Compare expected vs actual — any mismatch indicates a short or open on that net.

Detects: Opens (expected 1, captured 0), Shorts between adjacent nets, Wrong routing.
Does not detect: Resistive opens > a few kΩ, timing faults.
```

**JTAG chain design rules:**
- Chain all JTAG devices on the PCB in series (TDO of device N → TDI of device N+1).
- Terminate TMS and TCK with weak pull-up/pull-down as specified per device.
- Provide a dedicated JTAG header (minimum: TCK, TMS, TDI, TDO, GND; plus nTRST and VTREF if needed).
- Keep TCK trace length matched across devices if using a daisy chain at speeds > 10 MHz.
- For ARM Cortex-M devices, the SWD (Serial Wire Debug) interface uses only two wires (SWDIO, SWDCLK) and is preferred over full JTAG in space-constrained designs — but SWD does not support multi-device chains.

---

### In-Circuit Test (ICT) vs Functional Test

**ICT (In-Circuit Test):**
Measures individual component parameters (resistance, capacitance, inductance, semiconductor I/V curves) while the component is in circuit. Does not require the board to be powered by its own supply.

```
Checks:           Short circuits, open circuits, wrong component values,
                  missing components, reversed polarities
Cannot check:     Timing faults, speed-dependent behaviour, full digital functionality,
                  analogue performance (gain, bandwidth) without additional fixtures
Requires:         Bed-of-nails fixture or adequate test point coverage
```

**Functional Test (FT):**
Powers up the assembled board and exercises it through its normal operating interfaces. Software test vectors are applied, responses captured, and pass/fail criteria evaluated.

```
Checks:           All ICT faults that result in functional failure,
                  timing-dependent faults, software/firmware interaction,
                  power rail sequencing, analogue performance
Cannot check:     Latent defects that do not affect functionality
                  (a resistor at 2× its correct value may still pass)
Requires:         Power supply, programming interface, signal generators,
                  measurement instruments — more expensive test setup
```

**Combined strategy:**
Most production lines use ICT (or flying probe) for early defect detection, followed by functional test for the subset of boards that pass ICT. This minimises functional test time (faster, more expensive) by only presenting known-good-assembly boards.

---

## Tier 1 — Fundamentals

### Question F1
**Why should test points be placed on the bottom side of a PCB for bed-of-nails testing? What placement rules govern their geometry?**

**Answer:**

In-circuit test (ICT) with a bed-of-nails fixture presses the PCB onto an array of spring-loaded probes from below, with the board supported from above (or using a clamshell fixture for both sides). Placing all test points on the bottom side allows a single-sided fixture — cheaper, faster to build, and mechanically simpler.

If test points are on the top side, a more expensive clamshell fixture is required that simultaneously probes both sides.

**Placement rules:**

```
Grid:          Aim for 100 mil (2.54 mm) grid — off-grid positions increase
               fixture cost (special drill/probe positions are more expensive)
Pad size:      Minimum 1.0 mm diameter for standard BoN probes
Via TPs:       Must have solder mask opened — a via covered in solder mask
               cannot be reliably probed (probe tip slips, unreliable contact)
Clearance:     ≥ 0.25 mm from any nearby SMD pad or component body
               ≥ 1.5 mm from any tall component (connector, electrolytic cap)
               that would block probe access
Fiducials:     Include at least 3 PCB fiducials for fixture alignment
Coverage:      Every net should have at least one TP — aim for 90%+ net coverage
```

**Common mistake:** Treating TPs as optional extras and adding them after component placement is finalised. Correct practice: allocate TP positions during initial placement planning, before routing, to ensure adequate space exists.

---

### Question F2
**What is the purpose of the nTRST pin on a JTAG connector, and what happens if it is left floating?**

**Answer:**

`nTRST` is the asynchronous reset for the JTAG TAP (Test Access Port) state machine. When asserted (driven low), it immediately resets the TAP controller to the Test-Logic-Reset state, regardless of the current state or TCK activity.

**Why it matters:**
On power-up, the TCK, TMS, and TDI pins may receive undefined signals (bus contention, floating lines, capacitive coupling). The TAP state machine can reach an unknown state. Without `nTRST`, the only way to reset the TAP is to clock TMS high for five consecutive TCK cycles — which requires TCK to be active. If TCK is also floating at power-up, the TAP state machine may remain in an undefined state indefinitely, blocking JTAG access.

`nTRST` provides a reliable, clock-independent escape from this condition.

**If nTRST is left floating:**
The pin typically has an internal pull-up. It will be deasserted (high) unless explicitly driven low. This is safe in most cases — the TAP will not be spontaneously reset. However, in noisy environments, if `nTRST` capacitively couples to ground during power-on transients, the TAP may be held in reset unintentionally. Best practice: tie `nTRST` to VCC through a 10 kΩ pull-up resistor, with a test point for manual override.

---

### Question F3
**A board returns from assembly and does not power up. You have no functional test fixture available. What test point measurements would you make first to diagnose the fault?**

**Answer:**

Using a multimeter and oscilloscope, in order:

**1. Resistance check before applying power (board unpowered):**
```
Measure VCC to GND resistance:
  > 100 kΩ: normal (capacitors charged, no shorts)
  < 10 Ω:   likely solder bridge on power rail — do not apply power
  0 Ω:      definite short — must find and remove before powering up
```

**2. Apply power with current limit set to 10% of expected nominal current:**
```
Observe current draw:
  0 mA: open circuit in supply path (blown fuse, missing inductor, bad connector)
  Nominal: power supply is working, continue to step 3
  High current (> expected): short exists downstream
```

**3. Check primary supply rail voltage at the power input TP:**
```
If present (±5% of nominal): input power OK, fault is downstream
If absent or low: fault is in the power input path (fuse, connector, reverse-protection diode)
```

**4. Walk the power tree from input to each downstream rail:**
```
For each regulated rail in sequence (following the power tree):
  Measure voltage at the regulator output TP.
  If absent: check enable pin, check input to that regulator.
  If present but wrong value: check feedback resistors.
```

**5. Check enable pins and reset lines:**
```
If all power rails are correct but system does not respond:
  Measure nRESET TP: should be high after power-up delay (typically 10-200 ms)
  If nRESET stuck low: supervisory circuit or power-on reset has not released
  Check JTAG/SWD TPs: can the debugger connect?
```

This systematic approach uses the power tree and test points to isolate the fault to a specific stage without requiring the board to function normally.

---

## Tier 2 — Intermediate

### Question I1
**Explain the JTAG boundary scan EXTEST instruction. How would you use it to detect a solder bridge between two adjacent signal traces on a PCB, assuming both signals originate from JTAG-compliant devices?**

**Answer:**

The EXTEST (External Test) instruction connects the boundary scan cells to the IC pins (in place of the internal logic) and allows the test controller to drive pin outputs and capture pin inputs through the JTAG scan chain.

**Procedure to detect a solder bridge between Signal A (output of Device 1) and Signal B (input of Device 2), where both lines are adjacent:**

```
Setup:
  Signal A: output pin of Device 1 (JTAG-compliant)
  Signal B: input pin of Device 2 (JTAG-compliant)
  Fault hypothesis: solder bridge shorting A and B

  Additionally: Signal C — a separate input pin of Device 2 adjacent to B,
  driven by a non-JTAG driver. Signal C is unaffected by EXTEST.

Step 1 — Shift EXTEST instruction into both devices via the JTAG chain.

Step 2 — Preload pattern:
  Device 1, Signal A: set output cell to logic 1.
  Device 2, Signal B: preload input cell to a known state (irrelevant — it will be captured).

Step 3 — Apply EXTEST: Device 1 drives Signal A high.

Step 4 — Capture: Device 2 captures the state of Signal B.
  Expected (no fault):  Signal A = 1, Signal B = 0 (driven by its own source)
  With solder bridge:   Signal A = 1, Signal B = 1
                        (Signal A has pulled Signal B high through the bridge)

Step 5 — Repeat with Signal A = 0:
  With solder bridge:   Signal B = 0 (pulled low by Signal A)

A bridged net will mirror the state driven on Signal A regardless of what
Signal B's intended driver is doing — this is the diagnostic signature.
```

**Limitations:**
- Requires both ends of the potentially bridged nets to be observable through boundary scan.
- Cannot detect resistive bridges > several kΩ (resistance attenuates the logic level).
- Nets without JTAG access (power, GND, non-JTAG ICs) cannot be observed this way — requires ICT or flying probe for those.

---

### Question I2
**Compare flying probe and bed-of-nails testing for a new product in its first production run of 500 units per month, growing to 5,000 units per month in 18 months. What test strategy would you recommend at each volume stage?**

**Answer:**

**NPI phase (0-500 units/month):**

```
Recommended: Flying probe + functional test

Rationale:
  Flying probe has no fixture cost — critical during NPI when the board design
  is still changing (ECOs, component substitutions, footprint corrections).
  Each ECO would require a new BoN fixture (£5,000-£30,000 and 4-6 weeks lead time)
  — unacceptable during active design iteration.

  Flying probe identifies shorts, opens, and wrong component values.
  Functional test (programmable load board or bed-of-nails-free fixture with
  connectors only) exercises the full design.

  Trade-off: Flying probe test time is 3-10 minutes per board. At 500 units/month,
  this is 25-83 hours/month of tester time — manageable with one tester.
```

**Growth phase (500-5,000 units/month):**

```
Recommended: Transition to ICT bed-of-nails + functional test when volume justifies

Decision point: When fixture cost amortises over expected production volume.
  Fixture cost: £20,000 (mid-complexity board)
  Flying probe cost: £0.50-£2.00/minute, assume £1.00/minute, 5 min test = £5.00/unit
  BoN test cost: £0.05-£0.10/unit (once fixture exists)
  
  Break-even volume:
  £20,000 / (£5.00 - £0.10) = 4,082 units

  At 5,000 units/month, fixture pays back in < 1 month. Transition is warranted.

Requirements before transition:
  - PCB design is stable (no pending ECOs that would change TP positions)
  - TP coverage on design is adequate for BoN (confirm during DFT review)
  - Fixture design and build time allowed in programme schedule

Hybrid strategy during transition:
  Continue flying probe for NPI builds and engineering validation.
  Introduce BoN fixture for production line in parallel.
  Use BoN for first-pass screening, flying probe for investigating BoN failures.
```

---

### Question I3
**A 32-bit ARM Cortex-M4 microcontroller is used in a product. The device supports both JTAG and SWD interfaces. List the minimum test points you would expose on the PCB for programming, debug, and manufacturing test, and justify each one.**

**Answer:**

```
Signal     Direction  Justification
------     ---------  -------------
SWDIO      Bidir      Serial Wire Debug data — required for SWD programming and debug
SWDCLK     Input      Serial Wire Debug clock — required for SWD
nRESET     Input      Assert during programming to ensure clean start state;
                      also test point for supervisory circuit verification
GND        Return     Reference for debug probe — must be adjacent to SWDIO/SWDCLK TPs
VDD_MCU    Supply     Verify MCU supply voltage at the IC during power-on test
BOOT0      Input      On STM32 family: controls boot mode. Expose to force DFU (device
                      firmware upgrade) mode if JTAG fails. On other devices: expose
                      any strap pin that controls boot path.
UART_TX    Output     Serial console output — valuable during bring-up and production
                      test for reading self-test logs from firmware
UART_RX    Input      For firmware-controlled test vectors
SWO        Output     (Optional) Serial Wire Output — trace/ITM output for non-intrusive
                      profiling and printf debugging without halting the core
```

**Why SWD over JTAG for this application:**

SWD uses only two signal pins (SWDIO, SWDCLK) versus four for JTAG (TCK, TMS, TDI, TDO). For a Cortex-M4 with no multi-device JTAG chain, SWD provides identical debug capability in half the connector space. ARM's CoreSight SWD protocol also supports multi-drop topologies when needed.

**Connector format:**

The ARM Cortex standard debug connector is the 10-pin 0.05-inch Cortex connector (Samtec FTSH-105) or the 20-pin ARM JTAG connector. For space-constrained production boards, the 10-pin Cortex connector provides SWDIO, SWDCLK, nRESET, SWO, VCC (sense), and GND in a 5 mm × 10 mm footprint.

---

## Tier 3 — Advanced

### Question A1
**A production board has a 95% first-pass test yield using flying probe. The remaining 5% require manual rework. An engineering review shows that 60% of failures are solder bridges on fine-pitch ICs, 30% are missing or tombstoned passives, and 10% are wrong-value components. Propose a DFT improvement plan that targets higher fault coverage and shifts detection earlier in the build process.**

**Answer:**

**Root cause analysis:**

```
Fault distribution:
  60% solder bridges — detectable by: AOI (automated optical inspection), X-ray, ICT
  30% missing/tombstoned — detectable by: AOI, ICT
  10% wrong-value — detectable by: ICT (in-circuit measurement)

All three fault types are detectable before power-on.
Flying probe is finding them late — after assembly and before functional test.
The objective is to move detection earlier in the process.
```

**Improvement plan:**

**1. Add inline AOI after reflow (eliminates most of the 60% bridge faults):**
```
AOI inspection immediately after reflow oven, before electrical test.
AOI detects solder bridges, missing components, skewed components.
Cost: AOI system £50,000-£200,000, or outsource AOI inspection.
Benefit: Boards with visible bridges are caught immediately — rework while
         solder is still soft reduces rework cost significantly.
         AOI also detects cosmetic issues not visible to flying probe.
```

**2. Add X-ray inspection for hidden joints (BGA, QFN):**
```
Fine-pitch BGA solder bridges are invisible to AOI but visible to AXI (automated X-ray).
If the solder bridges are on the fine-pitch device package, AXI is required.
Cost: AXI system £200,000+. Justify only if the fine-pitch device volume warrants.
Alternative: Review PCB design — stencil aperture, pad design, solder paste volume
for the fine-pitch device. Often the root cause is a DFM issue, not just AOI absence.
```

**3. Improve JTAG coverage on fine-pitch ICs:**
```
If the fine-pitch IC is JTAG-compliant, add JTAG test points to the PCB.
EXTEST can detect solder bridges between adjacent pins without a BoN fixture.
This catches the bridge faults electrically even if AOI misses them.
```

**4. Increase test point coverage for the 30% missing-passive fault:**
```
Add TPs to each passive component's net that currently lacks TP coverage.
Flying probe can then verify passive component presence and value.
Alternatively, add 0.5 mm SMD TPs on both sides of every passive >= 0402.
```

**5. ICT introduction at higher volume:**
```
If volume increases, introduce BoN ICT fixture.
ICT can detect wrong-value resistors/capacitors by direct measurement.
This catches the 10% wrong-value faults that flying probe may miss if
it only performs open/short tests without in-circuit measurement.
```

**Expected improvement:**

With AOI added inline:
- Solder bridge detection rate rises from flying probe's ~90% to AOI's ~98%.
- Boards presenting bridges to flying probe reduce by ~60%.
- First-pass yield target: from 95% to ~98-99%.

---

### Question A2
**Describe how you would implement a production self-test strategy for a board that has no external test point access because it is a sealed, conformal-coated module. What firmware and hardware features would you include at design time to enable lifetime testability?**

**Answer:**

When physical probe access is eliminated by mechanical design, testability must be designed into the product's firmware and hardware architecture from the outset.

**Hardware provisions:**

```
1. Dedicated JTAG/SWD programming interface:
   Even a sealed module should have an unpopulated programming header footprint
   accessible by removing a few fasteners, or a pogo-pin programming interface
   on the module's base plate. This enables:
   - Firmware programming and update
   - Debug access during rework and repair
   - BIST invocation via debugger

2. UART or USB console port:
   A header or pogo-pin footprint for a console UART (TX, RX, GND).
   At factory: before sealing, connect console to run production self-test.
   In field: if the module fails, a repair technician opens it and connects.

3. Hardware self-test inputs/outputs:
   Loopback-capable interfaces: route UART TX back to UART RX through a
   0-Ω jumper (DNF = do not fit normally) — firmware can verify UART path.
   ADC: include a precision reference voltage (resistor divider from a known rail)
   routed to one ADC input. Firmware measures this and verifies ADC accuracy.
   DAC → ADC loopback via a 0-Ω resistor (DNF) for production test.

4. Power rail supervision outputs:
   Each regulated rail connects to a supervisor IC or ADC input.
   POWER_GOOD signals available to the host MCU, logged to non-volatile memory.
   A field technician can read power rail status from firmware without probing.
```

**Firmware provisions:**

```
1. Built-in self-test (BIST) sequence:
   Invoked by a specific power-on condition (hold a button, specific I2C command,
   or JTAG call) or at every power-on with results logged.
   BIST checks:
     - RAM: walking-ones test, checkerboard pattern
     - Flash: CRC32 of firmware image vs stored value
     - All ADC inputs: measure and compare to expected range
     - All communication interfaces: loopback test (UART, SPI, I2C)
     - External peripherals: ping each peripheral device, verify ACK
     - Power rails: read ADC-monitored rails, check within ±5%

2. Error logging to non-volatile storage:
   Any fault detected at runtime is logged with a timestamp, fault code, and
   context data to flash or EEPROM.
   Field technician retrieves log via the console port or a USB firmware tool.
   This enables fault correlation across multiple units in the field.

3. Telemetry output:
   Periodically report health data (temperatures, voltages, cycle counts,
   error counts) via the product's primary communication interface
   (CAN, Ethernet, Bluetooth) so failures can be detected remotely.

4. Production mode flag:
   A one-time programmable (OTP) bit or option byte distinguishes
   "production test mode" from "deployed mode".
   In production mode: run full BIST, report detailed results via UART.
   In deployed mode: run minimal BIST, report only critical faults.
```

**Production test flow with no external probing:**

```
Assembly → Conformal coat not yet applied
  ↓
Program firmware via pogo-pin SWD fixture on assembly line
  ↓
Firmware invokes production BIST automatically at first boot
  ↓
BIST results printed on UART console (connected to test fixture)
  ↓
Pass: write PRODUCTION_PASS flag to OTP; apply conformal coat; pack
Fail: rework; re-test
```

This approach provides near-equivalent test coverage to external probing because the firmware has direct access to every node the hardware exposes through the MCU's peripherals — which is most of the critical nets on an MCU-centric design.
