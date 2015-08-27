Cargo
===

Cargo 负责三件事：下载你代码中的依赖；编译这些依赖；最后编译你的代码。

如果你是通过官方的安装器来安装的 Rust ，那么同时你会安装好 Cargo。如果你用别的方法安装的请参考 Cargo 的 [README](https://github.com/rust-lang/cargo/blob/master/README.md) 来安装。

我们来把我们的 Hello World 程序转化成 Cargo 工程。

首先我们需要新建一个叫做 Cargo.toml 的文件，然后把源文件放到正确的目录下：

	$mkdir src
	$mv main.rs src/main.rs

Cargo 默认的源文件路径为 src 目录，他和 README　和 license 都处于工程文件夹的最上层。Cargo 把我们的工程组织的非常整洁。

接下来编辑我们的配置文件：

	$editor Cargo.toml // 注意 C 必须是大写

写入:

	[package]
	name = "hello_world"
	version = "0.0.1"
	authors = [ "Your name <you@example.com>" ]
	
	[[bin]]
	name = "hello_world"

这是 TOML 格式的文件，稍微解释一下：

	TOML aims to be a minimal configuration file format that's easy to read due to obvious semantics. TOML is designed to map unambiguously to a hash table. TOML should be easy to parse into data structures in a wide variety of languages.

TOML 类似于 INI，但相比有些优点。

这个文件里定义了两个 table，一个叫做 package 一个叫做 bin，package 告诉 Cargo 你的工程的一些基本信息。bin 告诉 Cargo 我这是一个可执行程序工程不是一个库工程。

接下来你就可以编译了：

	$ cargo build
	   Compiling hello_world v0.0.1 (file:///home/yourname/projects/hello_world)
	$ ./target/debug/hello_world
	Hello, world!

不管多大的项目你都可以用 cargo build 命令来完成编译，当你觉得程序足够稳定可以发布时，用 cargo build --release 可以用
	
你可能注意到 Cargo 还为我们创建了一个叫做 Cargo.lock 的文件：

	[root]
	name = "hello_world"
	version = "0.0.1"

Cargo 使用这个文件来追踪项目里的依赖项。我们的例子里没有依赖项，所以这个 Cargo.lock 没什么用。你不要动这个文件，这是由 Cargo 自动处理的。

到这里我们成功地用 Cargo 构建了 hello_world，虽然程序比较简单，但我们学会了很有用的工具，以后的 Rust 项目里都会用到。
