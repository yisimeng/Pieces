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

/// 链表
@implementation _YYLinkedMap{
    @package
    CFMutableDictionaryRef _dic; //  存放数据的字典
    NSUInteger _totalCost; // 当前总共占用的空间
    NSUInteger _totalCount; // 缓存中的总个数
    _YYLinkedMapNode *_head; // MRU, do not change it directly
    _YYLinkedMapNode *_tail; // LRU, do not change it directly
    BOOL _releaseOnMainThread; // 在主线程中释放缓存
    BOOL _releaseAsynchronously; // 异步在子线程中释放缓存
}

- (instancetype)init {
    self = [super init];
    _dic = CFDictionaryCreateMutable(CFAllocatorGetDefault(), 0, &kCFTypeDictionaryKeyCallBacks, &kCFTypeDictionaryValueCallBacks);
  // 默认子线程中释放缓存
    _releaseOnMainThread = NO;
    _releaseAsynchronously = YES;
    return self;
}

- (void)dealloc {
    CFRelease(_dic);
}
// 插入数据
- (void)insertNodeAtHead:(_YYLinkedMapNode *)node {
    CFDictionarySetValue(_dic, (__bridge const void *)(node->_key), (__bridge const void *)(node));
  	// 调整数据总数与消耗
    _totalCost += node->_cost;
    _totalCount++;
    if (_head) {
      // 调整MRU指针
        node->_next = _head;
        _head->_prev = node;
        _head = node;
    } else {
        // 无数据，两个指针都指向node
        _head = _tail = node;
    }
}
// 移动节点到头部
- (void)bringNodeToHead:(_YYLinkedMapNode *)node {
  	// 当前已经是第一个，不做调整
    if (_head == node) return;
    // 如果node是最后一个
    if (_tail == node) {
      	// 最后的指针前移,并设置向后指针为nil
        _tail = node->_prev;
        _tail->_next = nil;
    } else {
       // 将后节点的向前指针指向前一个节点
        node->_next->_prev = node->_prev;
      // 将前一个节点的next指向当前后面的节点
        node->_prev->_next = node->_next;
    }
  // 将node放到head
    node->_next = _head;
    node->_prev = nil;
    _head->_prev = node;
    _head = node;
}
// 移除指定节点
- (void)removeNode:(_YYLinkedMapNode *)node {
    CFDictionaryRemoveValue(_dic, (__bridge const void *)(node->_key));
    _totalCost -= node->_cost;
    _totalCount--;
    if (node->_next) node->_next->_prev = node->_prev;
    if (node->_prev) node->_prev->_next = node->_next;
    if (_head == node) _head = node->_next;
    if (_tail == node) _tail = node->_prev;
}
// 移除末尾节点
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
// 清空
- (void)removeAll {
    _totalCost = 0;
    _totalCount = 0;
    _head = nil;
    _tail = nil;
    if (CFDictionaryGetCount(_dic) > 0) {
      	// 取出原字典，准备释放
        CFMutableDictionaryRef holder = _dic;
      	// 重新创建一个字典
        _dic = CFDictionaryCreateMutable(CFAllocatorGetDefault(), 0, &kCFTypeDictionaryKeyCallBacks, &kCFTypeDictionaryValueCallBacks);
        
      // 是否为异步释放
        if (_releaseAsynchronously) {
          // 异步在主线程或子线程中释放元素
            dispatch_queue_t queue = _releaseOnMainThread ? dispatch_get_main_queue() : YYMemoryCacheGetReleaseQueue();
            dispatch_async(queue, ^{
                CFRelease(holder); // hold and release in specified queue
            });
        } else if (_releaseOnMainThread && !pthread_main_np()) {
          // 如果指定在主线程同步释放缓存，而且当前又是在主线程，这里做了个容错处理，改用异步
            dispatch_async(dispatch_get_main_queue(), ^{
                CFRelease(holder); // hold and release in specified queue
            });
        } else {
          // 直接释放
            CFRelease(holder);
        }
    }
}

