# PCB Design and Electronics Interview Preparation

![PCB Design](https://img.shields.io/badge/Subject-PCB%20Design%20%26%20Electronics-blue)

## Overview

This repository contains comprehensive interview preparation material for PCB design and electronics engineering roles. It covers schematic capture, PCB layout, design for manufacturing, analogue circuit design, board bring-up methodology, and compliance standards. Each topic includes fundamental concepts, design guidelines, worked problems, and interview-style quizzes.

## Table of Contents

- [01 Schematic Design](#01-schematic-design)
- [02 PCB Layout](#02-pcb-layout)
- [03 Design for Manufacturing](#03-design-for-manufacturing)
- [04 Analogue Design](#04-analogue-design)
- [05 Board Bring-Up](#05-board-bring-up)
- [06 Standards and Compliance](#06-standards-and-compliance)
- [07 Quizzes](#07-quizzes)
- [How to Use](#how-to-use)
- [Contributing](#contributing)
- [Related Repositories](#related-repositories)

## 01 Schematic Design

Fundamentals of electrical schematic design, component selection, and circuit architecture.

- [Component Selection](01_schematic_design/component_selection.md)
- [Power Supply Architecture](01_schematic_design/power_supply_architecture.md)
- [Decoupling Strategy](01_schematic_design/decoupling_strategy.md)
- [ESD Protection](01_schematic_design/esd_protection.md)
- [Design for Test](01_schematic_design/design_for_test.md)
- Worked Problems
  - [Problem 01: Power Tree Design](01_schematic_design/worked_problems/problem_01_power_tree_design.md)
  - [Problem 02: Reset Circuit](01_schematic_design/worked_problems/problem_02_reset_circuit.md)
  - [Problem 03: Level Shifting](01_schematic_design/worked_problems/problem_03_level_shifting.md)

## 02 PCB Layout

PCB design principles, layer stackup, routing strategies, and board assembly considerations.

- [Stackup Design](02_pcb_layout/stackup_design.md)
- [Component Placement](02_pcb_layout/component_placement.md)
- [Routing Rules](02_pcb_layout/routing_rules.md)
- [Return Current Paths](02_pcb_layout/return_current_paths.md)
- [Thermal Management](02_pcb_layout/thermal_management.md)
- Worked Problems
  - [Problem 01: BGA Fanout](02_pcb_layout/worked_problems/problem_01_bga_fanout.md)
  - [Problem 02: Differential Pair Routing](02_pcb_layout/worked_problems/problem_02_differential_pair_routing.md)
  - [Problem 03: Power Plane Splits](02_pcb_layout/worked_problems/problem_03_power_plane_splits.md)

## 03 Design for Manufacturing

Design for manufacturability, fabrication constraints, assembly processes, and cost optimisation.

- [PCB Fabrication Constraints](03_design_for_manufacturing/pcb_fabrication_constraints.md)
- [Assembly Considerations](03_design_for_manufacturing/assembly_considerations.md)
- [Via Types and Selection](03_design_for_manufacturing/via_types_and_selection.md)
- [Panelisation](03_design_for_manufacturing/panelisation.md)
- Worked Problems
  - [Problem 01: Via in Pad](03_design_for_manufacturing/worked_problems/problem_01_via_in_pad.md)
  - [Problem 02: Impedance Specification](03_design_for_manufacturing/worked_problems/problem_02_impedance_specification.md)
  - [Problem 03: BOM Optimisation](03_design_for_manufacturing/worked_problems/problem_03_bom_optimisation.md)

## 04 Analogue Design

Analogue circuit design, operational amplifiers, data converter interfaces, and signal conditioning.

- [Op-Amp Circuits](04_analogue_design/op_amp_circuits.md)
- [ADC/DAC Interfaces](04_analogue_design/adc_dac_interfaces.md)
- [Sensor Signal Conditioning](04_analogue_design/sensor_signal_conditioning.md)
- [Noise Analysis](04_analogue_design/noise_analysis.md)
- Worked Problems
  - [Problem 01: Instrumentation Amplifier](04_analogue_design/worked_problems/problem_01_instrumentation_amp.md)
  - [Problem 02: ADC SNR](04_analogue_design/worked_problems/problem_02_adc_snr.md)
  - [Problem 03: Filter Design](04_analogue_design/worked_problems/problem_03_filter_design.md)

## 05 Board Bring-Up

Practical board bring-up procedures, power sequencing, debugging, and rework techniques.

- [Bring-Up Methodology](05_board_bringup/bringup_methodology.md)
- [Power-On Sequence](05_board_bringup/power_on_sequence.md)
- [Debugging Techniques](05_board_bringup/debugging_techniques.md)
- [Rework and ECN](05_board_bringup/rework_and_ecn.md)
- Worked Problems
  - [Problem 01: No Power Debug](05_board_bringup/worked_problems/problem_01_no_power_debug.md)
  - [Problem 02: Clock Failure](05_board_bringup/worked_problems/problem_02_clock_failure.md)
  - [Problem 03: Thermal Issue](05_board_bringup/worked_problems/problem_03_thermal_issue.md)

## 06 Standards and Compliance

EMC, safety, creepage distances, environmental standards, and assembly process requirements.

- [EMC Basics](06_standards_and_compliance/emc_basics.md)
- [Safety and Creepage](06_standards_and_compliance/safety_and_creepage.md)
- [Environmental, REACH, RoHS](06_standards_and_compliance/environmental_reach_rohs.md)
- [Conformal Coating](06_standards_and_compliance/conformal_coating.md)

## 07 Quizzes

Interview-style quiz questions across all major topics.

- [Schematic Quiz](07_quizzes/quiz_schematic.md)
- [Layout Quiz](07_quizzes/quiz_layout.md)
- [DFM Quiz](07_quizzes/quiz_dfm.md)
- [Analogue Quiz](07_quizzes/quiz_analogue.md)
- [Bring-Up Quiz](07_quizzes/quiz_bringup.md)

## How to Use

This repository is structured as a comprehensive interview preparation guide. Approach it systematically:

1. **Foundation**: Start with the 01_schematic_design folder to understand circuit principles and schematic best practices.
2. **Layout and Manufacturing**: Progress through 02_pcb_layout and 03_design_for_manufacturing to understand physical board constraints and manufacturing realities.
3. **Specialisation**: Explore 04_analogue_design if working on signal processing or power management roles.
4. **Practical Skills**: Study 05_board_bringup for real-world debugging and troubleshooting scenarios.
5. **Compliance**: Review 06_standards_and_compliance for regulatory and safety topics.
6. **Self-Assessment**: Use the quizzes in 07_quizzes to identify knowledge gaps.

Each section includes:
- Conceptual foundations and design guidelines
- Worked problems demonstrating real design challenges
- Interview tips and common questions

Read the materials actively, sketch diagrams, work through the problems with pencil and paper, and refer to datasheets as needed.

## Contributing

Contributions are welcome. Please ensure:

- Additions follow the existing markdown style and structure
- New content includes both theory and practical worked examples
- Language is clear and suitable for interview preparation
- All diagrams and examples are original or properly attributed

Submit pull requests with clear descriptions of additions or improvements.

## Related Repositories

- [Interview_SI_PI](https://github.com/BrendanJamesLynskey/Interview_SI_PI) — Signal Integrity and Power Integrity interview preparation
- [Interview_Power_Supply_Design](https://github.com/BrendanJamesLynskey/Interview_Power_Supply_Design) — Power supply topology and design
- [COT_DCDC_Simulink](https://github.com/BrendanJamesLynskey/COT_DCDC_Simulink) — DCDC converter simulation and control

## License

MIT License — see [LICENSE](LICENSE) for details.
