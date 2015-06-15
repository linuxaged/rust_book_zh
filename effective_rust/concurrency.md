并发
===

并发和并行是计算机科学里非常重要的课题，现今在实际生产中也是一个热门话题。如今的计算机有着越来越多的核心，我们程序员却没能充分运用。

Rust 在并发的时候也能够保证内存安全。

在开始讨论并发之前首先要明确 Rust 提供了最基本的元素，然后并发是在这些基本元素上实现的。也就是说，如果对 Rust 提供的并发不满意，你可以自己去实现。[mio](https://github.com/carllerche/mio) 就是一个例子。

##Send , Sync

Rust 提供了两个 trait 来帮助代码产生并发

##Send

如果一个类型实现了 Send ，就是告诉编译器我这个类型里的一些东西的所有权能够安全的在线程之间传递。

比如我们用一个管道连通两个线程，通过管道把一些数据传到另一个线程，我们就要保证这些数据实现了 Send。

相反，如果我们封装了一个并不是线程安全的 FFI 库，我们就不实现 Send 从而让编译帮助我们保证数据只能在当前线程里传递。



##Sync

第二个 trait 叫做 Sync。当一个类型实现了 Sync 就等于告诉编译器这个类型跨线程使用是线程安全的。

以上两种 trait 借助于 Rust 的类型系统为你的并发代码提供了强有力的保障。为什么呢？我们先来看一下 Rust 里并发代码什么样。

##Threads

Rust 的标准库里有个线程库，我们用这个来编写并行的代码，举例：

	use std::thread::Thread;

	fn main() {
		Thread::scoped(|| {
			println!("Hello from a thread!");
			});
	}

这里我们传了一个闭包给 Thread::scoped() ，然后闭包就会被调度到新线程里执行。之所以叫做 scoped ，是因为他会返回一个 join 监视：

	use std::thread::Thread;

	fn main() {
	    let guard = Thread::scoped(|| {
	        println!("Hello from a thread!");
	    });

	    // guard goes out of scope here
	}

在 guard 生命周期结束的时候，如果线程还没执行完，程序会被阻塞，直到执行完为止。如果这不是你想要的，你可以用 `Thread::spawn()`:

	use std::thread::Thread;
	use std::old_io::timer;
	use std::time::Duration;

	fn main() {
	    Thread::spawn(|| {
	        println!("Hello from a thread!");
	    });

	    timer::sleep(Duration::milliseconds(50));
	}

或者使用 `.detach()`:

	use std::thread::Thread;
	use std::old_io::timer;
	use std::time::Duration;

	fn main() {
	    let guard = Thread::scoped(|| {
	        println!("Hello from a thread!");
	    });

	    guard.detach();

	    timer::sleep(Duration::milliseconds(50));
	}

我们这用 sleep 是因为在 main 退出的时候会杀死所有正在执行的线程。

scoped 有一个有意思的类型符号：

	fn scoped<'a, T, F>(self, f: F) -> JoinGuard<'a, T>
    where T: Send + 'a,
          F: FnOnce() -> T,
          F: Send + 'a


##Channels

我们可以用管道替代 sleep 来同步：

	use std::sync::{Arc, Mutex};
	use std::thread::Thread;
	use std::sync::mpsc;

	fn main() {
	    let data = Arc::new(Mutex::new(0u32));

	    let (tx, rx) = mpsc::channel();

	    for _ in (0..10) {
	        let (data, tx) = (data.clone(), tx.clone());

	        Thread::spawn(move || {
	            let mut data = data.lock().unwrap();
	            *data += 1;

	            tx.send(());
	        });
	    }

	    for _ in 0 .. 10 {
	        rx.recv();
	    }
	}

我们用 mpsc::channel() 方法创建了一个管道，在一端发送 `()` 然后另一端等待接收。

#Panics

当前线程执行过程中崩溃会返回一个 `panic!`，你可以利用这个把线程和其他代码隔离开来：

	use std::thread;

	let result = thread::spawn(move || {
	    panic!("oops!");
	}).join();

	assert!(result.is_err());

