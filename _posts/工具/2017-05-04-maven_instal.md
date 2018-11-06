---
layout: post
title: "maven安装"
date: 2017-05-04
author: silly
desc: "meven安装"
categories: ["工具"]
tags: ["maven"]
permalink: "/tool/maven-install.html"
--- 

下载带二进制源码包，解压

将bin设置为环境变量

加速器，修改conf文件夹下的`settings.xml`文件，添加如下镜像配置：


```
 <mirrors>
    <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
  </mirrors>
```
