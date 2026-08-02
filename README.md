<div align="center">

# Tanmay Gulhane

**RISC-V from the compiler down to the RTL: LLVM and GCC backends, hardware security extensions, and verification built to survive someone checking it.**

I build things that have to work outside controlled environments.

B.Tech Robotics & Automation · MIT World Peace University, Pune

[![Email](https://img.shields.io/badge/email-tanmaygulhane12%40gmail.com-blue?style=flat&logo=gmail)](mailto:tanmaygulhane12@gmail.com)
[![Hugging Face](https://img.shields.io/badge/Hugging%20Face-tanmaytrg-yellow?style=flat&logo=huggingface)](https://huggingface.co/spaces/tanmaytrg/Pretrained_CNN_6_class_image_classification)

</div>

## What I do

I work on RISC-V compiler toolchains, and on proving that low-level systems do what their specifications say. That runs in both directions: upstream patches to LLVM and GCC, RTL I have written and verified myself, and correctness findings in a real RISC-V core, each reproduced in simulation before I file it.

Everything here is reproducible from a single command. I document the tests that fail instead of tuning them away, I report the measurements that contradicted what I expected, and I defer performance claims to real hardware rather than dressing up emulator timings as throughput.

---

## Compilers, upstream

**LLVM, RISC-V backend** · [llvm-project#202201](https://github.com/llvm/llvm-project/pull/202201)

Fixed an assertion in the RISC-V codegen combine (`combineBinOpOfExtractToReduceTree`) that crashed on a `<1 x i1>` operation under `zve32x`. Found by fuzzing the backend with `llvm-stress`, reduced with `llvm-reduce`, diagnosed as a combine asserting where it should have declined. Regression test added, full RISC-V codegen suite passing. Reviewed and approved by a RISC-V LLVM maintainer.

**GCC, loop vectorizer** · [PR tree-optimization/116338](https://gcc.gnu.org/pipermail/gcc-patches/2026-July/725327.html)

Taught the vectorizer to handle chained first-order recurrences, a missed optimization that left TSVC s255 scalar in GCC while clang vectorized it. Adds a chain-detection function with cycle and value-cycle rejection, and fixes the permute insertion point for chained recurrences. Three new testcases. Bootstrapped and regression tested on x86_64 with no regressions, verified on riscv64 under QEMU with bit-exact output, and confirmed newly vectorizing on aarch64 and armv8l by the Linaro precommit CI.

---

## RISC-V security: Data-Flow Integrity

**[riscv-dfi](https://github.com/trg-rgb/riscv-dfi)**. DFI for RISC-V, from the compiler down.

An LLVM 18 pass that instruments every store with a SETDEF and every load with a CHECKDEF against a shadow reaching-definition table. It catches a non-control-data memory corruption that Control-Flow Integrity structurally cannot see, and does not false-positive on legitimate same-object access. Ships with a file-level map for porting the matching hardware extension onto the BSC Sargantana core: decode, check unit at the load-store unit, enforcement at retire, CSR state and fault path, with verified file and line references, and an explicit list of the design decisions it deliberately leaves open. Reproduced on two machines; CI asserts the demo actually traps.

**[count-mem-ops](https://github.com/trg-rgb/count-mem-ops)** is the pass that preceded it, counting load and store instructions per function. CI re-derives the expected counts from the raw IR and fails on mismatch.

---

## Hardware design and verification

**[cam](https://github.com/trg-rgb/cam)**. A parameterised content-addressable memory in SystemVerilog, with the verification it would need before anyone trusted it.

Four entries, 32-bit keys, one comparator per entry. Duplicate keys are resolved deterministically and reported on a status output rather than declared illegal and left undefined, because in a metadata table a duplicate address means the allocator has gone wrong and that is worth surfacing. Five concurrent assertions live in the design rather than in the testbench, so they hold for any instantiation.

Verified three ways, all of it from `./run.sh`, all of it on packages you can `apt-get install`, none of it needing a commercial licence.

**Simulation.** A reference model written in a deliberately different style from the design, eight directed cases at the corners a CAM actually gets wrong, then 20,000 randomised cycles compared every cycle. 20,020 checks, 0 errors, and thirteen functional coverage bins where the run *fails* if any bin was never exercised. Identical results under Verilator and Icarus, which are independent implementations, so agreement between them says more than a pass on either alone.

**Formal.** Every assertion proved for all reachable states by temporal induction, in four configurations, on the SAT engine built into Yosys. That includes the property simulation is worst at reaching, which is completeness: if any valid entry holds the searched key, the CAM must report a hit. A CAM that silently misses is the dangerous failure, not one that over-reports. The proof is also mutation tested, because a proof nothing can falsify proves nothing: five one-line bugs are injected one at a time and the run fails if any of them survives.

**Synthesis.** A sweep across depth, key width and the output-register setting, reporting cell count and logic depth on two Yosys versions, honest that without a PDK these are not areas in square micrometres. Two negative results are kept in the README rather than quietly removed: the output register did not shorten the measured critical path, and the cleverer priority encoder I tried was worse on both axes because I had replaced one carry chain with another.

Written as the hardware coding challenge for the Sargantana tightly-coupled DFI project, since an associative lookup from an address to the identifier of the instruction that last wrote it is exactly the structure a DFI metadata table is.

---

## RISC-V core: correctness findings in BSC Sargantana

Seven findings in the Sargantana core and its CSR module. Six are filed as a paired issue and pull request, and every one was reproduced on the core_tile Verilator simulator, with before-and-after output, before I filed it.

| Finding | Where |
|---|---|
| `vwsll` checked register alignment on the narrow source instead of the wide destination, both rejecting legal encodings and accepting illegal ones | [#70](https://github.com/bsc-loca/sargantana/issues/70), [#71](https://github.com/bsc-loca/sargantana/pull/71) |
| Zvbb VXUNARY0 ops denied in-place operation (`vd == vs2`) that RVV 1.0 permits for same-width unary ops | [#66](https://github.com/bsc-loca/sargantana/issues/66), [#69](https://github.com/bsc-loca/sargantana/pull/69) |
| `vfwcvt` and `vfncvt` skipped the widening and narrowing alignment check every integer widening op enforces | [#75](https://github.com/bsc-loca/sargantana/issues/75), [#76](https://github.com/bsc-loca/sargantana/pull/76) |
| The vector permute ops (`vrgather`, `vslideup`, `vslide1up`, `vcompress`) accepted reserved `vd`/`vs` overlap encodings the specification forbids | [#89](https://github.com/bsc-loca/sargantana/issues/89), [#90](https://github.com/bsc-loca/sargantana/pull/90) |
| `stvec` and `sepc` applied weaker WARL masking than `mtvec` and `mepc` | [csr#5](https://github.com/bsc-loca/csr/issues/5), [csr#6](https://github.com/bsc-loca/csr/pull/6) |
| `vsstatus.SD` did not summarise `VS`, so a guest OS testing the summary bit would skip saving dirty vector state across a context switch, silently and without a fault | [csr#8](https://github.com/bsc-loca/csr/issues/8), [csr#9](https://github.com/bsc-loca/csr/pull/9) |
| Widening and narrowing ops are limited to `vl <= VLMAX/2`, which software has no architectural way to discover | [#91](https://github.com/bsc-loca/sargantana/issues/91) |

The maintainer has marked three of these for the next release: the Zvbb fix outright, and the two alignment fixes once I added an overlap guard he asked for, which is now in both pull requests and verified at fractional LMUL against the reference model, with the full 315-test ISA suite re-run unchanged.

**The last row is not a bug, and that is the point.** A randomised differential fuzzer I wrote, which generates RVV programs and compares the core against the spike model the repository already vendors, flagged widening operations raising an illegal instruction above `vl = VLMAX/2`. The reproducer was minimal and it looked like a defect. Before filing I traced the limit through `vset_module.sv:98` into every widening guard in the decoder and concluded BSC had done it deliberately, so the issue says so in its title, asks for a documentation note instead of a fix, and explains the consequence: `vsetvli` for VLMAX followed by `vwadd` is what an autovectorizer emits, and it traps.

---

## RISC-V HPC porting: [riscv-hpc-port](https://github.com/trg-rgb/riscv-hpc-port)

A portfolio of HPC, scientific, and ML software cross-compiled and forensically verified for riscv64, built on x86_64 with `riscv64-linux-gnu-gcc 15.2` and run under `qemu-riscv64`.

| Port | Result |
|------|--------|
| **LAMMPS** (molecular dynamics) | Cross-compiled to riscv64 with **zero upstream patches**. Packaged as a one-command installable `.deb`. |
| **OpenMM 8.5** (molecular dynamics) | Ported with a 4-hunk upstream-friendly patch. **12/12 platform tests passing**, 861 RVV instructions in the hot nonbonded kernel. |
| **OpenBLAS** (linear algebra) | Built for the RVV-1.0 (ZVL128B) target. Documentation PR **merged upstream** ([#5819](https://github.com/OpenMathLib/OpenBLAS/pull/5819)). |
| **TensorFlow Lite** | Full TF Lite cross-compiled for riscv64, running INT8 CNN inference on a real model. |
| **Eigen 5.0** | Cross-compiled and tested under QEMU, 42/42 tests passing. |
| **Chocolate Doom** | Running on riscv64 under full-machine emulation, with a deterministic timedemo matching x86 frame-for-frame. |

**Tooling:** `verify-rvv-port.sh`, a forensic verifier I wrote that statically checks whether a binary's hot path is genuinely vectorized, rather than silently falling back to scalar. It caught an incorrect figure in my own published results before I shipped them, which is exactly why it exists.

Also merged upstream: [RuyiSDK packages-index #195](https://github.com/ruyisdk/packages-index/pull/195) (OpenCV manifest).

---

## Machine learning: [Pretrained-CNN](https://github.com/trg-rgb/Pretrained-CNN-with-6-class-image-classification)

6-class CNN for groundnut leaf disease detection. **Gold Medal, First Place at Sci Quest 2025, MIT-WPU**, against students from all branches and years. Trained, optimized for inference, and deployed live. **[Try the demo](https://huggingface.co/spaces/tanmaytrg/Pretrained_CNN_6_class_image_classification)**

The INT8-quantized version of this model is what later ran inference on RISC-V via TF Lite, which is how the ML work and the systems work connect.

---

## Tech

![C](https://img.shields.io/badge/C-00599C?style=flat&logo=c&logoColor=white)
![C++](https://img.shields.io/badge/C++-00599C?style=flat&logo=cplusplus&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![LLVM](https://img.shields.io/badge/LLVM-262D3A?style=flat&logo=llvm&logoColor=white)
![GCC](https://img.shields.io/badge/GCC-A42E2B?style=flat&logo=gnu&logoColor=white)
![SystemVerilog](https://img.shields.io/badge/SystemVerilog-1D5E8A?style=flat)
![Verilator](https://img.shields.io/badge/Verilator-2E6E4E?style=flat)
![Icarus Verilog](https://img.shields.io/badge/Icarus%20Verilog-4B7BA8?style=flat)
![Yosys](https://img.shields.io/badge/Yosys-6A4C93?style=flat)
![QEMU](https://img.shields.io/badge/QEMU-FF6600?style=flat&logo=qemu&logoColor=white)
![CMake](https://img.shields.io/badge/CMake-064F8C?style=flat&logo=cmake&logoColor=white)
![TensorFlow](https://img.shields.io/badge/TensorFlow-FF6F00?style=flat&logo=tensorflow&logoColor=white)
![RISC-V](https://img.shields.io/badge/RISC--V-283272?style=flat)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat&logo=linux&logoColor=black)

---

<div align="center">
B.Tech 2025–2029 · open to internships in RISC-V, compilers, systems software, and toolchains

</div>
