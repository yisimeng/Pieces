# block实现原理
当我们创建一个block，并调用之，编译器为我们做的事情如下：
1.创建block所有的部件代码：一个主体，一个真正的执行代码函数，一个描述信息（可能包含两个辅助函数,copy ,dispose）。
2.将我们的创建代码转码为block_impl的构造语句。
3.将我们的执行语句转码为对block的执行函数的调用。

** block实例实际上由6部分构成： **
- isa指针，所有对象都有该指针，用于实现对象相关的功能。
- flags，用于按bit位表示一些block的附加信息，本文后面介绍block copy的实现代码可以看到对该变量的使用。
- reserved，保留变量。
- invoke，函数指针，指向具体的block实现的函数调用地址。
- descriptor， 表示该block的附加描述信息，主要是size大小，以及copy和dispose函数的指针。
- variables，capture过来的变量，block能够访问它外部的局部变量，就是因为将这些变量（或变量的地址）复制到了结构体中。

** 在Objective-C语言中，一共有3种类型的block： **
- _NSConcreteGlobalBlock 全局的静态block，不会访问任何外部变量。
- _NSConcreteStackBlock 保存在栈中的block，当函数返回时会被销毁。
- _NSConcreteMallocBlock 保存在堆中的block，当引用计数为0时会被销毁。

在 ARC 下，对 block 变量进行 copy 始终是安全的，无论它是在栈上，还是全局数据段，还是已经拷贝到堆上。
对栈上的 block 进行 copy 是将它拷贝到堆上；
对全局数据段中的 block 进行 copy 不会有任何作用；
对堆上的 block 进行 copy 只是增加它的引用记数。

如果栈上的 block 中引用了\__block 类型的变量，在将该 block 拷贝到堆上时也会将 \__block 变量拷贝到堆上如果该 \__block 变量在堆上还没有对应的拷贝的话，否则就增加堆上对应的拷贝的引用记数
