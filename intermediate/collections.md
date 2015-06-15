容器类型
===

Rust 的标准库高效地实现了大多数常用的数据结构。使用标准库里的实现可以避免一些数据格式转换有利于程序之间的通信。

我们平时用的最多的可能就是 Vec 和 HashMap。

Rust 的容器类型可以被归为四类：

	* 线性：Vec, VecDeque, LinkedList, BitVec
	* Map: HashMap, BTreeMap, VecMap
	* 集合：HashSet, BTreeSet, BitSet
	* 混合：BinaryHeap

##我到底选择哪个容器？

###使用 Vec 的条件
	* 你想要把对象收集起来然后传递到别的地方，且不关心对象具体是什么类型
	* 你想要按一定顺序保存一序列元素，然后会在最后添加元素
	* 你需要一个 stack 数据结构
	* 你要一个大小可变的数组
	* 你需要一个堆上的数组

###使用 VecDeque 的条件
	* 你需要一个高效的能在两端插入元素的 vec
	* 你需要一个队列
	* 你需要一个双端队列

###使用 LinkedList 的条件

###使用 HashMap 的条件
###使用 BTreeMap 的条件
###使用 VecMap 的条件
###使用 Set 的条件
###使用 BitVec 的条件
	* 你要存放一些布尔类型，但是个数不一定
	* 你要一个存放 bit 的 Vector
###使用 BitSet 的条件

	* 你想要一个 BitVec， 但是还要有 Set 属性
###使用 BinaryHeap 的条件

##性能

