# GPU Server Components

> 本文件讲解 GPU 服务器的主要组成元件及其作用，重点包括 CPU、GPU、GPU 内部的 SM（Streaming Multiprocessor，流式多处理器）、并行计算资源以及 HBM 高带宽显存。
>
> GPU 服务器是 AI 训练和 AI 推理基础设施中的核心计算节点。它通过 CPU 负责控制和调度，通过 GPU 执行大规模并行计算，通过高速显存存储模型和计算数据，并通过高速网络与其他服务器组成 GPU 集群。

---

# 1. GPU 服务器整体组成

一台典型的 GPU 服务器可以抽象为：

```text
┌──────────────────────────────────────────────┐
│                 GPU Server                   │
│                                              │
│  ┌──────────┐                ┌──────────┐    │
│  │   CPU 0  │                │   CPU 1  │    │
│  └────┬─────┘                └────┬─────┘    │
│       │                           │          │
│       └───────────┬───────────────┘          │
│                   │                          │
│               PCIe/高速互联                   │
│                   │                          │
│      ┌────────────┼────────────┐             │
│      │            │            │             │
│   ┌──▼───┐     ┌──▼───┐     ┌──▼───┐         │
│   │ GPU0 │     │ GPU1 │ ... │ GPU7 │         │
│   └──────┘     └──────┘     └──────┘         │
│                                              │
│  ┌────────────────────────────────────────┐  │
│  │              System Memory             │  │
│  │                  DRAM                  │  │
│  └────────────────────────────────────────┘  │
│                                              │
│  ┌────────────────────────────────────────┐  │
│  │           SSD / Storage                │  │
│  └────────────────────────────────────────┘  │
│                                              │
│  ┌────────────────────────────────────────┐  │
│  │          High-Speed NIC                │  │
│  │       200G / 400G / 800G Network       │  │
│  └────────────────────────────────────────┘  │
└──────────────────────────────────────────────┘
```

GPU 服务器主要由以下部分组成：

```text
GPU Server
│
├── CPU
│
├── System Memory
│   └── DDR / DRAM
│
├── GPU
│   │
│   ├── SM
│   │
│   ├── CUDA Cores / Computing Units
│   │
│   ├── Tensor Cores
│   │
│   ├── Cache
│   │
│   └── Memory Controller
│
├── GPU Memory
│   └── HBM
│
├── PCIe / NVLink 等高速互联
│
├── SSD / Storage
│
└── High-Speed NIC
```

---

# 2. CPU：GPU 服务器的控制和调度中心

CPU 是：

> Central Processing Unit，中央处理器。

在 GPU 服务器中，CPU 并不主要负责执行大规模 AI 矩阵计算，而是负责整个服务器的控制、调度和通用计算任务。

可以理解为：

```text
CPU
│
├── 运行操作系统
├── 运行应用程序
├── 数据预处理
├── 管理内存
├── 调度 GPU
├── 管理网络通信
└── 协调多个 GPU
```

在 AI 训练过程中，一个简化的数据流如下：

```text
训练数据
    │
    ▼
CPU
    │
    ├── 数据读取
    ├── 数据预处理
    └── 任务调度
    │
    ▼
GPU
    │
    ▼
大规模矩阵计算
```

因此可以简单理解：

```text
CPU = 管理者 + 调度者

GPU = 大规模计算执行者
```

---

# 3. CPU 和 GPU 的区别

CPU 和 GPU 的设计目标不同。

## CPU

CPU 的特点：

```text
核心数量相对较少
↓
单个核心能力较强
↓
擅长复杂逻辑
↓
擅长任务控制和程序调度
```

适合：

* 操作系统
* 程序控制
* 逻辑判断
* 数据处理
* 任务调度
* GPU 管理

---

## GPU

GPU 的特点：

```text
大量计算单元
↓
可以同时执行大量相似计算
↓
适合并行计算
```

尤其适合：

* 矩阵乘法
* 向量计算
* 张量计算
* 神经网络计算

可以简单理解为：

