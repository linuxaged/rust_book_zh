Borrow 和 AsRef
===

Borrow 和 AsRef 这两个 trait 相似又有区别。

#Borrow

在你设计数据结构的时候，如果希望数据既可以被持有又可以被借贷就要用到 Borrow 这个 trait. 例如 HashMap 有个 [get](https://doc.rust-lang.org/nightly/std/collections/struct.HashMap.html#method.get) 方法用到了 Borrow：

	fn get<Q: ?Sized>(&self, k: &Q) -> Option<&V>
	    where K: Borrow<Q>,
	          Q: Hash + Eq

大多数时候用 `&T` 来表示持有或者借贷类型就够了，但当我们要同时表示多种借贷类型的时候，Borrow 就非常有用了。例如你要设计一个方法能同时处理 &T 和 &mut T 类型：

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

AsRef 用来将值类型转换成引用类型, 类似 C# 装箱（Boxing）：

	let s = "Hello".to_string();

	fn foo<T: AsRef<str>>(s: T) {
	    let slice = s.as_ref();
	}

#Borrow vs. AsRef

那在什么情况下应该用这个而不是另一个呢？

