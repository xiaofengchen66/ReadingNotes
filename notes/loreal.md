# LOREAL: Mitigating Low-Resolution Challenges in Vision-Language Models with Attribute-driven Prompt Self-Distillation

- **Venue / year:** CVPR 2026
- **Authors:** Xucong Wang, Pengkun Wang, Zhe Zhao, Liheng Yu, Rui Mao, Yang Wang
- **Link:** https://openaccess.thecvf.com/content/CVPR2026/html/Wang_LOREAL_Mitigating_Low-Resolution_Challenges_in_Vision-Language_Models_with_Attribute-driven_Prompt_CVPR_2026_paper.html ([本地PDF副本](pdf/loreal.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

VLM的prompt learning部署后面临分辨率-存储开销权衡问题。高效prompt自蒸馏框架：双学生架构，两个不同分辨率输入的学生模型协同互学；用LLM查询分辨率不变的属性语义，给学生prompt提供上下文；跨模态meta-net生成属性语义；低层蒸馏(LLD)+高层蒸馏(HLD)促进跨分辨率表示学习。

---

## 跟本repo已有内容的关系

跟驾驶关系不大，是VLM部署效率优化的一个案例，"双学生协同互学"这个思路可以作为"怎么设计自蒸馏机制"的参考。
