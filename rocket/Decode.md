## object DecodeLogic
- Takes instruction, default list of BitPat, and table to decode 
- 첫번째 CtrlSig에 해당하는 BitPat 부터 시작하여 Instruction을 비교하여 default list of BitPat을 override함. 이러한 방식으로 iterative하게 모든 CtrlSigs를 세팅한다. 

- `def term(lit: BitPat)`: bit-reverses the mask and tranforms BitPat to Term type.

- `def logic(addr: UInt, addrWidth: Int, cache: Map[Term,Bool], terms: Seq[Term])`: 
  cache는 중복된 Term에 대한 solution. cache: Map[Term, Bool] contatins mapping between Term & bool. 
  각각의 Term의 element t에 대해서 addr 와 t.value를 비교하여 일치하는 값이 하나라도 있으면 True를 반환.
  t.mask가 1이면 해당하는 부분은 dontcare로 그 비트는 비교를 생략한다. (나머지 부분만 일치하면 True)

- `private val caches = Map[UInt,Map[Term,Bool]]()` 
  

- `def apply(addr: UInt, default: BitPat, mapping: Iterable[(BitPat, BitPat)]): UInt`
  termvalues는 mapping의 key, value를 모두 Term type으로 변환.
  - mint (minterms): key(decode_table's instruction) of termvalues with its ctrl signal of 1.
  - maxt (maxterms): key(decode_table's instruction) of termvalues with its ctrl signal of 0.
  - dc : key(decode_table's instruction) of termvalues with its ctrl signal of ?.

  `if (((dterm.mask >> i) & 1) != 0`을 통해서 default Term의 각 decoding result가 ?인지를 확인하고, 맞다면 SimplifyDC 를 call하고, 아니라면 Simplify를 call하여 단순화 작업을 한다. (dontcare case is the typical one; other cases(0, 1) check whether the default bit is subject to change)
   
default term
if 0. decoding의 결과가 1이 될 경우만 값을 바꾸어주면 됨. 즉, mint에 해당하는 key들을 카르노방법을 통해 간략화하면 되는데, 추가적으로 dontcare에 해당하는 부분을 같이 넣어 logic을 조금 더 optimize할수 있게 한다.
if 1. decoding의 결과가 0이 될 경우만 값을 바꾸어주면 됨. 즉, maxt에 해당하는 key들을 카르노방법을 통해 간략화하면 되는데, 추가적으로 dontcare에 해당하는 부분을 같이 넣어 logic을 조금 더 optimize할수 있게 한다.
if ?. decoding 결과가 1이 되는 mint 기준으로 optimize하되, maxterm과 intersect하지 않는 dummy term(implicit DC)을 만들어서 추가적인 optimize 기회를 만듬)


- the second apply methods exists to iteratively call the first apply method for each bit pattern. 

## class Term
- arguments: `(val value: BigInt, val mask: BigInt = 0)`
- value에는 해당하는 값 (mask가 1인 부분은 0으로 채워짐). mask는 dontcare인 비트에 대해선 1로 설정.
- prime은 아마도 prime implicants를 의미하는 것 같다.
- `def similar(x: Term)`: 두 Term의 value (BigInt)를 비교하여 그 차이값이 onehot (001000) 꼴이면 True를 반환한다. 
- `def merge(x: Term)`: merge될 term들의 prime을 unset하고, onehot을 제외한 비트의 value와 onehot부분을 ?로 대체한 mask를 새로운 Term을 반환한다. (Note that the prime of new Term is initialized to true)
- `def cover(x: Term)`: check if this (prime Term) includes `x` (minterm)
- `def intersects(x: Term)`: 모든 비트에 대해서 포함관계( {(0,1), (1,0)을 제외한 모든 쌍 (e.g. (1,?), (?,1) )가 성립하는가를 보고, 성립하면 True 아니면 False 반환한다.

## object Simplify
- `def getPrimeImplicants(implicants: Seq[Term], bits: Int): Any` : 
  Make a table of implicant Terms (mint/maxt + dc) and merge the pairs of Terms in unit distance.
  cols는 mask의 dontcare 개수가 같은 minterm들의 묶음.

  table(i)(j)는 i dimension으로는 dontcare개수가 같고, j dimension으로는 value의 bitcount (1의 개수)가 같도록 minterm(decode_table의 instruction)의 묶음.
  `table(i)(j+1).filter(_ similar a).map(_ merge a)`는 현재 table의 element가 unit distance면 merge하여 새로운 element는 mask의 bitcount가 하나 늘었으므로 table(i+1)(j)에 append한다. 이때, 비교하는 두 table의 j dimension index가 1 차이 나기때문에 similar method가 잘 작동한다. (ex. 11000 과 10111은 similar하지만, bitcount 수가 1차이 나는 것이 아니므로 unit distance가 아니다. )

  As each row of the table is processed, add prime terms of the row to the list of prime terms (List[Term])
  sort and return the list of terms

- `def getEssentialPrimeImplicants(prime: Seq[Term], minterms: Seq[Term]): (Seq[Term],Seq[Term],Seq[Term])` : 
   - `val primeCovers`: Converted from the Seqeunce of prime, for each prime get a sequence of minterms which are covered by the prime.
Find the pairs of minterm sequence with inclusive relationship. return the larger sequence.
   - `val essentiallyCovered`: list of minterms which are covered by only one prime term
   - `val essential`: list of essential prime terms
   - `val uncovered`: list of minterms which are not covered by any essential prime term

- `def apply(minterms:Seq[Term], dontcares: Seq[Term], bits: Int): Any` :

- `def getCover`: Not yet covered
- `def getCost`: Not yet covered
    

## object SimplifyDC
- `def apply(minterms: Seq[Term], maxterms: Seq[Term], bits: Int)`:
  getPrimeImplicants(minterms, maxterms, bits) -> Simplify.getEssentialPrimeImplicants -> Simplify.getCover 순으로 EPI(essential prime implicants) 와 나머지 implicants 들(Seq[Term])을 반환. 카르노맵 참고. (Instruction: input variable, Ctrl signal: output)

- `def getImplicitDC(maxterms: Seq[Term], term: Term, bits: Int, above: Boolean): Term`:
  (above가 true일시) variable `t`를 선언하여 null로 만든 후, `if (above && ((term.value | term.mask) & (BigInt(1) << i)) == 0)`은 0인 비트중에서 가장 낮은 significance를 가지는 비트를 1로 바꾸어 저장.
  이 새로운 `t`와 intersect하는 `maxterm`이 전혀 없다면 `t`를 반환한다. 
  이는 decode_table에 포함되지 않은 implicit한 dontcare를 maxterm과 겹치지 않도록 비교하며 찾아내는 과정이다. 이렇게 찾아낸 implictDC는 minterm과 merge되어 추가적인 optimization이 가능하다.
  (above가 false일시) 반대로 1인 비트를 0으로 줄여나가는 반대의 상황이다.


- `def getPrimeImplicants(minterms: Seq[Term], maxterms: Seq[Term], bits: Int)`:
  Make a table of implicant Terms (mint/maxt + dc) and merge the pairs of Terms in unit distance.
  cols는 mask의 dontcare 개수가 같은 minterm들의 묶음.

  table(i)(j)는 i dimension으로는 dontcare개수가 같고, j dimension으로는 value의 bitcount (1의 개수)가 같도록 minterm(decode_table의 instruction)의 묶음.
  `table(i)(j+1).filter(_ similar a).map(_ merge a)`는 현재 table의 element가 unit distance면 merge하여 새로운 element는 mask의 bitcount가 하나 늘었으므로 table(i+1)(j)에 append한다. 이때, 비교하는 두 table의 j dimension index가 1 차이 나기때문에 similar method가 잘 작동한다. (ex. 11000 과 10111은 similar하지만, bitcount 수가 1차이 나는 것이 아니므로 unit distance가 아니다. )

  As each row of the table is processed, ***getImplicitDC and merge it***, then add prime terms of the row to the list of prime terms (List[Term])
  sort and return the list of terms


if default term is 0 then check term of decoded table is 0 or 1 ? if 0 then use mint else use maxt
