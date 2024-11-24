---
layout:     post
title:      "浅谈-主流笔电本地端布置开源大模型测试"
subtitle:   "测一测当前主流开源LLM在笔电端的实力"
date:       2024-11-24
author:     "Byolio"
header-img: "img/AI-PC.jpg"
catalog: true
tags:
    - 浅谈
    - AI
---
> 本次测试主要使用的是笔电常见的本地部署模型测试。

## 引子
在朋友圈里看到这样一条消息:
![first](https://cdn.jsdelivr.net/gh/byolio/newtc@main/img/first.jpg)
因此我突发奇想，想测测当前主流开源LLM在笔电端资源受限情况下的能力, 就用朋友圈里的题目:0-1000的数字里，只有一个5的数字个数有多少。
## 测试结果
1. ollama3.2:3b
![ollama3.2:3b](https://cdn.jsdelivr.net/gh/byolio/newtc@main/img/ollama3b.png)
2. qwen2.5:7b
![qwen2.5:7b](https://cdn.jsdelivr.net/gh/byolio/newtc@main/img/qwen.png)
3. ollama3.1:8b
![ollama3.1:8b](https://cdn.jsdelivr.net/gh/byolio/newtc@main/img/ollama8b.png)
4. gemma2:9b
![gemma2:9b](https://cdn.jsdelivr.net/gh/byolio/newtc@main/img/gemma.png)
5. phi3.5:8b
![phi3.5:8b](https://cdn.jsdelivr.net/gh/byolio/newtc@main/img/phi3.5.png)
这个模型一言难尽, 答案都给不直接, 最后找到了这样一段话:
![phianswer](https://cdn.jsdelivr.net/gh/byolio/newtc@main/img/phianswer.png)  
看来它给only one的结论, 这也是一种only one。
6. mistral:7b
![mistral:7b](https://cdn.jsdelivr.net/gh/byolio/newtc@main/img/mistral7b.png)

可以看出这些本地模型没一个答对的。
## 总结
目前主流的开源大模型在本地端的大参数模型部署, 都需要非常多的资源, 因此在资源受限的情况下, 模型在数学方面的能力还是比较弱的。因此ai开源模型从业者需要在包括架构, 参数, 算法等方面做更多的探索, 以寻求做出能在笔电端部署的好用的开源大模型, 让ai造福全人类。