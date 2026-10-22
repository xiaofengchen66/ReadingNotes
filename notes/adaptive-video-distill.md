# Adaptive Video Distillation: Mitigating Oversaturation and Temporal Collapse in Few-Step Generation

- **Venue / year:** arXiv 2603.21864, 2026 (CVPR 2026)
- **Authors:** Yuyang You, Yongzhi Li, Jiahui Li, Yadong Mu, Quan Chen, Peng Jiang
- **Link:** https://arxiv.org/abs/2603.21864 ([本地PDF副本](pdf/adaptive-video-distill.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

视频生成模型蒸馏成少步生成器时的两个失效模式：过饱和(自适应回归损失动态调整空间监督权重来防止)、时序崩塌(时序正则化损失促使采样轨迹平滑、物理合理)；另外用推理时帧插值策略减少采样开销。VBench/VBench2上全面超过现有蒸馏基线。

---

## 跟本repo已有内容的关系

视频生成蒸馏的失效模式分析，跟VidPrism点名的"专家同质化"类似，都是"做蒸馏/MoE设计时容易踩的具体坑"，可以归进同一个"失效模式案例库"。
