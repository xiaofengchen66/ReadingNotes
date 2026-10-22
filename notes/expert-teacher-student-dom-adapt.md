# Expert-Teacher-Student Collaborative Learning for Domain Adaptive Object Detection

- **Venue / year:** CVPR 2026
- **Authors:** Yiming Cui, Liang Li, Haibing Yin, Yuhan Gao, Xichun Sheng, Chenggang Yan
- **Link:** https://cvpr.thecvf.com/virtual/2026/poster/38408 ([本地PDF副本](pdf/expert-teacher-student-dom-adapt.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

域自适应目标检测：把源域训练的检测器泛化到目标域。ETS(Expert-Teacher-Student)协作学习框架，协同视觉基础模型的通用知识和教师模型的领域专属知识。ETCT(Expert-Teacher协作教学)模块用互补知识生成高质量伪标签；ETJC(Expert-Teacher联合巩固)模块在专家/教师/学生三者间引入类别原型对齐。把视觉基础模型当"免费午餐"式的专家模型用，不需要额外大量训练成本。

---

## 跟本repo已有内容的关系

"Expert-Teacher-Student"三层结构(不是传统的两层teacher-student)，跟PerceptDrive的"冻结Provider(专家)+训练WAM Backbone(学生)"结构类似，但这篇额外插入了一个"teacher"层做领域适配——如果PerceptDrive这类设计要跨数据集/跨城市做域自适应，这篇的三层协作结构值得参考。
