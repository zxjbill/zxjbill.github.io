---
layout: post
title:  "oatpp 搭建网页服务器"
date:   2020-10-06 18:53:00
categories: oatpp
tags: 服务器后台
excerpt_separator: <!--more-->
---

* content
{:toc}

利用 Oat++(oatpp) 在 Linux 上部署网站的服务器程序。目前阶段利用到基本的网络框架，根据浏览器的 request 返回相应的内容。
<!--more-->

以前将个人博客利用 jekyll 部署在 github 上，一直想尝试部署在自己的服务器上供用户访问。正好最近看到一个开源库 oatpp 想学习以下，所以看了一两个周，从文档中给的 example 来看可以实现服务器的搭建，所以就尝试利用 oatpp 搭建网站服务器程序。

## oatpp 介绍
* Oat++(oatpp) 是一个开源的 C++ web 框架，具有易于扩展和高效资源利用的特点。Oat++ is an open-source C++ web framework for highly scalable and resourc-efficient web applications.
* 主要提供以下功能：
  * Advanced REST framework (representational state transfer) 表征状态转移框架。利用请求参数映射 (request parameters mapping) 和 Swagger-UI annotations 来实现。这也是搭建简单网站服务器锁需要的基本功能。
  * WebSocket framework。
  * Object Mapping，对象映射。
  * Dependency Injection 依赖注入。
  * Swagger-UI。

## 搭建结构
* 利用异步 Api 搭建框架。

```bash
|- front/                           # Front-end sources
|- cert/                            # TLS certificate
|- serve/                           # Server sources
    |- cmake                        # Find the LibressL encryption library.
    |- CMakeLists.txt               # server projects CMakeLists.txt
    |- src/
        |- controller/              # Folder containing UserController where all endpoints are declared
            |- StaticController.hpp # all endpoints are declared in the file
        |- dto/                     # DTOs are declared here
            |- Config.hpp           
        |- utils/
            |- ServerUtils.hpp      # Some helper for server.
        |- AppComponent.hpp         # Service config
        |- App.cpp                  # main()
```

其中在 CMakeLists.txt 中间中要定义 front source 和 tsl 的文件夹位置变量 FRONT_PATH 和 CERT_PEM_PATH、CERT_CRT_PATH。

具体代码整理后放到 github 仓库中。

## 注意点
* ENDPOINT_ASYNC 声明时，特例要靠前，普遍性模板靠后，这样会先匹配特例的路径，未成功会继续匹配普遍性的 Endpoint。
* Endpoint 的路径设置和前端文件路径是相关的，可以通过大括号 `{}` 进行路径匹配。
* url 中如果有 Unicode 会被浏览器映射为 UTF-8 编码(至少 Chrome 会)，因此进入到 Endpoint 处理时需要处理去掉 `%`，并且将字符串表示的十六进制数转换成 `char` 进行解码得到正确的 Unicode 字符串。
* 可将被访问过的文件保存到静态变量中，避免每次读取。但是这也要注意映射关系 `unordered_map<oatpp::String, oatpp::String>` 的处理, 其中 `oatpp::String` 定义了 `hash` 可以作为键值。