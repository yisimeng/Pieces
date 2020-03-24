# Block

带有局部变量的匿名函数。

Block 语法： `^ 返回值类型 参数列表 表达式`。例如： `^ int (int count) {return count + 1}`

#### Block 类型变量

```
int (^block)(int) = ^(int count) {return count + 1;}; //初始化一个block
int (^block1)(int) = block;                           //block赋值给block1
int (^block2)(int);                                   //声明block2
block2 = block1; //赋值
```

## [block底层实现](http://www.dreamingwish.com/article/block%E4%BB%8B%E7%BB%8D%EF%BC%88%E4%B8%89%EF%BC%89%E6%8F%AD%E5%BC%80%E7%A5%9E%E7%A7%98%E9%9D%A2%E7%BA%B1%EF%BC%88%E4%B8%8A%EF%BC%89.html)

```
struct __block_impl {
  void *isa; // 说明block可以是一个对象
  int Flags;
  int Reserved;
  void *FuncPtr;
}; 
```

** block实例实际上由6部分构成： **
- isa指针，所有对象都有该指针，用于实现对象相关的功能。
- flags，用于按bit位表示一些block的附加信息，本文后面介绍block copy的实现代码可以看到对该变量的使用。
- reserved，保留变量。
- invoke，函数指针，指向具体的block实现的函数调用地址。
- desc， 表示该block的附加描述信息，主要是size大小，以及copy和dispose函数的指针。
- variables，capture过来的变量，block能够访问它外部的局部变量，就是因为将这些变量（或变量的地址）复制到了结构体中。


当我们创建一个block，并调用，编译器为我们做的事情如下：
1.创建block所有的部件代码：一个主体，一个真正的执行代码函数，一个描述信息（可能包含两个辅助函数,copy ,dispose）。
2.将我们的创建代码转码为block_impl的构造语句。
3.将我们的执行语句转码为对block的执行函数的调用。

### Block 分类

** 在Objective-C语言中，一共有3种类型的block： **
- _NSConcreteGlobalBlock 全局的静态block，不会访问任何外部变量。
- _NSConcreteStackBlock 保存在栈中的block，当函数返回时会被销毁。
- _NSConcreteMallocBlock 保存在堆中的block，当引用计数为0时会被销毁。

在 ARC 下，对 block 变量进行 copy 始终是安全的，无论它是在栈上，还是全局数据段，还是已经拷贝到堆上。
对栈上的 block 进行 copy 是将它拷贝到堆上；
对全局数据段中的 block 进行 copy 不会有任何作用；
对堆上的 block 进行 copy 只是增加它的引用记数。

如果栈上的 block 中引用了 __block 类型的变量，在将该 block 拷贝到堆上时也会将 __block 变量拷贝到堆上如果该 __block 变量在堆上还没有对应的拷贝的话，否则就增加堆上对应的拷贝的引用记数

#### 区分Block

* block 未捕获外部局部变量，那么就是**全局**(__NSGloabalBlock__);
* block 捕获外部局部变量，且没执行过 copy 操作，就是**栈**(__NSStackBlock__);
* __NSStackBlock__ 执行了 copy 操作，就是**堆**(__NSMallocBlock__)。

ARC 下 block 的 ‘=’ 赋值，表示 copy 操作。

