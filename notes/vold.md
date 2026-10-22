# VOLD: Reasoning Transfer from LLMs to Vision-Language Models via On-Policy Distillation

- **Venue / year:** arXiv 2510.23497, 2025 (CVPR 2026)
- **Authors:** Walid Bousselham, Hilde Kuehne, Cordelia Schmid
- **Link:** https://arxiv.org/abs/2510.23497 ([本地PDF副本](pdf/vold.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

把纯文本教师模型的推理能力迁移给VLM学生。结合GRPO强化学习和on-policy蒸馏：学生的推理轨迹被教师模型引导，相比单独用GRPO有显著提升。关键发现：冷启动对齐是有效迁移的前提——师生之间分布对齐不够的话，on-policy蒸馏没法提供有意义的指导。MMMU-Pro/MathVision/MathVista/LogicVista等基准上超过SOTA。

---

## 跟本repo已有内容的关系

"RL+on-policy蒸馏结合"跟moe-grpo(纯RL训练路由)是互补的两种路径，"冷启动对齐是有效迁移的前提"这个发现对PerceptDrive/SAMoE-VLA这类需要蒸馏外部先验的设计也有参考价值——如果学生和教师的表示空间一开始差异太大，蒸馏可能根本学不动。
