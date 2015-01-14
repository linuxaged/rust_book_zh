#指针
---

Rust 指针是其众多特性中格外引人注目的一个，也格外困扰 Rust 新手。即使你拥有 C++ 背景也会有不同程度的困惑。这篇手册有助你理解指针这个重要主题。

我们这有个[速查表](http://doc.rust-lang.org/book/pointers.html#cheat-sheet)可以帮助你了解指针的类型、名称、作用。

##介绍
---
如果你不了解指针，这儿提供一个简介。指针在系统级编程语言里是非常基础的部分，理解它很重要。

###指针基础
---
当你创建值绑定的时候其实是在给栈上分配的一个值命名。（如果不知道什么是栈和堆请移步 [stackoverflow](http://stackoverflow.com/questions/79923/what-and-where-are-the-stack-and-heap)) 例如：

	let x = 5i;
	let y = 8i;
	
|  location | value  |
|-----------|--------|
|  0xd3e030 |  5     |
|  0xd3e028 |  8     |

这里我们标记处了值的地址，`x` 对应于内存 0xd3e030 处 的 `5`。当我们提到 `x` 时，它的值就是 `5`.

让我们引进一个指针。在一些语言里，只有一种指针，但在 Rust 里我们有好几种，这儿我们使用 Rust 的引用，这是最简单的一种指针。

	let x = 5i;
	let y = 8i;
	let z = &y;
	
	
|  location | value  |
|-----------|--------|
|  0xd3e030 |  5     |
|  0xd3e028 |  8     |
|  0xd3e020 | 0xd3e028 |

看到不同之处了吗？区别于直接保存值，指针保存的是一个内存的一个地址，在这里 z 保存的是 y 的地址。x , y 是 int 型，而 z 是 &int 型。我们可以用 `{:p}` 打印地址：

	println!("{:p}",z);
	
这会打印 0xd3e028.

因为 int 和 &int 是不同的类型，我们不能将它们相加：

	println!("{}", x + z)
	
这会报错：

	hello.rs:6:24: 6:25 error: mismatched types: expected `int` but found `&int` (expected int 	but found &-ptr)
	hello.rs:6     println!("{}", x + z);
                                  ^
我们可以用 `*	` 符号解引用指针。解引用就是获取指针保存的地址指向的值，

	let x = 5i;
	let y = 8i;
	let z = &y;
	
	print!("{}", x + *z);
	
打印出 13.

好了，总结一下，指针就是指向某个内存地址。我们讨论完了什么是指针，接下来看一下为什么会有指针。



##指针的使用
---
