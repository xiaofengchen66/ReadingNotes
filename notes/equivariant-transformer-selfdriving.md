# Efficient Equivariant Transformer for Self-Driving Agent Modeling

- **Venue / year:** arXiv 2604.01466, 2026 (CVPR 2026, Waabi/University of Toronto)
- **Authors:** Scott Xu, Dian Chen, Kelvin Wong, Chris Zhang, Kion Fallah, Raquel Urtasun
- **Link:** https://arxiv.org/abs/2604.01466 ([本地PDF副本](pdf/equivariant-transformer-selfdriving.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

DriveGATr：不用现有方法那种高昂计算代价就能实现SE(2)等变性(场景旋转平移时模型行为可预测地变化)的Transformer架构。把场景元素编码成2D投影几何代数里的多向量，用一叠等变Transformer块处理；多向量之间用标准注意力建模几何关系，不需要昂贵的显式成对相对位置编码。Waymo Open Motion Dataset上交通仿真表现跟SOTA相当，性能-计算开销的帕累托前沿更优。

---

## 跟本repo已有内容的关系

跟MoE/路由设计关系不大，是"几何等变性+计算效率"这条技术路线，收录因为核心类目相关，"不需要昂贵显式相对位置编码"这个效率优化思路可以类比MoE设计里"不需要昂贵显式路由计算"的追求。
