
# Advanced-Physical-Design-using-OpenLane-SKY130


This is a 5 day workshop done on Advanced Physical design which is a crucial part of VLSI design flow using open source. In a glimpse, we take a netlist & produced GDSII file from it.



## Sky130 Day 1 - Inception of open-source EDA, OpenLANE and Sky130 PDK

A chip generally consists of PADS (where signals go in & out through pads), core (where logic gates sits) and Die (size of chip)

The core of the chip will contain two types of blocks as follows:
 - **Foundry IP Blocks** (e.g. ADC, DAC, PLL, and SRAM) = blocks which requires some amount of intelligent techniques to build which can only be designed by foundries.
 - **Macro blocks** (e.g. RISC-V SOC and SPI) = pure digital logic blocks compared to IP's which might require some analog parts. 

 Open Source Digital ASIC Design requires three open-source components:  
- **RTL Designs** = github.com, librecores.org, opencores.org
- **EDA Tools** = OpenROAD, OpenLANE,QFlow  
- **PDK** = Google + Skywater 130nm Production PDK

![Screenshot (277)](https://user-images.githubusercontent.com/83575446/215494429-0618b809-56e7-4206-bb10-d468faebf570.png)


**PDK (Process Design Kit)** = A set of data files and documents which serves as the interface between the designer and the fab. This includes cell libraries, IO libraries, design rules (DRC, LVS, etc.)

### Simplified RTL to GDSII Flow:
- **Sythesis** : RTL is converted into a gate level netlist made up of components of standard cell libary (SCL). 
- **Floor Planning/ Power Planning** : Objective is to plan silicon area and create robust power distribution network. The power network usually uses the upper metal layer which are thicker than lower layer and thus lower resistance. This lowers the IR drop problem
 - **Placement** : There are two steps, first is global placement which is the general optimal positons for cells  and might not be legal. Next is detailed placement which is the actual legal placements of the cells from global placement.
 - **Clock tree synthesis** : it delivers clock to all clock cells with minimum skew & latency and is usually a tree (H-tree, X-tree ... )
 - **Routing** : Use horizontal and vertical wires to connect cells together. The router uses PDK information (thickness, pitch, width,vias) for each metal layer to do the routing. The Sky130 defines 6 routing layers. It do global routing (use coarse grain grids) and detailed routing (use fine grain grids).
 - **Verification before sign-off** : Involves physical verification like DRC and LVS and timing verification. Design Rule Checking or DRC ensures final layout honors all design rules and Layout versus Schematic or LVS ensures final layout matches the gate level netlist from synthesis phase. Timing verification ensures timing constraints are met.
![Screenshot (279)](https://user-images.githubusercontent.com/83575446/215494588-e33e8ebd-0c3c-4187-b807-b04f41de1946.png)




