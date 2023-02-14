---
title: >-
  Relation Learning Using Temporal Episodes for Motor Imagery Brain-Computer
  Interfaces
date: 2023-02-13 09:25:09
tags:
  - Relation Learning
  - MI
---

基于关系网络的运动想象时间片段研究

<!--more-->

## Abstract

本文提出了一种基于时间片段学习的关系网络 temporal episode relation learning (TERL) 来解决 MI 数据量不足的问题，TERL 包含嵌入网络、时序网络、关系网络，基于 episode-training 的训练方式，在每个 episode 中加入时序编码，以提高分类精度。

## ISSUES&&Contributions

**issues：**

- 在之前的 MI 小样本研究中，样本间（Trial）的时序信息被忽略了。
- 之前的 MI 小样本研究中，只对离线实验进行了评估，没有对在线实验进行评估。

**contribute：**

- 本文提出了一种基于时间片段的关系网络，详细设计了 episode-train 的实验顺序和抽样策略，提高模型的性能。
- 除了离线实验，还模拟了在线实验，充分证明该网络的有效性。
- TERL 相对 baselines 在离线和在线实验中准确率分别提高了 0.5~2.9%和 1.2~4.0%。

## INTRODUCTION

目前大部分 sota 算法都是遵循标准的监督方式来学习迷行，损失函数一般用交叉熵来监督。这种方式在离线评估上有着不错的表现，但是并不适用于现实场景下的 MI-BCI（因为最初只能从目标被试收集有效的数据）。为了缓解该问题，利用迁移学习的方法将大量其他被试的数据生成模型，对目标被试进行预测，例如：fine-tuning、domain adaptation 等。然而这些策略仍然需要一个合理数量的样本集（至少 200 个试次），在 real-word 中收集数据时间较长，所以这也不是最佳的选择。  
相对于直接监督的方式，小样本学习中的度量学习，可以将模型的学习过程引导成一个更简单的任务--查找**样本与类之间**的关系。该模型将待预测的样本与少量有标记的样本进行比较，通过“距离”或者匹配分数来判别类别。本文参考 Relation Network，首先利用大量带标签数据训练一个可以识别样本之间关系的网络，然后将待识别样本，输入模型进行识别。这完美的适应了真实世界的 BCI 在线预测，以及脑机接口相关实现。

## TEMPORAL EPISODE RELATION LEARNING

A、问题介绍：_N-ways K-shot_  
小样本学习中存在训练集（training set）、测试集（testing set）、支持集(support set)、查询集（query set）的定义。

- Training Set：一般是大量的已标注的数据，包含支持集和查询集。
- Testing Set：少量目标域标注数据或无标注数据，以及待识别数据；包含支持集和查询集。
- _N-ways K-shot_：N 表示类别；K 表示样本数量。
  对于 Testing Set 一般会选择 N 类，每类选出 K+S 个样本（S 是待预测的样本数量），其中 K 个样本用作支持集，S 个样本用作查询集，而这个过程被称为 _N-ways K-shot_。但是 Training Set 对于支持集和查询集的选取，就不需遵从 K-N 的法则，只需要选择的类别大于 N 即可。
- episode：每次对数据集进行抽样，选择几个类，并选择每个类对应的**支持集**和**查询集**，然后进行训练，计算损失，更新参数，这个过程就被成为 episode-training。下一个 episode 可以再选择其他的类进行训练。一个 epoch 包含多个 episode。

B、整体框架：如图 1  
![图1、整体框架](\imgs\Relation-Learning-Using-Temporal-Episodes-for-Motor-Imagery-Brain-Computer-Interfaces\整体框图.png)
(这里是自己的一些话术)对于一个训练 episode 首先从所有的源被试抽样出支持集和查询集，然后在保持时序的情况下，输入 $f_\theta$ 嵌入网络，生成特征向量；然后通过时序网络进一步提取时序特征，在通过 $Sum$ 函数求每类特征向量；同样的查询集通过 $f_\theta$ 嵌入网络生成特征向量并进行复制，然后将类特征向量和查询特征向量相连接，输入 $h_\gamma$ 关系网络得到相关分数，分数最高的类即为标签。同样的，对于测试 episode，首先从目标被试中抽样，其余步骤都与训练 episode 相同。

C、网络结构：如图 2  
![图2、各部分网络结构](\imgs\Relation-Learning-Using-Temporal-Episodes-for-Motor-Imagery-Brain-Computer-Interfaces\各部分网络结构.png)

