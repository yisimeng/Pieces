# YYAsyncLayer

异步绘制layer，通过注册Runloop通知（kCFRunLoopBeforeWaiting | kCFRunLoopExit），在Runloop休眠之前，CoreAnimation执行完绘制任务之后，通过YYTransaction将绘制任务提交到Runloop。

YYAsyncLayer 是CALayer的子类，通过重写display方法，当要显示内容时，会向其delegate\<YYAsyncLayerDelegate\>（通常是UIView或其子类）请求一个异步绘制任务：`- (YYAsyncLayerDisplayTask *)newAsyncDisplayTask` ,可以将绘制任务通过这个方法进行返回。

YYAsyncLayerDisplayTask 类中是三个Block，willDisplay、display、didDisplay，来监听当前绘制状态。

### YYTransaction

注册Runloop通知，与Runloop回调方法

```objective-c
// 事务集合
static NSMutableSet *transactionSet = nil;
// runloop回调方法
static void YYRunLoopObserverCallBack(CFRunLoopObserverRef observer, CFRunLoopActivity activity, void *info) {
	  //如果当前没有任务，直接返回
    if (transactionSet.count == 0) return;
    // 取出当前已有的所有任务，并清空集合
    NSSet *currentSet = transactionSet;
    transactionSet = [NSMutableSet new];
    // 遍历执行任务
    [currentSet enumerateObjectsUsingBlock:^(YYTransaction *transaction, BOOL *stop) {
        [transaction.target performSelector:transaction.selector];
    }];
}
// 注册Runloop通知
static void YYTransactionSetup() {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        transactionSet = [NSMutableSet new];
        CFRunLoopRef runloop = CFRunLoopGetMain();
        CFRunLoopObserverRef observer;
        observer = CFRunLoopObserverCreate(CFAllocatorGetDefault(),
                                           // 注册runloop 的 before waiting 和 exit 通知
                                           kCFRunLoopBeforeWaiting | kCFRunLoopExit,
                                           true,      // repeat
                                           0xFFFFFF,  // after CATransaction(2000000)
                                           YYRunLoopObserverCallBack, NULL);
        CFRunLoopAddObserver(runloop, observer, kCFRunLoopCommonModes);
        CFRelease(observer);
    });
}
```

YYTransaction

```objective-c
@interface YYTransaction()
@property (nonatomic, strong) id target;
@property (nonatomic, assign) SEL selector;
@end
  
@implementation YYTransaction
+ (YYTransaction *)transactionWithTarget:(id)target selector:(SEL)selector{
    if (!target || !selector) return nil;
    YYTransaction *t = [YYTransaction new];
    t.target = target;
    t.selector = selector;
    return t;
}
// 将任务提交到任务集合中
- (void)commit {
    if (!_target || !_selector) return;
    YYTransactionSetup();
    [transactionSet addObject:self];
}
// 判等，确保不会将重复任务添加到集合中
- (NSUInteger)hash {
    long v1 = (long)((void *)_selector);
    long v2 = (long)_target;
    return v1 ^ v2;
}
- (BOOL)isEqual:(id)object {
    if (self == object) return YES;
    if (![object isMemberOfClass:self.class]) return NO;
    YYTransaction *other = object;
    return other.selector == _selector && other.target == _target;
}

@end
```

### YYSentinel

线程安全的自增标识符，用于判断当前绘制任务是否被取消。

```objective-c
@interface YYSentinel : NSObject
/// 当前的计数
@property (readonly) int32_t value;
/// 自增
- (int32_t)increase;
@end
```

### YYAsyncLayer

异步绘制的layer。包含：YYAsyncLayer、YYAsyncLayerDelegate、YYAsyncLayerDisplayTask。

#### YYAsyncLayerDelegate

异步layer的代理，通常是UIView。只有一个方法：

```objective-c
@protocol YYAsyncLayerDelegate <NSObject>
@required
/// 方法返回一个新的异步绘制任务.
- (YYAsyncLayerDisplayTask *)newAsyncDisplayTask;
@end
```

#### YYAsyncLayerDisplayTask

异步任务，包含三个Block，willDisplay、display、didDisplay。

```objective-c
// 回调任务可能在子线程中，也可能在主线程中
@interface YYAsyncLayerDisplayTask : NSObject
/**
layer 在绘制之前的回调
 */
@property (nullable, nonatomic, copy) void (^willDisplay)(CALayer *layer);
/**
绘制内容中的回调
因为绘制可能是在主线程也可能是在子线程，所以这必须是线程安全的，并提供isCancelled，来随时检查是否任务已经取消，取消就不用再绘制了
 */
@property (nullable, nonatomic, copy) void (^display)(CGContextRef context, CGSize size, BOOL(^isCancelled)(void));
/**
绘制完成的回调，当任务取消时finished为NO，否则为YES
 */
@property (nullable, nonatomic, copy) void (^didDisplay)(CALayer *layer, BOOL finished);

@end
```

