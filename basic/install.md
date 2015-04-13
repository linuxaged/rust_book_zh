安装 Rust
===

使用 Rust 的第一步当然是安装了！你又多种选择来安装 Rust，其中最简单的是使用 `rustup` 脚本。如果你用的 Linux 或者 Mac 只需要在命令行下输入以下命令：

	$ curl -L https://static.rust-lang.org/rustup.sh | sudo sh
	
如果你担心使用 `curl | sudo sh` 的安全性，请阅读我们下面的声明。然后使用两步安装来检查我们的脚本：

	$ curl -L https://static.rust-lang.org/rustup.sh -O
	$ sudo sh rustup.sh
	
如果你用的是 Windwos 请下载 32 位或者 64 位 安装包。

如果你想卸载 Rust 执行：

	$ curl -s https://static.rust-lang.org/rustup.sh | sudo sh -s -- --uninstall
	
Windwos 再次运行安装包，里面提供了卸载选项。

当你想要更新 Rust 版本的时候也只要执行和安装时候相同的脚本就行了。

目前官方支持的平台包括：

Windows (7, 8, Server 2008 R2)

Linux (2.6.18 or later, various distributions), x86 and x86-64

OSX 10.7 (Lion) 或更高版本, x86 and x86-64

安装完之后你可以查看安装版本：

	$rustc --version
	
