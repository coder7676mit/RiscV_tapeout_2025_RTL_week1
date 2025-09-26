## 5. Day 5 - Optimization in synthesis

### 5.1 If Case constructs
The if statement is a conditional statement which uses boolean conditions to determine which blocks of verilog code to execute. If always translates into Multiplexer. It is used for priority Logic and are always used inside always block.The variable should be assigned as a register.

**_Syntax for IF Statement_**
```
if<cond>
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
**_Syntax for IF- ELSE-IF Statement_**
```
if<cond1>
begin
.....
 executes cb1
.....
end
else if<cond2>
begin
.....
  executes cb2
.....
end
else if<cond3>
begin
.....
  executes cb3
.....
end
else
begin
.....
  executes cb4
.....
end
```
**_Hardware Implementation_**

<img width="400" alt="hd imp" src="https://user-images.githubusercontent.com/93824690/166259138-8166cfbc-1ae2-4260-87ad-4aca82c56c94.png">

**_Cautions with using IF Statements_**
Inferred latches can serve as a 'warning sign' that the logic design might not be implemented as intended. They represent a bad coding style, which happens because of incomplete if statements/crucial statements missing in the design. For ex: if a else statement is missing in the logic code, the hardware has not been informed on the decision, and hence it will latch and will tried retain the value. This type of design should be completely avoided unless intended to meet the design functionality (ex: Counter Design).

<img width="400" alt="inf if" src="https://user-images.githubusercontent.com/93824690/166259507-9a71c456-5108-42dc-ad1c-1ebb6e9b0d6f.png">

#### CASE Statements
The hardware implementation is a Multiplexer. Similar to IF Statements, Case statements are also used inside always block and the variable should be a register variable.

**Case Statements**
```
_reg y
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
end
```
**Caveats in CASE Statements**

*Case statements are dangerous when there is an incomplete Case Statement Structure may lead to inferred latches. To avoid inferred latches, code Case with default conditions. 
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
	default:begin
         ....
		       end
	endcase	
end
```
*Partial Assignments in Case statements - not specifying the values. This will also create inferred latches. To avoid inferred latches, assign all the inputs in all the segments of the case statement.

### 5.2 INCOMPLETE IF STATEMENTS

#### * (Case 1) incomplete if statements

**_Verilog File_**

<img width="400" alt="100" src="https://user-images.githubusercontent.com/93824690/166263361-bd07fa49-2177-47a7-96d2-5a23e67e9a30.png">

**_GTK Wave_**

<img width="" alt="Screenshot (250)" src="https://user-images.githubusercontent.com/93824690/166262139-4b40b5d0-55e9-448b-968f-1bc5dbd59c4a.png">

>_Else case is missing so there will be a D latch._

**_Synthesis Statistics_**																		  
<img width="400" alt="Screenshot (251)" src="https://user-images.githubusercontent.com/93824690/166262175-01dc3362-8128-43a2-bda4-14be20b26d7d.png">

**_Realization of Logic_**
<img width="700" alt="Screenshot (252)" src="https://user-images.githubusercontent.com/93824690/166262206-b46f9ea8-6fec-4c6f-a1ce-bae4dafc566a.png">

>_synthesized design has a D Latch inferred due to incomplete if structure (missing else statement)._

#### * (Case2) incomplete if statements

**_Verilog File_**

<img width="641" alt="200" src="https://user-images.githubusercontent.com/93824690/166263847-c791f930-eeb1-45ff-8052-f1d247b8bebd.png">

**_GTK Wave_**

<img width="700" alt="Screenshot (254)" src="https://user-images.githubusercontent.com/93824690/166262258-db09ddf3-47d0-44d3-888c-6b45b56d1cdf.png">

>_When i0 is high, the output follows i1. When i0 is low, the output latches to a constant value (when both i0 and i2 are 0). Presence of inferred latches due to incomplete if structure._

**_Synthesis Statistics_**

<img width="400" alt="Screenshot (256)" src="https://user-images.githubusercontent.com/93824690/166262277-2398c16a-7007-4b1e-937d-398989c891f6.png">

**_Realization of Logic_**

<img width="700" alt="Screenshot (257)" src="https://user-images.githubusercontent.com/93824690/166262309-0c66e904-20fb-49ba-859a-97667ca61996.png">


### 5.2 INCOMPLETE CASE STATEMENTS

#### CASE 1: incomplete case statements
**_Verilog File_**

<img width="641" alt="33" src="https://user-images.githubusercontent.com/93824690/166265632-97eb0312-e58f-40e6-9f26-fa6b54abb8ca.png">

**_GTK Wave_**
<img width="700" alt="Screenshot (258)" src="https://user-images.githubusercontent.com/93824690/166264918-dab97bee-102e-43b2-9887-d8d07419b281.png">

>_When select signal is 00, the output follows i0 and is i1 when the select value is 01. Since the output is undefined for 10 and 11 values, the ouput latches to the previously available value._

**_Synthesis Statistics_**

<img width="400" alt="i st" src="https://user-images.githubusercontent.com/93824690/166266006-0a03e9e8-0fc9-45bf-be6a-b82edbaa6f5c.png">

**_Realization of Logic_**

<img width="750" alt="Screenshot (259)" src="https://user-images.githubusercontent.com/93824690/166264934-397bc9a5-7e18-419c-898b-14d910aeafe5.png">

>_The synthesized design has a D Latch inferred due to incomplete case structure (missing output definition for 2 of the select statements)._

#### CASE 2: incomplete case statements with default

**_Verilog File_**

<img width="641" alt="comp case mo" src="https://user-images.githubusercontent.com/93824690/166266795-cf2d4b1c-76f3-434f-a86a-873ead2f66e8.png">

**_Synthesis Statistics_**