```text
CPU：

少量能力很强的工程师
负责处理复杂问题


GPU：

大量计算单元
同时处理大量相似任务
```

例如：

```text
计算 100 万个数据
```

CPU 可以依次或者分批处理：

```text
CPU
 │
 ├── 任务 1
 ├── 任务 2
 ├── 任务 3
 └── ...
```

GPU 可以将大量任务分配给不同的并行计算单元：

```text
GPU

计算单元 1  ──→ 数据 1
计算单元 2  ──→ 数据 2
计算单元 3  ──→ 数据 3
计算单元 4  ──→ 数据 4
...
计算单元 N  ──→ 数据 N
```

这就是 GPU 适合 AI 的根本原因之一。

---

# 4. GPU：大规模并行计算处理器

GPU 是：

> Graphics Processing Unit，图形处理器。

虽然 GPU 最初主要用于图形计算，但现代 GPU 已经成为 AI 训练和推理的核心计算设备。

一个 GPU 可以抽象为：

```text
┌────────────────────────────────────┐
│                 GPU                │
│                                    │
│  ┌──────┐  ┌──────┐  ┌──────┐      │
│  │ SM 0 │  │ SM 1 │  │ SM 2 │ ...  │
│  └──────┘  └──────┘  └──────┘      │
│                                    │
│  ┌──────────────────────────────┐  │
│  │       Cache / Memory         │  │
│  └──────────────────────────────┘  │
│                                    │
│  ┌──────────────────────────────┐  │
│  │     Memory Controller        │  │
│  └──────────────┬───────────────┘  │
└─────────────────┼──────────────────┘
                  │
                  ▼
               HBM Memory
```

GPU 内部最重要的并行计算组织单元之一是：

> SM（Streaming Multiprocessor）。

---

# 5. SM：GPU 的核心计算模块

SM 是：

> Streaming Multiprocessor，流式多处理器。

可以把一个 GPU 理解为由大量 SM 组成。

```text
GPU
│
├── SM 0
├── SM 1
├── SM 2
├── SM 3
├── ...
└── SM N
```

例如：

```text
┌───────────────────────────────┐
│             GPU               │
│                               │
│  ┌─────┐ ┌─────┐ ┌─────┐      │
│  │ SM0 │ │ SM1 │ │ SM2 │ ...  │
│  └─────┘ └─────┘ └─────┘      │
│                               │
│  ┌─────┐ ┌─────┐ ┌─────┐      │
│  │ SM3 │ │ SM4 │ │ SM5 │ ...  │
│  └─────┘ └─────┘ └─────┘      │
└───────────────────────────────┘
```

因此：

```text
一个 GPU
=
大量 SM
```

而：

```text
多个 SM
=
大量并行计算资源
```

---

# 6. SM 内部有什么

SM 本身也不是一个简单的计算单元。

可以进一步理解为：

```text
SM
│
├── 控制和调度单元
│
├── 通用计算单元
│
├── Tensor Core 等专用计算单元
│
├── Register
│
├── Shared Memory
│
└── Cache
```

简化结构：

```text
┌─────────────────────────────┐
│             SM              │
│                             │
│  ┌───────────────────────┐  │
│  │ Scheduler             │  │
│  │ 任务调度器             │  │
│  └───────────┬───────────┘  │
│              │              │
│     ┌────────┼────────┐     │
│     ▼        ▼        ▼     │
│ Compute   Compute   Tensor  │
│ Units     Units     Units   │
│                             │
│  Register / Shared Memory   │
└─────────────────────────────┘
```

因此 GPU 的计算层次可以理解为：

```text
GPU
│
├── SM 0
│   ├── 计算资源
│   ├── Tensor 计算资源
│   ├── Register
│   └── Shared Memory
│
├── SM 1
│
├── SM 2
│
└── SM N
```

---

# 7. SM 的作用

SM 的主要作用是：

> 接收 GPU 程序中的大量并行任务，并将这些任务分配给内部的计算资源执行。

可以理解为：

