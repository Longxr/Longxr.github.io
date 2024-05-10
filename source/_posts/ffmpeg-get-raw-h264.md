---
title: ffmpeg从MP4中提取H264裸流（新旧API对比）
categories:
  - ffmpeg
tags:
  - ffmpeg
  - H264
  - MP4
date: 2019-07-03 16:40:01
---

<Excerpt in index | 摘要> 
ffmpeg从MP4中提取H264裸流，新旧API对比<!-- more -->
<The rest of contents | 余下全文>

## 问题
在分离mpe中的H264码流的时候，如果使用ffmpeg在命令行操作非常简单，` ffmpeg -i video.mp4 -codec copy -f h264 video.h264`，这一句就可以将MP4中的H264裸流提取出来了。
尝试着想用代码来实现这一功能，发现将`av_read_frame()`得到的AVPacket存入文件，得到的文件用PotPlayer是不能播放的。通过查找相关内容发现，MP4格式需要使用FFMPEG中的名为"h264_mp4toannexb"的bitstream filter 来处理AVPacket，下面是新旧API在实现上的不同部分。

## 老版本API
```
AVBitStreamFilterContext* h264bsfc =  av_bitstream_filter_init("h264_mp4toannexb"); 

AVPacket packet;
while( av_read_frame(format_ctx_, &packet) >= 0 ) {
    if( packet.stream_index == video_stream_index ) {
        uint8_t* outbuf = nullptr;
        int outlen = 0;
        av_bitstream_filter_filter(h264bsfc, codec_ctx_, NULL, 
            &outbuf, &outlen, packet.data, packet.size, 0);
        fwrite(packet.data, packet.size, 1, fp);
        if(outbuf){
            av_free(outbuf);
        }
    }

    av_free_packet(&packet);
}

av_bitstream_filter_close(h264bsfc);
```

## 新版本API
```
AVBSFContext * h264bsfc;
const AVBitStreamFilter * filter = av_bsf_get_by_name("h264_mp4toannexb");
ret = av_bsf_alloc(filter, &h264bsfc);
avcodec_parameters_copy(h264bsfc->par_in, input_fmt_ctx->streams[video_stream_index]->codecpar);
av_bsf_init(h264bsfc);

AVPacket* packet = av_packet_alloc();
while( av_read_frame(format_ctx_, packet) >= 0 ) {
    if( packet.stream_index == video_stream_index ) {
		ret = av_bsf_send_packet(h264bsfc, packet);
		if(ret < 0) qDebug("av_bsf_send_packet error");

		while ((ret = av_bsf_receive_packet(h264bsfc, packet)) == 0) {
	        fwrite(packet->data, packet->size, 1, fp);
	    }
	}

	av_packet_unref(packet);
}

av_packet_free(&packet);
av_bsf_free(&h264bsfc);
```

## 参考链接
* [使用FFMPEG类库分离出多媒体文件中的H.264码流](https://blog.csdn.net/leixiaohua1020/article/details/11800877)
* [用ffmpeg从MP4提取H264裸流](https://xilixili.net/2018/08/20/ffmpeg-got-raw-h264/)


