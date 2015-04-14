容器类型
===

Rust 的标准库高效地实现了大多数常用的数据结构。使用标准库里的实现可以避免一些数据格式转换有利于程序之间的通信。

我们平时用的最多的可能就是 Vec 和 HashMap。

Rust 的容器类型可以被归为四类：

	* 线性：Vec, VecDeque, LinkedList, BitVec
	* Map: HashMap, BTreeMap, VecMap
	* 集合：HashSet, BTreeSet, BitSet
	* 混合：BinaryHeap