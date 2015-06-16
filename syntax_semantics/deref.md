`Deref` 约束
===

标准库提供了一个特殊的 trait 叫做 `Deref`。主要用来重载解引用运算符 `*`：

    use std::ops::Deref;

    struct DerefExample<T> {
        value: T,
    }

    impl<T> Deref for DerefExample<T> {
        type Target = T;

        fn deref(&self) -> &T {
            &self.value
        }
    }

    fn main() {
        let x = DerefExample { value: 'a' };
        assert_eq!('a', *x);
    }

