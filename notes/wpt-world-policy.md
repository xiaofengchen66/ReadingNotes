# WPT: World-to-Policy Transfer via Online World Model Distillation

- **Venue / year:** arXiv 2511.20095, 2025 (CVPR 2026)
- **Authors:** Guangfeng Jiang, Yueru Luo, Jun Liu, Yi Huang, Yiyao Zhu, Zhan Qu, Dave Zhenyu Chen, Bingbing Liu, Xu Yan — 中国科学技术大学 + 香港中文大学(深圳) + 香港科技大学 + 华为基础模型部门
- **Link:** https://arxiv.org/abs/2511.20095 ([本地PDF副本](pdf/wpt-world-policy.pdf))
- **Code:** 未查到开源仓库，暂按未开源处理
- **Input / 输出:** 多视角摄像头图像序列 → 规划轨迹(训练阶段额外用world model预测未来占据/世界嵌入做监督)
- **精读说明(2026-10-22)：用户让我把本地下载的论文都精读一遍，这是优先级最高的几篇之一(驾驶场景里对"SA-MoE没有真正省计算量"这个局限最直接的参照)，已完整读过PDF正文。**

---

## TL;DR

**问题 →** 现有把世界模型(World Model)用进自动驾驶策略的方法分两类：①直接把世界模型集成进策略内部做特征演化和轨迹推理，但存在推理和运行时紧耦合、严重依赖准确的未来预测和自回归rollout，拖慢实时效率；②把世界模型当仿真器做闭环强化学习训练，但高度依赖仿真器的生成保真度，主要在合成环境里验证。

**Gap →** 怎样让自动驾驶策略利用世界知识，同时避免额外的计算开销？

**Novelty →** 提出WPT(World-to-Policy Transfer)训练范式：训练时让策略跟一个预训练好的世界模型交互，获得对未来动态的预判能力；但部署时用一个简单的轻量策略网络，完全不需要世界模型参与推理。具体由四部分组成：①教师策略(多模态，生成N条候选轨迹)；②世界模型(冻结，按动作条件自回归预测未来世界嵌入)；③可训练的奖励模型(结合模仿奖励+仿真奖励，评估每条候选轨迹跟预测未来世界状态的联合表示)；④世界知识蒸馏(策略蒸馏+世界奖励蒸馏两种损失，把教师的推理能力迁移给轻量学生策略)。

**核心洞察 →** 世界模型的价值可以完全限定在训练阶段体现——用它去生成"哪条候选轨迹更符合未来动态"这个判断，再把这个判断蒸馏进一个简单的学生网络，推理时不需要再付出世界模型的开销(自回归rollout、多候选轨迹生成+打分)。

**方法/论证 →** 教师的规划解码器P_D(Q, F^w)用cross-attention融合规划查询和世界表征(BEV特征)，生成N条候选轨迹；世界模型(预训练的Drive-OccWorld，训练时冻结)按动作条件自回归预测下一时刻的世界嵌入F^w_{t+1}，并反馈给教师的规划解码器做"世界模型增强规划"(Eq.8，多轮迭代)；奖励模型结合模仿奖励(候选轨迹跟专家轨迹的L2距离，经softmax归一化，Eq.11)和仿真奖励(预测NC/DAC/TTC/EP/Comf五个类NAVSIM式指标，Eq.12-13)，加权求和得到最终奖励(Eq.14)，选出最优轨迹；学生策略只用单次前向(不依赖世界模型、不生成多候选)直接解码轨迹(Eq.3)，通过策略蒸馏(学生和教师的规划查询做L2对齐，Eq.15)和世界奖励蒸馏(学生轨迹的奖励分数要逼近教师最优轨迹的奖励分数，Eq.16)学习教师的推理能力。

