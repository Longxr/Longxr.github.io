---
title: H264码流分析
categories:
  - null
tags:
  - null
date: 2020-05-28 19:07:27
---

<Excerpt in index | 摘要>
H264 是属于视频的编码层的标准格式，平时用 ffmpeg 只是指定下参数就完事了，并没有仔细了解，今天来分析下 H264 码流具体的内容。<!-- more -->
<The rest of contents | 余下全文>

# ffmpeg 中使用 H264 编码

ffmpeg 已经有实现好了编码器，调用的时候指定`AV_CODEC_ID_H264`，会使用 x264 的软编码；如果需要硬编码将查找编码器改为`avcodec_find_encoder_by_name("h264_qsv")`。

```C++
int size;
int in_w = mWidth;
int in_h = mHeight;
 
//x264软编码初始化
pCodec = avcodec_find_encoder(AV_CODEC_ID_H264);
if(!pCodec) {
  fprintf(stderr, "h264 codec not found");
  return false;
}

//qsv硬编码
// pCodec = avcodec_find_encoder_by_name("h264_qsv");
// if(Q_NULLPTR == pCodec){
//     qDebug("avcodec_find_encoder failed!");
//     return false;
// }
 
pCodecCtx = avcodec_alloc_context3(pCodec);
pCodecCtx->codec_id = AV_CODEC_ID_H264;
pCodecCtx->codec_type = AVMEDIA_TYPE_VIDEO;
pCodecCtx->pix_fmt = PIX_FMT_YUV420P;
pCodecCtx->width = in_w;
pCodecCtx->height = in_h;
pCodecCtx->time_base.num = 1;
pCodecCtx->time_base.den = 25;//帧率
pCodecCtx->bit_rate = mBitRate; //比特率
pCodecCtx->gop_size=25;

···

//读取编码后数据，在AVPacket中
if(0 != avcodec_send_frame(m_pEncodeContext, m_pSwsFrame)) {
    qDebug("avcodec_send_frame failed");
    return false;
}
while(0 == avcodec_receive_packet(m_pEncodeContext, m_pDstPacket)) {
    if(m_pCallback) {
        if (m_pDstPacket->flags & AV_PKT_FLAG_KEY) {
            qDebug("encode a key frame");
        }

        m_pCallback->onVideoEncodeFinished(m_pDstPacket->data, m_pDstPacket->size);

    }

    av_packet_unref(m_pDstPacket);
}
```

# H264 原始码流结构

在 H.264/AVC 视频编码标准中，整个系统框架被分为两层，VCL(视频编码层 video coding layer) 和 NAL(网络提取层 network abstraction layer)：

1. VCL：核心算法引擎，块，宏块及片的语法级别的定义，负责高效的视频内容表示，通俗的讲就是编码器直接编码之后的数据，这部分数据还不能直接用于保存和网络传输，否则在解析上存在困难。
2. NAL：负责格式化数据并提供头信息，以保证数据适合各种信道和存储介质上的传输，通俗的讲 NAL 就是 VCL 加了一些头部信息封装了一下。

H264 码流数据由 NALU 序列组成，相邻的 NALU 由起始码 StartCode 隔开，起始码 StartCode 的两种形式：3 字节的 0x000001 和 4 字节的 0x00000001。3 字节的 起始码只有在一个完整的帧被编为多个 slice（片）的时候，从第二个 slice 开始，使用 3 字节起始码。
NALU 是一个 NAL 单元，NALU = 字节的头信息（NALU Header） + 原始字节序列载荷（RBSP）（实际为扩展字节序列载荷（EBSP）的字节流）

![H264-Analyze-2020-06-07-11-19-46](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/H264-Analyze-2020-06-07-11-19-46.jpg)

## NALU Header

NALU Header 由 3 部分组成，NALU Header = forbidden_bit(1bit) + nal_reference_bit(2bit) + nal_unit_type(5bit)，共占用 1 个字节

