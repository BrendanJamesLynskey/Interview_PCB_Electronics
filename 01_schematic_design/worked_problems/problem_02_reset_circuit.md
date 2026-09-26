# Problem 02: Reset Circuit Design

## Problem Statement

You are designing the reset circuitry for a mixed-signal embedded system with the following components:

- **MCU**: ARM Cortex-M4, 3.3 V supply, active-low reset (nRESET pin). Requires nRESET asserted for a minimum of 1 µs. Internal power-on reset (POR) holds nRESET low for 1 ms after VDD exceeds 1.5 V.
- **FPGA**: Xilinx Artix-7, active-low configuration reset (PROGRAM_B pin). Must be held low for ≥ 500 ns. Configuration begins when PROGRAM_B de-asserts.
- **External EEPROM**: 3.3 V, I2C interface. No dedicated reset pin. Requires 100 ms after power valid for internal initialisation.
- **Supervisor/watchdog IC**: TI TPS3813, open-drain RESET output, 200 ms timeout, active-low output.

**System requirements:**
1. At power-up, all components must be held in reset until VDD_3V3 exceeds 3.0 V and has been stable for at least 100 ms.
2. The MCU watchdog must reset the entire system (MCU and FPGA) if the firmware fails to kick the watchdog within 200 ms.
3. A manual reset button must reset both MCU and FPGA, but must be debounced.
4. The FPGA must not begin configuration until the EEPROM is accessible (the MCU reads FPGA bitstream from the EEPROM and programs the FPGA via SPI).
5. The system reset signal must be immune to supply glitches shorter than 1 µs.

Design a reset circuit that meets all five requirements.

---

## Solution Approach

### Step 1 — Identify the Reset Signal Hierarchy

```
System power-up sequence:

  VDD_3V3 rises
       │
  Supervisor IC monitors VDD_3V3
       │
  When VDD_3V3 > 3.0 V threshold AND stable for 100 ms:
       │
  Supervisor RESET output de-asserts (nRESET_SYS goes high)
       │
  MCU nRESET de-asserts → MCU boots
       │
  MCU firmware initialises I2C bus
       │
  MCU reads FPGA bitstream from EEPROM (~50 ms for a small bitstream)
       │
  MCU asserts FPGA PROGRAM_B low for ≥ 500 ns (initiates configuration)
       │
  FPGA configures from SPI stream provided by MCU
       │
  FPGA DONE pin asserts → system operational

The FPGA is therefore intentionally held in configuration-inactive state until
the MCU explicitly triggers it. This satisfies requirement 4 naturally.
```

### Step 2 — Supervisor IC Selection

The TPS3813 (specified) provides (TI TPS3813 datasheet):

```
Monitoring threshold: fixed per device variant — there is no adjustable SENSE input.
  TPS3813K33: VIT- = 2.93 V typical (2.87 V to 3.00 V), matching the ~3.0 V target
  (about 89% of the 3.3 V nominal).

Reset delay: fixed, 25 ms typical (20-30 ms) after VDD rises above VIT.
  This does NOT meet requirement 1 ("stable for at least 100 ms").
  Either choose a supervisor with a reset delay of at least 100 ms, or extend the
  hold-off (e.g. an additional delay stage on nRESET_SYS, or MCU firmware that waits
  a further 75 ms before releasing the FPGA and touching the EEPROM).

Watchdog function: window watchdog on WDI. A trigger that arrives too late OR too
  early asserts RESET. The window ratio is set by WDR and the timeout by WDT.
  WDI is driven by MCU firmware GPIO; minimum WDI trigger pulse width 50 ns.

RESET output: open-drain, active-low. Pull up to VDD_3V3 via 10 kΩ resistor.
```

### Step 3 — Manual Reset Button Debouncing

A mechanical pushbutton generates multiple contact bounces over 10-50 ms. A single RC debounce circuit is insufficient for an active-low reset:

```
Naive approach (RC debounce only):
  Button ── R (10 kΩ) ── RESET pin
  Pulled up to VDD via R_pull = 10 kΩ
  C to GND = 100 nF
  τ = 10 kΩ × 100 nF = 1 ms

  Problem: On release, the capacitor discharges through R and R_pull.
  Multiple bounces during release can create glitches on RESET.
```

