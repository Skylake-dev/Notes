# Embedded Systems

## Introduction
### Overview on technology
Current CMOS technology is reaching its physical limits:
- cannot increase frequency of the single cores
- "dark silicon": not all the areas of a piece of silicon can be used simoultaneously due to heat generated that cannot be dissipated
- limit to how low the PSU voltage can get
- limit to how small transistors can be

Packing more transistors in the same package increases the power density, raising thermal issues. This impacts the reliability of the system since it cannot operate at high temperatures for long periods.  
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

However, typically a custom hw solution has much better performace, not only in terms of computing speed but also in terms of:
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
Buy components and mount them on a PCB and the interfaces between the different parts.
- Intellectual Property (IP)  
Buy the design of the microprocessor, there can be different levels of abstractions:
  - soft-macro: HDL description at an RT level. It's like acquiring the "source code" for the processor, easier to modify but costs more in terms of royalties.
  - hard-macro: described to the level of the layout. It's like having a "binary" for the processor, cannot modify.

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
Microcontroller Units (MCU) embed in a single chip everything that is needed to run the system. Can achieve lower power consumption giving up raw performance. Often they do not offer interface to external memory because typically they run very simple programs and to reduce the pinout. Include serveral peripherals and interfaces (I2C, SPI, JTAG, PWM, UART, watchdog, timer, analog I/O, ... see later lectures).  
These are programmed in C (sometimes assembly). SDKs can range from a simple compiler to fully fledged develepment environments with analysis tools.

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
- clock, used to synchronize all subsystems (speed usually configurable). Typically there are two clocks:
  - main clock, used for core, memory and peripherals (2 - 144 MHz)
  - RTC clock, used mainly for timekeeping, almost always at 32,768 Hz. Can be external to the MCU. In some systems can be used as a clock for extremely low power modes.
  - can be internal or external
    - internal, cheap but less precise
    - external, independent from the MCU, more reliable
  - partitioned in domains to run different subsystems at different clock speeds or to perform power saving optimization (see clock gating). Require a network generate the different speed and distribute it to the different domains.
- memory, often 16 or 32 bit addressing mode.  
RAM and flash are mapped to different areas in the same address space. A small cache may be present.
- GPIO (General Purpose I/O) can be configured as input or output. Usually organized in groups and can be digital or analog.  
There are registers that can be used to configure the functionality of the GPIO.
- comparators, made with differential amplifiers. Usually the output is either 0 or saturated to Vdd. Can be read via interrupt (for rare events) or polling.
- ADC, encodes an input voltage to a numeric value. Require stable clock source and power supply. A single ADC can be multiplexed over multiple inputs at the expense of conversion speed.
- timer, basically a counter. Can generate interrupts periodically, measure time intervals, count events, generate PWM signals.
  - watchdog, particolar timer to monitor the evolution of the system and detect possible failures. It needs to be periodically reset by the sw to ensure that it is operating correctly.  
  Since the watchdog is used to monitor the functionality of the system it is better to have an external clock for more reliable operation.
- PLL, phase locked loop. Electronic circuit that consists of a phase detector, a low pass filter and a voltage controlled oscillator (VCO) connected in a loop to maintain the VCO frequency locked to that of the input signal frequency. Adding a multiplier/divider it is possible to obtain multiples of the input frequency allowing to synthesize different clock signals at different frequency. `F_out = F_ref*(N/R)`

### Boot process
Sequence of steps that brings the system from power on to a fully operational state. The complexity of this process depends on the system (what hw is used, presence of an OS, ...).

Linux boot process:
- HW
  - power on the system
  - keep in RESET state the CPU until the power supply is stable
  - start the oscillators and wait for the PLL output to be stable
- BIOS
  - Power On Self Test (POST) executed from flash or ROM
  - find and executes peripheral's BIOS
  - find memory, ports and HDD
  - select the device to boot from based on pior user setting
  - execute the specific bootloader
- Bootloader
  - locate and load the kernel at a fixed physical address
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
  - the binary have a specific format (e.g. ELF)
- RAM memory to store data during execution

Microcontroller boot sequence.  
The text section of the binary contains different information:
- Interrupt Vector Table (IVT), fixed size table containing
  - initial stack pointer
  - address for the reset handler
  - addresses of interrupt service routines
- startup code
  - to initialize the system, normally provided by the MCU vendor
  - usually written in assembly
- application code

The first thing to be executed is the second entry of the IVT (i.e. execute the reset handler):
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
    - switch to the new IVT by changing the value in the IVT offset register
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
  - Common Microcontroller Software Interface Standard (CMSIS), standardized platform for ARM Cortex based MCUs
- Gem5, allow also to simulate the system
- Code Composer Studio, focus on Texas Instruments products
  - includes C/C++ compiler
  - debugger, profiler
  - built on top of eclipse
  - processor trace (trace instructions executed, timings, ...)
  - support linux kernel and application development
- CodeWarrior
- MPLAB
- IAR Systems
  - targets different architectures
  - support for RISC V

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

The bootloader is optimized and kept at a minumum:
- minimize boot time
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

These actions are typically carried out by a Second-Stage Boot Loader (SSBL) that only enters into action when performing an update. Using a SSBL is better than having an application to do this work because it can cause issues:
- if there is a RTOS, the application runs as a thread in the RTOS concurrently with other applications that can mess with the update process.  
Since the SSBL is not a RTOS program it doesn't have concurrent code running alongside it and can do this operations safely.
- need to relocate the IVT to run the new application.

What does th SSBL do exactly?
- determine which is the current application and branch to it at the start (their location is usually kept in a Table of Contents)
- update the ToC when the update process completes
- portions of the OTA update functionality can be pushed to the SSBL
  - e.g. check the integrity of the applications

Design tradeoffs
- compression
  - saves potential bandwidth costs and battery power for the radio
  - more processing on device to extract the binary
- caching
  - *no cache* can decide to write directly to flash at each packet --> wear flash quicker but simpler to implement
  - *partial cache* in RAM to flash more packets in a single write --> more complex
  - *full caching* only available if the RAM is big enough to contain the firmware
- communication protocols
  - may be dictated by the presence of receivers on the device that are already there for other functionality
  - support for secure communication

Addressing security:
- keep OTA updates confidentials and verify integrity and identity of server  
  - use encryption: shared password between client and server to encrypt data. The crypto accelerator in the MCU may support AES 128/256.  
  - send hash of the firmware to enable integrity verification on the device.  
  - use of asymmetric encryption to verify identity of the server. Often there are not accelerators to do this and needs to be done in sw.

