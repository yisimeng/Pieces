# Flutter

React Native：是基于原生的封装。iOS与Android的区别导致兼容性有问题。

Flutter：有自己的渲染引擎。iOS与Android平台保持一致性。

##### 1. 性能

1.1 Flutter App 使用Dart开发。Dart在JIT（即时编译）下与JavaScript差不多，另外Dart还支持AOT（预先编译）

1.2 Flutter使用自己的渲染引擎绘制UI，进行布局等。所以不需要再RN那样在JavaScript和Native之间通信。在滑动时因为RN需要频繁的和native进行布局同步，体验上Flutter要更好一些。



#### 2. 创建flutter工程

1. Android Studio

   安装好flutter和Dart插件之后，打开Android Studio ，选择**New Flutter Project**。

2. Terminal

   ```shell
   # 直接创建工程（语言默认为Swift 和 Kotlin）
   $ flutter create myapp
   # 指定平台语言(i:iOS, a: Android)
   $ flutter create -i objc -a java myapp
   ```



1. [Material](https://material.io/guidelines/) 是一种标准的移动端和web端的视觉设计语言。 Flutter提供了一套丰富的Material widgets。

2. **Widget** Flutter 中大部分东西都是 Widget，包括对齐(alignment)、填充(padding)和布局(layout)等。

   * StatelessWidget： 无状态的，意味着属性是不可变的，所有值都是最终的。
   * StatefulWidget：有状态，可变化的，可能在生命周期内属性发生改变。

   > StatefulWidget 需要两个类实现，StatefulWidget类 和State类。
   >
   > State类声明时，`class _RandomWordsState extends State<RandomWords>` 中的 <RandomWords> 是泛型，关联StatefulWidget类。在State类中可以通过widget来获取当前的StatefulWidget对象。

3. **Scaffold** 是 Material library 中提供的一个widget, 它提供了默认的导航栏、标题和包含主屏幕widget树的body属性。widget树可以很复杂。widget的主要工作是提供一个build()方法来描述如何根据其他较低级别的widget来显示自己。

4. 多控件的三种布局

   Row，Column，Stack，三种布局方式，都有children属性，子部件按照不同方向布局。

* **Row**：横向排布。

* **Column**：纵向排布。

* **Stack**： 延Z轴排布。

  布局中有个主轴对齐属性：

  ```dart
  enum MainAxisAlignment {
    start,// 从这一行的首开始布局
    end, // 以这一行的最后为结束布局
    center, // 居中布局
    spaceBetween, // 剩余的空间，平均分布到控件的中间
    spaceAround, // 剩余的空间，平均分布到控件的周围
    spaceEvenly, // 剩余空间和控件平均分布
  }
  ```

  交叉轴属性（仅Row和Column，Stack是没有的）

  ```dart
  // 以Row布局为例，Row为横向布局，cross 就为纵向对齐
  enum CrossAxisAlignment {
    start, // 以顶部对齐
    end, // 底部对齐
    center, // 中心对齐
    stretch, // 
    baseline, //以文字的基线对齐， 必须要设置'textBaseline'属性（TextBaseline），否则报错
  }
  
  enum TextBaseline {
    alphabetic, // 英文字符对齐方式
    ideographic, // 中文字符对齐方式
  }
  ```



**Expanded布局**：按主轴方向拉伸，会填充满（设置width属性将没有意义）。

**Positioned 是做相对布局**

```dart
const Positioned({
    Key key,
    this.left,
    this.top,
    this.right,
    this.bottom,
    this.width,
    this.height,
    @required Widget child,
  })
```

Positioned 布局是相对与**父视图**的。

**AspectRatio 宽高比布局**： 需要设置aspectRatio属性



#### **StateFulWidget**

> 内部会全部渲染，所以尽可能少用。



**手势**： `onTap`,`onTapDown`,`onTapCancel`

### Flutter 与 native交互

Flutter中定义了三种类型的Channel用于与native交互。

- **BasicMessageChannel**：持续通信，用于传递字符串和半结构化的消息，收到消息后可以回复消息。
- **MethodChannel**：单次方法调用，调用之后可以直接回传结果。如Flutter调用原生相机相册等；
- **EventChannel**：持续通信，用于数据流的通信，但收到消息后不能回复消息。通常用户原生想Flutter传递手机电量变化、网络连接变化等。

### Tips

1. 取消按钮点击时的波纹效果

   MaterialApp中的ThemeData 设置属性`highlightColor`和`splashColor`为透明。

2. **MediaQuery.removePadding**
   Flutter会为某些Widget默认设置一些padding，使用它包裹可以设置去掉这些padding。

3. 图片设置圆角

   ```dart
    Container(
                 width: 200,
                 height: 100,
                 child: ClipRRect(
                   borderRadius: BorderRadius.circular(20),
                   child: Image.asset('images/jicheng.png'),
                 )
               )
   ```

4. State 类的初始化方法：`void initState() {}` 只会调用一次。