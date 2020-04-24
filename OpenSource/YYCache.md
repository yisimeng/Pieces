# YYCache

线程安全的缓存管理库，包含四个文件：

* YYCache：统一管理缓存（内存缓存和磁盘缓存）
* YYMemoryCache：内存缓存
* YYDiskCache：磁盘缓存
* YYKVSotrage：

## YYCache

是一个缓存管理类。有三个属性：缓存名称，内存缓存对象，磁盘缓存对象。

```objective-c
/** 缓存名称, readonly. */
@property (copy, readonly) NSString *name;
/** 内部的内存缓存 readonly.*/
@property (strong, readonly) YYMemoryCache *memoryCache;
/** 内部的磁盘缓存 readonly.*/
@property (strong, readonly) YYDiskCache *diskCache;
```

缓存的初始化方法：缓存初始化之后，不应该再手动去修改该文件目录

```objective-c
/** 通过名称在默认的路径下创建缓存目录。默认为Caches目录下 */
- (nullable instancetype)initWithName:(NSString *)name;
/** 在指定目录下创建缓存目录 */
- (nullable instancetype)initWithPath:(NSString *)path NS_DESIGNATED_INITIALIZER;
/** 类方法初始化 */
+ (nullable instancetype)cacheWithName:(NSString *)name;
/** 类方法初始化*/
+ (nullable instancetype)cacheWithPath:(NSString *)path;
```

缓存的使用方法：

```objective-c
/** 是否已存在缓存。 同步方法会阻塞线程 */
- (BOOL)containsObjectForKey:(NSString *)key;
/** 是否存在缓存。 异步方法，通过子线程回调结果 */
- (void)containsObjectForKey:(NSString *)key withBlock:(nullable void(^)(NSString *key, BOOL contains))block;
/** 取出缓存对象。 同步方法 */
- (nullable id<NSCoding>)objectForKey:(NSString *)key;
/** 取出缓存对象。 异步方法，子线程中回调结果 */
- (void)objectForKey:(NSString *)key withBlock:(nullable void(^)(NSString *key, id<NSCoding> object))block;
/** 添加对象到缓存。 同步方法阻塞线程 */
- (void)setObject:(nullable id<NSCoding>)object forKey:(NSString *)key;
/** 异步添加对象到缓存 */
- (void)setObject:(nullable id<NSCoding>)object forKey:(NSString *)key withBlock:(nullable void(^)(void))block;
/** 同步移除缓存对象 */
- (void)removeObjectForKey:(NSString *)key;
/** 异步移除缓存对象 */
- (void)removeObjectForKey:(NSString *)key withBlock:(nullable void(^)(NSString *key))block;
/** 同步清除缓存 */
- (void)removeAllObjects;
/** 异步清楚缓存 */
- (void)removeAllObjectsWithBlock:(void(^)(void))block;
/** 异步清楚缓存，并回调移除磁盘缓存进度 */
- (void)removeAllObjectsWithProgressBlock:(nullable void(^)(int removedCount, int totalCount))progress
                                 endBlock:(nullable void(^)(BOOL error))end;
```

## YYMemoryCache

内存缓存。内部采用LRU淘汰算法，默认每5秒进行数据优化。

### LRU MRU

使用CFMutableDictionaryRef进行存储数据，方便查询时快速命中（O(1)时间复杂度）。

通过双向链表链接节点，方便进行交换，将最新使用的放到表头，移除表尾节点等。

```objective-c
/// 双向链表节点
@interface _YYLinkedMapNode : NSObject {
    @package
    __unsafe_unretained _YYLinkedMapNode *_prev; // retained by dic
    __unsafe_unretained _YYLinkedMapNode *_next; // retained by dic
    id _key;
    id _value;
    NSUInteger _cost; // 当前对象所耗费内存大小
    NSTimeInterval _time; // 添加的时间
}
@end

@implementation _YYLinkedMap{
    @package
    CFMutableDictionaryRef _dic; //  do not set object directly
    NSUInteger _totalCost;
    NSUInteger _totalCount;
    _YYLinkedMapNode *_head; // MRU, do not change it directly
    _YYLinkedMapNode *_tail; // LRU, do not change it directly
    BOOL _releaseOnMainThread;
    BOOL _releaseAsynchronously;
}

- (instancetype)init {
    self = [super init];
    _dic = CFDictionaryCreateMutable(CFAllocatorGetDefault(), 0, &kCFTypeDictionaryKeyCallBacks, &kCFTypeDictionaryValueCallBacks);
    _releaseOnMainThread = NO;
    _releaseAsynchronously = YES;
    return self;
}

- (void)dealloc {
    CFRelease(_dic);
}

- (void)insertNodeAtHead:(_YYLinkedMapNode *)node {
    CFDictionarySetValue(_dic, (__bridge const void *)(node->_key), (__bridge const void *)(node));
    _totalCost += node->_cost;
    _totalCount++;
    if (_head) {
        node->_next = _head;
        _head->_prev = node;
        _head = node;
    } else {
        _head = _tail = node;
    }
}

- (void)bringNodeToHead:(_YYLinkedMapNode *)node {
    if (_head == node) return;
    
    if (_tail == node) {
        _tail = node->_prev;
        _tail->_next = nil;
    } else {
        node->_next->_prev = node->_prev;
        node->_prev->_next = node->_next;
    }
    node->_next = _head;
    node->_prev = nil;
    _head->_prev = node;
    _head = node;
}

- (void)removeNode:(_YYLinkedMapNode *)node {
    CFDictionaryRemoveValue(_dic, (__bridge const void *)(node->_key));
    _totalCost -= node->_cost;
    _totalCount--;
    if (node->_next) node->_next->_prev = node->_prev;
    if (node->_prev) node->_prev->_next = node->_next;
    if (_head == node) _head = node->_next;
    if (_tail == node) _tail = node->_prev;
}

- (_YYLinkedMapNode *)removeTailNode {
    if (!_tail) return nil;
    _YYLinkedMapNode *tail = _tail;
    CFDictionaryRemoveValue(_dic, (__bridge const void *)(_tail->_key));
    _totalCost -= _tail->_cost;
    _totalCount--;
    if (_head == _tail) {
        _head = _tail = nil;
    } else {
        _tail = _tail->_prev;
        _tail->_next = nil;
    }
    return tail;
}

- (void)removeAll {
    _totalCost = 0;
    _totalCount = 0;
    _head = nil;
    _tail = nil;
    if (CFDictionaryGetCount(_dic) > 0) {
        CFMutableDictionaryRef holder = _dic;
        _dic = CFDictionaryCreateMutable(CFAllocatorGetDefault(), 0, &kCFTypeDictionaryKeyCallBacks, &kCFTypeDictionaryValueCallBacks);
        
        if (_releaseAsynchronously) {
            dispatch_queue_t queue = _releaseOnMainThread ? dispatch_get_main_queue() : YYMemoryCacheGetReleaseQueue();
            dispatch_async(queue, ^{
                CFRelease(holder); // hold and release in specified queue
            });
        } else if (_releaseOnMainThread && !pthread_main_np()) {
            dispatch_async(dispatch_get_main_queue(), ^{
                CFRelease(holder); // hold and release in specified queue
            });
        } else {
            CFRelease(holder);
        }
    }
}

@end
```



