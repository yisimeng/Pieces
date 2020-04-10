## 数据共享

### 1. Deep Link

使用统一的资源标识符（Uniform Resource Identifier,URI），链接到应用内特定的位置。

iOS 通过定义 URL Scheme ，在设备中确保唯一的 scheme，系统会自动调用应用去处理。

应用不能响应一下的URL Scheme

* http、https：网络scheme，由Safari处理。YouTube链接例外，如果已安装由应用打开。
* mailto：发送电子邮件的scheme，由邮件应用处理。
* itms、itms-apps：由 AppStore处理。
* tel：拨打电话。
* app-settings：跳转设置。

URL 格式 `scheme://host/path?query#fragment`

1. `[UIApplication canOpnenURL:]` scheme 是否可以被处理
2. `[UIApplication openURL:]` 启动应用
3. 目标应用通过 `[UIAppDelegate application:openURL:sourceApplication:annotation]`接收通知，解析URL处理

#### 1.1 注意

* 使用较短的URL，构建速度和解析速度快
* 避免基于正则表达式的模式。
* 优先选择基于查询的URL进行标准解析。
* 在URL中支持deep-link回调，完善逻辑，例如：启动照片编辑器完成编辑后，携带照片返回原应用；身份验证是否成功。`x-callback-url`规范支持这些回调。
* 不要在URL中放置任何敏感数据。
* 不要相信任何传入数据。始终验证URL。
* 使用 sourceApplication 来标识源。

### 2. 剪贴板



