#Match(模式匹配)
---
很多时候简单的 if/else 是不够用的，因为我们需要处理的情况超过两种。而且 else 会让情况变得异常复杂，有什么好的解决方案呢？

Rust 有一个关键字 `match`，让你能够处理复杂情况，请看：

	let x = 5;
	match x {
    	1 => println!("one"),
    	2 => println!("two"),
    	3 => println!("three"),
    	4 => println!("four"),
    	5 => println!("five"),
    	_ => println!("something else"),
	}