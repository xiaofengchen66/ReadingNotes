# VGGDrive: Empowering Vision-Language Models with Cross-View Geometric Grounding for Autonomous Driving

- **Venue / year:** arXiv 2602.20794, 2026 (CVPR 2026, Tianjin University/Xiaomi EV)
- **Authors:** Jie Wang, Guang Li, Zhijian Huang, Chenxu Dang, Hangjun Ye, Yahong Han, Long Chen
- **Link:** https://arxiv.org/abs/2602.20794 ([本地PDF副本](pdf/vggdrive.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

现有VLM本质上缺乏跨视图3D几何建模能力，导致驾驶任务表现平平。即插即用的跨视图3D几何赋能器(CVGE)：桥接来自一个冻结视觉3D模型的跨视图3D几何特征和VLM的2D视觉特征，通过层级自适应注入机制解耦基座VLM架构、有效赋予VLM 3D特征。5个驾驶基准(跨视图风险感知/运动预测/轨迹规划等)上提升基座VLM性能。

---

## 跟本repo已有内容的关系

"桥接一个冻结视觉3D模型的几何特征和VLM的2D特征"这个思路，跟PerceptDrive"冻结Perception Provider+训练WAM Backbone"结构直接类似，这篇是另一个独立例证：冻结几何专家+轻量注入机制赋能VLM，而且明确用"即插即用"强调不需要重训基座VLM。
