# Rework and ECN

## Prerequisites
- Familiarity with SMT and through-hole soldering techniques
- Understanding of PCB layer stackup and net connectivity
- Basic knowledge of ESD-safe handling practices
- Ability to read and modify a schematic

---

## Concept Reference

### What Rework Is

Rework is the process of modifying an assembled PCB to correct a fault, implement a
design change, or verify a hypothesis. Rework ranges from minor (replacing a single
passive component) to complex (removing and replacing a BGA, cutting traces, adding
wire bridges). All rework must be controlled: uncontrolled rework on a production board
creates liability and traceability problems.

Rework in a development context is an engineering tool. It allows a design hypothesis
to be verified in hours rather than weeks (the time to spin a new PCB). The trade-off
is that reworked boards have reduced reliability compared to properly designed boards,
and rework results cannot always be replicated in production.

**Rework vs. Repair:**
- Rework: modifying a board to implement a design change or engineering evaluation
- Repair: restoring a board to its original design intent after a fault or damage

The distinction matters for documentation: rework changes what the board is, so the
resulting assembly state must be documented against a specific ECN. Repair restores
the original assembly state.

---

### Soldering Iron Technique

**Temperature settings:**
```
Lead-free solder (SAC305):  360-380°C iron tip temperature
Leaded solder (Sn63/Pb37):  320-350°C iron tip temperature
Fine-pitch QFP rework:       320°C maximum (to avoid pad lifting)
Wire bonding to via:         300°C, brief contact only
```

Higher temperature is not always faster. Excessive temperature damages pads, lifts
traces, degrades PCB laminate, and overheats adjacent components. The correct approach
is the correct tip shape and adequate thermal mass, not excessive temperature.

**Tip selection:**
- Chisel tip: highest thermal mass, best for drag soldering fine-pitch flat packs and
  for through-hole component removal
- Conical tip: precise placement, used for individual SMD component rework
- Knife tip: for soldering fine-pitch ICs by drag soldering along a row of pads
- Micro tip (0.1 mm): for 0402 and smaller components, requires high magnification

**Flux:**
Apply flux generously before soldering. Flux:
- Removes oxide from pad and component surfaces, enabling proper solder wetting
- Reduces surface tension of molten solder, improving joint formation
- Carries solder to the correct location by capillary action

Use no-clean flux for most rework. Fully wash flux off boards that will go into
enclosed environments or undergo high-reliability testing — residues can absorb moisture
and cause surface leakage in humid conditions.

---

### SMT Rework by Component Type

**0402 / 0603 / 0805 passive components:**
Removal: heat both pads simultaneously with a chisel tip and slide the component off
with tweezers. Do not lever the component — pad lifting is the most common damage mode.

Replacement: apply flux to pads. Place component with tweezers. Touch soldering iron
with a small amount of solder to one end to tack in place. Solder the second end fully,
then return to the first end.

**QFP (fine pitch flat pack):**
Removal:
1. Apply flux generously to all leads
2. Add solder to all leads to bridge them into a single block of solder per side
   (this ensures all pins melt simultaneously — a QFP removed with uneven heating
   will twist and damage pads)
3. Apply hot air at 350-380°C to the device while gently lifting with tweezers
4. Remove residual solder with wick and flux

Replacement:
1. Clean all pads with wick to remove old solder
2. Apply tacky flux to all pads
3. Place IC carefully, align pins 1 corner using a microscope
4. Tack pin 1 and the diagonally opposite corner pin with a small amount of solder
5. Inspect alignment under magnification before soldering all pins
6. Drag solder remaining pins with chisel tip plus small amount of solder
7. Inspect for bridges using magnification and/or continuity test

**QFN (quad flat no-lead):**
QFNs have pads only on the bottom of the package, making them impossible to solder
or inspect with standard iron techniques. Removal and replacement requires a rework
station with hot air (or preferably a rework oven profile) and X-ray inspection to
verify the joints after placement.

