# Physicaldesign_openlane

This project is done in the course ["Advanced Physical Design using OpenLANE/Sky130"]

## Table of Contents

- [Day1-Introduction](#Day1-introduction)
  
- [Day2-Good Floorplan vs bad Floorplan](#Day2-Good-Floorplan-vs-bad-Floorplan)
  
- [Day3-]


# Day1

<details>
  <summary>Introduction</summary>
With the introduction of open-source technology for chip creation, many RTL designs and EDA Tools were made available for free. The [SKY130 PDK] fills the gap in a whole Open source chip development.(https://skywater-pdk.readthedocs.io/en/latest/rules.html) from Skywater Technologies and Google. There were a number of EDA Tools with distinct functions throughout the design cycle. The design flow was not clear, and the Skywater pdk was only compatible with industrial equipment.  These problems were addressed by [OpenLane](https://github.com/The-OpenROAD-Project/OpenLane), which offered a fully automated and tidy RTL to GDSII flow. OpenLane is not a product; rather, it is a flow made up of a number of EDA tools, automation scripts, and Skywater-pdks that have been optimized for use with open-source EDA tools.    
</details>

<details>
 <summary> Overall Design Flow</summary>
An RTL design is created for a design specification using HDLs like Verilog or VHDL, or it can be created using high-level synthesis tools like SystemC, MATLAB HDL Coder, Bluespec, etc. 
The process of converting the RTL Netlist into a manufactured IC then starts, and is known as the Physical Design Flow.
Floor planning, which entails placing preplaced cells, power planning, etc., comes first in the physical design process. The placement of logical synthesis comes next. So that the clock's skew is at a minimal or under the necessary threshold, we now perform CTS (Clock Tree Synthesis). Following CTS, all of the assembled components are routed. A process known as "Static Timing Analysis" is used between each and every step in the physical design flow, from logic synthesis through routing, to analyze the design at each stage and confirm that it is actually right.  Magic is an open source application to view the layouts for every stage. You can extract a tiny netlist, run a SPICE simulation, and compare the results with the post-layout Simulation using ngspice.

Physical Design begins with Floor planning - placing the preplaced cells, power planning etc., secondly Placement 

![Simplified Design Flow](https://github.com/shariethernet/Physical-Design-with-OpenLANE-using-SKY130-PDK/blob/main/images/1.PNG)
  
</details>
<details>
  <summary>OpenLane Flow</summary>

### 1.  Synthesis 
The RTL Level Design is then synthesized using a Logic Synthesizer. We use Yosys which is an Open Source Logic Synthesizer. The RTL Netlist is then  converted into a synthesised netlist where there are details about the standard cells and its implementations. Yosys takes the RTL design and timing .libs and verilog models of standard cells and converts  into  a  RTL Netlist. abc does the tehnology mapping to the required skywater-pdk variants 

### 1.1 Synthesis Strategies
Different strategies can be used to synthesize for the either the least area or the best timing. To analyse this, synthesis exploration utility generates a report showing the effect on delays/timing/area et.,

### 1.2 Deign Exploration Utility 
This is used to suit the design configuration and generate reports with different metrics to select the best. This is also used for regression testing

### 1.3 Design For Test - DFT Insertion
This is an optional step carried out by Fault. It is used to test the design 

###  2. Floor Planning and Power Planning
This is done by OpenROAD flow. The macros and IPs are placed in the core before proceding further. This is called as pre-placement. Floor planning is done separately for the macros and it is called macro floor planning. They are placed in such a way that they are closer to the inputs/outputs/other macros where more connections are present. Then to prevent the loading effects de-coupling capacitors are placed so that the logic states are well within the noise margin. 

When several blocks tap power from a single source, there is a problem of Voltage Droop at the Vdd and Ground Bounce at the Vss which can again push the logic out of the required noise margin into the undefined state. To mitigate this Vdd and Vss are placed as horizontal and vertical strips in the chip so that the blocks can tap power from the nearest source. 

### 3. Placement
There are two types of placement.  The other required logic is placed optimally.
Placement is of two steps
- Global Placement- finds the optimal position for each cells. These positions are not necessarly correct, cells may overlap
- Detialed Placement - After Global placement is done minimal alterations are done to correct the issues

### 4. Clock Tree Synthesis 
To ensure minimum skew the Clock is routed optimally through the circuit using different algorithms. This is done in the OpenROAD flow. This is done by TritonCTS.

### 5. Fake Antenna and diode swapping
Long wires acts as antennas and cause accumulation of charges during the fabrication process damaging the transistor. To avoid this bridging is used to pass the wire through different layers or an antenna diode cell is added to leak away the charges
- OpenLane approach - Insert Fake Diode to every cell input during placement. This matches the footprint of the library of the antenna diode. The Antenna Checker is run to check for violations, if there are violations then the fake diode is swapped with a real one.
- OpenROAD approach - In the global route step, the antenna violation is addressed automatically by inserting an antenan diode
OpenLane allows the user to chose either of the above approaches

###  5. Routing
This step is used to implement the interconnect using the different metal layers specified in the PDK. There are two steps

 - Global Routing - This is done inside the OpenROAD flow (FastRoute)
 - Detailed Routing - This is performed using TritonRoute outside the OpenROAD flow after the global routing. Before performing this step the **Logic Equivalence Check** is performed by Yosys, since OpenROAD does some optimisations the circuit.  

### 6. RC Extraction
From the .def file, the parasitic extraction is done to generate the .spef file (Standard Prasitic Exchange Format) which produces an accurate analog model of the circuit by including the parasitic effects due to wires, parasitic capacitances, etc.,

### 7. STA
At this stage again OpenSTA is used to perform the Static Timing Analysis.  

### 8. Sign-off Steps
- Design Rule Check (DRC) is performed by Magic
- Layout Versus Schematic (LVS) is performed by Netgen

### 9. GDSII Extraction
The routed .def file is used my Magic to generate the GDSII file 

## OpenLane Installation and Environment Setup

Refer to [Kanish R1 GIthub](https://github.com/KanishR1/Physical-Design-Using-Openlane) or [OpenLane build Script by Nikson Jose] for OpenLane installation and environment setup.If the installation is carried out on a Virtual Machine/Linux, the following repository can be used from reference **(https://github.com/nickson-jose/openlane_build_script)**

## Working with OpenLane

### Start Openlane

First go to the openlane directory and open terminal

```
make mount

```
 The terminal changes into the docker instance. Open the OpenLane in interactive mode.
 
 ```
./flow.tcl -interactive

 ```
Set the package required by OpenLane

```package require openlane 0.9```

## Synthesis

Run the synthesis

```run_synthesis```

OpenLane invokes the following

- `Yosys` - RTL Synthesis and maps to yosys generic cells
- `abc` - Technology mapping with the Skywater130 PDK. Here `sky130_fd_sc_hd` Skywater Foundry produced High density standard cells are used.
- `OpenSTA` - This does the Static Timing Analysis on the netlist generated after synthesis and generated the timing reports 

View the synthesis statistics

![Screenshot from 2023-09-08 19-45-56](https://github.com/alwinshaju08/Physicaldesign_openlane/assets/69166205/d82b1b97-6817-4442-a6cd-7c9793a58aeb)


### Key concepts

#### Flops ratio 

- The flop ratio is defined as the ratio of the number of flops to the total number of cells
- Here flop ratio is **1596/10104 = 0.1579** (i.e: 15.8%) [From the synthesis statistics]


</details>

# Day2
<details>
<summary>Chip Floor Planning Consideration</summary>
  
#### Utilisation Factor

- The ratio of area occupied by the cells in the netlist to the total area of the core
- Best practice is to set the utilisation factor less than 50% so that there will be space for optimisations, routing, inserting buffers etc.,

### Aspect Ratio

- Aspect ratio is the ratio of height to the width of the die.
- Aspect Ratio of 1 indicates that the die is a square die

## Floorplanning

Floorplanning involves the following stages

### Pre-Placed cells

- Whenever there is a complex logic which is repeated multiple times or a design given by a third-party it can be perceived as abstract black box with input and output ports, clocks etc ., 
- These modules can be either macros or IP
    - Macro  - It is a module such as CPU Core which are developed by the entity fabicating the chip
    - IP - It is an "Intellectual Propertly" which the entity fabricating the chip gets as a package from a third party or even packaged Hard IPs developed by the same entity. Common examples of IPs are SRAM, PLL, Protocol Converters etc.,

- These Macros and IPs are placed in the core at first before placing the standard cells  and power planning
- These are optimally such that the cells which are more connected to each other are placed nearby and oriented for input and ouputs

### Decoupling Capacitors to the pre placed cells
- The power lines can have some RLC component causing the voltage to drop at the node where it enters the Blocks or the ground of the cell can be at a higher potential than ideally 0V
- When this happens, there is a chance such that the logic transitions are not to the upper or lower noise margins but to the forbidden state causing the circuit to misbehave
- This is prevented by adding a capacitor in parallel with the power and ground node of the block such that the capacitor decouples the block from the power source whenever there is a logic transition

### Power Planning

- When there are several cells or blocks drawing power from the same power rail and sinking power to the same ground pin the following effects are observed
    - Whenever there is alogic transition from 1 to 0 in a large number of cells then there is a Voltage Droop in the power lines as Voltage Drops from Vdd
    - Whener there is a logic transition from 0 to 1 in a large number of cells simultaneously causes the ground potential to raise above 0V calles as Ground Bump
    - These effects pose a risk of driving the logic state out of the specified noise margin.
    - To avoid this the Vdd and Gnd are placed as a grid of horizontal and vertical tracks and the cell nearer to an intersection can tap power or sink power to the Vdd or Gnd intersection respectively

### Pin Placement
 - The input, output and Clock pins are placed optimally such that there is less complication in routing or optimised delay
 - There are different styles of pin placement in openlane like `random pin placement` , `uniformly spaced` etc.,

  </details>

  <details>

<summary>Floorplan run on OpenLANE & review layout in Magic</summary>

**Floorplan envrionment variables or switches:**
1. ```FP_CORE_UTIL``` - core utilization percentage
2. ```FP_ASPECT_RATIO``` - the cores aspect ratio
3. ```FP_CORE_MARGIN``` - The length of the margin surrounding the core area
4. ```FP_IO_MODE``` - defines pin configurations around the core(1 = randomly equidistant/0 = not equidistant)
5. ```FP_CORE_VMETAL``` - vertical metal layer where I/O pins are placed
6. ```FP_CORE_HMETAL``` - horizontal metal layer where I/O pins are placed
 
***Note: Usually, the parameter values for vertical metal layer and horizontal metal layer will be 1 more than that specified in the files***

**Importance files in increasing priority order:**
1. ```floorplan.tcl``` - System default settings
2. ```conifg.tcl```
3. ```sky130A_sky130_fd_sc_hd_config.tcl```
 
 To run the picorv32a floorplan in openLANE:
 
 ```
 run_floorplan
 
 ```

![Screenshot from 2023-09-10 00-56-54](https://github.com/alwinshaju08/Physicaldesign_openlane/assets/69166205/23380a23-e4b3-45fa-9cb1-acd12d95e695)

Post the floorplan run, a .def file will have been created within the ```results/floorplan``` directory. We may review floorplan files by checking the ```floorplan.tcl.``` The system defaults will have been overriden by switches set in conifg.tcl and further overriden by switches set in ```sky130A_sky130_fd_sc_hd_config.tcl.```

To view the floorplan, Magic is invoked after moving to the results/floorplan directory:


![Screenshot from 2023-09-10 01-21-14](https://github.com/alwinshaju08/Physicaldesign_openlane/assets/69166205/4e118dda-0788-40cf-850d-8ee1b696858e)


```
magic -T /home/parallels/.volare/sky130A/libs.tech/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &

```

![Screenshot 2023-09-10 at 1 59 31 AM](https://github.com/alwinshaju08/Physicaldesign_openlane/assets/69166205/87c3c1e9-eb75-4365-932f-9865b3ef6ffd)

One can zoom into Magic layout by selecting an area with left and right mouse click followed by pressing "z" key.

Various components can be identified by using the what command in tkcon window after making a selection on the component.

Zooming in also provides a view of decaps present in picorv32a chip.

The standard cell can be found at the bottom left corner.

You can clearly see I/O pins, Decap cells and Tap cells. Tap cells are placed in a zig zag manner or you can say diagonally
  
</details>

<details>
  <summary>
    Library Binding and Placement
  </summary>
  
  ## Netlist Binding and initial place design

First we need to bind the netlist with physical cells. We have shapes for OR, AND and every cell for pratice purpose. But in reality we dont have such shapes, we have give an physical dimensions like rectangles or squares weight and width. This information is given in libs and lefs. Now we place these cells in our design by initilaising it. 

## Optimize Placement

The next step is placement. Once we initial the design, the logic cells in netlist in its physical dimisoins is placed on the floorplan. Placement is perfomed in 2 stages:

Global Placement: Cells will be placed randomly in optimal positions which may not be legal and cells may overlap. Optimization is done through reduction of half parameter wire length.
Detailed Placement: It alters the position of cells post global placement so as to legalise them.
Legalisation of cells is important from timing point of view.

Optimization is stage where we estimate the lenght and capictance, based on that we add buffers. Ideally, Optimization is done for better timing.

![Screenshot from 2023-09-10 02-08-14](https://github.com/alwinshaju08/Physicaldesign_openlane/assets/69166205/0137e529-9dae-4b60-a4a8-5967f8d4476c)

## Congestion aware Placement 

Post placement, the design can be viewed on magic within results/placement directory:

```
magic -T /home/parallels/.volare/sky130A/libs.tech/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def

```


</details>

## Word of Thanks
I sciencerly thank **Mr. Kunal Gosh**(Founder/**VSD**) for helping me out to complete this flow smoothly.

## Acknowledgement
- Kunal Ghosh, VSD Corp. Pvt. Ltd.
- Chatgpt
- Kanish R,Colleague,IIIT B
- Pruthvi Parate,Colleague,IIIT B
  
## Reference 
- https://www.vsdiat.com
- https://github.com/riscv/riscv-gnu-toolchain
- https://github.com/KanishR1
