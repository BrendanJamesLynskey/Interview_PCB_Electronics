# DFM Quiz

Fifteen multiple-choice questions covering PCB fabrication constraints, SMT and
through-hole assembly, via types and selection, and panelisation. Each question has
four options. Explanations are provided in the Answer Key at the end.

---

## Questions

---

### Question 1

A PCB design uses a 0.2 mm minimum trace width on an inner signal layer. The PCB
fabricator's capability specification states a minimum inner-layer trace width of
0.1 mm. The design uses FR4 with 0.5 oz copper on inner layers. What is the main
risk of using 0.2 mm traces on a 0.5 oz inner layer?

A) The 0.2 mm trace is within the fabricator's capability but the current-carrying
   capacity is inadequate for any signal above 100 mA at 25°C, which may be a concern
   for power distribution traces.

B) There is no risk: 0.2 mm traces exceed the minimum specification and are therefore
   fabricated with full yield.

C) The 0.5 oz copper with 0.2 mm trace width is more susceptible to under-etching,
   resulting in traces that are narrower than intended and possibly open-circuited in
   extreme cases.

D) Inner-layer traces at 0.2 mm always require impedance control and cannot be used
   for standard signal routing without a special fabrication note.

---

### Question 2

During SMT reflow, what is the function of the solder paste's flux?

A) To provide the metallic solder alloy for the joint; the flux activator carries the
   tin, silver, and copper particles to the correct locations.

B) To remove oxide layers from the pad and component lead surfaces immediately before
   and during reflow, enabling wetting of molten solder to clean metallic surfaces.

C) To provide mechanical adhesion of the paste deposit to the pad prior to component
   placement, preventing component movement.

D) To control the reflow temperature profile by absorbing latent heat, ensuring the
   solder reaches liquidus uniformly across the board.

---

### Question 3

A PCB assembly uses a 0402 (1 mm × 0.5 mm) MLCC adjacent to a QFN-32 device. Both are
reflow-soldered in the same SMT pass. The QFN pad is 5 mm × 5 mm with an exposed pad
underneath. What thermal consideration must the reflow profile account for?

A) The 0402 component will tombstone (stand on end) if the reflow profile heats the
   PCB too slowly, because slow heating gives the solder on one end more time to melt
   before the other.

B) The QFN exposed pad requires significantly more heat energy to reflow than the 0402
   pads; a single standard profile will cause the 0402 component to overheat before
   the QFN exposed pad reaches liquidus.

C) The QFN and 0402 have different thermal masses; the reflow profile must include a
   soak zone long enough to ensure both components reach liquidus temperature within
   the manufacturer's specification, without exceeding any component's maximum
   reflow temperature.

D) The 0402 component will absorb heat from the QFN exposed pad by radiation, causing
   the 0402 to experience a higher peak temperature than if placed remotely.

---

### Question 4

A designer plans to use a 0.3 mm drill diameter via to connect a signal trace between
the top and bottom layers of a 1.6 mm thick PCB. What is the minimum finished annular
ring width required by IPC-6012 Class 2, and what is the resulting minimum via pad
outer diameter?

A) Minimum annular ring: 0.05 mm. Minimum pad outer diameter: 0.3 + 2 × 0.05 = 0.4 mm.

B) Minimum annular ring: 0.1 mm. Minimum pad outer diameter: 0.3 + 2 × 0.1 = 0.5 mm.

C) Minimum annular ring: 0.2 mm. Minimum pad outer diameter: 0.3 + 2 × 0.2 = 0.7 mm.

D) There is no minimum annular ring for vias that are not on pads; the pad size is
   determined only by the drill diameter.

---

### Question 5

A PCB assembly has two adjacent SMT pads with 0.4 mm centre-to-centre spacing.
The stencil aperture is 0.3 mm × 0.4 mm for each pad. Solder paste bridges between
the two pads are observed after reflow. Which TWO changes would most directly reduce
bridging, without modifying the pad dimensions on the PCB?

