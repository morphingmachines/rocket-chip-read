
[Rocket](Readme.md)/[diplomacy](https://github.com/freechipsproject/rocket-chip/tree/master/src/main/scala/diplomacy)
========================

Diplomacy is a system-on-chip (SoC) generation framework for negotiating the parameterization of protocol implementations. Given a description of sets of interconnected master and slave devices, and a bus protocol template, Diplomacy cross-checks the requirements of all connected devices, negotiates free parameters, and supplies final parameter binding to adapters and endpoints for use in their own hardware generation processes. Beyond confirming the mutual compatibility of the system endpoints, Diplomacy enables them to specialize themselves based on knowledge about the capabilities of other endpoints included in a particular system.

In this framework, an SoC design is partitioned into **`LazyModules`** where a `LazyModule` consists of

- one or more sub-modules (child modules) as `LazyModules`
- one or more diplomatic nodes (extends from `BaseNode`), where parameterized hardware is going to be generated. Any module that wants to send or receive parameter information must have one or more nodes.
- `LazyModuleImp`: actual Chisel RTL implementation

The interactions (connections) between `LazyModule`s is defined as directed edges between the diplomatic nodes. Now the parameter negotiation is performed on a diplomatic graph that represents a SoC design. Basically in a diplomatic graph, diplomatic nodes represent points in the design where diplomatic parameters are used to emit hardware and diplomatic edges represent a directed pairing of client (master) and manager (slave) interfaces.


Diplomacy framework employees two-stage elaboration for SoC generation:

1. The first phase is parameter negotiation, wherein the topology of a graph is discovered and the nodes negotiate the value of the parameters on every edge. Parameter negotiation itself consists of two independent processes. Beginning with the source endpoint nodes (root nodes in diplomatic graph), some parameters flow downwards until they have reached all sink nodes (leaf nodes in diplomatic graph). Independently, beginning with the sink endpoint nodes, other parameters flow inwards until they have reached all source nodes. Thus every edge receives a complete set of parameters describing it in both directions.
1. The second phase is concrete hardware (RTL/FIRRTL) generation, in which the Chisel compiler is invoked on the module hierarchy associated with the node graph. As each Chisel module is elaborated, it can make use of the diplomatic parameters that have been precomputed by its associated diplomatic nodes.

**********************

+ **[AddressDecoder](diplomacy/AddressDecoder.md)**
+ **[DeviceTree](diplomacy/DeviceTree.md)**
  device tree generation functions.
+ **[LayzModule](diplomacy/LazyModule.md)**
  a module generator to allow run-time configuration negotiation before the actual module elaboration (generation).
+ **[Nodes](diplomacy/Nodes.md)**
  a port binding elaboration library based on virtual nodes.
+ **[Parameters](diplomacy/Parameters.md)**
  basic parameter definitions (address space).
+ **[Resources](diplomacy/Resources.md)**
  detailed definition of the resource attached to a device (device tree generation and PMA propagation).




