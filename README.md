Welcome to the RISC-V Code Size Reduction Group
------------------------------------------------

_If you can't measure it you can't improve it_

This is the current proposal for reducing code size:

- [Zce ISA proposal](https://github.com/riscv/riscv-code-size-reduction/blob/master/ISA%20proposals/Huawei/Zce_spec.adoc)

This will be the home for the all of the code size reduction proposals, analysis, results etc.

Documentation of existing ISA extensions
- [Existing ISA extensions to reduce code size](https://github.com/riscv/riscv-code-size-reduction/blob/master/existing_extensions/README.md)

Toolchain optimisation suggestions
- Anders: Second GP (thread pointer - requires ABI change) to allow more data to be in easy reach without building long addresses
- Jeremy: Recognising similar/same constants in the linker and simplifying them
- Jim Wilson: fix -mno-strict-align
- Reported by Matteo: https://github.com/riscv/riscv-gcc/issues/193
- Use C.SWSP instead of SB/SH when zeroing stack variables, so can use 16-bit encodings and if possible group variables together to allow fewer stores to cover them all

Code size analysis suggestions
- Build a histogram of instruction distribution
- find common pairs of instructions to see if they can be combined

Publicly available benchmarks
- [Embench](https://github.com/embench/embench-iot)

  - antecedents [BEEBS](https://github.com/mageec/beebs), [MiBench](http://vhosts.eecs.umich.edu/mibench/), [Mälardalen WCET benchmarks](https://drops.dagstuhl.de/opus/volltexte/2010/2833/pdf/15.pdf), [Hacker's Delight/A Hacker's Assistant](https://en.wikipedia.org/wiki/Hacker%27s_Delight)

= [Hi3961 Huawei WiFi IoT platform](https://github.com/riscv/riscv-code-size-reduction/tree/master/benchmarks/Hi3861_WiFi_IoT)

- [Code size benchmarks](http://szeged.github.io/csibe/)
- [Opus codec](https://github.com/xiph/opus)
- [TACLe Benchmarks](http://www.tacle.eu/)

  - I didn't see any code here, just details of a committee...... Tariq

- [softfloat](http://www.jhauser.us/arithmetic/SoftFloat.html) and [testfloat](http://www.jhauser.us/arithmetic/TestFloat.html)
- [MLPerf](https://mlperf.org/) - specifically for AI/ML applications
 
  - Seems to be Debian installation only which I don't have available....... Tariq

- [BDTI](https://www.bdti.com/services/bdti-dsp-kernel-benchmarks)

   - Need to be licensed? It's a US company so I can't talk to them.... Tariq

- Zephyr includes BLE (bluetooth)

  - [Nordic SDK](https://www.nordicsemi.com/Software-and-tools/Software/nRF-Connect-SDK)
  - [Examples](https://developer.nordicsemi.com/nRF_Connect_SDK/doc/latest/nrf/examples.html)

- Need to find more device drivers....
- LLVM test suite

Proprietary benchmarks
- Huawei IoT code
- others?

Useful papers
- [Peijie Li's Berkeley paper](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2019/EECS-2019-107.pdf)

Useful software
- [RISCV-NEWOP - auto detect fused instructions from a spike trace and add the instructions to spike](https://github.com/riscv-newop/riscv-newop) . For details of the presenter [click here](https://computerscience.engineering.unt.edu/people/faculty/nagendra-gulur)

Reference Architectures
-----------------------

These are architectures we could compare against. The "official" comparison architectures have not yet been decided, but almost certainly need freely available ISA manuals and GCC+LLVM ports

- ARMv7-M / Cortex-M3 [manual is here](https://developer.arm.com/documentation/ddi0403/ed/)
- ARCv1 / ARC700 [manual is here](http://me.bios.io/images/d/dd/ARCompactISA_ProgrammersReference.pdf) _ARCv2 would be better but is proprietary (ISA and toolchain)_
- NanoMIPs [manual is here](https://s3-eu-west-1.amazonaws.com/downloads-mips/I7200/I7200+product+launch/MIPS_nanomips32_ISA_TRM_01_01_MD01247.pdf) see "save" instruction on page 163 and "restore/restore.js" instructions on page 155
- AVR32 [manual is here](http://ww1.microchip.com/downloads/en/DeviceDoc/doc32000.pdf)
- J-core [manual is here](https://j-core.org/) a reimplementation of Hitachi SH2
- SuperH [instruction reference is here](http://www.shared-ptr.com/sh_insns.html)

Reference Toolchains
--------------------

- [GNU Arm Embedded Toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads) / LLVM? Version / download link?
- [ARC](https://github.com/foss-for-synopsys-dwc-arc-processors/toolchain/releases)

Benchmark issues
-------------------------

- Embench:
    - Add bigger programs to be more representative.
    - Add FP intensive benchmarks.
    - Cubic benchmark uses _long double_ type, which is interpreted as 128-bit by RISC-V and 64-bit by Arm. This brings to possibly unfair comparisons both with and without libraries. In the former case, the linker links FP-functions to deal with quad-precision FP numbers, which are way heavier than the ones for double-precision; however, even without considering the libraries, the RISC-V code size is bloated because the quad-precision FP numbers should be passed by reference, with higher memory usage.

RISC-V Known Compiler Inefficiencies
-------------------------

- Frame pointer allocation inefficiency for > 4KiB data on the stack: https://github.com/riscv/riscv-gcc/issues/193

Linker script and Global Pointer optimization
-------------------------

The position of the Global Pointer (GP) is fundamental for the code size problem since memory operations exploit it as a base register for the addressing without the need to pre-load an immediate (the base for the addressing) in a register. Given the 12-bit signed-immediate encoding of the memory operations, the GP is effective when the memory address falls within the interval [GP-2KiB, GP+2KiB), so frequently accessed data should be placed in this range to maximize the code-size/performance benefit. This operation can be done inside the linker script.

ABI modification proposal
-------------------------

A second GP would be especially beneficial for applications with scattered memory maps, in which some important data can be put far away from the GP. A possible approach can be a modification of the ABI to free-up the Thread Pointer (TP) register, which is often unused in many applications. This special register cannot be allocated by the compiler, and making it general-purpose or modifying its actual role (e.g., a second GP) could benefit both code-size and performance of the embedded applications. As already mentioned, the TP can also be used as a general-purpose register. This would be important also for the RV32E extension, as the TP is one of the 16 available registers (x4).

Library support
-------------------------

Optimized library functions can make the difference for two reasons:
 - Code tailored for embedded application
 - Smaller + Faster code, assembly-optimized in the critical parts
 - No intricate inter-dependencies, which call a lot of different functions even if they are not fundamental

Different libraries to optimize:
 - Low-level runtime library (e.g. libgcc)
  - Optimize the FP functions, which bloat the code especially when linked to small programs
 - Other _external_ libraries, like newlib
  - picolibc by Keith Packard (https://keithp.com/blogs/picolibc/, https://github.com/picolibc/picolibc) is the evolution of newlib-nano, a shrunk version of newlib. Citing the GitHub description: _"Picolibc is library offering standard C library APIs that targets small embedded systems with limited RAM. Picolibc was formed by blending code from Newlib and AVR Libc."_

Code size reduction ideas
-------------------------

Need a lot more detail for these, they're just placeholders at the moment

- runtime library optimisation
- link time optimisation including dead code elimination
- function prologue/epilogue optimisation in software, to close the gap with the PUSH/POP ISA extension proposal
- smaller instruction sequences to jump to distant addresses
  - including jump-chaining, where a tree of jumps is built to allow access to distant functions using mainly shorter encodings
- smaller instruction sequences to load/store to distant addresses
- smaller instruction sequences to load 32-bit constants
- load/store multiple
  - specified as a register list, or as base register and register count?
  - what's best for the compiler? base + count could probably fit in a 16-bit encoding
- ADD instruction(s) with a carry out for chaining into longer ADDs (Matteo)
  - add sl, xl, yl              -> implicitly save the carry bit
  - addH sh, xh, yh        <- implicitly add the carry bit
  - But I don't know if outside the soft-float FP functions we would have significant gains.


From Anders Lindgren:

- Better support for 8 and 16 bit data

  - Today, most RISC-V instructions work on the full registers. This makes the generated code more efficient to handle 32 bit data than 8 and 16 bit data. Effectively, the compiler must ensure that 8 and 16 bit data are properly extended before it can perform things like compares on them. To make things worse, RISC-V doesn't provide instructions to perform extensions so typically two instructions are needed to perform extensions (with the exception of 8 bit zero extension which can be done using "ANDI Rd, Rs, 0xFF"). Instructions to perform sign and zero extend (preferably with compact variants) are obvious candidates. In addition, we could consider 8 and 16 bit variants (and for RV64 32 bit variants) for various instructions like compare, right shift, division, and modulo. One thing that makes the situation worse is that the ABI requires arguments and return values to be correctly extended. Hence a small function like "short f(short x, short y) { return x + y; }" require 4 instructions (add, shift left, signed shift right, ret). I would like to see if the overall code size would shrink if the ABI didn't require this, and, if so, recommend that the EABI (which isn't ratified) is changed to that fewer extension instructions are needed.

- Insert and extract parts of registers

  - If it would be easier to insert and extract parts of registers, we could avoid storing things on the stack. Concretely, a RV32 processor register could be used to store four bytes or two halfwords.

- Improved compare with constants

  - Today, when comparing a value against a non-zero constant, at least two instructions are needed. Instructions that compare a register against commonly used constants (imm5?) could reduce code size. We need to see which constants and which comparisons are most effective.
   - See [this proposal for combined compare-immediate-branch](https://github.com/riscv/riscv-code-size-reduction/blob/master/existing_extensions/Huawei%20Custom%20Extension/riscv_condbr_imm_extension.rst)

- Address calculations with scaling

  - In C, when doing address calculations, the index value is scaled with the object size to produce the end address. Today, this is done using an explicit shift (when the size of the object is a power of two) or a multiplication. We should look into loads, stores, and load-effective-address with this scaling builtin. Since most arrays use elements of size 2, 4, and 8 we could restrict ourselves to this.

Experiments
-----------

- enable B-extension, maybe a subset could become part of a future code-size reduction ISA extension

Outputs from the group
----------------------

- Improved open source compiler technology (GCC and LLVM)
  - code size optimised compilers with and without `Zce` (see below)
  - for example function prologue/epilogue should be smaller than _-msave-restore_ is now in GCC.
- Code size reduction ISA extensions






