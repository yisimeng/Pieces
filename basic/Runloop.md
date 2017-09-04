## RunLoop

RunLoop：让线程能随时处理事件但并不退出的一种机制，其本质是一个do-while循环。

RunLoop的关键机制在于：管理事件/消息，让线程在没有处理消息时休眠以避免资源占用、在有消息到来时立刻被唤醒。

RunLoop和线程是对应的，一个线程至少有一个 RunLoop，以 Key-Value 存放在一个全局字典中。RunLoop不能手动创建，只能获取。

```
	 /// RunLoop 源码，获取 RunLoop 的部分实现
    CFRunLoopRef loop = CFDictionaryGetValue(loopsDic, thread));     
    if (!loop) {
        loop = _CFRunLoopCreate();
        CFDictionarySetValue(loopsDic, thread, loop);
        /// 注册一个回调，当线程销毁时，顺便也销毁其对应的 RunLoop。
        _CFSetTSD(..., thread, loop, __CFFinalizeRunLoop);
    }
    
```

CoreFoundation 中RunLoop的五个类

* CFRunLoopRef
* CFRunLoopModeRef
* CFRunLoopSourceRef
* CFRunLoopTimerRef
* CFRunLoopObserverRef

RunLoop中可以包含多个Mode，每个Mode中又包含多个 mode item，mode item 可以是 Source/Timer/Observer。

RunLoop 的运行必须指定一个 Mode，即currentMode。切换时必须先退出当前Mode，这样不同组的 mode item 可以互不干扰。

一个 item 可以被加入多个mode中，但是一个 item 被重复添加入 mode 不会有效果。如果 mode 中没有 item，RunLoop会退出。

**CFRunLoopSourceRef** 事件源，source0和source1

* source0 只包含一个回调，不能主动触发。 负责App内部事件，由App负责管理触发，例如触摸事件。
* source1 包好一个mach_port和一个回调，可以监听系统端口和其他线程互发消息，能主动唤醒RunLoop。

**CFRunLoopTimerRef** 时间源，包含一个时间点和一个回调，加入RunLoop后，会注册时间点，当到达时间点事，会唤醒RunLoop执行回调。

**CFRunLoopObserverRef** 观察者，检测RunLoop状态变化。

```
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry         = (1UL << 0), // 即将进入Loop
    kCFRunLoopBeforeTimers  = (1UL << 1), // 即将处理 Timer
    kCFRunLoopBeforeSources = (1UL << 2), // 即将处理 Source
    kCFRunLoopBeforeWaiting = (1UL << 5), // 即将进入休眠
    kCFRunLoopAfterWaiting  = (1UL << 6), // 刚从休眠中唤醒
    kCFRunLoopExit          = (1UL << 7), // 即将退出Loop
};
```

### RunLoop Modes

```
struct __CFRunLoopMode {
    CFStringRef _name;            // Mode Name, 例如 @"kCFRunLoopDefaultMode"
    CFMutableSetRef _sources0;    // Set
    CFMutableSetRef _sources1;    // Set
    CFMutableArrayRef _observers; // Array
    CFMutableArrayRef _timers;    // Array
    ...
};
  
struct __CFRunLoop {
    CFMutableSetRef _commonModes;     // Set
    CFMutableSetRef _commonModeItems; // Set
    CFRunLoopModeRef _currentMode;    // Current Runloop Mode
    CFMutableSetRef _modes;           // Set
    ...
};
```

* **NSDefaultRunLoopMode/kCFRunLoopDefaultMode:**默认Mode，通常主线程在所运行的Mode。
* **NSConnectionReplyMode:**Cocoa使用该模式与NSConnection对象联接用于监控响应。此模式用于处理NSConnection的回调事件,你应该很少用的此模式。
* **NSModalPanelRunLoopMode:**模态模式，此模式下，RunLoop只对处理模态相关事件
* **NSEventTrackingRunLoopMode:**此模式下限定引入事件在鼠标拖拽和其他的一些用户界面跟踪中。Cocoa使用该模式来限制光标拖动循环中上报的事件或其它用户界面相关的track loop。
* **NSRunLoopCommonModes/kCFRunLoopCommonModes:**这是一组可配置的通用模式。将input sources与该模式关联则同时也将input sources与该组中的其它模式进行了关联。对于Cocoa应用，该模式缺省的包含了default，modal以及event tracking模式。而Core Foundation则在初始化时只包含了default模式。你可以使用CFRunLoopAddCommonMode为该模式添加自定义的模式。

> Common Mode：RunLoop 切换时，会自动将 _commonModeItems 里的 item，同步到具有 Common 标记的所有 Mode 里。将 timer 加入到 commom 中，RunLoop会将 commonModeItems 里的item 同步到所有具有Commom属性的Mode里，这样在 RunLoop 切换到UITrackingRunLoopMode中时，依然可以回调。
