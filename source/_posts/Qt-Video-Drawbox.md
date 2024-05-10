---
title: Qt在视频画面上绘制动态矩形
categories:
  - Qt
tags:
  - ffmpeg
  - Qt
date: 2019-08-23 14:10:27
---

<Excerpt in index | 摘要> 
Qt可以通过QOpenGLWidget使用OpenGL渲染显示视频，在渲染的视频画面上，有可能需要绘制一些几何图案(识别之类的画个框框那种)。目前试了3种不同的方式在画面上绘制矩形，记录下使用过程。<!-- more -->
<The rest of contents | 余下全文>

## 通过QPainter绘制
Qt的绘制函数`paintEvent(QPaintEvent *event)`在QOpenGLWidget中可以绘制，并且和OpenGL的内容叠在一起，只需要在绘制之前先调用下基类的`paintEvent(QPaintEvent *event)`即可，可以理解为先在画布上画好视频，再在画布上画个矩形。这种方式灵活性最好。
```
void RenderWidget::paintEvent(QPaintEvent *event)
{
    QOpenGLWidget::paintEvent(event);

    qreal offset = sin(m_nCount * 0.1); //m_nCount是渲染的帧数

    QPainter painter(this);
    painter.setRenderHints(QPainter::SmoothPixmapTransform);
    painter.save();
    painter.setPen(QPen(QColor("#4FE3C1"), 1, Qt::SolidLine, Qt::RoundCap, Qt::RoundJoin));
    painter.drawRect(QRectF(width() * offset, height() * 0.7, width() * 0.25, height() * 0.25));
    painter.restore();
}
```

