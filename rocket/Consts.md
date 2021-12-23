## trait ScalarOpConstants
- `\_X` CtrlSigs indicates BitPat containing don't cares only (`???`)
- control signal bits definition
    - `BR`: branch
    - `A{1,2}`: ALU operand
    - `IMM` : immediate 
    - `DW`: Double Word // TODO: check again later


## trait MemoryOpConstants
- Memory Related singal bits
- methods like isAMO, isAMOLogical takes cmd as argument and returns T/F
