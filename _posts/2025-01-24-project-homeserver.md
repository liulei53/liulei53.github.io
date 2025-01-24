---
layout: post
title: "变废为宝-零成本！让下岗笔记本电脑“再就业”"
date: 2025-01-24
summary: 利用公网IP把家里的笔记本搭建成一台服务器的记述
categories: [学习分享]
tags: [学习]
---

# 一、简述步骤

1. 拥有一台下岗的笔记本电脑，以我的Thinkpad为例，配置：1.6GHz，四核心/八线程，4G内存，家里带宽300M，无限流量。这样的配置放在云上一年不得1000块？动手吧，立省1000元。
2. 跟网络运营商打电话，要一个公网IP。我家里是联通，打个10010转人工，就给设置了公网IP。
3. 给笔记本电脑选择一款服务器操作系统，我选择的是Ubuntu24桌面版，万一出差要用一下呢。
4. 把光猫设置为桥接模式，也就是说拨号上网的活儿不让它干了，运营商免费的光猫性能、功能都很low。
5. 打开路由器，设置内网转发和DDNS。

是不是很简单？我把这里面需要详细说的内容讲一下。

# 二、重点步骤

### 1. 确保网络环境
- **公网IP**：确认你的公网IP是固定的，如果是动态的，考虑使用DDNS服务。
- **路由器配置**：确保路由器支持端口转发，并允许外部访问。

### 2. 配置Ubuntu服务器
- **更新系统**：
  ```bash
  sudo apt update && sudo apt upgrade -y
  ```
- **安装必要软件**：
  ```bash
  sudo apt install openssh-server ufw
  ```
- **配置SSH**（可选）：
  编辑`/etc/ssh/sshd_config`，更改端口或禁用密码登录以提高安全性。
  ```bash
  sudo nano /etc/ssh/sshd_config
  ```
  修改后重启SSH服务：
  ```bash
  sudo systemctl restart ssh
  ```

- **配置防火墙**：
  允许SSH和其他必要端口：
  ```bash
  sudo ufw allow ssh
  sudo ufw allow 80/tcp  # HTTP
  sudo ufw allow 443/tcp # HTTPS
  sudo ufw enable
  ```

### 3. 配置路由器端口转发
- 登录路由器管理界面，找到“端口转发”或“NAT”设置。
- 添加规则，将外部端口（如80、443）转发到Ubuntu笔记本的内网IP和相应端口。

### 4. 域名解析
- **使用DDNS服务**（如果IP是动态的）：
  注册DDNS服务（如No-IP、DuckDNS），并在路由器或Ubuntu上配置DDNS客户端。
- **绑定域名**：
  在域名管理平台，将域名A记录指向你的公网IP。

*这里说明一下，联通的公网IP是动态分配的，也就是说需要使用DDNS服务把每次变动的IP地址解析到一个固定的域名上*

我家里使用的是华硕路由器，自带了域名注册的服务。

### 5. 安装Web服务器（可选）
- 安装Nginx或Apache：
  ```bash
  sudo apt install nginx
  ```
- 配置Nginx：
  编辑`/etc/nginx/sites-available/default`，设置服务器块：
  
  ```bash
  sudo nano /etc/nginx/sites-available/default
  ```
  示例配置：
  ```nginx
  server {
      listen 80;
      server_name yourdomain.com;
  
      location / {
          proxy_pass http://localhost:3000; # 转发到本地应用
      }
  }
  ```
  测试并重启Nginx：
  ```bash
  sudo nginx -t
  sudo systemctl restart nginx
  ```

### 6. 配置SSL（可选）
- 使用Let's Encrypt获取免费SSL证书：
  ```bash
  sudo apt install certbot python3-certbot-nginx
  sudo certbot --nginx -d yourdomain.com
  ```
  自动配置SSL并启用HTTPS。

### 7. 测试访问
- 在浏览器中输入`http://yourdomain.com`或`https://yourdomain.com`，确认能访问你的服务器。

# 三、服务器可以提供哪些常用的服务

## 1.web服务器

如果你有个人博客或者网站，可以部署在自己的服务器上，为自己或者身边的朋友提供服务。我自己的博客：<https://weekly.liulei.org>欢迎访问。

## 2.文件服务器

假如你经常出差，可以把文件放在服务器上，无论在哪里，只需要访问自己的服务器，就能把文件下载到本地使用。

假如你有多个终端，传输文件可能就会是一个非常头疼的事情，也可以通过文件服务器使用同一种协议进行传输。

## 3.媒体服务器

假如你喜欢看电影，可以在公司连上家里的服务器，设置好下载任务，到家使用电视就能连服务器看大片，内网流量跑完，看4K还是很爽的。在这里说明一下，家庭影院服务有更好的方案。

## 4. VPN服务器

这个要看个人水平了，搞不好很容易封IP。

## 5.虚拟化和自动化

可以使用轻量级的docker，部署一些好玩的项目，比如我自己的项目：

1. News now <https://hot.liulei.org>

2. 电子图书馆<https://book.liulei.org> 目前在AWS上，如果需要的人多了会考虑迁移到家里的服务器上，因为aws限流量。
3. 最近在写两个系统，也考虑放在服务器上玩一玩。

##  6.学习

可以学习各种技术，这个不必多说啦。