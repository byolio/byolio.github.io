---
layout:     post
title:      "OpenManus的部署与使用"
subtitle:   "让我们部署并使用OpenManus"
date:       2025-3-9
author:     "Byolio"
header-img: "img/deployOpenManus.jpg"
catalog: true
tags:
    - AI
    - OpenManus
---
> 本章主要介绍OpenManus的部署与使用。(请在外网下部署与使用OpenManus, 否则会出现无法安装依赖和无法使用google search的问题)
> OpenManus Github地址 : [https://github.com/mannaandpoem/OpenManus](https://github.com/mannaandpoem/OpenManus)

## 引言
随着上周全球首款通用智能体AI:[Manus](https://manus.im/)的爆火出圈, 其邀请码一码难求, 因此有五位大佬使用了3个小时复刻了一个开源的通用智能体AI, 并将其命名为OpenManus, 希望能够帮助更多的人使用上通用智能体。因此我将在本章节中介绍OpenManus的部署与使用。

## 部署OpenManus
### 安装conda依赖
OpenManus的使用需要安装conda依赖, 因此我们需要先安装conda环境, 我在之前的[文章](https://byolio.top/2024/10/03/conda/)提过, 这里就不再赘述。
### 安装OpenManus及其依赖
打开终端, 输入以下命令:
```cmd
conda create -n open_manus python=3.12
conda activate open_manus
```
等待配置环境完成后, 输入以下命令下载OpenManus:
```cmd
git clone https://github.com/mannaandpoem/OpenManus.git
```
下载完成后, 输入以下命令安装OpenManus的conda依赖:
```cmd
cd OpenManus
pip install -r requirements.txt
```
### 安装playwright浏览器
OpenManus的使用需要安装playwright浏览器, 因此我们需要先安装playwright浏览器, 输入以下命令:
```cmd
playwright install
```
等待下载完成后进入下一步。

### 配置OpenManus
打开下载项目文件`OpenManus`中的`config`文件夹, 将`config.example.toml`文件复制一份, 并重命名为config.toml, 然后打开config.toml文件, 将其中的内容修改为以下内容按照项目要求进行配置:
![configOpenManus](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/configOpenManus.png)
注: model, base_url, api_key需要根据实际情况进行配置。

### 启动OpenManus
打开之前终端所在的位置, 输入以下命令:
```cmd
python main.py
```
如果出现以下信息, 则说明OpenManus启动成功。
```cmd
INFO     [browser_use] BrowserUse logging setup complete with level info
INFO     [root] Anonymized telemetry enabled. See https://docs.browser-use.com/development/telemetry for more information.
Enter your prompt (or 'exit' to quit):
```
在下面输入你想要问的问题, 然后回车, 就可以得到OpenManus的回答了。
注 : OpenManus在使用过程中可能会调用浏览器并且会弹出网页, 这是正常智能体AI在查找信息, 请不要关闭网页, 否则会导致OpenManus跳过正在查找的信息。

### 总结
本章主要介绍了OpenManus的部署与使用, 希望能够帮助更多的人使用上OpenManus。