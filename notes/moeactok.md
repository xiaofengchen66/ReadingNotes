# MoEActok: A MoE-based Action Tokenizer for Vision-Language-Action Models

- **Venue / year:** CVPR 2026
- **Authors:** Chunpu Xu, Zhixuan Liang, Tianshuo Yang, Chi-Min Chan, Yang Xiao, Jessie Wang, Xiaokang Yang, Yao Mu (PolyU/SJTU/Shanghai AI Lab/HKU/HKUST)
- **Link:** https://cvpr.thecvf.com/virtual/2026/poster/36457 ([本地PDF副本](pdf/moeactok.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

针对"传统方法用单一tokenizer处理整段操作轨迹，但轨迹里混杂多种不同技能，导致优化目标互相妥协"这个问题。用聚类驱动的MoE VQ-VAE：每个专家专精一种技能。三个组件：动作-技能解耦(k-means聚类动作片段，按技能分组)；技能感知训练范式(用技能条件上下文增强VLA模型)；适配器把共享编码器表示投影到技能专属的隐空间做专精量化，再统一回共享空间供解码器重建。RoboTwin/Simpler-Env仿真器+3个真实世界任务上显著超过现有离散tokenizer方法。

---

## 跟本repo已有内容的关系

这是把MoE用在"动作token化"这一步，而不是用在感知或规划的特征处理上——专家按"技能类型"划分(而不是场景类型或知识来源)，是另一种专家划分维度，值得跟ExpertAD的8个功能专家、SAMoE-VLA的4个通用FFN专家做对比。
