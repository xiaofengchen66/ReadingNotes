# On Token's Dilemma: Dynamic MoE with Drift-Aware Token Assignment for Continual Learning of Large Vision Language Models

- **Venue / year:** arXiv 2603.27481, 2026 (CVPR 2026)
- **Authors:** Chongyang Zhao, Mingsong Li, Haodong Lu, Dong Gong — University of New South Wales (UNSW Sydney)
- **Link:** https://arxiv.org/abs/2603.27481 ([本地PDF副本](pdf/on-tokens-dilemma.pdf))
- **Code:** 项目页面 zhaoc5.github.io/DyMoE，具体仓库内容本笔记未逐一核实
- **Input / 输出:** 多模态指令微调数据(图像+指令+答案三元组，按任务流到达) → 对应任务的回答
- **精读说明(2026-10-22)：用户让我把本地下载的论文都精读一遍，这是优先级较高的一篇(直接回应MoE持续学习中的"路由漂移"问题)，已完整读过PDF正文。**

---

## TL;DR

**问题 →** 多模态持续指令微调(MCIT)要求LVLM在不断学习新任务的同时不遗忘旧任务的能力。MoE架构天然适合持续学习(增量加专家、冻结旧专家、按任务扩展路由)，但即使做了这种专家隔离，基于MoE的持续学习器依然会出现灾难性遗忘。

**Gap →** 现有方法要么依赖任务级的路由预测器(需要额外计算，任务多样复杂时不可靠)，要么直接做动态MoE扩展但没有深入分析"为什么专家隔离了还是会遗忘"这个token级的根本原因。

**Novelty →** 提出LLaVA-DyMoE：先做受控的两任务实验，精确定位出遗忘的token级成因是"路由漂移"(routing-drift)——训练新任务时，即使旧专家和旧路由参数都冻结，新加入的、未充分优化的路由器依然会把旧任务的token错误吸引到新专家上。进一步发现新任务数据里的token按它们跟新旧专家组的affinity可以分成三类(新token/旧token/模糊token)，每类对"新任务学习"和"遗忘风险"的贡献完全不同。基于此设计两重正则化：Token Assignment Guidance(TAG，识别并引导模糊/旧token远离新专家)+Routing Score Regularization(RSR，强制新旧专家组路由互斥+鼓励新专家专精)。

**核心洞察 →** 不是所有新任务token都一样有价值——"新token"(跟新专家组affinity明显更高)真正承载新知识、遗忘少；"旧token"(跟旧专家组affinity相近)对新任务学习贡献小，但如果被under-optimized router误导、分到少量非零权重给新专家，会反过来污染新路由策略；"模糊token"(新旧affinity接近、难以区分)对新任务学习价值最小，却是最直接的遗忘风险来源。

**方法/论证 →** 对给定层的token，分别取新旧专家组内的最大路由logit当confidence score(c_old/c_new，Eq.2)，算相对差异D_rel(Eq.3)识别模糊token(D_rel≤τ)；TAG用二值mask(Eq.4)，把模糊token和旧主导token(c_old≥c_new)都强制路由给旧(冻结)专家组，只让真正新主导且非模糊的token路由给新专家；RSR包含互斥损失L_exc(Eq.6，惩罚同一个token同时强激活新旧两组)和专精损失L_spe(Eq.7，用BCE鼓励新专家组的集体路由权重随"旧专家激活程度降低"而提高)；总损失(Eq.8)是标准指令微调损失+MoE负载均衡损失+这两个正则化项的加权和。

**结果 →** CoIN基准(8个VQA任务序列，569k训练样本)上，相比不加任何正则化的IncMoELoRA基线，LLaVA-DyMoE在MFN(平均最终准确率)提升7.35、MAA(平均准确率)提升8.20、BWT(遗忘指标)改善12.00个百分点(从-16.67到-4.67)；相比LoRA/MoELoRA/EWC/LWF/IncLoRA/O-LoRA等基线全面最优；跟数据层面的持续学习方法(SEFE的ASD范式、ProgLoRA的replay buffer)结合使用都有进一步提升，证明方法正交兼容。

**意义 →** 这是目前查到的、**把"MoE持续学习里的遗忘"精确定位到token级路由漂移机制**的论文，不是笼统地说"专家隔离不够好"，而是具体指出哪类token在什么情况下会破坏已有路由策略，并针对性设计正则化方案。

---

## 贡献列表(按重要性排序)

