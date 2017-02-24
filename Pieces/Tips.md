[Toc]  

# Tips

## NSNumberFormatter
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
当我们根据label中的文字,通过```- (CGRect)boundingRectWithSize:(CGSize)size options:(NSStringDrawingOptions)options attributes:(nullable NSDictionary<NSString *, id> *)attributes context:(nullable NSStringDrawingContext *)context```方法计算完之后在转换，获得准确的label的frame

---

## frame和bounds：
frame是相对于父视图，bounds是相对于子视图的。  
scrollView和其子类，在滚动的时候，size是不变的，但是origin是会发生改变的，向左滑动，origin.x是增加的；向上滑动origin.y是增加的。当初始状态下origin为（0，0），如果向上滑动的了，然后想在屏幕上原来（0,0）的位置添加一个view的话，这时的y值是大于0的。

---

## UISearchBar
 UISearchBar貌似默认是没有回收时的动画的，最后加了一个动画，让它取消第一响应。。。。

 ---

## UITabBarController
 当重写navigationController的方法```pushViewController(_ viewController: UIViewController, animated: Bool)```时，如果我们用到tabbarController的话，一般会在这里统一设置hidesBottomBarWhenPushed属性为true，当push的时候隐藏掉tabbar。
```
 override func pushViewController(_ viewController: UIViewController, animated: Bool) {
        viewController.hidesBottomBarWhenPushed = true
        super.pushViewController(viewController, animated: animated)
  }
```
但是这是有问题的,刚启动就找不到tabbar了，设置rootViewController的时候也会走这个方法，导致第进来之后就直接隐藏掉tabbar了```public init(rootViewController: UIViewController) // Convenience method pushes the root view controller without animation.```
