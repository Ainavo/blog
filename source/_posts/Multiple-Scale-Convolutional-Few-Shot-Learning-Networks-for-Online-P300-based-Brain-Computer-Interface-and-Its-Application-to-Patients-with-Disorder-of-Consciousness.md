---
title: >-
  Multiple Scale Convolutional Few Shot Learning Networks for Online P300-based
  Brain-Computer Interface and Its Application to Patients with Disorder of
  Consciousness
date: 2023-04-24 10:59:46
tags:
  - few-shot
  - p300
---

基于 P300 的首发原型网络

<!--more-->

## Abstract

本文提出了一种多尺度卷积小样本网络（MSCNNFSL）来检测和识别小样本 P300 信号。设计了多尺度卷积网络，以不同尺度的卷积核学习多尺度特征，以捕获更多脑电信息，分类器引入一个具有余弦距离的原型网络作为分类器。本文的主要贡献如下：

- P300 领域第一次小样本应用；
- 设计了一种多尺度的卷积神经网络结构（MSCNN），从 EEG 信号的不同感受域提取丰富的多尺度特征；
- 为了避免对 EEG 中的小样本 P300 信号进行过度拟合和分类，采用余弦距离的原型网络作为分类器对目标 P300（T-P300）和非目标 P300 进行分类；
- （在线实验）该方法已成功应用于 12 例 DoC 患者的意识水平评估，证明了其在临床上的可行性和有效性。

## METHODS

A、Network Architectures
1）多尺度特征提取器：MSCNN 由七层组成，分别为 L0 到 L6~：

- L0—Input Layer
- L1—Spatial Convolution Layer
- L2—Temporal Convolution Layer
- L3—Integration Layer
- L4—Maxpooling Layer
- L5— Flatten and Fully Connected Layer
- L6—Fully Connected Layer

2）Prototypical Classifer (PC)
_（常规的余弦距离而已）_
B、Model Training, Validation and Testing
1）Strategy：
