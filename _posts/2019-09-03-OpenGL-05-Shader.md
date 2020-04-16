---
layout: post
title: OpenGL-05-Shader
date: 2019-09-03 23:40:36
---

# 着色器
***

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

***

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

***

# 输入输出

GL3.x中，GLSL废弃了`attribute`，`varying`关键字，定义了`in` ，`out` 两个关键字来处理着色器的输入和输出

***

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

***

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

***

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

***

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

***

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



***

液✌

***


Inspired by 

http://zwqxin.com/archives/shaderglsl/communication-between-opengl-glsl-2.html

https://blog.csdn.net/candycat1992/article/details/8830894

[https://learnopengl-cn.github.io/01%20Getting%20started/05%20Shaders/#_4](https://learnopengl-cn.github.io/01 Getting started/05 Shaders/#_4)