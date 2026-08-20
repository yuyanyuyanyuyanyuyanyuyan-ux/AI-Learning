# GPU Server Interconnect

> 本文件讲解 GPU 服务器内部组件之间，以及 GPU 服务器与外部服务器之间的通信方式。
>
> 重点包括：
>
> * CPU 与 GPU 的通信
> * GPU 与 GPU 的通信
> * PCIe 的连接元件和作用
> * NVLink 的作用
> * NVSwitch 的作用
> * NIC 网卡的作用
> * RDMA 的作用
> * 单台 GPU 服务器与 GPU 集群之间的数据通信

---

# 1. GPU 服务器通信系统概览

GPU 服务器不仅包含 CPU、GPU、内存和存储设备，还包含一套高速通信系统。

这些组件之间需要不断交换数据。

一台 GPU 服务器可以简化为：

```text
                         GPU Server
┌─────────────────────────────────────────────────────┐
│                                                     │
│                 ┌──────────────┐                    │
│                 │     CPU      │                    │
│                 └──────┬───────┘                    │
│                        │                            │
│                       PCIe                          │
│                        │                            │
│        ┌───────────────┼───────────────┐            │
│        │               │               │            │
│        ▼               ▼               ▼            │
│      GPU 0           GPU 1           GPU 2 ...      │
│        │               │               │            │
│        └────── NVLink / NVSwitch ─────┘             │
│                                                     │
│                        │                            │
│                       PCIe                          │
│                        │                            │
│                        ▼                            │
│                      NIC                            │
└────────────────────────┬────────────────────────────┘
                         │
                         │ Ethernet / InfiniBand
                         │
                         ▼
                   Data Center Network
                         │
                         ▼
                    Other Servers
```

因此，GPU 服务器中的通信可以分成三个层次：

```text
第一层：CPU ↔ GPU

主要使用：

PCIe


第二层：GPU ↔ GPU

主要使用：

NVLink
NVSwitch


第三层：Server ↔ Server

主要使用：

NIC
+
Ethernet / InfiniBand
+
RDMA
```

---

# 2. GPU 服务器中的通信层次

从整体上看：

```text
CPU
 │
 │ PCIe
 ▼
GPU
 │
 │ NVLink
 ▼
Other GPU
 │
 │ NIC + RDMA
 ▼
Other Server
```

可以进一步理解为：

```text
┌──────────────────────────────┐
│        Server 1              │
│                              │
│ CPU                          │
│  │                           │
│ PCIe                         │
│  │                           │
│ GPU 0 ←── NVLink ──→ GPU 1   │
│  │                    │      │
│  └─────── NIC ────────┘      │
└───────────────┬──────────────┘
                │
                │ RDMA Network
                │
┌───────────────▼──────────────┐
│        Server 2              │
│                              │
│ GPU 0 ←── NVLink ──→ GPU 1   │
│                              │
└──────────────────────────────┘
```

---

# 3. CPU 与 GPU 如何通信

CPU 和 GPU 是两个独立的处理器。

CPU 有自己的：

```text
CPU
 │
 ▼
System Memory
```

GPU 有自己的：

```text
GPU
 │
 ▼
HBM
```

因此：

```text
CPU Memory ≠ GPU HBM
```

CPU 和 GPU 之间需要专门的数据传输通道。

最常见的方式是：

> PCIe。

---

# 4. PCIe 是什么

PCIe 的全称是：

> Peripheral Component Interconnect Express。

中文通常称为：

> 高速外设组件互连。

PCIe 是现代服务器和计算机中非常重要的高速互连总线。

例如：

```text
CPU
 │
 ├──── PCIe ──── GPU
 │
 ├──── PCIe ──── NIC
 │
 ├──── PCIe ──── SSD
 │
 └──── PCIe ──── Other Devices
```

因此 PCIe 可以理解为：

> **服务器内部用于连接 CPU 和各种高速设备的高速数据通道。**

---

# 5. PCIe 连接哪些元件

在 GPU 服务器中，PCIe 可以连接：

```text
CPU
 │
 ▼
PCIe
 │
 ├── GPU
 │
 ├── NIC
 │
 ├── NVMe SSD
 │
 ├── Storage Controller
 │
 └── Other Expansion Devices
```

例如：

```text
                     CPU
                      │
              ┌───────┴───────┐
              │               │
            PCIe             PCIe
              │               │
              ▼               ▼
             GPU             NIC
```

