---
title: >-
  Few-Shot Relation Learning with Attention for EEG-based Motor Imagery
  Classification
date: 2022-09-14 10:46:13
categories: "MI"
tags:
  - eeg
  - few-shot

mathjax: true
---

**《基于运动想象分类的注意力小样本学习》**

<!-- more -->

## **Abstract**

本文提出了一种 `two-way few-shot` 网络，该网络能够有效地学习 MI 特征并分类，其主要结构包括：

- 特征表示嵌入模块
- 注意力模块
- 基于 `support set`和 `query set` 的关系网络。

## **METHOD**

<center>![图1. 2-way K-shot learning framework](/imgs/Few-Shot-Relation-Learning-with-Attention-for-EEG-based-Motor-Imagery-Classification/network_structure.png)</center>
  如图1，本文的框架由三个模块组成：嵌入模块、注意模块和关系模块。

$A.Embedding Module$  
本文作者基于[HS-CNN](https://iopscience.iop.org/article/10.1088/1741-2552/ab405f/meta)网络进行修改，使用四阶巴特沃斯带通滤波器将数据分成 3 个不同的频带（对应图 2.），并使用不同尺度的 `1D` 卷积提取每个频带的特征，详见图 2。

<center>![图2 嵌入模块的具体结构](/imgs/Few-Shot-Relation-Learning-with-Attention-for-EEG-based-Motor-Imagery-Classification/embedding_module.png)</center>

$B.Attention Module$  
`N-way k-shot` 的支持集包含 $N*k$ 个数据样本，在原始的关系网络中，这些样本嵌入后的特征等权重的都进行查询预测，但是如果支持集中存在噪声样本，则会影响预测的准确性。因此为了更好的解决该问题，作者提出了基于注意力模块：将每个支持集的特征向量 $z_{i,j}^{S}(i=1,2,N;j=1,2,k)$ 和查询集的特征向量 ${\mathcal{Z}}^{Q}$ 在通道维度进行连接 $z_{i,j}^{S Q}=z_{i,j}^{S}\oplus z^{Q}$，计算注意力分数 $a_{i,j}^{S}=A(z_{i,j}^{S Q})$ ，注意模块由使用 16×1 和 4×1 核的卷积层、全局平均池层和 64 个一维全连接层组成，以考虑全局和局部特征。
举个例子，对于每一个类别，$\overline{z_{i}^{S}}$ 注意力加权的特征计算如下：

$$
\overline{z_{i}^{S}}=\frac{\sum_{j=1}^{k}a_{i,j}^{S}*z_{i,j}^{S}}{\sum_{j=1}^{k}a_{i,j}^{S}}
$$

然后将特征向量 $\overline{z_{i}^{S}}$ 与 $ z^{Q} $ 串联：$z_{i}^{S Q}=\bar{z}_{i}^{\bar{S}} \oplus z^{Q}$ ，得到 $z_{i}^{S Q}$ 用于标签的预测。

$C.Relation Module$  
 关系模块使用两个具有 30×1 和 15×1 核的卷积层、一个全局平均 Poing 层和两个维度为 256 和 100 的完全连接层对 $z_{i}^{S Q}$ 预测，将预测关系得分最大的类作为标签。

$D.Implementation details$  
对于训练，我们使用交叉熵损失函数优化模型，如下所示：

$$
  {\cal Loss=-\frac{1}{N}\ast\sum_{i=1}^{l N}y_{i}\ast log(r_{i}^{S}),}
$$

其中，如果支持数据的类别与查询数据的类别相同，则 $\ y_{i}$ 为 1，否则为 0。

## **EXPERIMENTAL RESULTS**

$A. Dataset$

> BCI 竞争 IV 2b 数据集用于评估网络性能。数据集包含 9 名 MI 分类受试者的原始 EEG 数据，每个实验约 120 次试验，其中每个受试者根据指令想象左手或右手运动。在每个受试者上进行了五个实验，共从 9 个受试者收集了 45 个实验。根据国际 10-20 系统的协议，在三个电极 C3、CZ 和 C4 上以 250Hz 的采样频率测量采集的样品。在 5400 个试验（即 120×45）中，我们忽略了一些被拒绝的试验，然后在本研究中使用剩余试验的 3.5s 到 7s 的信号。从 3 个电极中，作者获得了三个 875 值，并将它们叠加形成 875×3 矩阵，作为模型的输入。

$B. Experimental Setting$

<center>![数据集划分策略](/imgs/Few-Shot-Relation-Learning-with-Attention-for-EEG-based-Motor-Imagery-Classification/%E6%95%B0%E6%8D%AE%E9%9B%86%E5%88%92%E5%88%86%E7%AD%96%E7%95%A5.png)</center>

> 在本文的实验中，使用了 9 折交叉验证，使用 8 名受试者数据样本对模型进行训练，并对每个验证事件的剩余部分进行测试。每个训练集由每名受试者的前四个实验组成，第五个实验用作验证集。测试集由剩余的全部受试者组成。在训练期间，支持和查询数据样本在每次迭代时从训练和验证集中随机选择的。为了测试，我们在每个类中将整个样本分成两组，20 个样本作为支持，其余作为查询数据。

$C. Results and Analysis$

> 表 1 显示了建议方法和其他方法的总体精度和标准偏差。图 3 以图表形式显示了 9 倍的方法的精度。在所有情况下，我们的模型都比以前的方法实现了更好的性能。具体而言，基于监督学习的方法报告，当使用从目标对象获取的足够的训练数据时，分类性能达到 87.6%，但当模型在跨对象设置中训练时，性能显著下降到 65.3%。当使用从目标受试者获取的少量数据（每类 20 个样本）训练模型时，精度也受到限制。另一方面，提出的少镜头学习技术大大提高了性能。在单次拍摄设置中，每个目标受试者仅使用一个标记样本进行训练，与 HSCNN All 相比，观察到性能提高 1.7%，5 次拍摄提高 4.9%，10 次拍摄提高 6.4%，20 次拍摄提高 8.6%。最后，我们的方法在注意设置的 20 次射击中达到 74.6%（+9.3%）的平均准确率。(待整理)

![](/imgs/Few-Shot-Relation-Learning-with-Attention-for-EEG-based-Motor-Imagery-Classification/BCI2b9%E6%8A%98%E5%87%86%E7%A1%AE%E7%8E%87.png)

<center>![图1.基线算法比较](/imgs/Few-Shot-Relation-Learning-with-Attention-for-EEG-based-Motor-Imagery-Classification/%E5%9F%BA%E7%BA%BF%E7%AE%97%E6%B3%95%E6%AF%94%E8%BE%83.png)</center>

$D. Effect of data augmentation in the few shot setting$

数据增强后的 9 折，如表 2：  
![表2数据增强后的九折](/imgs/Few-Shot-Relation-Learning-with-Attention-for-EEG-based-Motor-Imagery-Classification/数据增强BCI2b9折准确率.png)

跨数据集（BCI competition IV 2a ）如表 3：
![表3跨数据集的结果](/imgs/Few-Shot-Relation-Learning-with-Attention-for-EEG-based-Motor-Imagery-Classification/跨数据集（BCI2a）准确率.png)
