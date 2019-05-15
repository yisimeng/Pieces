# OpenGL


### GLFW

GLFW 是专门针对OpenGL的C语言库，提供渲染物体所需最低限度的接口。

### GLAD

GLAD 是专门管理OpenGL的函数指针的，在调用任何OpenGL的函数之前需要初始化GLAD。

## 视口 Viewport

渲染之前告诉OpenGL渲染窗口的尺寸，即视口。调用 glViewport 设置窗口的维度：

```
// 前两个参数为左下角
glViewport(0,0,800,600);
```

#### 双缓冲

前帧缓冲保存最终输出的图像，并在屏幕显示，渲染指令都在后帧缓冲上绘制，当渲染完成之后，交换，图像就会显示出来。

### 图形渲染管线 Graphics Pipeline

原始图形数据经过输送管道，期间经过变化处理，最终呈现在屏幕的过程。

主要分为两部分：

* 将3D坐标转换为2D坐标；
* 将2D坐标转换为实际有颜色的像素。

### 着色器 Shader

图形渲染管线可以分为几个阶段，每个阶段会将前一阶段的输出作为输入。着色器是在管线中快速处理图形数据的处理核心。

着色器允许开发者配置，使用 着色器语言 GLSL (OpenGL shading Language)。

### 图形管线的阶段

顶点数据->顶点着色器->形状(图元)装配->几何着色器->光栅化->片段着色器->测试与混合

顶点数据：描述一个点的信息，是由顶点属性表示的。

各个阶段：

* 顶点着色器：把3D坐标转换为另一种3D坐标，同时允许对顶点属性做一些基本处理。
* 图元装配：将顶点装配成指定的图元形状。
* 几何着色器：通过产生新顶点构造出新的图元来生成其他形状。
* 光栅化阶段：把图元映射为屏幕上相应的像素，生成供片段着色器使用的片段，片段着色器之前会执行裁切，丢弃超出视图意外的所有像素，来提升效率。
* 片段着色器：计算像素的最终颜色。通常包含3D场景的数据（如光照，阴影，光的颜色等），可以用来倍计算最终像素的颜色。
* 测试和混合：检测片段对应的深度，用于判断像素与其他物体的前后位置，决定是否需要被丢弃。也会检查alpha值，并对物体进行混合，所以片段着色器计算出的像素颜色可能与实际效果不同。

一般至少定义一个顶点着色器和一个片段着色器（因为GPU中没有默认的顶点和片段着色器）。几何着色器是可选的，通常使用默认的。

### 顶点数据

定义一个三角形

```
float vertices[] = {
	-0.5f, -0.5f, 0.0f,
	 0.5f, -0.5f, 0.0f,
	 0.0f,  0.5f, 0.0f
};
```

### 标准化设备坐标

3D 坐标都在（-1，1）之间，范围内的才会被处理，范围外的会被裁剪丢弃。通常将z轴理解为**深度**，代表像素在空间中与你的距离，如果远可能被别的像素挡住，就会被丢弃。


### 顶点缓冲对象 VBO（Vertex Buffer Objects）

通过顶点缓冲对象管理GPU上存储顶点数据的内存。数据由CPU发送至显卡之后，顶点着色器立即能访问顶点。

```
// 使用唯一ID生成 VBO 对象
unsigned int VBO;
glGenBuffers(1, &VBO);
```

缓冲对象类型有多种，顶点缓冲对象的类型是 GL_ARRAY_BUFFER，OpenGL允许同时绑定多个不同类型缓冲。

```
// 缓冲绑定
glBindBuffer(GL_ARRAY_BUFFER, VBO);
```

绑定之后，使用任何在 GL_ARRAY_BUFFER 缓冲调用都会用来配置当前绑定的顶点缓冲对象。

```
// 将顶点数据复制到缓冲内存中， 绑定到指定缓冲中：数据大小：顶点数据：显卡管理数据的形势
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
```

* GL_STATIC_DRAW：数据几乎不会改变
* GL_DYNAMIC_DRAW：数据会改变很多
* GL_STREAM_DRAW：每次绘制都会改变

### 顶点着色器

```
// OpenGL 版本 模式
#version 330 core
// 使用in 声明输入的顶点属性
layout (location = 0) in vec3 aPos;

void main()
{
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}
```

gl_Position 是预定义的位置输出变量，是vec4类型

**向量** 前三个分量用顶点数据，第四个分量是用于透视除法上。

#### 编译着色器

```
// 创建着色器对象, 参数为着色器类型
unsigned int vertexShader;
vertexShader = glCreateShader(GL_VERTEX_SHADER);
// 给着色器赋值着色器源码  着色器对象：源码字符串数量：源码：？(长度？)
glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
// 编译着色器
glCompileShader(vertexShader);
        
// 检测编译是否成功
int success;
char infoLog[512];
glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);
if (!success) {
	// error msg
	glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
	std::cout << "编译失败\n" << infoLog << std::endl;
}
```

### 片段着色器

用于计算像素最后输出的颜色。

