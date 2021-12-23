## object Instructions
- defines multiple methods each corresponding to an instruction (32b-BitPat) such as `BEQ`, `JAL`, etc...
- the lower 7 bits corresponds to opcodes (see RISCV specification for details)
- upper 16bits of the compressed instructions (`C_XX`) are don't cares
- bits that contain arbitrary numbers (e.g. address) are represented as don't cares

## object Causes
not read yet

## object CSRs
not read yet
