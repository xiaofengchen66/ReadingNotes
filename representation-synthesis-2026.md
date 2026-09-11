# Representation in End-to-End Autonomous Driving:从综述框架到 2026 年这批论文,再到我们能做什么

这篇不是某一篇论文的精读笔记,是一篇**综合分析**——按 [End-to-end AD Survey](notes/end-to-end-ad-survey.md)(Chen et al., TPAMI 2024)自己 §4.2 "Dependence on Visual Abstraction"(依赖视觉抽象/表征)这一节的架构搭骨架,把这周读的几篇 2026 年论文(WA-JEPA、Drive-JEPA、Qwen-Drive-1.0、DriveZero)一一填进去,看它们各自在回答综述当年提出的哪个具体问题,最后给出几个我们可以做的具体方向。

所有对综述原文的转述都对照过我们自己精读时提取的原文(见 [notes/end-to-end-ad-survey.md](notes/end-to-end-ad-survey.md) 第 7 节的原文摘录),不是凭印象转述。

---

## 1. 综述原有的框架:§4.2 讲了什么

综述把端到端驾驶系统的内部流程拆成两步:**先把状态编码成一个 latent 表征(representation),再从这个表征解码出驾驶策略**。原文点出了核心难点:

> "In urban driving, the input state...is much more diverse and high-dimensional compared to common policy learning benchmarks such as video games, which might lead to the misalignment between representations and necessary attention areas for policy making."

翻译:城市驾驶的输入状态比常见的策略学习基准(比如电子游戏)要多样和高维得多,这可能导致表征和"做决策真正需要关注的区域"不对齐。综述把这个大问题拆成两个子问题:

### 1.1 §4.2.1 表征设计(Representation Design)——"用什么形式存这个表征"

