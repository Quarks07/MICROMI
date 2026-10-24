# MICROMI

### Adaptive LiDAR terrain mapping on the PolarFire SoC Icicle Kit

Robots need to understand the shape of the ground around them. A detailed LiDAR map can capture curbs, slopes and uneven surfaces, but keeping that detail everywhere takes memory and processing time.

We are proposing an FPGA mapper that uses larger cells for simple terrain and smaller cells where the ground changes. Each cell stores height information, producing a 2.5D map: a top-down view with elevation.

This project builds on our SIH 26053 adaptive LiDAR mapping work. We are developing the proposal for the Microchip PolarFire FPGA Design Contest 2026, using the PolarFire SoC Icicle Kit in Track 2. Hardware implementation and measurements are the next stage.

## How it works

1. A computer sends recorded LiDAR points to the board over Ethernet.
2. The RISC-V processor stores the points and starts the FPGA mapper.
3. The FPGA builds a coarse height map, then selects areas for finer cells within a fixed memory budget.
4. A second pass through the stored points fills those finer cells. The board returns the map for comparison with a software reference.

Cells without measurements stay unknown. If fine-cell storage fills up, the system keeps the coarse map and reports the limit.

![MICROMI system block diagram](diagrams/MICROMI-System-Block-Diagram.drawio.png)

[View PDF](diagrams/MICROMI-System-Block-Diagram.pdf) · [Editable draw.io file](diagrams/MICROMI-System-Block-Diagram.drawio) · [SVG](diagrams/MICROMI-System-Block-Diagram.svg)

## What we will build first

The first prototype will map one LiDAR frame at a time over a 32 m square area, using 10, 20 and 40 cm cells. Optional semantic labels can come from the host computer. Neural inference stays on the host.

We are targeting one million input points per second. We will compare memory use, terrain detail and complete processing time against software and fixed-resolution maps. The [implementation plan](docs/implementation-plan.md) covers the memory budget, development stages and tests.

## Project files

- [Registration text and diagram link](FORM-FIELDS.md), ready to copy into the form.
- [Submission text files](submission/), one file per form field.
- [Diagram notes](docs/diagram-layout.md), including how to edit the source.
- [Document checks](docs/validation.md) and [AI assistance disclosure](AI-ASSISTANCE.md).

## References

- [Microchip PolarFire FPGA Design Contest](https://www.microchip.com/en-us/campaigns/polarfire-fpga-design-contest)
- [Official contest rules](https://ww1.microchip.com/downloads/aemDocuments/documents/FPGA/campaign/PolarFire_FPGA_Design_Contest_2026_Offical_Rules.pdf)
- [PolarFire SoC Icicle Kit](https://www.microchip.com/en-us/development-tool/mpfs-icicle-kit)
- [Related FPGA LiDAR processing research](https://arxiv.org/abs/2408.10410)
