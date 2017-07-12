# Crash

## Crash收集
* 连接设备，window->devices->View device logs
* 线上：通过iTunes connect，app资讯页面，有一个【Crash Reports】链接。
* 线上：window->Organizer->上方【Crashs】
* iOS设备上保存的Crash，设置->隐私->诊断与用量->诊断与用量数据，这里保存的Crash数据列表。

## Exception Type

**SIGSEGV** 访问了非法的地址(地址还没有从系统映射到当前进程的内存空间), 一般是野指针导致, 而野指针一般由于多线程操作对象导致.

**SIGABRT** 一般是Exception或者其他的代码主动退出的问题.

**SIGTRAP** 代码里面触发了调试指令, 该指令可能由编译器提供的trap方法触发, 如'__builtin_trap()'

**SIGBUS** 一般由于地址对齐问题导致, 单纯的OC代码挺难触发的, 主要是系统库方法或者其他c实现的方法导致

**SIGILL** 表示执行了非法的cpu指令, 但是一般是由于死循环导致
