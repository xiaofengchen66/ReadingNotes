# EVLF: Early Vision-Language Fusion for Generative Dataset Distillation

- **Venue / year:** arXiv 2603.07476, 2026 (CVPR 2026)
- **Authors:** Wenqi Cai, Yawen Zou, Guang Li, Chunzhi Gu, Chao Zhang
- **Link:** https://arxiv.org/abs/2603.07476 ([本地PDF副本](pdf/evlf.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

数据集蒸馏(把大训练集压缩成小合成集)任务。在编码器和生成主干的衔接处做早期视觉-语言融合：加一个轻量cross-attention模块，让早期表示同时编码局部纹理和全局语义方向。即插即用，可以接入任何带编码器的扩散式数据集蒸馏pipeline。代码已开源。

---

## 跟本repo已有内容的关系

跟驾驶/MoE关系不大，是"数据集蒸馏"这个独立子领域里的VLM化尝试，收录是因为涉及vision-language融合机制设计，可作为"怎么把语言信号早期融入视觉表示"的一个参考案例。