因此 PCIe 是 GPU 服务器内部的重要基础通信结构。

---

# 6. PCIe 的连接元件

一个典型的 PCIe 通信路径可以理解为：

```text
CPU
 │
 ▼
PCIe Root Complex
 │
 ▼
PCIe Bus
 │
 ├──────── GPU
 │
 ├──────── NIC
 │
 └──────── NVMe SSD
```

其中 CPU 一侧通常存在 PCIe Root Complex。

可以简单理解为：

```text
CPU
 │
 ▼
PCIe Root Complex
```

它负责：

```text
管理 PCIe 设备

↓

建立 CPU 与 PCIe 设备之间的通信
```

如果 PCIe 设备数量很多，还可能存在：

```text
PCIe Switch
```

结构：

```text
CPU
 │
 ▼
PCIe Switch
 │
 ├── GPU 0
 ├── GPU 1
 ├── GPU 2
 └── GPU 3
```

因此：

```text
CPU
 ↓
PCIe Root Complex
 ↓
PCIe Switch
 ↓
GPU / NIC / SSD
```

---

# 7. PCIe 的作用

PCIe 的主要作用是：

> 在服务器内部提供高速数据传输通道。

例如：

```text
CPU
 │
 │ 数据
 ▼
PCIe
 │
 ▼
GPU
```

AI 训练中的数据可能经历：

```text
Storage
 │
 ▼
CPU Memory
 │
 ▼
PCIe
 │
 ▼
GPU HBM
```

CPU 将需要计算的数据传输给 GPU。

GPU 完成计算后：

```text
GPU
 │
 ▼
PCIe
 │
 ▼
CPU
```

也可以将结果返回给 CPU。

---

# 8. PCIe Lane

PCIe 可以理解为由多条数据通道组成。

这些通道通常称为：

> PCIe Lane。

例如：

```text
PCIe x1

[ Lane ]
```

```text
PCIe x4

[ Lane ][ Lane ][ Lane ][ Lane ]
```

```text
PCIe x16

[ Lane ][ Lane ][ Lane ] ... ×16
```

GPU 通常使用较宽的 PCIe 连接，例如：

```text
GPU
 │
 PCIe x16
 │
CPU
```

可以简单理解：

```text
更多 Lane

↓

更宽的数据通道

↓

更高的数据传输能力
```

因此：

```text
PCIe x1

类似：

1 条高速车道
```

而：

```text
PCIe x16

类似：

多条高速车道同时传输数据
```

---

# 9. 为什么 PCIe 对 GPU 很重要

GPU 本身虽然负责大量计算，但 GPU 不是孤立工作的。

GPU 需要：

```text
从 CPU 接收任务
```

需要：

```text
从系统读取数据
```

还需要：

```text
与 NIC 等设备协同工作
```

因此：

```text
CPU
 │
 │ PCIe
 ▼
GPU
```

是 GPU 服务器最基本的通信链路之一。

但是 PCIe 并不是 GPU 与 GPU 之间最高效的通信方式。

对于大规模 GPU 计算，还需要：

> NVLink。

---

# 10. GPU 与 GPU 的通信

假设服务器中有：

```text
GPU 0
GPU 1
GPU 2
GPU 3
```

这些 GPU 可能共同训练一个 AI 模型。

例如：

```text
GPU 0 → 计算一部分数据

GPU 1 → 计算一部分数据

GPU 2 → 计算一部分数据

GPU 3 → 计算一部分数据
```

训练完成一个步骤后，它们可能需要交换：

```text
Gradient
Activation
Model Data
```

如果 GPU 之间的数据通信路径为：

```text
GPU 0
 │
 PCIe
 │
CPU
 │
PCIe
 │
GPU 1
```

那么数据可能需要经过 CPU 和 PCIe 系统。

这会增加：

```text
通信路径

+

延迟

+

CPU / PCIe 压力
```

因此 GPU 之间需要更直接、更高速的连接。

这就是：

> NVLink。

---

# 11. NVLink 是什么

NVLink 是 GPU 之间的高速互连技术。

可以理解为：

```text
GPU 0
   ║
   ║ NVLink
   ║
GPU 1
```

相比：

```text
GPU 0
   │
  PCIe
   │
CPU / PCIe System
   │
  PCIe
   │
GPU 1
```