@end
```

#### YYMemoryCache

```objective-c
@implementation YYMemoryCache {
    pthread_mutex_t _lock; // 锁
    _YYLinkedMap *_lru; // 数据链表
    dispatch_queue_t _queue; // 一个串行队列
}
// 递归修剪，默认每5秒进行自我判断一遍，并在后台线程进行修剪
- (void)_trimRecursively {
    __weak typeof(self) _self = self;
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(_autoTrimInterval * NSEC_PER_SEC)), dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0), ^{
        __strong typeof(_self) self = _self;
        if (!self) return;
        [self _trimInBackground];
        [self _trimRecursively];
    });
}
// 异步修剪
- (void)_trimInBackground {
    dispatch_async(_queue, ^{
      // 减到合适的内存消耗
        [self _trimToCost:self->_costLimit];
      // 减到个数
        [self _trimToCount:self->_countLimit];
      // 去除超时失效文件
        [self _trimToAge:self->_ageLimit];
    });
}

- (void)_trimToCost:(NSUInteger)costLimit {
    BOOL finish = NO;
  // 加锁
    pthread_mutex_lock(&_lock);
    if (costLimit == 0) {
      // 如果总内存为0，直接清空全部
        [_lru removeAll];
        finish = YES;
    } else if (_lru->_totalCost <= costLimit) {
        finish = YES;
    }
  // 解锁
    pthread_mutex_unlock(&_lock);
    if (finish) return;
    // 生成临时数组，保存将要删除的节点
    NSMutableArray *holder = [NSMutableArray new];
    while (!finish) {
      // 尝试加锁
        if (pthread_mutex_trylock(&_lock) == 0) {
          // 如果现存数据量大于限制
            if (_lru->_totalCost > costLimit) {
              // 把将要删除的元素取出，放到将要删除的数组中
                _YYLinkedMapNode *node = [_lru removeTailNode];
                if (node) [holder addObject:node];
            } else {
              // 修剪到限制范围内，跳出循环
                finish = YES;
            }
            pthread_mutex_unlock(&_lock);
        } else {
          // 未能获取到锁，停顿10ms后重新尝试获取锁
            usleep(10 * 1000); //10 ms
        }
    }
  // 临时数组超出作用域后就会释放，内部对象没有其他引用，也将被释放
    if (holder.count) {
        dispatch_queue_t queue = _lru->_releaseOnMainThread ? dispatch_get_main_queue() : YYMemoryCacheGetReleaseQueue();
        dispatch_async(queue, ^{
          ///????? 这是什么作用，只是获取一下count？
            [holder count]; // release in queue
        });
    }
}
// 按内存个数修剪，具体思路与按内存相同
- (void)_trimToCount:(NSUInteger)countLimit {
    BOOL finish = NO;
    pthread_mutex_lock(&_lock);
  // 如果限制为0，直接移除所有
    if (countLimit == 0) {
        [_lru removeAll];
        finish = YES;
    } else if (_lru->_totalCount <= countLimit) {
      // 个数未达到限制
        finish = YES;
    }
    pthread_mutex_unlock(&_lock);
    if (finish) return;
    
    NSMutableArray *holder = [NSMutableArray new];
    while (!finish) {
        if (pthread_mutex_trylock(&_lock) == 0) {
            if (_lru->_totalCount > countLimit) {
                _YYLinkedMapNode *node = [_lru removeTailNode];
                if (node) [holder addObject:node];
            } else {
                finish = YES;
            }
            pthread_mutex_unlock(&_lock);
        } else {
            usleep(10 * 1000); //10 ms
        }
    }
    if (holder.count) {
        dispatch_queue_t queue = _lru->_releaseOnMainThread ? dispatch_get_main_queue() : YYMemoryCacheGetReleaseQueue();
        dispatch_async(queue, ^{
            [holder count]; // release in queue
        });
    }
}
// 按时间修剪
- (void)_trimToAge:(NSTimeInterval)ageLimit {
    BOOL finish = NO;
  // 获取绝对时间，CACurrentMediaTime不受系统时间影响，为从开机到现在（不包含休眠）的时间（单位：秒）
    NSTimeInterval now = CACurrentMediaTime();
    pthread_mutex_lock(&_lock);
    if (ageLimit <= 0) {
        [_lru removeAll];
        finish = YES;
    } else if (!_lru->_tail || (now - _lru->_tail->_time) <= ageLimit) {
      // 当前没有数据，或者最后一条数据在限制之内，直接返回
        finish = YES;
    }
    pthread_mutex_unlock(&_lock);
    if (finish) return;
    
    NSMutableArray *holder = [NSMutableArray new];
    while (!finish) {
        if (pthread_mutex_trylock(&_lock) == 0) {
            if (_lru->_tail && (now - _lru->_tail->_time) > ageLimit) {
                _YYLinkedMapNode *node = [_lru removeTailNode];
                if (node) [holder addObject:node];
            } else {
                finish = YES;
            }
            pthread_mutex_unlock(&_lock);
        } else {
            usleep(10 * 1000); //10 ms
        }
    }
    if (holder.count) {
        dispatch_queue_t queue = _lru->_releaseOnMainThread ? dispatch_get_main_queue() : YYMemoryCacheGetReleaseQueue();
        dispatch_async(queue, ^{
            [holder count]; // release in queue
        });
    }
}
// 监听到内存警告，将自身回调出去，可由开发者手动调用释放内存。默认为无回调，自动移除所有缓存对象
- (void)_appDidReceiveMemoryWarningNotification {
    if (self.didReceiveMemoryWarningBlock) {
        self.didReceiveMemoryWarningBlock(self);
    }
    if (self.shouldRemoveAllObjectsOnMemoryWarning) {
        [self removeAllObjects];
    }
}
// 进入后台，将自身回调出去。默认进入后台自动释放所有缓存对象
- (void)_appDidEnterBackgroundNotification {
    if (self.didEnterBackgroundBlock) {
        self.didEnterBackgroundBlock(self);
    }
    if (self.shouldRemoveAllObjectsWhenEnteringBackground) {
        [self removeAllObjects];
    }
}

