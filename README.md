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

### Introduction to Logic Optimization:

**Techniques of combinational optimization:**
1. Constant propagation: Optimizing combinational logic due to constant inputs,
	eg if an input of and gate is 0 the output is always 0 so the gate can be replaced with a logic 0.
2. Boolean logic optimization: Optimizing a logic equation to reduced form using boolean variables.


**Techniques of sequential optimization:**
1. Sequential constant propagation: Optimizing sequential logic based on input tied to sequential logics input,
	eg if D pin of a DFF without set tied to 0 then its output is always 0 so DFF can be removed.
2. State optimization: Optimization of unused states based on state diagram
3. Cloning: For multiple fanout flops the logic is replicated to optimize timing and reduce delays. 
4. Retiming: Moving/arranging combinational logic between pipelines to optimize timing and improve performance.

### Labs on Combinational logic Optimization

To do optimization logic in yosys use the following command post synthesis 
	`opt_clean -purge`

**Design1:** opt_check.v 
**Logic:** 	assign y = a?b:0;
**Logic optimization:**
y = a*b + a’*0 = a*b + 0 = a*b = ab

**note:** * implies and operation and + implies or operation
Requires 1 and gate to implement

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p16.png)  
*Fig: Schematic of optimized opt_check.v using one and gate*

**Design2:** opt_check2.v
**Logic:** 	assign y = a?1:b;
**Logic** optimization:
y = a*1 + a’*b = a + a’*b = a + b (absorption law)
Requires 1 or gate to implement

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p17.png)  
*Fig: Schematic of optimized opt_check2.v using one or gate*


**Design3:** opt_check3.v
**Logic:**	assign y = a?(c?b:0):0;
**Logic optimization:**
y = a’*0 + a*(c*b + c’*0) = 0 + a*(b*c + 0) = a*b*c
Requires a 3 input and gate
![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p18.png)  
*Fig: Schematic of optimized opt_check3.v using a 3 input and gate*

**Design4:** opt_check4.v
**Logic:**  assign y = a?(b?(a & c ):c):(!c);
**Logic optimization:** y = a*(b*a*c + b’*c) + a’*c’ = abc + ab’c + a’c’ = ac + a’c’ = a xnor c
Requires an xnor gate
![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p19.png)  
*Fig: Schematic of optimized opt_check4.v using an xnor gate*

In all above 4 design we were able to optimize them and same is done by yosys as well as shown in the lab snippets above.

