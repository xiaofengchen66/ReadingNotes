# ARTEMIS: Autoregressive End-to-End Trajectory Planning with Mixture of Experts for Autonomous Driving

- **Venue / year:** arXiv 2025(2504.19580v2,2025-05-04)
- **Authors:** Renju Feng, Ning Xi, Duanfeng Chu, Rukang Wang, Zejian Deng, Anzheng Wang, Liping Lu, Jinxiang Wang, Yanjun Huang — 武汉理工大学 + 香港大学 + 东南大学 + 同济大学
- **Link:** https://arxiv.org/abs/2504.19580
- **来源说明:** 从 WA-JEPA 的对比表格里发现这篇标题直接带"Mixture of Experts",用 arXiv API 核实后下载全文,用 `pdftotext` 提取逐句核对。
- **paper_matrix.csv id:** 无
- **Code:** https://github.com/Lg0914/ARTEMIS(**已核实:占位仓库,只有 README,2 次提交**——README 明确写"Code will be released once the paper is accepted",目前无法复现)
- **Input / 输入:** 8 路相机图像(前左/前/前右拼接为 1024×256)+ 5 个 LiDAR 点云融合(256×256,覆盖 64m×64m)+ ego state(控制指令/2D 速度/加速度)+ 2 秒历史(4 帧)
- **Output / 输出:** 未来 4 秒轨迹(8 个未来点,2Hz,每点含 x、y、朝向角)

## TL;DR 浓缩版

**问题** → 传统模块化自动驾驶有误差逐级传递问题;现有端到端模型大多是"一次性"(static one-shot)推理范式,不能很好捕捉环境的动态演变;单一网络结构在处理"同一场景下人类司机可能有多种合理选择"这种驾驶行为的内在多模态性时,表达能力有限。

**Gap** → 已有 MoE 用于规划的工作只在 nuPlan 这类非视觉、结构化输入场景验证过;扩散模型式端到端方法能建模轨迹多样性,但本质仍是"一次性"生成范式,没有真正逐步自回归地、随时间演化地生成轨迹同时做专家路由。

**Novelty** → 把自回归轨迹生成和 MoE 结合:每生成一个未来轨迹点,都根据当前场景上下文(BEV 特征 + 规划历史)动态路由到最合适的专家网络。作者自称"据我们所知,是首次把 MoE 引入端到端自动驾驶"——这个"首次"需要打个折扣看(见第 6 节)。

**核心洞察** → 与其用显式的驾驶指令(左转/直行/右转)去指导专家选择(这类指令经常和真实专家轨迹不一致,论文自己统计过训练集里存在这种偏差),不如让路由网络自己学一个"内生的"(endogenous)专家分配方式——这样能天然建模驾驶行为的多模态性,不依赖可能有偏差的外部引导信号。

**方法/论证** → 三大模块:感知模块(TransFuser 式,图像+LiDAR 融合 BEV 特征,**完全是单一共享网络,不含任何 MoE**)→ 自回归规划模块(每步用 MoE 块处理 BEV 特征 + 规划历史查询,路由到 top-2 专家,DeepSeekMoE 式:5 个专属专家 + 1 个共享专家,配一个"batch reallocation"工程加速策略)→ 轨迹精炼模块(语义运动学优化 + 级联交叉注意力)。

**结果** → NAVSIM navtest 标准指标 87.0 PDMS,扩展指标 83.1 EPDMS,同骨干网络(ResNet-34)下 SOTA;消融显示去掉 MoE 模块单项掉 4.1 分,是三个组件里掉分最多的;"内生路由"比"显式驾驶指令路由"高 3.5 分。

**意义** → 给"MoE 到底该加在自动驾驶 pipeline 的哪一层"这个问题一个具体、可验证的答案:**加在自回归轨迹生成/规划这一层**,而不是感知/表征学习这一层——感知模块依然是单一共享的 TransFuser 骨干,完全没有 MoE。这直接回应了我们在 [representation-synthesis-2026.md](../representation-synthesis-2026.md) 里提的问题(现有 MoE-driving 工作是决策层路由还是表征层路由):**ARTEMIS 明确是决策层路由**。