## 通过OpenGL绘制
通过不同的QOpenGLShaderProgram，可以指定不同的着色器程序来实现矩形的绘制。这种方式绘制的时候，偏移量这些变化参数要通过Uniform传递给OpenGL的顶点着色器，如果图形复杂或者带3D可以考虑。
```
void RenderWidget::initializeGL()
{
    initializeOpenGLFunctions();
    const char *vsrc =
            "attribute vec4 vertexIn; \
             attribute vec4 textureIn; \
             varying vec4 textureOut;  \
             void main(void)           \
             {                         \
                 gl_Position = vertexIn; \
                 textureOut = textureIn; \
             }";

    const char *fsrc =
            "varying mediump vec4 textureOut;\
            uniform sampler2D tex_y; \
            uniform sampler2D tex_u; \
            uniform sampler2D tex_v; \
            void main(void) \
            { \
                vec3 yuv; \
                vec3 rgb; \
                yuv.x = texture2D(tex_y, textureOut.st).r; \
                yuv.y = texture2D(tex_u, textureOut.st).r - 0.5; \
                yuv.z = texture2D(tex_v, textureOut.st).r - 0.5; \
                rgb = mat3( 1,       1,         1, \
                            0,       -0.39465,  2.03211, \
                            1.13983, -0.58060,  0) * yuv; \
                gl_FragColor = vec4(rgb, 1); \
            }";


    const char *rcvsrc = "#version 330 core\n \
            layout(location = 0) in vec3 aPos;\n \
            uniform vec2 offsetP; \
            void main(){\n \
                gl_Position = vec4(aPos.x + offsetP.x, aPos.y + offsetP.y, aPos.z, 1.0f);\n \
            }\n ";

    const char *rcfsrc = "#version 330 core\n \
            out vec4 FragColor;\n \
            void main(){\n \
                FragColor = vec4(1.0f, 0.0f, 0.0f, 1.0f);\n \
            }\n ";

    videoProgram.addCacheableShaderFromSourceCode(QOpenGLShader::Vertex,vsrc);
    videoProgram.addCacheableShaderFromSourceCode(QOpenGLShader::Fragment,fsrc);
    videoProgram.link();

    rcProgram.addShaderFromSourceCode(QOpenGLShader::Vertex,rcvsrc);
    rcProgram.addShaderFromSourceCode(QOpenGLShader::Fragment,rcfsrc);
    rcProgram.link();

    {
        QOpenGLVertexArrayObject::Binder vaoBind(&rcvao);

        GLfloat rcPoints[]{
            0.25f,  0.25f, 0.0f,  // top right
             0.25f, -0.25f, 0.0f,  // bottom right
            -0.25f, -0.25f, 0.0f,  // bottom left
            -0.25f,  0.25f, 0.0f,   // top left
        };

        rcvbo.create();
        rcvbo.bind();
        rcvbo.allocate(rcPoints, sizeof(rcPoints));

        int attr = -1;
        attr = rcProgram.attributeLocation("aPos");
        rcProgram.setAttributeBuffer(attr, GL_FLOAT, 0, 3, sizeof(GLfloat) * 3);
        rcProgram.enableAttributeArray(attr);
        rcvbo.release();
    }

    {
        QOpenGLVertexArrayObject::Binder vaoBind(&vao);

        GLfloat points[]{
            -1.0f, 1.0f,
             1.0f, 1.0f,
             1.0f, -1.0f,
            -1.0f, -1.0f,

            0.0f,0.0f,
            1.0f,0.0f,
            1.0f,1.0f,
            0.0f,1.0f
        };

        vbo.create();
        vbo.bind();
        vbo.allocate(points,sizeof(points));

        m_pTextureY = std::make_unique<QOpenGLTexture>(QOpenGLTexture::Target2D);
        m_pTextureU = std::make_unique<QOpenGLTexture>(QOpenGLTexture::Target2D);
        m_pTextureV = std::make_unique<QOpenGLTexture>(QOpenGLTexture::Target2D);
        m_pTextureY->create();
        m_pTextureU->create();
        m_pTextureV->create();

        int attr = -1;
        attr = videoProgram.attributeLocation("vertexIn");
        videoProgram.setAttributeBuffer(attr, GL_FLOAT, 0, 2, 2*sizeof(GLfloat));
        videoProgram.enableAttributeArray(attr);

        attr = videoProgram.attributeLocation("textureIn");
        videoProgram.setAttributeBuffer(attr, GL_FLOAT, 2 * 4 * sizeof(GLfloat),2,2*sizeof(GLfloat));
        videoProgram.enableAttributeArray(attr);

        vbo.release();
    }

    videoProgram.release();
    rcProgram.release();
}

void RenderWidget::render(uchar *yuvPtr, int w, int h)
{
    m_nCount++;
//    glDisable(GL_DEPTH_TEST);
    glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
    glClear(GL_COLOR_BUFFER_BIT);

    if(nullptr == yuvPtr || 0 >= w || 0 >= h){
        return;
    }

    videoProgram.bind();
    {
        QOpenGLVertexArrayObject::Binder vaoBind(&vao);

        //Y
        glActiveTexture(GL_TEXTURE2);
        glBindTexture(GL_TEXTURE_2D, m_pTextureY->textureId());
        glTexImage2D(GL_TEXTURE_2D, 0, GL_RED, w, h, 0, GL_RED,GL_UNSIGNED_BYTE, yuvPtr);
        glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MAG_FILTER,GL_LINEAR);
        glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MIN_FILTER,GL_LINEAR);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);

        //U
        glActiveTexture(GL_TEXTURE1);
        glBindTexture(GL_TEXTURE_2D, m_pTextureU->textureId());
        glTexImage2D(GL_TEXTURE_2D, 0, GL_RED, w >> 1, h >> 1, 0, GL_RED,GL_UNSIGNED_BYTE, yuvPtr + w * h);
        glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MAG_FILTER,GL_LINEAR);
        glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MIN_FILTER,GL_LINEAR);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);

        //V
        glActiveTexture(GL_TEXTURE0);
        glBindTexture(GL_TEXTURE_2D, m_pTextureV->textureId());
        glTexImage2D(GL_TEXTURE_2D, 0, GL_RED, w >> 1, h >> 1, 0, GL_RED, GL_UNSIGNED_BYTE, yuvPtr + w * h * 5 / 4);
        glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MAG_FILTER,GL_LINEAR);
        glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MIN_FILTER,GL_LINEAR);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);

        videoProgram.setUniformValue("tex_y",2);
        videoProgram.setUniformValue("tex_u",1);
        videoProgram.setUniformValue("tex_v",0);
        glDrawArrays(GL_QUADS, 0, 4);
    }
    videoProgram.release();

    rcProgram.bind();
    {
        QOpenGLVertexArrayObject::Binder rcvaoBind(&rcvao);
        glPolygonMode(GL_FRONT_AND_BACK, GL_LINE);

        float offset = sin(m_nCount * 0.1f);
        rcProgram.setUniformValue("offsetP", QVector2D(offset, 0.7f));
        glDrawArrays(GL_QUADS, 0, 4);
        glPolygonMode(GL_FRONT_AND_BACK, GL_FILL);
    }
    rcProgram.release();
}

void RenderWidget::resizeGL(int w, int h)
{
    glViewport(0, 0, w, h);
}
```

