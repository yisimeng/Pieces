## Dictionary

字典，其中的Key都是唯一的。

1. 如果键值对中包含重复Key，可以选择保留前者还是后者。

```swift
///     let pairsWithDuplicateKeys = [("a", 1), ("b", 2), ("a", 3), ("b", 4)]
///
///     let firstValues = Dictionary(pairsWithDuplicateKeys,
///                                  uniquingKeysWith: { (first, _) in first })
///     // ["b": 2, "a": 1]
///
///     let lastValues = Dictionary(pairsWithDuplicateKeys,
///                                 uniquingKeysWith: { (_, last) in last })
///     // ["b": 4, "a": 3]
@inlinable public init<S>(_ keysAndValues: S, uniquingKeysWith combine: (Value, Value) throws -> Value) rethrows where S : Sequence, S.Element == (Key, Value)
```

