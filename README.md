# RISCV Tapeout Week1

Week 1 of RISCV tapeout covering RTL design, Synthesis, Optimization, and Gate-Level Simulation (GLS).

<details>
<summary>Day 1 - Introduction to Verilog RTL Design and Synthesis</summary>

## Day 1 - Introduction to Verilog RTL Design and Synthesis

### Introduction to Iverilog and GTKWave

**Stimulator:**  
Stimulates the RTL design and compares its output to specifications. We use **Icarus Verilog (Iverilog)** as the simulator tool.

**Testbench:**  
Applies stimulus to the design to check its functionality.

The stimulator monitors changes in input and evaluates the output by providing primary inputs and observing primary outputs.

![Testing of Design using Testbench](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1d1p1.png)  
*Fig: Testing of Design using Testbench*

![Iverilog based simulation flow](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1d1p2.png)
*Fig: Iverilog based simulation flow*

**Simulation Flow:**  
- Input design and testbench files are provided to **Iverilog**, which generates a **VCD (Value Change Dump)** file containing signal changes over time.
- The VCD file can be viewed using **GTKWave** for waveform analysis.

### Labs on Logic Simulation

After installing the tools, clone the lab module from [sky130RTLDesignAndSynthesisWorkshop](https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop).

**Directory Structure:**
- `lib`: Standard cell liberty files
- `my_lib`: Standard cell Verilog models
- `verilog_files`: Verilog source files and testbenches (testbenches start with `tb_`)

**Simulation Steps:**
1. Simulate design with Iverilog:
    ```sh
    iverilog <design_file.v> <testbench_file.v>
    # Example:
    iverilog good_mux.v tb_good_mux.v
    ```
2. Run the generated `a.out` file to produce the VCD file.
3. View the VCD file in GTKWave:
    ```sh
    gtkwave <VCD_file>
    # Example:
    gtkwave tb_good_mux.vcd
    ```
4. In GTKWave, select the testbench and the UUT (Unit Under Test) to view signals.
5. Drag required signals to the signal section and use "Zoom Fit" to see the full simulation window.
6. Use "Find Next/Previous Edge" to track signal transitions.

![Simulation of good mux using iverilog and waves in GTKWave](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1d1p3.png)
*Fig: Simulation of good mux using iverilog and waves in GTKWave*

**Testbench Structure:**
- Instantiate the Unit Under Test (UUT)
- Initialize UUT inputs
- Generate stimulus by toggling inputs after a set time period

---

### Synthesis using Yosys

**Synthesis:**  
Converts behavioral RTL to gate-level netlist using a standard cell library.  
**Tool used:** Yosys

**Yosys Synthesis Flow:**
1. Read design:
    ```sh
    read_verilog <verilog_file>
    ```
2. Read liberty file:
    ```sh
    read_liberty -lib <liberty_file>
    ```
3. Start synthesis:
    ```sh
    synth -top <module_name>
    ```
4. Generate gate-level netlist:
    ```sh
    abc -liberty <liberty_file>
    ```
5. Visualize netlist in schematic viewer(requires setuptools for Python 3.12+):
    ```sh
    show
    ```
6. Write netlist:
    ```sh
    write_verilog -noattr <output_netlist.v>
    ```

![Synthesis of good_mux in Yosys post synth stage](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1d1p4.png)
*Fig: Synthesis of good_mux in Yosys post synth stage*

![Synthesis of good_mux in Yosys post netlist generation](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1d1p5.png)
*Fig: Synthesis of good_mux in Yosys post netlist generation*

![Yosys schematic viewer of good_mux.v](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1d1p6.png)
*Fig: Yosys schematic viewer of good_mux.v*

**Notes:**
- After synthesis, stimulate the generated netlist with the same testbench used for RTL and compare outputs.
- Standard cell libraries contain various logic gates (AND, OR, DFF, etc.) with different threshold voltages and drive strengths.
- Cell selection during synthesis is critical for timing and power optimization.

</details>

<details>
<summary>Day 2 - Timing Libraries, Hierarchical vs Flat Synthesis, and Efficient Flip-Flop Coding Styles</summary>

## Day 2 - Timing Libraries, Hierarchical vs Flat Synthesis, and Efficient Flip-Flop Coding Styles

### Timing Libraries

