# OpenGL 渲染





1. 图片撕裂

   苹果：垂直同步Vsync + 双帧缓冲区。

   垂直同步Vsync：对帧缓存区扫描信号加锁，扫描完成后解锁。

   双帧缓冲区：两个缓冲区，交换显示。

2. 掉帧

   接收Vsync，CPU/GPU 渲染流水线耗时过长（CPU或者GPU未处理完图片数据），单位时间内未完成渲染，导致仍显示之前的图像。

   三帧缓存区，（CPU/GPU闲置时，多渲染一个缓冲区），也有可能掉帧。



1. 屏幕卡顿的原因

   CPU/GPU 渲染流水线耗时过长，导致的掉帧。

#### Core Animation 渲染流程

1. HandleEvents：事件处理

2. Commit Transaction：图片->解码（CPU）

3. Render Server：等到下一个runloop，进行绘制。

Core Animation -> 提交到OpenGL，OpenGL调用GPU进行渲染（顶点数据->顶点着色器->片元着色器）->渲染完成后等待下一个runloop。