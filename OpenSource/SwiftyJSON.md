# SwiftyJSON

[GitHub](https://github.com/SwiftyJSON/SwiftyJSON)

保持JSON的语义，安全的从JSON字符串中通过字面量取值。

HandyJSON 将JSON转化为Model，操作Model；SwiftyJSON则是保持JSON数据格式，更安全的解析JSON。

swiftyJSON定义了结构体： JSON。

```
// 存放JSON字符串
public var object: Any { get set }

// 具体JSON字符串对应的类型
fileprivate var rawArray: [Any] = []
fileprivate var rawDictionary: [String : Any] = [:]
fileprivate var rawString: String = ""
fileprivate var rawNumber: NSNumber = 0
fileprivate var rawNull: NSNull = NSNull()
fileprivate var rawBool: Bool = false
```

初始化时将 JSON字符串 赋值给 object ，在 object 中的 set 方法判断类型，并将 JSON 赋值给该属性，递归解析JSON字符串。

重定义JSON的 **subscript**， 可以通过下标取值，并且取值依然为JSON。为JSON添加Array、dic、bool等字面量协议，使JSON在为任何类型是都可以通过字面量取值。

为各个类型添加可选类型，并重写get set方法，设置取值时进行判断当前类型。

```
 //Optional string
    public var string: String? {
        get {
            switch self.type {
            case .string:
                return self.object as? String
            default:
                return nil
            }
        }
        set {
            if let newValue = newValue {
                self.object = NSString(string:newValue)
            } else {
                self.object = NSNull()
            }
        }
    }

```