---
layout: post
title:  "hello 三角形"
categories: opengl
tags:  Qt C++
---

* content
{:toc}

hello 三角形

<!--excerpt-->
## OpenGL简介
一般它被认为是一个API(Application Programming Interface, 应用程序编程接口)，包含了一系列可以操作图形、图像的函数。然而，OpenGL本身并不是一个API，它仅仅是一个由Khronos组织制定并维护的规范(Specification)。
接下来的文档和代码都是基于OpenGL3.3.

## 技术选型
接下来这系列的文章，都会使用Qt5+OpenGL3.3+.目前只针对客户端，为什么选用Qt，一是Qt跨平台，画见面非常简单；二是Qt自从5开始对OpenGL的支持非常好。
后面有机会会更新移动端的版本。

## 参考教程

- 实战

参考JoeyDeVries的
[learn_opengl](https://learnopengl-cn.github.io/01%20Getting%20started/01%20OpenGL/)。秒杀国内99.9的教材。

- 原理

图形学算法参考：[GAMES101-现代计算机图形学入门](https://www.bilibili.com/video/BV1X7411F744?p=13).
闫令琪博士，加州大学圣芭芭拉分校助理教授，博士生导师，于2013年获清华大学学士学位，2018 年获加州大学伯克利分校博士学位。
课程通俗易懂。就是一个宝！！！



## 图形渲染管线
我们看到的窗口和屏幕是2D的，但是在OpenGL中，所有的事物是三维的。所以OpenGL的大部分工作是把3D坐标转化成2D。3D->2D这个处理过程是由**图像渲染管线**（Graphics Pipeline，大多译为管线，实际上指的是一堆原始图形数据途经一个输送管道，期间经过各种变化处理最终出现在屏幕的过程）来处理的。可以分为如下两个部分：

1. 3D坐标转换为2D坐标
2. 2D坐标转变为实际的有颜色的像素

### 着色器
为了更清楚的了解图形渲染管线的过程，这里解释一下**着色器**。
图形渲染管线可以被划分为几个阶段，每个阶段将会把前一个阶段的输出作为输入。所有这些阶段都是高度专门化的（它们都有一个特定的函数），并且很容易并行执行。正是由于它们具有并行执行的特性，当今大多数显卡都有成千上万的小处理核心，它们在GPU上为每一个（渲染管线）阶段运行各自的小程序，从而在图形渲染管线中快速处理你的数据。这些小程序叫做**着色器(Shader)**。OpenGL着色器是用OpenGL着色器语言(OpenGL Shading Language, GLSL)写成的.

图形渲染管线的每个阶段的抽象展示。蓝色部分代表的是我们可以注入自定义的着色器的部分:
![图形渲染管线的每个阶段的抽象展示](https://learnopengl-cn.github.io/img/01/04/pipeline.png)

- 顶点着色器
把单独的顶点作为输入，顶点着色器的主要目的是把3d坐标转换成另外一种3d坐标。
- 形状（图元）装配
将顶点着色器的输出的所有顶点作为输入，把输入的点组合成图元的形状。（比如点、线、三角形等）
- 几何着色器
把图元装配阶段的输出作为输入，产生一些新的点构造成新的图元，生成了一些其他形状。
- 光栅化
把图元映射为屏幕上的像素，生产供片段着色器使用的片段。并进行裁切。裁切会丢弃超出你的视图以外的所有像素，用来提升执行效率。
- 片段着色器
用来计算像素的最终颜色，光照、阴影、颜色等效果都是片段着色器实现的。**OpenGL中的一个片段是OpenGL渲染一个像素所需的所有数据。**
- 测试与混合
检测片段的深度，决定是否丢弃被遮挡的像素。同时检查alpha值（alpha值定义了一个物体的透明度）并对物体进行混合(Blend)。所以，即使在片段着色器中计算出来了一个像素输出的颜色，在渲染多个三角形的时候最后的像素颜色也可能完全不同。

## hello 三角形

- triangle_wgt.h

```cpp
#include <QOpenGLWidget>
#include <QOpenGLFunctions_3_3_Core>
#include <iostream>
#include <QVector>
class triangle_wgt : public QOpenGLWidget
{
	Q_OBJECT

public:
	triangle_wgt();
	~triangle_wgt();
protected:
	virtual void initializeGL() override;
	virtual void resizeGL(int w, int h) override;
	virtual void paintGL() override;

private:
	GLuint shader_program_;
	QOpenGLFunctions_3_3_Core*  core_;
	GLuint VBO, VAO;

	QVector<GLfloat> verties_;
};
```
-triangle_wgt.cpp

```cpp

#include "triangle_wgt.h"
#include <QDebug>
const char* vertexShaderSource = 
		"#version 330 core\n"
		"layout(location = 0) in vec3 aPos;\n"
		"void main(){\n"
		"  gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
		"}\n\0";
const char* fragmentShaderSource =
		"#version 330 core\n"
		"out vec4 FragColor;\n"
		"void main(){\n"
		"   FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);\n"
		"}\n\0";


triangle_wgt::triangle_wgt()
	: QOpenGLWidget()
{
	this->setGeometry(QRect(100,100,800,600));
}

triangle_wgt::~triangle_wgt()
{
}

void triangle_wgt::initializeGL() {

	core_ = QOpenGLContext::currentContext()
		->versionFunctions<QOpenGLFunctions_3_3_Core>();

	// 顶点着色器 -------------------------------------
	//创建着色器对象
	auto vert_shader = core_->glCreateShader(GL_VERTEX_SHADER);
	//把着色器源码附加到着色器对象
	core_->glShaderSource(vert_shader, 1, &vertexShaderSource, NULL);
	//编译
	core_->glCompileShader(vert_shader);

	//检测编译是否成功
	int success;
	char info_log[512];
	core_->glGetShaderiv(vert_shader, GL_COMPILE_STATUS, &success);
	if (!success) {
		core_->glGetShaderInfoLog(vert_shader, 512, NULL, info_log);
		qDebug() << "shader complie error:" << info_log;
	}
	// 片段着色器 -------------------------------------

	auto frag_shader = core_->glCreateShader(GL_FRAGMENT_SHADER);
	//把着色器源码附加到着色器对象
	core_->glShaderSource(frag_shader, 1, &fragmentShaderSource, NULL);
	//编译
	core_->glCompileShader(frag_shader);

	core_->glGetShaderiv(frag_shader, GL_COMPILE_STATUS, &success);
	if (!success) {
		core_->glGetShaderInfoLog(frag_shader, 512, NULL, info_log);
		qDebug() << "shader complie error:" << info_log;
	}
	// 着色器程序
	shader_program_ = core_->glCreateProgram();
	core_->glAttachShader(shader_program_, vert_shader);
	core_->glAttachShader(shader_program_, frag_shader);
	core_->glLinkProgram(shader_program_);
	core_->glGetShaderiv(shader_program_, GL_LINK_STATUS, &success);
	if (!success) {
		core_->glGetShaderInfoLog(shader_program_, 512, NULL, info_log);
		qDebug() << "shader program error:" << info_log;
	}
	core_->glDeleteShader(vert_shader);
	core_->glDeleteShader(frag_shader);

	verties_.clear();
	verties_ << -0.5f << -0.5f << 0.0f;
	verties_ << 0.5f << -0.5f << 0.0f;
	verties_ << 0.0f << 0.5f << 0.0f;

	//生产1个顶点VAO对象
	core_->glGenVertexArrays(1, &VAO);
	//vbo
	core_->glGenBuffers(1, &VBO);
	//绑定vao
	core_->glBindVertexArray(VAO);
	//开始复制顶点数据到opengl
	core_->glBindBuffer(GL_ARRAY_BUFFER, VBO);

	core_->glBufferData(GL_ARRAY_BUFFER, verties_.count()*sizeof(GLfloat),
		verties_.data(), GL_STATIC_DRAW);

	core_->glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(GLfloat), (void*)0);
	core_->glEnableVertexAttribArray(0);

	//解绑定
	core_->glBindBuffer(GL_ARRAY_BUFFER, 0);
	core_->glBindVertexArray(0);
}

void triangle_wgt::resizeGL(int w, int h) {

	core_->glViewport(0, 0, w, h);
}

void triangle_wgt::paintGL() {
	core_->glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
	core_->glClear(GL_COLOR_BUFFER_BIT);

	core_->glUseProgram(shader_program_);
	core_->glBindVertexArray(VAO);
	core_->glDrawArrays(GL_TRIANGLES, 0, 3);
	core_->glUseProgram(0);
}
```