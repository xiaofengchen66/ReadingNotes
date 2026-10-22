# Brewing Stronger Features: Dual-Teacher Distillation for Multispectral Earth Observation

- **Venue / year:** arXiv 2602.19863, 2026 (CVPR 2026)
- **Authors:** Filip Wolf, Blaž Rolih, Luka Čehovin Zajc
- **Link:** https://arxiv.org/abs/2602.19863 ([本地PDF副本](pdf/dual-teacher-earth-obs.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

多光谱遥感影像的双教师对比蒸馏框架，让学生的预训练目标跟现代光学视觉基础模型的对比自蒸馏范式对齐。光学和多光谱基准上语义分割+3.64pp、变化检测+1.2、分类+1.31。

---

## 跟本repo已有内容的关系

跟驾驶关系不大，收录是因为属于"多教师蒸馏"这个方法类别，可以跟Drive-KD的多教师蒸馏(感知-推理-规划三个教师)做方法论对比：这篇是两个教师各管一种模态/范式，Drive-KD是多个教师各管一种能力。
