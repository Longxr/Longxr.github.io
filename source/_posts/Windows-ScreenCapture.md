---
title: windows桌面采集（GDI、DSHOW、DXGI对比）
categories:
  - C++
tags:
  - windows
  - ffmpeg
  - GDI
  - DSHOW
  - DXGI
date: 2020-05-26 20:08:04
---

<Excerpt in index | 摘要>
windows 下 3 种桌面采集方式的对比<!-- more -->
<The rest of contents | 余下全文>

桌面采集就是定时截取桌面的画面，并且将截取后的 RGB 画面通过某种编码方式（比如 H264）压缩后再发送出去。采集的帧率达到 20 帧以上的时候，接收端再显示出来就有投屏的效果了。

# GDI

Windows 图形设备接口(GDI)是为与设备无关的图形设计的。基于 Windows 的应用程序不能直接访问图形硬件，应用程序通过 GDI 来与设备驱动程序进行交互。GDI 截图就是通过屏幕的设备环境（DC）获取到当前屏幕的位图数据。

## ffmpeg 调用

ffmpeg 封装了 GDI 截图的相关的函数，调用和文件的播放差不多，区别就是初始化时多调用了`av_find_input_format`。
获取到的每一帧数据通过`avcodec_receive_frame`拿到 AVFrame。

```C++
//GDI截屏初始化
m_pFormatContext = avformat_alloc_context();
if(nullptr == m_pFormatContext) {
    qDebug("avformat_alloc_context failed");
    break;
}

AVInputFormat * pInputFormat = av_find_input_format("gdigrab");
if(nullptr == pInputFormat) {
    qDebug("av_find_input_format failed");
    break;
}

AVDictionary * options = nullptr;
if(0 != avformat_open_input(&m_pFormatContext, "desktop", pInputFormat, &options)) {
    qDebug("avformat_open_input failed");
    break;
}
...

//文件播放初始化
m_pAVFormatContext = avformat_alloc_context();

int result = avformat_open_input(&m_pAVFormatContext, m_filePath.toStdString().c_str(), nullptr, nullptr);
if (result < 0) {
    char errbuf[64];
    av_strerror(result, errbuf, sizeof(errbuf));
    qDebug("open stream failed: %s", errbuf);
    return false;
}
```

## ffmpeg 实现

相关实现代码在`FFmpeg/libavcodec/gdigrab.c`。通过调用`gdigrab_read_header()`完成初始化，`gdigrab_read_packet()`完成实际的截图。

### gdigrab_read_header

1. 确定窗口的句柄 hwnd。如果指定了 title 的话，调用 FindWindow()获取 hwnd；如果指定了 desktop，则设定 hwnd 为 NULL
2. 确定抓屏的矩形区域。如果抓取指定窗口，则通过 GetClientRect()函数；否则就抓取整个屏幕
3. 调用 GDI 的 API 完成抓屏的一些初始化工作。包括：
   a)通过 GetDC()获得某个窗口句柄的 HDC
   b)通过 CreateCompatibleDC()创建一个与指定设备兼容的 HDC
   c)通过 CreateDIBSection()创建一个设备无关位图 HBITMAP
   d)通过 SelectObject()绑定 HBITMAP 和 HDC
4. 通过 avformat_new_stream()创建一个 AVStream
5. 将初始化时候的一些参数保存至 GDIGrab 的上下文结构体

### gdigrab_read_packet

1. 从 GDIGrab 上下文结构体读取初始化时候设定的参数
2. 根据帧率参数进行延时
3. 通过 av_new_packet()新建一个 AVPacket
4. 通过 BitBlt()完成抓屏功能
5. 如果需要画鼠标指针的话，调用 paint_mouse_pointer()
6. 拷贝图像内容至 AVPacket 的 data

### 相关源码

