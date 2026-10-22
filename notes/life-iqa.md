# Life-IQA: Boosting Blind Image Quality Assessment through GCN-enhanced Layer Interaction and MoE-based Feature Decoupling

- **Venue / year:** arXiv 2511.19024, 2025 (CVPR 2026)
- **Authors:** Long Tang, Guoquan Zheng, Jie Hao, Jianbo Zhang, Huiyu Duan, Liang Yuan, Guangtao Zhai
- **Link:** https://arxiv.org/abs/2511.19024 ([本地PDF副本](pdf/life-iqa.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

无参考图像质量评估(BIQA)任务。GCN增强的层间交互模块：用GCN增强的最深层特征当query，倒数第二层特征当key/value做cross-attention；MoE-based特征解耦模块：把融合后的表示拆解给不同专家，每个专家专精某类失真类型或质量维度。多个BIQA基准SOTA，CSIQ上SROCC/PLCC达0.966/0.971，常常超过参数量多50%的模型。

---

## 跟本repo已有内容的关系

跟驾驶/MoE路由设计关系不大，是"每个专家专精一种失真类型"这种按任务子类型划分专家的思路，跟ExpertAD"8个功能专家"的划分逻辑类似(按子任务而非按知识来源划分专家)。
