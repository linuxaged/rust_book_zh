% 指针

Rust 指针是其众多特性中格外引人注目的一个，当然，也格外困扰 Rust 新手。即使你拥有 C++ 背景也会有不同程度的困惑。这篇手册有助你理解指针这个重要主题。

我们这有个[速查表](http://doc.rust-lang.org/book/pointers.html#cheat-sheet)可以帮助你了解指针的类型、名称、作用。

#介绍
如果你不了解指针，这儿提供一个简介。指针在系统级编程语言里是非常基础的部分，理解它很重要。

##指针基础
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



#指针的使用

Rust 的指针非常有用，但用法上有别于其他语言。在讨论 Rust 指针的最佳实践之前我们来看看其他语言对指针的使用：

在 C 语言中字符串其实一个指向以 null 字符结尾的char数组的指针，使用字符串类型的唯一方式就是熟悉指针。

指针主要的作用就是指向非栈上的内存地址。前面的例子中使用两个栈上的变量，我们可以直接绑定名称。当我们在堆上分配内存的时候，我们就不能直接给它绑定名字了。在 c 里面使用 malloc 分配堆上内存，它返回一个指针。

综合前面讲的两点，每当你需要能动态改变大小的数据结构的时候就需要引入指针。编译器无法在编译的时候就分配堆上内存，所以我们编译的时候要用指针来指代它（将会在程序运行时候分配），在运行时也用这个指针来做一些操作。

#常见指针问题

我们已经讨论完指针并且指出了它的优点，那指针有哪些弊端呢？我们来看一下 Rust 一定程度上解决的指针问题：

未初始化的指针导致的问题，例如下面程序执行会发生什么就不好说了：

	%int x;
	*x = 5;
我们声明了一个指针，但没有指向任何位置，然后设置它指向位置的值为 5 。没有人知道这个没有指定的位置在哪，可能无关紧要也可能导致灾难性的后果。

当你结合函数使用指针的时候，非常容易让指针指向的内存变得无效，比如：

	func make_pointer(): &int {
		x = 5;
		return &x;
	}
	func main() {
		&int j = make_pointer();
		*j = 5; // 错误！
	}

x 是 make_pointer 函数的局部变量，在函数返回后立即失效（被释放）。但我们返回了一个指向它的指针，并且试图在 main 函数里使用这个指针，这种情况和使用未初始化的指针类似。后果不可预料。

最后一个问题，当多个指针指向同一个内存的时候会有问题。譬如：

	func mutate(&int i, int j) {
    	*i = j;
	}

	func main() {
  		x = 5;
  		y = &x;
  		z = &x; //y and z are aliased

  		run_in_new_thread(mutate, y, 1);
  		run_in_new_thread(mutate, z, 100);

  		// what is the value of x here?
	}

这里的 run_in_new_thread 创建一个新线程，在新线程里调用 mutate 函数并传入参数。我们这有两个线程，它们都通过 metate 函数操作 x 的引用，我们不能确定哪个线程先完成，x 的值也就不确定了。还可能有更糟的情况，如果有一个引用指向了非法地址，我们就会像前面说到的那样的对一个非法地址进行操作，后果变得无法预料。

#Boxes

`Box<T>` 表示一个装箱指针（boxes pointer 在堆上分配数据返回指针）：

	let x = Box::new(5);

虽然是在堆上分配，但是离开作用域后被自动释放：

	{
	    let x = Box::new(5);

	    // stuff happens

	} // 在这被释放

box 并没有像其他语言里那样使用析构函数或者垃圾回收。Box 实际上是一个仿射类型(affine type)。也就是说编译器会检测 Box 的作用域范围，然后插入响应的代码。更进一步， Box 事实上是一种特殊的仿射类型叫做 [region](http://www.cs.umd.edu/projects/cyclone/papers/cyclone-regions.pdf)。



#总结

到这我们对指针有了个大概的认识。我们之前提到 Rust 设计了多种指针，并且避免了我们前面提到的问题。同时，Rust也会比其他只有一种指针类型的语言要复杂，鉴于 Rust 解决了简单指针的问题，这点复杂性是可以接受的。

#引用

引用是 Rust 里最基本的一种指针， 记作：

	let x = 5;
	let y = &x;

	println!("{}", *y);
	println!("{:p}", y);
	println!("{}", y);

我们称 y 是 x 的引用。第一个 println! 通过解引用符号 * 打印出 y 引用对象的值，第二个会打印出 y 指向对象的地址，这里用了一个 :p 指明是指针类型。第三个同样会打印出 y 引用对象的值，println! 会自动为我们解引用。

下面是一个含有引用类型参数的函数：

	fn succ(x: &i32) -> i32 { *x + 1 }

你可以用 & 符号创建引用，因此我们有两种方式调用这个函数：

	fn succ(x: &i32) -> i32 { *x + 1 }

	fn main() {
	    let x = 5;
	    let y = &x;
	    println!("{}", succ(y));
	    println!("{}", succ(&x));
	}

两次调用都会打印出 6.

实际上这里完全可以不用引用：

	fn succ(x: i32) -> i32 { x + 1 }

引用默认是不可变类型：

	let x = 5;
	let y = &x;

	*y = 5; // error: cannot assign to immutable borrowed content `*y`

同样：

	let x = 5;
	let y = &mut x; // error: cannot borrow immutable local variable `x` as mutable

不可变指针是可以起别名的：

	let x = 5;
	let y = &x;
	let z = &x;

可变指针则不可以：

	let mut x = 5;
	let y = &mut x;
	let z = &mut x; // error: cannot borrow `x` as mutable more than once at a time

因为提供了这些安全检查，在运行时引用和传统的之中呢并没有区别。这些安全检查都是在编译期进行的，所以没有任何额外的运行时开销。这种原理叫做 region pointers -- 逐渐进化成了我们现在的*生命周期*。

#模式和 ref

当你对指针指向的内容进行匹配的时候，直接匹配并不是最好选择，

	fn possibly_print(x: &Option<String>) {
	    match *x {
	        // BAD: cannot move out of a `&`
	        Some(s) => println!("{}", s)

	        // GOOD: instead take a reference into the memory of the `Option`
	        Some(ref s) => println!("{}", *s),
	        None => {}
	    }
	}

这里 `ref s` 表示一个 &String 类型，而不是 String。这在你只是想引用一个对象而不是完全占有的时候很有用。

#速查表

这儿有个所有 Rust 指针类型的概述

|类型		|名称				|概述 									|
|-----------|-------------------|---------------------------------------|
|`&T`		|引用				|允许一个或者多个对象读 T 					|
|`&mut T`	|可变引用				|允许一个对象读写 T 						|
|`Box<T>`	|Box				|只允许一个持有者读写堆上分配的 T.			|
|`Rc<T>`	|引用计数指针			|可供多个对象读堆上分配的 T 					|
|`Arc<T>`	|线程安全的引用计数指针	|同上但是线程安全							|
|`*const T`	|裸指针				|读 T 是危险的							|
|`*mut T`	|可变裸指针			|读写 T 是危险的							|

#相关资源

	* [Box API 文档]()

	* [所有权系统指南]()

	* [Cyclone paper on regions]()