**结果 →** nuScenes开环评测，WPT-Teacher平均L2 0.61m、碰撞率0.11%(3秒预测碰撞率低至0.10%)，是对比方法里最好的，超过Drive-OccWorld(0.85m/0.29%)；WPT-Student(轻量、推理不用世界模型)保留了大部分提升(0.66m/0.14%)。Bench2Drive闭环评测，WPT-Teacher Driving Score 79.23、成功率54.54%、Efficiency 188.63，超过DriveTransformer(63.46 DS)、DriveDPO(62.02 DS)等强基线；WPT-Student紧随其后(DS 72.61、SR 45.45%、Efficiency 188.52)。摘要称**学生推理速度最高提升4.9倍**。

**意义 →** 这是目前查到的、**对"驾驶模型怎么又快又好"这个问题回答最完整的一篇**——把"世界模型指导+多候选轨迹+奖励打分"这套在推理时很贵的机制，完全限定在训练阶段，用蒸馏把效果迁移进一个轻量学生，部署时不付出任何世界模型开销。

---

## 贡献列表(按重要性排序)

**① World-to-Policy Transfer训练范式：训练时用世界模型，部署时完全不用**
这是全文最核心的设计哲学。跟"把世界模型直接塞进策略里做推理"(方法类别①)或"把世界模型当仿真器做RL"(方法类别②)都不同，WPT让策略在**训练时**跟世界模型交互获得预判能力，但部署的是一个**不依赖世界模型、单次前向**的轻量策略，彻底解决了"要不要为了利用世界知识而牺牲实时性"这个两难。

**② 可训练的奖励模型：模仿奖励+仿真奖励双重监督**
模仿奖励(Eq.11)衡量候选轨迹离专家轨迹的距离(越近奖励越高，softmax归一化)，捕捉人类驾驶偏好；仿真奖励(Eq.12-13)借鉴NAVSIM的PDM式指标，在**预测的未来世界状态**下评估候选轨迹的碰撞(NC)、可行驶区域合规(DAC)、碰撞时间(TTC)、自车进度(EP)、舒适度(Comf)五个维度，用二元交叉熵监督。最终奖励(Eq.14)是两者的加权对数和。这个设计让规划器评估轨迹时，参考的不是当前静态场景，而是世界模型预测出的、轨迹执行后的未来状态——这是"世界模型增强规划"的具体体现。

**③ 两种蒸馏目标：策略蒸馏+世界奖励蒸馏**
策略蒸馏(Eq.15)直接对齐学生和教师的规划查询表示(L2距离)，让学生继承教师结构化的规划意图，不需要自回归rollout就能获得未来感知的推理能力；世界奖励蒸馏(Eq.16)让学生生成的单一轨迹的奖励分数，逼近教师从N条候选里选出的最优轨迹的奖励分数——这一步隐式地把"世界模型告知的评价标准"传给了学生，学生虽然推理时看不到世界模型，但已经学会了"什么样的轨迹在未来世界状态下是好的"。

**④ 世界模型增强的教师规划(训练时的迭代refinement)**
教师策略把自己规划解码器的输出世界嵌入F^w，设成世界解码器W_D预测下一时刻世界状态F^w_{t+1}的输入，再用这个预测出的未来状态重新驱动规划(Eq.8)，形成一个训练时的自回归式精修循环，让教师生成的候选轨迹本身就带有对未来动态的预判。

---

## Novelty 分类判断

- **新问题定义**：部分算——"怎样让AD策略利用世界知识、同时避免额外计算开销"这个问题被专门提出，对应了此前两类方法各自的缺陷(紧耦合 vs 依赖仿真器)。
- **新insight**：算——"世界模型只在训练时需要、部署时可以完全省去"这个核心判断，有完整的teacher/student性能对比实验支撑(student保留大部分teacher的提升，同时推理快4.9倍)。
- **新方法**：算——奖励模型的双重监督设计(模仿+仿真奖励，在预测未来状态下评估)、两种蒸馏目标(策略蒸馏+世界奖励蒸馏)的组合都是具体工程创新。
- **新理论**：没有，是已有技术(世界模型、奖励建模、知识蒸馏)的系统性组合。
- **新数据集**：没有，用nuScenes和Bench2Drive已有基准。
- **新应用场景**：不算，是驾驶规划这个已有任务。