**Improved approach — Schmitt-trigger debounce with supervisor gating:**

```
Circuit:
  VDD_3V3 ──[10 kΩ pull-up]──┬── RESET_BTN_RAW
                               │
  GND ──[pushbutton]───────────┘
                               │
                              [10 kΩ]
                               │
                              [100 nF to GND]
                               │
                          [Schmitt trigger inverter, e.g., SN74LVC1G14]
                               │
                          RESET_BTN_DEBOUNCED (active high when button pressed)

The Schmitt trigger input hysteresis (typically 0.7 V on 3.3 V supply) prevents
the output from oscillating during slow transitions caused by the RC filter.

The RC time constant (1 ms) attenuates bounces < 5 ms. The Schmitt trigger
ensures a clean digital transition.
```

**Combining manual reset with supervisor reset:**

```
nRESET_SUPER (from TPS3813, active low, open-drain)  ─────┐
                                                           │
nRESET_BTN (from Schmitt-trigger, active low)          ───┤ Wired-AND (open-drain)
                                                           │
nRESET_SYS ───────────────────────────────────────────────┘
                                     │
                              [10 kΩ pull-up to VDD_3V3]

nRESET_SYS is low when either source asserts reset.
Both are open-drain — they can be safely wire-ANDed without short-circuit risk.
```

### Step 4 — Glitch Filtering (Requirement 5)

The supervisor asserts RESET whenever VDD_3V3 dips below its threshold; its delay applies only on release, so it does not filter short dips — a sub-millisecond glitch below threshold produces a full reset. For short glitches that cause nRESET_SYS to momentarily glitch low:

```
Add a glitch filter on nRESET_SYS before it reaches the MCU:

nRESET_SYS ──[1 kΩ]──┬── nRESET_MCU (MCU nRESET pin)
                      │
                     [10 nF to VDD_3V3]  (pull to logic high)
                     [Schmitt trigger input on MCU nRESET — MCUs typically
                      specify Schmitt trigger on this pin]

τ = 1 kΩ × 10 nF = 10 µs
Glitches < ~5 µs are attenuated below the Schmitt trigger threshold.
Satisfies "immune to supply glitches shorter than 1 µs" with significant margin.

Note: The 1 kΩ series resistor must not slow the legitimate reset assertion
below the MCU's minimum reset hold time. At assertion (nRESET going low),
the capacitor discharges through the resistor and the pull-down source.
With a supervisor open-drain pull-down through 330 Ω, the RC charge time
toward 0 V: τ = (1 kΩ || 330 Ω) × 10 nF ≈ 248 Ω × 10 nF = 2.5 µs.
Time to cross Schmitt low threshold (≈ 0.8 V on 3.3 V): ≈ 2.5 µs.
MCU requires nRESET ≤ VIL for ≥ 1 µs. The filter adds 2.5 µs delay, so the
total assertion hold time remains >> 1 µs required. ✓
```

### Step 5 — FPGA Reset and Configuration Gate

The FPGA is intentionally not connected to nRESET_SYS. Instead, the MCU controls it:

```
MCU GPIO_FPGA_PROG ──[33 Ω series]── FPGA_PROGRAM_B
                                     (FPGA Artix-7 pin M4, 3.3 V bank)

At power-up: MCU GPIO_FPGA_PROG is configured as INPUT (high-impedance) or driven low.
  If driven low: FPGA held in reset, cannot begin configuration.

Sequence:
  1. MCU boots after nRESET_SYS de-asserts.
  2. MCU initialises SPI and I2C.
  3. MCU reads FPGA bitstream from EEPROM via I2C (~50 ms).
  4. MCU asserts GPIO_FPGA_PROG low for ≥ 1 µs (PROGRAM_B minimum assert time: 500 ns).
  5. MCU releases GPIO_FPGA_PROG high.
  6. MCU drives SPI CLK and DATA to FPGA (Slave Serial or SPI mode programming).
  7. FPGA_DONE pin goes high → MCU reads this as a GPIO interrupt to confirm success.
  8. If FPGA_DONE does not assert within expected time → fault handling.

Additional: FPGA CFG_RESET_B (global set/reset, distinct from PROGRAM_B) can be
driven by nRESET_SYS to reset FPGA fabric during system reset events.
```

