# MoDES: Accelerating Mixture-of-Experts Multimodal Large Language Models via Dynamic Expert Skipping

- **Venue / year:** arXiv 2511.15690, 2025 (CVPR 2026)
- **Authors:** Yushi Huang, Zining Wang, Zhihang Yuan, Ruihao Gong, Yifu Ding, Jinyang Guo, Xianglong Liu, Jun Zhang — 香港科技大学 + 北京航空航天大学 + 北京大学
- **Link:** https://arxiv.org/abs/2511.15690 ([本地PDF副本](pdf/modes.pdf))
- **Code:** 已开源：https://github.com/ModelTC/MoDES
- **Input / 输出:** 多模态输入(图像/视频+文本) → 对应任务输出(VQA答案、描述等)；本身不是一个新模型，是给已有MoE MLLM做推理加速的插件式框架
- **精读说明(2026-10-22)：用户让我把本地下载的论文都精读一遍，这是优先级最高的几篇之一(直接回应"SA-MoE没有真正省计算量"这个已识别的局限)，已完整读过PDF正文。**

---

## TL;DR

**问题 →** MoE多模态大模型(MLLM)计算开销巨大(Qwen2-VL 72B处理4K-token输入在2张A100上只有<10 tokens/s)，专家跳过(expert skipping)是常见加速手段，但现有专家跳过方法都是给纯文本LLM设计的，直接套用到MLLM上会导致精度大幅下降(跳过83%专家时精度掉超过10%)。

**Gap →** 现有专家跳过方法只看层内(intra-layer)的局部路由概率来决定跳谁，忽略了两件事：①不同MoE层对最终输出的全局贡献差异很大(浅层比深层关键得多)；②同一层里，文本token和视觉token通过专家时的行为(更新幅度)完全不同。

**Novelty →** 提出MoDES(Multimodal Dynamic Expert Skipping)：①全局调制的局部门控(GMLG)，把离线校准出的层级全局重要性因子和推理时的局部路由概率结合，算出更准的token级专家重要性分数；②双模态阈值(DMT)，给文本token和视觉token分别设独立的跳过阈值；③为了高效找到最优阈值组合，提出frontier search算法，利用"阈值越高、跳得越多、精度越低"这个单调性，把搜索复杂度从O(ND²)降到O(ND)，搜索时间从几天压到几小时。

**核心洞察 →** 浅层专家更关键(深层的误差会被浅层已经引入的误差放大，"error explosion")；视觉token比文本token更能容忍专家被跳过(作者通过余弦相似度+角度分析发现，FFN对视觉token的更新幅度更小，因为视觉token表示跟FFN权重的夹角更接近90°/更正交)。

**方法/论证 →** 离线用KL散度校准每层的全局重要性α^(l)(对比原模型和"跳过第l层专家"后的输出分布差异)；推理时每个token的重要性分数s_i^(l) = α^(l)·π_i^(l)(局部路由概率乘全局重要性)；按模态分别判断s_i^(l)是否低于对应阈值(文本用τ_t，视觉用τ_v)，低于阈值的专家被跳过(赋一个无效的哨兵专家id，实际不参与计算)；frontier search基于单调性假设用动态规划式的方式快速定位最优(τ_t, τ_v)组合，论文给出了完整的正确性证明(Lemma 1-4)。

**结果 →** 在Kimi-VL/Qwen3-VL-MoE/InternVL3.5三个MoE MLLM家族、13个图文/视频理解基准上验证。跳过88%专家时，Qwen3-VL-MoE-30B-A3B-Instruct上相对原模型保留97.33%性能(基线方法只有86.66%，提升10.67%)；相比naive search，frontier search把搜索时间缩短约45倍；配合量化使用时兼容性好(2.5-bit量化下MoDES保留94.43%性能，MC-MoE只保留89.58%)；推理速度prefilling阶段约2倍加速，decoding阶段约1.2-1.3倍加速。

**意义 →** 这是目前查到的、**直接回应"MoE模型怎么真正省计算量"这个问题最扎实的一篇**——不是设计新的MoE架构，而是给已经训练好的MoE MLLM做训练无关的推理加速，而且专门针对"多模态"这个此前被忽略的维度(模态感知阈值)，代码已开源可直接复现。

---

## 贡献列表(按重要性排序)

