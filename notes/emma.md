# EMMA: End-to-End Multimodal Model for Autonomous Driving

- **Venue / year:** arXiv 2410.23262(v1: 2024-10,后续修订至 v3),Waymo
- **Authors:** Jyh-Jing Hwang, Runsheng Xu, Hubert Lin, Wei-Chih Hung, Jingwei Ji, Kristy Choi, Di Huang, Tong He, Paul Covington, Benjamin Sapp, Yin Zhou, James Guo, Dragomir Anguelov, Mingxing Tan
- **Link:** https://arxiv.org/abs/2410.23262
- **来源说明:** 用户先转述了论文对三大任务的描述(Motion Planning / Perception / Long-tail Reasoning),经 arXiv API 核实论文真实存在后,又通过 arXiv HTML 全文检索核对了 CoT 具体结构、三条局限性原文、2.1 节公式化表述、主要量化结果、贡献声明,逐项核实,不是转述二手资料。
- **paper_matrix.csv id:** 无
- **Code:** **未开源**——全文检索未发现任何 GitHub/HuggingFace 链接或权重发布声明,是这份阅读清单里少数没有可复现代码的论文之一(和 DriveVLM、DriveZero、DriveMLM 一样是纯技术报告性质)。
- **Input / 输入:** 环视摄像头视频(V,不含 LiDAR/雷达)+ 高级意图指令(T_intent,来自导航路由,如"直行""左转""右转")+ 历史自车状态(T_ego,BEV 坐标点序列,纯文本表示,可扩展含速度/加速度)
- **Output / 输出:** 未来轨迹路点(O_trajectory,BEV 坐标,纯文本)+(联合训练的其他任务)3D 检测框、道路图元素(车道线/路口拓扑)、场景理解文本回答

## ⚠ 一处需要修正的说法

用户最初转述"EMMA 的三大任务"时,把"Long-tail Reasoning"列为和 Motion Planning、Perception 并列的第三个正式任务。**经全文核实,论文里并没有把这个称为一个独立训练任务**——第 3.6 节展示的是定性可视化例子(比如没专门训练过也会主动避让松鼠),论文自己的措辞是"Generalizability"(泛化能力),属于观察到的**涌现能力**,不是一个有独立 loss/benchmark 支撑的训练目标。第 1 节列出的四条贡献声明原文分别是 motion planning、perception(3D detection/road graph/scene understanding)、generalist(联合多任务生成)、以及"reason and make decisions in complex, long-tail driving scenarios"——第四条确实提到了 long-tail,但表述为"能力"而非独立"任务",presentation 时这个区分值得讲清楚。

## TL;DR 浓缩版

**问题** → 端到端自动驾驶要怎么设计,才能既做好核心的轨迹规划,又能像人类司机一样利用导航意图、保持驾驶连贯性、还能应对训练数据里没见过的长尾场景?

**Gap** → 已有端到端方法大多是为驾驶任务专门设计的窄域网络,即使用了 VLM,也往往只是把语言模型当成感知或推理的辅助模块,没有真正把一个通用多模态大模型(MLLM)当成系统的第一公民(first-class citizen)去直接驱动规划、感知、问答等所有任务。

**Novelty** → 提出"驾驶即视觉问答"的统一范式——直接微调 Gemini,把轨迹规划、3D 检测、道路图提取、场景理解全部表述成同一个 MLLM 的文本生成任务,规划过程还引入四段式思维链(CoT:场景描述→关键物体→行为描述→元决策)。

**核心洞察** → 一个足够强的通用多模态大模型,只需要用文本统一表示各种输入输出(轨迹坐标、检测框、场景问答全都写成文字),就能同时胜任多个原本需要专门网络分别处理的驾驶任务,而且额外获得了从海量通用预训练里迁移来的常识推理能力,用于应对长尾场景。

