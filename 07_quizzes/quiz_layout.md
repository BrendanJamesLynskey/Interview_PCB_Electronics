# Layout Quiz

Fifteen multiple-choice questions covering PCB stackup design, component placement,
signal routing, return current paths, and thermal management. Each question has four
options. Explanations are provided in the Answer Key at the end.

---

## Questions

---

### Question 1

A four-layer PCB stackup is defined as: Top copper — Prepreg — Ground plane —
Core — Power plane — Prepreg — Bottom copper. A high-speed signal is routed on the
top layer. What is the primary return current path for this signal at 100 MHz?

A) The return current flows through the power plane (layer 3) because power and ground
   are at the same AC potential through the decoupling capacitors.

B) The return current flows directly beneath the signal trace on the ground plane
   (layer 2), following the path of minimum loop inductance.

C) The return current distributes equally across all ground connections on the board
   because copper is a conductor with negligible resistance.

D) The return current flows through the nearest board edge because that is the shortest
   path from the signal destination back to the source.

---

### Question 2

A designer routes a high-speed clock signal (50 MHz) across a split in the ground plane.
The split separates the analogue ground area from the digital ground area. What is the
most likely consequence?

A) The split has no effect on the clock signal because the return current can find an
   alternative path around the split through nearby vias.

B) The return current must detour around the split, creating a large current loop that
   radiates as a magnetic dipole at 50 MHz and its harmonics, increasing EMI emissions.

C) The split in the ground plane attenuates the clock signal by reflecting part of the
   signal energy at the impedance discontinuity.

D) The clock signal is slowed because the split increases the effective dielectric
   constant of the substrate.

---

### Question 3

What is the recommended maximum stub length for a via on a 10 Gbit/s serial link with
a rise time of approximately 35 ps? Assume signal propagation velocity is 180 ps/mm.

A) 50 mm — the stub is small compared to the wavelength at 10 GHz.

B) 0.6 mm — a stub longer than this creates a resonant notch within the signal's
   bandwidth that causes significant attenuation and signal integrity degradation.

C) 10 mm — the stub is only relevant when it exceeds one-quarter wavelength at the
   fundamental frequency.

D) There is no length restriction because the via is on the same net as the signal
   and signal integrity tools automatically compensate for via stubs.

---

### Question 4

A six-layer PCB has the stackup: Signal — GND — Signal — Power — GND — Signal.
Two 100 Ω differential pairs are routed on layers 1 and 6 respectively. Which routing
layer provides better common-mode rejection and lower EMI?

A) Layer 6, because routing on the bottom layer is shielded from above by the internal
   ground and power planes.

B) Layer 1, because the top layer has a larger ground reference area and better plane
   coupling due to the tighter prepreg spacing to layer 2.

C) Both layers are equivalent because each has a reference plane on only one side and
   returns are equidistant.

D) Neither layer is appropriate for high-speed differential pairs; all differential
   signals should be routed on internal signal layers for shielding.

---

### Question 5

A BGA device has 0.8 mm ball pitch with a 10 × 10 array. The PCB uses a standard
4-layer stackup. Vias have a minimum annular ring of 0.1 mm and a drill diameter of
0.3 mm, giving a finished via diameter of 0.3 mm. What technique is required to route
signals from the inner rows of the BGA?

A) Dog-bone fanout: signals from inner rows are routed on the top layer between the outer
   rows of vias, and via down to an inner layer only for the outermost row.

B) There is no additional technique needed; all signals from a 10 × 10 BGA at 0.8 mm
   pitch can be routed on the top layer with standard trace widths.

C) Dog-bone fanout for outer rows and microvias (laser drilled, < 0.15 mm drill) for
   inner rows where insufficient gap exists for through-hole vias plus trace routing.

D) The BGA must be redesigned to increase ball pitch to 1.0 mm before standard PCB
   routing rules can be applied.

---

### Question 6

A designer places the output capacitor of a buck converter 10 mm away from the
converter's feedback sense point, with 5 mm of trace between them. What is the most
likely design consequence?

A) The output capacitor placement has no effect on regulation; only the total capacitance
   value matters for loop stability.

B) The trace resistance and inductance between the capacitor and the feedback point
   introduce voltage drop and phase shift that the compensation network did not account
   for, potentially causing a loop stability problem or poor transient response.

C) The converter will have higher output ripple because the capacitor's ESR is increased
   by the trace resistance.

D) The switching noise from the converter will be attenuated by the trace inductance
   before reaching the output load.

