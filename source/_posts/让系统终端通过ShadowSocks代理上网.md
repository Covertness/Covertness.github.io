title: 让系统终端通过ShadowSocks代理上网
urlname: 让系统终端通过ShadowSocks代理上网
date: 2016/06/07 20:41:34
categories:
- 实用技巧
tags:
- 科学上网

---

使用 Privoxy 将 SOCKS5 代理转化为 HTTP 代理，然后在终端上设置 HTTP/HTTPS 代理实现上网。
<!-- more -->

1. 安装 Privoxy
```bash
$ brew install privoxy
```

2. 设置 Privoxy ，让 Privoxy 把 ShadowSocks 本地的 SOCKS5 代理转化为 HTTP 代理
编辑 `/usr/local/etc/privoxy/config` ，添加如下两行：
```
listen-address  0.0.0.0:8118       # http proxy port
forward-socks5 / localhost:1080 .  # SOCKS5 port
```

3. 启动 Privoxy
```bash
$ launchctl load /usr/local/opt/privoxy/homebrew.mxcl.privoxy.plist
```

4. 设置终端的 HTTP/HTTPS 代理
```bash
$ export http_proxy='http://localhost:8118'
$ export https_proxy='http://localhost:8118'
```