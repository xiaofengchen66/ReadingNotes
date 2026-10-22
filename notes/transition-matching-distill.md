# Transition Matching Distillation for Fast Video Generation

- **Venue / year:** arXiv 2601.09881, 2026 (CVPR 2026, NVIDIA/NYU)
- **Authors:** Weili Nie, Julius Berner, Nanye Ma, Chao Liu, Saining Xie, Arash Vahdat
- **Link:** https://arxiv.org/abs/2601.09881 ([本地PDF副本](pdf/transition-matching-distill.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

把视频扩散模型蒸馏成少步生成器。核心思路：不把扩散网络当成一个整体映射(大多数现有方法这么做)，而是匹配多步去噪轨迹与一个少步概率转移过程，每个转移建模成一个轻量条件流，显式利用大型视频扩散主干里固有的层级结构和语义递进。Wan文生视频模型上验证，速度-质量权衡优于现有蒸馏方法。

---

## 跟本repo已有内容的关系

跟DisCa、Reward Forcing是同一个"视频生成加速"方向的不同技术路线，三篇可以放在一起比较：DisCa做特征缓存、Reward Forcing做流式+奖励引导、这篇做轨迹转移匹配。
