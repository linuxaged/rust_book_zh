% 文档

文档在软件工程中是非常重要的，有时候比代码本身还要重要。我认为文档工具是现代编程语言的标配，既然 Rust 声称自己是现代（modern）的编程语言当然也提供了文档工具。

#rustdoc

Rust 的安装包里面包含了一个叫做 `rustdoc` 的工具，可以用它来生成文档。`cargo doc` 命令其实也是调用的 `rustdoc`。

文档来源有两种，一种是你的源代码注释，一种是单独的 markdown 文件。

#给你的代码写文档

按照一定的格式给 rust 代码写注释就可以方便地用 `rustdoc` 来自动生成文档:

    /// 创建然后返回一个 `Rc<T>` 类型
    ///
    /// # Example
    ///
    /// ```
    /// use std::rc::Rc;
    ///
    /// let five = Rc::new(5);
    /// ```
    pub fn new(value: T) -> Rc<T> {
        // ...
    }

注意，用于生成文档的注释应该是三个斜杠 `///` 开头，注释里面的内容是 markdown 格式的。

一开始写文档的时候经常会犯一些错误，比如：

    /// The `Option` type. See [the module level documentation](../) for more.
    enum Option<T> {
        None, /// No value
        Some(T), /// Some value `T`
    }

这里把文档注释写在了一行代码的最后，调用 rustdoc 生成文档的时候会报错：

    hello.rs:4:1: 4:2 error: expected ident, found `}`
    hello.rs:4 }
               ^
这是因为文档注释作用于注释后面 `///` 的语句，而 `/// Some value `T`` 之后没有语句了，所以报错。

##特殊段落

rustdoc 定义了一些特殊段落，

/// # Examples

/// # Panics

/// # Failtures

/// # Safety

如果你的代码是 unsafe 的，你可以在这个段落里面描述一下。

