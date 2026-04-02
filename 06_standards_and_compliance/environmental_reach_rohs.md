# Environmental, REACH, RoHS

## Prerequisites
- Basic knowledge of electronics manufacturing processes (PCB fabrication, SMT assembly)
- Understanding of supply chain roles: OEM, ODM, contract manufacturer, component distributor
- Awareness of CE marking and EU product compliance framework

---

## Concept Reference

### Overview: Why Environmental Compliance Matters

Electronics manufacturers operating in the EU (and in markets that have adopted similar
regulations globally) must comply with restrictions on hazardous substances in their
products and must register substances of concern in their supply chains. Non-compliance
results in:
- Prohibition of product sale in the regulated market
- Customs seizure or border rejection
- Financial penalties and product recall obligations
- Reputational damage

The two primary EU regulations affecting PCB electronics design are:
- **RoHS** — restricts specific hazardous substances in the product itself
- **REACH** — requires disclosure and authorisation for substances of very high concern
  in the supply chain

These are different in scope and mechanism: RoHS is a restriction (the substance must
not exceed a threshold), while REACH includes both restrictions and a registration/
authorisation framework covering a much broader range of chemicals.

---

### RoHS: Restriction of Hazardous Substances

#### Legislation

**EU Directive 2011/65/EU (RoHS 2)**, as amended by **Directive 2015/863/EU (RoHS 3)**
which added four phthalate substances. Implemented into UK law as the **RoHS Regulations
2012 (SI 2012/3032)**, as amended.

RoHS applies to **electrical and electronic equipment (EEE)** placed on the market
in the EU. It restricts the use of specific hazardous substances above threshold
concentrations in homogeneous materials within the product.

#### Restricted Substances (RoHS 3)

```
Substance                     Abbreviation   Max concentration in
                                              homogeneous material
-----------------------------  ------------   ---------------------
Lead                           Pb             0.1% by weight (1000 ppm)
Mercury                        Hg             0.1%
Cadmium                        Cd             0.01% (100 ppm)
Hexavalent chromium            Cr(VI)         0.1%
Polybrominated biphenyls       PBB            0.1%
Polybrominated diphenyl ethers PBDE           0.1%
Bis(2-ethylhexyl) phthalate    DEHP           0.1%
Butyl benzyl phthalate         BBP            0.1%
Dibutyl phthalate              DBP            0.1%
Diisobutyl phthalate           DIBP           0.1%
```

**"Homogeneous material"** means a material that cannot be mechanically separated into
different materials. The threshold applies to each homogeneous material, not to the
whole product or component. This matters in practice because:
- A lead-free ENIG PCB still has a thin lead-free surface finish. The underlying
  copper is lead-free, the solder mask is lead-free, the laminate is lead-free.
- If a component uses a ceramic package with a small amount of lead glass in the
  seal, that glass is a separate homogeneous material; if lead exceeds 0.1% of the
  glass (not of the whole component), that homogeneous material is non-compliant.

#### PCB-Relevant RoHS Considerations

**Lead-free soldering:**
The most significant change driven by RoHS was the elimination of lead-tin (SnPb)
solder. Lead-free solders — primarily SAC305 (96.5% tin, 3% silver, 0.5% copper) —
have replaced SnPb in most consumer products.

```
SAC305 solder vs. SnPb solder:
  Melting point: 217°C (SAC305) vs. 183°C (SnPb)
  → Reflow peak temperature: 245-260°C (SAC305) vs. 220-230°C (SnPb)
  → Higher reflow temperature stresses PCB laminate and components
  → FR4 Tg (glass transition temperature) must be ≥ 170°C for lead-free assembly

Wettability: SAC305 wets less readily than SnPb
  → More flux is required
  → Some component pad finishes that were acceptable for SnPb may need to be
     changed for SAC305 assembly (e.g., ENIG is excellent; OSP is acceptable
     but expires faster; bare copper is poor)

Joint reliability: SAC305 joints can be more brittle under shock/vibration
  → Under mechanical loading, SAC305 fails more suddenly than SnPb (tin-silver
     intermetallics are harder and less ductile)
  → For vibration-critical applications (aerospace, automotive), alternatives
     such as SnAgCuBi alloys or Sn-Cu alloys may be specified
```

