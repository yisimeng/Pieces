# ARC 自动引用计数（Automatic Reference Counting）

ARC是编译器在编译期所做的优化，不是运行时特性或者垃圾回收机制。是在代码编译时，自动在合适位置添加release或者autorelease，在没有强引用时，**适时**的释放对象（autorelease 并不一定会立即释放，在当前的runloop结束时，会统一销毁）。

### 引用计数式内存管理

* 自己生成的对象，自己持有。
* 非自己生成的对象，自己也能持有。
* 自己持有的对象在不需要时，需要释放。
* 非自己持有的对象，无法释放。

由 alloc/new/copy/mutableCopy 方法生成的对象， 自己生成，自己持有。

```
id obj = [[NSObject alloc] init];
```

其他方法生成的对象，非自己生成，retain 之后持有。

```
id obj = [NSMutableArray array];
```

自己持有的对象（自己生成或者retain后持有的），在不需要时需要使用release释放。

* release： 只是将对象的引用计数-1，当计数为0时会调用对象的 dealloc 方法释放。
* autorelease： 添加到autoreleasepool中，待runloop结束，统一释放。

引用计数的实现，GNUstep将引用计数保存在对象内存的头部，苹果则是保存在引用计数表（散列表）中。

GNUstep的优点：

1. 代码量少。
2. 统一管理引用计数和对象用的内存块。

苹果使用引用计数表的优点：

1. 对象分配内存时无需考虑引用计数。
2. 引用计数表中记录各个内存块地址，可追溯到各个对象。

利用工具监测内存泄漏时，引用计数表中的记录有助于监测各个对象的持有者是否存在。

### autorelease 

NSAutoreleasePool 调用 autorelease 方法会发生异常。

在 OC 中无论哪个对象调用 autorelease 方法，实际调用的都是 NSObject 的 autorelease 方法。但是 NSAutoreleasePool 类重载了该方法，所以会报错。

### 所有权修饰符

__strong：表示强引用，在超出作用于时释放。对象类型默认的所有权修饰符。
__weak：表示弱引用，避免循环引用，不持有对象，对象释放后，自动置为nil（iOS5以后）。

### ARC 下的使用规则

* 不能使用 retain/release/retainCount/autorelease。编译器自动插入，ARC下禁用方法。
* 不能使用 NSAllocateObject/NSDeallocateObject。ARC下禁用。
* 不能显式调用dealloc。没有强引用时会自动调用，ARC下直接调用报错。
* 使用@autoreleasepool代替NSAutoreleasePool。
* 不能使用NSZone。参数已被忽略。
* 对象类型变量不能作为C结构体（struct/union）成员。
* 显示转换 id 和 ‘void *’