A) Reduce the stencil aperture width to 0.25 mm and reduce the stencil thickness from
   0.15 mm to 0.12 mm, to decrease the volume of paste deposited between adjacent pads.

B) Switch from no-clean flux solder paste to water-washable flux paste, which provides
   better wetting control.

C) Increase the peak reflow temperature by 10°C to improve solder flow and allow
   excess solder to self-drain.

D) Increase the stencil aperture to 0.35 mm × 0.45 mm to ensure full pad coverage,
   relying on surface tension to prevent bridging.

---

### Question 6

A board requires a controlled impedance 50 Ω microstrip trace on the top layer. The
PCB fabricator measures the impedance as 48 Ω on the test coupon included in the panel.
What is the most appropriate action?

A) Reject the boards and require the fabricator to re-fabricate with corrected trace
   widths, because 48 Ω falls outside the ±5% impedance tolerance (47.5-52.5 Ω).

B) Accept the boards, because 48 Ω is only 4% below nominal (within a typical ±10%
   commercial fabrication tolerance) and the test coupon measurement reflects the same
   production conditions as the signal traces.

C) Calculate the insertion loss at the highest signal frequency to determine whether
   the impedance mismatch of 2 Ω causes a reflection coefficient that falls outside
   the link budget.

D) Manually adjust each trace width on the produced boards using a micro-abrasion tool
   to bring the impedance to exactly 50 Ω.

---

### Question 7

A PCB design includes a 0.4 mm pitch QFN-64 component. The assembly house asks for the
paste layer design. What stencil design guideline applies to the exposed thermal pad?

A) The exposed pad stencil aperture should be a single opening matching the full pad
   area, to maximise solder coverage and thermal conductivity.

B) The exposed pad stencil aperture should be a segmented array of smaller openings
   (typically 50-70% coverage of the pad area) to allow flux outgassing and prevent
   excessive voiding.

C) The exposed pad should not receive any solder paste; it should be soldered using
   selective soldering after QFN placement.

D) The exposed pad stencil should be designed at 100% coverage if the PCB has thermal
   vias beneath the pad, and 70% if there are no vias.

---

### Question 8

A PCB is designed in a 100 mm × 80 mm outline. The design must be assembled in a panel
for automated SMT. The assembly house's conveyor handles panels from 200 mm × 200 mm
to 350 mm × 250 mm. Which panelisation method is most appropriate, and what is the
minimum consideration for the breakaway tabs?

A) V-score panelisation in a 2 × 3 array (six boards per panel), with V-scores running
   the full length and width of the panel. Breakaway tabs are not needed because V-score
   removes the need for physical tabs.

B) Tab-routed panelisation with a 3 × 2 array, using breakaway tabs at least 5 mm wide
   with perforation holes, and a 5 mm border around the panel perimeter. V-score is not
   suitable because the board outline is irregular.

C) V-score panelisation in a 3 × 2 array (six boards, panel dimensions ~300 mm × 160 mm),
   with score depth set to leave approximately one-third of the board thickness, and
   consideration that V-scoring must use straight cuts — not suitable if the board has
   irregular edges.

D) The board dimensions cannot be panelised because the board width of 100 mm exceeds
   the standard panel half-width.

---

### Question 9

A designer proposes using ENIG (Electroless Nickel Immersion Gold) surface finish for
a mixed-technology PCB assembly (SMT components plus press-fit connectors). What is
the BLACK PAD defect and what process control prevents it?

A) Black pad is the oxidation of the gold finish during storage; it is prevented by
   packaging the PCBs in nitrogen-purged bags immediately after fabrication.

B) Black pad is excessive gold plating that reduces solderability; it is prevented by
   controlling gold thickness to below 0.1 µm.