---

### Question 7

Two parallel microstrip traces are routed adjacent to each other over a ground plane.
Trace A carries a 100 MHz aggressor clock signal. Trace B is a victim analogue signal
with 1 kΩ source impedance. The traces are 0.1 mm apart, each 0.15 mm wide, on a
substrate with 0.2 mm dielectric height. What physical design change most reduces
near-end cross-talk (NEXT)?

A) Add a ground trace between the two signals, connected to the ground plane by vias
   at both ends. This reduces capacitive and inductive coupling by interposing a
   grounded conductor.

B) Increase the trace width of the aggressor signal to reduce its characteristic
   impedance and ground the additional copper area.

C) Route the victim signal on the opposite face of the PCB (bottom layer) directly
   below the aggressor on the top layer.

D) Add a series resistor on the aggressor signal to slow its edge and reduce the
   dV/dt that drives capacitive coupling.

---

### Question 8

A switching regulator with a 48 V input and a 5 V/10 A output is placed on a PCB.
The switching node (VSW) transitions between 0 V and 48 V at 200 kHz. Where should
the input decoupling capacitor for this converter be placed?

A) At the input connector, as close to the board entry point as possible, to filter noise
   before it enters the PCB.

B) As close as possible to the converter's VIN and PGND pins, forming the smallest
   possible loop area with those pins and the switching devices, to minimise the
   high-frequency current loop that radiates at the switching frequency.

C) Adjacent to the output inductor, because the inductor's magnetic field couples to
   nearby capacitors and the close placement compensates for this coupling.

D) Distributed equally across the board to prevent any single point of high current
   density.

---

### Question 9

A four-layer PCB has a solid ground plane on layer 2 and a split power plane on layer 3
(two separate power domains: 3.3 V and 1.8 V). A high-speed signal trace on layer 1
crosses the boundary of the split power plane. What is the recommended design rule for
this trace?

A) The signal trace must never cross the split in the power plane because this always
   causes immediate signal integrity failure.

B) The trace may cross the split provided that there are stitching vias connecting the
   two sections of the power plane on each side of the split.

C) The signal trace on layer 1 references the ground plane on layer 2, not the power
   plane on layer 3. The return current follows the ground plane directly below the trace
   on layer 2, which is a solid plane. Crossing the power plane split on layer 3 is
   permissible for this signal, with no cross-talk penalty provided the layer-2 ground
   plane is uninterrupted.

D) The signal should be re-routed to layer 4 (bottom layer) to avoid proximity to the
   split power plane.

---

### Question 10

A designer calculates that the PCB must dissipate 4 W from an IC in a QFN-48 package
with a 7 mm × 7 mm exposed pad. The PCB is two-layer FR4. What is the MOST effective
single thermal management technique for this scenario?

A) Apply a thick conformal coating over the IC package to improve heat transfer to the
   air above the component.

B) Add a dense array of thermal vias (plated through-holes, 0.3 mm drill, 1.0 mm pitch)
   beneath the exposed pad, connected to a large copper fill on the bottom layer, to
   spread heat across the board and increase surface area for convection.

C) Place a 100 Ω resistor in series with the IC power pin to limit the current and
   therefore the power dissipation.

D) Increase the PCB thickness from 1.6 mm to 3.2 mm to increase the thermal mass and
   slow the temperature rise.

---

### Question 11

A USB 3.0 SuperSpeed differential pair (5 Gbit/s) requires 90 Ω differential impedance.
The PCB stackup has a dielectric height of 0.1 mm to the reference plane and a dielectric
constant (Dk) of 4.3. Using the microstrip formula, which trace geometry is closest to
achieving 90 Ω differential impedance?

A) Single-ended traces of 100 µm width with 100 µm spacing (centre-to-centre 200 µm).

B) Single-ended traces of 200 µm width with 150 µm spacing.

C) Single-ended traces of 100 µm width with 200 µm spacing.

D) Single-ended traces of 50 µm width with 100 µm spacing.

---

### Question 12

A 40-pin connector is placed at the edge of a PCB. Signals exit the PCB through the
connector. Some signals are 5 V GPIO at 1 MHz; others are 3.3 V SPI at 25 MHz. In which
order should signal types be assigned to connector pins for best EMI performance?

A) Place all high-speed signals on pins at the centre of the connector to maximise the
   distance from the board edge and reduce radiation.

B) Alternate ground pins between every signal pin (or every two signal pins) throughout
   the connector, and assign the lowest-speed signals to the outermost pins (furthest
   from the board).

