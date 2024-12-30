# SoC-Physical-Design-RTL2GDSII

## About Me
Hi there! I am Anirban, an aspiring chip designer. I am currently in my junior year at Birla Institute of Technology and Science, Pilani - Hyderabad Campus, pursuing an undergraduate degree in Electrical and Electronics Engineering with a minor degree in Computing and Intelligence. So far I've only undertaken frontend VLSI project work such as Functional Verification of a Lightweight GIFT Cipher and RTL design of a RISC-V processor for Neural Network loads. So I've decided to participate in this winter cohort on Physical Design of a System-on-Chip and gain some hands-on experience in the VLSI backend domain. 

## A Crisp Overview of the Basics
If I sat down to write about all there is to chip designing, Intel would catch up to TSMC before I finish. So I'm only going to touch upon the prerequisites essential to understanding the contents of this project assuming the reader covers their knowledge gaps through the great gift to humanity that is the world wide web.

Imagine you want to achieve a certain functionality (e.g.a video game). You describe it using a high-level-language like C. This code is then converted to an assembly language using a compiler and the assembly language is in turn converted to machine code using an assembler. Now what you might miss here is that this "assembly language" and it's corresponding "machine code" are not unique. They differ based on the Instruction-Set-Architecture (ISA) they are a product of. Different ISAs like ARM, RISC-V, MIPS, x86 etc. have compilers/assemblers specifically designed for them and hence have unique assembly and binary codes. This further entails that these machine codes can only run on a compatible device , i.e., a device whose RTL code is written in accordance with the guidelines of the respective ISA. In this project, we will exclusively be dealing with the RISC-V ISA which is an open-source Instruction-Set-Architecture.

Now, a key aspect to understanding this project is being aware of the VLSI Design Flow.

<img src="https://bhive-design.com/wp-content/uploads/2023/03/Physical-Design-1536x1457.png" alt="Alt Text" width="1000" height="500">

All of the processes depicted in the flowchart above have industry standard tools that come with expensive licences. In this project, we will limit our exploration to Physical Design only using open-source tools. The schematic below highlights how OpenLANE - an open-source digital design flow for ASIC development, can be used to go from RTL to GDSII. 

<img src="https://www.zerotoasiccourse.com/openlane-flow.png" alt="Alt Text" width="1000" height="500">

With the availability of open-source RTL IPs in abundance online, open-source EDA tools in the OpenLANE design flow and a 130nm open-source Process-Design-Kit (PDK) - thanks to the Google x SkyWater Technology Collaboration, we are now fully equipped to carry out physical design at an individual capacity and even have designs that are tape-out ready!

