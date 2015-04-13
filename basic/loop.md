循环
===

Rust 有两种方式表示循环：for 和 while.

##for

for 用于固定次数的循环，Rust 的for循环和其他语言不太一样，不同于 C 语言的for循环：

	for (x=0; x < 10; x++) {
		printf("%d\n", x );
	}
	
Rust 的写法是这样子：

	for x in 0..10 {
		println!("{}",x ); // x: i32
	}
	
语法是：

	for var in expression {
		code
	}
	
expression 是一个迭代器，我们会在进阶章节里面谈论这个。迭代器返回一个序列，序列里每个元素和 var 名称绑定然后参与一次循环的迭代，一次迭代结束迭代器会获取下一个，直到所有元素取完。

在我们的例子中 `0..10` 是一个返回开始和结束位置之间所有元素迭代器的表达式，这是一个前闭后开区间，所以我们的循环会打印 0 到 9

Rust 故意不提供 c 风格的 for 循环，手动控制循环中的每个元素很复杂而且容易出错，哪怕你是有经验的 c 程序员。

在后面我们讨论迭代器的时候，会进一步讨论 for 循环。

##while

另一种表示循环的方式是用 while:

	let mut x = 5u32;       // mut x: u32
	let mut done = false; // mut done: bool

	while !done {
    	x += x - 3;
    	println!("{}", x);
    	if x % 5 == 0 { done = true; }
	}
	
当你不确定要循环多少次的时候，就应该选择 while 了。
当你要一个死循环的时候，可以写成：

	while true {}
	
不过 Rust 贴心的为我们提供 loop 关键字来表示死循环：
	
	loop {}
	
Rust 的流程控制分析区别对待 loop 和 while true，loop 会明确告诉编译器我会一直循环下去，利于编译器更好的生成代码并提高安全性。所以当你需要使用死循环的时候应当选用 loop


##提前跳出迭代

让我们来看一下之前提到的 while 循环：

	let mut x = 5u32;
	let mut done = false;
	while !doen {
		x += x - 3;
		println!("{}",x);
		if x % 5 == 0 { done = true; }
	}
	
这里我们用一个 mut 布尔值 done 来记录我们什么时候应该跳出循环。Rust 有两个关键字辅助我们来修改迭代： break 和 continue

这儿我们可以用 break 来改进上面那个循环：

	let mut x = 5u32;

	loop {
    	x += x - 3;
    	println!("{}", x);
    	if x % 5 == 0 { break; }
	}
	
loop 创建了一个死循环，然后条件满足 break 出循环。

continue 是不执行当前迭代的剩余部分直接跳到下次迭代：

	for x in 0..10 {
		if x % 2 == 0 { continue; }
		println!("{}", x);
	}

以上只会打印出偶数。

continue 和 break 在两种循环方式里都可以使用。