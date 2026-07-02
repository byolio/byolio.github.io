---
layout:     post
title:      "WorkBuddy部署QQ的OpenClaw机器人助手"
subtitle:   "如何通过WorkBuddy部署QQ的OpenClaw机器人助手"
date:       2026-3-12
author:     "Byolio"
header-img: "img/OpenClaw.png"
catalog: true
tags:
    - OpenClaw
    - AI
---

## 引言
随着QQ在正式宣布亲自下场接入OpenClaw的QQ机器人后，推出了WorkBuddy平台软件帮助用户更快的部署私人QQ的OpenClaw机器人助手, 实现步骤如下

## 部署步骤
### 1. 下载WorkBuddy
官网地址 : [https://copilot.tencent.com/work/](https://copilot.tencent.com/work/), 点击下图的windows x64进行下载

![WorkBuddy](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/WorkBuddy.png)
安装包一路下一步安装即可

### 2. 注册QQ机器人
OpenClaw QQ机器人助手是腾讯最新支持的QQ机器人助手，因此必须在以下平台界面添加OpenClaw QQ机器人助手, 目前直接进入QQ开放平台创建机器人是不支持WorkBuddy接入的, 会报错, 网址如下 : 
[https://q.qq.com/qqbot/openclaw/login.html](https://q.qq.com/qqbot/openclaw/login.html) 
扫码登录会自动跳转创建管理员账号(没有使用过该平台), 这一步选择当前登录账号为管理员并输入自己身份证号和姓名等信息, 然后在该页面创建机器人, 如图:
![QQ平台](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/QQpingtai.png)

### 3. 获取机器人信息
获取创建机器人的AppID和AppSecret, 记住这两个信息, 在WorkBuddy中会用到

### 4. 配置WorkBuddy
打开Claw设置:
![workbuddy配置](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/PixPin_2026-03-12_20-42-45.png)
点击QQ机器人集成的配置, 选择使用URL回调, 输入之前获取的AppID和AppSecret, 点击注册:
![注册WorkBuddy](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/PixPin_2026-03-12_20-54-07.png) 
下面会显示注册成功, 然后出现回调URL(一串网址), 点击复制该URL

### 5. 配置QQ开放平台
在QQ开放平台中配置回调中请求地址填入该回调URL, 下面的事件建议和聊天相关的全部勾选(配置完后一定要点击确定配置, 否则配置不会生效), 如图:
![配置回调](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/PixPin_2026-03-12_20-48-14.png)

### 6. QQ Claw机器人助手就可以使用了
之后就可以在QQ上使用了, 可以在QQ上发送消息测试一下是否成功, 成功的话就会有Claw机器人回复了