**① 路由漂移的token级归因：三类token的受控实验**
通过两任务(SQA→TextVQA)的控制实验(Fig.2)，分别测试"只保留新token的贡献"、"屏蔽掉旧token的贡献"、"只保留模糊token的贡献"三种设置对新任务学习和旧任务遗忘的影响。发现：①只用新token训练，新任务学得好、旧任务几乎不遗忘(新token天然跟新专家对齐，不干扰旧路由)；②屏蔽旧token效果类似baseline(说明旧token对新学习贡献有限，但如果被under-optimized router分到小量非零权重，会悄悄把新路由器带偏向旧模式)；③只用模糊token训练，新任务学不好、旧任务也遗忘(模糊token学习价值最小、但遗忘风险最直接，因为它们在新旧专家组之间路由不稳定)。

**② Token Assignment Guidance(TAG)：按token类型引导路由分配**
TAG的核心逻辑：一个token只有同时满足"新主导(c_new>c_old)"和"不模糊(D_rel>τ)"两个条件，才被允许路由给新专家组；否则(旧主导，或者模糊)一律强制分配给旧(冻结)专家组，走安全路径。这个机制直接针对分析出的token类型差异，不需要任何额外的任务标签或先验假设，只依赖token自身的路由logit就能判断。

**③ Routing Score Regularization(RSR)：互斥损失+专精损失**
互斥损失L_exc(Eq.6)最小化新旧专家组各自集体路由权重的乘积，强制一个token不能同时强激活新旧两组，直接巩固TAG做出的条件路由决策；专精损失L_spe(Eq.7)用BCE，鼓励新专家组的集体路由权重g_new随"g_old趋近于0"的程度而提高，用来平衡stability(TAG+L_exc带来的保守性)和plasticity(确保新专家确实能被有效利用来学新知识)，两者协同使用。

**④ 跟数据层面持续学习方法的正交兼容性验证**
论文专门做了实验(Table 5)，证明LLaVA-DyMoE跟ASD数据范式(SEFE)、replay buffer方法(ProgLoRA)结合使用都能进一步改善BWT——跟SEFE的ASD结合BWT从-10.45改善到-4.75，跟小replay buffer结合BWT从ProgLoRA的-6.59改善到-1.55。这证明了方法是在"路由机制"这个层面解决问题，跟"数据怎么组织/要不要重放旧数据"这个层面是正交的，可以叠加使用。

---

## Novelty 分类判断

- **新问题定义**：算——明确提出"routing-drift"这个MoE持续学习里被忽视的token级遗忘成因，不是笼统地说专家隔离不够，而是具体指出"即使专家隔离了，router本身的更新依然会漂移"这个机制。
- **新insight**：算——三类token(新/旧/模糊)对新任务学习和遗忘的不同贡献，有Fig.2的受控两任务实验直接支撑，是本文最核心的实证发现。
- **新方法**：算——TAG(基于token-level ambiguity的路由引导)和RSR(互斥损失+专精损失)都是具体的新机制，直接对应分析出的问题根源设计。
- **新理论**：没有，是已有MoE+LoRA持续学习框架上的具体正则化设计。
- **新数据集**：没有，用已有CoIN基准(8个VQA任务)。
- **新应用场景**：不算，是LVLM持续学习这个已有任务。

综合结论：**新问题+新insight+新方法组合类贡献**，核心价值是把"MoE持续学习的遗忘"从笼统的现象描述，精确拆解到token级的路由漂移机制，并且给出了不依赖任务标签、只用路由logit就能实现的轻量正则化方案，实证和分析都比较扎实。

---

## 中文精读

### 1. 解决了什么问题

多模态持续指令微调(MCIT)要求大视觉语言模型(LVLM)像流水线一样持续学习新的任务，同时不能遗忘之前学过的任务能力。MoE架构(尤其是配合LoRA专家的动态MoE，DyMoE)看起来是天然适合这个场景的方案：给新任务增加新的LoRA专家和路由器，旧的专家和路由参数全部冻结。但论文发现，即使做了这种看似彻底的隔离，训练新任务时依然会侵蚀旧任务的性能——论文要回答：**专家都隔离了、参数都冻结了，遗忘到底是从哪里钻进来的？**

### 2. 新意在哪里

核心新意是**把分析粒度下沉到token级**——以往分析遗忘问题大多停留在模型/任务层面("多任务训练会互相干扰")，这篇指出：真正的问题出在**路由器本身**。即使旧专家冻结，训练新任务时新路由参数在更新，而新路由器一开始是欠优化的(under-optimized)，它会把跟新专家"看起来有点像"的旧任务token，错误地分配给新专家——这个token-路由器之间的错配关系，才是遗忘真正钻进来的缝隙。

### 3. Idea 具体落在哪里

**整体框架(Fig.3)：** 在LLaVA的语言模型每一层，MoE层由多个LoRA专家+路由器组成。新任务t到来时，新增N_t个新专家和对应的路由器输出维度，旧专家集合E_{t-1}连同路由参数全部冻结，只有新增的专家和路由参数可训练。输入token在每一层同时拿到对旧专家组和新专家组的路由logits s=[s_old, s_new]。

