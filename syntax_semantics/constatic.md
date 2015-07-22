#`const` 和 static
Rust 里可以用 const 关键字定义常量：

	const N: i32 = 5;

和 let 绑定不同的是，这里必须显示指定 const 的类型，`: i32` 不能省略

常量在程序的整个生命周期中都存在。更重要的是编译器并不把常量放在固定的地方，而是以内联的方式把他们嵌入到每个使用到的地方。因此虽然你引用同一个常量，但他们的地址可能并不一致。


#static

Rust 通过 static 来定义全局变量，和 const 不一样的是 static 有个全局唯一的固定地址。

	static N: i32 = 5;

同样这里你也必须显示声明 static 的类型。

static 变量也存在于程序的整个声明周期，所以任何常量的引用都有一个 `'static` 声明周期

	static NAME: &'static str = "Steve";

#可变性

你可以通过 mut 把 static 变量设置为可修改：

	static mut N: i32 = 5;

既然是全局变量那么就会出现一个线程在读取这个变量的时候另一个线程却在修改它，这种不一致可能导致严重后果，所以必须在 unsafe 里面操作：

	unsafe {
	    N += 1;

	    println!("N: {}", N);
	}

另外，任何 static 类型都必须 `Sync`

#初始化

const 和 static 都必须预先指定一个值，而且必须用常量表达式。使用函数或者在运行时指定都不行。

#到底应该用哪个

如果两个都可以那么用 const.很少时候你需要给常量分配一个固定地址，and using a const allows for optimizations like constant propagation not only in your crate but downstream crates.

const 可以理解成 C 的 `#define`: 他会导致代码变大但不会有运行时损耗。“在 C 里我们应该用 #define 还是  static？” 和 “在 Rust 里我应该用 const 还是 static？”是一个问题