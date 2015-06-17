Borrow 和 AsRef
===

Borrow 和 AsRef 这两个 trait 相似又有区别。

#Borrow
	use std::borrow::Borrow;
	use std::fmt::Display;

	fn foo<T: Borrow<i32> + Display>(a: T) {
	    println!("a is borrowed: {}", a);
	}

	let mut i = 5;

	foo(&i);
	foo(&mut i);

结果会打印打次 "a is borrowed: 5"

#AsRef

AsRef 用来将值类型转换成引用类型：

	let s = "Hello".to_string();

	fn foo<T: AsRef<str>>(s: T) {
	    let slice = s.as_ref();
	}

#Borrow vs. AsRef

那在什么情况下应该用这个而不是另一个呢？

