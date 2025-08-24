# 🚀 Boosting CPU Performance with ARM SVE & SME
_Vector Length Agnostic Programming & Matrix Acceleration for Everyone_ 🧑‍💻⚡

Welcome to Boosting Performance with ARM SVE/SME — a beginner‑friendly guide & codebase for learning how to vectorize your CPU code and unlock serious performance gains in AI, HPC, and scientific computing workloads.

Whether you're new to SIMD or just starting with ARM’s Scalable Vector Extension (SVE) and Scalable Matrix Extension (SME), this repo is here to help you:

- 📜 Understand vectorization from basics → advanced
- 💻 Write code that automatically adapts to your CPU’s vector length
- 🏎️ Boost performance without rewriting for each new CPU width
- 🧠 Accelerate matrix operations for AI & ML workloads

📚 Table of Contents
1. 💡 [Why Vectorization?](#why)
2. 🕰 [A Short History of SIMD](#history)
3. 📏 [The Problem with Fixed-Width SIMD](#problem)
4. 🌱 [ARM SVE — Vector Length Agnostic Magic](#sve)
5. 🧮 [ARM SME — Matrix Acceleration in Hardware](#sme)
6. 🛠 [Getting Started: Your First SVE/SME Code](#start)
9. 🖥️ [Running on Simulators / Emulators / QEMU](#qemu)
8. 📖 [References & Resources](#ref)

<a name="why"></a>
## 💡 Why Vectorization?
Traditional CPUs work on one piece of data at a time (scalar).
Vector processors work on many at once 🏋️‍♂️ — Single Instruction, Multiple Data (SIMD).

Benefits of vectorization:

- ⚡ Faster execution on repetitive, data-heavy loops
- 🔋 Lower power use by doing more work per instruction
- 💾 Better memory efficiency

```C++
// Scalar (Old Way)
for (int i = 0; i < 4; i++) {
    C[i] = A[i] + B[i];
}

// Vectorized (SIMD Way — all at once)
C[0..3] = A[0..3] + B[0..3]; // in one instruction!
```
<a name="history"></a>
## 🕰 A Short History of SIMD
- MMX (1997) — Integer-only SIMD for multimedia
- SSE → AVX → AVX-512 — Wider vectors, floats, fused multiply-add (FMA)
- NEON (ARM) — SIMD for mobile revolution 📱
- SVE (ARM) — The scalable future: works with any vector length
- SME (ARM) — Dedicated CPU hardware for matrices, inspired by GPUs

<a name="problem"></a>
## 📏 The Problem with Fixed-Width SIMD
Regular SIMD locks you into a fixed size — e.g., AVX2 = 256 bits.
If a new CPU has 512-bit vectors, your old AVX2 code 🔒 won’t get faster without a rewrite / recompile.

<a name="sve"></a>
## 🌱 ARM SVE — Vector Length Agnostic Magic
Write once, run everywhere — SVE works for any vector length from 128 → 2048 bits ✨.
Portable code, no need to recompile, vector length determined @ runtime 😊

Example (_auto‑scales to CPU width_):

```c++
void add_arrays_sve(float *C, float *A, float *B, long n) {
    long i = 0;
    svbool_t pg;
    while (pg = svwhilelt_b32(i, n), svptest_first(pg)) {
        svfloat32_t vecA = svld1_f32(pg, &A[i]);
        svfloat32_t vecB = svld1_f32(pg, &B[i]);
        svfloat32_t vecC = svadd_f32_x(pg, vecA, vecB);
        svst1_f32(pg, &C[i], vecC);
        i += svcntw(); // auto increments by CPU vector length
    }
}
```
✅ Future-proof
✅ No manual rewrite for wider CPUs

<a name="sme"></a>
## 🧮 ARM SME — Matrix Acceleration in Hardware
SME builds on SVE2 to supercharge matrix multiplication:

📌 ZA tiles — massive on‑chip matrix registers
📌 Streaming SVE mode (SSVE) — throughput-optimized
📌 Outer Product Instruction (FMOPA) — multiply rows & columns directly

Conceptual assembly:

```bash
SMSTART ZA                // Enable streaming mode
SMLOAD ZA0.S, [A]         // Load tile A
SMLOAD ZA1.S, [B]         // Load tile B
SMMLA ZA2.S, ZA0.S, ZA1.S // ZA2 += ZA0 * ZA1 (outer product)
SMSTORE [C], ZA2.S        // Store result tile
SMSTOP ZA                 // Exit
```

<a name="start"></a>
# 🛠 Getting Started: Your First SVE/SME Code
1. Learn SIMD basics — loops, data alignment, vector loads/stores
2. Start with SVE intrinsics — avoid fixed widths (svadd_f32_x, svld1_f32, etc.)
3. Move to SME for heavy matrix math:
    - Learn ZA registers and tile slices
    - Understand outer products
    - Use Streaming SVE mode
4. Try examples from this repo — e.g., vector add, matmul, tile-based FMOPA
5. Experiment with BF16 for AI — less memory, same range as FP32

📢 Disclaimer
 Hardware performance depends on CPU support — make sure to check your compiler & hardware capabilities before running the examples. You can check hte Makefile of this [repository](https://github.com/mhbiniaz/llama2.c-arm-sve-sme) (llama.cpp with sve/sme support)

<a name="qemu"></a>
## 🖥️ Running on Simulators / Emulators / QEMU
Not everyone has access to a physical ARMv9 CPU with SVE and SME support — but you can still learn, write, and test your code using CPU simulators & emulators 🧪.

🛠 Why use a simulator?
✅ Develop & debug before hardware is available
✅ Test different vector lengths without buying multiple CPUs
✅ Trace & analyze instruction flow for learning and optimization
✅ Great for unit tests in CI/CD without ARM hardware

Be careful that QEMU is not cycle accurate, you can try gem5 instead. But QEMU gives instruction count accurately. You can download a container for qemu + all the instruction couting plugins here [x86](https://hub.docker.com/repository/docker/hbiniaz/sme_x86_qemu/general) [aarch64](https://hub.docker.com/repository/docker/hbiniaz/native_arm/general)