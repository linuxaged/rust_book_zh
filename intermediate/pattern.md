模式（Patterns）
===

我们前面讲到的 `let` 和 `match` 其实已经用到了模式。这个现在我们看一下 `match` 的一些高级用法。

首先 `match` 可用于匹配字面量（literals），结合 `_` 可以方便地表达一些条件匹配：

	let x = 1;

	match x {
	    1 => println!("one"),
	    2 => println!("two"),
	    3 => println!("three"),
	    _ => println!("anything"),
	}

你还可以用 `|` 同时匹配多个模式：

	let x = 1;

	match x {
	    1 | 2 => println!("one or two"),
	    3 => println!("three"),
	    _ => println!("anything"),
	}

更强大的是，你可以用 `...` 匹配一个范围：

	let x = 1;

	match x {
	    1 ... 5 => println!("one through five"),
	    _ => println!("anything"),
	}

范围匹配常用于 int 型和字符型。

如果你用 | 或者 ... 同时匹配多个值的话，可以使用 @ 来给他们绑定个名称：

	let x = 1;

	match x {
	    e @ 1 ... 5 => println!("got a range element {}", e),
	    _ => println!("anything"),
	}

在匹配一个枚举类型的时候，你可以用 .. 忽略掉枚举的值和类型：

	enum OptionalInt {
	    Value(i32),
	    Missing,
	}

	let x = OptionalInt::Value(5);

	match x {
	    OptionalInt::Value(..) => println!("Got an int!"),
	    OptionalInt::Missing => println!("No such luck."),
	}

你还可以用 if 来控制 match:

	enum OptionalInt {
	    Value(i32),
	    Missing,
	}

	let x = OptionalInt::Value(5);

	match x {
	    OptionalInt::Value(i) if i > 5 => println!("Got an int bigger than five!"),
	    OptionalInt::Value(..) => println!("Got an int!"),
	    OptionalInt::Missing => println!("No such luck."),
	}

##指针和引用的匹配

当你 match 指针类型的时候，应该加上 & 符号：

	let x = &5;

	match x {
	    &val => println!("Got a value: {}", val),
	}

如果需要对引用进行 match 应该使用 ref 关键字：

	let x = 5;

	match x {
	    ref r => println!("Got a reference to {}", r),
	}

如果你还要这个引用是可变的，那么加上 mut:

	let mut x = 5;

	match x {
	    ref mut mr => println!("Got a mutable reference to {}", mr),
	}

##匹配 struct 中的成员

甚至你还可以对 struct 里面的成员进行匹配：

	struct Point {
	    x: i32,
	    y: i32,
	}

	let origin = Point { x: 0, y: 0 };

	match origin {
	    Point { x: x, y: y } => println!("({},{})", x, y),
	}

如果我们只关心某一个成员的话可以用 `..` 符号忽略其余成员：

	struct Point {
	    x: i32,
	    y: i32,
	}

	let origin = Point { x: 0, y: 0 };

	match origin {
	    Point { x: x, .. } => println!("x is {}", x),
	}

##匹配数组和切片

如果你需要匹配切片或者数组的话，就要用 `&`:

	fn main() {
	    let v = vec!["match_this", "1"];

	    match &v[..] {
	        ["match_this", second] => println!("The second element is {}", second),
	        _ => {},
	    }
	}

总之有着非常多匹配东西的方法，这些方法还可以相互组合相互匹配，看你怎么用了。

match x {
    Foo { x: Some(ref name), y: None } => ...
}

模式是非常强大的，好好利用！