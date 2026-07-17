# DriveVLM: The Convergence of Autonomous Driving and Large Vision-Language Models

- **Venue / year:** CoRL 2024
- **Authors:** Xiaoyu Tian, Junru Gu, Bailin Li, Yicheng Liu, Yang Wang, Zhiyong Zhao, Kun Zhan, Peng Jia, Xianpeng Lang, Hang Zhao (Tsinghua IIIS + Li Auto)
- **Link:** https://arxiv.org/abs/2402.12289
- **Local PDF:** `papers/autonomous_driving_moe/2402.12289v5.pdf`
- **paper_matrix.csv id:** `drivevlm`

## 中文精读

### 1. 解决了什么问题

传统自动驾驶 pipeline(处理流水线)是"感知→预测→规划"三段式,每段都是专门的神经网络模块。这套系统的死穴在于**长尾场景理解(long-tail scenario understanding)**——"长尾"指现实中占比很小但种类繁多、几乎不可能在训练数据里穷举的稀有场景:路上突然倒下一棵树、警察打手势指挥、一群牛横穿马路——传统感知模块要么没见过(检测不出来),要么检测到了也不知道这个物体对驾驶决策意味着什么。传统系统擅长"看见物体、预测轨迹",不擅长"理解语义、做常识推理"。

DriveVLM 要解决的问题是:能不能把 VLM(Vision-Language Model,视觉-语言大模型——把视觉编码器和大语言模型拼在一起、既能看图又能推理的大模型)的看图说话 + 推理能力,直接转化成自动驾驶可用的场景理解和规划输出?

### 2. 新意在哪里

新意不是"用了 VLM"(2024 年已不算新),而是**把 VLM 的输出结构化成一条可执行的规划链条**,一个三段式 Chain-of-Thought(CoT,思维链——让模型把推理过程拆成一步步中间输出,而不是直接跳到答案):

场景描述(天气/道路/车道)→ 场景分析(关键物体的静态属性/运动状态/特殊行为 + 对自车的"影响")→ 分层规划(hierarchical planning:meta-action〔元动作,如"减速""变道"这类粗粒度动作单元〕→ 决策描述 → 具体轨迹点/waypoints)

第二个新意是 **DriveVLM-Dual**:VLM 推理慢(几百毫秒级),没法实时开车。把 VLM 当"慢系统"(对应 Kahneman 双过程理论里的 System 2,即系统2慢思考)生成粗轨迹作参考,再用传统快速规划器(对应 System 1,系统1快思考)做高频微调,这个"快慢双系统"设计是论文能上真车的关键。

### 3. Idea 具体落在哪里

最值得学的是 **3.4 节分层规划**:不直接让 VLM 吐出精确坐标,而是先生成 17 类离散 meta-action(如"减速""变道左"),再生成带主语/动作/时长的决策描述,最后才生成轨迹点。本质是给 VLM 的推理搭"脚手架"——让语言模型在它擅长的抽象决策层面发挥,把精确数值层面交给最后一步或交给 DriveVLM-Dual 里的传统规划器。

### 4. 最大难点在哪里

不在模型结构,而在 **Section 4(数据集构建)和 Appendix B(评估指标)**:

- 长尾场景怎么挖?用 CLIP(Contrastive Language-Image Pretraining,对比语言-图像预训练——一个能把图片和文字映射到同一向量空间、从而支持"用一句话搜图"的模型)做语言查询检索,从海量行车记录里挖"路障""动物横穿"等稀有场景,工程量很大。
- 怎么评估"用语言描述场景"对不对?场景描述没有唯一正确答案,没法字符串匹配。他们用 GPT-4 当裁判(这类做法业内统称 LLM-as-judge,用大模型代替人工评分),把生成描述和标准答案拆成"关键信息点"逐点打分、幻觉(hallucination,模型编造出输入里没有的信息)倒扣分。meta-action 序列用改造版最长公共子序列(Longest Common Subsequence, LCS)动态规划(Dynamic Programming, DP),还要处理语义等价的不同表达。
- 这套评估体系的可信度很大程度依赖 GPT-4 打分器本身——这是所有"语言化输出"论文共同的软肋。

### 5. 局限 / 对我项目的相关性

- 驾驶场景理解,不是 MoE(Mixture of Experts,混合专家——用一个路由/门控网络把输入分派给不同"专家"子网络处理的架构),也不是 time-series/traffic imputation(时间序列/交通数据插补)。核心可迁移的不是方法而是**设计范式**:用"结构化中间表示"把黑箱模型的输出拆成可解释的阶段。
- 对老师的长期 modular AD MoE 方向有用:DriveVLM-Dual 的"慢系统(语言/推理)+ 快系统(数值/实时)"分工,和后续 gated LoRA / expert routing 的"专家做认知、backbone 做执行"思路是同一类设计哲学。

### 6. 是否能连 GitHub

**不能直接复现。** 有项目主页(`tsinghua-mars-lab.github.io/DriveVLM`),但代码和 SUP-AD 数据集都未开源(SUP-AD 是理想汽车内部专有数据)。只能作为架构设计参考,不能拉仓库跑起来。

---

## English at-a-glance summary

![DriveVLM summary poster](img/drivevlm_summary.svg)

| Aspect | Summary |
|---|---|
| **Problem** | Traditional perception→prediction→planning pipelines fail on long-tail scenes (fallen trees, police hand signals, animals crossing) because they lack semantic/common-sense reasoning. |
| **Novelty** | Structures a VLM's output into an interpretable 3-stage Chain-of-Thought (scene description → scene analysis → hierarchical planning) instead of an end-to-end black box. |
| **Core idea** | Hierarchical planning as a scaffold: meta-actions (17 discrete categories) → decision description → waypoints, letting the LLM reason at the abstract level while numeric precision is handled downstream. |
| **Key engineering trick** | DriveVLM-Dual: a slow VLM branch (System 2, low-frequency reference trajectory) + a fast traditional planner (System 1, high-frequency refinement) — makes onboard real-time deployment possible (~410ms on OrinX). |
| **Hardest part** | Building the SUP-AD dataset (CLIP-based long-tail scenario mining) and designing a GPT-4-based evaluation protocol for subjective scene descriptions + a DP-based meta-action sequence matcher. |
| **Evidence** | SOTA on nuScenes planning (L2/collision) when paired with VAD; deployed and tested on a production vehicle. |
| **Limitation for our project** | Vision-language driving system, not MoE, not time-series/traffic imputation. Transferable value is the *design pattern* (structured intermediate representations, slow/fast dual system), not the method itself. |
| **Code / GitHub** | **Not released.** Project page only (`tsinghua-mars-lab.github.io/DriveVLM`); SUP-AD dataset is proprietary (Li Auto). Cannot be cloned/reproduced. |
