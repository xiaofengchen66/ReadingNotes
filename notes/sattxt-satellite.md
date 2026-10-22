# Spectrally Distilled Representations Aligned with Instruction-Augmented LLMs for Satellite Imagery

- **Venue / year:** arXiv 2602.22613, 2026 (CVPR 2026)
- **Authors:** Minh Kha Do, Wei Xiang, Kang Han, Di Wu, Khoa Phan, Yi-Ping Phoebe Chen, Gaowen Liu, Ramana Rao Kompella
- **Link:** https://arxiv.org/abs/2602.22613 ([本地PDF副本](pdf/sattxt-satellite.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

SATtxt：推理时只用RGB输入，但保留训练时学到的光谱线索。两阶段：光谱表示蒸馏，把光谱先验从冻结的多光谱教师通过轻量投影器迁移给RGB学生；光谱接地对齐，用指令增强LLM连接蒸馏出的视觉空间和LLM嵌入空间。EuroSAT/BigEarthNet/ForestNet上零样本分类+4.2%、检索+5.9%。

---

## 跟本repo已有内容的关系

跟驾驶关系不大，收录因为是遥感VLM方向的蒸馏案例，"推理时只用部分模态、但保留训练时多模态学到的知识"这个思路，可以类比"驾驶模型推理时只有摄像头、但训练时融合过LiDAR知识"这类场景，是个可参考的设计模式。
