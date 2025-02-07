---
layout: post
title: "HomeAssistant接入米家集成打通IOS的HomeKit"
date: 2025-02-06
summary: 小米开源了米家集成，用户可以在HomeAssistant中使用小米IoT智能设备。
categories: [学习分享]
tags: [学习]
---


### 一、安装Home Assistant

简单介绍：

**Home Assistant** 是一个由社区维护的开源项目，用于智能家居的集成和控制，支持包括小米设备在内的多种设备和平台。

项目地址：<https://github.com/home-assistant/home-assistant.io>

这个仓库包含 Home Assistant 的核心代码。如果你想了解更多关于 Home Assistant 的开发、贡献指南或者安装方法，可以在 GitHub 上查看 README 文档和官方 Wiki。

Home Assistant相关的开源项目，还有**HACS**和**Home Assistant Supervisor**，这个HACS是需要安装的，类似一个应用商店。

**HACS（Home Assistant Community Store）**：https://github.com/hacs/integration

**Home Assistant Supervisor**（适用于 Home Assistant OS 和 Supervised 安装）：https://github.com/home-assistant/supervisor

**Home Assistant Docker Hub**：https://hub.docker.com/u/homeassistant

*我采用的是在Ubuntu系统上使用docker安装Home Assistant，然后再安装HACS，重启容器之后，在HACS里面搜索Xiaomi Miot Auto 插件，安装这个插件，输入小米账号和密码就可以啦。下面我会写一下详细步骤。*

**安装详细步骤：**

### （1）安装docker和docker-compose

```bash
#更新系统
sudo apt update && sudo apt upgrade -y
#安装docker
sudo apt install -y docker.io
#启动并设置docker自启动
sudo systemctl enable --now docker
#安装docker compose
sudo apt install -y docker-compose

```

### （2）部署Home Assistant

```bash
#创建home assistant目录
mkdir -p ~/homeassistant && cd ~/homeassistant
#使用nano或其他编辑器创建docker-compose.yml文件
nano docker-compose.yml
#粘贴以下内容：
version: '3.7'
services:
  homeassistant:
    container_name: homeassistant
    image: "ghcr.io/home-assistant/home-assistant:stable"
    volumes:
      - ./config:/config
      - /etc/localtime:/etc/localtime:ro
    restart: unless-stopped
    privileged: true
    network_mode: host
#启动Home Assistant
docker-compose up -d
#访问
http://serverIP:8123
```

注意事项：

1. 防火墙开放8123端口
2. 如果是公网IP，需要在路由器中配置端口转发8123端口

## 二、在Home Assistant安装米家集成

项目地址：<https://github.com/XiaoMi/ha_xiaomi_home>

官方给出的安装方法一共有3种，如下图所示：

![image-20250206133507686](https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250206133513306.png)

*总结来说方法一是手动下载插件放在Home Assistant的目录下面，重启Home Assistant加载插件。方法二是通过HACS应用商店，通过图形化界面安装。我选择的是通过HACS安装。但是前提是安装HACS插件。*

**安装HACS**

```bash
#进入docker容器
docker exec -it homeassistant bash
#下载HACS
wget -O - https://get.hacs.xyz | bash -
#重启Home Assistant
exit  # 退出容器
docker restart homeassistant  # 重启 Home Assistant
```

进入home assistant的web界面，设置——设备与服务——添加集成——搜索HACS——按照提示完成GitHub授权（国内网络需要多试几次）——完成后HACS会出现在home assistant的侧边栏。如下图所示：

![image-20250206141432312](https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250206141432507.png)

注意事项：

1. 如果下载hacs的时候提示网络问题，需要改下DNS，echo "nameserver 8.8.8.8" > /etc/resolv.conf，再尝试wget下载。

**安装小米集成**

在HACS中搜索“xiaomi miot”，根据提示一步一步安装，者其中会让登录GitHub授权，需要有github账号。授权过程中可能会提示“could_not_register“，这是因为网络原因，如果服务器可以访问外网则不会提示。可以多试几次就可以完成。过程中会要求输入小米账号和密码。这一步完成之后，米家app里那些设备就会出现在homeassistant平台里面啦。

## 三、安装HomeKit

在web界面，设置——设备与服务——搜索homekit birage——安装。

通知里会有一条二维码配对的通知。如下图：

![image-20250206144029069](https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250206144029217.png)

使用iPhone的家庭app，扫码添加即可。

这样操作下来，可以在iPhone和ipad上远程操控家里的任何智能设备。



20250207更新

---

**界面美化-主题安装**

**（1）打开主题功能**

今天看着界面太丑了，想着更换一下主题，就这也遇到了些许小问题。

更换主题的步骤：点击右下角用户——浏览器设置——主题，此时我发现主题选项是灰色不可用状态。原因是需要在configuration.yaml文件里面打开这个功能。

```bash
#找到配置目录
cd /homeassistant/config/
#编辑configuration.yaml文件，编辑之前做好备份一下
cp configuration.yaml configuration.yaml.bak
vi configuration.yaml
#添加这两行代码，打开主题功能
frontend:
  themes: !include_dir_merge_named /config/www/community/
#重启homeassistant
docker restart homeassistant
```

回到web UI界面，再去更换主题，就可以啦。

**（2）下载安装主题**

我的Ubuntu server没有配置VPN，所以从GitHub下载主题很慢，甚至连接不上。于是我用Mac下载了zip包再传到ubuntu上去。主题的目录是～/homeassistant/config/www/community。下载好主题解压之后，放在这个目录下面就可以啦。PS：记得重启homeassistant。下图就是我安装的IOS-dark主题。

![image-20250207173206263](https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250207173311732.png)