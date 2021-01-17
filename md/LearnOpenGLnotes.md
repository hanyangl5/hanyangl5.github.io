
Learn OpenGL notes

2019.8刚开始看opengl时的笔记（基本上是copypaste）

# OpenGL 简介


**OpenGL**（英语：Open Graphics Library，译名：开放图形库或者“开放式图形库”）是用于渲染2D、3D矢量图形的跨语言、跨平台的应用程序编程接口（API）

- OpenGL是一个由[Khronos组织](http://www.khronos.org/)制定并维护的规（Specification），它定义了一个跨编程语言、跨平台的编程接口的规格，OpenGL规范严格规定了每个函数该如何执行，以及它们的输出值。至于内部具体每个函数是如何实现（Implement）的，将由OpenGL库的开发者自行决定。




# 创建窗口



首先，包含OpenGL所用的头文件


    #include <glad/glad.h>
    #include <GLFW/glfw3.h>



### 创建窗口

实例化glfw窗口

<!-- more -->


```
//初始化函数库
glfwInit();

//将主版本号(Major)和次版本号(Minor)都设为3
glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);

//使用的是核心模式(Core-profile)glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

// uncomment this statement to fix compilation on OS X
#ifdef __APPLE__
glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE); 
#endif
```

我们在main函数中调用glfwInit函数来初始化GLFW，然后我们可以使用glfwWindowHint函数来配置GLFW。glfwWindowHint函数的第一个参数代表选项的名称，我们可以从很多以GLFW_开头的枚举值中选择；第二个参数接受一个整型，用来设置这个选项的值。
我们需要告诉GLFW我们要使用的OpenGL版本是3.3，这样GLFW会在创建OpenGL上下文时做出适当的调整。这也可以确保用户在没有适当的OpenGL版本支持的情况下无法运行。我们将主版本号(Major)和次版本号(Minor)都设为3。我们同样明确告诉GLFW我们使用的是核心模式(Core-profile)。明确告诉GLFW我们需要使用核心模式意味着我们只能使用OpenGL功能的一个子集（没有我们已不再需要的向后兼容特性）。

如果使用的是Mac OS X系统，你还需要加下面这行代码到你的初始化代码中这些配置才能起作用

```
#ifdef __APPLE__
glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE); 
#endif
```



### 配置窗口参数

接下来我们创建一个窗口对象，这个窗口对象存放了所有和窗口相关的数据，而且会被GLFW的其他函数频繁地用到。


```
GLFWwindow* window = glfwCreateWindow(800, 600, "LearnOpenGL", NULL, NULL);
glfwCreateWindow函数需要窗口的宽和高作为它的前两个参数。第三个参数表示这个窗口的名称
创建完窗口我们就可以通知GLFW将我们窗口的上下文设置为当前线程的主上下文了。

glfwMakeContextCurrent(window);
```

GLAD是用来管理OpenGL的函数指针的，在调用任何OpenGL的函数之前我们需要初始化GLAD。


```
if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
{
	std::cout << "Failed to initialize GLAD" << std::endl;
	return -1;
}
```

在我们开始渲染之前还有一件重要的事情要做，我们必须告诉OpenGL渲染窗口的尺寸大小，即视口(Viewport)，这样OpenGL才只能知道怎样根据窗口大小显示数据和坐标。我们可以通过调用glViewport函数来设置窗口的维度(Dimension)

```
glViewport(0, 0, 800, 600);
```

glViewport函数前两个参数控制窗口左下角的位置。第三个和第四个参数控制渲染窗口的宽度和高度（像素）。

我们实际上也可以将视口的维度设置为比GLFW的维度小，这样子之后所有的OpenGL渲染将会在一个更小的窗口中显示，这样子的话我们也可以将一些其它元素显示在OpenGL视口之外。

OpenGL幕后使用glViewport中定义的位置和宽高进行2D坐标的转换，将OpenGL中的位置坐标转换为你的屏幕坐标。例如，OpenGL中的坐标(-0.5, 0.5)有可能（最终）被映射为屏幕中的坐标(200,450)。注意，处理过的OpenGL坐标范围只为-1到1，因此我们事实上将(-1到1)范围内的坐标映射到(0, 800)和(0, 600)。



### 渲染循环

我们希望程序在我们主动关闭它之前不断绘制图像并能够接受用户输入。因此，我们需要在程序中添加一个while循环，我们可以把它称之为渲染循环(Render Loop)，

```
while(!glfwWindowShouldClose(window))
{
    glfwSwapBuffers(window);
    glfwPollEvents();    
}
```

1. glfwWindowShouldClose函数在我们每次循环的开始前检查一次GLFW是否被要求退出，如果是的话该函数返回true然后渲染循环便结束了，之后为我们就可以关闭应用程序了。

2. glfwPollEvents函数检查有没有触发什么事件（比如键盘输入、鼠标移动等）、更新窗口状态，并调用对应的回调函数（可以通过回调方法手动设置）。

3. glfwSwapBuffers函数会交换颜色缓冲（它是一个储存着GLFW窗口每一个像素颜色值的大缓冲），它在这一迭代中被用来绘制，并且将会作为输出显示在屏幕上。

4. glfwWindowShouldClose函数在我们每次循环的开始前检查一次GLFW是否被要求退出，如果是的话该函数返回true然后渲染循环便结束了，之后为我们就可以关闭应用程序了。

5. glfwPollEvents函数检查有没有触发什么事件（比如键盘输入、鼠标移动等）、更新窗口状态，并调用对应的回调函数（可以通过回调方法手动设置）。

6. glfwSwapBuffers函数会交换颜色缓冲（它是一个储存着GLFW窗口每一个像素颜色值的大缓冲），它在这一迭代中被用来绘制，并且将会作为输出显示在屏幕上。



### 回调函数

然而，当用户改变窗口的大小的时候，视口也应该被调整。我们可以对窗口注册一个回调函数(Callback Function)，它会在每次窗口大小被调整的时候被调用。这个回调函数的原型如下：
void framebuffer_size_callback(GLFWwindow* window, int width, int height);
这个帧缓冲大小函数需要一个GLFWwindow作为它的第一个参数，以及两个整数表示窗口的新维度。每当窗口改变大小，GLFW会调用这个函数并填充相应的参数供你处理。

```
void framebuffer_size_callback(GLFWwindow* window, int width, int height)
{
    glViewport(0, 0, width, height);
}
```

我们还需要注册这个函数，告诉GLFW我们希望每当窗口调整大小的时候调用这个函数：

```
glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);
```

当窗口被第一次显示的时候framebuffer_size_callback也会被调用。对于视网膜(Retina)显示屏，width和height都会明显比原输入值更高一点。
我们还可以将我们的函数注册到其它很多的回调函数中。比如说，我们可以创建一个回调函数来处理手柄输入变化，处理错误消息等。我们会在创建窗口之后，渲染循环初始化之前注册这些回调函数。



### 输入控制

我们同样也希望能够在GLFW中实现一些输入控制，这可以通过使用GLFW的几个输入函数来完成。我们将会使用GLFW的glfwGetKey函数，它需要一个窗口以及一个按键作为输入。这个函数将会返回这个按键是否正在被按下。我们将创建一个processInput函数来让所有的输入代码保持整洁。

 ```
void processInput(GLFWwindow *window)
{
    if(glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
      glfwSetWindowShouldClose(window, true);
}
 ```

  接着，把processInput(window)插入到渲染循环中，因为我们想让这些渲染指令在每次渲染循环迭代的时候都能被执行



### 测试

为了测试一切都正常工作，我们使用一个自定义的颜色清空屏幕。在每个新的渲染迭代开始的时候我们总是希望清屏，否则我们仍能看见上一次迭代的渲染结果（这可能是你想要的效果，但通常这不是）。我们可以通过调用glClear函数来清空屏幕的颜色缓冲，它接受一个缓冲位(Buffer Bit)来指定要清空的缓冲，可能的缓冲位有GL_COLOR_BUFFER_BIT，GL_DEPTH_BUFFER_BIT和GL_STENCIL_BUFFER_BIT。由于现在我们只关心颜色值，所以我们只清空颜色缓冲。除了glClear之外，我们还调用了glClearColor来设置清空屏幕所用的颜色。当调用glClear函数，清除颜色缓冲之后，整个颜色缓冲都会被填充为glClearColor里所设置的颜色。在这里，我们将屏幕设置为了类似黑板的深蓝绿色。

```
glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
glClear(GL_COLOR_BUFFER_BIT);
```



### 释放内存

当渲染循环结束后我们需要正确释放/删除之前的分配的所有资源。我们可以在main函数的最后调用glfwTerminate函数来完成。

```
glfwTerminate();
return 0;
```

# 三角形的绘制


## 顶点输入

要绘制一个三角形，首先我们要定义一个float类型的数组来储存三角形的三个顶点，我们将三个顶点的z坐标均设为0以表示我们要绘制的三角形是一个平面图形

```
float vertices[] = {
    -0.5f, -0.5f, 0.0f,
     0.5f, -0.5f, 0.0f,
     0.0f,  0.5f, 0.0f
};
```

<!-- more -->



## 顶点缓冲对象

定义顶点数据后，我们会把它作为输入发送给顶点着色器，它会在GPU上创建一片内存储存我们的顶点数据，我们用顶点缓冲对象（[Vertex Buffer Object](https://en.wikipedia.org/wiki/Vertex_buffer_object)，简称VBO）来操作这片内存，VBO实质上是一个`unsigned int`型变量，我们用[`glGenBuffer()`](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glGenBuffers.xhtml)函数创建缓冲并将缓冲的ID返回给VBO

```
GLuint VBO;
glGenBuffers(1, &VBO);
```

调用`glGenBuffer()`后，我们得到了缓冲对象的ID，但它还不是真正的缓冲对象，在缓冲对象的ID和缓冲类型通过[`glBindBuffer()`](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glBindBuffer.xhtml)函数绑定后，其对应的缓冲对象才会被创建出来

```
glBindBuffer(GL_ARRAY_BUFFER, VBO);
```

上文代码中的`GL_ARRAY_BUFFER`就是一个缓冲对象类型，一个缓冲类型只能和一个缓冲对象绑定，如果新的缓冲对象绑定了已绑定的缓冲对象类型，那么先前的绑定将会销毁。在OpenGL红宝书中给出了一个恰当的比喻：绑定对象的过程就像设置铁路的道岔开关，每一个缓冲类型中的各个对象就像不同的轨道一样，我们将开关设置为其中一个状态，那么之后的列车都会驶入这条轨道。

此后，我们使用的任何（在GL_ARRAY_BUFFER目标上的）缓冲调用都会用来配置当前绑定的缓冲(VBO)。然后我们可以调用`glBufferData()`函数，它会把之前定义的顶点数据复制到缓冲的内存中

```
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
```





 ## 顶点数组对象

顶点数组对象（VAO）也是一个 OpenGL 对象，它存储提供顶点数据所需的所有状态，即告诉OpenGL如何读取VBO中所储存的定点数据，创建VAO的方式和VBO类似，通过[`glGenVertexArrays()`](https://www.khronos.org/registry/OpenGL-Refpages/gl4/)，[`glBindVertexArray()`](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glBindVertexArray.xhtml)创建并绑定

```
GLunit VAO;
glGenVertexArrays(1, &VAO);
glBindVertexArray(VAO);
```

我们通过[`glVertexAttribPointer()`](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glVertexAttribPointer.xhtml)告诉OpenGL如何解析数据
![](https://learnopengl-cn.github.io/img/01/04/vertex_attribute_pointer.png)

```
glVertexAttribPointer(
   0,                  // attribute 0. No particularreason for 0, but must match the layout in the shader.
   3,                 // size
   GL_FLOAT,          // type
   GL_FALSE,          // normalized?
   3 * sizeof(float), // stride
   (void*)0           // array buffer offset
);

glEnableVertexAttribArray(0);
```
- 第一行代码表示我们需要启用索引为0的顶点属性。
- 然后第二行表明我们要使用vertexbuffer内的数据，下面的操作都是针对这个缓冲区的。
- 第三行代码中函数的参数比较多，第一个参数指明要操作的属性索引值，第二个参数则说明每个顶点属性需要几个数据（可以为1,2,3或4），因为vertexbuffer里现在存储了3X3=9个数据，而实际上3个是一组，每个顶点需要使用3个数据。第三个参数指定了缓冲区内每个数据的类型，这里顶点坐标使用的是浮点类型。
- 第四个参数表明数据是否需要normalized，归一化（对于有符号整数，归一化将使数据保持在[-1, 1]范围内，对于无符号整数，则在范围[0, 1]）。
- 第五个参数是步幅，指定了两个连续的顶点属性之间的偏移量（以字节为单位）。因此数值为3*sizeof(float)。
- 最后一个参数看似是一个指针，但实际上它并不是起到指针的作用。实际上，它表明缓冲区的开头距离第一个顶点属性之间的偏移量。这里，缓冲区里第一个顶点属性之前并没有任何额外的信息，因为我们取值为0。





## 着色器



### 顶点着色器
  顶点着色器(Vertex Shader)是几个可编程着色器中的一个。现代OpenGL需要我们至少设置一个顶点和一个片段着色器。我们会配置两个非常简单的着色器来绘制一个三角形

```
const char* vertexShaderSource =
"#version 330 core										 \n"
"layout(location = 0) in vec3 aPos;						  \n"
"void main() {											\n"
"	gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);}	   \n";

```

### 片段着色器


```
const char* fragmentShaderSource =
"#version 330 core										  \n"
"out vec4 FragColor;									  \n"
"void main(){											  \n"
"	FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);}			    \n";

```

###  编译着色器

```
	unsigned int vertexShader;
	vertexShader = glCreateShader(GL_VERTEX_SHADER);
	glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
	glCompileShader(vertexShader);

	unsigned int fragmentShader;
	fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
	glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
	glCompileShader(fragmentShader);
```
### 调用着色器程序
```
	unsigned int shaderProgram;
	shaderProgram = glCreateProgram();
	glAttachShader(shaderProgram, vertexShader);
	glAttachShader(shaderProgram, fragmentShader);
	glLinkProgram(shaderProgram);

```



## 绘制

要想绘制我们想要的物体，OpenGL给我们提供了glDrawArrays函数，它使用当前激活的着色器，之前定义的顶点属性配置，和VBO的顶点数据（通过VAO间接绑定）来绘制图元。

```
//绘制代码（渲染循环中）

glUseProgram(shaderProgram);
glBindVertexArray(VAO);
glDrawArrays(GL_TRIANGLES, 0, 3);
```

输出结果

![](https://ae01.alicdn.com/kf/H1d246230c31946fba17dd8479c9eeed2H.png)


# 着色器


# GLSL

着色器是使用一种叫GLSL的类C语言写成的。GLSL是为图形计算量身定制的，它包含一些针对向量和矩阵操作的有用特性。

着色器的开头总是要**声明版本**，接着是**输入和输出变量**、**uniform**和**main**函数

一个典型的着色器有下面的结构：

```
//声明版本
#version version_number 

//输入变量
in type in_variable_name;
in type in_variable_name;

//输出变量
out type out_variable_name;

//uniform
uniform type uniform_name;

//main函数
int main()
{
  // 处理输入并进行一些图形操作
  ...
  // 输出处理过的结果到输出变量
  out_variable_name = weird_stuff_we_processed;
}
```



<!-- more -->

# 数据类型

GLSL包含C语言等其它语言大部分的默认基础数据类型：`int` `float` `double` `uint`和`bool`，此外，GLSL还包含内置的一些容器类型，如`vecor ` `matrix` `sampler`

可以用如下方式声明一个`vector2`变量

```
vec2 testVec2 = vec2(0.5, 0.7);
```

可以用如下方式声明一个matrix4变量

```
mat4 testMat4 = mat4(1.0);
```



# 输入输出

GL3.x中，GLSL废弃了`attribute`，`varying`关键字，定义了`in` ，`out` 两个关键字来处理着色器的输入和输出



### 顶点着色器

以下是一个基本的**顶点着色器**

```
#version 330 core				

in vec3 testPos;						     

void main() 
{											 
	gl_Position = vec4(testPos.x, testPos.y, testPos.z, 1.0);
}
```

这种方式通常用来输入顶点坐标，法线，纹理坐标，顶点颜色等，它可以访问数据缓冲区，随时动态修改

在定义了输入变量后，需要链接程序中的顶点数据和Shader中的变量，GLSL允许我们用两种方式链接顶点数据和变量

 > **使用glBindAttribLocation**
>
  > 第一种方法是通过glBindAttribLocation函数来实现索引和变量之间的对应关系。
>
  > `glBindAttribLocation (shaderProgram, 0, "testPos");`
>
  > 上述代码规定，shader里名字为testPos的变量对应顶点属性索引为0。如果使用这种方法，你需要确保在编译程序之后、链接程序之前就调用它们



> **在Shader中直接指定**
>
> 例：
>
> ```
> layout(location = index) in vec3 testPos;
> ```
>
> 这样的代码layout(location = index)指定了顶点属性testPos对应了索引值index
>
> ```
>  glVertexAttribPointer(	
>  	GLuint index, //索引值
>  	GLint size,
>  	GLenum type,
>  	GLboolean normalized,
>  	GLsizei stride,
>  	const GLvoid * pointer);
> ```
>
> 在原文件中用glVertexAttribPointer()函数指定顶点的索引值index



**我们通常用第二种方式来定义索引的对应关系**

程序结束后，GLSL的内置变量gl_Position的值会成为该顶点着色器的输出值



### 片段着色器

以下是一个基本的**片段着色器**

```
#version 330 core		

out vec4 FragColor;		

void main()
{											  
	FragColor = vec4(1.0, 1.0, 1.0, 1.0);
}
```

片段着色器需要输出一个vec4类型的变量（R，G，B，A）作为着色器渲染的颜色，也可以用内置变量gl_FragColor作为片段着色器的输出



### 发送数据

如果我们要从一个着色器向另一个着色器发送数据，我们必须在发送方着色器中声明一个输出，在接收方着色器中声明一个类似的输入。当类型和名字都一样的时候，OpenGL就会把两个变量链接到一起，它们之间就能发送数据了，下图所示的代码演示了如何通过顶点着色器向片段着色器发送数据

```
#version 330 core				

layout(location = 0) in vec3 testPos;						     

out vec4 vertexColor; //输出给片段着色器

void main() 
{											 
	gl_Position = vec4(testPos.x, testPos.y, testPos.z, 1.0);
	
	vertexColor = vec4(1.0, 1.0, 1.0, 1.0);
}

```

```
#version 330 core	

in vec4 vertexColor; // 从顶点着色器输入

out vec4 fragColor;	


void main()
{											  
	fragColor = vertexColor ;
}
```



### Uniform

Uniform是一种从CPU中的应用向GPU中的着色器发送数据的方式，通常在顶点着色器和片段着色器中使用，用来储存一些不变的变量，类似const，Uniform变量是**全局可读**变量，它可以被当前着色器程序的任意着色器在任意阶段访问，我们可以在一个着色器中添加`uniform`关键字至类型和变量名前来声明一个GLSL的uniform变量

```
#version 330 core

out vec4 FragColor;

uniform vec4 uniformColor; //uniform变量

void main()
{
	FragColor = uniformColor;
}
```

我们需要手动输入uniform变量，在源程序中，用glUseProgram启用了某个着色器程序之后，要用[gluniform()](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glUniform.xhtml)函数给每个unifom变量关联数据

```
glUseProgram(shaderProgram);
GLint location = glGetUniformLocation(shaderProgram, "uniformColor");
glUniform4f(location, 1.0f, 0.0f, 0.0f, 1.0f); //red
```



# 着色器类

我们可以定义自己的着色器类来简化编写、编译、管理着色器的过程

```
//Shader.h

#pragma once
#include <string>

class Shader
{
public:
	Shader(const char* vertexPath, const char* fragmentPath);

	std::string vertexString;
	std::string fragmentString;

	const char* vertexSource;
	const char* fragmentSource;
	unsigned int ID; //shader program ID
	void use();
private:
	//check errors
	void checkCompileErrors(unsigned int ID, std::string Type);

};

```

```
#include "Shader.h"
#include <glad/glad.h>
#include <GLFW/glfw3.h>
#include <iostream>
#include<fstream>
#include <sstream>
#include <string>
using namespace std;

Shader::Shader(const char* vertexPath, const char* fragmentPath) {
	ifstream vertexFile;
	ifstream fragmentFile;
	stringstream vertexSStream;
	stringstream fragmentSStream;

	vertexFile.open(vertexPath);
	fragmentFile.open(fragmentPath);
	vertexFile.exceptions(ifstream::failbit || ifstream::badbit);
	fragmentFile.exceptions(ifstream::failbit || ifstream::badbit);
	
	try {
		if (!vertexFile.is_open()) {
			throw exception("open file error");
		}
		vertexSStream << vertexFile.rdbuf();
		fragmentSStream << fragmentFile.rdbuf();

		vertexString = vertexSStream.str();
		fragmentString = fragmentSStream.str();

		vertexSource = vertexString.c_str();
		fragmentSource = fragmentString.c_str();

		unsigned int vertex, fragment;
		vertex = glCreateShader(GL_VERTEX_SHADER);
		glShaderSource(vertex, 1, &vertexSource, NULL);
		glCompileShader(vertex);
		checkCompileErrors(vertex, "VERTEX");

		fragment = glCreateShader(GL_FRAGMENT_SHADER);
		glShaderSource(fragment, 1, &fragmentSource, NULL);
		glCompileShader(fragment);
		checkCompileErrors(fragment, "FRAGMENT");

		ID = glCreateProgram();
		glAttachShader(ID, vertex);
		glAttachShader(ID, fragment);
		glLinkProgram(ID);
		checkCompileErrors(ID, "PROGRAM");

		glDeleteShader(vertex);
		glDeleteShader(fragment);

	}
	catch (const std::exception& ex) {
		cout << ex.what();
	}

}

void Shader::use() {
	glUseProgram(ID);

}

//check errors
void Shader::checkCompileErrors(unsigned int ID, std::string Type) {
	int success;
	char infoLog[512];

	if (Type != "PROGRAM") {

		glGetShaderiv(ID, GL_COMPILE_STATUS, &success);

		if (!success) {
			glGetShaderInfoLog(ID, 512, NULL, infoLog);
			cout << "shader compile error" << infoLog << endl;
		}
	}
	else {
		glGetProgramiv(ID, GL_LINK_STATUS, &success);
		if (!success) {
			glGetProgramInfoLog(ID, 512, NULL, infoLog);
			cout << "program compile error" << infoLog << endl;

		}
	}
}
```

# 纹理



# 纹理坐标

纹理坐标看起来就像这样：

```
float texCoords[] = {
    0.0f, 0.0f, // 左下角
    1.0f, 0.0f, // 右下角
    0.5f, 1.0f // 上中
};
```

我们用纹理坐标获取纹理颜色，我们需要指定三角形的顶点对应的纹理的哪一部分，这样每个顶点就会关联着一个纹理坐标，用来标明该从纹理图像的哪个部分采样，我们只需要向顶点着色器提供顶点的纹理坐标，接下来它们会被传入片段着色器中进行片段插值

<!-- more -->

![](https://learnopengl-cn.github.io/img/01/06/tex_coords.png)

# 纹理环绕方式



纹理坐标的范围通常是(0,0)到(1,1)，如果我们把纹理坐标设置在范围之外时，OpenGL为我们提供了一些选项来设置范围外显示的图像

| 环绕方式           | 描述                                                         |
| :----------------- | :----------------------------------------------------------- |
| GL_REPEAT          | 对纹理的默认行为。重复纹理图像。                             |
| GL_MIRRORED_REPEAT | 和GL_REPEAT一样，但每次重复图片是镜像放置的。                |
| GL_CLAMP_TO_EDGE   | 纹理坐标会被约束在0到1之间，超出的部分会重复纹理坐标的边缘，产生一种边缘被拉伸的效果。 |
| GL_CLAMP_TO_BORDER | 超出的坐标为用户指定的边缘颜色。                             |

当纹理坐标超出默认范围时，每个选项都有不同的视觉效果输出。我们来看看这些纹理图像的例子：

![img](https://learnopengl-cn.github.io/img/01/06/texture_wrapping.png)

OpenGL提供了[`glTexParameter`](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glTexParameter.xhtml)函数帮助我们设置这些选项

# 纹理过滤

纹理坐标可以是任意浮点数，不依赖与分辨率，当图像放大或缩小时，纹理坐标的中心不一定正对着像素的中心，因此在贴图时会产生一定的偏差，这时我们就需要在纹理映射的过程中进行一定的处理，这就是纹理过滤

GL_NEAREST（也叫邻近过滤，Nearest Neighbor Filtering）是OpenGL默认的纹理过滤方式。当设置为GL_NEAREST的时候，OpenGL会选择中心点最接近纹理坐标的那个像素。下图中你可以看到四个像素，加号代表纹理坐标。左上角那个纹理像素的中心距离纹理坐标最近，所以它会被选择为样本颜色：

![img](https://learnopengl-cn.github.io/img/01/06/filter_nearest.png)

GL_LINEAR（也叫(双)线性过滤，(Bi)linear Filtering）它会基于纹理坐标附近的纹理像素，计算出一个插值，近似出这些纹理像素之间的颜色。一个纹理像素的中心距离纹理坐标越近，那么这个纹理像素的颜色对最终的样本颜色的贡献越大。下图中你可以看到返回的颜色是邻近像素的混合色：

![img](https://learnopengl-cn.github.io/img/01/06/filter_linear.png)

GL_NEAREST会产生**颗粒状**的图案，而GL_LINEAR能够产生更**平滑**的图案，也更加符合真实世界的输出。

> 多级渐远纹理
>
> 在纹理缩放的过程中还有一种常用的技术：多级渐远纹理，多级渐远纹理技术将原纹理提前用滤波处理来得到更小的图像，形成一个图像金字塔，每一层都是对上一层直接采样的结果。在实时运行时，就可以快速得到像素的颜色



OpenGL提供了[`glTexParameter()`](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glTexParameter.xhtml)函数帮助我们设置这些选项

# 加载和创建纹理

`stb_image.h`是[Sean Barrett](https://github.com/nothings)的一个非常流行的单头文件图像加载库，它能够加载大部分流行的文件格式，并且能够很简单得整合到你的工程之中。

在程序开头，包含加载纹理所用的头文件

```
#define STB_IMAGE_IMPLEMENTATION
#include "stb_image.h"
```



生成一个纹理的过程应该看起来像这样：

```
unsigned int texture;
glGenTextures(1, &texture);

glBindTexture(GL_TEXTURE_2D, texture);

// 为当前绑定的纹理对象设置环绕、过滤方式
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);   
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);

// 加载并生成纹理
int width, height, nrChannels;
unsigned char *data = stbi_load("container.jpg", &width, &height, &nrChannels, 0);
if (data)
{
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
    glGenerateMipmap(GL_TEXTURE_2D);
}
else
{
    std::cout << "Failed to load texture" << std::endl;
}

stbi_image_free(data);
```

- 首先我们用glGenTextures，glBindTextures创建并绑定纹理对象
- 接着用glTexParameter设置纹理的环绕方式和过滤方式
- 用stbi_load函数加载图片数据
- 使用前面载入的图片数据，通过glTexImage2D来生成纹理
- 为当前绑定的纹理自动生成多级渐远纹理
- 释放图像内存

# 应用纹理

我们需要告知OpenGL如何采样纹理，所以我们必须使用纹理坐标更新顶点数据：

```
float vertices[] = {
//     ---- 位置 ----       ---- 颜色 ----     - 纹理坐标 -
     0.5f,  0.5f, 0.0f,   1.0f, 0.0f, 0.0f,   1.0f, 1.0f,   // 右上
     0.5f, -0.5f, 0.0f,   0.0f, 1.0f, 0.0f,   1.0f, 0.0f,   // 右下
    -0.5f, -0.5f, 0.0f,   0.0f, 0.0f, 1.0f,   0.0f, 0.0f,   // 左下
    -0.5f,  0.5f, 0.0f,   1.0f, 1.0f, 0.0f,   0.0f, 1.0f    // 左上
};
```

随后，我们必须告诉OpenGL我们新的顶点格式

```
...
glVertexAttribPointer(2, 2, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void*)(6 * sizeof(float)));
glEnableVertexAttribArray(2);
...
```
在顶点着色器和片段着色器中设置属性

```
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aColor;
layout (location = 2) in vec2 aTexCoord;//纹理坐标

out vec3 ourColor;
out vec2 TexCoord;

void main()
{
    gl_Position = vec4(aPos, 1.0);
    ourColor = aColor;
    TexCoord = aTexCoord;
}
```

```
#version 330 core
out vec4 FragColor;

in vec3 ourColor;
in vec2 TexCoord;

uniform sampler2D ourTexture;//采样器

void main()
{
    FragColor = texture(ourTexture, TexCoord);
}
```

我们可以简单声明一个`uniform sampler2D`把一个纹理添加到片段着色器中

我们使用GLSL内建的texture函数来采样纹理的颜色，它第一个参数是纹理采样器，第二个参数是对应的纹理坐标。texture函数会使用之前设置的纹理参数对相应的颜色值进行采样。这个片段着色器的输出就是纹理的（插值）纹理坐标上的(过滤后的)颜色。

# 纹理单元

一个纹理的位置值通常称为一个纹理单元(Texture Unit)。一个纹理的默认纹理单元是0，它是**默认的激活纹理单元**，所以前面部分我们没有分配一个位置值。

纹理单元的主要目的是让我们在着色器中可以使用多个的纹理。通过把纹理单元赋值给采样器，我们可以一次绑定多个纹理，只要我们首先激活对应的纹理单元。就像glBindTexture一样，我们可以使用glActiveTexture激活纹理单元，传入我们需要使用的纹理单元：

```
glActiveTexture(GL_TEXTURE0); // 在绑定纹理之前先激活纹理单元
glBindTexture(GL_TEXTURE_2D, texture);
```

激活纹理单元之后，接下来的glBindTexture函数调用会绑定这个纹理到当前激活的纹理单元，**纹理单元GL_TEXTURE0默认总是被激活**，所以我们在前面的例子里当我们使用`glBindTexture`的时候，无需激活任何纹理单元。

我们仍然需要编辑片段着色器来接收另一个采样器。这应该相对来说非常直接了：

```
#version 330 core
...

uniform sampler2D texture1;
uniform sampler2D texture2;

void main()
{
    FragColor = mix(texture(texture1, TexCoord), texture(texture2, TexCoord), 0.2);
}
```

为了使用第二个纹理（以及第一个），我们必须改变一点渲染流程，先绑定两个纹理到对应的纹理单元，然后定义哪个uniform采样器对应哪个纹理单元：

```
glActiveTexture(GL_TEXTURE0);
glBindTexture(GL_TEXTURE_2D, texture1);
glActiveTexture(GL_TEXTURE1);
glBindTexture(GL_TEXTURE_2D, texture2);

glBindVertexArray(VAO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```

我们还要通过使用glUniform1i设置每个采样器的方式告诉OpenGL每个着色器采样器属于哪个纹理单元。我们只需要设置一次即可，所以这个会放在渲染循环的前面：

```
testShader.use(); 
glUniform1i(glGetUniformLocation(testShader->ID, "texture1"), 0);
glUniform1i(glGetUniformLocation(testShader->ID, "texture2"), 1);

while(...) 
{
    [...]
}
```

通过使用glUniform1i设置采样器，我们保证了每个uniform采样器对应着正确的纹理单元

**END**






液✌




Inspired by 

http://zwqxin.com/archives/shaderglsl/communication-between-opengl-glsl-2.html

https://blog.csdn.net/candycat1992/article/details/8830894

[https://learnopengl-cn.github.io/01%20Getting%20started/05%20Shaders/#_4](https://learnopengl-cn.github.io/01 Getting started/05 Shaders/#_4)

