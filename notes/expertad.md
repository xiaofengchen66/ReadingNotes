# ExpertAD: Enhancing Autonomous Driving Systems with Mixture of Experts

- **Venue / year:** arXiv 2511.11740, 2025-11-13投稿；**已确认被AAAI 2026(第40届)接收**
- **Authors:** Haowen Jiang, Xinyu Huang, You Lu, Dingji Wang, Yuheng Cao, Chaofeng Sha, Bihuan Chen, Keyu Chen, Xin Peng — 复旦大学计算机科学与人工智能学院
- **Link:** https://arxiv.org/abs/2511.11740 ([本地PDF副本](pdf/expertad.pdf))
- **Code:** 未查到开源仓库，暂按未开源处理
- **Input / 输出:** 多视角摄像头图像(经BEV Encoder编码) → 碰撞无关的安全轨迹
- **精读说明(2026-10-22)：用户指出之前摘要级笔记对架构的理解有误(以为PA和Planning段也是MoE)，已完整读过PDF正文Methodology+Fig.2纠正。结论先说在前面：ExpertAD的abstract写"end-to-end MoE architecture that spans perception and planning"，但实际架构里真正的MoE(稀疏专家+路由)只存在于Prediction模块，Perception段(PA)是特征重加权、不是MoE，Planning段直接复用基线解码器、也不是MoE——这是单段MoE，不是"全程"。**

---

## TL;DR

**问题 →** 复杂驾驶场景里语义信息丰富，但模糊/有噪声的语义会损害决策可靠性；多个驾驶任务(跟踪/建图/自车状态/导航)之间的相互干扰会损害规划效果；长推理延迟会拖慢决策、增加不安全驾驶风险。

**Gap →** 已有MoE用在自动驾驶里的工作，要么只用在感知段(摄像头选择、视觉token选择)，要么只用在规划段(轨迹选择、驾驶意图预测)，缺乏统一的语义/行为建模，对噪声很敏感，而且通常会引入额外的推理开销。

**Novelty →** 提出ExpertAD框架：**Perception Adapter(PA)**——不改变原感知模块结构，而是在BEV特征和跟踪/建图transformer之间插入一层，动态挑选+放大任务相关的特征通道；**Mixture of Sparse Experts(MoSE)**——把Prediction模块改造成8个稀疏专家(按环境/自车状态/导航三类)+噪声门控Top-K路由器，动态激活最相关的专家。整个框架保留原ADS(UniAD/VAD/VADv2)的BEV Encoder和Planning模块不变，只改造中间的感知和预测这两段。

**核心洞察 →** 不是所有驾驶任务在所有场景下都同等重要(比如建图任务对弯道规划有用、对直道规划没用)，所以"全部任务全部激活"是没必要的计算浪费，动态选择任务相关的特征/专家可以同时提升效果和效率。

**方法/论证 →** PA用约束优化(entropy正则化的软阈值选择，公式1-3)动态加权BEV特征的通道维度，输出结合跟踪agent query和建图map query的ego query；MoSE的8个专家各自用不同的稀疏注意力模式(环境专家用block-wise稀疏、自车状态专家用滑窗、导航专家用全局Top-K)，Router用噪声门控(公式6)+Top-K选择(公式7，实验里Top-4最优)+switch-loss(公式9，防止路由坍缩到少数专家)。

**结果 →** 集成进UniAD/VAD/VADv2三个基线后，碰撞率平均降低约20%(各基线14-23%不等)，推理延迟降低约25%(1.2-1.4倍加速)，同时FLOPs降低，是"效果和效率同时提升"而不是常见的"牺牲一个换另一个"。闭环Bench2Drive评测在Emergency Braking/Merging/Traffic Signs等场景组里效果提升明显。

**意义 →** 这是一篇扎实的工程论文，证明了"给Prediction模块加稀疏MoE+给Perception模块加轻量特征重加权"这个组合能同时改善效果和延迟，但**不是**一个真正横跨感知-预测-规划三段的"全程MoE"——这点跟abstract的表述有出入，细读架构图才能发现。

---

## 贡献列表(按重要性排序)

