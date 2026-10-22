# Open-Ended Instruction Realization with LLM-Enabled Multi-Planner Scheduling in Autonomous Vehicles

- **Venue / year:** arXiv 2604.08031, 2026 (CVPR 2026)
- **Authors:** Jiawei Liu, Xun Gong, Fen Fang, Muli Yang, Bohao Qu, Yunfeng Hu, Hong Chen, Xulei Yang, Qing Guo
- **Link:** https://arxiv.org/abs/2604.08031 ([本地PDF副本](pdf/open-ended-instruction-realization.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

针对乘客机动请求的人机交互(HMI)需求。LLM解读指令，生成可执行脚本，基于实时反馈调度多个基于MPC的运动规划器，再把规划轨迹转换成控制信号。调度中心式设计把语义推理和车辆控制在不同时间尺度解耦，建立从高层指令到底层动作的透明、可追溯决策链。显著提升任务完成率、降低LLM查询成本、安全性跟专用自动驾驶方法相当、能很好处理LLM推理延迟。

---

## 跟本repo已有内容的关系

"调度中心式设计，解耦语义推理(慢)和车辆控制(快)"这个思路，直接回应"怎么让VLM推理不拖慢实时控制"这个效率问题，跟ColaVLA"只用两次VLM前向"是同一类问题的不同解法。