C) Place power supply pins at both ends of the connector and concentrate all signals in
   the middle to benefit from the shielding of the power supply rails.

D) Assign all signals with common direction (all outputs together, all inputs together)
   to reduce connector-level cross-talk.

---

### Question 13

A PCB designer terminates a single-ended 50 Ω transmission line with a 50 Ω resistor
to ground at the receiver end. The trace is 100 mm long and the signal has a risetime
of 1 ns. Is parallel termination the correct approach, and what is the trade-off?

A) Yes — parallel termination at the receiver eliminates reflections because it matches
   the line impedance. The trade-off is that the termination resistor draws static current
   whenever the signal is in the high state (VCC/50 Ω), increasing power consumption.

B) No — parallel termination should always be at the source, not the receiver.
   A source termination resistor of 50 Ω in series at the driver correctly terminates
   the line.

C) Yes — but the termination resistor must be a matched pair (two 25 Ω resistors in
   series from VCC to GND with the mid-point at the signal) to prevent DC current flow
   and to present the correct DC operating point.

D) No — for a signal with 1 ns rise time and a 100 mm trace, the one-way flight time
   (approximately 0.6 ns) is less than the rise time, so the trace behaves as a lumped
   element and termination is not required.

---

### Question 14

A power supply designer routes the switching current loop of a flyback converter (primary
side: MOSFET + input capacitor + transformer primary) on a PCB. What is the correct
design priority for this loop?

A) Route the transformer primary trace first because it carries the most inductance and
   must be impedance-controlled.

B) Minimise the physical area enclosed by the loop formed by the MOSFET drain, the input
   capacitor, and the transformer primary, because this loop area directly determines
   the radiated magnetic field at the switching frequency.

C) Maximise the trace width of all high-current paths to minimise resistance, regardless
   of the loop area formed.

D) Place ferrite cores at the midpoints of the switching current loop traces to absorb
   radiated energy before it leaves the PCB.

---

### Question 15

A PCB layout review finds that a designer has placed a via through the pad of a 0.5 mm
pitch QFN device to provide a thermal connection to the back copper layer. No paste
mask modification has been made. What manufacturing defect will most likely result?

A) Solder will wick through the via into the board substrate, resulting in insufficient
   solder on the pad surface and a weak or open joint.

B) The via will act as a heat sink during reflow, causing the solder joint to solidify
   prematurely before the package self-aligns.

C) The paste deposited over the via opening will partially fill the via and partially
   solder the pad, creating an inconsistent joint height and potential for a cold joint.

D) The thermal expansion of the via barrel during reflow will crack the pad.

---

## Answer Key

---

**Question 1 — Answer: B**

At high frequencies, return current follows the path of minimum inductance (equivalently,
minimum loop area), not the path of minimum resistance. The minimum inductance path is
directly beneath the signal trace on the closest reference plane — the ground plane on
layer 2. The coupling capacitance between the power plane and ground plane (through
decoupling capacitors) allows AC return current via the power plane as well, but the
primary return path is the direct layer-2 ground plane path.

- A is partially true (power and ground are at AC ground potential through decoupling)
  but the dominant return path is the nearer ground plane (layer 2), not the more distant
  power plane (layer 3).
- C is incorrect: resistance (DC) is not the criterion for high-frequency return current.
- D is incorrect: the board edge is not the primary return path.

---

**Question 2 — Answer: B**

When the ground plane is split and a signal trace crosses the split, the return current
cannot travel directly beneath the trace. Instead, it must take a longer path around
the split. This elongated return path forms a much larger current loop. A large current
loop radiates a magnetic field proportional to the loop area and the current magnitude.
At 50 MHz, this creates a significant EMI problem, often visible as a spectral peak at
50 MHz (and harmonics) in pre-compliance scans.

- A is incorrect: alternative paths around the split significantly increase loop area;
  routing around the split is not equivalent to a direct return path.
- C is incorrect: the return current path issue is not a signal reflection mechanism;
  it is a radiated emission mechanism.
- D is incorrect: the propagation velocity and effective dielectric constant are not
  significantly affected by a ground plane split.

---

**Question 3 — Answer: B**

A 10 Gbit/s serial link has a signal bandwidth of approximately 0.35/0.035 ns = 10 GHz.
A via stub acts as a quarter-wave resonant stub that creates a notch in the frequency
response at:

```
f_notch = v_prop / (4 × stub_length)
```