**① Mixture of Sparse Experts(MoSE)：Prediction模块的真正MoE改造**
这是论文里唯一真正意义上的MoE：8个稀疏专家分三类——**环境专家(Environmental Experts，2个)**处理跟踪/建图输出的环境信息(交通信号、车道线、障碍物)，用block-wise稀疏注意力(限制在局部区域)；**自车状态专家(Ego State Experts，3个)**：速度专家、偏航角专家、加速度专家，协同处理车辆动力学，用滑窗注意力(窗口大小w)；**导航专家(Navigation Experts，3个)**：参考点专家、BEV专家、指令专家，做层级式路径规划，用全局Top-K稀疏注意力。Router用公式6(噪声门控：g(x)=x·W_gate+η·softplus(x·W_noise)，η~N(0,1))计算路由logits，公式7(TopK(softmax(g(x)),k))选出Top-k专家，实验显示**Top-4效果最好**(Table 4：DS 49.32 vs 全激活Top-8的46.24 vs 不用路由的baseline 44.62)。switch-loss(公式9)防止路由坍缩到少数专家。消融(Table 6)显示Router本身对L2误差和碰撞率的改善比稀疏注意力本身更大，稀疏注意力主要贡献在降延迟。

**② Perception Adapter(PA)：不是MoE，是任务感知的特征通道重加权**
给每个任务(跟踪、建图)引入一个可学习参数w^(t)，先算一个任务感知的通道重要性分数(公式1)，再通过熵正则化的约束优化问题(公式2)求解出[0,1]区间的软通道选择权重λ(保证τ个关键通道被聚焦、同时保持平滑性，而不是硬性的0/1选择)，最后用这组权重对BEV特征做重加权、过MLP、再残差加回原特征(公式3)，得到增强后的特征Falign，分别喂给跟踪transformer和建图transformer。τ=128时效果最好(Table 3：DS 52.53，比baseline UniAD的44.62提升17%)。**这整个过程没有专家池、没有稀疏门控路由选择机制**，是条件化的特征选择，不是MoE。

**③ 联合训练损失+消融验证各组件独立必要**
总损失(公式10)是感知损失+预测损失+规划损失+switch-loss的加权和。消融研究分别验证了PA(Table 3/5)和MoSE(Table 4/6)各自的贡献：PA的MLP层能提升AMOTA从0.388到0.390(加法连接比直接替换更稳)；MoSE的Router贡献比稀疏注意力更大(前者改善精度，后者改善延迟)。

**④ 跨城市泛化评测**
在Boston训练、Singapore测试(反之亦然)，Expert-UniAD在两个方向上都比UniAD基线泛化更好(碰撞率-17%或L2误差-13%)，证明改造后的框架不是"过拟合训练分布"的提升。

---

## Novelty 分类判断

- **新问题定义**：不算——"MoE用在自动驾驶里"这个大方向此前已有工作(camera selection、trajectory selection等)，这篇是把MoE系统性应用到Prediction模块这个此前较少被MoE化的环节。
- **新insight**：部分算——"不是所有驾驶任务在所有场景下都同等重要，动态选择比全部激活更好"这个论点有消融支撑(Table 4的Top-4 vs Top-8对比)，但这个insight本身并不新颖(是MoE的通用动机)，这篇是在Prediction这个具体模块上验证。
- **新方法**：算——MoSE的三类专家设计(环境/自车状态/导航，各配不同的稀疏注意力模式)、PA的熵正则化软通道选择，都是具体的工程创新。
- **新理论**：没有，是已有技术(Shazeer式噪声门控MoE、switch-loss、条件adapter)的组合应用。
- **新数据集**：没有，用nuScenes(开环)和Bench2Drive(闭环)已有基准。
- **新应用场景**：不算，是对已有ADS(UniAD/VAD/VADv2)的插件式改造，不是全新系统。

综合结论：**具体工程方法组合类贡献**，核心价值是证明"给预测模块加稀疏MoE+给感知模块加轻量特征重加权"这个具体组合能同时改善碰撞率和延迟，论证严谨(消融充分、三个基线+跨城市泛化都测了)，但**不要被abstract的"spans perception and planning"表述误导**——实际只有预测段是真MoE，这篇不构成"全程MoE"的证据。

