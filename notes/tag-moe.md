# TAG-MoE: Task-Aware Gating for Unified Generative Mixture-of-Experts

- **Venue / year:** arXiv 2601.08881, 2026 (CVPR 2026)
- **Authors:** Yu Xu, Hongbin Yan, Juan Cao, Yiji Cheng, Tiankai Hang, Runze He, Zijin Yin, Shiyi Zhang, Yuxin Zhang, Jintao Li, Chunyu Wang, Qinglin Lu, Tong-Yee Lee, Fan Tang (中科院/腾讯混元/成功大学)
- **Link:** https://arxiv.org/abs/2601.08881 ([本地PDF副本](pdf/tag-moe.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

统一图像生成+编辑模型在共享参数空间上要在互相冲突的目标间妥协，导致严重任务干扰。TAG-MoE把高层任务语义意图注入MoE门控网络的局部路由决策：层级任务语义标注方案构造结构化任务描述符+预测对齐正则化，让内部路由决策跟任务高层语义对齐。

---

## 跟本repo已有内容的关系

"把高层任务意图注入路由决策"这个思路，跟SAMoE-VLA用场景级信号(H_BEV)引导路由、PerceptDrive用scene-conditioned router是同一个大方向(路由要看懂"任务/场景整体是什么"，而不是只看局部token)，这篇是在图像生成/编辑这个不同任务域上的独立验证。
