# WAM-Diff: A Masked Diffusion VLA Framework with MoE and Online Reinforcement Learning for Autonomous Driving

- **Venue / year:** arXiv 2025(2512.11872v1,2025-12-06)
- **Authors:** Mingwang Xu, Jiahao Cui, Feipeng Cai, Hanlin Shang, Zhihao Zhu, Shan Luan, Yifang Xu, Neng Zhang, Yaoyi Li, Jia Cai, Siyu Zhu — 复旦大学 + Yinwang Intelligent Technology(问界背后的引望智能)
- **Link:** https://arxiv.org/abs/2512.11872
- **来源说明:** 从 WA-JEPA 对比表格发现,arXiv API 核实后下载全文,用 `pdftotext` 提取逐句核对。
- **paper_matrix.csv id:** 无
- **Code:** https://github.com/fudan-generative-vision/WAM-Diff(**部分开放**——已核实推理/训练代码和预训练权重真实可用〔2025-02-01 起放到 HuggingFace〕,但 README 自己的路线图标注 NAVSIM 评测代码和 RL〔GSPO〕实现仍是 "TBD",300 stars、51 forks)
- **Input / 输入:** **只有单前视相机图像**(1920×1080,切成 15 个 384×384 patch + 缩略全图共 16 个 patch,经 SigLIP-2 编码成 2185 个视觉 token)+ 文本(导航指令、ego state);**没有环视相机,也没有历史帧**
- **Output / 输出:** 离散化的未来轨迹 token 序列(数值 waypoint token 和语义语言 token 混合)+ 可选的 driving VQA 回答

## TL;DR 浓缩版

**问题** → VLA 做自动驾驶,主流是自回归 LLM 式(逐 token 生成,只能从左到右)或连续扩散 policy;离散 mask diffusion 这种"双向上下文、可并行解码"的生成范式,虽然在语言/多模态领域已经证明有潜力,但用在自动驾驶轨迹生成上基本没人系统研究过。

**Gap** → 已有把离散扩散用于自动驾驶的初步尝试,效果都不如 SOTA 自回归方法,存在明显性能差距。

**Novelty** → 系统性地把 mask diffusion 适配到 driving VLA:①支持灵活的(非因果)解码顺序,可以按场景类型选择因果/反因果/随机解码;②用 **LoRA-MoE** 把 backbone scale 到 64 个专家,联合训练轨迹预测和 driving VQA 两个任务;③用 **GSPO**(序列级 RL,专门为 MoE 策略的路由不稳定性设计)做在线强化学习,优化多维度安全奖励。

**核心洞察** → mask diffusion 的"双向上下文 + 并行解码"特性天然适合按场景类型选择不同解码顺序(转弯用因果、跟车/对向来车这种需要长程预判的用反因果);而 MoE 用 LoRA 形式加在共享 backbone 的 FFN 里,既能 scale 容量又保持参数高效——**这个 LoRA-MoE 的具体数学形式,和我们之前精读的 [LoRA](lora.md) 论文(`o=W0z+BAz`)完全是同一个公式的多专家推广**(`o=W0z+Σgi(z)BiAiz`)。

**方法/论证** → LLaDA-V 多模态 backbone(图像+文本),FFN 层嵌入 64 专家 LoRA-MoE(rank 32),混合训练 motion prediction + driving VQA;先监督预训练,再用 GSPO 在线 RL 用多维度奖励(碰撞/合规/TTC/舒适/进度)微调。

**结果** → NAVSIM-v1 91.0 PDMS,NAVSIM-v2 89.7 EPDMS;完整消融链条:baseline 80.2 → +反因果解码 +2.0 → +CFG +2.4 → +LoRA-MoE+多任务 +1.9 → +GSPO RL +5.3 = 91.0——**RL 贡献比 MoE 本身更大**,这是值得批判性看待的一点(见第 6 节)。

**意义** → 给"MoE 具体该怎么嵌进 VLA backbone"提供了一个和 [ARTEMIS](artemis.md) 完全不同的答案:不是只在最后的轨迹生成/路由决策层加专家,而是把 MoE(以 LoRA 形式)嵌进共享 backbone 的**每个 FFN 层**,同时服务于理解(VQA)和生成(轨迹)两个任务——这是"表征层面"和"决策层面"路由之间的一个中间形态,值得补充进 [representation-synthesis-2026.md](../representation-synthesis-2026.md)。

## 中文精读

### 1. 解决了什么问题

