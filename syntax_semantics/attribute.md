属性

我们可以给声明添加属性标记，例如：

	#[test]

或者

	#![test]

这两种添加属性的方式的唯一区别在于：#[test] 作用于后面的声明；#![test] 作用于它被包裹的那个声明（这里是 mod 声明）。这两者起到的作用是一样的，就是改变了声明的含义，比如：

	#[test]
	fn check() {
	    assert_eq!(2, 1 + 1);
	}

这里 check 函数被标记上了 test 属性，当你运行 tests 的时候这个方法就会被调用。正常编译的时候会忽略这个函数，他就成了一个测试函数。当然还有其他风格的属性，例如包含限定词的属性：

	#[inline(always)]
	fn super_fast_fn() {

包含键值的属性：

	#[cfg(target_os = "macos")]
	mod macos_only {

不同的属性有着不同的用途，你可以参考这个[手册](https://doc.rust-lang.org/nightly/reference.html#attributes)。目前你还不能自定义属性，只能使用编译器替你设定好的。