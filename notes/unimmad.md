# UniMMAD: Unified Multi-Modal and Multi-Class Anomaly Detection via MoE-Driven Feature Decompression

- **Venue / year:** arXiv 2509.25934, 2025 (CVPR 2026)
- **Authors:** Yuan Zhao, Youwei Pang, Lihe Zhang, Hanqi Liu, Jiaming Zuo, Huchuan Lu, Xiaoqi Zhao (大连理工IIAU-Lab等)
- **Link:** https://arxiv.org/abs/2509.25934 ([本地PDF副本](pdf/unimmad.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

统一多模态+多类别异常检测框架，用"通用→专用"范式的MoE驱动特征解压机制。编码阶段：多模态输入压缩成紧凑通用特征(抑制潜在异常、鼓励跨模态交互)；解码阶段：通用特征通过稀疏门控的交叉MoE解压成模态专属和类别专属形式，动态按输入模态和类别选择专家路径。分组动态过滤+MoE-in-MoE结构把MoE参数量减少约75%。9个异常检测数据集(3个领域、12种模态、66个类别)上SOTA。

---

## 跟本repo已有内容的关系

"MoE-in-MoE结构把参数量减少75%"这个具体的参数效率技巧，直接回应了之前讨论的"SA-MoE没有真正省计算量"这个局限——如果要让驾驶MoE模型既保持路由灵活性又真正省参数/计算，这篇的MoE-in-MoE设计值得细读。
