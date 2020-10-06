---
layout: post
title:  "OpenGL学习笔记二 Hello Window和Hello Triangle"
date:   2020-07-14 06:00:00
categories: OpenGL
tags: OpenGL C++
excerpt_separator: <!--more-->
---

* content
{:toc}

LearnOpenGL 的 4 和 5 节。主要是生成窗口，和在窗口中绘制三角形和矩形。使用到 Vertex Shader 和 Fragment Shader，Shader program，缓冲对象，数组对象等。
<!--more-->


## Hello Window

#### 包含库

保证在 glad 在 glfw 之前。

```cpp
#include <glad/glad.h>
#include <glfw3.h>
```
#### 初始化窗口

* glfw 初始化

```cpp
// 初始化glfw
glfwInit();
// 设置glfw版本
glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
// 设置glfw使用的模式是Core-profile模式
glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
```

* 窗口初始化及检查

```cpp
// 窗口创建函数glfwCreateWindow(宽度, 长度, 名称, ~, ~)
GLFWwindow* window = glfwCreateWindow(800, 600, "LearnOpenGL", NULL, NULL);
// 检查窗口是否创建成功
if (window == NULL)
{
	std::cout << "Failed to create GLFW window" << std::endl;
	glfwTerminate();
	return -1;
}
glfwMakeContextCurrent(window);
```

#### 检查GLAD加载

加载系统指定的 OpenGL 函数指针地址。

```cpp
if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
{
std::cout << "Failed to initialize GLAD" << std::endl;
return -1;
}
```

#### 视图窗口设置

设置窗口尺寸改变的回调函数及绑定相应的回调函数。设置的窗口大小回是之后的 OpenGL 坐标-1 到 1 映射到对应窗口坐标
```cpp
// 设置窗口尺寸set的回调函数
void framebuffer_size_callback(GLFWwindow* window, int width, int height)
{
	// 窗口尺寸改变函数，glViewport(左下角.x, 左下角.y, 宽，高)
	glViewport(0, 0, width, height);
}

// 主函数内绑定回调函数，窗口大小改变时调用
glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);
```

#### 准备引擎Engines
设置窗口的渲染循环（<font color=red size=4>render loop</font>）。一个 render loop 一般叫做一个 frame。
```cpp
// glfwWindowShouldClose检查循环是否要退出，可配合回调函数
while(!glfwWindowShouldClose(window))
{
	// 交换2D buffer(Double buffer， front buffer 包含现在展示数据， back buffer包含将要展示的数据)
	glfwSwapBuffers(window);
	// glfwPollEvents 检查输入触法(鼠标和按键)，更新窗口状态，触发相应的回调函数
	glfwPollEvents();
}
```
#### 设置输入Input

设置 Esc 按键检测及窗口退出
```cpp
// 设置输入回调函数
void processInput(GLFWwindow *window)
{
	// 检测输入是否为按下Esc, 是则退出。 GLFW_PRESS 按下, GLFW_RELEASE 释放。
	if(glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
	glfwSetWindowShouldClose(window, true);
}

// 绑定输入回调函数
processInput(window);
```

#### 渲染设置

在 render loop 中通常回设置渲染，这里设置窗口底色改变的渲染程序。
```cpp
// 设置用于清除屏幕的颜色。 setting function
glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
// 设置清除屏幕的方式, 
// GL_COLOR_BUFFER_BIT仅清除颜色, GL_DEPTH_BUFFER_BIT和GL_STENCIL_BUFFER_BIT。 
// state-using fanction
glClear(GL_COLOR_BUFFER_BIT);
```

#### 设置glfw资源清除

在窗口完全退出前，要释放 glfw 占用的资源
```cpp
glfwTerminate();
return 0;
```

## Hello Triangle

### 图形渲染管线 Graphics pipeline

图形渲染管线主要任务：1.将 3D 坐标转换为 2D 坐标， 2. 将 2D 坐标转换为渲染好的像素点。这些步骤高度定制，容易并行执行，因此可以同时运行多个并行小程序，这些小程序叫做着色器 shaders。 OpenGL 的 Shaders 使用的 OpenGL Shading Language.

graphics pipeine 工作流程如下：

1. Vertex Data(Vertex Attribute 存放顶点数据)
2. Vertex Shader(顶点着色器， 3D 坐标转换)
3. Shape Assembly(将顶点装配成图元 primitive, OpenGL 会需要指定绘制的图元类型， 常见有 GL_POINTS、 GL_TRIANGLES、 GL_LINE_STRIP)
4. Geometry Shader(构造新顶点，构造新 primitive)
5. Rasterization(光栅化， 将图元映射成片段着色器 fragment shader 使用的片段， 在进入片段着色器前进行相应的裁剪 Clipping)
6. Fragment Shader(片段着色器， 计算最终颜色，产生高级效果， 一个片段对应一个像素的所有数据)
7. Tests and Blending(测试 Alpha 值和混合 blending 阶段 检测深度和 stencil 值 判断物体的前后，决定是否丢弃).

