# YOLO-Master: MOE-Accelerated with Specialized Transformers for Enhanced Real-time Detection

- **Venue / year:** arXiv 2512.23273, 2025 (CVPR 2026)
- **Authors:** (腾讯优图实验室 + 新加坡管理大学)
- **Link:** https://arxiv.org/abs/2512.23273 ([本地PDF副本](pdf/yolo-master.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 MoE批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

YOLO类实时目标检测框架，引入实例条件的自适应计算：Efficient Sparse MoE(ES-MoE)模块根据每个输入的场景复杂度动态分配计算资源。轻量动态路由网络通过多样性增强目标在训练时引导专家专精(鼓励专家间互补)，自适应只激活最相关的专家，在降低计算开销的同时提升检测性能。MS COCO上42.4% AP、1.62ms延迟，比YOLOv13-N高0.8% mAP、推理快17.8%。

---

## 跟本repo已有内容的关系

"根据场景复杂度动态分配计算资源"这个思路，是MoE路由跟实时性/计算效率结合最直接的例子之一——对比ExpertAD"碰撞率降低但没强调省计算"，YOLO-Master把"省计算"作为核心卖点，如果要做"按需请求信息、减少计算量"这个方向，这篇的实例条件自适应计算设计值得细读。
