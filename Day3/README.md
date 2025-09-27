
# Day 3
---
## 1. Different Optimization Methods
### 1. Combinatonal Logic Opitmizations

Combinational logic is squeezed in order so as to save area and power consumption. Different methods of combibnational logic optimization are:

#### 1. Constant Propagation 
 - If a variable is taking some value, then that value is propagted through the design to see what all changes it bring. Finally we get a reduced hardware.

#### 2. Boolean Logic Optimization 
 - The boolean logic associated with the hardware can be converted to reduced form.
   
---

### 2. Sequential Logic Optimization

#### Sequential Constant Propagation
 - If the output of a Flip-Flop is turning out to be a constant, due to some constraints in input or set/reset pin, this constant can be propagated through design and the design can be simplified.

##### Other Methods:
 - State Optimization: Simplifying the design if there are unsued states involved in the design.
 - Cloning: Duplicate logical cells and redistribute connections to balance load.
 - Retiming: Moving combinational logic sections across registers in such a way that overall functionality is not changed. This can reduce the delay between flip-flops and can increase the effective frequency.

---

## 2. Labs on optimization
### Lab1

Module for Lab1:
```verilog
module opt_check (input a , input b , output y);
	assign y = a?b:0;
endmodule
```
Explanation:
 - The output ```y``` is equal to input ```b``` when input ```a``` is high, else it is 0.
 - Here the logic can simplified as ```y=a AND b```.

Do synthesis with yosys as done before. For optimizing execute this command after ```synth -top opt_check```:
```
 $ opt_clean -purge
```

<img src="images/opt_check.png" alt="Alt Text" width="500"/>

---

### Lab2
Module for Lab2:
```verilog
module opt_check2 (input a , input b , output y);
	assign y = a?1:b;
endmodule

```
Explanation:
 - The output ```y``` is equal to input ```1``` when input ```a``` is high, else it is equal to input ```b```.
 - Here the logic can simplified as ```y=a + a'.b = a + b```  which is just OR operation between a and b.


<img src="images/opt_check2.png" alt="Alt Text" width="500"/>

### Lab3
Module for Lab3:
```verilog
module opt_check3 (input a , input b, input c , output y);
	assign y = a?(c?b:0):0;
endmodule

```
Explanation:
 - Here is a two mux case. Output from inner mux can be written as ```c.b + c'.0 = c.b```. This is given to the outer mux which give the equation ``` y = c.b.a + 0.a' = a.b.c```
 - The output can be simplified as a 3 input AND gate.

<img src="images/opt_check3.png" alt="Alt Text" width="500"/>


### Lab4
Module for Lab4:
```verilog
module opt_check4 (input a , input b , input c , output y);
 assign y = a?(b?(a & c ):c):(!c);
 endmodule
```
Explanation:
 - ```b?(a & c ):c = b.a.c + b'.c = a.c + b'.c  ```
 - combining the above with the remaining we get ```y = a.c + a'.c'``` which is basically a XNOR gate.

<img src="images/opt_check4.png" alt="Alt Text" width="500"/>

### Lab5 : when the design contains submodules
Module for Lab5:
```verilog
module sub_module1(input a , input b , output y);
 assign y = a & b;
endmodule

module sub_module2(input a , input b , output y);
 assign y = a^b;
endmodule

module multiple_module_opt(input a , input b , input c , input d , output y);
wire n1,n2,n3;

sub_module1 U1 (.a(a) , .b(1'b1) , .y(n1));
sub_module2 U2 (.a(n1), .b(1'b0) , .y(n2));
sub_module2 U3 (.a(b), .b(d) , .y(n3));

assign y = c | (b & n1); 

endmodule
```
 - When there submodules inside the top module, after ```synth -top multiple_module_opt```, we have to ```flatten``` the netlist.
 - Then do ```opt_clean -purge```.
   
<div align="center">
  <img src="images/mult_mod.png" width="500px" />
  <img src="images/mult_mod_opt.png" width="500px" />
</div>


---
<div align="center">
  <img src="images/and_v0.png" width="500px" />
  <img src="images/and_v1.png" width="500px" />
</div>



---

## 3. Hierarchical vs. Flattened Synthesis

When there are multiple modules involved in design there are two options of synthesizing the design
 - Hierarchical
 - Flattened

Here we us multiple_modules.v to demonstrate this

1. Heirarchical synthesis

The default synthesis generates heirarchical netlist

```
$ synth -top multiple_modules
```
Netlist generated during Heirarchical synthesis:

![Alt Text](images/show_heir_synth.png)

```verilog
module multiple_modules(a, b, c, y);
  input a;
  input b;
  input c;
  wire net1;
  output y;
  sub_module1 u1 (
    .a(a),
    .b(b),
    .y(net1)
  );
  sub_module2 u2 (
    .a(net1),
    .b(c),
    .y(y)
  );
endmodule

module sub_module1(a, b, y);
  wire _0_;
  wire _1_;
  wire _2_;
  input a;
  input b;
  output y;
  sky130_fd_sc_hd__and2_2 _3_ (
    .A(_0_),
    .B(_1_),
    .X(_2_)
  );
  assign _0_ = b;
  assign _1_ = a;
  assign y = _2_;
endmodule

module sub_module2(a, b, y);
  wire _0_;
  wire _1_;
  wire _2_;
  wire _3_;
  wire _4_;
  input a;
  input b;
  output y;
  sky130_fd_sc_hd__clkinv_1 _5_ (
    .A(_0_),
    .Y(_3_)
  );
  sky130_fd_sc_hd__clkinv_1 _6_ (
    .A(_1_),
    .Y(_4_)
  );
  sky130_fd_sc_hd__nand2_1 _7_ (
    .A(_3_),
    .B(_4_),
    .Y(_2_)
  );
  assign _0_ = b;
  assign _1_ = a;
  assign y = _2_;
endmodule

```

