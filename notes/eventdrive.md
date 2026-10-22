# EventDrive: Event Cameras for Vision-Language Driving Intelligence

- **Venue / year:** arXiv 2606.18242, 2026 (CVPR 2026)
- **Authors:** Dongyue Lu, Rong Li, Ao Liang, Lingdong Kong, Wei Yin, Lai Xing Ng, Benoit R. Cottereau, Camille Simon Chane, Wei Tsang Ooi — 新加坡国立大学 + 香港科技大学(广州) + CNRS@CREATE + 地平线机器人 + A*STAR等
- **Link:** https://arxiv.org/abs/2606.18242 ([本地PDF副本](pdf/eventdrive.pdf))
- **Code:** 项目页面 github/EventDrive，数据集已发布于HuggingFace，本笔记未逐一核实代码仓库完整性
- **Input / 输出:** 事件流+同步RGB帧+语言指令 → 感知/理解/预测/规划四类任务的语言化输出(场景描述、grounding、运动状态、路点预测等)
- **精读说明(2026-10-22)：用户让我把本地下载的论文都精读一遍，这是优先级较高的一篇(驾驶专用、按传感器模态做真正MoE路由的独特案例)，已完整读过PDF正文。**

---

## TL;DR

**问题 →** 事件相机通过异步亮度变化感知世界，微秒级延迟、高动态范围、对运动模糊天然鲁棒，是驾驶场景(尤其模糊/眩光/快速运动)里对RGB帧的强力补充。但现有事件感知VLM仅限于通用感知任务(检测/分割/光流)，没有揭示事件感知如何贡献于完整驾驶环路里的推理和决策。

**Gap →** 需要一个统一框架，把事件流、RGB帧、语言监督整合起来，覆盖完整的自动驾驶栈(感知→理解→预测→规划)，而不是像此前工作那样只做孤立的感知任务或笼统的描述/问答。

**Novelty →** 提出EventDrive：首个统一事件流/RGB帧/语言监督、覆盖感知/理解/预测/规划四个核心维度的大规模基准(471,543条样本，17个子任务)；在此基础上提出EventDrive-VLM，引入Dynamic Horizon Encoding(动态horizon编码，对事件流做多时间分辨率体素化，用MoE门控动态选择最合适的时间尺度表示)+Event Q-Former Alignment(用结构化cross-attention把聚合后的事件特征对齐进LLM嵌入空间)+两阶段训练课程(先做事件-语言预对齐、再做全量指令微调)。

**核心洞察 →** 事件流在不同运动模式/场景下，最合适的时间分辨率是不一样的——快速运动需要细粒度短时间窗口保留高频动态细节，缓慢场景可以用粗粒度长时间窗口获得更稳定的聚合表示；用MoE动态选择最合适的时间尺度，比固定单一分辨率或简单的加权求和效果更好。

**方法/论证 →** 事件流体素化成多个不同时间分辨率(bin数B∈{20,50,100})的4D张量E_n(Eq.1)；每个"时间专家"网络处理一个分辨率的体素张量，输出编码特征F_n(Eq.2)；拼接各专家特征得到上下文表示，全局平均池化得到紧凑描述符，驱动带噪声的门控逻辑(Eq.3)，Top-k选择+softmax归一化后加权聚合各时间专家特征，得到最终事件表示F^e；Event Q-Former用一组可学习query token对F^e做cross-attention(Eq.4)，提取语言对齐、运动感知的紧凑嵌入，经投影层映射到LLM嵌入空间得到事件token，跟帧token、文本token拼接喂给LLM；两阶段训练：第一阶段冻结LLM和帧视觉塔，只训练事件编码器+Q-Former+投影层做语言建模式预对齐；第二阶段解冻LLM的transformer块连同事件通路一起全量微调(帧视觉编码器始终冻结)。

**结果 →** EventDrive基准上，EventDrive-VLM(事件+帧融合)在感知/理解/预测/规划四个任务族上全面超过纯事件模型(EventGPT-7B)和纯帧模型(Qwen2.5-VL-7B-Instruct等)；事件-帧融合模型在所有任务族上都提升鲁棒性，尤其增强grounding/关系推理和运动预测；消融显示多时间分辨率体素化(N=5比N=1好，但3个horizon后收益饱和)、MoE式动态聚合(比naive求和/加权求和好)、Event Q-Former对齐(比简单拼接/cross-attention好)都是各自环节里最优的设计；在Event-Chat数据集上做零样本迁移评测，证明EventDrive学到的表征具有跨数据集泛化性，不是过拟合特定标注风格。

