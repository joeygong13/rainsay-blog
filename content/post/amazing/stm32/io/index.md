---
title: "STM32 输入/输出"
date: 2024-03-21T10:48:31+08:00
draft: true
---

## 1. GPIO

STM32的GPIO（通用输入输出）端口支持多种输入输出模式，这些模式允许开发者根据应用需求灵活地配置端口的行为。

{{< figure class="img-center" src="./GPIO_Struct.jpg" >}}

### 输入模式

+ 浮空输入：在这种模式下，输入引脚既不上拉也不下拉，其电平状态是不确定的，取决于外部信号。
+ 上拉输入：输入引脚通过一个内部上拉电阻连接到高电平。如果没有外部信号，引脚将保持在高电平状态。当外部信号为低电平时，引脚电平会被拉低。
+ 下拉输入：输入引脚通过一个内部下拉电阻连接到低电平。如果没有外部信号，引脚将保持在低电平状态。当外部信号为高电平时，引脚电平会被拉高。
+ 模拟输入：在这种模式下，输入引脚用于接收模拟信号，不进行数字转换。这通常用于ADC（模数转换器）等需要直接处理模拟信号的场合。

```c
static void MX_GPIO_Init(void)  
{  
  GPIO_InitTypeDef GPIO_InitStruct;
  GPIO_InitStruct.Pin = KEY_Pin;  
  GPIO_InitStruct.Mode = GPIO_MODE_INPUT; // 输入模式 
  GPIO_InitStruct.Pull = GPIO_PULLUP;  // 上拉
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;  
  HAL_GPIO_Init(LED_GPIO_Port, &GPIO_InitStruct);  
}
```


### 输出模式

+ 推挽输出：在这种模式下，输出引脚可以直接输出高电平或低电平。当输出高电平时，上臂（P-MOS）导通，下臂（N-MOS）截止；当输出低电平时，上臂截止，下臂导通。这种模式适用于需要快速切换电平状态的场合。
+ 开漏输出：在这种模式下，输出引脚通过一个漏极开路的MOS管连接到地线或电源。这意味着输出引脚不能直接输出高电平或低电平，而需要外部上拉或下拉电阻来确定电平状态。开漏输出模式适用于需要电平转换或需要多个输出引脚共享同一电平的场合。

STM32还提供了推挽复用输出和开漏复用输出模式，这些模式通常用于特定的外设功能，如I2C、SPI等通信协议。

```c
static void MX_GPIO_Init(void)  
{  
  GPIO_InitTypeDef GPIO_InitStruct;
  GPIO_InitStruct.Pin = LED_Pin;  
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP; // pp 推挽输出模式 
  GPIO_InitStruct.Pull = GPIO_NOPULL;  // 无上拉/下拉
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;  
  HAL_GPIO_Init(LED_GPIO_Port, &GPIO_InitStruct);  
}
```

{{< notice tip >}}
当GPIO处于output模式时，一般选择no pull，这意味着引脚能够正确地输出目标值，而不受外部电路的影响。在某些情况下，即使GPIO被配置为输出，外部电路也可能存在不确定的状态，尤其是当没有明确的电源或地线连接时。此时，通过配置pull参数为上拉（pull-up）或下拉（pull-down），可以确保GPIO引脚在未被主动驱动时处于一个确定的电平状态，从而避免可能出现的电路不稳定或误操作。
{{< /notice >}}


## 2. 串口通信

### 电平标准

电平标准规定了信号的电压范围和传输方式，是物理层的定义。常用的电平标准有：TTL，RS232，RS485。

+ TTL 使用 0v 作为逻辑 0，3.3v 或 5 v 作为逻辑 1，TTL电平信号的特点是电压范围小，抗干扰能力差，传输距离短，因此它一般只用于同一板卡内的通信，距离通常在一米之内。

+ RS232，RS232 类似于 TTL，只不过改变了电平范围，它使用负逻辑，即逻辑 1 对应负电平，逻辑 0 对应正电平。典型的 RS232 电平标准在 ±3v 到 ±15v 之间。它的电平压差比 TTL  高，所以抗干扰能力比 TTL 较好。

+ RS485，RS485是一种差分信号标准，使用两个信号线（正向和反向）来传输数据，即数据通过两条线同时传输，其中一条线上的电压与另一条线上的电压之差表示逻辑状态。这种差分传输方式使得RS485更能抵抗外部干扰和噪声，适用于长距离传输。RS485支持多点通信，即一个主设备可以与多个从设备进行通信。这使得RS485非常适用于工业控制和自动化系统中的分布式设备。