**Format:** Liberty  
**Library Used:** `sky130_fd_sc_hd__tt_025C_1v80.lib`  
- **Sky130:** SkyWater 130nm process node  
- **FD:** Provided by SkyWater Foundry  
- **SC:** Standard Cell  
- **HD:** High Density  
- **TT:** Typical fabrication process  
- **025C:** 25°C junction temperature  
- **1v80:** 1.8V operating voltage  

*The last three parameters specify the PVT (Process, Voltage, Temperature) conditions for which the library is characterized.*

**Library Contents:**
- Technology information
- Units for time, voltage, current, etc.
- Leakage power for all input combinations
- Cell delay for all input combinations

---

### Hierarchical vs Flat Synthesis

#### Hierarchical Synthesis Steps

1. Start Yosys and read the library:
    ```sh
    read_liberty -lib <library_path>
    ```
2. Read the Verilog file:
    ```sh
    read_verilog <verilog_file>
    ```
3. Synthesize the design:
    ```sh
    synth -top <top_module_name>
    ```
4. Generate gate-level netlist:
    ```sh
    abc -liberty <library_path>
    ```
5. Visualize the data:
    ```sh
    show <module_name>
    ```
6. Dump the netlist:
    ```sh
    write_verilog -noattr <output_file_name>
    ```

![Hierarchical synthesis statistics](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p5.png)  
*Fig: Synthesis statistics of hierarchical synthesis of multiple modules*

![Hierarchical schematic](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p6.png)  
*Fig: Schematic and Yosys dumped netlist of hierarchical synthesis showing hierarchies*

#### Flat Synthesis Steps

1. Follow steps 1-4 of hierarchical synthesis.
2. Flatten the design:
    ```sh
    flatten
    ```
3. Continue with steps 5 and 6 of hierarchical synthesis.

![Flat synthesis statistics](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p7.png)  
*Fig: Synthesis statistics of flat synthesis of multiple modules*

![Flat schematic](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p8.png)  
*Fig: Schematic and Yosys dumped netlist of flat synthesis showing individual gates*

#### Synthesis of a Submodule

To synthesize a submodule, replace the top module name in step 3:
```sh
synth -top <submodule_name>
```

![Submodule synthesis statistics](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p9.png)  
*Fig: Synthesis statistics of sub_module1 in multiple_modules*

![Submodule schematic](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p10.png)  
*Fig: Schematic and Yosys dumped netlist of sub_module1*

---

### Flip-Flop Analysis and Synthesis

Flip-flops are used to hold signal values and avoid glitches in design.

- **Asynchronous reset:** Output resets instantly when reset is enabled.
- **Synchronous reset:** Output resets on the first clock edge after reset is enabled.

**Types of Flip-Flops:**
- Async reset DFF
- Sync reset DFF
- Async and Sync reset DFF

**Simulation of Different DFFs:**

![DFF simulation waveform](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p11.png)  
*Fig: Simulation waveform of DFF with async, sync, and both resets (complete window)*

![DFF simulation zoomed](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p12.png)  
*Fig: Simulation waveform of DFF with async, sync, and both resets (zoomed window showing transition)*

**Synthesis of DFFs:**  
For sequential cells, map DFF libraries after synthesis:
```sh
dfflibmap -liberty <dff_lib_area>
```

![DFF synthesis schematic](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p13.png)  
*Fig: Schematic and gate-level netlist after synthesis of DFF with synchronous reset*

---

### Optimization in Yosys

For the `mult2` function (multiplying input by 2), Yosys optimizes by shifting the input left by one position—no logic required.

![mult2 schematic](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p14.png)  
*Fig: Schematic after synthesis of mult2.v*

For `mult9` (if `a` is 3 bits):  
`a[2:0] * 9 = y[5:0]`  
`y[5:0] = a[2:0] * 8 + a[2:0]`  
So, `y[5:3] = a[2:0]` and `y[2:0] = a[2:0]`  
No further optimization required.

![mult8 schematic](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p15.png)  
*Fig: Schematic after synthesis of mult8.v*

</details>

<details>
<summary>Day 3 - Combinational and Sequential Optimization</summary>

## Day 3 - Combinational and Sequential Optimization

### Introduction to Logic Optimization

**Combinational Optimization Techniques:**
- **Constant propagation:**  
  Optimizes logic when inputs are constant.  
  *Example:* If an AND gate input is 0, output is always 0, so the gate can be replaced with logic 0.
