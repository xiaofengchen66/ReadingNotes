# PerceptDrive: Perception Prior World-Action Modeling with Adaptive Expert Routing for End-to-End Autonomous Driving

- **Venue / year:** arXiv 2607.20175, 2026-07(目前Under Review，未确认被接收)
- **Authors:** Yushan Liu, Tianxiong Lv, Bohua Wang, Hangqi Fan, Chenxu Zhao, He Zheng, Xuchang Zhong, Yifan Xie, Congyang Zhao, Zhihao Liao, Leigang Luo, Yang Cai, Xiao-Ping Zhang, Wenbo Ding — 清华大学深圳国际研究生院(Tsinghua SIGS)主导
- **Link:** https://arxiv.org/abs/2607.20175
- **Code:** 未查到开源仓库，暂按未开源处理
- **Input / 输入:** 多视角摄像头图像(喂给冻结的Perception Provider即微调+蒸馏后的InternVL3，产出专家特征`H_t`) + 视频流(喂给冻结的自监督视频编码器，产出`F_t`) + 自车状态(ego state：车速、航向角) + 导航指令(command：左转/右转等)
- **Output / 输出:** 单条确定性规划轨迹(one trajectory)，推理时不做候选轨迹打分/重排(without test-time scoring or reranking)
- **阅读进度说明：** 本笔记已覆盖Related Work片段(三类既有做法批判(a)(b)(c)、PerceptDrive方案总结句、Latent World-Action Models综述段落)和Method里"两阶段训练+两套骨干网络"的核心结构，**还没有读到Router/Future Head/Action Head的具体机制、完整损失函数、消融数字、完整架构图**。TL;DR部分已尽量结合之前搜索验证过的摘要信息(NAVSIM结果等)，但消融表格等需要后续读到对应章节再补充，补充前不要当成已验证的完整论点使用。

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

## 两阶段训练与两套骨干网络(容易混淆，务必分清)

系统里有**两个完全不同的骨干网络**，所处阶段、分工、参数状态都不一样，读Method时最容易搞混：

### 1. Perception Provider(感知基座)——Stage 1训练出来，Stage 2冻结

- **本体：** `InternVL3`(多模态大模型)，先经过Stage 1a微调。
- **Stage 1b怎么教：** 用三个现成的教师模型——**VGGT、V-JEPA 2、Wan 2.1**——把几何(geometric)/语义(semantic)/动态(dynamic)三类知识分别蒸馏进InternVL3内部的3个"专家槽位"：`[GEO]`、`[SEM]`、`[DYN]`。这三个槽位对应的就是前面笔记里说的"expert priors(专家先验)"。
- **教完之后：** **彻底冻结**，Stage 2里参数完全不更新，只当一个只读的"感知水龙头"(Perception Provider)，负责产出专家特征`H_t`。

### 2. World-Action Model Backbone Ψ(规划主干)——Stage 2里全新训练的部分

- **本体：** 一个**全新构建的小型/轻量级网络**(通常是几层Transformer或Cross-Attention模块)，专门用于下游规划与时序交互，和Perception Provider不是同一个东西。
- **输入：** Query Banks从两个冻结源(Perception Provider的`H_t`、冻结自监督视频编码器的`F_t`)里"读出并压缩"的特征向量，再加上自车状态(ego state：车速、航向角)和导航指令(command：左转/右转)。
- **职责：** 把以上信息融合起来，交给后面的Router(路由器)、Future Head(未来头)、Action Head(轨迹头)去处理。
- **状态：** **全参数可训练**，Stage 2专门为开车规划任务优化。

### 对照表

| 属性 | Perception Provider | WAM Backbone Ψ |
|---|---|---|
| 本质实体 | 大尺寸多模态大模型(InternVL3) | 下游专用的规划融合网络 |
| 主要任务 | 看懂图像，产出高维专家特征`H_t` | 融合感知特征+自车状态+导航指令，驱动决策 |
| Stage 2状态 | **完全冻结**，不参与反向传播 | **完全可训**，实时更新权重 |
| 训练方式 | Stage 1b：三个教师模型(VGGT/V-JEPA 2/Wan 2.1)蒸馏进`[GEO]/[SEM]/[DYN]`三个专家槽位 | Stage 2：端到端训练，优化规划任务 |

**为什么容易搞混：** 两者都叫"backbone"/都跟"感知"有关，但一个是**被蒸馏、练完就封存不动**的知识容器，另一个是**专门负责把冻结知识用起来、持续在学**的调度和融合网络——对应回前面"per-branch retention anchor"和"scene-conditioned router"的分工：retention anchor保真的对象就是Perception Provider这边产出的`H_t`，scene-conditioned router则是WAM Backbone里负责按场景动态加权的那部分。

---

## 消融实验(Table A1 / Table 3 / Table A2 / A4 / A5)

### 1. 专家槽位是否真的各管各的(Table A1 + Figure 4b)

两种独立验证互相印证专家分工是真实的，不是摆设：

