# GuideFlow: Constraint-Guided Flow Matching for Planning in End-to-End Autonomous Driving

- **Venue / year:** arXiv 2511.18729, 2025 (CVPR 2026)
- **Authors:** Lin Liu, Caiyan Jia, Guanyi Yu, Ziying Song, Junqiao Li, Feiyang Jia, Peiliang Wu, Xiaoshuai Hao, Yadan Luo
- **Link:** https://arxiv.org/abs/2511.18729 ([本地PDF副本](pdf/guideflow.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

约束流匹配(Constrained Flow Matching)：显式建模流匹配过程以缓解模式坍塌，允许来自各种条件信号的灵活引导，直接在流匹配生成过程内部强制执行显式约束(而非依赖隐式约束编码)。把流匹配训练和基于能量的模型(EBM)统一起来，增强模型自主优化、稳健满足物理约束的能力。Bench2Drive/NuScenes/NavSim/ADV-NuScenes等主流基准上评测，NavSim test hard split上EPDMS 43.0 SOTA。

---

## 跟本repo已有内容的关系

跟SAMoE-VLA/WAM-Flow一样都用flow matching生成轨迹，这篇的贡献是"怎么在流匹配过程里显式强制约束"，如果PerceptDrive/SAMoE-VLA这类用flow matching的模型要加入更强的物理约束保证，这篇的约束流匹配机制值得参考。
