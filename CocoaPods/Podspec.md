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

### 4. 当项目中包含 MRC 的文件，脚本编译报错

在 OC 项目中使用 protobuf 时，需要在 【Build Phases】中的【Compile Sources】中的protobuf 的文件 xx.proto.m 后添加 `-fno-objc-arc`， xx.proto.m 是 mrc 的，这样 Xcode 就能编译通过了。

但是在使用 `cocoapods-packager` 插件打包时，现在工程一般都是 ARC， 所以 .podspec 文件中一般是这样设置的 `s.requires_arc = true`。编译时，系统会以所有文件都是 ARC 来编译，导致 xx.proto.m 编译失败。

解决方法：将 MRC 的文件单独出来一个 subspec 来管理，这个 subspec 是非 ARC 的。

```
  s.default_subspec = 'mrc'

  # 这是需要添加mrc标识的文件，为相对路径
  non_arc_files = 'Pod/Classes/TimeSocket/Proto/ProtobufReport.pbobjc.m'
  # 将所有 mrc 文件从项目中排除
  s.exclude_files = non_arc_files
  # subspec 为需要添加mrc标识的文件进行设置
  s.subspec 'mrc' do |sp|
      sp.source_files = non_arc_files
      sp.requires_arc = false
  end
  
  ···
```

