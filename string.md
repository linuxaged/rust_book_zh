#Strings

对任何一个程序员来说字符串都是掌握一门语言的必经之路。鉴于 Rust 语言的侧重，它的字符串处理系统有别于其他语言。任何时候只要有大小可变的数据结构，事情就会变得棘手，字符串恰好是大小可变的数据结构。

接下来我们看一些细节。 string 其实是被编码成 UTF-8 字节的 Unicode 标量序列。所有的string 一定是能被编码成 UTF-8 序列。另外，string 并不是以 null 结尾 并且可以包含 null 字节。

Rust 有两种字符串类型： `&str` 和 `String`。

第一种 `&str`，叫做字符串切片（string slices）。字符串常量都是 `&str` 类型：

	let mystring = "Hello there."; // mystring: &str

这个 `Hello there.` 是静态分配的，直接编译到程序里，存在于整个程序运行周期内。 mystring 是静态分配的字符串的一个引用。字符串切片大小固定，无法修改。


另外一种是 `String`，是堆上的字符串。这个字符串大小可变，并且一定是 UTF-8 格式。

	let mut s = "Hello".to_string(); // mut s: String
	println!("{}", s);

你可以用 `as_slice()` 方法来获得 `String` 类型的一个切片：

	fn take_slice(slice: &str) {
		println!("Got: {}", slice);
	}

	fn main() {
		let s = "hello".to_string();
		take_slice(s.as_slice());
	}

比较 String 和字符串常量优先选择 `as_slice()`：

	fn compare(string: String) {
    	if string.as_slice() == "Hello" {
        	println!("yes");
    	}
	} 

而不是 `to_string()`：

	fn compare(string: String) {
    	if string == "Hello".to_string() {
        	println!("yes");
    	}
	}

把 `String` 切成 `&str` 是高效的，反之把 `&str` 类型转换成 `String` 会引发内存分配。除非你明确自己非得转换成 `String`。

以上就是 Rust 字符串的基础部分！如果你之前一直使用的脚本语言，Rust 的字符串可能要比他们的要复杂一点，但是涉及到底层细节的时候，这些复杂性很有必要。现在你只要记住 `String` 会在堆上分配内存并管理之，而 `&str` 是对另一个字符串的引用就可以了。
