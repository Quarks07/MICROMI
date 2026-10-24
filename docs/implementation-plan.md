# MICROMI implementation plan

Team MICROMI proposes a Track 2 prototype on the PolarFire SoC Icicle Kit. It will extend our SIH 26053 adaptive semantic 2.5D LiDAR work. The complete system and FPGA implementation have not yet been built. All sizes below are design calculations; throughput and fidelity objectives require measurement.

## Scope and processing contract

The first implementation will map one recorded frame of at most 100,000 points into a 32 m square region. Supported cell widths will be 40, 20, and 10 cm. A 5 cm mode will follow only if the core pipeline meets its checks. Each cell will describe elevation statistics and observation status, rather than a complete 3D scene.

The host will quantize coordinates to signed integer millimetres and reject non-finite source coordinates before encoding. The FPGA will independently validate record framing, coordinate bounds, and configuration. The initial vertical interval will be -32 m inclusive to +32 m exclusive. Horizontal bounds will also be half-open, with an explicit origin and no wrapping of negative coordinates. Frames will carry an identifier and declared point count.

Full CNN inference and visualization will remain on the host. Semantic metadata will be optional and disabled in the baseline. Multi-frame SLAM, motion compensation, map aging, and safety control are outside scope.

## Modules and dataflow

| Module | Planned responsibility |
|---|---|
| Host replay and reference | Encode recorded points, optionally attach host semantic metadata, send frames over Gigabit Ethernet, and compare returned maps with software. |
| RISC-V software | Receive and validate frame envelopes, own shared-DDR buffers, program bounds and budgets, start transfers, and export completed maps and status. |
| AXI/transfer bridge | Read point buffers, carry accelerator commands, and write results into shared DDR with defined completion and buffer ownership. Validate the exact attachment and transfer mechanism during board bring-up. |
| Point frontend | Validate records, crop the region, calculate cell addresses, and count accepted and rejected points by reason. |
| Cell accumulator | Update minimum, maximum, signed sum, count, and observed/overflow flags. Serialize or forward repeated updates to the same address correctly. |
| Coarse analysis and tile manager | Score geometric variation, optionally include host semantic priorities, assign bounded fine slots, and record unmet refinement requests. |
| Output and instrumentation | Export leaf records, coarse fallback, resolution descriptors, and counters; propagate backpressure without silently dropping accepted points. |

The planned sequence will be receive, coarse pass, tile selection, fine pass, then export. Pass one will fill the 40 cm grid and gather tile scores. The manager will rank requests deterministically, with tile index breaking ties, and choose either 20 or 10 cm for each selected tile. Pass two will replay the same immutable DDR frame and accumulate only its selected fine tiles. Export will identify the active resolution so the host does not count overlapping coarse and fine records twice.

Refinement cannot reconstruct child cells by splitting a parent's minimum, maximum, sum, or count. Fine cells will start unobserved and receive original points during replay. Exhausting the slot pool will retain coarse coverage and set a budget-limit flag. A failed or incomplete pass will mark the frame incomplete. Reusing the retained DDR frame avoids a second network upload.

## Memory budget and numerical formats

Each proposed input record will occupy 16 bytes: three signed 32-bit coordinates and a 32-bit metadata field. A zero metadata field will support geometry-only processing. Each cell record will occupy 24 bytes: a signed 64-bit elevation sum, signed 32-bit minimum and maximum, a 32-bit count, and 32-bit flags. The declared frame and height limits fit these accumulators; range and overflow checks will still be tested.

A 1.6 m square tile will contain 4 by 4 coarse cells, 8 by 8 cells at 20 cm, or 16 by 16 cells at 10 cm. The region will contain 20 by 20 tiles. The first allocator will reserve a full 10 cm slot even for a 20 cm tile, keeping its worst-case footprint simple and fixed.

| Map allocation | Calculation | Bytes |
|---|---|---:|
| Full 40 cm base grid | 80 Ã— 80 Ã— 24 | 153,600 |
| Pool of 128 fine slots | 128 Ã— 16 Ã— 16 Ã— 24 | 786,432 |
| Tile descriptors | 400 Ã— 32 | 12,800 |
| Bounded analysis/output scratch | Reserved | 65,536 |
| Total map allocation | Sum of the above | 1,018,368 |
| Remaining within a 1 MiB cap | 1,048,576 minus total | 30,208 |

