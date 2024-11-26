## Implicit nets 隐式网络

Implicit nets are often a source of hard-to-detect bugs. In Verilog, net-type signals can be implicitly created by an `assign` statement or by attaching something undeclared to a module port. Implicit nets are always one-bit wires and causes bugs if you had intended to use a vector. Disabling creation of implicit nets can be done using the ``default_nettype none` directive.

```verilog
wire [2:0] a, c;   // Two vectors 
assign a = 3'b101;  // a = 101 
assign b = a;       // b =   1  implicitly-created wire 
assign c = b;       // c = 001  <-- bug 
my_module i1 (d,e); // d and e are implicitly one-bit wide if not declared.                    // This could be a bug if the port was intended to be a vector. 
```

Adding ``default_nettype none` would make the second line of code an error, which makes the bug more visible.

## Unpacked vs. Packed Arrays 未打包与打包数组

You may have noticed that in *declarations*, the vector indices are written *before* the vector name. This declares the "packed" dimensions of the array, where the bits are "packed" together into a blob (this is relevant in a simulator, but not in hardware). The *unpacked* dimensions are declared *after* the name. They are generally used to declare memory arrays. Since ECE253 didn't cover memory arrays, we have not used packed arrays in this course. See http://www.asic-world.com/systemverilog/data_types10.html for more details.

```verilog
reg [7:0] mem [255:0];   // 256 unpacked elements, each of which is a 8-bit packed vector of reg.
reg mem2 [28:0];         // 29 unpacked elements, each of which is a 1-bit reg.
```

## Bitwise vs. Logical Operators 按位运算与逻辑运算