C) Black pad is a brittle nickel phosphide layer (Ni₃P) that forms at the nickel surface
   due to corrosion during the immersion gold process, creating a weak fracture plane
   under the solder joint. It is prevented by controlling the phosphorus content of the
   nickel bath (7-9% P for ductile nickel), gold plating bath chemistry, and process
   monitoring.

D) Black pad refers to a visual darkening of the solder mask adjacent to ENIG pads
   during reflow; it is an aesthetic issue with no structural consequence.

---

### Question 10

A PCB has a via-in-pad design (VIP) where microvias are used directly on BGA ball pads.
The IPC-4761 specification defines seven via protection types. Which type is required
for a via in a SMT pad that will be soldered in a reflow oven?

A) Type I (tented via): a dry film solder mask tent over the via opening provides
   adequate sealing for BGA pad vias.

B) Type V (filled via): a via filled with conductive or non-conductive epoxy but not
   capped, providing a flat surface suitable for direct BGA ball soldering.

C) Type VII (filled and capped via): the via is filled with material and capped with
   copper plating, providing a fully planar pad surface. This prevents solder wicking
   and flux entrapment during reflow.

D) Type III (tented and covered): a solder mask tent applied to one or both sides of
   the via, providing adequate protection without fill.

---

### Question 11

A PCB manufacturer specifies an aspect ratio limit of 10:1 for plated through-holes.
The board thickness is 2.4 mm. What is the minimum drill diameter that should be
specified for reliable plating?

A) 0.1 mm — the aspect ratio limit is a guideline, not a hard rule, so the smallest
   commercially available drill can always be used.

B) 0.24 mm — this gives exactly 10:1 (2.4 mm / 0.24 mm = 10), which is at the limit
   and should be avoided; the practical minimum is 0.3 mm for reliable copper plating
   continuity.

C) 0.5 mm — the 10:1 limit requires 2.4 mm / 10 = 0.24 mm minimum, but 0.5 mm is the
   industry-standard minimum for standard PCB fabrication.

D) 1.0 mm — blind vias are required for any hole smaller than 1 mm diameter on a
   2.4 mm thick board.

---

### Question 12

During manual visual inspection of a freshly assembled board, a technician observes
that several 0603 resistors have tilted at approximately 30° from the pad plane (one
end elevated). No bridges or opens are visible. What assembly defect has occurred and
what is the most likely cause?

A) Pad crater: the PCB pad has detached from the laminate, causing the component to
   tilt. Cause: excessive rework iron temperature.

B) Tombstoning (Manhattan effect): one end of the resistor has been pulled upright by
   the surface tension of the molten solder on one pad before the other pad's solder
   reaches liquidus. Cause: unequal heating of the two pads during reflow (one pad has
   greater thermal mass, a larger copper area, or receives less heat from the oven
   conveyor profile).

C) Component floating: the flux carrier dissolved during preheat and allowed the
   component to float off-centre on the solder paste. Cause: too much flux in the
   paste formulation.

D) Solder balling: excess solder has formed a ball on one pad and lifted the component.
   Cause: stencil aperture is larger than the pad.

---

### Question 13

A design for a high-volume consumer product requires the PCB to be depanelised
(separated from the panel) after assembly. The board has SMT components within 2 mm
of the board edge. What depanelisation method introduces the LEAST mechanical stress
on the near-edge components?

A) V-score snap (breaking along the V-groove by hand or with a separator tool), because
   the snap is fast and requires no tooling.

B) Tab routing with manual breakout (bending the tabs to break the perforation holes),
   because no tooling is needed.

C) Precision routing with a router bit after assembly (routing along the board outline
   after SMT), because the router bit cuts cleanly without flex stress on the board.

D) Laser scoring and snap, because laser energy is focused precisely and the snap is
   along a thin scored line.

---

### Question 14

A PCB layout contains a 0.1 mm minimum clearance between an SMT pad and an adjacent
trace. The PCB fabricator's minimum clearance capability is 0.1 mm. What solder mask
manufacturing consideration must the designer address?

