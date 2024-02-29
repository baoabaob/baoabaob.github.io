---
abbrlink: ''
categories: []
date: '2024-02-29T09:49:21.460070+08:00'
tags: []
title: ESP8266 WIFI通信
updated: '2024-02-29T09:49:22.496+08:00'
---
记录一下emb实验中学到的比较有用的技术。

---

### ESP8266

ESP8266 是高性能无线 SOC，以最低成本提供最大实用性，为 WiFi 功能嵌入其他系统提供无限可能。
ESP8266EX 是一个完整且自成体系的 WiFi 网络解决方案，能够独立运行，也可以作为从机搭载于其他主机 MCU 运行。ESP8266EX 在搭载应用并作为设备中唯的应处理器时，能够直接从外接闪存中启动。

另外一种情况是，ESP8266EX 负责无线上网接入承担 WiFi 适配器的任务时，可以将其添加到任何基于微控制器的设计中，连接简单易，只需通过 SPI/SDIO 接口或 I2C/UART 口即可。

有ESP8266EX 的系统表现出来的领先特征有：节能在睡眠/唤醒模式之间的快速切换、配合低功率操作的自适应无线电偏置、前端信号的处理功能、故障排除和无线电系统共存特性为消除蜂窝/蓝牙/DDR/LVDS/LCD 干扰。
ESP8266 模块支持 STA/AP/STA+AP 三种工作模式。
STA 模式：ESP8266 模块通过路由器连接互联网，手机或电脑通过互联网实现对设备的远程控制。
AP 模式：默认模式 ATK\_ESP8266 模块作为热点，实现手机或电脑直接与模块通信，实现局域网无线控制。（协调器需要配置成此模式）
STA+AP 模式：两种模式的共存模式，即可以通过互联网控制可实现无缝切换，方便操作。（节点是需要配置成此模式）

我在实验中主要使用AP模式。

### 实践

目的：通过串口配置esp8266，开启AP模式并且启用TCP服务器。使得其它设备能通过加入热点建立wifi通信，并且建立tcp连接，实现一个简单的回响，即服务端返回客户端发送的消息。

在STM32cubeMX中配置相关引脚。

![https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/ef0cdf8727cfaafe1445242d14cb9317.png](https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/ef0cdf8727cfaafe1445242d14cb9317.png)![](file:///C:\Users\翻白肚~1\AppData\Local\Temp\ksohtml24360\wps9.jpg)

本次实验用到ESP-12F 模块所占用的两个USART串口。

启用DMA接收以及USART全局中断。

![https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/a9d22159e9d78a05daf100b3be788855.png](https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/a9d22159e9d78a05daf100b3be788855.png)

![https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/04f47b8459c861b1a0bd4513de46697a.png](https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/04f47b8459c861b1a0bd4513de46697a.png)

编写wifi.c，设计wifi初始化函数，原理即逐条使用AT指令配置通信模块。
[*附官方指令集文档*]([https://](https://espressif-docs.readthedocs-hosted.com/projects/esp-at/zh-cn/release-v2.2.0.0_esp8266/AT_Command_Set/Wi-Fi_AT_Commands.html))

![https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/ec80e7918c7d8de81a76227cd55af3e0.png](https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/ec80e7918c7d8de81a76227cd55af3e0.png)

这里的Send_command函数基于HAL_UART_Transmit函数稍作改动。

此外提供了Send_message函数，用于发送指令让模块向客户端发送消息。

![https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/d787d33b42fb899ae8fb354acc316796.png](https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/d787d33b42fb899ae8fb354acc316796.png)

这里需要注意每条指令后的“\r\n”都是必要的，这是模块判断一条指令结束的依据。

main函数中调用set_wifi对8266模块进行初始化。

![https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/f4144a2d919ed2428dd8e56df8c10742.png](https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/f4144a2d919ed2428dd8e56df8c10742.png)

这里需要注意，通过SetPriority指令在主循环之前、串口初始化之后重新配置了两种中断的优先级。

![https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/da99daf0bfc5307af192e1c645875105.png](https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/da99daf0bfc5307af192e1c645875105.png)

可以看到初始化时SysTick中断的抢占优先级是0（数字越小优先级越高），而我们必须把USART中断和DMA中断的优先级设置的更高，这是因为我们在DMA空闲中断回调中使用了Send_message函数，其内部调用了HAL_Delay函数，而这个函数的实现是依赖于SysTick中断的，其具体方法是进入一个计数器判断死循环，等待每个Tick都会产生一次的中断，把计数器减一，直到计数器为0退出循环。如果SysTick中断的优先级低于USART中断，就会产生问题：发生空闲中断后程序进入USART中断回调函数，此时在中断内部遇到HAL_Delay后进入计数循环，但是由于SysTick中断的优先级较低，在中断内部无法进入SysTick中断（也就没法让计时衰减），导致计数器死循环无法退出。

最初没有这方面认识，中断后程序卡死，单步调试发现卡在了HAL_Delay，百思不得其解。饱受折磨后查到相关资料总结出上述教训。

最后是中断回调的编写。这里采用DMA空闲中断，其原理与功能不再赘述。


为了实现回响，一定要判断返回消息的关键字，比如+IPD代表收到来自客户端的消息，将其解析后用AT指令命令模块发回给客户端即可。由于模块会对每条指令都做出答复，比如“OK”，所以单纯的一进入回调（即收到完整消息发生了中断）就发送指令会导致无意义的高频循环，即发指令->回应OK->收到消息进入中断->发指令。。。因此必须解析响应，针对特定响应发出指令。

最初没有考虑到这一点，想先做个简单的收发测试，就遇到了这个问题，手机连接客户端后就疯狂收到消息，由于没有其它串口作测试，也困惑了很久。

下面是手机连接AP热点后，使用小工具建立TCP连接并发送消息的测试。

连接后收到hello，之后回响发送过去的消息。

![https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/71aeb113dcaf799fb2164590c4fdac0c.png](https://hexyl-1308974693.cos.ap-shanghai.myqcloud.com/imgs/71aeb113dcaf799fb2164590c4fdac0c.png)

