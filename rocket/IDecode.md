## class IntCtrlSigs
- def decode(inst: UInt, table: Iterable[(BitPat, List[BitPat])]) takes
  instruction and decode_table to wire the CtrlSigs. The decode is implemented using DecodeLogic in Decode.scala

## class XXDecode 

                //           jal                                                             renf1               fence.i
                //   val     | jalr                                                          | renf2             |
                //   | fp_val| | renx2                                                       | | renf3           |
                //   | | rocc| | | renx1       s_alu1                          mem_val       | | | wfd           |
                //   | | | br| | | |   s_alu2  |       imm    dw     alu       | mem_cmd     | | | | mul         |
                //   | | | | | | | |   |       |       |      |      |         | |           | | | | | div       | fence
                //   | | | | | | | |   |       |       |      |      |         | |           | | | | | | wxd     | | amo
                //   | | | | | | | | scie      |       |      |      |         | |           | | | | | | |       | | | dp
                List(N,X,X,X,X,X,X,X,X,A2_X,   A1_X,   IMM_X, DW_X,  FN_X,     N,M_X,        X,X,X,X,X,X,X,CSR.X,X,X,X,X)

- contains a table: Array[(BitPat, List[BitPat])] corresponding to instruction and control signals
- instructions can be found at rocket/Instructions.scala
- constants for control signals found at rocket/Consts.scala

- multiple classes according to needs (IDecode, FenceIDecode, SDecode, etc... ). These are concatenated in Rocket.scala to generate a decode_table

### class BitPat(value, mask, width)
- a bit pattern object to enable representation of dontcares
- for a string "b10_??10", value = "100010", mask = "110011", width = 6


