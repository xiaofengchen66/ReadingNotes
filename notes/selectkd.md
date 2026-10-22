# SelecTKD: Selective Token-Weighted Knowledge Distillation for LLMs

- **Venue / year:** arXiv 2510.24021, 2025 (CVPR 2026)
- **Authors:** Haiduo Huang, Jiangcheng Song, Yadong Zhang, Pengju Ren
- **Link:** https://arxiv.org/abs/2510.24021 ([本地PDF副本](pdf/selectkd.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

标准知识蒸馏里均匀的逐token损失会放大噪声、高熵信号。SelecTKD把问题从"怎么度量散度"转移到"该在哪里学习"：提出-验证流程(贪心Top-k和非贪心Spec-k两个变体)，被接受的token拿全部损失，被拒绝的token被遮蔽或降权。指令跟随/数学推理/代码生成/VLM设置上持续提升强基线。

---

## 跟本repo已有内容的关系

"不是所有token都值得同等对待"这个核心论点，跟SAMoE-VLA"场景级而非token级路由"的论点是同一个直觉的另一种表现形式——只是SelecTKD应用在蒸馏损失的加权上，SAMoE-VLA应用在MoE路由权重上，两者都在质疑"均匀对待所有token"这个默认假设。
