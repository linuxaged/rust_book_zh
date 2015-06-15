Drop
===

我们已经讨论过 traits 了，接下来我们来看标准库提供的一种特殊的 trait：`Drop`。Drop trait 使得你能够在离开作用域的地方插入一些代码，比如：

    struct HasDrop;

    impl Drop for HasDrop {
        fn drop(&mut self) {
            println!("Dropping!");
        }
    }

    fn main() {
        let x = HasDrop;

        // do stuff

    } // x goes out of scope here

当 x 离开作用域后， `Drop` 就会被调用，Drop 有一个 `drop()` 方法，它接收一个可变引用 `self`

    struct Firework {
        strength: i32,
    }

    impl Drop for Firework {
        fn drop(&mut self) {
            println!("BOOM times {}!!!", self.strength);
        }
    }

    fn main() {
        let firecracker = Firework { strength: 1 };
        let tnt = Firework { strength: 100 };
    }

运行输出：

    BOOM times 100!!!
    BOOM times 1!!!

Drop 主要用来清理和 struct 绑定的资源。比如，Arc<T> 类型是一种引用计数类型，当 Drop 被调用的时候，引用计数会减一，如果引用计数变为零，值就会被销毁。