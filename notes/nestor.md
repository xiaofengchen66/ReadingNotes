# NESTOR: A Nested MOE-based Neural Operator for Large-Scale PDE Pre-Training

- **Venue / year:** arXiv 2602.22059, 2026 (CVPR 2026)
- **Authors:** Dengdi Sun, Xiaoya Zhou, Xiao Wang, Hao Si, Wanli Lyu, Jin Tang, Bin Luo (安徽大学)
- **Link:** https://arxiv.org/abs/2602.22059 ([本地PDF副本](pdf/nestor.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

大规模PDE(偏微分方程)预训练神经算子。嵌套MoE设计：image级MoE捕捉全局依赖，token级Sub-MoE捕捉局部依赖，可以针对给定输入选择性激活最合适的专家网络，提升泛化性和可迁移性。12个PDE数据集上大规模预训练，成功迁移到下游任务。

---

## 跟本repo已有内容的关系

跟驾驶/视觉关系不大，但"image级MoE(全局)+token级Sub-MoE(局部)"这种嵌套/分层路由粒度设计，直接呼应了之前讨论的"scene-level vs token-level路由粒度要匹配任务需求"这个核心论点——NESTOR是在PDE领域独立得出类似"要分层处理全局/局部粒度"的设计。
