#类型转换

出于安全的考虑， Rust 提供了两种类型转换。一种是使用 as 关键字; 一种使用 transmute, 他可对任意类型进行转换，非常非常危险！

#as

可以用 as 做一些基本的转换：

	let x: i32 = 5;

	let y = x as i64;

他只会对特定类型进行转换，当我们：

	let a = [0u8, 0u8, 0u8, 0u8];

	let b = a as u32; // four eights makes 32

想把四字节转化成一个无符号整型的时候会报错：

	error: non-scalar cast: `[u8; 4]` as `u32`
	let b = a as u32; // four eights makes 32
	        ^~~~~~~~

想要完成这种转换要用到 transmute

#transmute

transmute 函数是由编译器 intrinsic 提供，他提供的操作既简单又可怕。他可以让 Rust 设定某个值为一个类型，尽管实际上他是另一个类型。使用它意味着忽略类型检测系统，一切由程序员来控制。

我们可以用 transmute 来完成之前的操作，把四个 u8 类型换换成 u32 类型：

	use std::mem;

	unsafe {
	    let a = [0u8, 0u8, 0u8, 0u8];

	    let b = mem::transmute::<[u8; 4], u32>(a);
	}

我们必须把这个操作放在 unsafe 里面。

transmute 几乎不做任何检查，只看两个类型的大小是否一致，如果一致就进行转换。当你使用 transmute 转换不同大小的类型时：

	use std::mem;

	unsafe {
	    let a = [0u8, 0u8, 0u8, 0u8];

	    let b = mem::transmute::<[u8; 4], u64>(a);
	}

编译报错：

	error: transmute called on types with different sizes: [u8; 4] (32 bits) to u64
	(64 bits)