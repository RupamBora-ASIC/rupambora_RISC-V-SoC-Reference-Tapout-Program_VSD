# Week 1 — RTL Synthesis and Gate-Level Simulation (GLS)

---

This week covers the fundamentals of RTL design, synthesis, and gate-level simulation (GLS) using open-source tools like Icarus Verilog (iverilog), GTKWave, and Yosys with the Sky130 process design kit (PDK). The focus is on creating Verilog RTL designs, simulating them to verify functionality, and synthesizing them into gate-level netlists using standard cell libraries. Key topics include understanding timing libraries (.lib files), exploring hierarchical versus flat synthesis, and applying combinational and sequential optimizations to reduce area, power, and delay. The course also addresses common pitfalls like simulation-synthesis mismatches caused by improper coding practices, such as missing sensitivity lists or incorrect use of blocking and non-blocking assignments. Additionally, it introduces scalable coding techniques using for-loops and for-generate constructs for efficient hardware design. Practical labs reinforce these concepts by simulating designs, synthesizing them with Yosys, and verifying netlists through GLS, ensuring alignment between RTL and synthesized hardware behavior.

---
📑
## Table of Contents
- [Introduction](#introduction)
- [Day 1 — Verilog RTL Design and Synthesis](#day-1--verilog-rtl-design-and-synthesis)
  - [1.1 Introduction to iverilog](#11-introduction-to-iverilog)
  - [1.2 Labs using iverilog and GTKWave](#12-labs-using-iverilog-and-gtkwave)
  - [1.3 Introduction to Yosys and Logic Synthesis](#13-introduction-to-yosys-and-logic-synthesis)
  - [1.4 Labs using Yosys and SKY130 PDKs](#14-labs-using-yosys-and-sky130-pdks)
  - [Day 1 — Conclusion](#day-1--conclusion)
- [Day 2 — Timing Libraries and Coding Styles](#day-2--timing-libraries-and-coding-styles)
  - [2.1 Introduction to Timing .lib Files](#21-introduction-to-timing-lib-files)
  - [2.2 Hierarchical vs Flat Synthesis](#22-hierarchical-vs-flat-synthesis)
  - [2.3 Flop Coding Styles and Optimization](#23-flop-coding-styles-and-optimization)
  - [Day 2 — Conclusion](#day-2--conclusion)
- [Day 3 — Combinational and Sequential Optimizations](#day-3--combinational-and-sequential-optimizations)
  - [3.1 Introduction to Optimizations](#31-introduction-to-optimizations)
  - [3.2 Combinational Logic Optimizations](#32-combinational-logic-optimizations)
  - [3.3 Sequential Logic Optimizations](#33-sequential-logic-optimizations)
  - [3.4 Optimizations for Unused Outputs](#34-optimizations-for-unused-outputs)
  - [Day 3 — Conclusion](#day-3--conclusion)
- [Day 4 — GLS and Simulation Mismatches](#day-4--gls-and-simulation-mismatches)
  - [4.1 GLS, Blocking vs Non-Blocking, and Mismatches](#41-gls-blocking-vs-non-blocking-and-mismatches)
  - [4.2 Labs on GLS and Mismatch](#42-labs-on-gls-and-mismatch)
  - [4.3 Labs on Blocking Statement Mismatch](#43-labs-on-blocking-statement-mismatch)
  - [Day 4 — Conclusion](#day-4--conclusion)
- [Day 5 — Synthesis Optimizations](#day-5--synthesis-optimizations)
  - [5.1 If-Case Constructs](#51-if-case-constructs)
  - [5.2 Labs on Incomplete If-Case](#52-labs-on-incomplete-if-case)
  - [5.3 Labs on Incomplete Overlapping Case](#53-labs-on-incomplete-overlapping-case)
  - [5.4 For-Loop and For-Generate](#54-for-loop-and-for-generate)
  - [5.5 Labs on For-Loop and For-Generate](#55-labs-on-for-loop-and-for-generate)
  - [Day 5 — Conclusion](#day-5--conclusion)
- [Conclusion](#conclusion)

---

## Introduction
Week 1 focuses on **RTL synthesis and gate-level simulation (GLS)** using open-source tools.  

### Tools  
- **Simulation:** Icarus Verilog (iverilog) + GTKWave  
- **Synthesis:** Yosys with **SKY130 standard-cell library**  

### Goals  
1. Write and simulate RTL with iverilog, analyze waveforms in GTKWave.  
2. Synthesize RTL into gate-level netlist mapped to SKY130 cells using Yosys.  
3. Reuse same testbench for RTL and GLS to confirm functional equivalence.  
4. Understand `.lib` timing libraries, PVT corners, and their effect on synthesis.  
5. Compare **hierarchical vs flat synthesis** flows.  
6. Study **combinational and sequential optimizations** (constant propagation, Boolean simplification, retiming, flop removal).  
7. Identify **simulation-synthesis mismatches (SSM)** caused by bad coding practices.  
8. Apply correct Verilog coding styles for `if`/`case`, blocking vs non-blocking, resets, and loops.  
9. Use `for` (inside `always`) for logic evaluation and `for-generate` (outside `always`) for hardware replication.  

### Key Topics  
- **Event-driven simulation:** Simulator updates outputs only on input changes; VCD records transitions.  
- **Technology libraries (.lib):** Define timing, power, and area for cells at different PVT corners.  
- **Optimizations:** Automatic removal of dead logic, constant propagation, unused outputs; area and power reduction.  
- **Coding practices:**  
  - Use `always @(*)` for combinational logic.  
  - Use non-blocking (`<=`) for sequential logic.  
  - Add `default` in all `case` statements.  
  - Avoid incomplete branches to prevent inferred latches.  
- **Synthesis vs Simulation:** Netlist preserves RTL I/O, enabling direct testbench reuse. GLS validates real hardware behavior beyond RTL simulation.  

### Deliverables  
1. Pre vs post-synthesis waveforms (screenshots).  
2. Yosys synthesis statistics (area, cell count, optimization logs).  
3. Notes on observed optimizations and mismatches.  
---

## Day 1 — Verilog RTL Design and Synthesis

### 1.1 Introduction to iverilog

#### 1. What is a Simulator?
- A **simulator** checks if RTL design follows the specification.  
- RTL design = Verilog code implementing the spec.  
- In this course, we use **iverilog** (Icarus Verilog, open-source).  
- Simulator output = **VCD (Value Change Dump)** file.  

---

#### 2. Design vs Testbench

| Item | Description | I/O |
|------|-------------|-----|
| **Design (DUT)** | Verilog code that implements logic. Example: inverter, counter, ALU. | Has **primary inputs** and **primary outputs** |
| **Testbench (TB)** | Applies inputs (stimulus) to DUT and checks outputs. Instantiates the DUT. | **No primary inputs/outputs** |

- **Stimulus** = input values applied to DUT.  
- **Observer** = mechanism to watch outputs.  

---

#### 3. How Does a Simulator Work?
- Event-driven: **output is evaluated only when inputs change**.  
- No input change → no output evaluation.  
- Dumps results to **VCD file**.  
- VCD contains **only value changes**, not constant values.  

---

#### 4. Simulation Flow

```text
[ design.v ] + [ testbench.v ] → iverilog → [ simulation executable ]

executable (vvp) → generates → [ dump.vcd ]

dump.vcd → viewed with → GTKWave
````

##### Commands

```bash
# compile design and testbench
iverilog -o sim.out design.v tb.v

# run simulation
vvp sim.out

# open waveform in GTKWave
gtkwave dump.vcd
```

---

#### 5. Example Code

**Design (inverter)**

```verilog
// design.v
module inverter(input a, output y);
  assign y = ~a;
endmodule
```

**Testbench**

```verilog
// tb.v
module tb;
  reg a;
  wire y;

  inverter uut (.a(a), .y(y));

  initial begin
    $dumpfile("dump.vcd");      // VCD file
    $dumpvars(0, tb);           // dump all signals in tb

    a = 0; #10;
    a = 1; #10;
    a = 0; #10;

    $finish;
  end
endmodule
```

---

#### 6. Viewing Results

* Run simulation to generate **dump.vcd**.
* Open `dump.vcd` in **GTKWave**.
* Inspect waveforms:

  * Input toggles
  * Corresponding output response

📌 *Insert diagram of `Design ↔ Testbench ↔ Simulator ↔ VCD ↔ GTKWave` here*
📌 *Insert GTKWave screenshot of inverter waveform here*

---

#### 7. Key Points

* Simulator = tool to check spec compliance.
* DUT (design) has inputs/outputs.
* Testbench applies stimulus and observes outputs.
* iverilog generates VCD.
* GTKWave visualizes VCD as waveforms.

---

[🔼 Back to Table of Contents](#table-of-contents)

---

### 1.2 Labs using iverilog and GTKWave

#### 1. Environment Setup

- Create a working directory:
  ```bash
  mkdir VLSI && cd VLSI
  ```

* Clone **VSD flow**:

  ```bash
  git clone <vsdflow-repo-link>
  ```
* Clone **Sky130 RTL Design and Synthesis Workshop**:

  ```bash
  git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
  cd sky130RTLDesignAndSynthesisWorkshop
  ```

**Directory contents:**

```
mylib/
 ├── lib/            # Sky130 standard-cell .lib timing models
 └── VerilogModel/   # Standard-cell Verilog models
verilog_files/       # All lab RTL designs and testbenches
```

📌 *Insert screenshot of directory structure here*

---

#### 2. Simulation Flow with iVerilog

* Move to `verilog_files/`:

  ```bash
  cd verilog_files/
  ```
* Each design has a **1:1 matching testbench**:

  * Example: `goodmux.v` ↔ `tb_goodmux.v`
* Compile and simulate:

  ```bash
  # compile design + testbench
  iverilog -o sim_goodmux goodmux.v tb_goodmux.v

  # run executable → generates dump.vcd
  ./sim_goodmux
  ```

📌 *Insert terminal screenshot of compile + run here*

---

#### 3. Viewing Waveforms with GTKWave

* Launch GTKWave:

  ```bash
  gtkwave dump.vcd
  ```
* Steps inside GTKWave:

  * Expand **tb → uut** hierarchy.
  * Drag signals (I0, I1, sel, Y) to waveform pane.
  * Use **Zoom Fit** to view full simulation time.
  * Use zoom in/out for details.
  * Use forward/backward arrows to trace signal transitions.

📌 *Insert GTKWave screenshot with signals plotted here*

---

#### 4. Example: Multiplexer Verification

**Design:**

```verilog
module goodmux(input I0, input I1, input sel, output Y);
  assign Y = sel ? I1 : I0;
endmodule
```

**Testbench:**

```verilog
module tb_goodmux;
  reg I0, I1, sel;
  wire Y;

  goodmux uut(.I0(I0), .I1(I1), .sel(sel), .Y(Y));

  initial begin
    $dumpfile("dump.vcd");
    $dumpvars(0, tb_goodmux);

    I0 = 0; I1 = 0; sel = 0;
    #300 $finish;
  end

  always #75 sel = ~sel;
  always #50 I0  = ~I0;
  always #25 I1  = ~I1;
endmodule
```

**Expected behavior:**

* `sel=0` → Y follows I0
* `sel=1` → Y follows I1

**Waveform observations:**

* Output Y switches immediately when `sel` toggles.
* Matches mux functional specification.

📌 *Insert waveform screenshot highlighting sel, I0, I1, Y*

---

#### 5. Key Points

* `verilog_files/` contains both designs and matching testbenches.
* iVerilog compiles RTL + testbench → generates **VCD dump**.
* GTKWave is used for waveform-based functional verification.
* Testbench applies stimulus, but does not check output automatically.
* Verification is done by **observing signals** in GTKWave.

---

[🔼 Back to Table of Contents](#table-of-contents)

---

### 1.3 Introduction to Yosys and Logic Synthesis

#### 1. What is Yosys?

* **Yosys** = open-source RTL synthesizer.
* Converts **RTL (Verilog behavioral code)** → **gate-level netlist**.
* Uses **technology library (`.lib`)** for standard cells.
* Netlist = same design, but expressed as **instances of standard cells**.

---

#### 2. Yosys Flow

##### Inputs

* RTL design file (`design.v`).
* Standard cell library (`.lib`).

##### Commands

```tcl
# Load RTL
read_verilog design.v

# Load standard cells
read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib

# Run synthesis
synth -top top_module

# Write netlist
write_verilog netlist.v
```

##### Output

* Gate-level **netlist.v** containing standard cells (e.g., NAND, NOR, MUX, DFF).

---

#### 3. Netlist Verification

* **Primary inputs/outputs remain same** between RTL and netlist.
* Same **testbench** can be reused.
* Flow:

  1. Simulate netlist with `iverilog`.
  2. Generate VCD.
  3. View in GTKWave.
  4. Compare waveforms with RTL simulation.
* ✅ If waveforms match → synthesis is correct.

```bash
iverilog netlist.v tb.v VerilogModel/*.v -o sim_gls
./sim_gls
gtkwave dump.vcd
```

---

#### 4. What is Logic Synthesis?

* Converts **behavioral Verilog (RTL)** → **logic gates**.
* Steps:

  1. Parse RTL.
  2. Map operations (`assign`, `always`) → gates + flops.
  3. Optimize with constraints.
  4. Write gate-level Verilog (netlist).

**Flow:**

```
Specification → RTL (Verilog) → Yosys + .lib → Netlist (gates)
```

---

#### 5. Technology Library (`.lib`)

* `.lib` contains **standard cells**:

  * Combinational (AND, OR, NAND, NOR, INV, XOR).
  * Sequential (DFF, latch).
* Multiple flavors:

  * 2-input, 3-input, 4-input gates.
  * Slow / Medium / Fast versions.
* Universal gates (NAND/NOR) → sufficient for any Boolean logic.

---

#### 6. Timing Constraints

##### Setup Time (Max Delay Path)

* To avoid **setup violation**:

```
T_clk ≥ T_cqA + T_comb + T_setupB
```

* Max frequency:

```
F_clk = 1 / T_clk
```

* Use **fast cells** to reduce `T_comb`.

##### Hold Time (Min Delay Path)

* To avoid **hold violation**:

```
T_cqA + T_comb ≥ T_holdB
```

* Sometimes need **slow cells** to add delay.

---

#### 7. Fast vs Slow Cells

| Cell Type                | Delay | Area/Power | Use Case                  |
| ------------------------ | ----- | ---------- | ------------------------- |
| Fast (wide transistor)   | Low   | High       | Critical paths (setup)    |
| Slow (narrow transistor) | High  | Low        | Hold fixing, power saving |

* Too many fast cells → high area/power, possible hold violations.
* Too many slow cells → performance bottleneck.
* **Constraints** guide synthesizer to choose balance.

---

#### 8. Example Mapping

##### RTL

```verilog
module top(input clk, rst, a, b, sel, output reg y);
  wire d = sel ? b : a;
  always @(posedge clk or posedge rst)
    if (rst) y <= 0;
    else y <= d;
endmodule
```

##### Netlist (structural)

```verilog
mux2x1 u1 (.A(a), .B(b), .S(sel), .Y(net1));
dff    u2 (.D(net1), .CLK(clk), .RST(rst), .Q(y));
```

* `? :` → multiplexer cell.
* `always` → flip-flop cell.
* Connections made using `.lib` cells.

---

#### 9. Key Points

* Yosys maps **RTL → netlist** using `.lib`.
* Netlist I/O = RTL I/O → same testbench works.
* Verification = simulate netlist and compare with RTL.
* `.lib` has multiple cell flavors to balance:

  * **Setup** (performance).
  * **Hold** (reliability).
  * **Power/area**.

---

[🔼 Back to Table of Contents](#table-of-contents)

---

### 1.4 Labs using Yosys and SKY130 PDKs

#### **Lab 3: Synthesizing `good_mux`**

#### 1. Objective

* Use **Yosys** to synthesize RTL (`good_mux.v`) into a gate-level netlist using the **Sky130 standard cell library**.
* Verify that the synthesized design matches expected MUX behavior.
* Generate and analyze the structural Verilog netlist.

---

#### 2. Directory Setup

After cloning the repo:

```
├── mylib/
│   └── lib/
│       └── sky130_fd_sc_hd__tt_025C_1v80.lib   # Sky130 library
└── verilog_files/
    └── good_mux.v                             # RTL design
```

📌 *Placeholder for directory screenshot*

---

#### 3. Launch Yosys

```bash
yosys
```

* Installed as part of **VSD-Flow**.
* Prompt changes to `yosys>` when active.

📌 *Placeholder for Yosys prompt screenshot*

---

#### 4. Read Technology Library

```tcl
read_liberty -lib ../mylib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

**Library naming breakdown**:

| Segment  | Meaning        |
| -------- | -------------- |
| `sky130` | 130nm node     |
| `fd`     | foundry design |
| `sc`     | standard cell  |
| `hd`     | high density   |
| `tt`     | typical corner |
| `025C`   | 25 °C          |
| `1v80`   | 1.8 V          |

---

#### 5. Read RTL Design

```tcl
read_verilog good_mux.v
```

* Expected log:

```
Successfully finished Verilog frontend.
```

---

#### 6. Define Top Module

```tcl
synth -top good_mux
```

* Ensures synthesis runs on the correct RTL module.
* Required when design has multiple modules.

---

#### 7. Map RTL to Standard Cells

```tcl
abc -liberty ../mylib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Yosys log shows:

* Inputs: `i0`, `i1`, `sel`
* Output: `Y`
* Internal signals: none
* Cells inferred:

  * `sky130_fd_sc_hd__inv` (inverter)
  * `sky130_fd_sc_hd__nand2` (2-input NAND)
  * `sky130_fd_sc_hd__o21ai` (OR-AND-INVERT complex gate)

📌 *Placeholder for Yosys log snippet*

---

#### 8. Visualize Logic

```tcl
show
```

Graphical schematic (via Graphviz):

* `i0` → inverter → O21AI input
* `i1` + `sel` → NAND → O21AI input
* `sel` also drives O21AI directly
* O21AI output → `Y`

Boolean equation realized:

```
Y = (i0 · sel̅) + (i1 · sel)
```

→ matches 2:1 MUX.

📌 *Placeholder for schematic screenshot*

---

#### 9. Write Netlist

Default (verbose):

```tcl
write_verilog goodmux_netlist.v
```

Clean version:

```tcl
write_verilog -noattr goodmux_netlist.v
```

---

#### 10. Example Netlist (Simplified)

```verilog
module good_mux(i0, i1, sel, Y);
  input i0, i1, sel;
  output Y;
  wire net4, net5;

  sky130_fd_sc_hd__inv_1   u1 (.A(i0), .Y(net4));
  sky130_fd_sc_hd__nand2_1 u2 (.A(i1), .B(sel), .Y(net5));
  sky130_fd_sc_hd__o21ai_1 u3 (.A1(sel), .A2(net4), .B1(net5), .Y(Y));
endmodule
```

📌 *Placeholder for netlist screenshot*

---

#### 11. Netlist Analysis

* **Primary inputs**: `i0`, `i1`, `sel`
* **Internal wires**: e.g., `net4` (inverter output), `net5` (NAND output)
* **Primary output**: `Y`
* **Top module**: same as RTL (`good_mux`) → allows testbench reuse in GLS.

---

#### 12. Synthesis Flow Recap

```
read_liberty  →  read_verilog  →  synth -top  
   →  abc -liberty  →  show  →  write_verilog
```

---

#### 13. Key Points

* RTL mapped to **Sky130 standard cells**.
* Complex gates (e.g., `O21AI`) used for optimization.
* Netlist preserves module name and I/O.
* Functional equivalence confirmed with Boolean simplification.
* Always use `-noattr` for clean netlists.

---

### Day 1 — Conclusion

- **Simulator basics**: iverilog compiles RTL + testbench → generates `dump.vcd` for waveform analysis in GTKWave.  
- **Design vs Testbench**: DUT has I/O; testbench provides stimulus and observes outputs, no I/O.  
- **Event-driven simulation**: signals update only on input changes; VCD records only value transitions.  
- **Functional verification**: waveforms confirm DUT behavior (e.g., inverter, mux).  
- **Yosys introduction**: converts RTL to gate-level netlist using Sky130 `.lib`.  
- **Netlist properties**: same I/O as RTL, structural Verilog with standard cells (INV, NAND, MUX, DFF).  
- **GLS flow**: RTL testbench reused for netlist simulation; matching waveforms confirm synthesis correctness.  
- **Technology library role**: provides timing, cell variants (fast/slow), and enables optimization for setup/hold.  

---

[🔼 Back to Table of Contents](#table-of-contents)

---


## Day 2 — Timing Libraries and Coding Styles

### 2.1 Introduction to Timing .lib Files

#### 1. What is a `.lib` File?

* `.lib` (Liberty file) = timing, power, and area model of **standard cells**.
* Required for:

  * **Synthesis** (map RTL → gates, optimize area/power/timing).
  * **Static Timing Analysis (STA)**.
  * **Power estimation**.
* Contains:

  * **Cell types**: INV, NAND, NOR, AND, OR, AOI/OAI, DFF, MUX.
  * **Cell variants** (different drive strengths).
  * **Leakage power** (all input combinations).
  * **Propagation delay** (rise/fall).
  * **Area** in µm².
  * **Operating conditions** (PVT: Process, Voltage, Temperature).

⚠️ Do not edit `.lib` files. They are generated by foundry characterization.

---

#### 2. PVT Corners (Process, Voltage, Temperature)

`.lib` files are characterized at specific **PVT corners**.

| Parameter   | Symbol | Example        | Notes                                                       |
| ----------- | ------ | -------------- | ----------------------------------------------------------- |
| Process     | P      | TT / SS / FF   | Variation from fabrication (transistor dimensions, doping). |
| Voltage     | V      | 1.8 V, 0.9 V   | Affects delay and power. Higher V = faster, more power.     |
| Temperature | T      | -40°C to 125°C | Higher T = slower transistors, lower T = faster.            |

**Example filename:**

```
sky130_fd_sc_hd__tt_025C_1v80.lib
```

Breakdown:

* `sky130` → 130nm technology.
* `fd_sc_hd` → Foundry, standard cell, high density.
* `tt` → Typical process.
* `025C` → 25 °C.
* `1v80` → 1.80 V supply.

✅ Designers must check circuits across all PVT corners to ensure reliable silicon.

📌 *Screenshot placeholder: `.lib` header with PVT info*

---

#### 3. Library Header and Units

At the top of `.lib` you will see:

```liberty
library (sky130_fd_sc_hd__tt_025C_1v80) {
  technology (cmos);
  delay_model : "table_lookup";
  time_unit : "1ns";
  voltage_unit : "1V";
  power_unit : "1nW";
  current_unit : "1mA";
  resistance_unit : "1kohm";
  capacitance_unit : "1pf";
  operating_conditions("tt_025C_1v80") {
    process : 1.0;
    voltage : 1.8;
    temperature : 25.0;
  }
}
```

* **Delay model**: lookup table (delay depends on input slew + output load).
* **Units**: defined once, used throughout all cell definitions.

📌 *Screenshot placeholder: Units section of `.lib`*

---

#### 4. Standard Cell Definitions

Each cell starts with the keyword:

```liberty
cell (cell_name) {
   ...
}
```

* Example cells:

  * `and2_0`, `and2_2`, `and2_4` (2-input AND gate, different strengths).
  * `a2111o_1` (complex AND-OR gate).
* Each cell entry contains:

  1. **Leakage power** for all input combinations.
  2. **Area**.
  3. **Pins**: capacitance, direction, timing arcs.
  4. **Power** and **timing tables**.

📌 *Snippet placeholder: `.lib` showing a cell definition*

---

#### 5. Example — AND2 Variants

Sky130 provides multiple drive strengths for the same function.

| Cell     | Area (µm²) | Delay   | Power   | Notes              |
| -------- | ---------- | ------- | ------- | ------------------ |
| `and2_0` | 6.256      | Slowest | Lowest  | Narrow transistors |
| `and2_2` | 7.500      | Medium  | Medium  | Balanced           |
| `and2_4` | 8.750      | Fastest | Highest | Wider transistors  |

* **Wider transistors** → faster, but more area + higher power.
* **Smaller transistors** → slower, but area/power efficient.

📌 *Screenshot placeholder: `.lib` entries of and2_0, and2_2, and2_4*

---

#### 6. Verilog Models

Alongside `.lib`, there are **behavioral Verilog models** used for Gate-Level Simulation (GLS).

Example: `sky130_fd_sc_hd__and2_0.v`

```verilog
module and2_0 (input A, input B, output X);
  assign X = A & B;
endmodule
```

* `.lib` = timing, power, area.
* `.v` = logical functionality.
* Two variants:

  * **Without power pins**: for functional simulation.
  * **With power pins (_pp.v)**: for power-aware simulation.

📌 *Screenshot placeholder: Verilog model for `and2_0`*

---

#### 7. Key Points

1. `.lib` files = **technology view of cells** (timing, power, area).
2. PVT = **process, voltage, temperature** variations.
3. Tools (Yosys, STA engines) use `.lib` for optimization and analysis.
4. Cell flavors balance **area vs speed vs power**.
5. Always run **multi-corner STA** before tapeout.

---

[🔼 Back to Table of Contents](#table-of-contents)

---

### 2.2 Hierarchical vs Flat Synthesis

This lab explains **hierarchical synthesis**, **flat synthesis**, and **submodule-level synthesis** in Yosys.  
We use the file `multiple_modules.v` for demonstration.

---

#### 1. Example RTL: `multiple_modules.v`

- **sub_module1** → 2-input AND gate  
- **sub_module2** → 2-input OR gate  
- **Top module (`multiple_modules`)**:
  - Instantiates `U1` = sub_module1 (A, B → net1)  
  - Instantiates `U2` = sub_module2 (net1, C → Y)  

```verilog
// Submodule 1: AND gate
module sub_module1(input A, B, output Y);
  assign Y = A & B;
endmodule

// Submodule 2: OR gate
module sub_module2(input A, B, output Y);
  assign Y = A | B;
endmodule

// Top module
module multiple_modules(input A, B, C, output Y);
  wire net1;
  sub_module1 U1 (.A(A), .B(B), .Y(net1));
  sub_module2 U2 (.A(net1), .B(C), .Y(Y));
endmodule
````

---

#### 2. Hierarchical Synthesis

##### Commands

```tcl
yosys
read_liberty -lib ../mylib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog multiple_modules.v
synth -top multiple_modules
abc -liberty ../mylib/sky130_fd_sc_hd__tt_025C_1v80.lib
show multiple_modules
write_verilog -noattr multiple_modules_hier.v
```

##### Observations

* Hierarchy preserved:

  * `multiple_modules` instantiates `sub_module1` and `sub_module2`.
* Netlist contains 3 modules:
  `multiple_modules`, `sub_module1`, `sub_module2`.
* `show` displays U1, U2 blocks instead of gate-level details.

##### Gate Mapping

* Expected → OR gate in sub_module2.
* Actual → NAND + 2 inverters (De Morgan’s theorem).

Equation:

```
Y = A OR B  
Y = ~(~A & ~B) → NAND + input inverters
```

##### Reason

* CMOS libraries favor **NAND/NOR** because:

  * NAND → stacked NMOS (better mobility).
  * NOR/OR → stacked PMOS (slower, wider transistors, more area).
* Tools optimize for drive strength and area.

📌 *[Insert screenshot: hierarchical netlist view]*

---

#### 3. Flat Synthesis

##### Commands

```tcl
flatten
show multiple_modules
write_verilog -noattr multiple_modules_flat.v
```

##### Observations

* Hierarchies removed.
* Only one module → `multiple_modules`.
* AND, NAND, and inverters instantiated directly.
* No U1, U2 instances.

📌 *[Insert screenshot: flat netlist view]*

---

#### 4. Submodule-Level Synthesis

##### Commands

```tcl
read_liberty -lib ../mylib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog multiple_modules.v
synth -top sub_module1
abc -liberty ../mylib/sky130_fd_sc_hd__tt_025C_1v80.lib
show sub_module1
write_verilog -noattr submodule1_netlist.v
```

##### Observations

* Only `sub_module1` synthesized (AND gate).
* Other parts of design ignored.
* Useful for:

  1. **Reuse** → synthesize one multiplier, replicate for all instances.
  2. **Divide-and-Conquer** → break large SoCs into smaller blocks for better optimization.

📌 *[Insert schematic: submodule-only netlist]*

---

#### 5. Key Takeaways

| Mode               | Output Netlist              | Use Case                          |
| ------------------ | --------------------------- | --------------------------------- |
| **Hierarchical**   | Top + submodules preserved  | Debug, modular IP, readability    |
| **Flat**           | Single netlist, all gates   | Final optimization, signoff       |
| **Submodule-only** | Only selected module mapped | Repeated IPs, large design blocks |

* `synth -top <mod>` → control synthesis scope.
* `flatten` → remove hierarchy, merge into single-level netlist.
* Libraries optimize OR as NAND + inverters for CMOS efficiency.

---

#### 6. Deliverables (to include in repo)

* **Netlists**:

  * `netlist/multiple_modules_hier.v`
  * `netlist/multiple_modules_flat.v`
  * `netlist/submodule1_netlist.v`

* **Screenshots**:

  * `screenshots/hier_netlist.png`
  * `screenshots/flat_netlist.png`
  * `screenshots/submodule_netlist.png`

* **Reports**:

  * Area/cell stats (hier vs flat)
  * Note on De Morgan optimization
 
---

[🔼 Back to Table of Contents](#table-of-contents)

---

### 2.3 Flop Coding Styles and Optimization

#### 1 Why Flops?

- Combinational logic has **unequal path delays** → causes **glitches**.
- Example: `Y = (A & B) | C`  
  - If `A=0→1`, `B=0→1`, `C=1→0`, output may glitch low due to AND/OR delay mismatch.
- With multiple stages, glitches propagate → unstable outputs.
- **D flip-flops (flops):**
  - Capture data **only at clock edge**.
  - Block glitches between stages.
  - Provide **stable outputs** to next stage.

📌 **Key**: Place flops between combinational blocks to ensure stable timing.

📸 *[Insert glitch waveform screenshot here]*

---

#### 2 Initialization of Flops

- Flops must start from a **known value**.
- Initialization via **reset** or **set**.
- Types:
  - **Asynchronous Reset/Set** → works immediately, independent of clock.
  - **Synchronous Reset/Set** → effective only at clock edge.

---

#### 3 Asynchronous Reset Flop

- `Q` resets immediately when reset = 1.
- Sensitivity list: `clk` + `reset`.

##### RTL
```verilog
always @(posedge clk or posedge arst) begin
    if (arst)
        Q <= 1'b0;
    else
        Q <= D;
end
````

##### Behavior

* Reset high → `Q=0` immediately.
* Otherwise → `Q=D` on clock edge.

📸 *[Insert async reset waveform]*

---

#### 4 Asynchronous Set Flop

* `Q` sets to 1 immediately when set = 1.

##### RTL

```verilog
always @(posedge clk or posedge aset) begin
    if (aset)
        Q <= 1'b1;
    else
        Q <= D;
end
```

📸 *[Insert async set waveform]*

---

#### 5 Synchronous Reset Flop

* Reset works **only on clock edge**.
* Sensitivity list: `clk`.

##### RTL

```verilog
always @(posedge clk) begin
    if (srst)
        Q <= 1'b0;
    else
        Q <= D;
end
```

📸 *[Insert sync reset waveform]*

---

#### 6 Combined Async + Sync Reset

* Async reset has **highest priority**.
* Sync reset checked at clock edge if async not active.

##### RTL

```verilog
always @(posedge clk or posedge arst) begin
    if (arst)
        Q <= 1'b0;
    else if (srst)
        Q <= 1'b0;
    else
        Q <= D;
end
```

📸 *[Insert schematic of combined reset flop]*

---

#### 7 Lab: Simulation of Flops

##### Commands

```bash
# Async reset
iverilog dff_async_reset.v tb_dff_async_reset.v
./a.out
gtkwave tb_dff_async_reset.vcd

# Async set
iverilog dff_async_set.v tb_dff_async_set.v
./a.out
gtkwave tb_dff_async_set.vcd

# Sync reset
iverilog dff_sync_reset.v tb_dff_sync_reset.v
./a.out
gtkwave tb_dff_sync_reset.vcd
```

##### Expected Waveforms

* Async reset → Q drops immediately.
* Async set → Q rises immediately.
* Sync reset → Q changes only on clock edge.

📸 *[Insert GTKWave screenshots for each]*

---

#### 8 Synthesis of Flops (Yosys + SKY130)

##### Yosys Commands

```tcl
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_async_reset.v
synth -top dff_async_reset
dfflibmap -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

##### Observations

1. **Async Reset (active-high RTL vs active-low cell)**

   * Inverter inserted between reset and flop.
   * Netlist: `reset -> inverter -> flop`.

2. **Async Set**

   * Same as above if mismatch between RTL and library cell polarity.

3. **Sync Reset**

   * No sync-reset flop in library.
   * Implemented as:

     * Simple DFF
     * MUX/AND gate on `D` input.

   ```verilog
   wire d_mux = (srst) ? 1'b0 : D;
   sky130_fd_sc_hd__dfxtp_1 flop (.CLK(clk), .D(d_mux), .Q(Q));
   ```

4. **Combined Async + Sync Reset**

   * Async reset pin connected to flop.
   * Sync reset multiplexed into D-path.

📸 *[Insert Yosys schematics/netlist screenshots]*

---

#### 9 Interesting Optimizations

##### Multiplication by Powers of 2

* `Y = A * 2` → `{A, 1'b0}`
* `Y = A * 4` → `{A, 2'b00}`
* `Y = A * 8` → `{A, 3'b000}`
* **No cells inferred** → wiring only.

📸 *[Insert Yosys stat screenshot showing 0 cells]*

---

##### Multiplication by 9 (Special Case)

* `Y = A * 9` → `(A << 3) + A`.
* Optimized as `{A, A}` (concatenation).
* **No hardware required**, wiring only.

📸 *[Insert Yosys schematic showing concatenation]*

---

#### 10 Key Takeaways

| Flop Type   | Behavior             | Clock Dep. | Library Mapping             |
| ----------- | -------------------- | ---------- | --------------------------- |
| Async Reset | Immediate reset to 0 | No         | Flop + inverter (if needed) |
| Async Set   | Immediate set to 1   | No         | Flop + inverter (if needed) |
| Sync Reset  | Reset at clk edge    | Yes        | Flop + MUX/AND on D         |
| Combined    | Async > Sync > Data  | Mixed      | Flop + inverter + mux       |

* Flops block glitches and stabilize outputs.
* Always initialize flops using reset/set.
* Yosys maps to available library cells; if cell missing, logic is synthesized using gates.
* Multipliers by constants (powers of 2, 9 for 3-bit input) optimized to wiring.

---

#### 11 Placeholders for Submission

* ✅ Verilog code (src/)
* ✅ Testbenches (tb/)
* 📸 Simulation waveforms (waves/)
* 📸 Yosys schematics & stat reports (screenshots/)
* 📝 Observations (reports/)

### Day 2 — Conclusion

- `.lib` files are the **technology view of standard cells** containing timing, power, and area data.  
- PVT (Process, Voltage, Temperature) corners define the **operating conditions** of libraries.  
- Hierarchical synthesis preserves module boundaries; flat synthesis merges all logic; submodule-level synthesis allows **block-level optimization and reuse**.  
- CMOS libraries prefer NAND/NOR implementations → tools may map OR to NAND + inverters for efficiency.  
- Flip-flops are required to **block glitches and stabilize signals** across stages.  
- Async resets/sets act immediately; sync resets work only at clock edge; combined flops give priority to async reset.  
- Yosys maps flops to available SKY130 cells; missing features (like sync reset) are synthesized using mux/logic around DFF.  
- Multiplication by constants (powers of 2, 9 for 3-bit case) is optimized to **pure wiring** without extra gates.  

---

[🔼 Back to Table of Contents](#table-of-contents)

---

## Day 3 — Combinational and Sequential Optimizations

### 3.1 Introduction to Optimizations

In digital design, **logic optimization** is applied to make circuits efficient in terms of **area, power, and timing**.  
Optimizations are grouped into two categories:

- **Combinational logic optimizations**
- **Sequential logic optimizations**

Synthesis tools (like Yosys, Synopsys DC) perform many of these automatically.

---

#### 🔹 Why Optimize?
- Reduce **transistor count** → save silicon area.
- Reduce **power consumption**.
- Improve **performance** by reducing delay.

---

### A. Combinational Logic Optimizations

#### 1. Constant Propagation
If an input is tied to a constant (`0` or `1`), the logic can collapse.

##### Example
Original expression:
```

Y = (A · B + C)'

```
If `A = 0`:
```

Y = (0 · B + C)' = (C)' = C'

````
Result:  
- Gate network reduces to a **single inverter**.

##### Hardware Cost
- AOI gate realization: **6 MOS transistors**  
- Optimized inverter: **2 MOS transistors**  
- **Savings**: ~67% in area and power.

📌 *[Insert schematic screenshot before/after]*

---

#### 2. Boolean Logic Simplification
Large boolean expressions can be reduced using **K-map** or **Quine-McCluskey**.

##### Example
```verilog
assign y = a ? (b ? c : (c ? a : 1'b0)) : ~c;
````

**Simplification steps** (tool or manual):

1. Break down ternary muxes into equations.
2. Simplify:

   ```
   Y = A̅·C̅ + A·C
   ```
3. Final optimized form:

   ```
   Y = A ⊕ C
   ```

Result:

* Complicated nested mux logic → **single XOR gate**.
* Shorter logic cone, faster evaluation, fewer cells.

📌 *[Insert Yosys schematic/log screenshot]*

---

### B. Sequential Logic Optimizations

#### 1. Sequential Constant Propagation

If the **D input** of a flop is tied to a constant:

* With reset → `Q = 0`
* Without reset → `Q = 0`
* **Q is always constant**.

Result:

* Flop + downstream logic can be **removed and replaced by constant driver**.

##### Example

```verilog
always @(posedge clk or posedge rst)
  if (rst) Q <= 1'b0;
  else     Q <= 1'b0;
```

* Here, `Q` is always 0 → can be optimized away.

📌 *[Insert schematic/waveform]*

---

#### 2. Non-Optimizable Case (Async Set)

```verilog
always @(posedge clk or posedge set)
  if (set) Q <= 1'b1;
  else     Q <= 1'b0;
```

* Behavior:

  * `set=1` → Q=1 immediately (async).
  * `set=0` → Q=0 only after next clock (sync).
* Q is **not constant**.
* This flop **cannot be removed**.

📌 *[Insert waveform showing async vs sync behavior]*

---

### C. Advanced Sequential Optimizations (Theory Only)

These are used in **industrial flows**, but not covered in lab.

| Technique              | Description                     | Purpose                              |
| ---------------------- | ------------------------------- | ------------------------------------ |
| **State Optimization** | Remove unused FSM states.       | Reduce state bits, smaller FSM.      |
| **Logic Cloning**      | Duplicate registers near sinks. | Reduce routing delay in large chips. |
| **Retiming**           | Move logic across flops.        | Balance delays, increase f_max.      |

---

#### 1. Retiming Example

* Original: Path delays = 5ns and 2ns → max freq = 200 MHz.
* After retiming: 4ns and 3ns → max freq = 250 MHz.
* **Uses slack** to increase performance.

📌 *[Insert retiming diagram]*

---

#### 2. Logic Cloning Example

* Flop A drives two distant flops B and C.
* Routing delay too large.
* Solution: Clone flop A near B and C.
* Reduces long interconnect delay.

📌 *[Insert cloning floorplan diagram]*

---

### Key Takeaways

* **Constant propagation** → replaces tied inputs with constants.
* **Boolean simplification** → collapses complex expressions.
* **Sequential constant propagation** → removes useless flops.
* **Async set/reset cases** often prevent optimization.
* **Advanced techniques** (state optimization, retiming, cloning) improve performance but need physical context.

---

[🔼 Back to Table of Contents](#table-of-contents)

---

### 3.2 Combinational Logic Optimizations

#### 1. Objective
- Learn how Yosys simplifies combinational RTL using:
  - **Constant propagation**
  - **Boolean simplification**
  - **Dead logic removal (`opt_clean -purge`)**
- Verify that mux-based RTL reduces to minimal **AND/OR gates**.

---

#### 2. Files Used
- `opt_check.v`
- `opt_check2.v`
- `opt_check3.v`
- `opt_check4.v` *(exercise)*
- `multiple_modules_opt.v` *(exercise, hierarchical)*

---

#### 3. Yosys Optimization Flow
Basic flow used for all files:

```tcl
yosys
read_liberty -lib ../mylib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog <design>.v
synth -top <module_name>
opt_clean -purge        # removes constants + unused cells
abc -liberty ../mylib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog -noattr <module_name>_opt.v
````

**Notes:**

* Always run `opt_clean -purge` after synthesis.
* Use `flatten` before optimization when the design has multiple modules.

---

#### 4. Case Studies

##### 4.1 `opt_check.v`

**RTL:**

```verilog
assign y = a ? b : 1'b0;
```

**Simplification:**

```
Y = A·B + A̅·0 = A·B
```

**Expected:** 2-input AND gate.
**Result:** `sky130_fd_sc_hd__and2_0`

📌 *Schematic placeholder → AND gate*

---

##### 4.2 `opt_check2.v`

**RTL:**

```verilog
assign y = a ? 1'b1 : b;
```

**Simplification:**

```
Y = A·1 + A̅·B = A + B
```

**Identity Used:** Absorption Law (`A + A'B = A + B`).

**Expected:** OR gate.
**Mapped Result:** NAND + 2 inverters (De Morgan’s theorem).
Reason: CMOS prefers NAND/NOR cells (fewer stacked PMOS).

📌 *Schematic placeholder → NAND + inverters implementing OR*

---

##### 4.3 `opt_check3.v`

**RTL:**

```verilog
assign y = a ? (c ? b : 1'b0) : 1'b0;
```

**Simplification:**

```
Inner mux: (C ? B : 0) = B·C
Outer mux: (A ? (B·C) : 0) = A·B·C
```

**Expected:** 3-input AND gate.
**Result:** `sky130_fd_sc_hd__and3_0`

📌 *Schematic placeholder → AND3 gate*

---

##### 4.4 `opt_check4.v` (Exercise)

* Run the same flow.
* Verify optimized gate-level result.
* Insert schematic screenshot.

---

##### 4.5 `multiple_modules_opt.v` (Exercise)

**Special Handling:**

```tcl
flatten
opt_clean -purge
abc -liberty ../mylib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

* Flattening required → allows optimizer to see across module boundaries.
* Without flatten → redundant logic may remain.

📌 *Schematic placeholder → flattened optimized netlist*

---

#### 5. Results Summary

| Module                 | RTL Expression        | Simplified Logic | Gate Mapped          |
| ---------------------- | --------------------- | ---------------- | -------------------- |
| `opt_check.v`          | `a ? b : 1'b0`        | `A·B`            | AND2                 |
| `opt_check2.v`         | `a ? 1'b1 : b`        | `A + B`          | OR2 (NAND+inverters) |
| `opt_check3.v`         | `a ? (c ? b : 0) : 0` | `A·B·C`          | AND3                 |
| `opt_check4.v`         | — exercise —          | TBD              | TBD                  |
| `multiple_modules_opt` | hierarchical modules  | Flatten + reduce | Optimized flat gates |

---

#### 6. Key Learnings

* `opt_clean -purge` is essential for **constant removal** and **cell cleanup**.
* Boolean simplification directly maps RTL mux into single gates.
* OR gates are not mapped directly in SKY130 → instead realized using NAND + inverters.
* Flattening is mandatory for multi-module optimization.
* Optimized netlists use fewer cells and reduce area.

---

[🔼 Back to Table of Contents](#table-of-contents)

---

### 3.3 Sequential Logic Optimizations

#### 1. Objective
- Learn how synthesis tools optimize **flip-flops with constant inputs**.
- Distinguish between:
  - **Sequential constants** → flop output depends on clock (flop kept).
  - **Combinational constants** → flop output is always constant (flop removed).
- Verify with **simulation** and **Yosys synthesis**.

---

#### 2. Files
- Source: `dff_const1.v`, `dff_const2.v`, `dff_const3.v`  
- Exercises: `dff_const4.v`, `dff_const5.v`  
- Testbenches: `tb_dff_const*.v`

---

#### 3. Yosys Flow
```tcl
yosys
read_liberty -lib ../mylib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_constX.v
synth -top dff_constX
dfflibmap -liberty ../mylib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../mylib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog -noattr dff_constX_opt.v
````

##### Notes

* `dfflibmap` is required → maps inferred flops to standard cells.
* SKY130 flops use **active-low reset/set** → if RTL uses active-high, tool inserts inverter.

---

#### 4. Case Studies

##### 4.1 `dff_const1.v`

```verilog
always @(posedge clk or posedge reset)
  if (reset) q <= 0;
  else       q <= 1;
```

* **Behavior**:

  * Reset → `q=0`.
  * After reset deassert → `q=1` only on next clock edge.
* **Result**: Flop **preserved** (q is not constant).

📌 *Waveform*: `q` waits for clock.
📌 *Synthesis*: 1 DFF + inverter on reset path (library expects active-low).

---

##### 4.2 `dff_const2.v`

```verilog
always @(posedge clk or posedge reset)
  if (reset) q <= 1;
  else       q <= 1;
```

* **Behavior**:

  * Reset → `q=1`.
  * Normal operation → `q=1`.
* **Result**: `q` is constant `1`. Flop **optimized away**.

📌 *Waveform*: `q` always high.
📌 *Synthesis*: no flop, direct tie to `1`.

---

##### 4.3 `dff_const3.v`

```verilog
always @(posedge clk or posedge reset) begin
  if (reset) begin
    q1 <= 0;
    q  <= 1;
  end else begin
    q1 <= 1;
    q  <= q1;
  end
end
```

* **Behavior**:

  * `q1`: 0 on reset, then 1 after next clock.
  * `q`: 1 on reset, then follows `q1`.
  * Effect: `q` dips to 0 for exactly one clock cycle.
* **Result**: Both flops **preserved**.

📌 *Waveform*: one-cycle glitch on `q`.
📌 *Synthesis*: 2 DFFs, inverters on set/reset.

---

#### 5. Summary

| Design     | Behavior                     | Optimization |
| ---------- | ---------------------------- | ------------ |
| dff_const1 | q syncs with clk after reset | Flop kept    |
| dff_const2 | q = 1 always                 | Flop removed |
| dff_const3 | two flops, one-cycle glitch  | Both kept    |
| dff_const4 | exercise                     | TBD          |
| dff_const5 | exercise                     | TBD          |

---

#### 6. Key Points

* A flop is **removed** only if its output is a true constant.
* If output depends on **clock edge**, flop must remain.
* Active-high resets/sets → synthesis inserts inverter (SKY130 cells use active-low).
* Always confirm with **simulation + yosys statistics**.

---

[🔼 Back to Table of Contents](#table-of-contents)

---

### 3.4 Optimizations for Unused Outputs

#### 1. Concept
- In synthesis, **any signal not driving a primary output is removed**.  
- This applies to:
  - Unused **register bits**  
  - Unused **combinational logic feeding them**  
- Tools perform **fanout tracing**: if a node does not contribute to outputs, it is optimized out.

---

#### 2. Case Study A — `counter_opt.v`

##### RTL
```verilog
module counter_opt(input clk, input rst, output q);
  reg [2:0] count;
  always @(posedge clk or posedge rst)
    if (rst) count <= 3'b000;
    else     count <= count + 1;
  assign q = count[0];  // only LSB used
endmodule
````

##### Behavior

* 3-bit up counter → rolls over at 7.
* Output `q` = LSB (`count[0]`).
* `count[1]` and `count[2]` unused.

##### Synthesis Result

* **1 DFF kept** (`count[0]`).
* **2 DFFs removed** (`count[1:2]`).
* Incrementer logic also removed.
* Remaining circuit: **toggle flop** → `D = ~Q`.
* Area/power minimized.

---

#### 3. Case Study B — `counter_opt2.v`

##### RTL

```verilog
module counter_opt2(input clk, input rst, output q);
  reg [2:0] count;
  always @(posedge clk or posedge rst)
    if (rst) count <= 3'b000;
    else     count <= count + 1;
  assign q = (count == 3'b100);  // uses all bits
endmodule
```

##### Behavior

* Output `q` = 1 when `count = 4 (100)`.
* Depends on all 3 bits.

##### Synthesis Result

* **3 DFFs kept** (`count[0]`, `count[1]`, `count[2]`).
* Incrementer logic preserved (adder-like).
* Comparator implemented as:

  ```
  q = count[2] & ~count[1] & ~count[0]
  ```

  Mapped to 3-input NOR with one inverted input.

---

#### 4. Yosys Flow

```tcl
read_liberty -lib ../mylib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog counter_opt.v       # or counter_opt2.v
synth -top counter_opt
dfflibmap -liberty ../mylib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../mylib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

---

#### 5. Hardware Comparison

| Design         | Flops Kept | Comb. Logic | Notes                             |
| -------------- | ---------- | ----------- | --------------------------------- |
| `counter_opt`  | 1          | 1 inverter  | Toggle flop, unused logic removed |
| `counter_opt2` | 3          | Adder + NOR | All flops and logic preserved     |

---

#### 6. Key Points

* **Rule**: Only logic in the **transitive fanout of primary outputs** is kept.
* Unused register bits and their feeding logic are removed.
* This reduces **area and power**.
* Always check synthesis reports and netlists to confirm optimizations.

---

### Day 3 — Conclusion

- **Combinational optimizations**: constant propagation, Boolean simplification, and dead logic removal reduce gate count; flattening enables cross-module cleanup.  
- **Sequential optimizations**: flops with constant outputs are removed; clock-dependent flops are preserved; async set/reset may block optimization.  
- **Unused outputs**: only logic driving primary outputs is kept; unused flops and their feeding logic are pruned.  
- **Technology mapping**: Yosys uses `dfflibmap` + `abc` to map optimized logic and flops to SKY130 standard cells.  
- **Advanced methods** (theory): state optimization, retiming, and logic cloning improve performance and timing closure in large designs.  
- **Key outcome**: synthesis aggressively minimizes area/power while preserving functional intent, confirmed by GLS.  

---

[🔼 Back to Table of Contents](#table-of-contents)

---

## Day 4 — GLS and Simulation Mismatches

### 4.1 GLS, Blocking vs Non-Blocking, and Mismatches

#### What is GLS?

* **Gate-Level Simulation (GLS)** = simulate the **synthesized netlist** with the **same testbench** used for RTL.
* The **netlist** is the RTL translated into **standard cells** (e.g., `AND2X1`, `DFF`).
* I/O ports are identical → testbench works without change.

---

#### Why Run GLS?

* Verify **logical correctness** after synthesis.
* Catch **synthesis-simulation mismatches** (SSM).
* Validate **timing** if delay-annotated models are used (`SDF`).

⚠️ In this lab we only use **functional GLS** (no timing).

---

#### GLS Flow (with iVerilog)

```bash
# Compile: netlist + testbench + std-cell models
iverilog -o gls_sim \
    design_netlist.v \
    tb_design.v \
    ../mylib/VerilogModel/*.v

# Run simulation → generates dump.vcd
vvp gls_sim

# View waveform
gtkwave dump.vcd
```

* **Extra step vs RTL sim**: include **std-cell Verilog models** (so simulator knows what gates mean).

---

#### Synthesis-Simulation Mismatches (SSM)

##### Causes

1. **Missing sensitivity list**
2. **Blocking (`=`) vs Non-blocking (`<=`) misuse**
3. **Non-standard Verilog coding**

---

##### Missing Sensitivity List

**Buggy MUX RTL:**

```verilog
// ❌ Wrong: ignores I0, I1 changes
always @(sel) begin
  if (sel) Y = I1;
  else     Y = I0;
end
```

* **Problem**: `Y` updates **only** when `sel` changes.
* Simulation looks latch-like.
* Synthesis builds correct MUX. → **Mismatch**.

**Fix:**

```verilog
// ✅ Correct: reacts to all inputs
always @(*) begin
  if (sel) Y = I1;
  else     Y = I0;
end
```

---

##### Blocking vs Non-Blocking Assignments

**1. Blocking (`=`)**

* Executes **in order**, like C.
* Risk: order-dependent bugs in sequential logic.

Example — shift register:

```verilog
// ❌ Wrong: only one flop
always @(posedge clk) begin
  q0 = D;
  q  = q0;   // q0 already got D
end
```

---

**2. Non-Blocking (`<=`)**

* Executes **in parallel**.
* Use for sequential circuits.

```verilog
// ✅ Correct: two flops
always @(posedge clk) begin
  q0 <= D;
  q  <= q0;
end
```

---

##### Combinational Pitfall with Blocking

```verilog
// ❌ Wrong: Y uses old Q0
always @(*) begin
  Y  = Q0 & C;
  Q0 = A | B;
end
```

* Simulation: `Y` uses **stale Q0**, looks like a flop.
* Synthesis: no flop → mismatch.
* Fix: reorder, or use non-blocking.

---

#### Key Rules

* **Sequential logic** → always use **non-blocking (`<=`)**.
* **Combinational logic** → blocking is ok, but watch ordering.
* Always use `always @(*)` for combinational sensitivity lists.
* Run **GLS** to ensure netlist matches RTL intent.

---

[🔼 Back to Table of Contents](#table-of-contents)

---

### 4.2 Labs on GLS and Mismatch

#### 1. Objective

* Run **Gate-Level Simulation (GLS)** using netlists and standard-cell models.
* Compare **RTL simulation vs GLS**.
* Show a **synthesis-simulation mismatch (SSM)** caused by missing sensitivity list.

---

#### 2. Setup

To run GLS, we need:

* **Netlist** (`*_net.v`) from synthesis.
* **Testbench** (same as RTL sim).
* **Standard-cell Verilog models** (`../mylib/verilog_model/*.v`).

##### Commands

**RTL Simulation**

```bash
iverilog design.v tb_design.v
./a.out
gtkwave dump.vcd
```

**Synthesis with Yosys**

```tcl
read_liberty -lib ../mylib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog design.v
synth -top <top_module>
abc -liberty ../mylib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr design_net.v
```

**GLS**

```bash
iverilog -o gls_sim \
  ../mylib/verilog_model/primitives.v \
  ../mylib/verilog_model/sky130_fd_sc_hd.v \
  design_net.v tb_design.v
./gls_sim
gtkwave dump.vcd
```

---

#### 3. Case 1 — Correct MUX (Reference)

**RTL Code**

```verilog
module ternary_operator_mux(input I0, I1, sel, output Y);
  assign Y = sel ? I1 : I0;
endmodule
```

**RTL Simulation**

* `Y = I0` when `sel=0`.
* `Y = I1` when `sel=1`.
* Hierarchy shows only `UUT` (no standard cells).

**Synthesis**

* Netlist maps into NAND, INV, OAI cells.
* Boolean equation:

  ```
  Y = (~sel & I0) | (sel & I1)
  ```

**GLS**

* Gate instances like `_6`, `_7`, `_8` appear.
* Behavior matches RTL.
* No mismatch.

📸 *Insert waveform screenshots here*

---

#### 4. Case 2 — Bad MUX (Mismatch Example)

**RTL Code (Buggy)**

```verilog
module badmux(input I0, I1, sel, output reg Y);
  always @(sel) begin
    if (sel) Y = I1;
    else     Y = I0;
  end
endmodule
```

**Problem**

* Sensitivity list has only `sel`.
* `Y` does not update when `I0` or `I1` change.
* Simulation looks like latch/flop behavior.

**RTL Simulation**

* `Y` updates only when `sel` toggles.
* Ignores activity on `I0` and `I1`.

**Synthesis**

* Yosys infers correct mux logic (ignores sensitivity list).
* Netlist is same as `ternary_operator_mux`.

**GLS**

* Behavior matches real mux:

  * `sel=0` → `Y=I0`
  * `sel=1` → `Y=I1`
* Confirms mismatch with RTL sim.

📸 *Insert waveform screenshots here*

---

#### 5. Comparison

| Design                 | RTL Sim Result      | GLS Result  | Notes                            |
| ---------------------- | ------------------- | ----------- | -------------------------------- |
| `ternary_operator_mux` | Correct mux         | Correct mux | No mismatch                      |
| `badmux` (buggy)       | Latch-like behavior | Correct mux | Sensitivity list missing `I0,I1` |

---

#### 6. Key Points

* RTL simulators depend on **sensitivity lists**.
* Synthesis ignores them; hardware is inferred correctly.
* **Mismatch** occurs when RTL sim ≠ synthesized hardware.
* Always use `always @(*)` for combinational logic.
* GLS is mandatory to validate post-synthesis behavior.

---

[🔼 Back to Table of Contents](#table-of-contents)

---

### 4.3 Labs on Blocking Statement Mismatch

#### 1. Objective

* Show how **blocking assignments (`=`)** in combinational logic can cause **synthesis-simulation mismatch (SSM)**.
* Compare **RTL simulation** vs **Gate-Level Simulation (GLS)**.
* Demonstrate why GLS must always be trusted over RTL sim.

---

#### 2. Target Logic

We want to implement:

```
D = (A | B) & C
```

This should be pure combinational logic.

---

#### 3. RTL Implementation (Buggy)

```verilog
module blocking_caveat (input A, B, C, output reg D);
    reg X;
    always @(*) begin
        D = X & C;   // uses old value of X
        X = A | B;   // X updated after D
    end
endmodule
```

* Problem: Blocking assignments execute **sequentially**.
* `D` is computed **before** `X` is updated, so `D` uses the **previous value of X**.
* RTL simulation shows **flop-like behavior** even though no flop exists.

---

#### 4. RTL Simulation

**Command:**

```bash
iverilog blocking_caveat.v tb_blocking_caveat.v
./a.out
gtkwave dump.vcd
```

**Observation:**

* Output `D` sometimes goes high when both `A=0` and `B=0`.
* This happens because `D` uses **stale X** from the previous cycle.
* RTL sim incorrectly shows a **registered version of X**.

📌 *Screenshot placeholder*: `screenshots/blocking_rtl_sim.png`

---

#### 5. Synthesis

**Yosys flow:**

```tcl
read_liberty -lib ../mylib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog blocking_caveat.v
synth -top blocking_caveat
abc -liberty ../mylib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr blocking_caveat_net.v
```

**Netlist result:**

* Simple **OR2 + AND2** gate network.
* No flop, no latch.
* Implements correct combinational logic: `D = (A | B) & C`.

📌 *Screenshot placeholder*: `screenshots/blocking_netlist.png`

---

#### 6. Gate-Level Simulation (GLS)

**Command:**

```bash
iverilog -o gls_sim \
  ../mylib/verilog_model/primitives.v \
  ../mylib/verilog_model/sky130_fd_sc_hd.v \
  blocking_caveat_net.v tb_blocking_caveat.v
./gls_sim
gtkwave dump.vcd
```

**Observation:**

* GLS waveform matches Boolean expectation.
* Example: `A=0, B=0, C=1 → D=0` (correct).
* No stale value effect.

📌 *Screenshot placeholder*: `screenshots/blocking_gls.png`

---

#### 7. Comparison

| Inputs (A,B,C) | Expected D | RTL Sim D (Buggy) | GLS D (Correct) |
| -------------- | ---------- | ----------------- | --------------- |
| 0,0,1          | 0          | 1 ❌               | 0 ✅             |
| 1,0,1          | 1          | 0 ❌               | 1 ✅             |

* RTL sim misbehaves because it evaluates in **execution order**, not logic order.
* GLS shows correct behavior because synthesis builds gates, not sequential semantics.

📌 *Screenshot placeholder*: `screenshots/blocking_comparison.png`

---

#### 8. Root Cause

* **Blocking assignments** update variables immediately, in sequence.
* In combinational always blocks, this can lead to **simulation artifacts** where values appear flopped.
* Synthesis tools, however, see only the logic equation → **pure combinational netlist**.
* Mismatch arises between RTL sim and GLS.

---

#### 9. Fixes

##### Option A: Use `wire + assign` (preferred)

```verilog
wire X = A | B;
assign D = X & C;
```

##### Option B: Reorder blocking statements

```verilog
always @(*) begin
    X = A | B;
    D = X & C;
end
```

⚠️ Risky — requires manual ordering, easy to break.

##### Option C: Use non-blocking (`<=`) — only for sequential logic

Not recommended for pure combinational paths.

---

#### 10. Key Takeaways

* Blocking assignments can cause **false sequential behavior** in simulation.
* Always prefer `wire` + `assign` for combinational logic.
* Use **non-blocking (`<=`)** for sequential (flop-based) logic.
* **GLS is the reference** — it matches hardware behavior.
* Run GLS to catch mismatches **before tape-out**.

---

### Day 4 — Conclusion

- **GLS is mandatory**: validates synthesized netlist against RTL using the same testbench.  
- **Extra step in GLS**: include standard-cell Verilog models (`sky130_fd_sc_hd.v`, `primitives.v`).  
- **SSM (Simulation-Synthesis Mismatch)** occurs due to:  
  - Missing sensitivity lists in combinational blocks.  
  - Misuse of blocking (`=`) in sequential logic.  
  - Ordering issues in combinational always blocks.  
- **RTL simulator vs synthesis**:  
  - RTL sim depends on coding style (sensitivity list, assignment type).  
  - Synthesis infers hardware equations, ignores sensitivity list bugs.  
- **Key rules**:  
  - Use `always @(*)` for all combinational logic.  
  - Use non-blocking (`<=`) for sequential logic.  
  - Avoid blocking assignment ordering pitfalls in combinational blocks.  
- **Trust GLS over RTL sim**: GLS reflects real hardware mapped to standard cells.  

---

[🔼 Back to Table of Contents](#table-of-contents)

---

## Day 5 — Synthesis Optimizations

### 5.1 If-Case Constructs

#### 1. If–Else Constructs → Priority Logic

##### 1.1 Priority Behavior
```verilog
always @(*) begin
  if (cond1)      Y = C1;  // highest priority
  else if (cond2) Y = C2;
  else if (cond3) Y = C3;
  else            Y = E;   // lowest priority
end
````

* Hardware: **Priority MUX chain**
* Only the **first true condition** executes
* Common for: **priority encoders**, **control signals**

📌 *[Placeholder: `screenshots/if_priority_mux.png`]*

---

##### 1.2 Danger: Incomplete If → Inferred Latch

```verilog
always @(*) begin
  if (cond1)      Y = A;
  else if (cond2) Y = B;
  // no else → latch inferred
end
```

* If `cond1=0` and `cond2=0` → tool must hold previous `Y`
* Synthesizer inserts a **transparent latch** (unwanted in combinational logic)

📌 *[Placeholder: `screenshots/inferred_latch_if.png`]*

---

##### 1.3 Exception: Sequential Logic (Safe Incomplete If)

```verilog
always @(posedge clk or posedge rst) begin
  if (rst)       count <= 3'b000;
  else if (en)   count <= count + 1;
  // no else → counter holds value (intended behavior)
end
```

* Here, latch is **intended** because the flop naturally stores state
* Used in counters, registers, FSM state holding

📌 *[Placeholder: `screenshots/counter_flop.png`]*

---

#### 2. Case Constructs → Multiplexers

##### 2.1 Syntax

```verilog
always @(*) begin
  case (sel)
    2'b00: Y = C1;
    2'b01: Y = C2;
    2'b10: Y = C3;
    2'b11: Y = C4;
    default: Y = 0;  // ✅ recommended
  endcase
end
```

* Hardware: **4:1 MUX**
* Each branch is parallel, **no priority**

📌 *[Placeholder: `screenshots/case_mux.png`]*

---

#### 3. Case Pitfalls

##### 3.1 Incomplete Case

```verilog
case (sel) // sel is 2-bit
  2'b00: Y = A;
  2'b01: Y = B;
  // missing 2'b10, 2'b11 → latch
endcase
```

✅ **Fix**: Always add `default`.

---

##### 3.2 Partial Assignments

```verilog
case (sel)
  2'b00: begin X = A; Y = B; end
  2'b01: begin X = C; end     // ❌ Y unassigned → latch
  default: begin X = D; Y = B; end
endcase
```

✅ **Fix**: Assign all outputs in every branch.

---

##### 3.3 Overlapping Cases

```verilog
case (sel)
  2'b10: Y = C1;
  2'b1?: Y = C2;  // overlaps with 2'b10 & 2'b11
endcase
```

* For `sel=2'b10`: **both branches match** → order-dependent, unpredictable
* Unlike `if-else`, `case` does **not exit early**

✅ **Fix**: Ensure mutually exclusive case items.

📌 *[Placeholder: `screenshots/case_overlap.png`]*

---

#### 4. If vs Case Summary

| Construct | Hardware Inferred | Priority? | Risks                        | Best Practice                     |
| --------- | ----------------- | --------- | ---------------------------- | --------------------------------- |
| If–Else   | Priority MUX      | Yes       | Incomplete → latch           | Always close with `else`          |
| Case      | Multiplexer       | No        | Incomplete, partial, overlap | Use `default`, assign all outputs |

---

#### 5. Best Practices

1. **If–Else** = use when **priority** matters
2. **Case** = use for **parallel selection**
3. **Always add `else` or `default`**
4. **Assign all outputs in all branches**
5. **Avoid overlapping cases**

---

[🔼 Back to Table of Contents](#table-of-contents)

---

### 5.2 Labs on Incomplete If-Case

#### Objective
- Show how **incomplete `if` constructs** create **inferred latches**.  
- Compare **RTL simulation vs synthesis behavior**.  
- Verify results with **Icarus Verilog (simulation)** and **Yosys + SKY130 PDK (synthesis)**.

---

#### 1. Lab: Incomplete If (Case 1)

##### 1.1 RTL Code
```verilog
module incomplete_if(input I0, input I1, input I2, output reg Y);
  always @(*) begin
    if (I0) Y = I1; 
    // else missing → latch inferred
  end
endmodule
````

* Inputs: `I0`, `I1`
* Output: `Y` (reg)
* **Issue**: Missing `else` → output is not defined when `I0=0`.
* **Result**: Tools preserve state using **transparent D latch**.

---

##### 1.2 Expected Hardware

* `if` statement → MUX with `I0` as select.
* Missing `else` → output must **hold last value** → latch behavior.
* Equivalent hardware: **D latch** (`EN = I0`, `D = I1`, `Q = Y`).

---

##### 1.3 Simulation

Command:

```sh
iverilog incomplete_if.v tb_incomplete_if.v -o sim.out
./sim.out
gtkwave incomplete_if.vcd
```

Observed behavior:

* `I0=1` → `Y = I1` (transparent).
* `I0=0` → `Y` **holds previous value**.

📌 *[Insert waveform screenshot: `incomplete_if_sim.png`]*

---

##### 1.4 Synthesis

Command:

```tcl
yosys> read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> read_verilog incomplete_if.v
yosys> synth -top incomplete_if
yosys> abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> show
```

Result:

* **1 D latch inferred**:

  * Enable = `I0`
  * D = `I1`
  * Q = `Y`

📌 *[Insert Yosys schematic screenshot: `incomplete_if_synth.png`]*

---

#### 2. Lab: Incomplete If (Case 2)

##### 2.1 RTL Code

```verilog
module incomplete_if2(input I0, input I1, input I2, input I3, output reg Y);
  always @(*) begin
    if (I0)       Y = I1;
    else if (I2)  Y = I3;
    // else missing → latch inferred
  end
endmodule
```

* Inputs: `I0`, `I1`, `I2`, `I3`
* Output: `Y` (reg)
* **Issue**: No final `else`.
* **Result**: If `I0=0` and `I2=0`, then `Y` is **latched**.

---

##### 2.2 Expected Hardware

* Priority MUX chain:

  * `I0=1` → `Y = I1`
  * `I0=0, I2=1` → `Y = I3`
  * Else → **latched**

* Latch enable = `(I0 | I2)`

* Data input = `(I0 ? I1 : I3)`

---

##### 2.3 Simulation

Command:

```sh
iverilog incomplete_if2.v tb_incomplete_if2.v -o sim.out
./sim.out
gtkwave incomplete_if2.vcd
```

Observed behavior:

* `I0=1` → `Y = I1`.
* `I0=0, I2=1` → `Y = I3`.
* `I0=0, I2=0` → `Y` **latches previous value**.

📌 *[Insert waveform screenshot: `incomplete_if2_sim.png`]*

---

##### 2.4 Synthesis

Command:

```tcl
yosys> read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> read_verilog incomplete_if2.v
yosys> synth -top incomplete_if2
yosys> abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> show
```

Result:

* Yosys infers:

  * **D latch** with enable = `(I0 | I2)`.
  * Data = `MUX(I0?I1:I3)`.
  * Q = `Y`.

📌 *[Insert Yosys schematic screenshot: `incomplete_if2_synth.png`]*

---

#### 3. Key Learnings

* Incomplete **if** → inferred **latch**.
* Both **simulation** and **synthesis** confirm latch insertion.
* Latch enable = OR of all tested conditions.
* **Dangers**: Latches are level-sensitive → timing issues, glitches, unintended storage.
* **Best practice**: Always code a **final else** in combinational logic.

---

#### 4. Fix Example

Corrected code (no latch):

```verilog
always @(*) begin
  if (I0)       Y = I1;
  else if (I2)  Y = I3;
  else          Y = 1'b0; // defined default
end
```

→ Synthesizes to **pure mux logic**, no latch.

---

[🔼 Back to Table of Contents](#table-of-contents)

---

### 5.3 Labs on Incomplete Overlapping Case

This section explores **Verilog case statements** and the problems that occur when they are **incomplete, partially assigned, or overlapping**.

We cover 4 scenarios:

1. Incomplete case (no default)
2. Complete case (with default)
3. Partial assignment case
4. Overlapping case

---

#### 1. Incomplete Case

**File:** `incomplete_case.v`

```verilog
always @* begin
  case (sel)
    2'b00: y = i0;
    2'b01: y = i1;
    // missing sel=10, 11 → no default
  endcase
end
```

* Inputs: `i0`, `i1`, `i2`, `sel[1:0]`
* Output: `y`

##### Behavior

* `sel=00 → y=i0`
* `sel=01 → y=i1`
* `sel=10` or `11 → y holds previous value` (latch)

**Latch Enable Condition:** `~sel[1]`
**D Input Logic:** `y = sel[0] ? i1 : i0`

##### Simulation

```bash
iverilog incomplete_case.v tb_incomplete_case.v
./a.out
gtkwave tb_incomplete_case.vcd
```

* Matches expected behavior: latch when `sel[1]=1`.

##### Synthesis

```bash
read_verilog incomplete_case.v
synth -top incomplete_case
abc -liberty sky130.lib
show
```

* Latch inferred
* Q driven by `y`
* Enable = `~sel[1]`
* D = `mux(i0, i1, sel[0])`

---

#### 2. Complete Case (with Default)

**File:** `comp_case.v`

```verilog
always @* begin
  case (sel)
    2'b00: y = i0;
    2'b01: y = i1;
    default: y = i2;
  endcase
end
```

* Complete case → no latches

##### Behavior

* `sel=00 → y=i0`
* `sel=01 → y=i1`
* `sel=10/11 → y=i2`

##### Simulation

```bash
iverilog comp_case.v tb_comp_case.v
./a.out
gtkwave tb_comp_case.vcd
```

* Pure combinational 4-to-1 MUX observed

##### Synthesis

```bash
read_verilog comp_case.v
synth -top comp_case
abc -liberty sky130.lib
show
```

* No latches
* Only combinational gates (AND/OR/INV)
* Reduces to 4:1 MUX logic

---

#### 3. Partial Case Assignment

**File:** `partial_case_assign.v`

```verilog
always @* begin
  case (sel)
    2'b00: begin y = i0; x = i0; end
    2'b01: y = i1;          // x not assigned
    default: begin y = i2; x = i1; end
  endcase
end
```

* Inputs: `i0, i1, i2, sel[1:0]`
* Outputs: `x, y`

##### Behavior

* `y`: complete → no latch
* `x`: missing assignment at `sel=01` → latch

| sel[1] | sel[0] | x behavior |
| ------ | ------ | ---------- |
| 0      | 0      | i0         |
| 0      | 1      | latch      |
| 1      | 0      | latch      |
| 1      | 1      | i1         |

**Latch Enable Condition:**

```
sel[1] + ~sel[0]
```

##### Synthesis

```bash
read_verilog partial_case_assign.v
synth -top partial_case_assign
abc -liberty sky130.lib
show
```

* `y`: pure combinational path
* `x`: latch inferred with enable = `sel[1] + ~sel[0]`

---

#### 4. Overlapping Case

**File:** `bad_case.v`

```verilog
always @* begin
  case (sel)
    2'b00: y = i0;
    2'b01: y = i1;
    2'b10: y = i2;
    2'b1?: y = i3;   // overlaps with 10 and 11
  endcase
end
```

##### Problem

* `sel=10` matches **both** `2'b10` and `2'b1?`
* Different simulators and synthesizers may resolve differently
* Leads to **simulation–synthesis mismatch**

##### RTL Simulation

```bash
iverilog bad_case.v tb_bad_case.v
./a.out
gtkwave tb_bad_case.vcd
```

* `y` may latch or show undefined behavior at `sel=10`

##### Gate-Level Simulation (GLS)

```bash
read_verilog bad_case.v
synth -top bad_case
abc -liberty sky130.lib
write_verilog bad_case_net.v

iverilog ../my_lib/primitives.v \
        ../my_lib/sky130_cells.v \
        bad_case_net.v tb_bad_case.v
./a.out
gtkwave bad_case.vcd
```

* Synthesized netlist behaves as clean 4:1 MUX
* But **mismatch vs RTL sim**

---

#### Best Practices

* Always use `default` in case statements
* Assign **all outputs** in all case branches
* Avoid **overlapping case conditions**
* Use GLS to verify actual hardware behavior
* Remember: **incomplete = latches, overlapping = mismatches**

---

[🔼 Back to Table of Contents](#table-of-contents)

---

### 5.4 For-Loop and For-Generate

#### 1. Overview

In Verilog, **loop constructs** are used to either:

| Construct      | Location              | Purpose                         | Synthesizable Hardware         |
| -------------- | --------------------- | ------------------------------- | ------------------------------ |
| `for` loop     | Inside `always` block | Evaluate logic repeatedly       | ✅ Yes (single logic structure) |
| `for-generate` | Outside `always`      | Instantiate hardware repeatedly | ✅ Yes (replicated instances)   |

* `for` is for **logic evaluation**, not hardware replication.
* `for-generate` is for **replicating modules or gates**, not evaluating expressions.

---

#### 2. `for` Loop – Expression Evaluation (Inside `always`)

##### 2.1 Key Points

* Used **inside `always` blocks** (`always @*` or `always @(posedge clk)`).
* Repeats a logic check or assignment.
* Synthesizes to **one hardware block**, not multiple instances.
* Good for wide **MUX**, **DEMUX**, or **priority logic**.

---

##### 2.2 Example: 32-to-1 Multiplexer

```verilog
module mux32 (
  input  [31:0] in,
  input  [4:0]  sel,
  output reg    y
);
  integer i;
  always @(*) begin
    y = 1'b0;
    for (i = 0; i < 32; i = i + 1) begin
      if (i == sel)
        y = in[i];
    end
  end
endmodule
```

✅ **Behavior**:

* Loops over all 32 inputs.
* Assigns `y` from `in[sel]`.

✅ **Why it matters**:

* Changing `32` → `256` instantly scales to 256:1 MUX.
* No replication — synthesizer collapses it to one MUX.

---

##### 2.3 Example: 1-to-8 Demultiplexer

```verilog
module demux8 (
  input       din,
  input [2:0] sel,
  output reg [7:0] dout
);
  integer i;
  always @(*) begin
    dout = 8'b0;
    for (i = 0; i < 8; i = i + 1) begin
      if (i == sel)
        dout[i] = din;
    end
  end
endmodule
```

✅ **Behavior**:

* Initializes all outputs to `0`.
* Drives only `dout[sel] = din`.

✅ **Synthesis**:

* Compiler infers 8 AND gates with decoder logic.

---

#### 3. `for-generate` – Hardware Replication (Outside `always`)

##### 3.1 Key Points

* Used **outside** `always` blocks.
* Each loop iteration creates **real hardware**.
* Must use a **`genvar`** variable.
* Common for repetitive **module instantiation** or **bit-slice design**.

---

##### 3.2 Example: AND Gate Array

```verilog
module and_array (
  input  [7:0] a,
  input  [7:0] b,
  output [7:0] y
);
  genvar i;
  generate
    for (i = 0; i < 8; i = i + 1) begin : gen_and
      and u_and (.a(a[i]), .b(b[i]), .y(y[i]));
    end
  endgenerate
endmodule
```

✅ **Behavior**:

* Instantiates 8 separate AND gates.
* Each instance has its own connectivity.

✅ **Synthesis**:

* Hardware contains **8 physical gates**, each independent.

---

##### 3.3 Example: Ripple-Carry Adder (RCA)

```verilog
module rca #(
  parameter N = 8
)(
  input  [N-1:0] a,
  input  [N-1:0] b,
  output [N-1:0] sum,
  output         cout
);
  wire [N:0] carry;
  assign carry[0] = 1'b0;

  genvar i;
  generate
    for (i = 0; i < N; i = i + 1) begin : gen_fa
      full_adder fa (
        .a(a[i]),
        .b(b[i]),
        .cin(carry[i]),
        .sum(sum[i]),
        .cout(carry[i+1])
      );
    end
  endgenerate

  assign cout = carry[N];
endmodule
```

✅ **Behavior**:

* Replicates `full_adder` N times.
* Carry chains between stages.
* Perfect for scalable adders, multipliers, or shift-register chains.

---

#### 4. `if-generate` – Conditional Instantiation

Sometimes hardware should only exist under certain conditions (e.g., parameter-controlled).

```verilog
generate
  if (WIDTH == 8) begin
    and8 u_and8 (.a(a), .b(b), .y(y));
  end else begin
    and16 u_and16 (.a(a), .b(b), .y(y));
  end
endgenerate
```

✅ **Usage**:

* Used for parameterized designs.
* Synthesizer includes only the block that matches the condition.

---

#### 5. Best Practices

✅ **Do**:

* Use `for` for **iterative logic** inside `always`.
* Use `for-generate` for **replicating instances** outside `always`.
* Name generate blocks (`: label`) for better synthesis hierarchy.
* Keep loop bounds **static and constant**.

❌ **Don’t**:

* Use `for-generate` inside `always` → syntax error.
* Use non-constant loop limits → synthesis may fail.
* Forget `genvar` → simulation/synthesis mismatch possible.

---

#### 6. Quick Reference

| Feature            | `for` (in `always`)    | `for-generate` (outside `always`) |
| ------------------ | ---------------------- | --------------------------------- |
| Location           | Inside `always`        | Outside `always`                  |
| Purpose            | Expression evaluation  | Hardware replication              |
| Hardware generated | Single logic structure | Multiple instances                |
| Example            | MUX, DEMUX             | Adders, Gate Arrays, FIFOs        |
| Requires `genvar`  | ❌ No                   | ✅ Yes                             |
| Loop bounds        | Must be static         | Must be static                    |

---

#### 7. Summary

* **`for`** → use it for *logic evaluation* inside `always`. Synthesizes to **one hardware block**.
* **`for-generate`** → use it for *hardware replication* outside `always`. Synthesizes to **many hardware instances**.
* **`if-generate`** → use it for *conditional instantiation* based on parameters.

These constructs are fundamental for **scalable RTL design**, **parameterized modules**, and **clean structural code**.

---

[🔼 Back to Table of Contents](#table-of-contents)

---

### 5.5 Labs on For-Loop and For-Generate

#### 1. Multiplexer with `for` Loop

**Files:**  
- RTL: `mux_generate.v`  
- TB:  `tb_mux_generate.v`

##### RTL Concept
- Input: `i0..i3`, select `sel[1:0]`, output `y`.  
- Internal bus `i_int[3:0]` groups the inputs.  
- `for` loop iterates 0→3, checks if `k == sel`, assigns `y = i_int[k]`.  
- Functionally equivalent to a case statement, but scales easily.

##### Simulation
```bash
iverilog mux_generate.v tb_mux_generate.v -o a.out
./a.out
gtkwave dump.vcd
````

**Expected waveform:**

* `sel=0 → y=i0`
* `sel=1 → y=i1`
* `sel=2 → y=i2`
* `sel=3 → y=i3`

📌 **Note:**
Case statement requires `N` lines for `N:1` MUX.
For-loop version stays at ~4 lines for any size.

---

#### 2. Demultiplexer with `for` Loop

**Files:**

* RTL (case): `dmux_case.v`
* RTL (for):  `dmux_for.v`
* TB: `tb_dmux.v`

##### RTL Concept

* Input: `in`, select `sel[2:0]`, outputs `O[7:0]`.
* Outputs initialized to `0` to avoid inferred latches.
* `for` loop sets only `O[sel] = in`.

##### Simulation

Case-based:

```bash
iverilog dmux_case.v tb_dmux.v -o a.out
./a.out
gtkwave dump.vcd
```

Loop-based:

```bash
iverilog dmux_for.v tb_dmux.v -o a.out
./a.out
gtkwave dump.vcd
```

**Expected waveform:**

* `sel=n → On = in`, all others `0`.

📌 **Note:**
Case-based scales to hundreds of lines for large DMUX.
For-loop version stays compact (~13 lines).

---

#### 3. Ripple-Carry Adder with `for-generate`

**Files:**

* FA module: `fa.v`
* RCA module: `rca.v`
* TB: `tb_rca.v`

##### RTL Concept

* **Full Adder:** 3 inputs (a, b, cin), 2 outputs (sum, cout).
* **RCA:**

  * Add two 8-bit numbers → 9-bit result.
  * First FA instance: `cin=0`.
  * Remaining instances created with `for-generate`.
  * Carries chained automatically: `carry[i+1] = cout[i]`.

##### Simulation

```bash
iverilog fa.v rca.v tb_rca.v -o a.out
./a.out
gtkwave dump.vcd
```

**Expected behavior:**

* Example: `221 + 93 = 314`
* Matches rule: `n-bit + n-bit → (n+1)-bit`

📌 **Note:**
Use `genvar` (not `integer`) for generate loops.

---

#### 4. Key Points

* **For-loop (inside always):**

  * Used for **logic evaluation**.
  * Good for scalable MUX/DMUX.

* **For-generate (outside always):**

  * Used for **hardware replication**.
  * Good for arrays of adders, multipliers, etc.

* **Best practices:**

  * Initialize outputs to avoid latches.
  * Label generate blocks for readability.
  * Prefer loops over huge case statements for scalable designs.

---

#### 5. Assignments

1. Synthesize each design with `yosys` + SKY130 library.
2. Compare **RTL sim** vs **GLS** waveforms.
3. Use `yosys stat` to check gate count and area.
4. Extend designs:

   * 256×1 MUX
   * 1×256 DMUX
   * 32-bit RCA

Check how loop constructs reduce code size.

---

### Day 5 — Conclusion

- **If–Else vs Case**
  - `if–else` → priority logic (MUX chain).  
  - `case` → parallel logic (multiplexer).  
  - Missing `else` or `default` → latch inferred.  
  - Overlapping `case` items → simulation–synthesis mismatch.  

- **Latch Risks**
  - Latches are level-sensitive → timing issues, glitches.  
  - Avoid by always assigning all outputs in all branches.  

- **Best Practices**
  - Use `if–else` only when priority is needed.  
  - Use `case` for selection logic.  
  - Always add a final `else` or `default`.  
  - Ensure no overlapping case items.  

- **Loop Constructs**
  - `for` (inside `always`) → logic evaluation, scales MUX/DMUX.  
  - `for-generate` (outside `always`) → hardware replication, scalable structures (e.g., adders).  
  - Always use static bounds and `genvar` for generate loops.  

- **Verification**
  - Incomplete constructs cause inferred latches (confirmed in simulation and synthesis).  
  - RTL sim ≠ synthesis netlist if code is incomplete or overlapping.  
  - Always cross-check with **GLS** (Gate Level Simulation).  

---

## Conclusion

- **Simulators are event-driven:** Outputs change only when inputs toggle. VCD captures transitions, not static states.  
- **Synthesis maps RTL to cells:** Netlist uses NAND, NOR, DFF, etc. Testbench reuse ensures GLS validates hardware vs RTL.  
- **Optimizations are automatic:** Yosys prunes unused logic, propagates constants, simplifies Boolean expressions.  
- **Simulation-synthesis mismatches (SSM):**  
  - Missing sensitivity lists  
  - Blocking assignments in combinational logic  
  - Incomplete or overlapping case statements  
- **GLS is mandatory:** RTL sim alone is insufficient. GLS checks timing, resets, and real cell behavior.  
- **Correct coding prevents bugs:** Always cover all branches, initialize outputs, use non-blocking for flops.  
- **Loops scale efficiently:** Case-based 256:1 mux = hundreds of lines; loop-based = a few lines. For-generate replicates hardware cleanly.

 ---

[🔼 Back to Table of Contents](#table-of-contents)

---


---


