# 1. 导读

## 1.1 Pre-Training 预训练

大模型预训练聚焦大量无标签语料，采用自监督学习，label 与 data 错位构造训练数据集。借下一个token当标签开展自监督，模型依据上下文预测下一个最可能单词，通过计算预测对数似然损失，让模型精准预测下一个单词，进行连续文本的自回归预测，以确保训练全面有效。**这个训练过程无需人工标注，借文本自身构造监督标签，实现模型训练，让模型储备语言语法规则**，让突破数据与知识瓶颈。

## 1.2 SFT 有监督微调

在 `Pre-Training` 完成后，进行 `SFT` 有监督微调。也就是用人工标注的「输入 → 理想输出」数据训练模型。

## 1.3 RLHF 人类反馈强化学习

在 `SFT` 有监督微调后，进行 `RLHF` 人类反馈强化学习（一般是PPO）。也就是用人类偏好来“优化模型行为”

|维度|SFT|RLHF|
|---|---|---|
|学习方式|监督学习|强化学习|
|训练目标|拟合标准答案|优化人类偏好|
|数据形式|(输入, 标准输出)|偏好排序（A 比 B 好）|
|是否涉及 reward|❌|✅|
|作用阶段|初始能力学习|行为对齐（alignment）|

# 2. Pre-Training 预训练

## 2.1 预训练数据

> Datasets for Large Language Models论文链接：/[https://arxiv.org/pdf/2402.18041](https://arxiv.org/pdf/2402.18041)

该论文对LLMs训练数据集进行了全面整理：
- 涵盖8种语言、32个领域，444个数据集
- 五个维度总结现有的代表性LLMs文本数据集，包括Pre-training Corpora、Fine-tuning Instruction Datasets、Preference Datasets、Evaluation Datasets和Traditional NLP Datasets

## 3. SFT 有监督学习