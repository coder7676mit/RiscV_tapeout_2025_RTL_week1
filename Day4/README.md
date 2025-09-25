## 4. Day 4 - Gate Level Simulation(GLS), Blocking vs Non-blocking and Synthesis-Simulation Mismatch

### 4.1.GLS, Synthesis-Simulation mismatch and Blocking/Non-blocking statements
**What is Gate Level Simulation (GLS) ?**

Running the testbench against the synthesized netlist ouput as a DUT is known as Gate Level Simulation (GLS). The Output netlist should logically be same as the RTL code so that the testbench will align itself when we simulate both the files to obtain the waveforms.

**_Advantages of GLS:_**

 *To logically verify the correctness of the design after Synthesis.
 *During the RTL Simulation, timing was not accounted. But for practical applications, there is a need to ensure the timing of the design to be met.
 
**Why GLS?**

GLS is required to verify the logical correctness of the design post synthesis with the help of the netlist file. It ensures whether the timing of the design is met and for thi, the GLS used to run with delay annotations.

<img width="400" alt="GLS" src="https://user-images.githubusercontent.com/93824690/166241284-6638d96e-4e21-4c93-a238-ed43834ecf37.png">

```
//consider a netlist
and uand (.a(a),.b(b))
or uor (.a(a),.b(b))
//There is a need to define the meaning of and and or
//Thus we need netlist, testbench and verilog models of the standard cells
```

>_Netlist consists of all standard cells instantiated and it's meaning is conveyed to the iVerilog using Gate Level Verilog Models. Gate Level Verilog Models can be functional or timing aware. If the gate level models are delay annotated, then GLS can be performed for timing validation also in addition to functional validation._

### SYNTHESIS SIMULATION MISMATCH

If netlist is a true reciprocation of RTL, what is the need to validate the functionality of netlist? There may be synthesis and simulation mismatch due to the following reasons:

**(I)_Missing Sensitivity List_**
**(II)_Blocking Vs Non Blocking Assignments_**
**(III)_Non Standard Verilog Coding_**

**(I)Missing Sensitivity List**

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
The output of Simulator changes only when the input changes. The output is not evaluated when there is no activity. In the above 2x1 mux code, when select is changing (when select is 1), the output is 1 when input is 1 else the output is 0. The always block evaluates only when there is a transition change in select pin, and is not sensitive (output does not reflect) to changes in the inputs 0 and 1.

**_Corrected code for missing sensitivity list:_**
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
>_mismatch is corrected by having always @ (*) where the always block is evaluated when any signal changes. So, any changes in inputs will also be seen in the output._

#### (II)Blocking Vs Non Blocking Assignments in Verilog

Blocking and Non-blocking statements are procedural assignment statements that can be implemented only inside an always block.

*Blocking Assignments --> =
   *Executes the statements in the order in which they are coded.
   
*Non-blocking Assignments --> <=
   *Executes the RHS of all such assignments when the always block is entered and assigned to LHS in a parallel evaluation.
   
_Synthesis-Simulation mismatches due to incorrect ordering of the blocking assignments done inside an always block._

#### CAVEAT 1:-
```
module code (input clk,input reset,
input d,
output reg q);
always @ (posedge clk,posedge reset)
begin
if(reset)
begin
        q0 = 1'b0;
        q = 1'b0;
end
else
        q = q0;
        q0 = d;    
end
endmodule
```
<img width="400" alt="cav1" src="https://user-images.githubusercontent.com/93824690/166245183-8f2608db-88cb-4966-beba-5038ff45390b.png">

>_The assignments inside the code represent the blocking statements. q0 and q are assigned to 1 bit 0s - so asynchronous reset connection happens. However, in the later parts, q0 is assigned to q and then d gets assigned to q0. If suppose, there is a change in the code._
```
module code (input clk,input reset,
input d,
output reg q);
always @ (posedge clk,posedge reset)
begin
if(reset)
begin
        q0 = 1'b0;
        q = 1'b0;
end
else
        q0 = d;
        q = q0;    
end
endmodule
```
>_In this case, d is assigned to q0 and then q0 is assigned to q. So, by the time the second statment gets executed, q0 has the value of d. This will lead to implementation of only one flop. Previously, q has the value of q0 and q0 has the value of d - which lead to implementation of 2 storage elements._

#### _Always Use Non- Blocking Statements when writng the Sequential circuits code_

#### CAVEAT 2:- Causing Synthesis Simulation Mismatch
```
module code (input a,b,c
output reg y);
reg q0;
always @ (*)
begin
        y = q0 & c;
        q0 = a|b ;    
end 
endmodule
```
>_The code is aimed to create a function of y = (A+B).C. In the above code, when the code enters always block, due to the presence of blocking statements, they get evaulated in order. So y gets evaluated first (q0.C), where the q0 results corresponds to the previous iteration's result. The q0 value gets updated only in the second statement._