- $f_\theta$:EEGNet 是 MI 分类较为常用的网络。对于嵌入网络（$f_\theta$）,本文选择了 EEGNet 同款架构，但是舍弃了全连接层（负责监督分类）。
- $g_\phi$:时序网络由一个简单的 1D CNN 构成，如图 2 所示：其内核大小为 $k_s$ 沿着样本大小的维度进行卷积。步长为 1，在第一个样本前补充 $(k_s-1, 0)$ 的零值。
- $h_\gamma$:关系网络由两个 `CNN` 层和两个 `dense` 组成，如图 3 下。

D、训练集抽样策略：
测试集中的支持集和测试集都是来自同一被试，为了更好的模拟这一过程，训练集的 S 和 Q 也分别来自同一被试【**约束 A**】；此外，对于在线预测，另一个重要的假设是测试集的实验重视发生在支持集之后【**约束 B**】。对于离线评估，本文只采用【约束 A】；对于在线实验，本文采用了【约束 A】和同时用【约束 A】和【约束 B】的采样策略。

E、TERL 的优势

- 对于支持集和样本的采样，TERL 框架中的 MI 实验是按照时间顺序排列的，以捕获样本间的时序关系。
- 本文提出的 $g_\gamma$ 只覆盖了样本自身和之前的样本，因此符合 real-world 的实验设定。

## EXPERIMENTS

A、Databases（略）  
![图3.数据划分](\imgs\Relation-Learning-Using-Temporal-Episodes-for-Motor-Imagery-Brain-Computer-Interfaces\数据划分.png)

数据划分大致如图 3 所示，选择一个作为目标被试，其余则为源域被试。

B、离线策略：如图 4
![图4.离线抽样策略](\imgs\Relation-Learning-Using-Temporal-Episodes-for-Motor-Imagery-Brain-Computer-Interfaces\离线抽样.png)

teseting set：支持集和查询集均来源于同一被试，在生成支持集的同时样本以时间序列排列。

C、在线策略：如图 5
![图5.在线抽样策略](\imgs\Relation-Learning-Using-Temporal-Episodes-for-Motor-Imagery-Brain-Computer-Interfaces\在线抽样策略.png)

testing set：相比于离线采样的策略，在线采样首先选取待测数据前一部分作为最小子集，剩余部分作为查询集，而支持集通过最小子集按照时序顺序生成。

D、**Baseline 和对比算法**  
1)、Baselines：

- **Few**：仅用支持集训练 EEGNet。
- **Source only(SO)**:仅用其他域数据进行训练 EEGNet。
- **Combine**:用训练集和支持集的数据训练 EEGNet。
- **Basic I**:模型中仅包含嵌入网络和关系网络。
- **Basic II**:模型中仅包含嵌入网络和关系网络，同时利用【约束 A】进行约束。

2）SOTAs：

- **Fine-tuning(FT)**:利用其他被试数据进行预训练，目标被试数据进行微调。
- **DDAN**:由交叉熵、最大平均差异（MMD）和中心损失联合训练。
- **DRDA**:利用域对抗训练学习域不变特征，利用中心损失增强模型对目标对象的识别能力。
- **Model A**：包含嵌入网络、时序网络、关系网络，同时利用【约束 A】进行约束。
- **Model B**：包含嵌入网络、时序网络、关系网络，同时利用【约束 A】和【约束 B】进行约束。

E、Training and optimization
对于 BCIIV-2a 和 BCIIV-2b 数据集 episodes 设置为 15000，k 设置为 10，m 为 $k*N$。优化器使用 Adam 优化器，学习率为 0.001。

## RESULTS AND ANALYSIS

A、离线评估结果:如图 6  
![图6.离线实验结果](\imgs\Relation-Learning-Using-Temporal-Episodes-for-Motor-Imagery-Brain-Computer-Interfaces\离线实验结果.png)

在四个数据集上都表现了仅以支持集训练的模型在所有方法中精度最低，这与预期相吻合，其他结果（除 SO、Combine）表示都需要更多的目标被试数据参与训练。从理论上说，TL 和 DA 的方法搜可以以最小化域间距离，从而提高模型在目标域上的拟合。但是与作者预期相反的这两种方法（即 FT、DDAN、DRDA）在四个数据集上的品骏分类精度没有 Combine 高，同时，只有少量来自于目标被试的样本时，这些方法无法表现出他们原文中展现出的优异性。  
B、在线评估结果：如图 7  
![图7.在线实验结果](\imgs\Relation-Learning-Using-Temporal-Episodes-for-Motor-Imagery-Brain-Computer-Interfaces\在线实验结果.png)

