---
layout: post
title:  "OpenGL-02-着色器"
categories: opengl
tags:  Qt C++
---


* content
{:toc}

着色器

<!--excerpt-->

## 概述

着色器(Shader)是运行在GPU上的小程序。这些小程序为图形渲染管线的某个特定部分而运行。从基本意义上来说，着色器只是一种把输入转化为输出的程序。着色器也是一种非常独立的程序，因为它们之间不能相互通信；它们之间唯一的沟通只有通过输入和输出。

## GLSL
着色器使用GLSL来实现输入和输出，类似于C语言，着色器的结构如下：

```cpp
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aColor;
 
out vec3 ourColor;
 
void main(){
  gl_Position = vec4(aPos, 1.0f);
  ourColor = aColor;
}
```
我们在开发中最多使用的是顶点着色器和片段着色器。如：

*顶点着色器*
```cpp
#version 330 core
layout (location = 0) in vec3 aPos; // 位置变量的属性位置值为0

out vec4 vertexColor; // 为片段着色器指定一个颜色输出

void main()
{
    gl_Position = vec4(aPos, 1.0); // 注意我们如何把一个vec3作为vec4的构造器的参数
    vertexColor = vec4(0.5, 0.0, 0.0, 1.0); // 把输出变量设置为暗红色
}
```
*片段着色器*

```cpp
#version 330 core
out vec4 FragColor;

in vec4 vertexColor; // 从顶点着色器传来的输入变量（名称相同、类型相同）

void main()
{
    FragColor = vertexColor;
}
```

## 定制一个通用的着色器

```cpp
#ifndef GL_SHADER_H
#define GL_SHADER_H
#include <QString>
#include <QOpenGLShader>
#include <QOpenGLShaderProgram>
#include <QDebug>
class GLShader
{
public:
	GLShader(const QString& vert_path, const QString& frag_path);
	virtual ~GLShader();
	void use();
private:
	QOpenGLShaderProgram shader_program;
};
#endif //GL_SHADER_H


cpp
#include "gl_shader.h"

GLShader::GLShader(const QString& vert_path, const QString& frag_path)
{
	QOpenGLShader vert_shader(QOpenGLShader::Vertex);
	if (!vert_shader.compileSourceFile(vert_path)) {
		qDebug() << vert_shader.log() << endl;
	}
	QOpenGLShader frag_shader(QOpenGLShader::Fragment);
	if (!frag_shader.compileSourceFile(frag_path)) {
		qDebug() << frag_shader.log() << endl;
	}
	shader_program.addShader(&vert_shader);
	shader_program.addShader(&frag_shader);
	if (!shader_program.link()) {
		qDebug() << shader_program.log();
	}
}

GLShader::~GLShader()
{

}

void GLShader::use()
{
	shader_program.bind();
}

```