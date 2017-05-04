## 引用计数

## autorelease

那些情况Xcode会添加autorelease？
* convenient的构造方法会加autorelease是因为有个编译属性NS_RETURNS_NOT_RETAIN，alloc/new/copy/mutablecopy生成的是普通对象，因为编译属性是NS_RETURNS_RETAIN。

什么对象自动加入到 autoreleasepool中？
* 当使用alloc/new/copy/mutableCopy开始的方法进行初始化时，会生成并持有对象(也就是不需要pool管理)
* __weak修饰符只持有对象的弱引用，而在访问引用对象的过程中，该对象可能被废弃。那么如果把对象注册到autorealeasepool中，那么在@autorealeasepool块结束之前都能确保对象的存在。
* id的指针或对象的指针在没有显式指定时会被附加上__autorealeasing修饰符
