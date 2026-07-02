---
layout: post
title: >-
  A Brief Discussion: Testing Mainstream Open-Source LLMs Deployed Locally on
  Laptops
subtitle: 测一测当前主流开源LLM在笔电端的实力
date: 2024-11-24T00:00:00.000Z
author: Byolio
header-img: img/AI-PC.jpg
catalog: true
tags:
  - Brief Discussion
  - AI
lang: en
---

> 🌐 [中文](https://www.byolio.blog/2024-11-24-openSourceAI/) | [English](https://www.byolio.blog/en/en/_posts/2024-11-24-openSourceAI/)

> This test primarily uses common locally deployed models on laptops.

## Introduction
I saw a message in my Moments:
![first](https://cdn.jsdelivr.net/gh/byolio/newtc@main/img/first.jpg)
So I had a sudden idea to test the capabilities of current mainstream open-source LLMs under resource-constrained conditions on laptops, using the question from Moments: How many numbers from 0 to 1000 contain exactly one digit '5'?
## Test Results
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
This model is hard to describe; it didn't give a direct answer. Finally, I found this paragraph:
![phianswer](https://cdn.jsdelivr.net/gh/byolio/newtc@main/img/phianswer.png)  
It seems it gave the conclusion 'only one', which is also a kind of 'only one'.
6. mistral:7b
![mistral:7b](https://cdn.jsdelivr.net/gh/byolio/newtc@main/img/mistral7b.png)

It can be seen that none of these local models answered correctly.
## Conclusion
Currently, deploying mainstream open-source large language models with large parameter sizes locally requires a lot of resources. Therefore, under resource-constrained conditions, the models' mathematical capabilities are still relatively weak. Consequently, practitioners in the open-source AI model field need to explore more in areas such as architecture, parameters, and algorithms, in order to create useful open-source large models that can be deployed on laptops, allowing AI to benefit all of humanity.
