# GTR-Turbo: Merged Checkpoint is Secretly a Free Teacher for Agentic VLM Training

- **Venue / year:** arXiv 2512.13043, 2025 (CVPR 2026)
- **Authors:** Tong Wei, Yijun Yang, Changhao Zhang, Junliang Xing, Yuanchun Shi, Zongqing Lu, Deheng Ye
- **Link:** https://arxiv.org/abs/2512.13043 ([本地PDF副本](pdf/gtr-turbo.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

多模态agent的多轮强化学习受限于稀疏奖励和长程信用分配。此前方法(GTR/on-policy distillation)靠查询昂贵的特权教师模型做step级反馈来稠密化奖励。GTR-Turbo：把RL训练过程中产生的多个checkpoint的权重合并，用合并出来的模型当"免费"教师，引导后续RL(通过监督微调或软logit蒸馏)，不需要训练/查询昂贵的教师模型。多个视觉agent任务上准确率提升10-30%，训练时间减半、算力成本降低60%。

---

## 跟本repo已有内容的关系

**"合并checkpoint当免费教师"这个思路直接呼应了SAMoE-VLA的参数融合机制**——都是"不需要额外训练/推理开销，靠权重空间操作(合并/插值)获得新能力"这个大类技巧的具体应用，而且这篇明确算过"免费"带来的具体训练时间/算力节省数字，对回应"SA-MoE没有真正省计算量"这个局限有直接参考价值。
