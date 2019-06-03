title: 让博客支持 HTTPS
urlname: 让博客支持 HTTPS
date: 2018/08/12 16:31:44
categories:
- 建站
tags:
- TLS
- Nginx

---
![](https://image.covertness.cn/openssl_https_server/https.png)

博客之前使用七牛云存储图片、视频等静态资源，由于七牛云只有在绑定已备案的自定义域名后才能开启 HTTPS ，步骤繁琐周期长，看到 [Let’s Encrypt](https://letsencrypt.org/) 提供颁发免费的 SSL 证书，遂决定基于此自建一个简单的 HTTPS 代理服务，方便博客以及日后使用。

<!-- more -->
自建的 HTTPS 代理服务提供了一种快速让网站支持 HTTPS 的功能，只需提供要通过 HTTPS 访问的域名及原来的地址，然后更改下域名的 CNAME 就可免费使用 HTTPS ，无需手动申请和更新证书，非常方便。待日后服务稳定后我将开放给大家使用，具体的实现细节也会在以后介绍。

目前博客的图片、视频等静态资源都是通过这个服务转变为 HTTPS 的地址，下面介绍一下如何将原来基于七牛云的资源链接迁移到新的 HTTPS 链接。

### 找出博客中所有的静态资源链接
由于博客是通过 Markdown 格式存储的，所有需要显示的静态资源都会以类似 `![](http://xxx)` 这样的方式标记，因此我们可以通过下面的命令将它们都筛选出来：
```bash
$ grep -Eoi '\!\[\]([^>]+)' -r source/_posts | grep -Eo '(http)://[^)]+' > assets_url.txt
```
我将筛选后的结果放到了 `assets_url.txt` 文件中以便后续下载使用。

### 下载所有的静态资源
通过 `wget` 可以很容易地下载 `assets_url.txt` 里的所有 URL ：
```bash
$ wget --restrict-file-names=nocontrol -i assets_url.txt
```
这里添加 `--restrict-file-names=nocontrol` 的作用是保证中文的文件名以 UTF-8 的形式存储，否则很可能会出现乱码。此外还需要注意 `wget` 会将所有文件都下载到当前目录下，所以如果 URL 中存在多级目录时需要手动处理下。

下载完成后便可以把它们放到一个能够通过 http 访问的文件服务器上，我使用的是 Nginx 搭建的一个文件服务器，为了支持跨域访问添加了如下配置：
```conf
            if ($request_method = 'OPTIONS') {
               add_header 'Access-Control-Allow-Origin' '*';
               add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
               #
               # Custom headers and headers various browsers *should* be OK with but aren't
               #
               add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
               #
               # Tell client that this pre-flight info is valid for 20 days
               #
               add_header 'Access-Control-Max-Age' 1728000;
               add_header 'Content-Type' 'text/plain; charset=utf-8';
               add_header 'Content-Length' 0;
               return 204;
            }
            if ($request_method = 'POST') {
               add_header 'Access-Control-Allow-Origin' '*';
               add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
               add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
               add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
            }
            if ($request_method = 'GET') {
               add_header 'Access-Control-Allow-Origin' '*';
               add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
               add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
               add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
            }
```

### 替换博客里原来的链接
HTTPS 代理服务里配置了 https://image.covertness.cn 作为博客的静态资源地址，下面通过 `sed` 把博客源码里原来七牛云的地址都替换掉：
```bash
$ find ./source/_posts -type f -exec sed -i '' -e 's/http:\/\/7rf2ia.com1.z0.glb.clouddn.com/https:\/\/image.covertness.me/g' {} \;
$ find ./source/_posts -type f -exec sed -i '' -e 's/http:\/\/covertness.qiniudn.com/https:\/\/image.covertness.me/g' {} \;
```

重新发布博客后通过浏览器会发现此时博客的连接是完全安全的了。
![](https://image.covertness.cn/blog_https_enhance.png)
