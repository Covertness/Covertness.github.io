title: 快速搭建OpenVPN服务
urlname: 快速搭建OpenVPN服务
date: 2015/12/20 21:00:11
categories:
- 实用技巧
tags:
- VPN

---

使用开源软件 OpenVPN 在一台 VPS 上搭建稳定安全的 VPN 代理服务。
<!-- more -->

**由于 OpenVPN 的实际效果并不理想（数据传输速度缓慢），推荐使用 [WireGuard](http://covertness.me/2018/03/25/%E8%BD%BB%E6%9D%BE%E5%87%A0%E6%AD%A5%E6%90%AD%E5%BB%BA%20WireGuard%20%EF%BC%88%E5%BF%AB%E9%80%9F%E5%AE%89%E5%85%A8%E7%9A%84%E4%B8%8B%E4%B8%80%E4%BB%A3%20VPN%EF%BC%89/) 代替。**

### 1. 在 VPS 上执行以下命令下载安装 OpenVPN
```bash
$ wget git.io/vpn --no-check-certificate -O openvpn-install.sh && bash openvpn-install.sh
```
其中的 DNS 配置建议使用 Google 。

### 2. 再次运行此脚本添加用户，添加完成后会自动生成客户端配置文件 *.ovpn （可选）
```bash
$ bash openvpn-install.sh
```

### 3. 启动一个 HTTP 服务器用于客户端下载配置文件
```bash
$ python -m SimpleHTTPServer
```

### 4. 在需要使用 VPN 服务的终端上下载客户端配置文件，然后安装对应的客户端即可，下面是 Windows 和 Android 的客户端下载地址：
- [Windows](https://openvpn.net/index.php/open-source/downloads.html) 由于需要改动系统路由表， Windows 版本需要以管理员权限运行
- [Android](https://play.google.com/store/apps/details?id=net.openvpn.openvpn) OpenVPN Connect

### 5. 如果发现 VPN 网速比较慢，可以通过修改 VPS 上的配置文件进行改善，在 `/etc/openvpn/server.conf` 中添加如下几行：
```
sndbuf 393216
rcvbuf 393216
push "sndbuf 393216"
push "rcvbuf 393216"
```
然后执行 `service openvpn restart` 重启 OpenVPN 服务即可。
