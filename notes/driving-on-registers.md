# Driving on Registers

- **Venue / year:** arXiv 2601.05083, 2026 (CVPR 2026)
- **Authors:** Ellington Kirby, Alexandre Boulch, Yihong Xu, Yuan Yin, Gilles Puy, Éloi Zablocki, Andrei Bursuc, Spyros Gidaris, Renaud Marlet, Florent Bartoccioni, Anh-Quan Cao, Nermin Samet, Tuan-Hung VU, Matthieu Cord
- **Link:** https://arxiv.org/abs/2601.05083 ([本地PDF副本](pdf/driving-on-registers.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Driving models/planning批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

DrivoR：简单高效的端到端驾驶Transformer架构，基于预训练ViT，引入相机感知的寄存器token(register tokens)把多摄像头特征压缩成紧凑的场景表示，大幅减少下游计算量且不牺牲精度。寄存器token驱动两个轻量Transformer解码器：先生成候选轨迹，再打分(打分解码器学习模仿一个oracle，预测可解释的子分数，代表安全/舒适/效率等方面)。NAVSIM-v1/v2和光真实感闭环HUGSIM基准上超过或匹配强基线，同时吞吐量提升3倍以上，GFLOPS和显存比ViT-L基线减少3倍。

---

## 跟本repo已有内容的关系

"相机感知寄存器token压缩多摄像头特征"这个思路，直接回应"怎么又快又省计算"——跟PerceptDrive的Query Bank(从冻结特征里读出压缩表征)是类似的"用少量token概括大量输入"的设计模式，而且这篇给出了具体3倍吞吐量提升的数字，值得细读其压缩机制的具体实现。
