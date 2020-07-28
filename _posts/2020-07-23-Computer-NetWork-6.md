---
layout: post
title:  "计算机网络学习笔记六 应用层"
date:   2020-07-24 06:00:00
categories: 计算机网络
tags: 计算机网络 应用层
mathjax: true
mermaid: true
---

* content
{:toc}

应用层协议都是为了解决<span style="color:red">某一类应用问题</span>，而往往是通过位于不同主机中的多个应用进程之间的通信和协同工作来完成的。许多协议都是基于客户-服务器模式。

本节内容主要包含域名系统DNS，文件传输系统，远程终端协议TELNET，万维网WWW，电子邮件，动态主机配置协议DHCP，P2P应用。



## 域名系统
### 域名系统概述
* 互联网采用层次结构的命名树作为主机名字，使用分布式的域名系统DNS，Domain Name System。
* 名字到IP地址解析，又若干个<span style="color:red">域名服务器</span>完成。

### 互联网域名结构
* 每台主机或路由器都有唯一的层次结构名字，即域名，域名由标号序列组成，标号间由点隔开：
  * <span style="color:blue">... .三级域名.二级域名.顶级域名</span>
* 域名只是<span style="color:blue">逻辑概念</span>，便于人使用，而IP地址才是机器使用的地址。
* 一般分为：顶级域名TLD，Top Level Domain(包括国家域名、通用域名等)、二级、三级等。

### 域名服务器
* 一般层次分为：根域名服务器、顶级域名服务器、权限域名服务器、本地域名服务器
* 根域名服务器：最高层次域名服务器，所有根域名服务器知道所有顶级域名服务器和IP地址。
* 顶级域名服务器：负责管理该顶级域名服务器注册的所有二级域名。
* 其中权限域名服务器：负责一个区的域名服务器。
* 本地域名服务器(默认域名服务器)：帮助主机DNS查询服务。
* 域名查询：迭代查询，递归查询。主机查询一般都是递归查询。其中查询DNS一般采用UDP用户数据报的报文形式进行。

<figure align="center">
  <img src = "/media/image/ComputerNetwork/DNSQuery.png" style="width:90%" />
</figure>

* 利用<span style="color:red">高速缓存</span>可以减少查询流程，减轻服务器复合，减小因DNS查询的报文数量。域名改动一般不频繁。

## 文件传送协议
### FTP概述
* 文件传送协议File Transfer Protocol:应用最广泛的文件传输协议。<span style="color:red">交互式访问，屏蔽了计算机系统的细节</span>。

### FTP的基本工作原理
* 网络环境复杂：计算机文件存储格式不同；目录和文件命名规定不同；相同文件存取功能，操作系统使用的命令不同；访问控制的方法不同。
* FTP特点：
  * FTP只提供<span style="color:red">文件传输的基本服务</span>，它使用TCP可靠的运输服务。
  * FTP主要功能是减少和消除不同操作系统下处理文件的<span style="color:red">不兼容性</span>。
  * FTP采用客户-服务器方式。一个服务器可以同时为多个客户进程提供服务。FTP的服务器进程有两部分组成:<span style="color:red">一个主进程</span>(负责处理新请求)，<span style="color:red">若干个从属进程</span>，处理单个请求。
  * FTP服务器数据连接和控制连接采用不同端口，控制连接(客户进程向服务器发器连接请求等)使用熟知端口21，数据连接(服务器给客户进程传输数据等)20。

### 简单文件传送协议 TFTP
* Trivial File Transfer Protocol是一个小而易于实现的文件传送协议。使用客户-服务器方式和<span style="color:red">UDP数据报</span>(因此应用层差错改正)。
* 不能对用户身份鉴别，没有列目录的功能。
* TFTP主要特点：
  * 每次传送的数据PDU称作<span style="color:red">文件块</span>，块编号从1开始，每块512字节(最后一块可小于512).
  * 使用ASCII码或二进制传送。
  * 可文件读写。
  * 使用简单的部首。
* 工作特点：类似停止等待协议。发送完一个数据块后，等待对方确认，确认时需要指明所确认的块编号。在规定时间内未接受确认，重发数据PDU，Protocol Data Unit。发送确认PDU的一方在规定时间内不能收到下一块，重发确认PDU。

## 远程终端协议TELNET
* TELNET是一个简单的远程终端协议，也是互联网的正式标准。通过<span style="color:red">TCP</span>连接，为用户提供<span style="color:red">透明服务</span>。
* 采用客户-服务器模式，本地运行客户进程，远地主机运行TELNET服务器进程。同样服务器主线程处理新请求、从属进程处理每一个请求。

