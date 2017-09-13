# Tips

**元组（Tuple）**的使用使用场景：方法有多个不同类型的返回值时，使用元组。

**nil：**只能复制给optional类型

**Extension：** 在extension中给系统的类扩充构造函数，只能扩充便利构造函数（关键字：convenience），参数有默认参数时，会自动生成两个函数（一个是完整参数函数，一个是使用的默认参数函数）。

**self.不能省略的情况**
> * 在方法中和其他的标识符有歧义
> * 闭包中的‘self.’
> * 在属性懒加载中的’self.’

**required：**使用required关键字修饰初始化方法，表示指定的初始化方法，不能再使用‘（）’初始化

**protocol extension：** 可以where语法可以指定代理类型，不能当做delegate传值使用。

**@discardableResult：**在swift3.0之后，如果方法有返回值，但是没有用到的时候会报警告，可以使用关键字修饰去掉警告



## Array

### Map

对数组中的每个元素执行转换操作，得到一个新数组。

```
// 可能的一种实现方式
extension Array {
    func map<T>(_ transform: (Element) -> T) -> [T] {
    	 // 初始化数组
        var result: [T] = []
        result.reserveCapacity(count)
        // 遍历每个元素执行转换并添加进新数组
        for x in self {
            result.append(transform(x))
        }
        return result
    }
}
```

### flatMap

类似于 Map，拼接结果时，使用的时```append(contentsOf:<#T##Sequence#>)```，添加的是一个序列，会对结果进行展开。

```
extension Array {
    func flatMap<T>(_ transform: (Element) -> [T]) -> [T] {
        var result: [T] = []
        for x in self {
            result.append(contentsOf: transform(x))
        }
        return result
    }
}
```

### filter

过滤符合条件的元素，组成一个新的数组。

```
extension Array {
    func filter(_ isIncluded: (Element) -> Bool) -> [Element] {
        var result: [Element] = []
        for x in self where isIncluded(x) {
            result.append(x)
        }
        return result
    }
}
```

### reduce

将元素整合为新的值。

```
extension Array {
    func reduce<Result>(_ initialResult: Result, 
        _ nextPartialResult: (Result, Element) -> Result) -> Result
    {
        var result = initialResult
        for x in self {
            result = nextPartialResult(result, x)
        }
        return result
    }
}
```

### forEach

迭代，与for类似。不过内部return 不能退出循环。


## Dictionary

### 一些字典的扩展

* 合并字典： 将一个字典并入另一个中。

```
extension Dictionary {
    mutating func merge<S>(_ other: S)
        where S: Sequence, S.Iterator.Element == (key: Key, value: Value) {
        for (k, v) in other {
            self[k] = v
        }
    }
}
```
> **mutating**：内部将会修改外部的值类型属性，用于 enum、struct，protocol。class 是引用类型，所以不需要。

* 由键值对来初始化字典

```
extension Dictionary {
    init<S: Sequence>(_ sequence: S)
        where S.Iterator.Element == (key: Key, value: Value) {
        self = [:]
        // 使用上步的合并
        self.merge(sequence)
    }
}
```

* 修改字典中的值：字典是```Sequence```类型，map函数是返回一个数组。这里的map 是用来修改字典中的值。

```
extension Dictionary {
    func mapValues<NewValue>(transform: (Value) -> NewValue)
        -> [Key:NewValue] {
        return Dictionary<Key, NewValue>(map { (key, value) in
            return (key, transform(value))
        })
    }
}

```