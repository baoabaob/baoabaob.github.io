---
title: linux自定义系统服务
date: 2024-03-03 16:53:52
categories:
- 技术
tags:
- linux
updated: 2024-03-03 16:53:52
---

kook越来越烂了，于是我找到了TeamSpeak这个软件，一通捣鼓部署在服务器上了。

<img src="https://cccake-bucket1.oss-cn-beijing.aliyuncs.com/imgs/202403032147198.png" alt="image-20240303214719689" style="zoom:67%;" />

不仅纯免费、音质佳，还能和朋友们一起听歌♪(´▽｀)

不过装好了以后，想启动需要找到其路径，每次服务器重启还要重新启动。我希望它能够像nginx之类的应用一样可以使用systemctl来方便地管理。

那么怎么把自定义的软件加入系统服务呢？

1. 在`/usr/lib/systemd/system`目录下添加服务对应的配置文件：

   ` vi /usr/lib/systemd/system/服务名称.service`

2. 最基本的配置格式：

   ```ini
   [Unit]
   Description=服务描述
   After=服务依赖（再这些服务后启动本服务）
   
   [Service]
   Type=服务类型
   ExecStart=启动命令
   ExecStop=终止命令
   ExecReload=重启命令
   
   [Install]
   WantedBy=服务安装设置
   ```

下面以对TeamSpeak的配置为例子，首先创建了ts3.service，然后输入内容：

```ini
[Unit]
Description=Teamspeak server
After=network.target
[Service]
WorkingDirectory=/home/teamspeak/teamspeak3
User=teamspeak
Group=teamspeak
Type=forking
ExecStart=/home/teamspeak/teamspeak3/ts3server_startscript.sh start 
ExecStop=/home/teamspeak/teamspeak3/ts3server_startscript.sh stop
RestartSec=15
Restart=always
[Install]
WantedBy=multi-user.target
```

其中`Restart`定义何种情况 Systemd 会自动重启当前服务，可能的值包括`always`（总是重启）、`on-success`、`on-failure`、`on-abnormal`、`on-abort`、`on-watchdog`。

`type`定义启动时的进程行为，它有以下几种值：

- `Type=simple`：默认值，执行ExecStart指定的命令，启动主进程

- `Type=forking`：以 fork 方式从父进程创建子进程，创建后父进程会立即退出

- `Type=oneshot`：一次性进程，Systemd 会等当前服务退出，再继续往下执行

- `Type=dbus`：当前服务通过D-Bus启动

- `Type=notify`：当前服务启动完毕，会通知Systemd，再继续往下执行

- `Type=idle`：若有其他任务执行完毕，当前服务才会运行

配置完毕后，就可以用熟悉的systemctl来启动或配置ts了：

启动服务端

```bash
systemctl start ts3
```

关闭服务端

```bash
systemctl stop ts3
```

开机自启

```bash
systemctl enable ts3
```

查看服务端运行信息

```bash
systemctl status ts3
```

