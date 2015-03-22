和其他语言一样 Rust 提供了非常多的内置类型。我们已经学习整型、字符串类型的一些简单用法，现在讨论复杂一点的数据表示方式。

##tuples
我们第一个讨论的复合数据类型叫做 tuple，tuple 是一个固定大小的有序列表：

	let x = (1, "hello");

这儿构造了一个长度为二的 tuple，如果要把 tuple 的类型标注出来的话：

	let x: (i32, &str) = (1, "hello");

相信你已经看出来了，tuple 类型就是这个 tuple 各个成员类型的列表。这有个 `&str` 类型你之前可能没有见过，这人先告诉你这叫做字符串切片（string slice）,后面讲字符串高级部分的时候会讲到。

我们说tuple 是个有序列表就是说在比较两个 tuple 的时候，不但他们各个成员的类型和值要一样，连他们出现的顺序也必须一样，他们才会相等。

	let x = (1, 2, 3);
	let y = (2, 2, 4);

	if x == y {
	    println!("yes");
	} else {
	    println!("no");
	}

只要有一个成员不一样就不等。

	let x = (1, 2, 3);
	let y = (2, 1, 3);

	if x == y {
	    println!("yes");
	} else {
	    println!("no");
	}

成员的顺序不一样也不等。

tuple 另外一个非常重要的功能，当我们的函数需要返回多个值时，我们可以返回一个 tuple。

	fn next_two(x: i32) -> (i32, i32) { (x + 1, x + 2) }

	fn main() {
	    let (x, y) = next_two(5);
	    println!("x, y = {}, {}", x, y);
	}

#struct

struct 是比 tuple 更常见更通用的类型，他们的区别在于 struct 有名字 tuple 没有：

	let black = (255,255,255); // tuple

	struct Color(i32, i32, i32);
	let black = Color(255,255,255); // struct



这里 black != origin

#tuple struct

Rust 还有一种复合数据类型叫做 tuple struct, 它自己有名字但是成员没有名字：

	struct Color(i32, i32, i32);
	struct Point(i32, i32, i32);

这里要注意，不同的名字代表不同的类型，即使他们里面的数据都一样：

	let black = Color(0, 0, 0);
	let origin = Point(0, 0, 0);

绝大多数情况下你应该使用 struct 而不是 tuple struct。通常我们把 Color 和 Point 写作：

	struct Color {
	    red: i32,
	    blue: i32,
	    green: i32,
	}

	struct Point {
	    x: i32,
	    y: i32,
	    z: i32,
	}

用名称将成员区别开来有利于减少错误发生。

但是有一种 tuple struct 非常有用，就是只包含一个数据成员的 tuple struct:

	struct Inches(i32);

	let length = Inches(10);

	let Inches(integer_length) = length;
	println!("length is {} inches", integer_length);

这相当于给 i32 起了个别名，同时也创建一个新类型叫 Inches

#enum