**方法/论证** → 用自监督的方式微调 Gemini:输入环视视频 + 高级意图 + 历史自车轨迹,输出公式化为 O_trajectory = G(T_intent, T_ego, V);标注三大性质——自监督(标签就是后续真实发生的自车轨迹,不需要额外人工标注)、纯摄像头(不需要 LiDAR/雷达)、免高精地图(只需要导航级别的路由信息)。感知任务(3D 检测/道路图/场景理解)作为联合训练目标共训进同一个模型。

**结果** → nuScenes 规划 SOTA(平均 L2 误差 0.29m,比此前最好的 0.33m 提升 12.1%);内部大规模基准(WOMD 量级)上,带 CoT 的 EMMA+ 在 5 秒时域 L2 误差 0.543m,比此前 SOTA 提升 13.5%;CoT 消融显示比标准端到端规划提升 6.7%;3D 检测(WOD)上,同召回率下精度提升 16.3%,同精度下召回率提升 5.5%。

**意义** → 证明了"把驾驶所有任务塞进一个通用 MLLM、用文本统一表示"这条路径是可行的,而且规划、感知能同时受益、互相促进(联合训练不是简单的多任务权衡,而是有正向迁移)。但论文自己在附录坦诚承认:纯视觉(不融合 LiDAR/雷达)、需要昂贵闭环仿真、计算需求高于传统模型,这三条限制目前都没有具体解决方案,只写了"未来工作会解决"这一句话,没有展开。

## 中文精读

### 1. 解决了什么问题

自动驾驶系统要不要用一个统一的大模型去处理所有事情,还是继续用分离的专门模块(感知网络+规划网络+语言推理模块各自独立)?EMMA 的立场很明确——直接在第 1 节写"we propose to develop an autonomous driving system in which the MLLM is a first class citizen",把 Gemini 这样的通用多模态大模型当成系统的核心驱动力,而不是外挂的辅助模块。

### 2. Idea 具体落在哪里:三个任务分别怎么做

**① Motion Planning(核心任务)。** 公式化为 O_trajectory = G(T_intent, T_ego, V)——所有输入输出都用纯文本表示(坐标写成浮点数文字,不用专门的 token)。三个性质:
- **自监督**:唯一需要的监督信号是自车未来的真实位置,这个位置是行车记录自带的事实(GPS/IMU/定位系统直接读出,或视觉里程计反推),不需要人工标注——这和 3D 检测/道路图任务需要人工画框/描线完全不同,是这套方法能大规模扩展训练数据、不受标注人力限制的关键原因。本质上和语言模型"预测下一个词"是同构的训练范式,只是把"词"换成了"坐标点",这也是它能自然套用 Gemini 现成微调框架的原因。但这也意味着它的性质是纯模仿学习(imitation learning)——学习目标就是复现录像里人类司机的行为,理论上限是人类驾驶水平,不会自动超越示范(这一点上 DriveZero 用闭环 RL 去突破了这个天花板,做到 95.3 PDMS > 人类 94.8;EMMA 目前的规划任务设定里没有类似的 RL 后训练)。
- **纯摄像头**:不需要 LiDAR/雷达,只用环视相机——这是纯视觉 VLM 路线的共同选择(和 DriveVLM、Qwen-Drive-1.0、DriveZero、Drive-JEPA、WA-JEPA 一致),背后的深层原因是摄像头能蹭上互联网规模的图文预训练红利,LiDAR 没有对应的预训练生态可用(详见 [sensor-fusion-qa.md](../sensor-fusion-qa.md))。
- **免高精地图**:只需要导航级别的路由信息(比如 Google Maps 的"直行/左转"),不需要传统意义上的车道级高精地图。

