Welcome to the RISC-V Code Size Reduction Group
------------------------------------------------

This will be the home for the all of the code size reduction proposals, analysis, results etc.

Documentation of existing ISA extensions
- [Existing ISA extensions to reduce code size](https://github.com/riscv/riscv-code-size-reduction/blob/master/existing_extensions/README.md)

ISA extension proposals
- (none yet)

Publicly available benchmarks
- [Embench](https://github.com/embench/embench-iot)
- softfloat, link needed
- others?

Proprietary benchmarks
- Huawei IoT code
- others?

Reference Architectures
-----------------------

- ARMv7-M / Cortex-M3 [manual is here](https://developer.arm.com/documentation/ddi0403/ed/)
- ARCv1 / ARC700 [manual is here](http://me.bios.io/images/d/dd/ARCompactISA_ProgrammersReference.pdf)

ARCv2 would be better but is proprietary (ISA and toolchain)

Reference Toolchains
--------------------

- ARM GCC / LLVM? Version / download link?
- [ARC](https://github.com/foss-for-synopsys-dwc-arc-processors/toolchain/releases)

Code size reduction ideas
-------------------------

Need a lot more detail for these, they're just placeholders at the moment

- runtime library optimisation
- link time optimisation including dead code elimination
- function prologue/epilogue optimisation in software, to close the gap with the PUSH/POP ISA extension proposal
- smaller instruction sequences to jump to distant addresses
- smaller instruction sequences to load/store to distant addresses
- smaller instruction sequences to load 32-bit constants
- many more I'm sure......

Experiments
-----------

- enable B-extension, maybe a subset could become part of a future code-size reduction ISA extension