```text
AI Program
    │
    ▼
GPU
    │
    ▼
大量并行任务
    │
    ├─────────────┐
    ▼             ▼
   SM 0          SM 1
    │             │
    ▼             ▼
执行任务       执行任务
```

如果有大量 SM：

```text
SM 0  → 执行任务
SM 1  → 执行任务
SM 2  → 执行任务
SM 3  → 执行任务
...
SM N  → 执行任务
```

那么 GPU 就可以同时执行大量计算。

因此：

> SM 的数量和每个 SM 内部的计算资源，是决定 GPU 并行计算能力的重要因素。

---

# 8. GPU 的并行计算资源

GPU 最核心的特点是：

> Parallel Computing，并行计算。

AI 中经常需要进行：

```text
矩阵 × 矩阵
```

例如：

```text
A × B = C
```

假设矩阵：

```text
A = 10000 × 10000

B = 10000 × 10000
```

需要计算大量元素。

GPU 可以将计算任务拆分：

```text
矩阵计算
│
├── 计算元素 1
├── 计算元素 2
├── 计算元素 3
├── ...
└── 计算元素 N
```

然后：

```text
GPU
│
├── SM 0  → 一部分计算
├── SM 1  → 一部分计算
├── SM 2  → 一部分计算
└── SM N  → 一部分计算
```

SM 内部继续将任务分配给更细粒度的计算资源。

因此形成：

```text
一个大型计算任务

        ↓

GPU

        ↓

多个 SM

        ↓

大量并行计算单元

        ↓

同时执行
```

这就是 GPU 大规模并行计算的基本思想。

---

# 9. CUDA Core 和通用计算资源

在 NVIDIA GPU 的概念体系中，常见的计算资源包括：

> CUDA Core。

可以简单理解为：

```text
GPU
│
├── SM
│   │
│   ├── CUDA Core
│   ├── CUDA Core
│   ├── CUDA Core
│   └── ...
│
├── SM
│
└── ...
```

这些计算资源主要负责大量通用并行计算。

例如：

```text
数据

1
2
3
4
5
6
...
```

可以分配：

```text
CUDA Core 1 → 数据 1

CUDA Core 2 → 数据 2

CUDA Core 3 → 数据 3

CUDA Core 4 → 数据 4
```

大量计算任务同时进行。

因此可以理解：

```text
GPU
    ↓
SM
    ↓
大量计算资源
    ↓
并行执行
```

---

# 10. Tensor Core：AI 专用计算资源

现代 AI GPU 中还有非常重要的一类计算资源：

> Tensor Core。

AI 模型中最常见的计算之一是：

```text
Matrix Multiplication
```

例如：

```text
[A] × [B] = [C]
```

神经网络中的：

```text
输入数据
    ↓
权重矩阵
    ↓
矩阵乘法
    ↓
输出
```

会反复出现。

Tensor Core 是针对这类计算进行优化的专用硬件资源。

可以简单理解：

```text
普通计算资源：

适合通用计算


Tensor Core：

专门优化矩阵和张量计算
```

因此：

```text
AI Training
      │
      ▼
大量 Matrix Multiplication
      │
      ▼
Tensor Core
      │
      ▼
提高计算吞吐量
```

在现代 AI 训练和推理中，Tensor Core 是 GPU 性能的重要组成部分。

---

# 11. 从程序到 SM：任务是如何执行的

一个 GPU 程序不会直接告诉某一个计算核心：

```text
你计算这个数据。
```

而是会将任务组织成大量并行线程。

可以抽象为：

```text
AI Program
    │
    ▼
大量 Threads
    │
    ▼
GPU Scheduler
    │
    ▼
多个 SM
```

例如：

```text
100 万个线程

        │
        ▼

GPU

        │

 ┌──────┼──────┐
 ▼      ▼      ▼

SM0    SM1    SM2
 │      │      │
 ▼      ▼      ▼

大量线程并行执行
```

因此：

```text
Program
↓
Thread
↓
GPU
↓
SM
↓
计算资源
```