NVLink 提供了 GPU 之间更直接的高速通信方式。

因此：

```text
PCIe：

通用高速设备互联


NVLink：

主要用于高性能 GPU 之间的高速互联
```

---

# 12. NVLink 的作用

NVLink 的主要作用包括：

```text
GPU ↔ GPU 高速数据传输

↓

降低 GPU 之间通信延迟

↓

提高多 GPU 协同计算效率
```

例如：

```text
GPU 0
 │
 │ 计算结果
 ▼
NVLink
 │
 ▼
GPU 1
```

在 AI 训练中：

```text
GPU 0 ─┐
GPU 1 ─┼──→ 交换 Gradient
GPU 2 ─┤
GPU 3 ─┘
```

NVLink 可以帮助 GPU 更高效地交换这些数据。

---

# 13. NVLink 的基本通信结构

多个 GPU 可以通过 NVLink 进行连接。

简单情况：

```text
GPU 0 ═════════ GPU 1
```

更复杂：

```text
GPU 0 ═══ GPU 1
  ║           ║
  ║           ║
GPU 2 ═══ GPU 3
```

但是当 GPU 数量增加时：

```text
GPU 0
GPU 1
GPU 2
GPU 3
GPU 4
GPU 5
GPU 6
GPU 7
```

如果每个 GPU 都直接连接所有其他 GPU：

```text
连接数量会迅速增加
```

这时需要：

> NVSwitch。

---

# 14. NVSwitch 是什么

NVSwitch 可以理解为：

> 专门用于连接多个 GPU 的高速交换设备。

结构类似：

```text
GPU 0 ──┐
GPU 1 ──┤
GPU 2 ──┤
GPU 3 ──┼── NVSwitch
GPU 4 ──┤
GPU 5 ──┤
GPU 6 ──┤
GPU 7 ──┘
```

GPU 之间不一定需要全部直接互相连接。

而是：

```text
GPU
 │
 ▼
NVLink
 │
 ▼
NVSwitch
 │
 ▼
NVLink
 │
 ▼
Other GPU
```

---

# 15. NVSwitch 的作用

NVSwitch 的主要作用是：

> **扩展多个 GPU 之间的高速通信能力。**

例如：

```text
GPU 0
   │
   ▼
NVSwitch
   │
   ▼
GPU 7
```

或者：

```text
GPU 2
   │
   ▼
NVSwitch
   │
   ▼
GPU 5
```

可以理解为：

```text
NVLink

=

GPU 之间的高速链路


NVSwitch

=

管理和交换多个 GPU 数据的高速交换中心
```

---

# 16. NVLink 和 NVSwitch 的关系

两者的关系可以理解为：

```text
GPU
 │
 │ NVLink
 ▼
NVSwitch
 │
 │ NVLink
 ▼
GPU
```

因此：

```text
NVLink：

负责提供 GPU 与 GPU 之间的高速连接


NVSwitch：

负责将多个 GPU 通过 NVLink 组织成高速互联网络
```

例如一个多 GPU 系统：

```text
GPU 0 ─┐
GPU 1 ─┤
GPU 2 ─┤
GPU 3 ─┼──── NVSwitch
GPU 4 ─┤
GPU 5 ─┤
GPU 6 ─┤
GPU 7 ─┘
```

这样多个 GPU 可以更高效地进行数据交换。

---

# 17. GPU 服务器内部通信总结

单台 GPU 服务器内部：

```text
                CPU
                 │
                 │ PCIe
                 │
       ┌─────────┼─────────┐
       ▼         ▼         ▼
     GPU 0     GPU 1     GPU 2
       │         │         │
       └──── NVLink ──────┘
                 │
              NVSwitch
```

不同技术负责不同任务：

| 通信技术     | 主要作用                    |
| -------- | ----------------------- |
| PCIe     | CPU 与 GPU、NIC、SSD 等设备通信 |
| NVLink   | GPU 与 GPU 高速通信          |
| NVSwitch | 组织多个 GPU 的高速互联          |
| NIC      | 服务器与外部网络通信              |
| RDMA     | 降低跨服务器数据传输的 CPU 参与和延迟   |

---

# 18. NIC：服务器的网络接口

NIC 是：

> Network Interface Card。

中文通常称为：

> 网卡。

NIC 的主要作用是：

