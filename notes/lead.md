# LEAD: Minimizing Learner-Expert Asymmetry in End-to-End Driving

- **Venue / year:** arXiv 2512.20563, 2025 (CVPR 2026, NVIDIA)
- **Authors:** Long Nguyen, Micha Fauth, Bernhard Jaeger, Daniel Dauner, Maximilian Igl, Andreas Geiger, Kashyap Chitta
- **Link:** https://arxiv.org/abs/2512.20563 ([本地PDF副本](pdf/lead.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

特权专家示范和基于传感器的学生观测之间的错位会限制模仿学习的有效性：专家可见度远高于学生(比如能忽略遮挡)、不确定性远低于学生(比如知道其他车辆的动作)，导致专家行为很难可靠模仿；导航意图(该走哪条路线)在测试时学生模型只能靠单一目标点欠规范地指定。TransFuser v6(TFv6)学生策略在CARLA闭环基准上全面SOTA(Bench2Drive 95 DS)，把感知监督整合进共享sim-to-real pipeline，在NAVSIM和Waymo基准上也持续提升。代码数据模型已开源。

---

## 跟本repo已有内容的关系

"学生和专家之间的可见度/不确定性不对称"这个问题诊断，对PerceptDrive/SAMoE-VLA这类"冻结教师/先验+训练学生"的设计有警示意义——如果教师(专家/冻结先验)能看到的信息比学生推理时能看到的多，蒸馏/模仿可能从根上有偏差，这篇是专门研究这个问题的参考文献。
