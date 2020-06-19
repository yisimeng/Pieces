# App Extension

应用扩展。

#### 应用扩展中不可以做的的事情：

1. 不可使用sharedApplication对象。
2. 不能使用标记 NS_EXTENSION_UNAVAILABLE 的API。
3. 运行一个长时间的后台任务（根据不同平台而异）

## 推送扩展

推送扩展包含两个：Notification Service Extension 和 Notification Content Extension。

### Notification Service Extension

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

### Notification Content Extension

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

### UNNotificationCategory

[文档](https://developer.apple.com/documentation/usernotifications/declaring_your_actionable_notification_types)

通过 Category 对通知进行区分，以显示不同的页面按钮。

在离线状态下，接收到通知之后，不用启动应用即可进行一些快速处理。

#### UNNotificationAction & UNTextInputNotificationAction

显示通知详情时（长按或者左滑查看），支持的快捷操作按钮。

UNTextInputNotificationAction 快捷回复输入操作。

一个Category可以包含多个Action，但如果只有一个Action且为 **UNTextInputNotificationAction** 时，将直接弹出键盘。

#### 实现步骤

#### 1. 声明自定义操作和通知类型

在应用启动时设置支持的通知类型，可以定义多个类型的Category，分别有不同的自定义操作。

`[[UNUserNotificationCenter currentNotificationCenter] setNotificationCategories: [NSSet setWithObjects:noneCategory, commentCategory, messageCategory, nil]];`

#### 2. 接收自定义操作响应

一般可在 AppDelegate 中设置 `[UNUserNotificationCenter currentNotificationCenter] `的代理，实现 UNUserNotificationCenterDelegate 代理方法用以接收事件响应：

`\- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void(^)(void))completionHandler `

#### 3. 离线推送支持自定义操作

在NotificationContent 的 Info.plist 文件中，找到 NSExtension -> NSExtensionAttributes -> UNNotificationExtensionCategory 字段。

UNNotificationExtensionCategory 对应的value，如果只有一个Category的话，可以使String，如果多个Category，则为Array，设置 Category 的 identifier。

在 NotificationViewController 的 `\- (void)didReceiveNotificationResponse:(UNNotificationResponse *)response completionHandler:(void (^)(UNNotificationContentExtensionResponseOption))completion`方法中处理事件回调。

#### 4. 发送通知

本地通知：设置 **UNMutableNotificationContent** 的 **categoryIdentifier** 属性

远程通知：在 **aps** 字段下添加 **category** 字段，并设置 identifier。

#### 举例

应用可能收到的推送信息有：IM消息，朋友动态消息（类似朋友圈），系统通知。

不同消息类型的自定义操作也不同：

* IM消息：长按快捷回复。
* 朋友动态消息：点赞、评论。
* 系统通知：不需要快捷操作。

就可以根据消息类型分别设置Category的Identifier：

* IM消息：category.message。
* 朋友动态消息：category.comment。
* 系统通知：category.system。

最后在回调中去根据不同的category id 和 action id  进行处理。



# 注意

**Payload 大小不能超过4K**

```
Put the JSON payload with the notification’s content into the body of your request. The JSON payload must not be compressed and is limited to a maximum size of 4 KB (4096 bytes). For a Voice over Internet Protocol (VoIP) notification, the maximum size is 5 KB (5120 bytes).
```

