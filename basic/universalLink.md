## Universal Link 通用链接

iOS 9之后引入的通用链接，运行应用处理之前只能被Safari打开的http、https链接。可以从其他应用直接调起自己的应用，不用再通过Safari。

1. 通过`openURL:`打开http或https的URL。
2. 系统会检测已安装的应用是否能处理URL。如果能，则直接启动应用；否则由Safari打开。

### 1.配置

1. Apple Developer上打开 Associated Domains权限。
2. Target->Capabilities 添加  Associated Domains，并设置Domains。格式：`applinks:your.website.address`。
3. 网站根目录或者.well-known目录下放配置好的`apple-app-site-association`文件。
4. AppDelegate实现回调方法`- (**BOOL**)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity
    restorationHandler:(**void** (^)(NSArray * restorableObjects))restorationHandler`。



`apple-app-site-association`文件格式：

```json
{
  "applinks": {
    "details": { 
      "ABCDEFGHIJ.com.mydomain.bundleid": {  
        "paths": [ 
          "/mypath/",
          "/basepath/*"
        ]
      }
    }
  }
}
```

其中 paths 为可处理的路径列表，可以通过深度定制启动应用并打开对应页面。