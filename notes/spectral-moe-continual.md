# Spectral Mixture-of-Experts for Continual Learning

- **Venue / year:** CVPR 2026
- **Authors:** Chen Yin, Xingbo Dong, Xuelin Shen, Zhe Jin
- **Link:** https://openaccess.thecvf.com/content/CVPR2026/html/Yin_Spectral_Mixture-of-Experts_for_Continual_Learning_CVPR_2026_paper.html ([本地PDF副本](pdf/spectral-moe-continual.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

解决MoE做参数高效持续学习时的两个失效模式：结构干扰(专家更新互相干扰)和组合性遗忘(路由策略漂移)。三个组件：Spectral Experts用不重叠的频谱掩码把可学参数限制在不同频率子空间，先验保证更新正交、防止结构干扰；Dual-Router把在线路由(学新任务)和离线记忆(归档历史专家重要性)解耦；离线记忆支撑Dynamic Consistency Projection，一个几何约束，抑制路由漂移、按专家历史贡献自适应保护专家，缓解组合性遗忘。代码已开源：github.com/ouycc/Spectral_MoE。

---

## 跟本repo已有内容的关系

跟"on-tokens-dilemma"是同一类问题(MoE持续学习中的路由漂移)的不同解法，一个用token分配引导、一个用频谱正交化+几何约束——如果以后要做"MoE驾驶模型怎么在线持续更新"，这两篇是直接相关的参考方案，而且这篇代码已开源可以直接跑。
