## cocoapods-packager

Cocoapods 开源的通过【podspec】文件的配置打包的插件。

命令：`pod package xxx.podspec`。

参数：

```
// 强制覆盖之前生成的文件
--force
// 不使用name-mangling技术，也就是自动改类名等符号
--no-mangle
// 生成静态的framework
--embedded
// 生成静态.a
--library
// 生成动态framework
--dynamic
// 使用本地文件
--local
// 生成动态framework的时候需要这个BundleId来签名
--bundle-identifier
// 不包含依赖的符号表，也就是不把依赖的第三方库打包进去
--exclude-deps
// 生成debug还是release的库，默认是release
--configuration=Release 
// 如果你的pod库有subspec，那么加上这个命名表示只给某个或几个subspec生成二进制库
--subspecs=subspec1,subspec2
// 默认是CocoaPods的Specs仓库，如果你的项目有私有的source，就可以通过这个参数来设置
--spec-sources=private,https://github.com/CocoaPods/Specs.git
```

### 着重介绍一下：name-mangling

[cocoapods-packager mangle.rb 源码](https://github.com/CocoaPods/cocoapods-packager/blob/461686593c521796c723fe5f1c460e2aa2adbe55/lib/cocoapods-packager/mangle.rb)

```
#
  # performs symbol aliasing
  #
  # for each dependency:
  # 	- determine symbols for classes and global constants
  # 	- alias each symbol to Pod#{pod_name}_#{symbol}
  # 	- put defines into `GCC_PREPROCESSOR_DEFINITIONS` for passing to Xcode
  #
```

name-mangling 会把类名和全局常量修改成：`Pod#{pod_name}_#{symbol}` 的形式，比如AFN，会被改成：`PodCustomPod_AFNetworking`。

> **注意1：** name-mangling 不会修改方法名，所以如果三方库中增加了分类的方法，而且接入方也接入了这个三方库，两个都有相同的分类方法，分类方法不会冲突，只会覆盖，一般情况下不会有问题。
> **注意2：** 像上面说的这样使用 name-mangling 技术，会将第三方库修改类名，那么即使接入方也接入了相同的三方库，也不会造成冲突（类名不同）。

#### name-mangling 无法对静态库生效

如果podspec文件中添加了 `vendored_frameworks` 或者 `vendored_libraries`，那么打包的时候会报错：

【错误提示[!] podspec has binary-only depedencies, mangling not possible.】

**name-mangling 对静态库无效。**

如果在打包命令后添加 `--no-mangling` 参数：

```
$ pod package xxx.podspec --embedded --force --no-mangling
```

这样可以打包成功，但是会将第三方库打进去，即使使用了 `--exclude-deps` 参数，只会去除使用的第三方开源库，但是第三方静态库还是会被打进去。如果介入放也使用了第三方静态库，名字相同，肯定会引起冲突。

> 所以如果使用这种方式打包，需要删除 podspec 文件中的`vendored_frameworks` 和 `vendored_libraries`，将静态库手动导入，并添加search path。

或者改成使用 Xcode 的静态库模板的方式打包，新建 Library target，设置search path。

参考：

1. [SDK开发和打包静态库遇到的坑](http://www.jintiankansha.me/t/kWn6e21gXM)