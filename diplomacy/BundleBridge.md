[Rocket](../Readme.md)/[diplomacy](../diplomacy.md)/[BundleBridge](https://github.com/freechipsproject/rocket-chip/blob/master/src/main/scala/diplomacy/BundleBridge.scala)
=====================
*Use BundleBridge to turn non-diplomatic IO into diplomatic IO*

> Explanation:
> - A module has either normal IO or diplomatic IO - this can be decided at module level. But when connecting the module in a larger context, the normal IOs can be converted into diplomatic IOs using BundleBridge
> - For example, a single CE tile has the following normal IOs: reset, bootROMReset, hartID - and can be individually connected to external IOs. But when connecting multiple CE tiles - the configuration of connection (i.e. the way it’s connected) - can become a design time parameter. Therefore, in the multiple CE tile case, these normal IOs have to become diplomatic IOs to negotiate parameters at system level. For example, the reset address can change based on the number of CE tiles and hence has to be negotiated at system level.

> Explanation from References:   
> [Adding external IO to RoCC accelerators](https://groups.google.com/g/chipyard/c/_q6b4OneO14/m/LGrx308oAwAJ)      
> - Probably the best way is to use BundleBridge diplomacy nodes. The general idea is that you create a "BundleBridgeSink" in a custom DigitalSystem trait, and add a BundleBridgeSource in the BaseTile definition. Then when you connect the BundleBridge nodes together, diplomacy will generate IOs between your source and sink. Connecting these two nodes may be tricky, since the rocc accelerator is generated within a function. Another approach could be BoringUtils, https://www.chisel-lang.org/api/latest/chisel3/util/experimental/BoringUtils$.html. But BundleBridges are cleaner and safer.
> [Connecting A Device to Rocket Signals](https://groups.google.com/g/chipyard/c/XRS0qayFwKM/m/7LlzAXf2AAAJ)           
> - Use BundleBridge to turn non-diplomatic IO into diplomatic IO
> - Create a BundleBridgeSinkNode(or SourceNode) to turn diplomatic IO into non-diplomatic IO