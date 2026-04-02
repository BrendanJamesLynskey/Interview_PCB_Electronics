# Conformal Coating

## Prerequisites
- Familiarity with PCB assembly processes (SMT, wave soldering, selective soldering)
- Basic knowledge of PCB surface finishes and materials
- Understanding of operating environment classifications (IPC-CC-830, IEC 60068)
- Awareness of relevant standards for the target market (automotive, industrial, consumer)

---

## Concept Reference

### What Conformal Coating Is

Conformal coating is a protective chemical coating applied to a populated PCB assembly
to protect it from the operating environment. The coating conforms to the board and
component contours, providing a barrier against:

- **Moisture and humidity:** Water ingress causes corrosion of copper traces and pads,
  electrolytic migration between conductors, and leakage currents on insulating surfaces.
- **Chemical contamination:** Solvents, oils, salt spray, cleaning agents, and
  atmospheric pollutants can corrode metallic surfaces and degrade polymer insulators.
- **Mechanical stress:** Some coatings (particularly silicone and polyurethane) provide
  a degree of vibration damping and protection against small mechanical impacts.
- **Dust and particulates:** A coating prevents conductive particles from settling
  on and bridging between conductors.
- **Fungi and biological growth:** Conformal coatings inhibit the growth of fungi on
  PCB surfaces in humid tropical environments.

Conformal coating is not a structural or thermal solution. It does not provide
significant structural reinforcement (for that, use potting/encapsulation) and it
does not significantly improve heat dissipation.

---

### Coating Types

#### Acrylic (Type AR)

```
Property             Value/Characteristic
-------------------  ----------------------------------------
IPC designation      Type AR
Dielectric strength  >100 V/mil (≈4 kV/mm)
Operating temp       -40°C to +125°C
Moisture resistance  Good
Chemical resistance  Moderate (attacked by ketones, esters)
Reworkability        Excellent — easy to remove with solvents
                     (MEK, acetone) without PCB damage
Flexibility          Moderate (brittle at low temperature)
Fungus resistance    Good
Cure method          Air dry or UV cure; some heat cure variants
Application method   Spray, dip, selective coating robot
Typical thickness    25-75 µm
Cost                 Low
Common use           General-purpose, indoor consumer electronics,
                     industrial panels where rework is required
```

**Advantages:** Acrylic is the most widely used conformal coating because of its easy
application and excellent reworkability. A failed component can be locally removed with
a solvent-dipped swab without damage to adjacent areas.

**Disadvantages:** Poor resistance to solvents — if the product will be exposed to
solvent-based cleaning agents, acrylic is not appropriate. Also weaker than other types
under sustained mechanical stress.

#### Silicone (Type SR)

```
Property             Value/Characteristic
-------------------  ----------------------------------------
IPC designation      Type SR
Dielectric strength  >150 V/mil (≈6 kV/mm)
Operating temp       -65°C to +200°C (some grades to +250°C)
Moisture resistance  Excellent
Chemical resistance  Good (resistant to most chemicals except
                     concentrated acids and alkalis)
Reworkability        Very difficult — must be mechanically removed
                     (scalpel, abrasive brush) or chemically dissolved
                     with specialised silicone solvents
Flexibility          Excellent — remains flexible at -65°C
Fungus resistance    Fair (some grades are listed as fungus-supporting)
Cure method          Moisture cure (RTV), UV cure, or heat cure
Application method   Spray, dip, selective coating
Typical thickness    50-200 µm
Cost                 Medium-high
Common use           High-temperature environments (under-bonnet automotive,
                     industrial motors), cryogenic applications, aerospace,
                     outdoor electronics exposed to UV
```

**Advantages:** The best choice for extreme temperature ranges and outdoor UV exposure.
Silicone does not crack at low temperatures (critical for outdoor or cryogenic applications)
and maintains flexibility over its full operating range.

**Disadvantages:** Silicone is the hardest coating to remove for rework. Once cured,
it requires physical scraping or a specialised silicone-dissolving solvent. It can also
outgas silicone compounds that contaminate nearby surfaces (problematic in optical
assemblies or clean-room environments). Silicone contamination prevents adhesion of
subsequent coatings and solder.

#### Polyurethane (Type UR)