这就是 GPU 并行计算从软件任务到硬件资源的基本映射过程。

---

# 12. HBM：GPU 的高带宽显存

GPU 不仅需要强大的计算能力，还需要高速的数据供应能力。

GPU 使用的高性能显存通常包括：

> HBM，High Bandwidth Memory，高带宽存储器。

可以理解为：

```text
GPU
 │
 ▼
HBM
```

HBM 主要用于存储：

```text
模型参数
训练数据
中间计算结果
梯度
KV Cache
```

在 AI 场景中：

```text
GPU

┌─────────────────────────────┐
│                             │
│       计算                   │
│                             │
│   SM / Tensor Core           │
│                             │
└──────────────┬──────────────┘
               │
               ▼
          HBM Memory
```

---

# 13. 为什么 AI 需要 HBM

假设 GPU 计算能力非常强：

```text
GPU：

每秒可以完成大量计算
```

但是如果数据读取速度不够：

```text
GPU

等待数据
   │
   ▼

计算资源空闲
```

那么 GPU 的计算能力无法充分发挥。

因此 AI GPU 需要：

```text
高计算能力
+
高内存带宽
```

HBM 的核心优势之一就是：

> 提供非常高的数据传输带宽。

可以理解为：

```text
普通内存：

数据管道较窄


HBM：

数据管道更宽
```

因此：

```text
更多数据

同时传输

↓

GPU

↓

持续进行计算
```

---

# 14. HBM 和系统内存的区别

GPU 服务器通常同时存在：

```text
System Memory
+
GPU Memory
```

例如：

```text
┌───────────────┐
│      CPU      │
└───────┬───────┘
        │
        ▼
┌───────────────┐
│ DDR / DRAM    │
│ 系统内存       │
└───────────────┘
```

GPU：

```text
┌───────────────┐
│      GPU      │
└───────┬───────┘
        │
        ▼
┌───────────────┐
│      HBM      │
│ GPU 高带宽显存 │
└───────────────┘
```

两者主要区别可以简单理解为：

| 类型         | 主要服务对象 | 主要作用            |
| ---------- | ------ | --------------- |
| DDR / DRAM | CPU    | 存储 CPU 运行的数据和程序 |
| HBM        | GPU    | 存储模型和 GPU 计算数据  |

在 AI 训练过程中：

```text
SSD / Storage
      │
      ▼
CPU Memory
      │
      ▼
GPU HBM
      │
      ▼
GPU Computing
```

---

# 15. HBM 容量为什么重要

假设一个模型需要：

```text
模型参数 = 120 GB
```

如果 GPU HBM：

```text
80 GB
```

那么：

```text
120 GB > 80 GB
```

单个 GPU 无法完整存储该模型。

这时可以：

```text
GPU 0
HBM：模型的一部分

GPU 1
HBM：模型的一部分
```

形成：

```text
Model

      ↓

┌──────────────┐
│ GPU 0        │
│ Model Part A │
└──────────────┘

┌──────────────┐
│ GPU 1        │
│ Model Part B │
└──────────────┘
```

因此：

> HBM 容量决定了单个 GPU 能够容纳多大的模型和中间计算数据。

---

# 16. AI 训练中的显存占用

AI 训练时，HBM 中通常不仅存储模型参数。

还可能包括：

```text
GPU HBM
│
├── Model Parameters
│
├── Input Data
│
├── Activations
│
├── Gradients
│
├── Optimizer States
│
└── Temporary Buffers
```

因此：

```text
训练模型所需要的显存
>
单纯的模型参数大小
```

这也是为什么 AI 训练通常比推理需要更多显存资源。

---

# 17. AI 推理中的显存占用

推理阶段通常不需要：

```text
反向传播
梯度计算
参数更新
```

因此显存主要用于：

```text
GPU HBM
│
├── Model Parameters
│
├── Input
│
├── Intermediate Results
│
└── KV Cache
```

对于大语言模型，KV Cache 是重要的显存消耗来源之一。

可以理解为：

