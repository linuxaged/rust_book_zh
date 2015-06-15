宏
===

到现在为止我们已经学习了非常多的抽象和复用代码的工具。

##定义一个宏

你已经用过 vec! 这个宏了，用它可以初始化若干个 vector 成员：

    let x: Vec<u32> = vec![1, 2, 3];

这不是一个普通的函数，因为它可以传入任意个参数。但我们可以把它看做是下面代码的简写：

    let x: Vec<u32> = {
        let mut temp_vec = Vec::new();
        temp_vec.push(1);
        temp_vec.push(2);
        temp_vec.push(3);
        temp_vec
    };

我们可以用宏来表达这种简写：

    macro_rules! vec {
        ( $( $x:expr ),* ) => {
            {
                let mut temp_vec = Vec::new();
                $(
                    temp_vec.push($x);
                )*
                temp_vec
            }
        };
    }

我们一句一句来看，首先

    macro_rules! vec { ... }

这是定义了一个叫做 vec 的宏
