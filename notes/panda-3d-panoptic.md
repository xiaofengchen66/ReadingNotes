# PanDA: Unsupervised Domain Adaptation for Multimodal 3D Panoptic Segmentation in Autonomous Driving

- **Venue / year:** arXiv 2604.19379, 2026 (CVPR 2026)
- **Authors:** Yining Pan, Shijie Li, Yuchen Wu, Xulei Yang, Na Zhao
- **Link:** https://arxiv.org/abs/2604.19379 ([本地PDF副本](pdf/panda-3d-panoptic.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

多模态3D全景分割的无监督域适应首个研究。现有监督方法严重依赖LiDAR和RGB之间强跨模态互补性，在某个模态退化(光照差/恶劣天气)的域偏移下很脆弱；常规伪标签只保留高置信度区域，导致掩码碎片化、物体监督不完整。非对称多模态丢弃：源域里模拟模态退化的数据增强策略，促进域不变表示学习；DualRefine：利用互补的2D视觉和3D几何先验做伪标签精修。从SemanticKITTI迁移到nuScenes(传感器配置和地理位置都变化更大)提升53.3%。

---

## 跟本repo已有内容的关系

跟MoE路由关系不大，是"域自适应3D分割"这条技术路线，收录因为核心类目相关。
