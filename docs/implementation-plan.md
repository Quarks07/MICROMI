# Implementation plan

We will start with a single-frame terrain mapper on the PolarFire SoC Icicle Kit. The aim is to test whether adaptive cells can retain useful terrain detail within a fixed memory budget. The FPGA implementation and measurements are planned work.

## First prototype

Each recorded frame will contain up to 100,000 points. The mapper will cover a 32 m square area with 40 cm cells, refining selected tiles to 20 or 10 cm. Cells store height statistics and observation status. Multi-frame mapping, SLAM and vehicle control are outside this prototype. A 5 cm mode is a later extension.

The host will encode coordinates as signed integer millimetres and reject non-finite values before transmission. The FPGA will check record format, coordinate bounds and configuration. Horizontal bounds will include the lower edge and exclude the upper edge, with a configured origin. Heights will range from -32 m inclusive to +32 m exclusive. Each frame will carry an ID and point count.

## Processing flow

| Component | Responsibility |
|---|---|
| Host computer | Replay recorded frames over Gigabit Ethernet, run the software reference and display comparisons. Optional semantic labels come from the host. |
| RISC-V processor | Receive frames, manage shared DDR buffers, configure the FPGA and return completed maps. |
| AXI transfer interface | Move points and cell data between shared DDR and the FPGA. Confirm the board connection and buffer ownership during bring-up. |
| FPGA mapper | Validate points, calculate cell addresses, accumulate heights, select fine tiles and record status. |

The first pass builds the 40 cm grid. The tile manager ranks areas by geometric variation and assigns a limited number of fine slots. Tile index breaks equal-score ties so results are repeatable. Optional semantic priorities will be evaluated separately.

The second pass reads the same original points from DDR and fills the selected fine cells. Coarse-cell statistics cannot reconstruct the distribution of points within smaller cells. The output therefore identifies which resolution is active in each tile, so overlapping cells are not counted twice.

When fine slots run out, the coarse map remains available and a flag records the limit. Empty cells remain unknown. Incomplete passes mark the frame incomplete. Repeated updates to one cell and stalled transfers must preserve every accepted point.

## Memory budget

Each point uses 16 bytes: three signed 32-bit coordinates and a 32-bit metadata field. Each cell uses 24 bytes: a signed 64-bit height sum, signed 32-bit minimum and maximum, a 32-bit count and 32-bit flags.

A 1.6 m square tile holds 4 by 4 cells at 40 cm, 8 by 8 at 20 cm, or 16 by 16 at 10 cm. The map contains 400 tiles. Each fine slot reserves enough space for 10 cm cells, including when used at 20 cm.

| Allocation | Calculation | Bytes |
|---|---|---:|
| Coarse grid | 80 x 80 x 24 | 153,600 |
| 128 fine slots | 128 x 16 x 16 x 24 | 786,432 |
| Tile descriptors | 400 x 32 | 12,800 |
| Analysis and output scratch | Reserved | 65,536 |
| Total map storage | | 1,018,368 |

This leaves 30,208 bytes within a 1 MiB map budget. At most 128 of the 400 tiles can use fine slots. For comparison, uniform cell records use 614,400 bytes at 20 cm or 2,457,600 bytes at 10 cm, excluding metadata. These are storage calculations; the tests must establish how much terrain detail each map retains.

Two input buffers require another 3,200,000 bytes. Map storage and input buffers therefore total 4,218,368 bytes before software, network buffers and any extra output copy. Most storage will use shared DDR. On-chip FIFOs and caches need a separate synthesis budget. A 5 cm tile would require 24,576 bytes and a revised allocation.

## Development stages

| Weeks | Work and completion check |
|---|---|
| 1-2 | Bring up the kit, tools and DDR transfers. Agree on data formats and build the software reference. |
| 3-5 | Implement the coarse mapper. Match the reference for boundaries, repeated cell updates, resets and stalls. |
| 6-8 | Add tile selection and point replay. Verify fine cells and fallback when storage fills. |
| 9-11 | Integrate Ethernet, processor control and map export. Test malformed and incomplete frames. |
| 12-14 | Compare recorded data, measure memory traffic and processing time, and check synthesis timing and resource use. |
| 15-16 | Resolve failures and prepare a repeatable demonstration. Add optional features only after the core tests pass. |

## Evaluation

The target is 100,000 input points per frame at 10 frames per second. Two passes require up to two million point visits per second. At 16 bytes per point, this means 16 MB/s of input payload and 32 MB/s of point-buffer reads, plus cell updates and other traffic.

We will check exact agreement with a fixed-point software reference for cell statistics, flags and tile selection. Tests will cover empty frames, negative coordinates, boundary points, invalid records, concentrated points, resets, stalled output and exhausted fine slots. Accepted and rejected counts must reconcile for every completed frame.

We will compare adaptive maps with uniform 10, 20 and 40 cm grids on identical data. Measurements will cover elevation error, retention of annotated edges and depressions with observed returns, peak memory and unmet refinement requests.

Timing will include both passes, clearing, allocation and export. Network transfer, optional host inference and visualization will be reported separately. We will also record latency distributions, clock frequency, timing slack and FPGA resource use. Simulation and board results will be identified separately; power comparisons require actual measurements.
