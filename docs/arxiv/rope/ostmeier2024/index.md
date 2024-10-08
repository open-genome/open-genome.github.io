---
layout: default
parent: Positional Embedding
grand_parent: ArXiv
title: "LieRE: Generalizing Rotary Position Encodings"
nav_order: 3
discuss: true
math: katex
---

# LieRE: Generalizing Rotary Position Encodings

Paper link: [https://arxiv.org/abs/2406.10322v1](https://arxiv.org/abs/2406.10322v1)

在自然语言处理和计算机视觉领域，transformer 模型的注意力机制已成为主流。然而，传统的绝对位置编码和相对位置编码（如 RoPE）在高维数据上的表现并不理想。本文将探讨一种新的位置编码方法——Lie 群相对位置编码（LieRE），它能够在不改变模型架构的情况下处理多种数据模态和维度。通过利用 Lie 群和 Lie 代数的数学特性，LieRE 提供了一种通用且高效的编码方案。

## 1. 引言

随着 transformer 模型在各类任务中的广泛应用，如何在注意力机制中有效地编码位置信息成为一个重要的研究方向。传统的旋转位置编码（RoPE）在处理一维数据（如文本序列）时表现优异，但在更高维的数据（如 2D 图像和 3D 数据）上应用较少，且存在性能瓶颈。为了解决这个问题，我们引入了 Lie 群相对位置编码（LieRE），一种通用的编码方法，可以支持多维输入数据，并显著提升模型性能。

## 2. 相关工作

在注意力机制的研究中，位置编码可分为三类：绝对位置编码、相对位置编码和上下文位置编码。传统的绝对位置编码方法通常在输入序列的每个 token 上加入位置信息，这种方法对于捕捉全局模式的能力有限。而相对位置编码，如 RoPE，通过利用旋转矩阵编码 token 之间的相对位置，能够更好地捕捉局部依赖关系。然而，RoPE 主要设计用于一维数据，无法自然扩展到高维数据上。LieRE 通过引入 Lie 群和 Lie 代数的数学结构，克服了这些限制，使得模型能够同时处理局部和全局依赖关系。

## 3. 数学背景

### 3.1 Lie 群和 Lie 代数

**Lie 群** 是在矩阵乘法和逆运算下闭合的光滑流形。例如，特殊正交群 $SO(n)$ 包含了所有 $n \times n$ 的正交矩阵，其行列式为 1。正交矩阵可以描述旋转变换，因此在高维空间的旋转中有重要应用。

**Lie 代数** 是与 Lie 群紧密相关的一个概念，表示 Lie 群在单位元附近的无穷小生成元。Lie 代数的元素是反对称矩阵（即 $A^T = -A$），这些矩阵通过矩阵指数 $\exp(A)$ 可以生成 Lie 群的元素。

#### 反对称矩阵的性质

- 一个反对称矩阵 $A$ 满足 $A^T = -A$，这意味着它的对角线元素为零，且非对角线元素成对出现为相反数。
- 反对称矩阵的特征值是纯虚数，成对出现。对于 $n \times n$ 的反对称矩阵 $A$，其特征值为 $\pm i\lambda_j$（其中 $i$ 是虚数单位，$\lambda_j \geq 0$）。
- 矩阵指数 $\exp(A)$ 生成了一个正交矩阵，特别是一个特殊正交矩阵，即行列式为 1。

### 3.2 矩阵指数与旋转

矩阵指数 $\exp(A)$ 定义为幂级数的形式：
$$
\exp(A) = I + A + \frac{A^2}{2!} + \frac{A^3}{3!} + \cdots
$$
其中 $I$ 是单位矩阵。

对于反对称矩阵 $A$，矩阵指数 $\exp(A)$ 生成了一个正交矩阵。正交矩阵保留向量的长度和角度，因此在旋转群中扮演着关键角色。

## 4. 方法：LieRE 的提出

LieRE 的核心思想是通过学习一个反对称矩阵 $A$ 来生成旋转矩阵 $R = \exp(A)$，并将其应用于 transformer 模型的注意力机制中。在这种方法中，每个 token 的位置信息首先被嵌入为一个反对称矩阵 $P_i$ 和 $P_j$，然后通过矩阵指数生成旋转矩阵 $R_i = \exp(P_i)$ 和 $R_j = \exp(P_j)$。

### 4.1 LieRE 的数学实现

在 LieRE 中，注意力计算公式被修改为：
$$
\text{Attention} = \text{softmax}\left(\frac{(R_i \cdot Q)(R_j \cdot K)^T}{\sqrt{d}}\right) V
$$
其中：
- $Q$ 和 $K$ 是查询和键向量。
- $R_i = \exp(P_i)$ 和 $R_j = \exp(P_j)$ 是基于位置生成的旋转矩阵。

#### 旋转矩阵的构造

设 $x_i \in \mathbb{R}^d$ 是第 $i$ 个 token 的位置信息，通过一个线性变换 $A: \mathbb{R}^d \to \text{Skew}(d)$ 将其映射为一个反对称矩阵：

$$
P_i = A(x_i)
$$

然后计算旋转矩阵：

$$
R_i = \exp(P_i)
$$

旋转矩阵 $R_i$ 被用来调整查询和键向量的方向，使得注意力机制不仅能够捕捉局部信息，也能适应全局依赖。

## 5. 实验与分析

### 5.1 实验设计

为了验证 LieRE 的有效性，我们在多个数据集上进行了实验，包括 CIFAR100、ImageNet、UCF101 和 RSNA 数据集。这些实验测试了 LieRE 在不同模态和维度数据上的表现。

### 5.2 实验结果

实验结果显示，LieRE 在多个任务上表现优于其他位置编码方法。尤其是在高维数据上（如 3D 医学图像和视频数据），LieRE 显著提高了模型的准确率，同时减少了训练时间。

## 6. 反对称矩阵的谱分析

### 6.1 特征值分析

反对称矩阵 $A$ 的特征值和特征向量提供了关于注意力机制如何处理长短程依赖的深刻洞见。

#### 特征值的性质

对于反对称矩阵 $A$，其特征值是纯虚数 $\pm i\lambda_j$（$\lambda_j \geq 0$），这些特征值描述了旋转的角度和方向：

- **特征值的模（$|\lambda_j|$）与旋转角度的关系**：
 $$ 
  R_i = \exp(A) = I + A + \frac{A^2}{2!} + \cdots
 $$ 
  特征值的模越大，表示旋转的角度越大。对于长程注意力，特征值的模通常较大，这意味着模型在较大的范围内关注 token 之间的关系。

- **谱的结构与注意力模式**：
 $$ 
  A \mathbf{v} = \lambda \mathbf{v}
 $$ 
  通过分析特征值的分布，可以推断出模型是更关注局部依赖还是全局依赖。较大的特征值模表示对长程依赖的关注，较小的特征值模表示对短程依赖的关注。

### 6.2 奇异值分析

奇异值分解（SVD）可以用来进一步理解反对称矩阵的性质。

对于一个反对称矩阵 $A$，其奇异值分解形式为：

$$
A = U \Sigma V^H
$$

其中：
- $U$ 和 $V$ 是正交矩阵，
- $\Sigma$ 是对角矩阵，其对角元素是奇异值。

#### 奇异值的物理意义

- **奇异值的大小**：奇异值描述了矩阵在特定方向上的扩展或收缩。较大的奇异值表示强烈的旋转效果，适合捕捉全局模式；较小的奇异值表示轻微的旋转，适合捕捉局部模式。

### 6.3 对旋转行为的解释

- **局部旋转与局部注意力**：
 $$ 
  R_i \approx I + A
 $$ 
  当特征值较小（旋转角度较小）时，旋转矩阵近似于单位矩阵加上小的调整项，主要关注局部关系。

- **全局旋转与全局注意力**：
  当特征值较大（旋转角度较大）时，旋转矩阵进行更大的变换，能够在新的空间中将相隔较远的 token 映射到相似的方向上，从而实现全局依赖。

## 7. 结论

LieRE 提供了一种新的位置编码方法，通过利用 Lie 群和 Lie 代数的数学结构，在多模态和多维度数据上表现出色。通过分析反对称矩阵的谱，我们进一步理解了模型如何在长短程依赖中进行权衡。未来的工作可以继续探索 LieRE 在其他领域的应用，以及优化反对称矩阵的学习方法。