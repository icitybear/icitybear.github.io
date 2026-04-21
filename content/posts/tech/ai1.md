---
title: "ai大白话" #标题
date: 2026-01-16T14:43:37+08:00 #创建时间
lastmod: 2026-01-16T14:43:37+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- 工具
keywords: 
- 
description: "" #描述 每个文章内容前面的展示描述
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: true #是否展示评论 有自带的扩展成twikoo
showToc: true # 显示目录 文章侧边栏toc目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示当前路径
cover:
    image: "" #图片路径：posts/tech/文章1/picture.png
    caption: "" #图片底部描述
    alt: ""
    relative: false

# reward: true # 打赏
mermaid: true #自己加的是否开启mermaid
---

# 资料

[简单学AI，有一些知识点的图文概要说明和简易交互](https://mmh1.top/ai-navigation?category=AI%E5%86%99%E4%BD%9C%E5%B7%A5%E5%85%B7#/ai-knowledge)

[AI大模型知识图谱](https://www.processon.com/view/link/67bf1c697b5d1a5bcc109a3b)

[提示词](https://github.com/2025Emma/vibe-coding-cn)

# 阶段一：建立直观认知与基础概念

| 知识点      | 专业名称全称       | 一句白话理解     | 举个例子         |
|-------------|----------------------|---------------------------------|------------|
| NLP         | 自然语言处理 (Natural Language Processing) | 教电脑看懂、听懂和说人话的技术。  | 就像手机输入法的智能纠错和预测下一个词。     |
| Token       | 词元 (在分词中的最小单位)     | 模型眼里的“文字积木”，是它处理文本的最小块。   | 句子“你好！”可能被拆成“你”、“好”、“！”三个Token。                       |
| Tokenization| 分词 / 令牌化 (Tokenization)  | 将文本切割成Token的过程，是模型“读”懂文本的第一步。 | 英文“I'm happy”可能切成 ["I", "'", "m", "happy"]；中文“我很开心”可能切成 ["我", "很", "开心"]。 |
| Transformer | Transformer架构 (基于自注意力机制的神经网络) | 让AI能“一眼看完”整个句子并理解词与词关系的核心技术。 | 就像你读文章时，能同时联系前后文理解“它”指的是什么。|
| LLM         | 大语言模型 (Large Language Model)  | 一个读过海量书本的“语言大脑”，能和你聊天、写文章。 | ChatGPT、文心一言这类对话机器人的核心就是LLM。  |
| 多模态      | 多模态人工智能 (Multimodal AI)   | 让AI从“只认字”升级到“能看图、听声、说话”。  | 你可以上传一张图片，让AI描述内容并生成一段故事。 |


# 阶段二：理解核心模型、训练过程与能力边界

| 知识点      | 专业名称全称        | 一句白话理解          | 例子           |
|-------------|--------------------|------------------------------|-----------------|
| BERT        | 来自Transformer的双向编码器表示 (Bidirectional Encoder Representations from Transformers) | 擅长做阅读理解的AI，用来分析文本情感、提取关键信息。 | 用于搜索引擎，更好地理解你的问题意图。   |
| GPT         | 生成式预训练Transformer (Generative Pre-trained Transformer) | 擅长续写和聊天的AI，能根据上文生成连贯的下文。 | 你给出开头“在一个雨夜...”，它能续写一个完整的故事。 |
| Llama      | 大型语言模型元AI (Large Language Model Meta AI)  | Meta公司开源的一个“GPT”，技术社区可以免费研究和改进。 | 许多中小公司和开发者用它来构建自己的AI应用。 |
| T5      | 文本到文本转换Transformer (Text-To-Text Transfer Transformer) | 一个“多面手”AI，能把所有任务都当成“填空”来做。 | 给它指令“把‘你好’翻译成英语：____”，它输出“hello”。 |
| 预训练      | 预训练 (Pre-training)     | 让AI“博览群书”打基础的过程，先学会通用语言规律。   | 就像学生先上通识课，广泛学习语文、数学、常识。   |
| 模型幻觉    | 模型幻觉 / 胡言乱语 (Hallucination)    | AI有时会自信地编造看似合理但完全错误的信息。  | 你问“谁发明了时间机器？”，它可能编出一个不存在的科学家和故事。|
| 模型评估    | 模型评估 (Model Evaluation)    | 给AI出题考试，看它回答得对不对、好不好。       | 用一份有标准答案的测试卷，比较不同模型的得分。  |
| 过拟合      | 过拟合 (Overfitting)  模型成为“死记硬背的书呆子”，对新问题束手无策。       | 把习题答案全背下来，但考试题型一变就不会。    |
| 泛化        | 泛化能力 (Generalization Ability)  | 模型“举一反三”的能力，能处理好没见过的新问题。  | 掌握了数学公式，遇到没见过的应用题也能解。 |

# 阶段三：掌握模型定制与优化技术

| 知识点      | 专业名称全称    | 一句白话理解    | 例子              |
|-------------|---------------|-------------------------|-----------------------|
| 微调        | 微调 / 精调 (Fine-tuning)   | 给通才AI上"特训班"，让它成为某个领域的专家。    | 用一个法律文书库训练通用模型，让它变成"AI律师"。     |
| SFT         | 监督微调 (Supervised Fine-Tuning) | 用高质量的"问答示范"来教AI如何更好地回答问题。 | 用人写的"用户问-专家答"对话数据，训练AI模仿专家口吻。   |
| LoRA        | 低秩自适应 (Low-Rank Adaptation)| 一种"轻量特训法"，只动模型的一小部分参数。      | 想优化一件衣服，LoRA不是重做，而是只改袖口和领子。   |
| 微调参数    | 超参数 (Hyperparameters)    | 特训班的"教学计划"：学多快、学几遍、一次学多少。 | 学习率太高会学得不扎实，太低则学得太慢。  |
| Loss        | 损失函数值 (Loss)       | AI在特训中每次答题的"错误得分"。 | 就像模拟考试，Loss值下降说明AI出错的题越来越少了。  |
| RLHF        | 基于人类反馈的强化学习 (Reinforcement Learning from Human Feedback) | 根据人类的"好/坏"评价调教AI的品味和价值观。   | AI生成两个回答，人类标注哪个更好，AI就从反馈中学习。 |
| MGA         | 机器生成增强 (Machine-Generated Augmentation) | "一句话百样说"，自动改写资料扩充训练数据。 | 把"今天天气很好"自动生成"今日晴空万里"、"天气不错"等。  |
| 模型蒸馏    | 知识蒸馏 (Knowledge Distillation)  | 让"小学生"模型模仿"大学教授"模型。| 让一个能在手机上运行的轻量模型，学会大模型80%的本事。  |
| 模型量化    | 模型量化 (Quantization) | 把AI模型的"体重"减下来，跑得更快更省地方。     | 就像把高清照片转成压缩包，画质略有损失但体积小很多。  |

# 阶段四：探索应用模式与部署实践

| 知识点  | 专业名称全称      | 一句白话理解          | 例子       |
|----------------|-----------------------|---------------|-----------|
| 长文本 & 知识库 | 长文本处理 & 知识库 (Long Context & Knowledge Base) | 给AI准备的"参考书"和"档案库"，用来弥补它记忆的不足。 | 你可以上传一本产品手册作为知识库，让AI基于它回答客户问题。|
| RAG | 检索增强生成 (Retrieval-Augmented Generation) | 让AI在回答前先"翻书查资料"，避免胡说，回答有据可依。| 就像开卷考试，问"公司某政策是什么？"，AI先检索内部文档再回答。 |
| Agent    | 智能体 (Agent)| 一个能自己"思考"并"使用工具"完成复杂任务的AI助理。 | 你让它"订机票"，它会自己搜索航班、比价、填写信息。  |
| Function Calling | 函数调用 (Function Calling)  | AI理解你的需求后，能自动"按按钮"调用其他软件功能。 | 你说"明天早上8点提醒我开会"，AI调用日历软件创建日程。|
| MCP | 模型上下文协议 (Model Context Protocol)    | 一套连接AI和外部工具的"标准插头"，让扩展功能更方便。 | 有了它，AI可以像用USB接口一样，轻松连接各种新工具。   |
| Ollama   | Ollama (一款本地大模型运行框架)   | 一个在你自己电脑上就能轻松下载和运行开源AI模型的"软件商店"。 | 像在电脑上安装一个软件一样，一键安装并运行Llama模型。 |
| VLLM   | 易用大语言模型推理服务 (vLLM: Easy, Fast, and Cheap LLM Serving) | 一个专门用来让AI模型在服务器上高速、高效处理大量请求的"引擎"。 | 像双十一的淘宝服务器，能同时快速处理成千上万个用户的AI请求。  |
| GGUF     | GPT-Generated Unified Format    | 一种特别适合在个人电脑上运行的、压缩好的AI模型"文件格式"。 | 就像.mp3是音乐文件格式，.GGUF是方便本地运行的AI模型格式。  |