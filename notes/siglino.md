# SigLino: Efficient Multi-Teacher Distillation for Agglomerative Vision Foundation Models

- **Venue / year:** arXiv 2512.20157, 2025 (CVPR 2026)
- **Authors:** Sofian Chaybouti, Sanath Narayan, Yasser Dahou, Phúc H. Lê Khắc, Ankit Singh, Ngoc Dung Huynh, Wamiq Reyaz Para, Hilde Kuehne, Hakim Hacid — Technology Innovation Institute (Abu Dhabi) + Tübingen AI Center + MIT-IBM Watson AI Lab
- **Link:** https://arxiv.org/abs/2512.20157 ([本地PDF副本](pdf/siglino.pdf))
- **Code:** 项目页 sofianchay.github.io/amoe，论文写"We release OpenLVD200M and five distilled checkpoints comprising MoE and dense variants"——写之时未单独核实GitHub仓库是否已公开
- **Input / 输出:** 原始图像(以及grounding微调阶段的文本expression) → 统一的视觉表征(patch token + 全局CLS token)，可直接用于分类/检索/分割/grounding等下游任务
- **精读说明(2026-10-21)：用户让我判断哪些论文最像朱子欣"全程MoE+蒸馏别的MoE当冻结专家训练轻量路由"这个idea，这篇是目前查到的机制最接近的一篇，已完整读过PDF正文，不再是摘要级。**

---

## TL;DR

**问题 →** 聚合视觉基础模型(Agglomerative VFM，把多个基础模型的优势蒸馏进一个统一视觉主干)是条有前景的路线，但训练开销很大，多教师蒸馏的学习动态和数据效率一直没被系统研究过。

**Gap →** 已有的聚合VFM工作(如AM-RADIO)证明了这个方向可行，但需要海量训练样本(RADIO用了1.1万亿image token)，而且怎么处理不同分辨率教师、怎么设计多个损失函数，都没有标准化的方法论。

**Novelty →** 提出SigLino：一套系统化的高效多教师蒸馏配方，核心是四点——①非对称关系知识蒸馏损失(ARKD)保留每个教师的几何结构同时实现有效知识迁移；②token平衡批处理(把不同分辨率图像打包进统一token预算的序列)稳定多分辨率训练；③分层聚类+采样(原本是自监督学习专用的技巧)用在训练数据构建上，显著提升多教师蒸馏的样本效率；④证明这套蒸馏出的表征能有效迁移到early-fusion grounding VLM，而且**学生架构本身可以是Dense也可以是MoE**。

**核心洞察 →** 多教师蒸馏不是"数据越多越好"，而是数据的**均匀概念覆盖**(而非随机采样)+**训练稳定性**(token平衡)+**保留教师间相对几何关系**(而非只对齐绝对位置)这三件事同时做对，才能用远少于RADIO的数据量做到更好。

**方法/论证 →** 两个教师：SigLIP2(ViT-L，图文对齐强但稠密特征不可分)和DINOv3(ViT-L，自蒸馏+Gram-anchoring训练，稠密特征质量极高)。学生backbone(可以是Dense FFN或MoE)输出patch token+全局CLS token+register token，每个教师配一个独立的单层MLP投影头，把学生特征投影进该教师的嵌入空间；损失同时对齐CLS/patch/register三类表征。在此基础上，用蒸馏出的"聚合MoE"(AMoE)去**初始化**一个early-fusion grounding VLM里的模态专属视觉专家(vision experts)，并用Gram-anchoring防止下游微调时稠密特征质量退化。

**结果 →** 只用约2300亿image token(RADIO的1.1万亿的1/4.7)，SigLino-MoE(0.3B/0.6B总参数，6个活跃专家)在图文分类宏平均上超过RADIOv2.5-H(84.13 vs 82.26)，kNN分类也超过(88.06 vs 85.12)；蒸馏出的视觉专家初始化grounding VLM后，RefCOCO检测准确率从scratch训练的29.15提升到57.49(初始化)/61.06(初始化+Gram-anchoring)。

