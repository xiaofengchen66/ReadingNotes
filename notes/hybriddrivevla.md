# HybridDriveVLA: Vision-Language-Action Model with Visual CoT reasoning and ToT Evaluation for Autonomous Driving

- **Venue / year:** CVPR 2026
- **Authors:** Yipene Cedric Francois Bassole, Sungwoo Kim, Jiwoo Jung, Yunsick Sung
- **Link:** https://openaccess.thecvf.com/content/CVPR2026/html/Bassole_HybridDriveVLA_Vision-Language-Action_Model_with_Visual_CoT_reasoning_and_ToT_Evaluation_CVPR_2026_paper.html ([本地PDF副本](pdf/hybriddrivevla.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

传统VLA模型常靠图像到文本的思维链(CoT)推理，把连续视觉场景转换成文本符号，低效利用视觉信息里的空间上下文。首次把视觉思维链(V-CoT)推理和思维树(ToT)启发的路点评估整合进一个VLA模型：V-CoT预判未来场景作为ToT评估的目标，ToT评估按安全/进度/舒适度生成并打分路点。nuScenes上平均碰撞率0.17%，超过传统VLA模型。

---

## 跟本repo已有内容的关系

跟MoE路由关系不大，是"推理结构设计(CoT+ToT)"这条技术路线，收录因为核心类目相关。
