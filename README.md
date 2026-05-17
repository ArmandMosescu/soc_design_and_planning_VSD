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

#

# Day 3 — Design library cell using Magic Layout and ngspice characterization

First we will analyze an inverter cell design, the design can be found at:

```bash
https://github.com/nickson-jose/vsdstdcelldesign
```
#
We will create a copy of the technology file in the design's directory
```bash
~/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic$ cp sky130A.tech vsdstdcelldesign/libs
```
then launch Magic layout tool
```bash
~/Desktop/work/tools/openlane_working_dir/openlane/vsdstdcelldesign/libs$ magic -T sky130A.tech sky130_inv.mag
```
<img width="1038" height="617" alt="Image" src="https://github.com/user-attachments/assets/3321d73b-7a38-439a-983e-2e7214844977" />

#
After we extract all the parameters to a .spice file we will modify it like this to include a transient analysis
<img width="718" height="470" alt="Image" src="https://github.com/user-attachments/assets/cf788223-b969-4a7b-b78a-65347508855f" />

We will launch ng spice with the following command, including our pmos and nmos libs as well

```bash
~/Desktop/work/tools/openlane_working_dir/openlane/vsdstdcelldesign/libs$ ngspice sky130_inv.spice nshort.lib pshort.lib
```
And view our plotted input and output
```bash
ngspice 1 -> plot y vs time a
```
<img width="1326" height="745" alt="Image" src="https://github.com/user-attachments/assets/6de9b18c-c031-474d-9ee1-52c5a905a813" />

#

# Lab exercise to fix poly.9 error in Sky130 tech-file
#
Incorrectly implemented poly.9 rule resulting in no drc violation even though the spacing < 0.48u. Find problem in the DRC section of the old magic tech file for the skywater process and fix them.
#
We will download the drc_test archive from 

```bash
http://opencircuitdesign.com/open_pdks/archive/drc_tests.tgz
```

We will launch Magic layout tool and select "met3.mag"

<img width="1196" height="732" alt="Image" src="https://github.com/user-attachments/assets/929914b4-d76e-42fd-b5c7-d135736722d2" />

Then, we will enter "load poly" in our command prompt

<img width="1207" height="760" alt="Image" src="https://github.com/user-attachments/assets/cbabfa49-1e20-462a-91e5-e17e24af2c6d" />

We will begin by creating 2 instances of the poly.9 violation and deleting the poly layer, then placing n diffusion and p diffusion respectively along with their coresponding psubtratepdiff and nsubstratendiff taps, for the p diffusion and n tap we will also add an n well to clean up irrelevant erros. This approach ensures that our drc rule does not apply only to poly.

<img width="689" height="357" alt="Image" src="https://github.com/user-attachments/assets/5a48ae73-05e9-47bf-92f7-a73b4a746364" />

#
We will start by fixing the drc spacing error between the poly resistors and poly, and then changing nsd (N-diff) to alldiff in the sky130A.tech file.

<img width="667" height="105" alt="Image" src="https://github.com/user-attachments/assets/7f0b7efd-d945-4e4d-8395-525e203326e9" />


<img width="532" height="77" alt="Image" src="https://github.com/user-attachments/assets/67005706-cd64-4d7b-95dd-df65b1256a86" />

Now the violations appear on our report.

<img width="512" height="113" alt="Image" src="https://github.com/user-attachments/assets/5f3b1f4c-8d69-48aa-8599-2d2985b41691" />

##
# Day 4 — Sky130 Day 4 - Pre-layout timing analysis and importance of good clock tree

About LEF files and guidelines for standard cell ports

Before custom cells can be used inside OpenLane they need proper __LEF__ files describing their physical boundaries, pin locations and metal layer info.
port definitions follow 2 main rules:

- All I/O ports lay on the intersection of horizontal and vertical routing tracks.
- The cell width must be an odd multiple of the horizontal track pitch, and the height must be an odd multiple of the vertical track pitch
 
#
### STA (Static timing analysis) conceps:

- __Setup slack__ = Data required time - Data arrival time (equal or more than 0)

Uncertain variables accounted for in STA:
- __OCV__ (On chip variation) - process/voltage/temperature variation modelled using derate factors
- __Clock Uncertainty__ - Jitter and skew margins added to timing paths
- __CRPR__(Clock Reconvergence Pessimism Removal) - Removes artificial pessimism when launch and capture paths share clock buffers
#
### Clock Tree Synthesis (CTS)

CTS Builds a balanced tree of clock buffers that distribute the clock signal evenlt across the chip with minimal skew.

Rules after CTS:
- Hold time must be rechecked as CTS inserts buffers that add quantitative delays
- Setup time must be reverified as CTS changes clock paths
#
 # Lab Timing modelling using delay tables


## L1 - Steps to convert grid info to track info 

  <img width="1028" height="696" alt="Image" src="https://github.com/user-attachments/assets/8e7c5ff0-33c9-42ad-9f5f-d12760e34b58" />
  As seen in the picture our I/O ports respect the rules, and now the place and route's alignment can happen along our grid.
  
 #
## L2 - Steps to convert magic layout to std cell LEF 

<img width="1217" height="741" alt="Image" src="https://github.com/user-attachments/assets/295fc4ea-54a3-4eba-b18b-4149f0426bff" />
 
 After defining our pins A,Y,GND,PWR we will save a .mag file of our design called sky130_vsdinv.mag
 Then, we will execute the "lef write" command
 
 ```bash
lef write
```
<img width="941" height="657" alt="Image" src="https://github.com/user-attachments/assets/518ab54c-0f5a-41eb-8ce0-4f327e0dd985" />

Our .lef file is ready

#
## L3 - Introduction to timing libs and steps to include new cell in synthesis

We will begin by adding our inverter's .lef and the slow, fast and typical timing libs to our picorv32a core's src directory, and then modfying our picorv32a's config.tcl file to include the changes

<img width="1193" height="570" alt="Image" src="https://github.com/user-attachments/assets/ad0cc1eb-dabd-48a1-bdfa-bbc9fc170773" />
After loading our picorv32 design into OpenLane we will enter the following commands:

 ```bash
% set lefs [glob $::env(DESIGN_DIR)/src/*lef]
% add_lefs -src $lefs
```

After synthesizing, running floorplan and then placement we are greeted with our succesful layout of the inv cell implemented in our core

<img width="780" height="783" alt="Image" src="https://github.com/user-attachments/assets/023464dd-6e42-4d66-8641-5fcb089f9481" />

#
With a quick search in our layout we can identify one of our new placed cells the sky130_vsdinv

<img width="582" height="379" alt="Image" src="https://github.com/user-attachments/assets/ae38857c-ef3f-4685-af94-ca7c06325600" />