For a stub length of 0.6 mm and a propagation velocity corresponding to 180 ps/mm:

```
f_notch = 1 / (4 × 0.6 mm × 180 ps/mm) = 1 / (432 ps) ≈ 2.3 GHz
```

This notch falls within the signal bandwidth (up to 10 GHz) and causes significant
attenuation. Back-drilling (stub removal) is required for stubs longer than approximately
0.25–0.5 mm at 10 Gbit/s.

- A is incorrect: 50 mm would create a resonance well below 1 GHz — catastrophic.
- C is incorrect: the one-quarter wavelength criterion applies, and at 10 GHz the
  quarter wavelength is extremely short.
- D is incorrect: signal integrity tools model via stubs accurately, but they cannot
  "compensate" for a stub; the stub must be physically removed or minimised.

---

**Question 4 — Answer: B**

Layer 1 (top) references layer 2 (GND), with a prepreg (thinner, typically ~100 µm) as
the dielectric, giving tighter coupling to the reference plane. Layer 6 (bottom)
references layer 5 (GND), with a similar arrangement. In most stackups, the prepreg
between layers 1-2 and 5-6 is similar in thickness. The answer depends on the specific
stackup geometry. In most four-layer stackups used as the basis for six-layer designs,
layer 1 and layer 6 outer layers reference adjacent inner GND planes with similar
dielectric spacing.

However, option B specifically states "larger ground reference area and better plane
coupling due to tighter prepreg spacing" — which is a function of the stackup geometry,
not the layer position per se. For a balanced symmetric stackup where layer 2 (GND) and
layer 5 (GND) are equidistant from their outer layer references, both layers perform
equivalently. The key principle is that both layers have a single reference plane on
one side, and a second plane (power) on the other side, which provides asymmetric
referencing. Neither is clearly better than the other without specific stackup dimensions.

The best answer in this context is B, which correctly identifies the prepreg spacing as
the governing parameter.

- A is incorrect: the bottom layer is not "shielded from above" in a meaningful way by
  the internal planes.
- C is partially correct (both have one reference plane) but concludes incorrectly that
  they are equivalent — actual performance depends on the specific prepreg thicknesses.
- D is incorrect: outer signal layers are commonly used for high-speed differential pairs
  when the reference plane is close and the dielectric height is controlled.

---

**Question 5 — Answer: C**

For a 0.8 mm pitch BGA with 10 × 10 balls, the outer rows can be routed between the
via pads using dog-bone fanout (a short stub trace from the ball pad to a through-hole
via placed just outside the ball row, with the via connecting to an inner routing layer).
However, as you move to inner rows, the space between via pads in the outer rows becomes
too small to route a trace plus a through-hole via (minimum 0.3 mm drill + 0.1 mm
annular ring = 0.5 mm via diameter; 0.8 mm pitch minus 0.5 mm via = 0.3 mm gap, which
is too small for a trace plus spacing). Microvias (laser drilled, typically 0.1–0.15 mm
drill diameter) can be placed on the ball pad itself or in a dog-bone with much tighter
geometry, enabling inner row fanout.

- A is incorrect: dog-bone fanout alone cannot handle all inner rows at 0.8 mm pitch
  without microvias or HDI features.
- B is incorrect: inner rows of a dense BGA cannot all be surface-routed without vias.
- D is incorrect: increasing pitch is not a PCB design solution; it requires a different
  device or package.

---

**Question 6 — Answer: B**

The feedback sense point of a buck converter should measure the actual output voltage
at the point of use — ideally at the output capacitor terminals. If the output capacitor
is 10 mm away and the feedback sense point is at the converter output, the regulator
measures the voltage before the trace IR drop between the converter and the capacitor.
Under transient load conditions, the trace inductance between capacitor and IC can
introduce ringing in the loop. Additionally, if the feedback traces are routed close to
noisy switching nodes, noise couples into the feedback and degrades regulation accuracy.

- A is incorrect: placement significantly affects loop stability and transient response
  in switchmode regulators; it is not only the total capacitance value that matters.
- C is incorrect: trace resistance adds to ESR (reducing ripple current, not increasing it
  directly) but the primary concern is loop stability, not ripple.
- D is incorrect: trace inductance does not usefully filter switching noise; it is more
  likely to create resonance with the output capacitor.

---

**Question 7 — Answer: A**

