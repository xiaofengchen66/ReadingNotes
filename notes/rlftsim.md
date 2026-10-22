# RLFTSim: Realistic and Controllable Multi-Agent Traffic Simulation via Reinforcement Learning Fine-Tuning

- **Venue / year:** arXiv 2605.19033, 2026 (CVPR 2026, Huawei Canada)
- **Authors:** Ehsan Ahmadi, Hunter Schofield, Behzad Khamidehi, Fazel Arasteh, Jinjun Shan, Lili Mou, Dongfeng Bai, Kasra Rezaee
- **Link:** https://arxiv.org/abs/2605.19033 ([本地PDF副本](pdf/rlftsim.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

强化学习微调框架，让仿真器rollout对齐真实世界数据分布以提升场景真实感，并蒸馏出目标条件可控性。在一个预训练仿真模型基础上实例化，设计平衡保真度和可控性的奖励。Waymo Open Motion Dataset上评测，真实感SOTA，相比启发式搜索式微调方法需要的样本显著更少(低方差稠密奖励信号)。

---

## 跟本repo已有内容的关系

跟MoE路由关系不大，是"交通仿真微调"这条技术路线，收录因为核心类目相关。
