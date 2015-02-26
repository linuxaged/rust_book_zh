#Crate 和 模块

当项目变得越来越大，我们就需要将其模块化。 Rust 有一个模块系统。

## Crate 和 模块

##定义模块

我们用 mod 关键字定义我们的模块。我们的 src/lib.rs 写成：

	// in src/lib.rs

	mod english {
	    mod greetings {

	    }

	    mod farewells {

	    }
	}

	mod japanese {
	    mod greetings {

	    }

	    mod farewells {

	    }
	}

#多文件 Crate

如果把 crate 里的东西全放在一个文件，会显得拥挤。把他们分开放在不同的文件里更便于管理。我们有两种做法：

一、用

	mod english;

取代

	mod english {

	}

这样 Rust 就会去找一个叫 english.rs 或者 english/mod.rs 的文件。这样我们可以把上面的例子分解成 7 个文件，两个文件夹：

	$ tree .
	.
	├── Cargo.lock
	├── Cargo.toml
	├── src
	│   ├── english
	│   │   ├── farewells.rs
	│   │   ├── greetings.rs
	│   │   └── mod.rs
	│   ├── japanese
	│   │   ├── farewells.rs
	│   │   ├── greetings.rs
	│   │   └── mod.rs
	│   └── lib.rs
	└── target
	    ├── deps
	    ├── libphrases-a7448e02a0468eaa.rlib
	    └── native

`src/lib.rs` 使我们的根 crate :

	mod english;
	mod japanese;

这两句告诉 Rust 去找 叫做 `src/english.rs` `src/japanese.rs` 或者 `src/english/mod.rs` `src/japanese/mod.rs` 的文件。这里我们有子 crate，所以是第二种。`src/english/mod.rs` `src/japanese/mod.rs` 是一样的：

	mod greetings;
	mod farewells;