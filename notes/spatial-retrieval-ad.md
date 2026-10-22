# Spatial Retrieval Augmented Autonomous Driving

- **Venue / year:** arXiv 2512.06865, 2025 (CVPR 2026)
- **Authors:** Xiaosong Jia, Chenhe Zhang, Yule Jiang, Songbur Wong, Zhiyuan Zhang, Chen Chen, Shaofeng Zhang, Xuanhe Zhou, Xue Yang, Junchi Yan, Yu-Gang Jiang
- **Link:** https://arxiv.org/abs/2512.06865 ([本地PDF副本](pdf/spatial-retrieval-ad.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

现有驾驶系统靠车载传感器感知环境，受限于行驶时的感知视野，在视野受限/遮挡/极端条件(黑暗/雨)下常失效。空间检索范式：引入离线检索的地理图像作为额外输入，这些图像能从离线缓存(Google Maps或已有驾驶数据集)轻松获取、不需要额外传感器，是对现有驾驶任务的即插即用扩展。扩展nuScenes数据集加入通过Google Maps API检索的地理图像，在目标检测/在线建图/占据预测/端到端规划/生成式世界建模五个核心任务上建立基线。

---

## 跟本repo已有内容的关系

跟MoE路由关系不大，是"离线地图检索增强"这条技术路线，收录因为核心类目相关，"不需要额外传感器的即插即用扩展"这个设计哲学值得借鉴。
