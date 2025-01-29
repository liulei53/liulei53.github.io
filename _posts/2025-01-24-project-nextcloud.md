---
layout: post
title: "Nextcloud搭建个人私有云盘"
date: 2025-01-28
summary: 在Ubuntu上挂载一块1T硬盘，搭建家庭私有云盘，解决手机存储空间不足问题
categories: [学习分享]
tags: [学习]
---

前两天和朋友小聚，听说他要买1TB的手机，因为有很大的存储照片和视频的需求。生活中这样的需求应该很普遍，喜欢给孩子拍照、拍视频，并且想把照片视频存储到安全的地方等孩子大了还可以有一个美好的回忆。针对这样的需求，非常适合搭建一个**家庭NAS**。在针对存储的需求上它有这么几个优点：

1. 稳定性高，NAS一般是采用磁盘阵列，我给朋友推荐3块盘做一个raid5方案。如果有一块盘故障，可以直接换盘解决，数据不会丢失。
2. 安全性高，数据都在自己的硬盘，不经过任何第三方。另外相对手机来说，不易丢失、不易受外力破坏。
3. 其他优点：支持通过电脑、手机、平板跨设备访问；支持多用户并发访问；存储空间可扩展；可以远程访问，随时查看数据；节约成本，NAS的价格相比iPhone手机的存储来说要便宜的多；另外NAS还有媒体服务器、虚拟主机、备份中心等功能。

总得来说，NAS非常适合家庭用户或者小型团队使用。全家人每个人都可以分一个账号，把照片、视频集中存储到一个NAS系统中，随时可以翻看、调阅，节省手机的存储空间。

### NAS方案

NAS（网络附加存储）其实就是一个“联网的硬盘”。它是专门用来存储和共享文件的设备，可以让你在家里或公司通过网络随时随地访问里面的数据，就像你用网盘（比如百度网盘或 Google Drive）一样方便，但这些数据是完全属于你的，不存在隐私泄露的风险。简单说NAS就是：

​	•	**个人版云盘**：NAS就像是你家里的“私人百度网盘”，你可以把照片、视频、文档都存进去，通过电脑、手机、平板等随时访问。

​	•	**全家共享硬盘**：家里人都可以通过NAS存储或取用自己的文件，比如看电影、备份手机里的照片。

​	•	**小型服务器**：它还能干一些高端操作，比如下载电影、当媒体播放器、备份手机和电脑，甚至搭建个人网站。

### NAS方案选择

**成品NAS**

NAS其实就是硬件+软件的组合体，有一些厂商开发了专门针对NAS的操作系统，配合硬件进行销售。这种叫成品NAS。这样的NAS主流品牌有：

1. Synology群晖，操作系统DSM易用，软件生态丰富，适合家庭和小企业；
2. QNAP威联通，硬件性能强悍，适合进阶用户，支持虚拟机，Docker；
3. 绿联，系统UI简洁，适合新手，同样也支持docker和虚拟机，性价比高；
4. 极空间，影音功能强大，操作简单，硬件配置优秀。

**DIY方案**

这样的方案有很多，核心问题就是找一台稳定运行的设备搭载一个存储的磁盘阵列。

1. 使用普通PC或者旧笔记本改装NAS
2. 使用单板机（树莓派）构建NAS
3. 迷你电脑搭建NAS

### 搭建私人云盘

我选择的就是穷鬼方案1，使用家里的旧机笔记本搭载一块1T机械硬盘安装了nextcloud软件，做了一个私人云盘。

**操作系统选择：**专业NAS系统（TrueNAScore、OpenMediaVault）、通用Linux。我选择了Ubuntu，这样笔记本相当于一个两用设备，放在那里可以当服务器，拿起来可以移动办公。

**硬盘选择：**几年前买的一块希捷1T硬盘。其实空间也只剩300G，因为有500G格式化成了mac的存储空间。

**软件安装：**

