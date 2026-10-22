# Enhancing Mixture-of-Experts Specialization via Cluster-Aware Upcycling

- **Venue / year:** CVPR 2026
- **Authors:** Sanghyeok Chu, Pyunghwan Ahn, Gwangmo Song, Seung Hwan Kim, Honglak Lee, Bohyung Han (LG AI Research)
- **Link:** https://openaccess.thecvf.com/content/CVPR2026/html/Chu_Enhancing_Mixture-of-Experts_Specialization_via_Cluster-Aware_Upcycling_CVPR_2026_paper.html ([本地PDF副本](pdf/cluster-aware-upcycling.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

解决sparse upcycling(从预训练dense模型初始化MoE)时"专家对称、早期缺乏分化"的问题：把dense模型的输入激活按语义聚类，每个专家用对应聚类的子空间表示(截断SVD)初始化，router初始权重设为聚类中心。CLIP ViT-B/32、ViT-B/16上zero-shot/few-shot全面超过现有初始化方法，专家表征更分散、路由更自信。

---

## 跟本repo已有内容的关系

跟SAMoE-VLA/PerceptDrive关心的"路由怎么设计"不同，这篇关心的是"MoE怎么从dense模型初始化"——如果未来要把别人训练好的dense模型升级成MoE(而不是从零训练专家)，这篇的聚类初始化方法是直接可用的技术。
