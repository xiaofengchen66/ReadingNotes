# Drive My Way: Preference Alignment of Vision-Language-Action Model for Personalized Driving

- **Venue / year:** arXiv 2603.25740, 2026 (CVPR 2026)
- **Authors:** Zehao Wang, Huaide Jiang, Shuaiwu Dong, Yuping Wang, Hang Qiu, Jiachen Li
- **Link:** https://arxiv.org/abs/2603.25740 ([本地PDF副本](pdf/drive-my-way.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

现有端到端驾驶系统要么优化通用目标，要么依赖固定驾驶模式，无法适配个人偏好或理解自然语言意图。DMW(Drive My Way)：从跨多个真实司机采集的个性化驾驶数据集里学一个用户嵌入，规划时用这个嵌入做条件，自然语言指令提供额外的短期引导。Bench2Drive闭环评测显示风格指令适配能力提升，用户研究显示生成的行为能被识别为对应司机自己的风格。

---

## 跟本repo已有内容的关系

跟MoE/路由设计关系不大，是"个性化驾驶偏好对齐"这条技术路线，收录因为核心类目相关。
