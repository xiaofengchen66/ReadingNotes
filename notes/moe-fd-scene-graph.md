# Mixture-of-Experts based Feature Decoupling for Open Vocabulary Scene Graph Generation

- **Venue / year:** CVPR 2026
- **Authors:** Yiming Li, Sisi You, Bing-Kun Bao (合肥工业大学 + 鹏城实验室)
- **Link:** https://cvpr.thecvf.com/virtual/2026/poster/37100 ([本地PDF副本](pdf/moe-fd-scene-graph.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

开放词汇场景图生成(Open Vocabulary SGG)任务，物体/关系类别不再局限于预定义集合。MoE-FD框架：用多个专家自适应学习物体和关系的特征解耦，通过门控网络权重突出关键特征；用迭代cross-attention建模物体和关系之间的语义交互，增强关系三元组关联和视觉-语义对齐。Visual Genome数据集上大量实验验证有效性。

---

## 跟本repo已有内容的关系

跟驾驶关系不大，是"用MoE做特征解耦"这一类思路在场景图生成任务上的应用，划分依据是"物体特征 vs 关系特征"这两类不同性质的信息，而不是场景类型。