When the order of the statements is changed: In this case, a OR b is evaluated first and the latest value is used for calculating y.
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

**> Therefore there is a paramount importance to run the GLS on the netlist and match the specifications, to ensure there is no simulation synthesis mismatch.**

#### (i) ternary_operator.v
>_Note: Mux function is written using a ternary operator. Ternary operator takes 3 operands with the format.
```
<Condition>?<True>:<False>
```
**_Verilog File_**

<img width="641" alt="Screenshot (288)" src="https://user-images.githubusercontent.com/93824690/166249218-f94c6544-cef7-4861-853b-9e99f0f250f5.png">

**_GTK Wave_**

<img width="641" alt="Screenshot (237)" src="https://user-images.githubusercontent.com/93824690/166249367-e4b5567f-f683-4a5e-89fb-b875ad6eeaf4.png">

**_Statistics_**

<img width="400" alt="ter st" src="https://user-images.githubusercontent.com/93824690/166249983-99797adb-801c-43d4-983d-4c8154f34cbd.png">

**_Realization of Logic_**

<img width="641" alt="Screenshot (238)" src="https://user-images.githubusercontent.com/93824690/166249396-10ea2950-f074-4f53-b8e1-296fea18491d.png">

>_NAND gate with i1 and sel, inverted io and Or to And invert gate, to which the inputs are sel and inverted i0. The output y is given by the expression = sel'.i0 + sel.i1_


**_GLS OUTPUT_**

<img width="700" alt="gls gtk" src="https://user-images.githubusercontent.com/93824690/166250831-c7b446ef-d96e-46a0-893c-edf6e9780d30.png">

#### MISSING SENSITIVITY LIST

#### (ii): bad_mux.v showing mismatch due to missing sensitivity list

**_Verilog File_**

<img width="400" alt="bad mo" src="https://user-images.githubusercontent.com/93824690/166252999-87ca21b7-ef7e-4f61-98fa-3167fa3be26f.png">

>_during Simulation, the logic acts as a latch and during synthesis, it acts as a mux._

**_GTK Wave_**

<img width="700" alt="bad gtk" src="https://user-images.githubusercontent.com/93824690/166253027-c182567f-82ff-483a-af77-aabdf6e1d2cd.png">

**_Synthesis Statistics_**

<img width="400" alt="bad st" src="https://user-images.githubusercontent.com/93824690/166253012-6caad5c4-e707-454b-b2f8-d5f1f58885cd.png">

**_GLS Output_**


<img width="700" alt="bad gls" src="https://user-images.githubusercontent.com/93824690/166253052-a961d8b0-dc45-4b4a-a90e-ff5409fa0448.png">


**_Realization of Logic_**

<img width="641" alt="Screenshot (241)" src="https://user-images.githubusercontent.com/93824690/166253100-20d54e2c-58b6-479c-bb7a-296833f7d4ee.png">

>_Confirms the functionality of 2x1 mux after synthesis where when the select is low, activity of input 0 is reflected on y. Similarly, when the select is hight, activity of input 1 is reflected on y. Hence there is a synthesis simulation mismatch due to missing sensitivity list._

#### CAVEATS IN BLOCKING ASSIGNMENTS

#### (iii) blocking_caveat.v showing mismatch due to blocking assignments

**_Verilog File_**

<img width="400" alt="cav mo" src="https://user-images.githubusercontent.com/93824690/166254558-27b5559d-cf33-4089-9863-116efb61f9f3.png">

>_when the code enters always block, due to the presence of blocking statements, they get evaulated in order. So d gets evaluated first (x.c), where the x results corresponds to the previous iteration's result (a|b). The d value gets updated only in the second statement. The output expression is given as d = (a+b).c_

**_Synthesis Statistics_**

<img width="400" alt="cav st" src="https://user-images.githubusercontent.com/93824690/166254570-e078ddec-8f95-4aba-ac01-4d552eb742a3.png">

**_GTK Wave_**

<img width="700" alt="cav gtk" src="https://user-images.githubusercontent.com/93824690/166254577-420c3b5f-2e2e-43ba-a670-f41a828e0e75.png">

>_d = (a+b).c, if the inputs a,b = 0; then a+b = 0. The output d = 0. But, we observe the output d = 1 because it looks at the past value where a+b was 1._

**_GLS Output_**

<img width="700" alt="cav gls" src="https://user-images.githubusercontent.com/93824690/166254673-5b2ff6aa-4aea-4ca1-bb06-28079dc50910.png">

**_Realization of Logic_**

<img width="750" alt="cav re" src="https://user-images.githubusercontent.com/93824690/166254689-2095c677-6fe6-4264-b840-880b8017675b.png">

<_value of output d is 0 after simulation and 1 after synthesis for the same set of input values. Hence there is a synthesis simulation mismatch due to blocking assignments._

------------------------
