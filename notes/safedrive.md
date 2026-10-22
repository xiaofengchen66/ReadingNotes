# SafeDrive: Fine-Grained Safety Reasoning for End-to-End Driving in a Sparse World

- **Venue / year:** arXiv 2602.18887, 2026 (CVPR 2026)
- **Authors:** Jungho Kim, Jiyong Oh, Seunghoon Yu, Hongjae Shin, Donghyuk Kwak, Jun Won Choi
- **Link:** https://arxiv.org/abs/2602.18887 ([本地PDF副本](pdf/safedrive.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

通过轨迹条件的稀疏世界模型做显式、可解释的安全推理。SWNet(稀疏世界网络)：构建轨迹条件的稀疏世界，仿真关键动态智能体和道路实体的未来行为，为下游推理提供交互为中心的表示；FRNet(细粒度推理网络)：评估智能体特定的碰撞风险和对可行驶区域的时序遵循，精确识别未来时间步的安全关键事件。核心洞察：显式实例级交互建模(而非整体场景编码)对精确识别安全关键事件是必要的。NAVSIM上PDMS 91.6、EPDMS 87.5(12146个场景中仅61次碰撞，0.5%)，Bench2Drive闭环驾驶分数66.8%。

---

## 跟本repo已有内容的关系

"轨迹条件的稀疏世界模型，仿真关键动态智能体"这个思路，跟PerceptDrive的"self-predicted future latent conditioning"是同一方向(用预测未来辅助规划)，但SafeDrive做的是实例级(每个智能体单独建模)而非整体场景级预测，粒度更细，是"场景级 vs 实例级"又一个值得对照的维度。
