title: Rust学习记录——FFI
date: 2015/08/11 20:43:26
categories:
- 学习记录
tags:
- Rust
- Hashcash

---
![](https://image.covertness.me/rust_ffi_languages.png)

Rust 无需运行时（ runtime ）的优势使得 Rust 与其他语言的相互调用变得简单而高效。
<!-- more -->

由于 Rust 致力于成为系统级编程语言，因而它并没有像其他语言一样的运行时环境（ runtime ），这也给其他语言与之结合提供了便利。 [Hashcash](http://www.hashcash.org) 是用来验证计算机计算能力的一种算法，被用于比特币挖矿以及部分反垃圾邮件的系统中，算法简明而有效。此算法作者的实现用的是 C 语言，下面就通过 Rust 的外部函数接口（ Foreign Function Interface ）来调用他实现的 Hashcash C 库接口，以快速实现 Hashcash 算法的 Rust 版本。

## 获取 C 语言版本的 Hashcash
### 1. 下载源码
可从 GitHub 上下载到 Hashcash C 库的镜像，地址如下：
```bash
git clone https://github.com/jbboehr/hashcash.git
```

### 2. 编译安装
由于这个库使用 autoconf 自动构建工具， Ubuntu 下可执行如下命令安装（其他平台类似）：
```bash
sudo apt-get install autoconf automake libtool
```

完成后再执行下列命令即可。
```bash
autoreconf -i
./configure
sudo make install
```

### 3. 验证安装
我们可以通过运行下面这个简单的测试程序来验证 Hashcash C 库是否可用。
```c
#include <stdio.h>
#include <hashcash.h>

int main() {
	const char *version = hashcash_version();
	printf("hashcash version: %s\n", version);
	return 0;
}
```

将上述代码保存为 test.c 文件，然后执行如下命令编译运行：
```bash
gcc -o test test.c -lhashcash
./test
```

如果看到如下输出就说明 Hashcash C 库是可以正常使用的了。
```bash
hashcash version: 1.22
```

## 使用 Rust FFI
### 查看 Hashcash 版本号
首先我们用 Rust 来实现跟刚才验证程序一样的功能：打印出 Hashcash 版本号。
```rust
#![feature(libc)]

extern crate libc;

use std::str;
use std::ffi::CStr;
use libc::c_char;

#[link(name = "hashcash")]
extern {
	fn hashcash_version() -> *const c_char;
}

fn main() {
	let version: &CStr = unsafe {
		let c_buf: *const c_char = hashcash_version();
		CStr::from_ptr(c_buf)
	};
	println!("hashcash version: {}", str::from_utf8(version.to_bytes()).unwrap());
}
```

同样保存为 test.rs 文件后执行如下命令验证：
```bash
rustc test.rs 
./test
```

运行之后便会看到和之前验证程序一样的输出。

### 产生 stamp
Hashcash 算法的核心是一串叫做 stamp 的字符串，如下所示：
```
1:20:150811:my_test::Kw6sW7wzgMGBNzSV:00000000001E3c
```
stamp 总共有7个域，相互之间像 IPv6 地址一样用冒号隔开。7个域的作用分别如下：
- 版本号 现在都是1
- 比特数 这个 stamp 保证其 [SHA](http://baike.baidu.com/view/531723.htm) 后能有几比特前导0
- 日期 产生 stamp 的时间
- 资源 stamp 所承载的信息
- 扩展 暂时保留
- 随机串 依据 a-zA-Z0-9+/= 这些字符随机生成的字符串
- 计数器 产生 stamp 总共尝试了多少次

Hashcash 是一种验证计算能力的算法，那么如何验证产生一个 stamp 的计算机的计算能力如何呢？只需要看`比特数`和`日期`这两个域即可。`日期`大概可以猜到是为了保证 stamp 的时效性，`比特数`是验证的关键，可以通过如下指令进行验证：
```bash
echo -n 1:20:150811:my_test::Kw6sW7wzgMGBNzSV:00000000001E3c | shasum
00000af7f14703c7b9168aaa468fc7cb3dfcd6bd
```
可以看到 `shasum` 输出的字符串前面有5个0，即20比特0，与`比特数`域所示一致，故该 stamp 有效。一个 stamp 进行 `shasum` 后出现前导几个数字都为0是一个极小概率的事件，而且随着0的个数的增加概率也随之降低， Hashcash 便是通过这样的方法验证计算能力的。下面就使用 Rust 编写一段程序来验证一下计算机的计算能力，代码如下：
```rust
#![feature(libc)]

extern crate libc;

use std::str;
use std::ptr;
use std::ffi::{CStr, CString};
use libc::{c_int, c_long, c_char, c_void, size_t};

#[link(name = "hashcash")]
extern {
    fn hashcash_simple_mint(resource: *const c_char,
        bits: size_t,
        anon_period: c_long,
        ext: *mut c_char,
        compress: c_int) -> *mut c_char;

    fn hashcash_free(ptr: *mut c_void);
}

fn main() {
    let (resource, bits) = (CString::new("my_test").unwrap(), 28);
    let stamp: &CStr = unsafe {
        let c_buf: *mut c_char = hashcash_simple_mint(resource.as_ptr(), bits, 0, ptr::null_mut(), 0);
        CStr::from_ptr(c_buf)
    };
    println!("hashcash stamp: {}", str::from_utf8(stamp.to_bytes()).unwrap());
    unsafe {
        hashcash_free(stamp.as_ptr() as *mut c_void);
    };
}
```

编译运行：
```rust
rustc test.rs 
time ./test
hashcash stamp: 1:28:150811:my_test::ZIkGrcoTk59SGw23:000000000JunhG

real    0m46.534s
user    0m46.289s
sys     0m0.122s
```

从上面可以看到我的电脑整整花了46秒的时间才找到符合要求的 stamp ，有兴趣你也可以试下哦！

### C 回调 Rust 函数
上面的例子只是简单地使用 Rust 调用 C 的接口，其实反过来也一样简单，下面是完成同样功能的一段代码，不过它实现了一个函数以供 Hashcash C 库回调。
```rust
#![feature(libc)]

extern crate libc;
extern crate time;

use std::str;
use std::ptr;
use std::ffi::{CStr, CString};
use libc::{c_int, c_long, c_ulong, c_char, c_double, c_void, size_t};
use time::now_utc;

#[link(name = "hashcash")]
extern {
    fn hashcash_free(ptr: *mut c_void);

    fn hashcash_mint(now_time: c_ulong,
        time_width: c_int, 
        resource: *const c_char,
        bits: size_t,
        anon_period: c_long,
        stamp: *const *mut c_char,
        anon_random: *mut c_long,
        tries_taken: *mut c_double,
        ext: *mut c_char,
        compress: c_int,
        cb: extern fn(percent: c_int,
            largest: c_int,
            target: c_int,
            count: c_double,
            expected: c_double,
            user: *const c_void) -> c_int,
        user_arg: *const c_void) -> c_int;
}

extern "C" fn callback(_percent: c_int, _largest: c_int,
            _target: c_int, count: c_double,
            _expected: c_double, _user: *const c_void) -> c_int {
    println!("mint count: {}", count);
    return 1;
}

fn main() {
    let (resource, bits) = (CString::new("my_test").unwrap(), 28);
    let now_time = now_utc().to_timespec().sec as u64;
    let stamp2: &CStr = unsafe {
        let c_buf: *mut c_char = ptr::null_mut();
        let user_arg: *const c_void = ptr::null();
        hashcash_mint(now_time, 6, resource.as_ptr(), bits, 0, &c_buf as *const *mut c_char, ptr::null_mut(), ptr::null_mut(), ptr::null_mut(), 0, callback, user_arg);
        CStr::from_ptr(c_buf)
    };
    println!("hashcash stamp: {}", str::from_utf8(stamp2.to_bytes()).unwrap());
    unsafe {
        hashcash_free(stamp2.as_ptr() as *mut c_void);
    };
}
```