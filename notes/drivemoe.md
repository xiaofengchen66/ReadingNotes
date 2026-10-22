# DriveMoE: Mixture-of-Experts for Vision-Language-Action Model in End-to-End Autonomous Driving

- **Venue / year:** arXiv 2505.16278, 2025 (CVPR 2026)
- **Authors:** Zhenjie Yang, Yilin Chai, Xiaosong Jia, Qifeng Li, Yuqian Shao, Xuekai Zhu, Haisheng Su, Junchi Yan
- **Link:** https://arxiv.org/abs/2505.16278 ([本地PDF副本](pdf/drivemoe.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

两个MoE模块：Scene-Specialized Vision MoE用router动态选择相关摄像头视角(模仿人类驾驶员选择性关注关键视觉线索，不穷举处理全部视觉信息)；Skill-Specialized Action MoE用另一个router激活不同驾驶行为(车道保持/避障/激进机动等)对应的专精专家，避免"众口难调"式的mode-averaging。底座是VLA baseline Drive-π0。Bench2Drive闭环评测SOTA。

---

## 跟本repo已有内容的关系

这是SAMoE-VLA笔记里反复点名批评的那篇(Yang et al.)——SAMoE-VLA批评它"用预定义router标签、限定摄像头视角选择的做法引入安全隐患"。现在终于有了自己的独立笔记，后续可以直接对照SAMoE-VLA笔记里的批评原文核实。
