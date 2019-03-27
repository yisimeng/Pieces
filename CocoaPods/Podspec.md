# Podspec

### 1. weak_framework

`spec.weak_framework = ’Twitter’`

弱链接framework，项目兼容多个系统版本，但是有些framework在新版系统中才有，低版本如果强链接framework会导致崩溃。

### 2. 执行 pod install 报错

```The 'Pods-Example' target has transitive dependencies that include static binaries:...```

开启 ```use_framework!```，pod 集成的所有库都会以 dynamic library framework 的方式引入工程，但是开发中经常会有一些以静态库存在的三方库，直接引入会导致报错。

解决方案：

1. CocoaPods1.3.1 及之前不支持静态库框架依赖，1.4.0之后增加 static_framework 选项，可以指定将pod构建为 static framework。
2. 去掉 `use_framework!`。
3. 通过hook安装过程，通过`pre_install`中禁用静态库检查。
```
pre_install do |installer|
def installer.verify_no_static_framework_transitive_dependencies; end
end
```

http://www.cocoachina.com/ios/20170427/19136.html

https://www.jianshu.com/p/56e17c8e3a94?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation

### 3. target 版本过低

```Specs satisfying the `Example (from `../`)` dependency were found, but they required a higher minimum deployment target.```

修改 Podfile 中 platform 的版本。