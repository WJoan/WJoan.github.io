---
layout:     post
title:      "基于Java的网站开发环境配置"
date:       2017-05-10 15:48:30
author:     "Joan"
tags:		["web","java"]
---

## Java 环境配置

### 在官网上下载 jdk

需要注意选择正确的操作系统以及位数，以64位 windows 为例，下载得到 `jdk-8u101-windows-x64.exe`

### 安装 jdk

直接运行上面下载得到的可执行程序，默认安装即可。路径可以其他盘符，不建议路径包含中文名及特殊符号。如：D:\Program Files\Java\jdk1.8.0_101

### 配置 jdk 环境变量

1. 新建变量名：`JAVA_HOME`，变量值：D:\Program Files\Java\jdk1.8.0_101
2. 打开`PATH`，添加变量值：%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin
3. 新建变量名：`CLASSPATH`，变量值：.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar

### 测试 jdk
在CMD命令下输入javac，java，javadoc命令：出现图示界面，表示安装成功。
![jdk-config]({{ site.baseurl }}/img/jdk-config.png)

## Tomcat 配置
### 下载 Tomcat
在 http://tomcat.apache.org/ 上下载 `apache-tomcat-8.0.37.zip`，然后直接解压在C盘下（推荐）。

### 配置 Tomcat 环境变量
1. 新建变量名：`CATALINA_BASE`，变量值：C:\tomcat
2. 新建变量名：`CATALINA_HOME`，变量值：C:\tomcat
3. 打开PATH，添加变量值：%CATALINA_HOME%\lib;%CATALINA_HOME%\bin

### 启动 Tomcat 服务
打开 Tomcat 安装目录，再打开 bin 文件夹，找到 `startup.bat` 双击运行（linux下面使用 startup.sh）。启动后在浏览器中打开 `http://localhost:8080` 可以看到一个 Tomcat自带的 JSP 页面说明启动成功。