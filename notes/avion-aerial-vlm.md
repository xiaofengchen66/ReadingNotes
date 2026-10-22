# AVION: Aerial Vision-Language Instruction from Offline Teacher to Prompt-Tuned Network

- **Venue / year:** CVPR 2026
- **Authors:** Yu Hu, Jianyang Gu, Hao Liu, Yue Cao, Jozsef Hamari, Zheng Liu, Mohsen Zardadi
- **Link:** https://arxiv.org/abs/2603.12659 ([本地PDF副本](pdf/avion-aerial-vlm.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

VLM适配遥感影像任务。教师模块收集LLM描述构造语义丰富的文本原型，用遥感图像特征验证有效性；学生模块把轻量可学习prompt整合进视觉和语言编码器，由教师引导对齐嵌入及其跨模态关系。6个光学遥感基准上提升少样本分类和基类准确率，且不损害对新类别的泛化。

---

## 跟本repo已有内容的关系

跟驾驶关系不大，是VLM适配遥感这个垂直领域的蒸馏案例，收录因为涉及航拍视角(跟自动驾驶的鸟瞰图BEV视角有一定几何相似性)下的VLM prompt蒸馏设计。
