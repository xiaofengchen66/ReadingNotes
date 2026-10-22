# Unifying Language-Action Understanding and Generation for Autonomous Driving

- **Venue / year:** arXiv 2603.01441, 2026 (CVPR 2026)
- **Authors:** Xinyang Wang, Qian Liu, Wenjie Ding, Zhao Yang, Wei Li, Chang Liu, Bailin Li, Kun Zhan, Xianpeng Lang, Wei Chen
- **Link:** https://arxiv.org/abs/2603.01441 ([本地PDF副本](pdf/unifying-language-action.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

LinkVLA：解决VLA模型两个关键局限。结构性连接：把语言和动作token统一进一个单一多模态模型内共享的离散码本，结构性地强制跨模态一致性；辅助动作理解目标：训练模型从轨迹生成描述性字幕，培养双向语言-动作映射。用两步由粗到细的生成方法替代慢速逐步生成，高效解码动作序列，节省86%推理时间。

---

## 跟本repo已有内容的关系

"语言和动作token共享一个离散码本"这个思路，是"怎么统一语言理解和动作生成"这个VLA核心问题的一个具体解法，跟"节省86%推理时间"的效率数字一起，直接回应"怎么又快又统一"的需求。