**RoHS exemptions:**
Many applications are exempt from RoHS restrictions. Key exemptions for electronics:
```
Exemption 5:  Pb in glass of electronic components (CRTs, fluorescent lamps)
Exemption 6c: Pb in solders for network infrastructure equipment (Annex III)
Exemption 7a: Pb in high-melting-point solders (Pb ≥ 85%, for high-temp applications)
Exemption 7c: Pb in solders for servers and storage arrays (currently reviewed)
Exemption 8b: Pb in solders in military electronic equipment (Annex IV — Medical/monitoring)
Exemption 15: Pb in soldering for large power semiconductors (thyristors, etc.)
```

Exemptions have sunset dates and are periodically reviewed. A product relying on an
exemption must track its expiry and re-evaluate compliance before the exemption lapses.

#### RoHS Compliance Process

1. **Identify the product category:** EEE? Does it fall into one of RoHS's 11 categories?
2. **Map all homogeneous materials:** PCB laminate, copper, solder, component packages,
   leads, coatings, adhesives, plastics.
3. **Obtain substance declarations from the supply chain:** Component suppliers provide
   RoHS declarations. Use standardised formats such as IPC-1752A or IEC 62321 test data.
4. **Verify no restricted substance exceeds its threshold in any homogeneous material.**
5. **Compile a Technical File:** Maintain evidence of compliance. Include supplier
   declarations, test reports (if required for high-risk substances), and the DoC.
6. **Declaration of Conformity:** RoHS compliance must be stated in the CE Declaration
   of Conformity (or equivalent) for EU products.

---

### REACH: Registration, Evaluation, Authorisation and Restriction of Chemicals

#### Legislation

**EU Regulation 1907/2006 (REACH)**, with ongoing amendments. UK equivalent: **UK REACH
(SI 2019/758)**, maintained by UKCA. REACH is administered by ECHA (European Chemicals
Agency) in the EU and by HSE (Health and Safety Executive) in the UK.

#### Key Differences from RoHS

| Aspect | RoHS | REACH |
|--------|------|-------|
| Scope | Specific substances in EEE products | All chemicals across all industries |
| Mechanism | Restrict use above thresholds in products | Register substances; authorise/restrict SVHCs |
| Manufacturer obligation | Ensure product below limits; provide DoC | Disclose SVHCs above 0.1% in articles; SVHCs in supply chain |
| Enforcement | Product conformity (border control, market surveillance) | Chemical supply chain registration |
| Who is affected | EEE product manufacturers/importers | Manufacturers, importers, downstream users of chemicals |

#### REACH SVHC List (Substances of Very High Concern)

REACH identifies SVHCs through the **Candidate List** maintained by ECHA. SVHCs are
substances that are:
- **CMR** (Carcinogenic, Mutagenic, or toxic to Reproduction) — categories 1A or 1B
- **PBT** (Persistent, Bioaccumulative, Toxic)
- **vPvB** (very Persistent, very Bioaccumulative)
- Endocrine disruptors or other substances of equivalent concern

As of 2025, the REACH Candidate List contains over 240 substances.

**Article 33 obligation:**
If an article (a manufactured product) contains a SVHC above 0.1% by weight of the
article, the supplier must:
1. Notify the business customer, providing information to allow safe use
2. Upon consumer request within 45 days, provide the same information free of charge

**Article 7(2) obligation:**
If an article contains a SVHC above 0.1% by weight AND above 1 tonne/year is imported
or manufactured, a notification to ECHA is required.

#### REACH-Relevant Substances in PCB Electronics