2. Flattened synthesis

Inorder to convert the previously generated netlist to flattened mode:
```
$ flatten
```

Netlist corresponding to the flattened synthesis:

![Alt Text](images/show_flat_synth.png)

```verilog
/* Generated by Yosys 0.7 (git sha1 61f6811, gcc 6.2.0-11ubuntu1 -O2 -fdebug-prefix-map=/build/yosys-OIL3SR/yosys-0.7=. -fstack-protector-strong -fPIC -Os) */

module multiple_modules(a, b, c, y);
  wire _00_;
  wire _01_;
  wire _02_;
  wire _03_;
  wire _04_;
  wire _05_;
  wire _06_;
  wire _07_;
  input a;
  input b;
  input c;
  wire net1;
  wire \u1.a ;
  wire \u1.b ;
  wire \u1.y ;
  wire \u2.a ;
  wire \u2.b ;
  wire \u2.y ;
  output y;
  sky130_fd_sc_hd__and2_2 _08_ (
    .A(_00_),
    .B(_01_),
    .X(_02_)
  );
  sky130_fd_sc_hd__clkinv_1 _09_ (
    .A(_03_),
    .Y(_06_)
  );
  sky130_fd_sc_hd__clkinv_1 _10_ (
    .A(_04_),
    .Y(_07_)
  );
  sky130_fd_sc_hd__nand2_1 _11_ (
    .A(_06_),
    .B(_07_),
    .Y(_05_)
  );
  assign \u1.a  = a;
  assign \u1.b  = b;
  assign net1 = \u1.y ;
  assign _00_ = \u1.b ;
  assign _01_ = \u1.a ;
  assign \u1.y  = _02_;
  assign \u2.a  = net1;
  assign \u2.b  = c;
  assign y = \u2.y ;
  assign _03_ = \u2.b ;
  assign _04_ = \u2.a ;
  assign \u2.y  = _05_;
endmodule

```

In this netlist there are no instantiation of submodule. The top module is build directly from the standard cells.


For synthesising submodules:
```
synth -top submodule1
```
Here "submodule1" was one of the submodule in the main module "multiple_modules".
Need of synthesizing submodule:
 - If there are multiple instances of the same module. This can save time by avoiding synthesis of same module multiple times.
 - When there are large designs, it will be optimal to synthesis certain number of modules at a time and finish the synthesis of complete module(Divide and Conquer).

---

## 4. Using Flip-Flops in design

As the combiantional elements in the design introduce delay which are different for different elements, the output can have glitches. In order to prevent glitches from happening we need elements that can store values. This is why flip-flops are used in the design.

### 1. Flip-Flop with asynchronous reset

```verilog
module dff_asyncres ( input clk ,  input async_reset , input d , output reg q );
always @ (posedge clk , posedge async_reset)
begin
	if(async_reset)
		q <= 1'b0;
	else	
		q <= d;
end
endmodule
```
 - Whenever the ```async_reset``` goes high the output is made 0, without waiting for the positive edge of the clock. Reset is not synchronised with the clock.
 - When no reseting the output is equal to the value of input at each positive edge of the clock.

### 2. Flip-Flop with synchronous reset

```verilog
module dff_syncres ( input clk , input async_reset , input sync_reset , input d , output reg q );
always @ (posedge clk )
begin
	if (sync_reset)
		q <= 1'b0;
	else	
		q <= d;
end
endmodule
```
 - Resetting happens only at the first posedge of clock after ```sync_reset``` is made high.

---

## 5. Simulating and synthesis of flip-flops

### Simulation 

1. Compile design and testbench:
```
$ iverilog dff_asyncres.v tb_dff_asyncres.v
```
2. Run:
```
$ ./a.out
```
3. Observe the waveform:
```
$ gtkwave tb_dff_asyncres.vcd 
```
<div align="center">
  <img src="images/dff_async.png" width="800px" />
</div>

### Synthesis using Yosys

1. Start Yosys (from directory containing the design):
   ```
   $ yosys
   ```
2. Reading Liberty library:
   ```
   $ read_liberty -lib /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
   ```
3. Read verilog code of the Flip-Flop module:
   ```
   $ read_verilog dff_asyncres.v
   ```
4. Synthesize:
   ```
   $ synth -top dff_asyncres
   ```
   
5. Map the designs to the flip-flops in the library file:
   ```
   $ dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
   ```
6. Technology mapping:
   ```
   $ abc -liberty /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
   ```
7. Observe the gate-level netlist:
   ```
   $ show
   ```
   
<div align="center">
  <img src="images/dff_async_net.png" width="800px" />
</div>

   In the netlist there is inverter. This is because the standard cell of Flip-Flop have active low reset, but our design implements a active high reset.

---
## 6. Implementing multiplication with 2 as shifting

<div align="center">
  <img src="images/mult2.png" width="800px" />
</div>

Here there is no any cell associated with the design, the multiplication by 2 operation is realized as left shift by 1.

---

