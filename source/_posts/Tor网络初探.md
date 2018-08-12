title: Tor网络初探
date: 2015/04/14 21:06:22
categories:
- 探索
tags:
- Tor
- P2P
- 匿名

---
![](https://image.covertness.me/tor_wangluochutan_0d3c2fc1c3b0009633f8c65e5dcdbdcf_b.jpg)

Tor网络是出现较早的一种匿名网络，因其经过多次代理的网络特性，不论是在上面查看别人发布的信息还是主动发布信息都较难被其他人跟踪。
<!-- more -->

## 原理
Tor网络上存在很多中继节点，通过这些中继节点的多次转发来隐藏真实的网络地址，可以通过一个简单的实验加以验证：
![](https://image.covertness.me/tor_wangluochutan_20150302_214314.jpg)
> 在Tor网络中查询本机IP，如上图所示，可以看到每次查询结果都不一致。

## 使用方法
### 1. 代理上网
因Tor项目的官方网站已无法直接访问，需要设置前置代理（如HTTP代理、VPN）。

### 2. Tor浏览器
连接Tor网络需要本地客户端，Tor项目已经为各平台制作了一个定制的浏览器，一般使用它便可访问Tor网络。
> 下载地址：https://www.torproject.org/download/download-easy.html.en

下载安装后打开Tor Browser，出现网络设置界面，如果使用VPN选择直接连接即可，如果是HTTP代理按照提示配置即可。之后便会显示一个页面提示已接入Tor网络，如下图。
![](https://image.covertness.me/tor_wangluochutan_%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202015-04-14%20%E4%B8%8B%E5%8D%888.54.29.png)

### 3. 常用网站：
搜索引擎：http://hss3uro2hsxfogfq.onion/
![](https://image.covertness.me/tor_wangluochutan_%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202015-04-14%20%E4%B8%8B%E5%8D%889.14.59.png)

Wiki：http://torwikignoueupfm.onion/index.php?title=Main_Page
![](https://image.covertness.me/tor_wangluochutan_%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202015-04-14%20%E4%B8%8B%E5%8D%889.14.50.png)

Tor网络中隐藏的网站域名都以*.onion*结尾，其他一些网站可以参考[这个网站](http://dirnxxdraygbifgc.onion/)，也可通过Wiki上的链接找到，网站交易大多使用比特币。

### 4. 如何搭建 Tor 网络中的网站：
需要在本地搭建HTTP服务器，详细步骤可参考：https://www.torproject.org/docs/tor-hidden-service.html.en
