# Soft Modality-Guided Expert Specialization in MoE-VLMs (SMoES)

- **Venue / year:** arXiv 2604.23996, 2026 (CVPR 2026)
- **Authors:** Zi-Hao Bo, Yaqian Li, Anzhou Hou, Rinyoichi Takezoe, Ertao Zhao, Tianxiang Pan, Jiale Yan, Mo Guang, Kaiwen Long (理想汽车)
- **Link:** https://arxiv.org/abs/2604.23996 ([本地PDF副本](pdf/smoes.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

探究模态特定信号该怎么指导MoE-VLM里的专家路由。动态软模态分数捕捉层级依赖的融合模式+专家分箱机制(跟专家并行部署对齐)+箱间互信息正则化鼓励一致的模态专精。4个MoE-VLM、16个基准上验证：多模态任务+0.9%、纯语言任务+4.2%，专家并行通信开销降低56.1%，真实部署下吞吐量提升12.3%。

---

## 跟本repo已有内容的关系

作者来自理想汽车(Li Auto)，虽然这篇本身是通用VLM-MoE工作不是驾驶专用，但出自汽车公司、且"模态引导专家路由"的思路跟CMCA(条件token静态可见、动作token因果可见)有相通之处，值得留意理想汽车在MoE+VLM方向的其他工作。
