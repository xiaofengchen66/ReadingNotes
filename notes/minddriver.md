# MindDriver: Introducing Progressive Multimodal Reasoning for Autonomous Driving

- **Venue / year:** arXiv 2602.21952, 2026 (CVPR 2026)
- **Authors:** Lingjun Zhang, Yujian Yuan, Changjie Wu, Xinyuan Chang, Xin Cai, Shuang Zeng, Linzhe Shi, Sijin Wang, Hang Zhang, Mu Xu
- **Link:** https://arxiv.org/abs/2602.21952 ([本地PDF副本](pdf/minddriver.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

渐进式多模态推理框架，让VLM模仿人类渐进式思考方式：语义理解→语义到物理空间想象→物理空间轨迹规划。反馈引导的自动数据标注pipeline生成对齐的多模态推理训练数据；渐进式强化微调方法，通过渐进的高层奖励学习优化对齐。nuScenes开环和Bench2Drive闭环评测都表现优越。代码已开源。

---

## 跟本repo已有内容的关系

"语义理解→空间想象→轨迹规划"三阶段渐进式推理，跟progressive-mask-distill(渐进式掩码蒸馏)、masters-vlm-distill(渐进式恢复教师容量)是同一个"由简到难渐进式"设计哲学在驾驶VLA上的体现。
