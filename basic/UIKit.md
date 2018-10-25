# UIKit

## UIView

1. UIView 的初始化顺序

自定义 UIView 时重写父类方法，常用到 `init、initWithCoder、initWithFrame`，还有由nib初始化之后的方法 `awakeFromNib`。

| 初始化方法 | 调用顺序 |
|:---:|:---:|
| init | initWithFrame -> init |
| initWithFrame | initWithFrame |
| initWithCoder | initWithCoder -> awakeFromNib |


### UIViewAutoresizing

简单的自动布局，不如autolayout（iOS 6）那么强大，对于一些简单的页面使用autolayout设置约束过于繁琐，可以使用autoresizing。

```
typedef enum UIViewAutoresizing : NSUInteger {
    UIViewAutoresizingNone = 0, // 不会随父视图改变而改变
    UIViewAutoresizingFlexibleLeftMargin = 1 << 0, // 自动调整左边距，保证右边距保持不变。
    UIViewAutoresizingFlexibleWidth = 1 << 1, // 自动调整宽度，保证两侧间距不变。
    UIViewAutoresizingFlexibleRightMargin = 1 << 2, // 自动调整右边距，保证左边距保持不变。
    UIViewAutoresizingFlexibleTopMargin = 1 << 3, // 自动调整上边距， 保证下边距不变。
    UIViewAutoresizingFlexibleHeight = 1 << 4, // 自动调整高度，保证上下两边距不变。
    UIViewAutoresizingFlexibleBottomMargin = 1 << 5 //自动调整底部边距，保证上边距不变。
} UIViewAutoresizing;
```



## UINavigationController
1. 每个根视图控制器都有一个navigationItem属性，当这个视图控制器视图被嵌入到导航控制器中的时候，UINavigationController的navigationBar会根据视图控制器的navigationItem属性显示标题，按钮等  

2. UINavigationController push到哪个视图控制器，哪个视图控制器的视图将显示在导航控制器的contentView中，同时navigationBar也会根据navigationItem属性配置相应的外观   
3. UINavigationBar属于MVC中的V层，主要负责导航条内容的显示。并且以栈的形式管理一组navigationItems，显示内容由navigationItem提供，同时navigationBar还有自己的属性（barTintColor...）。   
4. UINavigationItem属于MVC中的M层，主要负责为navigationBar提供显示数据（title，按钮等）。    
5. UIBarButtonItem用于描述navigationBar上的按钮，也属于Model层。   
6. UINavigationController导航控制器属于MVC中的C层，主要管理视图控制器的切换（视图切换和Bar切换），并伴有动画效果。    
7. 视图控制器控制自己的布局和事件处理，导航控制器只负责切换，重要属性有viewControllers，navigationBar。
    


## UICollectionView

UICollectionView 重新定义布局，重写

```
func shouldInvalidateLayoutForBoundsChange(newBounds: CGRect) -> Bool
```
方法，返回true，在collectionView bounds改变时（滚动时），强制重新布局，会调用

```
layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]?
```
方法去获取之后的cell的位置。


## UIApplication

