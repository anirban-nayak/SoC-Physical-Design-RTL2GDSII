# SoC-Physical-Design-RTL2GDSII

## About Me
Hi there! I am Anirban, an aspiring chip designer. I am currently in my junior year at Birla Institute of Technology and Science, Pilani - Hyderabad Campus, pursuing an undergraduate degree in Electrical and Electronics Engineering with a minor degree in Computing and Intelligence. So far I've only undertaken frontend VLSI project work such as Functional Verification of a Lightweight GIFT Cipher and RTL design of a RISC-V processor for Neural Network loads. So I've decided to participate in this winter cohort on Physical Design of a System-on-Chip and gain some hands-on experience in the VLSI backend domain. 

## A Crisp Overview of the Basics
If I sat down to write about all there is to chip designing, Intel would catch up to NVIDIA before I finish. So I'm only going to touch upon the prerequisites essential to understanding the contents of this project assuming the reader covers their knowledge gaps through the great gift to humanity that is the world wide web.

Imagine you want to achieve a certain functionality (e.g.a video game). You describe it using a high-level-language like C. This code is then converted to an assembly language using a compiler and the assembly language is in turn converted to machine code using an assembler. Now what you might miss here is that this "assembly language" and it's corresponding "machine code" are not unique. They differ based on the Instruction-Set-Architecture (ISA) they are a product of. Different ISAs like ARM, RISC-V, MIPS, x86 etc. have compilers/assemblers specifically designed for them and hence have unique assembly and binary codes. This further entails that these machine codes can only run on a compatible device , i.e., a device whose RTL code is written in accordance with the guidelines of the respective ISA. In this project, we will exclusively be dealing with the RISC-V ISA which is an open-source Instruction-Set-Architecture.

Now, a key aspect to understanding this project is being aware of the VLSI Design Flow.

<img src="https://bhive-design.com/wp-content/uploads/2023/03/Physical-Design-1536x1457.png" alt="Alt Text" width="600" height="300">

All of the processes depicted in the flowchart above have industry standard tools that come with expensive licences. In this project, we will limit our exploration to Physical Design only using open-source tools. The schematic below highlights how OpenLANE - an open-source digital design flow for ASIC development, can be used to go from RTL to GDSII. 

<img src="https://www.zerotoasiccourse.com/openlane-flow.png" alt="Alt Text" width="600" height="300">
