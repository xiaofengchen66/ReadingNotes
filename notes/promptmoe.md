# PromptMoE: A Segmentation Refinement Framework Leveraging Mixture of Experts for Improved Prompting

- **Venue / year:** arXiv 2603.12538, 2026 (CVPR 2026)
- **Authors:** Stephen Price, Danielle L. Cote, Elke A. Rundensteiner
- **Link:** https://arxiv.org/abs/2603.12538 ([本地PDF副本](pdf/promptmoe.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

模型无关的MoE驱动prompt精修器，用于分割精修任务(医学诊断、质量控制等对边界精度要求高的场景)。三个协作模块：MoE-based Image-Informed Prompting(IIP)产出专家分数图指导prompt生成；Dynamic Expert Selector(DES)只激活最相关的专家并融合其分数图；Prompt-Placement Explorer(PPE)放置高置信度、空间分散的点prompt。5个基准数据集上显著超过SOTA方法，平均提升+6.24 IoU/+8.99 BIoU。

---

## 跟本repo已有内容的关系

跟驾驶关系不大，是MoE用在"怎么生成更好的分割prompt"这个具体子任务上，Dynamic Expert Selector(只激活最相关专家)这个设计跟ExpertAD的Top-4稀疏选择思路接近。
