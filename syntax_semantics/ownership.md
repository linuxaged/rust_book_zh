所有权系统
===

这篇将讲述 Rust 的所有权（ownership）系统。这是 Rust 最为特殊和引人入胜的部分，也是 Rust 程序员必须要掌握的。Rust 是通过*所有权*来实现内存安全的。所有权系统分为几个部分：

	* *所有权*(ownership)
	* *借贷*(borrowing)
	* *生命周期*(lifetimes)

我们会依次介绍。

##前言

在开始讨论之前我们先指出所有权系统两点需要注意的地方。

Rust 的最大卖点就是兼顾安全和速度，并且这两项是通过零代价的抽象来实现的。所有权系统是零代价抽象的代表。我们即将开始的所有讨论都是在编译时完成的，并没有任何运行时开销。

说实话这个系统的实现确实有一点代价：牺牲了简易性，导致学习曲线陡峭。很多初学者都有一种和编译器作斗争的感觉，自己觉得明明能编译通过的程序却报错。这是由我们理解的所有权系统工作方式和实际的 Rust 实现有所不同引起了。当然这类编译问题会随着你对 Rust 所有权系统的了解逐步减少。

##所有权

所有权的核心是资源。资源有好多种（内存，硬盘上的文件等），方便起见这里我们只讨论最常用的一种资源：内存。

使用任何不提供垃圾回收的语言，只要你从堆上申请了内存就要负担起以销毁它的责任。比如一个 `foo` 函数分配了 4 字节的内存，然后一直没有销毁。这就会引起内存泄露，每次我们调用 `foo` 都会分配 4 字节。一定次数的调用之后，程序就会吃光我们的内存。这很糟糕，所以我们要以某种方式回收这 4 字节内存，另外我们也不能多回收。多次回收也会导致问题，这里先不说为什么。总之任何时候我们申请了内存，我们要确保回收那片内存并且只回收一次。多一次不行，少一次也不行。分配的次数和回收的次数要对应。

关于内存分配有个重要的细节。任何时候我们申请一块内存返回的都是指向这块内存的指针。我们通过这个指针对内存进行操作。只要指针在，我们就可以操作内存，一旦指针被销毁我们也失去了对内存操作的能力。

历来，系统编程语言都需要你手动追踪内存分配和释放。我们看一个 C 语言的例子：

	{
		int *x = malloc(sizeof(int)); // 申请内存
		*x = 5;
		free(x);	// 释放内存
	}

Rust 将资源的申请和释放设计成了一个整体，就是所有权系统。

我们先来看一种简单的情况。

我们用 Box::new() 申请堆上内存，得到一个指向该内存的引用。和 C 不同，Rust 会分析这个引用的作用域，离开当前作用域后，你就不能使用它进行任何操作，然后Rust负责帮你释放内存。上面的例子用 Rust 表示：

	{
		let x = Box::new(5);
	}

这里用 Box::new 在堆上分配了能放下一个 i32 的内存。我们前面说了，申请的内存必须释放，但是我们并没有看到有关释放内存的操作，实际上 Rust 自动帮你释放了。它知道 x 是 box 的一个引用，并且知道在代码块结束时 x 会离开当前作用域，于是 Rust 编译器会在代码块最后插入释放内存的代码。这些操作都是编译器替我们完成的，不用担心会忘记，也不用担心分配和释放的次数不匹配。

这比较简单，但是如果我们要把我们的 box 传给另一个函数呢？你们也许会这样写：

	fn main() {
    	let x = Box::new(5);
    	add_one(x);
	}

	fn add_one(mut num: Box<i32>) {
    	*num += 1;
	}

这段代码能运行，但存在问题。假设我们要加一行代码，打印出 x 的值：

	fn main() {
    	let x = Box::new(5);
    	add_one(x);
    	println!("{}", x);
	}

	fn add_one(mut num: Box<i32>) {
    	*num += 1;
	}

尝试编译的话，会得到一个错误：

	error: use of moved value: `x`
   		println!("{}", x);
                       ^

每个内存申请都要对应一个内存释放。当我们把 box 传交给 add_one 时，将会有两个指针指向同一块内存：main 函数中的 x 和 add_one 函数中的 num。如果我们还是向前面说的，在指针离开作用域就释放内存，那么这块内存将会被释放两次，这会导致程序出错。因此 Rust 规定一个资源只能有一个拥有者，所以当我们调用 add_one 的时候 Rust 会解除 x 对 box 的所有权，然后将 num 设为拥有者，x 不再拥有 box 资源，于是有了那个错误：use of moved value: `x`。