**BGA (ball grid array):**
BGA rework requires a dedicated BGA rework station (Finetech, ERSA, Heller) with:
- Thermocouple profiling of the BGA area
- A precisely controlled temperature profile that matches the solder paste profile
- A vision system for BGA alignment
- Reballing tooling if the BGA is to be reused

BGA rework is the most technically demanding rework operation and should only be
attempted if:
- The correct rework station is available and the engineer is trained on it
- The board design has adequate clearance around the BGA for hot air impingement
- X-ray inspection is available to verify the result

**Warning:** Never attempt BGA rework with a standard hot air pencil. The temperature
uniformity is insufficient, and the result will be a mix of bridged, open, and correct
joints that cannot be inspected without X-ray.

---

### Trace Cutting and Wire Bridging (Blue Wires)

When the schematic contains a net connectivity error (wrong connection, missing
connection), the correction requires physically modifying the copper traces. This is
performed by:

1. **Trace cutting:** Using a sharp knife or micro-drill to sever a copper trace,
   breaking the unwanted connection.
2. **Wire bridging (blue wire):** Soldering a thin insulated wire directly onto pads
   or component leads to create a connection that does not exist on the PCB.

**Why "blue wire"?**
Historically, wire-wrapping wire (30 AWG, PTFE insulated) was available in blue. Blue
became the standard colour for hand-wired modifications on PCBs in defence and aerospace
work. The term persists even when wires of other colours are used.

**Wire selection:**
- 30 AWG wire-wrap wire (PTFE insulated): standard for low-current (< 500 mA) signal
  modifications. PTFE insulation withstands iron temperatures, does not melt.
- 28 AWG PTFE: for currents up to 1 A
- Enamelled copper (magnet wire): thin diameter (as small as 0.1 mm), excellent for
  routing through vias. Requires burning off the enamel at endpoints with the iron.
- Standard hookup wire (PVC insulated): NOT recommended — PVC melts at iron temperature

**Trace cutting procedure:**
```
1. Identify the trace to cut on the PCB using the schematic and layout together
2. Verify the cut location: the trace must be cut at a point that breaks only the
   unwanted connection (not cutting through a via, under a component, or near a
   closely parallel trace)
3. Score the trace twice with a sharp scalpel (#11 blade), 1-2 mm apart, cutting
   through the copper and slightly into the FR4
4. Remove the short copper segment between the two cuts with a dental pick
5. Use an ohmmeter to verify the connection is broken before proceeding
6. Apply a small amount of conformal coating or nail varnish over the cut to prevent
   moisture ingress and mechanical damage
```

**Wire attachment procedure:**
```
1. Identify the two endpoints: use pads, component leads, or via barrel as attachment
   points. Never attach to a trace mid-span without a mechanical anchor.
2. Apply flux to both endpoints
3. Tin the wire ends (pre-solder the wire) and tin the PCB endpoints
4. Tack one wire end to one endpoint with minimum solder
5. Route the wire to the second endpoint, cutting to length
6. Tack the second end
7. Verify the connection with a continuity test
8. Anchor the wire at intervals with a drop of UV-curable adhesive or epoxy to prevent
   wire movement and mechanical stress on the solder joints
9. Document the modification with a photograph
```

---

### The ECN (Engineering Change Notice) Process

An Engineering Change Notice (also called Engineering Change Order, ECO) is the formal
process by which design changes are documented, reviewed, approved, and implemented.
A proper ECN system ensures that:

- Every board in the field can be identified by its configuration state
- Design changes are tracked with rationale and approval history
- Production boards are built to a defined, documented configuration
- Regulatory submissions remain accurate (changes to a safety-critical design may
  require re-testing and re-submission)

**ECN fields — minimum set:**

