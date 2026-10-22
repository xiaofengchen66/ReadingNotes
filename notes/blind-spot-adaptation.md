# The Blind Spot of Adaptation: Quantifying and Mitigating Forgetting in Fine-tuned Driving Models

- **Venue / year:** arXiv 2604.04857, 2026 (CVPR 2026)
- **Authors:** Runhao Mao, Hanshi Wang, Yixiang Yang, Qianli Ma, Jingmeng Zhou, Zhipeng Zhang
- **Link:** https://arxiv.org/abs/2604.04857 ([本地PDF副本](pdf/blind-spot-adaptation.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

把VLM整合进自动驾驶有望解决长尾场景，但微调让VLM适配驾驶专用数据的过程同时侵蚀了其宝贵的预训练世界知识。构建18万场景的新大规模数据集，建立首个专门量化自动驾驶中灾难性遗忘的基准。提出Drive Expert Adapter(DEA)：把适配从权重空间转移到prompt空间，按场景专属线索动态路由推理到不同的知识专家，从而规避这个权衡。驾驶任务上SOTA，同时有效缓解灾难性遗忘。

---

## 跟本repo已有内容的关系

**这篇是这个大批次里跟持续学习/路由漂移问题关系最直接、也最驾驶专用的一篇**——"把适配从权重空间转移到prompt空间，动态路由到不同知识专家"直接回应了on-tokens-dilemma、spectral-moe-continual、prompt-anchored-reid讨论过的"MoE/VLM怎么防止微调/持续学习时遗忘"这个问题，而且是专门针对驾驶VLM场景设计的，建议优先精读，跟之前那几篇的解法做系统对比。