本文所提方法（model A 和 model B）在四个评估数据集上表现最好，与其他模型相比，模型 A 的品骏分类准确分别提高了 2.8%、1.2%、4.0%和 1.6%。本文的两种抽样方案在准确率上没有显著性差异（即所有 p>0.05），模型 A 在四个数据集上都小优模型 B。这与作者的预期不符，即在支持集之后，再采样查询集更适合测试界面，理应好于模型 A。分析原因：B 的抽样技术大大限制源样本的 S 和 Q 的组合，降低了泛化性。  
![图 8.校准时间](\imgs\Relation-Learning-Using-Temporal-Episodes-for-Motor-Imagery-Brain-Computer-Interfaces\校准时间.png)

在图 8 中给出模型再训练的时间，除了 SO 之外的模型均需要二次训练，其中 DDAN 和 DRDA 要求原数据和目标数据像组合，以便模型拟合目标被试，因此需要更长的时间。相反，小样本学习不需要对目标数据二次训练，即可以做到即插即用，极大的增加了现实中 MI-BCI 的可用性。

C、K-way setting

从目标被试获得的数据越多，意味着模型表现出来的性能越好，在现实场景中，假定来自目标被试的数据特别有限，作者研究了取不同 K 值，对模型性能的影响，
因此在三个数据集上，分别尝试了 $K=\{0，5，10，15，20\}$ ，其中 K=0，对应零样本实验，即从源域随机选取 10 个样本作为支持集。  
![图9.不同K值的影响](\imgs\Relation-Learning-Using-Temporal-Episodes-for-Motor-Imagery-Brain-Computer-Interfaces\不同K值的影响.png)

如图 9 所示，当 K 取值较小的情况下，TERL 表现好于 FT 和 DRDA。当 K=10 时，TERL 模型效果最好，但随着样本数量的增加，TERL 的表现在部分数据集上不如其余两个模型。

D、Temporal kernel size

1D 卷积的内核大小通过不同的参数，来验证参数的敏感性。通过在 $\{0, 5, 10, 15, 20\}$ 的范围内改变内核的大小，以表示不同的时间长度。当 $k_s=0$ 时，意味着不用时序网络。图 10 展示了不同卷积核下模型的性能对比。当不使用时序网络时，准确率最低，当内核在 5~20 之间变化时，模型性能相对稳定，表明了该模型对内核大小变化不敏感，最短的内核即可获得最高的精度。  
![图10.时序网络不同卷积核的影响](\imgs\Relation-Learning-Using-Temporal-Episodes-for-Motor-Imagery-Brain-Computer-Interfaces\时序网络不同卷积核的影响.png)

E、Temporal kernel weights  
![图11.1D CNN 网络热力图](\imgs\Relation-Learning-Using-Temporal-Episodes-for-Motor-Imagery-Brain-Computer-Interfaces\时序网络权重.png)
如图 11 所示，是时序网络在 S1~S9 被试，在时序排序情况下和非时序排序情况下的热力图，可以看到时序情况下的当前样本权重是最大的，同时越靠近底部的权重越大，说明了 MI 模型中时序编码的重要性。

## CONCLUSION

本文提出了一种基于时间片段的小样本关系网络，包含嵌入网络、时序网络、关系网络。相比之前的研究关注了样本间的时序信息，在离线和在线评估中准确率分别提高了 2.9%和 4.0%。此外该模型，不需要对新被试再训练，可以有效的节约时间减少成本。本文使用了一个简单的 1D CNN 对时序信息进行编码，这里可以引进其他更复杂的结构，例如**LSTM**或**Attention**模块。
在本文中，作者只关注了 BCI 的同步设置，而没有反馈。实际上，在每次实验后，进行反馈，常常被用于患者的运动康复项目。在未来的研究中，$g_\gamma$ 时序网络可以应用到异步脑机接口，这种类型的 MI 常被应用于现实中常见的 MI-BCI 控制场景。异步 MI-BCI 通常分为有意控制（IC，执行 MI 任务）和非控制（NC，空闲时间）。$g_\gamma$ 可以被设计针对当前点和当前时间点之前的信号信息进行编码，而无需考虑它们是否来自同一个 IC，而不是忽略 NC 信号。这允许 $g_\gamma$ 对异步 BCI 中的时间模式进行编码，并提高其性能。
