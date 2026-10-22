# Local Precise Refinement: A Dual-Gated Mixture-of-Experts for Enhancing Foundation Model Generalization against Spectral Shifts

- **Venue / year:** arXiv 2603.13352, 2026 (CVPR 2026)
- **Authors:** Xi Chen, Maojun Zhang, Yu Liu, Shen Yan
- **Link:** https://arxiv.org/abs/2603.13352 ([本地PDF副本](pdf/local-precise-refinement.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

双门控(dual-gated) MoE机制，对视觉和深度特征做逐token路由，用于细粒度特征调制；专家被参数化为自适应token(低秩分解，学习低维矩阵相乘生成自适应token)，用于增强基础模型against光谱偏移(spectral shift)的泛化能力。

---

## 跟本repo已有内容的关系

token级(不是scene级)双门控路由的一个具体实现，可以跟SAMoE-VLA消融实验里"token级软路由(SoftMoE-style)"这一类做法做对照参考。
