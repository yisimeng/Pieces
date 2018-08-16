# 静态库、动态库

### 1. 介绍

**静态库：**链接是直接拷贝到可执行文件中，app间不共用。文件为`.a`和`.framework`。

**动态库：**链接时不复制，程序运行时由系统动态加载到内存，系统只加载一次，APP间共用。文件为`.framework`和`.tdb`。

> 注意：系统的`.framework`为动态库，开发者创建的动态库不允许上线App Store。

### 2. 打包库文件

#### 2.1 使用Xcode创建framework工程或者static library工程
网上有很多教程，介绍也很多。

#### 2.2 使用Cocoapods插件打包
主要是用到了插件：[cocoapods-packager](https://github.com/CocoaPods/cocoapods-packager)。通过修改`.podspec`文件的配置进行打包。

命令：`pod package XXX.podspec`

#### 3. `.a`、`.framework`和`.embeddedframework`的区别

**.a：**只有代码，不包含图片等资源文件，需要配合.h。

**.framework：**除了包含代码文件，还有图片等资源文件。
**.embeddedframework：**





.framework 与 .embeddedframework
.framework 只包含代码
.embeddedframework 除了代码还包含资源文件（图片、脚本、xib、bundle等）


Embedded Binaries: 嵌入二进制，会把库文件嵌入到APP的Bundle中，程序运行时会从bundle中加载库
Linked Frameworks and Libraries：链接库。
 - 如果是静态库.a，会直接拷贝到可执行文件中，不需要再运行时解决依赖问题。
 - 如果是动态库.dylib或者系统的framework，默认是在系统中的，运行时会从系统中加载。共用。

Pod package 编译时 需要将本地代码commit和清空~/CocoaPods/Pods/External 否则会导致新添加内容不会打进sdk中