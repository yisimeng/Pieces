# 坐标系统

顶点在最终被转化为片段之前要经历五种不同状态：

* 局部空间（Local Space, 或物体空间（Object Space））
* 世界空间（World Space）：
* 观察空间（View Space，或视觉空间（Eye Space））
* 裁剪空间（Clip Space）
* 屏幕空间（Screen Space）

**模型矩阵**能对物体进行位移、缩放、旋转，来改变物体的位置或者朝向。
**观察矩阵**通常有一系列位移和旋转组合完成，
**投影矩阵**指定一个范围的坐标，坐标外的都会被裁剪忽略

1. 物体的坐标从局部转换到世界空间，是由**模型矩阵**实现的。
2. 将世界空间坐标转化为视觉空间，由**观察矩阵**实现。
3. 将视觉空间转为裁剪空间，有**投影矩阵**实现。
4. 

**投影**：将特定范围内的坐标转化到标准化设备坐标系的过程。

**正射投影**：定义了一个类似立方体的平截头箱，空间之外的都被裁剪。平截头体由宽、高、近平面和远平面所指定。直接将坐标映射到2D平面，会产生不真实效果。

**透视投影**：近大远小。定义了一个不规则的立方体。平截头体由**视野**、宽高比、近平面和远平面索指定。

> **视野**值为45.0f时的效果更为真实，末日风格的值更大。
> 近平面通常设为0.1。远平面通常设为100.0f。

**视口变换**OpenGL对裁剪坐标执行透视除法，将其转换为**标准化设备坐标**。OpenGL使用 glViewPort 的内部参数来讲标准化设备坐标映射到**屏幕坐标**，每个坐标都关联一个屏幕的点。

> 怎么理解： 摄像机向后移动，和将整个场景向前移动是一样的？
> 
> 摄像机是朝向物体的方向的，摄像机的后方即是场景的前进的方向。

**深度(Z)缓冲**深度信息决定何时去覆盖一个像素。

**深度测试**深度值存储在每个片段的Z值中，片段着色器要输出颜色时，OpenGL会将它的深度值与Z缓冲进行比较，如果当前片段在其他片段之后，就会被丢弃，否则覆盖。默认是关闭的。

```
// 启用深度测试
glEnable(GL_DEPTH_TEST);
// 禁用深度测试
glDisable(GL_DEPTH_TEST);
```














