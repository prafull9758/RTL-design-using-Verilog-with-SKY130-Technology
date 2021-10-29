
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

## Various Flop Coding Styles and Optimization

*What is a GLITCH?*
A glitch is a fast “spike” usually unwanted.

![Screenshot 2021-10-28 123947](https://user-images.githubusercontent.com/93269502/139204610-44c433d8-a5b6-483d-a152-cea7b847c0ba.png)

Glitch propogates through combinational circuit and results into unstable output.
*Flipflop acts as a shield to a glitch.*
A flip-flop is a device which stores a single bit (binary digit) of data; one of its two states represents a "one" and the other represents a "zero". Such data storage can be used for storage of state, and such a circuit is described as sequential logic in electronics.
Set and Reset acts as initializations to the flops.

![Screenshot 2021-10-28 130100](https://user-images.githubusercontent.com/93269502/139207175-bc9a2d64-da68-40b4-9f64-54a7dce92c6f.png)

![Screenshot 2021-10-28 130231](https://user-images.githubusercontent.com/93269502/139207401-2fdf7295-13d2-4df9-b129-e9f3d46be431.png)

The asynchronous reset is not awaiting the clock edge 

![Screenshot 2021-10-28 145150](https://user-images.githubusercontent.com/93269502/139226448-1fa168ca-21d6-48dc-9142-eecc55bd9d9d.png)


![syncres](https://user-images.githubusercontent.com/93269502/139228576-53e3c9de-7aec-4b85-99d0-9ba73ebd4d88.png)

![syn_asyn](https://user-images.githubusercontent.com/93269502/139229792-9d359ca9-18d3-4866-b924-88c98172118c.png)

![Screenshot 2021-10-28 151705](https://user-images.githubusercontent.com/93269502/139230780-2a19ffa9-4190-42f9-b049-c2514ae2e68e.png)

# D3 : Combinational and sequential optimization

  ### Combinational Logic Optimization
  ```
  1. Squeezing the logic to get most optimized design (Area and power saving)
  2. Constant Propogation (Direct optimization)
  3. Boolean logic optimization using K-map or Quin Mcksklusey.
   ```
   * Constant Propogation 
   ![Screenshot 2021-10-29 092522](https://user-images.githubusercontent.com/93269502/139372754-02854c09-b52a-430a-b1b2-ba961c46a106.png)

   * Boolean logic optimization
   ![Screenshot 2021-10-29 092814](https://user-images.githubusercontent.com/93269502/139372873-69306f05-c427-4619-95df-86332f029e81.png)

   # Sequential Logic optimisation
In Sequential Logic Optimisation there are 2 Techniques

1 .Basic
   
   * Sequential Constant Propogation
Some of the Sequential design in which D input is tied off the Squential Constant is propogated to give Q pin as a Constant and gives the most optmised design of the Squential Circuits.
The Sequential Design in which Q does not remains as constant cannot be optimised and flop needs to be retained in the circuit.

*Note*

Every flop in which the D input is tied off is a sequential constant for the flop to become sequential constant the Q pin should always take a constant value.

2. Advanced
 * State Optimisation
   ( Optimisation of unused states)
  * Retiming(Technique to improve the Performance of the Circuit.)
 *  Sequential Logic Cloning (Floor Plan Aware Synthesis):
        It is done when we are doing a Physical Aware Synthesis.
