# Beyond Weak Supervision: MLLMs-Guided Graded Knowledge Distillation for Unsupervised Camouflaged Object Detection

- **Venue / year:** CVPR 2026
- **Authors:** Huafeng Chen, Chenguang Zhu, Yueming Lyu, Caifeng Shan
- **Link:** https://openaccess.thecvf.com/content/CVPR2026/html/Chen_Beyond_Weak_Supervision_MLLMs-Guided_Graded_Knowledge_Distillation_for_Unsupervised_Camouflaged_CVPR_2026_paper.html ([本地PDF副本](pdf/beyond-weak-supervision-cod.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

无监督伪装目标检测。融合MLLM和SAM构建强教师模型；提出伪装感知思维链(CA-CoT)引导MLLM；设计分级掩码评估器(GME)缓解级联误差，再做分级知识蒸馏(GKD)，按伪标签质量自适应调整蒸馏强度。大幅超过现有无监督方法，零样本设置下表现良好。

---

## 跟本repo已有内容的关系

跟驾驶关系不大，收录是因为属于"用MLLM构造教师信号、再分级蒸馏"这个方法论，"按伪标签质量自适应调整蒸馏强度"的思路可以类比PerceptDrive"按场景动态调整路由权重"——都是"不同情况区别对待，而不是一刀切"的设计哲学。
