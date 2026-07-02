---
layout:     post
title:      "在电脑中部署Deepseek-R1"
subtitle:   "如何在笔记本电脑中部署Deepseek-R1"
date:       2025-1-28
author:     "Byolio"
header-img: "img/post-deepseekR1.jpg"
catalog: true
tags:
    - AI
    - linux
    - docker
---
> 本文旨在介绍两种在电脑使用docker部署Deepseek-R1的方法。(本文涉及docker官方仓库和chrome插件商店的访问, 请确保网络通畅)

## 什么是Deepseek-R1
Deepseek-R1是由中国人工智能公司深度求索(DeepSeek)研发的开源LLM, 官方公布其基准测试如下:
![deepseekR1](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/DeepSeekR1image.png)
可以看到其在多个领域都有对标ChatGPT-o1的能力, 在数学、代码、自然语言推理等任务上，性能比肩OpenAI o1正式版, 因此我们可以将其部署在电脑docker上做一个私人的LLM助手。
(官方论文链接: [https://github.com/deepseek-ai/DeepSeek-R1/blob/main/DeepSeek_R1.pdf](https://github.com/deepseek-ai/DeepSeek-R1/blob/main/DeepSeek_R1.pdf))

## 安装需求
### 硬件需求 
目前笔记本的显卡主要是nvidia的RTX4060, RTX4070, 50系尚未大量用于笔电, 显存大小主要为8GB, 可以安装deepseek-R1:7b或8b版本, 低于此建议安装deepseek-R1:1.5b版本
### 软件需求
我这边安装的是`nvidia GPU`的版本, 因此需要有`nvidia GPU`显卡且安装`nvidia GPU`驱动, 且安装好`docker desktop`和`WSL`, 如果不会安装请查看我之前写的[文章](https://byolio.top/2024/11/06/llama3.2/)


## 如何安装deepseek-R1
我这边使用ollama进行安装, 并提供以下两种方法 :
### web-ui
我之前写的[文章](https://byolio.top/2024/11/06/llama3.2/)中写了如何安装, 这边就不过多说明了, 不过如果安装的是最新版本的`ollama web-ui`需要使用新的方法下载模型: 
![webuideepseek](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/webuideepseek.png)
点击上面的从Ollama.com拉去自动完成下载即可
### page assist
这是一个chrome插件, 可以在网页中显示LLM图标
1. 在cmd运行这行指令(确保英伟达驱动存在的情况下):
`docker run -d --gpus=all -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama`
在docker中下载ollama镜像并将其部署
2. 在chrome插件商店下载`Page Assist`插件, 并将其Ollama URL设置为: `http://127.0.0.1:11434`
3. 使用`docker exec -it ollama ollama run deepseek-r1:8b`下载 `deepseek-r1:8b`的LLM
4. 回到`Page Assist`插件, 选择`deepseek-r1:8b`模型, 即可使用

## 总结
以上就是在笔记本电脑中部署Deepseek-R1的方法, 希望对大家有所帮助。