Summary of best practices
- digital signing <-- critical for integrity and verify identity
- encryption
- protect communication channels, both physically and the protocol level
- versioning <-- prevent downgrading
- recoverability
- logging and status reporting during DFU process <-- both for anomaly detection and debugging purposes
- timely updates
- minimize downtime
- make user aware that there is an update to ensure ideal conditions
- use crypto accelerators if possible
- small scale tests before deploying to all devices to check functionality

## Timer and watchdog
A timer is a specialized type of clock used to measure time intervals while a counter stores the number of times an event occurred w.r.t. a clock signal.  
To read a timer we can look at the value stored in the register or wait to detect the overflow.
Timers and counters are the most pervasive peripherals in MCU design:
- improve performance
- replace looping CPU with timer interrupt
- there are COTS that are tailored to support programmable timers
- PWM for motor control to free the processor from this task

Every timer needs a clock source:
- can have different clock source
- there can be a prescaler to divide the clock before inputting it to the counter (factors of powers of 2)
- the count range is stored in a register and is called the modulus M

Common configuration for timers:
- select clock source
- dividing factor of the prescaler
- modulus value
- enable an interrupt
- ...

Periodic timers are useful to schedule activity, sampling correctly in an ADC, etc.

### Watchdog
It is a particular timer used to detect software anomalies and reset the processor if necessary.  
Basic idea:
- normally the system resets the watchdog timer to prevent it from timing out
- in case of a fault the timer is not reset and the timeout generates an interrupt
- the interrupt initiates the corresponding actions (not necessariliy a RESET)
- the action usually include going in a safe state (e.g. turn off controlled motors or heaters) and restore the correct functionality

Why are those critical? --> Detect and recover from faults on its own, necessary in IoT world with billions of devices.

There are different ways to implement a watchdog:
- internal vs external
- software enabled/disabled vs cannot disable
- windowed vs not windowed
  - windowed means that there is a minimum time that needs to pass before the watchdog can be cleared. If an attempt is made before this time the watchdog forces the reset.

Single stage watchdog: invokes a restart immediately after the timeout. It relies on the system reset to force outputs to safe states.  
Multiple stage watchdog: more timers in cascade. Each stages kicks the next and perform a set of corrective actions and only at the end a reset will be forced.  

To ensure safety there can be two sets of control states: runmode vs safemode. The timeout of the watchdog causes the selector to switch to safemode state to control the ouput and ensure that there are not potentially harmful values.

To have a more reliable watchdog it needs to be implemented as an external component with a separate oscillator to generate the clock. This is called independent watchdog

TIPS:
- never disable the watchdogs
- do not clear the watchdog using a periodic interrupt without checking functionality
- use an independent watchdog
- windowed is better

## Design of hardware systems
Different possibilities
- COTS
- ASIC
- programmable logic
- microprocessors or microcontrollers

### ASIC
Based on planar process. Use a semiconductor to realize all the main components (transistors, capacitances, diodes, ...) directly on silicon. The elements to do this are:
- silicon n --> more free electrons
- silicon p --> less free electrons
- insulator --> SiO2
- conductor --> depositions of aluminum or copper

All the regions are created on silicon at the same time using diffusion masks, which are very expensive to produce. This is why ASICs are an option only on large volume components.

Once the chips are ready they need to be integrated in a package to give mechanical stability and provide interconnect to the external world. There can be different choice for the package material like plastic, ceramic, metal.

New packaging techniques allow to build System in Package (SIP), interconnecting different components inside the same package. 

To reduce the complexity of a custom design there are so called *standard cells* that are offered by the silicon provider as a sort of "library" to develop the IC.

### Programmable logic
Hw platforms that allow fast prototyping of a system. These types of systems consume more power and take up more area than an ASIC but they have the advantage to be reprogrammable and cheaper to produce.

Classification can be based on different criteria:
- programming modes
  - One Time Programmable OTP (fuse and antifuse)  
    The lines are always connected and the programming step burns some connections and maintain only the one that are needed (opposite for antifuse).
  - Reprogrammable (EEPROM)  
    Non disruptive connections based on a floating gate. The idea is to place/remove charge from the floating gate to create/destroy a channel.
  - Reprogrammable and reconfigurable (SRAM, Flash)  
    Allows to reprogram the system while it is running (reconfigurable).
- connections
  - global connection  
    Can be shared by may components on the device --> long delays and low flexibility
  - local connection  
    Shared among few elements --> less delay, more flexible but also more complex routing 
  - hierarchical connection  
  - other e.g. programmable switch matrix

### Architecture and design
Due to their intrinsic heterogeneity and wide variety of constraints there is no standard architecture for embedded systems. The development process is usually done by prototyping the system on a development board to see if the performance of the selected processor is fine, the range of power consumption, etc. These boards often offer also integrate FPGA modules to add pieces of logic allowing for fast development of complex application.  
The connectivity that the board provides allow the developer to focus the effort on the application (see example on slides).  

## PCB based design
Printed Circuit Boards provide support and connectivity for the different components that will be mounted on it. They can also provide points for testing the system like monitoring power consumption or debugging interface that will be later removed from the final product.  
Virtually any ES makes use of PCBs to connect components, except in very particular cases.

The first step in designing a PCB based system is to select the componets that will be mounted on it (i.e. compile the Bill Of Materials BOM):
- passive --> resistors, buttons, LEDs, capacitors
- power supply --> create a specific power distribution to power the components, e.g. control power up sequences, create the different voltages, ...
- converters and filters --> ADC, DAC, amplifiers, oscillators
- others
  - electro-optical components to interface electromagnetic signals and optical signals (e.g. photodiodes, laser diodes, ...)
  - RF components like antennas
  - display
  - sensors --> convert physical measurements to voltage/current
  - digital components (microcontrollers, dedicated ICs, ...)  

It is also important to evaluate the different packages available that can influence the design of the PCB:
- mounting 
  - Through Hole TH, more robust connection and easier prototyping
  - Surface Mounted SMD has several advanteges:
    - smaller parts
    - denser layout
    - cheaper PCB (no holes to drill)
    - easier to shield from EMI/RFI
    - easier to automate
    - improved frequency response  
    But there are also some disadvantages:
    - small clearence for cleaning
    - generate more heat
    - difficult to visually inspect
- pin positioning
  - pin count
  - pitch (distance between pins)