| Field | Content |
|-------|---------|
| ECN number | Unique identifier, usually sequential (ECN-0042) |
| Date | Date of creation |
| Author | Engineer raising the change |
| Approver | Engineering manager or peer reviewer |
| Affected documents | Schematic number, PCB layout revision, BOM revision |
| Before state | Description or schematic snippet of the circuit before the change |
| After state | Description or schematic snippet of the circuit after the change |
| Reason | Root cause of the defect or motivation for the change |
| Rework instructions | Specific step-by-step board modification procedure |
| Verification | How to verify the rework is correct and the change achieves the desired result |
| Affected units | Serial numbers of boards to be reworked, or "all units from batch X" |

**ECN states:**

```
DRAFT ──► IN REVIEW ──► APPROVED ──► IMPLEMENTED ──► CLOSED
                │
                └──► REJECTED
```

A rejected ECN is kept in the archive — it documents what was considered and why it
was not implemented.

**Board marking:**
After rework under an ECN, the board must be marked to indicate its modified state.
Methods:
- Permanent marker label with ECN number
- Sticky label on the board (not suitable for high-temperature environments)
- A modification record card stored with the board
- In production, the hardware revision number on the silkscreen is incremented for
  PCB-level changes (R1.0 → R1.1)

---

### Board Revision vs. ECN

A board revision (hardware revision increment) is appropriate when a design change
affects the PCB layout itself (new copper required). An ECN can implement the change
by rework on existing boards, but production boards from revision R1.1 onwards will
have the correct layout without rework.

The revision decision tree:
```
Is the change achievable by rework (trace cut + wire add + component change)?
  YES → ECN on existing boards; implement in next layout revision
  NO  → New board spin required immediately; ECN is for existing inventory only
```

---

## Rework Techniques

### Hot Air Rework Station

A hot air rework station delivers a controlled flow of heated air to a specific area
of the board. Key parameters:

- **Temperature:** 300-400°C at the nozzle (actual temperature at the board surface
  is lower due to air cooling — profile must be verified with a thermocouple)
- **Air flow rate:** 10-30 litres/minute (too high = blows adjacent components;
  too low = uneven heating)
- **Nozzle selection:** use a nozzle that matches the package size to concentrate heat
  and protect adjacent components

**Thermal profiling for hot air rework:**
Before reworking a valuable component on a first-article board, practice on a scrap
board with the same stackup and package:
1. Attach thermocouples to the package body and to a pad 5 mm away from the package
2. Apply the hot air rework procedure
3. Plot temperature vs. time
4. Verify: peak temperature at package ≥ liquidus (183°C for SnPb, 217°C for SAC305)
   and temperature 5 mm away < 200°C (to avoid damaging adjacent components)

---

## Engineering Change Order (ECN)

### When to Raise an ECN vs. Making an Informal Change

Every modification to a board in a development programme should be tracked. The level
of formality scales with the programme stage:

| Stage | Formality level | Rationale |
|-------|----------------|-----------|
| Early prototype | Engineering log entry, photograph | Speed of iteration is primary concern |
| Pre-production validation board | Full ECN required | These results will be used for regulatory submissions |
| Production | Full ECN with CM notification | Changes affect manufactured units with traceability requirements |
| Post-market (safety-critical) | ECN + regulatory re-assessment | Field units may need recall or service bulletin |

Informal rework without documentation is acceptable only in the earliest prototype
phase and only if the board is clearly marked as an informal prototype. Any board that
will be used for EMC testing, safety testing, or customer demonstration must be tracked
formally.

---

## Best Practices

- Never rework a board without first documenting the intended change in writing (at
  minimum an email). If the rework makes things worse, the documentation allows
  recovery to the pre-rework state.
- Photograph the board before rework, during any trace cutting, and after rework.
  The photographs are part of the ECN record.
- Limit rework iterations on any single board. Three or more rework cycles in the same
  area significantly increases the risk of pad lifting and delamination. A board with
  extensive rework history should be set aside as a reference only, with a new board
  built to the corrected design.
- ESD precautions during rework are often neglected. Use a grounded wrist strap and
  ESD-safe tools. Rework exposes internal device nodes that are not protected by the
  final assembly.
