---
layout: post
title:  "OpenGL-03-纹理"
categories: opengl
tags:  Qt C++
---


* content
{:toc}

纹理

<!--excerpt-->

## 概述
纹理是一个2D图片（甚至也有1D和3D的纹理），它可以用来添加物体的细节；你可以想象纹理是一张绘有砖块的纸，无缝折叠贴合到你的3D的房子上，这样你的房子看起来就像有砖墙外表了。因为我们可以在一张图片上插入非常多的细节，这样就可以让物体非常精细而不用指定额外的顶点。

## 纹理的环绕方式

纹理坐标的范围通常是从(0, 0)到(1, 1)，那如果我们把纹理坐标设置在范围之外会发生什么？OpenGL默认的行为是重复这个纹理图像（我们基本上忽略浮点纹理坐标的整数部分），但OpenGL提供了更多的选择：

|环绕方式|	描述|
|----|----|
|GL_REPEAT|	对纹理的默认行为。重复纹理图像。|
|GL_MIRRORED_REPEAT	|和GL_REPEAT一样，但每次重复图片是镜像放置的。|
|GL_CLAMP_TO_EDGE	|纹理坐标会被约束在0到1之间，超出的部分会重复纹理坐标的边缘，产生一种边缘被拉伸的效果。|
|GL_CLAMP_TO_BORDER|	超出的坐标为用户指定的边缘颜色。|
![](https://learnopengl-cn.github.io/img/01/06/texture_wrapping.png)

更多详细的参考 [纹理详情](https://learnopengl-cn.github.io/01%20Getting%20started/06%20Textures/)

## 一个封装的纹理类

```cpp
class GLTexture
{
public:
	GLTexture();
	~GLTexture();
	void generate(const QString& file);
	void bind() const;
	QOpenGLTexture* get_texture();

	//Format of texture object
	QOpenGLTexture::TextureFormat internal_format;
	QOpenGLTexture::WrapMode wrap_s;
	QOpenGLTexture::WrapMode wrap_t;
	QOpenGLTexture::Filter filter_min;
	QOpenGLTexture::Filter filter_max;
	
private:
	QOpenGLTexture* texture_;
};
GLTexture::GLTexture() :texture_(nullptr)
, internal_format(QOpenGLTexture::RGBFormat)
, wrap_s(QOpenGLTexture::Repeat)
, wrap_t(QOpenGLTexture::Repeat)
, filter_min(QOpenGLTexture::Linear)
, filter_max(QOpenGLTexture::Linear)
{

}

GLTexture::~GLTexture()
{

}

void GLTexture::generate(const QString& file)
{
	//直接生成绑定一个2d纹理, 并生成多级纹理MipMaps
	texture_ = new QOpenGLTexture(QOpenGLTexture::Target2D);
	texture_->setFormat(internal_format);
	texture_->setData(QImage(file).mirrored(), QOpenGLTexture::GenerateMipMaps);

	texture_->setWrapMode(QOpenGLTexture::DirectionS, wrap_s);
	texture_->setWrapMode(QOpenGLTexture::DirectionT, wrap_t);

	texture_->setMinificationFilter(filter_min);
	texture_->setMagnificationFilter(filter_max);
}

void GLTexture::bind() const
{
	texture_->bind();
}

QOpenGLTexture* GLTexture::get_texture()
{
	return texture_;
}


```

### 使用方式
```cpp
		//shader.use();
	resource_manager->shader("shader").use();
	resource_manager->texture("texture").bind();
	core->glBindVertexArray(VAO);
	//core->glDrawArrays(GL_TRIANGLES, 0, 3);

	core->glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```