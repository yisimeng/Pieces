#OpenGLES  三  纹理  
**概念**   
纹理：用来保存图像的颜色元素值的OpenGLES缓存。  
像素：计算机屏幕上的一个实际的颜色点  
纹素：图像中每个像素的颜色数据  
视口坐标：帧缓存中的像素位置  
点阵化（reasterizing）：将几何形状的数据转换为帧缓存中的颜色像素的渲染步骤  
片元（fragment）：帧缓存中一个颜色像素  


OpenGLES坐标系：OpenGLES中的坐标系是和纯数据中相等的，但是渲染在屏幕上时GPU会转换成帧缓存中对应的真实像素位置，所以绘制的集合图形都会被拉伸。

**glTexParameteri()** 函数配置绑定的纹理，使OpenGLES知道该如何处理可用纹素的数量和需要被着色的片元间的数量不匹配  
- 多纹素对应少片元时：  
1>glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MIN_FILTER,GL_NEAREST)
从相配的多个纹素中取样，使用线性内插法混合颜色以得到最终颜色。  
2>glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MIN_FILTER,GL_LINEAR)使用与片元的UV坐标最接近的纹素颜色。
- 少纹素多片元时：  
1>glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MAG_FILTER,GL_NEAREST)
拾取与片元UV坐标位置接近的纹素的颜色并放大纹理，会像素化的渲染在三角形上  
2>glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MAG_FILTER, GL_LINEAR)
混合附近的纹素的颜色来计算片元颜色，会有纹理放大的效果

## 纹理缓存的初始化