**Combinational optimization for hierarchical deisng:**
**Design 5:** multiple_module_opt.v
**Logic:**
sub_module1 U1 (.a(a) , .b(1'b1) , .y(n1));
sub_module2 U2 (.a(n1), .b(1'b0) , .y(n2));
sub_module2 U3 (.a(b), .b(d) , .y(n3));

assign y = c | (b & n1);

sub_module1: y = a & b;
sub_module2: y = a^b;

**Logic optimization:**
n1 = a&1 = a
n2 = n1^0 = a^0 = a
n3 = a^d
y = c  | (b&n1) = c + b&a = ab + c
Requires 1 and gate & 1 or gate.
For hierarchical design we need to do `flatten` after synthesis follower by `opt_clean -purge`.

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p20.png)  
*Fig: Schematic of optimized multiple_module_opt.v using an and21or gate*

Yosys used andor cell, A1 and A2 inputs are anded (signal a and b) result is ored with B1 (c) giving output A1*A2 + B1 = a*b + c

**Design 6:** multiple_module_opt2.v
**Logic:**
sub_module U1 (.a(a) , .b(1'b0) , .y(n1));
sub_module U2 (.a(b), .b(c) , .y(n2));
sub_module U3 (.a(n2), .b(d) , .y(n3));
sub_module U4 (.a(n3), .b(n1) , .y(y));

sub_module: assign y = a & b;

**Logic optimization:** n1 = a & 0 = 0
n2  = b&c or b*c
n3 = n2 & d  = b*c*d
y = n3 & n1 = b*c*d*0 = 0

Output tied to 0

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p21.png)  
*Fig: Schematic of optimized multiple_module_opt2.v where output is tied to 0 as expected*

### Labs on Sequential logic Optimization

For sequential optimization we need to follow steps of synthesizing sequential logic and do `opt_clean -purge` post synth command.
**Design 1:** dff_cons1.v
**Logic:**
``module dff_const1(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b0;
	else
		q <= 1'b1;
end

endmodule``

It is an async reset DFF with D tied to logic 1 and async reset makes output logic 0, as it is async reset the flop cant be optimized. This is because the reset instantly makes output 0 but after reset is removed the output becomes 1 after next clock edge.

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p22.png)  
*Fig: Schematic of optimized dff_const1.v with a DFF and inverter*
 As the reset of the flop in library was of active low reset was inverted by yosys


**Design2:** dff_cons2.v
**Logic:** 
``module dff_const2(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b1;
	else
		q <= 1'b1;
end

endmodule``

It is an async reset DFF with D input tied to 1 and reset also making output 1, hence the output will always be 1 in all scenario. So DFF can be optimized and output tied to 1.

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p23.png)  
*Fig: Schematic of optimized dff_const2.v with flop optimized and output tied to logic1*

**Design 3:** dff_const3.v
**Logic:**
``module dff_const3(input clk, input reset, output reg q);
reg q1;

always @(posedge clk, posedge reset)
begin
	if(reset)
	begin
		q <= 1'b1;
		q1 <= 1'b0;
	end
	else
	begin
		q1 <= 1'b1;
		q <= q1;
	end
end

endmodule``

**Logic optimization:** When reset: q = 1, q1 = 0
After reset removed first clock edge : q1 = 1, q = q1 (previous value) = 0
second clock edge: q1 = 1, q = q1(prev) = 1
same for all subsequent clock cycles
Hence flop cant be optimized.

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p24.png)  
*Fig: Schematic of optimized dff_const3.v with two sets of flops and inverters*

There are 2 flops used for two signals q and q1.




**Design 4:** dff_const4.v
**Logic:**
``module dff_const4(input clk, input reset, output reg q);
reg q1;

always @(posedge clk, posedge reset)
begin
	if(reset)
	begin
		q <= 1'b1;
		q1 <= 1'b1;
	end
	else
	begin
		q1 <= 1'b1;
		q <= q1;
	end
end

endmodule``

**Logic optimization:**
When reset: q = 1, q1 = 1
After reset removed first clock edge : q1 = 1, q = q1 (previous value) = 1
second clock edge: q1 = 1, q = q1(prev) = 1
same for all subsequent clock cycles

q value is always 1 so it can be tied to logic 1 which is also observed in yosys optimization.

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p25.png)  
*Fig: Schematic of optimized dff_const4.v with flop optimized as expected*

**Design 5:** dff_count5.v
**Logic:**
`` module dff_const5(input clk, input reset, output reg q);
reg q1;

always @(posedge clk, posedge reset)
begin
	if(reset)
	begin
		q <= 1'b0;
		q1 <= 1'b0;
	end
	else
	begin
		q1 <= 1'b1;
		q <= q1;
	end
end

endmodule``


**Logic optimization:**
When reset: q = 0, q1 = 0
After reset removed first clock edge : q1 = 1, q = q1 (previous value) = 0
second clock edge: q1 = 1, q = q1(prev) = 1
same for all subsequent clock cycles

As values of q is not same and changes after reset is removed for first clock perior to 0 then goes to 1, flop cant be optimized and we require 2 flops in design.

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p26.png)  
*Fig: Schematic of optimized dff_const5.v with two sets of flops and inverters*

### Labs on unused output optimization:
**Design 1:** counter_opt.v
**Logic:**
``module counter_opt (input clk , input reset , output q);
reg [2:0] count;
assign q = count[0];

always @(posedge clk ,posedge reset)
begin
	if(reset)
		count <= 3'b000;
	else
		count <= count + 1;
end

endmodule``

It is a 3 bit counter and output is the LSB (count[0]) of the counter as count[1] and count[2] is not required, the design can be reduced to a 1 bit counter.

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p27.png)  
*Fig: Schematic of optimized counter_opt.v with 1 DFF and combinational logic*

It is using 1 DFF with inverted output q used as input forming toggle function. Which is representation of 1bit counter.

**Design2:** counter_opt2.v
**Logic:**
``module counter_opt (input clk , input reset , output q);
reg [2:0] count;
assign q = (count[2:0] == 3'b100);

always @(posedge clk ,posedge reset)
begin
	if(reset)
		count <= 3'b000;
	else
		count <= count + 1;
end

endmodule``
The design has a 3 bit counter and output is MSB of counter(count[2]), so we have to implement 3 bit counter with 3 flipflops and logic for inrementing count.

![Image](https://github.com/Santosh3672/RISCV-Tapeout-Week1/blob/main/Images%20W1/W1p28.png)  
*Fig: Schematic of optimized counter_opt2.v with 3 DFF and combinational logic*

As expected yosys optimized logic has 3 flip flops and combinational gates to implement incremental logic.

</details>