一种解决这个错误的方式是让 num 在用完资源后再把资源转交，怎么转交呢？返回一个 box 就行了：

	fn main() {
    	let x = Box::new(5);
    	let y = add_one(x);
    	println!("{}", y);
	}

	fn add_one(mut num: Box<i32>) -> Box<i32> {
    	*num += 1;
    	num
	}

这样就可以正常运行了。我们返回了一个完整的 box 而不是指向他的指针，资源转交给了 y 。仔细想一下， add_one 只是临时地用一下 box 而已，没必要完全将 box 转交给他。Rust 为此引入了一个*借贷*的概念，专指将资源临时借给别人但我还是拥有者的情况。Rust 里面的引用 '&' 可以用来表示借贷。

##借贷

再来看一下当前版本的 add_one 函数：

	fn add_one(mut num: Box<i32>) -> Box<i32> {
    	*num += 1;
    	num
	}

这个函数传入的是 box （堆上分配的资源），一个资源不能同时有两个拥有者，所以 box 被转交给了 num。最后函数返回一个 box 又将资源转交出去了。（转交给你就是你的了，英国将香港转交给中国，香港就是中国的了，当然香港本来就是中国的）

生活中，我们常把东西借给别人用一段时间。这东西还是我的，我只是借给人家用而已。东西在谁那，谁就有责任保管好，所以东西借给他，保管责任也转移到他那边。

Rust 的所有权系统于此类似，允许拥有者将资源临时的借给别人。下面是一个借贷版本的 add_one，你可以和转交版本的对比一下：

	fn add_one(num: &mut i32) {
    	*num += 1;
	}

这个版本从调用者那边借贷来一个 i32 ，然后递增一次。函数结束，num 被释放，借贷关系终结。

相应我们也要改一下 main 函数：

	fn main() {
    	let mut x = 5;
    	add_one(&mut x);
    	println!("{}", x);
	}

	fn add_one(num: &mut i32) {
    	*num += 1;
	}

这里我们只是临时借贷 box ，并不需要像之前通过返回 box 转交来转交去了那么麻烦了。

##生命周期

如果你已经理解了上面的内容，我们再来看一些更复杂的情况：

	1.A 申请了一块资源
	2.A 把这资源以引用方式借贷给 B
	3.A 用完了资源决定销毁，而此时 B 的借贷关系并没有解除
	4.B 又要用资源了

因为在 B 用资源之前，资源已经被释放掉了，这就会出现危险指针错误或者叫 use after free错误。

为了解决这个问题，我们要确保 4 事件永不会在 3 事件之后进行。为此 Rust 引入了*生命周期*的概念，生命周期是指借贷关系的生命周期。在 B 使用资源的时候 Rust 会检查 B 的借贷关系的生命周期。

我们再来看一下 add_one 函数：

	fn add_one(num: &i32) -> i32 {
    	*num + 1
	}

这里其实省略了 num 的生命周期，因为能够被推断出来：

	fn add_one<'a>(num: &'a i32) -> i32 {
    	*num + 1
	}

'a 就叫做生命周期，生命经常用 'a 'b 'c 表示，简单清晰。我们分析一下句法：

	fn add_one<'a>(...)

这句的意思是 add_one 有一个叫做 'a 的生命周期。如果我们需要两个可以写成：

	fn add_two<'a, 'b>(...)