```
Property             Value/Characteristic
-------------------  ----------------------------------------
IPC designation      Type UR
Dielectric strength  >80 V/mil (≈3.2 kV/mm)
Operating temp       -55°C to +125°C
Moisture resistance  Excellent
Chemical resistance  Good (resistant to fuels, lubricants, mild acids)
Reworkability        Moderate — removable with specialised solvents
                     (N-methyl-2-pyrrolidone, methylene chloride);
                     harder to remove than acrylic
Flexibility          Good
Fungus resistance    Good
Cure method          Two-component (moisture cure or chemical catalyst),
                     UV cure, or heat cure
Typical thickness    25-75 µm
Cost                 Medium
Common use           Automotive electronics (underdash), industrial controls,
                     marine electronics (bilge area, engine room)
```

**Advantages:** Good chemical resistance, especially to fuels and lubricants. Better
abrasion resistance than acrylic. Good balance of properties for automotive and marine
environments.

**Disadvantages:** More difficult to rework than acrylic. Two-component formulations
have a limited pot life after mixing. Some formulations are sensitive to moisture during
cure (moisture-curing UR can blister if applied to a humid PCB).

#### Epoxy (Type ER)

```
Property             Value/Characteristic
-------------------  ----------------------------------------
IPC designation      Type ER
Dielectric strength  >100 V/mil (≈4 kV/mm)
Operating temp       -55°C to +150°C (hard epoxy grades)
Moisture resistance  Excellent
Chemical resistance  Excellent (resistant to most solvents, acids, alkalis)
Reworkability        Very difficult to remove — essentially requires
                     mechanical abrasion or burning; not recommended
                     for products requiring field repair
Flexibility          Low (stiff, brittle)
Fungus resistance    Excellent
Cure method          Two-component (chemical catalyst) or UV cure
Typical thickness    50-100 µm
Cost                 Medium
Common use           Harsh chemical environments, some military applications.
                     Rarely used where any rework is anticipated.
```

**Advantages:** Hardest and most chemically resistant coating. Excellent long-term
stability. Used when the product will not require field repair and maximum chemical
protection is needed.

**Disadvantages:** Brittle — can crack and delaminate if the board flexes or is
subjected to thermal shock. Near-impossible to rework without destroying the board.
The CTE mismatch between epoxy coating and FR4/component packages generates stress
during temperature cycling.

#### Parylene (Type XY)

```
Property             Value/Characteristic
-------------------  ----------------------------------------
IPC designation      Type XY
Dielectric strength  >250 V/mil (≈10 kV/mm)
Operating temp       -200°C to +150°C (Parylene C, most common)
Moisture resistance  Exceptional (pinhole-free deposition)
Chemical resistance  Exceptional
Reworkability        Very difficult — laser ablation or O2 plasma etch
                     required for selective removal
Flexibility          Good
Pinhole coverage     Excellent — vapor deposition fills all voids
Thickness control    Very precise (1-25 µm typical)
Application method   Chemical vapour deposition (CVD) — batch process,
                     requires specialised equipment; cannot be done in-house
                     without capital investment
Cost                 High (process cost, not material cost)
Common use           Medical implantables, aerospace, military, harsh
                     environments where thin uniform coverage is critical
```

**Advantages:** Parylene is deposited as a vapour and polymerises on all surfaces,
including the insides of connectors, under components, and in blind vias. It provides
the most uniform and pinhole-free coverage of any conformal coating. Even at 1-2 µm
thickness, it is an effective moisture barrier.

**Disadvantages:** Expensive batch process. Requires masking of all areas that must
not be coated (connectors, test points, programming headers) before the deposition
chamber is loaded. Rework requires laser ablation, which is time-consuming and costly.

#### Summary Comparison

```
Property           Acrylic   Silicone  Polyurethane  Epoxy    Parylene
---------          -------   --------  -----------   -----    --------
Reworkability      Excellent Poor      Moderate      Poor     Very poor
Temp range (max)   125°C     200°C     125°C         150°C    150°C
Moisture resist.   Good      Excellent Excellent     Excellent Exceptional
Chemical resist.   Moderate  Good      Good          Excellent Exceptional
Flexibility        Moderate  Excellent Good          Poor      Good
Cost               Low       Med-high  Medium        Medium   High
Coverage (voids)   Moderate  Moderate  Moderate      Moderate Excellent
```

