# MoRE: 3D Visual Geometry Reconstruction Meets Mixture-of-Experts

- **Venue / year:** arXiv 2510.27234, 2025 (CVPR 2026)
- **Authors:** Jingnan Gao, Zhe Wang, Xianze Fang, Xingyu Ren, Zhuo Chen, Shengqi Liu, Yuhao Cheng, Jiangjing Lyu, Xiaokang Yang, Yichao Yan (上海交大 + 阿里)
- **Link:** https://arxiv.org/abs/2510.27234 ([本地PDF副本](pdf/more-3d-geometry.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

密集3D视觉基础模型，用MoE架构动态把特征路由给任务专属专家，让专家在互补的数据方面各自专精，提升可扩展性和适应性。前馈式模型，输入未摆姿图像，输出高质量3D点云图；带置信度的深度精修模块稳定几何估计；融合稠密语义特征和全局对齐的3D主干表示，做高保真表面法线预测。多个基准SOTA，下游应用无需额外计算。

---

## 跟本repo已有内容的关系

跟PerceptDrive的DSE(可变形场景编码器)类似，都是"先做几何/场景编码，再用MoE路由"的思路，可以作为PerceptDrive的几何编码部分的对照/参考实现。
