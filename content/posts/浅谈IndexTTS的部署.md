+++
date = '2025-09-16T20:23:33+08:00'
draft = false
title = '浅谈IndexTTS的部署'
categories = ['经验分享']
tags = ['IndexTTS', 'modelscope']
slug = 'Index-TTS-deploy'
+++

项目地址：
https://github.com/index-tts/index-tts

## 什么是IndexTTS2?

IndexTTS2是一个文字转语音模型，类似于我们树脂的RVC，但目前看合成的样本好于RVC，它可以紧提供一个三秒左右的音频样本即可生成已假论真的声音效果，你也可以自定义它所需要的语速、语音、情绪、停顿等，下面我们来看看官方的介绍：

 - 提出自回归TTS模型的时长自适应方案。IndexTTS2是首个将精确时长控制与自然时长生成结合的自回归零样本TTS模型，方法可扩展至任意自回归大模型。
 - 情感与说话人特征从提示中解耦，设计特征融合策略，在高情感表达下保持语义流畅与发音清晰，并开发了基于自然语言描述的情感控制工具。
 - 针对高表达性语音数据缺乏，提出高效训练策略，显著提升零样本TTS情感表达至SOTA水平。
 - 代码与预训练权重将公开，促进后续研究与应用。

## 如何部署？

啰嗦的一堆如何部署呢？有人说了，不是有一件整合包吗，为什么还要这么麻烦的部署？

确实，网上已经有很方便的整合包，但如果你爱折腾，喜欢玩，不妨也跟着我尝试一下自己部署它，如果你自己把它亲自动手让它跑起来，是不是有一些成就感？

### 环境要求

请确保已安装 [git](https://git-scm.com/downloads) 和 [git-lfs](https://git-lfs.com/)。

如果你在windows上已经装了git，那么你就不用在特意去装git-lfs了。

在仓库中启用Git-LFS：

```bash
git lfs install
```

安装uv和modelscope,uv是一个非常快的Python包和项目管理器，官方号称比pip快115倍。modelscope是从(https://modelscope.cn)上下载所需模型用的。

```bash
pip install uv modelscope
```

### 下载代码：

```bash
git clone https://github.com/index-tts/index-tts.git && cd index-tts
```

### 安装所需依赖项

```bash
uv sync --all-extras
```

如果下载速度缓慢可以使用国内的镜像：

- 阿里云镜像：

```bash
uv sync --all-extras --default-index "https://mirrors.aliyun.com/pypi/simple"
```
- 清华大学镜像：

```bash
uv sync --all-extras --default-index "https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple"
```

如果安装DeepSpeed报错可以去除`--all-extras`。及变成了

```bash
uv sync
```

如使用镜像，复制上面带镜像的地址自行去除`--all-extras`。

如果遇到cuda报错，要注意，请确保已安装NVIDIA [CUDA Toolkit](https://developer.nvidia.com/cuda-toolkit) 12.8及以上。

### 下载模型：

```bash
modelscope download --model IndexTeam/IndexTTS-2 --local_dir checkpoints
```

耐心等待，慢慢下载。

至此，部署完成！

## 运行web界面

```bash
uv run webui.py
```

首次运行可能会很慢，仍需要下载一些东西，耐心等待。

当然以后开起来也不会很快，我电脑不是很新，需要开一分钟,当命令行看到

* Running on local URL:  http://0.0.0.0:7860

* To create a public link, set `share=True` in `launch()`.

的时候，就表示已经运行起来了，你可以在浏览器访问(http://127.0.0.1:7860)打开web界面。

可通过命令行参数开启FP16推理（降低显存占用）、DeepSpeed加速、CUDA内核编译加速等。可运行以下命令查看所有选项：

```bash
uv run webui.py -h
```

## 声明

本文章为个人整理，方便以后自行部署，部分文字参考官方说明，如有侵权可联系删除。