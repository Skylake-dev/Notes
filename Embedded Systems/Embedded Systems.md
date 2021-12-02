# Embedded Systems

## Introduction
### Overview on technology
Current CMOS technology is reaching its physical limits:
- cannot increase frequency of the single cores
- "dark silicon" not all the areas of a piece of silicon can be used simoultaneously due to heat generated that cannot be dissipated
- limit to how low the PSU voltage can get
- limit to how small transistor can be

Packing more transistor in the same package increase the power density raising thermal issues. This impacts the reliability of the system since it cannot operate at high temperatures for long periods.  
Managing the thermals is becoming more and more complex:
- dynamic voltage and frequency scaling
- better and more spread out sensors to monitor hot spots
- liquid cooling, two phase cooling for datacenters
- air cooling not always is applicable due to noise

### Platform based design
Design is becoming increasingly more complex and takes more time. Entering late in the market leads to great losses in potential revenue. Adding more people to the project is not always beneficial because in each project there is a critical number of people after which productivity starts to fall.  
This is why there is the need to design reusable platforms that are suited to develop other devices to speed up the design process.

## Microprocessors
Microprocessors are present in embedded systems (ES) to implement in sw the algorithms:
- more flexible
  - maintainability
  - faster and less complex to develop
  - verification is less critical
  - sw designers are more available
- shorter time to market vs designing hw
- lower cost (depends on volume)

However, typically a custom hw solution has a much better performace, not only in terms of computing speed but also in terms of:
- chip area
- power consumption
- security
- memory footprint
- ...

Moreover, designing a sw solution requires a deep knowledge of the microprocessor architecture and the different alternatives that are present on the market both in terms of class of processors and the type required ("form").

Classes of processors
- Application Specific Processor (ASP):  
tailored to specific types of applications (e.g. image processing, audio processing)
- General Purpose Processor (GPP):  
flexible general purpose, can be used in different types of applications

A typical ES use a GPP for supervising and controlling the operation of other components or ASP that do the real computations. The choice is dictated by the nature of the algorithm that needs to run on the system.

Form of processors
- Component Off The Shelf (COTS)  
buy components and mount them on a PCB and the interfaces between the different parts
- Intellectual Property (IP)  
buy the design of the microprocessor, there can be different levels of abstractions:
  - soft-macro: HDL description at an RT level. It's like acquiring the "source code" for the processor, easier to modify but costs more in terms of royalties
  - hard-macro: described to the level of the layout. It's like having a "binary" for the processor, cannot modify

Due to the diffusion of programmable logic devices like FPGAs, the use of IP is becoming more  and more popular.

Performance, power consumption and royalties can make the difference in the final decision.

### Selection process
Different metrics to consider:
- performance
  - IPC instruction per clock, comparison only make sense between the same family of components
  - MIPS absolute measure of the throughput, also in this case it dependes on the ISA
    -  MFLOPS only focus on floating point operation
  - for ASP there can be different metrics depending on the specific functionality that it needs to implement
- power consumption
  - average power, related to the total energy consumed by the system
  - peak power, affects the choice of the cooling solution and psu
  - can be considered in conjunction with the performance, especially at the beginning of the selection process
    - mW/MHz
    - MIPS/mW
- memory
  - size, how much memory do i need?
  - bandwidth
  - sometimes it is not possible to increase it with external memory
  - address space, simple arch still use 16 bit addressing (64kB of memory)
- peripherals
- available software
  - OS and SDK platform
    - community and ecosystem present
    - compilation flow and quality of the resulting code
    - presence of analysis tools, debuggers etc
    - always need to assess their quality
  - availability of libraries to do standard functions
    - simplifies design and validation
    - makes development more practical
- packaging
  - COTS comes in different packages (pinout, material, certification)
- certifications
  - needing particular certifications can constraint the choice of components

### GPP
Can have different architectures
- CISC, many instruction types and addressing modes
  - built to have a compact code due to the memory limitations that there were back in the day
  - ideally addressing mode and operations are orthogonal but in practice this is too complex to achieve
  - instructions are usually of different lengths --> more complex fetch and decode (read expensive and power hungry)
  - complex datapath for the ALU
  - complex dynamic schedule to exploit pipelines and parallelism --> more area more power
  - prevalent arch in desktop and laptops but RISC is starting to gain traction
- RISC, few simple instruction and addressing mode
  - fixed length instruction to exploit pipelining more easily
  - simplify the architecture design
  - increase software size but memory is no longer a pressing issue
  - more instructions but they execute faster
  - typically load/store, data transfers are crucial to performance --> need to design good memory hierarchy
  - inherently lower power than CISC (20-100 mW/MHz)

Both can exploit superscalarity (duplicate execution units to execute instructions in parallel). Of course this improves the performance but require complex scheduling and take up significantly more area --> more cost, increased power consumption.  
CISC instructions (e.g. x86) can be decomposed in RISC instructions and executed on a simpler core. This is useful for backwards compatibility with older programs.

- VLIW, designed to overcome the complexity of scheduling in hw.
All parallel instruction are scheduled statically by the compiler because the arch explicitly support parallelism. This has limitation due to the poor intrinsic parallelism of code (e.g. many data dependencies that force sequential execution). For this reason it is not very common for GPP.

In ES, if there is a GPP usually it exploits a RISC architecture because of their lower power consumption and comparable performance to CISC.

### ASP
- Digital Signal Processing, use very simple operations to perform complex operations like convolutions. Hw is optimized to do this operation in parallel when possible and optimize loops (e.g. increment control variable without using ALU). Characterized by large bus to memory to increase bandwidth. Frequently implemented using a VLIW approach.
- Network processors, optimized for packet managing in routers/switches (e.g. CRC computations, crypto operations)

## Microcontrollers
Embed in a single chip everything that is needed to run the system. Can achieve lower power consumption giving up raw performance. Often they do not offer interface to external memory because typically they run very simple programs and to reduce the pinout. Include serveral peripherals and interfaces (I2C, SPI, JTAG, PWN, UART, watchdog, timer, analog I/O, ... see later lectures).  
These are programmed in C (sometimes assembly) SDK can range from a simple compiler to fully fledge develepment environments with analysis tools.