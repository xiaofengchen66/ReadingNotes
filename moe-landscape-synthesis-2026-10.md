# MoE 全景知识图谱(CVPR 2026批次，2026-10-21)

> 把这个repo里所有跟MoE相关的论文(驾驶专用 + CVPR 2026通用MoE批量摘要级整理)按"解决的是哪个层面的问题"分组，标出谁能直接回应谁。驾驶专用的几篇是精读过的，通用MoE这23篇目前只是摘要级([阅读进度说明]里有标注)，细读后再回填具体结论。

## 分组一：驾驶专用MoE(已精读/精读中)

| 论文 | 专家按什么划分 | 路由方式 | 核心主张 |
|---|---|---|---|
| [DriveMoE](notes/drivemoe.md) | 摄像头视角(Vision MoE) + 驾驶行为(Action MoE) | token级，预定义router标签 | 被SAMoE-VLA点名批评"引入安全隐患"的那篇 |
| [SAMoE-VLA](notes/samoe-vla.md) | 4个通用FFN专家 | scene级，参数融合(非选择) | token级路由颗粒度跟场景级决策不匹配，碰撞率+38.4% |
| [ARTEMIS](notes/artemis.md) | — | 真正路由(input-dependent) | 自回归轨迹规划 |
| [WAM-Diff](notes/wam-diff.md) | — | 真正路由(input-dependent) | 单图输入 |
| [PerceptDrive](notes/perceptdrive.md) | 几何/语义/动态(蒸馏自外部基础模型) | scene级，冻结专家+metric蒸馏监督路由 | 冻结感知先验+轻量路由，跟朱子欣idea机制最接近 |
| [ExpertAD](notes/expertad.md) | 环境/自车状态/导航(8个专家，**仅存在于Prediction模块**) | Top-4稀疏选择+噪声门控 | ⚠精读后修正：Perception段(PA)是特征重加权非MoE，Planning段完全复用基线，不是"全程"；AAAI 2026已接收 |
| [Drive-KD](notes/drive-kd.md) | 感知/推理/规划三元组 | 无路由(纯蒸馏) | "全程"覆盖最广，但完全没有MoE结构 |

## 分组二：通用MoE技术，按解决的问题分类(CVPR 2026，摘要级)

### A. 路由颗粒度该多大(呼应SAMoE-VLA的核心论点)
- [NESTOR](notes/nestor.md) — image级MoE(全局) + token级Sub-MoE(局部)，PDE领域独立得出"要分层处理全局/局部粒度"
- [local-precise-refinement](notes/local-precise-refinement.md) — token级双门控路由的对照样本

### B. 路由器该怎么训练(呼应PerceptDrive的"privileged metric distillation"设计)
- [MoE-GRPO](notes/moe-grpo.md) — 用强化学习(GRPO)训练路由，而非监督信号
- [Teacher-Guided Routing (TGR-MoE)](notes/tgr-moe.md) — 用预训练dense教师的中间表示构造伪监督信号训练router
- [TAG-MoE](notes/tag-moe.md) — 把高层任务语义意图注入路由决策，跟"scene-conditioned routing"是同一方向在生成任务上的独立验证

### C. 怎么让MoE真正省计算量(直接回应"SA-MoE没有真正省计算量"这个已识别的局限)
- [MoDES](notes/modes.md) — 训练无关的动态专家跳过，88%专家被跳过反而精度更高
- [UniMMAD](notes/unimmad.md) — MoE-in-MoE结构，MoE参数量减少约75%
- [YOLO-Master](notes/yolo-master.md) — 按场景复杂度动态分配计算资源，"省计算"是核心卖点