在参数列表里我们使用了刚声明的生命周期：

	...(num: &'a i32) -> ...

对比一下 `&i32` 和 `&'a i32` 可知只是在 & 和 i32 之间加了个生命周期 'a。`&i32` 读作“一个 i32 类型的引用“，`&'a i32` 读作“一个具有 'a 生命周期的 i32类型的引用”。

##作用域

最直观的理解生命周期的方法是把作用域内引用的生命周期画出来，例如：

    fn main() {
        let y = &5;     // -+ y 的生命周期从这开始
                        //  |
        //do sth        //  |
                        //  |
    }                   // -+ y 的生命周期在这结束

添加一个 Foo 方法：

    struct Foo<'a> {
        x: &'a i32,
    }

    fn main() {
        let y = &5;           // -+ y 的生命周期从这开始
        let f = Foo { x: y }; // -+ f 的生命周期从这开始
        //do sth              //  |
                              //  |
    }                         // -+ f 和 y 的生命周期在这结束

f 的生命周期包含在 y 的生命周期内，这儿没有问题；可是一旦把一个短生命周期的对象绑定到一个常生命周期的对象上问题就来了：

	struct Foo<'a> {
	    x: &'a i32,
	}

	fn main() {
	    let x;                    // -+ x goes into scope
	                              //  |
	    {                         //  |
	        let y = &5;           // ---+ y goes into scope
	        let f = Foo { x: y }; // ---+ f goes into scope
	        x = &f.x;             //  | | error here
	    }                         // ---+ f and y go out of scope
	                              //  |
	    println!("{}", x);        //  |
	}                             // -+ x goes out of scope

这儿 x 的生命周期最长，f 的生命周期最短。但我们把 f 的一个成员绑定到了 x，这会导致 f 已经销毁了我们还在使用绑定到 x 的这个 f 的字段。

我们可以给这些声明周期命名。
##'static

有一种特殊的生命周期叫做 'static。它表示对象的生命周期是整个程序的运行时。我们第一次遇到 'static 是在使用字符串常量的时候：

	let x: &'stattic str = "hello, world";

另外一个例子是全局变量：

	static FOO: i32 = 5;
	let x: &'static i32 = &FOO;

这会添加一个 i32 类型到程序的数据段上，x 是它的一个引用。

#共享声明周期

到目前为止我们都是举得一个资源只有一个所有者的例子。但有时这并不合理。不如车有四个轮子，我们要把四个轮子都赋给一辆车：

	struct Car {
	    name: String,
	}

	struct Wheel {
	    size: i32,
	    owner: Car,
	}

	fn main() {
	    let car = Car { name: "DeLorean".to_string() };

	    for _ in 0..4 {
	        Wheel { size: 360, owner: car };
	    }
	}

会报错：

	error: use of moved value: `car`
	    Wheel { size: 360, owner: car };
	                              ^~~
	note: `car` moved here because it has type `Car`, which is non-copyable
	    Wheel { size: 360, owner: car };
	                              ^~~

这里我们需要 Car 能够被多个 Wheel 引用，Box<T> 不行因为它只能有一个持有者，这里我们要用 Rc<T>:

	use std::rc::Rc;

	struct Car {
	    name: String,
	}

	struct Wheel {
	    size: i32,
	    owner: Rc<Car>,
	}

	fn main() {
	    let car = Car { name: "DeLorean".to_string() };

	    let car_owner = Rc::new(car);

	    for _ in 0..4 {
	        Wheel { size: 360, owner: car_owner.clone() };
	    }
	}

这我们把 Car 放在了 Rc<T> 里面，得到一个 Rc<Car>,然后使用 clone() 方法来创建一个新的引用。

这是最简单的多人持有的情况。当我们需要考虑到多线程时，要用 Arc<T> ，这个相对于 Rc<T> 它的引用计数是线程安全的。

#生命周期的省略

我们引入两个术语来讨论生命周期的省略，一个叫输入生命周期(input lifetime)，一个叫输出生命周期(output lifetime)。输入生命周期和函数的参数绑定，输出生命周期和函数返回值绑定。例如：

	fn foo<'a>(bar: &'a str) // 有个输入生命周期

	fn foo<'a>() -> &'a str // 有个输出生命周期

	fn foo<'a>(bar: &'a str) -> &'a str // 输入输出生命周期都有

这儿有三条规则：

	* 编译器会把函数各个参数省略掉的生命周期变成不同的生命周期参数。
	* 如果仅有一个输入生命周期，不管它省略与否，所有输出生命周期和这个输入生命周期保持一致。
	* 如果有多个输入生命周期，其中有一个是 `&self` 或者 `&mut self`，那么所有省略掉的输出生命周期都被设成和 self 一致。

#例子

	fn print(s: &str); // 省略形式
	fn print<'a>(s: &'a str); // 完整形式

	fn debug(lvl: u32, s: &str); // 省略形式
	fn debug<'a>(lvl: u32, s: &'a str); // 完整形式

	// In the preceding example, `lvl` doesn't need a lifetime because it's not a
	// reference (`&`). Only things relating to references (such as a `struct`
	// which contains a reference) need lifetimes.

	fn substr(s: &str, until: u32) -> &str; // 省略形式
	fn substr<'a>(s: &'a str, until: u32) -> &'a str; // 完整形式

	fn get_str() -> &str; // ILLEGAL, no inputs

	fn frob(s: &str, t: &str) -> &str; // ILLEGAL, two inputs
	fn frob<'a, 'b>(s: &'a str, t: &'b str) -> &str; // 完整形式: Output lifetime is unclear

	fn get_mut(&mut self) -> &mut T; // 省略形式
	fn get_mut<'a>(&'a mut self) -> &'a mut T; // 完整形式

	fn args<T:ToCStr>(&mut self, args: &[T]) -> &mut Command // 省略形式
	fn args<'a, 'b, T:ToCStr>(&'a mut self, args: &'b [T]) -> &'a mut Command // 完整形式

	fn new(buf: &mut [u8]) -> BufWriter; // 省略形式
	fn new<'a>(buf: &'a mut [u8]) -> BufWriter<'a> // 完整形式