```C++
/**
 * Initializes the gdi grab device demuxer (public device demuxer API).
 *
 * @param s1 Context from avformat core
 * @return AVERROR_IO error, 0 success
 */
static int
gdigrab_read_header(AVFormatContext *s1)
{
    struct gdigrab *gdigrab = s1->priv_data;

    HWND hwnd;
    HDC source_hdc = NULL;
    HDC dest_hdc   = NULL;
    BITMAPINFO bmi;
    HBITMAP hbmp   = NULL;
    void *buffer   = NULL;

    const char *filename = s1->url;
    const char *name     = NULL;
    AVStream   *st       = NULL;

    int bpp;
    int horzres;
    int vertres;
    int desktophorzres;
    int desktopvertres;
    RECT virtual_rect;
    RECT clip_rect;
    BITMAP bmp;
    int ret;

    if (!strncmp(filename, "title=", 6)) {
        name = filename + 6;
        hwnd = FindWindow(NULL, name);
        if (!hwnd) {
            av_log(s1, AV_LOG_ERROR,
                   "Can't find window '%s', aborting.\n", name);
            ret = AVERROR(EIO);
            goto error;
        }
        if (gdigrab->show_region) {
            av_log(s1, AV_LOG_WARNING,
                    "Can't show region when grabbing a window.\n");
            gdigrab->show_region = 0;
        }
    } else if (!strcmp(filename, "desktop")) {
        hwnd = NULL;
    } else {
        av_log(s1, AV_LOG_ERROR,
               "Please use \"desktop\" or \"title=<windowname>\" to specify your target.\n");
        ret = AVERROR(EIO);
        goto error;
    }

    /* This will get the device context for the selected window, or if
     * none, the primary screen */
    source_hdc = GetDC(hwnd);
    if (!source_hdc) {
        WIN32_API_ERROR("Couldn't get window device context");
        ret = AVERROR(EIO);
        goto error;
    }
    bpp = GetDeviceCaps(source_hdc, BITSPIXEL);

    horzres = GetDeviceCaps(source_hdc, HORZRES);
    vertres = GetDeviceCaps(source_hdc, VERTRES);
    desktophorzres = GetDeviceCaps(source_hdc, DESKTOPHORZRES);
    desktopvertres = GetDeviceCaps(source_hdc, DESKTOPVERTRES);

    if (hwnd) {
        GetClientRect(hwnd, &virtual_rect);
        /* window -- get the right height and width for scaling DPI */
        virtual_rect.left   = virtual_rect.left   * desktophorzres / horzres;
        virtual_rect.right  = virtual_rect.right  * desktophorzres / horzres;
        virtual_rect.top    = virtual_rect.top    * desktopvertres / vertres;
        virtual_rect.bottom = virtual_rect.bottom * desktopvertres / vertres;
    } else {
        /* desktop -- get the right height and width for scaling DPI */
        virtual_rect.left = GetSystemMetrics(SM_XVIRTUALSCREEN);
        virtual_rect.top = GetSystemMetrics(SM_YVIRTUALSCREEN);
        virtual_rect.right = (virtual_rect.left + GetSystemMetrics(SM_CXVIRTUALSCREEN)) * desktophorzres / horzres;
        virtual_rect.bottom = (virtual_rect.top + GetSystemMetrics(SM_CYVIRTUALSCREEN)) * desktopvertres / vertres;
    }

    /* If no width or height set, use full screen/window area */
    if (!gdigrab->width || !gdigrab->height) {
        clip_rect.left = virtual_rect.left;
        clip_rect.top = virtual_rect.top;
        clip_rect.right = virtual_rect.right;
        clip_rect.bottom = virtual_rect.bottom;
    } else {
        clip_rect.left = gdigrab->offset_x;
        clip_rect.top = gdigrab->offset_y;
        clip_rect.right = gdigrab->width + gdigrab->offset_x;
        clip_rect.bottom = gdigrab->height + gdigrab->offset_y;
    }

    if (clip_rect.left < virtual_rect.left ||
            clip_rect.top < virtual_rect.top ||
            clip_rect.right > virtual_rect.right ||
            clip_rect.bottom > virtual_rect.bottom) {
            av_log(s1, AV_LOG_ERROR,
                    "Capture area (%li,%li),(%li,%li) extends outside window area (%li,%li),(%li,%li)",
                    clip_rect.left, clip_rect.top,
                    clip_rect.right, clip_rect.bottom,
                    virtual_rect.left, virtual_rect.top,
                    virtual_rect.right, virtual_rect.bottom);
            ret = AVERROR(EIO);
            goto error;
    }


    if (name) {
        av_log(s1, AV_LOG_INFO,
               "Found window %s, capturing %lix%lix%i at (%li,%li)\n",
               name,
               clip_rect.right - clip_rect.left,
               clip_rect.bottom - clip_rect.top,
               bpp, clip_rect.left, clip_rect.top);
    } else {
        av_log(s1, AV_LOG_INFO,
               "Capturing whole desktop as %lix%lix%i at (%li,%li)\n",
               clip_rect.right - clip_rect.left,
               clip_rect.bottom - clip_rect.top,
               bpp, clip_rect.left, clip_rect.top);
    }

    if (clip_rect.right - clip_rect.left <= 0 ||
            clip_rect.bottom - clip_rect.top <= 0 || bpp%8) {
        av_log(s1, AV_LOG_ERROR, "Invalid properties, aborting\n");
        ret = AVERROR(EIO);
        goto error;
    }

    dest_hdc = CreateCompatibleDC(source_hdc);
    if (!dest_hdc) {
        WIN32_API_ERROR("Screen DC CreateCompatibleDC");
        ret = AVERROR(EIO);
        goto error;
    }

    /* Create a DIB and select it into the dest_hdc */
    bmi.bmiHeader.biSize          = sizeof(BITMAPINFOHEADER);
    bmi.bmiHeader.biWidth         = clip_rect.right - clip_rect.left;
    bmi.bmiHeader.biHeight        = -(clip_rect.bottom - clip_rect.top);
    bmi.bmiHeader.biPlanes        = 1;
    bmi.bmiHeader.biBitCount      = bpp;
    bmi.bmiHeader.biCompression   = BI_RGB;
    bmi.bmiHeader.biSizeImage     = 0;
    bmi.bmiHeader.biXPelsPerMeter = 0;
    bmi.bmiHeader.biYPelsPerMeter = 0;
    bmi.bmiHeader.biClrUsed       = 0;
    bmi.bmiHeader.biClrImportant  = 0;
    hbmp = CreateDIBSection(dest_hdc, &bmi, DIB_RGB_COLORS,
            &buffer, NULL, 0);
    if (!hbmp) {
        WIN32_API_ERROR("Creating DIB Section");
        ret = AVERROR(EIO);
        goto error;
    }

    if (!SelectObject(dest_hdc, hbmp)) {
        WIN32_API_ERROR("SelectObject");
        ret = AVERROR(EIO);
        goto error;
    }

    /* Get info from the bitmap */
    GetObject(hbmp, sizeof(BITMAP), &bmp);

    st = avformat_new_stream(s1, NULL);
    if (!st) {
        ret = AVERROR(ENOMEM);
        goto error;
    }
    avpriv_set_pts_info(st, 64, 1, 1000000); /* 64 bits pts in us */

    gdigrab->frame_size  = bmp.bmWidthBytes * bmp.bmHeight * bmp.bmPlanes;
    gdigrab->header_size = sizeof(BITMAPFILEHEADER) + sizeof(BITMAPINFOHEADER) +
                           (bpp <= 8 ? (1 << bpp) : 0) * sizeof(RGBQUAD) /* palette size */;
    gdigrab->time_base   = av_inv_q(gdigrab->framerate);
    gdigrab->time_frame  = av_gettime() / av_q2d(gdigrab->time_base);

    gdigrab->hwnd       = hwnd;
    gdigrab->source_hdc = source_hdc;
    gdigrab->dest_hdc   = dest_hdc;
    gdigrab->hbmp       = hbmp;
    gdigrab->bmi        = bmi;
    gdigrab->buffer     = buffer;
    gdigrab->clip_rect  = clip_rect;

    gdigrab->cursor_error_printed = 0;

    if (gdigrab->show_region) {
        if (gdigrab_region_wnd_init(s1, gdigrab)) {
            ret = AVERROR(EIO);
            goto error;
        }
    }

    st->avg_frame_rate = av_inv_q(gdigrab->time_base);

    st->codecpar->codec_type = AVMEDIA_TYPE_VIDEO;
    st->codecpar->codec_id   = AV_CODEC_ID_BMP;
    st->codecpar->bit_rate   = (gdigrab->header_size + gdigrab->frame_size) * 1/av_q2d(gdigrab->time_base) * 8;

    return 0;

error:
    if (source_hdc)
        ReleaseDC(hwnd, source_hdc);
    if (dest_hdc)
        DeleteDC(dest_hdc);
    if (hbmp)
        DeleteObject(hbmp);
    if (source_hdc)
        DeleteDC(source_hdc);
    return ret;
}
...
/**
 * Grabs a frame from gdi (public device demuxer API).
 *
 * @param s1 Context from avformat core
 * @param pkt Packet holding the grabbed frame
 * @return frame size in bytes
 */
static int gdigrab_read_packet(AVFormatContext *s1, AVPacket *pkt)
{
    struct gdigrab *gdigrab = s1->priv_data;

    HDC        dest_hdc   = gdigrab->dest_hdc;
    HDC        source_hdc = gdigrab->source_hdc;
    RECT       clip_rect  = gdigrab->clip_rect;
    AVRational time_base  = gdigrab->time_base;
    int64_t    time_frame = gdigrab->time_frame;

    BITMAPFILEHEADER bfh;
    int file_size = gdigrab->header_size + gdigrab->frame_size;

    int64_t curtime, delay;

    /* Calculate the time of the next frame */
    time_frame += INT64_C(1000000);

    /* Run Window message processing queue */
    if (gdigrab->show_region)
        gdigrab_region_wnd_update(s1, gdigrab);

    /* wait based on the frame rate */
    for (;;) {
        curtime = av_gettime();
        delay = time_frame * av_q2d(time_base) - curtime;
        if (delay <= 0) {
            if (delay < INT64_C(-1000000) * av_q2d(time_base)) {
                time_frame += INT64_C(1000000);
            }
            break;
        }
        if (s1->flags & AVFMT_FLAG_NONBLOCK) {
            return AVERROR(EAGAIN);
        } else {
            av_usleep(delay);
        }
    }

    if (av_new_packet(pkt, file_size) < 0)
        return AVERROR(ENOMEM);
    pkt->pts = curtime;

    /* Blit screen grab */
    if (!BitBlt(dest_hdc, 0, 0,
                clip_rect.right - clip_rect.left,
                clip_rect.bottom - clip_rect.top,
                source_hdc,
                clip_rect.left, clip_rect.top, SRCCOPY | CAPTUREBLT)) {
        WIN32_API_ERROR("Failed to capture image");
        return AVERROR(EIO);
    }
    if (gdigrab->draw_mouse)
        paint_mouse_pointer(s1, gdigrab);

    /* Copy bits to packet data */

    bfh.bfType = 0x4d42; /* "BM" in little-endian */
    bfh.bfSize = file_size;
    bfh.bfReserved1 = 0;
    bfh.bfReserved2 = 0;
    bfh.bfOffBits = gdigrab->header_size;

    memcpy(pkt->data, &bfh, sizeof(bfh));

    memcpy(pkt->data + sizeof(bfh), &gdigrab->bmi.bmiHeader, sizeof(gdigrab->bmi.bmiHeader));

    if (gdigrab->bmi.bmiHeader.biBitCount <= 8)
        GetDIBColorTable(dest_hdc, 0, 1 << gdigrab->bmi.bmiHeader.biBitCount,
                (RGBQUAD *) (pkt->data + sizeof(bfh) + sizeof(gdigrab->bmi.bmiHeader)));

    memcpy(pkt->data + gdigrab->header_size, gdigrab->buffer, gdigrab->frame_size);

    gdigrab->time_frame = time_frame;

    return gdigrab->header_size + gdigrab->frame_size;
}
```

