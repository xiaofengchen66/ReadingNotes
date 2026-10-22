# MoDES: Accelerating Mixture-of-Experts Multimodal Large Language Models via Dynamic Expert Skipping

- **Venue / year:** arXiv 2511.15690, 2025 (CVPR 2026)
- **Authors:** Yushi Huang, Zining Wang, Zhihang Yuan, Yifu Ding, Ruihao Gong, Jinyang Guo, Xianglong Liu, Jun Zhang
- **Link:** https://arxiv.org/abs/2511.15690 ([本地PDF副本](pdf/modes.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

首个训练无关(training-free)的MoE多模态大模型专家跳过框架。核心问题：直接把专家跳过方法用到MLLM上会掉点严重，因为没考虑不同MoE层里专家贡献的异质性、以及不同模态token行为的差异。解法：全局调制的局部门控机制(把全局层级重要性融入局部路由概率)+双模态阈值方法(不同模态token分开处理)+前沿搜索算法(把收敛时间从几天压缩到几小时)。Qwen3-VL-MoE-30B上跳过88%专家，性能反而提升10.67%(97.33% vs 86.66%)。

---

## 跟本repo已有内容的关系

这篇直接对应之前讨论过的"SA-MoE没有真正省计算量"这个局限——MoDES是专门做"怎么在MoE推理时真正跳过不需要的专家、省计算量"的工作，如果要把这个思路搬到驾驶MoE场景(比如SAMoE-VLA/PerceptDrive)上，这篇是直接相关的参考文献。
