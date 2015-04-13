#闭包

我们写了很多函数，他们都有一个名字。Rust 也能创建匿名函数。Rust 的匿名函数叫做闭包（closure）。

创建一个 闭包：

	let add_one = |x: i32| x + 1;

	assert_eq!(2, add_one(1));

我们使用 `|...|{...} ` 句法创建一个闭包，然后绑定一个名称。注意，我们使用绑定的名称加小括号来调用这个函数，和调用普通函数一样。

我们比较一下句法。很接近：

	let add_one = |x: i32| x + 1;
	fn  add_one   (x: i32) -> i32 { 1 + x }
	
你可能已经注意到，闭包可以推断出参数和返回值，因此没必要声明一个。这点和实名函数不一样。


# 闭包和上下文

之所以叫做闭包是因为它创建了一个作用域：

	let num = 5;
	let plus_num = |x: i32| x + num;

	assert_eq!(10, plus_num(5));
	
这里 plus_num 其实包含了后面闭包作用域里的 num 的一个绑定。当我们违反绑定规则就会报错，比如：

	let mut num = 5;
	let plus_num = |x: i32| x + num;

	let y = &mut num;

报错：

	error: cannot borrow `num` as mutable because it is also borrowed as immutable
	    let y = &mut num;
	                 ^~~
	note: previous borrow of `num` occurs here due to use in closure; the immutable
	  borrow prevents subsequent moves or mutable borrows of `num` until the borrow
	  ends
	    let plus_num = |x| x + num;
	                   ^~~~~~~~~~~
	note: previous borrow ends here
	fn main() {
	    let mut num = 5;
	    let plus_num = |x| x + num;
	    
	    let y = &mut num;
	}
	^
	
错误信息很明确的告诉我们不能再借用 num 了， 因为在当前上下文里它仍被 plus_num 所持有。明确闭包的上下文，可以解决这个问题：

	let mut num = 5;
	{
	    let plus_num = |x: i32| x + num;

	} // plus_num goes out of scope, borrow of num ends

	let y = &mut num;

有时候 num 会被强制加入闭包的上下文中：

	let nums = vec![1, 2, 3];

	let takes_nums = || nums;

	println!("{:?}", nums);

这里因为 Vector 类型是没法拷贝的，闭包拿走 nums 的所有权：

	note: `nums` moved into closure environment here because it has type
	  `[closure(()) -> collections::vec::Vec<i32>]`, which is non-copyable
	let takes_nums = || nums;
	                    ^~~~~~~

这和你把 Vector 作为参数传递给函数时候是一样。

#转移闭包(move closures)

Rust 还有另外一种闭包，叫做转移闭包。转移闭包用 move 关键字标示。转移闭包和普通闭包不同之处在于，转移闭包总是拿走所有用到变量的 ownership。相比之下普通闭包只是在当前栈帧内创建了一个引用。转移闭包最大的用处是和 Rust 的并发结合（参见并发章节）。这里暂且不表，后面在线程章节里面再谈论。

#闭包作为参数

闭包最为参数传递给一谈函数很有用，举个例子：

	fn twice<F: Fn(i32) -> i32>(x: i32, f: F) -> i32 {
	    f(x) + f(x)
	}

	fn main() {
	    let square = |x: i32| { x * x };

	    twice(5, square); // evaluates to 50
	}

可能有点小复杂，我们分解一下，首先我们定义了一个返回 i32 值并带有两个参数的模板函数 twice, 特殊之处在于这个模板是一个函数模板 `F: Fn(i32) -> i32` 也就是说任意一个函数只要满足，传入的是一个 i32 然后返回一个 i32 就行。twice 的第二个参数使用了前面定义的模板，意思就是这个参数必须是函数，且这个函数参数是 i32 返回值是 i32。

很显然闭包在这起到把一个函数传递给另一个函数的作用，这是一个非常强大的功能。

接下来讲一个闭包值得注意的地方，当我们同时传入两个闭包给同一函数时候

	fn compose<F, G>(x: i32, f: F, g: G) -> i32
	    where F: Fn(i32) -> i32, G: Fn(i32) -> i32 {
	    g(f(x))
	}

	fn main() {
	    compose(5,
	            |n: i32| { n + 42 },
	            |n: i32| { n * 2 }); // evaluates to 94
	}

你可能会纳闷为什么这要用两个模板 `F` 和 `G` 他们的标识符都是一样的呀，都是 `Fn(i32) -> i32`(参入一个 i32 返回一个 i32 的函数)嘛。其实，标识符只限定了闭包的入口和出口，两个闭包入口和出口虽然一样但内部的操作可能完全不同，比如上面一个是 `n + 42` 一个是 `n * 2`。所以 Rust 内部规定任意一个闭包都是单独一个特殊类型，不但不同标识符的闭包类型不一样，同样标识符的闭包类型也不一样。

这里引入了一个 where，用 where 来指定模板参数类型。

#闭包的实现

Rust 里的闭包其实是 trait 的语法糖，在阅读接下来的内容的时候请确保对 trait 有所了解。
理解闭包原理的关键在于意识到函数调用符号 () 是一个重载操作符。Rust 里使用 trait 系统来实现操作符重载，函数调用操作符也不例外

	pub trait Fn<Args> : FnMut<Args> {
	    extern "rust-call" fn call(&self, args: Args) -> Self::Output;
	}

	pub trait FnMut<Args> : FnOnce<Args> {
	    extern "rust-call" fn call_mut(&mut self, args: Args) -> Self::Output;
	}

	pub trait FnOnce<Args> {
	    type Output;

	    extern "rust-call" fn call_once(self, args: Args) -> Self::Output;
	}

开始学习闭包的时候可能会有点不习惯，一旦你开始使用并习惯使用的时候就会有一种爱不释手的感觉。在关键的地方闭包非常有用，比如后面会讲到的迭代器。
