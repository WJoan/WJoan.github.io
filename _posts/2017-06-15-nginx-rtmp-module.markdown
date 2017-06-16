---
layout:     post
title:      "搭建 RTMP 直播服务器"
subtitle:   "ubuntu下nginx-rtmp-module的使用"
date:       2017-06-15 16:38:26
author:     "Joan"
tags:		["web开发"]
---

# 整体架构

### 1. 思路

<img src="{{site.baseurl}}/img/rtmp/nginx_rtmp_ffmpeg_0.png">

**思路**：nginx通过rtmp模块（即nginx-rtmp-module）提供rtmp服务，ffmpeg推送一个rtmp流到nginx，然后客户端通过访问nginx来收看实时视频流。HLS也是差不多的原理，只是最终客户端是通过HTTP协议来访问的,但是ffmpeg推送流仍然是rtmp的。

### 2. 文章结构

- 搭建 nginx 服务器
- 配置 nginx 的 rtmp 模块
- 使用 ffmpeg 模拟推流
- 直播测试

# 搭建 nginx 服务器

### 1. 从github下载nginx-rtmp-module

```
git clone https://github.com/arut/nginx-rtmp-module.git
```

如果没有安装git,直接在Github上下载并解压就可以了

### 2. 下载nginx压缩包并解压

```
http://nginx.org/en/download.html
```

<img src="{{site.baseurl}}/img/rtmp/nginx_rtmp_ffmpeg_2.png">

### 3. 解压nginx-1.11.1.tar.gz并进入查看

```
tar -xvf nginx-1.11.1.tar.gz
cd nginx-1.11.1
```

### 4. 配置nginx

```
./configure --prefix=/usr/local/nginx --add-module=/path/to/nginx-rtmp-module --with-http_ssl_module --with-debug
```

<img src="{{site.baseurl}}/img/rtmp/nginx_rtmp_ffmpeg_4.png">

`--prefix=PATH` 指定nginx的安装目录，默认 `/usr/local/nginx`

`--with-http_ssl_module` 使用https协议模块，前提是 openssl 与 openssl-devel 已安装

`--add-module=PATH` 第三方外部模块路径，如 `nginx-rtmp-module` 或缓存模块。每次添加新的模块都要重新编译。

如果安装失败，请检查系统是否有PCRE、OpenSSL、zlib 等library

【注意】安装PCRE2是不行的 [PCRE 安装教程](http://chenzhou123520.iteye.com/blog/1817563)

### 5. 编译

```
make
make install
```

<img src="{{site.baseurl}}/img/rtmp/nginx_rtmp_ffmpeg_7.png">
<img src="{{site.baseurl}}/img/rtmp/nginx_rtmp_ffmpeg_8.png">

### 6. 修改nginx.conf

```
cd /usr/local/nginx/conf
vim nginx.conf
```

这里根据需要对http的端口号进行修改。

### 7. 启动nginx

```
cd /usr/local/nginx/sbin
./nginx
```

### 8. 检查端口是否占用

```
netstat -ntlp
```

最后在浏览器输入地址，如localhost:8080，看是否能成功进入nginx的欢迎页面
<img src="{{site.baseurl}}/img/rtmp/nginx_rtmp_ffmpeg_13.png">


参考链接:
(搭建nginx-rtmp直播服务器，ffmpeg模拟推流)[http://cxuef.github.io/linux/%E3%80%90%E7%BD%AE%E9%A1%B6%E3%80%91%E6%90%AD%E5%BB%BAnginx-rtmp%E7%9B%B4%E6%92%AD%E6%9C%8D%E5%8A%A1%E5%99%A8%EF%BC%8Cffmpeg%E6%A8%A1%E6%8B%9F%E6%8E%A8%E6%B5%81/]
(nginx服务器安装及配置文件详解)[https://segmentfault.com/a/1190000002797601]

# nginx-rtmp-module的配置

在 nginx-rtmp-module/test 中包含了一个简单 RTMP 的配置以及前端的测试页面。

### 1. 修改配置文件

把 nginx-rtmp-module/test/nginx.conf 配置文件中的内容替换到 nginx 服务器配置文件中。

```
rtmp {
    server {
        listen 1935;

        application myapp {
            live on;
        }
    }
}
```

这是一个简单的 rtmp 服务设置，其中 `1935` 是 rtmp 协议的默认端口号，如果在这里将端口号更改为其他，那么在接下来访问 rtmp 地址时都要包含修改后的端口号，如果不加端口号将自动使用默认的端口号`1935`。

【注意】由于服务器1935端口被占用，因此我将端口号改为了1112

```
http {
    server {
        listen      8080;

        location /stat {
            rtmp_stat all;
            rtmp_stat_stylesheet stat.xsl;
        }

        location /stat.xsl {
            root /path/to/nginx-rtmp-module/;
        }

        location /control {
            rtmp_control all;
        }

        location /rtmp-publisher {
            root /path/to/nginx-rtmp-module/test;
        }

        location / {
            root /path/to/nginx-rtmp-module/test/www;
        }
    }
}
```

该部分定义了一些 http 服务访问路径，其中 `root /path/to/nginx-rtmp-module/` 中的路径都要改为安装 nginx-rtmp-module 时的路径。

### 2. 修改前端播放器

 RTMP 模块有四个测试用例，分别位于以下路径：

- nginx-rtmp-module/test/www/index.html
- nginx-rtmp-module/test/www/record.html
- nginx-rtmp-module/test/rtmp-publisher/player.html
- nginx-rtmp-module/test/rtmp-publisher/publisher.html

```
var flashVars = {
        streamer: 'rtmp://localhost/myapp',
        file:'mystream'
    };
```

以 player.html 为例，`streamer` 是直播流的 URL，由于我之前把 RTMP 端口号改成了非默认端口 `1112`，并且我是从内网其他的机器去访问服务器，因此这里的 URL 就需要更改为 `rtmp://192.168.1.10/myapp:1112`。其他的几个页面同样需要类似的修改。如果使用默认端口号并且在本机上访问，那么就不需要更改前端文件了。

# ffmpeg模拟推流

### 1. 配置ffmpeg

(ffmpeg处理RTMP流媒体的命令大全)[http://blog.csdn.net/leixiaohua1020/article/details/12029543]

# 直播测试

### 测试用例

RTMP 端口: 1935, HTTP 端口: 8080

|测试URL				|应用							   |
|-----------------------|----------------------------------|
|http://localhost:8080/ | play myapp/mystream with JWPlayer|
|http://localhost:8080/record.html | capture myapp/mystream from webcam with old JWPlayer|
| http://localhost:8080/rtmp-publisher/player.html | play myapp/mystream with the test flash applet|
| http://localhost:8080/rtmp-publisher/publisher.html | capture myapp/mystream with the test flash applet|

【注意】由于服务器8080端口被 Tomcat 占用，因此我将端口号改为了1111