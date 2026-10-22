# AMap: Distilling Future Priors for Ahead-Aware Online HD Map Construction

- **Venue / year:** arXiv 2512.19150, 2025 (CVPR 2026)
- **Authors:** Ruikai Li, Xinrun Li, Mengwei Xie, Hao Shan, Shoumeng Qiu, Xinyuan Chang, Yizhe Fan, Feng Xiong, Han Jiang, Yilong Ren, Haiyang Yu, Mu Xu, Yang Long, Varun Ojha, Zhiyong Cui
- **Link:** https://arxiv.org/abs/2512.19150 ([本地PDF副本](pdf/amap.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

在线高精地图构建。首创"从未来蒸馏"范式：一个能访问未来时序上下文的教师模型，引导一个轻量学生模型，把前瞻性知识隐式压缩进学生模型，推理时零额外成本就有"向前看"的能力。nuScenes/Argoverse 2上显著提升当前帧感知，在关键前方区域超过SOTA时序模型。

---

## 跟本repo已有内容的关系

**驾驶相关，明确标注**。"从未来蒸馏"这个思路直接回应了之前讨论的"H_BEV只算一次、场景突变路由滞后"这个局限的另一种解法：不是靠"变化检测触发重算"，而是靠训练时就把"教师能看到的未来"蒸馏进模型，让模型推理时(看不到未来)也带有前瞻性——这是跟PerceptDrive"self-predicted future conditioning"(自预测未来隐特征)不同的实现路径，用蒸馏而不是自监督预测来获得前瞻能力。
