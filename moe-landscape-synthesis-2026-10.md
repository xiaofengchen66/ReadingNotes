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
| [ExpertAD](notes/expertad.md) | 地图/跟踪/速度/朝向/导航(8个功能专家) | token级，Top-4稀疏选择 | 经典稀疏MoE路线，AAAI 2026已接收 |
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
