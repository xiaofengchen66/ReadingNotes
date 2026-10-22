# SemLT3D: Semantic-Guided Expert Distillation for Camera-only Long-Tailed 3D Object Detection

- **Venue / year:** arXiv 2604.18476, 2026 (CVPR 2026)
- **Authors:** Hao Vo, Khoa Vo, Thinh Phan, Ngo Xuan Cuong, Gianfranco Doretto, Hien Nguyen, Anh Nguyen, Ngan Le
- **Link:** https://arxiv.org/abs/2604.18476 ([本地PDF副本](pdf/semlt3d.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

纯摄像头3D目标检测的长尾问题：儿童、婴儿车、应急车辆这类罕见但安全关键的类别严重欠表示。语言引导的MoE模块：按语义亲和度把3D查询路由给专精专家，更好地区分易混淆类别、专精长尾分布；语义投影蒸馏pipeline：把3D查询和CLIP式2D语义对齐，产出更一致、更有判别力的特征。Medium类别+6.02 mAP，Few类别+2.81 mAP，推理时间只增加约9%。

---

## 跟本repo已有内容的关系

**驾驶相关，明确标注**。"语言引导的MoE路由，专精长尾罕见类别"这个设计，是MoE+驾驶里一个很具体的安全相关应用——专家按"类别语义亲和度"划分，而不是按场景类型或知识来源，是又一种专家划分维度，而且直接针对"安全关键但少见"这个驾驶场景的真实痛点。