综合结论：**新insight+新方法组合类贡献**，核心价值是给"驾驶模型怎么既吸收世界模型的预判能力、又保持实时部署效率"这个矛盾提供了一个完整、经过充分验证的解法，是目前查到的回应"怎么让驾驶模型真正又快又好"这个问题最扎实的一篇。

---

## 中文精读

### 1. 解决了什么问题

自动驾驶的世界模型研究近年进展很多，但怎么把世界模型的"预判未来"能力用进实际部署的驾驶策略，一直面临两难：直接把世界模型塞进策略(比如Drive-OccWorld联合建模占据和流预测)会导致策略强依赖准确的未来预测和自回归rollout，严重拖慢实时推理；把世界模型当闭环RL训练的仿真器，又高度依赖仿真器生成数据的保真度，主要在合成环境(CARLA等)里验证，真实场景泛化性存疑。论文要回答：**能不能让策略学到世界模型蕴含的预判能力，但部署时完全不需要世界模型本身？**

### 2. 新意在哪里

核心新意是把"利用世界模型"这件事，从"推理时的架构组件"重新定位成"训练时的监督信号来源"。这个转变让WPT同时规避了两类此前方法的缺陷：不像方法①那样有运行时耦合和自回归开销(因为部署的学生策略单次前向)，也不像方法②那样依赖仿真器环境(因为WPT在真实数据nuScenes和半仿真的Bench2Drive上都直接训练+评测)。

### 3. Idea 具体落在哪里

**整体架构(Fig.2)：** 训练时(上半部分)，世界模型在给定动作条件下预测未来世界状态；教师AD策略(T)生成多模态候选轨迹；奖励模型评估这些候选轨迹，产出世界奖励。蒸馏阶段(下半部分)，学生AD策略(S)通过策略蒸馏(对齐规划表征)和世界奖励蒸馏(匹配教师最优奖励轨迹)向教师学习。

**世界模型结构(§3.2)：** 观测编码器W_E处理历史观测得到世界嵌入F_t^w(Eq.5)；特征聚合模块A_M聚合历史世界嵌入得到增强的未来预测特征(Eq.6)；世界解码器W_D在动作条件c_t下自回归预测下一时刻世界嵌入F_{t+1}^w(Eq.7)。

**奖励模型(Fig.3，§3.3)：** 把潜在世界表示、规划查询、轨迹编码器的输出拼接后，分别过模拟奖励头和模仿奖励头，两类奖励相加再取argmax选出最佳轨迹(Eq.10)。模拟奖励用二元交叉熵对齐预测世界状态下的NC/DAC/TTC/EP/Comf指标真值；模仿奖励用softmax归一化的L2距离(离专家轨迹越近权重越高)。

**蒸馏(§3.4)：** 训练后教师策略和世界模型都被丢弃，只保留紧凑的学生策略。策略蒸馏最小化学生和教师规划查询之间的L2距离(Eq.15)；世界奖励蒸馏最小化学生轨迹的最终奖励分数和教师最优轨迹奖励分数之间的L2距离(Eq.16)——这一步让学生隐式学到"世界模型会怎么评价这条轨迹"，而不需要学生自己拥有世界模型。

**消融(§4.4)：** Table 3验证了奖励模型各组成部分的贡献——只加模仿奖励让碰撞率从0.72%降到0.23%(0.70m L2)；在推理时也用模仿奖励效果更好(0.69m/0.22%)；加入仿真奖励收益最大，仅训练时用达到0.62m/0.14%，训练+推理都用达到最好的0.61m/0.11%。作者发现大部分收益来自训练时的监督，推理时的额外打分只带来小幅边际提升。

