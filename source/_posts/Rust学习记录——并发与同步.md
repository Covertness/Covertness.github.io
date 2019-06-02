title: Rust学习记录——并发与同步
urlname: Rust学习记录——并发与同步
date: 2015/05/30 21:11:23
categories:
- 学习记录
tags:
- Rust
- 并发

---
![](https://image.covertness.me/rust_bingfayutongbu_concurrency-movie-poster.jpg)

并发计算往往带来很多棘手的问题， Rust 通过严格的规范使得部分问题能够尽早地被发现。
<!-- more -->

随着处理器的核心数日益增多，程序的并发能力逐渐成为其性能提升的关键，但数据不同步、死锁等很多问题让不少人对并发编程敬而远之。不过令人欣慰的是现在越来越多的语言开始从基础层面加强对并发的支持，让并发处理不再那么困难。 Rust 通过更严格的编译期检查对并发编程提供了更强有力的支持，使得并发代码不再那么容易出问题。

参考下面的这段 C 程序代码，它实现了如下逻辑：
> 假设售票点总共有5000张余票，每张售价20元，10个窗口同时售票，不断有人购买直到售完为止。为了实现方便，这里假设所有买票的人都从同一个银行帐号取钱购买，且该银行帐号刚好仅有100000元。

```C
#include <stdio.h>
#include <pthread.h>
#include <string.h>
#include <stdlib.h>

#define THREAD_NUM 10

static int money = 100000;
static int ticket = 5000;

static void* buy(void* arg) {
	while (1) {
		if (ticket <= 0 || money <= 0) {
			break;
		}

		money -= 20;
		--ticket;
	}

	pthread_exit(NULL);
}

int main() {
	pthread_t tid[THREAD_NUM];

	for (int i = 0; i < THREAD_NUM; ++i) {
		int err = pthread_create(&(tid[i]), NULL, &buy, NULL);
        if (err != 0) {
            printf("\ncan't create thread :[%s]", strerror(err));
            exit(1);
        }
	}

	for (int i = 0; i < THREAD_NUM; ++i) {
		pthread_join(tid[i], NULL);
	}

	printf("money: %d\nticket: %d\n", money, ticket);
	return 0;
}
```

由于银行帐号的余额刚好够买完所有的余票，预期结果应该是`money`和`ticket`均为0，但如果在多核机器上测试该程序会发现大多数时候测试结果与预期并不相符，且结果似乎没有任何规律。通过对代码的分析不难发现是对`money`和`ticket`的操作没有加锁而导致的问题。此类问题在并发程序中很容易出现，而且往往出现在程序正常运行一段时间后，因而给定位这类问题的原因增加了不少难度。下面使用 Rust 重写这段逻辑：

```Rust
use std::thread;

fn main() {
	let (mut money, mut ticket) = (100000, 5000);

	let mut handles = Vec::new();

	for _ in 0..10 {
		let handle = thread::spawn(|| {
			loop {
			 	if money <= 0 || ticket <= 0 {
			 		break;
			 	}

			 	money -= 20;
				ticket -= 1;
			}
		});

		handles.push(handle);
	}

	for h in handles {
        h.join().unwrap();
    }

    println!("money: {}", money);
    println!("ticket: {}", ticket);
}
```

编译此程序时会发现如下错误：
```bash
book_ticket.rs:9:30: 18:4 error: closure may outlive the current function, but it borrows `money`, which is owned by the current function [E0373]
book_ticket.rs:9 		let handle = thread::spawn(|| {
book_ticket.rs:10 			loop {
book_ticket.rs:11 			 	if money <= 0 || ticket <= 0 {
book_ticket.rs:12 			 		break;
book_ticket.rs:13 			 	}
book_ticket.rs:14
```

编译器指出了问题所在，执行比较`money`操作的函数没有`money`的所有权( ownership )。在 Rust 中不同的线程拥有不同的所有权，它们的堆栈并不能直接共享，这是语言特性所规定的，因而在编译期就发现了上述的数据同步问题。
Rust 中给出的解决方案也是比较灵活的，既可以使用传统的锁方案（参考[官方文档](https://doc.rust-lang.org/stable/book/concurrency.html#safe-shared-mutable-state)），也可使用基于消息传递的方案（参考[官方文档](https://doc.rust-lang.org/stable/book/concurrency.html#channels)）,依据具体的业务场景选用合适的方案即可。