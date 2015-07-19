#可变性

可变性就是表示一个绑定能不能被修改的属性。Rust 语言和其他语言不通在于所有绑定默认是不可修改的：

    let x = 5;
    x = 6; // 报错

我们要用 mut 关键字明确告诉编译器这个绑定是可以修改的之后才能修改它。

如果绑定本身是一个指针你也可以通过给它加上 mut 来修改它指向的对象：

    let mut x = 5;
    let y = &mut x;
    *y = 5; // OK

这里 y 本身是不能修改的，你不能将 y 重新绑定到另一个指针；但是你能通过它来修改它所指向的值。

另外重要的是 mut 也是[模式]()的一部分，你能够：

    let (mut x, y) = (5, 6);

    fn foo(mut x: i32) {

    }

#内部可变性 vs. 外部可变性



#字段级(Field-level)可变性
只有借贷(`&mut`)和绑定(`let mut`)有可变性。你也就无法给 struct 成员指定可变性：

    struct Point {
        mut x: f32, // 错误!
        mut y: i32 // 错误!
    }

你只能设置 Point 类型的绑定的可变性：

    struct Point {
        x: f32,
        y: f32
    }
    // p 可修改
    let mut p = Point{x: 1.0, y: 1.0};
    p = Point{x: 2.0, y: 2.0};
    // 成员也可修改
    p.x = 3.0;

如果我们确实要字段级别的可变性怎么办呢？

使用 [Cell<T>](https://doc.rust-lang.org/nightly/std/cell/struct.Cell.html)!

    use std::cell::Cell;

    struct Point {
        x: i32,
        y: Cell<i32>,
    }

    let point = Point { x: 5, y: Cell::new(6) };

    point.y.set(7);

    println!("y: {:?}", point.y);

我们成功修改了 y 的值，打印出结果 `y: Cell { value: 7 }.`