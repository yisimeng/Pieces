# LLVM

Low Level Virtual Machine，可以用于常规编译器、JIT（实时编译）编译器、汇编器、调试器、静态分析工具等一系列跟编程语言相关的工具开发。

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
```

**语法分析**：验证语法是否正确，并将所有节点组成抽象语法树。

```
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

