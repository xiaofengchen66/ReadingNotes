# Prompt-Anchored Vision-Text Distillation for Lifelong Person Re-identification

- **Venue / year:** CVPR 2026
- **Authors:** Wen Wen, Hao Chen, Shiliang Zhang
- **Link:** https://arxiv.org/abs/2605.05027 ([本地PDF副本](pdf/prompt-anchored-reid.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

终身行人重识别(LReID)面临语义漂移、灾难性遗忘。用预训练VLM里冻结的文本编码器当跨域稳定的语义锚点。PAD(Prompt-Anchored vision-text Distillation)：文本侧蒸馏prompt以在固定语义空间下保持视觉-文本对齐；视觉侧用EMA教师+自适应prompt池，给新域分配新槽位、冻结旧槽位。代码已开源。

---

## 跟本repo已有内容的关系

"冻结文本编码器当语义锚点"+"EMA教师+自适应prompt池为新域分配新槽位、冻结旧槽位"，这个设计直接对应之前讨论的"MoE持续学习怎么防止路由漂移"这个问题——"新增槽位+冻结旧槽位"是跟on-tokens-dilemma(漂移感知token分配)、spectral-moe-continual(频谱正交)不同的第三种防遗忘机制。
