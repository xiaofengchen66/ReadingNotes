# Learning to Drive is a Free Gift: Large-Scale Label-Free Autonomy Pretraining from Unposed In-The-Wild Videos

- **Venue / year:** arXiv 2602.22091, 2026 (CVPR 2026, Applied Intuition/Stanford/UC Berkeley)
- **Authors:** Matthew Strong, Wei-Jer Chang, Quentin Herau, Jiezhi Yang, Yihan Hu, Chensheng Peng, Wei Zhan
- **Link:** https://arxiv.org/abs/2602.22091 ([本地PDF副本](pdf/learning-to-drive-free-gift.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

无标签、教师引导的框架，直接从未摆姿视频里学习自动驾驶表示：前馈架构配轻量自回归模块，用多模态监督信号联合预测当前和未来点云图、相机姿态、语义布局、运动mask。多模态教师提供序列级伪监督，让模型从原始YouTube视频(无姿态、无标签、无LiDAR)学到统一的伪4D表示。编码器迁移到NAVSIM规划任务，仅用单目摄像头就超过多摄像头和LiDAR基线。

---

## 跟本repo已有内容的关系

"多模态教师提供伪监督，从无标注视频学统一4D表示"的思路，跟PerceptDrive的Perception Provider构建方式(蒸馏多个教师模型)类似，但这篇的教师提供的是伪标签监督信号而不是直接的特征蒸馏，是"怎么获得感知先验"这个问题的又一种路线(自监督伪标签 vs 直接特征蒸馏)。
