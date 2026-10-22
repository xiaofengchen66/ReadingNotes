# E3AD: An Emotion-Aware Vision-Language-Action Model for Human-Centric End-to-End Autonomous Driving

- **Venue / year:** arXiv 2512.04733, 2025 (CVPR 2026)
- **Authors:** Yihong Tang, Haicheng Liao, Tong Nie, Junlin He, Ao Qu, Kehua Chen, Wei Ma, Zhenning Li, Lijun Sun, Chengzhong Xu
- **Link:** https://arxiv.org/abs/2512.04733 ([本地PDF副本](pdf/e3ad.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

端到端驾驶VLA通常忽略乘客情绪状态，而这对舒适性和接受度很关键。提出开放域端到端(OD-E2E)驾驶：自动驾驶车辆必须理解自由形式自然语言指令、推断情绪、规划物理可行轨迹。E3AD：连续的Valence-Arousal-Dominance(VAD)情绪模型捕捉语言里的语气和紧迫感+双路径空间推理模块融合自我中心和他者中心视角。真实数据集上视觉接地和路点规划提升，情绪估计VAD相关性SOTA。

---

## 跟本repo已有内容的关系

跟MoE/路由设计关系不大，是"情感感知个性化驾驶"这条技术路线，收录因为核心类目相关。
