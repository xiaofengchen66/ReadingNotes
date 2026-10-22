# TrafficAlign: Aligning Large Language Models for Traffic Scenario Generation

- **Venue / year:** arXiv 2606.29097, 2026 (CVPR 2026)
- **Authors:** Zhi Tu, Liangkun Niu, Tianyi Zhang
- **Link:** https://arxiv.org/abs/2606.29097 ([本地PDF副本](pdf/trafficalign.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

用大语言模型基于真实驾驶视频生成交通场景、做数据验证、并把LLM跟合成场景对齐。默认用Llama-3.2-3B-Instruct作为生成器模型，用从真实驾驶视频合成的交通场景描述微调LLM。生成的场景比SOTA方法平均多暴露10.8%的碰撞；用生成场景微调驾驶模型使碰撞率降低36.1%；在6个地理多样区域展现跟对应交通分布的强对齐。

---

## 跟本repo已有内容的关系

跟MoE路由关系不大，是"LLM生成对抗性/长尾交通场景"这条技术路线，收录因为核心类目相关。