---

### Application Methods

**Aerosol/manual spray:**
Low capital cost, suitable for prototypes and small batches. Inconsistent coverage
thickness. Difficult to mask accurately. Human operator introduces variability.
Not suitable for production.

**Automated spray (conformal coating robot):**
A 3- or 5-axis robot with a spray nozzle follows a programmed path over the board,
applying coating to defined areas while avoiding others (connectors, test points,
components requiring no coating). Consistent coverage. Setup cost is the programmed
path per board design; ongoing cost is low.

**Dip coating:**
The board is dipped into a bath of coating material. Complete coverage of all surfaces.
Requires thorough masking of all areas that must not be coated. Poor for selective
application. Used when maximum coverage is required and rework access is not needed.

**Selective coating with fluid dispensing:**
A needle or atomised jet dispenses coating to specific pads or components. Very
precise placement; avoids connectors and test points without masking. Higher cost
per board than spray; slower. Used for high-value or complex boards.

---

### Design Considerations

#### Areas That Must Not Be Coated

Connectors, programming/JTAG headers, test pads, potentiometers, switches, speakers,
microphones, and any component requiring post-assembly adjustment must be masked
before coating and inspected after coating. Masked areas must be defined in the assembly
drawing.

Masking methods:
- Peelable masking (latex or silicone applied by robot, peeled after cure)
- Tape masking (for large areas, less accurate)
- Liquid mask (pin-selectively applied, peelable or dissolvable)
- Connector boots (rubber caps that fit over connector housings)

#### IPC-CC-830: The Industry Standard for Conformal Coating

IPC-CC-830 (Qualification and Performance of Electrical Insulating Compound for
Printed Board Assemblies) defines:
- Material qualification tests (humidity resistance, insulation resistance, thermal
  shock, chemical resistance, flammability)
- Acceptance criteria for coated assemblies
- Visual inspection criteria (holidays, dewetting, blistering)

**Common IPC-CC-830 acceptance criteria:**
```
Coverage:   Coating must cover all conductor surfaces in the coated area
            No holidays (uncoated voids) visible at 1-2× magnification
Thickness:  Defined by the project specification (typically 25-75 µm)
            Measured with a micrometer or eddy-current gauge on test coupons
Adhesion:   No lifting, blistering, or delamination after humidity test
            (typically 24-hour exposure at 65°C/95% RH)
Colour:     Uniform; no discolouration indicating contamination under the coating
Fluorescence: Most conformal coatings contain UV fluorescent tracers (absent in
              parylene and some clear coatings). UV lamp inspection under 365 nm
              light shows coated areas as fluorescent — a fast in-line QC method.
```

#### Thickness Requirements

Too thin: inadequate protection (holidays, pinhole failure modes, moisture ingress).
Too thick: risk of cracking, delamination under thermal cycling; increased CTE mismatch
stress; may encapsulate solder balls or bridges that are hidden under coating.

```
Recommended thickness by application:
  General industrial:          25-75 µm
  Severe environment:          75-125 µm
  IPC-A-610 Class 3 (high-rel): as specified, typically 50-75 µm per side
  Medical (IEC 60601-1):       as specified by the insulating compound class
```

#### Rework After Coating

Before reworking a component on a coated board:
1. Identify the coating type (check the product traveller or test with a solvent drop)
2. Remove coating from the rework area using the appropriate method:
   - Acrylic: solvent (MEK, acetone, or specialist acrylic stripper)
   - Silicone/epoxy: mechanical scraping with scalpel; specialist strippers
3. Perform the rework (component replacement, trace modification)
4. Clean the rework area thoroughly (flux residues are hygroscopic and will cause
   failure under the new coating)
5. Re-apply coating to the rework area, feathering into the existing coating boundary
6. Inspect the re-coated area under UV light and at the required magnification

---

## Best Practices

- Specify the conformal coating type, thickness, and exclusion zones on the **assembly
  drawing**, not just in a work instruction. The drawing travels with the design and
  is visible to the contract manufacturer.
- Perform **incoming adhesion tests** on each batch of conformal coating material.
  Coating formulations can change between batches; a brief adhesion test (cross-hatch
  test, ASTM D3359) confirms the coating bonds correctly to the PCB surface finish.