**意义 →** 这是目前查到的、**驾驶专用、路由维度是"传感器模态/处理粒度"的真正MoE应用**，跟SAMoE-VLA(按场景)、ExpertAD(按功能子任务，仅限预测模块)、SemLT3D(按类别语义)、MoEActok(按技能)一起，构成了"专家该按什么维度划分"这个问题的又一个独立案例。

---

## 贡献列表(按重要性排序)

**① EventDrive基准：首个覆盖完整驾驶栈的事件-语言数据集**
整合M3ED(多机器人平台、高速高动态场景)、DSEC(立体事件相机、几何精确标注)、PKU-DAVIS-SOD(挑战性光照/运动条件)三个现有事件数据集，用Qwen3-VL半自动生成语言化标注，构建471,543条事件-帧-语言样本，覆盖感知(场景类型/能见度/交通灯/天气/道路状况/交通流，6个子任务)、理解(物体存在/外观/状态/ego关系/环境关系/grounding，6个子任务)、预测(速度变化/方向变化，2个子任务)、规划(速度意图/方向意图/路点规划，3个子任务)四大维度、17个子任务。专门设立了只含低光/运动模糊场景的hard split，用于针对性评估事件感知的优势场景。

**② Dynamic Horizon Encoding：用MoE动态选择时间分辨率**
现有方法通常用固定数量的时间bin把事件流体素化，这会在长曝光窗口里模糊掉快速运动的高频细节。EventDrive用多个并行的时间分辨率(短/中/长horizon)分别体素化同一段事件流，每个分辨率配一个专家网络编码，再用MoE门控(标准的噪声门控+Top-k+softmax机制)根据当前场景的全局特征，动态决定该更依赖哪个时间尺度的表示——快速运动场景会更依赖细粒度短时间窗口的专家，缓慢场景会更依赖粗粒度长时间窗口的专家。

**③ Event Q-Former Alignment：结构化cross-attention对齐事件表示和LLM空间**
不是简单地把事件token和帧token拼接(消融证明这样效果最差，因为忽略了模态不对称性、增加了不必要的计算量和序列长度)，而是用一组可学习的event query token，对聚合后的事件特征图做cross-attention，提取出最显著的时序和运动线索，映射成跟帧token/文本token对齐的紧凑嵌入——这个设计借鉴了BLIP-2式的Q-Former架构，但专门针对事件数据的运动感知特性做了适配。

**④ 两阶段训练课程：从事件-语言预对齐到全量指令微调**
第一阶段(事件-语言预适应)：LLM和帧视觉塔冻结，只让梯度流经事件通路(编码器+Q-Former+投影层)，用语言建模目标在配对的caption数据上训练，让事件编码器学会组织时序线索、产出跟LLM嵌入空间兼容的表示，而不改变预训练的帧语义；第二阶段(指令微调)：解冻LLM的transformer块，连同事件通路一起在全量caption+QA数据上微调，帧视觉编码器始终保持冻结。

---

## Novelty 分类判断

- **新问题定义**：算——首次提出覆盖感知-理解-预测-规划全流程的事件驱动驾驶推理基准，此前的事件VLM工作局限于孤立的感知任务(检测/分割/光流)或笼统的caption/QA。
- **新insight**：算——不同运动模式/场景需要不同时间分辨率的事件表示，用MoE动态选择比固定分辨率/简单聚合更好，有消融(Table 4的Multi-Horizon Voxelization和Dynamic Horizon Event Aggregation两组对比)支撑。
- **新方法**：算——Dynamic Horizon Encoding(多时间分辨率体素化+MoE门控)、Event Q-Former(结构化cross-attention对齐)、两阶段训练课程都是具体工程创新。
- **新理论**：没有。
- **新数据集**：算——EventDrive数据集本身(整合三个现有事件数据集、自动化语言标注pipeline生成471k样本)是新贡献。
- **新应用场景**：算，是"覆盖完整驾驶推理栈的事件驱动VLM"这个此前没有的应用场景组合。

