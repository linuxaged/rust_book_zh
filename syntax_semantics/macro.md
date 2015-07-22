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

这是定义了一个叫做 `vec` 的宏，好似 `fn vec` 定义了一个叫做 `vec` 的方法。调用宏的时候我们要加上感叹号，一边和普通方法调用区分开：`vec!`

#Matching

宏是由一组规则定义的，这些规则使用模式匹配来描述：

    ( $( $x:expr ),* ) => { ... };

这像是模式匹配的一个条件分支。

`$x:expr` 会匹配任意的 Rust 表达式，把语法树绑定到元变量 `$x`，expr 是一个 fragment specifier.
把匹配条件写在 $(...),* 里面表示匹配零个或者多个用逗号分开的表达式。

出了特殊的匹配符号，表达式里面的每个符号都要匹配。比如：

    macro_rules! foo {
        (x => $e:expr) => (println!("mode X: {}", $e));
        (y => $e:expr) => (println!("mode Y: {}", $e));
    }

    fn main() {
        foo!(y => 3);
    }

打印出

    mode Y: 3

如果是

    foo!(z => 3)

就会报错：

    error: no rules expected the token `z`

#Expansion

#Repetition

#Hygiene

C 语言的宏有个著名的问题：

    #define FIVE_TIMES(x) 5 * x

    int main() {
        printf("%d\n", FIVE_TIMES(2 + 3));
        return 0;
    }

这里的宏被展开后得到的表达式是  `5 * 2 + 3` 使用 Rust 的宏的时候我们不需要担心这个问题：

    macro_rules! five_times {
        ($x:expr) => (5 * $x);
    }

    fn main() {
        assert_eq!(25, five_times!(2 + 3));
    }

