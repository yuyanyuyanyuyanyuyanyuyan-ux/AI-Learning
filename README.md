# Data Center and AI Learning Knowledge Base

> 从数据中心硬件到 AI 模型训练与推理的个人技术学习知识库。

## 学习路线

```text
数据中心基础
        ↓
GPU 服务器
        ↓
数据中心网络
        ↓
AI 模型基础
        ↓
模型训练
        ↓
分布式训练
        ↓
模型推理
        ↓
Jupyter Notebook 实战项目
```

---

# 目录

## 01 Data Center

学习数据中心和智算中心的基础组成。

内容包括：

* 数据中心整体结构
* 智算中心硬件组成
* 计算、存储和网络
* 数据中心整体运行流程

---

## 02 GPU Server

学习 GPU 服务器的硬件组成和计算原理。

内容包括：

* CPU
* GPU
* 内存
* 存储
* 网卡
* PCIe
* GPU 之间的数据通信

核心问题：

> 一台 GPU 服务器内部的数据是如何流动的？

---

## 03 Data Center Network

学习数据中心网络结构。

内容包括：

* 交换机
* 路由器
* 局域网
* 数据中心网络
* Spine-Leaf 网络
* 数据传输流程

核心问题：

> 多台 GPU 服务器之间如何高速通信？

---

## 04 AI Model

学习 AI 模型的基本原理。

内容包括：

* 什么是模型
* 神经网络
* 训练数据
* 参数
* 损失函数
* 反向传播
* GPU 为什么适合 AI 训练

核心流程：

```text
训练数据
    ↓
输入模型
    ↓
模型计算
    ↓
得到预测结果
    ↓
计算损失
    ↓
反向传播
    ↓
更新参数
    ↓
重复训练
```

---

## 05 AI Training System

学习 AI 模型如何在 GPU 服务器上训练。

内容包括：

* 单 GPU 训练
* 多 GPU 训练
* 分布式训练
* 数据并行
* 模型训练完整流程

---

## 06 AI Inference

学习模型训练完成之后如何提供服务。

内容包括：

* 模型推理
* 推理框架
* 请求处理
* 模型加载
* GPU 推理
* 返回结果

核心流程：

```text
用户请求
    ↓
推理服务器
    ↓
模型
    ↓
GPU
    ↓
生成结果
    ↓
返回用户
```

---

## 07 Jupyter Notebook

使用 Jupyter Notebook 学习和实践：

* Python
* NumPy
* PyTorch
* 神经网络
* 模型训练

---

## 08 Mini Project

最终使用 Jupyter Notebook 完成一个小型 AI 模型训练项目。

项目流程：

```text
准备数据
    ↓
读取数据集
    ↓
构建神经网络
    ↓
定义损失函数
    ↓
训练模型
    ↓
观察 Loss
    ↓
测试模型
```

---

# 学习目标

通过这个知识库建立完整的认知链条：

```text
数据中心

↓ 提供

GPU 计算资源

↓ 通过

数据中心网络

↓ 组成

大规模 AI 训练集群

↓ 用于

训练 AI 模型

↓ 得到

训练完成的模型

↓ 部署到

推理服务器

↓ 最终提供

AI 服务
```

---

# 技术栈

* GitHub
* Markdown
* Python
* Jupyter Notebook
* NumPy
* PyTorch

---

# 学习状态

* [x] 数据中心硬件基础
* [x] GPU 服务器基本组成
* [x] 交换机与路由器基础
* [x] AI 模型训练基本流程
* [x] Jupyter Notebook 基础认识
* [ ] 完成第一个小型 AI 训练项目
* [ ] 学习 Git 本地版本控制
* [ ] 学习多人协作和 Pull Request

---

# Repository Structure

```text
datacenter-ai-learning
│
├── 01-datacenter
├── 02-gpu-server
├── 03-datacenter-network
├── 04-ai-model
├── 05-ai-training-system
├── 06-ai-inference
├── 07-jupyter-notebook
├── 08-mini-project
├── images
│
└── README.md
```

This repository will continue to grow as I learn more about data centers, AI infrastructure, model training and AI systems.