综合结论：**新问题+新数据集+新方法组合类贡献**，这是目前驾驶专用论文里**唯一一篇按"传感器模态/处理粒度"做真正MoE路由**的论文，为"专家该按什么维度划分"这个设计问题补上了一个之前案例库里没有的选项。

---

## 中文精读

### 1. 解决了什么问题

事件相机具有微秒级延迟、高动态范围、对运动模糊天然鲁棒等独特优势，这些特性在自动驾驶这类安全攸关场景里特别有价值，尤其是传统帧式感知在强光/低光/快速运动下容易失效的情况。但是此前把事件感知跟视觉语言模型结合的工作，大多局限在生成式的caption/QA或者孤立的感知子任务(检测/分割)，没有探索事件感知在完整自动驾驶推理链路(从感知到最终规划决策)里到底能贡献什么。论文要回答：**事件流信息在驾驶的每一个推理阶段(感知/理解/预测/规划)分别能带来什么具体收益，又该怎么把这种异步、高频的信号跟帧式语义信息、语言推理统一起来？**

### 2. 新意在哪里

核心新意有两层：一是**评测设计的新意**——把事件感知的价值评估范围从孤立的感知任务扩展到完整的驾驶推理链路，用统一的语言化QA框架，让感知/理解/预测/规划四个阶段的事件贡献可以被系统比较；二是**架构设计的新意**——发现"不同时间尺度的事件表示适合不同运动场景"这个问题，并用MoE(而不是固定分辨率或简单融合)来动态解决，这是这批论文里少数几篇把MoE用在"传感器时间粒度"这个维度上的工作。

### 3. Idea 具体落在哪里

**数据构建(Fig.2，§3.2)：** 感知标注用Qwen3-VL对短时间窗口生成全局场景描述，再拆解成关于能见度/交通流/天气等6个属性的平衡问答对；理解标注基于DSEC的真实边界框，让Qwen3-VL生成5类物体属性描述，转换成视觉问答和grounding任务；预测标注用M3ED提供的自车位姿轨迹监督，把3D物体框投影进自车坐标系，提取物体轨迹，转换成速度/方向变化的自然语言描述；规划标注同样基于M3ED轨迹，按自车中心坐标对齐，构建过去/未来轨迹，并做阈值式行为标注(加速/减速/变道等)。

**Dynamic Horizon Event Encoder(§4.1)：** 给定事件流ε={e_k}，标准体素化用固定bin数B把事件映射进4D张量(Eq.1)；EventDrive用多个不同的bin数{b_n}构建多个体素张量E_n，每个专家网络σ处理一个张量输出编码特征F_n(Eq.2，H×W×d维)；拼接N个专家特征得到上下文表示F_c，全局平均池化得到紧凑描述符f_c∈R^(Nd)；门控逻辑z=W_g f_c + Softplus(ε⊙(W_noise f_c))(Eq.3，ε~N(0,1)引入受控随机性鼓励专家多样性)，保留top-k个logits做softmax得到权重α_n，最终事件表示F^e=Σα_n F_n。

**Event Q-Former Alignment(§4.2，Eq.4)：** 可学习的event query token q^e对展平后的事件特征图f^e做cross-attention：z^e=softmax((q^e W_Q)(f^e W_K)^T/√d)(f^e W_V)，输出紧凑表示z^e经投影层映射成跟帧token h^f、文本token h^t对齐的事件token h^e，三者拼接作为LLM解码器的输入。

**训练(§4.3)：** 第一阶段事件-语言预适应，LLM和帧视觉编码器冻结，只训练事件编码器/Q-Former/投影层做语言建模式监督，让事件通路学会组织时序线索而不破坏预训练的帧语义；第二阶段指令微调，解冻LLM的transformer块连同事件通路在全量caption+QA数据上微调，帧视觉编码器始终冻结。

**消融(§5.3，Table 4)：** 多时间分辨率体素化从N=1到N=5性能持续提升，但N=3之后收益饱和、计算成本线性增长，作者因此选择N=3作为效率-精度的最优平衡；动态horizon聚合对比(Add朴素求和 vs Wt.sum加权求和 vs MoE)，MoE式动态聚合(只激活得分最高的专家)效果最好，证明"选择性激活单一专家"比"模糊混合"更能保留horizon专属信息；多模态对齐对比(Concat拼接 vs Attention简单注意力 vs Q-Former)，Q-Former效果最好且计算成本更低，因为可学习query只提取最显著的运动模式，而不是对完整时空图做attention。

