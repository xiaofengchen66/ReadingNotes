# Teacher-Guided Routing for Sparse Vision Mixture-of-Experts

- **Venue / year:** arXiv 2604.21330, 2026 (CVPR 2026)
- **Authors:** Masahiro Kada, Ryota Yoshihashi, Satoshi Ikehata, Rei Kawakami, Ikuro Sato
- **Link:** https://arxiv.org/abs/2604.21330 ([本地PDF副本](pdf/tgr-moe.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

用预训练的dense教师模型的监督信号稳定MoE router的学习：从教师模型的中间表示构造"教师router"，把教师router的路由输出当成伪监督信号训练学生router。ImageNet-1K/CIFAR-100上，TGR持续提升准确率和路由一致性，即使在高度稀疏配置下训练依然稳定。

---

## 跟本repo已有内容的关系

这篇名字跟Drive-KD看起来像(都有teacher-guided/distillation)，但本质不同：Drive-KD是蒸馏知识进学生模型(没有路由结构)，这篇是蒸馏"路由决策"本身(学生依然是MoE结构，只是router的学习被teacher router引导)——如果要让PerceptDrive那种场景条件路由器训练得更稳定，这篇的"教师路由监督"思路可以直接参考。