```
// 除了类型传片段着色器，其他与顶点一样
fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
```

### 着色器程序

着色器程序对象(Shader Program Object)是多个着色器合并之后并最终链接完成的版本。

```
// 创建着色器程序
unsigned int shaderProgram;
shaderProgram = glCreateProgram();

// 附加着色器对象
glAttachShader(shaderProgram, vertexShader);
glAttachShader(shaderProgram, fragmentShader);
// 链接
glLinkProgram(shaderProgram);
// 激活之后就可以着色器调用和渲染调用都会使用这个程序对象
glUseProgram(shaderProgram);
// 着色器链接到程序对象之后就可以删除了
glDeleteShader(vertexShader);
glDeleteShader(fragmentShader);
```

### 链接顶点属性

将顶点数据链接到顶点着色器的属性上，手动指定输入数据和顶点着色器顶点属性的对应关系，指定OpenGL如何解释顶点数据。

顶点缓冲数据是紧密排列的，数据间没有其他值。

```
// 解析顶点数据，应用到逐个顶点属性上
// $1：指定要配置的顶点属性
// $2：顶点属性的大小（vec3由三个值组成）
// $3：指定数据类型
// $4：数据是否标准化：true数据会被映射到0-1之间
// $5：步长，两个顶点属性之间的间隔。
// $6：位置数据在缓冲中起始位置的便宜量，类型是 void*
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FLASE, 3 * sizeof(float), (void*)0);
// 启用顶点属性，默认是禁用的
glEnableVertexAttribArray(0);
```

> 每个顶点属性从顶点缓冲对象（VBO）管理的内存中获取数据，顶点属性0会连接到他的顶点数据。

### 顶点数组对象 VAO（Vertex Array Object）

可以像缓冲对象一样被绑定，随后的顶点属性调用都会存储在VAO中，当配置顶点属性指针时，只需要将所有调用执行一次，之后在绘制物体只需要绑定相应的VAO。VAO存储以下内容：

* glEnableVertexAttribArray和glDisableVertexAttribArray的调用
* 通过glVertexAttribPointer设置的顶点属性配置
* 通过glVertexAttribPointer调用与定点属性关联的顶点缓冲对象

```
// 渲染循环中：绘制物体
// 激活着色程序
glUseProgram(shaderProgram);
// 绑定顶点数组对象
glBindVertexArray(VAO);
// 渲染三角形 绘制的图元的类型：顶点数组起始索引：绘制顶点个数
glDrawArrays(GL_TRIANGLES, 0, 3);
```

### 索引缓冲对象 EBO（Element Buffer Object）或者 IBO（Index Buffer Object）

加入绘制一个矩形，可以以两个三角形来组成一个矩形（OpenGL主要处理三角形）。

```
// 矩形的顶点集合
float vertexs[] = {
	 0.5f,  0.5f, 0.0f, //右上
	 0.5f, -0.5f, 0.0f, //右下 *
	-0.5f,  0.5f, 0.0f, //左上 #
	
	 0.5f, -0.5f, 0.0f, //右下 *
	-0.5f, -0.5f, 0.0f, //左下
	-0.5f,  0.5f, 0.0f  //左上 #
}
```

其中有重复的顶点，会造成浪费。EBO 是一个专门存储索引的缓冲，OpenGL调用顶点的索引，来决定改绘制那个顶点，即：**索引绘制**。

```
// 重新定义矩形顶点（不重复）
float vertexs[] = {
	 0.5f,  0.5f, 0.0f, //右上
	 0.5f, -0.5f, 0.0f, //右下
	-0.5f, -0.5f, 0.0f, //左下
	-0.5f,  0.5f, 0.0f, //左上
}
// 索引缓冲
unsigned int indices[] = {
	0, 1, 3, // 第一个三角形
	1, 2, 3  // 第二个三角形
}

// 创建索引缓冲对象
unsigned int EBO;
glGenBuffers(1, &EBO);
// 绑定缓冲类型为GL_ELEMENT_ARRAY_BUFFER 与 复制索引数据到缓冲
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);
```

**注意**： 索引从 0 开始。

```
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
//由于传的是索引当做缓冲目标，所以最后绘制要使用glDrawElements函数，来指明从索引缓冲渲染。
// 指明绘制三角形：顶点个数：索引的类型：EBO中的偏移量
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```

> 当目标是GL_ELEMENT_ARRAY_BUFFER时，VAO会存储glBindBuffer的调用，也会存储解绑调用，所以确保没在解绑VAO之前解绑EBO，否则就没有这个EBO配置了（先解绑VAO 在解绑EBO，这么理解？）。

**线框模式**（Wireframe Mode）：`glPolygonMode(GL_FRONT_AND_BACK, GL_LINE)`, $1 表示应用到三角形的正面和背面，$2 表示以线框绘制，默认为：GL_FILL。

### 重点理解！！！！：

* VBO 是管理显存中的数据的。
* VAO 是存储顶点属性调用的。
* EBO 是存储顶点数据索引的。

