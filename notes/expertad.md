# ExpertAD: Enhancing Autonomous Driving Systems with Mixture of Experts

- **Venue / year:** arXiv 2511.11740, 2025-11-13投稿；**已确认被AAAI 2026(第40届)接收**
- **Authors:** Haowen Jiang, Xinyu Huang, You Lu, Dingji Wang, Yuheng Cao, Chaofeng Sha, Bihuan Chen, Keyu Chen, Xin Peng
- **Link:** https://arxiv.org/abs/2511.11740 ([本地PDF副本](pdf/expertad.pdf))
- **Code:** 未查到开源仓库，暂按未开源处理
- **Input / 输出:** 多功能驾驶输入(地图、跟踪、速度、朝向、导航等) → 轨迹规划
- **阅读进度说明：** 本笔记目前只基于论文摘要(已通过WebFetch直接核实)和用户整理的"ExpertAD/PerceptDrive/Drive-KD三篇对比表"，**还没有读到Method/Experiments具体章节、消融数字、架构图**。下方内容以此为基础，读到更多内容后再补充。

---

## TL;DR(基于摘要，待补全)

**核心问题(用户对比表口径)→** 怎样减少任务干扰(task interference)和计算量？

**方法 →** 引入**Perception Adapter**放大任务关键特征(task-critical features)，配合**Mixture of Sparse Experts**减少任务间干扰；能力划分方式是按**8个功能专家**切分(地图、跟踪、速度、朝向、导航等)，推理时用**特征重加权+Top-4稀疏专家+稀疏注意力**，只选部分专家参与计算(不是像SAMoE-VLA那样融合参数，是传统的稀疏选择式MoE)。

**结果(摘要原文数据)→** 相比现有方法，平均碰撞率降低最多20%，推理延迟降低25%，在罕见场景(rare scenarios)和未见过的环境(unseen environments)上表现较强。主要验证指标是轨迹误差、碰撞率、闭环驾驶表现。

**跟本repo其他MoE驾驶论文的关键差异：**
- 跟SAMoE-VLA比：SAMoE-VLA是"参数级融合"(融合FFN权重)，ExpertAD是"选择式"(Top-4稀疏选择，类似经典MoE/SparseMoE路线)——两者路由范式不同，可以直接对照SAMoE-VLA笔记里"四象限消融图"里的SparseMoE一类。
- 跟PerceptDrive比：PerceptDrive的专家是"感知先验"(几何/语义/动态，来自外部冻结模型蒸馏)，ExpertAD的专家是"驾驶子任务"(地图/跟踪/速度/朝向/导航，8个功能专家)——专家的"定义方式"完全不同，一个按"知识来源"划分，一个按"任务功能"划分。

---

## 三篇横向对比表(用户整理，2026-10-21)

| 比较点 | ExpertAD | PerceptDrive | Drive-KD |
|---|---|---|---|
| 核心问题 | 怎样减少任务干扰和计算量？ | 怎样让感知知识有效帮助规划？ | 怎样让小VLM学会大模型的驾驶能力？ |
| 能力怎么划分 | 地图、跟踪、速度、朝向、导航等八个功能专家 | 几何、语义、动态三类感知先验 | 感知、推理、规划三类能力 |
| 核心方法 | 特征重加权＋Top-4稀疏专家＋稀疏注意力 | 先验保留＋场景相关的软路由＋世界-动作模型 | 多教师蒸馏＋特定层的注意力对齐＋梯度冲突处理 |
| 推理时如何工作 | 选择部分专家参与计算 | 对专家条件加权融合，再生成轨迹 | 由训练好的学生模型完成任务 |
| 主要验证 | 轨迹误差、碰撞率、闭环驾驶表现 | NAVSIM上的驾驶规划表现 | DriveBench上的驾驶视觉问答与能力表现 |

**这张表最值得记住的一点：** 三篇论文名字里都有"MoE/Expert/KD"，但解决的根本不是同一类问题——ExpertAD是经典的"稀疏选专家、省计算"思路，PerceptDrive是"怎么把外部先验知识保真地用起来"思路，Drive-KD干脆不是MoE、是多教师蒸馏(没有路由/专家选择机制，是把多个教师的能力蒸馏进一个学生模型)。三者容易被一起提及，但彼此替代不了。

---

## 待补充(读到对应章节后再写)

- [ ] Perception Adapter具体结构和怎么"放大任务关键特征"
- [ ] Mixture of Sparse Experts的路由机制细节(Top-4怎么选、稀疏注意力具体怎么做)
- [ ] 8个功能专家具体怎么定义/怎么训练出来的
- [ ] 消融实验数字
- [ ] 完整架构图
- [ ] 跟SAMoE-VLA"参数融合vs选择式"这两种路由范式的直接优劣对比(如果论文里有讨论)

---

## 本文提到的词

| 术语 | 首次出现位置 |
|------|------------|
| Perception Adapter | Abstract |
| Mixture of Sparse Experts | Abstract |
| Task interference(任务干扰) | Abstract |
