# Inter-Base Conversion Pivot Benchmark (SRCC-Decision)

This repository contains the official C source code, synthetic and realistic workloads, and statistical validation scripts for the paper:  
**"The Choice of Pivot in Inter-Base Conversion: An Empirical Study on the Cost of Parse and Format"** (Submitted to *Software: Practice and Experience*).

## 📌 Project Overview

When developing heterogeneous systems, distributed architectures, or microservices, software engineers frequently face serialization dilemmas. If a system must ingest and emit mixed string representations (e.g., JSON decimals, IPv6 hexadecimal addresses, Unix octal file permissions, and Base 32 TOTP tokens), an architectural question arises: **What fixed canonical textual format (pivot) should be adopted internally to achieve maximum throughput?**

While classical textbook models suggest that conversion costs are dictated by mathematical prime factorization (implying that binary is the universal pivot), this project provides a rigorous empirical refutation. Our benchmarks demonstrate that conversion costs are strictly dominated by **digit-stream loop volume** on modern CPU microarchitectures (`x86-64`). Consequently, power-of-two bases with high bit-density per digit (such as **Base 32** and **Hexadecimal**) consistently outperform traditional decimal and binary pivots.

## 🚀 Key Experimental Findings

Our full 64-bit integer execution-level calibration (`u64` values) on modern environments shows a stark contrast between classical theory and empirical reality:

| Pivot Base | Character Count (per u64) | Mean Parse Time | Mean Format Time | Total Latency | Performance Relative to Base 32 |
| :---: | :---: | :---: | :---: | :---: | :---: |
| **Base 32 (b32)** | **13** | **15.91 ns** | **15.41 ns** | **31.32 ns** | **1.0x (Optimal)** |
| **Hexadecimal (hex)** | 16 | 20.83 ns | 22.46 ns | 43.29 ns | 1.38x Slower |
| **Octal (oct)** | 22 | 17.98 ns | 28.85 ns | 46.82 ns | 1.49x Slower |
| **Decimal (dec)** | 20 | 17.03 ns | 38.78 ns | 55.81 ns | 1.78x Slower |
| **Binary (bin)** | 64 | 46.39 ns | 59.25 ns | 105.64 ns | **3.37x Slower (Worst)** |

*   **The Power-of-Two Superiority:** Power-of-two bases with $n \ge 3$ bits per digit bypass heavy division/multiplication arithmetic loops (`%`, `/`, `*`) by leveraging microarchitectural bitwise masks and shifts (`&`, `>>`, `<<`).
*   **The Digit Volume Fallacy:** Binary uses fast bitwise operations but processes 64 loops. Base 32 processes only 13 loops. Thus, Base 32 yields up to a **44% latency reduction** over the standard decimal pivot and a **3.4x speedup** over raw binary.

## 📂 Repository Structure

*   `calibrate.c`: Base calibration harness used to measure isolated string parsing and formatting overheads.
*   `crossvalidate.c`: Synthetic operational testing script modeling raw `A -> P -> B` pipeline transactions.
*   `validate_final.c`: Multi-magnitude statistical harness (Small, Medium, Large ranges with standard deviation aggregation).
*   `sanity_workload.c`: Real-world scenario simulation modeling JSON API payloads, XML configurations, and production system logs.

## 🛠️ Reproduction and Compilation Guide

To replicate our microarchitectural results on any Linux-based machine (such as Ubuntu), follow the steps below.

### Prerequisites

Ensure you have GCC and build essentials installed:
```bash
sudo apt update
sudo apt install build-essential -y
```

### Execution Steps

1. **Compile and Run Multi-Magnitude Statistics:**
   The compiler flag `-march=native` is required to allow GCC to leverage specific microarchitectural optimizations on the host pipeline.
   ```bash
   gcc -O2 -march=native -o validate_final validate_final.c -lm
   ./validate_final
   ```

2. **Compile and Run Production Workload Simulations:**
   This execution simulates high-volume transaction routing for active application pipelines.
   ```bash
   gcc -O2 -march=native -o sanity sanity_workload.c
   ./sanity
   ```

## 📜 Citations and Metadata

This code repository belongs to an ongoing peer-review track at Wiley's *Software: Practice and Experience*. 

```bibtex
% Main data reference placeholder
@article{brandao2026pivot,
  author  = {Brand{\~a}o Junior, Luiz Carlos and Magalh{\~a}es, Ricardo Rodrigues},
  title   = {The Choice of Pivot in Inter-Base Conversion: An Empirical Study on the Cost of Parse and Format},
  journal = {Software: Practice and Experience},
  year    = {2026},
  note    = {Under Review}
}
```

## 🔒 License and Reproducibility

This project is built under a completely open framework for academic audit. The generated pseudo-random values use fixed logical seeds (`srand(42)`) to ensure that execution-step datasets remain identical across local host systems, guaranteeing absolute statistical transparency.
