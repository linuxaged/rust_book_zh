回顾一下我们是怎么实现方法的：

	struct Circle {
	    x: f64,
	    y: f64,
	    radius: f64,
	}

	impl Circle {
	    fn area(&self) -> f64 {
	        std::f64::consts::PI * (self.radius * self.radius)
	    }
	}
	
trait 和它的区别在于先给出方法定义然后再实现：

	struct Circle {
    	x: f64,
    	y: f64,
    	radius: f64,
	}

	struct Circle {
	    x: f64,
	    y: f64,
	    radius: f64,
	}

	trait HasArea {
	    fn area(&self) -> f64;
	}

	impl HasArea for Circle {
	    fn area(&self) -> f64 {
	        std::f64::consts::PI * (self.radius * self.radius)
	    }
	}
	
`trait` 代码块 和 `impl` 代码块很像，区别是我们只放了一个声明在里面。当我们要实现 trait 时，用 `impl Trait for Item` 句式 而不是  `impl Item`

这有什么用处呢？如果你还记得之前我们遇到的这个错误：

	error: binary operation `==` cannot be applied to type `T`
	
我们可以用 traits 来约束我们的模板。看下面这个例子：

	fn print_area<T>(shape: T) {
    	println!("This shape has an area of {}", shape.area());
	}
	
编译会报错：

	error: type `T` does not implement any method in scope named `area`
	
因为 T 可以使任意类型，我们不能确定 T 实现了 area 方法。这时候我们就能在 T 模板后面加一个 trait 约束来确保 T 类型实现了 area 方法：

	fn print_area<T: HasArea>(shape: T) {
    	println!("This shape has an area of {}", shape.area());
	}
	
<T: HasArea> 的意思就是“任意实现了 HasArea 这个 trait 的类型”。trait 里包含必须要实现的方法（这里是 area ）声明，我们可以肯定实现了 HasArea 的类型一定有个 `.area()` 方法.

下面是完整地例子：

	trait HasArea {
	    fn area(&self) -> f64;
	}

	struct Circle {
	    x: f64,
	    y: f64,
	    radius: f64,
	}

	impl HasArea for Circle {
	    fn area(&self) -> f64 {
	        std::f64::consts::PI * (self.radius * self.radius)
	    }
	}

	struct Square {
	    x: f64,
	    y: f64,
	    side: f64,
	}

	impl HasArea for Square {
	    fn area(&self) -> f64 {
	        self.side * self.side
	    }
	}

	fn print_area<T: HasArea>(shape: T) {
	    println!("This shape has an area of {}", shape.area());
	}

	fn main() {
	    let c = Circle {
	        x: 0.0f64,
	        y: 0.0f64,
	        radius: 1.0f64,
	    };

	    let s = Square {
	        x: 0.0f64,
	        y: 0.0f64,
	        side: 1.0f64,
	    };

	    print_area(c);
	    print_area(s);
	}
	
运行输出：

	This shape has an area of 3.141593
	This shape has an area of 1
	
这儿 print_area 就成了通用函数了，同时还确保参数必须是实现了 HasArea 的类型，如果我们传入 int 型作为参数：

	print_area(5);
	
会报错：
	
	error: failed to find an implementation of trait main::HasArea for int
	
目前为止我们只给 struct 实现过 trait，实际上你可以为任意类型实现 trait。包括 i32 类型；

	trait HasArea {
	    fn area(&self) -> f64;
	}

	impl HasArea for i32 {
	    fn area(&self) -> f64 {
	        println!("this is silly");

	        *self as f64
	    }
	}

	5.area();
	
虽然可以这么做，但不鼓励这么做。

trait 还有两个约束，当我们把 trait 的实现放在一个 mod 时：

	mod shapes {
	    use std::f64::consts;

	    trait HasArea {
	        fn area(&self) -> f64;
	    }

	    struct Circle {
	        x: f64,
	        y: f64,
	        radius: f64,
	    }

	    impl HasArea for Circle {
	        fn area(&self) -> f64 {
	            consts::PI * (self.radius * self.radius)
	        }
	    }
	}

	fn main() {
	    let c = shapes::Circle {
	        x: 0.0f64,
	        y: 0.0f64,
	        radius: 1.0f64,
	    };

	    println!("{}", c.area());
	}
	
