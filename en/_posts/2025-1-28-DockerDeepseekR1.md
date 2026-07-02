---
layout: post
title: Deploying Deepseek-R1 on a Computer
subtitle: 如何在笔记本电脑中部署Deepseek-R1
date: 2025-1-28
author: Byolio
header-img: img/post-deepseekR1.jpg
catalog: true
tags:
  - AI
  - linux
  - docker
lang: en
---

> 🌐 [中文](https://www.byolio.blog/2025-1-28-DockerDeepseekR1/) | [English](https://www.byolio.blog/en/en/_posts/2025-1-28-DockerDeepseekR1/)

> This article aims to introduce two methods for deploying Deepseek-R1 on a computer using Docker. (This article involves accessing the official Docker repository and the Chrome Web Store; please ensure your network is unobstructed.)

## What is Deepseek-R1
Deepseek-R1 is an open-source LLM developed by the Chinese AI company DeepSeek. Its official benchmark results are as follows:
![deepseekR1](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/DeepSeekR1image.png)
It can be seen that it has capabilities comparable to ChatGPT-o1 in multiple fields. In tasks such as mathematics, code, and natural language reasoning, its performance is on par with the official version of OpenAI o1. Therefore, we can deploy it on a computer using Docker as a personal LLM assistant.
(Official paper link: [https://github.com/deepseek-ai/DeepSeek-R1/blob/main/DeepSeek_R1.pdf](https://github.com/deepseek-ai/DeepSeek-R1/blob/main/DeepSeek_R1.pdf))

## Installation Requirements
### Hardware Requirements
Currently, laptop GPUs are mainly NVIDIA RTX 4060 and RTX 4070. The 50 series has not yet been widely adopted in laptops. The VRAM size is primarily 8GB, which can accommodate the deepseek-R1:7b or 8b versions. For lower VRAM, it is recommended to install the deepseek-R1:1.5b version.
### Software Requirements
I am installing the `nvidia GPU` version, so you need an `nvidia GPU` graphics card with the `nvidia GPU` driver installed, as well as `docker desktop` and `WSL`. If you are unsure how to install these, please refer to my previous [article](https://byolio.top/2024/11/06/llama3.2/).


## How to Install deepseek-R1
I am using ollama for installation and provide the following two methods:
### web-ui
My previous [article](https://byolio.top/2024/11/06/llama3.2/) explained how to install it, so I won't go into detail here. However, if you are installing the latest version of `ollama web-ui`, you need to use a new method to download the model:
![webuideepseek](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/webuideepseek.png)
Click the button above to pull from Ollama.com and complete the download automatically.
### page assist
This is a Chrome extension that displays an LLM icon on web pages.
1. Run the following command in cmd (ensure the NVIDIA driver is present):
`docker run -d --gpus=all -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama`
This downloads the ollama image in Docker and deploys it.
2. Download the `Page Assist` extension from the Chrome Web Store and set its Ollama URL to: `http://127.0.0.1:11434`
3. Use `docker exec -it ollama ollama run deepseek-r1:8b` to download the `deepseek-r1:8b` LLM.
4. Return to the `Page Assist` extension, select the `deepseek-r1:8b` model, and you can start using it.

## Summary
The above are the methods for deploying Deepseek-R1 on a laptop. I hope this helps everyone.