# DSHOW

DirectShow 是 windows 上基于 COM 的流媒体处理的开发包，与 DirectX 开发包一起发布。使用 DSHOW 在捕获的设备上有更多的选择，比如 USB 摄像机、声卡设备等注册了的设备都可以作为输入源

## ffmpeg 调用

ffmpeg 封装了 DSHOW 截图的相关的函数，调用和 GDI 截图的区别就是初始化的参数不同。

```C++
//DSHOW截屏初始化
m_pFormatContext = avformat_alloc_context();
if(nullptr == m_pFormatContext) {
    qDebug("avformat_alloc_context failed");
    break;
}

AVInputFormat * pInputFormat = av_find_input_format("dshow");
if(nullptr == pInputFormat) {
    qDebug("av_find_input_format failed");
    break;
}

AVDictionary * options = nullptr;
if(0 != avformat_open_input(&m_pFormatContext, "video=screen-capture-recorder", pInputFormat, &options)) {
    qDebug("avformat_open_input failed");
    break;
}
```

## ffmpeg 实现

相关实现代码在`FFmpeg/libavcodec/dshow.c`。通过调用`dshow_read_header()`完成初始化，`dshow_read_packet()`完成实际的截图。

## dshow_read_header

1. 判断输入参数、输入格式、帧率是否正确
2. 通过 CoCreateInstance() 创建 IGraphBuilder 对应的 Com 对象
3. 在设备列表中检索指定设备
4. 根据音频、视频设备分别设置参数以及打开对应设备
5. 创建 IGraphBuilder 的 Event、Mutex

