---
title: FreeRTOS
date: 2023-09-13 09:08:56
tags:
  - RTOS
---

[FreeRTOS github](https://github.com/FreeRTOS/FreeRTOS)

<!--more-->

## FreeRTOS 简介

FreeRTOS 是一个可裁剪的小型 RTOS 系统，其特点包括：

- FreeRTOS 的内核支持抢占式，合作式和时间片调度。
- SafeRTOS 衍生自 FreeRTOS、SafeRTOS 在代码完整性上相比 FreeRTOS 更胜一筹
- 提供了一个用于低功耗的 Tickless 模式
- 系统的组件在创建时可以选择动态或静态 RAM，比如任务、消息队列、信号量、软件定时器等等
- 已经在超过 30 种架构的芯片上进行了移植
- FreeRTOS-MPU 支持 Corex-M 系列中的 MPU 单元如 STM32F407
- FreeRTOS 系统简单、小巧、易用，通常情况下内核占用 4k~9k 字节的空间
- 高可移植性，代码主要 C 语言编写
- 支持实时任务和协程(co-routines 也有称为合作式、协同程序)
- 任务与任务、任务与中断之间可以使用任务通知、消息队列、二值信号量、数值信号量、递归互斥量和互斥信号量进行通信和同步
- 创新的事件组（或者事件标志）
- 高效的软件定时器
- 强大的跟踪执行功能
- 堆栈溢出检测功能
- 任务数量不限
- 任务优先级不限

### FreeRTOS 编程风格