## 中文精读

### 1. 解决了什么问题

传统模块化自动驾驶(感知→预测→规划分开做)有误差逐级传递的老问题。端到端方法解决了这个问题,但论文指出一个新问题:现有端到端模型大多是"一次性"(static one-shot)推理——一次性吐出整条轨迹或者做多轮去噪(扩散模型),没有真正逐步跟随环境动态演化去调整决策。更深一层的问题是:驾驶行为本身有内在的多模态性——同一个路口场景,人类司机可能选择直行、可能选择左转,单一网络架构很难同时表达这种多样性,尤其是当引导信号(比如导航给的驾驶指令)本身和实际最优轨迹不一致的时候。

### 2. 最关键的贡献(按重要性排序,附证据)

**① MoE 模块本身的消融证据,是全文最重要的证据。** 去掉 MoE 模块单项掉分 4.1 (Table III),比去掉自回归模块(-3.0)、去掉精炼模块(-2.3)都大——说明 MoE 不是锦上添花,是三个组件里贡献最大的一个。

**② "内生路由 vs 显式驾驶指令路由"的对比实验,直接回应了论文自己提出的问题。** Table IV:内生路由(让模型自己学路由)87.0,显式驾驶指令路由 83.5——差 3.5 分。论文用具体数字支撑了这个设计动机:训练集里左转样本超 2 万条、右转不到 1 万条、直行超 5 万条,分布严重不均衡,而且一部分样本的驾驶指令和实际专家轨迹方向直接矛盾(Fig.3)。

**③ 专家数量的"甜点"实验(Table VII)。** 3→5 个专家逐步变好,但 5→10 个反而掉 1.5 分——过多专家在有限训练数据下会"资源分散、功能重叠"(原文原话)。这是一个值得记住的工程经验:不是专家越多越好。

**④ 单专家激活实验(Table V),证明没有专家坍缩现象。** 没有任何一个专家能单独达到路由混合后的效果(最好的单专家 86.2 vs 混合 87.0),而且 5 个专家的单独表现相对均衡(83.4-86.2 之间,没有"死专家")——从侧面证明了这套 MoE 设计没有陷入常见的负载不均衡/专家坍缩问题。

**⑤ 明确不用负载均衡损失(expert balance loss)这个反直觉设计。** 这和 Switch Transformer 等主流 MoE 论文的标准做法正面冲突,论文认为在场景分布本就不均衡的数据集上强行均衡专家负载,反而会妨碍专家学到真正专门化的知识——但这个判断**没有做"加上均衡损失"的对照实验去验证**,是一个声称但未证实的假设(见第 6 节)。

### 3. Idea 具体落在哪里

**路由查询(routing query)的构造是最值得学的细节**:为了保证路由网络输入维度固定,论文特意把历史规划查询从拼接向量里**剔除**,只用"当前时间步嵌入 + ego state + 当前规划查询"构造路由查询 `Qr = Concat(TEt, Qs, Qt)`——这个"喂给路由器哪些信息、不喂哪些"的取舍是隐藏在架构细节里但很关键的设计判断:如果把容易带偏的历史信息也喂给路由器,可能会重新引入前面想避免的"跟着错误引导信号走"的问题。

**Batch Reallocation(批量重分配)是一个纯工程优化,但效果很实在**:把激活相同专家的样本重新排序、批量处理、再恢复原始顺序,训练速度从 19.2 提升到 43.5 samples/sec(batch size 从 64 到 256)。这不是理论创新,但对训练大规模 MoE 模型很实用。

### 4. 最大难点在哪里

