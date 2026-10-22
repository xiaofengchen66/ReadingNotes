# SigLino: Efficient Multi-Teacher Distillation for Agglomerative Vision Foundation Models

- **Venue / year:** arXiv 2512.20157, 2025 (CVPR 2026)
- **Authors:** Sofian Chaybouti, Sanath Narayan, Yasser Dahou, Phúc H. Lê Khắc, Ankit Singh, Ngoc Huynh, Wamiq Reyaz Para, Hilde Kuehne, Hakim Hacid
- **Link:** https://arxiv.org/abs/2512.20157 ([本地PDF副本](pdf/siglino.pdf))
- **Code:** 未核实，暂不标注
- **阅读进度说明：** 本笔记是CVPR 2026 Distillation批量摘要级整理(2026-10-21)，基于WebSearch检索到的摘要/简介信息，**不是精读笔记**。细读时请在此基础上补充方法细节、公式、消融数字、架构图。

---

## 摘要级概述

同时从SigLIP2和DINOv3两个教师蒸馏知识，**蒸馏目标既包括Dense模型也包括MoE模型**。非对称关系知识蒸馏损失(保留每个教师的几何性质)+token平衡批处理(把不同分辨率图像打包成统一token预算序列，稳定表示学习)+分层聚类采样(比随机采样显著提升样本效率)。构建了2亿图像的OpenLVD200M语料库。

---

## 跟本repo已有内容的关系

**这篇是这批里跟"全程MoE+蒸馏别的模型"这个方向机制最接近的一篇**——它明确把多教师蒸馏的目标设成MoE模型(不只是Dense学生)，直接回应了PerceptDrive"蒸馏对象是通用基础模型、不是MoE模型"这个局限中的另一半：这篇展示了"多教师蒸馏进MoE学生"本身是可行的，只是教师(SigLIP2/DINOv3)依然不是驾驶专用MoE——如果要蒸馏DriveMoE/SAMoE-VLA这类驾驶专用MoE当教师，这篇的蒸馏损失设计(非对称关系KD)值得细读。