**识别模糊/旧/新token(§3.3-3.4)：** 对某一层的token表示，取旧专家组路由logits里的最大值c_old、新专家组里的最大值c_new(Eq.2)，算相对差异D_rel=|c_new-c_old|/max(|c_new|,|c_old|,ε)(Eq.3)。D_rel≤τ(消融显示τ在10%-20%之间最优)判定为模糊token；c_old≥c_new判定为旧主导token；只有c_new>c_old且D_rel>τ才是真正的"新主导、非模糊"token。

**TAG的路由分配(Eq.4)：** 用二值mask M_new=1[(c_new>c_old)∧(D_rel>τ)]，M_old=1-M_new，把最终路由logits s'构造成：满足新主导+非模糊条件的token保留其新专家组的logits，其余所有情况(旧主导或模糊)的token在新专家组上的logits被设为-∞(softmax后权重变成0)——即强制路由到旧(冻结)专家组。这个设计是"保守优先"：只要不是明确该学新知识的token，一律走安全的旧路径。

**RSR的两个损失(Eq.5-7)：** 先定义新旧专家组各自的collective gate输出g_old/g_new(该组内所有专家路由权重之和，Eq.5)；互斥损失L_exc=g_old·g_new(Eq.6)，最小化这个乘积等于强制一个token不能同时在新旧两组都拿到高权重；专精损失L_spe用BCE(Eq.7)，目标y定义为1-g_old的某种软化形式，鼓励g_new随"旧专家激活程度趋于0"而提高，确保新专家组确实在被有效使用，而不是TAG+L_exc把所有token都保守地推给旧专家导致新任务学不动。

**总损失(Eq.8)：** L=L_NTP(标准自回归指令微调损失)+λL_aux(标准MoE负载均衡损失)+α(L_exc+L_spe)，用统一的α系数控制两个正则化项的影响。

### 4. 最大难点在哪里

不在"给MoE加专家隔离"这个大方向判断上(这是已有做法)，难点在于**怎么在不依赖任务标签、不引入额外计算开销的前提下，仅凭token自身的路由logit就准确识别出"这个token该不该被允许路由给新专家"**——这正是D_rel这个相对差异指标要解决的问题：用一个简单、可微、不需要额外监督的量化方式刻画token的"路由模糊性"，是整个方法能落地的关键基础设施。

### 5. 局限 / 对我项目的相关性

**只在LLaVA-v1.5-7B这一个LVLM基座上验证**，论文结论部分提到未来工作要研究在更大规模模型上的可扩展性，没有在更大/更小的模型上验证过。

**ambiguity阈值τ需要人工调参**(消融显示10%-20%之间较优，Table 3)，不是自适应学出来的，不同数据分布/任务数量下最优τ是否会变化没有讨论。

**只在CoIN基准(8个VQA任务序列)上验证**，这8个任务之间的相似度/多样性程度跟更复杂的真实世界持续学习场景(比如跨城市、跨季节的驾驶场景)是否可比，没有讨论。

**对我项目的相关性：这是"MoE持续学习怎么防止路由漂移"这个问题目前查到最扎实的解法之一**，跟PerceptDrive笔记里讨论过的"H_BEV只算一次、场景突变路由滞后"是相关但不同的问题——On Token's Dilemma关心的是"增量学习新任务/新场景时，旧场景的路由模式会被悄悄破坏"(跨任务/跨时间的持续学习问题)，PerceptDrive的局限关心的是"单次推理内，场景实时突变时路由跟不上"(单次推理内的实时性问题)。如果SAMoE-VLA/PerceptDrive要做真正的持续学习(比如不断适应新城市、新驾驶场景、新传感器配置)，这篇"区分新/旧/模糊token、分别引导路由"的机制，以及"D_rel这种基于路由logit相对差异的轻量判别指标"，可以直接参考复用。

### 6. 是否能连 GitHub

项目页面：zhaoc5.github.io/DyMoE，撰写本笔记时未逐一核实具体仓库内容和代码完整性。

---

## 假设、局限与质疑点

- **三类token的划分完全基于路由logit的相对差异**，没有讨论这种划分方式跟token的语义内容(比如token对应的具体视觉/文本信息)之间的关系是否稳定，presentation时如果要深入可以追问这一点。
- **τ的消融(Table 3)显示10%-20%最优，30%-50%效果变差**，但论文没有讨论为什么是这个具体区间、背后有没有更一般的理论解释。
- **RSR的两个损失(互斥+专精)之间有一定的张力**(一个推动保守、一个推动利用新专家)，论文用统一的α系数平衡，但没有讨论两个损失各自独立的权重是否会有更好的组合。

---

## 一个月后记住 5 件事

