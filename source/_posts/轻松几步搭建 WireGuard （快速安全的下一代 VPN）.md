title: 轻松几步搭建 WireGuard （快速安全的下一代 VPN）
urlname: 轻松几步搭建 WireGuard （快速安全的下一代 VPN）
date: 2018/03/25 16:21:13
categories:
- 实用技巧
tags:
- VPN

---
![](https://image.covertness.cn/wire_guard_vpn/best-vpn-software.png)

WireGuard 是一个快速安全的新型 VPN 隧道程序，它简单高效的特性特别适合在手机等低能耗设备上使用。
<!-- more -->

WireGuard 不同于 IPSec ，它的设计简单（目前整体只有几千行代码），在不使用的情况下默认不会传输任何 UDP 数据包，而且能够无缝漫游在不同的 IP 地址间，这些特定都使它特别适合于移动设备的使用。目前 WireGuard 基于 Linux 内核实现，得到了 [Linux 内核主要维护者 Greg KH 的肯定](http://plus.google.com/+gregkroahhartman/posts/jD6N4BzToa3)。下面介绍如何在 Ubuntu 16.04 搭建使用 WireGuard 。

## 搭建步骤
### 下载安装 WireGuard （服务端和客户端都需要安装）
```bash
$ sudo add-apt-repository ppa:wireguard/wireguard
$ sudo apt-get update
$ sudo apt-get install wireguard
```

### 配置服务端相关参数，创建并编辑 `/etc/wireguard/wg0.conf` ，内容如下：
```ini
[Interface]
PrivateKey = <Private Key>
Address = 10.0.0.1/24
ListenPort = 56660
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE; ip6tables -A FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE; ip6tables -D FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
SaveConfig = true
```
其中 `PrivateKey` 通过命令 `wg genkey` 生成。

### 启动服务端 WireGuard
```bash
$ wg-quick up wg0
```
可以通过命令 `wg` 检查启动是否成功，成功的话会输出如下内容：
```bash
interface: wg0
  public key: xxxxxxxxxx
  private key: (hidden)
  listening port: 56660
```

### 将 WireGuard 设置成开机启动
```bash
$ systemctl enable wg-quick@wg0
```

### 配置客户端相关参数，创建并编辑 `/etc/wireguard/wg0.conf` ，内容如下：
```ini
[Interface]
PrivateKey = <Private Key>
Address = 10.0.0.3/24
DNS = 8.8.8.8

[Peer]
PublicKey = xxxxxxxxxx
Endpoint = <Server Public IP>:56660
AllowedIPs = 0.0.0.0/0
```
其中 `PrivateKey` 也是通过命令 `wg genkey` 生成， `Peer` 的 `PublicKey` 填入上面服务端 `wg` 命令返回的 `public key`， `Endpoint` 的 IP 设置为服务端可访问的公网 IP 。

### 启动客户端 WireGuard
```bash
$ wg-quick up wg0
```
启动后同样通过命令 `wg` 可查看公钥。

### 在服务端添加客户端信息
```bash
$ sudo wg set wg0 peer <Public Key> allowed-ips 10.0.0.3/24
```
`Public Key` 是客户端的公钥。
如果在服务端配置信息里设置了 `SaveConfig = true` 那么刚才添加的客户端参数信息会在服务端关闭时自动保存到配置文件中。如果想立即存储刚设置的参数也可以执行命令 `wg-quick save wg0` 。

### 测试验证
完成上述步骤后客户端便能直接访问服务端所在的网络了，可以通过 `ping 10.0.0.1` 进行验证。

## Tips
1. WireGuard 能够提供类似 TCP keepalive 的功能，如果客户端在 NAT 子网可以考虑[开启这一选项](https://www.wireguard.com/quickstart/#nat-and-firewall-traversal-persistence)。
2. WireGuard 目前仅实现了 Linux 内核模块版本，所以目前客户端仅支持部分 Linux 和 Android 。
