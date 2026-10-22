# Generalizable Knowledge Distillation from Vision Foundation Models for Semantic Segmentation

- **Venue / year:** arXiv 2603.02554, 2026 (CVPR 2026)
- **Authors:** Chonghua Lv, Dong Zhao, Shuang Wang, Dou Quan, Ning Huyan, Nicu Sebe, Zhun Zhong
- **Link:** https://arxiv.org/abs/2603.02554 ([本地PDF副本](pdf/gkd-semantic-seg.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

语义分割任务，解决"从视觉基础模型蒸馏时，域外泛化能力会被常规KD损害"的问题。GKD两阶段框架：第一阶段学生通过选择性特征蒸馏获得领域无关表示；第二阶段冻结这些表示做任务适配；另外提出基于query的软蒸馏机制，学生特征当query去教师表示里选择性检索可迁移的空间知识。5个域泛化基准上持续超过现有KD方法，基础模型到基础模型提升1.9%，基础模型到本地模型提升10.6%。

---

## 跟本repo已有内容的关系

"先学领域无关表示、再冻结做任务适配"这个两阶段范式，跟PerceptDrive的"Stage 1蒸馏感知先验(冻结)→Stage 2训练任务专属的WAM Backbone"结构几乎一样，是同一个设计模式在语义分割上的独立验证，而且专门讨论了"怎么蒸馏才不破坏泛化能力"这个PerceptDrive笔记里还没展开的问题。
