title: Rust学习记录——内存所有权
urlname: Rust学习记录——内存所有权
date: 2015/06/06 20:11:53
categories:
- 学习记录
tags:
- Rust
- 内存管理
- Ownership
- Borrowing

---
![](https://image.covertness.me/rust_neicunsuoyouquan_20120202th-memory-freer-apple-manager-cleaner.jpg)

Rust 基于 RAII 的内存管理机制让内存的使用变得既高效又安全。
<!-- more -->

虽然计算设备的内存大小随着时间的推移一直成几何倍的增长，但内存始终是造成程序各种运行时问题的主要因素之一，因而在编程语言几十年的发展过程中一直在尝试各种方法来帮助程序更加有效地管理内存。
众所周知，程序可用的内存区域主要被分为栈区域和堆区域：栈区域在程序编译阶段便固定下来，使用方便但是大小固定；堆区域用于在程序运行期间需要更多内存时使用，可动态扩展但需要自行管理。随着程序处理的事务越来越多，堆区域使用愈加频繁，随之而来的问题也越来越多：内存泄漏、非法地址请求、内存碎片等等。因而希望内存能够自动化管理的需求越来越强烈，一种叫做垃圾回收（ Garbage Collectors ）的内存管理机制应运而生。
垃圾回收通过扫描内存来发现不再需要的内存区块然后自动释放它们，这种扫描一般会根据需要不定期的被触发。这种方法使得程序不再需要手动地去释放内存，大大减少了很多内存管理方面的问题。但由于这种方法减弱了程序对内存的控制能力，随之也带来了一些新的问题。垃圾回收过程需要占用一定的 CPU 时间，而且不受程序直接控制，这对于比较繁忙的场景（如 Web 后台程序）以及实时性要求比较高的场景（如 IoT 系统）会造成一定的干扰，有时甚至会大大降低系统的稳定性。鉴于此一些编程语言选择了另外一种解决思路——使用 RAII 来解决内存管理的问题。
RAII 基于面向对象编程（ OOP ）的思维，在对象初始化时申请内存资源，在对象销毁时释放内存资源，使得内存分配情况固定下来，不再需要动态化的内存管理手段，保证了程序运行时期的稳定性。 Rust 通过[所有权（ Ownership ）](https://doc.rust-lang.org/stable/book/ownership.html)的概念实现了基于 RAII 的内存管理机制。在 Rust 中变量对其所持有的内存拥有所有权，而且同一时间仅允许一个变量拥有同一块内存资源，当变量被销毁时其拥有的内存资源随即也被释放。不过由于所有权是唯一的，这可能会给资源的使用带来困难，为了解决这个问题， Rust 引进了[租借（ Borrowing ）](https://doc.rust-lang.org/stable/book/references-and-borrowing.html)的概念。一个变量所拥有的内存资源可以租借给另外一个变量，借到资源的变量可以读写这个资源，类似于 C++ 中的引用，不同的是租借者只允许有一个，而且租借期间原来的所有者不能改变此资源。 对于一些更加复杂的情况（如多线程环境）， Rust 也通过其他的特性规范（如 Send 和 Sync）予以支持。 Rust 通过这些方式不但解决了内存管理自动化的问题，而且还能帮助程序避免由于内存竞争导致的运行时问题。
下面是在其他语言中稍不注意便会出现的一个内存竞争问题。遍历列表的过程中删除一个匹配的值，下面是一个使用 Java 编写的例子：
```java
import java.util.HashMap;
import java.util.Map;

public class MapRemove {
    private static Map<Integer, String> map = new HashMap<Integer, String>();

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++){
            map.put(i, "value" + i);
        }
 
        for (Map.Entry<Integer, String> entry : map.entrySet()) {
            Integer key = entry.getKey();
            System.out.println("Check the key " + + key);

            if (entry.getValue().equals("value5")) {
                map.remove(key);
                System.out.println("The key " + + key + " was deleted");
            }
        }

        for (Map.Entry<Integer, String> entry : map.entrySet()) {
            System.out.println(entry.getKey() +" = " + entry.getValue());
        }
    }
}
```

程序逻辑看上去没有任何问题，但运行此程序时会发现在删除操作后抛出如下的异常：
```bash
Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.HashMap$HashIterator.nextEntry(HashMap.java:922)
	at java.util.HashMap$EntryIterator.next(HashMap.java:962)
	at java.util.HashMap$EntryIterator.next(HashMap.java:960)
	at MapRemove.main(MapRemove.java:12)
```

原来是删除操作导致原来的迭代器失效进而引发的异常，这类迭代器的实现机制导致的问题理应由语言本身或者其标准库负责，但这里却导致了程序运行时的崩溃。同样逻辑的 Rust 代码如下：
```rust
use std::collections::HashMap;

fn main() {
	let mut map = HashMap::new();

	for i in 0..9 {
		map.insert(i, "value".to_string() + &i.to_string());
	}

	for (key, value) in map.iter() {
		println!("Check the key {} ", key);

		if *value == "value5" {
			map.remove(key);
			println!("The key {} was deleted", key);
		}
	}

	for (key, value) in map.iter() {
		println!("{} = {}", key, value);
	}
}
```

编译这段代码时 Rust 编译器便会打印如下错误：
```bash
map_remove.rs:15:4: 15:7 error: cannot borrow `map` as mutable because it is also borrowed as immutable
map_remove.rs:15 			map.remove(key);
```

由于此时`map`所有的资源已经被租借了，因而`map`自己暂时是不能改变其所有的资源的。 Rust 的语法规则又一次帮助程序纠正了潜在的错误。