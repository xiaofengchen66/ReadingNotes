# KnowVal: A Knowledge-Augmented and Value-Guided Autonomous Driving System

- **Venue / year:** arXiv 2512.20299, 2025 (CVPR 2026, 北大/UC Merced)
- **Authors:** Zhongyu Xia, Wenhao Chen, Yongtao Wang, Ming-Hsuan Yang
- **Link:** https://arxiv.org/abs/2512.20299 ([本地PDF副本](pdf/knowval.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

通过开放世界感知和知识检索的协同集成实现视觉-语言推理：构建一个综合驾驶知识图谱，编码交通法规、防御性驾驶原则、伦理规范，配合针对驾驶场景的高效LLM检索机制。包含人类偏好数据集，训练一个价值模型引导可解释、价值对齐的轨迹评估。nuScenes上碰撞率最低，Bench2Drive上SOTA。

---

## 跟本repo已有内容的关系

跟MoE路由关系不大，是"知识图谱+价值对齐"这条技术路线，收录因为核心类目相关。
