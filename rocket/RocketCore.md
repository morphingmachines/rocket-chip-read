
## class RocketCoreParams
- sets parameters to the rocket core, lazily evaluated 
- extends `CoreParams` from `tile/Core.scala`


## class RocketCustomCSRs
Not read yet

## class Rocket
- generates IO using `HasCoreIO`, with parameter from `HasRocketCoreParameters`
- `NoChiselNamePrefix` for avoiding RocketImpl being prefixed
- `XX_reg_YY`: instantiates XX stage registers named YY

#### Decode Stage
- `val decode_table`: a mapping of Instructions -> List of control signals which are assigned into `id_ctrl` Wire.
- `def decodeReg`: checks whether the instruction is valid where number of Reg differs (in case of RVE or not)
- `val rf`: instatiates a RegFile. 31 entries RV32, 15 entries in RVE, each with `xLen` width.
- `val id_rs`: contains rs1, rs2 values read from the RegFile
- CSRFile (Control Status Register File): not read yet
- SCIEDecoder (SiFive Custom Instruction Extension Decoder): not read yet
- BPU (Breakpoint Unit): not read yet
- Picks the highest priority (exception, cause) from various sources(`csr`, `bpu`, `FrontendResp`,`id_illegal_inst`) and store it into `(id_xcpt, id_cause)`
- Detect bypass opportunities
  - `bypass_sources`는 EX, MEM stage에서 write하는 instrcution인지, write address, 해당하는 wdata를 IndexedSeq에 저장한다.
  - EX, MEM stage에서 write을 하는 instruction인지와 write address가 현 ID stage의 read address가 같은지를 판단하여 bypass 필요여부를 `id_bypass_src`에 넣어둔다


#### Execute Stage

- `val bypass_mux`: map bypassed data from `bypass_sources`
- `val ex_rs` : for each rs, yield a mux to select bypassed data indexed by rs_lsb or original rs data
- `val ex_op1` and `ex_op2` : select which operands to use (rs1/pc, rs2/imm/size)
- `val alu` : instantiate a alu module with injecting io signals (dw(double word), fn(function), in2, in1)
- `val div` : instantiate a MulDiv module (when `pipelinedMul` is turned off then set mulUnroll to 0)
- `val mul` : instantiate a pipelinedMul module
- `ex_reg_mem_size := id_inst(0)(13,12)` : [13:12] bit field represents log2 of data width in Bytes (00: 8bit(LB), 01: 16bit(LH), 10: 32bit(LW)) (+ [14] bit field represents unsigned memory operation)
![load instruction](./load.png)
- `val do_bypass` : checks whether bypassing needed
- `val bypass_src` : select the source to bypass by PriorityEncoder (x0 -> mem_wdata -> wb_wdata -> D$data)
- `val ex_reg_rs_lsb` and `val ex_reg_rs_msb` : When bypassing occurs, assign bypass source to lsb and ignore msb. Otherwise, pass through original rs.
- `when (id_illegal_insn)` : store the instruction in rs1
- `val ctrl_killx` : checks whether to replay inst in ex stage // TODO: put in a control flow diagram
- `val ex_slow_bypass` : set flag of slow bypassing when it takes 2 cycles to use data from LB/LH/SC




### class Scoreboard
- keeps information about the (long-latency) instructions being executed. The register has to be kept reserved until the instruction is finished thus invoking the set method. When instruction finishes, it clears the bit.
- set(en: Bool, addr: UInt): updates the bit of the sboard
- clear(en: Bool, addr: UInt): clears the bit of the sboard
- read(addr: UInt): reads sboard content

## class RegFile
- arguments: `(n: Int, w: Int, zero: Boolean = false)`
  - `xLen` as width `w`
  - `regAddrMask` as entry number `n`: 15 if `useRVE (embedded)` else 31
- used to make register file of the core
- rf is implemented as Mem(n, w)
- `read(addr: UInt)`: reads from certain register
- `write(addr: UInt, data: UInt)`: supports forwarding (when `wb_waddr == id_radder`, other cases are dealt using `bypass_sources`)