Driving VLA 目前主要有两条技术路线:自回归 LLM 式(逐 token 从左到右生成动作序列,靠海量多模态预训练获得强泛化能力)和连续扩散 policy(通过去噪过程迭代精化动作预测,能建模复杂多模态分布)。**离散 mask diffusion** 是第三条正在语言/多模态领域兴起的路线——从全遮盖的序列开始,每一步并行预测所有被遮盖的 token,同时对低置信度的预测重新遮盖,让模型能利用双向上下文,不受自回归"从左到右"这个约束限制。这条路线理论上特别适合轨迹生成(比如转弯适合因果顺序、跟车这种需要长程预判的场景适合反因果顺序),但此前只有初步尝试,效果不如 SOTA 自回归方法,这篇论文要解决的就是这个性能差距。

### 2. 最关键的贡献(按重要性排序,附证据)

**① LoRA-MoE 的具体数学设计,这是全文和 MoE 直接相关的核心贡献。** 公式(3):

> "the output of a LoRA MoE layer with N experts is given by: `o = W0z + Σgi(z)Ei(z)`, where W0 denotes the pre-trained feed forward network (FFN) projection matrix, `Ei(z) = BiAiz` represents the low-rank adaptation of the i-th expert"

翻译:冻结的预训练 FFN 权重 `W0` 保持不变,再叠加 N 个低秩适配专家 `Ei(z)=BiAiz`,由一个 softmax 门控网络 `gi(z)` 决定每个专家的权重。**这就是 LoRA 的 `o=W0z+BAz` 公式,把单一的 `{A,B}` 换成了 N 对 `{Ai,Bi}` 加路由。** 消融证据(Table 2):无 MoE 84.7 → 16 专家 85.0 → 64 专家 86.6;LoRA rank=32(86.6)优于 rank=8(85.5)。

**② GSPO 这个 RL 算法本身的设计动机,是一个很具体、有针对性的 insight。** 原文点出了 token 级别 RL(GRPO)对 MoE 策略的一个根本问题:

> "Optimizing entire sequences avoids token-wise credit assignment and the associated instability from changing expert routes, which is a severe problem for GRPO."

翻译:按整条序列优化,能避开逐 token 做信用分配时因为专家路由发生变化而导致的不稳定——这是 GRPO 用在 MoE 策略上的一个严重问题。GSPO 把重要性比率的计算粒度从"每个 token"提升到"整条序列",从根本上绕开了这个问题。图 10 显示 GSPO 训练曲线比 GRPO 持续更高、更稳定。

**③ 完整消融链条揭示了各组件的真实贡献排序,这是这篇论文最值得批判性阅读的地方。**

> "Introducing our proposed reverse-causal decoding scheduler yields an additional +2.0 PDMS...Incorporating the LoRA-MoE layer and jointly training on both VQA and trajectory data provides a further +1.9 PDMS...applying GSPO-based reinforcement learning contributes a substantial +5.3 PDMS"

完整链条:baseline 80.2 → 反因果解码 +2.0(82.2)→ CFG +2.4(84.7 左右)→ LoRA-MoE+多任务 +1.9 → GSPO RL +5.3 = 91.0。**RL(+5.3)的边际贡献比 MoE 本身(+1.9)大得多**——论文标题主打"MoE",但真正的最大功臣其实是 GSPO 这个 RL 组件,presentation 时如果要强调 MoE 的价值,需要诚实说清楚这一点。

**④ 灵活解码顺序按场景类型适配,有具体场景分析支撑。** Table 7 + Fig.9:反因果解码整体最优(91.0 PDMS),在跟车、对向来车这种需要"看到后段结果、回头调整前段轨迹"的场景里效果最突出;因果解码更适合近期转弯这类场景;随机解码是均衡默认选项。

**⑤ 联合训练 VQA 和轨迹预测,证明"理解任务能正则化生成任务"这个思路在 driving VLA 里确实有用。** 消融显示联合训练比纯轨迹监督再加 +1.9 PDMS——不只是空谈"通用能力有用",有具体数字支撑。

### 3. Idea 具体落在哪里

**LoRA-MoE 嵌入的位置**是理解这篇论文和 ARTEMIS 本质区别的关键:LoRA-MoE 加在**共享多模态 backbone(基于 LLaDA-V)每一层的 FFN 里**,这个 backbone 同时处理轨迹生成和 driving VQA 两个任务。也就是说,专家路由发生在"理解 + 生成"共享的表征处理阶段,不像 ARTEMIS 那样只在最后的轨迹解码步骤才路由——这是一种更"深入 backbone 内部"的 MoE 集成方式。

