# EventDrive: Event Cameras for Vision-Language Driving Intelligence

- **Venue / year:** arXiv 2606.18242, 2026 (CVPR 2026, NUS/HKUST)
- **Authors:** Dongyue Lu, Rong Li, Ao Liang, Lingdong Kong, Wei Yin, Lai Xing Ng, Benoit R. Cottereau, Camille Simon Chane, Wei Tsang Ooi
- **Link:** https://arxiv.org/abs/2606.18242 ([本地PDF副本](pdf/eventdrive.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

事件相机通过异步亮度变化感知世界，微秒级延迟、高动态范围，在模糊/眩光/快速运动场景下比基于帧的感知更可靠。大规模基准+模型套件EventDrive，统一事件流/RGB帧/语言监督，覆盖感知/理解/预测/规划四个核心维度，47.1万事件-帧-语言样本、17个子任务。**EventDrive-VLM引入多尺度事件金字塔+模态路由的混合专家(modality-routing mixture-of-experts)**，自适应编码融合异步和基于帧的信息。

---

## 跟本repo已有内容的关系

**这篇有真正的MoE组件，而且是驾驶专用**——"模态路由MoE"按输入模态(事件流 vs 帧)而非场景类型划分路由，是又一种专家划分维度(按传感器模态)，跟SAMoE-VLA(按场景)、ExpertAD(按功能子任务)、SemLT3D(按类别语义)、MoEActok(按技能)形成一个完整的"专家划分维度"案例库，建议细读其MoE路由具体机制。
