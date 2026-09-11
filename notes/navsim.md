# NAVSIM: Data-Driven Non-Reactive Autonomous Vehicle Simulation and Benchmarking

- **Venue / year:** NeurIPS 2024(Datasets and Benchmarks Track)
- **Authors:** Daniel Dauner, Marcel Hallgarten, Tianyu Li, Xinshuo Weng, Zhiyu Huang, Zetong Yang, Hongyang Li, Igor Gilitschenski, Boris Ivanovic, Marco Pavone, Andreas Geiger, Kashyap Chitta — 图宾根大学 + OpenDriveLab(上海AI Lab)+ NVIDIA Research + Bosch + 多伦多大学 + 斯坦福
- **Link:** https://arxiv.org/abs/2406.15349(v2)
- **来源说明:** 通过 arXiv API 核实 ID 后下载全文,用 `pdftotext` 提取原文逐句核对。
- **paper_matrix.csv id:** 无
- **Code:** https://github.com/autonomousvision/navsim(**已核实:真实、成熟、持续维护的代码库**——1.1k stars,128 forks,最新 release v2.2.2(2025年9月),含训练/评测脚本、leaderboard 提交工具、多个数据集 split、baseline agent 实现、可视化教程,Apache 2.0)
- **Input / 输入:** 单帧真实传感器数据(8 路环视相机 1920×1080 + 5 个 LiDAR 合并点云,可选 3 个历史帧,共 1.5 秒 @2Hz)+ ego status(速度/加速度/导航目标)
- **Output / 输出:** 一段未来 h=4 秒的轨迹(位姿序列);评测系统本身输出 PDM Score(PDMS ∈ [0,1]),由 NC/DAC/EP/TTC/Comfort 等子分数聚合而成

## TL;DR 浓缩版

**问题** → 怎么评测端到端驾驶策略?开环评测(拿真实数据算轨迹位移误差)简单但不反映真实驾驶表现;闭环评测(在仿真器里真开)更真实,但算力开销巨大,且现有仿真器和真实数据之间存在明显 domain gap。

**Gap** → 领域内论文数量快速增长,但缺乏一个能大规模复现、又能可靠反映闭环表现的评测标准,导致同一批方法在不同论文里排名不一致,难以下定论。

**Novelty** → 提出"非反应式仿真"(non-reactive simulation)这个折中方案:只在场景初始帧查询一次策略,之后轨迹固定不变、用简化 BEV 抽象去推演 4 秒,不需要真正跑一个完整的传感器仿真器,却能算出考虑碰撞/舒适度/进度的仿真类指标,而不只是位移误差。

**核心洞察** → "非反应式"这个简化牺牲了长期闭环交互的真实性,但用真实传感器数据(不是仿真渲染的)配合规则化的 PDM Score 评分函数,比传统开环位移误差指标、甚至比真正的闭环评分(CLS)相关性更高——用一个廉价的中间方案,换到了比开环更可信、又比闭环更可扩展的评测。

**方法/论证** → 提出 PDM Score(NC 碰撞惩罚 × DAC 区域合规惩罚 × [EP 进度 + TTC 防碰撞时间 + 舒适度的加权平均]);在 OpenScene(nuPlan 的重新分发版)上做场景过滤,筛出"有挑战性"的 10 万+ 场景(navtrain/navtest);用 nuPlan 真闭环仿真器,对 265 个不同 planner 系统性验证 PDMS 和真实闭环分数(CLS)的相关性;举办 CVPR 2024 挑战赛(143 队 463 份提交)验证实用性。

**结果** → PDMS 和闭环 CLS 的相关性显著高于开环 OLS;TransFuser(1 块 GPU 训 1 天)PDMS 达 84.0,反超需要 80 块 GPU 训 3 天的 PARA-Drive(同为 84.0,追平而非反超)和 UniAD(83.4,确实被反超)。

**意义** → 给整个端到端驾驶领域提供了一个已被广泛采纳、可复现、开源、持续维护的评测基础设施——这周读的 WA-JEPA、Drive-JEPA、DriveZero、SparseDriveV2、Qwen-Drive-1.0 引用的 PDMS/EPDMS 分数,几乎全部来自这个框架(或它的 v2 扩展)。

## 中文精读

### 1. 解决了什么问题(为什么重要)