在现代 OpenGL 中一定至少要定义一个顶点着色器和一个片段着色器。

在接下来开始前，先记住一下几个对象
* **VAO**: 顶点数组对象， Vertex Array Object
* **VBO**: 顶点缓冲对象， Vertex Buffer Object
* **EBO**: 索引缓冲对象， Element Buffer Object, 或者 Index Buffer Object

### 顶点输入
OpenGL 在 3D 坐标下工作， 通过 `glViewport` 将标准化设备坐标 NDC(<font color=red size=4>normalized device coordinates</font>)（-1.0 到 1.0 范围）转化到屏幕 2D 坐标系， 完成视角转化<font color=red size=4>viewport transform</font>。

```cpp
float vertices[] = {
	-0.5f, -0.5f, 0.0f,
	 0.5f, -0.5f, 0.0f,
	 0.0f,  0.5f, 0.0f
}
```

将数据传入 graphics pipeline 的第一步， vertex shader。在 GPU 中创建 vertex data 储存的 memory, 配置 OpenGL 如何解释 memory 和指定发送数据给 graphics card。

顶点缓冲对象 VBO（实际是个数组缓冲对象， 在这里并没有告诉 OpenGL 这是顶点，OpenGL 只知道是个数组）管理 vertex memory, 可以批量管理和发送数据。

```cpp
// 生成VBO对象
unsigned int VBO;
glGenBuffers(1, &VBO);
// 绑定VBO
glBindBuffer(GL_ARRAY_BUFFER, VBO);
// 设置缓冲对象数据，从vertices中复制数据 glBufferData(缓冲类型, 数据大小, 实际数据, 数据管理方式)
// 数据管理方式： GL_STREAM_DRAW:频繁改变, GL_STATIC_DRAW: 数据不改变, GL_DYNAMIC_DRAW:数据改变较多
// GL_STATIC_DRAW在不改变数据情况下，会把数据存储在内存中，可以更高效率
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
```

### 点着色器Vertex shader
使用 GLSL 写一个 Vertex shader。本例中并会进行计算，但真实使用时可能会在将输入数据换到 NDC 中。

* 指定版本 330->OpenGL3.3 版本， Core-profile 模式。
* 通过关键字 `in` 声明输入顶点属性（vertex attributes），`layout (location = 0)` 指明输入变量位置值， `vec3 aPos` 指明一个包含 3 个浮点数的 variable, 名为 aPos。
* 一个 vec 包含 x, y, z 和 w, 其中 w 并非表示位置信息，而在透视除法（perspective division）中使用。
* `gl_Position` 用于接收 vec4 的输出值，这里不需要考虑透视，所以设置 w 为 1.0。

```cpp
#version 330 core
layout (location = 0) in vec3 aPos;
void main()
{
	gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}
```

### 编译Shader

将 shader 用 C string 形式存储，利用 `glCreateShader` 创建 shader object, `glShaderSource` 指定 shader object 的 source code, `glCompileShader` 用于编译着色器。

```cpp
const char *vertexShaderSource = "#version 330 core\n"
	"layout (location = 0) in vec3 aPos;\n"
	"void main()\n"
	"{\n"
	" gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
	"}\0";

// 创建着色器
unsigned int vertexShader;
vertexShader = glCreateShader(GL_VERTEX_SHADER);

// 指定着色器资源文件. glShaderSource(着色器对象, 资源文件strings个数, source code string, ~)
glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
// 编译着色器代码
glCompileShader(vertexShader);
```

编译是否成功检查

```cpp
int success;
char infoLog[512];
// 检查函数 glGetShaderiv(着色器名称, 检查类型, 返回结果的地址引用)
glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);

// success != 0 代表失败
if(!success)
{
	// 获得error信息
	glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
	std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" <<
	infoLog << std::endl;
}
```

### 片段着色器 Fragment shader
使用 GLSL 写一个 fragment shader。

* 指定版本 330->OpenGL3.3 版本， Core-profile 模式。
* `out` 指定输出变量类型 vec4, 名称 FragColor。
* 一个 vec 包含 r, g, b 和 alpha, 其中 alpha=1.0 为完全可见。

```cpp
#version 330 core
out vec4 FragColor;
void main()
{
	FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0);
}
```

生成着色器，指定着色器资源，编译着色器和检查着色器编译。
```cpp
unsigned int fragmentShader;
fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
// fragmentShaderSource为资源的字符串变量
glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
glCompileShader(fragmentShader);

glGetShaderiv(fragmentShader, GL_COMPILE_STATUS, &success);

if(!success)
{
	glGetShaderInfoLog(fragmentShader, 512, NULL, infoLog);
	std::cout << "ERROR::SHADER::FRAGMENT::COMPILATION_FAILED\n" <<
	infoLog << std::endl;
}
```