---

## 中文精读

### 1. 解决了什么问题

传统端到端ADS(UniAD/VAD/VADv2这类)按感知→预测→规划顺序模块化处理，但存在两个问题：(1)不同驾驶任务(跟踪、建图、自车状态估计、导航)之间会相互干扰，比如建图任务对弯道规划有用但对直道规划是噪声；(2)全部任务全部激活会增加不必要的计算延迟。ExpertAD要回答：**能不能在不改变原有感知-预测-规划大框架的前提下，通过动态选择"该激活哪些特征/专家"，同时改善规划效果和推理效率？**

### 2. 新意在哪里

核心新意不在"要不要用MoE"(这个判断不难想到)，而在于**精确定位MoE该用在哪里、不该用在哪里**：作者发现直接把MoE套用到感知段(camera selection类工作)或规划段(trajectory selection类工作)，要么对噪声敏感，要么引入额外推理开销，于是选择只在**预测段**(协调跟踪/建图/自车状态/导航多个子任务交互的地方)用真正的稀疏MoE，感知段只做轻量的特征重加权(不改变原有感知头结构，避免引入新的不稳定性)，规划段完全复用(避免破坏已验证的规划器)。

### 3. Idea 具体落在哪里

**整体架构(Fig. 2)：** 原始ADS的BEV Encoder直接复用(Reuse标注)。BEV特征喂给PA：Learned Adapter(LA)算任务感知的通道选择分数并求解软权重λ，Alignment Layer(AL)用λ重加权BEV特征生成对齐特征，分别过Tracking Head和Mapping Head得到agent query和map query，两者拼接+一个可学习embedding构成ego query。ego query喂给MoSE：8个稀疏专家(3类)各自处理，Router根据ego query从8个专家里选Top-k(实验显示Top-4最优)并加权融合，输出Motion Query。Motion Query直接传给原始Planning模块(完全复用，Reuse标注)解码最终轨迹。

**PA的具体机制：** 不是MoE。公式1算每个任务t的通道重要性分数s^(t)(BEV特征和任务专属可学习参数w^(t)的内积，空间维度上平均)；公式2是一个带熵正则化的约束优化(max λ^T s + εΩ(λ)，约束1^Tλ=τ、λ∈[0,1]^d)，用迭代算法求出软的、可微的通道选择权重(不是硬性0/1选择，而是让τ个关键通道被突出、同时保持平滑)；公式3用这组权重重加权BEV特征、过MLP、残差加回原特征。这整个过程是单一确定性的特征变换(给定输入，输出是唯一确定的)，不涉及"从多个候选专家里选择并组合"这个MoE的核心特征。

**MoSE的具体机制：** 这才是真正MoE。8个专家分三类，每类用不同的稀疏注意力模式(公式4，MHCA稀疏多头交叉注意力，C_i定义每个专家的稀疏计算范围)：环境专家用block-wise局部稀疏(C_i={j|⌊j/m⌋=⌊i/m⌋})、自车状态专家用滑窗(C_i=[i-w,i+w])、导航专家用全局Top-K(C_i={j|K_j∈Top_k(Q_iK^T)})。每个专家的输出跟ego query、专家自身的embedded query一起再过一次MHCA(公式5)融合成专家特定的query。Router(公式6-7)是标准的Shazeer式噪声门控Top-K选择，switch-loss(公式9)按查询负载惩罚过载的专家，保证路由不坍缩。最终Motion Query(公式8)是Top-k个被选中专家的路由分数加权求和。

**训练：** 总损失(公式10)是感知+预测+规划损失(来自原基线ADS)加switch-loss的加权和，四个系数α1-α4。

### 4. 最大难点在哪里

不在MoSE这部分(标准MoE设计思路搬过来问题不大)，难点在**PA这部分怎么做到"软性、可微、不破坏原有感知头结构"的特征选择**——如果用硬性的0/1通道选择(传统方法)，会引入不可微的离散决策，难以端到端训练；这篇用熵正则化约束优化求出连续的软权重，公式2那个优化问题"没有闭式解"(论文原文承认)，要用迭代算法(借鉴Lei et al. 2023的conditional adapter工作)求次可微的软权重——这是全文工程上最精细的部分，但也正因为PA是这样一个"软选择+残差"的设计，它本质上更接近注意力/门控机制，而不是MoE。