### 4. 最大难点在哪里

不在"用世界模型指导训练"这个大方向判断上，难点在于**怎么设计一套蒸馏机制，让学生即使看不到世界模型、也能隐式继承"世界模型会怎么判断轨迹好坏"这个知识**——这正是世界奖励蒸馏(Eq.16)要解决的问题：不是直接蒸馏世界模型本身的参数或特征，而是蒸馏"教师用世界模型辅助评估后选出的最优轨迹的奖励分数"，这是一种更间接、但计算上更轻量的知识迁移方式。

### 5. 局限 / 对我项目的相关性

**世界模型(预训练的Drive-OccWorld)训练时是冻结的**，论文没有讨论如果这个世界模型本身预测质量不够准，蒸馏出来的奖励信号是否会带偏学生策略。

**存在"效率-舒适度"的权衡**("We note a known efficiency–comfort trade-off")，论文原文承认这一点，并留给未来工作做奖励函数的精修。

**代码未开源**，查不到对应仓库。

**对我项目的相关性：这是"怎么让驾驶MoE/驾驶模型真正省计算量"这个局限目前查到的最完整解法之一**——虽然WPT本身不是MoE架构，但"训练时用重型机制(世界模型+奖励模型+多候选轨迹打分)、蒸馏进一个轻量学生、推理时完全不需要重型机制"这个设计模式，可以直接类比到SA-MoE/PerceptDrive这类驾驶MoE模型：如果SA-MoE"4个专家参数加权融合"这个推理时的计算负担，能像WPT一样被蒸馏进一个更轻量的单一"融合后"专家(推理时只跑这一个蒸馏出的专家，不需要每次都现场加权融合4个完整参数矩阵)，就有可能真正做到省计算量。另外，WPT用**外部冻结的预训练世界模型**在训练时监督，而PerceptDrive用的是**模型自己预测的action-free未来隐特征**——这是"怎么获得前瞻性"这个问题的两种不同路线(外部世界模型蒸馏 vs 自监督自预测)，值得对照。

### 6. 是否能连 GitHub

未开源，论文未提供代码链接。

---

## 假设、局限与质疑点

- **奖励模型的仿真奖励部分依赖NAVSIM式的5个指标**(NC/DAC/TTC/EP/Comf)，这些指标本身是否能完整刻画"好轨迹"是整个领域的共性局限，不是WPT独有，但presentation时要讲清楚这是个已有的评测范式局限。
- **只在nuScenes(开环)和Bench2Drive(闭环，基于CARLA)两个基准上验证**，没有在更复杂的真实世界长尾场景做过专门测试。
- **Table 1里WPT-Teacher和WPT-Student的输入/辅助监督标注都是"Camera"+"4D-Occ"**，但两者的具体推理路径差异(教师用世界模型+多候选打分，学生单次前向)在表格里不够直观，容易被误读成两者架构完全一样，presentation时要讲清楚两者的实际区别是"推理阶段是否依赖世界模型"。

---

## 一个月后记住 5 件事

1. **核心范式：世界模型只在训练阶段使用，部署时完全不需要**——学生策略通过蒸馏继承教师(带世界模型指导)的推理能力，单次前向出轨迹，不做自回归rollout或多候选打分。
2. **奖励模型双重监督：模仿奖励(贴近专家)+仿真奖励(在预测未来世界状态下评估碰撞/合规/舒适度等指标)**，两者加权求和选出最优候选轨迹。
3. **两种蒸馏目标：策略蒸馏(对齐规划查询)+世界奖励蒸馏(学生轨迹的奖励分数逼近教师最优轨迹奖励分数)**，后者是让学生隐式学到"世界模型会怎么评价"的关键机制。
4. **核心结果：WPT-Teacher nuScenes开环0.61m/0.11%碰撞率、Bench2Drive闭环79.23 Driving Score**，WPT-Student保留大部分提升、**推理最高快4.9倍**。
5. **这是回应"驾驶模型怎么又快又好"这个问题目前最完整的参照之一**，设计模式(训练时用重型机制、蒸馏进轻量学生、推理时完全不用重型机制)可以类比到MoE驾驶模型的参数融合开销问题。