规划前还加了**四段式思维链**(第 2.2 节):R1 场景描述(天气/时间/交通状况/路况)→ R2 关键物体(要求给出精确 3D/BEV 坐标,比如"行人在 [9.01, 3.22]")→ R3 行为描述(评估已识别关键物体的状态和意图)→ R4 元驾驶决策(12 类高层驾驶决策,总结给定观察下的驾驶计划),推理文本(rationale)在轨迹之前生成,轨迹是条件在推理之上的,不是并行输出头。这个结构和 DriveVLM 的"场景描述→场景分析→分层规划"几乎同构,只是拆分粒度更细——这不是论文自己声称的首创对比,是我读过两篇后自己做的结构比较,presentation 时如果要讲"EMMA 首创了驾驶 CoT",需要注意这个时间线和结构相似性。

**② Perception(感知)。** 三个子任务全部表述成文本生成:3D 物体检测("detect every object in 3D",输出用两位小数的浮点数写出 7D 框);道路图提取(用文本编码的有序折线表示路点);场景理解(比如用 prompt "is the road ahead temporarily blocked?" 做临时封路检测这类问答)。这三者和规划任务共训在同一个模型里,不是外挂的独立网络——这一点是这份阅读清单里比较独特的设计,只有 Qwen-Drive-1.0 走了类似方向(但机制不同:Qwen-Drive-1.0 论证"VLM 表征里已经隐含足够的 3D 理解,不需要专门训,直接探针读出就够",EMMA 选择的是"都放进同一个模型里训")。

**③ Generalizability(不是正式任务,是观察到的能力)。** 第 3.6 节用可视化例子展示模型能应对没有专门训练过的稀有场景(比如没见过某种动物也能识别出是需要避让的障碍物),论文将这归因于 Gemini 预训练带来的通用常识迁移。

### 3. 最大难点在哪里

不在把各任务表述成文本生成这个想法本身(这个想法本身在 DriveGPT4、DriveLM 这批前作里已经出现过),难点在于**怎么让规划、感知、问答这几个形式和难度都不同的任务在同一个模型里联合训练还能互相促进,而不是互相干扰**。论文用 CoT 消融(+6.7%)和感知任务的正向结果说明这个联合训练确实起作用,但没有详细拆解"为什么感知任务的梯度不会干扰规划任务"这个训练动态层面的问题——这是我读完全文后觉得论文论证不够深入的地方,值得存疑。

### 4. 哪些内容可以略过

具体 CoT 12 类元驾驶决策的完整枚举列表——记住"这是一个离散的高层动作分类,类似 DriveVLM 的 meta-action"就够了。

3D 检测的具体框回归损失函数设计——这是标准检测任务的常规工程细节。

### 5. 假设、局限、值得质疑的地方

**论文附录 A.5 自己承认的三条局限(原文):**

> "it faces challenges for real-world deployment due to: (1) limitations in 3D spatial reasoning due to its inability to fuse camera inputs with LiDAR or radar, (2) the need for realistic and computationally expensive sensor simulation to power its closed-loop evaluation, and (3) the increased computational requirements relative to conventional models. We plan to better understand and address such challenges in future work."

**这三条论文完全没有展开,经过两次全文检索确认:** 没有提出任何具体的 LiDAR/雷达融合方案或架构思路;全文没有一处拿 EMMA 和 Waymo 真实上路的 Waymo Driver 量产车队做对比(虽然作者都是 Waymo 的人,但完全没有借用"我们公司真车队已经解决了"这种说法来给局限找补);没有任何实验形式(哪怕是辅助监督)用到过 LiDAR,2.1 节明确写"Camera-only: the only sensor input required is surround-view cameras"。这三条局限详细的跨论文对比分析(和 DriveVLM-Dual/Alpamayo-R1/Qwen-Drive-1.0 的计算优化对比、和 NAVSIM/WA-JEPA/DriveZero 的闭环仿真应对方式对比、和 DriveMLM/ARTEMIS 的 LiDAR 融合方式对比)见 [sensor-fusion-qa.md](../sensor-fusion-qa.md) 第 1 节。