```text
用户输入
    │
    ▼
模型计算
    │
    ▼
产生中间结果
    │
    ▼
KV Cache
```

模型生成下一个 Token 时：

```text
不需要重新计算所有历史内容

↓

直接使用缓存结果
```

从而提高推理效率。

---

# 18. GPU 内部计算资源与 HBM 的关系

GPU 的性能不能只看计算单元数量。

完整的计算过程是：

```text
HBM
 │
 │ 提供数据
 ▼
GPU
 │
 ├── SM
 │
 ├── CUDA / 通用计算资源
 │
 └── Tensor Core
 │
 ▼
计算结果
```

如果：

```text
计算资源很强

但是

HBM 数据供应不足
```

那么：

```text
GPU

等待数据

↓

计算资源利用率下降
```

反过来：

```text
HBM 很快

但是

GPU 计算能力不足
```

那么计算速度仍然有限。

因此 GPU 性能可以简单理解为：

```text
GPU Performance

=

计算能力

+

内存容量

+

内存带宽

+

数据传输能力
```

---

# 19. 多 GPU 服务器

AI 服务器通常不只有一块 GPU。

例如：

```text
GPU Server
│
├── GPU 0
├── GPU 1
├── GPU 2
├── GPU 3
├── GPU 4
├── GPU 5
├── GPU 6
└── GPU 7
```

形成：

```text
CPU
 │
 ├──────── GPU 0
 ├──────── GPU 1
 ├──────── GPU 2
 ├──────── GPU 3
 ├──────── GPU 4
 ├──────── GPU 5
 ├──────── GPU 6
 └──────── GPU 7
```

多个 GPU 可以共同执行一个大型 AI 任务。

---

# 20. GPU 之间的高速互联

多个 GPU 需要交换数据。

例如：

```text
GPU 0

计算结果
   │
   ▼

GPU 1
```

常见的互联方式包括：

```text
PCIe
```

以及 GPU 专用高速互联：

```text
NVLink 等
```

可以简单理解：

```text
GPU 0
   │
   │ 高速互联
   ▼
GPU 1
```

在多 GPU AI 训练中，GPU 之间需要频繁交换：

```text
Gradient
Activation
Model Data
```

因此 GPU 之间的通信能力会直接影响分布式计算效率。

---

# 21. GPU 服务器中的完整数据流

以 AI 模型训练为例：

```text
Storage
   │
   ▼
CPU
   │
   │ 数据读取和预处理
   ▼
System Memory
   │
   ▼
GPU
   │
   ▼
HBM
   │
   ▼
SM
   │
   ├── 通用计算资源
   │
   └── Tensor Core
   │
   ▼
计算结果
```

进一步展开：

```text
训练数据

    │

    ▼

SSD / Distributed Storage

    │

    ▼

CPU

    │
    │ 数据加载
    │ 数据预处理
    │ GPU 调度
    │
    ▼

System Memory

    │

    ▼

GPU HBM

    │

    ▼

GPU SM

    │
    ├── Parallel Computing
    │
    └── Tensor Computing
    │
    ▼

Forward Propagation

    │

    ▼

Loss

    │

    ▼

Backward Propagation

    │

    ▼

Gradient

    │

    ▼

Update Model Parameters
```

---

# 22. GPU 服务器的组件关系

整个 GPU 服务器可以总结为：

```text
                         GPU SERVER
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
       CPU                System Memory          Storage
        │                     │                     │
        │                     │                     │
        └───────────┬─────────┴─────────┬───────────┘
                    │                   │
                    ▼                   ▼
                 PCIe / 高速互联       NIC
                    │                   │
                    ▼                   ▼
              ┌────────────┐       Data Center
              │    GPU     │       Network
              │            │
              │ SM × N     │
              │            │
              │ ├─ Compute │
              │ ├─ Tensor  │
              │ ├─ Cache   │
              │ └─ Shared  │
              └──────┬─────┘
                     │
                     ▼
                    HBM
```

---

# 23. 从硬件角度理解 AI 训练

