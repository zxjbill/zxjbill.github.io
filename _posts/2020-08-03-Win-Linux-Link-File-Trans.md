---
layout: post
title:  "Windows和Linux服务器的连接和文件传输"
date:   2020-08-03 23:59:59
categories: Linux
tags: Linux 文件传输 PuTTY 
mathjax: false
mermaid: false
excerpt_separator: <!--more-->
---

* content
{:toc}
本篇主要关于 Windows 和 Linux 连接和文件传输上的一些解决方案，几乎是使用 PuTTY 的套件，是目前用起来比较顺手的。配置的时候花了一些功夫研究了一下，所以在此做下记录。

最近学习操作系统，找的资料无论如何都避不开 Linux 或者 Unix，自己在办公室使用的电脑也还需要继续用 windows 系统。所以用寝室的旧电脑装了个双系统，其中一个作为远程 Linux 服务器用于一些练习。使用服务器，不可避免需要远程连接和文件传输。
<!--more-->

## 远程连接
* 服务器端电脑是 wifi 连接，没有公网 IP，分配的公网 IP 地址给路由器了。
* 步骤如下：
  * 本地路由器页面设置端口映射。登录路由器，找到相应服务，设置出入端口均 22，转发地址是服务器的在局域网的 IP 地址。
  * PuTTY 中设置 IP address 为路由器的 IP 地址，端口 22，连接方式可选选 SSH。可在 Data 中记录登录账户等信息。
  * 可保存为一个 sessions。
  * 连接后输入对应账户密码即可。
* 连接后就如同直接使用终端。

## ssh公钥私钥配置免密登录
* 在客户端主机用 puttygen.exe 生成相应的公钥私钥，私钥类型设置 RSA, Number of bits 可设置 2048，generate 后分别保存公钥和私钥。私钥最好设置使用密码。
* 修改公钥文件为一行，密钥部分不改，头部添加 ssh-rsa, 尾部可一个邮箱地址。(仿 Linux 生成的密钥格式， 不一定是不必须的)
* 公钥配置：
  * 通过 psftp.exe 将公钥文件传送到服务器端。
  * 服务器端被登录账户的根目录下建文件夹，并修改权限。(如果没有 .ssh 文件夹)

  ```bash
  mkdir .ssh          # 建文件夹 
  chmod 700 .ssh      # 修改文件夹权限为仅本人的账号与群组才能修改
  ```

  * 将刚才的公钥文件修改文件名并放入到 .ssh 文件夹中，并修改权限。
  
  ```bash
  cat *** >> .ssh/authorized_keys   # 修改文件夹
  chmod 644 .ssh/authorized_keys    # 修改文件权限
  ls -l .ssh                        # 查看权限
  ```
* 私钥配置：由两种方法，可以通过 pageant.exe 使用，另外可以在 auth 里设置。

## 文件传输
* 可在 windows 的 cmd 中使用 psftp 来远程传输文件
* 常用命令
  ```bash
  psftp sessions 名字                     # 建立链接
  # 建立连接之后，是在 psftp 的命令行模式下使用
  open IP 地址 端口                       # 打开链接
  quit                                   # 退出
  close                                  # 关闭连接
  help                                   # 查看帮助
  cd 文件夹                              # 改变远程目录
  pwd                                    # 显示当前目录
  lcd, lpwd                              # 对本地目录的操作
  get 文件名/-r 文件夹名 新文件夹名       # 复制远端到本地文件/文件夹
  put 文件名/-r 文件夹名 新文件夹名       # 复制本地到远端文件/文件夹
  mget/mput                              # 可使用通配符传输多个文件
  del, chmod, dir, mkdir, rmdir, mv 等    # 与 Linux 命令类似
  ```