论文开篇点出评测端到端驾驶面临的四个具体困境:①像 nuScenes 这样的数据集本来是为感知任务(目标检测)设计的,拿来做规划评测时,大部分场景其实"外推历史轨迹"就能拿高分,导致"瞎开"(只看过去轨迹、不看传感器)的策略也能刷出 SOTA;②驾驶是多目标任务(安全/舒适/进度要兼顾),但常见的位移误差(ADE)指标经常错判——一条安全但和人类录像不一样的轨迹会被扣分,即使它本身没问题(见论文 Fig.1);③驾驶涉及多智能体交互,理想情况下评测应该是交互式的(闭环),但现有仿真器的传感器渲染和真实数据差距很大;④缺乏标准化评测协议,导致不同论文之间的指标口径不一致、没法公平比较。

NAVSIM 要解决的问题就是:**在"开环太简单不可信"和"闭环太贵难复现"这两个极端之间,找一个两边都要不完全要的中间地带。**

### 2. 最关键的贡献(按重要性排序,附证据)

**① PDM Score 本身的两段式聚合设计,这是全文最核心的方法论贡献。** 公式:

`PDMS = (∏_{m∈{NC,DAC}} score_m) × (Σ weight_w × score_w / Σ weight_w)`,w ∈ {EP, TTC, Comfort}

前半部分是**乘法硬惩罚**——碰撞(NC)或出界(DAC)直接让 `score=0`,整个 PDMS 清零,不可能被后面的"进度快"抵消;后半部分才是**加权平均**——进度(EP)、防碰撞时间(TTC)、舒适度(Comfort)三项加权综合。这个设计哲学很明确:**"绝对不能做的事"(撞车、出界)必须是硬性否决,"做得好不好"才是可以打分权衡的**。

**② 场景过滤策略,证明了"大多数人类驾驶数据其实很无聊"这件事本身需要被系统性解决。** 证据:在未过滤的 OpenScene 数据上,一个"恒速恒向"的傻瓜 baseline 能拿到 79% 的 PDMS(人类才 91%)——说明大部分场景闭着眼睛直行就能应付。过滤掉这类平凡场景后(排除恒速 agent 能拿高分、或人类轨迹本身得分低于 0.8 的场景),恒速 agent 的分数暴跌到 22%,人类还能保持 95%——数据集难度被大幅拉开,才有区分度可言。

**③ 非反应式仿真的相关性验证,这是"我们的廉价指标真的可信"这个说法唯一的实证支撑。** 第 4.1 节系统性地找了 265 个不同类型的 planner(37 个规则based + 114 个学习式,涵盖 Constant/IDM/PDM-Closed/PlanCNN/UrbanDriver 五大类),全部在真正的 nuPlan 闭环仿真器里跑一遍,拿真实闭环分数(CLS)去对比 PDMS 和传统开环指标(OLS)的 Spearman/Pearson 相关系数,证明 PDMS 全方位优于 OLS,而且对每一类 planner 单独看都成立(Fig.4)。**这不是空口宣称"我们的指标更好",是拿 265 个不同风格的策略去实测验证出来的。**

**④ "简单方法能追平/反超复杂架构"这个发现,是这篇论文能被广泛引用、成为事实标准的重要原因。** 原文明确写出:

> "UniAD reaches a PDMS of 83.4, which, together with PARA-Drive, do not surpass the performance of TransFuser and LTF, despite the need for more demanding training, e.g., 80 GPUs for 3 days to train PARA-Drive versus 1 GPU for 1 day for TransFuser."

这个结果本身不是方法论贡献,但它证明了这套评测**有真实的区分度**,而且戳破了"堆算力堆架构就一定更好"这个默认假设——这也是为什么这篇论文之后几乎整个领域都开始用它当标准 benchmark。

**⑤ CVPR 2024 挑战赛 + 持续维护的排行榜,把这套框架从一篇论文变成了活的社区基础设施。** 143 队、463 份提交、13 个国家参赛——这也是为什么这周读的所有论文,无论是 WA-JEPA、DriveZero 还是 Qwen-Drive-1.0,报的分数全都是 PDMS/EPDMS,它们其实都在向同一个"共同语言"对齐。

### 3. Idea 具体落在哪里

**非反应式仿真的具体机制**(第 3 节):策略只在场景开始时被查询一次,产出一整段未来轨迹;之后用一个 LQR 控制器 + 运动学自行车模型,以 10Hz 频率把这段轨迹"执行"出来,期间环境不对策略的决策做出任何反馈——也就是说,场景里的其他车辆、行人依然按照真实录像里的轨迹走,只有自车按照策略给出的轨迹走,两者互不影响。这个简化省掉了"真正模拟其他智能体会怎么反应"这个最贵的部分,但因为背景是真实传感器数据(不是渲染出来的),画面本身没有 domain gap 问题。

