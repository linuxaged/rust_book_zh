#数组，Vector 和 切片

这一章介绍线性数据结构。最基础的就是数组，包含一列同类型元素且大小固定。数组默认是不可变的。

	let a = [1, 2, 3]; // a: [i32; 3]
	let mut m = [1, 2, 3]; // mut m: [i32; 3]
	
有一种简写可以快速初始化数组里所有元素为特定值。举例：

	let a = [0; 20];
	
数组是一个 `[T; N]` 类型。T 表示元素类型，N 表示元素个数。

你可以用 `.len()` 方法来获取元素个数，用 `.iteor()` 方法来循环迭代数组里每个元素。举例，依次打印每个元素：

	let a = [1, 2, 3];

	println!("a has {} elements", a.len());
	for e in a.iter() {
	    println!("{}", e);
	}
	
你可以用下标获取特定位置的元素：

	let names = ["Graydon", "Brian", "Niko"]; // names: [&str; 3]
	println!("The second name is: {}", names[1]);
	
和大多数其他编程语言一样，下标重 0 开始。因此第一个元素是 `names[0]` 第二个是 `names[1]`。例子中将打印 `Brian`。如果你的下标越界了，会返回一个错误：“array access is bounds-checked at run-time”。数组越界在其他编程语言里是很多问题的根源。

vector 是动态的或者说大小可变的数组，实现为标准库类型 Vec<T>。vector 相对于数组好比 `String` 相对于 `&str`。你可以用 `vec!` 宏创建 vector。

	let v = vec![1, 2, 3]; // v: Vec<i32>
	
注意和我们之前用的 println! 宏不同，这里vec!用的是方括号 [].Rust 为了方便两种符号都支持 。

你可以像数组一样获取 vector 的长度，迭代里面的元素，用下标去索引里面的元素。另外可变的 vector 可以自动增长：

	let mut nums = vec![1, 2, 3]; // mut nums: Vec<i32>
	nums.push(4);
	println!("The length of nums is now {}", nums.len()); // Prints 4
	
vector 还内置了非常多有用的方法。这个你可以查询[标准库手册](http://doc.rust-lang.org/std/)。

切片（slice）是数组的一个引用（或者叫视图 view）。切片用于安全地、高效地获取数组的一部分而不需要引入拷贝。例如，你只需要获取读入内存的文件的某一行. 切片无法直接创建，都是在现有变量上切出来的。切片有一个长度，可设置为可变或者不可变，很多方面和数组类似：

	let a = [0, 1, 2, 3, 4];
	let middle = &a[1..4]; // 切出元素 1, 2, 3

	for e in middle.iter() {
    	println!("{}", e); // Prints 1, 2, 3
	}

`vector`, `String`, `&str` 都是可切的，因为他们都是基于数组的类型。切片是 &[T] 类型。

到这里 Rust 的基础部分就学完了。我们可以开始编写一个猜谜游戏了，不过还要知道怎么从键盘获取输入。因为如果不能通过键盘输入我们的猜测，这个游戏就玩不起来。