Inserting a ground trace between the aggressor and victim signals, connected to the
reference plane through vias at regular intervals, provides a shielding conductor that
absorbs the capacitive electric field coupling and provides a return path for magnetically
coupled current close to the aggressor. This is the most effective single change for
reducing near-end crosstalk on a PCB.

- B is incorrect: widening the aggressor trace reduces its impedance (slightly) but
  does not meaningfully reduce coupling to adjacent traces; it may worsen coupling by
  increasing the aggressor's fringing field.
- C is incorrect: routing the victim on the opposite layer directly below the aggressor
  creates near-perfect broadside coupling — one of the worst possible configurations for
  a sensitive analogue trace.
- D is incorrect: slowing the edge rate reduces high-frequency coupling but does not
  address the fundamental coupling at 100 MHz fundamental frequency; the noise coupled
  at 100 MHz depends on the peak voltage, not only the edge rate.

---

**Question 8 — Answer: B**

The high-frequency AC switching current loop of a converter — the current path that
changes rapidly with each switching transition — must be minimised in physical area to
reduce radiated EMI. At 200 kHz and its harmonics, this loop acts as a magnetic dipole
antenna. The input decoupling capacitor provides the local charge reservoir for the
switching events; its loop area with the converter's VIN and PGND pins is the relevant
antenna area. Placing the capacitor directly at these pins minimises the loop.

- A is incorrect: input connector filtering addresses a different noise path (conducted
  emissions on the input cable). While beneficial, it does not address the primary
  switching loop radiation.
- C is incorrect: the inductor's magnetic field does not couple to the input capacitor
  in a way that the capacitor placement compensates; and the inductor is on the output
  side of the converter.
- D is incorrect: distributing capacitance across the board increases loop areas and
  worsens EMI.

---

**Question 9 — Answer: C**

A signal on layer 1 (top) is referenced to layer 2 (ground plane). The return current
travels in the layer-2 ground plane directly beneath the signal trace. Since the ground
plane on layer 2 is a solid, continuous plane, the return current path is not disrupted
by the split in the layer-3 power plane. The layer-3 power plane split is irrelevant
to the return current of a layer-1 signal.

This answer requires understanding that each signal references its nearest return plane,
and that a split in a more distant plane does not affect the return path of the near
reference.

- A is incorrect: the rule that signals must not cross splits applies when the signal's
  return plane has the split. If the reference plane is solid, the signal may cross a
  split in a non-reference plane.
- B is incorrect: stitching vias connect the two power domains capacitively but are
  not required for this signal's return path, as the return plane is layer 2.
- D is incorrect: re-routing to layer 4 is not necessary and introduces unnecessary
  via stubs.

---

**Question 10 — Answer: B**

Thermal vias from the exposed pad through to a large copper area on the bottom of the
PCB are the most effective single technique for a two-layer board. The copper area on
the bottom acts as a heat spreader, increasing the effective convective surface area.
The via array reduces the thermal resistance from the junction to the bottom copper:

```
R_via_array ≈ R_via_single / N_vias
```

For a 7 × 7 array of 0.3 mm via drills (effective area ratio approximately 10-15%),
the thermal resistance from package to board is typically reduced from ~50°C/W (no
vias) to ~25-30°C/W, and with the bottom copper spread this can reach 15-20°C/W.

- A is incorrect: conformal coating is a poor thermal conductor (thermal conductivity
  ~0.2 W/m·K) and provides negligible heat transfer improvement.
- C is incorrect: a series resistor would limit current and reduce power only if the
  IC's function is compatible with reduced supply voltage — this is not a valid thermal
  management technique.
- D is incorrect: increased PCB thickness increases thermal mass (slower temperature rise
  to steady state) but does not change the steady-state thermal resistance.

---

**Question 11 — Answer: A**

For a differential pair with 90 Ω differential impedance, the single-ended impedance
target is approximately 45 Ω (for tightly coupled pairs where coupling coefficient K
is significant). Using a microstrip impedance calculator with H = 0.1 mm, Dk = 4.3,
and T = 35 µm copper:

- A 100 µm trace width gives a single-ended impedance of approximately 50 Ω. With
  100 µm spacing (centre-to-centre 200 µm), the differential impedance is approximately
  90 Ω due to the coupling correction.

This is consistent with standard USB 3.0 design guidelines which commonly specify
100 µm trace / 100 µm space on 0.1 mm dielectric for 90 Ω differential impedance.

- B: 200 µm trace width at this dielectric height gives significantly lower impedance
  (~35-40 Ω single-ended); differential would be around 65-70 Ω.