**① 两个关键诊断：层级全局贡献差异 + 模态行为差异**
通过控制变量实验(Fig.2，对ChartQA/MME/VideoMMMU三个基准、不同层区间分别测试降低top-k的影响)发现：浅层(1-10层)降低k值比深层(17-26层)掉分严重得多。作者给出的解释是深层的误差会被浅层已经引入的误差级联放大(error explosion)，所以浅层专家更不能随便跳。通过t-SNE可视化(Fig.3左)+余弦相似度分析(Fig.3中)+角度分析(Fig.3右)发现：视觉token经过FFN前后的表示变化幅度明显小于文本token，而且视觉token表示跟FFN权重的夹角更接近90°(更正交)，这从几何上解释了为什么视觉token对专家跳过更不敏感。

**② GMLG(全局调制的局部门控)：把层级重要性和token级路由概率结合**
现有方法只用公式π_i^(l)(局部路由概率，Eq.1)判断要不要跳某个专家，MoDES额外引入一个离线校准出的层级全局因子α^(l)——用一个校准集(GQA数据集1024个样本)，对每一层分别计算"跳过这一层专家"前后模型输出分布的KL散度(Eq.4)，散度越大说明这层越关键、α^(l)越大。最终token重要性分数s_i^(l)=α^(l)·π_i^(l)(Eq.3)，这个分数同时考虑了"这层整体有多重要"和"这个token对这个专家有多依赖"两件事，比只看局部路由概率更准。

**③ DMT(双模态阈值)：文本和视觉分开设阈值，不是一刀切**
给定每层的重要性分数后，设两个独立阈值τ_t(文本)和τ_v(视觉)(Eq.5)，低于对应阈值的专家被跳过。这个设计直接对应第①点里发现的"视觉token更能容忍跳过"这个insight——如果用同一个阈值，要么对视觉token跳得不够狠(浪费计算)，要么对文本token跳得太狠(伤精度)。

**④ Frontier search：把阈值搜索从几天压缩到几小时**
两个阈值(τ_t, τ_v)的组合空间是O(D²)(D是候选阈值格点数)，naive search要对每个组合都跑一遍评测，代价是O(ND²)。作者证明在"f(性能损失)和g(跳过比例)分别对τ_t、τ_v单调"这个假设下，最优阈值一定落在一条"frontier"(边界)上，可以用类似双指针的算法在O(ND)时间内找到，并给出了完整的数学证明(Lemma 1-4、Proposition 1-2)。实测对20-30B参数规模的MoE MLLM，校准+搜索总耗时从几天压到不到4小时，相比naive search快约45倍。

---

## Novelty 分类判断

- **新问题定义**：不算——expert skipping这个大方向此前已有工作(NAEE、MC-MoE、DiEP等)，这篇是指出"直接把这些方法从纯文本LLM搬到MLLM上会失效"这个具体问题。
- **新insight**：算——"浅层比深层关键"+"视觉token比文本token更能容忍跳过"两条insight都有扎实的控制变量实验和几何分析支撑(Fig.2/Fig.3)，此前文献没有系统指出过。
- **新方法**：算——GMLG(全局+局部重要性融合)、DMT(模态感知阈值)、frontier search(高效搜索算法+完整理论证明)都是具体的工程创新，frontier search的数学证明部分尤其扎实。
- **新理论**：算，frontier search给出了完整的单调性假设、正确性证明(Lemma 1-4)和复杂度分析，是这批论文里理论部分最完整的之一。
- **新数据集**：没有，用已有基准。
- **新应用场景**：不算，是通用MLLM加速方法。

综合结论：**新insight+新方法+部分新理论组合类贡献**，核心价值是把"专家跳过"这件事从"纯文本LLM的经验做法直接照搬"升级成"针对MLLM的层级和模态特性专门设计"，而且给出了高效搜索算法的严格理论证明，是方法论和理论证明都比较扎实的一篇工程论文。

---

## 中文精读

### 1. 解决了什么问题

MoE MLLM(比如Kimi-VL、Qwen3-VL-MoE)为了处理更复杂的图文/视频任务，模型越做越大，但这带来严重的推理效率瓶颈——专家跳过是一个常见的加速方法(跳过对当前输入不重要的专家，省掉这部分计算)，但这些方法本来是给纯文本LLM设计的。论文要回答的问题是：**直接把这些方法套用到MLLM上，到底行不行？如果不行，问题出在哪？**

### 2. 新意在哪里

