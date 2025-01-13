---
layout: post
title: "十分钟使用docker自建远程桌面访问服务器-Rustdesk"
date: 2025-01-13
summary: 手头有好几台电脑和终端散落在家里和公司，使用远程桌面软件把它们用起来。
categories: [学习分享]
tags: [学习]
---

# Rustdesk简介

1. 开源的远程访问和支持软件
2. 向日葵、TeamViewr的平替
3. 自建服务器安全可靠

# 软件功能

**远程访问**

对于普通用户而言，假如你有三四台电脑分别安装家里、公司、实验室不同的位置，而又不便携带。那么有一款这样的软件，可以你在任何一台电脑上，打开并控制其余电脑。甚至可以使用平板控制任意一台电脑。这种控制的方式和身临其境一样。市面上有这样的软件，但是服务器都在服务提供商那里，用户拥有的只是一个账号和权限。Rustdesk是一款服务器端和客户端软件。

**文件传输**

支持不同终端之间的文件传输。

**核心原理**

RustDesk的核心功能是建立远程连接，这涉及客户端和远程主机之间的通信。P2P连接，它通过 NAT 穿透技术（如 STUN、TURN、ICE）在两个设备之间建立直接连接，减少延迟。

**优点**

自建服务器，数据不经第三方，且使用端到端的加密。

*免费、免费、免费* 快乐的事情说三遍。

# 搭建步骤

## 前期准备

1. 云服务器（公网IP）
2. 安装docker
3. 各终端上下载客户端

## 安装过程

### 服务器端

使用docker部署，到[官方文档](https://rustdesk.com/docs/en/self-host/rustdesk-server-oss/docker/)中查看docker-compose配置文件。算了，我直接拿过来吧。

```bash
services:
  hbbs:
    container_name: hbbs
    image: rustdesk/rustdesk-server:latest
    environment:
      - ALWAYS_USE_RELAY=Y
    command: hbbs
    volumes:
      - ./data:/root
    network_mode: "host"

    depends_on:
      - hbbr
    restart: unless-stopped

  hbbr:
    container_name: hbbr
    image: rustdesk/rustdesk-server:latest
    command: hbbr
    volumes:
      - ./data:/root
    network_mode: "host"
    restart: unless-stopped

```

1. 打开服务器远程连接，找个目录，`mkdir rustdesk` 
2. 创建dokcer-compose.yml文件，`sudo vim docker-compose.yml`
3. 把上述配置文件直接粘贴进去，`:wq`
4. 运行配置文件，`sudo docker-compose up -d`

此时hbbs和hbbr两个容器应该都启动了，如果不放心，可以`netstat -tuln`看一下21115-21119端口是否listen监听。**此时rustdesk目录下多了一个data目录，后面有用**

5. 去云服务器的安全组，调整准入策略，放开21115-21119的tcp策略，21116需要一个udp策略。
6. telnet 公网IP:port 可以测试一下，策略是否生效。

到此为止服务器端的活就干完了。

### 客户端

到[github下载客户端](https://github.com/rustdesk/rustdesk/releases/tag/1.3.6)，支持windows、Ubuntu、Mac、安卓、IOS（我手机上已经用起来了）。打开客户端进行配置。首先会提示安装客户端服务并需要开放后台运行和屏幕捕获的权限。然后最关键的两个配置来了：

**ID/中继服务器**

ID填写公网IP地址，中继服务器也填写公网IP地址。

**KEY**

我安装的时候没有仔细看key在哪里，有的人使用的是docker pull拉取镜像安装的方式，在run的时候加上了k_ 这个参数，会生成key。其实我们使用docker-compose.yml文件直接启动，会在当前目录下，生成一个data目录，里面有一个.pub文件，就是公钥，也就是key。我遇到的问题是粘贴了一遍，在手机上测试提示说key不匹配，后来又重新粘贴了两遍就可以了。

## 使用

在手机上，打开rustdesk app，输入要访问设备的ID，再输入密码，就可以连接了。我在手机上测试了，使用体验还是不错的。晚上回家把家里的电脑都安装一下。

我原来一直使用的是**Splashtop**，这也是一款非常好的远程桌面软件，但是每年120元左右。这个价格几乎可以买台云服务器了。
