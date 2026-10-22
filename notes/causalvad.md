# CausalVAD: De-confounding End-to-End Autonomous Driving via Causal Intervention

- **Venue / year:** arXiv 2603.18561, 2026 (CVPR 2026)
- **Authors:** (复旦大学/Embodiq Robotics/北京理工大学/华东师范大学团队)
- **Link:** https://arxiv.org/abs/2603.18561 ([本地PDF副本](pdf/causalvad.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

规划导向的端到端驾驶模型容易学到统计相关性而非真正的因果关系，导致模型用数据集偏差走捷径，损害复杂场景下的可靠性和安全性(因果混淆)。核心是稀疏因果干预方案(SCIS)：轻量级即插即用模块，在神经网络里实例化后门调整理论，构建代表潜在驾驶上下文的原型字典，用这个字典对模型的稀疏向量化查询做干预，主动消除混杂因子导致的虚假关联。nuScenes等基准上规划精度和安全性SOTA，对数据偏差和噪声场景的鲁棒性更强。

---

## 跟本repo已有内容的关系

"因果干预消除虚假关联"这个思路，跟MoE路由设计关系不大，但"构建原型字典、对查询做干预"这个机制模式，跟SA-MoE/PerceptDrive"用场景信号调整处理方式"有一定结构相似性(都是用一个额外信号去调整主模型的处理过程)。
