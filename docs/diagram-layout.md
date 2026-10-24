# Architecture diagram

Team MICROMI. Track 2, PolarFire SoC Icicle Kit.

The project uses the descriptive title Adaptive LiDAR terrain mapping. The native draw.io source is in the adjacent diagrams directory. The source contains editable vector shapes, Segoe UI text and orthogonal connectors. Exports use the full drawing bounds.

## Design

A navy title band identifies the system, team and platform. Blue boundaries contain the RISC-V processor subsystem and shared LPDDR4 memory. A teal boundary contains the FPGA terrain-mapping engine. Amber marks optional semantic preprocessing. Data connections are solid, processor pass control is dashed blue, and optional semantic metadata is dashed amber.

The diagram uses formal labels and complete terms. Proposed architecture appears once in the subtitle. The sheet makes no measured performance or completed-hardware claim.

## Data movement

The host supplies recorded LiDAR frames over Gigabit Ethernet. The frame receiver verifies the frame ID and point count, then writes point records to an owned shared-memory buffer. Host CNN inference may supply class and confidence metadata aligned with the same frame and point IDs. Geometry-based operation remains available without those labels.

The shared-memory boxes describe storage regions. The MSS memory controller arbitrates processor and fabric access. The AXI interface reads retained point records and cell state, then writes completed maps and hardware status. RISC-V software configures buffer addresses, schedules passes and reads completed results for Ethernet export to the host. Read and write operations have separate directed connections.

## Refinement

The first pass validates points, crops the 32 m square region, calculates cell addresses and accumulates minimum height, maximum height, height sum, point count and observation flags. Refinement analysis uses height range, coverage and optional host label cues to request fine tiles within a fixed-capacity pool.

The second pass replays the original point records. Coordinate binning uses the selected tile table and accumulation fills the fine cells from those original points. Refinement analysis makes allocation decisions during the first pass; second-pass data proceeds to map commit. Splitting coarse aggregates cannot recover the distribution of points among child cells.

The tile manager keeps coarse coverage when the fine-tile pool is full and reports the budget limit. Status registers record accepted, invalid and dropped points, transfer stalls and tile-budget limits. Cells without observations remain unknown. The initial cell sizes are 10, 20 and 40 cm, with selective 5 cm cells as an extension.

## Scope and export

The architecture extends our SIH 26053 adaptive semantic 2.5D LiDAR work. It covers single-frame mapping. CNN inference stays on the host. Multi-frame SLAM and safety-control integration remain outside the proposed scope.

The submitting team manually revised the final draw.io source and supplied its PNG preview. Both supplied files are retained byte for byte. The PDF and SVG are regenerated from that final source using draw.io Desktop and checked before publication.
