OpenGL横跨CPU和GPU两个处理器，高效的协调两个内存区域的数据交换。
首先，从一个内存复制数据到另一个内存速度是相对较慢的。其次所有的内存访问都是相对较慢的，处理器访问内存的速度达不到运算的速度（数据饥饿）。因此对于渲染速度，最快的数据交换方式是没有数据交换。

缓存（buffer）:指GPU能够控制管理的连续内存。程序从CPU的内存复制数据到缓存，GPU通过控制独占的缓存，就能尽可能一最快的方式读写内存。

#### 为缓存提供数据的七个步骤：
1、生成（Generate）—请求OpenGLES为GPU控制的缓存生成唯一标识。’glGenBuffers()’  
2、绑定（Bind）—告诉OpenGLES为接下来的运算使用一个缓存。’glBindBuffer()’  
3、缓存数据（Buffer Data）—让OpenGLES为绑定的缓存分配并初始化足够的连续内存（通常是从CPU中复制数据到分配的内存）。’glBufferData()’或者‘glBufferSubData()’  
4、启用/禁止（Enable/Disable）—在渲染中是否使用缓存中的数据。’glEnableVertexAttribArry()’或者’glDisableVertexAttribArray()’  
5、设置指针（set pointer）—确定缓存中的数据类型和访问数据的内存偏移值。’glVertexAttribPointer()’  
6、绘图（draw）—使用当前绑定并启用的缓存中的数据进行渲染。’glDrawArrays()’或者‘glDrawElements()’  
7、删（delete）—删除生成的缓存并释放资源。’glDeleteBuffers()’

渲染：将程序提供的几何数据转换为屏幕上的图像的过程。GPU控制的缓存是高效渲染的关键。渲染的结果通常保存在帧缓存中，其中前帧缓存和后帧缓存控制着屏幕像素的最终颜色。
OpenGLES上下文保存状态信息，包括用于提供数据的缓存地址和用于接受结果的缓存地址。

OpenGLES只渲染’点’，’线段’，’三角形’，复杂图像由大量三角形建立