#### YYAsyncLayer

```objective-c
@interface YYAsyncLayer : CALayer
/// 是否是异步绘制. Default is YES.
@property BOOL displaysAsynchronously;
@end

@implementation YYAsyncLayer {
  	/// 当前layer的标识符
    YYSentinel *_sentinel;
}
#pragma mark - Override
/// 给一些自定义属性设置默认值
+ (id)defaultValueForKey:(NSString *)key {
    if ([key isEqualToString:@"displaysAsynchronously"]) {
        return @(YES);
    } else {
        return [super defaultValueForKey:key];
    }
}
// 初始化
- (instancetype)init {
    self = [super init];
    static CGFloat scale; //global
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        scale = [UIScreen mainScreen].scale;
    });
    self.contentsScale = scale;
    _sentinel = [YYSentinel new];
    _displaysAsynchronously = YES;
    return self;
}
// 释放时，标识符自增，确保layer唯一
- (void)dealloc {
    [_sentinel increase];
}
// 重写setNeedsDisplay，立即显示内容，取消异步绘制，调用super在主线程中绘制
- (void)setNeedsDisplay {
    [self _cancelAsyncDisplay];
    [super setNeedsDisplay];
}
// 重写display方法
- (void)display {
    super.contents = super.contents;
    [self _displayAsync:_displaysAsynchronously];
}
#pragma mark - Private
// 绘制任务
- (void)_displayAsync:(BOOL)async {
  /// 获取到layer的delegate，拿到绘制任务
    __strong id<YYAsyncLayerDelegate> delegate = (id)self.delegate;
    YYAsyncLayerDisplayTask *task = [delegate newAsyncDisplayTask];
	  /// 检查是否有绘制任务，如果没有，直接调用绘制完成并返回
    if (!task.display) {
        if (task.willDisplay) task.willDisplay(self);
        self.contents = nil;
        if (task.didDisplay) task.didDisplay(self, YES);
        return;
    }
    /// 有绘制任务
    if (async) {
	      /// 将要绘制回调
        if (task.willDisplay) task.willDisplay(self);
      	/// 初始化判断任务是否已经取消的回调
        YYSentinel *sentinel = _sentinel;
        int32_t value = sentinel.value;
        BOOL (^isCancelled)(void) = ^BOOL() {
            return value != sentinel.value;
        };
        CGSize size = self.bounds.size;
        BOOL opaque = self.opaque;
        CGFloat scale = self.contentsScale;
        CGColorRef backgroundColor = (opaque && self.backgroundColor) ? CGColorRetain(self.backgroundColor) : NULL;
      	/// 如果宽或高小于1则没必要进行内容绘制了
        if (size.width < 1 || size.height < 1) {
            CGImageRef image = (__bridge_retained CGImageRef)(self.contents);
            self.contents = nil;
            if (image) {
              	/// 异步调用释放队列，进行内容释放
                dispatch_async(YYAsyncLayerGetReleaseQueue(), ^{
                    CFRelease(image);
                });
            }
            if (task.didDisplay) task.didDisplay(self, YES);
            CGColorRelease(backgroundColor);
            return;
        }
        /// 在绘制队列中，进行绘制
        dispatch_async(YYAsyncLayerGetDisplayQueue(), ^{
          	/// 首先检查是否已经取消
            if (isCancelled()) {
                CGColorRelease(backgroundColor);
                return;
            }
          	/// 开启上下文
            UIGraphicsBeginImageContextWithOptions(size, opaque, scale);
            CGContextRef context = UIGraphicsGetCurrentContext();
          	/// 如果为不透明layer
            if (opaque) {
                CGContextSaveGState(context); {
                  	/// 如果没有背景颜色，或背景颜色的透明度小于1，绘制白色背景
                    if (!backgroundColor || CGColorGetAlpha(backgroundColor) < 1) {
                        CGContextSetFillColorWithColor(context, [UIColor whiteColor].CGColor);
                        CGContextAddRect(context, CGRectMake(0, 0, size.width * scale, size.height * scale));
                        CGContextFillPath(context);
                    }
                  	/// 如果有背景颜色，直接绘制
                    if (backgroundColor) {
                        CGContextSetFillColorWithColor(context, backgroundColor);
                        CGContextAddRect(context, CGRectMake(0, 0, size.width * scale, size.height * scale));
                        CGContextFillPath(context);
                    }
                } CGContextRestoreGState(context); /// 保存当前状态
                CGColorRelease(backgroundColor);
            }
          	/// 上下文已经准备好，开始进行绘制，调用绘制任务
            task.display(context, size, isCancelled);
          	/// 检查是否已经取消，取消则结束绘制
            if (isCancelled()) {
                UIGraphicsEndImageContext();
                dispatch_async(dispatch_get_main_queue(), ^{
                    if (task.didDisplay) task.didDisplay(self, NO);
                });
                return;
            }
          	/// 从上下文中取出已绘制完成的图片
            UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
          	/// 结束上线文
            UIGraphicsEndImageContext();
          	/// 如果任务已经取消
            if (isCancelled()) {
                dispatch_async(dispatch_get_main_queue(), ^{
                    if (task.didDisplay) task.didDisplay(self, NO);
                });
                return;
            }
            dispatch_async(dispatch_get_main_queue(), ^{
                if (isCancelled()) {
                    if (task.didDisplay) task.didDisplay(self, NO);
                } else {
                  	/// 在主线程中更新UI
                    self.contents = (__bridge id)(image.CGImage);
                    if (task.didDisplay) task.didDisplay(self, YES);
                }
            });
        });
    } else {
      	/// 非异步绘制任务，在主线程中的绘制任务，绘制中不会取消
        [_sentinel increase];
        if (task.willDisplay) task.willDisplay(self);
        UIGraphicsBeginImageContextWithOptions(self.bounds.size, self.opaque, self.contentsScale);
        CGContextRef context = UIGraphicsGetCurrentContext();
        if (self.opaque) {
            CGSize size = self.bounds.size;
            size.width *= self.contentsScale;
            size.height *= self.contentsScale;
            CGContextSaveGState(context); {
                if (!self.backgroundColor || CGColorGetAlpha(self.backgroundColor) < 1) {
                    CGContextSetFillColorWithColor(context, [UIColor whiteColor].CGColor);
                    CGContextAddRect(context, CGRectMake(0, 0, size.width, size.height));
                    CGContextFillPath(context);
                }
                if (self.backgroundColor) {
                    CGContextSetFillColorWithColor(context, self.backgroundColor);
                    CGContextAddRect(context, CGRectMake(0, 0, size.width, size.height));
                    CGContextFillPath(context);
                }
            } CGContextRestoreGState(context);
        }
      	/// 绘制过程中不能取消
        task.display(context, self.bounds.size, ^{return NO;});
        UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
        UIGraphicsEndImageContext();
        self.contents = (__bridge id)(image.CGImage);
        if (task.didDisplay) task.didDisplay(self, YES);
    }
}
/// 取消绘制
- (void)_cancelAsyncDisplay {
    [_sentinel increase];
}
@end
```

