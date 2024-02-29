---
abbrlink: ''
categories: []
date: '2024-02-29T08:36:41.315305+08:00'
tags:
- 嵌入式
- STM32
title: UART串口通信
updated: '2024-02-29T11:28:19.180+08:00'
---
记录一下emb实验中学到的比较有用的技术。

---

### UART

UART是一种异步串行通信接口，常用于通过串口与外部设备进行通信。它通过发送和接收数据帧来实现数据传输，使用起来相对简单。UART通常包含发送器（Transmitter）和接收器（Receiver），通过两根信号线（传输线）进行双向通信。

通常开发板提供了一个usb转串口的miniusb口，其接到核心板的usart1串口。USART可以通过时钟控制进行同步通信，在异步通信时与UART没有区别。

UART协议将一长串数据切成很多固定长度的小段，分别发送。每小段数据前后会加上一些附加数据以保证通信的实时性和准确性，最后形成的每个小段叫做一个数据包——即1帧数据。

- 起始位：发出1位低电平信号，表示开始传输字符。
- 数据位：真正发送的数据，一般为8位（1个字节），常采用ASCII编码，从最低位开始发送。
- 校验位：用于检验接收到的数据是否正确，分为奇校验和偶校验。
- 停止位：一组数据的结束传输的标志。可以是1位、1.5位、2位的高电平。
- 空闲位：空闲时数据线为高电平状态，代表无数据传输。
- 波特率：衡量传输速率的指标。UART通信中波特率等于比特率。

UART通信的两个设备间，以上因素必须完全一致才能实现数据通信。（实践中通常不用特意配置，默认即可）

UART有轮询、中断和DMA三种收发方式：

- 轮询：CPU 不断检测串口的状态标志来判断数据收发的情况。程序设计简单，但CPU 在检测标志位期间，无法执行其他任务，CPU 利用率较低。
- 中断：使能中断后，接收一字节数据或发送一字节后申请中断，在 ISR 中完成后续处理。在数据收发期间CPU 可以执行其他任务，CPU 利用率较高。
- DMA：初始化时设置相关参数，启动 DMA 传输后，数据传输过程不需要CPU 的干预。传输完成后，再产生DMA中断，由CPU进行后续处理，传输效率最高。

可以看到DMA方式是相对最高效的，它无需CPU干预直接把串口数据写入内存，仅当接收足够长的数据后才进入中断（相对地，中断方式收到每个字节都会申请中断）。

### 使用DMA空闲中断接收不定长数据

实际应用中，我们常常需要接收一段消息，其总长度无法预知。当然，从底层的角度一切都是断断续续的01数据流，什么叫一段消息？这里我简单将其定义为相对连续的一段数据流。

为了接收不定长数据，我们需要考虑一种接收方式。

考虑过几种方法，一种方法是在发送时给不定长数据填空，变成定长数据，通过DMA接收（HAL库中，可以在开启DMA接收的函数中传参指定触发中断的数据量）

一种是规定一个数据帧的起始或终止符号，逐字节接收判断帧头帧尾来确定是不是收到了完整的一段数据，可以采取中断方式接收。

当然上述方法其实是比较通用的，可以用任何接收方式包括轮询，但是显然比较低效且麻烦。

最后查阅了相关资料，找到一种比较高效的方式，即串口空闲中断+DMA接收。

空闲中断（Idle Interrupt）是UART通信中的一种中断类型。当UART处于空闲状态（没有接收到数据）且持续时间超过一个帧的传输时间时，空闲中断会触发，调用中断处理函数。空闲中断可以用于检测数据帧的结束或接收数据的完成。

使用DMA+空闲中断即可直接把数据写入内存，且仅在接收到一个完整的数据帧后触发中断，实现高效的不定长数据接收。

HAL库提供了现成的一组函数

HAL\_UARTEx\_ReceiveToIdle\_DMA(UART\_HandleTypeDef \*huart, uint8\_t \*pData, uint16\_t Size)

HAL\_UARTEx\_RxEventCallback(UART\_HandleTypeDef \*huart, uint16\_t Size)

前者可以开启DMA空闲接收，后者是中断触发时的回调函数，由用户自行重载，在这里处理接收到的数据。

想要实现命令行交互的功能，只需要在回调函数中解析命令，根据命令控制GPIO或向串口发送对应消息即可。

### 实践

使用STM32cubeMX生成代码，引脚分配略。

把USART1设置成异步收发模式。

![](https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/f1b00df7b9d13942426b0c661943e957.png)

由于需要使用DMA接收，开启外围到内存的DMA流。

![https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/812d87d772881b7ff8aa58cd78c8d9d6.png](https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/812d87d772881b7ff8aa58cd78c8d9d6.png)

开启全局中断。

![https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/7b46922fa3c06c5510899ca474020c59.png](https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/7b46922fa3c06c5510899ca474020c59.png)

生成代码。

声明接收数组。

![https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/d41fa8c9502cd4c54b0801f5dc1a7642.png](https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/d41fa8c9502cd4c54b0801f5dc1a7642.png)

在主函数中调用DMA空闲接收函数启动接收。这里我会在中断回调函数重新启动接收，因此无需将其写在while循环中。

![alt=https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/a23446f54a227caf841c9aefea8e7f1c.png](https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/a23446f54a227caf841c9aefea8e7f1c.png)

**重载fputc，把printf重定向到串口输出。（有用的trick）**

![https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/cd0e4e183a5086993e36c27d3133b1b7.png](https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/cd0e4e183a5086993e36c27d3133b1b7.png)

重载回调函数。由于可以接收不定长数据，这里代码是从实验中截取，大概是做一个串口CLI。逻辑很简洁，直接匹配字符串前缀然后进入对应命令的处理函数即可。处理完数据后清空接收缓冲区并重新启动DMA接收，等待接收下一次数据。

![https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/1ac7c7d6b70baa7f213ecfe8a601a1f2.png](https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/1ac7c7d6b70baa7f213ecfe8a601a1f2.png)

![](file:///C:\Users\翻白肚~1\AppData\Local\Temp\ksohtml24360\wps4.jpg)
