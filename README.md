# ERS 3864K, a Logisim Evolution Harvard RISC 8Bit computer

## What is a ERS 3864K?
The Einfaches RechenSystem (simple calculation system) 3(version)8(bit)64K(Byte RAM) is a complete computer system build in the digital logic simulator [Logisim Evolution](https://github.com/logisim-evolution/logisim-evolution). The goal was to create a "simple" computer system with an CPU that supports all functions needed for multitasking.

## The CPU: RX 8 Lepton
The heart of the computer system is the RX 8 Lepton CPU, which is based on an own developed ISA. The architecture is based on an Harvard RISC like design. It needs 2 cycles to execute one instruction (no pipelining, sry). It can run in different modes and can switch between them during runtime (kernel/user mode, virtual/direct addressing). The CPU consists of the following parts:
- The 8Bit ALU
- Register bank consisting of 8 universal registers, 4 kernel register, 2 pointer and 2 interrupt call register
- CU with the CPU-mode registers
- Counter with build in branch logic
- RAM with 16MByte capacity. Yeah there is more then 64KByte but more to that later
- Interrupt controller with 32 (maskable) interrupts
- and a very simple MMU

## Interrupts
The interrupt controller has 16 module interrupts (one for each module) 12 special hardware interrupts (one is used for user mode violation and a second is the timer interrupt) and 4 interrupts which are reserved for software. The prioritization of the interrupts is handled with some sort of "daisy chain". Software fired interrupts have the highest priority. The next in the list are the modules with the highest priority to module 0. Then comes the user mode violation interrupt, the 10 free special hardware interrupts and the timer interrupt. Theoretical the 4 software reserved interrupts would have the lowest priority but because they are only manual activated they have actually the highest priority.

## Multitasking and the not-64KByte-RAM
The CPU can directly address 64KByte ROM and 64KByte RAM. Not more! But thanks to the MMU it's possible to cheat a little bit and extend the range to 16MByte ROM and 16MByte RAM. All TLB-entries in the MMU are 16Bit width and are set with the 2 pointer register. The page size or offset part is fixed to 256Byte (8Bit). With that it can address 24Bit addresses. That means 256 processes could (only) theoretically work isolated from each other in it's own 64KByte address space. With some software tinkering you could even use the MMU as "additional 8Bit pointer register" to make all 16MByte available to one process.

The interrupt controller has an timer interrupt, which can be set and activated in kernel mode. The timer fires an interrupt if the counter reaches an reference value (which can be set in kernel mode too). The timer counter counts + 1 every instruction (not clock signal) and can be reset by switching it off in the CPU mode register. Some addition: the MMU is NOT automated and all entries must be set directly by the programmer.

## Bus and Modules
The CPU can handle a simple bus system for I/O. The CPU can send values to up to 16 different modules. For that it uses the pointer as data source and a register for module addressing. To receive data from a module it has to trigger an interrupt and send the (16Bit) value to the CPU. The bus and it's modules are not able to handle DMA. Atm. 3 modules are build and functional:
- A keyboard module with address 0 (read only)
- A terminal module with address 2 (write only)
- A crude IDE-like Not-an-IDE-controller with address 4 (read and write, it can't be used as program memory)

## Handbook
Since it would take a whole handbook to understand the computer i wrote one with detailed information about the computer, ISA references, modules (and a simple description what new modules need to do to become functional and compatible) and a whole programming tutorial. It's in the repository and named HowToERS3864K.

## Known problems and bugs
1. Some module interrupts are send to the shadow realm (rare but yeah)
2. The internal timing is oof (the system works anyway)
3. The overflow flag of the ALU is not trustworthy

## Future
The ERS3864K is just a tech demonstrator for my future neumann-like CPUs and as such will not be futher developed. I will try to fix as many function breaking bugs as possible and finishing the handbook but not more.