**意义 →** 这是目前查到的、**明确验证"多教师蒸馏→MoE学生→用MoE的专家去初始化下游任务模型"这整条链路可行**的论文——跟朱子欣说的"蒸馏别的模型、冻结当专家(或至少复用专家参数)、训练轻量路由"机制上高度重合，只是这篇蒸馏的教师是通用视觉基础模型(SigLIP2/DINOv3)，不是别的驾驶专用MoE系统(DriveMoE/SAMoE-VLA)，也完全不是驾驶场景。

---

## 贡献列表(按重要性排序)

**① 非对称关系知识蒸馏(ARKD)**
标准做法只对齐学生和教师的"一对一"表征(比如CLS对CLS)，这篇额外引入关系蒸馏(Relational KD)：匹配批内样本两两之间的相对几何关系。但作者发现直接套用对称的RKD会损害kNN聚类质量(把该分开的样本硬拉近)，于是设计了非对称版本：只在教师认为"该近/该远"时才惩罚学生偏离，用教师空间里距离的批内中位数当决策边界，分成expand(该远却近)和shrink(该近却远)两种情况分别加权。消融显示(Table 4)ARKD对DINOv3对齐提升最大(Img-Text Avg从77.48到78.21，kNN从81.36到83.63)，且不像对称RKD那样伤害kNN。

**② Token平衡批处理**
原生分辨率训练时每张图的patch数差异极大(256×256约256个patch，768×768约2304个)，朴素按图片数量定批大小会导致跨rank的token量严重不均衡、产生高范数梯度。解法：用FlexAttention把多张不同分辨率图像打包进固定token预算的序列(避免图像间互相attend)，并对每张图的patch/register损失做per-image token数归一化，再在全局批次上无偏聚合。效果：训练速度从7.5k token/s提到20k token/s，同时防止低分辨率表征被遗忘。

**③ 分层聚类+采样构建训练数据(OpenLVD200M)**
这项原本是自监督学习(SSL)数据构建的技巧(用于DINOv3的训练数据LVD-1.7B)，这篇首次证明对多教师蒸馏同样关键：从23亿图像(DFN+LAION混合)里，先用DINOv3编码，做4层分层聚类(20M/500K/50K/20K质心)，再分层采样出均匀覆盖概念空间的2亿图像子集。消融(Table 7)显示比等量随机采样平均准确率+4.15，细粒度/长尾数据集(FGVC-Aircraft)上+18.64。

**④ 蒸馏表征能有效初始化下游MoE grounding VLM**
用蒸馏出的AMoE视觉专家去初始化一个轻量0.2B-1B、12层、28个模态专属专家(6个活跃)+8个共享专家(2个活跃)的early-fusion grounding VLM。蒸馏初始化相比从零训练，RefCOCO检测准确率几乎翻倍(29.15→57.49)，配合Gram-anchoring(防止微调时稠密特征塌缩)进一步提升到61.06。

---

## Novelty 分类判断

- **新问题定义**：部分算——"多教师蒸馏的数据效率/训练动态"此前没被系统研究过，这篇算是系统化填补。
- **新insight**：算——"均匀概念覆盖的数据采样对多教师蒸馏同样关键"(此前只知道对SSL关键)有明确消融支撑。
- **新方法**：算——ARKD(非对称关系蒸馏)、token平衡批处理两个具体机制都是这篇的工程创新，此前没见过完全一样的设计。
- **新理论**：没有，是已有技术(RKD、FlexAttention、分层聚类)的巧妙组合+系统化配方。
- **新数据集**：算——OpenLVD200M(2亿图像精选子集)是新贡献，但构建方法论借用自DINOv3的LVD-1.7B。
- **新应用场景**：不算，是方法论文，但"蒸馏出的MoE专家可以初始化下游grounding VLM"这个应用验证是新的。

综合结论：**系统化方法论+具体机制组合类贡献**，核心价值在于把"多教师蒸馏"从一个能跑通但很贵的流程，变成一套有理有据的高效配方，并且是目前查到的少数**明确验证蒸馏进MoE学生、再复用MoE专家初始化下游模型**这条完整链路可行的工作。

---

## 中文精读

### 1. 解决了什么问题

