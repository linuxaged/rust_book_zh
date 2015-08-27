#类型别名

你可以用 `type` 关键字声明一个类型的别名：

	type Name = String;

然后使用这个别名：

	let x: Name = "Hello".to_string();

因为这只是个别名，事实上是同一个类型，所以在比较类型的时候，得到的结果是一致的：

	type Num = i32;

	let x: i32 = 5;
	let y: Num = 5;

	if x == y {
	   // ...
	}

我们经常把类型别名和模板放一起使用：

	use std::result;

	enum ConcreteError {
	    Foo,
	    Bar,
	}

	type Result<T> = result::Result<T, ConcreteError>;

这里我们定制一个 Result<T> 类型，用 ConcreteError 取代了 E。这个技巧在标准库里应用广泛。