# Tips

##### 1. NSNumberFormatter

在计算View的frame时，我们通常设置的是自适应，“sizeToFit”,或者是一些通过计算出来的frame，但是实际效果总是跟我们设置的值有些偏差，在经过测试分析后的发现系统在最后确定frame时，会进行舍入，并总是保留一位小数：当“.”后的值小于0.5时，会入成0.5；当等于0.5，会保持不变；当大于0.5，会入成1.0。

```
CGFloat frameFormatterNumber(CGFloat number){
    NSNumberFormatter * formatter = [[NSNumberFormatter alloc] init];
    //允许1位小数
    formatter.maximumFractionDigits = 1;
    //增量设置为0.5
    formatter.roundingIncrement = @0.5;
    //向上取整
    formatter.roundingMode = kCFNumberFormatterRoundUp;
    return [[formatter stringFromNumber:[NSNumber numberWithFloat:number]] floatValue];
}
```

当我们根据label中的文字,通过```- (CGRect)boundingRectWithSize:(CGSize)size options:(NSStringDrawingOptions)options attributes:(nullable NSDictionary<NSString *, id> *)attributes context:(nullable NSStringDrawingContext *)context```方法计算完之后在转换，获得准确的label的frame。

##### 2. frame和bounds 

frame是相对于父视图，bounds是相对于子视图的。

scrollView和其子类，在滚动的时候，size是不变的，但是origin是会发生改变的，向左滑动，origin.x是增加的；向上滑动origin.y是增加的。当初始状态下origin为（0，0），如果向上滑动的了，然后想在屏幕上原来（0,0）的位置添加一个view的话，这时的y值是大于0的。


##### 3. UISearchBar

UISearchBar貌似默认是没有回收时的动画的，最后加了一个动画，让它取消第一响应。。。。

#####  4、UITabBarController

当重写navigationController的方法```pushViewController(_ viewController: UIViewController, animated: Bool)```时，如果我们用到tabbarController的话，一般会在这里统一设置hidesBottomBarWhenPushed属性为true，当push的时候隐藏掉tabbar。

```
override func pushViewController(_ viewController: UIViewController, animated: Bool) {
        viewController.hidesBottomBarWhenPushed = true
        super.pushViewController(viewController, animated: animated)
}
```

但是这是有问题的,刚启动就找不到tabbar了，设置rootViewController的时候也会走这个方法，导致第进来之后就直接隐藏掉tabbar了```public init(rootViewController: UIViewController) // Convenience method pushes the root view controller without animation.```


##### 5、图片的解压缩
从磁盘中加载一张图片，并将它显示到屏幕上，中间的主要工作流如下：

1. 假设我们使用 +imageWithContentsOfFile: 方法从磁盘中加载一张图片，这个时候的图片并没有解压缩；
2. 然后将生成的 UIImage 赋值给 UIImageView；
3. 接着一个隐式的 CATransaction 捕获到了 UIImageView 图层树的变化；
4. 在主线程的下一个 run loop 到来时，Core Animation 提交了这个隐式的 transaction ，这个过程可能会对图片进行 copy 操作，而受图片是否字节对齐等因素的影响，这个 copy 操作可能会涉及以下部分或全部步骤：

> 1. 分配内存缓冲区用于管理文件 IO 和解压缩操作；
> * 将文件数据从磁盘读到内存中；
> * 将压缩的图片数据解码成未压缩的位图形式，这是一个非常耗时的 CPU 操作；
> * 最后 Core Animation 使用未压缩的位图数据渲染 UIImageView 的图层。

图片的解压缩是一个非常耗时的 CPU 操作，并且它默认是在主线程中执行的。那么当需要加载的图片比较多时，就会对我们应用的响应性造成严重的影响，尤其是在快速滑动的列表上，这个问题会表现得更加突出。

位图就是一个像素数组，数组中的每个像素就代表着图片中的一个点。在将磁盘中的图片渲染到屏幕之前，必须先要得到图片的原始像素数据，才能执行后续的绘制操作，这就是为什么需要对图片解压缩的原因。

