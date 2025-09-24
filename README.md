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
<summary>Day 2 - Timing libs, hierarchical vs flat synthesis and efficient flop coding styles </summary>

### Timing libs
Format: Liberty  
Name of library used: sky130_fd_sc_hd__tt_025C_1v80.lib  
Sky130: Sky 130 process node  
FD: Lib provided by skywater foundry  
SC: Standard cell  
HD: High density  
TT: Typical fabrication process  
025C: 25 degree Celsius junction temperature  
1v80: 1.8V operating voltage  
Last 3 parameters are the PVT condition for which the library is characterized.  

**Content of library:**
Technology info  
Units detail of time, voltage, current, etc  
Leakage power information for all input combinations  
Cell Delay for all input combinations  

### Hierarchical and flat synthesis:

**Steps for doing hierarchical synthesis:**
1. Start Yosys and read library `read_library -lib <library path>`
2. Read Verilog file `read_verilog <Verilog file>`
3. Synthesize the design: `synth -top < top module name>`
4. Generate gate level netlist: `abc -liberty <library path>`
5. Visualize the data using `show <module name>`
6. Dump the netlist using `write_verilog -noattr <output file name>`

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p5.png)
*Fig: Synthesis statistics of hierachical synthesys of multiple_modules*

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p6.png)
*Fig: Schematic and yosys dumped netlist of hier synthesis showing hierarchies*

**Steps for doing flat synthesis**

1. Follow step 1-4 of hierarchical synthesis
2. Flatten design using `flatten` command. 
3. follow step 5 and 6 of hierarchical synthesis.

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p7.png)
*Fig: Synthesis statistics of flat synthesys of multiple_modules*

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p8.png)
*Fig: Schematic and yosys dumped netlist of flat synthesis showing individual gates*


**Synthesis of a submodule of hierarchical design:**  
During step 3 replace top module name with name of the submodule:  
`synth -top <sub module name>`

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p9.png)
*Fig: Synthesis statistics of sub_module1 in multiple_modules*

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p10.png)
*Fig: Schematic and yosys dumped netlist of sub_module1*


### Flipflop analysis and Synthesis:
Used to hold value of signal to avoid glitch in design.  
Asynchronous reset: Output resets instantly when reset is enabled  
Synchronous reset: Output resets during the first clock edge post reset is enabled  
Types of flops:  
- Async reset DFF
- Sync reset DFF
- Async and Sync reset DFF
 
Simulation of Different types of DFF:

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p11.png)
*Fig: Simulation waveform of DFF with async, sync and both resets present complete window*

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p12.png)
*Fig: Simulation waveform of DFF with async, sync and both resets present zoomed window showing transition*


Synthesis of various DFFs:  
For synthesis of sequential cells we need to perform an additional step of mapping dff libraries after synthesis using the command `dfflibmap -liberty <dff lib area>`

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p13.png)
*Fig: Schematic and gate level netlist after synth of DFF with synchronous reset*


### Optimization in Yosys:
For mult2 function that multiplies input by 2 can be done by shifting of input to left by one position. There is no logic required  
Same is also seen in Yosys runs:  

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p14.png)
*Fig: Schematic after synth of mult2.v*

Similarly for mult9 if a is 3 bit  
    so a[2:0]*9 = y[5:0]  
	y[5:0] = a[2:0]*8 + a[2:0]   
	=> y[5:3] = a[2:0]  & y[2:0] = a[2:0]  
Hence here also no optimization is required.  

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p15.png)
*Fig: Schematic after synth of mult8.v*

</details>
