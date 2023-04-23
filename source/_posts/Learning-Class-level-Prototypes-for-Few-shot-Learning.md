---
title: Learning Class-level Prototypes for Few-shot Learning
date: 2023-02-19 09:54:30
tags:
  - Prototypes Network
  - Class-level Prototypes
---

用于小样本学习的类级原型

<!--more-->

## Abstract

小样本学习一般通过平均运算来得到原型，易受离群点的影响。因此，本文提出了一个简单有效的小样本分类框架，该框架可以 episode 中学习更好的原型表示。生成的原型旨在接近某个类级别（class-level）的原型，使其不受原型抖动的影响。

## Introduction

尽管原型网络已经取得了令人满意的成果，但是他们很少专注于设计一个好的原型来表示支持集中的每个类。在一些小样本学习中，由于支持集样本较少，一个离群点就会对平均策略有很大的影响，这种现象被命名为类级原型抖动，如图 1 所示。

![](\imgs\Learning-Class-level-Prototypes-for-Few-shot-Learning\类级原型抖动.png)

<center>图1.类级原型抖动</center>
为了解决这个问题，本文提出了一种基于episode的原型生成器，来提取类级监督的原型。通过这种方式这一生成鲁棒性较强的原型，减少离群点的影响。本文的框架包含三个主要模块：类级原型生成器模块、episode原型生成器、度量模块。其中类级原型生成器旨在从训练数据中学习原型（训练集中每类样本特征的全局平均值）。episode原型生成器是一个用Transformer构建的可学习模块，他可以帮助网络从相同类别的样本中捕获上下文信息。此外，episode原型生成器输出有类级原型生进行监控，通过这种方式，所提出的epside原型生成器模型可以使原型更接近类级原型。本文的贡献主要包括：

- 作者假定在小样本学习中，类级原型是良好的选择，并通过消融实验证明其合理性。
- 作者提出一种机制，将支持样本作为输入，并生成与相应类级原型相对接近的类级模型。
- 所提模型在跨数据集任务上也有高效的表现（miniImageNet $\rightarrow$ CUB-200-2011）。

## Related Works

原始的原型网络仅用算术平均值作为原型，这无法解决样本中存在的离群点问题。因此，已经提出了几种方法用来缓解这个问题，例如：

- CTM 引入了用于生成原型的类别遍历模块，该模块可以显示的提取类内共性，类间唯一性。
- IFSM 假设相对可行的原型可以由该支持类样本嵌入的线性组合表示，并采用迭代过程来逐步优化该线性组合。
- Subspace Network 基于类似假设，为每一个查询样本生成一组独立的原型，但仅限于某些类型的距离度量。

本文基于以上的工作提出可学习的神经网络生成原型，本文通过类级原型监督，有助于获取更借鉴类级原型的原型。

## Method

### Framework

为了克服原型抖动的问题，作者提出了一种使用少量支持样本的来拟合类级原型。图 2 展示了作者提出的总体架构，由三部分组成：类级原型生成器、episode 原型生成器、度量模块。

![](\imgs\Learning-Class-level-Prototypes-for-Few-shot-Learning\总体框架.png)

<center>图2.总体框架</center>
如图2所示，类级原型分三步：训练数据首先被输入到类级原型生成器，训练通用分类模型，宁冻结分类器的参数；然后，使用训练的分类器获得每个样本的嵌入特征；最后，将每个类别中多有样本特征的算术平均值作为类级原型。类级原型生成器为episode原型生成器提供固定参数的主干，episode原型生成器用于设计更接近类级原型的原型，于4.3中详述。度量模块则通过度量距离来判别样本所属。

### **Class-level Prototype Generator Module**

类原型生成器模型旨在生成同一类中的所有样本特征的平均值，首先利用 $D_{train}$ 训练分类器，该分类器由主干网络 $F_\theta$ 和 具有 $softmax$ 的全连接层组成，并利用交叉熵损失进行优化。然后使用训练后的主干网络 $f_\theta$ 提取训练集中的样本特征，并采用同一类中所有特征的算术平均值作为类级原型。

$$
g_i = \frac{1}{M}\times\sum_{j=1}^{n}F_\theta(x_{ig})
$$

其中 $M$，是第 $i$类的样本总数，$x_{ij}$ 表示输入样本。

### **Episodic Prototype Generator Module**