1. openssl，方便远程访问服务器。
2. apache，web服务器，nextcloud的服务由它支持提供；
3. nextcloud，这就是主角了，下载下来解压放在/var/www/html目录下面，然后在nextcloud.conf配置虚拟主机。

```bash
<VirtualHost *:9090>
    ServerAdmin admin@yourdomain.com
    DocumentRoot /var/www/html/nextcloud
    ServerName simithbob.asuscomm.com
		#设置SSL证书
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/cert.pem
    SSLCertificateKeyFile /etc/ssl/private/key.pem

    <Directory /var/www/html/nextcloud/>
        Require all granted
        AllowOverride All
        Options FollowSymLinks MultiViews
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/nextcloud_error.log
    CustomLog ${APACHE_LOG_DIR}/nextcloud_access.log combined
</VirtualHost>
```

**踩过的坑：**

1. 上传的视频没有缩略图的问题

安装apt-get install ffmpeg、apt-get install ghostscript、在应用商店下载Preview Generator 应用

在/var/www/html/nextcloud/config/config.php文件里添加enabledPreviewProviders的配置后如下：

```bash
<?php
$CONFIG = array (
  'instanceid' => 'oc2pco9bf3ca',
  'passwordsalt' => '2MByx+fRRG6n8h2ww/ZnNcrAtESqgQ',
  'secret' => 't2XzNxsajT5fBiyXF1viEKm4yRSwWSjTYWKhSLWcwVZsg7Kt',
  'trusted_domains' =>
  array (
    0 => 'simithbob.asuscomm.com',
  ),
  'datadirectory' => '/var/www/html/nextcloud_storage/',
  'dbtype' => 'mysql',
  'version' => '30.0.5.1',
  'overwrite.cli.url' => 'http://simithbob.asuscomm.com',
  'dbname' => 'nextcloud',
  'dbhost' => 'localhost',
  'dbport' => '3306',
  'dbtableprefix' => 'oc_',
  'mysql.utf8mb4' => true,
  'dbuser' => 'nextclouduser',
  'dbpassword' => 'Nextcloud@123',
  'installed' => true,

'enabledPreviewProviders' => array (
  0 => 'OC\Preview\PNG',
  1 => 'OC\Preview\JPEG',
  2 => 'OC\Preview\GIF',
  3 => 'OC\Preview\HEIC',
  4 => 'OC\Preview\BMP',
  5 => 'OC\Preview\XBitmap',
  6 => 'OC\Preview\MP3',
  7 => 'OC\Preview\TXT',
  8 => 'OC\Preview\MarkDown',
  9 => 'OC\Preview\Movie',
  10 => 'OC\Preview\MP4'
),
);
```

工作原理是：previewproviders预览系统调用ffmpeg生成缩略图。

2. SSL证书

过程是生成公私钥对，生成CSR，提交给CA申请证书，CA颁发SSL证书，配置服务器使用证书，客户端和服务器建立TLS连接，使用对称加密传输数据。搞到证书以后，就在虚拟机配置里，把证书和key的路径填写进去，就ok了。443端口默认用户Https加密传输，https流量默认走443端口。所以一般在443端口配置证书加密，访问时直接输入域名即可访问。如果443端口被禁用，只能在监听的端口配置加密证书。用户在访问时需要手动输入端口号。

联通运营商禁用了443端口，所以我配置好以后，在内网环境测试都没有问题，但是换域名访问就是访问不了，最后才想到可能是443端口被禁用。使用telnet加端口号，发现也会connected，但是被远程服务器关闭。我还以为443端口没有被禁用，其实这是一种错误的想法，事实是443端口被网络运营商禁用了。使用端口扫描工具就可以得到验证。

<img src="https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250129162136188.HEIC"/>

**nextcloud的功能**

1. 自动上传手机的视频和照片
2. 上传文件，支持白板、cad、markdown文件、通讯录等等
3. 支持talk语音对讲
