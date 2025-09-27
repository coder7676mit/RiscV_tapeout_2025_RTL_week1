## 2. Day 2 - Timing Libs, Hierarchial Vs Flat Synthesis and Efficient Flop Coding Styles
### 2.1. Introduction to timing labs
```
__Command to open the libary file
$ gvim ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
__To shut off the background colors/ syntax off:
: syn off
__To enable the line numbers
: se nu
```

#### Contents

For a design to work, there are three important parameters that determines how the Silicon works: Process (Variations due to Fabrications), Voltage (Changes in the behavior of the circuit) and Temperature (Sensitivity of semiconductors). Libraries are characterized to model these variations. 

![alt](../Day2/lib_file.jpeg)

#### Various Flavours of AND Cell

![alt](../Day2/gates_descrip.jpeg)

### 2.2. Hierarchial synthesis vs Flat synthesis 

#### Hierarchial synthesis  
````
_Opening the file used for this experiment
$ gvim multiple_modules.v
_Invoke Yosys
$ yosys
_Read library 
$ read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
_Read Design
$ read_verilog multiple_modules.v
_Synthesize Design
$ synth -top multiple_modules
_Generate Netlist
$ abc -liberty ../lib/sky130_fd_sc_hd__t_025C_1v80.lib
_Realizing Graphical Version of Logic for multiple modules
$ show multiple_modules
_Writing the netlist in a crisp manner 
$ write_verilog -noattr multiple_modules_hier.v
$ !gvim multiple_modules_hier.v
````
**Multiple Modules:** - 2 SubModules
**Staistics of Multiple Modules**

<img width="641" alt="Screenshot (165)" src="https://user-images.githubusercontent.com/93824690/166205646-1597fd0a-12e7-4244-b0a7-875b36e8366b.png">


**Netlist file**

![alt](../Day2/hierarchical_synth.png)

#### Flat synthesis  

```
_To flatten the netlist
$ flatten
_Writing the netlist in a crisp manner and to view it
$ write_verilog -noattr multiple_modules_flat.v
$ !gvim multiple_modules_flat.v
```
**Realization of the Logic**

<img width="750" alt="Screen Shot 2021-09-02 at 6 14 16 PM" src="https://user-images.githubusercontent.com/89927660/131927662-d25c4d37-c0c1-41a7-ab14-9399840eb3ee.png">
  
 
**Netlist file**

![alt](../Day2/hierar_vs_flatten.png)

#### SUB MODULE LEVEL SYNTHESIS

Sub-module level synthesis is preferred when there are multiple instances of same module. Sythesizing the same module over several times may not be advantageous with respect to time. Instead, synthsis can be performed for one module, its netlist can be replicated and then stitched together in the top module. This is also used particulary in massive designs using divide and conquer method. 

**Statistics of Sub-module**

<img width="300" alt="Screen Shot 2021-09-02 at 9 11 28 PM" src="https://user-images.githubusercontent.com/89927660/131940332-d8272cc3-affb-471d-9dd0-1ad951d86c22.png">


**NetList File of Sub-module**

![alt](../Day2/submod1_netl.png)


### 2.3. Various Flop coding styles and optimization
In a digital design, when an input signal changes state, the output changes after a propogation delay. All logic gates add some delay to singals. These delays cause expected and unwanted transitions in the output, called as _Glitches_ where the output value is momentarily different from the expected value. An increased delay in one path can cause glitch when those signals are combined at the output gate. In short, more combinational circuits lead to more glitchy outputs that will not settle down with the output value. 

#### Flip flop overview
A D flip-flop is a sequential element that follows the input pin d at the clock's given edge. D flip-flop is a fundamental component in digital logic circuits.
There are two types of D Flip-Flops being implemented: Rising-Edge D Flip Flop and Falling-Edge D Flip Flop.

<img width="400" alt="fe verilog-d-flip-flop" src="https://user-images.githubusercontent.com/93824690/166153680-a44e9aa1-4056-4cfe-8a09-a096e3da9dc1.png">

Every flop element needs an initial state, else the combinational circuit will evaluate to a garbage value. In order to achieve this, there are control pins in the flop namely: Set and Reset which can either be Synchronous or Asynchronous. 

#### _Asynchronous Reset/Set:_

<img width="641" alt="Aset" src="https://user-images.githubusercontent.com/93824690/166290793-db957126-074e-4036-a224-4bd71c4ee38a.png">

<img width="641" alt="Aset2" src="https://user-images.githubusercontent.com/93824690/166290797-81e1f637-55bc-4e8b-b24b-3aeaa6e7e848.png">