1. forbidden_bit 禁止位：编码中默认值为 0，当网络识别此单元中存在比特错误时，可将其设为 1，以便接收方丢掉该单元，主要用于适应不同种类的网络环境
2. nal_reference_bit 重要性指示位：用于在重构过程中标记一个 NAL 单元的重要性，值越大，越重要。尤其是当前 NALU 为图像参数集、序列参数集或 IDR 图像时，或者为参考图像条带（片/Slice），或者为参考图像的条带数据分割时，nal_ref_idc 值肯定不为 0。
3. nal_unit_type：NALU 类型位: 可以表示 NALU 的 32 种不同类型特征，类型 1 ～ 12 是 H.264 定义的，类型 24 ～ 31 是用于 H.264 以外的，RTP 负荷规范使用这其中的一些值来定义包聚合和分裂，其他值为 H.264 保留。

![H264-Analyze-2020-06-07-11-31-18](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/H264-Analyze-2020-06-07-11-31-18.png)

判断起始码后的第一个字节的后 5bit 就可以判断出这个 NALU 的类型，例如：`00 00 00 01 67: 0x67&0x1f = 0x07 :SPS`。另外说下重要的类型：

* al_unit_type=5：表示当前 NAL 是 IDR 图像的一个片，在这种情况下，IDR 图像中的每个片的 nal_unit_type 都应该等于 5。注意，IDR 图像不能使用分区。
* nal_unit_type=7：SPS，包括一个图像序列的所有信息，即两个IDR图像间的所有图像信息，如标识符 seq_parameter_set_id、帧数及 POC 的约束、参考帧数目、解码图像尺寸和帧场编码模式选择标识等等。
* nal_unit_type=8：PPS，PPS对应的是一个序列中某一幅图像或者某几幅图像，包括一个图像的所有slice的所有相关信息，如图像类型、序列号、标识符 pic_parameter_set_id、可选的 seq_parameter_set_id、熵编码模式选择标识、片组数目、初始量化参数和去方块滤波系数调整标识等等，解码时某些序列号的丢失可用来校验信息包的丢失与否。

## RBSP

NALU 的主体部分需要先介绍下面 3 个名词。

1. SODB: String Of Data Bits  原始数据比特流，编码后的原始数据，即VCL数据。
2. RBSP: 原始字节序列载荷。由于 SODB 长度不一定是 8 的倍数，为了字节对齐，在 SODB 的后面填加了结尾比特（rbsp_trailing_bits）就得到了 RBSP。rbsp_trailing_bits = rbsp_stop_one_bit（值为 1，1bit）+ rbsp_alignment_zero_bit(若干个 0，字节对齐填充)
3. EBSP: 扩展字节序列载荷。NALU 之间是通过 StartCode 来隔开的，如果编码后的原始数据含有 0x000001，就无法知道这个到底是不是 StartCode，因此为了使 NALU 主体中不包括与开始码相冲突的数据，在 RBSP 数据中每遇到两个字节连续为 0，就插入一个字节的 0x03，这样就得到了 EBSP。

所以，SODB + 添加尾部字节对齐 ==> RBSP + 碰到 0x0000 插入 0x03 ==> EBSP。
VCL层是对核心算法引擎、块、宏块及片的语法级别的定义，最终输出压缩编码后的数据 SODB。VCL数据在传输或存储之前，先被映射或封装进NAL单元中。NAL层将SODB打包成RBSP然后加上NALU Header，组成一个NALU。

![H264-Analyze-2020-06-07-12-32-45](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/H264-Analyze-2020-06-07-12-32-45.png)

相应的 RBSP 数据类型描述如下：

![H264-Analyze-2020-06-07-12-57-26](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/H264-Analyze-2020-06-07-12-57-26.png)

## 视频帧

从宏观上来说，SPS、PPS、IDR 帧（包含一个或多个 I-Slice）、P 帧（包含一个或多个 P-Slice ）、B 帧（包含一个或多个 B-Slice ）组成的视频序列构成了 H264 的码流。
包含具体视频数据的帧有：
| 类型 | 意义                                                                                                                    |
| ---- | ----------------------------------------------------------------------------------------------------------------------- |
| I 帧 | I 帧通常是每个 GOP（MPEG 所使用的一种视频压缩技术）的第一个帧，经过适度地压缩，做为随机访问的参考点，可以当成图象。     |
| P 帧 | 通过充分将低于图像序列中前面已编码帧的时间冗余信息来压缩传输数据量的编码图像，也叫预测帧                                |
| B 帧 | 既考虑与源图像序列前面已编码帧，也顾及源图像序列后面已编码帧之间的时间冗余信息来压缩传输数据量的编码图像,也叫双向预测帧 |

