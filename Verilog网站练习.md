[TOC]





# Getting Started

### Step one

Build a circuit with no inputs and one output. That output should always drive 1 (or logic high).

```verilog
module top_module( output one );

	
    assign one = 1'b1;

endmodule
```



### Zero

Build a circuit with no inputs and one output that outputs a constant `0`

```verilog
module top_module(
    output zero
);// Module body starts after semicolon
	assign zero=1'b0;
            
endmodule
```



# Verilog Language

## Basics

### Wire

Create a module with one input and one output that behaves like a wire.

```verilog
module top_module( input in, output out );

    assign out=in;
    
endmodule
```



### Wires

Create a module with 3 inputs and 4 outputs that behaves like wires that makes these connections:

```
a -> w
b -> x
b -> y
c -> z
```

```verilog
module top_module( 
    input a,b,c,
    output w,x,y,z );

    assign w=a;
    assign x=b;
    assign y=b;
    assign z=c;
    // If we're certain about the width of each signal, using 
	// the concatenation operator is equivalent and shorter:
	// assign {w,x,y,z} = {a,b,b,c};
    
endmodule
```



### Inverter

Create a module that implements a NOT gate.

```verilog
module top_module( input in, output out );

    assign out=~in;
    
endmodule
```



### AND gate

Create a module that implements an AND gate.

```verilog
module top_module( 
    input a, 
    input b, 
    output out );

    assign out=a&b;
    
endmodule
```



### NOR gate

Create a module that implements a NOR gate. 

```verilog
module top_module( 
    input a, 
    input b, 
    output out );

    assign out=~(a|b);
    
endmodule
```



### XNOR gate

Create a module that implements an XNOR gate.

```verilog
module top_module( 
    input a, 
    input b, 
    output out );

    assign out=~(a^b);
    
endmodule
```



### Declaring wires

Implement the following circuit. Create two intermediate wires (named anything you want) to connect the AND and OR gates together. Note that the wire that feeds the NOT gate is really wire `out`, so you do not necessarily need to declare a third wire here. Notice how wires are driven by exactly one source (output of a gate), but can feed multiple inputs.

If you're following the circuit structure in the diagram, you should end up with four assign statements, as there are four signals that need a value assigned.

(Yes, it is possible to create a circuit with the same functionality without the intermediate wires.)

