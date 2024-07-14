---
title: Datawhale AI 夏令营-机器学习笔记
date: 2024-07-14 18:27:53
updated: 2024-07-14 18:27:53
categories:
tags:
- 笔记
- AI
---

# Re0：从零开始的机器学习竞赛入门

前情提要：报名参加了第二期Datawhale 2024 年 AI 夏令营的机器学习方向，本篇作一个笔记的汇总。

## TASK1：五分钟跑通baseline 一站式体验真方便

首先在task1中明确这次学习的任务：基于讯飞开放平台的**电力需求预测挑战赛**，一边学习一边优化模型，提高预测分数。

### 背景说明

#### 1. 电力需求挑战赛

- 官方链接：https://challenge.xfyun.cn/topic/info?type=electricity-demand&ch=dw24_uGS8Gs

- 任务：给定多个房屋对应电力消耗历史N天的相关序列数据等信息，预测房屋对应电力的消耗。

- 数据格式：

  | 特征字段 | 字段描述               |
  | -------- | ---------------------- |
  | id       | 房屋id                 |
  | dt       | 日标识，[1,N]          |
  | type     | 房屋类型               |
  | target   | 实际电力消耗，预测目标 |

- 判准：预测数据与真实数据的均方差：

  ![img](https://openres.xfyun.cn/xfyundoc/2024-06-29/341fad9d-203f-409d-b118-a4ed9d411f8e/1719632410485/108.png)

#### 2. 飞桨AI Studio

- 官网：https://aistudio.baidu.com/

- 基于百度深度学习开源平台*飞桨*的人工智能学习与实训社区，为开发者提供了功能强大的线上训练环境、免费GPU算力及存储资源。

#### 3. BML codelab

- 百度的一站式AI开发平台

---

未完待续。。。
