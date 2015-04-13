数组，Vector 和 切片
===

##数组

和其他编程语言一样，Rust 有一系列表示线性数据的数据结构。其中最基础的就是数组，数组由一排连续存储的同类型元素组成且大小固定。数组默认是不可变(immutable)的。

	let a = [1, 2, 3]; // 不可改写里面的成员
	let mut m = [1, 2, 3]; // 可改写里面的成员
	
相比于其他语言，在 Rust 里把数组初始化成特定的值要简单得多：

	let a = [0; 20]; // 初始值和大小用分号隔开
	
在 Rust 里数组是一个 `[T; N]` 类型。T 表示元素类型，N 表示元素个数。

你可以用 `.len()` 方法来获取元素个数，用 `.iteor()` 方法来循环迭代数组里每个元素。举例，依次打印每个元素：

	let a = [1, 2, 3];

	println!("a has {} elements", a.len());
	for e in a.iter() {
	    println!("{}", e);
	}
	
你可以用下标获取特定位置的元素：

	let names = ["Graydon", "Brian", "Niko"]; // names: [&str; 3]
	println!("The second name is: {}", names[1]);
	
和大多数其他编程语言一样，Rust 数组的下标也从 `0` 开始。因此第一个元素是 `names[0]` 第二个是 `names[1]`。例子中将打印 `Brian`。如果你的下标越界了，会返回一个错误：“array access is bounds-checked at run-time”。数组越界在其他编程语言里是很多问题的根源。

##Vector

vector 是动态的或者说大小可变的数组，是一个标准库类型： `Vec<T>`。vector 相对于数组好比 `String` 相对于 `&str`。你可以用 `vec!` 宏创建 vector。

	let v = vec![1, 2, 3]; // v: Vec<i32>
	
如果想把所有成员初始化一样的值：

	let v = vec![0; 20]; // 20 个成员全初始化为 0

你可以像数组一样获取 vector 的长度，迭代里面的元素，用下标去索引里面的元素。vector 还可以自动增长：

	let mut nums = vec![1, 2, 3]; // mut nums: Vec<i32>
	nums.push(4);
	println!("The length of nums is now {}", nums.len()); // Prints 4
	
vector 还内置了非常多有用的方法。这个你可以查询[标准库手册](http://doc.rust-lang.org/std/)。

##切片

切片（slice）是数组的一个引用（或者叫视图 view）。切片用于安全地、高效地获取数组的一部分而不需要引入拷贝。例如，你只需要获取读入内存的文件的某一行，这时候就应该使用切片。切片无法直接创建，都是在现有数据上切出来的。切片有一个长度，可设置为可变或者不可变，很多方面和数组类似：

	let a = [0, 1, 2, 3, 4];
	let middle = &a[1..4]; // 切出元素 1, 2, 3；切片是一个前开后闭区间

	for e in middle.iter() {
    	println!("{}", e); // Prints 1, 2, 3
	}

`Vector`, `String`, `&str` 都是可切的，因为他们都是基于数组的类型。切片是 `&[T]` 类型。

到这里 Rust 的基础部分就学完了。我们可以开始编写一个猜谜游戏了，不过还要知道怎么从键盘获取输入。因为如果不能通过键盘输入我们的猜测，这个游戏就玩不起来。