- **推理时token-removal诊断：** 直接在推理时挖掉某个专家token，看哪个子指标掉得最多——挖[GEO]主要伤道路几何，挖[SEM]主要伤安全，挖[DYN]主要伤进度(progress)和长时舒适性(extended comfort)。Figure 4(b)把这个画成各专家的"伤害剖面图"，每个专家被挖掉后主要伤自己那块。
- **更强的验证：重新训练(不只是推理时挖掉)：** 直接不用动态教师V-JEPA 2重新训练Perception Provider(并重新优化WAM)，EPDMS掉到88.6，掉分依然集中在EP(Ego Progress?)和EC(Extended Comfort)两个子指标——这一步排除了"推理时挖token掉分只是因为模型没见过缺输入"的可能(input-distribution artifact)，证明[DYN]专家的贡献是**因果性的**，不是巧合。

### 2. 整条pipeline是否层层有效、增益来自哪里(Table 3 + A4 + A5)

- **嵌套式(nested)消融：** 按构建顺序逐步加入future conditioning → metric-aware supervision → per-branch prior retention → metric-distilled routing(简称MR)，性能**单调递增**，全部加上效果最好，在两套评测协议下都成立。不加MR时三个专家分支保持均匀加权(uniform gating)。
- **鲁棒性：** 三个训练随机种子下，提升的顺序都保持一致(evaluation-seed std ≤0.05，Table A4)，不是随机波动。
- **EC(长时舒适性)指标的增益来源定位：** EC是"跨已发表系统里最不稳定的子指标"，但在这里——**大部分EC提升来自future conditioning这一步**(78.9→84.0)，剩下一小部分才来自dynamic-prior通路和路由机制。
- **机制增益 vs. provider强弱是否纠缠(Table A5)：** 给一个弱的provider(只做QA微调、没蒸馏)加上prior retention+metric-distilled routing，提升+1.8 EPDMS；给完整蒸馏过的provider加同样机制，提升+3.1 EPDMS——说明这两个机制不是"provider强才显灵"的伪增益，但provider越强、这两个机制能撬动的收益确实越大(协同效应)。

### 3. 具体设计选择是否都必要(Table A2)

- 把future-conditioning投影换成标准初始化、改成单阶段训练、堵住metric分支经过`st`的梯度——**三者都会让两个指标同时变差**，验证了论文自己选的初始化方式、分阶段训练、特定梯度路径都不是随意选择(对应Appendix C的"预期梯度路径")。
- **路由/融合方式的分层对比(最关键的一组对照)：**
  - **轨迹层面平均**(每个专家各自生成一条轨迹，最后把轨迹平均)——**掉分最多**(88.5 PDMS)，说明融合必须发生在"条件/特征空间"，不能等到轨迹生成完再平均。
  - **静态可学习权重** vs **完全均匀权重(uniform gating)**——**几乎没区别**(89.7 vs 89.8 PDMS)，说明"权重是可学的"这件事本身不是增益来源，不随场景动态变化就等于没用。
  - **端到端训练路由器(不用metric蒸馏信号单独监督)**——仍接近均匀水平(90.0 PDMS)，离完整版有明显差距——**真正的增益主要来自用privileged metric distillation信号专门监督路由器，不是单纯端到端梯度能学出来的**。
  - **只用导航指令(左转/右转)当路由条件**——89.0 EPDMS，比完整版低1.2分，说明光看指令不够，还得看场景本身的特征。

**这组消融的核心论证逻辑：** 路由"是不是可学习的"不重要，路由"是不是按场景动态变化的、且是不是用了metric蒸馏信号专门监督"才是真正的增益来源——这和SAMoE-VLA消融实验证明"给token级路由加场景偏置不够、必须整体换成scene级"是同一种论证模式(单纯打补丁不够，必须是机制本身对了)。

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

- [x] 两阶段训练流程 + 两套骨干网络的区别(Perception Provider vs. WAM Backbone Ψ)——已补充，见上方章节
- [x] "frozen perception foundation model"具体是什么模型——已确认是**微调+蒸馏后冻结的InternVL3**，不是一个驾驶专用MoE模型，蒸馏来源是VGGT(几何)/V-JEPA 2(动态)/Wan 2.1(语义)三个教师模型，和SAMoE-VLA/DriveMoE这类驾驶专用MoE模型没有重叠，撞车风险较低
- [ ] Retention anchor具体的损失函数形式(目前只知道机制意图，没看到具体公式)
- [ ] Router/Future Head/Action Head各自的具体网络结构和作用
- [x] 消融实验数字——已补充，见上方"消融实验"一节(Table A1/3/A2/A4/A5)
- [ ] 完整架构图(目前没有图，无法画SVG海报)
- [ ] 跟SAMoE-VLA的正面对比(如果论文里有引用或对比表格)
- [ ] 代码开源情况复核

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
| Perception Provider(感知基座，微调+蒸馏后冻结的InternVL3) | Method/Stage 1 |
| `[GEO]`/`[SEM]`/`[DYN]`专家槽位 | Method/Stage 1b |
| World-Action Model Backbone Ψ(规划主干) | Method/Stage 2 |
| Query Bank(查询库，用于从冻结特征里读出压缩表征) | Method/Stage 2 |
