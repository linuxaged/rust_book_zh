条件编译

Rust 提供了一个特殊的属性 #[cfg]，使用它你可以通过传参给编译器来控制代码的编译，它有两种形式：

	#[cfg(foo)]

	#[cfg(bar = "baz")]

还有一些帮助：

	#[cfg(any(unix, windows))]

	#[cfg(all(unix, target_pointer_width = "32"))]

	#[cfg(not(foo))]