- materials
  - influence thermal conductivity, metal transfers better but it's more expensive

### Structure of a PCB
A PCB consists of 3 main components:
- conductor
  - usually copper, used for wires and connecting components
- insulator
  - fiberglass epoxy
- glue (still an insulator)
  - adhesive materials that are called pre-peg

In PCBs with many layers there are ground and power supply layers in between.  
The final board is a stack of layers of insulators, where their faces host the wires, separated by pre-peg. The process to realize a PCB is quite complex and there exist specific file formats  to specify the design like [*Gerber*](https://en.wikipedia.org/wiki/Gerber_format).  
The footprint of the components that needs to be mounted on the board is necessary to make sure that all the components fits and the connections are right.

The manufacturing steps are
- realization of the dielectric supports
- realization of the wires 
  - use photoresist to cover parts, use acids to remove other areas and expose the wires
- assembly of the layers
  - stack them alternating pre-peg foils, a heated press finalizes this step
- drilling of the *vias*
- fabricating external layers, writing the schema
- finishing by applying a protective coating

After manufacturing the boards are tested to ensure that all the connections are working properly, this involve both optical and electrical inspections.  
After this we can proceed to the mounting step. For small boards and limited volumes it is done by hand, otherwise specific equipment is used. In any case the assembly requires the following steps:
- solder paste dispensing
- component placement
- reflow
  - increase the temperature to solder the components following a specific temperature profile
- inspection (look for misplaced components)
  - some fixes can be carried out by hand
- wash up the flux residuals
- passivate the surface with insulators if necessary

### PCB design criteria
Partition the development in separate groups in order to work in parallel until the final integration. Partitioning is not only driven by the functionality:
- number of signals necessary to connect components
- bandwidth of the signals --> at high frequency is better to keep the components on a single chip
- delays of the signals --> length and width of the wires affect the speed at which the signals can go

Software to design PCB:
- altium
- eagle

### Testing
Boundary scan testing: use a chain of cells and multiplexers that can be used to force certain operating states/read some values:
- normal operation --> the cells work normally
- test mode --> connect the registers in a chain and use them to force inputs or read values

See [JTAG interface](https://en.wikipedia.org/wiki/JTAG).

## System-on-Chip
Complete system integrated on a single silicon die:
- reduction of discrete external components
- small form factor

However not all parts of the system can be integrated for mechanical or electrical reasons
- high currents
- not supported by the integrated silicon process
- robustness of connections

Why implement an SoC?
- unit cost
  - NOTE: this includes only the production not the design
- performance, greater than PCB based solution
- reduction of energy and power required
- number of requested I/O, it's like having the possibility of fusing toghether multiple components
- security, both IP and sensitive information in memory

An SoC can include a variety of components:
- one or more processors
- different types of memories
- dedicated accelerators for specific operations
- time related functions --> timers, watchdogs, oscillators
- power supply distribution
- interfaces --> network, serial ports, ADC, DAC

Similarly to a PCB design, the first step is to partition the system in different functional units.  
The next step is to define the memory hierarchy and architecture. Classical buses solutions are falling out of favors due to bottleneck and are being substituted by a Network on Chip (NoC) approach.  
A NoC basically implements a small network directly on the chip with basic functionality like addressing and routing to allow communications between different parts of the chip. NoC can take up to 20/30% of the hw resources but they solve several complex problem in the design of the communicaiton part and are suitable for highly scalable solutions (e.g. beyond 8 cores a NoC is necessary).  

### Testability
Testing an single chip made of different components is a difficult task because they cannot be tested independently. This is why the design needs to account also for the testing part by preparing suitable interfaces:
- boundary scan like in PCB
- Built In Self Test (BIST)  to overcome the limitation of the scan (like operating at lower frequency)

## Distributed/Networked ES
Processing and data storage are split over multiple devices. There is a slight difference in terminology:
- distributed when computation are equally distributed
- networked when the nodes are just connected but are different

The critical part is implementing the communication between the nodes:
- wired
- wireless --> more complex
  - A relevant interest is in Sensor Networks

Examples:
- domotic  
This is more tricky because there are a variety of different devices with different computing capabilities and functionalities.
- automotive  
Usually in this fields distributed systems are used to ensure safety and avoid a single point of failure that can be a centralized system. the different systems are interconnected over a shared bus (CAN bus).
- Wireless Sensors Networks (WSN)
  - limited possibility to connect and store energy
  - operating in harsh enviroments
  - need for redundancy, both for failure of nodes and communications  
  The general architecture consists of a high number of sensors that communicate with a gateway to exchange information with the host. The typical approach to expose data is using a publish-subscribe model.

## Thermal and power management
Digital logic power consumption can be distinguished in two categories:
- active power `P = C*V*f^2`  
Related to the switching activity of the circuit, cannot be changed arbitrarily since to operate at a given frequency a certain minimum voltage is required.  
Over the past years due to Dennard's scaling it was possible to build a chip on smaller process, increasing density and frequency but consuming the same active power due to the possibility of operating at lower voltages and decrease in capacitance `C`. This is coming to an end because we are at the limits of how low the voltage can get.
- leakage power (static power) `P = I_leak*V`  
Increases as transistor size decreases and increases exponentially with temperature. This is not related to the circuit doing actual computations, it is just due to the fact the the system is powered on. This can only be reduced by reducing the voltage or interrupting the current flow by powering off devices via *power gating*.

High temperatures have negative effects on ICs in general:
- increase gate delay, slowing down the operating frequency
- lower MTBF
- thermal runaway: leakage power increases with temperature lead to increase the temperature --> positive feedback to the destruction of the IC.

To model the thermals of a chip we can use a simplified model that exploits the parallelism with electrical circuits (e.g. thermal conductivity --> electrical conductivity, heat source --> voltage generator, thermal mass --> capacitance).  
For silicon, thermal phenomenas are very fast (+20 °C in <100ms) so in chip design the approach is to run thermal simulators to builld a 2D map of the chip (e.g. 3D-ICE).  
To model it directly an IR camera is needed and the chip needs to be delidded (NOTE: this changes the conditions w.r.t. normal operation) or use a Thermal Test Chip that is able to simulate the behaviour of the chip by heating up some areas and compare different cooling solutions.

### Heat dissipations
The Thermal Design Power (TDP) is the power rating of a computing system that represent the maximum amount of heat that the cooling system is required to dissipate. We can have different approaches to cooling based on the power dissipated by the chip:
- Passive cooling: no heatsink, transfer heat directly from package to environment. Only viable for low power IC (single digit W).
- Heat sinks: metal components with high surface area and thermal conductivity placed on top on the package. Can deal up to tens of W because it still relies on natural convection.
- Forced air cooling: a heat sink with fans mounted on it to force convection and increase heat transfer up to a few hundred W.
- water cooling: replace air with water that can transfer heat faster, requires a radiator to cool the water.
- evaporative cooling: takes advantage of latent heat of state transitions of fluids with very specific boiling points.
- thermoelectric cooling: use Peltier effect, useful to remove heat from localized hotspot (still research in progress).

### Thermal control policies
The amount of heat that can be removed from a high end IC is limited but in many cases the thermal output is variable and depends on the load.  
The idea is to design a MPSoC (Multi-Processor SoC) that in the worst case draws more power that it can be dissipated and use thermal control policies to limit power draw during peaks. This allows the MPSoC to be faster on average than if it were limited by worst case dissipation.

Possible policies:
- stop and go: very simple policy (idle injection). Halt the CPU if exceeding a certain temperature threshold and restart when it falls below the threshold. This has a high impact on performance so it is often kept as a last resort policy to prevent thermal runaway.
- use Dynamic Voltage and Frequency Scaling (DVFS): very complex to implement, change dynamically the voltage and the frequency reducing the active power of the chip. Selecting the correct operating point is also complex but this allow to regulate more finely the performance.  
Typical policies are:
  - based on control theory and needs to be executed at a fast rate (1-100ms)
  - optimization-based, maximize performance given certain thermal constraints
- task migration: in multicore architectures when a core is overheating, the tasks it is executing can be moved to another core. This doesn't require to slow down cores but incurs in overhead to migrate tasks and there is still the need for a secondary policy if all cores are equally hot.
- event based: not execute policy at a fixed rate, require hw-sw co-design. A small hw state machine monitors sensors and generates events when it detects the needs for thermal response. Those events trigger the execution of the policy.  
This approach allows to have a faster response with less overhead.

### Power management at OS level
Power states are power saving policies that can be applied to the system. They are divided depending on the load of the system, we can have:
- active power states, save dynamic power, regulate the operating point depending on the load. Lose some performance.
- idle power states, save static power, put the system in sleep state by switching off components after some inactivity. Overhead to wake up the system.

![power_states](assets/power_states.png)  

Different power states can be defined at a device level (e.g. switch off display while CPU is active). To do this a standard interface was defined, ACPI (Advanced Configuration and Power Interface), that moves the power management under the control of the operating system by providing a standard way to discover and configure power state of the devices.  
The power states defined by ACPI have different hierarchical levels:
- G-states, global system states, example
  - G0 --> system is working, processes running, peripherals powered on
  - G1 --> sleeping, no user processes, preserve context info and resume without reboot  
  In G1 we can be in different S-states depending on the power saving measures.
  - G2 --> soft off, no code is run, no context preserved, needs a reboot (e.g. wake-on-LAN)
  - G3 --> mechanical switch off, no power supplied, no context, require complete restart
- S-states, system wide sleep states
  - S1 --> standby
  - S2 --> CPU powered off
  - S3 --> sleep or suspend to RAM:  
  Low wake up latency because all the state is saved in main memory that is kept powered on while the rest of the system is turned off.
  - S4 --> hybernation or suspend to disk  
  Higher latency to recover since all context is saved to disk but all the hw and peripherals can be switched off.
- CPU specific (note, can be applied to the single cores or at a package level)
  - C-states for power/sleep states (more can be implemented)
    - C0 active --> CPU is fully operational according to a P state
    - C1 halt --> CPU is idle, scale down clock
    - C2 stop-clock --> CPU idle, clock and frequency are scaled down
    - C3 sleep --> cache retained but disable coherency
  - P-states for performance states
    - P0 --> full speed
    - P1 --> reduced speed, scaled voltage and frequency, lower performance than P0
    - Pn --> further levels of scaling up to 255 maximum but usually there are not many due to the difficulty of selecting an optimal policy
- D-states for device level power/sleep
  - D0 --> device is fully on
  - D1 --> intermediate sleep, device specific, can be more than one
  - D2 --> switched off

#### Power management philosophy
We can have different power consumption profiles with equivalent energy consumption. Which approach is better?
- race-to-idle: completes the tasks as fast as possible and goes to sleep.  
Good for HPC or systems that do not require interactivity due to the wake-up latency. This approach also allows to save static power but can lead to have higher peak temperatures, affecting the choice of the cooling.
- slow down: usually adopted on ARM processors, use DVFS to save dynamic power.  
Takes longer to complete the task but no delays in waking up the system, suitable for interactivity (e.g. smartphones).

The best approach depends on the applications and the requirements that we have.

#### OS integration: Linux
There is an ACPI daemon that listen to events to handle power events (e.g. close laptop lid, battery events, ...). The device tree (DT) is exploited to also include power management information. Specific frameworks are used in the kernel in order to manage active and static frameworks. For the active power we have:
- cpuidle, manage transitions between C-states
  - latency limitations
  - minimum residency time (i.e. is it worth to save power?)
  - heuristics on CPU load
  - next predictable events (e.g. timers)
- Operating Performance Point (OPP) library, manage the P states by setting a pair of frequency-voltage values supported by the SoC power domains.
- cpufreq, most popular framework to perform DVFS, several governors ara available based on the different policies:
  - performance
  - powersave
  - schedutil, select according to utilization
  - ...  
  Interacts with the CPU driver to set the actual P-state.
- devfreq, same idea but for specific devices
- PM QoS Interface, used to set performance goals by drivers and application, can be set system wide or device speific. Poses constraints on the other frameworks governors for the selection of C, P and D states

![power_management_frameworks](assets/power_management_frameworks.png)  

For the static power management:
- Runtime PM, each device can register callbacks to be called on inactivity, does not involve directly user-space. In many case a driver must perform explicit active or idle requests. There is a reference counting mechanism to make sure not to turn off a devices that is needed by someone else.
- Generic Power Domains (PD), group devices in groups and apply policies directly at the domain level.
- Suspend --> standy, suspend to RAM, hybernation

## Battery operating embedded systems
Typically embedded systems are powered by rechargable batteries, factors to consider when choosing a battery are:
- reliability
- capacity in terms of Ah
- maximum peak current
- temperature operating range
- charging time
- deterioration
- cost

All batteries exploit a redox reaction to produce current, today the most common are based on litium ions and alkaline (NiMH).  
Capacity rating systems
- Reserve Capacity (RC): time required for a fully charged battery under a costant 25 A current draw to reach 10.5 volts at 80 °F.
- Cold Cranking Amps (CCA): Amperes that can be provided for 30 seconds at 0 °F while maintaining 1.2 V per cell.

Batteries also have a leakage current that discarge them over time (8 - 20% per year) so it is often the case that the operating period of an embedded system is more limited by the battery discharging itself than the power consumption of the system.  
Litium batteries needs specific circuitry to protect the battery from:
- short circuit on power supply
- complete discharge
- high voltage during charging
- high temperatures

### Supercapacitors
High-capacity capacitor with higher capacitance but very low voltage limits. Can store 10-100 times more energy per unit volume than electrolytic capacitors and can accept/deliver charge much faster than batteries.  
They are used where there is the need for many rapid charge/discharge cycles and to have a burst in power delivery or store temporarily some power.

### Power management
Often it is needed to generate different voltages from a single battery. It is done using DC-DC converters, like an LDO.

### Wireless charging
Works using magnetic induction like in a transformer, but without the ferromagnetic core. In compact devices planar coils are used and can reach good efficiency (>90%) if the distance is small enough, usually around 0.5 cm.

Example  
Qi standard: can provide up to 5 W of power from transmitter to receiver and defines 3 key areas of the system:
- transmitter: provides the inductive power
- receiver: uses the energy
- communication: unidirectional from receiver to transmitter, usually done using load modulation
  - protocol:
    - analog ping: detects presence of an object.
    - digital ping: longer version of analog ping, gives the receiver time to reply to ensure that the device is capable of wireless charging.
    - identification and configuration: receiver send necessary information to be identified and configure power transmission.
    - power transfer: receiver send regular messages (every 250 ms) to increase/decrease charge
    - end transfer: explicitly tells the transmitter to end or doesn't give signal for 1.25s

### Battery models
Energy density in battery is increasing but not at a rate to keep up with the computational and portability needs. Moreover they are non linear:
- the amount of energy that can be extracted depends on the current drawn (discharge profile)
- battery recovers some charge when it is given some rest

Guidelines:
- whenever possible use small current, taking into account the efficiency of the DC-DC converters
- avoid large current variations, use (super)capacitances to absorb these peaks

## Design space exploration
Build a system with the desired functionality while optimizing various metrics:
- unit cost
- NRE cost, one time monetary cost of the design
- size
- performance
  - not only average performance but also best and worst case.
- power
- flexibility, adapt functionality without incurring in heavy NRE costs
- time to prototype, time needed to build a working version of the system, not the final product
- time to market, time to deploy a system that can be sold
- maintainability
- correctness
- safety
- ...

Those metrics typically compete with one another, it is not possibile to optimize all of them at the same time, there is always a trade-off.  
Of course those metrics needs to be estimated during the design time and needs to be evaluated when the prototype is complete. There can be different indicators of the goodness of an estimation:
- accuracy `A = 1 - (E(D) - M(D))/M(D)` where E(D) and M(D) are the estimations and measurements of metric D. The perfect estimate have `A = 1`.
- fidelity, percentage of correctly predicted comparisons between design implementations, allows to discard the less promising alternatives.  
In practice take all the comparisons `if E(D_i) < E(D_j) then M(D_i) < M(D_j)` and viceversa. The fidelity measures how many of these actually hold.

### Automatic design space exploration
Frame the DSE as an optimization process to tune the different options while optimizing some target metric. The design space can be explored following different policies:
- random
- full factorial --> more entries leads to more accuracy but it takes more time

In these experiments we need to consider the statistical significance of variations between different options.

## Development cycle of an ES
The traditional development cycles is similar to the waterfall approach of software engineering.  
The main phases of development are:
- requirement identification
- design the hw and the sw parts
- production
- maintenance

In ES there is the so called V-model, more used in large companies  

![v_model_for_development](assets/v_model_for_development.png)  

or the spiral model  

![spiral_model_for_development](assets/spiral_model_for_development.png)  

The nice thing about the spiral model is the possibility to have fast prototyping and adjustments of the requirements down the line. It is more used in small companies or start-ups.

Of course in a real world scenario the different phases are not so clearly separated and there can be overlappings. The different phases are not balanced w.r.t. the effort required to complete them.

## Testing
Need to test the hw, the sw and the communication part.  
There can be different phases:
- simulation, all the elements are simulated, lower accuracy
- prototyping, some parts are simulated while others are actually implemented although they are not the final ones. Often use prototype boards and test directly on field.
- pre-production, all elements are real, need to validate that all the requirements have been met
- post-production, inspect quality of final product, cost of production and time required to produce. 
- End Of Line testing (EOL), test the final products with the sw flashed on it, both to configure the system and to test the connectivity of the components. Some of the testing routines can alse be left in the final product to facilitate repair or perform some remote analysis.

The transitions between the simulated and the real system can go through different steps, like testing the real sw on a simulated hw or test the hw in a simulated environment.  
Components that can be simulated:
- sw, directly on the host platform or on an emulator.
- CPU, can be susbstitued by one more powerful or use that to simulate the target one.
- other hw
  - emulated
  - build a prototype board, like on a breadboard or some specific SDK boards
- environment
  - static, only generate signals.
  - dynamic, interface with a simulation environment to have a feedback loops

In addition to the unit and integration testing for software, in an ES there is also the need to test the integration between hw and sw components and the whole system. It can be also tested under some specific environmental conditions to check the behaviour in a real environment.

## Documentation writing style
The software needs to be documented thoroughly in order to be maintainable. The quality of the code needs to be ensured by following coding styles and proper testing flow.  
Since the source code needs to be read by others it needs to be easy to understand. Optimized but messy code is more difficult to deal with while easy to read code can be optimized later.  
The programming languages used in the ES field are mainly C/C++ and sometimes assembly for low level operations.  
The quality of the code can be assessed using various metrics, some are more subjective, others can be properly measured:
- defects detected after the release
- mean time to fix a defect or add new functionality
- follow the MISRA guidelines (can be checked automatically by some commercial tools)
  - main goals: ensure same behaviour on different compilers/CPU

The documentation can be embedded directly in the source code and generated automatically with some tools like doxygen.

## Communication in ES
The communication is a critical part of and embedded system and it is implemented using some buses. There is a wide variety of protocols that can be used, it depends on the affordable cost and the requirements.  
A bus is a set of wires (that can be uni or bi-directional) with a single function. An associated protocol is used on top of the bus to perform the communication.  
Timing diagrams are the most common method to describe a protocol, representing the sequence of signals involved in the communication. To fully describe a protocol we need to identify the different parts involved:
- actors, entities involved in the communication
- data direction
- addresses, special kind of data to specify a certain memory location, peripheral or register.
- time multiplexing, share the bus for multiple pieces of data, used to save wires at the expense of time.

Control methods
- strobe
  - master assert a `req` to receive the data
  - servant puts data on bus within a certain time window
  - master receives data and de-asserts `req`
- handshake
  - master assert a `req` to receive the data
  - servant puts data on bus and asserts `ack`
  - master receives data and de-asserts `req`

Can have both approaches combined to adapt the master to fast or slow servants.

A microprocessor communicates with other devices using its pins. In a bus-based I/O we can have:
- memory mapped I/O, a peripheral is assigned a specific address in the memory.  
Do not require special instructions, accessing the peripheral is just like any other memory operation.
- standard I/O, no loss of addresses to peripherals.  
Require specific instructions to be accessed but simplify address decoding logic in the peripheral. 

If more peripherals are connected to the same bus there is the need to arbitrate the access. The access can be based on some kind of priority policy or in a round robin fashion. Daisy chaining is also possible (can have starvation issues for the peripherals further down the chain).

The communication with the peripherals can also be done using:
- polling
- interrupt
  - fixed, ISR is positioned at a fixed address
  - vectored, peripheral provide the address of the service routine

Interrupt controllers allow the CPU to increase the number of interrupt that it can handle and set different priority.

NOTE: DMA  
Direct memory access, used to allow peripherals to access directly the memory instead of passing through the processor every time. This minimizes the overhead.

### Communication protocols
A signal can have different properties depending on the purpose and the scale of the communication. Some general attributes are
- bandwidth
- encoding
- simplex, half-duplex, full duplex

In any case there are some control signals used to synchronize both ends to correctly decode the data. The main goals are:
- prevent errors in the transmission
- correct errors
- avoid conflict in accessing the media

#### Parallel communication
Suitable for short distance (a few meters). The transmission is done in groups of 8-16 bits toghether with additional wires for control signals to provide synchronization. Some examples of parallel protocols are SCSI, PCI, PCIe.

#### Serial communication
Can be used on longer distances, use of a single signal to trasmit bits one after the other toghether with control signals.  
- isochronous transmission, synchronization is based on the single clock signal that is shared from sender to receiver, only suitable for small scale communication due to clock skew.
- asynchronous transmission, tx and rx have their own clock but of similar frequency, require synchronization at each byte transmitted. Typical sequence  
` start -> LSB -> ... -> MSB -> parity -> stop `  
- synchronous transmission, receiver has a PLL that can extract the clock from the signal and use it for the synchronization.

Error detection and correction can be done at character level with the parity bit or at a message level using checksums or CRC.

Typical protocols: RS-232, UART, I2C and SPI.

I2C (Inter IC) was developed in the '80 by philips, the goal was to connect devices on a small scale with low bandwidth (100 kbit/s - 3.4 Mbit/s) in an half duplex communication. It uses only 2 lines:
- Serial Data Line (SDA)
- Serial Clock line (SCL)

The different devices can be master or slave, only masters can start the communication. there can be more than one master and there is a simple mechanism to negotiate the bus.  
The bus is synchronous and there is an ack message between every symbol sent by the receiver.  

SPI (Serial Peripheral Interface) synchronous, bidrectional with only 1 master. There are 4 lines for each slave:
- SCLK Serial CLocK
- MOSI Master Output Slave Input
- MISO Master Input Slave Output
  - wired-or on a single line, when not selected the slave needs to keep an high impedance mode
- SS Slave Select, one for each slave

The pros of SPI are:
- higher frequency
- MOSI and MISO lines allow a full duplex communication
- arbitrary length of data (not limited to 8 bits)
- simplicity
- robust hw structure
- clock produced by a single source
- signals can be buffered or isolated

The cons of SPI are:
- use more lines
- new pin for each slave
- not possible to have multi master
- no confirmation of transmission (ack)
- no peripheral addressing, can only select using SS signal

Comparison I2C and SPI:  
- both are synchrounous
- SPI has higher thoughput
- SPI is simpler

| I2C is recommended when | SPI is recommended when |
| -- | -- |
| need a multi-master | limited number of slaves |
| many units connected to one bus, often not known a priori | maximum transmission speed is required |
| need confirmation of received messages |  |

CAN (Controller Area Network), used in the automotive field. Designed to be resistant to electromagnetic interference. It is suitable for transmitting small messages and uses a CRC-15 to protect from errors. The messages are encapsulated in frames with fixed structure:  
` SoF -> arbitration (decide who is master) -> control -> data -> CRC -> ack -> EoF `  
The speed that can be achieved depends on the distance, it can reach up to 1km.  
The bus has no need for arbitration because it is based on priority.

## Sensors
Device that translates a physical measurements in a voltage/current value. There are many types of sensors both in terms of accuracy and form factors, not to mention power consumption.  
A sensor can be modeled as an equation that links the quantity we want to measure to the output, that can be fed into an ADC to get a digital value.  

Typically sensors only have a range of values in which the activity is linear and there is a uncertainty on the measure given depending on the sensitivity. Some sensors are not linear anywhere and typically have an exponential or logarithmic curve.  
Often integrated in the sensors there is the required electronics to convert the signal to digital and to smooth out the reading.

A specific class of sensors that is becoming popular in the embedded/IoT field is called MEMS, Micro Electro Mechanical Systems, a combination of electronics and mechanics that can be integrated in the IC, they can be mass produced at a low cost.  
They are very used today from consumer electronics to automotive and aeronautics because of their low cost. The main classes of MEMS are:
- pressure sensors
  - piezoresistive sensors
  - capacitive sensors
- inertial sensors
  - accelerometers  
  Used to detect crash, free-fall, pedometer, ...
  - gyroscopes  
  Vibration reduction for camera, roll-over detection
- optical
- RF
  - resonators
  - switches

Producers of sensors: Texas Instruments, Denso, STMicroelectronics, Seiko, Lexmark, Hewlett Packard, Bosch.  
MEMS improvements followed the same trends of the Moore's law.

### Energy harvesting
Is there a way to recover energy directly from the environment instead of relying on batteries?  
The most common approach in those case is to use a solar cell but in some cases this is not possible so a wide variety of techniques can be used:
- vibrations
- thermoelectric

The problem of these approaches is that they are not always applicable and usually not enough energy is harvested to guarantee operativity without a battery.

## Real time systems
A real time system is a system in which the correctness depends both on the logical correctness of the task's output and the time within the output is produced.  
An alternative definition is a system that has to respond to external inputs within a finite and specified period.  
` sensor and data processing -> computation -> actuation`  
`^------------------response time----------------------^`  
Example: airbag, when a collision is detected it needs to trigger the inflation within 10-20 ms.  
Timing constraints sometimes require the code to be on bare metal to avoid entirely OS overhead or implement some functionality directly in hw. When timing requirements are more shallow a Real Time Operating System (RToS) can be used.  
We can distinguish 3 classes of real time, depending on the consequences of missing a deadline:
- hard real time, lead to a catastrophic event on people or the system under control
- firm real time, invalid output value
- soft real time, degradation of performance

Properties:
- timeliness --> system must include time management mechanisms.
- predicatability, very hard to guarantee
  - predicting a priori if the timing constraints can be guaranteed
  - analyze temporal behaviour of the system at design-time
- determinism --> timing guarantees on the task execution at runtime also in case of external events to handle.

### Possible sources of unpredictability
- processor
  - modern processors include hw mechanisms to improve performance (prefetching, speculative execution, ...), it's ok for average performance but they are a source of non determinism
- cache
  - memory accesses can require an unpredictable amount of time due to cache hit/miss

To overcome this problems there is the need to specifically design simple microcontrollers or adopt processors with specific real-time features, for example ARM has different families of CPU depending on the purpose:
- Cortex-A, highest performance.
- Cortex-R, fast response, optimized for hard real time.
- Cortex-M, lowest power, optmized for discrete processing and microcontrollers.

Other sources of unpredictability are
- DMA, can "steal" bus cycles from the CPU to perform memory transfers
  - can be solved using a time-slice method: share deterministically the bus between CPU and DMA
- interrupts, can interrupt the execution of a task causing the missing af a deadline
  - can disable the interrupts except the timer interrupt and switch to polling to manage I/O (not efficient)
  - queue the interrupts and work on them periodically
  - minimize time spent in the ISR
- system calls, may delay termination of the task with the risk of missing a deadline
  - interruptible kernel routines: high priority tasks approaching the deadline should preempt also system calls
- memory management, page faults causes the same problems as cache misses but with higher latencies
- locking mechanisms
- programming languages, most programming languages do not provide constructs to express time constraints, moreover some techniques introduce not predictable latencies (recursion, switch statements, loops, ...).
  - use a specific language for real time
  - avoid recursion
  - avoid dynamic memory allocation
  - maximum number of loop iteration must be know a priori

### RTOS
OS specifically designed to guarantee real time requirements. There are many advantages in using a RTOS:
- part of the complexity and unpredictability is managed by the OS
- faster development w.r.t. bare metal programming
- real time schedulers manage timing constraints

The structure of an RTOS is similar to a normal OS with particular care to the time management and scheduler and often targets small microcontrollers.

Common features:
- multi-tasking and concurrency support (small number of task)
  - creation and execution of tasks
  - real time scheduling policies
  - inter-task synch mechanisms
- low-level user control
  - select task priority
  - prevent pages from being evicted from memory
  - select scheduling policy
  - select power management strategies, if present
- reliability and robustness support
  - manage failures to prevent data loss and system capabilities
  - multiple attempts to improve data consistency
  - guarantee minimum service level for some critical systems
- small memory footprint, usually require few kB of memory
- preemptive kernel
- bounded context-switch and interrupt management latencies
- determinism, handle external events such that no deadlines are missed

Examples of RTOS: VxWorks, FreeRTOS, Erika Enterprise, Miosix, ...

#### Tasks
In RTOS, tasks are characterized by timing constraints:
- d_i, absolute deadline, time before which the task should be completed
- D_i, relative deadline, difference between absolute deadline and arrival time a_i
- L_i, lateness, delay of completion w.r.t. its deadline --> `L_i = f_i - d_i`
- X_i, laxity or slack time, maximum time a task can be delayed to complete within the deadline --> `X_i = d_i - a_i - C_i` where C_i is the computational time required by the task to complete  

![RTOS_task](assets/RTOS_task.png)  

Tasks can be either periodic (e.g. sensor reads) or aperiodic (e.g. interrupts). The first type are more predictable and more easily manageable and make up most of the tasks in a typical ES.  
For periodic tasks we can define the *jitter*, the deviation of start time between two consecutive instances (jobs) of the task.

#### Real time scheduling
Schedulers that take into account also the timing constraints.  
A schedule is *feasible* if all the tasks of the given set can be completed without violationg time constraints.  
A set of task is *schedulable* if there exists at least one algorithm capable of producing a feasible schedule in a reasonable time.  

Assumptions:
- instances of periodic tasks are activated regularly at a constant rate
- all instances have the worst case execution time
- all instances have the same deadline equal to the period
- all tasks are independent (no precedence relation or resource constraints)
- tasks cannot suspend themselves
- release time coincides with the arrival of the task
- kernel overheads are considered zero

Define CPU utilization as the sum over all the tasks of the computational time over the period of activation, useful to check the feasibility of a schedule (U > 1 --> not feasible). A good compromise is to design the system not to use more tha 70% of the resources available to have some headroom for some unpredictable events or longer than expected tasks.  
Given a scheduling algorithm the *least upper bound* U(A) is the minimum utilization factor over all the tasks set that fully utilizes the processor.  

Scheduling classes:
- timeline scheduling, also known as cyclic executive scheduling
  - divide temporal axis in equal length slots
  - one or more tasks are activated per slots
  - greatest common divisor of the activation periods is the time slot length (minor cycle)
  - least common multiple of activation periods for the time interval after which the schedule repeats itself (major cycle)
  - schedulability: Worst Case Execution Time (WCET) is lower than the minor slot
  - pros: simple, low overhead, no jitter
  - cons: not robust, sensitivity to application changes, hard to handle aperiodic tasks
- rate monotonic
  - assign priority according to the activation rate: higher rates (short period) --> higher priority
  - fixed priority
    - optimal among all fixed priority algorithms
  - preemptive, a new task preempts the current one if it has a shorter period
  - actually exploited in industrial solutions
  - stable in case of transient system problems
- earliest deadline first (EDF)
  - dynamic priority assignement --> according to the absolute deadline, i.e. the more a deadline is close, the more the task has priority
  - preemptive, if a task with earlier deadline is activated it preempts the current one
  - agnostic w.r.t. to periodicity
  - EDF is optimal for single core processors
  - higher overhead w.r.t. rate monotonic due to dynamic priority but less context switch
- deadline monotonic DM
  - generalization of rate monotonicity
  - fixed priority, inversely proportional to the relative deadline

#### WCET analysis
How can the designer of a system estimate the worst case execution time of tasks? This is a particularly problematic job in hard real time systems to properly schedule the tasks and guarantee timing requirements.  
To start, during design it is possible to collect some samples of the execution time in different conditions to have an estimation of the WCET.  
Ideally we want a tradeoff between having a safe value of the WCET and not wasting resources.  

![WCET_estimation](assets/WCET_estimation.png)  

We need a way to approximate the WCET as accurately as possible since we cannot explore exhaustively the input space (jumps, branches, FP operation depend on operands) and machine state (cache, pipelines, peripheral statuses, DMA, multicore, concurrent tasks, ...).  
Possible interferences in estimating the WCET:
- intra-task, self inflicted by the task itself, e.g. evicting a cache line that is later needed
- intra-core, another concurrent task running on the same core e.g. evitcs a shared cache line
- inter-core, tasks running on other cores e.g. occupation of shared bus

Computing a safe and tight WCET when taking into account the task interference is hard, especially with multicore and multilevel caches.  

Other sources of interference can come from the hw itself:
- memory controller: variability in latency for memory operations
- NoC traffic: routing algorithms can not always guarantee latencies
- access to front side bus
- cache state
- cache latencies
- power and thermal management: techniques like DVFS can impact execution time
- system management interrupts: special purpose interrupts managed directly by hw
- instruction prefetching and speculation: optimize average time but often worsen the worst case.

Due to this complexity, many hard real time systems try to cut the complexity:
- use of single core
- only 1 level of cache
- no prefetching
- no speculation
- use stable and old technology

The WCET clearly depends on the target machine. Is it better to analyze the binary code instead of the source code:
- have direct one-to-one match with machine instructions
- not susceptible to compiler optimization or use of different compilers

The steps that traditional analysis tools follow are:  

![WCET_analysis](assets/WCET_analysis.png)  

However, exploring all the possible combinations is unfeasible because the state space is too large.  
Other issues:
- computing the correct WCET is an undecidable problem
- scheduler decisions and task interference impact the WCET
- architectural analysis of moder arch is computationally unfeasible
- approximate architecture is not easy due to *timing anomalies*
  - e.g. cache hit/miss --> should we always assume a miss? NO, timing anomalies may happen.  
  This appen because of data dependency between instructions that can lead to have higher exectution time in case of a cache hit  
  ![WCET_timing_anomalies](assets/WCET_timing_anomalies.png)

Estimation of WCET for concurrently running tasks:
- murphy approach: compute WCET of each task considering all possible interference
  - pros: simple
  - cons: very pessimistic analysis
- integrated analysis: take into account all possible interferences among all the tasks in the entire task set
  - pros: very high precision in the estimation
  - cons: computationally unfeasible even for 2 tasks
- isolation approach: partition resources to avoid interference (e.g. isolate tasks on specific cores, time-slice for buses, ...) --> probably most used approach
  - pros: easy to analyze
  - cons: underutilization of resources, partitioning require hw support
- time composability: compute WCET without interference --> compute interferences associated to each resource --> recompute WCET of each task accounting for the interfences at each resource
  - pros: precise and efficient
  - cons: hard to compute interferences, often assumption-based and hard to verify

##### Evolutions
- probabilistic approaches
  -  provide a probabilistic-WCET, a statistical distribution given a probability of failure P(ex_time > WCET).  
Technically the WCET is underestimated and therefore unsafe but sometimes it is good enough, not yet suitable for real life-critical systems.
  - Static Probabilistic Timing Analyss (SPTA), add probability values to the CFG analysis, statistical operators are used to combine different execution time estimations.
  - Measurement Bases Probabilistic Timing Analysis (MBPTA), treat the system as a black box and measure the timings giving inputs to the system using Extreme Value Theory EVT
  - pros: 
    - probabilistic analysis provides lower WCET than traditional approaches, exploiting better the resources
    - system complexity is not an issue
  - cons:
    - difficult to perform some fine grained analysis of the system to identify problems
    - hard to prove the necessary EVT hypotheses for MBPTA

NOTE: WCET is related to the WCEC (Worst Case Energy Consumption), a measurement crucial in energy constrained environments to guarantee uptime.

#### Mixed-criticality systems
Modern systems are composed by tens of subsystems/microcontrollers to control different parts (e.g. think of a car).  
Problems:
- development cost
- underutilization of resources
- reliability and safety
- system interoperability --> how do all of these systems communicate and synchronize?

Possible solutions:
- system consolidation (not typical in automotive for safety)
  - more difficult than it seems because this places on a single system different parts with different criticality (e.g. steering control and infotainment system).
  - pros:
    - lower design cost
    - better power and energy optimization
    - improves reliability of the overall system
    - lower maintenance costs
    - reduce bus congestion
  - cons:
    - increase shared resources contentions and unpredictable behaviour
    - large integration effort required
    - how to manage different criticality levels?

A mixed-criticality system is a system running tasks with different criticality levels.  
Criticality is determined by considering both the impact of failure and the required rate of failure.  
Trivial solution: consider all the tasks in hard real time --> unfeasible for cost or impossibility to compute WCET for some tasks (e.g. infotainment)  

Mixed-criticality task model, all tasks have:
- criticality level, can be expressed in the form of [ASIL levels](https://en.wikipedia.org/wiki/Automotive_Safety_Integrity_Level).
- a set of WCET for each criticality level

When a task overrides its WCET, a system mode switch occurs, each task that is under the current critical mode is dropped and not scheduled, only the higher criticality tasks are scheduled (NOTE: ASIL-D tasks are never dropped, the most critical).  
A good system should be designed to
- ASIL-D tasks are executed in any condition
- mode switches accure rarely to minimize disruption of other tasks

Multiple scheduling algorithms have been proposed, the most famous is a variation of EDF, called EDF-VD (Virtual Deadline) that takes into account possible mode switches.