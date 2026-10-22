# Masking Teacher and Reinforcing Student for Distilling Vision-Language Models

- **Venue / year:** arXiv 2512.22238, 2025 (CVPR 2026, NVIDIA)
- **Authors:** Byung-Kwan Lee, Yu-Chiang Frank Wang, Ryo Hachiuma
- **Link:** https://arxiv.org/abs/2512.22238 ([本地PDF副本](pdf/masters-vlm-distill.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

大规模VLM部署到移动/边缘设备困难。Masters(Masking Teacher and Reinforcing Student)：掩码渐进式强化学习蒸馏框架，先遮蔽教师的非主导权重降低不必要的复杂度，再训练中逐渐恢复教师容量，让学生平滑、稳定地学到更丰富的表示；离线RL阶段整合准确率奖励+蒸馏奖励两种互补奖励。

---

## 跟本repo已有内容的关系

"先简化教师、再逐渐恢复复杂度"这个渐进式蒸馏思路，是又一个"课程学习式蒸馏"的案例，跟progressive-mask-distill(渐进式掩码率)是同一个设计哲学(由简到难)的不同实现。
