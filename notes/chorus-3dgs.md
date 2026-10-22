# Chorus: Multi-Teacher Pretraining for Holistic 3D Gaussian Scene Encoding

- **Venue / year:** arXiv 2512.17817, 2025 (CVPR 2026 Oral)
- **Authors:** Yue Li, Qi Ma, Runyi Yang, Mengjiao Ma, Bin Ren, Nikola Popovic, Nicu Sebe, Theo Gevers, Luc Van Gool, Danda Pani Paudel, Martin R. Oswald
- **Link:** https://arxiv.org/abs/2512.17817 ([本地PDF副本](pdf/chorus-3dgs.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

多教师预训练框架，学一个整体的前馈式3D高斯泼溅(3DGS)场景编码器，从2D基础模型蒸馏互补信号。共享3D编码器+教师专属投影器，从语言对齐、通才、物体感知三类教师学习，鼓励一个从高层语义到细粒度结构都能捕捉的共享嵌入空间。只用高斯的中心/颜色/估计法向当输入，用39.9倍更少的训练场景就超过点云基线。

---

## 跟本repo已有内容的关系

"共享编码器+教师专属投影器，从多个不同类型的教师(语言对齐/通才/物体感知)学习互补信号"，是"多教师蒸馏"这个方法在3D场景理解上的一个很完整的范例，可以直接对照Drive-KD的"感知-推理-规划"三教师设计，看看教师类型划分的合理性。
