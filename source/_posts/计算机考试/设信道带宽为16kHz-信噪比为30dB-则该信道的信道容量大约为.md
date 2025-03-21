---
title: 设信道带宽为16kHz,信噪比为30dB,则该信道的信道容量大约为
date: 2024-10-22 17:19:02
categories: 计算机考试
tag: 自学公式
---
根据香农信道容量公式：

\[
C = B \log_2(1 + S/N)
\]

其中：
- \( C \) 是信道容量（单位：比特每秒，bps），
- \( B \) 是信道带宽（单位：赫兹，Hz），
- \( S/N \) 是信噪比，通常以线性值表示。

### 已知条件：
- 信道带宽 \( B = 16 \, \text{kHz} = 16,000 \, \text{Hz} \),
- 信噪比 \( \text{S/N} = 30 \, \text{dB} \).

首先，将信噪比从 dB 转换为线性值：
\[
S/N = 10^{30/10} = 10^3 = 1000
\]

将这些值代入香农公式：
\[
C = 16000 \log_2(1 + 1000)
\]

计算出 \( \log_2(1001) \)，然后再乘以带宽 \( B \)。

我们来计算这个值。

根据计算结果，信道容量大约为 **159,476 bps**（比特每秒）。