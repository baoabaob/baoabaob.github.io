---
title: asm图中的条件输出框和状态框
date: 2022-12-05 21:24:00
tags: 数字逻辑
---

在绘制asm图时,某些情况下我们必须使用条件输出图框。

比如下图

<img src="https://cccake-bucket1.oss-cn-beijing.aliyuncs.com/imgs/202212052127704.png" alt="image-20221205212708637" style="zoom:50%;" />

我们要求在ADD指令之后，以co为条件，把加法器的结果打入RA或RB，那么在条件框之后必须跳转到两个条件输出框LA和LB，保证LA或LB与ADD在同一周期内执行。

用状态输出框是因为这里LA和LB打入数据是与加法器的输出有关的，因此必须和ADD在同一个周期内（否则加法器没有输出）；但是如果只是根据信号跳转状态，而前后状态下的数据不相互影响就不必在同一个周期。

比如下图，A>B之后就无所谓接状态框或条件输出框。

<img src="https://cccake-bucket1.oss-cn-beijing.aliyuncs.com/imgs/202212052132815.png" alt="image-20221205213215767" style="zoom:50%;" />



