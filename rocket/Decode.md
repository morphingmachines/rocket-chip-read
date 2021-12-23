## object DecodeLogic
- Takes instruction, default list of BitPat, and table to decode 
- 2 apply methods exists to recursively handle decoding logic. 
- 첫번째 CtrlSig에 해당하는 BitPat 부터 시작하여 Instruction을 비교하여 default list of BitPat을 override함. 이러한 방식으로 iterative하게 모든 CtrlSigs를 세팅한다. 

## class Term(val value: BigInt, val mask: BigInt = 0)

## object Simplify
