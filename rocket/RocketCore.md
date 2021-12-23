
## class RocketCoreParams
- sets parameters to the rocket core, lazily evaluated 
- extends `CoreParams` from `tile/Core.scala`


## class RocketCustomCSRs
Not read yet

## class Rocket
- generates IO using `HasCoreIO`, with parameter from `HasRocketCoreParameters`
- `NoChiselNamePrefix` for avoiding RocketImpl being prefixed
- `val decode_table`: a mapping of Instructions -> List of control signals

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
- `write(addr: UInt, data: UInt)`: supports forwarding

