# Quant Experts: Token-aware Adaptive Error Reconstruction with Mixture of Experts for Large Vision-Language Models Quantization

- **Venue / year:** arXiv 2602.24059, 2026 (CVPR 2026)
- **Authors:** (西安交通大学团队)
- **Link:** https://arxiv.org/abs/2602.24059 ([本地PDF副本](pdf/quant-experts.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

VLM训练后量化(PTQ)任务。观察到重要通道的分布和出现频率在不同模态、不同token间差异很大。把重要通道分成token无关和token相关两组：前者用共享专家(低秩适配器)补偿全局量化误差；后者用多个路由低秩适配器专家补偿特定token相关的局部量化误差。2B到70B参数规模上持续提升量化后精度，接近全精度模型表现。

---

## 跟本repo已有内容的关系

跟驾驶关系不大，但"共享专家处理通用情况+路由专家处理特定情况"这个思路，跟SAMoE-VLA/PerceptDrive里"固定处理+场景特定调整"的分工逻辑是同一个思想在量化任务上的应用。
