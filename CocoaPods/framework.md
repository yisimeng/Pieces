# 静态库、动态库

### 1. 介绍

**静态库：**链接是直接拷贝到可执行文件中，app间不共用。文件为`.a`和`.framework`。

**动态库：**链接时不复制，程序运行时由系统动态加载到app内存空间，系统只加载一次，APP间共用，例如UIKit，CoreFoundation。文件为`.framework`和`.tdb`。

**Embedded Binaries:** 嵌入二进制，会把库文件嵌入到APP的Bundle中（ipa包中），程序运行时会从bundle中加载库。

**Linked Frameworks and Libraries：**链接库。

#### 1.1 静态库

静态库.a，会直接拷贝到可执行文件中(二进制文件)，不需要再运行时解决依赖问题。

**使用时需要指定依赖库信息**。

#### 1.2 动态库

动态库.dylib或者系统的framework，默认是在系统中的，运行时会从系统中加载。共用。

**动态库中包含了依赖的信息，使用时直接使用动态库即可**。

> 注意：系统的`.framework`为动态库，开发者创建的动态库不允许上线App Store。  **？？？？？**
 
##### 1.2.1 程序调用动态库

在程序的 pre-main 中，会先加载 macho文件，然后加载动态链接库（进行 rebase 指针调整和 bind 符号绑定），此时会将动态库加载到分配的内存空间，然后根据动态库中的依赖信息，将其包含的依赖库也加载到内存地址空间。

这些动态库在设备中只存在一份，通过 mapping 机制让他们同时存在多个应用的内存地址。

> 问题：物理内存中只存在一份，如果多个应用使用同一个变量，是如何确保没有问题的？
> 
> Copy on Write

### 2. 打包库文件

#### 2.1 使用Xcode创建framework工程或者static library工程
网上有很多教程，介绍也很多。

#### 2.2 使用Cocoapods插件打包
主要是用到了插件：[cocoapods-packager](https://github.com/CocoaPods/cocoapods-packager)。通过修改`.podspec`文件的配置进行打包。

命令：`pod package XXX.podspec`

### 3. `.a`、`.framework`和`.embeddedframework`的区别

- .a：只有代码，不包含图片等资源文件，需要配合`.h`。
- .framework：除了包含代码文件，还有图片等资源文件，资源文件在`.framework`目录下。
- .embeddedframework：是一个文件夹，包含`.framework`和资源文件的替身。

`.framework`如果包含资源的bundle，而且没有把bundle添加到工程中的话，会因找不到bundle而崩溃。
> `.framework`内部的二进制文件和资源文件会一起被打进可执行文件中；
> `.embeddedframework`中的二进制文件会被打进可执行文件中，资源文件作为bundle存放在app的bundle内。

### 4 Build Phases

- Target Dependencies: 项目依赖。
- Compile Sources: 项目源文件.m，会打进二进制文件中。
- Link Binary With Libraries: 链接的库文件和二进制文件，会打进二进制文件中。(Cocoapods-packager 是根据podspec文件进行打包)。
- Copy Bundle Resources: 项目的资源文件。

##### 注意事项
1. 插件打包，如果包含第三方framework，需要将其中的替身文件删除，否则会因识别不了替身文件导致打包出错。
![](./images/framework_error.tiff)

2. Pod package 编译时 需要将本地代码commit和清空`~/Library/Caches/CocoaPods/Pods/External/FrameworkName` 否则会导致新添加内容不会打进SDK中。