**强制解压缩**
图片的解压缩是不可避免的，当未解压缩的图片将要渲染到屏幕时，系统会在主线程对图片进行解压缩，而如果图片已经解压缩了，系统就不会再对图片进行解压缩。为了不让解压缩在主线程执行影响性能，可以在手动在子线程提前进行强制解压缩，强制解压缩的原理就是对图片进行重新绘制，得到一张新的解压缩后的位图。

##### 6、drawRect调用
drawRect调用的前提：

* view第一次被加载到屏幕上
* 顶部的其他视图移动
* view的hidden属性被改变
* 手动调用了setNeedsDisplay()或者setNeedsDisplayInRect()方法

**注意**: drawRect(\_:)方法中的所有绘制都会进入视图的上下文，如果在drawRect(\_:)外部进行绘制，必须创建自己的上下文。
永远不要直接调用drawRect(\_:)方法，如果想更新视图，调用setNeedsDisplay()，它会将view进行标记，当下一次屏幕更新周期触发drawRect(_:)时重绘。

##### 7、Category属性问题
不能再分类中给已有类添加存储属性，可以添加计算属性（UIView的left，right）。分类中的属性只会创建setter和getter方法，不会生成实例变量，所以在setter和getter方法中无法赋值。

* **不能向编译后得到的类中增加实例变量**
* **能向运行时创建的类中添加实例变量**

1. 因为编译后的类已经注册在 runtime 中,类结构体中的 `objc_ivar_list` 实例变量的链表和 `instance_size` 实例变量的内存大小已经确定，同时runtime会调用 `class_setvarlayout` 或 `class_setWeaklvarLayout` 来处理strong weak 引用。所以不能向存在的类中添加实例变量。
2. 运行时创建的类是可以添加实例变量，调用`class_addIvar`函数。但是得在调用 `objc_allocateClassPair` 之后，`objc_registerClassPair` 之前，原因同上.

##### 8、循环引用
1. A强引用B，B强引用A。
2. A有个属性为block，在block中又调用A（A在进入block中时，block会强引用A）。
3. 在viewcontroller中使用timer，在dealloc中释放timer，只要timer活跃，就不会进入到dealloc。

##### 9、深拷贝与浅拷贝
集合对象copy：

* 不可变对象是浅拷贝
* 可变对象是深拷贝

集合对象mutableCopy：深拷贝

非集合对象copy：

* 不可变对象是浅拷贝
* 可变对象是深拷贝

非集合对象mutableCopy:深拷贝

* [immutableObject copy] 浅拷贝
* [mutableObject copy] 深拷贝
* [object mutableCopy] 深拷贝

##### 10、视频直播秒开
1. 改写播放器逻辑，让播放器拿到第一个关键帧后就给予显示。
GOP的第一帧通常是关键帧，直播服务器支持GOP缓存，播放器在和服务器建立连接后可以立即拿到关键帧。减少关键帧的距离，是可以改善画质，让播放器快速拿到关键帧，但同时增加了宽带和网络的负载，如果网络不佳，不能快速下载到GOP，会影响体验。
2. 提前做好DNS解析，和做好测速选线（择取最优线路）。
3. 缓存，缓存最近的一个关键帧。

##### 11、图片拉伸的几种方式
1. 通过UIImage的方法```- (UIImage *)resizableImageWithCapInsets:(UIEdgeInsets)capInsets resizingMode:(UIImageResizingMode)resizingMode```来设置图片的可拉伸区域。
2. 如果是通过Assets添加的图片，则可以在Assets.xcassets中需要设置拉伸的图片，点击右下角的**Show Slicing**，在页面中可视化的去拖动拉伸的区域。
3. CALyaer有个contentsCenter的属性，这是一个CGRect，定义了一个固定的边框和一个在图层上可拉伸的区域，值是0.0-1.0。在Interface Builder中为Stretching属性。

<img src="images/5A0B0C3C-045B-446C-8FF8-F0A970B45766.png" width = "250" height = "419" alt="图片名称">