- Apply conformal coating **after all testing** is complete. Coating prior to functional
  test obscures components and prevents rework of found faults.
- **Clean the PCB before coating.** Flux residues, handling contamination, and moisture
  absorbed by the laminate all degrade adhesion and may cause blistering. Clean with
  an appropriate wash process and verify cleanliness with ion chromatography if
  required by the application standard.
- In high-humidity environments, specify **pre-bake** of PCB assemblies (typically
  125°C for 1-2 hours) before coating to drive out absorbed moisture that would
  otherwise cause blistering.

---

## Tier 1 — Fundamentals

### Question F1
**What are the four main types of conformal coating? For each, state its key advantage and its main limitation.**

**Answer:**

1. **Acrylic (AR):**
   Key advantage: Easiest to rework — dissolves readily in common solvents (MEK, acetone)
   without damaging the PCB or surrounding components.
   Main limitation: Poor resistance to solvents — unsuitable for environments where
   the product may be exposed to solvent-based chemicals.

2. **Silicone (SR):**
   Key advantage: Best extreme-temperature performance (-65°C to +200°C) and flexibility
   at all temperatures. Ideal for outdoor UV exposure and high-temperature environments.
   Main limitation: Very difficult to remove for rework; requires mechanical scraping
   or specialised solvents. Silicone outgassing can contaminate nearby surfaces.

3. **Polyurethane (UR):**
   Key advantage: Good chemical resistance (fuels, lubricants) and excellent moisture
   resistance. A good all-round choice for automotive and marine environments.
   Main limitation: More difficult to rework than acrylic; some two-component
   formulations have limited pot life and moisture sensitivity during cure.

4. **Epoxy (ER):**
   Key advantage: Maximum chemical and abrasion resistance. Hardest and most durable
   coating. Best for harsh chemical environments.
   Main limitation: Brittle — cracks under thermal cycling or board flexure. Very
   difficult to remove; essentially rules out field repair.

Parylene is sometimes listed as a fifth type. Its key advantage is pinhole-free vapour-
deposited coverage. Its main limitation is cost and the requirement for a specialised
deposition facility.

---

### Question F2
**Why must test points and programming headers be masked before conformal coating is applied? What happens if they are coated?**

**Answer:**

Test points must remain uncoated because:
1. **Electrical contact:** Test point pads are probed by flying probe testers, bed-of-nails
   fixtures, or manual probes during functional test. A coating over the pad creates an
   insulating layer that prevents reliable electrical contact. The probe tip may not
   break through the coating, or may create a ragged hole that prevents future access.
2. **Post-coating test invalidity:** If coating is applied before functional test, any
   faults found on test require rework, which then requires coating removal and re-application.
   Best practice: coat after all testing is complete.

Programming/JTAG headers must remain uncoated because:
1. Connector pins must make electrical contact with a mating connector or clip. Coating
   prevents this.
2. In field service, headers must be accessible for firmware updates or debugging.
   Coated headers cannot be used.

Connectors in general must not be coated on their mating surfaces, but the body/mounting
area may be coated for structural retention. This requires precise masking.

---

## Tier 2 — Intermediate

### Question I1
**A PCB for a marine application (bilge water pump controller) must operate at -20°C to +70°C, in high humidity (95% RH), with exposure to diesel fuel and salt spray. Which conformal coating type would you specify, and what application and inspection requirements would you include in the assembly specification?**

**Answer:**

**Recommended coating type: Polyurethane (UR)**

Justification:
- Temperature range -20°C to +70°C: All four standard types are suitable; silicone
  would only be needed if temperatures went below -55°C or above +130°C.
- High humidity (95% RH): Polyurethane and silicone both provide excellent moisture
  resistance. Acrylic is adequate but polyurethane offers better long-term performance.
- Diesel fuel exposure: Polyurethane has good resistance to fuels and lubricants.
  Silicone also resists diesel but is much harder to rework. Acrylic would be dissolved
  by diesel fuel contact — unsuitable.
- Salt spray: Polyurethane and silicone both handle salt spray well.

Silicone would be an alternative if the product were truly never reworked, but for a
commercial marine product where field service is expected, the easier reworkability of
polyurethane (compared to silicone) favours polyurethane.

