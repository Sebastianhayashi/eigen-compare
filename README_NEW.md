# Eigen RISC-V Vector (RVV) Acceleration Benchmark

##  项目简介 (Introduction)

本项目旨在探究并验证非官方 **RISC-V Vector (RVV) Patch** 对 **Eigen** 线性代数库在嵌入式 RISC-V 硬件上的实际性能影响。

**背景**：在尝试将 ROS 移植到 RISC-V 平台的过程中，我们在社区发现了针对 Eigen 的 RVV 加速 Patch。
**目的**：为了评估该 Patch 是否能为嵌入式开发板（如 Lichee Pi 4A）带来实质性的算力提升，我们设计了一套严格控制变量的 **A/B 基准测试（Benchmark）**。

我们不仅关注“跑分”，更关注在不同计算负载（In-Cache vs Out-of-Cache）下的性能边界，为后续的 ROS 算法优化提供数据支撑。

>  **[项目背景与立项动因](https://www.google.com/search?q=doc/background.md)**

---

##  核心结论 (Key Findings)

通过对比官方版（Vanilla Eigen）与加速版（Eigen + RVV Patch），我们发现在当前的硬件与编译器环境下，性能提升呈现出明显的**负载相关性**：

| 测试场景 | 性能趋势 | 现象分析 |
| --- | --- | --- |
| **高负载矩阵运算** <br>

<br> (GEMM, TRMM) | 🚀 **显著提升** | **+4% 至 +18%**。RVV 指令集有效提升了稠密计算任务的吞吐量，并行优势抵消了指令开销。 |
| **矩阵分解** <br>

<br> (Cholesky, LU) | 📈 **中度提升** | 在 Out-of-Cache 场景下表现更佳，表明 RVV 在处理访存密集型的大规模数据时具有一定优势。 |
| **轻量级/向量运算** <br>

<br> (Vector Add, Rank-1 Update) | 🔻 **持平或回退** | **-2% 至 -16%**。对于简单的小规模运算，RVV 指令的 Setup/Teardown 开销超过了其带来的收益，或存在内存访问模式未优化的问题。 |

>  **[数据分析报告与图表](https://www.google.com/search?q=doc/analysis_report.md)** > *(包含 Bianbu OS 与 openEuler 环境下的详细对比数据与可视化图表)*

---

## 🛠️ 实验设计与方法 (Methodology)

为了确保数据的客观性与可复现性，我们并未直接运行 Patch，而是设计了一套严格的**控制变量实验（Controlled Experiment）**：

1. **A/B 测试架构**：构建两套完全相同的测试套件，唯一变量为“是否应用 Patch 并开启 RVV 编译选项”。
2. **严格的环境控制**：
* 统一使用 GCC Toolchain，锁定 `-O3` 与架构参数。
* 在测试期间锁定 CPU 频率（Performance Governor），排除动态调频干扰。


3. **多维度指标**：
* **In-Cache**：测试 CPU 流水线与向量单元的极限吞吐。
* **Out-of-Cache**：测试在内存带宽瓶颈下的实际表现。



>  **[实验设计方案](https://www.google.com/search?q=doc/experimental_design.md)** > *(包含详细的硬件参数、编译 Flags 配置以及测试用例选择逻辑)*

---

## 📂 仓库结构 (Repository Structure)

| 目录 | 说明 |
| --- | --- |
| **[`doc/`](https://www.google.com/search?q=doc/)** | **核心文档区**：包含背景、实验设计细节及详细分析报告。 |
| **[`benchmark/`](https://www.google.com/search?q=benchmark/)** | C++ 基准测试源代码。 |
| **[`scripts/`](https://www.google.com/search?q=scripts/)** | 自动化脚本（包含环境构建、批量测试脚本）。 |
| **[`data/`](https://www.google.com/search?q=data/)** | 原始测试日志与处理后的 CSV 数据。 |
| **[`patches/`](https://www.google.com/search?q=patches/)** | 本次实验所评估的非官方 RVV Patch 文件。 |

---

## 🚀 如何复现 (Reproduction)

如果你拥有支持 RVV 0.7.1 或 1.0 的 RISC-V 开发板（如 Lichee Pi 4A, VisionFive 2），可以通过以下步骤复现测试：

```bash
# 1. 克隆仓库
git clone https://github.com/your-username/eigen-rvv-benchmark.git
cd eigen-rvv-benchmark

# 2. 自动构建 (脚本会自动编译 "Origin" 和 "Patch" 两个版本)
./scripts/build.sh

# 3. 运行基准测试
./scripts/run_tests.sh

```

