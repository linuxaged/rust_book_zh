大多数程序员都不重视测试，我们先聆听一下 Dijkstra 的教诲：

    Program testing can be a very effective way to show the presence of bugs, but it is hopelessly inadequate for showing their absence.

    Edsger W. Dijkstra, "The Humble Programmer" (1972)

    测试代码能揭示我们程序里的 bug，但对保证我们的程序不出 bug 无能为力。

这里我门就来学习在 Rust 里如何正确地测试我们的代码。

#test 属性

给函数加上一个 test 属性，他就成了一个最简单的测试。

#test 模块

#test 目录

标注了 test 属性的函数就是一个最简单的测试。

#文档测试

#性能测试