## dshow_read_packet

1. 事件循环，Mutex 触发后开始解析
2. 取出 priv_data 中的 packet 并处理数据
3. 检查 DShow 的 media event

## 源码

```C++
static int dshow_read_header(AVFormatContext *avctx)
{
    struct dshow_ctx *ctx = avctx->priv_data;
    IGraphBuilder *graph = NULL;
    ICreateDevEnum *devenum = NULL;
    IMediaControl *control = NULL;
    IMediaEvent *media_event = NULL;
    HANDLE media_event_handle;
    HANDLE proc;
    int ret = AVERROR(EIO);
    int r;

    CoInitialize(0);

    if (!ctx->list_devices && !parse_device_name(avctx)) {
        av_log(avctx, AV_LOG_ERROR, "Malformed dshow input string.\n");
        goto error;
    }

    ctx->video_codec_id = avctx->video_codec_id ? avctx->video_codec_id
                                                : AV_CODEC_ID_RAWVIDEO;
    if (ctx->pixel_format != AV_PIX_FMT_NONE) {
        if (ctx->video_codec_id != AV_CODEC_ID_RAWVIDEO) {
            av_log(avctx, AV_LOG_ERROR, "Pixel format may only be set when "
                              "video codec is not set or set to rawvideo\n");
            ret = AVERROR(EINVAL);
            goto error;
        }
    }
    if (ctx->framerate) {
        r = av_parse_video_rate(&ctx->requested_framerate, ctx->framerate);
        if (r < 0) {
            av_log(avctx, AV_LOG_ERROR, "Could not parse framerate '%s'.\n", ctx->framerate);
            goto error;
        }
    }

    r = CoCreateInstance(&CLSID_FilterGraph, NULL, CLSCTX_INPROC_SERVER,
                         &IID_IGraphBuilder, (void **) &graph);
    if (r != S_OK) {
        av_log(avctx, AV_LOG_ERROR, "Could not create capture graph.\n");
        goto error;
    }
    ctx->graph = graph;

    r = CoCreateInstance(&CLSID_SystemDeviceEnum, NULL, CLSCTX_INPROC_SERVER,
                         &IID_ICreateDevEnum, (void **) &devenum);
    if (r != S_OK) {
        av_log(avctx, AV_LOG_ERROR, "Could not enumerate system devices.\n");
        goto error;
    }

    if (ctx->list_devices) {
        av_log(avctx, AV_LOG_INFO, "DirectShow video devices (some may be both video and audio devices)\n");
        dshow_cycle_devices(avctx, devenum, VideoDevice, VideoSourceDevice, NULL, NULL);
        av_log(avctx, AV_LOG_INFO, "DirectShow audio devices\n");
        dshow_cycle_devices(avctx, devenum, AudioDevice, AudioSourceDevice, NULL, NULL);
        ret = AVERROR_EXIT;
        goto error;
    }
    if (ctx->list_options) {
        if (ctx->device_name[VideoDevice])
            if ((r = dshow_list_device_options(avctx, devenum, VideoDevice, VideoSourceDevice))) {
                ret = r;
                goto error;
            }
        if (ctx->device_name[AudioDevice]) {
            if (dshow_list_device_options(avctx, devenum, AudioDevice, AudioSourceDevice)) {
                /* show audio options from combined video+audio sources as fallback */
                if ((r = dshow_list_device_options(avctx, devenum, AudioDevice, VideoSourceDevice))) {
                    ret = r;
                    goto error;
                }
            }
        }
    }
    if (ctx->device_name[VideoDevice]) {
        if ((r = dshow_open_device(avctx, devenum, VideoDevice, VideoSourceDevice)) < 0 ||
            (r = dshow_add_device(avctx, VideoDevice)) < 0) {
            ret = r;
            goto error;
        }
    }
    if (ctx->device_name[AudioDevice]) {
        if ((r = dshow_open_device(avctx, devenum, AudioDevice, AudioSourceDevice)) < 0 ||
            (r = dshow_add_device(avctx, AudioDevice)) < 0) {
            av_log(avctx, AV_LOG_INFO, "Searching for audio device within video devices for %s\n", ctx->device_name[AudioDevice]);
            /* see if there's a video source with an audio pin with the given audio name */
            if ((r = dshow_open_device(avctx, devenum, AudioDevice, VideoSourceDevice)) < 0 ||
                (r = dshow_add_device(avctx, AudioDevice)) < 0) {
                ret = r;
                goto error;
            }
        }
    }
    if (ctx->list_options) {
        /* allow it to list crossbar options in dshow_open_device */
        ret = AVERROR_EXIT;
        goto error;
    }
    ctx->curbufsize[0] = 0;
    ctx->curbufsize[1] = 0;
    ctx->mutex = CreateMutex(NULL, 0, NULL);
    if (!ctx->mutex) {
        av_log(avctx, AV_LOG_ERROR, "Could not create Mutex\n");
        goto error;
    }
    ctx->event[1] = CreateEvent(NULL, 1, 0, NULL);
    if (!ctx->event[1]) {
        av_log(avctx, AV_LOG_ERROR, "Could not create Event\n");
        goto error;
    }

    r = IGraphBuilder_QueryInterface(graph, &IID_IMediaControl, (void **) &control);
    if (r != S_OK) {
        av_log(avctx, AV_LOG_ERROR, "Could not get media control.\n");
        goto error;
    }
    ctx->control = control;

    r = IGraphBuilder_QueryInterface(graph, &IID_IMediaEvent, (void **) &media_event);
    if (r != S_OK) {
        av_log(avctx, AV_LOG_ERROR, "Could not get media event.\n");
        goto error;
    }
    ctx->media_event = media_event;

    r = IMediaEvent_GetEventHandle(media_event, (void *) &media_event_handle);
    if (r != S_OK) {
        av_log(avctx, AV_LOG_ERROR, "Could not get media event handle.\n");
        goto error;
    }
    proc = GetCurrentProcess();
    r = DuplicateHandle(proc, media_event_handle, proc, &ctx->event[0],
                        0, 0, DUPLICATE_SAME_ACCESS);
    if (!r) {
        av_log(avctx, AV_LOG_ERROR, "Could not duplicate media event handle.\n");
        goto error;
    }

    r = IMediaControl_Run(control);
    if (r == S_FALSE) {
        OAFilterState pfs;
        r = IMediaControl_GetState(control, 0, &pfs);
    }
    if (r != S_OK) {
        av_log(avctx, AV_LOG_ERROR, "Could not run graph (sometimes caused by a device already in use by other application)\n");
        goto error;
    }

    ret = 0;

error:

    if (devenum)
        ICreateDevEnum_Release(devenum);

    if (ret < 0)
        dshow_read_close(avctx);

    return ret;
}
...

static int dshow_read_packet(AVFormatContext *s, AVPacket *pkt)
{
    struct dshow_ctx *ctx = s->priv_data;
    AVPacketList *pktl = NULL;

    while (!ctx->eof && !pktl) {
        WaitForSingleObject(ctx->mutex, INFINITE);
        pktl = ctx->pktl;
        if (pktl) {
            *pkt = pktl->pkt;
            ctx->pktl = ctx->pktl->next;
            av_free(pktl);
            ctx->curbufsize[pkt->stream_index] -= pkt->size;
        }
        ResetEvent(ctx->event[1]);
        ReleaseMutex(ctx->mutex);
        if (!pktl) {
            if (dshow_check_event_queue(ctx->media_event) < 0) {
                ctx->eof = 1;
            } else if (s->flags & AVFMT_FLAG_NONBLOCK) {
                return AVERROR(EAGAIN);
            } else {
                WaitForMultipleObjects(2, ctx->event, 0, INFINITE);
            }
        }
    }

    return ctx->eof ? AVERROR(EIO) : pkt->size;
}
```

