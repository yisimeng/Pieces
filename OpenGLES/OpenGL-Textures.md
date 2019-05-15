# 纹理（Textures）

纹理是一个2D图片，可以用来添加物体的细节。

**纹理坐标**：起始于左下（0，0），终于右上（1，1）。不依赖于分辨率，可以是任意浮点值。坐标之外，默认是重复这个纹理图像，可修改为其他环绕方式。

**采样**：使用纹理坐标，获取纹理颜色。

定义纹理坐标：

```
float texCoords[] = {
	0.0f, 0.0f, //左下
	1.0f, 0.0f, //右下
	0.5f, 1.0f, //上中
}
```

### 纹理环绕方式

| 环绕方式 | 描述 |
| :-- | :-- |
| GL_REPEAT | 重复纹理图像。默认 |
| GL_MIRRORED\_REPEAT | 重复，但图片是镜像放置的 |
| GL_CLAMP\_TO\_EDGE | 纹理坐标被约束在0-1，超出部分会重复纹理边缘，边缘被拉伸 |
| GL_CLAMP\_TO\_BORDER | 超出的坐标为用户指定的边缘颜色 |

纹理坐标轴：s、t、r。

```
// $1 指定纹理目标（2D）,$2 设置应用的纹理轴, $3 指定环绕方式
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_MIRRORED_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_MIRRORED_REPEAT);

// 如果是GL_CLAMP_TO_BORDER 还需要指定边缘颜色，需要使用后缀为fv的形式
float borderColor[] = {1.0f, 1.0f, 0.0f, 1.0f};
glTexParameterfv(GL_TEXTURE_2D, GL_TEXTURE_BORDER_COLOR, borderColor);
```

### 纹理过滤

将分辨率低的纹理映射到很大的物体上时，需要用到**纹理过滤**。纹理过滤有多种选项，最重要的是两种。

| 过滤方式 | 描述 | 低分辨率纹理应用到大物体 |
|:--|:--|:--|
| GL_NEAREST | 临近过滤，默认，选择中心点最接近纹理坐标的像素 | 会像素化，有颗粒感 |
| GL_LINEAR | 线性过滤，基于纹理坐标附近的纹理像素，计算插值，近似附近纹理像素之间的颜色 | 图案平滑，更真实 |

可以在进行放大和缩小操作时，选择纹理过滤，放大时使用线性，缩小时使用临近。

### 多级渐远纹理

距离观察者的距离超过一定阈值，OpenGL会使用不同的最适合物体距离的多级渐远纹理。距离远的解析度不高也不会被用户注意到。

渲染中切换多级渐远纹理级别时，会在不同级别的多级渐远纹理层之间产生不真实的边界，这里也可以使用NEAREST和LINEAR进行过滤。

| 过滤方式 | 描述 |
| :-- | :-- |
| GL_NEAREST_MIPMAP_NEAREST | 使用临近的多级渐远纹理来匹配像素大小，并临近插值进行纹理采样 |
| GL_LINEAR_MIPMAP_NEAREST | 使用最邻近的多级渐远纹理级别，并线性插值采样 |
| GL_NEAREST_MIPMAP_LINEAR | 在两个最匹配像素大小的多级渐远纹理间线性插值，并临近插值采样 |
| GL_LINEAR_MIPMAP_LINEAR | 两个临近的多级渐远纹理间线性插值，并线性插值采样 |

和纹理过滤一样，使用glTexParameteri设置四种过滤方式之一。

> 常见错误：将放大过滤的选项设置为多级渐远纹理过滤选项之一。不会有任何效果，因为多级渐远纹理主要使用在纹理被缩小的情况下。纹理放大不会使用多级渐远纹理，会产生GL_INVALID_ENUM错误。

## 加载与创建纹理

通过图像加载器，将图像转化为字节序列。

**stb_image.h**：是一个常用的弹头文件图像加载库，支持大部分流行的文件格式，易于集成。

### 生成纹理

```
unsigned int texture;
// $1 生成纹理数量，存储在$2中
glGenTextures(1, &texture);

// 绑定纹理
glBindTexture(GL_TEXTURE_2D, texture);

// 通过图片数据生成纹理
// $1 纹理目标；$2 指定多级渐远纹理的级别；$3 将纹理存储为何种格式，测试使用图像只有RGB值；$4、$5 设置纹理的宽和高；$6 历史遗留，总为0；$7、$8 源图的格式和类型；$9 图像数据
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
glGenerateMipmap(GL_TEXTURE_2D);
```

调用glTexImage2D，纹理图像会附加到纹理对象。基本级别的纹理图像，如果要使用多级渐变纹理，必须手动设置所有不同的图像（不断递增第二个参数）。或者在生成纹理之后，调用glGenerateMipmap自动为当前绑定的纹理生成所有需要的多级渐变纹理。

```
生成纹理和相应的多级渐变纹理后，释放图像的内存
stbi_image_free(data);
```

**采样器**：GLSL中供纹理对象使用的内建数据类型，以纹理类型作为后缀，sampler1D、sampler2D等。在片段着色器中使用GLSL内建的texture函数来采样纹理颜色。

### 纹理单元（Texture Unit）

一个纹理的位置通常称为一个纹理单元，片段着色器中可以设置多个纹理。一个纹理的默认纹理单元是0，默认激活。

```
// 在绑定之前激活纹理单元，GL_TEXTURE0默认总是被激活。
glActiveTexture(GL_TEXTURE0);
// 绑定纹理到当前激活的纹理单元
glBindTexture(GL_TEXTURE_2D, texture);
```

> OpenGL至少保证有16个纹理单元，从GL_TEXTURE0-GL_TEXTURE15，按顺序定义，在需要循环纹理单元时可以使用：GL_TEXTURE0+8获得GL_TEXTURE8。

**纹理混合**：OpenGL内建`mix(texture1, texture2, value)`函数，value在0.0-1.0之间，0则混合为texture1的颜色，1为texture2的颜色。











