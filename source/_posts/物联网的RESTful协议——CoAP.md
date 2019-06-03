title: 物联网的RESTful协议——CoAP
urlname: 物联网的RESTful协议——CoAP
date: 2015/07/26 14:07:26
categories:
- 探索
tags:
- IoT

---
![](https://image.covertness.cn/coap_54b7cd308da24.jpg)

Constrained Application Protocol (CoAP) 是为实现物联网中微型设备间互相通讯而设计的网络传输协议，它具有易于实现和使用、低功耗低带宽、能够保障数据安全等特性。
<!-- more -->

## 适用场景
### 请求－响应式通讯模型
CoAP 采用和 HTTP 协议类似的 REST 模型，服务内容通过 URL 获取，支持 GET 、 POST 等请求方法，易于与 HTTP 数据互相转化，实现与大多数现有 Web 应用的互联互通。

### 低性能的微型设备
CoAP 基于易于实现的 UDP 协议，数据包头仅占4个字节，需要的功耗和带宽都极低，能够运行在仅有 10KB RAM 的微型设备上。

### 保证数据安全
CoAP 支持 DTLS 协议保障数据安全。

## 数据包格式
CoAP 数据包分为 Header 、 Token （可选）、 Options （可选）、 Payload （可选）四个部分。

### Header
长度为4个字节。包含协议版本号、数据包类型、请求码、消息 ID 以及后面 Token 的长度。

### Token
长度由 Header 指定。请求与响应的 Token 需保持一致。

### Options
Key-Value 列表，其中 Value 也是列表，即每个 Key 下可以有多个 Value ，承载的内容与 HTTP Header 类似。

### Payload
数据包剩余的内容为 Payload ，为了便于解包与前面的部分通过 0xFF 隔开。

> 更详细的信息可参考[ RFC 文档](https://tools.ietf.org/html/rfc7252#section-3)。

## 实现
作为 IETF 设计的互联网标准协议，大部分平台已有相应的开源类库实现，可参考[这里](http://coap.technology/impls.html)。此外我自己也使用 [Rust](http://covertness.me/2015/05/24/Rust学习记录——语言特性及环境搭建/) 写了[一个类库](https://github.com/Covertness/coap-rs)，仅实现了一些基本的特性，因而代码也比较简单，有兴趣的童鞋可以看看。
