title: Linux 添加 Swap 空间
urlname: Linux 添加 Swap 空间
date: 2015/04/11 16:16:21
categories:
- 实用技巧
tags:
- Linux
- Swap

---

最近在一台比较旧的设备上遇到内存不足的问题，临时通过添加 Swap 的方式解决。
<!-- more -->

### 1. 创建swap文件
```bash
$ dd if=/dev/zero of=/path/swap.img bs=1M count=512   # 512MB
```

### 2. 格式化swap文件
```bash
$ mkswap /path/swap.img
```

### 3. 启用swap文件
```bash
$ sudo swapon /path/swap.img
```

### 4. 更新/etc/fstab，在底部添加一行
```
/path/swap.img     swap      swap defaults 0 0
```

### 5. 添加物理内存后可通过下面的命令关闭swap
```bash
$ swapoff -a
```
