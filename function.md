#函数

其实你已经见过一个函数了， main 函数：

	fn main() {
	
	}

这是最简单的函数声明，fn 表示这是一个函数，后面跟着函数的名称，如果没有参数的话，() 内为空，{} 表示函数体，再比如一个叫做 foo 的函数：

	fn foo() {

	}

如果有参数的话，比如打印一个数字：

	fn print_number(x: i32) {
		println!("x is: {}", x);
	}

看一下完整版本：

	fn main() {
		print_number(5);
	}

	fn print_number(x: i32) {
		println!("x is: {}", x);
	}

参数的写法和 let 类似，名称在前，类型在后，中间用 : 符号连接。

再写一个把两个数字相加的函数：

	fn main() {
		print_sum(5, 6);
	}

	fn print_sum(x: i32, y: i32) {
		println!("sum is: {}", x + y);
	}

声明和调用的时候不同参数之间都是用逗号隔开。

和 let 有所不同的是，你必须指定参数的类型。不指定没法编译通过：

	fn print_sum(x, y) {
		println!("sum is: {}", x + y);
	}

报错：
	
	hello.rs:5:18: 5:19 expected one of `!`, `:`, or `@`, found `)`
	hello.rs:5 fn print_number(x, y) {

理论上如果不指定参数类型，也能在调用的时候推断出来。某些语言就是这么干的，比如 haskell。不过这些语言也提倡在函数声明的时候就指定参数类型。还有一些与语言在函数体内也需要指定参数类型。Rust 设计者认为只在声明时候强制需要指定参数类型是一个很好地折中。

我们再看一下带返回值的函数怎么写：

	fn add_one(x: i32) -> i32 {
		x + 1
	}

Rust 函数只能有一个返回值，在 `->` 这个箭头符号后面声明。

你可能发现这里面少了一个分号，如果加上一个分号：

	fn add_one(x: i32) -> i32 {
		x + 1
	}

会报错：

	error: not all control paths return a value
	fn add_one(x: i32) -> i32 {
	     x + 1;
	}

	help: consider removing this semicolon:
	     x + 1;
	          ^

还记得我们之前关于 `;` 和 `()` 的讨论吗，如果加上了这个分号我们的函数就变成了返回 `()`。这不是我们原意，所以编译器会建议我们去掉分号。

这很像我们的 if 声明： {} 块里面的计算结果就是返回值。Ruby 也使用了类似的语法，在系统编程语言里确实比较少见。人们开始的时候会认为这种语法会导致 BUG，但不要忘记 Rust 有非常健壮的类型系统，另外 () 也是一种特殊的类型，我还没见过因为遗漏或者多加了分号而导致的 BUG。

如果我们在函数运行途中就要返回呢？Rust 是提供了 return 关键字的：

	fn foo(x: i32) -> i32 {
    	if x < 5 { return x; }
    	x + 1
	}