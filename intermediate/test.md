% 测试

大多数程序员都不重视测试，我们先聆听一下 Dijkstra 的教诲：

    Program testing can be a very effective way to show the presence of bugs, but it is hopelessly inadequate for showing their absence.

    Edsger W. Dijkstra, "The Humble Programmer" (1972)

    通过测试能找出我们程序里的 bug，但无法保证我们的程序不出 bug。

这里我门就来学习在 Rust 里如何正确地测试我们的代码。

#test 属性

给函数加上一个 test 属性，他就成了一个最简单的测试。我还是先创建一个工程吧：

	$ cargo new adder
	$ cd adder

在创建新工程的时候 Cargo 会为自动为你在 src/lib.rs 里生成一个测试：

	#[test]
	fn it_works() {
	}

`#[test]` 就表示这是一个测试函数。我通过 cargo 来运行这个测试：

	$ cargo test
	   Compiling adder v0.0.1 (file:///home/you/projects/adder)
	     Running target/adder-91b3e234d4ed382a

	running 1 test
	test it_works ... ok

	test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

	   Doc-tests adder

	running 0 tests

	test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured

认真的你会发现 cargo 执行了两个测试，一个是我们写的测试，另一个是文档测试（暂且不表）。"test it_works ... ok" 表明我们的测试通过了。
那为什么我们什么都没做的测试都通过了呢，在 rust 里只要不 panic 的测试都能算作通过。我们来试一下通不过的测试：

	#[test]
	fn it_works() {
	    assert!(false);
	}
	
assert! 是 Rust 内置的一个宏，他有一个布尔类型的参数，如果这个参数是 true，什么都不做，如果为 false 就会 panic! 程序。我们执行一下：

	$ cargo test
	   Compiling adder v0.0.1 (file:///home/you/projects/adder)
	     Running target/adder-91b3e234d4ed382a

	running 1 test
	test it_works ... FAILED

	failures:

	---- it_works stdout ----
	        thread 'it_works' panicked at 'assertion failed: false', /home/steve/tmp/adder/src/lib.rs:3



	failures:
	    it_works

	test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured

	thread '<main>' panicked at 'Some tests failed', /home/steve/src/rust/src/libtest/lib.rs:247

除了这些错误信息之外我们还得到一个操作系统的状态量：

	$ echo $?
	101

当你把 cargo test 集成到别的工具里的时候或许有用。

我们还能给测试添加 `should_panic` 属性：

	#[test]
	#[should_panic]
	fn it_works() {
	    assert!(false);
	}

这个属性的意思是说 这个方法是可以 panic 的，我们试一下：

	$ cargo test
	   Compiling adder v0.0.1 (file:///home/you/projects/adder)
	     Running target/adder-91b3e234d4ed382a

	running 1 test
	test it_works ... ok

	test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

	   Doc-tests adder

	running 0 tests

	test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured

rust 还有一个 assert_eq! 的宏，用于比较两个参数是否相等：

	#[test]
	#[should_panic]
	fn it_works() {
	    assert_eq!("Hello", "world");
	}

同样因为 should_panic 的存在，测试也通过了：

	$ cargo test
	   Compiling adder v0.0.1 (file:///home/you/projects/adder)
	     Running target/adder-91b3e234d4ed382a

	running 1 test
	test it_works ... ok

	test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

	   Doc-tests adder

	running 0 tests

	test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured

#test 模块

#test 目录

标注了 test 属性的函数就是一个最简单的测试。

#文档测试

#性能测试