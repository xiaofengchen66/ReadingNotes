# CoWorld-VLA: Thinking in a Multi-Expert World Model for Autonomous Driving

- **Venue / year:** arXiv 2026(2605.10426v3,2026-08-25 最新版)
- **Authors:** Minqing Huang, Yujiao Xiang, Zihan Liang, Jiajie Huang, Jingqi Wang(通讯作者)等 10 人 — Afari Intelligent Drive(和 WA-JEPA 同一家公司)+ 电子科技大学 + 上海交通大学 + 北京邮电大学 + 东南大学 + 天津大学
- **Link:** https://arxiv.org/abs/2605.10426
- **来源说明:** 从 WA-JEPA 对比表格发现,arXiv API 核实后下载全文,用 `pdftotext` 提取逐句核对(PDF 本身有几处语法警告,不影响正文文字提取)。
- **paper_matrix.csv id:** 无
- **Code:** https://github.com/AFARI-Research/CoWorld-VLA(**已核实:真实发布**——含推理代码、VLM 特征缓存构建工具、HuggingFace 上的模型 checkpoint,10 stars、1 fork,较新但确实可跑)
- **Input / 输入:** **单帧图像**(和 WAM-Diff 一样,没有环视相机、没有历史帧)+ 文本(导航指令等)
- **Output / 输出:** 连续的未来轨迹(通过 diffusion 去噪生成,不是离散 token)

## ⚠ 精读最重要的发现:这篇的"Multi-Expert"不是 MoE

**这是本周精读里最需要单独拎出来讲的一点。** 我把全文搜了一遍,**"gate"、"gating"、"router"、"routing"、"sparse activation"、"top-k" 这几个词一个都没有出现**。CoWorld-VLA 标题里的"Multi-Expert"指的是**四条始终同时激活的并行分支**(语义交互、几何结构、动态演化、自车轨迹),各自独立生成一条候选轨迹,最后用**固定的全局标量权重**加权平均融合(见第 4 节的原文引用)。这是一种**多源监督表征融合(multi-source supervised representation fusion)**架构,**不是** ARTEMIS/WAM-Diff 那种"输入决定激活哪几个专家"的稀疏路由 MoE(Mixture-of-Experts)。

两者的关键区别:
- **MoE(ARTEMIS/WAM-Diff)**:多个可互相替代的子网络,一个门控网络根据当前输入**动态决定**用哪几个 + 权重多少,不同输入可能激活不同专家组合。
- **CoWorld-VLA 的"multi-expert"**:四个**功能不同、不可互相替代**的固定分支(一个管语义、一个管几何……),**所有输入都会激活全部四个**,只是最后融合权重是训练时学出来的**固定值**,不随输入变化。

presentation 给乔老师时,如果把这篇归到"MoE 用于自动驾驶"的例子里,会是一个事实性错误——它的英文标题用词确实容易造成这种误解,这也是为什么"读原文再下结论"这件事在这次精读里格外重要。

## TL;DR 浓缩版

**问题** → VLA 做自动驾驶,现有的中间推理机制都有缺陷:文本 CoT 没法保留连续的时空结构;单一的 latent world reasoning(比如一个统一的世界模型表征)要么信息不完整,要么和最终动作生成耦合太弱,只能当辅助监督信号。

**Gap** → 已有 world model 大多只是"辅助监督信号"(训练时用来正则化,推理时不直接参与动作生成),没有真正把预测出的世界知识当成显式条件去指导轨迹规划。

**Novelty** → 用四种"专家 token"(语义交互、几何结构、动态演化、自车轨迹)分别编码不同来源的世界知识,组成一个"多专家 Latent CoT";再用一个"分层多专家融合"(Hierarchical Multi-Expert Fusion, HMEF)diffusion 规划器,把这四类 token 转成轨迹生成的条件,联合去噪生成连续轨迹。

**核心洞察** → 单一的世界表征不够完整,因为规划真正需要的世界知识是多维度的(交互意图、空间结构、时间演化、行为目标),每一类知识都有最适合的"老师"信号去监督(JEPA 式表征监督语义交互、VGGT 特征监督几何结构、生成式世界模型 Wan 监督动态演化、轨迹级监督自车轨迹)——所以用四条并行的专门化分支各自学,再在动作生成阶段融合,比硬塞进一个表征里更完整。

**方法/论证** → 三阶段训练:①用 NuPlan 视频预训练一个 video diffusion transformer(Wan2.2-5B)做未来视频生成(**只在训练阶段当老师,推理时完全不跑**);②在 VLM(Qwen3-VL-2B)上用多专家监督微调;③训练 HMEF 融合规划器——每个专家分支独立生成候选轨迹,再用"detach 后加权平均"的方式融合(专家分支的梯度和融合权重的梯度互不干扰)。

