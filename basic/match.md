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

match 后面跟一个表达式，然后根据表达式的值来建立分支，每个分支的形式都是 `val => expression`. 值匹配，分值就会被执行。

使用 match 的最大好处在于编译器会帮你检查是否覆盖了所有的情况，如果我把上面最后一个分支去掉，编译器报错：

	error: non-exhaustive patterns: `_` not covered

'_' 的意思是“剩余所有情况”，和前面形成互补，覆盖了所有可能的情况。