>_ Here, always block gets evaluated when there is a change in the clock or change in the set/reset.The circuit is sensitive to positive edge of the clock. Upon the signal going low/high depending on reset or set control, singal q line goes changes respectively. Hence, it does not wait for the positive edge of the clock and happens irrespective of the clock_.

#### _Synchronous Reset:_

<img width="641" alt="syn 1" src="https://user-images.githubusercontent.com/93824690/166290831-db5d6a70-d4d3-4cd1-b8be-01db5fb1fe2f.png">


#### _Both Synchronous and Asynchronous Reset:_

<img width="641" alt="both" src="https://user-images.githubusercontent.com/93824690/166290817-8e06a04d-df32-4ce4-9eb1-8d5490036e36.png">


#### FLIP FLOP SIMULATION

```
#Steps Followed for analysing Asynchronous behavior:
//Load the design in iVerilog by giving the verilog and testbench file names
$ iverilog dff_asyncres.v tb_dff_asyncres.v 
//List so as to ensure that it has been added to the simulator
$ ls
//To dump the VCD file
$ ./a.out
//To load the VCD file in GTKwaveform
$ gtkwave tb_dff_asyncres.vcd
```
**GTK WAVE OF ASYNCHRONOUS RESET**

![alt](../Day2/asyn_rst_gtk.png)

#### FLIP FLOP SYNTHESIS

```
_Invoke Yosys
$ yosys
_Read library 
$ read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
_Read Design
$ read_verilog dff_asyncres.v
_Synthesize Design - this controls which module to synthesize
$ synth -top dff_asyncres
_There will be a separate flop library under a standard library
_But here we point back to the same library and tool looks only for DFF instead of all cells
$ dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
_Generate Netlist
$ abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
_Realizing Graphical Version of Logic for single modules
$ show 
_Writing the netlist in a crisp manner 
$ write_verilog -noattr dff_asyncres_ff.v
$ !gvim dff_asyncres_ff.v
```
**Statistics of D FLipflop with Asynchronous Reset**

<img width="400" alt="Screenshot (185)" src="https://user-images.githubusercontent.com/93824690/166207965-d6027bd0-b4be-4bd9-9e98-fff90892aae3.png">

**Realization of Logic**

![alt](../Day2/async_flop.png)

**Statistics of D FLipflop with Asynchronous set**

<img width="400" alt="Screenshot (188)" src="https://user-images.githubusercontent.com/93824690/166207987-427e675c-d2d2-4677-949f-5eb27b18ce03.png">

**GTK WAVE OF ASYNCHRONOUS SET**

![alt](../Day2/async_set_gtk.png)

**Statistics of D FLipflop with Synchronous Reset**

<img width="400" alt="Screenshot (190)" src="https://user-images.githubusercontent.com/93824690/166209133-d0fef1ae-39d3-43e3-8ab3-25d746b881a1.png">

**Realization of Logic**

![alt](../Day2/sync_flop.png)


#### Interesting Optimizations
```
modules used are opened using the command
$ gvim mult_*.v -o
_Invoke Yosys
$ yosys
_Read library 
$ read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
_Read Design
$ read_verilog mult_2.v
_Synthesize Design - this controls which module to synthesize
$ synth -top mul2
_Generate Netlist
$ abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
_Realizing Graphical Version of Logic for single modules
$ show 
_Writing the netlist in a crisp manner 
$ write_verilog -noattr mult_2.v
$ !gvim mult_2.v
```
## (i) mult_2.v 

**_Expected Logic_**

<img width="400" alt="Screenshot (195)" src="https://user-images.githubusercontent.com/93824690/166213003-2cf85b52-fd17-4f12-a404-b86da6d559fb.png">


 ##### No hardware requirements - No # of memories, memory bites, processes and cells. Number of cells inferred is 0.
 
 **_NetList File of Sub-module_**
 
![alt](../Day2/spec_mul2nett.png)

 **_Realization of Logic_**
 
![alt](../Day2/special_case.png)


## (ii) mult_8.v

**_Expected Logic_**

<img width="400" alt="Screenshot (194)" src="https://user-images.githubusercontent.com/93824690/166212974-d283029d-43ec-45be-bb02-6769eb3331f9.png">
<img width="300" alt="Screen Shot 2021-09-04 at 4 19 20 AM" src="https://user-images.githubusercontent.com/89927660/132089537-ea9225f5-00ed-462f-b61e-208a3ae8d25e.png">


 **_NetList File of Sub-module_**
 
![alt](../Day2/spec_mul9net.png)


 **_Realization of Logic_**
 
![alt](../Day2/spec_case.png)

----------------------------------------------------------------------------------------------------------------------------------------------------------