## 万维网WWW
### 万维网概述
* 万维网WWW, World Wide Web，是一种大规模的、联机式的信息储藏所。
* 使用<span style="color:red">链接</span>的方式简单的访问，按需获取信息。
* 超媒体与超文本：
  * 万维网是分布式<span style="color:red">超媒体系统</span>hypermedia，是超文本系统扩充。
  * 超文本系统：有多个信息源链接成，利用链接可以让用户找到任意一个位于互联网上的找文本系统的档案。
  * 超媒体与超文本区别是文档内容不同。
* 万维网的工作方式：
  * 客户-服务器方式；
  * 浏览器：用户计算机上的客户程序；文档所在计算机则运行服务器程序(其计算机为万维网服务器)。
  * 发送的数据是万维网文档。 而客户窗口显示的万维网文档称为页面。
* 万维网要解决的问题：
  * 如何标志分布在互联网上的万维网文档：采用统一资源定位符：URL Uniform Resource Locator。
  * 如何实现万维网上各种超链的链接：客户程序与服务器程序交互协议：<span style="color:red">超文本传送协议</span>HTTP, HyperText Transfer Protocol，使用<span style="color:red">TCP</span>传输。
  * 万维网文档如何显示，用户如何知道超链的地址：<span style="color:red">超文本标记语言</span>HTML, HyperText Markup Language，并且能实现超链接。
  * 如何使用户能方便找到所需信息：搜索工具/搜索引擎。

### URL的格式
* URL的统一格式：$<协议>://<主机>:<端口>/<路径>$
* 常见的协议：http，ftp，News等。
* URL对大小写不敏感。很多部分可省略：如端口，路径，协议，甚至主机域名前的www。