```
Substance category           Common application          REACH/SVHC status
-----------------------      ------------------------    ----------------------
Bisphenol A (BPA)            Epoxy resin (FR4)           Potential SVHC; some
                                                          grades listed
Antimony trioxide (ATO)      Flame retardant (FR4)       SVHC candidate (CMR)
Lead compounds               Legacy SnPb solder, glass   Pb acetate is SVHC
Phthalates (DEHP etc.)       Cable insulation, PVC       SVHC, also RoHS restricted
Perfluorooctanoic acid (PFOA) Water-resistant coatings,  SVHC, restricted
                              some PTFE precursors
Cobalt dichloride            Some inks and coatings      SVHC (CMR)
Short-chain chlorinated       Cutting oils, some plastics SVHC, REACH restricted
paraffins (SCCP)
Boric acid                   Some PCB flux formulations  SVHC (reproductive toxin)
```

#### REACH Authorisation

Substances on the **Authorisation List (Annex XIV)** cannot be used after their
sunset date unless:
- An authorisation has been granted by ECHA for a specific use
- An exemption applies (e.g., intermediates, R&D quantities < 1 tonne)

For electronics manufacturers, authorisation-relevant substances include certain
chromates (used in metal surface treatments) and some flame retardants.

#### Practical REACH Compliance for PCB Designers

1. **Obtain REACH declarations from component suppliers** (alongside RoHS declarations).
   IPC-1752A Class D provides SVHC substance disclosure.
2. **Calculate substance concentrations per article.** REACH's 0.1% threshold applies
   to the weight of the **article** (the product or a sub-article). For a PCB:
   - The PCB assembly is the article
   - If a single component contains a SVHC at > 0.1% of its own weight, AND
     that component's weight × SVHC concentration > 0.1% of the total PCB weight:
     the disclosure obligation applies
3. **Monitor the ECHA Candidate List.** New SVHCs are added regularly. Subscribe to
   ECHA updates and review the product's supply chain declarations when new SVHCs
   are added.
4. **Document the compliance assessment.** Maintain records of SVHC assessments and
   supplier declarations. REACH compliance is not a one-time certification — it is
   an ongoing obligation.

---

### WEEE: Waste Electrical and Electronic Equipment

While not directly restricting design, WEEE Directive 2012/19/EU (and UK WEEE
Regulations 2013) imposes obligations on producers of EEE regarding end-of-life
collection and recycling:

- Register with a Producer Compliance Scheme (PCS)
- Finance the take-back and recycling of waste EEE
- Mark products with the crossed-out wheelie bin symbol

Design implications:
- Minimise hazardous substances to reduce recycling cost (aligned with RoHS)
- Consider disassembly design: use screws rather than glue for enclosures where
  possible, to allow component separation at end of life
- Specify materials in the product documentation to facilitate recycling

---

## Best Practices

- Maintain a **substances database** for the product. Record the supplier declaration
  status of every component and the substances present.
- Use **standardised declaration formats** (IPC-1752A Class C or D) rather than
  ad-hoc supplier letters. Standardised formats are machine-readable and can be
  aggregated automatically.
- Treat **RoHS and REACH as ongoing obligations**, not one-time activities. Supply
  chain changes (component substitutions, second-source qualification) can introduce
  new substances that require reassessment.
- For **military and aerospace exemptions**, track exemption sunset dates and begin
  re-qualification before expiry. Late re-qualification leads to production halts.
- Verify that **PCB surface finishes are lead-free** (ENIG, HASL-LF, ENEPIG, OSP,
  ImAg, ImSn) unless a specific RoHS exemption applies to the product.
- Confirm **FR4 laminate grade** is compatible with lead-free reflow temperatures
  (Tg ≥ 170°C for lead-free assembly; some lower-grade FR4 materials have Tg = 130-150°C
  which is insufficient).

---

## Tier 1 — Fundamentals

### Question F1
**What is the maximum permitted concentration of lead in a homogeneous material under RoHS 2/3? What does "homogeneous material" mean in this context?**

**Answer:**

The maximum permitted concentration of lead (Pb) under RoHS 2/3 is **0.1% by weight**
(1000 ppm) in any homogeneous material.

A homogeneous material is a material that cannot be mechanically separated into
different materials. The key word is "mechanically" — a material is homogeneous if
the only way to separate its constituents is by chemical or thermal processing. Examples:

