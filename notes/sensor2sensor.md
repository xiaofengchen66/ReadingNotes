# Sensor2Sensor: Cross-Embodiment Sensor Conversion for Autonomous Driving

- **Venue / year:** arXiv 2605.22809, 2026 (CVPR 2026, Waymo/JHU/Google DeepMind)
- **Authors:** Jiahao Wang, Bo Sun, Yijing Bai, Vincent Casser, Songyou Peng, Zehao Zhu, Meng-Li Shih, Xander Masotto, Shih-Yang Su, Kanaad Parvate, Tiancheng Ge, Linn Bieske, Dragomir Anguelov, Mingxing Tan, Chiyu Max Jiang
- **Link:** https://arxiv.org/abs/2605.22809 ([本地PDF副本](pdf/sensor2sensor.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

把来自不同来源(行车记录仪、网络驾驶视频、手机、其他ADS/ADAS/车辆平台)的野外单目视频，转换成针对目标车辆配置的高保真多模态多传感器AV日志。车队采集的专有数据虽然高保真但规模、传感器配置多样性、地理和长尾行为覆盖都有限，野外数据规模大、多样但缺少配对训练数据。通过4D高斯泼溅(4DGS)重建和新视角渲染把真实AV日志转换成行车记录仪风格视频，再用扩散架构做生成式转换。

---

## 跟本repo已有内容的关系

跟MoE路由关系不大，是"跨传感器平台数据转换"这条技术路线，收录因为核心类目相关。
