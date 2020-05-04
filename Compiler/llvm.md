# LLVM

Low Level Virtual Machine，可以用于常规编译器、JIT（实时编译）编译器、汇编器、调试器、静态分析工具等一系列跟编程语言相关的工具开发。

### Clang 

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

## 1 编译流程

在命令行输入以下命令，可以看到编译源文件经理的不同阶段

```shell
clang -ccc-print-phases main.m
```

```
0: input,'main.m',objective-c
1: preprocessor, {0}, objective-c-cpp-output
2: compiler, {1}, ir
3: backend, {2}, assembler
4: assembler, {3}, object
5: linker, {4}, image
6: bind-arch, "x86-64", {5}, image
```

**预处理**：可以通过:```clang -E main.m``` ，查看预编译时所做的内容：头文件导入，宏替换等。

**词法分析**：会将代码切分成token，比如大小括号、算法符号和字符串等。

```
clang -fmodules -fsyntax-only -Xclang -dump-tokens main.m
# 可以获得所有token类型和所在文件的具体位置
clang -fmodules -E -Xclang -dump-tokens main.n
```

**语法分析**：验证语法是否正确，并将所有节点组成抽象语法树。

```
# 打印语法树
clang -fmodules -fsyntax-only -Xclang -ast-dump main.m
```

CodeGen从上而下遍历抽象语法树，逐步翻译成LLVM IR，这里可以设置优化级别 -O1，-O3， -Os。

```
clang -O3 -S -fobjc-arc -emit-llvm main.o -o main.ll
```

> LLVM IR(Intermedia Representation): 中间表示。
>
> 编译器前端负责对源码进行词法分析、语法分析生成IR；
>
> 编译器后端负责将IR汇编成对应的机器指令。

生成汇编：

```
clang -S -fobjc-arc main.m -o main.o
```

生成目标文件：

```
clang -fmodules -c main.m -o main.o
```

生成可执行文件：

```
clang main.o main
```

执行：

```
./main
```

总结完整流程：

* 将编译信息写入辅助文件，创建文件架构 .app 文件
* 处理文件打包信息
* 执行CocoaPods编译前脚本，checkPods Manifest.lock
* 编译.m文件，使用CompileC和clang命令
* 链接器链接所需要的Framework
* 编译xib
* 编译资源文件
* 编译ImageAsset
* 处理Info.plist
* 执行CocoaPods脚本
* 拷贝标准库
* 创建 .app 文件和签名

### 2 IR

IR 是前端编译器的输出，后端编译器的输入。无论是OC还是swift代码，llvm唯一不变的是中间语言--LLVM IR。

LLVM 会跟runtime桥接，类型包括：

* 类、方法、成员变量等结构体的生成，并将其放到对应的Mach-O的session中。
* Non-Fragile ABI 可以标注多个 OBJC_IVAR\_$\_偏移值常量。
* 将ObjcMessageExpr 翻译成相应版本的 objc_msgSend，将super翻译成objc_msgSendSuper。
* strong、weak、copy、atomic 组合成 @property，自动实现setter 和getter。
* 对@synthesize的处理。
* 生成block_layout数据结构。
* \_\_block 和 \_\_weak 的处理。
* \_\_block_invoke。
* ARC处理包括插入 objc_storeStrong 和 objc_storeWeak 等ARC代码；ObjcAutoreleasePoolStmt 转 objc_autoreleasePoolPush/Pop;自动添加[super dealloc]; 给每个ivar的类合成 .cxx_destructor 方法，自动释放类的成员变量。