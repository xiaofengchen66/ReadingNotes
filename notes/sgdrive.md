# SGDrive: Scene-to-Goal Hierarchical World Cognition for Autonomous Driving

- **Venue / year:** arXiv 2601.05640, 2026 (CVPR 2026)
- **Authors:** Jingyu Li, Junjie Wu, Dongnan Hu, Xiangkai Huang, Bin Sun, Zhihui Hao, Xianpeng Lang, Xiatian Zhu, Li Zhang
- **Link:** https://arxiv.org/abs/2601.05640 ([本地PDF副本](pdf/sgdrive.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

围绕驾驶专用知识层级显式构建VLM表示学习：把驾驶理解分解成场景-智能体-目标层级，模仿人类驾驶认知(先感知整体环境，再关注安全关键智能体及其行为，最后形成短期目标再执行动作)。结构化的注意力掩码机制防止信息泄漏、抑制跨类别噪声；集成基于DiT的规划器调控轨迹生成。NAVSIM基准上纯摄像头方法里SOTA。代码已开源。

---

## 跟本repo已有内容的关系

"场景-智能体-目标"三层认知层级，跟SAMoE-VLA的CMCA(条件token/动作token分层处理)、DrivePI的统一理解-感知-预测-规划结构，是"怎么把驾驶决策拆层"这个问题的又一种划分方式，可以对照看哪种层级划分效果更好。
