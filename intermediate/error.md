#错误处理

墨菲定律告诉我们只要有可能出错就一定会出错。对不可避免的事情有所规划是非常重要的。Rust 内置了各种错误处理机制。

程序出错主要分两种：failtures, panics。我们先说它们的不同点，然后再来讨论它们分别应该如何处理。最后讨论何如把 failtures 升级为 panics。

##Failture vs. Panic

Rust 用两个专有名词来区分两种形式错误：failture 和 panic。failture 是指能够以某种方式恢复的错误。panic 是指无法恢复的错误。

那这里的“恢复”是什么意思呢？大多数情况下，我们都需要对出错的可能性有个预判。例如 `from_str` 函数：

	from_str("5");
	
这个函数将传入的 string 类型的参数转化成另一种类型。因为传入的是 string 类型，我们无法确定转化一定成功，譬如对

	from_str("helloworld");
	
我们就不知道该怎么转化。我们用这个函数的时候很清楚这个函数实际上只能将特定的一些参数转化，其他的参数则会出错。这种出错被称之为 failture。

除此之外还有一种完全不应该发生的或者我们没法恢复的错误。一个常见的例子 `assert!` 宏:

	assert!(x == 5);
	
我们用 assert! 表示判断总是对的。如果错了，就是很严重的错误，错到我们无法再继续执行下去。另一个例子是 `unreachable!` 宏:

    enum Event {
        NewRelease,
    }

    fn probability(_: &Event) -> f64 {
        // real implementation would be more complex, of course
        0.95
    }

    fn descriptive_probability(event: Event) -> &'static str {
        match probability(&event) {
            1.00          => "certain",
            0.00          => "impossible",
            0.00 ... 0.25 => "very unlikely",
            0.25 ... 0.50 => "unlikely",
            0.50 ... 0.75 => "likely",
            0.75 ... 1.00  => "very likely",
        }
    }

    fn main() {
        std::io::println(descriptive_probability(NewRelease));
    }

运行会报错：

	error: non-exhaustive patterns: `_` not covered [E0004]

尽管我们想表达的是获得从 0.0 到 1.0 的所有的可能性描述，但是 Rust 并不知道这个规则。我们还需要对预料之外的肯能行进行处理：

    use Event::NewRelease;

    enum Event {
        NewRelease,
    }

    fn probability(_: &Event) -> f64 {
        // real implementation would be more complex, of course
        0.95
    }

    fn descriptive_probability(event: Event) -> &'static str {
        match probability(&event) {
            1.00 => "certain",
            0.00 => "impossible",
            0.00 ... 0.25 => "very unlikely",
            0.25 ... 0.50 => "unlikely",
            0.50 ... 0.75 => "likely",
            0.75 ... 1.00 => "very likely",
            _ => unreachable!()
        }
    }

    fn main() {
        println!("{}", descriptive_probability(NewRelease));
    }

这里 '_' 这种条件永远不会触发，这里用 unreachable! 来处理，一旦触发就会返回 panic。

# 使用 Option 和 Result 进行错误处理

最简单的错误处理方式就是使用 Option<T> 类型，以我们之前提到的 `from_str()` 方法为例：

    pub fn from_str<A: FromStr>(s: &str) -> Option<A>

`from_str()` 方法返回一个 `Option<T>` 类型，如果成功返回的是 `Some(value)`, 失败返回 `None`

Option<T> 只适用于最简单的场景，他并能给出出错信息，当我们需要出错信息的时候，我们就需要使用 Result<T, E> 类型了，它的定义如下：

    enum Result<T, E> {
       Ok(T),
       Err(E)
    }

这是个 Rust 自带类型。Ok(T) 表示成功， Err(E) 表示失败。大多数情况下我们都应该使用 Result<T, E>, 除了前面提到的那些简单情形。下面举个例子：

    #[derive(Debug)]
    enum Version { Version1, Version2 }

    #[derive(Debug)]
    enum ParseError { InvalidHeaderLength, InvalidVersion }

    fn parse_version(header: &[u8]) -> Result<Version, ParseError> {
        if header.len() < 1 {
            return Err(ParseError::InvalidHeaderLength);
        }
        match header[0] {
            1 => Ok(Version::Version1),
            2 => Ok(Version::Version2),
            _ => Err(ParseError::InvalidVersion)
        }
    }

    let version = parse_version(&[1, 2, 3, 4]);
    match version {
        Ok(v) => {
            println!("working with version: {:?}", v);
        }
        Err(e) => {
            println!("error parsing header: {:?}", e);
        }
    }

这里定义了一个 ParseError 枚举类型来表示各种可能的错误。

#panic!

`panic!` 用于处理那些无法恢复的错误，一旦这样的错误发生，当前线程就会崩溃。

    panic!("boom");

会报错：

    thread '<main>' panicked at 'boom', hello.rs:2

无法恢复的错误相对来说比较少见，请节制使用 `panic!`。

#把 failture 提升为 panic

在某些特殊情况下我们需要把 fail 提升为 panic。比如 `io::stdin().read_line(&mut buffer)` 在读取一行的时候如果出错会返回一个 `Result<usize>` 类型，我可以处理这个错误然后继续执行，但我们不需要处理而是想让程序直接终止，就要用 unwarp 方法：

    io::stdin().read_line(&mut buffer).unwrap();

unwrap 方法会调用 panic! 如果 Option 是 None 的话。也就是“把值返回给我，如果出错就崩溃”的意思。这种处理方式相比对错误类型进行匹配 要不可靠些。但有时候就需要让程序直接崩溃。

还有个比 unwrap 优雅一点的处理方式：

    let mut buffer = String::new();
    let input = io::stdin().read_line(&mut buffer)
                           .ok()
                           .expect("Failed to read line");

ok() 会把 Result 转化成 Option, expect() 作用和 unwrap 一样但是可以传入一个信息参数，你可以利用这个参数来提醒自己什么错误发生了。

#try!

调用很多个 返回 Result 类型的函数时候，处理返回错误就变得非常啰嗦。这时候 try! 就能简化代码。在没有 try! 的时候我们写成：

    use std::fs::File;
    use std::io;
    use std::io::prelude::*;

    struct Info {
        name: String,
        age: i32,
        rating: i32,
    }

    fn write_info(info: &Info) -> io::Result<()> {
        let mut file = File::open("my_best_friends.txt").unwrap();

        if let Err(e) = writeln!(&mut file, "name: {}", info.name) {
            return Err(e)
        }
        if let Err(e) = writeln!(&mut file, "age: {}", info.age) {
            return Err(e)
        }
        if let Err(e) = writeln!(&mut file, "rating: {}", info.rating) {
            return Err(e)
        }

        return Ok(());
    }

使用 try! 写成：

    use std::fs::File;
    use std::io;
    use std::io::prelude::*;

    struct Info {
        name: String,
        age: i32,
        rating: i32,
    }

    fn write_info(info: &Info) -> io::Result<()> {
        let mut file = try!(File::open("my_best_friends.txt"));

        try!(writeln!(&mut file, "name: {}", info.name));
        try!(writeln!(&mut file, "age: {}", info.age));
        try!(writeln!(&mut file, "rating: {}", info.rating));

        return Ok(());
    }

你只能在返回 Result 的函数里使用 try!, 也就意味你不能在 main 函数里用，因为 main 函数什么都不返回。

try! 用了 [FromError](http://doc.rust-lang.org/1.0.0-beta/std/error/#the-fromerror-trait) 来判断当前错误应该返回什么。