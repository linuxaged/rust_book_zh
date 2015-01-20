#Crate 和 模块

当项目变得越来越大，我们就需要将其模块化。 Rust 有一个模块系统。

## Crate 和模块

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