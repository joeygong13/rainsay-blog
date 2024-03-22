---
title: "STM32 的基础架构"
date: 2024-03-21T11:28:27+08:00
draft: true
description: "介绍 STM32 F1 的整体架构"
---

{{< figure src="./stm32-arch-f.jpg"  class="img-center" >}}


## 1. 总线

### 2.1 内部总线

+ AXI 总线(Advanced eXtensible Interface，高级可扩展接口): 用于连接 CPU 和内部存储器、外设。AXI 总线是高性能、多主控总线，支持读写操作。 
+ AHB 总线(Advanced High-performance Bus，高级高性能总线): 用于连接 CPU 和高速外设。AHB 总线是高性能、单主控总线，支持读写操作。
+ APB 总线(Advanced Peripheral Bus，高级外围总线): 用于连接 CPU 和低速外设。APB 总线是低功耗、单主控总线，支持读写操作。
  + APB1 工作频率通常为系统时钟的 2 倍，APB1 通常连接低速外设，如定时器、看门狗、I2C 等
  + APB2 工作频率通常为系统时钟的 4 倍，APB2 通常连接高速外设，如 SPI、USART、ADC

### 2.2 外部总线

+ FSMC 总线: 用于连接外部存储器，如 Flash、SRAM。FSMC 总线支持多种存储器类型和读写操作。
+ SDIO 总线: 用于连接 SD 卡等存储设备。SDIO 总线支持高速数据传输和读写操作。
+ USB 总线: 用于连接 USB 设备。USB 总线支持多种数据传输模式和读写操作。
+ Ethernet 总线: 用于连接以太网设备。Ethernet 总线支持高速数据传输和读写操作。