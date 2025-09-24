# RISCV Tapeout Week1
 Week1 of RISCV tapeout covering RTL design, Synthesis, Optimization and GLS

<details>
<summary> Day 1 - Introduction to Verilog RTL design and Synthesis </summary>
## Day 1 - Introduction to Verilog RTL design and Synthesis

### Introduction to Iverilog and GTKWave
**Stimulator:**  
Used to stimulate the RTL design and compare its output in adherence to specifications. We use **Icarus Verilog (Iverilog)** as the simulator tool.

**Testbench:**  
Set up to apply stimulus to the design to check its functionality.

The stimulator monitors changes in input and evaluates the output of the design by providing primary inputs and observing primary outputs.

Image here:

Iverilog based simulation flow:

Image here:

Input design and Testbench is given to iverilog which then generates a **VCD (value change dump)** file containing response of changing values of output wrt input. The VDC files can be viewed through another tool **GTKWave**.

### Labs on logic Simulation

After tools are installed cloned the lab module from https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.
Directory structure:
sky130RTLDesignAndSynthesisWorkshop
	-lib: Contains all standard cell liberty file
	-my_lib: Contains standard cell verilog models
	-verilog_files: Contains all verilog source file and testbench (name starting with tb_) that will be used during the labs.

Steps for simulating a design with iverilog and Viewing result in gtkwave:

1. Simulate design with iverilog using: iverilog <design  file.v> <testbench file.v>
	eg: iverilog good_mux.v tb_good_mux.v
2. After simulation it iverilog generates a.out file. Execute a.out file generated to generate vcd file.
3. Load the vcdfile using gtkwave: gtkwave <VCD file>
	eg: gtkwave tb_good_mux.vcd
4. In GTK viewer select the tesebench and select uut (unit under test) under the testbench.
5. After selection all signals of uut will show up, drag required signals to the signal section on right to view their wave, select zoom fit option to see full simulating window.
6. We can also track next or previous transition of a signal with find next/previous edge feature.

Image here:

Structure of a Testbench:

Instantiation of the unit under test (UUT).
Initializing inputs of uut.
Stimulus generator changing inputs usually done by togling inputs after a certain time period.

### Synthesis using Yosys:

Synthesis: Conversion of behavioral RTL to gate level Netlist using a standard cell library that.
Tool used: Yosys

Yosys synthesis flow:
1. Design reading: read_verilog
2. Liberty reading: read_liberty
3. Netlist dump: write_verilog

 To verify synthesis the generated netlist is stimulated with testbench used for RTL and the output is compared with RTL output. 
Set of primary input and output remains same between RTL and netlist design.

Standard cell library has various logic gates (AND, OR, DFF, etc) with different flavours of threshold voltage and drive strength. 
The different variant of cells offer different delays fast cells are useful in meeting setup timing  while hold cells are for hold timing.
Faster cell are achieved with faster charging/discharging wider transistor and they consume more area and power.

Hence selection of cells during synthesis is critical to meet timing and not have high power consumption. A constraint is required to guide synthesiser

### Yosys on Labs:

1. Invoke yosys using `yosys` command
2. Read the liberty file using `read_liberty -lib ./lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
3. Read verilog file using `read_verilog ./verilog_files/good_mux.v`
4. Now all inputs are available start synthesis using `synth -top <module name>`

Image here

5. Generate gate level netlist using abc command `abc -liberty ./lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
After execution the command prints number of inputs and outputs of the design and logic cell used for generating netlist which can be verified with the RTL.

Image here

6. After netlist dump we can visualize it using show command. For system with python 3.12 and above version setuptools package must be installed.

Image here

7. To write netlist use write_verilog command as below:
	`write_verilog -noattr good_mux_netlist.v`



</details>