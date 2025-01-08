---
layout: post
title: "搭建开源项目Newsnow的学习分享"
date: 2025-01-06
summary: 通过这个小项目，跟着小白从零学习docker、nginx反向代理、域名解析等基础知识。
categories: [学习分享]
tags: [学习]
---

**为了执行力**

大晚上刷X，看到一个非常不错的开源项目，名字叫Newsnow，名字就狠浪漫呀（其实是News now），遂打开看了看，嘿，功能也不赖，还支持docker部署。刚刚制定了**执行力**提升的计划，这不实践的机会就来了嘛。马上打开电脑开搞。

<table>
  <tr>
    <td><img src="https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250106095315104.png"/></td>
    <td><img src="https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250106095357496.png"/></td>
  </tr>
</table>
**项目预览**

经过一个小时的学习摸索，终于搭建成功啦。

项目地址：<http://hot.liulei.org/>

**搭建Newsnow项目学习过程**

搭建这个项目，需要用到的知识和工具：

1. 了解git和Github
2. 熟悉docker技术
3. 了解nginx技术
4. 熟悉域名解析
5. 拥有一台服务器，没有的话可以在我博客里看教程去撸一台
6. 拥有一个域名

#### (1) git和github

Git是一个免费、开源的分布式版本控制系统，它是一个软件，用来控制版本的变化。Github是一个开源软件代码托管平台，比如你和你的团队在做一个项目，可以通过github托管代码，用git来控制版本。

本项目里，在GitHub上找到这Newsnow项目，fork到自己的仓库。ssh到自己的服务器上，把项目clone到本地。`git clone 仓库地址`

#### (2) 使用docker把项目跑起来

项目克隆到本地之后，根据项目的readme所介绍的，进入项目根目录，执行：`docker compose up`

本项目只需要执行这么一行代码，就可以把项目跑起来。

从零基础到精通docker，这一篇文章就够了。<https://blog.csdn.net/leah126/article/details/131871717>

#### (3) 域名解析

在本地运行的项目，一般都是使用localhost+[端口号](https://baike.baidu.com/item/端口号/10883658)进行访问的，如果想通过互联网访问，可以使用服务器的公网IP地址+端口号进行访问。但是这样依然很麻烦，所以就需要把一个相对好记忆的域名解析到一个复杂难记的IP地址上。我是通过<https://www.cloudflare-cn.com>托管的DNS解析服务。解析后，只需要点击hot.liulei.org就可以访问公网IP+端口号所对应的服务啦。

其实到这里还不能真正访问，还需要到服务器安全组，在安全策略中的入站策略放行TCP协议的4444端口，出站策略一般是所有流量。然后在本地服务器上，修改iptables策略，确保外部地址都可以访问到本地服务，或者直接关闭firewalld服务（不推荐）。经过以上设置，**理论上**就可以访问服务器了。为什么说是理论上可以访问服务器呢，我们来看一下访问服务的整个流程。

1. 域名解析通过域名找到服务器，用户和服务器建立网络连接，默认的端口号是80，所以访问的是映射到80端口的服务。所以目前的设置不足以访问到4444端口的服务。
2. 问题就来了，如果想访问服务，难道只能把服务的端口改成80吗？这个时候nginx反向代理就应运而生了。

#### (4) nginx反向代理

Nginx 是一款轻量级的高性能 Web 服务器、反向代理服务器以及电子邮件（IMAP/POP3）代理服务器。一个`server`块定义了一个虚拟主机。

```nginx
server {  listen       80;  
  				server_name  example.com;  
  				location / {    
    				root   /var/www/html;    
    				index  index.html index.htm;  
  	} 
	}
```

这里我们用到的是nginx的反向代理功能。

```nginx
server {
  listen       80;
  server_name  frontend.example.com;
  location / {
    proxy_pass http://backend.example.com;
    proxy_set_header Host $host;
    proxy_set_header X - Real - IP $remote_addr;
    proxy_set_header X - Forwarded - For $proxy_add_x_forwarded_for;
  }
}
```

这样，当客户端访问`frontend.example.com`时，Nginx会将请求转发到`backend.example.com`，并对请求头进行适当设置，使得后端服务器能够正确处理请求，就好像请求是直接来自客户端一样。假如我们有多个项目部署在同一台服务器上，就可以使用一个server块监听80端口，接受外部请求，根据server_name来区分不同的域名，通过proxy_pass将请求转发到对应的后端服务端口。Nginx还有其他功能，比如负载均衡、安全过滤等，🔗[学习链接](https://blog.csdn.net/2301_76966984/article/details/144075607)

**部署过程中遇到的问题**

在cloudflare中添加了DNS解析记录之后，始终无法访问，通过dig命令检查发现，解析的目标IP地址是错误的。但是设置是没有错的。通过AI了解，发现是代理缓存的问题，解决方案是尝试清楚cloudflare缓存，或者暂时关闭cloudflare代理（设置为DNS only模式），测试后解析正常。
