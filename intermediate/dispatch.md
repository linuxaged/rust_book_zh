#Trait Objects

当涉及到多态的时候，有一种判断到底应该调用哪个版本的机制，这种机制叫做调度（dispatch）。有两种调度，静态调度和动态调度。Rust 更倾向于使用静态调度，但是也支持动态调度；动态调度是通过 trait objects 来实现的。

#Backgroud

为了后面讨论方便，我们先写一个 trait 的例子：

	trait Foo {
	    fn method(&self) -> String;
	}

然后替 `u8` 和 `String` 类型实现这个 trait ：

	impl Foo for u8 {
	    fn method(&self) -> String { format!("u8: {}", *self) }
	}

	impl Foo for String {
	    fn method(&self) -> String { format!("string: {}", *self) }
	}

# 静态调度

我们可以通过 trait 绑定来实现静态调度：

	fn do_something<T: Foo>(x: T) {
	    x.method();
	}

	fn main() {
	    let x = 5u8;
	    let y = "Hello".to_string();

	    do_something(x);
	    do_something(y);
	}

这里 Rust 用了一种叫做 monomorphization 的方法来实现静态调度，就是生成了两个版本的 do_something ，然后在调用的地方替换成对应版本。也就是 Rust 生成了如下的代码：

	fn do_something_u8(x: u8) {
	    x.method();
	}

	fn do_something_string(x: String) {
	    x.method();
	}

	fn main() {
	    let x = 5u8;
	    let y = "Hello".to_string();

	    do_something_u8(x);
	    do_something_string(y);
	}

这种实现的优势在于方法调用可以内联，效率更高；弊端在于，如果有非常多版本的方法，二进制代码的体积会迅速膨胀。