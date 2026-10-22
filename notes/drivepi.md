# DrivePI: Spatial-aware 4D MLLM for Unified Autonomous Driving Understanding, Perception, Prediction and Planning

- **Venue / year:** arXiv 2512.12799, 2025 (CVPR 2026)
- **Authors:** Zhe Liu, Runhui Huang, Rui Yang, Siming Yan, Zining Wang, Lu Hou, Di Lin, Xiang Bai, Hengshuang Zhao
- **Link:** https://arxiv.org/abs/2512.12799 ([本地PDF副本](pdf/drivepi.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

空间感知的4D MLLM，作为统一VLA框架，端到端优化下并行完成空间理解、3D感知(占据栅格)、预测(占据流)、规划(动作输出)。跟主流只靠摄像头的VLA方法不同，引入LiDAR作为补充感知模态，提供精确3D几何信息更好地激发MLLM的空间理解能力。只用0.5B的Qwen2.5当MLLM主干，作为单一统一模型就能匹配或超过现有VLA模型和专用VA模型。相比OpenDriveVLA-7B在nuScenes-QA上平均准确率+2.5%，相比ORION碰撞率从0.37%降到0.11%(-70%)。代码已开源。

---

## 跟本repo已有内容的关系

**这是"全程(感知+预测+规划)覆盖"做得最完整的一篇之一**(跟Drive-KD相比，DrivePI是单一统一模型端到端联合优化，不是分开蒸馏三个能力)——而且只用0.5B参数就超过7B模型，直接回应"怎么又快又好"这个问题，值得跟朱子欣讨论"全程MoE"这个方向时对照参考：DrivePI证明了统一4D理解+规划可以做到很小的模型规模。
