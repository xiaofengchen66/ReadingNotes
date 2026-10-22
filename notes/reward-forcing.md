# Reward Forcing: Efficient Streaming Video Generation with Rewarded Distribution Matching Distillation

- **Venue / year:** arXiv 2512.04678, 2025 (CVPR 2026)
- **Authors:** Yunhong Lu, Yanhong Zeng, Haobo Li, Hao Ouyang, Qiuyu Wang, Ka Leong Cheng, Jiapeng Zhu, Hengyuan Cao, Zhipeng Zhang, Xing Zhu, Yujun Shen, Min Zhang
- **Link:** https://arxiv.org/abs/2512.04678 ([本地PDF副本](pdf/reward-forcing.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

流式视频生成。现有滑窗注意力方法把初始帧当固定"锚点token"，导致后续帧过度依赖这些静态token、运动退化。EMA-Sink：维护固定大小的token，用指数滑动平均持续融合被淘汰的token，同时捕捉长期上下文和近期动态；Rewarded Distribution Matching Distillation(Re-DMD)：用VLM给动态程度打分，让输出分布偏向高奖励(更有动感)的区域。单张H100上23.1 FPS。

---

## 跟本repo已有内容的关系

跟驾驶世界模型的"流式/实时生成未来帧"需求直接相关，EMA-Sink这种"固定大小token+滑动平均更新"的机制，也可以类比成H_BEV(PerceptDrive)"只算一次、不随时间更新"这个局限的一种解法：与其完全不更新或每次重算，EMA式持续融合是第三种选择。
