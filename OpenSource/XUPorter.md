# XUPorter

喵神写的Unity编译值Xcode工程后自动添加文件和库的插件

基本摘抄自 [喵神XUPorter博客地址](https://onevcat.com/2012/12/xuporter/)	[Github](https://github.com/onevcat/XUPorter)

#### 介绍

由于Unity是跨平台开发，开发中需要针对不同平台添加不同的依赖（源码/库/框架等）。有一些特定的功能实现还是要等在Unity生成iOS工程后进行开发，XUPorter就是在生成iOS工程后自动引入依赖的插件。

XUPorter可以读取Xcode工程文件并进行解析，之后在Unity工程的Assets目录下寻找所有的.projmods文件，并根据文件内容向工程中添加文件或库。

### 使用方法

将XUPorter文件夹copy到Unity工程文件夹下的/Assets/Editor目录中。

XUPorter的目录（删了部分示例文件）:

```
XUPorter/
├── LICENSE
├── MiniJSON
│   └── MiniJSON.cs
├── Mods
│   ├── GameKit.projmods
│   └── iOS
│       └── GameCenter
│          ├── GKWrapper.h
│          └── GKWrapper.m
├── PBX Editor
│   ├── PBXBuildFile.cs
│   ├── PBXBuildPhase.cs
│   ├── PBXDictionary.cs
│   ├── PBXFileReference.cs
│   ├── PBXGroup.cs
│   ├── PBXList.cs
│   ├── PBXObject.cs
│   ├── PBXParser.cs
│   ├── PBXProject.cs
│   ├── PBXSortedDictionary.cs
│   └── PBXVariantGroup.cs
├── Plist
│   └── Plist.cs
├── Readme.mdown
├── XCBuildConfiguration.cs
├── XCConfigurationList.cs
├── XCMod.cs
├── XCPlist.cs
├── XCProject.cs
└── XCodePostProcess.cs
```

#### .projmods文件

其中iOS文件夹下为要导入到Xcode项目里的文件/库；.projmods文件是JSON格式的配置patch文件，定义了如何设置XCode工程：

```
    "group": "KKKeychain",
    "libs": [],
    "frameworks": ["Security.framework"],
    "headerpaths": [],
    "files":   [],
    "folders": ["iOS/KKKeychain/"],
    "linker_flags": [],
    "excludes": ["^.*.meta$", "^.*.mdown$", "^.*.pdf$"]
    "plist": {
        "NSCameraUsageDescription" : "需要使用您的摄像头",
     }
```

参数定义如下：

 * group：所有由该projmods添加的文件和文件夹所属的Xcode中的group名称
 * libs：在Xcode Build Phases中需要添加的动态链接库的名称，比如libz.dylib
 * frameworks：在Xcode Build Phases中需要添加的框架的名称，比如Security.framework
 * headerpaths：Xcode中编译设置中的Header Search Paths路径
 * files：加入工程的文件名
 * folders：加入工程的文件夹，其中所有的文件和文件夹都将被加入工程中
 * linker_flags：添加到工程linker flag中的链接配置，比如-ObjC
 * excludes：忽略的文件的正则表达式，匹配的文件将不会被加入工程中
 * plist：项目中 info.plist 的设置

#### XcodePostProcess.cs

Unity 会在编译完成之后调用XcodePostProcess.cs，可以在文件里对应的方法中添加对编译的修改。

##### 修改工程编译项： OnPostProcessBuild 方法
例如：

```
project.overwriteBuildSetting("DEBUG_INFORMATION_FORMAT", "dwarf-with-dsym");
```

##### 修改工程代码
也可以对XcodePostProcess.cs做一些扩展，例如修改代码。

```
namespace UnityEditor.XCodeEditor{
	public partial class XClass : System.IDisposable{
		private string filePath;
		public XClass(string fPath){
			filePath = fPath;
			if( !System.IO.File.Exists( filePath ) ) {
				Debug.LogError( filePath +"路径下文件不存在" );
				return;
			}
		}
		// 在代码后添加新代码
		public void WriteBelow(string below, string text){
			StreamReader streamReader = new StreamReader(filePath);
			string text_all = streamReader.ReadToEnd();
			streamReader.Close();
			int beginIndex = text_all.IndexOf(below);
			if(beginIndex == -1){
				Debug.LogError(filePath +"中没有找到标致"+below);
				return; 
			}
			int endIndex = text_all.LastIndexOf("\n", beginIndex + below.Length);
			text_all = text_all.Substring(0, endIndex) + "\n"+text+"\n" + text_all.Substring(endIndex);
			StreamWriter streamWriter = new StreamWriter(filePath);
			streamWriter.Write(text_all);
			streamWriter.Close();
		}
		// 代码替换
		public void Replace(string below, string newText){
			StreamReader streamReader = new StreamReader(filePath);
			string text_all = streamReader.ReadToEnd();
			streamReader.Close();
			int beginIndex = text_all.IndexOf(below);
			if(beginIndex == -1){
				Debug.LogError(filePath +"中没有找到标致"+below);
				return; 
			}
			text_all =  text_all.Replace(below,newText);
			StreamWriter streamWriter = new StreamWriter(filePath);
			streamWriter.Write(text_all);
			streamWriter.Close();
		}
		public void Dispose(){
		}
	}
}
```

修改代码的方法调用

```
//读取UnityAppController.mm文件
XClass UnityAppController = new XClass(filePath + "/Classes/UnityAppController.mm");

//在指定代码后面增加一行代码
UnityAppController.WriteBelow("原文代码","新增代码");

//替换代码
UnityAppController.Replace("原文代码","新增代码");

```

完全可以实现在XcodePostProcess.cs文件中实现对整个工程的配置，可以省去后面的开发成本。

 