> **让服务器能够连接到数据中心网络，并与其他服务器交换数据。**

结构：

```text
GPU Server
    │
    ▼
   NIC
    │
    ▼
Network
    │
    ▼
Other Server
```

例如：

```text
Server 1
   │
   ▼
 NIC
   │
   ▼
Switch
   │
   ▼
 NIC
   │
Server 2
```

---

# 19. NIC 在 GPU 集群中的作用

GPU 集群由多台服务器组成：

```text
Server 1
├── GPU × 8

Server 2
├── GPU × 8

Server 3
├── GPU × 8

Server 4
├── GPU × 8
```

这些服务器需要通过网络通信。

因此：

```text
GPU
 │
 ▼
NIC
 │
 ▼
Data Center Network
 │
 ▼
NIC
 │
 ▼
Other GPU Server
```

NIC 就是服务器进入数据中心网络的入口。

---

# 20. 服务器与外界的通信方式

服务器不仅需要与 GPU 集群通信，还可能与：

```text
其他服务器

存储系统

交换机

用户请求

管理系统
```

通信路径通常是：

```text
Server
 │
 ▼
NIC
 │
 ▼
Switch
 │
 ▼
Data Center Network
 │
 ├── Other GPU Server
 │
 ├── Storage Server
 │
 └── External Network
```

例如：

```text
                 ┌──────────────┐
                 │   Switch     │
                 └──────┬───────┘
                        │
       ┌────────────────┼───────────────┐
       │                │               │
       ▼                ▼               ▼
   Server 1         Server 2        Storage
       │                │
      NIC              NIC
```

---

# 21. 为什么普通网络通信不够

传统网络通信中，数据可能经历：

```text
Application
    │
    ▼
CPU
    │
    ▼
System Memory
    │
    ▼
NIC
    │
    ▼
Network
```

接收端：

```text
NIC
 │
 ▼
System Memory
 │
 ▼
CPU
 │
 ▼
Application
```

因此数据传输过程中：

```text
CPU 需要参与
```

同时数据可能需要：

```text
多次复制
```

例如：

```text
GPU Data

↓

CPU Memory

↓

NIC

↓

Network
```

对于 AI 分布式训练：

```text
大量 GPU

+

大量 Gradient

+

频繁通信
```

这种方式可能产生：

```text
CPU 负担

+

数据复制

+

更高延迟
```

因此需要：

> RDMA。

---

# 22. RDMA 是什么

RDMA 是：

> Remote Direct Memory Access。

中文通常称为：

> 远程直接内存访问。

核心思想是：

> **允许一台服务器直接访问另一台服务器的内存，而减少远端 CPU 对数据传输过程的参与。**

传统通信：

```text
Server A

CPU
 │
 ▼
Memory
 │
 ▼
NIC
 │
 ▼
Network
 │
 ▼
NIC
 │
 ▼
Memory
 │
 ▼
CPU

Server B
```

RDMA：

```text
Server A

Memory / Device
      │
      ▼
     NIC
      │
      ▼
   Network
      │
      ▼
     NIC
      │
      ▼
Memory / Device

Server B
```

核心目标是减少：

```text
CPU 参与

+

数据复制

+

通信延迟
```

---

# 23. RDMA 的作用

RDMA 的主要作用包括：

```text
减少 CPU 参与

↓

减少数据复制

↓

降低通信延迟

↓

提高数据传输效率
```

因此非常适合：

```text
AI Distributed Training

HPC

Large GPU Cluster

High Performance Storage
```

---

# 24. GPU 与 GPU 跨服务器通信

现在考虑两台 GPU 服务器。

```text
Server 1

GPU 0
GPU 1
GPU 2
GPU 3
```

```text
Server 2

GPU 0
GPU 1
GPU 2
GPU 3
```

如果：

```text
Server 1 GPU 0
```

需要和：

```text
Server 2 GPU 2
```

交换数据。

简化路径：

```text
GPU
 │
 ▼
PCIe
 │
 ▼
NIC
 │
 ▼
Network
 │
 ▼
NIC
 │
 ▼
PCIe
 │
 ▼
GPU
```

如果使用支持 RDMA 的网络技术，可以减少传统通信过程中 CPU 的参与。

概念上可以理解为：

```text
GPU 0
 │
 ▼
NIC
 │
 ═══════ RDMA Network ═══════
 │
 ▼
NIC
 │
 ▼
GPU 1
```

