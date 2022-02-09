
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

