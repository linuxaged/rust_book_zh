Trait Objects
===

当涉及到多态的时候，有一种判断到底应该调用哪个版本的机制，这种机制叫做调度（dispatch）。有两种调度，静态调度和动态调度。Rust 更倾向于使用静态调度，但是也支持动态调度；动态调度是通过 trait objects 来实现的。

##准备

为了后面讨论方便，我们先写一个 trait 的例子：

	trait Foo {
	    fn method(&self) -> String;
	}

然后替 `u8` 和 `String` 类型实现这个 trait ：

	impl Foo for u8 {
	    fn method(&self) -> String { format!("u8: {}", *self) }
	}

	impl Foo for String {
	    fn method(&self) -> String { format!("string: {}", *self) }
	}

##静态调度

我们可以通过 trait 绑定来实现静态调度：

	fn do_something<T: Foo>(x: T) {
	    x.method();
	}

	fn main() {
	    let x = 5u8;
	    let y = "Hello".to_string();

	    do_something(x);
	    do_something(y);
	}

这里 Rust 用了一种叫做 monomorphization 的方法来实现静态调度，就是生成了两个版本的 do_something ，然后在调用的地方替换成对应版本。也就是 Rust 生成了如下的代码：

	fn do_something_u8(x: u8) {
	    x.method();
	}

	fn do_something_string(x: String) {
	    x.method();
	}

	fn main() {
	    let x = 5u8;
	    let y = "Hello".to_string();

	    do_something_u8(x);
	    do_something_string(y);
	}

这种实现的优势在于方法调用可以内联，效率更高；弊端在于，如果有非常多版本的方法，二进制代码的体积会迅速膨胀，因为编译器替每个版本的 trait 实现都生成了一个函数。

另外值得注意的是编译器并不是万能的，它的优化可能导致代码运行变慢。比如过多的使用内联函数会阻塞指令缓存（instruction cache），因此你应该小心使用`#[incline]` 和 `#[inline(always)]`。同样的原因，静态调度是有可能比动态调度慢的。

##动态调度

Rust 通过 trait objects 来实现动态调度。像 `&Foo` ，`Box<Foo>` 这样的 trait object 包含了一个实现了 Foo trait 的对象。这个对象的类型具体什么类型只有在运行时才知道。

trait object 可以通过转化指针来获得，但这个指针指向的对象必须实现了相关的 trait。比如 `&x as &Foo`； 或者是强制转换，比如一个以 &Foo 为参数的函数，你调用时候传入 `&x` （注意！这里是引用，和前面的静态调度不一样）, 编译器会把 &x 转换成 trait object。这些转换同样适用于可变类型的指针，例如把 `&mut T` 转换到 `&mut Foo` 也适用于 Box 类型，比如把 `Box<T>` 转换到 `Box<Foo>`。

这些转换的实际作用是消除编译器对指针类型的记录，trait object 因此也叫做“类型消除”。

回到上面的例子，我们尝试用把指针转换到 trait object 的方法来实现动态调度：

	fn do_something(x: &Foo) {
	    x.method();
	}

	fn main() {
	    let x = 5u8;
	    do_something(&x as &Foo);
	    // 或者
	    // do_something(&x);

	    let y = "test".ToString();
	    do_something(&y as &Foo);
	    // 或者
	    // do_something(&y);
	}

以 trait object 为参数的函数只有一份，并不会像静态调度那样生成多个版本的函数。和静态调度相比避免了代码的膨胀，但是引入了虚函数调用的开销而且编译器也很难对这类函数进行内联等相关的优化。

##为什么用指针？

和其他语言不一样，Rust 并没有默认使用指针来表示一些类型。在 rust 里不同类型有着不同的长度，这个长度信息在参数传递和把他从栈上移到堆上是非常重要的。

但在这里，我们需要一个对象既能装下 `String` 又能装下 `u8`，不用指针是很难办到的，除非这个对象非常非常大，但是这么做很浪费。

把对象放在指针后面意味着我们不需要关心对象的大小了，只要关心指针本身的大小。

##[trait object 的表示](http://programmers.stackexchange.com/a/247313/79822)

trait 的所有方法可以通过 trait object 里面一个 vtable 类型的指针来调用。（vtable 由编译器创建和管理）

我们从简单的 trait object 运行时表示开始，Rust 源代码的 `std::raw` 模块里包含了 trait object 的结构体：

	pub struct TraitObject {
	    pub data: *mut (),
	    pub vtable: *mut (),
	}

也就是说一个 trait object 包含了 data 指针和 vtable 指针。

data 指针指向 trait object 里保存的数据， vtable 指针指向了实现了 Foo Trait 的类型的 vtable。

vtable 是一个包含了若干个函数指针的结构体，这些函数指针指向该类型实现的 trait 方法。一个像 trait_object.method() 这样的方法调用会从 vtable 里获取一个对应的指针然后动态调用它：

	struct FooVtable {
	    destructor: fn(*mut ()),
	    size: usize,
	    align: usize,
	    method: fn(*const ()) -> String,
	}

	// u8:

	fn call_method_on_u8(x: *const ()) -> String {
	    // the compiler guarantees that this function is only called
	    // with `x` pointing to a u8
	    let byte: &u8 = unsafe { &*(x as *const u8) };                                

	    byte.method()
	}

	static Foo_for_u8_vtable: FooVtable = FooVtable {
	    destructor: /* compiler magic */,
	    size: 1,
	    align: 1,

	    // cast to a function pointer
	    method: call_method_on_u8 as fn(*const ()) -> String,
	};


	// String:

	fn call_method_on_String(x: *const ()) -> String {
	    // the compiler guarantees that this function is only called
	    // with `x` pointing to a String
	    let string: &String = unsafe { &*(x as *const String) };

	    string.method()
	}

	static Foo_for_String_vtable: FooVtable = FooVtable {
	    destructor: /* compiler magic */,
	    // values for a 64-bit computer, halve them for 32-bit ones
	    size: 24,
	    align: 8,

	    method: call_method_on_String as fn(*const ()) -> String,
	};

其实这里的核心思想就是把函数调用替换成了函数指针，然后放到 vtable 里面，从而使得包含了 vtable 的 trait object 可以作为参数来控制最终调用函数的位置。
vtable 里面的 destructor 字段指向一个函数，这个函数会清理掉 vtable 占用的资源，对 u8 来说这个字段为空，但 String 就需要它来释放内存。对占有类型的 trait object，比如 Box<Foo> 来说这个 destructor 把自身由 Box 分配的空间 和 trait object 里面占用的资源一同清理掉。size 和  align 字段保存了被擦除类型的大小和对齐要求；这两个字段目前还没有用，将来会用到。

下面例举了编译器如何显示地构造和使用 trait object ：

	let a: String = "foo".to_string();
	let x: u8 = 1;

	// let b: &Foo = &a;
	let b = TraitObject {
	    // store the data
	    data: &a,
	    // store the methods
	    vtable: &Foo_for_String_vtable
	};

	// let y: &Foo = x;
	let y = TraitObject {
	    // store the data
	    data: &x,
	    // store the methods
	    vtable: &Foo_for_u8_vtable
	};

	// b.method();
	(b.vtable.method)(b.data);

	// y.method();
	(y.vtable.method)(y.data);

如果 b 或者 y 是持有类型的 trait object 的话，在离开作用域的时候还会有个 (b.vtable.destructor)(b.data) 调用。