没有复杂的运行时（runtime）使得 Rust 程序变得非常轻量，可以方便的嵌入到别的语言中。

实际项目中往往同时用到好几种语言，因为不同的程序语言有各自的优缺点，掌握多种语言利用他们的优点犀利地解决各类问题是非常重要的。

大多数语言的运行时提供了强大的功能，开发效率非常高，但是运行效率却不高；这时候我们就想能不能在保留开发效率的同时也能获得运行时效率呢？我们可以利用各语言提供的 C 接口解决这类问题，用 C 编写一些关键代码然后再通过接口调用。这种接口叫做 FFI："foreign function interface"

Rust 支持两种形式的 FFI：从其他语言调用 Rust；在 Rust 里调用其他语言。

我们另外有一个完整的章节讲述 FFI，这一章就 Ruby、Python、JavaScript 为例讲一下如何在这些语言里调用 Rust。

#Rust 库



#Python
在 Python 里调用 Rust 就更简单了：

	from ctypes import cdll

	lib = cdll.LoadLibrary("target/release/libembed.so")

	lib.process()

	print("done!")

从 ctypes 里导入 cdll 模块后通过 LoadLibrary 方法加载 .so 库，然后就可以调用到 process() 方法了，在我的机子上耗时 0.017 秒。很快！

#Node.js

为了在 Node.js 中调用 Rust 我们先要安装一个库：

	$npm install ffi

安装后就可以调用：

	var ffi = require('ffi');

	var lib = ffi.Library('target/release/libembed', {
	  'process': ['void', []]
	});

	lib.process();

	console.log("done!");

#总结

把 Rust 嵌入到别的语言里非常简单。我们还可以利用 FFI 做更多，具体参考 FFI 章节。