### Link to Valuable Resources
The following list of resources related to this project should be more than sufficient to show you the ropes of open-source ASIC design and exploration:-
1. [OpenCores](https://opencores.org) - If you're looking for RTL IPs to complete your HDL patchwork, this site has an enormous collection of open-source designs to assist you. You might even find what you need right here on GitHub!
2. [OpenLANE Repository by The OpenROAD Project](https://github.com/The-OpenROAD-Project/OpenLane) - If you really want to get into open-source ASIC designing using openlane, this repository is the holy grail! The installation doc in this repo covers everything from navigating the intricacies of operating systems to guided tutorials which will help you get a hang of this thing.
3. [FOSSi Foundation Youtube Playlist](https://www.youtube.com/playlist?list=PLUg3wIOWD8yoZCg9XpFSgEgljx6MSdm9L) - These are some videos I've personally found insightful in my journey so far.

## Taking the leap - RTL2GDSII !
### 0. Getting Started
I am running openlane on **Ubuntu 18.04 LTS (Bionic Beaver)(64-bit)** using **Oracle VirtualBox**. No, I'm not an expert in Linux and neither are you expected to be. I highly recommend you go through the installation doc on the OpenLane Repository linked above. A few hours of reading, downloading, installing, troubleshooting and building familiarity with terminal commands like cd, ls -ltr etc. should get you to the *~/Desktop/work/tools/openlane_working_dir/openlane* directory. Once you reach here, go ahead and type *docker* and then follow it up with *./flow.tcl -interactive*. When the interactive mode is enabled type *package require openlane 0.9* to load the required package.

![Screenshot 2024-12-28 020400](https://github.com/user-attachments/assets/59ea0bbe-78f7-4651-bc1c-841714f2af4f)

We are now set to begin this project ,i.e., **Implementation and Characterization of RISC-V Based Core - PicoRV32**. 

### 1. Synthesis
PicoRV32 is the RTL IP we are working with. It is present in the *designs* folder of the same directory. Feel free to go ahead and explore it's specifications. 

We are now gonna boot it up for synthesis. In this stage of OpenLANE flow, the tools Yosys, ABC and OpenSTA are used to synthesize, map and generate timing reports for the design. All you need to do is type *prep -design picorv32a* followed by *run_synthesis* and wait for the synthesis to successfully conclude. Spring another tab in the terminal and follow the commands shown below:-

![Screenshot 2024-12-28 023040](https://github.com/user-attachments/assets/0acd2409-20b3-411e-a4ad-4ad1970f235a)

Every run generates a time-stamped folder with all the necessary reports and results as shown above. Here's a peek into what you can find inside the *./reports/synthesis/1-yosys_4.stat.rpt* directory. This is a snippet from the statistics report of the PicoRV32 architecture. 

![Screenshot 2024-12-28 024023](https://github.com/user-attachments/assets/9f9fa331-eba0-4d72-b327-01fe4a93f882)


Let us do a small exercise here. We are gonna go ahead and calculate something called the Flop Ratio.

$$
\text{Flop Ratio} = \frac{\text{No. of D Flip Flops}}{\text{Total No. of Cells}} = \frac{1613}{14876} = 0.1084296854
$$

This implies the % of D FFs in this design is 10.84%. This number is on the lower side which means the logic is more combinationally dominated than sequentially. This is aligned with PicoRV32's goal of being a "Size-Optimized" CPU, trading-off latency for area.

Feel free to go ahead and look into the STA reports that have been generated as well. You can access the synthesized gate-level netlist in the *results* folder of the time-stamped directory.

### 2. Floorplanning
Any Physical Design Engineer should be familiar with some very basic terms encountered during this stage:-

$$
\text{Utilization Factor} = \frac{\text{Area Occupied by a Netlist}}{\text{Total Area of the Core}}
$$

Core here refers to the area on the die available for placing logic.

$$
\text{Aspect Ratio} = \frac{\text{Height of core}}{\text{Width of Core}}
$$

Similarly learning about de-coupling capacitors and noise margin is recommended at this point.

In floorplanning, the IPs that have user-defined locations are arranged on the chip before automated placement and routing is carried out. So these are also called pre-placed cells. Before we move into the practical end of things, please also look into Power-Planning, the need for Power-Meshes. Build a familiarity with terms like Ground Bounce, Voltage Drop and Logical Cell Placement Blockage during Pin Placement.

On your terminal, head over to *~/Desktop/work/tools/openlane_working_dir/openlane/configuration/* and open the *README* file:-

![Screenshot 2024-12-29 003415](https://github.com/user-attachments/assets/8fea1b35-2d27-4039-94aa-6303753b0c8f)
![Screenshot 2024-12-29 003452](https://github.com/user-attachments/assets/4ddb7b70-10e2-4339-88c2-7c1ccf3e4399)

The values of all these switches can be set in the tcl files available in the configuration folder. For example, the utilization factor has been set to 50 here as a system default. This is a generic setting and can be overridden by a more specific direction. Now head over to *~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/* and open the *config.tcl* file there. You will find a more specific setting for the utilization factor there of 35. This takes precedence as we will see later. Note that if there is a more specific config specification in *sky130A_sky130_fd_sc_hd_config.tcl* file, then that will take precedence as per the heirarchy.

![Screenshot 2024-12-29 003846](https://github.com/user-attachments/assets/90621619-7651-4e21-b171-d2d3627eefb7)

Let us now go and use the floorplanning provisions of OpenROAD. In the interactive mode tab of your terminal, execute the *run_floorplan* command:-

![Screenshot 2024-12-29 004121](https://github.com/user-attachments/assets/b11cbcb0-4129-447e-83dd-d8a2639d47dd)
![Screenshot 2024-12-29 004415](https://github.com/user-attachments/assets/6f444503-9ea9-41a7-a61b-e790be667173)

After successfully running the floorplan, we can explore the results at *~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/*. Open up the latest run.

![Screenshot 2024-12-29 010457](https://github.com/user-attachments/assets/28b64b09-e6b2-439f-bda0-d1d3715dc3f8)

In this directory, you want to head over to *logs/floorplan/* and open your *ioplacer.log* :-

![Screenshot 2024-12-29 005303](https://github.com/user-attachments/assets/765e2098-bb75-4dd6-a0a2-d220d0038faf)

Some relevant details like core area, die area, no. of layers etc. can be found here among other things. Now in the latest run directory, you can open up the *config.tcl* file and as predicted earlier the utlization factor of 35 has prevailed.

![Screenshot 2024-12-29 010214](https://github.com/user-attachments/assets/15b9b68e-d34e-4971-9eb7-4470ac322397)

Furthering our exploration, let us head into *results/floorplan/* of the latest run directory and open up the *picorv32a.floorplan.def* file.

![Screenshot 2024-12-29 010623](https://github.com/user-attachments/assets/c7c3138c-988e-4b08-9163-86e1b0072aef)

The coordinates of the lower left corner of our die and upper right corner of our die are provided as (0,0) and (660685,671405). These values are in database units. The unit distance in database units is defined above as 1000 database units equal 1 micron. Therefore the actual floorplanning has been done on a die area whose corresponding coordinates are given by 0.0 0.0 660.685 671.405 (microns).

