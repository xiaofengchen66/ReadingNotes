# Foundry: Distilling 3D Foundation Models for the Edge

- **Venue / year:** arXiv 2511.20721, 2025 (CVPR 2026)
- **Authors:** Guillaume Letellier, Siddharth Srivastava, Frederic Jurie, Gaurav Sharma
- **Link:** https://arxiv.org/abs/2511.20721 ([本地PDF副本](pdf/foundry-3d-edge.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

把3D基础模型蒸馏成适合边缘设备部署的轻量版本。

---

## 跟本repo已有内容的关系

"3D基础模型蒸馏到边缘设备"直接呼应"SA-MoE没有真正省计算量"这个局限——如果PerceptDrive这类模型最终要部署到车端(边缘设备)，这篇的蒸馏到边缘的具体方法值得细读。