### 5. 局限 / 对我项目的相关性

**代码未开源**，查不到对应仓库。

**最重要的局限(本次精读新发现，之前摘要级笔记没抓到)：这不是一个"全程MoE"。** Abstract写"ExpertAD achieves both effectiveness and efficiency through an end-to-end MoE architecture that spans perception and planning"，这句话容易让人以为感知和规划两段也用了MoE，但细读Methodology和Fig.2会发现：**PA(感知段)是特征重加权，不是MoE；Planning模块完全复用原始解码器，也不是MoE；真正的稀疏专家+路由只存在于Prediction模块。** 这是论文表述(abstract宣传语)和实际架构之间的一个出入，引用这篇论文时要注意别被abstract误导，复述成"ExpertAD在感知规划全流程用了MoE"是不准确的。

**对我项目的相关性：** 这个发现直接影响"全程MoE有没有人做过"这个判断——此前因为只看摘要，错误地以为ExpertAD可能是感知+规划两段都有MoE的候选；精读后确认它实际只是单段(预测)MoE，连DriveMoE那种两段(感知+规划)都不如。这进一步巩固了"真正三段(感知+预测+规划)都做MoE路由，目前没人做过"这个结论——查了96+篇、精读了这几篇重点候选，这个空子依然成立。

### 6. 是否能连 GitHub

未开源，论文和补充材料均未提供代码链接。

---

## 假设、局限与质疑点

- **Abstract表述("spans perception and planning")与实际架构不符**，这是presentation时必须主动澄清的一点，不能直接照搬abstract的说法。
- **PA的τ(关键通道数)是人工搜索出的超参数**(τ=128最优，Table 3)，不是自适应学出来的，不同场景/不同基线下τ的最优值是否会变化，论文没有展开讨论。
- **MoSE的8专家、3类划分(环境/自车状态/导航)是人工预先设计的**，不是数据驱动自动聚类或学出来的划分方式，划分合理性没有跟其他划分方式(比如ExpertAD自己也可以试试按场景类型划分)做对比消融。

---

## 一个月后记住 5 件事

1. **核心澄清：ExpertAD的MoE只在Prediction模块，不是"感知+预测+规划全流程MoE"**——Abstract的"spans perception and planning"表述有marketing夸大成分，PA(感知)是特征重加权不是MoE，Planning完全复用基线解码器。
2. **MoSE(预测段MoE)**：8个稀疏专家分环境/自车状态/导航三类，各用不同稀疏注意力模式，噪声门控Top-K路由(Top-4最优)+switch-loss防坍缩。
3. **PA(感知段，非MoE)**：熵正则化约束优化求软通道选择权重，重加权BEV特征，没有专家池和路由选择机制。
4. **核心结果：碰撞率平均降20%，推理延迟降25%，效果和效率同时提升**(不是常见的"牺牲一个换另一个"的权衡)，三个基线(UniAD/VAD/VADv2)+跨城市泛化都验证过。
5. **对"全程MoE"研究空缺判断的影响：这篇精读后进一步确认该空缺依然成立**——ExpertAD实际只是单段MoE，比最初摘要级印象里的"可能跨两段"更弱。

---

## 关键原文摘录(中英对照)

**① Abstract的表述(容易引起误解的地方)**
> "ExpertAD achieves both effectiveness and efficiency through an end-to-end MoE architecture that spans perception and planning."

中译：ExpertAD通过一个横跨感知和规划的端到端MoE架构，同时实现了有效性和高效性。
解读：这是abstract里容易让人误以为感知和规划段也是MoE的一句话，实际架构(见下一条引文)并非如此，presentation时要主动澄清这个出入。

**② Methodology明确说明实际改造范围**
> "ExpertAD is built upon ADS models by retaining all the original modules except for the perception and prediction modules. The perception module is restructured as the Perception Adapter (PA) to amplify task-critical features... The prediction module is transformed into the Mixture of Sparse Experts (MoSE)..."

