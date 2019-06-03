title: R语言——数据统计分析利器
urlname: R语言——数据统计分析利器
date: 2015/09/16 21:00:22
categories:
- 探索
tags:
- 大数据

---
![](https://image.covertness.cn/r_shujutongjifenxiliqi_76fa2713d8acbde7ad04266a37d8ac57.jpg)

R 语言广泛应用于金融、社交网络、机器学习等众多需要大数据分析的领域，其拥有大量实用的开源程序包供使用者直接调用。
<!-- more -->

## 优势
### 小巧灵活
R 是一门脚本语言，大量的算法实现均以第三方包的形式存在，需要时可通过 [CRAN](https://cran.r-project.org/mirrors.html) 下载使用，因而其本身极其轻便，而且可以方便地进行扩展。

### 丰富的第三方包
CRAN 是 R 语言的包管理服务，由于 R 语言基于自由开源的 GPL 协议，因而众多的开发者都乐于将自己开发的源码分享到 CRAN 上。在 CRAN 上能找到几乎所有和统计相关的实现代码，不论是最新的算法还是经典的统计方法。

## 小试牛刀
### 安装
可以使用[官网](https://cran.rstudio.com)提供的安装包直接安装，也可使用系统的包管理工具（如 Debian 的 apt ）进行安装。

### 启动
在终端窗口输入 r 命令即可打开 R 终端，然后就可以在这里执行 R 代码啦。

### 设置 CRAN 源
CRAN 在全球有很多镜像服务器，在 [官网页面](https://cran.r-project.org/mirrors.html) 可以查看，查找到离你最近的一个服务器地址后将它配置到 R 的配置文件（ ~/.Rprofile ）中，格式如下：
```
options(repos = c(CRAN = "https://mirrors.ustc.edu.cn/CRAN"))
```

### 分析股市行情
R 语言被广泛地用于金融领域，CRAN 中的 `quantmod` 包可以获取全球股市交易数据并绘制交易图形，下面就用它简要介绍 R 语言的一些基本用法。

#### 下载安装 `quantmod`
```R
install.packages("quantmod")
```
在 R 语言中只需执行 `install.packages` 便可自动下载安装指定包及其依赖，仅需要在第一次运行前执行此命令，以后便可直接使用。

#### 导入 `quantmod`
```R
require(quantmod)
```
在新启动的 R 运行实例中第一次使用某个包前，需要执行 `require` 将其导入。

#### 获取上证交易所今年的交易数据
```R
ssec<-getSymbols("^SSEC",from = "2015-01-01",to = Sys.Date(),src = "yahoo", auto.assign=FALSE)
```
上面的指令使用 `quantmod` 包的 `getSymbols` 函数从雅虎数据中心获取上证交易所（ `SSEC` 表示上证 ）今年的交易数据，然后将获取的数据存入了 `ssec` 这个变量，为了方便查看可以使用下面的函数将其绘制成图表：
```R
chartSeries(ssec)
```
![](https://image.covertness.cn/r_shujutongjifenxiliqi_1.png)

从图中可以看到数据中包含了指数和交易量等一些信息。当然也可以只查看某只股票的具体信息，下面的命令是查看工商银行（ 交易代码为`601398` ）今年的数据：
```R
gsyh<-getSymbols("601398.ss",from = "2015-01-01",to = Sys.Date(),src = "yahoo",auto.assign=FALSE)
chartSeries(gsyh)
```
![](https://image.covertness.cn/r_shujutongjifenxiliqi_2.png)

除了获取股市的数据， R 语言也能通过其他第三方包方便地获取其他网络的海量数据。结合 R 语言丰富的数据分析包即可方便地对这些数据进行数学建模分析。
