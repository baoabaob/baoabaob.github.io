---
abbrlink: ''
categories: []
date: '2024-02-29T10:19:29.932321+08:00'
tags:
- 笔记
- 后端
title: web后端开发技术
updated: '2024-02-29T11:25:18.317+08:00'
---
王老师的课，做一个简单的笔记。

---

### 第一章

- PV（Page View）
  一个统计周期内（一般是一天）页面访问量。
- QPS（Query Per Second）
  即每秒处理请求数，最大吞吐能力。
  QPS = 并发请求数/平均响应时间，QPS到达系统上限后，响应时间随并发数增加快速增长。

  ![](https://cccake-bucket1.oss-cn-beijing.aliyuncs.com/imgs/8f8499b52ad04e9a8905078e82a455da.png)

  如果流量平稳：日均QPS= 每日PV / (3600 24)
  一般情况下，假定每天80%的访问集中在20%的时间，则峰值QPS = (每日PV * 80%)/(360024*20%)
- CDN（content delivery network）
  基本原理是缓存，将静态文件等变化较慢的数据放置在离用户较近的边缘服务器中，加速用户访问速度，减轻后端服务器压力。常搭配动静分离技术。