### Step 6 — Watchdog Integration

```
MCU firmware: On each pass of the main loop (< 200 ms period):
  Toggle WDI_MCU GPIO to kick the TPS3813 watchdog.

If firmware hangs (infinite loop, hard fault, exception):
  WDI_MCU stops toggling.
  After 200 ms timeout: TPS3813 asserts nRESET_SYS low.
  Both MCU and FPGA (via nRESET_SYS) receive reset.
  System performs a full cold reboot.

WDI pulse requirement (TPS3813): Minimum pulse width 50 ns to register a kick.
  MCU GPIO toggle = 1 clock cycle at 168 MHz = ~6 ns — too short!
  Solution: hold the WDI GPIO high for at least 50 ns in firmware (about 9 CPU
  cycles at 168 MHz — a short delay loop), then drive it low.
  Do not use an RC "pulse stretcher" (e.g. 1 kΩ + 10 nF): an RC low-pass attenuates
  a 6 ns pulse — to well under 1% of its amplitude with τ = 10 µs — rather than
  lengthening it, so the kick would never register.

  Because the TPS3813 has a window watchdog, kicks must also not come too early:
  schedule them inside the window set by WDR/WDT, not simply "as often as possible".
```

---

## Complete Circuit Summary

```
VDD_3V3
    │
    ├──── TPS3813K33 VDD pin (fixed 2.93 V threshold, no divider)
    │
    ├──[10 kΩ pull-up]── TPS3813 RESET (open-drain output) ──────┐
    │                                                              │ (wire-AND)
    ├──[Manual button] ──[10 kΩ/100 nF RC] ──[Schmitt 74LVC1G14]─┘
    │                                                              │
    │                                                          nRESET_SYS
    │                                                              │
    │                                              ┌──[1 kΩ + 10 nF glitch filter]──┐
    │                                              │                                  │
    │                                          nRESET_MCU (MCU)           (MCU GPIO → TPS3813 WDI)
    │                                                                      (firmware-timed pulse ≥ 50 ns)
    │
    └── MCU GPIO_FPGA_PROG ──[33 Ω]── FPGA PROGRAM_B
        MCU GPIO_FPGA_DONE ──[33 Ω]── FPGA DONE (input to MCU)
```

---

## Analysis

### Why Not Use nRESET_SYS Directly for the FPGA?

The FPGA's PROGRAM_B signal initiates configuration: asserting it low erases the FPGA's configuration memory and, when de-asserted, starts the configuration process. If nRESET_SYS drives PROGRAM_B directly, the FPGA begins trying to configure itself as soon as nRESET_SYS de-asserts — before the MCU has had time to read the bitstream from EEPROM. Since the FPGA cannot configure itself without a bitstream source, it would stall or fail.

Decoupling FPGA configuration from the system reset allows the MCU to manage the sequence deterministically. The MCU can also implement error handling: if the first configuration attempt fails (corrupt bitstream, I2C error), the firmware can retry before declaring a fault.

### What Happens at Watchdog Reset?

```
t = 0:      MCU firmware hangs. WDI stops toggling.
t = 200 ms: TPS3813 asserts nRESET_SYS low.
t = 200 ms: MCU nRESET asserts (via glitch filter, 2.5 µs delay).
t = 200 ms: MCU enters reset.
t = 200 ms: FPGA fabric reset (via CFG_RESET_B if connected to nRESET_SYS)
             or FPGA continues running if only PROGRAM_B is MCU-controlled.

Note: If the FPGA should be reset on watchdog events, CFG_RESET_B (or PROGRAM_B)
must be connected to nRESET_SYS. If the FPGA holds critical state that must survive
MCU resets, isolate FPGA reset from nRESET_SYS and implement FPGA-independent
watchdog in firmware.

t = 200 ms + ~25 ms (supervisor reset delay after the watchdog event):
  Supervisor de-asserts nRESET_SYS
  MCU boots fresh, reconfigures FPGA.
```