1. **核心发现：MoE持续学习即使做了专家隔离，依然会因"路由漂移"产生遗忘**——训练新任务时，欠优化的新路由器会把旧任务token错误吸引到新专家上。
2. **三类token：新token(学习价值高、遗忘风险低)、旧token(学习价值低、有一定遗忘风险)、模糊token(学习价值最小、遗忘风险最直接)**——这个划分来自受控两任务实验(Fig.2)。
3. **TAG机制：只有"新主导且不模糊"的token才允许路由给新专家，其余一律强制路由给冻结的旧专家组**，不依赖任务标签，只用路由logit的相对差异判断。
4. **RSR两个损失：互斥损失(防止同时激活新旧两组)+专精损失(鼓励新专家组确实被有效利用)**，两者协同平衡stability和plasticity。
5. **核心结果：CoIN基准上MFN+7.35、MAA+8.20、BWT改善12.00个百分点**，跟数据层面方法(ASD/replay)正交兼容、结合使用效果更好。

---

## 关键原文摘录(中英对照)

**① 核心问题定位**
> "Despite expert isolation, MoE-based continual learners still suffer from forgetting due to routing-drift: old-task tokens become mistakenly attracted to newly added experts, degrading performance on prior tasks."

中译：尽管做了专家隔离，基于MoE的持续学习器依然会因为路由漂移而遭受遗忘：旧任务的token被错误地吸引到新加入的专家上，导致先前任务的性能下降。
解读：这是全文的核心问题诊断，presentation开场直接用这句话能把问题讲清楚。

**② 核心结果数字**
> "Our method significantly outperforms these methods through our token-level routing regularization. This enforces the assignment of ambiguous cross-task tokens to old, frozen experts, bringing improvements of 7.35%, 8.20%, and 12.00% on MFN, MAA, and BWT, respectively."

中译：我们的方法通过token级路由正则化显著超过了这些方法。这强制把跨任务的模糊token分配给旧的冻结专家，在MFN、MAA、BWT上分别带来了7.35%、8.20%、12.00%的提升。
解读：这组具体数字是presentation里最直接的证据，"把模糊token强制分给旧专家"这个机制的效果很明确。

---

## English at-a-glance summary

| Aspect | Summary |
|---|---|
| **Problem** | Even with expert isolation, MoE-based continual learners for LVLMs still forget due to "routing-drift" — old-task tokens get mistakenly attracted to newly added experts.<br>即使做了专家隔离，LVLM的MoE持续学习器依然会因"路由漂移"而遗忘——旧任务token被错误吸引到新加入的专家上。 |
| **Novelty** | A token-level analysis identifying three token types (new/old/ambiguous) with different contributions to learning and forgetting, plus Token Assignment Guidance (TAG) and Routing Score Regularization (RSR) to mitigate drift.<br>Token级分析识别出三类token(新/旧/模糊)对学习和遗忘的不同贡献，加上TAG和RSR两重机制缓解漂移。 |
| **Core idea** | An under-optimized new router, even with old experts frozen, will mis-route old-task tokens to new experts — the drift is a router-level problem, not just a missing-isolation problem.<br>即使旧专家冻结，一个欠优化的新路由器依然会把旧任务token错路由给新专家——漂移是路由器层面的问题，不只是隔离不够的问题。 |
| **Hardest part** | Identifying which tokens should be allowed to route to new experts using only the token's own routing logits, with no task labels or extra supervision.<br>只靠token自身的路由logit(不需要任务标签或额外监督)判断哪些token该被允许路由给新专家。 |
| **Evidence** | On the CoIN benchmark (8 VQA tasks): +7.35 MFN, +8.20 MAA, +12.00pp BWT improvement over an unregularized incremental MoE baseline; compatible with and complementary to data-based continual learning methods.<br>CoIN基准(8个VQA任务)上，相比无正则化的增量MoE基线：MFN+7.35、MAA+8.20、BWT改善12.00个百分点；跟数据层面持续学习方法兼容互补。 |
| **Limitation** | Only validated on LLaVA-v1.5-7B and the CoIN benchmark; the ambiguity threshold τ is manually tuned, not adaptive.<br>只在LLaVA-v1.5-7B和CoIN基准上验证；模糊阈值τ靠人工调参，不是自适应的。 |
| **Code / GitHub** | Project page released (zhaoc5.github.io/DyMoE); exact repository contents not individually verified at note-writing time.<br>项目页面已发布；具体仓库内容撰写本笔记时未逐一核实。 |

---

## 本文提到的词

| 术语 | 首次出现位置 |
|------|------------|
| Routing-drift(路由漂移) | Abstract |
| Token's Dilemma(token的困境) | Abstract |
| Token Assignment Guidance(TAG) | Abstract |
| Routing Score Regularization(RSR) | Abstract |
| Multimodal Continual Instruction Tuning(MCIT) | §1 |
| Dynamic MoE(DyMoE) | §3.2 |
