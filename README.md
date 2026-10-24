# MICROMI

## FPGA-accelerated adaptive 2.5D LiDAR terrain mapping

MICROMI's proposal for the Microchip PolarFire FPGA Design Contest 2026.

**Selected platform:** PolarFire SoC Icicle Kit, Track 2.

MICROMI will investigate a memory-bounded hardware engine for constructing elevation maps from LiDAR point streams. The FPGA will perform repeated point-to-cell operations and adaptive tile allocation. The RISC-V processor subsystem will manage transfers, configuration, and output. Optional semantic metadata will come from a host perception pipeline.

This repository contains proposal documents and architecture drawings. FPGA implementation and hardware measurements are planned work. Performance targets are not measured results.

Use [FORM-FIELDS.md](FORM-FIELDS.md) for the exact text and diagram link to enter in the registration form.

## Submission files

| File | Use |
|---|---|
| [Project overview](submission/project-overview.txt) | Paste into the form's Project Overview field, maximum 300 words |
| [Innovation and expected impact](submission/innovation-and-impact.txt) | Paste into Innovation & Expected Impact, maximum 250 words |
| [Project title](submission/project-title.txt) | Use wherever the form requests the project title |
| [Development platform](submission/development-platform.txt) | Select the matching radio option |
| [System block diagram, PDF](diagrams/MICROMI-System-Block-Diagram.pdf) | Diagram for reviewers |
| [Editable draw.io source](diagrams/MICROMI-System-Block-Diagram.drawio) | Open in draw.io or diagrams.net to edit native text, shapes, and connectors |
| [Scalable diagram](diagrams/MICROMI-System-Block-Diagram.svg) | Vector preview |
| [PNG preview](diagrams/MICROMI-System-Block-Diagram.drawio.png) | Image preview |
| [Implementation plan](docs/implementation-plan.md) | Scope, development stages, and evaluation |
| [AI assistance](AI-ASSISTANCE.md) | Disclosure of assistance used in proposal preparation |

![MICROMI proposed system architecture](diagrams/MICROMI-System-Block-Diagram.drawio.png)

## Architecture boundaries

- The first demonstrator will replay recorded point clouds and construct a single-frame map within a 32 m square region.
- Host software will supply optional semantic metadata. The initial FPGA design will not run the neural segmenter or detector.
- The coarse/fine tile budget will be explicit. Refinement will replay or retain the underlying points; coarse aggregates alone cannot reconstruct fine child cells.
- Observed, unknown, invalid, and resource-limited states will remain distinct.
- Kernel timing will be reported separately from input transfer, optional inference, and visualization.

## Official references

- [PolarFire FPGA Design Contest](https://www.microchip.com/en-us/campaigns/polarfire-fpga-design-contest)
- [Official contest rules](https://ww1.microchip.com/downloads/aemDocuments/documents/FPGA/campaign/PolarFire_FPGA_Design_Contest_2026_Offical_Rules.pdf)
- [PolarFire SoC Icicle Kit](https://www.microchip.com/en-us/development-tool/mpfs-icicle-kit)
- [Microchip FPGA tool licensing](https://www.microchip.com/en-us/products/fpgas-and-plds/fpga-and-soc-design-tools/fpga/licensing)
- [Related research: stream-based FPGA LiDAR ground segmentation](https://arxiv.org/abs/2408.10410)

The related paper establishes existing FPGA LiDAR work. MICROMI's proposed allocation policy and architecture will need their own implementation and comparative evaluation.

## Preparing the submission

Use the plain-text files for the two prose fields. For System Block Diagram, share the public `diagrams` folder from the final proposal commit so reviewers can open the PDF and editable source together. Keep submitted files fixed after the organizer's deadline unless the organizer permits a change.

The package does not contain existing private project source code, trained weights, or datasets. No contest form was submitted by publishing these materials.