#### 着色器程序

生成着色器程序，附加 Vertex shader，附加 fragment shader, 链接着色器程序，检查链接是否成功， 创建对象时使用程序。
在链接完成后可删除 vertex 和 fragment 着色器对象。

```cpp
// 生成着色器程序
unsigned int shaderProgram;
shaderProgram = glCreateProgram();
// 附加着色器和链接程序
glAttachShader(shaderProgram, vertexShader);
glAttachShader(shaderProgram, fragmentShader);
glLinkProgram(shaderProgram);
// 检查程序是否成功, 注意改变检查类型
glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
if(!success) {
glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);
	std::cout << "ERROR::SHADER::PROGRAM::LINKING_FAILED\n" <<
	infoLog << std::endl;
}

// 使用程序
glUseProgram(shaderProgram);
```

### 链接顶点属性

目前顶点缓冲区数据特征：
* float 数据格式，占用 32bit(4 字节)；
* 每个位置由 3 个 float 数据组成；
* 每个位置数据之间没有间隔，紧密排布（tightly packed）在数组中；
* 第一个数据就排在缓冲区的开始。

#### 顶点数组对象 VAO
VAO 可以 `glEnableVertexAttribArray` 和 `glDisableVertexAttribArray` 被激活/非激活函数， 可以通过 `glVertexAttribPointer` 获取顶点属性配置和绑定到顶点属性的 VBO。

```cpp
// 生成顶点数组对象
unsigned int VAO;
glGenVertexArrays(1, &VAO);
// 绑定顶点数组对象
glBindVertexArray(VAO);
// 绑定VBO, 复制数组里的数据到缓冲对象中
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

// glVertexAttribPointer 告诉OpenGL要如何使用顶点缓冲对象VBO(数组缓冲对象)
// 第一个变量: 指定vertex attribute,
// 第二个变量: vertex attribute的尺寸,
// 第三个变量: 数据类型,
// 第四个变量: 数据是否需要标准化,
// 第五个变量: 连续的vertex attibutes之间的间隔，步长。如果用0, 则默认告诉OpenGL是紧密排布的。
// 第六个变量: 在缓冲区位置的偏移量, 但要使用`void*`类型。
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
// 激活vertex attribute, 它默认是非激活状态的
glEnableVertexAttribArray(0);
```

#### 激活三角形
可以看到三角形

```cpp
glUseProgram(shaderProgram);
glBindVertexArray(VAO);
// 绘制图元 glDrawArrays(图元类型, 开始位置, 顶点个数)
glDrawArrays(GL_TRIANGLES, 0, 3);
```

### 索引缓冲对象 EBO
OpenGL 主要工作对象是三角形，三角形和另一个三角形有公用顶点，为减少数据存储，<font color="red" size="4">indexed drawing</font>是其解决方案。

```cpp
float vertices[] = {
	 0.5f,  0.5f, 0.0f, // top right
	 0.5f, -0.5f, 0.0f, // bottom right
	-0.5f, -0.5f, 0.0f, // bottom left
	-0.5f,  0.5f, 0.0f // top left
};
unsigned int indices[] = { // note that we start from 0!
	0, 1, 3, // first triangle
	1, 2, 3 // second triangle
};

// 生成三种对象
unsigned int VBO, VAO, EBO;
glGenVertexArrays(1, &VAO);
glGenBuffers(1, &VBO);
glGenBuffers(1, &EBO);

// 绑定顶点数组对象
glBindVertexArray(VAO);

// 绑定顶点缓冲对象
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

// 绑定EBO, 复制indices数据到缓冲对象中。GL_ELEMENT_ARRAY_BUFFER指定了缓冲对象的目标
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);

// 设置顶点属性指针的配置, 激活顶点属性
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);

// 在渲染循环中, 激活shader程序,绑定VAO
glUseProgram(shaderProgram);
glBindVertexArray(VAO);
// 绘制使用索引缓冲对象的图像
// glDrawElements(参数指定绘制图元, 第二个绘制的索引中的点的个数, 索引的数据类型，索引的开始位置)
// 不使用EBO时, 第四个参数也可以是index array
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```

`glDrawElements` 绘制图像时会从被绑定到 GL_ELEMENT_ARRAY_BUFFER 的 EBO 取索引。 在绑定 EBO 时，会将其存储为已绑定的 VAO 对象一部分。因此绑定 VAO 时也会绑定 EBO。在解绑 VAO 之前， 不要解绑 EBO。

在绘图前可以设置不同绘制模式，通过 `glPolygonMode(GL_FRONT_AND_BACK, GL_LINE)` 设置画线模式， 改成 `GL_FILL` 可切换回来。
