---
title: STM32F10x_hc
date: 2023-09-16 15:04:41
tags:
  - ARM
  - STM32
---

[STM32F103 入门](https://www.bilibili.com/video/BV1N7411x7Yk)

<!--more-->

## 库函数和寄存器操作

- 库函数操作相当于 C++，面向对象
- 寄存器操作相当于 C，面向过程

## stm32f103 相关介绍

参考手册：怎么配置相关寄存器
数据手册：引脚分布、电气特性

- 使用 GPIO 步骤：

  - 打开 PC13 对应时钟
  - 配置输出，确定输出模式
  - 输出低电平（点亮 LED）

- 头文件中寄存器的预定义（总线-->分线-->端口-->寄存器-->位）

```C
#ifndef __STM32F01X__H
#define __STM32F10X__H

#define PERIPH_BASE ((unsigned int)0x40000000)  //外设总线地址
#define APB2PERIPH_BASE (PERIPH_BASE+0x10000)  // APB2 线上基地址，偏移量 0x10000
#define AHBPERIPH_BASE (PERIPH_BASE+0x20000)  // AHB 线上基地址，0x2000偏移量是为了方便计算，手册基地址为 +0x8000
#define GPIOC_BASE (APB2PERIPH_BASE+0x1000) //GPIOC 端口基地址
#define GPIOC_CRH *(unsigned int*)(GPIOC_BASE+0x04) //高位寄存器地址
#define GPIOC_IDR *(unsigned int*)(GPIOC_BASE+0x08) //低位寄存器地址
#define GPIOC_ODR *(unsigned int*)(GPIOC_BASE+0x0C)

#define RCC_BASE (AHBPERIPH_BASE+0x1000)  // 时钟基地址
#define RCC_APB2ENR *(unsigned int*)(RCC_BASE+0x18)
#endif

```

## GPIOC 结构体

```C
typedef unsigned int uint32_t;
typedef struct{
  uint32_t CRL;
  uint32_t CRH;
  uint32_t IDR;
  uint32_t ODR;
  uint32_t BSRR;
  uint32_t BRR;
  uint32_t LCKR;
} GPOI_TypeDef;  // 结构体内部成员是依次排列的；两个之间相差4个字节 0x04

(GPOI_TypeDef*)GPIOC_BASE //将 GPIOC_BASE 转换为指针并赋予GPOI结构
#define GPIOC (GPOI_TypeDef*)GPIOC_BASE

GPOIC -> CRL; //直接访问CRL...嘿嘿
```

## 模块化

```C
void GPIOC_SetBits(GPIO_TypeDef *GPIOx, uint16 GPIO_Pin)
{
  GPIOx->BSRR |= GPIO_Pin;  //GPIO_Pin 表示引脚位置
}

```

## 引脚配置

- 配置时钟
- 确定引脚组
- 配置模式：
  - 输出模式，最大速度为 10MHz
  - 通用推挽模式
- 确定引脚，以及确实输出低电平

```C
  RCC->APB2ENR |= 1<<4;
  GPIOC->CRH &=~(0x0F<<(4*5));
  GPIOC->CRH |=(1<<(4*5));
  GPIOC_ResetBites(GPIOC, GPIO_Pin_13);

  // 等效

  typedef struct
  {
    uint16_t GPIO_Pin;
    uint16_t GPIO_Speed;
    uint16_t GPIO_Mode;
  } GPIO_InitTypeDef;

  RCC->APB2ENR |= 1<<4;

  GPIO_InitTypeDef GPIO_InitStructure;
  GPIO_InitStructure.GPIO_Pin = GPIO_Pin_13;
  GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
  GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
  GPIO_Init(GPIOC, &GPIO_InitStructure);

  GPIOC_ResetBites(GPIOC, GPIO_Pin_13);
```

## 枚举(enum)

```C
typedef enum
{
  GPIO_Mode_AIN = 0x0,
  GPIO_Mode_IN_FLOATING = 0x04,
  GPIO_Mode_IPD = 0x28,
  GPIO_Mode_IPU = 0x48,

  GPIO_Mode_Out_OD = 0x14,
  GPIO_Mode_Out_PP = 0x10,
  GPIO_Mode_AF_OD = 0x1C,
  GPIO_Mode_AF_PP = 0x18
}GPIOMode_TypeDef;  // 对应8种工作模式，GPIOMode_TypeDef所对应的变量取值只能取8个中的一个
```

## 库函数操作和寄存器操作

类比(C++和 C)

- 面向对象(C++):类似调用接口函数

- 面向过程(C)：具体一步步怎么做

- 库函数开发方式：驱动层--(调用库接口)-->库函数层--(以函数、宏封装配置寄存器的操作)-->特殊寄存器层

- 直接寄存器配置：驱动层--(直接配置寄存器)-->特殊寄存器

```C
void GPIOC_Init()
```

## ARM 公司规定的 CMSIS 架构

- 用户层：用户代码
- CMSIS 层：CMSIS 处于中间层，向上提供给用户所需函数接口向下负责与内核和其他外设通信的函数。
  - 内核访问层：包含了用来访问内核的寄存器设备的函数
  - 中间设备访问层：为软件提供访问外设的通用方法。
  - 芯片厂商提供外设访问层--提供所有外设的定义的函数
- MCU：
  - Contex 内核
  - SysTick 实时系统内核时钟
  - NVIC 中断控制器
  - 调用跟踪模块
  - 其他外设

## 库函数特点

- stm32f10x_it.c -- 中断程序相关 it:`intrrrupt`
- stm32f10x_conf.h -- 配置相关 conf:`config`
- stm32f10x_ppp.c -- `PPP`表示外设名称
  - stm32f10x_i2c.c ...

`core_cm3.c` 和 `core_cm3.h`
`core_cm3.c`：实现了内核的寄存器映射
`core_cm3.h`：操作内核外设寄存器函数

`stm32f10x.h`
片上外设的所有寄存器的映射
`stm32f10x.h`包含`stm32f10x_conf.h`头文件

- `core_cm3.h`针对内核的外设，`stm32f10x.h`针对片上的外设

- `misc.c` 和 `misc.h`

  - `NVIC`:嵌套向量中断控制器
  - `SysTick`:系统滴答定时器

- `startup_stm32f10x_md.s`:启动文件
  | 启动文件 | 区别 |
  | ---- | ---- |
  | startup_stm32f10x_ld.s | ld:low-density 小容量， FLASH 容量在 16~32K 之间 |
  | startup_stm32f10x_md.s | md:medium-density 中容量， FLASH 容量在 64~128K 之间 |
  | startup_stm32f10x_hd.s | hd:high-density 大容量， FLASH 容量在 256~512K 之间 |
  | startup_stm32f10x_xd.s | xl：超大容量， FLASH 容量在 512~1024K 之间 |

## STM32F103 固件库

## 12 讲点亮 LED（模块化）-- 以 LED 为例

- 在 USER 文件夹新建文件夹 LED（led.c 和 led.h）
- 打开工程，在 USER 下添加 led.c，并进行简单的程序编写
- 添加 led.h 头文件路径
- 编写响应的头文件和源文件
- 最后完成主函数，注意：在主函数添加模块的头文件

## 13 讲时钟讲解

- HSE（高速外部时钟）
- HSI（高速内部时钟）
- LSE（低速外部时钟）
- LSI（低速内部时钟）
- PLL（锁相环时钟）

## 滴答定时器

- 针对模板
  - 子模块文件添加到相应的文件夹
  - 在 main.c 调价相应的头文件
  - 在主函数添加相应的初始化函数
  - 调用相应功能

## 中断概念

- 内核中断
- 外部中断

NVIC 跟内核紧密耦合，是内核里面的一个外设（参考 STM32F10x 编程手册和 Cortex-M3 权威指南(中文)）
STM32 的 NVIC 是 COrtex-M3 的 NVIC 的一个子集

- 如何管理中断

  - 抢占优先级和响应优先级的联系和区别
    - 高优先级的抢占可以打断低级别的抢占
    - 抢占相同，高响应不能打断低响应
    - 抢占相同，若响应同时发生，则响应优先级高的先执行
    - 若抢占和响应皆相同，则看中断发生的先后顺序来定

- 中断控制寄存器

```C
  typedef {
    __IO uint32_t ISER[8]; // 中断使能寄存器
    uint32_t RESERVED0[24];
    __IO uint32_t ICER[8];  // 中断清除寄存器
    uint32_t RESERVED1[24];
    __IO uint32_t ISPR[8];  // 中断使能悬起寄存器
    uint32_t RESERVED2[24];
    __IO uint32_t ICPR[8];  // 中断清除悬起寄存器
    uint32_t RESERVED3[24];
    __IO uint32_t IABR[8];  // 中断有效位寄存器
    uint32_t RESERVED4[56];
    __IO uint8_t IP[240];  // 中断有限级寄存器
    uint32_t RESERVED4[644];
    __O uint32_t STIR;  // 软件触发中断寄存器
  } NVIC_Type;
```

中断过程

```C
  NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2); //选择中断组别
  TIM_ITConfig(TIM3,TIM_IT_Update,ENABLE);  // 选择定时器
  TIM3_Init(9999,7199);  // 定时器初始化

  TIM_TimeBaseInitTypeDef  TIM_TimeBaseStructure; // 定时器结构体声明
  NVIC_InitStruct.NVIC_IRQChannel = TIM3_IRQn;
	NVIC_InitStruct.NVIC_IRQChannelPreemptionPriority=2;
	NVIC_InitStruct.NVIC_IRQChannelSubPriority = 2;
	NVIC_InitStruct.NVIC_IRQChannelCmd = ENABLE;  // 初始化定时器结构体
  NVIC_Init(&NVIC_InitStruct);  // 初始化结构体
  TIM_Cmd(TIM3, ENABLE);  //使能TIMx外设
```

## 第 16 讲 SYSTICK 定时器

控制寄存器：
| 位段 | 名称 | 类型 | 复位值 | 描述 |
| ---- | --------- | ---- | ------ | ---------------------------------------------------------------------------------- |
| 16 | COUNTFLAG | R | 0 | 如果上次读取寄存器后，SysTick 已经数到了 0，则该位为 1。如果读取该位，该位自动清零 |
| 0 | CLKSOURCE | R/W | 0 | 0=外部时钟源(STCLK) AHB/8;1=内核时钟(FCLK) AHB |
| 1 | TICKINT | R/W | 0 | 1=SysTick 倒数到 0 时产生 SysTick 异常请求；0=数到 0 时无动作 |
| 0 | ENABLE | R/W | 0 | SysTick 定时器的使能位 |

SysTick 重装载寄存器
| 位段 | 名称 | 类型 | 复位值 | 描述 |
| ---- | --------- | ---- | ------ | ---------------------------------------------------------------------------------- |
| 23:0 | RELOAD | R?W | 0 | 当倒数至零时，将被重装载的值 |

SysTick 当前数值寄存器
| 位段 | 名称 | 类型 | 复位值 | 描述 |
| ---- | --------- | ---- | ------ | ---------------------------------------------------------------------------------- |
| 23:0 | CURRENT | R/Wc | 0 | 读取时返回当前倒计数的值，写它则使之清零，同时还会清除在 SysTick 控制及状态寄存器中 COUNTFLAG 标志 |

设置思路：

- 确定时钟
- 重装数值
- 清空计数器
- 启动计数器
- 查询
- 关闭计数器
- 清空计数器

## 19 讲 f103c8t6

定时器简介

- 时钟源
- 控制器
- 实际单元

在一定时钟频率下，从零递增到一定的值或者从一定的值递减到零，*标志位改变*或者*发生中断*

- 控制寄存器 1（TIMx_CR1）
- DMA/中断使能寄存器（TIMx_DIER）
- 预分频器（TIMx_PSC）
- 自动重装载寄存器（TIMx_ARR）

定时器配置

- 使能定时器时钟：`RCC_APB1PeriphClockCmd();`
- 初始化定时器：`TIM_TimeBaseInit();`
- 定时器中断配置：`void TIM_ITConfig();`
- 使能定时器：`TIM_Cmd();`
- 编写中断服务函数：`TIMx_IRQHandler();`
- 最后要配置 NVIC

## 第 17 讲 位带操作

位带转换：xx.cc -- > xx\*32 再转 16 进制，小数部分\*32/8

```C
  // eg.0x4001080C.8(小数部分表示位号) --> 0x422101A0
  AliasAddr = 0x42000000+(A-0x40000000)*8*4+n*32/8  // A表示未带区所在的字节的地址；n表示位号(0<<n<<7), STM32 是 32 位的，按 4 字节访问最快
  // 程序中写法
  ((A&0xF0000000)+0x02000000+((A&0x000FFFFF)<<5)+(n<<2))

```

## 第 18 讲 按键控制

- 按键检测到低电平
- 延时 5s
- 再次检测是否为低电平
- 若为低电平则执行相应动作

第 20 讲 串口通信
UART(通用异步收发器)
USART(通用同步和异步收发器)

同步：发送方发出数据后，等待收方发回响应后才发下一个数据，阻塞模式
异步：发送方发送数据后，不等待收方发回响应，接着发送下一个数据包，非阻塞模式

| 通信标准 | 通信方式 | 通信方向 |
| -------- | -------- | -------- |
| UART     | 异步通信 | 全双工   |
| SPI      | 同步通信 | 全双工   |
| I2C      | 同步通信 | 半双工   |

串口编写流程

- 串口时钟、GPIO 时钟使能`RCC_APB2PeriphClockCmd();`
- GPIO 端口模式设置：`GPIO_Init();`
- 串口参数初始化：`USART_Init();`
- 开启中断并且初始化 NVIC：`NVIC_Init();USART_ITConfig();`
- 使能串口：`USART_Cmd();`
- 编写中断处理函数：`USARTx_IRQHandler();`

状态寄存(USART_SR)

- TXE:发送数据寄存器空（Transmit data register empty）
- TC：发送完成（Transmission complete）
- RXNE：读数据寄存器非空（Read data register not empty）

上位机通过串口助手发送数据给单片机，单片机再将数据通过串口助手在上位机上显示(3~7 节通过标志位判断字符的发送和接收--->轮询法(没有用到中断))

轮询法和中断法：

- 共同点：都是访问串口的 SR 状态寄存器
- 不同点：
- USART_GetITStatus():该函数不仅会判断标志位是否置 1，同时还会判断是否使能相应的中断。所以在串口中断函数中，通常会使用该函数。
- USART_GetFlagStatus()：该函数只判断标志位。在没有使能相应的中断时，通常使用该函数来判断标志位是否置 1。

eg.`ITStatus USART_GetITStatus(USART_TypeDef* USARTx, uint16_t USART_IT);`

## printf 调试(加在串口的源文件)

要想使用 printf 将内容打印到串口助手，需要将 fputc 和 fgetc 函数重定向

```C
  // 重定向c库函数printf到串口，重定向后可使用printf函数
  int fputc(int ch, FILE *f)
  {
    USART_SendData(USART1, (uint8_t) ch);
    while(USART_GetFlagStatus(USART1, USART_FLAG_RXNE)==RESET);

    return (ch);
  }

  // 重定向c库函数scanf到串口，重写后可使用scanf、getchar等函数
  int fgetc(FILE *f)
  {
    while(USART_GetFlagStatus(USART1, USART_FLAG_RXNE)==RESET);
    return (int)USART_ReceiveData(USART1);
  }

  // FILE *f是文件指针
```

```C
  #pragma import(__use_no_semihosting)
  struct __FILE
  {
    int handle;
  };
  FILE __stdout;
  _sys_exit(int x)
  {
    x=x;
  }
```
