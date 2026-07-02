---
layout: post
title: Deployment and Usage of OpenManus
subtitle: 让我们部署并使用OpenManus
date: 2025-3-9
author: Byolio
header-img: img/deployOpenManus.jpg
catalog: true
tags:
  - AI
  - OpenManus
lang: en
---

> 🌐 [中文](https://www.byolio.blog/2025-3-9-deployOpenManus/) | [English](https://www.byolio.blog/en/en/_posts/2025-3-9-deployOpenManus/)

> This chapter mainly introduces the deployment and usage of OpenManus. (Please deploy and use OpenManus under an external network, otherwise you may encounter issues with installing dependencies and using Google Search.)
> OpenManus Github address: [https://github.com/mannaandpoem/OpenManus](https://github.com/mannaandpoem/OpenManus)

## Introduction
With the explosive popularity of the world's first general-purpose AI agent [Manus](https://manus.im/) last week, its invitation codes became extremely hard to obtain. Therefore, five experts recreated an open-source general-purpose AI agent in just three hours, naming it OpenManus, hoping to help more people use general-purpose agents. In this chapter, I will introduce the deployment and usage of OpenManus.

## Deploying OpenManus
### Installing conda dependencies
Using OpenManus requires installing conda dependencies, so we need to set up the conda environment first. I have mentioned this in a previous [article](https://byolio.top/2024/10/03/conda/), so I won't repeat it here.
### Installing OpenManus and its dependencies
Open the terminal and enter the following commands:
```cmd
conda create -n open_manus python=3.12
conda activate open_manus
```
After the environment configuration is complete, enter the following command to download OpenManus:
```cmd
git clone https://github.com/mannaandpoem/OpenManus.git
```
Once the download is complete, enter the following command to install OpenManus's conda dependencies:
```cmd
cd OpenManus
pip install -r requirements.txt
```
### Installing the Playwright browser
Using OpenManus requires installing the Playwright browser, so we need to install it first. Enter the following command:
```cmd
playwright install
```
Wait for the download to complete, then proceed to the next step.

### Configuring OpenManus
Open the `config` folder in the downloaded project file `OpenManus`, copy the `config.example.toml` file, rename it to `config.toml`, then open the `config.toml` file and modify its content according to the project requirements as shown below:
![configOpenManus](https://cdn.jsdelivr.net/gh/byolio/tc3@main/img/configOpenManus.png)
Note: The model, base_url, and api_key need to be configured based on the actual situation.

### Starting OpenManus
Navigate to the terminal location from before and enter the following command:
```cmd
python main.py
```
If the following information appears, it means OpenManus has started successfully:
```cmd
INFO     [browser_use] BrowserUse logging setup complete with level info
INFO     [root] Anonymized telemetry enabled. See https://docs.browser-use.com/development/telemetry for more information.
Enter your prompt (or 'exit' to quit):
```
Enter the question you want to ask below, then press Enter, and you will receive OpenManus's response.

Note: During use, OpenManus may call the browser and pop up web pages. This is normal as the AI agent searches for information. Please do not close the web pages, as this may cause OpenManus to skip the information being searched.

### Summary
This chapter mainly introduced the deployment and usage of OpenManus, hoping to help you get started with OpenManus.
