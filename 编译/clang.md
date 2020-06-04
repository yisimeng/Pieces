# Clang

LLVM 的子项目，编译器前端，C、C++、OC的编译器，用于快速编译，主要进行语法分析、词法分析并生成中间代码。

##### 常用命令

* -x: 指定后续输入文件的编译语言，例 Objective-C
* -arch: 指定编译架构， ARM7
* -f: 以-f开头的命令参数，用以诊断、分析代码
* -W: 以-W开头的命令参数，可以通过逗号分隔不同的参数以定制编译警告
* -D：以-D开头的参数，指预编译宏，通过这些宏可以实现条件编译
* -I：添加目录到搜索路径中 
* -F：需要的Framework
* -c：运行预处理、编译、汇编
* -o：将编译结果输出到指定文件

### 1. Clang attribute

clang 提供的能够让开发者在编译过程中参与源代码控制的方法，格式为 attribute((xxx))。

##### 1.1 format 格式化字符串

NSLog

##### 1.2 提示方法或属性已被弃用

```objective-c
- (void)oldMethod  __attribute__((deprecated("oldMethod 方法已被弃用，请使用newMethod")));
- (void)oldMethod  DEPRECATED_ATTRIBUTE; // 系统定义的弃用方法宏
```

##### 1.3 unavailable 方法不可用提示

使用了不可用的方法，会导致编译失败。

##### 1.4 unused 取消静态函数未使用警告

默认情况下，未使用的静态函数编译时会显示警告`Unused function 'testFunc'`。

##### 1.5 warn_unused_result  必须使用函数返回值

修饰函数时，调用者必须使用返回值，否则会出现警告。

swift3.0之后所有函数默认支持这个clang attribute，如果返回值是可以丢弃的，则需要@discardableResult 修饰。

##### 1.6 cleanup 作用域结束时自动执行一个指定的方法

作用域结束包括：right brace(右大括号)，return，goto，break，exception等情况。

可以在OC下实现defer。详细使用可见ReactiveCocoa中的 onExit 宏定义。

##### 1.7 overloadable 方法重载

在C函数上实现函数的重载。

```c
__attribute__((overloadable)) void printArgument(int num) {
    NSLog(@"a int number: %d", num);
}
__attribute__((overloadable)) void printArgument(NSString * str) {
    NSLog(@"a String value: %@", str);
}
```

##### 1.8 objc_designated_initializer 指定内部初始化方法

如果当前类有指定初始化方法，其他的初始化方法必须调用指定初始化方法。

如果当前为指定初始化方法，那么必须调用super的指定初始化方法。

否则会有警告。

##### 1.9 objc_subclassing_restricted 限定不能有子类

相当于Java的final。有子类继承时会报错。

```
Cannot subclass a class that was declared with the 'objc_subclassing_restricted' attribute
```

##### 1.10 objc_requires_super 子类继承这个方法后必须调用super

需要在super的方法里执行一些必要的操作，可以使用这个修饰，提示使用者。

##### 1.11 const 重复调用相同数值参数时的优化处理

用于参数为数值型函数，多次调用相同数值型参数，返回结果是相同的，所以只在第一次进行运算，以后都是返回首次的运算结果。编译器的一种优化处理方式。

##### 1.12 constructor 和 destructor

调用顺序： `+load -> constructor 方法 -> main 函数 -> descructor 方法`。

可以将swizzling相关代码写到 constructor 中。

### 2 代码警告

忽视代码警告，中间字符串可以替换。

```objective-c
// 废弃代码警告
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wdeprecated-declarations"
/**老版本方法调用*/
#pragma clang diagnostic pop
```

```objective-c
// 忽略可能存在的内存泄漏
#define SuppressPerformSelectorLeakWarning(Stuff) \
do { \
_Pragma("clang diagnostic push") \
_Pragma("clang diagnostic ignored \"-Warc-performSelector-leaks\"") \
Stuff; \
_Pragma("clang diagnostic pop") \
} while (0)
```