##### 12、+load()与+initialize()方法
**调用时机**

* +load()方法：官方文档介绍:Invoked whenever a class or category is added to the Objective-C runtime;，意思是说当类被加载到runtime的时候就会运行，也就是说是在**main.m之前**,会根据Compile Sources中的顺序来加载，但还有一个需注意的加载顺序：父类 > 子类 > 分类。
* +initialize()方法：官方文档上介绍:Initializes the class before it receives its first message.意思是在类接收第一条消息之前初始化类。值得注意的点是：类初始化的时候每个类只会调用一次+initialize()，如果子类没有实现+initialize()，那么将会调用父类的+initialize()，也就是意味着父类的+initialize()可能会被多次调用。父类 > 子类，分类会覆盖主类。

**使用场景：**

* +load():通常用来进行Method Swizzle，尽量避免过于复杂以及不必要的代码。
* +initialize():一般用于初始化全局变量或静态变量。

> Tips： 为什么 + (void)load 方法都会执行，没有被覆盖。因为系统会自动调用所有的+load，所以内部不需要调用super。
>
> 因为load方法是 runtime 直接调用方法列表中load的方法地址，而不是通过消息机制调用的。
>
> ```c
> static void call_class_loads(void){
> 		...
>     for (i = 0; i < used; i++) {
> 				...
>         if (PrintLoading) {
>             _objc_inform("LOAD: +[%s load]\n", cls->nameForLogging());
>         }
>         (*load_method)(cls, SEL_load);
>     }
> 		...
> }
> ```

##### 13、约束布局优先级

当两个控件并排的时候，如果需要优先满足其中之一，可以通过设置约束的优先级来控制。有两个方法：

“Content Compression Resistance Priority”，也叫内容压缩阻力优先级，该优先级越高，则越晚轮到被压缩。

“Content Hugging Priority”，也叫内容紧靠优先级，该优先级越高，这越晚轮到被拉伸。

##### 14、滚动时回收键盘

当ScrollView滚动时将键盘回收，通过设置UIScrollViewKeyboardDismissMode属性设置。

```
typedef NS_ENUM(NSInteger, UIScrollViewKeyboardDismissMode) {
    UIScrollViewKeyboardDismissModeNone,
    UIScrollViewKeyboardDismissModeOnDrag,      // dismisses the keyboard when a drag begins
    UIScrollViewKeyboardDismissModeInteractive, // the keyboard follows the dragging touch off screen, and may be pulled upward again to cancel the dismiss
} NS_ENUM_AVAILABLE_IOS(7_0);
```

##### 15、设置UIView的透明度，影响subView的透明度

解决方案：设置background color的颜色中的透明度。

##### 16、在工程中查看是否使用 IDFA

打开终端，到工程目录中， 输入：

```grep -r advertisingIdentifier .```

可以看到那些文件中用到了IDFA，如果用到了就会被显示出来。

##### 17、获取手机已安装的应用

