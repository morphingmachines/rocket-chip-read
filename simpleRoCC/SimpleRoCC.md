[RocketTile](../Readme.md)/[simpleRoCC](https://github.com/morphingmachines/RocketTile/tree/master/src/main/scala/simpleRoCC)
=============================
This package has various configurations of RocketTile with ROCCIO

********************
### class SimpleRoCCCommand
**
~~~scala
class SimpleRoCCCommand(xLen: Int) extends Bundle {
  val inst   = new RoCCInstruction
  val rs1    = Bits(xLen.W)
  val rs2    = Bits(xLen.W)
  val status = new MStatus
}
~~~
+ **inst** [RoCCInstruction]()
+ **rs1** `UInt`
+ **rs2** `UInt`
+ **status** [MStatus]()

### class SimpleRoCCResponse


### class SimpleHellaCacheReq


### class SimpleHellaCacheResp

### class SimpleCustomCSRIO

### class SimpleRoCCCoreIO
**

### class RoCCIOBridge

### class RocketTileWithRoCCIO
* RocketTile with RoCCIO *

