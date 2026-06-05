<div align="center">

 # Tanmay Gulhane
 
**I port HPC libraries to RISC-V and verify they actually vectorize.**
 
I build things that have to work outside controlled environments.
 
B.Tech Robotics & Automation · MIT World Peace University, Pune
 
[![Email](https://img.shields.io/badge/email-tanmaygulhane12%40gmail.com-blue?style=flat&logo=gmail)](mailto:tanmaygulhane12@gmail.com)
[![Hugging Face](https://img.shields.io/badge/Hugging%20Face-tanmaytrg-yellow?style=flat&logo=huggingface)](https://huggingface.co/spaces/tanmaytrg/Pretrained_CNN_6_class_image_classification)
 
</div>
---
 
## What I do
 
I cross-compile HPC and scientific libraries to RISC-V (riscv64), then check, by disassembling the result, that the hot loops genuinely use the vector unit instead of silently falling back to scalar code. My focus is embedded systems, computer architecture, and machine learning on constrained hardware.
 
Everything is verified under QEMU and reproducible from a single command. I document the tests that fail instead of tuning them away, and I defer performance claims to real hardware rather than dressing up emulator timings as throughput.
 
---
 
## RISC-V HPC porting: [riscv-hpc-port](https://github.com/trg-rgb/riscv-hpc-port)
 
A portfolio of HPC, scientific, and ML software cross-compiled and forensically verified for riscv64, built on x86_64 with `riscv64-linux-gnu-gcc 15.2` and run under `qemu-riscv64`.
 
| Port | Result |
|------|--------|
| **LAMMPS** (molecular dynamics) | Cross-compiled to riscv64 with **zero upstream patches**. Packaged as a one-command installable `.deb`. |
| **OpenMM 8.5** (molecular dynamics) | Ported with a 4-hunk upstream-friendly patch. **12/12 platform tests passing**, 861 RVV instructions in the hot nonbonded kernel. |
| **OpenBLAS** (linear algebra) | Built for the RVV-1.0 (ZVL128B) target. Documentation PR under review upstream. |
| **TensorFlow Lite** | Full TF Lite cross-compiled for riscv64, running INT8 CNN inference on a real model. |
| **Eigen 5.0** | Cross-compiled and tested under QEMU, 42/42 tests passing. |
| **Chocolate Doom** | Running on riscv64 under full-machine emulation, with a deterministic timedemo matching x86 frame-for-frame. |
 
**Tooling:** `verify-rvv-port.sh`, a forensic verifier I wrote that statically checks whether a binary's hot path is genuinely vectorized, rather than silently falling back to scalar. It caught an incorrect figure in my own published results before I shipped them, which is exactly why it exists.
 
---
 
## Machine learning: [Pretrained-CNN](https://github.com/trg-rgb/Pretrained-CNN-with-6-class-image-classification)
 
6-class CNN for groundnut leaf disease detection. **Gold Medal, First Place at Sci Quest 2025, MIT-WPU**, against students from all branches and years. Trained, optimized for inference, and deployed live. **[Try the demo](https://huggingface.co/spaces/tanmaytrg/Pretrained_CNN_6_class_image_classification)**
 
The INT8-quantized version of this model is what later ran inference on RISC-V via TF Lite, which is how the ML work and the systems work connect.
 
---
 
## Tech
 
![C](https://img.shields.io/badge/C-00599C?style=flat&logo=c&logoColor=white)
![C++](https://img.shields.io/badge/C++-00599C?style=flat&logo=cplusplus&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![GCC](https://img.shields.io/badge/GCC-A42E2B?style=flat&logo=gnu&logoColor=white)
![QEMU](https://img.shields.io/badge/QEMU-FF6600?style=flat&logo=qemu&logoColor=white)
![CMake](https://img.shields.io/badge/CMake-064F8C?style=flat&logo=cmake&logoColor=white)
![TensorFlow](https://img.shields.io/badge/TensorFlow-FF6F00?style=flat&logo=tensorflow&logoColor=white)
![RISC-V](https://img.shields.io/badge/RISC--V-283272?style=flat)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat&logo=linux&logoColor=black)
 
---
 
<div align="center">
B.Tech 2025–2029 · open to internships in RISC-V, systems software, and toolchains
 
</div>