[LSApplicationWorkspace](https://github.com/nst/iOS-Runtime-Headers/blob/master/Frameworks/MobileCoreServices.framework/LSApplicationWorkspace.h)

##### 18、数字格式化输出
```
//通过NSNumberFormatter，同样可以设置NSNumber输出的格式。例如如下代码：
NSNumberFormatter *formatter = [[NSNumberFormatter alloc] init];
formatter.numberStyle = NSNumberFormatterDecimalStyle;
NSString *string = [formatter stringFromNumber:[NSNumber numberWithInt:123456789]];
NSLog(@"Formatted number string:%@",string);
//输出结果为：[1223:403] Formatted number string:123,456,789

//其中NSNumberFormatter类有个属性numberStyle，它是一个枚举型，设置不同的值可以输出不同的数字格式。该枚举包括：
typedef NS_ENUM(NSUInteger, NSNumberFormatterStyle) {
    NSNumberFormatterNoStyle = kCFNumberFormatterNoStyle,
    NSNumberFormatterDecimalStyle = kCFNumberFormatterDecimalStyle,
    NSNumberFormatterCurrencyStyle = kCFNumberFormatterCurrencyStyle,
    NSNumberFormatterPercentStyle = kCFNumberFormatterPercentStyle,
    NSNumberFormatterScientificStyle = kCFNumberFormatterScientificStyle,
    NSNumberFormatterSpellOutStyle = kCFNumberFormatterSpellOutStyle
};
//各个枚举对应输出数字格式的效果如下：其中第三项和最后一项的输出会根据系统设置的语言区域的不同而不同。
[1243:403] Formatted number string:123456789
[1243:403] Formatted number string:123,456,789
[1243:403] Formatted number string:￥123,456,789.00
[1243:403] Formatted number string:-539,222,988%
[1243:403] Formatted number string:1.23456789E8
[1243:403] Formatted number string:一亿二千三百四十五万六千七百八十九
```

##### 19、navigationBar根据滑动距离的渐变色实现
```
- (void)scrollViewDidScroll:(UIScrollView *)scrollView{
    CGFloat offsetToShow = 200.0;//滑动多少就完全显示
    CGFloat alpha = 1 - (offsetToShow - scrollView.contentOffset.y) / offsetToShow;
    [[self.navigationController.navigationBar subviews] objectAtIndex:0].alpha = alpha;
}
```

##### 20、NSString进行URL编码和解码
```
NSString *string = @"http://abc.com?aaa=你好&bbb=tttee";

//编码 打印：http://abc.com?aaa=%E4%BD%A0%E5%A5%BD&bbb=tttee
string = [string stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]];

//解码 打印：http://abc.com?aaa=你好&bbb=tttee
string = [string stringByRemovingPercentEncoding];
```

##### 22、swift字面量初始化

swift定义了以下协议，这些协议可以使一种类型通过字面量的方式来初始化并赋值。

```
NilLiteralConvertible
BooleanLiteralConvertible
IntegerLiteralConvertible
FloatLiteralConvertible
UnicodeScalarLiteralConvertible
ExtendedGraphemeClusterLiteralConvertible
StringLiteralConvertible
ArrayLiteralConvertible
DictionaryLiteralConvertible
```

##### 23、iOS系统自带悬浮调试框

在 AppDelegate 的 didFinishLaunchingWithOptions 方法中加入两行代码即可。

```
let overlayClass = NSClassFromString("UIDebuggingInformationOverlay") as? UIWindow.Type
_ = overlayClass?.perform(NSSelectorFromString("prepareDebuggingOverlay"))
```
运行程序后，两根手指点击状态栏即可调起这个调试的悬浮层。

[具体使用教程](http://swift.gg/2017/05/27/ui-debugging-information-overlay/)

> Note: 方法在iOS11之后非苹果内部设备不可使用。[iOS11上使用](http://www.cocoachina.com/ios/20171208/21467.html)

##### 24、监听系统时间的三个通知

* **NSSystemTimeZoneDidChangeNotification：**监听修改时间界面的两个按钮状态变化
* **UIApplicationSignificantTimeChangeNotification：** 监听用户改变时间 （只要点击自动设置按钮就会调用）,新的一天开始或者时区变化。 
* **NSSystemClockDidChangeNotification：** 监听用户修改时间（时间不同才会调用）

##### 25、UITableView的headerView/footerView的高度问题

开发中如果要动态修改tableView的tableHeaderView或者tableFooterView的高度，需要给tableView重新设置，而不是直接更改高度。正确的做法是重新设置一下```tableView.tableFooterView = 更改过高度的view```。

为什么？其实在iOS8以上直接改高度是没有问题的，在iOS8中出现了contentSize不准确的问题，这是解决办法。

##### 26、自定义聊天输入框弹出键盘修改frame

注册通知监听键盘将要改变的高度（UIKeyboardWillChangeFrame）。

保持输入框始终在键盘上方：

**初始状态**: inputTextView.top = kDeviceHeight-inputTextView.height-kNavBarHeight

**弹出状态**: inputTextView.top = kDeviceHeight-inputTextView.height-kNavBarHeight-keyBoardHeight

##### 28、找不到CAMetalLayer

原因：需要连接真机。

##### 29、tableView报错：failed to obtain a cell from its dataSource

* xib的cell没有注册。 
* 内存中已经有这个cell的缓存了(也就是说通过你的cellId找到的cell并不是你想要的类型)，这时候需要改下cell的标识

##### 30、Other Linker Flags的作用

苹果官方Q&A的一段话：

> The "selector not recognized" runtime exception occurs due to an issue between the implementation of standard UNIX static libraries, the linker and the dynamic nature of Objective-C. Objective-C does not define linker symbols for each function (or method, in Objective-C) - instead, linker symbols are only generated for each class. If you extend a pre-existing class with categories, the linker does not know to associate the object code of the core class implementation and the category implementation. This prevents objects created in the resulting application from responding to a selector that is defined in the category.

OC 的链接器不会给**每个方法**都建立符号表，而只是为**类**建立符号表。

如果在静态库中为一个**已存在的类**定义了category，链接器就会认为类已经存在，而不会把category和核心类的代码结合起来。因此会导致最终的可执行文件中，缺少category的代码，最终导致方法调用失败。报错信息一般为： unrecognized selector to instance 0xXXXXXXXX.

Other Linker Flags 的三个参数：```-Objc```,```-all_load```,```-force_load```。

* ```-ObjC```: 链接器会将静态库中所有的Objective-C类和category都加载到可执行文件中，这样会因为加载了很多不必要的文件而导致可执行文件增大。
* ```-all_load```（**慎用**）: 链接器会把所有找到的目标文件都加载到可执行文件中。如果项目中不止一个静态库文件，然后又使用的本参数。因为不同的库文件中有可能存在相同的文件，所以有可能会遇到```ld: duplicate symbol```错误，因此在```-ObjC```失效的情况下使用```-force_load```.
* ```-force_load```: 与-all_load相同，只是在不同库存在相同文件的情况下，只是会加载一个文件，不影响其他文件的按需加载.

##### 31、提交代码到GitHub，图表不计算contributions

代码库的user.email与GitHub账户没有关联。修改本地代码库的邮箱。可以在三个位置配置：

* ```/etc/gitconfig```: 系统所有用户。
* ```~/.gitconfig```: 当前用户，```git config --global user.email "邮箱地址"```。
* ```repo/.git/config```: 当前仓库， ```git config user.email "邮箱地址"```。

##### 32、 为UIWindow添加view

做悬浮球需求的时候，应该先设置```[window makeKeyAndVisiable]```，使window可见，然后再添加悬浮球，否则悬浮球的层级会出现在rootVC.view的下面。

##### 33、 .gitignore 未生效

.gitignore只能忽略没有被跟踪的文件，如果文件已经被纳入版本管理，修改.gitignore是无效的。

解决方案：需要先将本地缓存删除（将文件改为未跟踪状态），然后再提交。

```
git rm -r --cached .  // 删除全部缓存。 如果删除单个，将‘.’改为文件地址
git add .
git commit -m 'update .gitignore'
```

##### 34、valueforkey与objectforkey：   

在字典中，如果key是以“@”开头，那么objectforkey可以取出value，而valueforkey则会crash。    
因为valueforkey是KVC的方法，取值是找和指定key同名的属性，查不到时执行valueForUndefinedKey默认是直接抛出NSUndefinedKeyException异常。

##### 35、iOS 申请栈字符空间如果大于1024，需要申请到堆上，否则可能导致未知错误

##### 36、那些情况Xcode会添加autorelease？

* convenient的构造方法会加autorelease是因为有个编译属性 NS_RETURNS_NOT_RETAIN，alloc/new/copy/mutablecopy生成的是普通对象，因为编译属性是NS_RETURNS_RETAIN。

什么对象自动加入到 autoreleasepool中？

* 当使用alloc/new/copy/mutableCopy开始的方法进行初始化时，会生成并持有对象(也就是不需要pool管理)
* __weak修饰符只持有对象的弱引用，而在访问引用对象的过程中，该对象可能被废弃。那么如果把对象注册到autorealeasepool中，那么在@autorealeasepool块结束之前都能确保对象的存在。
* id的指针或对象的指针在没有显式指定时会被附加上__autorealeasing修饰符

##### 37、UITableViewCell 优化

1. 圆角优化
2. 缓存数据行高
3. 复杂层级重复性大的视图采用CoreText和CoreGraphic绘制
4. tableView快速滑动时，加载滚动结束处的内容

##### 39、Xcode 10 New Build System pod package 打包 framework 报错

Xcode 10 的默认编译系统变成了**New Build System**，之前为**Legacy Build System**。更新Xcode10之后有可能会编译报错。

报错信息：

```
Multiple commands produce '/User/...../Info.plist'
1) Target 'your target' has copy command from '/Users/.../One/Info.plist' to 'var/..../Info.plist'
1) Target 'your target' has copy command from '/Users/.../Two/Info.plist' to 'var/..../Info.plist'
```

新的编译系统下不允许出现多个Info.plist文件。

两种解决方案:

1. File -> Project Settings / Workspace Settings 修改 Build System 为 Legacy Build System。
2. project -> target -> Build Phases -> Copy Bundle Resources 删除冲突的Info.plist。

> Cocoapods-packager 打包插件使用的是默认编译系统，所以在Xcode 10环境下打包，也会报错，由于不会修改插件，所以采用第二种方式解决。

##### 41、Xcode Playground 自动运行卡住，一直处于Running App 状态

1. 在右侧的修改 platform 为 macOS
2. 在下方长按运行按钮，修改为 Manually Run (手动运行)

以上两种都不好使的话，使用万能方法：重启Xcode。

##### 42、swift Array 里的 map/filter/reduce/flatten

* map: 映射，将数组映射为另一个数组。
* filter: 过滤，数组通过过滤条件后返回符合条件的数组。
* reduce: 整合，遍历数组计算结果。
* flatten: 展开，将数组展开为一个数组（在数组元素为数组时）
* flatMap: 返回数组中不存在nil，会把Optional解包，高维数组降维

##### 42、swift 泛型和Any类型

Any 类型会避开 swift 的类型系统；泛型仍会由编译器进行类型检查。使用泛型的话可以在编译器的类型检查下安全的定义函数，推荐使用泛型。

```
func noOp<T>(_ x: T) -> T{
    return x
}
func noOpAny(_ x: Any) -> Any {
    return x
}
var result1 = noOp(1) + 2 // 3
var result = noOpAny(1) + 2  // Binary operator '+' cannot be applied to operands of type 'Any' and 'Int'
```

上例中，noOp 的参数和返回值是一致的，而noOpAny的参数和返回值则可以是不同的。将noOpAny的返回值设为0，可以通过编译，而noOp的返回值修改为0，就会报错，因为入参不一定是Int类型。

##### 42、NSString rangeOfString: NSNotFound

NSNotFound: 是 NSIntegerMax。当字符串为 nil 时，通过`rangeOfString:`取得的NSRange 的location和length的值都为0。因此下面判断不严谨：

```
    NSString * str = nil;
    if ([str rangeOfString:@"aa"].location != NSNotFound) {
        NSLog(@"%@",str);
    }else {
        NSLog(@"str is nil");
    }
```

##### 43、 mas 下载应用报错“无法使用此 Apple ID 重新下载”

苹果下载应用分为两步，先“获取”，然后安装，报错原因是还未获取（未添加到已购项目中），所以无法下载安装，添加之后就可以下载了。

##### 45、 什么是User-Agent

User-Agent是Http协议中的一部分，属于头域的组成部分，User Agent也简称UA。用较为普通的一点来说，是一种向访问网站提供你所使用的浏览器类型、操作系统及版本、CPU 类型、浏览器渲染引擎、浏览器语言、浏览器插件等信息的标识。UA字符串在每次浏览器 HTTP 请求时发送到服务器！

浏览器UA 字串的标准格式为： 浏览器标识 (操作系统标识; 加密等级标识; 浏览器语言) 渲染引擎标识 版本信息

##### 46、 Tagged Pointer

在64位系统中，为了优化 NSNumber NSString等小对象的存储，将数据直接存储在指针地址中。

##### 47、setNeedsDisplay 和 setNeedslayout

两个方法都是异步执行的。就字面意思理解：setNeedsDisplay 是标记为将要显示了，setNeedslayout 是标记为需要重新布局。

setNeedsDisplay 之后会调用 drawRect: 进行绘图。setNeedslayout 之后会调用 layoutSubViews 进行重新布局。

##### 48、 swift 中使用预处理

swift 支持 #if / #else/ #endif 的操作，类似于OC中的预处理宏定义。OC中是设置在Preprocessor Macros中，swift是要设置在 other swift flags中，且前要加上`-D`，可以写成`-DDEBUG`、`-D DEBUG`或者`"-D" "DEBUG"`。

[参考](https://www.cnblogs.com/Bob-wei/p/5237761.html)

##### 49、 swift 三方库报错

由于swift版本更新，很多三方库没有及时更新，或者本地的三方库版本过低，跟项目中默认的swift版本不同，导致的编译报错，可通过以下方式解决

1. 升级三方库。
2. 在该Pod的Target的 `swift language version` 中设置版本。
3. 在Podfile文件中，通过hook的方式，设置swift版本（CocoaPods post_install）。

##### 50、 重连调试进程

debug到真机上的进程，在断开连接之后，重新连线，继续debug的操作步骤：

Xcode菜单栏 -> Debug -> Attach to Process/Attach to Process by PID or Name.. -> 选择进程。

##### 51、 session 与 context

在iOS框架中，一般带 session 和 context 后缀的类的作用：

1. 管理其他类，负责沟通，可以很好的解耦。
2. 帮助管理复杂环境的内存。

两者不同之处：一般与硬件交互，例如摄像头（AVCaptureSession）、网卡（NSURLSession）等。没有硬件参与一般用context，例如绘图、自定义转场动画上下文等。

##### 52、 error: No signing certificate "iOS Development" found: No "iOS Development" signing certificate matching team ID "xxxx" with a private key was found.

证书文件没有导入或已失效，重新导入证书。

##### 52、 xxx.modulemap not found

使用pod-packager编译swift库报错，暂时未解决。[GitHub的issure](https://github.com/CocoaPods/cocoapods-packager/issues/211)

##### 53、 ViewController中的Timer的正确用法

ViewController中使用Timer的主要问题：

* ViewController强引用Timer，Timer又将target设为ViewController，造成的循环引用。

> 1. Timer的target是强引用，如果将ViewController弱引用持有Timer，ViewController会迟迟走不到dealloc中，无法将Timer销毁，所以ViewController依然无法被释放，还是循环引用。
> 2. 将销毁Timer放到viewDidDisappear等方法中，如果是从ViewController中push到其他的VC中，会导致Timer被销毁，pop之后还需要重新设置Timer。

因此需要一个第三方打破这种循环引用，初始化一个TimerHandler作为ViewController的属性（强引用），初始化Timer设置targe为TimerHandler（强引用），设置ViewController作为TimerHandler的delegate（弱引用），在ViewController的dealloc方法中将Timer置空。

```
	ViewController-->Timer-->TimerHandler
		 ∧_________(弱引用)________|

```

当ViewController被pop之后，只有一个TimerHandler的弱引用，没有其他强引用，所以ViewController会调用dealloc方法，将Timer置空，Timer被销毁后，Timer强持有的TimerHandler也因为失去强引用，所以会走到dealloc方法销毁，打破了循环引用的问题。

当从ViewController中push到其他VC时，因为Timer并没有被销毁，因此再pop回来时依然正常使用，不需要重新启动Timer。


##### 54、 Swift项目中通过CocoaPods引入OC的三方库，关于头文件引入问题

在swift项目中使用CocoaPods引入三方库，需要使用`use_frameworks!`（将三方库以静态库的方式导入）。

在CocoaPods版本0.36及以上在swift中导入OC的pod时不需要桥接头文件，直接在swift文件头部直接`import AFNetworking`即可。

* 如果引入的三方库是静态库`xxx.a`，还是需要在桥接文件中import头文件。
* 如果引入的三方库是`xxx.framework`，需要在podspec文件中设置: `spec.static_framework = true`

> [参考](!https://stackoverflow.com/questions/24122914/how-to-integrate-cocoapods-with-a-swift-project)


##### 55、 didMoveToWindow 与 didMoveToSuperview

* didMoveToSuperview：view被添加到父视图上，不一定会显示在屏幕上。
* didMoveToWindow：view被添加到当前的视图堆栈中。


##### 56、 静默推送

payload中 content-available = 1，表明会告诉系统，远程推送不应展示给用户，而是必须直接传递给应用，与普通推送类似，有可能会唤醒应用。

收到静默推送会走这个方法：

```
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler
```

##### 57、UNNotificationAttachment 附件内容通知

UNNotificationAttachment（附件通知）是指可以包含音频，图像或视频内容，并且可以将其内容显示出来的通知。使用本地通知时，可以在通知创建时，将附件加入即可。

创建附件的方法是`attachmentWithIdentifier:URL:options:error:`。在使用时，必须指定使用文件附件的内容，并且文件格式必须是支持的类型之一。

**URL必须是一个有效的文件路径**

##### 58、 WKWebView Cookie

UIWebView 会共享 NSHTTPCookieStorage 中的Cookie，WKWebView需要手动注入Cookie。

iOS 11 之后可以通过WKWebsiteDataStore，实现Cookie的存取。

```objective-c
// 注入Cookie
[self.webView.configuration.websiteDataStore.httpCookieStore setCookie:newCookie completionHandler:nil];
// 查询Cookie
[self.webView.configuration.websiteDataStore.httpCookieStore getAllCookies:^(NSArray<NSHTTPCookie *> * _Nonnull cookies) {
}];

```

在iOS 11 之前注入 Cookie：

```objective-c
// 初始化Cookie，注意格式，注意domain
NSString * cookieString = [NSString stringWithFormat:@"document.cookie='%@=%@;Domain=%@; Path=%@;'", cookie.name, cookie.value, cookie.domain, cookie.path];
// cookie script，
WKUserScript *cookieScript = [[WKUserScript alloc] initWithSource:qstring injectionTime:WKUserScriptInjectionTimeAtDocumentStart forMainFrameOnly:NO];
// user content
WKUserContentController *userContentController = [[WKUserContentController alloc] init];
// 添加脚本
[userContentController addUserScript:qscript];

// 在load request 之前再次设置Cookie，解决首次请求没有cookie的情况
[request addValue:[NSString stringWithFormat:@"%@=%@",cookie.name,cookie.value] forHTTPHeaderField:@"Cookie"];

```

上述代码可以解决同域后续请求的Cookie问题，但是无法解决跨域问题，下面可以实现跨域注入Cookie：

```
// 拦截网页请求，将Cookie添加到navigationAction.request.allHTTPHeaderFields中，重新load
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler;
```

WKWebView 不会将Cookie写入NSHTTPCookieStorage中（网上资料：会写入，但是有延迟。测试1天，未发现写入）。FireFox工程师建议通过重置WKProcessPool，来强制将Cookie同步到NSHTTPCookieStorage（网上：只能真机，未实验），确认模拟器下没有作用。

iOS11 之前WKWebView 读取Cookie 未实现。

##### 59、 命令行启动iOS模拟器

```shell
$ open -a Simulator
```

##### 60、 UILabel adjustsFontSizeToFitWidth

自动缩小字体，以适应最大的宽度。

当文本显示不下时，自动缩放字体，以使能显示全部文本。





