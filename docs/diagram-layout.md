# Diagram notes

The [system diagram](../diagrams/MICROMI-System-Block-Diagram.pdf) follows LiDAR data from the host computer, through the RISC-V processor and shared memory, into the FPGA mapper and back to the host.

Blue groups show the processor and shared memory. Teal shows the FPGA mapper. Amber marks optional semantic processing on the host. Solid arrows carry data; dashed arrows show pass control or optional metadata.

The FPGA first builds a coarse map and selects areas for refinement. It then reads the original points again to fill the finer cells. If fine-cell storage fills up, the coarse map remains available and the output reports the limit.

## Editing

Open [the draw.io source](../diagrams/MICROMI-System-Block-Diagram.drawio) in draw.io or diagrams.net. Text, shapes and connectors are editable. Export the full drawing bounds when producing a new PDF or SVG.

The current draw.io file and PNG are the team's manually edited versions. The PDF and SVG were exported from that source using draw.io Desktop 32.6.1.
