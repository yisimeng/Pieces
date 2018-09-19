# xcodeproj

**xcodeproj** 是 CocoaPods 的一个组件，可用于修改 Xcode 的工程配置。

### project.pbxproj介绍

Xcode 工程的 **projectName.xcodeproj** 的文件下都有一个 **project.pbxproj** 文件，
采用老式的 plist 文件（old ASCII plist），最早是Next公司使用的一种文件格式，类似于XML。

**project.pbxproj** 中的文件信息：

	源代码，包含头文件和实现文件；
	内部和外部的静态库和动态库；			
	资源文件、图片文件、界面构建文件(nib)；
	编译配置信息。

**project.pbxproj** 的目录：

![](../images/pbxproj_dir)

**project.pbxproj** 中的每个元素都由isa来标识类型，元素：

	PBXBuildFile	// 文件
	PBXBuildPhase // 编译阶段
	PBXAppleScriptBuildPhase 
	PBXCopyFilesBuildPhase // 编译阶段需要copy到bundle的文件
	PBXFrameworksBuildPhase // 编译阶段链接的framework
	PBXHeadersBuildPhase // 编译阶段的头文件
	PBXResourcesBuildPhase // 编译阶段的资源文件
	PBXShellScriptBuildPhase // 执行的shell脚本
	PBXSourcesBuildPhase // 编译阶段的源文件
	PBXContainerItemProxy 
	PBXFileElement 
	PBXFileReference  // 跟踪文件路径
	PBXGroup // 文件组
	PBXVariantGroup 
	PBXTarget 
	PBXAggregateTarget 
	PBXLegacyTarget 
	PBXNativeTarget // 项目中的target
	PBXProject // project 工程
	PBXTargetDependency 
	XCBuildConfiguration // 编译配置
	XCConfigurationList // Release/Debug
文件中 Xcode 为每个元素生成了一个唯一标识码，是一个24位的16进制数，代表每一个object。没有确切文档具体说明标识码的生成算法，但是，标识码不止在项目中是唯一的，跨工程也是唯一的。

[xcodeproj API](https://www.rubydoc.info/gems/xcodeproj)

示例1： 打开工程

```
require 'xcodeproj'

#打开项目工程A.xcodeproj
project_path = 'Pods/Pods.xcodeproj'
project = Xcodeproj::Project.open(project_path)

<!--工程修改-->

# 保存
project.save
```

示例2：移除header引用

```
project.targets.each do |target|
    if target.name == "AmuletSDK"
		#找到要插入的group (参数中true/false表示如果找不到group，是否创建一个group)
        group = project.main_group.find_subpath(File.join('Development Pods/AmuletSDK'),false)
        #遍历group的children
        group.children.each do |child|
            if child.name == "AMLoginManager.h"
            	#从工程中移除引用
                child.remove_from_project
            end
        end
    end
end
```

示例3：证书配置

```
project.targets[0].build_configurations.each do |config|
  if config.name == 'Debug'
    config.build_settings["PROVISIONING_PROFILE_SPECIFIER"] = "xxProfileName"
    config.build_settings["DEVELOPMENT_TEAM"] = "xxTeamName"
    config.build_settings["CODE_SIGN_IDENTITY"] = "xxIdentityName"
    config.build_settings["CODE_SIGN_IDENTITY[sdk=iphoneos*]"] = "iPhone Developer"
  end
end
```

参考：

[ruby库xcodeproj使用心得](https://blog.csdn.net/darya_1/article/details/78095821)

[通过Xcodeproj深入探究Xcode工程文件](http://www.cocoachina.com/ios/20161008/17689.html)

[xcodeproj API](https://www.rubydoc.info/gems/xcodeproj)