# DXGI

windows 8.1 以上的系统，则实现了一个 DXGI，集成在 DirectX 之中，是 DirectX 的一个子功能。DXGI 效率最高，不过对系统要求高。初始化都是固定的格式就是比较麻烦，[可以看看微软官方 demo](https://github.com/microsoftarchive/msdn-code-gallery-microsoft/tree/master/Official%20Windows%20Platform%20Sample/DXGI%20desktop%20duplication%20sample)，创建和查询各种接口，最终获取到 IDXGIOutputDuplication 接口。截屏的时候，使用 AcquireNextFrame() 函数获取当前桌面图像， GetFrameDirtyRects() 用来获取发生了变化的矩形区域。

## ffmpeg 调用

ffmpeg 没有对应的接口，需要自己拿到 RGB 数据后封装成 AVFrame 给后续的编码环节使用。

```C++
//初始化略
...

//创建对应的AVFrame
m_pFrame = av_frame_alloc();
if(nullptr == m_pFrame) return;
m_pFrame->width = m_srcWidth;
m_pFrame->height = m_srcHeight;
m_pFrame->format = AV_PIX_FMT_BGRA;
if(0 != av_frame_get_buffer(m_pFrame, 0)) return;

...
//将DXGI拿到的数据存到AVFrame
m_pFrame->data[0] = directx.buffer;
```