A) A solder mask defined (SMD) pad must have solder mask opening exactly equal to the
   copper pad dimensions to prevent solder bridging to the adjacent trace.

B) The solder mask expansion (the margin by which the mask opening is larger than the
   copper pad) must not encroach on the 0.1 mm trace clearance. If the mask expansion
   is 0.05 mm on each side, the mask opening is 0.1 mm wider than the pad. If the
   adjacent trace is only 0.1 mm from the pad edge, the mask opening extends to within
   0.05 mm of the trace — which may expose the trace if registration tolerance is poor.

C) The 0.1 mm clearance is adequate as long as the solder paste aperture in the stencil
   does not extend over the trace.

D) No solder mask consideration is required because the trace is on the opposite side of
   the PCB from the SMT pad.

---

### Question 15

A designer uses a 0.5 mm pitch BGA on a two-layer PCB. The BGA has a 15 × 15 ball
array (225 balls). Why is a two-layer PCB fundamentally inadequate for this component,
and what is the minimum recommended layer count?

A) A two-layer PCB cannot support the BGA mechanically; at least four layers are needed
   for adequate stiffness. Layer count does not affect routing.

B) A 0.5 mm pitch 15 × 15 BGA requires fanout vias for every ball; the via land diameter
   plus the trace width from the via to the pad leaves insufficient routing channels
   on two layers for the inner rows. Minimum recommended is four layers (with at least
   two inner routing layers accessible via vias) for this array size, and HDI (microvia)
   technology for pitch ≤ 0.5 mm.

C) Two-layer PCBs cannot achieve the controlled impedance required for BGA signal
   integrity at any pitch.

D) A 0.5 mm pitch BGA requires ENIG surface finish, which is not available on two-layer
   boards.

---

## Answer Key

---

**Question 1 — Answer: A**

IPC-2152 current-carrying capacity tables show that a 0.2 mm trace in 0.5 oz copper
(17.5 µm thick) carries approximately 100 mA with a 10°C temperature rise in a
standard environment. This is adequate for signal traces but insufficient for any power
distribution trace carrying more than 100 mA. The fabrication risk from under-etching
(C) is real but secondary — at 0.2 mm (which is double the 0.1 mm minimum), the etch
process has significant headroom. The more practical concern for the designer is the
current-carrying capacity.

- B is technically correct that the design is within fabricator capability, but it does
  not identify the relevant risk.
- C is a real manufacturing concern at minimum trace widths but less significant at
  0.2 mm (twice the minimum).
- D is incorrect: impedance control is not mandatory for all traces; it is specified by
  the designer only where required.

---

**Question 2 — Answer: B**

Flux in solder paste performs the critical function of removing metal oxides from the
solder pad (copper or nickel-gold finish) and the component lead or termination. Molten
solder cannot wet an oxidised surface. The flux activator chemically reacts with copper
oxide and other oxides at elevated temperature, clearing the surface just as the solder
reaches liquidus and allowing intermetallic bonding to occur.

- A is incorrect: flux does not contain the solder alloy particles; the metal particles
  are the solder component of the paste.
- C describes an adhesive property — some flux formulations do provide tack (which helps
  hold components before reflow), but this is a secondary function, not the primary one.
- D is incorrect: flux does not control the reflow temperature profile by heat absorption.

---

**Question 3 — Answer: C**

The reflow profile must include a soak zone to allow the entire board, including the
thermally massive exposed pad of the QFN, to come up to temperature before entering the
reflow zone. Without an adequate soak, the small 0402 components (low thermal mass) reach
liquidus while the exposed pad (high thermal mass) has not yet reached liquidus. If the
oven temperature is increased to force the exposed pad to reflow, the 0402 may exceed
its maximum rated reflow temperature. The soak zone balances this temperature spread.

- A is true that tombstoning can occur with slow heating, but it is a consequence of
  unbalanced heating of the two ends of a small component, not of the overall heating
  rate.
