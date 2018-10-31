# [HandyJSON](https://www.jianshu.com/p/eac4a92b44ef)
HandyJSON是一个用于Swift语言中的JSON序列化/反序列化库。

HandyJSON 将JSON转化为Model，操作Model；SwiftyJSON则是保持JSON数据格式，更安全的解析JSON。

与其他流行的Swift JSON库相比，HandyJSON的特点是，它支持纯swift类，使用也简单。它反序列化时(把JSON转换为Model)不要求Model从NSObject继承(因为它不是基于KVC机制)，也不要求你为Model定义一个Mapping函数。只要你定义好Model类，声明它服从HandyJSON协议，HandyJSON就能自行以各个属性的属性名为Key，从JSON串中解析值。

### Dictionary->Model原理:
1、Model类遵循HandyJSON协议，协议中init（）方法。    
2、Model.self，获取model的type，type.init()，创建变量用于存储model。    
3、通过上一步创建的model，创建model的mirror，根据mirror的显示类型区分model是class还是struct。    
4、根据model和strcut的不同结构，定位到model在存储器中的头部，然后在mirror中遍历children，读取属性，通过type(of: child.value)，获取属性的类型。   
5、通过属性名，去字典中匹配获取value，然后重新绑定指针指向的内存区域。
6、赋值完之后偏移指针。    
