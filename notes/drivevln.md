# DriveVLN: Towards Mapless Vision-and-Language Navigation in Autonomous Driving

- **Venue / year:** CVPR 2026
- **Authors:** Dongqian Guo, Haoran Wei, Wencheng Han, Runzhou Tao, Zhongying Qiu, Jianfei Yang, Jianbing Shen
- **Link:** https://openaccess.thecvf.com/content/CVPR2026/html/Guo_DriveVLN_Towards_Mapless_Vision-and-Language_Navigation_in_Autonomous_Driving_CVPR_2026_paper.html ([本地PDF副本](pdf/drivevln.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

把视觉-语言导航(VLN)扩展到自动驾驶：仅凭简洁的自然语言描述、不依赖预定义地图或路线就能引导车辆到达目的地，针对室内停车场这类无地图场景。要求模型基于多样的视觉线索(标志、地标、文字指示)和历史生成导航信息，而不是像传统VLN依赖室内环境里逐步的详细指令。基于CARLA构建了200多个真实道路扫描重建的场景仿真引擎，先监督微调再在仿真里强化学习。

---

## 跟本repo已有内容的关系

跟MoE路由关系不大，是"无地图导航"这个驾驶子任务，收录因为核心类目相关。
