# ERMoE: Eigen-Reparameterized Mixture-of-Experts for Stable Routing and Interpretable Specialization

- **Venue / year:** arXiv 2511.10971, 2025 (CVPR 2026)
- **Authors:** Anzhe Cheng, Shukai Duan, Shixuan Li, Chenzhong Yin, Mingxi Cheng, Heng Ping, Tamoghna Chattopadhyay, Sophia I. Thomopoulos, Shahin Nazarian, Paul Thompson, Paul Bogdan
- **Link:** https://arxiv.org/abs/2511.10971 ([本地PDF副本](pdf/ermoe.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

每个专家被重参数化到一个学到的正交基(orthonormal eigenbasis)里，路由不再用学习出的gating logits，而是用token特征跟专家基的余弦相似度(Eigenbasis Score)——把"分配给哪个专家"跟"专家自己的表示空间"直接绑定，去掉了显式的负载均衡损失(和它带来的梯度干扰)。ImageNet、图文检索(COCO/Flickr30K)SOTA，3D MRI脑龄预测变体(ERMoE-ba)提升7%+且专家分工有解剖学可解释性。

---

## 跟本repo已有内容的关系

跟SA-MoE(参数融合)、PerceptDrive(场景条件路由)都不同的第三条路由范式：不学门控权重，靠专家自己的几何结构(正交基)决定路由——路由稳定性和可解释性是这篇的卖点，跟驾驶场景的关系需要进一步看是否能应用到scene-level路由上。