### 4. 最大难点在哪里

不在"事件相机对驾驶有用"这个大方向判断上(这是已有共识)，难点在于**怎么设计一个既能保留事件数据固有的时间稀疏性和异步性、又能跟帧式语义信息及语言推理顺畅融合的架构**——如果直接把事件流简单离散化成固定分辨率的图像式张量，会损失事件数据最核心的高时间分辨率优势(论文明确指出固定bin大小的体素化"压缩了长曝光窗口，丢失了对运动推理至关重要的高频细节")；Dynamic Horizon Encoding+Event Q-Former这套组合设计，是解决这个"既要保留异步特性、又要兼容标准LLM输入格式"矛盾的核心技巧。

### 5. 局限 / 对我项目的相关性

**数据主要基于三个现有事件数据集重新标注，不是全新采集**，标注本身是用Qwen3-VL半自动生成的，论文没有详细讨论VLM生成标注固有的噪声/幻觉风险的人工校验流程。

**MoE门控的"专家"数量和具体时间分辨率(bin数B∈{20,50,100})是人工预先设定的**，不是自适应学出来的，不同传感器配置/帧率下这组超参数是否需要重新调整没有讨论。

**没有在真实车辆上做闭环评测**，所有评测都是开环的QA/检测/轨迹预测任务，事件相机在真实驾驶场景里的硬件部署、功耗、延迟特性都没有涉及。

**对我项目的相关性：这是目前查到的、驾驶场景里"专家按传感器模态/处理粒度划分"的唯一真实MoE案例**，而且明确覆盖了感知-理解-预测-规划四个阶段的评测——不过需要注意的是，MoE路由机制本身只用在事件编码这一个环节(选择时间分辨率)，不是四个阶段各自独立做场景条件路由，所以跟"全程MoE"(三段都路由)这个理想状态依然有距离，但"按时间尺度/处理粒度做专家划分"这个新维度，给"专家该怎么分工"这个问题增加了SAMoE-VLA(场景)、ExpertAD(功能子任务)、SemLT3D(类别语义)、MoEActok(技能)之外的第五种选项。

### 6. 是否能连 GitHub

项目页面提供了代码仓库链接(github/EventDrive)和数据集链接(HuggingFace)，撰写本笔记时未逐一核实具体内容的完整性。

---

## 假设、局限与质疑点

- **时间分辨率的专家数量(论文选N=3)是基于计算成本-精度权衡的经验选择**，不是理论推导出的最优值，不同数据集/场景分布下这个数字是否需要重新调整没有讨论。
- **Event Q-Former的query token数量**(论文没有在正文明确给出具体数值N_q)和这个超参数对性能的影响，本笔记未在已读页面里找到对应的消融分析。
- **Event-Chat数据集上的零样本迁移评测**(Table 3)只报告了Detailed Captioning/Complex Reasoning/VQA三个粗粒度指标，没有像EventDrive基准那样细分到感知/理解/预测/规划四个维度，迁移验证的粒度不如主要实验部分细致。

---

## 一个月后记住 5 件事

1. **核心贡献：首个覆盖感知-理解-预测-规划完整驾驶推理栈的事件-语言数据集**(471,543条样本、17个子任务)，整合M3ED/DSEC/PKU-DAVIS-SOD三个现有事件数据集。
2. **Dynamic Horizon Encoding：对事件流做多时间分辨率体素化，用MoE门控动态选择最合适的时间尺度**——快速运动用细粒度短窗口、缓慢场景用粗粒度长窗口。
3. **Event Q-Former用结构化cross-attention对齐事件特征和LLM嵌入空间**，比简单拼接或普通注意力效果更好、计算成本更低。
4. **消融证明"选择性激活单一时间专家"(MoE)比"模糊混合多个时间尺度"(加权求和)效果更好**，horizon数N=3是效率-精度最优平衡点。
5. **这是驾驶场景里路由维度是"传感器模态/处理粒度"的真正MoE案例**，为"专家该怎么分工"这个问题增加了一个新的划分维度选项。

