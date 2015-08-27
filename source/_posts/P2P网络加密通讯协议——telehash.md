title: P2P网络加密通讯协议——telehash
date: 2015/08/25 21:11:15
categories:
- 探索
tags:
- IoT
- P2P

---
![](http://7rf2ia.com1.z0.glb.clouddn.com/telehash_mesh-logo-128.png)

telehash 是由多位原 XMPP 协议作者设计的下一代 Jabber 网络通讯协议，它拥有去中心化、消息加密、多协议适配、接口简单易用等特点。
<!-- more -->

## 设计理念
### 完全的端到端消息加密
近年来随着网络监控、黑客入侵等信息泄露事件愈发频繁，人们对于个人隐私的保护越来越重视，各种对称/非对称加密手段越来越多地应用于互联网中。 telehash 采用不同于 HTTP 等基于[C/S 架构](http://baike.baidu.com/view/268856.htm)协议的传统加密方式，消息在发送端基于[公钥加密算法](http://baike.baidu.com/view/444169.htm)加密，而后消息只有到达接收端方可解密，确保中转传输过程中消息安全可靠，不会被任何第三者非法获取，因而利用它可以搭建出具有相当安全度的个人私有网络。

### 支持绝大多数设备终端
通过同时兼容 UDP、 TCP、 HTTP 等协议，现有大多数终端都可直接接入 telehash 网络。最近 IBM 已经基于此协议开发出一套物联网系统，详见[《IBM联合三星：利用比特币协议打造去中心化的物联网》](http://tech.163.com/15/0123/11/AGL33NQO00094ODU.html)。

### 去中心化
传统 C/S 架构的网络由于信息过多集中于服务器端，造成隐私泄露、信息被窃取的概率大大增加，而且随着数据量日益增大服务端的维护成本也节节攀升。 telehash 基于 P2P 网络设计，网络中每个节点地位完全对等，因而不会造成网络中某个节点不堪重负。

## 网络架构
一个 telehash 网络中每个节点拥有一个 Mesh ，每个 Mesh 上会有很多 Link ， Link 代表自身节点与其他任一节点的连接。网络中的节点还可以开启路由功能，帮助其他节点转发消息和发现更多节点。

### Mesh
它是节点的控制中心，管理节点上所有的 Link ，处理其他节点发来的建立 Link 请求。此外它还可以为其他节点寻找更多节点提供帮助。

### Link
它代表一个节点与其他任一节点的连接，每个 Link 都存有对方节点的基本信息，包括名称（ hashname ）、公钥（ CSKs ）、连接方式（ Paths ）等，消息的传递通过它进行。

## 快速搭建一个 telehash 网络
telehash 已有 C 、 node.js 、 python、 go 等多种语言实现，可以满足大多数应用场景的需求。下面使用 node.js 在本地网络快速搭建一个 telehash 网络。
### 下载 telehash 库
可以使用 `npm install telehash` 下载安装，也可以使用如下命令从 GitHub 上下载最新的版本：
```bash
git clone https://github.com/telehash/telehash-js.git
```

### 编码实现
这里实现一个拥有3个节点的 telehash 网络，其中一个节点（ 路由 ）用来转发消息，然后另外两个节点通过这个节点转发消息。 node.js 库已经写好了一个实现路由功能节点的代码，可直接通过下面的命令启动：
```bash
node telehash/bin/router.js
```

启动后可以看到屏幕上打印出了这个节点的信息，注意与 `router` 相关的信息后面会用到，类似下面这些：
```js
router: { hashname: '325mkv4v34onuusttfz3aeela4wjkj3lesvfmr4loefmg4fbfida',
  paths: 
   [ { type: 'udp4', ip: '192.168.1.24', port: 42424 },
     { type: 'http', url: 'http://192.168.1.24:62958' },
     { type: 'tcp4', ip: '192.168.1.24', port: 42424 } ],
  keys: 
   { '1a': 'ami5lxvrk2x57v56dgabaejqvjnyntpyzi',
     '2a': 'gcbacirqbudaskugjcdpodibaeaqkaadqiaq6abqqiaquaucaeaqbke3xpthucop55fpq2qtz7wq6ouaeeizanewi3notl55onhs74wsrsyq6hcw3dm53dfaltj4rs22veqbe32nrslgwluoo3gjm2jplwzchmujg3tmcyblq7avv75uskpd6bdw362e7csw4tmafel2sws5k2efyeywlytnlpl3xcmadxuliyhxezl5ad6rpzlqy75uulochkujtgqt4auknxibz5j2o6gpx5uoy4pk6xncliahh7aemz3pufoetgzj2w4u464mkpiaudsmoqbnt3rpm3iddatjg6rzu62z4jdopowuxqrigyhzmcby5pmticobdxr2y7v7suiendzmkjsiaroqz75fizlqgzo77ovqsf2kyejikk4dv2tvmetsxuuitzi3dwqeevadhgfhkqhdbnycamaqaai',
     '3a': 'nwqd2nv4trvjcoemhjrgb6nq3urh5542jt54dm5geboc4umggi2q' },
  router: true }
```

这段信息主要包含3个字段： `hashname` 是这个节点的名字，可以被网络中的其他节点用来查找此节点； `paths` 是这个节点的一些连接信息，可以看到其他节点可通过 UDP、 TCP 或者 HTTP 与它连接； `keys` 包含这个节点的公钥信息，为了适用不同的安全级别这里提供了3个公钥，具体使用哪个公钥依节点情况而定，具体的规则可以参考[这里](https://github.com/telehash/telehash.org/tree/master/v3/e3x/cs)。
接下来实现第二个节点，它会连上第一个节点并接收其他节点的连接请求，当其他节点发来消息时会打印到屏幕上，实现如下：
```js
var th = require("telehash");
var es = require('event-stream');

th.generate(function(err, endpoint) {                                   // 生成节点初始化信息
    if (err) return console.log("endpoint generation failed",err);

    th.mesh({                                                           // 初始化节点
        id: endpoint
    }, function(err, mesh) {
        if (err) return console.log("mesh failed to initialize", err);

        mesh.accept = function(from){                                   // 处理其他节点发来的连接请求
            var peer_link = mesh.link(from);

            peer_link.status(function(err) {
                if (err) {
                    console.log('peer disconnected', err);
                    return;
                }
                console.log('peer connected');
            });
        };

        mesh.stream(function(link, req, accept) {                       // 处理其他节点发来的数据流
            var peer_stream = accept();
            peer_stream.pipe(es.writeArray(function(err, items) {
                console.log("peer messages: ", items);
            }));
        });

        var router_link = mesh.link({                                   // 连接路由节点，输入参数为路由节点启动后打印的那个 JSON 串
            hashname: '325mkv4v34onuusttfz3aeela4wjkj3lesvfmr4loefmg4fbfida',
            paths: [{
                type: 'udp4',
                ip: '192.168.1.24',
                port: 42424
            }, {
                type: 'http',
                url: 'http://192.168.1.24:62958'
            }, {
                type: 'tcp4',
                ip: '192.168.1.24',
                port: 42424
            }],
            keys: {
                '1a': 'ami5lxvrk2x57v56dgabaejqvjnyntpyzi',
                '2a': 'gcbacirqbudaskugjcdpodibaeaqkaadqiaq6abqqiaquaucaeaqbke3xpthucop55fpq2qtz7wq6ouaeeizanewi3notl55onhs74wsrsyq6hcw3dm53dfaltj4rs22veqbe32nrslgwluoo3gjm2jplwzchmujg3tmcyblq7avv75uskpd6bdw362e7csw4tmafel2sws5k2efyeywlytnlpl3xcmadxuliyhxezl5ad6rpzlqy75uulochkujtgqt4auknxibz5j2o6gpx5uoy4pk6xncliahh7aemz3pufoetgzj2w4u464mkpiaudsmoqbnt3rpm3iddatjg6rzu62z4jdopowuxqrigyhzmcby5pmticobdxr2y7v7suiendzmkjsiaroqz75fizlqgzo77ovqsf2kyejikk4dv2tvmetsxuuitzi3dwqeevadhgfhkqhdbnycamaqaai',
                '3a': 'nwqd2nv4trvjcoemhjrgb6nq3urh5542jt54dm5geboc4umggi2q'
            },
            router: true
        });

        router_link.status(function(err) {
            if (err) {
                console.log('router disconnected', err);
                return;
            }
            console.log('router connected');
        });
    });
});
```

在一个新的窗口启动这个节点，稍后会看到此节点的名称（ `hashname` ），如下所示：
```js
generated new id wassrhxpyb4tj67qgt6ih3cukky6qsqw6l6xlzsnmmyyw22a2aja // 这段 hash 串即为 hashname
```

最后实现一个节点来向刚才的节点发送消息，可以将上面第二个节点的代码复制一份，然后只需要在路由节点连接成功的回调函数中加上连接第二个节点的代码即可，下面是完整代码：
```js
var th = require("telehash");
var es = require('event-stream');

th.generate(function(err, endpoint) {
    if (err) return console.log("endpoint generation failed",err);

    th.mesh({
        id: endpoint
    }, function(err, mesh) {
        if (err) return console.log("mesh failed to initialize", err);

        var router_link = mesh.link({                                   // 连接路由节点，输入参数为路由节点启动后打印的那个 JSON 串
            hashname: '325mkv4v34onuusttfz3aeela4wjkj3lesvfmr4loefmg4fbfida',
            paths: [{
                type: 'udp4',
                ip: '192.168.1.24',
                port: 42424
            }, {
                type: 'http',
                url: 'http://192.168.1.24:62958'
            }, {
                type: 'tcp4',
                ip: '192.168.1.24',
                port: 42424
            }],
            keys: {
                '1a': 'ami5lxvrk2x57v56dgabaejqvjnyntpyzi',
                '2a': 'gcbacirqbudaskugjcdpodibaeaqkaadqiaq6abqqiaquaucaeaqbke3xpthucop55fpq2qtz7wq6ouaeeizanewi3notl55onhs74wsrsyq6hcw3dm53dfaltj4rs22veqbe32nrslgwluoo3gjm2jplwzchmujg3tmcyblq7avv75uskpd6bdw362e7csw4tmafel2sws5k2efyeywlytnlpl3xcmadxuliyhxezl5ad6rpzlqy75uulochkujtgqt4auknxibz5j2o6gpx5uoy4pk6xncliahh7aemz3pufoetgzj2w4u464mkpiaudsmoqbnt3rpm3iddatjg6rzu62z4jdopowuxqrigyhzmcby5pmticobdxr2y7v7suiendzmkjsiaroqz75fizlqgzo77ovqsf2kyejikk4dv2tvmetsxuuitzi3dwqeevadhgfhkqhdbnycamaqaai',
                '3a': 'nwqd2nv4trvjcoemhjrgb6nq3urh5542jt54dm5geboc4umggi2q'
            },
            router: true
        });

        router_link.status(function(err) {
            if (err) {
                console.log('router disconnected', err);
                return;
            }
            console.log('router connected');

            // 连接第二个节点，输入参数为第二个节点的 hashname
            var peer_link = mesh.link('wassrhxpyb4tj67qgt6ih3cukky6qsqw6l6xlzsnmmyyw22a2aja');

            peer_link.status(function(err2) {
                if (err2) {
                    console.log('peer disconnected', err2);
                    return;
                }

                console.log('peer connected');

                var peer_stream = peer_link.stream();
                es.readArray([1, 2, 3]).pipe(peer_stream);       // 向第二个节点发送一个数组 [1, 2, 3]
            })
        });
    });
});
```

同样新打开一个窗口来启动这个节点，然后查看第二个节点所在的窗口，等待片刻便能看到新启动的节点发给它的消息，如下所示：
```js
peer messages:  [ 1, 2, 3 ]
```

这样便搭建好了一个拥有3个节点的 telehash 网络，网络中的所有通讯完全加密，比如最后一个节点发出的那个数组，仅有第二个节点可以接收解密出数组的内容，而转发消息的路由节点仅能看到一堆加密的乱码。

可以想象将来结合最新的[容器（ docker ）技术](https://www.docker.com)仅需要配置几个参数就可搭建出一个 telehash 网络，相信随着人们对于隐私保护的日益重视以及物联网的发展这项技术会得到越来越广泛的使用。