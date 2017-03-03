GLKBaseEffect提供了不依赖于OpenGLES版本的控制OpenGLES渲染方法。

第一步：glGenBuffers (GLsizei n, GLuint* buffers);
n：用于指定生成的标识符数量
buffers：是一个指针，指向生成的标识符的保存地址。

第二步：glBindBuffer (GLenum target, GLuint buffer);
target：指定绑定缓存的类型
buffer：要绑定的缓存的标识符

第三步：glBufferData (GLenum target, GLsizeiptr size, const GLvoid* data, GLenum usage);
target：指定要更新当前上下文中所绑定的缓存
size：要复制进缓存的字节数量
data：要复制的字节的地址
usage：提示缓存在未来可能会被怎样使用

第四步：
glEnableVertexAttribArray (GLuint index)

第五步：glVertexAttribPointer (GLuint indx, GLint size, GLenum type, GLboolean normalized, GLsizei stride, const GLvoid* ptr)
indx：指示当前绑定的缓存包含每个顶点的位置信息
size：指示每个位置有几个部分
type：每个部分保存的值类型
normalized：小数点固定数据是否可以被改变
stride：每个顶点所占的字节
ptr：访问顶点数据的起始位置

第六步：glDrawArrays (GLenum mode, GLint first, GLsizei count);
mode：GPU处理顶点数据的方式
first：要渲染的第一个顶点的位置
count：需要渲染的顶点数量

第七步：glDeleteBuffers (GLsizei n, const GLuint* buffers);

每当一个GLKView要被重绘时，都会让保存在视图的上下文属性中的OpenGLES的上下文成为当前上下文。  
glClear()函数用来清楚缓存中的颜色。