---

## 关键原文摘录(中英对照)

**① 核心问题提出**
> "A natural question arises: how can the AD policy leverage world knowledge while avoiding extra computational overhead?"

中译：一个自然的问题随之而来：自动驾驶策略怎样才能利用世界知识，同时避免额外的计算开销？
解读：这是全文的核心问题，presentation开场直接用这句话能把问题讲清楚。

**② 核心结果数字**
> "WPT achieves state-of-the-art performance with a simple policy architecture: it attains a 0.11 collision rate (open-loop) and achieves a 79.23 driving score (closed-loop), surpassing both world-model-based and imitation-learning methods in accuracy and safety. Moreover, the student sustains up to 4.9× faster inference, while retaining most of the gains."

中译：WPT用一个简单的策略架构就达到了SOTA性能：开环碰撞率0.11%，闭环驾驶分数79.23，在精度和安全性上超过了基于世界模型和模仿学习的方法。此外，学生策略推理最高快4.9倍，同时保留了大部分提升。
解读：这段话里的具体数字是presentation里最有说服力的证据，"又快又好"这个核心卖点一句话就能讲清楚。

---

## English at-a-glance summary

| Aspect | Summary |
|---|---|
| **Problem** | World-model-integrated AD policies suffer from runtime coupling and autoregressive-rollout latency; simulator-based RL depends heavily on simulator fidelity.<br>直接集成世界模型的驾驶策略有运行时耦合和自回归延迟问题；基于仿真器的RL高度依赖仿真器保真度。 |
| **Novelty** | World-to-Policy Transfer: the policy interacts with a world model only during training; the deployed student is a lightweight, single-forward-pass network trained via policy distillation + world reward distillation.<br>世界到策略的迁移：策略只在训练时跟世界模型交互；部署的学生是单次前向的轻量网络，通过策略蒸馏+世界奖励蒸馏训练。 |
| **Core idea** | The value of a world model can be fully captured at training time and distilled away, so deployment needs none of its overhead.<br>世界模型的价值可以完全在训练阶段被捕获并蒸馏出来，部署时不需要它的任何开销。 |
| **Hardest part** | Designing a distillation signal (world reward distillation) that transfers "what the world model would judge as good" without the student ever seeing the world model.<br>设计一种蒸馏信号(世界奖励蒸馏)，让学生在从不接触世界模型的情况下，学到"世界模型会如何评判"。 |
| **Evidence** | WPT-Teacher: 0.61m L2 / 0.11% collision (nuScenes open-loop), 79.23 Driving Score (Bench2Drive closed-loop); WPT-Student retains most gains with up to 4.9x faster inference.<br>WPT-Teacher：nuScenes开环0.61m/0.11%碰撞率，Bench2Drive闭环79.23驾驶分数；WPT-Student保留大部分提升、推理最高快4.9倍。 |
| **Limitation** | Frozen world model quality isn't interrogated; a known efficiency-comfort trade-off is left for future work; no code released.<br>没有讨论冻结世界模型本身质量的影响；存在已知的效率-舒适度权衡，留给未来工作；未开源。 |
| **Code / GitHub** | Not released.<br>未开源。 |

---

## 本文提到的词

| 术语 | 首次出现位置 |
|------|------------|
| World-to-Policy Transfer(WPT) | Abstract |
| Policy Distillation(策略蒸馏) | §3.4 |
| World Reward Distillation(世界奖励蒸馏) | §3.4 |
| Imitation Reward / Simulation Reward(模仿奖励/仿真奖励) | §3.3 |
| World Model Enhanced Planning(世界模型增强规划) | §3.1 |
