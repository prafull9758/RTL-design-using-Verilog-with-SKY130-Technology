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

 


