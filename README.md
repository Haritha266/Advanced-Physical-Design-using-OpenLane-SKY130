
# Advanced-Physical-Design-using-OpenLane-SKY130


This is a 5 day workshop done on Advanced Physical design which is a crucial part of VLSI design flow using open source. In a glimpse, we take a netlist & produced GDSII file from it.


## What is Physical design?

Firstly, let us understand what is meant by the physical design. Physical design is the process of turning a design into manufacturable geometries. It comprises a number of steps, including floorplanning, placement, clock tree synthesis, and routing.

Physical design begins with a netlist, which is synthesized from RTL. The netlist describes the components of a circuit and how they connect.

Floorplanning is the first major step. It involves identifying which structures should be placed near others, taking into account area restrictions, speed, and the various constraints required by components.

Partitioning divides a chip into functional blocks.

Placement determines the locations of each component or block on the die, considering timing and interconnect length.

Clock tree synthesis involves inserting buffers or inverters such that the clock is distributed evenly to sequential elements in a design, minimizing skew and latency.

Routing determines the paths of interconnects, including standard cell and macro pins. This stage completes all connections defined in the netlist, ideally in the most efficient way and without violating timing constraints.

The final output of the physical design process is typically GDSII, a data format representing layout information.