中译：ExpertAD建立在ADS模型之上，保留了除感知模块和预测模块之外的所有原始模块。感知模块被重构为Perception Adapter(PA)以放大任务关键特征……预测模块被改造成Mixture of Sparse Experts(MoSE)……
解读：这句话(Fig.2图注)才是准确描述——明确说"除了感知和预测模块，其余都保留"，而感知模块被重构成PA(特征放大，非MoE)，只有预测模块被改造成MoSE(真正的MoE)。规划模块("Planning module")在图上明确标注"Reuse"，完全复用原基线。

**③ Router消融证明路由本身的价值**
> "MoSE with Router routes the ego query to the top-4 experts, while MoSE without Router activates all experts. ... Router notably enhances planning metrics, while Sparse Attention significantly reduces latency."

中译：带Router的MoSE把ego query路由给Top-4专家，不带Router的MoSE激活所有专家……Router显著提升规划指标，而稀疏注意力显著降低延迟。
解读：这条消融(Table 6)把"路由选择"和"稀疏注意力"两个机制的贡献拆开看，证明两者分别贡献了不同的收益(精度 vs 速度)，是全文消融最清晰的一条证据。

---

## English at-a-glance summary

| Aspect | Summary |
|---|---|
| **Problem** | Driving tasks (tracking, mapping, ego-state, navigation) interfere with each other, and activating all of them for every scenario wastes computation and hurts planning reliability.<br>驾驶任务(跟踪/建图/自车状态/导航)之间会相互干扰，每个场景都全部激活既浪费计算又损害规划可靠性。 |
| **Novelty** | A Perception Adapter (soft, differentiable channel reweighting — NOT MoE) for the perception stage, and a Mixture of Sparse Experts (genuine sparse routing, 8 experts in 3 categories) for the prediction stage only; the planning stage is left untouched.<br>感知段用Perception Adapter(软性可微通道重加权，不是MoE)，预测段用Mixture of Sparse Experts(真正的稀疏路由，8个专家分3类)，规划段完全不动。 |
| **Core idea** | Precisely locating where MoE helps (the multi-task-interference point in prediction) versus where lighter mechanisms suffice (perception) avoids both noise sensitivity and unnecessary overhead.<br>精确定位MoE该用在哪(预测段的多任务干扰点)、哪里用更轻量的机制就够了(感知段)，避免了对噪声敏感和不必要的开销。 |
| **Hardest part** | Making the Perception Adapter's channel selection soft and differentiable (no closed-form solution, requires an iterative solver) without disrupting the original perception head.<br>让Perception Adapter的通道选择保持软性、可微(没有闭式解，需要迭代求解)，同时不破坏原有感知头结构。 |
| **Evidence** | ~20% average collision-rate reduction and ~25% latency reduction across three baselines (UniAD/VAD/VADv2), plus cross-city generalization gains.<br>三个基线(UniAD/VAD/VADv2)上平均碰撞率降约20%、延迟降约25%，另有跨城市泛化提升。 |
| **Limitation** | **The abstract's "spans perception and planning" is misleading** — only the prediction module is genuine MoE; the perception module is feature reweighting, and the planning module is entirely reused from the baseline. Not evidence of a true "full-pipeline MoE."<br>**Abstract"横跨感知和规划"的表述容易误导**——只有预测模块是真正的MoE，感知模块是特征重加权，规划模块完全复用基线。不构成"全程MoE"的证据。 |
| **Code / GitHub** | Not released, no repository found.<br>未开源，未查到仓库。 |

---

## 本文提到的词

| 术语 | 首次出现位置 |
|------|------------|
| Perception Adapter(PA) | Abstract |
| Mixture of Sparse Experts(MoSE) | Abstract |
| Learned Adapter(LA) | §Perception Adapter |
| Alignment Layer(AL) | §Perception Adapter |
| Environmental Experts / Ego State Experts / Navigation Experts | §Mixture of Sparse Experts |
| Switch-loss | §Training Loss |
| Noisy gating(噪声门控) | Eq. 6 |
| MHCA(稀疏多头交叉注意力) | Eq. 4 |
