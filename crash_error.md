# crash 和 error 记录

## crash
### 收集crash
* 连接设备，window->devices->View device logs
* 线上：通过iTunes connect，app资讯页面，有一个【Crash Reports】链接。
* 线上：window->Organizer->上方【Crashs】
* iOS设备上保存的Crash，设置->隐私->诊断与用量->诊断与用量数据，这里保存的Crash数据列表。

### Exception Type

**SIGSEGV** 访问了非法的地址(地址还没有从系统映射到当前进程的内存空间), 一般是野指针导致, 而野指针一般由于多线程操作对象导致.

**SIGABRT** 一般是Exception或者其他的代码主动退出的问题.

**SIGTRAP** 代码里面触发了调试指令, 该指令可能由编译器提供的trap方法触发, 如'__builtin_trap()'

**SIGBUS** 一般由于地址对齐问题导致, 单纯的OC代码挺难触发的, 主要是系统库方法或者其他c实现的方法导致

**SIGILL** 表示执行了非法的cpu指令, 但是一般是由于死循环导致


## Error

#### 编译器报错
1. library not found for -liPhone-lib

	**描述：**Unity 导出 Xcode 工程编译报错。

	**原因：**library 编译路径错误。

	**解决：**Library search paths 中去掉```"$(SRCROOT)/Libraries"```的引号。

2. ```ld: warning: directory not found for option '-F/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator10.2.sdk/Developer/Library/Frameworks'```
<img src="./images/找不到framework.png" width = "695" height = "37" alt="图片名称">
	**原因：**找不到framework库
	**解决：**Framework search Paths 中替换为```$(PLATFORM_DIR)/Developer/Library/Frameworks```。
	<img src="./images/解决方案.png" width = "801" height = "193" alt="图片名称">
	
3. ProductName was compiled with optimization - stepping may behave oddly; variables may not be available.

	**描述：**调试时断点后打印信息。
	**原因：**release模式下编译会做一些优化，导致单步程序异常，变量不可访问。
	**解决：**编译方式改为debug。
