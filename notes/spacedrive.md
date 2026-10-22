# SpaceDrive: Infusing Spatial Awareness into VLM-based Autonomous Driving

- **Venue / year:** arXiv 2512.10719, 2025 (CVPR 2026, Mercedes-Benz/Tübingen)
- **Authors:** Peizheng Li, Zhenghao Zhang, David Holtz, Hang Yu, Yutong Yang, Yuzhi Lai, Rui Song, Andreas Geiger, Andreas Zell
- **Link:** https://arxiv.org/abs/2512.10719 ([本地PDF副本](pdf/spacedrive.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

当前VLM难以理解细粒度3D空间关系，这是跟物理世界交互的系统的基本要求。把空间信息当显式位置编码(PE)而非文本数字token处理，对来自多视角深度估计、历史自车状态、文本提示的所有3D坐标统一用一个通用位置编码器。nuScenes开环SOTA，Bench2Drive闭环驾驶分数78.02，VLM式方法中第二好。

---

## 跟本repo已有内容的关系

跟MoE路由关系不大，是"空间信息表示方式(PE vs文本token)"这条技术路线，收录因为核心类目相关。