### 超文本传送协议HTTP
#### HTTP 操作过程
* HTTP协议HyperText Transfer Protocol，高效传送一切必须的信息，是可靠交换文件的重要基础。
* 请求文档所需时间：2倍的RTT时间(TCP连接连接，HTTP请求报文）+文档传输时间。

#### 代理服务器
* 代理服务器proxy server：又称为万维网高速缓存Web cache，代表浏览器发出Http请求。
* 代理服务器会将最近的一些请求和响应暂存在本地磁盘。
* 代理服务器可将最近暂存的响应发送出去，不用每次都按URL地址去互联网访问该资源。
* 当没有缓存该URL链接响应时，则代表用户浏览器(实际是<span style="color:red">高速缓存</span>)与源点服务器建立TCP连接，并发送HTTP请求报文。

#### 服务器上存放用户的信息
* Cookie表示HTTP服务器和客户之间传递的状态信息。
* 使用Cookie的网站服务器为用户生成一个唯一的识别码，利用此识别码，网站能够跟踪用户在该网站的活动。

### 万维网的文档
#### 超文本标记语言HTML
* HTML中的markup是设置标记的意思，为了在浏览器上显示数据。定义了许多<span style="color:red">排版的命令</span>(<span style="color:red">标签</span>)。可以用任何文本编辑器创建和编辑的ASCII码文件。
* XML Extensible Markup Language是扩展标记语言。宗旨是传输数据，作为HTML的补充。
* HTML5是HTML的未来版本。
* CSS Cascading Style Sheets层叠样式表，给HTML文档定义布局。HTML用于结构化内容，CSS用于格式化结构化的内容。

#### 动态万维网文档
* 静态和动态的区别：是否是浏览器访问万维网时才由应用程序动态创建的。差别仅体现在<span style="color:red">服务器</span>这一侧。
* 通用网关接口CGI Common Gateway Interface：一种标准，定义了动态文档如何创建，输入数据如何提供给应用程序以及输出结果应如何使用。
  * 通用：CGI标准对其他任何语言都是通用的。
  * 网关；CGI程序像网关一样，可以访问其他的服务器资源，如数据库和图形软件包等。
  * 接口：有一些已定义好的变量和可供调用的CGI程序使用。
* 区分CGI程序和CGI标准：CGI程序(又叫CGI脚本script)指的是按照CGI标准与万维网服务器通信的程序。

#### 活动万维网文档
* 活动文档技术把所有工作转移给浏览器。避免服务器连续推送。
* Java技术创建活动文档。使用Java小应用程序applet来描述活动文档。

### 万维网的信息检索系统
* 在万维网中进行搜索的程序叫作搜索引擎。
* 全文检索搜索：纯技术型的检索工具。搜索软件在互联网上的各网站收集信息，建立很大的在线数据库，用户输入关键词，在已建立的索引数据库上进行查询。
* 垂直搜索引擎：Vertical Search Engine针对特定领域提供搜索。

### 一些特殊的应用
* 博客：Web log 音译blog。
* 微博：微型博客micro blog
* 社交网站：SNS, social Networking Site

## 电子邮件
### 电子邮件概述
* 电子邮件发送协议：SMTP，读取协议：POP3和IMAP。通信采用TCP协议。
* 电子邮件地址：<span style="color:red">$收件人邮箱@邮箱所在主机的域名$</span>
* 电子邮件工作的流程举例：

<div class="mermaid" align="center">
graph LR
    class id2 cssClass;
    id1(SMTP<br/>客户)-->id2(SMTP<br/>服务器);
    id3(SMTP<br/>客户)-->id4(SMTP<br/>服务器);
    id5(POP3<br/>服务器)-->id6(POP3<br/>客户);
    subgraph 发件用户代理
    id1
    end
    subgraph 发送服务器
    id2 
    id3
    end
    subgraph 接收服务器
    id5
    id4
    end
    subgraph 收件用户代理
    id6
    end
    style id1 fill:#f9f,stroke:#333,stroke-width:4px
    style id2 fill:#f9a,stroke:#333,stroke-width:4px
    style id3 fill:#f9f,stroke:#333,stroke-width:4px
    style id4 fill:#f9a,stroke:#333,stroke-width:4px
    style id5 fill:#a8f,stroke:#333,stroke-width:4px
    style id6 fill:#a8d,stroke:#333,stroke-width:4px
</div>

### 简单邮件传送协议SMTP
* SMTP Simple Mail Transfer Protocol:采用客户-服务器模式
* 通信三阶段；建立连接、邮件传递、连接释放(发送完毕，释放TCP)。

### 电子邮件的信息格式
* 电子邮件分为信封和内容两部分
* 邮件内容首部包括一些关键字，后面加上冒号。最重要的关键字：To和Subject。

### 邮件读取协议POP3和IMAP
* 邮局协议POP, Post Office Protocol采用<span style="color:red">客户-服务器</span>模式，POP3是该协议第三个版本。通过PC机中的POP客户程序，连接邮件服务器中的POP服务器程序。
* 交互邮件访问协议IMAP, Internet Message Access Protocol：是一个<span style="color:red">联机协议</span>，也采用<span style="color:red">客户-服务器</span>模式。用户可以看到邮件首部，当用户打开某个邮件才传输到计算机。并且允许收件人<span style="color:red">仅读取部分邮件内容</span>。

### 基于万维网的电子邮件
* 用户代理和服务器之间的连接采用<span style="color:red">HTTP协议</span>(接受和发送均可为HTTP协议)，而发送方服务器和接收方服务器任然采用SMTP协议。
* 使用浏览器的程序作为用户代理程序。

## 动态主机配置协议DHCP
* 软件协议在运行之间，必须对每一个参数赋值，给这些参数赋值的动作叫作协议配置。这样便于软件协议的移植和通用化。
* 例如<span style="color:red">动态主机配置协议</span>DHCP, Dynamic Host Configuration Protocol提供了即插即用连网的机制。主机使用广播寻找DHCP服务器或DHCP中继代理，所以采用<span style="color:red">UDP用户数据报</span>，DHCP报文只是UDP报中的数据。允许一台主机接入网络时IP地址不用手工配置。
* 其中可能利用到DHCP中继代理，中继代理与服务器之间的DHCP发现报文以<span style="color:red">单播</span>方式转发。
* DHCP服务器分配的IP地址具是临时的，有一定的<span style="color:red">租用期</span>。

## P2P应用
* 下载者即上传者。客户即服务器。

### 集中目录服务器P2P
* Napster的文件传输是分散的，但<span style="color:red">文件定位是集中</span>的。
* 集中式目录服务器的缺点是可靠性差。(还有版权问题)

### 全分布式结构的P2P
* Gnutella：全分布式定位内容，<span style="color:red">有限范围的洪泛法</span>查询。影响到查询定位的准确性。
* eMule采用分散定位和分散传输技术(多源文件传输协议MFTP)。文件分成不同小块，并行传输。
* BT协议：Bit Torrent协议(也是一种应用程序)，广泛用于对等网络通信。
  * BitTorrent所有对等方集合称为一个<span style="color:red">洪流torrent</span>。
  * 下载的数据单元为长度固定的<span style="color:red">文件块chunk</span>。
  * 基础设施节点叫<span style="color:red">追踪器tracker</span>。
  * 与己方建立TCP连接的对等方叫作“<span style="color:red">相邻对等方</span>”。
  * <span style="color:red">最稀有优先</span>rarest first，先下载相邻对等方拥有数最少的chunk。

### 在P2P对等方中搜索对象
* 分布式散列表DHT，Distributed Hash Table。大量对等方共同维护散列表。

计网基础部分End

前前后后一共画了两周时间，把计算机网络的基本内容梳理了一遍，对网络结构大致了解。暂时计网部分告一段落，后续可能设计到实际应用的时候在添加。