---
title: 基于Qt的OpenGL学习（2）—— 着色器
categories:
  - Qt
tags:
  - Qt
  - OpenGL
date: 2019-08-08 15:46:22
---

<Excerpt in index | 摘要> 
要学习OpenGL的话，强烈安利这个教程JoeyDeVries的[learnopengl](https://learnopengl-cn.github.io/)，这里是中文翻译好的版本。教程中使用OpenGL是通过[GLFW](http://www.glfw.org/download.html)这个库，而在Qt中对OpenGL封装得很好，并且和GUI以及IO相关的处理Qt更便捷，学习起来更轻松。这里就对每篇教程，在Qt在分别直接使用OpenGL的函数和Qt封装好的类以作对比。
教程中使用的OpenGL版本为3.3，在Qt中需要使用此版本的OpenGL只需要继承类`QOpenGLFunctions_3_3_Core`即可。如果为了在不同设备上都能用OpenGL的话，Qt提供了类`QOpenGLFunctions`，这个类包含了大部分公共的函数，可能会有个别函数不能用。 <!-- more -->
<The rest of contents | 余下全文>

## 对比说明
### 教程地址
[原教程地址](https://learnopengl-cn.github.io/01%20Getting%20started/05%20Shaders/)，相关知识可以点击链接学习。
[我的工程地址](https://github.com/Longxr/QOpenGLWidgetTest)，准备后期每篇教程一个commit，可以切换着看，查看本篇代码 `git checkout 4623abc`，喜欢就点个Star吧~

### 不同点 (仅列出新增)
1. 我的代码实现的是原教程中练习2的三角形沿着X轴水平移动
2. 原教程是一个while循环不停重绘，Qt中的`paintGL`则是在需要时才重绘，因此采用QTimer定时器触发重绘刷新。

## 运行结果
![运行结果](https://upload-images.jianshu.io/upload_images/2756183-121cf4606a7e32c0.gif?imageMogr2/auto-orient/strip)

## 使用OpenGL函数版
### CoreFunctionWidget.h
```
#ifndef COREFUNCTIONWIDGET_H
#define COREFUNCTIONWIDGET_H

#include <QOpenGLWidget>
#include <QOpenGLExtraFunctions>
#include <QOpenGLFunctions_3_3_Core>
#include <QOpenGLShader>
#include <QOpenGLShaderProgram>

class CoreFunctionWidget : public QOpenGLWidget
                           , protected /*QOpenGLExtraFunctions*/QOpenGLFunctions_3_3_Core
{
    Q_OBJECT
public:
    explicit CoreFunctionWidget(QWidget *parent = nullptr);
    ~CoreFunctionWidget();

protected:
    virtual void initializeGL();
    virtual void resizeGL(int w, int h);
    virtual void paintGL();

private:
    QOpenGLShaderProgram shaderProgram;

    QTimer* m_pTimer = nullptr;
    float m_uniformValue = 0.0f;
};

#endif // COREFUNCTIONWIDGET_H
```

### CoreFunctionWidget.cpp
```
#include "CoreFunctionWidget.h"
#include <QDebug>
#include <QTimer>

static GLuint VBO, VAO, EBO;

CoreFunctionWidget::CoreFunctionWidget(QWidget *parent) : QOpenGLWidget(parent)
{
    m_pTimer = new QTimer(this);
    m_pTimer->setInterval(200);

    connect(m_pTimer, &QTimer::timeout, this, [=]{
        m_uniformValue += 0.1f;

        if (m_uniformValue > 1.5f) {
            m_uniformValue = -1.5f;
        }

        update();
    });

    m_pTimer->start();
}

CoreFunctionWidget::~CoreFunctionWidget()
{
    glDeleteVertexArrays(1, &VAO);
    glDeleteBuffers(1, &VBO);
//    glDeleteBuffers(1, &EBO);
}

void CoreFunctionWidget::initializeGL(){
    this->initializeOpenGLFunctions();

    bool success = shaderProgram.addShaderFromSourceFile(QOpenGLShader::Vertex, ":/colortriangle.vert");
    if (!success) {
        qDebug() << "shaderProgram addShaderFromSourceFile failed!" << shaderProgram.log();
        return;
    }

    success = shaderProgram.addShaderFromSourceFile(QOpenGLShader::Fragment, ":/colortriangle.frag");
    if (!success) {
        qDebug() << "shaderProgram addShaderFromSourceFile failed!" << shaderProgram.log();
        return;
    }

    success = shaderProgram.link();
    if(!success) {
        qDebug() << "shaderProgram link failed!" << shaderProgram.log();
    }

    //VAO，VBO数据部分
    float vertices[] = {
        // positions         // colors
        0.5f, -0.5f, 0.0f,  1.0f, 0.0f, 0.0f,  // bottom right
        -0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f,  // bottom left
        0.0f,  0.5f, 0.0f,  0.0f, 0.0f, 1.0f   // top
    };
//    unsigned int indices[] = {  // note that we start from 0!
//        0, 1, 3,  // first Triangle
//        1, 2, 3   // second Triangle
//    };

    glGenVertexArrays(1, &VAO);
    glGenBuffers(1, &VBO);
//    glGenBuffers(1, &EBO);
    // bind the Vertex Array Object first, then bind and set vertex buffer(s), and then configure vertex attributes(s).
    glBindVertexArray(VAO);

    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);  //顶点数据复制到缓冲

//    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
//    glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);

    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(GLfloat), (void*)0);
    glEnableVertexAttribArray(0);
    glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(GLfloat), (GLvoid*)(3*sizeof(GLfloat)));
    glEnableVertexAttribArray(1);

    glBindBuffer(GL_ARRAY_BUFFER, 0);//取消VBO的绑定, glVertexAttribPointer已经把顶点属性关联到顶点缓冲对象了

//    remember: do NOT unbind the EBO while a VAO is active as the bound element buffer object IS stored in the VAO; keep the EBO bound.
//    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, 0);

//    You can unbind the VAO afterwards so other VAO calls won't accidentally modify this VAO, but this rarely happens. Modifying other
//    VAOs requires a call to glBindVertexArray anyways so we generally don't unbind VAOs (nor VBOs) when it's not directly necessary.
    glBindVertexArray(0);   //取消VAO绑定

    //线框模式，QOpenGLExtraFunctions没这函数, 3_3_Core有
//    glPolygonMode(GL_FRONT_AND_BACK, GL_LINE);
}

void CoreFunctionWidget::resizeGL(int w, int h){
    glViewport(0, 0, w, h);
}

void CoreFunctionWidget::paintGL(){
    glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
    glClear(GL_COLOR_BUFFER_BIT);

    shaderProgram.bind();
    {
        int xOffsetLocation = shaderProgram.uniformLocation("xOffset");
        glUniform1f(xOffsetLocation, m_uniformValue);

        // 绘制三角形
        glBindVertexArray(VAO);
        glDrawArrays(GL_TRIANGLES, 0, 3);

//        glBindVertexArray(VAO); // seeing as we only have a single VAO there's no need to bind it every time, but we'll do so to keep things a bit more organized
//        glDrawArrays(GL_TRIANGLES, 0, 6);
//        glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
    }
    shaderProgram.release();
}


```

## 使用Qt相关函数版
### QtFunctionWidget.h
```
#ifndef QTFUNCTIONWIDGET_H
#define QTFUNCTIONWIDGET_H

#include <QOpenGLWidget>


#include <QOpenGLWidget>
#include <QOpenGLShader>
#include <QOpenGLShaderProgram>
#include <QOpenGLFunctions>
#include <QOpenGLVertexArrayObject>
#include <QOpenGLBuffer>

class QtFunctionWidget : public QOpenGLWidget, protected QOpenGLFunctions
{
public:
    QtFunctionWidget(QWidget *parent = nullptr);
    ~QtFunctionWidget() Q_DECL_OVERRIDE;

protected:
    virtual void initializeGL() Q_DECL_OVERRIDE;
    virtual void resizeGL(int w, int h) Q_DECL_OVERRIDE;
    virtual void paintGL() Q_DECL_OVERRIDE;

private:
    QOpenGLShaderProgram shaderProgram;
    QOpenGLBuffer vbo, ebo;
    QOpenGLVertexArrayObject vao;

    QTimer* m_pTimer = nullptr;
    float m_uniformValue = 0.0f;
};

#endif // QTFUNCTIONWIDGET_H


```
### QtFunctionWidget.cpp
```
#include "QtFunctionWidget.h"
#include <QDebug>
#include <QTimer>

QtFunctionWidget::QtFunctionWidget(QWidget *parent) : QOpenGLWidget (parent),
    vbo(QOpenGLBuffer::VertexBuffer),
    ebo(QOpenGLBuffer::IndexBuffer)
{
    m_pTimer = new QTimer(this);
    m_pTimer->setInterval(200);

    connect(m_pTimer, &QTimer::timeout, this, [=]{
        m_uniformValue += 0.1f;

        if (m_uniformValue > 1.5f) {
            m_uniformValue = -1.5f;
        }

        update();
    });

    m_pTimer->start();
}

QtFunctionWidget::~QtFunctionWidget(){
    makeCurrent();

    vbo.destroy();
    ebo.destroy();
    vao.destroy();

    doneCurrent();
}

void QtFunctionWidget::initializeGL(){
    this->initializeOpenGLFunctions();

    bool success = shaderProgram.addShaderFromSourceFile(QOpenGLShader::Vertex, ":/colortriangle.vert");
    if (!success) {
        qDebug() << "shaderProgram addShaderFromSourceFile failed!" << shaderProgram.log();
        return;
    }

    success = shaderProgram.addShaderFromSourceFile(QOpenGLShader::Fragment, ":/colortriangle.frag");
    if (!success) {
        qDebug() << "shaderProgram addShaderFromSourceFile failed!" << shaderProgram.log();
        return;
    }

    success = shaderProgram.link();
    if(!success) {
        qDebug() << "shaderProgram link failed!" << shaderProgram.log();
    }

    //VAO，VBO数据部分
    GLfloat vertices[] = {
        // positions         // colors
        0.5f, -0.5f, 0.0f,  1.0f, 0.0f, 0.0f,  // bottom right
        -0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f,  // bottom left
        0.0f,  0.5f, 0.0f,  0.0f, 0.0f, 1.0f   // top
    };
//    unsigned int indices[] = {  // note that we start from 0!
//        0, 1, 3,  // first Triangle
//        1, 2, 3   // second Triangle
//    };

    QOpenGLVertexArrayObject::Binder vaoBind(&vao);

    vbo.create();
    vbo.bind();
    vbo.allocate(vertices, sizeof(vertices));

//    ebo.create();
//    ebo.bind();
//    ebo.allocate(indices, sizeof(indices));

    int attr = -1;
    attr = shaderProgram.attributeLocation("aPos");
    shaderProgram.setAttributeBuffer(attr, GL_FLOAT, 0, 3, sizeof(GLfloat) * 6);
    shaderProgram.enableAttributeArray(attr);

    attr = shaderProgram.attributeLocation("aColor");
    shaderProgram.setAttributeBuffer(attr, GL_FLOAT, sizeof(GLfloat) * 3, 3, sizeof(GLfloat) * 6);
    shaderProgram.enableAttributeArray(attr);

    vbo.release();
//    remember: do NOT unbind the EBO while a VAO is active as the bound element buffer object IS stored in the VAO; keep the EBO bound.
//    ebo.release();

}

void QtFunctionWidget::resizeGL(int w, int h){
    glViewport(0, 0, w, h);
}

void QtFunctionWidget::paintGL(){
    glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
    glClear(GL_COLOR_BUFFER_BIT);

    shaderProgram.bind();
    {
        shaderProgram.setUniformValue("xOffset", m_uniformValue);

        QOpenGLVertexArrayObject::Binder vaoBind(&vao);
        glDrawArrays(GL_TRIANGLES, 0, 3);

//        glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
    }
    shaderProgram.release();
}

```

## GLSL
### colortriangle.vert
```
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aColor;
 
out vec3 ourColor;
uniform float xOffset;
 
void main(){
  gl_Position = vec4(aPos.x + xOffset, aPos.y, aPos.z, 1.0);
  ourColor = aColor;
}
```

### colortriangle.frag
```
#version 330 core
out vec4 FragColor;
in vec3 ourColor;
 
void main(void)
{
    FragColor = vec4(ourColor, 1.0f);
}
```

## main.cpp
```
#include <QApplication>
#include "MainWindow.h"
#include "QtFunctionWidget.h"
#include "CoreFunctionWidget.h"

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);

//    MainWindow w;
    QtFunctionWidget w1;
    CoreFunctionWidget w2;

    w1.setWindowTitle(QObject::tr("QtFunction"));
    w2.setWindowTitle(QObject::tr("CoreFunction"));

    w1.show();
    w2.show();

    return a.exec();
}

```
