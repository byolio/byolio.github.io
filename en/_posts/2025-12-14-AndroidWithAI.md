---
layout: post
title: Combining Android with AI — Deploy AutoGLM to Turn Your Phone into an AI Phone
subtitle: 如何部署AutoGLM将手机变成AI手机
date: 2025-12-14T00:00:00.000Z
author: Byolio
header-img: img/androidwithAI.jpg
catalog: true
tags:
  - AI
  - android
lang: en
---

> 🌐 [中文](https://www.byolio.blog/2025-12-14-AndroidWithAI/) | [English](https://www.byolio.blog/en/en/_posts/2025-12-14-AndroidWithAI/)

> ADBKeyBoard GitHub URL: [https://github.com/senzhk/ADBKeyBoard](https://github.com/senzhk/ADBKeyBoard) \
> AutoGLM GitHub URL: [https://github.com/zai-org/Open-AutoGLM](https://github.com/zai-org/Open-AutoGLM)

## Introduction
Recently, the Doubao AI assistant phone has taken the internet by storm. Its excellent features and performance have garnered widespread attention, with many calling it the future of AI phones. I recently discovered the AutoGLM project, which can turn a phone into a device similar to the Doubao AI assistant phone. Therefore, I will share how I deployed it and my evaluation of it.

## What is AutoGLM
AutoGLM is an open-source automatic dialogue assistant based on the GLM model by Zhipu AI. It can run on a phone and provide functionality similar to the Doubao AI assistant. The GitHub URL is: [https://github.com/zai-org/Open-AutoGLM](https://github.com/zai-org/Open-AutoGLM)

## Deployment Steps

### Environment Preparation
1. The computer needs Python 3.10 or higher. Python [Windows download](https://www.python.org/downloads/windows/)
2. Install ADB and add it to the environment variables. [ADB download](https://dl.google.com/android/repository/platform-tools-latest-windows.zip?hl=zh-cn). If you are an Android developer and have the SDK on your computer, the SDK includes ADB by default. Simply add the SDK to the environment variables. You can verify ADB installation by running the following command in cmd:
`adb --version`
3. Prepare an Android phone, enable developer mode (tap the build number repeatedly), turn on USB debugging, and connect the phone to the computer via a USB cable. You can check if the connection is successful by running the following command in cmd:
`adb devices`
If a device is listed, USB debugging is enabled successfully.
4. If the ADB Keyboard app is not installed on the Android phone when running AutoGLM, you will get the following error at startup:
`Checking ADB Keyboard... ❌ FAILED`
Therefore, you need to install the ADB Keyboard app, which converts ADB commands into text. [Download the APK](https://github.com/senzhk/ADBKeyBoard/blob/master/ADBKeyboard.apk) and install it on the phone. After installation, restart the phone (the restart may be slow, so be patient) and enable the ADB Keyboard app as the input method in Settings -> Language & Input. Then re-enable developer mode and USB debugging on the phone, and connect it to the computer via USB.

### Installing AutoGLM
1. Clone the AutoGLM project using git:
    ```cmd
    git clone https://github.com/senzhk/ADBKeyBoard.git
    ```
2. Open cmd in the AutoGLM root directory and install the project dependencies with the following commands:
    ```cmd
    pip install -r requirements.txt 
    pip install -e .
    ```

### Configuring the ModelScope API Service
Here, we use the free service provided by Alibaba's ModelScope as an example. [Use the model AutoGLM-Phone-9B](https://modelscope.cn/models/ZhipuAI/AutoGLM-Phone-9B/summary). Register an account, go to Personal Homepage -> Access Token, generate an access token to obtain ModelScope's free API-Inference services, and copy the access token.

### Using AutoGLM
Now you can use AutoGLM with the following command:
```cmd
python main.py --base-url https://api-inference.modelscope.cn/v1 --model "ZhipuAI/AutoGLM-Phone-9B" --apikey "your-modelscope-api-key" "###"
```
Replace `your-modelscope-api-key` with the access token you just copied, and replace `"###"` with the instruction you want AutoGLM to execute, for example: "Open Meituan and search for nearby hotpot restaurants". Then press Enter to run. If everything goes smoothly, you will see output like the following:
```txt
==================================================
💭 Thought Process:
--------------------------------------------------
The user wants to open Meituan and search for nearby hotpot restaurants. I need to:
1. First launch the Meituan app
2. Then search for nearby hotpot restaurants in Meituan

Currently on the system desktop, I can see some app icons. I need to use the Launch function to open the Meituan app.

According to the list of allowed apps, Meituan is one of the available apps.


==================================================
⏱️  Performance Metrics:
--------------------------------------------------
Time to First Token (TTFT): 10.027s
Time to Complete Thinking:  10.955s
Total Inference Time:       11.085s
==================================================
--------------------------------------------------
🎯 Executed Action:
{
  "_metadata": "do",
  "action": "Launch",
  "app": "Meituan"
}
==================================================
```
At this point, the phone will automatically open the Meituan app and search for nearby hotpot restaurants. This successfully turns your phone into an AI phone assistant.

## Summary
The AutoGLM project can turn a phone into a device similar to the Doubao AI assistant phone. Deployment is relatively simple; just follow the steps above. However, it is important to note that the AutoGLM project is still under development, and its functionality and performance are yet to be improved. It often misunderstands complex tasks or fails to execute them, resulting in various errors. Nevertheless, its potential is enormous, offering more convenience and fun for phone users.
