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
/openLANE_flow
./flow.tcl -interactive
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
__Utilisation Factor__ = (Area of Netlist) / (Total Core Area)
A utilisation of 0.5–0.6 is typical — leaving space for buffers, routing, etc.
__Aspect Ratio__ = Height / Width of  core
A ratio of 1 = square;  else it's a rectangle.





