
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

### ASIC design Flow:
- **Sythesis** : RTL is converted into a gate level netlist made up of components of standard cell libary (SCL). 
- **Floor Planning/ Power Planning** : Objective is to plan silicon area and create robust power distribution network. The power network usually uses the upper metal layer which are thicker than lower layer and thus lower resistance. This lowers the IR drop problem
 - **Placement** : There are two steps, first is global placement which is the general optimal positons for cells  and might not be legal. Next is detailed placement which is the actual legal placements of the cells from global placement.
 - **Clock tree synthesis** : it delivers clock to all clock cells with minimum skew & latency and is usually a tree (H-tree, X-tree ... )
 - **Routing** : Use horizontal and vertical wires to connect cells together. The router uses PDK information (thickness, pitch, width,vias) for each metal layer to do the routing. The Sky130 defines 6 routing layers. It do global routing (use coarse grain grids) and detailed routing (use fine grain grids).
 - **Verification before sign-off** : Involves physical verification like DRC and LVS and timing verification. Design Rule Checking or DRC ensures final layout honors all design rules and Layout versus Schematic or LVS ensures final layout matches the gate level netlist from synthesis phase. Timing verification ensures timing constraints are met.
![Screenshot (279)](https://user-images.githubusercontent.com/83575446/215494588-e33e8ebd-0c3c-4187-b807-b04f41de1946.png)

## What is Openlane?

 [OpenLane](https://github.com/The-OpenROAD-Project/OpenLane) = An open-source ASIC development flow reference. It consists of multiple open-source tools needed for the whole RTL to GDSII flow. This is tuned epecially for Sky130 PDK. It also works for OSU 130nm. It is recommended to read the [OpenLANE documentation](https://openlane.readthedocs.io/en/latest/)  before moving forward.
 
 ![image](https://user-images.githubusercontent.com/87559347/182759711-6b9352ec-7652-4589-af31-53a409eb2830.png)

- The input for the whole flow are the rtl files, sdc file, and PDK files. The output is GDSII/LEF file.

- Yosys is used to convert the HDL to gate level netlist using generic components. The [ABC script](http://people.eecs.berkeley.edu/~alanmi/abc/) is then used to map the generic components to the standard cell library of the PDK. These ABC scripts is used to make various synthesis strategies (using the Synthesis Exploration) which can optimize the design either with least area or best timing.  

- The Logic Equivalency Cheking (LEC) is used to compare the resulting netlist after optimization of place and route to the gate level netlist from synthesis phase

- Antenna Rules Violation = long wire segments will act as antennna and will accumulate charges, this might damage the connected transistor gates. Solution is to either use bridging or antenna diode insertion to leak away the charges  

### OpenLane Directory Hierarchy:

``` 
├── OOpenLane             -> directory where the tool can be invoked (run docker first)
│   ├── designs          -> All designs must be extracted from this folder
│   │   │   ├── picorv32a -> Design used as case study for this workshop
│   |   |   ├── ...
|   |   ├── ...
├── pdks                 -> contains pdk related files 
│   ├── skywater-pdk     -> all Skywater 130nm PDKs
│   ├── open-pdks        -> contains scripts that makes the commerical PDK (which is normally just compatible to commercial tools) to also be compatible with the open-source EDA tool
│   ├── sky130A          -> pdk variant made especially compatible for open-source tools
│   │   │  ├── libs.ref  -> files specific to node process (timing lib, cell lef, tech lef) for example is `sky130_fd_sc_hd` (Sky130nm Foundry Standard Cell High Density)  
│   │   │  ├── libs.tech -> files specific for the tool (klayout,netgen,magic...) 
```

Inside a specific design folder contains a `config.tcl` which overrides the default settings on OpenLANE. These configurations are specific to a design (e.g. clock period, clock port, verilog files...). The priority order for the OpenLANE settings:
1. sky130_xxxxx_config.tcl in `OpenLane/designs/[design]/`
2. config.tcl in `OpenLane/designs/[design]/`
3. Default values in `OpenLane/configuration/`

### Lab [Day 1] - Determine Flip-flop Ratio:
The task is to find the flip-flop ratio ratio for the design `picorv32a`. This is the ratio of the number of flip flops to the total number of cells. For the OpenLane installation, the steps are very straight forward and can be found on the [OpenLane repo](https://github.com/The-OpenROAD-Project/OpenLane).

**1. Run OpenLANE:**
 - `cd work/tools/openlane_working_dif/openlane` = set this directory
 - `docker` = Open the docker platform inside the `openlane/`
 - `% flow.tcl -interactive` = run script for automating the whole RTL to GDSII flow but in step by step `-interactive` mode
 - `% package require openlane 0.9` = retrives all dependencies for running v0.9 of OpenLANE  
 
<img width="458" alt="o" src="https://user-images.githubusercontent.com/83575446/215518032-3000c1da-d551-4fdd-aa31-a780750e3a03.png">
 
**2. Design Setup Stage:**
 - `% prep -design picorv32a` = Setup the filesystem where the OpenLANE tools can dump the outputs. This also creates a `run/` folder inside the specific design directory which contains the command log files, results, and the reports dump by each tools. These folders will be empty for now except for lef files generated by this design setup stage. This merged the [cell LEF files](https://teamvlsi.com/2020/05/lef-lef-file-in-asic-design.html) `.lef` and [technology LEF files](https://teamvlsi.com/2020/05/lef-lef-file-in-asic-design.html) `.tlef` generating `merged.nom.lef` inside `run/tmp/`
 
<img width="959" alt="1" src="https://user-images.githubusercontent.com/83575446/215518224-44e7fca5-08c9-4f3f-ace0-dec2caefb226.png">

**3. Run synthesis:**
 - `% run_synthesis` = Run yosys RTL synthesis, ABC scripts (for technology mapping), and OpenSTA.  
 
 <img width="728" alt="0 calci" src="https://user-images.githubusercontent.com/83575446/215518318-dec3390f-e0de-4c29-9ebf-19cf3acb4b66.png">

 
**The flipflop ratio is (number of flip flops)/(total number of cells) is 1613/14876 = 0.10843. Or 10.843%**

After running synthesis, inside the `runs/[date]/results/synthesis` is `picorv32a_synthesis.v` which is the mapping of the netlist to standard cell library using ABC. The `runs/[date]/reports/synthesis` will contain synthesis statistic reports and static timing analysis reports. The `runs/[date]/synthesis/logs` contains log files for the terminal output dumps for running yosys and OpenSTA.

# DAY 2: Good Floorplan vs Bad Floorplan and Introduction to Library Cells

### Floorplan Stage:

1. Define height and width of core and die.   
Core is where the logic blocks are placed and this seats at the center of the die. The width and height depends on dimensions of each standard cells on the netlist.
Utilization factor is (area occupied by netlist)/(total area of the core)
In practical scenario, utilization factor is 0.5 to 0.6. This is space occupied by netlist only, the remaining space is for routing and more additional cells. 
Aspect ratio is (height)/(width)
Aspect ratio of core, so only aspect ratio of 1 will produce a square core shape.

2. Define location of Preplaced Cell.   
These are reusable complex logicblocks or modules or IPs or macros that is already implemented (memory, clock-gating cell, mux, comparator...) . The placement on the core is user-defined and must be done before placement and routing (thus preplaced cells). The automated place and route tools will not be able to touch and move these preplaced cells so this must be very well defined

3. Surround preplaced cells with decoupling capacitors. 
The complex preplaced logicblock requires a high amount of current from the powersource for current switching. But since there is a distance between the main powersource and the logicblock, there will be voltage drop due to the resistance and inductance of the wire. This might cause the voltage at the logicblock to be not within the [noise margin](https://www.electronics-tutorial.net/digital-logic-families/noise-margin/) range anymore (logic is unstable). The solution is to use decoupling capacitors near the logic block, this capacitor will send enough current needed by the logicblock to switch within the noise margin range.

![image](https://user-images.githubusercontent.com/87559347/183011079-5f08eb01-28cf-4617-bcbc-f8f26eacc83d.png)

4. Power Planning
Decoupling capactor for sourcing logic blocks with enough current is not feasible to be applied all over the chip but only on the critical elements (preplaced complex logicblocks). Large number of elements switching to logic 0 might cause ground bounce due to large amount of current that needs to be sink at the same time, and switcing to logic 1 might cause voltage droop due to not enough current from the powersource to source needed current of all elements. Ground bounce and voltage droop might cause the voltage to not be within the noise margin range. The solution is to have multiple powersource taps (power mesh) where elements can source current from the nearest VDD and sink current to the nearest VSS tap. This is the reason why most chips have multiple powersource pins.

5. Pin Placement
The input and output ports are placed on the space between the core and the die. The placements of the ports depens on where the cells connected to those ports are placed on the core. The clock port is thicker(least resistance path) than data ports since this clock must be capable to drive the whole chip.

6. Logical Cell Placement Blockage
This makes sure that the automated placement and routing tool does not place any cell on the pin locations of the die.

Below are all 6  steps for floor planning:  

![image](https://user-images.githubusercontent.com/87559347/183446309-a0714ec5-0619-4327-bdfe-890c19cc97e0.png)


### Placement Stage:
1. Bind the netlist to a physical cell with real dimensions. The physical cell will come from a library that can provide multiple options for shapes, dimensions, and delay for same cells. 
2. Next is placement of those physical cells to the floorplan. The flip flops must be placed as near as possible to the input and output pins to reduce timing delay. 
3. Optimize placement to maintain signal integrity. This is where we estimate wirelength and capacitance (C=EA/d) and based on that insert repeaters/buffers. The wirelength will form a resistanace which will cause unnecessary voltage drop and a capacitance which will cause a slew rate that might not be permissible for fast current switching of logic gates. The solution to reduce resistance and capacitance is to insert buffers for long routes that will act as intermediary and separate a single long wire to multilple ones. Sometime we also do abutment where logic cells are placed very close to each other (almost zero delay) if it has to run at high frequency (2GHz). Crisscrossing of routes is a normal condition for PnR since we can use separate metal layer (using vias) for crisscrossed path.
4. After placement optimization, We will setup timing analysis using idle clock (zero delay for wires and has no clock buffer related delays) considering we have not yet done CTS.   

The goal of placement is not yet on timing but on congestion. Also, standard cells are not placed on floorplan stage, it is done on Placement stage. Macros or preplaced cells are the ones placed on floorplan stage.Macros or preplaced cells are placed on floorplan stage.

![image](https://user-images.githubusercontent.com/87559347/183224947-67a29c54-9a18-45a4-bbd1-9132bcebc304.png)  

Placement is done on two stages:
 - Global Placement = placement with no legalizations and goal is to reduce wirelength. It uses Half Perimeter Wirelength (HPWL) reduction model. 
 - Detailed Placement = placement with legalization where the standard cells are placed on stadard rows, abutted, and must have no overlaps    

### Lab [Day 2] - Determine Die Area: 

**1. Set configuration variables.** Before running floorplan stage, the configuration variables or switches must be configured first. The configuration variables are on `openlane/configuration`:  

```
.
├── README.md      
├── checkers.tcl
├── cts.tcl
├── floorplan.tcl  
├── general.tcl
├── lvs.tcl
├── placement.tcl
├── routing.tcl
└── synthesis.tcl 

```  

The  `README.md` describes all configuration variables for every stage and the tcl files contain the default OpenLANE settings. All configurations accepted by the current run is on `openlane/designs/picorv32a/runs/config.tcl`. This may come either from (with priority order):
 - PDK specific configuration inside the design folder
 - `config.tcl` inside the design folder
 - System default settings inside `openlane/configurations`
 
**2. Run floorplan on OpenLane:** `% run floor_plan`

**3. Check the results.** The output of this stage is `runs/[date]/results/floorplan/picorv32a.floorplan.def` which is a [design exchange format](https://teamvlsi.com/2020/08/def-file-in-vlsi-design-exchange.html), containing the die area and positions. 
```
...........
DESIGN picorv32a ;
UNITS DISTANCE MICRONS 1000 ;
DIEAREA ( 0 0 ) ( 660685 671405 ) ;
............
```
The die area here is in database units and 1 micron is equivalent to 1000 database units. **Thus area of the die is (660685/1000)microns\*(671405/1000)microns = 443587 microns squared.** 

**4. View the layout on magic**. Open def file using `magic` as follows: 

<img width="794" alt="directory" src="https://user-images.githubusercontent.com/83575446/215528630-edf4e34d-f3c8-4ba5-8de5-0ef0d3939f87.png">

Then the following window opens

<img width="955" alt="6 floorplan" src="https://user-images.githubusercontent.com/83575446/215527011-b7b45518-6af1-4db2-865b-3913f707b5b8.png">

To center the view, press "s" to select whole die then press "v" to center the view. Point the cursor to a cell then press "s" to select it, zoom into it by pressing 'z". Type "what" in `tkcon` to display information of selected object. These objects might be IO pin, decap cell, or well taps as shown below.  

<img width="802" alt="tkcon" src="https://user-images.githubusercontent.com/83575446/215530325-b50072f3-0fdf-4493-babe-474794dcbd7f.png">

 Useful Magic commands are listed on the [Magic Commands section](https://github.com/AngeloJacobo/OpenLANE-Sky130-Physical-Design-Workshop#magic-commands).

**5 Run placement:** `% run_placement`. This commmand is a wrapper which does global placement (performed by RePlace tool), Optimization (by Resier tool), and detailed placement (by OpenDP tool). It displays hundreds of iterations displaying HPWL and OVFL. The algorithm is said to be converging if the overflow is decreasing. It also checks the legality. 

**6. View the output of this stage**. The output of this stage is `runs/[date]/results/placement/picorv32a.placement.def.` To see actual layout after placement, open def file using `magic`:  

```
magic -T /home/kunalg123/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def
```  
The placement is as follows

<img width="668" alt="placement" src="https://user-images.githubusercontent.com/83575446/215532046-100d8a25-504c-4fbb-af2c-dd549ab45aee.png">

### Library Characterization:
Of all RTL-to-GDSII stages, one common thing that the EDA tool always need is data from the library of gates which keeps all standards cells (and, or, buffer gates,...), macros, IPs, decaps, etc. Same cells might have different flavors inside the library (different sizes, delays, threshold voltage). Bigger cell sizes means bigger drive strength to drive longer and thicker wires. Bigger threshold voltage (due to bigger size) will take more time to switch(slower clock) than those with smaller threshold voltage.  

A single cell needs to go through the cell design flow. The inputs to make a single cell comes from the foundry Process Design Kits:
 - DRC & LVS Rules = tech files and poly subtrate paramters (CUSTOME LAYOUT COURSE)
 - SPICE Models  = Threshold, linear regions, saturation region equations with added foundry parameters. Including NMOS and PMOS parameteres (Ciruit Deisgn and Spice simulation Course)
 - User defined Spec = Cell height (separation between power and ground rail), Cell width (depends on drive strength), supply voltage, metal layer requirement (which metal layer the cell needs to work)

The library cell developer must adhere to the rules given on the inputs so that when the cell is used on a real design, there will be no errors. Next is design the library cell:
1. Design the circuit function (Output: circuit design language (CDL))
2. Model the pmos and nmos that meets input library requirement
3. Layout the design using Euler's path and sticky diagram to produce best area. This can be done on `magic` layout tool.The outputs are:
   - GDSII (layout file)
   - LEF (defines the width and height of cell)
   - extract spice netlist .cir (parasitics of each element of cell: resistance, capacitance)
 Afte design is characterization using GUNA software, where the outputs are timing, noise, and power characterization.
 
 ### Timing Characterization:
 
It contains :
Propogation Delay - (out_thr)-time(in_thr)
Transition Time - time(slew_high_rise_thr)-time(slew_low_rise_thr)} or time(slew_high_fall_thr)-time(slew_low_fall_thr)
 
Below are the timing variables for slew. This is two inverters in series, red is output of first inverter and blue is output of second inverter:  

![image](https://user-images.githubusercontent.com/87559347/183231913-a9b3826b-5139-4bdc-b12b-3495b87cd8b9.png)

Below are the timing variables for propagation delay. The red is input waveform and blue is output waveform of the buffer. The left side is rise delay and right side is fall delay.

![image](https://user-images.githubusercontent.com/87559347/183232515-fe3cef76-8a2f-475d-9a64-392fc2fda111.png)

Negative propagation delay is unexpected. That means the output comes before the input so designer needs to choose correct threshold point to produce positive delay. Delay threshold is usually 50% and slew rate threshold is usually 20%-80%.

# DAY 3: Design a Library Cell using Magic Layout and Ngspice Characterization

### Designing a Library Cell:
1. SPICE deck = component connectivity (basically a netlist) of the CMOS inverter.
2. SPICE deck values = value for W/L (0.375u/0.25u means width is 375nm and lengthis 250nm). 
3. PMOS should be wider in width(2x or 3x) than NMOS. 
4. The gate and supply voltages are normally a multiple of length (in the example, gate voltage can be 2.5V)  
5. Add nodes to surround each component and name it. This will be used in SPICE to identify a component.    

### SPICE Deck Netlist Description:  

![image](https://user-images.githubusercontent.com/87559347/183240195-608727e5-2d04-4e44-ab4a-2df545cd13ea.png)

**Notes:**
 - Syntax for the PMOS and NMOS descriptiom:
     - `[component name] [drain] [gate] [source] [substrate] [transistor type] W=[width] L=[length]`
 - All components are described based on nodes and its values
 - `.op` is the start of SPICE simulation operation where Vin will be sweep from 0 to 2.5 with 0.5 steps
 - `tsmc_025um_model.mod` is the model file containing the technological parameters for the 0.25um NMOS and PMOS

### SPICE Analysis for Switching Threshold and Propagation Delay:
CMOS robustness depends on:  

1. Switching threshold = Vin is equal to Vout. This the point where both PMOS and NMOS is in saturation or kind of turned on, and leakage current is high. If PMOS is thicker than NMOS, the CMOS will have higher switching threshold (1.2V vs 1V) while threshold will be lower when NMOS becomes thicker.
2. Propagation delay = rise or fall delay
 ### CMOS Fabrication Process (16-Mask CMOS Process):  
 **1. Selecting a substrate** = Layer where the IC is fabricated. Most commonly used is P-type substrate  
 **2. Creating active region for transistor** = Separate the transistor regions using SiO2 as isolation
  - Mask 1 = Covers the photoresist layer that must not be etched away (protects the two transistor active regions)
  - Photoresist layer = Can be etched away via UV light  
  - Si3N4 layer = Protection layer to prevent SiO2 layer to grow during oxidation (oxidation furnace)  
  - SiO2 layer = Grows during oxidation (LOCOS = Local Oxidation of Silicon) and will act as isolation regions between transistors or active regions  
3. **N-Well and P-Well Fabrication** = Fabricate the substrate needed by PMOS (N-Well) and NMOS (P-Well)  
  - Phosporus (5 valence electron) is used to form N-well  
  - Boron (3 valence electron) is used to form P-Well.  
  - Mask 2 protects the N-Well (PMOS side) while P-Well (NMOS side) is being fabricated then Mask 3 while N-Well (PMOS side) is being fabricated
4. **Formation of Gate** = Gate fabrication affects threshold voltage. Factors affecting threshold voltage includes:    
    Main parameters are:
  - Doping Concentration = Controlled by ion implantation (Mask 4 for Boron implantation in NMOS P-Well and Mask 5 for Arsenic implantation in PMOS N-Well)
  - Oxide capacitance = Controlled by oxide thickness  (SiO2 layer is removed then rebuilt to the desire thickness)  
  - Mask 6 is for gate formation using polysilicon layer.
5. **Lightly Doped Drain formation** = Before forming the source and drain layer, lightly doped impurity is added: 
  - Mask 7 for N- implantation (lightly doped N-type) for NMOS 
  - Mask 8 for P- implantation (lightly doped P-type) for PMOS.  
Heavily doped impurity (N+ for NMOS and P+ for PMOS) is for the actual source and drain but the lightly doped impurity will help maintain spacing between the source and drain and prevent hot electron effect and short channel effect. 
6. **Source and Drain Formation** = Mask 9 is for N+ implantation and Mask 10 for P+ implantation  
 - Channeling is when implantations dig too deep into substrate so add screen oxide before implantation
 - The side-wall spacers maintains the N-/P- while implanting the N+/P+    
7. **Form Contacts and Interconnects** =  TiN is for local interconnections and also for bringing contacts to the top. TiS2 is for the contact to the actual Drain-Gate-Source. Mask 11 is for etching off the TiN interconnect for the first layer contact. 
8. **Higher Level Metal Formation** = We need to planarize first the layer via CMP before adding a metal interconnect. Aluminum contact is used to connect the lower contact to higher metal layer. Process is repeated until the contact reached the outermost layer.
 - Mask 12 is for first contact hole
 - Mask 13 is for first Aluminum contact layer
 - Mask 14 is for second contact hole
 - Mask 15 is for second Aluminum contact layer. Mask 16 is for making contact to topmost layer. 
 
![image](https://user-images.githubusercontent.com/87559347/187158161-4d230654-5102-4225-8e58-d6d8ed950990.png)

### Lab 3 - Introduction to Sky130 basic layers layout and LEF using inverter:
  
1. Clone [vsdstdcelldesign](https://github.com/nickson-jose/vsdstdcelldesign). 
2. Copy the techfile `sky130A.tech` from `pdks/sky130A/libs.tech/magic/` to directory of the cloned repo. In the below figure, you can see tech file getting added to vsdstdcelldesign

<img width="959" alt="7 techfile added" src="https://user-images.githubusercontent.com/83575446/215544859-deadc35e-d812-47dd-acd1-6a84c83f1502.png">

3. View the mag file using magic `magic -T sky130A.tech sky130_inv.mag &`:  

<img width="457" alt="inverter" src="https://user-images.githubusercontent.com/83575446/215546262-98a5d2f0-c150-4f14-a9e0-7bca19b5caf3.png">

4. Try DRC at top tool bar to find the DRC violations and type `what` in tkcon window to find the error

<img width="958" alt="11 dimensions of grid" src="https://user-images.githubusercontent.com/83575446/215548187-5df11006-b78c-4ee8-8458-0191aa7421ee.png">

### Extraction of SPICE file & Slew Rate and Propagation Delay Characterization 

1. Make an extract file `.ext` by typing `extract all` in the tkon terminal. 
2. Extract the `.spice` file from this ext file by typing `ext2spice cthresh 0 rthresh 0` then `ext2spice` in the tcon terminal.  

<img width="960" alt="10 spice created" src="https://user-images.githubusercontent.com/83575446/215549589-55e619df-34ac-4060-ad89-6c34ea426a58.png">

3. Modify the spice file to be able to plot a transient response
4. Change the grid size by checking in layout

<img width="958" alt="11 dimensions of grid" src="https://user-images.githubusercontent.com/83575446/215550902-cb4c00c1-86e0-4753-a13a-92bccb914956.png">

5. Include pmos, nmos libraries from `\vsdstdcelldesign-master\libs\nshort.lib` and `\vsdstdcelldesign-master\libs\nphort.lib` which are the spice model files for nmos and pmos
6. Rename the pmos and nmos names

<img width="959" alt="12 pmos name" src="https://user-images.githubusercontent.com/83575446/215551628-bb865cca-7ce9-482d-b300-c49a7f1d6ac6.png">

8. Change C3(Cload) to 2fF(increase it) to get the higher delay
9. Define ground, supply & input signals

```
* SPICE3 file created from sky130_inv.ext - technology: sky130A

.option scale=0.01u
.include ./libs/pshort.lib
.include ./libs/nshort.lib

* .subckt sky130_inv A Y VPWR VGND
M0 Y A VGND VGND nshort_model.0 ad=1435 pd=152 as=1365 ps=148 w=35 l=23
M1 Y A VPWR VPWR pshort_model.0 ad=1443 pd=152 as=1517 ps=156 w=37 l=23
C0 A VPWR 0.08fF
C1 Y VPWR 0.08fF
C2 A Y 0.02fF
C3 Y VGND 0.18fF
C4 VPWR VGND 0.74fF
* .ends

* Power supply 
VDD VPWR 0 3.3V 
VSS VGND 0 0V 

* Input Signal
Va A VGND PULSE(0V 3.3V 0 0.1ns 0.1ns 2ns 4ns)

* Simulation Control
.tran 1n 20n
.control
run
.endc
.end
```  

10. Open the spice file by typing `ngspice sky130A_inv.spice`. 
11. Generate a graph using `plot y vs time a` :  

<img width="959" alt="13 spice graph" src="https://user-images.githubusercontent.com/83575446/215551955-d3adaaf5-198b-4e70-a4bc-d153f6e50401.png">

Using this transient response, we will now characterize the cell's slew rate and propagation delay:  
- Rise Transition [output transition time from 20%(0.66V) to 80%(2.64V)]:
    - **Tr_r = 2.19981ns - 2.15739ns = 0.04242 ns**  

- Fall Transition [output transition time from 80%(2.64V) to 20%(0.66V)]:
   - **Tr_f = 4.0672ns - 4.04007ns = 0.02713ns**   

- Rise Delay [delay between 50%(1.65V) of input to 50%(1.65V) of output]:
   - **D_r = 2.18197ns - 2.15003ns = 0.03194ns**   

- Fall Delay [delay between 50%(1.65V) of input to 50%(1.65V) of output]:
   - **D_f = 4.05364ns - 4.05001ns =0.00363ns**  
