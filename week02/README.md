# Week 2 — 🧠 VSDBabySoC:Step into Real Chip Design

---

This isn’t just another academic exercise. VSDBabySoC is a **hands-on playground** to learn how real chips are built—from CPU to analog output—using open-source tools and actual silicon-ready flows.

---

## 🤔 What *Exactly* Is an SoC?

An **SoC (System-on-Chip)** is a full computer squeezed onto a single piece of silicon.  
No separate CPU board. No external memory controller. No extra sound card.  
**Everything lives together on one die.**

Think of your phone:  
📱 It runs apps, plays music, connects to Wi-Fi, and lasts all day on a tiny battery.  
That’s only possible because it uses an SoC—not a bunch of loose chips.

### Why SoCs Rule:
- 🔋 **Low power**: Shorter wires = less energy wasted  
- 📏 **Tiny footprint**: Fits in wearables, sensors, even medical implants  
- ⚡ **Fast**: No waiting for signals to cross board traces  
- 💰 **Cheaper**: One chip > ten chips + PCB + assembly

![Apple-M1-Architecture-1024x475](https://github.com/user-attachments/assets/a3428bc5-1ddd-4bdc-b3a6-10a74205c8d8)

---

## 🔧 What’s Inside a Typical SoC?

Every SoC has four key pieces:

### 1. **CPU** → The brain  
In BabySoC: **RVMYTH**, a minimal RISC-V core. No fancy pipelines—just enough to run real code.

### 2. **Memory** → Short-term and long-term storage  
BabySoC keeps it simple: external RAM/ROM. We’re focused on **integration**, not memory controllers.

### 3. **Peripherals (I/O)** → The chip’s senses  
BabySoC includes:
- 🕒 **PLL**: Cleans up the clock so the CPU doesn’t glitch
- 📡 **10-bit DAC**: Turns digital numbers into analog voltage (for audio, video, sensors)
- 🔌 **SPI**: Lets you load code or debug

### 4. **Interconnect** → The nervous system  
A bus (like Wishbone or AXI) that lets CPU talk to DAC, PLL, etc.  
In BabySoC: a simple point-to-point wiring—no fancy NoC needed.

---

## 🎓 Why BabySoC? (And Why It’s Perfect for Learning)

Let’s be real: commercial SoCs are **huge**, **complex**, and **closed-source**.  
You can’t see how the GPU talks to the memory controller in a Snapdragon.

**BabySoC is different:**  
✅ Only **3 blocks**: CPU + PLL + DAC  
✅ **Fully open-source**—read, modify, break, fix  
✅ Built to **teach fundamentals**, not ship to customers

### How it actually works:
1. You power it on → crystal oscillator starts ticking  
2. **PLL** locks onto that signal and outputs a clean, stable clock  
3. **RVMYTH** boots up and runs your code  
4. Your program writes values to register `r17`  
5. **DAC** reads `r17` and outputs a matching analog voltage → `OUT`  
6. Plug `OUT` into a speaker or TV → hear/see your code in action! 🎶📺

![333622249-04238eab-4d48-4d57-9061-f8b660a83d6e](https://github.com/user-attachments/assets/38253bb7-b658-496d-a043-15402219e089)
Signal flow: crystal → PLL → CPU → DAC → analog output*

### What you’ll actually learn:
- How clock domains affect timing  
- Why you can’t just feed CPU output directly to analog pins  
- How to verify a full system—not just a single module  
- The pain (and joy) of integrating IP blocks that weren’t made to work together

---

## 🧪 Why Functional Modeling Comes *Before* RTL

Here’s a hard truth: **writing RTL too early is a waste of time.**

Before you touch Verilog:
1. **Model the system in C/Python**  
   → Simulate the DAC output when `r17 = 512`  
   → Check if the PLL locks in 10 µs or 100 µs  
2. **Run your firmware on the model**  
   → Does your sine-wave generator actually produce a sine wave?  
3. **Tweak the architecture**  
   → Need 12-bit DAC instead of 10-bit? Change it now—not after 2 weeks of RTL

This is how real teams work.  
You don’t start coding until you’ve **proven the idea works** at a high level.

### Where it fits:
```
Spec → 🧠 Functional Model → ✍️ RTL → ⚙️ Synthesis → 🗺️ P&R → 🖨️ GDSII → 🖥️ Silicon
```

![img_61d89021d8d47 copy](https://github.com/user-attachments/assets/54b5e8f9-f03d-4b53-a535-859360589119)

In BabySoC, your functional model answers:  
❓ *“If the CPU writes 0x200 to r17, what voltage comes out of the DAC?”*  
Get that right *before* you write a single line of Verilog.

---

## 💡 Bottom Line

VSDBabySoC isn’t about building the next iPhone chip.  
It’s about **learning the right way** to design silicon:  
- Start simple  
- Verify early  
- Integrate step by step  
- Trust **simulation**, but prove it with **GLS**  

And yes—you’ll end up with a real chip that outputs real analog signals.  
That’s the magic. 🔮

> “Don’t just simulate—**fabricate**.”  
> — India’s Semiconductor Mission 🇮🇳

---

# 🧪 VSDBabySoC: Hands-on Functional Modelling Lab

This lab walks you through **functional simulation** of the **VSDBabySoC**—a minimal RISC-V SoC with PLL clock generation and 10-bit DAC analog output. You’ll use **Icarus Verilog (`iverilog`)** and **GTKWave** to verify system behavior **before synthesis**.

---

## 🛠️ Tools Required
- **Icarus Verilog** (`iverilog`, `vvp`) → compiles and simulates Verilog
- **GTKWave** → visualizes waveforms (`.vcd` files)
- **Git** → to clone the project

> ✅ Works on Linux/macOS. Install via:
> ```bash
> # Ubuntu/Debian
> sudo apt install iverilog gtkwave git
> # macOS (with Homebrew)
> brew install icarus-verilog gtkwave git
> ```

---

## 📂 Step 1: Clone the Project
```bash
git clone <repo>
```

Expected structure:
```
VSDBabySoC/
├── src/
│   ├── include/      # .vh header files
│   └── module/       # rvmyth.v, pll.v, dac.v, vsdbabysoc.v, testbench.v
└── output/           # simulation outputs (you’ll create this)
```

---

## ⚙️ Step 2: Compile & Run Pre-Synthesis Simulation

### Create output directory
```bash
mkdir -p output/pre_synth_sim
```

### Compile with `iverilog`
```bash
iverilog -o output/pre_synth_sim/sim.out \
  -DPRE_SYNTH_SIM \
  -I src/include \
  -I src/module \
  src/module/testbench.v \
  src/module/vsdbabysoc.v \
  src/module/rvmyth.v \
  src/module/avsdpll.v \
  src/module/avsddac.v
```

### Run simulation
```bash
cd output/pre_synth_sim
../sim.out  # generates pre_synth_sim.vcd
```

---

## 🔍 Step 3: Analyze Waveforms in GTKWave

```bash
gtkwave pre_synth_sim.vcd
```

In GTKWave:
1. Click **`tb`** → expand hierarchy
2. Drag signals to waveform pane:
   - `reset`
   - `CLK` (from PLL)
   - `rvmyth.OUT` (10-bit digital data)
   - `dac.OUT` (analog output)
3. Use **Zoom Fit** (`F`) to see full simulation

📌 *Image placeholder: `screenshots/babysoc_full_wave.png` — full waveform view*

---

## 📊 Key Signals to Observe

### 1. **Reset Behavior**
- At start, `reset = 1` → RISC-V core held in reset
- When `reset = 0`, CPU begins execution
- DAC output stays at 0 during reset

📌 *Image placeholder: `screenshots/reset_phase.png`*  
> **What it shows**: CPU inactive during reset; DAC output = 0.

---

### 2. **PLL Clock Generation**
- `CLK` starts oscillating after PLL locks
- Stable, clean clock drives RISC-V core
- No glitches or jitter in simulation

📌 *Image placeholder: `screenshots/pll_clock.png`*  
> **What it shows**: PLL generates stable clock for CPU.

---

### 3. **Data Flow: CPU → DAC**
- `rvmyth.OUT` updates every few clock cycles (e.g., counting pattern)
- `dac.OUT` mirrors digital value as analog voltage
- Example: `rvmyth.OUT = 10'h200` → `dac.OUT` ≈ mid-scale voltage

📌 *Image placeholder: `screenshots/cpu_to_dac.png`*  
> **What it shows**: Digital data from CPU converted to analog by DAC.

---

### 4. **Analog Output (`dac.OUT`)**
- Smooth step changes (no spikes)
- Output scales with `VREFH` (reference voltage)
- Matches expected DAC transfer function

📌 *Image placeholder: `screenshots/dac_output.png`*  
> **What it shows**: DAC produces correct analog levels for digital inputs.

---

## 📁 Deliverables Checklist

| Item | Location |
|------|--------|
| ✅ Simulation log (`sim.out` compile output) | `output/pre_synth_sim/compile.log` |
| ✅ VCD waveform file | `output/pre_synth_sim/pre_synth_sim.vcd` |
| 📸 Screenshot: Reset phase | `screenshots/reset_phase.png` |
| 📸 Screenshot: PLL clock | `screenshots/pll_clock.png` |
| 📸 Screenshot: CPU → DAC data | `screenshots/cpu_to_dac.png` |
| 📸 Screenshot: DAC analog output | `screenshots/dac_output.png` |
| 📝 Short explanation per screenshot | In this README (see above) |

> 💡 **Pro tip**: Use `gtkwave`’s **zoom** and **cursor** to measure clock period or DAC settling time.

---

## 🚨 Troubleshooting

| Issue | Fix |
|------|-----|
| ❌ `Module redefined` | Don’t include modules in testbench with `` `include ``—list them explicitly in `iverilog` command |
| ❌ `File not found` | Double-check `-I` paths; use absolute paths if needed |
| ❌ No `dac.OUT` in GTKWave | Ensure `avsddac.v` is compiled; check testbench `$dumpvars` scope |

---

## 🎯 Why This Matters
- **Functional simulation catches bugs early**—before synthesis, P&R, or tapeout.
- You’re verifying **system-level integration**: CPU + clock + analog output.
- This is how real SoC teams validate behavior **before spending weeks on physical design**.

> 🔚 **Next step**: After pre-synthesis sim, you’ll synthesize with Yosys and run **post-synthesis GLS** to confirm no mismatches.

---
