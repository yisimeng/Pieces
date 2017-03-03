## Runloop
**RunLoop：**让线程能随时处理事件但并不退出的一种机制
Runloop的本质就是一个do-while循环，只要条件满足，就会不停的循环，进而程序一直保持运行的状态。
Runloop的关键机制在于：如何管理事件/消息，如何让线程在没有处理消息时休眠以避免资源占用、在有消息到来时立刻被唤醒。

### Run Loop Modes
系统默认有五种mode：
* **NSDefaultRunLoopMode/kCFRunLoopDefaultMode:**大多数操作中使用的模式。大多数时间，你使用该模式来启动run loop并配置你的input sources。
* **NSConnectionReplyMode:**Cocoa使用该模式与NSConnection对象联接用于监控响应。此模式用于处理NSConnection的回调事件,你应该很少用的此模式。
* **NSModalPanelRunLoopMode:**模态模式，此模式下，RunLoop只对处理模态相关事件
* **NSEventTrackingRunLoopMode:**此模式下限定引入事件在鼠标拖拽和其他的一些用户界面跟踪中。Cocoa使用该模式来限制光标拖动循环中上报的事件或其它用户界面相关的track loop。
* **NSRunLoopCommonModes/kCFRunLoopCommonModes:**这是一组可配置的通用模式。将input sources与该模式关联则同时也将input sources与该组中的其它模式进行了关联。对于Cocoa应用，该模式缺省的包含了default，modal以及event tracking模式。而Core Foundation则在初始化时只包含了default模式。你可以使用CFRunLoopAddCommonMode为该模式添加自定义的模式。