- C: 100 µm trace with 200 µm spacing has less coupling; differential impedance would
  be closer to 95-100 Ω.
- D: 50 µm trace width at 0.1 mm dielectric gives approximately 65 Ω single-ended,
  with 100 µm spacing approximately 110-115 Ω differential — too high.

---

**Question 12 — Answer: B**

Alternating ground pins throughout the connector pin assignment provides return current
paths close to every signal, minimising signal current loops and reducing both emission
and susceptibility. Ground pins adjacent to high-speed signals reduce the loop area for
those signals. Placing the slowest signals at the outermost connector position reduces
the antenna efficiency of those traces for the lowest-frequency content.

- A is incorrect: placing high-speed signals in the centre does not reduce their emission;
  the trace from the IC to the connector carries the signal regardless of the connector
  pin location.
- C is incorrect: power supply pins are not effective shields for signal pins in an open
  connector environment.
- D is incorrect: grouping by signal direction reduces driver-to-receiver cross-talk within
  the connector but does not address the primary EMI concern (loop area and return current
  paths).

---

**Question 13 — Answer: A**

Parallel (receiver-end) termination is valid for this signal. The one-way flight time
of a 100 mm trace at approximately 170 ps/mm is 17 ns. The signal risetime is 1 ns.
Since the flight time (17 ns) is much greater than the risetime (1 ns), this trace
behaves as a transmission line and requires termination. Parallel termination at the
receiver eliminates reflections completely (the received signal sees the matched
termination). The trade-off is static power consumption of VCC/R = 3.3/50 = 66 mA
per signal line when the signal is high.

- B is incorrect: source termination (series resistor at the driver) is also valid for
  transmission lines, but the question asks about the parallel termination configuration,
  which is correct. The statement "should always be at the source" is wrong; both source
  and load terminations are valid, each with different trade-offs.
- C describes a Thévenin termination (AC-coupled end termination) which is valid, but
  the question asks about the simple parallel-to-ground configuration, which is the
  described circuit.
- D is incorrect: the criterion for termination is that the round-trip flight time
  (2 × 17 ns = 34 ns) exceeds the signal risetime (1 ns); termination is required when
  this condition is met (i.e., when round-trip flight > 2-3× the risetime). 34 ns >>
  1 ns, so termination is definitely required.

---

**Question 14 — Answer: B**

The switching current loop of a converter (input capacitor → switch → transformer primary
→ back to input capacitor) is the primary source of radiated EMI at the switching
frequency. The area of this loop is the effective magnetic dipole area that determines
the radiated field strength:

```
E ∝ f² × A × I
```

Where A is the loop area. Minimising this area is the single most important layout
objective for the switching current loop. This means placing the input capacitor, MOSFET,
and transformer primary as close together as possible in a compact geometry.

- A is incorrect: the transformer primary trace carries high current but it is not
  "impedance-controlled" in the transmission-line sense; it is a low-impedance power path.
- C is incorrect: maximising trace width reduces resistance (beneficial) but if it
  increases loop area, it worsens EMI.
- D is incorrect: ferrite cores on switching current traces would saturate at the
  currents involved and are not a practical board-level EMI remedy for primary switching
  loops.

---

**Question 15 — Answer: A**

A tented or open via through a surface pad will draw solder from the pad surface through
capillary action during reflow. The molten solder wicks into the via barrel and toward
the other side of the board, depleting the solder available for the pad joint. The result
is a solder-starved joint with inadequate fillet height, high resistance, and reduced
mechanical strength. This is a well-documented assembly defect called "via solder wicking."

The correct approach is to either:
- Use a via-in-pad with filled and capped vias (IPC-4761 Type VII): the via is filled
  with copper or resin and then capped with plating, providing a flat soldering surface.
- Offset the via outside the pad (dog-bone style) and connect with a short trace.

- B describes thermal mass effects which can occur but are secondary to solder wicking.
- C is partially correct but less specific than A; the primary concern is solder wicking,
  not inconsistent joint height per se.
- D is incorrect: via barrel thermal expansion during reflow is negligible and does not
  crack pads under normal conditions.

---

## Related Topics

- [Stackup Design](../02_pcb_layout/stackup_design.md)
- [Component Placement](../02_pcb_layout/component_placement.md)
- [Routing Rules](../02_pcb_layout/routing_rules.md)
- [Return Current Paths](../02_pcb_layout/return_current_paths.md)
- [Thermal Management](../02_pcb_layout/thermal_management.md)
