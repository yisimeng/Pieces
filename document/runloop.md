# Runloop

runloop 是与线程相关的基础结构的一部分，runloop是任务调度和协调接收传入事件的事件处理循环，runloop的作用就是当有任务是让线程忙碌起来，没有时使其睡眠。

runloop的管理不是完全自动的，你需要设计线程代码，在适当的时间start runloop响应传入的事件。Cocoa 和 Core Foundation 都提供了runloop对象来帮助配置和管理线程的runloop。你的应用不需要显示的去创建这些对象，包括应用主线程在内的每一个线程，都有一个对应的runloop对象。只有子线程需要显示的去start runloop，主线程会在程序启动的时候自动设置并运行。

## 剖析runloop

runloop 运行循环，见名知意，是线程中用于响应接收的事件，并处理的一个循环，就像是一个while或for的循环语句，在循环中，使用runloop对象来开始接收事件并调用处理任务。

一个runloop从两个sources来接收事件：input sources 和 timer sources，这两种类型都是用指定处理程序来处理事件。
* Input sources（输入源）: 传递异步事件，通常是其他线程或者其他应用的消息；
* Timer sources（定时器源）: 交付同步事件，在预定的时间或者重复的间隔调用。

以下是runloop与sources的结构示意图，input sources 异步传递时间使得`runUntilDate:`方法跳出循环，而timer sources同步处理则不会跳出循环。

![runloop与源的结构示意图](sources/structure_of_runloop.jpg)

除了处理输入源，runloop还会生成关于运行循环行为的通知，注册 runloop的观察者可以接收到通知，并用来进行其他处理，你可以使用Core Foundation在线程上运行观察者。

### Run Loop Modes

run loop modes 是要监视的 input sources 和 timer sources，以及要通知的runloop观察者的集合。每次运行runloop时，需要指定（显式或者隐式）一个mode。在runloop的整个过程中，只有与该模式相关联的源才会受到监控，并被允许传递它们的事件。（类似地，只有与该mode关联的观察者才会被通知runloop的进度。）与其他mode相关联的sources将被保留，直到之后有合适的mode来通过循环。

在代码中可以通过名称来标识modes。Cocoa和Core Foundation都定义了默认mode和一些常用的modes，以及在代码中指定mode的字符串。可以通过为mode名称指定自定义字符串来定义自定义mode，自定义mode的名称可以是任意的，但是内容不是，必须添加一个或多个input sources，timer sources，或者runloop的观察者。

在runloop的特定循环期间，使用mode来过滤不需要的源的事件。大部分情况下我们都希望在系统默认的‘default mode’中运行runloop。modal panel（模态面板） 运行在 ‘modal’mode，在这个mode下，线程中只有modal panel相关的源才能传递事件。对于辅助线程，您可以使用自定义模式来防止低优先级sources在时间紧迫的操作期间传递事件。

> modes区分基于事件的源，而不是事件的类型。例如，不能使用mode来只匹配鼠标点击事件或者只匹配键盘事件，可以使用modes来监听一组不同的端口，暂停计时器，或者更改runloop当前正在被监听的sources。

下表是Cocoa和Core Foundation定义的标准modes以及介绍。名称列 是代码中定义好的常量。

| mode | 名称 | 描述 |
| :-- | :-- | :-- |
| Default | NSDefaultRunLoopMode(Cocoa) / kCFRunLoopDefaultMode(Core Foundation) | default mode 用于大部分的操作，大部分情况下你需要使用这个mode来开启runloop，配置input sources |
| Connection | NSConnectionReplyMode(Cocoa) | Cocoa 将此mode与NSConnection对象结合使用以监视应答，我们很少需要使用这个mode  |
| Modal | NSModalPanelRunLoopMode(Cocoa) | Cocoa 使用此mode来标识modal panel的事件 |
| Event tracking | NSEventTrackingRunLoopMode(Cocoa) | Cocoa 使用此mode来限制用户界面跟踪循环期间的传入事件 |
| Common modes | NSRunLoopCommonModes(Cocoa) / kCFRunLoopCommonModes(Core Foundation) | 这是一组可配置的通用modes，将输入源与此mode关联，也将其与组中每个mode相关联。Cocoa程序中，默认包含default、modal和event tracking mode。Core Foundation 中初始化时只包含 default mode，我们可以使用 `CFRunLoopAddCommonMode` 方法添加自定义modes到集合中 |