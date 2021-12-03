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

Main characteristics:
- simple arch (8, 16, 32 bits) with small pipelines
- low clock (8 -160 MHz)
- low cost (0.5 -10 €)
- very low power consumption (0.1 - 300 mW)
- designed to interact with the environments (often include ADC/DAC to get signals)

The whole package includes everything it needs:
- CPU (usually an ARM-based core)
- RAM
- flash
- peripherals
- buses to external devices

Since they are very simple components, there is a variety of manufactures that produce them (contrary to the microprocessor space where basically there are only Intel and AMD).

### Overview of microcontroller subsystems
- clock, used to synchronize all subsystem (speed usually configurable). Typically there are two clocks:
  - main clock, used for core, memory and peripherals (2 - 144 MHz)
  - RTC clock, used mainly for timekeeping, almost always at 32,768 Hz. Can be external to the MCU. In some systems can be used as a clock for extremely low power modes.
  - can be internal or external
    - internal, cheap but less precise
    - external, independent from the MCU, more reliable
  - partitioned in domains to run different subsystems at different clock speeds or to perform power saving optimization (see clock gating). Require a network generate the different speed and distribute it to the different domains.
- memory, often 16 or 32 bit addressing mode.  
RAM and flash are mapped to different areas in the same address space. A small cache may be present.
- GPIO (General Purpose I/O) can be configured as input or output. Usually organized in groups and can be digital or analog.  
There are registers that can be used to configure the functionality of the GPIO
- comparators, made with differential amplifiers. Usually the output is either 0 or saturated to Vdd. Can be read via interrupt (for rare events) or polling
- ADC, encodes an input voltage to a numeric value. Require stable clock source and power supply. A single ADC can be multiplexed over multiple inputs at the expense of conversion speed.
- timer, basically a counter. Can generate interrupt periodically, measure time intervals, count events, generate PWM signals.
  - watchdog, particolar timer to monitor the evolution of the system and detect possible failures. It needs to be periodically reset by the sw to ensure that it is operating correctly.  
  Since the watchdog is used to monitor the functionality of the system it is better to have an external clock for more reliable operation.
- PLL, phase locked loop. Electronic circuit that consist of a phase detector, a low pass filter and a voltage controlled oscillator (VCO) connected in a loop to maintain the VCO frequency locked to that of the input signal frequency. Adding a multiplier/divider it is possible to obtain multiples of the input frequency allowing to synthesize different clock signals at different frequency. F_out = F_ref*(N/R)

### Boot process
Sequence of steps that brings the system from power on to a fully operational state. The complexity of this process depends on the system (what hw is used, presence of an OS, ...).

Linux boot process:
- HW
  - poewr on the system
  - keep in RESET state the CPU until the power supply is stable
  - start the oscillators and waits for the PLL output to be stable
- BIOS
  - Power On Self Test (POST) executed from flash or ROM
  - find and executes peripheral's BIOS
  - find memory, ports and HDD
  - select the device to boot from based on pior user setting
  - executes the specific bootloader
- Bootloader
  - locates and loads the kernel at a fixed physical address
  - start executing from the kernel entry point
- Linux
  - load additional kernel modules to
  - start the init process
  - init starts all the services
  - init runs the login process
  - on login, a shell is started to allow interaction with the system

The BIOS execution is a crucial part for each system because it is the first firmware execution to start. For microcontrollers we will consider a system with:
- microcontroller
- flash memory where the binary is stored
  - the binary contains all the code to run
  - the code is run directly from flash
  - the binary have specific formats (e.g. ELF)
- RAM memory to store data during execution

Microcontroller boot sequence.  
The text section of the binary contains different infromation:
- Interrupt Vector Table, fixed size table containing
  - initial stack pointer
  - address for the reset handler
  - addresses of interrupt service routines
- startup code
  - to initialize the system, normally provided by the MCU vendor
  - usually written in assembly
- application code

The first thing to be exectued is the second entry of the IVT (i.e. execute the reset handler). :
- copy stack pointer in the SP register
- invokes a system initialization function (configure clocks)
- prepares memory
  - copy DATA section to RAM
  - zeroes the BSS section
- jump to main (OS or bare metal application)

What is the bootloader role in a MCU?
Loading the binary in the flash memory requires specific hw and sw tools and can only be done in a controlled environment. If in-field firmware update is required then a bootloader is needed to handle the replacement of the old firmware with the updated version.  
The bootloader is a separate binary from the application code, has a dedicate flash area and IVT.  
When the bootloader has completed the replacement of the firmware it has to:
- switch to the correct IVT of the new firmware
- jump to the firmware

There can be two cases, depending on the MCU
- presence of an IVT offset register
  - prepare the firmware execution
    - disable interrupts
    - wait for all memory operation to complete
  - execute the firmware
    - switch to the new IVT by changeing the value in the IVT offset register
    - load the new stack pointer value according to the new IVT
    - jump to reset handler
