# M4-SAM: Multi-Modal Mixture-of-Experts with Memory-Augmented SAM for RGB-D Video Salient Object Detection

- **Venue / year:** arXiv 2605.11760, 2026 (CVPR 2026)
- **Authors:** Jiyuan Liu, Jia Lin, Xiaofei Zhou, Runmin Cong, Deyang Liu, Zhi Liu
- **Link:** https://arxiv.org/abs/2605.11760 ([本地PDF副本](pdf/m4-sam.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

基于SAM2做RGB-D视频显著目标检测。三个组件：Modality-Aware MoE-LoRA(卷积专家编码局部空间先验+模态分发器，注入SAM2编码器)解决LoRA线性空间建模能力有限的问题；Gated Multi-Level Feature Fusion分层聚合多尺度编码器特征；Pseudo-Guided Initialization用粗糙mask当伪先验，实现zero-shot(无需人工prompt)。三个公开RGB-D VSOD数据集SOTA。

---

## 跟本repo已有内容的关系

MoE-LoRA(把MoE和LoRA结合，用卷积专家做模态感知微调)是一个具体的工程技巧，如果以后要给某个冻结的大模型做轻量、模态感知的微调，这是一个可以参考的实现方式。
