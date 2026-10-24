# Registration text

Copy the paragraphs below into the matching fields. The diagram link points to the saved diagram version.

## Project title

FPGA-Accelerated Adaptive 2.5D LiDAR Terrain Mapping

## Project overview

185 words · Limit: 300

LiDAR helps robots understand the ground around them, but detailed maps can use a lot of memory and processing time. Flat ground needs less detail than a curb, slope or uneven patch of terrain.

MICROMI proposes an FPGA-based mapper that adjusts map detail to the terrain. It builds on our SIH 26053 adaptive LiDAR mapping work and uses the PolarFire SoC Icicle Kit. The output is a 2.5D map: a top-down grid that also stores ground height.

The FPGA will group LiDAR points into cells, calculate height statistics and assign smaller cells to areas with more variation. A second pass through the original points will fill these finer cells. The onboard RISC-V processor will manage data transfers and map output. Areas with no observations will remain marked as unknown.

We will start with recorded LiDAR data and compare the maps with a software reference and fixed-resolution grids. Our target is one million input points per second. Tests will measure processing time, memory use and how well the map preserves terrain detail. The prototype is intended for embedded mapping research in mobile robots and off-road vehicles.

## System block diagram

https://github.com/Quarks07/MICROMI/tree/proposal-v1/diagrams

## Innovation and expected impact

129 words · Limit: 250

MICROMI focuses on where to spend limited map memory. Simple terrain will use larger cells, while areas with greater height variation will receive finer cells within a fixed memory budget.

The proposed FPGA design combines this allocation with parallel point processing. Fine cells will use the original LiDAR points, and areas without measurements will stay unknown. If fine-cell storage runs out, the system will retain the coarse map and report the limit.

The expected outcome is a working prototype that helps us understand the trade-off between terrain detail, memory and speed. We will compare it with fixed-resolution maps on the same data and measure the complete processing time, including both passes and data transfers. These results will show whether the design is useful for robots with limited computing resources.

## Development platform

Select **PolarFire SoC Icicle Kit**.

[AI assistance disclosure](AI-ASSISTANCE.md)
