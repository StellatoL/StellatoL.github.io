---
title: "基本设置"
published: 2025-01-01
description: "RCCAPB2PeriphClockCmdRCCAPB2PeriphGPIOX,ENABLE;"
tags:
  - "STM32"
  - "GPIO"
category: "STM32"
categoryPath:
  - "RM_PRIME"
  - "RM电控"
  - "学习记录"
  - "STM32"
draft: false
---

# 1.开启时钟以及初始化GPIO口
---
## 开启时钟
```c
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOX,ENABLE);
```
**具体代码解释：**
	因为GPIO挂载在APB2总线上，所以此函数是用于开启GPIO_x上的时钟信号

**参数：**
	1.*RCC_APB2Periph_GPIOx*
		其中，x代表数字，这个参数用于控制开启时钟的GPIO端口号
	2.*ENABLE*
		这个参数表示开启或关闭时钟，对应*ENABLE*和*DISABLE*

---
## 初始化GPIO口
```c
GPIO_InitTypeDef GPIO_InitStructure;

GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_x;
GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;

GPIO_Init(GPIOX,&GPIO_InitStructure);
```
**具体代码解释：**
