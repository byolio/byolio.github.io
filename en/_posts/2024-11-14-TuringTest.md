---
layout: post
title: 'A Brief Discussion: Is the Turing Test Really Useful for AI?'
subtitle: 简单聊一聊当今如何测试ai的智能
date: 2024-11-16T00:00:00.000Z
author: Byolio
header-img: img/post-bg-TuringTest.png
catalog: true
tags:
  - Brief Discussion
  - AI
  - Turing Test
lang: en
---

> 🌐 [中文](https://www.byolio.blog/2024-11-14-TuringTest/) | [English](https://www.byolio.blog/en/en/_posts/2024-11-14-TuringTest/)

> This article aims to discuss the currently well-known AI intelligence testing methods.

## Introduction
The father of computer science, `Alan Turing`, published a landmark paper in the field of artificial intelligence in 1950: `Computing Machinery and Intelligence`, in which he raised a revolutionary question: `Can machines think?`, and described a testing method, which is the most well-known `Turing test`. However, with the advent of the AI wave, whether the Turing test is still effective has become a controversial topic.
## Turing Test
The `Turing test` is typically set up as follows: a human judge engages in a conversation with a human and a machine. The judge's task is to determine which is the machine and which is the human. If the judge cannot correctly distinguish, or the probability of misjudgment is very high, then the machine is considered to have passed the `Turing test`. \
The advantage of this test is that it cleverly transforms the complex and difficult-to-judge question of whether machines can possess consciousness, emotions, or thinking ability into judging whether they can successfully imitate humans. Therefore, many people call it an imitation game.

## Limitations of the Turing Test
As a behavioral imitation game, the Turing test inevitably becomes a benchmark for judging and understanding things from a `behaviorism` perspective, but it inevitably conflicts with `cognitivism`. Cognitivists believe that external `behavioral performance` cannot replace the more important internal `cognition`, and that external behavior alone cannot determine whether a machine possesses intelligence. They famously used the `Chinese Room` argument to refute it. \
In addition, factors such as `the number of judges`, `the threshold for judge misjudgment`, `the type of questions`, `the testing time and environment`, and `the background and ability of judges` lack precise definitions, which can affect the results of the Turing test. In particular, `whether the judge regularly uses AI` has a significant impact on the experimental results. \
Moreover, the Turing test focuses on how to simulate humans, in other words, how to deceive humans. Deliberately answering questions incorrectly or saying "I don't know" as a goal is meaningless, because the purpose of artificial intelligence should be to improve productivity and free people from heavy physical labor.

## Chinese Room
Among the experiments opposing the Turing test, the most famous is `John Searle`'s `Chinese Room experiment`. \
Searle imagines himself locked in a closed room, not understanding Chinese, but with a detailed rulebook at hand. Someone passes a note with a Chinese question through the window. He simply follows the rules in the book to combine the characters into an appropriate answer and passes the answer back out. \
The Chinese Room experiment highlights several key limitations of the Turing test:
1. Syntax vs Semantics: The Turing test relies on external behavior (syntactically correct output), while the Chinese Room experiment emphasizes the importance of semantic understanding—merely generating correct answers does not equate to true understanding. 
2. Imitation rather than intelligence: If the machine's goal is only to imitate human behavior, it may be good at pretending but may not necessarily possess cognitive abilities. For example, modern large language models can simulate conversations, but their understanding ability is questioned. 
3. Lack of consciousness and cognition: Searle points out that even if a system passes the Turing test, it may still lack "consciousness" or "cognition" because it is merely executing rules, not thinking. 

Although there are many who agree and disagree with `John Searle`, his work indeed provides new thinking directions for testing internal machine cognition.

## LLM
Those who understand the principles of large language models (LLMs) know that they are essentially a `black-box experiment`, based on pattern statistics and statistical prediction to answer human questions. Their hallucinatory outputs themselves indicate a lack of true knowledge understanding and reasoning ability. Therefore, in last year's article published in `Nature`, `ChatGPT broke the Turing test — the race is on for new ways to assess AI` ([link](https://www.nature.com/articles/d41586-023-02361-7)), although it stated that `ChatGPT-4` successfully passed the Turing test, people still doubt whether it possesses intelligence.

## Test Set Methods
Because the Turing test has obvious limitations in many aspects mentioned above, people have come up with methods to measure `AI intelligence` using test sets. That is, using different test sets containing various types of questions, testing the accuracy rate, and scoring to quantify the model. This method evaluates AI capabilities from different angles by constructing diverse test sets. \
However, this approach also has obvious drawbacks. First, the model's ability to answer questions does not fully equal its generalization ability. Second, a model being strong in certain specific areas does not mean it has generality. In addition, the model's training data also affects its test scores; the more training in a certain area, the more accurate the answers. These reasons make the results difficult to be representative.

## ARC-AGI Test
In 2019, `François Chollet` proposed `ARC` (Abstraction and Reasoning Corpus) to evaluate the ability of general artificial intelligence. `ARC` is a novel testing method that focuses on measuring a machine's abstract reasoning and inductive abilities.
Example as shown:
![ARC-AGI](https://cdn.jsdelivr.net/gh/byolio/newtc@main/img/ARC-AGI.png)
[ARC-AGI Github Link](https://github.com/fchollet/ARC-AGI?tab=readme-ov-file) \
The AI must find the pattern from the first three sets of data that it should return a red segment different from other red segments, in order to correctly return the red segment of the corresponding image. Because it requires the AI to judge the internal abstract logic of the image, this method requires the AI to have a certain generalization ability, and therefore it can serve as a representative testing method to determine whether the AI possesses intelligence.

## Conclusion
The testing of artificial intelligence, from the early Turing test to modern test set evaluations and then to the `ARC-AGI` testing method, is constantly exploring how to scientifically measure the true capabilities of machine intelligence. In future intelligence evaluation methods, it is even more necessary to balance the model's generalization ability, cognitive depth, and adaptability, in order to truly reveal the potential of AI in complex environments and push it towards true general intelligence.

