# Prune2Drive: A Plug-and-Play Framework for Accelerating Vision-Language Models in Autonomous Driving

- **Venue / year:** arXiv 2508.13305, 2025 (CVPR 2026)
- **Authors:** Minhao Xiong, Zichen Wen, Zhuangcheng Gu, Xuyang Liu, Rui Zhang, Hengrui Kang, Jiabing Yang, Junyuan Zhang, Weijia Li, Conghui He, Linfeng Zhang
- **Link:** https://arxiv.org/abs/2508.13305 ([本地PDF副本](pdf/prune2drive.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

VLM在驾驶里要处理6个以上同步摄像头的高分辨率多视角图像，自注意力的二次复杂度导致推理延迟和显存消耗巨大。即插即用的多视角VLM视觉token剪枝框架：多样性感知的token选择机制(受最远点采样启发，优先保证跨视角的语义和空间覆盖，而不是只靠注意力分数)+视角自适应剪枝控制器(按每个摄像头视角对下游驾驶任务的重要性自动学习最优剪枝比例)。不需要重新训练模型或访问注意力图，跟现代高效注意力实现兼容。

---

## 跟本repo已有内容的关系

**直接回应"SA-MoE没有真正省计算量"这个局限的又一个具体方案**——"视角自适应剪枝控制器"跟场景条件路由(PerceptDrive)逻辑类似，都是"根据重要性动态决定资源分配"，但这篇是即插即用、不需重训，门槛更低，值得优先参考。
