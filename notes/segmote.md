# SegMoTE: Token-Level Mixture of Experts for Medical Image Segmentation

- **Venue / year:** arXiv 2602.19213, 2026 (CVPR 2026 Oral)
- **Authors:** Yujie Lu, Jingwen Li, Sibo Ju, Yanzhou Su, He Yao, Yisong Liu, Min Zhu, Junlong Cheng (四川大学/新疆大学/福州大学/阿里达摩院)
- **Link:** https://arxiv.org/abs/2602.19213 ([本地PDF副本](pdf/segmote.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

交互式医学图像分割，基于MoE范式，token级专家选择，针对每种成像模态动态激活最合适的专家token，实现模态特定表示和自适应处理。冻结SAM编码器，只训练轻量的MoTE，保留SAM原有能力的同时扩展到新数据集；渐进式prompt token化机制支持全自动分割，大幅降低标注依赖。在MedSeg-HQ(不到现有大规模数据集1%的精选数据集)上训练，达到SOTA。

---

## 跟本repo已有内容的关系

"冻结大模型编码器+只训练轻量MoE路由"这个思路，跟PerceptDrive"冻结Perception Provider+只训练WAM Backbone"的结构几乎一样，是同一个设计模式在医学分割领域的应用——可以作为"冻结专家+轻量路由"这个大方向下又一个独立验证该思路有效的例子。
