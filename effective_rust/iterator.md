迭代器
===

我们接着聊循环。

还记得 for 循环吗，

	for x in range(0i, 10i) {
		println!("{}", x);
	}
	
在你对 Rust 有了进一步了解的基础上我们可以讨论上面的代码是如何实现的。range 函数返回一个迭代器。迭代器就是一种不停的调用 .next() 方法，得到一串连续对象的函数。比如：

	let mut range = range(0i, 10i);
	
	loop {
		match range.next() {
			Some(x) => {
				println!("{}",x);
			},
			None => { break }
		}
	}
	
我们用 range 的返回值构造了一个可变绑定，这就是我们的迭代器。然后创建一个 loop， 内置 match.match 是用作 range.next() 结果匹配的。next 返回的是 Option<int> 类型，

迭代器并不仅仅用于 for 循环。你可以实现 Iterator trait 来自定义自己的迭代器。这里暂且不表。

range 还是语言的一个基础元素。当你需要对一个 vector 进行迭代时，你可能会这么写：

	let nums = vec![1i, 2i, 3i];
	
	for i in range(0u, nums.len()) {
		println!("{}", nums[i]);
	}
	
这种写法比不上使用迭代器的写法，.iter() 方法会依次返回 vector 中每个元素的引用：

	let nums = vec![1i, 2i, 3i];

	for num in nums.iter() {
    	println!("{}", num);
	}
	
选择这种写法有两个原因。首先更直接地表达我们的意图。我们直接迭代 vector 内容而不是先迭代 vector 的索引再迭代 vector 内容。再者，这种写法效率更高：第一种用了 nums[i],这会引入越界检查 bounds checking.第二种用的是迭代器返回的引用，就不需要 bounds checking。这是迭代器常见的使用场景，不需要越界检查，我们的代码依然安全。

这儿有个关于 println! 的细节，num 其实是 &int 类型，println! 为我们做了解引用。也可以这么写：

	let nums = vec![1i, 2i, 3i];

	for num in nums.iter() {
    	println!("{}", *num);
	}
	
这就显式的解引用了 num。为什么 iter() 返回一个引用呢？避免拷贝！