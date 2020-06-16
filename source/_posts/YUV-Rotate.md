---
title: YUV图像旋转
categories:
  - 音视频
tags:
  - libyuv
  - yuv
date: 2020-06-16 13:56:57
---

<Excerpt in index | 摘要>
手机竖屏录制的视频，PC 播放时需要对解码后的数据做下旋转。<!-- more -->
<The rest of contents | 余下全文>

# 视频旋转角度获取

## 命令行获取

使用 ffprobe.exe，`ffprobe video.mp4`，会输出视频的详细信息。在视频的 Metadata 的 rotate 属性如果有值的话就是旋转的角度了。

![YUV-Rotate-2020-06-16-14-04-03](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/YUV-Rotate-2020-06-16-14-04-03.png)

## 代码中获取

在获取 video 的 AVStream 时调用：

```C++
int getVideoRotation(AVStream *st)
{
    AVDictionaryEntry *rotate_tag = av_dict_get(st->metadata, "rotate", nullptr, 0);
    int rotation = 0;

    if (rotate_tag) {
        rotation = QString(rotate_tag->value).toInt();
    }

    while (rotation < 0) {
        rotation += 360;
    }

    while (rotation > 360) {
        rotation -= 360;
    }

    LOG->info("vodeo rotation: {}", rotation);

    return rotation;
}
```

# 旋转 YUV

旋转可以使用 libyuv 这个库，[github 地址](https://github.com/lemenkov/libyuv)。旋转相关函数可以看`libyuv\include\libyuv\rotate.h`。比如将 I420（ffmpeg 里的 YUV420P）旋转 270 度只需要调用：

```C++
uchar * i420_y = _pBuffer;
uchar * i420_u = i420_y + (width * height);
uchar * i420_v = i420_y + (width * height) * 5 / 4;

uchar * rotated_y = _pRotateBuffer;
uchar * rotated_u = rotated_y + (width * height);
uchar * rotated_v = rotated_y + (width * height) * 5 / 4;

libyuv::I420Rotate(
        i420_y, width,
        i420_u, (width >> 1),
        i420_v, (width >> 1),
        rotated_y, height,
        rotated_u, (height >> 1),  //90度和270度宽高会互换，步长需要注意
        rotated_v, (height >> 1),
        width, height, libyuv::kRotate270);
```

libyuv 旋转的同时还能把格式也转换了，比如`NV12ToI420Rotate`，有需要可以翻头文件看看。
