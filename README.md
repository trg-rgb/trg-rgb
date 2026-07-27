<div align="center">

# Tanmay Gulhane

**Compiler and toolchain work on RISC-V: LLVM and GCC backends, hardware security extensions, and correctness verification.**

I build things that have to work outside controlled environments.

B.Tech Robotics & Automation · MIT World Peace University, Pune

[![Email](https://img.shields.io/badge/email-tanmaygulhane12%40gmail.com-blue?style=flat&logo=gmail)](mailto:tanmaygulhane12@gmail.com)
[![Hugging Face](https://img.shields.io/badge/Hugging%20Face-tanmaytrg-yellow?style=flat&logo=huggingface)](https://huggingface.co/spaces/tanmaytrg/Pretrained_CNN_6_class_image_classification)

</div>

## What I do

I work on RISC-V compiler toolchains and on proving that low-level systems do what their specifications say. That runs in both directions: upstream patches to LLVM and GCC, and correctness findings in a real RISC-V core, each reproduced in simulation before I file it.

Everything here is reproducible from a single command. I document the tests that fail instead of tuning them away, and I defer performance claims to real hardware rather than dressing up emulator timings as throughput.

---

## Compilers, upstream

**LLVM, RISC-V backend** · [llvm-project#202201](https://github.com/llvm/llvm-project/pull/202201)

Fixed an assertion in the RISC-V codegen combine (`combineBinOpOfExtractToReduceTree`) that crashed on a `<1 x i1>` operation under `zve32x`. Found by fuzzing the backend with `llvm-stress`, reduced with `llvm-reduce`, diagnosed as a combine asserting where it should have declined. Regression test added, full RISC-V codegen suite passing. Reviewed and approved by a RISC-V LLVM maintainer.

**GCC, loop vectorizer** · [PR tree-optimization/116338](https://gcc.gnu.org/pipermail/gcc-patches/2026-July/725327.html)

Taught the vectorizer to handle chained first-order recurrences, a missed optimization that left TSVC s255 scalar in GCC while clang vectorized it. Adds a chain-detection function with cycle and value-cycle rejection, and fixes the permute insertion point for chained recurrences. Three new testcases. Bootstrapped and regression tested on x86_64 with no regressions, verified on riscv64 under QEMU with bit-exact output, and confirmed newly vectorizing on aarch64 and armv8l by the Linaro precommit CI.

---

## RISC-V security: Data-Flow Integrity

**[riscv-dfi](https://github.com/trg-rgb/riscv-dfi)** — DFI for RISC-V, from the compiler down.

An LLVM 18 pass that instruments every store with a SETDEF and every load with a CHECKDEF against a shadow reaching-definition table. It catches a non-control-data memory corruption that Control-Flow Integrity structurally cannot see, and does not false-positive on legitimate same-object access. Ships with a file-level map for porting the matching hardware extension onto the BSC Sargantana core: decode, check unit at the load-store unit, enforcement at retire, CSR state and fault path, with verified file and line references. Reproduced on two machines; CI asserts the demo actually traps.

**[count-mem-ops](https://github.com/trg-rgb/count-mem-ops)** — the pass that preceded it, counting load and store instructions per function. CI re-derives the expected counts from the raw IR and fails on mismatch.

---

## RISC-V core: correctness findings in BSC Sargantana

Four findings in the Sargantana core and its CSR module, each filed as a paired issue and pull request, and each reproduced on the core_tile Verilator simulator with before-and-after output before filing.

| Finding | Where |
|---|---|
| `vwsll` checked register alignment on the narrow source instead of the wide destination, both rejecting legal encodings and accepting illegal ones | [#70](https://github.com/bsc-loca/sargantana/issues/70), [#71](https://github.com/bsc-loca/sargantana/pull/71) |
| Zvbb VXUNARY0 ops denied in-place operation (`vd == vs2`) that RVV 1.0 permits for same-width unary ops | [#66](https://github.com/bsc-loca/sargantana/issues/66), [#69](https://github.com/bsc-loca/sargantana/pull/69) |
| `vfwcvt` and `vfncvt` skipped the widen and narrow alignment check every integer widening op enforces | [#75](https://github.com/bsc-loca/sargantana/issues/75), [#76](https://github.com/bsc-loca/sargantana/pull/76) |
| `stvec` and `sepc` applied weaker WARL masking than `mtvec` and `mepc` | [csr#5](https://github.com/bsc-loca/csr/issues/5), [csr#6](https://github.com/bsc-loca/csr/pull/6) |

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
![QEMU](https://img.shields.io/badge/QEMU-FF6600?style=flat&logo=qemu&logoColor=white)
![CMake](https://img.shields.io/badge/CMake-064F8C?style=flat&logo=cmake&logoColor=white)
![TensorFlow](https://img.shields.io/badge/TensorFlow-FF6F00?style=flat&logo=tensorflow&logoColor=white)
![RISC-V](https://img.shields.io/badge/RISC--V-283272?style=flat)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat&logo=linux&logoColor=black)

---

<div align="center">
B.Tech 2025–2029 · open to internships in RISC-V, compilers, systems software, and toolchains

</div>