**场景过滤的具体阈值**(第 3.1 节):用一个恒速 agent 作为"这个场景是不是太简单"的探测器——如果恒速 agent 在某个场景能拿到 PDMS > 0.8,或者人类的真实轨迹本身在这个场景只能拿到 PDMS < 0.8(标注误差导致),这个场景就被剔除。最终得到 navtrain(103k)和 navtest(12k)两个标准 split。

### 4. 最大难点在哪里

不在理解"非反应式仿真"这个概念本身(挺直观:只问一次、之后轨迹不变,用规则去推演后果),难点在于**证明这个简化不会让评测结果失真**——这需要第 4.1 节那一整套系统性相关性实验,没有这套验证工作,"非反应式仿真"就只是一个偷懒的简化,不会被社区采纳。另外,理解 PDM Score 公式本身"乘法硬惩罚 + 加权平均"这个两段式设计,以及为什么碰撞/出界必须用乘法惩罚而不是加权平均里的一项,也是需要仔细读公式才能理解的细节——这背后的道理是:碰撞和出界是"做错了就不该有任何东西能弥补"的硬约束,不能被"进度很快"这种软指标抵消掉。

### 5. 假设、局限、值得质疑的地方

**论文自己在 Discussion 一节非常诚实地列了三条局限**,这是一篇诚实度很高的基础设施论文:

> "A high PDMS does not always imply a high CLS, since our framework does not consider reactiveness or the compounding accumulation of errors in closed-loop simulation."

翻译:PDMS 高不代表 CLS(真闭环分数)一定高,因为这套框架不考虑反应性、也不考虑闭环仿真里误差的累积效应。**这正是为什么 WA-JEPA、DriveZero 这些论文除了报 NAVSIM 分数,还要额外在 HUGSIM 上做零样本闭环验证的原因——NAVSIM 自己就承认它不能完全替代真正的闭环仿真。**

> "rear-end collisions into the ego vehicle are currently not classified as at-fault"

追尾自车不算自车的过错——这意味着"车后方发生了什么"在这套 benchmark 里被系统性地不重视。这或许也解释了 Table 1 里一个反直觉的现象:只用前视相机的 LTF,能和配备了 LiDAR + 环视相机的 TransFuser 打平分数——benchmark 的设计本身没有充分奖励全向感知能力。

**值得质疑但论文没有讨论的一点**:场景过滤策略本身存在一个潜在的循环性(circularity)——用同一套 PDM Score 去筛选"哪些场景有挑战性",又用同一套 PDM Score 去评测最终表现。如果 PDM Score 本身有系统性盲点(比如上面提到的"追尾不算过错"),这个盲点会在筛选阶段被隐藏、而不是被暴露出来——因为筛选逻辑本身信任的正是这套可能有盲点的评分函数。

### 6. 是否能连 GitHub

**可以,而且是这周读过的论文里维护得最好、影响力最大的开源项目。** 我核实了 https://github.com/autonomousvision/navsim :1.1k stars、128 forks、最新版本 v2.2.2(2025 年 9 月发布),持续更新的 changelog,内含完整训练/评测脚本、leaderboard 提交工具、多个数据集 split、baseline agent(ConstantVelocity/MLP/TransFuser)实现、可视化教程 notebook,Apache 2.0 协议,并且和 CoRL 2025、NeurIPS 2024 两篇论文的引用要求绑定维护。

---

## English at-a-glance summary

### 图解读 Diagram walkthrough(文字版,无海报)

这篇笔记没有配单独的 SVG 海报——NAVSIM 的核心不是一张模型架构图,而是一条评测流水线,用文字梳理更清楚:

**真实传感器数据(单帧,8 相机 + LiDAR)** → **策略只被查询一次**,输出未来 4 秒的轨迹 → **LQR 控制器 + 运动学自行车模型**以 10Hz 把轨迹"执行"出来(背景车按真实录像走,不响应自车)→ 逐帧计算子分数:**NC(无碰撞)、DAC(区域合规)是硬性乘法惩罚**;**EP(进度)、TTC(防碰撞时间)、Comfort(舒适度)加权平均** → 汇总成 **PDM Score(PDMS)**,平均到整个场景。这条流水线,就是这周几乎每一篇论文报的"PDMS/EPDMS 91.7"之类数字背后真正发生的事。

### English at-a-glance table(中英对照)