#pragma mark - public

- (instancetype)init {
    self = super.init;
    pthread_mutex_init(&_lock, NULL);
    _lru = [_YYLinkedMap new];
    _queue = dispatch_queue_create("com.ibireme.cache.memory", DISPATCH_QUEUE_SERIAL);
    // 默认缓存大小时间为无限大
    _countLimit = NSUIntegerMax;
    _costLimit = NSUIntegerMax;
    _ageLimit = DBL_MAX;
    _autoTrimInterval = 5.0;
    _shouldRemoveAllObjectsOnMemoryWarning = YES;
    _shouldRemoveAllObjectsWhenEnteringBackground = YES;
    // 监听内存警告和进入后台
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(_appDidReceiveMemoryWarningNotification) name:UIApplicationDidReceiveMemoryWarningNotification object:nil];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(_appDidEnterBackgroundNotification) name:UIApplicationDidEnterBackgroundNotification object:nil];
    
    [self _trimRecursively];
    return self;
}

- (void)dealloc {
    [[NSNotificationCenter defaultCenter] removeObserver:self name:UIApplicationDidReceiveMemoryWarningNotification object:nil];
    [[NSNotificationCenter defaultCenter] removeObserver:self name:UIApplicationDidEnterBackgroundNotification object:nil];
    [_lru removeAll];
    pthread_mutex_destroy(&_lock);
}
// 当前缓存总个数
- (NSUInteger)totalCount {
    pthread_mutex_lock(&_lock);
    NSUInteger count = _lru->_totalCount;
    pthread_mutex_unlock(&_lock);
    return count;
}
// 总内存消耗
- (NSUInteger)totalCost {
    pthread_mutex_lock(&_lock);
    NSUInteger totalCost = _lru->_totalCost;
    pthread_mutex_unlock(&_lock);
    return totalCost;
}
// 是否是在主线程释放缓存，默认为NO
- (BOOL)releaseOnMainThread {
    pthread_mutex_lock(&_lock);
    BOOL releaseOnMainThread = _lru->_releaseOnMainThread;
    pthread_mutex_unlock(&_lock);
    return releaseOnMainThread;
}
// 设置在主线程释放缓存
- (void)setReleaseOnMainThread:(BOOL)releaseOnMainThread {
    pthread_mutex_lock(&_lock);
    _lru->_releaseOnMainThread = releaseOnMainThread;
    pthread_mutex_unlock(&_lock);
}
// 是否为异步释放
- (BOOL)releaseAsynchronously {
    pthread_mutex_lock(&_lock);
    BOOL releaseAsynchronously = _lru->_releaseAsynchronously;
    pthread_mutex_unlock(&_lock);
    return releaseAsynchronously;
}
// 设置异步释放
- (void)setReleaseAsynchronously:(BOOL)releaseAsynchronously {
    pthread_mutex_lock(&_lock);
    _lru->_releaseAsynchronously = releaseAsynchronously;
    pthread_mutex_unlock(&_lock);
}
// 是否包含对象，通过CFDictionaryContainsKey判断
- (BOOL)containsObjectForKey:(id)key {
    if (!key) return NO;
    pthread_mutex_lock(&_lock);
    BOOL contains = CFDictionaryContainsKey(_lru->_dic, (__bridge const void *)(key));
    pthread_mutex_unlock(&_lock);
    return contains;
}
// 通过key获取缓存对象
- (id)objectForKey:(id)key {
    if (!key) return nil;
    pthread_mutex_lock(&_lock);
    _YYLinkedMapNode *node = CFDictionaryGetValue(_lru->_dic, (__bridge const void *)(key));
    if (node) {
      // 如果node存在，修改使用时间，并将其放到头部
        node->_time = CACurrentMediaTime();
        [_lru bringNodeToHead:node];
    }
    pthread_mutex_unlock(&_lock);
    return node ? node->_value : nil;
}
// 添加缓存对象
- (void)setObject:(id)object forKey:(id)key {
    [self setObject:object forKey:key withCost:0];
}
// 添加缓存对象
- (void)setObject:(id)object forKey:(id)key withCost:(NSUInteger)cost {
  // key不存在，直接返回
    if (!key) return;
  // 如果对象为空，认为是要删除缓存，则从缓存中移除对应的条目
    if (!object) {
        [self removeObjectForKey:key];
        return;
    }
    pthread_mutex_lock(&_lock);
  // 通过key 获取对应缓存对象
    _YYLinkedMapNode *node = CFDictionaryGetValue(_lru->_dic, (__bridge const void *)(key));
    NSTimeInterval now = CACurrentMediaTime();
    if (node) {
      // 如果对象已在缓存中，则更新总内存消耗
        _lru->_totalCost -= node->_cost;
        _lru->_totalCost += cost;
      // 重新设置node的信息
        node->_cost = cost;
        node->_time = now;
        node->_value = object;
      // 将node放到头
        [_lru bringNodeToHead:node];
    } else {
      // 缓存中没有对象，插入对象
        node = [_YYLinkedMapNode new];
        node->_cost = cost;
        node->_time = now;
        node->_key = key;
        node->_value = object;
        [_lru insertNodeAtHead:node];
    }
  // 检测是否超出内存消耗限制、数量限制
    if (_lru->_totalCost > _costLimit) {
        dispatch_async(_queue, ^{
            [self trimToCost:_costLimit];
        });
    }
    if (_lru->_totalCount > _countLimit) {
        _YYLinkedMapNode *node = [_lru removeTailNode];
        if (_lru->_releaseAsynchronously) {
            dispatch_queue_t queue = _lru->_releaseOnMainThread ? dispatch_get_main_queue() : YYMemoryCacheGetReleaseQueue();
            dispatch_async(queue, ^{
                [node class]; //hold and release in queue
            });
        } else if (_lru->_releaseOnMainThread && !pthread_main_np()) {
            dispatch_async(dispatch_get_main_queue(), ^{
                [node class]; //hold and release in queue
            });
        }
    }
    pthread_mutex_unlock(&_lock);
}
// 释放指定缓存对象
- (void)removeObjectForKey:(id)key {
    if (!key) return;
    pthread_mutex_lock(&_lock);
    _YYLinkedMapNode *node = CFDictionaryGetValue(_lru->_dic, (__bridge const void *)(key));
    if (node) {
        [_lru removeNode:node];
        if (_lru->_releaseAsynchronously) {
            dispatch_queue_t queue = _lru->_releaseOnMainThread ? dispatch_get_main_queue() : YYMemoryCacheGetReleaseQueue();
            dispatch_async(queue, ^{
                [node class]; //hold and release in queue
            });
        } else if (_lru->_releaseOnMainThread && !pthread_main_np()) {
            dispatch_async(dispatch_get_main_queue(), ^{
                [node class]; //hold and release in queue
            });
        }
    }
    pthread_mutex_unlock(&_lock);
}
// 释放所有元素
- (void)removeAllObjects {
    pthread_mutex_lock(&_lock);
    [_lru removeAll];
    pthread_mutex_unlock(&_lock);
}
// 释放至指定个数
- (void)trimToCount:(NSUInteger)count {
    if (count == 0) {
        [self removeAllObjects];
        return;
    }
    [self _trimToCount:count];
}
// 释放至指定内存大小
- (void)trimToCost:(NSUInteger)cost {
    [self _trimToCost:cost];
}
// 释放至指定时间段内
- (void)trimToAge:(NSTimeInterval)age {
    [self _trimToAge:age];
}
- (NSString *)description {
    if (_name) return [NSString stringWithFormat:@"<%@: %p> (%@)", self.class, self, _name];
    else return [NSString stringWithFormat:@"<%@: %p>", self.class, self];
}

@end
```



