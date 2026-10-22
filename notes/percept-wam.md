# Percept-WAM: Perception-Enhanced World-Awareness-Action Model for Robust End-to-End Autonomous Driving

- **Venue / year:** arXiv 2511.19221, 2025 (CVPR 2026, Yinwang Intelligent Technology + Fudan)
- **Authors:** Jianhua Han, Meng Tian, Jiangtong Zhu, Fan He, Huixin Zhang, Sitong Guo, Dechang Zhu, Hao Tang, Pei Xu, Yuze Guo, Minzhe Niu, Haojie Zhu, Qichao Dong, Xuechao Yan, Siyuan Dong, Lu Hou, Qingqiu Huang, Xiaosong Jia, Hang Xu
- **Link:** https://arxiv.org/abs/2511.19221 ([本地PDF副本](pdf/percept-wam.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

**注意：这篇跟PerceptDrive名字很像、方向也相关，但是不同的论文、不同的作者团队。** 首次在单个VLM内隐式集成2D/3D场景理解能力。不靠QA式空间推理，把2D/3D感知任务统一成World-PV和World-BEV token，同时编码空间坐标和置信度。栅格条件预测机制做密集物体感知，结合IoU感知打分和并行自回归解码。下游感知基准上匹配或超过经典检测器/分割器，COCO 2D检测51.7 mAP、nuScenes BEV 3D检测58.9 mAP。

---

## 跟本repo已有内容的关系

**命名与PerceptDrive高度相似，务必区分**：Percept-WAM是"感知增强的世界感知-动作模型"(单VLM隐式集成2D/3D感知)，PerceptDrive是"感知先验世界-动作建模+自适应专家路由"(冻结先验+场景条件路由)。两者都在做"World-Action Model"但架构完全不同——Percept-WAM没有冻结专家/路由机制，是单一VLM直接输出统一的感知token。整理朱子欣汇报材料时务必不要把这两篇搞混。