![img](https://hdlbits.01xz.net/mw/images/3/3a/Wiredecl2.png)

```verilog
`default_nettype none
module top_module(
    input a,
    input b,
    input c,
    input d,
    output out,
    output out_n   ); 

    wire and_1;//直接wire w1,w2;更简便
    wire and_2;
    wire or_1;//这个线可以不用
    
    assign and_1=a&b;
    assign and_2=c&d;
    assign or_1=and_1|and_2;
    assign out=or_1;//直接out=w1|w2;更简便
    assign out_n=~or_1;//直接out_n=~out;更简便
    
endmodule
```



### 7458 chip

The 7458 is a chip with four AND gates and two OR gates. This problem is slightly more complex than [7420](https://hdlbits.01xz.net/wiki/7420).

Create a module with the same functionality as the 7458 chip. It has 10 inputs and 2 outputs. You may choose to use an `assign` statement to drive each of the output wires, or you may choose to declare (four) wires for use as intermediate signals, where each internal wire is driven by the output of one of the AND gates. For extra practice, try it both ways.

![img](https://hdlbits.01xz.net/mw/images/e/e1/7458.png)

```verilog
module top_module ( 
    input p1a, p1b, p1c, p1d, p1e, p1f,
    output p1y,
    input p2a, p2b, p2c, p2d,
    output p2y );

    wire w1,w2,w3,w4;
    
    assign w1=p1a&p1b&p1c;
    assign w2=p2a&p2b;
    assign w3=p1f&p1e&p1d;
    assign w4=p2c&p2d;
    assign p1y=w1|w3;
    assign p2y=w2|w4;

endmodule//用wire的
```

```verilog
module top_module ( 
    input p1a, p1b, p1c, p1d, p1e, p1f,
    output p1y,
    input p2a, p2b, p2c, p2d,
    output p2y );

    assign p1y=(p1a&p1b&p1c)|(p1f&p1e&p1d);
    assign p2y=(p2a&p2b)|(p2c&p2d);

endmodule//不用wire的
```



## Vectors

### Vectors

Build a circuit that has one 3-bit input, then outputs the same vector, and also splits it into three separate 1-bit outputs. Connect output `o0` to the input vector's position 0, `o1` to position 1, etc.

![img](https://hdlbits.01xz.net/mw/images/a/ae/Vector0.png)

```verilog
module top_module ( 
    input wire [2:0] vec,
    output wire [2:0] outv,
    output wire o2,
    output wire o1,
    output wire o0  ); // Module body starts after module declaration

    assign outv=vec;
    assign o0=vec[0];	// This is ok too: assign {o2, o1, o0} = vec;注意顺序
    assign o1=vec[1];
    assign o2=vec[2];
    
endmodule
```



### Vectors in more detail

Build a combinational circuit that splits an input half-word (16 bits, [15:0] ) into lower [7:0] and upper [15:8] bytes.

```verilog
`default_nettype none     // Disable implicit nets. Reduces some types of bugs.
module top_module( 
    input wire [15:0] in,
    output wire [7:0] out_hi,
    output wire [7:0] out_lo );

    assign out_hi=in[15:8];
    assign out_lo=in[7:0];
    
endmodule
```



### Vector part select

A 32-bit vector can be viewed as containing 4 bytes (bits [31:24], [23:16], etc.). Build a circuit that will reverse the *byte* ordering of the 4-byte word.

```
AaaaaaaaBbbbbbbbCcccccccDddddddd => DdddddddCcccccccBbbbbbbbAaaaaaaa
```

This operation is often used when the [endianness](https://en.wikipedia.org/wiki/Endianness) of a piece of data needs to be swapped, for example between little-endian x86 systems and the big-endian formats used in many Internet protocols.

```verilog
module top_module( 
    input [31:0] in,
    output [31:0] out );//

    assign out[31:24]=in[7:0];
    assign out[23:16]=in[15:8];
    assign out[15:8]=in[23:16];
    assign out[7:0]=in[31:24];
    

endmodule
```



### Bitwise operators

Build a circuit that has two 3-bit inputs that computes the bitwise-OR of the two vectors, the logical-OR of the two vectors, and the inverse (NOT) of both vectors. Place the inverse of `b` in the upper half of `out_not` (i.e., bits [5:3]), and the inverse of `a` in the lower half.

![img](https://hdlbits.01xz.net/mw/images/1/1b/Vectorgates.png)

```verilog
module top_module( 
    input [2:0] a,
    input [2:0] b,
    output [2:0] out_or_bitwise,
    output out_or_logical,
    output [5:0] out_not
);

    assign out_or_bitwise=a|b;
    assign out_or_logical=a||b;
    assign out_not[5:3]=~b;
    assign out_not[2:0]=~a;
    
endmodule
```



### Gates100

Build a combinational circuit with 100 inputs, `in[99:0]`.

There are 3 outputs:

- out_and: output of a 100-input AND gate.
- out_or: output of a 100-input OR gate.
- out_xor: output of a 100-input XOR gate.

```verilog
module top_module( 
    input [99:0] in,
    output out_and,
    output out_or,
    output out_xor 
);

    assign out_and=&in;//The reduction operators
    assign out_or=|in;
    assign out_xor=^in;
    
endmodule
```



### Vector concatenation operator

Given several input vectors, concatenate them together then split them up into several output vectors. There are six 5-bit input vectors: a, b, c, d, e, and f, for a total of 30 bits of input. There are four 8-bit output vectors: w, x, y, and z, for 32 bits of output. The output should be a concatenation of the input vectors followed by two `1` bits:

![img](https://hdlbits.01xz.net/mw/images/0/0c/Vector3.png)

```verilog
module top_module (
    input [4:0] a, b, c, d, e, f,
    output [7:0] w, x, y, z );//

    assign {w, x, y, z} = {a, b, c, d, e, f, 2'b11};

endmodule
```



### Vector reversal

Given an 8-bit input vector [7:0], reverse its bit ordering.

```verilog
module top_module( 
    input [7:0] in,
    output [7:0] out
);
	assign out = {in[0], in[1], in[2], in[3], in[4], in[5], in[6], in[7]};
endmodule
```



Given a 100-bit input vector [99:0], reverse its bit ordering.

```verilog
module top_module( 
    input [99:0] in,
    output [99:0] out
);

    integer i;
    always @(*) begin
        for(i=0;i<100;i=i+1)begin
            out[i]=in[99-i];
        end
    end
endmodule
```



### Replication operator

Build a circuit that sign-extends an 8-bit number to 32 bits. This requires a concatenation of 24 copies of the sign bit (i.e., replicate bit[7] 24 times) followed by the 8-bit number itself.

```verilog
module top_module (
    input [7:0] in,
    output [31:0] out );//

    assign out = { {24{in[7]}} , in };

endmodule
```



### More replication

Given five 1-bit signals (a, b, c, d, and e), compute all 25 pairwise one-bit comparisons in the 25-bit output vector. The output should be 1 if the two bits being compared are equal.

![img](https://hdlbits.01xz.net/mw/images/a/ac/Vector5.png)

```verilog
module top_module (
	input a, b, c, d, e,
	output [24:0] out
);

	wire [24:0] top, bottom;
	assign top    = { {5{a}}, {5{b}}, {5{c}}, {5{d}}, {5{e}} };
	assign bottom = {5{a,b,c,d,e}};
	assign out = ~top ^ bottom;	// Bitwise XNOR

	// This could be done on one line:
	// assign out = ~{ {5{a}}, {5{b}}, {5{c}}, {5{d}}, {5{e}} } ^ {5{a,b,c,d,e}};
	
endmodule
```



## Modules: Hierarchy

### Modules

The figure below shows a very simple circuit with a sub-module. In this exercise, create one *instance* of module `**mod_a**`, then connect the module's three pins (`in1`, `in2`, and `out`) to your top-level module's three ports (wires `a`, `b`, and `out`). The module `mod_a` is provided for you — you must instantiate it.

![img](https://hdlbits.01xz.net/mw/images/c/c0/Module.png)

```verilog
module top_module ( input a, input b, output out );

    mod_a instance_name (a,b,out);
endmodule
```

```verilog
module top_module ( input a, input b, output out );

    mod_a instance_name (.in1(a),.in2(b),.out(out));
endmodule
```



### Connecting ports

This problem is similar to [module](https://hdlbits.01xz.net/wiki/module). You are given a module named that has 2 outputs and 4 inputs, in some order. You must connect the 6 ports *by name* to your top-level module's ports: `mod_a`

| Port in `**mod_a**` | Port in `**top_module**` |
| :------------------ | :----------------------- |
| `output out1`       | `out1`                   |
| `output out2`       | `out2`                   |
| `input in1`         | `a`                      |
| `input in2`         | `b`                      |
| `input in3`         | `c`                      |
| `input in4`         | `d`                      |

You are given the following module:

```verilog
module mod_a ( output out1, output out2, input in1, input in2, input in3, input in4);
```

by position：

```verilog
module top_module ( 
    input a, 
    input b, 
    input c,
    input d,
    output out1,
    output out2
);

    mod_a ex1(out1,out2,a,b,c,d);
endmodule
```

by name：

```verilog
module top_module ( 
    input a, 
    input b, 
    input c,
    input d,
    output out1,
    output out2
);

    mod_a ex2(.in1(a),.in2(b),.in3(c),.in4(d),.out1(out1),.out2(out2));
endmodule
```



### Three modules

You are given a module `my_dff` with two inputs and one output (that implements a D flip-flop). Instantiate three of them, then chain them together to make a shift register of length 3. The `clk` port needs to be connected to all instances.

The module provided to you is: `module my_dff ( input clk, input d, output q );`

![img](https://hdlbits.01xz.net/mw/images/6/60/Module_shift.png)

```verilog
module top_module ( input clk, input d, output q );

    wire q1,q2;
    my_dff md1(clk,d,q1);
    my_dff md2(clk,q1,q2);
    my_dff md3(clk,q2,q);
endmodule
```



### Modules and vectors

You are given a module `my_dff8` with two inputs and one output (that implements a set of 8 D flip-flops). Instantiate three of them, then chain them together to make a 8-bit wide shift register of length 3. In addition, create a 4-to-1 multiplexer (not provided) that chooses what to output depending on `sel[1:0]`: The value at the input d, after the first, after the second, or after the third D flip-flop. (Essentially, `sel` selects how many cycles to delay the input, from zero to three clock cycles.)

The module provided to you is: `module my_dff8 ( input clk, input [7:0] d, output [7:0] q );`

The multiplexer is not provided. One possible way to write one is inside an `always` block with a `case` statement inside. (See also: [mux9to1v](https://hdlbits.01xz.net/wiki/mux9to1v))

![img](https://hdlbits.01xz.net/mw/images/7/76/Module_shift8.png)

```verilog
module top_module (
	input clk,
	input [7:0] d,
	input [1:0] sel,
	output reg [7:0] q
);

	wire [7:0] o1, o2, o3;		// output of each my_dff8
	
	// Instantiate three my_dff8s
	my_dff8 d1 ( clk, d, o1 );
	my_dff8 d2 ( clk, o1, o2 );
	my_dff8 d3 ( clk, o2, o3 );

	// This is one way to make a 4-to-1 multiplexer
	always @(*)		// Combinational always block
		case(sel)
			2'h0: q = d;
			2'h1: q = o1;
			2'h2: q = o2;
			2'h3: q = o3;
		endcase

endmodule
```



### Adder1

You are given a module `add16` that performs a 16-bit addition. Instantiate two of them to create a 32-bit adder. One add16 module computes the lower 16 bits of the addition result, while the second add16 module computes the upper 16 bits of the result, after receiving the carry-out from the first adder. Your 32-bit adder does not need to handle carry-in (assume 0) or carry-out (ignored), but the internal modules need to in order to function correctly. (In other words, the `add16` module performs 16-bit a + b + cin, while your module performs 32-bit a + b).

Connect the modules together as shown in the diagram below. The provided module `add16` has the following declaration:

```verilog
module add16 ( input[15:0] **a**, input[15:0] **b**, input **cin**, output[15:0] **sum**, output **cout** );
```

![img](https://hdlbits.01xz.net/mw/images/a/a3/Module_add.png)

```verilog
module top_module(
    input [31:0] a,
    input [31:0] b,
    output [31:0] sum
);

    wire cc;
    add16 a1(a[15:0],b[15:0],0,sum[15:0],cc);
    add16 a2(a[31:16],b[31:16],cc,sum[31:16], );
endmodule
```



### Adder2

In this exercise, you will create a circuit with two levels of hierarchy. Your `top_module` will instantiate two copies of `add16` (provided), each of which will instantiate 16 copies of `add1` (which you must write). Thus, you must write *two* modules: `top_module` and `add1`.

In summary, there are three modules in this design:

- `top_module` — Your top-level module that contains two of...
- `add16`, provided — A 16-bit adder module that is composed of 16 of...
- `add1` — A 1-bit full adder module.

![img](https://hdlbits.01xz.net/mw/images/f/f3/Module_fadd.png)

```verilog
module top_module (
    input [31:0] a,
    input [31:0] b,
    output [31:0] sum
);//
    wire cc;
    add16 ad1(a[15:0] ,b[15:0] , 0 ,sum[15:0] , cc);
    add16 ad2(a[31:16] ,b[31:16] , cc ,sum[31:16] , );

endmodule

module add1 ( input a, input b, input cin,   output sum, output cout );

	assign sum = a ^ b ^ cin;
    assign cout = (a & b) | (a & cin) | (b & cin);

endmodule
```



### Carry-select adder

In this exercise, you are provided with the same module `add16` as the previous exercise, which adds two 16-bit numbers with carry-in and produces a carry-out and 16-bit sum. You must instantiate *three* of these to build the carry-select adder, using your own 16-bit 2-to-1 multiplexer.

Connect the modules together as shown in the diagram below. The provided module `add16` has the following declaration:

```verilog
module add16 ( input[15:0] **a**, input[15:0] **b**, input **cin**, output[15:0] **sum**, output **cout** );
```

![img](https://hdlbits.01xz.net/mw/images/3/3e/Module_cseladd.png)

```verilog
module top_module(
    input [31:0] a,
    input [31:0] b,
    output [31:0] sum
);

    wire c;
    wire [15:0] c0;
    wire [15:0] c1;
    add16 a0(a[15:0],b[15:0],0,sum[15:0],c);
    add16 a1(a[31:16],b[31:16],0,c0, );
    add16 a2(a[31:16],b[31:16],1,c1, );
    
    always @(*)
        case(c)
            1'b0:sum[31:16]=c0;
            1'b1:sum[31:16]=c1;
        endcase
    
endmodule

```



### Adder-subtractor

An adder-subtractor can be built from an adder by optionally negating one of the inputs, which is equivalent to inverting the input then adding 1. The net result is a circuit that can do two operations: (a + b + 0) and (a + ~b + 1). See [Wikipedia](https://en.wikipedia.org/wiki/Adder–subtractor) if you want a more detailed explanation of how this circuit works.

Build the adder-subtractor below.

You are provided with a 16-bit adder module, which you need to instantiate twice:

```
module add16 ( input[15:0] **a**, input[15:0] **b**, input **cin**, output[15:0] **sum**, output **cout** );
```

Use a 32-bit wide XOR gate to invert the `b` input whenever `sub` is 1. (This can also be viewed as `b[31:0]` XORed with sub replicated 32 times. See [replication operator](https://hdlbits.01xz.net/wiki/vector4).). Also connect the `sub` input to the carry-in of the adder.

![img](https://hdlbits.01xz.net/mw/images/a/ae/Module_addsub.png)

```verilog
module top_module(
    input [31:0] a,
    input [31:0] b,
    input sub,
    output [31:0] sum
);

    wire [31:0] subb,bb;
    wire cc;
    assign subb={32{sub}};
    assign bb=b^subb;
    
    add16 a1(a[15:0],bb[15:0],sub,sum[15:0],cc);//注意变为bb
    add16 a2(a[31:16],bb[31:16],cc,sum[31:16],);
endmodule
```



## Procedures

### Always blocks (combinational)

Build an AND gate using both an assign statement and a combinational always block. (Since assign statements and combinational always blocks function identically, there is no way to enforce that you're using both methods. But you're here for practice, right?...)

![img](https://hdlbits.01xz.net/mw/images/2/2b/Alwayscomb.png)

```verilog
module top_module(
    input a, 
    input b,
    output wire out_assign,
    output reg out_alwaysblock
);

    assign out_assign=a&b;
    always @(*) out_alwaysblock=a&b;
    
endmodule
```



### Always blocks (clocked)

Build an XOR gate three ways, using an assign statement, a combinational always block, and a clocked always block. Note that the clocked always block produces a different circuit from the other two: There is a flip-flop so the output is delayed.

![img](https://hdlbits.01xz.net/mw/images/4/40/Alwaysff.png)

```
module top_module(
    input clk,
    input a,
    input b,
    output wire out_assign,
    output reg out_always_comb,
    output reg out_always_ff   );

    assign out_assign=a^b;
    always@(*) out_always_comb=a^b;
    always@(posedge clk) out_always_ff=a^b;
endmodule
```



### If statement

Build a 2-to-1 mux that chooses between `a` and `b`. Choose `b` if *both* `sel_b1` and `sel_b2` are true. Otherwise, choose `a`. Do the same twice, once using `assign` statements and once using a procedural if statement.

| sel_b1 | sel_b2 | out_assign out_always |
| :----- | :----- | :-------------------- |
| 0      | 0      | a                     |
| 0      | 1      | a                     |
| 1      | 0      | a                     |
| 1      | 1      | b                     |

```verilog
module top_module(
    input a,
    input b,
    input sel_b1,
    input sel_b2,
    output wire out_assign,
    output reg out_always   ); 

    assign out_assign=(sel_b1*sel_b2==1) ? b:a;
    always @(*) begin
        if (sel_b1*sel_b2==1) begin
            out_always=b;
        end
        else begin
            out_always=a;
        end
    end
endmodule
```



### If statement latches

The following code contains incorrect behaviour that creates a latch. Fix the bugs so that you will shut off the computer only if it's really overheated, and stop driving if you've arrived at your destination or you need to refuel.

```verilog
module top_module (
    input      cpu_overheated,
    output reg shut_off_computer,
    input      arrived,
    input      gas_tank_empty,
    output reg keep_driving  ); //

    always @(*) begin
        if (cpu_overheated)
           shut_off_computer = 1;
        else
            shut_off_computer=0;
    end

    always @(*) begin
        if (~arrived)
           keep_driving = ~gas_tank_empty;
        else
            keep_driving=0;
    end

endmodule
```



### Case statement

Case statements are more convenient than if statements if there are a large number of cases. So, in this exercise, create a 6-to-1 multiplexer. When `sel` is between 0 and 5, choose the corresponding data input. Otherwise, output 0. The data inputs and outputs are all 4 bits wide.

Be careful of inferring latches (See.[always_if2](https://hdlbits.01xz.net/wiki/always_if2))

```verilog
module top_module ( 
    input [2:0] sel, 
    input [3:0] data0,
    input [3:0] data1,
    input [3:0] data2,
    input [3:0] data3,
    input [3:0] data4,
    input [3:0] data5,
    output reg [3:0] out   );//

    always@(*) begin  // This is a combinational circuit
        case(sel)
            3'b000:out=data0;
            3'b001:out=data1;
            3'b010:out=data2;
            3'b011:out=data3;
            3'b100:out=data4;
            3'b101:out=data5;
            default:out=0;
        endcase
    end

endmodule
```



### Priority encoder

A *priority encoder* is a combinational circuit that, when given an input bit vector, outputs the position of the first `1` bit in the vector. For example, a 8-bit priority encoder given the input `8'b10010000` would output `3'd4`, because bit[4] is first bit that is high.

Build a 4-bit priority encoder. For this problem, if none of the input bits are high (i.e., input is zero), output zero. Note that a 4-bit number has 16 possible combinations.

```verilog
module top_module (
    input [3:0] in,
    output reg [1:0] pos  );

    always @(*) begin
        case(in)
            4'b0001:pos=2'd0;
            4'b0010:pos=2'd1;
            4'b0110:pos=2'd1;
            4'b1010:pos=2'd1;
            4'b1110:pos=2'd1;
            4'b0100:pos=2'd2;
            4'b1100:pos=2'd2;
            4'b1000:pos=2'd3;
            default:pos=2'd0;
        endcase
    end
    
endmodule
```



### Priority encoder with casez

Build a priority encoder for 8-bit inputs. Given an 8-bit vector, the output should report the first (least significant) bit in the vector that is `1`. Report zero if the input vector has no bits that are high. For example, the input `8'b10010000` should output `3'd4`, because bit[4] is first bit that is high.

```verilog
module top_module (
    input [7:0] in,
    output reg [2:0] pos );

    always@(*) begin
        casez(in)
            8'bzzzzzzz1:pos=3'd0;
            8'bzzzzzz10:pos=3'd1;
            8'bzzzzz100:pos=3'd2;
            8'bzzzz1000:pos=3'd3;
            8'bzzz10000:pos=3'd4;
            8'bzz100000:pos=3'd5;
            8'bz1000000:pos=3'd6;
            8'b10000000:pos=3'd7;
            default:pos=0;
        endcase
    end
    
endmodule
```

