# SimScale: Learning to Drive via Real-World Simulation at Scale

- **Venue / year:** arXiv 2511.23369, 2025 (CVPR 2026 Oral, OpenDriveLab)
- **Authors:** Haochen Tian, Tianyu Li, Haochen Liu, Jiazhi Yang, Yihang Qiu, Guang Li, Junli Wang, Yinfeng Gao, Zhang Zhang, Liang Wang, Hangjun Ye, Tieniu Tan, Long Chen, Hongyang Li
- **Link:** https://arxiv.org/abs/2511.23369 ([本地PDF副本](pdf/simscale.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

真实世界语料里安全关键/分布外场景严重欠表示。可扩展仿真框架，在已有驾驶日志基础上合成大量未见过的状态：先进神经渲染配反应式环境，生成由扰动自车轨迹控制的高保真多视角观测；为这些新仿真状态开发伪专家轨迹生成机制提供动作监督。真实+仿真样本联合训练策略，显著提升各类规划方法的鲁棒性和泛化性，navhard上+8.6 EPDMS。仅增加仿真数据(不需要额外真实数据流)就能平滑提升策略。代码数据已开源。

---

## 跟本repo已有内容的关系

跟MoE路由关系不大，是"大规模仿真数据扩增"这条技术路线，收录因为核心类目相关。
