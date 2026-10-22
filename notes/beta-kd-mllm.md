# Uncertainty-Aware Knowledge Distillation for Multimodal Large Language Models

- **Venue / year:** arXiv 2603.21426, 2026 (CVPR 2026)
- **Authors:** Jingchen Sun, Shaobo Han, Deep Patel, Wataru Kohno, Can Jin, Changyou Chen
- **Link:** https://arxiv.org/abs/2603.21426 ([本地PDF副本](pdf/beta-kd-mllm.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

Beta-KD：不确定性感知的蒸馏框架，自适应调节学生对教师指导的依赖程度。从统一贝叶斯视角建模师生学习，把教师监督解读成学生激活上的Gibbs先验，得到一个闭式的、不确定性感知的加权机制。多模态VQA基准上持续提升从大教师VLM蒸馏学生VLM的效果。代码已开源。

---

## 跟本repo已有内容的关系

"自适应调节对教师指导的依赖程度"这个思路，如果以后PerceptDrive这类设计要蒸馏多个来源不等可靠的先验(比如某些专家模型置信度更高)，Beta-KD的不确定性加权机制是一个可以直接借用的工具。
