# PerceptDrive: Perception Prior World-Action Modeling with Adaptive Expert Routing for End-to-End Autonomous Driving

- **Venue / year:** arXiv 2607.20175, 2026-07(目前Under Review，未确认被接收)
- **Authors:** Yushan Liu, Tianxiong Lv, Bohua Wang, Hangqi Fan, Chenxu Zhao, He Zheng, Xuchang Zhong, Yifan Xie, Congyang Zhao, Zhihao Liao, Leigang Luo, Yang Cai, Xiao-Ping Zhang, Wenbo Ding — 清华大学深圳国际研究生院(Tsinghua SIGS)主导
- **Link:** https://arxiv.org/abs/2607.20175
- **Code:** 未查到开源仓库，暂按未开源处理
- **Input / 输入:** 多视角摄像头图像 → BEV/感知特征；冻结的感知基础模型(frozen perception foundation models)输出 + 冻结的自监督视频编码器(frozen self-supervised video encoder)输出，作为"专家先验"(expert priors)
- **Output / 输出:** 单条确定性规划轨迹(one trajectory)，推理时不做候选轨迹打分/重排(without test-time scoring or reranking)
- **阅读进度说明：** 本笔记目前只覆盖了用户读到的Related Work片段(对三类既有做法的批判(a)(b)(c)、PerceptDrive方案总结句、Latent World-Action Models综述段落)，**还没有读到Method/Experiments具体细节、消融数字、完整架构图**。TL;DR和术语解析部分已尽量结合之前搜索验证过的摘要信息(NAVSIM结果等)，但方法细节、公式、消融表格等需要后续读到对应章节再补充，补充前不要当成已验证的完整论点使用。

---

## TL;DR(基于目前读到的部分，后续需要补全)

**问题 →** 用"世界模型"(World Model，预测未来隐空间/未来帧)辅助自动驾驶规划，这条路线这几年做法很多(预测未来隐特征、视频生成式未来想象、未来-动作交替预测等)，但这些工作全都只关心"怎么预测未来"，没人系统处理"怎么把来自不同专家(感知基础模型、视频编码器等)的现成冻结先验，在压缩进规划器的过程中不丢失、还能按场景动态分配权重"这两个问题。

**Gap →** 现有把专家先验接入VLM/规划器的三种做法都有明确缺陷(见下文(a)(b)(c))，而且"预测未来"这件事本身并不能替代解决这两个问题——预测未来只是提供时序上下文，不等于解决了先验保真和动态融合。

**Novelty →** PerceptDrive = 冻结感知先验(frozen perception foundation model + frozen self-supervised video encoder) + per-branch retention anchor(逐分支先验保真约束) + scene-conditioned router(场景条件动态路由) + 以自预测的、不含动作信息的未来隐特征(action-free future latent)作为条件，直接输出一条轨迹，不做测试时打分重排。

**核心洞察 →** "预测未来"和"先验怎么保留/怎么按场景加权"是两个独立的问题，现有工作把前者做好了，但完全没处理后者——这是PerceptDrive切入的空档。

**结果(来自此前已核实的摘要信息，非本次用户贴的原文)：** NAVSIM基准上PDMS(v1) 90.4、EPDMS(v2) 90.2。

---

## 问题诊断：三种现有做法的缺陷(a)(b)(c)

论文在Related Work里用三步递进的方式批判了"怎么把专家先验接入VLM/规划器"这件事现有的三种做法，PerceptDrive是对这三个缺陷的针对性回应：

**(a) Direct VLM conditioning(直接对VLM做条件输入)**
> "knowledge stays implicit in a single narrow interface, where task-relevant cues are attenuated."

把所有先验/提示一股脑塞进VLM同一个狭窄的跨模态接口(比如简单的线性投影层)，知识只能隐式存在、全靠网络自己意会，而这个接口带宽有限，导致真正跟任务相关的关键线索在挤压式压缩中被稀释、冲淡。

**(b) Expert priors inside the VLM(专家先验混在VLM参数内部，没有显式拆分)**
> "without explicit separation, no objective specifies which prior survives compression."

如果不在结构上把不同专家知识显式拆开(比如没有MoE门控、没有独立适配器)，而是混在同一套参数里，那么模型做通用的压缩/投影/多任务微调时，优化目标(比如常规的交叉熵/自回归损失)只会去找全局最优，没有专门机制保证"哪个先验必须被保留"——先验会在无人看管的情况下被随意压缩掉。

**(c) Separated expert representations under static fusion(专家表征拆开了，但融合权重是静态的)**
> "scene-invariant weights cannot adapt each prior's contribution to the current scene."

即便把不同专家的特征显式拆开独立建模了(这一步比(b)进了一步)，如果融合这些特征时用的是一套训练完就固定住的权重(scene-invariant，不随输入场景变化)，那遇到不同场景(晴天vs大雾、空旷路面vs拥挤路口)时，各专家该被倚重多少完全没法动态调整——该多听几何专家的时候听不到，该多听语义专家的时候也调不过去。

**三者递进关系：** (a)→(b)→(c) 是缺陷被逐步修正但没修干净的过程——(b)比(a)好在哪里都没拆分，(c)比(b)好在拆分了，但(c)依然没解决"怎么按场景动态分配权重"。PerceptDrive要同时解决(b)的"压缩中保真"问题和(c)的"按场景动态加权"问题。

---

## PerceptDrive的方案：三个机制对应三个缺陷

> "(Ours): per-branch retention anchors each compressed readout to its prior, and a scene-conditioned router reweights the expert conditions per scene, yielding one trajectory under self-predicted future conditioning without test-time scoring or reranking."

