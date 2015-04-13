模板
===

有时候我们想要自己的函数最大程度的复用，能适用于多种数据类型。还记得我们的 OptionalInt 类型吗？

	enum OptionalInt {
		Value(i32),
		Missing,
	}
	
如果我们还要一种 OptionalFloat64，

	enum OptionalFloat64 {
    	Valuef64(f64),
    	Missingf64,
	}

这么写很啰嗦。我们可以用 Rust 的模板：

	enum Option<T> {
		Some(T),
		None,
	}
	
之前你可能见过这个 <T>，这是一个模板参数，在你使用 Option 的时候他会把 Option 内的 所有 T 替换成你指定的 T.