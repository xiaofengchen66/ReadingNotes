# Deformable Gaussian Occupancy: Decoupling Rigid and Nonrigid Motion with Factorized Distillation

- **Venue / year:** arXiv 2605.28587, 2026 (CVPR 2026)
- **Authors:** Yang Gao, Wuyang Li, Po-Chien Luan, Alexandre Alahi
- **Link:** https://arxiv.org/abs/2605.28587 ([本地PDF副本](pdf/deformable-gaussian-occupancy.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

DeGO框架：把解耦的高斯形变和分解式4D基础模型蒸馏统一起来，解耦刚性和非刚性运动，让每个高斯基元通过形变和基于偏移的更新两种方式演化。Occ3D-NuScenes基准弱监督下SOTA，人体中心实例+13.5%，整体+10.9%。

---

## 跟本repo已有内容的关系

**驾驶相关，明确标注**。这是占据栅格(occupancy)预测里"怎么用蒸馏获得更精细的运动建模"的具体做法，跟PerceptDrive的DSE(可变形场景编码器，按距离调整采样密度)用的都是可变形(deformable)机制处理驾驶场景里的空间异质性，可以互相对照。
