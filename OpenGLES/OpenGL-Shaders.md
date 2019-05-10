# 着色器 Shaders

着色器是运行在GPU上的小程序，为图形渲染管线的特定部分而运行。只有输入和输出。

### 着色器语言（GLSL）

GLSL 的语法：1. 声明版本；2. 输入和输出变量；3. uniform和main函数。

```
#version version_number (mode 可选？)
in type in_variable_name;
in type in_variable_name;

out type out_variable_name;

uniform type uniform_name;

int main () {
	
	/** 处理输入并进行图形操作 */
	
	// 输出处理后的结果到输出变量
	out_variable_name = processed;
}
```

### 数据类型

GLSL包含基础数据类型：int、float、double、uint、bool。除此还有两个容器类型，向量和矩阵。

#### 向量

向量是可以包含1-4个分量的容器，分量的类型可以是默认基础类型的任意一个。

| 类型 | 含义 |
|:-:|:-:|
| vecn | 包含n个float分量 |
| bvecn | 包含n个bool分量 |
| ivecn | 包含n个int分量 |
| uvecn | 包含n个uint分量 |
| dvecn | 包含n个double分量 |

**分量重组**

### 输入输出

GLSL定义 in 和 out 来表示着色器的输入和输出。

**顶点着色器**为了快速从顶点数据中接受输入，使用 layout (location = 0)，链接到顶点数据。

**片段着色器**因为要生成最终输出颜色，所以需要vec4。如果没在片段着色器中定义输出颜色，会被渲染成黑色。

从一个着色器向另一个着色器发送数据，发送方的输出和接收方的输入要同名同类型。

### Uniform

Uniform 是一种从CPU向GPU中的着色器发送数据的方式，但与顶点属性有些不同：uniform是全局的，可以被着色器程序中的任意着色器任意阶段访问；uniform会一直保存被设置的数据，直到重置或更新。

> 如果声明了uniform缺在GLSL代码中未使用，编译器会静默移除变量，可能会导致几个非常麻烦的错误。

```
// glGetUniformLocation 查询 uniform 关键字修饰的 ourColor的值，-1表示未找到
int vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor");
glUseProgram(shaderProgram);
// 设置 uniform 的值
glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);
```

更新 uniform 的值之前**必须**先激活着色程序（glUseProgram），是需要在激活的程序中设置的，查询 uniform 地址之前不需要。

### 更多属性

将颜色属性添加进顶点数据中

```
float vertices[] = {
	// 位置					//颜色
	 0.5f, -0.5f, 0.0f,  1.0f, 0.0f, 0.0f, // 右下
	-0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f, // 左下
	 0.5f,  0.5f, 0.0f,  0.0f, 0.0f, 1.0f, // 顶部
}
```

顶点数据中包含位置和颜色信息，所以定义着色器：

```
// 顶点着色器
#version 330 core
layout (location = 0) in vec3 aPos; //位置变量的属性位置为0
layout (location = 1) in vec3 aColor;// 颜色变量属性值为1
// 输出颜色
out vec3 ourColor;
void main() {
	gl_Position = vec4(aPos, 1.0);
	// 直接从顶点数据中指定颜色
	ourColor = aColor;
}

// 片段着色器
#version 330 core
out vec4, FragColor;
in vec3 ourColor;
void main() {
	FragColor = vec4(ourColor, 1.0f);
}
```

设置顶点属性指针：

```
// 设置位置属性 位置+颜色所以步长是6*sizeof(float)
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6*sizeof(float), (void*)0);
glEnableVertexAttribArray(0);
// 设置颜色属性 
// $1 顶点着色器设置了颜色变量属性值是1
// $6 偏移量，位置属性在前面偏移量是0，所以后面的颜色属性的偏移量是3*sizeof(float)
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6*sizeof(float), (void*)(3*sizeof(float)));
```

**片段插值**

### 着色器类













