还有获取绘制队列，和获取释放队列的函数

```objective-c
/// 用于绘制内容的全局队列
static dispatch_queue_t YYAsyncLayerGetDisplayQueue() {
/// 如果项目中使用了YYDispatchQueuePool
#ifdef YYDispatchQueuePool_h
    return YYDispatchQueueGetForQOS(NSQualityOfServiceUserInitiated);
#else
#define MAX_QUEUE_COUNT 16
  	/// 队列总数最大为16
    static int queueCount;
    static dispatch_queue_t queues[MAX_QUEUE_COUNT];
    static dispatch_once_t onceToken;
  	/// 自增计数器
    static int32_t counter = 0;
    dispatch_once(&onceToken, ^{
      	/// 获取当前活跃的线程数
        queueCount = (int)[NSProcessInfo processInfo].activeProcessorCount;
      	/// 设置队列总数
        queueCount = queueCount < 1 ? 1 : queueCount > MAX_QUEUE_COUNT ? MAX_QUEUE_COUNT : queueCount;
      	/// 初始化队列
        if ([UIDevice currentDevice].systemVersion.floatValue >= 8.0) {
            for (NSUInteger i = 0; i < queueCount; i++) {
                dispatch_queue_attr_t attr = dispatch_queue_attr_make_with_qos_class(DISPATCH_QUEUE_SERIAL, QOS_CLASS_USER_INITIATED, 0);
                queues[i] = dispatch_queue_create("com.ibireme.yykit.render", attr);
            }
        } else {
            for (NSUInteger i = 0; i < queueCount; i++) {
                queues[i] = dispatch_queue_create("com.ibireme.yykit.render", DISPATCH_QUEUE_SERIAL);
                dispatch_set_target_queue(queues[i], dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0));
            }
        }
    });
  	/// 保证当前始终在队列数中取值
    int32_t cur = OSAtomicIncrement32(&counter);
    if (cur < 0) cur = -cur;
    return queues[(cur) % queueCount];
#undef MAX_QUEUE_COUNT
#endif
}
```