整个 AI 训练实际上可以看成：

```text
数据

↓

CPU 负责调度

↓

数据进入 GPU HBM

↓

GPU 将计算任务分配给多个 SM

↓

SM 内部的计算资源并行执行

↓

Tensor Core 等专用计算资源执行大量矩阵运算

↓

得到计算结果

↓

HBM 存储中间数据

↓

重复执行

↓

模型参数不断更新
```

因此：

> AI 训练本质上是在大量 GPU 硬件资源上不断执行大规模并行数学计算。

---

# 24. 从硬件角度理解 AI 推理

AI 推理过程则可以理解为：

```text
用户请求

↓

CPU 接收请求

↓

推理程序进行调度

↓

模型加载到 GPU HBM

↓

输入数据进入 GPU

↓

SM 执行并行计算

↓

Tensor Core 等资源执行矩阵计算

↓

模型生成结果

↓

CPU / 推理服务返回结果

↓

用户
```

与训练不同的是：

```text
训练：

Forward
+
Backward
+
Parameter Update
```

推理：

```text
Forward Only
```

即：

> 使用已经训练完成的模型进行前向计算。

---

# 25. 核心组件总结

| 组件            | 主要作用                     |
| ------------- | ------------------------ |
| CPU           | 运行操作系统、任务调度、数据处理和 GPU 管理 |
| System Memory | 存储 CPU 正在处理的数据           |
| GPU           | 执行大规模并行计算                |
| SM            | GPU 内部的重要并行计算模块          |
| 通用计算资源        | 执行大量通用并行计算               |
| Tensor Core   | 加速 AI 中的大规模矩阵和张量计算       |
| HBM           | 为 GPU 提供高容量、高带宽的数据存储     |
| PCIe          | CPU 与 GPU、设备之间的数据传输      |
| GPU 高速互联      | 多 GPU 之间高速交换数据           |
| SSD / Storage | 存储数据集、模型和检查点             |
| NIC           | 与其他服务器和数据中心网络通信          |

---

# 26. 最终知识结构

```text
GPU Server
│
├── CPU
│   ├── Operating System
│   ├── Task Scheduling
│   ├── Data Processing
│   └── GPU Management
│
├── System Memory
│   └── CPU Working Data
│
├── GPU
│   │
│   ├── SM
│   │   │
│   │   ├── Scheduler
│   │   ├── Parallel Computing Resources
│   │   ├── General Compute Units
│   │   ├── Tensor Computing Resources
│   │   ├── Registers
│   │   └── Shared Memory
│   │
│   ├── Cache
│   │
│   └── Memory Controller
│
├── HBM
│   ├── Model Parameters
│   ├── Input Data
│   ├── Activations
│   ├── Gradients
│   └── KV Cache
│
├── PCIe / High-Speed Interconnect
│
├── SSD / Storage
│
└── High-Speed NIC
```

---

# 27. 总结

一台 GPU 服务器可以理解为一个分层的计算系统：

```text
CPU
│
│ 负责控制、调度和管理
│
▼

GPU
│
│ 负责大规模并行计算
│
▼

SM
│
│ GPU 内部的主要并行计算模块
│
▼

Parallel Computing Resources
│
│ 同时执行大量计算任务
│
▼

HBM
│
│ 为 GPU 提供模型和计算数据
│
▼

AI Training / AI Inference
```

整个核心逻辑可以总结为：

> **CPU 负责管理和调度，GPU 负责大规模计算；GPU 由多个 SM 构成，SM 内部包含大量并行计算资源和 AI 专用计算资源；HBM 为 GPU 提供高带宽的数据和模型存储能力。多个 GPU 通过高速互联组成更强大的计算节点，而多个 GPU 服务器进一步通过高速网络组成 AI 计算集群。**

```text
CPU
 ↓
任务调度
 ↓
GPU
 ↓
多个 SM
 ↓
大量并行计算资源
 ↓
HBM 提供高速数据
 ↓
矩阵 / 张量计算
 ↓
AI 模型训练或推理
```