**值得质疑的点:** 论文强调"self-supervised"这个性质,但严格说这是纯模仿学习(imitation learning)的自监督版本,不是真正意义上不需要任何驾驶示范的自监督——它依然完全依赖大量人类司机的真实行车记录当"标准答案",只是标注环节被省掉了(标签是行车记录自带的,不需要人工画)。这和"不需要人类监督"是两回事,论文的用词可能会让读者误以为这是一种更弱监督的方法,实际上它对人类驾驶数据的依赖程度和传统模仿学习完全一样,只是标注成本(而非数据依赖）被大幅降低。

### 6. 是否能连 GitHub

**不能。** 全文检索没有发现任何 GitHub/HuggingFace 链接或权重发布声明,和 DriveVLM、DriveZero、DriveMLM 一样,是纯技术报告性质,没有可复现代码。

## Novelty 分类判断

- **新问题定义**:部分算——"把驾驶所有任务统一表述成一个通用 MLLM 的文本生成问题"这个具体框架是这篇论文明确提出并系统化的,但"用 VLM/LLM 做驾驶决策"这个大方向本身不是首创(DriveGPT4、DriveLM、DriveVLM 都是前作)。
- **新 insight**:算——"规划、感知、问答联合训练能互相促进而非互相牺牲"这个论证有具体的消融/结果数据支撑(CoT +6.7%,联合训练下感知和规划都提升)。
- **新方法**:部分算——四段式 CoT(R1-R4)结构和 DriveVLM 的三段式 CoT 高度相似,更像是改进组合(细化拆分)而非全新方法;"驾驶即 VQA"这个统一表述方式的具体工程实现是这篇论文自己的贡献。
- **新理论**:没有。
- **新数据集**:没有,用的是 nuScenes、WOD、WOMD 等已有基准,以及自己的内部数据。
- **新应用场景**:不算,这是方法论文。

综合结论:**改进组合类贡献**,核心价值在于系统性地把"驾驶即 VQA"这个思路落地成一个能同时处理规划+感知+问答的统一 MLLM,并用扎实的量化结果证明联合训练有正向收益,但和同期的 DriveVLM 在 CoT 结构设计上有明显的相似性,不宜说成从零首创。

## 关键原文摘录

1. 核心定位(§1): "we propose to develop an autonomous driving system in which the MLLM is a first class citizen"

2. 公式化表述(§2.1): "Otrajectory = G(Tintent, Tego, V)"，"the only required supervision is the future locations of the ego vehicle. No dedicated human labels are needed."

3. CoT 四段结构(§2.2): R1 场景描述、R2 关键物体("pedestrian at [9.01, 3.22], vehicle at [11.58, 0.35]")、R3 行为描述、R4 元驾驶决策(12 类)

4. 局限性(附录 A.5): "it faces challenges for real-world deployment due to: (1) limitations in 3D spatial reasoning due to its inability to fuse camera inputs with LiDAR or radar, (2) the need for realistic and computationally expensive sensor simulation to power its closed-loop evaluation, and (3) the increased computational requirements relative to conventional models."

5. 结果证据(§3.2-3.4): nuScenes 平均 L2 误差 0.29m(SOTA,+12.1%);内部基准 EMMA+ 5秒 L2 误差 0.543m(+13.5%);CoT 消融 +6.7%;3D 检测同召回率下精度 +16.3%。

## English at-a-glance summary

| Aspect | Summary |
|---|---|
| **Problem** | Should driving be handled by separate specialized modules, or unified inside one general-purpose MLLM? EMMA argues for making the MLLM (Gemini) a first-class citizen driving the whole system.<br>驾驶该用分离的专门模块,还是统一进一个通用MLLM?EMMA主张把MLLM(Gemini)当成驱动整个系统的第一公民。 |
| **⚠ Correction** | "Long-tail Reasoning" is NOT a formally named third task alongside Motion Planning/Perception — the paper frames it as "Generalizability," an emergent capability shown via qualitative visualization (§3.6), not a trained objective with its own benchmark.<br>"长尾推理"不是和运动规划/感知并列的正式第三任务——论文称之为"泛化能力",是定性可视化展示的涌现能力,不是有独立训练目标和基准的正式任务。 |
| **Novelty** | Unifies planning, 3D detection, road graph, and scene QA as one MLLM's text-generation tasks; adds a 4-stage CoT (scene description → critical objects → behavior → meta decision) before trajectory output.<br>把规划、3D检测、道路图、场景问答统一为同一个MLLM的文本生成任务;轨迹输出前加入四段式CoT(场景描述→关键物体→行为→元决策)。 |
| **Core idea** | Otrajectory = G(Tintent, Tego, V): self-supervised (label = actual future ego position, free from logs, no human annotation), camera-only, HD-map-free.<br>Otrajectory = G(Tintent, Tego, V):自监督(标签=真实未来自车位置,行车记录自带,无需人工标注)、纯摄像头、免高精地图。 |
| **Hardest part** | Not the text-unification idea itself (prior work did this too), but making planning/perception/QA jointly trained tasks reinforce rather than interfere with each other — the paper shows it works (+6.7% from CoT) but doesn't deeply explain the training dynamics behind why.<br>难点不在文本统一化想法本身(前作已有),而在让规划/感知/问答联合训练互相促进而非干扰——论文证明了有效(CoT+6.7%)但没深入解释背后的训练动态原因。 |
| **Evidence** | nuScenes planning SOTA (0.29m L2, +12.1%); internal WOMD-scale benchmark +13.5%; CoT ablation +6.7%; WOD 3D detection +16.3% precision at same recall.<br>nuScenes规划SOTA(0.29m L2,+12.1%);内部WOMD级基准+13.5%;CoT消融+6.7%;WOD 3D检测同召回率精度+16.3%。 |
| **Assumptions/limitations** | Self-authored: no LiDAR/radar fusion, expensive closed-loop sensor sim needed, higher compute than conventional models — all stated but NOT addressed (verified via two full-text searches: no proposed solution, no comparison to Waymo's real production fleet, zero LiDAR use anywhere). Also: "self-supervised" still fully depends on human driving logs — only the annotation step is removed, not the human-demonstration dependency itself.<br>作者自曝:无LiDAR/雷达融合、需要昂贵闭环传感器仿真、比传统模型计算需求更高——都只是陈述,未展开解决方案(两次全文检索确认:无具体方案、未与Waymo真实量产车队对比、全程零LiDAR使用)。另外:"自监督"依然完全依赖人类行车记录——省掉的只是标注环节,不是对人类示范的依赖本身。 |
| **Code / GitHub** | **Not released** — no GitHub/HuggingFace links or weight releases found in the full text.<br>**未开源**——全文未发现任何GitHub/HuggingFace链接或权重发布。 |

## 一个月后只记住 5 件事

① EMMA 把驾驶的规划、3D检测、道路图提取、场景问答全部统一表述成同一个 Gemini 微调模型的文本生成任务——"驾驶即视觉问答"。

② "Long-tail Reasoning"不是正式的第三个任务,是论文称为"Generalizability"的涌现能力(定性展示,无独立训练目标),presentation 时不要说成三个并列训练任务。

③ 规划任务是"自监督"的,因为标签(未来自车位置)是行车记录自带的事实,不需要人工标注——但这依然是纯模仿学习,依然完全依赖人类驾驶数据,只是省掉了标注成本,理论上限还是人类水平。

④ 论文自曝三条局限(无LiDAR融合、闭环仿真昂贵、计算需求高),但全文检索确认完全没有展开解决方案,也没有拿 Waymo 真实量产车队做对比。

⑤ 没有开源代码/权重,四段式 CoT(R1-R4)结构和 DriveVLM 的三段式 CoT 高度相似,presentation 时不宜说成首创。
