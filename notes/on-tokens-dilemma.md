# On Token's Dilemma: Dynamic MoE with Drift-Aware Token Assignment for Continual Learning of Large Vision Language Models

- **Venue / year:** arXiv 2603.27481, 2026 (CVPR 2026)
- **Authors:** Chongyang Zhao, Mingsong Li, Haodong Lu, Dong Gong (UNSW Sydney)
- **Link:** https://arxiv.org/abs/2603.27481 ([本地PDF副本](pdf/on-tokens-dilemma.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

持续学习场景下，即使做了专家隔离，MoE模型依然会遗忘，根源是"路由漂移"(routing-drift)：旧任务的token被错误吸引到新加入的专家上。提出LLaVA-DyMoE：动态扩展MoE+漂移感知的token分配，token级分配引导把模糊/旧token推离新专家、保住已有路由模式，配合路由分数正则化促进专家组分离和新专家专精。比基线平均最终准确率提升7%+，遗忘率降低12%。

---

## 跟本repo已有内容的关系

跟之前讨论的"H_BEV只算一次、场景突变路由滞后"这个局限是同一类问题的另一种表现——这篇处理的是"持续学习过程中路由会漂移"，如果SAMoE-VLA/PerceptDrive要做持续学习/在线更新，这篇的"漂移感知token分配"思路直接相关。
