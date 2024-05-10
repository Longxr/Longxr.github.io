---
title: ffmpeg音频采集、编码
categories:
  - Qt
tags:
  - ffmpeg
  - Audio 
  - Capture
  - AAC
date: 2018-08-17 18:00:46
---

<Excerpt in index | 摘要> 
不同于视频采集一帧编码一帧，音频采集和编码之间稍微复杂一些，记录下相关内容<!-- more -->
<The rest of contents | 余下全文>

### 音频采集
在windows上获取音频的方法，主要尝试了2种，效果差不多。一种是通过ffmpeg的dshow获取，另一种是直接从windows的Core Audio API 来获取。通过这两种方式采集到的音频的采样率都是当前声音播放的扬声器的采样率。

#### ffmpeg dshow 采集
使用dshow抓屏需要安装抓屏软件：[screen-capture-recorder](https://sourceforge.net/projects/screencapturer/)。
##### 在命令行用dshow:
```
ffmpeg -f dshow -i video="screen-capture-recorder" -f dshow -i audio="virtual-audio-capturer" -t 30 -r 20 -vcodec libx264 output.mp4
```
##### 在代码中用dshow:
```
//视频采集
AVInputFormat *ifmt=av_find_input_format("dshow");
 if(avformat_open_input(&pFormatCtx,"video=screen-capture-recorder",ifmt,NULL)!=0){
  qDebug() << "Couldn't open input stream.";
  return -1;
 }

//音频采集
AVInputFormat *ifmt=av_find_input_format("dshow");
 if(avformat_open_input(&pFormatCtx,audio="virtual-audio-capturer",ifmt,NULL)!=0){
  qDebug() << "Couldn't open input stream.";
  return -1;
 }
```
具体采集可以参考[ffmpeg实现录屏+录音](https://blog.csdn.net/dancing_night/article/details/46698853)，懒得贴代码了。

#### Core Audio 音频采集
Core Audio不支持XP，只可以在Vista以上（包括Vista）的操作系统中才能使用，主要用来取代Wave系列API函数和DirectSound。

#####
具体采集可以参考[windows音频声卡采集](https://blog.csdn.net/su_vast/article/details/78317584)。主要说下采集到的buffer中framesAvailable就是采集一次包含的采样点多少了，同AVFrame中的nb_samples：
```
hr = m_CACaptureClient->GetBuffer(&pData, &framesAvailable, &flags, NULL, NULL);
        if (SUCCEEDED(hr))
        {
            if (framesAvailable!=0)
            {
                if (flags & AUDCLNT_BUFFERFLAGS_SILENT)
                {
                    pData = NULL;
                }
                else
                {
                    //Copy data from the audio engine buffer to the output buffer.
                    int nDataLen = framesAvailable*m_CAFrameSize;

                    AVFrame *frame;
                    frame = av_frame_alloc();
                    frame->format = m_SrcParams.sample_fmt;
                    frame->nb_samples = framesAvailable;
                    frame->channels = m_SrcParams.channels;
                    frame->channel_layout = av_get_default_channel_layout(m_SrcParams.channels);
                    frame->sample_rate = m_SrcParams.sample_rate;

                    av_frame_get_buffer(frame, 1);
                    memcpy(frame->data[0], pData, nDataLen);

                    EncoderData *node = new EncoderData();
                    node->type = DATA_TYPE_AUDIO;
                    node->frame = frame;

//                    fwrite(pData, 1, nDataLen, pcmFp);

                    async_queue_push(m_AvailPtr, (void *)node);

                    CalcCapRate();
                }
            }
        }
```
### 音频格式中的Plane
默认用ffmpeg采集到的格式是AV_SAMPLE_FMT_S16，但是AAC编码要的又是AV_SAMPLE_FMT_FLTP，中间需要通过swr_convert来转换。
```
enum AVSampleFormat {
    AV_SAMPLE_FMT_NONE = -1,
    AV_SAMPLE_FMT_U8,          ///< unsigned 8 bits
    AV_SAMPLE_FMT_S16,         ///< signed 16 bits
    AV_SAMPLE_FMT_S32,         ///< signed 32 bits
    AV_SAMPLE_FMT_FLT,         ///< float
    AV_SAMPLE_FMT_DBL,         ///< double

    AV_SAMPLE_FMT_U8P,         ///< unsigned 8 bits, planar
    AV_SAMPLE_FMT_S16P,        ///< signed 16 bits, planar
    AV_SAMPLE_FMT_S32P,        ///< signed 32 bits, planar
    AV_SAMPLE_FMT_FLTP,        ///< float, planar
    AV_SAMPLE_FMT_DBLP,        ///< double, planar
    AV_SAMPLE_FMT_S64,         ///< signed 64 bits
    AV_SAMPLE_FMT_S64P,        ///< signed 64 bits, planar

    AV_SAMPLE_FMT_NB           ///< Number of sample formats. DO NOT USE if linking dynamically
};
```

1. 无论是不是带P的数据总量是相同的.
2. 带P的格式是左右声道分开存储的，data[0]：LLLLLLLLLLLLL....; data[1]: RRRRRRRRR...
3. 不带P的格式是交替存储的，只在data[0] 有数据，data[0] :LRLRLRLRLRLRLRLRLRLR....

### AAC编码
####  AAC帧时长
一个AAC原始帧包含某段时间内1024个采样点相关数据（mp3则包含1152个采样点）。音频帧的播放时间 = 一个AAC帧对应的采样样本的个数 / 采样频率(单位为s)。

1. 采样率(sample rate)为 44100Hz，表示每秒 44100个采样点, 根据公式，当前一帧的播放时间 = 1024 * 1000000/44100= 22.32ms(单位为ms)

2. 采样率为48000Hz，根据公式，当前一帧的播放时间 = 1024 * 1000000/48000= 21.32ms(单位为ms)

#### AAC帧数据大小
AAC帧一帧包含1024个采样点，数据量大小 = 一个AAC帧对应的采样样本的个数 * 通道数* 数据位数/8。采集数据为双通道，16位数据时，根据公式， 一帧数据量大小 = 1024*2 *16/8 = 4096。

**在采集采样率和编码采样率相同的情况下（就是不需要重采样），通道数，数据位数一般不会改变，那么采集的数据大小和编码的数据大小是一样的。也就是说，当采集了4096个字节的数据后，再送去给编码器编码一帧AAC帧，不同的采样率只是会改变每秒钟的AAC帧的数量。**

1. 采样率(sample rate)为 44100Hz，1秒钟有：44100 / 1024 = 43.066帧

2. 采样率为48000Hz，1秒钟有：48000 / 1024 = 48000 / 1024 = 46.875帧

所以，如果自己弄一个音频缓冲buffer，就需要每满4096字节就编码一帧，更简单的方式是用ffmpeg提供的AVAudioFifo，每次读取写入都是根据采样点的个数来的：
```
    av_audio_fifo_write(m_pAudioFifo, (void **)frame->data, frame->nb_samples);

    av_frame_free(&frame);
    RELEASE_CLASS(node);

    while(av_audio_fifo_size(m_pAudioFifo) >= m_inputSamples)
    {
        av_audio_fifo_read(m_pAudioFifo, (void **)m_pInputData, m_inputSamples);
        ...
    }
```
#### AAC的ADTS
ADTS 头中相对有用的信息 采样率、声道数、帧长度。想想也是，我要是解码器的话，你给我一堆得AAC音频ES流我也解不出来。每一个带ADTS头信息的AAC流会清晰的告送解码器他需要的这些信息。

这里有个坑爹的情况，ffmpeg的2.5版本，编码出的AAC帧好像是带有ADTS的，但是最新的ffmpeg4.0没有ADTS，直接把编码的AAC帧扔给播放器是没有声音的，需要你自己加...想要了解ADTS相关信息的可以[看看这里](https://blog.csdn.net/andyhuabing/article/details/40983423)，我就贴下从gayhub上翻到的添加ADTS的相关代码：
```
#define ADTS_HEADER_SIZE 7

const int avpriv_mpeg4audio_sample_rates[16] = {
    96000, 88200, 64000, 48000, 44100, 32000,
    24000, 22050, 16000, 12000, 11025, 8000, 7350
};

const uint8_t ff_mpeg4audio_channels[8] = {
    0, 1, 2, 3, 4, 5, 6, 8
};

static int GetSampleIndex(int sample_rate)
{
    for (int i = 0; i < 16; i++)
    {
        if (sample_rate == avpriv_mpeg4audio_sample_rates[i])
        {
            return i;
        }
    }
    return -1;
}

void AudioEncoder::WriteADTSHeader(int Size, int sample_rate,int channels)
{
    if (ADTSHeader == nullptr)
    {
        ADTSHeader = (char*)av_malloc(ADTS_HEADER_SIZE);
    }
    memset(ADTSHeader,0, ADTS_HEADER_SIZE);

    int length = ADTS_HEADER_SIZE + Size;
    length &= 0x1FFF;

    int sample_index = GetSampleIndex(sample_rate);
    int channel = 0;

    if (channels < (int)FF_ARRAY_ELEMS(ff_mpeg4audio_channels))
        channel = ff_mpeg4audio_channels[channels];

    ADTSHeader[0] = (char)0xff;
    ADTSHeader[1] = (char)0xf1;
    ADTSHeader[2] = (char)(0x40 | (sample_index << 2) | (channel >> 2));
    ADTSHeader[3] = (char)((channel & 0x3) << 6 | (length >> 11));
    ADTSHeader[4] = (char)(length >> 3) & 0xff;
    ADTSHeader[5] = (char)(((length & 0x7) << 5) & 0xff) | 0x1f;
    ADTSHeader[6] = (char)0xfc;
}

int AudioEncoder::ADTS(AVPacket *src, AVPacket **des)
{
    if (src == nullptr) {
        return -1;
    }
    if (des == nullptr) {
        return -1;
    }

    AVPacket *adtsPacket = av_packet_alloc();
    av_init_packet(adtsPacket);
    av_new_packet(adtsPacket, src->size + ADTS_HEADER_SIZE);
    WriteADTSHeader(src->size, m_pCoderCtx->sample_rate, m_pCoderCtx->channels);
    memcpy(adtsPacket->data, ADTSHeader, ADTS_HEADER_SIZE);
    memcpy(adtsPacket->data + ADTS_HEADER_SIZE, src->data, src->size);

    adtsPacket->pts = src->pts;
    adtsPacket->dts = src->dts;
    adtsPacket->duration = src->duration;
    adtsPacket->flags = src->flags;
    adtsPacket->stream_index = src->stream_index;
    adtsPacket->pos = src->pos;

    if (*des == src)
    {
        av_packet_unref(src);
        av_packet_move_ref(*des, adtsPacket);
    }
    else if (*des != nullptr)
    {
        av_packet_move_ref(*des, adtsPacket);
    }
    else
    {
        *des = adtsPacket;
    }

    return 0;
}
```
使用方法就是在初始化的时候调用下AudioEncoder::WriteADTSHeader；编码出来一个AAC帧的packet就调用下AudioEncoder::ADTS(packet, *packet);  播放器就可以愉快的播放啦~~

### 重采样
先说下我碰到的问题，我的需求是将声卡采集后传输到另一个设备上播放，用swr_convert重采样的时候碰到个坑，我设置声卡的采样率为44100Hz的时候，如果编码采样率设置为48000Hz，每次要是扔给编码器4096的数据的话，编码出来的AAC帧拿去播放时间会变长，实时采集再播放的时候声音就会越来越慢，但是我也没有什么buffer存着数据，后来发现：
```
/** Convert audio.
 *
 * in and in_count can be set to 0 to flush the last few samples out at the
 * end.
 *
 * If more input is provided than output space, then the input will be buffered.
 * You can avoid this buffering by using swr_get_out_samples() to retrieve an
 * upper bound on the required number of output samples for the given number of
 * input samples. Conversion will run directly without copying whenever possible.
 *
 * @param s         allocated Swr context, with parameters set
 * @param out       output buffers, only the first one need be set in case of packed audio
 * @param out_count amount of space available for output in samples per channel
 * @param in        input buffers, only the first one need to be set in case of packed audio
 * @param in_count  number of input samples available in one channel
 *
 * @return number of samples output per channel, negative value on error
 */
int swr_convert(struct SwrContext *s, uint8_t **out, int out_count,
                                const uint8_t **in , int in_count);
```
注释说到，如果提供的输入多于输出空间，则输入将被缓冲...应该是缓冲在这里了Orz...打印了下swrcontext的buffer：
```
int fifo_size = swr_get_out_samples(m_pSwrCtx, 0);
qDebug() << "swr_get_out_samples"  << fifo_size;
```
确实在涨....emmm

看了下其他博客的[重采样](https://blog.csdn.net/dancing_night/article/details/45642107)，感觉是解码了AAC帧再重新采样编码，我的情况是想采集了(采集的帧只有448个采样点) 直接重采样编码，不想编码完了再重采样一遍。

回到上面关于数据大小的讨论，在采集44100->编码44100、采集48000->编码48000的情况下，采集和编码的数据量是相同的，在用swr_convert的时候，in_count、out_count相等就行：
```
swr_convert(m_pSwrCtx, m_pDesFrame->data, m_pCoderCtx->frame_size, (const uint8_t**)m_pConvertData, m_pCoderCtx->frame_size);
```
那么，在44100->48000的情况下，out_count由于AAC帧需要，1024不变，设in_count为x，得到个等式：
```
44100/x = 48000 /1024  =>  x = 44100*1024/48000 =>  x = 940.8
```
差不多就取个941，由于不是整数，out_count也要设置比1024大点：
```
swr_convert(m_pSwrCtx, m_pDesFrame->data, 1040, (const uint8_t**)m_pInputData, 941);
```
swr的buffer差不多保持在38左右，不是很明白为啥是这个值，但是需求基本达到了.

不过反过来就不行了，48000->44100要是把in_count直接改成1114的话，程序会崩溃_(:з)∠)_

### 参考链接
- [windows音频声卡采集](https://blog.csdn.net/su_vast/article/details/78317584)
- [新版ffmpeg PCM编码到AAC](https://blog.csdn.net/venice0708/article/details/80783870)
- [最简单的基于FFmpeg的AVDevice例子（屏幕录制)](https://blog.csdn.net/leixiaohua1020/article/details/39706721)
- [ffmpeg实现录屏+录音](https://blog.csdn.net/dancing_night/article/details/46698853)
- [AAC ADTS格式分析](https://blog.csdn.net/andyhuabing/article/details/40983423)
- [ffmpeg关于音频的总结(一)](https://blog.csdn.net/zhuweigangzwg/article/details/51499123)
- [ffmpeg音频转码，采用swr_convert重取样](https://blog.csdn.net/dancing_night/article/details/45642107)