- After rework, perform a visual inspection followed by the full bring-up test sequence
  for the affected subsystem. Never assume rework succeeded without verification.

---

## Tier 1 — Fundamentals

### Question F1
**What is an Engineering Change Notice and what is the minimum information it must contain?**

**Answer:**

An Engineering Change Notice (ECN) is a formal document that records a change to an
existing design. It provides traceability: anyone looking at a board with an ECN applied
can determine exactly what was changed, why, when, and by whom.

Minimum required content:

1. **Unique identifier:** A number that can be referenced in manufacturing records,
   test reports, and fault reports (e.g., ECN-0042).
2. **Date and author:** When the change was created and who created it.
3. **Affected documents:** Which schematic version, PCB layout revision, and BOM are
   affected.
4. **Before and after state:** What the circuit looked like before (including a schematic
   reference or snippet) and what it looks like after the change.
5. **Reason for change:** The root cause of the defect or design limitation being
   corrected.
6. **Rework instructions:** Specific, unambiguous instructions for making the physical
   modification on an existing board.
7. **Verification:** How to confirm the rework was performed correctly and achieves
   the intended result.
8. **Approval:** Name and signature/sign-off of the approving engineer or manager.

---

### Question F2
**A junior engineer asks: "Can I just cut this trace with a knife and solder a wire across to fix the net connection error? Do I need to raise an ECN?" How do you respond?**

**Answer:**

Yes, you need to raise an ECN, and here is why it matters:

Without an ECN:
- There is no record of what the board's actual configuration is. If someone else
  picks up that board and runs tests, they cannot know it has been modified. The
  test results are attached to the wrong configuration.
- If the rework is incorrect (wire adds noise, cut severs the wrong net), there is no
  documentation to guide reverting to the previous state.
- If this board is used for any formal testing (EMC, safety, customer evaluation), the
  results will be untrustworthy because the build state is unknown.

The ECN process ensures:
- The change is documented and photographed
- A second person reviews the change (catching errors in the proposed fix)
- The board is marked with the ECN number after rework
- Future engineers know the board's configuration history

For early prototype boards, the formality can be lightweight (an email or a design
notes entry), but it must exist. The practice of documenting changes should be a
professional habit, not an optional formality.

---

## Tier 2 — Intermediate

### Question I1
**Describe the correct procedure for replacing a 0.5 mm pitch QFP-64 component. What are the three most common failure modes during this operation?**

**Answer:**

**Procedure:**

1. **Preparation:** Apply flux (no-clean tacky flux) to all leads of the existing
   component. Set the hot air station to 350°C, medium airflow.

2. **Removal:** Apply hot air to the component while watching for solder to melt around
   all sides. As the solder becomes liquid (approximately 30-60 seconds), gently lift
   the component with ceramic-tip tweezers. Do not apply force until all pins have
   clearly melted — forcing a partially melted joint will tear pads.

3. **Clean pads:** Apply flux. Use a chisel-tip iron and copper braid wick to remove
   residual solder from all 64 pads. The pads must be flat and clean before placing the
   new component. Inspect under magnification.

4. **Place new component:** Apply fresh tacky flux. Place the new QFP, aligning pin 1
   to the pad 1 marking on the silkscreen. Under a microscope, verify all pins are
   positioned correctly over their pads before soldering.

5. **Tack:** With a fine iron tip, tack pin 1 and the diagonally opposite corner pin
   to hold the component in position. Re-verify alignment.

6. **Solder all pins:** Use a fine chisel tip plus a small diameter solder wire, drag
   solder along each row. Move the iron from one end of the row to the other in a
   single stroke, applying a thin wire of solder as you go.

7. **Inspect and test bridges:** Visually inspect every pin under magnification. Test
   electrically: use an ohmmeter to test for continuity between adjacent pins on each
   side (a reading significantly below the expected resistance between those two nets
   indicates a bridge).

**Three most common failure modes:**

