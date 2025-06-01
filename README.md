# SoC-Physical-Design-RTL2GDSII

## About Me
Hi there! I am Anirban, an aspiring chip designer. I am currently in my junior year at Birla Institute of Technology and Science, Pilani - Hyderabad Campus, pursuing an undergraduate degree in Electrical and Electronics Engineering with a minor degree in Computing and Intelligence. So far I've only undertaken frontend VLSI project work such as Functional Verification of a Lightweight GIFT Cipher and RTL design of a RISC-V processor for Neural Network loads. So I've decided to participate in this winter cohort on Physical Design of a System-on-Chip and gain some hands-on experience in the VLSI backend domain. 

## A Crisp Overview of the Basics
If I sat down to write about all there is to chip designing, foundries would catch up to TSMC before I finish. So I'm only going to touch upon the prerequisites essential to understanding the contents of this project assuming the reader covers their knowledge gaps through the great gift to humanity that is the world wide web.

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

In floorplanning, the IPs that have user-defined locations are arranged on the chip before automated placement and routing is carried out. So these are also called pre-placed cells. Before we move into the practical end of things, please also look into Power-Planning, the need for Power-Meshes. Build a familiarity with terms like Ground Bounce, Voltage Drop and Logical Cell Placement Blockage during Pin Placement. Ideally Power Distribution Network (PDN) Generation should happen during the floorplanning stage. But we shall be taking it up later in the flow.

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