这使得：

```text
GPU Server 1

GPU
 │
NIC
 │
════════════════════

GPU Server 2

NIC
 │
GPU
```

形成更高效的跨服务器 GPU 通信。

---

# 25. GPU Direct RDMA 的概念

在 GPU 集群中，还有一个非常重要的思想：

> 尽可能减少 GPU 数据在 CPU 内存中的中转。

传统概念：

```text
GPU

↓

CPU Memory

↓

NIC

↓

Network
```

更高效的方式：

```text
GPU

↓

NIC

↓

Network
```

接收端：

```text
Network

↓

NIC

↓

GPU
```

这样可以减少：

```text
GPU → CPU Memory → NIC
```

中的额外数据中转。

因此可以提高 GPU 集群通信效率。

---

# 26. 从单 GPU 到 GPU 集群

单个 GPU：

```text
CPU
 │
PCIe
 │
GPU
```

多个 GPU：

```text
GPU 0 ═ NVLink ═ GPU 1
GPU 2 ═ NVLink ═ GPU 3
```

大量 GPU：

```text
GPU
 │
NVLink
 │
NVSwitch
 │
NVLink
 │
GPU
```

多个服务器：

```text
Server 1
 │
NIC
 │
══════════ Network ══════════
 │
NIC
 │
Server 2
```

大型 GPU 集群：

```text
GPU
 │
NVLink
 │
NVSwitch
 │
GPU Server
 │
NIC
 │
RDMA Network
 │
NIC
 │
Other GPU Server
```

---

# 27. AI 训练中的完整通信过程

假设一个大型 AI 模型运行在：

```text
4 台服务器

每台服务器 8 张 GPU
```

总共有：

```text
4 × 8 = 32 GPUs
```

通信层次：

```text
第 1 层：

GPU 内部计算


第 2 层：

同一服务器 GPU 通信

GPU
 ↓
NVLink
 ↓
GPU


第 3 层：

多 GPU 高速交换

GPU
 ↓
NVLink
 ↓
NVSwitch
 ↓
NVLink
 ↓
GPU


第 4 层：

跨服务器通信

GPU
 ↓
PCIe / 高速 I/O
 ↓
NIC
 ↓
RDMA Network
 ↓
NIC
 ↓
GPU
```

---

# 28. AI 分布式训练中的通信

假设：

```text
GPU 0
GPU 1
GPU 2
GPU 3
```

每个 GPU 计算：

```text
Gradient
```

然后需要同步：

```text
GPU 0 ─┐
GPU 1 ─┼──── Gradient Exchange
GPU 2 ─┤
GPU 3 ─┘
```

如果所有 GPU 都在同一服务器：

```text
GPU
 ↓
NVLink
 ↓
NVSwitch
 ↓
GPU
```

如果 GPU 位于不同服务器：

```text
GPU
 ↓
NIC
 ↓
RDMA Network
 ↓
NIC
 ↓
GPU
```

因此大型 AI 训练不仅仅依赖：

```text
GPU 性能
```

还依赖：

```text
GPU Interconnect Performance
```

即：

> GPU 之间的数据通信能力。

---

# 29. PCIe、NVLink、NVSwitch、NIC 和 RDMA 的关系

可以将它们放在同一张结构图中理解：

```text
                         GPU SERVER
┌───────────────────────────────────────────────────────┐
│                                                       │
│                        CPU                            │
│                         │                             │
│                        PCIe                            │
│                         │                             │
│          ┌──────────────┼──────────────┐              │
│          ▼              ▼              ▼              │
│        GPU 0          GPU 1          GPU 2             │
│          │              │              │              │
│          └────── NVLink / NVSwitch ───┘              │
│                         │                             │
│                        PCIe                            │
│                         │                             │
│                        NIC                            │
└─────────────────────────┬─────────────────────────────┘
                          │
                          │ Ethernet / InfiniBand
                          │
                          │ RDMA
                          ▼
                    Data Center Network
                          │
                          ▼
                    Other GPU Server
```

---

# 30. 各通信技术的职责