不在 MoE 概念本身(路由 + 专家这套机制在 LLM 领域已经很成熟),难点在于**怎么设计路由查询,让路由决策既能感知当前驾驶场景、又不被有噪声的外部信号带偏**——上面第 3 节提到的"剔除历史规划查询"就是为了解决这个问题的具体设计选择,需要仔细权衡"给路由器多少上下文"和"避免引入偏差"之间的取舍。另外,batch reallocation 这类工程优化虽然只是效率技巧,但要做对(排序、分块、恢复顺序全部正确,不引入 bug)也需要细致的工程实现——训练速度提升超过一倍证明这不是可有可无的优化。

### 5. 哪些内容可以略过

Table I/II 里和 DiffusionDrive、VADv2、Hydra-MDP(++) 等一堆 baseline 的逐指标对比——记住"同骨干网络下达到 SOTA,尤其 EP/NC/TTC 领先"这个结论就够了。

公式(4)-(9) 里 batch reallocation 的具体张量重排索引推导——除非要自己实现,理解"把同专家的样本分到一起批量处理、再还原顺序"这个思路就够了。

轨迹精炼模块(Semantic Kinematic Optimization + Cross-Attention Refinement)的具体网络结构(GRU、卷积层数等)——这个模块的消融贡献(-2.3)不如 MoE 模块(-4.1)重要,细节可以先跳过。

### 6. 假设、局限、值得质疑的地方

**"首次把 MoE 引入端到端自动驾驶"这个 claim 需要打折扣看。** 论文自己在引言里写:

> "Previous work, which was based on structured data representations as input, incorporated Mixture-of-Experts (MoE) into planning tasks and demonstrated strong performance on the NuPlan dataset."

翻译:此前已有工作(基于结构化数据输入)把 MoE 用进了规划任务,并在 NuPlan 数据集上取得了不错的表现。**这说明 MoE 用于自动驾驶规划本身并不是完全的首次**,作者的"首次"实际上限定在"视觉端到端 + 自回归生成"这个更窄的范围内。presentation 时如果要引用这个"首次",最好把限定条件说清楚,不要简化成"第一个把 MoE 用到自动驾驶的工作"——这正是我们上周从综述里学到的教训(读 novelty 声明要对照 related work 核实,不能照单全收)。

**测试集规模的一个疑点**:论文写"the training set comprises 1192 scenarios, while the test set contains 136 scenarios"——这和 NAVSIM 标准 navtest(12k 场景)差了近 100 倍。论文没有解释这个更小的子集是怎么筛出来的、和标准 navtest 是什么关系,这让 Table I/II 里和其他方法的对比可比性存疑——除非其他 baseline 也是在同样这个 136 场景子集上测的(论文没有明说)。这是我读的时候自己发现的一个具体疑点,presentation 时如果要引用这篇的数字,最好先确认清楚这一点。

**不用负载均衡损失的判断没有消融验证。** 论文只是论证式地说"可能会妨碍专门化",但没有做"加上均衡损失会怎样"的对照实验去证实——这是一个声称但未证实的设计选择。

### 7. 是否能连 GitHub

**不能。** 我核实了 https://github.com/Lg0914/ARTEMIS :仓库只有一个 README 文件,2 次提交,明确写着"Code will be released once the paper is accepted"(论文被接收后才放代码),22 stars。目前完全无法复现。

## English at-a-glance summary(无海报,理由见下)

这篇没有配单独的 SVG 海报——它的架构本身不复杂(感知→自回归 MoE 规划→精炼三段式),用文字描述比画一张新图更省力也更清楚:感知模块(TransFuser,单一共享网络,无 MoE)输出 BEV 特征 → 自回归规划模块逐点生成轨迹,每一步用路由查询(仅含当前时间步 + ego state + 当前规划查询,刻意排除历史规划查询)选择 top-2 专家(5 专属 + 1 共享,DeepSeekMoE 式)处理 BEV 特征 → 精炼模块做运动学优化和交叉注意力微调 → 输出 4 秒/8 点轨迹。

