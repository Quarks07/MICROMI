# MICROMI form fields

## Project title

FPGA-Accelerated Adaptive 2.5D LiDAR Terrain Mapping

## Project Overview

249 words, maximum 300. Paste only the following text.

Mobile robots need terrain maps that preserve elevation changes without spending the same memory on every patch of ground. Uniform fine grids can consume storage on simple terrain, while software mapping competes with other perception tasks.

Team MICROMI proposes an FPGA-accelerated adaptive 2.5D LiDAR terrain mapper on the PolarFire SoC Icicle Kit. The project builds on our SIH 26053 work in adaptive semantic mapping.

The host will replay recorded LiDAR frames over Gigabit Ethernet. RISC-V software will buffer points in shared DDR, configure the accelerator, and control transfers through an AXI data interface. Optional semantic metadata will come from the host; full neural inference will remain there.

The FPGA will validate points, crop a 32 m square region, compute cell addresses, and accumulate minimum and maximum elevation, sum, count, and observation flags. Coarse-grid analysis will guide a memory-bounded tile manager using 10, 20, and 40 cm cells. A second pass over the original points will populate selected fine tiles correctly. Hardware counters will expose stalls, rejected points, and exhausted refinement capacity. Unknown cells will remain distinguishable from observed terrain.

RISC-V software will export the bounded map and status through shared DDR to a host comparison view. Evaluation will compare numerical agreement, terrain-detail retention, memory, throughput, and frame latency against software and uniform-grid baselines. The performance target is one million input points per second, with both passes and transfer costs reported. Initial scope is single-frame mapping; 5 cm refinement is an extension. Multi-frame SLAM and safety control are outside scope.

## System Block Diagram

Paste this public folder link. It contains the PDF, editable draw.io source, supplied PNG preview and SVG.

https://github.com/Quarks07/MICROMI/tree/proposal-v1/diagrams

## Innovation & Expected Impact

189 words, maximum 250. Paste only the following text.

MICROMI will investigate how a fixed memory budget can preserve useful terrain detail in an FPGA mapping engine. Its proposed contribution combines streaming elevation statistics, geometry-guided coarse/fine tile allocation, correct refinement through point replay, and explicit reporting when requested detail exceeds available storage. Optional host-generated semantic metadata will let the team compare geometry-only allocation with allocation guided by selected classes.

The design will keep unknown cells, rejected inputs, and refinement limits visible. This matters because missing returns do not establish safe ground, and a coarse cell cannot reveal how its points were distributed among finer children. The second pass will rebuild selected tiles from original observations.

Expected impact is a reproducible prototype for research on embedded terrain mapping, with documented trade-offs between map detail, memory, and processing time. Evaluation will compare identical frames against a fixed-point software reference and uniform grids. It will report both accelerator and complete processing times, including replay, rather than treating a kernel target as achieved system performance.

Building on established research in FPGA LiDAR processing and adaptive mapping, MICROMI will evaluate a specific contribution: predictable storage allocation with explicit limits and reproducible hardware/software comparisons.

## Development Platform

Select **PolarFire SoC Icicle Kit**. This is Track 2, Connected Real-Time Systems.

## Assistance disclosure

The disclosure is included in [AI-ASSISTANCE.md](AI-ASSISTANCE.md).
