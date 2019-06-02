title: Rust学习记录——语言特性及环境搭建
urlname: Rust学习记录——语言特性及环境搭建
date: 2015/05/24 15:23:43
categories:
- 学习记录
tags:
- Rust
- Cargo
- Sublime Text

---
![](http://www.rust-lang.org/logos/rust-logo-blk.svg)

Rust 是为编写高性能应用程序而设计的一门系统编程语言，旨在替代 C/C++ 完成更加高效安全的系统级程序开发。
<!-- more -->

## 为什么使用 Rust
### 更加安全的内存操作
众所周知使用 C/C++ 编写程序非常容易遇到无效内存访问（ Segmentation fault ），对于新手这个问题从程序编写调试阶段就会出现，就算对于有多年相关编程经验的人士也难免在程序正式运行一段时间后遇到此类问题。 Rust 通过所有权 ( ownership ) 、变量不变（ immutable ）、借用检查（ borrow check ）、不共享内存的线程（ threads without data races ）等特性大大降低了发生此类问题的概率。

### 函数式编程
随着计算设备并行化处理能力的不断增强，近年来支持函数式编程的语言优势越来越明显。Rust 支持包括函数式在内的多种编程风格，这无疑有助于适应未来高并行化的应用场景。

## 环境搭建
### 1. 从[官网](http://www.rust-lang.org)下载安装包进行安装
![](https://image.covertness.me/rust_yuyantexingjihuanjindajian_1.png)

**Windows平台需要添加相应的环境变量。**

### 2. 创建测试项目
在命令行界面输入如下指令创建一个可执行程序项目：
```bash
$ cargo new hello_rust --bin
```

### 3. 编译项目
```bash
$ cd hello_rust/
$ cargo build
```

### 4. 运行项目
```bash
$ cargo run
```

### Sublime Text 的 Rust 插件
- [Rust](https://packagecontrol.io/packages/Rust) 提供 Rust 语法高亮功能
- [Rust​Auto​Complete](https://packagecontrol.io/packages/RustAutoComplete) 提供 Rust 代码自动完成功能