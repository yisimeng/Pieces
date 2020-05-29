# Flutter

React Native：是基于原生的封装。iOS与Android的区别导致兼容性有问题。

Flutter：有自己的渲染引擎。iOS与Android平台保持一致性。

1. #### 创建flutter工程

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
   > State类声明时，`class _RandomWordsState extends State<RandomWords>` 中的 <RandomWords> 是泛型，关联StatefulWidget类。

3. **Scaffold** 是 Material library 中提供的一个widget, 它提供了默认的导航栏、标题和包含主屏幕widget树的body属性。widget树可以很复杂。widget的主要工作是提供一个build()方法来描述如何根据其他较低级别的widget来显示自己。