title: covertness.me出生记
date: 2014/11/16 09:31:44
categories:
- 建站
tags:
- Dynadot
- GitCafe
- Hexo
- 多说
- 七牛云存储

---
一个独立博客的诞生历程，记录了从域名注册到完成第一篇博客（即本文）的整个过程。
<!-- more -->

## 什么是独立博客
博客，相信现在大部分人都对它并不陌生了，维基百科上对**博客**一词是这样解释的：
> 博客是一种由个人管理、不定期张贴新的文章、图片或视频的网页或在线日记，用来抒发情感或分享信息。

可以看到博客是由个人发布和管理的，当然现在也有很多组织以博客的形式来向公众发布一些信息，但可以肯定博客最原始的需求来源于个人。
[前博客时代](http://www.15yan.com/story/fyYlkIp03v5/)大家一般在一些[博客平台](http://zh.wikipedia.org/wiki/部落格服务提供商列表)上建立自己的博客，新浪博客、网易博客、blogcn等等，当时我也在新浪博客上建立了[一个自己的博客](http://blog.sina.com.cn/wuyingfengsui)。伴随着博客群体的不断壮大许多社会问题也相伴而来，诽谤污蔑他人、制造谣言、隐私泄露，为了解决这些问题，各大博客平台纷纷加强了监管的力度。但互联网与生俱来的天性——自由却似乎总与监管相处并不融洽，于是一些人选择建立**独立博客**。维基百科中对独立博客作了如下解释：
> 独立博客一般指在采用独立域名和网络主机的博客，既在空间、域名和内容上相对独立的博客。

在前面提到的前博客时代要建立这样的一个博客需要做很多方面的准备，域名申请、主机购买、服务器搭建等很多工作都需要自己完成。幸运的是随着近几年云服务的快速发展，独立博客不再是一些极客的专属玩物，任何人都可以基于现有的云服务在几个小时之内建立起专属自己的博客，而且这一切都是免费的。

## 需要用到的工具
- [Sublime Text](http://www.sublimetext.com/) 写博客的编辑器，或者使用[在线编辑器](https://www.zybuluo.com)
- [Hexo](http://hexo.io/) 将写好的博客转换为浏览器能够识别并显示的文档
- [GitCafe](https://gitcafe.com/) 存放博客的文档
- [Git](http://git-scm.com/) 将写好的博客上传到GitCafe
- [多说](http://duoshuo.com/) 为博客提供评论功能
- [七牛云存储](http://www.qiniu.com/) 为博客存储附件（图片、文件等）
- [Dynadot](https://www.dynadot.com/zh) 获取一个独立的域名，博客使用[一级域名](http://baike.baidu.com/view/254595.htm)才需要

## 搭建过程
### 1. 在Dynadot上获取域名：covertness.me
Dynadot界面友好，操作简单，根据网站提示指引便可完成。
![](http://covertness.qiniudn.com/born_domain.PNG)


### 2. 注册多说账号
在[管理页面](http://covertness.duoshuo.com/admin/settings/)获得shortname。shortname即域名的第一节，如域名是covertness.duoshuo.com，则shortname为covertness。
![](http://covertness.qiniudn.com/born_duoshuo.PNG)


### 3. 在GitCafe上创建博客的仓库
建立一个与用户名一样项目名的项目即可。
![](http://covertness.qiniudn.com/born_resp.PNG)


### 4. 使用Hexo创建一个博客
只需在一个合适的目录下执行以下命令即可：
```bash
$ hexo init blog
$ cd blog
$ npm install
```


### 5. 写博客
使用Sublime Text打开刚才Hexo创建的blog目录，在source/_post/目录下创建一个名为born.md的文件，然后使用[Markdown](http://www.jianshu.com/p/q81RER)撰写第一篇博客（即本文）。


### 6. 添加博客附件
在七牛云存储上新建一个空间，上传博客附件，然后将外链地址添加到博客原文对应位置即可。
![](http://covertness.qiniudn.com/born_attach.PNG)


### 7. 删除默认主题landscape及默认博客hello-world
删除在themes/目录下的landscape子目录和source/_post/目录下的hello-world.md即可。


### 8. 安装博客主题landscape-plus
只需在blog目录下执行以下命令即可：
```bash
$ git clone https://github.com/xiangming/landscape-plus.git themes/landscape-plus
```


### 9. 设置博客参数
打开blog目录下的_config.yml，修改如下几个参数：
```yml
title: 无影风随                  # 博客网站标题
author: 无影风随                 # 博主名
email: wuyingfengsui55@sina.com  # 博主邮箱
language: zh-CN                  # 博客语言，使用IETF格式
url: http://covertness.me        # 博客域名
theme: landscape-plus            # 博客主题
deploy:
  type: github
  repository: git@gitcafe.com:wuyingfengsui/wuyingfengsui.git
  branch: gitcafe-pages
```


### 10. 设置博客主题参数
打开themes/landscape-plus目录下的_config.yml，修改如下参数：
```yml
duoshuo_shortname: covertness    # 多说的shortname
```


### 11. 生成博客文档
只需在blog目录下执行以下命令即可：
```bash
$ hexo g
```


### 12. 预览
只需在blog目录下执行以下命令，然后使用浏览器访问[127.0.0.1:4000](http://127.0.0.1:4000/)查看。
```bash
$ hexo s
```


### 13. 部署到GitCafe
只需在blog目录下执行以下命令即可：
```bash
$ hexo d
```


### 14. 关联域名
#### 设置GitCafe中的自定义域名
![](http://covertness.qiniudn.com/born_domain2dns.PNG)

#### 将Dynadot的域名dns绑定到GitCafe服务器的IP
![](http://covertness.qiniudn.com/born_dns.PNG)


### 15. 等待一段时间后（DNS广播时间）访问[covertness.me](http://covertness.me/)
![](http://covertness.qiniudn.com/born_release.PNG)