## 网络

### HTTPS

HTTPS是一种通过计算机网络进行安全通信的传输协议，经由HTTP进行通信，利用SSL/TLS建立全信道，加密数据包。HTTPS使用的主要目的是提供对网站服务器的身份认证，同时保护交换数据的隐私与完整性。

> TLS 是传输层安全协议，SSL是安全套接层，SSL是TLS前身。

1. HTTPS的工作原理

与HTTP相同，在数据传输之前，需要客户端和服务端进行握手。具体过程如下：

* 客户端将自己支持的一套加密规则和随机数A发送至服务器；
* 服务器从中选出一组加密算法，加上服务器的证书（包含网站地址，公钥，颁发机构的等信息），并生成一个随机数B，返回给客户端；
* 客户端验证证书颁发机构是否合法，证书中包含地址是否与当前一致等。如果信任，将生成随机数C，用公钥加密发送服务端。
* 服务端用私钥解密获得随机值，通过三个随机数生成**会话私钥**。至此握手结束，之后通信就是HTTP协议，只不过内容使用**会话私钥**进行**对称**加密的。

由于最终第三个随机数是用服务器的公钥进行加密的，只能由服务器的私钥进行解密，第三方无法获取这个值，也就无法知道最终生成的对称加密的会话密钥。

#### 为什么Charles还可以进行HTTPS抓包？

这是由于在抓包之前需要在Mac上和iPhone上安装证书并信任，**配置iPhone代理**，iPhone 发起的请求需要通过 Mac 上的 Charles。

1. 客户端发起握手请求；
2. Charles拦截请求，并将请求内容进行提取，然后自身模仿客户端向服务器发起请求；
3. 服务器收到请求之后，将证书等信息进行返回；
4. Charles拦截返回的信息，将证书替换为Charles的证书，返回给客户端；
5. 客户端由于信任了证书，因此会将信息使用Charles的公钥进行加密，并发送出去；
6. Charles再次拦截，通过私钥解密信息后可以获得“会话私钥A”，同时再将内容通过服务器公钥进行加密后发送至服务器；
7. 服务器收到之后使用服务器的私钥解密后，生成“会话密钥B”。
8. 握手结束。至此以后的所有信息传输，Charles都会进行拦截、包装、发送。

> 在客户端与Charles的交互过程中，Charles始终充当服务器的角色；而在Charles接收到客户端的请求之后，又会将自己伪装成客户端的角色，向服务器发送请求。

##### 如何防止HTTPS进行抓包

从上述Charles抓包HTTPS请求的过程中有一步，是设置代理，网络请求都需要经过Mac。所以判断当前如果是被设置了代理了，就有可能被抓包。

判断代理方法：

```objective-c
+ (BOOL)getProxyStatus {
    CFDictionaryRef dicRef = CFNetworkCopySystemProxySettings();
    const CFStringRef proxyCFstr = CFDictionaryGetValue(dicRef, (const void*)kCFNetworkProxiesHTTPProxy);
    CFRelease(dicRef);
    NSString *proxy = (__bridge NSString*)(proxyCFstr);
    if(proxy) {
        return YES;
    }
    return NO;
}
```