**混合离散 token 化方案**:轨迹用数值 waypoint token 表示,和语义语言 token 交织在同一个序列里,论文提到这比纯文本表示的轨迹精度更高(具体对比数据在附录,笔记里不展开)。

### 4. 最大难点在哪里

不在 mask diffusion 本身(双向、可并行解码这个概念在 NLP 领域已经成熟),难点在于**GSPO 这个 RL 算法的设计**——要同时理解 MoE 的路由机制和策略梯度 RL 的信用分配原理,才能想到"把优化粒度提到整条序列级别"这个解法来避开路由不稳定问题。另外,联合训练 VQA 和轨迹预测这两个不同任务、还要通过 64 个 LoRA 专家做路由,训练数据配比怎么定、怎么避免一个任务的梯度压过另一个,论文没有展开讨论这些工程细节,是一个可能被低估的隐藏难点。

### 5. 哪些内容可以略过

公式(4)(5)GSPO 具体的重要性比率和 clip 目标函数推导——除非要自己实现,记住"序列级重要性比率 + PPO 式 clip"这个思路就够了。

网络架构细节(SigLIP-2 编码、图像 patch 切分方式、词表扩展到 146,350)——这些是工程实现细节,不影响理解核心方法。

Table 8/9 里和 nuScenes、其他 VLA baseline 的逐指标对比——记住"碰撞率 0.11%,匹配 UniAD 最优水平"这个结论就够了。

### 6. 假设、局限、值得质疑的地方

**论文自己在 4.5 节坦诚承认两条局限**,原文明确写:

> "our model currently receives only a front-view image as input, which leads to perception failures when important obstacles lie outside this field of view. Second, the model processes only the current frame without any temporal history, making it difficult to infer other agents' motion patterns and intent."

翻译:①目前只用单前视相机,视野外的重要障碍物会导致感知失败;②只处理当前帧、没有任何历史时序信息,难以推断其他智能体的运动模式和意图。**这两条局限值得反过来读一遍**:WAM-Diff 的 91.0 PDMS / 89.7 EPDMS,是在"比大多数同期论文输入信息更少"(没有环视、没有历史帧)的条件下取得的——这既可能说明这套 LoRA-MoE + GSPO 组合本身效率很高,也可能说明它在信息更完整的设置下还有进一步提升空间,presentation 时可以把这个"输入受限但分数不差"当成一个值得强调的效率论据,但也要提醒自己不能直接和有环视输入的方法(比如 Qwen-Drive-1.0)做完全公平的横向比较。

**值得质疑但需要读者自己算出来的一点**:论文标题把 "MoE" 放在很显著的位置,但自己的消融数字显示,纯粹 MoE 的边际贡献(+1.9,甚至消融表 2 单独看只有 84.7→86.6 即 +1.9)明显小于 RL(+5.3)。presentation 或写文献综述时,如果只看标题和摘要,容易高估 MoE 在这篇论文里的实际权重——这是需要读消融表格才能发现的落差,论文正文没有主动提醒读者注意这一点。

### 7. 是否能连 GitHub

**部分开放。** 我核实了 https://github.com/fudan-generative-vision/WAM-Diff 的 README:推理代码、训练代码、预训练权重都已经真实发布(权重放在 HuggingFace,2025 年 2 月 1 日起可用),300 stars、51 forks,活跃度不低。但 README 自己的路线图明确标注 **NAVSIM 评测代码和 GSPO 强化学习实现都还是 "TBD"(待定)**——也就是说你现在能跑推理、能做监督训练,但复现论文里报的 RL 后训练效果和标准评测流程,目前还做不到。

---

## English at-a-glance summary(无海报,理由见下)

这篇没有配单独的 SVG 海报——它的核心是一条"输入 → LLaDA-V backbone(FFN 嵌 64 专家 LoRA-MoE)→ mask diffusion 解码器(灵活解码顺序)→ GSPO 在线 RL 微调"的流水线,和 WA-JEPA/ARTEMIS 的图示逻辑高度相似,用文字描述能更快讲清楚这次的独特之处(LoRA-MoE 嵌入位置 + GSPO 设计动机),不需要再画一张新图重复类似的箱线结构。

