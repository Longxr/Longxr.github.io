---
title: ffmpeg将多个flv文件合成为mp4（python版）
categories:
  - 音视频
tags:
  - ffmpeg
  - python
  - flv
date: 2020-03-20 20:58:37
---

<Excerpt in index | 摘要> 
视频文件归档，将flv合并为mp4 <!-- more -->
<The rest of contents | 余下全文>

## 需求
直播生成的flv片段需要做个归档，把指定的文件夹中的flv合并成一个mp4，简单的转码合并操作直接用命令行调用来实现即可。

# 注意事项
* flv文件直接合并生成mp4的话只有第一个flv的内容才能播放，需要先转换成ts再合成mp4
* 使用的第三方库ffmpy，只是简单封装了命令行参数，ffmpeg还是需要自己另外安装，并配置环境变量的
* `-loglevel quiet`是调用ffmpeg不输出日志的选项，需要日志的话可以不加

## 代码
```
# coding=utf-8
import logging
import os
import shutil
import sys
from time import perf_counter

from ffmpy import FFmpeg
from natsort import natsorted

logger = logging.getLogger(__name__)


def video_convert(in_path, out_path):
    FFmpeg(inputs={in_path: None}, outputs={
           out_path: '-loglevel quiet -c copy -bsf:v h264_mp4toannexb -f mpegts'}).run()


def del_path(path):
    if not os.path.exists(path):
        return

    if os.path.isdir(path):
        shutil.rmtree(path)
    elif os.path.isfile(path):
        os.remove(path)


def flv_dir_to_mp4(in_path_dir, out_path_file):
    """
    将flv文件转换为mp4
    法一:(只显示第一段，有问题)
    ffmpeg -safe 0 -f concat -i filelist.txt -c copy out.mp4
    法二:
    ffmpeg -i input1.mp4 -c copy -bsf:v h264_mp4toannexb -f mpegts intermediate1.ts
    ffmpeg -i "concat:1.ts|2.ts|...n.ts" -c copy -absf aac_adtstoasc out.mp4
    """

    tsfile_dir = os.path.join(in_path_dir, 'tsfile')
    del_path(out_path_file)
    del_path(tsfile_dir)

    os.mkdir(tsfile_dir)

    out_files = []

    # 生成中间文件ts
    for root, dirs, files in os.walk(in_path_dir):
        files = natsorted(files)
        for file in files:
            if os.path.splitext(file)[1] == '.flv':
                file_path = os.path.join(root, file)
                video_convert(os.path.join(root, file), os.path.join(
                    tsfile_dir, os.path.splitext(file)[0] + '.ts'))

    # 获取列表参数
    for root, dirs, files in os.walk(tsfile_dir):
        files = natsorted(files)
        for file in files:
            if os.path.splitext(file)[1] == '.ts':
                file_path = os.path.join(root, file)
                out_files.append(file_path)

    ff = FFmpeg(inputs={'concat:' + '|'.join(out_files): None},
                outputs={out_path_file: '-loglevel quiet -c copy -absf aac_adtstoasc -movflags faststart'})
    ff.run()
    # cmd = '|'.join(out_files)
    # print(out_files)
    # cmd = 'concat:' + "\"" + cmd + "\""
    # cmd = 'ffmpeg -i ' + cmd + ' -loglevel quiet -c copy -absf aac_adtstoasc -movflags faststart ' + out_path_file
    # cmd = 'ffmpeg -i ' + cmd + ' -safe 0 -segment_time_metadata 1 -vf select=concatdec_select -af aselect=concatdec_select,aresample=async=1 ' + out_path_file
    # print(cmd)


if __name__ == '__main__':
    start = perf_counter()

    print('sys argv', sys.argv)
    if len(sys.argv) == 3:
        flv_dir_to_mp4(sys.argv[1], sys.argv[2])
    elif len(sys.argv) == 2:
        flv_dir_to_mp4(sys.argv[1], os.path.join(sys.argv[1], 'out.mp4'))
    else:
        root_dir = os.path.dirname(os.path.abspath(__file__))
        flv_dir_to_mp4(root_dir, os.path.join(root_dir, 'out.mp4'))

    end = perf_counter()
    print('Running time: %s Seconds' % (end-start))
```

## 运行结果
![flvs2mp4-python-2020-3-20-21-59-36.png](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/flvs2mp4-python-2020-3-20-21-59-36.png)

11个30s左右的flv合成mp4需要时间7.74秒，多了的话可能会比较慢...