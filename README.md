<h1 align="center">
🚀 RISC-V Reference SoC Tapeout Program  
</h1>

<p align="center">
<img src="asset/VSD_IITGN_SYNP_SCL-1024x576.png" alt="VSD IITGN SCL Logo" width="800" style="display:inline-block; margin:0 10px;"/>
</p>
<p align="center">
  <a href="https://riscv.org/">
    <img src="https://img.shields.io/badge/RISC--V-Open%20ISA-blue?style=flat-square&logo=risc-v" alt="RISC-V"/>
  </a>
  <a href="https://www.vlsisystemdesign.com/soc-labs/">
    <img src="https://img.shields.io/badge/VSD-Initiative-orange?style=flat-square" alt="VSD"/>
  </a>
  <a href="https://www.scl.gov.in/">
  <img src="https://img.shields.io/badge/SCL-Tapeout-blue?style=flat-square" alt="SCL"/>
</a>
<a href="#">
  <img src="https://img.shields.io/badge/Participants-3500%2B-orange?style=flat-square" alt="Participants"/>
</a>
<a href="http://opencircuitdesign.com/">
  <img src="https://img.shields.io/badge/Tools-Open--Source-green?style=flat-square" alt="Tools"/>
</a>
</p>


---

## 📖 About This Repository  

This repository documents my **20-week journey** through the **RISC-V Reference SoC Tapeout Program**, a national-level initiative organized by **IIT Gandhinagar (IITGN)** and **VLSI System Design (VSD)**.  

The program provides **hands-on training** in complete chip design — from **RTL design, synthesis, verification, and physical design, all the way to GDSII tapeout and silicon fabrication** — using industry-grade open source and Synopsys tools and the **SCL180nm PDK**.  

I will update this repo **week-by-week** with tasks, learnings, tool setups, and results.  

---

## 📅 Weekly Progress Tracker  

| Week | Focus Area                            | Hands-on Labs (tool-only)                                                                                                 | Deliverables / Checkpoints                                                                            | Status       |
| ---- | ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- | ------------ |
| 0    | Environment Setup & Tool Installation | Install tools, set up environment                                                                                         | Screenshots/logs of installed tools                                                                   | ✅ Completed  |
| 1    | Environment + RTL sim basics          | Set up toolchain; run iverilog simulations; inspect signals in GTKWave; inject a small RTL bug and detect via sim         | iverilog run logs; 2 annotated GTKWave screenshots; 5–7 lines on the bug & fix                        | 🔄 Submitted |
| 2    | Synthesis & GLS                       | Synthesize with Yosys; Gate-level simulation (GLS) with std-cell models; compare RTL vs GLS behavior                      | Pre/post-synth sim match screenshots; Yosys stat area/cell snippet; note on an observed optimization  | 🔜 Upcoming  |
| 3    | RISC-V ISA & toolchain                | Build & run a small RISC-V program (riscv-gnu-toolchain); simulate CPU testbench with iverilog + GTKWave (fetch/decode)   | Waves showing PC, instruction, decode outputs; brief note on pipeline stage boundaries                | 🔜 Upcoming  |
| 4    | Pipeline bugs, clock gating & CPU GLS | Reproduce a load-use hazard; apply clock-gating variant; re-synthesize (Yosys); CPU-level GLS with cell models            | Waveforms: buggy vs fixed; one slide on toggle reduction with clock gating; GLS pass log snippet      | 🔜 Upcoming  |
| 5    | STA fundamentals                      | Write core timing constraints (SDC); run OpenSTA on post-synth netlist; identify worst setup/hold paths & constraint gaps | Setup/Hold report excerpts (WNS/TNS); 5–8 lines justifying clocks/IO delays/false or multicycle paths | 🔜 Upcoming  |
| 6    | CMOS & SPICE with SKY130              | Draw basic gates in Xschem; simulate in ngspice; sweep VDD/Temp/Load; extract rise/fall delays                            | 3 plots (delay vs load; delay vs VDD; rise vs fall); one-pager explaining PVT effects on delay        | 🔜 Upcoming  |
| 7    | OpenLane: floorplan → placement       | Run OpenLane through floorplan & placement; tune utilization/aspect; tap/decap insertion; check congestion metrics        | Floorplan/placement metrics snapshot; congestion heatmap/metrics; note on best util/aspect found      | 🔜 Upcoming  |
| 8    | CTS, routing & post-route STA         | OpenLane: CTS → global/detail route; OpenSTA post-CTS & post-route; review hold fixes                                     | Post-route timing summary (WNS, TNS) at TT; final routed layout screenshot; note on hold-fix strategy | 🔜 Upcoming  |
| 9    | Multi-corner timing + sign-off        | Multi-corner OpenSTA (slow/typ/fast); generate SDF and run SDF-annotated GLS; assemble sign-off artifacts                 | Table of WNS/TNS across corners; SDF-GLS pass log & wave snapshot; list of sign-off artifacts         | 🔜 Upcoming  |
| 10   | Final polish & documentation          | Re-run best config to clean violations; summarize RTL→GDS flow, constraints, corner timing; short walkthrough video       | Final timing & DRC status summary; 4–6 page PDF report; 2–3 min demo video link                       | 🔜 Upcoming  |


---

## 🛠️ Tools & Environment  

For Week-0, I installed the essential **open-source EDA tools**:  

- **Yosys** → Logic synthesis & optimization  
- **Icarus Verilog (iverilog)** → Verilog simulation  
- **GTKWave** → Waveform viewing and debugging  

---

## 🙏 Acknowledgments

I would like to thank:

* [**Kunal Ghosh**](https://github.com/kunalg123) and Team **[VLSI System Design (VSD)](https://vsdiat.vlsisystemdesign.com/)** for leading the initiative.
* [**IIT Gandhinagar**](https://iitgn.ac.in/), [**Semiconductor Laboratory (SCL)**](https://www.scl.gov.in/), and [**Synopsys**](https://www.synopsys.com/) for their support.  
* The [**India Semiconductor Mission (ISM)**](https://ism.gov.in/) for enabling students across India to participate.

---

👨‍💻 **Participant:** [Rupam Bora](https://github.com/RupamBora-ASIC)
📅 **Program Duration:** 20 Weeks
🎯 **Goal:** Reach Top-20 Finalists for **GDSII Tapeout @ IITGN**

