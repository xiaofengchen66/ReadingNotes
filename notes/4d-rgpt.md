# 4D-RGPT: Toward Region-level 4D Understanding via Perceptual Distillation

- **Venue / year:** arXiv 2512.17012, 2025 (CVPR 2026, NVIDIA)
- **Authors:** Chiao-An Yang, Ryo Hachiuma, Sifei Liu, Subhashree Radhakrishnan, Raymond A. Yeh, Yu-Chiang Frank Wang, Min-Hung Chen
- **Link:** https://arxiv.org/abs/2512.17012 ([本地PDF副本](pdf/4d-rgpt.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

多模态大模型在3D结构+时序动态上推理能力弱，受限于4D感知不足。三个贡献：4D-RGPT(专精捕捉4D表示的MLLM)；Perceptual 4D Distillation(P4D)训练框架，把4D表示从一个冻结的专家模型蒸馏进4D-RGPT；R4D-Bench(深度感知动态场景+区域级prompt的基准)。

---

## 跟本repo已有内容的关系

"从冻结专家模型蒸馏4D表示进MLLM"这个思路，跟PerceptDrive"冻结Perception Provider+蒸馏先验"的设计模式一致，只是应用在通用4D理解任务而非驾驶规划上，可作为同一设计模式的又一例证。
