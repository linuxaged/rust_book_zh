我们已经是用了一些模式了，比如 let 比如 match 现在我们看一下 match 到底能做哪些事情！



这种匹配使用与任意成员，不仅仅是第一个：

	struct Point {
	    x: i32,
	    y: i32,
	}

	let origin = Point { x: 0, y: 0 };

	match origin {
	    Point { y: y, .. } => println!("y is {}", y),
	}

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