- B has the thermal mass problem reversed: the QFN exposed pad is thermally massive and
  requires MORE energy, not less; the 0402 is low mass and reaches temperature faster.
- D is incorrect: radiation from the exposed pad to a 0402 component is negligible for
  the small distances involved and cannot cause meaningful excess temperature.

---

**Question 4 — Answer: B**

IPC-6012 Class 2 (which covers most commercial electronics) specifies a minimum annular
ring of 0.1 mm for external layers (Class 2, Table 3-3). The minimum pad outer diameter
is therefore:

```
Pad OD = Drill diameter + 2 × annular ring = 0.3 + 2 × 0.1 = 0.5 mm
```

Class 3 (high-reliability, aerospace, military) requires a larger annular ring and
tighter tolerances.

- A (0.05 mm) is below the IPC-6012 Class 2 minimum. Some high-density designs use
  smaller annular rings under Class 2 tolerances if the manufacturer has demonstrated
  process capability, but the IPC-6012 requirement is 0.1 mm.
- C (0.2 mm) exceeds the minimum and would be used for Class 3 or for land patterns
  with larger mechanical requirements.
- D is incorrect: IPC-6012 specifies annular ring requirements for all plated through-holes,
  including signal vias.

---

**Question 5 — Answer: A**

Reducing the stencil aperture width from 0.3 mm to 0.25 mm decreases the solder paste
volume deposited on each pad, reducing the amount of solder available to bridge to the
adjacent pad. Reducing stencil thickness from 0.15 mm to 0.12 mm further reduces paste
volume. Together, these changes directly address the root cause of bridging, which is
excess paste volume.

- B is incorrect: flux chemistry affects wetting and spread of the solder, but switching
  flux type does not reliably prevent bridging caused by excessive paste volume.
- C is incorrect: increasing peak reflow temperature can cause solder to flow more freely
  under increased surface tension, potentially worsening bridging; it does not
  reliably prevent it.
- D is incorrect: increasing the aperture size deposits more paste and increases the
  probability of bridging; surface tension cannot reliably prevent bridging with
  excess paste volume at 0.4 mm pitch.

---

**Question 6 — Answer: B**

A tolerance of ±10% is typical for commercial-grade controlled impedance PCBs. At 48 Ω,
the deviation is (50 − 48)/50 = 4%, which falls within the typical ±10% commercial
tolerance and is within the tighter ±5% tolerance commonly specified for high-speed
designs. The test coupon is processed on the same panel under identical conditions,
so it represents the actual impedance of the signal traces.

- A would be appropriate if ±5% were strictly required and the deviation exceeded this
  (48 Ω is exactly at the ±5% lower boundary of 47.5 Ω). If the specification requires
  ±5%, a designer must evaluate whether 48 Ω meets the system requirements.
- C is a valid additional engineering check but is not the "most appropriate" first
  action when the measurement is within standard tolerance.
- D is not a real manufacturing technique; individual trace width adjustment after
  fabrication is not feasible.

---

**Question 7 — Answer: B**

A segmented stencil aperture (array of small openings covering 50-70% of the exposed
pad area) is the industry-standard design for exposed pad devices. The webs between the
segments provide escape paths for flux gases generated during reflow. If the aperture
is a single solid opening (option A), the outgassing flux has no escape path, and the
gas pockets trapped under the molten solder create voids. Voiding above 25-30% of the
pad area significantly increases thermal resistance.

- A is incorrect because a solid aperture causes excessive voiding.
- C is incorrect: the exposed pad is designed to be soldered and provides the primary
  thermal and ground connection for QFN devices.
- D is partially correct in that the coverage percentage may be adjusted based on the
  presence of thermal vias (higher coverage with vias to prevent excessive paste volume
  wicking into the vias), but 100% coverage is never correct regardless of via design.

---

**Question 8 — Answer: C**