1. **Pad lifting:** Caused by applying force before solder has melted, excessive dwell
   time at high temperature, or thermal shock. A lifted pad is very difficult to recover —
   requires a wire bridge from the lifted pad to the nearest accessible point on the same net.

2. **Solder bridges:** Fine-pitch pins bridged by excess solder. Most are visible under
   magnification. Fix by applying flux and drawing excess solder away with a clean chisel tip
   or wicking braid. Prevention: use minimum solder and adequate flux.

3. **Cold joint or missed pin:** A pin that appears to be soldered but has insufficient
   intermetallic bond, or a pin that was skipped during drag soldering. Identified by high
   resistance or open circuit at that pin. Fix by re-fluxing and re-soldering the specific
   pin.

---

## Tier 3 — Advanced

### Question A1
**During bring-up of a new board, a design error is found: a net that should connect a microcontroller UART_TX to an FTDI chip UART_RX is not connected (the PCB trace was routed to the wrong pad). Simultaneously, a second net connecting UART_RX and UART_TX is reversed. Describe the complete resolution: rework, ECN, and what changes must be made to the design database.**

**Answer:**

This fault is a classic pin swap: TX connects to TX and RX connects to RX (same-to-same
rather than cross-connection). The fix requires two trace cuts and two wire bridges.

**Rework plan:**

First, verify the fault by:
1. Confirming the MCU UART_TX does not have continuity to FTDI UART_RX
2. Confirming MCU UART_TX has continuity to FTDI UART_TX (the wrong connection)
3. Doing the same check for the RX pins

Rework steps:
1. Identify the nearest accessible points for cutting on each incorrect trace. Ideally
   choose a point close to the FTDI side where both traces are routed nearby and
   accessible.
2. Cut the MCU_UART_TX → FTDI_UART_TX trace.
3. Cut the MCU_UART_RX → FTDI_UART_RX trace.
4. Verify both cuts with a continuity test.
5. Bridge MCU_UART_TX to FTDI_UART_RX with a 30 AWG PTFE wire.
6. Bridge MCU_UART_RX to FTDI_UART_TX with a second 30 AWG PTFE wire.
7. Verify the corrected connections with continuity test.
8. Anchor both wires with UV adhesive.
9. Photograph the rework.

**ECN content:**
- Reason: pin swap error — UART TX/RX nets connected same-to-same instead of crossed
- Before state: schematic snippet showing MCU_TX → FTDI_TX, MCU_RX → FTDI_RX
- After state: corrected snippet showing MCU_TX → FTDI_RX, MCU_RX → FTDI_TX
- Rework instructions: as above
- Verification: UART loopback test at 115200 baud, send 0x55 pattern, confirm received

**Design database changes (for next PCB spin):**
1. Schematic: swap the FTDI UART_TX and UART_RX net labels at the FTDI symbol
2. PCB layout: re-route the two traces to connect MCU_TX → FTDI_RX and vice versa
3. Increment the PCB revision number (e.g., R1.0 → R1.1)
4. Update the BOM revision to match (even though no component changes, the build
   document must reference the correct layout revision)
5. Re-run DRC (design rule check) on the revised layout to confirm no violations
   were introduced by the re-route
6. Update the ECN to reference the schematic and PCB revision numbers that implement
   the permanent fix

**Lesson for future designs:**
Add a UART loopback self-test in the bring-up procedure. A 5-second loopback test on
power-on would have caught this error in Phase 5 of bring-up with no additional hardware
needed, saving the trace cutting and wiring rework time.

---

## Related Topics

- [Bring-Up Methodology](bringup_methodology.md)
- [Debugging Techniques](debugging_techniques.md)
- [Design for Test](../01_schematic_design/design_for_test.md)
- [Worked Problem: No Power Debug](worked_problems/problem_01_no_power_debug.md)
- [PCB Fabrication Constraints](../03_design_for_manufacturing/pcb_fabrication_constraints.md)
