[Rocket](../Readme.md)/[rocket](../rocket.md)/[ALU](https://github.com/freechipsproject/rocket-chip/blob/master/src/main/scala/rocket/ALU.scala)
========================
*Rocket ALU implementation.*

*****************

class ALU
-------------------
*Rocket ALU*

+ I/O, type and Parameter

| name                   | type             | direction  | description                           |
| :---                   | :--:             | :--:       | :---                                  |
| p                      | Parameters       | param      | configuration                         |
| dw                     | Bits             | I          | datawidth: `DW_64` `DW_32`            |
| fn                     | Bits             | I          | ALU function                          |
| in1                    | UInt             | I          | ALU input 1                           |
| in2                    | UInt             | I          | ALU input 2                           |
| out                    | UInt             | O          | ALU output                            |
| adder\_out             | UInt             | O          | address for D$                        |
| cmp\_out               | Bool             | O          | compare result                        |

`DW_64`, `DW_32` corresponds to datawidth, used for 64I and 32I set respectively in most cases. Note that `DW_32` is used in 64I when instruction has notation of XX.W (See `IDecode.scala` and the RISC-V spec for more details)

`object ALU` contains methods for ALU functions. ALU functions corresponding to the instruction is determined in decode stage. (See signal `alu_fn` from `IDecode.scala` for details on decoding). The methods are annotated in FN_XXX form. There are also helper methods such as `isSub`, `isCmp` that determines the ALU function type.

### ADD, SUB
The `adder_out` output of the ALU is used for D$ address. It returns the two's complement result of the addition/subtraction.
```
io.adder_out := io.in1 + in2_inv + isSub(io.fn)
io.dmem.req.bits.addr := encodeVirtualAddress(ex_rs(0), alu.io.adder_out) // from RocketCore.scala

```


### Branch comparison (`cmp_out`)
There are 4 types of branch instructions each utilizing EQ(Equal), NE(Not Equal), GE(Greater than or Equal), LT (Less Than). 
Of those types, GEU and NEU are for unsigned value comparison.

For instructions including EQ or NE, `cmpEq` is true, thus checks if in1 equals in2 (`in1_xor_in2 == UInt(0)`) and inverts the result if `cmpInverted` (which is true for NE) and outputs the value to `cmp_out`
For other instructions (including BE and GE), value `slt` is set to `cmp_out` (inverted in GE type instructions)

`val slt` is calculated in the below steps.
  1. Since the function is LT type, thus `isSub` is true and `io.adder_out` value contains the subtraction result.
  2. If the sign-bits of `in1` and `in2` are the same, the MSB of the substraction result (0 if `in1` is larger, 1 o.w.) is returned.
  3. When the sign-bit differs and with signed comparison, if `in1` is positive and `in2` is negative `in1` is always larger thus 0 is returned and vice versa.
  4. When the sign-bit differs and with unsigned comparison, if `in1` has MSB 1 and `in2` has MSB 0, `in1` is always larger thus 0 is returned and vice versa.


### Bit-shifter
For SLL(Shift Left Logical), SRL(Shift Right Logical), SRA (Shift Right Arithematic), shifter is used.
 - `val shamt` (shift amount) is determined to be `in2`'s least 5(32I and .W instructions in 64I) or 6 bits(in case of 64I). 
 - `val shin_r` (shift right input) contains the input to be shifted. It usually contains `in1` value,  but fills its upper 32bits with 0 in case of .W instructions in 64I

 - `val shin`: contains either `shin_r` or the reversed `shin_r` 
 - `val shout_r`: (shift right output) essentially does the shifting operation on `shin` with given `shamt`. Then with its reversed version gives the final `shout`. During the process, `shin` is sign-extended by a single bit to easily handle arithmatic shift.


### AND, OR, XOR and final output
Simple AND, OR, XOR is implmented into `logic`. `shift_logic` contains `slt`, `logic`, `shout` results bitwise-ORed together. (Each bitwise-ORed values contain zeros when the corresponding ALU function is not selected)
Combined with the `io.adder_out` selected in ADD and SUB instruction and signed extended in DW_32 case, it generates the final `io.out` output.



<br><br><br><p align="right">
<sub>
Last updated: 08/07/2017<br>
[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/), &copy; (2017) [Wei Song](mailto:wsong83@gmail.com)<br>
[Apache 2.0](https://github.com/freechipsproject/rocket-chip/blob/master/LICENSE.SiFive), &copy; (2016-2017) SiFive, Inc<br>
[BSD](https://github.com/freechipsproject/rocket-chip/blob/master/LICENSE.Berkeley), &copy; (2012-2014, 2016) The Regents of the University of California (Regents)
</sub>
</p>
