值绑定
===

你可能第一次听到`绑定`这种说法，在 Rust 里绑定需要用到关键字 `let`

	fn main() {
		let x = 5;
	}

一看，咦，不就是别的程序语言里的赋值嘛！其实不然，Rust 里面的绑定除了赋值还有另一层含义：模式匹配(pattern matching)，等号两边的会自动对应起来。rust 可以写出这样的语句：

	let (x, y) = (1, 2);

这个表达式执行之后 x 变成 1, y 变成 2。模式匹配是 Rust 一个非常强大的语法，这个我们后面会详细的讲。

Rust 是一门静态类型语言，也就是说在赋值前要明确值的类型，上面那个例子可以编译通过是因为 rust 可以根据给出的值自动推导出类型(类似于 c++ 里的 auto)，当然你也可以明确一下类型：

	let x: i32 = 5;

这里我们用了 i32 类型，这是有符号32位整型，Rust 还有其他的整型。都是以 `i`（有符号） 或者 `u`（无符号） 开头后面跟位数，例如 `i64`，`u32` 等等。

值绑定默认创建的是不可变(immutable)类型，

	let x = 10;
	x = 11;

这么写是没法编译通过的，报错：

	error: re-assignment of immutable variable `x`
     x = 10;
     ^~~~~~~

如果需要一个可变类型的绑定，需要加上 mut 关键字 ：

	let mut x = 10;
	x = 11;
 
 关于绑定还有一个需要了解的是，在使用值绑定之前需要初始化：

 	fn main() {
	    let x: i32;

	    println!("Hello world!");
	}

这里我们只是声明了一个没有初始化的 x 绑定，没有使用它，编译器只是报了警：

	Compiling hello_world v0.0.1 (file:///home/you/projects/hello_world)
	src/main.rs:2:9: 2:10 warning: unused variable: `x`, #[warn(unused_variable)] on by default
	src/main.rs:2     let x: i32;
	                      ^

当我们使用这个没有初始化的绑定的时候，编译器就会报错了：

	fn main() {
	    let x: i32;

	    println!("The value of x is: {}", x);
	}

	$ cargo build
    Compiling hello_world v0.0.1 (file:///home/you/projects/hello_world)
	src/main.rs:4:39: 4:40 error: use of possibly uninitialized variable: `x`
	src/main.rs:4     println!("The value of x is: {}", x);
	                                                    ^
	note: in expansion of format_args!
	<std macros>:2:23: 2:77 note: expansion site
	<std macros>:1:1: 3:2 note: in expansion of println!
	src/main.rs:4:5: 4:42 note: expansion site
	error: aborting due to previous error
	Could not compile `hello_world`.
	
Rust 是不允许使用未经初始化的值的。另外我们在解释一下 `println!` 宏的用法。

第一个参数是一个字符串，里面还有一对大括号，这对大括号叫做 String Interpolation。它的作用相当于一个标记，就是告诉编译器我这有个位置你帮我填充一个值。这个要填充的值就是第二个参数，和第一个参数之间用逗号隔开。

Rust 可以打印[多种值](https://doc.rust-lang.org/nightly/std/fmt/) 目前我们只是打印最简单的整型。