---

## 关键原文摘录(中英对照)

**① 核心问题定位**
> "Despite this potential, events for driving research remain fragmented. Most existing efforts focus on upstream supervised tasks... Very few works explore high-level reasoning and decision-making that a complete driving stack requires."

中译：尽管有这样的潜力，事件感知在驾驶研究中的应用依然是碎片化的。大多数现有工作聚焦于上游的监督式任务……很少有工作探索完整驾驶栈所需要的高层推理和决策能力。
解读：这是全文的核心问题诊断，presentation开场直接用这句话能把贡献的意义讲清楚。

**② 动态horizon编码的设计动机**
> "Conventional voxelization applies a fixed number of temporal bins and thus compresses long exposure windows while blurring fast motion, losing the high-frequency detail essential for motion reasoning. To address this issue, we propose a dynamic horizon encoding strategy that adapts to varying time scales and scene dynamics."

中译：传统的体素化方法用固定数量的时间bin，因此会压缩长曝光窗口、模糊快速运动，丢失对运动推理至关重要的高频细节。为解决这个问题，我们提出了一种能适应不同时间尺度和场景动态的动态horizon编码策略。
解读：这是Dynamic Horizon Encoding这个核心机制的设计动机，直接解释了为什么固定分辨率不够、为什么需要MoE式的动态选择。

---

## English at-a-glance summary

| Aspect | Summary |
|---|---|
| **Problem** | Event cameras offer strong complementary advantages for driving, but existing event-aware VLMs are limited to generic perception tasks and don't reveal how events contribute across the full driving loop.<br>事件相机对驾驶有很强的互补优势，但现有事件感知VLM局限于通用感知任务，没有揭示事件如何贡献于完整驾驶环路。 |
| **Novelty** | EventDrive: the first benchmark unifying event streams, frames, and language across perception/understanding/prediction/planning; EventDrive-VLM adds Dynamic Horizon Encoding (MoE over temporal resolutions) and an Event Q-Former.<br>EventDrive：首个统一事件流/帧/语言、覆盖感知/理解/预测/规划的基准；EventDrive-VLM加入动态horizon编码(对时间分辨率做MoE)和Event Q-Former。 |
| **Core idea** | Different motion patterns/scenes need different event temporal resolutions; an MoE gate should dynamically select the right scale rather than using a fixed resolution or naive blending.<br>不同运动模式/场景需要不同的事件时间分辨率；MoE门控应该动态选择合适的尺度，而不是用固定分辨率或简单混合。 |
| **Hardest part** | Designing an architecture that preserves events' native temporal sparsity/asynchrony while still integrating cleanly with frame semantics and LLM-based language reasoning.<br>设计一个既保留事件数据原生时间稀疏性/异步性、又能跟帧语义和基于LLM的语言推理顺畅整合的架构。 |
| **Evidence** | EventDrive-VLM (event+frame fusion) outperforms both event-only (EventGPT-7B) and frame-only (Qwen2.5-VL-7B) models across all four task families; ablations confirm MoE-based horizon selection and Q-Former alignment are each individually optimal.<br>事件+帧融合模型在四个任务族上全面超过纯事件和纯帧模型；消融证明MoE式horizon选择和Q-Former对齐各自都是最优设计。 |
| **Limitation** | Annotations are semi-automatically generated by Qwen3-VL without detailed human verification discussion; temporal resolution hyperparameters are manually fixed; no closed-loop real-vehicle evaluation.<br>标注由Qwen3-VL半自动生成，没有详细讨论人工校验流程；时间分辨率超参数是人工固定的；没有真实车辆闭环评测。 |
| **Code / GitHub** | Project page and dataset released; exact repository completeness not individually verified at note-writing time.<br>项目页面和数据集已发布；具体仓库完整性撰写本笔记时未逐一核实。 |

---

## 本文提到的词

| 术语 | 首次出现位置 |
|------|------------|
| Dynamic Horizon Encoding(动态horizon编码) | Abstract |
| Event Q-Former Alignment | Abstract |
| Multi-horizon voxelization(多时间分辨率体素化) | §4.1 |
| EventDrive-VLM | §4 |
| Hard split(低光/运动模糊专用测试集) | §3.3 |