V-score panelisation is appropriate for boards with straight edges (rectangular outline).
A 3 × 2 array of 100 mm × 80 mm boards gives a panel of 300 mm × 160 mm — within the
assembly house's 200-350 mm × 200-250 mm range (note: 160 mm is slightly under 200 mm
minimum width; a 2 mm border on each side would bring it to 164 mm, still under 200 mm).
In practice, a 2 × 3 array (160 mm × 240 mm with border) or 3 × 2 (164 mm × 306 mm
with border) may be adjusted with larger borders to reach the minimum panel size.

The key point: V-score is appropriate for rectangular boards; score depth at one-third
of board thickness is standard; straight cuts are the constraint for V-scoring.

- A is partially correct about V-score for rectangular boards but the array size and
  panel dimensions need verification against the assembly house minimum.
- B proposes tab routing for a rectangular board, which is more expensive and less
  accurate than V-scoring for rectangular outlines.
- D is incorrect: 100 mm does not exceed any standard panel half-width.

---

**Question 9 — Answer: C**

Black pad is a critical ENIG failure mode. It is caused by corrosion of the nickel
surface during the immersion gold deposition step. The gold displacement reaction
(Au displacing Ni at the surface) can be aggressive if the bath chemistry is not
controlled, particularly at grain boundaries and high-phosphorus regions of the nickel.
The corroded nickel forms a brittle, phosphorus-rich layer (Ni₃P) that provides a
fracture plane under solder joints. The joint appears to solder normally but fails
under mechanical stress (shock, vibration, thermal cycling). Prevention is through
process control: phosphorus content of the nickel bath, gold bath activity, and
regular bath analysis.

- A describes a real (though less critical) storage issue but is not black pad.
- B confuses cause and effect; excessive gold thickness is measured to avoid excessive
  gold embrittlement of solder joints, but this is a different mechanism from black pad.
- D is incorrect: black pad is a structural defect, not merely aesthetic.

---

**Question 10 — Answer: C**

IPC-4761 Type VII (filled and capped via) is the correct choice for via-in-pad on SMT
surfaces. The via must be completely filled (typically with non-conductive resin or
copper) and then capped with electroplated copper to provide a planar, continuous
metallic surface flush with the pad. Without the copper cap, the filled resin may
contract during reflow and leave a small depression that traps flux and prevents uniform
solder joint formation.

- A (Type I, tented): a solder mask tent cannot withstand the pressure of solder wicking
  and will fail, allowing solder to wick into the via.
- B (Type V, filled only, not capped): without the copper cap, the surface is not planar
  and is not fully metallic; BGA ball attachment requires a copper surface.
- D (Type III, tented and covered): similar limitation to Type I — not suitable for SMT
  pad vias.

---

**Question 11 — Answer: B**

The aspect ratio is defined as board thickness divided by drill diameter. At 10:1:

```
Minimum drill diameter = Board thickness / Aspect ratio = 2.4 mm / 10 = 0.24 mm
```

However, 0.24 mm is at the absolute limit and reliable copper plating throughout the
hole depth becomes problematic at this ratio. Industry practice uses 10:1 as the
absolute maximum; for reliable production, 8:1 is preferred. At 2.4 mm board thickness:

```
Reliable minimum: 2.4 / 8 = 0.3 mm
```

0.3 mm is the practical minimum for this board thickness, meeting the 10:1 hard limit
(2.4/0.3 = 8:1) with margin.

- A is incorrect: the aspect ratio limit is an important manufacturing constraint based
  on acid/copper bath solution exchange in the drill hole.
- C (0.5 mm) is very conservative; 0.3 mm is achievable at 2.4 mm board thickness.
- D is incorrect: blind vias are not required merely because of board thickness; they
  are used for HDI designs where through-hole is space-inefficient.

---

**Question 12 — Answer: B**