| 技术          | 主要通信对象                | 核心作用                  |
| ----------- | --------------------- | --------------------- |
| PCIe        | CPU ↔ GPU / NIC / SSD | 服务器内部高速设备互联           |
| PCIe Switch | 多个 PCIe 设备            | 扩展 PCIe 连接能力          |
| NVLink      | GPU ↔ GPU             | GPU 之间高速通信            |
| NVSwitch    | 多 GPU                 | 构建大规模 GPU 高速交换网络      |
| NIC         | Server ↔ Network      | 服务器接入数据中心网络           |
| RDMA        | Server ↔ Server       | 减少 CPU 参与，提高低延迟数据传输效率 |

---

# 31. 一个完整的 GPU 集群通信结构

```text
                     GPU SERVER 1
┌────────────────────────────────────────────┐
│                                            │
│                    CPU                     │
│                     │                      │
│                    PCIe                    │
│                     │                      │
│         ┌───────────┼───────────┐          │
│         ▼           ▼           ▼          │
│       GPU 0       GPU 1       GPU 2        │
│         │           │           │          │
│         └──── NVLink / NVSwitch ────┘     │
│                     │                      │
│                    NIC                     │
└─────────────────────┬──────────────────────┘
                      │
                      │ RDMA Network
                      │
                      ▼
┌────────────────────────────────────────────┐
│                    NIC                     │
│                     │                      │
│         ┌───────────┼───────────┐          │
│         ▼           ▼           ▼          │
│       GPU 0       GPU 1       GPU 2        │
│         │           │           │          │
│         └──── NVLink / NVSwitch ────┘     │
│                     │                      │
│                    CPU                     │
│                                            │
│                 GPU SERVER 2               │
└────────────────────────────────────────────┘
```

---

# 32. 最终通信链路总结

整个 GPU 服务器通信体系可以总结为：

```text
【CPU 与 GPU】

CPU
 │
 ▼
PCIe
 │
 ▼
GPU
```

```text
【GPU 与 GPU】

GPU
 │
 ▼
NVLink
 │
▼
GPU
```

```text
【多个 GPU】

GPU
 │
 ▼
NVLink
 │
 ▼
NVSwitch
 │
 ▼
NVLink
 │
 ▼
GPU
```

```text
【服务器与服务器】

Server
 │
 ▼
NIC
 │
 ▼
Data Center Network
 │
 ▼
NIC
 │
 ▼
Other Server
```

```text
【跨服务器高性能通信】

GPU
 │
 ▼
NIC
 │
 ═══════ RDMA ═══════
 │
 ▼
NIC
 │
 ▼
GPU
```

---

# 33. 最终知识结构

```text
GPU Server Interconnect
│
├── CPU ↔ GPU
│   │
│   └── PCIe
│       ├── PCIe Root Complex
│       ├── PCIe Switch
│       └── PCIe Lanes
│
├── GPU ↔ GPU
│   │
│   ├── PCIe
│   │
│   └── NVLink
│
├── Multi-GPU Interconnect
│   │
│   └── NVSwitch
│
└── Server ↔ Server
    │
    ├── NIC
    │
    ├── Ethernet / InfiniBand
    │
    └── RDMA
        │
        └── High Performance GPU Communication
```

---

# 34. 核心总结

GPU 服务器中的通信体系可以理解为三个层次：

## 第一层：CPU 与设备通信

```text
CPU
 │
 ▼
PCIe
 │
 ├── GPU
 ├── NIC
 └── SSD
```

PCIe 是服务器内部的基础高速互联。

---

## 第二层：GPU 与 GPU 通信

```text
GPU
 │
NVLink
 │
GPU
```

多个 GPU：

```text
GPU
 │
NVLink
 │
NVSwitch
 │
NVLink
 │
GPU
```

NVLink 负责高速 GPU 通信，NVSwitch 用于扩展多个 GPU 之间的高速交换能力。

---

## 第三层：服务器之间通信

```text
GPU Server
 │
NIC
 │
Network
 │
NIC
 │
Other GPU Server
```

通过 RDMA 等技术，可以减少 CPU 参与和数据复制，提高跨服务器数据传输效率。

---

# 35. 一句话理解整个 GPU 集群通信系统

> **CPU 通过 PCIe 连接 GPU、NIC 和其他高速设备；同一台服务器中的多个 GPU 通过 NVLink 和 NVSwitch 进行高速通信；服务器通过 NIC 接入数据中心网络；多个 GPU 服务器之间利用高速网络和 RDMA 实现低延迟、高带宽的数据交换，最终组成大规模 GPU 计算集群。**