**结果** → NAVSIM v1 PDMS 90.0,NAVSIM v2 EPDMS 90.0;消融显示四类 token 逐个加入持续提升(83.7→85.1→87.3/87.7→88.7),互补而非冗余;HMEF 融合规划器比简单 action expert 多贡献 0.9 分(89.1→90.0)。

**意义** → 证明了"把多源世界知识拆开监督、在动作生成阶段显式融合"这个思路本身是有效的——但它验证的是**多源表征融合**的价值,不是 MoE 路由的价值。presentation 时如果要谈"MoE 用于自动驾驶世界模型有哪些例子",这篇需要被排除在外(或者明确注明它不是真正的 MoE),真正的例子是 ARTEMIS 和 WAM-Diff。

## 中文精读

### 1. 解决了什么问题

VLA 做自动驾驶,想让模型具备"中间推理"能力,而不只是感知直接映射到动作。已有两条路都有问题:文本形式的 Chain-of-Thought(CoT)会丢失连续的空间/运动细节(语言本质上是离散、稀疏的,没法精确表达"这条轨迹在 t=2s 时的曲率是多少");单一的 latent world model 表征试图把所有世界知识压进一个向量,但规划需要的知识本身是多维度的(交通参与者的交互意图、道路几何结构、未来场景演化、自车的行为目标),硬塞进一个表征容易不完整,而且这类表征通常只在训练时当辅助监督信号,推理时并不真正参与轨迹生成。

### 2. 最关键的贡献(按重要性排序,附证据)

**① 四类专家 token 的设计和消融验证。** 每一类都有独立的监督信号来源,不是同一个 loss 硬分出来的四个分支。Table 4 的消融清楚显示互补性:只用自车轨迹 token(EgoT.)83.7 → 加几何结构 85.1 → 加语义交互 85.2 → 三者组合 87.3/87.7 → 四类全加 88.7,每加一类都有正贡献。原文结论:

> "using all four expert-token groups achieves 88.7, improving PDMS by 5.0 over EgoT. The two branches show that geometric, semantic, and dynamic supervision provide complementary planning cues rather than interchangeable information."

**② HMEF 融合规划器的"detach 后加权平均"训练技巧。** 每个专家分支各自算自己的去噪 loss 独立训练,融合权重的梯度**不会**传回专家分支内部:

> "the individual expert trajectories are detached from the computation graph before weighted averaging. Consequently, the fusion objective updates only the fusion weights rather than propagating gradients back into the individual expert denoising branches."

这样能把"专家各自学好"和"怎么加权组合"两件事解耦,避免融合阶段的梯度扰乱专家分支自己的学习——这是一个需要对训练动态有经验判断才能想到的细节。

**③ Table 5 的"表征 × 规划器"交叉消融,证明两个维度都独立有贡献。** 固定 VLM only,Full Experts 比 EgoT only 从 83.7 提升到 88.7;固定 Full Experts,HMEF 比简单 action expert(借用 ReCogDrive 的设计)从 89.1 提升到 90.0——不是"表征好就够了"或"规划器好就够了",两者叠加才是最好。

**④ 生成式世界模型 Wan 只在训练阶段当"老师",推理时完全不跑。**

> "the Wan model is not used in Stage 3 or during planning inference, while the Stage-2 VLM and the V-JEPA/VGGT encoders are frozen during Stage 3."

这是一个重要的工程设计:昂贵的视频生成模型只用来蒸馏监督信号,真正上车推理时不需要跑这个大模型,控制了实际部署的计算成本——这个思路和 DriveZero 的 DriveVFM(用冻结视觉基础模型蒸馏出感知骨干,推理时也不需要原模型)是同一类工程哲学。

**⑤ 种子稳定性验证(Table 7)。** 6 个不同随机种子下 PDMS 在 89.964-90.000 之间波动极小——证明这套 diffusion 规划器的结果是稳定可复现的,不是运气好抽到一个好种子,这是一个容易被忽略但值得肯定的严谨性细节。

### 3. Idea 具体落在哪里

**四类专家 token 各自的监督来源**是理解这篇论文的关键:语义交互 token 由 JEPA 式表征引导(呼应我们精读过的 WA-JEPA);几何结构 token 用 VGGT(视觉几何基础模型)特征监督;动态演化 token 由生成式世界模型 Wan 监督;自车轨迹 token 直接用轨迹级监督训练。四条分支分别在 VLM 的 latent 空间里学习,组成一个"面向规划的 Latent CoT"。

