# SegMoTE: Token-Level Mixture of Experts for Medical Image Segmentation

- **Venue / year:** arXiv 2602.19213, 2026 (CVPR 2026 Oral)
- **Authors:** Yujie Lu, Jingwen Li, Sibo Ju, Yanzhou Su, He Yao, Yisong Liu, Min Zhu, Junlong Cheng — 四川大学 + 新疆大学 + 福州大学 + 阿里巴巴达摩院
- **Link:** https://arxiv.org/abs/2602.19213 ([本地PDF副本](pdf/segmote.pdf))
- **Code:** 已开源：https://github.com/InMyDreamer/SegMoTE
- **Input / 输出:** 医学影像(CT/MRI/X光/皮肤镜等任意模态)+可选的mask/文本提示 → 分割掩码
- **精读说明(2026-10-22)：用户让我把本地下载的论文都精读一遍，这是优先级较高的一篇("冻结大模型+轻量token级MoE插件"这个设计模式，跟PerceptDrive结构高度相似，已完整读过PDF正文。**

---

## TL;DR

**问题 →** 医学图像分割面临成像模态异质性(CT/MRI/X光等差异巨大)和像素级标注成本高两大挑战；通用交互式分割模型(SAM)迁移到医学影像时，现有适配方法要么对大规模异构数据集做全参数/部分参数微调(监督噪声大、成本高、容易负迁移)，要么缺乏模态/任务感知的自适应机制、跨模态泛化差。

**Gap →** 怎么高效利用自然图像预训练好的基础模型(SAM)，让它能适配到多种医学影像模态，同时不需要大规模微调、不破坏原有的zero-shot能力？

**Novelty →** 提出SegMoTE：冻结SAM的编码器和mask解码器主体，只引入一组可学习的"专家token"(expert tokens)，用token级的Mixture of Token Experts(MoTE)机制动态选择最适合当前输入的专家token，实现模态自适应表征，同时保留SAM原有能力和轻量架构；另外设计Progressive Prompt Tokenization(PPT)机制，用随机采样的mask/文本提示引导可学习query token逐步对齐前景/背景语义，实现无需人工交互的全自动二分类分割。

**核心洞察 →** 不需要重训或微调SAM本身的参数，只需要给它配一组token级的"专家插件"，就能让冻结的通用分割模型适应不同医学影像模态——把"模态适应"这件事完全限定在一组新增的轻量token上，而不是改动SAM本身的权重。

**方法/论证 →** 引入N×256维的可学习专家token，拼接进SAM原有的output token和prompt token序列，先在decoder的self-attention里跟其他token交互，再经token-to-image/image-to-token双向注意力融合视觉特征、prompt几何语义信息；之后送进MoTE：路由器算每个token对所有专家的logits(Eq.1)，训练时加噪声门控鼓励探索(Eq.2)，取top-k专家的置信分数(Eq.3-4)，经softmax得到路由权重(Eq.5)，加权更新选中的专家token表示(Eq.6)；负载均衡损失L_balance(Eq.7-9)用重要性和负载的变异系数平方和约束专家利用率均衡；Progressive Prompt Tokenization(PPT)随机采样mask/文本prompt当前景先验，让可学习query通过attention逐步捕捉前景/背景的分布线索，推理时不需要人工给prompt就能全自动分割(目前只支持二分类前景/背景任务)；总损失=Dice分割损失+λ·负载均衡损失(λ=0.01)。

**结果 →** 在自建的MedSeg-HQ数据集(整合12个公开数据集，154,569条高质量标注，只有现有大规模医学分割数据集约1%的规模)上训练，域内和域外数据集上都达到SOTA，比第二好的方法提升1%-6%；只用17M可训练参数(远小于全量微调SAM)、0.15M训练图像就达到这个效果。

**意义 →** 这是目前查到的、**跟PerceptDrive"冻结Perception Provider+训练轻量WAM Backbone"结构几乎一样的设计模式，在医学分割这个完全不同的垂直领域的独立验证**——进一步佐证了"冻结大模型主干+只训练轻量的token级MoE插件"这条路线是通用、跨领域都有效的设计哲学。

---

## 贡献列表(按重要性排序)

**① Expert Token + MoTE：用新增token而不是改权重做模态自适应**
SAM原本的输出token在训练过程中会逐渐"同质化"(论文指出这是SAM迁移到医学影像的根本瓶颈之一)，导致不同模态之间缺乏足够的区分度。SegMoTE的解法不是去微调SAM的权重，而是引入一组全新的、可学习的专家token，让这些token去承担"模态专属表征"的职责——SAM本身保持冻结、原有zero-shot能力不受影响，模态适应能力完全由这组新增token+MoTE路由机制提供。

**② MoTE机制：token级动态选择与融合**
给定一批输入(B张图×T个token×D维)，路由器算出每个token对E个专家的logits(Eq.1)，训练时注入噪声门控(Eq.2，借鉴经典Sparse MoE的噪声鼓励探索)；对每个token取其top-k专家里的最大置信分数c_{b,t}(Eq.4)当作该token被路由的可信度；路由权重G用这个置信分数做softmax(Eq.5)，直接当作一个"可靠性度量"去加权token表示本身(Eq.6)，而不是像常见MoE那样把多个专家的输出加权求和——这个设计让高置信度的token表示被放大、低置信度的被压低，是一种"置信度自适应"的token更新方式。

**③ 负载均衡损失：用变异系数而非常见的辅助损失**
L_balance(Eq.7-9)分别计算每个专家的重要性(importance，所有token对该专家的门控值之和)和负载(load，被路由到该专家的token数)，再用两者各自的平方变异系数(CV²，标准差平方/均值平方)之和作为损失。CV²越小说明不同专家之间的重要性/负载越均匀，这个损失鼓励专家被均衡利用，设置较小权重(λ_balance=0.01)避免压过主任务(分割)的学习。

**④ Progressive Prompt Tokenization(PPT)：无交互的全自动分割**
现有基于prompt的分割方法依赖用户手动提供点/框/mask提示，增加操作负担，尤其在稀疏类别分割任务(比如只有前景/背景两类的皮肤病变分割)上更明显。PPT把mask和文本prompt当作前景的具体表征，让一个可学习的query向量通过多头注意力对归一化图像特征做attention，逐步学会区分前景和背景，这些经过注意力增强的表征再经MLP投影+残差融合生成特征条件化的prompt token，训练完之后这些动态token本身就能承担prompt的功能，推理时不再需要人工交互。

**⑤ MedSeg-HQ数据集：用极少数据达到SOTA**
整合12个公开数据集(CHAOS T1/T2、ISIC 2016-2018、BTCV、ACDC、AMOS CT/MRI、WORD、TotalSegmentator CT、SZ-CXR)，覆盖6种模态、100多个语义类别，用5个专家按清晰度/对比度/熵/前景比例/连通区域等标准筛选高质量图像，最终154,569条标注，只有现有大规模数据集(COSMOS/IMed-361M)约1%的规模，但足够支撑SegMoTE达到SOTA。

---

## Novelty 分类判断

- **新问题定义**：不算——SAM适配医学图像这个方向已有不少工作(MedSAM/SAM-Med2D/IMIS)。
- **新insight**：部分算——"只需要token级的轻量插件、不需要微调SAM本身权重"这个思路，跟MoE-LoRA这类adapter方法理念上有重叠，但"用专家token而非参数/权重做模态自适应、并用置信分数直接加权token表示"这个具体切入点有一定新颖性。
- **新方法**：算——Expert Token+MoTE机制、Progressive Prompt Tokenization(用随机采样mask/文本prompt引导、实现无交互全自动分割)都是具体工程创新。
- **新理论**：没有，是已有MoE/注意力机制的巧妙组合应用。
- **新数据集**：算——MedSeg-HQ(整合12个公开数据集、154,569条精选高质量标注)是新贡献。
- **新应用场景**：不算，是医学图像分割这个已有任务。

综合结论：**新方法+新数据集组合类贡献**，核心价值是证明了"冻结大模型主干+轻量token级MoE插件"这个设计模式在医学分割这个垂直领域的有效性，而且用极少的训练数据(约1%)和参数(17M)就达到SOTA，效率优势明显。

---

## 中文精读

### 1. 解决了什么问题

医学图像分割需要处理多种成像模态(CT、MRI、X光、皮肤镜等)，而且像素级标注极其昂贵(需要专业医生标注)。SAM这类通用交互式分割基础模型在自然图像上表现出色，但迁移到医学影像时面临两个瓶颈：①缺乏模态/解剖结构感知的自适应机制，域外医学场景泛化差；②现有适配方法往往对大规模异构数据集不加选择地做微调，引入噪声监督、代价高、还可能造成负迁移(让模型在新模态上学歪，反而破坏了原有的zero-shot能力)。论文要回答：**怎样用最少的新增参数和训练数据，让冻结的SAM适配到任意医学影像模态？**

### 2. 新意在哪里

核心新意是把"模态适应"这件事完全外挂到一组新增的、轻量的token上，而不去碰SAM本身的任何权重。这跟常见的"微调decoder某几层"或"加LoRA改权重"的adapter思路不同——专家token是全新加入的、专门负责捕捉模态特定语义的载体，MoTE负责动态决定"当前这个token该参考哪个专家的经验"，整个过程SAM原本的权重分布完全不受影响，zero-shot能力自然保留。

### 3. Idea 具体落在哪里

**整体架构(Fig.2)：** 冻结的SAM编码器提取图像嵌入；专家token([N×256])跟SAM原有的output token([4×256])和prompt token沿序列维度拼接，一起送进decoder；两层decoder layer里，先做token间self-attention，再做token-to-image attention(整合视觉特征、prompt的几何/语义信息、mask表征)，更新后的专家token送进MoTE动态加权更新；MoTE输出的token再经image-to-token attention，最终跟图像特征逐点相乘生成分割mask预测。

**MoTE具体公式(§3.3)：** 输入专家token x∈R^(B×T×D)，路由器logits L=XW_g(Eq.1)；训练时的噪声门控L̃=L+(softplus(XW_n)+ε)⊙Z，Z~N(0,1)(Eq.2)，鼓励探索防止路由过早收敛到单一模式；对batch里第b张图第t个token，取其top-k专家的logits向量s_{b,t}(Eq.3)，用其中最大值当置信度c_{b,t}、对应专家索引idx_{b,t}(Eq.4)；路由权重G=softmax(c_{b,t})(Eq.5)；最终更新后的token表示z̃_{b,t}=G(·)_{b,t}·h_{b,t}^(idx)(Eq.6)——注意这里不是把多个专家的输出加权求和，而是只取置信度最高的那一个专家分支的表示，再用置信度本身去缩放这个表示(高置信度放大、低置信度压低)。

**Progressive Prompt Tokenization(Fig.3)：** 训练过程中，随机选取mask prompt或文本prompt当前景先验，可学习query Q通过多头注意力对归一化的图像特征做attention，逐步捕捉前景/背景的分布线索；注意力增强的表征再经MLP投影+残差融合，生成特征条件化的prompt token。这个方法目前明确限定在二分类(前景/背景)任务，论文坦诚说明多类别分割("class interference complicates accurate prompt token mapping")还没解决，所以这篇只在ISIC(皮肤病变)、SZ-CXR(胸片)这类二分类数据集上用PPT实现全自动分割。

**负载均衡(Eq.7-9)：** 重要性imp_e=所有token对专家e的门控值之和，负载load_e=被路由到专家e的token数量，L_balance是两者各自平方变异系数之和，CV²(x)=std(x)²/mean(x)²，衡量一组数值的不均匀程度。

### 4. 最大难点在哪里

不在"给SAM加专家token"这个大方向判断上，难点在于**怎么设计MoTE的更新机制，让路由决策既能区分不同模态/任务，又不会因为训练数据少(只有0.15M图像)而过拟合或专家坍缩**——论文用噪声门控+负载均衡损失两个机制共同约束这一点，而且MoTE直接用置信度缩放token表示(而非加权求和多个专家输出)这个设计，相比常见MoE在参数利用效率上更紧凑，这可能是在极少训练数据下依然能达到SOTA的关键原因之一(论文本身没有专门做这个角度的消融对比，是本笔记的推测，需要进一步验证)。

### 5. 局限 / 对我项目的相关性

**PPT目前只支持二分类(前景/背景)任务**，论文明确承认多类别分割(比如多器官CT分割)存在类别干扰问题，还未解决。

**只验证了SAM这一个基础模型的适配**，没有测试换成其他视觉基础模型(比如DINOv3)是否同样有效。

**MoTE的"只取置信度最高的单个专家、不做多专家加权求和"这个设计**，跟SAMoE-VLA的"多专家参数融合"、PerceptDrive的"场景条件路由融合"都不同，是更接近经典SparseMoE(Top-1选择)的路由范式，论文没有讨论这个选择vs融合的设计取舍，presentation时要讲清楚这是SparseMoE式的，不是SA-MoE式的参数融合。

**对我项目的相关性：这篇"冻结大模型主干+只训练轻量的token级MoE插件"这个设计模式，跟PerceptDrive"冻结Perception Provider+训练轻量WAM Backbone"结构几乎一样**，是同一个设计模式在医学分割这个完全不同领域的独立验证——进一步佐证了这条路线(而不是从头训练整个MoE系统)在跨领域场景下都有效，而且这篇用极少训练数据(0.15M图像，约为常见大规模数据集的1%)就达到SOTA，对"怎么用少量数据高效训练驾驶MoE模型的轻量路由部分"这个问题也有参考价值。

### 6. 是否能连 GitHub

**能，代码已开源**：https://github.com/InMyDreamer/SegMoTE

---

## 假设、局限与质疑点

- **MoTE的路由权重直接用来缩放单个专家的token表示，而不是加权融合多个专家的输出**，这个设计选择没有跟"多专家融合"的替代方案做对比消融，presentation时需要讲清楚这是经典Top-k稀疏选择范式，不是参数/输出融合范式。
- **MedSeg-HQ的质量筛选标准(清晰度/对比度/熵/前景比例/连通区域)由5个专家评估**，具体评估流程和一致性(专家间是否有分歧、怎么仲裁)论文没有详细展开。
- **域外泛化实验**(论文提到"both in-domain and out-of-domain datasets")的具体数据集划分和结果数字，本笔记还没有细读实验部分的完整表格，需要后续补充。

---

## 一个月后记住 5 件事

1. **核心设计：冻结SAM编码器和mask解码器，只新增一组可学习的专家token，用token级MoE(MoTE)做模态自适应**，SAM本身的权重和zero-shot能力完全不受影响。
2. **MoTE用置信度直接缩放单个被选中专家的token表示**(Top-k选择，取最高置信度那个)，不是多专家加权融合输出——是经典SparseMoE式的路由范式。
3. **Progressive Prompt Tokenization(PPT)实现无人工交互的全自动分割**，但目前只支持二分类(前景/背景)任务，多类别分割还有类别干扰问题待解决。
4. **MedSeg-HQ数据集：整合12个公开数据集，154,569条精选标注，只有现有大规模数据集约1%的规模**，但足够支撑SOTA性能。
5. **只用17M可训练参数、0.15M训练数据就达到SOTA，比第二好方法提升1%-6%**，代码已开源(github.com/InMyDreamer/SegMoTE)。

---

## 关键原文摘录(中英对照)

**① 核心设计定位**
> "We freeze the SAM encoder and train only lightweight MoTE, preserving SAM's original capability while extending it to new datasets."

中译：我们冻结SAM编码器，只训练轻量的MoTE，在把SAM扩展到新数据集的同时保留它原有的能力。
解读：这句话直接点明了全文的核心设计哲学——用轻量外挂模块适配，而不是改动基础模型本身，presentation时可以直接引用。

**② 效率优势数字**
> "Extensive experiments on both in-domain and out-of-domain datasets demonstrate that our method improves 1% to 6% over the second best approach by training only 17M parameters on 0.15M dataset."

中译：在域内和域外数据集上的大量实验表明，我们的方法只用0.15M数据集训练17M参数，就比第二好的方法提升了1%到6%。
解读：这组数字直接体现了方法的效率优势(远少于其他方法的参数量和数据量)，presentation里可以用来对比凸显"轻量外挂模块"这个设计的价值。

---

## English at-a-glance summary

| Aspect | Summary |
|---|---|
| **Problem** | Medical image segmentation faces modality heterogeneity and high pixel-level annotation cost; adapting SAM to medical imaging either lacks modality-awareness or requires costly, noisy fine-tuning on large heterogeneous datasets.<br>医学图像分割面临模态异质性和高标注成本；SAM适配医学影像要么缺乏模态感知能力，要么需要在大规模异构数据集上做昂贵、嘈杂的微调。 |
| **Novelty** | Freeze SAM's encoder and mask decoder; introduce learnable expert tokens with a token-level Mixture of Token Experts (MoTE) for modality-adaptive processing, plus Progressive Prompt Tokenization (PPT) for interaction-free binary segmentation.<br>冻结SAM的编码器和mask解码器；引入可学习专家token，用token级MoTE做模态自适应处理，配合PPT实现无交互二分类分割。 |
| **Core idea** | Modality adaptation can be fully delegated to a small set of newly added tokens, leaving the frozen foundation model's weights (and its zero-shot ability) untouched.<br>模态适应可以完全外挂到一小组新增token上，冻结基础模型的权重(和zero-shot能力)完全不受影响。 |
| **Hardest part** | Designing a token update rule that differentiates modalities/tasks without overfitting or expert collapse, given extremely limited training data (0.15M images).<br>在训练数据极少(0.15M图像)的情况下，设计一种既能区分模态/任务、又不过拟合或专家坍缩的token更新机制。 |
| **Evidence** | SOTA across diverse modalities and anatomical tasks using only 17M trainable parameters and 0.15M training images (~1% of existing large-scale datasets), improving 1-6% over the second-best method.<br>只用17M可训练参数和0.15M训练图像(约为现有大规模数据集的1%)就在多种模态和解剖任务上达到SOTA，比第二好方法提升1%-6%。 |
| **Limitation** | Progressive Prompt Tokenization only supports binary (foreground/background) segmentation; multi-class segmentation still suffers from class interference.<br>PPT只支持二分类(前景/背景)分割；多类别分割还有类别干扰问题。 |
| **Code / GitHub** | Released: github.com/InMyDreamer/SegMoTE<br>已开源：github.com/InMyDreamer/SegMoTE |

---

## 本文提到的词

| 术语 | 首次出现位置 |
|------|------------|
| Mixture of Token Experts(MoTE) | Abstract |
| Progressive Prompt Tokenization(PPT) | Abstract |
| Expert Token(专家token) | §3.2 |
| MedSeg-HQ | Abstract |
| Coefficient of Variation(CV²，变异系数) | §3.3 |
