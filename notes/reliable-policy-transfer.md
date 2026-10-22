# Reliable Policy Transfer for Safety-Aware End-to-End Driving with Deep Reinforcement Learning

- **Venue / year:** CVPR 2026
- **Authors:** Uddin Md. Borhan, Arif Raza, Zhiliang Lin, Lu Wang, Jianqiang Li, Jie Chen
- **Link:** https://openaccess.thecvf.com/content/CVPR2026/html/Borhan_Reliable_Policy_Transfer_for_Safety-Aware_End-to-End_Driving_with_Deep_Reinforcement_CVPR_2026_paper.html ([本地PDF副本](pdf/reliable-policy-transfer.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

端到端强化学习驾驶在分布偏移下安全性和泛化性差。统一深度RL框架，围绕一个控制层可靠性接口：同一个不确定性信号同时指导关系注意力、门控策略熵、正则化迁移对齐。自我中心关系图用不确定性加权注意力编码智能体间影响；多目标可微奖励塑造安全/进度/舒适；偶然和认知不确定性捕捉调节策略熵；因果-语义迁移目标在跨域间对齐动作/注意力/不确定性统计量。

---

## 跟本repo已有内容的关系

"同一个不确定性信号同时指导多个下游模块(注意力/熵/迁移对齐)"这个思路，跟beta-kd-mllm(不确定性感知蒸馏加权)是同一类"用不确定性做统一调度信号"的设计，可以互相对照。
