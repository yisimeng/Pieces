## NSProxy

NSObject 和 NSProxy 是OC中的两个根类。

```objective-c
@interface NSProxy <NSObject> {
    Class	isa;
}
```

```objective-c
@interface NSObject <NSObject> {
    Class isa  OBJC_ISA_AVAILABILITY;
}
```

官方定义：NSProxy是一个抽象的superClass，实现了NSObject协议，定义了对象的API，作为其他对象（或者不存在的对象）的替身。

发送给NSProxy的消息会被转发给实际的对象，NSProxy的子类可用于透明的分发消息（例如：NSDistantObject），或者重大开销对象的懒初始化。

NSProxy实现了NSObject协议，实现了跟类所需要的基本方法。抽象类不需要提供初始化方法，它在接收到不能响应的方法后会抛出异常。因此，它的具体实现子类必须停工一个初始化方法，并重写`forwardInvocation(_:)`方法和`methodSignatureForSelector:`来处理他自己没有实现的消息，即将消息转发给真正的对象。

#### NSProxy 与 NSObject 消息转发

NSProxy 做消息转发效率更高。

NSObject：本类->父类->根类->动态方法解析->消息转发；

NSProxy：本类->消息转发。

### 用途

1. 模拟多继承。NSProxy把多个类的消息，转发给实现了这些方法的对象。并非是真正的多继承。
2. 在不修改源码的情况下hook对象的某个方法。NSProxy持有实际对象，在转发消息之前，可以进行自己的操作。
3. 解决NSTimer的循环引用。NSProxy持有self，作为NSTimer的target，解决循环引用。