# ParkGaussian: Surround-view 3D Gaussian Splatting for Autonomous Parking

- **Venue / year:** arXiv 2601.01386, 2026 (CVPR 2026)
- **Authors:** Xiaobao Wei, Zhangjie Ye, Yuxiang Gu, Zunjie Zhu, Yunfei Guo, Yingying Shen, Shan Zhao, Ming Lu, Haiyang Sun, Bing Wang, Guang Chen, Rongfeng Lu, Hangjun Ye
- **Link:** https://arxiv.org/abs/2601.01386 ([本地PDF副本](pdf/parkgaussian.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

首个把3D高斯泼溅(3DGS)用于停车场景重建的框架。构建首个停车场景重建专用基准ParkRecon3D(4个环视鱼眼相机+标定外参+密集停车位标注)。现有工作聚焦2D停车位感知/建图/定位，3D重建在捕捉停车场景复杂空间几何方面尚未被充分探索。提出槽位感知重建策略，利用现有停车感知方法增强槽位区域的合成质量。

---

## 跟本repo已有内容的关系

跟MoE路由关系不大，是停车场景3D重建这个细分任务，收录因为核心类目相关。