| Aspect 方面 | Summary |
|---|---|
| **Problem 问题** | How to evaluate E2E driving policies? Open-loop (displacement error on real data) is easy but unrealistic; closed-loop (full simulation) is realistic but computationally expensive and simulators have a large domain gap to real data.<br>怎么评测端到端驾驶策略?开环(真实数据算位移误差)简单但不真实;闭环(完整仿真)真实但算力昂贵,且仿真器和真实数据存在domain gap。 |
| **Novelty 新意** | "Non-reactive simulation": query the policy only once at the scene's start, roll out the fixed trajectory over a simplified BEV abstraction for 4s — no full sensor simulator needed, yet still yields collision/comfort/progress metrics beyond displacement error.<br>"非反应式仿真":只在场景开始时查询一次策略,在简化BEV抽象上推演固定轨迹4秒——不需要完整传感器仿真器,却能算出碰撞/舒适度/进度指标,不只是位移误差。 |
| **Core idea 核心想法** | PDM Score = hard multiplicative penalty (collision NC, off-road DAC → score=0, zeroes the whole PDMS) × weighted average (progress EP, time-to-collision TTC, comfort). Hard constraints can't be offset by soft metrics.<br>PDM Score = 硬性乘法惩罚(碰撞NC、出界DAC→分数清零)× 加权平均(进度EP、防碰撞时间TTC、舒适度)。硬约束不能被软指标抵消。 |
| **Hardest part 最大难点** | Not the non-reactive concept itself, but proving the simplification doesn't distort results — required systematically correlating PDMS/OLS against true closed-loop CLS across 265 different planners in real nuPlan simulation.<br>难点不在非反应式概念本身,而在证明这个简化不会扭曲结果——需要在265个不同planner上系统性地对比PDMS/OLS和真实nuPlan闭环CLS的相关性。 |
| **Evidence 证据** | PDMS correlates better with CLS than OLS across all planner types (Fig.3-4). TransFuser (1 GPU, 1 day) reaches PDMS 84.0, matching/beating PARA-Drive (80 GPUs, 3 days, 84.0) and UniAD (83.4) — proving compute-heavy architectures aren't automatically better.<br>PDMS对所有planner类型都比OLS更好地关联CLS(图3-4)。TransFuser(1GPU,1天)PDMS达84.0,追平/反超需要80GPU训3天的PARA-Drive(84.0)和UniAD(83.4)——证明堆算力的架构不一定更好。 |
| **Assumptions/limitations 假设/局限** | Authors admit: high PDMS doesn't guarantee high CLS (no reactiveness/error accumulation modeled); rear-end collisions into the ego aren't classified as at-fault, undervaluing all-around perception. Unaddressed: the scenario-filtering strategy uses the same PDM Score it later evaluates with — a potential circularity.<br>作者承认:PDMS高不保证CLS高(不建模反应性/误差累积);追尾自车不算过错,低估了全向感知的价值。未讨论:场景过滤策略用的是之后评测时同一套PDM Score——存在潜在循环性。 |
| **Code / GitHub 代码开源情况** | **The most actively maintained project in this reading list.** github.com/autonomousvision/navsim: 1.1k stars, 128 forks, v2.2.2 (Sept 2025), full training/eval pipeline, leaderboard tools, baseline agents, tutorials.<br>**这份清单里维护最活跃的项目。** 1.1k星,128 forks,v2.2.2(2025年9月),完整训练/评测流水线,leaderboard工具,baseline agent,教程。 |

## 一个月后只记住 5 件事

① 非反应式仿真:只在初始帧查一次策略,之后轨迹固定用简化 BEV 推演 4 秒,用这个换取比开环更好、但比真闭环便宜得多的评测。

② PDM Score = 碰撞/出界的乘法硬惩罚(=0 清零)× (进度 + 防碰撞时间 + 舒适度)的加权平均——硬约束不能被软指标抵消,是设计的核心哲学。

③ 场景过滤很关键:原始数据大部分是无聊的直行/停车场景,过滤掉恒速 agent 就能拿高分的场景后,难度大幅提升(恒速 agent 从 79 分暴跌到 22 分)。

④ "简单打平复杂":TransFuser(1GPU/1天)追平/反超需要 80GPU/3天训练的 UniAD、PARA-Drive——评测设计比堆算力更重要。

⑤ 这是这周读的几乎所有论文(WA-JEPA/Drive-JEPA/DriveZero/SparseDriveV2/Qwen-Drive-1.0)引用的 PDMS/EPDMS 分数的真正出处——理解了这篇,之前那些论文报的数字才真正有意义。
