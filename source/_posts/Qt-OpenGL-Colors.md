---
title: 基于Qt的OpenGL学习（7）—— 颜色
categories:
  - Qt
tags:
  - Qt
  - OpenGL
date: 2019-08-13 13:23:33
---

<Excerpt in index | 摘要> 
要学习OpenGL的话，强烈安利这个教程JoeyDeVries的[learnopengl](https://learnopengl-cn.github.io/)，这里是中文翻译好的版本。教程中使用OpenGL是通过[GLFW](http://www.glfw.org/download.html)这个库，而在Qt中对OpenGL封装得很好，并且和GUI以及IO相关的处理Qt更便捷，学习起来更轻松。这里就对每篇教程，在Qt在分别直接使用OpenGL的函数和Qt封装好的类以作对比。
教程中使用的OpenGL版本为3.3，在Qt中需要使用此版本的OpenGL只需要继承类`QOpenGLFunctions_3_3_Core`即可。如果为了在不同设备上都能用OpenGL的话，Qt提供了类`QOpenGLFunctions`，这个类包含了大部分公共的函数，可能会有个别函数不能用。 <!-- more -->
<The rest of contents | 余下全文>

## 对比说明
### 教程地址
[原教程地址](https://learnopengl-cn.github.io/02%20Lighting/01%20Colors/)，相关知识可以点击链接学习。
[我的工程地址](https://github.com/Longxr/QOpenGLWidgetTest)，每篇教程一个commit，可以切换着看，查看本篇代码 `git checkout 829430e`，喜欢就点个Star吧~

### 不同点 (仅列出新增)
1. 之前程序只有一个shaderProgram、VAO，直接用的shaderProgram的`setAttributeBuffer()`函数绑定VBO数据；现在有两个shaderProgram、VAO，不确定后面shaderProgram和VAO还是不是一对一的关系，还是改成默认的`glVertexAttribPointer()`函数了，两个都能用。（感觉之前的好像VAO没啥用啊，shaderprogram可以直接去绑定VBO）

## 运行结果
![运行结果](https://upload-images.jianshu.io/upload_images/2756183-7742167b01f93803.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 修改的文件
### QtFunctionWidget.h
```
#ifndef QTFUNCTIONWIDGET_H
#define QTFUNCTIONWIDGET_H

#include <QOpenGLWidget>
#include <QOpenGLShaderProgram>
#include <QOpenGLFunctions>
#include <QOpenGLVertexArrayObject>
#include <QOpenGLBuffer>
#include <QOpenGLTexture>

#include "Camera.h"

class QtFunctionWidget : public QOpenGLWidget, protected QOpenGLFunctions
{
public:
    QtFunctionWidget(QWidget *parent = nullptr);
    ~QtFunctionWidget() Q_DECL_OVERRIDE;

protected:
    virtual void initializeGL() Q_DECL_OVERRIDE;
    virtual void resizeGL(int w, int h) Q_DECL_OVERRIDE;
    virtual void paintGL() Q_DECL_OVERRIDE;

    void keyPressEvent(QKeyEvent *event) Q_DECL_OVERRIDE;
    void keyReleaseEvent(QKeyEvent *event) Q_DECL_OVERRIDE;
    void mousePressEvent(QMouseEvent *event) Q_DECL_OVERRIDE;
    void mouseReleaseEvent(QMouseEvent *event) Q_DECL_OVERRIDE;
    void mouseMoveEvent(QMouseEvent *event) Q_DECL_OVERRIDE;
    void wheelEvent(QWheelEvent *event) Q_DECL_OVERRIDE;

private:
    bool createShader();

private:
    QOpenGLShaderProgram lightingShader, lampShader;
    QOpenGLBuffer vbo;
    QOpenGLVertexArrayObject cubeVAO, lightVAO;

    QTimer* m_pTimer = nullptr;
    int     m_nTimeValue = 0;

    // camera
    std::unique_ptr<Camera> camera;
    bool m_bLeftPressed;
    QPoint m_lastPos;
};

#endif // QTFUNCTIONWIDGET_H
```
### QtFunctionWidget.cpp
```
#include "QtFunctionWidget.h"
#include <QDebug>
#include <QTimer>

// lighting
static QVector3D lightPos(1.2f, 1.0f, 2.0f);

QtFunctionWidget::QtFunctionWidget(QWidget *parent) : QOpenGLWidget (parent),
    vbo(QOpenGLBuffer::VertexBuffer)
{
    camera = std::make_unique<Camera>(QVector3D(3.0f, 0.0f, 10.0f));
    m_bLeftPressed = false;

    m_pTimer = new QTimer(this);
    connect(m_pTimer, &QTimer::timeout, this, [=]{
        m_nTimeValue += 1;
        update();
    });
    m_pTimer->start(40);
}

QtFunctionWidget::~QtFunctionWidget(){
    makeCurrent();

    lightVAO.destroy();
    cubeVAO.destroy();
    vbo.destroy();

    doneCurrent();
}

void QtFunctionWidget::initializeGL(){
    this->initializeOpenGLFunctions();

    createShader();

    //VAO，VBO data
    float vertices[] = {
        -0.5f, -0.5f, -0.5f,
         0.5f, -0.5f, -0.5f,
         0.5f,  0.5f, -0.5f,
         0.5f,  0.5f, -0.5f,
        -0.5f,  0.5f, -0.5f,
        -0.5f, -0.5f, -0.5f,

        -0.5f, -0.5f,  0.5f,
         0.5f, -0.5f,  0.5f,
         0.5f,  0.5f,  0.5f,
         0.5f,  0.5f,  0.5f,
        -0.5f,  0.5f,  0.5f,
        -0.5f, -0.5f,  0.5f,

        -0.5f,  0.5f,  0.5f,
        -0.5f,  0.5f, -0.5f,
        -0.5f, -0.5f, -0.5f,
        -0.5f, -0.5f, -0.5f,
        -0.5f, -0.5f,  0.5f,
        -0.5f,  0.5f,  0.5f,

         0.5f,  0.5f,  0.5f,
         0.5f,  0.5f, -0.5f,
         0.5f, -0.5f, -0.5f,
         0.5f, -0.5f, -0.5f,
         0.5f, -0.5f,  0.5f,
         0.5f,  0.5f,  0.5f,

        -0.5f, -0.5f, -0.5f,
         0.5f, -0.5f, -0.5f,
         0.5f, -0.5f,  0.5f,
         0.5f, -0.5f,  0.5f,
        -0.5f, -0.5f,  0.5f,
        -0.5f, -0.5f, -0.5f,

        -0.5f,  0.5f, -0.5f,
         0.5f,  0.5f, -0.5f,
         0.5f,  0.5f,  0.5f,
         0.5f,  0.5f,  0.5f,
        -0.5f,  0.5f,  0.5f,
        -0.5f,  0.5f, -0.5f,
    };

    vbo.create();
    vbo.bind();
    vbo.allocate(vertices, sizeof(vertices));

    {
        QOpenGLVertexArrayObject::Binder vaoBind(&cubeVAO);

//        position attribute
//        int attr = -1;
//        attr = lightingShader.attributeLocation("aPos");
//        lightingShader.setAttributeBuffer(attr, GL_FLOAT, 0, 3, sizeof(GLfloat) * 3);
//        lightingShader.enableAttributeArray(attr);
        glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
        glEnableVertexAttribArray(0);
    }

    {
        QOpenGLVertexArrayObject::Binder vaoBind(&lightVAO);

//        position attribute
//        int attr = -1;
//        attr = lampShader.attributeLocation("aPos");
//        lampShader.setAttributeBuffer(attr, GL_FLOAT, 0, 3, sizeof(GLfloat) * 3);
//        lampShader.enableAttributeArray(attr);
        glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
        glEnableVertexAttribArray(0);
    }


    vbo.release();

    // configure global opengl state
    // -----------------------------
    glEnable(GL_DEPTH_TEST);
}

void QtFunctionWidget::resizeGL(int w, int h){
    glViewport(0, 0, w, h);
}

void QtFunctionWidget::paintGL(){
    // input
    // -----
    camera->processInput(1.0f);

    // render
    // ------
    glClearColor(0.1f, 0.1f, 0.1f, 1.0f);
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

    // be sure to activate shader when setting uniforms/drawing objects
    lightingShader.bind();
    lightingShader.setUniformValue("objectColor", QVector3D(1.0f, 0.5f, 0.31f));
    lightingShader.setUniformValue("lightColor",  QVector3D(1.0f, 1.0f, 1.0f));

    // view/projection transformations
    QMatrix4x4 projection;
    projection.perspective(camera->zoom, 1.0f * width() / height(), 0.1f, 100.0f);
    QMatrix4x4 view = camera->getViewMatrix();
    lightingShader.setUniformValue("projection", projection);
    lightingShader.setUniformValue("view", view);

    // world transformation
    QMatrix4x4 model;
    lightingShader.setUniformValue("model", model);
    {// render the cube
        QOpenGLVertexArrayObject::Binder vaoBind(&cubeVAO);
        glDrawArrays(GL_TRIANGLES, 0, 36);
    }
    lightingShader.release();


    // also draw the lamp object
    lampShader.bind();
    lampShader.setUniformValue("projection", projection);
    lampShader.setUniformValue("view", view);
    model = QMatrix4x4();
    model.translate(lightPos);
    model.scale(0.2f); // a smaller cube
    lampShader.setUniformValue("model", model);
    {
        QOpenGLVertexArrayObject::Binder vaoBind(&lightVAO);
        glDrawArrays(GL_TRIANGLES, 0, 36);
    }
    lampShader.release();
}

void QtFunctionWidget::keyPressEvent(QKeyEvent *event)
{
    int key = event->key();
    if (key >= 0 && key < 1024)
        camera->keys[key] = true;
}

void QtFunctionWidget::keyReleaseEvent(QKeyEvent *event)
{
    int key = event->key();
    if (key >= 0 && key < 1024)
        camera->keys[key] = false;
}

void QtFunctionWidget::mousePressEvent(QMouseEvent *event)
{
    if(event->button() == Qt::LeftButton){
        m_bLeftPressed = true;
        m_lastPos = event->pos();
    }
}

void QtFunctionWidget::mouseReleaseEvent(QMouseEvent *event)
{
    Q_UNUSED(event);

    m_bLeftPressed = false;
}

void QtFunctionWidget::mouseMoveEvent(QMouseEvent *event)
{
    if (m_bLeftPressed) {
        int xpos = event->pos().x();
        int ypos = event->pos().y();

        int xoffset = xpos - m_lastPos.x();
        int yoffset = m_lastPos.y() - ypos;
        m_lastPos = event->pos();
        camera->processMouseMovement(xoffset, yoffset);
    }
}

void QtFunctionWidget::wheelEvent(QWheelEvent *event)
{
    QPoint offset = event->angleDelta();
    camera->processMouseScroll(offset.y()/20.0f);
}

bool QtFunctionWidget::createShader()
{
    bool success = lightingShader.addShaderFromSourceFile(QOpenGLShader::Vertex, ":/colors.vert");
    if (!success) {
        qDebug() << "shaderProgram addShaderFromSourceFile failed!" << lightingShader.log();
        return success;
    }

    success = lightingShader.addShaderFromSourceFile(QOpenGLShader::Fragment, ":/colors.frag");
    if (!success) {
        qDebug() << "shaderProgram addShaderFromSourceFile failed!" << lightingShader.log();
        return success;
    }

    success = lightingShader.link();
    if(!success) {
        qDebug() << "shaderProgram link failed!" << lightingShader.log();
    }

    success = lampShader.addShaderFromSourceFile(QOpenGLShader::Vertex, ":/lamp.vert");
    if (!success) {
        qDebug() << "shaderProgram addShaderFromSourceFile failed!" << lampShader.log();
        return success;
    }

    success = lampShader.addShaderFromSourceFile(QOpenGLShader::Fragment, ":/lamp.frag");
    if (!success) {
        qDebug() << "shaderProgram addShaderFromSourceFile failed!" << lampShader.log();
        return success;
    }

    success = lampShader.link();
    if(!success) {
        qDebug() << "shaderProgram link failed!" << lampShader.log();
    }

    return success;
}
```

## GLSL
### colors.vert
```
#version 330 core
layout (location = 0) in vec3 aPos;
 
uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;
 
void main(){
  gl_Position = projection * view * model * vec4(aPos, 1.0f);
}
```

### colors.frag
```
#version 330 core
out vec4 FragColor;
 
uniform vec3 objectColor;
uniform vec3 lightColor;
 
void main()
{
  FragColor = vec4(objectColor * lightColor, 1.0f);
}
```

### lamp.vert
```
#version 330 core
layout (location = 0) in vec3 aPos;
 
uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;
 
void main(){
  gl_Position = projection * view * model * vec4(aPos, 1.0f);
}
```

### lamp.frag
```
#version 330 core
out vec4 FragColor;
 
void main()
{
  FragColor = vec4(1.0f);
}
```

