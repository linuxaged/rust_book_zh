#方法（Method）

函数有时候用起来会不太方便，比如当我们对一个数据施加多个函数的时候：

	baz(bar(foo(x)));

我们的习惯是从左往右阅读：baz,bar,foo ；这和函数被调用的顺序是相反的。如果阅读的顺序和函数被调用的顺序一致岂不是更好？比如：

	x.foo().bar().baz();

没错！Rust 是支持这种写法的，我们把这种调用方式叫做*方法调用*(method call)。可以通过 `impl` 关键字实现：

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

	fn main() {
	    let c = Circle { x: 0.0, y: 0.0, radius: 2.0 };
	    println!("{}", c.area());
	}

运行，打印得到 12.566371

##实例方法

我们定义了一个叫 Circle 的数据结构来表示圆，然后用 `impl` 为这个数据结构添加了一个叫 `area` 的方法用来计算圆的面积。方法如果有一个叫做 `self` 的特殊参数，这里的是 `&self`，另外还有 `self` 和 `&mut self` 可选，我们称之为*实例方法*(instance method)。如果数据结构是栈上的就用 `self`，如果我们想以引用的方式关联数据结构就用 `&self`，如果想以可变引用的方式关联数据结构就用  `&mut self`。`&self` 是最常用的。

##静态方法

另外还有一种不包含 self 参数的方法，我们叫做静态方法，也很常见：

	struct Circle {
	    x: f64,
	    y: f64,
	    radius: f64,
	}

	impl Circle {
	    fn new(x: f64, y: f64, radius: f64) -> Circle {
	        Circle {
	            x: x,
	            y: y,
	            radius: radius,
	        }
	    }
	}

	fn main() {
	    let c = Circle::new(0.0, 0.0, 2.0);
	}

这个静态方法创建了一个 Circle 实例。需要注意的是这里调用使用的 `Struct::method` 写法，要和 `ref.method()` 写法区分开来。