- **Boolean logic optimization:**  
  Reduces logic equations using Boolean algebra.

**Sequential Optimization Techniques:**
- **Sequential constant propagation:**  
  Optimizes sequential logic when inputs are tied to constants.  
  *Example:* If D input of a DFF is always 0, output is always 0, so DFF can be removed.
- **State optimization:**  
  Removes unused states based on the state diagram.
- **Cloning:**  
  Replicates logic for multiple fanout flops to optimize timing and reduce delays.
- **Retiming:**  
  Rearranges combinational logic between pipeline stages to improve timing and performance.

---

### Labs on Combinational Logic Optimization

Use the following command in Yosys after synthesis for optimization:
```sh
opt_clean -purge
```

#### Design 1: `opt_check.v`
- **Logic:** `assign y = a ? b : 0;`
- **Optimization:**  
  `y = a*b` (requires one AND gate)

![Schematic](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p16.png)  
*Fig: Optimized schematic using one AND gate*

#### Design 2: `opt_check2.v`
- **Logic:** `assign y = a ? 1 : b;`
- **Optimization:**  
  `y = a + b` (requires one OR gate)

![Schematic](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p17.png)  
*Fig: Optimized schematic using one OR gate*

#### Design 3: `opt_check3.v`
- **Logic:** `assign y = a ? (c ? b : 0) : 0;`
- **Optimization:**  
  `y = a * b * c` (requires a 3-input AND gate)

![Schematic](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p18.png)  
*Fig: Optimized schematic using a 3-input AND gate*

#### Design 4: `opt_check4.v`
- **Logic:** `assign y = a ? (b ? (a & c) : c) : (!c);`
- **Optimization:**  
  `y = a xnor c` (requires an XNOR gate)

![Schematic](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p19.png)  
*Fig: Optimized schematic using an XNOR gate*

#### Hierarchical Design Optimization

**Design 5:** `multiple_module_opt.v`
- **Logic:**  
  - `sub_module1: y = a & b;`
  - `sub_module2: y = a ^ b;`
  - Top-level: `assign y = c | (b & n1);`
- **Optimization:**  
  `y = a*b + c` (requires one AND gate and one OR gate)

![Schematic](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p20.png)  
*Fig: Optimized schematic using AND-OR gate*

**Design 6:** `multiple_module_opt2.v`
- **Logic:**  
  All submodules are AND gates, but one input is always 0.
- **Optimization:**  
  Output is always 0.

![Schematic](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p21.png)  
*Fig: Output tied to 0 as expected*

---

### Labs on Sequential Logic Optimization

After synthesizing sequential logic, use:
```sh
opt_clean -purge
```

#### Design 1: `dff_const1.v`
- **Logic:** Async reset DFF, D input tied to 1.
- **Optimization:**  
  Cannot be optimized; output changes after reset.

![Schematic](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p22.png)  
*Fig: DFF and inverter used*

#### Design 2: `dff_const2.v`
- **Logic:** Async reset DFF, D input and reset both set output to 1.
- **Optimization:**  
  Output always 1; DFF can be removed.

![Schematic](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p23.png)  
*Fig: Output tied to logic 1*

#### Design 3: `dff_const3.v`
- **Logic:** Two registers, output changes after reset.
- **Optimization:**  
  Cannot be optimized; requires two flops.

![Schematic](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p24.png)  
*Fig: Two flops and inverters used*

#### Design 4: `dff_const4.v`
- **Logic:** Two registers, output always 1.
- **Optimization:**  
  Output tied to logic 1.

![Schematic](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p25.png)  
*Fig: Flop optimized as expected*

#### Design 5: `dff_const5.v`
- **Logic:** Two registers, output changes after reset.
- **Optimization:**  
  Cannot be optimized; requires two flops.

![Schematic](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p26.png)  
*Fig: Two flops and inverters used*

---

### Labs on Unused Output Optimization

#### Design 1: `counter_opt.v`
- **Logic:** 3-bit counter, output is LSB.
- **Optimization:**  
  Reduced to 1-bit counter.

![Schematic](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p27.png)  
*Fig: 1 DFF and combinational logic (toggle function)*

#### Design 2: `counter_opt2.v`
- **Logic:** 3-bit counter, output is MSB.
- **Optimization:**  
  Requires 3 flip-flops and combinational logic.

![Schematic](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p28.png)  
*Fig: 3 DFFs and combinational logic for increment*

