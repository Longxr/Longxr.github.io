---
title: ffmpeg解码H264裸流为YUV数据
categories:
  - Qt
tags:
  - ffmpeg
  - H264
  - YUV
date: 2018-11-08 13:58:56
---

<Excerpt in index | 摘要> 
视频画面的传输中，由于原始数据过大，实际传输的数据是已经编码好的数据，一般是H264, 当客户端收到后就需要解码并显示出来。<!-- more -->
<The rest of contents | 余下全文>

### 裸流解析成AVPacket
#### AVCodecParser
AVCodecParser用于解析输入的数据流并把它分成一帧一帧的压缩编码数据。就像是你把肉塞进火腿，再交给它负责帮你切片，够一个完整的帧就返回给你处理。

#### 使用
```
int parser_len = 0;

while(parser_len < buffer.length()){
    parser_len += av_parser_parse2(m_pVideoParserContext, m_pVideoDecoder->GetCodecContext(),
                                   &pParsePacket->data, &pParsePacket->size,
                                   (uint8_t *)(buffer.data() + parser_len), buffer.length() - parser_len,
                                   AV_NOPTS_VALUE, AV_NOPTS_VALUE, AV_NOPTS_VALUE);

    if(0 == pParsePacket->size){
        continue;
    }

    switch(m_pVideoParserContext->pict_type){
        case AV_PICTURE_TYPE_I: qDebug("Type:I\t");break;
        case AV_PICTURE_TYPE_P: qDebug("Type:P\t");break;
        case AV_PICTURE_TYPE_B: qDebug("Type:B\t");break;
    default: qDebug("Type:Other\t");break;
}
    //pParsePacket就是一帧的数据包AVPacket，这里可以保存作为录像
    NotifyReceiveVideoPacket(pParsePacket);

    //丢给解码器处理
    m_pVideoDecoder->Decode(pParsePacket);

    av_packet_unref(pParsePacket);
} // parse H264 packet while
```
每次塞给AVCodecParser的数据是buffer中的数据，其中可能包含多个帧，所以内部会有一个while循环，parser_len是已经解析了的数据的长度，直到解析完一次的buffer，再塞新的数据到buffer继续处理。

### 解码为AVFrame
```
if(0 != avcodec_send_packet(pCodecContext, pPacket)){
    qDebug("avcodec_send_packet failed");
    return false;
}

while(0 == avcodec_receive_frame(pCodecContext, pFrame)){
    //pFrame包含了解码后的YUV数据
    ...
}
```
由于AVFrame是ffmpeg的数据结构，要分别提取出Y、U、V三个通道的数据才能用于显示。

### YUV数据格式
RGB来表示颜色大家都不陌生，R(红色)、G(绿色)、B(蓝色)，通过这三基色就可以组合成其他需要的颜色。YUV也是一种表示颜色的方式，其中Y(亮度)、U(色度)、V(浓度)。YUV根据采样方式的不同又有YUV444、YUV422、YUV420等多种格式。这里主要介绍YUV420采样格式。

![YUV格式](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/ffmpeg-H264-to-YUV_01.jpg)

#### YUV420
YUV420格式是指，每个像素都保留一个Y分量 (亮度全抽样)，而在水平方向上，不是每行都取U和V分量，而是一行只取U分量，接着一行就只取V分量，以此重复(即4:2:0, 4:0:2, 4:2:0, 4:0:2 .......)。所以420不是指没有V，而是指一行采样只取U，另一行采样只取V。从4x4矩阵列来看，每4个矩阵点Y区域中，只有一个U和V，所以它们的比值是4:1。

#### 数据大小
##### 对于1个像素的信息存储
1. RGB格式：R、G、B各占8位，共24位，即3byte
2. YUV420格式： Y占8位，U、V每4个点共有一个，共8 + 8 / 4 + 8 / 4  = 12位，即3 / 2byte

##### 对于一张图像的数据大小
1. RGB格式：width * height * 3byte
2. YUV420格式：width * height * 3 / 2 byte

所以采取YUV420来存储图像数据比RGB格式节省了一半的空间。

#### YUV420P
采样好了数据，在存储YUV数据的时候，对于YUV不同的存储方式又有YUV420P(YV12)、YUV420SP(NV12)等分类。

![yv12](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/ffmpeg-H264-to-YUV_02.png)

![nv12](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/ffmpeg-H264-to-YUV_03.png)

从图上可以看出，YUV420P和NV12的区别就是一个是UV交替存储，一个是先存U再存V。这个在解码的时候就看你解码格式指定的是哪个了，默认解码是YUV420P。

### 从AVFrame中获取YUV数据
YUV420P在AVFrame中的存储，data[0]存Y分量，data[1]存U分量，data[2]存V分量。**其中，图像每一行Y、U、V数据的大小分别是linesize[0]、linesize[1]、linesize[2]，除了实际的图像数据，还有一些填充数据是不需要的。**填充数据应该是为了内存对齐，保留的话可能会导致花屏，具体可以看[这里的分析](https://blog.csdn.net/dancing_night/article/details/80830920)。

所以，获取实际的图像YUV数据代码为：
```
for(int i = 0; i < pFrame->height; i++)
  memcpy(m_pBuffer + pFrame->width * i, pFrame->data[0]  + pFrame->linesize[0] * i, pFrame->width);

for(int j = 0; j < pFrame->height / 2; j++)
  memcpy(m_pBuffer + pFrame->width / 2 * j + ysize, pFrame->data[1]  + pFrame->linesize[1] * j, pFrame->width / 2);

for(int k = 0; k < pFrame->height / 2; k++)
  memcpy(m_pBuffer + pFrame->width / 2 * k + ysize * 5 / 4, pFrame->data[2]  + pFrame->linesize[2] * k, pFrame->width / 2);
```
我是直接用一个pBuffer来保存YUV数据了，记住偏移量就行，想分开存也没问题。这里的数据就可以直接交给渲染部分来显示图像了。

### 参考资料
* [最简单的基于FFmpeg的解码器-纯净版（不包含libavformat）](https://blog.csdn.net/leixiaohua1020/article/details/42181571)
* [YUV颜色编码解析](https://www.jianshu.com/p/a91502c00fb0)
* [AVFrame存储YUV420P对齐分析](https://blog.csdn.net/dancing_night/article/details/80830920)
* [NV12与YV12，YUV的主要格式](https://blog.csdn.net/Evankaka/article/details/38176025)

