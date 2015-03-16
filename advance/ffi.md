#外部函数接口(Foreign Function Interface)

#介绍

这篇指南会以 [snappy](https://github.com/google/snappy) 压缩/解压 库为例介绍如何编写外部函数的绑定。Rust 目前还不能直接调用 C++ 库，但 snappy 有一个 C 接口（参考 [snappy-c.h](https://github.com/google/snappy/blob/master/snappy-c.h)）。

下面是一个调用外部函数的例子，如果安装了 snappy 就能编译通过：

	extern crate libc;
	use libc::size_t;

	#[link(name = "snappy")]
	extern {
	    fn snappy_max_compressed_length(source_length: size_t) -> size_t;
	}

	fn main() {
	    let x = unsafe { snappy_max_compressed_length(100) };
	    println!("max compressed length of a 100 byte buffer: {}", x);
	}
	
extern 块是外部库的函数记号列表。`#[link...]` 属性用来指示链接器链接 snappy 库来对上函数记号。

外部函数默认是危险的，所以调用他们需要置于 unsafe{} 内。C 库经常暴露出非线程安全的接口，

#创建安全的接口



#外部调用约定

#和外部代码交互

#空指针优化
	
#从 C 调用 Rust

有时候你可能需要编写能够从 C 调用的 Rust 函数，这很容易实现，只要添加一些东西：

	#[no_mangle]
	pub extern fn hello_rust() -> *const u8 {
    	"Hello, world!\0".as_ptr()
	}
	
extern 告诉编译器这个函数需要遵守C调用规范。no_mangle 属性关闭 Rust 的 name mangling，让函数更容易链接。