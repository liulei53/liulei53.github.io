---
layout: post
title: "homeassistant集成系列之zigbee wifi开关"
date: 2025-02-07
summary: 今天更新了一下Home Assistant那篇blog，但是GitHub给我报错了，简单记录一下排障过程的学习过程。
categories: [学习分享]
tags: [学习]
---





这篇教程是将zigbee wifi开关集成进homeassistant的过程简单记录一下。
## zigbee wifi开关
这是今天的主角，淘宝上这种插座很多，价格在10-50元不等。这种插座里面应该有一个由网络信号控制的断路器。远程控制开关非常灵活，有很多使用场景，比如：
1. 连接在电脑上，电脑上设置通电自动启动，这样就可以远程控制电脑的开关机;
2. 连接在热水器上，这样热水器不需要24小时待机，可以跟随联动控制开启，如果接入AI,甚至可以记录主人的洗澡习惯，提前打开热水器；
3. 连接在鱼缸配件上、宠物电器上；
4. 连接在充电器上，防止电动车、充电设备长时间连接电源。

<img src="https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250210092822983.jpeg"/>

## 连接步骤
### （1）下载涂鸦智能APP,添加wifi开关
- 打开涂鸦智能APP,添加设备
- 根据APP提示，连接2.5Gwifi,给开关断电然后重新插入电源，按住开关键5秒中，会提示手机连接开关的wifi热点
- 连接好wifi热点回到APP内，开关就连接好了
涂鸦智能APP中的开关功能有开/关、倒计时、定时，三项功能。这里提一下，因为继承到homeassistant中，就会只剩下一个开/关功能。
### （2）在homeassistant添加tuya集成
登录homeassistant web UI,设置——添加继承——搜索tuya,根据提示一步一步关联tuya账户。用户代码需要在手机端涂鸦智能app：ps,设置中查询到。
设置好之后，wifi开关就集成进来了。添加实体的时候搜索一下wifi,这个开关就能搜索到，然后添加进来。正如上文所说，只剩下一个开/关功能。
### （3）接入APPLE的homekit
这一步非常简单，在homeassistant web UI中，找到homekit插件，点击旁边的三点，接着点击重新载入。在iphone上的“家庭”APP中，就可以看到这个开关了，也如上文所说，只有一个开关功能。

好啦，这就是homeassistant集成zigbee协议的wifi开关的过程啦，是不是很简单。

