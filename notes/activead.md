# ActiveAD: Planning-Oriented Active Learning for End-to-End Autonomous Driving

- **Venue / year:** arXiv 2403.02877, 2024 (CVPR 2026)
- **Authors:** Han Lu, Xiaosong Jia, Yichen Xie, Wenlong Liao, Xiaokang Yang, Junchi Yan
- **Link:** https://arxiv.org/abs/2403.02877 ([本地PDF副本](pdf/activead.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

端到端可微学习对高质量标注数据(3D框、语义分割)需求极大，标注成本高昂。提出规划导向的主动学习方法：按多样性和对规划路线的有用性标准，逐步标注收集到的原始数据的一部分。只用30%的nuScenes数据就达到跟SOTA端到端方法相当的性能。

---

## 跟本repo已有内容的关系

跟驾驶/MoE路由关系不大，是数据标注效率优化，收录因为属于"driving models/planning"核心类目。
