# HG-Lane: High-Fidelity Generation of Lane Scenes under Adverse Weather and Lighting Conditions without Re-annotation

- **Venue / year:** arXiv 2603.10128, 2026 (CVPR 2026)
- **Authors:** Daichao Zhao, Qiupu Chen, Feng He, Xin Ning, Qiankun Li
- **Link:** https://arxiv.org/abs/2603.10128 ([本地PDF副本](pdf/hg-lane.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

现有车道数据集(CULane/TuSimple)在极端天气下数据有限，导致检测模型在极端条件下不可靠。不需要重新标注，生成恶劣天气/光照条件下的高保真车道场景。以SOTA的CLRNet为例，基准上整体mF1提升20.87%，F1@50在雪天+38.8%、雾天+26.84%、夜间+21.5%。代码和数据集已开源。

---

## 跟本repo已有内容的关系

跟MoE路由关系不大，是数据增强/生成这条技术路线，收录因为核心类目相关。
