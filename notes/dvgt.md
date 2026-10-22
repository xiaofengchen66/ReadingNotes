# DVGT: Driving Visual Geometry Transformer

- **Venue / year:** arXiv 2512.16919, 2025 (CVPR 2026, Tsinghua/Xiaomi EV)
- **Authors:** Sicheng Zuo, Zixun Xie, Wenzhao Zheng, Shaoqing Xu, Fang Li, Shengyin Jiang, Long Chen, Zhi-Xin Yang, Jiwen Lu
- **Link:** https://arxiv.org/abs/2512.16919 ([本地PDF副本](pdf/dvgt.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

从一串未摆姿的多视角视觉输入重建全局稠密3D点云图。用DINO主干给每张图像提取视觉特征，交替用视图内局部注意力、跨视图空间注意力、跨帧时序注意力推断图像间的几何关系。不依赖显式3D几何先验，能灵活处理任意相机配置；直接预测度量尺度的几何，不需要和外部传感器做后对齐。在nuScenes/OpenScene/Waymo/KITTI/DDAD等驾驶数据集混合训练。

---

## 跟本repo已有内容的关系

跟MoE路由关系不大，是纯几何重建的基础模型，收录因为跟PerceptDrive的几何专家(VGGT蒸馏来源)属于同一类"驾驶视觉几何Transformer"，可以作为VGGT的同类对照。