## 通过AVFilter绘制
ffmpeg的滤镜有很多强大的功能，这里就用了个最简单的drawbox, 使用滤镜的话初始化要做很多设置，另外我还没找到在哪设置可以改变绘制矩形的位置，`filter_descr`就在初始化的时候用了一次。这种方式会直接将绘制内容输出到最后的视频数据上，如果不是需要保留包含绘制内容的视频的话更倾向于上面两种方式。

### 初始化

```
  ...
    static const char *filter_descr = "drawbox=x=200:y=400:w=200:h=200:color=blue";

    AVFilterContext* buffersrc_ctx = nullptr;
    AVFilterContext* buffersink_ctx = nullptr;
    AVFilterGraph* filter_graph = nullptr;
  ...

int InitFilters(const char *filters_descr)
{
    int ret;
    const AVFilter *buffersrc  = avfilter_get_by_name("buffer");
    const AVFilter *buffersink = avfilter_get_by_name("buffersink");
    AVFilterInOut *outputs = avfilter_inout_alloc();
    AVFilterInOut *inputs  = avfilter_inout_alloc();
    enum AVPixelFormat pix_fmts[] = { AV_PIX_FMT_YUV420P, AV_PIX_FMT_NONE };
    AVBufferSinkParams *buffersink_params;

    filter_graph = avfilter_graph_alloc();

    /* buffer video source: the decoded frames from the decoder will be inserted here. */
    QString args = QString("video_size=%1x%2:pix_fmt=%3:time_base=%4/%5:pixel_aspect=%6/%7")
            .arg(m_pCodecContext->width).arg(m_pCodecContext->height).arg(m_pCodecContext->pix_fmt)
            .arg(/*m_pCodecContext->time_base.num*/1).arg(30)
            .arg(/*m_pCodecContext->sample_aspect_ratio.num*/16).arg(/*m_pCodecContext->sample_aspect_ratio.den*/9);

    ret = avfilter_graph_create_filter(&buffersrc_ctx, buffersrc, "in",
                                       args.toStdString().c_str(), nullptr, filter_graph);
    if (ret < 0) {
        qDebug() << "Cannot create buffer source " << AVErr2QString(ret);
        return ret;
    }

    /* buffer video sink: to terminate the filter chain. */
    buffersink_params = av_buffersink_params_alloc();
    buffersink_params->pixel_fmts = pix_fmts;
    ret = avfilter_graph_create_filter(&buffersink_ctx, buffersink, "out",
                                       nullptr, buffersink_params, filter_graph);
    av_free(buffersink_params);
    if (ret < 0) {
        qDebug() << "Cannot create buffer sink " << AVErr2QString(ret);
        return ret;
    }

    /* Endpoints for the filter graph. */
    outputs->name       = av_strdup("in");
    outputs->filter_ctx = buffersrc_ctx;
    outputs->pad_idx    = 0;
    outputs->next       = nullptr;

    inputs->name       = av_strdup("out");
    inputs->filter_ctx = buffersink_ctx;
    inputs->pad_idx    = 0;
    inputs->next       = nullptr;

    if ((ret = avfilter_graph_parse_ptr(filter_graph, filters_descr,
                                    &inputs, &outputs, nullptr)) < 0)
        return ret;

    if ((ret = avfilter_graph_config(filter_graph, nullptr)) < 0)
        return ret;
    return 0;
} 
    
```

### 解码后的AVFrame应用Filter
```
    if (av_buffersrc_add_frame(buffersrc_ctx, pFrame) < 0) {
        qDebug("Error while add frame.\n");
        return;
    }

    /* pull filtered pictures from the filtergraph */
    int ret = av_buffersink_get_frame(buffersink_ctx, m_pFrameOut);//frameout中的数据就是加好滤镜的内容了
    if (ret < 0)
        return;
```
## 运行效果
绿的QPainter绘制，红的OpenGL绘制，蓝色AVFilter绘制

![运行效果](https://upload-images.jianshu.io/upload_images/2756183-fa7eb9cb6ebc0c6f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 参考资料
* [qt采用opengl显示yuv视频数据](https://blog.csdn.net/su_vast/article/details/52214642)
* [最简单的基于FFmpeg的AVfilter例子（水印叠加）](https://blog.csdn.net/leixiaohua1020/article/details/29368911)

