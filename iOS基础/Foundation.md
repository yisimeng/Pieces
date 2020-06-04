# Foundation

## NSCache

缓存，存储键值对，当内存不够时，自动释放内容。

* NSCache 包含很多自动释放策略，当内存不足时，会根据策略进行释放。
* 是线程安全的，不需要加锁。
* 不像NSMutableDictionary等，NSCache不需要copy对象

通常使用NSCache对象临时存储创建成本大的对象，减少重复创建。当内存不足时，内部对象可能被释放，所以不要存放重要数据。


## JSONSerialization

json 序列化与反序列化的类，比较常用。

###  data->str/dic/array时的参数 options：

```
@available(iOS 5.0, *)
    public struct ReadingOptions : OptionSet {
        public init(rawValue: UInt)
        public static var mutableContainers: JSONSerialization.ReadingOptions { get }
        public static var mutableLeaves: JSONSerialization.ReadingOptions { get }
        public static var allowFragments: JSONSerialization.ReadingOptions { get }
    }
```
* mutableContainers：创建一个可变的字典或者数组接收结果
* mutableLeaves：创建一个可变的字符串接收结果
* allowFragments：允许不是NSDictionary或者NSArray的其他顶级实例对象

###  json->data时的参数 options介绍：

```
 @available(iOS 5.0, *)
    public struct WritingOptions : OptionSet {
        public init(rawValue: UInt)
        /* 序列化时添加空格，使输出时更可读，如果不设置此项，默认采用紧凑的json
        */
        public static var prettyPrinted: JSONSerialization.WritingOptions { get }
        /* 使用[NSLocale systemLocale]排序输出dic的key。 使用NSNumericSearch比较密钥。 具体排序方法可能会更改。
         */
        @available(iOS 11.0, *)
        public static var sortedKeys: JSONSerialization.WritingOptions { get }
    }
```