综述总结了当时(2023-2024)的主流答案:经典 CNN 骨干仍占主导(平移不变性 + 高效率);Transformer 骨干可扩展性强但当时还没被端到端驾驶广泛采用;驾驶专用的表征是 **BEV(Bird's-Eye-View,鸟瞰视图)**——把多传感器、多时间步信息融合进统一的 3D 空间;更进一步是**网格化 3D occupancy(占据栅格)**,能表达任意形状物体但计算量大。综述专门点出一个当时悬而未决的问题:

> "the most suitable formulation for end-to-end systems remains unvalidated."(什么样的表征形式最适合端到端系统,当时仍未有定论)

以及一个更尖锐的怀疑:

> "given the trends observed in several simple yet effective approaches with scaling up training resources, the ultimate necessity of explicit representations such as maps is uncertain."(考虑到"简单方法 + 堆算力"这个趋势,像地图这种显式表征到底是不是必需的,其实并不确定)

**这句怀疑,现在回头看,正是 2026 年这批论文里最激进的答案的伏笔**——见下文 Qwen-Drive-1.0。

### 1.2 §4.2.2 表征学习(Representation Learning)——"这个表征怎么学出来"

综述梳理了从"直接用现成分割模型的输出当表征"(早期方法,人为定义的瓶颈,容易丢有用信息),到"用预训练任务的中间特征做表征"(PPGeo 用运动预测+深度估计自监督学习,ViDAR 用点云预测做预训练任务)这条演化路线,最后给出一句明确的前瞻判断:

> "These works demonstrate that self-supervised representation learning from large-scale unlabeled data for policy learning is promising and worthy of future exploration."

翻译:这些工作证明了——用大规模无标注数据做自监督表征学习,再服务于策略学习,是一个有前景、值得未来探索的方向。**这句"未来展望",正是我们这周读的 WA-JEPA / Drive-JEPA 这条 JEPA 路线的直接延续。**

### 1.3 一个容易被忽略的边界:§4.2 和 §4.3 是分开写的

综述把"表征学习"(§4.2)和"世界模型/预测未来"(§4.3 World Modeling for Model-based RL)当成**两个独立的挑战**分别讨论——§4.2 关心"怎么把当前状态编码好",§4.3 关心"怎么预测未来会发生什么"。这个划分在 2023-2024 年是合理的,因为当时大多数方法确实是分开做这两件事(先编码,再单独训一个世界模型去预测未来)。

**这个边界在 2026 年这批论文里正在消失**——这是这篇分析文档最重要的一个观察,细节见下一节。

---

## 2. 2026 年这批论文,各自在回答哪个问题

### 2.1 表征设计的新答案:Qwen-Drive-1.0——"不造表征,读表征"

综述 §4.2.1 那句"explicit representations 到底是不是必需的,并不确定"在 [Qwen-Drive-1.0](notes/qwen-drive.md) 这里得到了一个明确答案:**不需要专门造一个驾驶用的表征,通用 VLM 自己的表征里已经隐含了足够的 3D 场景信息**。

> "It serves as a probe of the 3D information accessible from the shared representations and provides an explicit, inspectable interface to 3D scene structure."(BEV 感知头是一个探针,用来读取共享表征中可访问的 3D 信息,并提供一个显式、可检查的 3D 场景结构接口)

也就是说,BEV 表征没有消失,但它的角色从"主动理解世界的核心模块"(综述里 BEV 作为表征设计的主流选项)降级成了"从 VLM 表征里读取信息的探针(probe)"——这是对综述 §4.2.1 那个悬而未决问题一个具体的、可验证的回答:显式表征(BEV)仍然有用,但不是用来"理解"世界,而是用来"检验"通用表征里到底编码了什么。

### 2.2 表征学习的新答案:WA-JEPA / Drive-JEPA——自监督预测未来 latent,不重建像素

这正是综述 §4.2.2 那句"未来展望"的直接兑现。[Drive-JEPA](notes/drive-jepa.md) 把 V-JEPA(Video Joint Embedding Predictive Architecture)引入驾驶预训练,核心论证是:在 latent 空间预测(而非重建像素)会强迫模型学习"可预测的语义结构"(谁在移动、往哪走),而不是纠结于不重要的像素噪声。[WA-JEPA](notes/wa-jepa.md) 在此基础上更进一步,把 V-JEPA 原本"随机 mask 补全 + 确定性回归"的训练目标,针对性改造成"因果未来 mask + flow matching 生成式建模",专门为"这个表征要能拿去规划"这个目标服务。

### 2.3 §4.2 和 §4.3 边界消失的具体证据

综述原本把"表征学习"和"世界模型"分开讨论,但 WA-JEPA/Drive-JEPA 做的事情是**同一个训练目标同时完成了这两件事**:预测未来的 latent 表征,这个动作本身——

- 从 §4.2.2(表征学习)的角度看:是在学一个更好的表征,因为迫使模型编码"什么是可预测的语义"而不是像素噪声;
- 从 §4.3(世界建模)的角度看:是在学一个世界模型,因为输出就是对未来状态的预测。

综述写作时(2023-2024)这两件事是分开的两条技术路线(表征学习论文只管当下状态编码好不好;世界模型论文假设已经有一个好表征,再在这个表征上学转移动态)。JEPA 这套框架把它们**在同一个损失函数里统一**了——这是我认为综述框架需要更新的地方,值得在给乔老师的 presentation 里明确指出:不是"表征学习"和"世界模型"变得不重要了,而是**这两个曾经被当作独立问题的挑战,现在被同一个技术方案同时解决了**。

### 2.4 一个不完全落在 §4.2 框架里的答案:DriveZero——表征学习和策略学习要不要分开做

[DriveZero](notes/drivezero.md) 提供了另一个视角,不完全是"表征怎么设计/怎么学"的问题,而是**表征学习和策略学习该不该用同一套训练范式**。DriveZero 的核心论证:

> "The two models call for different learning recipes: perception must understand the world, and benefits from massive and diverse visual data; action must interact with it, and requires closed-loop feedback."

翻译:感知和行动需要完全不同的学习方案——感知靠海量多样视觉数据(适合用 DINOv3/SigLIP2/SAM/Depth Anything V2 这类冻结视觉基础模型蒸馏),行动需要闭环反馈(适合 RL,不适合在静态日志上做监督学习)。这个论证隐含地批评了综述 §4.2 把"表征"当成一个单一流程环节的默认假设——DriveZero 说的是:**感知表征的学习方式,和用这个表征去做决策的学习方式,本来就不该混在一起训**。这是这四篇里唯一明确质疑"表征应该是策略学习管线里的一个中间产物"这个默认设定的。

### 2.5 一个提醒:SparseDriveV2 不是感知表征问题,是动作空间的表征问题

[SparseDriveV2](notes/sparsedrivev2.md) 的"factorized trajectory vocabulary"(把轨迹分解成路径 × 速度两个正交维度)严格来说不属于综述 §4.2 讨论的范畴——它不是在问"怎么表征感知到的世界",而是在问"怎么表征输出的动作空间"。放在这里提一句是为了标注清楚:**"表征"这个词在端到端驾驶里至少有两层含义**——感知/世界状态的表征(§4.2 讨论的),和决策/动作空间的表征(SparseDriveV2 这类工作讨论的)——presentation 时如果要用"representation"这个词统摄全场,最好明确区分这两层,不然容易把两类完全不同的设计问题混为一谈。

---

## 3. 我们可以做的几件事

基于上面的分析,提几个具体的、有据可查的方向,按"离现有证据的距离"从近到远排:

**① 先把 ARTEMIS / WAM-Diff / CoWorld-VLA 这三篇 MoE 相关论文读完,再判断"表征层面的 MoE 路由"是不是真空。** 我们目前只从标题和 WA-JEPA 的引用列表确认了这三篇存在且用了 MoE,但具体是在决策层面做专家路由(比如给不同驾驶动作分专家),还是在表征层面做路由(比如给不同 driving regime 的 latent 表征分专家)——这个区别还没读过原文,不能先假设结论。这是最优先的一步,因为它直接决定了下面②③是不是还成立。

**② 如果①确认现有 MoE 论文都是"决策层面路由",而不是"表征层面路由",这就是一个具体、可论证的空档。** 论证链条:WA-JEPA/Drive-JEPA 证明了"用同一个 JEPA 目标统一学习表征和预测未来"这条路径有效;DriveZero 证明了"把表征学习和策略学习分开训"也有效且能突破模仿学习上限。两条路径目前都是**单一模型**在做表征学习这件事。如果 MoE 目前只用在决策/动作输出层(选专家给动作打分或生成动作),而没有人在"表征本身"这一层引入多专家(比如:路口场景、恶劣天气、施工区分别用不同的 JEPA encoder/predictor 处理,再路由),这就是一个具体的研究问题,而不是泛泛的"MoE 加自动驾驶"。

**③ DriveZero 的"感知/行动分离"思路 + WA-JEPA 的自监督表征学习,两者目前没人结合过。** DriveZero 用的感知骨干是 4 个现成视觉基础模型蒸馏出来的(DriveVFM),不是自监督在驾驶视频上学出来的;WA-JEPA/Drive-JEPA 的自监督表征目前是直接接一个规划头,没有像 DriveZero 那样把"表征学习"和"策略学习"彻底分成两个独立训练阶段。值得追问:**如果用 JEPA 式自监督预训练替换 DriveZero 里的 DriveVFM(4 模型蒸馏),再配合 DriveZero 的 RL 策略训练,是不是能同时拿到"自监督表征的可扩展性"和"RL 突破模仿学习上限"这两个优点?** 这个方向的风险在于:DriveZero 论证"感知用海量多样数据、行动用闭环反馈",而 JEPA 式预训练本质上也是"感知/表征"层面的自监督,理论上兼容,但具体怎么接口(表征维度、更新频率、训练顺序)需要实际设计。

**④ 表征的可解释性/可检查性,目前是个明确的空白。** 综述 §4.6 讨论 cost learning 时提到,规则化的决策方式天然带一定可解释性(能拆解出每条候选轨迹的具体扣分项);但 WA-JEPA/Drive-JEPA 这类 JEPA latent 表征完全是黑箱数字向量,没有任何"可检查接口"。Qwen-Drive-1.0 的 BEV 探针(probe)提供了一种局部解法——用一个轻量、可读的下游任务去检验表征里到底编码了什么语义。**能不能把"表征探针"这个思路系统化,做成一个通用的诊断框架,用统一的一套探针任务(比如:能不能读出车道拓扑?能不能读出障碍物运动趋势?能不能读出交通规则?)去评估任意一个 driving world model 的表征质量,而不是像 Qwen-Drive-1.0 这样只探测 3D 检测这一件事?** 这个方向偏方法论/评测,和我们已经读过的 RoboLab(用受控扰动轴系统评测策略泛化能力)是同一类"诊断性评测框架"思路,可以互相借鉴。

**⑤(风险更高,标注为开放猜想)表征层面要不要做 domain adaptation。** 综述 §4.9.3 讨论的 sim-to-real 主要是视觉层面(仿真画面 vs 真实画面长得不一样)。JEPA 式的自监督表征,理论上因为学的是"语义结构"而不是像素,可能天然对这类视觉域偏移更鲁棒——但这纯粹是一个假设,目前读过的论文都没有专门验证这一点(WA-JEPA/Drive-JEPA 都只在同一数据分布内训练评测)。这个方向目前没有直接证据支撑,列在这里是作为一个"值得问、但还没法回答"的问题,不建议现在就当成研究提案的核心,除非先找到相关的对照实验。

---

## 一句话总结

综述在 2023-2024 年把"表征怎么设计"“表征怎么学”“世界模型怎么预测未来”当成三个相对独立的挑战;2026 年这批论文的共同趋势是**把这三者融合**——Qwen-Drive-1.0 用现成 VLM 表征替代专门设计的驾驶表征,WA-JEPA/Drive-JEPA 用同一个自监督目标同时完成表征学习和未来预测,DriveZero 则反其道而行、把表征学习和策略学习彻底拆开训练。我们能做的增量工作,大概率不在"再造一个表征学习方法",而在**这几条已经被验证有效的路径之间,还没被系统探索过的组合方式**——尤其是"表征层面的 MoE 路由"和"表征的可解释性评测框架"这两个,目前看起来证据链最完整、风险最低。
