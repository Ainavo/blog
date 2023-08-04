---
title: 关于Paddle C++ 算子开发的 a little notes~
date: 2023-04-23 12:40:07
tags:
  - paddle
  - C++
published: false
---

<!--more-->

# C++ 算子的开发流程

1、新增算子描述及定义：描述前向、反向算子的输入、输出、属性，实现 InferMeta 函数；
2、新增算子 Kernel：实现算子在各种设备上的逻辑；
3、封装 Python API：封装 Python 端调用算子的接口；
4、添加单测：验证算子的正确性；

# C++ 算子中函数对应
