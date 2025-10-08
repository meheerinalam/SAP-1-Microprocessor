# SAP-1-Microprocessor
This repository contains the Logisim Evolution implementation of a Simple-As-Possible (SAP-1) CPU. The SAP-1 is a foundational computer architecture used to teach the basic principles of CPU design.

This repository documents the design and implementation journey of a Simple-As-Possible (SAP-1) Microprocessor.

The project began with the schematic modeling of a basic CPU architecture in Logisim — including the general-purpose register, ALU, Program Counter, RAM, and Instruction Register — gradually evolving into a fully functional SAP-1 microprocessor.
Each stage of development, from manual control sequencing to automated microcoded execution, reflects a step toward understanding the fundamental working principles of computer architecture.

The repository is organized into branches:

1. *early CPU design* — contains the primary CPU design, built during the early development phase.

2. *main* — holds the latest and complete SAP-1 implementation with enhanced features and control automation.
   
Kindly find them on the branch for the fundamental understanding of the prototype. All the neccessary images for the circuita are attached to sap-1 v.2 file.

To implement this CPU, specific components have been modified and a new control sequencer circuit has been designed.

## SAP-1: Design Philosophy & Build Steps 

### Design Philosophy
- **Go beyond textbook SAP-1** by adding automation and software–hardware integration (microcoded control, RAM auto-loader, assembler) to make the platform scalable and realistic. :contentReference[oaicite:0]{index=0}
- **Layered architecture** to isolate concerns:
  - *Hardware (data path + bus)* for correctness and timing,
  - *Microcoded control layer* for instruction sequencing and extensibility,
  - *Automation layer* for practical program loading and tooling. :contentReference[oaicite:1]{index=1}
- **Unique capability**: add `JMP` to enable control flow (loops/branching) absent in classical SAP-1. :contentReference[oaicite:2]{index=2}

### 1) Define scope, tools, and objectives
- Objectives: implement **microcode control sequencer**, **RAM auto-loader**, **JMP**, and a **Python assembler**. :contentReference[oaicite:3]{index=3}
- Tooling: Logisim Evolution (design/simulation) + Jupyter/Python (assembler). :contentReference[oaicite:4]{index=4}

### 2) Establish the baseline SAP-1 data path
- Build the classic modules (PC, MAR, RAM 16×8, IR, A, B, ALU, Output) on an **8-bit system bus** with a clock/timing unit. Emphasize modularity and synchronized signals across T-states. :contentReference[oaicite:5]{index=5}

### 3) Microcoded Control Sequencer (heart of the system)
- **Addressing**: 7-bit microaddress = `Opcode[3:0] + T-state[2:0]`. One entry per `(instruction, T)` pair. :contentReference[oaicite:6]{index=6}
- **Microinstruction width**: 13-bit control word (drives enables/loads for each unit). :contentReference[oaicite:7]{index=7}
- **Problem solved**: manual wiring is brittle; microcode centralizes control and **scales** as you add instructions. :contentReference[oaicite:8]{index=8}

### 4) Define universal T-states (Fetch/Decode/Execute framing)
- Execute each instruction in **6 T-states**; map standard fetch first, then instruction-specific steps. :contentReference[oaicite:9]{index=9}
- Baseline sequence used in your design:
  - `T1: PC → MAR`, `T2: RAM → IR`, `T3: PC++`, `T4–T6: opcode-specific`. :contentReference[oaicite:10]{index=10}

### 5) Generate timing (ring counter) and decode opcodes
- **Ring counter** (PC + 4×16 decoder) emits T1…T6; **auto-reset at T6**; buffered for controlled resets. :contentReference[oaicite:11]{index=11}
- **Instruction decoder** (4×16 decoder) driven by IR’s upper nibble → one-hot opcode lines (dictionary below). :contentReference[oaicite:12]{index=12}
- **Opcode dictionary** used: `LDA=0001`, `LDB=0010`, `ADD=0100`, `SUB=0101`, `JMP=1000`, `HLT=1111`. :contentReference[oaicite:13]{index=13}

### 6) Compose the Control Sequencer
- Combine **ring counter** (T-state) with **instruction decoder** (opcode) to index microcode and assert control lines.
- Documented micro-ops per instruction (examples):
  - **LDA addr**: `T4 IR(addr)→MAR`, `T5 RAM→A`.  
  - **ADD addr**: `T4 IR(addr)→MAR`, `T5 RAM→B`, `T6 A+B→A`.  
  - **SUB addr**: `…`, `T6 A−B→A`.  
  - **OUT**: `T4 A→OUT`.  
  - **JMP**: `T4 CONST(1000₂)→Bus`, `T5 Bus→PC`.  
  - **HLT**: `T4 Stop clock`. :contentReference[oaicite:14]{index=14}

### 7) RAM Auto-Loader (solve slow/fragile manual entry)
- **Goal**: eliminate switch-by-switch RAM programming; simulate realistic boot. :contentReference[oaicite:15]{index=15}
- **Design**: two counters orchestrate alternating cycles—odd clock loads address (`MAR_in` high), even clock writes data (`RAM_wr` high). :contentReference[oaicite:16]{index=16}
- **Data path**: `Debug → Address → MAR_in → clk → Instruction → RAM_wr → clk → data committed`. :contentReference[oaicite:17]{index=17}

