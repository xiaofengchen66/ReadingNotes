# CogDriver: Integrating Cognitive Inertia for Temporally Coherent Planning in Autonomous Driving

- **Venue / year:** arXiv 2509.00789, 2025 (CVPR 2026, HKUST + Li Auto)
- **Authors:** (香港科技大学 + 理想汽车团队)
- **Link:** https://arxiv.org/abs/2509.00789 ([本地PDF副本](pdf/cogdriver.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

当前VLM缺乏"认知惯性"：只处理孤立快照，无法形成时序连贯的世界观，导致决策抖动、无法执行复杂多步机动。CogDriver-Data：大规模VLA数据集，叙事式标注为学习世界模型的时序动态提供监督信号；CogDriver-Agent：带稀疏时序记忆、维持稳定内部状态的架构，通过时空知识蒸馏显式教会决策连贯性。Bench2Drive闭环驾驶分数+22%，nuScenes平均L2误差-21%，SOTA。

---

## 跟本repo已有内容的关系

理想汽车(Li Auto)团队又一篇，跟之前的SMoES是同一家公司。"稀疏时序记忆维持稳定内部状态"这个思路，直接回应了PerceptDrive"H_BEV只算一次、场景突变路由滞后"这个局限——跟之前讨论的几种解法(定期刷新/变化检测触发/EMA持续更新)又是不同的第五种思路：维护一个显式的"认知惯性"记忆模块，专门对抗时序不连贯。
