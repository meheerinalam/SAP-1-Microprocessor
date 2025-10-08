# SAP-1-Microprocessor-
This repository contains the Logisim Evolution implementation of a Simple-As-Possible (SAP-1) CPU. The SAP-1 is a foundational computer architecture used to teach the basic principles of CPU design.

This branch contains the initial CPU design that served as the foundation for the later SAP-1 microprocessor.
The design includes the core functional units — General-Purpose Register, Arithmetic Logic Unit (ALU), and Program Counter — interconnected through an 8-bit bus architecture. These components were modeled and tested individually in Logisim, verifying data flow, control signal behavior, and arithmetic operations.

This early version represents the first functional prototype of the CPU, built to understand and validate the essential concepts of data transfer, storage, and execution before developing the full SAP-1 system.

The execution for this CPU was manually controlled, meaning all the control signals were pulled up and pulled down one-by one by hand.

## SAP-1 Build & Run — Step-by-Step

### 1) Set up the core building blocks
- **General-purpose 8-bit register** (used as A, B, and Output):
  - 8× D-FF bank with input MUX (select previous value vs. bus) gated by `reg_in_en`.
  - 8-bit tri-state output buffer gated by `reg_out_en`; splitters connect to the system bus. :contentReference[oaicite:0]{index=0}
- **8-bit ALU (Adder–Subtractor)**:
  - Cascade 8 full adders (ripple-carry). Feed A directly; feed B through XORs controlled by `alu_sub` to form 1’s complement; add `alu_sub` to Cin for 2’s-complement subtraction.
  - Drive result onto bus via tri-state under `alu_out_en`; expose `alu_carry_out` for flags. :contentReference[oaicite:1]{index=1}
- **Program Counter (PC)**:
  - 4-bit asynchronous counter (JK-FF based) with `pc_en`, `pc_reset`, and bus output enable `pc_out_en`. :contentReference[oaicite:2]{index=2}

### 2) Build memory & addressing path
- **(Optional) 4→16 decoder** to select one of 16 RAM addresses from a 4-bit address. :contentReference[oaicite:3]{index=3}
- **SRAM cell (subcircuit)**:
  - Register core with `wr_en`, `rd_en`, combined by `cs` (chip-select). :contentReference[oaicite:4]{index=4}
- **16×8 RAM**:
  - Arrange 16 cells; address from MAR; provide `sram_wr`/`sram_rd` controls; data bus connects to central bus. :contentReference[oaicite:5]{index=5}
- **Memory Address Register (MAR)**:
  - 4-bit register that latches address from bus under `mar_in_en` and fans out to RAM address lines. :contentReference[oaicite:6]{index=6}

### 3) Instruction path
- **Instruction Register (IR)**:
  - Latch 8-bit instruction from bus under `ins_reg_in_en`.
  - Expose upper nibble (opcode) to control; present lower nibble (operand) back to bus via `ins_reg_out_en`. :contentReference[oaicite:7]{index=7}

### 4) Control concept (manual or sequencer)
- **Control/Sequencer signals** (12-bit “control word”) drive all enables/loads per T-state:
  - `CON = Cp Ep ~Lm ~CE ~Li ~Ei ~La Eu Su Eu ~Lb ~Lo`. (Active-low denoted by `~`.) :contentReference[oaicite:8]{index=8}
- For the early build, **operate manually** by toggling control pins to learn the cycle; later replace with an automated sequencer. :contentReference[oaicite:9]{index=9}

### 5) Integrate the system (the bus)
- Connect PC → (via bus) → MAR → RAM; RAM → (via bus) → IR / A / B.
- Permanently feed A into ALU; feed B through XOR bank; return ALU result to bus/output register under `alu_out_en`.
- Tunnel all control pins to a single “control panel” for easy manipulation; add `debug_control` and `debug_data` pins to program RAM. :contentReference[oaicite:10]{index=10}

---

## Program & Execute

### A) Program the RAM (example: `LDA 10` with data at address `1010`)
- Turn **debug** on; **reset PC**.
- Write instruction `0001 1010` to address `0000`:
  - Set `debug_data=0000 0000` → `mar_in_en` + clock (select addr 0).
  - Set `debug_data=0001 1010` → `sram_wr` + clock (store opcode/operand). :contentReference[oaicite:11]{index=11}
- Write data `0000 0111` to address `1010`:
  - Set `debug_data=0000 1010` → `mar_in_en` + clock.
  - Set `debug_data=0000 0111` → `sram_wr` + clock. :contentReference[oaicite:12]{index=12}
- Turn **debug** off.

### B) Run the instruction cycle (manual control)

**Fetch (3 T-states, each with a clock):**
- **T1**: `pc_out_en` + `mar_in_en` → clock (PC → MAR). :contentReference[oaicite:13]{index=13}
- **T2**: `sram_rd` + `ins_reg_in_en` → clock (RAM → IR). :contentReference[oaicite:14]{index=14}
- **T3**: `pc_en` → clock (increment PC). :contentReference[oaicite:15]{index=15}

**Decode:**
- Interpreted combinationally from IR (manual build = you act as the decoder). :contentReference[oaicite:16]{index=16}

**Execute (`LDA addr`, 3 T-states):**
- **T1**: `ins_reg_out_en` + `mar_in_en` → clock (operand → MAR). :contentReference[oaicite:17]{index=17}
- **T2**: `sram_rd` + `a_in` → clock (read RAM[addr] into A). :contentReference[oaicite:18]{index=18}
- **T3**: Unused for LDA. :contentReference[oaicite:19]{index=19}

> Extend this pattern for `ADD`, `OUT`, `HLT`, etc.; once verified, replace manual toggling with a control sequencer to emit the proper control word per T-state. :contentReference[oaicite:20]{index=20}

