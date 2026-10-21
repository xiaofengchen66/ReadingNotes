# Drive-KD: Multi-Teacher Distillation for VLMs in Autonomous Driving

- **Venue / year:** arXiv 2601.21288, 2026-01-29(v1)/2026-06-04(v2)；目前未查到会议/期刊接收信息，按preprint处理
- **Authors:** Weitong Lian, Zecong Tang, Haoran Li, Tianjian Gao, Yifei Wang, Zixu Wang, Lingyi Meng, Tengju Ru, Zhejun Cui, Yichen Zhu, Hangshuo Cao, Qi Kang, Tianxing Chen, Kaixuan Wang, Yu Zhang
- **Link:** https://arxiv.org/abs/2601.21288 ([本地PDF副本](pdf/drive-kd.pdf))
- **Code:** 未查到开源仓库，暂按未开源处理
- **Input / 输出:** 驾驶场景多模态输入 → 驾驶VQA(视觉问答)与能力评测任务输出(DriveBench基准)
- **阅读进度说明：** 本笔记目前只基于论文摘要(已通过WebFetch直接核实)和用户整理的"ExpertAD/PerceptDrive/Drive-KD三篇对比表"，**还没有读到Method/Experiments具体章节、消融数字、架构图**。下方内容以此为基础，读到更多内容后再补充。

---

## TL;DR(基于摘要，待补全)

**核心问题(用户对比表口径)→** 怎样让小VLM学会大模型的驾驶能力？

**方法 →** 把自动驾驶能力拆成**感知-推理-规划三元组**(perception-reasoning-planning triad)，用**多教师蒸馏**(multi-teacher distillation)把能力从多个大模型教师迁移到一个小模型学生上，核心技术是**特定层的注意力对齐**(layer-specific attention as distillation signal)作为蒸馏信号，配合**非对称梯度投影**(asymmetric gradient projection)缓解"不同能力之间蒸馏时梯度互相打架"的问题(cross-capability gradient conflicts)。

**结果(摘要原文数据)→** 1B参数的学生模型能匹配/超过更大的基线模型，相比一个78B的基线，显存少用约42倍、吞吐量高11.4倍。主要在DriveBench(驾驶视觉问答与能力评测基准)上验证。

**关键定位：这篇不是MoE，是知识蒸馏(KD)。** 虽然经常和ExpertAD、PerceptDrive一起被提到，但Drive-KD推理时**没有路由/专家选择机制**——它是把多个教师的能力蒸馏进同一个学生模型的参数里，推理时就是这一个学生模型独立完成任务，不存在"选哪个专家"或"怎么融合多个专家"这回事。核心贡献在于解决"同时蒸馏感知、推理、规划三种不同能力时，它们的训练梯度会互相冲突"这个问题(用非对称梯度投影处理)，这是一个纯粹的多任务蒸馏优化问题，跟SAMoE-VLA/PerceptDrive/ExpertAD关心的"路由/融合"问题是两条不同的技术路线。

---

## 三篇横向对比表(用户整理，2026-10-21)

| 比较点 | ExpertAD | PerceptDrive | Drive-KD |
|---|---|---|---|
| 核心问题 | 怎样减少任务干扰和计算量？ | 怎样让感知知识有效帮助规划？ | 怎样让小VLM学会大模型的驾驶能力？ |
| 能力怎么划分 | 地图、跟踪、速度、朝向、导航等八个功能专家 | 几何、语义、动态三类感知先验 | 感知、推理、规划三类能力 |
| 核心方法 | 特征重加权＋Top-4稀疏专家＋稀疏注意力 | 先验保留＋场景相关的软路由＋世界-动作模型 | 多教师蒸馏＋特定层的注意力对齐＋梯度冲突处理 |
| 推理时如何工作 | 选择部分专家参与计算 | 对专家条件加权融合，再生成轨迹 | 由训练好的学生模型完成任务 |
| 主要验证 | 轨迹误差、碰撞率、闭环驾驶表现 | NAVSIM上的驾驶规划表现 | DriveBench上的驾驶视觉问答与能力表现 |

**跟朱子欣idea的相关性：** Drive-KD的"感知-推理-规划三元组多教师蒸馏"是目前查到的、在"全程(感知+预测+规划)"覆盖范围上最接近朱子欣说法的一篇——但它蒸馏出来的是一个**没有路由机制的单一小模型**，不是"冻结多个专家+轻量路由"这个结构，所以朱子欣说的"distill别的MoE放到我们的MoE上"这个说法里"放到我们的MoE上"这部分，Drive-KD完全没有对应——它证明了"感知-推理-规划三段能力可以被联合蒸馏"这件事本身是可行的，但没有保留专家结构、也没有路由。

---

## 待补充(读到对应章节后再写)

- [ ] "特定层注意力对齐"具体怎么做(哪些层、怎么对齐)
- [ ] 非对称梯度投影的具体机制(怎么判断梯度冲突、怎么不对称处理)
- [ ] 多教师具体是哪几个模型
- [ ] 消融实验数字(比如去掉梯度投影会掉多少分)
- [ ] 完整架构图
- [ ] DriveBench基准的具体构成

---

## 本文提到的词

| 术语 | 首次出现位置 |
|------|------------|
| Perception-reasoning-planning triad(感知-推理-规划三元组) | Abstract |
| Multi-teacher distillation(多教师蒸馏) | Abstract |
| Asymmetric gradient projection(非对称梯度投影) | Abstract |
| DriveBench | Abstract |