对于每个 episode，同类中的样本 $\{x_{i1},...,x_{ik}\}$ 首先被送入预训练的特征提取网络 $f_\theta$ 中，可以获得其特征向量，表示 $f_{ij}=f_{x_{ij}}$。然后提取到的特征向量 $\{f_{i1},...,f_{ik}\}$ 被送到 episode 原型生成器 $G_\gamma$ 产生原型 $f_i'$。如下式：

$$
f_i'=G_\gamma(f_{i1,...,fik})
$$

其中 $\gamma$ 是 episode 原型发生器 $G$ 的参数，$i$ 表示第 $i$ 类。
最后作者计算了 episode 原型 $f_i'$ 和类级原型 $\hat{f}_i$，并使用距离损失来优化原型生成器 $G_\gamma$ 的参数 $\gamma$，本文采用欧式距离来计算距离损失。

$$
L_{distance}=\frac{1}{N}\sum_{i=1}^NM(f_i',\hat{f}_i)
$$

其中，$M$ 是度量模块，$N$ 是一个 episode 中类别的数量， $\hat{f}_i$ 是第 $i$-th 个类级原型。
在本文中，作者采用了一种多头自注意力的机制作为特征生成器 $G$，考虑了一个 episode 中样本的上下文信息，使用多头自注意力输出数额的样本细化特惠总能，在使用更新的特征向量的均值作为原型，这些原型可以在损失函数 $L_{distance}$ 下收敛到类级原型，从而消除离群点的影响。

### Metric Module

度量模块 $M'$ 对通过计算支持集和查询集间的距离进行分类。

## Experiments

### Datasets

_mini_ ImageNet:训练集：验证集：测试集=64：16：20

_tiered_ ImageNet:训练集：验证集：测试集=351：97：160
_mini_ ImageNet $\rightarrow$ CUB-200-2011: 验证模型的跨域学习能力。

### Implementation Details

- Backbone：_ResNet_
- Training Schema：使用 _N-ways K-shot_ 策略
- Evalution protocols：随机抽样 600 个样本，对于每次抽样随机选取 15 个样本。

### Comparison with State-of-the-arts

![](\imgs\Learning-Class-level-Prototypes-for-Few-shot-Learning\结果.png)

<center>图3.实验结果</center>
结果如图 3 所示，本文的算法在 _one-shot_ 分类任务上取得了比其他方法更先进的分数。PSN、DSNMR 和 PN+IFSM 分别提出通过学习形成放仿射空间或通过迭代机制来优化类级原型。

### Performance in Cross-domain Scenario

![](\imgs\Learning-Class-level-Prototypes-for-Few-shot-Learning\跨域任务小样本准确率.png)

图 4 表现了作者在跨任务域上的实验，该方法在 MinnieImageNet 上训练，并在 CUB-2011 上测试，backbone 同用 ResNet-12，与其他方法相比，展现了本文算法的优越性。

### The Effect of Episodic Prototype Generator

本文进行了消融实验，以验证所提机制的有效性，实验所用数据集时 _mini_ ImageNet。如图 5 所示，_ProtoNet+Lock_ 是冻结了主干参数，以算术平均值作为原型计算。本文的方法基于 _ProtoNet+Lock_ 并应用了特征生成器模块。与 _ProtoNet+Lock_ 相比，所提模块在 _one-shot_ 实验上改进最大，证明了本文机制的有效性。

![](\imgs\Learning-Class-level-Prototypes-for-Few-shot-Learning\消融实验.png)

<center>图 5.消融实验</center>
如图所示当使用类级原型进行预测时，预测精度可以达到89.1%，与其他方法还是有很大差距的，因此本文通过改进episode原型生成器，进一步缩小差距。为了验证所提方法的有效性，图图6所示，作者使用t-sne可视化了支持样本、类级原型、episode原型、均值原型，不同颜色表示不同类别，**星形**点表示类级原型、**三角形**表示平均原型、**五边形**表示本文算法形成的原型，**大圆点**表示支持集特征、**小圆点**表示样本特征。

![](\imgs\Learning-Class-level-Prototypes-for-Few-shot-Learning\不同情境下原型特征的可视化.png)

<center>图6.不同模式下原型的可视化</center>

## Dicussion

在小样本任务中，原型往往受离群点的影响，通过观察，作者发现 _class-level_ 可以大大改进小样本学习模型。本文中提出了一种从少数支持集学习类级原型的简单方法。这种方法可以生成更有效的原型，并使用类级原型进行监督，与其他改进原型的方法相比，取得了有效的结果。
