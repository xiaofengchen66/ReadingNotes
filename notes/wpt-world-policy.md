# WPT: World-to-Policy Transfer via Online World Model Distillation

- **Venue / year:** arXiv 2511.20095, 2025 (CVPR 2026)
- **Authors:** Guangfeng Jiang, Yueru Luo, Jun Liu, Yi Huang, Yiyao Zhu, Zhan Qu, Dave Zhenyu Chen, Bingbing Liu, Xu Yan
- **Link:** https://arxiv.org/abs/2511.20095 ([本地PDF副本](pdf/wpt-world-policy.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

在端到端世界模型指导下做在线蒸馏的训练范式。训练一个可训练的奖励模型，把世界知识注入教师策略：让候选轨迹跟世界模型预测的未来动态对齐；提出策略蒸馏+世界奖励蒸馏，把教师的推理能力迁移进轻量学生策略。开环碰撞率0.11，闭环driving score 79.23，超过基于世界模型和模仿学习的方法；学生策略推理快4.9倍，同时保留大部分收益。

---

## 跟本repo已有内容的关系

**驾驶相关，明确标注，而且是这批里跟驾驶规划关系最直接的一篇**。"世界模型指导在线蒸馏"+"学生策略推理快4.9倍"直接回应了"SA-MoE没有真正省计算量"这个局限——这篇证明了用世界模型蒸馏可以让学生策略又快又好，是"怎么让驾驶规划模型又快又聪明"这个问题的一个完整解法，值得优先精读。
