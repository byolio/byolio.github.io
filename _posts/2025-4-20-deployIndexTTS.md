---
layout:     post
title:      "如何部署IndexTTS"
subtitle:   "让我们部署IndexTTS人声生成模型"
date:       2025-4-20
author:     "Byolio"
header-img: "img/IndexTTSModel.png"
catalog: true
tags:
    - AI
    - IndexTTS
---
> 本文旨在介绍如何在windows平台上部署IndexTTS模型。

## IndexTTS是什么
IndexTTS是阿里最新开源的人声生成模型, 可以生成通过输入不同的声音库和文字合成不同的人声。

## IndexTTS的部署
打开终端clone代码仓库:
```bash
git clone https://github.com/index-tts/index-tts.git
```
进入IndexTTS项目仓库再打开终端输入如下指令:
```bash
conda create -n index-tts python=3.10
conda activate index-tts
pip install -r requirements.txt
```
因为在windows平台下部署使用`pip install -r requirements.txt`会出现以下问题:
```bash
Successfully built jieba encodec antlr4-python3-runtime distance
Failed to build pynini
ERROR: Failed to build installable wheels for some pyproject.toml based projects (pynini)
```
解决方案为(注意需要先执行`pip install -r requirements.txt`安装其他依赖包):
```bash
conda install -c conda-forge pynini==2.1.5
pip install WeTextProcessing==1.0.3
pip install -e ".[webui]"
```
下载model(此时需要科学上网且需要在项目仓库的根目录下打开终端输入如下指令):
```bash
wget https://huggingface.co/IndexTeam/Index-TTS/resolve/main/bigvgan_discriminator.pth -P checkpoints
wget https://huggingface.co/IndexTeam/Index-TTS/resolve/main/bigvgan_generator.pth -P checkpoints
wget https://huggingface.co/IndexTeam/Index-TTS/resolve/main/bpe.model -P checkpoints
wget https://huggingface.co/IndexTeam/Index-TTS/resolve/main/dvae.pth -P checkpoints
wget https://huggingface.co/IndexTeam/Index-TTS/resolve/main/gpt.pth -P checkpoints
wget https://huggingface.co/IndexTeam/Index-TTS/resolve/main/unigram_12000.vocab -P checkpoints
```

## IndexTTS的使用
在项目仓库的根目录下打开终端输入如下指令:
```bash
python webui.py
```
其会默认在运行时返回一个链接, 如下图所示:
```bash
>> TextNormalizer loaded
* Running on local URL:  http://127.0.0.1:7860
```
将其中的`http://127.0.0.1:7860`复制粘贴到浏览器, 即可访问IndexTTS的webui界面。
然后输入文字, 选择声音库, 点击生成, 即可生成对应的人声。