会报错：

	error: type `shapes::Circle` does not implement any method in scope named `area`
	
这时我们加上 use 引用就可以了：

	use shapes::HasArea;

	mod shapes {
	    use std::f64::consts;

	    pub trait HasArea {
	        fn area(&self) -> f64;
	    }

	    pub struct Circle {
	        pub x: f64,
	        pub y: f64,
	        pub radius: f64,
	    }

	    impl HasArea for Circle {
	        fn area(&self) -> f64 {
	            consts::PI * (self.radius * self.radius)
	        }
	    }
	}


	fn main() {
	    let c = shapes::Circle {
	        x: 0.0f64,
	        y: 0.0f64,
	        radius: 1.0f64,
	    };

	    println!("{}", c.area());
	}
	
这就意味着即使被人为 int 类型实现了 trait 只有你不 use 那个 trait ，那么他的实现就对你没有影响。

另外还有一个更严格的约束，trait 和需要实现这个 trait 的类型必须在同一个 crate 下面。我们能为 i32 实现 HasArea 是因为 HasArea 在我们的 crate 内。但是如果我们要实现 Float 这个 trait 就不行了，这个 trait 由 Rust 提供并不在 我们的 crate 内。

最后需要注意的是包含 trait 约束的 模板函数编译后会生成多个版本，看一下  print_area：

	fn print_area<T: HasArea>(shape: T) {
    	println!("This shape has an area of {}", shape.area());
	}

	fn main() {
    	let c = Circle { ... };

    	let s = Square { ... };

    	print_area(c);
    	print_area(s);
	}
	
分别以 c 和 s 为参数调用  print_area ，Rust 会根据参数类型生成两个不一样的函数，实际上代码看上去应该是这样：

	fn __print_area_circle(shape: Circle) {
	    println!("This shape has an area of {}", shape.area());
	}

	fn __print_area_square(shape: Square) {
	    println!("This shape has an area of {}", shape.area());
	}

	fn main() {
	    let c = Circle { ... };

	    let s = Square { ... };

	    __print_area_circle(c);
	    __print_area_square(s);
	}
	
这么做节省动态决定调用哪个的开销，但是也增大了程序的体积，应为我们多出一个同样功能的函数。

#回到 inverse 例子

在模板一章里我们讲到：

	fn inverse<T>(x: T) -> Result<T, String> {
    	if x == 0.0 { return Err("x cannot be zero!".to_string()); }
    	Ok(1.0 / x)
	}
	
编译报错：

	error: binary operation `==` cannot be applied to type `T`
	
这是因为不确定传入的 T 类型到底能不能比较，这里我们试着加上PartialEq trait 约束：

	fn inverse<T: PartialEq>(x: T) -> Result<T, String> {
    	if x == 0.0 { return Err("x cannot be zero!".to_string()); }
    	Ok(1.0 / x)
	}
	
但是报错了：

		error: mismatched types:
 	expected `T`,
    	found `_`
	(expected type parameter,
    	found floating-point variable)
    	
我们希望 T 能和 另一个实现了 PartialEq 的 T 类型比较，但这里却是 浮点型。这里就要把 trait 换成 Float 了：

	use std::num::Float;

	fn inverse<T: Float>(x: T) -> Result<T, String> {
    	if x == Float::zero() { return Err("x cannot be zero!".to_string()) }
    	let one: T = Float::one();
    	Ok(one / x)
	}

我们要把 0,0 和 1.0 替换成Float trait 里对应的方法。f32 和 f64 类型都实现了 Float 所以就对了。

	println!("the inverse of {} is {:?}", 2.0f32, inverse(2.0f32));
	println!("the inverse of {} is {:?}", 2.0f64, inverse(2.0f64));

	println!("the inverse of {} is {:?}", 0.0f32, inverse(0.0f32));
	println!("the inverse of {} is {:?}", 0.0f64, inverse(0.0f64));
	