- Solder alloy (tin-lead or tin-silver-copper) — homogeneous
- Copper conductor — homogeneous
- FR4 laminate (epoxy resin + glass fabric) — treated as homogeneous at the laminate
  level (the glass fibres cannot be mechanically separated from the resin in normal use)
- A component's lead frame coating — separate homogeneous material from the lead frame
  base metal

The homogeneous material rule means that a leaded glass seal inside a semiconductor
package is a different homogeneous material from the package body or lead frame. If
the glass contains > 0.1% lead (which traditional lead glass does), that component
may be non-compliant unless covered by a RoHS exemption. Many traditional glass-sealed
components rely on Exemption 5 (lead in glass of certain components).

---

### Question F2
**A product designer proposes using SnPb (60/40) solder for a prototype to reduce cost. What regulatory issues does this create for eventual product certification, and what is the recommended approach?**

**Answer:**

SnPb solder contains approximately 40% lead by weight — far above the 0.1% RoHS
threshold. Using SnPb solder in a product intended for the EU or UK market (or any
market with RoHS-equivalent regulations) violates RoHS unless the product qualifies
for a specific exemption.

**Regulatory issues:**

1. **CE/UKCA marking is not achievable** with SnPb solder unless the product falls
   into an exempt category or uses a specific exemption (e.g., Exemption 7a for
   high-melting-point solders, or military exemptions under Annex IV).

2. **Mixed-soldering risk:** If prototype boards are built with SnPb and then reworked
   with lead-free solder, the resulting solder joints contain tin-lead at concentrations
   above the threshold — non-compliant.

3. **Test report invalidity:** Safety testing (EMC, LVD) performed on a SnPb prototype
   may not be valid for a lead-free production board if the SnPb → lead-free transition
   changes the product's characteristics (unlikely to affect EMC or safety, but a
   competent authority could challenge this).

**Recommended approach:**

Use lead-free solder (SAC305 or SAC0307) from the start of prototyping. The additional
cost is minimal (SAC305 solder paste costs slightly more than SnPb but is not significant
compared to prototype PCB and assembly costs). This ensures the prototype is materially
representative of the production build and eliminates a rework step.

If SnPb is used for early feasibility builds, clearly label all boards as "NON-ROHS
PROTOTYPE — NOT FOR SALE" and track them separately from compliant builds.

---

## Tier 2 — Intermediate

### Question I1
**REACH Article 33 requires that suppliers disclose SVHC content in articles to their customers. Under what circumstances is this obligation triggered, and what information must be disclosed?**

**Answer:**

**When the obligation is triggered:**

Article 33 is triggered when ALL of the following conditions are met:
1. The article contains a substance on the REACH SVHC Candidate List
2. The concentration of the SVHC in the article exceeds **0.1% by weight** of the article
3. The supplier is placing the article on the market (selling it commercially)

The 0.1% threshold applies to the weight of the article as a whole — not to
homogeneous materials within the article (this is different from RoHS). For a 10 g
PCB assembly containing a component with a SVHC at 1% of the component's 0.05 g weight:
```
SVHC mass in article = 0.01 × 0.05 g = 0.0005 g
As % of article = 0.0005 / 10 = 0.005% → below 0.1% threshold, no obligation
```

**What information must be disclosed:**

Sufficient information to allow safe use of the article, which at minimum includes:
- The name of the SVHC
- The concentration of the SVHC (at least whether it is above 0.1% and if possible
  a quantitative value or range)
- Safe use instructions (where relevant — for a finished electronic product this is
  often not applicable)

For **business-to-business** (B2B) supply: disclosure is automatic (proactively
provided with the article or accessible on request).

For **consumer supply**: the consumer may request the information within 45 days of
receiving the product, and the supplier must respond free of charge.

In practice, this obligation is fulfilled through product substance declarations
(e.g., IPC-1752A Class D), Safety Data Sheets for chemical products, and product
compliance portals.

---

## Tier 3 — Advanced

