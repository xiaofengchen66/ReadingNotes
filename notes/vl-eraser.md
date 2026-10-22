# VL-Eraser: Vacuum Distillation for Machine Unlearning in Vision-Language Models

- **Venue / year:** CVPR 2026
- **Authors:** (作者见CVPR页面)
- **Link:** https://cvpr.thecvf.com/virtual/2026/poster/39075 ([本地PDF副本](pdf/vl-eraser.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

VLM机器遗忘任务：从预训练模型里移除敏感/不想要的内容。传统遗忘方法会破坏跨模态对齐，导致多模态场景下遗忘不彻底。VL-Eraser把遗忘重新表述成两阶段：蒸馏+删除。真空蒸馏(vacuum distillation)把不想要的知识从VLM参数里解耦出来，迁移进LoRA适配器；蒸馏完之后，通过直接删除LoRA参数高效完成遗忘。

---

## 跟本repo已有内容的关系

跟驾驶关系不大，是VLM安全/隐私方向的蒸馏应用，收录因为"把特定知识解耦进独立的LoRA模块再整体移除"这个思路，是"怎么模块化地管理模型里的特定知识"这个更大问题的一个有意思的解法。