**HMEF 的具体融合方式**:每个专家 token 先按未来时间步组织(如果多个 token 对应同一个未来时间步,取平均),得到和规划时间步对齐的"逐步专家条件";历史自车轨迹和当前 ego state 单独编码后融合成一个条件向量;这些条件和噪声动作 token 一起送进 diffusion 去噪器,联合去噪生成连续轨迹——每个专家分支各自产出一条轨迹,最后用**训练时学出的固定全局标量权重**加权平均。

### 4. 最大难点在哪里

不在四类 token 的设计动机本身(每类用什么信号监督,论证清楚),难点在于**设计"detach-then-average"这个训练技巧,把专家分支的学习和融合权重的学习解耦**——如果直接端到端训练融合后的 loss,梯度会同时更新专家分支和融合权重,容易让某个学得快的分支"带偏"其他分支的学习(某种意义上是标准 MoE"赢者通吃"问题的另一种表现形式,只是这里换了个场景)。另外,三阶段训练要协调三个不同模型(Wan2.2-5B 视频扩散、Qwen3-VL-2B 的 VLM、外加冻结的 JEPA/VGGT encoder)的训练顺序和学习率,工程复杂度不低。

### 5. 哪些内容可以略过

Table 6 的去噪步数消融(5/10/20 步)——记住"10 步是精度和效率的最佳平衡点"就够了。

Table 3 里 FVD(视频生成质量指标)的具体对比数字——这是评估 Wan 视频生成分支质量的辅助指标,和最终轨迹规划表现关系不大。

Appendix A.1.4 三阶段训练的具体超参数(学习率、batch size、训练步数)——除非要复现训练。

### 6. 假设、局限、值得质疑的地方

**论文自己在 Limitations 一节承认两条局限**:

> "The primary limitation of our framework is the substantial computational overhead incurred during multi-stage training...The current model is also limited to single-image inputs."

翻译:①三阶段训练的计算开销很大(虽然推理时 Wan 不参与,但训练阶段这些 teacher 模型都要跑一遍);②目前只支持单图输入,没有多摄像头——这和 WAM-Diff 是同一个局限,两篇都来自同一家公司(Afari Intelligent Drive),可能反映了这家公司当前的一个共同技术选择或数据限制。

**值得质疑、论文没有讨论的一点**:融合权重是"global scalar"(全局标量,不随输入变化)——不管当前是什么场景,四个专家的融合权重都是同一组固定值,不会根据"这个场景更需要几何信息还是更需要语义交互信息"去动态调整。如果真的要做到"不同场景侧重不同专家",理论上应该让融合权重依赖输入(这才更接近真正 MoE 路由的精神),但 CoWorld-VLA 选择了更简单、更稳定、但表达力更受限的全局固定权重方案。这个"稳定性 vs 场景自适应能力"的取舍,论文完全没有讨论或做对照实验——**这其实正好指向了我们在 [representation-synthesis-2026.md](../representation-synthesis-2026.md) 里提的研究方向②(表征层面的 MoE 路由):如果把 CoWorld-VLA 的固定融合权重换成一个真正依赖场景输入的动态路由,会不会更好?这篇论文本身没有做,但它的架构恰好是一个现成的、可以直接拿来验证这个想法的起点。**

### 7. 是否能连 GitHub

**可以,而且是真实、较新发布的代码。** 我核实了 https://github.com/AFARI-Research/CoWorld-VLA:含推理代码、VLM 特征缓存构建工具、HuggingFace 上的模型 checkpoint,10 stars、1 fork,发布时间较新(2026 年 5 月一系列更新)。规模比 WA-JEPA/WAM-Diff 小(星标少很多),但确实是可运行的真实代码,不是占位符。

---

## English at-a-glance summary(无海报,理由见下)

这篇没有配单独的 SVG 海报——它的架构本质上是"四条独立监督分支 + 加权融合"这个相对简单的结构,而且和这周已经画过图的 WA-JEPA、ARTEMIS 结构相似度较高,重复画意义不大。最需要传达的信息(它不是真正的 MoE)已经在最前面的专门章节里讲清楚了,比塞进一张图更醒目。

