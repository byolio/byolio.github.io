---
layout: post
title: How to Deploy IndexTTS
subtitle: 让我们部署IndexTTS人声生成模型
date: 2025-4-20
author: Byolio
header-img: img/IndexTTSModel.png
catalog: true
tags:
  - AI
  - IndexTTS
lang: en
---

> 🌐 [中文](/2025-4-20-deployIndexTTS/) | [English](/en/en/_posts/2025-4-20-deployIndexTTS/)

> This article aims to introduce how to deploy the IndexTTS model on the Windows platform.

## What is IndexTTS
IndexTTS is the latest open-source voice generation model from Alibaba. It can synthesize different voices by inputting different voice libraries and text.

## Deploying IndexTTS
Open a terminal and clone the code repository:
```bash
git clone https://github.com/index-tts/index-tts.git
```
Enter the IndexTTS project directory, open a terminal again, and run the following commands:
```bash
conda create -n index-tts python=3.10
conda activate index-tts
pip install -r requirements.txt
```
When deploying on the Windows platform, using `pip install -r requirements.txt` may cause the following issues:
```bash
Successfully built jieba encodec antlr4-python3-runtime distance
Failed to build pynini
ERROR: Failed to build installable wheels for some pyproject.toml based projects (pynini)
```
The solution is (note that you need to run `pip install -r requirements.txt` first to install other dependencies):
```bash
conda install -c conda-forge pynini==2.1.5
pip install WeTextProcessing==1.0.3
pip install -e ".[webui]"
```
Download the models (you need a VPN/proxy and run the following commands in the terminal at the root of the project directory):
```bash
wget https://huggingface.co/IndexTeam/Index-TTS/resolve/main/bigvgan_discriminator.pth -P checkpoints
wget https://huggingface.co/IndexTeam/Index-TTS/resolve/main/bigvgan_generator.pth -P checkpoints
wget https://huggingface.co/IndexTeam/Index-TTS/resolve/main/bpe.model -P checkpoints
wget https://huggingface.co/IndexTeam/Index-TTS/resolve/main/dvae.pth -P checkpoints
wget https://huggingface.co/IndexTeam/Index-TTS/resolve/main/gpt.pth -P checkpoints
wget https://huggingface.co/IndexTeam/Index-TTS/resolve/main/unigram_12000.vocab -P checkpoints
```

## Using IndexTTS
Open a terminal at the root of the project directory and run the following command:
```bash
python webui.py
```
It will return a link by default when running, as shown below:
```bash
>> TextNormalizer loaded
* Running on local URL:  http://127.0.0.1:7860
```
Copy and paste `http://127.0.0.1:7860` into your browser to access the IndexTTS web UI.
Then enter text, select a voice library, and click generate to produce the corresponding voice.