核心新意是精确定位了"直接套用会失效"的两个根本原因，而不是笼统地说"MLLM更复杂所以不行"：一是层级粒度的问题(现有方法只看层内局部信息，没有跨层的全局视角)，二是模态粒度的问题(现有方法为纯文本设计，根本没考虑过视觉token存在)。针对这两个具体原因分别设计了对应的机制(GMLG解决第一个，DMT解决第二个)，而不是笼统地调一个更保守的全局阈值。

### 3. Idea 具体落在哪里

**整体框架(Fig.4)：** 推理时，对MoE层每个token(文本或视觉)，先按Eq.1算局部路由概率π_i^(l)，再乘以离线校准好的全局因子α^(l)得到重要性分数s_i^(l)(Eq.3)；然后看这个token的模态，用对应的阈值(τ_t或τ_v)跟s_i^(l)比较(Eq.5)，低于阈值的专家被跳过。

**全局因子α^(l)怎么校准(Eq.4)：** 用校准集C(GQA数据集采样1024条)，对每条数据分别计算原模型的输出概率分布prob_j和"跳过第l层专家后"的输出概率分布prob_j^(l)，算两者的KL散度，在校准集上取平均得到α^(l)。这个过程完全离线，推理时直接用预计算好的α^(l)，不引入额外开销。

**阈值搜索(Algorithm 1)：** 候选阈值集合B里有D个格点，frontier search从p=D开始，对每个q(文本阈值候选的index)，用while循环递减p直到违反目标跳过比例ρ的约束，记录下这个边界点，整个过程最多访问O(D)次评测，比穷举所有D²组合快得多。论文证明了这个边界(frontier)上一定包含最优解(Lemma 4)。

**跟量化的兼容性：** 论文专门测试了MoDES配合MC-MoE的混合精度量化策略一起用(Table 2)，发现量化对MoDES造成的性能损失远小于对MC-MoE本身造成的损失——说明MoDES选出来的、被保留激活的专家，本身对量化噪声更鲁棒。

### 4. 最大难点在哪里

不在GMLG或DMT这两个机制本身(思路比较直观)，难点在于**frontier search算法的正确性证明**——要严格证明"在f/g关于阈值单调"这个假设下，最优解一定落在某条可以用线性时间定位的边界上，需要完整的数学论证(Appendix A的Lemma 1-4、Proposition 1-2)，这部分是全文工程严谨度最高的地方，也是这篇论文能把搜索时间从几天压到几小时的根本原因。

### 5. 局限 / 对我项目的相关性

**只在通用图文/视频理解基准上验证过**，没有在驾驶/具身智能这类场景做过实验，驾驶场景的token类型(BEV token、语言指令token、动作token)跟"文本/视觉"这个二分法不完全对应，直接套用需要重新设计模态划分。

**校准仍需要offline阶段**(1024个样本、几小时到4小时不等)，不是零成本的，虽然比naive search快很多。

**依赖"性能/效率关于阈值单调"这个假设**，论文在自己的场景下验证了这个假设合理，但没有讨论什么情况下这个假设可能不成立。

**对我项目的相关性：这是回应"SA-MoE/驾驶MoE没有真正省计算量"这个局限最直接的参考**——如果要给SAMoE-VLA或PerceptDrive这类驾驶MoE/路由模型加推理加速，可以尝试把MoDES的框架迁移过去：把"文本/视觉"模态划分换成驾驶场景里更自然的划分(比如"语言指令token/BEV感知token/自车状态token/动作token")，分别设计跳过阈值；"浅层更关键、深层可以更激进地跳"这个layer-wise insight也值得在驾驶MoE模型上重新验证一遍，看是否同样成立。

### 6. 是否能连 GitHub

**能，代码已开源**：https://github.com/ModelTC/MoDES

---

## 假设、局限与质疑点

- **"浅层更关键"这个结论是在通用MLLM(文本+图像/视频理解)上验证的**，驾驶场景里"浅层"和"深层"各自承担的角色可能不同(比如浅层可能更偏重原始感知特征提取，深层更偏重决策推理)，这个insight能否直接套用需要重新验证。
- **frontier search的单调性假设**虽然有数学证明支撑最优性，但这个证明建立在"假设成立"的前提下，论文没有讨论万一某个新的MoE MLLM架构不满足这个单调性会怎样。
- **校准集只用了GQA一个数据集**(1024样本)，论文附录有讨论不同数据集/样本量的消融(Appendix F)，但本笔记还没细读那部分。

---

## 一个月后记住 5 件事

