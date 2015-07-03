`Deref` 约束
===

标准库提供了一个特殊的 trait 叫做 `Deref`。主要用来重载解引用运算符 `*`：

    use std::ops::Deref;

    struct DerefExample<T> {
        value: T,
    }

    impl<T> Deref for DerefExample<T> {
        type Target = T;

        fn deref(&self) -> &T {
            &self.value
        }
    }

    fn main() {
        let x = DerefExample { value: 'a' };
        assert_eq!('a', *x);
    }

这在写自定义指针类型的时候很有用。Rust 有个语言特性叫做“解引用约束”。如果你的类型 `U` 实现了 `Deref<Target=T>`,那么使用 &U 的时候会被约束成 &T，例如：

    fn foo(s: &str) {
        // borrow a string for a second
    }

    // String 实现了 Deref<Target=str>
    let owned = "Hello".to_string();

    // therefore, this works:
    foo(&owned);

这是唯一能让 Rust 替你自动进行类型转换的方式。

还有一个例子就是：

    fn foo(s: &[i32]) {
        // borrow a slice for a second
    }

    // Vec<T> implements Deref<Target=[T]>
    let owned = vec![1, 2, 3];

    foo(&owned);

Vector 类型被约束成 slice

#解引用和方法调用

在调用一个方法的时候也会用到 Deref，例如：

    struct Foo;

    impl Foo {
        fn foo(&self) { println!("Foo"); }
    }

    let f = Foo;

    f.foo();

foo 需要一个 &self 参数，但 f 不是一个引用。那这里还能通过 f 来调用 foo 呢？在 Rust 里下面的调用都是等价的：

    f.foo();
    (&f).foo();
    (&&f).foo();
    (&&&&&&&&f).foo();

不管你引用了多少次，编译器会自动给加上对应次数的解引用。这里的解引用就用到了 Deref