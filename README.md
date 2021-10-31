![Capture](https://user-images.githubusercontent.com/93269502/139590205-f287ba81-0a14-448f-b373-da48c009e3e3.PNG)
# RTL-design-using-Verilog-with-SKY130-Technology

This is a repository on RTL design using Verilog with SKY130 Technology workshop conducted by VSD which intends to teach the verilog coding guidelines that results in predictable logic in Silicon by validating the functionality of the design (functional RTL code)  using Functional Simulation, writing Test Benches to validate the functionality of the RTL design, doing logic synthesis of the Functional RTL Code and doing Gate Level Simulation of  Synthesized Netlist.
 
* ### [Introduction to Verilog RTL Design and Synthesis](https://github.com/prafull9758/RTL-design-using-Verilog-with-SKY130-Technology#introduction-to-verilog-rtl-design-and-synthesis)
**Contents:**

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false`} -->

<!-- code_chunk_output -->

- [Day-1](#day-1)
	
- [Day-2](#day-2)
 
- [Day-3](#day-3)
  
- [Day-4](#day-4)
 
- [Day-5](#day-5)
  
- [Acknowledgements](#acknowledgements)
- [References](#references)

<!-- /code_chunk_output -->

## Day-1
## Introduction to Verilog RTL Design and Synthesis

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

### Lab
1. good_mux.v
```
module good_mux (input i0 , input i1 , input sel , output reg y);
always @ (*)
begin
	if(sel)
		y <= i1;
	else 
		y <= i0;
end
endmodule 
```
![gmux](https://user-images.githubusercontent.com/93269502/139588745-c1b90b12-fd16-4f25-bab6-a0a86ff19fc8.PNG)

### Synthesis using YOSYS open-source tool

A Synthesizer is a tool used to convert the RTL Design into a netlist file (Standard Cell Format). To be more specific, a netlist is a standard gate level file that consists of nets, sequential and combinational cells and their connectivity of the corresponding RTL file coded using a HDL. In simple words, an rtl file is a code that describes the functionality of the design and a netlist is a file that expresses the same code in the form of logic cells like logic gates, flipflops, multiplexers with net connections etc.
In this workshop we use yosys as a synthesizer tool.
Command flow for synthesis using YOSYS
```
$yosys                                                                             // invokes YOSYS tool

yosys> read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib           // reads the corresponding library file

yosys> read_verilog good_mux.v                                                     // reads the Verilog script

yosys> synth -top good_mux                                                         // reads the top level module

yosys> abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib                 // converts the logic file to netlist

yosys> show          
                                                              // Final netlist output display
```
![synt](https://user-images.githubusercontent.com/93269502/139589334-b87d8627-2c2c-4354-a73a-53a95591d103.PNG)

# Day-2
# Timing libs Hierarical vs Flat Synthesis and efficient Flop Coding Styles

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

# Day-3
# Combinational and sequential optimization

  ### Combinational Logic Optimization
  ```
  1. Squeezing the logic to get most optimized design (Area and power saving)
  2. Constant Propogation (Direct optimization)
  3. Boolean logic optimization using K-map or Quin Mcksklusey.
   ```
   * Constant Propogation 
   ![Screenshot 2021-10-29 092522](https://user-images.githubusercontent.com/93269502/139372754-02854c09-b52a-430a-b1b2-ba961c46a106.png)

   * Boolean logic optimization
   Boolean logic optimization is nothing simplifying a complex boolean expression into a simplified expression by utilizing the laws of boolean logic algebra.


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

### Lab
```
$yosys

yosys> read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib           

yosys> read_verilog opt_check.v                                                     

yosys> synth -top opt_check                                                         

yosys> opt_clean -purge

yosys> abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib                    

yosys> show 
```
*we see a new code opt_clean -purge which is used to optimize the design by removing un-used net and components in the design after the design top level is synthesized using synt -top*

### *Combinational logic optimization*
 
 ![opt](https://user-images.githubusercontent.com/93269502/139582429-173b093c-8ba6-4d48-84c9-8eb64a5ae8a6.PNG)

![opt_check](https://user-images.githubusercontent.com/93269502/139582453-5b50c2c3-b92e-464c-b94b-a74bf2e95e94.png)
![opt_check2](https://user-images.githubusercontent.com/93269502/139582468-1e52039d-bbae-4ace-95a4-45ecbf538219.png)
![opt_check3](https://user-images.githubusercontent.com/93269502/139582476-4e681f87-73e8-408d-a328-1a40bb6d009a.png)
![optcheck4](https://user-images.githubusercontent.com/93269502/139582494-c0f26b08-1efe-4e4b-a68d-dbc1aa61e90a.png)
 incase we use multiple modules in a single code, we use flatten command as used in Flat Sysnthesis to perform the logic optimization of multiple modules after they are reduced to simple modules using flatten.
![mul](https://user-images.githubusercontent.com/93269502/139582650-e2a6db94-6ef8-4c6a-be7c-de64720ab880.PNG)

![multiple_module_opt](https://user-images.githubusercontent.com/93269502/139582679-4d3a9607-25d7-4dac-9997-cbde2ad237fa.png)

![multiple_module_opt2](https://user-images.githubusercontent.com/93269502/139582703-19489637-63a4-4766-aaaf-85e8ac569ebd.png)

![optimization commands](https://user-images.githubusercontent.com/93269502/139582797-2f97bee2-fb4f-4d47-8f1f-04b61e3a8aa7.png)

### *Sequential logic optimization*
```
module dff_const1(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b0;
	else
		q <= 1'b1;
end

endmodule
```
![dff_const1](https://user-images.githubusercontent.com/93269502/139583533-bc68f813-945b-4aee-9f08-08c7b4253e42.PNG)

```
module dff_const2(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b1;
	else
		q <= 1'b1;
end

endmodule
```
![dff_cons2](https://user-images.githubusercontent.com/93269502/139583609-036239ae-ab5d-48e1-9a9d-f07ae6d3cd86.PNG)

```
module dff_const3(input clk, input reset, output reg q);
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

endmodule
```
![dff_cons3](https://user-images.githubusercontent.com/93269502/139583723-754b7cb5-12f0-47e2-b321-af64ee90bc3c.PNG)

```
module dff_const4(input clk, input reset, output reg q);
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

endmodule
```
![dff_const4](https://user-images.githubusercontent.com/93269502/139583807-0b981e0a-ef7a-4893-bde2-e8d518c8f312.png)
![dff_cons4_synth](https://user-images.githubusercontent.com/93269502/139583814-b6c52288-c274-42d0-afdf-5c880543e926.png)

```
module dff_const5(input clk, input reset, output reg q);
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

endmodule
```
![dff_const5](https://user-images.githubusercontent.com/93269502/139583866-38a81e9f-24fd-4ec8-91ec-fcab4d729ca8.png)
![dff_cons5_synth](https://user-images.githubusercontent.com/93269502/139583875-ad0a60c7-45bc-4efa-92a1-4577bbc20b94.png)




# Day-4
# GLS blocking vs non blocking and Synthesis Simulation mismatch

### What is GLS?
* It stands for Gate Level Simulation.
* Running the Test Bench With Netlist as design under Test.
* Netlist is logically same as the RTL Code(Same testbench will align with the design)

### Why GLS ?
* Verify the LOgical Correctness of the design after synthesis.
* Ensuring the Timing of design is met (For this GLS needs to be run with delay annotation)

### GLS using iverilog

![GLS_iverilog](https://user-images.githubusercontent.com/93269502/139537557-826c48eb-a0c9-412a-8ee2-7e889b9fa562.png)

*Netlist has all the standard cell instantiated and the meaning of standard cell is conveyed to the iverilog by Gate Level Verilog Models.
Post which it has the same flow giving the vcd file which will generate Waveform using GTKwave.*

## Synthesis Simulation Mismatch

### Why Synthesis Simulation Mismatch happens ?

* Missing Sensitivity List
* Blocking Vs Non-Blocking Assignments
* Non Standard verilog Coding

*How does simulator work ??*

Basically the simulator will look for activity. The output of simulator changes only when its input changes

### Missing Sensitivity List

1. *bad_mux.v*
```
module mux(
input i0,input i1
input sel,
output reg y
);
always @ (sel)
begin
   if (sel)
            y = i1;
   else 
            y = i0;
            
end
endmodule
```
Step 1 : Perform iverilog simulation using RTL and tb, observe output waveforms 

Step 2 : Do synthesis and generate netlist.

Step 3 : Perform iverilog simulation using Gate level netlist and tb(same as used in step 1), observe output waveforms

Step 4 : Compare both waveforms in stage 1 and 3 , verify Synthesis-Simulation match or missmatch


Step 1 : Perform iverilog simulation using RTL and tb, observe output waveforms 

![Screenshot 2021-10-30 203137](https://user-images.githubusercontent.com/93269502/139538283-79746df6-36a3-4ec7-8cb1-64940e384700.png)

Step 2 : Do synthesis and generate netlist.

Step 3 : Perform iverilog simulation using Gate level netlist and tb(same as used in step 1), observe output waveforms
![Screenshot 2021-10-30 204303](https://user-images.githubusercontent.com/93269502/139538743-d5d3a0fb-9faf-451a-adc1-10edc63d2eab.png)

Step 4 : Compare both waveforms in stage 1 and 3 , verify Synthesis-Simulation match or missmatch
As we can observe there is a clear mismatch in step 1 and 3 waveforms, this is Synthesis simulation mismatch. The step 3 waveforms are correct but step 1 waveforms are wrong as there is missing sensitivity list in RTL code.
When we simulate it acts as a latch.But when we synthesis it will act as a Mux.

2. *good_mux.v*
```
module mux(
input i0,input i1
input sel,
output reg y
);
always @ (*)
begin
   if (sel)
            y = i1;
   else 
            y = i0;
            
end
endmodule
```
Step 1 : Perform iverilog simulation using RTL and tb, observe output waveforms
![good_mux](https://user-images.githubusercontent.com/93269502/139538997-d18767e8-cc77-40a6-b611-05e6faf77b84.png)
Step 2 : Do synthesis and generate netlist.

Step 3 : Perform iverilog simulation using Gate level netlist and tb(same as used in step 1), observe output waveforms
![ggg](https://user-images.githubusercontent.com/93269502/139539301-95e450a9-0354-451e-af6f-c91c60bb9442.png)
Step 4 : Compare both waveforms in stage 1 and 3 , verify Synthesis-Simulation match or missmatch
Observing both waveforms its a synth-simulation match

### Blocking and Non Blocking Statements

* Inside always block if we are using '=' to make assignments, it executes the Statement in the order it is written.
So the first statement is evaluated before the second statement.
* If we are using '<=' to make assignments, it executes all the RHS when always block is entered and assigns to LHS
It does Parallel Evaluation.

### Caveats with Blocking Statements
![Screenshot 2021-10-30 224556](https://user-images.githubusercontent.com/93269502/139542523-2a4d5a61-1493-47f3-8d12-e477421a04cc.png)
1. *blocking_caveat.v*

```
module code (input a,b,c
output reg y);             
always @ (*)
begin
        y = q0 & c;
        q0 = a|b ;
        
end 
endmodule
```

* RTL simulation results

![caveat](https://user-images.githubusercontent.com/93269502/139542466-28ee866e-5615-4cad-ada5-7a5403bdb985.png)

* GLS results
![cc](https://user-images.githubusercontent.com/93269502/139542809-a4495ff1-33ee-4296-bfe4-48cc985952ec.png)

We assign the value of y with q0 and with c first and then assign q0 as a or b.
So we see that q0 value is the old q0 value and is not updated.
When we simulate old q0 value will mimic a delay or flop but when we synthesis there will not be a flop.

2. 
```
module code (input a,b,c
output reg y);
reg q0;
always @ (*)
begin
        q0 = a|b ;
        y = q0 & c;
        
        
end 
endmodule
```
here we changed the order of block statement which will solve our problem and fix the problem.

# Day-5
# if, case, for loop and for generate

### IF CASE Constructs

If is used to implement a priority logic !
```
if<cond>
begin
.....
.....
end
else if<cond2>
begin
.....
.....
end
else if<cond3>
begin
.....
.....
end
else
begin
.....
.....
end
```
*The Hardware Implementation*
![cc](https://user-images.githubusercontent.com/84860957/120141063-6296ce80-c1f9-11eb-8669-a2262fc45191.JPG)

* Caution/Danger with if statement
> Inferred latches : Bad coding style (incomplete if statement with missing else)




*Sometimes we need the inferred latch like in the case of counters if there is no enable the counter should latch on to the previous value.*


### Case Statement
*if and case statement are used inside the always block*,
whatever variable we are trying to assign inside if or case statement should be a register variable

Hardware implementation of Case statement is a MUX.
```
reg y
always @ (*)
begin
	case(sel)
		2'b00:begin
		      ....
		      end
		2'b01:begin
		      ....
		      end
		      .
		      .
		      .
	endcase	
```
* Caution with case statement
  
  inferred latches : occurs due to incomplete case statements (bad coding style). 
  *Always use default statement to avoid inferred latches*



### lab 

1. incomplete if :
![inc](https://user-images.githubusercontent.com/93269502/139569224-59e99d84-716b-4f13-a1aa-6d23bf4637e1.png)
 
 ![Screenshot 2021-10-31 110521](https://user-images.githubusercontent.com/93269502/139569393-d2ff694e-8c9b-4df2-af17-b881768c4025.png)

 ![dd](https://user-images.githubusercontent.com/93269502/139569548-c570217f-d50b-490a-a59c-1f8b38dc0282.png)

2. incomplete if2 :

![if2](https://user-images.githubusercontent.com/93269502/139569605-f8abcd8c-5d5a-4ef2-ada1-f049581082d0.png)
![exp](https://user-images.githubusercontent.com/93269502/139569713-3f8f9adf-46a4-4cb9-a786-94816388d96b.png)

![inc2](https://user-images.githubusercontent.com/93269502/139569910-9be4c6f2-cbe5-48cb-b3a8-9f9044cfb7c0.PNG)

![Capture](https://user-images.githubusercontent.com/93269502/139570684-8818277f-edb1-4d89-81ff-a7a3f4f20310.PNG)

3. incomplete case :
```
module incomp_case (input i0 , input i1 , input i2 , input [1:0] sel, output reg y);
always @ (*)
begin
	case(sel)
		2'b00 : y = i0;
		2'b01 : y = i1;
	endcase
end
endmodule
```


![csl](https://user-images.githubusercontent.com/93269502/139571840-401c253b-c823-4879-b9cc-f2d6e035957f.PNG)

![synth_case](https://user-images.githubusercontent.com/93269502/139572065-49a56095-2b4e-4b17-9d73-40bbe945e8ea.PNG)

4. complete case (with default)
```
module comp_case (input i0 , input i1 , input i2 , input [1:0] sel, output reg y);
always @ (*)
begin
	case(sel)
		2'b00 : y = i0;
		2'b01 : y = i1;
		default : y = i2;
	endcase
end
endmodule
```
![comp_case](https://user-images.githubusercontent.com/93269502/139572357-4b63bcf8-b9e2-4456-933e-8951764216b8.PNG)

![cmpl](https://user-images.githubusercontent.com/93269502/139572534-c8837d82-53a8-4b86-bfe7-e1e9c078c7fb.PNG)

5. partial_case_assign
```
module partial_case_assign (input i0 , input i1 , input i2 , input [1:0] sel, output reg y , output reg x);
always @ (*)
begin
	case(sel)
		2'b00 : begin
			y = i0;
			x = i2;
			end
		2'b01 : y = i1;
		default : begin
		           x = i1;
			   y = i2;
			  end
	endcase
end
endmodule
```

![pArt](https://user-images.githubusercontent.com/93269502/139573331-4e6566b1-5299-4bef-8908-eaa0c6e0a6aa.PNG)

### Looping Constructs
There are 2 types of loops

* For Loop
Used inside always block,
used for evaluating expressions,
not used for instantiating hardware.

* Generate For Loop
Used outside always block,
cannot be used inside always,
used for instantiating hardware.
Used to replicate the hardware.

### LAB 

1. mux_generate.v
```
module mux_generate (input i0 , input i1, input i2 , input i3 , input [1:0] sel  , output reg y);
wire [3:0] i_int;
assign i_int = {i3,i2,i1,i0};
integer k;
always @ (*)
begin
for(k = 0; k < 4; k=k+1) begin
	if(k == sel)
		y = i_int[k];
end
end
endmodule
```

![mux](https://user-images.githubusercontent.com/93269502/139579059-1cc380b0-9f3a-4587-9b5c-b8a28416b5e1.PNG)

Synthesis results
![muxxx](https://user-images.githubusercontent.com/93269502/139579285-c1aa0f78-8666-4a0d-b98e-f252bb225fb2.PNG)

2. demux_case.v
```
module demux_case (output o0 , output o1, output o2 , output o3, output o4, output o5, output o6 , output o7 , input [2:0] sel  , input i);
reg [7:0]y_int;
assign {o7,o6,o5,o4,o3,o2,o1,o0} = y_int;
integer k;
always @ (*)
begin
y_int = 8'b0;
	case(sel)
		3'b000 : y_int[0] = i;
		3'b001 : y_int[1] = i;
		3'b010 : y_int[2] = i;
		3'b011 : y_int[3] = i;
		3'b100 : y_int[4] = i;
		3'b101 : y_int[5] = i;
		3'b110 : y_int[6] = i;
		3'b111 : y_int[7] = i;
	endcase

end
endmodule
```
![dem](https://user-images.githubusercontent.com/93269502/139579946-e1fb1e68-6dea-46e8-ae06-3a333969d893.PNG)

3. demux_generate.v
```
module demux_generate (output o0 , output o1, output o2 , output o3, output o4, output o5, output o6 , output o7 , input [2:0] sel  , input i);
reg [7:0]y_int;
assign {o7,o6,o5,o4,o3,o2,o1,o0} = y_int;
integer k;
always @ (*)
begin
y_int = 8'b0;
for(k = 0; k < 8; k++) begin
	if(k == sel)
		y_int[k] = i;
end
end
endmodule
```
![dem](https://user-images.githubusercontent.com/93269502/139580194-6c744d4c-2201-4de8-9258-191fed66bdea.PNG)

4. Ripple carry adder
rca.v
```
module rca (input [7:0] num1 , input [7:0] num2 , output [8:0] sum);
wire [7:0] int_sum;
wire [7:0]int_co;

genvar i;
generate
	for (i = 1 ; i < 8; i=i+1) begin
		fa u_fa_1 (.a(num1[i]),.b(num2[i]),.c(int_co[i-1]),.co(int_co[i]),.sum(int_sum[i]));
	end

endgenerate
fa u_fa_0 (.a(num1[0]),.b(num2[0]),.c(1'b0),.co(int_co[0]),.sum(int_sum[0]));


assign sum[7:0] = int_sum;
assign sum[8] = int_co[7];
endmodule
```
fa.v
```
module fa (input a , input b , input c, output co , output sum);
	assign {co,sum}  = a + b + c ;
endmodule
```
![rca](https://user-images.githubusercontent.com/93269502/139581247-e05a98c6-b62f-4711-8854-691210adc9e6.PNG)

*Note*
>>  Rules for addition

* N and N bit number --> Sum will be N+1 bit
* N and M bit number --> Sum will be Max(N,M)+1 bit

# Acknowledgements

* [Kunal Ghosh](https://github.com/kunalg123)

# References

* https://www.vlsisystemdesign.com/rtl-design-using-verilog-with-sky130-technology/?q=%2Frtl-design-using-verilog-with-sky130-technology%2F&v=a98eef2a3105
* https://github.com/kunalg123/vsdflow
* https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop
