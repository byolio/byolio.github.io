---
layout:     post
title:      "Android与AI结合——部署AutoGLM将手机变成AI手机"
subtitle:   "如何部署AutoGLM将手机变成AI手机"
date:       2025-12-14
author:     "Byolio"
header-img: "img/androidwithAI.jpg"
catalog: true
tags:
    - AI
    - android
---
> ADBkeyBoard github地址: [https://github.com/senzhk/ADBKeyBoard](https://github.com/senzhk/ADBKeyBoard)
> AutoGLM github地址: [https://github.com/zai-org/Open-AutoGLM](https://github.com/zai-org/Open-AutoGLM)
## 引言
前段时间的豆包AI助手手机火遍互联网, 其优秀的功能和性能受到了广泛的关注, 被许多人称为AI手机的未来, 刚好我最近发现了AutoGLM项目可以将手机变成一个类似豆包AI手机助手手机的手机, 因此说一下我是如何部署的和对它的评价

## 什么是AutoGLM
AutoGLM是智谱开源的一个基于GLM模型的自动对话助手, 它可以在手机上运行, 并提供与豆包AI助手类似的功能, github地址为: [https://github.com/zai-org/Open-AutoGLM](https://github.com/zai-org/Open-AutoGLM)

## 部署步骤

### 环境准备工作
1. 电脑需要有python3.10及以上的环境, python [windows下载](https://www.python.org/downloads/windows/)
2. 安装ADB, 并将其添加到环境变量中, [ADB下载](https://dl.google.com/android/repository/platform-tools-latest-windows.zip?hl=zh-cn), 如果你是android开发者电脑中有SDK, SDK中默认包含ADB, 直接将SDK添加到环境变量中即可, 可以输入cmd指令:
`adb --version`
来确认adb是否安装成功
1. 准备一台安卓手机, 并开启开发者模式(版本号连续点击)打开USB调试模式, 并将手机通过数据线连接到电脑, 可以在cmd输入如下指令检查是否连接成功
`adb devices`
如果有设备输出即为USB调试模式开启成功
1. 如果你在运行AutoGLM时android手机上没有安装ADB KeyBoard应用, 那么在启动时会得到一个 \
`Checking ADB Keyboard... ❌ FAILED` \
因此需要安装一个ADB KeyBoard应用, 用于通过ADB KeyBoard应用将ADB命令转变成文字, [下载apk](https://github.com/senzhk/ADBKeyBoard/blob/master/ADBKeyboard.apk), 并将其安装到手机上, 安装完成后, 重启手机(重启速度稍慢, 需要耐心等待)并在手机的设置 -> 语法和输入法中开启ADB KeyBoard应用作为输入法, 然后再重新启动上面的手机开发者模式和USB调试模式, 并通过数据线连接到电脑

### 安装AutoGLM
1. 通过git克隆AutoGLM项目
    ```cmd
    git clone https://github.com/senzhk/ADBKeyBoard.git
    ```
2. 在AutoGLM根目录下打开cmd, 输入如下指令安装AutoGLM项目依赖
    ```cmd
    pip install -r requirements.txt 
    pip install -e .
    ```

### 配置魔塔模型API服务
这里以免费的阿里ModelScope提供的服务为例, [使用模型AutoGLM-Phone-9B](https://modelscope.cn/models/ZhipuAI/AutoGLM-Phone-9B/summary), 注册一个账号, 点开个人主页->访问令牌, 生成一个访问令牌以获取魔塔的免费API-Inference等服务, 将访问令牌复制下来

### 使用AutoGLM
接下来就可以使用AutoGLM了, 指令为:
```cmd
python main.py --base-url https://api-inference.modelscope.cn/v1 --model "ZhipuAI/AutoGLM-Phone-9B" --apikey "your-modelscope-api-key" "###"
```
将上面的`your-modelscope-api-key`替换成你刚才复制的访问令牌, `"###"`替换成你要让AutoGLM执行的指令, 例如: "打开美团搜索附近的火锅店", 然后回车运行, 如果一切顺利, 你会看到如下输出:
```txt
==================================================
💭 思考过程:
--------------------------------------------------
用户想要打开美团并搜索附近的火锅店。我需要：
1. 首先启动美团应用
2. 然后在美团中搜索附近的火锅店

当前在系统桌面，我可以看到一些应用图标。我需要使用Launch功能来打开美团应用。

根据允许的应用列表，美团是可用的应用之一。


==================================================
⏱️  性能指标:
--------------------------------------------------
首 Token 延迟 (TTFT): 10.027s
思考完成延迟:        10.955s
总推理时间:          11.085s
==================================================
--------------------------------------------------
🎯 执行动作:
{
  "_metadata": "do",
  "action": "Launch",
  "app": "美团"
}
==================================================
```
此时手机上会自动打开美团应用, 并搜索附近的火锅店, 这样就成功将手机变成了一个AI手机助手

## 总结
AutoGLM项目可以将手机变成一个类似豆包AI手机助手手机的手机, 部署起来也比较简单, 只需要按照上面的步骤操作即可, 但是需要注意的是, AutoGLM项目目前还在开发阶段, 功能和性能都还有待完善, 在复杂任务上经常理解有误或无法执行而报各种各样的错误, 但是它的潜力是巨大的, 可以为手机用户提供更多的便利和乐趣