1. **核心论点：直接把纯文本LLM的专家跳过方法套用到MLLM上会严重掉分**(跳过83%专家精度掉超10%)，根源是忽略了层级全局重要性差异和模态行为差异。
2. **两个insight：浅层专家更关键(深层误差会被浅层误差放大)；视觉token比文本token更能容忍专家被跳过**(FFN对视觉token更新幅度更小，表示更正交)。
3. **MoDES两大机制：GMLG(全局层重要性×局部路由概率)算token重要性分数；DMT(双模态独立阈值)按模态区别对待**。
4. **Frontier search把阈值搜索从O(ND²)降到O(ND)**，搜索时间从几天压到几小时，有完整数学证明支撑。
5. **跳过88%专家反而性能提升(97.33% vs 86.66%)**，证明某些专家确实是冗余/有害的，不是单纯的"精度-效率权衡"，代码已开源(github.com/ModelTC/MoDES)。

---

## 关键原文摘录(中英对照)

**① 核心问题诊断**
> "we find that applying these methods—originally designed for unimodal large language models (LLMs)—to MLLMs results in considerable performance degradation. This is primarily because such methods fail to account for the heterogeneous contributions of experts across MoE layers and modality-specific behaviors of tokens within these layers."

中译：我们发现，把这些最初为单模态大语言模型设计的方法应用到MLLM上，会导致显著的性能下降。这主要是因为这些方法没有考虑到MoE各层之间专家贡献的异质性，以及这些层内token的模态特定行为。
解读：这是全文的核心问题诊断，presentation开场直接用这句话能把问题讲清楚。

**② 核心结果数字**
> "when skipping 88% experts for Qwen3-VL-MoE-30B-A3B-Instruct, the performance boost is up to 10.67% (97.33% vs. 86.66%)."

中译：在Qwen3-VL-MoE-30B-A3B-Instruct上跳过88%的专家时，性能提升最高达10.67%(97.33% vs 86.66%)。
解读：这个数字很反直觉——跳掉88%的专家，性能不但没掉，反而比基线方法更高，说明"该不该跳"这件事判断准了，比"跳多少"本身更重要。

---

## English at-a-glance summary

| Aspect | Summary |
|---|---|
| **Problem** | Expert-skipping methods designed for text-only LLMs cause severe accuracy drops when applied directly to MoE MLLMs.<br>为纯文本LLM设计的专家跳过方法，直接用到MoE MLLM上会导致精度大幅下降。 |
| **Novelty** | GMLG (global layer-wise importance × local routing probability) + DMT (separate thresholds per modality) + a frontier search algorithm with proven O(ND) complexity for threshold optimization.<br>GMLG(全局层级重要性×局部路由概率)+DMT(按模态分别设阈值)+有理论证明的O(ND)复杂度阈值搜索算法。 |
| **Core idea** | Shallow layers matter more (error explosion), and vision tokens tolerate expert skipping better than text tokens (smaller, more orthogonal FFN updates) — expert skipping must be layer- and modality-aware.<br>浅层更关键(误差会被放大)，视觉token比文本token更能容忍专家跳过(FFN更新幅度更小、更正交)——专家跳过必须对层和模态都敏感。 |
| **Hardest part** | Proving the frontier search algorithm's correctness and O(ND) complexity under a monotonicity assumption.<br>在单调性假设下证明frontier search算法的正确性和O(ND)复杂度。 |
| **Evidence** | Skipping 88% of experts on Qwen3-VL-MoE-30B-A3B-Instruct improves retained performance from 86.66% to 97.33%; ~45x faster threshold search than naive search; compatible with quantization.<br>在Qwen3-VL-MoE-30B-A3B-Instruct上跳过88%专家，保留性能从86.66%提升到97.33%；阈值搜索比naive search快约45倍；跟量化兼容性好。 |
| **Limitation** | Only validated on general image/video understanding benchmarks, not on driving/embodied scenarios; still requires an offline calibration stage.<br>只在通用图文/视频理解基准上验证，没在驾驶/具身场景测过；仍需要离线校准阶段。 |
| **Code / GitHub** | Released: github.com/ModelTC/MoDES<br>已开源：github.com/ModelTC/MoDES |

---

## 本文提到的词

| 术语 | 首次出现位置 |
|------|------------|
| Expert skipping(专家跳过) | Abstract |
| Globally-Modulated Local Gating(GMLG，全局调制的局部门控) | Abstract |
| Dual-Modality Thresholding(DMT，双模态阈值) | Abstract |
| Frontier search(边界搜索) | Abstract |
| Error explosion(误差爆炸) | §4.1 |
| Modality gap(模态差异) | §4.2 |
