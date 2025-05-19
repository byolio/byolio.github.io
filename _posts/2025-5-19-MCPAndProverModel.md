---
layout:     post
title:      "浅谈 -- 什么是MCP和ProverModel"
subtitle:   "目前大火的MCP和ProverModel是什么？"
date:       2025-5-19
author:     "Byolio"
header-img: "img/MCPAndProverModel.jpg"
catalog: true
tags:
    - AI
    - 浅谈
---

## 引言
早就想写MCP了，但是一直因为内容过少没写，最近看了一下deepseek-prover-v2的相关资料，感觉很有意思，就合在一起写了这篇文章。

## MCP
MCP(模型上下文协议，Model Context Protocol) 是由人工智能公司Anthropic于`2024 年 11 月 25 日`正式提出并开源发布的 。该协议旨在为大型语言模型（如 Claude3.7 sonnect）提供一个标准化接口，使其能够连接外部数据源和工具，并与其交互。 其因为可以调用各种各样的外部工具而被认为是推动`AI Agent`革命的关键节点, 帮助目前大火的`AI Agent`模型实现了如下功能:
1. 动态访问文件、API、数据库
2. 读取和调用工具（tools）或插件
3. 与操作系统或浏览器交互

## MCP对目前AI界的影响
### 1. manus为代表的全流程自动化AI Agent
ManusAI是由中国初创公司Monica于`2025 年 3 月 6 日`推出的通用型自主 AI Agent，旨在实现从用户意图到实际执行的全流程自动化, 用户只需要提供问题和AI所需要的资料，其会在后台自动创建虚拟机和调用各种工具，完成任务。

![manus](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/manus%E4%BD%BF%E7%94%A8.png)
### 2. coze为代表的工作流AI Agent
Coze是字节跳动推出的轻量化AI Agent构建平台。它通过“模块拖拽+对话驱动”的方式，将任务拆解为工作流（Workflow）节点，让用户无需编程就能创建能自动执行复杂任务的AI Agent。
![coze](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/coze.png)
### 3. 以Cline为代表的IDE AI Agent
Cline是`Moonshot AI`推出的新一代开发辅助AI Agent，嵌入式集成在主流IDE 中，支持自然语言指令驱动的代码重构、跨文件调试、单元测试生成等高阶功能。Cline的底层构建亦采用MCP协议，能够在需要时调用外部智能体协助完成复杂任务，如调用浏览器, 调用分析性能工具等，实现开发流程中多智能体的并行协作。
![cline](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/cline.png)
### 4. 以OpenManus为代表的开源全流程AI Agent
OpenManus是一个基于MCP协议的开源项目，由社区开发者推动构建，其目标是提供一个无需依赖闭源服务即可运行的全流程AI Agent系统。与ManusAI相似，OpenManus 同样支持用户从自然语言输入出发，由Agent自动规划任务、创建执行环境、调用工具链并完成目标。区别在于，OpenManus完全开源，用户可自由定制其智能体行为、插件接口和工具调用逻辑，极大地推动了MCP协议的社区化应用与生态建设。
![open manus](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/openManus.png)

## ProverModel
DeepSeek于2025年4月30日在AI开源社区HuggingFace上正式发布了其新一代数学定理证明模型--DeepSeek-Prover-V2，该模型是全球首个形式化数学大模型, 其671B模型拥有目前最理性的数学推理能力, 其与其他模型的推理能力对比图如下:
![Deepseek-prover-V2](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/Deepseek-prover-V2.png)

## ProverModel对目前AI界的影响
不同于目前的混合推理模型, 数学定理证明模型生成的数据可以作为极佳的理性训练数据，用于训练其他模型，加速非理性的模型走向理性, 以实现更快速的走向AGI。

## 总结
MCP和ProverModel是目前AI界的重要创新, 其对AI的发展具有重要意义。
