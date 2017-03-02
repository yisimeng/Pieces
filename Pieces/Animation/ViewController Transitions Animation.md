### 自定义控制器present过度动画
UIKit允许自定义过度动画，只需使主视图控制器遵循UIViewControllerTransitioningDelegate协议（设置控制器的```transitioningDelegate```属性）。

当需要present或者dismiss一个ViewController时，UIKit会调用delegate方法询问是否使用自定义的过渡，并返回一个遵循```UIViewControllerAnimatedTransitioning```协议的对象
```
func animationController(forPresented presented: UIViewController, presenting: UIViewController, source: UIViewController) -> UIViewControllerAnimatedTransitioning?

func animationController(forDismissed dismissed: UIViewController) -> UIViewControllerAnimatedTransitioning?
```   

UIViewControllerAnimatedTransitioning协议有两个必须要实现的方法：
```
1、func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval

2、func animateTransition(using transitionContext: UIViewControllerContextTransitioning)
```

分别是控制过渡动画的时间，和通过上下文设置动画的具体实现。
重点就在第二个方法，唯一的UIViewControllerContextTransitioning参数。  
当两个视图控制器转换的时候，当前存在并显示的view会被添加到transitionContext的containerView中，另一个view被创建但是不可见的，因此转换的工作就是需要手动的，将新的view动画进来，老的view动画移走。   
默认情况下，在过渡转换结束后会将旧的view移除。   
通过上下文的```func view(forKey key: UITransitionContextViewKey) -> UIView?```方法，可以获取到‘toView’和‘fromView’。  

**确定动画的view**
* 当present时：toView是需要动画的view
* 当dismiss时：fromView是需要动画的view
