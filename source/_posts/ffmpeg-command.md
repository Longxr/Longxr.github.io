---
title: ffmpeg相关常用命令及可选参数
categories:
  - ffmpeg
tags:
  - ffmpeg
  - ffplay
  - ffprobe
date: 2019-07-05 14:48:48
---

<Excerpt in index | 摘要> 
ffmpeg相关常用命令及可选参数, 包括ffmpeg.exe、 ffplay.exe、 ffprobe.exe, 程序下载链接[戳这里](https://ffmpeg.zeranoe.com/builds/), 选择static即可 <!-- more -->
<The rest of contents | 余下全文>

## ffmpeg

###  ffmpeg 常用命令
```
//列出所有设备
ffmpeg -list_devices true -f dshow -i dummy

//提取h264裸流
ffmpeg -i video.mp4 -codec copy -f h264 video.h264
ffmpeg -i video.mp4 -an -vcodec copy video.h264

//提取aac裸流
ffmpeg -i video.mp4 -vn -acodec copy audio.aac

//桌面采集录制
ffmpeg -f gdigrab -video_size 1920x1080 -t 10 -r 30 -i desktop -vcodec libx264 x264.mp4
ffmpeg -f gdigrab -video_size 1920x1080 -t 40 -r 30 -i desktop -vcodec h264_qsv x264.mp4
ffmpeg -f dshow -i video="screen-capture-recorder" -f dshow -i audio="virtual-audio-capturer" -t 40 -r 30 -vcodec libx264 output.mp4

//桌面采集推流
ffmpeg -f gdigrab -i desktop -vcodec libx264 -preset:v ultrafast -tune:v zerolatency -f flv rtmp://server/live/streamName

//桌面+声卡采集推流 (virtual-audio-capturer需要安装相关程序，https://sourceforge.net/projects/screencapturer/files/)
ffmpeg -f dshow -i video="screen-capture-recorder" -f dshow -i audio="virtual-audio-capturer" -pix_fmt yuv420p -vcodec libx264 -acodec aac -s 1280x720 -r 25 -q 10 -ar 44100 -ac 2 -tune zerolatency -preset ultrafast -f flv rtmp://server/live/streamName

//文件源推流
ffmpeg -re -i video.mp4 -c copy -f flv rtmp://server/live/streamName

//直播流录制
ffmpeg -i rtmp://server/live/streamName -c copy dump.flv

//mkv转mp4
ffmpeg -i video.mkv -codec copy video.mp4

//视频切割
ffmpeg -ss 00:01:00 -t 00:01:03 -y -i video.mp4 -vcodec copy -acodec copy cutting.mp4 

//视频缩放, iw是输入的宽度, iw/2就是一半;-1 为保持宽高比
ffmpeg -i video.mp4 -vf scale=iw/2:-1 output.mp4

//视频截取, 从[80,60]的位置开始, 截取200x100的视频
ffmpeg -i video.mp4 -filter:v "crop=80:60:200:100" -c:a copy output-200x100.mp4

//视频截取, 右下角1/4
ffmpeg -i video.mp4 -filter:v "crop=in_w/2:in_h/2:in_w/2:in_h/2" -c:a copy output-1-4.mp4

//指定时间截图
ffmpeg -ss 00:00:30 -i video.mp4 -y -f image2 -vframes 1 image.png

//每隔20S截图
ffmpeg -i video.mp4 -f image2 -vf fps=fps=1/20 out%d.png

//将视频转为gif
ffmpeg -i video.mp4 -ss 0:0:30 -t 10 -s 320x240 -pix_fmt rgb24 output.gif

//图片转换为视频
ffmpeg -f image2 -i out%4d.png -r 25 video.mp4

//添加图片水印,colorkey将图片中指定颜色变为透明
ffmpeg -i video.mp4 -i test.png -filter_complex "[1:v]colorkey=0xFFFFFF:0.6:1.0[ckout];[0:v][ckout]overlay=x=W-w-10:y=0[out]" -map "[out]" -movflags faststart drawimage.mp4

//添加文字水印，simsun.ttc是宋体，字体可更换
ffmpeg.exe  -i video.mp4 -vf "drawtext=fontfile=simsun.ttc: text='文字水印':x=100:y=100:fontsize=28:fontcolor=red:shadowy=2" drawtext.mp4

//内嵌字幕(嵌入视频中)
ffmpeg -i video.mp4 -vf ass=subtitles.ass output.mp4

//内挂字幕(视频字幕还是分离的，会导致进度条不能拖动)
ffmpeg -i video.mp4 -i subtitles.ass -c:s mov_text -c:v copy -c:a copy output.mp4

//加密视频
ffmpeg -i video.mp4 -vcodec copy -acodec copy -encryption_scheme cenc-aes-ctr -encryption_key 76a6c65c5ea76 2046bd749a2e632ccbb -encryption_kid a7e61c373e219033c21091fa607bf3b8 video_encrypted.mp4
//播放加密视频
ffplay video_encrypted.mp4 -decryption_key 76a6c65c5ea762046bd749a2e632ccbb

//视频拼接
ffmpeg -f concat -i filelist.txt -c copy output.mp4
```

### ffmpeg可选参数

#### 通用选项

| 参数 | 说明 |
| --- | --- |
| -L | license |
| -h | 帮助 |
| -fromats | 显示可用的格式，编解码的，协议的... |
| -f | 强迫采用格式fmt |
| -i filename | 输入文件 |
| -y | 覆盖输出文件 |
| -t duration | 设置纪录时间 hh:mm:ss[.xxx]格式的记录时间也支持 |
| -ss position | 搜索到指定的时间 [-]hh:mm:ss[.xxx]的格式也支持 |
| -title string | 设置标题 |
| -author string | 设置作者 |
| -copyright string | 设置版权 |
| -comment string | 设置评论 |
| -target type | 设置目标文件类型(vcd,svcd,dvd) 所有的格式选项（比特率，编解码以及缓冲区大小）自动设置 |
| -hq | 激活高质量设置 |
| -itsoffset offset | 置以秒为基准的时间偏移，该选项影响所有后面的输入文件，定义一个正偏移意味着相应的流被延迟了 offset秒 |

#### 视频选项

| 参数 | 说明 |
| --- | --- |
| -b bitrate | 设置比特率，缺省200kb/s |
| -r fps | 设置帧频 缺省25 |
| -s size | 设置帧大小 格式为WXH 缺省160X128 |
| -aspect aspect | 设置横纵比 4:3 16:9 或 1.3333 1.7777 |
| -croptop size | 设置顶部切除带大小 像素单位 |
| -cropbottom size | 同上，方向不同 |
| –cropleft size | 同上，方向不同 |
| –cropright size | 同上，方向不同 |
| -padtop size | 设置顶部补齐的大小 像素单位 |
| -padbottom size | 同上，方向不同 |
| –padleft size | 同上，方向不同 |
| –padright size | 同上，方向不同 |
| –padcolor color | 设置补齐条颜色(hex,6个16进制的数，红:绿:兰排列，比如 000000代表黑色) |
| -vn | 不做视频记录 |
| -bt tolerance | 设置视频码率容忍度kbit/s |
| -maxrate bitrate | 设置最大视频码率容忍度 |
| -minrate bitreate | 设置最小视频码率容忍度 |
| -bufsize size | 设置码率控制缓冲区大小 |
| -bufsize size | 设置码率控制缓冲区大小 |
| -vcodec codec | 强制使用codec编解码方式。如果用copy表示原始编解码数据必须被拷贝 |
| -sameq | 使用同样视频质量作为源（VBR） |
| -pass n | 选择处理遍数（1或者2）。两遍编码非常有用。第一遍生成统计信息，第二遍生成精确的请求的码率 |
| -passlogfile file | 选择两遍的纪录文件名为file |


#### 高级视频选项

| 参数 | 说明 |
| --- | --- |
| -g gop_size | 设置图像组大小 |
| -intra | 仅适用帧内编码 |
| -qscale q | 使用固定的视频量化标度(VBR) |
| -qmin q | 最小视频量化标度(VBR) |
| -qmax q | 最大视频量化标度(VBR) |
| -qdiff q | 量化标度间最大偏差 (VBR) |
| -qblur blur | 视频量化标度柔化(VBR) |
| -qcomp compression | 视频量化标度压缩(VBR) |
| -rc_init_cplx complexity | 一遍编码的初始复杂度 |
| -b_qfactor factor | 在p和b帧间的qp因子 |
| -i_qfactor factor | 在p和i帧间的qp因子 |
| -b_qoffset offset | 在p和b帧间的qp偏差 |
| -i_qoffset offset | 在p和i帧间的qp偏差 |
| -rc_eq equation | 设置码率控制方程 默认tex^qComp |
| -rc_override override | 特定间隔下的速率控制重载 |
| -me method | 设置运动估计的方法 可用方法有 zero phods log x1 epzs(缺省) full |
| -dct_algo algo | 设置dct的算法 可用的有 0 FF_DCT_AUTO 缺省的DCT 1 FF_DCT_FASTINT 2 FF_DCT_INT 3 FF_DCT_MMX 4 FF_DCT_MLIB 5 FF_DCT_ALTIVEC |
| -idct_algo algo | 设置idct算法。可用的有 0 FF_IDCT_AUTO 缺省的IDCT 1 FF_IDCT_INT 2 FF_IDCT_SIMPLE 3 FF_IDCT_SIMPLEMMX 4 FF_IDCT_LIBMPEG2MMX 5 FF_IDCT_PS2 6 FF_IDCT_MLIB 7 FF_IDCT_ARM 8 FF_IDCT_ALTIVEC 9 FF_IDCT_SH4 10 FF_IDCT_SIMPLEARM |
| -er n | 设置错误残留为n 1 FF_ER_CAREFULL 缺省 2 FF_ER_COMPLIANT 3 FF_ER_AGGRESSIVE 4 FF_ER_VERY_AGGRESSIVE |
| -ec bit_mask | 设置错误掩蔽为bit_mask,该值为如下值的位掩码 1 FF_EC_GUESS_MVS (default=enabled) 2 FF_EC_DEBLOCK (default=enabled) |
| -bf frames | 使用frames B 帧，支持mpeg1,mpeg2,mpeg4 |
| -mbd mode | 宏块决策 0 FF_MB_DECISION_SIMPLE 使用mb_cmp 1 FF_MB_DECISION_BITS 2 FF_MB_DECISION_RD |
| -4mv | 使用4个运动矢量 仅用于mpeg4 |
| -part | 使用数据划分 仅用于mpeg4 |
| -bug param | 绕过没有被自动监测到编码器的问题 |
| -strict strictness | 跟标准的严格性 |
| -aic | 使能高级帧内编码 h263+ |
| -umv | 使能无限运动矢量 h263+ |
| -deinterlace | 不采用交织方法 |
| -interlace | 强迫交织法编码仅对mpeg2和mpeg4有效。当你的输入是交织的并且你想要保持交织以最小图像损失的时候采用该选项。可选的方法是不交织，但是损失更大 |
| -psnr | 计算压缩帧的psnr |
| -vstats | 输出视频编码统计到vstats_hhmmss.log |
| -vhook module | 插入视频处理模块 module 包括了模块名和参数，用空格分开 |

#### 音频选项

| 参数 | 说明 |
| --- | --- |
| -ab bitrate  | 设置音频码率 |
| -ar freq | 设置音频采样率 |
| -ac channels | 设置通道 缺省为1 |
| -an | 不使能音频纪录 |
| -acodec codec | 使用codec编解码 |

#### 音频/视频捕获选项

| 参数 | 说明 |
| --- | --- |
| -vd device | 设置视频捕获设备。比如/dev/video0 |
| -vc channel | 设置视频捕获通道 DV1394专用 |
| -tvstd standard | 置电视标准 NTSC PAL(SECAM) |
| -dv1394 | 设置DV1394捕获 |
| -av device | 设置音频设备 比如/dev/dsp |

#### 高级选项

| 参数 | 说明 |
| --- | --- |
| -map file:stream | 设置输入流映射 |
| -debug | 打印特定调试信息 |
| -benchmark | 为基准测试加入时间 |
| -hex | 倾倒每一个输入包 |
| -bitexact | 仅使用位精确算法 用于编解码测试 |
| -ps size | 设置包大小，以bits为单位 |
| -re | 以本地帧频读数据，主要用于模拟捕获设备 |
| -loop | 循环输入流（只工作于图像流，用于ffserver测试） |

## ffplay

### ffplay常用命令
```
//从30s开始播放10s，播放完退出
ffplay -ss 00:00:30 -t 10 -autoexit video.mp4

//指定播放窗口大小为320x180
ffplay -x 320 -y 180 video.mp4

//音频波形可视化
ffplay -showmode 1 audio.aac
```

### ffplay可选参数

| 参数 | 说明 |
| --- | --- |
| -x width | 强制以 "width" 宽度显示 |
| -y height | 强制以 "height" 高度显示 |
| -an | 禁止音频 |
| -vn | 禁止视频 |
| -ss pos | 跳转到指定的位置(秒) |
| -t duration | 播放 "duration" 秒音/视频 |
| -bytes | 按字节跳转 |
| -nodisp | 禁止图像显示(只输出音频) |
| -f fmt | 强制使用 "fmt" 格式 |
| -window_title title | 设置窗口标题(默认为输入文件名) |
| -loop number | 循环播放 "number" 次(0将一直循环) |
| -showmode mode | 设置显示模式,可选mode： 0, 显示视频; 1, 显示音频波形; 2, 显示音频频带。默认为0, 播放时按w键可切换mode |
| -i input_file | 指定输入文件 |
| -sync type | 设置主时钟为音频、视频、或者外部。默认为音频。主时钟用来进行音视频同步 |
| -threads count | 设置线程个数 |
| -autoexit | 播放完成后自动退出 |
| -exitonkeydown | 任意键按下时退出 |
| -exitonmousedown | 任意鼠标按键按下时退出 |
| -acodec codec_name | 强制指定音频解码器为 "codec_name" |
| -vcodec codec_name | 强制指定视频解码器为 "codec_name" |
| -scodec codec_name | 强制指定字幕解码器为 "codec_name" |


### ffplay快捷键

| 参数 | 说明 |
| --- | --- |
| q | 退出 |
| f | 全屏 |
| p | 暂停 |
| w | 切换显示模式(视频/音频波形/音频频带) |
| s | 步进到下一帧 |
| left/right | 快退/快进 10 秒 |
| down/up | 快退/快进 1 分钟 |
| page down/page up | 跳转到前一章/下一章(如果没有章节，快退/快进 10 分钟) |
| mouse click | 跳转到鼠标点击的位置(根据鼠标在显示窗口点击的位置计算百分比) |

## ffprobe

### ffprobe常用命令
```
//打印视频format信息，代码里就是av_dump_format()函数
ffprobe -show_format video.mp4

//以json格式输出"streams"音视频编码信息，还有“format”音视频封包信息
ffprobe -v quiet -show_format -show_streams -print_format json video.mp4
```

### ffprobe可选参数

| 参数 | 说明 |
| --- | --- |
| -show_packets | [PACKET]标签的多媒体信息 |
| -show_data | packets多媒体数据 |
| -show_format | 多媒体封装格式 |
| -show_frames | 视频帧信息 |
| -print_format | 信息输出格式，支持xml、csv、json、flat、ini |
| -select_streams | 参数可以是a、v、s分别表示只查看音频、视频、字幕 |

## 参考链接
* [ffmpeg参数中文详细解释](https://blog.csdn.net/leixiaohua1020/article/details/12751349)
* [FFmpeg常用命令及参数](https://www.jianshu.com/p/11b6c4eb9e49)
* [FFmpeg命令行工具学习(二)：播放媒体文件的工具ffplay](https://www.cnblogs.com/renhui/p/8458802.html)