### 8) Implement the unique `JMP`
- **Approach**: keep fetch (`T1–T3`) standard; on `JMP`, assert `jmp_en` to **restart at T1** and **load PC from bus** (added PC load path). :contentReference[oaicite:18]{index=18}
- **Result**: non-sequential flow (loops/branches) without rewiring the data path. :contentReference[oaicite:19]{index=19}

### 9) Build the software toolchain (Assembler)
- **Assembler in Python/Jupyter** converts assembly → hex machine code that the auto-loader ingests. :contentReference[oaicite:20]{index=20}
- **Why**: guarantees consistency between source, opcode dictionary, and microcode; speeds iteration. :contentReference[oaicite:21]{index=21}

### 10) Test & validate
- Run sample programs through the **auto-loader → sequencer → data path**; verify ALU ops and control timing across T-states. :contentReference[oaicite:22]{index=22}
- Outcomes: correct execution on all tests; stable control; large reduction in setup time; `JMP` verified; assembler–hardware compatibility confirmed. :contentReference[oaicite:23]{index=23}

# Control Signals
### Control flow (big picture)
`[Opcode + T-state] → Microcode ROM → Control Word → Enables/Loads → Bus/Data Path → Result`

### Fetch cycle (universal)
`T1: PC→MAR → T2: RAM→IR → T3: PC++`

### Execute (LDA addr)
`T4: IR[addr]→MAR → T5: RAM[addr]→A → T6: (idle)`

### Execute (LDB addr)
`T4: IR[addr]→MAR → T5: RAM[addr]→B → T6: (idle)`

### Execute (ADD addr)
`T4: IR[addr]→MAR → T5: RAM[addr]→B → T6: A+B→A`

### Execute (SUB addr)
`T4: IR[addr]→MAR → T5: RAM[addr]→B → T6: A−B→A`

### Execute (OUT)
`T4: A→OUT → T5: (idle) → T6: (idle)`

### Execute (JMP addr)
`T4: IR[addr]→Bus → T5: Bus→PC → T6: Restart at T1`

### Execute (HLT)
`T4: Halt Clock/Sequencer → T5: — → T6: —`

### Data path (ALU view)
`Bus→A/B → ALU{Su=0: A+B | Su=1: A+B′+1} → Bus/OUT`

### Bus organization
`PC/MAR/RAM/IR/A/B/ALU/OUT ↔ [8-bit System Bus] (tri-state enables)`

### RAM auto-loader (your approach)
`Addr Ctr→MAR → (clk) → Data Ctr→RAM_WR → (clk) → Next Addr/Data`

### Instruction register split
`IR[7:4]=Opcode → Decoder → Microcode  |  IR[3:0]=Operand → Bus (when enabled)`

### Timing generation
`Clock → Ring Counter (T1–T6) → Micro-address + Opcode → Microcode Word`

### Program memory access
`PC→MAR → RAM[PC]→IR → PC++`


## Conclusion

This SAP-1 build started as a textbook datapath and grew into a practical, extensible microprocessor. By layering a microcoded control sequencer over a clean 8-bit bus, adding a RAM auto-loader, and pairing the hardware with a lightweight assembler, the project moved from switch-level demos to repeatable, software-driven execution. The inclusion of `JMP` enabled real control flow without complicating the datapath, and the modular design kept every block testable in isolation.

**Outcome highlights**
- Deterministic 6-T-state fetch/execute cycle driven by microcode  
- Stable ALU operations (ADD/SUB) with clean A/B register interfacing  
- Automated, error-free program loading and consistent assembly → machine code flow  
- Readable control word and timing, suitable for teaching and extension


**Next steps**
- Add flags (Z, C) and conditional branches  
- Expand the ISA (STA, AND/OR, INC/DEC) and microcode pages  
- Explore a simple I/O bus and clock divisors for hardware demos

## References

- https://karenok.github.io/SAP-1-Computer/
- https://github.com/aukhalid/SAP-1-CPU-Logisim/tree/main
- https://github.com/FaisalAhmedBijoy/SAP-1-Computer-Design-Logisim/tree/main
- https://sap1-simulator.almeda.io/
- https://circuitverse.org/users/7241/projects/sap-1-d52a7168-e6e6-4910-87e9-35aa9999a112
- https://circuitverse.org/users/61326/projects/sap-1-controller-sequencer-579c2cdd-c15f-4f92-a51f-419ae70492a2
- https://en.wikipedia.org/wiki/Finite-state_machine

> **Note:** The following are Chrome-extension PDF viewer links and may **not** work on GitHub. Replace them with the original PDF URLs if available.
>
> - chrome-extension://efaidnbmnnnibpcajpcglclefindmkaj/https://www.robots.ox.ac.uk/~adutta/data/bce/cad-sapreport.pdf  
> - chrome-extension://efaidnbmnnnibpcajpcglclefindmkaj/https://www2.cs.sfu.ca/CourseCentral/150/dbg/overheads/w7_sap-1-notes.pdf
