Rust 支持有限的几个操作符重载。当你要重载某个操作符的时候只要实现特定的 trait 就可以了。

例如，可以通过实现 `Add` 这个 `trait` 来重载 `+` 操作符：

use std::ops::Add;

#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point { x: self.x + other.x, y: self.y + other.y }
    }
}

fn main() {
    let p1 = Point { x: 1, y: 0 };
    let p2 = Point { x: 2, y: 3 };

    let p3 = p1 + p2;

    println!("{:?}", p3);
}

这里我们为 Point 类型实现了 Add<Output=Point> 然后就能用 + 符号表示 Point 类型的加法了。

你可以在 std::ops 模块里找到所有可以重载的操作符。重载操作符的 trait 有着统一的范式，以 Add 为例：

    pub trait Add<RHS = Self> {
        type Output;

        fn add(self, rhs: RHS) -> Self::Output;
    }
这儿有三种类型。
在 `let z = x + y` 这样的表达式中 x 就是 Self 类型，y 就是 RHS，z 就是 Self::Output 类型。

如果给 Point 实现如下的 Add trait:

    impl Add<i32> for Point {
        type Output = f64;

        fn add(self, rhs: i32) -> f64 {
            // add an i32 to a Point and get an f64
        }
    }

就可以：

    let p: Point = // ...
    let x: f64 = p + 2i32;