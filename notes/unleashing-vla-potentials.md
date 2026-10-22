# Unleashing VLA Potentials in Autonomous Driving via Explicit Learning from Failures

- **Venue / year:** arXiv 2603.01063, 2026 (CVPR 2026, Tsinghua/UMacau/BJTU)
- **Authors:** Yuechen Luo, Fang Li, Qimao Chen, Shaoqing Xu, Jiaxin Liu, Ziying Song, Zhi-xin Yang, Fuxi Wen
- **Link:** https://arxiv.org/abs/2603.01063 ([本地PDF副本](pdf/unleashing-vla-potentials.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

驾驶VLA模型在RL优化时常遇到性能瓶颈，这种停滞源于受此前监督微调限制的探索能力，导致长尾场景里持续失败。ELF-VLA：用结构化诊断反馈增强强化学习，产出识别具体失效模式的详细可解释报告，而不是依赖模糊的标量奖励。公开NAVSIM基准上整体PDMS、EPDMS分数和高层规划准确率SOTA。

---

## 跟本repo已有内容的关系

"结构化诊断反馈代替模糊标量奖励"这个思路，如果SAMoE-VLA/PerceptDrive要用RL训练路由器(类似moe-grpo的路线)，这篇"怎么给RL提供更精细反馈"的方法值得参考。
