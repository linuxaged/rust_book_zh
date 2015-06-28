我们选择实现一个猜谜游戏作为我们第一个工程，通过完成这个游戏来了解 Rust 如何解决一些基本的编程问题。首先解释一下这个游戏：程序生成一个0 到 100 的随机数，然后让我们输入一个猜测；程序会告诉你，你的猜测大了还是小了。猜中了他会发出祝贺。听起来不错吧？

##搭建项目

我们先搭建一个项目。切换到你的工作目录。还记得我们是怎么创建 hello world 工程文件结构以及 Cargo.toml 文件的吗？Cargo 为我们提供了一个命令：

	$ cd ~/projects
	$ cargo new guessing_game --bin
	$ cd guessing_game

我们把项目名称 `guessing_game` 传给 `cargo new` 然后再加上 `--bin` 参数，这个参数表示我们要创建的是一个可执行程序而不是库。

然后根目录下会有 Cargo.toml 文件：

    [package]

    name = "guessing_game"
    version = "0.1.0"
    authors = ["Your Name <you@example.com>"]

程序主文件 src/main.rs:

    fn main() {
        println!("Hello, world!");
    }

然后我们可以用 cargo build 命令来编译工程，或者用 cargo run 来编译并且执行编译好的程序。

#处理玩家的猜测

我们首先要让玩家能够输入猜测的数字，这就要用到标准输入输出库 std::io

    use std::io; // 引用标准库

    fn main() {
        println!("Guess the number!");

        println!("Please input your guess.");

        let mut guess = String::new(); // 创建一个 String 类型存放玩家输入的字符串

        io::stdin().read_line(&mut guess) // 从标准输入读取一行
            .ok()
            .expect("Failed to read line");

        println!("You guessed: {}", guess);
    }
