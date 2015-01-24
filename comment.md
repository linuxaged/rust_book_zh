Rust 有两种注释：行注释和文档注释。

	// 所有在 '//' 符号后面的都是行注释
	let x = 5;
	// 如果你的注释很长，你需要分行然后在每行开始添加行注释符，
	// 在行注释符和注释内容直接加上一个空格会让注释看上去更清晰

另外一种注释就是文档注释。用 /// 表示，并且支持 Markdown 语法：

	/// `hello` is a function that prints a greeting that is personalized based on
	/// the name given.
	///
	/// # Arguments
	///
	/// * `name` - The name of the person you'd like to greet.
	///
	/// # Example
	///
	/// ```rust
	/// let name = "Steve";
	/// hello(name); // prints "Hello, Steve!"
	/// ```
	fn hello(name: &str) {
   		println!("Hello, {}!", name);
	}

你可以用 rustdoc 工具为你添加了文档注释的代码生成 html 格式的文档。

写文档注释的时候，添加参数、返回值段落并且提供例子代码对阅读和维护你代码的人来说是非常非常有用的。