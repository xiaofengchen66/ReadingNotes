# WAM-Flow: Parallel Coarse-to-Fine Motion Planning via Discrete Flow Matching for Autonomous Driving

- **Venue / year:** arXiv 2512.06112, 2025 (CVPR 2026, Fudan)
- **Authors:** Yifang Xu, Jiahao Cui, Zhihao Zhu, Hanlin Shang, Shan Luan, Mingwang Xu, Feipeng Cai, Neng Zhang, Yaoyi Li, Jia Cai, Siyu Zhu
- **Link:** https://arxiv.org/abs/2512.06112 ([本地PDF副本](pdf/wam-flow.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

把自车轨迹规划建模成在结构化token空间上的离散流匹配。跟自回归解码器不同，WAM-Flow做完全并行、双向去噪，实现由粗到精的精修，带可调的计算-精度权衡。度量对齐的数值tokenizer通过三元组边际学习保留标量几何+几何感知流目标+仿真器引导的GRPO对齐(整合安全/自车进度/舒适奖励，同时保留并行生成)。多阶段适配把预训练自回归主干(Janus-1.5B)从因果解码转换成非因果流模型。NAVSIM v1上1步推理89.1 PDMS，5步推理90.3 PDMS，超过自回归和扩散式VLA基线。代码已开源。

---

## 跟本repo已有内容的关系

**这篇名字带"WAM"，跟PerceptDrive的World-Action Model(WAM) Backbone术语撞名**，但完全是不同的论文/机制(这篇是离散流匹配规划器，不涉及冻结感知先验或场景路由)——又一个需要注意命名混淆的案例(跟Percept-WAM类似)，整理材料时要分清WAM-Flow(复旦)、Percept-WAM(银河通用+复旦)、PerceptDrive(清华深研院)三者是完全不同的三篇论文，只是都用了"WAM/World-Action Model"相关命名。
