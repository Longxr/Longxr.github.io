---
title: Python 安装 onnxruntime-gpu、paddlepaddle-gpu 报错处理
categories:
  - Python
tags:
  - Python
  - onnxruntime
  - paddlepaddle
date: 2024-06-12 22:30:14
---

<Excerpt in index | 摘要>
最近看到一个挂机脚本，依赖里面用到了 onnxruntime-gpu、paddlepaddle-gpu，本机环境搭建时碰到的坑做个记录。 <!-- more -->
<The rest of contents | 余下全文>
[刷声骸脚本](https://github.com/lazydog28/mc_auto_boss)，成熟的电脑要学会自己玩游戏了。

## CUDA 安装

注意看要用到 CUDA 的库支持的版本，比如 [onnxruntime 安装依赖](https://onnxruntime.ai/docs/execution-providers/CUDA-ExecutionProvider.html#requirements)

### 下载链接

[CUDA 历史版本下载](https://developer.nvidia.com/cuda-toolkit-archive)
[cuDNN 历史版本下载](https://developer.nvidia.com/rdp/cudnn-archive)

### CUDA 配置查看

nvidia-smi //查看显卡支持的 CUDA 版本
nvcc --version //查看当前安装的 CUDA 版本

## 问题处理

### onnxruntime-gpu 报错处理

#### CUDA_PATH is set but CUDA wasnt able to be loaded

onnxruntime::python::CreateExecutionProviderInstance CUDA_PATH is set but CUDA wasnt able to be loaded. Please install the correct version of CUDA andcuDNN as mentioned in the GPU requirements page (https://onnxruntime.ai/docs/execution-providers/CUDA-ExecutionProvider.html#requirements), make sure they're in the PATH, and that your GPU is supported

CUDA、cuDNN、onnxruntime-gpu 版本不匹配，比如我安装时官网最新 CUDA 已经 12.5 了，但是 onnxruntime-gpu 最新版本 1.18 只支持到 CUDA 12.4, cuDNN 8.9.2.26。于是又重新去装 CUDA...cuDNN 下载解压后复制到 CUDA 所在版本目录即可。

另外默认 onnxruntime-gpu 是用的 CUDA 11.8，如果要用 CUDA 12.4，需要卸载默认装的再重新安装，可以参考[官方安装文档](https://onnxruntime.ai/docs/install/#install-onnx-runtime-gpu-cuda-12x)

```shell
pip uninstall onnxruntime-gpu
pip install onnxruntime-gpu --extra-index-url https://aiinfra.pkgs.visualstudio.com/PublicPackages/_packaging/onnxruntime-cuda-12/pypi/simple/
```

#### Could not locate zlibwapi.dll. Please make sure it is in your library path!

cuDNN 8.9.x 版本需要装下 zlib，由于 CUDA bin 目录已经加到环境变量，直接把 zlib dll 放到 CUDA bin 目录下就行
[cuDNN on Windows](https://docs.nvidia.com/deeplearning/cudnn/archives/cudnn-892/install-guide/index.html#install-zlib-windows)

### paddlepaddle-gpu

#### The third-party dynamic library (cublas64_118.dll;cublas64_11.dll) that Paddle depends on is not configured correctly. (error code is 126)

默认 paddlepaddle-gpu 和 CUDA 版本不匹配...重装。[paddlepaddle 安装](https://www.paddlepaddle.org.cn/install/quick?docurl=/documentation/docs/zh/install/pip/windows-pip.html)

```shell
pip uninstall paddlepaddle-gpu
pip install paddlepaddle-gpu==2.6.1.post120 -f https://www.paddlepaddle.org.cn/whl/windows/mkl/avx/stable.html
```

装好后可以检测下

```python
import paddle
paddle.utils.run_check()
```
