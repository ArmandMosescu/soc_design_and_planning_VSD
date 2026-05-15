# SOC_design_and_planning_VSD
RTL-to-GDSII physical design flow using OpenLANE &amp; Sky130 PDK Workshop




##
A 2-week hands-on workshop on complete RTL-to-GDSII flow for digital VLSI SoC design, organised by VSD (VLSI System Design) in collaboration with NASSCOM. This repository documents my learning, lab outputs, and key takeaways from each day.
##
# Day 1 — Inception of Open-Source EDA, OpenLANE & Sky130 PDK

When we are looking at an embedded board and spot the chip, we are actually looking at the package, the protective casing of the silicon die, the actual chip sits inside this package and comunicates with the outside world via wire bonding the die's pads to the package's pins.

The pads are placed at the periphery of the core and all the signals between the chip and outside world pass through them.

The core is surrounded by the pads and represent the location where the digital logic is executed.

Concluding, the chip is made up of the package and the die within it, the pads of the die are wire bonded to the package's pins, and the die is made up of it's core and pads.
#

## The Shift to Open-Source Silicon

Historically, ASIC design was restricted by expensive tools and proprietary PDKs (Process Design Kits) hidden behind NDAs. Creating a chip requires three essentials: RTL designs, EDA tools, and PDK data.

The landscape changed in June 2020 when Google and SkyWater Technology released Sky130, the first open-source PDK. This milestone democratized hardware by removing the legal and financial barriers to chip manufacturing.

__OpenLANE__ is the primary engine of this movement. It integrates multiple open-source tools into a single, automated flow that converts RTL netlists (code) into GDSII files (physical blueprints). By streamlining the path to silicon, OpenLANE makes professional-grade chip design accessible to everyone.

| Stage         | Tools    |
| --------------| -------- |
|Synthesis|	Yosys, ABC|
|Floorplan & PDN|	OpenROAD|
|Placement	|OpenROAD|
|CTS	|TritonCTS|
|Routing	|FastRoute, TritonRoute|
|SPEF Extraction|	OpenRCX|
|GDS Streaming	|Magic, KLayout|
|Timing Analysis	|OpenSTA|
|DRC & LVS	|Magic, Netgen|

## LAB - Running OpenLane for picorv32a file

 Setting up OpenLane 
 First we will access the working directory and launch the tool in interactive mode, then we will prepare the design by merging the LEF files before launching synthesis.
 


```bash
cd ~/Desktop/work/tools/openlane_working_dir/openlane
docker
flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a
run_synthesis
```
After syntheis runs succesfully we will complete the asignment of calculating the  D flip flop ratio.

<img width="594" height="438" alt="Image" src="https://github.com/user-attachments/assets/df023aa0-dc91-4eaf-a65c-5caf64ad6dd9" />

Total cells = 1613

D flip flops= 14876

```bash
D flip flop ratio=(Nr of D flip flops)/(Total nr of cells)=1613/14876=0.1087 ~ 10.87%
```

# Day 2 — Floorplanning and Introduction to Library Cells

Floorplanning is the process of placing everything inside the chip, it relies on 2 parameters:

- __Utilisation Factor__ = (Area of Netlist) / (Total Core Area).   
 A factor of 0.5–0.6 is typical — leaving space for buffers, routing, etc.
- __Aspect Ratio__ = Height / Width of  core
A ratio of 1 = square;  else it's a rectangle.
#
### Preplaced Cells

Cells like: memories, mux, complex ip's, etc are defined as __preplaced cells__ as their placement is user-defined, not automated.
#
### Decoupling Capacitors

Capacitors that supply the circuit block with current are called __decoupling capacitors__ or __decap__.

#
### Power Planning

A proper power grid uses power rings around the blocks and a power mesh across the whole core. VDD and VSS rails are disributed so that every power ring can supply current to the circuits minimising V drop and and electromigration risk

#
### Pin Placement and Logical Cell Blockage

The input and output pins are placed on different sides of the core and their placement depends on the placement of the cells, one important note is that the __CLK__ pins are always bigger than the rest so that they provide the least resistance, as they drive signals to the whole core.

A critical step in order to stop the automated routing tool from placing cells in the pin area, is blocking the I/O ring area.
#
## LAB - Running the Floorplan of our picorv32a design

First, we will continue from the succesful synthesis of our design, adding the run_floorplan command
```bash
cd ~/Desktop/work/tools/openlane_working_dir/openlane
docker
flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a
run_synthesis
run_floorplan
```
After floorplan is complete we will navigate to our runs directory and view the floorplan using the Sky130A technology
```bash
cd ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/15-05_17-40/results/floorplan
magic -T/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def

```
<img width="1437" height="752" alt="Image" src="https://github.com/user-attachments/assets/90d1f18f-14c3-44b3-98fa-27066081398b" />
#
After viewing the floorplan we will place all our cells with the run_placement command and see them on our floorplan
```bash
cd ~/Desktop/work/tools/openlane_working_dir/openlane
docker
flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a
run_synthesis
run_floorplan
run_placement
```
then
```bash
cd ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/15-05_17-40/results/floorplan
magic -T/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def
```
<img width="1820" height="869" alt="Image" src="https://github.com/user-attachments/assets/107fd7a2-389a-4e53-9bfc-6b58c9dc349b" />
