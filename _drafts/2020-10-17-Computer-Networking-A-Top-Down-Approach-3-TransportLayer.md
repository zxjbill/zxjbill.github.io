---
layout: post
title:  "计算机网络自顶向下方法-第3章运输层"
date:   2020-07-28 23:59:59
categories: 计算机网络
tags: 计算机网络 运输层 自顶向下方法
mathjax: false
mermaid: true
excerpt_separator: <!--more-->
---

* content
{:toc}

## 引言
解决两个问题
* 运输层和网络层的关系。从两个端系统之间的交付服务到运行在两个不同端系统上的应用进程之间的交付服务 （通过 UDP 讲解）和 如何在会丢失不可靠的媒体上实现可靠通信 （TCP 运输协议）。
* 控制运输层实体的传输速率以避免网络拥塞，或从拥塞中恢复过来。