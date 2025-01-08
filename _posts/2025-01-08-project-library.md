---
layout: post
title: "搭建开源项目Calibre个人图书馆"
date: 2025-01-08
summary: 因为有朋友总跟我要电子书资源，索性搭建个电子图书馆，把电子书共享一下。
categories: [学习分享]
tags: [学习]
---

## 一、Calibre和Calibre-web

## Calibre:

- 功能完整的桌面应用程序
- 重点在于电子书的管理和处理
- 独立运行的本地软件

## Calibre-web：

- 基于web的在线图书馆系统
- 重点在于图书的展示和阅读
- 需要服务器部署的网页应用

**项目地址：**<https://book.liulei.org> [如有侵权请联系我删除，抱歉。](liulei53@icloud.com)

<img src="https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250108203030127.png"/>

## 知识点

1. docker技术
2. nginx反向代理
3. 域名解析
4. 介绍了SSL和一款开源SSL证书获取工具
5. Linux基础命令（mkdir、chown、chmod）

特别鸣谢：CSDN博主：木有会， 这是他的[详细教程](https://blog.csdn.net/mudarn/article/details/144267291?spm=1001.2101.3001.6650.4&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EYuanLiJiHua%7EPosition-4-144267291-blog-100166473.235%5Ev43%5Epc_blog_bottom_relevance_base3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EYuanLiJiHua%7EPosition-4-144267291-blog-100166473.235%5Ev43%5Epc_blog_bottom_relevance_base3&utm_relevant_index=9)。

## 搭建过程

**1.创建必要的目录和授权**

`mkdir -p`：创建目录的命令，这个命令非常基本，对标windows系统的新建文件夹即可。

`chown -R`：更改文件所有者和更改文件所属组，主要用于文件的访问控制。

`chmod -R xxx`：用于修改文件或目录权限的命令，Linux的权限系统通过三类用户（所有者U、组G、其他用户O）以及三种权限（读read、写write、执行execute）来控制对文件和目录的访问。以下是常用的几种权限组合：

**权限字符串**	**八进制表示**	**含义**

---------	 000	所有人都没有任何权限。

r--r--r--	444	所有人只能读取文件。

rw-r--r--   644	所有者可以读写，组和其他用户只能读取。

rwx------   700	所有者有所有权限，组和其他用户没有任何权限。

rwxr-xr-x  755	所有者有所有权限，组和其他用户有读和执行权限。

rw-rw----   660	所有者和组有读写权限，其他用户无权限。

**2.创建doker-compose.yml文件**

docker-compose.yml 文件是一个集中式的配置文件：

​	1.	**统一配置**：

所有服务的配置都集中在一个文件中，避免了分散配置的复杂性。

​	2.	**默认路径**：

docker-compose.yml 是 Docker Compose 的默认文件名，运行命令时会自动读取当前目录下的此文件。

​	3.	**自动化**：

docker-compose 会自动处理镜像构建、容器启动、网络配置和卷挂载等工作，简化了手动操作。

​	4.	**项目隔离**：

每个项目都有自己的 docker-compose.yml 文件，不同项目互不干扰。

**3.启动容器**

```bash
docker-compose up -d #启动容器

docker-compose logs -f #查看日志

docker-compose ps #查看容器状态

docker-compose restart #重启容器

docker-compose down #停止容器
```

**4.登录应用后配置**

在本地启动容器之后，服务就跑起来了。在服务器上可以使用`curl localhost:端口号`测试一下，如果返回的内容是html文件，就说明服务没有问题。如果有公网IP就可以用公网IP+端口号访问服务啦。噢对了，记得在服务器的安全组里面放行这个服务映射的端口。

配置也很简单，一般系统有一个默认的用户名密码。然后就是设置数据库的连接、配置文件存储的位置等等。可视化的界面配置起来相对简单一些。报错的点大多数目录权限不够，数据库用户权限不够，数据库用户名密码填错，等等。

**5.使用nginx做反向代理**

在nginx.conf文件中增加server块，填写反向代理的配置。本次的nginx代理中还添加了监听443端口的server块，用于把用户的http请求转向https请求，使用SSL加密。

**6.域名解析**

将服务器的公网IP解析到book.liulei.org

**7.SSL**

[什么是SSL？](https://info.support.huawei.com/info-finder/encyclopedia/zh/SSL.html)

SSL使用证书对通信双方之间建立的连接的两端进行身份认证，并使用证书对加密的通信信道进行协商，从而确保安全。

<img src="https://download.huawei.com/mdl/image/download?uuid=768fb79b6dbe48e1b0135b0ceb016e2a"/>

**8.Certbot**

**Certbot** 是一个开源工具，用于轻松获取和自动续订由 **Let’s Encrypt** 颁发的免费 SSL/TLS 证书。它简化了 HTTPS 配置的过程，使网站和应用程序能够安全地加密通信。

**Certbot 的主要特点**

​	1.	**自动化**：

​	•	Certbot 可以自动完成证书的申请、安装和续订。

​	•	支持大多数主流 Web 服务器（如 Apache 和 Nginx）的自动配置。

​	2.	**免费证书**：

​	•	使用 Let’s Encrypt 提供的免费证书，无需支付 SSL 费用。

​	3.	**跨平台支持**：

​	•	Certbot 支持多种操作系统，包括 Linux、macOS 和 Windows。

​	4.	**安全**：

​	•	提供 HTTPS 加密，确保数据在传输中的安全性。

​	5.	**灵活性**：

​	•	支持手动模式，适用于不常见的 Web 服务器或复杂环境。

**Certbot 的工作原理**

Certbot 使用 Let’s Encrypt 提供的 ACME 协议（Automatic Certificate Management Environment）来完成以下操作：

​	1.	向 Let’s Encrypt 服务器申请证书。

​	2.	验证域名的所有权（常见验证方式包括 HTTP 验证和 DNS 验证）。

​	3.	颁发证书并安装到 Web 服务器。

​	4.	定期自动续订证书（默认每 60 天续订一次，证书有效期为 90 天）。

SSL/TLS 证书和私钥的使用流程涉及多个步骤，从申请证书到配置 Web 服务器（如 Nginx 或 Apache），再到定期续期证书。以下是详细的使用流程：

---

### 1. **申请证书**
使用 `certbot` 或其他工具为您的域名申请 SSL/TLS 证书。例如：
```bash
sudo certbot certonly --standalone -d example.com
```
- 证书文件将保存在 `/etc/letsencrypt/live/example.com/` 目录下。
- 主要文件包括：
  - **证书文件**：`fullchain.pem`
  - **私钥文件**：`privkey.pem`

---

### 2. **配置 Web 服务器**
将证书和私钥配置到 Web 服务器（如 Nginx 或 Apache）中，以启用 HTTPS。

#### **Nginx 配置示例**
```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
        proxy_pass http://localhost:8080;  # 将请求转发到后端服务
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;  # 强制跳转到 HTTPS
}
```

#### **Apache 配置示例**
```apache
<VirtualHost *:443>
    ServerName example.com

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem

    ProxyPass / http://localhost:8080/
    ProxyPassReverse / http://localhost:8080/
</VirtualHost>

<VirtualHost *:80>
    ServerName example.com
    Redirect permanent / https://example.com/
</VirtualHost>
```

---

### 3. **重启 Web 服务器**
完成配置后，重启 Web 服务器以应用更改。

#### **Nginx**
```bash
sudo systemctl restart nginx
```

#### **Apache**
```bash
sudo systemctl restart apache2
```

---

### 4. **验证 HTTPS**
访问 `https://example.com`，确保网站可以正常加载，并且浏览器地址栏显示绿色锁图标。

---

### 5. **自动续期**
Let's Encrypt 证书有效期为 90 天，需要定期续期。`certbot` 提供了自动续期功能。

#### **测试续期**
运行以下命令，测试续期功能是否正常工作：
```bash
sudo certbot renew --dry-run
```

#### **设置定时任务**
1. 打开 Crontab 编辑器：
   ```bash
   sudo crontab -e
   ```
2. 添加以下行（每天凌晨 2 点检查并续期证书）：
   ```plaintext
   0 2 * * * /usr/bin/certbot renew --quiet
   ```

---

### 6. **处理续期后的 Web 服务器重启**
续期证书后，Web 服务器需要重新加载证书。可以通过以下方式实现：

#### **Nginx**
在 Crontab 中添加续期后重启 Nginx 的命令：
```plaintext
0 2 * * * /usr/bin/certbot renew --quiet && systemctl reload nginx
```

#### **Apache**
在 Crontab 中添加续期后重启 Apache 的命令：
```plaintext
0 2 * * * /usr/bin/certbot renew --quiet && systemctl reload apache2
```

---

### 7. **监控证书状态**
定期检查证书的状态，确保续期成功。可以通过以下命令查看证书的到期时间：
```bash
sudo certbot certificates
```
输出示例：
```plaintext
Found the following certs:
  Certificate Name: example.com
    Domains: example.com
    Expiry Date: 2025-04-08 12:34:56+00:00 (VALID: 89 days)
    Certificate Path: /etc/letsencrypt/live/example.com/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/example.com/privkey.pem
```

---

### 8. **备份证书和私钥**
定期备份证书和私钥文件，以防止数据丢失。可以将 `/etc/letsencrypt/live/` 和 `/etc/letsencrypt/archive/` 目录下的文件备份到安全的位置。

---

### 9. **处理证书续期失败**
如果证书续期失败，`certbot` 会发送电子邮件通知。您可以手动运行以下命令尝试续期：
```bash
sudo certbot renew
```
如果续期仍然失败，检查日志文件以获取更多信息：
```bash
sudo tail -f /var/log/letsencrypt/letsencrypt.log
```

## 踩的一个坑

必须提前下载metadata.db放在books目录下。问题是登录后选择配置文件保存的目录，一直提示DB存放的路径不合法。这个问题让我百思不得其解，浪费了几个小时的时间，一直以为是目录权限的问题。原来是还缺少这么一个数据库文件。
```bash
# 下载初始数据库文件
wget https://raw.githubusercontent.com/janeczku/calibre-web/master/library/metadata.db -O books/metadata.db

# 设置权限，所有者和所属组更改为 UID 和 GID 为 1000 的用户和组。
sudo chown 1000:1000 books/metadata.db

# 设置权限 644，即文件所有者可以读取和写入，所属组和其他用户只能读取。
sudo chmod 644 books/metadata.db
设置权限，所有者和所属组更改为 UID 和 GID 为 1000 的用户和组。
sudo chown 1000:1000 books/metadata.db
设置权限 644，即文件所有者可以读取和写入，所属组和其他用户只能读取。
sudo chmod 644 books/metadata.db
```