### Reset Circuit Failures — Common Mistakes

```
1. Floating nRESET pin:
   If nRESET has no pull-up and the supervisor is not connected, the pin floats.
   Capacitive coupling can hold the pin at a mid-rail voltage, causing:
     - MCU trapped in a partial-reset state
     - Intermittent behaviour that only manifests at certain temperatures
   Rule: Always have a defined resistive pull-up or pull-down on every reset pin.

2. Insufficient watchdog kick period margin:
   If the main loop occasionally takes 190 ms and the watchdog timeout is 200 ms,
   interrupt latency or OS scheduling jitter can cause spurious resets.
   Rule: Nominal watchdog kick period ≤ 50% of timeout. For 200 ms timeout,
   kick at ≤ 100 ms intervals.

3. Wrong supervisor threshold:
   Selecting a threshold too close to nominal VDD means small load transients
   trigger reset. Selecting too low means the MCU runs on an undervoltage rail.
   Rule: Threshold = 90-92% of nominal VDD. 3.0 V for a 3.3 V system (91%) is standard.

4. Button not debounced:
   A single press generates 20-50 contact bounces in 10-50 ms.
   Without debouncing, the MCU may reset multiple times from one press.
   The RC + Schmitt trigger approach provides ≥ 10 ms debounce — sufficient
   for any mechanical switch.

5. nRESET asserted too briefly:
   An RC delay that provides 100 µs of reset assertion seems safe, but if the
   MCU clock stabilises quickly, the internal POR timer may require a longer
   external reset. Always check the IC datasheet for minimum RESET assert duration.
```

---

## Key Takeaways

1. **Separate the power-on reset function from the system reset function.** The supervisor IC owns power-on reset. The MCU and FPGA have their own hold-off requirements — implement them as part of the design, not as afterthoughts.

2. **Decouple FPGA configuration from reset.** For FPGAs that are programmed by the MCU, the MCU controls configuration timing explicitly. This provides deterministic startup and the ability to implement retry logic.

3. **Watchdog timeout must be longer than worst-case software loop time by a significant margin.** A watchdog that is frequently almost-expired provides no reliability benefit and creates intermittent reset faults.

4. **Every open-drain reset output needs a pull-up resistor.** The resistor value determines the assertion drive strength — too large and rise time is slow; too small and the open-drain device may not pull fully low.

5. **Glitch filtering on the reset net prevents false resets from supply noise.** A simple RC + Schmitt trigger on the input to the MCU adds insurance against ESD or switching transients that momentarily disturb the reset line.

---

## Interview Notes

**Likely follow-up questions:**

- "How would you test that the watchdog actually resets the system?" → In firmware test mode, stop kicking the watchdog deliberately and verify that the system resets within the expected timeout. Log a "watchdog reset" flag in non-volatile memory so the MCU can detect and report that the previous reset was watchdog-triggered.

- "How do you prevent the watchdog from being accidentally kicked in an interrupt service routine while the main loop is stuck?" → The watchdog kick must only occur in the main thread context after completing a checklist of critical tasks (not just toggling a GPIO). Some designs use a "watchdog service record" where each software module sets a bit when it completes its iteration; the main loop only kicks the watchdog if all required bits are set, then clears them.

- "What if VDD_3V3 rises very slowly — does the supervisor still work correctly?" → The TPS3813 threshold detection is ratiometric to VDD, so a slow supply ramp does not cause metastability. The 25 ms reset delay begins only after the threshold is crossed — if the supply takes 500 ms to rise, the system is held in reset for about 525 ms total. This is correct behaviour.

- "Can you use the MCU's internal power-on reset instead of an external supervisor?" → The internal POR monitors VDD for the MCU only. It provides no watchdog function, no monitoring of other rails, no monitoring of threshold crossings during operation (brownout detection exists on some devices but is less configurable). For production designs, external supervisors with precise thresholds and watchdog timers provide more reliable system protection.
