# Status Bar

关于 Info.plist 中的属性介绍

* **UIStatusBarHidden（Status bar is initially hidden）**: 布尔值，程序启动时的状态栏是否隐藏；
* **UIStatusBarStyle（Status bar style）**: 程序启动时状态栏的样式（default/Light Content/Dark Content）；
* **UIStatusBarTintParameters（Status bar tinting parameters）**：初始化时状态栏的图片/样式/透明
* **UIViewControllerBasedStatusBarAppearance（View controller-based status bar appearance）**：是否通过当前ViewController来控制状态栏外观。

### 由当前 ViewController 控制状态栏外观

**代码设置的前提是 Info.plist 文件中的 UIViewControllerBasedStatusBarAppearance 属性要设置为 YES。**

```
    /// 状态栏外观类型
    override var preferredStatusBarStyle: UIStatusBarStyle{
        return .default
    }
    /// 状态栏是否隐藏
    override var prefersStatusBarHidden: Bool{
        return true;
    }
    /// 状态栏隐藏属性改变时的动画
    override var preferredStatusBarUpdateAnimation: UIStatusBarAnimation{
        return .slide
    }
```

由当前视图控制器控制 status bar 是否因藏，外观类型，以及动画效果。

#### UINavigationController 的状态栏

当前视图如果是 UINavigationController，那么就会以 UINavigationController 的设置为准。如果想使用子视图控制器的设置，需要在 UINavigationController 中重写以下方法：

```
    /// 返回控制状态栏的子视图控制器
    override var childForStatusBarStyle: UIViewController?{
        // 由当前的顶视图控制器控制外观
        return self.topViewController
    }
    /// 状态栏外观
    override var preferredStatusBarStyle: UIStatusBarStyle{
        // 如果子视图控制器有单独设置，返回子视图的设置
        if let style = self.topViewController?.preferredStatusBarStyle {
            return style
        }
        return super.preferredStatusBarStyle
    }
    /// 状态栏隐藏
    override var prefersStatusBarHidden: Bool{
        // 如果子视图控制器有单独设置，返回子视图的设置
        if let hidden = self.topViewController?.prefersStatusBarHidden {
            return hidden
        }
        return super.prefersStatusBarHidden
    }
```