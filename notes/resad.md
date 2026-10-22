# ResAD: Normalized Residual Trajectory Modeling for End-to-End Autonomous Driving

- **Venue / year:** arXiv 2510.08562, 2025 (CVPR 2026)
- **Authors:** (见项目页面 duckyee728.github.io/ResAD)
- **Link:** https://arxiv.org/abs/2510.08562 ([本地PDF副本](pdf/resad.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

端到端驾驶系统受轨迹数据固有的时空不均衡挑战：造成显著优化负担，让模型学到虚假相关性而非稳健驾驶逻辑，同时过度偏重不确定的远处预测、损害即时安全。不直接预测未来轨迹，而是重新表述成预测相对于一个确定性惯性参考(强物理先验)的残差偏差，迫使模型把容量聚焦在学习必要的、由上下文驱动的偏离上。逐点归一化重新加权优化目标，防止远处不确定路点的大幅误差主导学习信号。NAVSIM v1/v2上PDMS 88.8、EPDMS 85.5，SOTA。

---

## 跟本repo已有内容的关系

"预测相对于物理先验的残差，而不是绝对轨迹"这个重参数化思路，是一个通用但很有效的设计技巧，跟SAMoE-VLA的"参数融合(相对基准加权)"不是同一个层面，但"引入强先验简化学习目标"的设计哲学是相通的。