Earlier, we mentioned that there are bitwise and logical versions of the various boolean operators (e.g., [norgate](https://hdlbits.01xz.net/wiki/norgate)). When using vectors, the distinction between the two operator types becomes important. A bitwise operation between two N-bit vectors replicates the operation for each bit of the vector and produces a N-bit output, while a logical operation treats the entire vector as a boolean value (true = non-zero, false = zero) and produces a 1-bit output.

Look at the simulation waveforms at how the bitwise-OR and logical-OR differ.

[![img](https://hdlbits.01xz.net/mw/images/1/1b/Vectorgates.png)](https://hdlbits.01xz.net/wiki/File:Vectorgates.png)

1. ##### 按位运算（Bit - wise Operations）

   - 按位与（`&`）
     - **定义**：对两个操作数的每一位进行与运算。只有当两个相应位都为 1 时，结果位才为 1，否则为 0。
     - 示例
       - 假设`a = 4'b1010`，`b = 4'b1100`。进行按位与运算`c = a & b`时，首先将`a`和`b`按位对齐：
       - `a`：`1010`
       - `b`：`1100`
       - 然后逐位进行与运算：第 1 位（从右往左），`0 & 0 = 0`；第 2 位，`1 & 0 = 0`；第 3 位，`0 & 1 = 0`；第 4 位，`1 & 1 = 1`。所以`c = 4'b1000`。
     - **应用场景**：常用于屏蔽某些位。例如，若要清除一个字节数据的低 4 位，可以将该字节与`8'b11110000`进行按位与运算。
   - 按位或（`|`）
     - **定义**：对两个操作数的每一位进行或运算。只要两个相应位中有一个为 1，结果位就为 1；只有当两个位都为 0 时，结果位才为 0。
     - 示例
       - 对于`a = 4'b1010`，`b = 4'b1100`，进行按位或运算`c = a | b`。
       - 按位对齐后：
       - `a`：`1010`
       - `b`：`1100`
       - 逐位进行或运算：第 1 位，`0 | 0 = 0`；第 2 位，`1 | 0 = 1`；第 3 位，`0 | 1 = 1`；第 4 位，`1 | 1 = 1`。所以`c = 4'b1110`。
     - **应用场景**：在设置某些位时很有用。例如，若要将一个字节数据的低 4 位全部设置为 1，可以将该字节与`8'b00001111`进行按位或运算。
   - 按位异或（`^`）
     - **定义**：对两个操作数的每一位进行异或运算。当两个相应位不同时，结果位为 1；当两个位相同时，结果位为 0。
     - 示例
       - 对于`a = 4'b1010`，`b = 4'b1100`，进行按位异或运算`c = a ^ b`。
       - 按位对齐：
       - `a`：`1010`
       - `b`：`1100`
       - 逐位异或运算：第 1 位，`0 ^ 0 = 0`；第 2 位，`1 ^ 0 = 1`；第 3 位，`0 ^ 1 = 1`；第 4 位，`1 ^ 1 = 0`。所以`c = 4'b0110`。
     - **应用场景**：常用于数据加密、校验等领域。例如，简单的奇偶校验可以通过对数据位进行按位异或运算来实现。
   - 按位取反（`~`）
     - **定义**：对一个操作数的每一位进行取反运算。原来是 0 的位变为 1，原来是 1 的位变为 0。
     - 示例
       - 若`a = 4'b1010`，进行按位取反运算`b = ~a`。则`b = 4'b0101`。
     - **应用场景**：在需要对数据的每一位进行反转的情况下使用，如某些编码转换场景。

2. ##### 逻辑运算（Logical Operations）

   - 逻辑与（`&&`）
     - **定义**：对两个操作数进行逻辑与运算。操作数被视为布尔值（0 为假，非 0 为真），只有当两个操作数都为真时，结果才为真，否则为假。
     - 示例
       - 假设`a = 4'b0000`（视为假），`b = 4'b1010`（视为真），进行逻辑与运算`c = a && b`，结果`c`为假（0）。
     - **应用场景**：在条件判断语句中用于判断多个条件是否同时满足。例如，在`if`语句中判断`if (enable && valid)`，只有当`enable`和`valid`都为真时，`if`语句中的代码块才会执行。
   - 逻辑或（`||`）
     - **定义**：对两个操作数进行逻辑或运算。只要两个操作数中有一个为真，结果就为真；只有当两个操作数都为假时，结果才为假。
     - 示例
       - 对于`a = 4'b0000`（假），`b = 4'b1010`（真），进行逻辑或运算`c = a || b`，结果`c`为真（1）。
     - **应用场景**：用于判断多个条件中是否至少有一个满足。例如，在`if`语句中`if (error || warning)`，只要`error`或者`warning`中有一个为真，`if`语句中的代码块就会执行。
   - 逻辑非（`!`）
     - **定义**：对一个操作数进行逻辑非运算。如果操作数为假（0），结果为真（1）；如果操作数为真（非 0），结果为假（0）。
     - 示例
       - 若`a = 4'b0000`，进行逻辑非运算`b =!a`，结果`b`为真（1）。若`a = 4'b1010`，`b =!a`，结果`b`为假（0）。
     - **应用场景**：用于对条件进行取反。例如，在`if`语句中`if (!ready)`，当`ready`为假时，`if`语句中的代码块会执行。

## Concatenation operator 连接符

Concatenation needs to know the width of every component (or how would you know the length of the result?). Thus, `{1, 2, 3}` is illegal and results in the error message: `unsized constants are not allowed in concatenations`.

```verilog
input [15:0] in;
output [23:0] out;
assign {out[7:0], out[15:8]} = in;         // Swap two bytes. Right side and left side are both 16-bit vectors.
assign out[15:0] = {in[7:0], in[15:8]};    // This is the same thing.
assign out = {in[7:0], in[15:8]};       // This is different. The 16-bit vector on the right is extended to match the 24-bit vector on the left, so out[23:16] are zero. In the first two examples, out[23:16] are not assigned.
```

## Connecting Signals to Module Ports 模块端口连接

There are two commonly-used methods to connect a wire to a port: by position or by name.

### By position

The syntax to connect wires to ports by position should be familiar, as it uses a C-like syntax. When instantiating a module, ports are connected left to right according to the module's declaration. For example:

```
mod_a instance1 ( wa, wb, wc );
```

This instantiates a module of type `mod_a` and gives it an *instance name* of "instance1", then connects signal `wa` (outside the new module) to the **first** port (`in1`) of the new module, `wb` to the **second** port (`in2`), and `wc` to the **third** port (`out`). One drawback of this syntax is that if the module's port list changes, all instantiations of the module will also need to be found and changed to match the new module.

### By name

Connecting signals to a module's ports *by name* allows wires to remain correctly connected even if the port list changes. This syntax is more verbose, however.

```
mod_a instance2 ( .out(wc), .in1(wa), .in2(wb) );
```

The above line instantiates a module of type `mod_a` named "instance2", then connects signal `wa` (outside the module) to the port **named** `in1`, `wb` to the port **named** `in2`, and `wc` to the port **named** `out`. Notice how the ordering of ports is irrelevant here because the connection will be made to the correct name, regardless of its position in the sub-module's port list. Also notice the period immediately preceding the port name in this syntax.

## Always 

Combinational always blocks are equivalent to assign statements, thus there is always a way to express a combinational circuit both ways. The choice between which to use is mainly an issue of which syntax is more convenient. **The syntax for code inside a procedural block is different from code that is outside.** Procedural blocks have a richer set of statements (e.g., if-then, case), cannot contain continuous assignments, but also introduces many new non-intuitive ways of making errors. *(**Procedural continuous assignments* do exist, but are somewhat different from *continuous assignments*, and are not synthesizable.)

For example, the assign and combinational always block describe the same circuit. Both create the same blob of combinational logic. Both will recompute the output whenever any of the inputs (right side) changes value. ` **assign** out1 = a & b | c ^ d; **always @(\*)** out2 = a & b | c ^ d;`

For combinational always blocks, always use a sensitivity list of `(*)`. Explicitly listing out the signals is error-prone (if you miss one), and is ignored for hardware synthesis. If you explicitly specify the sensitivity list and miss a signal, the synthesized hardware will still behave as though `(*)` was specified, but the simulation will not and not match the hardware's behaviour. (In SystemVerilog, use `always_comb`.)

A note on wire vs. reg: The left-hand-side of an assign statement must be a *net* type (e.g., `wire`), while the left-hand-side of a procedural assignment (in an always block) must be a *variable* type (e.g., `reg`). These types (wire vs. reg) have nothing to do with what hardware is synthesized, and is just syntax left over from Verilog's use as a hardware *simulation* language.

## Always case

Case statements in Verilog are nearly equivalent to a sequence of if-elseif-else that compares one expression to a list of others. Its syntax and functionality differs from the `switch` statement in C.



```
always @(*) begin     // This is a combinational circuit
    case (in)
      1'b1: begin 
               out = 1'b1;  // begin-end if >1 statement
            end
      1'b0: out = 1'b0;
      default: out = 1'bx;
    endcase
end
```



- The case statement begins with `case` and each "case item" ends with a colon. There is no "switch".
- Each case item can execute *exactly one* statement. This makes the "break" used in C unnecessary. But this means that if you need more than one statement, you must use `begin ... end`.
- Duplicate (and partially overlapping) case items are permitted. The first one that matches is used. C does not allow duplicate case items.