- no IVT offset register
  - IVT can only reside at two fixed addresses, one flash and one RAM (assume it is in flash)
  - prepare the firmware execution
    - disable interrupts
    - copy IVT from flash to RAM
  - execute firmware
    - switch to new IVT by enabling memory remapping
    - load the new stack pointer value according to the new IVT
    - jump to reset handler

### Programming microcontrollers
Microcontrollers are conceived to host the final application not to support code development, also because of limited resources available. This is why there is the need to use an IDE to support development like:
- KEIL, for ARM microcontrollers
  - C/C++ compiler
  - debugger support
  - libraries, templates
  - device driver support
  - RTOS features
  - Common Microcontroller Software Interface (CSIM), standardized platform for ARM Cortex based MCUs
- Gem5, allow also to simulate the system
- Code Composer Studio, focus on Texas Instruments products
  - includes C/C++ compiler
  - debugger, profile
  - built on top of eclipse
  - processor trace (trace instruction executed, timings, ...)
  - support linux kernel and application development
- CodeWarrior
- MPLAB
- IAR Systems
  - targets different arch
  - support RISC V

These are frequently linked to boards they target, some features may depend on the family of MCUs.

Fixed Virtual Platforms: software simulators of target boards, can execute code at near native speed, provides an environment to start developing and testing code.

## Firmware Update Over The Air
NOTE: naming --> OTA DFU, FOTA, OTAF  

Objective:
- add new functionality to the system
- address bugs and vulnerabilities
- ship product faster and add non-core functionality later on

Device Firmware Update (DFU) is the operation used to replace, partially or fully, the firmware on a device and relies on the presence of a bootloader.  
This functionality does not come for free, there needs to be specific hw support, extra memory to store new firmware, etc... not to mention the risks involved during the update operation like a power failure.

The bootloader is optimized and kept at a minumum
- to minimize boot time
- reduce the possiblity of having bugs
- maximize the available space for the application code

The DFU process involves many operations that add complexity:
- update the application
- verify the authenticity
- verify the integrity
- decrypt data
- downgrade prevention

Ensuring a robust OTA DFU:
- security: encrypt + verify identity with digital signatures
- reliability: verify integrity and recover in case of failures
- version management: rollback prevention + versioning system

Steps:
- firmware image and manifest are encrypted, signed and uploaded to the firmware update server
- devices query the server to fetch the new image and manifest
- decrypt, validate and apply the update

The firmware server is also a vulnerable part of the process and it must be kept secure.
Usually the binary is split in different packets (packetizing), each packet contains part of the data toghether with the address to where to store it in memory.  
Challenges:
- memory
  - new software must be organized in memory to be possible to execute it after the update
  - keep the previous version in case the new update has problems
  - retain the state of the device between resets and power cycles
- communication
  - packetization of the new binary
  - cost, both in terms of
    - battery energy consumed by the radio
    - metered network connections (e.g. GPRS)
- security, to ensure authentication, confidentiality and integrity

These actions are typically carried out by a Second-Stage Boot Loader (SSBL) that only enters into action when performing an update. Using a SSBL is better than having an application on to do this work because this can cause issues:
- if there is a RTOS, the application runs as a thread in the RTOS concurrently with other applications that can mess with the update process.  
Since the SSBL is not a RTOS program it doesn't have concurrent code running alongside it and can do this operations safely
- need to relocate the IVT to run the new application

What does th SSBL do exactly?
- determine which is the current application and branch to it at the start (their location is usually kept in a table of contents)
- update the ToC when the update process completes
- portions of the OTA update functionality can be pushed to the SSBL
  - e.g. check the integrity of the applications

Design tradeoffs
- compression
  - saves potential bandwidth costs and battery power for the radio
  - more processing on device to extract the binary
- caching
  - *no cache* can decide to wirte directly to flash at each packet --> wear flash quicker but simple to implement
  - *partial cache* in RAM to flash more packets in a single write --> more complex
  - *full caching* only available if the RAM is big enough to contain the firmware
- communication protocols
  - may be dictated by the presence of receivers on the device that are already there for other functionality
  - support for secure communication

Addressing security:
- keep OTA updates confidentials and verify integrity and identity of server  
--> use encryption: shared password between client and server to encrypt data. The crypto accelerator in the MCU may support AES 128/256.  
--> send hash of the firmware to enable integrity verification on the device.  
--> use of asymmetric encryption to verify identity of the server. Often there are not accelerators to do this and needs to be done in sw.

Summary of best practices
- digital signing <-- critical for integrity and verify identity
- encryption
- protect communication channels, both physically and the protocol used
- versioning <-- prevent downgrading
- recoverability
- logging and status reporting during DFU process <-- both anomaly detection and debugging purposes
- timely updates
- minimize downtime
- make user aware that there is an update to ensure ideal conditions
- use crypto accelerators if possible
- small scale tests before deploying to all devices to check functionality