These allocations will primarily use shared DDR. On-chip FIFOs and update caches will need a separate budget after synthesis. With all slots at 10 cm, up to 128 of 400 tiles, or 32% of the region, can receive that resolution. This is an allocation limit, not a prediction of retained terrain detail.

For comparison, dense cell records alone occupy 153,600 bytes at 40 cm, 614,400 at 20 cm, 2,457,600 at 10 cm, and 9,830,400 at 5 cm. These calculations exclude uniform-grid metadata. Evaluation will report the same categories for both implementations and will not claim a measured saving from this arithmetic alone.

Two 100,000-point input buffers will require another 3,200,000 bytes. Combined with the proposed map allocation, this is 4,218,368 bytes before software, network buffers, and any separate output copy. The 1 MiB cap applies to map storage, not total board memory. The output will reuse a completed map buffer where practical; any extra buffer will be counted explicitly. A 5 cm tile would need 32 Ã— 32 Ã— 24 = 24,576 bytes, so that extension will require fewer slots or a revised budget.

At the target of one million unique input points per second, encoded ingress payload is 16 MB/s before protocol overhead. Two full passes imply up to two million point visits per second and 32 MB/s of point-buffer reads. Cell read/modify/write traffic, clearing, analysis, and export are additional costs. The transfer bridge and memory arbitration will therefore be measured with concentrated as well as dispersed points.

## Staged schedule

| Period after development begins | Deliverable and exit check |
|---|---|
| Weeks 1 and 2 | Confirm kit programming, Libero SoC and embedded tool access, bridge attachment, and DDR buffer ownership. Freeze formats and build the fixed-point reference. |
| Weeks 3 to 5 | Simulate and bring up a uniform 40 cm accumulator. Match the reference for boundaries, repeated addresses, resets, and stalls. |
| Weeks 6 to 8 | Add coarse scores, bounded allocation, and two-pass refinement. Verify all supported resolutions and exhausted-pool fallback. |
| Weeks 9 to 11 | Integrate Gigabit Ethernet, RISC-V transfers, completion handling, and status export. Exercise malformed and incomplete frames and slow consumers. |
| Weeks 12 to 14 | Compare recorded frames, characterize memory traffic, close timing, and record resource use and latency distributions. |
| Weeks 15 and 16 | Fix demonstrated failures, freeze reproducible builds, and prepare the demonstration and report. Consider optional semantics or 5 cm tiles only after core checks pass. |

The uniform accumulator will remain a working intermediate result. Initial serialized cell updates may establish correctness before pipelining. Any later concurrency will need collision handling and randomized stall verification.

## Verification and reporting

The performance goal will be 100,000 input points per frame at 10 frames per second, equivalent to one million unique input points per second. It is a target, not an achieved result. Complete processing time will include both passes, allocation, clearing, and export. Network transfer, optional host inference, and visualization will also be timed separately, with overlap disclosed.

- Compare integer minimum, maximum, sum, count, observation flags, and tile choices exactly with a fixed-point software reference. Declare any tolerance used when comparing with floating-point source data.
- Test negative coordinates, region edges, empty frames, invalid lengths, out-of-range values, all points in one cell, output stalls, reset during a frame, and an exhausted tile pool. Accepted and rejected counts must reconcile with the declared point count for completed frames.
- Enforce the map cap, record peak allocated bytes, and expose input/output stalls, update conflicts, rejection reasons, overflow, frame completeness, and unmet refinement requests. Empty cells must remain unknown.
- Compare geometry-only adaptive output with uniform 10, 20, and 40 cm maps on identical observations. Measure elevation error and retention of annotated raised edges and depressions with actual returns. Report allocation failures alongside fidelity. Optional semantic priorities will be a separate comparison.
- Record throughput, p50/p95/worst observed frame latency, clock frequency, timing slack, logic and RAM use, and test conditions. Report power only if suitable measurements are available with a comparable baseline.

Public results will separate simulation, synthesis, board measurements, and untested behavior. Prior FPGA LiDAR acceleration and adaptive mapping will be acknowledged. No ASIC implementation, field readiness, or guaranteed speedup is claimed. This proposal package will contain newly written material only, with no private project code or datasets.

## Submission field checks

The plain-text fields were checked using whitespace-separated word counts.

| Field | Words | Maximum | Result |
|---|---:|---:|---|
| Project overview | 249 | 300 | Within limit |
| Innovation and expected impact | 189 | 250 | Within limit |

The platform text is exactly `PolarFire SoC Icicle Kit`. The title is exactly `FPGA-Accelerated Adaptive 2.5D LiDAR Terrain Mapping`.
