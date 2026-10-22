# Continual Distillation of Teachers from Different Domains

- **Venue / year:** CVPR 2026
- **Authors:** Nicolas Michel, Maorong Wang, Jiangpeng He, Toshihiko Yamasaki
- **Link:** https://openaccess.thecvf.com/content/CVPR2026/html/Michel_Continual_Distillation_of_Teachers_from_Different_Domains_CVPR_2026_paper.html ([本地PDF副本](pdf/continual-distill-teachers.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

提出"持续蒸馏"(Continual Distillation)新范式：学生依次从一串教师模型学习，不保留对早期教师的访问。两个挑战：教师训练数据不可得、教师专长各不相同。用外部无标注数据实现"未见知识迁移"(UKT)，但序贯蒸馏会导致"未见知识遗忘"(UKF)。提出Self External Data Distillation(SE2D)：在外部数据上保留logits，稳定跨异构教师的学习。代码已开源。

---

## 跟本repo已有内容的关系

跟"持续学习中MoE路由会漂移"(on-tokens-dilemma、spectral-moe-continual)是同一类问题在"蒸馏"(而非MoE路由)场景下的版本——如果PerceptDrive/SAMoE-VLA要持续吸收新的驾驶专用MoE模型(而不是一次性蒸馏)，这篇的UKT/UKF框架提供了一个诊断"吸收新知识vs遗忘旧知识"这对矛盾的分析工具。
