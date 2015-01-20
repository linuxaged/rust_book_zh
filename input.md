#标准输入

获取键盘输入很简单，我们来看一下简单的将键盘的输入打印出来是怎么写的：

	fn main() {
	    println!("Type something!");

	    let input = std::io::stdin().read_line().ok().expect("Failed to read line");

	    println!("{}", input);
	}

我们一句一句的看，首先：

	std::io::stdin();

这是从 std::io 模块里调用 stdin() 函数。std 里面的东西都是由 Rust 标准库提供的。后面我们会降到模块系统。

每次调用都要输入前面的路径很麻烦，和 C++ 一样我们可以用 use 将函数导入进来：

	use std::io::stdin;
	stdin();

但我们基本上不会只用某个模块里的一个函数，所以我们常常是导入一个模块：

	use std::io;
	io::stdin();

用这种方法把例子重写一下：

	use std::io;
	fn main() {
	    println!("Type something!");
	    let input = io::stdin().read_line().ok().expect("Failed to read line");
	    println!("{}", input);
	}

好了，我们接着看下一句：

	read_line()

在 stdin() 返回的结果上调用这个函数将会得到输入的一整行内容。很简洁。

	.ok().expect("failed to read line");

还记得下面这个吗：

	enum OptionalInt {
		Value(i32),
		Missing,
	}

	fn main() {
	    let x = OptionalInt::Value(5);
	    let y = OptionalInt::Missing;

	    match x {
	        OptionalInt::Value(n) => println!("x is {}", n),
	        OptionalInt::Missing => println!("x is missing!"),
	    }

	    match y {
	        OptionalInt::Value(n) => println!("y is {}", n),
	        OptionalInt::Missing => println!("y is missing!"),
	    }
	}