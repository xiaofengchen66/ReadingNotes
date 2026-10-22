# Multimodal Distribution Matching for Vision-Language Dataset Distillation

- **Venue / year:** arXiv 2605.23482, 2026 (CVPR 2026)
- **Authors:** Jongoh Jeong, Hoyong Kwon, Minseok Kim, Kuk-Jin Yoon
- **Link:** https://arxiv.org/abs/2605.23482 ([本地PDF副本](pdf/mdm-vl-dataset-distill.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

几何感知的多模态数据集蒸馏框架MDM。数据层面：从联合嵌入空间的聚类里采样初始化合成图文对；模型层面：按与预训练锚点的角度偏差，在权重空间插值出一个"混合教师"；损失层面：在单位超球面上匹配联合分布。图文检索基准上跨架构评测显示压缩后的合成集保持多模态语义、鲁棒性好。

---

## 跟本repo已有内容的关系

"在权重空间插值多个独立微调模型形成混合教师"这个思路，跟SAMoE-VLA的参数融合(加权合并多个专家的FFN参数)在数学操作上是同一类(权重空间插值/融合)，只是用途不同(一个用于蒸馏教师构造，一个用于MoE推理时的专家融合)。
