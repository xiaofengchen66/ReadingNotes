# Progressive Mask Distillation for Self-supervised Video Representation

- **Venue / year:** CVPR 2026
- **Authors:** Kewei Wu, Chong Liang, Zhao Xie, Dan Guo
- **Link:** https://cvpr.thecvf.com/virtual/2026/poster/37040 ([本地PDF副本](pdf/progressive-mask-distill.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

掩码视觉建模的自监督学习任务。渐进式掩码蒸馏(PMD)：用动态掩码率实现"由易到难"的渐进语义学习，三个组件——渐进式学生蒸馏器(训练多个掩码率递增的学生模型，前一阶段学到的易学低级语义指导下一阶段学难的高级语义)、难度感知区域增强器(按重建损失学习区域级权重)、跨层特征对齐器。Something-Something V2/Kinetics-400等视频基准SOTA。

---

## 跟本repo已有内容的关系

"由易到难的渐进式学习"这个课程学习(curriculum learning)思路，跟驾驶场景里"先处理简单驾驶场景、再处理复杂场景"的训练策略设计有潜在借鉴意义，尤其是如果SA-MoE/PerceptDrive要做渐进式训练。
