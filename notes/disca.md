# DisCa: Accelerating Video Diffusion Transformers with Distillation-Compatible Learnable Feature Caching

- **Venue / year:** arXiv 2602.05449, 2026 (CVPR 2026)
- **Authors:** Chang Zou, Changlin Li, Yang Li, Patrol Li, Jianbing Wu, Xiao He, Songtao Liu, Zhao Zhong, Kailin Huang, Linfeng Zhang
- **Link:** https://arxiv.org/abs/2602.05449 ([本地PDF副本](pdf/disca.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

视频扩散Transformer加速。首个"可学习特征缓存+蒸馏"方案：用轻量可学习神经预测器代替传统的训练无关启发式缓存，更准确捕捉高维特征演化；配合Restricted MeanFlow(一种更保守的训练方案，裁剪掉高度压缩的场景，让视频生成更稳定)。几乎无损加速11.8倍。

---

## 跟本repo已有内容的关系

视频生成加速技术，跟驾驶世界模型(GAIA-1/GAIA-2/OmniDreams)的"怎么快速生成未来帧"这个需求直接相关——如果以后驾驶世界模型要做实时推理，这篇的可学习特征缓存思路可以参考。
