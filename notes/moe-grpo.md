# MoE-GRPO: Optimizing Mixture-of-Experts via Reinforcement Learning in Vision-Language Models

- **Venue / year:** arXiv 2603.24984, 2026 (CVPR 2026)
- **Authors:** (见CVPR页面，核心作者含 Seoung Choi 等)
- **Link:** https://arxiv.org/abs/2603.24984 ([本地PDF副本](pdf/moe-grpo.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

把专家选择建模成序贯决策问题，用GRPO(Group Relative Policy Optimization)优化，让模型通过探索和奖励反馈学习自适应的专家路由策略，解决确定性top-K路由可能忽略更优专家组合、导致专家过拟合的问题。引入模态感知的router引导，阻止router探索某个模态很少被激活的专家。多模态图像/视频基准上持续优于标准top-K路由及其变体。

---

## 跟本repo已有内容的关系

用强化学习(而非监督/蒸馏信号)训练路由器是第四种路由训练范式，跟PerceptDrive用"privileged metric distillation"监督路由器形成对比——这篇用RL奖励代替特权监督信号，训练信号来源不同但目标类似(让路由学得更好，不是盲目端到端)。
