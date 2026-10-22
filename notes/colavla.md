# ColaVLA: Leveraging Cognitive Latent Reasoning for Hierarchical Parallel Trajectory Planning in Autonomous Driving

- **Venue / year:** arXiv 2512.22939, 2025 (CVPR 2026, Tsinghua/CUHK/Didi)
- **Authors:** Qihang Peng, Xuesong Chen, Chenye Yang, Shaoshuai Shi, Hongsheng Li
- **Link:** https://arxiv.org/abs/2512.22939 ([本地PDF副本](pdf/colavla.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

解决VLM式规划器的三个问题：离散文本推理和连续控制不匹配、自回归思维链解码延迟高、规划器效率低/非因果限制实时部署。认知潜在推理器(Cognitive Latent Reasoner)：通过自车自适应选择，把场景理解压缩成紧凑的、决策导向的meta-action嵌入，只用两次VLM前向传播；层级并行规划器(Hierarchical Parallel Planner)：一次前向传播生成多尺度、因果一致的轨迹。nuScenes开环+闭环都SOTA，效率和鲁棒性也好。代码已开源。

---

## 跟本repo已有内容的关系

"只用两次VLM前向传播就压缩出决策导向的meta-action嵌入"，直接回应了"SA-MoE没有真正省计算量"这个局限——这是驾驶VLA里"怎么减少推理时VLM前向次数"的一个具体解法，跟SAMoE-VLA/PerceptDrive每层都要跑一遍MoE/路由比，这篇是从根上减少VLM调用次数。