### D. 持续学习/在线更新时路由会漂移(呼应"H_BEV只算一次、场景突变路由滞后"这个局限)
- [On Token's Dilemma (LLaVA-DyMoE)](notes/on-tokens-dilemma.md) — 漂移感知的token分配，防止旧token被新专家错误吸引
- [Spectral MoE](notes/spectral-moe-continual.md) — 频谱正交掩码+双路由器+几何约束，代码已开源

### E. 专家从哪来：训练 vs 蒸馏 vs 初始化(呼应"全程MoE+distill别的MoE"这个方向)
- [Cluster-Aware Upcycling](notes/cluster-aware-upcycling.md) — 从dense模型语义聚类初始化专家(不是蒸馏外部模型，是升级同一个dense模型)
- [SegMoTE](notes/segmote.md) — 冻结SAM编码器+只训练轻量token级MoE，跟PerceptDrive"冻结Provider+轻量WAM"是同一个设计模式在医学分割的独立验证
- [MoEActok](notes/moeactok.md)、[moe-fd-scene-graph](notes/moe-fd-scene-graph.md)、[M4-SAM](notes/m4-sam.md)、[quant-experts](notes/quant-experts.md)、[PromptMoE](notes/promptmoe.md)、[life-iqa](notes/life-iqa.md) — 专家按"技能/物体-关系/模态/token重要性/图像区域/失真类型"等不同维度划分的具体案例，可作为"专家该怎么分工"这个设计问题的案例库

### F. 路由本身的失效模式(设计自己的MoE时要主动规避)
- [VidPrism](notes/vidprism.md) — 明确命名"专家同质化"(expert homogenization)：没有差异化驱动信号，多个专家会退化成功能重复的"假专精"
- [ERMoE](notes/ermoe.md) — 用专家自身的正交基代替学习出的门控做路由，解决路由不稳定、专家利用率不均的问题
- [MoRE](notes/more-3d-geometry.md) — 先做几何编码再MoE路由，跟PerceptDrive的DSE结构相似，可做对照

### G. 驾驶公司/非驾驶专用，但值得留意
- [SMoES](notes/smoes.md) — 理想汽车(Li Auto)出品，虽非驾驶专用但来自汽车公司，MoE-VLM模态路由方向，值得跟踪这个团队后续是否发驾驶专用工作

## 跟朱子欣idea、乔老师方向的直接映射

| 已识别的空子/局限 | 可以直接参考的技术 |
|---|---|
| SA-MoE没有真正省计算量 | MoDES(动态跳专家)、UniMMAD(MoE-in-MoE)、YOLO-Master(场景复杂度自适应计算) |
| H_BEV只算一次、场景突变路由滞后 | On Token's Dilemma(漂移感知token分配)、Spectral MoE(几何约束防漂移，代码已开源) |
| "蒸馏别的MoE/冻结专家+轻量路由"这个机制 | SegMoTE(冻结SAM+轻量MoE，验证了同一设计模式在另一领域可行)、Cluster-Aware Upcycling(另一种不靠外部蒸馏的专家初始化方式，作为对照) |
| 路由器训练信号的选择 | MoE-GRPO(RL信号) vs TGR-MoE(教师监督信号) vs PerceptDrive(privileged metric蒸馏信号)——三种不同训练路由器的范式，可以写成一张对比表 |

## 下一步

这23篇目前只是摘要级整理，还没有精读。按你说的"细读时一点一点加进对应单篇笔记"，建议细读顺序：**MoDES、UniMMAD、On Token's Dilemma、Spectral MoE、SegMoTE** 这5篇，因为它们分别直接对应上面表格里已经识别出的4个具体空子/局限，细读收益最高；其余18篇可以之后按需补。

---

## 补充：Distillation批次(30篇，2026-10-21)筛选后的高价值发现

从90篇Distillation列表里按"跟CV/驾驶/VLM相关"筛出30篇处理，以下几篇对研究方向有直接意义：

### 最重要的三个发现

1. **[WPT](notes/wpt-world-policy.md)** — 世界模型指导在线蒸馏出轻量驾驶策略，学生推理快4.9倍、闭环driving score 79.23超过基于世界模型和模仿学习的方法。**这是目前查到的、跟"怎么让驾驶MoE真正省计算量"这个局限最直接的完整解法**，建议优先精读。

2. **[SigLino](notes/siglino.md)** — 多教师蒸馏的目标**明确包括MoE学生**(不只是Dense)，是"蒸馏进MoE"这半个机制的独立验证；但教师依然是通用基础模型(SigLIP2/DINOv3)，不是驾驶专用MoE——跟PerceptDrive共同构成"蒸馏机制已被验证可行，但没人蒸的是别的驾驶MoE系统"这个判断的证据。

3. **[GTR-Turbo](notes/gtr-turbo.md)** — "合并RL训练checkpoint当免费教师"，跟SAMoE-VLA的参数融合是同一类权重空间操作技巧，而且这篇明确算出了具体的训练时间(-50%)和算力成本(-60%)节省数字——是回应"SA-MoE没有真正省计算量"这个局限的又一个具体参照。

### H_BEV滞后问题的第三、第四种解法(补充此前讨论的"定期刷新"/"变化检测触发"两种)

- **[AMap](notes/amap.md)**："从未来蒸馏"——训练时用能看到未来的教师，把前瞻性蒸馏进学生，推理时零额外成本就有"向前看"能力，不需要在线重算。
- **[Reward Forcing](notes/reward-forcing.md)** 的EMA-Sink机制——不是"只算一次"也不是"定期重算"，而是用指数滑动平均持续、低成本地融合新信息，是第三条路线。
- **[Prompt-Anchored ReID](notes/prompt-anchored-reid.md)** 的"新增槽位+冻结旧槽位"——是持续学习防遗忘的第三种机制(对照之前的on-tokens-dilemma漂移感知token分配、spectral-moe-continual频谱正交)。

### 其他跟PerceptDrive设计模式直接对照的案例

- **[GKD(语义分割)](notes/gkd-semantic-seg.md)** — "先学领域无关表示、冻结、再任务适配"两阶段范式，跟PerceptDrive的Stage 1/Stage 2结构几乎一样，而且专门讨论了"怎么蒸馏才不破坏泛化能力"。
- **[Expert-Teacher-Student](notes/expert-teacher-student-dom-adapt.md)** — 三层(专家/教师/学生)协作结构，是PerceptDrive两层(冻结Provider/可训Backbone)结构的一个扩展版本，多了一层域适配。
- **[4D-RGPT](notes/4d-rgpt.md)** — 冻结专家蒸馏进MLLM，同一设计模式在4D理解任务上的例证。

### 驾驶专用，补充进分组二

- **[AMap](notes/amap.md)**、**[Deformable Gaussian Occupancy](notes/deformable-gaussian-occupancy.md)**、**[SemLT3D](notes/semlt3d.md)**、**[WPT](notes/wpt-world-policy.md)** 四篇明确是驾驶场景(HD地图/占据栅格/长尾3D检测/世界模型策略蒸馏)，但都不是MoE(除了SemLT3D有语言引导的MoE路由组件)，是"蒸馏"这条技术路线在驾驶具体子任务上的独立应用，可以跟分组一的MoE驾驶论文对照，看蒸馏和MoE这两条技术路线在驾驶场景里各自的优势区间。

---

## 补充：Driving models/planning批次(43篇，2026-10-21)

CVPR 2026 Driving列表里"driving models/planning"这个核心类目(共45篇，DriveMoE已在分组一处理，EE-RL找不到可用链接跳过)，覆盖VLA/规划/世界认知等跟本repo研究方向最直接相关的论文。

### 最重要的发现

1. **[The Blind Spot of Adaptation (DEA)](notes/blind-spot-adaptation.md)** — **这批里跟路由漂移/遗忘问题关系最直接、也最驾驶专用的一篇**。明确指出VLM微调适配驾驶数据会侵蚀预训练世界知识，提出把适配从权重空间转移到prompt空间，按场景线索动态路由到不同知识专家来规避这个权衡。建议跟on-tokens-dilemma、spectral-moe-continual、prompt-anchored-reid这几篇的解法放在一起系统对比，这是目前查到的"路由漂移"问题最贴近驾驶场景的答案。

2. **[DrivePI](notes/drivepi.md)** — **"全程"覆盖做得最完整的单一模型**：统一理解+感知(3D占据)+预测(占据流)+规划(动作)，端到端联合优化，只用0.5B参数的Qwen2.5就超过7B的VLA模型。是跟朱子欣讨论"全程MoE"时最该参考的反例/对照：不用MoE结构、不用蒸馏，靠统一架构+小模型也能做到全程覆盖，这提示"全程覆盖"本身可能不需要MoE/蒸馏才能实现，MoE/蒸馏的价值可能更在别处(比如本repo已经在讨论的计算效率、安全长尾、持续学习)。

3. **[EventDrive](notes/eventdrive.md)** — **驾驶专用的真正MoE**，路由维度是"传感器模态"(事件流 vs 帧)，跟SAMoE-VLA(场景)、ExpertAD(功能子任务，但仅在Prediction一段)、SemLT3D(类别语义)、MoEActok(技能)一起构成一个较完整的"专家按什么维度划分"案例库。

### ⚠ 2026-10-22更正：ExpertAD精读后发现不构成"全程MoE"证据

此前摘要级笔记一度以为ExpertAD可能横跨感知+规划两段(因为abstract写"end-to-end MoE architecture that spans perception and planning")，精读Methodology+Fig.2后发现：**真正的MoE(MoSE，8专家+Top-4路由)只存在于Prediction模块**，Perception段的PA(Perception Adapter)是基于约束优化的软通道重加权，没有专家池/路由选择，不是MoE；Planning段直接复用基线解码器(UniAD/VAD/VADv2)，完全没有改动。

这次更正让"全程(感知+预测+规划三段都做MoE路由)目前没人做过"这个结论**更加确定**——此前以为ExpertAD可能是潜在的两段候选，现在确认它实际只是单段(预测)MoE，连DriveMoE那种两段(感知+规划)都不如。查了96+篇、精读了关键候选(PerceptDrive/SigLino/DEA/ExpertAD)之后，这个空子依然成立，而且比最初判断时更稳固。

**给后续研究的提醒**：引用ExpertAD时不要直接照搬它abstract的"spans perception and planning"说法，这跟实际架构不符。

### ⚠ 命名混淆警告(整理材料时务必注意)

这批里出现了两次跟PerceptDrive相关术语撞名、但完全是不同论文的情况：

- **[Percept-WAM](notes/percept-wam.md)**(arXiv 2511.19221) vs **PerceptDrive**(arXiv 2607.20175)：名字和研究方向都很像("感知增强的世界感知-动作模型" vs "感知先验世界-动作建模")，但Percept-WAM是单一VLM直接输出统一感知token，**没有**PerceptDrive那种冻结专家+场景条件路由机制。
- **[WAM-Flow](notes/wam-flow.md)**(arXiv 2512.06112，复旦) vs PerceptDrive的**WAM Backbone**(World-Action Model Backbone，PerceptDrive笔记里的Stage 2规划主干)：WAM-Flow是一篇独立论文(离散流匹配规划器)，"WAM"这个缩写只是巧合撞上了PerceptDrive笔记里对自家Stage 2模块的称呼。

跟朱子欣/乔老师汇报时，这三者(Percept-WAM、WAM-Flow、PerceptDrive的WAM Backbone)一定要分清楚，避免引用错论文。

### 省计算量的更多案例(呼应"SA-MoE没有真正省计算量"这个局限)

- **[ColaVLA](notes/colavla.md)** — 每次决策只需两次VLM前向传播
- **[Prune2Drive](notes/prune2drive.md)** — 即插即用多视角VLM token剪枝，不需要重训
- **[Driving on Registers](notes/driving-on-registers.md)** — 相机感知寄存器token压缩多摄像头特征，吞吐量提升3倍
- **[Unifying Language-Action (LinkVLA)](notes/unifying-language-action.md)** — 共享离散码本+两步生成，推理时间-86%

这四篇加上之前Distillation批次找到的WPT(4.9倍)、Foundry(边缘蒸馏)，已经有6个独立的"怎么让驾驶VLA/MoE真正变快"的具体方案，可以整理成一张方法对比表。