| PerceptDrive机制 | 对应解决的缺陷 | 做法 |
|---|---|---|
| **Per-branch retention anchor**(逐分支保留锚点) | 缺陷(b)：压缩中先验丢失，没人管 | 给每个专家分支单独加一个保真约束，把压缩后的输出特征(compressed readout)强行"锚定"回它对应的原始专家先验(prior)上——实现上通常是一个辅助损失(比如让压缩特征能重建/对齐回原始先验特征)，防止压缩时随驾驶损失的梯度把专家细节冲没 |
| **Scene-conditioned router**(场景条件路由器) | 缺陷(c)：静态融合权重无法按场景调整 | 一个轻量路由模块，拿当前场景特征作为条件，实时算出每个专家先验该占多少权重，替代写死的固定融合比例 |
| **Self-predicted future conditioning**(自预测未来条件) | 补充时序上下文(不是直接回应abc，是额外加的能力) | 先在隐空间自监督地预测一个**不含动作信息**的未来状态(action-free future latent)，拿这个预测结果当条件去生成轨迹，但不是靠它来解决先验保真/动态加权问题——论文原文明确说"预测未来本身不能替代解决这两个问题"，这是独立叠加的一层 |
| **直接输出一条轨迹，无测试时打分重排** | 推理效率 | 很多端到端方法要在推理时生成一堆候选轨迹再跑评分器挑一条(scoring/reranking)，耗时；PerceptDrive靠高质量的先验保真+动态路由，一次前向直接出最终轨迹 |

---

## 关键术语解析

**Retention anchor(保留锚点)**
不是标准固定术语，是"retention(保留/防遗忘)"+"anchor(锚定)"组合出的机制描述：给压缩后的特征加一个显式约束(比如余弦相似度/重建损失)，强迫它必须能对应/还原出压缩前的原始专家先验，防止专家知识在只看最终驾驶损失的训练压力下被悄悄抹掉。类比：给每个专家分支配一个"质检员"，拿精简摘要去对照专家原话校验，校验不过就要罚(loss惩罚)。

**Static fusion(静态融合) vs 动态融合**
静态融合：`F = w1·E1 + w2·E2 + ...`，权重w在训练完之后就固定了，不随输入场景变化，相当于"不管讨论什么议题，各专家意见永远占固定比例"。
动态融合(PerceptDrive用的)：`[w1(S), w2(S), ...] = Softmax(Router(S))`，权重是拿当前场景S现算出来的，场景变、权重跟着变。

**Scene-conditioned router(场景条件路由器)**
一个以当前场景特征为输入、输出各专家融合权重的小模块，本质上是把前面SAMoE-VLA笔记里讨论过的"场景级路由"思路，应用在"融合多个不同来源的冻结专家先验"这件事上，而不是"融合FFN参数"——**这是和SAMoE-VLA的关键差异点**：SAMoE-VLA路由的是同一个模型内部训练出来的多个FFN参数，PerceptDrive路由/融合的是多个外部冻结专家模型的输出先验。

---

## 相关工作脉络：Latent World-Action Models

论文Related Work把"隐空间世界-动作模型"这条研究线梳理成4类范式，然后指出共性盲区：

1. **表征自监督路线**：预测未来隐特征，或视觉token+动作token联合建模，作为规划表征的自监督信号。
2. **场景演进+轨迹生成耦合路线**：未来图像预测、视频生成隐空间、动作条件想象(imagination)、未来-动作交替预测、视觉-动作双向交叉条件。
3. **几何/生成监督直接引导路线**：额外加3D几何或生成式监督，把预测出的未来隐状态直接当规划器的输入条件。
4. **前沿变体**：自车-环境协同演变+前瞻规划；驾驶风格隐动态接入下游规划行为。

**共性盲区(论文原话)：**
> "Future prediction supplies complementary temporal context for continuous planning, but does not by itself specify how heterogeneous frozen priors should be retained after compression or weighted across scenes."

预测未来确实能提供有用的时序上下文，但**不能自动解决**"异构冻结先验怎么在压缩中保真"和"怎么按场景动态加权"这两个问题——这两个问题跟"预测未来"是两条独立的轴，前人工作只做了一条轴。PerceptDrive = 继续做"预测未来"这条轴(自预测action-free未来隐特征) + 同时补上另外两条轴(per-branch retention + scene-conditioned routing)。

---

## 待补充(读到对应章节后再写)

- [ ] 完整Method章节：retention anchor具体的损失函数形式、router的具体网络结构
- [ ] 消融实验：去掉retention anchor / 去掉scene-conditioned router分别掉多少点
- [ ] 完整架构图(目前没有图，无法画SVG海报)
- [ ] 跟SAMoE-VLA的正面对比(如果论文里有引用或对比表格)
- [ ] 代码开源情况复核
- [ ] 跟10-15号已经记录在`samoe-vla.md`"相关工作排查"里的撞车风险点对照：PerceptDrive冻结的是通用感知基础模型，不是别的MoE驾驶模型本身——这一点需要读Method部分确认"frozen perception foundation model"具体指哪个模型(是通用视觉基础模型，还是已经在驾驶数据上训练过的专用模型)

---

## 本文提到的词(持续增补)

| 术语 | 首次出现位置 |
|------|------------|
| Expert priors(专家先验) | Related Work |
| Direct VLM conditioning | Related Work |
| Retention anchor / Per-branch retention(保留锚点) | Related Work |
| Static fusion(静态融合) | Related Work |
| Scene-conditioned router(场景条件路由器) | Related Work |
| Action-free future latent(不含动作信息的未来隐特征) | Related Work |
| Latent world-action modeling(隐空间世界-动作建模) | Related Work |
