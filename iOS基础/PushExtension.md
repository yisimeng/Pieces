## 推送扩展

推送扩展包含两个：Notification Service Extension 和 Notification Content Extension。

## Notification Service Extension

[文档](https://developer.apple.com/documentation/usernotifications/modifying_content_in_newly_delivered_notifications)

在消息通知到达设备之后，会先被传递到这个扩展中，在展示之前，可以修改通知所携带的内容。

在这里可以做的事情：

* 解密被加密发送过来的数据。
* 可以下载多媒体数据，或者其他附件信息。
* 通过整合来自用户设备的数据，更新通知的内容。

> 扩展仅对开放alert的有效，如果在通知设置中，关闭了alert，将不会到达扩展。

#### NotificationService 中默认的方法

* 使用`didReceive(_:withContentHandler:)` 方法，创建一个新的 UNMutableNotificationContent 对象回调回去。
* `serviceExtensionTimeWillExpire()` 方法表示通知扩展快要超时，如果未完成修改，可以选择放弃，或者返回默认的。

`didReceive(_:withContentHandler:)` 有大约**30s**的时间来进行数据处理，如果超时，系统将调用`serviceExtensionTimeWillExpire()`方法，必须立即向系统返回数据，否则系统将按原始数据进行展示。

## Notification Content Extension

[文档](https://developer.apple.com/documentation/usernotificationsui/customizing_the_appearance_of_notifications)

可以自定义通知展示样式的扩展。

设备收到横幅的通知时，系统将分为两个阶段显示通知内容。

1. 基础的横幅alert，仅包含标题，副标题，和通知内容2-4行的正文文本。
2. 通过长按（或者左滑点击查看），将显示完整的通知界面，系统提供了基础版的界面，中间可以通过扩展来显示自定义的界面内容。

Notification Content Extension 管理了一个视图控制器，继承自UIViewController。

#### NotificationViewController 中默认的方法

* `viewDidLoad` 可以添加自定义view等。
* 实现了 **UNNotificationContentExtension** 的代理方法：`\- (void)didReceiveNotification:(UNNotification *)notification`,获取到通知内容后可以给自定义view赋值。
* **UNNotificationContentExtension** 的可选方法：`\- (void)didReceiveNotificationResponse:(UNNotificationResponse *)response completionHandler:(void (^)(UNNotificationContentExtensionResponseOption))completion` 收到通知扩展的事件响应，这个可以配合。