"聚合视觉基础模型"(Agglomerative VFM)这条路线，想把多个各有所长的基础模型(比如一个擅长图文对齐、一个擅长稠密特征)统一蒸馏进一个主干网络里，让一个模型同时继承多个"专长"。此前的代表工作AM-RADIO证明了这条路可行，但训练代价极其昂贵(1.1万亿image token)，而且没人系统研究过：数据该怎么选、不同分辨率教师该怎么处理、损失函数该怎么设计，才能既省训练量又不损失表征质量。

### 2. 新意在哪里

核心新意是**把"多教师蒸馏"这件事拆解成四个可以分别诊断、分别优化的子问题**(数据覆盖、训练稳定性、关系保真、下游迁移)，每个子问题都给出具体机制和消融证据，而不是像此前工作那样堆数据硬train。

### 3. Idea 具体落在哪里

**整体架构(Figure 1)：** 左边是多教师蒸馏(MT-Distillation)阶段——输入图像同时喂给SigLIP2和DINOv3两个冻结教师，学生backbone(图上标注可以是Dense或MoE)输出patch token+CLS token(还有register token，借鉴DINOv3)，每个教师各配一个单层MLP投影头把学生特征投影进该教师的嵌入空间，损失同时对齐CLS/patch/register。右边是用蒸馏出的AMoE去初始化一个early-fusion grounding VLM，视觉专家和文本专家分别处理图文输入，做检测/分割grounding任务。

**MoE设计：** 这篇原文明确写"a Mixture-of-Experts (MoE) student is distilled from multiple frozen vision teachers"——学生backbone本身可以配置成MoE结构(SigLino-MoE：18层，28个专家6个活跃，768维embedding，0.3B活跃/0.6B总参数)，用的是**跟Dense变体完全相同的多教师蒸馏训练配方**，只是backbone换成了MoE层。消融(Table 1)显示SigLino-MoE(0.23万亿token训练量)跟SigLino-Dense表现相当接近，且都显著超过RADIOv2.5-H(用了1.1万亿token)。

**下游复用机制：** 蒸馏完的AMoE模型，其视觉专家权重被直接拿去**初始化**一个新的、专门做grounding任务的early-fusion VLM里的模态专属视觉专家槽位——这一步是整篇论文跟朱子欣idea最贴近的地方：先蒸馏出一组"通才"专家，再把这组专家的参数复用到一个新的下游专用模型里，而不是重新训练。为了防止这个初始化+微调的过程把稠密特征质量搞坏(论文观察到：微调时全局表征变好，但分割用的patch特征会退化，因为patch特征开始向CLS token对齐、丢失空间结构)，用了Gram-anchoring：让学生patch特征的Gram矩阵(衡量patch间两两相似度结构)尽量贴近蒸馏时冻结AMoE的Gram矩阵，从而在微调下游任务时依然保留蒸馏阶段学到的空间结构。

**训练分两阶段：** Stage 1在256×256分辨率上蒸馏5万步(快速学全局+稠密表征)；Stage 2在768×768(13M高分辨率图像，含SAM的11.5M和1.5M网络爬取)上继续训练9万步做高分辨率适配，用token平衡批处理+per-image归一化损失防止高分辨率图像主导梯度导致低分辨率表征被遗忘。

### 4. 最大难点在哪里

不在"要不要做多教师蒸馏"这个大方向判断上，难点在于**怎么证明每一个具体设计选择(ARKD、token平衡、数据采样)都是必要的，而不是"反正堆了很多技巧、刚好work"**。论文为此做了相当细致的消融(Table 4/7，以及正文里专门分析ARKD为什么对DINOv3收益大、对SigLIP2收益小的原因：DINOv3通过LiT流程"后验"对齐到文本，ground-truth图文相似度分数本身就更低，所以关系蒸馏作为正则化项更有用)——这种"每个技巧单独消融+给出直觉解释"的严谨度，是这篇方法论文比较扎实的部分。

### 5. 局限 / 对我项目的相关性

**完全不是驾驶场景**，教师是通用视觉基础模型(SigLIP2/DINOv3)，评测全是ImageNet/COCO/Flickr30k/ADE20k/Cityscapes/PASCAL-VOC这类通用视觉基准，没有任何驾驶相关实验。