一个视频序列的第一帧又叫 IDR 帧。IDR 帧的作用是立刻刷新，使错误不致传播，从 IDR 帧开始，重新算一个新的序列开始编码。
I 帧可以不依赖其他帧就解码出一幅完整的图像，而 P 帧、B 帧不行。P 帧需要依赖视频流中排在它前面的帧才能解码出图像，B 帧则需要依赖视频流中排在它前面或后面的帧才能解码出图像。在视频流中，先到来的 B 帧无法立即解码，需要等待它依赖的后面的 P 帧先解码，播放时间与解码时间不一致，这时就需要DTS和PTS了。

* DTS（Decoding Time Stamp）：解码时间戳，这个时间戳的意义在于告诉播放器该在什么时候解码这一帧的数据。
* PTS（Presentation Time Stamp）：显示时间戳，这个时间戳用来告诉播放器该在什么时候显示这一帧的数据。

在没有B帧的时候，DTS、PTS顺序是一致的，比如直播等延迟要求小的使用场景。音频也有PTS、DTS，但是音频没有类似视频的 B 帧，不需要双向预测，所以音频帧的 DTS、PTS 顺序是一致的。

![H264-Analyze-2020-06-07-17-28-25](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/H264-Analyze-2020-06-07-17-28-25.png)

## slice

帧图像可编码成一个或者多个片（slice），分片的目的是为了限制误码的扩散和传输，使编码片相互间保持独立。
每一个 slice 总体来看都由两部分组成，一部分作为 slice header，用于保存 slice 的总体信息（如当前 slice 的类型等），另一部分为 slice body，通常是一组连续的宏块结构。

![H264-Analyze-2020-06-07-17-07-27](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/H264-Analyze-2020-06-07-17-07-27.png)

slice 的类型如下：
| 类型     | 意义                                    |
| -------- | --------------------------------------- |
| I slice  | 只包含 I 宏块                           |
| P slice  | 包含 P 和 I 宏块                        |
| B slice  | 包含 B 和 I 宏块                        |
| SP slice | 包含 P 或 I 宏块,用于不同码流之间的切换 |
| SI slice | 一种特殊类型的编码宏块                  |

## 宏块

宏块是视频信息的主要承载者。一个编码图像通常划分为多个宏块组成.包含着每一个像素的亮度和色度信息。视频解码最主要的工作则是提供高效的方式从码流中获得宏块中像素阵列。
一个宏块由一个 16×16 亮度像素和附加的一个 8×8 Cb 和一个 8×8 Cr 彩色像素块组成。宏块分类如下：

| 类型   | 意义                                                       |
| ------ | ---------------------------------------------------------- |
| I 宏块 | 利用从当前片中已解码的像素作为参考进行帧内预测             |
| P 宏块 | 利用前面已编码图像作为参考进行帧内预测                     |
| B 宏块 | 利用双向的参考图像（当前和未来的已编码图像帧）进行帧内预测 |

![H264-Analyze-2020-06-07-17-19-09](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/H264-Analyze-2020-06-07-17-19-09.png)

# H264 解析流程

拿到 RBSP 或 SODB 之后，根据对应的 NALU Header 类型来解析编码的数据，解析流程如下：

![H264-Analyze-2020-06-07-16-24-28](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/H264-Analyze-2020-06-07-16-24-28.jpg)

使用 ffmpeg 的话一般是不会自己去解析的，因为数据都在对应的结构体变量中，比如判断一帧是否是 I 帧：

```C++
if (m_pDstPacket->flags & AV_PKT_FLAG_KEY) {
    qDebug("encode a key frame");
}
```

# 参考链接：

- [H264 协议文档](https://www.itu.int/rec/T-REC-H.264-200503-S/en)
- [“视频”是怎么来的？H.264、码率这些词又是什么意思？](https://www.bilibili.com/video/av37424012)
- [H264 基础简介](https://www.jianshu.com/p/8edb448cf22e)
- [h264 码流格式与 NALU 详解一](https://www.jianshu.com/p/a2dc69c8bf70)
- [H.264 数据解析](https://blog.csdn.net/qq214517703/article/details/80359911)
- [H264 编解码协议详解](https://blog.csdn.net/qq_19923217/article/details/83348095)
