#错误处理

有时候，事情总会出错。对不可避免的事情有所规划是非常重要的。Rust提供了非常多的错误处理。

程序出错主要分两种：failtures, panics。我们先说它们的不同点，然后再来讨论它们分别应该如何处理。最后讨论何如把 failtures 升级为 panics。

##Failture vs. Panic
Rust 用两个专有名词来区分两种形式错误：failture 和 panic。failture 是指能够以某种方式恢复的错误。panic 是指无法恢复的错误。

那这里的“恢复”是什么意思呢？大多数情况下，我们都需要对出错的可能性有个预判。例如 `from_str` 函数：

	from_str("5");
	
这个函数将传入的 string 类型的参数转化成另一种类型。因为传入的是 string 类型，我们无法确定转化一定成功，譬如对

	from_str("helloworld");
	
我们就不知道该怎么转化。我们用这个函数的时候很清楚这个函数实际上只能将特定的一些参数转化，其他的参数则会出错。这种出错被称之为 failture。

除此之外还有一种完全不应该发生的或者我们没法恢复的错误。一个常见的例子 `assert!`:

	assert!(x == 5);
	
我们用 assert! 表示判断总是对的。如果错了，就是很严重的错误，错到我们无法再继续执行下去。另一个例子是 `unreachable!`:

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

