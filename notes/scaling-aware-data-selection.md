# Scaling-Aware Data Selection for End-to-End Autonomous Driving Systems

- **Venue / year:** arXiv 2604.08366, 2026 (CVPR 2026, NYU/NVIDIA)
- **Authors:** Tolga Dimlioglu, Nadine Chang, Maying Shen, Rafid Mahmood, Jose M. Alvarez
- **Link:** https://arxiv.org/abs/2604.08366 ([本地PDF副本](pdf/scaling-aware-data-selection.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

MOSAIC(Mixture Optimization via Scaling-Aware Iterative Collection)：通用数据选择框架，把数据集按领域划分，为每个数据领域拟合神经缩放律(对评测指标)，迭代地从能最大化指标变化的领域添加数据来优化数据混合比例。应用到端到端驾驶规划器(用EPDMS评测)，用最多少80%的数据就超过一组多样化基线。

---

## 跟本repo已有内容的关系

跟MoE路由关系不大，是"数据选择/课程学习"这条技术路线，收录因为核心类目相关。
