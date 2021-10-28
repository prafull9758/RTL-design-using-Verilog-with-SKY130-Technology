
# RTL-design-using-Verilog-with-SKY130-Technology

This is a repository on RTL design using Verilog with SKY130 Technology workshop conducted by VSD which intends to teach the verilog coding guidelines that results in predictable logic in Silicon by validating the functionality of the design (functional RTL code)  using Functional Simulation, writing Test Benches to validate the functionality of the RTL design, doing logic synthesis of the Functional RTL Code and doing Gate Level Simulation of  Synthesized Netlist.
 
## Day 1 Introduction to Verilog RTL Design and Synthesis

* ### Introduction to open source simulator iverilog

***Simulator***

RTL design is checked adherence to the spec by simulating the design.Simulator is the tool used for simulating this design.
*iverilog* is the tool used for simulation.

***Design***

Design is the actual Verilog Code or set of verilog codes which has the intended functionality to meet with the required specifications.

***Testbench***

TestBench is setup toapply stimulus (test vectors) to the design to check its Functionality.

***How the Simulator Works?***

Simulator looks for changes in input signal.  Upon change in input the output is observed.  
*If there is no change to the input there will be no change to the output!!*
Simulator looks for changes in the Value of input.

![Design](https://user-images.githubusercontent.com/84860957/120113197-056a3100-c197-11eb-89a3-6cd78ab1ba34.JPG)

***Note:***
> _Design may have 1 or more primary inputs,1 or more primary outputs.TestBench doesn't have a Primary inputs or a primary outputs._

![Screenshot 2021-10-27 220359](https://user-images.githubusercontent.com/93269502/139107705-e56e9182-d64d-47ff-855f-039e85b179f3.png)

 

# Day 2 Timing libs Hierarical vs Flat Synthesis and efficient Flop Coding Styles

## Introduction to timing .lib

The .lib file used here is  sky130_fd_sc_hd__tt_025C_1v80.lib .
Each term in nomenclature has its own significance :
```
> fd - the skywater foundary
> sc - digital standard cell
> hd - high density
> tt - typical process
> 025C - temperature
> 1v - voltage
 
 ```
If we look into the library there is a three letter word coming into picture
```
 PVT
 where P = process (variations due to fabrications)
       v = voltage (variations due to voltage)
       T = temperature (variations due to temperature)

```
we need our design to work efficiently across all three corners.

*our libraries should be characterized to model these variations*

.lib files contains same cells of different flavours

Details which are provided in the .lib file are
```
Internal Power
Leakage Power
Area
Pin details
Timings
Capacitance
```
![Screenshot 2021-10-28 095243](https://user-images.githubusercontent.com/93269502/139186128-1233cb37-a332-4c22-b95c-61ca6caebb0e.png)


```
area(and2_4) > area(and2_2) > area(and2_0)

power(and2_4) > power(and2_2) > power(and2_0)

```
# Hierarchical vs Flat Synthesis

* *Hierarchical netlist :* Since pins of submodules are accessible, it's easier to track paths for functional debugging and timing analysis. Pins can be forced or probed in post-synthesis simulations.

* *Flattened netlist :* Synthesis tool can optimize the circuit better. That provides better speed, area, and power. Conversely, debugging capabilities are limited.
Example : multiple_modules.v

```
module sub_module2 (input a, input b, output y);
	assign y = a | b;
endmodule

module sub_module1 (input a, input b, output y);
	assign y = a&b;
endmodule


module multiple_modules (input a, input b, input c , output y);
	wire net1;
	sub_module1 u1(.a(a),.b(b),.y(net1));  //net1 = a&b
	sub_module2 u2(.a(net1),.b(c),.y(y));  //y = net1|c ,ie y = a&b + c;
endmodule
 ```

 ### Hierarchical Synthesis command flow
> #yosys         //Invoke yosys

> #read_liberty -lib <path to .lib file>  //Read liberty file

> #read_verilog <RTL_code.v>      // Read verilog file

> #synth -top <top_module_name>     //synthesis command

> #abc _liberty <path to .lib file>  //map to standard cell

> #show  //see the Logic design

> #write_verilog -noattr <netlist_name.v>  //generate netlist

### Hierarchical Synthesis of multiple_modules.v
![Design](https://user-images.githubusercontent.com/84860957/120082318-e0ff4d80-c0df-11eb-8e55-d167fa8fb90d.JPG)
note that it does not show 'and' gate or 'or' gate but instead shows the 2 sub modules u1 and u2.This is what we call the Hierarical design.

### Flatten Synthesis command flow
> #yosys         //Invoke yosys

> #read_liberty -lib <path to .lib file>  //Read liberty file

> #read_verilog <RTL_code.v>      // Read verilog file

> #synth -top <top_module_name>     //synthesis command

> #abc _liberty <path to .lib file>  //map to standard cell

> #flatten

> #show  //see the Logic design

> #write_verilog -noattr <netlist_name.v>  //generate netlist

### Flatten Synthesis of multiple_modules.v
![Design](https://user-images.githubusercontent.com/84860957/120082884-16f20100-c0e3-11eb-8e9f-f2e96955816a.JPG)
here we can see submodules are flattened into gates.

### We can also perform just a particular submodule synthesis than synthesizing the whole top_level module

*Why submodule level synthesis?*
> When we want to use multiple instances of the same module.
> We want to do divide and conquer approch if we have a massive design.

*How?*
command --> #synth -top sub_module1
![Screenshot 2021-10-28 105623](https://user-images.githubusercontent.com/93269502/139191937-02ea176d-8f6b-4be0-87fb-4d352b3a142b.png)