<img width="400" alt="comp st" src="https://user-images.githubusercontent.com/93824690/166266812-2ae22097-2241-49a3-8a10-96bab1d0ff09.png">

**_GTK Wave_**

<img width="700" alt="Screenshot (260)" src="https://user-images.githubusercontent.com/93824690/166264948-65dc25ba-0bdd-4ff2-b6de-8b3616d665f5.png">

>_When select signal is 00, the output follows i0 and is i1 when the select value is 01. Since the output is undefined for 10 and 11 values, the presence of default sets the output to i2 when the select line is 10 or 11. The ouput will not latch and be a proper combinational circuit._
**_Realization of Logic_**

<img width="750" alt="Screenshot (261)" src="https://user-images.githubusercontent.com/93824690/166264984-ce6f0d0d-473f-409c-9149-f37b5b82234b.png">

#### CASE 3: partial case statement
**_Verilog File_**

<img width="641" alt="p mo" src="https://user-images.githubusercontent.com/93824690/166267883-dfbcec82-f803-458d-bec6-81cd09ad7d4e.png">

**_Synthesis Statistics_**

<img width="400" alt="p st" src="https://user-images.githubusercontent.com/93824690/166267889-5976618e-4223-470e-b9ad-2a33636b8ba6.png">

**_Realization of Logic_**

<img width="781" alt="p re" src="https://user-images.githubusercontent.com/93824690/166267900-ca0787fd-c00b-4a33-96b4-7ea15e5f1c41.png">


### 5.3 STATEMENTS USING FOR

_Understanding the Usage of For and Generate Statements:_

|FOR STATEMENTS|GENERATE STATEMENTS|
|:---:|:---:|
|These statements are used inside the always block|These statements are used outsde the always block|
|Used for evaluating expressions|Used for instantiating/replicating Hardwares|

#### CASE 1: using generate if statement

**_Verilog File_**

<img width="641" alt="1 mo" src="https://user-images.githubusercontent.com/93824690/166277488-bee0a9c4-dc04-4d27-a4fb-28a255550110.png">

**_GTK Wave_**

<img width="750" alt="1 gtk" src="https://user-images.githubusercontent.com/93824690/166269945-ad59baba-f1d2-46a5-94e5-f1bbe2c33c57.png">

#### CASE 2: demux using case statement.v

**_Verilog File_**

<img width="641" alt="2 mo" src="https://user-images.githubusercontent.com/93824690/166270108-568450d2-3a12-4a2c-8a6b-fd9985507a16.png">

**_GTK Wave_**

<img width="750" alt="2 gtk" src="https://user-images.githubusercontent.com/93824690/166270133-35d829cf-49c9-4dd3-bb01-f4c6e3ad7c40.png">

**_Synthesis Statistics_**

<img width="400" alt="2 stk" src="https://user-images.githubusercontent.com/93824690/166270162-b0ac5bc1-e795-427c-8ba2-ef292afaa420.png">

**_Realization of Logic_**

<img width="780" alt="2 re" src="https://user-images.githubusercontent.com/93824690/166270182-340652d4-b730-4d3f-aa08-e41d6a3a8657.png">

**_GLS Output_**

<img width="750" alt="2 gls" src="https://user-images.githubusercontent.com/93824690/166270199-ac2a299b-d0c1-43af-90f8-47070c8dcaaf.png">

#### CASE 3: demux using generate if statement.v 

**_Verilog File_**

<img width="641" alt="3 mo" src="https://user-images.githubusercontent.com/93824690/166270221-3f80357f-b973-46b2-b406-a629f891de48.png">

**_GTK Wave_**

<img width="750" alt="3 gtk" src="https://user-images.githubusercontent.com/93824690/166270250-42c21b20-c041-4a00-b594-5152620a00ac.png">

**_Synthesis Statistics_**

<img width="400" alt="3 st" src="https://user-images.githubusercontent.com/93824690/166270278-9deb11bb-847f-4d8f-9917-3ddddba214e4.png">

**_Realization of Logic_**

<img width="750" alt="3 re" src="https://user-images.githubusercontent.com/93824690/166270294-71548a2e-6f23-49bb-9aa5-a39e7d1bf1d0.png">

**_GLS Output_**

<img width="750" alt="3 gls" src="https://user-images.githubusercontent.com/93824690/166270307-c3b44ff7-47eb-43c3-a2dd-8fd28150234e.png">


### 5.3 STATEMENTS USING GENERATE

**_Experiment on Ripple Carry Adder_**
>_Instantiating the full adder in a loop to replicate the hardware_
**_Verilog File_**

<img width="641" alt="1 mo" src="https://user-images.githubusercontent.com/93824690/166288993-0b0dbf2a-6e77-4510-b38c-9aa11013dedf.png">

**_GTK Wave_**

<img width="750" alt="1 gtk" src="https://user-images.githubusercontent.com/93824690/166289014-d5cc1b41-f990-48d8-99b0-acf9813e3f45.png">

**_Synthesis Statistics_**

<img width="400" alt="1 st" src="https://user-images.githubusercontent.com/93824690/166289026-0906729e-2892-436e-b297-b176b97c0fc2.png">

**_Realization of Logic - rca_**

<img width="850" alt="1 re" src="https://user-images.githubusercontent.com/93824690/166289044-113508a0-3f96-4aca-9edf-3f492b0f265d.png">

**_Realization of Logic- fa_**

<img width="850" alt="1 re2" src="https://user-images.githubusercontent.com/93824690/166289066-d06cbd35-015e-4dfb-baf9-138399256918.png">

**_GLS Output_**

<img width="750" alt="1 gls" src="https://user-images.githubusercontent.com/93824690/166289126-d0980b69-c9d8-4e66-8011-631610d98480.png">