| Aspect 方面 | Summary |
|---|---|
| **Problem 问题** | Discrete masked diffusion (bidirectional context, parallel decoding) is promising for driving VLAs but underexplored — prior attempts underperform SOTA autoregressive methods.<br>离散mask diffusion(双向上下文、并行解码)对driving VLA很有潜力但研究不足——已有尝试效果不如SOTA自回归方法。 |
| **Novelty 新意** | Systematic adaptation: flexible (non-causal) decoding orders + LoRA-MoE scaling the shared backbone to 64 experts (jointly trained on trajectory + VQA) + GSPO, a sequence-level RL algorithm designed specifically to avoid MoE routing instability.<br>系统性适配:灵活(非因果)解码顺序+LoRA-MoE把共享backbone扩展到64专家(联合训练轨迹+VQA)+GSPO(专为避免MoE路由不稳定设计的序列级RL算法)。 |
| **Core idea 核心想法** | LoRA-MoE: `o = W0z + Σgi(z)·Biai·z` — literally LoRA's reparametrization generalized to N routed experts. Embedded in every FFN layer of the shared backbone, not just the final decision step (contrast with ARTEMIS).<br>LoRA-MoE:`o = W0z + Σgi(z)·Biai·z`——就是LoRA重参数化推广到N个带路由的专家。嵌在共享backbone的每个FFN层,不只是最后的决策步骤(对比ARTEMIS)。 |
| **Hardest part 最大难点** | Not masked diffusion itself, but designing GSPO — requires understanding both MoE routing and RL credit assignment to realize sequence-level optimization avoids the instability of token-level credit assignment under changing expert routes.<br>难点不在mask diffusion本身,而在设计GSPO——需要同时理解MoE路由和RL信用分配,才能想到序列级优化能避开专家路由变化导致的token级信用分配不稳定问题。 |
| **Evidence 证据** | Full ablation ladder: baseline 80.2 → +reverse-causal decoding +2.0 → +CFG +2.4 → +LoRA-MoE+multitask +1.9 → +GSPO RL +5.3 = 91.0 PDMS. RL contributes MORE than MoE itself.<br>完整消融链:基线80.2→+反因果解码+2.0→+CFG+2.4→+LoRA-MoE+多任务+1.9→+GSPO RL+5.3=91.0 PDMS。RL的贡献比MoE本身更大。 |
| **Assumptions/limitations 假设/局限** | Authors admit: single front-view camera only (no surround view), no temporal history (single frame only) — the 91.0/89.7 scores come with less input than most peer papers. Also worth noting: MoE's own marginal contribution (+1.9) is much smaller than RL's (+5.3), despite MoE being in the title.<br>作者承认:只有单前视相机(无环视)、无历史帧(单帧)——91.0/89.7的分数是在比多数同期论文更少的输入下取得的。也值得注意:尽管标题带MoE,MoE自身的边际贡献(+1.9)远小于RL(+5.3)。 |
| **Code / GitHub 代码开源情况** | **Partially released** — github.com/fudan-generative-vision/WAM-Diff: inference/training code + weights are real (HuggingFace, since Feb 2025), but NAVSIM eval code and the GSPO RL implementation are still marked "TBD" in the repo's own roadmap. 300 stars, 51 forks.<br>**部分开放**——推理/训练代码+权重真实可用(HuggingFace,2025年2月起),但NAVSIM评测代码和GSPO RL实现仓库自己的路线图标注仍是"待定"。300星,51 forks。 |

## 一个月后只记住 5 件事

① LoRA-MoE 的具体公式和 LoRA(`o=W0z+BAz`)同构,只是从单一 adapter 扩展成 N 个 adapter + softmax 路由(`o=W0z+Σgi(z)BiAiz`)——是"共享 backbone + 轻量专家 adapter"这个思路在真实论文里的具体实现,直接呼应了我们读 LoRA 时想到的 CSDI-MoE 设计。

② GSPO 是专门为 MoE 策略设计的 RL 算法——因为 token 级别 RL(GRPO)对"专家路由会变"这件事不稳定,GSPO 把优化粒度提到整条序列级别来避开这个问题。

③ 完整消融链条显示 RL(GSPO,+5.3)比 MoE 本身(+1.9)贡献更大——presentation 时不能因为论文标题带 MoE 就默认 MoE 是最大功臣。

④ 反因果解码在"需要长程预判"的场景(跟车/对向来车)最有效,因果解码适合近期转弯——不是一种解码顺序打天下。

⑤ 诚实的局限:只用单前视相机、无历史帧,比大多数同期论文输入更少——这个分数是在信息更受限的条件下取得的,presentation 时可以当效率优势讲,但不能直接和输入更完整的方法做完全公平的横向比较。
