## weak 

weak 是弱引用（不持有），与 strong 相对。

### 内部实现

Runtime 维护着一个全局的哈希表，即由自旋锁控制的弱引用表（weak_table_t），以对象地址为key，以指向对象的弱引用的散列集合为value。

```
// 全局弱引用表
struct weak_table_t {
    weak_entry_t *weak_entries; // 所有的弱引用记录
};
// 弱引用结构体
struct weak_entry_t {
    DisguisedPtr<objc_object> referent; // 对象
    struct {
        weak_referrer_t *referrers; // 所有指向对象的指针地址
    };
};
// 指向弱引用对象的指针的地址
typedef objc_object ** weak_referrer_t;
```

当对象被释放调用 dealloc 方法后
1. 通过对象地址获取全局 weak 哈希表中的记录。
2. for循环遍历 weak 指针数组中的指针并置为nil，并从哈希表中移除。

调用路径：
```
// NSObject.mm
- (void)dealloc {
    _objc_rootDealloc(self);
}
void _objc_rootDealloc(id obj){
    obj->rootDealloc();
}

// objc-object.h
inline void objc_object::rootDealloc(){
    object_dispose((id)this);
}

// objc-runtime-new.mm
id object_dispose(id obj){
    objc_destructInstance(obj);
}
void *objc_destructInstance(id obj) {
    if (assoc) _object_remove_assocations(obj); // 删除关联对象
    if (dealloc) obj->clearDeallocating(); // 回收
}

// objc-object.h
inline void objc_object::clearDeallocating(){
    clearDeallocating_slow();
}

// NSObject.mm
NEVER_INLINE void objc_object::clearDeallocating_slow(){
    if (isa.weakly_referenced) { // 包含弱引用
        weak_clear_no_lock(&table.weak_table, (id)this); 
    }
}

// objc-weak.mm
void weak_clear_no_lock(weak_table_t *weak_table, id referent_id){
    // for 循环遍历设置指针为nil后，删除。
}
```

> OC 中向 nil 发送方法是安全的。

### 相关问题

1. 使用weak的情景

    * 在 ARC 下，遇到循环引用问题是，使一方使用 weak 修饰。例如：delegate。
    * 本身已经有了强引用时，可以使用weak。例如：xib或者storyboard中的控件属性一般使用weak。
2. weak 与 assign

    weak 弱持有对象，对象销毁时，属性值也跟着销毁，必须用于 OC 对象。assign 只进行简单赋值操作，一般用于基础类型数据。
    