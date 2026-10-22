# Diffusion Forcing Planner: History-Annealed Planning with Time-Dependent Guidance for Autonomous Driving

- **Venue / year:** arXiv 2606.11019, 2026 (CVPR 2026)
- **Authors:** Zehan Zhang, Neng Zhang, Yaoyi Li, Jia Cai, Zhiling Wang
- **Link:** https://arxiv.org/abs/2606.11019 ([本地PDF副本](pdf/diffusion-forcing-planner.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

基于学习的运动规划器常有时序不一致问题，跨帧小扰动累积成不稳定轨迹，损害闭环驾驶的舒适性和安全性；把历史当静态条件信号注入，容易让规划器照抄历史模式而不是适配环境上下文。DFP(Diffusion Forcing Planner)把完整轨迹拆成历史/当前/未来三段，给每段分配独立的噪声水平，联合去噪历史和未来段，形成异构联合扩散过程；推理时用classifier-free guidance，以退火历史可控地引导未来采样。nuPlan闭环评测+消融显示生成连续、稳定、可控的运动规划。

---

## 跟本repo已有内容的关系

"给历史和未来分配不同噪声水平、联合去噪"这个思路，是处理"历史信息该怎么用"这个问题的又一种方案，跟PerceptDrive的"action-free future latent conditioning"类似但更精细(显式区分历史/当前/未来三段)，可以作为PerceptDrive未来预测机制的对照参考。
