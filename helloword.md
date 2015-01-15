Rust 安装好了，我们开始写第一个 Rust 程序。我们遵循古老的传统，打印 Hello World 到屏幕。

	$ mkdir ~/projects
	$ cd ~/projects
	$ mkdir hello_world
	$ cd hello_world
	
	fn main() {
	    println!("Hello, world!");
	}
	
保存文件然后执行：

	$rustc main.rs
	$./main
	Hello, World!