### Question A1
**A product is being redesigned to incorporate a higher-performance FPGA. The new FPGA's package requires a high-temperature reflow process (peak temperature 260°C). The existing PCB uses a standard FR4 laminate (Tg = 135°C). What materials changes are required, and what other design considerations arise from the lead-free high-temperature process?**

**Answer:**

**PCB Laminate Upgrade:**

A standard FR4 laminate with Tg = 135°C is not suitable for lead-free assembly at
260°C peak reflow temperature. When the PCB temperature exceeds Tg during reflow, the
laminate transitions from glassy (rigid) to rubbery (soft) state. While it returns
to glassy state on cooling, the thermal excursion causes:

- Z-axis expansion (CTE increases from ~50 ppm/°C below Tg to ~200-300 ppm/°C above Tg)
- This expansion stresses via barrels, causing barrel cracking and copper fracture
- Delamination at prepreg-copper interfaces in severe cases
- Degradation of resin with repeated thermal cycling

Required upgrade: Use a high-Tg laminate with Tg ≥ 170°C.

```
Common laminate options for lead-free assembly:
  Material                 Tg        Td (decomposition)   Notes
  ---------------------    --------  -----------------    -----------------------
  FR4 standard             135°C     315°C                Not suitable for LF
  FR4 mid-Tg              150°C     330°C                Marginal for LF
  FR4 high-Tg (e.g., IT180A) 175°C  355°C               Suitable for LF assembly
  Halogen-free FR4         ≥170°C    350°C               Suitable; also RoHS-aligned
  Polyimide                250°C     >450°C              Best thermal stability;
                                                          expensive; absorbs moisture
  Rogers 4000 series       ≥280°C    >450°C              RF/high-speed; high cost
```

In addition to Tg, specify the laminate's **Td (thermal decomposition temperature)**
and **T260 or T288 (time to delamination at 260/288°C)**. IPC-4101 Class C and
Class E laminates define requirements for lead-free assembly.

**Via Design for High-Temperature Assembly:**

High-Tg laminates have lower z-axis CTE but are not immune. For a 16-layer board with
many reflow passes:
- Use filled and capped microvias (IPC-4761 Type VII) for any via that passes through
  the board and is covered by BGA lands — uncapped vias under BGAs can collect
  solder, causing shorts
- Increase via barrel copper plating thickness: target 25 µm minimum (vs. 18 µm
  standard) for improved thermal cycle reliability

**Component Temperature Ratings:**

Verify that all components are rated for the lead-free reflow peak temperature:
- Most commercial ICs are rated for 260°C peak, 30-second duration (JEDEC J-STD-020)
- Electrolytic capacitors: verify rated for 260°C reflow (many are rated for 230°C
  only — these must be upgraded to 260°C-rated types)
- Connectors: plastic housings must be rated for 260°C (many are; some low-cost
  connectors use plastics with lower melting points)
- Crystals and oscillators: check manufacturer's reflow profile — 32.768 kHz tuning
  fork crystals are particularly sensitive to high reflow temperatures

**Moisture Sensitivity:**

BGA packages and other complex ICs are moisture sensitive. Before lead-free reflow at
260°C, packages must be baked to remove absorbed moisture. Rapid steam generation
during reflow above the boiling point causes "popcorning" — delamination and cracking
inside the package.

- Verify MSL (Moisture Sensitivity Level) of the FPGA and all other ICs
- MSL 3 components (most BGAs): 168 hour floor life, bake at 125°C for 24 hours
  if floor life exceeded
- MSL 1 components: unlimited floor life, no baking required

**Summary of changes required:**

1. PCB laminate: upgrade to high-Tg FR4 (Tg ≥ 170°C, T260 ≥ 60 min)
2. Verify all component temperature ratings for 260°C peak reflow
3. Review and upgrade any capacitors not rated for 260°C
4. Implement moisture control procedures for MSL ≥ 3 components
5. Review via design: filled/capped vias under BGA footprint
6. Update PCB fabrication specification to reference IPC-4101 Class C or E