**DINOv3的register token蒸馏有已知的不稳定性**：论文提到PHI-S(一种教师损失平衡/归一化技术)没法准确估计DINOv3第二个register token的变换，因为它表现出多峰统计特性，导致梯度范数高、训练变慢，最终论文选择不对register应用PHI-S——这是一个具体的、承认存在但没完全解决的技术局限。

**对我项目的相关性：** 这是目前查到的、明确验证"多教师蒸馏进MoE学生、再复用蒸馏出的专家初始化下游专用模型"这条完整链路可行的论文，跟朱子欣的idea机制高度重合。**如果要把这篇的配方直接搬到驾驶场景，核心要做的替换是**：把SigLIP2/DINOv3换成DriveMoE/SAMoE-VLA这类已训练好的驾驶专用MoE模型(或其中的专家子模块)当教师；ARKD的"保留教师间相对几何关系"这个设计思路，在驾驶场景下可能需要重新设计(因为驾驶MoE模型的\"几何\"指的是什么还不明确，不像视觉基础模型那样有清晰的patch空间结构)。

### 6. 是否能连 GitHub

项目页 sofianchay.github.io/amoe，论文写"We release OpenLVD200M and five distilled checkpoints"，撰写本笔记时未逐一核实每个checkpoint和代码仓库的实际开放状态。

---

## 假设、局限与质疑点

- **"均匀概念覆盖的数据采样对多教师蒸馏关键"这个结论目前只在这一篇、这两个教师(SigLIP2/DINOv3)组合上验证过**，换成其他类型的教师(比如驾驶专用MoE)是否依然成立，没有证据。
- **ARKD对两个教师的收益不对称**(DINOv3受益明显，SigLIP2收益边际)，论文给出了直觉解释但没有更深入的理论分析，presentation时要讲清楚这不是普适收益。
- **Gram-anchoring解决的是"下游微调会破坏蒸馏阶段学到的空间结构"这个具体问题**，但只在grounding(检测+分割)这一个下游任务上验证，泛化到其他下游任务(比如驾驶规划)是否同样需要/同样有效，未知。

---

## 一个月后记住 5 件事

1. **核心贡献：一套系统化的高效多教师蒸馏配方**(ARKD关系蒸馏+token平衡批处理+分层聚类数据采样)，用1/4.7的训练数据量超过此前SOTA(RADIOv2.5-H)。
2. **学生backbone既可以是Dense也可以是MoE**，用完全相同的蒸馏配方训练，MoE变体(SigLino-MoE)表现接近Dense变体，证明"蒸馏进MoE学生"这件事本身是可行的。
3. **蒸馏出的MoE专家可以直接拿去初始化一个全新的下游专用模型**(early-fusion grounding VLM)的对应专家槽位，初始化后RefCOCO检测准确率几乎翻倍——这是跟朱子欣idea最贴近的机制验证。
4. **Gram-anchoring是防止下游微调破坏蒸馏阶段空间结构的关键**，没有它初始化带来的收益会在微调过程中被侵蚀。
5. **教师依然是通用视觉基础模型，不是驾驶专用MoE**——如果要把这个配方搬到驾驶场景，核心空缺是把教师换成DriveMoE/SAMoE-VLA这类已有驾驶MoE系统。

---

## 关键原文摘录(中英对照)

**① 核心方法定位**
> "We introduce SigLino, an efficient family of agglomerative vision foundation models that distill knowledge from SigLIP2 and DINOv3 simultaneously into Dense and Mixture-of-Experts students."

中译：我们提出SigLino，一个高效的聚合视觉基础模型家族，同时从SigLIP2和DINOv3蒸馏知识，蒸馏目标既包括Dense学生也包括混合专家(MoE)学生。
解读：这句话直接确认了"蒸馏进MoE学生"是论文的核心设计，不是附带实验。

**② 下游复用机制**
> "Instantiated in a Mixture-of-Experts, our SigLino-MoE initializes an early-fusion Grounding-VLM that replaces the conventional ViT→LLM stack, demonstrating improved performance compared to a model trained from scratch."

中译：以混合专家形式实例化后，我们的SigLino-MoE用于初始化一个取代传统ViT→LLM架构的early-fusion Grounding-VLM，相比从零训练的模型表现有提升。
解读：这是"蒸馏出的专家被复用去初始化下游专用模型"这整条链路的直接文字证据。

**③ ARKD设计动机**
> "We hypothesize this is due to the loss aggressively pushing or attracting samples when they should be relatively far apart in the embedding space. We propose a simple fix: making RKD asymmetric (ARKD) by bringing two samples closer or pushing them only if they are close/far in teacher space."

中译：我们假设这是因为损失函数在样本本该相对远离时，依然激进地推近或拉远它们。我们提出一个简单的修复：让RKD变成非对称的(ARKD)，只在教师空间里样本确实近/远时，才把学生空间里对应的两个样本拉近/推远。
解读：这是ARKD机制的核心设计逻辑，解释了为什么对称的关系蒸馏会伤害kNN聚类质量。

---

## English at-a-glance summary

| Aspect | Summary |
|---|---|
| **Problem** | Agglomerative VFMs (unifying multiple foundation models via distillation) are promising but expensive, and the data efficiency of multi-teacher distillation is underexplored.<br>聚合视觉基础模型有前景但训练昂贵，多教师蒸馏的数据效率没被系统研究过。 |
| **Novelty** | A systematic recipe: asymmetric relational KD (ARKD) + token-balanced batching + hierarchically-sampled training data, validated on both Dense and MoE students, with the distilled MoE reused to initialize a downstream grounding VLM.<br>系统化配方：非对称关系蒸馏+token平衡批处理+分层采样数据，在Dense和MoE学生上都验证过，蒸馏出的MoE被复用去初始化下游grounding VLM。 |
| **Core idea** | Multi-teacher distillation quality depends on uniform concept coverage in training data, training stability across resolutions, and preserving teacher-relative geometry — not just raw data scale.<br>多教师蒸馏质量取决于训练数据的均匀概念覆盖、跨分辨率训练稳定性、保留教师相对几何关系——而不只是数据规模。 |
| **Hardest part** | Proving each design choice (ARKD, token balancing, data sampling) is independently necessary via ablation, not just a lucky combination.<br>通过消融证明每个设计选择(ARKD/token平衡/数据采样)都是独立必要的，而不是巧合凑效的组合。 |
| **Evidence** | SigLino-MoE surpasses RADIOv2.5-H (84.13 vs 82.26 avg image-text, 88.06 vs 85.12 kNN) using 4.7x fewer tokens; distilled initialization nearly doubles RefCOCO detection accuracy over scratch training (29.15→57.49, →61.06 with Gram-anchoring).<br>SigLino-MoE用4.7倍更少的token超过RADIOv2.5-H；蒸馏初始化让RefCOCO检测准确率比从零训练几乎翻倍。 |
| **Limitation** | Not driving-specific at all — teachers are generic foundation models, all benchmarks are general vision tasks. DINOv3 register-token distillation has known instability.<br>完全不是驾驶场景，教师是通用基础模型，全是通用视觉基准。DINOv3的register token蒸馏有已知不稳定性。 |
| **Code / GitHub** | Project page released (sofianchay.github.io/amoe); exact repo/checkpoint availability not individually verified at note-writing time.<br>项目页已发布；具体仓库/checkpoint可用性撰写本笔记时未逐一核实。 |

---

## 本文提到的词

| 术语 | 首次出现位置 |
|------|------------|
| Agglomerative Vision Foundation Model(聚合视觉基础模型) | Abstract |
| Multi-Teacher Distillation(多教师蒸馏) | Abstract |
| Asymmetric Relational Knowledge Distillation(ARKD，非对称关系知识蒸馏) | Abstract |
| Token-balanced batching(token平衡批处理) | Abstract |
| Hierarchical clustering and sampling(分层聚类与采样) | Abstract |
| Early-fusion Grounding-VLM | Abstract |
| OpenLVD200M | Abstract |
| Gram-anchoring | §3.5 |
| PHI-S(PCA–Hadamard Isotropic Standardization) | §3.2 |
| Relational Knowledge Distillation(RKD) | §2 Related Work |
