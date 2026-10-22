# The Blind Spot of Adaptation: Quantifying and Mitigating Forgetting in Fine-tuned Driving Models

- **Venue / year:** arXiv 2604.04857, 2026 (CVPR 2026)
- **Authors:** Runhao Mao, Hanshi Wang, Yixiang Yang, Qianli Ma, Jingmeng Zhou, Zhipeng Zhang — AutoLab, Shanghai Jiao Tong University
- **Link:** https://arxiv.org/abs/2604.04857 ([本地PDF副本](pdf/blind-spot-adaptation.pdf))
- **Code:** 数据集+模型已发布于 FidelityDrivingBench(论文abstract里给了链接)
- **Input / 输出:** 驾驶场景图像+自然语言问题(Scene Description/Traffic-QA/Noteworthy Objects' Perception三类任务) → 自然语言回答
- **精读说明(2026-10-21)：用户让我判断哪些论文最像朱子欣"全程MoE+蒸馏别的MoE当冻结专家训练轻量路由"这个idea，这篇已完整读过PDF正文。结论先说在前面：这篇**不是**"蒸馏别的模型当专家"这条机制，是"冻结基座+场景条件路由到LoRA专家委员会"——跟PerceptDrive/SAMoE-VLA在"冻结+轻量路由"这个结构上相似，但专家本身是训练出来的、不是蒸馏自外部模型，之前摘要级笔记里把这篇和"路由漂移"问题放一起讨论是合理的，但不要误以为它验证了"蒸馏别的MoE"这半个机制。**

---

## TL;DR

**问题 →** 把VLM整合进自动驾驶本该靠预训练世界知识解决长尾场景，但微调VLM去适配驾驶专用数据的过程，恰恰会侵蚀这份宝贵的预训练世界知识——这是一个自我削弱的悖论：用来增强泛化能力的手段，反而破坏了泛化能力的来源。

**Gap →** 这个问题此前几乎没人在VLM驾驶场景里系统研究过，也没有专门的基准能检测/量化这种遗忘。

**Novelty →** ①Fidelity Driving Bench：18万场景、15个数据源、90万QA对的大规模基准，首个专门为量化驾驶VLM遗忘设计的评测体系，配套一个新指标Knowledge Retention Rate(KRR)；②Driving Expert Adapter(DEA)：把适配从权重空间转移到prompt空间，用Prompt Adapter(可学习、可检索的prompt embedding)编码任务先验、不碰模型权重，配合Task-Adaptive Expert Module(TAEM，MoE启发的、耦合在完全冻结VLM外部的LoRA专家委员会，按场景线索动态路由)提供专精能力。

**核心洞察 →** 作者做了系统性诊断，发现：(1)全参数微调遗忘最严重；(2)简单加LoRA这种参数高效方法虽然遗忘率更低，但在真正的长尾场景上能力提升不够(甚至不如原模型)；(3)这揭示了一个"知识保留 vs 任务适配"的根本权衡，此前没人提出能同时兼顾两者的方案。

**方法/论证 →** 用自动化pipeline(GPT-OSS-120B提取场景元素+IDF稀有度打分+BM25式长度归一化)从15个已有数据集+WOD-E2E里挖出长尾场景，构建180K训练集+1000张人工复核的测试集；用LLM-as-judge(Qwen-judge为主，交叉验证Qwen3-Max/Gemini2.5-Pro/GPT-5)评Scene Description/Traffic-QA/Noteworthy Objects' Perception三个任务；KRR = 微调后模型在测试集上的Noteworthy Objects' Perception Recall ÷ 原始预训练模型的同一指标。DEA应用在Qwen2.5VL-3B(ImpromptuVLA的基座)上做实验。

**结果 →** ImpromptuVLA-3B(全参数微调)KRR只有68.4%，加上DEA后KRR提升到76.0%，同时Scene Description(59.1→58.7，基本持平)、Traffic-QA(33.0→38.6，+5.6)、NoPR(25.2→27.8，+2.6)都有提升——不是"保知识就得牺牲性能"的简单权衡，而是两者同时改善。

**意义 →** 这是目前查到的、**专门针对驾驶VLM场景**研究"微调会破坏预训练知识"这个问题的首篇系统性工作，提出的解法(prompt空间适配+MoE式专家路由)跟SAMoE-VLA/PerceptDrive"冻结+路由"这个大的设计哲学是同一类，但具体机制(LoRA专家委员会 vs 冻结感知先验+场景条件融合)不同，而且**专家不是蒸馏来的**，这点要跟朱子欣讨论"蒸馏别的MoE"这个idea时讲清楚区别。

---

## 贡献列表(按重要性排序)

**① Fidelity Driving Bench：首个驾驶VLM遗忘量化基准**
整合15个已有驾驶VQA数据集(DriveLM/LingoQA/CODA-LM/ImpromptuVLA等)+额外标注WOD-E2E，用GPT-OSS-120B做场景元素提取和IDF稀有度打分(公式r(e)=log((N+α)/(n_e+α)))，配合BM25式长度归一化(防止物体列表长的图片仅因枚举多而拿高分)，挖出长尾场景。最终180K训练场景+900K QA对+1000张人工复核的测试集，是目前最大规模的语言标注驾驶训练语料(论文自称)。

**② Knowledge Retention Rate(KRR)指标**
受增量学习里Backward Transfer(BWT)概念启发，但BWT衡量的是"学新任务后旧任务表现怎么变"，这里要衡量的是"微调后模型是否还保留预训练阶段学到的、没被专门训练过的感知能力"。KRR = NoPR(微调后模型)/NoPR(原始预训练模型)，用"能不能识别没见过的长尾物体"当预训练知识保留程度的操作化代理指标。KRR接近1.0代表保留良好，大于1.0代表微调后甚至比原模型更强。

**③ Driving Expert Adapter(DEA)：Prompt Adapter + Task-Adaptive Expert Module两件套**
Prompt Adapter(PA)：学一组可训练的embedding，对应核心问题族群及其子技能；推理时先对输入问题算一个语义表示，检索最相关的prompt embedding拼接到原始输入前面——整个过程不改VLM参数本身，靠"注入可检索的prompt"而不是"改权重"来稳定任务理解、增强泛化、缓解灾难性遗忘。

Task-Adaptive Expert Module(TAEM)：灵感来自MoE，"作为一个外部的、模块化的系统，耦合在一个完全冻结的VLM外部"，包含多个LoRA专家(各自对应一类驾驶场景，比如密集城市路口/高速并线/恶劣天气)，一个轻量门控网络处理Prompt Adapter增强后的prompt、结合场景专属线索(能见度/交通密度)，选择并融合最相关的专家输出。

**④ 系统性诊断实验揭示三条Guideline**
(1)数据多样性比数据量更关键，多源训练的模型KRR持续高于单源训练；(2)全参数微调遗忘最严重，LoRA这类参数高效方法遗忘率更低，但TAEM之前的消融显示简单LoRA在真正长尾评测集上性能反而没提升甚至下降(Table 4)；(3)这个权衡的根源不在"改了多少参数"，而在"该不该改权重"——这直接推出DEA的设计动机：完全不改权重(Prompt Adapter)+外部模块化专家(TAEM)。

---

## Novelty 分类判断

- **新问题定义**：算——"VLM驾驶微调会侵蚀预训练世界知识"这个问题此前没被系统提出和验证过，论文明确自称"first to systematically investigate and identify"。
- **新insight**：算——"全参数微调遗忘最严重，LoRA遗忘率更低但长尾性能不够"+"数据多样性是遗忘的关键变量"这两条都有扎实的系统对比实验支撑(Table 2/3/4)。
- **新方法**：算——DEA的两个组件(可检索Prompt Adapter、外部LoRA专家委员会+场景条件门控)是具体的工程设计，尤其"适配发生在prompt空间而非权重空间"这个思路是直接针对诊断出的根因设计的。
- **新理论**：没有，KRR是对已有BWT概念的具体化应用，不是新理论。
- **新数据集**：算——Fidelity Driving Bench(180K场景+900K QA对)是新贡献，整合+挖掘已有数据源+自动化长尾挖掘pipeline。
- **新应用场景**：不算，是方法论文+基准论文的组合。

综合结论：**新问题+新基准+新方法组合类贡献**，核心价值在于首次把"灾难性遗忘"这个持续学习领域的经典问题，系统地搬到"VLM微调适配驾驶场景"这个具体场景里量化并提出解法，而不是又一个"精度更高的驾驶VLM"。

---

## 中文精读

### 1. 解决了什么问题

VLM被引入自动驾驶，核心卖点是它在大规模通用数据上预训练获得的世界知识和常识推理能力，能帮端到端模型应付长尾场景。但要让VLM真正适应驾驶任务，几乎所有现有方法都要在驾驶专用数据上做微调——而这篇论文证明：**这个微调过程本身会系统性侵蚀VLM赖以成立的那份预训训练知识**，造成一个自相矛盾的局面：你为了让VLM"更懂开车"而微调它，结果它反而在那些VLM本该擅长的、没见过的长尾场景上变得更差。论文开篇用一个具体案例(Figure 2)说明：RecogDrive微调后，连基座模型InternVL3-8B本来能认出来的路边障碍物(路缘石、石块)都认不出来了，导致不安全的轨迹规划。

### 2. 新意在哪里

**核心新意是提出一个专门的、操作化的量化手段(KRR)去测这个"看不见的遗忘"**，并且证明现有的缓解手段(LoRA这种参数高效微调)并不像想象中那样能自动解决问题——加了LoRA确实KRR比全参数微调高，但在真正设计出来考验长尾能力的评测集上，LoRA微调后的模型性能甚至可能比原始基座模型还差(论文称这是一个"counter-intuitive phenomenon")。这打破了"参数高效微调=天然防遗忘"的直觉假设。

### 3. Idea 具体落在哪里

**基准构建(Figure 3，§3.1)：** 整合15个数据集(2500万级)标准化成QA格式；用GPT-OSS-120B把每条标注自动分类成Traffic Scene Understanding和Meta Action Decision两类语义；统计出9个最常见物体子类(中型车辆/交通信号/人群/公交卡车/城市元素/移动设备/交通管制/交通标志标线/建筑道路设施)；对每张图提取场景元素集合{e1,...,en}，算每个元素的IDF稀有度r(e)=log((N+α)/(n_e+α))，再用BM25式长度归一化公式(公式2)算整张图的稀有度分数R(i)，取top 1.25%稀有场景(约2500张)做人工复核，最终筛到1000张测试集。训练集用剩下约200K场景(标准化多视角数据集只留正前视角，连续片段蒸馏成关键帧)，最终180K场景+90万QA对。

**指标设计(§3.2)：** NoPR(Noteworthy Objects' Perception Recall)是模型识别场景里关键物体能力的分数(LLM judge按"清楚感知/模糊感知/未感知"打1.0/0.5/0.0分再平均)；KRR=微调后NoPR÷原始预训练模型NoPR，用这个比值把"知识保留"这个抽象概念变成一个具体数字。

**诊断分析(§3.3)：** 发现(1)多源训练数据的模型(比如RecogDrive，15个数据源混合)KRR持续高于单源训练模型，不分基座模型大小；(2)全参数微调在域内任务(Traffic-QA)表现好，但KRR掉得厉害(对照RecogDrive-8B跟其基座InternVL3-8B的KRR差距)；(3)反直觉发现：给基座模型套LoRA微调后，在这篇设计的高难度长尾评测集上，大多数模型性能不升反降(Table 4)，证明"LoRA这种简单参数高效方法本身不足以提升真正长尾场景的能力"。

**DEA架构(Figure 5，§4)：** Prompt Adapter在推理时先把输入问题编码成语义表示，检索最相关的可学习prompt embedding，拼接进输入，全程不碰VLM权重。Task-Adaptive Expert Module是耦合在冻结VLM外部的模块化系统，包含多个LoRA专家(各自专精一类场景)，一个轻量门控网络结合增强后的prompt和场景线索(能见度/交通密度)，选择+融合最相关的专家。两者协同：Prompt Adapter靠"把知识压缩进prompt"保证稳定性(防遗忘)，TAEM靠"部署专精LoRA专家"提供适应性(防LoRA单体适配器能力不够的问题)。

**实验(§5)：** 在Qwen2.5VL-3B(ImpromptuVLA-3B的基座)上对比：Base(原始，无KRR定义，自身就是基准)；Base+Full(全参数微调，KRR 64.3%)；Base+LoRA(64.6%，跟全参数微调几乎一样差，证明LoRA本身不解决问题)；Base+TAEM(74.4%，仅加专家模块就有明显提升)；Base+TAEM+PA即完整DEA(79.0%，两个组件叠加最好)。在真实数据集ImpromptuVLA上，DEA把KRR从68.4%提到76.0%，同时三个任务指标都有提升而非牺牲。

### 4. 最大难点在哪里

不在"微调会遗忘"这个直觉判断本身，难点在于**怎么证明"知识保留"和"任务适配"这两者之间不是必然互斥的、存在一个能同时兼顾的方案**——论文为此做的最关键实验是Table 4那组LoRA反而变差的反直觉结果，它排除了"只要用参数高效方法就行"这种简单想法，逼着论文去设计真正"不碰权重"(Prompt Adapter)+"模块化专精"(TAEM)的组合方案，而不是随便套一个PEFT方法就了事。

### 5. 局限 / 对我项目的相关性

**TAEM里的LoRA专家不是蒸馏自外部已训练好的模型，是在DEA自己的训练数据上端到端训练出来的**——这是这篇论文跟"蒸馏别的MoE当冻结专家"这个idea最大的差异点，之前摘要级笔记把这篇归进"路由漂移问题的解法"是合理的，但不能当成"蒸馏别的MoE"这半个机制的验证案例。

**专家数量/场景划分方式论文里举例是"密集城市路口/高速并线/恶劣天气"这类直觉式分类**，没有看到像SAMoE-VLA那样严谨的消融(专家数量到底几个最优、划分维度是否经过系统验证)，这块可能需要看Appendix确认。

**对我项目的相关性：** ①如果要做"驾驶MoE怎么防止持续学习/微调时遗忘"，这篇的KRR指标和Fidelity Driving Bench基准可以直接拿来当评测工具，不需要重新造轮子；②这篇"适配发生在prompt空间而不是权重空间"这个设计原则，对SAMoE-VLA/PerceptDrive这类本来就在权重空间做路由融合的设计是一个值得警惕的参照——如果以后要给这些模型加"持续学习/防遗忘"能力，这篇的诊断(权重空间改动是遗忘的根源之一)提示或许该往prompt空间或者外挂模块方向想，而不是继续在权重空间做更复杂的融合。

### 6. 是否能连 GitHub

论文abstract明确写"Data and model are released at FidelityDrivingBench"，但撰写本笔记时未逐一核实链接可用性和完整性。

---

## 假设、局限与质疑点

- **KRR用"长尾物体识别"这一个代理任务衡量"预训练知识保留"**，这是一个operationalization choice，不代表预训练VLM的所有知识类型(比如常识推理、语言理解)都能被这一个指标覆盖，presentation时要讲清楚KRR测的是"感知知识保留"这个子集，不是全部。
- **实验主要在Qwen2.5VL-3B一个基座模型上做**(其他基座的实验在Appendix，本笔记还没细读)，DEA的收益是否能泛化到更大/更小的基座模型，需要看Appendix确认。
- **TAEM专家的场景划分(城市路口/高速/恶劣天气等)像是直觉式设计，不是数据驱动自动聚类出来的**，划分是否最优没有消融验证。

---

## 一个月后记住 5 件事

1. **核心论点：微调VLM去适配驾驶数据会系统性侵蚀预训练世界知识**，这是一个此前没被量化研究过的自我削弱悖论，论文首次提出KRR指标把它变成可测量的数字。
2. **反直觉发现：简单套LoRA并不能自动解决这个问题**，在真正设计出来考验长尾能力的评测集上，LoRA微调后的模型性能有可能不如原始基座模型。
3. **DEA的解法是"不改权重"：Prompt Adapter在prompt空间注入知识(不碰VLM参数)+TAEM用外部LoRA专家委员会(耦合在冻结VLM外部)按场景路由提供专精能力**。
4. **DEA不是"保知识牺牲性能"的权衡，是两者同时改善**：KRR从68.4%提到76.0%的同时，Scene Description/Traffic-QA/NoPR三个任务指标也都提升了。
5. **TAEM里的专家是训练出来的，不是蒸馏自外部已有模型**——跟"蒸馏别的MoE当冻结专家"这个idea的机制不同，这篇验证的是"冻结基座+外部可训练专家委员会+场景条件路由"这个结构，不是"复用别人训练好的MoE"。

---

## 关键原文摘录(中英对照)

**① 核心问题诊断**
> "the very fine-tuning process used to adapt these models to driving-specific data simultaneously erodes their invaluable pre-trained world knowledge, creating a self-defeating paradox that undermines the core reason for their use."

中译：用来把这些模型适配到驾驶专用数据的微调过程，同时侵蚀了它们宝贵的预训练世界知识，造成一个自我削弱的悖论，破坏了使用它们的核心理由。
解读：这是全文的核心论点，presentation开场直接用这句话能把问题讲清楚。

**② TAEM的结构定位**
> "TAEM operates as an external, modular system coupled with a completely frozen VLM. It comprises multiple 'experts', each tailored to a distinct driving scenario."

中译：TAEM作为一个外部的、模块化系统运作，耦合在一个完全冻结的VLM上。它由多个"专家"组成，每个专家都针对一类不同的驾驶场景量身定制。
解读：明确了专家是"针对场景量身定制"训练出来的，不是蒸馏自外部已有模型——这是区分这篇和"蒸馏别的MoE"idea的关键原文证据。

**③ LoRA反直觉发现**
> "When we applied LoRA fine-tuning to base models using our diverse training corpus, most models' performance on our challenging, long-tail evaluation set did not improve; it actually degraded."

中译：当我们用我们多样化的训练语料对基座模型做LoRA微调时，大多数模型在我们这个有挑战性的长尾评测集上的表现没有提升，反而下降了。
解读：这是推翻"参数高效微调=天然防遗忘"这个直觉假设的关键证据，直接推动了DEA的设计。

---

## English at-a-glance summary

| Aspect | Summary |
|---|---|
| **Problem** | Fine-tuning VLMs on driving-specific data erodes the pre-trained world knowledge that motivated using VLMs in the first place.<br>微调VLM适配驾驶数据会侵蚀VLM之所以被引入自动驾驶的那份预训练世界知识。 |
| **Novelty** | Fidelity Driving Bench (180K scenes, first forgetting-specific benchmark) + Knowledge Retention Rate metric + Driving Expert Adapter (prompt-space adaptation + MoE-inspired frozen-VLM-external LoRA expert committee).<br>首个遗忘专用驾驶基准+知识保留率指标+把适配转移到prompt空间、配合冻结VLM外部的MoE式LoRA专家委员会。 |
| **Core idea** | Naive PEFT (LoRA) doesn't solve the forgetting-vs-adaptation trade-off — on a truly hard long-tail eval, LoRA-tuned models can underperform the untuned base model. The fix must avoid touching the backbone's weights at all.<br>朴素的参数高效微调(LoRA)并不能解决遗忘-适配权衡——在真正困难的长尾评测上，LoRA微调后的模型可能不如未微调的基座。解法必须完全不碰主干权重。 |
| **Hardest part** | Proving knowledge retention and task adaptation aren't inherently in tension — DEA improves both KRR and task metrics simultaneously, not a trade-off.<br>证明知识保留和任务适配不是天生互斥的——DEA同时提升KRR和任务指标，不是权衡取舍。 |
| **Evidence** | On ImpromptuVLA-3B, DEA raises KRR from 68.4% to 76.0% while also improving Scene Description, Traffic-QA, and NoPR.<br>在ImpromptuVLA-3B上，DEA把KRR从68.4%提到76.0%，同时Scene Description/Traffic-QA/NoPR都有提升。 |
| **Limitation** | TAEM's LoRA experts are trained end-to-end on DEA's own data, not distilled from other pretrained models — this is NOT a "distill external MoE as frozen experts" mechanism, despite the MoE-inspired framing.<br>TAEM的LoRA专家是在DEA自己的数据上端到端训练出来的，不是蒸馏自其他预训练模型——尽管设计灵感来自MoE，这不是"蒸馏外部MoE当冻结专家"这个机制。 |
| **Code / GitHub** | Released at FidelityDrivingBench per the abstract; link/completeness not individually verified at note-writing time.<br>Abstract写明发布于FidelityDrivingBench；具体链接/完整性撰写本笔记时未逐一核实。 |

---

## 本文提到的词

| 术语 | 首次出现位置 |
|------|------------|
| Catastrophic Forgetting(灾难性遗忘) | Abstract |
| Fidelity Driving Bench | Abstract |
| Knowledge Retention Rate(KRR，知识保留率) | Abstract |
| Driving Expert Adapter(DEA) | Abstract |
| Prompt Adapter(PA) | §4 |
| Task-Adaptive Expert Module(TAEM) | §4 |
| Noteworthy Objects' Perception Recall(NoPR) | §3.2.2 |
| Backward Transfer(BWT) | §3.2.2 |
| IDF(Inverse Document Frequency，逆文档频率) | §3.1.1 |