| Aspect 方面 | Summary |
|---|---|
| **⚠ Critical note 关键提醒** | Despite the title, CoWorld-VLA has NO gating/routing mechanism (verified via full-text search — zero occurrences of "gate/router/sparse activation"). Its "multi-expert" means 4 always-active parallel branches fused via FIXED global scalar weights, not input-dependent MoE routing.<br>尽管标题如此,CoWorld-VLA没有任何门控/路由机制(全文搜索验证——"gate/router/sparse activation"零出现)。它的"multi-expert"指4条始终激活的并行分支,用固定的全局标量权重融合,不是依赖输入的MoE路由。 |
| **Problem 问题** | Existing intermediate reasoning for driving VLAs is flawed: textual CoT loses continuous spatiotemporal structure; a single latent world representation is incomplete and usually only an auxiliary training signal, not a direct planning condition.<br>驾驶VLA现有的中间推理机制有缺陷:文本CoT丢失连续时空结构;单一latent世界表征不完整,通常只是辅助训练信号,不直接参与规划。 |
| **Novelty 新意** | 4 specialized "expert tokens" (semantic interaction, geometric structure, dynamic evolution, ego trajectory), each supervised by a different teacher signal (JEPA / VGGT / Wan generative world model / trajectory supervision), fused by a Hierarchical Multi-Expert Fusion (HMEF) diffusion planner.<br>4个专门化的"专家token"(语义交互、几何结构、动态演化、自车轨迹),各自由不同的老师信号监督(JEPA/VGGT/生成式世界模型Wan/轨迹监督),由分层多专家融合(HMEF)diffusion规划器融合。 |
| **Core idea 核心想法** | Each expert branch independently generates a trajectory; a "detach-then-average" scheme learns global fusion weights without letting fusion gradients disturb each branch's own denoising training.<br>每个专家分支独立生成一条轨迹;"detach后加权平均"方案学习全局融合权重,不让融合梯度干扰各分支自己的去噪训练。 |
| **Hardest part 最大难点** | Not the 4-token design itself, but the detach-then-average trick that decouples per-expert learning from fusion-weight learning — and coordinating a 3-stage pipeline across 3 different models (Wan2.2-5B, Qwen3-VL-2B, frozen JEPA/VGGT encoders).<br>难点不在四token设计本身,而在把各专家学习和融合权重学习解耦的detach-then-average技巧——以及协调跨3个不同模型(Wan2.2-5B、Qwen3-VL-2B、冻结的JEPA/VGGT编码器)的三阶段训练流程。 |
| **Evidence 证据** | Ablation shows all 4 tokens are complementary (83.7→88.7 as tokens are added one by one); HMEF beats a simple action expert by +0.9 PDMS (89.1→90.0); results stable across 6 random seeds (89.964-90.000).<br>消融显示四类token互补(逐个加入从83.7升到88.7);HMEF比简单action expert多0.9分PDMS(89.1→90.0);6个随机种子下结果稳定(89.964-90.000)。 |
| **Assumptions/limitations 假设/局限** | Authors admit substantial multi-stage training overhead and single-image-only input. Unaddressed: fusion weights are fixed GLOBAL scalars, not input-dependent — a real gap versus true MoE routing that the paper doesn't explore or discuss.<br>作者承认多阶段训练开销大、只支持单图输入。未讨论:融合权重是固定的全局标量,不依赖输入——这和真正的MoE路由有实质差距,论文完全没有探讨或讨论这一点。 |
| **Code / GitHub 代码开源情况** | **Released, real code** — github.com/AFARI-Research/CoWorld-VLA: inference code, VLM feature cache builder, HuggingFace checkpoint. 10 stars, 1 fork, fairly new but genuinely runnable.<br>**已开源,真实代码**——含推理代码、VLM特征缓存构建工具、HuggingFace checkpoint。10星,1 fork,较新但确实可运行。 |

## 一个月后只记住 5 件事

① **最重要**:CoWorld-VLA 的"Multi-Expert"不是 MoE(全文搜不到 gate/router/routing 这些词)——是 4 个始终同时激活的并行分支,用固定的全局标量权重加权平均融合,是多源监督表征融合架构。

② 四类专家 token 各有独立的监督来源:语义交互 ← JEPA 式表征,几何结构 ← VGGT,动态演化 ← 生成式世界模型 Wan,自车轨迹 ← 轨迹级监督——不是同一个 loss 硬分出来的。

③ Wan 这个大视频扩散模型只在训练时当"老师",推理时完全不跑——控制了实际部署的计算成本。

④ 融合权重是全局固定的,不随场景变化——这是一个"稳定但不自适应"的设计取舍,没有做动态路由,这恰好是一个可以直接拿来验证"表征层面 MoE 路由"这个研究方向的现成起点。

⑤ 和 WAM-Diff 一样,只支持单图输入,没有多摄像头/历史帧——这两篇都来自同一家公司(Afari Intelligent Drive)。