**Assembly specification requirements:**

```
Conformal coating specification:
  Material:          Polyurethane (IPC-CC-830 Type UR), 1K UV-cure formulation
  Thickness:         50-75 µm (both sides where practical)
  Application:       Automated spray robot, programmed path
  Coverage:          All components and conductors in the coated area;
                     no holidays > 0.5 mm visible at 2× magnification
  Exclusion zones:   J1 (power connector) — full connector body
                     J2 (programming header) — full connector body
                     TP1-TP8 (test points) — 2 mm radius from pad centre
                     SW1 (mode switch) — full switch body
  Masking method:    Automated peelable mask on J1, J2; manual Kapton tape TP1-TP8
  Pre-bake:          125°C for 60 minutes before coating application
  Cure:              UV cure (2 J/cm² minimum at 365 nm) then heat cure at
                     60°C for 30 minutes for shadow areas not reached by UV
  Inspection:
    Visual (white light): no uncovered conductors, no blistering, no delamination
    UV inspection (365 nm): fluorescent coverage confirms all required areas coated
    Thickness check: eddy-current measurement on test coupon processed with each batch
    Adhesion:        Crosshatch test (ASTM D3359) per batch on test coupon
    Record:          Serial number, coating batch number, and UV inspection photograph
                     for each board
```

---

## Tier 3 — Advanced

### Question A1
**During in-circuit test (ICT) at the contract manufacturer, 8% of boards fail at a specific group of test points. The failure mode is high contact resistance between the ICT probe and the test pad. The boards have been conformal coated with acrylic before ICT. What is the root cause, and how should the process be corrected?**

**Answer:**

**Root Cause:**

The conformal coating was applied before in-circuit test. The ICT probe must pierce or
displace the coating to make electrical contact with the underlying copper pad. Even
though acrylic is relatively soft, the probe tip may:
1. Hydroplane on the coating rather than penetrating it (particularly with low-force
   probes or worn probe tips)
2. Penetrate but leave residue on the probe tip, increasing contact resistance
3. Create a larger penetration hole than the probe pin, allowing coating to spring
   back partially after the probe withdraws

The 8% failure rate (not 100%) indicates partial penetration is occurring — some
boards or probe tips are achieving adequate contact and some are not. This intermittency
is characteristic of coating-related contact resistance, not an electrical fault in
the boards themselves.

**Verification:**

Re-test the failing boards without conformal coating (if the test can be re-run after
locally removing the coating from the test pad areas). If the boards pass, the root
cause is confirmed as coating interference with ICT.

Alternatively, microscopically inspect a failing test point pad after ICT contact.
If the probe mark shows coating residue or a partial penetration crater rather than
a clean metallic contact mark, the diagnosis is confirmed.

**Process Correction:**

The correct process sequence is:

```
WRONG sequence: Coat → ICT → Functional test → Ship
RIGHT sequence: ICT → Functional test → Coat → Ship
```

Conformal coating must be applied after all electrical testing is complete. This is
standard practice in IPC-A-610. The product traveller must reflect this sequence.

**If the coating process cannot be moved (e.g., quality sign-off requires coating
before ICT in the workflow for regulatory reasons):**

Option A: Mask all ICT test points during coating (add test point exclusion to the
masking programme). Additional masking cost vs. current yield loss must be evaluated.

Option B: Use a coating penetrating probe design (spring-loaded probe tips with a
pointed geometry and higher probe force). This is a workaround, not a solution, and
increases pad wear.

Option C: Switch to a flying probe test rather than ICT — flying probe probes are
typically higher force and specifically designed for coated boards.

**Lesson for future designs:**
The assembly process specification must explicitly state the sequence of test and coating
steps. "Coat before ship" is correct; "coat before test" is an error. This should be
a standard process engineering review checkpoint when a new product is released to
manufacturing.

---

## Related Topics

- [EMC Basics](emc_basics.md)
- [Safety and Creepage](safety_and_creepage.md)
- [Environmental, REACH, RoHS](environmental_reach_rohs.md)
- [Assembly Considerations](../03_design_for_manufacturing/assembly_considerations.md)
- [PCB Fabrication Constraints](../03_design_for_manufacturing/pcb_fabrication_constraints.md)
