# Runtime

## 类的结构

```
typedef struct objc_class *Class;

struct objc_class {
    Class isa;
    Class super_class;
    const char *name;
    long version;
    long info;
    long instance_size;
    struct objc_ivar_list *ivars;
    struct objc_method_list **methodLists;
    struct objc_cache *cache;
    struct objc_protocol_list *protocols;
};
```

## 方法缓存 struct objc_cache *cache

类的所有缓存都存在metaclass上，所以每个类都只有一份方法缓存，而不是每一个类的object都保存一份。

从父类中继承的方法，也会存在类本身的方法缓存中。当父类的对象调用那个方法的时候，会在父类的metaclass中缓存一份。

**方法缓存限制**

这个问题翻了下runtime的源码：[runtime-cache](https://github.com/opensource-apple/objc4/blob/master/runtime/objc-cache.mm)
```
static void cache_fill_nolock(Class cls, SEL sel, IMP imp, id receiver){
  //  省略
      if (cache->isConstantEmptyCache()) {
          // Cache is read-only. Replace it.
          cache->reallocate(capacity, capacity ?: INIT_CACHE_SIZE);
      }else if (newOccupied <= capacity / 4 * 3) {
          // Cache is less than 3/4 full. Use it as-is.
      }else {
          // Cache is too full. Expand it.
          cache->expand();            //  1
      }
  //插入数据省略
}

void cache_t::expand(){
    uint32_t oldCapacity = capacity();
    uint32_t newCapacity = oldCapacity ? oldCapacity*2 : INIT_CACHE_SIZE;   //  2
    if ((uint32_t)(mask_t)newCapacity != newCapacity) {     //  3
        // mask overflow - can't grow further
        // fixme this wastes one bit of mask
        newCapacity = oldCapacity;
    }
    reallocate(oldCapacity, newCapacity);
}

/* Initial cache bucket count. INIT_CACHE_SIZE must be a power of two. */
enum {
    INIT_CACHE_SIZE_LOG2 = 2,
    INIT_CACHE_SIZE      = (1 << INIT_CACHE_SIZE_LOG2)
};

#if __LP64__
typedef uint32_t mask_t;  // x86_64 & arm64 asm are less efficient with 16-bits
#else
typedef uint16_t mask_t;
#endif

```
由源码可以看出：
1. 当缓存已满时，调用expand()方法进行扩容。
2. 计算新的缓存个数，INIT_CACHE_SIZE由下面代码可以看出是一个枚举值为4。
3. 计算完新的缓存大小，进行了两次强转之后判断是否与原值相等。-强转为mask-t类型，在64位下uint32_t类型。-强转为uint32_t类型。**结论：缓存的最大限制为2^mask_t**


## SEL与IMP，Method等
**SEL：**是用字符串表示的某个对象的方法（虚拟表中指向某个函数指针的字符串）

**IMP：**表示的是指向函数实现的指针。

**Method：**是SEL+IMP+类型
![method](../images/method.png)

**Ivar:**实例变量

**property:**实例变量+setter+getter