Tombstoning (also called the Manhattan effect) is a well-documented SMT defect where
a small component stands nearly vertical on one end. It is caused by asymmetric wetting
forces: the solder on one pad melts and reflows before the other, and the surface
tension of the first-to-melt solder pulls that end of the component up. This occurs
most commonly with small passive components (0402, 0603, 0805) when the two ends of
the component have unequal thermal environments.

Prevention: symmetric pad design, symmetric copper fill on both pads (avoid large
ground fill on one pad and none on the other), uniform reflow heating from both sides,
and ensuring both pad connections have similar thermal mass.

- A (pad crater) is a different defect involving pad detachment from the laminate,
  typically caused by rework.
- C is a different defect; floating is caused by flux carrier evaporation with very
  fluid paste, resulting in lateral displacement rather than tilting.
- D (solder balling) is excess solder forming separate balls on or near the joint,
  not related to component tilting.

---

**Question 13 — Answer: C**

Precision routing after assembly (using a router bit to cut along the board outline
after SMT reflow) imparts the least mechanical stress on near-edge components because
the router bit cuts without bending or stressing the PCB. The PCB remains flat
throughout the process.

V-score snap (A) and tab breakout (B) both require bending the PCB, which generates
flexural stress that propagates from the breakout region into the board. Components
within 2 mm of the break zone can experience strain sufficient to cause solder joint
fracture, especially for large capacitors and resistors.

Laser scoring (D) produces a higher-quality cut than mechanical V-scoring but still
requires a snap step, which introduces the same bending stress.

- A is fast but imparts bending stress.
- B requires hand bending — the most variable and potentially highest stress method.
- D still requires a snap step.

---

**Question 14 — Answer: B**

Solder mask expansion is the amount by which the mask opening is enlarged beyond the
copper pad edge. A typical expansion is 0.05 mm per side (0.1 mm total). If the mask
opening is 0.1 mm wider than the pad and the adjacent trace is only 0.1 mm from the
pad edge, the mask opening extends over the trace. If the mask registration is off by
0.05 mm (typical mask registration tolerance), the exposed trace may receive solder
paste and create a solder bridge.

The solution is to use solder mask defined (SMD) pads (mask opening equal to pad size,
no expansion) or to reduce the mask expansion for this specific area. Alternatively,
the adjacent trace can be moved further away, but the question specifies no PCB changes.

- A is partially relevant but does not address the registration tolerance issue.
- C is incorrect: the stencil aperture alignment is important but the solder mask
  opening also participates in defining which conductors are exposed during reflow.
- D is incorrect: the question specifies that the trace is on the same side as the pad
  (both are copper features on the same layer, implied by the 0.1 mm clearance between them).

---

**Question 15 — Answer: B**

A 0.5 mm pitch BGA with a 15 × 15 array has inner rows that cannot be reached by
surface routing alone. Fanout vias (dog-bone pattern) for each ball pad require a via
land plus a trace. At 0.5 mm pitch, the geometry between adjacent ball centres leaves
insufficient space for a through-hole via land (typically 0.4-0.5 mm diameter) plus
a routable trace width (typically 0.1-0.15 mm) plus the required clearances. Inner
rows simply cannot be reached on two copper layers without HDI (microvia) technology.
Four layers with inner routing layers provide the needed channel density, and microvias
(laser-drilled, 0.1 mm drill) allow via-in-pad designs without consuming routing channels.

- A is incorrect: layer count does not primarily govern mechanical stiffness; the copper
  and prepreg stackup determine mechanical properties, not just layer count.
- C is partially valid (impedance control is easier with more layers) but is not the
  fundamental reason two layers are inadequate.
- D is incorrect: ENIG and other surface finishes are available on any layer count.

---

## Related Topics

- [PCB Fabrication Constraints](../03_design_for_manufacturing/pcb_fabrication_constraints.md)
- [Assembly Considerations](../03_design_for_manufacturing/assembly_considerations.md)
- [Via Types and Selection](../03_design_for_manufacturing/via_types_and_selection.md)
- [Panelisation](../03_design_for_manufacturing/panelisation.md)
