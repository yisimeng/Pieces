# 导航控制器总结：
1、每个根视图控制器都有一个navigationItem属性，当这个视图控制器视图被嵌入到导航控制器中的时候，UINavigationController的navigationBar会根据视图控制器的navigationItem属性显示标题，按钮等  
2、UINavigationController push到哪个视图控制器，哪个视图控制器的视图将显示在导航控制器的contentView中，同时navigationBar也会根据navigationItem属性配置相应的外观   
3、UINavigationBar属于MVC中的V层，主要负责导航条内容的显示。并且以栈的形式管理一组navigationItems，显示内容由navigationItem提供，同时navigationBar还有自己的属性（barTintColor...）。   
4、UINavigationItem属于MVC中的M层，主要负责为navigationBar提供显示数据（title，按钮等）。    
5、UIBarButtonItem用于描述navigationBar上的按钮，也属于Model层。   
6、UINavigationController导航控制器属于MVC中的C层，主要管理视图控制器的切换（视图切换和Bar切换），并伴有动画效果。    
7、视图控制器控制自己的布局和事件处理，导航控制器只负责切换，重要属性有viewControllers，navigationBar。    