We will now have a look at the generated floorplan using Magic (the open-source EDA layout tool). So in the *~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/<latest_run>/results/floorplan/* directory go ahead and type the following command *magic -T /home/username/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &* . This will open the design exchange file in magic. You will have two open windows - a layout window and a tkcon window.

![Screenshot 2024-12-29 020653](https://github.com/user-attachments/assets/e30582ec-5c1d-4ea4-8b60-d9c89ef5f32a)

You can use the S key to select your layout and the V key to center it on the screen. In order to zoom in on a specific area in your layout, a left mouse click on the bottom left followed by a right mouse click on the top right will create a window which you can zoom into by pressing the Z key.

![Screenshot 2024-12-29 020842](https://github.com/user-attachments/assets/071eba41-96a5-426a-96f8-cb8ff4693174)

The blue pins along the edges are the io pins of the design. Hover over one and use the S key to select it. Head over to the tkcon window and type *what* to learn more about the component you've selected.

![Screenshot 2024-12-29 022040](https://github.com/user-attachments/assets/edde283c-8802-428c-96e6-98e23f004f38)
![Screenshot 2024-12-29 022447](https://github.com/user-attachments/assets/e0104e85-b3e3-4502-b45d-64d89a738602)

The floorplan has the io pins, de-coupling capacitors, tap cells etc. arranged but it does not have the standard cells placed yet. You will find them clumped together in the bottom left corner of your layout as shown below.

![Screenshot 2024-12-29 022928](https://github.com/user-attachments/assets/335d442a-818d-43db-8b6e-ef5f05573673)

### 3. Placement
When I say netlist, I assume you're imagining all these different logic-gates linked with each other as per your design specifications. In this imagination of yours, an OR gate probably looks like a guitar pick and a not gate like a nacho. I hate to break it to you but, in the physical world, these logic gates actually look quite different with well defined shapes and sizes that are well documented in standard cell libraries. In placement, we will be placing these standard cells onto the bulk of the core.  

We will be focusing on reducing congestion right now and not on timing aspects of the design. Placement happens in two steps - first the global placement followed by the detailed placement. Global Placement is a rough or approximate placement of standard cells that doesn't necessarily abide by legalization rules. The main objective here is to reduce the wire length also called *Half Parameter Wire Length (HPWL)*. We need the *Overflow (OVFL)* here to converge. In Detailed Placement, legality checks are enforced till success with the standard cells being placed in fixed standard cell rows ensuring there is no overlap.

We will begin the process by executing *run_placement* in the interactive terminal.

![Screenshot 2024-12-30 205558](https://github.com/user-attachments/assets/9c31d05e-4c7e-4f79-929b-3013ae3bc5b8)
![Screenshot 2024-12-30 210056](https://github.com/user-attachments/assets/f9c66920-800b-49e4-90c7-8fb784741500)
![Screenshot 2024-12-30 210128](https://github.com/user-attachments/assets/21517e83-2514-4f90-978f-134f269f3847)

After the run successfully concludes, head over to magic and get a glimpse of the updated layout. You want to follow the instructions as shown in the image below:-

![Screenshot 2024-12-30 210153](https://github.com/user-attachments/assets/63d1e28c-55d2-404b-8870-235471c5d38b)

The updated layout is shown below:-

![Screenshot 2025-01-03 171012](https://github.com/user-attachments/assets/58a52d5d-ddc8-4de1-90d7-464f37c18168)

You can zoom into the design and see how the standard cells have been placed in the standard cell rows.

![Screenshot 2025-01-03 172604](https://github.com/user-attachments/assets/34c635d8-b47a-45b9-86fc-c78ee00354dc)

Before we move to further steps in the flow such as Routing or CTS, I would like to cover a couple of things related to standard cell design. Every standard cell, even something as simple as an inverter, has to pass through a cell design flow as shown below:-

![Screenshot 2024-12-31 183728](https://github.com/user-attachments/assets/27f1cc83-5d2f-438c-a130-63e0a175bc3f)

In this flow the characterization step can be further broken down into:-
  1. Reading the model file
  2. Reading the extracted spice netlist
  3. Recognizing the behaviour of buffer
  4. Reading the sub-circuit of the inverter
  5. Reading in the necessary power supply
  6. Applying the stimulus
  7. Providing necessary output load capacitances which need to be varied
  8. Providing necessary simulation commands

Once all this is done, GUNa software gives us a model with the timing, noise and power characterizations. 

I would recommend learning the meanings of the following threshold parameters relevant to timing characterization. This will be helpful in a small exercise we will do further down:-

![Screenshot 2025-01-03 174328](https://github.com/user-attachments/assets/c8f6f214-32af-4d85-becd-15c4927bb938)

Also *Propagation Delay = time(out_*_thr) - time(in_*_thr)*. When we get a negative delay, it is a cause for concern. This mainly happens due to improper threshold selection or improper circuit design.

In our exploration of the standard inverter cell, we will not be building one from scratch. Rather we will clone a custom inverter standard cell design from the following github repository: [Standard cell design and characterization using openlane flow](https://github.com/nickson-jose/vsdstdcelldesign.git)

We will perform post-layout simulation of this inverter on ngspice and plug this cell into our picoRV32 core post characterization. A working knowledge of Voltage-Transfer Characteristics (VTC), switching thresholds etc. is recommended for this section.

In order to clone the repo above, click on code and copy the https link. In a fresh terminal, open *~/Desktop/work/tools/openlane_working_dir/openlane* directory and type *git clone (paste the copied link here)* as shown below:-

![Screenshot 2025-01-03 205609](https://github.com/user-attachments/assets/7da56d46-b94f-47e2-a489-10d040769e28)

After running the command when you check the directory a *vsdstdcelldesign* folder would have been created with a *sky130_inv.mag* file present. Before we open this file we will need to copy a technology file. You can do so as shown below:-

![Screenshot 2025-01-03 230203](https://github.com/user-attachments/assets/625cbdc1-cbbc-45ba-a6ad-0b14f56853ed)

As you can see below, the tech file is now in our required directory and we open this file in magic using the command shown below:-

![Screenshot 2025-01-03 230358](https://github.com/user-attachments/assets/bd13f213-97b1-4f55-9a61-e95bb67b7de8)

I'd like to mention that the 2D schematic shown below is actually a multi-layered 3D structure constructed using a 16 - mask CMOS process. Feel free to look into it for a better understanding of all the layers we will be working with in magic.

![Screenshot 2025-01-03 230649](https://github.com/user-attachments/assets/8511b9e4-2f1a-4ea5-a32e-4df0ccd6d1f6)

You can see a colour-texture palette on the right. Hovering over the icons on this palette, you'll be shown the functionality on the top bar e.g. local interconnects, metal contacts, metal layers etc.
On the layout window, you can select different elements and check what they are by typing *what* in the console window.

Now on the top bar you can see a DRC checkbox. Whenever there is a Design Rule Violation, this box would be red with the number of violations indicated. On your layout window, a violation is shown using an area filled with white dotted lines. There is also a DRC tab to assist you in fixing these violations. You can use the middle mouse button to apply something from the palette on the selected area.

Now to explore the logical functioning of inverter, we will extract spice netlist and do simulations using it on ngspice tool.

First, on your console window, follow the steps shown below:-

![Screenshot 2025-01-04 011739](https://github.com/user-attachments/assets/f06cdab1-a84d-4444-b9a4-4d79f9f92640)

These files will now be available in the directory shown here:-

![Screenshot 2025-01-04 013415](https://github.com/user-attachments/assets/441ffa4b-7481-4e55-8f86-0022043f9db2)

The vim command will open the netlist file where you will see this:-

![Screenshot 2025-01-04 014121](https://github.com/user-attachments/assets/8ee04955-13c1-4c68-846c-d7d202dc34ef)

We will alter the specifications as follows to meet our requirements:-

![Screenshot 2025-01-04 025504](https://github.com/user-attachments/assets/7bc60616-b3db-40f1-b360-e0ecd5dc26de)

We have changed the technology scale to match 0.01 microns, included relevant pmos and nmos libraries, created supply and ground voltage provisions, specified pulse and given the details for transient analysis. You can make these changes by pressing *i* to enter insert mode and then save and quit by pressing *esc* followed by *:wq!*. Following this, on the terminal, simulate the netlist on ngspice as shown. If ngspice is not installed already, you would have to install it using *sudo apt install ngspice*.

![Screenshot 2025-01-04 025401](https://github.com/user-attachments/assets/deb67603-eacf-46fe-b314-d5d7c16a93c6)

In the ngspice prompt window, enter *plot y vs time a*. A transient analysis of the pulse appears as shown below:-

![Screenshot 2025-01-04 030125](https://github.com/user-attachments/assets/eb5f8adf-27b1-42aa-bd45-be2c2ef41904)

It is now finally time to characterize the cell. In order to characterize a standard cell we need to calculate 4 parameters:-
  1. Rise Transition - Time taken by output waveform to go from 20% to 80%
  2. Fall Transition - Time taken by output waveform to go from 80% to 20%
  3. Rise Cell Delay - Time taken by output to reach 50% of high voltage after rising input crosses 50%
  4. Fall Cell Delay - Time taken by output to reach 50% of low voltage after falling input crosses 50%

![Screenshot 2025-01-04 031344](https://github.com/user-attachments/assets/42e055c5-1c49-4d82-b3e8-a3167ee2cf37)
![Screenshot 2025-01-04 031545](https://github.com/user-attachments/assets/909f413e-c967-404d-9a17-d42e7d830930)
![Screenshot 2025-01-04 032141](https://github.com/user-attachments/assets/5b1d604d-acc5-4bb7-af1c-6a071da56234)
![Screenshot 2025-01-04 032351](https://github.com/user-attachments/assets/f23b7d92-5f60-493a-b3f2-87ab18c76b19)
![Screenshot 2025-01-04 032503](https://github.com/user-attachments/assets/729574f4-6238-437a-924b-2d08bc8f6093)

From the graphs shown above, the required coordinates were determined in order:-

![Screenshot 2025-01-04 033225](https://github.com/user-attachments/assets/7d372f84-ee05-4bf9-be32-9ae82068e73e)

*Rise Transition Time = 2.24624ns - 2.18214ns = 0.0641ns = 64.1ps*

*Fall Transition Time = 4.09522ns - 4.05235ns = 0.04287ns = 42.87ps*

*Rise Cell Delay = 2.21109ns - 2.14989ns = 0.0612ns = 61.2ps*

*Fall Cell Delay = 4.07771ns - 4.05ns = 0.02771ns = 27.71ps*

With the characterization of our inverter now complete, we will go ahead and create a LEF file for it. But before that we'll take a small detour and explore DRC in magic.

Check out the following links which can be useful in learning more about Design-Rule-Checks in Magic:-
  1. [opencircuitdesign.com/magic/](opencircuitdesign.com/magic/) - You would be interested to go to the *Technology Files* section and open the *Magic Technology File Manual*. Look into tutorial 2 and 6 specifically.
  2. [The Skywater Build Docs](https://skywater-pdk.readthedocs.io/en/main/) - Look into the *Design Rules* specifically.

In a terminal tab, open your home directory. There type out the commands as shown below:-

![Screenshot 2025-01-04 180049](https://github.com/user-attachments/assets/4d0adc04-aaa2-41dc-8ae4-e83dafc07431)

You can do *vi .magicrc* to look into the startup file for magic. Our experiment here is self contained so we have the tech file mentioned as well. However it is not a recommended practice. You will now get access to the magic files of a bunch of masks and layers as shown above. Type *magic -d XR* to open the layout and console window of magic. We will go ahead and open the met3.mag file in the layout window.

In this file we can tinker around and explore various things as shown below:-

![Screenshot 2025-01-04 182440](https://github.com/user-attachments/assets/7d5a88d6-1351-434c-ab6d-ae26f20284ea)

You can see the reasons for violations on screen by pressing *:drc why*, you can select areas and apply textures from the right. I have applied the pattern shown in the bottom left using *:cif see VIA2*.

![Screenshot 2025-01-04 182957](https://github.com/user-attachments/assets/806772e7-e756-44e7-827a-7b6184798345)

Our tech file has commands to tell magic how to draw contact cuts inside the contact area. We will now go ahead and do a small exercise where we fix errors in a file.

![Screenshot 2025-01-04 183257](https://github.com/user-attachments/assets/a3739668-2032-4225-8781-d0472aa7c548)

E.g. As you can see, poly.9 is incorrect. You can find out the design specifications on the website given in link 2 as shown below. The required rule distance is 0.48 microns. Go ahead and fix this violation.

![Screenshot 2025-01-04 183844](https://github.com/user-attachments/assets/e9194b0c-1f81-4765-baaf-d845fdde746a)

I shall fix a violation in met1.mag file as a part of my exercise. As can be seen, m1.6 is violating a drc. On checking the website I learnt the area of this metal should be atleast 0.083 microns squared.

![Screenshot 2025-01-04 185217](https://github.com/user-attachments/assets/0354d3de-e562-4c40-a037-7f2b4600d306)

I now fix this DRC by increasing the area to 0.888 microns bringing the DRC violations down from 12 to 10.

![Screenshot 2025-01-04 185714](https://github.com/user-attachments/assets/1602cf9e-27c9-42eb-8455-2cac824679f1)

Now, for place and route we don't really need all the detailed information inside a standard cell. Very basic information about the cells boundaries, power and ground rails etc. suffices and this is available in a LEF file. We will now extract the LEF file for the inverter and plug it into the picoRV32 design.

There are some pre-requisites to creating a LEF file. Make sure you have these specifications met, before you proceed:-
  1. The ports in your standard cell should be at the intersection of horizantal and vertical grid lines.
  2. Width of the standard cell should be an odd multiple of the horizantal pitch.
  3. Height of the standard cell should be an odd multiple of the vertical pitch.
  4. The ports of your standard cell need to be properly specified as they will translate to pins on the LEF file.

Information regarding the pitch can be found in the *sky130_fd_sc_hd* file located deep in the pdks directory in a file called *tracks.info*.

We will save our inverter in the *vsdstdcelldesign* folder with a personalized name *sky130_anirbaninv.mag* using the console window . It is this file that will give us the LEF. Open this mag file, head over to the console window and just type *lef write*. Return to the terminal and check your directory. The lef file would have been created with the same name. On opening it will look something like this:-

![Screenshot 2025-01-06 002423](https://github.com/user-attachments/assets/a0ed86d5-e2a3-4736-a9fd-ec45d5a1db9d)

Follow the steps shown below to include the custom lef file in the source folder of your design directory:-

![Screenshot 2025-01-06 224344](https://github.com/user-attachments/assets/8a3090f5-f61d-46cb-9b45-a0cbee04d523)

You can open and see the contents of some library files as shown below:-

![Screenshot 2025-01-06 213809](https://github.com/user-attachments/assets/c6ddf22c-ab33-41cb-a1a7-6c77d9a771ff)

You need to copy these library files into your design source directory as well as shown earlier. You will be able to see all the copied files when you check at different stages as shown below.

![Screenshot 2025-01-06 225043](https://github.com/user-attachments/assets/351a0e32-a0c3-4332-ad9f-d370aedd46f0)

Go ahead and do a *vim config.tcl*. This is what should be visible:-

![Screenshot 2025-01-06 214317](https://github.com/user-attachments/assets/0c0e1109-ff0d-4cc4-b283-aae7784eb784)

Alter it to this:-

![Screenshot 2025-01-06 222440](https://github.com/user-attachments/assets/f11b75b9-bf7a-45ef-a84d-89ff064aac82)

Now we will go ahead and enter interactive mode. Follow the steps exactly as shown below:-

![Screenshot 2025-01-06 222339](https://github.com/user-attachments/assets/01afb7f1-dcee-4b9a-a00b-eaa4bd1bbf58)
![Screenshot 2025-01-06 222416](https://github.com/user-attachments/assets/8925b6b7-e1a8-4b52-9835-c0bd69d6ba67)
![Screenshot 2025-01-06 223019](https://github.com/user-attachments/assets/ce8d3c5c-9537-4cb1-989b-5d0daa3dc2d0)
![Screenshot 2025-01-06 223817](https://github.com/user-attachments/assets/78a37dfe-d254-43f4-9656-a9f1d077c3bd)

At the end you can see two values mentioned. These are the total negative slack and worst slack being reported by OpenSTA tool. These are NOT favourable so we will go ahead and fix it. Head over to *configurations* in the *openlane* folder and open the *README.md* file. You will be able to see the parameters below that can be tweaked as per our requirements for synthesis.

![Screenshot 2025-01-07 000319](https://github.com/user-attachments/assets/f922d633-3282-4536-9a71-3af8f9ba47a9)

We will try making a more timing optimized design employing our custom inverter at the cost of a bit more area by making the following changes:-

![Screenshot 2025-01-07 001646](https://github.com/user-attachments/assets/8129113c-0714-4663-ab6d-cce126658d05)

Now when we re-run synthesis:- 

![Screenshot 2025-01-07 003037](https://github.com/user-attachments/assets/f7edc816-ac76-4e5e-aee9-813bf2645191)

VOILA! Negative slack is removed. Now we go ahead and run the floorplan:-

![Screenshot 2025-01-07 004806](https://github.com/user-attachments/assets/a3aadb61-66d8-4cb8-b3fd-1ca7057f24c6)

And then we run the placement. Open the newly generated .mag layout file for placement in Magic and this is what you'll see:-

![Screenshot 2025-01-07 005729](https://github.com/user-attachments/assets/3ce3206b-7d3d-4d5f-8832-06b087c8d064)

Let us now go and find our custom inverter:-

![Screenshot 2025-01-07 024552](https://github.com/user-attachments/assets/9eeaad9a-3036-4001-83a0-c022a69df637)

When you find the needle in the haystack, it should look something like this. Congratulations, your custom inverter is now actually in the layout!

### 4. Clock Tree Synthesis
Before we delve deep into this section, there are some basic Static Timing Analysis terminology you should be familiar with. 
Setup time *(Tsetup)* is the minimum time before the active clock edge during which the data signal must be stable to ensure proper capture by the flip-flop. Hold time *(Thold)*, on the other hand, is the minimum time after the clock edge during which the data signal must remain stable. These constraints ensure correct data transfer between the launch and capture flip-flops. The data arrival time *(Tarrival)* represents when the data reaches the capture flop, while the data required time *(Trequired)* defines when the data must be valid to avoid timing violations. Setup and hold checks are performed to validate these conditions:-

  1. Setup Check: *Tarrival < Trequired* where *Tarrival = TclktoQ + Tpropagation + Tdatapathdelay* and *Trequired = Tclkperiod - Tsetup*
  2. Setup Slack: *Ssetup = Trequired - Tarrival*
  3. Hold Check: *Tarrival > Trequired* where *Tarrival = TclktoQ + Tpropagataion + Tdatapathdelay* and *Trequired = Thold*
  4. Hold Slack: *Shold = Tarrival - Trequired*

Slew refers to the rate at which a signal transitions and affects timing margins by influencing *Tsetup* and *Thold*. Other critical factors include *clock skew*, *clock jitter*, and *cell/wire delays*, which must be considered in propagation delay calculations to ensure robust timing analysis. Violations of either setup or hold timing constraints lead to timing violations, potentially impacting circuit functionality and reliability. Lastly you might also want to look into Delay Tables. These tables are made to deal with the varying transistions in cells of different sizes and natures. They are based on two important principles of having each node driving the same load at every level and having identical cell at same level. They are created for every cell.

We will now set up two files which are required for STA. The tool we use for STA in open-source physical design is *OpenSTA*. The first file is the *my_base.sdc* file that you need to create in the src folder of your design directory. The contents of it are shown below:-

![Screenshot 2025-01-07 160039](https://github.com/user-attachments/assets/fe4a9e4b-4ebb-4d75-88eb-4500d82a7e1a)

The second file is the pre_sta.conf file that you need to create and configure in the openlane folder as shown below:-

![Screenshot 2025-01-07 160539](https://github.com/user-attachments/assets/70bb858f-be8c-467d-a79a-0e8418e5c902)

Once you've created and saved both the files, we will perform STA on it as shown below:-

![Screenshot 2025-01-07 160829](https://github.com/user-attachments/assets/e14808dc-d823-408a-a76e-081efd4156ea)
![Screenshot 2025-01-07 160847](https://github.com/user-attachments/assets/79a00d44-5325-4753-a64e-f5d65086598b)

We will now set the max fanout to 4 and run synthesis again as shown below:-

![Screenshot 2025-01-07 161724](https://github.com/user-attachments/assets/7308c2ea-6da4-4b1f-9bac-86949562bc0f)
![Screenshot 2025-01-07 163436](https://github.com/user-attachments/assets/911303ca-ffba-413c-aa2a-c484559a9b13)

When the synthesis concludes and you run sta again you will see these new values:-

![Screenshot 2025-01-07 163207](https://github.com/user-attachments/assets/bf933902-0f37-409c-b39e-65d2f0e38489)

We will now try to reduce the violations by replacing cells:-

![Screenshot 2025-01-07 164506](https://github.com/user-attachments/assets/dfcdafc9-ec3a-434b-9135-d284abe5ac12)
![Screenshot 2025-01-07 164520](https://github.com/user-attachments/assets/2487585b-f2ab-4043-9e77-5382076233d0)
![Screenshot 2025-01-07 164657](https://github.com/user-attachments/assets/1f7aa8e7-22e2-4c5b-9389-4dd389289b01)
![Screenshot 2025-01-07 164709](https://github.com/user-attachments/assets/98ce8148-14a9-4118-add5-1c2b3217f90f)

Slack has been reduced to -1.7987 which is great. But we are still in violation. We will fix this after CTS when we replace ideal clock with actual one. Update the verilog file with these new settings that we want for minimal slack. As you can see below, the latest timestamp indicates the netlist has been updated with our prefered settings.

![Screenshot 2025-01-07 171805](https://github.com/user-attachments/assets/b481b583-df23-4843-8f9d-e4c285a90deb)
![Screenshot 2025-01-07 171826](https://github.com/user-attachments/assets/dbe59bd7-0b49-4e67-bbed-b1b4414be353)

Now run the floorplan and placement again. Once you have done that, we are ready to begin CTS.

Clock Tree Synthesis (CTS) is a crucial step in digital circuit design, ensuring the clock signal is distributed efficiently and uniformly across a chip. The primary goal is to minimize clock skew and delay, delivering the clock signal to all sequential elements with consistent timing. One popular approach to achieve this is using an H-Tree structure, which distributes the clock symmetrically by splitting the signal at each level. This symmetry helps ensure equal path lengths to all endpoints, naturally reducing skew. To further enhance balance in the clock network, the Midpoint Strategy is often employed. This involves selecting midpoints for buffer insertion to evenly distribute delays and maintain consistent timing across the chip.

Repeaters, also known as buffers, play a vital role in maintaining the integrity of the clock signal as it traverses long distances. By amplifying the signal and sharpening transitions, repeaters counteract the natural degradation that occurs in the clock net. However, the dense and intricate nature of modern chip layouts makes cross-talk a significant challenge. To address this, clock net shielding is employed, placing grounded or low-noise lines alongside the clock signal to prevent electromagnetic interference. This shielding is essential in preserving signal quality and ensuring that noise from adjacent nets does not distort the clock signal.

Another key challenge in CTS is managing glitches, which can arise from timing mismatches, cross-talk, or parasitic effects. Glitches disrupt circuit operation and can lead to serious timing errors. Effective CTS design mitigates these risks by optimizing buffer placement, balancing wire lengths, and applying robust shielding techniques. 

These are the parameters used in CTS:-

![Screenshot 2025-01-07 172553](https://github.com/user-attachments/assets/1b5eb93d-7015-4806-b83e-4a20da411094)

Go ahead and do *run_cts* and you'll see *TritonCTS* run:-

![Screenshot 2025-01-07 173136](https://github.com/user-attachments/assets/99d4fb3c-07b2-4ce3-81cb-032521746586)
![Screenshot 2025-01-07 173315](https://github.com/user-attachments/assets/62c87b5b-c66f-4428-bd0b-857e611b8ae4)

You can see above how a new netlist gets added as well post CTS. We will now perform post-CTS analysis inside OpenROAD:-

![Screenshot 2025-01-07 180233](https://github.com/user-attachments/assets/dc164774-1eb4-409b-9cc8-1122a3a4e913)

Follow the commands exactly as shown above starting with the *openroad* command. When you get the report you will see the *Hold Slack* and *Setup Slack* are still in violation:-

![Screenshot 2025-01-07 180749](https://github.com/user-attachments/assets/041cdf0c-1b33-4501-b3e5-9ecbea1198e7)
![Screenshot 2025-01-07 180808](https://github.com/user-attachments/assets/65dac37d-82c5-4022-bee2-f9bf23fb1258)

Follow the steps below to fix this:-

![Screenshot 2025-01-07 181658](https://github.com/user-attachments/assets/1751fcfc-66f4-42a6-ac0e-8d3b4ff873e2)

Once the report runs you should see that now the slack constraints have been met!

![Screenshot 2025-01-07 181811](https://github.com/user-attachments/assets/426084c0-fe2b-48b1-8922-bbd75d3a802d)
![Screenshot 2025-01-07 181836](https://github.com/user-attachments/assets/9282265e-2c05-418e-82a0-189b0c11085e)

### 5. Routing
Routing in physical design is the process of connecting various components and pins on a chip using metal wires while adhering to design rules and optimizing for performance metrics such as delay, power, and area. A fundamental goal in routing is to find the shortest path between pins with the minimum number of bends, as this minimizes wire length and signal degradation while reducing manufacturing complexity.

One of the earliest algorithms used for routing is the *Maze Routing Algorithm*, commonly referred to as *Lee's Algorithm*. It is a breadth-first search method that guarantees finding the shortest path between two points in a grid, provided such a path exists. Lee’s Algorithm works by expanding a search wave from the source, marking each grid cell with its distance from the source until the destination is reached. After the destination is found, the algorithm backtracks to construct the path. While it is effective for finding optimal paths, it has significant drawbacks in terms of time and memory efficiency. The exhaustive grid search can become computationally expensive, especially for large designs with complex constraints, making it impractical for modern large-scale routing tasks.

To address these inefficiencies, alternative algorithms have been developed. The *Line Search Algorithm* improves efficiency by focusing on line-based explorations rather than expanding uniformly across a grid, which reduces the search space. Similarly, the *Steiner Tree Algorithm* constructs a minimal interconnection tree by introducing additional nodes, known as Steiner points, to reduce the total wire length. These methods are not only faster but also better suited for complex routing scenarios, where performance and resource constraints are critical. Modern physical design tools often incorporate these advanced algorithms or hybrids of them to achieve optimal routing solutions efficiently.

The variables below are available to us for configuration in this last stage:-

![Screenshot 2025-01-07 190548](https://github.com/user-attachments/assets/b2304b00-83e8-4ac9-bb17-43c9daaeefd0)

To begin the process go ahead and follow the steps shown below:-

![Screenshot 2025-01-07 185721](https://github.com/user-attachments/assets/a9f52f4b-0e43-496a-b9e5-ce6d02b2dfc8)
![Screenshot 2025-01-07 190747](https://github.com/user-attachments/assets/fcd69682-bff8-4c97-b74b-cbb8b3654eeb)

The routing can take a while. In the meantime, you can look into TritonRoute and how it handles the incredibly complex routing process. Once your routing concludes successfuly have a look at the last optimization iteration. It will tell you if there are any pending violations to be fixed. Those you'd have to do manually followed by SPEC extraction. Luckily for me the process has concluded with 0 violations as shown below:-

![Screenshot 2025-01-07 202342](https://github.com/user-attachments/assets/227d54bc-b868-4168-a250-2e0fd19b14f1)

Let's go have a look at our finished product!

![Screenshot 2025-01-07 204338](https://github.com/user-attachments/assets/445804cc-dd61-4e85-b9ad-fd108de93a36)
![Screenshot 2025-01-07 204430](https://github.com/user-attachments/assets/12cfb786-f234-4415-97ab-f6b6d2b5909c)
![Screenshot 2025-01-07 203811](https://github.com/user-attachments/assets/7b0428c3-e06b-4600-a31d-b7694f0f0bd3)

## Certificate
![17_VSD_nasscom Workshop Certificate 2025GitHub Repo (1)-1](https://github.com/user-attachments/assets/c0c3e535-f21c-492e-86b6-cbef170fe0f4)
## Acknowledgements
* [Kunal Ghosh](https://github.com/kunalg123), Co-founder, VSD Corp. Pvt. Ltd.
* [Nickson P Jose](https://github.com/nickson-jose), Physical Design Engineer, Intel Corporation.
* [R. Timothy Edwards](https://github.com/RTimothyEdwards), Senior Vice President of Analog and Design, efabless Corporation.
