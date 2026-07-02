---
layout: post
title: 'Discussion: Is the Turing Test Really Useful for AI?'
subtitle: 简单聊一聊当今如何测试ai的智能
date: 2024-11-16T00:00:00.000Z
author: Byolio
header-img: img/post-bg-TuringTest.png
catalog: true
tags:
  - Discussion
  - AI
  - Turing Test
lang: en
---

> 🌐 [中文](/2024-11-14-TuringTest/) | [English](/en/en/_posts/2024-11-14-TuringTest/)

> This article aims to discuss the currently well-known AI intelligence testing methods

## Introduction
The father of computer science, `Alan Turing`, published a landmark paper in the field of artificial intelligence in 1950: `Computing Machinery and Intelligence`, in which he raised a revolutionary question: `Can machines think?` and described a testing method, which is the most well-known `Turing test`. However, with the advent of the AI wave, whether the Turing test is still valid has become a controversial topic.

## Turing Test
The setup of the `Turing test` is usually as follows: a human judge engages in a conversation with a human and a machine. The judge's task is to determine which is the machine and which is the human. If the judge cannot correctly distinguish, or the probability of misjudgment is very high, then the machine is considered to have passed the `Turing test`. \
The advantage of this test is that it cleverly transforms the complex and difficult-to-judge question of whether a machine can possess consciousness, emotions, or thinking ability into judging whether it can successfully imitate humans. Therefore, many people call it an imitation game.

## Limitations of the Turing Test
As a behavioral imitation game, the Turing test inevitably becomes a benchmark for `behaviorist` judgment and understanding of things, but it will inevitably conflict with `cognitivism`. Cognitivists believe that external `behavioral performance` cannot replace the more important internal `cognition`, and that merely relying on external behavior cannot determine whether a machine possesses intelligence. They conducted the famous `Chinese Room` argument to refute it. \
In addition, factors such as `number of judges`, `threshold for judge misjudgment`, `question types`, `test time and environment`, and `background and ability of judges` have no precise definitions, which affects the results of the Turing test. In particular, whether `the judge uses AI in daily life` has a huge impact on the experimental results. \
Moreover, the Turing test focuses on how to simulate humans, in other words, how to deceive humans. Using this as a goal to deliberately answer questions incorrectly or say "I don't know" is meaningless, because the purpose of artificial intelligence should be to improve productivity and free people from heavy physical labor.

## Chinese Room
Among the experiments opposing the Turing test, the most famous is `John Searle`'s `Chinese Room experiment`. \
Searle imagines himself locked in a closed room, not knowing Chinese, but with a detailed rulebook at hand. Someone passes a note with a Chinese question through the window; he simply combines the characters into an appropriate answer according to the rules in the book, and then passes the answer out. \
The Chinese Room experiment highlights several key limitations of the Turing test:
1. Syntax vs. Semantics: The result of the Turing test depends on external behavior (syntactically correct output), while the Chinese Room experiment emphasizes the importance of semantic understanding—merely generating correct answers does not equate to true understanding.
2. Imitation rather than intelligence: If the machine's goal is only to imitate human behavior, it may be good at pretending but does not necessarily possess cognitive abilities. For example, modern large language models can simulate conversations, but their understanding ability is questioned.
3. Lack of consciousness and cognition: Searle points out that even if a system passes the Turing test, it may still have no "consciousness" or "cognition" because it is merely executing rules, not thinking.

Although there are many who agree and disagree with `John Searle`, his argument indeed provides new thinking directions for testing internal cognition of machines.

## LLM
Those who understand the principles of large language models (LLMs) know that they are essentially a `black-box experiment`, answering human questions based on pattern statistics and statistical prediction. Their hallucinatory outputs themselves indicate a lack of true knowledge understanding and reasoning ability. Therefore, although the article `ChatGPT broke the Turing test — the race is on for new ways to assess AI` ([link](https://www.nature.com/articles/d41586-023-02361-7)) published in `Nature` last year stated that `ChatGPT-4` successfully passed the Turing test, people still doubt whether it possesses intelligence.

## Test Set Methods
Because the Turing test has obvious limitations in many aspects mentioned above, people have come up with methods to measure `AI intelligence` using test sets, i.e., using different test sets containing various types of questions, testing the accuracy rate, and scoring to quantify the model. This method evaluates AI capabilities from different angles by constructing diverse test sets. \
However, this approach still has obvious drawbacks. First, a model's ability to solve problems does not fully equal its generalization ability. Second, a model being strong in certain specific areas does not mean it has generality. In addition, the model's training data also affects its test scores; the more it is trained on a certain aspect, the more accurate its answers are. These reasons make the results difficult to be representative.

## ARC-AGI Test
In 2019, `François Chollet` proposed `ARC` (Abstraction and Reasoning Corpus) to evaluate the capabilities of general artificial intelligence. `ARC` is a novel testing method that focuses on measuring a machine's abstract reasoning and inductive abilities.
Example as shown:
![ARC-AGI](https://cdn.jsdelivr.net/gh/byolio/newtc@main/img/ARC-AGI.png)
[ARC-AGI Github Link](https://github.com/fchollet/ARC-AGI?tab=readme-ov-file) \
The AI must find the pattern from the first three sets of data that it should return a red segment different from other red segments, in order to correctly return the red segment of the corresponding image. Because it requires the AI to judge the intrinsic abstract logic of the image, this method requires the AI to have a certain degree of generalization ability, and therefore it can be a representative testing method for determining whether this AI possesses intelligence.

## Conclusion
The testing of artificial intelligence, from the early Turing test to modern test set evaluations and then to the `ARC-AGI` testing method, has been continuously exploring how to scientifically measure the true capabilities of machine intelligence. In future intelligence evaluation methods, it is even more necessary to balance the model's generalization ability, cognitive depth, and adaptability, in order to truly reveal the potential of AI in complex environments and push it towards true general intelligence.
