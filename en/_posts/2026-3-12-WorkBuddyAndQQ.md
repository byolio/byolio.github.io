---
layout: post
title: Deploying OpenClaw Bot Assistant for QQ with WorkBuddy
subtitle: 如何通过WorkBuddy部署QQ的OpenClaw机器人助手
date: 2026-3-12
author: Byolio
header-img: img/OpenClaw.png
catalog: true
tags:
  - OpenClaw
  - AI
lang: en
---

> 🌐 [中文](/2026-3-12-WorkBuddyAndQQ/) | [English](/en/en/_posts/2026-3-12-WorkBuddyAndQQ/)

## Introduction
After QQ officially announced its direct integration with the OpenClaw QQ bot, it launched the WorkBuddy platform software to help users quickly deploy a private QQ OpenClaw bot assistant. The implementation steps are as follows:

## Deployment Steps
### 1. Download WorkBuddy
Official website: [https://copilot.tencent.com/work/](https://copilot.tencent.com/work/), click on "Windows x64" in the image below to download.

![WorkBuddy](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/WorkBuddy.png)
Follow the installation wizard by clicking "Next" all the way through.

### 2. Register a QQ Bot
The OpenClaw QQ bot assistant is the latest QQ bot assistant supported by Tencent, so you must add the OpenClaw QQ bot assistant on the following platform interface. Currently, directly creating a bot on the QQ Open Platform does not support WorkBuddy integration and will result in an error. The URL is as follows:
[https://q.qq.com/qqbot/openclaw/login.html](https://q.qq.com/qqbot/openclaw/login.html)
Scan the QR code to log in, and you will be automatically redirected to create an admin account (if you haven't used this platform before). In this step, select the current logged-in account as the administrator, enter your ID number and name, etc., and then create a bot on this page, as shown:
![QQ Platform](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/QQpingtai.png)

### 3. Obtain Bot Information
Retrieve the AppID and AppSecret of the created bot. Remember these two pieces of information, as they will be used in WorkBuddy.

### 4. Configure WorkBuddy
Open Claw settings:
![WorkBuddy Configuration](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/PixPin_2026-03-12_20-42-45.png)
Click on the QQ bot integration configuration, select "Use URL callback", enter the previously obtained AppID and AppSecret, and click "Register":
![Register WorkBuddy](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/PixPin_2026-03-12_20-54-07.png)
Below, it will display "Registration successful", and then a callback URL (a string of web address) will appear. Click to copy this URL.

### 5. Configure QQ Open Platform
On the QQ Open Platform, fill in the callback request URL with the copied callback URL. For the events below, it is recommended to check all chat-related ones (after configuration, be sure to click "Confirm Configuration", otherwise the configuration will not take effect), as shown:
![Configure Callback](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/PixPin_2026-03-12_20-48-14.png)

### 6. The QQ Claw Bot Assistant is Ready to Use
After that, you can use it on QQ. Send a message on QQ to test if it works. If successful, the Claw bot will reply.