| Aspect 方面 | Summary |
|---|---|
| **Problem 问题** | E2E driving models are typically "static one-shot" (generate the whole trajectory at once) and struggle to capture the inherent multi-modality of driving behavior (multiple valid choices at the same scene).<br>端到端驾驶模型通常是"一次性"生成整条轨迹,难以捕捉驾驶行为的内在多模态性(同一场景可能有多个合理选择)。 |
| **Novelty 新意** | Combines autoregressive trajectory generation with MoE: routes each waypoint-generation step to specialized experts based on current scene context, not an explicit (and often noisy) driving-command signal.<br>把自回归轨迹生成和MoE结合:根据当前场景上下文(而非有噪声的显式驾驶指令)为每一步生成的轨迹点路由到专门的专家。 |
| **Core idea 核心想法** | Routing happens at the PLANNING stage (autoregressive waypoint generation), not the PERCEPTION stage — the TransFuser backbone stays a single shared network with zero MoE.<br>路由发生在规划阶段(自回归轨迹点生成),不是感知阶段——TransFuser骨干始终是单一共享网络,完全没有MoE。 |
| **Hardest part 最大难点** | Designing the routing query to sense scene context without being biased by noisy external signals — solved by deliberately excluding historical planning queries from the routing input.<br>设计路由查询,既能感知场景上下文又不被有噪声的外部信号带偏——解法是刻意把历史规划查询排除在路由输入之外。 |
| **Evidence 证据** | Removing MoE alone drops PDMS by 4.1 (largest single-component drop, Table III); intrinsic routing beats driving-command routing by 3.5 PDMS (Table IV); 5 experts is the sweet spot, 10 hurts (Table VII); no single expert matches the routed mixture (Table V).<br>单独去掉MoE掉4.1分(消融里最大的单项掉分,表III);内生路由比驾驶指令路由高3.5分(表IV);5个专家是甜点,10个反而变差(表VII);没有单个专家能匹敌路由混合(表V)。 |
| **Assumptions/limitations 假设/局限** | "First to bring MoE into E2E AD" is overstated — the paper's own related work cites prior MoE-for-planning work on NuPlan. Test set is only 136 scenarios vs. NAVSIM's standard 12k navtest, unexplained. No expert-balance-loss ablation to justify skipping it.<br>"首次把MoE引入端到端自动驾驶"这个说法有夸大——论文自己的related work就引用了在NuPlan上做MoE规划的先例。测试集只有136个场景,远小于NAVSIM标准navtest的12k,没有解释原因。跳过负载均衡损失这个决定没有消融实验支撑。 |
| **Code / GitHub 代码开源情况** | **Not released** — github.com/Lg0914/ARTEMIS is a placeholder (README only, 22 stars), code promised "once accepted."<br>**未开源**——仓库只是占位符(只有README,22星),代码承诺"论文被接收后"才放出。 |

## 一个月后只记住 5 件事

① MoE 加在"自回归轨迹生成"这一层,不是感知/表征层——感知模块(TransFuser 骨干)完全是单一共享网络,专家路由只发生在逐点生成轨迹的规划阶段。

② 消融显示 MoE 是三个模块里贡献最大的一个(去掉掉 4.1 分,比自回归结构本身 -3.0、精炼模块 -2.3 都大)。

③ 用"内生路由"(让模型自己学路由)代替"显式驾驶指令路由",因为训练数据里指令和专家轨迹经常不一致——这是一个具体、有数据支撑的设计动机,不是拍脑袋。

④ 专家数量有甜点(5 个最好,10 个反而掉分)——不是越多越好,会资源分散。

⑤ "首次把 MoE 用于端到端自动驾驶"这个说法要打折扣看——之前有工作在 nuPlan(非视觉、结构化输入)场景做过,ARTEMIS 首创的是"视觉输入 + 自回归生成"这个更具体的组合。
