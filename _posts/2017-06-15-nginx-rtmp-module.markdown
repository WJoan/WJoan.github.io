---
layout:     post
title:      "搭建 RTMP 直播服务器"
subtitle:   "nginx-rtmp-module的使用"
date:       2017-06-15 16:38:26
author:     "Joan"
tags:		["web开发"]
---

# 整体架构

### 1. 思路

<img src="{{site.baseurl}}/img/rtmp/nginx_rtmp_ffmpeg_0.png">

**思路**：nginx通过rtmp模块（即nginx-rtmp-module）提供rtmp服务，ffmpeg推送一个rtmp流到nginx，然后客户端通过访问nginx来收看实时视频流。HLS也是差不多的原理，只是最终客户端是通过HTTP协议来访问的,但是ffmpeg推送流仍然是rtmp的。

### 2. RTMP 协议

`实时消息协议（RTMP）` 最初是由Macromedia开发的专有协议，用于通过互联网，Flash播放器和服务器之间流式传输音频，视频和数据。Macromedia现在由Adobe拥有，该公司已经发布了用于公共使用的协议规范的不完整版本。

`RTMP` 协议是**应用层协议**，是要靠底层可靠的传输层协议（通常是 `TCP`）来保证信息传输的可靠性的。在基于传输层协议的链接建立完成后，RTMP 协议也要客户端和服务器通过**握手**来建立基于传输层连接之上的 RTMP 连接，在连接上会传输一些控制信息，如 `SetChunkSize` , `SetACKWindowSize` 。其中 CreateStream 命令会创建一个 Stream 连接，用于传输具体的音视频数据和控制这些信息传输的命令信息。

推流类型为 `live`、`record`、`append` 中的一种。`live` 表示该推流文件不会在服务器端存储；`record` 表示该推流的文件会在服务器应用程序下的子目录下保存以便后续播放，如果文件已经存在的话删除原来所有的内容重新写入；`append` 也会将推流数据保存在服务器端，如果文件不存在的话就会建立一个新文件写入，如果对应该流的文件已经存在的话保存原来的数据，在文件末尾接着写入。


### 3. 文章结构

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

## 启动、关闭
# ./sbin/nginx 
# ./sbin/nginx -s stop
## 重启，不会改变启动时指定的配置文件
# ./sbin/nginx -s reload
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

### 测试用例

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

### 3. 测试用例

由于更改了配置文件，测试前先重启一下服务器

```
cd /usr/local/nginx/sbin
./nginx -s reload
```

RTMP 端口: 1935, HTTP 端口: 8080

|测试URL				|应用							   |
|-----------------------|----------------------------------|
|http://localhost:8080/ | play myapp/mystream with JWPlayer|
|http://localhost:8080/record.html | capture myapp/mystream from webcam with old JWPlayer|
|http://localhost:8080/rtmp-publisher/player.html | play myapp/mystream with the test flash applet|
|http://localhost:8080/rtmp-publisher/publisher.html | capture myapp/mystream with the test flash applet|

以上为默认端口时，测试 URL 以及对应的应用。比如打开 player.html 就会看到如下的画面，由于还没有推送视频流所以只有一个播放器的框框。

<img src="{{site.baseurl}}/img/rtmp/player.png">
【注意】由于服务器8080端口被 Tomcat 占用，因此我将端口号改为了1111

# ffmpeg模拟推流

ffmpeg是一个非常快的视频和音频转换器，也可以从现场音频/视频源获取。它还可以在任意采样率之间进行转换，并通过高质量多相滤波器即时调整视频大小。

### 1. 配置ffmpeg

下载网址：http://www.ffmpeg.org/download.html

解压缩

```
tar -zxvf ffmpeg-2.0.1.tar.gz
```

编辑profile文件：

```
vim /etc/profile
#在文件末尾加上两句话：
export FFMPEG_HOME=/usr/local/ffmpeg 
export PATH=$FFMPEG_HOME/bin:$PATH
```

配置与安装

```
./configure --enable-shared --prefix=/usr/local/ffmpeg
make
make install
```

若出现error while loading shared libraries: libavdevice.so.52的错误
修改/etc/ld.so.conf 在最后一行加上/usr/local/ffmpeg/lib

```
ldconfig -v
```

并修改 /usr/local/ffmpeg/lib目录下的文件权限为777

(ffmpeg处理RTMP流媒体的命令大全)[http://blog.csdn.net/leixiaohua1020/article/details/12029543]

### 2. 主要参数

> 命令格式
ffmpeg [global_options] {[input_file_options] -i input_url} ... {[output_file_options] output_url} ...

`ffmpeg` 从 `-i` 选项指定的任意数量的输入“文件”（可以是常规文件，管道，网络流，抓取设备等）读取，并写入任意数量的输出“文件”，其中由一个纯输出url指定。在命令行上发现的任何不能被解释为选项的内容都被认为是一个输出url。不要混合输入和输出文件 - 首先指定所有输入文件，然后指定所有输出文件。也不要混合属于不同文件的选项。所有选项仅适用于下一个输入或输出文件，并在文件之间复位。

> -i url(input)
输入文件 URL。

> - f fmt (input/output)
强制输入或输出文件格式。通常会自动检测格式的输入文件，并从文件扩展名猜出输出文件，因此在大多数情况下不需要此选项。

> -re (input)
以原始帧速率读取输入。主要用于模拟抓取设备或实时输入流（例如，当从文件读取时）。不应与实际抓取设备或实时输入流（可能导致数据包丢失）一起使用。默认情况下，ffmpeg尝试尽可能快地读取输入。此选项将使输入读取速度降低到输入的本机帧速率。它对实时输出（例如直播）很有用。

# 直播测试

### 点播：使用 ffmpeg 推送视频流到 nginx-rtmp-module

从网上下载 .flv 视频文件，后通过 ffmpeg 推送视频流到网站后台

```
ffmpeg -re -i test.flv -f flv rtmp://192.168.1.10:1112/live/mystream
```

推送成功后台显示如果下：

<img src="{{site.baseurl}}/img/rtmp/ffmpeg.png">

打开网页 http://192.168.1.10:1111/rtmp-publisher/player.html 点击 `Play` 将看到如下效果：

<img src="{{site.baseurl}}/img/rtmp/live.png">

### 直播：通过 nginx-rtmp-module 实现网站视频直播 

打开网页 http://192.168.1.10:1111/rtmp-publisher/publisher.html 后允许网页使用摄像头和麦克风，然后点击 `Publish` 开始直播视频。

再次打开网页 http://192.168.1.10:1111/rtmp-publisher/player.html 就可以同步看到 publisher 页面中的内容啦。


参考链接：
[nginx-rtmp开发手册中文版](http://www.cnblogs.com/zx-admin/p/5783523.html)